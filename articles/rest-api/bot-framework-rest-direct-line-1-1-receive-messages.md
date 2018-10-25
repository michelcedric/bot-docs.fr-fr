---
title: Recevoir des messages du robot | Microsoft Docs
description: Découvrez comment recevoir des messages du robot à l’aide de l’API Direct Line v1.1.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
ms.openlocfilehash: 6679afa688917bdf3d558d5ed47717ee30d0e52e
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2018
ms.locfileid: "49999356"
---
# <a name="receive-messages-from-the-bot"></a>Recevoir des messages du robot

> [!IMPORTANT]
> Cet article décrit comment recevoir des messages du robot à l’aide de l’API Direct Line 1.1. Si vous créez une connexion entre votre application cliente et votre bot, utilisez plutôt [l’API Direct Line 3.0](bot-framework-rest-direct-line-3-0-receive-activities.md).

À l’aide du protocole Direct Line 1.1, les clients doivent interroger une interface `HTTP GET` pour recevoir des messages. 

## <a name="retrieve-messages-with-http-get"></a>Récupérer des messages avec HTTP GET

Pour récupérer les messages d’une conversation, envoyez une requête `GET` au point de terminaison `api/conversations/{conversationId}/messages`, en spécifiant, si vous le souhaitez, le paramètre `watermark`, pour indiquer le dernier message que le client a vu. Une valeur `watermark` mise à jour sera retournée dans la réponse JSON, même si aucun message n’est inclus.

Les extraits de code suivants illustrent la demande et la réponse Obtenir les messages. La réponse Get Messages (Obtenir les messages) contient `watermark` en tant que propriété de [ MessageSet ](bot-framework-rest-direct-line-1-1-api-reference.md#messageset-object). Les clients doivent parcourir les messages disponibles en augmentant la valeur `watermark` jusqu’à ce qu’aucun message ne soit plus renvoyé. 

### <a name="request"></a>Requête

```http
GET https://directline.botframework.com/api/conversations/abc123/messages?watermark=0001a-94
Authorization: Bearer RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0
```

### <a name="response"></a>response

```http
HTTP/1.1 200 OK
[other headers]
```

```json
{
    "messages": [
        {
            "conversation": "abc123",
            "id": "abc123|0000",
            "text": "hello",
            "from": "user1"
        }, 
        {
            "conversation": "abc123",
            "id": "abc123|0001",
            "text": "Nice to see you, user1!",
            "from": "bot1"
        }
    ],
    "watermark": "0001a-95"
}
```

## <a name="timing-considerations"></a>Éléments à prendre en compte concernant le temps

Même si Direct Line est un protocole en plusieurs parties avec de potentiels écarts de temps, le protocole et le service sont conçus pour faciliter la création d’un client fiable. La propriété `watermark` envoyée dans la réponse Get Messages (Obtenir les messages) est fiable. Tant qu’il relira la chaîne textuelle du filigrane, le client ne manquera aucun message.

Les clients doivent choisir un intervalle d’interrogation correspondant à leur utilisation prévue.

- Les applications de service à service utilisent souvent un intervalle d’interrogation de 5 ou 10 secondes.

- Les applications orientées client utilisent souvent un intervalle d’interrogation de 1 seconde, et émettent une requête supplémentaire environ 300 millisecondes après chaque message envoyé par le client (pour récupérer rapidement la réponse d’un robot). Ce délai de 300 millisecondes doit être ajusté en fonction de la rapidité du robot et de son temps de transit.

## <a name="additional-resources"></a>Ressources supplémentaires

- [Concepts clés](bot-framework-rest-direct-line-1-1-concepts.md)
- [Authentification](bot-framework-rest-direct-line-1-1-authentication.md)
- [Démarrer une conversation](bot-framework-rest-direct-line-1-1-start-conversation.md)
- [Envoyer un message au robot](bot-framework-rest-direct-line-1-1-send-message.md)