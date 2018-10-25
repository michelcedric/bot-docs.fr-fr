---
title: Gérer un flux de conversation avec des dialogues | Microsoft Docs
description: Découvrez comment gérer une conversation avec des dialogues entre un bot et un utilisateur dans le Kit de développement logiciel (SDK) Bot Builder pour Node.js.
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 133f085a857d1bb8bf7622e7adab19374902327d
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2018
ms.locfileid: "49997766"
---
# <a name="manage-conversation-flow-with-dialogs"></a>Gérer un flux de conversation avec des dialogues

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-manage-conversation-flow.md)
> - [Node.JS](../nodejs/bot-builder-nodejs-dialog-manage-conversation-flow.md)

La gestion de flux de conversation est une tâche essentielle dans la création de bots. Un bot doit être en mesure de s’acquitter des tâches principales en finesse et de traiter les interruptions de manière appropriée. Le Kit de développement logiciel (SDK) Bot Builder pour Node.js vous permet de gérer un flux de conversation avec des dialogues.

Un dialogue est comparable à une fonction dans un programme. Il est généralement conçu pour effectuer une opération spécifique, et peut être appelé aussi souvent que nécessaire. Vous pouvez enchaîner plusieurs dialogues afin de prendre en charge quasiment tous les flux de conversation que votre bot doit pouvoir traiter. Le Kit de développement logiciel (SDK) Bot Builder pour Node.js intègre différentes fonctionnalités, telles que des [invites](bot-builder-nodejs-dialog-prompt.md) et des [cascades](bot-builder-nodejs-dialog-waterfall.md), conçues pour vous aider à gérer un flux de conversation.

Cet article fournit une série d’exemples expliquant comment prendre en charge des flux de conversation simples ou complexes, dans lesquels votre bot peut traiter les interruptions et reprendre le flux correctement à l’aide de dialogues. Ces exemples reposent sur les scénarios suivants : 

1. Votre bot enregistre une réservation pour un dîner organisé.
2. Votre bot peut traiter la demande « Help » (Aide) à tout moment pendant la réservation.
3. Votre bot peut traiter une demande d’aide contextuelle pour l’étape actuelle de la réservation.
4. Votre bot peut gérer plusieurs sujets de conversation.

## <a name="manage-conversation-flow-with-a-waterfall"></a>Gérer un flux de conversation avec une cascade

Une [cascade](bot-builder-nodejs-dialog-waterfall.md) est un dialogue permettant au bot de guider facilement un utilisateur pour l’exécution d’un ensemble de tâches. Dans cet exemple, le bot de réservation pose une série de questions à l’utilisateur afin de traiter la demande de réservation. Le bot demande à l’utilisateur les informations suivantes :

1. Date et heure de la réservation
2. Nombre d’invités à la soirée
3. Nom de la personne qui effectue la réservation

L’exemple de code ci-après indique comment utiliser une cascade pour guider l’utilisateur tout au long d’une série d’invites.

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
        builder.Prompts.number(session, "How many people are in your party?");
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

La fonctionnalité principale de ce bot se déroule dans le dialogue par défaut. Le dialogue par défaut est défini lorsque le bot est créé : 

```javascript
var bot = new builder.UniversalBot(connector, [..waterfall steps..]); 
```

En outre, au cours de ce processus de création, vous pouvez définir le [stockage de données](bot-builder-nodejs-state.md) que vous souhaitez utiliser. Par exemple, pour utiliser le stockage en mémoire, vous pouvez définir le code suivant :

```javascript
var inMemoryStorage = new builder.MemoryBotStorage();
var bot = new builder.UniversalBot(connector, [..waterfall steps..]).set('storage', inMemoryStorage); // Register in-memory storage 
```

Le dialogue par défaut est créé sous la forme d’un tableau de fonctions qui définissent les différentes étapes de la cascade. Cet exemple comporte quatre fonctions, ce qui signifie que la cascade se compose de quatre étapes. Chaque étape exécute une tâche unique, et les résultats sont traités à l’étape suivante. Ce processus se poursuit jusqu’à la dernière étape, au cours de laquelle la réservation est confirmée et où le dialogue prend fin.

La capture d’écran ci-après illustre les résultats de l’exécution de ce bot dans l’application [Bot Framework Emulator](../bot-service-debug-emulator.md) :

![Gestion d’un flux de conversation avec une cascade](../media/bot-builder-nodejs-dialog-manage-conversation/waterfall-results.png)

### <a name="prompt-user-for-input"></a>Inviter l’utilisateur à saisir une entrée

Chaque étape de cet exemple utilise une invite demandant à l’utilisateur de saisir une entrée. Une invite est un type spécial de dialogue qui demande une entrée utilisateur, attend une réponse et renvoie cette dernière à l’étape suivante de la cascade. Pour plus d’informations sur les différents types d’invites que vous pouvez utiliser dans votre bot, consultez l’article [Demander aux utilisateurs d’effectuer une saisie](bot-builder-nodejs-dialog-prompt.md).

Dans cet exemple, le bot utilise l’élément `Prompts.text()` pour solliciter une réponse en forme libre de la part de l’utilisateur au format texte. L’utilisateur peut répondre par un texte quelconque, et le bot doit décider comment traiter la réponse. `Prompts.time()` utilise la bibliothèque [Chrono](https://github.com/wanasit/chrono) pour analyser les informations de date et d’heure d’une chaîne. Cette approche accroît la quantité de langage naturel en matière de date et d’heure que peut comprendre votre bot. Par exemple : « 6 octobre 2018 à 21:00 », «Aujourd’hui à 19:30 », « lundi prochain à 18:00 », etc.

> [!TIP] 
> L’heure entrée par l’utilisateur est convertie en heure UTC en fonction du fuseau horaire auquel appartient le serveur qui héberge le bot. Étant donné que le serveur peut se trouver dans un autre fuseau horaire que celui de l’utilisateur, veillez à tenir compte des fuseaux horaires. Pour convertir la date et l’heure en heure locale de l’utilisateur, prévoyez de demander à l’utilisateur le fuseau horaire auquel il appartient.

## <a name="manage-a-conversation-flow-with-multiple-dialogs"></a>Gérer un flux de conversation avec plusieurs dialogues

Une autre technique de gestion des flux de conversation consiste à combiner une cascade et plusieurs dialogues. La cascade vous permet de chaîner des fonctions dans un dialogue, tandis que les dialogues vous offrent la possibilité de décomposer une conversation en fonctionnalités plus petites qui sont réutilisables à tout moment.

Considérons l’exemple du bot qui enregistre la réservation pour le dîner organisé. L’exemple de code ci-après présente l’exemple précédent réécrit pour l’utilisation d’une cascade et de plusieurs dialogues.

```javascript
var inMemoryStorage = new builder.MemoryBotStorage();

// This is a dinner reservation bot that uses multiple dialogs to prompt users for input.
var bot = new builder.UniversalBot(connector, [
    function (session) {
        session.send("Welcome to the dinner reservation.");
        session.beginDialog('askForDateTime');
    },
    function (session, results) {
        session.dialogData.reservationDate = builder.EntityRecognizer.resolveTime([results.response]);
        session.beginDialog('askForPartySize');
    },
    function (session, results) {
        session.dialogData.partySize = results.response;
        session.beginDialog('askForReserverName');
    },
    function (session, results) {
        session.dialogData.reservationName = results.response;

        // Process request and display reservation details
        session.send(`Reservation confirmed. Reservation details: <br/>Date/Time: ${session.dialogData.reservationDate} <br/>Party size: ${session.dialogData.partySize} <br/>Reservation name: ${session.dialogData.reservationName}`);
        session.endDialog();
    }
]).set('storage', inMemoryStorage); // Register in-memory storage 

// Dialog to ask for a date and time
bot.dialog('askForDateTime', [
    function (session) {
        builder.Prompts.time(session, "Please provide a reservation date and time (e.g.: June 6th at 5pm)");
    },
    function (session, results) {
        session.endDialogWithResult(results);
    }
]);

// Dialog to ask for number of people in the party
bot.dialog('askForPartySize', [
    function (session) {
        builder.Prompts.text(session, "How many people are in your party?");
    },
    function (session, results) {
        session.endDialogWithResult(results);
    }
])

// Dialog to ask for the reservation name.
bot.dialog('askForReserverName', [
    function (session) {
        builder.Prompts.text(session, "Who's name will this reservation be under?");
    },
    function (session, results) {
        session.endDialogWithResult(results);
    }
]);
```

L’exécution de ce bot produit exactement les mêmes résultats que celle du bot précédent qui utilisait uniquement une cascade. Toutefois, sur le plan de la programmation, il existe deux différences principales :

1. Le dialogue par défaut est dédié à la gestion du flux de la conversation.
2. La tâche propre à chaque étape de la conversation est gérée par un dialogue distinct. Dans ce cas précis, le bot a besoin de trois informations ; il sollicite donc l’utilisateur à trois reprises par le biais d’une invite. Chaque invite figure désormais dans un dialogue qui lui est propre.

Cette technique vous permet de séparer le flux de conversion de la logique des tâches. Les dialogues sont ainsi réutilisables par différents flux de conversation si nécessaire. 

## <a name="respond-to-user-input"></a>Répondre à une entrée utilisateur

Pour le processus de guidage de l’utilisateur dans l’exécution d’une série de tâches, vous devez déterminer comment traiter les éventuelles questions ou demandes de complément d’informations émanant de l’utilisateur. Par exemple, quel que soit le stade de la conversation auquel se trouve l’utilisateur, comment le bot doit-il répondre si l’utilisateur entre les termes « Help » (Aide), « Support » (Support) ou « Cancel » (Annuler) ? Que doit-il se passer si l’utilisateur souhaite obtenir un complément d’informations au sujet d’une étape ? Et si l’utilisateur change d’avis et souhaite abandonner la tâche actuelle pour démarrer une tâche complètement différente ?

Le Kit de développement logiciel (SDK) Bot Builder pour Node.js permet à un bot d’écouter une entrée spécifique dans un contexte global ou local dans le cadre du dialogue actuel. Ces entrées sont appelées [actions](bot-builder-nodejs-dialog-actions.md) et permettent au bot d’écouter les entrées utilisateur basées sur une clause `matches`. La manière de réagir aux différentes entrées utilisateur est déterminée par le bot.

### <a name="handle-global-action"></a>Traiter une action globale

Si vous souhaitez autoriser votre bot à traiter les actions à n’importe quel stade d’une conversation, utilisez `triggerAction`. Les déclencheurs permettent au bot d’appeler un dialogue spécifique lorsque l’entrée correspond au terme spécifié. Par exemple, si vous souhaitez prendre en charge une option « Help » (Aide) globale, vous pouvez créer un dialogue d’aide et y attacher un élément `triggerAction` qui écoute l’entrée « Help » (Aide).

L’exemple de code ci-après indique comment attacher un élément `triggerAction` à un dialogue afin que ce dernier soit appelé lorsque l’utilisateur entre le terme « help » (aide).

```javascript
// The dialog stack is cleared and this dialog is invoked when the user enters 'help'.
bot.dialog('help', function (session, args, next) {
    session.endDialog("This is a bot that can help you make a dinner reservation. <br/>Please say 'next' to continue");
})
.triggerAction({
    matches: /^help$/i,
});
```

Par défaut, lorsqu’un élément `triggerAction` s’exécute, la pile de dialogues est effacée, et le dialogue déclenché devient le nouveau dialogue par défaut. Dans cet exemple, quand `triggerAction` s’exécute, la pile de dialogues est effacée, puis le dialogue `help` est ajouté à la pile en tant que nouveau dialogue par défaut. Si ce comportement n’est pas souhaité, vous pouvez ajouter l’option `onSelectAction` à l’élément `triggerAction`. L’option `onSelectAction` permet au bot de démarrer un nouveau dialogue sans effacer la pile de dialogues ; la conversation peut ainsi être redirigée temporairement, puis reprendre par la suite à l’endroit exact où elle s’était interrompue.

L’exemple de code ci-après indique comment utiliser l’option `onSelectAction` avec `triggerAction` afin que le dialogue `help` soit ajouté à la pile de dialogues existante (sans que la pile soit effacée).

```javascript
bot.dialog('help', function (session, args, next) {
    session.endDialog("This is a bot that can help you make a dinner reservation. <br/>Please say 'next' to continue");
})
.triggerAction({
    matches: /^help$/i,
    onSelectAction: (session, args, next) => {
        // Add the help dialog to the dialog stack 
        // (override the default behavior of replacing the stack)
        session.beginDialog(args.action, args);
    }
});
```

Dans cet exemple, le dialogue `help` prend le contrôle de la conversation lorsque l’utilisateur entre le terme « help » (aide). Étant donné que l’élément `triggerAction` contient l’option `onSelectAction`, le dialogue `help` est envoyé (push) vers la pile de dialogues existante sans que la pile soit effacée. Lorsque le dialogue `help` prend fin, il est supprimé de la pile de dialogues, et la conversation reprend à l’endroit où elle avait été interrompue par la commande `help`.

### <a name="handle-contextual-action"></a>Traiter une action contextuelle

Dans l’exemple précédent, le dialogue `help` est appelé si l’utilisateur entre le terme « help » (aide) à tout moment dans la conversation. Par conséquent, le dialogue ne peut offrir qu’une aide générale, car la demande d’aide de l’utilisateur n’est associée à aucun contexte spécifique. Comment faire pour que le bot réponde à une demande d’aide de l’utilisateur portant sur un point spécifique de la conversation ? Dans ce cas, le dialogue `help` doit être déclenché dans le contexte du dialogue actuel.

Considérons l’exemple du bot qui enregistre la réservation pour le dîner organisé. Que se passe-t-il si un utilisateur souhaite connaître la capacité maximale de la salle de réception lorsque le bot lui demande d’indiquer le nombre de participants à la soirée ? Pour prendre en charge ce scénario, vous pouvez attacher un élément `beginDialogAction` au dialogue `askForPartySize` écoutant l’entrée utilisateur « help » (aide).

L’exemple de code ci-après indique comment attacher une aide contextuelle à un dialogue à l’aide de l’élément `beginDialogAction`.

```javascript
// Dialog to ask for number of people in the party
bot.dialog('askForPartySize', [
    function (session) {
        builder.Prompts.text(session, "How many people are in your party?");
    },
    function (session, results) {
       session.endDialogWithResult(results);
    }
])
.beginDialogAction('partySizeHelpAction', 'partySizeHelp', { matches: /^help$/i });

// Context Help dialog for party size
bot.dialog('partySizeHelp', function(session, args, next) {
    var msg = "Party size help: Our restaurant can support party sizes up to 150 members.";
    session.endDialog(msg);
})
```

Dans cet exemple, chaque fois que l’utilisateur entre le terme « help » (aide), le bot pousse le dialogue `partySizeHelp` dans la pile. Ce dialogue envoie un message d’aide à l’utilisateur, puis prend fin en rendant le contrôle au dialogue `askForPartySize` qui réinvite l’utilisateur à entrer le nombre de participants.

Il est important de noter que cette aide contextuelle ne s’exécute que lorsque l’utilisateur se trouve dans le dialogue `askForPartySize`. Dans les autres cas, le message d’aide général généré par le déclencheur `triggerAction` s’exécute à la place. En d’autres termes, la clause `matches` locale est toujours prioritaire sur la clause `matches` globale. Par exemple, si un élément `beginDialogAction` rencontre la correspondance **help**, les correspondances **help** du déclencheur `triggerAction` ne s’exécuteront pas. Pour plus d’informations, consultez la section [Priorité d’action](bot-builder-nodejs-dialog-actions.md#action-precedence).

### <a name="change-the-topic-of-conversation"></a>Modifier le sujet de conversation

Par défaut, l’exécution d’un élément `triggerAction` efface la pile de dialogues et réinitialise la conversation, en commençant par le dialogue spécifié. Ce comportement est souvent préférable lorsque votre bot doit basculer d’un sujet de conversation vers un autre, par exemple si un utilisateur en train d’effectuer une réservation au restaurant décide plutôt de commander à dîner dans sa chambre d’hôtel. 

L’exemple ci-après repose sur l’exemple précédent, de sorte que le bot permet à l’utilisateur soit d’effectuer une réservation au restaurant, soit de se commander à dîner. Dans ce bot, le dialogue par défaut est un dialogue d’accueil qui propose deux options à l’utilisateur : `Dinner Reservation` et `Order Dinner`.

```javascript
var inMemoryStorage = new builder.MemoryBotStorage();

// This bot enables users to either make a dinner reservation or order dinner.
var bot = new builder.UniversalBot(connector, function(session){
    var msg = "Welcome to the reservation bot. Please say `Dinner Reservation` or `Order Dinner`";
    session.send(msg);
}).set('storage', inMemoryStorage); // Register in-memory storage 
```

La logique de réservation pour dîner qui figurait dans le dialogue par défaut de l’exemple précédent apparaît désormais dans son propre dialogue appelé `dinnerReservation`. Le flux du dialogue `dinnerReservation` reste identique à celui de la version à plusieurs dialogues précédemment décrite. La seule différence réside dans le fait qu’un élément `triggerAction` est attaché au dialogue. Notez que dans cette version, l’élément `confirmPrompt` invite l’utilisateur à confirmer le changement du sujet de conversation, avant d’appeler le nouveau dialogue. Il est recommandé d’inclure une invite `confirmPrompt` dans les scénarios de ce type, car une fois la pile de dialogues effacée, l’utilisateur est dirigé vers le nouveau sujet de conversation, tandis que le sujet en cours est abandonné.

```javascript
// This dialog helps the user make a dinner reservation.
bot.dialog('dinnerReservation', [
    function (session) {
        session.send("Welcome to the dinner reservation.");
        session.beginDialog('askForDateTime');
    },
    function (session, results) {
        session.dialogData.reservationDate = builder.EntityRecognizer.resolveTime([results.response]);
        session.beginDialog('askForPartySize');
    },
    function (session, results) {
        session.dialogData.partySize = results.response;
        session.beginDialog('askForReserverName');
    },
    function (session, results) {
        session.dialogData.reservationName = results.response;

        // Process request and display reservation details
        session.send(`Reservation confirmed. Reservation details: <br/>Date/Time: ${session.dialogData.reservationDate} <br/>Party size: ${session.dialogData.partySize} <br/>Reservation name: ${session.dialogData.reservationName}`);
        session.endDialog();
    }
])
.triggerAction({
    matches: /^dinner reservation$/i,
    confirmPrompt: "This will cancel your current request. Are you sure?"
});
```

Le second sujet de conversation est défini dans le dialogue `orderDinner` à l’aide d’une cascade. Ce dialogue affiche simplement le menu du dîner et invite l’utilisateur à entrer son numéro de chambre après avoir passé sa commande. Un déclencheur `triggerAction` est attaché au dialogue pour spécifier que ce dernier doit être appelé lorsque l’utilisateur entre « order dinner » (commander à dîner) et pour garantir l’invitation de l’utilisateur à confirmer son choix, au cas où il manifesterait le souhait de changer de sujet de conversation.

```javascript
// This dialog help the user order dinner to be delivered to their hotel room.
var dinnerMenu = {
    "Potato Salad - $5.99": {
        Description: "Potato Salad",
        Price: 5.99
    },
    "Tuna Sandwich - $6.89": {
        Description: "Tuna Sandwich",
        Price: 6.89
    },
    "Clam Chowder - $4.50":{
        Description: "Clam Chowder",
        Price: 4.50
    }
};

bot.dialog('orderDinner', [
    function(session){
        session.send("Lets order some dinner!");
        builder.Prompts.choice(session, "Dinner menu:", dinnerMenu);
    },
    function (session, results) {
        if (results.response) {
            var order = dinnerMenu[results.response.entity];
            var msg = `You ordered: ${order.Description} for a total of $${order.Price}.`;
            session.dialogData.order = order;
            session.send(msg);
            builder.Prompts.text(session, "What is your room number?");
        } 
    },
    function(session, results){
        if(results.response){
            session.dialogData.room = results.response;
            var msg = `Thank you. Your order will be delivered to room #${session.dialogData.room}`;
            session.endDialog(msg);
        }
    }
])
.triggerAction({
    matches: /^order dinner$/i,
    confirmPrompt: "This will cancel your order. Are you sure?"
});
```

Une fois que l’utilisateur a démarré une conversation et sélectionné `Dinner Reservation` ou `Order Dinner`, il peut changer d’avis à tout moment. Par exemple, si l’utilisateur est en train d’effectuer une réservation au restaurant, puis entre « order dinner » (commander à dîner), le bot confirme l’opération en indiquant : « This will cancel your current request. Are you sure? » (Ceci annulera votre demande précédente. Voulez-vous vraiment continuer ?) Si l’utilisateur entre la réponse « no » (non), la nouvelle demande est annulée, et l’utilisateur peut poursuivre le processus de réservation au restaurant. Si l’utilisateur entre « yes » (oui), le bot efface la pile de dialogues et transfère le contrôle de la conversation au dialogue `orderDinner`.

## <a name="end-conversation"></a>Mettre fin à la conversation

Dans les exemples ci-dessus, les dialogues sont fermés à l’aide des éléments `session.endDialog` ou `session.endDialogWithResult`, qui mettent fin au dialogue, suppriment ce dernier de la pile, puis rendent le contrôle au dialogue appelant. Dans les cas où l’utilisateur est arrivé à la fin de la conversation, vous devez utiliser `session.endConversation` pour indiquer que la conversation est terminée.

La méthode [`session.endConversation`](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session#endconversation) met fin à une conversation et envoie éventuellement un message à l’utilisateur. Par exemple, le dialogue `orderDinner` de l’exemple précédent pourrait mettre fin à la conversation à l’aide de la méthode `session.endConversation`, comme indiqué dans l’exemple de code suivant.

```javascript
bot.dialog('orderDinner', [
    //...waterfall steps...
    // Last step
    function(session, results){
        if(results.response){
            session.dialogData.room = results.response;
            var msg = `Thank you. Your order will be delivered to room #${session.dialogData.room}`;
            session.endConversation(msg);
        }
    }
]);
```

L’appel de `session.endConversation` met fin à la conversation en effaçant la pile de dialogues et en réinitialisant le stockage [`session.conversationData`](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session#conversationdata). Pour plus d’informations sur le stockage des données, consultez l’article [Gérer les données d’état](bot-builder-nodejs-state.md).

Il est logique d’appeler la méthode `session.endConversation` lorsque l’utilisateur achève le flux de conversation pour lequel le bot a été conçu. Vous pouvez également utiliser `session.endConversation` pour mettre fin à la conversation dans les cas où l’utilisateur entre les termes « cancel » (annuler) ou « goodbye » (au revoir) au milieu d’une conversation. Pour effectuer cette opération, il vous suffit d’attacher un déclencheur `endConversationAction` au dialogue et de faire en sorte que ce déclencheur écoute les entrées « cancel » (annuler) ou « goodbye » (au revoir).

L’exemple de code ci-après indique comment attacher un élément `endConversationAction` à un dialogue pour mettre fin à la conversation si l’utilisateur entre les termes « cancel » (annuler) ou « goodbye » (au revoir).

```javascript
bot.dialog('dinnerOrder', [
    //...waterfall steps...
])
.endConversationAction(
    "endOrderDinner", "Ok. Goodbye.",
    {
        matches: /^cancel$|^goodbye$/i,
        confirmPrompt: "This will cancel your order. Are you sure?"
    }
);
```

Étant donné que l’achèvement d’une conversation avec `session.endConversation` ou `endConversationAction` efface la pile de dialogues et contraint l’utilisateur à repartir de zéro, vous devez inclure une invite `confirmPrompt` pour vous assurer que l’utilisateur veut vraiment effectuer cette opération.

## <a name="next-steps"></a>Étapes suivantes

Cet article vous a expliqué comment gérer des conversations qui sont séquentielles par nature. Que se passe-t-il si vous souhaitez répéter un dialogue ou utiliser un modèle en boucle dans votre conversation ? Examinons la façon dont vous pouvez atteindre cet objectif en remplaçant des dialogues dans la pile.

> [!div class="nextstepaction"]
> [Remplacer des dialogues](bot-builder-nodejs-dialog-replace.md)

