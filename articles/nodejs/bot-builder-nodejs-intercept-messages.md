---
title: Intercepter des messages | Microsoft Docs
description: Découvrez comment créer des journaux ou d’autres enregistrements en interceptant et traitant des échanges d’informations à l’aide du kit SDK Bot Framework pour Node.js.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 11/02/2018
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: b262e936cd48bb73d7b5aa3fa4f7b6318ea7c2a0
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/11/2019
ms.locfileid: "54225624"
---
# <a name="intercept-messages"></a>Intercepter des messages

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-middleware.md)
> - [Node.JS](../nodejs/bot-builder-nodejs-intercept-messages.md)

[!INCLUDE [Introduction to message logging](../includes/snippet-message-logging-intro.md)]

## <a name="example"></a>Exemples

L’exemple de code suivant montre comment intercepter des messages échangés entre un utilisateur et un bot en utilisant le concept d’**intergiciel (middleware)** dans le kit SDK Bot Framework pour Node.js. 

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

Pour obtenir un exemple complet qui montre comment intercepter et journaliser les messages à l’aide du kit SDK Bot Framework pour Node.js, consultez l’<a href="https://aka.ms/v3-js-capability-middlewareLogging" target="_blank">exemple Middleware and Logging</a> dans GitHub.
