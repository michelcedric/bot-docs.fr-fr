---
title: Informations de référence sur l’API - API Direct Line 3.0 | Microsoft Docs
description: Découvrez les en-têtes, les codes d’état HTTP, le schéma, les opérations et les objets de l’API Direct Line 3.0.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: 2e47591b04a91ce02cfeb6bd6485080426d201b5
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39299197"
---
# <a name="api-reference---direct-line-api-30"></a>Informations de référence sur l’API - API Direct Line 3.0

Vous pouvez activer la communication de votre application cliente avec votre bot à l’aide de l’API Direct Line 3.0. L’API Direct Line 3.0 utilise les standards REST et JSON sur HTTPS.

## <a name="base-uri"></a>URI de base

Pour accéder à l’API Direct Line 3.0, utilisez l’URI de base ci-après pour toutes les requêtes d’API :

`https://directline.botframework.com`

## <a name="headers"></a>headers

Outre les en-têtes de requête HTTP standard, une requête d’API Direct Line doit inclure un en-tête `Authorization` qui spécifie un secret ou un jeton pour authentifier le client émettant la requête. Spécifiez l’en-tête `Authorization` au format suivant :

```http
Authorization: Bearer SECRET_OR_TOKEN
```

Pour plus d’informations sur l’obtention d’un secret ou d’un jeton que votre client peut utiliser pour authentifier ses requêtes d’API Direct Line, consultez l’article [Authentification](bot-framework-rest-direct-line-3-0-authentication.md).

## <a name="http-status-codes"></a>Codes d’état HTTP

Le <a href="http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html" target="_blank">code d’état HTTP</a> retourné avec chaque réponse indique le résultat de la requête correspondante. 

| Code d'état HTTP | Signification |
|----|----|
| 200 | La requête a réussi. |
| 201 | La requête a réussi. |
| 202 | La requête a été acceptée en vue de son traitement. |
| 204 | La requête a réussi, mais aucun contenu n’a été retourné. |
| 400 | La requête présentait un format inadéquat ou était incorrecte. |
| 401 | Le client n’est pas autorisé à adresser cette requête. Dans la plupart des cas, ce code d’état signale que l’en-tête `Authorization` est manquant ou qu’il présente un format incorrect. |
| 403 | Le client n’est pas autorisé à effectuer l’opération demandée. Si la requête a spécifié un jeton préalablement valide, mais qui a expiré, la propriété `code` de l’[erreur](bot-framework-rest-connector-api-reference.md#error-object) retournée dans l’objet [ErrorResponse](bot-framework-rest-connector-api-reference.md#errorresponse-object) est définie sur `TokenExpired`. |
| 404 | La ressource demandée est introuvable. Ce code d’état signale généralement un URI de requête non valide. |
| 500 | Une erreur de serveur interne s’est produite dans le service Direct Line. |
| 502 | Le bot n’est pas disponible ou a retourné une erreur. **Ce code d’erreur est courant.** |

> [!NOTE]
> Le code d’état HTTP 101 est utilisé dans le chemin d’accès de la connexion WebSocket, bien que ceci soit probablement géré par votre client WebSocket.

### <a name="errors"></a>Errors

Toute réponse qui spécifie un code d’état HTTP dans la plage 4xx ou 5xx va inclure un objet [ErrorResponse](bot-framework-rest-connector-api-reference.md#errorresponse-object) dans le corps de la réponse qui fournit des informations sur l’erreur. Si vous recevez une réponse d’erreur dans la plage 4xx, examinez l’objet **ErrorResponse** pour identifier la cause de l’erreur et résoudre votre problème avant de renvoyer la requête.

> [!NOTE]
> Les codes d’état HTTP et les valeurs spécifiées dans la propriété `code` à l’intérieur de l’objet **ErrorResponse** sont stables. Les valeurs spécifiées dans la propriété `message` à l’intérieur de l’objet **ErrorResponse** peuvent changer au fil du temps.

Les extraits suivants montrent un exemple de requête et la réponse d’erreur qui en résulte.

#### <a name="request"></a>Requête

```http
POST https://directline.botframework.com/v3/directline/conversations/abc123/activities
[detail omitted]
```

#### <a name="response"></a>response
```http
HTTP/1.1 502 Bad Gateway
[other headers]
```
```json
{
    "error": {
        "code": "BotRejectedActivity",
        "message": "Failed to send activity: bot returned an error"
    }
}
```

## <a name="token-operations"></a>Opérations de jeton 
Utilisez les opérations ci-après pour créer ou actualiser un jeton permettant à un client d’accéder à une conversation spécifique.

| Opération | Description |
|----|----|
| [Générer un jeton](#generate-token) | Permet de générer un jeton pour une nouvelle conversation. | 
| [Actualiser le jeton](#refresh-token) | Permet d’actualiser un jeton. | 

### <a name="generate-token"></a>Générer un jeton
Permet de générer un jeton valide pour une seule conversation. 
```http 
POST /v3/directline/tokens/generate
```

| | |
|----|----|
| **Corps de la demande** | n/a |
| **Retourne** | Objet [Conversation](#conversation-object) | 

### <a name="refresh-token"></a>Actualiser le jeton
Permet d’actualiser le jeton. 
```http 
POST /v3/directline/tokens/refresh
```

| | |
|----|----|
| **Corps de la demande** | n/a |
| **Retourne** | Objet [Conversation](#conversation-object) | 

## <a name="conversation-operations"></a>Opérations de conversation 
Utilisez les opérations ci-après pour ouvrir une conversation avec votre bot et pour échanger des activités entre le client et le bot.

| Opération | Description |
|----|----|
| [Start Conversation](#start-conversation) (Démarrer une conversation) | Ouvre une nouvelle conversation avec le bot. | 
| [Obtenir des informations de Conversation](#get-conversation-information) | Obtient des informations sur une conversation existante. Cette opération génère une nouvelle URL de flux de données WebSocket qu’un client peut utiliser pour se [reconnecter](bot-framework-rest-direct-line-3-0-reconnect-to-conversation.md) à une conversation. |
| [Obtenir des activités](#get-activities) | Récupère des activités du bot. |
| [Envoyer une activité](#send-an-activity) | Envoyer une activité au bot. | 
| [Upload and Send File(s)](#upload-send-files) (Charger et envoyer des fichiers) | Charge et envoie un ou plusieurs fichiers sous forme de pièces jointes. |

### <a name="start-conversation"></a>Start Conversation (Démarrer une conversation)
Ouvre une nouvelle conversation avec le bot. 
```http 
POST /v3/directline/conversations
```

| | |
|----|----|
| **Corps de la demande** | n/a |
| **Retourne** | Objet [Conversation](#conversation-object) | 

### <a name="get-conversation-information"></a>Obtenir des informations de Conversation
Cette opération obtient des informations sur une conversation existante et génère également une nouvelle URL de flux de données WebSocket qu’un client peut utiliser pour se [reconnecter](bot-framework-rest-direct-line-3-0-reconnect-to-conversation.md) à une conversation. Vous pouvez éventuellement fournir le paramètre `watermark` dans l’URI de requête pour indiquer le dernier message vu par le client.
```http 
GET /v3/directline/conversations/{conversationId}?watermark={watermark_value}
```

| | |
|----|----|
| **Corps de la demande** | n/a |
| **Retourne** | Objet [Conversation](#conversation-object) | 

### <a name="get-activities"></a>Obtenir des activités
Récupère des activités provenant du bot pour la conversation spécifiée. Vous pouvez éventuellement fournir le paramètre `watermark` dans l’URI de requête pour indiquer le dernier message vu par le client. 

```http 
GET /v3/directline/conversations/{conversationId}/activities?watermark={watermark_value}
```

| | |
|----|----|
| **Corps de la demande** | n/a |
| **Retourne** | Objet [ActivitySet](#activityset-object). La réponse contient `watermark` en tant que propriété de l’objet `ActivitySet`. Les clients doivent parcourir les activités disponibles en augmentant la valeur `watermark` jusqu’à ce qu’aucune activité ne soit plus retournée. | 

### <a name="send-an-activity"></a>Envoyer une activité
Envoyer une activité au bot. 
```http 
POST /v3/directline/conversations/{conversationId}/activities
```

| | |
|----|----|
| **Corps de la demande** | Objet [Activity](bot-framework-rest-connector-api-reference.md#activity-object) |
| **Retourne** | Objet [ResourceResponse](bot-framework-rest-connector-api-reference.md#resourceresponse-object) qui contient une propriété `id` spécifiant l’ID de l’activité envoyée au bot. | 

### <a id="upload-send-files"></a> Upload and Send File(s) (Charger et envoyer des fichiers)
Charge et envoie un ou plusieurs fichiers sous forme de pièces jointes. Définissez le paramètre `userId` dans l’URI de requête pour spécifier l’ID de l’utilisateur envoyant la ou les pièces jointes.
```http 
POST /v3/directline/conversations/{conversationId}/upload?userId={userId}
```

| | |
|----|----|
| **Corps de la demande** | Dans le cas d’une pièce jointe unique, remplissez le corps de la requête avec le contenu du fichier. Dans le cas de plusieurs pièces jointes, créez un corps de requête en plusieurs parties, contenant une partie pour chaque pièce jointe, et également (en option) une partie pour l’objet [Activity](bot-framework-rest-connector-api-reference.md#activity-object) qui doit servir de conteneur aux pièces jointes spécifiées. Pour plus d’informations, consultez [Envoyer une activité au bot](bot-framework-rest-direct-line-3-0-send-activity.md). |
| **Retourne** | Objet [ResourceResponse](bot-framework-rest-connector-api-reference.md#resourceresponse-object) qui contient une propriété `id` spécifiant l’ID de l’activité envoyée au bot. | 

> [!NOTE]
> Les fichiers chargés sont supprimés au bout de 24 heures.

## <a name="schema"></a>Schéma

Le schéma Direct Line 3.0 inclut tous les objets définis par le [schéma Bot Framework v3](bot-framework-rest-connector-api-reference.md#objects) ainsi que l’objet `ActivitySet` et l’objet `Conversation`.

### <a name="activityset-object"></a>Objet ActivitySet 
Définit un ensemble d’activités.<br/><br/>

| Propriété | type | Description |
|----|----|----|
| **activités** | [Activité](bot-framework-rest-connector-api-reference.md#activity-object)[] | Tableau d’objets **Activity**. |
| **watermark** | chaîne | Filigrane maximal des activités au sein de l’ensemble. Un client peut utiliser la valeur `watermark` pour indiquer le message le plus récent qu’il a vu, soit lors de la [récupération d’activités à partir du bot](bot-framework-rest-direct-line-3-0-receive-activities.md#http-get), soit lors de la [génération d’une nouvelle URL de flux de données WebSocket](bot-framework-rest-direct-line-3-0-reconnect-to-conversation.md). |

### <a name="conversation-object"></a>Objet Conversation
Définit une conversation Direct Line.<br/><br/>

| Propriété | type | Description |
|----|----|----|
| **conversationId** | chaîne | ID identifiant de manière unique la conversation pour laquelle le jeton spécifié est valide. |
| **expires_in** | number | Nombre de secondes avant l’expiration du jeton. |
| **streamUrl** | chaîne | URL pour le flux de messages de la conversation. |
| **token** | chaîne | Jeton valide pour la conversation spécifiée. |

### <a name="activities"></a>Activités

Pour chaque [Activité](bot-framework-rest-connector-api-reference.md#activity-object) qu’un client reçoit d’un bot via Direct Line :

- Les cartes de pièces jointes sont conservées.
- Les URL pour les pièces jointes chargées sont masquées avec un lien privé.
- La propriété `channelData` est conservée sans modification. 

Les clients peuvent [recevoir](bot-framework-rest-direct-line-3-0-receive-activities.md) plusieurs activités à partir du bot en tant que partie d’un [ActivitySet](#activityset-object). 

Lorsqu’un client envoie une [activité](bot-framework-rest-connector-api-reference.md#activity-object) à un bot via Direct Line :

- La propriété `type` spécifie l’activité de type envoyé (généralement **message**).
- La propriété `from` doit être remplie avec un ID d’utilisateur choisi par le client.
- Les pièces jointes peuvent contenir des URL de ressources existantes, ou des URL chargés via le point de terminaison de pièce jointe de Direct Line.
- La propriété `channelData` est conservée sans modification.

Un client peut [envoyer](bot-framework-rest-direct-line-3-0-send-activity.md) une seule activité par requête. 
