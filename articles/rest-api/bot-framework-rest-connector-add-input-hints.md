---
title: Ajouter des conseils de saisie aux messages | Microsoft Docs
description: Découvrez comment ajouter des conseils de saisie aux messages à l’aide du service Bot Connector.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: 66c6bc20013ff2de82e29af76e9c99898c8b13d9
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39299873"
---
# <a name="add-input-hints-to-messages"></a>Ajouter des conseils de saisie aux messages
> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-add-input-hints.md)
> - [Node.JS](../nodejs/bot-builder-nodejs-send-input-hints.md)
> - [REST](../rest-api/bot-framework-rest-connector-add-input-hints.md)

En spécifiant un conseil de saisie pour un message, vous pouvez indiquer si votre robot accepte, attend ou ignore l’entrée utilisateur une fois le message remis au client. Pour de nombreux canaux, cela permet aux clients de définir l’état des contrôles d’entrée utilisateur en conséquence. Par exemple, si le conseil de saisie d’un message indique que le bot ignore l’entrée utilisateur, le client peut fermer le microphone et désactiver la zone de saisie pour empêcher l’utilisateur de fournir une entrée.

## <a name="accepting-input"></a>Acceptation d’entrées

Pour indiquer que votre bot est passivement prêt pour l’entrée sans être en attente d’une réponse de l’utilisateur, définissez la propriété `inputHint` sur **acceptingInput** dans l’objet [Activity][Activity] représentant votre message. Sur de nombreux canaux, cela entraîne l’activation de la zone de saisie du client, ainsi que la coupure du microphone, tout en restant accessible à l’utilisateur. Par exemple, Cortana ouvre le microphone afin d’accepter une entrée de l’utilisateur si l’utilisateur maintient le bouton du microphone enfoncé. 

L’exemple suivant montre une demande qui envoie un message et spécifie que le robot accepte l’entrée. Dans cet exemple de demande, `https://smba.trafficmanager.net/apis` représente l’URI de base. L’URI de base pour les demandes émises par votre robot peut être différente. Pour plus d’informations sur la définition de l’URI de base, voir [Informations de référence sur l’API](bot-framework-rest-connector-api-reference.md#base-uri).

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
        "name": "sender's name"
    },
    "conversation": {
        "id": "abcd1234",
        "name": "conversation's name"
   },
   "recipient": {
        "id": "1234abcd",
        "name": "recipient's name"
    },
    "text": "Here's a picture of the house I was telling you about.",
    "inputHint": "acceptingInput",
    "replyToId": "5d5cdc723"
}
```

## <a name="expecting-input"></a>Attente d’entrée

Pour indiquer que votre robot est en attente d’une réponse de l’utilisateur, définissez la propriété `inputHint` sur **expectingInput** dans l’objet [Activité][Activity] représentant votre message. Sur de nombreux canaux, cela entraîne l’activation de la zone de saisie du client, ainsi que l’ouverture du microphone. 

L’exemple suivant montre une demande qui envoie un message et spécifie que le robot attend une entrée. Dans cet exemple de demande, `https://smba.trafficmanager.net/apis` représente l’URI de base. L’URI de base pour les demandes émises par votre robot peut être différente. Pour plus d’informations sur la définition de l’URI de base, voir [Informations de référence sur l’API](bot-framework-rest-connector-api-reference.md#base-uri).

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
        "name": "sender's name"
    },
    "conversation": {
        "id": "abcd1234",
        "name": "conversation's name"
   },
   "recipient": {
        "id": "1234abcd",
        "name": "recipient's name"
    },
    "text": "What is your favorite color?",
    "inputHint": "expectingInput",
    "replyToId": "5d5cdc723"
}
```

## <a name="ignoring-input"></a>Ignorer l’entrée
 
Pour indiquer que votre robot n’est pas prêt à recevoir une entrée de l’utilisateur, définissez la propriété `inputHint` sur **ignoringInput** dans l’objet [Activité][Activity] représentant votre message. Sur de nombreux canaux, cela entraîne la désactivation de la zone de saisie du client ainsi que la coupure du microphone. 

L’exemple suivant montre une demande qui envoie un message et spécifie que le robot ignore l’entrée. Dans cet exemple de demande, `https://smba.trafficmanager.net/apis` représente l’URI de base. L’URI de base pour les demandes émises par votre robot peut être différente. Pour plus d’informations sur la définition de l’URI de base, voir [Informations de référence sur l’API](bot-framework-rest-connector-api-reference.md#base-uri).

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
        "name": "sender's name"
    },
    "conversation": {
        "id": "abcd1234",
        "name": "conversation's name"
   },
   "recipient": {
        "id": "1234abcd",
        "name": "recipient's name"
    },
    "text": "Please hold while I perform the calculation.",
    "inputHint": "ignoringInput",
    "replyToId": "5d5cdc723"
}
```

## <a name="additional-resources"></a>Ressources supplémentaires

- [Créer des messages](bot-framework-rest-connector-create-messages.md)
- [Envoyer et recevoir des messages](bot-framework-rest-connector-send-and-receive-messages.md)

[Activity]: bot-framework-rest-connector-api-reference.md#activity-object