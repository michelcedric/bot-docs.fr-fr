---
title: Concepts clés de Bot Framework Direct Line API 3.0 | Microsoft Docs
description: Découvrez les concepts clés de Bot Framework Direct Line API 3.0.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 01/06/2019
ms.openlocfilehash: 94ef9cc221e67f4f3762eb7a1a006a915e3c5307
ms.sourcegitcommit: fd60ad0ff51b92fa6495b016e136eaf333413512
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 02/06/2019
ms.locfileid: "55764097"
---
# <a name="key-concepts-in-direct-line-api-30"></a>Concepts clés de Direct Line API 3.0

Vous pouvez activer la communication entre votre bot et votre propre application cliente à l’aide de l’API Direct Line. Cet article présente les concepts clés de Direct Line API 3.0 et fournit des informations sur les ressources de développement associées.

## <a name="authentication"></a>Authentication

Les requêtes Direct Line API 3.0 peuvent être authentifiées soit à l’aide d’un **secret** que vous obtenez dans la page de configuration du canal Direct Line sur le <a href="https://dev.botframework.com/" target="_blank">portail Bot Framework</a>, soit à l’aide d’un **jeton** que vous obtenez au moment de l’exécution. Pour en savoir plus, consultez [Authentification](bot-framework-rest-direct-line-3-0-authentication.md).

## <a name="starting-a-conversation"></a>Démarrage d’une conversation

Les conversations Direct Line sont explicitement ouvertes par les clients et peuvent durer tant que le bot et le client y participent, et tant que leurs informations d’identification sont valides. Pour plus d’informations, consultez [Démarrer une conversation](bot-framework-rest-direct-line-3-0-start-conversation.md).

## <a name="sending-messages"></a>Envoi de messages

À l’aide de Direct Line API 3.0, un client peut envoyer des messages à votre bot en émettant des requêtes `HTTP POST`. Un client peut envoyer un seul message par requête. Pour plus d’informations, consultez [Envoyer une activité au bot](bot-framework-rest-direct-line-3-0-send-activity.md).

## <a name="receiving-messages"></a>Réception de messages

Avec Direct Line API 3.0, un client peut recevoir des messages de votre bot via un flux `WebSocket` ou en envoyant des requêtes `HTTP GET`. Avec ces techniques, un client peut recevoir plusieurs messages à la fois de la part d’un bot, dans le cadre d’un `ActivitySet`. Pour plus d’informations, consultez [Recevoir des activités du bot](bot-framework-rest-direct-line-3-0-receive-activities.md).

## <a name="developer-resources"></a>Ressources pour les développeurs

### <a name="client-libraries"></a>Bibliothèques clientes

Bot Framework fournit des bibliothèques de client qui facilitent l’accès à Direct Line API 3.0 via le langage C# et Node.js. 

- Pour utiliser la bibliothèque de client .NET dans un projet Visual Studio, installez le <a href="https://www.nuget.org/packages/Microsoft.Bot.Connector.DirectLine" target="_blank">package NuGet</a> `Microsoft.Bot.Connector.DirectLine`. 

- Pour utiliser la bibliothèque de client Node.js, installez la bibliothèque `botframework-directlinejs` à l’aide de <a href="https://www.npmjs.com/package/botframework-directlinejs" target="_blank">NPM</a> (ou <a href="https://github.com/Microsoft/BotFramework-DirectLineJS" target="_blank">téléchargez</a> le code source).

::: moniker range="azure-bot-service-3.0"

### <a name="sample-code"></a>Exemple de code

Le dépôt GitHub <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/v3-sdk-samples" target="_blank">BotBuilder-Samples</a> contient plusieurs exemples qui montrent comment utiliser Direct Line API 3.0 avec C# et Node.js.

| Exemple | Langage | Description |
|----|----|----|
| <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/v3-sdk-samples/CSharp/core-DirectLine" target="_blank">Exemple de bot Direct Line</a> | C# | Exemple de bot et client personnalisé qui communiquent entre eux à l’aide de l’API Direct Line. |
| <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/v3-sdk-samples/CSharp/core-DirectLineWebSockets" target="_blank">Exemple de bot Direct Line (à l’aide de WebSockets clients)</a> | C# | Exemple de bot et client personnalisé qui communiquent entre eux à l’aide de l’API Direct Line et WebSocket. |
| <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/v3-sdk-samples/Node/core-DirectLine" target="_blank">Exemple de bot Direct Line</a> | JavaScript | Exemple de bot et client personnalisé qui communiquent entre eux à l’aide de l’API Direct Line. |
| <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/v3-sdk-samples/Node/core-DirectLineWebSockets" target="_blank">Exemple de bot Direct Line (à l’aide de WebSockets clients)</a> | JavaScript | Exemple de bot et client personnalisé qui communiquent entre eux à l’aide de l’API Direct Line et WebSocket. |

::: moniker-end

### <a name="web-chat-control"></a>Contrôle de webchat 

Bot Framework fournit un contrôle qui vous permet d’incorporer un bot Direct-Line dans votre application cliente. Pour plus d’informations, consultez <a href="https://github.com/Microsoft/BotFramework-WebChat" target="_blank">Contrôle WebChat Microsoft Bot Framework</a>.
