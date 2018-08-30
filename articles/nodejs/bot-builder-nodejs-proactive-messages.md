---
title: Envoyer des messages proactifs | Microsoft Docs
description: Découvrez comment interrompre le flux de conversation actuel avec un message proactif en utilisant le Kit de développement logiciel (SDK) Bot Builder pour Node.js
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 509cba25eae229d1e454cd1846a97474f0af957b
ms.sourcegitcommit: 2dc75701b169d822c9499e393439161bc87639d2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/24/2018
ms.locfileid: "42904263"
---
# <a name="send-proactive-messages"></a>Envoyer des messages proactifs
[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-proactive-messages.md)
> - [Node.JS](../nodejs/bot-builder-nodejs-proactive-messages.md)

[!INCLUDE [Introduction to proactive messages - part 1](../includes/snippet-proactive-messages-intro-1.md)]

## <a name="types-of-proactive-messages"></a>Types de messages proactifs

[!INCLUDE [Introduction to proactive messages - part 2](../includes/snippet-proactive-messages-intro-2.md)]

## <a name="send-an-ad-hoc-proactive-message"></a>Envoyer un message proactif ad hoc

Les exemples de code suivants montrent comment envoyer un message proactif ad hoc à l’aide du Kit de développement logiciel (SDK) Bot Builder pour Node.js.

Pour pouvoir envoyer un message ad hoc à un utilisateur, le robot doit d’abord collecter et enregistrer des informations sur l’utilisateur à partir de la conversation en cours. La propriété **adresse** du message inclut toutes les informations dont le robot aura besoin pour envoyer un message ad hoc à l’utilisateur ultérieurement. 

```javascript
bot.dialog('adhocDialog', function(session, args) {
    var savedAddress = session.message.address;

    // (Save this information somewhere that it can be accessed later, such as in a database, or session.userData)
    session.userData.savedAddress = savedAddress;

    var message = 'Hello user, good to meet you! I now know your address and can send you notifications in the future.';
    session.send(message);
})
```

> [!NOTE]
> Le robot peut stocker les données utilisateur de n’importe quelle manière pour autant qu’il puisse y accéder ultérieurement.

Après avoir collecté des informations sur l’utilisateur, le robot peut envoyer un message proactif ad hoc à l’utilisateur à tout moment. Pour ce faire, il récupère simplement les données utilisateur stockées précédemment, construit le message, puis l’envoie.

```javascript
var inMemoryStorage = new builder.MemoryBotStorage();

var bot = new builder.UniversalBot(connector)
                .set('storage', inMemoryStorage); // Register in-memory storage 

function sendProactiveMessage(address) {
    var msg = new builder.Message().address(address);
    msg.text('Hello, this is a notification');
    msg.textLocale('en-US');
    bot.send(msg);
}
```

> [!TIP]
> Un message proactif ad hoc peut être initié, par exemple, à partir de déclencheurs asynchrones tels que des requêtes http, des minuteurs, des files d’attente ou de tout autre emplacement choisi par le développeur.

## <a name="send-a-dialog-based-proactive-message"></a>Envoyer un message proactif basé sur un dialogue

Les exemples de code suivants montrent comment envoyer un message proactif basé sur un dialogue à l’aide du Kit de développement logiciel (SDK) Bot Builder pour Node.js. L’exemple fonctionnel complet se trouve dans le dossier [Microsoft/BotBuilder-Samples/Node/core-proactiveMessages/startNewDialog](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/core-proactiveMessages/startNewDialog).

Pour pouvoir envoyer un message basé sur un dialogue à un utilisateur, le robot doit d’abord collecter et enregistrer les informations de la conversation en cours. L’objet `session.message.address` inclut toutes les informations dont le robot aura besoin pour envoyer à l’utilisateur un message proactif basé sur un dialogue. 

```javascript
// proactiveDialog dialog
bot.dialog('proactiveDialog', function (session, args) {

    savedAddress = session.message.address;

    var message = 'Hey there, I\'m going to interrupt our conversation and start a survey in five seconds...';
    session.send(message);

    message = `You can also make me send a message by accessing: http://localhost:${server.address().port}/api/CustomWebApi`;
    session.send(message);

    setTimeout(() => {
        startProactiveDialog(savedAddress);
    }, 5000);
});
```

Quand il est temps d’envoyer le message, le robot crée un dialogue et l’ajoute en haut de la pile de dialogues. Le nouveau dialogue prend le contrôle de la conversation, délivre le message proactif, se ferme, puis rend le contrôle au dialogue précédent dans la pile. 

```javascript
// initiate a dialog proactively 
function startProactiveDialog(address) {
    bot.beginDialog(address, "*:survey");
}
```

> [!NOTE]
> L’exemple de code ci-dessus nécessite un fichier personnalisé, **botadapter.js**, que vous pouvez [télécharger à partir de GitHub](https://github.com/Microsoft/BotBuilder-Samples/blob/master/Node/core-proactiveMessages/startNewDialog/botadapter.js).

La boîte d’enquête contrôle la conversation jusqu’à la fin. Ensuite, il se ferme (en appelant `session.endDialog()`), en rendant le contrôle au dialogue précédent. 


```javascript
// handle the proactive initiated dialog
bot.dialog('survey', function (session, args, next) {
  if (session.message.text === "done") {
    session.send("Great, back to the original conversation");
    session.endDialog();
  } else {
    session.send('Hello, I\'m the survey dialog. I\'m interrupting your conversation to ask you a question. Type "done" to resume');
  }
});
```

## <a name="sample-code"></a>Exemple de code

Pour obtenir un exemple complet montrant comment envoyer des messages proactifs à l’aide du Kit de développement logiciel (SDK) Bot Builder pour Node.js, voir les <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/core-proactiveMessages" target="_blank">exemples de messages proactifs</a> dans GitHub. Dans les exemples de messages proactifs, <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/core-proactiveMessages/simpleSendMessage" target="_blank">simpleSendMessage</a> montre comment envoyer un message proactif ad hoc, et <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/core-proactiveMessages/startNewDialog" target="_blank"> startNewDialog </a> comment envoyer un message proactif basé sur un dialogue.

## <a name="additional-resources"></a>Ressources supplémentaires

- [Conception du flux de conversation](../bot-service-design-conversation-flow.md)
