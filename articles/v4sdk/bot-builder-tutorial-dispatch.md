---
title: Intégrer plusieurs services LUIS et QnA avec l’outil Dispatch | Microsoft Docs
description: Découvrez comment utiliser LUIS et QnA Maker dans votre bot.
keywords: Luis, QnA, outil Dispatch, services multiples
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 08/27/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 02dffcdd8cb4fd27f59fa8a763d7f2027aa0bcf2
ms.sourcegitcommit: 0b2be801e55f6baa048b49c7211944480e83ba95
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/28/2018
ms.locfileid: "43115034"
---
# <a name="integrate-multiple-luis-and-qna-services-with-the-dispatch-tool"></a>Intégrer plusieurs services LUIS et QnA avec l’outil Dispatch

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Ce tutoriel montre comment utiliser un modèle LUIS généré par l’outil Dispatch, pour intégrer votre bot à plusieurs applications Language Understanding (LUIS) et services QnAMaker. 

Imaginez que vous avez développé les services suivants et que vous souhaitez créer un bot qui s’intègre à chacun d’eux.

| Type de service | NOM | Description |
|------|------|------|
| Application LUIS | HomeAutomation | Reconnaît les intentions HomeAutomation.TurnOn, HomeAutomation.TurnOff et HomeAutomation.None.|
| Application LUIS | Météo | Reconnaît les intentions Weather.GetForecast et Weather.GetCondition.|
| Service QnAMaker | Forum Aux Questions  | Fournit des réponses aux questions sur un système d’éclairage domotique |

Commençons par créer les applications et services, puis intégrons-les ensemble à l’aide de l’outil Dispatch.

## <a name="create-the-luis-apps"></a>Créer les applications LUIS

Le moyen le plus rapide de créer les applications LUIS *HomeAutomation* et *Weather* consiste à télécharger les fichiers [homeautomation.json][HomeAutomationJSON] et [weather.json][WeatherJSON]. Ensuite, importez ces deux applications LUIS en procédant comme suit.

1. Accédez au [site web LUIS](https://www.luis.ai/home) et connectez-vous. 
2. À partir de la page **My Apps** (Mes applications), cliquez sur **Import new app** (Importer une nouvelle application).
   1. Pour l’application *homeautomation*, choisissez le fichier **homeautomation.json** et nommez-le « homeautomation ». Cliquez sur **Done** (Terminé) pour importer l’application. Une fois l’application importée, cliquez sur **Publish** (Publier) pour publier l’application.
   1. Pour l’application *weather*, choisissez le fichier **weather.json** et nommez-le « weather ». Cliquez sur **Done** (Terminé) pour importer l’application. Une fois l’application importée, cliquez sur **Publish** (Publier) pour publier l’application.

## <a name="create-the-qna-cognitive-service-in-azure"></a>Créer le service cognitif de questions/réponses dans Azure

Un service QnA Maker implique deux parties, que sont le Service cognitif dans Azure et la base de connaissances de paires questions et réponses que vous publiez à l’aide du Service cognitif. Lorsque vous créez votre base de connaissances, vous pouvez la lier au service Azure en choisissant le **nom de service Azure** que vous avez créé dans le cadre de cette étape.

Pour créer le service cognitif dans Azure, connectez-vous au [Portail Azure](https://portal.azure.com), puis procédez comme suit :

1. Cliquez sur **Tous les services**.
1. Effectuez une recherche sur `Cognitive` et sélectionnez **Cognitive Services**.
1. Cliquez sur **Add**.
1. Effectuez une recherche sur `QnA` et sélectionnez **QnA Maker**.
1. Dans le panneau QnA Maker, cliquez sur **Créer**.
1. Renseignez les informations et créez le service QnA Maker.

    <!-- TODO: Add screenshot.-->

    * Entrez un nom pour le service. Pour ce tutoriel, nous allons utiliser `SmartLightQnA`.
    * Sélectionnez l’abonnement à utiliser.
    * Sélectionnez le niveau tarifaire de gestion à utiliser ; nous allons utiliser le niveau F0 (gratuit).
    * Créez le groupe de ressources à utiliser, ou sélectionnez-en un. Pour ce tutoriel, nous allons créer un groupe de ressources `SmartLightQnA`.
    * Sélectionnez un niveau tarifaire de recherche ; nous allons utiliser le niveau B (base).
    * Sélectionnez un emplacement de recherche ; nous allons utiliser `West US`.
    * Entrez le nom d’application à utiliser ; nous allons conserver le nom par défaut `SmartLightQnA`.
    * Sélectionnez l’emplacement de site web ; nous allons utiliser `West US`.
    * Conservez l’activation par défaut d’App Insights.
    * Sélectionnez un emplacement App Insights ; nous allons utiliser `West US`.
    * Cliquez sur **Créer** pour créer votre service QnA Maker.
    * Azure crée votre service et commence à le déployer.

1. Une fois le service déployé, affichez la notification et cliquez sur **Accéder à la ressource** pour accéder au panneau du service.
1. Cliquez sur **Clés** pour récupérer vos clés.

    * Copiez le nom du service et la première clé. Vous devrez utiliser ce nom lors de la création de votre base de connaissances afin de lier le service à cette dernière. Vous utiliserez également cette clé à la place de la chaîne YOUR-AZURE-QNA-SUBSCRIPTION-KEY dans les étapes relatives au répartiteur ci-après.
    * Vous obtenez deux clés, ce qui vous permet de régénérer une d’elle à la fois sans devoir interrompre votre service.

## <a name="create-and-publish-the-qna-maker-knowledge-base"></a>Créer et publier la base de connaissances QnA Maker

Pour créer la base de connaissances, procédez comme suit :

1. Accédez au [site web QnA Maker](https://qnamaker.ai) et connectez-vous. 
1. Cliquez sur **Create a knowledge base** (Créer une base de connaissances) pour créer une base de connaissances. Étant donné que vous avez déjà créé le service cognitif dans Azure, vous pouvez ignorer **l’Étape 1**.
1. À **l’Étape 2**, pour le champ **Azure QnA service** (Service Azure QnA), choisissez le nom du service cognitif que vous avez créé dans Azure. Cette opération associe cette base de connaissances au service que vous avez créé dans Azure.
1. À **l’Étape 3**, nommez cette base de connaissances « FAQ ». 
1. À **l’Étape 4**, remplissez la base de connaissances avec cet [exemple de fichier TSV][FAQ_TSV]. Sinon, utilisez le contenu d’une page web. Dans ce cas, collez le lien dans le champ **URL**.
1. À **l’Étape 5**, cliquez sur **Create your KB** (Créer votre base de connaissances) pour créer la base de connaissances.

Une fois la base de connaissances créée, vous devez la publier en cliquant sur **Publish** (Publier) afin d’obtenir l’ID et le point de terminaison de la base de connaissances. Vous aurez besoin de ces informations dans la suite de ce processus.


## <a name="use-the-dispatch-tool-to-create-the-dispatcher-luis-app"></a>Utiliser l’outil Dispatch pour créer l’application LUIS de distribution
Maintenant que vous avez créé deux applications LUIS et une base de connaissances, utilisons l’outil Dispatch pour générer une application LUIS qui combine tous ces éléments. 

### <a name="step-1-install-the-dispatch-tool"></a>Étape 1 : Installer l’outil Dispatch

Ouvrez une fenêtre **Invite de commandes** et accédez à votre projet de répartiteur. Ensuite, exécutez la commande NPM ci-après pour installer [l’outil Dispatch][DispatchTool].

```cmd
npm install -g botdispatch
```

### <a name="step-2-initialize-the-dispatch-tool"></a>Étape 2 : Initialiser l’outil Dispatch

Exécutez la commande ci-après pour initialiser l’outil Dispatch avec le nom `CombineWeatherAndLights`. Dans la ligne de commande ci-après, substituez votre [clé de création LUIS](https://docs.microsoft.com/en-us/azure/cognitive-services/LUIS/luis-concept-keys) à la chaîne `"YOUR-LUIS-AUTHORING-KEY"`.

```cmd
dispatch init -name CombineWeatherAndLights -luisAuthoringKey "YOUR-LUIS-AUTHORING-KEY" -luisAuthoringRegion westus
```

### <a name="step-3-add-apps-to-dispatcher"></a>Étape 3 : Ajouter des applications au répartiteur

Obtenez l’ID de chacune des applications LUIS que vous avez créées. Cette information figure pour chaque application sous **My Apps** (Mes applications) sur le [site LUIS](https://www.luis.ai/home) ; cliquez sur le nom de l’application, puis cliquez sur **Settings** (Paramètres) pour voir **l’ID de l’application**. Utilisez la *clé de création LUIS* que vous avez obtenue à **l’Étape 2**.

Exécutez la commande ci-après pour chacune des applications LUIS que vous avez créées (par exemple : **homeautomation** et **weather**). Veillez à remplacer l’ID approprié pour la commande adéquate ci-dessous.

```
dispatch add -type luis -id "HOMEAUTOMATION-APP-ID" -name homeautomation -version 0.1 -key "YOUR-LUIS-AUTHORING-KEY"
dispatch add -type luis -id "WEATHER-APP-ID" -name weather -version 0.1 -key "YOUR-LUIS-AUTHORING-KEY"

```

Ensuite, ajoutez la base de connaissances QnAMaker en exécutant la commande suivante. Dans cette commande, la chaîne `QNA-KB-ID` fait référence à l’ID de base de connaissances qui vous a été fourni une fois que vous avez publié la base de connaissances, et la chaîne `YOUR-AZURE-QNA-SUBSCRIPTION-KEY` fait référence à la clé obtenue auprès du service cognitif que vous avez créé dans Azure.

```
dispatch add -type qna -id "QNA-KB-ID" -name faq -key "YOUR-AZURE-QNA-SUBSCRIPTION-KEY"
```

### <a name="step-4-create-the-dispatcher-luis-app"></a>Étape 4 : Créer l’application LUIS de répartiteur

Une fois toutes les applications ajoutées à l’outil Dispatch, exécutez la commande ci-après pour créer l’application de répartiteur. Si vous souhaitez inspecter le fichier dispatch, vous pouvez trouver les informations correspondantes dans un fichier portant l’extension **.dispatch**.

```
dispatch create
```

Cette commande crée l’application LUIS de répartiteur nommée **CombineWeatherAndLights**. Vous pouvez voir la nouvelle application dans [https://www.luis.ai/home](https://www.luis.ai/home). 

![Application de distribution dans LUIS.ai](media/tutorial-dispatch/dispatch-app-in-luis.png)

Cliquez sur la nouvelle application. Sous **Intents** (Intentions), vous pouvez voir qu’elle a les intentions `l_homeautomation`, `l_weather` et `q_faq`.

![Intentions de distribution dans LUIS.ai](media/tutorial-dispatch/dispatch-intents-in-luis.png)

Cliquez sur le bouton **Train** (former) pour former l’application LUIS et utilisez l’onglet **PUBLISH** (Publier) pour la [publier](https://docs.microsoft.com/en-us/azure/cognitive-services/LUIS/publishapp). Cliquez sur **Settings** (Paramètres) pour copier l’ID de la nouvelle application. Vous aurez besoin de cet ID pour permettre au bot de se connecter à cette application.

## <a name="create-the-bot"></a>Créer le bot

Vous pouvez maintenant associer les intentions de l’application de distribution à la logique de votre bot, qui achemine les messages vers les applications LUIS et service QnAMaker d’origine.

Vous pouvez utiliser l’exemple fourni avec le Kit Bot Builder comme point de départ.

# <a name="ctabcsaddref"></a>[C#](#tab/csaddref)

Démarrez avec le code de [l’exemple de distribution LUIS][DispatchBotCS]. Dans Visual Studio, [mettez à jour les packages Nuget](https://docs.microsoft.com/en-us/nuget/tools/package-manager-ui#updating-a-package) vers les dernières préversions des éléments suivants :

* `Microsoft.Bot.Builder.Integration.AspNet.Core`
* `Microsoft.Bot.Builder.Ai.QnA` (requis pour QnA Maker)
* `Microsoft.Bot.Builder.Ai.Luis` (requis pour LUIS)

# <a name="javascripttabjsaddref"></a>[JavaScript](#tab/jsaddref)

Téléchargez [l’exemple de distribution LUIS][DispatchBotJs].  Installez les packages requis, y compris le package `botbuilder-ai` pour LUIS et QnA Maker, à l’aide de npm :

* `npm install --save botbuilder@preview`
* `npm install --save botbuilder-ai@preview`

---


Configurez l’exemple pour utiliser l’application de distribution.

# <a name="ctabcsbotconfig"></a>[C#](#tab/csbotconfig)

Dans **appsettings.json** dans [l’exemple de distribution LUIS][DispatchBotCS], modifiez les champs suivants.

| NOM | Description |
|------|------|
| `Luis-SubscriptionKey` |  Votre clé d’abonnement LUIS. Ce peut être une clé de point de terminaison ou une clé de création, décrite [ici](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/luis-concept-keys). | 
| `Luis-ModelId-Dispatcher` | ID de l’application LUIS que génère l’outil Dispatch. | 
| `Luis-ModelId-HomeAutomation` | ID de l’application que vous avez créée à partir de homeautomation.json  | 
| `Luis-ModelId-Weather` | ID de l’application que vous avez créée à partir de weather.json | 
| `QnAMaker-Endpoint-Url` | Doit être défini sur https://westus.api.cognitive.microsoft.com/qnamaker/v2.0 pour les services QnA Maker en préversion. <br/>Définissez ce champ sur https://YOUR-QNA-SERVICE-NAME.azurewebsites.net/qnamaker pour les nouveaux services QnA Maker (disponibilité générale).|
| `QnAMaker-SubscriptionKey` | Votre clé d’abonnement Azure QnA Maker. | 
| `QnAMaker-KnowledgeBaseId` | ID de la base de connaissances que vous créez sur le [portail QnAMaker](https://qnamaker.ai).| 



Dans **Startup.cs**, jetez un coup de œil à la méthode `ConfigureServices`. Elle contient du code pour initialiser `LuisRecognizerMiddleware` à l’aide de l’ID de l’application LUIS que vous venez de générer.

```csharp
// This method gets called by the runtime. Use this method to add services to the container.
public void ConfigureServices(IServiceCollection services)
{
    services.AddSingleton(this.Configuration);
    services.AddBot<LuisDispatchBot>(options =>
    {
        options.CredentialProvider = new ConfigurationCredentialProvider(Configuration);

        var (luisModelId, luisSubscriptionKey, luisUri) = GetLuisConfiguration(this.Configuration, "Dispatcher");

        var luisModel = new LuisModel(luisModelId, luisSubscriptionKey, luisUri);

        // If you want to get all the intents and scores that LUIS recognized, set Verbose = true in luisOptions
        var luisOptions = new LuisRequest { Verbose = true };

        var middleware = options.Middleware;
        middleware.Add(new LuisRecognizerMiddleware(luisModel, luisOptions: luisOptions));
    });
}
```
Les méthodes `GetLuisConfiguration` et `GetQnAMakerConfiguration` récupèrent du fichier appsettings.json la configuration LUIS et du service de questions/réponses.
```csharp
public static (string modelId, string subscriptionId, Uri uri) GetLuisConfiguration(IConfiguration configuration, string serviceName)
{
    var modelId = configuration.GetSection($"Luis-ModelId-{serviceName}")?.Value;
    var subscriptionId = configuration.GetSection("Luis-SubscriptionKey")?.Value;
    var uri = new Uri(configuration.GetSection("Luis-Url")?.Value);
    return (modelId, subscriptionId, uri);
}

public static (string knowledgeBaseId, string subscriptionKey, string uri) GetQnAMakerConfiguration(IConfiguration configuration)
{
    var knowledgeBaseId = configuration.GetSection("QnAMaker-KnowledgeBaseId")?.Value;
    var subscriptionKey = configuration.GetSection("QnAMaker-SubscriptionKey")?.Value;
    var uri = configuration.GetSection("QnAMaker-Endpoint-Url")?.Value;
    return (knowledgeBaseId, subscriptionKey, uri);
}
```

### <a name="dispatch-the-message"></a>Distribuer le message

Jetez un coup de œil à **LuisDispatchBot.cs**, où le bot distribue le message à l’application LUIS ou QnA Maker pour un sous-composant. 

Dans `DispatchToTopIntent`, si votre application de distribution détecte l’intention `l_homeautomation` ou `l_weather`, elle appelle une méthode `DispatchToTopIntent` qui crée un `LuisRecognizer` pour appeler les applications `homeautomation` et `weather` d’origine. Si le bot détecte l’intention `q_faq`, ou l’intention `none` qui est utilisée comme cas de repli, il appelle une méthode qui interroge QnAMaker.

> [!NOTE] 
> Si les noms d’intention `l_homeautomation`, `l_weather` ou `q_faq` ne correspondent pas à l’application LUIS que vous avez créée à l’aide de l’outil Dispatch, modifiez-les afin qu’ils correspondent à la version en minuscules des noms d’intention que vous voyez dans le [portail LUIS](https://www.luis.ai).

```csharp
private async Task DispatchToTopIntent(ITurnContext context, (string intent, double score)? topIntent)
{
    switch (topIntent.Value.intent.ToLowerInvariant())
    {
        case "l_homeautomation":
            await DispatchToLuisModel(context, this.luisModelHomeAutomation, "home automation");
            break;
        case "l_weather":
            await DispatchToLuisModel(context, this.luisModelWeather, "weather");
            break;
        case "none":
        // You can provide logic here to handle the known None intent (none of the above).
        // In this example we fall through to the QnA intent.
        case "q_faq":
            await DispatchToQnAMaker(context, this.qnaEndpoint, "FAQ");
            break;
        default:
            // The intent didn't match any case, so just display the recognition results.
            await context.SendActivity($"Dispatch intent: {topIntent.Value.intent} ({topIntent.Value.score}).");

            break;
    }
}
```

La méthode `DispatchToQnAMaker` envoie le message de l’utilisateur au service QnA Maker. Vérifiez que vous avez publié ce service dans le [portail QnA Maker](https://qnamaker.ai) avant d’exécuter le bot.

```csharp
private static async Task DispatchToQnAMaker(ITurnContext context, QnAMakerEndpoint qnaOptions, string appName)
{
    QnAMaker qnaMaker = new QnAMaker(qnaOptions);
    if (!string.IsNullOrEmpty(context.Activity.Text))
    {
        var results = await qnaMaker.GetAnswers(context.Activity.Text.Trim()).ConfigureAwait(false);
        if (results.Any())
        {
            await context.SendActivity(results.First().Answer);
        }
        else
        {
            await context.SendActivity($"Couldn't find an answer in the {appName}.");
        }
    }
}
```

La méthode `DispatchToLuisModel` envoie le message de l’utilisateur aux applications LUIS `homeautomation` et `weather` d’origine. Vérifiez que vous avez publié ces application LUIS dans le [portail LUIS](https://www.luis.ai) avant d’exécuter le bot.

```csharp
private static async Task DispatchToLuisModel(ITurnContext context, LuisModel luisModel, string appName)
{
    await context.SendActivity($"Sending your request to the {appName} system ...");
    var (intents, entities) = await RecognizeAsync(luisModel, context.Activity.Text);

    await context.SendActivity($"Intents detected by the {appName} app:\n\n{string.Join("\n\n", intents)}");

    if (entities.Count() > 0)
    {
        await context.SendActivity($"The following entities were found in the message:\n\n{string.Join("\n\n", entities)}");
    }
    
    // Here, you can add code for calling the hypothetical home automation or weather service, 
    // passing in the appName and any intent or entity information that you need 
}
```

La méthode `RecognizeAsync` appelle un `LuisRecognizer` pour obtenir les résultats à partir d’une application LUIS.

```cs
private static async Task<(IEnumerable<string> intents, IEnumerable<string> entities)> RecognizeAsync(LuisModel luisModel, string text)
{
    var luisRecognizer = new LuisRecognizer(luisModel);
    var recognizerResult = await luisRecognizer.Recognize(text, System.Threading.CancellationToken.None);

    // list the intents
    var intents = new List<string>();
    foreach (var intent in recognizerResult.Intents)
    {
        intents.Add($"'{intent.Key}', score {intent.Value}");
    }

    // list the entities
    var entities = new List<string>();
    foreach (var entity in recognizerResult.Entities)
    {
        if (!entity.Key.ToString().Equals("$instance"))
        {
            entities.Add($"{entity.Key}: {entity.Value.First}");
        }
    }

    return (intents, entities);
}
```

# <a name="javascripttabjsbotconfig"></a>[JavaScript](#tab/jsbotconfig)

Démarrez avec le code de [l’exemple de bot de distribution][DispatchBotJs]. Ouvrez **app.js** et éventuellement remplacez les champs `appId` par les ID des applications LUIS que vous avez créées. Si vous ne touchez pas aux champs `appId`, vous allez utiliser des applications LUIS publiques créées à des fins de démonstration. En ce qui concerne la clé `LUIS_SUBSCRIPTION_KEY`, vous pouvez la trouver dans la page de publication de toute application LUIS. Elle est incorporée dans le lien du point de terminaison figurant sur cette page.

```javascript
// Packages
const { BotFrameworkAdapter, MemoryStorage, ConversationState } = require('botbuilder');
const { restify } = require('restify');
const { LuisRecognizer, QnAMaker } = require('botbuilder-ai');
const { DialogSet } = require('botbuilder-dialogs');

// Create LuisRecognizers and QnAMaker
// The LUIS applications are public, meaning you can use your own subscription key to test the applications.
// For QnAMaker, users are required to create their own knowledge base.
// The exported LUIS applications and QnAMaker knowledge base can be found with the sample bot.

// The corresponding LUIS application JSON is `dispatchSample.json`
const dispatcher = new LuisRecognizer({
    appId: '0b18ab4f-5c3d-4724-8b0b-191015b48ea9',
    subscriptionKey: process.env.LUIS_SUBSCRIPTION_KEY,
    serviceEndpoint: 'https://westus.api.cognitive.microsoft.com/',
    verbose: true
});

// The corresponding LUIS application JSON is `homeautomation.json`
const homeAutomation = new LuisRecognizer({
    appId: 'c6d161a5-e3e5-4982-8726-3ecec9b4ed8d',
    subscriptionKey: process.env.LUIS_SUBSCRIPTION_KEY,
    serviceEndpoint: 'https://westus.api.cognitive.microsoft.com/',
    verbose: true
});

// The corresponding LUIS application JSON is `weather.json`
const weather = new LuisRecognizer({
    appId: '9d0c9e9d-ce04-4257-a08a-a612955f2fb5',
    subscriptionKey: process.env.LUIS_SUBSCRIPTION_KEY,
    serviceEndpoint: 'https://westus.api.cognitive.microsoft.com/',
    verbose: true
});
```

Dans le code suivant, remplacez `endpointKey` et `knowledgeBaseID` par vos clé et ID de base de connaissances issus de QnAMaker. `host` doit être défini sur `https://westus.api.cognitive.microsoft.com/qnamaker/v2.0` pour les services QnA Maker en préversion, et sur `https://YOUR-QNA-SERVICE-NAME.azurewebsites.net/qnamaker` pour les nouveaux services QnA Maker (disponibilité générale).

```javascript
const faq = new QnAMaker(
    {
        knowledgeBaseId: '',
        endpointKey: '',
        host: ''
    },
    {
        answerBeforeNext: true
    }
);

```

Le reste du code dans **app.js** gère les intentions `l_homeautomation`, `l_weather` et `None` en lançant les dialogues appropriés. Dans le cas de l’intention `q_faq`, il renvoie la réponse du service QnAMaker.

> [!NOTE] 
> Si les noms d’intention `l_homeautomation`, `l_weather` ou `q_faq` ne correspondent pas à l’application LUIS que vous avez créée à l’aide de l’outil Dispatch, modifiez-les afin qu’ils correspondent aux noms d’intention que vous voyez dans le [portail LUIS](https://www.luis.ai).

```javascript
// create conversation state
const conversationState = new ConversationState(new MemoryStorage());
adapter.use(conversationState);

// register some dialogs for usage with the LUIS apps that are being dispatched to
const dialogs = new DialogSet();

function findEntities(entityName, entityResults) {
    let entities = []
    if (entityName in entityResults) {
        entityResults[entityName].forEach((entity, idx) => {
            entities.push(entity);
        });
    }
    return entities.length > 0 ? entities : undefined;
}

dialogs.add('HomeAutomation_TurnOff', [
    async (dialogContext, args) => {
        const devices = findEntities('HomeAutomation_Device', args.entities);
        const operations = findEntities('HomeAutomation_Operation', args.entities);

        const state = conversationState.get(dialogContext.context);
        state.homeAutomationTurnOff = state.homeAutomationTurnOff ? state.homeAutomationTurnOff + 1 : 1;
        await dialogContext.context.sendActivity(`${state.homeAutomationTurnOff}: You reached the "HomeAutomation.TurnOff" dialog.`);
        if (devices) {
            await dialogContext.context.sendActivity(`Found these "HomeAutomation_Device" entities:\n${devices.join(', ')}`);
        }
        if (operations) {
            await dialogContext.context.sendActivity(`Found these "HomeAutomation_Operation" entities:\n${operations.join(', ')}`);
        }
        await dialogContext.end();
    }
]);

dialogs.add('HomeAutomation_TurnOn', [
    async (dialogContext, args) => {
        const devices = findEntities('HomeAutomation_Device', args.entities);
        const operations = findEntities('HomeAutomation_Operation', args.entities);

        const state = conversationState.get(dialogContext.context);
        state.homeAutomationTurnOn = state.homeAutomationTurnOn ? state.homeAutomationTurnOn + 1 : 1;
        await dialogContext.context.sendActivity(`${state.homeAutomationTurnOn}: You reached the "HomeAutomation.TurnOn" dialog.`);
        if (devices) {
            await dialogContext.context.sendActivity(`Found these "HomeAutomation_Device" entities:\n${devices.join(', ')}`);
        }
        if (operations) {
            await dialogContext.context.sendActivity(`Found these "HomeAutomation_Operation" entities:\n${operations.join(', ')}`);
        }
        await dialogContext.end();
    }
]);

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

dialogs.add('Weather_GetCondition', [
    async (dialogContext, args) => {
        const locations = findEntities('Weather_Location', args.entities);

        const state = conversationState.get(dialogContext.context);
        state.weatherGetCondition = state.weatherGetCondition ? state.weatherGetCondition + 1 : 1;
        await dialogContext.context.sendActivity(`${state.weatherGetCondition}: You reached the "Weather.GetCondition" dialog.`);
        if (locations) {
            await dialogContext.context.sendActivity(`Found these "Weather_Location" entities:\n${locations.join(', ')}`);
        }
        await dialogContext.end();
    }
]);

dialogs.add('None', [
    async (dialogContext) => {
        const state = conversationState.get(dialogContext.context);
        state.noneIntent = state.noneIntent ? state.noneIntent + 1 : 1;
        await dialogContext.context.sendActivity(`${state.noneIntent}: You reached the "None" dialog.`);
        await dialogContext.end();
    }
]);

adapter.use(dispatcher);

// Listen for incoming Activities
server.post('/api/messages', (req, res) => {
    adapter.processActivity(req, res, async (context) => {
        if (context.activity.type === 'message') {
            const state = conversationState.get(context);
            const dc = dialogs.createContext(context, state);

            // Retrieve the LUIS results from our dispatcher LUIS application
            const luisResults = dispatcher.get(context);

            // Extract the top intent from LUIS and use it to select which LUIS application to dispatch to
            const topIntent = LuisRecognizer.topIntent(luisResults);

            const isMessage = context.activity.type === 'message';
            if (isMessage) {
                switch (topIntent) {
                    case 'l_homeautomation':
                        const homeAutoResults = await homeAutomation.recognize(context);
                        const topHomeAutoIntent = LuisRecognizer.topIntent(homeAutoResults);
                        await dc.begin(topHomeAutoIntent, homeAutoResults);
                        break;
                    case 'l_weather':
                        const weatherResults = await weather.recognize(context);
                        const topWeatherIntent = LuisRecognizer.topIntent(weatherResults);
                        await dc.begin(topWeatherIntent, weatherResults);
                        break;
                    case 'q_faq':
                        await faq.answer(context);
                        break;
                    default:
                        await dc.begin('None');
                }
            }

            if (!context.responded) {
                await dc.continue();
                if (!context.responded && isMessage) {
                    await dc.context.sendActivity(`Hi! I'm the LUIS dispatch bot. Say something and LUIS will decide how the message should be routed.`);
                }
            }
        }
    });
});

```

---
## <a name="run-the-bot"></a>Exécuter le bot

Testez le bot à l’aide de [l’émulateur Bot Framework](../bot-service-debug-emulator.md). Envoyez-lui des messages tels que « allumer la lumière » à distribuer à l’application LUIS domotique, et des messages tels que « obtenir la météo de Seattle » à distribuer à l’application LUIS d’informations météorologiques.

> [!NOTE] 
> Avant d’exécuter le bot, vérifiez que vous avez publié toutes les applications LUIS que vous avez créées dans le [portail LUIS](https://www.luis.ai) et vérifiez que vous avez publié le service QnA Maker dans le [portail QnA Maker](https://qnamaker.ai).

![Envoyer des messages au bot de distribution](media/tutorial-dispatch/run-dispatch-bot.png)

## <a name="evaluate-the-dispatchers-performance"></a>Évaluer les performances de l’application de distribution

Il y a parfois des messages utilisateur qui sont fournis comme exemples dans les applications LUIS et les services QnA Maker ; l’application LUIS combinée que génère l’outil Dispatch ne traite pas ces entrées correctement. Vérifiez les performances de votre application à l’aide de l’option `eval`. 

```
dispatch eval
```

L’exécution de `dispatch eval` génère un fichier **Summary.html** qui fournit des statistiques sur les performances du nouveau modèle de langage.

> [!TIP] 
> Vous pouvez exécuter `dispatch eval` sur n’importe quelle application LUIS, pas seulement sur les applications LUIS créées par l’outil Dispatch.

### <a name="edit-intents-for-duplicates-and-overlaps"></a>Modifier les intentions en cas de doublons et de redondances

Passez en revue les exemples d’énoncés marqués comme étant des doublons dans **Summary.html** et supprimez les exemples similaires ou redondants. Par exemple, supposons que dans l’application LUIS `homeautomation` domotique, des demandes comme « allumer la lumière » correspondent à une intention « TurnOnLights » (Allumer la lumière), mais que des demandes comme « Pourquoi la lumière ne s’allume-t-elle pas ? » correspondent à une intention « None » (Aucune) afin qu’elles puissent être transmises à QnA Maker. Quand vous combinez l’application LUIS et le service QnA Maker à l’aide de l’outil Dispatch, vous devez effectuer l’une des opérations suivantes : 

* Supprimez l’intention « None » (Aucune) de l’application LUIS `homeautomation` d’origine et ajoutez les énoncés dans cette intention à l’intention « None » (Aucune) dans l’application de distribution.
* Si vous ne supprimez pas l’intention « None » (Aucune) de l’application LUIS d’origine, vous devez ajouter de la logique à votre bot pour transmettre les messages qui correspondent à cette intention au service QnA Maker.

> [!TIP] 
> Consultez [Bonnes pratiques pour Language Understanding](./bot-builder-concept-luis.md#best-practices-for-language-understanding) pour obtenir des conseils sur l’amélioration des performances de votre modèle de langage.

## <a name="additional-resources"></a>Ressources supplémentaires

* [Utilisation de LUIS pour le flux de la conversation][luis-v4-how-to]

[luis-v4-how-to]: bot-builder-howto-v4-luis.md
<!-- links -->

[HomeAutomationJSON]: https://aka.ms/dispatch-luis1
[WeatherJSON]: https://aka.ms/dispatch-luis2
[DispatchJSON]: https://aka.ms/dispatch-luis
[FAQ_TSV]: https://aka.ms/dispatch-qna-tsv

[DispatchTool]: https://github.com/Microsoft/botbuilder-tools/tree/master/Dispatch
[DispatchBotCS]: https://aka.ms/dispatch-sample-cs
[DispatchBotJs]: https://aka.ms/dispatch-sample-js



