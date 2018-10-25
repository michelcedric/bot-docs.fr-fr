---
title: Inviter les utilisateurs à saisir une entrée à l’aide de la bibliothèque de dialogues | Microsoft Docs
description: Découvrez comment demander aux utilisateurs de saisir une entrée à l’aide de la bibliothèque Dialogs du Kit de développement logiciel (SDK) Bot Builder pour Node.js.
keywords: invites, dialogues, AttachmentPrompt, ChoicePrompt, ConfirmPrompt, DatetimePrompt, NumberPrompt, TextPrompt, inviter, validation
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 9/25/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 16ef274bc7e8301825e574c566a49d53f01115c1
ms.sourcegitcommit: aef7d80ceb9c3ec1cfb40131709a714c42960965
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/17/2018
ms.locfileid: "49383124"
---
# <a name="prompt-users-for-input-using-the-dialogs-library"></a>Inviter les utilisateurs à saisir une entrée à l’aide de la bibliothèque Dialogs

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Pour interagir avec les utilisateurs, un bot collecte généralement des informations à travers des questions. Pour ce faire, vous pouvez utiliser directement la méthode d’_envoi d’activité_ de l’objet [contexte de tour](~/v4sdk/bot-builder-basics.md#defining-a-turn), puis traiter le message entrant suivant en tant que réponse. Toutefois, le SDK Bot Builder offre une bibliothèque de **dialogues** qui fournit des méthodes permettant de poser des questions plus facilement et de s’assurer que la réponse correspond à un type de données spécifique ou respecte des règles de validation personnalisées. Cette rubrique explique comment y parvenir à l’aide des **invites** pour demander à un utilisateur d’effectuer une saisie.

Cet article décrit comment utiliser des invites au sein d’un dialogue. Pour plus d’informations sur l’utilisation de dialogues en général, consultez l’article [Gérer un flux de conversation simple avec des dialogues](bot-builder-dialog-manage-conversation-flow.md).

## <a name="prompt-types"></a>Types d’invites

La bibliothèque de dialogues propose divers types d’invites, chacune étant utilisée pour recueillir un type de réponse différent.

| Prompt | Description |
|:----|:----|
| **AttachmentPrompt** | Invite l’utilisateur à ajouter une pièce jointe comme un document ou une image. |
| **ChoicePrompt** | Invite l’utilisateur à choisir parmi un ensemble d’options. |
| **ConfirmPrompt** | Invite l’utilisateur à confirmer son action. |
| **DatetimePrompt** | Invite l’utilisateur à saisir une date/heure. Les utilisateurs puissent répondre à l’aide d’un langage naturel, par exemple « demain à 20 h 00 » ou « vendredi à 10 h ». Le Kit SDK Bot Framework utilise l’entité prédéfinie LUIS `builtin.datetimeV2`. Pour plus d’informations, consultez [builtin.datetimev2](https://docs.microsoft.com/azure/cognitive-services/luis/luis-reference-prebuilt-entities#builtindatetimev2). |
| **NumberPrompt** | Invite l’utilisateur à saisir une valeur numérique. L’utilisateur peut répondre par « 10 » ou « dix ». Si la réponse est « dix », par exemple, l’invite convertit la réponse en une valeur numérique et renvoie `10` comme résultat. |
| **TextPrompt** | Invite l’utilisateur à saisir une chaîne de texte. |

## <a name="add-references-to-prompt-library"></a>Ajouter des références à la bibliothèque d’invites

Vous pouvez accéder à la bibliothèque de **dialogues** en ajoutant le package **botbuilder-dialogs** à votre bot. Nous décrivons les dialogues dans l’article [Gérer un flux de conversation simple avec des dialogues](bot-builder-dialog-manage-conversation-flow.md), mais nous allons utiliser des dialogues pour nos invites.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Installez le package **Microsoft.Bot.Builder.Dialogs** à partir de NuGet.

Incluez ensuite une référence à la bibliothèque à partir du code de votre bot.

```cs
using Microsoft.Bot.Builder.Dialogs;
```

Vous aurez besoin de configurer l’état de dialogue de la conversation via les accesseurs. Nous n’allons pas étudier ce code en détail ici, mais consultez l’article sur l’[état](bot-builder-howto-v4-state.md) pour en savoir plus.

Dans les options de votre bot, dans le fichier **Startup.cs**, commencez par définir vos objets d’état, puis ajoutez le singleton pour fournir la classe d’accesseur pour le constructeur de bot. La classe pour `BotAccessor` stocke simplement l’état de la conversation et de l’utilisateur, ainsi que les accesseurs pour chacun de ces éléments. La définition complète de classe est fournie dans l’exemple associé à la fin de cet article. 

```cs
    services.AddBot<MultiTurnPromptsBot>(options =>
    {
        InitCredentialProvider(options);

        // Create and add conversation state.
        var convoState = new ConversationState(dataStore);
        options.State.Add(convoState);

        // Create and add user state.
        var userState = new UserState(dataStore);
        options.State.Add(userState);
    });

    services.AddSingleton(sp =>
    {
        // We need to grab the conversationState we added on the options in the previous step
        var options = sp.GetRequiredService<IOptions<BotFrameworkOptions>>().Value;
        if (options == null)
        {
            throw new InvalidOperationException("BotFrameworkOptions must be configured prior to setting up the State Accessors");
        }

        var conversationState = options.State.OfType<ConversationState>().FirstOrDefault();
        if (conversationState == null)
        {
            throw new InvalidOperationException("ConversationState must be defined and added before adding conversation-scoped state accessors.");
        }

        var userState = options.State.OfType<UserState>().FirstOrDefault();
        if (userState == null)
        {
            throw new InvalidOperationException("UserState must be defined and added before adding user-scoped state accessors.");
        }

        // The dialogs will need a state store accessor. Creating it here once (on-demand) allows the dependency injection
        // to hand it to our IBot class that is create per-request.
        var accessors = new BotAccessors(conversationState, userState)
        {
            ConversationDialogState = conversationState.CreateProperty<DialogState>("DialogState"),
            UserProfile = userState.CreateProperty<UserProfile>("UserProfile"),
        };

        return accessors;
    });
```

Ensuite, dans le code de votre bot, définissez les objets suivants pour l’ensemble de dialogues.

```cs
    private readonly BotAccessors _accessors;

    /// <summary>
    /// The <see cref="DialogSet"/> that contains all the Dialogs that can be used at runtime.
    /// </summary>
    private DialogSet _dialogs;

    /// <summary>
    /// Initializes a new instance of the <see cref="MultiTurnPromptsBot"/> class.
    /// </summary>
    /// <param name="accessors">A class containing <see cref="IStatePropertyAccessor{T}"/> used to manage state.</param>
    public MultiTurnPromptsBot(BotAccessors accessors)
    {
        _accessors = accessors ?? throw new ArgumentNullException(nameof(accessors));

        // The DialogSet needs a DialogState accessor, it will call it when it has a turn context.
        _dialogs = new DialogSet(accessors.ConversationDialogState);

        // ...
        // other constructor items
        // ...
    }
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Créez un bot JavaScript à l’aide du modèle Echo. Pour plus d’informations, consultez le didacticiel de [démarrage rapide de JavaScript](../javascript/bot-builder-javascript-quickstart.md).

Installez le package de dialogues à partir de npm :

```cmd
npm install --save botbuilder-dialogs
```

Pour utiliser des **dialogues** dans votre bot, incluez-les dans le code de celui-ci.

1. Dans votre fichier **bot.js**, ajoutez ce qui suit.

    ```javascript
    // Import components from the dialogs library.
    const { DialogSet, TextPrompt, WaterfallDialog } = require("botbuilder-dialogs");

    // Name for the dialog state property accessor.
    const DIALOG_STATE_PROPERTY = 'dialogState';

    // Define the names for the prompts and dialogs for the dialog set.
    const TEXT_PROMPT = 'textPrompt';
    const MAIN_DIALOG = 'mainDialog';
    ```

    Le _jeu de dialogues_ contiendra les dialogues de ce bot, et nous utiliserons le _texte d’invite_ pour demander des informations à l’utilisateur. Nous aurons également besoin d’un accesseur de propriété d’état de dialogue que le jeu de dialogues pourra utiliser pour suivre son état.

1. Mettez à jour le code du constructeur de votre bot. Nous en dirons plus à ce sujet sous peu.

    ```javascript
      constructor(conversationState) {
        // Track the conversation state object.
        this.conversationState = conversationState;

        // Create a state property accessor for the dialog set.
        this.dialogState = conversationState.createProperty(DIALOG_STATE_PROPERTY);
    }
    ```

---

## <a name="prompt-the-user"></a>Inviter l'utilisateur

Pour inviter un utilisateur à saisir une entrée, définissez une invite à l’aide de l’une des classes intégrées (**TextPrompt**, par exemple), ajoutez-la à votre ensemble de dialogues, puis attribuez-lui un ID de dialogue.

Une fois que l’invite est ajoutée, utilisez-la dans un dialogue en cascade à deux étapes. Un dialogue *en cascade* permet de définir une séquence d’étapes. Plusieurs invites peuvent s’enchaîner pour créer des conversations à plusieurs étapes. Pour plus d’informations, consultez la section [Utilisation de dialogues](bot-builder-dialog-manage-conversation-flow.md#using-dialogs-to-guide-the-user-through-steps) de l’article [Gérer un flux de conversation simple avec des dialogues](bot-builder-dialog-manage-conversation-flow.md).

Par exemple, le dialogue suivant invite l’utilisateur à saisir son nom, puis utilise sa réponse pour l’accueillir. Au premier tour, le dialogue demande son nom à l’utilisateur. La réponse de l’utilisateur est transmise à la deuxième fonction d’étape sous forme de paramètre. La deuxième fonction traite l’entrée et envoie le message d’accueil personnalisé.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Chaque invite que vous utilisez dans votre dialogue reçoit également un nom, utilisé par le dialogue ou votre bot pour accéder à l’invite. Dans tous ces exemples, nous exposons les ID d’invites comme des constantes.

Dans le constructeur de bot, ajoutez des définitions pour votre cascade à deux étapes et l’invite de dialogue à utiliser. Dans notre exemple, nous les ajoutons en tant que fonctions indépendantes, mais elles peuvent être définies en tant que fonctions lambda incluses si vous préférez.

```csharp
 public MultiTurnPromptsBot(BotAccessors accessors)
{
    _accessors = accessors ?? throw new ArgumentNullException(nameof(accessors));

    // The DialogSet needs a DialogState accessor, it will call it when it has a turn context.
    _dialogs = new DialogSet(accessors.ConversationDialogState);

    // This array defines how the Waterfall will execute.
    var waterfallSteps = new WaterfallStep[]
    {
        NameStepAsync,
        SayHiAsync,
    };

    _dialogs.Add(new WaterfallDialog("details", waterfallSteps));
    _dialogs.Add(new TextPrompt("name"));
}
```

Ensuite, définissez les deux étapes de cascade au sein de votre bot. Pour le texte d’invite, vous spécifiez l’ID de *nom* du `TextPrompt` que vous avez défini ci-dessus. Notez que les noms des méthodes correspondent à ceux de `WaterfallStep[]` ci-dessus. Nos futurs exemples n’incluent pas ce code, mais sachez que pour les étapes supplémentaires, vous devez ajouter le nom de la méthode dans l’ordre approprié de `WaterfallStep[]`.

```cs
    private static async Task<DialogTurnResult> NameStepAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
    {
        // WaterfallStep always finishes with the end of the Waterfall or with another dialog; here it is a Prompt Dialog.
        // Running a prompt here means the next WaterfallStep will be run when the users response is received.
        return await stepContext.PromptAsync("name", new PromptOptions { Prompt = MessageFactory.Text("Please enter your name.") }, cancellationToken);
    }

    private static async Task<DialogTurnResult> SayHiAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
    {
        await stepContext.Context.SendActivityAsync($"Hi {stepContext.Result}");

        return await stepContext.EndDialogAsync(cancellationToken);
    }
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

1. Dans le constructeur du bot, créez le jeu de dialogues et ajoutez-y un texte d’invite et un dialogue en cascade.

    ```javascript
    // Create the dialog set, and add the prompt and the waterfall dialog.
    this.dialogs = new DialogSet(this.dialogState)
        .add(new TextPrompt(TEXT_PROMPT))
        .add(new WaterfallDialog(MAIN_DIALOG, [
            async (step) => {
                // The results of this prompt will be passed to the next step.
                return await step.prompt(TEXT_PROMPT, 'What is your name?');
            },
            async (step) => {
                // The result property contains the result from the previous step.
                const userName = step.result;
                await step.context.sendActivity(`Hi ${userName}!`);
                return await step.endDialog();
            }
        ]));
    ```

1. Mettez à jour le gestionnaire de tour du bot pour qu’il exécute le dialogue.

    ```javascript
    async onTurn(turnContext) {
        // See https://aka.ms/about-bot-activity-message to learn more about the message and other activity types.
        if (turnContext.activity.type === ActivityTypes.Message) {
            // Create a dialog context for the dialog set.
            const dc = await this.dialogs.createContext(turnContext);
            // Continue the dialog if it's active.
            await dc.continueDialog();
            if (!turnContext.responded) {
                // Otherwise, start the dialog.
                await dc.beginDialog(MAIN_DIALOG);
            }
        } else {
            // Send a default message for activity types that we don't handle.
            await turnContext.sendActivity(`[${turnContext.activity.type} event detected]`);
        }
        // Save state changes
        await this.conversationState.saveChanges(turnContext);
        }
    }
    ```

---

> [!NOTE]
> Pour démarrer un dialogue, obtenez un contexte de dialogue et utilisez sa méthode _begin dialog_. Pour plus d’informations, consultez l’article [Gérer un flux de conversation simple avec des dialogues](./bot-builder-dialog-manage-conversation-flow.md).

## <a name="reusable-prompts"></a>Invites réutilisables

Une invite peut être réutilisée pour poser différentes questions, tant que les réponses sont du même type. Par exemple, l’exemple de code ci-dessus a défini un texte d’invite et l’a utilisé pour demander à l’utilisateur son nom. Vous pouvez également utiliser cette même invite pour demander à l’utilisateur une autre chaîne de texte, par exemple, « Où travaillez-vous ? ».

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Dans cet exemple, l’ID de notre texte d’invite, *nom*, ne participe pas à la clarté du code. Toutefois, c’est un exemple qui montre bien que vous pouvez choisir l’ID d’invite.

À présent, nos méthodes incluent une troisième étape permettant de demander où travaille l’utilisateur.

```cs
    private static async Task<DialogTurnResult> NameStepAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
    {
        // WaterfallStep always finishes with the end of the Waterfall or with another dialog; here it is a Prompt Dialog.
        // Running a prompt here means the next WaterfallStep will be run when the users response is received.
        return await stepContext.PromptAsync("name", new PromptOptions { Prompt = MessageFactory.Text("Please enter your name.") }, cancellationToken);
    }

    private static async Task<DialogTurnResult> WorkAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
    {
        await stepContext.Context.SendActivityAsync($"Hi {stepContext.Result}!");

        return await stepContext.PromptAsync("name", new PromptOptions { Prompt = MessageFactory.Text("Where do you work?") }, cancellationToken);
    }

    private static async Task<DialogTurnResult> SayHiAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
    {
        await stepContext.Context.SendActivityAsync($"{stepContext.Result} is a cool place!");

        return await stepContext.EndDialogAsync(cancellationToken);
    }
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Dans le constructeur du bot, modifiez la cascade pour poser une deuxième question.

```javascript
// Create the dialog set, and add the prompt and the waterfall dialog.
this.dialogs = new DialogSet(this.dialogState)
    .add(new TextPrompt(TEXT_PROMPT))
    .add(new WaterfallDialog(MAIN_DIALOG, [
    async (step) => {
        // Ask the user for their name.
        return await step.prompt(TEXT_PROMPT, 'What is your name?');
    },
    async (step) => {
        // Acknowledge their response and ask for their place of work.
        const userName = step.result;
        return await step.prompt(TEXT_PROMPT, `Hi ${userName}; where do you work?`);
    },
    async (step) => {
        // Acknowledge their response and exit the dialog.
        const workPlace = step.result;
        await step.context.sendActivity(`${workPlace} is a cool place!`);
        return await step.endDialog();
    }
    ]));
```

---

Si vous devez utiliser plusieurs invites différentes, choisissez un ID *dialogId* unique pour chaque invite. Chaque dialogue ou invite ajouté à un ensemble de dialogues doit posséder un ID unique. Vous pouvez également créer plusieurs **invites** de dialogue du même type. Par exemple, vous pouvez créer deux dialogues **TextPrompt** pour l’exemple ci-dessus :

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```cs
_dialogs.Add(new WaterfallDialog("details", waterfallSteps));
_dialogs.Add(new TextPrompt("name"));
_dialogs.Add(new TextPrompt("workplace"));
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Par exemple, vous pouvez remplacer ceci.

```javascript
.add(new TextPrompt(TEXT_PROMPT))
```

Par ce qui suit.

```javascript
.add(new TextPrompt('namePrompt'))
.add(new TextPrompt('workPlacePrompt'))
```

Puis vous pouvez mettre à jour les étapes de cascade respectives afin qu’elles utilisent ces invites avec leurs noms respectifs.

---

Pour bien réutiliser le code, définissez une seule valeur `TextPrompt` à appliquer à toutes ces invites, car elles attendent toutes une réponse sous forme de chaîne de texte. La possibilité de nommer les dialogues s’avère utile lorsque vous devez appliquer différentes règles de validation aux entrées des invites. Voyons maintenant comment valider les réponses aux invites à l’aide d’une valeur `NumberPrompt`.

## <a name="specify-prompt-options"></a>Spécifier des options d’invite

Lorsque vous utilisez une invite au sein d’une étape de dialogue, vous pouvez également fournir des options d’invite, par exemple une chaîne de nouvelle invite.

Spécifier une chaîne de nouvelle invite est utile lorsque l’entrée utilisateur est incapable de répondre à une invite, soit parce que son format ne peut pas être analysé par l’invite, par exemple « demain » en réponse à une invite numérique, ou parce que l’entrée ne remplit pas un critère de validation. L’invite numérique peut interpréter un large éventail d’entrées, par exemple « douze » ou « un quart », ainsi que « 12 » et « 0,25 ».

La langue est un paramètre facultatif pour certaines invites, comme **NumberPrompt**. Ce paramètre aide l’invite à analyser plus précisément l’entrée, mais il n’est pas obligatoire.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Le code suivant ajoute une invite numérique à un ensemble existant de dialogues, **_dialogs**.

```csharp
_dialogs.Add(new NumberPrompt<int>("age"));
```

Au sein d’une étape de dialogue, le code suivant invite l’utilisateur à effectuer une saisie et à fournir une chaîne de nouvelle invite si son entrée ne peut pas être interprétée comme une valeur numérique.

```csharp
return await stepContext.PromptAsync(
    "age",
    new PromptOptions {
        Prompt = MessageFactory.Text("Please enter your age."),
        RetryPrompt = MessageFactory.Text("I didn't get that. Please enter a valid age."),
    },
    cancellationToken);
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Importez la classe `NumberPrompt` à partir de la bibliothèque de dialogues.

```javascript
const { NumberPrompt } = require("botbuilder-dialogs");
```

Utilisez l’invite numérique dans votre dialogue en cascade, en spécifiant les chaînes d’invite initiale et d’invite de nouvelle tentative.

```javascript
// Create the dialog set, and add the prompt and the waterfall dialog.
this.dialogs = new DialogSet(this.dialogState)
    .add(new NumberPrompt('partySize'))
    .add(new WaterfallDialog(MAIN_DIALOG, [
    async (step) => {
        // Ask the user for their party size.
        return await step.prompt('partySize', {
            prompt: 'How many people in your party?',
            retryPrompt: 'Sorry, please specify the number of people in your party.'
        });
    },
    async (step) => {
        // Acknowledge their response and exit the dialog.
        const partySize = step.result;
        await step.context.sendActivity(`That's a party of ${partySize}, thanks.`);
        return await step.endDialog();
    }
]));
```

---

L’invite de choix dispose d’un paramètre obligatoire supplémentaire : une liste de choix à proposer à l’utilisateur.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Lorsque nous utilisons **ChoicePrompt** pour demander à l’utilisateur de choisir parmi un ensemble d’options, nous devons spécifier l’invite avec cet ensemble d’options, fournie dans un objet **PromptOptions**. Ici, nous utilisons **ChoiceFactory** pour convertir une liste d’options dans un format approprié.

```csharp
private static async Task<DialogTurnResult> FavoriteColorAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
{
    await stepContext.Context.SendActivityAsync($"Hi {stepContext.Result}!");

    return await stepContext.PromptAsync(
        "color",
        new PromptOptions {
            Prompt = MessageFactory.Text("What's your favorite color?"),
            Choices = ChoiceFactory.ToChoices(new List<string> { "blue", "green", "red" }),
        },
        cancellationToken);
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Importez la classe `NumberPrompt` à partir de la bibliothèque de dialogues.

```javascript
const { ChoicePrompt } = require("botbuilder-dialogs");
```

Utilisez l’invite de choix dans votre dialogue en cascade, en spécifiant les choix disponibles.

```javascript
// Create the dialog set, and add the prompt and the waterfall dialog.
const list = ['green', 'blue', 'red', 'yellow'];
this.dialogs = new DialogSet(this.dialogState)
    .add(new ChoicePrompt('choicePrompt'))
    .add(new WaterfallDialog(MAIN_DIALOG, [
    async (step) => {
        // Ask the user for their party size.
        return await step.prompt('choicePrompt', {
            prompt: 'Please choose a color:',
            retryPrompt: 'Sorry, please choose a color from the list.',
            choices: list
        });
    },
    async (step) => {
        // Acknowledge their response and exit the dialog.
        const choice = step.result;
        await step.context.sendActivity(`That's ${choice.value}, thanks.`);
        return await step.endDialog();
    }
]));
```

---

## <a name="validate-a-prompt-response"></a>Valider la réponse à une invite

Vous pouvez valider la réponse à une invite avant de retourner la valeur à l’étape suivante de la **cascade**. Par exemple, pour valider une invite **NumberPrompt** au sein d’une plage de nombres compris entre **6** et **20**, vous utilisez une fonction de validation semblable à la suivante :

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Effectuez une modification lors de l’ajout de l’invite à votre ensemble de dialogues afin d’inclure la fonction de validation.

```cs
_dialogs.Add(new NumberPrompt<int>("partySize", PartySizeValidatorAsync));
```

La validation est alors définie en tant que méthode propre, indiquant true ou false selon le résultat de la validation. Si la valeur false est retournée, l’utilisateur reçoit une nouvelle invite.

```cs
private Task<bool> PartySizeValidatorAsync(PromptValidatorContext<int> promptContext, CancellationToken cancellationToken)
{
    var result = promptContext.Recognized.Value;

    if (result < 6 || result > 20)
    {
        return Task.FromResult(false);
    }

    return Task.FromResult(true);
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Ajoutez une méthode de validation lors de la création de l’invite.

```javascript
// Create the dialog set, and add the prompt and the waterfall dialog.
this.dialogs = new DialogSet(this.dialogState)
    .add(new NumberPrompt('partySizePrompt', async (promptContext) =>                                                 {
        // Check to make sure a value was recognized.
        if (promptContext.recognized.succeeded) {
            const value = promptContext.recognized.value;
            try {
                if (value < 6) {
                    throw new Error('Party size too small.');
                } else if (value > 20) {
                    throw new Error('Party size too big.')
                } else {
                    return true; // Indicate that this is a valid value.
                }
            } catch (err) {
                await promptContext.context.sendActivity(`${err.message} <br/>Please provide a valid number between 6 and 20.`);
                return false; // Indicate that this is invalid.
            }
        } else {
            return false;
        }
    }))
    .add(new WaterfallDialog(MAIN_DIALOG, [
        async (step) => {
            // Ask the user for their party size.
            return await step.prompt('partySizePrompt', {
                prompt: 'How large is your party?',
                retryPrompt: 'Sorry, please specify a size between 6 and 20.'
            });
        },
        async (step) => {
            // Acknowledge their response and exit the dialog.
            const size = step.result;
            await step.context.sendActivity(`That's a party of ${size}, thanks.`);
            return await step.endDialog();
        }
    ]));
```

---

De même, si vous souhaitez valider une réponse **DatetimePrompt** pour une date et une heure ultérieures, vous pouvez utiliser une logique de validation similaire à ceci :

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```cs
    private Task<bool> DateTimeValidatorAsync(PromptValidatorContext<IList<DateTimeResolution>> prompt, CancellationToken cancellationToken)
    {
        if (prompt.Recognized.Succeeded)
        {
            var resolution = prompt.Recognized.Value.First();

            // Verify that the Timex received is within the desired bounds, compared to today.
            var now = DateTime.Now;
            DateTime.TryParse(resolution.Value, out var time);

            if (time < now)
            {
                return Task.FromResult(false);
            }

            return Task.FromResult(true);
        }

        return Task.FromResult(false);
    }
```

```csharp
_dialogs.Add(new DateTimePrompt("date", DateTimeValidatorAsync));
```

Vous trouverez d’autres exemples dans notre [référentiel d’exemples](https://aka.ms/bot-samples-readme).

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
const { DateTimePrompt } = require("botbuilder-dialogs");
```

```JavaScript
// Create the dialog set, and add the prompt and the waterfall dialog.
this.dialogs = new DialogSet(this.dialogState)
    .add(new DateTimePrompt('dateTimePrompt', async (promptContext) => {
        try {
            if (!promptContext.recognized.succeeded) { throw new Error('Value not recognized.') }
            const values = promptContext.recognized.value;
            if (!Array.isArray(values) || values.length < 0) { throw new Error('Value missing.'); }
            if ((values[0].type !== 'datetime') && (values[0].type !== 'date')) { throw new Error('Unsupported type.'); }
            const now = new Date();
            const value = new Date(values[0].value);
            if (value.getTime() < now.getTime()) { throw new Error('Value in the past.') }

            // update the return value of the prompt to be a real date object
            promptContext.recognized.value = [value];
            return true; // indicate valid
        } catch (err) {
            await promptContext.context.sendActivity(`${err} Please specify a date or a date and time in the future, like tomorrow at 9am.`);
            return false; // indicate invalid
        }
    }))
    .add(new WaterfallDialog(MAIN_DIALOG, [
        async (step) => {
            // Ask the user for their party size.
            return await step.prompt('dateTimePrompt', 'When would you like to schedule that for?');
        },
        async (step) => {
            // Acknowledge their response and exit the dialog.
            const time = step.result;
            await step.context.sendActivity(`That's ${time}, thanks.`);
            return await step.endDialog();
        }
    ]));
```

Vous trouverez d’autres exemples dans notre [référentiel d’exemples](https://aka.ms/bot-samples-readme).

---

> [!TIP]
> Les invites de date/heure peuvent être résolues avec quelques dates différentes si l’utilisateur fournit une réponse ambiguë. Selon l’utilisation que vous comptez en faire, vous pouvez sélectionner toutes les résolutions fournies par le résultat de l’invite, au lieu de choisir simplement la première option.

Vous pouvez utiliser des techniques similaires pour valider les réponses à tout type d’invite.

## <a name="save-user-data"></a>Enregistrer les données utilisateur

Lorsque vous invitez l’utilisateur à effectuer une saisie, vous pouvez gérer cette saisie de plusieurs façons. Par exemple, vous pouvez utiliser et ignorer l’entrée, l’enregistrer dans une variable globale, dans un conteneur de stockage volatile ou en mémoire, dans un fichier, ou dans une base de données externe. Pour plus d’informations sur la façon d’enregistrer les données utilisateur, consultez [Gérer les données utilisateur](bot-builder-howto-v4-state.md).

## <a name="additional-resources"></a>Ressources supplémentaires

Pour obtenir un exemple complet illustrant quelques-unes de ces invites, étudiez le bot conversationnel à tours multiples pour [C#](https://aka.ms/cs-multi-prompts-sample) ou [JavaScript](https://aka.ms/js-multi-prompts-sample).

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous savez comment inviter un utilisateur à effectuer une saisie, améliorons l’expérience utilisateur et le code du bot en gérant les différents flux de conversation via des dialogues.

> [!div class="nextstepaction"]
> [Gérer un flux de conversation simple avec des dialogues](bot-builder-dialog-manage-conversation-flow.md)
