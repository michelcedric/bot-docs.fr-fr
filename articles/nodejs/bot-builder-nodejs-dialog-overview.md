---
title: Vue d’ensemble des dialogues | Microsoft Docs
description: Découvrez comment utiliser les dialogues dans le kit SDK Bot Framework pour Node.js pour modéliser les conversations et gérer le flux de conversation.
author: DucVo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: dfa52914b3f0a2e81f4ff3a2f90c7404bfe53d4a
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/11/2019
ms.locfileid: "54225994"
---
# <a name="dialogs-in-the-bot-framework-sdk-for-nodejs"></a>Dialogues dans le kit SDK Bot Framework pour Node.js

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-dialogs.md)
> - [Node.JS](../nodejs/bot-builder-nodejs-dialog-overview.md)

Dans le kit SDK Bot Framework pour Node.js, les dialogues permettent de modéliser les conversations et de gérer le flux de conversation. Un robot communique avec un utilisateur par le biais de conversations. Les conversations sont organisées en dialogues. Les dialogues peuvent contenir des étapes en cascade et des invites. Lorsque l’utilisateur interagit avec le robot, celui-ci se lance, s’arrête et passe d’un dialogue à un autre pour répondre aux messages de l’utilisateur. Pour concevoir et créer des robots d’excellente qualité, il est essentiel de bien comprendre le fonctionnement des dialogues. 

Cet article présente les concepts liés aux dialogues. Après avoir lu cet article, suivez les liens qui se trouvent dans la section [Étapes suivantes](#next-steps) afin de mieux comprendre ces concepts.

## <a name="conversations-through-dialogs"></a>Conversations par le biais de dialogues

Le kit SDK Bot Framework pour Node.js définit une conversation comme étant la communication entre un bot et un utilisateur par le biais d’un ou de plusieurs dialogues. Au niveau le plus élémentaire, un dialogue est un module réutilisable qui effectue une opération ou recueille des informations auprès d’un utilisateur. Il est possible d’encapsuler la logique complexe du robot dans un code de dialogue réutilisable.

Il existe plusieurs façons de structurer et de modifier une conversation :

- Elle peut provenir de votre [dialogue par défaut](#default-dialog).
- Elle peut être redirigée d’un dialogue à un autre.
- Elle peut être poursuivie.
- Elle peut suivre un modèle [en cascade](bot-builder-nodejs-dialog-waterfall.md) qui guide l’utilisateur au fil d’une série d’étapes ou poser une série de questions à l’utilisateur via des [invites](bot-builder-nodejs-dialog-prompt.md).
- Elle peut faire appel à des [actions](bot-builder-nodejs-dialog-actions.md) prévues pour détecter des mots ou des phrases qui déclenchent un autre dialogue. 

Une conversation peut être vue comme un parent de dialogues. Une conversation contient donc une *pile de dialogues* et conserve son propre ensemble de données d’état, à savoir `conversationData` et `privateConversationData`. Par ailleurs, un dialogue conserve `dialogData`. Pour plus d’informations sur les données d’état, consultez [Gérer les données d’état](bot-builder-nodejs-state.md).

## <a name="dialog-stack"></a>Pile de dialogues

Un robot interagit avec un utilisateur par le biais d’une série de dialogues conservés dans une pile de dialogues. Les dialogues sont poussés dans la pile et en sortent au fil des conversations. La pile fonctionne comme une pile LIFO normale : le dernier dialogue ajouté est le premier à être exécuté. Lorsqu’un dialogue se termine, le contrôle revient au dialogue précédent dans la pile.

Lorsqu’une conversation de robot se lance en premier ou qu’une conversation se termine, la pile de dialogues est vide. À ce moment-là, le robot répond avec le *dialogue par défaut* lorsque l’utilisateur lui envoie un message.

## <a name="default-dialog"></a>Dialogue par défaut

Dans les versions de Bot Framework antérieures à la version 3.5, un dialogue *racine* est défini en ajoutant le dialogue `/` avec des conventions d’affectation de noms similaires à celles des URL. Cette convention n’est pas adaptée à l’affectation de noms des dialogues. 

> [!NOTE]
> Depuis la version 3.5 du Bot Framework, le dialogue *par défaut* est enregistré comme second paramètre dans le constructeur de [`UniversalBot`](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.universalbot.html#constructor).  

L’extrait de code suivant montre comment définir le dialogue par défaut lors de la création de l’objet `UniversalBot`.

```javascript
var bot = new builder.UniversalBot(connector, [
    //...Default dialog waterfall steps...
    ]);
```

Le dialogue par défaut s’exécute lorsque la pile de dialogues est vide et qu’aucun autre dialogue n’est [déclenché](bot-builder-nodejs-dialog-actions.md) via LUIS ou un autre [module de reconnaissance](bot-builder-nodejs-recognize-intent-messages.md). Comme le dialogue par défaut est la première réponse du robot à l’utilisateur, il doit lui fournir des informations contextuelles, par exemple une liste des commandes disponibles ou une vue d’ensemble de ce que le robot peut effectuer.

## <a name="dialog-handlers"></a>Gestionnaires de dialogues

Le gestionnaire de dialogues gère le déroulement des conversations. Pour avancer dans une conversation, le gestionnaire de dialogues oriente le flux en commençant et en terminant les dialogues. 

## <a name="starting-and-ending-dialogs"></a>Ouverture et fermeture des dialogues

Pour ouvrir un nouveau dialogue (et le pousser dans la pile), utilisez [`session.beginDialog()`](http://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session#begindialog). Pour fermer un dialogue (et le supprimer de la pile en renvoyant le contrôle au dialogue en appel), utilisez [`session.endDialog()`](http://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session#enddialog) ou [`session.endDialogWithResult()`](http://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session#enddialogwithresult). 

## <a name="using-waterfalls-and-prompts"></a>Utilisation des cascades et des invites

La [cascade](bot-builder-nodejs-dialog-waterfall.md) est un moyen simple de modéliser et de gérer le flux de conversation. Une cascade comprend une succession d’étapes. À chaque étape, vous pouvez effectuer une action au nom de l’utilisateur ou présenter des [invites](bot-builder-nodejs-dialog-prompt.md) à l’utilisateur pour obtenir des informations.

Une cascade est implémentée à l’aide d’un dialogue composé d’une collection de fonctions. Chaque fonction définit une étape de la cascade. L’exemple de code suivant montre une conversation simple qui utilise une cascade à deux étapes pour demander le nom de l’utilisateur et le saluer par son nom.

```javascript
// Ask the user for their name and greet them by name.
bot.dialog('greetings', [
    function (session) {
        builder.Prompts.text(session, 'Hi! What is your name?');
    },
    function (session, results) {
        session.endDialog(`Hello ${results.response}!`);
    }
]);
```

Lorsqu’un robot arrive à la fin de la cascade sans terminer le dialogue, le prochain message de l’utilisateur redémarre le dialogue à la première étape de la cascade. Il peut ainsi arriver que l’utilisateur ressente de la frustration, car il peut avoir l’impression d’être bloqué dans une boucle. Pour éviter ce genre de situation, lorsqu’une conversation ou un dialogue se termine, il est préférable d’appeler `endDialog`, `endDialogWithResult` ou `endConversation` de façon explicite.

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir davantage sur les dialogues, il est important de bien comprendre comment fonctionne le modèle de cascade et comment s’en servir pour guider les utilisateurs tout au long d’un processus.

> [!div class="nextstepaction"]
> [Définir les étapes de la conversation avec une cascade](bot-builder-nodejs-dialog-waterfall.md)
