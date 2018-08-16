---
title: Concepts clés de Bot Framework Direct Line API 1.1 | Microsoft Docs
description: Découvrez les concepts clés de Bot Framework Direct Line API 1.1.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: f37d8e9215b0a2cd640431f237d1b8c53fad576b
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39298728"
---
# <a name="key-concepts-in-direct-line-api-11"></a>Concepts clés de Direct Line API 1.1

Vous pouvez activer la communication entre votre bot et votre propre application cliente à l’aide de l’API Direct Line. 

> [!IMPORTANT]
> Cet article présente les concepts clés de Direct Line API 1.1 et fournit des informations sur les ressources de développement associées. Si vous créez une connexion entre votre application cliente et votre bot, utilisez plutôt [l’API Direct Line 3.0](bot-framework-rest-direct-line-3-0-concepts.md).

## <a name="authentication"></a>Authentification

Les requêtes Direct Line API 1.1 peuvent être authentifiées soit à l’aide d’un **secret** que vous obtenez dans la page de configuration du canal Direct Line sur le <a href="https://dev.botframework.com/" target="_blank">portail Bot Framework</a>, soit à l’aide d’un **jeton** que vous obtenez au moment de l’exécution.  Pour en savoir plus, consultez [Authentification](bot-framework-rest-direct-line-1-1-authentication.md).

## <a name="starting-a-conversation"></a>Démarrage d’une conversation

Les conversations Direct Line sont explicitement ouvertes par les clients et peuvent durer tant que le bot et le client y participent, et tant que leurs informations d’identification sont valides. Pour plus d’informations, consultez [Démarrer une conversation](bot-framework-rest-direct-line-1-1-start-conversation.md).

## <a name="sending-messages"></a>Envoi de messages

À l’aide de Direct Line API 1.1, un client peut envoyer des messages à votre bot en émettant des requêtes `HTTP POST`. Un client peut envoyer un seul message par requête. Pour plus d’informations, consultez [Envoyer un message au bot](bot-framework-rest-direct-line-1-1-send-message.md).

## <a name="receiving-messages"></a>Réception de messages

À l’aide de Direct Line API 1.1, un client peut recevoir des messages par interrogation à l’aide de requêtes `HTTP GET`. Pour chaque requête, un client peut recevoir de la part du bot plusieurs messages de réponse appartenant à un `MessageSet`. Pour plus d’informations, consultez [Recevoir des messages du bot](bot-framework-rest-direct-line-1-1-receive-messages.md).

## <a name="developer-resources"></a>Ressources pour les développeurs

### <a name="client-library"></a>Bibliothèque cliente

Bot Framework fournit une bibliothèque de client qui facilite l’accès à Direct Line API 1.1 via le langage C#. Pour utiliser la bibliothèque de client au sein d’un projet Visual Studio, installez le <a href="https://www.nuget.org/packages/Microsoft.Bot.Connector.DirectLine/1.1.1" target="_blank">package NuGet v1.x</a> `Microsoft.Bot.Connector.DirectLine`. 

Comme alternative à l’utilisation de la bibliothèque de client C#, vous pouvez générer votre propre bibliothèque de client dans le langage de votre choix à l’aide du <a href="https://docs.botframework.com/en-us/restapi/directline/swagger.json" target="_blank">fichier Swagger Direct Line API 1.1</a>.

### <a name="web-chat-control"></a>Contrôle de webchat 

Bot Framework fournit un contrôle qui vous permet d’incorporer un bot Direct-Line dans votre application cliente. Pour plus d’informations, consultez <a href="https://github.com/Microsoft/BotFramework-WebChat" target="_blank">Contrôle WebChat Microsoft Bot Framework</a>.
