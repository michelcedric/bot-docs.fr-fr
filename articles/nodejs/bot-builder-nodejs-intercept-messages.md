---
title: Intercepter des messages | Microsoft Docs
description: Apprenez à créer des journaux ou d’autres enregistrements en interceptant et traitant des échanges d’informations à l’aide du Kit de développement logiciel (SDK) Bot Builder pour Node.js.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 11/02/2018
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 31380961f117a2b4a3ffaae3c82d682a63001c0c
ms.sourcegitcommit: 984705927561cc8d6a84f811ff24c8c71b71c76b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/02/2018
ms.locfileid: "50965677"
---
# <a name="intercept-messages"></a>Intercepter des messages

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-middleware.md)
> - [Node.JS](../nodejs/bot-builder-nodejs-intercept-messages.md)

[!INCLUDE [Introduction to message logging](../includes/snippet-message-logging-intro.md)]

## <a name="example"></a>Exemples

L’exemple de code suivant montre comment intercepter des messages échangés entre un utilisateur et robot en utilisant le concept d’**intergiciel (middleware)** dans le Kit de développement logiciel (SDK) Bot Builder pour Node.js. 

Tout d’abord, configurez le gestionnaire pour les messages entrants (`botbuilder`) et le gestionnaire pour les messages sortants (`send`).

```javascript
server.post('/api/messages', connector.listen());
var bot = new builder.UniversalBot(connector);
bot.use({
    botbuilder: function (session, next) {
        myMiddleware.logIncomingMessage(session, next);
    },
    send: function (event, next) {
        myMiddleware.logOutgoingMessage(event, next);
    }
})
```

Ensuite, implémentez chacun des gestionnaires pour définir l’action à effectuer pour chaque message intercepté.

```javascript
module.exports = {
    logIncomingMessage: function (session, next) {
        console.log(session.message.text);
        next();
    },
    logOutgoingMessage: function (event, next) {
        console.log(event.text);
        next();
    }
}
```

Désormais, chaque message entrant (de l’utilisateur ou du robot) déclenche `logIncomingMessage`, et chaque message sortant (du robot ou de l’utilisateur) déclenche `logOutgoingMessage`.
Dans cet exemple, le robot imprime simplement des informations sur chaque message, mais vous pouvez mettre à jour `logIncomingMessage` et `logOutgoingMessage` si nécessaire pour définir les actions à effectuer pour chaque message. 

## <a name="sample-code"></a>Exemple de code

Pour obtenir un exemple complet montrant comment intercepter et journaliser les messages à l’aide du Kit de développement logiciel (SDK) Bot Builder pour Node.js, voir l’<a href="https://aka.ms/v3-js-capability-middlewareLogging" target="_blank">exemple d’intergiciel (middleware) et de journalisation</a> dans GitHub.
