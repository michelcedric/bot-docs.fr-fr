---
title: Ajouter des pièces jointes de cartes enrichies aux messages | Microsoft Docs
description: Découvrez comment ajouter des cartes enrichies aux messages à l’aide du kit SDK Bot Framework pour .NET.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 5a6fc63005797a1c645de7506a8f15df2dcd0557
ms.sourcegitcommit: b94361234816e6b95459f142add936732fc40344
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/15/2019
ms.locfileid: "54317674"
---
# <a name="add-rich-card-attachments-to-messages"></a>Ajouter des pièces jointes de cartes enrichies aux messages

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-add-rich-card-attachments.md)
> - [Node.JS](../nodejs/bot-builder-nodejs-send-rich-cards.md)
> - [REST](../rest-api/bot-framework-rest-connector-add-rich-cards.md)

Un échange de messages entre l’utilisateur et le robot peut contenir une ou plusieurs cartes enrichies présentées sous forme de liste ou de carrousel. La propriété `Attachments` de l’objet <a href="https://docs.botframework.com/en-us/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html" target="_blank">Activité</a> contient un tableau d’objets <a href="https://docs.microsoft.com/en-us/dotnet/api/microsoft.bot.connector.attachments?view=botconnector-3.12.2.4" target="_blank">Pièces jointes</a> qui représentent les cartes enrichies et les pièces jointes du message. 

> [!NOTE]
> Pour en savoir plus sur la façon d’ajouter des pièces jointes aux messages, consultez [Ajouter des pièces jointes aux messages](bot-builder-dotnet-add-media-attachments.md).

## <a name="types-of-rich-cards"></a>Types de cartes enrichies

Bot Framework prend actuellement en charge huit types de cartes enrichies : 

| Type de carte | Description |
|----|----|
| <a href="/adaptive-cards/get-started/bots">Carte adaptative</a> | Carte personnalisable pouvant inclure n’importe quelle combinaison de texte, données vocales, images, boutons et champs d’entrée. Consultez l’article sur la [prise en charge de ces cartes par canal](/adaptive-cards/get-started/bots#channel-status).  |
| [Carte d’animation][animationCard] | Carte pouvant lire des images GIF animées ou de courtes vidéos. |
| [Carte audio][audioCard] | Carte pouvant lire un fichier audio. |
| [Carte de bannière][heroCard] | Carte contenant généralement une image de grande taille, un ou plusieurs boutons, ainsi que du texte. |
| [Carte de miniature][thumbnailCard] | Carte contenant généralement une image miniature, un ou plusieurs boutons, ainsi que du texte. |
| [Carte de reçu][receiptCard] | Carte permettant à un robot de fournir un reçu à l’utilisateur. Elle contient généralement la liste des articles à inclure sur le reçu, la taxe et le total, ainsi que du texte. |
| [Carte de connexion][signinCard] | Carte permettant à un robot de demander à un utilisateur de se connecter. Elle contient généralement du texte et un ou plusieurs boutons sur lesquels l’utilisateur peut cliquer pour lancer le processus de connexion. |
| [Carte vidéo][videoCard] | Carte pouvant lire des vidéos. |

> [!TIP]
> Pour afficher plusieurs cartes enrichies au format liste, définissez la propriété `AttachmentLayout` de l’activité à « liste ». Pour afficher plusieurs cartes enrichies au format carrousel, définissez la propriété `AttachmentLayout` de l’activité à « carrousel ». Si le canal ne prend pas en charge le format carrousel, les cartes riches s’afficheront au format liste, bien que la propriété `AttachmentLayout` indique « carrousel ».

## <a name="process-events-within-rich-cards"></a>Traiter des événements dans les cartes enrichies

Pour traiter les événements dans les cartes enrichies, définissez les objets `CardAction` pour spécifier ce qui doit se produire quand l’utilisateur clique sur un bouton ou appuie sur une section de la carte. Chaque objet `CardAction` contient les propriétés suivantes :

| Propriété | type | Description | 
|----|----|----|
| type | chaîne | type d’action (une des valeurs indiquées dans le tableau ci-dessous) |
| Intitulé | chaîne | titre du bouton |
| Image | chaîne | URL d’image du bouton |
| Valeur | chaîne | valeur nécessaire pour effectuer le type d’action spécifié |

> [!NOTE]
> Les boutons dans les cartes adaptatives ne sont pas créés avec les objets `CardAction`, mais à l’aide du schéma défini par les <a href="http://adaptivecards.io" target="_blank">cartes adaptatives</a>. Pour obtenir un exemple illustrant comment ajouter des boutons à une carte adaptative, consultez [Ajouter une carte adaptative à un message](#adaptive-card).

Ce tableau répertorie les valeurs valides pour `CardAction.Type` et décrit le contenu attendu de `CardAction.Value` pour chaque type :

| CardAction.Type | CardAction.Value | 
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

## <a name="add-a-hero-card-to-a-message"></a>Ajouter une carte de héros à un message

La carte de héros contient généralement une image de grande taille, un ou plusieurs boutons et du texte. 

Cet exemple de code illustre comment créer un message de réponse contenant trois cartes héros au format carrousel : 

[!code-csharp[Add HeroCard attachment](../includes/code/dotnet-add-attachments.cs#addHeroCardAttachment)]

## <a name="add-a-thumbnail-card-to-a-message"></a>Ajouter une carte de miniature à un message

La carte miniature contient généralement une image miniature, un ou plusieurs boutons et du texte. 

Cet exemple de code illustre comment créer un message de réponse contenant deux cartes de miniatures au format liste : 

[!code-csharp[Add ThumbnailCard attachment](../includes/code/dotnet-add-attachments.cs#addThumbnailCardAttachment)]

## <a name="add-a-receipt-card-to-a-message"></a>Ajouter une carte de reçu à un message

La carte de reçu permet à un robot de fournir un reçu à l’utilisateur. Elle contient généralement la liste des articles à inclure sur le reçu, la taxe et le total, ainsi que du texte. 

Cet exemple de code illustre comment créer un message de réponse contenant une carte de reçu : 

[!code-csharp[Add ReceiptCard attachment](../includes/code/dotnet-add-attachments.cs#addReceiptCardAttachment)]

## <a name="add-a-sign-in-card-to-a-message"></a>Ajouter une carte de connexion à un message

La carte de connexion permet à un robot de demander à un utilisateur de se connecter. Elle contient généralement du texte et un ou plusieurs boutons sur lesquels l’utilisateur peut cliquer pour lancer le processus de connexion. 

Cet exemple de code illustre comment créer un message de réponse contenant une carte de connexion :

[!code-csharp[Add SignInCard attachment](../includes/code/dotnet-add-attachments.cs#addSignInCardAttachment)]

## <a id="adaptive-card"></a> Ajouter une carte adaptative à un message

La carte adaptive peut inclure n’importe quelle combinaison de texte, données vocales, images, boutons et champs d’entrée. Les cartes adaptatives sont créées à l’aide du format JSON spécifié sur le site <a href="http://adaptivecards.io" target="_blank">Adaptive Cards</a> (Cartes adaptatives), ce qui vous donne un contrôle total sur le contenu et le format de la carte. 

Pour créer une carte adaptative à l’aide de .NET, installez le package NuGet `AdaptiveCards`. Tirez ensuite profit des informations fournies par le site <a href="http://adaptivecards.io" target="_blank">Adaptive Cards</a> (Cartes adaptatives) pour comprendre le schéma de carte adaptative, explorer les éléments de carte adaptative et découvrir des exemples JSON qui permettent de créer des cartes présentant différents types de compositions et niveaux de complexité. En outre, vous pouvez utiliser le visualiseur interactif pour concevoir des charges utiles de carte adaptative et afficher un aperçu de la sortie des cartes.

Cet exemple de code indique comment créer un message contenant une carte adaptative pour un rappel du Calendrier : 

[!code-csharp[Add Adaptive Card attachment](../includes/code/dotnet-add-attachments.cs#addAdaptiveCardAttachment)]

La carte résultante contient trois blocs de texte, un champ d’entrée (liste de choix) et trois boutons :

![Carte adaptative de rappel du Calendrier](../media/adaptive-card-reminder.png)

## <a name="additional-resources"></a>Ressources supplémentaires

- [Aperçu des fonctionnalités du bot avec l’inspecteur de canaux][inspector]
- <a href="http://adaptivecards.io" target="_blank">Cartes adaptatives</a>
- [Vue d’ensemble des activités](bot-builder-dotnet-activities.md)
- [Créer des messages](bot-builder-dotnet-create-messages.md)
- [Ajouter des pièces jointes multimédia aux messages](bot-builder-dotnet-add-media-attachments.md)
- <a href="https://docs.botframework.com/en-us/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html" target="_blank">Classe d’activité</a>
- <a href="https://docs.microsoft.com/en-us/dotnet/api/microsoft.bot.connector.attachments?view=botconnector-3.12.2.4" target="_blank">Attachments Class</a> (Classe Attachments)

[animationCard]: /dotnet/api/microsoft.bot.connector.animationcard

[audioCard]: /dotnet/api/microsoft.bot.connector.audiocard 

[heroCard]: /dotnet/api/microsoft.bot.connector.herocard 

[thumbnailCard]: /dotnet/api/microsoft.bot.connector.thumbnailcard 

[receiptCard]: /dotnet/api/microsoft.bot.connector.receiptcard 

[signinCard]: /dotnet/api/microsoft.bot.connector.signincard 

[videoCard]: /dotnet/api/microsoft.bot.connector.videocard

[inspector]: ../bot-service-channel-inspector.md
