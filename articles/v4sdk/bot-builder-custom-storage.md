---
title: Implémenter un stockage personnalisé pour votre bot | Microsoft Docs
description: Comment créer un stockage personnalisé dans le kit SDK Bot Framework v4.0
keywords: personnalisé, stockage, état, dialogue
author: johnataylor
ms.author: johtaylo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 10/31/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 4503e2953543d2ec9c06e8cd60484a5c87d95987
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/11/2019
ms.locfileid: "54224004"
---
# <a name="implement-custom-storage-for-your-bot"></a>Implémenter un stockage personnalisé pour votre bot

Les interactions d’un bot s’articulent autour de trois éléments : d’abord, l’échange d’activités avec Azure Bot Service, puis le chargement et l’enregistrement de l’état du dialogue avec un Store et enfin un service back-end dont le bot a besoin pour faire son travail.

![diagramme de scale-out](../media/scale-out/scale-out-interaction.png)

Dans cet article, nous allons explorer la sémantique des interactions du bot avec Azure Bot Service et le Store.

Bot Framework propose une implémentation par défaut qui répondra très probablement aux besoins d’un grand nombre d’applications. Pour l’utiliser, il suffit d’assembler les éléments ensemble avec quelques lignes de code d’initialisation. De nombreux exemples illustrent exactement cela.

Toutefois, l’objectif ici est de décrire ce que vous pouvez faire lorsque la sémantique de l’implémentation par défaut ne fonctionne pas comme vous le voudriez au sein de votre application. Le point fondamental est qu’il s’agit d’une infrastructure et non pas d’une application prédéfinie pourvue d’un comportement fixe. En d’autres termes, l’implémentation d’un grand nombre des mécanismes dans l’infrastructure est simplement l’implémentation par défaut et non pas la seule implémentation.

Plus précisément, l’infrastructure n’impose pas de relation entre l’échange d’activités avec Azure Bot Service et le chargement ainsi que l’enregistrement de n’importe quel état de bot. Elle fournit simplement une valeur par défaut. Pour illustrer encore plus ce point, nous développerons une implémentation alternative possédant une sémantique différente. Cette solution alternative réside aussi bien dans l’infrastructure et peut même être plus appropriée pour l’application en développement. Cela dépend entièrement du scénario.

## <a name="behavior-of-the-default-botframeworkadapter-and-storage-providers"></a>Comportement des fournisseurs par défaut BotFrameworkAdapter et Stockage

Tout d’abord, examinons l’implémentation par défaut fournie avec les packages d’infrastructure, comme illustré dans le diagramme de séquence suivant :

![diagramme de scale-out](../media/scale-out/scale-out-default.png)

À la réception d’une activité, le bot charge l’état correspondant à cette conversation. Il exécute ensuite la logique de dialogue avec cet état et l’activité qui vient d’arriver. Dans le processus d’exécution de dialogue, une ou plusieurs activités sortantes sont créées et envoyées immédiatement. Lorsque le traitement du dialogue est terminé, le bot enregistre l’état mis à jour, en remplaçant l’ancien état par le nouveau.

Il est intéressant d’examiner que deux choses peuvent mal se passer avec ce comportement.

Tout d’abord, si l’opération d’enregistrement échoue pour une raison quelconque, l’état est implicitement désynchronisé avec celui vu sur le canal, car l’utilisateur, qui a vu les réponses, a l’impression que l’état a avancé alors que ce n’est pas le cas. C’est souvent pire si l’état et la messagerie de réponse ont réussi. Cela peut avoir des conséquences sur la conception de la conversation. Par exemple, le dialogue peut inclure des échanges de confirmation supplémentaires et redondants avec l’utilisateur. 

D’autre part, si l’implémentation est déployée sur plusieurs nœuds, l’état peut être accidentellement remplacé. Cela peut être particulièrement déroutant, car le dialogue aura probablement envoyé des activités vers le canal transportant des messages de confirmation. Prenons l’exemple d’un bot de commande de pizza. Si l’utilisateur modifie la garniture et ajoute des champignons puis du fromage tout de suite après, dans un scénario scale-out avec plusieurs instances en cours d’exécution, les activités suivantes peuvent être envoyées simultanément vers différentes machines exécutant le bot. Dans un tel cas, il existe quelque chose appelé « condition de concurrence » où une seule machine est susceptible de remplacer l’état écrit par une autre. Toutefois, dans notre scénario, étant donné que les réponses ont déjà été envoyées, l’utilisateur a reçu une confirmation que le champignon et le fromage ont été ajoutés. Malheureusement, lorsque la pizza arrive, elle ne contient que des champignons ou du fromage, mais pas les deux.

## <a name="optimistic-locking"></a>Verrouillage optimiste

La solution consiste à introduire certains verrouillages autour de l’état. Le verrouillage particulier que nous utilisons ici est appelé verrouillage optimiste car nous maintiendrons toutes les exécutions comme si chacune d’elles était la seule, puis nous détectons toutes les violations d’un accès concurrentiel une fois le traitement effectué. Cela peut sembler compliqué, mais c’est très facile à concevoir à l’aide des technologies de stockage cloud et des points d’extension appropriés dans l’infrastructure du bot.

Nous allons utiliser un mécanisme HTTP standard basé sur l’en-tête de l’étiquette d’entité (ETag). La compréhension de ce mécanisme est cruciale pour comprendre le code qui suit. Le diagramme suivant illustre la séquence.

![diagramme de scale-out](../media/scale-out/scale-out-precondition-failed.png)

Le diagramme illustre le cas de deux clients effectuant une mise à jour sur une ressource. Lorsqu’un client émet une requête GET et qu’une ressource est retournée par le serveur, elle est accompagnée par un en-tête ETag. L’en-tête ETag est une valeur opaque qui représente l’état de la ressource. Si une ressource est modifiée, l’ETag sera mise à jour. Lorsque le client a terminé sa mise à jour vers l’état, il la renvoie au serveur. En faisant cette requête, le client attache la valeur ETag reçue précédemment dans un en-tête de condition préalable If-Match. Si cette ETag ne correspond pas à la valeur, le serveur retourne (sur une réponse, vers n’importe quel client) un échec de vérification de condition préalable avec un échec de précondition 412. Cet échec indique au client effectuant la requête POST que la ressource a été mise à jour. En voyant cet échec, le comportement habituel d’un client sera d’effectuer une nouvelle requête GET pour obtenir la ressource, d’appliquer la mise à jour qu’il souhaitait et publier (POST) la ressource de nouveau. Ce deuxième POST sera réussi, en supposant bien sûr, qu’aucun autre client n’est venu et n’a mis à jour la ressource, et si c’est le cas, le client devra simplement réessayer.

Ce processus est appelé « optimiste », car le client, ayant reçu une ressource, effectue son traitement, la ressource elle-même n’est pas « verrouillée » en ce sens que les autres clients peuvent y accéder sans aucune restriction. Tout conflit entre des clients concernant l’état de la ressource n’est pas déterminé jusqu’au traitement. En règle générale, dans un système distribué, cette stratégie est plus efficace que l’approche opposée « pessimiste ».

Le mécanisme de verrouillage optimiste que nous avons abordé suppose que la logique de programme peut être retentée en toute sécurité, inutile de préciser que la chose importante à prendre en compte ici est ce qui se passe pour les appels de service externe. La solution idéale ici est de rendre ces services idempotents. En informatique, une opération idempotente est une opération n’entraînant aucun effet supplémentaire en cas d’appels multiples avec les mêmes paramètres d’entrée. Les services REST HTTP purs qui implémentent GET, PUT et DELETE correspondent à cette description. Le raisonnement ici est intuitif : nous pouvons faire une nouvelle tentative de traitement et le fait d’effectuer les appels dont il a besoin sans que cela n’ait des effets supplémentaires du fait d’avoir été effectués dans le cadre d’une nouvelle tentative est une bonne chose. Dans cette discussion, nous supposerons que nous vivons dans un monde idéal et les services principaux illustrés à droite de l’image du système au début de cet article sont tous des services REST HTTP idempotents, et à partir de là nous allons nous concentrer uniquement sur l’échange d’activités.

## <a name="buffering-outbound-activities"></a>Mise en mémoire tampon des activités sortantes

L’envoi d’une activité n’est pas une opération idempotente et il n’est pas clair qu’elle aurait beaucoup de sens dans un scénario de bout en bout. Après tout, l’activité consiste simplement à transmettre un message ajouté à une vue ou peut-être énoncé par un agent de synthèse vocale.

La chose principale que nous souhaitons éviter concernant l’envoi des activités est l’envoi multiple. Notre problème est que le mécanisme de verrouillage optimiste nécessite que nous puissions exécuter notre logique plusieurs fois. La solution est simple : nous devons mettre en mémoire tampon les activités sortantes à partir du dialogue jusqu’à ce que nous soyons certains de ne pas avoir à réexécuter la logique. Ceci jusqu’à réussir une opération d’enregistrement. Nous recherchons un flux qui ressemble à ce qui suit :

![diagramme de scale-out](../media/scale-out/scale-out-buffer.png)

En supposant que nous pouvons créer une boucle de nouvelle tentative autour de l’exécution du dialogue, nous obtenons le comportement suivant en cas d’un échec de condition préalable lors d’une opération de sauvegarde :

![diagramme de scale-out](../media/scale-out/scale-out-save.png)

En appliquant ce mécanisme et en modifiant notre exemple précédent, il est possible d’empêcher tout accusé de réception positif erroné en ce qui concerne l’ajout d’une garniture à une commande de pizza. En fait, bien que nous ayons augmenté la taille des instances de notre déploiement sur plusieurs machines, nous avons sérialisé efficacement nos mises à jour d’état avec le modèle de verrouillage optimiste. Lors de la commande de notre pizza, l’accusé de réception donné pour l’ajout d’un élément peut désormais être écrit pour refléter précisément l’état complet. Par exemple, si l’utilisateur tape immédiatement « fromage », puis « champignon » avant que le bot ait pu répondre, les deux réponses peuvent maintenant être « pizza avec fromage », puis « pizza avec fromage et champignon ».

En examinant le diagramme de séquence, nous pouvons voir que les réponses peuvent être perdues après une opération de sauvegarde réussie. Toutefois, elles peuvent être perdues n’importe où dans la communication de bout en bout. Le point est que ce n’est pas un problème pouvant être résolu par l’infrastructure de gestion d’état. Cela requiert un protocole de niveau supérieur et éventuellement un impliquant l’utilisateur du canal. Par exemple, si le bot semble ne pas avoir répondu aux yeux de l’utilisateur, il est raisonnable de penser que l’utilisateur fera un nouvel essai, ou un comportement similaire. Par conséquent, bien qu’il soit raisonnable pour un scénario d’avoir de telles pannes temporaires occasionnelles, il est beaucoup moins raisonnable de penser qu’un utilisateur puisse filtrer les accusés de réception positifs erronés ou les autres messages inattendus. 

En mettant tout cela ensemble, dans notre nouvelle solution de stockage personnalisée, nous allons faire trois choses que l’implémentation par défaut dans l’infrastructure ne fait pas. Tout d’abord, nous allons utiliser des ETags pour détecter les contentions, puis nous allons réessayer le traitement en cas de détection de l’échec de l’ETag et enfin, nous allons mettre en mémoire tampon toutes les activités sortantes jusqu’à réussir une sauvegarde. Le reste de cet article décrit l’implémentation de ces trois parties.

## <a name="implementing-etag-support"></a>Implémentation de la prise en charge de l’ETag

Pour prendre en charge les tests unitaires, nous commençons par définir une interface pour notre nouveau magasin avec la prise en charge de l’ETag. Disposer de cette interface signifie que nous pouvons écrire deux versions, une pour les tests unitaires et qui s’exécute en mémoire sans avoir besoin d’atteindre le réseau, et l’autre pour la production. L’interface facilite grandement l’exploitation des mécanismes d’injection de dépendance que nous possédons dans ASP.NET.

L’interface se compose des méthodes de chargement et d’enregistrement. Les deux utilisent la clé que nous allons utiliser pour l’état. Le chargement renvoie les données et l’ETag associée. L’enregistrement les recueille. En outre, l’enregistrement retourne la valeur booléenne. Cette valeur booléenne indique si l’ETag correspondait et si l’enregistrement a réussi. Cela n’est pas considéré comme un indicateur d’erreur général mais plutôt comme un indicateur spécifique d’un échec de condition préalable, nous modélisons cela comme un code de retour plutôt qu’une exception, car nous écrirons une logique de flux de contrôle autour de cela dans la forme de notre boucle de nouvelle tentative.

Comme nous aimerions que cette pièce de stockage du plus bas niveau soit enfichable, nous voulons être sûrs d’éviter le placement d’exigences de sérialisation dessus. Toutefois, nous aimerions spécifier que l’enregistrement du contenu doit être au format JSON, une implémentation de magasin peut ainsi définir l’en-tête Content-type. Le moyen le plus simple et le plus naturel pour faire ça dans .NET passe par les types d’arguments. Plus spécifiquement, nous taperons l’argument du contenu sous forme de JObject. En JavaScript ou TypeScript, ce sera simplement un objet natif normal.  

Voici l’interface résultante :

```csharp
public interface IStore
{
  Task<(JObject content, string eTag)> LoadAsync(string key);
  Task<bool> SaveAsync(string key, JObject content, string eTag);
}
```
Cette implémentation avec le stockage d’objets Blob Azure est très simple.
```csharp
public class BlobStore : IStore
{
  private CloudBlobContainer _container;

  public BlobStore(string myAccountName, string myAccountKey, string containerName)
  {
    var storageCredentials = new StorageCredentials(myAccountName, myAccountKey);
    var cloudStorageAccount = new CloudStorageAccount(storageCredentials, useHttps: true);
    var client = cloudStorageAccount.CreateCloudBlobClient();
    _container = client.GetContainerReference(containerName);
  }

  public async Task<(JObject content, string eTag)> LoadAsync(string key)
  {
    var blob = _container.GetBlockBlobReference(key);
    try
    {
      var content = await blob.DownloadTextAsync();
      var obj = JObject.Parse(content);
      var eTag = blob.Properties.ETag;
      return (obj, eTag);
    }
    catch (StorageException e)
      when (e.RequestInformation.HttpStatusCode ==
        (int)HttpStatusCode.NotFound)
    {
      return (new JObject(), null);
    }
  }

  public async Task<bool> SaveAsync(string key, JObject obj, string eTag)
  {
    var blob = _container.GetBlockBlobReference(key);
    blob.Properties.ContentType = "application/json";
    var content = obj.ToString();
    if (eTag != null)
    {
      try
      {
        await blob.UploadTextAsync(content,
          new AccessCondition { IfMatchETag = eTag },
          new BlobRequestOptions(),
          new OperationContext());
      }
      catch (StorageException e)
        when (e.RequestInformation.HttpStatusCode ==
          (int)HttpStatusCode.PreconditionFailed)
      {
        return false;
      }
    }
    else
    {
      await blob.UploadTextAsync(content);
    }
    return true;
  }
}
```

Comme vous pouvez le voir, le stockage d’objets Blob Azure effectue le véritable travail ici. Notez la capture d’exceptions spécifiques et la façon dont cela est traduit pour répondre à ce que seront les attentes du code appelant. Autrement dit, dans le chargement, nous voulons une exception Introuvable pour retourner la valeur null et l’exception d’échec de condition préalable sur l’enregistrement pour retourner une valeur booléenne.

Tout ce code source sera disponible dans un [exemple](https://aka.ms/scale-out) correspondant et cet exemple comprendra l’implémentation d’un magasin de mémoire.

## <a name="implementing-the-retry-loop"></a>Implémentation de la boucle de nouvelle tentative
La forme de base de la boucle dérive directement du comportement affiché dans les diagrammes de séquence.

À la réception d’une activité, nous créons une clé pour l’état correspondant pour cette conversation. Nous ne modifions pas la relation entre l’activité et l’état de la conversation, donc nous allons créer la clé dans la même façon que dans l’implémentation de l’état par défaut.

Une fois la clé appropriée créée, nous tenterons de charger l’état correspondant. Exécutez ensuite les dialogues du bot, puis essayez d’enregistrer. Si l’enregistrement réussit, nous enverrons les activités sortantes provenant de l’exécution du dialogue. Sinon, nous reviendrons en arrière et répéterons l’ensemble du processus précédant le chargement. Un nouveau chargement nous donne une nouvelle ETag, de sorte que le prochain enregistrement devrait réussir.

L’implémentation de OnTurn qui en résulte ressemble à ceci :
```csharp
public async Task OnTurnAsync(ITurnContext turnContext,
  CancellationToken cancellationToken = default(CancellationToken))
{
  // Create the storage key for this conversation.
  string key = $"{turnContext.Activity.ChannelId}/conversations/{turnContext.Activity.Conversation?.Id}";

  // The execution sits in a loop because there might be a retry if the save operation fails.
  while (true)
  {
    // Load any existing state associated with this key
    var (oldState, etag) = await _store.LoadAsync(key);

    // Run the dialog system with the old state and inbound activity,
    // resulting in a new state and outbound activities.
    var (activities, newState) = await DialogHost.RunAsync(_rootDialog, turnContext.Activity, oldState);

    // Save the updated state associated with this key.
    bool success = await _store.SaveAsync(key, newState, etag);

    // Following a successful save, send any outbound Activities, otherwise retry everything.
    if (success)
    {
      if (activities.Any())
      {
        // This is an actual send on the TurnContext we were given and so will actual do a send this time.
        await turnContext.SendActivitiesAsync(activities);
      }
      break;
    }
  }
}
```
Notez que nous avons modélisé l’exécution du dialogue en un appel de fonction. Une implémentation plus sophistiquée aurait défini une interface et aurait permis l’injection de dépendances, mais dans notre cas, le fait d’avoir le dialogue derrière une fonction statique met l’accent sur la nature fonctionnelle de notre approche. En tant qu’instruction générale, l’organisation de notre implémentation de sorte que les parties cruciales deviennent fonctionnelles nous place très bien lorsqu’il s’agit de le faire fonctionner correctement sur les réseaux.


## <a name="implementing-outbound-activity-buffering"></a>Implémentation de la mise en tampon des activités sortantes 

La condition suivante est que nous mettions en mémoire tampon les activités sortantes jusqu’à obtenir un enregistrement terminé avec succès. Cela nécessite l’implémentation personnalisée de BotAdapter. Dans ce code, nous implémenterons la fonction SendActivity abstraite pour ajouter l’activité à une liste au lieu de l’envoyer. Le dialogue que nous allons héberger sera non-the-wiser.
Dans ce scénario particulier, les opérations UpdateActivity et DeleteActivity ne sont pas prises en charge et elles lèveront donc la valeur Pas implémenté à partir de ces méthodes. En outre, nous ne nous soucions pas de la valeur renvoyée à partir de SendActivity. Elle est utilisée par certains canaux dans des scénarios où les mises à jour des activités doivent être envoyées, par exemple, pour désactiver les boutons sur les cartes affichées dans le canal. Ces échanges de message peuvent devenir complexes, en particulier lorsque l’état est obligatoire, ce qui est hors de portée de cet article. L’implémentation complète du BotAdapter personnalisé ressemble à ceci :

```csharp
public class DialogHostAdapter : BotAdapter
{
  private List<Activity> _response = new List<Activity>();

  public IEnumerable<Activity> Activities => _response;

  public override Task<ResourceResponse[]> SendActivitiesAsync(ITurnContext turnContext,
    Activity[] activities, CancellationToken cancellationToken)
  {
    foreach (var activity in activities)
    {
      _response.Add(activity);
    }
    return Task.FromResult(new ResourceResponse[0]);
  }

  public override Task DeleteActivityAsync(ITurnContext turnContext,
    ConversationReference reference, CancellationToken cancellationToken)
  {
    throw new NotImplementedException();
  }

  public override Task<ResourceResponse> UpdateActivityAsync(ITurnContext turnContext,
    Activity activity, CancellationToken cancellationToken)
  {
    throw new NotImplementedException();
  }
}
```
Tout ce qu’il reste à faire est de coller ensemble ces divers nouveaux éléments et de les incorporer dans les éléments de l’infrastructure existante. La principale boucle de nouvelle tentative se trouve uniquement dans la fonction IBot OnTurn. Elle contient notre implémentation IStore personnalisée pour laquelle nous avons autorisé l’injection de dépendances à des fins de test. Nous avons mis tout le code d’hébergement du dialogue dans une classe nommée DialogHost exposant une seule fonction statique publique. Cette fonction est définie pour prendre l’activité entrante et l’ancien état et pour retourner les activités qui en résultent et un nouvel état.

La première chose à faire dans cette fonction consiste à créer le BotAdapter personnalisé que nous avons introduit plus tôt. Puis nous allons simplement exécuter le dialogue comme nous le faisons généralement en créant un DialogSet et un DialogContext et en effectuant les flux Continuer ou Commencer habituels. La seule information que nous n’avons pas abordée est le besoin d’un accesseur personnalisé. Cela s’avère être un shim très simple qui facilite le passage de l’état du dialogue dans le système de dialogue. L’accesseur utilise une sémantique de référence lors d’un fonctionnement avec le système de dialogue et le passage du descripteur est la seule chose nécessaire. Pour rendre les choses plus claires, nous avons restreint le modèle de classe que nous utilisons à la sémantique de référence.

Nous sommes très prudents dans l’organisation en couches, nous plaçons le JsonSerialization inclus dans notre code d’hébergement, car nous n’en voulons pas à l’intérieur de la couche de stockage enfichable lorsque différentes implémentations peuvent sérialiser différemment.

Voici le code du pilote :
```csharp
public class DialogHost
{
  private static readonly JsonSerializer StateJsonSerializer = new JsonSerializer()
    { TypeNameHandling = TypeNameHandling.All };

  public static async Task<Tuple<Activity[], JObject>> RunAsync(Dialog rootDialog,
    Activity activity, JObject oldState)
  {
    // A custom adapter and corresponding TurnContext that buffers any messages sent.
    var adapter = new DialogHostAdapter();
    var turnContext = new TurnContext(adapter, activity);

    // Run the dialog using this TurnContext with the existing state.
    JObject newState = await RunTurnAsync(rootDialog, turnContext, oldState);

    // The result is a set of activities to send and a replacement state.
    return Tuple.Create(adapter.Activities.ToArray(), newState);
  }

  private static async Task<JObject> RunTurnAsync(Dialog rootDialog,
    TurnContext turnContext, JObject state)
  {
    if (turnContext.Activity.Type == ActivityTypes.Message)
    {
      // If we have some state, deserialize it. (This mimics the shape produced by BotState.cs.)
      var dialogState = state?[nameof(DialogState)]?.ToObject<DialogState>(StateJsonSerializer);

      // A custom accessor is used to pass a handle on the state to the dialog system.
      var accessor = new RefAccessor<DialogState>(dialogState);

      // The following is regular dialog driver code.
      var dialogs = new DialogSet(accessor);
      dialogs.Add(rootDialog);

      var dialogContext = await dialogs.CreateContextAsync(turnContext);
      var results = await dialogContext.ContinueDialogAsync();

      if (results.Status == DialogTurnStatus.Empty)
      {
        await dialogContext.BeginDialogAsync("root");
      }

      // Serialize the result, and put its value back into a new JObject.
      return new JObject
      {
        { nameof(DialogState), JObject.FromObject(accessor.Value, StateJsonSerializer) }
      };
    }

    return state;
  }
}
```
Et enfin, l’accesseur personnalisé, il nous suffit d’implémenter l’ensemble, car l’état est par référence :
```csharp
public class RefAccessor<T> : IStatePropertyAccessor<T> where T : class
{
  public RefAccessor(T value)
  {
    Value = value;
  }

  public T Value { get; private set; }

  public string Name => nameof(T);

  public Task<T> GetAsync(ITurnContext turnContext, Func<T> defaultValueFactory = null,
    CancellationToken cancellationToken = default(CancellationToken))
  {
    if (Value == null)
    {
      if (defaultValueFactory == null)
      {
        throw new KeyNotFoundException();
      }
      else
      {
        Value = defaultValueFactory();
      }
    }
    return Task.FromResult(Value);
  }

  public Task DeleteAsync(ITurnContext turnContext,
    CancellationToken cancellationToken = default(CancellationToken))
  {
    throw new NotImplementedException();
  }

  public Task SetAsync(ITurnContext turnContext, T value,
    CancellationToken cancellationToken = default(CancellationToken))
  {
    throw new NotImplementedException();
  }
}
```

## <a name="additional-resources"></a>Ressources supplémentaires
Le code source [C#](http://aka.ms/scale-out) utilisé dans cet article est disponible sur GitHub.

