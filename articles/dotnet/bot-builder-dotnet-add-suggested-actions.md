---
title: Ajouter des actions suggérées aux messages | Microsoft Docs
description: Découvrez comment ajouter des actions suggérées aux messages à l’aide du Kit de développement logiciel Bot Builder pour .NET.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 03/13/2018
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 63e5f4b8ac86b6b0e29d09796dbe74295bf3e213
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39299012"
---
# <a name="add-suggested-actions-to-messages"></a>Ajouter des actions suggérées aux messages
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

Lorsque l’utilisateur appuie sur l’une des actions suggérées, le bot reçoit un message de l’utilisateur contenant la valeur `Value` de l’action correspondante.

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


