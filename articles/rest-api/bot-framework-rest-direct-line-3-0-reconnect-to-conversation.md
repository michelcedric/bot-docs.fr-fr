---
title: Se reconnecter à une conversation | Microsoft Docs
description: Découvrez comment vous reconnecter à une conversation à l’aide de l’API Direct Line v3.0.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: 2c6b3a7e9f0fdc7d5227fc8112cb6f3e330a2bcc
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39299129"
---
# <a name="reconnect-to-a-conversation"></a>Se reconnecter à une conversation

Si un client utilise [l’interface WebSocket](bot-framework-rest-direct-line-3-0-receive-activities.md#connect-via-websocket) pour recevoir des messages mais perd sa connexion, il peut avoir besoin de se reconnecter. Dans ce cas, le client doit générer une nouvelle URL de flux WebSocket, qu’il peut utiliser pour se reconnecter à la conversation.

## <a name="generate-a-new-websocket-stream-url"></a>Générer une nouvelle URL de flux WebSocket

Pour générer une nouvelle URL de flux WebSocket qui peut être utilisée pour se reconnecter à une conversation existante, exécutez cette requête : 

```http
GET https://directline.botframework.com/v3/directline/conversations/{conversationId}?watermark={watermark_value}
Authorization: Bearer SECRET_OR_TOKEN
```

Dans cet URI de requête, remplacez **{conversationId}** par l’ID de conversation et remplacez **{watermark_value}** par la valeur de limite (si le paramètre `watermark` est fourni). Le paramètre `watermark` est facultatif. Si le paramètre `watermark` est spécifié dans l’URI de requête, la conversation reprend à partir de la limite, ce qui garantit qu’aucun message n’est perdu. Si le paramètre `watermark` est omis de l’URI de requête, seuls les messages reçus après la requête de reconnexion sont repris.

Les extraits de code suivants illustrent la requête de reconnexion et la réponse.

### <a name="request"></a>Requête

```http
GET https://directline.botframework.com/v3/directline/conversations/abc123?watermark=0000a-42
Authorization: Bearer RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0y8qbOF5xPGfiCpg4Fv0y8qqbOF5x8qbOF5xn
```

### <a name="response"></a>response

Si la requête aboutit, la réponse contient un ID pour la conversation, un jeton et une nouvelle URL de flux WebSocket.

```http
HTTP/1.1 200 OK
[other headers]
```

```json
{
  "conversationId": "abc123",
  "token": "RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0y8qbOF5xPGfiCpg4Fv0y8qqbOF5x8qbOF5xn",
  "streamUrl": "https://directline.botframework.com/v3/directline/conversations/abc123/stream?watermark=000a-4&amp;t=RCurR_XV9ZA.cwA..."
}
```

## <a name="reconnect-to-the-conversation"></a>Se reconnecter à la conversation

Le client doit utiliser la nouvelle URL de flux WebSocket pour [se reconnecter à la conversation](bot-framework-rest-direct-line-3-0-receive-activities.md#connect-via-websocket) sous 60 secondes. Si la connexion ne peut pas être établie avant la fin de ce délai, le client doit émettre une autre requête de reconnexion pour générer une nouvelle URL de flux.

## <a name="additional-resources"></a>Ressources supplémentaires

- [Concepts clés](bot-framework-rest-direct-line-3-0-concepts.md)
- [Authentification](bot-framework-rest-direct-line-3-0-authentication.md)
- [Receive activities via WebSocket stream](bot-framework-rest-direct-line-3-0-receive-activities.md#connect-via-websocket) (Recevoir des activités par le biais d’un flux WebSocket)