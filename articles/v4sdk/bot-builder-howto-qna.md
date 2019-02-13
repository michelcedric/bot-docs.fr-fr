---
title: Utiliser QnA Maker pour répondre aux questions | Microsoft Docs
description: Découvrez comment utiliser QnA Maker dans votre bot.
keywords: questions et réponses, QnA, FAQ, QnA Maker
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: cognitive-services
ms.date: 01/15/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: f1e3e3fa05a297aa50a2368a103a7aa00be49009
ms.sourcegitcommit: fd60ad0ff51b92fa6495b016e136eaf333413512
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 02/06/2019
ms.locfileid: "55764137"
---
# <a name="use-qna-maker-to-answer-questions"></a>Utiliser QnA Maker pour répondre aux questions

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Vous pouvez utiliser le service QnA Maker pour ajouter une prise en charge des questions et des réponses à votre bot. Une des exigences de base de la création de votre propre service QnA Maker est à l’amorçage des questions et des réponses. Très souvent, les questions et les réponses existent déjà dans des contenus comme une FAQ ou d’autres documents. Mais vous voudrez parfois personnaliser vos réponses aux questions afin de leur donner un ton plus naturel, comme dans une véritable conversation.

Dans cette rubrique, nous allons créer une base de connaissances et l’utiliser dans un bot.

## <a name="prerequisites"></a>Prérequis
- Compte [QnA Maker](https://www.qnamaker.ai/)
- Le code de cet article est basé sur l’exemple **QnA Maker**. Vous aurez besoin d’une copie de l’[exemple C# ](https://aka.ms/cs-qna) ou de l’[exemple Javascript](https://aka.ms/js-qna-sample).
- [Bot Framework Emulator](https://github.com/Microsoft/BotFramework-Emulator/blob/master/README.md#download)
- Connaissances des [concepts de base des bots](bot-builder-basics.md), de [QnA Maker](https://docs.microsoft.com/en-us/azure/cognitive-services/qnamaker/overview/overview), et du fichier [.bot](bot-file-basics.md).

## <a name="create-a-qna-maker-service-and-publish-a-knowledge-base"></a>Créer un service QnA Maker et publier une base de connaissances
1. Tout d’abord, vous devez créer un [service QnA Maker](https://docs.microsoft.com/en-us/azure/cognitive-services/qnamaker/how-to/set-up-qnamaker-service-azure).
1. Ensuite, vous allez créer une base de connaissances à l’aide du fichier `smartLightFAQ.tsv` situé dans le dossier CognitiveModels du projet. Les étapes permettant de créer, de former et de publier votre [base de connaissances](https://docs.microsoft.com/en-us/azure/cognitive-services/qnamaker/quickstarts/create-publish-knowledge-base) QnA Maker sont répertoriés dans la documentation de QnA Maker. Pendant que vous suivez ces étapes, nommez votre base de connaissances `qna`et utilisez le fichier `smartLightFAQ.tsv` pour la remplir.
> Remarque. Cet article peut également être utilisé pour accéder à votre propre base de connaissances QnA Maker développée par l’utilisateur.

## <a name="obtain-values-to-connect-your-bot-to-the-knowledge-base"></a>Obtenir des valeurs pour connecter votre bot à la base de connaissances
1. Sur le site [QnA Maker](https://www.qnamaker.ai/), sélectionnez votre base de connaissances.
1. Ouvrez votre base de connaissances et sélectionnez **Settings** (Paramètres). Enregistrez la valeur indiquée pour _service name_. Cette valeur est utile pour rechercher la base de connaissances qui vous intéresse quand vous utilisez l’interface du portail QnA Maker. Elle n’est pas utilisée pour connecter votre application bot à cette base de connaissances. 
1. Faites défiler la page jusqu’à la section **Deployment details** et enregistrez les valeurs suivantes :
   - POST /knowledgebases/<ID_Votre_Base_de_connaissances>/getAnswers
   - Host: <Votre_Nom_d’hôte>/qnamaker
   - Autorisation : EndpointKey <Votre_Clé_de_Point de terminaison>
   
Ces trois valeurs fournissent les informations nécessaires pour que votre application se connecte à votre base de connaissances QnA Maker par le biais de votre service Azure QnA.  

## <a name="update-the-bot-file"></a>Mettre à jour le fichier bot
Tout d’abord, ajoutez les informations requises pour accéder à votre base de connaissances, notamment le nom d’hôte,la clé de point de terminaison et l’ID de la base de connaissances (KbId) dans `qnamaker.bot`. Il s’agit des valeurs que vous avez enregistrées à partir des paramètres (**Settings**) de votre base de connaissances dans QnA Maker. 
> Remarque. Si vous ajoutez un accès à une base de connaissances QnA Maker dans une application bot existante, veillez à ajouter une section "type": "qna" comme celle illustrée ci-dessous dans votre fichier .bot. La valeur « name » de cette section fournit la clé nécessaire pour accéder à ces informations à partir de votre application.

```json
{
  "name": "qnamaker",
  "services": [
    {
      "type": "endpoint",
      "name": "development",
      "endpoint": "http://localhost:3978/api/messages",
      "appId": "",
      "appPassword": "",
      "id": "25"    
    },
    {
      "type": "qna",
      "name": "QnABot",
      "KbId": "<Your_Knowledge_Base_Id>",
      "subscriptionKey": "",
      "endpointKey": "<Your_Endpoint_Key>",
      "hostname": "<Your_Hostname>",
      "id": "117"
    }
  ],
  "padlock": "",
   "version": "2.0"
}
```

# <a name="ctabcs"></a>[C#](#tab/cs)
Ensuite, nous initialisons une nouvelle instance de la classe BotService dans **BotServices.cs**, qui extrait les informations ci-dessus à partir de votre fichier .bot. Le service externe est configuré à l’aide de la classe BotConfiguration.

```csharp
private static BotServices InitBotServices(BotConfiguration config)
{
    var qnaServices = new Dictionary<string, QnAMaker>();
    foreach (var service in config.Services)
    {
        switch (service.Type)
        {
            case ServiceTypes.QnA:
            {
                // Create a QnA Maker that is initialized and suitable for passing
                // into the IBot-derived class (QnABot).
                var qna = (QnAMakerService)service;
                if (qna == null)
                {
                    throw new InvalidOperationException("The QnA service is not configured correctly in your '.bot' file.");
                }

                if (string.IsNullOrWhiteSpace(qna.KbId))
                {
                    throw new InvalidOperationException("The QnA KnowledgeBaseId ('kbId') is required to run this sample. Please update your '.bot' file.");
                }

                if (string.IsNullOrWhiteSpace(qna.EndpointKey))
                {
                    throw new InvalidOperationException("The QnA EndpointKey ('endpointKey') is required to run this sample. Please update your '.bot' file.");
                }

                if (string.IsNullOrWhiteSpace(qna.Hostname))
                {
                    throw new InvalidOperationException("The QnA Host ('hostname') is required to run this sample. Please update your '.bot' file.");
                }

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
    var connectedServices = new BotServices(qnaServices);
    return connectedServices;
}
```

Ensuite, dans **QnABot.cs**, nous donnons cette instance de QnAMaker au bot. Si vous accédez à votre propre base de connaissances, modifiez le message _welcome_ (bienvenue) ci-dessous pour fournir des instructions initiales utiles à vos utilisateurs. Cette classe est également l’endroit où la variable statique _QnAMakerKey_ est définie. L’emplacement pointé est la section de votre fichier .bot contenant les informations de connexion pour accéder à votre base de connaissances QnA Mkaer.

```csharp
public class QnABot : IBot
{
    public static readonly string QnAMakerKey = "QnABot";
    private const string WelcomeText = @"This bot will introduce you to QnA Maker.
                                         Ask a question to get started.";
    private readonly BotServices _services;
    public QnABot(BotServices services)
    {
        _services = services ?? throw new System.ArgumentNullException(nameof(services));
        Console.WriteLine($"{_services}");
        if (!_services.QnAServices.ContainsKey(QnAMakerKey))
        {
            throw new System.ArgumentException($"Invalid configuration. Please check your '.bot' file for a QnA service named '{QnAMakerKey}'.");
        }
    }
}
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

Dans notre exemple, le code de démarrage se trouve dans un fichier **index.js**, le code de la logique du bot se trouve dans un fichier **bot.js**, et les informations de configuration supplémentaires se trouvent dans le fichier **qnamaker.bot**.

Dans le fichier **index.js**, lisons les informations de configuration servant à générer le service QnA Maker et à initialiser le bot.

Mettez à jour la valeur de `QNA_CONFIGURATION` avec la valeur « name: » de votre fichier .bot. Il s’agit de la clé dans la section "type": "qna" de votre fichier .bot contenant des paramètres de connexion pour accéder à votre base de connaissances QnA Maker.

```js
// Name of the QnA Maker service in the .bot file. 
const QNA_CONFIGURATION = '<BOT_FILE_NAME>';

// Get endpoint and QnA Maker configurations by service name.
const endpointConfig = botConfig.findServiceByNameOrId(BOT_CONFIGURATION);
const qnaConfig = botConfig.findServiceByNameOrId(QNA_CONFIGURATION);

// Map the contents to the required format for `QnAMaker`.
const qnaEndpointSettings = {
    knowledgeBaseId: qnaConfig.kbId,
    endpointKey: qnaConfig.endpointKey,
    host: qnaConfig.hostname
};

// Create adapter...

// Create the QnAMakerBot.
let bot;
try {
    bot = new QnAMakerBot(qnaEndpointSettings, {});
} catch (err) {
    console.error(`[botInitializationError]: ${ err }`);
    process.exit();
}
```

Créons ensuite le serveur HTTP qui écoutera les requêtes entrantes et générera les appels à la logique de notre bot.

```js
// Create HTTP server.
let server = restify.createServer();
server.listen(process.env.port || process.env.PORT || 3978, function() {
    console.log(`\n${ server.name } listening to ${ server.url }.`);
    console.log(`\nGet Bot Framework Emulator: https://aka.ms/botframework-emulator.`);
    console.log(`\nTo talk to your bot, open qnamaker.bot file in the emulator.`);
});

// Listen for incoming requests.
server.post('/api/messages', (req, res) => {
    adapter.processActivity(req, res, async (turnContext) => {
        await bot.onTurn(turnContext);
    });
});
```

---

## <a name="calling-qna-maker-from-your-bot"></a>Appel de QnA Maker à partir de votre bot

# <a name="ctabcs"></a>[C#](#tab/cs)

Lorsque votre bot a besoin d’une réponse de QnA Maker, appelez `GetAnswersAsync()` à partir du code de votre bot pour obtenir la réponse appropriée en fonction du contexte actuel. Si vous accédez à votre propre base de connaissances, modifiez le message _no answers_ (aucune réponse) ci-dessous pour fournir des instructions initiales utiles à vos utilisateurs.

```csharp
// Check QnA Maker model
var response = await _services.QnAServices[QnAMakerKey].GetAnswersAsync(turnContext);
if (response != null && response.Length > 0)
{
    await turnContext.SendActivityAsync(response[0].Answer, cancellationToken: cancellationToken);
}
else
{
    var msg = @"No QnA Maker answers were found. This example uses a QnA Maker Knowledge Base that focuses on smart light bulbs.
                To see QnA Maker in action, ask the bot questions like 'Why won't it turn on?' or 'I need help'.";
    await turnContext.SendActivityAsync(msg, cancellationToken: cancellationToken);
}

    /// ...
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

Dans le fichier **bot.js**, nous passons l’entrée de l’utilisateur dans la méthode `getAnswers` du service QnA Maker pour obtenir des réponses à partir de la base de connaissances. Si vous accédez à votre propre base de connaissances, modifiez les messages _no answers_ (aucune réponse) et _welcome_ (bienvenue) ci-dessous pour fournir des instructions initiales utiles à vos utilisateurs.

```javascript
const { ActivityTypes, TurnContext } = require('botbuilder');
const { QnAMaker, QnAMakerEndpoint, QnAMakerOptions } = require('botbuilder-ai');

/**
 * A simple bot that responds to utterances with answers from QnA Maker.
 * If an answer is not found for an utterance, the bot responds with help.
 */
class QnAMakerBot {
    /**
     * The QnAMakerBot constructor requires one argument (`endpoint`) which is used to create an instance of `QnAMaker`.
     * @param {QnAMakerEndpoint} endpoint The basic configuration needed to call QnA Maker. In this sample the configuration is retrieved from the .bot file.
     * @param {QnAMakerOptions} config An optional parameter that contains additional settings for configuring a `QnAMaker` when calling the service.
     */
    constructor(endpoint, qnaOptions) {
        this.qnaMaker = new QnAMaker(endpoint, qnaOptions);
    }

    /**
     * Every conversation turn for our QnAMakerBot will call this method.
     * @param {TurnContext} turnContext Contains all the data needed for processing the conversation turn.
     */
    async onTurn(turnContext) {
        // By checking the incoming Activity type, the bot only calls QnA Maker in appropriate cases.
        if (turnContext.activity.type === ActivityTypes.Message) {
            // Perform a call to the QnA Maker service to retrieve matching Question and Answer pairs.
            const qnaResults = await this.qnaMaker.getAnswers(turnContext.activity.text);

            // If an answer was received from QnA Maker, send the answer back to the user.
            if (qnaResults[0]) {
                await turnContext.sendActivity(qnaResults[0].answer);

            // If no answers were returned from QnA Maker, reply with help.
            } else {
                await turnContext.sendActivity('No QnA Maker answers were found. This example uses a QnA Maker Knowledge Base that focuses on smart light bulbs. To see QnA Maker in action, ask the bot questions like "Why won\'t it turn on?" or "I need help."');
            }

        // If the Activity is a ConversationUpdate, send a greeting message to the user.
        } else if (turnContext.activity.type === ActivityTypes.ConversationUpdate &&
                   turnContext.activity.recipient.id !== turnContext.activity.membersAdded[0].id) {
            await turnContext.sendActivity('Welcome to the QnA Maker sample! Ask me a question and I will try to answer it.');

        // Respond to all other Activity types.
        } else if (turnContext.activity.type !== ActivityTypes.ConversationUpdate) {
            await turnContext.sendActivity(`[${ turnContext.activity.type }]-type activity detected.`);
        }
    }
}

module.exports.QnAMakerBot = QnAMakerBot;
```

---

## <a name="test-the-bot"></a>Tester le bot

Exécutez l’exemple en local sur votre machine. Si vous avez besoin d’instructions, consultez le fichier Lisez-moi pour l’exemple [ C# ](https://github.com/Microsoft/BotBuilder-Samples/tree/master/samples/csharp_dotnetcore/11.qnamaker) ou [JS](https://github.com/Microsoft/BotBuilder-Samples/blob/master/samples/javascript_nodejs/11.qnamaker/README.md).

Dans l’émulateur, envoyer un message au bot comme indiqué ci-dessous.

![exemple de qna test](~/media/emulator-v4/qna-test-bot.png)


## <a name="next-steps"></a>Étapes suivantes

Vous pouvez combiner QnA Maker à d’autres services Cognitive Services pour rendre votre bot encore plus puissant. L’outil Dispatch offre un moyen de combiner QnA avec Language Understanding (LUIS) dans votre bot.

> [!div class="nextstepaction"]
> [Combiner des services QnA et LUIS à l’aide de l’outil Dispatch](./bot-builder-tutorial-dispatch.md)
