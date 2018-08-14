---
title: Authentification | Microsoft Docs
description: Découvrez comment authentifier des demandes d’API dans l’API Direct Line v3.0.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: 18a5815a96e2052a54c48f6af211d8b28e20d983
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39299365"
---
# <a name="authentication"></a>Authentification

Un client peut authentifier les demandes adressées à l’API Direct Line 3.0 soit à l’aide d’un **secret** que vous [obtenez dans la page de configuration du canal Direct Line](../bot-service-channel-connect-directline.md) sur le portail Bot Framework, ou à l’aide d’un **jeton** que vous obtenez au moment de l’exécution. Le secret ou le jeton doivent être spécifiés dans l’en-tête `Authorization` de chaque demande, en utilisant le format suivant : 

```http
Authorization: Bearer SECRET_OR_TOKEN
```

## <a name="secrets-and-tokens"></a>Secrets et jetons

Un **secret** Direct Line est une clé principale qui peut être utilisée pour accéder à toute conversation appartenant au robot associé. Un **secret** peut également être utilisé pour obtenir un **jeton**. Les secrets n’expirent pas. 

Un **jeton** Direct Line est une clé qui peut être utilisée pour accéder à une seule conversation. Un jeton expire, mais peut être actualisé. 

Si vous créez une application de service à service, l’approche la plus simple peut consister à spécifier le **secret** dans l’en-tête `Authorization` des demandes d’API Direct Line. Si vous écrivez une application où le client s’exécute dans un navigateur web ou une application mobile, vous souhaiterez peut-être échanger votre secret contre un jeton (qui fonctionne uniquement pour une conversation et expire sauf s’il est actualisé) et spécifier le **jeton** dans l’en-tête `Authorization` des demandes d’API Direct Line. Choisissez le modèle de sécurité qui vous convient le mieux.

> [!NOTE]
> Les informations d’identification de votre client Direct Line sont différentes de celles de votre robot. Cela vous permet de réviser vos clés de manière indépendante et de partager des jetons de client sans révéler le mot de passe de votre robot. 

## <a name="get-a-direct-line-secret"></a>Obtenir un secret Direct Line

Vous pouvez [obtenir un secret Direct Line](../bot-service-channel-connect-directline.md) à partir de la page de configuration de canal Direct Line pour votre robot dans le <a href="https://dev.botframework.com/" target="_blank">portail Bot Framework</a> :

![Configuration de Direct Line](../media/direct-line-configure.png)

## <a id="generate-token"></a> Générer un jeton Direct Line

Pour générer un jeton Direct Line qui peut être utilisé pour accéder à une conversation unique, commencez par obtenir le secret Direct Line à partir de la page de configuration de canal Direct Line dans le <a href="https://dev.botframework.com/" target="_blank">portail Bot Framework</a>. Ensuite, émettez la demande suivante pour échanger votre secret Direct Line contre un jeton Direct Line :

```http
POST https://directline.botframework.com/v3/directline/tokens/generate
Authorization: Bearer SECRET
```

Dans l’en-tête `Authorization` de cette demande, remplacez **SECRET** par la valeur de votre secret Direct Line.

Les extraits de code suivants illustrent la demande et la réponse dans le cadre de la génération de jeton.

### <a name="request"></a>Requête

```http
POST https://directline.botframework.com/v3/directline/tokens/generate
Authorization: Bearer RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0
```

### <a name="response"></a>response

Si la demande réussit, la réponse contient un `token` valide pour une conversation, et une valeur `expires_in` qui indique le nombre de secondes avant l’expiration du jeton. Pour que le jeton reste valide, vous devez [l’actualiser](#refresh-token) avant son expiration.

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

### <a name="generate-token-versus-start-conversation"></a>Générer un jeton ou démarrer une conversation

L’opération de génération de jeton (`POST /v3/directline/tokens/generate`) est similaire à l’opération de [démarrage de conversation](bot-framework-rest-direct-line-3-0-start-conversation.md) (`POST /v3/directline/conversations`) dans la mesure où les deux opérations retournent un `token` qui peut être utilisé pour accéder à une seule conversation. Toutefois, contrairement à l’opération de démarrage de conversation, l’opération de génération de jeton ne démarre pas la conversation, ne contacte pas le robot, et ne crée pas d’URL WebSocket de diffusion en continu. 

Si vous envisagez de distribuer le jeton aux clients et souhaitez qu’ils initialisent la conversation, utilisez l’opération de génération de jeton. Si vous envisagez de démarrer la conversation immédiatement, utilisez plutôt l’opération de [démarrage de conversation](bot-framework-rest-direct-line-3-0-start-conversation.md).

## <a id="refresh-token"></a> Actualiser un jeton Direct Line

Un jeton Direct Line peut être actualisé un nombre illimité de fois, tant qu’il n’a pas expiré. Il n’est pas possible d’actualiser un jeton qui a expiré. Pour actualiser un jeton Direct Line, émettez la demande suivante : 

```http
POST https://directline.botframework.com/v3/directline/tokens/refresh
Authorization: Bearer TOKEN_TO_BE_REFRESHED
```

Dans l’en-tête `Authorization` de cette demande, remplacez **TOKEN_TO_BE_REFRESHED** par le jeton Direct Line à actualiser.

Les extraits de code suivants illustrent la demande et la réponse dans le cadre de l’actualisation du jeton.

### <a name="request"></a>Requête

```http
POST https://directline.botframework.com/v3/directline/tokens/refresh
Authorization: Bearer CurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0y8qbOF5xPGfiCpg4Fv0y8qqbOF5x8qbOF5xn
```

### <a name="response"></a>response

Si la demande réussit, la réponse contient un nouveau `token` valide pour la même conversation que le jeton précédent, et une valeur `expires_in` qui indique le nombre de secondes avant l’expiration du jeton. Pour que le nouveau jeton reste valide, vous devez [l’actualiser](#refresh-token) avant son expiration.

```http
HTTP/1.1 200 OK
[other headers]
```

```json
{
  "conversationId": "abc123",
  "token": "RCurR_XV9ZA.cwA.BKA.y8qbOF5xPGfiCpg4Fv0y8qqbOF5x8qbOF5xniaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0",
  "expires_in": 1800
}
```

## <a name="additional-resources"></a>Ressources supplémentaires

- [Concepts clés](bot-framework-rest-direct-line-3-0-concepts.md)
- [Connecter un robot à Direct Line](../bot-service-channel-connect-directline.md)