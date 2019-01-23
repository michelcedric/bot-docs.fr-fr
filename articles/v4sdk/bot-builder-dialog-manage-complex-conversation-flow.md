---
title: Créer des flux de conversation avancés à l’aide de branches et de boucles | Microsoft Docs
description: Découvrez comment gérer un flux de conversation complexe avec des dialogues dans le kit SDK Bot Framework.
keywords: flux de conversation complexe, répétition, boucle, menu, dialogues, invites, cascades, jeu de dialogues
author: JonathanFingold
ms.author: v-jofing
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 11/28/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 27bcd53e9f3d582a1206c5ed74cc844acab795b4
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/11/2019
ms.locfileid: "54225884"
---
# <a name="create-advanced-conversation-flow-using-branches-and-loops"></a>Créer des flux de conversation avancés à l’aide de branches et de boucles

[!INCLUDE [pre-release-label](~/includes/pre-release-label.md)]

Dans cet article, nous allons vous montrer comment gérer des conversations complexes qui forment des branches et des boucles. Nous vous montrerons également comment passer des arguments entre différentes parties du dialogue.

## <a name="prerequisites"></a>Prérequis

- [Bot Framework Emulator](https://github.com/Microsoft/BotFramework-Emulator/blob/master/README.md#download)
- Le code dans cet article est basé sur l’exemple de **dialogue complexe**. Vous aurez besoin d’une copie de l’exemple en [C# ](https://aka.ms/cs-complex-dialog-sample) ou en [JS](https://aka.ms/js-complex-dialog-sample).
- Connaissances des [concepts de base des bots](bot-builder-basics.md), de la [bibliothèque de dialogues](bot-builder-concept-dialog.md), de [l’état de dialogue](bot-builder-dialog-state.md) et des fichiers [.bot](bot-file-basics.md).

## <a name="about-the-sample"></a>À propos de l’exemple

Dans cet exemple, un bot inscrit des utilisateurs qui sont ensuite invités à évaluer jusqu’à deux entreprises parmi celles figurant dans une liste.

- Le bot demande le nom et l’âge de l’utilisateur, puis _choisit une branche_ en fonction de l’âge de l’utilisateur.
  - Si l’utilisateur est trop jeune, il ne lui demande pas d’évaluer une entreprise.
  - Si l’utilisateur a l’âge minimum, il commence à collecter les préférences d’évaluation de l’utilisateur.
    - Il autorise l’utilisateur à sélectionner une entreprise à évaluer.
    - Si l’utilisateur choisit une entreprise, le bot _effectue une boucle_ pour lui permettre de choisir une autre entreprise.
- Enfin, il remercie l’utilisateur pour sa participation.

Le bot utilise deux dialogues en cascade et quelques invites pour gérer une conversation complexe.

## <a name="configure-state-for-your-bot"></a>Configurer l’état de votre bot

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Définissez les informations utilisateur à collecter.

```csharp
public class UserProfile
{
    public string Name { get; set; }
    public int Age { get; set; }

    //The list of companies the user wants to review.
    public List<string> CompaniesToReview { get; set; } = new List<string>();
}
```

Définissez la classe pour héberger les objets de gestion d’état et les accesseurs de propriété d’état pour le bot.

```csharp
public class ComplexDialogBotAccessors
{
    public ComplexDialogBotAccessors(ConversationState conversationState, UserState userState)
    {
        ConversationState = conversationState ?? throw new ArgumentNullException(nameof(conversationState));
        UserState = userState ?? throw new ArgumentNullException(nameof(userState));
    }

    public IStatePropertyAccessor<DialogState> DialogStateAccessor { get; set; }
    public IStatePropertyAccessor<UserProfile> UserProfileAccessor { get; set; }

    public ConversationState ConversationState { get; }
    public UserState UserState { get; }
}
```

Créez l’objet de gestion d’état et inscrivez la classe d’accesseurs dans la méthode `ConfigureServices` de la classe `Statup`.

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // Register the bot.

    // Create conversation and user state management objects, using memory storage.
    IStorage dataStore = new MemoryStorage();
    var conversationState = new ConversationState(dataStore);
    var userState = new UserState(dataStore);

    // Create and register state accessors.
    // Accessors created here are passed into the IBot-derived class on every turn.
    services.AddSingleton<ComplexDialogBotAccessors>(sp =>
    {
        // Create the custom state accessor.
        // State accessors enable other components to read and write individual properties of state.
        var accessors = new ComplexDialogBotAccessors(conversationState, userState)
        {
            DialogStateAccessor = conversationState.CreateProperty<DialogState>("DialogState"),
            UserProfileAccessor = userState.CreateProperty<UserProfile>("UserProfile"),
        };

        return accessors;
    });
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Dans le fichier **index.js**, nous définissons les objets de gestion d’état.

```javascript
const { BotFrameworkAdapter, MemoryStorage, UserState, ConversationState } = require('botbuilder');

// ...

// Define state store for your bot.
const memoryStorage = new MemoryStorage();

// Create user and conversation state with in-memory storage provider.
const userState = new UserState(memoryStorage);
const conversationState = new ConversationState(memoryStorage);

// Create the bot.
const myBot = new MyBot(conversationState, userState);
```

Le constructeur du bot crée les accesseurs de propriété d’état pour le bot.

---

## <a name="initialize-your-bot"></a>Initialiser votre bot

Créez un _ensemble de dialogues_ pour le bot, auquel nous ajouterons tous les dialogues de cet exemple.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Créez l’ensemble de dialogues au sein du constructeur du bot en ajoutant les invites et les deux dialogues en cascade à l’ensemble.

Ici, nous définissons chacune des étapes en tant que méthode distincte. Nous les implémenterons dans la section suivante.

```csharp
public class ComplexDialogBot : IBot
{
    // Define constants for the bot...

    // Define properties for the bot's accessors and dialog set.
    private readonly ComplexDialogBotAccessors _accessors;
    private readonly DialogSet _dialogs;

    // Initialize the bot and add dialogs and prompts to the dialog set.
    public ComplexDialogBot(ComplexDialogBotAccessors accessors)
    {
        _accessors = accessors ?? throw new ArgumentNullException(nameof(accessors));

        // Create a dialog set for the bot. It requires a DialogState accessor, with which
        // to retrieve the dialog state from the turn context.
        _dialogs = new DialogSet(accessors.DialogStateAccessor);

        // Add the prompts we need to the dialog set.
        _dialogs
            .Add(new TextPrompt(NamePrompt))
            .Add(new NumberPrompt<int>(AgePrompt))
            .Add(new ChoicePrompt(SelectionPrompt));

        // Add the dialogs we need to the dialog set.
        _dialogs.Add(new WaterfallDialog(TopLevelDialog)
            .AddStep(NameStepAsync)
            .AddStep(AgeStepAsync)
            .AddStep(StartSelectionStepAsync)
            .AddStep(AcknowledgementStepAsync));

        _dialogs.Add(new WaterfallDialog(ReviewSelectionDialog)
            .AddStep(SelectionStepAsync)
            .AddStep(LoopStepAsync));
    }

    // Turn handler and other supporting methods...
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Dans le fichier **bot.js**, définissez et créez l’ensemble de dialogues au sein du constructeur du bot en ajoutant les invites et les dialogues en cascade à l’ensemble.

Ici, nous définissons chacune des étapes en tant que méthode distincte. Nous les implémenterons dans la section suivante.

```javascript
const { ActivityTypes } = require('botbuilder');
const { DialogSet, WaterfallDialog, TextPrompt, NumberPrompt, ChoicePrompt, DialogTurnStatus } = require('botbuilder-dialogs');

// Define constants for the bot...

class MyBot {
    constructor(conversationState, userState) {
        // Create the state property accessors and save the state management objects.
        this.dialogStateAccessor = conversationState.createProperty(DIALOG_STATE_PROPERTY);
        this.userProfileAccessor = userState.createProperty(USER_PROFILE_PROPERTY);
        this.conversationState = conversationState;
        this.userState = userState;

        // Create a dialog set for the bot. It requires a DialogState accessor, with which
        // to retrieve the dialog state from the turn context.
        this.dialogs = new DialogSet(this.dialogStateAccessor);

        // Add the prompts we need to the dialog set.
        this.dialogs
            .add(new TextPrompt(NAME_PROMPT))
            .add(new NumberPrompt(AGE_PROMPT))
            .add(new ChoicePrompt(SELECTION_PROMPT));

        // Add the dialogs we need to the dialog set.
        this.dialogs.add(new WaterfallDialog(TOP_LEVEL_DIALOG)
            .addStep(this.nameStep.bind(this))
            .addStep(this.ageStep.bind(this))
            .addStep(this.startSelectionStep.bind(this))
            .addStep(this.acknowledgementStep.bind(this)));

        this.dialogs.add(new WaterfallDialog(REVIEW_SELECTION_DIALOG)
            .addStep(this.selectionStep.bind(this))
            .addStep(this.loopStep.bind(this)));
    }

    // Turn handler and other supporting methods...
}
```

---

## <a name="implement-the-steps-for-the-waterfall-dialogs"></a>Implémenter les étapes pour les dialogues en cascade

Nous allons à présent implémenter les étapes de nos deux dialogues.

### <a name="the-top-level-dialog"></a>Dialogue de niveau supérieur

Le dialogue initial de niveau supérieur comporte quatre étapes :

1. Demander le nom de l’utilisateur.
1. Demander l’âge de l’utilisateur.
1. Choisir une branche en fonction de l’âge de l’utilisateur.
1. Enfin, remercier l’utilisateur pour sa participation et retourner les informations collectées.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// The first step of the top-level dialog.
private static async Task<DialogTurnResult> NameStepAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
{
    // Create an object in which to collect the user's information within the dialog.
    stepContext.Values[UserInfo] = new UserProfile();

    // Ask the user to enter their name.
    return await stepContext.PromptAsync(
        NamePrompt,
        new PromptOptions { Prompt = MessageFactory.Text("Please enter your name.") },
        cancellationToken);
}

// The second step of the top-level dialog.
private async Task<DialogTurnResult> AgeStepAsync(
    WaterfallStepContext stepContext,
    CancellationToken cancellationToken)
{
    // Set the user's name to what they entered in response to the name prompt.
    ((UserProfile)stepContext.Values[UserInfo]).Name = (string)stepContext.Result;

    // Ask the user to enter their age.
    return await stepContext.PromptAsync(
        AgePrompt,
        new PromptOptions { Prompt = MessageFactory.Text("Please enter your age.") },
        cancellationToken);
}

// The third step of the top-level dialog.
private async Task<DialogTurnResult> StartSelectionStepAsync(
    WaterfallStepContext stepContext,
    CancellationToken cancellationToken)
{
    // Set the user's age to what they entered in response to the age prompt.
    int age = (int)stepContext.Result;
    ((UserProfile)stepContext.Values[UserInfo]).Age = age;

    if (age < 25)
    {
        // If they are too young, skip the review-selection dialog, and pass an empty list to the next step.
        await stepContext.Context.SendActivityAsync(
            MessageFactory.Text("You must be 25 or older to participate."),
            cancellationToken);
        return await stepContext.NextAsync(new List<string>(), cancellationToken);
    }
    else
    {
        // Otherwise, start the review-selection dialog.
        return await stepContext.BeginDialogAsync(ReviewSelectionDialog, null, cancellationToken);
    }
}

// The final step of the top-level dialog.
private async Task<DialogTurnResult> AcknowledgementStepAsync(
    WaterfallStepContext stepContext,
    CancellationToken cancellationToken)
{
    // Set the user's company selection to what they entered in the review-selection dialog.
    List<string> list = stepContext.Result as List<string>;
    ((UserProfile)stepContext.Values[UserInfo]).CompaniesToReview = list ?? new List<string>();

    // Thank them for participating.
    await stepContext.Context.SendActivityAsync(
        MessageFactory.Text($"Thanks for participating, {((UserProfile)stepContext.Values[UserInfo]).Name}."),
        cancellationToken);

    // Exit the dialog, returning the collected user information.
    return await stepContext.EndDialogAsync(stepContext.Values[UserInfo], cancellationToken);
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
async nameStep(stepContext) {
    // Create an object in which to collect the user's information within the dialog.
    stepContext.values[USER_INFO] = {};

    // Ask the user to enter their name.
    return await stepContext.prompt(NAME_PROMPT, 'Please enter your name.');
}

async ageStep(stepContext) {
    // Set the user's name to what they entered in response to the name prompt.
    stepContext.values[USER_INFO].name = stepContext.result;

    // Ask the user to enter their age.
    return await stepContext.prompt(AGE_PROMPT, 'Please enter your age.');
}

async startSelectionStep(stepContext) {
    // Set the user's age to what they entered in response to the age prompt.
    stepContext.values[USER_INFO].age = stepContext.result;

    if (stepContext.result < 25) {
        // If they are too young, skip the review-selection dialog, and pass an empty list to the next step.
        await stepContext.context.sendActivity('You must be 25 or older to participate.');
        return await stepContext.next([]);
    } else {
        // Otherwise, start the review-selection dialog.
        return await stepContext.beginDialog(REVIEW_SELECTION_DIALOG);
    }
}

async acknowledgementStep(stepContext) {
    // Set the user's company selection to what they entered in the review-selection dialog.
    const list = stepContext.result || [];
    stepContext.values[USER_INFO].companiesToReview = list;

    // Thank them for participating.
    await stepContext.context.sendActivity(`Thanks for participating, ${stepContext.values[USER_INFO].name}.`);

    // Exit the dialog, returning the collected user information.
    return await stepContext.endDialog(stepContext.values[USER_INFO]);
}
```

---

### <a name="the-review-selection-dialog"></a>Dialogue de sélection de l’évaluation

Le dialogue de sélection de l’évaluation comporte deux étapes :

1. Demander à l’utilisateur de choisir une entreprise à évaluer ou choisir `done` pour terminer.
1. Répéter ce dialogue ou en sortir selon le cas.

Dans cette conception, le dialogue de niveau supérieur précède toujours le dialogue de sélection de l’évaluation sur la pile, et le dialogue de sélection de l’évaluation peut être considéré comme un enfant du dialogue de niveau supérieur.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// The first step of the review-selection dialog.
private async Task<DialogTurnResult> SelectionStepAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
{
    // Continue using the same selection list, if any, from the previous iteration of this dialog.
    List<string> list = stepContext.Options as List<string> ?? new List<string>();
    stepContext.Values[CompaniesSelected] = list;

    // Create a prompt message.
    string message;
    if (list.Count is 0)
    {
        message = $"Please choose a company to review, or `{DoneOption}` to finish.";
    }
    else
    {
        message = $"You have selected **{list[0]}**. You can review an additional company, " +
            $"or choose `{DoneOption}` to finish.";
    }

    // Create the list of options to choose from.
    List<string> options = _companyOptions.ToList();
    options.Add(DoneOption);
    if (list.Count > 0)
    {
        options.Remove(list[0]);
    }

    // Prompt the user for a choice.
    return await stepContext.PromptAsync(
        SelectionPrompt,
        new PromptOptions
        {
            Prompt = MessageFactory.Text(message),
            RetryPrompt = MessageFactory.Text("Please choose an option from the list."),
            Choices = ChoiceFactory.ToChoices(options),
        },
        cancellationToken);
}

// The final step of the review-selection dialog.
private async Task<DialogTurnResult> LoopStepAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
{
    // Retrieve their selection list, the choice they made, and whether they chose to finish.
    List<string> list = stepContext.Values[CompaniesSelected] as List<string>;
    FoundChoice choice = (FoundChoice)stepContext.Result;
    bool done = choice.Value == DoneOption;

    if (!done)
    {
        // If they chose a company, add it to the list.
        list.Add(choice.Value);
    }

    if (done || list.Count is 2)
    {
        // If they're done, exit and return their list.
        return await stepContext.EndDialogAsync(list, cancellationToken);
    }
    else
    {
        // Otherwise, repeat this dialog, passing in the list from this iteration.
        return await stepContext.ReplaceDialogAsync(ReviewSelectionDialog, list, cancellationToken);
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
async selectionStep(stepContext) {
    // Continue using the same selection list, if any, from the previous iteration of this dialog.
    const list = Array.isArray(stepContext.options) ? stepContext.options : [];
    stepContext.values[COMPANIES_SELECTED] = list;

    // Create a prompt message.
    let message;
    if (list.length === 0) {
        message = 'Please choose a company to review, or `' + DONE_OPTION + '` to finish.';
    } else {
        message = `You have selected **${list[0]}**. You can review an addition company, ` +
            'or choose `' + DONE_OPTION + '` to finish.';
    }

    // Create the list of options to choose from.
    const options = list.length > 0
        ? COMPANY_OPTIONS.filter(function (item) { return item !== list[0] })
        : COMPANY_OPTIONS.slice();
    options.push(DONE_OPTION);

    // Prompt the user for a choice.
    return await stepContext.prompt(SELECTION_PROMPT, {
        prompt: message,
        retryPrompt: 'Please choose an option from the list.',
        choices: options
    });
}

async loopStep(stepContext) {
    // Retrieve their selection list, the choice they made, and whether they chose to finish.
    const list = stepContext.values[COMPANIES_SELECTED];
    const choice = stepContext.result;
    const done = choice.value === DONE_OPTION;

    if (!done) {
        // If they chose a company, add it to the list.
        list.push(choice.value);
    }

    if (done || list.length > 1) {
        // If they're done, exit and return their list.
        return await stepContext.endDialog(list);
    } else {
        // Otherwise, repeat this dialog, passing in the list from this iteration.
        return await stepContext.replaceDialog(REVIEW_SELECTION_DIALOG, list);
    }
}
```

---

## <a name="update-the-bots-turn-handler"></a>Mettre à jour le gestionnaire de tour du bot

Le gestionnaire de tour du bot répète le flux de conversation défini par ces dialogues.
Quand nous recevons un message de l’utilisateur :

1. Continuer le dialogue actif, le cas échéant.
   - Si aucun dialogue n’est actif, nous effaçons le profil utilisateur et démarrons le dialogue de niveau supérieur.
   - Si le dialogue actif est terminé, nous collectons et enregistrons les informations retournées, puis affichons un message d’état.
   - Sinon, le dialogue actif est toujours en cours de traitement, et il n’y a rien d’autre à faire pour le moment.
1. Enregistrer l’état de la conversation pour rendre persistante toute mise à jour de l’état du dialogue.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
{
    if (turnContext == null)
    {
        throw new ArgumentNullException(nameof(turnContext));
    }

    if (turnContext.Activity.Type == ActivityTypes.Message)
    {
        // Run the DialogSet - let the framework identify the current state of the dialog from
        // the dialog stack and figure out what (if any) is the active dialog.
        DialogContext dialogContext = await _dialogs.CreateContextAsync(turnContext, cancellationToken);
        DialogTurnResult results = await dialogContext.ContinueDialogAsync(cancellationToken);
        switch (results.Status)
        {
            case DialogTurnStatus.Cancelled:
            case DialogTurnStatus.Empty:
                // If there is no active dialog, we should clear the user info and start a new dialog.
                await _accessors.UserProfileAccessor.SetAsync(turnContext, new UserProfile(), cancellationToken);
                await _accessors.UserState.SaveChangesAsync(turnContext, false, cancellationToken);
                await dialogContext.BeginDialogAsync(TopLevelDialog, null, cancellationToken);
                break;

            case DialogTurnStatus.Complete:
                // If we just finished the dialog, capture and display the results.
                UserProfile userInfo = results.Result as UserProfile;
                string status = "You are signed up to review "
                    + (userInfo.CompaniesToReview.Count is 0
                        ? "no companies"
                        : string.Join(" and ", userInfo.CompaniesToReview))
                    + ".";
                await turnContext.SendActivityAsync(status);
                await _accessors.UserProfileAccessor.SetAsync(turnContext, userInfo, cancellationToken);
                await _accessors.UserState.SaveChangesAsync(turnContext, false, cancellationToken);
                break;

            case DialogTurnStatus.Waiting:
                // If there is an active dialog, we don't need to do anything here.
                break;
        }

        await _accessors.ConversationState.SaveChangesAsync(turnContext, false, cancellationToken);
    }

    // Processes ConversationUpdate Activities to welcome the user.
    else if (turnContext.Activity.Type == ActivityTypes.ConversationUpdate)
    {
        // Welcome new users...
    }
    else
    {
        // Give a default reply for all other activity types...
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
async onTurn(turnContext) {
    if (turnContext.activity.type === ActivityTypes.Message) {
        // Run the DialogSet - let the framework identify the current state of the dialog from
        // the dialog stack and figure out what (if any) is the active dialog.
        const dialogContext = await this.dialogs.createContext(turnContext);
        const results = await dialogContext.continueDialog();
        switch (results.status) {
            case DialogTurnStatus.cancelled:
            case DialogTurnStatus.empty:
                // If there is no active dialog, we should clear the user info and start a new dialog.
                await this.userProfileAccessor.set(turnContext, {});
                await this.userState.saveChanges(turnContext);
                await dialogContext.beginDialog(TOP_LEVEL_DIALOG);
                break;
            case DialogTurnStatus.complete:
                // If we just finished the dialog, capture and display the results.
                const userInfo = results.result;
                const status = 'You are signed up to review '
                    + (userInfo.companiesToReview.length === 0 ? 'no companies' : userInfo.companiesToReview.join(' and '))
                    + '.';
                await turnContext.sendActivity(status);
                await this.userProfileAccessor.set(turnContext, userInfo);
                await this.userState.saveChanges(turnContext);
                break;
            case DialogTurnStatus.waiting:
                // If there is an active dialog, we don't need to do anything here.
                break;
        }
        await this.conversationState.saveChanges(turnContext);
    } else if (turnContext.activity.type === ActivityTypes.ConversationUpdate) {
        // Welcome new users...
    } else {
        // Give a default reply for all other activity types...
    }
}
```

---

## <a name="test-your-dialog"></a>Tester votre dialogue

1. Exécutez l’exemple en local sur votre machine. Si vous avez besoin d’instructions, consultez le fichier LISEZ-MOI pour l’exemple en [C# ](https://aka.ms/cs-complex-dialog-sample) ou en [JS](https://aka.ms/js-complex-dialog-sample).
1. Utilisez l’émulateur pour tester le bot comme indiqué ci-dessous.

![exemple de dialogue complexe à des fins de test](~/media/emulator-v4/test-complex-dialog.png)

## <a name="additional-resources"></a>Ressources supplémentaires

Pour obtenir une introduction sur la façon d’implémenter un dialogue, consultez [Implémenter un flux de conversation séquentiel](bot-builder-dialog-manage-conversation-flow.md). Cet article utilise un dialogue unique en cascade et quelques invites pour créer une interaction simple qui pose à l’utilisateur une série de questions.

La bibliothèque de dialogues inclut la validation de base des invites. Vous pouvez également ajouter une validation personnalisée. Pour plus d’informations, consultez [Collecter les entrées utilisateur avec une invite de dialogue](bot-builder-prompts.md).

Pour simplifier votre code de dialogue et le réutiliser dans plusieurs bots, vous pouvez définir chaque partie d’un ensemble de dialogues comme une classe distincte.
Pour plus d’informations, consultez [Réutiliser des dialogues](bot-builder-compositcontrol.md).

## <a name="next-steps"></a>Étapes suivantes

Vous pouvez améliorer vos bots de manière à ce qu’il réagissent à des entrées supplémentaires, comme « aide » ou « annuler », qui peuvent interrompre le flux normal de la conversation.

> [!div class="nextstepaction"]
> [Gérer les interruptions de l’utilisateur](bot-builder-howto-handle-user-interrupt.md)
