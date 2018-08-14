---
title: Ajouter des pièces jointes multimédias aux messages | Microsoft Docs
description: Découvrez comment ajouter des pièces jointes multimédias aux messages à l’aide du service Bot Connector.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: 39247ec3be4da7129989041269e930de8fa766ae
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39299409"
---
# <a name="add-media-attachments-to-messages"></a>Ajouter des pièces jointes multimédia aux messages
> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-add-media-attachments.md)
> - [Node.JS](../nodejs/bot-builder-nodejs-send-receive-attachments.md)
> - [REST](../rest-api/bot-framework-rest-connector-add-media-attachments.md)

Les robots et les canaux échangent généralement des chaînes de texte, mais certains canaux prennent également en charge l’échange de pièces jointes, ce qui permet à votre robot d’envoyer des messages enrichis aux utilisateurs. Par exemple, votre robot peut envoyer des [cartes riches](bot-framework-rest-connector-add-rich-cards.md) et des pièces jointes multimédias (comme des images, des vidéos, des données audio ou des fichiers). Cet article décrit comment ajouter des pièces jointes multimédia à des messages à l’aide du service Bot Connector.

> [!TIP]
> Pour déterminer le type et le nombre de pièces jointes qu’un canal prend en charge, et la manière dont le canal effectue le rendu des pièces jointes, voir [Inspecteur de canaux][ChannelInspector].

## <a name="add-a-media-attachment"></a>Ajouter une pièce jointe multimédia  

Pour ajouter une pièce jointe multimédia à un message, créez un objet [Pièce jointe][Attachment], définissez la propriété `name`, définissez la propriété `contentUrl` sur URL du fichier multimédia, puis définissez la propriété `contentType` sur le type de média approprié (par exemple, **image/png**, **audio/wav**, **vidéo/mp4**). Ensuite, dans l’objet [Activité][Activity] qui représente votre message, spécifiez votre objet [Pièce jointe][Attachment] dans le tableau `attachments`. 

L’exemple ci-après présente une demande qui envoie un message contenant du texte et une image en pièce jointe. Dans cet exemple de demande, `https://smba.trafficmanager.net/apis` représente l’URI de base. L’URI de base pour les demandes émises par votre robot peut être différente. Pour plus d’informations sur la définition de l’URI de base, voir [Informations de référence sur l’API](bot-framework-rest-connector-api-reference.md#base-uri).

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
    "text": "Here's a picture of the duck I was telling you about.",
    "attachments": [
        {
            "contentType": "image/png",
            "contentUrl": "http://aka.ms/Fo983c",
            "name": "duck-on-a-rock.jpg"
        }
    ],
    "replyToId": "5d5cdc723"
}
```

Pour les canaux qui prennent en charge les fichiers binaires d’image en ligne, vous pouvez définir la propriété `contentUrl` de la `Attachment` sur un fichier binaire en base 64 de l’image (par exemple, **data:image/png;base64,iVBORw0KGgo…**). Le canal affiche l’image ou l’URL de l’image en regard de la chaîne de texte du message.

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
    "text": "Here's a picture of the duck I was telling you about.",
    "attachments": [
        {
            "contentType": "image/png",
            "contentUrl": "data:image/png;base64,iVBORw0KGgo…",
            "name": "duck-on-a-rock.jpg"
        }
    ],
    "replyToId": "5d5cdc723"
}
```

Vous pouvez joindre un fichier vidéo ou audio à un message en procédant comme décrit ci-dessus pour un fichier image. Selon le canal, la vidéo et l’audio peuvent être lus en ligne ou affichés sous forme de liens.

> [!NOTE] 
> Votre robot peut également recevoir des messages contenant des pièces jointes multimédia.
> Par exemple, un message que votre robot reçoit peut contenir une pièce jointe si le canal permet à l’utilisateur de charger une photo à analyser ou un document à stocker.

## <a name="add-an-audiocard-attachment"></a>Ajouter une pièce jointe AudioCard

L’ajout d’une pièce jointe [AudioCard](bot-framework-rest-connector-api-reference.md#audiocard-object) ou [VideoCard](bot-framework-rest-connector-api-reference.md#videocard-object) est similaire à l’ajout d’une pièce jointe multimédia. Par exemple, le code JSON suivant montre comment ajouter une carte audio dans la pièce jointe multimédia.

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
    "attachments": [
    {
      "contentType": "application/vnd.microsoft.card.audio",
      "content": {
        "title": "Allegro in C Major",
        "subtitle": "Allegro Duet",
        "text": "No Image, No Buttons, Autoloop, Autostart, Sharable",
        "media": [
          {
            "url": "https://contoso.com/media/AllegrofromDuetinCMajor.mp3"
          }
        ],
        "shareable": true,
        "autoloop": true,
        "autostart": true,
        "value": {
            // Supplementary parameter for this card
        }
      }
    }],
    "replyToId": "5d5cdc723"
}
```

Lorsque le canal reçoit cette pièce jointe, il commence à lire le fichier audio. Si un utilisateur interagit avec l’audio en cliquant sur le bouton **Pause**, par exemple, le canal envoie un rappel au robot avec un JSON ressemblant à ceci :

```json
{
    ...
    "type": "event",
    "name": "media/pause",
    "value": {
        "url": // URL for media
        "cardValue": {
            // Supplementary parameter for this card
        }
    }
}
```

Le nom de l’événement multimédia **média/pause** s’affiche dans le champ `activity.name`. Référencez le tableau ci-dessous pour obtenir la liste de tous les noms d’événements multimédia.

| Événement | Description |
| ---- | ---- |
| **média/suivant** | Le client est passé au média suivant |
| **média/pause** | Le client a mis en pause la lecture du média |
| **média/lecture** | Le client a commencé la lecture du média |
| **média/précédent** | Le client est passé au média précédent |
| **média/reprise** | Le client a repris la lecture du média |
| **média/arrêt** | Le client a arrêté la lecture du média |

## <a name="additional-resources"></a>Ressources supplémentaires

- [Créer des messages](bot-framework-rest-connector-create-messages.md)
- [Envoyer et recevoir des messages](bot-framework-rest-connector-send-and-receive-messages.md)
- [Ajouter des cartes riches aux messages](bot-framework-rest-connector-add-rich-cards.md)
- [Inspecteur de canaux][ChannelInspector]

[ChannelInspector]: ../bot-service-channel-inspector.md

[Activity]: bot-framework-rest-connector-api-reference.md#activity-object
[Attachment]: bot-framework-rest-connector-api-reference.md#attachment-object
