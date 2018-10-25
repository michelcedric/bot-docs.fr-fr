---
title: Ajouter des actions suggérées aux messages | Microsoft Docs
description: Découvrez comment envoyer des actions suggérées au sein des messages à l’aide du Kit de développement logiciel (SDK) Bot Builder pour Node.js.
author: v-ducvo
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 06/06/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 1be5423de3859b32416b5acf9ba7b9d8e92ecb14
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2018
ms.locfileid: "49997246"
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

## <a name="suggested-actions-example"></a>Exemples d’actions suggérées

Pour ajouter des actions suggérées à un message, définissez la propriété `suggestedActions` du message sur une liste [d’actions de carte][ICardAction] qui représentent les boutons à présenter à l’utilisateur.

Cet exemple de code indique comment envoyer un message qui présente trois actions suggérées à l’utilisateur :

[!code-javascript[Send suggested actions](../includes/code/node-send-suggested-actions.js#sendSuggestedActions)]

Lorsque l’utilisateur appuie sur l’une des actions suggérées, le bot reçoit un message de l’utilisateur contenant la valeur `value` de l’action correspondante.

N’oubliez pas que la méthode `imBack` publiera la valeur `value` dans la fenêtre de discussion du canal que vous utilisez. Si ce comportement n’est pas souhaité, vous pouvez utiliser la méthode `postBack`, qui renvoie toujours la sélection à votre bot, mais sans l’afficher dans la fenêtre de discussion. Toutefois, certains canaux ne prennent pas en charge la méthode `postBack` ; dans ce cas, cette dernière se comportera comme la méthode `imBack`.

## <a name="additional-resources"></a>Ressources supplémentaires

* [Aperçu des fonctionnalités du bot avec l’inspecteur de canaux][inspector]
* [IMessage][IMessage]
* [ICardAction][ICardAction]
* [session.send][SessionSend]

[IMessage]: http://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.imessage

[SessionSend]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session.html#send

[ICardAction]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.icardaction.html

[inspector]: ../bot-service-channel-inspector.md

[channelInspector]: ../bot-service-channel-inspector.md
