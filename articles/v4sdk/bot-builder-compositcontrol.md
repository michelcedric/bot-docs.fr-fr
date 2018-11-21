---
title: Réutiliser des dialogues | Microsoft Docs
description: Apprenez à modulariser votre logique de bot à l’aide du conteneur de dialogue du kit SDK Bot Builder pour Node.js et C#.
keywords: contrôle composite, logique de bot modulaire
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 11/08/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 5e1ddbef6181e265213ffa1fdcb65909e524be51
ms.sourcegitcommit: cb0b70d7cf1081b08eaf1fddb69f7db3b95b1b09
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/09/2018
ms.locfileid: "51332993"
---
# <a name="reuse-dialogs"></a>Réutiliser des dialogues

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Imaginez que vous créez un bot d’hôtel qui gère plusieurs tâches telles que l’accueil de l’utilisateur, la réservation d’une table dans un restaurant, la commande d’un plat, le réglage d’une alarme, l’affichage de la météo, et bien d’autres tâches. Vous pouvez gérer chacune de ces tâches au sein de votre bot à l’aide d’un objet de dialogue, mais cela risque de rendre votre code de dialogue trop volumineux et confus.

Vous pouvez activer des parties de votre flux de conversation dans les dialogues composants, en divisant un grand jeu de dialogues en portions plus facilement gérables. Cela peut simplifier votre code et faciliter son débogage, et permettre à plusieurs équipes de travailler simultanément sur le bot.
Vous pouvez également créer une bibliothèque de dialogues composants pour une réutilisation dans d’autres jeux de dialogues et d’autres bots.

Dans cet exemple, nous allons créer un bot d’hôtel qui combine des dialogues composants d’enregistrement, de service de réveil et de réservation de table.

Cet article s’appuie sur les informations sur la gestion de flux de conversation [simples](bot-builder-dialog-manage-conversation-flow.md) et [complexes](bot-builder-dialog-manage-complex-conversation-flow.md).

## <a name="create-your-project"></a>Créer votre projet

Nous commençons avec le modèle de bot d’écho et nous incluons la bibliothèque de dialogues au projet.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Nous allons partir du modèle **EchoBot**. Pour obtenir des instructions, consultez le [démarrage rapide pour .NET](../dotnet/bot-builder-dotnet-sdk-quickstart.md).

Pour utiliser des dialogues, vous devez installer le package NuGet `Microsoft.Bot.Builder.Dialogs` pour votre projet ou solution.
Référencez ensuite la bibliothèque de dialogues dans les instructions d’utilisation de vos fichiers de code.

Renommez le fichier **EchoBotWithCounter.cs** en **HotelBot.cs**, et renommez la classe **HotelBot**.

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Nous allons partir du modèle **Echo**. Pour obtenir des instructions, consultez le [démarrage rapide pour JavaScript](../javascript/bot-builder-javascript-quickstart.md).

Vous pouvez télécharger la bibliothèque `botbuilder-dialogs` à partir de npm. Pour installer la bibliothèque `botbuilder-dialogs`, exécutez la commande npm suivante :

```cmd
npm install --save botbuilder-dialogs
```

---

## <a name="managing-state"></a>Gestion de l’état

Il existe de nombreuses façons de configurer la gestion d’état pour un bot utilisant des dialogues composites. Dans ce bot, chaque dialogue composant retourne un objet comme résultat du dialogue. Le contexte d’appel gère les valeurs renvoyées. Pour plus d’informations sur la gestion des données d’état, consultez la rubrique [Enregistrer l’état à l’aide des propriétés de conversation et d’utilisateur](bot-builder-howto-v4-state.md).

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Chaque dialogue recueille des informations que le gestionnaire de tour ou le menu principal du bot enregistre dans l’état utilisateur. Nous allons définir des dialogues composants pour l’enregistrement, la réservation de table et la définition d’un réveil. Chacun d’eux retournera un objet de la classe appropriée. Ajoutez chacune des classes suivantes à votre projet en tant que module de classe C#.

```csharp
/// <summary>
/// User state information.
/// </summary>
public class UserInfo
{
    public GuestInfo Guest { get; set; }
    public TableInfo Table { get; set; }
    public WakeUpInfo WakeUp { get; set; }
}

/// <summary>
/// State information associated with the check-in dialog.
/// </summary>
public class GuestInfo
{
    public string Name { get; set; }
    public string Room { get; set; }
}

/// <summary>
/// State information associated with the reserve-table dialog.
/// </summary>
public class TableInfo
{
    public string Number { get; set; }
}

/// <summary>
/// State information associated with the wake-up call dialog.
/// </summary>
public class WakeUpInfo
{
    public string Time { get; set; }
}
```

Renommez le fichier **EchoBotAccessors.cs** en **BotAccessors.cs** et mettez également à jour le nom de classe en la remplaçant par **BotAccessors**.

Mettez ensuite à jour le fichier à l’aide de ce code. Nous avons besoin d’accesseurs pour l’état du dialogue et les informations utilisateur collectées.

```csharp
using System;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Dialogs;

/// <summary>
/// Contains the state objects and the state property accessors for the bot.
/// </summary>
public class BotAccessors
{
    // The property accessor keys to use.
    public const string UserInfoAccessorName = "HotelBot.UserInfo";
    public const string DialogStateAccessorName = "HotelBot.DialogState";

    /// <summary>
    /// Initializes a new instance of the <see cref="BotAccessors"/> class.
    /// Contains the <see cref="ConversationState"/> and associated <see cref="IStatePropertyAccessor{T}"/>.
    /// </summary>
    /// <param name="conversationState">The state object that stores the counter.</param>
    public BotAccessors(ConversationState conversationState, UserState userState)
    {
        ConversationState = conversationState ?? throw new ArgumentNullException(nameof(conversationState));
        UserState = userState ?? throw new ArgumentNullException(nameof(userState));
    }

    /// <summary>Gets or sets the state property accessor for the user information we're tracking.</summary>
    /// <value>Accessor for user information.</value>
    public IStatePropertyAccessor<UserInfo> UserInfoAccessor { get; set; }

    /// <summary>Gets or sets the state property accessor for the dialog state.</summary>
    /// <value>Accessor for the dialog state.</value>
    public IStatePropertyAccessor<DialogState> DialogStateAccessor { get; set; }

    /// <summary>Gets the conversation state for the bot.</summary>
    /// <value>The conversation state for the bot.</value>
    public ConversationState ConversationState { get; }

    /// <summary>Gets the user state for the bot.</summary>
    /// <value>The user state for the bot.</value>
    public UserState UserState { get; }
}
```

Dans le fichier **Startup.cs**, mettez à jour le code dans la méthode `ConfigureServices` qui configure l’état et les accesseurs de propriété d’état du bot.

```csharp
using Microsoft.Bot.Builder.Dialogs;

public void ConfigureServices(IServiceCollection services)
{
    services.AddBot<HotelBot>(options =>
    {
        //...

        // The Memory Storage used here is for local bot debugging only. When the bot
        // is restarted, everything stored in memory will be gone.
        IStorage dataStore = new MemoryStorage();

        // Create conversation and user state objects.
        options.State.Add(new ConversationState(dataStore));
        options.State.Add(new UserState(dataStore));
    });

    // Create and register state accessors.
    // Accessors created here are passed into the IBot-derived class on every turn.
    services.AddSingleton<BotAccessors>(sp =>
    {
        var options = sp.GetRequiredService<IOptions<BotFrameworkOptions>>().Value;
        var conversationState = options.State.OfType<ConversationState>().FirstOrDefault();
        var userState = options.State.OfType<UserState>().FirstOrDefault();

        // Create the custom state accessor.
        // State accessors enable other components to read and write individual properties of state.
        var accessors = new BotAccessors(conversationState, userState)
        {
            UserInfoAccessor = userState.CreateProperty<UserInfo>(BotAccessors.UserInfoAccessorName),
            DialogStateAccessor = conversationState.CreateProperty<DialogState>(BotAccessors.DialogStateAccessorName),
        };

        return accessors;
    });
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Chaque dialogue recueille des informations que le gestionnaire de tour ou le menu principal du bot enregistre dans l’état utilisateur. Nous allons définir des dialogues composants pour l’enregistrement, la réservation de table et la définition d’un réveil. Chacun d’eux retournera un objet avec les propriétés appropriées. Nous regrouperons ces propriétés dans l’état utilisateur du bot.

Dans votre fichier **bot.js**, mettez à jour le constructeur de votre bot pour créer des accesseurs de propriété d’état qui suivront l’état utilisateur et l’état du dialogue.

```javascript
// Define the identifiers for our state property accessors.
const DIALOG_STATE_PROPERTY = 'dialogStatePropertyAccessor';
const USER_INFO_PROPERTY = 'userInfoPropertyAccessor';

constructor(conversationState, userState) {
    // Record the conversation and user state management objects.
    this.conversationState = conversationState;
    this.userState = userState;

    // Create our state property accessors.
    this.dialogStateAccessor = conversationState.createProperty(DIALOG_STATE_PROPERTY);
    this.userInfoAccessor = userState.createProperty(USER_INFO_PROPERTY);
}
```

Dans votre fichier **index.js**, mettez à jour les classes importées à partir de la bibliothèque `botbuilder` et le code qui crée les objets d’état et votre bot :

```javascript
// Import required bot services.
const { BotFrameworkAdapter, MemoryStorage, ConversationState, UserState } = require('botbuilder');
```

```javascript
// Define state store for your bot.
const memoryStorage = new MemoryStorage();

// Create conversation and user state with in-memory storage provider.
const conversationState = new ConversationState(memoryStorage);
const userState = new UserState(memoryStorage);

// Create the bot.
const myBot = new MyBot(conversationState, userState);
```

---

## <a name="define-the-check-in-component-dialog"></a>Définir le dialogue composant d’enregistrement

Tout d’abord, commençons avec un simple dialogue d’enregistrement, qui demande à l’utilisateur son nom et la chambre qu’il souhaite occuper. Nous créons une classe `CheckInDialog` qui étend `ComponentDialog`. Cette classe comporte un constructeur qui définit le nom du dialogue racine, que nous définissons comme un dialogue en cascade à trois étapes.

Les étapes du dialogue d’enregistrement sont les suivantes.

1. Demander le nom du client.
1. Demander la chambre qu’il souhaite occuper.
1. Envoyer un message de confirmation et finaliser le dialogue.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Ajoutez une classe `CheckInDialog` à votre projet en utilisant le code ci-dessous.

Tout au long de ce dialogue, nous pouvons inscrire des données dans un objet d’état local, accessible via la propriété `Values` du contexte d’étape. Une fois le dialogue terminé, l’objet d’état local est supprimé. Par conséquent, nous renvoyons une valeur du dialogue qui contient les informations du client.

```csharp
using System.Threading;
using System.Threading.Tasks;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Dialogs;

public class CheckInDialog : ComponentDialog
{
    private const string GuestKey = nameof(CheckInDialog);
    private const string TextPrompt = "textPrompt";

    // You can start this from the parent using the dialog's ID.
    public CheckInDialog(string id)
        : base(id)
    {
        InitialDialogId = Id;

        // Define the prompts used in this conversation flow.
        AddDialog(new TextPrompt(TextPrompt));

        // Define the conversation flow using a waterfall model.
        WaterfallStep[] waterfallSteps = new WaterfallStep[]
        {
            NameStepAsync,
            RoomStepAsync,
            FinalStepAsync,
        };
        AddDialog(new WaterfallDialog(Id, waterfallSteps));
    }

    private static async Task<DialogTurnResult> NameStepAsync(
        WaterfallStepContext step,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        // Clear the guest information and prompt for the guest's name.
        step.Values[GuestKey] = new GuestInfo();
        return await step.PromptAsync(
            TextPrompt,
            new PromptOptions
            {
                Prompt = MessageFactory.Text("What is your name?"),
            },
            cancellationToken);
    }

    private static async Task<DialogTurnResult> RoomStepAsync(
        WaterfallStepContext step,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        // Save the name and prompt for the room number.
        string name = step.Result as string;
        ((GuestInfo)step.Values[GuestKey]).Name = name;
        return await step.PromptAsync(
            TextPrompt,
            new PromptOptions
            {
                Prompt = MessageFactory.Text($"Hi {name}. What room will you be staying in?"),
            },
            cancellationToken);
    }

    private static async Task<DialogTurnResult> FinalStepAsync(
        WaterfallStepContext step,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        // Save the room number and "sign off".
        string room = step.Result as string;
        ((GuestInfo)step.Values[GuestKey]).Room = room;

        await step.Context.SendActivityAsync(
            "Great, enjoy your stay!",
            cancellationToken: cancellationToken);

        // End the dialog, returning the guest info.
        return await step.EndDialogAsync(
            (GuestInfo)step.Values[GuestKey],
            cancellationToken);
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Créez un fichier **checkInDialog.js**, et ajoutez-y une classe `CheckInDialog` étendant `ComponentDialog`.

Tout au long de ce dialogue, nous pouvons inscrire des données dans un objet d’état local, accessible via la propriété `values` du contexte d’étape. Une fois le dialogue terminé, l’objet d’état local est supprimé. Par conséquent, nous renvoyons une valeur du dialogue qui contient les informations du client.

```JavaScript
const { ComponentDialog, TextPrompt, WaterfallDialog } = require('botbuilder-dialogs');

class CheckInDialog extends ComponentDialog {
    constructor(dialogId) {
        super(dialogId);

        // ID of the child dialog that should be started anytime the component is started.
        this.initialDialogId = dialogId;

        // Define the prompts used in this conversation flow.
        this.addDialog(new TextPrompt('textPrompt'));

        // Define the conversation flow using a waterfall model.
        this.addDialog(new WaterfallDialog(dialogId, [
            async function (step) {
                // Clear the guest information and prompt for the guest's name.
                step.values.guestInfo = {};
                return await step.prompt('textPrompt', "What is your name?");
            },
            async function (step) {
                // Save the name and prompt for the room number.
                step.values.guestInfo.userName = step.result;
                return await step.prompt('textPrompt', `Hi ${step.result}. What room will you be staying in?`);
            },
            async function (step) {
                // Save the room number and "sign off".
                step.values.guestInfo.roomNumber = step.result;
                await step.context.sendActivity(`Great! Enjoy your stay in room ${step.result}!`);

                // End the dialog, returning the guest info.
                return await step.endDialog(step.values.guestInfo);
            }
        ]));
    }
}

exports.CheckInDialog = CheckInDialog;
```

---

## <a name="define-the-reserve-table-and-wake-up-component-dialogs"></a>Définir les dialogues composants de réservation de table et de réveil

L’un des avantages de l’utilisation d’un dialogue composant est la possibilité d’utiliser plusieurs dialogues différents à la fois. Comme chaque `DialogSet` gère un ensemble exclusif de `dialogs`, cela complique le partage ou la référence croisée de `dialogs`. C’est là qu’intervient le dialogue composant. Vous pouvez encapsuler les aspects complexes d’un flux de conversation dans des dialogues composants, ce qui peut faciliter la gestion et la maintenance des dialogues. Nous allons créer deux autres dialogues composants : un pour demander à l’utilisateur quelle table il souhaite réserver pour le dîner, et un autre pour demander un service de réveil. Là encore, nous allons utiliser une classe distincte pour chaque dialogue, et chaque dialogue étendra l’élément principal `ComponentDialog`.

Nous allons faire en sorte qu’ils acceptent les informations des clients dans les options de dialogue au démarrage.

Les étapes du dialogue de réservation de table sont les suivantes.

1. Demander la réservation d’une table.
1. Envoyer un message de confirmation et finaliser le dialogue, renvoyer le numéro de table.

Les étapes du dialogue de demande de service de réveil sont les suivantes.

1. Demander l’heure pour le service de réveil.
1. Envoyer un message de confirmation et finaliser le dialogue, renvoyer l’heure du réveil.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Ajoutez une classe `ReserveTableDialog` à votre projet en utilisant le code ci-dessous.

Nous obtenons le nom du client dans l’objet d’options passé au démarrage du dialogue. Ensuite, nous renvoyons une valeur dans le dialogue, qui contient le numéro de table cette fois.

```csharp
using System.Threading;
using System.Threading.Tasks;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Bot.Builder.Dialogs.Choices;

public class ReserveTableDialog : ComponentDialog
{
    private const string TablePrompt = "choicePrompt";

    public ReserveTableDialog(string id)
        : base(id)
    {
        InitialDialogId = Id;

        // Define the prompts used in this conversation flow.
        AddDialog(new ChoicePrompt(TablePrompt));

        // Define the conversation flow using a waterfall model.
        WaterfallStep[] waterfallSteps = new WaterfallStep[]
        {
                TableStepAsync,
                FinalStepAsync,
        };
        AddDialog(new WaterfallDialog(Id, waterfallSteps));
    }

    private static async Task<DialogTurnResult> TableStepAsync(
        WaterfallStepContext step,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        string greeting = step.Options is GuestInfo guest
                && !string.IsNullOrWhiteSpace(guest?.Name)
                ? $"Welcome {guest.Name}" : "Welcome";

        string prompt = $"{greeting}, How many diners will be at your table?";
        string[] choices = new string[] { "1", "2", "3", "4", "5", "6" };
        return await step.PromptAsync(
            TablePrompt,
            new PromptOptions
            {
                Prompt = MessageFactory.Text(prompt),
                Choices = ChoiceFactory.ToChoices(choices),
            },
            cancellationToken);
    }

    private static async Task<DialogTurnResult> FinalStepAsync(
        WaterfallStepContext step,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        // ChoicePrompt returns a FoundChoice object.
        string table = (step.Result as FoundChoice).Value;

        // Send a confirmation message.
        await step.Context.SendActivityAsync(
            $"Sounds great;  we will reserve a table for you for {table} diners.",
            cancellationToken: cancellationToken);

        // End the dialog, returning the table info.
        return await step.EndDialogAsync(
            new TableInfo { Number = table },
            cancellationToken);
    }
}
```

Ajoutez une classe `SetAlarmDialog` à votre projet en utilisant le code ci-dessous.

Nous obtenons le numéro de chambre du client dans l’objet d’options passé au démarrage du dialogue. Ensuite, nous renvoyons une valeur dans le dialogue, qui contient l’heure à laquelle le client souhaite être réveillé.

```csharp
using System.Collections.Generic;
using System.Linq;
using System.Threading;
using System.Threading.Tasks;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Dialogs;

public class SetAlarmDialog : ComponentDialog
{
    private const string AlarmPrompt = "dateTimePrompt";

    public SetAlarmDialog(string id)
        : base(id)
    {
        InitialDialogId = Id;

        // Define the prompts used in this conversation flow.
        // Ideally, we'd add validation to this prompt.
        AddDialog(new DateTimePrompt(AlarmPrompt));

        // Define the conversation flow using a waterfall model.
        WaterfallStep[] waterfallSteps = new WaterfallStep[]
        {
                AlarmStepAsync,
                FinalStepAsync,
        };

        AddDialog(new WaterfallDialog(Id, waterfallSteps));
    }

    private static async Task<DialogTurnResult> AlarmStepAsync(
        WaterfallStepContext step,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        string greeting = step.Options is GuestInfo guest
                && !string.IsNullOrWhiteSpace(guest?.Name)
                ? $"Hi {guest.Name}" : "Hi";

        string prompt = $"{greeting}. When would you like your alarm set for?";
        return await step.PromptAsync(
            AlarmPrompt,
            new PromptOptions { Prompt = MessageFactory.Text(prompt) },
            cancellationToken);
    }

    private static async Task<DialogTurnResult> FinalStepAsync(
        WaterfallStepContext step,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        // Ambiguous responses can generate multiple results.
        var resolution = (step.Result as IList<DateTimeResolution>)?.FirstOrDefault();

        // Time ranges have a start and no value.
        var alarm = resolution.Value ?? resolution.Start;
        string roomNumber = (step.Options as GuestInfo)?.Room;

        // Send a confirmation message.
        await step.Context.SendActivityAsync(
            $"Your alarm is set to {alarm} for room {roomNumber}.",
            cancellationToken: cancellationToken);

        // End the dialog, returning the alarm info.
        return await step.EndDialogAsync(
            new WakeUpInfo { Time = alarm },
            cancellationToken);
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Créez un fichier **reserveTableDialog.js**, et ajoutez-y une classe `ReserveTableDialog` étendant `ComponentDialog`.

Nous obtenons le nom du client dans l’objet d’options passé au démarrage du dialogue. Ensuite, nous renvoyons une valeur dans le dialogue, qui contient le numéro de table cette fois.

```JavaScript
const { ComponentDialog, ChoicePrompt, WaterfallDialog } = require('botbuilder-dialogs');

class ReserveTableDialog extends ComponentDialog {
    constructor(dialogId) {
        super(dialogId);

        // ID of the child dialog that should be started anytime the component is started.
        this.initialDialogId = dialogId;

        // Define the prompts used in this conversation flow.
        this.addDialog(new ChoicePrompt('choicePrompt'));

        // Define the conversation flow using a waterfall model.
        this.addDialog(new WaterfallDialog(dialogId, [
            async function (step) {
                // Welcome the user and ask for their table preference.
                const greeting = step.options && step.options.userName ? `Welcome ${step.options.userName}` : `Welcome`;

                const promptOptions = {
                    prompt: `${greeting}, How many diners will be at your table?`,
                    reprompt: 'That was not a valid choice, please select a table size between 1 and 6 guests.',
                    choices: ['1', '2', '3', '4', '5', '6']
                };
                return await step.prompt('choicePrompt', promptOptions);
            },
            async function (step) {
                const choice = step.result;

                // Send a confirmation message.
                const tableNumber = choice.value;
                await step.context.sendActivity(`Sounds great, we will reserve a table for you for ${tableNumber} diners.`);

                // End the dialog, returning the table info.
                return await step.endDialog({ table: tableNumber });
            }
        ]));
    }
}

exports.ReserveTableDialog = ReserveTableDialog;
```

Créez un fichier **setAlarmDialog.js**, et ajoutez-y une classe `SetAlarmDialog` étendant `ComponentDialog`.

Nous obtenons le numéro de chambre du client dans l’objet d’options passé au démarrage du dialogue. Ensuite, nous renvoyons une valeur dans le dialogue, qui contient l’heure à laquelle le client souhaite être réveillé.

```JavaScript
const { ComponentDialog, DateTimePrompt, WaterfallDialog } = require('botbuilder-dialogs');

class SetAlarmDialog extends ComponentDialog {
    constructor(dialogId) {
        super(dialogId);

        // ID of the child dialog that should be started anytime the component is started.
        this.initialDialogId = dialogId;

        // Define the prompts used in this conversation flow.
        this.dialogs.add(new DateTimePrompt('datePrompt'));

        this.dialogs.add(new WaterfallDialog(dialogId, [
            async function (step) {
                step.values.wakeUp = {};
                if (step.options && step.options.roomNumber) {
                    step.values.roomNumber = step.options.roomNumber;
                }

                const greeting = step.options && step.options.userName ? `Hi ${step.options.userName}` : `Hi`;
                return await step.prompt('datePrompt', `${greeting}. What time would you like your alarm to be set?`);
            },
            async function (step) {
                // Ambiguous responses can generate multiple results.
                const resolution = step.result[0];

                // Time ranges have a start and no value.
                const alarm = resolution.value ? resolution.value : resolution.start;
                const roomNumber = step.values.roomNumber;

                // Send a confirmation message.
                await step.context.sendActivity(`Your alarm is set to ${alarm} for room ${roomNumber}.`);

                // End the dialog, returning the alarm info.
                return await step.endDialog({ alarm: alarm });
            }]));

        // Defining the prompt used in this conversation flow
    }
}

exports.SetAlarmDialog = SetAlarmDialog;
```

---

## <a name="add-the-component-dialogs-to-the-bot"></a>Ajouter les dialogues composants au bot

Maintenant que nous avons défini nos trois dialogues composants, nous pouvons les utiliser dans notre bot.

- Ces dialogues sont ajoutés au jeu de dialogues principal de notre bot.
- Lorsqu’une nouvelle conversation démarre, aucun dialogue n’est actif et la logique de tour du bot prend le relais.
- Sans informations sur le client, le dialogue d’enregistrement démarre.
- Une fois que nous disposons des informations sur le client, le dialogue principal prend le relais, permettant à plusieurs reprises à l’utilisateur de démarrer le dialogue de réservation de table ou de service de réveil.

Nous allons mettre à jour la logique du gestionnaire de tour du bot.

1. Obtenir l’état utilisateur.
1. Continuer tous les dialogues actifs.
1. Si un dialogue est terminé pendant ce tour, il devrait s’agir du dialogue d’enregistrement.
   1. Stocker les informations sur le client.
   1. Démarrer le dialogue principal.
1. Si le bot n’a pas encore envoyé de message à l’utilisateur, alors aucun dialogue actif n’est en cours.
    1. Sans informations sur le client, démarrer le dialogue d’enregistrement.
    1. Sinon, démarrer le dialogue principal.
1. Enregistrer toutes les modifications d’état éventuelles.

Les étapes du dialogue principal sont les suivantes.

1. Demander au client ce qu’il souhaite faire : réserver une table ou demander un réveil par téléphone.
1. Démarrer le dialogue enfant correspondant, ou envoyer un message _input not understood_ (entrée incompréhensible), puis recommencer à partir de la première étape.
1. Traiter la valeur de retour du dialogue enfant et recommencer le dialogue principal.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Dans le fichier **HotelBot.cs**, mettez à jour les instructions using.

```csharp
using System.Collections.Generic;
using System.Threading;
using System.Threading.Tasks;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Bot.Schema;
using Microsoft.Extensions.Logging;
```

Mettez à jour le code d’initialisation et définissez le jeu de dialogues et le dialogue principal du bot.

```csharp
// Define the IDs for the dialogs in the bot's dialog set.
private const string MainDialogId = "mainDialog";
private const string CheckInDialogId = "checkInDialog";
private const string TableDialogId = "tableDialog";
private const string AlarmDialogId = "alarmDialog";

// Define the dialog set for the bot.
private readonly DialogSet _dialogs;

// Define the state accessors and the logger for the bot.
private readonly BotAccessors _accessors;
private readonly ILogger _logger;

/// <summary>
/// Initializes a new instance of the <see cref="HotelBot"/> class.
/// </summary>
/// <param name="accessors">Contains the objects to use to manage state.</param>
/// <param name="loggerFactory">A <see cref="ILoggerFactory"/> that is hooked to the Azure App Service provider.</param>
public HotelBot(BotAccessors accessors, ILoggerFactory loggerFactory)
{
    if (loggerFactory == null)
    {
        throw new System.ArgumentNullException(nameof(loggerFactory));
    }

    _logger = loggerFactory.CreateLogger<HotelBot>();
    _logger.LogTrace($"{nameof(HotelBot)} turn start.");
    _accessors = accessors ?? throw new System.ArgumentNullException(nameof(accessors));

    // Define the steps of the main dialog.
    WaterfallStep[] steps = new WaterfallStep[]
    {
        MenuStepAsync,
        HandleChoiceAsync,
        LoopBackAsync,
    };

    // Create our bot's dialog set, adding a main dialog and the three component dialogs.
    _dialogs = new DialogSet(_accessors.DialogStateAccessor)
        .Add(new WaterfallDialog(MainDialogId, steps))
        .Add(new CheckInDialog(CheckInDialogId))
        .Add(new ReserveTableDialog(TableDialogId))
        .Add(new SetAlarmDialog(AlarmDialogId));
}

private static async Task<DialogTurnResult> MenuStepAsync(
    WaterfallStepContext stepContext,
    CancellationToken cancellationToken = default(CancellationToken))
{
    // Present the user with a set of "suggested actions".
    List<string> menu = new List<string> { "Reserve Table", "Wake Up" };
    await stepContext.Context.SendActivityAsync(
        MessageFactory.SuggestedActions(menu, "How can I help you?"),
        cancellationToken: cancellationToken);
    return Dialog.EndOfTurn;
}

private async Task<DialogTurnResult> HandleChoiceAsync(
    WaterfallStepContext stepContext,
    CancellationToken cancellationToken = default(CancellationToken))
{
    // Get the user's info. (Since the type factory is null, this will throw if state does not yet have a value for user info.)
    UserInfo userInfo = await _accessors.UserInfoAccessor.GetAsync(stepContext.Context, null, cancellationToken);

    // Check the user's input and decide which dialog to start.
    // Pass in the guest info when starting either of the child dialogs.
    string choice = (stepContext.Result as string)?.Trim()?.ToLowerInvariant();
    switch (choice)
    {
        case "reserve table":
            return await stepContext.BeginDialogAsync(TableDialogId, userInfo.Guest, cancellationToken);

        case "wake up":
            return await stepContext.BeginDialogAsync(AlarmDialogId, userInfo.Guest, cancellationToken);

        default:
            // If we don't recognize the user's intent, start again from the beginning.
            await stepContext.Context.SendActivityAsync(
                "Sorry, I don't understand that command. Please choose an option from the list.");
            return await stepContext.ReplaceDialogAsync(MainDialogId, null, cancellationToken);
    }
}

private async Task<DialogTurnResult> LoopBackAsync(
    WaterfallStepContext stepContext,
    CancellationToken cancellationToken = default(CancellationToken))
{
    // Get the user's info. (Because the type factory is null, this will throw if state does not yet have a value for user info.)
    UserInfo userInfo = await _accessors.UserInfoAccessor.GetAsync(stepContext.Context, null, cancellationToken);

    // Process the return value from the child dialog.
    switch (stepContext.Result)
    {
        case TableInfo table:
            // Store the results of the reserve-table dialog.
            userInfo.Table = table;
            await _accessors.UserInfoAccessor.SetAsync(stepContext.Context, userInfo, cancellationToken);
            break;
        case WakeUpInfo alarm:
            // Store the results of the set-wake-up-call dialog.
            userInfo.WakeUp = alarm;
            await _accessors.UserInfoAccessor.SetAsync(stepContext.Context, userInfo, cancellationToken);
            break;
        default:
            // We shouldn't get here, since these are no other branches that get this far.
            break;
    }

    // Restart the main menu dialog.
    return await stepContext.ReplaceDialogAsync(MainDialogId, null, cancellationToken);
}
```

Mettez ensuite à jour le gestionnaire de tour du bot pour qu’il utilise le jeu de dialogues.

```csharp
public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
{
    if (turnContext.Activity.Type == ActivityTypes.Message)
    {
        // Establish dialog state from the conversation state.
        DialogContext dc = await _dialogs.CreateContextAsync(turnContext, cancellationToken);

        // Get the user's info.
        UserInfo userInfo = await _accessors.UserInfoAccessor.GetAsync(turnContext, () => new UserInfo(), cancellationToken);

        // Continue any current dialog.
        DialogTurnResult dialogTurnResult = await dc.ContinueDialogAsync();

        // Process the result of any complete dialog.
        if (dialogTurnResult.Status is DialogTurnStatus.Complete)
        {
            switch (dialogTurnResult.Result)
            {
                case GuestInfo guestInfo:
                    // Store the results of the check-in dialog.
                    userInfo.Guest = guestInfo;
                    await _accessors.UserInfoAccessor.SetAsync(turnContext, userInfo, cancellationToken);
                    break;
                default:
                    // We shouldn't get here, since the main dialog is designed to loop.
                    break;
            }
        }

        // Every dialog step sends a response, so if no response was sent,
        // then no dialog is currently active.
        else if (!turnContext.Responded)
        {
            if (string.IsNullOrEmpty(userInfo.Guest?.Name))
            {
                // If we don't yet have the guest's info, start the check-in dialog.
                await dc.BeginDialogAsync(CheckInDialogId, null, cancellationToken);
            }
            else
            {
                // Otherwise, start our bot's main dialog.
                await dc.BeginDialogAsync(MainDialogId, null, cancellationToken);
            }
        }

        // Save the new turn count into the conversation state.
        await _accessors.ConversationState.SaveChangesAsync(turnContext, false, cancellationToken);
        await _accessors.UserState.SaveChangesAsync(turnContext, false, cancellationToken);
    }
    else
    {
        await turnContext.SendActivityAsync($"{turnContext.Activity.Type} event detected");
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Dans notre fichier bot, **bot.js**, nous devons non seulement importer les classes que nous allons utiliser depuis le SDK, mais également les classes que nous avons définies pour nos dialogues composants.

```JavaScript
const { ActivityTypes, MessageFactory } = require('botbuilder');
const { DialogSet, WaterfallDialog, Dialog, DialogTurnStatus } = require('botbuilder-dialogs');

// Import our component dialogs.
const { CheckInDialog } = require("./checkInDialog");
const { ReserveTableDialog } = require("./reserveTableDialog");
const { SetAlarmDialog } = require("./setAlarmDialog");
```

Nous devrons également créer un jeu de dialogues et y ajouter tous les dialogues que nous allons utiliser.

Nous définissons les étapes de la cascade du dialogue principal en tant que fonctions dans la classe, et non pas en ligne. Nous utilisons `bind()` sur ces fonctions, de telle sorte qu’au sein de la fonction, `this` se résolve correctement.

Voici le constructeur de notre bot mis à jour.

```JavaScript
constructor(conversationState, userState) {
    // Record the conversation and user state management objects.
    this.conversationState = conversationState;
    this.userState = userState;

    // Create our state property accessors.
    this.dialogStateAccessor = conversationState.createProperty(DIALOG_STATE_PROPERTY);
    this.userInfoAccessor = userState.createProperty(USER_INFO_PROPERTY);

    // Create our bot's dialog set, adding a main dialog and the three component dialogs.
    this.dialogs = new DialogSet(this.dialogStateAccessor)
        .add(new CheckInDialog('checkInDialog'))
        .add(new ReserveTableDialog('reserveTableDialog'))
        .add(new SetAlarmDialog('setAlarmDialog'))
        .add(new WaterfallDialog('mainDialog', [
            this.promptForChoice.bind(this),
            this.startChildDialog.bind(this),
            this.saveResult.bind(this)
    ]));
}
```

Sous le constructeur du bot, ajoutez le code suivant. Il implémente les étapes du dialogue principal.

```JavaScript
async promptForChoice(step) {
    const menu = ["Reserve Table", "Wake Up"];
    await step.context.sendActivity(MessageFactory.suggestedActions(menu, 'How can I help you?'));
    return Dialog.EndOfTurn;
}

async startChildDialog(step) {
    // Get the user's info.
    const user = await this.userInfoAccessor.get(step.context);
    // Check the user's input and decide which dialog to start.
    // Pass in the guest info when starting either of the child dialogs.
    switch (step.result) {
        case "Reserve Table":
            return await step.beginDialog('reserveTableDialog', user.guestInfo);
            break;
        case "Wake Up":
            return await step.beginDialog('setAlarmDialog', user.guestInfo);
            break;
        default:
            await step.context.sendActivity("Sorry, I don't understand that command. Please choose an option from the list.");
            return await step.replaceDialog('mainDialog');
            break;
    }
}

async saveResult(step) {
    // Process the return value from the child dialog.
    if (step.result) {
        const user = await this.userInfoAccessor.get(step.context);
        if (step.result.table) {
            // Store the results of the reserve-table dialog.
            user.table = step.result.table;
        } else if (step.result.alarm) {
            // Store the results of the set-wake-up-call dialog.
            user.alarm = step.result.alarm;
        }
        await this.userInfoAccessor.set(step.context, user);
    }
    // Restart the main menu dialog.
    return await step.replaceDialog('mainDialog'); // Show the menu again
}
```

Mettez à présent le gestionnaire de tour de votre bot à jour :

```JavaScript
async onTurn(turnContext) {
    if (turnContext.activity.type === ActivityTypes.Message) {
        const user = await this.userInfoAccessor.get(turnContext, {});
        const dc = await this.dialogs.createContext(turnContext);
        const dialogTurnResult = await dc.continueDialog();
        if (dialogTurnResult.status === DialogTurnStatus.complete) {
            user.guestInfo = dialogTurnResult.result;
            await this.userInfoAccessor.set(turnContext, user);
            await dc.beginDialog('mainDialog');
        } else if (!turnContext.responded) {
            if (!user.guestInfo) {
                await dc.beginDialog('checkInDialog');
            } else {
                await dc.beginDialog('mainDialog');
            }
        }
        // Save state changes
        await this.conversationState.saveChanges(turnContext);
        await this.userState.saveChanges(turnContext);
    }
}
```

---

Comme vous pouvez le constater, les dialogues composants sont ajoutés au dialogue principal du bot de la même façon que des [invites](bot-builder-prompts.md) à un dialogue. Vous pouvez ajouter autant de dialogues enfants à votre dialogue principal que vous le souhaitez. Chaque module ajoute des fonctionnalités et des services supplémentaires que le bot peut offrir à vos utilisateurs.

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous savez comment utiliser les dialogues composants, voyons comment utiliser le Language Understanding (LUIS) pour aider votre bot à choisir le moment où il doit commencer les dialogues.

> [!div class="nextstepaction"]
> [Utiliser LUIS pour la compréhension de la langue](./bot-builder-howto-v4-luis.md)
