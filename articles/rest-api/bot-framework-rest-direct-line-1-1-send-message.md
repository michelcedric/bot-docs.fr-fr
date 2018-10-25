---
title: Envoyer un message au robot | Microsoft Docs
description: Découvrez comment envoyer un message au robot à l’aide de l’API Direct Line v1.1.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
ms.openlocfilehash: 360ec3a6a6c9a3be16370aaf445f24a237a702e3
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2018
ms.locfileid: "49998016"
---
# <a name="send-a-message-to-the-bot"></a>Envoyer un message au robot

> [!IMPORTANT]
> Cet article décrit comment envoyer un message au robot à l’aide de l’API Direct Line 1.1. Si vous créez une connexion entre votre application cliente et votre robot, utilisez plutôt l’[API Direct Line 3.0](bot-framework-rest-direct-line-3-0-send-activity.md).

Le protocole Direct Line 1.1 permet aux clients d’échanger des messages avec des robots. Ces messages sont convertis dans le schéma que le robot prend en charge (Bot Framework v1 ou Bot Framework v3). Un client peut envoyer un seul message par demande. 

## <a name="send-a-message"></a>Envoyer un message

Pour envoyer un message au bot, le client doit créer un objet [Message](bot-framework-rest-direct-line-1-1-api-reference.md#message-object) pour définir le message, puis envoyer une demande `POST` à `https://directline.botframework.com/api/conversations/{conversationId}/messages`, en spécifiant l’objet Message dans le corps de la demande.

Les extraits de code suivants illustrent la demande et la réponse Envoyer un message.

### <a name="request"></a>Requête

```http
POST https://directline.botframework.com/api/conversations/abc123/messages
Authorization: Bearer RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0
[other headers]
```

```json
{
  "text": "hello",
  "from": "user1"
}
```

### <a name="response"></a>response

Lorsque le message est remis au robot, le service répond avec un code d’état HTTP qui reflète le code d’état du robot. Si le robot génère une erreur, une réponse HTTP 500 (« Erreur interne du serveur ») est retournée au client en réponse à sa demande Envoyer un Message. Si l’opération POST réussit, le service retourne un code d’état HTTP 204. Aucune donnée n’est retournée dans le corps de la réponse. Le message du client et tous les messages du robot peuvent être obtenus via une [interrogation](bot-framework-rest-direct-line-1-1-receive-messages.md). 

```http
HTTP/1.1 204 No Content
[other headers]
```

### <a name="total-time-for-the-send-message-requestresponse"></a>Durée totale de la demande/réponse Envoyer un message

La durée totale pour publier (POSt) un message dans une conversation Direct Line est la somme des valeurs suivantes :

- Délai de transmission de la requête HTTP du client au service Direct Line
- Temps de traitement interne dans Direct Line (généralement inférieur à 120 ms)
- Délai de transmission entre le service Direct Line et le robot
- Temps de traitement au sein du robot
- Délai de transmission de la réponse HTTP au client

## <a name="send-attachments-to-the-bot"></a>Envoyer des pièces jointes au robot

Dans certaines situations, un client peut avoir besoin d’envoyer au robot des pièces jointes telles que des images ou des documents. Un client peut envoyer des pièces jointes au robot soit en [spécifiant les URL](#send-by-url) des pièces jointes dans l’objet [Message](bot-framework-rest-direct-line-1-1-api-reference.md#message-object) qu’il envoie en utilisant `POST /api/conversations/{conversationId}/messages`, ou en [chargeant des pièces jointes](#upload-attachments) en utilisant `POST /api/conversations/{conversationId}/upload`.

## <a id="send-by-url"></a> Envoyer des pièces jointes par URL

Pour envoyer une ou plusieurs pièces jointes dans un objet [Message](bot-framework-rest-direct-line-1-1-api-reference.md#message-object) en utilisant `POST /api/conversations/{conversationId}/messages`, spécifiez l’URL de la pièce jointe dans le tableau `images` du message et/ou dans le tableau `attachments`.

## <a id="upload-attachments"></a> Envoyer des pièces jointes par chargement

Il arrive souvent qu’un client ait des images ou des documents sur un périphérique qu’il veut envoyer au bot, mais ne dispose pas des URL correspondant aux fichiers. Dans cette situation, un client peut lancer une requête `POST /api/conversations/{conversationId}/upload` afin d’envoyer des pièces jointes au robot par téléchargement. Le format et le contenu de la demande varient selon que le client [envoie une pièce jointe unique](#upload-one-attachment) ou [envoie plusieurs pièces jointes](#upload-multiple-attachments).

### <a id="upload-one-attachment"></a>Envoyer une seule pièce jointe par téléchargement

Pour envoyer une seule pièce jointe par chargement, exécutez la demande suivante : 

```http
POST https://directline.botframework.com/api/conversations/{conversationId}/upload?userId={userId}
Authorization: Bearer SECRET_OR_TOKEN
Content-Type: TYPE_OF_ATTACHMENT
Content-Disposition: ATTACHMENT_INFO
[other headers]

[file content]
```

Dans l’URI de cette demande, remplacez **{conversationId}** par l’ID de la conversation et **{userId}** avec l’ID de l’utilisateur qui envoie le message. Dans les en-têtes de demande, définissez `Content-Type` pour spécifier le type de la pièce jointe, et définissez `Content-Disposition` pour spécifier le nom de fichier de la pièce jointe.

Les extraits de code suivants montrent la requête et la réponse de l’envoi d’une seule pièce jointe.

#### <a name="request"></a>Requête

```http
POST https://directline.botframework.com/api/conversations/abc123/upload?userId=user1
Authorization: Bearer RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0
Content-Type: image/jpeg
Content-Disposition: name="file"; filename="badjokeeel.jpg"
[other headers]

[JPEG content]
```

#### <a name="response"></a>response

Si la demande réussit, un message est envoyé au robot lorsque le chargement est terminé et que le service revient à un code d’état HTTP 204.

```http
HTTP/1.1 204 No Content
[other headers]
```

### <a id="upload-multiple-attachments"></a> Envoyer plusieurs pièces jointes par chargement

Pour envoyer plusieurs pièces jointes par chargement, publiez (`POST`) une demande en plusieurs parties sur le point de terminaison `/api/conversations/{conversationId}/upload`. Définissez l’en-tête `Content-Type` de la requête sur `multipart/form-data` et insérez les en-têtes `Content-Type` et `Content-Disposition` à chaque partie pour indiquer le type et le nom de fichier de chaque pièce jointe. Dans l’URI de la demande, définissez le paramètre `userId` sur l’ID de l’utilisateur qui envoie le message. 

Vous pouvez inclure un objet [Message](bot-framework-rest-direct-line-1-1-api-reference.md#message-object) dans la demande en ajoutant une partie spécifiant la valeur d’en-tête `Content-Type` `application/vnd.microsoft.bot.message`. Cela permet au client de personnaliser le message contenant les pièces jointes. Si la demande inclut un message, les pièces jointes spécifiées par d’autres parties de la charge utile sont ajoutées en tant que pièces jointes à ce Message avant son envoi. 

Les extraits de code suivants fournissent un exemple de demande et réponse Envoyer (plusieurs) pièces jointes. Dans cet exemple, la demande envoie un message contenant du texte et une image unique en pièce jointe. La requête peut comporter des parties supplémentaires afin d’inclure plusieurs pièces jointes dans le message.

#### <a name="request"></a>Requête

```http
POST https://directline.botframework.com/api/conversations/abc123/upload?userId=user1
Authorization: Bearer RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0
Content-Type: multipart/form-data; boundary=----DD4E5147-E865-4652-B662-F223701A8A89
[other headers]

----DD4E5147-E865-4652-B662-F223701A8A89
Content-Type: image/jpeg
Content-Disposition: form-data; name="file"; filename="badjokeeel.jpg"
[other headers]

[JPEG content]

----DD4E5147-E865-4652-B662-F223701A8A89
Content-Type: application/vnd.microsoft.bot.message
[other headers]

{
  "text": "Hey I just IM'd you\n\nand this is crazy\n\nbut here's my webhook\n\nso POST me maybe",
  "from": "user1"
}

----DD4E5147-E865-4652-B662-F223701A8A89
```

#### <a name="response"></a>response

Si la demande réussit, un message est envoyé au robot lorsque le chargement est terminé et que le service revient à un code d’état HTTP 204.

```http
HTTP/1.1 204 No Content
[other headers]
```

## <a name="additional-resources"></a>Ressources supplémentaires

- [Concepts clés](bot-framework-rest-direct-line-1-1-concepts.md)
- [Authentification](bot-framework-rest-direct-line-1-1-authentication.md)
- [Démarrer une conversation](bot-framework-rest-direct-line-1-1-start-conversation.md)
- [Recevoir des messages du robot](bot-framework-rest-direct-line-1-1-receive-messages.md)