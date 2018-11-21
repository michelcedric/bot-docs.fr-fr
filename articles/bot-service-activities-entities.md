---
title: Entités et types d’activités | Microsoft Docs
description: Entités et les types d’activités.
keywords: mentionner des entités, types d’activités, consommer des entités
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 03/01/2018
ms.openlocfilehash: d329fcbe5b4a34cb3e9c1fbf0160c5248020a508
ms.sourcegitcommit: cb0b70d7cf1081b08eaf1fddb69f7db3b95b1b09
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/09/2018
ms.locfileid: "51332963"
---
# <a name="entities-and-activity-types"></a>Entités et types d’activités

Les entités font partie d’une activité et fournissent des informations supplémentaires sur l’activité ou la conversation.

[!include[Entity boilerplate](includes/snippet-entity-boilerplate.md)]

## <a name="entities"></a>Entités

La propriété *entités* d’un message est un tableau d’objets <a href="http://schema.org/" target="_blank">schema.org</a> de durée indéterminée, permettant l’échange de métadonnées contextuelles communes entre le canal et le bot.

### <a name="mention-entities"></a>Mentionner des entités

De nombreux canaux permettent à un utilisateur ou un bot de « mentionner » une personne dans le cadre d’une conversation.
Pour mentionner un utilisateur dans un message, remplissez la propriété Entities du message avec un objet *mention*.
L’objet mention contient ces propriétés :

| Propriété | Description |
|----|----|
| type | type de l’entité (« mention ») |
| Mentionné | objet de compte de canal indiquant l’utilisateur qui a été mentionné | 
| Texte | texte de la propriété *activity.text* représentant la mention (peut être null ou vide) |

Cet exemple de code montre comment ajouter une entité mention à la collection d’entités.

# <a name="ctabcs"></a>[C#](#tab/cs)
[!code-csharp[set Mention](includes/code/dotnet-create-messages.cs#setMention)]

> [!TIP]
> Lorsque vous tentez de déterminer l’intention de l’utilisateur, le bot peut ignorer la partie du message dans laquelle cette intention est mentionnée. Appelez la méthode `GetMentions` et évaluez les objets `Mention` retournés dans la réponse.

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)
```javascript
var entity = context.activity.entities;

const mention = {
    type: "Mention",
    text: "@johndoe",
    mentioned: {
        name: "John Doe",
        id: "UV341235"
    }
}

entity = [mention];
```

---

### <a name="place-objects"></a>Placer des objets

Les <a href="https://schema.org/Place" target="_blank">informations liées à l’emplacement</a> peuvent être transmises au sein d’un message en remplissant la propriété des entités du message à l’aide d’un objet *Place* ou *GeoCoordinates*.

L’objet Place contient les propriétés suivantes :

| Propriété | Description |
|----|----|
| type | type de l’entité (« Place ») |
| Adresse | objet de description ou d’adresse postale (à venir) |
| Zone géographique | GeoCoordinates |
| HasMap | URL vers une carte ou un objet de carte (à venir) |
| NOM | nom du lieu |

L’objet geoCoordinates contient ces propriétés :

| Propriété | Description |
|----|----|
| type | type de l’entité (« GeoCoordinates ») |
| NOM | nom du lieu |
| Longitude | longitude de l’emplacement (<a href="https://en.wikipedia.org/wiki/World_Geodetic_System" target="_blank">WGS 84</a>) |
| Longitude | latitude de l’emplacement (<a href="https://en.wikipedia.org/wiki/World_Geodetic_System" target="_blank">WGS 84</a>) |
| Elevation | élévation de l’emplacement (<a href="https://en.wikipedia.org/wiki/World_Geodetic_System" target="_blank">WGS 84</a>) |

Cet exemple de code montre comment ajouter une entité place à la collection d’entités :

# <a name="ctabcs"></a>[C#](#tab/cs)
[!code-csharp[set GeoCoordinates](includes/code/dotnet-create-messages.cs#setGeoCoord)]

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)
```javascript
var entity = context.activity.entities;

const place = {
    elavation: 100,
    type: "GeoCoordinates",
    name : "myPlace",
    latitude: 123,
    longitude: 234
};

entity = [place];

```

---

### <a name="consume-entities"></a>Consommer des entités

# <a name="ctabcs"></a>[C#](#tab/cs)

Pour consommer des entités, utilisez le mot clé `dynamic` ou des classes fortement typées.

Cet exemple de code montre comment utiliser le mot clé `dynamic` pour traiter une entité dans la propriété `Entities` d’un message :

[!code-csharp[examine entity using dynamic keyword](includes/code/dotnet-create-messages.cs#examineEntity1)]

Cet exemple de code montre comment utiliser une classe fortement typée pour traiter une entité dans la propriété `Entities` d’un message :

[!code-csharp[examine entity using typed class](includes/code/dotnet-create-messages.cs#examineEntity2)]

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

Cet exemple de code montre comment traiter une entité dans la propriété `entity` d’un message :

```javascript
if (entity[0].type === "GeoCoordinates" && entity[0].latitude > 34) {
    // do something
}
```

---

## <a name="activity-types"></a>Types d'activités

Cet exemple de code montre comment traiter une activité de type **message** :

# <a name="ctabcs"></a>[C#](#tab/cs)

```cs
if (context.Activity.Type == ActivityTypes.Message){
    // do something
}
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

```js
if(context.activity.type === 'message'){
    // do something
}
```

---

Les activités peuvent être de différents types, au-delà du **message** le plus courant. Il existe plusieurs types d’activités :

| Activity.Type | Interface | Description |
|-----|-----|-----|
| [message](#message) | IMessageActivity (C#) <br> Activité (JS) | Représente une communication entre le bot et l’utilisateur. |
| [contactRelationUpdate](#contactrelationupdate) | IContactRelationUpdateActivity (C#) <br> Activité (JS) | Indique que le bot a été ajouté ou supprimé de la liste des contacts d’un utilisateur. |
| [conversationUpdate](#conversationupdate) | IConversationUpdateActivity (C#) <br> Activité (JS) | Indique que le bot a été ajouté à une conversation, que d’autres membres ont été ajoutés ou supprimés de la conversation, ou que les métadonnées de la conversation ont changé. |
| [deleteUserData](#deleteuserdata) | n/a | Indique à un robot qu’un utilisateur lui a demandé de supprimer toutes les données utilisateur qu’il a stockées. |
| [endOfConversation](#endofconversation) | IEndOfConversationActivity (C#) <br> Activité (JS) | Indique la fin d’une conversation. |
| [event](#event) | IEventActivity (C#) <br> Activité (JS) | Représente une communication envoyée à un bot non visible par l’utilisateur. |
| [installationUpdate](#installationupdate) | IInstallationUpdateActivity (C#) <br> Activité (JS) | Représente une installation ou désinstallation d’un bot au sein d’une unité d’organisation (par exemple un locataire client ou une « équipe ») d’un canal. |
| [invoke](#invoke) | IInvokeActivity (C#) <br> Activité (JS) | Représente une communication envoyée à un bot pour lui demander d’effectuer une opération spécifique. Ce type d’activité est réservé à un usage interne par Microsoft Bot Framework. |
| [messageReaction](#messagereaction) | IMessageReactionActivity (C#) <br> Activité (JS) | Indique qu’un utilisateur a réagi à une activité existante. Par exemple, un utilisateur clique sur le bouton « J’aime » sur un message. |
| [typing](#typing) | ITypingActivity (C#) <br> Activité (JS) | Indique que l’utilisateur ou le robot à l’autre extrémité de la conversation prépare une réponse. |

## <a name="message"></a>Message

<!-- Only the last link is different. -->

::: moniker range="azure-bot-service-3.0"

Votre bot enverra des activités de message pour communiquer des informations et recevoir des activités de message de la part des utilisateurs.
Certains messages peuvent consister simplement en un texte brut, tandis que d’autres peuvent avoir un contenu plus riche, par exemple un texte à énoncer, des [actions suggérées](v4sdk/bot-builder-howto-add-suggested-actions.md), des [pièces jointes multimédias](v4sdk/bot-builder-howto-add-media-attachments.md), des [cartes riches](v4sdk/bot-builder-howto-add-media-attachments.md#send-a-hero-card) et des [données spécifiques du canal](~/dotnet/bot-builder-dotnet-channeldata.md).

::: moniker-end

::: moniker range="azure-bot-service-4.0"

Votre bot enverra des activités de message pour communiquer des informations et recevoir des activités de message de la part des utilisateurs.
Certains messages peuvent consister simplement en un texte brut, tandis que d’autres peuvent contenir un contenu plus riche, par exemple un texte à énoncer, des [actions suggérées](v4sdk/bot-builder-howto-add-suggested-actions.md), des [pièces jointes multimédias](v4sdk/bot-builder-howto-add-media-attachments.md), des [cartes riches](v4sdk/bot-builder-howto-add-media-attachments.md#send-a-hero-card) et des [données spécifiques du canal](~/v4sdk/bot-builder-channeldata.md).

::: moniker-end

## <a name="contactrelationupdate"></a>contactRelationUpdate

Un bot reçoit une activité de mise à jour liée à un contact chaque fois qu’il est ajouté ou supprimé de la liste des contacts d’un utilisateur. La valeur de la propriété action de l’activité (add | remove) indique si le bot a été ajouté ou supprimé de la liste des contacts de l’utilisateur.

## <a name="conversationupdate"></a>conversationUpdate

Un bot reçoit une activité de mise à jour d’une conversation chaque fois qu’il est ajouté à une conversation, que d’autres membres ont été ajoutés ou supprimés d’une conversation, ou que les métadonnées de la conversation ont changé.

Si des membres ont été ajoutés à la conversation, la propriété de l’activité des membres ajoutés contient un tableau d’objets de compte de canal permettant d’identifier les nouveaux membres.

Pour déterminer si votre bot a été ajouté à la conversation (par exemple, est un des nouveaux membres), déterminez si la valeur d’ID destinataire de l’activité (par exemple, l’ID de votre bot) correspond à la propriété ID d’un des comptes dans le tableau des membres ajoutés.

Si des membres ont été supprimés de la conversation, la propriété des membres supprimés contient un tableau d’objets de compte de canal permettant d’identifier les membres supprimés.

> [!TIP]
> Si votre bot reçoit une activité de mise à jour de conversation indiquant qu’un utilisateur a rejoint la conversation, vous pouvez lui demander de répondre en envoyant un message de bienvenue à cet utilisateur.

## <a name="deleteuserdata"></a>deleteUserData

Un bot reçoit une activité de suppression des données utilisateur lorsqu’un utilisateur demande la suppression de toutes les données que le bot a précédemment conservées à son sujet. Si votre bot reçoit ce type d’activité, il doit supprimer les informations d’identification personnelle (PII) qu’il a précédemment stockées pour l’utilisateur qui a effectué la demande.

## <a name="endofconversation"></a>endOfConversation

Un bot reçoit une activité de fin de conversation pour indiquer que l’utilisateur a mis fin à la conversation. Un bot peut envoyer une activité de fin de conversation pour informer l’utilisateur que la conversation va se terminer.

## <a name="event"></a>événement

Votre bot peut recevoir une activité d’événement provenant d’un processus ou d’un service externe qui souhaite lui communiquer des informations, sans que ces informations soient visibles par les utilisateurs. En règle générale, l’expéditeur d’une activité d’événement n’attend aucun accusé de réception de la part du bot.

## <a name="installationupdate"></a>installationUpdate

Les activités de mise à jour d’installation représentent une installation ou désinstallation d’un bot au sein d’une unité d’organisation (par exemple un locataire client ou une « équipe ») d’un canal. En général, elles ne représentent pas l’ajout ou la suppression d’un canal. Les canaux peuvent envoyer des activités d’installation quand un bot est ajouté ou supprimé d’un locataire, d’une équipe ou autre unité d’organisation dans le canal. Les canaux ne devraient pas envoyer d’activités d’installation quand le bot est installé ou supprimé d’un canal.

## <a name="invoke"></a>invoke

Votre bot peut recevoir une activité invoke lui demandant d’effectuer une opération spécifique.
L’expéditeur d’une activité invoke attend généralement un accusé de réception du bot via une réponse HTTP.
Ce type d’activité est réservé à un usage interne par Microsoft Bot Framework.

## <a name="messagereaction"></a>messageReaction

Certains canaux enverront des activités de réaction au message à votre bot lorsqu’un utilisateur réagit à une activité existante. Par exemple, un utilisateur clique sur le bouton « J’aime » sur un message. La propriété replyToId indiquera l’activité à laquelle l’utilisateur a réagi.

L’activité de réaction à un message peut correspondre à n’importe quel nombre de types de réaction à un message défini par le canal. Par exemple, « Like » (j’aime) ou « PlusOne » (+1) sont des types de réactions qu’un canal peut envoyer.

## <a name="typing"></a>typing

Un bot reçoit une activité de saisie pour indiquer que l’utilisateur saisit une réponse.
Un bot peut envoyer une activité de saisie pour indiquer à l’utilisateur qu’il est en train de répondre à une demande ou de formuler une réponse.

::: moniker range="azure-bot-service-3.0"

## <a name="additional-resources"></a>Ressources supplémentaires

- <a href="https://docs.botframework.com/en-us/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html" target="_blank">Classe Activité</a>
::: moniker-end
