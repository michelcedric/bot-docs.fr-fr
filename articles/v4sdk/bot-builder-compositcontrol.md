---
title: Créer un ensemble intégré de dialogues | Microsoft Docs
description: Apprenez à modulariser votre logique de bot à l’aide du conteneur de dialogue du kit SDK Bot Builder pour Node.js et C#.
keywords: contrôle composite, logique de bot modulaire
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 4/27/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: b0ce3e6d11d20985e23c1bb74bd5b9dce8190d0c
ms.sourcegitcommit: 3bf3dbb1a440b3d83e58499c6a2ac116fe04b2f6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/23/2018
ms.locfileid: "46707155"
---
# <a name="create-an-integrated-set-of-dialogs"></a>Créer un ensemble intégré de dialogues

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Imaginez que vous créez un bot d’hôtel qui gère plusieurs tâches telles que l’accueil de l’utilisateur, la réservation d’une table dans un restaurant, la commande d’un plat, le réglage d’une alarme, l’affichage de la météo, et bien d’autres tâches. Vous pouvez gérer chacune de ces tâches au sein de votre bot à l’aide d’un objet de dialogue, mais cela risque de rendre votre code de dialogue trop volumineux et confus dans le fichier principal de votre bot.

Vous pouvez résoudre ce problème via la modularisation. La modularisation simplifiera votre code et le rendra plus facile à déboguer. En outre, vous pouvez le décomposer en sections pour permettre à plusieurs équipes de travailler simultanément sur le bot. Nous pouvons créer un bot qui gère plusieurs flux de conversation en les fractionnant en composants à l’aide d’un conteneur de dialogue. Nous allons créer quelques flux de conversation de base et montrer comment ils peuvent être combinés à l’aide d’un conteneur de dialogue.

Dans cet exemple, nous allons créer un bot d’hôtel qui combine des modules d’enregistrement, de service de réveil et de réservation de table.

## <a name="managing-state"></a>Gestion de l’état

Il existe de nombreuses façons de configurer la gestion d’état pour un bot utilisant des dialogues composites. Ce bot présente une méthode pour y parvenir.

Pour plus d’informations sur la gestion des données d’état, consultez la rubrique [Enregistrer l’état à l’aide des propriétés de conversation et d’utilisateur](bot-builder-howto-v4-state.md).

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Chaque dialogue recueillera des informations, et nous allons enregistrer ces informations dans l’état utilisateur. Nous allons définir une classe pour chaque dialogue, puis utiliser ces classes en tant que propriétés dans notre classe d’état utilisateur.

```csharp
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

/// <summary>
/// User state information.
/// </summary>
public class UserInfo
{
    public GuestInfo Guest { get; set; }
    public TableInfo Table { get; set; }
    public WakeUpInfo WakeUp { get; set; }
}
```

Lors d’un tour de bot, la méthode `CreateContext` du jeu de dialogues établit l’état du dialogue.
La méthode utilise le [contexte de tour](bot-builder-concept-activity-processing.md#turn-context) et un objet d’état comme paramètres.

Pour les dialogues, cet objet d’état doit implémenter l’interface `IDictionary<string, object>`. Dans la mesure où ce bot utilise uniquement un état de conversation pour héberger l’état du dialogue, notre classe d’état de conversation peut représenter un simple dictionnaire.

```csharp
using System.Collections.Generic;

/// <summary>
/// Conversation state information.
/// We are also using this directly for dialog state, which needs an <see cref="IDictionary{string, object}"/> object.
/// </summary>
public class ConversationInfo : Dictionary<string, object> { }
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Pour effectuer le suivi de l’entrée utilisateur, nous transmettrons à partir du dialogue parent un objet `userInfo` sous la forme d’un paramètre de dialogue. Dans chaque classe de dialogue, le dialogue sera créé dans le constructeur, pour vous permettre d’enregistrer les informations dans `userInfo`. Tout au long de ce dialogue, nous pouvons inscrire des données dans un objet d’état local défini en tant que propriété sur l’objet `step.values`, à mesure que l’utilisateur saisit des informations. Une fois le dialogue terminé, l’objet d’état local est supprimé. Par conséquent, nous allons enregistrer l’objet d’état local vers l’élément `userInfo` local, qui conservera les informations sur l’utilisateur pour toutes les conversations que vous avez avec lui. 

---

## <a name="define-a-modular-check-in-dialog"></a>Définir un dialogue d’enregistrement modulaire

Tout d’abord, commençons avec un simple dialogue d’enregistrement, qui demande à l’utilisateur son nom et la chambre qu’il souhaite occuper. Pour modulariser cette tâche, nous créons une classe `CheckIn` qui étend `ComponentDialog`. Cette classe comporte un constructeur qui définit le nom du dialogue racine, soit une *en cascade* à trois étapes. La signature et la construction de l’objet du dialogue sont exactement identiques à celles d’une cascade standard.

**Étapes du dialogue d’enregistrement**

1. Demander le nom du client.
1. Demander la chambre qu’il souhaite occuper.
1. Envoyer un message de confirmation et finaliser le dialogue.

Pour plus d’informations sur les dialogues et les cascades, consultez l’article [Gérer un flux de conversation simple avec des dialogues](bot-builder-dialog-manage-conversation-flow.md).

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

La classe `CheckIn` contient un constructeur privé qui définit les étapes de notre dialogue d’enregistrement, crée une instance unique, puis l’expose dans une propriété `Instance` statique.

Tout au long de ce dialogue, nous pouvons inscrire des données dans un objet d’état local, accessible via une propriété du contexte d’étape, `step.values`. Une fois le dialogue terminé, l’objet d’état local est supprimé. Par conséquent, nous enregistrons l’objet d’état local vers l’élément `userState` du bot, qui conservera les informations sur l’utilisateur pour toutes les conversations que vous avez avec lui.

Pour plus d’informations sur la gestion des données d’état, consultez la rubrique [Enregistrer l’état à l’aide des propriétés de conversation et d’utilisateur](bot-builder-howto-v4-state.md). 

**CheckIn.cs**
```csharp
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Recognizers.Text;

namespace HotelBot
{
    public class CheckIn : DialogContainer
    {
        public const string Id = "checkIn";

        private const string GuestKey = nameof(CheckIn);

        public static CheckIn Instance { get; } = new CheckIn();

        // You can start this from the parent using the dialog's ID.
        private CheckIn() : base(Id)
        {
            // Define the conversation flow using a waterfall model.
            this.Dialogs.Add(Id, new WaterfallStep[]
            {
                async (dc, args, next) =>
                {
                    // Clear the guest information and prompt for the guest's name.
                    dc.ActiveDialog.State[GuestKey] = new GuestInfo();
                    await dc.Prompt("textPrompt", "What is your name?");
                },
                async (dc, args, next) =>
                {
                    // Save the name and prompt for the room number.
                    var name = args["Value"] as string;

                    var guestInfo = dc.ActiveDialog.State[GuestKey];
                    ((GuestInfo)guestInfo).Name = name;

                    await dc.Prompt("numberPrompt",
                        $"Hi {name}. What room will you be staying in?");
                },
                async (dc, args, next) =>
                {
                    // Save the room number and "sign off".
                    var room = (string)args["Text"];

                    var guestInfo = dc.ActiveDialog.State[GuestKey];
                    ((GuestInfo)guestInfo).Room = room;

                    await dc.Context.SendActivity("Great, enjoy your stay!");

                    // Save dialog state to user state and end the dialog.
                    var userState = UserState<UserInfo>.Get(dc.Context);
                    userState.Guest = (GuestInfo)guestInfo;

                    await dc.End();
                }
            });

            // Define the prompts used in this conversation flow.
            this.Dialogs.Add("textPrompt", new TextPrompt());
            this.Dialogs.Add("numberPrompt", new NumberPrompt<int>(Culture.English));
        }
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**checkIn.js**
```JavaScript
const { ComponentDialog, DialogSet, TextPrompt, NumberPrompt, WaterfallDialog } = require('botbuilder-dialogs');

class CheckIn extends ComponentDialog {
    constructor(dialogId, userInfo) {
        // Dialog ID of 'checkIn' will start when class is called in the parent
        super(dialogId);

        // Defining the conversation flow using a waterfall model
        this.dialogs.add(new WaterfallDialog('checkIn', [
            async function (step) {
                // Create a new local guestInfo step.values object
                step.values.guestInfo = {};
                return await step.prompt('textPrompt', "What is your name?");
            },
            async function (step) {
                // Save the name 
                step.values.guestInfo.userName = step.result;
                return await step.prompt('numberPrompt', `Hi ${name}. What room will you be staying in?`);
            },
            async function (step) { 
                // Save the room number
                step.values.guestInfo.room = step.result;
                await step.context.sendActivity(`Great! Enjoy your stay!`);

                // Save dialog's state object to the parent's state object
                const user = await userInfo.get(step.context);
                user.guestInfo = step.values.guestInfo;
                // Save changes
                await userInfo.set(step.context, user);
                return await step.endDialog();
            }
        ]));
        
        // Defining the prompt used in this conversation flow
        this.dialogs.add(new TextPrompt('textPrompt'));
        this.dialogs.add(new NumberPrompt('numberPrompt'));
    }
}
exports.CheckIn = CheckIn;
```

---

## <a name="define-modular-reserve-table-and-wake-up-dialogs"></a>Définir des dialogues modulaires pour une réservation de table et un service de réveil

L’un des avantages de l’utilisation d’un dialogue composant est la capacité à composer plusieurs dialogues à la fois. Comme chaque `DialogSet` gère un ensemble exclusif de `dialogs`, cela complique le partage ou la référence croisée de `dialogs`. C’est là qu’intervient le dialogue composant. Vous pouvez utiliser les dialogues composants pour créer un dialogue composite qui facilite la gestion du flux entre les dialogues. Pour illustrer cela, nous allons créer deux autres dialogues : un pour demander à l’utilisateur quelle table il souhaite réserver pour le dîner, et un autre pour demander un service de réveil. Là encore, nous allons utiliser une classe distincte pour chaque dialogue, et chaque dialogue étendra `ComponentDialog`.

**Étapes du dialogue de réservation d’une table**

1. Demander la réservation d’une table.
1. Envoyer un message de confirmation et finaliser le dialogue.

**Étapes du dialogue du service de réveil**

1. Demander l’heure pour le service de réveil.
1. Envoyer un message de confirmation et finaliser le dialogue.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**ReserveTable.cs**
```csharp
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Recognizers.Text;
using System.Linq;
using Choice = Microsoft.Bot.Builder.Prompts.Choices.Choice;
using FoundChoice = Microsoft.Bot.Builder.Prompts.Choices.FoundChoice;

namespace HotelBot
{
    public class ReserveTable : DialogContainer
    {
        public const string Id = "reserveTable";

        private const string TableKey = "Table";

        public static ReserveTable Instance { get; } = new ReserveTable();

        private ReserveTable() : base(Id)
        {
            this.Dialogs.Add(Id, new WaterfallStep[]
            {
                async (dc, args, next) =>
                {
                    // Clear the table information and prompt for the table number.
                    dc.ActiveDialog.State[TableKey] = new TableInfo();

                    var guestInfo = UserState<UserInfo>.Get(dc.Context).Guest;

                    var prompt = $"Welcome {guestInfo.Name}, " +
                        $"which table would you like to reserve?";
                    var choices = new string[] { "1", "2", "3", "4", "5", "6" };
                    await dc.Prompt("choicePrompt", prompt,
                        new ChoicePromptOptions
                        {
                            Choices = choices.Select(s => new Choice { Value = s }).ToList()
                        });
                },
                async (dc, args, next) =>
                {
                    // Save the table number and "sign off".
                    var table = (args["Value"] as FoundChoice).Value;

                    var tableInfo = dc.ActiveDialog.State[TableKey];
                    ((TableInfo)tableInfo).Number = table;

                    await dc.Context.SendActivity(
                        $"Sounds great; we will reserve table number {table} for you.");

                    // Save dialog state to user state and end the dialog.
                    var userState = UserState<UserInfo>.Get(dc.Context);
                    userState.Table = (TableInfo)tableInfo;

                    await dc.End();
                }
            });

            // Define the prompts used in this conversation flow.
            this.Dialogs.Add("choicePrompt", new ChoicePrompt(Culture.English));
        }
    }
}
```

**WakeUp.cs**
```csharp
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Recognizers.Text;
using System.Collections.Generic;
using System.Linq;
using DateTimeResolution = Microsoft.Bot.Builder.Prompts.DateTimeResult.DateTimeResolution;

namespace HotelBot
{
    public class WakeUp : DialogContainer
    {
        public const string Id = "wakeUp";

        private const string WakeUpKey = "WakeUp";

        public static WakeUp Instance { get; } = new WakeUp();

        private WakeUp() : base(Id)
        {
            this.Dialogs.Add(Id, new WaterfallStep[]
            {
                async (dc, args, next) =>
                {
                    // Clear the wake up call information and prompt for the alarm time.
                   dc.ActiveDialog.State[WakeUpKey] = new WakeUpInfo();

                    var guestInfo = UserState<UserInfo>.Get(dc.Context).Guest;

                    await dc.Prompt("datePrompt", $"Hi {guestInfo.Name}, " +
                        $"what time would you like your alarm set for?");
                },
                async (dc, args, next) =>
                {
                    // Save the alarm time and "sign off".
                    var resolution = (List<DateTimeResolution>)args["Resolution"];

                    var wakeUp = dc.ActiveDialog.State[WakeUpKey];
                    ((WakeUpInfo)wakeUp).Time = resolution?.FirstOrDefault()?.Value;

                    var userState = UserState<UserInfo>.Get(dc.Context);
                    await dc.Context.SendActivity(
                        $"Your alarm is set to {((WakeUpInfo)wakeUp).Time}" +
                        $" for room {userState.Guest.Room}.");

                    // Save dialog state to user state and end the dialog.
                    userState.WakeUp = (WakeUpInfo)wakeUp;

                    await dc.End();
                }
            });

            // Define the prompts used in this conversation flow.
            this.Dialogs.Add("datePrompt", new DateTimePrompt(Culture.English));
        }
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**reserveTable.js**
```JavaScript
const { ComponentDialog, DialogSet, ChoicePrompt, WaterfallDialog } = require('botbuilder-dialogs');

class ReserveTable extends ComponentDialog {
    constructor(dialogId, userInfo) {
        super(dialogId); 

        // Defining the conversation flow using a waterfall model
        this.dialogs.add(new WaterfallDialog('reserve_table', [
            async function (step) {
                // Get the user state from context
                const user = await userInfo.get(step.context);

                // Create a new local reserveTable state object
                step.values.reserveTable = {};

                const prompt = `Welcome ${ user.guestInfo.userName }, which table would you like to reserve?`;
                const choices = ['1', '2', '3', '4', '5', '6'];
                return await step.prompt('choicePrompt', prompt, choices);
            },
            async function(step) {
                const choice = step.result;
                
                // Save the table number
                step.values.reserveTable.tableNumber = choice.value;
                await step.context.sendActivity(`Sounds great, we will reserve table number ${ choice.value } for you.`);
                
                // Get the user state from context
                const user = await userInfo.get(dc.context);
                // Persist dialog's state object to the parent's state object
                user.reserveTable = step.values.reserveTable;
               
                // Save changes
                await userInfo.set(step.context, user);

                // End the dialog
                return await step.endDialog();
            }
        ]));

        // Defining the prompt used in this conversation flow
        this.dialogs.add(new ChoicePrompt('choicePrompt'));
    }
}
exports.ReserveTable = ReserveTable;
```

**wakeUp.js**
```JavaScript
const { ComponentDialog, DialogSet, DatetimePrompt, WaterfallDialog } = require('botbuilder-dialogs');

class WakeUp extends ComponentDialog {
    constructor(userInfo) {
        // Dialog ID of 'wakeup' will start when class is called in the parent
        super('wakeUp');

        this.dialogs.add(new WaterfallDialog('wakeUp', [
            async function (step) {
                // Get the user state from context
                const user = await userInfo.get(step.context); 

                // Create a new local reserveTable state object
                step.values.wakeUp = {};  
                             
                return await step.prompt('datePrompt', `Hello, ${ user.guestInfo.userName }. What time would you like your alarm to be set?`);
            },
            async function (step) {
                const time = step.result;
            
                // Get the user state from context
                const user = await userInfo.get(step.context);

                // Save the time
                step.values.wakeUp.time = time[0].value

                await step.context.sendActivity(`Your alarm is set to ${ time[0].value } for room ${ user.guestInfo.room }`);
                
                // Save dialog's state object to the parent's state object
                user.wakeUp = step.values.wakeUp;

                // Save changes
                await userInfo.step(step.context, user);

                // End the dialog
                return await step.endDialog();
            }]));

        // Defining the prompt used in this conversation flow
        this.dialogs.add(new DatetimePrompt('datePrompt'));
    }
}
exports.WakeUp = WakeUp;
```

---

## <a name="add-modular-dialogs-to-a-bot"></a>Ajouter des dialogues modulaires à un robot

Le fichier principal du bot associe ces trois dialogues modularisés dans un même bot.

- Ces dialogues sont ajoutés au jeu de dialogues principal de notre bot.
- Les dialogues de réservation d’une table et de service de réveil sont intégrés au flux de la conversation du dialogue principal.
- Lorsqu’une nouvelle conversation démarre, aucun dialogue n’est actif et la logique de tour du bot prend le relais.

**Gestionnaire de tour du bot**

Chaque fois que le tour du bot survient en dehors d’un dialogue actif, il vérifie l’état utilisateur.
1. S’il dispose déjà des informations concernant le client, il démarre son dialogue principal.
1. Sinon, il démarre le dialogue d’enregistrement enfant du dialogue principal.

**Étapes du dialogue principal**

1. Demander au client ce qu’il souhaite faire : réserver une table ou demander un réveil par téléphone.
1. Démarrer le dialogue enfant correspondant, ou envoyer un message _input not understood_ (entrée incompréhensible), puis passer à l’étape suivante.
1. Retourner au début de ce dialogue.


# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Le flux du dialogue est mis à jour à l’aide de la méthode `Continue` du contexte du dialogue. Cette méthode exécute la prochaine étape de la cascade sur la pile de dialogues.

**HotelBot.cs**
```csharp
using System.Collections.Generic;
using System.Threading.Tasks;
using Microsoft.Bot;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Bot.Schema;

namespace HotelBot
{
    public class HotelBot : IBot
    {
        private const string MainMenuId = "mainMenu";

        private DialogSet _dialogs { get; } = ComposeMainDialog();

        /// <summary>
        /// Every Conversation turn for our bot calls this method. 
        /// </summary>
        /// <param name="context">The current turn context.</param>        
        public async Task OnTurn(ITurnContext context)
        {
            if (context.Activity.Type is ActivityTypes.Message)
            {
                // Get the user and conversation state from the turn context.
                var userInfo = UserState<UserInfo>.Get(context);
                var conversationInfo = ConversationState<ConversationInfo>.Get(context);

                // Establish dialog state from the conversation state.
                var dc = _dialogs.CreateContext(context, conversationInfo);

                // Continue any current dialog.
                await dc.Continue();

                // Every turn sends a response, so if no response was sent,
                // then there no dialog is currently active.
                if (!context.Responded)
                {
                    // If we don't yet have the guest's info, start the check-in dialog.
                    if (string.IsNullOrEmpty(userInfo?.Guest?.Name))
                    {
                        await dc.Begin(CheckIn.Id);
                    }
                    // Otherwise, start our bot's main dialog.
                    else
                    {
                        await dc.Begin(MainMenuId);
                    }
                }
            }
        }

        /// <summary>
        /// Composes a main dialog for our bot.
        /// </summary>
        /// <returns>A new main dialog.</returns>
        private static DialogSet ComposeMainDialog()
        {
            var dialogs = new DialogSet();

            dialogs.Add(MainMenuId, new WaterfallStep[]
            {
                async (dc, args, next) =>
                {
                    var menu = new List<string> { "Reserve Table", "Wake Up" };
                    await dc.Context.SendActivity(MessageFactory.SuggestedActions(menu, "How can I help you?"));
                },
                async (dc, args, next) =>
                {
                    // Decide which dialog to start.
                    var result = (args["Activity"] as Activity)?.Text?.Trim().ToLowerInvariant();
                    switch (result)
                    {
                        case "reserve table":
                            await dc.Begin(ReserveTable.Id);
                            break;
                        case "wake up":
                            await dc.Begin(WakeUp.Id);
                            break;
                        default:
                            await dc.Context.SendActivity("Sorry, I don't understand that command. Please choose an option from the list below.");
                            await next();
                            break;
                    }
                },
                async (dc, args, next) =>
                {
                    // Show the main menu again.
                    await dc.Replace(MainMenuId);
                }
            });

            // Add our child dialogs.
            dialogs.Add(CheckIn.Id, CheckIn.Instance);
            dialogs.Add(ReserveTable.Id, ReserveTable.Instance);
            dialogs.Add(WakeUp.Id, WakeUp.Instance);

            return dialogs;
        }
    }
}
```

Enfin, nous devons mettre à jour la méthode `ConfigureServices` de la classe `StartUp` pour connecter notre bot et ajouter l’intergiciel (middleware) d’état.

**Startup.cs**
```csharp
// This method gets called by the runtime. Use this method to add services to the container.
public void ConfigureServices(IServiceCollection services)
{
    services.AddBot<HotelBot>(options =>
    {
        options.CredentialProvider = new ConfigurationCredentialProvider(Configuration);

        options.Middleware.Add(new CatchExceptionMiddleware<Exception>(async (context, exception) =>
        {
            await context.TraceActivity($"{nameof(HotelBot)} Exception", exception);
            await context.SendActivity("Sorry, it looks like something went wrong!");
        }));

        // Add state management middleware for conversation and user state.
        var path = Path.Combine(Path.GetTempPath(), nameof(HotelBot));
        if (!Directory.Exists(path))
        {
            Directory.CreateDirectory(path);
        }
        IStorage storage = new FileStorage(path);

        options.Middleware.Add(new ConversationState<ConversationInfo>(storage));
        options.Middleware.Add(new UserState<UserInfo>(storage));
    });
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Le flux du dialogue est mis à jour à l’aide de la méthode `continue` du contexte du dialogue. Cette méthode exécute la prochaine étape de la cascade sur la pile de dialogues.

**app.js**
```JavaScript
const { BotFrameworkAdapter, ConversationState, UserState, MemoryStorage, MessageFactory } = require("botbuilder");
const { DialogSet } = require("botbuilder-dialogs");
const restify = require("restify");
var azure = require('botbuilder-azure'); 

// Create server
let server = restify.createServer();
server.listen(process.env.port || process.env.PORT || 3978, function () {
    console.log(`${server.name} listening to ${server.url}`);
});

// Create adapter
const adapter = new BotFrameworkAdapter({
    appId: process.env.MICROSOFT_APP_ID,
    appPassword: process.env.MICROSOFT_APP_PASSWORD
});

// Memory Storage
const storage = new MemoryStorage();
// ConversationState lasts for the entirety of a conversation then gets disposed of
const conversationState = new ConversationState(storage);

// UserState persists information about the user across all of the conversations you have with that user
const userState  = new UserState(storage);

// Create a place in the conversation state to store dialog state.
const dialogState = conversationState.createProperty('dialogState');

// Create a place in the user storage to store a user info.
const userInfo = userState.createProperty('userInfo');

// Create a dialog set and pass in our dialogState property.
const dialogs = new DialogSet(dialogState);

// Listen for incoming requests 
server.post('/api/messages', (req, res) => {
    adapter.processActivity(req, res, async (context) => {
        const isMessage = context.activity.type === 'message';
        const dc = dialogs.createContext(context);
 
        // Continue the current dialog if one is currently active
        await dc.continueDialog();

        // Default action
        if (!context.responded && isMessage) {

            // Getting the user info from the state
            const user = await userInfo.get(dc.context); 
            // If guest info is undefined prompt the user to check in
            if (!user.guestInfo) {
                await dc.beginDialog('checkInPrompt');
            } else {
                await dc.beginDialog('mainMenu'); 
            }           
        }
        
        // End by saving any changes to the state that have occured during this turn.
        await conversationState.saveChanges(dc.context);
        await userState.saveChanges(dc.context);
    });
});

dialogs.add(new WaterfallDialog('mainMenu', [
    async function (step) {
        const menu = ["Reserve Table", "Wake Up"];
        await step.context.sendActivity(MessageFactory.suggestedActions(menu));
        return await step.next();
    },
    async function (step) {
        // Decide which module to start
        switch (step.result) {
            case "Reserve Table":
                return await step.beginDialog('reservePrompt');
                break;
            case "Wake Up":
                return await step.beginDialog('wakeUpPrompt');
                break;
            default:
                await step.context.sendActivity("Sorry, i don't understand that command. Please choose an option from the list below.");
                return await step.next();
                break;            
        }
    },
    async function (step){
        return await step.replaceDialog('mainMenu'); // Show the menu again
    }

]));

// Importing the dialogs 
const checkIn = require("./checkIn");
dialogs.add(new checkIn.CheckIn('checkInPrompt', userState));

const reserve_table = require("./reserveTable");
dialogs.add(new reserve_table.ReserveTable('reservePrompt', userState));

const wake_up = require("./wake_up");
dialogs.add(new wake_up.WakeUp('wakeUpPrompt', userState));
```

---
<!-- TODO: These dialogs are not fully modularized, as there are cross dependencies:
    - Importantly, the dialogs need to know details about the bot's user state.
-->

Comme vous pouvez le constater, les dialogues modulaires sont ajoutés au dialogue principal du bot de la même façon que des [invites](bot-builder-prompts.md) à un dialogue. Vous pouvez ajouter autant de dialogues enfants à votre dialogue principal que vous le souhaitez. Chaque module ajoute des fonctionnalités et des services supplémentaires que le bot peut offrir à vos utilisateurs.

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous savez comment modulariser votre logique de bot à l’aide de dialogues, voyons comment utiliser la reconnaissance vocale (LUIS) pour aider votre bot à choisir le moment où il doit commencer les dialogues.

> [!div class="nextstepaction"]
> [Utiliser LUIS pour la compréhension de la langue](./bot-builder-howto-v4-luis.md)
