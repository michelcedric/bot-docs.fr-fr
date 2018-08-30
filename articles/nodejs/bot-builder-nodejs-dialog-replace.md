---
title: Remplacer des boîtes de dialogue | Microsoft Docs
description: Découvrez comment remplacer des boîtes de dialogue pour demander à effectuer une nouvelle entrée et gérer le flux de conversation à l’aide du Kit de développement logiciel (SDK) Bot Builder pour Node.js.
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 4343e9742d6bfabeab1ccb6a96f0d379d824125e
ms.sourcegitcommit: 2dc75701b169d822c9499e393439161bc87639d2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/24/2018
ms.locfileid: "42905733"
---
# <a name="replace-dialogs"></a>Remplacer des boîtes de dialogue

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

Il peut être très utile de pouvoir remplacer une boîte de dialogue si vous avez besoin de valider une entrée utilisateur ou de répéter une action au cours d’une conversation. Avec le Kit de développement logiciel (SDK) Bot Builder pour Node.js, vous pouvez remplacer une boîte de dialogue à l’aide de la méthode [`session.replaceDialog`](http://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session#replacedialog). Cette méthode vous permet de mettre fin à la boîte de dialogue en cours et de la remplacer par une nouvelle boîte de dialogue sans retour à l’appelant. 

## <a name="create-custom-prompts-to-validate-input"></a>Créer des invites personnalisées pour valider une entrée

Le Kit de développement logiciel (SDK) Bot Builder pour Node.js inclut la validation d’entrée pour certains types [d’invites](bot-builder-nodejs-dialog-prompt.md) comme `Prompts.time` et `Prompts.choice`. Pour valider une entrée de texte que vous recevez en réponse à `Prompts.text`, vous devez créer votre propre logique de validation et vos propres invites personnalisées. 

Par exemple, vous pouvez valider une entrée si celle-ci doit respecter une certaine valeur, un certain modèle, une certaine plage ou certains critères que vous définissez. Si la validation de l’entrée échoue, le bot peut inviter l’utilisateur à entrer de nouveau ces informations à l’aide de la méthode `session.replaceDialog`.

L’exemple de code suivant montre comment créer une invite personnalisée pour valider une entrée utilisateur, en l’occurrence un numéro de téléphone.

```javascript
// This dialog prompts the user for a phone number. 
// It will re-prompt the user if the input does not match a pattern for phone number.
bot.dialog('phonePrompt', [
    function (session, args) {
        if (args && args.reprompt) {
            builder.Prompts.text(session, "Enter the number using a format of either: '(555) 123-4567' or '555-123-4567' or '5551234567'")
        } else {
            builder.Prompts.text(session, "What's your phone number?");
        }
    },
    function (session, results) {
        var matched = results.response.match(/\d+/g);
        var number = matched ? matched.join('') : '';
        if (number.length == 10 || number.length == 11) {
            session.userData.phoneNumber = number; // Save the number.
            session.endDialogWithResult({ response: number });
        } else {
            // Repeat the dialog
            session.replaceDialog('phonePrompt', { reprompt: true });
        }
    }
]);
```

Dans cet exemple, l’utilisateur est initialement invité à fournir son numéro de téléphone. La logique de validation utilise une expression régulière mettant en correspondance l’entrée utilisateur et une plage de chiffres. Si l’entrée contient 10 ou 11 chiffres, ce nombre est retourné dans la réponse. Sinon, la méthode `session.replaceDialog` est exécutée pour afficher de nouveau la boîte de dialogue `phonePrompt` invitant l’utilisateur à renouveler sa saisie. Cette fois, cependant, elle inclut des instructions plus spécifiques concernant le format d’entrée attendu.

Lorsque vous appelez la méthode `session.replaceDialog`, vous spécifiez le nom de la boîte de dialogue à afficher de nouveau, ainsi qu’une liste d’arguments. Dans cet exemple, la liste d’arguments contient `{ reprompt: true }`, qui permet au bot de fournir différents messages d’invite selon qu’il s’agit d’une invite initiale ou d’une invite de validation. Vous pouvez cependant spécifier tous les arguments dont votre bot peut avoir besoin. 

## <a name="repeat-an-action"></a>Répéter une action

Dans certains cas, au cours d’une conversation, il peut être utile d’afficher de nouveau une boîte de dialogue pour permettre à l’utilisateur d’effectuer une certaine action plusieurs fois. Par exemple, si votre bot offre une variété de services, vous pouvez commencer par afficher le menu des services, guider l’utilisateur tout au long de la procédure de demande de service, puis afficher de nouveau le menu des services pour permettre à l’utilisateur de demander un autre service. Pour ce faire, vous pouvez utiliser la méthode [`session.replaceDialog`](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session#replacedialog) pour afficher de nouveau le menu des services au lieu de mettre fin à la conversation avec la méthode ['session.endConversation`](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session#endconversation). 

L’exemple suivant montre comment utiliser la méthode `session.replaceDialog` pour implémenter ce type de scénario. Tout d’abord, le menu des services est défini :

```javascript
// Main menu
var menuItems = { 
    "Order dinner": {
        item: "orderDinner"
    },
    "Dinner reservation": {
        item: "dinnerReservation"
    },
    "Schedule shuttle": {
        item: "scheduleShuttle"
    },
    "Request wake-up call": {
        item: "wakeupCall"
    },
}
```

La boîte de dialogue `mainMenu` est appelée par la boîte de dialogue par défaut. Le menu est donc présenté à l’utilisateur au début de la conversation. En outre, une [`triggerAction`](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.dialog.html#triggeraction) est jointe à boîte de dialogue `mainMenu`, de sorte que le menu s’affiche également dès lors que l’utilisateur entre « main menu » (menu principal). Lorsque le menu est présenté à l’utilisateur et que ce dernier sélectionne une option, la boîte de dialogue correspondant à la sélection de l’utilisateur est appelée à l’aide de la méthode `session.beginDialog`.

```javascript
var inMemoryStorage = new builder.MemoryBotStorage();

// This is a reservation bot that has a menu of offerings.
var bot = new builder.UniversalBot(connector, [
    function(session){
        session.send("Welcome to Contoso Hotel and Resort.");
        session.beginDialog("mainMenu");
    }
]).set('storage', inMemoryStorage); // Register in-memory storage 

// Display the main menu and start a new request depending on user input.
bot.dialog("mainMenu", [
    function(session){
        builder.Prompts.choice(session, "Main Menu:", menuItems);
    },
    function(session, results){
        if(results.response){
            session.beginDialog(menuItems[results.response.entity].item);
        }
    }
])
.triggerAction({
    // The user can request this at any time.
    // Once triggered, it clears the stack and prompts the main menu again.
    matches: /^main menu$/i,
    confirmPrompt: "This will cancel your request. Are you sure?"
});
```

Dans cet exemple, si l’utilisateur choisit l’option 1 pour se faire livrer un dîner dans sa chambre, la boîte de dialogue `orderDinner` est appelée et guide l’utilisateur tout au long du processus correspondant. À la fin du processus, le bot confirme la commande et affiche de nouveau le menu principal à l’aide de la méthode `session.replaceDialog`.

```javascript
// Menu: "Order dinner"
// This dialog allows user to order dinner to be delivered to their hotel room.
bot.dialog('orderDinner', [
    function(session){
        session.send("Lets order some dinner!");
        builder.Prompts.choice(session, "Dinner menu:", dinnerMenu);
    },
    function (session, results) {
        if (results.response) {
            var order = dinnerMenu[results.response.entity];
            var msg = `You ordered: %(Description)s for a total of $${order.Price}.`;
            session.dialogData.order = order;
            session.send(msg);
            builder.Prompts.text(session, "What is your room number?");
        } 
    },
    function(session, results){
        if(results.response){
            session.dialogData.room = results.response;
            var msg = `Thank you. Your order will be delivered to room #${results.response}.`;
            session.send(msg);
            session.replaceDialog("mainMenu"); // Display the menu again.
        }
    }
])
.reloadAction(
    "restartOrderDinner", "Ok. Let's start over.",
    {
        matches: /^start over$/i
    }
)
.cancelAction(
    "cancelOrder", "Type 'Main Menu' to continue.", 
    {
        matches: /^cancel$/i,
        confirmPrompt: "This will cancel your order. Are you sure?"
    }
);
```

Deux déclencheurs sont joints à la boîte de dialogue `orderDinner`, permettant à l’utilisateur de recommencer ou d’annuler la commande à tout moment pendant le processus de commande. 

Le premier déclencheur ([`reloadAction`](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.dialog.html#reloadaction)) permet à l’utilisateur de recommencer le processus de commande à l’aide de l’entrée « start over » (recommencer). Lorsque le déclencheur correspond à l’énoncé « start over », le `reloadAction` redémarre la boîte de dialogue à partir du début. 

Le deuxième déclencheur ([`cancelAction`](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.dialog.html#cancelaction)) permet à l’utilisateur d’interrompre complètement le processus de commande à l’aide de l’entrée « cancel » (annuler). Ce déclencheur n’affiche pas de nouveau le menu principal automatiquement. Il envoie un message indiquant à l’utilisateur la prochaine étape : « Type 'Main Menu' to continue » (Saisissez Main Menu pour continuer).

### <a name="dialog-loops"></a>Boucles de boîte de dialogue

Dans l’exemple ci-dessus, l’utilisateur peut sélectionner uniquement un élément par commande. Ainsi, si l’utilisateur souhaite commander deux éléments à partir du menu, il doit suivre l’ensemble du processus de commande pour le premier élément, puis reprendre tout le processus pour le deuxième élément. 

L’exemple suivant montre comment améliorer le bot précédent en refactorisant le menu du dîner dans une boîte de dialogue distincte. Cela permet au bot de répéter le menu du dîner dans une boucle. Par conséquent, l’utilisateur peut sélectionner plusieurs éléments au sein d’une seule commande.

Tout d’abord, ajoutez une option « Check out » (Paiement) au menu. Cette option permet à l’utilisateur de quitter le processus de sélection d’élément et d’accéder au processus de paiement.

```javascript
// The dinner menu
var dinnerMenu = { 
    //...other menu items...,
    "Check out": {
        Description: "Check out",
        Price: 0 // Order total. Updated as items are added to order.
    }
};
```

Ensuite, refactorisez l’invite de commande dans sa propre boîte de dialogue, de sorte que le bot puisse répéter le menu et permette à l’utilisateur d’ajouter plusieurs éléments à sa commande.

```javascript
// Add dinner items to the list by repeating this dialog until the user says `check out`. 
bot.dialog("addDinnerItem", [
    function(session, args){
        if(args && args.reprompt){
            session.send("What else would you like to have for dinner tonight?");
        }
        else{
            // New order
            // Using the conversationData to store the orders
            session.conversationData.orders = new Array();
            session.conversationData.orders.push({ 
                Description: "Check out",
                Price: 0
            })
        }
        builder.Prompts.choice(session, "Dinner menu:", dinnerMenu);
    },
    function(session, results){
        if(results.response){
            if(results.response.entity.match(/^check out$/i)){
                session.endDialog("Checking out...");
            }
            else {
                var order = dinnerMenu[results.response.entity];
                session.conversationData.orders[0].Price += order.Price; // Add to total.
                var msg = `You ordered: ${order.Description} for a total of $${order.Price}.`;
                session.send(msg);
                session.conversationData.orders.push(order);
                session.replaceDialog("addDinnerItem", { reprompt: true }); // Repeat dinner menu
            }
        }
    }
])
.reloadAction(
    "restartOrderDinner", "Ok. Let's start over.",
    {
        matches: /^start over$/i
    }
);
```

Dans cet exemple, les commandes sont stockées dans un magasin de données de bot dont l’étendue est définie selon la conversation actuelle, `session.conversationData.orders`. Pour chaque nouvelle commande, la variable est réinitialisée avec un nouveau tableau. De plus, le bot ajoute chaque élément choisi par l’utilisateur au tableau `orders` et ajoute le prix au montant total, qui est stocké dans la variable `Price` de la caisse. Lorsque l’utilisateur a sélectionné tous les éléments de sa commande, il peut dire « check out » (payer) et poursuivre le processus de commande.

> [!NOTE]
> Pour plus d’informations sur le stockage des données de bot, consultez [Gérer les données d’état](bot-builder-nodejs-state.md). 

Enfin, mettez à jour la deuxième étape de la cascade au sein de la boîte de dialogue `orderDinner` pour procéder à la confirmation de la commande. 

```javascript
// Menu: "Order dinner"
// This dialog allows user to order dinner and have it delivered to their room.
bot.dialog('orderDinner', [
    function(session){
        session.send("Lets order some dinner!");
        session.beginDialog("addDinnerItem");
    },
    function (session, results) {
        if (results.response) {
            // Display itemize order with price total.
            for(var i = 1; i < session.conversationData.orders.length; i++){
                session.send(`You ordered: ${session.conversationData.orders[i].Description} for a total of $${session.conversationData.orders[i].Price}.`);
            }
            session.send(`Your total is: $${session.conversationData.orders[0].Price}`);

            // Continue with the check out process.
            builder.Prompts.text(session, "What is your room number?");
        } 
    },
    function(session, results){
        if(results.response){
            session.dialogData.room = results.response;
            var msg = `Thank you. Your order will be delivered to room #${results.response}`;
            session.send(msg);
            session.replaceDialog("mainMenu");
        }
    }
])
//...attached triggers...
;
```

## <a name="cancel-a-dialog"></a>Annuler une boîte de dialogue

La méthode `session.replaceDialog` peut être utilisée pour remplacer la boîte de dialogue *actuelle* par une autre. Cependant, elle ne peut pas être utilisée pour remplacer une boîte de dialogue située en aval dans la pile de boîtes de dialogue. Pour remplacer une boîte de dialogue de la pile de boîtes de dialogue qui n’est pas la boîte de dialogue *actuelle*, utilisez plutôt la méthode [`session.cancelDialog`](http://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session#canceldialog). 

La méthode `session.cancelDialog` peut être utilisée pour mettre fin à une boîte de dialogue (quel que soit son emplacement dans la pile de boîtes de dialogue) et appeler éventuellement une nouvelle boîte de dialogue à la place. Pour appeler la méthode `session.cancelDialog`, spécifiez l’ID de la boîte de dialogue à annuler et, si vous le souhaitez, spécifiez l’ID de la boîte de dialogue à appeler à la place. Par exemple, cet extrait de code annule la boîte de dialogue `orderDinner` et la remplace par la boîte de dialogue `mainMenu` :

```javascript
session.cancelDialog('orderDinner', 'mainMenu'); 
```

Lorsque la méthode `session.cancelDialog` est appelée, une recherche inversée est effectuée dans la pile de boîtes de dialogue, et la première occurrence de cette boîte de dialogue est annulée. Elle est alors supprimée de la pile de boîtes de dialogue, tout comme les boîtes de dialogue enfants (le cas échéant). Le contrôle revient ensuite à la boîte de dialogue appelante, qui peut vérifier la présence d’un code [`results.resumed`](http://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.ipromptresult.html#resumed) équivalent à [`ResumeReason.notCompleted`](http://docs.botframework.com/en-us/node/builder/chat-reference/enums/_botbuilder_d_.resumereason.html#notcompleted) pour détecter l’annulation.

Au lieu de spécifier l’ID de la boîte de dialogue à annuler lorsque vous appelez la méthode `session.cancelDialog`, vous pouvez également spécifier l’index de base zéro de la boîte de dialogue à annuler, où l’index représente la position de la boîte de dialogue dans la pile de boîtes de dialogue. Par exemple, l’extrait de code suivant met fin à la boîte de dialogue active (index = 0) et démarre la boîte de dialogue `mainMenu` à la place. La boîte de dialogue `mainMenu` est appelée à la position 0 de la pile de boîtes de dialogue, devenant ainsi la nouvelle boîte de dialogue par défaut.

```javascript
session.cancelDialog(0, 'mainMenu');
```

Prenons l’exemple utilisé pour les [boucles de boîte de dialogue](#dialog-loops) plus haut. Lorsque l’utilisateur atteint le menu de sélection d’élément, cette boîte de dialogue (`addDinnerItem`) est la quatrième dans la pile de boîtes de dialogue : `[default dialog, mainMenu, orderDinner, addDinnerItem]`. Comment permettre à l’utilisateur d’annuler sa commande à partir de la boîte de dialogue `addDinnerItem` ? Si vous joignez un déclencheur `cancelAction` à la boîte de dialogue `addDinnerItem`, il renvoie uniquement l’utilisateur à la boîte de dialogue précédente (`orderDinner`), qui renvoie l’utilisateur à la boîte de dialogue `addDinnerItem`.

La méthode `session.cancelDialog` s’avère alors très utile. En reprenant [l’exemple des boucles de boîte de dialogue](#dialog-loops), ajoutez « Cancel order » (Annuler la commande) en tant qu’option explicite du menu du dîner.

```javascript
// The dinner menu
var dinnerMenu = { 
    //...other menu items...,
    "Check out": {
        Description: "Check out",
        Price: 0      // Order total. Updated as items are added to order.
    },
    "Cancel order": { // Cancel the order and back to Main Menu
        Description: "Cancel order",
        Price: 0
    }
};
```

Ensuite, mettez à jour la boîte de dialogue `addDinnerItem` pour vérifier la présence d’une requête « cancel order ». Si « cancel » est détecté, utilisez la méthode `session.cancelDialog` pour annuler la boîte de dialogue par défaut (c’est-à-dire la boîte de dialogue à l’index 0 de la pile) et appeler la boîte de dialogue `mainMenu` à la place. 

```javascript
// Add dinner items to the list by repeating this dialog until the user says `check out`. 
bot.dialog("addDinnerItem", [
    //...waterfall steps...,
    // Last step
    function(session, results){
        if(results.response){
            if(results.response.entity.match(/^check out$/i)){
                session.endDialog("Checking out...");
            }
            else if(results.response.entity.match(/^cancel/i)){
                // Cancel the order and start "mainMenu" dialog.
                session.cancelDialog(0, "mainMenu");
            }
            else {
                //...add item to list and prompt again...
                session.replaceDialog("addDinnerItem", { reprompt: true }); // Repeat dinner menu.
            }
        }
    }
])
//...attached triggers...
;
```

En utilisant la méthode `session.cancelDialog` de cette façon, vous pouvez implémenter n’importe quel flux de conversation requis par votre bot.

## <a name="next-steps"></a>Étapes suivantes

Comme vous pouvez le constater, vous pouvez utiliser différents types **d’actions** pour remplacer des boîtes de dialogue de la pile. Ces actions vous permettent de gérer le flux de conversation avec une grande flexibilité. Examinons les **actions** plus en détail, et découvrons comment mieux gérer les actions de l’utilisateur.

> [!div class="nextstepaction"]
> [Gérer les actions de l’utilisateur](bot-builder-nodejs-dialog-actions.md)
