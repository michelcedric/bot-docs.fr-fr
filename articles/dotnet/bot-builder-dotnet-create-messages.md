---
title: Créer des messages avec le Kit SDK Bot Builder pour .NET | Microsoft Docs
description: En savoir plus sur les propriétés de message couramment utilisées au sein du kit de développement logiciel (SDK) pour .NET.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: c35e651f674d65728ac93a815cc7116515790f53
ms.sourcegitcommit: 3bf3dbb1a440b3d83e58499c6a2ac116fe04b2f6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/23/2018
ms.locfileid: "46707875"
---
# <a name="create-messages"></a>Créer des messages

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

Votre bot enverra des [activités](bot-builder-dotnet-activities.md) de **message** pour communiquer des informations aux utilisateurs et, en retour, recevoir des activités de **message** de la part des utilisateurs. Certains messages peuvent consister simplement en un texte brut, tandis que d’autres peuvent contenir un contenu plus riche, par exemple un [texte à énoncer](bot-builder-dotnet-text-to-speech.md), des [actions suggérées](bot-builder-dotnet-add-suggested-actions.md), des [pièces jointes multimédia](bot-builder-dotnet-add-media-attachments.md), des [cartes riches](bot-builder-dotnet-add-rich-card-attachments.md) et des [données spécifiques du canal](bot-builder-dotnet-channeldata.md). 

Cet article décrit certaines des propriétés de message couramment utilisées.

## <a name="customizing-a-message"></a>Personnalisation d’un message

Pour avoir plus de contrôle sur la mise en forme du texte de vos messages, vous pouvez créer un message personnalisé utilisant l’[activité](https://docs.botframework.com/en-us/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html) et définir les propriétés nécessaires avant de l’envoyer à l’utilisateur.

Cet exemple montre comment créer un objet `message` personnalisé et définir les propriétés `Text`, `TextFormat` et `Local`.

[!code-csharp[Set message properties](../includes/code/dotnet-create-messages.cs#setBasicProperties)]

La propriété `TextFormat` d’un message peut être utilisée pour spécifier le format du texte. La propriété `TextFormat` peut être définie sur **brut**, **markdown** ou **xml**. La valeur par défaut de `TextFormat` est **markdown**. 

## <a name="attachments"></a>Pièces jointes

La propriété `Attachments` d’une activité de message peut être utilisée pour envoyer et recevoir des pièces jointes de média simples (fichier image, audio, vidéo) et des cartes riches. Pour plus d’informations, consultez [Ajouter des pièces jointes multimédia aux messages](bot-builder-dotnet-add-media-attachments.md) et [Ajouter des cartes riches aux messages](bot-builder-dotnet-add-rich-card-attachments.md).

## <a name="entities"></a>Entités

La propriété `Entities` d’un message est un tableau d’objets <a href="http://schema.org/" target="_blank">schema.org</a> de durée indéterminée, permettant l’échange de métadonnées contextuelles communes entre le canal et le bot.

### <a name="mention-entities"></a>Mentionner des entités

De nombreux canaux permettent à un utilisateur ou un bot de « mentionner » une personne dans le cadre d’une conversation. Pour mentionner un utilisateur dans un message, remplissez la propriété `Entities` du message avec un objet `Mention`. L’objet `Mention` contient les propriétés suivantes : 

| Propriété | Description | 
|----|----|
| type | type de l’entité (« mention ») | 
| Mentionné | Objet `ChannelAccount` indiquant l’utilisateur qui a été mentionné | 
| Texte | texte de la propriété `Activity.Text` représentant la mention (peut être null ou vide) |

Cet exemple de code montre comment ajouter une entité `Mention` à la collection `Entities`.

[!code-csharp[set Mention](../includes/code/dotnet-create-messages.cs#setMention)]

> [!TIP]
> Lorsque vous tentez de déterminer l’intention de l’utilisateur, le bot peut ignorer la partie du message dans laquelle cette intention est mentionnée. Appelez la méthode `GetMentions` et évaluez les objets `Mention` retournés dans la réponse.

### <a name="place-objects"></a>Placer des objets

Les <a href="https://schema.org/Place" target="_blank">informations liées à l’emplacement</a> peuvent être transmises au sein d’un message en remplissant la propriété `Entities` du message à l’aide d’un objet `Place` ou `GeoCoordinates`. 

L’objet `Place` contient les propriétés suivantes :

| Propriété | Description | 
|----|----|
| type | type de l’entité (« Place ») |
| Adresse | description ou objet `PostalAddress` (à venir) | 
| Zone géographique | GeoCoordinates | 
| HasMap | URL vers une carte ou un objet `Map` (à venir) |
| NOM | nom du lieu |

L’objet `GeoCoordinates` contient les propriétés suivantes :

| Propriété | Description | 
|----|----|
| type | type de l’entité (« GeoCoordinates ») |
| NOM | nom du lieu |
| Longitude | longitude de l’emplacement (<a href="https://en.wikipedia.org/wiki/World_Geodetic_System" target="_blank">WGS 84</a>) | 
| Latitude | latitude de l’emplacement (<a href="https://en.wikipedia.org/wiki/World_Geodetic_System" target="_blank">WGS 84</a>) | 
| Elevation | élévation de l’emplacement (<a href="https://en.wikipedia.org/wiki/World_Geodetic_System" target="_blank">WGS 84</a>) | 

Cet exemple de code montre comment ajouter une entité `Place` à la collection `Entities` :

[!code-csharp[set GeoCoordinates](../includes/code/dotnet-create-messages.cs#setGeoCoord)]

### <a name="consume-entities"></a>Consommer des entités

Pour consommer des entités, utilisez le mot clé `dynamic` ou des classes fortement typées.

Cet exemple de code montre comment utiliser le mot clé `dynamic` pour traiter une entité dans la propriété `Entities` d’un message :

[!code-csharp[examine entity using dynamic keyword](../includes/code/dotnet-create-messages.cs#examineEntity1)]

Cet exemple de code montre comment utiliser une classe fortement typée pour traiter une entité dans la propriété `Entities` d’un message :

[!code-csharp[examine entity using typed class](../includes/code/dotnet-create-messages.cs#examineEntity2)]

## <a name="channel-data"></a>Données de canal

La propriété `ChannelData`d’une activité de message peut être utilisée pour implémenter les fonctionnalités spécifiques au canal. Pour plus d’informations, consultez [Implémenter une fonctionnalité spécifique au canal](bot-builder-dotnet-channeldata.md).

## <a name="text-to-speech"></a>Synthèse vocale

La propriété `Speak` d’une activité de message peut être utilisée pour spécifier le texte à énoncer par votre bot sur un canal de reconnaissance vocale. La propriété `InputHint` d’une activité de message peut être utilisée pour contrôler l’état du micro du client et la zone d’entrée (le cas échéant). Pour plus d’informations, consultez [Ajouter la reconnaissance vocale aux messages](bot-builder-dotnet-text-to-speech.md).

## <a name="suggested-actions"></a>Actions suggérées

La propriété `SuggestedActions` d’une activité de message peut être utilisée pour présenter des boutons sur lesquels l’utilisateur peut appuyer pour effectuer une saisie. Contrairement aux boutons qui apparaissent dans les cartes enrichies (qui restent visibles et accessibles à l’utilisateur même après être touchées), les boutons qui apparaissent dans les volets des actions suggérées disparaissent une fois que l’utilisateur effectue une sélection. Pour plus d’informations, consultez [Ajouter des actions suggérées aux messages](bot-builder-dotnet-add-suggested-actions.md).

## <a name="next-steps"></a>Étapes suivantes

Un bot et un utilisateur peuvent s’envoyer des messages. Lorsque le message est plus complexe, votre bot peut envoyer une carte riche dans un message à l’utilisateur. Les cartes riches couvrent de nombreux scénarios de présentation et d’interaction dont la plupart des bots ont besoin.

> [!div class="nextstepaction"]
> [Envoyer une carte riche dans un message](bot-builder-dotnet-add-rich-card-attachments.md)

## <a name="additional-resources"></a>Ressources supplémentaires

- [Vue d’ensemble des activités](bot-builder-dotnet-activities.md)
- [Envoyer et recevoir des activités](bot-builder-dotnet-connector.md)
- [Ajouter des pièces jointes multimédia aux messages](bot-builder-dotnet-add-media-attachments.md)
- [Ajouter des cartes détaillées aux messages](bot-builder-dotnet-add-rich-card-attachments.md)
- [Ajouter la reconnaissance vocale aux messages](bot-builder-dotnet-text-to-speech.md)
- [Ajouter des actions suggérées aux messages](bot-builder-dotnet-add-suggested-actions.md)
- [Implémenter des fonctionnalités spécifiques au canal](bot-builder-dotnet-channeldata.md)
- <a href="https://docs.botframework.com/en-us/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html" target="_blank">Classe Activité</a>
- <a href="/dotnet/api/microsoft.bot.connector.imessageactivity" target="_blank">Interface IMessageActivity</a>

