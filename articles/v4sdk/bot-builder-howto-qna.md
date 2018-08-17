---
title: Utilisation de QnA Maker | Microsoft Docs
description: Découvrez comment utiliser QnA Maker dans votre bot.
keywords: questions et réponses, QnA, FAQ, intergiciel (middleware)
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 03/13/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 78bc2c849a2c1900da33c7419693a7ff84c43cb0
ms.sourcegitcommit: dcbc8ad992a3e242a11ebcdf0ee99714d919a877
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/30/2018
ms.locfileid: "39352948"
---
# <a name="how-to-use-qna-maker"></a>Guide pratique pour utiliser QnA Maker

Pour ajouter une prise en charge basique des questions et des réponses à votre bot, vous pouvez utiliser le service [QnA Maker](https://qnamaker.ai/).

Une des exigences de base de l’écriture de votre propre service QnA Maker est à l’amorçage des questions et des réponses. Très souvent, les questions et les réponses existent déjà dans des contenus comme une FAQ ou d’autres documents. Mais vous voudrez parfois personnaliser vos réponses aux questions afin de leur donner un ton plus naturel, comme dans une véritable conversation. 

## <a name="create-a-qna-maker-service"></a>Créer un service QnA Maker
Créez d’abord un compte et connectez-vous à [QnA Maker](https://qnamaker.ai/). Accédez ensuite à la section **Créer une base de connaissances**. Cliquez sur **Créer un service QnA** et suivez les instructions pour créer un service Azure QnA.

![Image 1 pour qna](media/QnA_1.png)

Vous allez être redirigé vers la section [Créer QnA Maker](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesQnAMaker). Remplissez le formulaire et cliquez sur **Créer**.

![Image 2 pour qna](media/QnA_2.png)
 
Une fois votre service QnA créé dans le portail Azure, des clés apparaissent sous le titre Gestion des ressources : vous pouvez les ignorer. Passez à l’étape 2 pour vous connecter à votre service Azure QnA. Actualisez la page pour sélectionner le service Azure que vous venez de créer, puis entrez le nom de votre base de connaissances.

![Image 3 pour qna](media/QnA_3.png)

Cliquez sur **Create your KB** (Créer votre base de connaissances). Entrez votre propre question et vos réponses, ou copiez les exemples suivants. 

![Image 4 pour qna](media/QnA_4.png)

Vous pouvez également choisir l’option **Populate your KB** (Remplir votre base de connaissances) et fournir un fichier ou une URL. Vous trouverez [ici](https://aka.ms/qna-tsv) un exemple de fichier source pour générer un simple service QnA Maker.

Après avoir ajouté de nouvelles paires QnA ou rempli votre base de connaissances, cliquez sur **Enregistrer et former**. Une fois que vous avez terminé, dans l’onglet **PUBLIER**, cliquez sur **Publier**.

Pour connecter votre service QnA à votre bot, vous aurez besoin de la chaîne de requête HTTP contenant l’ID de la base de connaissances et la clé d’abonnement QnA Maker. Copiez l’exemple de requête HTTP à partir du résultat de publication. 

![Image 5 pour qna](media/QnA_5.png)

## <a name="installing-packages"></a>Installation des packages

Avant de passer au codage, assurez-vous que vous avez installé les packages nécessaires pour QnA Maker.

# <a name="ctabcs"></a>[C#](#tab/cs)

[Ajoutez une référence](https://docs.microsoft.com/en-us/nuget/tools/package-manager-ui) à la préversion 4 des packages NuGet suivants :

* `Microsoft.Bot.Builder.Ai.QnA`

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

Un de ces services peut être ajouté à votre bot à l’aide du package botbuilder-ai. Vous pouvez ajouter ce package à votre projet via npm :

* `npm install --save botbuilder@preview`
* `npm install --save botbuilder-ai@preview`

---


## <a name="using-qna-maker"></a>Utilisation de QnA Maker

QnA Maker est d’abord ajouté comme intergiciel (middleware). Nous pouvons alors utiliser les résultats au sein de notre logique de bot.

# <a name="ctabcs"></a>[C#](#tab/cs)

Mettez à jour la méthode `ConfigureServices` dans votre fichier `Startup.cs` pour ajouter un objet `QnAMakerMiddleware`. Vous pouvez configurer votre bot afin de consulter votre base de connaissances pour chaque message reçu d’un utilisateur, en l’ajoutant simplement à la pile de l’intergiciel (middleware) de votre bot.


```csharp
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Hosting;
using Microsoft.Bot.Builder.Ai.Qna;
using Microsoft.Bot.Builder.BotFramework;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Builder.Integration.AspNet.Core;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Logging;
using System;

public void ConfigureServices(IServiceCollection services)
{
    services.AddSingleton(_ => Configuration);
    services.AddBot<AiBot>(options =>
    {
        options.CredentialProvider = new ConfigurationCredentialProvider(Configuration);

        var endpoint = new QnAMakerEndpoint
        {
           KnowledgeBaseId = "YOUR-KB-ID",
           // Get the Host from the HTTP request example at https://www.qnamaker.ai
           // For GA services: https://<Service-Name>.azurewebsites.net/qnamaker
           // For Preview services: https://westus.api.cognitive.microsoft.com/qnamaker/v2.0           
           Host = "YOUR-HTTP-REQUEST-HOST",
           EndpointKey = "YOUR-QNA-MAKER-SUBSCRIPTION-KEY"
        };
        options.Middleware.Add(new QnAMakerMiddleware(endpoint));
    });
}
```



Modifiez le code dans le fichier EchoBot.cs afin que `OnTurn` envoie un message de secours si l’intergiciel (middleware) QnA Maker n’a pas envoyé une réponse à la question de l’utilisateur :

```csharp
using System.Threading.Tasks;
using Microsoft.Bot;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Schema;

namespace Bot_Builder_Echo_Bot_QnA
{
    public class EchoBot : IBot
    {    
        public async Task OnTurn(ITurnContext context)
        {
            // This bot is only handling Messages
            if (context.Activity.Type == ActivityTypes.Message)
            {             
                if (!context.Responded)
                {
                    // QnA didn't send the user an answer
                    await context.SendActivity("Sorry, I couldn't find a good match in the KB.");

                }
            }
        }
    }
}
```


Pour voir un exemple de bot, reportez-vous à l’[exemple QnA Maker](https://aka.ms/qna-cs-bot-sample).

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

Tout d’abord, demandez/importez la classe [QnAMaker](https://github.com/Microsoft/botbuilder-js/tree/master/doc/botbuilder-ai/classes/botbuilder_ai.qnamaker.md) :

```js
const { QnAMaker } = require('botbuilder-ai');
```

Créez un `QnAMaker` en l’initialisant avec une chaîne basée sur la requête HTTP pour votre service QnA. Vous pouvez copier un exemple de requête pour votre service à l’aide du menu [Portail QnA Maker](https://qnamaker.ai) sous Paramètres > Détails du déploiement.


Le format de la chaîne varie selon que votre service QnA Maker utilise la version de disponibilité générale (GA) ou la version préliminaire de QnA Maker. Copiez l’exemple de requête HTTP et obtenez l’ID de la base de connaissances, la clé d’abonnement et l’hôte à utiliser lorsque vous initialisez le `QnAMaker`.

<!--
**Preview**
```js
const qnaEndpointString = 
    // Replace xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx with your knowledge base ID
    "POST /knowledgebases/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/generateAnswer\r\n" + 
    "Host: https://westus.api.cognitive.microsoft.com/qnamaker/v2.0\r\n" +
    // Replace xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx with your QnAMaker subscription key
    "Ocp-Apim-Subscription-Key: xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx\r\n"
const qna = new QnAMaker(qnaEndpointString);
```

**GA**
```js
const qnaEndpointString = 
    // Replace xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx with your knowledge base ID
    "POST /knowledgebases/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/generateAnswer\r\n" + 
    // Replace <Service-Name> to match the Azure URL where your service is hosted
    "Host: https://<Service-Name>.azurewebsites.net/qnamaker\r\n" +
    // Replace xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx with your QnAMaker subscription key
    "Authorization: EndpointKey xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx\r\n"
const qna = new QnAMaker(qnaEndpointString);
```
-->
**Aperçu**
```js
const qna = new QnAMaker(
    {
        knowledgeBaseId: '<KNOWLEDGE-BASE-ID>',
        endpointKey: '<QNA-SUBSCRIPTION-KEY>',
        host: 'https://westus.api.cognitive.microsoft.com/qnamaker/v2.0'
    },
    {
        // set this to true to send answers from QnA Maker
        answerBeforeNext: true
    }
);
```
**GA**
```js
const qna = new QnAMaker(
    {
        knowledgeBaseId: '<KNOWLEDGE-BASE-ID>',
        endpointKey: '<QNA-SUBSCRIPTION-KEY>',
        host: 'https://<Service-Name>.azurewebsites.net/qnamaker'
    },
    {
        answerBeforeNext: true
    }
);
```
Vous pouvez configurer votre bot pour appeler automatiquement QNA Maker en l’ajoutant à la pile de l’intergiciel (middleware) de votre bot :

```js
// Add QnA Maker as middleware
adapter.use(qna);

// Listen for incoming requests 
server.post('/api/messages', (req, res) => {
    // Route received request to adapter for processing
    adapter.processActivity(req, res, async (context) => {
        // If `!context.responded`, that means an answer wasn't found for the user's utterance.
        // In this case, we send the user a fallback message.
        if (context.activity.type === 'message' && !context.responded) {
            await context.sendActivity('No QnA Maker answers were found.');
        } else if (context.activity.type !== 'message') {
            await context.sendActivity(`[${context.activity.type} event detected]`);
        }
    });
});
```

L’initialisation de `answerBeforeNext` à `true` lors de l’initialisation `QnAMaker` signifie que le l’intergiciel QnA Maker répond automatiquement s’il trouve une réponse, avant d’accéder à la logique principale de votre bot dans `processActivity`. Si, au lieu de cela, vous définissez `answerBeforeNext` sur `false`, le bot appellera uniquement QnA Maker après exécution de toute la logique principale de votre bot dans `processActivity`, et uniquement si ce bot n’a pas répondu à l’utilisateur. 

## <a name="calling-qna-maker-without-using-middleware"></a>Appel de QnA Maker sans utiliser d’intergiciel (middleware)

Dans l’exemple précédent, l’instruction `adapter.use(qna);` signifie que QnA est en cours d’exécution en tant qu’intergiciel (middleware), et par conséquent répond à chaque message reçu par le bot. Pour plus de contrôle sur la façon et le moment où QnA Maker est appelé, vous pouvez appeler `qna.answer()` directement depuis la logique de votre bot au lieu de l’installer comme une partie de l’intergiciel (middleware).

Supprimez l’instruction `adapter.use(qna);` et utilisez le code suivant pour appeler directement QnA Maker.

```js
// Listen for incoming activity 
server.post('/api/messages', (req, res) => {
    // Route received activity to adapter for processing
    adapter.processActivity(req, res, async (context) => {
        if (context.activity.type === 'message') {
            var handled = await qna.answer(context)
                if (!handled) {
                    await context.sendActivity(`I'm sorry. I didn't understand.`);
                }
        }
    });
});
```

Une autre méthode consiste à personnaliser QnA Maker avec `qna.generateAnswer()`. Cette méthode permet d’obtenir plus de détails sur les réponses à partir de QnA Maker.


```js
// Listen for incoming activity 
server.post('/api/messages', (req, res) => {
    // Route received activity to adapter for processing
    adapter.processActivity(req, res, async (context) => {
        if (context.activity.type === 'message') {
            // Get all the answers QnA Maker finds
            var results = await qna.generateAnswer(context.activity.text);
                if (results && results.length > 0) {
                    await context.sendActivity(results[0].answer);
                } else {
                    await context.sendActivity(`I don't know.`);
                }
    
        }
    });
});
```
---

Posez des questions à votre bot pour afficher les réponses de votre service QnA Maker.

![Image 6 pour qna](media/QnA_6.png)



## <a name="next-steps"></a>Étapes suivantes

Vous pouvez combiner QnA Maker à d’autres services Cognitive Services pour rendre votre bot encore plus puissant. L’outil Dispatch offre un moyen de combiner QnA avec Language Understanding (LUIS) dans votre bot.

> [!div class="nextstepaction"]
> [Combiner des applications LUIS et des services QnA à l’aide de l’outil Dispatch](./bot-builder-tutorial-dispatch.md)
