---
title: Envoyer et recevoir des messages | Microsoft Docs
description: Découvrez comment envoyer et recevoir des messages à l’aide du service Bot Connector.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: 60d96297ea4bdc6ba920b4f8f990fabb0af8b8d9
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39298610"
---
# <a name="send-and-receive-messages"></a>Envoyer et recevoir des messages

Le service Bot Connector permet à un bot de communiquer sur plusieurs canaux tels que Skype, Email, Slack, et bien plus encore. Il facilite la communication entre bot et utilisateur en relayant les [activités](bot-framework-rest-connector-activities.md) du bot vers le canal, et du canal vers le bot. Chaque activité contient des informations utilisées pour acheminer le message vers la destination appropriée, ainsi que des informations sur l’auteur du message, le contexte du message et le destinataire du message. Cet article explique comment utiliser le service Bot Connector pour échanger des activités de **message** entre bot et utilisateur sur un canal. 

## <a id="create-reply"></a> Répondre à un message

### <a name="create-a-reply"></a>Créer une réponse 

Lorsque l’utilisateur envoie un message à votre bot, ce dernier reçoit le message comme un objet [Activity][Activity]de type **message**. Pour créer une réponse au message d’un utilisateur, créez un nouvel objet [Activity][Activity], puis définissez les propriétés suivantes :

| Propriété | Valeur |
|----|----|
| conversation | Définissez cette propriété sur le contenu de la propriété `conversation` dans le message de l’utilisateur. |
| from | Définissez cette propriété sur le contenu de la propriété `recipient` dans le message de l’utilisateur. |
| locale | Définissez cette propriété sur le contenu de la propriété `locale` dans le message de l’utilisateur, si spécifié. |
| recipient | Définissez cette propriété sur le contenu de la propriété `from` dans le message de l’utilisateur. |
| replyToId | Définissez cette propriété sur le contenu de la propriété `id` dans le message de l’utilisateur. |
| Type | Définissez cette propriété sur **message**. |

Ensuite, définissez les propriétés qui spécifient les informations que vous souhaitez communiquer à l’utilisateur. Par exemple, vous pouvez définir la propriété `text` pour spécifier le texte à afficher dans le message, définir la propriété `speak` pour spécifier le texte qui doit être prononcé par votre bot, et définir la propriété `attachments` pour spécifier les pièces jointes multimédias ou les cartes enrichies à inclure dans le message. Pour plus d’informations sur les propriétés de message les plus couramment utilisées, consultez [Créer des messages](bot-framework-rest-connector-create-messages.md).

### <a name="send-the-reply"></a>Envoyer la réponse

Utilisez la propriété `serviceUrl` dans l’activité entrante pour [identifier l’URI de base](bot-framework-rest-connector-api-reference.md#base-uri) que votre bot doit utiliser pour émettre sa réponse. 

Pour envoyer la réponse, envoyez la requête suivante : 

```http
POST /v3/conversations/{conversationId}/activities/{activityId}
```

Dans cet URI de requête, remplacez **{conversationId}** par la valeur de la propriété `id` de l’objet `conversation` de votre activité de réponse, et remplacez **{activityId}** par la valeur de la propriété `replyToId` de votre activité de réponse. Définissez le corps de la requête sur l’objet [Activity][Activity] que vous avez créé pour représenter votre réponse.

L’exemple suivant montre une requête qui envoie une réponse textuelle simple au message d’un utilisateur. Dans cet exemple de requête, `https://smba.trafficmanager.net/apis` représente l’URI de base, qui peut être différent de celui des requêtes émises par votre bot. Pour plus d’informations sur la définition de l’URI de base, consultez [API Reference](bot-framework-rest-connector-api-reference.md#base-uri) (Informations de référence sur l’API).

```http
POST https://smba.trafficmanager.net/apis/v3/conversations/abcd1234/activities/5d5cdc723 
Authorization: Bearer ACCESS_TOKEN 
Content-Type: application/json 
```

```json
{
    "type": "message",
    "from": {
        "id": "12345678",
        "name": "Pepper's News Feed"
    },
    "conversation": {
        "id": "abcd1234",
        "name": "Convo1"
   },
   "recipient": {
        "id": "1234abcd",
        "name": "SteveW"
    },
    "text": "My bot's reply",
    "replyToId": "5d5cdc723"
}
```

## <a id="send-message"></a> Envoyer un message (autre qu’une réponse)

La majorité des messages envoyés par votre bot le seront en réponse aux messages reçus par l’utilisateur. Toutefois, il peut arriver que votre bot doive envoyer un message à la conversation qui n’est pas une réponse directe à un message envoyé par l’utilisateur. Par exemple, votre bot peut avoir besoin de lancer un nouveau sujet de conversation ou d’envoyer un message pour dire au revoir à la fin d’une conversation. 

Pour envoyer un message à une conversation qui ne soit pas une réponse directe à un message de l’utilisateur, exécutez la requête suivante : 

```http
POST /v3/conversations/{conversationId}/activities
```

Dans cet URI de requête, remplacez **{conversationId}** par l’ID de la conversation. 
    
Définissez le corps de la requête sur l’objet [Activity][Activity] que vous avez créé pour représenter votre réponse.

> [!NOTE]
> Bot Framework n’impose aucune restriction concernant le nombre de messages qu’un bot peut envoyer. Toutefois, la plupart des canaux appliquent des limitations de requêtes pour empêcher les bots d’envoyer un grand nombre de messages dans un court laps de temps. En outre, si le bot envoie plusieurs messages successifs rapidement, le canal risque de ne pas toujours afficher les messages dans le bon ordre.

## <a name="start-a-conversation"></a>Démarrer une conversation

Il peut arriver que votre bot ait besoin de démarrer une conversation avec un ou plusieurs utilisateurs. Pour démarrer une conversation avec un utilisateur sur un canal, votre bot doit connaître ses informations de compte, ainsi que les informations de compte de l’utilisateur sur ce canal. 

> [!TIP]
> Si votre bot aura besoin plus tard de démarrer des conversations avec ses utilisateurs, mettez dans le cache les informations de compte d’utilisateur, d’autres informations pertinentes comme les préférences utilisateur et les paramètres régionaux, et enfin, l’URL du service (à utiliser comme URI de base dans la requête Start Conversation). 

Pour démarrer une conversation, exécutez la requête suivante : 

```http
POST /v3/conversations
```

Définissez le corps de la requête sur un objet [Conversation][Conversation] qui spécifie les informations de compte du bot et les informations de compte des utilisateurs que vous souhaitez inclure dans la conversation.

> [!NOTE]
> Tous les canaux ne prennent pas en charge les conversations de groupe. Consultez la documentation du canal pour déterminer si un canal prend en charge les conversations de groupe, et pour connaître le nombre maximal de participants autorisés par un canal dans une conversation.

L’exemple suivant montre une requête qui démarre une conversation. Dans cet exemple de requête, `https://smba.trafficmanager.net/apis` représente l’URI de base, qui peut être différent de celui des requêtes émises par votre bot. Pour plus d’informations sur la définition de l’URI de base, consultez [API Reference](bot-framework-rest-connector-api-reference.md#base-uri) (Informations de référence sur l’API).

```http
POST https://smba.trafficmanager.net/apis/v3/conversations 
Authorization: Bearer ACCESS_TOKEN
Content-Type: application/json
```

```json
{
    "bot": {
        "id": "12345678",
        "name": "bot's name"
    },
    "isGroup": false,
    "members": [
        {
            "id": "1234abcd",
            "name": "recipient's name"
        }
    ],
    "topicName": "News Alert"
}
```

Si la conversation est établie avec les utilisateurs spécifiés, la réponse contient un ID qui identifie la conversation. 

```json
{
    "id": "abcd1234"
}
```

Votre bot peut ensuite utiliser cet ID de conversation pour [envoyer un message](#send-message) aux utilisateurs dans le cadre de la conversation.

## <a name="additional-resources"></a>Ressources supplémentaires

- [Vue d’ensemble des activités](bot-framework-rest-connector-activities.md)
- [Créer des messages](bot-framework-rest-connector-create-messages.md)

[Activity]: bot-framework-rest-connector-api-reference.md#activity-object
[ConversationAccount]: bot-framework-rest-connector-api-reference.md#conversationaccount-object
[Conversation]: bot-framework-rest-connector-api-reference.md#conversation-object

