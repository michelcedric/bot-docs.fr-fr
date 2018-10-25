---
title: Conserver les données utilisateur | Microsoft Docs
description: Découvrez comment enregistrer les données d’état utilisateur sur un stockage dans le SDK Bot Builder.
keywords: conserver les données utilisateur, stockage, données de conversation
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 09/19/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 61e86ce9536bc5d77dc7bd411054b2f65bce8dd9
ms.sourcegitcommit: b8bd66fa955217cc00b6650f5d591b2b73c3254b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/15/2018
ms.locfileid: "49326556"
---
# <a name="persist-user-data"></a>Conserver les données utilisateur

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Quand le bot invite l’utilisateur à entrer des informations, il est possible que vous souhaitiez conserver une partie de ces informations sur un stockage sous une forme ou une autre. Le SDK Bot Builder vous permet de stocker des entrées utilisateur à l’aide d’un *stockage en mémoire* ou d’un stockage de base de données tel que *CosmosDB*. Les types de stockage local sont principalement utilisés lors du test ou du prototypage de votre bot. Toutefois, les types de stockage persistant (le stockage de base de données, par exemple) sont plus adaptés aux bots de production.

Cette rubrique vous explique comment définir votre objet de stockage et enregistrer les entrées utilisateur dans l’objet de stockage afin de les rendre persistantes. Si nous ne disposons pas déjà de cette information, nous utiliserons un dialogue pour demander son nom à l’utilisateur. Quel que soit le type de stockage que vous choisissez d’utiliser, le processus de traitement et de conservation des données est le même. Dans le code de cette rubrique, le stockage `CosmosDB` est utilisé pour conserver les données.

## <a name="prerequisites"></a>Prérequis

Certaines ressources sont requises selon l’environnement de développement que vous souhaitez utiliser.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

* [Installez Visual Studio 2015 ou version ultérieure](https://www.visualstudio.com/downloads/).
* [Installez le modèle BotBuilder V4](https://marketplace.visualstudio.com/items?itemName=BotBuilder.botbuilderv4).

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

* [Installez Visual Studio Code](https://www.visualstudio.com/downloads/).
* [Installez Node.js version 8.5 ou ultérieure](https://nodejs.org/en/).
* [Installez Yeoman](http://yeoman.io/).
* Installez le générateur de modèles Node.js Bot Builder v4.

    ```shell
    npm install generator-botbuilder
    ```

---

Les packages suivants sont utilisés au cours de ce didacticiel.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Nous allons partir d’un modèle EchoBot de base. Pour obtenir des instructions, consultez le [démarrage rapide pour .NET](~/dotnet/bot-builder-dotnet-quickstart.md).

Installez ces packages supplémentaires à partir du gestionnaire de paquets NuGet.

* **Microsoft.Bot.Builder.Azure**
* **Microsoft.Bot.Builder.Dialogs**

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Nous allons partir d’un modèle EchoBot de base. Pour obtenir des instructions, consultez le [démarrage rapide pour JavaScript](~/javascript/bot-builder-javascript-quickstart.md).

Installez ces packages npm supplémentaires.

```cmd
npm install --save botbuilder-dialogs
npm install --save botbuilder-azure
```

---

Pour tester le bot créé dans ce didacticiel, vous devez installer l’[émulateur BotFramework](https://github.com/Microsoft/BotFramework-Emulator).

## <a name="create-a-cosmosdb-service-and-update-your-application-settings"></a>Créer un service CosmosDB et mettre à jour vos paramètres d’application

Pour configurer un service et une base de données CosmosDB, suivez les instructions [d’utilisation de CosmosDB](bot-builder-howto-v4-storage.md#using-cosmos-db). En bref, les étapes sont les suivantes :

1. Dans une nouvelle fenêtre du navigateur, connectez-vous au <a href="http://portal.azure.com/" target="_blank">portail Azure</a>.
1. Cliquez sur **Créer une ressource > Bases de données > Azure Cosmos DB**.
1. Dans la page du **nouveau compte**, indiquez un nom unique dans le champ **ID**. Pour **API**, sélectionnez **SQL**et fournissez les informations relatives à l’**abonnement**, à l’**emplacement** et au **groupe de ressources**.
1. Cliquez sur **Créer**.

Ensuite, ajoutez une collection à ce service, que vous utiliserez avec ce bot.

Enregistrez les ID de base de données et de collection que vous avez utilisés pour ajouter la collection. Enregistrez également l’URI et la clé primaire à partir des paramètres de clés de la collection. Nous aurons besoin de ces éléments pour connecter notre bot au service.

### <a name="update-your-application-settings"></a>Mettre à jour les paramètres d’application

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Mettez à jour votre fichier **appsettings.json** afin d’y inclure les informations de connexion pour CosmosDB.

```csharp
{
  // Settings for CosmosDB.
  "CosmosDB": {
    "DatabaseID": "<your-database-identifier>",
    "CollectionID": "<your-collection-identifier>",
    "EndpointUri": "<your-CosmosDB-endpoint>",
    "AuthenticationKey": "<your-primary-key>"
  }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Dans le dossier de votre projet, localisez le fichier **.env** et ajoutez-y ces entrées, accompagnées de vos données Cosmos.

**.env**

```text
DB_SERVICE_ENDPOINT=<your-CosmosDB-endpoint>
AUTH_KEY=<authentication key>
DATABASE=<your-primary-key>
COLLECTION=<your-collection-identifier>
```

Ensuite, dans le fichier **index.js** principal de votre bot, remplacez `storage` de manière à utiliser `CosmosDbStorage` au lieu de `MemoryStorage`. Pendant l’exécution, les variables d’environnement seront extraites et viendront renseigner ces champs.

```javascript
const storage = new CosmosDbStorage({
    serviceEndpoint: process.env.DB_SERVICE_ENDPOINT,
    authKey: process.env.AUTH_KEY, 
    databaseId: process.env.DATABASE,
    collectionId: process.env.COLLECTION
});
```

---

## <a name="create-storage-state-manager-and-state-property-accessor-objects"></a>Créer le stockage, le gestionnaire d’état et les objets d’accesseur de propriété d’état

Les bots utilisent la gestion de l’état et les objets de stockage pour gérer et conserver l’état. Le gestionnaire fournit une couche d’abstraction qui vous permet d’accéder aux propriétés d’état à l’aide d’accesseurs de propriété d’état, indépendamment du type de stockage sous-jacent. Utilisez le gestionnaire d’état pour écrire des données dans le stockage.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

### <a name="define-a-class-for-your-user-data"></a>Définir une classe pour vos données utilisateur

Renommez le fichier **CounterState.cs** en **UserData.cs** et renommez la classe **CounterState** en **UserData**.

Mettez cette classe à jour afin de contenir les données que vous recueillerez.

```csharp
/// <summary>
/// Class for storing persistent user data.
/// </summary>
public class UserData
{
    public string Name { get; set; }
}
```

### <a name="define-a-class-for-your-state-and-state-property-accessor-objects"></a>Définir une classe pour votre état et vos objets d’accesseur de propriété d’état

Renommez le fichier **EchoBotAccessors.cs** en **BotAccessors.cs** et renommez la classe **EchoBotAccessors** en **BotAccessors**.

Mettez cette classe à jour afin de stocker les objets d’état et les accesseurs de propriété d’état requis par votre bot.

```csharp
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Dialogs;
using System;

public class BotAccessors
{
    public UserState UserState { get; }

    public ConversationState ConversationState { get; }

    public IStatePropertyAccessor<DialogState> DialogStateAccessor { get; set; }

    public IStatePropertyAccessor<UserData> UserDataAccessor { get; set; }

    public BotAccessors(UserState userState, ConversationState conversationState)
    {
        this.UserState = userState
            ?? throw new ArgumentNullException(nameof(userState));

        this.ConversationState = conversationState
            ?? throw new ArgumentNullException(nameof(conversationState));
    }
}
```

### <a name="update-the-startup-code-for-your-bot"></a>Mettre à jour le code de démarrage pour votre bot

Dans le fichier **Startup.cs**, mettez les instructions using à jour.

```csharp
using System;
using System.Linq;
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Hosting;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Azure;
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Bot.Builder.Integration;
using Microsoft.Bot.Builder.Integration.AspNet.Core;
using Microsoft.Bot.Configuration;
using Microsoft.Bot.Connector.Authentication;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Logging;
using Microsoft.Extensions.Options;
```

Dans votre méthode `ConfigureServices`, mettez à jour l’appel d’ajout de bot à partir de l’endroit où vous créez l’objet de stockage de sauvegarde, puis inscrivez votre objet d’accesseurs de bot.

Nous souhaitons que l’état de la conversation pour l’objet `DialogState` effectue le suivi de l’état du dialogue. Nous inscrivons des singletons pour l’accesseur de la propriété d’état du dialogue et l’ensemble des dialogues que notre bot utilisera. Le bot créera son propre accesseur de propriété d’état pour l’état utilisateur.

L’accesseur `BotAccessors` est un moyen efficace permettant de gérer le stockage de plusieurs objets d’état de votre bot.

```cs
public void ConfigureServices(IServiceCollection services)
{
    // Register your bot.
    services.AddBot<UserDataBot>(options =>
    {
        // ...

        // Use persistent storage and create state management objects.
        var cosmosSettings = Configuration.GetSection("CosmosDB");
        IStorage storage = new CosmosDbStorage(
            new CosmosDbStorageOptions
            {
                DatabaseId = cosmosSettings["DatabaseID"],
                CollectionId = cosmosSettings["CollectionID"],
                CosmosDBEndpoint = new Uri(cosmosSettings["EndpointUri"]),
                AuthKey = cosmosSettings["AuthenticationKey"],
            });
        options.State.Add(new ConversationState(storage));
        options.State.Add(new UserState(storage));
    });

    // Register the bot's state and state property accessor objects.
    services.AddSingleton<BotAccessors>(sp =>
    {
        var options = sp.GetRequiredService<IOptions<BotFrameworkOptions>>().Value;
        var userState = options.State.OfType<UserState>().FirstOrDefault();
        var conversationState = options.State.OfType<ConversationState>().FirstOrDefault();

        return new BotAccessors(userState, conversationState)
        {
            UserDataAccessor = userState.CreateProperty<UserData>("UserDataBot.UserData"),
            DialogStateAccessor = conversationState.CreateProperty<DialogState>("UserDataBot.DialogState"),
        };
    });
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

### <a name="update-your-server-code"></a>Mettre à jour votre code serveur

Dans le fichier **index.js** de votre projet, mettez à jour les éléments obligatoires suivants.

```javascript
// Import required bot services.
const { BotFrameworkAdapter, ConversationState, UserState } = require('botbuilder');
const { CosmosDbStorage } = require('botbuilder-azure');
```

Au cours de se didacticiel, nous utiliserons `UserState` pour stocker les données. Nous devons créer un nouvel objet `userState` et mettre à jour cette ligne de code pour transmettre un second paramètre à la classe `MainDialog`.

```javascript
// Create conversation state with in-memory storage provider.
const conversationState = new ConversationState(storage);
const userState = new UserState(storage);

// Create the main dialog.
const bot = new MyBot(conversationState, userState);
```

En cas d’erreur générale, effacez l’état de la conversation et de l’utilisateur.

```javascript
// Catch-all for errors.
adapter.onTurnError = async (context, error) => {
    // This check writes out errors to console log .vs. app insights.
    console.error(`\n [onTurnError]: ${error}`);
    // Send a message to the user
    context.sendActivity(`Oops. Something went wrong!`);
    // Clear out state
    await conversationState.load(context);
    await conversationState.clear(context);
    await userState.load(context);
    await userState.clear(context);
    // Save state changes.
    await conversationState.saveChanges(context);
    await userState.saveChanges(context);
};
```

Puis mettez à jour la boucle de serveur HTTP pour qu’elle appelle notre objet de bot.

```javascript
// Listen for incoming requests.
server.post('/api/messages', (req, res) => {
    adapter.processActivity(req, res, async (context) => {
        // Route to main dialog.
        await bot.onTurn(context);
    });
});
```

### <a name="update-your-bot-logic"></a>Mettre à jour la logique de votre bot

Dans la classe `MyBot`, vous devez indiquer les bibliothèques nécessaires au bon fonctionnement de votre bot. Dans ce didacticiel, nous utiliserons la bibliothèque des **dialogues**.

```javascript
// Required packages for this bot
const { ActivityTypes } = require('botbuilder');
const { DialogSet, WaterfallDialog, TextPrompt, NumberPrompt } = require('botbuilder-dialogs');

```

Mettez à jour le constructeur de classe `MyBot` afin d’accepter un deuxième paramètre, `userState`. De plus, mettez à jour le constructeur pour définir les états, les dialogues et les invites dont nous avons besoin dans ce didacticiel. Dans ce cas, nous définissons une cascade à deux étapes où l’_étape 1_ demande à l’utilisateur son nom et l’_étape 2_ renvoie la réponse de l’utilisateur. La logique principale du bot est tenue de conserver ces informations.

```javascript
constructor(conversationState, userState) {

    // creates a new state accessor property.
    this.conversationState = conversationState;
    this.userState = userState;

    this.dialogState = this.conversationState.createProperty('dialogState');
    this.userDataAccessor = this.userState.createProperty('userData');

    this.dialogs = new DialogSet(this.dialogState);

    // Add prompts
    this.dialogs.add(new TextPrompt('textPrompt'));

    // Add a waterfall dialog to collect and return the user's name.
    this.dialogs.add(new WaterfallDialog('greetings', [
        async function (step) {
            return await step.prompt('textPrompt', "What is your name?");
        },
        async function (step) {
            return await step.endDialog(step.result);
        }
    ]));
}
```

---

Au moment d’enregistrer les données utilisateur, plusieurs possibilités s’offrent à vous. Le SDK fournit quelques objets d’état avec différentes étendues parmi lesquelles vous pouvez choisir. Ici, nous utilisons l’état de conversation pour gérer l’objet d’état du dialogue et l’état utilisateur pour gérer les données utilisateur.

Pour plus d’informations sur le stockage et l’état (de manière générale), consultez les articles relatifs au [stockage](bot-builder-howto-v4-storage.md) et à la [gestion de l’état de la conversation et de l’utilisateur](bot-builder-howto-v4-state.md).

## <a name="create-a-greeting-dialog"></a>Créer un dialogue d’accueil

Nous utiliserons un dialogue pour recueillir le nom de l’utilisateur. Pour simplifier le scénario, le dialogue renverra le nom de l’utilisateur et le bot gérera l’objet des données utilisateur et son état.

Créez une classe **GreetingsDialog** et incluez le code suivant.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```cs
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Dialogs;

/// <summary>Defines a dialog for collecting a user's name.</summary>
public class GreetingsDialog : DialogSet
{
    /// <summary>The ID of the main dialog.</summary>
    public const string MainDialog = "main";

    /// <summary>The ID of the text prompt to use in the dialog.</summary>
    private const string TextPrompt = "textPrompt";

    /// <summary>Creates a new instance of this dialog set.</summary>
    /// <param name="dialogState">The dialog state property accessor to use for dialog state.</param>
    public GreetingsDialog(IStatePropertyAccessor<DialogState> dialogState)
        : base(dialogState)
    {
        // Add the text prompt to the dialog set.
        Add(new TextPrompt(TextPrompt));

        // Define the main dialog and add it to the set.
        Add(new WaterfallDialog(MainDialog, new WaterfallStep[]
        {
            async (stepContext, cancellationToken) =>
            {
                // Ask the user for their name.
                return await stepContext.PromptAsync(TextPrompt, new PromptOptions
                {
                    Prompt = MessageFactory.Text("What is your name?"),
                }, cancellationToken);
            },
            async (stepContext, cancellationToken) =>
            {
                // Assume that they entered their name, and return the value.
                return await stepContext.EndDialogAsync(stepContext.Result, cancellationToken);
            },
        }));
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Consultez la section précédente où le dialogue est créé dans la propriété `MainDialog` du constructeur.

---

Pour plus d’informations sur les dialogues, consultez les articles [Inviter les utilisateurs à saisir une entrée](bot-builder-prompts.md) et [Gérer un flux de conversation simple avec des dialogues](bot-builder-dialog-manage-conversation-flow.md).

## <a name="update-your-bot-to-use-the-dialog-and-user-state"></a>Mettre à jour votre bot afin d’utiliser l’état du dialogue et l’état utilisateur

Nous allons aborder la construction de bot et la gestion des entrées utilisateur séparément.

### <a name="add-the-dialog-and-a-user-state-accessor"></a>Ajouter le dialogue et un accesseur d’état utilisateur

Nous souhaitons effectuer le suivi de l’instance du dialogue et de l’accesseur de propriété d’état pour les données utilisateur.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Ajoutez le code pour initialiser votre bot.

```cs
using System.Linq;
using System.Threading;
using System.Threading.Tasks;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Bot.Schema;

/// <summary>Defines the bot for the persisting user data tutorial.</summary>
public class UserDataBot : IBot
{
    /// <summary>The bot's state and state property accessor objects.</summary>
    private BotAccessors Accessors { get; }

    /// <summary>The dialog set that has the dialog to use.</summary>
    private GreetingsDialog GreetingsDialog { get; }

    /// <summary>Create a new instance of the bot.</summary>
    /// <param name="options">The options to use for our app.</param>
    /// <param name="greetingsDialog">An instance of the dialog set.</param>
    public UserDataBot(BotAccessors botAccessors)
    {
        // Retrieve the bot's state and accessors.
        Accessors = botAccessors;

        // Create the greetings dialog.
        GreetingsDialog = new GreetingsDialog(Accessors.DialogStateAccessor);
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Consultez la section précédente où ces accesseurs d’état sont définis dans la propriété `MainDialog` du constructeur.

---

### <a name="update-the-turn-handler"></a>Mettre à jour le gestionnaire de tour

Le gestionnaire de tour accueille l’utilisateur lorsqu’il rejoint la conversation et lui répond à chaque fois qu’il envoie un message au bot. Si le bot ne dispose pas déjà du nom de l’utilisateur, il lance le dialogue d’accueil en lui demandant son nom. Au terme du dialogue, nous enregistrons le nom dans l’état utilisateur à l’aide d’un objet d’état généré par notre accesseur de propriété d’état. À la fin du tour, l’accesseur et le gestionnaire d’état écrivent les modifications de l’objet vers le stockage.

Nous ajouterons également la prise en charge de l’activité de _suppression des données utilisateur_.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Mettez à jour la méthode `OnTurnAsync` du bot.

```cs
/// <summary>Handles incoming activities to the bot.</summary>
/// <param name="turnContext">The context object for the current turn.</param>
/// <param name="cancellationToken">A cancellation token that can be used by other objects
/// or threads to receive notice of cancellation.</param>
/// <returns>A task that represents the work queued to execute.</returns>
public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
{
    // Retrieve user data from state.
    UserData userData = await Accessors.UserDataAccessor.GetAsync(turnContext, () => new UserData());

    // Establish context for our dialog from the turn context.
    DialogContext dc = await GreetingsDialog.CreateContextAsync(turnContext);

    // Handle conversation update, message, and delete user data activities.
    switch (turnContext.Activity.Type)
    {
        case ActivityTypes.ConversationUpdate:

            // Greet any user that is added to the conversation.
            IConversationUpdateActivity activity = turnContext.Activity.AsConversationUpdateActivity();
            if (activity.MembersAdded.Any(member => member.Id != activity.Recipient.Id))
            {
                if (userData.Name is null)
                {
                    // If we don't already have their name, start a dialog to collect it.
                    await turnContext.SendActivityAsync("Welcome to the User Data bot.");
                    await dc.BeginDialogAsync(GreetingsDialog.MainDialog);
                }
                else
                {
                    // Otherwise, greet them by name.
                    await turnContext.SendActivityAsync($"Hi {userData.Name}! Welcome back to the User Data bot.");
                }
            }

            break;

        case ActivityTypes.Message:

            // If there's a dialog running, continue it.
            if (dc.ActiveDialog != null)
            {
                var dialogTurnResult = await dc.ContinueDialogAsync();
                if (dialogTurnResult.Status == DialogTurnStatus.Complete
                    && dialogTurnResult.Result is string name
                    && !string.IsNullOrWhiteSpace(name))
                {
                    // If it completes successfully and returns a valid name, save the name and greet the user.
                    userData.Name = name;
                    await turnContext.SendActivityAsync($"Pleased to meet you {userData.Name}.");
                }
            }
            else if (userData.Name is null)
            {
                // Else, if we don't have the user's name yet, ask for it.
                await dc.BeginDialogAsync(GreetingsDialog.MainDialog);
            }
            else
            {
                // Else, echo the user's message text.
                await turnContext.SendActivityAsync($"{userData.Name} said, '{turnContext.Activity.Text}'.");
            }

            break;

        case ActivityTypes.DeleteUserData:

            // Delete the user's data.
            userData.Name = null;
            await turnContext.SendActivityAsync("I have deleted your user data.");

            break;
    }

    // Update the user data in the turn's state cache.
    await Accessors.UserDataAccessor.SetAsync(turnContext, userData, cancellationToken);

    // Persist any changes to storage.
    await Accessors.UserState.SaveChangesAsync(turnContext, false, cancellationToken);
    await Accessors.ConversationState.SaveChangesAsync(turnContext, false, cancellationToken);
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Mettez à jour le gestionnaire `onTurn` du bot.

```javascript
async onTurn(turnContext) {
    const dc = await this.dialogs.createContext(turnContext); // Create dialog context
    const userData = await this.userDataAccessor.get(turnContext, {});

    switch (turnContext.activity.type) {
        case ActivityTypes.ConversationUpdate:
            if (turnContext.activity.membersAdded[0].name !== 'Bot') {
                if (userData.name) {
                    await turnContext.sendActivity(`Hi ${userData.name}! Welcome back to the User Data bot.`);
                }
                else {
                    // send a "this is what the bot does" message
                    await turnContext.sendActivity('Welcome to the User Data bot.');
                    await dc.beginDialog('greetings');
                }
            }
            break;
        case ActivityTypes.Message:
            // If there is an active dialog running, continue it
            if (dc.activeDialog) {
                var turnResult = await dc.continueDialog();
                if (turnResult.status == "complete" && turnResult.result) {
                    // If it completes successfully and returns a value, save the name and greet the user.
                    userData.name = turnResult.result;
                    await this.userDataAccessor.set(turnContext, userData);
                    await turnContext.sendActivity(`Pleased to meet you ${userData.name}.`);
                }
            }
            // Else, if we don't have the user's name yet, ask for it.
            else if (!userData.name) {
                await dc.beginDialog('greetings');
            }
            // Else, echo the user's message text.
            else {
                await turnContext.sendActivity(`${userData.name} said, ${turnContext.activity.text}.`);
            }
            break;
        case ActivityTypes.DeleteUserData:
            // Delete the user's data.
            // Note: You can use the Emulator to send this activity.
            userData.name = null;
            await this.userDataAccessor.set(turnContext, userData);
            await turnContext.sendActivity("I have deleted your user data.");
            break;
    }

    // Save changes to the conversation and user states.
    await this.conversationState.saveChanges(turnContext);
    await this.userState.saveChanges(turnContext);
}
```

---

## <a name="start-your-bot-in-visual-studio"></a>Démarrer votre bot dans Visual Studio

Créez et exécutez votre application.

## <a name="start-the-emulator-and-connect-your-bot"></a>Démarrer l’émulateur et connecter votre robot

Démarrez à présent l’émulateur, puis connectez-vous à votre bot dans l’émulateur :

1. Cliquez sur le lien **Ouvrir le bot** de l’onglet de bienvenue de l’émulateur.
2. Sélectionnez le fichier .bot situé dans le répertoire où vous avez créé le projet Visual Studio.

## <a name="interact-with-your-bot"></a>Interagir avec votre bot

Envoyez un message à votre bot, et le bot vous enverra un message à son tour.
![Émulateur en cours d’exécution](../media/emulator-v4/emulator-running.png)

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Gérer l’état de la conversation et de l’utilisateur](bot-builder-howto-v4-state.md)
