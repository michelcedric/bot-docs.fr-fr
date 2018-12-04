---
title: Implémenter des flux de conversation séquentiels | Microsoft Docs
description: Découvrez comment gérer un flux de conversation simple avec des dialogues dans le Kit de développement logiciel (SDK) Bot Builder pour Node.js.
keywords: flux de conversation simple, flux de conversation séquentiel, dialogues, invites, cascades, ensemble de dialogues
author: JonathanFingold
ms.author: v-jofing
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 11/18/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 2264b6927ccb863f153f2feb829cc0fb99c711f7
ms.sourcegitcommit: 6c719b51c9e4e84f5642100a33fe346b21360e8a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/28/2018
ms.locfileid: "52452081"
---
# <a name="implement-sequential-conversation-flow"></a>Implémenter des flux de conversation séquentiels

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Vous pouvez gérer des flux de conversation simples et complexes avec la bibliothèque de dialogues. Dans une interaction simple, le bot s’exécute via une séquence fixe d’étapes, et la conversation se termine. Dans cet article, nous utilisons un _dialogue en cascade_, quelques _invites_ et un _ensemble de dialogues_ pour créer une interaction simple qui pose à l’utilisateur une série de questions.

## <a name="prerequisites"></a>Prérequis
- [Bot Framework Emulator](https://github.com/Microsoft/BotFramework-Emulator/blob/master/README.md#download)
- Le code dans cet article est basé sur l’exemple d’**invite à plusieurs tours**. Vous aurez besoin d’une copie de l’exemple en [C# ](https://aka.ms/cs-multi-prompts-sample) ou en [JS](https://aka.ms/js-multi-prompts-sample).
- Connaissances des [concepts de base des bots](bot-builder-basics.md), de la [bibliothèque de boîtes de dialogue](bot-builder-concept-dialog.md), de [l’état de boîte de dialogue](bot-builder-dialog-state.md), et du fichier [.bot](bot-file-basics.md).


Les sections suivantes reflètent les étapes que vous devez suivre pour implémenter des dialogues simples pour la plupart des bots :

## <a name="configure-your-bot"></a>Configurer votre bot

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Nous allons initialiser l’accesseur de propriété d’état pour l’état du dialogue du bot dans le code de configuration dans le fichier **Startup.cs**.

Nous définissons une classe `MultiTurnPromptsBotAccessors` pour héberger les objets de gestion d’états et les accesseurs de propriété d’état pour le bot.
Ici, nous appelons uniquement des parties du code.

```csharp
public class MultiTurnPromptsBotAccessors
{
    // Initializes a new instance of the class.
    public MultiTurnPromptsBotAccessors(ConversationState conversationState, UserState userState)
    {
        ConversationState = conversationState ?? throw new ArgumentNullException(nameof(conversationState));
        UserState = userState ?? throw new ArgumentNullException(nameof(userState));
    }

    public IStatePropertyAccessor<DialogState> ConversationDialogState { get; set; }
    public IStatePropertyAccessor<UserProfile> UserProfile { get; set; }

    public ConversationState ConversationState { get; }
    public UserState UserState { get; }
}
```

Nous inscrivons la classe d’accesseurs dans la méthode `ConfigureServices` de la classe `Statup`.
Une fois encore, nous appelons uniquement des parties du code.

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // ...

    // Create and register state accessors.
    // Accessors created here are passed into the IBot-derived class on every turn.
    services.AddSingleton<MultiTurnPromptsBotAccessors>(sp =>
    {
        // We need to grab the conversationState we added on the options in the previous step
        var options = sp.GetRequiredService<IOptions<BotFrameworkOptions>>().Value;
        var conversationState = options.State.OfType<ConversationState>().FirstOrDefault();
        var userState = options.State.OfType<UserState>().FirstOrDefault();

        // Create the custom state accessor.
        // State accessors enable other components to read and write individual properties of state.
        var accessors = new MultiTurnPromptsBotAccessors(conversationState, userState)
        {
            ConversationDialogState = conversationState.CreateProperty<DialogState>("DialogState"),
            UserProfile = userState.CreateProperty<UserProfile>("UserProfile"),
        };

        return accessors;
    });
}
```

Via l’injection de dépendances, les accesseurs sont disponibles pour le code de constructeur du bot.

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Dans le fichier **index.js**, nous définissons les objets de gestion d’état.
Ici, nous appelons uniquement des parties du code.

```javascript
// Import required bot services. See https://aka.ms/bot-services to learn more about the different part of a bot.
const { BotFrameworkAdapter, ConversationState, MemoryStorage, UserState } = require('botbuilder');

// Define the state store for your bot. See https://aka.ms/about-bot-state to learn more about using MemoryStorage.
// A bot requires a state storage system to persist the dialog and user state between messages.
const memoryStorage = new MemoryStorage();

// Create conversation state with in-memory storage provider.
const conversationState = new ConversationState(memoryStorage);
const userState = new UserState(memoryStorage);

// Create the main dialog, which serves as the bot's main handler.
const bot = new MultiTurnBot(conversationState, userState);
```

Le constructeur du bot va créer les accesseurs de propriété d’état pour le bot : `this.dialogState` et `this.userProfile`.

---

## <a name="update-the-bot-turn-handler-to-call-the-dialog"></a>Mettre à jour le gestionnaire de tours du bot pour qu’il appelle le dialogue

Pour exécuter le dialogue, le gestionnaire de tours du bot doit créer un contexte de dialogue pour l’ensemble de dialogues qui contient les dialogues du bot. Un bot peut définir plusieurs ensembles de boîtes de dialogue, mais en règle générale, vous devez simplement en définir un pour votre bot. 

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Le dialogue est exécuté depuis le gestionnaire de tours du bot. Le gestionnaire commence par créer un élément `DialogContext` et continue le dialogue actif ou débute un nouveau dialogue selon ce qui est nécessaire. Le gestionnaire enregistre ensuite la conversation et l’état de l’utilisateur à la fin du tour.

Dans la classe `MultiTurnPromptsBot`, nous avons défini une propriété `_dialogs` qui contient l’ensemble de dialogues à partir duquel nous générons un contexte de dialogue. Ici encore, nous affichons uniquement une partie du code du gestionnaire de tours.

```csharp
public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
{
    // ...
    if (turnContext.Activity.Type == ActivityTypes.Message)
    {
        // Run the DialogSet - let the framework identify the current state of the dialog from
        // the dialog stack and figure out what (if any) is the active dialog.
        var dialogContext = await _dialogs.CreateContextAsync(turnContext, cancellationToken);
        var results = await dialogContext.ContinueDialogAsync(cancellationToken);

        // If the DialogTurnStatus is Empty we should start a new dialog.
        if (results.Status == DialogTurnStatus.Empty)
        {
            await dialogContext.BeginDialogAsync("details", null, cancellationToken);
        }
    }

    // ...
    // Save the dialog state into the conversation state.
    await _accessors.ConversationState.SaveChangesAsync(turnContext, false, cancellationToken);

    // Save the user profile updates into the user state.
    await _accessors.UserState.SaveChangesAsync(turnContext, false, cancellationToken);
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Le code du bot utilise quelques-unes des classes dans la bibliothèque des dialogues.

```javascript
const { ChoicePrompt, DialogSet, NumberPrompt, TextPrompt, WaterfallDialog } = require('botbuilder-dialogs');
```

Le dialogue est exécuté depuis le gestionnaire de tours du bot. Le gestionnaire commence par créer un élément `DialogContext` (`dc`) et continue le dialogue actif ou débute un nouveau dialogue selon ce qui est nécessaire. Le gestionnaire enregistre ensuite la conversation et l’état de l’utilisateur à la fin du tour.

La classe `MultiTurnBot` est définie dans le fichier **bot.js**. Le constructeur de cette classe ajoute une propriété `dialogs` pour l’ensemble de dialogues à partir duquel nous générons un contexte de dialogue. Ce bot collecte les données utilisateur une seule fois, à l’aide du dialogue `WHO_ARE_YOU`. Une fois que le profil utilisateur est rempli, le bot utilise le dialogue `HELLO_USER` pour répondre. Ici encore, nous affichons uniquement une partie du code du gestionnaire de tours.

```javascript
async onTurn(turnContext) {
    if (turnContext.activity.type === ActivityTypes.Message) {
        // Create a dialog context object.
        const dc = await this.dialogs.createContext(turnContext);

        const utterance = (turnContext.activity.text || '').trim().toLowerCase();

        // ...
        // If the bot has not yet responded, continue processing the current dialog.
        await dc.continueDialog();

        // Start the sample dialog in response to any other input.
        if (!turnContext.responded) {
            const user = await this.userProfile.get(dc.context, {});
            if (user.name) {
                await dc.beginDialog(HELLO_USER);
            } else {
                await dc.beginDialog(WHO_ARE_YOU);
            }
        }
    }

    // ...
    // Save changes to the user state.
    await this.userState.saveChanges(turnContext);

    // End this turn by saving changes to the conversation state.
    await this.conversationState.saveChanges(turnContext);
}
```

---

Dans le gestionnaire de tours du bot, nous créons un contexte de dialogue pour l’ensemble de dialogues. Le contexte de dialogue accède au cache d’état du bot, qui se rappelle de l’endroit de la conversation où le dernier tour a été laissé.

S’il existe un dialogue actif, la méthode _continuer le dialogue_ du contexte de dialogue le fait avancer en utilisant l’entrée de l’utilisateur qui a déclenché ce tour ; sinon, le bot appelle la méthode _commencer le dialogue_ du contexte de dialogue pour initier un dialogue.

Enfin, nous appelons la méthode _enregistrer les modifications_ sur les objets de gestion d’états pour conserver les modifications apportées lors de ce tour.

### <a name="about-dialog-and-bot-state"></a>Au sujet du dialogue et de l’état du bot

Dans ce bot, nous avons défini deux accesseurs de propriété d’état :

* Un créé dans l’état de conversation pour la propriété d’état de dialogue. L’état de dialogue suit l’endroit où se trouve l’utilisateur dans les dialogues d’un ensemble de dialogues, et il est mis à jour par le contexte de dialogue, de la même façon que nous appelons les méthodes de début de dialogue ou de suite de dialogue.
* Un créé dans l’état de l’utilisateur de la propriété de profil utilisateur. Le bot utilise cet élément pour suivre les informations qu’il possède sur l’utilisateur, et nous gérons explicitement cet état dans le code de notre bot.

Les méthodes _obtenir_ et _définir_ d’un accesseur de propriété d’état obtiennent et définissent la valeur de la propriété dans le cache de l’objet de gestion d’états. Le cache est rempli la première fois que la valeur d’une propriété d’état est demandée dans un tour, mais il doit être conservé explicitement. Afin de conserver les modifications apportées à ces propriétés d’état, nous appelons la méthode _enregistrer les modifications_ de l’objet de gestion d’états correspondant.

## <a name="initialize-your-bot-and-define-your-dialog"></a>Initialiser votre bot et définir votre dialogue

Notre conversation simple est modélisée sous la forme d’une série de questions posées à l’utilisateur. Les versions C# et JavaScript ont des étapes légèrement différentes :

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

1. Demandez-lui comment il s’appelle.
1. Demandez-lui s’il souhaite communiquer son âge.
1. Si c’est le cas, demandez-lui son âge. Sinon, ignorez cette étape.
1. Demandez si les informations collectées sont correctes.
1. Envoyez un message d’état et terminez.

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Pour le dialogue `who_are_you` :

1. Demandez-lui comment il s’appelle.
1. Demandez-lui s’il souhaite communiquer son âge.
1. Si c’est le cas, demandez-lui son âge. Sinon, ignorez cette étape.
1. Envoyez un message d’état et terminez.

Pour le dialogue `hello_user` :

1. Affichez les informations utilisateur que le bot a collectées.

---

Voici quelques points à retenir quand vous définissez vos propres étapes en cascade.

* Chaque tour de bot reflète une entrée de l’utilisateur suivie d’une réponse du bot. Par conséquent, vous demandez une entrée à l’utilisateur à la fin d’une étape en cascade et vous demandez sa réponse dans l’étape en cascade suivante.
* Chaque invite est effectivement un dialogue en deux étapes qui présente son invite et effectue une boucle jusqu’à ce qu’il reçoive une entrée « valide ». 

Dans cet exemple, le dialogue est défini dans le fichier de bot et initialisé dans le constructeur du bot.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Définissez une propriété d’instance pour l’ensemble de dialogues.

```csharp
// The DialogSet that contains all the Dialogs that can be used at runtime.
private DialogSet _dialogs;
```

Créez l’ensemble de dialogues au sein du constructeur du bot en ajoutant les invites et le dialogue en cascade à l’ensemble.

```csharp
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

Dans cet exemple, nous définissons chacune des étapes en tant que méthode distincte. Vous pouvez également définir les étapes en ligne dans le constructeur via des expressions lambda.

```csharp
private static async Task<DialogTurnResult> NameStepAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
{
    // WaterfallStep always finishes with the end of the Waterfall or with another dialog; here it is a Prompt Dialog.
    // Running a prompt here means the next WaterfallStep will be run when the users response is received.
    return await stepContext.PromptAsync("name", new PromptOptions { Prompt = MessageFactory.Text("Please enter your name.") }, cancellationToken);
}

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

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Dans cet exemple, le dialogue en cascade est défini au sein du fichier **bot.js**.

Définissez les identificateurs à utiliser pour les accesseurs de propriété d’état, les invites et les dialogues.

```javascript
const DIALOG_STATE_PROPERTY = 'dialogState';
const USER_PROFILE_PROPERTY = 'user';

const WHO_ARE_YOU = 'who_are_you';
const HELLO_USER = 'hello_user';

const NAME_PROMPT = 'name_prompt';
const CONFIRM_PROMPT = 'confirm_prompt';
const AGE_PROMPT = 'age_prompt';
```

Définissez et créez l’ensemble de dialogues au sein du constructeur du bot en ajoutant les invites et les dialogues en cascade à l’ensemble.
L’élément `NumberPrompt` inclut une validation personnalisée pour vérifier que l’utilisateur entre un âge supérieur à 0.

```javascript
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

Puisque nos méthodes d’étape de dialogue font référence aux propriétés d’instance, nous devons utiliser la méthode `bind` de façon que l’objet `this` se résolve correctement dans chaque méthode d’étape.

Dans cet exemple, nous définissons chacune des étapes en tant que méthode distincte. Vous pouvez également définir les étapes en ligne dans le constructeur via des expressions lambda.

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

---

Cet exemple met à jour l’état du profil utilisateur dans le dialogue. Cette pratique peut fonctionner pour un bot simple, mais elle ne fonctionnera pas si vous souhaitez réutiliser un dialogue pour des bots.

Il existe différentes options pour séparer les étapes de dialogue de l’état du bot. Par exemple, une fois que votre dialogue a collecté des informations complètes, vous pouvez effectuer les actions suivantes :

* Utiliser la méthode _terminer un dialogue_ pour fournir les données collectées en tant que valeurs renvoyées au contexte parent. Il peut s’agir du gestionnaire de tours du bot ou d’un dialogue précédemment actif sur la pile du dialogue. C’est de cette façon que les classes d’invite sont conçues.
* Générer une demande vers un service approprié. Cela peut fonctionner correctement si votre bot agit en tant que serveur frontal vers un service plus volumineux.

## <a name="test-your-dialog"></a>Tester votre dialogue

Générez et exécutez votre bot en local, puis interagissez avec votre bot à l’aide de l’émulateur.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

1. Le bot envoie un message d’accueil initial en réponse à l’activité de mise à jour de conversation lors de laquelle l’utilisateur est ajouté à la conversation.
1. Entrez `hi` ou autre chose. Dans la mesure où il n’y a pas encore de dialogue actif dans ce tour, le bot commence le dialogue `details`.
   * Le bot envoie la première invite du dialogue et attend d’autres entrées.
1. Répondez aux questions à mesure que le bot les pose, en avançant tout au long du dialogue.
1. La dernière étape du dialogue envoie un message `Thanks` en fonction de vos entrées.
   * Lorsque le dialogue se termine, il est supprimé de la pile du dialogue, et le bot n’a plus de dialogue actif.
1. Entrez `hi` ou autre chose pour débuter une nouvelle fois le dialogue.

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

1. Le bot envoie un message d’accueil initial en réponse à l’activité de mise à jour de conversation lors de laquelle l’utilisateur est ajouté à la conversation.
1. Entrez `hi` ou autre chose. Dans la mesure où il n’y a pas encore de dialogue actif dans ce tour, ni de profil utilisateur, le bot commence le dialogue `who_are_you`.
   * Le bot envoie la première invite du dialogue et attend d’autres entrées.
1. Répondez aux questions à mesure que le bot les pose, en avançant tout au long du dialogue.
1. La dernière étape du dialogue envoie un bref message de confirmation.
1. Entrez `hi` ou autre chose.
   * Le bot démarre le dialogue `hello_user` en une seule étape, qui affiche des informations à partir des données collectées et se termine immédiatement.

---

## <a name="additional-resources"></a>Ressources supplémentaires
Vous pouvez compter sur la validation intégrée de chaque type d’invite de commandes comme indiqué ici, ou vous pouvez ajouter votre validation personnalisée à l’invite. Pour plus d’informations, consultez [Collecter les entrées utilisateur avec une invite de dialogue](bot-builder-prompts.md).

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Créer des flux de conversation avancés à l’aide des branches et boucles](bot-builder-dialog-manage-complex-conversation-flow.md)
