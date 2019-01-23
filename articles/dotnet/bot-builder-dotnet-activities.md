---
title: Vue d’ensemble des activités | Microsoft Docs
description: Découvrez les différents types d’activité disponibles dans le kit SDK Bot Framework pour .NET.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 076e460f393c5db524cfade81e5c007484fe2cca
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/11/2019
ms.locfileid: "54225664"
---
# <a name="activities-overview"></a>Vue d’ensemble des activités

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

[!INCLUDE [Activity concept overview](../includes/snippet-dotnet-concept-activity.md)]

## <a name="activity-types-in-the-bot-framework-sdk-for-net"></a>Types d’activité dans le kit SDK Bot Framework pour .NET

Les types d’activité pris en charge par le kit SDK Bot Framework pour .NET sont les suivants.

| Activity.Type | Interface | Description |
|------|------|------|
| [message](#message) | IMessageActivity | Représente une communication entre le robot et l’utilisateur. |
| [conversationUpdate](#conversationupdate) | IConversationUpdateActivity | Indique que le robot a été ajouté à une conversation, que d’autres membres ont été ajoutés ou supprimés dans la conversation, ou que des métadonnées de la conversation ont changé. |
| [contactRelationUpdate](#contactrelationupdate) | IContactRelationUpdateActivity | Indique que le robot a été ajouté ou supprimé dans la liste des contacts d’un utilisateur. |
| [typing](#typing) | ITypingActivity | Indique que l’utilisateur ou le robot à l’autre extrémité de la conversation prépare une réponse. | 
| [deleteUserData](#deleteuserdata) | n/a | Indique à un robot qu’un utilisateur lui a demandé de supprimer toutes les données utilisateur qu’il a stockées. |
| [endOfConversation](#endofconversation) | IEndOfConversationActivity | Indique la fin d’une conversation. |
| [event](#event) | IEventActivity | Représente une communication envoyée à un robot non visible par l’utilisateur. |
| [invoke](#invoke) | IInvokeActivity | Représente une communication envoyée à un robot pour lui demander d’effectuer une opération spécifique. Ce type d’activité est réservé à un usage interne par Microsoft Bot Framework. |
| [messageReaction](#messagereaction) | IMessageReactionActivity | Indique qu’un utilisateur a réagi à une activité existante. Par exemple, un utilisateur clique sur le bouton « J’aime » sur un message. |

## <a name="message"></a>Message

Votre robot enverra des activités de **message** pour communiquer des informations et recevoir des activités de **message** de la part des utilisateurs. Certains messages peuvent consister simplement en un texte brut, tandis que d’autres peuvent contenir un contenu plus riche, par exemple un [texte à énoncer](bot-builder-dotnet-text-to-speech.md), des [actions suggérées](bot-builder-dotnet-add-suggested-actions.md), des [pièces jointes multimédia](bot-builder-dotnet-add-media-attachments.md), des [cartes riches](bot-builder-dotnet-add-rich-card-attachments.md) et des [données spécifiques du canal](bot-builder-dotnet-channeldata.md). Pour plus d’informations sur les propriétés de message les plus couramment utilisées, voir [Créer des messages](bot-builder-dotnet-create-messages.md).

## <a name="conversationupdate"></a>conversationUpdate

Un bot reçoit une activité **conversationUpdate** chaque fois qu’il est ajouté à une conversation, que d’autres membres ont été ajoutés ou supprimés dans une conversation, ou que les métadonnées de la conversation ont changé. 

Si des membres ont été ajoutés à la conversation, la propriété de l’activité `MembersAdded` contient un tableau d’objets `ChannelAccount` permettant d’identifier les nouveaux membres. 

Pour déterminer si votre robot a été ajouté à la conversation (par exemple, est l’un des nouveaux membres), déterminez si la valeur `Recipient.Id` pour l’activité (par exemple, l’ID de votre robot) correspond à la propriété `Id` d’un des comptes dans le tableau `MembersAdded`.

Si des membres ont été supprimés de la conversation, la propriété `MembersRemoved` contient un tableau d’objets `ChannelAccount` permettant d’identifier les membres supprimés. 

> [!TIP]
> Si votre robot reçoit une activité **conversationUpdate** indiquant qu’un utilisateur a rejoint la conversation, vous pouvez lui demander de répondre en envoyant un message de bienvenue à cet utilisateur. 

## <a name="contactrelationupdate"></a>contactRelationUpdate

Un robot reçoit une activité **contactRelationUpdate** chaque fois qu’il est ajouté ou supprimé dans la liste des contacts d’un utilisateur. La valeur de la propriété `Action` de l’activité (add | remove) indique si le robot a été ajouté ou supprimé dans la liste des contacts de l’utilisateur.

## <a name="typing"></a>typing

Un bot reçoit une activité **typing** pour indiquer que l’utilisateur est en train de saisir une réponse. Un robot peut envoyer une activité **typing** pour indiquer à l’utilisateur qu’il est en train de répondre à une demande ou de préparer une réponse. 

## <a name="deleteuserdata"></a>deleteUserData

Un robot reçoit une activité **deleteUserData** quand un utilisateur demande la suppression de toutes les données que le robot a précédemment conservées à son sujet. Si votre robot reçoit ce type d’activité, il doit supprimer les informations d’identification personnelle (PII) qu’il a précédemment stockées pour l’utilisateur qui a effectué la requête.

## <a name="endofconversation"></a>endOfConversation 

Un bot reçoit une activité **endOfConversation** pour indiquer que l’utilisateur a mis fin à la conversation. Un bot peut envoyer une activité **endOfConversation** pour informer l’utilisateur que la conversation va se terminer. 

## <a name="event"></a>événement

Votre bot peut recevoir une activité **event** provenant d’un processus ou d’un service externe qui souhaite lui communiquer des informations, sans que ces informations soient visibles par les utilisateurs. L’expéditeur d’une activité **event** n’attend généralement aucun accusé de réception de la part du robot.

## <a name="invoke"></a>invoke

Votre robot peut recevoir une activité **invoke** lui demandant d’effectuer une opération spécifique. L’expéditeur d’une activité **invoke** attend généralement un accusé de réception du robot via une réponse HTTP. Ce type d’activité est réservé à un usage interne par Microsoft Bot Framework.

## <a name="messagereaction"></a>messageReaction

Certains canaux envoient des activités **messageReaction** à votre robot quand un utilisateur réagit à une activité existante. Par exemple, un utilisateur clique sur le bouton « J’aime » sur un message. La propriété **ReplyToId** indiquera l’activité à laquelle l’utilisateur a réagi.

L’activité **messageReaction** peut correspondre à n’importe quel nombre de **messageReactionTypes** défini par le canal. Par exemple, « Like » (j’aime) ou « PlusOne » (+1) sont des types de réactions qu’un canal peut envoyer. 

## <a name="additional-resources"></a>Ressources supplémentaires

- [Envoyer et recevoir des activités](bot-builder-dotnet-connector.md)
- [Créer des messages](bot-builder-dotnet-create-messages.md)
- <a href="https://docs.botframework.com/en-us/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html" target="_blank">Classe Activité</a>
