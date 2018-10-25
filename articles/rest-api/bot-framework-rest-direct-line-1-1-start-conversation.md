---
title: Démarrer une conversation | Microsoft Docs
description: Découvrez comment démarrer une conversation à l’aide de l’API Direct Line v1.1.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
ms.openlocfilehash: da81182d80ebac0d5aaba5a2660899d87c7e2b40
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2018
ms.locfileid: "50000216"
---
# <a name="start-a-conversation"></a>Démarrer une conversation

> [!IMPORTANT]
> Cet article décrit comment démarrer une conversation à l’aide de l’API Direct Line v1.1. Si vous créez une connexion entre votre application cliente et votre robot, utilisez plutôt [l’API Direct Line 3.0](bot-framework-rest-direct-line-3-0-start-conversation.md).

Les conversations Direct Line sont explicitement ouvertes par les clients et peuvent durer tant que le robot et le client y participent et tant que leurs informations d’identification sont valides. La conversation étant ouverte, le robot et le client peuvent envoyer des messages. Plusieurs clients peuvent se connecter à une conversation donnée, et chaque client peut participer pour le compte de plusieurs utilisateurs.

## <a name="open-a-new-conversation"></a>Ouvrir une nouvelle conversation

Pour ouvrir une nouvelle conversation avec un robot, émettez la demande suivante :

```http
POST https://directline.botframework.com/api/conversations
Authorization: Bearer SECRET_OR_TOKEN
```

Les extraits de code suivants illustrent la demande et la réponse pour le démarrage d’une conversation.

### <a name="request"></a>Requête

```http
POST https://directline.botframework.com/api/conversations
Authorization: Bearer RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0y8qbOF5xPGfiCpg4Fv0y8qqbOF5x8qbOF5xn
```

### <a name="response"></a>response

Si la demande réussit, la réponse contient un ID pour la conversation, un jeton et une valeur indiquant le nombre de secondes avant l’expiration du jeton.

```http
HTTP/1.1 200 OK
[other headers]
```

```json
{
  "conversationId": "abc123",
  "token": "RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0y8qbOF5xPGfiCpg4Fv0y8qqbOF5x8qbOF5xn",
  "expires_in": 1800
}
```

## <a name="start-conversation-versus-generate-token"></a>Démarrer une conversation ou générer un jeton

L’opération de démarrage de conversation (`POST /api/conversations`) est similaire à l’opération de [génération de jeton ](bot-framework-rest-direct-line-1-1-authentication.md#generate-token) (`POST /api/tokens/conversation`) dans la mesure où les deux opérations retournent un `token` qui peut être utilisé pour accéder à une seule conversation. Toutefois, contrairement à l’opération de génération de jeton, l’opération de démarrage de conversation entame la conversation et contacte le robot. 

Si vous envisagez de démarrer la conversation immédiatement, utilisez l’opération de démarrage de conversation. Si vous envisagez de distribuer le jeton aux clients et souhaitez qu’ils initialisent la conversation, utilisez plutôt l’opération de [génération de jeton](bot-framework-rest-direct-line-1-1-authentication.md#generate-token). 

## <a name="additional-resources"></a>Ressources supplémentaires

- [Concepts clés](bot-framework-rest-direct-line-1-1-concepts.md)
- [Authentification](bot-framework-rest-direct-line-1-1-authentication.md)