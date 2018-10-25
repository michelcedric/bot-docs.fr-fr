---
title: Gérer les interruptions par l’utilisateur | Microsoft Docs
description: Découvrez comment gérer les interruptions par l’utilisateur et diriger le flux de conversation.
keywords: interruption, interruptions, changement de sujet, arrêt
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 09/20/2018
ms.reviewer: ''
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 74d4bb07274643d61da332d6ee1cdfb1a14372dc
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2018
ms.locfileid: "49998426"
---
# <a name="handle-user-interruptions"></a>Gérer les interruptions par l’utilisateur

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

La gestion des interruptions représente un aspect essentiel d’un bot efficace.

Vous pensez peut-être que vos utilisateurs suivront pas à pas le flux de conversation que vous avez défini. Pourtant, il est probable qu’ils changent d’avis ou posent une question au milieu du processus au lieu de répondre à la question posée. Dans ce type de situation, comment votre bot gérera-t-il l’entrée utilisateur ? Quelle sera l’expérience utilisateur ? Comment gérerez-vous les données d’état de l’utilisateur ? Gérer les interruptions signifie s’assurer que votre robot est prêt à gérer ce type de situations.

Il n’existe aucune réponse correcte à ces questions dans la mesure où chaque situation est spécifique au scénario que votre bot est supposé gérer. Dans cette rubrique, nous examinerons certaines méthodes courantes pour gérer les interruptions par l’utilisateur et suggérerons différentes façons de les implémenter dans votre bot.

## <a name="handle-expected-interruptions"></a>Gérer les interruptions attendues

Un flux de conversation procédural repose sur un ensemble d’étapes clés à travers lesquelles vous souhaitez guider l’utilisateur. Toute action de l’utilisateur ne correspondant pas à ces étapes représente une interruption potentielle. Dans le cadre d’un flux normal, vous pouvez anticiper certaines interruptions.

**Réservation de table** Dans un bot de réservation de table, les principales étapes pourraient consister à demander à l’utilisateur la date et l’heure de la réservation, le nombre d’invités et le nom auquel effectuer la réservation. Dans ce processus, vous pouvez anticiper certaines interruptions, notamment :

* `cancel` : pour quitter le processus.
* `help` : pour fournir une assistance supplémentaire concernant ce processus.
* `more info` : pour donner des conseils et des suggestions ou pour proposer d’autres moyens de réserver une table (par exemple, à l’aide d’une adresse e-mail ou d’un numéro de téléphone).
* `show list of available tables` : le cas échéant, pour afficher une liste des tables disponibles à date et à l’heure souhaitées par l’utilisateur.

**Commande d’un dîner** Dans un bot de commande de dîner, les principales étapes pourraient consister à fournir une liste de plats et à permettre à l’utilisateur d’en ajouter à son panier. Dans ce processus, vous pouvez anticiper certaines interruptions, notamment :

* `cancel` : pour quitter le processus de commande.
* `more info` : pour fournir des informations diététiques sur chaque plat.
* `help` : pour fournir une aide sur l’utilisation du système.
* `process order` : pour traiter la commande.

Vous pouvez mettre ces options à la disposition de l’utilisateur sous la forme d’une liste **d’actions suggérées** ou de conseils, de sorte que l’utilisateur connaisse au moins les commandes qu’il peut envoyer et que le bot comprendra.

Par exemple, dans le flux de commande de dîner, vous pouvez inclure les interruptions attendues avec les plats. Dans ce cas, les plats sont envoyés sous forme de tableau de `choices`.

# <a name="ctabcsharptab"></a>[C#](#tab/csharptab)

Nous allons définir le jeu de dialogues comme une sous-classe de **DialogSet**.

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading;
using System.Threading.Tasks;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Bot.Builder.Dialogs.Choices;

public class OrderDinnerDialogs : DialogSet
{
    public OrderDinnerDialogs(IStatePropertyAccessor<DialogState> dialogStateAccessor)
        : base(dialogStateAccessor)
    {
    }
}
```

Nous allons définir quelques classes internes pour décrire le menu.

```cs
/// <summary>
/// Contains information about an item on the menu.
/// </summary>
public class DinnerItem
{
    public string Description { get; set; }

    public double Price { get; set; }
}

/// <summary>
/// Describes the dinner menu, including the items on the menu and options for
/// interrupting the ordering process.
/// </summary>
public class DinnerMenu
{
    /// <summary>Gets the items on the menu.</summary>
    public static Dictionary<string, DinnerItem> MenuItems { get; } = new Dictionary<string, DinnerItem>
    {
        ["Potato salad"] = new DinnerItem { Description = "Potato Salad", Price = 5.99 },
        ["Tuna sandwich"] = new DinnerItem { Description = "Tuna Sandwich", Price = 6.89 },
        ["Clam chowder"] = new DinnerItem { Description = "Clam Chowder", Price = 4.50 },
    };

    /// <summary>Gets all the "interruptions" the bot knows how to process.</summary>
    public static List<string> Interrupts { get; } = new List<string>
    {
        "More info", "Process order", "Help", "Cancel",
    };

    /// <summary>Gets all of the valid inputs a user can make.</summary>
    public static List<string> Choices { get; }
        = MenuItems.Select(c => c.Key).Concat(Interrupts).ToList();
}
```

# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)

Nous allons partir d’un modèle EchoBot de base. Pour obtenir des instructions, consultez les instructions de [démarrage rapide pour JavaScript](~/javascript/bot-builder-javascript-quickstart.md).

Vous pouvez télécharger la bibliothèque `botbuilder-dialogs` à partir de NPM. Pour installer la bibliothèque `botbuilder-dialogs`, exécutez la commande npm suivante :

```cmd
npm install --save botbuilder-dialogs
```

Dans votre fichier **bot.js**, référencez les classes auxquelles nous ferons référence et définissez les identificateurs que nous utiliserons pour le dialogue.

```javascript
const { ActivityTypes } = require('botbuilder');
const { DialogSet, ChoicePrompt, WaterfallDialog, DialogTurnStatus } = require('botbuilder-dialogs');

// Name for the dialog state property accessor.
const DIALOG_STATE_PROPERTY = 'dialogStateProperty';

// Name of the order-prompt dialog.
const ORDER_PROMPT = 'orderingDialog';

// Name for the choice prompt for use in the dialog.
const CHOICE_PROMPT = 'choicePrompt';
```

Définissez les options que nous souhaitons voir apparaître dans le dialogue de commande.

```javascript
// The options on the dinner menu, including commands for the bot.
const dinnerMenu = {
    choices: ["Potato Salad - $5.99", "Tuna Sandwich - $6.89", "Clam Chowder - $4.50",
        "Process order", "Cancel", "More info", "Help"],
    "Potato Salad - $5.99": {
        description: "Potato Salad",
        price: 5.99
    },
    "Tuna Sandwich - $6.89": {
        description: "Tuna Sandwich",
        price: 6.89
    },
    "Clam Chowder - $4.50": {
        description: "Clam Chowder",
        price: 4.50
    }
}
```

---

Dans votre logique de commande, vous pouvez les vérifier à l’aide de correspondances de chaînes ou d’expressions régulières.

# <a name="ctabcsharptab"></a>[C#](#tab/csharptab)

Tout d’abord, nous devons définir une assistance pour assurer le suivi de nos commandes.

```cs
/// <summary>Helper class for storing the order.</summary>
public class Order
{
    public double Total { get; set; } = 0.0;

    public List<DinnerItem> Items { get; set; } = new List<DinnerItem>();

    public bool ReadyToProcess { get; set; } = false;

    public bool OrderProcessed { get; set; } = false;
}
```

Ajoutez des constantes pour suivre les ID dont nous aurons besoin.

```csharp
/// <summary>The ID of the top-level dialog.</summary>
public const string MainDialogId = "mainMenu";

/// <summary>The ID of the choice prompt.</summary>
public const string ChoicePromptId = "choicePrompt";

/// <summary>The ID of the order card value, tracked inside the dialog.</summary>
public const string OrderCartId = "orderCart";
```

Mettez à jour le constructeur pour ajouter une invite de choix et notre dialogue en cascade. Nous allons également définir les méthodes qui implémentent les étapes de la cascade.

```cs
public OrderDinnerDialogs(IStatePropertyAccessor<DialogState> dialogStateAccessor)
    : base(dialogStateAccessor)
{
    // Add a choice prompt for the dialog.
    Add(new ChoicePrompt(ChoicePromptId));

    // Define and add the main waterfall dialog.
    WaterfallStep[] steps = new WaterfallStep[]
    {
        PromptUserAsync,
        ProcessInputAsync,
    };

    Add(new WaterfallDialog(MainDialogId, steps));
}

/// <summary>
/// Defines the first step of the main dialog, which is to ask for input from the user.
/// </summary>
/// <param name="stepContext">The current waterfall step context.</param>
/// <param name="cancellationToken">The cancellation token.</param>
/// <returns>The task to perform.</returns>
private async Task<DialogTurnResult> PromptUserAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
{
    // Initialize order, continuing any order that was passed in.
    Order order = (stepContext.Options is Order oldCart && oldCart != null)
        ? new Order
        {
            Items = new List<DinnerItem>(oldCart.Items),
            Total = oldCart.Total,
            ReadyToProcess = oldCart.ReadyToProcess,
            OrderProcessed = oldCart.OrderProcessed,
        }
        : new Order();

    // Set the order cart in dialog state.
    stepContext.Values[OrderCartId] = order;

    // Prompt the user.
    return await stepContext.PromptAsync(
        "choicePrompt",
        new PromptOptions
        {
            Prompt = MessageFactory.Text("What would you like for dinner?"),
            RetryPrompt = MessageFactory.Text(
                "I'm sorry, I didn't understand that. What would you like for dinner?"),
            Choices = ChoiceFactory.ToChoices(DinnerMenu.Choices),
        },
        cancellationToken);
}

/// <summary>
/// Defines the second step of the main dialog, which is to process the user's input, and
/// repeat or exit as appropriate.
/// </summary>
/// <param name="stepContext">The current waterfall step context.</param>
/// <param name="cancellationToken">The cancellation token.</param>
/// <returns>The task to perform.</returns>
private async Task<DialogTurnResult> ProcessInputAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
{
    // Get the order cart from dialog state.
    Order order = stepContext.Values[OrderCartId] as Order;

    // Get the user's choice from the previous prompt.
    string response = (stepContext.Result as FoundChoice).Value;

    if (response.Equals("process order", StringComparison.InvariantCultureIgnoreCase))
    {
        order.ReadyToProcess = true;

        await stepContext.Context.SendActivityAsync(
            "Your order is on it's way!",
            cancellationToken: cancellationToken);

        // In production, you may want to store something more helpful.
        // "Process" the order and exit.
        order.OrderProcessed = true;
        return await stepContext.EndDialogAsync(null, cancellationToken);
    }
    else if (response.Equals("cancel", StringComparison.InvariantCultureIgnoreCase))
    {
        // Cancel the order.
        await stepContext.Context.SendActivityAsync(
            "Your order has been canceled",
            cancellationToken: cancellationToken);

        // Exit without processing the order.
        return await stepContext.EndDialogAsync(null, cancellationToken);
    }
    else if (response.Equals("more info", StringComparison.InvariantCultureIgnoreCase))
    {
        // Send more information about the options.
        string message = "More info: <br/>" +
            "Potato Salad: contains 330 calories per serving. Cost: 5.99 <br/>"
            + "Tuna Sandwich: contains 700 calories per serving. Cost: 6.89 <br/>"
            + "Clam Chowder: contains 650 calories per serving. Cost: 4.50";
        await stepContext.Context.SendActivityAsync(
            message,
            cancellationToken: cancellationToken);

        // Continue the ordering process, passing in the current order cart.
        return await stepContext.ReplaceDialogAsync(MainDialogId, order, cancellationToken);
    }
    else if (response.Equals("help", StringComparison.InvariantCultureIgnoreCase))
    {
        // Provide help information.
        string message = "To make an order, add as many items to your cart as " +
            "you like. Choose the `Process order` to check out. " +
            "Choose `Cancel` to cancel your order and exit.";
        await stepContext.Context.SendActivityAsync(
            message,
            cancellationToken: cancellationToken);

        // Continue the ordering process, passing in the current order cart.
        return await stepContext.ReplaceDialogAsync(MainDialogId, order, cancellationToken);
    }

    // We've checked for expected interruptions. Check for a valid item choice.
    if (!DinnerMenu.MenuItems.ContainsKey(response))
    {
        await stepContext.Context.SendActivityAsync("Sorry, that is not a valid item. " +
            "Please pick one from the menu.");

        // Continue the ordering process, passing in the current order cart.
        return await stepContext.ReplaceDialogAsync(MainDialogId, order, cancellationToken);
    }
    else
    {
        // Add the item to cart.
        DinnerItem item = DinnerMenu.MenuItems[response];
        order.Items.Add(item);
        order.Total += item.Price;

        // Acknowledge the input.
        await stepContext.Context.SendActivityAsync(
            $"Added `{response}` to your order; your total is ${order.Total:0.00}.",
            cancellationToken: cancellationToken);

        // Continue the ordering process, passing in the current order cart.
        return await stepContext.ReplaceDialogAsync(MainDialogId, order, cancellationToken);
    }
}
```

# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)

Définissez votre dialogue dans le constructeur du bot. Notez que le code vérifie et gère les interruptions _en premier_, puis il passe à l’étape logique suivante.

```javascript
constructor(conversationState) {
    this.dialogStateAccessor = conversationState.createProperty(DIALOG_STATE_PROPERTY);
    this.conversationState = conversationState;

    this.dialogs = new DialogSet(this.dialogStateAccessor)
        .add(new ChoicePrompt(CHOICE_PROMPT))
        .add(new WaterfallDialog(ORDER_PROMPT, [
            async (step) => {
                if (step.options && step.options.orders) {
                    // If an order cart was passed in, continue to use it.
                    step.values.orderCart = step.options;
                } else {
                    // Otherwise, start a new cart.
                    step.values.orderCart = {
                        orders: [],
                        total: 0
                    };
                }
                return await step.prompt(CHOICE_PROMPT, "What would you like?", dinnerMenu.choices);
            },
            async (step) => {
                const choice = step.result;
                if (choice.value.match(/process order/ig)) {
                    if (step.values.orderCart.orders.length > 0) {
                        // If the cart is not empty, process the order by returning the order to the parent context.
                        await step.context.sendActivity("Your order has been processed.");
                        return await step.endDialog(step.values.orderCart);
                    } else {
                        // Otherwise, prompt again.
                        await step.context.sendActivity("Your cart was empty. Please add at least one item to the cart.");
                        return await step.replaceDialog(ORDER_PROMPT);
                    }
                } else if (choice.value.match(/cancel/ig)) {
                    // Cancel the order, and return "cancel" to the parent context.
                    await step.context.sendActivity("Your order has been canceled.");
                    return await step.endDialog("cancelled");
                } else if (choice.value.match(/more info/ig)) {
                    // Provide more information, and then continue the ordering process.
                    var msg = "More info: <br/>Potato Salad: contains 330 calories per serving. <br/>"
                        + "Tuna Sandwich: contains 700 calories per serving. <br/>"
                        + "Clam Chowder: contains 650 calories per serving."
                    await step.context.sendActivity(msg);
                    return await step.replaceDialog(ORDER_PROMPT, step.values.orderCart);
                } else if (choice.value.match(/help/ig)) {
                    // Provide help information, and then continue the ordering process.
                    var msg = `Help: <br/>`
                        + `To make an order, add as many items to your cart as you like then choose `
                        + `the "Process order" option to check out.`
                    await step.context.sendActivity(msg);
                    return await step.replaceDialog(ORDER_PROMPT, step.values.orderCart);
                } else {
                    // The user has chosen a food item from the menu. Add the item to cart.
                    var item = dinnerMenu[choice.value];
                    step.values.orderCart.orders.push(item.description);
                    step.values.orderCart.total += item.price;

                    await step.context.sendActivity(`Added ${item.description} to your cart. <br/>`
                        + `Current total: $${step.values.orderCart.total}`);

                    // Continue the ordering process.
                    return await step.replaceDialog(ORDER_PROMPT, step.values.orderCart);
                }
            }
        ]));
}
```

Mettez à jour le gestionnaire de tour pour appeler le dialogue et afficher les résultats du processus de commande.

```javascript
async onTurn(turnContext) {
    if (turnContext.activity.type === ActivityTypes.Message) {
        let dc = await this.dialogs.createContext(turnContext);
        let dialogTurnResult = await dc.continueDialog();
        if (dialogTurnResult.status === DialogTurnStatus.complete) {
            // The dialog completed this turn.
            const result = dialogTurnResult.result;
            if (!result || result === "cancelled") {
                await turnContext.sendActivity('You cancelled your order.');
            } else {
                await turnContext.sendActivity(`Your order came to $${result.total}`);
            }
        } else if (!turnContext.responded) {
            // No dialog was active.
            await turnContext.sendActivity("Let's order dinner...");
            await dc.cancelAllDialogs();
            await dc.beginDialog(ORDER_PROMPT);
        } else {
            // The dialog is active.
        }
    } else {
        await turnContext.sendActivity(`[${turnContext.activity.type} event detected]`);
    }
    // Save state changes
    await this.conversationState.saveChanges(turnContext);
}
```

---

## <a name="handle-unexpected-interruptions"></a>Gérer les interruptions inattendues

Certaines interruptions ne correspondent pas au scénario pour lequel votre bot est conçu.
Même si vous ne pouvez pas anticiper toutes les interruptions, vous pouvez programmer votre bot pour gérer certains modèles d’interruption.

### <a name="switching-topic-of-conversations"></a>Changement de sujet de conversation

Que se passe-t-il si, au milieu d’une conversation, l’utilisateur souhaite passer à une autre conversation ? Par exemple, votre bot peut réserver une table et commander un dîner.
L’utilisateur suit le flux de _réservation de table_. Cependant, au lieu de répondre à la question « Combien de personnes participent au repas ? », il envoie le message « Commander un dîner ». Dans ce cas, l’utilisateur change d’avis et veut commencer une conversation sur la commande d’un dîner. Comment allez-vous gérer cette interruption ?

Vous pouvez changer de sujet et basculer vers le flux de commande de dîner, ou vous pouvez obliger l’utilisateur à résoudre le problème en lui indiquant que vous attendez un nombre et en reposant la question. Si vous autorisez l’utilisateur à changer de sujet, vous devez décider si vous allez enregistrer la progression de la conversation afin qu’il puisse la reprendre au même stade plus tard ou si vous allez supprimer toutes les informations que vous avez collectées, de sorte qu’il reprenne l’ensemble du processus la prochaine fois qu’il souhaite réserver une table. Pour plus d’informations sur la gestion des données d’état de l’utilisateur, consultez la rubrique [Save state using conversation and user properties](bot-builder-howto-v4-state.md) (Enregistrer l’état à l’aide des propriétés de conversation et d’utilisateur).

### <a name="apply-artificial-intelligence"></a>Appliquer l’intelligence artificielle

Pour les interruptions ne correspondant pas au scénario, vous pouvez essayer de deviner l’intention de l’utilisateur. Pour cela, vous pouvez utiliser des services d’intelligence artificielle comme QnA Maker, LUIS ou votre logique personnalisée, puis proposer des suggestions quant aux intentions de l’utilisateur envisagées par le bot.

Par exemple, au milieu du flux de réservation de table, l’utilisateur dit « Je veux commander un burger ». Le bot ne sait pas comment gérer cette situation dans le cadre de ce flux de conversation. Étant donné que le flux actuel n’a rien à voir avec une procédure de commande et que l’autre commande de conversation du bot est « Commander un dîner », le bot ne sait pas comment gérer cette entrée. Si vous appliquez le service LUIS, par exemple, vous pouvez former le modèle de sorte qu’il reconnaisse que l’utilisateur souhaite commander un repas (par exemple, LUIS peut retourner une intention « orderFood »). Par conséquent, le bot peut répondre « Vous souhaitez certainement commander un repas. Souhaitez-vous basculer vers notre processus de commande de dîner ? » Pour plus d’informations sur la formation de LUIS et sur la détection des intentions de l’utilisateur, consultez l’article [Utilisation de LUIS pour la compréhension langagière](bot-builder-howto-v4-luis.md).

### <a name="default-response"></a>Réponse par défaut

Si tout le reste échoue, envoyez une réponse par défaut au lieu de ne rien faire et de laisser l’utilisateur dans l’interrogation. La réponse par défaut doit indiquer à l’utilisateur les commandes comprises par le bot, de sorte qu’il puisse revenir sur la bonne voie.

Vous pouvez consulter l’indicateur de contexte **responded** à la fin de la logique du bot pour voir si le bot a renvoyé quelque chose à l’utilisateur durant ce tour. Si le bot traite l’entrée de l’utilisateur mais ne répond pas, il est probable qu’il ne sache pas comment gérer l’entrée. Dans ce cas, vous pouvez l’intercepter et envoyer un message par défaut à l’utilisateur.

L’action par défaut de votre bot consiste à présenter à l’utilisateur le dialogue `mainMenu`. C’est à vous de décider de l’expérience dont bénéficiera votre utilisateur dans cette situation avec votre bot.

# <a name="ctabcsharptab"></a>[C#](#tab/csharptab)

```cs
// Check whether we replied. If not then clear the dialog stack and present the main menu.
if (!turnContext.Responded)
{
    await dc.CancelAllDialogsAsync(cancellationToken);
    await dc.BeginDialogAsync(OrderDinnerDialogs.MainDialogId, null, cancellationToken);
}
```

# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)

```javascript
// Check to see if anyone replied. If not then clear all the stack and present the main menu
if (!context.responded) {
    await dc.cancelAllDialogs();
    await step.beginDialog('mainMenu');
}
```

---

## <a name="handling-global-interruptions"></a>Gestion des interruptions globales

Dans l’exemple ci-dessus, nous gérons les interruptions pouvant se produire lors d’un tour spécifique, dans un dialogue donné. Mais que se passe-t-il en cas d’interruptions globales, qui peuvent survenir à tout moment ?

Il est possible de les gérer en plaçant notre logique de gestion des interruptions dans le gestionnaire principal du bot, la fonction qui traite les classes `turnContext` entrantes et décide ce qu’il faut en faire.

Dans l’exemple ci-dessous, la _première action_ du bot consiste à vérifier si le texte du message entrant contient un indice montrant que l’utilisateur a besoin d’aide ou qu’il souhaite annuler : deux interruptions très courantes dans les bots. À la fin de cette vérification, le bot appelle `dc.continueDialog()` pour traiter tous les dialogues actifs en attente.

# <a name="ctabcsharptab"></a>[C#](#tab/csharptab)

```cs
// Check for top-level interruptions.
string utterance = turnContext.Activity.Text.Trim().ToLowerInvariant();

if (utterance == "help")
{
    // Start a general help dialog. Dialogs already on the stack remain and will continue
    // normally if the help dialog exits normally.
    await dc.BeginDialogAsync(OrderDinnerDialogs.HelpDialogId, null, cancellationToken);
}
else if (utterance == "cancel")
{
    // Cancel any dialog on the stack.
    await turnContext.SendActivityAsync("Canceled.", cancellationToken: cancellationToken);
    await dc.CancelAllDialogsAsync(cancellationToken);
}
else
{
    await dc.ContinueDialogAsync(cancellationToken);

    // Check whether we replied. If not then clear the dialog stack and present the main menu.
    if (!turnContext.Responded)
    {
        await dc.CancelAllDialogsAsync(cancellationToken);
        await dc.BeginDialogAsync(OrderDinnerDialogs.MainDialogId, null, cancellationToken);
    }
}
```

# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)

Nous avons pu gérer les interruptions avant d’envoyer une réponse à l’utilisateur dans le dialogue.

Cet extrait de code suppose que nous avons un élément `helpDialog` et un élément `mainMenu` dans notre jeu de dialogues.

```javascript
const utterance = (context.activity.text || '').trim();

// Let's look for some interruptions first!
if (utterance.match(/help/ig)) {
    // Launch a new help dialog if the user asked for help
    await dc.beginDialog('helpDialog');
} else if (utterance.match(/cancel/ig)) {
    // Cancel any active dialogs if the user says cancel
    await dc.context.sendActivity('Canceled.');
    await dc.cancelAllDialogs();
}

// If the bot hasn't yet responded...
if (!context.responded) {
    // Continue any active dialog, which might send a response...
    await dc.continueDialog();

    // Finally, if the bot still hasn't sent a response, send instructions.
    if (!context.responded) {
        await dc.cancelAllDialogs();
        await context.sendActivity("Starting the main menu...");
        await dc.beginDialog('mainMenu');
    }
}
```

---
