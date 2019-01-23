---
title: Définir des étapes de conversation avec des cascades | Microsoft Docs
description: Découvrez comment utiliser des cascades pour définir les étapes d’une conversation avec le kit SDK Bot Framework pour Node.js.
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 526091d61f10ac0c241b994aa3ea99c1d2a70074
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/11/2019
ms.locfileid: "54225324"
---
# <a name="define-conversation-steps-with-waterfalls"></a>Définir des étapes de conversation avec des cascades

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

Une conversation se compose d’une série de messages échangés entre un utilisateur et un robot. Quand l’objectif du robot est de guider l’utilisateur dans une série d’étapes, vous pouvez utiliser une cascade pour définir les étapes de la conversation.

Une cascade est une implémentation spécifique d’un [dialogue](bot-builder-nodejs-dialog-overview.md), utilisée le plus souvent pour collecter des informations de l’utilisateur ou guider celui-ci dans une série de tâches. Les tâches sont implémentées en tant que suite de fonctions, où les résultats de la première fonction sont passés en tant qu’entrée à la fonction suivante, et ainsi de suite. Chaque fonction représente généralement une seule étape dans le processus global. À chaque étape, le robot invite l’utilisateur à effectuer une entrée, attend une réponse, puis passe le résultat à l’étape suivante.

Cet article vous aidera à comprendre le fonctionnement d’une cascade et comment l’utiliser pour [gérer le flux de conversation](bot-builder-nodejs-dialog-manage-conversation.md).

## <a name="conversation-steps"></a>Étapes d’une conversation

Une conversation implique généralement plusieurs échanges d’invites/réponses entre l’utilisateur et le robot. Chaque échange d’invite/réponse constitue une étape de la conversation. Vous pouvez créer une conversation à l’aide d’une cascade ne comportant pas plus de deux étapes.

Par exemple, considérez le dialogue `greetings` suivant : La première étape de la cascade demande le nom de l’utilisateur, et la deuxième utilise la réponse pour accueillir l’utilisateur en l’appelant pas son nom.

```javascript
bot.dialog('greetings', [
    // Step 1
    function (session) {
        builder.Prompts.text(session, 'Hi! What is your name?');
    },
    // Step 2
    function (session, results) {
        session.endDialog(`Hello ${results.response}!`);
    }
]);
```

Cela est rendu possible grâce à l’utilisation d’invites. Le kit SDK Bot Framework pour Node.js propose plusieurs types d’[invites](bot-builder-nodejs-dialog-prompt.md) intégrées que vous pouvez utiliser pour demander à l’utilisateur différents types d’informations.

L’exemple de code suivant montre un dialogue utilisant des invites pour recueillir divers éléments d’information de l’utilisateur au fil d’une cascade en 4 étapes.

```javascript
var inMemoryStorage = new builder.MemoryBotStorage();

// This is a dinner reservation bot that uses a waterfall technique to prompt users for input.
var bot = new builder.UniversalBot(connector, [
    function (session) {
        session.send("Welcome to the dinner reservation.");
        builder.Prompts.time(session, "Please provide a reservation date and time (e.g.: June 6th at 5pm)");
    },
    function (session, results) {
        session.dialogData.reservationDate = builder.EntityRecognizer.resolveTime([results.response]);
        builder.Prompts.text(session, "How many people are in your party?");
    },
    function (session, results) {
        session.dialogData.partySize = results.response;
        builder.Prompts.text(session, "Whose name will this reservation be under?");
    },
    function (session, results) {
        session.dialogData.reservationName = results.response;

        // Process request and display reservation details
        session.send(`Reservation confirmed. Reservation details: <br/>Date/Time: ${session.dialogData.reservationDate} <br/>Party size: ${session.dialogData.partySize} <br/>Reservation name: ${session.dialogData.reservationName}`);
        session.endDialog();
    }
]).set('storage', inMemoryStorage); // Register in-memory storage 
```

Dans cet exemple, le dialogue par défaut dispose de quatre fonctions, chacune représentant une étape de la cascade. Chaque étape invite l’utilisateur à effectuer une entrée, et envoie les résultats à l’étape suivante pour traitement. Ce processus se poursuit jusqu’à l’exécution de la dernière étape, ce qui a pour effet de confirme la réservation et de mettre fin au dialogue.

La capture d’écran ci-dessous illustre les résultats de l’exécution de ce robot dans l’[émulateur Bot Framework](~/bot-service-debug-emulator.md).

![Gérer un flux de conversation avec une cascade](~/media/bot-builder-nodejs-dialog-manage-conversation/waterfall-results.png)

## <a name="create-a-conversation-with-multiple-waterfalls"></a>Créer une conversation avec plusieurs cascades

Vous pouvez utiliser plusieurs cascades au sein d’une conversation pour définir toute structure de conversation requise par votre robot. Par exemple, vous pouvez utiliser une cascade pour gérer le flux de conversation, et des cascades supplémentaires pour recueillir des informations de l’utilisateur. Chaque cascade est encapsulée dans un dialogue et peut être invoquée en appelant la fonction `session.beginDialog`.

L’exemple de code suivant montre comment utiliser plusieurs cascades dans une conversation. La cascade dans le dialogue `greetings` gère le flux de la conversation, tandis que la cascade dans le dialogue `askName` recueille des informations de l’utilisateur.

```javascript
bot.dialog('greetings', [
    function (session) {
        session.beginDialog('askName');
    },
    function (session, results) {
        session.endDialog('Hello %s!', results.response);
    }
]);
bot.dialog('askName', [
    function (session) {
        builder.Prompts.text(session, 'Hi! What is your name?');
    },
    function (session, results) {
        session.endDialogWithResult(results);
    }
]);
```

## <a name="advance-the-waterfall"></a>Progression de la cascade

Une cascade progresse d’étape en étape dans la séquence des fonctions définies dans le tableau. La première fonction dans une cascade peut recevoir les arguments passés au dialogue. Toute fonction au sein d’une cascade peut utiliser la fonction `next` pour passer à l’étape suivante sans inviter l’utilisateur à effectuer d’entrée.

L’exemple de code suivant montre comment utiliser la fonction `next` à l’intérieur d’un dialogue qui guide l’utilisateur dans le processus de fourniture d’informations pour son profil. À chaque étape de la cascade, le robot invite l’utilisateur à fournir un élément d’information (si nécessaire), et la réponse de l’utilisateur (le cas échéant) est traitée par l’étape suivante de la cascade. L’étape finale de la cascade dans le dialogue `ensureProfile` met fin à ce dialogue, et retourne les informations de profil complètes au dialogue appelant qui envoie ensuite un message d’accueil personnalisé à l’utilisateur.

```javascript
var inMemoryStorage = new builder.MemoryBotStorage();

// This bot ensures user's profile is up to date.
var bot = new builder.UniversalBot(connector, [
    function (session) {
        session.beginDialog('ensureProfile', session.userData.profile);
    },
    function (session, results) {
        session.userData.profile = results.response; // Save user profile.
        session.send(`Hello ${session.userData.profile.name}! I love ${session.userData.profile.company}!`);
    }
]).set('storage', inMemoryStorage); // Register in-memory storage 

bot.dialog('ensureProfile', [
    function (session, args, next) {
        session.dialogData.profile = args || {}; // Set the profile or create the object.
        if (!session.dialogData.profile.name) {
            builder.Prompts.text(session, "What's your name?");
        } else {
            next(); // Skip if we already have this info.
        }
    },
    function (session, results, next) {
        if (results.response) {
            // Save user's name if we asked for it.
            session.dialogData.profile.name = results.response;
        }
        if (!session.dialogData.profile.company) {
            builder.Prompts.text(session, "What company do you work for?");
        } else {
            next(); // Skip if we already have this info.
        }
    },
    function (session, results) {
        if (results.response) {
            // Save company name if we asked for it.
            session.dialogData.profile.company = results.response;
        }
        session.endDialogWithResult({ response: session.dialogData.profile });
    }
]);
```

> [!NOTE]
> Cet exemple utilise deux conteneurs de données distincts pour stocker les données : `dialogData` et `userData`. Si le robot est distribué sur plusieurs nœuds de calcul, chaque étape de la cascade peut être traitée par un nœud différent. Pour plus d’informations sur le stockage des données du robot, voir [Gérer les données d’état](bot-builder-nodejs-state.md).

## <a name="end-a-waterfall"></a>Fin d’une cascade

Un dialogue créé en utilisant une cascade doit être terminé explicitement. Autrement, le robot répète la cascade indéfiniment. Vous pouvez terminer une cascade en utilisant l’une des méthodes suivantes :

* `session.endDialog` : utilisez cette méthode pour terminer la cascade s’il n’y a aucune donnée à retourner au dialogue appelant.

* `session.endDialogWithResult` : utilisez cette méthode pour terminer la cascade s’il y a des données à retourner au dialogue appelant. L’argument `response` retourné peut être un objet JSON ou tout type de données primitives JavaScript. Par exemple : 
  ```javascript
  session.endDialogWithResult({
    response: { name: session.dialogData.name, company: session.dialogData.company }
  });
  ```

* `session.endConversation` : utilisez cette méthode pour terminer la cascade si la fin de celle-ci représente la fin de la conversation.

Au lieu d’utiliser l’une de ces trois méthodes pour terminer une cascade, vous pouvez attacher le déclencheur `endConversationAction` au dialogue. Par exemple : 

```javascript
bot.dialog('dinnerOrder', [
    //...waterfall steps...,
    // Last step
    function(session, results){
        if(results.response){
            session.dialogData.room = results.response;
            var msg = `Thank you. Your order will be delivered to room #${session.dialogData.room}`;
            session.endConversation(msg);
        }
    }
])
.endConversationAction(
    "endOrderDinner", "Ok. Goodbye.",
    {
        matches: /^cancel$|^goodbye$/i,
        confirmPrompt: "This will cancel your order. Are you sure?"
    }
);
```

## <a name="next-steps"></a>Étapes suivantes

Une cascade vous permet de collecter des informations de l’utilisateur à l’aide d’*invites*. Examinons comment vous pouvez inviter l’utilisateur à effectuer une entrée.

> [!div class="nextstepaction"]
> [Inviter l’utilisateur à effectuer une entrée](bot-builder-nodejs-dialog-prompt.md)
