---
title: Gérer un flux de conversation simple avec des dialogues | Microsoft Docs
description: Découvrez comment gérer un flux de conversation simple avec des dialogues dans le Kit de développement logiciel (SDK) Bot Builder pour Node.js.
keywords: flux de conversation simple, dialogues, invites, cascades, jeu de dialogues
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 9/25/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 07035c8f0dfc7473192d8c51667ed1f5cefbc555
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2018
ms.locfileid: "49999394"
---
# <a name="manage-simple-conversation-flow-with-dialogs"></a>Gérer un flux de conversation simple avec des dialogues

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Vous pouvez gérer des flux de conversation simples et complexes avec la bibliothèque de dialogues. Dans un flux de conversation simple, l’utilisateur commence à la première étape d’une *cascade*, puis continue jusqu’à la dernière étape où se termine la conversation. Les [flux de conversation complexes](~/v4sdk/bot-builder-dialog-manage-complex-conversation-flow.md) comprennent des branches et une boucle.

<!-- TODO: This paragraph belongs in a conceptual topic. -->

Les dialogues sont des structures de votre bot se comportant comme des fonctions dans le programme de votre bot. Les dialogues créent les messages que votre bot envoie, et mènent à bien les tâches informatiques nécessaires. Ils sont conçus pour effectuer des opérations spécifiques, dans un ordre particulier. Ils peuvent être appelés de différentes façons : en réponse à un utilisateur, en réponse à des stimuli extérieurs ou par d’autres dialogues.

L’utilisation de dialogues permet au développeur du bot de guider le flux conversationnel. Vous pouvez créer plusieurs dialogues et les associer pour créer un flux de conversation que votre bot doit pouvoir gérer. La bibliothèque **Dialogues** du Kit de développement logiciel (SDK) Bot Builder comprend des fonctionnalités intégrées, telles que des _invites_, des _dialogues en cascade_ et des _dialogues de composant_ conçus pour vous aider à gérer un flux de conversation. Vous pouvez utiliser les invites pour demander différents types d’informations aux utilisateurs. Vous pouvez recourir aux cascades pour combiner plusieurs étapes dans une séquence. En outre, vous pouvez utiliser les dialogues de composant pour créer des systèmes de dialogues modulaires contenant plusieurs dialogues secondaires.

Dans cet article, nous utilisons des _ensembles de dialogues_ pour créer un flux de conversation contenant à la fois des invites et des cascades. Nous allons nous appuyer sur le code de l’exemple de **l’invite à plusieurs tours** [[C#](https://aka.ms/cs-multi-prompts-sample)|[JS](https://aka.ms/js-multi-prompts-sample)].

Pour une présentation des dialogues, consultez [Dialogs library](bot-builder-concept-dialog.md) (Bibliothèque Dialogues) et [Dialog state](bot-builder-dialog-state.md) (État des dialogues).

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Généralement, pour utiliser des dialogues, vous avez besoin du package NuGet `Microsoft.Bot.Builder.Dialogs` pour votre projet ou solution.

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Généralement, pour utiliser des dialogues, vous avez besoin de la bibliothèque `botbuilder-dialogs`, que vous pouvez télécharger depuis NPM.

---

## <a name="using-dialogs-to-guide-the-user-through-steps"></a>Utilisation de dialogues pour guider l’utilisateur dans des étapes

Dans cet exemple, nous allons créer un dialogue composé de plusieurs étapes pour inviter l’utilisateur à fournir certaines informations via un ensemble de dialogues.

### <a name="create-a-dialog-with-waterfall-steps"></a>Créer un dialogue avec des étapes en cascade

**WaterfallDialog** est une implémentation spécifique d’un dialogue, qui est couramment utilisée pour recueillir des informations auprès de l’utilisateur ou pour le guider dans une série de tâches. Chaque étape de la conversation est implémentée en tant que fonction. À chaque étape, le bot [invite l’utilisateur à effectuer une entrée](bot-builder-prompts.md), attend une réponse, puis passe le résultat à l’étape suivante. Le résultat de la première fonction est transmis en tant qu’argument à la fonction suivante, et ainsi de suite.

L’exemple de code ci-après définit un tableau de délégués qui représente les étapes d’une **cascade**. Après chaque invite, le bot accuse réception de l’entrée utilisateur. De nombreuses méthodes vous permettent de conserver l’entrée que vous avez recueillie dans un dialogue. Consultez [Conserver les données utilisateur](bot-builder-tutorial-persist-user-inputs.md) pour découvrir certaines des options qui s’offrent à vous.

Cet exemple écrit les informations directement dans le profil utilisateur à mesure qu’elles sont recueillies dans le dialogue.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Dans cet exemple, le dialogue en cascade est défini au sein du fichier de bot.

Référencez les espaces de noms utilisés dans ce fichier.

```csharp
using System;
using System.Linq;
using System.Threading;
using System.Threading.Tasks;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Bot.Schema;
```

Définissez une propriété d’instance pour l’ensemble de dialogues.

```csharp
/// <summary>
/// The <see cref="DialogSet"/> that contains all the Dialogs that can be used at runtime.
/// </summary>
private DialogSet _dialogs;
```

Créez l’ensemble de dialogues au sein du constructeur du bot en ajoutant les invites et le dialogue en cascade à l’ensemble.

```csharp
/// <summary>
/// Initializes a new instance of the <see cref="MultiTurnPromptsBot"/> class.
/// </summary>
/// <param name="accessors">A class containing <see cref="IStatePropertyAccessor{T}"/> used to manage state.</param>
public MultiTurnPromptsBot(MultiTurnPromptsBotAccessors accessors)
{
    _accessors = accessors ?? throw new ArgumentNullException(nameof(accessors));

    // The DialogSet needs a DialogState accessor, it will call it when it has a turn context.
    _dialogs = new DialogSet(accessors.ConversationDialogState);

    // This array defines how the Waterfall will execute.
    var waterfallSteps = new WaterfallStep[]
    {
        NameStepAsync,
        NameConfirmStepAsync,
        AgeStepAsync,
        ConfirmStepAsync,
        SummaryStepAsync,
    };

    // Add named dialogs to the DialogSet. These names are saved in the dialog state.
    _dialogs.Add(new WaterfallDialog("details", waterfallSteps));
    _dialogs.Add(new TextPrompt("name"));
    _dialogs.Add(new NumberPrompt<int>("age"));
    _dialogs.Add(new ConfirmPrompt("confirm"));
}
```

Enfin, définissez chacune des étapes en tant que méthode distincte. Vous pouvez également définir les étapes en ligne via des expressions lambda.

```csharp
/// <summary>
/// One of the functions that make up the <see cref="WaterfallDialog"/>.
/// </summary>
/// <param name="stepContext">The <see cref="WaterfallStepContext"/> gives access to the executing dialog runtime.</param>
/// <param name="cancellationToken">A <see cref="CancellationToken"/>.</param>
/// <returns>A <see cref="DialogTurnResult"/> to communicate some flow control back to the containing WaterfallDialog.</returns>
private static async Task<DialogTurnResult> NameStepAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
{
    // WaterfallStep always finishes with the end of the Waterfall or with another dialog; here it is a Prompt Dialog.
    // Running a prompt here means the next WaterfallStep will be run when the users response is received.
    return await stepContext.PromptAsync("name", new PromptOptions { Prompt = MessageFactory.Text("Please enter your name.") }, cancellationToken);
}

/// <summary>
/// One of the functions that make up the <see cref="WaterfallDialog"/>.
/// </summary>
/// <param name="stepContext">The <see cref="WaterfallStepContext"/> gives access to the executing dialog runtime.</param>
/// <param name="cancellationToken">A <see cref="CancellationToken"/>.</param>
/// <returns>A <see cref="DialogTurnResult"/> to communicate some flow control back to the containing WaterfallDialog.</returns>
private async Task<DialogTurnResult> NameConfirmStepAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
{
    // Get the current profile object from user state.
    var userProfile = await _accessors.UserProfile.GetAsync(stepContext.Context, () => new UserProfile(), cancellationToken);

    // Update the profile.
    userProfile.Name = (string)stepContext.Result;

    // We can send messages to the user at any point in the WaterfallStep.
    await stepContext.Context.SendActivityAsync(MessageFactory.Text($"Thanks {stepContext.Result}."), cancellationToken);

    // WaterfallStep always finishes with the end of the Waterfall or with another dialog; here it is a Prompt Dialog.
    return await stepContext.PromptAsync("confirm", new PromptOptions { Prompt = MessageFactory.Text("Would you like to give your age?") }, cancellationToken);
}

/// <summary>
/// One of the functions that make up the <see cref="WaterfallDialog"/>.
/// </summary>
/// <param name="stepContext">The <see cref="WaterfallStepContext"/> gives access to the executing dialog runtime.</param>
/// <param name="cancellationToken">A <see cref="CancellationToken"/>.</param>
/// <returns>A <see cref="DialogTurnResult"/> to communicate some flow control back to the containing WaterfallDialog.</returns>
private async Task<DialogTurnResult> AgeStepAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
{
    if ((bool)stepContext.Result)
    {
        // User said "yes" so we will be prompting for the age.

        // Get the current profile object from user state.
        var userProfile = await _accessors.UserProfile.GetAsync(stepContext.Context, () => new UserProfile(), cancellationToken);

        // WaterfallStep always finishes with the end of the Waterfall or with another dialog, here it is a Prompt Dialog.
        return await stepContext.PromptAsync("age", new PromptOptions { Prompt = MessageFactory.Text("Please enter your age.") }, cancellationToken);
    }
    else
    {
        // User said "no" so we will skip the next step. Give -1 as the age.
        return await stepContext.NextAsync(-1, cancellationToken);
    }
}

/// <summary>
/// One of the functions that make up the <see cref="WaterfallDialog"/>.
/// </summary>
/// <param name="stepContext">The <see cref="WaterfallStepContext"/> gives access to the executing dialog runtime.</param>
/// <param name="cancellationToken">A <see cref="CancellationToken"/>.</param>
/// <returns>A <see cref="DialogTurnResult"/> to communicate some flow control back to the containing WaterfallDialog.</returns>
private async Task<DialogTurnResult> ConfirmStepAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
{
    // Get the current profile object from user state.
    var userProfile = await _accessors.UserProfile.GetAsync(stepContext.Context, () => new UserProfile(), cancellationToken);

    // Update the profile.
    userProfile.Age = (int)stepContext.Result;

    // We can send messages to the user at any point in the WaterfallStep.
    if (userProfile.Age == -1)
    {
        await stepContext.Context.SendActivityAsync(MessageFactory.Text($"No age given."), cancellationToken);
    }
    else
    {
        // We can send messages to the user at any point in the WaterfallStep.
        await stepContext.Context.SendActivityAsync(MessageFactory.Text($"I have your age as {userProfile.Age}."), cancellationToken);
    }

    // WaterfallStep always finishes with the end of the Waterfall or with another dialog, here it is a Prompt Dialog.
    return await stepContext.PromptAsync("confirm", new PromptOptions { Prompt = MessageFactory.Text("Is this ok?") }, cancellationToken);
}

/// <summary>
/// One of the functions that make up the <see cref="WaterfallDialog"/>.
/// </summary>
/// <param name="stepContext">The <see cref="WaterfallStepContext"/> gives access to the executing dialog runtime.</param>
/// <param name="cancellationToken">A <see cref="CancellationToken"/>.</param>
/// <returns>A <see cref="DialogTurnResult"/> to communicate some flow control back to the containing WaterfallDialog.</returns>
private async Task<DialogTurnResult> SummaryStepAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
{
    if ((bool)stepContext.Result)
    {
        // Get the current profile object from user state.
        var userProfile = await _accessors.UserProfile.GetAsync(stepContext.Context, () => new UserProfile(), cancellationToken);

        // We can send messages to the user at any point in the WaterfallStep.
        if (userProfile.Age == -1)
        {
            await stepContext.Context.SendActivityAsync(MessageFactory.Text($"I have your name as {userProfile.Name}."), cancellationToken);
        }
        else
        {
            await stepContext.Context.SendActivityAsync(MessageFactory.Text($"I have your name as {userProfile.Name} and age as {userProfile.Age}."), cancellationToken);
        }
    }
    else
    {
        // We can send messages to the user at any point in the WaterfallStep.
        await stepContext.Context.SendActivityAsync(MessageFactory.Text("Thanks. Your profile will not be kept."), cancellationToken);
    }

    // WaterfallStep always finishes with the end of the Waterfall or with another dialog, here it is the end.
    return await stepContext.EndDialogAsync(cancellationToken: cancellationToken);
}
```

Le dialogue est exécuté à partir du gestionnaire de tours du bot, qui commence par créer un contexte de dialogue et continue ou débute le dialogue comme nécessaire. Il enregistre ensuite la conversation et l’état de l’utilisateur à la fin du tour.

```csharp
// Run the DialogSet - let the framework identify the current state of the dialog from
// the dialog stack and figure out what (if any) is the active dialog.
var dialogContext = await _dialogs.CreateContextAsync(turnContext, cancellationToken);
var results = await dialogContext.ContinueDialogAsync(cancellationToken);

// If the DialogTurnStatus is Empty we should start a new dialog.
if (results.Status == DialogTurnStatus.Empty)
{
    await dialogContext.BeginDialogAsync("details", null, cancellationToken);
}
```

```csharp
// Save the dialog state into the conversation state.
await _accessors.ConversationState.SaveChangesAsync(turnContext, false, cancellationToken);

// Save the user profile updates into the user state.
await _accessors.UserState.SaveChangesAsync(turnContext, false, cancellationToken);
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Dans cet exemple, le dialogue en cascade est défini au sein du fichier **bot.js**.

Importez les objets dont vous avez besoin pour le code.

```javascript
const { ActivityTypes } = require('botbuilder');
const { ChoicePrompt, DialogSet, NumberPrompt, TextPrompt, WaterfallDialog } = require('botbuilder-dialogs');
```

Définissez et créez l’ensemble de dialogues au sein du constructeur du bot en ajoutant les invites et le dialogue en cascade à l’ensemble.

```javascript
/**
*
* @param {ConversationState} conversationState A ConversationState object used to store the dialog state.
* @param {UserState} userState A UserState object used to store values specific to the user.
*/
constructor(conversationState, userState) {
    // Create a new state accessor property. See https://aka.ms/about-bot-state-accessors to learn more about bot state and state accessors.
    this.conversationState = conversationState;
    this.userState = userState;

    this.dialogState = this.conversationState.createProperty(DIALOG_STATE_PROPERTY);

    this.userProfile = this.userState.createProperty(USER_PROFILE_PROPERTY);

    this.dialogs = new DialogSet(this.dialogState);

    // Add prompts that will be used by the main dialogs.
    this.dialogs.add(new TextPrompt(NAME_PROMPT));
    this.dialogs.add(new ChoicePrompt(CONFIRM_PROMPT));
    this.dialogs.add(new NumberPrompt(AGE_PROMPT, async (prompt) => {
        if (prompt.recognized.succeeded) {
            if (prompt.recognized.value <= 0) {
                await prompt.context.sendActivity(`Your age can't be less than zero.`);
                return false;
            } else {
                return true;
            }
        }

        return false;
    }));

    // Create a dialog that asks the user for their name.
    this.dialogs.add(new WaterfallDialog(WHO_ARE_YOU, [
        this.promptForName.bind(this),
        this.confirmAgePrompt.bind(this),
        this.promptForAge.bind(this),
        this.captureAge.bind(this)
    ]));

    // Create a dialog that displays a user name after it has been collected.
    this.dialogs.add(new WaterfallDialog(HELLO_USER, [
        this.displayProfile.bind(this)
    ]));
}
```

Enfin, définissez chacune des étapes en tant que méthode distincte. Vous pouvez également définir les étapes en ligne via des expressions lambda.

```javascript
// This step in the dialog prompts the user for their name.
async promptForName(step) {
    return await step.prompt(NAME_PROMPT, `What is your name, human?`);
}

// This step captures the user's name, then prompts whether or not to collect an age.
async confirmAgePrompt(step) {
    const user = await this.userProfile.get(step.context, {});
    user.name = step.result;
    await this.userProfile.set(step.context, user);
    await step.prompt(CONFIRM_PROMPT, 'Do you want to give your age?', ['yes', 'no']);
}

// This step checks the user's response - if yes, the bot will proceed to prompt for age.
// Otherwise, the bot will skip the age step.
async promptForAge(step) {
    if (step.result && step.result.value === 'yes') {
        return await step.prompt(AGE_PROMPT, `What is your age?`,
            {
                retryPrompt: 'Sorry, please specify your age as a positive number or say cancel.'
            }
        );
    } else {
        return await step.next(-1);
    }
}

// This step captures the user's age.
async captureAge(step) {
    const user = await this.userProfile.get(step.context, {});
    if (step.result !== -1) {
        user.age = step.result;
        await this.userProfile.set(step.context, user);
        await step.context.sendActivity(`I will remember that you are ${ step.result } years old.`);
    } else {
        await step.context.sendActivity(`No age given.`);
    }
    return await step.endDialog();
}

// This step displays the captured information back to the user.
async displayProfile(step) {
    const user = await this.userProfile.get(step.context, {});
    if (user.age) {
        await step.context.sendActivity(`Your name is ${ user.name } and you are ${ user.age } years old.`);
    } else {
        await step.context.sendActivity(`Your name is ${ user.name } and you did not share your age.`);
    }
    return await step.endDialog();
}
```

Le dialogue est exécuté à partir du gestionnaire de tours du bot, qui commence par créer un contexte de dialogue et continue ou débute le dialogue comme nécessaire. Il enregistre ensuite la conversation et l’état de l’utilisateur à la fin du tour.

```javascript
// Create a dialog context object.
const dc = await this.dialogs.createContext(turnContext);
```

```javascript
// If the bot has not yet responded, continue processing the current dialog.
await dc.continueDialog();
```

```javascript
// Start the sample dialog in response to any other input.
if (!turnContext.responded) {
    const user = await this.userProfile.get(dc.context, {});
    if (user.name) {
        await dc.beginDialog(HELLO_USER);
    } else {
        await dc.beginDialog(WHO_ARE_YOU);
    }
}
```

```javascript
// Save changes to the user state.
await this.userState.saveChanges(turnContext);

// End this turn by saving changes to the conversation state.
await this.conversationState.saveChanges(turnContext);
```

---

## <a name="dialog-context-and-waterfall-step-context-objects"></a>Contexte du dialogue et objets de contexte de l’étape en cascade

Utilisez l’objet de contexte de dialogue pour interagir avec un ensemble de dialogues issu du gestionnaire de tours de votre bot.
Utilisez l’objet de contexte d’étape en cascade pour interagir avec un ensemble de dialogues défini dans une étape en cascade.

## <a name="to-start-a-dialog"></a>Pour démarrer un dialogue

Pour démarrer un dialogue, transmettez l’élément *dialogId* du dialogue que vous souhaitez démarrer à la méthode _beginDialog_, _prompt_ ou _replaceDialog_ du contexte de dialogue. La méthode _beginDialog_ envoie (push) le dialogue en haut de la pile, tandis que la méthode _replaceDialog_ dépile le dialogue actuel et envoie le dialogue de remplacement dans la pile.

La méthode _prompt_ du contexte de dialogue est une méthode d’assistance qui accepte des arguments et construit les options appropriées pour l’invite. Ensuite, elle démarre le dialogue d’invite. Pour plus d’informations sur les invites, consultez [Inviter les utilisateurs à saisir une entrée](bot-builder-prompts.md).

## <a name="to-end-a-dialog"></a>Pour terminer un dialogue

La méthode _end dialog_ termine un dialogue en le dépilant et renvoie un résultat facultatif au dialogue parent.

Il est recommandé d’appeler explicitement la méthode _endDialog_ à la fin du dialogue.

## <a name="to-clear-the-dialog-stack"></a>Pour effacer la pile de dialogues

Si vous souhaitez dépiler tous les dialogues, vous pouvez effacer la pile de dialogues en appelant la méthode _cancel all dialogs_ du contexte de dialogue.

## <a name="to-repeat-a-dialog"></a>Pour répéter un dialogue

Pour répéter un dialogue, utilisez la méthode _replace dialog_, qui dépile le dialogue actuel, puis envoie (push) le dialogue de remplacement en haut de la pile et démarre ce dialogue. Il s’agit là d’un excellent moyen de traiter les [flux de conversation complexes](~/v4sdk/bot-builder-dialog-manage-complex-conversation-flow.md) et de gérer les menus.

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous avez appris à gérer des flux de conversation simples, examinons la manière dont vous pouvez tirer profit de la méthode _replace dialog_ pour traiter des flux de conversation complexes.

> [!div class="nextstepaction"]
> [Gérer des flux de conversation complexes](bot-builder-dialog-manage-complex-conversation-flow.md)
