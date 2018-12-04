---
title: Utiliser plusieurs modèles LUIS et QnA | Microsoft Docs
description: Découvrez comment utiliser LUIS et QnA Maker dans votre bot.
keywords: LUIS, QnA, outil Dispatch, services multiples, intentions de route
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 11/26/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 9b0ddf5cf8af61048ba78f10824c9573da82fc08
ms.sourcegitcommit: a722f960cd0a8513d46062439eb04de3a0275346
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/27/2018
ms.locfileid: "52336269"
---
# <a name="use-multiple-luis-and-qna-models"></a>Utiliser plusieurs modèles LUIS et QnA

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Dans ce tutoriel, nous vous montrons comment utiliser le service Dispatch pour router des énoncés quand un bot prend en charge plusieurs modèles LUIS et services QnA Maker pour différents scénarios. Dans ce cas, nous configurons Dispatch avec plusieurs modèles LUIS pour des conversations portant sur la domotique et la météo, plus le service QnA Maker pour répondre aux questions basées sur un fichier texte FAQ comme entrée. Cet exemple combine les services suivants.

| Nom | Description |
|------|------|
| HomeAutomation | Application LUIS qui reconnaît une intention domotique avec les données d’entité associées.|
| Météo | Application LUIS qui reconnaît les intentions `Weather.GetForecast` et `Weather.GetCondition` avec des données de localisation.|
| Forum Aux Questions  | Base de connaissances QnA Maker qui fournit des réponses à certaines questions simples concernant le bot. |

## <a name="prerequisites"></a>Prérequis

- Le code de cet article est basé sur l’exemple **NLP with Dispatch**. Vous aurez besoin d’une copie de l’exemple en [C# ](https://aka.ms/dispatch-sample-cs) ou en [JS](https://aka.ms/dispatch-sample-js).
- Connaissances des [concepts de base des bots](bot-builder-basics.md), du [traitement en langage naturel](bot-builder-howto-v4-luis.md), de [QnA Maker](bot-builder-howto-qna.md) et du fichier [.bot](bot-file-basics.md).
- [Bot Framework Emulator](https://github.com/Microsoft/BotFramework-Emulator/blob/master/README.md#download) à des fins de test.

## <a name="create-the-services-and-test-the-bot"></a>Créer les services et tester le bot

Suivez les instructions du fichier **README** pour [C#](https://github.com/Microsoft/BotBuilder-Samples/blob/master/samples/csharp_dotnetcore/14.nlp-with-dispatch/README.md) ou [JS](https://github.com/Microsoft/BotBuilder-Samples/blob/master/samples/javascript_nodejs/14.nlp-with-dispatch/README.md) pour générer et exécuter l’exemple à l’aide de l’émulateur. 

Pour référence, voici quelques-unes des questions et des commandes qui sont couvertes par les services que nous avons inclus :

* QnA Maker
  * `hi`, `good morning`
  * `what are you`, `what do you do`
* LUIS (domotique)
  * `turn on bedroom light`
  * `turn off bedroom light`
  * `make some coffee`
* LUIS (météo)
  * `whats the weather in redmond washington`
  * `what's the forecast for london`
  * `show me the forecast for nebraska`

### <a name="connecting-to-the-services-from-your-bot"></a>Connexion aux services à partir de votre bot

Pour vous connecter aux services Dispatch, LUIS et QnA Maker, votre bot extrait les informations du fichier **.bot**.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Dans **Startup.cs**, `ConfigureServices` lit le fichier de configuration, et `InitBotServices` utilise ces informations pour initialiser les services. Chaque fois qu’il est créé, le bot est initialisé avec l’objet `BotServices` inscrit. Voici les parties pertinentes de ces deux méthodes.

```csharp
public void ConfigureServices(IServiceCollection services)
{
    //...
    var botConfig = BotConfiguration.Load(botFilePath ?? @".\nlp-with-dispatch.bot", secretKey);
    services.AddSingleton(sp => botConfig
        ?? throw new InvalidOperationException($"The .bot config file could not be loaded. ({botConfig})"));

    // ...
    
    var connectedServices = InitBotServices(botConfig);
    services.AddSingleton(sp => connectedServices);
    
    services.AddBot<NlpDispatchBot>(options =>
    {
          
          // The Memory Storage used here is for local bot debugging only. 
          // When the bot is restarted, everything stored in memory will be gone.

          Storage dataStore = new MemoryStorage();

          // ...

          // Create Conversation State object.
          // The Conversation State object is where we persist anything at the conversation-scope.

          var conversationState = new ConversationState(dataStore);
          options.State.Add(conversationState);
     });
}

```
Le code suivant initialise les références du bot aux services externes. Par exemple, les services LUIS et QnA Maker sont créés ici. Ces services externes sont configurés à l’aide de la classe `BotConfiguration`, en fonction du contenu de votre fichier « .bot ».

```csharp
private static BotServices InitBotServices(BotConfiguration config)
{
    var qnaServices = new Dictionary<string, QnAMaker>();
    var luisServices = new Dictionary<string, LuisRecognizer>();

    foreach (var service in config.Services)
    {
        switch (service.Type)
        {
            case ServiceTypes.Luis:
                {
                    // ...
                    var app = new LuisApplication(luis.AppId, luis.AuthoringKey, luis.GetEndpoint());
                    var recognizer = new LuisRecognizer(app);
                    luisServices.Add(luis.Name, recognizer);
                    break;
                }

            case ServiceTypes.Dispatch:
                // ...
                var dispatchApp = new LuisApplication(dispatch.AppId, dispatch.AuthoringKey, dispatch.GetEndpoint());

                // Since the Dispatch tool generates a LUIS model, we use the LuisRecognizer to resolve the
                // dispatching of the incoming utterance.
                var dispatchARecognizer = new LuisRecognizer(dispatchApp);
                luisServices.Add(dispatch.Name, dispatchARecognizer);
                break;

            case ServiceTypes.QnA:
                {
                    // ...
                    var qnaEndpoint = new QnAMakerEndpoint()
                    {
                        KnowledgeBaseId = qna.KbId,
                        EndpointKey = qna.EndpointKey,
                        Host = qna.Hostname,
                    };

                    var qnaMaker = new QnAMaker(qnaEndpoint);
                    qnaServices.Add(qna.Name, qnaMaker);
                    break;
                }
        }
    }

    return new BotServices(qnaServices, luisServices);
}
```

<!--
# [JavaScript](#tab/javascript)

```javascript
```
-->

---

### <a name="calling-the-services-from-your-bot"></a>Appel des services à partir de votre bot

La logique du bot compare l’entrée de l’utilisateur au modèle Dispatch combiné.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Dans le fichier **NlpDispatchBot.cs**, le constructeur du bot obtient l’objet `BotServices` que nous avons inscrit au démarrage.

```csharp
private readonly BotServices _services;

public NlpDispatchBot(BotServices services)
{
    _services = services ?? throw new System.ArgumentNullException(nameof(services));

    //...
}
```

Dans la méthode `OnTurnAsync` du bot, nous comparons les messages entrants de l’utilisateur au modèle Dispatch.

```csharp
// Get the intent recognition result
var recognizerResult = await _services.LuisServices[DispatchKey].RecognizeAsync(context, cancellationToken);
var topIntent = recognizerResult?.GetTopScoringIntent();

if (topIntent == null)
{
    await context.SendActivityAsync("Unable to get the top intent.");
}
else
{
    await DispatchToTopIntentAsync(context, topIntent, cancellationToken);
}
```

<!--
# [JavaScript](#tab/javascript)

```javascript
```
-->

---

### <a name="working-with-the-recognition-results"></a>Utilisation des résultats de reconnaissance

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Lorsque le modèle renvoie un résultat, il indique quel service est le mieux à même de gérer l’énoncé. Le code de ce bot achemine la demande vers le service correspondant, puis il résume la réponse du service appelé.

```csharp
// Depending on the intent from Dispatch, routes to the right LUIS model or QnA service.
private async Task DispatchToTopIntentAsync(
    ITurnContext context,
    (string intent, double score)? topIntent,
    CancellationToken cancellationToken = default(CancellationToken))
{
    const string homeAutomationDispatchKey = "l_Home_Automation";
    const string weatherDispatchKey = "l_Weather";
    const string noneDispatchKey = "None";
    const string qnaDispatchKey = "q_sample-qna";

    switch (topIntent.Value.intent)
    {
        case homeAutomationDispatchKey:
            await DispatchToLuisModelAsync(context, HomeAutomationLuisKey);

            // Here, you can add code for calling the hypothetical home automation service, passing in any entity
            // information that you need.
            break;
        case weatherDispatchKey:
            await DispatchToLuisModelAsync(context, WeatherLuisKey);

            // Here, you can add code for calling the hypothetical weather service,
            // passing in any entity information that you need
            break;
        case noneDispatchKey:
            // You can provide logic here to handle the known None intent (none of the above).
            // In this example we fall through to the QnA intent.
        case qnaDispatchKey:
            await DispatchToQnAMakerAsync(context, QnAMakerKey);
            break;

        default:
            // The intent didn't match any case, so just display the recognition results.
            await context.SendActivityAsync($"Dispatch intent: {topIntent.Value.intent} ({topIntent.Value.score}).");
            break;
    }
}

// Dispatches the turn to the request QnAMaker app.
private async Task DispatchToQnAMakerAsync(
    ITurnContext context,
    string appName,
    CancellationToken cancellationToken = default(CancellationToken))
{
    if (!string.IsNullOrEmpty(context.Activity.Text))
    {
        var results = await _services.QnAServices[appName].GetAnswersAsync(context).ConfigureAwait(false);
        if (results.Any())
        {
            await context.SendActivityAsync(results.First().Answer, cancellationToken: cancellationToken);
        }
        else
        {
            await context.SendActivityAsync($"Couldn't find an answer in the {appName}.");
        }
    }
}


// Dispatches the turn to the requested LUIS model.
private async Task DispatchToLuisModelAsync(
    ITurnContext context,
    string appName,
    CancellationToken cancellationToken = default(CancellationToken))
{
    await context.SendActivityAsync($"Sending your request to the {appName} system ...");
    var result = await _services.LuisServices[appName].RecognizeAsync(context, cancellationToken);

    await context.SendActivityAsync($"Intents detected by the {appName} app:\n\n{string.Join("\n\n", result.Intents)}");

    if (result.Entities.Count > 0)
    {
        await context.SendActivityAsync($"The following entities were found in the message:\n\n{string.Join("\n\n", result.Entities)}");
    }
}
```

<!--
# [JavaScript](#tab/javascript)

```javascript
```
-->

---

## <a name="evaluate-the-dispatchers-performance"></a>Évaluer les performances de l’application de distribution

Il y a parfois des messages utilisateur qui sont fournis comme exemples dans les applications LUIS et les services QnA Maker ; l’application LUIS combinée que génère l’outil Dispatch ne traite pas ces entrées correctement. Vous pouvez vérifier les performances de votre application à l’aide de l’option `eval`.

```shell
dispatch eval
```

L’exécution de `dispatch eval` génère un fichier **Summary.html** qui fournit des statistiques sur les performances prédites du modèle de langage. Vous pouvez exécuter `dispatch eval` sur n’importe quelle application LUIS, pas seulement sur les applications LUIS créées par l’outil Dispatch.

### <a name="edit-intents-for-duplicates-and-overlaps"></a>Modifier les intentions en cas de doublons et de redondances

Passez en revue les exemples d’énoncés marqués comme étant des doublons dans **Summary.html** et supprimez les exemples similaires ou redondants. Par exemple, supposons que dans l’application LUIS `Home Automation`, des demandes comme « allumer la lumière » correspondent à une intention « TurnOnLights » (Allumer la lumière), mais que des demandes comme « Pourquoi la lumière ne s’allume-t-elle pas ? » correspondent à une intention « None » (Aucune) afin qu’elles puissent être transmises à QnA Maker. Quand vous combinez l’application LUIS et le service QnA Maker à l’aide de l’outil Dispatch, vous devez effectuer l’une des opérations suivantes :

* Supprimez l’intention « None » (Aucune) de l’application LUIS `Home Automation` d’origine et ajoutez les énoncés dans cette intention à l’intention « None » (Aucune) dans l’application de distribution.
* Si vous ne supprimez pas l’intention « None » (Aucune) de l’application LUIS d’origine, vous devez ajouter de la logique à votre bot pour transmettre les messages qui correspondent à cette intention au service QnA Maker.


## <a name="additional-resources"></a>Ressources supplémentaires 

**Supprimer des ressources** : cet exemple crée un certain nombre d’applications et de ressources que vous pouvez supprimer à l’aide de la procédure ci-dessous. Veillez toutefois à ne pas supprimer les ressources dont dépendent *d’autres applications ou services*. 

_Ressources LUIS_
1. Connectez-vous au portail [luis.ai](https://www.luis.ai).
1. Accédez à la page _My Apps_ (Mes applications).
1. Sélectionnez les applications créées par cet exemple.
   * `Home Automation`
   * `Weather`
   * `NLP-With-Dispatch-BotDispatch`
1. Cliquez sur _Delete_ (Supprimer), puis sur _Ok_ pour confirmer.

_Ressources QnA Maker_
1. Connectez-vous au portail [qnamaker.ai](https://www.qnamaker.ai/).
1. Accédez à la page _My knowledge bases_ (Mes bases de connaissances).
1. Cliquez sur le bouton de suppression de la base de connaissances `Sample QnA`, puis cliquez sur _Delete_ (Supprimer) pour confirmer.

**Bonnes pratiques** : pour améliorer les services utilisés dans cet exemple, reportez-vous aux bonnes pratiques pour [LUIS](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/luis-concept-best-practices) et [QnA Maker](https://docs.microsoft.com/en-us/azure/cognitive-services/qnamaker/concepts/best-practices).
