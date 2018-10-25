---
title: Gérer les actions de l’utilisateur | Microsoft Docs
description: Découvrez comment gérer les actions de l’utilisateur en rendant votre bot capable d’écouter et de gérer les entrées utilisateur qui contiennent certains mots clés, à l’aide du kit SDK Bot Builder pour Node.js.
author: DucVo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 26f6e9520fe5d2ebb83ceb4e6a497a35e9d2611f
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2018
ms.locfileid: "49999256"
---
# <a name="handle-user-actions"></a>Gérer les actions de l’utilisateur

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-global-handlers.md)
> - [Node.JS](../nodejs/bot-builder-nodejs-dialog-actions.md)

Les utilisateurs tentent souvent d’accéder à certaines fonctionnalités dans un bot en utilisant des mots clés, tels que « aider », « annuler » ou « recommencer ». Ils agissent de cette façon au milieu d’une conversation alors que le bot attend une réponse différente. En implémentant des **actions**, vous pouvez concevoir votre bot pour qu’il gère de telles demandes de façon plus appropriée. Les gestionnaires examinent l’entrée utilisateur par rapport aux mots clés que vous spécifiez, comme « aider », « annuler » ou « recommencer », et réagissent en conséquence. 

![manière dont les utilisateurs s’expriment](../media/designing-bots/capabilities/trigger-actions.png)

## <a name="action-types"></a>Types d’actions

Les types d’actions que vous pouvez attacher à un dialogue sont répertoriés dans le tableau ci-dessous. Le lien associé à chaque nom d’action vous dirige vers une section fournissant plus de détails sur l’action en question.

| Action | Étendue | Description |
|------|------| ---- |
| [triggerAction](#bind-a-triggeraction) | Globale | Lie une action au dialogue, qui efface la pile de dialogues et se place (push) d’elle-même au bas de la pile. Utilisez l’option `onSelectAction` permettant de remplacer ce comportement par défaut. |
| [customAction](#bind-a-customaction) | Globale | Lie une action personnalisée au bot, qui peut traiter les informations ou prendre des mesures sans affecter la pile de dialogues. Utilisez l’option `onSelectAction` pour personnaliser les fonctionnalités de cette action. |
[beginDialogAction](#bind-a-begindialogaction) | Contextuelle | Lie une action au dialogue, qui démarre un autre dialogue quand elle est déclenchée. Le dialogue de départ est poussé dans la pile, et retiré dès qu’il est terminé. |
[reloadAction](#bind-a-reloadaction) | Contextuelle | Lie une action au dialogue, qui provoque le rechargement de ce dialogue quand elle est déclenchée. Vous pouvez utiliser `reloadAction` pour gérer des énoncés d’utilisateur, tels que « recommencer ». |
[cancelAction](#bind-a-cancelaction) | Contextuelle | Lie une action au dialogue, qui annule ce dialogue quand elle est déclenchée. Vous pouvez utiliser `cancelAction` pour gérer des énoncés d’utilisateur, tels que « annuler » ou « tant pis ». |
[endConversationAction](#bind-an-endconversationaction) | Contextuelle | Lie une action au dialogue, qui termine la conversation avec l’utilisateur lorsqu’elle est déclenchée. Vous pouvez utiliser `endConversationAction` pour gérer des énoncés d’utilisateur, tels que « au revoir ». |

## <a name="action-precedence"></a>Priorité d’action 

Lorsqu’un bot reçoit un énoncé d’utilisateur, il vérifie cet énoncé par rapport à toutes les actions inscrites qui se trouvent dans la pile de dialogues. La mise en correspondance commence en haut de la pile et se termine au bas de celle-ci. Si rien ne correspond, il vérifie ensuite l’énoncé par rapport à l’option `matches` de toutes les actions globales.

La priorité d’action est importante dans les cas où vous utilisez la même commande dans des contextes différents. Par exemple, vous pouvez utiliser la commande « Aide » pour le bot, en tant qu’aide générale. Vous pouvez aussi utiliser « Aide » pour chacune des tâches, ces commandes d’aide étant contextuelles à chaque tâche. Pour un exemple fonctionnel qui approfondit ce point, consultez [Répondre à une entrée utilisateur](bot-builder-nodejs-dialog-manage-conversation-flow.md#respond-to-user-input).

## <a name="bind-actions-to-dialog"></a>Lier des actions à un dialogue

Les énoncés ou les clics de bouton de l’utilisateur peuvent *déclencher* une action qui est associée à un [dialogue](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.dialog.html).
Si la valeur de *matches* est spécifiée, l’action doit écouter l’utilisateur prononcer un mot ou une expression qui déclenche l’action.  L’option `matches` peut prendre une expression régulière ou le nom d’un [module de reconnaissance][RecognizeIntent].
Pour lier l’action à un clic de bouton, utilisez [CardAction.dialogAction()][CardAction] pour déclencher l’action.

Les actions sont *chainable*, ce qui vous permet de lier à un dialogue autant d’actions que vous le souhaitez.

### <a name="bind-a-triggeraction"></a>Lier TriggerAction

Pour lier [triggerAction][triggerAction] à un dialogue, procédez comme suit :

```javascript
// Order dinner.
bot.dialog('orderDinner', [
    //...waterfall steps...
])
// Once triggered, will clear the dialog stack and pushes
// the 'orderDinner' dialog onto the bottom of stack.
.triggerAction({
    matches: /^order dinner$/i
});
```

Lier `triggerAction` à un dialogue l’inscrit pour le bot. Une fois déclenchée, `triggerAction` efface la pile de dialogues et pousse le dialogue déclenché dans la pile. Cette action est idéale pour changer de [sujet de conversation](bot-builder-nodejs-dialog-manage-conversation-flow.md#change-the-topic-of-conversation), ou pour permettre aux utilisateurs de demander des tâches autonomes arbitraires. Si vous souhaitez remplacer le comportement, dans lequel cette action efface la pile de dialogues, ajoutez une option `onSelectAction` à `triggerAction`.

L’extrait de code ci-dessous montre comment fournir une aide générale à partir d’un contexte global, sans effacer la pile de dialogues.

```javascript
bot.dialog('help', function (session, args, next) {
    //Send a help message
    session.endDialog("Global help menu.");
})
// Once triggered, will start a new dialog as specified by
// the 'onSelectAction' option.
.triggerAction({
    matches: /^help$/i,
    onSelectAction: (session, args, next) => {
        // Add the help dialog to the top of the dialog stack 
        // (override the default behavior of replacing the stack)
        session.beginDialog(args.action, args);
    }
});
```

Dans ce cas, l’action `triggerAction` est attachée au dialogue `help` lui-même (plutôt qu’au dialogue `orderDinner`). L’option `onSelectAction` vous permet de démarrer ce dialogue sans effacer la pile de dialogues. Vous pouvez ainsi gérer les demandes générales, telles que « aide », « à propos de », « support », etc. Notez que l’option `onSelectAction` appelle explicitement la méthode `session.beginDialog` pour démarrer le dialogue déclenché. L’ID du dialogue déclenché est fourni par le biais de `args.action`. Ne codez pas manuellement l’ID du dialogue (par exemple : 'help') dans cette méthode ou vous risquez des erreurs d’exécution. Si vous souhaitez déclencher un message d’aide contextuelle pour la tâche `orderDinner` elle-même, prévoyez plutôt d’attacher `beginDialogAction` au dialogue `orderDinner`.

### <a name="bind-a-customaction"></a>Lier customAction

Contrairement à d’autres types d’actions, `customAction` n’a pas d’action définie par défaut. C’est à vous de définir ce que cette action fait. L’avantage d’utiliser `customAction` réside dans la possibilité que vous avez de traiter les demandes d’utilisateur sans manipuler la pile de dialogues. Lors du déclenchement de `customAction`, l’option `onSelectAction` peut traiter la demande sans envoyer (push) de nouveaux dialogues dans la pile. Une fois l’action terminée, le contrôle est repassé au dialogue qui se trouve en haut de la pile, et le bot peut continuer.

Vous pouvez utiliser `customAction` pour fournir des demandes d’actions générales et rapides, telles que « Qu’elle est la température extérieure en ce moment ? », « Quelle heure est-il maintenant à Paris ? », « Rappelle-moi d’acheter du lait à 17 h 00 aujourd’hui. », etc. Il s’agit d’actions générales que le bot peut accomplir sans manipulation de la pile.

Une autre différence essentielle avec l’action `customAction` est qu’elle lie au bot et non à un dialogue.

L’exemple de code suivant montre comment lier `customAction` à `bot` qui est à l’écoute de demandes de définition d’un rappel.

```javascript
bot.customAction({
    matches: /remind|reminder/gi,
    onSelectAction: (session, args, next) => {
        // Set reminder...
        session.send("Reminder is set.");
    }
})
```

### <a name="bind-a-begindialogaction"></a>Lier beginDialogAction

Lier `beginDialogAction` à un dialogue inscrit l’action pour le dialogue. Cette méthode démarre un autre dialogue lorsqu’elle est déclenchée. Le comportement de cette action revient à appeler la méthode [beginDialog](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session.html#begindialog). Le nouveau dialogue est poussé en haut de la pile de dialogues, pour qu’il ne mette pas fin automatiquement à la tâche en cours. La tâche en cours se poursuit dès que le nouveau dialogue s’achève. 

L’extrait de code suivant montre comment lier [beginDialogAction][beginDialogAction] à un dialogue.

```javascript
// Order dinner.
bot.dialog('orderDinner', [
    //...waterfall steps...
])
// Once triggered, will start the 'showDinnerCart' dialog.
// Then, the waterfall will resumed from the step that was interrupted.
.beginDialogAction('showCartAction', 'showDinnerCart', {
    matches: /^show cart$/i
});

// Show dinner items in cart
bot.dialog('showDinnerCart', function(session){
    for(var i = 1; i < session.conversationData.orders.length; i++){
        session.send(`You ordered: ${session.conversationData.orders[i].Description} for a total of $${session.conversationData.orders[i].Price}.`);
    }

    // End this dialog
    session.endDialog(`Your total is: $${session.conversationData.orders[0].Price}`);
});
```

Dans les cas où vous devez passer des arguments supplémentaires dans le nouveau dialogue, vous pouvez ajouter une option [`dialogArgs`](https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.idialogactionoptions#dialogargs) à l’action.

En utilisant l’exemple ci-dessus, vous pouvez apporter des modifications pour accepter des arguments passés par le biais de `dialogArgs`.

```javascript
// Order dinner.
bot.dialog('orderDinner', [
    //...waterfall steps...
])
// Once triggered, will start the 'showDinnerCart' dialog.
// Then, the waterfall will resumed from the step that was interrupted.
.beginDialogAction('showCartAction', 'showDinnerCart', {
    matches: /^show cart$/i,
    dialogArgs: {
        showTotal: true;
    }
});

// Show dinner items in cart with the option to show total or not.
bot.dialog('showDinnerCart', function(session, args){
    for(var i = 1; i < session.conversationData.orders.length; i++){
        session.send(`You ordered: ${session.conversationData.orders[i].Description} for a total of $${session.conversationData.orders[i].Price}.`);
    }

    if(args && args.showTotal){
        // End this dialog with total.
        session.endDialog(`Your total is: $${session.conversationData.orders[0].Price}`);
    }
    else{
        session.endDialog(); // Ends without a message.
    }
});
```

### <a name="bind-a-reloadaction"></a>Lier reloadAction

Lier `reloadAction` à un dialogue l’inscrit pour le dialogue. La liaison de cette action à un dialogue entraîne le redémarrage du dialogue quand l’action est déclenchée. Le déclenchement de cette action revient à appeler la méthode [replaceDialog](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session.html#replacedialog). Cela s’avère utile pour implémenter une logique permettant de gérer des énoncés utilisateur, comme « recommencer », ou de créer des [boucles](bot-builder-nodejs-dialog-replace.md#repeat-an-action).

L’extrait de code suivant montre comment lier [reloadAction][reloadAction] à un dialogue.

```javascript
// Order dinner.
bot.dialog('orderDinner', [
    //...waterfall steps...
])
// Once triggered, will restart the dialog.
.reloadAction('startOver', 'Ok, starting over.', {
    matches: /^start over$/i
});
```

Dans les cas où vous devez passer des arguments supplémentaires dans le dialogue rechargé, vous pouvez ajouter une option [`dialogArgs`](https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.idialogactionoptions#dialogargs) à l’action. Cette option est passée dans le paramètre `args`. La réécriture de l’extrait de code ci-dessus, permettant de recevoir un argument sur une action de rechargement, doit ressembler à ceci :

```javascript
// Order dinner.
bot.dialog('orderDinner', [
    function(session, args, next){
        if(args && args.isReloaded){
            // Reload action was triggered.
        }

        session.send("Lets order some dinner!");
        builder.Prompts.choice(session, "Dinner menu:", dinnerMenu);
    }
    //...other waterfall steps...
])
// Once triggered, will restart the dialog.
.reloadAction('startOver', 'Ok, starting over.', {
    matches: /^start over$/i,
    dialogArgs: {
        isReloaded: true;
    }
});
```

### <a name="bind-a-cancelaction"></a>Lier cancelAction

Lier `cancelAction` l’inscrit pour le dialogue. Une fois déclenchée, cette action met fin brutalement au dialogue. Dès la fin du dialogue, le dialogue parent continue avec un code de reprise indiquant qu’il était `canceled`. Cette action vous permet de gérer des énoncés, tels que « tant pis » ou « annuler ». Si vous devez plutôt annuler par programmation un dialogue, consultez [Annuler un dialogue](bot-builder-nodejs-dialog-replace.md#cancel-a-dialog). Pour plus d’informations sur le *code de reprise*, consultez [Résultats des invites](bot-builder-nodejs-dialog-prompt.md#prompt-results). 

L’extrait de code suivant montre comment lier [cancelAction][cancelAction] à un dialogue.

```javascript
// Order dinner.
bot.dialog('orderDinner', [
    //...waterfall steps...
])
//Once triggered, will end the dialog.
.cancelAction('cancelAction', 'Ok, cancel order.', {
    matches: /^nevermind$|^cancel$|^cancel.*order/i
});
```

### <a name="bind-an-endconversationaction"></a>Lier endConversationAction

Lier `endConversationAction` l’inscrit pour le dialogue. Une fois déclenchée, cette action termine la conversation avec l’utilisateur. Le déclenchement de cette action revient à appeler la méthode [endConversation](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session.html#endconversation). Lorsqu’une conversation se termine, le kit SDK Bot Builder pour Node.js efface la pile de dialogues et les données d’état persistant. Pour plus d’informations sur les données d’état persistant, consultez [Gérer les données d’état](bot-builder-nodejs-state.md).

L’extrait de code suivant montre comment lier [endConversationAction][endConversationAction] à un dialogue.

```javascript
// Order dinner.
bot.dialog('orderDinner', [
    //...waterfall steps...
])
// Once triggered, will end the conversation.
.endConversationAction('endConversationAction', 'Ok, goodbye!', {
    matches: /^goodbye$/i
});
```

## <a name="confirm-interruptions"></a>Confirmer les interruptions

La majorité, si ce n’est la totalité de ces actions interrompent le flux normal d’une conversation. Bon nombre d’entre elles entraînent des perturbations et doivent donc être utilisées avec précaution. Par exemple, `triggerAction`, `cancelAction` ou `endConversationAction` efface la pile de dialogues. Si l’utilisateur a commis l’erreur de déclencher une de ces actions, il doit recommencer la tâche. Pour vous assurer que l’utilisateur a bien déclenché ces actions délibérément, vous pouvez ajouter une option `confirmPrompt` à ces actions. `confirmPrompt` demande à l’utilisateur s’il est sûr de vouloir annuler, ou de vouloir mettre fin à la tâche en cours. Elle permet à l’utilisateur de changer d’avis et de poursuivre le processus.

L’extrait de code ci-dessous montre une méthode [cancelAction][cancelAction] avec une propriété [confirmPrompt](http://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.itriggeractionoptions#confirmprompt) pour s’assurer que l’utilisateur souhaite vraiment annuler le processus de commande.

```javascript
// Order dinner.
bot.dialog('orderDinner', [
    //...waterfall steps...
])
// Confirm before triggering the action.
// Once triggered, will end the dialog. 
.cancelAction('cancelAction', 'Ok, cancel order.', {
    matches: /^nevermind$|^cancel$|^cancel.*order/i,
    confirmPrompt: "Are you sure?"
});
```

Lorsque cette action est déclenchée, elle demande à l’utilisateur : « Êtes-vous sûr ? » L’utilisateur doit répondre « Oui » pour poursuivre l’action, ou « Non » pour annuler l’action et reprendre là où il en était avant.

## <a name="next-steps"></a>Étapes suivantes

Les **actions** vous donnent la possibilité d’anticiper les demandes des utilisateurs, et permettent au bot de gérer ces demandes convenablement. La plupart de ces actions perturbent le flux de la conversation en cours. Si vous souhaitez donner aux utilisateurs la possibilité de sortir et de reprendre un flux de conversation, vous devez enregistrer l’état utilisateur avant l’extraction. Examinons de plus près l’enregistrement de l’état utilisateur et la gestion des données d’état.

> [!div class="nextstepaction"]
> [Gérer les données d’état](bot-builder-nodejs-state.md)


[triggerAction]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.dialog.html#triggeraction

[cancelAction]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.dialog.html#cancelaction

[reloadAction]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.dialog.html#reloadaction

[beginDialogAction]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.dialog.html#begindialogaction

[endConversationAction]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.dialog.html#endconversationaction

[RecognizeIntent]: bot-builder-nodejs-recognize-intent-messages.md

[CardAction]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.cardaction#dialogaction
