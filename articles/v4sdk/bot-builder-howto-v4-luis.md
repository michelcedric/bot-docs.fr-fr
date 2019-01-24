---
title: Ajouter la compréhension du langage naturel à votre bot | Microsoft Docs
description: Découvrez comment utiliser LUIS pour la compréhension du langage naturel avec le kit SDK Bot Framework.
keywords: Language Understanding, LUIS, intention, module de reconnaissance, entités, middleware (intergiciel)
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: cognitive-services
ms.date: 11/28/18
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 4c43426f508d629c325889da6a9f7b06cac7e846
ms.sourcegitcommit: c6ce4c42fc56ce1e12b45358d2c747fb77eb74e2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/22/2019
ms.locfileid: "54453893"
---
# <a name="add-natural-language-understanding-to-your-bot"></a>Ajouter la compréhension du langage naturel à votre bot

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

La capacité à comprendre ce que veut dire votre utilisateur du point de vue de la conversation et du contexte peut être une tâche difficile, mais peut donner à votre bot un sens de la conversation plus naturel. Le service de compréhension langagière, appelé LUIS, vous permet justement de faire cela ; votre bot peut ainsi reconnaître l’intention des messages de l’utilisateur, prendre en charge un langage plus naturel de l’utilisateur et mieux diriger le flux de la conversation. Cette rubrique vous montre comment configurer un bot simple qui utilise LUIS pour reconnaître quelques intentions différentes. 
## <a name="prerequisites"></a>Prérequis
- Compte [luis.ai](https://www.luis.ai)
- [Bot Framework Emulator](https://github.com/Microsoft/BotFramework-Emulator/blob/master/README.md#download)
- Le code de cet article est basé sur l’exemple **de traitement du langage naturel avec LUIS**. Vous aurez besoin d’une copie de l’exemple en [ C# ](https://aka.ms/cs-luis-sample) ou en [JS](https://aka.ms/js-luis-sample). 
- Connaissances des [concepts de base des bots](bot-builder-basics.md), du [traitement en langage naturel](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/what-is-luis), et du fichier [.bot](bot-file-basics.md).

## <a name="create-a-luis-app-in-the-luis-portal"></a>Créer une application LUIS dans le portail LUIS
Connectez-vous au portail LUIS pour créer votre propre version de l’exemple d’application LUIS. Vous pouvez créer et gérer vos applications sur la page **Mes applications**. 

1. Sélectionnez **Importer une nouvelle application**. 
1. Cliquez sur **Choisir un fichier d’application (format JSON)...** 
1. Sélectionnez le fichier `reminders.json` situé dans le dossier `CognitiveModels` de l’exemple. Dans **Nom facultatif**, entrez **LuisBot**. Ce fichier contient trois intentions : Calendar_Add, Calendar_Find et None. Nous allons utiliser ces intentions pour comprendre ce que l’utilisateur voulait dire lorsqu’il envoie un message au bot. Si vous souhaitez inclure des entités, consultez la [section facultative](#optional---extract-entities) à la fin de cet article.
1. Effectuez l’[apprentissage](https://docs.microsoft.com/en-us/azure/cognitive-services/LUIS/luis-how-to-train) de l’application.
1. [Publiez](https://docs.microsoft.com/en-us/azure/cognitive-services/LUIS/publishapp) l’application dans un environnement de *production*.

### <a name="obtain-values-to-connect-to-your-luis-app"></a>Obtenir les valeurs de connexion à votre application LUIS

Une fois votre application LUIS publiée, vous pouvez y accéder à partir de votre bot. Vous devrez enregistrer plusieurs valeurs pour accéder à votre application LUIS à partir de votre bot. Vous pouvez récupérer ces informations à l’aide du portail LUIS.

#### <a name="retrieve-application-information-from-the-luisai-portal"></a>Récupérer des informations sur l’application à partir du portail LUIS.ai
Le fichier .bot agit comme le lieu de rassemblement pour toutes les références de service. Les informations que vous récupérez seront ajoutées au fichier .bot dans la section suivante. 
1. Sélectionnez votre application LUIS publiée à partir du portail [luis.ai](https://www.luis.ai).
1. Ouvrez votre application LUIS publiée, et sélectionnez l’onglet **MANAGE** (Gérer).
1. Sélectionnez l’onglet **Application Information** (Informations sur l’application) à gauche, enregistrez la valeur affichée pour _Application ID_ (ID d’Application) en tant que <YOUR_APP_ID>.
1. Sélectionnez l’onglet **Keys and Endpoints** (Clés et points de terminaison) à gauche, enregistrez la valeur indiquée pour _Authoring Key_ (Clé de création) en tant que <YOUR_AUTHORING_KEY>. Notez que *votre clé d’abonnement* est identique à *votre clé de création*. 
1. Faites défiler jusqu'à la fin de la page, enregistrez la valeur affichée pour _Région_ comme <YOUR_REGION>.
1. Enregistrez la valeur affichée pour _point de terminaison_ comme <YOUR_ENDPOINT>.

#### <a name="update-the-bot-file"></a>Mettre à jour le fichier bot
Ajoutez les informations requises pour accéder à votre application LUIS, notamment l’ID d’application, la clé de création, la clé d’abonnement, le point de terminaison et la région dans le fichier `nlp-with-luis.bot`. Il s’agit des valeurs que vous avez enregistrées précédemment à partir de votre application LUIS publiée.

```json
{
    "name": "LuisBot",
    "description": "",
    "services": [
        {
            "type": "endpoint",
            "name": "development",
            "endpoint": "http://localhost:3978/api/messages",
            "appId": "",
            "appPassword": "",
            "id": "166"
        },
        {
            "type": "luis",
            "name": "LuisBot",
            "appId": "<luis appid>",
            "version": "0.1",
            "authoringKey": "<luis authoring key>",
            "subscriptionKey": "<luis subscription key>",
            "region": "<luis region>",
            "id": "158"
        }
    ],
    "padlock": "",
    "version": "2.0"
}
```
# <a name="ctabcs"></a>[C#](#tab/cs)

### <a name="configure-your-bot-to-use-your-luis-app"></a>Configurer votre bot pour qu’il utilise votre application LUIS

Ensuite, nous allons initialiser une nouvelle instance de la classe BotService en `BotServices.cs`, qui extrait les informations ci-dessus à partir de votre fichier `.bot`. Le service externe est configuré à l’aide de la classe `BotConfiguration`.

```csharp
public class BotServices
{
    // Initializes a new instance of the BotServices class
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

    // Gets the set of LUIS Services used. LuisServices is represented as a dictionary.  
    public Dictionary<string, LuisRecognizer> LuisServices { get; } = new Dictionary<string, LuisRecognizer>();
}
```

Enregistrez ensuite l’application LUIS en tant que singleton dans le fichier `Startup.cs` en utilisant le code suivant dans la méthode `ConfigureServices`.

```csharp
public void ConfigureServices(IServiceCollection services)
{
    var secretKey = Configuration.GetSection("botFileSecret")?.Value;
    var botFilePath = Configuration.GetSection("botFilePath")?.Value;

    // Loads .bot configuration file and adds a singleton that your Bot can access through dependency injection.
    var botConfig = BotConfiguration.Load(botFilePath ?? @".\nlp-with-luis.bot", secretKey);
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

        // ...
    });
}
```

Ensuite, dans le fichier `LuisBot.cs`, le bot obtient cette instance LUIS.

```csharp
public class LuisBot : IBot
{
    public static readonly string LuisKey = "LuisBot";
    private const string WelcomeText = "This bot will introduce you to natural language processing with LUIS. Type an utterance to get started";

    // Services configured from the ".bot" file.
    private readonly BotServices _services;

    // Initializes a new instance of the LuisBot class.
    public LuisBot(BotServices services)
    {
        _services = services ?? throw new System.ArgumentNullException(nameof(services));
        if (!_services.LuisServices.ContainsKey(LuisKey))
        {
            throw new System.ArgumentException($"Invalid configuration....");
        }
    }
    // ...
}
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

Dans notre exemple, le code de démarrage se trouve dans un fichier **index.js**, le code de la logique du bot se trouve dans un fichier **bot.js**, et les informations de configuration supplémentaires se trouvent dans le fichier **nlp-with-luis.bot**.

Dans le fichier **bot.js**, nous lisons les informations de configuration pour générer le service LUIS et initialiser le bot.
Remplacez la valeur de `LUIS_CONFIGURATION` par le nom de votre application LUIS, tel qu’il apparaît dans votre fichier de configuration.

```javascript
// Language Understanding (LUIS) service name as defined in the .bot file.YOUR_LUIS_APP_NAME is "LuisBot" in the JavaScript code.
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

### <a name="get-the-intent-by-calling-luis"></a>Obtenir l’intention en appelant LUIS

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

### <a name="test-the-bot"></a>Tester le bot

1. Exécutez l’exemple en local sur votre machine. Si vous avez besoin d’instructions, consultez le fichier Lisez-moi pour l’exemple [ C# ](https://github.com/Microsoft/BotBuilder-Samples/blob/master/samples/csharp_dotnetcore/12.nlp-with-luis/README.md) ou [JS](https://github.com/Microsoft/BotBuilder-Samples/blob/master/samples/javascript_nodejs/12.nlp-with-luis/README.md).

1. Dans l’émulateur, tapez un message, comme indiqué ci-dessous. 

![exemple d’entrée en langage naturel à des fins de test](./media/nlp-luis-sample-message.png)

Le bot répond avec l’intention ayant le score le plus élevé, « Calendar_Add » dans ce cas. N’oubliez pas que le fichier `reminders.json` importé dans le portail LUIS.AI a défini les intentions « Calendar_Add », « Calendar_Find » et « None ». 

![exemple de réponse en langage naturel à des fins de test](./media/nlp-luis-sample-response.png) 

Un score de prédiction indique le degré de confiance que LUIS a dans les résultats de prédiction. Un score de prédiction est compris entre zéro (0) et un (1). Un exemple de score LUIS de grande confiance est égal à 0,99. Un exemple de score faible confiance est égal à 0,01. 

## <a name="optional---extract-entities"></a>Facultatif - Extraire les entités

En plus de reconnaître l’intention de l’utilisateur, une application LUIS peut également retourner des entités. Les entités sont des mots importants liés à l’intention et peuvent parfois s’avérer essentielles pour répondre à la demande d’un utilisateur ou permettre à un bot de se comporter de manière plus intelligente. 

### <a name="why-use-entities"></a>Pourquoi utiliser des entités

Les entités LUIS permettent à votre bot de comprendre intelligemment certains éléments ou événements qui diffèrent des intentions standard. Vous pouvez ainsi collecter des informations supplémentaires auprès de l’utilisateur, ce qui permet à votre bot de répondre plus intelligemment ou même d’ignorer certaines questions visant à obtenir ces mêmes informations. Par exemple, dans un bot météo, l’application LUIS peut utiliser une entité de _localisation_ pour extraire la localisation du rapport météo demandé à partir du message de l’utilisateur. Votre bot n’est donc pas obligé de demander à l’utilisateur où il se trouve, ce qui permet d’avoir conversation plus pertinente et plus concise avec l’utilisateur.

### <a name="prerequisites"></a>Prérequis

Pour utiliser des entités avec cet exemple, vous devez créer une application LUIS qui inclut des entités. Suivez les étapes décrites dans la section ci-dessus pour [créer votre application LUIS](#create-a-luis-app-in-the-luis-portal), mais au lieu d’utiliser le fichier `reminders.json`, utilisez le fichier[reminders-with-entities.json](https://github.com/Microsoft/BotFramework-Samples/tree/master/SDKV4-Samples/dotnet_core/nlp-with-luis) pour générer votre application LUIS. Ce fichier fournit les mêmes intentions auxquelles s’ajoutent trois entités supplémentaires : Appointment, Meeting et Schedule. Ces entités aident LUIS à déterminer l’intention du message de l’utilisateur. 

### <a name="extract-and-display-entities"></a>Extraire et afficher des entités
Vous pouvez ajouter le code facultatif suivant à cet exemple d’application pour extraire les informations d’entité et les afficher quand une entité est utilisée par LUIS pour mieux identifier l’intention de l’utilisateur. 

# <a name="ctabcs"></a>[C#](#tab/cs)

La fonction d’assistance suivante peut être ajoutée à votre bot pour obtenir des entités de `RecognizerResult` à partir de LUIS. La bibliothèque `Newtonsoft.Json.Linq` doit être utilisée ; vous devez l’ajouter à vos instructions **using**. Si des informations d’entité sont trouvées durant l’analyse du code JSON retourné par LUIS, la fonction Newtonsoft _DeserializeObject_ convertit ce code JSON en objet dynamique, fournissant l’accès aux informations d’entité.

```cs
using Newtonsoft.Json.Linq;

private string ParseLuisForEntities(RecognizerResult recognizerResult)
{
   var result = string.Empty;

   // recognizerResult.Entities returns type JObject.
   foreach (var entity in recognizerResult.Entities)
   {
       // Parse JObject for a known entity types: Appointment, Meeting, and Schedule.
       var appointFound = JObject.Parse(entity.Value.ToString())["Appointment"];
       var meetingFound = JObject.Parse(entity.Value.ToString())["Meeting"];
       var schedFound = JObject.Parse(entity.Value.ToString())["Schedule"];

       // We will return info on the first entity found.
       if (appointFound != null)
       {
           // use JsonConvert to convert entity.Value to a dynamic object.
           dynamic o = JsonConvert.DeserializeObject<dynamic>(entity.Value.ToString());
           if (o.Appointment[0] != null)
           {
              // Find and return the entity type and score.
              var entType = o.Appointment[0].type;
              var entScore = o.Appointment[0].score;
              result = "Entity: " + entType + ", Score: " + entScore + ".";
              
              return result;
            }
        }

        if (meetingFound != null)
        {
            // use JsonConvert to convert entity.Value to a dynamic object.
            dynamic o = JsonConvert.DeserializeObject<dynamic>(entity.Value.ToString());
            if (o.Meeting[0] != null)
            {
                // Find and return the entity type and score.
                var entType = o.Meeting[0].type;
                var entScore = o.Meeting[0].score;
                result = "Entity: " + entType + ", Score: " + entScore + ".";
                
                return result;
            }
        }

        if (schedFound != null)
        {
            // use JsonConvert to convert entity.Value to a dynamic object.
            dynamic o = JsonConvert.DeserializeObject<dynamic>(entity.Value.ToString());
            if (o.Schedule[0] != null)
            {
                // Find and return the entity type and score.
                var entType = o.Schedule[0].type;
                var entScore = o.Schedule[0].score;
                result = "Entity: " + entType + ", Score: " + entScore + ".";
                
                return result;
            }
        }
    }

    // No entities were found, empty string returned.
    return result;
}
```

Ces informations d’entité détectées ainsi que l’intention identifiée de l’utilisateur peuvent ensuite être affichées. Pour afficher ces informations, ajoutez les quelques lignes de code suivantes dans la tâche _OnTurnAsync_ de l’exemple de code, juste après l’affichage des informations d’intention.

```cs
// See if LUIS found and used an entity to determine user intent.
var entityFound = ParseLuisForEntities(recognizerResult);

// Inform the user if LUIS used an entity.
if (entityFound.ToString() != string.Empty)
{
   await turnContext.SendActivityAsync($"==>LUIS Entity Found: {entityFound}\n");
}
else
{
   await turnContext.SendActivityAsync($"==>No LUIS Entities Found.\n");
}
```
# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

Vous pouvez ajouter le code suivant à votre bot pour extraire des informations d’entité du résultat `luisRecognizer` retourné par LUIS. Au sein de l’opération de traitement `onTurn` de l’exemple de fichier de code bot.js, ajoutez la ligne suivante juste après la déclaration de la constante _topIntent_. Toute information d’entité retournée est alors capturée : 

```javascript
// Since the LuisRecognizer was configured to include the raw results, get returned entity data.
var entityData = results.luisResult.entities;

```

Pour montrer à l’utilisateur les informations d’entité retournées, ajoutez les lignes de code suivantes juste après l’appel _sendActivity_, lequel est utilisé dans l’exemple de code pour informer l’utilisateur qu’un topIntent a été trouvé.

```javascript
// See if LUIS found and used an entity to determine user intent.
if (entityData.length > 0)
{
   if ((entityData[0].type == "Appointment") || (entityData[0].type == "Meeting") || (entityData[0].type == "Schedule") )
   {
      // inform user if LUIS used an entity.
      await turnContext.sendActivity(`LUIS Entity Found: Entity: ${entityData[0].entity}, Score: ${entityData[0].score}.`);
   }
}
else{
       await turnContext.sendActivity(`No LUIS Entities Found.`);
}
```

Ce code vérifie d’abord si LUIS a retourné des informations d’entité dans le résultat retourné et, le cas échéant, affiche des informations concernant la première entité détectée.

---

### <a name="test-bot-with-entities"></a>Tester le bot avec des entités

1. Pour tester votre bot qui inclut des entités, exécutez l’exemple localement comme expliqué [ci-dessus](#test-the-bot).

1. Dans l’émulateur, entrez le message indiqué ci-dessous. 

![exemple d’entrée en langage naturel à des fins de test](./media/nlp-luis-sample-message.png)

Le bot répond désormais avec l’intention ayant le score le plus élevé, « Calendar_Add », et avec l’entité « Meetings » utilisée par LUIS pour déterminer l’intention de l’utilisateur.

![exemple de réponse en langage naturel à des fins de test](./media/nlp-luis-sample-entity-response.png) 

La détection d’entités peut contribuer à améliorer les performances globales de votre bot. Par exemple, la détection de l’entité « Meeting » (illustrée ci-dessus) peut permettre à votre application d’appeler une fonction spécialisée conçue pour créer une réunion dans le calendrier de l’utilisateur.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Utiliser QnA Maker pour répondre aux questions](./bot-builder-howto-qna.md)
