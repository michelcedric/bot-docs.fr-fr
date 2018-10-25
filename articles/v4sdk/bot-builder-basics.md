---
title: Activités du bot dans le SDK Bot Builder | Microsoft Docs
description: Décrit le fonctionnement des activités et du http dans le SDK Bot Builder.
keywords: flux de conversation, tour, conversation du bot, dialogues, invites, cascades, jeu de dialogues
author: johnataylor
ms.author: johtaylo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 9/26/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: b929020908990a97e039be6467eef033c50228db
ms.sourcegitcommit: aef7d80ceb9c3ec1cfb40131709a714c42960965
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/17/2018
ms.locfileid: "49383135"
---
# <a name="understanding-how-bots-work"></a>Comprendre le fonctionnement des bots

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Un bot est une application avec laquelle les utilisateurs interagissent par le biais d’une conversation textuelle, graphique (cartes ou images) ou vocale. Chaque interaction entre l’utilisateur et le bot génère une *activité*. Le Service Bot envoie des informations entre l’application de l’utilisateur connectée au bot (Facebook, Skype, Slack, etc., que nous appelons le *canal*) et le bot. Chaque canal peut inclure des informations supplémentaires dans les activités qu’ils envoient. Avant de créer des bots, il est important de comprendre comment un bot utilise les objets d’activité pour communiquer avec ses utilisateurs. Examinons tout d’abord les activités qui sont échangées lorsque nous exécutons un simple bot d’écho.

![diagramme d’activités](media/bot-builder-activity.png)

Voici les deux types d’activités illustrés ici : *mise à jour de conversation* et *message*.

Le service Bot Framework peut envoyer une mise à jour de conversation lorsque quelqu’un rejoint la conversation. Par exemple, au démarrage d’une conversation avec l’émulateur Bot Framework, vous verrez deux activités de mise à jour de conversation (une pour l’utilisateur rejoignant la conversation et l’autre pour le bot rejoignant la conversation). Pour distinguer ces activités de mise à jour de conversation, vérifiez si la propriété *members added* inclut un membre autre que le robot. 

L’activité de message transfère les informations de conversation entre les parties. Dans un exemple de bot d’écho, les activités de message transfèrent du texte simple et le canal rend ce texte. L’activité de message peut également contenir du texte à prononcer, des actions suggérées ou des cartes à afficher.

Dans cet exemple, le bot crée et envoie une activité de message en réponse à l’activité de message entrant reçu. Toutefois, un robot peut répondre par d’autres moyens à une activité de message reçu. Il n’est pas rare qu’un robot répondre à une activité de mise à jour de conversation en envoyant un texte de bienvenue dans une activité de message. La section [Accueillir l’utilisateur](bot-builder-welcome-user.md) fournit plus d’informations à ce sujet.

### <a name="http-details"></a>Détails HTTP

Les activités arrivent dans le bot à partir du service Bot Framework via une requête HTTP POST. Le bot répond à la requête POST entrante avec un code d’état HTTP 200. Les activités sont envoyées du bot au canal dans une requête HTTP POST séparée vers le service Bot Framework. Leur réception génère un code d’état HTTP 200.

Le protocole ne spécifie pas l’ordre dans lequel ces demandes POST et leurs accusés de réception sont transmis. Toutefois, pour s’adapter aux infrastructures de service HTTP courantes, ces demandes sont généralement imbriquées, ce qui signifie que la requête HTTP sortante est effectuée à partir du bot dans le périmètre de la requête HTTP entrante. Ce schéma de fonctionnement est illustré dans le diagramme ci-dessus. Dans la mesure où il existe deux connexions HTTP distinctes à la suite, le modèle de sécurité doit couvrir les deux.

### <a name="defining-a-turn"></a>Définition d’un tour

Un tour, en ce qui concerne nos bots, est utilisé pour décrire tout le traitement associé à l’arrivée d’une activité. 

L’objet de *contexte de tour* fournit des informations sur l’activité, comme l’expéditeur et le destinataire, le canal, et d’autres données nécessaires pour traiter l’activité. Il permet également d’ajouter des informations pendant le tour entre les différentes couches du bot.

Le contexte de tour est l’une des abstractions les plus importantes du SDK. Il transfère non seulement l’activité entrante à tous les composants d’intergiciel et à la logique d’application, mais il fournit également le mécanisme par lequel les composants d’intergiciel et la logique d’application peuvent envoyer des activités sortantes.

## <a name="the-activity-processing-stack"></a>Pile de traitement d’une activité

Nous allons explorer le diagramme précédent en nous concentrant sur l’arrivée d’une activité de message.

![pile de traitement d’une activité](media/bot-builder-activity-processing-stack.png)

Dans l’exemple ci-dessus, le bot a répondu à l’activité de message avec une autre activité de message contenant le même message texte. Le traitement commence par la requête HTTP POST. Les informations de l’activité sont transportées sous la forme d’une charge JSON arrivant sur le serveur Web. En C#, il s’agit généralement d’un projet ASP.NET. Dans un projet JavaScript Node.js, il s’agit souvent d’une infrastructure populaire comme Express ou Restify.

L’*adaptateur*, un composant intégré du SDK, sert de conducteur de l’infrastructure. Le service utilise les informations de l’activité pour créer un objet d’activité, puis il appelle la méthode *process activity* de l’adaptateur, tout en passant l’objet et les informations d’authentification de l’activité (cet appel est encapsulé dans les bibliothèques en C#, mais il est visible en JavaScript). Lors de la réception de l’activité, l’adaptateur crée un objet de contexte de tour et appelle l’[intergiciel](#middleware). Le traitement se poursuit de l’intergiciel vers la logique du bot, le pipeline se termine et l’adaptateur supprime l’objet de contexte de tour.

Le *gestionnaire de tour* du bot, qui constitue la majeure partie de la logique d’application, prend le contexte de tour comme argument. Le gestionnaire de tour traite généralement le contenu de l’activité entrante et génère une ou plusieurs activités pour y répondre, puis elle les envoie à l’aide de la méthode *send activity* du contexte. L’appel de la méthode send activity envoie une activité au canal de l’utilisateur, sauf si le traitement est interrompu. L’activité passe dans tous les [gestionnaires d’événements](#response-event-handlers) inscrits avant d’être envoyée au canal.

## <a name="middleware"></a>Middlewares

L’intergiciel est un ensemble linéaire de composants qui sont tous ajoutés et exécutés dans l’ordre, afin que chacun puisse intervenir sur l’activité avant et après le gestionnaire de tour du bot, et que chacun puisse accéder au contexte de tour de cette activité. À moins que l’intergiciel ne soit [interrompu](~/v4sdk/bot-builder-concept-middleware.md#short-circuiting), la dernière étape de son pipeline est un rappel du gestionnaire de tour du bot, avant de retourner en haut de la pile. Pour plus d’informations sur les intergiciels, consultez [la rubrique consacrée aux intergiciels](~/v4sdk/bot-builder-concept-middleware.md).

## <a name="generating-responses"></a>Génération de réponses

Le contexte de tour fournit des méthodes de réponse aux activités pour permettre au code de répondre à une activité :

* Les méthodes _d’envoi d’une activité_ et _d’envoi d’activités_ envoient une ou plusieurs activités à la conversation.
* Si le canal la prend en charge, la méthode de _mise à jour d’une activité_ met à jour une activité au sein de la conversation.
* Si le canal la prend en charge, la méthode de _suppression d’une activité_ supprime une activité de la conversation.

Chaque méthode de réponse s’exécute dans un processus asynchrone. Quand elle est appelée, la méthode de réponse à une activité clone la liste de [gestionnaires d’événements](#response-event-handlers) associée avant de commencer à appeler les gestionnaires, ce qui signifie qu’elle contient chaque gestionnaire ajouté à ce stade, mais rien qui est ajouté après le démarrage du processus.

Cela signifie également que l’ordre de vos réponses n’est pas garanti pour les appels d’activités indépendantes, en particulier quand une tâche est plus complexe qu’une autre. Si votre bot peut générer plusieurs réponses à une activité entrante, vérifiez qu’elles ont du sens quel que soit l’ordre dans lequel l’utilisateur les reçoit. La seule exception est la méthode *send activities*, qui vous permet d’envoyer un ensemble ordonné d’activités.

> [!IMPORTANT]
> Le thread qui traite le tour de bot principal se charge de supprimer l’objet de contexte quand il prend fin. **Veillez à exécuter une opération `await` sur tous les appels d’activité** afin que le thread principal attende l’activité générée avant de terminer son traitement et de supprimer le contexte de tour. Sinon, si une réponse (y compris ses gestionnaires) prend un certain temps et essaie d’agir sur l’objet de contexte, elle risque de recevoir une erreur `Context was disposed`. 

## <a name="response-event-handlers"></a>Gestionnaires d’événements de réponse

En plus de la logique d’application et d’intergiciel, vous pouvez ajouter des gestionnaires de réponse (parfois également appelés gestionnaires d’événements ou gestionnaires d’événements d’activité) à l’objet de contexte. Ces gestionnaires sont appelés quand la [réponse](#generating-responses) associée se produit sur l’objet de contexte actuel, avant l’exécution de la réponse proprement dite. Ces gestionnaires sont utiles quand vous savez que vous souhaitez faire quelque chose, avant ou après l’événement réel, pour toute activité de ce type pour le reste de la réponse actuelle.

> [!WARNING]
> Veillez à ne pas appeler une méthode de réponse d’activité à partir de son propre gestionnaire d’événements de réponse, par exemple, en appelant la méthode d’envoi d’activité depuis un gestionnaire _écoutant les envois d’activité_. Cela peut générer une boucle infinie.

Rappelez-vous que chaque nouvelle activité obtient un nouveau thread sur lequel s’exécuter. Quand le thread destiné à traiter l’activité est créé, la liste des gestionnaires de cette activité est copiée sur ce nouveau thread. Aucun gestionnaire ajouté après ce point n’est exécuté pour cet événement d’activité spécifique.

Les gestionnaires inscrits sur un objet de contexte sont traités de façon très semblable à la façon dont l’adaptateur gère le [pipeline d’intergiciels](~/v4sdk/bot-builder-concept-middleware.md#the-bot-middleware-pipeline). Concrètement, les gestionnaires sont appelés dans l’ordre dans lequel ils sont ajoutés, et l’appel du délégué _next_ transmet le contrôle au gestionnaire d’événements inscrit suivant. Si un gestionnaire n’appelle pas le délégué next, aucun des gestionnaires d’événements suivants n’est appelé ; l’événement est victime d’un [court-circuitage](~/v4sdk/bot-builder-concept-middleware.md#short-circuiting), ce qui empêche l’adaptateur d’envoyer la réponse au canal.

## <a name="bot-structure"></a>Structure du bot

Examinons l’exemple de bot d’écho avec compteur [[C#](https://aka.ms/EchoBotWithStateCSharp) | [JS](https://aka.ms/EchoBotWithStateJS)] et ses composants clés.

# <a name="ctabcs"></a>[C#](#tab/cs)

Un bot est un type d’application web [ASP.NET Core](https://docs.microsoft.com/aspnet/core/?view=aspnetcore-2.1). Si vous étudiez les notions de base d’[ASP.NET](https://docs.microsoft.com/aspnet/core/fundamentals/index?view=aspnetcore-2.1&tabs=aspnetcore2x), vous découvrirez qu’il existe un code similaire dans les fichiers tels que Program.cs et Startup.cs. Ces fichiers sont requis pour toutes les applications web et ne sont pas propres à un bot. Le code de certains de ces fichiers ne sera pas copié ici, mais vous pouvez vous référer à l’exemple de bot d’écho avec compteur.

### <a name="echowithcounterbotcs"></a>EchoWithCounterBot.cs

La logique principale du bot est définie dans la classe `EchoWithCounterBot` dérivée de l’interface `IBot`. `IBot` définit une méthode `OnTurnAsync` unique. Votre application doit implémenter cette méthode. `OnTurnAsync` possède une classe turnContext qui fournit des informations sur l’activité entrante. L’activité entrante correspond à la requête HTTP entrante. Les activités peuvent être de différents types. Nous allons donc commencer par vérifier si votre bot a reçu un message. S’il s’agit d’un message, l’état de la conversation est obtenu à partir du contexte de tour, le compteur de tours est incrémenté, et la valeur du compteur de tours est conservée dans l’état de la conversation. Un message est ensuite renvoyé à l’utilisateur à l’aide de l’appel SendActivityAsync. L’activité sortante correspond à la requête HTTP sortante.

```cs
public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
{
    if (turnContext.Activity.Type == ActivityTypes.Message)
    {
        // Get the conversation state from the turn context.
        var oldState = await _accessors.CounterState.GetAsync(turnContext, () => new CounterState());

        // Bump the turn count for this conversation.
        var newState = new CounterState { TurnCount = oldState.TurnCount + 1 };

        // Set the property using the accessor.
        await _accessors.CounterState.SetAsync(turnContext, newState);

        // Save the new turn count into the conversation state.
        await _accessors.ConversationState.SaveChangesAsync(turnContext);

        // Echo back to the user whatever they typed.
        var responseMessage = $"Turn {newState.TurnCount}: You sent '{turnContext.Activity.Text}'\n";
        await turnContext.SendActivityAsync(responseMessage);
    }
    else
    {
        await turnContext.SendActivityAsync($"{turnContext.Activity.Type} event detected");
    }
}
```

### <a name="startupcs"></a>Startup.cs

La méthode `ConfigureServices` charge les services connectés depuis le fichier [.bot](bot-builder-basics.md#the-bot-file), intercepte toutes les erreurs qui se produisent pendant un tour de conversation et les enregistre, configure votre fournisseur d’informations d’identification, et crée un objet d’état de conversation pour stocker les données de la conversation dans la mémoire.

```csharp
services.AddBot<EchoWithCounterBot>(options =>
{
    // Creates a logger for the application to use.
    ILogger logger = _loggerFactory.CreateLogger<EchoWithCounterBot>();

    var secretKey = Configuration.GetSection("botFileSecret")?.Value;
    var botFilePath = Configuration.GetSection("botFilePath")?.Value;

    // Loads .bot configuration file and adds a singleton that your Bot can access through dependency injection.
    BotConfiguration botConfig = null;
    try
    {
        botConfig = BotConfiguration.Load(botFilePath ?? @".\BotConfiguration.bot", secretKey);
    }
    catch
    {
        //...
    }

    services.AddSingleton(sp => botConfig);

    // Retrieve current endpoint.
    var environment = _isProduction ? "production" : "development";
    var service = botConfig.Services.Where(s => s.Type == "endpoint" && s.Name == environment).FirstOrDefault();
    if (!(service is EndpointService endpointService))
    {
        throw new InvalidOperationException($"The .bot file does not contain an endpoint with name '{environment}'.");
    }

    options.CredentialProvider = new SimpleCredentialProvider(endpointService.AppId, endpointService.AppPassword);

    // Catches any errors that occur during a conversation turn and logs them.
    options.OnTurnError = async (context, exception) =>
    {
        logger.LogError($"Exception caught : {exception}");
        await context.SendActivityAsync("Sorry, it looks like something went wrong.");
    };

    // The Memory Storage used here is for local bot debugging only. When the bot
    // is restarted, everything stored in memory will be gone.
    IStorage dataStore = new MemoryStorage();

    // ...

    // Create Conversation State object.
    // The Conversation State object is where we persist anything at the conversation-scope.
    var conversationState = new ConversationState(dataStore);

    options.State.Add(conversationState);
});
```

Elle crée et enregistre également les `EchoBotAccessors` qui sont définis dans le fichier **EchoBotStateAccessors.cs** et qui sont passés dans le constructeur `EchoWithCounterBot` public à l’aide de l’infrastructure d’injection de dépendance dans ASP.NET Core.

```csharp
// Create and register state accessors.
// Accessors created here are passed into the IBot-derived class on every turn.
services.AddSingleton<EchoBotAccessors>(sp =>
{
    var options = sp.GetRequiredService<IOptions<BotFrameworkOptions>>().Value;
    // ...
    var conversationState = options.State.OfType<ConversationState>().FirstOrDefault();
    // ...

    // Create the custom state accessor.
    // State accessors enable other components to read and write individual properties of state.
    var accessors = new EchoBotAccessors(conversationState)
    {
        CounterState = conversationState.CreateProperty<CounterState>(EchoBotAccessors.CounterStateName),
    };

    return accessors;
});
```

La méthode `Configure` achève la configuration de votre application en précisant que l’application utilise Bot Framework et quelques autres fichiers. Tous les bots qui utilisent Bot Framework nécessitent cet appel de configuration. `ConfigureServices` et `Configure` sont appelés par le runtime au démarrage de l’application.

### <a name="counterstatecs"></a>CounterState.cs

Ce fichier contient une classe simple que notre bot utilise pour conserver l’état actuel. Il comporte uniquement un élément `int` que nous utilisons pour incrémenter le compteur.

```cs
public class CounterState
{
    public int TurnCount { get; set; } = 0;
}
```

### <a name="echobotaccessorscs"></a>EchoBotAccessors.cs

La classe `EchoBotAccessors` est créée comme un singleton dans la classe `Startup` et passée dans la classe IBot dérivée. Dans ce cas, `public class EchoWithCounterBot : IBot`. Le bot utilise l’accesseur pour conserver les données de la conversation. Le constructeur de `EchoBotAccessors` est passé dans un objet de conversation créé dans le fichier Startup.cs.

```cs
public class EchoBotAccessors
{
    public EchoBotAccessors(ConversationState conversationState)
    {
        ConversationState = conversationState ?? throw new ArgumentNullException(nameof(conversationState));
    }

    public static string CounterStateName { get; } = $"{nameof(EchoBotAccessors)}.CounterState";

    public IStatePropertyAccessor<CounterState> CounterState { get; set; }

    public ConversationState ConversationState { get; }
}
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

La section système contient principalement les fichiers **package.json**, **.env**, **index.js** et **README.md**. Le code de certains fichiers n’est pas copié ici, mais vous le verrez lorsque vous exécuterez le bot.

### <a name="packagejson"></a>package.json

Le fichier **package.json** spécifie les dépendances et leurs versions associées pour votre bot. Toutes ces informations sont configurées par le modèle et par votre système.

### <a name="env-file"></a>Fichier .env

Le fichier **.env** spécifie les informations de configuration de votre bot, notamment le numéro de port, l’ID de l’application et le mot de passe. Si vous utilisez certaines technologies ou ce bot en production, vous devrez ajouter vos propres clés ou URL à cette configuration. Toutefois, pour les besoins de ce bot Echo, vous n’avez rien à faire à ce stade ; l’ID de l’application et le mot de passe peuvent rester non définis pour l’instant.

Pour utiliser le fichier de configuration **.env**, le modèle doit inclure un package supplémentaire.  Commencez par obtenir le package `dotenv` à partir de npm :

`npm install dotenv`

### <a name="indexjs"></a>index.js

`index.js` configure votre bot et le service d’hébergement qui transférera les activités à la logique de votre bot.

#### <a name="required-libraries"></a>Bibliothèques requises

Le tout début de votre fichier `index.js` répertorie une série de modules ou de bibliothèques requis. Ces modules vous donnent accès à un ensemble de fonctions que vous souhaiterez peut-être inclure dans votre application.

```javascript
// Import required packages
const path = require('path');
const restify = require('restify');

// Import required bot services. See https://aka.ms/bot-services to learn more about the different parts of a bot.
const { BotFrameworkAdapter, ConversationState, MemoryStorage } = require('botbuilder');
// Import required bot configuration.
const { BotConfiguration } = require('botframework-config');

const { EchoBot } = require('./bot');

// Read botFilePath and botFileSecret from .env file
// Note: Ensure you have a .env file and include botFilePath and botFileSecret.
const ENV_FILE = path.join(__dirname, '.env');
const env = require('dotenv').config({ path: ENV_FILE });
```

#### <a name="bot-configuration"></a>Configuration du bot

La partie suivante charge les informations à partir du fichier de configuration de votre bot.

```javascript
// Get the .bot file path
// See https://aka.ms/about-bot-file to learn more about .bot file its use and bot configuration.
const BOT_FILE = path.join(__dirname, (process.env.botFilePath || ''));
let botConfig;
try {
    // Read bot configuration from .bot file.
    botConfig = BotConfiguration.loadSync(BOT_FILE, process.env.botFileSecret);
} catch (err) {
    console.error(`\nError reading bot file. Please ensure you have valid botFilePath and botFileSecret set for your environment.`);
    console.error(`\n - The botFileSecret is available under appsettings for your Azure Bot Service bot.`);
    console.error(`\n - If you are running this bot locally, consider adding a .env file with botFilePath and botFileSecret.`);
    console.error(`\n - See https://aka.ms/about-bot-file to learn more about .bot file its use and bot configuration.\n\n`);
    process.exit();
}

// For local development configuration as defined in .bot file
const DEV_ENVIRONMENT = 'development';

// Define name of the endpoint configuration section from the .bot file
const BOT_CONFIGURATION = (process.env.NODE_ENV || DEV_ENVIRONMENT);

// Get bot endpoint configuration by service name
// Bot configuration as defined in .bot file
const endpointConfig = botConfig.findServiceByNameOrId(BOT_CONFIGURATION);
```

#### <a name="bot-adapter-http-server-and-bot-state"></a>Adaptateur de bot, serveur HTTP et état du bot

Les parties suivantes configurent le serveur et l’adaptateur qui permettent à votre bot de communiquer avec l’utilisateur et d’envoyer des réponses. Le serveur écoute le port spécifié à partir du fichier de configuration **BotConfiguration.bot** ou revient au port _3978_ pour la connexion à votre émulateur. L’adaptateur jouera le rôle de conducteur de votre bot en dirigeant les communications entrantes et sortantes, l’authentification, et ainsi de suite.

Nous créons également un objet d’état qui utilise `MemoryStorage` en tant que fournisseur de stockage. Cet état est défini sous la forme `ConversationState`, ce qui signifie simplement que le bot conserve l’état de votre conversation. `ConversationState` stocke en mémoire les informations qui vous intéressent, un simple compteur de tours dans le cas présent.

```javascript
// Create bot adapter.
// See https://aka.ms/about-bot-adapter to learn more about bot adapter.
const adapter = new BotFrameworkAdapter({
    appId: endpointConfig.appId || process.env.microsoftAppID,
    appPassword: endpointConfig.appPassword || process.env.microsoftAppPassword
});

// Catch-all for any unhandled errors in your bot.
adapter.onTurnError = async (context, error) => {
    // This check writes out errors to console log .vs. app insights.
    console.error(`\n [onTurnError]: ${ error }`);
    // Send a message to the user
    context.sendActivity(`Oops. Something went wrong!`);
    // Clear out state
    await conversationState.clear(context);
    // Save state changes.
    await conversationState.saveChanges(context);
};

// Define a state store for your bot. See https://aka.ms/about-bot-state to learn more about using MemoryStorage.
// A bot requires a state store to persist the dialog and user state between messages.
let conversationState;

// For local development, in-memory storage is used.
// CAUTION: The Memory Storage used here is for local bot debugging only. When the bot
// is restarted, anything stored in memory will be gone.
const memoryStorage = new MemoryStorage();
conversationState = new ConversationState(memoryStorage);

// Create the main dialog.
const bot = new EchoBot(conversationState);

// Create HTTP server
let server = restify.createServer();
server.listen(process.env.port || process.env.PORT || 3978, function() {
    console.log(`\n${ server.name } listening to ${ server.url }`);
    console.log(`\nGet Bot Framework Emulator: https://aka.ms/botframework-emulator`);
    console.log(`\nTo talk to your bot, open echoBot-with-counter.bot file in the Emulator`);
});
```

#### <a name="bot-logic"></a>Logique du bot

L’élément `processActivity` de l’adaptateur envoie les activités entrantes à la logique de votre bot.
Le troisième paramètre de `processActivity` est un gestionnaire de fonctions qui est appelé pour exécuter la logique du bot une fois que [l’activité](#the-activity-processing-stack) reçue a été prétraitée par l’adaptateur et acheminée par le biais d’un intergiciel. La variable de contexte de tour, transmise sous la forme d’un argument au gestionnaire de fonctions, peut être utilisée pour fournir des informations sur l’activité entrante, l’expéditeur et le destinataire, le canal, la conversation, etc. Le traitement de l’activité est acheminé vers l’élément `onTurn` de l’EchoBot.

```javascript
// Listen for incoming requests.
server.post('/api/messages', (req, res) => {
    // Route received request to adapter for processing
    adapter.processActivity(req, res, (context) => {
        // Route to main dialog.
        await bot.onTurn(context);
    });
});
```

### <a name="echobot"></a>EchoBot

Tout traitement d’activité est acheminé vers le gestionnaire `onTurn` de cette classe. Lorsque la classe est créée, un objet d’état y est passé. Avec cet objet d’état, le constructeur crée un accesseur `this.countProperty` pour conserver le compteur de tours de ce bot.

À chaque tour, nous commençons par vérifier si le bot a reçu un message. Si ce n’est pas le cas, nous renvoyons le type d’activité reçu. Ensuite, nous créons une variable d’état qui contient les informations de conversation de votre bot. Si la variable de compteur est `undefined`, elle est définie sur 1 (ce qui se produit lors du premier démarrage de votre bot) ou elle est incrémentée à chaque nouveau message. Nous renvoyons le nombre de tours à l’utilisateur, ainsi que le message envoyé par ce dernier. Enfin, nous définissons le nombre de tours et enregistrons les modifications de l’état.

```javascript
const { ActivityTypes } = require('botbuilder');

// Turn counter property
const TURN_COUNTER_PROPERTY = 'turnCounterProperty';

class EchoBot {

    constructor(conversationState) {
        // Creates a new state accessor property.
        // See https://aka.ms/about-bot-state-accessors to learn more about the bot state and state accessors
        this.countProperty = conversationState.createProperty(TURN_COUNTER_PROPERTY);
        this.conversationState = conversationState;
    }

    async onTurn(turnContext) {
        // Handle message activity type. User's responses via text or speech or card interactions flow back to the bot as Message activity.
        // Message activities may contain text, speech, interactive cards, and binary or unknown attachments.
        // see https://aka.ms/about-bot-activity-message to learn more about the message and other activity types
        if (turnContext.activity.type === ActivityTypes.Message) {
            // read from state.
            let count = await this.countProperty.get(turnContext);
            count = count === undefined ? 1 : ++count;
            await turnContext.sendActivity(`${ count }: You said "${ turnContext.activity.text }"`);
            // increment and set turn counter.
            await this.countProperty.set(turnContext, count);
        } else {
            // Generic handler for all other activity types.
            await turnContext.sendActivity(`[${ turnContext.activity.type } event detected]`);
        }
        // Save state changes
        await this.conversationState.saveChanges(turnContext);
    }
}

exports.EchoBot = EchoBot;
```

---

### <a name="the-bot-file"></a>Le fichier .bot

Le fichier **.bot** contient des informations, notamment le point de terminaison, l’ID/mot de passe d’application et des références aux services qui sont utilisés par le bot. Ce fichier est créé à votre intention lorsque vous commencez à créer un bot à partir d’un modèle, mais vous pouvez créer votre propre fichier par le biais de l’émulateur ou d’autres outils. Vous pouvez spécifier le fichier .bot à utiliser lorsque vous testez votre bot avec l’[émulateur](../bot-service-debug-emulator.md).

```json
{
    "name": "echobot-with-counter",
    "services": [
        {
            "type": "endpoint",
            "name": "development",
            "endpoint": "http://localhost:3978/api/messages",
            "appId": "",
            "appPassword": "",
            "id": "1"
        }
    ],
    "padlock": "",
    "version": "2.0"
}
```

## <a name="additional-resources"></a>Ressources supplémentaires

Pour plus d’informations sur la gestion des données d’état, consultez la rubrique [Gérer l’état de la conversation et l’état utilisateur](bot-builder-howto-v4-state.md)

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Créer un robot](~/bot-service-quickstart.md)
