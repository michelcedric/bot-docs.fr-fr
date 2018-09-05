---
title: Envoyer des messages | Microsoft Docs
description: Découvrez comment envoyer des messages dans le Kit de développement logiciel Bot Builder.
keywords: envoi de messages, activités de message, message texte simple, discours, message parlé
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 08/23/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: b56ad56e19691cfbd7b39606832ed10fce951aa3
ms.sourcegitcommit: f89ed979eb6321232fb21100ef376d9b0d5113c9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/24/2018
ms.locfileid: "42914589"
---
# <a name="send-messages"></a>Envoyer des messages

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Pour communiquer avec les utilisateurs ainsi que pour recevoir des communications, votre bot aura pour moyen principal les activités de **message**. Certains messages peuvent simplement se composer de texte brut, tandis que d’autres peuvent contenir un contenu plus riche, comme les cartes ou les pièces jointes. Le gestionnaire de tours de votre robot reçoit des messages de l’utilisateur, et vous pouvez envoyer des réponses à l’utilisateur à ce moment. L’objet de [contexte de tour](bot-builder-concept-activity-processing.md#turn-context) fournit des méthodes pour renvoyer des messages à l’utilisateur. Pour plus d’informations sur le traitement de l’activité en général, consultez [Activity processing](bot-builder-concept-activity-processing.md) (traitement de l’activité).

Cet article décrit comment envoyer des messages texte et vocaux simples. Pour envoyer des contenus plus riches, voyez comment [ajouter des pièces jointes multimédias enrichies](bot-builder-howto-add-media-attachments.md). Pour plus d’informations sur la façon d’utiliser des objets d’invite, voyez comment [inviter les utilisateurs à entrer des données](bot-builder-prompts.md).

## <a name="send-a-simple-text-message"></a>Envoyer un message texte simple

Pour envoyer un message texte simple, spécifiez la chaîne que vous souhaitez envoyer en tant qu’activité :

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Dans la méthode **OnTurn** du bot, utilisez la méthode **SendActivity** de l’objet de contexte de tour pour envoyer une réponse de message unique. Vous pouvez également utiliser la méthode **SendActivities** de l’objet pour envoyer plusieurs réponses à la fois.

```cs
await context.SendActivity("Greetings from sample message.");
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Dans le gestionnaire de tour du bot, utilisez la méthode **SendActivity** de l’objet de contexte de tour pour envoyer une réponse de message unique. Vous pouvez également utiliser la méthode **SendActivities** de l’objet pour envoyer plusieurs réponses à la fois.

```javascript
await context.sendActivity("Greetings from sample message.");
```

---

## <a name="send-a-spoken-message"></a>Envoyer un message parlé

Certains canaux prennent en charge les robots de reconnaissance vocale, ce qui leur permet de parler à l’utilisateur. Le contenu d’un message peut être à la fois écrit et parlé.

> [!NOTE]
> Pour les canaux qui ne prennent pas en charge le discours, le contenu vocal est ignoré.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Utilisez le paramètre facultatif **speak** pour fournir le texte à énoncer dans le cadre de la réponse.

```cs
await context.SendActivity(
    "This is the text to be displayed.",
    "This is the text to be spoken.");
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Pour ajouter le discours, vous aurez besoin de `Microsoft.Bot.Builder.MessageFactory` afin de générer le message. `MessageFactory` est plutôt utilisé avec [les médias enrichis](bot-builder-howto-add-media-attachments.md). Vous trouverez un peu plus d’explications sur cette page, mais pour le moment nous allons simplement l’utiliser ici selon les besoins.

```javascript
// Require MessageFactory from botbuilder
const {MessageFactory} = require('botbuilder');

const basicMessage = MessageFactory.text('This is the text that will be displayed.', 'This is the text that will be spoken.');
await context.sendActivity(basicMessage);
```

---
