---
title: Gérer un flux de conversation complexe avec des dialogues | Microsoft Docs
description: Découvrez comment gérer un flux de conversation complexe avec des dialogues dans le Kit de développement logiciel (SDK) Bot Builder pour Node.js.
keywords: flux de conversation complexe, répétition, boucle, menu, dialogues, invites, cascades, jeu de dialogues
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 10/03/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 871bfd9f8d693c5082fe1ccf38349f4d3d46ece2
ms.sourcegitcommit: b8bd66fa955217cc00b6650f5d591b2b73c3254b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/15/2018
ms.locfileid: "49326566"
---
# <a name="manage-complex-conversation-flows-with-dialogs"></a>Gérer des flux de conversation complexes avec dialogues

[!INCLUDE [pre-release-label](~/includes/pre-release-label.md)]

Dans le dernier article, nous avons expliqué comment utiliser la bibliothèque des dialogues pour gérer des conversations simples. Dans des [flux de conversation simples](bot-builder-dialog-manage-conversation-flow.md), l’utilisateur commence à la première étape d’une *cascade*, continue jusqu’à la dernière étape où se termine l’échange conversationnel. Dans cet article, nous allons utiliser des dialogues pour gérer plus de conversations complexes avec des sections en branche ou en boucle. Pour ce faire, nous allons utiliser différentes méthodes définies sur le contexte des dialogues et le contexte des étapes de cascade, et nous allons passer des arguments entre les différentes parties du dialogue.

Consultez la section [Bibliothèque des dialogues](bot-builder-concept-dialog.md) pour plus d’informations sur les dialogues.

Pour vous offrir plus de contrôle sur la *pile de dialogue*, la bibliothèque **Dialogues** fournit une méthode _replace dialog_. Cette méthode vous permet d’échanger le dialogue actif avec un autre tout en conservant l’état et le flux de la conversation. Les méthodes _begin dialog_ et _replace dialog_ vous permettent de créer autant de branches et de boucles que nécessaire pour créer des interactions plus complexes. Si votre conversation devient si complexe que les dialogues en cascade deviennent difficiles à gérer, envisagez d’utiliser un [dialogue composant](bot-builder-compositcontrol.md) ou de créer une classe de gestion de dialogue personnalisée s’appuyant sur la classe `Dialog` de base.

Dans cet article, nous allons créer des exemples de dialogues pour un bot de concierge d’hôtel qu’un client pourrait utiliser pour accéder à des services courants : réserver une table dans le restaurant de l’hôtel et commander un repas au room service.  Chacun de ces composants, ainsi qu’un menu les reliant, sont créés sous forme de dialogues dans un jeu de dialogues.

Le dialogue de niveau supérieur du bot propose ces deux options au client. Si le client souhaite réserver une table, le dialogue de niveau supérieur utilise la méthode async _begin dialog_ pour commencer le dialogue de réservation de table. Si le client souhaite commander un repas au room service, le dialogue de niveau supérieur démarre un dialogue de commande de repas.

Le dialogue de commande de repas demande d’abord au client de sélectionner les plats depuis un menu, puis d’indiquer son numéro de chambre. La sélection des produits s’effectue _également_ via un dialogue qui réapparaît plusieurs fois pour laisser le client faire son choix dans le menu avant d’envoyer sa commande de repas.

Ce diagramme illustre les dialogues que nous allons créer dans cet article, et les relations qu’ils entretiennent entre eux.

![Illustration des dialogues utilisés dans cet article](~/media/complex-conversation-flows.png)

## <a name="how-to-branch"></a>Créer une branche

Le contexte du dialogue maintient une _pile de dialogue_, et suit l’étape suivante de chaque dialogue sur la pile. Sa méthode de _démarrage de dialogue_ envoie un dialogue par push sur le dessus de la pile, et sa méthode _fin de dialogue_ fait sortir le dialogue du dessus de la pile.

Pour créer une branche, choisissez un élément à partir d’un jeu de dialogues enfants pour commencer. Pour plus d’informations sur ce concept, consultez la section [Créer une branche de conversation](bot-builder-concept-dialog.md#branch-a-conversation).

## <a name="how-to-loop"></a>Créer une boucle

La méthode _replace dialog_ du contexte de dialogue vous permet de remplacer le dialogue du dessus de la pile. L’état de l’ancien dialogue est abandonné et le nouveau dialogue commence du début. Vous pouvez utiliser cette méthode pour créer une boucle en remplaçant un dialogue par lui-même. Toutefois, pour [conserver les informations que le bot a déjà collectées](bot-builder-howto-v4-state.md), vous devrez passer ces informations à une nouvelle instance du dialogue dans l’appel de la méthode _replace dialog_.

Dans l’exemple suivant, vous verrez que la commande au room service est stockée dans l’état du dialogue, et lorsque le dialogue `orderPrompt` est répété, l’état actuel est passé pour que le nouvel état du dialogue puisse continuer à ajouter des éléments. Si vous préférez stocker ces informations dans un état de bot hors dialogue, consultez la section expliquant comment [Conserver les données utilisateur](bot-builder-tutorial-persist-user-inputs.md).

## <a name="create-the-dialogs-for-the-hotel-bot"></a>Créer des dialogues pour le bot d’hôtel

Dans cette section, nous allons créer des dialogues pour gérer une conversation pour le bot d’hôtel que nous avons décrit.

### <a name="install-the-dialogs-library"></a>Installer la bibliothèque de dialogues

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Nous allons partir d’un modèle EchoBot de base. Pour obtenir des instructions, consultez le [démarrage rapide pour .NET](../dotnet/bot-builder-dotnet-sdk-quickstart.md).

Pour utiliser des dialogues, vous devez installer le package NuGet `Microsoft.Bot.Builder.Dialogs` pour votre projet ou solution.
Référencez ensuite la bibliothèque de dialogues dans les instructions d’utilisation de vos fichiers de code.

```csharp
using Microsoft.Bot.Builder.Dialogs;
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Nous allons partir d’un modèle EchoBot. Pour obtenir des instructions, consultez le [démarrage rapide pour JavaScript](../javascript/bot-builder-javascript-quickstart.md).



Vous pouvez télécharger la bibliothèque `botbuilder-dialogs` à partir de npm. Pour installer la bibliothèque `botbuilder-dialogs`, exécutez la commande npm suivante :

```cmd
npm install --save botbuilder-dialogs
```

Pour utiliser des **dialogues** dans votre bot, incluez-les dans le code de celui-ci. Par exemple, ajoutez ce qui suit à votre fichier **index.js** :

```javascript
const { DialogSet } = require('botbuilder-dialogs');
```

Et ceci à votre fichier **bot.js** :

```javascript
const { DialogSet, NumberPrompt, ChoicePrompt, WaterfallDialog } = require('botbuilder-dialogs');
```

---

### <a name="create-a-dialog-set"></a>Créer un jeu de dialogues

Créez un _jeu de dialogues_ auquel nous ajouterons tous les dialogues de cet exemple.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Créez une classe **HotelDialogs**, puis ajoutez les instructions d’utilisation dont nous aurons besoin.

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading;
using System.Threading.Tasks;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Bot.Builder.Dialogs.Choices;
using Microsoft.Bot.Schema;
```

Dérivez la classe de **DialogSet**. Incluez un constructeur qui accepte un paramètre `IStatePropertyAccessor<DialogState>` à utiliser pour gérer l’état interne d’une instance du jeu de dialogues. Définissez également les ID et les clés que nous utiliserons pour identifier les dialogues, les invites et les informations d’état de ce jeu de dialogues.

```csharp
/// <summary>Contains the set of dialogs and prompts for the hotel bot.</summary>
public class HotelDialogs : DialogSet
{
    /// <summary>The ID of the top-level dialog.</summary>
    public const string MainMenu = "mainMenu";

    public HotelDialogs(IStatePropertyAccessor<DialogState> dialogStateAccessor)
        : base(dialogStateAccessor)
    {
    }

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

Dans le fichier **index.js**, ajoutez du code pour créer un accesseur de propriété d’état pour la gestion de l’état du dialogue, et utilisez-le pour créer le jeu de dialogues que nous utiliserons pour le bot.

```javascript
// Create conversation state with in-memory storage provider.
const conversationState = new ConversationState(memoryStorage);
const dialogStateAccessor = conversationState.createProperty('dialogState');

// Create a dialog set for the bot.
const dialogSet = new DialogSet(dialogStateAccessor);

// Create the bot.
const bot = new MyBot(conversationState, dialogSet)
```

Mettez ensuite à jour l’appel de traitement de l’activité pour qu’il utilise l’objet de bot.

```javascript
// Listen for incoming requests.
server.post('/api/messages', (req, res) => {
    adapter.processActivity(req, res, async (context) => {
        // Route to the bot's turn handler.
        await bot.onTurn(context);
    });
});
```

---

### <a name="add-the-prompts-to-the-set"></a>Ajouter les invites au jeu

Nous allons utiliser une invite **ChoicePrompt** pour demander aux invités s’ils souhaitent commander ou réserver une table, et pour leur fournir les options du menu à sélectionner. Nous utiliserons une invite **NumberPrompt** pour demander leur numéro de chambre s’ils commandent.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Dans le constructeur **HotelDialogs**, ajoutez les deux invites.

```csharp
// Add the prompts.
Add(new ChoicePrompt(Inputs.Choice));
Add(new NumberPrompt<int>(Inputs.Number));
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Mettez à jour le constructeur du bot, et ajoutez deux invites au jeu de dialogues.

```javascript
constructor(conversationState, dialogSet) {
    // Creates a new state accessor property.
    // See https://aka.ms/about-bot-state-accessors to learn more about the bot state and state accessors.
    this.countProperty = conversationState.createProperty(TURN_COUNTER_PROPERTY);
    this.conversationState = conversationState;
    this.dialogSet = dialogSet;

    this.dialogSet.add(new ChoicePrompt('choicePrompt'));
    this.dialogSet.add(new NumberPrompt('numberPrompt'));
}
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
    /// <summary>Gets or sets the text to show the guest for this option.</summary>
    public string Description { get; set; }

    /// <summary>Gets or sets the ID of the associated dialog for this option.</summary>
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

    /// <summary>Gets or sets the name of the meal item or the request.</summary>
    public string Name { get; set; }

    /// <summary>Gets or sets the price of the meal item; or NaN for a request.</summary>
    public double Price { get; set; }

    /// <summary>Gets the text to show the guest for this option.</summary>
    public string Description => double.IsNaN(Price) ? Name : $"{Name} - ${Price:0.00}";
}
```

```csharp
/// <summary>Contains the lists used to present options to the guest.</summary>
private static class Lists
{
    /// <summary>Gets the options for the top-level dialog.</summary>
    public static List<WelcomeChoice> WelcomeOptions { get; } = new List<WelcomeChoice>
    {
        new WelcomeChoice { Description = "Order dinner", DialogName = Dialogs.OrderDinner },
        new WelcomeChoice { Description = "Reserve a table", DialogName = Dialogs.ReserveTable },
    };

    private static readonly List<string> _welcomeList = WelcomeOptions.Select(x => x.Description).ToList();

    /// <summary>Gets the choices to present in the choice prompt for the top-level dialog.</summary>
    public static IList<Choice> WelcomeChoices { get; } = ChoiceFactory.ToChoices(_welcomeList);

    /// <summary>Gets the reprompt action for the top-level dialog.</summary>
    public static Activity WelcomeReprompt
    {
        get
        {
            var reprompt = MessageFactory.SuggestedActions(_welcomeList, "Please choose an option");
            reprompt.AttachmentLayout = AttachmentLayoutTypes.List;
            return reprompt as Activity;
        }
    }

    /// <summary>Gets the options for the food-selection dialog.</summary>
    public static List<MenuChoice> MenuOptions { get; } = new List<MenuChoice>
    {
        new MenuChoice { Name = "Potato Salad", Price = 5.99 },
        new MenuChoice { Name = "Tuna Sandwich", Price = 6.89 },
        new MenuChoice { Name = "Clam Chowder", Price = 4.50 },
        new MenuChoice { Name = MenuChoice.Process, Price = double.NaN },
        new MenuChoice { Name = MenuChoice.Cancel, Price = double.NaN },
    };

    private static readonly List<string> _menuList = MenuOptions.Select(x => x.Description).ToList();

    /// <summary>Gets the choices to present in the choice prompt for the food-selection dialog.</summary>
    public static IList<Choice> MenuChoices { get; } = ChoiceFactory.ToChoices(_menuList);

    /// <summary>Gets the reprompt action for the food-selection dialog.</summary>
    public static Activity MenuReprompt
    {
        get
        {
            var reprompt = MessageFactory.SuggestedActions(_menuList, "Please choose an option");
            reprompt.AttachmentLayout = AttachmentLayoutTypes.List;
            return reprompt as Activity;
        }
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Dans le fichier **bot.js**, créez une constante **dinnerMenu** pour contenir les informations.

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

Ce dialogue utilise un `ChoicePrompt` pour afficher le menu et attend que l’utilisateur choisisse une option. Lorsque l’utilisateur choisit `Order dinner` ou `Reserve a table`, il démarre le dialogue approprié. Lorsque le dialogue enfant est terminé, au lieu de simplement terminer le dialogue de la dernière étape et laisser l’utilisateur se demander ce qu’il se passe, le dialogue `mainMenu` se répète en redémarrant le dialogue `mainMenu`. Avec cette structure simple, le bot affichera toujours le menu et l’utilisateur connaîtra ainsi tous les choix à sa disposition.

Le dialogue `mainMenu` contient ces étapes en cascade :

* Dans la première étape de la cascade, nous initialisons ou supprimons l’état du dialogue, accueillons l’invité et lui demandons de choisir parmi les options disponibles : `Order dinner` ou `Reserve a table`.
* Dans la deuxième étape, nous récupérons son choix et commençons le dialogue enfant associé à ce choix. Une fois le dialogue enfant terminé, il reprendra à l’étape précédente.
* Dans la dernière étape, nous utilisons la méthode _replace dialog_ pour remplacer ce dialogue par une nouvelle instance de lui-même, qui fera du dialogue d’accueil un menu en boucle.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Dans le constructeur, ajoutez le dialogue en cascade. Définissez ensuite les étapes de la cascade. Ici, nous les avons définies dans une classe imbriquée.

Dans le constructeur **HotelDialogs**.

```csharp
// Define the steps for and add the main welcome dialog.
WaterfallStep[] welcomeDialogSteps = new WaterfallStep[]
{
    MainDialogSteps.PresentMenuAsync,
    MainDialogSteps.ProcessInputAsync,
    MainDialogSteps.RepeatMenuAsync,
};

Add(new WaterfallDialog(MainMenu, welcomeDialogSteps));
```

Dans la classe **HotelDialogs**, ajoutez les définitions de l’étape.

```csharp
/// <summary>
/// Contains the waterfall dialog steps for the order-dinner dialog.
/// </summary>
private static class MainDialogSteps
{
    public static async Task<DialogTurnResult> PresentMenuAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken)
    {
        // Greet the guest and ask them to choose an option.
        await stepContext.Context.SendActivityAsync(
            "Welcome to Contoso Hotel and Resort.",
            cancellationToken: cancellationToken);
        return await stepContext.PromptAsync(
            Inputs.Choice,
            new PromptOptions
            {
                Prompt = MessageFactory.Text("How may we serve you today?"),
                RetryPrompt = Lists.WelcomeReprompt,
                Choices = Lists.WelcomeChoices,
            },
            cancellationToken);
    }

    public static async Task<DialogTurnResult> ProcessInputAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken)
    {
        // Begin a child dialog associated with the chosen option.
        var choice = (FoundChoice)stepContext.Result;
        var dialogId = Lists.WelcomeOptions[choice.Index].DialogName;

        return await stepContext.BeginDialogAsync(dialogId, null, cancellationToken);
    }

    public static async Task<DialogTurnResult> RepeatMenuAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken)
    {
        // Start this dialog over again.
        return await stepContext.ReplaceDialogAsync(MainMenu, null, cancellationToken);
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Dans le constructeur du bot, ajoutez le dialogue en cascade `mainMenu`.

```javascript
// Display a menu and ask user to choose a menu item. Direct user to the item selected otherwise, show
// the menu again.
this.dialogSet.add(new WaterfallDialog('mainMenu', [
    async function (step) {
        // Welcome the user and send a prompt.
        await step.context.sendActivity("Welcome to Contoso Hotel and Resort.");
        return await step.prompt('choicePrompt', "How may we serve you today?", ['Order Dinner', 'Reserve a table']);
    },
    async function (step) {
        // Handle the user's response to the previous prompt and branch the dialog.
        if (step.result.value.match(/order dinner/ig)) {
            return await step.beginDialog('orderDinner');
        } else if (step.result.value.match(/reserve a table/ig)) {
            return await step.beginDialog('reserveTable');
        }
    },
    async function (step) {
        // Calling replaceDialog will loop the main menu
        return await step.replaceDialog('mainMenu');
    }
]));
```

---

### <a name="create-the-order-dinner-dialog"></a>Créer un dialogue de commande de menu

Dans le dialogue de commande, nous accueillons l’invité pour le « service de commande » et démarrons immédiatement le dialogue de choix du repas, que nous aborderons dans la prochaine section. Plus important, si l’invité demande au service de procéder à la commande, le dialogue de choix du repas renvoie la liste des éléments de la commande. Pour terminer le processus, ce dialogue demande ensuite le numéro de chambre pour livrer le repas, puis envoie un message de confirmation. Si l’invité annule la commande, ce dialogue ne demande pas de numéro de chambre et se termine immédiatement.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Pour la classe **HotelDialog**, ajoutez une structure de données que nous pouvons utiliser pour suivre la commande de l’invité. Étant donné que ces informations sont conservées dans l’état du dialogue, la classe a besoin d’un constructeur par défaut pour la sérialisation.

```csharp
/// <summary>Contains the guest's dinner order.</summary>
private class OrderCart : List<MenuChoice>
{
    public OrderCart() : base() { }

    public OrderCart(OrderCart other) : base(other) { }
}
```

Dans le constructeur, ajoutez le dialogue en cascade. Définissez ensuite les étapes de la cascade. Ici, nous les avons définies dans une classe imbriquée.

Dans le constructeur **HotelDialogs**.

```csharp
// Define the steps for and add the order-dinner dialog.
WaterfallStep[] orderDinnerDialogSteps = new WaterfallStep[]
{
    OrderDinnerSteps.StartFoodSelectionAsync,
    OrderDinnerSteps.GetRoomNumberAsync,
    OrderDinnerSteps.ProcessOrderAsync,
};

Add(new WaterfallDialog(Dialogs.OrderDinner, orderDinnerDialogSteps));
```

Dans la classe **HotelDialogs**, ajoutez les définitions de l’étape.

* Dans la première étape, nous envoyons un message d’accueil et démarrons le dialogue de choix du repas.
* Dans la deuxième étape, nous vérifions si le dialogue de choix du repas renvoie un panier de commande.
  * Si c’est le cas, nous demandons le numéro de sa chambre et enregistrons les informations du panier.
  * Sinon, nous supposons qu’il a annulé sa commande, et appelons la méthode _end dialog_ pour terminer le dialogue.
* Dans la dernière étape, nous enregistrons le numéro de sa chambre et lui envoyons un message de confirmation avant de terminer. Il s’agit de l’étape où votre bot enverra la commande à un service de traitement.

```csharp
/// <summary>
/// Contains the waterfall dialog steps for the order-dinner dialog.
/// </summary>
private static class OrderDinnerSteps
{
    public static async Task<DialogTurnResult> StartFoodSelectionAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken)
    {
        await stepContext.Context.SendActivityAsync(
            "Welcome to our Dinner order service.",
            cancellationToken: cancellationToken);

        // Start the food selection dialog.
        return await stepContext.BeginDialogAsync(Dialogs.OrderPrompt, null, cancellationToken);
    }

    public static async Task<DialogTurnResult> GetRoomNumberAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken)
    {
        if (stepContext.Result != null && stepContext.Result is OrderCart cart)
        {
            // If there are items in the order, record the order and ask for a room number.
            stepContext.Values[Outputs.OrderCart] = cart;
            return await stepContext.PromptAsync(
                Inputs.Number,
                new PromptOptions
                {
                    Prompt = MessageFactory.Text("What is your room number?"),
                    RetryPrompt = MessageFactory.Text("Please enter your room number."),
                },
                cancellationToken);
        }
        else
        {
            // Otherwise, assume the order was cancelled by the guest and exit.
            return await stepContext.EndDialogAsync(null, cancellationToken);
        }
    }

    public static async Task<DialogTurnResult> ProcessOrderAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken)
    {
        // Get and save the guest's answer.
        var roomNumber = (int)stepContext.Result;
        stepContext.Values[Outputs.RoomNumber] = roomNumber;

        // Process the dinner order using the collected order cart and room number.

        await stepContext.Context.SendActivityAsync(
            $"Thank you. Your order will be delivered to room {roomNumber} within 45 minutes.",
            cancellationToken: cancellationToken);
        return await stepContext.EndDialogAsync(null, cancellationToken);
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Dans le constructeur du bot, ajoutez le dialogue en cascade `orderDinner`.

```javascript
// Order dinner:
// Help user order dinner from a menu
this.dialogSet.add(new WaterfallDialog('orderDinner', [
    async function (step) {
        await step.context.sendActivity("Welcome to our dinner order service.");

        return await step.beginDialog('orderPrompt', step.values.orderCart = {
            orders: [],
            total: 0
        }); // Prompt for orders
    },
    async function (step) {
        if (step.result == "Cancel") {
            return await step.endDialog();
        } else {
            return await step.prompt('numberPrompt', "What is your room number?");
        }
    },
    async function (step) {
        await step.context.sendActivity(`Thank you. Your order will be delivered to room ${step.result} within 45 minutes.`);
        return await step.endDialog();
    }
]));
```

---

### <a name="create-the-order-prompt-dialog"></a>Créer le dialogue de l’invite de commande

Dans le dialogue du choix du repas, nous présenterons à l’invité une liste d’options incluant les repas qu’il peut commander et les deux requêtes de traitement de commandes. Ce dialogue est en boucle jusqu’à ce que l’invité choisisse de traiter la commande ou annule sa commande.

* Lorsqu’il sélectionne un repas, nous l’ajoutons à son _panier_.
* Si l’invité choisit de traiter sa commande, nous vérifions d’abord si le panier est vide.
  * S’il est vide, nous envoyons un message d’erreur et la boucle continue.
  * Sinon, nous terminons le dialogue et renvoyons les informations du panier dans le dialogue parent.
* Si l’invité choisit d’annuler sa commande, nous terminons le dialogue sans renvoyer d’informations.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Dans le constructeur, ajoutez le dialogue en cascade. Définissez ensuite les étapes de la cascade. Ici, nous les avons définies dans une classe imbriquée.

Dans le constructeur **HotelDialogs**.

```csharp
// Define the steps for and add the order-prompt dialog.
WaterfallStep[] orderPromptDialogSteps = new WaterfallStep[]
{
    OrderPromptSteps.PromptForItemAsync,
    OrderPromptSteps.ProcessInputAsync,
};

Add(new WaterfallDialog(Dialogs.OrderPrompt, orderPromptDialogSteps));
```

* Dans la première étape, nous initialisons l’état du dialogue. Si les arguments d’entrée du dialogue contiennent les informations du panier, nous les enregistrons dans l’état du dialogue ; sinon, nous créons un panier vide pour les y ajouter. Nous demandons ensuite à l’invité de choisir dans le menu.
* Dans l’étape suivante, nous regardons l’option choisie par l’invité :
  * S’il s’agit d’une requête pour traiter la commande, nous vérifions si le panier contient des éléments.
    * Si c’est le cas, nous terminons le dialogue en renvoyant le contenu du panier.
    * Sinon, nous envoyons un message d’erreur à l’invité et recommençons au début du dialogue.
  * S’il s’agit d’une requête d’annulation de la commande, nous terminons le dialogue en renvoyant un dictionnaire vide.
  * S’il s’agit d’un repas, nous l’ajoutons au panier, envoyons un message de statut et redémarrons le dialogue, en passant l’état actuel de la commande.

Dans la classe **HotelDialogs**, ajoutez les définitions de l’étape.

```csharp
/// <summary>
/// Contains the waterfall dialog steps for the order-prompt dialog.
/// </summary>
private static class OrderPromptSteps
{
    public static async Task<DialogTurnResult> PromptForItemAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken)
    {
        // First time through, options will be null.
        var cart = (stepContext.Options is OrderCart oldCart && oldCart != null)
            ? new OrderCart(oldCart) : new OrderCart();

        stepContext.Values[Outputs.OrderCart] = cart;
        stepContext.Values[Outputs.OrderTotal] = cart.Sum(item => item.Price);

        return await stepContext.PromptAsync(
            Inputs.Choice,
            new PromptOptions
            {
                Prompt = MessageFactory.Text("What would you like?"),
                RetryPrompt = Lists.MenuReprompt,
                Choices = Lists.MenuChoices,
            },
            cancellationToken);
    }

    public static async Task<DialogTurnResult> ProcessInputAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken)
    {
        // Get the guest's choice.
        var choice = (FoundChoice)stepContext.Result;
        var menuOption = Lists.MenuOptions[choice.Index];

        // Get the current order from dialog state.
        var cart = (OrderCart)stepContext.Values[Outputs.OrderCart];

        if (menuOption.Name is MenuChoice.Process)
        {
            if (cart.Count > 0)
            {
                // If there are any items in the order, then exit this dialog,
                // and return the list of selected food items.
                return await stepContext.EndDialogAsync(cart, cancellationToken);
            }
            else
            {
                // Otherwise, send an error message and restart from
                // the beginning of this dialog.
                await stepContext.Context.SendActivityAsync(
                    "Your cart is empty. Please add at least one item to the cart.",
                    cancellationToken: cancellationToken);
                return await stepContext.ReplaceDialogAsync(Dialogs.OrderPrompt, null, cancellationToken);
            }
        }
        else if (menuOption.Name is MenuChoice.Cancel)
        {
            await stepContext.Context.SendActivityAsync(
                "Your order has been cancelled.",
                cancellationToken: cancellationToken);

            // Exit this dialog, returning null.
            return await stepContext.EndDialogAsync(null, cancellationToken);
        }
        else
        {
            // Add the selected food item to the order and update the order total.
            cart.Add(menuOption);
            var total = (double)stepContext.Values[Outputs.OrderTotal] + menuOption.Price;
            stepContext.Values[Outputs.OrderTotal] = total;

            await stepContext.Context.SendActivityAsync(
                $"Added {menuOption.Name} (${menuOption.Price:0.00}) to your order." +
                    Environment.NewLine + Environment.NewLine +
                    $"Your current total is ${total:0.00}.",
                cancellationToken: cancellationToken);

            // Present the order options again, passing in the current order state.
            return await stepContext.ReplaceDialogAsync(Dialogs.OrderPrompt, cart);
        }
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Dans le constructeur du bot, ajoutez le dialogue en cascade `orderPrompt`.

```javascript
// Helper dialog to repeatedly prompt user for orders
this.dialogSet.add(new WaterfallDialog('orderPrompt', [
    async function (step) {
        // Define a new cart of one does not exists
        step.values.orderCart = step.options;

        return await step.prompt('choicePrompt', "What would you like?", dinnerMenu.choices);
    },
    async function (step) {
        const choice = step.result;
        if (choice.value.match(/process order/ig)) {
            if (step.values.orderCart.orders.length > 0) {
                // Process the order
                await step.context.sendActivity("Processing your order.");
                // ...
                step.values.orderCart = undefined; // Reset cart
                return await step.endDialog();
            } else {
                await step.context.sendActivity("Your cart was empty. Please add at least one item to the cart.");
                // Ask again
                return await step.replaceDialog('orderPrompt', step.values.orderCart);
            }
        } else if (choice.value.match(/cancel/ig)) {
            await step.context.sendActivity("Your order has been canceled.");
            return await step.endDialog(choice.value);
        } else {
            var item = dinnerMenu[choice.value];

            // Only proceed if user chooses an item from the menu
            if (!item) {
                await step.context.sendActivity("Sorry, that is not a valid item. Please pick one from the menu.");

                // Ask again
                return await step.replaceDialog('orderPrompt', step.values.orderCart);
            } else {
                // Add the item to cart
                step.values.orderCart.orders.push(item);
                step.values.orderCart.total += item.Price;

                await step.context.sendActivity(`Added to cart: ${choice.value}. <br/>Current total: $${step.values.orderCart.total}`);

                // Ask again
                return await step.replaceDialog('orderPrompt', step.values.orderCart);
            }
        }
    }
]));
```

L’exemple de code ci-dessus montre que le dialogue `orderDinner` principal utilise un dialogue d’assistance nommé `orderPrompt` pour traiter les choix de l’utilisateur. Le dialogue `orderPrompt` affiche le menu, invite l’utilisateur à choisir un article, ajoute celui-ci au panier, puis invite à nouveau l’utilisateur en créant une boucle. Cela permet à l’utilisateur d’ajouter plusieurs articles à sa commande. Le dialogue se répète en boucle jusqu’à ce que l’utilisateur choisisse `Process order` ou `Cancel`. À ce moment-là, l’exécution est rendue au dialogue parent (le dialogue `orderDinner`). Le dialogue `orderDinner` fait un ménage de dernière minute si l’utilisateur souhaite traiter la commande. Autrement, il se termine et rend l’exécution à son dialogue parent (par exemple, `mainMenu`). Le dialogue `mainMenu` continue à son tour l’exécution de la dernière étape qui consiste simplement à réafficher les choix du menu principal.

---

### <a name="create-the-reserve-table-dialog"></a>Créer le dialogue de réservation d’une table

<!-- TODO: Update the "Manage simple conversation flows" topic to use a reserveTable dialog, and then reference that from here. -->

Pour que l’exemple reste simple, nous ne montrons ici qu’une implémentation minimale du dialogue `reserveTable`.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Dans le constructeur, ajoutez le dialogue en cascade. Définissez ensuite les étapes de la cascade. Ici, nous les avons définies dans une classe imbriquée.

Dans le constructeur **HotelDialogs**.

```csharp
// Define the steps for and add the reserve-table dialog.
WaterfallStep[] reserveTableDialogSteps = new WaterfallStep[]
{
    ReserveTableSteps.StubAsync,
};

Add(new WaterfallDialog(Dialogs.ReserveTable, reserveTableDialogSteps));
```

Dans la classe **HotelDialogs**, ajoutez les définitions de l’étape.

```csharp
/// <summary>
/// Contains the waterfall dialog steps for the reserve-table dialog.
/// </summary>
private static class ReserveTableSteps
{
    public static async Task<DialogTurnResult> StubAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken)
    {
        await stepContext.Context.SendActivityAsync(
            "Your table has been reserved.",
            cancellationToken: cancellationToken);

        return await stepContext.EndDialogAsync(null, cancellationToken);
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Dans le constructeur du bot, ajoutez le dialogue en cascade `reserveTable`.

```javascript
// Reserve a table:
// Help the user to reserve a table
this.dialogSet.add(new WaterfallDialog('reserveTable', [
    // Replace this waterfall with your reservation steps.
    async function(step){
        await step.context.sendActivity("Your table has been reserved");
        await step.endDialog();
    }
]));
```

---

### <a name="update-the-bot-code-to-call-the-dialogs"></a>Mettre à jour le code du bot pour l’appel des dialogues

Mettez à jour le code du gestionnaire de tour du bot pour appeler le dialogue.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Renommez **EchoBotAccessors.cs** en **BotAccessors.cs** et remplacez également le nom de classe `EchoBotAccessors` par `BotAccessors`. Mettez à jour les instructions using et la définition de classe pour fournir l’accesseur de propriété d’état dont nous avons besoin pour ce bot.

```csharp
using System;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Dialogs;
```

```csharp
/// <summary>
/// This class is created as a Singleton and passed into the IBot-derived constructor.
///  - See <see cref="EchoWithCounterBot"/> constructor for how that is injected.
///  - See the Startup.cs file for more details on creating the Singleton that gets
///    injected into the constructor.
/// </summary>
public class BotAccessors
{
    /// <summary>
    /// Initializes a new instance of the <see cref="BotAccessors"/> class.
    /// Contains the <see cref="ConversationState"/> and associated <see cref="IStatePropertyAccessor{T}"/>.
    /// </summary>
    /// <param name="conversationState">The state object that stores the counter.</param>
    public BotAccessors(ConversationState conversationState)
    {
        ConversationState = conversationState ?? throw new ArgumentNullException(nameof(conversationState));
    }

    /// <summary>
    /// Gets the <see cref="IStatePropertyAccessor{T}"/> name used for the <see cref="DialogState"/> accessor.
    /// </summary>
    /// <remarks>Accessors require a unique name.</remarks>
    /// <value>The accessor name for the dialog state accessor.</value>
    public static string DialogStateAccessorName { get; } = $"{nameof(BotAccessors)}.DialogState";

    /// <summary>
    /// Gets or sets the DialogState property accessor.
    /// </summary>
    /// <value>
    /// The DialogState property accessor.
    /// </value>
    public IStatePropertyAccessor<DialogState> DialogStateAccessor { get; set; }

    /// <summary>
    /// Gets the <see cref="ConversationState"/> object for the conversation.
    /// </summary>
    /// <value>The <see cref="ConversationState"/> object.</value>
    public ConversationState ConversationState { get; }
}
```

Mettez à jour le fichier **Startup.cs** pour configurer le singleton `BotAccessors`.

1. Mettez à jour les instructions using.

    ```csharp
    using System;
    using System.Linq;
    using Microsoft.AspNetCore.Builder;
    using Microsoft.AspNetCore.Hosting;
    using Microsoft.Bot.Builder;
    using Microsoft.Bot.Builder.Dialogs;
    using Microsoft.Bot.Builder.Integration;
    using Microsoft.Bot.Builder.Integration.AspNet.Core;
    using Microsoft.Bot.Configuration;
    using Microsoft.Bot.Connector.Authentication;
    using Microsoft.Extensions.Configuration;
    using Microsoft.Extensions.DependencyInjection;
    using Microsoft.Extensions.Logging;
    using Microsoft.Extensions.Options;
    ```

1. Mettez à jour la partie de la méthode `ConfigureServices` qui enregistre les accesseurs de propriété d’état du bot.

    ```csharp
    // Create and register state accesssors.
    // Acessors created here are passed into the IBot-derived class on every turn.
    services.AddSingleton<BotAccessors>(sp =>
    {
        var options = sp.GetRequiredService<IOptions<BotFrameworkOptions>>().Value;
        var conversationState = options.State.OfType<ConversationState>().FirstOrDefault();

        // Create the custom state accessor.
        // State accessors enable other components to read and write individual properties of state.
        var accessors = new BotAccessors(conversationState)
        {
            DialogStateAccessor = conversationState.CreateProperty<DialogState>(BotAccessors.DialogStateAccessorName),
        };

        return accessors;
    });
    ```

Renommez le fichier EchoWithCounterBot.cs en HotelBot.cs, et remplacez la classe EchoWithCounterBot par HotelBot.

1. Mettez à jour le code d’initialisation du bot.

    ```csharp
    private readonly BotAccessors _accessors;
    private readonly HotelDialogs _dialogs;
    private readonly ILogger _logger;

    /// <summary>
    /// Initializes a new instance of the <see cref="HotelBot"/> class.
    /// </summary>
    /// <param name="accessors">A class containing <see cref="IStatePropertyAccessor{T}"/> used to manage state.</param>
    /// <param name="loggerFactory">A <see cref="ILoggerFactory"/> that is hooked to the Azure App Service provider.</param>
    public HotelBot(BotAccessors accessors, ILoggerFactory loggerFactory)
    {
        _logger = loggerFactory.CreateLogger<HotelBot>();
        _logger.LogTrace("EchoBot turn start.");
        _accessors = accessors;
        _dialogs = new HotelDialogs(_accessors.DialogStateAccessor);
    }
    ```

1. Mettez à jour le gestionnaire de tour du bot, afin qu’il exécute le dialogue.

    ```csharp
    public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
    {
        var dc = await _dialogs.CreateContextAsync(turnContext, cancellationToken);

        if (turnContext.Activity.Type == ActivityTypes.Message)
        {
            await dc.ContinueDialogAsync(cancellationToken);
            if (!turnContext.Responded)
            {
                await dc.BeginDialogAsync(HotelDialogs.MainMenu, null, cancellationToken);
            }
        }
        else if (turnContext.Activity.Type == ActivityTypes.ConversationUpdate)
        {
            var activity = turnContext.Activity.AsConversationUpdateActivity();
            if (activity.MembersAdded.Any(member => member.Id != activity.Recipient.Id))
            {
                await dc.BeginDialogAsync(HotelDialogs.MainMenu, null, cancellationToken);
            }
        }

        await _accessors.ConversationState.SaveChangesAsync(turnContext, false, cancellationToken);
    }
    ```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
async onTurn(turnContext) {
    let dc = await this.dialogSet.createContext(turnContext);

    // See https://aka.ms/about-bot-activity-message to learn more about the message and other activity types.
    if (turnContext.activity.type === ActivityTypes.Message) {

        await dc.continueDialog();

        if (!turnContext.responded) {
            await dc.beginDialog('mainMenu');
        }
    } else if (turnContext.activity.type === ActivityTypes.ConversationUpdate) {
        // Do we have any new members added to the conversation?
        if (turnContext.activity.membersAdded.length !== 0) {
            // Iterate over all new members added to the conversation
            for (var idx in turnContext.activity.membersAdded) {
                // Greet anyone that was not the target (recipient) of this message.
                // Since the bot is the recipient for events from the channel,
                // context.activity.membersAdded === context.activity.recipient.Id indicates the
                // bot was added to the conversation, and the opposite indicates this is a user.
                if (turnContext.activity.membersAdded[idx].id !== turnContext.activity.recipient.id) {
                    // Start the dialog.
                    await dc.beginDialog('mainMenu');
                }
            }
        }
    }

    // Save state changes
    await this.conversationState.saveChanges(turnContext);
}
```

---

## <a name="next-steps"></a>Étapes suivantes

Vous pouvez améliorer ce bot en offrant d’autres options comme « plus d’informations » ou « aide » comme choix du menu. Pour en savoir plus sur l’implémentation de ce type d’interruptions, consultez [Gérer les interruptions utilisateur](bot-builder-howto-handle-user-interrupt.md).

À présent que vous avez appris à utiliser des dialogues, des invites et des cascades pour gérer des flux de conversation complexes, voyons comment décomposer les dialogues (comme les dialogues `orderDinner` et `reserveTable`) en objets distincts au lieu de les regrouper tous dans un seul jeu de dialogues important.

> [!div class="nextstepaction"]
> [Créer une logique modulaire de bot avec contrôle composite](bot-builder-compositcontrol.md)
