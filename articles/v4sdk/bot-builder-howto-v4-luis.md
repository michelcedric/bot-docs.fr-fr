---
title: Utilisation de LUIS pour la compréhension langagière | Microsoft Docs
description: Découvrez comment utiliser LUIS pour la compréhension du langage naturel avec le SDK Bot Builder.
keywords: Language Understanding, LUIS, intention, module de reconnaissance, entités, middleware (intergiciel)
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 09/19/18
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: c4a7cfba6f588c95dbebf4886ffd7e432d99c3ae
ms.sourcegitcommit: 3cb288cf2f09eaede317e1bc8d6255becf1aec61
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/27/2018
ms.locfileid: "47389798"
---
# <a name="using-luis-for-language-understanding"></a>Utilisation de LUIS pour la compréhension langagière

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

La capacité à comprendre ce que veut dire votre utilisateur du point de vue de la conversation et du contexte peut être une tâche difficile, mais peut donner à votre bot un sens de la conversation plus naturel. Le service de compréhension langagière, appelé LUIS, vous permet justement de faire cela ; votre bot peut ainsi reconnaître l’intention des messages de l’utilisateur, prendre en charge un langage plus naturel de l’utilisateur et mieux diriger le flux de la conversation. Pour plus d’informations sur la façon dont LUIS s’intègre à un bot, consultez [Compréhension langagière pour les bots](./bot-builder-concept-LUIS.md). 

Cette rubrique vous montre comment configurer des bots simples qui utilisent LUIS pour reconnaître quelques intentions différentes.

## <a name="installing-packages"></a>Installation des packages

Tout d’abord, vérifiez que vous avez installé les packages nécessaires pour LUIS.

# <a name="ctabcs"></a>[C#](#tab/cs)

[Ajoutez une référence](https://docs.microsoft.com/en-us/nuget/tools/package-manager-ui) à la version 4 des packages NuGet suivants :


* `Microsoft.Bot.Builder.AI.LUIS`

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

Installez les packages botbuilder et botbuilder-ai dans votre projet à l’aide de npm :

* `npm install --save botbuilder`
* `npm install --save botbuilder-ai`

---

## <a name="set-up-your-luis-app"></a>Configurer votre application LUIS

Tout d’abord, configurez une _application LUIS_, qui est un service que vous créez à l’emplacement [luis.ai](https://www.luis.ai). Cette application LUIS peut être formée pour certaines intentions qu’elle doit être capable de reconnaître. Vous trouverez plus d’informations sur la création de votre application LUIS sur le site LUIS.

Pour cet exemple, vous utiliserez seulement une démonstration de l’application LUIS capable de reconnaître les intentions Aide, Annuler et Météo ; l’ID d’application se trouve déjà dans l’exemple de code. Vous devez disposer d’une clé Cognitive Services que vous pouvez obtenir en vous connectant à [luis.ai](https://www.luis.ai) et en copiant la clé à partir de **Paramètres utilisateur** > **Clé de création**.

> [!NOTE]
> Pour créer votre propre copie de l’application LUIS publique utilisée dans cet exemple, copiez le fichier LUIS [JSON](https://github.com/Microsoft/LUIS-Samples/blob/master/examples/simple-bot-example/FirstSimpleBotExample.json). Ensuite, [importez](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/create-new-app#import-new-app) l’application LUIS, [formez-la](https://docs.microsoft.com/en-us/azure/cognitive-services/LUIS/luis-how-to-train) et [publiez-la](https://docs.microsoft.com/en-us/azure/cognitive-services/LUIS/publishapp). Remplacez l’ID d’application publique dans l’exemple de code par l’ID d’application de votre nouvelle application LUIS.


### <a name="configure-your-bot-to-call-your-luis-app"></a>Configurez votre bot pour appeler votre application LUIS.

# <a name="ctabcs"></a>[C#](#tab/cs)

Bien qu’il soit possible de créer et d’appeler votre application LUIS à chaque tour, les meilleures pratiques de codage recommandent d’inscrire votre service LUIS en tant que singleton, puis de le transmettre sous forme de paramètres à votre constructeur de bot. Nous allons présenter cette méthode ici, car elle est légèrement plus complexe.

Commencez avec le modèle d’echo bot, puis ouvrez **Startup.cs**. 

Ajoutez une instruction `using` pour `Microsoft.Bot.Builder.AI.LUIS`.

```csharp
// add this
using Microsoft.Bot.Builder.AI.LUIS;
```

Ajoutez le code suivant à la fin de `ConfigureServices`, après l’initialisation de l’état. Cela extrait les informations du fichier `appsettings.json`, mais ces chaînes peuvent être extraites à partir de votre fichier `.bot`, comme dans l’exemple associé présent à la fin de cet article, ou codées en dur à des fins de tests.

Le singleton renvoie un nouvel élément `LuisRecognizer` au constructeur.

```csharp
    // Create and register a LUIS recognizer.
    services.AddSingleton(sp =>
    {
        // Get LUIS information from appsettings.json.
        var section = this.Configuration.GetSection("Luis");
        var luisApp = new LuisApplication(
            applicationId: section["AppId"],
            endpointKey: section["SubscriptionKey"],
            azureRegion: section["Region"]);

        // Specify LUIS options. These may vary for your bot.
        var luisPredictionOptions = new LuisPredictionOptions
        {
            IncludeAllIntents = true,
        };

        return new LuisRecognizer(
            application: luisApp,
            predictionOptions: luisPredictionOptions,
            includeApiResults: true);
    });
```

Collez votre clé d’abonnement à partir de [luis.ai](https://www.luis.ai) à la place de `<subscriptionKey>`. Pour la trouver plus facilement, vous pouvez cliquer sur le nom de votre compte en haut à droite et accéder à **Paramètres**, où elle est présentée sous le nom **Clé de création**.

> [!NOTE]
> Si vous utilisez votre propre application LUIS au lieu de l’application LUIS publique, vous pouvez obtenir l’ID, la clé d’abonnement et l’URL correspondants à partir de [luis.ai](https://www.luis.ai). Vous pouvez trouver ces derniers sous les onglets **Publier** et **Paramètres** de la page de votre application.
>
>Pour obtenir l’URL de base à utiliser dans votre `LuisModel`, connectez-vous à [luis.ai](https://www.luis.ai), accédez à l’onglet **Publier** et examinez la colonne **Point de terminaison** sous **Resources and Keys** (Ressources et clés). L’URL de base est la partie de **l’URL du point de terminaison** située avant l’ID d’abonnement et les autres paramètres.

Ensuite, nous devons donner à votre bot cette instance de LUIS. Ouvrez **EchoBot.cs** et ajoutez le code suivant en haut du fichier. L’en-tête de classe et les éléments d’état sont également inclus à titre de référence, mais ils ne sont pas décrits ici.

```csharp
public class EchoBot : IBot
{
    /// <summary>
    /// Gets the Echo Bot state.
    /// </summary>
    private IStatePropertyAccessor<EchoState> EchoStateAccessor { get; }

    /// <summary>
    /// Gets the LUIS recognizer.
    /// </summary>
    private LuisRecognizer Recognizer { get; } = null;

    public EchoBot(ConversationState state, LuisRecognizer luis)
    {
        EchoStateAccessor = state.CreateProperty<EchoState>("EchoBot.EchoState");

        // The incoming luis variable is the LUIS Recognizer we added above.
        this.Recognizer = luis ?? throw new ArgumentNullException(nameof(luis));
    }

    /// ...
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

Tout d’abord, suivez la procédure dans le [démarrage rapide](../javascript/bot-builder-javascript-quickstart.md) JavaScript pour créer un bot. Ici, nous codons en dur nos informations LUIS dans notre bot, mais elles peuvent être extraites du fichier `.bot`, comme c’est le cas dans l’exemple associé présenté à la fin de l’article.

Dans le nouveau bot, modifiez **app.js** pour demander la classe `LuisRecognizer` et créez une instance de votre modèle LUIS :

```javascript
const { ActivityTypes } = require('botbuilder');
const { LuisRecognizer } = require('botbuilder-ai');

const luisApplication = {
    // This appID is for a public app that's made available for demo purposes
    // You can use it or use your own LUIS "Application ID" at https://www.luis.ai under "App Settings".
     applicationId: 'eb0bf5e0-b468-421b-9375-fdfb644c512e',
    // Replace endpointKey with your "Subscription Key"
    // your key is at https://www.luis.ai under Publish > Resources and Keys, look in the Endpoint column
    // The "subscription-key" is embeded in the Endpoint link. 
    endpointKey: '<your subscription key>',
    // You can find your app's region info embeded in the Endpoint link as well.
    // Some examples of regions are `westus`, `westcentralus`, `eastus2`, and `southeastasia`.
    azureRegion: 'westus'
}

// Create configuration for LuisRecognizer's runtime behavior.
const luisPredictionOptions = {
    includeAllIntents: true,
    log: true,
    staging: false
}

// Create the bot that handles incoming Activities.
const luisBot = new LuisBot(luisApplication, luisPredictionOptions);
```

Ensuite, dans le constructeur de votre bot `LuisBot`, récupérez l’application pour créer l’instance de LuisRecognizer.

```javascript
    /**
     * The LuisBot constructor requires one argument (`application`) which is used to create an instance of `LuisRecognizer`.
     * @param {object} luisApplication The basic configuration needed to call LUIS. In this sample the configuration is retrieved from the .bot file.
     * @param {object} luisPredictionOptions (Optional) Contains additional settings for configuring calls to LUIS.
     */
    constructor(application, luisPredictionOptions) {
        this.luisRecognizer = new LuisRecognizer(application, luisPredictionOptions, true);
    }
```

> [!NOTE] 
> Si vous utilisez votre propre application LUIS au lieu de l’application LUIS publique, vous pouvez obtenir l’ID d’application, la clé d’abonnement et la région correspondants sur [https://www.luis.ai](https://www.luis.ai). Vous pouvez trouver ces derniers sous les onglets Publier et Paramètres de la page de votre application.

---

La compréhension langagière LUIS est maintenant configurée pour votre bot. Ensuite, examinons la façon dont nous allons obtenir l’intention de LUIS.

## <a name="get-the-intent-by-calling-luis"></a>Obtenir l’intention en appelant LUIS

Votre bot obtient des résultats de LUIS en appelant le module de reconnaissance LUIS.

# <a name="ctabcs"></a>[C#](#tab/cs)

Pour que votre bot envoie simplement une réponse basée sur l’intention que l’application LUIS a détectée, appelez `LuisRecognizer` pour obtenir un élément `RecognizerResult`. Vous pouvez effectuer cette action dans votre code toutes les fois où vous devez obtenir l’intention de LUIS.

```cs
using System.Threading.Tasks;
using Microsoft.Bot;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Schema;
// add this reference
using Microsoft.Bot.Builder.AI.LUIS;

namespace EchoBot
{
    public class EchoBot : IBot
    {
        /// <summary>
        /// Echo bot turn handler 
        /// </summary>
        /// <param name="context">Turn scoped context containing all the data needed
        /// for processing this conversation turn. </param>        
        public async Task OnTurnAsync(ITurnContext context, System.Threading.CancellationToken token)
        {            
            // This bot is only handling Messages
            if (context.Activity.Type == ActivityTypes.Message)
            {
                // Call LUIS recognizer
                var result = this.Recognizer.RecognizeAsync(context, System.Threading.CancellationToken.None);
                
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
                        // Report the weather.
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

Les intentions reconnues dans l’énoncé sont retournées sous la forme d’un mappage des noms d’intention avec des scores et sont accessibles à partir `result.Intents`. Une méthode `LuisRecognizer.topIntent()` statique est mise à votre disposition, qui facilite la recherche de l’intention ayant reçu le meilleur score pour un jeu de résultats.

Toutes les entités reconnues sont renvoyées sous la forme d’un mappage de noms d’entité avec des valeurs et sont accessibles par le biais de `results.entities`. Vous pouvez retourner des métadonnées d’entité supplémentaires en passant un paramètre `verbose=true` quand vous créez l’objet LuisRecognizer. Les métadonnées ajoutées sont ensuite accessibles par le biais de `results.entities.$instance`.

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

Modifiez le code pour écouter l’activité entrante, afin qu’elle appelle `LuisRecognizer` pour obtenir un élément `RecognizerResult`.

```javascript
const { ActivityTypes } = require('botbuilder');
const { LuisRecognizer } = require('botbuilder-ai');

// Listen for incoming requests 
server.post('/api/messages', (req, res) => {
    // Route received request to adapter for processing
    adapter.processActivity(req, res, async (context) => {
        if (context.activity.type === 'message') {
            // Perform a call to LUIS to retrieve results for the user's message.
            const results = await this.luisRecognizer.recognize(turnContext);

            // Since the LuisRecognizer was configured to include the raw results, get the `topScoringIntent` as specified by LUIS.
            const topIntent = results.luisResult.topScoringIntent;
            
            switch (topIntent) {
                case 'None':
                    await context.sendActivity("Sorry, I don't understand.")
                    break;
                case 'Cancel':
                    await context.sendActivity("<cancelling the process>")
                    break;
                case 'Help':
                    await context.sendActivity("<here's some help>");
                    break;
                case 'Weather':
                    await context.sendActivity("The weather today is sunny.");
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


---

Essayez d’exécuter le bot dans l’émulateur Bot Framework, et dites-lui quelque chose comme « météo », « aide » et « annuler ».

![Exécuter le bot](./media/how-to-luis/run-luis-bot.png)

## <a name="extract-entities"></a>Extraire les entités

En plus de reconnaître une intention, une application LUIS peut extraire des entités qui sont des mots importants pour répondre à la demande d’un utilisateur. Par exemple, pour un bot de renseignements météorologiques, l’application LUIS peut être en mesure d’extraire du message de l’utilisateur le lieu dont il souhaite connaître le bulletin météorologique.

Une façon courante de structurer votre conversation consiste à identifier les entités dans le message de l’utilisateur et à l’inviter à indiquer les entités requises introuvables. Ensuite, les étapes suivantes traitent la réponse à l’invite.

# <a name="ctabcs"></a>[C#](#tab/cs)

Supposons que le message de l’utilisateur était « Quelle est la météo de Seattle » ? L’élément `LuisRecognizer` vous donne un élément `RecognizerResult` avec une propriété `Entities` qui a la structure suivante :

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

La fonction d’assistance suivante peut être ajoutée à votre bot pour obtenir des entités de `RecognizerResult` à partir de LUIS. La bibliothèque `Newtonsoft.Json.Linq` doit être utilisée ; vous devez l’ajouter à vos instructions **using**.

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

Quand vous collectez des informations telles que des entités à partir de plusieurs étapes d’une conversation, il peut être utile d’enregistrer les informations dont vous avez besoin dans un état. Si une entité est trouvée, elle peut être ajoutée au champ d’état approprié. Dans votre conversation, si l’étape actuelle a déjà le champ associé renseigné, l’étape visant à demander ces informations peut être ignorée.

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

Supposons que le message de l’utilisateur était « Quelle est la météo de Seattle » ? L’élément `LuisRecognizer` vous donne un élément `RecognizerResult` avec une propriété `entities` qui a la structure suivante :

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

Cette fonction `findEntities` recherche les entités reconnues par l’application LUIS qui correspondent à l’élément `entityName` entrant.


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

Quand vous collectez des informations telles que des entités à partir de plusieurs étapes d’une conversation, il peut être utile d’enregistrer les informations dont vous avez besoin dans un état. Si une entité est trouvée, elle peut être ajoutée au champ d’état approprié. Dans votre conversation, si l’étape actuelle a déjà le champ associé renseigné, l’étape visant à demander ces informations peut être ignorée.

## <a name="additional-resources"></a>Ressources supplémentaires

Pour obtenir un exemple utilisant LUIS, consultez les projets pour [[C#](https://aka.ms/cs-luis-sample)] ou [[JavaScript](https://aka.ms/js-luis-sample)].

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Combiner des services QnA et LUIS à l’aide de l’outil Dispatch](./bot-builder-tutorial-dispatch.md)
