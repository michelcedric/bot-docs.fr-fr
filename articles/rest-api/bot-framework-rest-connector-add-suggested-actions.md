---
title: Ajouter des actions suggérées aux messages | Microsoft Docs
description: Découvrez comment ajouter des actions suggérées aux messages à l’aide du service Bot Connector.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: d162bdd3f34848b16380317c776f445bc4611157
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39299877"
---
# <a name="add-suggested-actions-to-messages"></a>Ajouter des actions suggérées aux messages
> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-add-suggested-actions.md)
> - [Node.JS](../nodejs/bot-builder-nodejs-send-suggested-actions.md)
> - [REST](../rest-api/bot-framework-rest-connector-add-suggested-actions.md)

[!INCLUDE [Introduction to suggested actions](../includes/snippet-suggested-actions-intro.md)]

> [!TIP]
> Pour savoir comment les différents canaux restituent les actions suggérées, voir [Inspecteur de canaux][channelInspector].

## <a name="send-suggested-actions"></a>Envoyer des actions suggérées

Pour ajouter des actions suggérées à un message, définissez la propriété `suggestedActions` de l’[Activité][Activity] pour spécifier la liste d’objets [CardAction][CardAction] qui représentent les boutons à présenter à l’utilisateur. 

La demande suivante envoie un message qui présente trois actions suggérées à l’utilisateur. Dans cet exemple de demande, `https://smba.trafficmanager.net/apis` représente l’URI de base. L’URI de base pour les demandes émises par votre robot peut être différente. Pour plus d’informations sur la définition de l’URI de base, voir [Informations de référence sur l’API](bot-framework-rest-connector-api-reference.md#base-uri).

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
    "text": "I have colors in mind, but need your help to choose the best one.",
    "inputHint": "expectingInput",
    "suggestedActions": {
        "actions": [
            {
                "type": "imBack",
                "title": "Blue",
                "value": "Blue"
            },
            {
                "type": "imBack",
                "title": "Red",
                "value": "Red"
            },
            {
                "type": "imBack",
                "title": "Green",
                "value": "Green"
            }
        ]
    },
    "replyToId": "5d5cdc723"
}
```

Lorsque l’utilisateur appuie sur l’une des actions suggérées, le robot reçoit un message de l’utilisateur contenant la valeur `value` de l’action correspondante.

## <a name="additional-resources"></a>Ressources supplémentaires

- [Créer des messages](bot-framework-rest-connector-create-messages.md)
- [Envoyer et recevoir des messages](bot-framework-rest-connector-send-and-receive-messages.md)

[channelInspector]: ../bot-service-channel-inspector.md

[Activity]: bot-framework-rest-connector-api-reference.md#activity-object

[CardAction]: bot-framework-rest-connector-api-reference.md#cardaction-object

[SuggestedAction]: bot-framework-rest-connector-api-reference.md#suggestedactions-object