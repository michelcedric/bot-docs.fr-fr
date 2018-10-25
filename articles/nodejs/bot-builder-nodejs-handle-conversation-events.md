---
title: Gérer les événements utilisateur et de conversation  | Microsoft Docs
description: Découvrez comment gérer les événements tels qu’un utilisateur rejoignant une conversation à l’aide du Kit de développement logiciel (SDK) Bot Builder pour Node.js.
author: DucVo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 2e8cb9661079eafa988af4a7639e3e7dc1580bfa
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2018
ms.locfileid: "49998136"
---
# <a name="handle-user-and-conversation-events"></a>Gérer les événements utilisateur et de conversation

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

Cet article montre comment votre bot peut gérer des événements, par exemple, un utilisateur qui rejoint une conversation ou ajoute un bot à une liste de contacts, ou encore un bot qui dit **Goodbye** lorsqu’il est supprimé d’une conversation.


## <a name="greet-a-user-on-conversation-join"></a>Accueillir un utilisateur rejoignant une conversation
Le Bot Framework fournit l’événement [conversationUpdate][conversationUpdate] pour informer votre bot qu’un membre rejoint ou quitte une conversation. Un membre de la conversation peut être un utilisateur ou un bot.

L’extrait de code suivant permet au bot d’accueillir un ou plusieurs membres rejoignant une conversation ou de dire **Goodbye** s’il est supprimé de la conversation.

[!INCLUDE [conversationUpdate sample Node.js](../includes/snippet-code-node-conversationupdate-1.md)]

## <a name="acknowledge-add-to-contacts-list"></a>Notification d’ajout à la liste de contacts

L’événement [contactRelationUpdate][contactRelationUpdate] informe votre bot qu’un utilisateur vous a ajouté à sa liste de contacts.

[!INCLUDE [contactRelationUpdate sample Node.js](../includes/snippet-code-node-contactrelationupdate-1.md)]

## <a name="add-a-first-run-dialog"></a>Ajouter une boîte de dialogue pour la première exécution

Les événements **conversationUpdate** et **contactRelationUpdate** ne sont pas pris en charge par tous les canaux. Il existe cependant un moyen universel d’accueillir un utilisateur rejoignant une conversation, qui consiste à ajouter une boîte de dialogue pour la première exécution.

Dans l’exemple suivant, nous avons ajouté une fonction qui déclenche l’affichage la boîte de dialogue si l’utilisateur n’a pas été vu auparavant. Vous pouvez personnaliser le mode de déclenchement d’une action en fournissant un descripteur [onFindAction][onFindAction] pour votre action. 

[!INCLUDE [first-run sample Node.js](../includes/snippet-code-node-first-run-dialog-1.md)]

Vous pouvez également personnaliser ce que fait une action après son déclenchement en fournissant un descripteur [onSelectAction][onSelectAction]. Pour les actions de déclencheur, vous pouvez fournir un descripteur [onInterrupted][onInterrupted] pour intercepter une interruption avant qu’elle ne se produise. Pour plus d’informations, consultez l’article [Gérer les actions de l’utilisateur](bot-builder-nodejs-dialog-actions.md).

## <a name="additional-resources"></a>Ressources supplémentaires

* [conversationUpdate][conversationUpdate]
* [contactRelationUpdate][contactRelationUpdate]

[conversationUpdate]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.iconversationupdate.html
[contactRelationUpdate]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.icontactrelationupdate.html

[onFindAction]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.itriggeractionoptions#onfindaction
[onSelectAction]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.itriggeractionoptions#onselectaction
[onInterrupted]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.itriggeractionoptions#oninterrupted

[SendTyping]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session#sendtyping
[IMessage]: http://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.imessage
[ChatConnector]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.chatconnector.html
[session_userData]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session.html#userdata
