---
title: Enregistrer l’état à l’aide des propriétés de la conversation et de l’utilisateur | Microsoft Docs
description: Découvrez comment enregistrer et récupérer des données avec la version 4 du Kit SDK Bot Builder pour .NET.
keywords: état de la conversation, état utilisateur, intergiciel d’état, flux de la conversation, stockage de fichiers, stockage de table azure
author: ivorb
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 05/03/18
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 16df371b1cabb4b3eb47d1f491a5d45e26627d38
ms.sourcegitcommit: dcbc8ad992a3e242a11ebcdf0ee99714d919a877
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/30/2018
ms.locfileid: "39352848"
---
# <a name="save-state-using-conversation-and-user-properties"></a>Enregistrer l’état à l’aide des propriétés de la conversation et de l’utilisateur

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Pour permettre à votre bot d’enregistrer l’état de la conversation et l’état utilisateur, initialisez d’abord l’intergiciel (middleware) du gestionnaire d’état, puis utilisez les propriétés d’état de la conversation et utilisateur.
Pour plus d’informations sur l’utilisation de cet état, consultez [État et stockage](./bot-builder-storage-concept.md).

## <a name="initialize-state-manager-middleware"></a>Initialiser l’intergiciel (middleware) du gestionnaire d’état

Dans le Kit SDK, vous devez initialiser l’adaptateur du bot pour utiliser l’intergiciel (middleware) du gestionnaire d’état avant de pouvoir utiliser les magasins de propriétés de conversation ou d’utilisateur. _État de la conversation_ est utilisé pour les propriétés de conversation, et _État utilisateur_ est utilisé pour les propriétés de l’utilisateur. (Les propriétés de l’état utilisateur sont accessibles dans plusieurs conversations.) L’intergiciel du gestionnaire d’état fournit une abstraction qui vous permet d’accéder aux propriétés à l’aide d’un magasin de clé-valeur simple ou d’objet, indépendamment du type de stockage sous-jacent. Le gestionnaire d’état s’occupe de l’écriture des données dans le stockage et de la gestion de la concurrence, quel que soit le type de stockage sous-jacent (en mémoire, fichier ou table Azure).


# <a name="ctabcsharp"></a>[C#](#tab/csharp)
Pour voir comment `ConversationState` est initialisé, consultez `Startup.cs` dans l’exemple Microsoft.Bot.Samples.EchoBot-AspNetCore.

```csharp
services.AddBot<EchoBot>(options =>
{
    options.CredentialProvider = new ConfigurationCredentialProvider(Configuration);

    IStorage dataStore = new MemoryStorage();
    options.Middleware.Add(new ConversationState<EchoState>(dataStore));
});
```

Dans la ligne `options.Middleware.Add(new ConversationState<EchoState>(dataStore));`, `ConversationState` est l’objet du gestionnaire d’état de la conversation, qui est ajouté au bot comme un intergiciel. Le paramètre de type `EchoState` est le type qui représente la façon dont vous souhaitez stocker les informations sur l’état de la conversation. Un bot peut utiliser n’importe quel type de classe pour les données de la conversation ou d’état utilisateur.

L’implémentation de `EchoState` se trouve dans `EchoBot.cs` :
```csharp
public class EchoState
{
    public int TurnNumber { get; set; }
}
``` 

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

Définissez le fournisseur de stockage et assignez-le au gestionnaire d’état que vous souhaitez utiliser.
Vous pouvez utiliser le même fournisseur de stockage pour les intergiciels de gestion `ConversationState` et `UserState`.
Vous utilisez ensuite la bibliothèque `BotStateSet` pour la connecter à la couche d’intergiciel qui gérera la persistance des données à votre place.

```javascript
const { BotFrameworkAdapter, MemoryStorage, ConversationState, UserState, BotStateSet } = require('botbuilder');
const restify = require('restify');

// Create adapter
const adapter = new BotFrameworkAdapter({ 
    appId: process.env.MICROSOFT_APP_ID, 
    appPassword: process.env.MICROSOFT_APP_PASSWORD 
});

// Add conversation state middleware.
const conversationState = new ConversationState(new MemoryStorage());
adapter.use(conversationState);

// Alternatively, use both conversation and user state middleware.
// const storage = new MemoryStorage;
// const conversationState = new ConverstationState(storage);
// const userState = new UserState(storage);
// adapter.use(new BotStateSet(conversationState, userState));
```    
---

> [!NOTE] 
> Le stockage de données en mémoire est uniquement destiné aux tests. Ce stockage est volatile et temporaire. Les données sont effacées chaque fois que le bot est redémarré. Consultez les sections [Stockage de fichiers](#file-storage) et [Stockage de table Azure](#azure-table-storage) plus loin dans cet article pour configurer d’autres supports de stockage sous-jacents pour l’état de la conversation et l’état utilisateur. 

### <a name="configuring-state-manager-middleware"></a>Configuration de l’intergiciel (middleware) du gestionnaire d’état

Lors de l’initialisation de l’intergiciel (middleware) d’état, une option _paramètres d’état_ vous permet de modifier le comportement par défaut de l’enregistrement des propriétés. Les paramètres par défaut sont :

* Conserver les propriétés au-delà de la durée du contexte de tour.
* Si une propriété contient plusieurs instances du bot, autoriser la dernière instance du bot à remplacer la précédente.

## <a name="use-conversation-and-user-state-properties"></a>Utiliser les propriétés d’état de la conversation et d’état utilisateur 
<!-- middleware and message context properties -->

Une fois l’intergiciel du gestionnaire d’état configuré, vous pouvez obtenir les propriétés d’état de la conversation et d’état utilisateur à partir de l’objet de contexte.
<!-- Changes are written to storage before the `SendActivity()` pipeline completes. -->

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Vous pouvez voir comment cela fonctionne à l’aide de l’exemple `Microsoft.Bot.Samples.EchoBot` dans le kit SDK Bot Builder. 

Dans le gestionnaire `OnTurn`, `context.GetConversationState` obtient l’état de la conversation pour accéder aux données que vous avez définies, et vous pouvez modifier cet état à l’aide des propriétés (dans ce cas, en incrémentant `TurnNumber`).

```csharp
public async Task OnTurn(ITurnContext context)
{
    // This bot is only handling Messages
    if (context.Activity.Type == ActivityTypes.Message)
    {
        // Get the conversation state from the turn context
        var state = context.GetConversationState<EchoState>();

        // Bump the turn count. 
        state.TurnCount++;

        // Echo back to the user whatever they typed.
        await context.SendActivity($"Turn {state.TurnCount}: You sent '{context.Activity.Text}'");
    }
}
```   

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

Vous pouvez voir comment cela fonctionne à l’aide de l’EchoBot généré à partir de l’exemple de générateur Yeoman.

Cet exemple de code montre comment stocker un compteur de tours dans l’état de la conversation.

```javascript
const { BotFrameworkAdapter, MemoryStorage, ConversationState } = require('botbuilder');
const restify = require('restify');

// Create server
let server = restify.createServer();
server.listen(process.env.port || process.env.PORT || 3978, function () {
    console.log(`${server.name} listening to ${server.url}`);
});

// Create adapter
const adapter = new BotFrameworkAdapter({ 
    appId: process.env.MICROSOFT_APP_ID, 
    appPassword: process.env.MICROSOFT_APP_PASSWORD 
});

// Add conversation state middleware
const conversationState = new ConversationState(new MemoryStorage());
adapter.use(conversationState);

// Listen for incoming requests 
server.post('/api/messages', (req, res) => {
    // Route received request to adapter for processing
    adapter.processActivity(req, res, async (context) => {
        if (context.activity.type === 'message') {
            const state = conversationState.get(context);
            const count = state.count === undefined ? state.count = 0 : ++state.count;
            await context.sendActivity(`${count}: You said "${context.activity.text}"`);
        } else {
            await context.sendActivity(`[${context.activity.type} event detected]`);
        }
    });
});
```
---

## <a name="using-conversation-state-to-direct-conversation-flow"></a>Utilisation de l’état de la conversation pour diriger le flux de la conversation

Lorsque vous concevez un flux de conversation, il est utile de définir un indicateur d’état afin de diriger le flux de la conversation. L’indicateur peut être un simple type **booléen** ou un type incluant le nom de la rubrique active. L’indicateur peut vous aider à vous situer dans une conversation. Par exemple, un indicateur de type **booléen** peut vous indiquer si vous participez ou non à une conversation. Une propriété de nom de rubrique peut vous indiquer à quelle conversation vous participez.

L’exemple suivant utilise une propriété booléenne _have asked name_ pour signaler quand le bot a demandé à l’utilisateur son nom. Lorsque le message suivant est reçu, le bot vérifie la propriété. Si la propriété est définie sur `true`, le bot sait que l’utilisateur a été invité à fournir son nom et interprète le message entrant comme un nom à enregistrer sous la forme d’une propriété d’utilisateur.


# <a name="ctabcsharp"></a>[C#](#tab/csharp)
```csharp
public class ConversationInfo
{
    public bool haveAskedNameFlag { get; set; }
    public bool haveAskedNumberFlag { get; set; }
}

public class UserInfo
{
    public string name { get; set; }
    public string telephoneNumber { get; set; }
    public bool done { get; set; }
}

public async Task OnTurn(ITurnContext context)
{
    // Get state objects. Default objects are created if they don't already exist.
    var convo = ConversationState<ConversationInfo>.Get(context);
    var user = UserState<UserInfo>.Get(context);

    if (context.Activity.Type is ActivityTypes.Message)
    {
        if (string.IsNullOrEmpty(user.name) && !convo.haveAskedNameFlag)
        {
            // Ask for the name.
            await context.SendActivity("Hello. What's your name?");

            // Set flag to show we've asked for the name. We save this out so the
            // context object for the next turn of the conversation can check haveAskedName
            convo.haveAskedNameFlag = true;
        }
        else if (!convo.haveAskedNumberFlag)
        {
            // Save the name.
            var name = context.Activity.AsMessageActivity().Text;
            user.name = name;
            convo.haveAskedNameFlag = false; // Reset flag

            // Ask for the phone number. You might want a flag to track this, too.
            await context.SendActivity($"Hello, {name}. What's your telephone number?");
            convo.haveAskedNumberFlag = true;
        }
        else if (convo.haveAskedNumberFlag)
        {
            // save the telephone number
            var telephonenumber = context.Activity.AsMessageActivity().Text;

            user.telephoneNumber = telephonenumber;
            convo.haveAskedNumberFlag = false; // Reset flag
            await context.SendActivity($"Got it. I'll call you later.");
        }
    }
}
```

Pour configurer l’état utilisateur afin de le retourner par `UserState<UserInfo>.Get(context)`, vous ajoutez l’intergiciel de l’état utilisateur. Par exemple, dans la section `Startup.cs` d’ASP .NET Core EchoBot, modifiez le code dans ConfigureServices.cs :

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddBot<EchoBot>(options =>
    {
        options.CredentialProvider = new ConfigurationCredentialProvider(Configuration);
        
        IStorage dataStore = new MemoryStorage();
        options.Middleware.Add(new ConversationState<ConversationInfo>(dataStore));
        options.Middleware.Add(new UserState<UserInfo>(dataStore));
    });
}
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

**app.js**

```js
const { BotFrameworkAdapter, MemoryStorage, ConversationState, UserState, BotStateSet } = require('botbuilder');
const restify = require('restify');

// Create server
let server = restify.createServer();
server.listen(process.env.port || process.env.PORT || 3978, function () {
    console.log(`${server.name} listening to ${server.url}`);
});

// Create adapter (it's ok for MICROSOFT_APP_ID and MICROSOFT_APP_PASSWORD to be blank for now)  
const adapter = new BotFrameworkAdapter({ 
    appId: process.env.MICROSOFT_APP_ID, 
    appPassword: process.env.MICROSOFT_APP_PASSWORD 
});

// Storage
const storage = new MemoryStorage();
const conversationState = new ConversationState(storage);
const userState  = new UserState(storage);
adapter.use(new BotStateSet(conversationState, userState));

// Listen for incoming requests 
server.post('/api/messages', (req, res) => {
    // Route received request to adapter for processing
    adapter.processActivity(req, res, async (context) => {
        const isMessage = (context.activity.type === 'message');
        const convo = conversationState.get(context);
        const user = userState.get(context);

        if (isMessage) {
            if(!user.name && !convo.haveAskedNameFlag){
                // Ask for the name.
                await context.sendActivity("What is your name?")
                // Set flag to show we've asked for the name. We save this out so the
                // context object for the next turn of the conversation can check haveAskedNameFlag
                convo.haveAskedNameFlag = true;
            } else if(convo.haveAskedNameFlag){
                // Save the name.
                user.name = context.activity.text;
                convo.haveAskedNameFlag = false; // Reset flag

                await context.sendActivity(`Hello, ${user.name}. What's your telephone number?`);
                convo.haveAskedNumberFlag = true; // Set flag
            } else if(convo.haveAskedNumberFlag){
                // save the phone number
                user.telephonenumber = context.activity.text;
                convo.haveAskedNumberFlag = false; // Reset flag
                await context.sendActivity(`Got it. I'll call you later.`);
            }
        }

        // ...
    });
});

```

---

Une alternative consiste à utiliser le modèle _en cascade_ d’un dialogue. Le dialogue suit l’état de la conversation pour vous, ce qui vous évite d’avoir à créer des indicateurs pour suivre votre état. Pour plus d’informations, voir [Gérer une conversation à l’aide de dialogues](bot-builder-dialog-manage-conversation-flow.md).

## <a name="file-storage"></a>Stockage Fichier

Le fournisseur de stockage de mémoire utilise le stockage en mémoire qui est détruit lorsque le bot redémarre. Il est utile uniquement à des fins de test. Si vous voulez conserver les données mais que vous ne souhaitez pas restreindre votre bot à une base de données, vous pouvez utiliser le fournisseur de stockage de fichiers. Même si ce fournisseur peut également servir à des fins de test, il conserve les données d’état dans un fichier pour vous permettre de l’inspecter. Les données sont inscrites au format JSON dans le fichier.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Accédez à `Startup.cs` dans l’exemple Microsoft.Bot.Samples.EchoBot-AspNetCore, puis modifiez le code dans la méthode `ConfigureServices`.
```csharp
// This method gets called by the runtime. Use this method to add services to the container.
public void ConfigureServices(IServiceCollection services)
{
    services.AddBot<EchoBot>(options =>
    {
        options.CredentialProvider = new ConfigurationCredentialProvider(Configuration);

        // Using file storage instead of in-memory storage.
        IStorage dataStore = new FileStorage(System.IO.Path.GetTempPath());
        options.Middleware.Add(new ConversationState<EchoState>(dataStore));
    });
}
``` 

Exécutez le code, puis laissez echobot retourner votre entrée plusieurs fois.

Accédez ensuite au répertoire spécifié par `System.IO.Path.GetTempPath()`. Vous devriez voir un fichier dont le nom commence par « conversation ». Ouvrez-le et examinez le code JSON. Le contenu du fichier ressemble à ceci :
```json
{
  "$type": "Microsoft.Bot.Samples.Echo.EchoState, Microsoft.Bot.Samples.EchoBot",
  "TurnNumber": "3",
  "eTag": "ecfe2a23566b4b52b2fe697cffc59385"
}
```

`$type` spécifie le type de la structure de données que vous utilisez dans votre bot pour stocker l’état de la conversation. Le champ `TurnNumber` correspond à la propriété `TurnNumber` dans la classe `EchoState`. Le champ `eTag` est hérité de `IStoreItem` et représente une valeur unique automatiquement actualisée chaque fois que votre bot met à jour l’état de la conversation.  Le champ eTag permet à votre bot d’activer l’accès concurrentiel optimiste.

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

Pour utiliser `FileStorage`, mettez à jour votre exemple de bot d’écho décrit dans la section : [Utiliser les propriétés d’état de la conversation et d’état utilisateur](#use-conversation-and-user-state-properties). Assurez-vous que `storage` est défini sur `FileStorage` au lieu de `MemoryStorage` et demande `FileStorage` à botbuilder. Ce sont les seules modifications nécessaires. 

```javascript
// Storage
const storage = new FileStorage("c:/temp");
const conversationState = new ConversationState(storage);
const userState  = new UserState(storage);
adapter.use(new BotStateSet(conversationState, userState));
```

Le fournisseur `FileStorage` utilise un « chemin d'accès » comme paramètre. Spécifier un chemin d’accès vous permet de trouver facilement le fichier contenant les informations persistantes à partir de votre bot. Un nouveau fichier sera créé pour chaque *conversation*. Ainsi, le *chemin d’accès* peut comporter plusieurs noms de fichiers commençant par `conversation!`. Vous pouvez les trier par date pour trouver facilement la dernière conversation. En revanche, vous ne voyez qu’un seul fichier pour l’état *utilisateur*. Le nom de fichier commencera par `user!`. À chaque changement d’état d’un de ces objets, le gestionnaire d’état met à jour le fichier pour refléter ce changement.

Exécutez le bot et envoyez-lui quelques messages. Recherchez ensuite le fichier de stockage et ouvrez-le. Voici le contenu JSON possible d’un bot d’écho qui suit le compteur de tours.

```json
{
  "turnNumber": "3",
  "eTag": "322"
}
```
---

## <a name="azure-table-storage"></a>Stockage de table Azure

Vous pouvez également utiliser le stockage de table Azure comme support de stockage.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Dans l’exemple Microsoft.Bot.Samples.EchoBot-AspNetCore, ajoutez une référence au package NuGet `Microsoft.Bot.Builder.Azure`.

Accédez ensuite à `Startup.cs`, ajoutez une instruction `using Microsoft.Bot.Builder.Azure;` et modifiez le code dans la méthode `ConfigureServices`.
```csharp
services.AddBot<EchoBot>(options =>
{
    options.CredentialProvider = new ConfigurationCredentialProvider(Configuration);
    // The parameters are the connection string and table name.
    // "UseDevelopmentStorage=true" is the connection string to use if you are using the Azure Storage Emulator.
    // Replace it with your own connection string if you're not using the emulator
    options.Middleware.Add(new ConversationState<EchoState>(new AzureTableStorage("UseDevelopmentStorage=true","conversationstatetable")));
    // you could also specify the cloud storage account instead of the connection string
    /* options.Middleware.Add(new ConversationState<EchoState>(
        new AzureTableStorage(WindowsAzure.Storage.CloudStorageAccount.DevelopmentStorageAccount, "conversationstatetable"))); */
    options.EnableProactiveMessages = true;
});
```
`UseDevelopmentStorage=true` représente la chaîne de connexion que vous pouvez utiliser avec l’[émulateur de stockage Azure][AzureStorageEmulator]. Remplacez-la par votre propre chaîne de connexion si vous n’utilisez pas l’émulateur.

Si la table avec le nom que vous spécifiez dans le constructeur sur `AzureTableStorage` n’existe pas, elle est créée.

<!-- 
TODO: step-by-step inspection of the stored table
-->

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

Dans l’exemple de bot d’écho `app.js`, vous pouvez créer ConversationState à l’aide de `AzureTableStorage`. 

```bash
npm install --save botbuilder-azure@preview
```

```javascript
const { BotFrameworkAdapter, FileStorage, MemoryStorage, ConversationState } = require('botbuilder');
const { TableStorage } = require('botbuilder-azure');

// ...

// Create adapter
const adapter = new BotFrameworkAdapter({
    appId: process.env.MICROSOFT_APP_ID,
    appPassword: process.env.MICROSOFT_APP_PASSWORD
});

// Add conversation state middleware
// The parameters are the connection string and table name.
// "UseDevelopmentStorage=true" is the connection string to use if you are using the Azure Storage Emulator.
// Replace it with your own connection string if you're not using the emulator
var azureStorage = new TableStorage({ tableName: "TestAzureTable1", storageAccountOrConnectionString: "UseDevelopmentStorage=true"})

// You can alternatively use your account name and table name
// var azureStorage = new TableStorage({tableName: "TestAzureTable2", storageAccessKey: "V3ah0go4DLkMtQKUPC6EbgFeXnE6GeA+veCwDNFNcdE6rqSVE/EQO/kjfemJaitPwtAkmR9lMKLtcvgPhzuxZg==", storageAccountOrConnectionString: "storageaccount"});

const conversationState = new ConversationState(azureStorage);
adapter.use(conversationState);

```

`UseDevelopmentStorage=true` représente la chaîne de connexion que vous pouvez utiliser avec l’[émulateur de stockage Azure][AzureStorageEmulator].
Si la table avec le nom que vous spécifiez dans le constructeur sur `AzureTableStorage` n’existe pas, elle est créée.

---

Pour consulter les données d’état de la conversation qui ont été enregistrées, exécutez l’exemple, puis ouvrez la table à l’aide de l’[Explorateur de stockage Azure][AzureStorageExplorer].

![données d’état de la conversation du bot d’écho dans l’Explorateur de stockage Azure](media/how-to-state/echostate-azure-storage-explorer.png)


Le **clé de partition** est une clé générée de façon unique, spécifique à la conversation actuelle. Si vous redémarrez le bot ou lancez une nouvelle conversation, cette nouvelle conversation obtiendra sa propre ligne avec sa propre clé de partition. La structure de données `EchoState` est sérialisée dans JSON et enregistrée dans la colonne **Json** de la table Azure. 

```json
{
    "$type":"Microsoft.Bot.Samples.Echo.AspNetCore.EchoState, Microsoft.Bot.Samples.EchoBot-AspNetCore",
    "TurnNumber":2,
    "LastMessage":"second message",
    "eTag":"*"
}
```

Les champs `$type` et `eTag` sont ajoutés par le kit SDK Bot Builder. Pour plus d’informations sur eTags, voir [Gestion de l’accès concurrentiel optimiste](bot-builder-howto-v4-storage.md#manage-concurrency-using-etags)


## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous savez comment utiliser l’état pour vous aider à lire et à enregistrer des données bot dans le stockage, voyons comment lire et enregistrer ces données directement dans le stockage.

> [!div class="nextstepaction"]
> [Guide pratique pour écrire directement dans le stockage](bot-builder-howto-v4-storage.md).

## <a name="additional-resources"></a>Ressources supplémentaires
Pour plus d’informations sur le stockage, voir [Stockage dans le kit SDK Bot Builder](bot-builder-storage-concept.md)

<!-- Links -->
[AzureStorageEmulator]: https://docs.microsoft.com/en-us/azure/storage/common/storage-use-emulator
[AzureStorageExplorer]: https://azure.microsoft.com/en-us/features/storage-explorer/
