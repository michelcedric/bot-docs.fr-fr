---
title: Recevoir des activités du bot | Microsoft Docs
description: Découvrez comment envoyer des activités à partir du bot à l’aide de l’API Direct Line v3.0.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
ms.openlocfilehash: c2d4b9a8e2b8ffc1656df44e04ee1bde912e36ea
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2018
ms.locfileid: "49998156"
---
# <a name="receive-activities-from-the-bot"></a>Recevoir des activités du bot

À l’aide du protocole Direct Line 3.0, les clients peuvent recevoir des activités via le flux `WebSocket` ou récupérer des activités en envoyant des requêtes `HTTP GET`. 

## <a name="websocket-vs-http-get"></a>Comparaison entre WebSocket et HTTP GET

Un WebSocket qui diffuse des données en continu envoie (push) les messages aux clients, alors que l’interface GET permet aux clients de demander explicitement des messages. Bien que l’utilisation des WebSockets soit souvent préférable en raison de leur efficacité, l’utilisation de GET peut être utile pour les clients qui ne peuvent pas utiliser de WebSockets ou pour ceux qui souhaitent récupérer l’historique des conversations. 

Certains [types d’activités](bot-framework-rest-connector-activities.md) ne sont pas disponibles à la fois dans WebSocket et dans HTTP GET. Le tableau suivant montre la disponibilité des différents types d’activités pour les clients qui utilisent le protocole Direct Line.

| Type d’activité | Disponibilité | 
|----|----|
| Message | HTTP GET et WebSocket |
| typing | WebSocket uniquement |
| conversationUpdate | Non envoyé/reçu via le client |
| contactRelationUpdate | Non pris en charge dans Direct Line |
| endOfConversation | HTTP GET et WebSocket |
| Tous les autres types d’activités | HTTP GET et WebSocket |

## <a id="connect-via-websocket"></a> Recevoir des activités par le biais d’un flux WebSocket

Lorsqu’un client envoie une requête [Démarrer une conversation](bot-framework-rest-direct-line-3-0-start-conversation.md) pour ouvrir une conversation avec un bot, la réponse du service inclut la propriété `streamUrl`, que le client peut ensuite utiliser pour se connecter via WebSocket. L’URL de flux est préautorisée. De fait, la requête du client pour se connecter via WebSocket ne nécessite pas d’un en-tête `Authorization`.

L’exemple suivant montre une requête qui utilise un `streamUrl` pour se connecter via WebSocket.

```http
-- connect to wss://directline.botframework.com --
GET /v3/directline/conversations/abc123/stream?t=RCurR_XV9ZA.cwA..."
Upgrade: websocket
Connection: upgrade
[other headers]
```

Le service répond avec le code d’état HTTP 101 (« Échange de protocoles »).

```http
HTTP/1.1 101 Switching Protocols
[other headers]
```

### <a name="receive-messages"></a>Recevoir des messages

Une fois connecté via WebSocket, le client peut recevoir les types de messages suivants de la part du service Direct Line :

- Un message qui contient un [ActivitySet](bot-framework-rest-direct-line-3-0-api-reference.md#activityset-object) avec une ou plusieurs activités, et un filigrane (décrits plus bas)
- Un message vide, que le service Direct Line utilise pour vérifier que la connexion est toujours valide
- Types supplémentaires, à définir ultérieurement Ces types sont identifiés par les propriétés de la racine JSON.

Un `ActivitySet` contient des messages envoyés par le bot et par tous les utilisateurs de la conversation. L’exemple suivant montre un `ActivitySet` qui contient un seul message.

```json
{
    "activities": [
        {
            "type": "message",
            "channelId": "directline",
            "conversation": {
                "id": "abc123"
            },
            "id": "abc123|0000",
            "from": {
                "id": "user1"
            },
            "text": "hello"
        }
    ],
    "watermark": "0000a-42"
}
```

### <a name="process-messages"></a>Traitement des messages

Un client doit effectuer le suivi de la valeur `watermark` qu’il reçoit dans chaque [ActivitySet](bot-framework-rest-direct-line-3-0-api-reference.md#activityset-object) de manière à pouvoir utiliser le filigrane. Ainsi, il peut garantir qu’aucun message ne sera perdu s’il perd sa connexion et doit se [reconnecter à la conversation](bot-framework-rest-direct-line-3-0-reconnect-to-conversation.md). Si le client reçoit un `ActivitySet`, dans lequel la propriété `watermark` est `null` ou est manquante, il doit ignorer cette valeur et ne pas remplacer le filigrane qu’il a reçu précédemment.

Un client doit ignorer les messages vides qu’il reçoit de la part du service Direct Line.

Un client peut envoyer des messages vides au service Direct Line pour vérifier la connectivité. Le service Direct Line va ignorer les messages vides qu’il reçoit de la part du client.

Le service Direct Line peut forcer la fermeture de la connexion WebSocket sous certaines conditions. Si le client n’a pas reçu d’activité `endOfConversation`, il peut générer une [nouvelle URL de flux WebSocket](bot-framework-rest-direct-line-3-0-reconnect-to-conversation.md), qu’il peut utiliser pour se reconnecter à la conversation. 

Le flux WebSocket contient des mises à jour automatiques et des messages très récents (puisque l’appel pour se connecter via WebSocket a été émis), mais il n’inclut pas les messages qui ont été envoyés avant le plus récent `POST` à `/v3/directline/conversations/{id}`. Pour récupérer les messages qui ont été envoyés plus tôt dans la conversation, utilisez `HTTP GET` comme décrit ci-dessous.

## <a id="http-get"></a> Récupérer des activités avec HTTP GET

Les clients qui ne peuvent pas utiliser les WebSockets, ou ceux qui souhaitent obtenir l’historique des conversations peuvent récupérer des activités à l’aide de `HTTP GET`.

Pour récupérer les messages d’une conversation, envoyez une requête `GET` au point de terminaison `/v3/directline/conversations/{conversationId}/activities`, en spécifiant, si vous le souhaitez, le paramètre `watermark`, pour indiquer le dernier message qu’a vu le client. 

Les extraits de code suivants montrent un exemple de requête Obtenir les activités de la conversation, avec sa réponse. La réponse à la requête contient `watermark` en tant que propriété de [ActivitySet](bot-framework-rest-direct-line-3-0-api-reference.md#activityset-object). Les clients doivent parcourir les activités disponibles en augmentant la valeur `watermark` jusqu’à ce qu’aucune activité ne soit plus retournée.

### <a name="request"></a>Requête

```http
GET https://directline.botframework.com/v3/directline/conversations/abc123/activities?watermark=0001a-94
Authorization: Bearer RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0
```

### <a name="response"></a>response

```http
HTTP/1.1 200 OK
[other headers]
```

```json
{
    "activities": [
        {
            "type": "message",
            "channelId": "directline",
            "conversation": {
                "id": "abc123"
            },
            "id": "abc123|0000",
            "from": {
                "id": "user1"
            },
            "text": "hello"
        }, 
        {
            "type": "message",
            "channelId": "directline",
            "conversation": {
                "id": "abc123"
            },
            "id": "abc123|0001",
            "from": {
                "id": "bot1"
            },
            "text": "Nice to see you, user1!"
        }
    ],
    "watermark": "0001a-95"
}
```

## <a name="timing-considerations"></a>Éléments à prendre en compte concernant le temps

La plupart des clients souhaitent conserver l’historique complet de leurs messages. Même si Direct Line est un protocole en plusieurs parties avec de potentiels écarts de temps, le protocole et le service sont conçus pour faciliter la création d’un client fiable.

- La propriété `watermark` qui est envoyée dans le flux WebSocket et dans la réponse à la requête pour obtenir les activités de conversation, est une propriété fiable. Tant qu’il relira la chaîne textuelle du filigrane, le client ne manquera aucun message.

- Lorsqu’un client démarre une conversation et se connecte au flux WebSocket, toutes les activités qui sont envoyées après la publication (POST), mais avant l’ouverture du socket, sont relues avant toute nouvelle activité.

- Lorsqu’un client envoie une requête pour obtenir les activités d’une conversation (pour actualiser l’historique) alors qu’il est connecté au flux WebSocket, les activités peuvent être dupliquées sur les deux canaux. Les clients doivent effectuer le suivi de tous les ID d’activité connus afin de pouvoir rejeter les activités en double, le cas échéant.

Les clients qui interrogent les données à l’aide de `HTTP GET` doivent choisir un intervalle d’interrogation qui corresponde à leur utilisation prévue.

- Les applications de service à service utilisent souvent un intervalle d’interrogation de 5 ou 10 secondes.

- Les applications orientées client utilisent souvent un intervalle d’interrogation de 1 seconde, et émettent une requête supplémentaire environ 300 millisecondes après chaque message envoyé par le client (pour récupérer rapidement la réponse d’un robot). Ce délai de 300 millisecondes doit être ajusté en fonction de la rapidité du robot et de son temps de transit.

## <a name="additional-resources"></a>Ressources supplémentaires

- [Concepts clés](bot-framework-rest-direct-line-3-0-concepts.md)
- [Authentification](bot-framework-rest-direct-line-3-0-authentication.md)
- [Démarrer une conversation](bot-framework-rest-direct-line-3-0-start-conversation.md)
- [Se reconnecter à une conversation](bot-framework-rest-direct-line-3-0-reconnect-to-conversation.md)
- [Envoyer une activité au bot](bot-framework-rest-direct-line-3-0-send-activity.md)
- [Mettre fin à une conversation](bot-framework-rest-direct-line-3-0-end-conversation.md)