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
ms.date: 01/15/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: c798c26f108458e1caeb16aa22c02c6e7c70fb61
ms.sourcegitcommit: 3cc768a8e676246d774a2b62fb9c688bbd677700
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/16/2019
ms.locfileid: "54323655"
---
# <a name="use-multiple-luis-and-qna-models"></a>Utiliser plusieurs modèles LUIS et QnA

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Dans ce tutoriel, nous vous montrons comment utiliser le service Dispatch pour router des énoncés quand un bot prend en charge plusieurs modèles LUIS et services QnA Maker pour différents scénarios. Dans ce cas, nous configurons Dispatch avec plusieurs modèles LUIS pour des conversations portant sur la domotique et la météo, plus le service QnA Maker pour répondre aux questions basées sur un fichier texte FAQ comme entrée. Cet exemple combine les services suivants.

| NOM | Description |
|------|------|
| HomeAutomation | Application LUIS qui reconnaît une intention domotique avec les données d’entité associées.|
| Météo | Application LUIS qui reconnaît les intentions `Weather.GetForecast` et `Weather.GetCondition` avec des données de localisation.|
| Forum Aux Questions  | Base de connaissances QnA Maker qui fournit des réponses à certaines questions simples concernant le bot. |

## <a name="prerequisites"></a>Prérequis

- Le code de cet article est basé sur l’exemple **NLP with Dispatch**. Vous aurez besoin d’une copie de l’exemple en [C# ](https://aka.ms/dispatch-sample-cs) ou en [JS](https://aka.ms/dispatch-sample-js).
- Connaissances des [concepts de base des bots](bot-builder-basics.md), du [traitement en langage naturel](bot-builder-howto-v4-luis.md), de [QnA Maker](bot-builder-howto-qna.md) et du fichier [.bot](bot-file-basics.md).
- [Bot Framework Emulator](https://github.com/Microsoft/BotFramework-Emulator/blob/master/README.md#download) à des fins de test.

## <a name="create-the-services-and-test-the-bot"></a>Créer les services et tester le bot

Vous pouvez suivre les instructions du **README** concernant [C#](https://aka.ms/dispatch-sample-readme-cs) ou [JS](https://aka.ms/dispatch-sample-readme-js) pour créer ce bot à l’aide d’appels d’interface de ligne de commande, ou suivre les étapes ci-dessous pour créer manuellement votre bot à l’aide des interfaces utilisateur Azure, LUIS et QnAMaker.

 ### <a name="create-your-bot-using-service-ui"></a>Créer votre bot à l’aide de l’interface utilisateur du service
 
Pour commencer à créer votre bot, téléchargez dans un dossier local les 4 fichiers suivants situés dans le dépôt GitHub [BotFramework-Samples](https://aka.ms/botdispatchgitsamples) : [home-automation.json](https://aka.ms/dispatch-home-automation-json), [weather.json ](https://aka.ms/dispatch-weather-json), [nlp-with-dispatchDispatch.json](https://aka.ms/dispatch-dispatch-json), [QnAMaker.tsv](https://aka.ms/dispatch-qnamaker-tsv). Pour ce faire, vous pouvez ouvrir le lien du dépôt GitHub ci-dessus, cliquer sur **BotFramework-Samples**, puis « cloner ou télécharger » le dépôt sur votre ordinateur local. Notez que ces fichiers se trouvent dans un autre dépôt que celui de l’exemple mentionné dans les prérequis.

### <a name="manually-create-luis-apps"></a>Créer manuellement des applications LUIS

Connectez-vous au [portail web de LUIS](https://www.luis.ai/). Dans la section _My apps_, sélectionnez l’onglet _Import new app_. La boîte de dialogue suivante s’affiche :

![Importer un fichier json LUIS](./media/tutorial-dispatch/import-new-luis-app.png)

Sélectionnez le bouton _Choose app file_ et sélectionnez le fichier téléchargé « home-automation.json ». Laissez le champ de nom facultatif vide. Sélectionnez _Terminé_.

Une fois que LUIS ouvre votre application Home Automation, sélectionnez le bouton _Train_. Cela vous permet d’entraîner votre application avec l’ensemble des énoncés que vous venez d’importer à l’aide du fichier « home-automation.json ».

Une fois l’entraînement terminé, sélectionnez le bouton _Publish_. La boîte de dialogue suivante s’affiche :

![Publier l’application LUIS](./media/tutorial-dispatch/publish-luis-app.png)

Choisissez l’environnement « production » et sélectionnez le bouton _Publish_.

Une fois que votre nouvelle application LUIS a été publiée, sélectionnez l’onglet _MANAGE_. Dans la page Application Informations, enregistrez les valeurs `Application ID` et `Display name`. Dans la page Key and Endpoints, enregistrez les valeurs `Authoring Key` et `Region`. Ces valeurs seront utilisées plus tard par votre fichier « nlp-with-dispatch.bot ».

Une fois que vous avez terminé, _entraînez_ et _publiez_ votre application de météo LUIS et votre application de distribution LUIS en répétant ces mêmes étapes pour les fichiers « weather.json » et « nlp-with-dispatchDispatch.json » téléchargés localement.

### <a name="manually-create-qna-maker-app"></a>Créer manuellement une application QnA Maker

La première étape pour configurer une base de connaissances QnA Maker est de commencer par configurer un service QnA Maker dans Azure. Pour ce faire, suivez les instructions étape par étape [ici](https://aka.ms/create-qna-maker). Connectez-vous maintenant au [portail web de QnAMaker](https://qnamaker.ai). Descendez à l’étape 2

![Créer QnA - Étape 2](./media/tutorial-dispatch/create-qna-step-2.png)

et sélectionnez
1. Votre compte Azure AD.
1. Le nom de votre abonnement Azure.
1. Le nom que vous avez créé pour votre service QnA Maker. (Si votre service Azure QnA n’apparaît pas dans cette liste déroulante, essayez d’actualiser la page.) 

Passer à l’étape 3

![Créer QnA - Étape 3](./media/tutorial-dispatch/create-qna-step-3.png)

Fournissez un nom pour votre base de connaissances QnA Maker. Pour cet exemple, nous allons utiliser le nom « sample-qna ».

Passer à l’étape 4

![Créer QnA - Étape 4](./media/tutorial-dispatch/create-qna-step-4.png)

Sélectionnez l’option _+ Add File_ et sélectionnez le fichier téléchargé « QnAMaker.tsv ».

Il existe une sélection supplémentaire pour ajouter une personnalité _Chit-chat_ à votre base de connaissances, mais notre exemple n’inclut pas cette option.

Sélectionnez _Save and train_ et une fois l’opération terminée, sélectionnez l’onglet _PUBLISH_ et publiez votre application.

Une fois que votre application QnA Maker est publiée, sélectionnez l’onglet _SETTINGS_ et faites défiler jusqu’à « Deployment details ». Enregistrez les valeurs suivantes à partir de la requête HTTP de l’exemple _Postman_.

```
POST /knowledgebases/<Your_Knowledgebase_Id>/generateAnswer
Host: <Your_Hostname>
Authorization: EndpointKey <Your_Endpoint_Key>
```
Ces valeurs seront utilisées plus tard par votre fichier « nlp-with-dispatch.bot ».

### <a name="manually-update-your-bot-file"></a>Mettre à jour manuellement votre fichier .bot

Une fois que toutes vos applications de service sont créées, les informations de chacune d’elles doivent être ajoutées dans votre fichier « nlp-with-dispatch.bot ». Ouvrez ce fichier dans l’exemple de fichier C# ou JS que vous avez précédemment téléchargé. Ajoutez les valeurs suivantes dans chaque section de "type": "luis" ou "type": "dispatch"

```
"appId": "<Your_Recorded_App_Id>",
"authoringKey": "<Your_Recorded_Authoring_Key>",
"subscriptionKey": "<Your_Recorded_Authoring_Key>",
"version": "0.1",
"region": "<Your_Recorded_Region>",
```

Pour la section "type": "qna", ajoutez les valeurs suivantes :

```
"type": "qna",
"name": "sample-qna",
"id": "201",
"kbId": "<Your_Recorded_Knowledgebase_Id>",
"subscriptionKey": "<Your_Azure_Subscription_Key>", // Used when creating your QnA service.
"endpointKey": "<Your_Recorded_Endpoint_Key>",
"hostname": "<Your_Recorded_Hostname>"
```

Quand tous les changements sont en place, enregistrez ce fichier.

### <a name="test-your-bot"></a>Tester votre robot

Exécutez maintenant l’exemple à l’aide de l’émulateur. Une fois que l’émulateur est ouvert, sélectionnez le fichier « nlp-with-dispatch.bot ».

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

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

L’exemple de code utilise des constantes de nommage prédéfinies pour identifier les différentes sections de votre fichier .bot. Si vous avez modifié des noms de section prédéfinis dans votre fichier _nlp-with-dispatch.bot_, veillez à placer la déclaration de constante associée dans le fichier **bot.js**,  **homeAutomation.js**, **qna.js** ou **weather.js** et à remplacer cette entrée par le nom modifié.  
```javascript
// In file bot.js
// this is the LUIS service type entry in the .bot file.
const DISPATCH_CONFIG = 'nlp-with-dispatchDispatch';

// In file homeAutomation.js
// this is the LUIS service type entry in the .bot file.
const LUIS_CONFIGURATION = 'Home Automation';

// In file qna.js
// Name of the QnA Maker service in the .bot file.
const QNA_CONFIGURATION = 'sample-qna';

// In file weather.js
// this is the LUIS service type entry in the .bot file.
const WEATHER_LUIS_CONFIGURATION = 'Weather';
```

Dans **bot.js**, les informations contenues dans le fichier de configuration _nlp-with-dispatch.bot_ permettent de connecter votre bot de distribution aux différents services. Chaque constructeur recherche et utilise les sections appropriées du fichier de configuration selon les noms de section détaillés plus haut.

```javascript
class DispatchBot {
    constructor(conversationState, userState, botConfig) {
        //...
        this.homeAutomationDialog = new HomeAutomation(conversationState, userState, botConfig);
        this.weatherDialog = new Weather(botConfig);
        this.qnaDialog = new QnA(botConfig);

        this.conversationState = conversationState;
        this.userState = userState;

        // dispatch recognizer
        const dispatchConfig = botConfig.findServiceByNameOrId(DISPATCH_CONFIG);
        //...
```
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

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Dans la méthode `onTurn` de **bot.js**, nous vérifions les messages entrants de l’utilisateur. Si le type _ActivityType.Message_ est reçu, ce message est envoyé via le _dispatchRecognizer_ du bot.

```javascript
if (turnContext.activity.type === ActivityTypes.Message) {
    // determine which dialog should fulfill this request
    // call the dispatch LUIS model to get results.
    const dispatchResults = await this.dispatchRecognizer.recognize(turnContext);
    const dispatchTopIntent = LuisRecognizer.topIntent(dispatchResults);
    //...
 }
```
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

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Lorsque le modèle renvoie un résultat, il indique quel service est le mieux à même de gérer l’énoncé. Le code dans ce bot route la demande au service correspondant.

```javascript
switch (dispatchTopIntent) {
   case HOME_AUTOMATION_INTENT:
      await this.homeAutomationDialog.onTurn(turnContext);
      break;
   case WEATHER_INTENT:
      await this.weatherDialog.onTurn(turnContext);
      break;
   case QNA_INTENT:
      await this.qnaDialog.onTurn(turnContext);
      break;
   case NONE_INTENT:
      default:
      // Unknown request
       await turnContext.sendActivity(`I do not understand that.`);
       await turnContext.sendActivity(`I can help with weather forecast, turning devices on and off and answer general questions like 'hi', 'who are you' etc.`);
 }
 
 // In homeAutomation.js
 async onTurn(turnContext) {
    // make call to LUIS recognizer to get home automation intent + entities
    const homeAutoResults = await this.luisRecognizer.recognize(turnContext);
    const topHomeAutoIntent = LuisRecognizer.topIntent(homeAutoResults);
    // depending on intent, call turn on or turn off or return unknown
    switch (topHomeAutoIntent) {
       case HOME_AUTOMATION_INTENT:
          await this.handleDeviceUpdate(homeAutoResults, turnContext);
          break;
       case NONE_INTENT:
       default:
         await turnContext.sendActivity(`HomeAutomation dialog cannot fulfill this request.`);
    }
}
    
// In weather.js
async onTurn(turnContext) {
   // Call weather LUIS model.
   const weatherResults = await this.luisRecognizer.recognize(turnContext);
   const topWeatherIntent = LuisRecognizer.topIntent(weatherResults);
   // Get location entity if available.
   const locationEntity = (LOCATION_ENTITY in weatherResults.entities) ? weatherResults.entities[LOCATION_ENTITY][0] : undefined;
   const locationPatternAnyEntity = (LOCATION_PATTERNANY_ENTITY in weatherResults.entities) ? weatherResults.entities[LOCATION_PATTERNANY_ENTITY][0] : undefined;
   // Depending on intent, call "Turn On" or "Turn Off" or return unknown.
   switch (topWeatherIntent) {
      case GET_CONDITION_INTENT:
         await turnContext.sendActivity(`You asked for current weather condition in Location = ` + (locationEntity || locationPatternAnyEntity));
         break;
      case GET_FORECAST_INTENT:
         await turnContext.sendActivity(`You asked for weather forecast in Location = ` + (locationEntity || locationPatternAnyEntity));
         break;
      case NONE_INTENT:
      default:
         wait turnContext.sendActivity(`Weather dialog cannot fulfill this request.`);
   }
}
    
// In qna.js
async onTurn(turnContext) {
   // Call QnA Maker and get results.
   const qnaResult = await this.qnaRecognizer.generateAnswer(turnContext.activity.text, QNA_TOP_N, QNA_CONFIDENCE_THRESHOLD);
   if (!qnaResult || qnaResult.length === 0 || !qnaResult[0].answer) {
       await turnContext.sendActivity(`No answer found in QnA Maker KB.`);
       return;
    }
    // respond with qna result
    await turnContext.sendActivity(qnaResult[0].answer);
}
```
---

## <a name="edit-intents-to-improve-performance"></a>Modifier les intentions pour améliorer les performances

Une fois que votre bot est en cours d’exécution, il est possible d’améliorer les performances du bot en supprimant les énoncés similaires ou qui se recoupent. Par exemple, supposons que dans l’application LUIS `Home Automation`, des demandes comme « allumer la lumière » correspondent à une intention « TurnOnLights » (Allumer la lumière), mais que des demandes comme « Pourquoi la lumière ne s’allume-t-elle pas ? » correspondent à une intention « None » (Aucune) afin qu’elles puissent être transmises à QnA Maker. Quand vous combinez l’application LUIS et le service QnA Maker à l’aide de l’outil Dispatch, vous devez effectuer l’une des opérations suivantes :

* Supprimez l’intention « None » de l’application LUIS `Home Automation` d’origine et ajoutez plutôt les énoncés de cette intention dans l’intention « None » de l’application de distribution.
* Si vous ne supprimez pas l’intention « None » de l’application LUIS d’origine, vous devrez ajouter la logique dans votre bot visant à transmettre les messages qui correspondent à votre intention « None » au service QnA Maker.

Les deux actions ci-dessus réduisent le nombre de fois où votre bot répond à vos utilisateurs avec le message « Réponse introuvable ». 

## <a name="additional-resources"></a>Ressources supplémentaires

**Mettre à jour ou créer un modèle LUIS :** cet exemple est basé sur un modèle LUIS préconfiguré. Vous trouverez [ici](https://aka.ms/create-luis-model#updating-your-cognitive-models
) des informations complémentaires qui vous aideront à mettre à jour ce modèle ou à créer un modèle LUIS.

**Supprimer les ressources :** cet exemple crée un certain nombre d’applications et de ressources que vous pouvez supprimer à l’aide de la procédure ci-dessous. Veillez toutefois à ne pas supprimer les ressources dont dépendent *d’autres applications ou services*. 

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

**Bonne pratique :** Pour améliorer les services utilisés dans cet exemple, reportez-vous aux bonnes pratiques pour [LUIS](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/luis-concept-best-practices) et [QnA Maker](https://docs.microsoft.com/en-us/azure/cognitive-services/qnamaker/concepts/best-practices).