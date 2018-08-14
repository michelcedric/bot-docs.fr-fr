---
title: Créer un bot avec le service Bot Connector | Microsoft Docs
description: Créez un bot avec le service Bot Connector.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: 3baf5bde772e67084a6046a8d2a8e7d631b245f6
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39299136"
---
# <a name="create-a-bot-with-the-bot-connector-service"></a>Créer un bot avec le service Bot Connector
> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-quickstart.md)
> - [Node.JS](../nodejs/bot-builder-nodejs-quickstart.md)
> - [Service de robot](../bot-service-quickstart.md)
> - [REST](../rest-api/bot-framework-rest-connector-quickstart.md)

Le service Bot Connector permet à votre bot d’échanger des messages avec des canaux configurés dans le <a href="https://dev.botframework.com/" target="_blank">portail Bot Framework</a> en utilisant les standards REST et JSON sur HTTPS. Ce didacticiel vous guide tout au long des processus d’obtention d’un jeton d’accès à partir de Bot Framework et d’utilisation du service Bot Connecteur pour échanger des messages avec l’utilisateur.

## <a id="get-token"></a> Obtenir un jeton d’accès

> [!IMPORTANT]
> Si vous ne l’avez pas encore fait, vous devez [inscrire votre bot](../bot-service-quickstart-registration.md) auprès de Bot Framework pour obtenir son ID d’application et son mot de passe. Vous devrez disposer de l’ID d’application et du mot de passe du bot pour obtenir un jeton d’accès.

Pour communiquer avec le service Bot Connector, vous devez indiquer un jeton d’accès dans l’en-tête `Authorization` de chaque requête d’API, au format suivant : 

```http
Authorization: Bearer ACCESS_TOKEN
```

Vous pouvez obtenir le jeton d’accès de votre bot en émettant une requête d’API.

### <a name="request"></a>Requête

Pour demander un jeton d’accès permettant d’authentifier les requêtes adressées au service Bot Connector, émettez la requête ci-après en remplaçant **MICROSOFT-APP-ID** et **MICROSOFT-APP-PASSWORD** par l’ID d’application et le mot de passe que vous avez obtenus lorsque vous avez [inscrit](../bot-service-quickstart-registration.md) votre bot auprès de Bot Framework.

```http
POST https://login.microsoftonline.com/botframework.com/oauth2/v2.0/token
Host: login.microsoftonline.com
Content-Type: application/x-www-form-urlencoded

grant_type=client_credentials&client_id=MICROSOFT-APP-ID&client_secret=MICROSOFT-APP-PASSWORD&scope=https%3A%2F%2Fapi.botframework.com%2F.default
```

### <a name="response"></a>response

Si la requête aboutit, vous recevrez une réponse HTTP 200 spécifiant le jeton d’accès et les informations d’expiration de ce dernier. 

```json
{
    "token_type":"Bearer",
    "expires_in":3600,
    "ext_expires_in":3600,
    "access_token":"eyJhbGciOiJIUzI1Ni..."
}
```

> [!TIP]
> Pour plus d’informations sur l’authentification dans le service Bot Connector, consultez l’article [Authentification](bot-framework-rest-connector-authentication.md).

## <a name="exchange-messages-with-the-user"></a>Échanger des messages avec l’utilisateur

Une conversation est une série de messages échangés entre un utilisateur et votre bot. 

### <a name="receive-a-message-from-the-user"></a>Recevoir un message de l’utilisateur

Lorsque l’utilisateur envoie un message, le service Bot Framework Connector publie une requête destinée au point de terminaison que vous avez spécifié lorsque vous avez [inscrit](../bot-service-quickstart-registration.md) votre bot. Le corps de la requête est un objet [Activity][Activity]. L’exemple ci-après présente le corps de la requête qu’un bot reçoit lorsque l’utilisateur lui envoie un message simple. 

```json
{
    "type": "message",
    "id": "bf3cc9a2f5de...",
    "timestamp": "2016-10-19T20:17:52.2891902Z",
    "serviceUrl": "https://smba.trafficmanager.net/apis",
    "channelId": "channel's name/id",
    "from": {
        "id": "1234abcd",
        "name": "user's name"
    },
    "conversation": {
        "id": "abcd1234",
        "name": "conversation's name"
    },
    "recipient": {
        "id": "12345678",
        "name": "bot's name"
    },
    "text": "Haircut on Saturday"
}
```

### <a name="reply-to-the-users-message"></a>Répondre au message de l’utilisateur

Lorsque le point de terminaison de votre bot reçoit une requête `POST` qui représente un message émanant de l’utilisateur (par exemple, `type` = **message**), utilisez les informations de cette requête pour créer l’objet [Activity][Activity] de votre réponse.

1. Définissez la propriété **conversation** sur le contenu de la propriété **conversation** dans le message de l’utilisateur.
2. Définissez la propriété **from** sur le contenu de la propriété **recipient** dans le message de l’utilisateur.
3. Définissez la propriété **recipient** sur le contenu de la propriété **from** dans le message de l’utilisateur.
4. Définissez les propriétés **text** et **attachments** selon vos besoins.

Utilisez la propriété `serviceUrl` dans la requête entrante pour [identifier l’URI de base](bot-framework-rest-connector-api-reference.md#base-uri) que votre bot doit utiliser pour émettre sa réponse. 

Pour envoyer la réponse, utilisez la requête `POST` pour publier votre objet [Activity][Activity] sur `/v3/conversations/{conversationId}/activities/{activityId}`, comme indiqué dans l’exemple suivant. Le corps de cette requête est un objet [Activity][Activity] qui invite l’utilisateur à sélectionner une heure de rendez-vous disponible.

```http
POST https://smba.trafficmanager.net/apis/v3/conversations/abcd1234/activities/bf3cc9a2f5de... 
Authorization: Bearer eyJhbGciOiJIUzI1Ni...
Content-Type: application/json
```

```json
{
    "type": "message",
    "from": {
        "id": "12345678",
        "name": "bot's name"
    },
    "conversation": {
        "id": "abcd1234",
        "name": "conversation's name"
   },
   "recipient": {
        "id": "1234abcd",
        "name": "user's name"
    },
    "text": "I have these times available:",
    "replyToId": "bf3cc9a2f5de..."
}
```

Dans cet exemple de requête, `https://smba.trafficmanager.net/apis` représente l’URI de base, qui peut être différent de celui des requêtes émises par votre bot. Pour plus d’informations sur la définition de l’URI de base, consultez l’article [Informations de référence sur l’API](bot-framework-rest-connector-api-reference.md#base-uri). 

> [!IMPORTANT]
> Comme indiqué dans cet exemple, l’en-tête `Authorization` de chaque requête d’API que vous envoyez doit contenir le mot **Bearer** suivi du jeton d’accès que vous avez [obtenu auprès de Bot Framework](#get-token).

Pour envoyer un autre message permettant à un utilisateur de sélectionner une heure de rendez-vous disponible en cliquant sur un bouton, utilisez la requête `POST` pour publier une autre requête destinée au même point de terminaison :

```http
POST https://smba.trafficmanager.net/apis/v3/conversations/abcd1234/activities/bf3cc9a2f5de... 
Authorization: Bearer eyJhbGciOiJIUzI1Ni...
Content-Type: application/json
```

```json
{
    "type": "message",
    "from": {
        "id": "12345678",
        "name": "bot's name"
    },
    "conversation": {
        "id": "abcd1234",
        "name": "conversation's name"
   },
   "recipient": {
        "id": "1234abcd",
        "name": "user's name"
    },
    "attachmentLayout": "list",
    "attachments": [
      {
        "contentType": "application/vnd.microsoft.card.thumbnail",
        "content": {
          "buttons": [
            {
              "type": "imBack",
              "title": "10:30",
              "value": "10:30"
            },
            {
              "type": "imBack",
              "title": "11:30",
              "value": "11:30"
            },
            {
              "type": "openUrl",
              "title": "See more",
              "value": "http://www.contososalon.com/scheduling"
            }
          ]
        }
      }
    ],
    "replyToId": "bf3cc9a2f5de..."
}
```   

## <a name="next-steps"></a>Étapes suivantes

Ce didacticiel vous a appris à obtenir un jeton d’accès à partir de Bot Framework et à utiliser le service Bot Connecteur pour échanger des messages avec l’utilisateur. Vous pouvez utiliser l’application [Bot Framework Emulator](../bot-service-debug-emulator.md) pour tester et déboguer votre bot. Si vous souhaitez partager votre bot avec d’autres, vous devrez le [configurer](../bot-service-manage-channels.md) pour qu’il s’exécute sur un ou plusieurs canaux.


[Activity]: bot-framework-rest-connector-api-reference.md#activity-object