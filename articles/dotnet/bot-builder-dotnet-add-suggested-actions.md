---
title: Ajouter des actions suggérées aux messages | Microsoft Docs
description: Découvrez comment ajouter des actions suggérées aux messages à l’aide du kit SDK Bot Framework pour .NET.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 03/13/2018
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: ad9b791cf74c4a67515fdf8d60eab29c51f93bc2
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/11/2019
ms.locfileid: "54224741"
---
# <a name="add-suggested-actions-to-messages"></a>Ajouter des actions suggérées aux messages

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-add-suggested-actions.md)
> - [Node.JS](../nodejs/bot-builder-nodejs-send-suggested-actions.md)
> - [REST](../rest-api/bot-framework-rest-connector-add-suggested-actions.md)

[!INCLUDE [Introduction to suggested actions](../includes/snippet-suggested-actions-intro.md)]

> [!TIP]
> Utilisez [l’inspecteur de canaux][channelInspector] pour découvrir l’aspect et le mode de fonctionnement des actions suggérées sur les différents canaux.

## <a name="send-suggested-actions"></a>Envoyer des actions suggérées

Pour ajouter des actions suggérées à un message, définissez la propriété `SuggestedActions` de l’activité sur une liste d’objets [CardAction][cardAction] qui représentent les boutons à présenter à l’utilisateur. 

Cet exemple de code indique comment créer un message qui présente trois actions suggérées à l’utilisateur :

[!code-csharp[Add suggested actions](../includes/code/dotnet-add-suggested-actions.cs#addSuggestedActions)]

Lorsque l’utilisateur appuie sur l’une des actions suggérées, le robot reçoit un message de l’utilisateur contenant la valeur `Value` de l’action correspondante.

## <a name="additional-resources"></a>Ressources supplémentaires

- [Aperçu des fonctionnalités du bot avec l’inspecteur de canaux][inspector]
- [Vue d’ensemble des activités](bot-builder-dotnet-activities.md)
- [Créer des messages](bot-builder-dotnet-create-messages.md)
- <a href="https://docs.botframework.com/en-us/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html" target="_blank">Classe Activity</a>
- <a href="/dotnet/api/microsoft.bot.connector.imessageactivity" target="_blank">Interface IMessageActivity</a>
- <a href="/dotnet/api/microsoft.bot.connector.cardaction" target="_blank">CardAction class</a> (Classe CardAction)
- <a href="/dotnet/api/microsoft.bot.connector.suggestedactions" target="_blank">SuggestedActions class</a> (Classe SuggestedActions)

[cardAction]: /dotnet/api/microsoft.bot.connector.cardaction

[inspector]: ../bot-service-channel-inspector.md

[channelInspector]: ../bot-service-channel-inspector.md


