---
title: Démarrer une conversation | Microsoft Docs
description: Découvrez comment démarrer une conversation à l’aide de l’API Direct Line v3.0.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: ead16c0ff41ae93daff8952ca135fa0771bbbe78
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39299360"
---
# <a name="start-a-conversation"></a>Démarrer une conversation

Les conversations Direct Line sont explicitement ouvertes par les clients et peuvent durer tant que le bot et le client y participent et tant que leurs informations d’identification sont valides. La conversation étant ouverte, le bot et le client peuvent envoyer des messages. Plusieurs clients peuvent se connecter à une conversation donnée, et chaque client peut participer pour le compte de plusieurs utilisateurs.

## <a name="open-a-new-conversation"></a>Ouvrir une nouvelle conversation

Pour ouvrir une nouvelle conversation avec un bot, émettez cette requête :

```http
POST https://directline.botframework.com/v3/directline/conversations
Authorization: Bearer SECRET_OR_TOKEN
```

Les extraits de code suivants illustrent la requête et la réponse pour le démarrage d’une conversation.

### <a name="request"></a>Requête

```http
POST https://directline.botframework.com/v3/directline/conversations
Authorization: Bearer RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0y8qbOF5xPGfiCpg4Fv0y8qqbOF5x8qbOF5xn
```

### <a name="response"></a>response

Si la requête aboutit, la réponse contient un ID pour la conversation, un jeton, une valeur indiquant le nombre de secondes avant l’expiration du jeton et une URL de flux que le client peut utiliser pour [recevoir des activités par le biais du flux WebSocket](bot-framework-rest-direct-line-3-0-receive-activities.md#connect-via-websocket).

```http
HTTP/1.1 201 Created
[other headers]
```

```json
{
  "conversationId": "abc123",
  "token": "RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0y8qbOF5xPGfiCpg4Fv0y8qqbOF5x8qbOF5xn",
  "expires_in": 1800,
  "streamUrl": "https://directline.botframework.com/v3/directline/conversations/abc123/stream?t=RCurR_XV9ZA.cwA..."
}
```

En règle générale, une requête de démarrage de conversation est utilisée pour ouvrir une nouvelle conversation, et un code d’état **HTTP 201** est retourné si la nouvelle conversation démarre correctement. Toutefois, si un client soumet une requête de démarrage de conversation avec un jeton Direct Line dans l’en-tête `Authorization` qui a déjà été utilisé pour démarrer une conversation avec l’opération de démarrage de conversation, un code d’état **HTTP 200** est retourné pour indiquer que la requête était acceptable, mais qu’aucune conversation n’a été créée (dans la mesure où elle existait déjà).

> [!TIP]
> Vous disposez de 60 secondes pour vous connecter à l’URL de flux WebSocket. Si la connexion ne peut pas être établie avant la fin de ce délai, vous pouvez [vous reconnecter à la conversation](bot-framework-rest-direct-line-3-0-reconnect-to-conversation.md) pour générer une nouvelle URL de flux.

## <a name="start-conversation-versus-generate-token"></a>Démarrer une conversation ou générer un jeton

L’opération de démarrage de conversation (`POST /v3/directline/conversations`) est similaire à l’opération de [génération de jeton ](bot-framework-rest-direct-line-3-0-authentication.md#generate-token) (`POST /v3/directline/tokens/generate`) dans la mesure où les deux opérations retournent un `token` qui peut être utilisé pour accéder à une seule conversation. Toutefois, contrairement à l’opération de génération de jeton, l’opération de démarrage de conversation démarre également la conversation, contacte le bot et crée une URL de flux WebSocket. 

Si vous envisagez de démarrer la conversation immédiatement, utilisez l’opération de démarrage de conversation. Si vous envisagez de distribuer le jeton aux clients et souhaitez qu’ils initialisent la conversation, utilisez plutôt l’opération de [génération de jeton](bot-framework-rest-direct-line-3-0-authentication.md#generate-token). 

## <a name="additional-resources"></a>Ressources supplémentaires

- [Concepts clés](bot-framework-rest-direct-line-3-0-concepts.md)
- [Authentification](bot-framework-rest-direct-line-3-0-authentication.md)
- [Receive activities via WebSocket stream](bot-framework-rest-direct-line-3-0-receive-activities.md#connect-via-websocket) (Recevoir des activités par le biais d’un flux WebSocket)
- [Se reconnecter à une conversation](bot-framework-rest-direct-line-3-0-reconnect-to-conversation.md)