---
title: Ajouter la compréhension du langage naturel à votre bot | Microsoft Docs
description: Découvrez comment utiliser LUIS pour la compréhension du langage naturel avec le SDK Bot Builder.
keywords: Language Understanding, LUIS, intention, module de reconnaissance, entités, middleware (intergiciel)
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: cognitive-services
ms.date: 11/08/18
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: eab8e2f9d437748d0bb0fefd31c03c8fb350c6b1
ms.sourcegitcommit: 8b7bdbcbb01054f6aeb80d4a65b29177b30e1c20
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/14/2018
ms.locfileid: "51645699"
---
# <a name="add-natural-language-understanding-to-your-bot"></a>Ajouter la compréhension du langage naturel à votre bot

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

La capacité à comprendre ce que veut dire votre utilisateur du point de vue de la conversation et du contexte peut être une tâche difficile, mais peut donner à votre bot un sens de la conversation plus naturel. Le service de compréhension langagière, appelé LUIS, vous permet justement de faire cela ; votre bot peut ainsi reconnaître l’intention des messages de l’utilisateur, prendre en charge un langage plus naturel de l’utilisateur et mieux diriger le flux de la conversation. Si vous avez besoin de plus de contexte sur LUIS, consultez l’article dédié à la [compréhension du langage](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/what-is-luis) pour les bots.

## <a name="prerequisites"></a>Prérequis
Cette rubrique vous montre comment configurer un bot simple qui utilise LUIS pour reconnaître quelques intentions différentes. Le code de cet article est basé sur l’exemple de traitement du langage naturel avec LUIS en [ C# ](https://aka.ms/cs-luis-sample) et [JavaScript](https://aka.ms/js-luis-sample).

## <a name="create-a-luis-app-in-the-luis-portal"></a>Créer une application LUIS dans le portail LUIS

Tout d’abord, créez un compte sur le portail [luis.ai](https://www.luis.ai) et créez une application LUIS dans le portail LUIS en suivant les instructions indiquées [ici](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/luis-how-to-start-new-app). Si vous souhaitez créer votre propre version de l’exemple d’application LUIS utilisé dans cet article, [importez](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/create-new-app#import-new-app) ce fichier `LUIS.Reminders.json` ([C#](https://github.com/Microsoft/BotBuilder-Samples/blob/v4/samples/csharp_dotnetcore/12.nlp-with-luis/CognitiveModels/LUIS-Reminders.json) | [JS](https://github.com/Microsoft/BotBuilder-Samples/blob/master/samples/javascript_nodejs/12.nlp-with-luis/cognitiveModels/reminders.json)) dans le portail LUIS pour générer votre application LUIS, puis [entraînez-la](https://docs.microsoft.com/en-us/azure/cognitive-services/LUIS/luis-how-to-train) et [publiez-la](https://docs.microsoft.com/en-us/azure/cognitive-services/LUIS/publishapp).

### <a name="obtain-values-to-connect-to-your-luis-app"></a>Obtenir les valeurs de connexion à votre application LUIS

Une fois votre application LUIS publiée, vous pouvez y accéder à partir de votre bot. Vous devrez enregistrer plusieurs valeurs pour accéder à votre application LUIS à partir de votre bot. Vous pouvez récupérer ces informations à l’aide du portail LUIS ou des outils CLI.

#### <a name="using-luis-portal"></a>Utilisation du portail LUIS
- Sélectionnez votre application LUIS publiée à partir du portail [luis.ai](https://www.luis.ai).
- Ouvrez votre application LUIS publiée, et sélectionnez l’onglet **MANAGE** (Gérer).
- Sélectionnez l’onglet **Application Information** (Informations sur l’application) à gauche, enregistrez la valeur affichée pour _Application ID_ (ID d’Application) en tant que <YOUR_APP_ID>.
- Sélectionnez l’onglet **Keys and Endpoints** (Clés et points de terminaison) à gauche, enregistrez la valeur indiquée pour _Authoring Key_ (Clé de création) en tant que <YOUR_AUTHORING_KEY>. Notez que <YOUR_SUBSCRIPTION_KEY> est identique à <YOUR_AUTHORING_KEY>. Faites défiler la page jusqu’à la fin, enregistrez la valeur affichée pour _Region_ (Région) en tant que <YOUR_REGION> et enregistrez la valeur affichée pour _Endpoint_ (Point de terminaison) en tant que <YOUR_ENDPOINT>.

#### <a name="using-cli-tools"></a>Utilisation des outils de l’interface de ligne de commande

Vous pouvez utiliser les outils CLI BotBuilder [luis](https://aka.ms/botbuilder-tools-luis) et [msbot](https://aka.ms/botbuilder-tools-msbot-readme) pour obtenir les métadonnées de votre application LUIS et les ajouter à votre fichier **.bot**.

1. Ouvrez un terminal ou une invite de commandes et accédez au répertoire racine du projet de votre bot.
2. Assurez-vous que les outils `luis` et `msbot` sont installés.

    ```shell
    npm install luis msbot
    ```

3. Exécutez `luis init` pour créer un fichier de ressources LUIS (**.luisrc**). Fournissez votre clé de création LUIS création et votre région lorsque vous y êtes invité. Vous n’avez pas besoin d’entrer votre ID d’application pour l’instant.
4. Exécutez la commande suivante pour télécharger vos métadonnées et les ajouter au fichier de configuration de votre bot.
    Si vous avez chiffré votre fichier de configuration, vous devrez fournir votre clé secrète pour mettre à jour le fichier.

    ```shell
    luis get application --appId <your-app-id> --msbot | msbot connect luis --stdin [--secret <YOUR-SECRET>]
    ```

## <a name="configure-your-bot-to-use-your-luis-app"></a>Configurer votre bot pour qu’il utilise votre application LUIS

Une référence à l’application LUIS est d’abord ajoutée lors de l’initialisation du bot. Nous pouvons ensuite l’appeler dans la logique de notre bot.

### <a name="prerequisite"></a>Configuration requise

Avant de passer au codage, assurez-vous que vous avez installé les packages nécessaires pour l’application LUIS.

# <a name="ctabcs"></a>[C#](#tab/cs)

Installez le [package NuGet](https://docs.microsoft.com/en-us/nuget/tools/package-manager-ui) suivant dans votre bot.

* `Microsoft.Bot.Builder.AI.Luis`

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

Les fonctionnalités LUIS se trouvent dans le package `botbuilder-ai`. Vous pouvez ajouter ce package à votre projet par le biais de npm :

```shell
npm install --save botbuilder-ai
```

---

# <a name="ctabcs"></a>[C#](#tab/cs)

Téléchargez et ouvrez l’[exemple de code de traitement du langage naturel LUIS](https://aka.ms/cs-luis-sample) disponible ici. Nous allons modifier le code en fonction de nos besoins. 

Tout d’abord, ajoutez les informations requises pour accéder à votre application LUIS, notamment l’ID d’application, la clé de création, la clé d’abonnement, le point de terminaison et la région dans le fichier `BotConfiguration.bot`. Il s’agit des valeurs que vous avez enregistrées précédemment à partir de votre application LUIS publiée.

```csharp
{
  "name": "LuisBot",
  "services": [
    {
      "type": "endpoint",
      "name": "development",
      "endpoint": "http://localhost:3978/api/messages",
      "appId": "",
      "appPassword": "",
      "id": "1"
    },
    {
      "type": "luis",
      "name": "LuisBot",
      "AppId": "<YOUR_APP_ID>",
      "SubscriptionKey": "<YOUR_SUBSCRIPTION_KEY>",
      "AuthoringKey": "<YOUR_AUTHORING_KEY>",
      "GetEndpoint": "<YOUR_ENDPOINT>",
      "Region": "<YOUR_REGION>"
    }
  ],
  "padlock": "",
  "version": "2.0"
}
```

Ensuite, nous allons initialiser une nouvelle instance de la classe BotService `BotServices.cs`, qui extrait les informations ci-dessus à partir de votre fichier `.bot`. Ajoutez le code suivant au fichier `BotServices.cs`.

```csharp
public class BotServices
{
    /// Initializes a new instance of the BotServices class
    public BotServices(BotConfiguration botConfiguration)
    {
        foreach (var service in botConfiguration.Services)
        {
            switch (service.Type)
            {
                case ServiceTypes.Luis:
                {
                    var luis = (LuisService)service;
                    if (luis == null)
                    {
                        throw new InvalidOperationException("The LUIS service is not configured correctly in your '.bot' file.");
                    }

                    var app = new LuisApplication(luis.AppId, luis.AuthoringKey, luis.GetEndpoint());
                    var recognizer = new LuisRecognizer(app);
                    this.LuisServices.Add(luis.Name, recognizer);
                    break;
                    }
                }
            }
        }

    /// Gets the set of LUIS Services used.
    /// LuisServices is represented as a dictionary.  
    public Dictionary<string, LuisRecognizer> LuisServices { get; } = new Dictionary<string, LuisRecognizer>();
}
```

Nous allons ensuite enregistrer l’application LUIS en tant que singleton dans le fichier `Startup.cs` en ajoutant le code suivant dans `ConfigureServices`.

```csharp
public void ConfigureServices(IServiceCollection services)
{
    var secretKey = Configuration.GetSection("botFileSecret")?.Value;
    var botFilePath = Configuration.GetSection("botFilePath")?.Value;

    // Loads .bot configuration file and adds a singleton that your Bot can access through dependency injection.
    var botConfig = BotConfiguration.Load(botFilePath ?? @".\BotConfiguration.bot", secretKey);
    services.AddSingleton(sp => botConfig ?? throw new InvalidOperationException($"The .bot config file could not be loaded. ({botConfig})"));

    // Initialize Bot Connected Services clients.
    var connectedServices = new BotServices(botConfig);
    services.AddSingleton(sp => connectedServices);
    services.AddSingleton(sp => botConfig);

    services.AddBot<LuisBot>(options =>
    {
        // Retrieve current endpoint.
        var environment = _isProduction ? "production" : "development";
        var service = botConfig.Services.Where(s => s.Type == "endpoint" && s.Name == environment).FirstOrDefault();
        if (!(service is EndpointService endpointService))
        {
            throw new InvalidOperationException($"The .bot file does not contain an endpoint with name '{environment}'.");
        }

        options.CredentialProvider = new SimpleCredentialProvider(endpointService.AppId, endpointService.AppPassword);

        // Creates a logger for the application to use.
        ILogger logger = _loggerFactory.CreateLogger<LuisBot>();

        // Catches any errors that occur during a conversation turn and logs them.
        options.OnTurnError = async (context, exception) =>
        {
            logger.LogError($"Exception caught : {exception}");
            await context.SendActivityAsync("Sorry, it looks like something went wrong.");
        };
        /// ...
    });
}
```

Ensuite, nous devons donner à votre bot cette instance de LUIS. Ouvrez `LuisBot.cs` et ajoutez le code suivant au début du fichier.

```csharp
public class LuisBot : IBot
{
    public static readonly string LuisKey = "LuisBot";
    private const string WelcomeText = "This bot will introduce you to natural language processing with LUIS. Type an utterance to get started";

    /// Services configured from the ".bot" file.
    private readonly BotServices _services;

    /// Initializes a new instance of the LuisBot class.
    public LuisBot(BotServices services)
    {
        _services = services ?? throw new System.ArgumentNullException(nameof(services));
        if (!_services.LuisServices.ContainsKey(LuisKey))
        {
            throw new System.ArgumentException($"Invalid configuration. Please check your '.bot' file for a LUIS service named '{LuisKey}'.");
        }
    }
    /// ...
}
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

Dans notre exemple, le code de démarrage se trouve dans un fichier **index.js**, le code de la logique du bot se trouve dans un fichier **bot.js**, et les informations de configuration supplémentaires se trouvent dans le fichier **nlp-with-luis.bot**.

Une fois les instructions de création de votre application LUIS et de mise à jour de votre fichier **.bot** suivies, votre fichier **nlp-with-luis.bot** devrait inclure une entrée de service correspondant à votre application LUIS.

```json
{
    "name": "YOUR_LUIS_APP_NAME",
    "description": "",
    "services": [
        {
            "type": "endpoint",
            "name": "development",
            "endpoint": "http://localhost:3978/api/messages",
            "appId": "",
            "appPassword": "",
            "id": "35"
        },
        {
            "type": "luis",
            "name": "YOUR_LUIS_APP_NAME",
            "appId": "<YOUR_APP_ID>",
            "version": "0.1",
            "authoringKey": "<YOUR_AUTHORING_KEY>",
            "subscriptionKey": "<YOUR_SUBSCRIPTION_KEY>>",
            "region": "<YOUR_REGION>",
            "id": "83"
        }
    ],
    "padlock": "",
    "version": "2.0"
}
```

Dans le fichier **bot.js**, nous lisons les informations de configuration pour générer le service LUIS et initialiser le bot.
Remplacez la valeur de `LUIS_CONFIGURATION` par le nom de votre application LUIS, tel qu’il apparaît dans votre fichier de configuration.

```javascript
// Language Understanding (LUIS) service name as defined in the .bot file.YOUR_LUIS_APP_NAME is "LuisBot" in the C# code.
const LUIS_CONFIGURATION = '<YOUR_LUIS_APP_NAME>';

// Get endpoint and LUIS configurations by service name.
const endpointConfig = botConfig.findServiceByNameOrId(BOT_CONFIGURATION);
const luisConfig = botConfig.findServiceByNameOrId(LUIS_CONFIGURATION);

// Map the contents to the required format for `LuisRecognizer`.
const luisApplication = {
    applicationId: luisConfig.appId,
    endpointKey: luisConfig.subscriptionKey || luisConfig.authoringKey,
    azureRegion: luisConfig.region
};

// Create configuration for LuisRecognizer's runtime behavior.
const luisPredictionOptions = {
    includeAllIntents: true,
    log: true,
    staging: false
};

// Create adapter...

// Create the LuisBot.
let bot;
try {
    bot = new LuisBot(luisApplication, luisPredictionOptions);
} catch (err) {
    console.error(`[botInitializationError]: ${ err }`);
    process.exit();
}
```

Créons ensuite le serveur HTTP qui écoutera les requêtes entrantes et générera les appels à la logique de notre bot.

```javascript
// Create HTTP server.
let server = restify.createServer();
server.listen(process.env.port || process.env.PORT || 3978, function() {
    console.log(`\n${ server.name } listening to ${ server.url }.`);
    console.log(`\nGet Bot Framework Emulator: https://aka.ms/botframework-emulator.`);
    console.log(`\nTo talk to your bot, open nlp-with-luis.bot file in the emulator.`);
});

// Listen for incoming requests.
server.post('/api/messages', (req, res) => {
    adapter.processActivity(req, res, async(turnContext) => {
        await bot.onTurn(turnContext);
    });
});
```

---

LUIS est maintenant configuré pour votre bot. Ensuite, examinons la façon dont nous allons obtenir l’intention de LUIS.

## <a name="get-the-intent-by-calling-luis"></a>Obtenir l’intention en appelant LUIS

Votre bot obtient des résultats de LUIS en appelant le module de reconnaissance LUIS.

# <a name="ctabcs"></a>[C#](#tab/cs)

Pour que votre bot envoie simplement une réponse basée sur l’intention que l’application LUIS a détectée, appelez `LuisRecognizer` pour obtenir un élément `RecognizerResult`. Vous pouvez effectuer cette action dans votre code toutes les fois où vous devez obtenir l’intention de LUIS.

```cs
public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))

{
    if (turnContext.Activity.Type == ActivityTypes.Message)
    {
        // Check LUIS model
        var recognizerResult = await _services.LuisServices[LuisKey].RecognizeAsync(turnContext, cancellationToken);
        var topIntent = recognizerResult?.GetTopScoringIntent();
        if (topIntent != null && topIntent.HasValue && topIntent.Value.intent != "None")
        {
            await turnContext.SendActivityAsync($"==>LUIS Top Scoring Intent: {topIntent.Value.intent}, Score: {topIntent.Value.score}\n");
        }
        else
        {
            var msg = @"No LUIS intents were found.
                        This sample is about identifying two user intents:
                        'Calendar.Add'
                        'Calendar.Find'
                        Try typing 'Add Event' or 'Show me tomorrow'.";
            await turnContext.SendActivityAsync(msg);
        }
        }
        else if (turnContext.Activity.Type == ActivityTypes.ConversationUpdate)
        {
            // Send a welcome message to the user and tell them what actions they may perform to use this bot
            await SendWelcomeMessageAsync(turnContext, cancellationToken);
        }
        else
        {
            await turnContext.SendActivityAsync($"{turnContext.Activity.Type} event detected", cancellationToken: cancellationToken);
        }
}
```

Les intentions reconnues dans l’énoncé sont retournées sous la forme d’un mappage des noms d’intention avec des scores et sont accessibles à partir `recognizerResult.Intents`. Une méthode `recognizerResult?.GetTopScoringIntent()` statique est mise à votre disposition, qui facilite la recherche de l’intention ayant reçu le meilleur score pour un jeu de résultats.

Toutes les entités reconnues sont renvoyées sous la forme d’un mappage de noms d’entité avec des valeurs et sont accessibles par le biais de `recognizerResults.entities`. Vous pouvez retourner des métadonnées d’entité supplémentaires en passant un paramètre `verbose=true` quand vous créez l’objet LuisRecognizer. Les métadonnées ajoutées sont ensuite accessibles par le biais de `recognizerResults.entities.$instance`.

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

Dans le fichier **bot.js**, nous passons l’entrée de l’utilisateur dans la méthode `recognize` du module de reconnaissance LUIS pour obtenir des réponses à partir de l’application LUIS.

```javascript
const { ActivityTypes } = require('botbuilder');
const { LuisRecognizer } = require('botbuilder-ai');

/**
 * A simple bot that responds to utterances with answers from the Language Understanding (LUIS) service.
 * If an answer is not found for an utterance, the bot responds with help.
 */
class LuisBot {
    /**
     * The LuisBot constructor requires one argument (`application`) which is used to create an instance of `LuisRecognizer`.
     * @param {LuisApplication} luisApplication The basic configuration needed to call LUIS. In this sample the configuration is retrieved from the .bot file.
     * @param {LuisPredictionOptions} luisPredictionOptions (Optional) Contains additional settings for configuring calls to LUIS.
     */
    constructor(application, luisPredictionOptions, includeApiResults) {
        this.luisRecognizer = new LuisRecognizer(application, luisPredictionOptions, true);
    }

    /**
     * Every conversation turn calls this method.
     * @param {TurnContext} turnContext Contains all the data needed for processing the conversation turn.
     */
    async onTurn(turnContext) {
        // By checking the incoming Activity type, the bot only calls LUIS in appropriate cases.
        if (turnContext.activity.type === ActivityTypes.Message) {
            // Perform a call to LUIS to retrieve results for the user's message.
            const results = await this.luisRecognizer.recognize(turnContext);

            // Since the LuisRecognizer was configured to include the raw results, get the `topScoringIntent` as specified by LUIS.
            const topIntent = results.luisResult.topScoringIntent;

            if (topIntent.intent !== 'None') {
                await turnContext.sendActivity(`LUIS Top Scoring Intent: ${ topIntent.intent }, Score: ${ topIntent.score }`);
            } else {
                // If the top scoring intent was "None" tell the user no valid intents were found and provide help.
                await turnContext.sendActivity(`No LUIS intents were found.
                                                \nThis sample is about identifying two user intents:
                                                \n - 'Calendar.Add'
                                                \n - 'Calendar.Find'
                                                \nTry typing 'Add Event' or 'Show me tomorrow'.`);
            }
        } else if (turnContext.activity.type === ActivityTypes.ConversationUpdate &&
            turnContext.activity.recipient.id !== turnContext.activity.membersAdded[0].id) {
            // If the Activity is a ConversationUpdate, send a greeting message to the user.
            await turnContext.sendActivity('Welcome to the NLP with LUIS sample! Send me a message and I will try to predict your intent.');
        } else if (turnContext.activity.type !== ActivityTypes.ConversationUpdate) {
            // Respond to all other Activity types.
            await turnContext.sendActivity(`[${ turnContext.activity.type }]-type activity detected.`);
        }
    }
}

module.exports.LuisBot = LuisBot;
```

Le module de reconnaissance LUIS retourne des informations sur le degré de correspondance entre l’énoncé et les intentions disponibles. La propriété `luisResult.intents` de l’objet de résultat contient un tableau d’intentions notées. La propriété `luisResult.topScoringIntent` de l’objet de résultat contient l’intention la mieux notée et son score.

---

## <a name="extract-entities"></a>Extraire les entités

En plus de reconnaître une intention, une application LUIS peut extraire des entités qui sont des mots importants pour répondre à la demande d’un utilisateur. Par exemple, pour un bot de renseignements météorologiques, l’application LUIS peut être en mesure d’extraire du message de l’utilisateur le lieu dont il souhaite connaître le bulletin météorologique.

Une façon courante de structurer votre conversation consiste à identifier les entités dans le message de l’utilisateur et à l’inviter à indiquer les entités requises introuvables. Ensuite, les étapes suivantes traitent la réponse à l’invite.

<!--Snip
# [C#](#tab/cs)

Let's say the message from the user was "What's the weather in Seattle"? The [LuisRecognizer](https://docs.microsoft.com/en-us/dotnet/api/microsoft.bot.builder.ai.luis.luisrecognizer) gives you a [RecognizerResult](https://docs.microsoft.com/en-us/dotnet/api/microsoft.bot.builder.core.extensions.recognizerresult) with an [`Entities` property](https://docs.microsoft.com/en-us/dotnet/api/microsoft.bot.builder.core.extensions.recognizerresult#properties-) that has this structure:

```json
{
"$instance": {
    "Weather_Location": [
        {
            "startIndex": 22,
            "endIndex": 29,
            "text": "seattle",
            "score": 0.8073087
        }
    ]
},
"Weather_Location": [
        "seattle"
    ]
}
```

The following helper function can be added to your bot to get entities out of the `RecognizerResult` from LUIS. It will require the use of the `Newtonsoft.Json.Linq` library, which you'll have to add to your **using** statements.

```cs
// Get entities from LUIS result
private T GetEntity<T>(RecognizerResult luisResult, string entityKey)
{
    var data = luisResult.Entities as IDictionary<string, JToken>;
    if (data.TryGetValue(entityKey, out JToken value))
    {
        return value.First.Value<T>();
    }
    return default(T);
}
```

When gathering information like entities from multiple steps in a conversation, it can be helpful to save the information you need in your state. If an entity is found, it can be added to the appropriate state field. In your conversation if the current step already has the associated field completed, the step to prompt for that information can be skipped.

# [JavaScript](#tab/js)

Let's say the message from the user was "What's the weather in Seattle"? The [LuisRecognizer](https://docs.microsoft.com/en-us/javascript/api/botbuilder-ai/luisrecognizer) gives you a [RecognizerResult](https://docs.microsoft.com/en-us/javascript/api/botbuilder-core-extensions/recognizerresult) with an `entities` property that has this structure:

```json
{
"$instance": {
    "Weather_Location": [
        {
            "startIndex": 22,
            "endIndex": 29,
            "text": "seattle",
            "score": 0.8073087
        }
    ]
},
"Weather_Location": [
        "seattle"
    ]
}
```

This `findEntities` function looks for any entities recognized by the LUIS app that match the incoming `entityName`.

```javascript
// Helper function for finding a specified entity
// entityResults are the results from LuisRecognizer.get(context)
function findEntities(entityName, entityResults) {
    let entities = []
    if (entityName in entityResults) {
        entityResults[entityName].forEach(entity => {
            entities.push(entity);
        });
    }
    return entities.length > 0 ? entities : undefined;
}
```
/Snip-->

Quand vous collectez des informations telles que des entités à partir de plusieurs étapes d’une conversation, il peut être utile d’enregistrer les informations dont vous avez besoin dans un état. Si une entité est trouvée, elle peut être ajoutée au champ d’état approprié. Dans votre conversation, si l’étape actuelle a déjà le champ associé renseigné, l’étape visant à demander ces informations peut être ignorée.

## <a name="additional-resources"></a>Ressources supplémentaires

Pour obtenir un exemple utilisant LUIS, consultez les projets pour [[C#](https://aka.ms/cs-luis-sample)] ou [[JavaScript](https://aka.ms/js-luis-sample)].

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Combiner des services QnA et LUIS à l’aide de l’outil Dispatch](./bot-builder-tutorial-dispatch.md)
