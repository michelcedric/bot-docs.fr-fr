---
title: Collecter les entrées utilisateur avec une invite de dialogue | Microsoft Docs
description: Découvrez comment demander aux utilisateurs de saisir une entrée à l’aide de la bibliothèque Dialogs du kit SDK Bot Framework.
keywords: invites, invite, entrée utilisateur, dialogues, AttachmentPrompt, ChoicePrompt, ConfirmPrompt, DatetimePrompt, NumberPrompt, TextPrompt, réinviter, validation
author: JonathanFingold
ms.author: v-jofing
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 02/19/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 68c01b0f12790393fe0ee7ae0bd28addf2d26ae7
ms.sourcegitcommit: 05ddade244874b7d6e2fc91745131b99cc58b0d6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 02/21/2019
ms.locfileid: "56591120"
---
# <a name="gather-user-input-using-a-dialog-prompt"></a>Collecter les entrées utilisateur avec une invite de dialogue

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Pour interagir avec les utilisateurs, un bot collecte généralement des informations en posant des questions. La bibliothèque de *dialogues* vous permet de poser des questions facilement et de valider la réponse pour vous assurer qu’elle correspond à un type de données spécifique ou répond aux règles de validation personnalisées. Cette rubrique décrit en détail comment créer et appeler des invites à partir d’un dialogue en cascade.

## <a name="prerequisites"></a>Prérequis

- Le code de cet article est basé sur l’exemple **DialogPromptBot**. Vous aurez besoin d’une copie de l’[exemple C# ](https://aka.ms/dialog-prompt-cs) ou de l’[exemple JS](https://aka.ms/dialog-prompt-js).
- Une connaissance élémentaire de la [bibliothèque de dialogues](bot-builder-concept-dialog.md) et de la façon de [gérer les conversations](bot-builder-dialog-manage-conversation-flow.md) est nécessaire.
- [Bot Framework Emulator](https://github.com/Microsoft/BotFramework-Emulator) à des fins de test.

## <a name="using-prompts"></a>Utilisation des invites

Un dialogue peut utiliser une invite de commandes uniquement si le dialogue et l’invite de commandes se situent dans le même ensemble de dialogues. Vous pouvez utiliser la même invite à plusieurs étapes d’un dialogue et dans plusieurs dialogues au sein du même ensemble de dialogues. Toutefois, vous associez une validation personnalisée à une invite de commandes au moment de l’initialisation. Pour utiliser une validation différente pour le même type d’invite, vous avez besoin de plusieurs instances du type d’invite, chacun avec son propre code de validation.

### <a name="define-a-state-property-accessor-for-the-dialog-state"></a>Définissez un accesseur de propriété d’état pour l’état du dialogue

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

L’exemple d’invite de dialogue utilisé dans cet article demande des informations de réservation à l’utilisateur. Pour gérer la date et la taille d’un tiers, nous définissons une classe interne pour les informations de réservation dans le fichier DialogPromptBot.cs.

```csharp
public class Reservation
{
    public int Size { get; set; }

    public string Location { get; set; }

    public string Date { get; set; }
}
```

Ensuite, nous ajoutons un accesseur de propriété d’état pour les données de réservation.

```csharp
public class DialogPromptBotAccessors
{
    public DialogPromptBotAccessors(ConversationState conversationState)
    {
        ConversationState = conversationState
            ?? throw new ArgumentNullException(nameof(conversationState));
    }

    public static string DialogStateAccessorKey { get; }
        = "DialogPromptBotAccessors.DialogState";
    public static string ReservationAccessorKey { get; }
        = "DialogPromptBotAccessors.Reservation";

    public IStatePropertyAccessor<DialogState> DialogStateAccessor { get; set; }
    public IStatePropertyAccessor<DialogPromptBot.Reservation> ReservationAccessor { get; set; }

    public ConversationState ConversationState { get; }
}
```

Dans Startup.cs, nous mettons à jour la méthode `ConfigureServices` pour définir les accesseurs.

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // ...

    IStorage dataStore = new MemoryStorage();
    var conversationState = new ConversationState(dataStore);

    // Create and register state accesssors.
    services.AddSingleton<DialogPromptBotAccessors>(sp =>
    {
        // State accessors enable other components to read and write individual properties of state.
        var accessors = new DialogPromptBotAccessors(conversationState)
        {
            DialogStateAccessor =
                conversationState.CreateProperty<DialogState>(
                    DialogPromptBotAccessors.DialogStateAccessorKey),
            ReservationAccessor =
                conversationState.CreateProperty<DialogPromptBot.Reservation>(
                    DialogPromptBotAccessors.ReservationAccessorKey),
        };

        return accessors;
    });

    // ...
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Aucune modification du code du service HTTP n’est exigée pour JavaScript, nous pouvons laisser notre fichier index.js en l’état.

Dans bot.js, nous incluons les instructions `require` nécessaires pour le bot d’invite de dialogue.

```javascript
const { ActivityTypes } = require('botbuilder');
const { DialogSet, WaterfallDialog, NumberPrompt, DateTimePrompt, ChoicePrompt, DialogTurnStatus }
    = require('botbuilder-dialogs');
```

Ajoutez des identificateurs pour les accesseurs de propriété d’état, les dialogues et les invites.

```javascript
// Define identifiers for our state property accessors.
const DIALOG_STATE_ACCESSOR = 'dialogStateAccessor';
const RESERVATION_ACCESSOR = 'reservationAccessor';

// Define identifiers for our dialogs and prompts.
const RESERVATION_DIALOG = 'reservationDialog';
const SIZE_RANGE_PROMPT = 'rangePrompt';
const LOCATION_PROMPT = 'locationPrompt';
const RESERVATION_DATE_PROMPT = 'reservationDatePrompt';
```

---

### <a name="create-a-dialog-set-and-prompts"></a>Créer un jeu de dialogues et des invites

En règle générale, vous créez et ajoutez des invites et des dialogues dans votre ensemble de dialogues lorsque vous initialisez votre bot. L’ensemble de dialogues peut ensuite résoudre les ID de l’invite lorsque le bot reçoit une entrée de la part de l’utilisateur.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Dans la classe `DialogPromptBot`, définissez des identificateurs pour les dialogues, les invites et les valeurs à suivre dans le dialogue.

```csharp
// Define identifiers for our dialogs and prompts.
private const string ReservationDialog = "reservationDialog";
private const string PartySizePrompt = "partySizePrompt";
private const string SizeRangePrompt = "sizeRangePrompt";
private const string LocationPrompt = "locationPrompt";
private const string ReservationDatePrompt = "reservationDatePrompt";

// Define keys for tracked values within the dialog.
private const string LocationKey = "location";
private const string PartySizeKey = "partySize";
```

Dans le constructeur du bot, créez l’ensemble de dialogues, ajoutez les invites et ajoutez le dialogue de réservation. Nous incluons la validation personnalisée lorsque nous créons les invites, et nous appliquons les fonctions de validation plus tard.

```csharp
private readonly DialogSet _dialogSet;
private readonly DialogPromptBotAccessors _accessors;

// ...

// Initializes a new instance of the <see cref="DialogPromptBot"/> class.
public DialogPromptBot(DialogPromptBotAccessors accessors, ILoggerFactory loggerFactory)
{
    // ...

    _accessors = accessors ?? throw new System.ArgumentNullException(nameof(accessors));

    // Create the dialog set and add the prompts, including custom validation.
    _dialogSet = new DialogSet(_accessors.DialogStateAccessor);

    _dialogSet.Add(new NumberPrompt<int>(PartySizePrompt, PartySizeValidatorAsync));
    _dialogSet.Add(new NumberPrompt<int>(SizeRangePrompt, RangeValidatorAsync));
    _dialogSet.Add(new ChoicePrompt(LocationPrompt));
    _dialogSet.Add(new DateTimePrompt(ReservationDatePrompt, DateValidatorAsync));

    // Define the steps of the waterfall dialog and add it to the set.
    WaterfallStep[] steps = new WaterfallStep[]
    {
        PromptForPartySizeAsync,
        PromptForLocationAsync,
        PromptForReservationDateAsync,
        AcknowledgeReservationAsync,
    };

    _dialogSet.Add(new WaterfallDialog(ReservationDialog, steps));
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Dans le constructeur, créez des accesseurs de propriété d’état.
Ensuite, créez l’ensemble de dialogues et ajoutez les invites, notamment la validation personnalisée.
Définissez ensuite les étapes du dialogue en cascade, puis ajoutez-les à l’ensemble.

```javascript
constructor(conversationState) {
    // Creates our state accessor properties.
    // See https://aka.ms/about-bot-state-accessors to learn more about the bot state and state accessors.
    this.dialogStateAccessor = conversationState.createProperty(DIALOG_STATE_ACCESSOR);
    this.reservationAccessor = conversationState.createProperty(RESERVATION_ACCESSOR);
    this.conversationState = conversationState;

    // Create the dialog set and add the prompts, including custom validation.
    this.dialogSet = new DialogSet(this.dialogStateAccessor);
    this.dialogSet.add(new NumberPrompt(SIZE_RANGE_PROMPT, this.rangeValidator));
    this.dialogSet.add(new ChoicePrompt(LOCATION_PROMPT));
    this.dialogSet.add(new DateTimePrompt(RESERVATION_DATE_PROMPT, this.dateValidator));

    // Define the steps of the waterfall dialog and add it to the set.
    this.dialogSet.add(new WaterfallDialog(RESERVATION_DIALOG, [
        this.promptForPartySize.bind(this),
        this.promptForLocation.bind(this),
        this.promptForReservationDate.bind(this),
        this.acknowledgeReservation.bind(this),
    ]));
}
```

---

### <a name="implement-dialog-steps"></a>Implémenter des étapes de dialogue

Dans le fichier principal du bot, nous implémentons chacune de ses étapes du dialogue en cascade. Une fois qu’une invite est ajoutée, nous l’appelons dans une seule étape d’un dialogue en cascade et obtenons le résultat de l’invite à l’étape suivante du dialogue. Pour appeler une invite à partir d’une étape en cascade, appelez la méthode _invite_ du _contexte de l’étape en cascade_ de l’objet. Le premier paramètre est l’ID de l’invite à utiliser, et le deuxième paramètre contient les options pour l’invite, comme le texte utilisé pour demander une entrée à l’utilisateur.

Ces méthodes montrent :

- Comment appeler une invite depuis une étape en cascade, notamment comment passer des _options d’invite_.
- Comment fournir des paramètres supplémentaires à un validateur personnalisé avec la propriété _validations_.
- Comment fournir les choix d’une invite de choix avec la propriété _choices_.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Dans le fichier **DialogPromptBot.cs**, nous implémentons les étapes du dialogue en cascade.

Vous pouvez voir ici les deux premières étapes de la cascade, `PromptForPartySizeAsync` et `PromptForLocationAsync`.

```csharp
// First step of the main dialog: prompt for party size.
private async Task<DialogTurnResult> PromptForPartySizeAsync(
    WaterfallStepContext stepContext,
    CancellationToken cancellationToken = default(CancellationToken))
{
    // Prompt for the party size. The result of the prompt is returned to the next step of the waterfall.
    return await stepContext.PromptAsync(
        SizeRangePrompt,
        new PromptOptions
        {
            Prompt = MessageFactory.Text("How many people is the reservation for?"),
            RetryPrompt = MessageFactory.Text("How large is your party?"),
            Validations = new Range { Min = 3, Max = 8 },
        },
        cancellationToken);
}

// Second step of the main dialog: prompt for location.
private async Task<DialogTurnResult> PromptForLocationAsync(
    WaterfallStepContext stepContext,
    CancellationToken cancellationToken)
{
    // Record the party size information in the current dialog state.
    var size = (int)stepContext.Result;
    stepContext.Values[PartySizeKey] = size;

    // Prompt for the location.
    return await stepContext.PromptAsync(
        LocationPrompt,
        new PromptOptions
        {
            Prompt = MessageFactory.Text("Please choose a location."),
            RetryPrompt = MessageFactory.Text("Sorry, please choose a location from the list."),
            Choices = ChoiceFactory.ToChoices(new List<string> { "Redmond", "Bellevue", "Seattle" }),
        },
        cancellationToken);
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Dans le fichier **bot.js**, nous implémentons les étapes du dialogue en cascade.

Vous pouvez voir ici les deux premières étapes de la cascade, `promptForPartySize` et `promptForLocation`.

```javascript
async promptForPartySize(stepContext) {
    // Prompt for the party size. The result of the prompt is returned to the next step of the waterfall.
    return await stepContext.prompt(
        SIZE_RANGE_PROMPT, {
            prompt: 'How many people is the reservation for?',
            retryPrompt: 'How large is your party?',
            validations: { min: 3, max: 8 },
        });
}

async promptForLocation(stepContext) {
    // Record the party size information in the current dialog state.
    stepContext.values.size = stepContext.result;

    // Prompt for location.
    return await stepContext.prompt(LOCATION_PROMPT, {
        prompt: 'Please choose a location.',
        retryPrompt: 'Sorry, please choose a location from the list.',
        choices: ['Redmond', 'Bellevue', 'Seattle'],
    });
}
```

---

Le deuxième paramètre de la méthode _prompt_ prend un objet _prompt options_, qui a les propriétés suivantes.

| Propriété | Description |
| :--- | :--- |
| _prompt_ | L’activité initiale à envoyer à l’utilisateur, pour demander son entrée. |
| _retry prompt_ | L’activité à envoyer à l’utilisateur si sa première entrée n’a pas été validée. |
| _choix_ | Une liste des choix que l’utilisateur peut sélectionner, pour une utilisation avec une invite de choix. |
| _validations_ | Paramètres supplémentaires à utiliser avec un validateur personnalisé. |

En règle générale, les propriétés prompt et retry prompt sont des activités, bien qu’il existe des variantes sur la façon dont cela est géré par les différents langages de programmation.

Vous devez toujours spécifier l’activité prompt initiale à envoyer à l’utilisateur.

Spécifier une nouvelle invite est utile lorsque l’entrée de l’utilisateur est incapable de valider, soit parce que son format ne peut pas être analysé par l’invite, par exemple « demain » en réponse à une invite numérique, ou parce que l’entrée ne remplit pas un critère de validation. Dans ce cas, si aucune nouvelle invite n’a été fournie, l’invite utilisera l’activité prompt initiale pour redemander une saisie à l’utilisateur.

Pour une invite de choix, vous devez toujours fournir la liste des choix disponibles.

## <a name="custom-validation"></a>Validation personnalisée

Vous pouvez valider la réponse à une invite avant de retourner la valeur à l’étape suivante de la **cascade**. Une fonction du validateur possède un paramètre de _contexte de validateur d’invite_ et retourne une valeur booléenne indiquant si l’entrée passe la validation.

Le contexte de validateur d’invite inclut les propriétés suivantes :

| Propriété | Description |
| :--- | :--- |
| _Contexte_ | Le contexte de tour actuel pour le bot. |
| _Reconnu_ | Un _résultat du module de reconnaissance de l’invite_ qui contient des informations sur l’entrée de l’utilisateur, comme traitées par le module de reconnaissance. |
| _Options_ | Contient les _options de l’invite_ qui ont été fournies dans l’appel pour démarrer l’invite. |

Le résultat du module de reconnaissance de l’invite a les propriétés suivantes :

| Propriété | Description |
| :--- | :--- |
| _Réussi_ | Indique si le module de reconnaissance a été en mesure d’analyser l’entrée. |
| _Valeur_ | La valeur renvoyée du module de reconnaissance. Si nécessaire, le code de validation peut modifier cette valeur. |

### <a name="implement-validation-code"></a>Implémenter le code de validation

Vous associez une validation personnalisée à une invite de commandes au moment de l’initialisation dans le constructeur du bot.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// ...
_dialogSet.Add(new NumberPrompt<int>(SizeRangePrompt, RangeValidatorAsync));
// ...
_dialogSet.Add(new DateTimePrompt(ReservationDatePrompt, DateValidatorAsync));
// ...
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// ...
this.dialogSet.add(new NumberPrompt(SIZE_RANGE_PROMPT, this.rangeValidator));
// ...
this.dialogSet.add(new DateTimePrompt(RESERVATION_DATE_PROMPT, this.dateValidator));
// ...
```

---

**Validateur de taille de tiers**

Nous limitons la taille des tiers qui peut effectuer une réservation. La plage valide est définie par la propriété _validations_ que nous avons utilisée pour appeler l’invite de taille de tiers.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// Validates whether the party size is appropriate to make a reservation.
private async Task<bool> RangeValidatorAsync(
    PromptValidatorContext<int> promptContext,
    CancellationToken cancellationToken)
{
    // Check whether the input could be recognized as an integer.
    if (!promptContext.Recognized.Succeeded)
    {
        await promptContext.Context.SendActivityAsync(
            "I'm sorry, I do not understand. Please enter the number of people in your party.",
            cancellationToken: cancellationToken);
        return false;
    }

    // Check whether the party size is appropriate.
    var size = promptContext.Recognized.Value;
    var validRange = promptContext.Options.Validations as Range;
    if (size < validRange.Min || size > validRange.Max)
    {
        await promptContext.Context.SendActivitiesAsync(
            new Activity[]
            {
                MessageFactory.Text($"Sorry, we can only take reservations for parties " +
                    $"of {validRange.Min} to {validRange.Max}."),
                promptContext.Options.RetryPrompt,
            },
            cancellationToken: cancellationToken);
        return false;
    }

    return true;
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
async rangeValidator(promptContext) {
    // Check whether the input could be recognized as an integer.
    if (!promptContext.recognized.succeeded) {
        await promptContext.context.sendActivity(
            "I'm sorry, I do not understand. Please enter the number of people in your party.");
        return false;
    }
    else if (promptContext.recognized.value % 1 != 0) {
        await promptContext.context.sendActivity(
            "I'm sorry, I don't understand fractional people.");
        return false;
    }

    // Check whether the party size is appropriate.
    var size = promptContext.recognized.value;
    if (size < promptContext.options.validations.min
        || size > promptContext.options.validations.max) {
        await promptContext.context.sendActivity(
            'Sorry, we can only take reservations for parties of '
            + `${promptContext.options.validations.min} to `
            + `${promptContext.options.validations.max}.`);
        await promptContext.context.sendActivity(promptContext.options.retryPrompt);
        return false;
    }

    return true;
}
```

---

**Validation de date et heure**

Dans le validateur de date de réservation, nous limitons les réservations à une heure ou plus à partir de l’heure actuelle. Nous gardons la première résolution correspondant à nos critères et nous effaçons le reste.

Ce code de validation n’est pas exhaustif, et il convient mieux pour l’entrée qui analyse à une date et une heure précises. Il montre certaines des options pour la validation d’une invite de date-heure, et votre implémentation dépendra des informations que vous essayez de collecter par l’intermédiaire de l’utilisateur.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// Validates whether the reservation date is appropriate.
private async Task<bool> DateValidatorAsync(
    PromptValidatorContext<IList<DateTimeResolution>> promptContext,
    CancellationToken cancellationToken = default(CancellationToken))
{
    // Check whether the input could be recognized as an integer.
    if (!promptContext.Recognized.Succeeded)
    {
        await promptContext.Context.SendActivityAsync(
            "I'm sorry, I do not understand. Please enter the date or time for your reservation.",
            cancellationToken: cancellationToken);

        return false;
    }

    // Check whether any of the recognized date-times are appropriate,
    // and if so, return the first appropriate date-time.
    var earliest = DateTime.Now.AddHours(1.0);
    var value = promptContext.Recognized.Value.FirstOrDefault(v =>
        DateTime.TryParse(v.Value ?? v.Start, out var time) && DateTime.Compare(earliest, time) <= 0);

    if (value != null)
    {
        promptContext.Recognized.Value.Clear();
        promptContext.Recognized.Value.Add(value);
        return true;
    }

    await promptContext.Context.SendActivityAsync(
            "I'm sorry, we can't take reservations earlier than an hour from now.",
            cancellationToken: cancellationToken);

    return false;
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
async dateValidator(promptContext) {
    // Check whether the input could be recognized as an integer.
    if (!promptContext.recognized.succeeded) {
        await promptContext.context.sendActivity(
            "I'm sorry, I do not understand. Please enter the date or time for your reservation.");
        return false;
    }

    // Check whether any of the recognized date-times are appropriate,
    // and if so, return the first appropriate date-time.
    const earliest = Date.now() + (60 * 60 * 1000);
    let value = null;
    promptContext.recognized.value.forEach(candidate => {
        // TODO: update validation to account for time vs date vs date-time vs range.
        const time = new Date(candidate.value || candidate.start);
        if (earliest < time.getTime()) {
            value = candidate;
        }
    });
    if (value) {
        promptContext.recognized.value = [value];
        return true;
    }

    await promptContext.context.sendActivity(
        "I'm sorry, we can't take reservations earlier than an hour from now.");
    return false;
}
```

---

L’invite de date-heure retourne une liste ou un tableau des _résolutions de date-heure_ possibles correspondant à l’entrée de l’utilisateur. Par exemple, 9:00 peut vouloir dire 9 h 00 ou 21 h 00, et dimanche est également ambigu. En outre, une résolution de date-heure peut représenter une date, une heure, une date-heure ou une plage. L’invite de date-heure utilise [Microsoft/Recognizers-Text](https://github.com/Microsoft/Recognizers-Text) pour analyser l’entrée de l’utilisateur.

## <a name="update-the-turn-handler"></a>Mettre à jour le gestionnaire de tour

Mettez à jour le gestionnaire de tour du bot pour démarrer le dialogue et acceptez une valeur de retour du dialogue lorsqu’il est terminé. Ici, nous supposons que l’utilisateur interagit avec un bot, que le bot a un dialogue en cascade actif et que l’étape suivante du dialogue utilise une invite.

Lorsque l’utilisateur envoie un message au bot, il effectue les opérations suivantes :

1. Le bot récupère les informations d’état.
1. Le bot crée un contexte de dialogue
    - S’il n’y a pas de dialogue actif, il démarre le dialogue, sauf si l’utilisateur a déjà fait une réservation.
    - Si un dialogue est actif, le bot le continue. Si le dialogue se termine, les détails de la réservation sont enregistrés dans le cache d’état.
1. Le bot enregistre les changements de l’état.

Quand une étape du dialogue appelle la méthode _prompt_ du contexte de l’étape :

1. Une nouvelle instance de l’invite est créée, placée sur la pile du dialogue et démarrée. (Le dialogue principal attend que l’invite se termine avant de continuer.)
1. L’invite envoie une activité à l’utilisateur, lui demandant une entrée.

Quand l’entrée est envoyée à l’invite :

1. L’invite tente de traiter l’entrée, en fonction du type d’invite, comme une invite de nombre ou une invite de choix, etc.
1. Si l’invite inclut une validation personnalisée, le code de cette validation est exécuté.
1. Si l’entrée est validée, l’invite se termine et retourne l’entrée traitée ; sinon, l’invite se redémarre elle-même.

**Gestion des résultats des invites**

L’action que vous effectuez avec le résultat de l’invite dépend de la raison pour laquelle vous avez demandé les informations à l’utilisateur. Options disponibles :

- Utilisez les informations pour contrôler le flux de votre dialogue, par exemple lorsque l’utilisateur répond à une invite de confirmation ou de choix.
- Mettez en cache les informations contenues dans l’état du dialogue, comme la définition d’une valeur dans la propriété _valeurs_ du contexte d’étape en cascade, puis retournez ensuite les informations collectées lorsque le dialogue se termine.
- Enregistrez les informations dans l’état du bot. Cela vous oblige à concevoir votre dialogue de façon à avoir accès aux accesseurs de la propriété d’état du bot.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
public async Task OnTurnAsync(
    ITurnContext turnContext,
    CancellationToken cancellationToken = default(CancellationToken))
{
    switch (turnContext.Activity.Type)
    {
        // On a message from the user:
        case ActivityTypes.Message:

            // Get the current reservation info from state.
            var reservation = await _accessors.ReservationAccessor.GetAsync(
                turnContext,
                () => null,
                cancellationToken);

            // Generate a dialog context for our dialog set.
            var dc = await _dialogSet.CreateContextAsync(turnContext, cancellationToken);

            if (dc.ActiveDialog is null)
            {
                // If there is no active dialog, check whether we have a reservation yet.
                if (reservation is null)
                {
                    // If not, start the dialog.
                    await dc.BeginDialogAsync(ReservationDialog, null, cancellationToken);
                }
                else
                {
                    // Otherwise, send a status message.
                    await turnContext.SendActivityAsync(
                        $"We'll see you on {reservation.Date}.",
                        cancellationToken: cancellationToken);
                }
            }
            else
            {
                // Continue the dialog.
                var dialogTurnResult = await dc.ContinueDialogAsync(cancellationToken);

                // If the dialog completed this turn, record the reservation info.
                if (dialogTurnResult.Status is DialogTurnStatus.Complete)
                {
                    reservation = (Reservation)dialogTurnResult.Result;
                    await _accessors.ReservationAccessor.SetAsync(
                        turnContext,
                        reservation,
                        cancellationToken);

                    // Send a confirmation message to the user.
                    await turnContext.SendActivityAsync(
                        $"Your party of {reservation.Size} is confirmed for " +
                        $"{reservation.Date} in {reservation.Location}.",
                        cancellationToken: cancellationToken);
                }
            }

            // Save the updated dialog state into the conversation state.
            await _accessors.ConversationState.SaveChangesAsync(
                turnContext, false, cancellationToken);
            break;
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
async onTurn(turnContext) {
    switch (turnContext.activity.type) {
        case ActivityTypes.Message:
            // Get the current reservation info from state.
            const reservation = await this.reservationAccessor.get(turnContext, null);

            // Generate a dialog context for our dialog set.
            const dc = await this.dialogSet.createContext(turnContext);

            if (!dc.activeDialog) {
                // If there is no active dialog, check whether we have a reservation yet.
                if (!reservation) {
                    // If not, start the dialog.
                    await dc.beginDialog(RESERVATION_DIALOG);
                }
                else {
                    // Otherwise, send a status message.
                    await turnContext.sendActivity(
                        `We'll see you on ${reservation.date}.`);
                }
            }
            else {
                // Continue the dialog.
                const dialogTurnResult = await dc.continueDialog();

                // If the dialog completed this turn, record the reservation info.
                if (dialogTurnResult.status === DialogTurnStatus.complete) {
                    await this.reservationAccessor.set(
                        turnContext,
                        dialogTurnResult.result);

                    // Send a confirmation message to the user.
                    await turnContext.sendActivity(
                        `Your party of ${dialogTurnResult.result.size} is ` +
                        `confirmed for ${dialogTurnResult.result.date} in ` +
                        `${dialogTurnResult.result.location}.`);
                }
            }

            // Save the updated dialog state into the conversation state.
            await this.conversationState.saveChanges(turnContext, false);
            break;
        case ActivityTypes.EndOfConversation:
        case ActivityTypes.DeleteUserData:
            break;
        default:
            break;
    }
}
```

---

Vous pouvez utiliser des techniques similaires pour valider les réponses à tout type d’invite.

## <a name="test-your-bot"></a>Tester votre robot

1. Exécutez l’exemple en local sur votre machine. Si vous avez besoin d’instructions, consultez le fichier LISEZ-MOI pour [ C# ](https://aka.ms/dialog-prompt-cs) ou [JS](https://aka.ms/dialog-prompt-js).
2. Démarrez l’émulateur, envoyez des messages comme indiqué ci-dessous pour tester le bot.

![exemple d’invite de dialogue de test](~/media/emulator-v4/test-dialog-prompt.png)

## <a name="additional-resources"></a>Ressources supplémentaires

Pour appeler une invite de commandes directement à partir de votre gestionnaire de tour, consultez l’exemple de _validations d’invite_ pour [ C# ](https://aka.ms/cs-prompt-validation-sample) ou [JS](https://aka.ms/js-prompt-validation-sample).

La bibliothèque de dialogues inclut également une _invite OAuth_ pour obtenir un _jeton OAuth_ permettant d’accéder à une autre application pour le compte de l’utilisateur. Pour plus d’informations sur l’authentification, consultez comment [ajouter l’authentification](bot-builder-authentication.md) à votre bot.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Créer des flux de conversation avancés à l’aide des branches et boucles](bot-builder-dialog-manage-complex-conversation-flow.md)
