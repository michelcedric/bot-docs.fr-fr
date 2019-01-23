---
title: Tutoriel Azure Bot Service pour que votre bot réponde aux questions | Microsoft Docs
description: Tutoriel sur l’utilisation de QnA Maker pour que votre bot réponde à des questions.
keywords: QnA Maker, question et réponse, base de connaissances
author: JonathanFingold
ms.author: v-jofing
manager: kamrani
ms.topic: tutorial
ms.service: bot-service
ms.subservice: sdk
ms.date: 01/15/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: b1c34531ee60b2ce9037f42e4f5a7093501cf83a
ms.sourcegitcommit: bdb981c0b11ee99d128e30ae0462705b2dae8572
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/17/2019
ms.locfileid: "54360944"
---
# <a name="tutorial-use-qna-maker-in-your-bot-to-answer-questions"></a>Tutoriel : Utiliser QnA Maker dans votre bot pour répondre à des questions

Vous pouvez utiliser le service QnA Maker et une base de connaissances pour ajouter une prise en charge des questions et réponses à votre bot. Quand vous créez une base de connaissances, vous l’amorcez avec des questions et réponses.

Ce tutoriel vous montre comment effectuer les opérations suivantes :

> [!div class="checklist"]
> * Créer un service QnA Maker et une base de connaissances
> * Ajouter les informations de la base de connaissances à votre fichier .bot
> * Mettre à jour votre votre bot pour qu’il interroge la base de connaissances
> * Republier le bot

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

## <a name="prerequisites"></a>Prérequis

* Le bot créé dans le [tutoriel précédent](bot-builder-tutorial-basic-deploy.md). Nous allons ajouter une fonctionnalité de questions et réponses au bot.
* Il est utile de connaître les bases de QnA Maker. Nous allons créer, entraîner et publier la base de connaissances à utiliser avec le bot via le portail QnA Maker.

Vous devez déjà respecter les prérequis du tutoriel précédent :

[!INCLUDE [deployment prerequisites snippet](~/includes/deploy/snippet-prerequisite.md)]

## <a name="sign-in-to-qna-maker-portal"></a>Se connecter au portail QnA Maker

<!-- This and the next step are close duplicates of what's in the QnA How-To -->

Connectez-vous au [portail QnA Maker](https://qnamaker.ai/) avec vos informations d’identification Azure.

## <a name="create-a-qna-maker-service-and-knowledge-base"></a>Créer un service QnA Maker et une base de connaissances

Nous allons importer une définition de base de connaissances existante de l’exemple QnA Maker dans le dépôt [BotBuilder/Microsoft-Samples](https://github.com/Microsoft/BotBuilder-Samples).

1. Clonez ou copiez le dépôt d’exemples sur votre ordinateur.
1. Sur le portail QnA Maker, sélectionnez **Create a knowledge base** (Créer une base de connaissances).
   1. Si nécessaire, créez un service QnA. (Vous pouvez utiliser un service QnA Maker existant ou en créer un pour ce tutoriel.) Pour des instructions plus détaillées sur QnA Maker, consultez [Créer un service QnA Maker](https://docs.microsoft.com/en-us/azure/cognitive-services/qnamaker/how-to/set-up-qnamaker-service-azure) et [Créer, entraîner et publier votre base de connaissances QnA Maker](https://docs.microsoft.com/en-us/azure/cognitive-services/qnamaker/quickstarts/create-publish-knowledge-base).
   1. Connectez votre service QnA à votre base de connaissances.
   1. Nommez votre base de connaissances.
   1. Pour remplir votre base de connaissances, utilisez le fichier **BotBuilder-Samples\samples\csharp_dotnetcore\11.qnamaker\CognitiveModels\smartLightFAQ.tsv** à partir du dépôt d’exemples.
   1. Cliquez sur **Create your KB** (Créer votre base de connaissances) pour créer la base de connaissances.
1. **Enregistrez et entraînez** votre base de connaissances.
1. **Publiez** votre base de connaissances.

   La base de connaissances est maintenant prête à être utilisée par votre bot. Inscrivez l’ID de la base de connaissances, la clé du point de terminaison et le nom d’hôte. Vous en aurez besoin à l’étape suivante.

## <a name="add-knowledge-base-information-to-your-bot-file"></a>Ajouter les informations de la base de connaissances à votre fichier .bot

Ajoutez à votre fichier .bot les informations nécessaires pour accéder à votre base de connaissances.

1. Ouvrez le fichier .bot dans un éditeur.
1. Ajoutez un élément `qna` au tableau `services`.

    ```json
    {
        "type": "qna",
        "name": "<your-knowledge-base-name>",
        "kbId": "<your-knowledge-base-id>",
        "hostname": "<your-qna-service-hostname>",
        "endpointKey": "<your-knowledge-base-endpoint-key>",
        "subscriptionKey": "<your-azure-subscription-key>",
        "id": "<a-unique-id>"
    }
    ```

    | Champ | Valeur |
    |:----|:----|
    | type | Doit être `qna`. Indique que cette entrée de service décrit une base de connaissances QnA. |
    | name | Nom que vous avez attribué à votre votre base de connaissances. |
    | kbId | ID de base de connaissances que le portail QnA Maker a généré automatiquement. |
    | hostname | URL d’hôte généré par le portail QnA Maker. Utilisez l’URL complète, avec `https://` au début et `/qnamaker` à la fin. |
    | endpointKey | Clé du point de terminaison que le portail QnA Maker a générée automatiquement. |
    | subscriptionKey | ID de l’abonnement que vous avez utilisé pour créer le service QnA Maker dans Azure. |
    | id | ID unique pas déjà utilisé pour l’un des autres services listés dans votre fichier .bot, comme « 201 ». |

1. Enregistrez vos modifications.

## <a name="update-your-bot-to-query-the-knowledge-base"></a>Mettre à jour votre votre bot pour qu’il interroge la base de connaissances

Mettez à jour votre code d’initialisation pour charger les informations du service pour votre base de connaissances.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

1. Ajoutez le package NuGet **Microsoft.Bot.Builder.AI.QnA** à votre projet.
1. Renommez la classe qui implémente **IBot** en `QnaBot`.
1. Renommez la classe qui contient les accesseurs de votre robot en `QnaBotAccessors`.
1. Dans votre fichier **Startup.cs**, ajoutez ces références d’espace de noms.
    ```csharp
    using System.Collections.Generic;
    using System.Linq;
    using Microsoft.Bot.Builder.AI.QnA;
    using Microsoft.Bot.Builder.Integration;
    ```
1. Ensuite, modifiez la méthode **ConfigureServices** pour initialiser et inscrire les bases de connaissances définies dans le fichier **.bot**. Notez que ces premières lignes, qui étaient placées dans le corps de l’appel `services.AddBot<QnaBot>(options =>`, le précèdent désormais.
    ```csharp
    public void ConfigureServices(IServiceCollection services)
    {
        var secretKey = Configuration.GetSection("botFileSecret")?.Value;
        var botFilePath = Configuration.GetSection("botFilePath")?.Value;

        // Loads .bot configuration file and adds a singleton that your Bot can access through dependency injection.
        var botConfig = BotConfiguration.Load(botFilePath ?? @".\jfEchoBot.bot", secretKey);
        services.AddSingleton(sp => botConfig ?? throw new InvalidOperationException($"The .bot config file could not be loaded. ({botConfig})"));

        // Initialize the QnA knowledge bases for the bot.
        services.AddSingleton(sp => {
            var qnaServices = new List<QnAMaker>();
            foreach (var qnaService in botConfig.Services.OfType<QnAMakerService>())
            {
                qnaServices.Add(new QnAMaker(qnaService));
            }
            return qnaServices;
        });

        services.AddBot<QnaBot>(options =>
        {
            // Retrieve current endpoint.
            // ...
        });

        // Create and register state accessors.
        // ...
    }
    ```
1. Dans votre fichier **QnaBot.cs**, ajoutez ces références d’espace de noms.
    ```csharp
    using System.Collections.Generic;
    using Microsoft.Bot.Builder.AI.QnA;
    ```
1. Ajoutez une propriété `_qnaServices` et initialisez-la dans le constructeur du bot.
    ```csharp
    private readonly List<QnAMaker> _qnaServices;

    /// ...
    public QnaBot(QnaBotAccessors accessors, List<QnAMaker> qnaServices, ILoggerFactory loggerFactory)
    {
        // ...
        _qnaServices = qnaServices;
    }
    ```
1. Modifiez le gestionnaire de tour pour interroger les bases de connaissances inscrites par rapport à l’entrée utilisateur. Lorsque votre bot a besoin d’une réponse de QnA Maker, appelez `GetAnswersAsync` à partir du code de votre bot pour obtenir la réponse appropriée en fonction du contexte actuel. Si vous accédez à votre propre base de connaissances, modifiez le message _no answers_ (aucune réponse) ci-dessous pour fournir des instructions initiales utiles à vos utilisateurs.
    ```csharp
    public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
    {
        if (turnContext.Activity.Type == ActivityTypes.Message)
        {
            foreach(var qnaService in _qnaServices)
            {
                var response = await qnaService.GetAnswersAsync(turnContext);
                if (response != null && response.Length > 0)
                {
                    await turnContext.SendActivityAsync(
                        response[0].Answer,
                        cancellationToken: cancellationToken);
                    return;
                }
            }

            var msg = "No QnA Maker answers were found. This example uses a QnA Maker knowledge base that " +
                "focuses on smart light bulbs. Ask the bot questions like 'Why won't it turn on?' or 'I need help'.";

            await turnContext.SendActivityAsync(msg, cancellationToken: cancellationToken);
        }
        else
        {
            await turnContext.SendActivityAsync($"{turnContext.Activity.Type} event detected");
        }
    }
    ```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

1. Ouvrez un terminal ou une invite de commandes dans le répertoire racine de votre projet.
1. Ajoutez le package npm **botbuilder-ai** à votre projet.
    ```shell
    npm i botbuilder-ai
    ```
1. Dans le fichier **index.js**, ajoutez cette instruction require.
    ```javascript
    const { QnAMaker } = require('botbuilder-ai');
    ```
1. Lisez les informations de configuration pour générer les services QnA Maker.
    ```javascript
    // Read bot configuration from .bot file.
    // ...

    // Initialize the QnA knowledge bases for the bot.
    // Assume each QnA entry in the .bot file is well defined.
    const qnaServices = [];
    botConfig.services.forEach(s => {
        if (s.type == 'qna') {
            const endpoint = {
                knowledgeBaseId: s.kbId,
                endpointKey: s.endpointKey,
                host: s.hostname
            };
            const options = {};
            qnaServices.push(new QnAMaker(endpoint, options));
        }
    });

    // Get bot endpoint configuration by service name
    // ...
    ```
1. Mettez à jour la construction de bot à passer dans les services QnA.
    ```javascript
    // Create the bot.
    const myBot = new MyBot(qnaServices);
    ```
1. Dans le fichier **bot.js**, ajoutez un constructeur.
    ```javascript
    constructor(qnaServices) {
        this.qnaServices = qnaServices;
    }
    ```
1. Ensuite, mettez à jour votre gestionnaire de tour pour qu’il recherche une réponse dans vos bases de connaissances.
    ```javascript
    async onTurn(turnContext) {
        if (turnContext.activity.type === ActivityTypes.Message) {
            for (let i = 0; i < this.qnaServices.length; i++) {
                // Perform a call to the QnA Maker service to retrieve matching Question and Answer pairs.
                const qnaResults = await this.qnaServices[i].getAnswers(turnContext);

                // If an answer was received from QnA Maker, send the answer back to the user and exit.
                if (qnaResults[0]) {
                    await turnContext.sendActivity(qnaResults[0].answer);
                    return;
                }
            }
            // If no answers were returned from QnA Maker, reply with help.
            await turnContext.sendActivity('No QnA Maker answers were found. '
                + 'This example uses a QnA Maker Knowledge Base that focuses on smart light bulbs. '
                + `Ask the bot questions like "Why won't it turn on?" or "I need help."`);
        } else {
            await turnContext.sendActivity(`[${ turnContext.activity.type } event detected]`);
        }
    }
    ```

---

### <a name="test-the-bot-locally"></a>Tester le bot localement

À ce stade, votre bot doit pouvoir répondre à certaines questions. Exécutez le bot localement et ouvrez-le dans l’émulateur.

![exemple de qna test](~/media/emulator-v4/qna-test-bot.png)

## <a name="re-publish-your-bot"></a>Republier le bot

Nous pouvons maintenant republier le bot.

[!INCLUDE [publish snippet](~/includes/deploy/snippet-publish.md)]

### <a name="test-the-published-bot"></a>Tester le bot publié

Après avoir publié le bot, patientez une à deux minutes, le temps qu’Azure mette à jour le bot et le démarre.

1. Servez-vous de l’émulateur pour tester le point de terminaison de production pour votre bot ou utilisez le portail Azure pour tester le bot dans Web Chat.

   Dans les deux cas, vous devez observer le même comportement qu’au moment du test local.

## <a name="clean-up-resources"></a>Supprimer des ressources

<!-- In the first tutorial, we should tell them to use a new resource group, so that it is easy to clean up resources. We should also mention in this step in the first tutorial not to clean up resources if they are continuing with the sequence. -->

Si vous ne pensez pas continuer à utiliser cette application, supprimez les ressources associées en effectuant les étapes suivantes :

1. Sur le portail Azure, ouvrez le groupe de ressources de votre bot.
1. Cliquez sur **Supprimer le groupe de ressources** pour supprimer le groupe et toutes les ressources qu’il contient.
1. Dans le volet de confirmation, entrez le nom du groupe de ressources, puis cliquez sur **Supprimer**.

## <a name="next-steps"></a>Étapes suivantes

Pour savoir comment ajouter des fonctionnalités à votre bot, consultez les articles figurant dans la section Développement, sous Procédure.
> [!div class="nextstepaction"]
> [Bouton Étapes suivantes](bot-builder-howto-send-messages.md)
