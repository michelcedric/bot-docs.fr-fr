---
title: Envoyer une activité au robot | Microsoft Docs
description: Découvrez comment envoyer une activité au robot à l’aide de l’API Direct Line v3.0.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
ms.openlocfilehash: 290a2733b96a458eb3529b0b0854703631e05f22
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2018
ms.locfileid: "50000036"
---
# <a name="send-an-activity-to-the-bot"></a>Envoyer une activité au bot

Le protocole Direct Line 3.0 permet aux clients et aux robots d’échanger différents types d’[activités](bot-framework-rest-connector-activities.md), notamment des activités de **messagerie** et de **saisie**, ainsi que des activités personnalisées prises en charge par le robot. Un client ne peut envoyer qu’une seule activité par requête. 

## <a name="send-an-activity"></a>Envoyer une activité

Pour envoyer une activité au robot, le client doit créer un objet [Activité](bot-framework-rest-connector-api-reference.md#activity-object) pour définir l’activité, puis envoyer une requête `POST` à `https://directline.botframework.com/v3/directline/conversations/{conversationId}/activities` en précisant l’objet de l’activité dans le corps de la requête.

Les extraits de code suivants montrent la requête et la réponse de l’activité d’envoi.

### <a name="request"></a>Requête

```http
POST https://directline.botframework.com/v3/directline/conversations/abc123/activities
Authorization: Bearer RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0
Content-Type: application/json
[other headers]
```

```json
{
    "type": "message",
    "from": {
        "id": "user1"
    },
    "text": "hello"
}
```

### <a name="response"></a>response

Lorsque l’activité est fournie au robot, le service répond avec un code d’état HTTP qui correspond au code d’état du robot. Si le robot génère une erreur, une réponse HTTP 502 (« passerelle incorrecte ») est renvoyée au client en réponse à sa requête d’activité d’envoi. Si la PUBLICATION est réussie, la réponse contient une charge utile JSON qui spécifie l’identifiant de I’activité envoyée au robot.

```http
HTTP/1.1 200 OK
[other headers]
```

```json
{
    "id": "0001"
}
```

### <a name="total-time-for-the-send-activity-requestresponse"></a>Durée totale de requête/réponse de l’activité d’envoi

La durée totale de PUBLICATION d’un message à une conversation Direct Line est égale à la somme des éléments suivants :

- Délai de transmission de la requête HTTP du client au service Direct Line
- Temps de traitement interne dans Direct Line (généralement inférieur à 120 ms)
- Délai de transmission entre le service Direct Line et le robot
- Temps de traitement au sein du robot
- Délai de transmission de la réponse HTTP au client

## <a name="send-attachments-to-the-bot"></a>Envoyer des pièces jointes au robot

Dans certains cas, un client peut avoir besoin d’envoyer des pièces jointes au robot, par exemple des images ou des documents. Un client peut envoyer des pièces jointes au robot en [ indiquant la ou les URL ](#send-by-url) de la ou des pièces jointes dans l’objet [Activité](bot-framework-rest-connector-api-reference.md#activity-object) envoyé à l’aide de `POST /v3/directline/conversations/{conversationId}/activities` ou en [ téléchargeant la ou les pièces jointes ](#upload-attachments) à l’aide de `POST /v3/directline/conversations/{conversationId}/upload`.

## <a id="send-by-url"></a>Envoyer une ou des pièces jointes par URL

Pour envoyer une ou plusieurs pièces jointes faisant partie de l’objet [Activité](bot-framework-rest-connector-api-reference.md#activity-object) à l’aide de `POST /v3/directline/conversations/{conversationId}/activities`, il suffit d’inclure un ou plusieurs objets [Pièce jointe](bot-framework-rest-connector-api-reference.md#attachment-object) dans l’objet Activité et de définir la propriété `contentUrl` de chaque objet Pièce jointe pour spécifier l’URI HTTP, HTTPS ou `data`. URI de la pièce jointe.

## <a id="upload-attachments"></a>Envoyer une ou des pièces jointes par téléchargement

Il arrive souvent qu’un client ait des images ou des documents sur un périphérique qu’il veut envoyer au bot, mais ne dispose pas des URL correspondant aux fichiers. Dans cette situation, un client peut lancer une requête `POST /v3/directline/conversations/{conversationId}/upload` afin d’envoyer des pièces jointes au robot par téléchargement. Le format et le contenu de la demande varient selon que le client [envoie une pièce jointe unique](#upload-one-attachment) ou [envoie plusieurs pièces jointes](#upload-multiple-attachments).

### <a id="upload-one-attachment"></a>Envoyer une seule pièce jointe par téléchargement

Pour envoyer une seule pièce jointe par chargement, exécutez la demande suivante : 

```http
POST https://directline.botframework.com/v3/directline/conversations/{conversationId}/upload?userId={userId}
Authorization: Bearer SECRET_OR_TOKEN
Content-Type: TYPE_OF_ATTACHMENT
Content-Disposition: ATTACHMENT_INFO
[other headers]

[file content]
```

Dans cette requête URI, remplacez **{conversationId}** par l’identifiant de la conversation et **{userId}** par l’identifiant de l’utilisateur qui envoie le message. Le paramètre `userId` est obligatoire. Dans les en-têtes de requête, définissez `Content-Type` pour indiquer le type de pièce jointe et définissez `Content-Disposition` pour indiquer le nom de fichier de la pièce jointe.

Les extraits de code suivants montrent la requête et la réponse de l’envoi d’une seule pièce jointe.

#### <a name="request"></a>Requête

```http
POST https://directline.botframework.com/v3/directline/conversations/abc123/upload?userId=user1
Authorization: Bearer RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0
Content-Type: image/jpeg
Content-Disposition: name="file"; filename="badjokeeel.jpg"
[other headers]

[JPEG content]
```

#### <a name="response"></a>response

Si la requête est acceptée, un **message** d’activité est envoyé au robot à la fin du téléchargement et la réponse reçue par le client contient l’identifiant de l’activité envoyée.

```http
HTTP/1.1 200 OK
[other headers]
```

```json
{
  "id": "0003"
}
```

### <a id="upload-multiple-attachments"></a> Envoyer plusieurs pièces jointes par téléchargement

Pour envoyer plusieurs pièces jointes par chargement, publiez (`POST`) une demande en plusieurs parties sur le point de terminaison `/v3/directline/conversations/{conversationId}/upload`. Définissez l’en-tête `Content-Type` de la requête sur `multipart/form-data` et insérez les en-têtes `Content-Type` et `Content-Disposition` à chaque partie pour indiquer le type et le nom de fichier de chaque pièce jointe. Dans la requête URI, définissez le paramètre `userId` à l’identifiant de l’utilisateur qui envoie le message. 

Vous pouvez inclure un objet [Activité](bot-framework-rest-connector-api-reference.md#activity-object) dans la requête en ajoutant une partie qui indique la valeur `Content-Type` de l’en-tête `application/vnd.microsoft.activity`. Si la requête comprend une activité, les pièces jointes indiquées par d’autres parties de la charge utile sont ajoutées comme pièces jointes à cette activité avant qu’elle ne soit envoyée. Si la requête n’inclut pas d’activité, une activité vide est créée pour servir de conteneur dans lequel les pièces jointes indiquées sont envoyées.

Les extraits de code suivants montrent la requête et la réponse de l’envoi de plusieurs pièces jointes. Dans cet exemple, la demande envoie un message contenant du texte et une image unique en pièce jointe. La requête peut comporter des parties supplémentaires afin d’inclure plusieurs pièces jointes dans le message.

#### <a name="request"></a>Requête

```http
POST https://directline.botframework.com/v3/directline/conversations/abc123/upload?userId=user1
Authorization: Bearer RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0
Content-Type: multipart/form-data; boundary=----DD4E5147-E865-4652-B662-F223701A8A89
[other headers]

----DD4E5147-E865-4652-B662-F223701A8A89
Content-Type: image/jpeg
Content-Disposition: form-data; name="file"; filename="badjokeeel.jpg"
[other headers]

[JPEG content]

----DD4E5147-E865-4652-B662-F223701A8A89
Content-Type: application/vnd.microsoft.activity
[other headers]

{
  "type": "message",
  "from": {
    "id": "user1"
  },
  "text": "Hey I just IM'd you\n\nand this is crazy\n\nbut here's my webhook\n\nso POST me maybe"
}

----DD4E5147-E865-4652-B662-F223701A8A89
```

#### <a name="response"></a>response

Si la requête est acceptée, un message d’activité est envoyé au robot à la fin du téléchargement et la réponse reçue par le client contient l’identifiant de l’activité envoyée.

```http
HTTP/1.1 200 OK
[other headers]
```

```json
{
    "id": "0004"
}
```

## <a name="additional-resources"></a>Ressources supplémentaires

- [Concepts clés](bot-framework-rest-direct-line-3-0-concepts.md)
- [Authentification](bot-framework-rest-direct-line-3-0-authentication.md)
- [Démarrer une conversation](bot-framework-rest-direct-line-3-0-start-conversation.md)
- [Se reconnecter à une conversation](bot-framework-rest-direct-line-3-0-reconnect-to-conversation.md)
- [Recevoir des activités du bot](bot-framework-rest-direct-line-3-0-receive-activities.md)
- [Mettre fin à une conversation](bot-framework-rest-direct-line-3-0-end-conversation.md)
