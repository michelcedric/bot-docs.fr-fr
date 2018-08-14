---
title: Informations de référence sur l’API - API Direct Line 1.1 | Microsoft Docs
description: Découvrez les en-têtes, les codes d’état HTTP, le schéma, les opérations et les objets de l’API Direct Line 1.1.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: 2f688b9c80e762b93c2eba8f4671ff1760f624f9
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39300061"
---
# <a name="api-reference---direct-line-api-11"></a>Informations de référence sur l’API - API Direct Line 1.1

> [!IMPORTANT]
> Cet article contient les informations de référence relatives à l’API Direct Line 1.1. Si vous créez une connexion entre votre application cliente et votre bot, utilisez [l’API Direct Line 3.0](bot-framework-rest-direct-line-3-0-api-reference.md) à la place.

Vous pouvez activer la communication de votre application cliente avec votre bot à l’aide de l’API Direct Line 1.1. L’API Direct Line 1.1 utilise les standards REST et JSON sur HTTPS.

## <a name="base-uri"></a>URI de base

Pour accéder à l’API Direct Line 1.1, utilisez l’URI de base ci-après pour toutes les requêtes d’API :

`https://directline.botframework.com`

## <a name="headers"></a>headers

Outre les en-têtes de requête HTTP standard, une requête d’API Direct Line doit inclure un en-tête `Authorization` qui spécifie un secret ou un jeton pour authentifier le client émettant la requête. Vous pouvez spécifier l’en-tête `Authorization` à l’aide du schéma « Bearer » ou « BotConnector ». 

**Schéma Bearer** :
```http
Authorization: Bearer SECRET_OR_TOKEN
```

**Schéma BotConnector** :
```http
Authorization: BotConnector SECRET_OR_TOKEN
```

Pour plus d’informations sur l’obtention d’un secret ou d’un jeton que votre client peut utiliser pour authentifier ses requêtes d’API Direct Line, consultez l’article [Authentification](bot-framework-rest-direct-line-1-1-authentication.md).

## <a name="http-status-codes"></a>Codes d’état HTTP

Le <a href="http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html" target="_blank">code d’état HTTP</a> renvoyé avec chaque réponse indique le résultat de la requête correspondante. 

| Code d'état HTTP | Signification |
|----|----|
| 200 | La requête a réussi. |
| 204 | La requête a réussi, mais aucun contenu n’a été renvoyé. |
| 400 | La requête présentait un format inadéquat ou était incorrecte. |
| 401 | Le client n’est pas autorisé à adresser cette requête. Dans la plupart des cas, ce code d’état signale que l’en-tête `Authorization` est manquant ou qu’il présente un format incorrect. |
| 403 | Le client n’est pas autorisé à effectuer l’opération demandée. Dans la plupart des cas, ce code d’état signale que l’en-tête `Authorization` spécifie un jeton ou un secret non valides. |
| 404 | La ressource demandée est introuvable. Ce code d’état signale généralement un URI de requête non valide. |
| 500 | Une erreur de serveur interne s’est produite dans le service Direct Line, ou une défaillance s’est produite au sein du bot. Si vous recevez une erreur 500 lors de la publication d’un message sur un bot par le biais d’une requête POST, il est possible que cette erreur ait été déclenchée par une défaillance du bot. **Ce code d’erreur est courant.** |

## <a name="token-operations"></a>Opérations de jeton 
Utilisez les opérations ci-après pour créer ou actualiser un jeton permettant à un client d’accéder à une conversation spécifique.

| Opération | Description |
|----|----|
| [Générer un jeton](#generate-token) | Permet de générer un jeton pour une nouvelle conversation. | 
| [Actualiser le jeton](#refresh-token) | Permet d’actualiser un jeton. | 

### <a name="generate-token"></a>Générer un jeton
Permet de générer un jeton valide pour une seule conversation. 
```http 
POST /api/tokens/conversation
```

| | |
|----|----|
| **Corps de la demande** | n/a |
| **Retourne** | Chaîne représentant le jeton | 

### <a name="refresh-token"></a>Actualiser le jeton
Permet d’actualiser le jeton. 
```http 
GET /api/tokens/{conversationId}/renew
```

| | |
|----|----|
| **Corps de la demande** | n/a |
| **Retourne** | Chaîne représentant le nouveau jeton | 

## <a name="conversation-operations"></a>Opérations de conversation 
Utilisez les opérations ci-après pour ouvrir une conversation avec votre bot et pour échanger des messages entre le client et le bot.

| Opération | Description |
|----|----|
| [Start Conversation](#start-conversation) (Démarrer une conversation) | Ouvre une nouvelle conversation avec le bot. | 
| [Get Messages](#get-messages) | Récupère les messages émanant du bot. |
| [Envoyer un message](#send-a-message) | Envoie un message au bot. | 
| [Upload and Send File(s)](#upload-send-files) (Charger et envoyer des fichiers) | Charge et envoie un ou plusieurs fichiers sous forme de pièces jointes. |

### <a name="start-conversation"></a>Start Conversation (Démarrer une conversation)
Ouvre une nouvelle conversation avec le bot. 
```http 
POST /api/conversations
```

| | |
|----|----|
| **Corps de la demande** | n/a |
| **Retourne** | Objet [Conversation](#conversation-object) | 

### <a name="get-messages"></a>Obtenir les messages
Récupère les messages émanant du bot pour la conversation spécifiée. Définissez le paramètre `watermark` dans l’URI de requête pour indiquer le dernier message vu par le client. 

```http
GET /api/conversations/{conversationId}/messages?watermark={watermark_value}
```

| | |
|----|----|
| **Corps de la demande** | n/a |
| **Retourne** | Objet [MessageSet](#messageset-object). La réponse contient `watermark` en tant que propriété de l’objet `MessageSet`. Les clients doivent parcourir les messages disponibles en augmentant la valeur `watermark` jusqu’à ce qu’aucun message ne soit plus renvoyé. | 

### <a name="send-a-message"></a>Envoyer un message
Envoie un message au bot. 
```http 
POST /api/conversations/{conversationId}/messages
```

| | |
|----|----|
| **Corps de la demande** | Objet [Message](#message-object) |
| **Retourne** | Aucune donnée n’est renvoyée dans le corps de la réponse. Le service répond par un code d’état HTTP 204 si le message a été envoyé avec succès. Le client peut obtenir le message qu’il a envoyé (ainsi que tous les messages qu’il a reçus du bot) en utilisant l’opération [Obtenir les messages](#get-messages). |

### <a id="upload-send-files"></a> Upload and Send File(s) (Charger et envoyer des fichiers)
Charge et envoie un ou plusieurs fichiers sous forme de pièces jointes. Définissez le paramètre `userId` dans l’URI de requête pour spécifier l’ID de l’utilisateur envoyant les pièces jointes.
```http 
POST /api/conversations/{conversationId}/upload?userId={userId}
```

| | |
|----|----|
| **Corps de la demande** | Dans le cas d’une pièce jointe unique, remplissez le corps de la requête avec le contenu du fichier. Dans le cas de plusieurs pièces jointes, créez un corps de requête en plusieurs parties, contenant une partie pour chaque pièce jointe, et également (en option) une partie pour l’objet [Message](#message-object) qui doit servir de conteneur pour les pièces jointes spécifiées. Pour plus d’informations, consultez l’article [Envoyer un message au bot](bot-framework-rest-direct-line-1-1-send-message.md). |
| **Retourne** | Aucune donnée n’est renvoyée dans le corps de la réponse. Le service répond par un code d’état HTTP 204 si le message a été envoyé avec succès. Le client peut obtenir le message qu’il a envoyé (ainsi que tous les messages qu’il a reçus du bot) en utilisant l’opération [Obtenir les messages](#get-messages). | 

> [!NOTE]
> Les fichiers chargés sont supprimés au bout de 24 heures.

## <a name="schema"></a>Schéma

Le schéma Direct Line 1.1 est une copie simplifiée du schéma Bot Framework v1 incluant les objets suivants.

### <a name="message-object"></a>Objet Message

Définit un message qu’un client envoie à un bot ou reçoit d’un bot.

| Propriété | type | Description |
|----|----|----|
| **id** | chaîne | ID identifiant de manière unique le message (attribué par Direct Line). | 
| **conversationId** | chaîne | ID identifiant la conversation.  | 
| **created** | chaîne | Date et heure de création du message, exprimées au format <a href="https://en.wikipedia.org/wiki/ISO_8601" target="_blank">ISO-8601</a>. | 
| **from** | chaîne | ID identifiant l’utilisateur qui a expédié le message. Lorsque vous créez un message, les clients doivent définir cette propriété sur un ID d’utilisateur stable. Bien que Direct Line attribue un ID d’utilisateur si aucun n’est fourni, cette opération entraîne généralement un comportement inattendu. | 
| **text** | chaîne | Texte du message envoyé par l’utilisateur au bot ou par le bot à l’utilisateur. | 
| **channelData** | objet | Objet contenant le contenu propre au canal. Certains canaux fournissent des fonctionnalités qui nécessitent des informations supplémentaires impossibles à représenter à l’aide du schéma de pièce jointe. Dans ce type de cas, définissez cette propriété sur le contenu propre au canal, tel que défini dans la documentation du canal. Ces données sont envoyées telles quelles entre le client et le bot. Cette propriété doit être définie sur un objet complexe ou laissée vide. Ne la définissez pas sur une chaîne, un nombre ou un autre type simple. | 
| **images** | string[] | Tableau de chaînes contenant l’URL des images figurant dans le message. Dans certains cas, les chaînes de ce tableau peuvent être des URL relatives. Si l’une des chaînes de ce tableau ne commence pas par « http » ou « https », ajoutez `https://directline.botframework.com` au début de la chaîne pour former l’URL complète. | 
| **attachments** | [Attachment](#attachment-object)[] | Tableau d’objets **Attachment** représentant les pièces jointes autres que des images qui figurent dans le message. Chaque objet du tableau contient une propriété `url` et une propriété `contentType`. Dans les messages qu’un client reçoit d’un bot, la propriété `url` peut parfois spécifier une URL relative. Si l’une des valeurs de propriété `url` ne commence pas par « http » ou « https », ajoutez `https://directline.botframework.com` au début de la chaîne pour former l’URL complète. | 

L’exemple ci-après présente un objet Message contenant toutes les propriétés possibles. Dans la plupart des cas, lors de la création d’un message, le client doit seulement fournir la propriété `from` et au moins une propriété de contenu (par exemple, `text`, `images`, `attachments` ou `channelData`).

```json
{
    "id": "CuvLPID4kDb|000000000000000004",
    "conversationId": "CuvLPID4kDb",
    "created": "2016-10-28T21:19:51.0357965Z",
    "from": "examplebot",
    "text": "Hello!",
    "channelData": {
        "examplefield": "abc123"
    },
    "images": [
        "/attachments/CuvLPID4kDb/0.jpg?..."
    ],
    "attachments": [
        {
            "url": "https://example.com/example.docx",
            "contentType": "application/vnd.openxmlformats-officedocument.wordprocessingml.document"
        }, 
        {
            "url": "https://example.com/example.doc",
            "contentType": "application/msword"
        }
    ]
}
```

### <a name="messageset-object"></a>Objet MessageSet 
Définit un ensemble de messages.<br/><br/>

| Propriété | type | Description |
|----|----|----|
| **messages** | [Message](#message-object)[] | Tableau d’objets **Message**. |
| **watermark** | chaîne | Filigrane maximal des messages au sein de l’ensemble. Un client peut utiliser la valeur `watermark` pour indiquer le dernier message qu’il a vu lors de la [récupération des messages émanant du bot](bot-framework-rest-direct-line-1-1-receive-messages.md). |

### <a name="attachment-object"></a>Objet Attachment
Définit une pièce jointe autre qu’une image.<br/><br/> 

| Propriété | type | Description |
|----|----|----|
| **contentType** | chaîne | Type de média du contenu de la pièce jointe. |
| **url** | chaîne | URL du contenu de la pièce jointe. |

### <a name="conversation-object"></a>Objet Conversation
Définit une conversation Direct Line.<br/><br/>

| Propriété | type | Description |
|----|----|----|
| **conversationId** | chaîne | ID identifiant de manière unique la conversation pour laquelle le jeton spécifié est valide. |
| **token** | chaîne | Jeton valide pour la conversation spécifiée. |
| **expires_in** | number | Nombre de secondes avant l’expiration du jeton. |

### <a name="error-object"></a>Objet Error
Définit une erreur.<br/><br/> 

| Propriété | type | Description |
|----|----|----|
| **code** | chaîne | Code d’erreur. Valeurs possibles : **MissingProperty**, **MalformedData**, **NotFound**, **ServiceError**, **Internal**, **InvalidRange**, **NotSupported**, **NotAllowed**, **BadCertificate**. |
| **message** | chaîne | Description de l’erreur. |
| **statusCode** | number | Code d’état. |

### <a name="errormessage-object"></a>Objet ErrorMessage
Charge utile d’erreur de message normalisée.<br/><br/> 


|        Propriété        |          type          |                                 Description                                 |
|------------------------|------------------------|-----------------------------------------------------------------------------|
| <strong>error</strong> | [Error](#error-object) | Objet <strong>Error</strong> contenant des informations concernant l’erreur. |

