---
title: Implémenter des fonctionnalités spécifiques du canal | Microsoft Docs
description: Découvrez comment implémenter des fonctionnalités spécifiques du canal à l’aide de l’API Bot Connector.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
ms.openlocfilehash: d69013c721552483cfd38b204936cb1c7f508f82
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2018
ms.locfileid: "49996896"
---
# <a name="implement-channel-specific-functionality"></a>Implémenter une fonctionnalité spécifique du canal

Certains canaux fournissent des fonctionnalités qui ne peuvent pas être implémentées à l’aide uniquement de [texte de messages et de pièces jointes](bot-framework-rest-connector-create-messages.md). Pour implémenter une fonctionnalité spécifique d’un canal, vous pouvez transmettre des métadonnées natives à un canal dans la propriété `channelData` de l’objet [Activité][Activity]. Par exemple, votre robot peut utiliser la propriété `channelData` pour demander à Telegram d’envoyer un autocollant ou demander à Office 365 d’envoyer un e-mail.

Cet article explique comment utiliser une propriété `channelData` d’activité de message pour implémenter cette fonctionnalité propre au canal :

| Canal | Fonctionnalités |
|----|----|
| Email | Envoyer et recevoir un e-mail qui contient un corps, un objet et des métadonnées d’importance |
| Slack | Envoyer des messages Slack de fidélité |
| Facebook | Envoyer des notifications Facebook en mode natif |
| Telegram | Effectuer des actions spécifiques à Telegram, comme le partage d’un mémo vocal ou d’un autocollant |
| Kik | Envoyer et recevoir des messages Kik natifs | 

> [!NOTE]
> La valeur de la propriété `channelData` d’un objet [Activité][Activity] est un objet JSON. La structure de l’objet JSON varie en fonction du canal et des fonctionnalités implémentées, comme décrit ci-dessous. 

## <a name="create-a-custom-email-message"></a>Créer un e-mail personnalisé

Pour créer un e-mail, définissez la propriété `channelData` de l’objet [Activité][Activity] sur un objet JSON contenant ces propriétés :

[!INCLUDE [Email channelData table](~/includes/snippet-channelData-email.md)]

Cet extrait de code présente un exemple de la propriété `channelData` pour un e-mail personnalisé.

```json
"channelData":
{
    "htmlBody": "<html><body style = /"font-family: Calibri; font-size: 11pt;/" >This is more than awesome.</body></html>",
    "subject": "Super awesome message subject",
    "importance": "high",
    "ccRecipients": "Yasemin@adatum.com;Temel@adventure-works.com"
}
```

## <a name="create-a-full-fidelity-slack-message"></a>Créer un message Slack de fidélité optimale

Pour créer un message Slack de fidélité, définissez la propriété `channelData` de l’objet [activité][Activity] sur un objet JSON qui spécifie <a href="https://api.slack.com/docs/messages" target="_blank">Messages Slack</a>, <a href="https://api.slack.com/docs/message-attachments" target="_blank">Pièces jointes Slack</a> et/ou <a href="https://api.slack.com/docs/message-buttons" target="_blank">Boutons Slack</a>. 

> [!NOTE]
> Pour prendre en charge des boutons dans les messages Slack, vous devez activer **Messages interactifs** lorsque vous [connectez votre robot](../bot-service-manage-channels.md) au canal Slack.

Cet extrait de code montre un exemple de la propriété `channelData` pour un message Slack personnalisé.

```json
"channelData": {
   "text": "Now back in stock! :tada:",
   "attachments": [
        {
            "title": "The Further Adventures of Slackbot",
            "author_name": "Stanford S. Strickland",
            "author_icon": "https://api.slack.com/img/api/homepage_custom_integrations-2x.png",
            "image_url": "http://i.imgur.com/OJkaVOI.jpg?1"
        },
        {
            "fields": [
                {
                    "title": "Volume",
                    "value": "1",
                    "short": true
                },
                {
                    "title": "Issue",
                    "value": "3",
                    "short": true
                }
            ]
        },
        {
            "title": "Synopsis",
            "text": "After @episod pushed exciting changes to a devious new branch back in Issue 1, Slackbot notifies @don about an unexpected deploy..."
        },
        {
            "fallback": "Would you recommend it to customers?",
            "title": "Would you recommend it to customers?",
            "callback_id": "comic_1234_xyz",
            "color": "#3AA3E3",
            "attachment_type": "default",
            "actions": [
                {
                    "name": "recommend",
                    "text": "Recommend",
                    "type": "button",
                    "value": "recommend"
                },
                {
                    "name": "no",
                    "text": "No",
                    "type": "button",
                    "value": "bad"
                }
            ]
        }
    ]
}
```

Lorsqu’un utilisateur clique sur un bouton dans un message Slack, votre bot reçoit un message de réponse dans lequel la propriété `channelData` est remplie par un objet JSON `payload`. L’objet `payload` précise le contenu du message d’origine, il identifie le bouton qui a été cliqué et identifie l’utilisateur qui a cliqué sur le bouton. 

Cet extrait de code montre un exemple de la propriété `channelData` dans le message reçu par un bot lorsqu’un utilisateur clique sur un bouton dans le message Slack.

```json
"channelData": {
    "payload": {
        "actions": [
            {
                "name": "recommend",
                "value": "yes"
            }
        ],
        . . .
        "original_message": "{…}",
        "response_url": "https://hooks.slack.com/actions/..."
    }
}
```

Votre bot peut répondre à ce message [normalement](bot-framework-rest-connector-send-and-receive-messages.md#create-reply), ou il peut publier sa réponse directement sur le point de terminaison spécifié par la propriété `response_url` de l’objet `payload`. Pour plus d’informations sur le moment et la façon de publier une réponse sur `response_url`, consultez <a href="https://api.slack.com/docs/message-buttons" target="_blank">Slack Buttons</a>. 

## <a name="create-a-facebook-notification"></a>Créer une notification de Facebook

Pour créer une notification de Facebook, définissez la propriété `channelData` de l’objet [Activité][Activity] sur un objet JSON spécifiant ces propriétés : 

| Propriété | Description |
|----|----|
| notification_type | Type de notification (par exemple, **REGULAR**, **SILENT_PUSH**, **NO_PUSH**).
| attachment | Pièce jointe qui spécifie une image, une vidéo ou un autre type multimédia, ou encore une pièce jointe basée sur un modèle, comme un accusé de réception. |

> [!NOTE]
> Pour plus d’informations sur le format et le contenu de la propriété `notification_type` et de la propriété `attachment`, consultez la <a href="https://developers.facebook.com/docs/messenger-platform/send-api-reference#guidelines" target="_blank">documentation de l’API Facebook</a>. 

Cet extrait de code montre un exemple de la propriété `channelData` pour une pièce jointe d’accusé de réception de Facebook.

```json
"channelData": {
    "notification_type": "NO_PUSH",
    "attachment": {
        "type": "template"
        "payload": {
            "template_type": "receipt",
            . . .
        }
    }
}
```

## <a name="create-a-telegram-message"></a>Créer un message Telegram

Pour créer un message qui implémente des actions spécifiques de Telegram, telles que le partage d’un mémo vocal ou d’un autocollant, définissez la propriété `channelData` de l’objet [Activité][Activity] sur un objet JSON qui spécifie ces propriétés : 

| Propriété | Description |
|----|----|
| method | Méthode de l’API Bot Telegram à appeler. |
| parameters | Paramètres de la méthode spécifiée. |

Les méthodes Telegram prises en charge sont les suivantes : 

- answerInlineQuery
- editMessageCaption
- editMessageReplyMarkup
- editMessageText
- forwardMessage
- kickChatMember
- SendVideo
- sendChatAction
- sendContact
- sendDocument
- sendLocation
- SendMessage
- sendPhoto
- sendSticker
- sendVenue
- sendVideo
- sendVoice
- unbanChateMember

Pour plus d’informations sur ces méthodes Telegram et leurs paramètres, consultez la <a href="https://core.telegram.org/bots/api#available-methods" target="_blank">documentation de l’API Bot Telegram</a>.

> [!NOTE]
> <ul><li>Le paramètre <code>chat_id</code> est commun à toutes les méthodes Telegram. Si vous ne spécifiez pas <code>chat_id</code> en tant que paramètre, l’infrastructure fournit l’ID pour vous.</li>
> <li>Au lieu de passer le contenu de fichier inline, spécifiez le fichier à l’aide d’une URL et d’un type de média, comme indiqué dans l’exemple ci-dessous.</li>
> <li>Dans chaque message que votre bot reçoit du canal Telegram, la propriété <code>channelData</code> inclut le message précédemment envoyé par votre bot.</li></ul>

Cet extrait de code montre l’exemple d’une propriété `channelData` qui spécifie une méthode Telegram unique.

```json
"channelData": {
    "method": "sendSticker",
    "parameters": {
        "sticker": {
            "url": "https://domain.com/path/gif",
            "mediaType": "image/gif",
        }
    }
}
```

Cet extrait de code montre l’exemple d’une propriété `channelData` qui spécifie un tableau de méthodes Telegram.

```json
"channelData": [
    {
        "method": "sendSticker",
        "parameters": {
            "sticker": {
                "url": "https://domain.com/path/gif",
                "mediaType": "image/gif",
            }
        }
    },
    {
        "method": "sendMessage",
        "parameters": {
            "text": "<b>This message is HTML formatted.</b>",
            "parse_mode": "HTML"
        }
    }
]
```

## <a name="create-a-native-kik-message"></a>Créer un message Kik natif

Pour créer un message Kik natif, définissez la propriété `channelData` de l’objet [Activité][Activity] sur un objet JSON spécifiant cette propriété : 

| Propriété | Description |
|----|----|
| Cloud vers appareil | Un tableau de messages Kik. Pour obtenir des informations détaillées sur le format des messages Kik, consultez <a href="https://dev.kik.com/#/docs/messaging#message-formats" target="_blank">Formats des messages Kik</a>. |

Cet extrait de code montre un exemple de la propriété `channelData` pour un message Kik natif.

```json
"channelData": {
    "messages": [
        {
            "chatId": "c6dd8165…",
            "type": "link",
            "to": "kikhandle",
            "title": "My Webpage",
            "text": "Some text to display",
            "url": "http://botframework.com",
            "picUrl": "http://lorempixel.com/400/200/",
            "attribution": {
                "name": "My App",
                "iconUrl": "http://lorempixel.com/50/50/"
            },
            "noForward": true,
            "kikJsData": {
                    "key": "value"
                }
        }
    ]
}
```

## <a name="additional-resources"></a>Ressources supplémentaires

- [Vue d’ensemble des activités](bot-framework-rest-connector-activities.md)
- [Créer des messages](bot-framework-rest-connector-create-messages.md)
- [Envoyer et recevoir des messages](bot-framework-rest-connector-send-and-receive-messages.md)
- [Aperçu des fonctionnalités avec l’inspecteur de canaux](../bot-service-channel-inspector.md)

[Activity]: bot-framework-rest-connector-api-reference.md#activity-object
