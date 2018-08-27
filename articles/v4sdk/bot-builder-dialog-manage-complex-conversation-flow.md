---
title: Gérer un flux de conversation complexe avec des dialogues | Microsoft Docs
description: Découvrez comment gérer un flux de conversation complexe avec des dialogues dans le Kit de développement logiciel (SDK) Bot Builder pour Node.js.
keywords: flux de conversation complexe, répétition, boucle, menu, dialogues, invites, cascades, jeu de dialogues
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 7/27/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 304de6783a268140bf74f95d96cd9c24e12c4c05
ms.sourcegitcommit: 1abc32353c20acd103e0383121db21b705e5eec3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/21/2018
ms.locfileid: "40236359"
---
# <a name="manage-complex-conversation-flows-with-dialogs"></a>Gérer des flux de conversation complexes avec dialogues

[!INCLUDE [pre-release-label](~/includes/pre-release-label.md)]

Vous pouvez gérer des flux de conversation simples et complexes avec la bibliothèque de dialogues. Dans des [flux de conversation simples](bot-builder-dialog-manage-conversation-flow.md), l’utilisateur commence à la première étape d’une *cascade*, continue jusqu’à la dernière étape où se termine l’échange conversationnel. Dans cet article, nous allons utiliser des dialogues pour gérer plus de conversations complexes avec des sections en branche ou en boucle. Pour ce faire, nous allons utiliser la méthode _replace_ du contexte de dialogue et passer des arguments entre différentes parties du dialogue.

<!-- TODO: We need a dialogs conceptual topic to link to, so we can reference that here, in place of describing what they are and what their features are in a how-to topic. -->

Pour vous offrir plus de contrôle sur la *pile de dialogue*, la bibliothèque **Dialogues** fournit une méthode _replace_. Cette méthode vous permet de faire sortir le dialogue de la pile, envoyer par push un nouveau dialogue sur le dessus de la pile et de le démarrer. Cette fonctionnalité vous permet d’offrir une conversation plus complexe. Ces techniques peuvent être utilisées pour créer des conversations complexes de façon arbitraire. Si la complexité de votre conversation augmente au point que les dialogues deviennent difficiles à gérer, il est possible de créer votre propre flux de logique de contrôle, pour pouvoir suivre la conversation de votre utilisateur.

<!-- TODO: This is probably a good place to add a link to the modular/composite dialogs topic. -->

Dans cet article, nous allons créer des dialogues pour un bot d’hôtel qu’un invité peut utiliser pour réserver une table ou commander un repas en service de chambre. Le dialogue de niveau supérieur offre à l’invité ces deux options. S’il souhaite réserver une table, le dialogue de niveau supérieur démarre un dialogue de réservation de table. S’il souhaite commander un repas, le dialogue de niveau supérieur démarre un dialogue de commande de repas. Le dialogue de commande de repas demande d’abord à l’invité de sélectionner les plats depuis un menu, puis son numéro de chambre. La sélection des plats est aussi un dialogue, permettant ainsi à l’invité de sélectionner plusieurs éléments avant de procéder à la commande.

Ce diagramme illustre les dialogues que nous allons créer dans cet article, et les relations qu’ils entretiennent entre eux.

![Illustration des dialogues utilisés dans cet article](~/media/complex-conversation-flows.png)

## <a name="how-to-branch"></a>Créer une branche

Le contexte du dialogue maintient une _pile de dialogue_, et suit l’étape suivante de chaque dialogue sur la pile. Sa méthode _begin_ envoie par push un dialogue sur le dessus de la pile, et sa méthode _end_ fait sortir le dialogue du dessus de la pile.

Un dialogue peut démarrer un nouveau dialogue (ou créer une branche) au sein du même jeu de dialogue en appelant la méthode _begin_ du contexte et en fournissant l’ID du nouveau dialogue. Le dialogue original est toujours sur la pile, mais les appels de la méthode _continue_ du contexte de dialogue ne sont envoyés qu’au dialogue sur le dessus de la pile, le _dialogue actif_. Lorsqu’un dialogue est sorti de la pile, le contexte du dialogue procèdera à la prochaine étape sur la cascade de la pile où il s’est arrêté.

Ainsi, vous pouvez créer une branche au sein de votre flux de conversation en incluant une étape dans un dialogue pouvant choisir sous condition un dialogue à démarrer parmi l’ensemble des dialogues potentiels.

## <a name="how-to-loop"></a>Créer une boucle

La méthode _replace_ du contexte de dialogue vous permet de remplacer le dialogue du dessus de la pile. L’état de l’ancien dialogue est abandonné et le nouveau dialogue commence du début. Vous pouvez utiliser cette méthode pour créer une boucle en remplaçant un dialogue par lui-même. Toutefois, pour [conserver l’état](bot-builder-howto-v4-state.md), vous devrez passer les informations à une nouvelle instance du dialogue dans l’appel de la méthode _replace_. Dans l’exemple suivant, vous verrez que le panier de la commande du repas est stocké dans l’état du dialogue, et lorsque le dialogue `orderPrompt` est répété, l’état actuel est passé pour que le nouvel état du dialogue puisse continuer à ajouter des éléments. Si vous préférez stocker ces informations dans les autres états, consultez [Conserver les données utilisateur](bot-builder-tutorial-persist-user-inputs.md).

## <a name="create-the-dialogs-for-the-hotel-bot"></a>Créer des dialogues pour le bot d’hôtel

Dans cette section, nous allons créer des dialogues pour gérer une conversation pour le bot d’hôtel que nous avons décrit.

### <a name="install-the-dialogs-library"></a>Installer la bibliothèque de dialogues

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Nous allons partir d’un modèle EchoBot de base. Pour obtenir des instructions, consultez le [démarrage rapide pour .NET](~/dotnet/bot-builder-dotnet-quickstart.md).

Pour utiliser des dialogues, vous devez installer le package NuGet `Microsoft.Bot.Builder.Dialogs` pour votre projet ou solution.
Référencez ensuite la bibliothèque de dialogues dans les instructions d’utilisation de vos fichiers de code.

```csharp
using Microsoft.Bot.Builder.Dialogs;
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Vous pouvez télécharger la bibliothèque `botbuilder-dialogs` à partir de NPM. Pour installer la bibliothèque `botbuilder-dialogs`, exécutez la commande NPM suivante :

```cmd
npm install --save botbuilder-dialogs
```

Pour utiliser des **dialogues** dans votre bot, incluez-les dans le code de celui-ci. Par exemple, ajoutez ce qui suit à votre fichier **app.js** :

```javascript
const botbuilder_dialogs = require('botbuilder-dialogs');
```

---

### <a name="create-a-dialog-set"></a>Créer un jeu de dialogues

Créez un _jeu de dialogues_ auquel nous ajouterons tous les dialogues de cet exemple.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Créez une classe **HotelDialog**, puis ajoutez les instructions d’utilisation dont nous aurons besoin.

```csharp
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Bot.Builder.Prompts.Choices;
using Microsoft.Bot.Schema;
using Microsoft.Recognizers.Text;
using System;
using System.Collections.Generic;
using System.Linq;
```

Dérivez la classe de **DialogSet**, et définissez les ID et les clés que nous utiliserons pour identifier les dialogues, les invites et les informations d’état de ce jeu de dialogues.

```csharp
/// <summary>Contains the set of dialogs and prompts for the hotel bot.</summary>
public class HotelDialog : DialogSet
{
    /// <summary>The ID of the top-level dialog.</summary>
    public const string MainMenu = "mainMenu";

    /// <summary>Contains the IDs for the other dialogs in the set.</summary>
    private static class Dialogs
    {
        public const string OrderDinner = "orderDinner";
        public const string OrderPrompt = "orderPrompt";
        public const string ReserveTable = "reserveTable";
    }

    /// <summary>Contains the IDs for the prompts used by the dialogs.</summary>
    private static class Inputs
    {
        public const string Choice = "choicePrompt";
        public const string Number = "numberPrompt";
    }

    /// <summary>Contains the keys used to manage dialog state.</summary>
    private static class Outputs
    {
        public const string OrderCart = "orderCart";
        public const string OrderTotal = "orderTotal";
        public const string RoomNumber = "roomNumber";
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Pour utiliser des **dialogues** dans votre bot, incluez-les dans le code de celui-ci.

Référencez la bibliothèque depuis votre fichier **app.js**.

```javascript
const botbuilder_dialogs = require('botbuilder-dialogs');
```

Créez ensuite le jeu de dialogues.

```javascript
const dialogs = new botbuilder_dialogs.DialogSet();
```

---

### <a name="add-the-prompts-to-the-set"></a>Ajouter les invites au jeu

Nous allons utiliser une invite **ChoicePrompt** pour demander aux invités s’ils souhaitent commander ou réserver une table, et pour leur fournir les options du menu à sélectionner. Nous utiliserons une invite **NumberPrompt** pour demander leur numéro de chambre s’ils commandent.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Dans le constructeur **HotelDialog**, ajoutez les deux invites.

```csharp
// Add the prompts.
this.Add(Inputs.Choice, new ChoicePrompt(Culture.English));
this.Add(Inputs.Number, new NumberPrompt<int>(Culture.English));
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Ajoutez ces deux invites au jeu de dialogues.

```javascript
dialogs.add('choicePrompt', new botbuilder_dialogs.ChoicePrompt());
dialogs.add('numberPrompt', new botbuilder_dialogs.NumberPrompt());
```

---

### <a name="define-some-of-the-supporting-information"></a>Définir certaines informations de support

Étant donné que nous aurons besoin d’informations sur chaque option du menu, définissons-les maintenant.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Créez une classe statique interne **Lists** pour contenir les informations. Nous créerons également des classes internes **WelcomeChoice** et **MenuChoice** pour contenir les informations de chaque option.

Tant que nous y sommes, nous ajouterons des informations pour la liste des options du dialogue d’accueil supérieur, et nous créerons aussi des listes de support que nous utiliserons plus tard lorsque nous demanderons ces informations aux invités. C’est du travail en plus fait en avance, mais cela simplifiera le code du dialogue.

```csharp
/// <summary>Describes an option for the top-level dialog.</summary>
private class WelcomeChoice
{
    /// <summary>The text to show the guest for this option.</summary>
    public string Description { get; set; }

    /// <summary>The ID of the associated dialog for this option.</summary>
    public string DialogName { get; set; }
}

/// <summary>Describes an option for the food-selection dialog.</summary>
/// <remarks>We have two types of options. One represents meal items that the guest
/// can add to their order. The other represents a request to process or cancel the
/// order.</remarks>
private class MenuChoice
{
    /// <summary>The request text for cancelling the meal order.</summary>
    public const string Cancel = "Cancel order";

    /// <summary>The request text for processing the meal order.</summary>
    public const string Process = "Process order";

    /// <summary>The name of the meal item or the request.</summary>
    public string Name { get; set; }

    /// <summary>The price of the meal item; or NaN for a request.</summary>
    public double Price { get; set; }

    /// <summary>The text to show the guest for this option.</summary>
    public string Description => (double.IsNaN(Price)) ? Name : $"{Name} - ${Price:0.00}";
}

/// <summary>Contains the lists used to present options to the guest.</summary>
private static class Lists
{
    /// <summary>The options for the top-level dialog.</summary>
    public static List<WelcomeChoice> WelcomeOptions { get; } = new List<WelcomeChoice>
    {
        new WelcomeChoice { Description = "Order dinner", DialogName = Dialogs.OrderDinner },
        new WelcomeChoice { Description = "Reserve a table", DialogName = Dialogs.ReserveTable },
    };

    private static List<string> WelcomeList { get; } = WelcomeOptions.Select(x => x.Description).ToList();

    /// <summary>The choices to present in the choice prompt for the top-level dialog.</summary>
    public static List<Choice> WelcomeChoices { get; } = ChoiceFactory.ToChoices(WelcomeList);

    /// <summary>The reprompt action for the top-level dialog.</summary>
    public static Activity WelcomeReprompt
    {
        get
        {
            var reprompt = MessageFactory.SuggestedActions(WelcomeList, "Please choose an option");
            reprompt.AttachmentLayout = AttachmentLayoutTypes.List;
            return reprompt as Activity;
        }
    }

    /// <summary>The options for the food-selection dialog.</summary>
    public static List<MenuChoice> MenuOptions { get; } = new List<MenuChoice>
    {
        new MenuChoice { Name = "Potato Salad", Price = 5.99 },
        new MenuChoice { Name = "Tuna Sandwich", Price = 6.89 },
        new MenuChoice { Name = "Clam Chowder", Price = 4.50 },
        new MenuChoice { Name = MenuChoice.Process, Price = double.NaN },
        new MenuChoice { Name = MenuChoice.Cancel, Price = double.NaN },
    };

    private static List<string> MenuList { get; } = MenuOptions.Select(x => x.Description).ToList();

    /// <summary>The choices to present in the choice prompt for the food-selection dialog.</summary>
    public static List<Choice> MenuChoices { get; } = ChoiceFactory.ToChoices(MenuList);

    /// <summary>The reprompt action for the food-selection dialog.</summary>
    public static Activity MenuReprompt
    {
        get
        {
            var reprompt = MessageFactory.SuggestedActions(MenuList, "Please choose an option");
            reprompt.AttachmentLayout = AttachmentLayoutTypes.List;
            return reprompt as Activity;
        }
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Créer une constante **dinnerMenu** pour contenir les informations.

```javascript
const dinnerMenu = {
    choices: ["Potato Salad - $5.99", "Tuna Sandwich - $6.89", "Clam Chowder - $4.50", 
        "Process order", "Cancel"],
    "Potato Salad - $5.99": {
        Description: "Potato Salad",
        Price: 5.99
    },
    "Tuna Sandwich - $6.89": {
        Description: "Tuna Sandwich",
        Price: 6.89
    },
    "Clam Chowder - $4.50": {
        Description: "Clam Chowder",
        Price: 4.50
    }
}
```

---

### <a name="create-the-welcome-dialog"></a>Créer le dialogue d’accueil

Ce dialogue utilise un `ChoicePrompt` pour afficher le menu et attend que l’utilisateur choisisse une option. Lorsque l’utilisateur choisit `Order dinner` ou `Reserve a table`, il démarre le dialogue pour le choix approprié, puis, une fois ce dialogue terminé, au lieu de simplement terminer le dialogue de la dernière étape et laisser l’utilisateur se demander ce qu’il se passe, ce dialogue `mainMenu` se répète en recommençant le dialogue `mainMenu`. Avec cette structure simple, le bot affichera toujours le menu et l’utilisateur connaîtra ainsi tous les choix à sa disposition.

Le dialogue **MainMenu** contient ces étapes en cascade.

* Dans la première étape de la cascade, nous initialisons ou supprimons l’état du dialogue, accueillons l’invité et lui demandons de choisir parmi les options disponibles : `Order dinner` ou `Reserve a table`.
* Dans la deuxième étape, nous récupérons son choix et commençons le dialogue enfant associé à ce choix. Une fois le dialogue enfant terminé, il reprendra à l’étape précédente.
* Dans la dernière étape, nous utilisons la méthode **DialogContext.Replace** pour remplacer ce dialogue par une nouvelle instance de lui-même, qui fera du dialogue d’accueil un menu en boucle.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// Add the main welcome dialog.
this.Add(MainMenu, new WaterfallStep[]
{
    async (dc, args, next) =>
    {
        // Greet the guest and ask them to choose an option.
        await dc.Context.SendActivity("Welcome to Contoso Hotel and Resort.");
        await dc.Prompt(Inputs.Choice, "How may we serve you today?", new ChoicePromptOptions
        {
            Choices = Lists.WelcomeChoices,
            RetryPromptActivity = Lists.WelcomeReprompt,
        });
    },
    async (dc, args, next) =>
    {
        // Begin a child dialog associated with the chosen option.
        var choice = (FoundChoice)args["Value"];
        var dialogId = Lists.WelcomeOptions[choice.Index].DialogName;

        await dc.Begin(dialogId, dc.ActiveDialog.State);
    },
    async (dc, args, next) =>
    {
        // Start this dialog over again.
        await dc.Replace(MainMenu);
    },
});
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// Display a menu and ask user to choose a menu item. Direct user to the item selected otherwise, show
// the menu again.
dialogs.add('mainMenu', [
    async function(dc){
        await dc.context.sendActivity("Welcome to Contoso Hotel and Resort.");
        await dc.prompt('choicePrompt', "How may we serve you today?", ['Order Dinner', 'Reserve a table']);
    },
    async function(dc, result){
        if(result.value.match(/order dinner/ig)){
            await dc.begin('orderDinner');
        }
        else if(result.value.match(/reserve a table/ig)){
            await dc.begin('reserveTable');
        }
    },
    async function(dc, result){
        // Start over
        await dc.endAll().begin('mainMenu');
    }
]);
```

---

### <a name="create-the-order-dinner-dialog"></a>Créer un dialogue de commande de menu

Dans le dialogue de commande, nous accueillons l’invité pour le « service de commande » et démarrons immédiatement le dialogue de choix du repas, que nous aborderons dans la prochaine section. Plus important, si l’invité demande au service de procéder à la commande, le dialogue de choix du repas renvoie la liste des éléments de la commande. Pour terminer le processus, ce dialogue demande ensuite le numéro de chambre pour livrer le repas, puis envoie un message de confirmation avant de terminer. Si l’invité annule la commande, ce dialogue ne demande pas de numéro de chambre et se termine immédiatement.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Pour la classe **HotelDialog**, ajoutez une structure de données que nous pouvons utiliser pour suivre la commande de l’invité.

```csharp
/// <summary>Contains the guest's dinner order.</summary>
private class OrderCart : List<MenuChoice> { }
```

Dans le constructeur **HotelDialog**, ajoutez le dialogue de commande du repas.

* Dans la première étape, nous envoyons un message d’accueil et démarrons le dialogue de choix du repas.
* Dans la deuxième étape, nous vérifions si le dialogue de choix du repas renvoie un panier de commande.
  * Si c’est le cas, nous demandons le numéro de sa chambre et enregistrons les informations du panier.
  * Sinon, nous supposons qu’il a annulé sa commande, et appelons **DialogContext.End** pour terminer le dialogue.
* Dans la dernière étape, nous enregistrons le numéro de sa chambre et lui envoyons un message de confirmation avant de terminer. Il s’agit de l’étape où votre bot enverra la commande à un service de traitement.

```csharp
// Add the order-dinner dialog.
this.Add(Dialogs.OrderDinner, new WaterfallStep[]
{
    async (dc, args, next) =>
    {
        await dc.Context.SendActivity("Welcome to our Dinner order service.");

        // Start the food selection dialog.
        await dc.Begin(Dialogs.OrderPrompt);
    },
    async (dc, args, next) =>
    {
        if (args.TryGetValue(Outputs.OrderCart, out object arg) && arg is OrderCart cart)
        {
            // If there are items in the order, record the order and ask for a room number.
            dc.ActiveDialog.State[Outputs.OrderCart] = cart;
            await dc.Prompt(Inputs.Number, "What is your room number?", new PromptOptions
            {
                RetryPromptString = "Please enter your room number."
            });
        }
        else
        {
            // Otherwise, assume the order was cancelled by the guest and exit.
            await dc.End();
        }
    },
    async (dc, args, next) =>
    {
        // Get and save the guest's answer.
        var roomNumber = args["Text"] as string;
        dc.ActiveDialog.State[Outputs.RoomNumber] = roomNumber;

        // Process the dinner order using the collected order cart and room number.

        await dc.Context.SendActivity($"Thank you. Your order will be delivered to room {roomNumber} within 45 minutes.");
        await dc.End();
    },
});
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// Order dinner:
// Help user order dinner from a menu
dialogs.add('orderDinner', [
    async function (dc){
        await dc.context.sendActivity("Welcome to our Dinner order service.");
        
        await dc.begin('orderPrompt', dc.activeDialog.state.orderCart); // Prompt for orders
    },
    async function (dc, result) {
        if(result == "Cancel"){
            await dc.end();
        }
        else { 
            await dc.prompt('numberPrompt', "What is your room number?");
        }
    },
    async function(dc, result){
        await dc.context.sendActivity(`Thank you. Your order will be delivered to room ${result} within 45 minutes.`);
        await dc.end();
    }
]);
```

---

### <a name="create-the-order-prompt-dialog"></a>Créer le dialogue de l’invite de commande

Dans le dialogue du choix du repas, nous présenterons à l’invité une liste d’options incluant les repas qu’il peut commander et les deux requêtes de traitement de commandes. Ce dialogue est en boucle jusqu’à ce que l’invité choisisse de traiter la commande ou annule sa commande.

* Lorsqu’il sélectionner un repas, nous l’ajoutons à son _panier_.
* Si l’invité choisit de traiter sa commande, nous vérifions d’abord si le panier est vide.
  * S’il est vide, nous envoyons un message d’erreur et la boucle continue.
  * Sinon, nous terminons le dialogue et renvoyons les informations du panier dans le dialogue parent.
* Si l’invité choisit d’annuler sa commande, nous terminons le dialogue sans renvoyer d’informations.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Dans le constructeur **HotelDialog**, ajoutez le dialogue de choix du repas.

* Dans la première étape, nous initialisons l’état du dialogue. Si les arguments d’entrée du dialogue contiennent les informations du panier, nous les enregistrons dans l’état du dialogue ; sinon, nous créons un panier vide pour les y ajouter. Nous demandons ensuite à l’invité de choisir dans le menu.
* Dans l’étape suivante, nous regardons l’option choisie par l’invité :
  * S’il s’agit d’une requête pour traiter la commande, nous vérifions si le panier contient des éléments.
    * Si c’est le cas, nous terminons le dialogue en renvoyant le contenu du panier.
    * Sinon, nous envoyons un message d’erreur à l’invité et recommençons au début du dialogue.
  * S’il s’agit d’une requête d’annulation de la commande, nous terminons le dialogue en renvoyant un dictionnaire vide.
  * S’il s’agit d’un repas, nous l’ajoutons au panier, envoyons un message de statut et redémarrons le dialogue, en passant l’état actuel de la commande.

```csharp
// Add the food-selection dialog.
this.Add(Dialogs.OrderPrompt, new WaterfallStep[]
    {
        async (dc, args, next) =>
        {
            if (args is null || !args.ContainsKey(Outputs.OrderCart))
            {
                // First time through, initialize the order state.
                dc.ActiveDialog.State[Outputs.OrderCart] = new OrderCart();
                dc.ActiveDialog.State[Outputs.OrderTotal] = 0.0;
            }
            else
            {
                // Otherwise, set the order state to that of the arguments.
                dc.ActiveDialog.State = new Dictionary<string, object>(args);
            }

            await dc.Prompt(Inputs.Choice, "What would you like?", new ChoicePromptOptions
            {
                Choices = Lists.MenuChoices,
                RetryPromptActivity = Lists.MenuReprompt,
            });
        },
        async (dc, args, next) =>
        {
            // Get the guest's choice.
            var choice = (FoundChoice)args["Value"];
            var option = Lists.MenuOptions[choice.Index];

            // Get the current order from dialog state.
            var cart = (OrderCart)dc.ActiveDialog.State[Outputs.OrderCart];

            if (option.Name is MenuChoice.Process)
            {
                if (cart.Count > 0)
                {
                    // If there are any items in the order, then exit this dialog,
                    // and return the list of selected food items.
                    await dc.End(new Dictionary<string, object>
                    {
                        [Outputs.OrderCart] = cart
                    });
                }
                else
                {
                    // Otherwise, send an error message and restart from
                    // the beginning of this dialog.
                    await dc.Context.SendActivity(
                        "Your cart is empty. Please add at least one item to the cart.");
                    await dc.Replace(Dialogs.OrderPrompt);
                }
            }
            else if (option.Name is MenuChoice.Cancel)
            {
                await dc.Context.SendActivity("Your order has been cancelled.");

                // Exit this dialog, returning an empty property bag.
                dc.ActiveDialog.State.Clear();
                await dc.End(new Dictionary<string, object>());
            }
            else
            {
                // Add the selected food item to the order and update the order total.
                cart.Add(option);
                var total = (double)dc.ActiveDialog.State[Outputs.OrderTotal] + option.Price;
                dc.ActiveDialog.State[Outputs.OrderTotal] = total;

                await dc.Context.SendActivity($"Added {option.Name} (${option.Price:0.00}) to your order." +
                    Environment.NewLine + Environment.NewLine +
                    $"Your current total is ${total:0.00}.");

                // Present the order options again, passing in the current order state.
                await dc.Replace(Dialogs.OrderPrompt, dc.ActiveDialog.State);
            }
        },
    });
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// Helper dialog to repeatedly prompt user for orders
dialogs.add('orderPrompt', [
    async function(dc, orderCart){
        // Define a new cart of one does not exists
        if(!orderCart){
            // Initialize a new cart
            // convoState = conversationState.get(dc.context);
            dc.activeDialog.state.orderCart = {
                orders: [],
                total: 0
            };
        }
        else {
            dc.activeDialog.state.orderCart = orderCart;
        }
        await dc.prompt('choicePrompt', "What would you like?", dinnerMenu.choices);
    },
    async function(dc, choice){
        // Get state object
        // convoState = conversationState.get(dc.context);

        if(choice.value.match(/process order/ig)){
            if(dc.activeDialog.state.orderCart.orders.length > 0) {
                // Process the order
                // ...
                dc.activeDialog.state.orderCart = undefined; // Reset cart
                await dc.context.sendActivity("Processing your order.");
                await dc.end();
            }
            else {
                await dc.context.sendActivity("Your cart was empty. Please add at least one item to the cart.");
                // Ask again
                await dc.replace('orderPrompt');
            }
        }
        else if(choice.value.match(/cancel/ig)){
            //dc.activeDialog.state.orderCart = undefined; // Reset cart
            await dc.context.sendActivity("Your order has been canceled.");
            await dc.end(choice.value);
        }
        else {
            var item = dinnerMenu[choice.value];

            // Only proceed if user chooses an item from the menu
            if(!item){
                await dc.context.sendActivity("Sorry, that is not a valid item. Please pick one from the menu.");
                
                // Ask again
                await dc.replace('orderPrompt');
            }
            else {
                // Add the item to cart
                dc.activeDialog.state.orderCart.orders.push(item);
                dc.activeDialog.state.orderCart.total += item.Price;

                await dc.context.sendActivity(`Added to cart: ${choice.value}. <br/>Current total: $${dc.activeDialog.state.orderCart.total}`);

                // Ask again
                await dc.replace('orderPrompt', dc.activeDialog.state.orderCart);
            }
        }
    }
]);
```

L’exemple de code ci-dessus montre que le dialogue `orderDinner` principal utilise un dialogue d’assistance nommé `orderPrompt` pour traiter les choix de l’utilisateur. Le dialogue `orderPrompt` affiche le menu, invite l’utilisateur à choisir un article, ajoute celui-ci au panier, puis invite à nouveau l’utilisateur en créant une boucle. Cela permet à l’utilisateur d’ajouter plusieurs articles à sa commande. Le dialogue se répète en boucle jusqu’à ce que l’utilisateur choisisse `Process order` ou `Cancel`. À ce moment-là, l’exécution est rendue au dialogue parent (le dialogue `orderDinner`). Le dialogue `orderDinner` fait un ménage de dernière minute si l’utilisateur souhaite traiter la commande. Autrement, il se termine et rend l’exécution à son dialogue parent (par exemple, `mainMenu`). Le dialogue `mainMenu` continue à son tour l’exécution de la dernière étape qui consiste simplement à réafficher les choix du menu principal.

---
### <a name="create-the-reserve-table-dialog"></a>Créer le dialogue de réservation d’une table

<!-- TODO: Update the "Manage simple conversation flows" topic to use a reserveTable dialog, and then reference that from here. -->

Pour que l’exemple reste simple, nous ne montrons ici qu’une implémentation minimale du dialogue `reserveTable`.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// Add the table-reservation dialog.
this.Add(Dialogs.ReserveTable, new WaterfallStep[]
    {
        // Replace this waterfall with your reservation steps.
        async (dc, args, next) =>
        {
            await dc.Context.SendActivity("Your table has been reserved.");
            await dc.End();
        }
    });
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// Reserve a table:
// Help the user to reserve a table

dialogs.add('reserveTable', [
    // Replace this waterfall with your reservation steps.
    async function(dc, args, next){
        await dc.context.sendActivity("Your table has been reserved");
        await dc.end();
    }
]);
```

---

## <a name="next-steps"></a>Étapes suivantes

Vous pouvez améliorer ce bot en offrant d’autres options comme « plus d’informations » ou « aide » comme choix du menu. Pour en savoir plus sur l’implémentation de ce type d’interruptions, consultez [Gérer les interruptions utilisateur](bot-builder-howto-handle-user-interrupt.md).

À présent que vous avez appris à utiliser des dialogues, des invites et des cascades pour gérer des flux de conversation complexes, voyons comment décomposer les dialogues (comme les dialogues `orderDinner` et `reserveTable`) en objets distincts au lieu de les regrouper tous dans un seul jeu de dialogues important.

> [!div class="nextstepaction"]
> [Créer une logique modulaire de robot avec contrôle composite](bot-builder-compositcontrol.md)
