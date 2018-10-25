---
title: Vue d’ensemble des activités | Microsoft Docs
description: Apprenez-en davantage sur les différents types d’activités disponibles dans service Bot Connector.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
ms.openlocfilehash: b246e9e07243e4064f92e72ee3909541f642714e
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2018
ms.locfileid: "49999926"
---
# <a name="activities-overview"></a>Vue d’ensemble des activités

Le service Bot Connector échange des informations entre robot et canal (utilisateur) en transmettant un objet [Activité][Activity]. Le type d’activité le plus courant est **message**, mais il existe d’autres types d’activités qui peuvent être utilisés pour communiquer différents types d’informations à un bot ou à un canal. 

## <a name="activity-types-in-the-bot-connector-service"></a>Types d’activités dans le service Bot Connector

Les types d’activités suivants sont pris en charge par le service Bot Connector.

| Type d’activité | Description |
|------|------|------|
| Message | Représente une communication entre le robot et l’utilisateur. |
| conversationUpdate | Indique que le robot a été ajouté à une conversation, que d’autres membres ont été ajoutés ou supprimés dans la conversation, ou que des métadonnées de la conversation ont changé. |
| contactRelationUpdate | Indique que le robot a été ajouté ou supprimé dans la liste des contacts d’un utilisateur. |
| typing | Indique que l’utilisateur ou le robot à l’autre extrémité de la conversation prépare une réponse. | 
| deleteUserData | Indique à un robot qu’un utilisateur lui a demandé de supprimer toutes les données utilisateur qu’il a stockées. |
| endOfConversation | Indique la fin d’une conversation. |

## <a name="message"></a>Message

Votre robot enverra des activités de **message** pour communiquer des informations et recevoir des activités de **message** de la part des utilisateurs. Certains messages peuvent être du texte brut, tandis que d’autres peuvent avoir un contenu plus riche, comme les [pièces jointes multimédia](bot-framework-rest-connector-add-media-attachments.md), les [boutons et les cartes](bot-framework-rest-connector-add-rich-cards.md), ou les [pièces spécifiques au canal](bot-framework-rest-connector-channeldata.md). Pour plus d’informations sur les propriétés de message les plus couramment utilisées, voir [Créer des messages](bot-framework-rest-connector-create-messages.md). Pour savoir comment envoyer et recevoir des messages, consultez [Envoyer et recevoir des messages](bot-framework-rest-connector-send-and-receive-messages.md). 

## <a name="conversationupdate"></a>conversationUpdate

Un bot reçoit une activité **conversationUpdate** chaque fois qu’il est ajouté à une conversation, que d’autres membres ont été ajoutés ou supprimés dans une conversation, ou que les métadonnées de la conversation ont changé. 

Si des membres ont été ajoutés à la conversation, la propriété `addedMembers` de l’activité identifiera les nouveaux membres. Si des membres ont été supprimés de la conversation, la propriété `removedMembers` identifiera les membres supprimés. 

> [!TIP]
> Si votre robot reçoit une activité **conversationUpdate** indiquant qu’un utilisateur a rejoint la conversation, vous pouvez choisir de répondre en envoyant un message de bienvenue à cet utilisateur. 

## <a name="contactrelationupdate"></a>contactRelationUpdate

Un robot reçoit une activité **contactRelationUpdate** chaque fois qu’il est ajouté ou supprimé dans la liste des contacts d’un utilisateur. La valeur de la propriété `action` de l’activité (add | remove) indique si le robot a été ajouté ou supprimé dans la liste des contacts de l’utilisateur.

## <a name="typing"></a>typing

Un bot reçoit une activité **typing** pour indiquer que l’utilisateur est en train de saisir une réponse. Un robot peut envoyer une activité **typing** pour indiquer à l’utilisateur qu’il est en train de répondre à une demande ou de préparer une réponse. 

## <a name="deleteuserdata"></a>deleteUserData

Un robot reçoit une activité **deleteUserData** quand un utilisateur demande la suppression de toutes les données que le robot a précédemment conservées à son sujet. Si votre robot reçoit ce type d’activité, il doit supprimer les informations d’identification personnelle (PII) qu’il a précédemment stockées pour l’utilisateur qui a effectué la requête.

## <a name="endofconversation"></a>endOfConversation 

Un bot reçoit une activité **endOfConversation** pour indiquer que l’utilisateur a mis fin à la conversation. Un bot peut envoyer une activité **endOfConversation** pour informer l’utilisateur que la conversation va se terminer. 

## <a name="additional-resources"></a>Ressources supplémentaires

- [Créer des messages](bot-framework-rest-connector-create-messages.md)
- [Envoyer et recevoir des messages](bot-framework-rest-connector-send-and-receive-messages.md)

[Activity]: bot-framework-rest-connector-api-reference.md#activity-object