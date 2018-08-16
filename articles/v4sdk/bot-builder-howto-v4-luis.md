---
title: Utilisation de LUIS pour la compréhension langagière | Microsoft Docs
description: Découvrez comment utiliser LUIS pour la compréhension du langage naturel avec le SDK Bot Builder.
keywords: Language Understanding, LUIS, intention, module de reconnaissance, entités, middleware (intergiciel)
author: ivorb
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 4/30/17
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 9a04a73e8ac8716ec528586e069aa2d16d90e28f
ms.sourcegitcommit: f95702d27abbd242c902eeb218d55a72df56ce56
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/19/2018
ms.locfileid: "39300593"
---
# <a name="using-luis-for-language-understanding"></a>Utilisation de LUIS pour la compréhension langagière

La capacité à comprendre ce que veut dire votre utilisateur du point de vue de la conversation et du contexte peut être une tâche difficile, mais peut donner à votre bot un sens de la conversation plus naturel. Le service de compréhension langagière, appelé LUIS, vous permet justement de faire cela ; votre bot peut ainsi reconnaître l’intention des messages de l’utilisateur, prendre en charge un langage plus naturel de l’utilisateur et mieux diriger le flux de la conversation. Pour plus d’informations sur la façon dont LUIS s’intègre à un bot, consultez [Compréhension langagière pour les bots](./bot-builder-concept-LUIS.md). 

Cette rubrique vous montre comment configurer des bots simples qui utilisent LUIS pour reconnaître quelques intentions différentes.

## <a name="installing-packages"></a>Installation des packages

Tout d’abord, vérifiez que vous avez installé les packages nécessaires pour LUIS.

# <a name="ctabcs"></a>[C#](#tab/cs)

[Ajoutez une référence](https://docs.microsoft.com/en-us/nuget/tools/package-manager-ui) à la préversion 4 des packages NuGet suivants :


* `Microsoft.Bot.Builder.Ai.LUIS`

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

Vous pouvez ajouter une référence aux packages botbuilder et botbuilder-ai dans votre projet par le biais de npm :

* `npm install --save botbuilder@preview`
* `npm install --save botbuilder-ai@preview`

---


## <a name="set-up-middleware-to-use-your-luis-app"></a>Configurer des middlewares (intergiciels) pour utiliser votre application LUIS

Tout d’abord, configurez une _application LUIS_, service que vous créez sur le site [www.luis.ai](https://www.luis.ai). Cette application LUIS peut être formée pour certaines intentions qu’elle doit être capable de reconnaître. Vous trouverez plus d’informations sur la création de votre application LUIS sur le [site LUIS](https://www.luis.ai).

Pour cet exemple, vous utiliserez seulement une démonstration de l’application LUIS capable de reconnaître les intentions Aide, Annuler et Météo ; l’ID d’application se trouve déjà dans l’exemple de code. Vous devrez disposer d’une clé Cognitive Services que vous pouvez obtenir en vous connectant au site [www.luis.ai](https://www.luis.ai) et en copiant la clé à partir de **User settings** (Paramètres utilisateur) > **Authoring Key** (Clé de création).

> [!NOTE] 
> Pour créer votre propre copie de l’application LUIS publique utilisée dans cet exemple, copiez le fichier LUIS [FirstSimpleBotExample.json](https://github.com/Microsoft/LUIS-Samples/blob/master/examples/simple-bot-example/FirstSimpleBotExample.json). Ensuite, [importez l’application LUIS](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/create-new-app#import-new-app), [formez-la](https://docs.microsoft.com/en-us/azure/cognitive-services/LUIS/luis-how-to-train) et [publiez-la](https://docs.microsoft.com/en-us/azure/cognitive-services/LUIS/publishapp). Remplacez l’ID d’application publique dans l’exemple de code par l’ID d’application de votre nouvelle application LUIS.

Configurez votre bot afin d’appeler votre application LUIS pour chaque message reçu d’un utilisateur, en l’ajoutant simplement à la pile de middlewares de votre bot. Les middlewares stockent les résultats de reconnaissance sur l’objet de contexte, ce qui les rend accessibles à la logique de votre bot.

# <a name="ctabcs"></a>[C#](#tab/cs)
Commencez avec le modèle d’echo bot, puis ouvrez **Startup.cs**. 

Ajoutez une instruction `using` pour `Microsoft.Bot.Builder.Ai.LUIS`.

```csharp
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Hosting;
using Microsoft.Bot.Builder.BotFramework;
using Microsoft.Bot.Builder.Integration.AspNet.Core;
// add this
using Microsoft.Bot.Builder.Ai.LUIS;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Logging;
using System;
```

Mettez à jour la méthode `ConfigureServices` dans votre fichier `Startup.cs` pour ajouter un objet `LuisRecognizerMiddleware` qui se connecte à votre application LUIS. 

```csharp
// This method gets called by the runtime. Use this method to add services to the container.
public void ConfigureServices(IServiceCollection services)
{
    services.AddBot<EchoBot>(options =>
    {
        options.CredentialProvider = new ConfigurationCredentialProvider(Configuration);
. 
        options.Middleware.Add(new CatchExceptionMiddleware<Exception>(async (context, exception) =>
        {
            await context.TraceActivity("EchoBot Exception", exception);
            await context.SendActivity("Sorry, it looks like something went wrong!");
        }));

        // The Memory Storage used here is for local bot debugging only. When the bot
        // is restarted, anything stored in memory will be gone. 
        IStorage dataStore = new MemoryStorage();

        options.Middleware.Add(new ConversationState<EchoState>(dataStore));

        // Add LUIS recognizer as middleware
        options.Middleware.Add(
            new LuisRecognizerMiddleware(
                new LuisModel(
                    // This appID is for a public app that's made available for demo purposes
                    "eb0bf5e0-b468-421b-9375-fdfb644c512e",
                    // You can use it by replacing <subscriptionKey> with your Authoring Key
                    // which you can find at https://www.luis.ai under User settings > Authoring Key
                    "<subscriptionKey>",
                    // The location-based URL begins with "https://<region>.api.cognitive.microsoft.com", where region is the region associated with the key you are using. Some examples of regions are `westus`, `westcentralus`, `eastus2`, and `southeastasia`.
                    new Uri("https://westus.api.cognitive.microsoft.com/luis/v2.0/apps/"))));
    });
}
```

<!--TODO: DOES PUBLIC APP WORK WITH KEYS IN DIFFERENT REGIIONS? --> Collez votre clé d’abonnement à partir de [https://www.luis.ai](https://www.luis.ai) à la place de `<subscriptionKey>`.

> [!NOTE] 
> Si vous utilisez votre propre application LUIS au lieu de l’application LUIS publique, vous pouvez obtenir l’ID, la clé d’abonnement et l’URL correspondants sur [https://www.luis.ai](https://www.luis.ai). 
>
>Pour obtenir l’URL de base à utiliser dans votre `LuisModel`, connectez-vous au [site LUIS](https://www.luis.ai), accédez à l’onglet **Publish** (Publier) et examinez la colonne **Endpoint** (Point de terminaison) sous **Resources and Keys** (Ressources et les clés). L’URL de base est la partie de **l’URL du point de terminaison** située avant l’ID d’abonnement et les autres paramètres.

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

Tout d’abord, demandez/importez la classe [LuisRecognizer](https://github.com/Microsoft/botbuilder-js/tree/master/doc/botbuilder-ai/classes/botbuilder_ai.luisrecognizer.md) et créez une instance pour votre modèle LUIS :

```javascript
const { LuisRecognizer } = require('botbuilder-ai');

const model = new LuisRecognizer({
    // This appID is for a public app that's made available for demo purposes
    // You can use it by providing your LUIS subscription key
     appId: 'eb0bf5e0-b468-421b-9375-fdfb644c512e',
    // replace subscriptionKey with your Authoring Key
    // your key is at https://www.luis.ai under User settings > Authoring Key 
    subscriptionKey: '<your subscription key>',
    // The serviceEndpoint URL begins with "https://<region>.api.cognitive.microsoft.com", where region is the region associated with the key you are using. Some examples of regions are `westus`, `westcentralus`, `eastus2`, and `southeastasia`.
    serviceEndpoint: 'https://westus.api.cognitive.microsoft.com'
});
```

Ajoutez le modèle à la pile de middlewares :

```javascript
adapter.use(model);
```


> [!NOTE] 
> Si vous utilisez votre propre application LUIS au lieu de l’application LUIS publique, vous pouvez obtenir l’ID, l’ID d’abonnement et l’URL correspondants sur [https://www.luis.ai](https://www.luis.ai). 

---



La compréhension langagière LUIS est maintenant intégrée à votre bot. À présent, examinons comment obtenir l’intention à partir du modèle LUIS stocké sur l’objet de contexte.


## <a name="get-the-intent-from-the-turn-context"></a>Obtenir l’intention à partir du contexte de tour

Les résultats de LUIS sont ensuite accessibles à partir de votre bot à l’aide du contexte sur chaque tour conversationnel.

# <a name="ctabcs"></a>[C#](#tab/cs)

Pour que votre bot envoie simplement une réponse basée sur l’intention que l’application LUIS a détectée, remplacez le code dans `OnTurn` par le code suivant :

```cs
using System.Threading.Tasks;
using Microsoft.Bot;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Ai.LUIS;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Schema;

namespace Bot_Builder_Echo_Bot1
{
    public class EchoBot : IBot
    {
        /// <summary>
        /// Every Conversation turn for our EchoBot will call this method. In here
        /// the bot checks the Activty type to verify it's a message, checks the /// intent from the LUIS recognizer, and sends a reply based on the recognized intent
        /// </summary>
        /// <param name="context">Turn-scoped context containing all the data needed
        /// for processing this conversation turn. </param>        
        public async Task OnTurn(ITurnContext context)
        {
            // This bot is only handling Messages
            if (context.Activity.Type == ActivityTypes.Message)
            {

                var result = context.Services.Get<RecognizerResult>
                    (LuisRecognizerMiddleware.LuisRecognizerResultKey);
                var topIntent = result?.GetTopScoringIntent();
                switch ((topIntent != null) ? topIntent.Value.intent : null)
                {
                    case null:
                        await context.SendActivity("Failed to get results from LUIS.");
                        break;
                    case "None":
                        await context.SendActivity("Sorry, I don't understand.");
                        break;
                    case "Help":
                        await context.SendActivity("<here's some help>");
                        break;
                    case "Cancel":
                        // Cancel the process.
                        await context.SendActivity("<cancelling the process>");
                        break;
                    case "Weather":
                        // Cancel the process.
                        await context.SendActivity("The weather today is sunny.");
                        break;
                    default:
                        // Received an intent we didn't expect, so send its name and score.
                        await context.SendActivity($"Intent: {topIntent.Value.intent} ({topIntent.Value.score}).");
                        break;
                }
            }
        }
    }    
}

```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

```javascript
// Listen for incoming activity 
server.post('/api/messages', (req, res) => {
    // Route received activity to adapter for processing
    adapter.processActivity(req, res, async (context) => {
        if (context.activity.type === 'message') {
            const results = model.get(context);
            const topIntent = LuisRecognizer.topIntent(results);
            switch (topIntent) {

                case 'Cancel':
                    await context.sendActivity("<cancelling the process>")
                    break;
                case 'Help':
                    await context.sendActivity("<here's some help>");
                    break;
                case 'Weather':
                    await context.sendActivity("The weather today is sunny.");
                    break;
                case 'None':                    
                    await context.sendActivity("Sorry, I don't understand.")
                    break;
                case 'null':                    
                    await context.sendActivity("Failed to get results from LUIS.")
                    break;
                default:
                    // Received an intent we didn't expect, so send its name and score.
                    await context.sendActivity(`The top intent was ${topIntent}`);
            }
        }
    });
});
```

Les intentions reconnues dans l’énoncé sont retournées sous la forme d’un mappage des noms d’intention avec des scores et sont accessibles à partir `results.intents`. Une méthode `LuisRecognizer.topIntent()` statique est mise à votre disposition, qui facilite la recherche de l’intention ayant reçu le meilleur score pour un jeu de résultats.
Toutes les entités reconnues sont retournées sous la forme d’un mappage de noms d’entité avec des valeurs et sont accessibles par le biais de `results.entities`. Vous pouvez retourner des métadonnées d’entité supplémentaires en passant un paramètre `verbose=true` quand vous créez l’objet LuisRecognizer. Les métadonnées ajoutées sont ensuite accessibles par le biais de `results.entities.$instance`.

---

<!-- TODO: SHOW RUNNING THE FIRST BOT --> Essayez d’exécuter le bot dans l’émulateur Bot Framework, et dites-lui quelque chose comme « météo », « aide » et « annuler ».

![Exécuter le bot](./media/how-to-luis/run-luis-bot.png)


## <a name="using-a-luis-recognizer-with-conversation-state"></a>Utilisation d’un module de reconnaissance LUIS avec l’état de la conversation
<!-- TBD, complete example --> Si votre réponse à l’utilisateur implique plus d’un tour, vous pouvez décider de suivre votre avancement dans la conversation en enregistrant l’état de la conversation. Vous pouvez utiliser l’intention à partir de LUIS pour vous aider à définir les données de l’état de la conversation, telles que le fait qu’un sujet de conversation a été démarré ou clos.

Les middlewares du module de reconnaissance LUIS s’exécutent pour chaque tour de votre bot ; vous pouvez ainsi obtenir une intention pour chaque message reçu par votre utilisateur. Si vous souhaitez démarrer un flux de conversation à plusieurs tours basé sur une intention, vous pouvez ignorer la logique de changement de sujets jusqu’à ce que vous ayez totalement traité le sujet actuel.

# <a name="ctabcs"></a>[C#](#tab/cs)

Dans EchoState.cs, changez EchoState comme suit :

```csharp
public class ConversationStateInfo 
{
    public bool WeatherTopicStarted  { get; set; }
    public bool HelpTopicStarted  { get; set; }
    public bool CancelTopicStarted  { get; set; }
}
```

Dans Startup.cs, changez l’initialisation de ConversationState de manière à utiliser `ConversationStateInfo`.
```cs
    options.Middleware.Add(new ConversationState<ConversationStateInfo>(dataStore));
```

Dans EchoBot.cs, modifiez `OnTurn` :
```cs
public async Task OnTurn(ITurnContext context)
{
    // This bot is only handling Messages
    if (context.Activity.Type == ActivityTypes.Message)
    {
        var text = context.Activity.Text;
        var conversationState = context.GetConversationState<ConversationStateInfo>() ?? new ConversationStateInfo();

        // Here, you can add some other logic based on the topic flags in conversation state
        // For example, if you know that a particular topic was started in a previous turn,
        // you can send the reply for that topic and bypass getting the intent from LUIS 
        if (conversationState.WeatherTopicStarted)
        {
            // Set this flag to false since this reply concludes the topic.
            conversationState.WeatherTopicStarted = false;
            // Assume that they responded to the prompt with a location.
            await context.SendActivity($"The weather in {text} is sunny.");
        }
        else
        {
            var result = context.Services.Get<RecognizerResult>
            (LuisRecognizerMiddleware.LuisRecognizerResultKey);
            var topIntent = result?.GetTopScoringIntent();
            switch ((topIntent != null) ? topIntent.Value.intent : null)
            {
                case null:
                    await context.SendActivity("Failed to get results from LUIS.");
                    break;
                case "None":
                    await context.SendActivity("Sorry, I don't understand.");
                    break;
                case "Weather":
                    conversationState.WeatherTopicStarted = true;
                    await context.SendActivity($"Looks like you want a weather forecast. What city do you want the forecast for?");

                    break;
                case "Help":
                    conversationState.HelpTopicStarted = true;
                    await context.SendActivity("<here's some help>");
                    break;
                case "Cancel":
                    // Cancel the process.
                    conversationState.CancelTopicStarted = true;
                    await context.SendActivity("<cancelling the process>");
                    break;
                default:
                    // Received an intent we didn't expect, so send its name and score.
                    await context.SendActivity($"Intent: {topIntent.Value.intent} ({topIntent.Value.score}).");
                    break;
            }
        }
    }
}
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

Ajoutez les middlewares d’état de la conversation après avoir ajouté `LuisRecognizer`.

```javascript
const model = new LuisRecognizer({
    // This appID is for a public app that's made available for demo purposes
    // You can use it by providing your LUIS subscription key
     appId: 'eb0bf5e0-b468-421b-9375-fdfb644c512e',
    // replace subscriptionKey with your Authoring Key
    // your key is at https://www.luis.ai under User settings > Authoring Key 
    subscriptionKey: '<your subscription>',
    // The serviceEndpoint URL begins with "https://<region>.api.cognitive.microsoft.com", where region is the region associated with the key you are using. Some examples of regions are `westus`, `westcentralus`, `eastus2`, and `southeastasia`.
    serviceEndpoint: 'https://westus.api.cognitive.microsoft.com'
});
adapter.use(model)

// Add conversation state middleware
const conversationState = new ConversationState(new MemoryStorage());
adapter.use(conversationState);
```

Vous pouvez ensuite enregistrer l’état indiquant le sujet qui a été démarré.

```javascript
// Listen for incoming activities 
server.post('/api/messages', (req, res) => {
    // Route received activity to adapter for processing
    adapter.processActivity(req, res, async(context) => {
        if (context.activity.type === 'message') {
            var utterance = context.activity.text;
            
            // Check topic flags in conversation state 
            if (conversationState.weatherTopicStarted) 
            {
                // Assume the user's message is a reply to the bot's prompt for a location
                await context.sendActivity(`The weather in ${utterance} is sunny.`);
                // This conversation flow is now finished. Set flag to false,
                // so that on the next turn the user can ask for another weather forecast.
                conversationState.WeatherTopicStarted = false;
            }
            // To add more steps to the other topics
            // you could check the topic flags here
            else 
            {
                const results = model.get(context);
                const topIntent = LuisRecognizer.topIntent(results);
                switch (topIntent) {
                    case 'None':
                        //Add app logic when there is no result
                        await context.sendActivity("<null case>")
                        break;
                    case 'Cancel':
                        conversationState.cancelTopicStarted = true;
                        await context.sendActivity("<cancelling the process>")
                        break;
                    case 'Help':
                        conversationState.helpTopicStarted = true;
                        await context.sendActivity("<here's some help>");
                        break;
                    case 'Weather':
                        conversationState.weatherTopicStarted = true;
                        await context.sendActivity("Looks like you want a weather forecast. What city do you want the forecast for?");
                        break;
                    default:
                        // Add app logic for the recognition results.
                        await context.sendActivity(`Received this intent: ${topIntent}`);
                }
            }
        }
    });
});
```

---

Essayez d’exécuter le bot dans l’émulateur Framework Bot ; vous pouvez constater que « obtenir la météo » est désormais un flux de conversation à deux tours.

![Exécuter le bot](./media/how-to-luis/run-luis-bot-2step-weather.png)


## <a name="using-luis-with-dialogs"></a>Utilisation de LUIS avec des dialogues

Si vous utilisez l’intention à partir de LUIS pour déclencher un flux de conversation à plusieurs tours, il peut être utile de recourir à des dialogues pour encapsuler ce flux. Cet exemple de bot utilise une application LUIS qui détecte des intentions permettant de déclencher un dialogue pour une application domotique ou de renseignements météorologiques.

> [!NOTE] 
> Pour créer votre propre copie de l’application LUIS publique utilisée dans cet exemple, copiez le fichier LUIS [WeatherOrHomeAutomation.json](https://github.com/Microsoft/LUIS-Samples/blob/master/examples/simple-bot-example/WeatherOrHomeAutomation.json). Ensuite, [importez l’application LUIS](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/create-new-app#import-new-app), [formez-la](https://docs.microsoft.com/en-us/azure/cognitive-services/LUIS/luis-how-to-train) et [publiez-la](https://docs.microsoft.com/en-us/azure/cognitive-services/LUIS/publishapp). Remplacez l’ID d’application publique dans l’exemple de code par l’ID d’application de votre nouvelle application LUIS.

# <a name="ctabcs"></a>[C#](#tab/cs)

Tout d’abord, modifiez `ConfigureServices` dans Startup.cs pour ajouter les middlewares pour votre application LUIS. Renommez `EchoBot` en `LuisDialogBot`.
Dans cet exemple, l’application LUIS ajoutée en tant que `LuisRecognizerMiddleware` détecte les intentions `homeautomation` ou `weather`, pour déclencher des dialogues avec ces noms. 

```csharp
public void ConfigureServices(IServiceCollection services)
{  
    // Rename EchoBot to LuisDialogBot
    services.AddBot<LuisDialogBot>(options =>
    {
        options.CredentialProvider = new ConfigurationCredentialProvider(Configuration); 
        options.Middleware.Add(new CatchExceptionMiddleware<Exception>(async (context, exception) =>
        {
            await context.TraceActivity("EchoBot Exception", exception);
            await context.SendActivity("Sorry, it looks like something went wrong!");
        }));

        // The Memory Storage used here is for local bot debugging only. When the bot
        // is restarted, anything stored in memory will be gone. 
        IStorage dataStore = new MemoryStorage();

        // Use Dictionary<string, object> for the conversation state type
        options.Middleware.Add(new ConversationState<Dictionary<string, object>>(dataStore));

        // Add LUIS recognizer as middleware
        options.Middleware.Add(
            new LuisRecognizerMiddleware(
                new LuisModel(
                    // This appID is for a public app that's made available for demo purposes
                    "428affb6-7650-46ac-9184-68c00a4f1729",
                    // You can use it by replacing <subscriptionKey> with your Authoring Key
                    // which you can find at https://www.luis.ai under User settings > Authoring Key
                    "<subscriptionKey>",
                    // The location-based URL begins with "https://<region>.api.cognitive.microsoft.com", where region is the region associated with the key you are using. Some examples of regions are `westus`, `westcentralus`, `eastus2`, and `southeastasia`.
                    new Uri("https://westus.api.cognitive.microsoft.com/luis/v2.0/apps/"))));
    });
}
```

Renommez **EchoBot.cs** en **LuisDialogBot.cs**, et la classe `EchoBot` en `LuisDialogBot`. Ensuite, dans **LuisDialogBot.cs**, ajoutez les instructions `using` suivantes.

```csharp
using System;
using System.Collections.Generic;
using System.Threading.Tasks;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Ai.LUIS;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Bot.Builder.Prompts;
using Microsoft.Bot.Schema;
```

Dans **LuisDialogBot.cs**, ajoutez le code suivant à la classe `LuisDialogBot`.

```csharp
    public class LuisDialogBot : IBot
    {
        private DialogSet _dialogs;

        public LuisDialogBot()
        {
            _dialogs = new DialogSet();

            _dialogs.Add("homeautomation", CreateHomeAutomationWaterfall());
            _dialogs.Add("weather", CreateWeatherWaterfall());
            _dialogs.Add("weather_city", new Builder.Dialogs.TextPrompt());
        }

        // App ID for a separate LUIS app used to tell if the user wants to turn the lights on or off
        // The `homeautomation` dialogs uses results from this app to determine the "on/off" argument. 
        private static LuisModel luisHomeAutomation =
            new LuisModel("76feb726-515b-44c4-acc9-adb216965a58", "SUBSCRIPTION-KEY", new System.Uri("https://westus.api.cognitive.microsoft.com/luis/v2.0/apps/"));

        public async Task OnTurn(ITurnContext context)
        {
            // This bot is only handling Messages
            if (context.Activity.Type == ActivityTypes.Message)
            {

                // Create a dialog context
                var state = ConversationState<Dictionary<string, object>>.Get(context);
                var dc = _dialogs.CreateContext(context, state);

                // Run the next dialog step
                await dc.Continue();
                
                // Check if any dialog has responded on this turn
                if (!context.Responded)
                {

                    var luisResult = context.Services.Get<RecognizerResult>(LuisRecognizerMiddleware.LuisRecognizerResultKey);
                    var topIntent = luisResult?.GetTopScoringIntent();
                    var utterance = context.Activity.Text;
                    var dialogArgs = new Dictionary<string, object>();

                    switch ((topIntent != null) ? topIntent.Value.intent.ToLowerInvariant() : null)
                    {
                        case "homeautomation":
                            // The Homeautomation_TurnOn and Homeautomation_TurnOff dialogs 
                            // use results from a separate LUIS app.
                            // The results determine the "on/off" argument to pass to the dialog. 
                            var recognizerHomeAutomation = new LuisRecognizer(luisHomeAutomation);
                            RecognizerResult recognizerResult = await recognizerHomeAutomation.Recognize(utterance, System.Threading.CancellationToken.None);
                            var topHomeAutoIntent = recognizerResult.GetTopScoringIntent().intent;


                            dialogArgs.Add("IntentFromHomeAuto", "");
                            switch ((topHomeAutoIntent != null) ? topHomeAutoIntent.ToLowerInvariant() : null)
                            {
                                case "homeautomation_turnon":
                                    dialogArgs["Intent_HomeAutomation"] = "on";
                                    await dc.Begin("homeautomation", dialogArgs);
                                    break;
                                case "homeautomation_turnoff":
                                    dialogArgs["Intent_HomeAutomation"] = "off";
                                    await dc.Begin("homeautomation", dialogArgs);
                                    break;
                                case null:
                                    await dc.Begin("homeautomation", null);
                                    break;
                                default:
                                    dialogArgs["Intent_HomeAutomation"] = topHomeAutoIntent;
                                    await dc.Begin("homeautomation", dialogArgs);
                                    break;
                            }
                            break;
                        case "weather":
                            dialogArgs.Add("LuisResult", luisResult);
                            await dc.Begin("weather", dialogArgs);
                            break;
                        case null:
                            await context.SendActivity($"Couldn't get a result from LUIS. You said: {utterance}");
                            break;
                        default:
                            // The intent didn't match any case, so just display the recognition results.
                            await context.SendActivity($"you said: {utterance}");
                            await context.SendActivity($"Recognized intent: {topIntent.Value.intent}.");

                            break;

                    }

                }
            }
        }

    }


```

Après `OnTurn`, ajoutez du code au sein de la classe `LuisDialogBot` pour implémenter les dialogues.

```csharp
        // The home automation waterfall has one step
        private WaterfallStep[] CreateHomeAutomationWaterfall()
        {
            return new WaterfallStep[] {
                TurnLightsOnOrOff
            };
        }

        // The weather waterfall has two steps
        private WaterfallStep[] CreateWeatherWaterfall()
        {
            return new WaterfallStep[] {
                AskWeatherLocation,
                SendWeatherReport
            };
        }

        /// <summary>
        /// This is the first step and only of the home automation dialog.
        /// </summary>
        /// <param name="dc"></param>
        /// <param name="args">Can be "on", "off", another string for an intent name, or null.
        /// null indicates that no result was received from which to get an intent name.</param>
        /// <param name="next"></param>
        /// <returns></returns>
        private async Task TurnLightsOnOrOff(DialogContext dc, IDictionary<string, object> args, SkipStepFunction next)
        {
            var intentFromHomeAutomation = args["Intent_HomeAutomation"];
            if (args != null)
            {
                switch (intentFromHomeAutomation)
                {
                    case "on":
                    case "off":
                        await dc.Context.SendActivity($"Turning {intentFromHomeAutomation} your lights!");
                        break;
                    default:
                        await dc.Context.SendActivity($"Intent detected by homeautomation was: {intentFromHomeAutomation}, but the home automation system doesn't support that yet.");
                        break;
                }

            }
            else
            {
                await dc.Context.SendActivity($"You said {dc.Context.Activity.AsMessageActivity().Text}. Unable to get a result from which to determine on/off/other operation");
            }

            await dc.End();

        }

        // This is the first step of the weather dialog
        private async Task AskWeatherLocation(DialogContext dc, IDictionary<string, object> args, SkipStepFunction next)
        {
            var dialogState = dc.ActiveDialog.State as IDictionary<string, object>;
            
            if (args["LuisResult"] is RecognizerResult luisResult)
            {
                var location = GetEntity<string>(luisResult, "Weather_Location");
                if (!string.IsNullOrEmpty(location))
                {
                    dialogState.Add("Location", location);
                }
            }

            // Save info back to the dialog instance
            dc.ActiveDialog.State = dialogState;

            if (!dialogState.ContainsKey("Location"))
            {
                await dc.Prompt("weather_city", "What city do you want the weather for?");
            }
            else
            {
                // We've set the location parameter for the weather report,
                // so go to the next step in the waterfall
                await dc.Continue();
            }
        }

        // This the second step of the weather dialog
        private async Task SendWeatherReport(DialogContext dc, IDictionary<string, object> args, SkipStepFunction next)
        {
            var dialogState = dc.ActiveDialog.State as IDictionary<string, object>;
            if (args != null)
            {
                if (!dialogState.ContainsKey("Location"))
                {   // Interpret args as a reply to prompt only if they didn't give location
                    TextResult locationResult = (TextResult)args;
                    dialogState.Add("Location", locationResult.Text);
                }

            }

            // You can add some logic that uses the location 
            // to get the weather from a weather service instead of hard-coding it
            await dc.Context.SendActivity($"The weather forecast for '{dialogState["Location"]}' is sunny and 70 degrees F.");

            dc.ActiveDialog.State = new Dictionary<string, object>(); // clear the dialog state 
            await dc.End();
        }
```

Ajoutez cette fonction d’assistance.

```cs
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

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

Vous devrez peut-être installer la bibliothèque de dialogues si ce n’est déjà fait. 

```cmd
npm install --save botbuilder-dialogs
```

Tout d’abord, créez une application LUIS et ajoutez-la à votre bot à l’aide de `adapter.use` :

```javascript
const { BotFrameworkAdapter, ConversationState, MemoryStorage, TurnContext } = require('botbuilder');
const { LuisRecognizer } = require('botbuilder-ai');
const { DialogSet } = require('botbuilder-dialogs');
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

// Create LuisRecognizer 
// The LUIS application is public, meaning you can use your own subscription key to test the applications.
const luisRecognizer = new LuisRecognizer({
    appId: '1fefd4a7-ae5b-4e63-99f7-0cf499a1423b',
    subscriptionKey: process.env.LUIS_SUBSCRIPTION_KEY,
    serviceEndpoint: 'https://westus.api.cognitive.microsoft.com/',
    verbose: true
});

// Add the recognizer to your bot
adapter.use(luisRecognizer);

// create conversation state
const conversationState = new ConversationState(new MemoryStorage());
adapter.use(conversationState);

// register some dialogs for usage with the intents detected by the LUIS app
const dialogs = new DialogSet();
```

Ensuite, ajoutez les dialogues :

```javascript

dialogs.add('HomeAutomation_TurnOn', [
    async (dialogContext) => {
        const state = conversationState.get(dialogContext.context);
        // state.homeAutomationTurnOn counts how many times this dialog was called 
        state.homeAutomationTurnOn = state.homeAutomationTurnOn ? state.homeAutomationTurnOn + 1 : 1;
        await dialogContext.context.sendActivity(`${state.homeAutomationTurnOn}: You reached the "HomeAutomation_TurnOn" dialog.`);

        await dialogContext.end();
    }
]);

dialogs.add('Weather_GetForecast', [
    async (dialogContext) => {
        const state = conversationState.get(dialogContext.context);
        // state.weatherGetForecast counts how many times this dialog was called  
        state.weatherGetForecast = state.weatherGetForecast ? state.weatherGetForecast + 1 : 1;
        await dialogContext.context.sendActivity(`${state.weatherGetForecast}: You reached the "Weather_GetForecast" dialog.`);

        await dialogContext.end();
    }
]);

dialogs.add('None', [
    async (dialogContext) => {
        const state = conversationState.get(dialogContext.context);
        // state.None counts how many times this dialog was called        
        state.None = state.None ? state.None + 1 : 1;
        await dialogContext.context.sendActivity(`${state.None}: You reached the "None" dialog.`);

        await dialogContext.end();
    }
]);
```

Dans votre bot, appelez chaque dialogue en fonction de l’intention reconnue :
```javascript
server.post('/api/messages', (req, res) => {
    adapter.processActivity(req, res, async (context) => {
        if (context.activity.type === 'message') {
            const state = conversationState.get(context);
            const dc = dialogs.createContext(context, state);

            // Retrieve the LUIS results from our LUIS application
            const luisResults = luisRecognizer.get(context);

            // Extract the top intent from LUIS and use it to select which dialog to start
            // "NotFound" is the intent name for when no top intent can be found.
            const topIntent = LuisRecognizer.topIntent(luisResults, "NotFound");

            const isMessage = context.activity.type === 'message';
            if (isMessage) {
                switch (topIntent) {
                    case 'homeautomation':                    
                        await dc.begin("HomeAutomation_TurnOn", luisResults);
                        break;
                    case 'weather':                    
                        await dc.begin("Weather_GetForecast", luisResults);
                        break;
                    case 'None':
                        await dc.begin("None");
                        break;
                    case 'NotFound':
                        await context.sendActivity(`Sorry, I didn't get any results from LUIS.`);
                        break;
                    default:
                        // handle intents for which we have no dialog
                        await context.sendActivity(`You reached the ${topIntent} intent.`);
                        break;
                }
            }
            
            if (!context.responded) {
                await dc.continue();
                if (!context.responded && isMessage) {
                    await dc.context.sendActivity(`Hi! I'm the LUIS dialog bot. Say something and LUIS will decide how the message should be routed.`);
                }
            }
        }
    });
});
```

Essayez d’exécuter le bot dans l’émulateur Bot Framework, et dites-lui quelque chose comme « allumer la lumière » et « obtenir la météo de Seattle ».

![Exécuter le bot](./media/how-to-luis/run-luis-bot-js-single-step-dialog.png)

---

## <a name="extract-entities"></a>Extraire les entités

En plus de reconnaître une intention, une application LUIS peut extraire des entités qui sont des mots importants pour répondre à la demande d’un utilisateur. Ainsi, dans l’exemple de dialogue de renseignements météorologiques, l’application LUIS peut-être en mesure d’extraire du message de l’utilisateur le lieu dont il souhaite connaître le bulletin météorologique. 

Une façon courante d’utiliser des dialogues consiste à identifier les entités dans le message de l’utilisateur et à l’inviter à indiquer les entités requises introuvables. Ensuite, l’étape de dialogue suivante traite la réponse à l’invite.

# <a name="ctabcs"></a>[C#](#tab/cs)

Supposons que le message de l’utilisateur était « Quelle est la météo de Seattle » ? Pour le bot de cet exemple, [LuisRecognizer](https://docs.microsoft.com/en-us/dotnet/api/microsoft.bot.builder.ai.luis.luisrecognizer) vous donne un [RecognizerResult](https://docs.microsoft.com/en-us/dotnet/api/microsoft.bot.builder.core.extensions.recognizerresult) avec une [propriété `Entities`](https://docs.microsoft.com/en-us/dotnet/api/microsoft.bot.builder.core.extensions.recognizerresult#properties-) qui présente la structure suivante :

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

La fonction d’assistance suivante que vous avez ajoutée à votre classe `LuisDialogBot` obtient des entités de `RecognizerResult` à partir de LUIS.

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

Quand vous collectez des informations telles que des entités à partir de plusieurs étapes d’un dialogue, il peut être utile d’enregistrer les informations dont vous avez besoin dans un état propre au dialogue. Par exemple, dans le dialogue `AskWeatherLocation`, les entités sont extraites des résultats LUIS passés au dialogue. Si une entité `Weather_Location` est trouvée, elle est ajoutée à l’état du dialogue.

```cs

        // This is the first step of the weather dialog
        private async Task AskWeatherLocation(DialogContext dc, IDictionary<string, object> args, SkipStepFunction next)
        {
            var dialogState = dc.ActiveDialog.State as IDictionary<string, object>;
            
            if (args["LuisResult"] is RecognizerResult luisResult)
            {
                var location = GetEntity<string>(luisResult, "Weather_Location");
                if (!string.IsNullOrEmpty(location))
                {
                    dialogState.Add("Location", location);
                }
            }

            // Save info back to the dialog instance
            dc.ActiveDialog.State = dialogState;

            if (!dialogState.ContainsKey("Location"))
            {
                await dc.Prompt("weather_city", "What city do you want the weather for?");
            }
            else
            {
                // We've set the location parameter for the weather report,
                // so go to the next step in the waterfall
                await dc.Continue();
            }
        }
```

À la deuxième étape du dialogue, vous pouvez obtenir les informations enregistrées à l’étape précédente de l’état de l’instance de dialogue, ainsi que la réponse de l’utilisateur à l’invite relative au lieu, et les utiliser pour envoyer un bulletin météorologique à l’utilisateur.

```cs

// This the second step of the weather dialog
private async Task SendWeatherReport(DialogContext dc, IDictionary<string, object> args, SkipStepFunction next)
{
    var dialogState = dc.ActiveDialog.State as IDictionary<string, object>;
    if (args != null)
    {
        if (!dialogState.ContainsKey("Location"))
        {   // Interpret args as a reply to prompt only if they didn't give location
            TextResult locationResult = (TextResult)args;
            dialogState.Add("Location", locationResult.Text);
        }

    }

    // You can add some logic that uses the location and date
    // to get the weather from a weather service instead of hard-coding it
    await dc.Context.SendActivity($"The weather forecast for '{dialogState["Location"]}' is sunny and 70 degrees F.");
    
    // clear the dialog state 
    dc.ActiveDialog.State = new Dictionary<string, object>(); 
    await dc.End();
}
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

Par exemple, supposons que le message de l’utilisateur était « Quelle est la météo de Seattle » ? Pour le bot de cet exemple, [LuisRecognizer](https://docs.microsoft.com/en-us/javascript/api/botbuilder-ai/luisrecognizer) vous donne un [RecognizerResult](https://docs.microsoft.com/en-us/javascript/api/botbuilder-core-extensions/recognizerresult) avec une propriété `entities` qui présente la structure suivante :

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

Cette fonction `findEntities` recherche les entités `Weather_Location` reconnues par l’application LUIS.

<!-- TODO: Turn into a waterfall -->

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

Appelez `findEntities` à partir du dialogue `Weather_GetForecast`.

```javascript
// Pass the LUIS recognizer result to the args parameter
dialogs.add('Weather_GetForecast', [
    async (dialogContext, args) => {
        const locations = findEntities('Weather_Location', args.entities);

        const state = conversationState.get(dialogContext.context);
        state.weatherGetForecast = state.weatherGetForecast ? state.weatherGetForecast + 1 : 1;
        await dialogContext.context.sendActivity(`${state.weatherGetForecast}: You reached the "Weather.GetForecast" dialog.`);
        if (locations) {
            await dialogContext.context.sendActivity(`Found these "Weather_Location" entities:\n${locations.join(', ')}`);
        }
        await dialogContext.end();
    }
]);
```

Les entités sont extraites des résultats LUIS passés à `dc.begin`.

```javascript
await dc.begin("Weather_GetForecast", luisResults);
```
---

## <a name="additional-resources"></a>Ressources supplémentaires

Pour plus de contexte sur LUIS, voir [Language Understanding](./bot-builder-concept-luis.md).

Pour plus d’informations sur la façon de générer les applications LUIS utilisées dans ces exemples, consultez [Applications LUIS pour Bot Builder](https://aka.ms/luis-bot-examples).

Vous pouvez combiner LUIS à d’autres services Cognitive Services pour rendre votre bot encore plus puissant. Consultez [Combiner des applications LUIS et des services de questions/réponses à l’aide de l’outil Dispatch](./bot-builder-tutorial-dispatch.md) pour apprendre à combiner un service de questions/réponses à Language Understanding (LUIS) dans votre bot.

## <a name="next-steps"></a>Étapes suivantes


> [!div class="nextstepaction"]
> [Générer des résultats LUIS fortement typés](./bot-builder-howto-v4-luisgen.md)


