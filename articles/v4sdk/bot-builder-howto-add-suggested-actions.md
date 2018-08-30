---
title: Ajouter des actions suggérées aux messages | Microsoft Docs
description: Découvrez comment envoyer des actions suggérées au sein des messages à l’aide du Kit SDK Bot Builder pour JavaScript.
keywords: actions suggérées, boutons, entrée supplémentaire
author: Kaiqb
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 03/13/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: b90b3c2635121f7a12f7766852990addb314542e
ms.sourcegitcommit: f89ed979eb6321232fb21100ef376d9b0d5113c9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/24/2018
ms.locfileid: "42914609"
---
# <a name="add-suggested-actions-to-messages"></a>Ajouter des actions suggérées aux messages

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

[!include[Introduction to suggested actions](../includes/snippet-suggested-actions-intro.md)] 

## <a name="send-suggested-actions"></a>Envoyer des actions suggérées

Vous pouvez créer une liste d’actions suggérées (également appelée « réponses rapides ») qui s’affichera à l’utilisateur une seule fois pendant la conversation :

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Schema;

// Create the activity and add suggested actions.
var activity = MessageFactory.SuggestedActions(
    new CardAction[]
    {
        new CardAction(title: "red", type: ActionTypes.ImBack, value: "red"),
        new CardAction( title: "green", type: ActionTypes.ImBack, value: "green"),
        new CardAction(title: "blue", type: ActionTypes.ImBack, value: "blue")
    }, text: "Choose a color");

// Send the activity as a reply to the user.
await context.SendActivity(activity);
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// Require MessageFactory from botbuilder.
const {MessageFactory} = require('botbuilder');

//  Initialize the message object.
const basicMessage = MessageFactory.suggestedActions(['red', 'green', 'blue'], 'Choose a color');

await context.sendActivity(basicMessage);
```

---

## <a name="additional-resources"></a>Ressources supplémentaires

Pour afficher un aperçu des fonctionnalités, vous pouvez utiliser l’[inspecteur de canaux](../bot-service-channel-inspector.md)

