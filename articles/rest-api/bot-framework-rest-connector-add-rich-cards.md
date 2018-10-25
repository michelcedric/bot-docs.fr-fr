---
title: Ajouter des pièces jointes de cartes enrichies aux messages | Microsoft Docs
description: Découvrez comment ajouter des cartes enrichies aux messages à l’aide du service Bot Connector.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
ms.openlocfilehash: e38bb7ca93c5fc4174d67d1c5ebb0655eef68653
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2018
ms.locfileid: "49997910"
---
# <a name="add-rich-card-attachments-to-messages"></a>Ajouter des pièces jointes de cartes enrichies aux messages
> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-add-rich-card-attachments.md)
> - [Node.JS](../nodejs/bot-builder-nodejs-send-rich-cards.md)
> - [REST](../rest-api/bot-framework-rest-connector-add-rich-cards.md)

Les robots et les canaux échangent généralement des chaînes de texte, mais certains canaux prennent également en charge l’échange de pièces jointes, ce qui permet à votre robot d’envoyer des messages enrichis aux utilisateurs. Par exemple, votre bot peut envoyer des cartes enrichies et des pièces jointes multimédias (comme des images, des vidéos, des données audio ou des fichiers). Cet article vous expliquer comment ajouter des pièces jointes de cartes enrichies aux messages à l’aide du service Bot Connector.

> [!NOTE]
> Pour plus d’informations sur l’ajout de pièces jointes multimédias aux messages, consultez l’article [Ajouter des pièces jointes multimédia aux messages](bot-framework-rest-connector-add-media-attachments.md).

## <a id="types-of-cards"></a> Types de cartes enrichies

Une carte enrichie comporte un titre, une description, un lien et des images. Un message peut contenir plusieurs cartes enrichies, affichées au format liste ou au format carrousel.
Bot Framework prend actuellement en charge huit types de cartes enrichies : 

| Type de carte | Description |
|----|----|
| <a href="/adaptive-cards/get-started/bots">AdaptiveCard</a> | Carte personnalisable pouvant inclure n’importe quelle combinaison de texte, données vocales, images, boutons et champs d’entrée. Consultez l’article sur la [prise en charge de ces cartes par canal](/adaptive-cards/get-started/bots#channel-status).  |
| [AnimationCard][animationCard] | Carte pouvant lire des images GIF animées ou de courtes vidéos. |
| [AudioCard][audioCard] | Carte pouvant lire un fichier audio. |
| [HeroCard][heroCard] | Carte contenant généralement une grande image, un ou plusieurs boutons et du texte. |
| [ThumbnailCard][thumbnailCard] | Carte contenant généralement une image miniature, un ou plusieurs boutons et du texte. |
| [ReceiptCard][receiptCard] | Carte permettant à un bot de fournir un reçu à l’utilisateur. Elle contient généralement la liste des articles à inclure sur le reçu, la taxe et le total, ainsi que du texte. |
| [SignInCard][signinCard] | Carte permettant à un bot de demander à un utilisateur de se connecter. Elle contient généralement du texte et un ou plusieurs boutons sur lesquels l’utilisateur peut cliquer pour lancer le processus de connexion. |
| [VideoCard][videoCard] | Carte pouvant lire des vidéos. |

> [!TIP]
> Pour déterminer le type de cartes enrichies prises en charge par un canal et voir comment le canal affiche les cartes enrichies, consultez l’article concernant [l’inspecteur de canaux][ChannelInspector]. Pour plus d’informations sur les limites applicables au contenu des cartes (par exemple, nombre maximal de boutons ou longueur maximale du titre), consultez la documentation du canal.

## <a name="process-events-within-rich-cards"></a>Traiter des événements dans les cartes enrichies

Pour traiter les événements dans les cartes enrichies, utilisez les objets [CardAction][CardAction] afin de spécifier ce qui doit se produire quand l’utilisateur clique sur un bouton ou appuie sur une section de la carte. Chaque objet [CardAction][CardAction] contient les propriétés suivantes :

| Propriété | type | Description | 
|----|----|----|
| Type | chaîne | type d’action (une des valeurs indiquées dans le tableau ci-dessous) |
| title | chaîne | titre du bouton |
| image | chaîne | URL d’image du bouton |
| value | chaîne | valeur nécessaire pour effectuer le type d’action spécifié |

> [!NOTE]
> Les boutons dans les cartes adaptatives ne sont pas créés avec les objets `CardAction`, mais à l’aide du schéma défini par les <a href="http://adaptivecards.io" target="_blank">cartes adaptatives</a>. Pour obtenir un exemple illustrant comment ajouter des boutons à une carte adaptative, consultez la section [Ajouter une carte adaptative à un message](#adaptive-card).

Ce tableau répertorie les valeurs valides pour la propriété `type` d’un objet [CardAction][CardAction] et décrit le contenu attendu de la propriété `value` pour chaque type :

| Type | value | 
|----|----|
| openUrl | URL à ouvrir dans le navigateur intégré |
| imBack | Texte du message à envoyer au bot (de la part de l’utilisateur qui a cliqué sur le bouton ou appuyé sur la carte). Ce message (de l’utilisateur au bot) sera visible par tous les participants à la conversation par le biais de l’application cliente qui héberge la conversation. |
| postBack | Texte du message à envoyer au bot (de la part de l’utilisateur qui a cliqué sur le bouton ou appuyé sur la carte). Certaines applications clientes peuvent afficher ce texte dans le flux de messages, où il sera visible par tous les participants à la conversation. |
| appel | Destination d’un appel téléphonique au format suivant : **tel:123123123123** |
| playAudio | URL du fichier audio à lire. |
| playVideo | URL du fichier vidéo à lire. |
| showImage | URL de l’image à afficher. |
| downloadFile | URL du fichier à télécharger. |
| signin | URL du flux OAuth à démarrer. |

## <a name="add-a-hero-card-to-a-message"></a>Ajouter une carte de bannière à un message

Pour ajouter une pièce jointe de carte enrichie à un message, commencez par créer un objet qui correspond au [type de carte](#types-of-cards) que vous souhaitez ajouter au message. Puis créez un objet [Attachment][Attachment], et définissez sa propriété `contentType` sur le type multimédia de la carte et sa propriété `content` sur l’objet que vous avez créé pour représenter la carte. Spécifiez votre objet [Attachment][Attachment] dans le tableau `attachments` du message.

> [!TIP]
> Les messages qui contiennent des pièces jointes de cartes enrichies ne spécifient généralement pas d’éléments `text`.

Certains canaux vous permettent d’ajouter plusieurs cartes enrichies au tableau `attachments` dans un message. Cette fonctionnalité peut être utile dans les cas où vous souhaitez offrir plusieurs options aux utilisateurs. Par exemple, si votre bot permet aux utilisateurs de réserver des chambres d’hôtel, il peut leur présenter une liste de cartes enrichies affichant les types de chambres disponibles. Chacune de ces cartes peut contenir une photo, accompagnée de la liste des équipements correspondant au type de la chambre ; l’utilisateur peut alors sélectionner un type de chambre en appuyant sur une carte ou en cliquant sur un bouton.

> [!TIP]
> Pour afficher plusieurs cartes enrichies au format liste, définissez la propriété `attachmentLayout` de l’objet [Activity][Activity] sur « list ». Pour afficher plusieurs cartes enrichies au format carrousel, définissez la propriété `attachmentLayout` de l’objet [Activity][Activity] sur « carousel ». Si le canal ne prend pas en charge le format carrousel, les cartes enrichies s’afficheront au format liste, même si la propriété `attachmentLayout` indique « carousel ».

L’exemple ci-après présente une requête qui envoie un message ne contenant qu’une seule pièce jointe de carte de bannière. Dans cet exemple de requête, `https://smba.trafficmanager.net/apis` représente l’URI de base, qui peut être différent de celui des requêtes émises par votre bot. Pour plus d’informations sur la définition de l’URI de base, consultez l’article [Informations de référence sur l’API](bot-framework-rest-connector-api-reference.md#base-uri).

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
    "attachments": [
        {
            "contentType": "application/vnd.microsoft.card.hero",
            "content": {
                "title": "title goes here",
                "subtitle": "subtitle goes here",
                "text": "descriptive text goes here",
                "images": [
                    {
                        "url": "http://aka.ms/Fo983c",
                        "alt": "picture of a duck",
                        "tap": {
                            "type": "playAudio",
                            "value": "url to an audio track of a duck call goes here"
                        }
                    }
                ],
                "buttons": [
                    {
                        "type": "playAudio",
                        "title": "Duck Call",
                        "value": "url to an audio track of a duck call goes here"
                    },
                    {
                        "type": "openUrl",
                        "title": "Watch Video",
                        "image": "http://aka.ms/Fo983c",
                        "value": "url goes here of the duck in flight"
                    }
                ]
            }
        }
    ],
    "replyToId": "5d5cdc723"
}
```

## <a id="adaptive-card"></a> Ajouter une carte adaptative à un message

La carte adaptive peut inclure n’importe quelle combinaison de texte, données vocales, images, boutons et champs d’entrée. Les cartes adaptatives sont créées à l’aide du format JSON spécifié sur le site <a href="http://adaptivecards.io" target="_blank">Adaptive Cards</a> (Cartes adaptatives), ce qui vous donne un contrôle total sur le contenu et le format de la carte. 

Tirez profit des informations fournies par le site <a href="http://adaptivecards.io" target="_blank">Adaptive Cards</a> (Cartes adaptatives) pour comprendre le schéma de carte adaptative, explorer les éléments de carte adaptative et découvrir des exemples JSON qui permettent de créer des cartes présentant différents types de compositions et niveaux de complexité. En outre, vous pouvez utiliser le visualiseur interactif pour concevoir des charges utiles de carte adaptative et afficher un aperçu de la sortie des cartes.

L’exemple ci-après présente une requête qui envoie un message contenant une carte adaptative pour un rappel du Calendrier. Dans cet exemple de requête, `https://smba.trafficmanager.net/apis` représente l’URI de base, qui peut être différent de celui des requêtes émises par votre bot. Pour plus d’informations sur la définition de l’URI de base, consultez l’article [Informations de référence sur l’API](bot-framework-rest-connector-api-reference.md#base-uri).

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
    "attachments": [
        {
            "contentType": "application/vnd.microsoft.card.adaptive",
            "content": {
                "type": "AdaptiveCard",
                "body": [
                    {
                        "type": "TextBlock",
                        "text": "Adaptive Card design session",
                        "size": "large",
                        "weight": "bolder"
                    },
                    {
                        "type": "TextBlock",
                        "text": "Conf Room 112/3377 (10)"
                    },
                    {
                        "type": "TextBlock",
                        "text": "12:30 PM - 1:30 PM"
                    },
                    {
                        "type": "TextBlock",
                        "text": "Snooze for"
                    },
                    {
                        "type": "Input.ChoiceSet",
                        "id": "snooze",
                        "style": "compact",
                        "choices": [
                            {
                                "title": "5 minutes",
                                "value": "5",
                                "isSelected": true
                            },
                            {
                                "title": "15 minutes",
                                "value": "15"
                            },
                            {
                                "title": "30 minutes",
                                "value": "30"
                            }
                        ]
                    }
                ],
                "actions": [
                    {
                        "type": "Action.Http",
                        "method": "POST",
                        "url": "http://foo.com",
                        "title": "Snooze"
                    },
                    {
                        "type": "Action.Http",
                        "method": "POST",
                        "url": "http://foo.com",
                        "title": "I'll be late"
                    },
                    {
                        "type": "Action.Http",
                        "method": "POST",
                        "url": "http://foo.com",
                        "title": "Dismiss"
                    }
                ]
            }
        }
    ],
    "replyToId": "5d5cdc723"
}
```

La carte résultante contient trois blocs de texte, un champ d’entrée (liste de choix) et trois boutons :

![Carte adaptative de rappel du Calendrier](../media/adaptive-card-reminder.png)


## <a name="additional-resources"></a>Ressources supplémentaires

- [Créer des messages](bot-framework-rest-connector-create-messages.md)
- [Envoyer et recevoir des messages](bot-framework-rest-connector-send-and-receive-messages.md)
- [Ajouter des pièces jointes multimédia aux messages](bot-framework-rest-connector-add-media-attachments.md)
- [Inspecteur de canaux][ChannelInspector]
- <a href="http://adaptivecards.io" target="_blank">Cartes adaptatives</a>

[ChannelInspector]: ../bot-service-channel-inspector.md

[animationCard]: bot-framework-rest-connector-api-reference.md#animationcard-object
[audioCard]: bot-framework-rest-connector-api-reference.md#audiocard-object
[heroCard]: bot-framework-rest-connector-api-reference.md#herocard-object
[thumbnailCard]: bot-framework-rest-connector-api-reference.md#thumbnailcard-object
[receiptCard]: bot-framework-rest-connector-api-reference.md#receiptcard-object
[signinCard]: bot-framework-rest-connector-api-reference.md#signincard-object
[videoCard]: bot-framework-rest-connector-api-reference.md#videocard-object

[CardAction]: bot-framework-rest-connector-api-reference.md#cardaction-object
[Activity]: bot-framework-rest-connector-api-reference.md#activity-object
[Attachment]: bot-framework-rest-connector-api-reference.md#attachment-object