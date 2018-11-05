---
title: Créer des messages avec l’API Bot Connector | Microsoft Docs
description: En savoir plus sur les propriétés de message couramment utilisées au sein de l’API Bot Connector.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
ms.openlocfilehash: cb78837c136379aab1ddf9ceebeb2aeb8360d83e
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2018
ms.locfileid: "50000286"
---
# <a name="create-messages"></a>Créer des messages

Votre bot enverra des objets [Activity][Activity] de type **message** pour communiquer des informations aux utilisateurs et, en retour, recevoir des activités de **message** de la part des utilisateurs. Certains messages peuvent consister simplement en un texte brut, tandis que d’autres peuvent contenir un contenu plus riche, par exemple un [texte à énoncer](bot-framework-rest-connector-text-to-speech.md), des [actions suggérées](bot-framework-rest-connector-add-suggested-actions.md), des [pièces jointes multimédia](bot-framework-rest-connector-add-media-attachments.md), des [cartes riches](bot-framework-rest-connector-add-rich-cards.md) et des [données spécifiques du canal](bot-framework-rest-connector-channeldata.md). Cet article décrit certaines des propriétés de message couramment utilisées.

## <a name="message-text-and-formatting"></a>Texte du message et mise en forme

Le texte du message peut être mis en forme à l’aide de **brut**, **markdown**, ou **xml**. Le format par défaut de la propriété `textFormat` est **markdown** et interprète le texte à l’aide de normes de mise en forme Markdown. Le niveau de prise en charge du format de texte varie en fonction des canaux. Pour vérifier si une fonctionnalité que vous souhaitez utiliser est prise en charge par le canal ciblé, affichez un aperçu de la fonctionnalité avec [Channel Inspector][ChannelInspector]. 

La propriété `textFormat` d’un objet [Activity][Activity] peut être utilisée pour spécifier le format du texte. Par exemple, pour créer un message de base contenant uniquement du texte brut, définissez la propriété `textFormat` de l’objet [Activity][Activity] sur **brut**, définissez la propriété `text` sur les contenus du message et définissez la propriété `locale` sur les paramètres régionaux de l’expéditeur. 

## <a name="attachments"></a>Pièces jointes

La propriété `attachments` de l’objet [Activity][Activity] peut être utilisée pour envoyer des pièces jointes de média simples (fichier image, audio, vidéo) et des cartes riches. Pour plus d’informations, consultez [Ajouter des pièces jointes multimédia aux messages](bot-framework-rest-connector-add-media-attachments.md) et [Ajouter des cartes riches aux messages](bot-framework-rest-connector-add-rich-cards.md).

## <a name="entities"></a>Entités

La propriété `entities` de l’objet [Activity][Activity] est un tableau d’objets <a href="http://schema.org/" target="_blank">schema.org</a> de durée indéterminée qui permet l’échange de métadonnées contextuelles communes entre le canal et le bot.

### <a name="mention-entities"></a>Mentionner des entités

De nombreux canaux permettent à un utilisateur ou à un bot de « mentionner » une personne dans le cadre d’une conversation. Pour mentionner un utilisateur dans un message, remplissez la propriété `entities` du message avec un objet [Mention][Mention]. 

### <a name="place-entities"></a>Placer des entités

Pour transmettre des <a href="https://schema.org/Place" target="_blank">informations liées à l’emplacement</a> dans un message, remplissez la propriété `entities` du message avec un objet [Place][Place]. 

## <a name="channel-data"></a>Données de canal

La propriété `channelData` de l’objet [Activity][Activity] peut être utilisée pour implémenter les fonctionnalités spécifiques au canal. Pour plus d’informations, consultez [Implémenter une fonctionnalité spécifique au canal](bot-framework-rest-connector-channeldata.md).

## <a name="text-to-speech"></a>Synthèse vocale

La propriété `speak` de l’objet [Activity][Activity] peut être utilisée pour spécifier le texte à prononcer par votre bot sur un canal de reconnaissance vocale, et la propriété `inputHint` de l’objet [ Activity][Activity] peut être utilisée pour influencer l’état du micro du client. Pour plus d’informations, consultez [Ajouter des fonctionnalités vocales aux messages](bot-framework-rest-connector-text-to-speech.md) et [Ajouter des conseils de saisie aux messages](bot-framework-rest-connector-add-input-hints.md).

## <a name="suggested-actions"></a>Actions suggérées

La propriété `suggestedActions` de l’objet [Activity][Activity] peut être utilisée pour présenter des boutons sur lesquels l’utilisateur peut appuyer pour effectuer une saisie. Contrairement aux boutons qui apparaissent dans les cartes enrichies (qui restent visibles et accessibles à l’utilisateur même après être touchées), les boutons qui apparaissent dans les volets des actions suggérées disparaissent une fois que l’utilisateur effectue une sélection. Pour plus d’informations, consultez [Ajouter des actions suggérées aux messages](bot-framework-rest-connector-add-suggested-actions.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Aperçu des fonctionnalités du bot avec l’inspecteur de canaux][ChannelInspector]
- [Vue d’ensemble des activités](bot-framework-rest-connector-activities.md)
- [Envoyer et recevoir des messages](bot-framework-rest-connector-send-and-receive-messages.md)
- [Ajouter des pièces jointes multimédia aux messages](bot-framework-rest-connector-add-media-attachments.md)
- [Ajouter des cartes détaillées aux messages](bot-framework-rest-connector-add-rich-cards.md)
- [Ajouter de la reconnaissance vocale aux messages](bot-framework-rest-connector-text-to-speech.md)
- [Ajouter des conseils de saisie aux messages](bot-framework-rest-connector-add-input-hints.md)
- [Ajouter des actions suggérées aux messages](bot-framework-rest-connector-add-suggested-actions.md)
- [Implémenter des fonctionnalités spécifiques au canal](bot-framework-rest-connector-channeldata.md)

[Mention]: bot-framework-rest-connector-api-reference.md#mention-object
[Place]: bot-framework-rest-connector-api-reference.md#place-object
[Activity]: bot-framework-rest-connector-api-reference.md#activity-object
[ChannelInspector]: ../bot-service-channel-inspector.md
[textFormating]: ../bot-service-channel-inspector.md#text-formatting
