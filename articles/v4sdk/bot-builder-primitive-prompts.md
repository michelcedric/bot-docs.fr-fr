---
redirect_url: /bot-framework/bot-builder-howto-v4-state
ms.openlocfilehash: 081c7c1f3e354d4352baea029840c8175152116e
ms.sourcegitcommit: a54a70106b9fdf278fd7270b25dd51c9bd454ab1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/08/2018
ms.locfileid: "51273116"
---
<a name="--"></a><!--
---
titre : créer vos propres invites pour collecter des entrées utilisateur | Microsoft Docs description : découvrez comment gérer un flux de conversation avec des invites primitives dans le Kit de développement logiciel (SDK) Bot Builder.
mots clés : conversation flow, prompts auteur : v-ducvo ms.auteur : v-ducvo responsable : kamrani ms.topic : article ms.service : bot-service ms.sous-service : sdk ms.date: 31/10/2018 monikerRange: « azure-bot-service-4.0 »
---

# <a name="create-your-own-prompts-to-gather-user-input"></a>Créer vos propres invites pour collecter des entrées utilisateur

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Une conversation entre un bot et un utilisateur implique souvent une demande (invite) d’informations à l’utilisateur, l’analyse de sa réponse et l’action en fonction des informations obtenues. La rubrique [Inviter les utilisateurs à saisir une entrée à l’aide de la bibliothèque Dialogs](bot-builder-prompts.md) détaille la marche à suivre pour demander une entrée aux utilisateurs avec la bibliothèque de **dialogues**. Entre autres choses, la bibliothèque de **dialogues** se charge de suivre la conversation actuelle ainsi que la question actuelle posée. Elle fournit aussi des méthodes pour demander et confirmer différents types d’informations telles que du texte, un nombre, une date, une heure, etc.

Dans certains cas, les **dialogues** intégrés peuvent ne pas constituer la solution pour votre bot ; les **dialogues** peuvent surcharger le bot, être trop rigides ou empêcher le bot d’accomplir ce que vous voulez. Dans ces cas, vous pouvez ignorer la bibliothèque et implémenter votre propre logique d’invite. Cette rubrique vous montrera comment configurer votre *bot Echo* de base afin que vous puissiez gérer une conversation avec vos propres invites.

## <a name="track-prompt-states"></a>Suivre les états d’invite

Dans une conversation guidée par invite, vous devez suivre où vous vous trouvez dans la conversation, et les questions qui y sont posées. Dans le code, cela se traduit par la gestion de deux balises.

Par exemple, vous pouvez créer deux propriétés que vous voulez suivre.

Ces états suivent les sujets et les invites dans lesquelles nous nous trouvons actuellement. Pour s’assurer que les balises fonctionnent comme prévu une fois déployées dans le cloud, nous les stockons dans l’[état de la conversation](bot-builder-howto-v4-state.md). 

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Nous créons deux classes pour suivre l’état. **TopicState** nous permet de suivre la progression des invites conversationnelles, et **UserProfile** de suivre les informations fournies par l’utilisateur. Nous stockerons ces informations dans l’[état](bot-builder-howto-v4-state.md) du bot dans une étape ultérieure.

```csharp
/// <summary>
/// Contains conversation state information about the conversation in progress.
/// </summary>
public class TopicState
{
    /// <summary>
    /// Identifies the current "topic" of conversation.
    /// </summary>
    public string Topic { get; set; }

    /// <summary>
    /// Indicates whether we asked the user a question last turn, and
    /// if so, what it was.
    /// </summary>
    public string Prompt { get; set; }
}
```

```csharp
/// <summary>
/// Contains user state information for the user's profile.
/// </summary>
public class UserProfile
{
    public string UserName { get; set; }

    public int? Age { get; set; }

    public string WorkPlace { get; set; }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Dans **index.js**, mettez à jour l’instruction nécessaire pour inclure `UserState`.

```javascript
const { BotFrameworkAdapter, MemoryStorage, ConversationState, UserState } = require('botbuilder');
```

Créez ensuite un objet de gestion d’état utilisateur et passez-le lors de la création de votre bot.

```javascript
// Create conversation and user state with in-memory storage provider.
const conversationState = new ConversationState(memoryStorage);
const userState = new UserState(memoryStorage);

// Create the bot.
const myBot = new MyBot(conversationState, userState);
```

Dans **bot.js**, définissez des identificateurs pour les accesseurs de propriété d’état que nous allons utiliser pour gérer l’[état](bot-builder-howto-v4-state.md) du bot. Définissez également les invites à utiliser pour les informations que nous souhaitons obtenir de l’utilisateur.

Ajoutez ce code en dehors de la classe `MyBot`.

```javascript
// Define identifiers for our state property accessors.
const TOPIC_STATE_PROPERTY = 'topicStateProperty';
const USER_PROFILE_PROPERTY = 'userProfileProperty';

// Define the prompts to use to ask for user profile information.
const fields = {
    userName: "What is your name?",
    age: "How old are you?",
    workPlace: "Where do you work?"
}
```

---

## <a name="manage-a-topic-of-conversation"></a>Gérer le sujet de conversation

Dans une conversation séquentielle, telle qu’une conversation où vous récoltez des informations de l’utilisateur, vous devez savoir quand l’utilisateur est entré dans la conversation et où il se trouve. Vous pouvez suivre ces étapes dans le gestionnaire de tour du bot en configurant et en vérifiant les balises d’invite définies ci-dessus, et agir en conséquence. L’exemple ci-dessous montre comment vous pouvez utiliser ces balises pour récolter les informations du profil utilisateur dans la conversation.

Le code du bot est présenté ici et étudié dans la section suivante.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Pour ASP.NET Core, nous devons commencer par configurer l’injection de bot et de dépendance.

Renommez le fichier **EchoWithCounterBot.cs** en **PrimitivePromptsBot.cs** et mettez également à jour le nom de classe. Cette classe conserve la logique du bot. Nous la mettrons à jour prochainement.

Renommez le fichier **EchoBotAccessors.cs** en **BotAccessors.cs** et mettez également à jour le nom de classe. Cette classe contient les objets de gestion d’état et les accesseurs de propriété d’état pour notre bot. Mettez à jour la définition de la manière suivante.

```csharp
using Microsoft.Bot.Builder;
using System;

/// <summary>
/// Contains the state and state property accessors for the primitive prompts bot.
/// </summary>
public class BotAccessors
{
    public const string TopicStateName = "PrimitivePrompts.TopicStateAccessor";

    public const string UserProfileName = "PrimitivePrompts.UserProfileAccessor";

    public ConversationState ConversationState { get; }

    public UserState UserState { get; }

    public IStatePropertyAccessor<TopicState> TopicStateAccessor { get; set; }

    public IStatePropertyAccessor<UserProfile> UserProfileAccessor { get; set; }

    public BotAccessors(ConversationState conversationState, UserState userState)
    {
        if (conversationState is null)
        {
            throw new ArgumentNullException(nameof(conversationState));
        }

        if (userState is null)
        {
            throw new ArgumentNullException(nameof(userState));
        }

        this.ConversationState = conversationState;
        this.UserState = userState;
    }
}
```

Mettez à jour la méthode `ConfigureServices` du fichier **Startup.cs**, à partir de l’endroit où vous avez configuré l’objet `IStorage`.

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddBot<PrimitivePromptsBot>(options =>
    {
        // ...

        // The Memory Storage used here is for local bot debugging only. When the bot
        // is restarted, everything stored in memory will be gone.
        IStorage dataStore = new MemoryStorage();

        var conversationState = new ConversationState(dataStore);
        options.State.Add(conversationState);

        var userState = new UserState(dataStore);
        options.State.Add(userState);
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
            TopicStateAccessor = conversationState.CreateProperty<TopicState>(BotAccessors.TopicStateName),
            UserProfileAccessor = userState.CreateProperty<UserProfile>(BotAccessors.UserProfileName),
        };

        return accessors;
    });
}
```

Enfin, mettez à jour la logique de bot dans le fichier **PrimitivePromptsBot.cs**.

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading;
using System.Threading.Tasks;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Schema;

public class PrimitivePromptsBot : IBot
{
    public const string ProfileTopic = "profile";

    /// <summary>
    /// Describes a field in the user profile.
    /// </summary>
    private class UserFieldInfo
    {
        /// <summary>
        /// The ID to use for this field.
        /// </summary>
        public string Key { get; set; }

        /// <summary>
        /// The prompt to use to ask for a value for this field.
        /// </summary>
        public string Prompt { get; set; }

        /// <summary>
        /// Gets the value of the corresponding field.
        /// </summary>
        public Func<UserProfile, string> GetValue { get; set; }

        /// <summary>
        /// Sets the value of the corresponding field.
        /// </summary>
        public Action<UserProfile, string> SetValue { get; set; }
    }

    /// <summary>
    /// The prompts for the user profile, indexed by field name.
    /// </summary>
    private static List<UserFieldInfo> UserFields { get; } = new List<UserFieldInfo>
    {
        new UserFieldInfo {
            Key = nameof(UserProfile.UserName),
            Prompt = "What is your name?",
            GetValue = (profile) => profile.UserName,
            SetValue = (profile, value) => profile.UserName = value,
        },
        new UserFieldInfo {
            Key = nameof(UserProfile.Age),
            Prompt = "How old are you?",
            GetValue = (profile) => profile.Age.HasValue? profile.Age.Value.ToString() : null,
            SetValue = (profile, value) =>
            {
                if (int.TryParse(value, out int age))
                {
                    profile.Age = age;
                }
            },
        },
        new UserFieldInfo {
            Key = nameof(UserProfile.WorkPlace),
            Prompt = "Where do you work?",
            GetValue = (profile) => profile.WorkPlace,
            SetValue = (profile, value) => profile.WorkPlace = value,
        },
    };

    /// <summary>
    /// The state and state accessors for the bot.
    /// </summary>
    private BotAccessors Accessors { get; }

    public PrimitivePromptsBot(BotAccessors accessors)
    {
        Accessors = accessors ?? throw new ArgumentNullException(nameof(accessors));
    }

    public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
    {
        if (turnContext.Activity.Type is ActivityTypes.Message)
        {
            // Use the state property accessors to get the topic state and user profile.
            TopicState topicState = await Accessors.TopicStateAccessor.GetAsync(
                turnContext,
                () => new TopicState { Topic = ProfileTopic, Prompt = null },
                cancellationToken);
            UserProfile userProfile = await Accessors.UserProfileAccessor.GetAsync(
                turnContext,
                () => new UserProfile(),
                cancellationToken);

            // Check whether we need more information.
            if (topicState.Topic is ProfileTopic)
            {
                // If we're expecting input, record it in the user's profile.
                if (topicState.Prompt != null)
                {
                    UserFieldInfo field = UserFields.First(f => f.Key.Equals(topicState.Prompt));
                    field.SetValue(userProfile, turnContext.Activity.Text.Trim());
                }

                // Determine which fields are not yet set.
                List<UserFieldInfo> emptyFields = UserFields.Where(f => f.GetValue(userProfile) is null).ToList();

                if (emptyFields.Any())
                {
                    // If all the fields are empty, send a welcome message.
                    if (emptyFields.Count == UserFields.Count)
                    {
                        await turnContext.SendActivityAsync("Welcome new user, please fill out your profile information.");
                    }

                    // We have at least one empty field. Prompt for the next empty field,
                    // and update the prompt flag to indicate which prompt we just sent,
                    // so that the response can be captured at the beginning of the next turn.
                    UserFieldInfo field = emptyFields.First();
                    await turnContext.SendActivityAsync(field.Prompt);
                    topicState.Prompt = field.Key;
                }
                else
                {
                    // Our user profile is complete!
                    await turnContext.SendActivityAsync($"Thank you, {userProfile.UserName}. Your profile is complete.");
                    topicState.Prompt = null;
                    topicState.Topic = null;
                }
            }
            else if (turnContext.Activity.Text.Trim().Equals("hi", StringComparison.InvariantCultureIgnoreCase))
            {
                await turnContext.SendActivityAsync($"Hi. {userProfile.UserName}.");
            }
            else
            {
                await turnContext.SendActivityAsync("Hi. I'm the Contoso cafe bot.");
            }

            // Use the state property accessors to update the topic state and user profile.
            await Accessors.TopicStateAccessor.SetAsync(turnContext, topicState, cancellationToken);
            await Accessors.UserProfileAccessor.SetAsync(turnContext, userProfile, cancellationToken);

            // Save any state changes to storage.
            await Accessors.ConversationState.SaveChangesAsync(turnContext, false, cancellationToken);
            await Accessors.UserState.SaveChangesAsync(turnContext, false, cancellationToken);
        }
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Dans **bot.js**, mettez à jour la définition de classe `MyBot`.

Nous préparons les accesseurs de propriété d’état dans les constructeurs `topicStateAccessor` et `userProfileAccessor` du bot. L’état du sujet effectue le suivi du sujet de la conversation et le profil utilisateur suit les informations que nous avons collectées pour l’utilisateur.

```javascript
constructor(conversationState, userState) {
    // Create state property accessors.
    this.topicStateAccessor = conversationState.createProperty(TOPIC_STATE_PROPERTY);
    this.userProfileAccessor = userState.createProperty(USER_PROFILE_PROPERTY);

    // Track the conversation and user state management objects.
    this.conversationState = conversationState;
    this.userState = userState;
}
```

Mettez ensuite à jour le gestionnaire de tour pour utiliser l’état du bot pour contrôler le flux de conversation et enregistrer les informations utilisateur collectées.

```javascript
async onTurn(turnContext) {
    // Handle only message activities from the user.
    if (turnContext.activity.type === ActivityTypes.Message) {
        // Get state properties using their accessors, providing default values.
        const topicState = await this.topicStateAccessor.get(turnContext, {
            prompt: undefined,
            topic: 'profile'
        });
        const userProfile = await this.userProfileAccessor.get(turnContext, {
            "userName": undefined,
            "age": undefined,
            "workPlace": undefined
        });

        if (topicState.topic === 'profile') {
            // If a prompt flag is set in the conversation state, use it to capture the incoming value
            // into the appropriate field of the user profile.
            if (topicState.prompt) {
                userProfile[topicState.prompt] = turnContext.activity.text;
            }

            // Determine which fields are not yet set.
            const empty_fields = [];
            Object.keys(fields).forEach(function (key) {
                if (!userProfile[key]) {
                    empty_fields.push({
                        key: key,
                        prompt: fields[key]
                    });
                }
            });

            if (empty_fields.length) {

                // If all the fields are empty, send a welcome message.
                if (empty_fields.length == Object.keys(fields).length) {
                    await turnContext.sendActivity('Welcome new user, please fill out your profile information.');
                }

                // We have at least one empty field. Prompt for the next empty field.
                await turnContext.sendActivity(empty_fields[0].prompt);

                // update the flag to indicate which prompt we just sent
                // so that the response can be captured at the beginning of the next turn.
                topicState.prompt = empty_fields[0].key;

            } else {
                // Our user profile is complete!
                await turnContext.sendActivity('Thank you. Your profile is complete.');
                topicState.prompt = null;
                topicState.topic = null;

            }
        } else if (turnContext.activity.text && turnContext.activity.text.match(/hi/ig)) {
            // Check to see if the user said "hi" and respond with a greeting
            await turnContext.sendActivity(`Hi ${userProfile.userName}.`);
        } else {
            // Default message
            await turnContext.sendActivity("Hi. I'm the Contoso bot.");
        }

        // Save state changes
        await this.conversationState.saveChanges(turnContext);
        await this.userState.saveChanges(turnContext);
    }
}
```

---

L’exemple de code ci-dessus initialise l’indicateur de _sujet_ sur `profile` afin de démarrer la conversation de collecte du profil. Le bot avance dans la conversation en mettant à jour l’indicateur d’_invite_ sur une valeur qui représente la question qui est posée actuellement. Lorsque cet indicateur est défini sur la valeur appropriée, le bot sait ce qu’il doit faire avec le prochain message que lui envoie l’utilisateur et comment le traiter.

Enfin, les indicateurs sont réinitialisés lorsque le bot a terminé sa collecte d’informations. Dans le cas contraire, votre bot est bloqué dans une boucle et ne cesse de poser la dernière question.

Vous pouvez étendre ce modèle pour gérer des flux de conversation plus complexes en définissant simplement d’autres indicateurs ou en créant une branche pour la conversation selon l’entrée de l’utilisateur.

## <a name="manage-multiple-topics-of-conversations"></a>Gérer plusieurs sujets de conversations

Une fois que vous êtes en mesure de gérer un sujet de conversation, vous pouvez étendre la logique du bot afin qu’il puisse gérer plusieurs sujets de conversations. Plusieurs sujets peuvent être gérés grâce à la vérification des conditions supplémentaires, puis en choisissant le chemin approprié.

Vous pouvez étendre l’exemple ci-dessus pour prendre en charge d’autres fonctionnalités et sujets de conversation, comme la réservation d’une table ou la commande d’articles. Au besoin, ajoutez des indicateurs supplémentaires à l’état du sujet pour effectuer le suivi de la conversation.

Vous pouvez aussi trouver utile de diviser le code en fonctions ou méthodes indépendantes, facilitant ainsi le suivi du flux de conversation. Voici un modèle employé couramment : le bot évalue le message et l’état, puis il délègue le contrôle aux fonctions qui implémentent les détails de la fonctionnalité.

Pour aider vos utilisateurs à naviguer entre plusieurs sujets de conversation, pensez à fournir un menu principal. Par exemple, les [actions suggérées](bot-builder-howto-add-suggested-actions.md) vous permettent de proposer à l’utilisateur plusieurs choix de sujets à engager, au lieu de deviner ce que le bot peut faire.

## <a name="validate-user-input"></a>Valider les entrées utilisateur

La bibliothèque de **dialogues** fournit des méthodes intégrées pour valider l’entrée de l’utilisateur, mais nous pouvons aussi le faire avec nos propres invites. Par exemple, si nous demandons l’âge de l’utilisateur, nous devons être sûrs d’obtenir un nombre, et non une réponse du genre « Bob ».

L’analyse d’un nombre ou d’une date et heure est une tâche complexe qui dépasse l’objectif de cette rubrique. Heureusement, il existe une bibliothèque que nous pouvons exploiter. Pour analyser ces informations, nous utilisons la bibliothèque de [reconnaissance de texte Microsoft](https://github.com/Microsoft/Recognizers-Text). Ce package est disponible via NuGet et npm. Vous pouvez également le télécharger directement à partir du référentiel. (Il est également inclus dans la bibliothèque de **dialogues**. Cette information est importante, même si nous ne l’utiliserons pas ici.)

Cette bibliothèque est particulièrement utile pour analyser des entrées complexes telles que des dates, des heures ou des numéros de téléphone. Dans cet exemple, nous validons le nombre de participants pour une réservation, mais nous pourrions appliquer cette idée à des opérations de validation bien plus complexes.

Dans cet exemple, nous présentons seulement l’utilisation de `RecognizeNumber`. Vous pouvez trouver des informations sur l’utilisation d’autres méthodes de reconnaissance de la bibliothèque dans cette [documentation de référentiel](https://github.com/Microsoft/Recognizers-Text/blob/master/README.md).

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Pour utiliser la bibliothèque **Microsoft.Recognizers.Text.Number**, intégrez le package NuGet et ajoutez-ces instructions using à votre fichier bot.

```csharp
using System.Linq;
using Microsoft.Recognizers.Text;
using Microsoft.Recognizers.Text.Number;
```

Nous pourrions gérer la validation de nombreuses façons. Ici, nous allons mettre à jour notre classe de programmes d’assistance pour y inclure la validation.

Ajoutez les membres suivants à la classe `UserFieldInfo` interne du bot.

```csharp
/// <summary>Delegate for validating input.</summary>
/// <param name="turnContext">The current turn context. turnContext.Activity.Text contains the input to validate.</param>
/// <returns><code>true</code> if the input is valid; otherwise, <code>false</code>.</returns>
public delegate Task<bool> ValidatorDelegate(
    ITurnContext turnContext,
    CancellationToken cancellationToken = default(CancellationToken));

/// <summary>By default, evaluate all input as valid.</summary>
private static readonly ValidatorDelegate NoValidator =
    async (ITurnContext turnContext, CancellationToken cancellationToken) => true;

/// <summary>Gets or sets the validation function to use.</summary>
public ValidatorDelegate ValidateInput { get; set; } = NoValidator;
```

Ensuite, mettez à jour l’entrée _âge_ dans le `UserFields` du bot pour définir la validation à utiliser.
Étant donné que nous vérifions l’entrée avant de définir la valeur d’âge, nous pouvons simplifier un peu la fonction `SetValue` et tirer parti de la bibliothèque de modules de reconnaissance de texte.

```csharp
private static List<UserFieldInfo> UserFields { get; } = new List<UserFieldInfo>
{
    // ...
    new UserFieldInfo {
        Key = nameof(UserProfile.Age),
        Prompt = "How old are you?",
        GetValue = (profile) => profile.Age.HasValue? profile.Age.Value.ToString() : null,
        SetValue = (profile, value) =>
        {
            // As long as the input validates, this should work.
            List<ModelResult> result = NumberRecognizer.RecognizeNumber(value, Culture.English);
            profile.Age = int.Parse(result.First().Text);
        },
        ValidateInput = async (turnContext, cancellationToken) =>
        {
            try
            {
                // Recognize the input as a number. This works for responses such as
                // "twelve" as well as "12".
                List<ModelResult> result = NumberRecognizer.RecognizeNumber(
                    turnContext.Activity.Text, Culture.English);

                // Attempt to convert the Recognizer result to an integer
                int.TryParse(result.First().Text, out int age);

                if (age < 18)
                {
                    await turnContext.SendActivityAsync(
                        "You must be 18 or older.",
                        cancellationToken: cancellationToken);
                    return false;
                }
                else if (age > 120)
                {
                    await turnContext.SendActivityAsync(
                        "You must be 120 or younger.",
                        cancellationToken: cancellationToken);
                    return false;
                }
            }
            catch
            {
                await turnContext.SendActivityAsync(
                    "I couldn't understand your input. Please enter your age in years.",
                    cancellationToken: cancellationToken);
                return false;
            }

            // If we got through this, the number is valid.
            return true;
        },
    },
    // ...
};
```

Enfin, nous mettons à jour notre gestionnaire de tour pour valider toutes les entrées avant d’enregistrer une valeur dans la propriété.
La validation fait défaut à notre fonction NoValidator, qui accepte n’importe quelle entrée. Par conséquent, le comportement de l’invite d’âge est le seul qui changera. Si la validation de l’entrée échoue, nous ne définissons pas le champ et le bot demandera de nouveau une entrée pour ce champ la prochaine fois.

Ici, nous examinons la partie du gestionnaire de tour que nous devons mettre à jour.

```csharp
public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
{
    if (turnContext.Activity.Type is ActivityTypes.Message)
    {
        // ...
        // Check whether we need more information.
        if (topicState.Topic is ProfileTopic)
        {
            // If we're expecting input, record it in the user's profile.
            if (topicState.Prompt != null)
            {
                UserFieldInfo field = UserFields.First(f => f.Key.Equals(topicState.Prompt));
                if (await field.ValidateInput(turnContext, cancellationToken))
                {
                    field.SetValue(userProfile, turnContext.Activity.Text.Trim());
                }
            }

            // ...
        }
        //...
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Pour utiliser la bibliothèque **Recognizers**, ajoutez le package et exigez-le dans le code de votre bot (dans **bot.js**) :

```bash
npm i @microsoft/recognizers-text-suite --save
```

```javascript
// Required packages for this bot.
const Recognizers = require('@microsoft/recognizers-text-suite');
```

Mettez ensuite à jour les métadonnées `fields` pour y inclure la reconnaissance de texte et le code de validation :

```javascript
// Define the prompts to use to ask for user profile information.
const fields = {
    userName: { prompt: "What is your name?" },
    age: {
        prompt: "How old are you?",
        recognize: (turnContext) => {
            var result = Recognizers.recognizeNumber(
                turnContext.activity.text, Recognizers.Culture.English);
            return parseInt(result[0].resolution.value);
        },
        validate: async (turnContext) => {
            try {
                // Recognize the input as a number. This works for responses such as
                // "twelve" as well as "12".
                var result = Recognizers.recognizeNumber(
                    turnContext.activity.text, Recognizers.Culture.English);
                var age = parseInt(result[0].resolution.value);
                if (age < 18) {
                    await turnContext.sendActivity("You must be 18 or older.");
                    return false;
                }
                if (age > 120 ) {
                    await turnContext.sendActivity("You must be 120 or younger.");
                    return false;
                }
            } catch (_) {
                await turnContext.sendActivity(
                    "I couldn't understand your input. Please enter your age in years.");
                return false;
            }
            return true;
        }
    },
    workPlace: { prompt: "Where do you work?" }
}
```

Dans le gestionnaire de tour de notre bot, mettez à jour les blocs suivants, au sein desquels nous enregistrerons l’entrée de l’utilisateur et où nous sollicitons l’utilisateur. Nous devons mettre à jour ces sections pour prendre en compte les modifications apportées aux métadonnées du champ.

```javascript
async onTurn(turnContext) {
    // Handle only message activities from the user.
    if (turnContext.activity.type === ActivityTypes.Message) {
        // ...

        if (topicState.topic === 'profile') {
            // If a prompt flag is set in the conversation state, use it to capture the incoming value
            // into the appropriate field of the user profile.
            if (topicState.prompt) {
                const field = fields[topicState.prompt];
                // If the prompt has validation, check whether the input validates.
                if (!field.validate || await field.validate(turnContext)) {
                    // Set the field, using a recognizer if one is defined.
                    userProfile[topicState.prompt] = (field.recognize)
                        ? field.recognize(turnContext)
                        : turnContext.activity.text;
                }
            }

            // ...

            if (empty_fields.length) {

                // ...

                // We have at least one empty field. Prompt for the next empty field.
                await turnContext.sendActivity(empty_fields[0].prompt.prompt);

                // ...

            } // ...
        } // ...

        // Save state changes
        await this.conversationState.saveChanges(turnContext);
        await this.userState.saveChanges(turnContext);
    }
}
```

---

## <a name="next-step"></a>Étape suivante

Maintenant que vous savez comment gérer les états d’invite, voyons comment profiter de la bibliothèque de **dialogues** pour demander des entrées aux utilisateurs.

> [!div class="nextstepaction"]
> [Demander aux utilisateurs des entrées en utilisant la bibliothèque de dialogues](bot-builder-prompts.md)

-->
