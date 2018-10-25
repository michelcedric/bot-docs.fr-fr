---
title: Utiliser QnA Maker pour répondre aux questions | Microsoft Docs
description: Découvrez comment utiliser QnA Maker dans votre bot.
keywords: questions et réponses, QnA, FAQ, intergiciel (middleware)
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: cognitive-services
ms.date: 10/08/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 3d488cc2bb61ef460ed45707596cb7db9e6c23e8
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2018
ms.locfileid: "49999078"
---
# <a name="use-qna-maker-to-answer-questions"></a>Utiliser QnA Maker pour répondre aux questions

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Vous pouvez utiliser le service QnA Maker pour ajouter une prise en charge des questions et des réponses à votre bot. Une des exigences de base de la création de votre propre service QnA Maker est à l’amorçage des questions et des réponses. Très souvent, les questions et les réponses existent déjà dans des contenus comme une FAQ ou d’autres documents. Mais vous voudrez parfois personnaliser vos réponses aux questions afin de leur donner un ton plus naturel, comme dans une véritable conversation.

## <a name="prerequisites"></a>Prérequis
- Créer un compte [QnA Maker](https://www.qnamaker.ai/)
- Télécharger l’exemple QnA Maker [[C#](https://aka.ms/cs-qna) | [JavaScript](https://aka.ms/js-qna-sample)]

## <a name="create-a-qna-maker-service-and-publish-a-knowledge-base"></a>Créer un service QnA Maker et publier une base de connaissances

Après avoir créé le compte QnA Maker, suivez les instructions de création du [service QnA Maker](https://docs.microsoft.com/en-us/azure/cognitive-services/qnamaker/how-to/set-up-qnamaker-service-azure) et de la [base de connaissances](https://docs.microsoft.com/en-us/azure/cognitive-services/qnamaker/quickstarts/create-publish-knowledge-base). 

Après la publication de votre base de connaissances, vous devez enregistrer les valeurs suivantes par programme pour y connecter votre bot.
- Sur le site [QnA Maker](https://www.qnamaker.ai/), sélectionnez votre base de connaissances.
- Ouvrez votre base de connaissances et sélectionnez **Settings** (Paramètres). Enregistrez la valeur indiquée pour _service name_ (nom du service) en tant que <your_kb_name>.
- Faites défiler la page jusqu’à la section **Deployment details** (Informations sur le déploiement) et enregistrez les valeurs suivantes :
   - POST /knowledgebases/<your_knowledge_base_id>/generateAnswer
   - Host: https://<you_hostname>.azurewebsites.net/qnamaker
   - Authorization: EndpointKey <your_endpoint_key>

## <a name="installing-packages"></a>Installation des packages

Avant de passer au codage, assurez-vous que vous avez installé les packages nécessaires pour QnA Maker.

# <a name="ctabcs"></a>[C#](#tab/cs)

Installez le [package NuGet](https://docs.microsoft.com/en-us/nuget/tools/package-manager-ui) suivant dans votre bot.

* `Microsoft.Bot.Builder.AI.QnA`

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

Les fonctionnalités QnA Maker se trouvent dans le package `botbuilder-ai`. Vous pouvez ajouter ce package à votre projet par le biais de npm :

```shell
npm install --save botbuilder-ai
```

---

## <a name="using-cli-tools-to-update-your-bot-configuration"></a>Utilisation des outils d’interface de ligne de commande pour mettre à jour votre fichier de configuration .bot

Une autre méthode permettant d’obtenir les valeurs d’accès à votre base de connaissances consiste à utiliser les outils CLI BotBuilder [qnamaker](https://aka.ms/botbuilder-tools-qnaMaker) et [msbot](https://aka.ms/botbuilder-tools-msbot-readme) pour obtenir ses métadonnées et les ajouter à votre fichier .bot.

1. Ouvrez un terminal ou une invite de commandes et accédez au répertoire racine du projet de votre bot.
2. Exécutez `qnamaker init` pour créer un fichier de ressources QnA Maker (**.qnamakerrc**). Vous êtes invité à fournir votre clé d’abonnement QnA Maker.
3. Exécutez la commande suivante pour télécharger vos métadonnées et les ajouter au fichier de configuration de votre bot.

    ```shell
    qnamaker get kb --kbId <your-kb-id> --msbot | msbot connect qna --stdin [--secret <your-secret>]
    ```
Si vous avez chiffré votre fichier de configuration, vous devrez fournir votre clé secrète pour mettre à jour le fichier.

## <a name="using-qna-maker"></a>Utilisation de QnA Maker
Une référence à QnA Maker est d’abord ajoutée lors de l’initialisation du bot. Nous pouvons ensuite l’appeler dans la logique de notre bot.

# <a name="ctabcs"></a>[C#](#tab/cs)
Ouvrez l’exemple QnA Maker que vous avez téléchargé précédemment. Nous allons modifier ce code en fonction de nos besoins.
Tout d’abord, ajoutez les informations requises pour accéder à votre base de connaissances, notamment le nom d’hôte,la clé de point de terminaison et l’ID de la base de connaissances (KbId) dans `BotConfiguration.bot`. Il s’agit des valeurs que vous avez enregistrées à partir des paramètres (**Settings**) de votre base de connaissances dans QnA Maker.

```json
{
  "name": "QnABotSample",
  "services": [
    {
      "type": "endpoint",
      "name": "development",
      "endpoint": "http://localhost:3978/api/messages",
      "appId": "",
      "id": "1",
      "appPassword": ""
    },
    {
      "type": "qna",
      "name": "QnABot",
      "KbId": "<YOUR_KNOWLEDGE_BASE_ID>",
      "Hostname": "https://<YOUR_HOSTNAME>.azurewebsites.net/qnamaker",
      "EndpointKey": "<YOUR_ENDPOINT_KEY>"
    }
  ],
  "version": "2.0",
  "padlock": ""
}
```

Ensuite, nous allons créer une instance de QnA Maker dans `Startup.cs`. Elle va récupérer les informations mentionnées ci-dessus dans le fichier `BotConfiguration.bot`. Ces chaînes peuvent également être codées en dur pour les tests.

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

Ensuite, nous devons donner à votre bot cette instance de QnA Maker. Ouvrez `QnABot.cs` et ajoutez le code suivant au début du fichier. Si vous accédez à votre propre base de connaissances, modifiez le message _welcome_ (bienvenue) ci-dessous pour fournir des instructions initiales utiles à vos utilisateurs.

```csharp
public class QnABot : IBot
{
    public static readonly string QnAMakerKey = "QnABot";
    private const string WelcomeText = @"This bot will introduce you to QnA Maker.
                                         Ask a quesiton to get started.";
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
Ouvrez l’exemple QnA Maker que vous avez téléchargé précédemment. Nous allons modifier ce code en fonction de nos besoins.
Dans notre exemple, le code de démarrage se trouve dans un fichier **index.js**, le code de la logique du bot se trouve dans un fichier **bot.js**, et les informations de configuration supplémentaires se trouvent dans le fichier **qnamaker.bot**.

Une fois les instructions de création de votre base de connaissances et de mise à jour de votre fichier **.bot** suivies, votre fichier **qnamaker.bot** devrait inclure une entrée de service correspondant à votre base de connaissances QnA Maker.

```json
{
    "name": "qnamaker",
    "description": "",
    "services": [
        {
            "type": "endpoint",
            "name": "development",
            "id": "1",
            "appId": "",
            "appPassword": "",
            "endpoint": "http://localhost:3978/api/messages"
        },
        {
            "type": "qna",
            "name": "<YOUR_KB_NAME>",
            "kbId": "<YOUR_KNOWLEDGE_BASE_ID>",
            "endpointKey": "<YOUR_ENDPOINT_KEY>",
            "hostname": "https://<YOUR_HOSTNAME>.azurewebsites.net/qnamaker",
            "id": "221"
        }
    ],
    "padlock": "",
    "version": "2.0"
}
```

Dans le fichier **index.js**, lisons les informations de configuration servant à générer le service QnA Maker et à initialiser le bot.

Remplacez la valeur de `QNA_CONFIGURATION` par le nom de votre base de connaissances, tel qu’il apparaît dans votre fichier de configuration.

```js
// QnA Maker knowledge base name as specified in .bot file.
const QNA_CONFIGURATION = '<YOUR_KB_NAME>';

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

Dans le fichier **bot.js**, nous passons l’entrée de l’utilisateur dans la méthode `generateAnswer` du service QnA Maker pour obtenir des réponses à partir de la base de connaissances. Si vous accédez à votre propre base de connaissances, modifiez les messages _no answers_ (aucune réponse) et _welcome_ (bienvenue) ci-dessous pour fournir des instructions initiales utiles à vos utilisateurs.

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
            const qnaResults = await this.qnaMaker.generateAnswer(turnContext.activity.text);

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

Posez des questions à votre bot pour afficher les réponses de votre service QnA Maker. Pour plus d’informations sur le test et la publication de votre service QnA, consultez l’article QnA Maker sur le [test d’une base de connaissances](https://docs.microsoft.com/en-us/azure/cognitive-services/qnamaker/how-to/test-knowledge-base).

## <a name="next-steps"></a>Étapes suivantes

Vous pouvez combiner QnA Maker à d’autres services Cognitive Services pour rendre votre bot encore plus puissant. L’outil Dispatch offre un moyen de combiner QnA avec Language Understanding (LUIS) dans votre bot.

> [!div class="nextstepaction"]
> [Combiner des services QnA et LUIS à l’aide de l’outil Dispatch](./bot-builder-tutorial-dispatch.md)
