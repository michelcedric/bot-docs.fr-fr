---
title: Utiliser un bouton pour fournir une entrée | Microsoft Docs
description: Découvrez comment envoyer des actions suggérées au sein des messages à l’aide du Kit SDK Bot Builder pour JavaScript.
keywords: actions suggérées, boutons, entrée supplémentaire
author: Kaiqb
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 11/08/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 10b9fa9664e8c18cdc5dcd2fcf3ae400296a4abb
ms.sourcegitcommit: 6c719b51c9e4e84f5642100a33fe346b21360e8a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/28/2018
ms.locfileid: "52451981"
---
# <a name="use-button-for-input"></a>Utiliser un bouton pour fournir une entrée

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Vous pouvez permettre à votre bot de proposer des boutons sur lesquels l’utilisateur peut appuyer afin de fournir une entrée. Les boutons améliorent l’expérience utilisateur en lui permettant de répondre à une question ou d’effectuer une sélection en appuyant simplement sur un bouton, plutôt que d’avoir à taper une réponse avec un clavier. Contrairement aux boutons qui apparaissent dans les cartes enrichies (qui restent visibles et accessibles à l’utilisateur même après être touchées), les boutons qui apparaissent dans les volets des actions suggérées disparaissent une fois que l’utilisateur effectue une sélection. Cela empêche que l’utilisateur appuie sur les boutons périmés lors d’une conversation et simplifie le développement du bot (dans la mesure où il est inutile de prendre en compte ce scénario). 

## <a name="suggest-action-using-button"></a>Suggérer l’action d’utiliser un bouton

Les *actions suggérées* permettent à votre bot de présenter des boutons. Vous pouvez créer une liste d’actions suggérées (également appelée « réponses rapides ») qui s’affichera à l’utilisateur une seule fois pendant la conversation : 

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Vous pouvez accéder au code source utilisé ici à partir de [GitHub](https://aka.ms/SuggestedActionsCSharp).

```csharp
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Schema;

var reply = turnContext.Activity.CreateReply("What is your favorite color?");

reply.SuggestedActions = new SuggestedActions()
{
    Actions = new List<CardAction>()
    {
        new CardAction() { Title = "Red", Type = ActionTypes.ImBack, Value = "Red" },
        new CardAction() { Title = "Yellow", Type = ActionTypes.ImBack, Value = "Yellow" },
        new CardAction() { Title = "Blue", Type = ActionTypes.ImBack, Value = "Blue" },
    },

};
await turnContext.SendActivityAsync(reply, cancellationToken);
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)
Vous pouvez accéder au code source utilisé ici à partir de [GitHub](https://aka.ms/SuggestActionsJS).

```javascript
const { ActivityTypes, MessageFactory, TurnContext } = require('botbuilder');

async sendSuggestedActions(turnContext) {
    var reply = MessageFactory.suggestedActions(['Red', 'Yellow', 'Blue'], 'What is the best color?');
    await turnContext.sendActivity(reply);
}
```

---

## <a name="additional-resources"></a>Ressources supplémentaires

Vous pouvez également accéder au code source complet affiché ici à partir de GitHub [[C#](https://aka.ms/SuggestedActionsCSharp) | [JS](https://aka.ms/SuggestActionsJS)].
