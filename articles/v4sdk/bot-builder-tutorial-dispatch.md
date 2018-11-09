---
title: Utiliser les services LUIS et QnA avec l’outil Dispatch | Microsoft Docs
description: Découvrez comment utiliser LUIS et QnA Maker dans votre bot.
keywords: Luis, QnA, outil Dispatch, services multiples
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 10/31/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 4d029dc7361ac8a7fadb61141faf60d8a62eab3c
ms.sourcegitcommit: a496714fb72550a743d738702f4f79e254c69d06
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/01/2018
ms.locfileid: "50736677"
---
# <a name="use-luis-and-qna-services-with-the-dispatch-tool"></a>Utiliser les services LUIS et QnA avec l’outil Dispatch

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

<!--TODO: Add the JS sections back in and update them once the JS sample is working.-->

Ce tutoriel montre comment utiliser un modèle LUIS généré par l’outil Dispatch, pour intégrer votre bot à plusieurs applications Language Understanding (LUIS) et services QnAMaker. Cet exemple combine les services suivants.

| Type de service | NOM | Description |
|------|------|------|
| Application LUIS | HomeAutomation | Reconnaît une intention domotique avec les données d’entité associées.|
| Application LUIS | Météo | Reconnaît les intentions Weather.GetForecast et Weather.GetCondition avec les données de localisation.|
| Service QnAMaker | Forum Aux Questions  | Fournit des réponses à certaines questions simples concernant le bot. |

Le code présenté dans cet article est extrait de l’exemple **Traitement du langage naturel avec Dispatch** [[C#](https://aka.ms/dispatch-sample-cs)].

<!-- | [JS](https://aka.ms/dispatch-sample-js)-->

Consultez la section [Language Understanding](bot-builder-concept-luis.md) pour une vue d’ensemble des services de langage. Consultez les articles sur [LUIS](bot-builder-howto-v4-luis.md) et [QnA Maker](bot-builder-howto-qna.md) pour obtenir des instructions sur l’implémentation de ces applications et services dans un bot.

Vous pouvez suivre les instructions du fichier README pour configurer et tester le bot, ou vous pouvez passer directement aux [remarques supplémentaires sur le code](#notes-about-the-code).

## <a name="create-the-services-and-test-the-bot"></a>Créer les services et tester le bot

Suivez les instructions du fichier README pour obtenir l’exemple. Vous allez utiliser des outils CLI pour créer et publier ces services et mettre à jour les informations les concernant dans votre fichier de configuration (**.bot**).

1. Clonez ou extrayez le référentiel d’exemples.
1. Installez les outils CLI BotBuilder V4.
1. Configurez manuellement les services requis.

### <a name="test-your-bot"></a>Tester votre robot

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Démarrez votre bot à l’aide de Visual Studio ou de Visual Studio Code.

<!--
# [JavaScript](#tab/javascript)
-->

---

Connectez-vous à votre bot à l’aide de l’émulateur Bot Framework.

Voici quelques entrées couvertes par les services que nous avons inclus :

* QnA Maker
  * `hi`, `good morning`
  * `what are you`, `what do you do`
* LUIS (domotique)
  * `turn on bedroom light`
  * `turn off bedroom light`
  * `make some coffee`
* LUIS (météo)
  * `whats the weather in chennai india`
  * `what's the forecast for bangalore`
  * `show me the forecast for nebraska`

## <a name="notes-about-the-code"></a>Remarques supplémentaires concernant le code

### <a name="packages"></a>Packages

Cet exemple utilise les packages suivants.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Les dernières versions v4 de ces [packages NuGet](https://docs.microsoft.com/en-us/nuget/tools/package-manager-ui#updating-a-package).

* `Microsoft.Bot.Builder`
* `Microsoft.Bot.Builder.AI.Luis`
* `Microsoft.Bot.Builder.AI.QnA`
* `Microsoft.Bot.Builder.Integration.AspNet.Core`
* `Microsoft.Bot.Configuration`

<!--
# [JavaScript](#tab/javascript)

Download the [LUIS Dispatch sample][DispatchBotJs].  Install the required packages, including the `botbuilder-ai` package for LUIS and QnA Maker, using npm:

* `npm install --save botbuilder`
* `npm install --save botbuilder-ai`
-->

---

### <a name="botbuilder-cli-tools"></a>Outils CLI BotBuilder

L’exemple utilise ces [outils CLI BotBuilder](https://aka.ms/botbuilder-tools-readme) (disponible via npm) pour créer, entraîner et publier les services LUIS, QnA Maker et Dispatch, et pour enregistrer des informations sur ces services dans le fichier de configuration de votre bot (**.bot**).

* [Dispatch](https://aka.ms/botbuilder-tools-dispatch)
* [LUDown](https://aka.ms/botbuilder-tools-ludown-readme)
* [LUIS](https://aka.ms/botbuilder-tools-luis)
* [MSBot](https://aka.ms/botbuilder-tools-msbot-readme)
* [QnAMaker](https://aka.ms/botbuilder-tools-qnaMaker)

> [!TIP]
> Pour vous assurer que vous avez la dernière version de npm et de ces outils CLI, exécutez la commande suivante.
>
> ```shell
> npm i -g npm dispatch ludown luis-apis msbot qnamaker
> ```

Une fois que vous avez utilisé ces outils pour configurer les services, le fichier **.bot** pour cet exemple devrait ressembler à ce qui suit. (Vous pouvez exécuter `msbot secret -n` pour chiffrer les valeurs sensibles de ce fichier.)

```json
{
    "name": "NLP-With-Dispatch-Bot",
    "description": "",
    "services": [
        {
            "type": "endpoint",
            "name": "development",
            "id": "http://localhost:3978/api/messages",
            "appId": "",
            "appPassword": "",
            "endpoint": "http://localhost:3978/api/messages"
        },
        {
            "type": "luis",
            "name": "Home Automation",
            "appId": "<your-home-automation-luis-app-id>",
            "version": "0.1",
            "authoringKey": "<your-luis-authoring-key>",
            "subscriptionKey": "<your-cognitive-services-subscription-key>",
            "region": "westus",
            "id": "110"
        },
        {
            "type": "luis",
            "name": "Weather",
            "appId": "<your-weather-luis-app-id>",
            "version": "0.1",
            "authoringKey": "<your-luis-authoring-key>",
            "subscriptionKey": "<your-cognitive-services-subscription-key>",
            "region": "westus",
            "id": "92"
        },
        {
            "type": "qna",
            "name": "Sample QnA",
            "kbId": "<your-qna-knowledge-base-id>",
            "subscriptionKey": "<your-cognitive-services-subscription-key>",
            "endpointKey": "<your-qna-endpoint-key>",
            "hostname": "<your-qna-host-name>",
            "id": "184"
        },
        {
            "type": "dispatch",
            "name": "NLP-With-Dispatch-BotDispatch",
            "appId": "<your-dispatch-app-id>",
            "authoringKey": "<your-luis-authoring-key>",
            "subscriptionKey": "<your-cognitive-services-subscription-key>",
            "version": "Dispatch",
            "region": "westus",
            "serviceIds": [
                "110",
                "92",
                "184"
            ],
            "id": "27"
        }
    ],
    "padlock": "",
    "version": "2.0"
}
```

### <a name="connecting-to-the-services-from-your-bot"></a>Connexion aux services à partir de votre bot

Pour vous connecter aux services Dispatch, LUIS et QnA Maker, votre bot extrait les informations du fichier **.bot**.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Dans **Startup.cs**, `ConfigureServices` lit le fichier de configuration, et `InitBotServices` utilise ces informations pour initialiser les services. Chaque fois qu’il est créé, le bot est initialisé avec l’objet `BotServices` inscrit. Voici les parties pertinentes de ces deux méthodes.

```csharp
public void ConfigureServices(IServiceCollection services)
{
    //...
    var botConfig = BotConfiguration.Load(botFilePath ?? @".\BotConfiguration.bot", secretKey);
    services.AddSingleton(sp => botConfig
        ?? throw new InvalidOperationException($"The .bot config file could not be loaded. ({botConfig})"));

    // Retrieve current endpoint.
    var environment = _isProduction ? "production" : "development";
    var service = botConfig.Services.Where(s => s.Type == "endpoint" && s.Name == environment).FirstOrDefault();
    if (!(service is EndpointService endpointService))
    {
        throw new InvalidOperationException($"The .bot file does not contain an endpoint with name '{environment}'.");
    }

    var connectedServices = InitBotServices(botConfig);

    services.AddSingleton(sp => connectedServices);
    //...
}
```

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
/// <summary>
/// Depending on the intent from Dispatch, routes to the right LUIS model or QnA service.
/// </summary>
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

/// <summary>
/// Dispatches the turn to the request QnAMaker app.
/// </summary>
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

/// <summary>
/// Dispatches the turn to the requested LUIS model.
/// </summary>
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

L’exécution de `dispatch eval` génère un fichier **Summary.html** qui fournit des statistiques sur les performances prédites du modèle de langage.

> [!TIP]
> Vous pouvez exécuter `dispatch eval` sur n’importe quelle application LUIS, pas seulement sur les applications LUIS créées par l’outil Dispatch.

### <a name="edit-intents-for-duplicates-and-overlaps"></a>Modifier les intentions en cas de doublons et de redondances

Passez en revue les exemples d’énoncés marqués comme étant des doublons dans **Summary.html** et supprimez les exemples similaires ou redondants. Par exemple, supposons que dans l’application LUIS `Home Automation`, des demandes comme « allumer la lumière » correspondent à une intention « TurnOnLights » (Allumer la lumière), mais que des demandes comme « Pourquoi la lumière ne s’allume-t-elle pas ? » correspondent à une intention « None » (Aucune) afin qu’elles puissent être transmises à QnA Maker. Quand vous combinez l’application LUIS et le service QnA Maker à l’aide de l’outil Dispatch, vous devez effectuer l’une des opérations suivantes :

* Supprimez l’intention « None » (Aucune) de l’application LUIS `Home Automation` d’origine et ajoutez les énoncés dans cette intention à l’intention « None » (Aucune) dans l’application de distribution.
* Si vous ne supprimez pas l’intention « None » (Aucune) de l’application LUIS d’origine, vous devez ajouter de la logique à votre bot pour transmettre les messages qui correspondent à cette intention au service QnA Maker.

> [!TIP]
> Consultez [Bonnes pratiques pour Language Understanding](./bot-builder-concept-luis.md#best-practices-for-language-understanding) pour obtenir des conseils sur l’amélioration des performances de votre modèle de langage.

## <a name="to-clean-up-resources-from-this-sample"></a>Pour nettoyer les ressources de cet exemple

Cet exemple crée un certain nombre d’applications et de ressources. Vous pouvez suivre ces instructions pour les supprimer.

### <a name="luis-resources"></a>Ressources LUIS

1. Connectez-vous au portail [luis.ai](https://www.luis.ai).
1. Accédez à la page **My Apps** (Mes applications).
1. Sélectionnez les applications créées par cet exemple.
   * `Home Automation`
   * `Weather`
   * `NLP-With-Dispatch-BotDispatch`
1. Cliquez sur **Delete** (Supprimer), puis sur **Ok** pour confirmer.

### <a name="qna-maker-resources"></a>Ressources QnA Maker

1. Connectez-vous au portail [qnamaker.ai](https://www.qnamaker.ai/).
1. Accédez à la page **My knowledge bases** (Mes bases de connaissances).
1. Cliquez sur le bouton de suppression de la base de connaissances `Sample QnA`, puis cliquez sur **Delete** (Supprimer) pour confirmer.

### <a name="azure-resources"></a>Ressources Azure

> [!WARNING]
> Ne supprimez pas les ressources desquelles dépendent d’autres applications ou services.

1. Connectez-vous au [Portail Azure](https://portal.azure.com/).
1. Accédez à la page **Vue d’ensemble** de la ressource Cognitive Services que vous avez créée pour l’exemple.
1. Cliquez sur **Supprimer**, puis sur **Oui** pour confirmer.

## <a name="additional-resources"></a>Ressources supplémentaires

* [Language Understanding](bot-builder-concept-luis.md)
* [Concevoir des bots de connaissances](../bot-service-design-pattern-knowledge-base.md)
* [Configurer la préparation vocale](../bot-service-manage-speech-priming.md)
* [Utilisation de LUIS pour la compréhension langagière](bot-builder-howto-v4-luis.md)
* [Utiliser QnA Maker pour répondre aux questions](bot-builder-howto-qna.md)
