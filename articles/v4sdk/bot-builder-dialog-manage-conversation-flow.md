---
title: Gérer un flux de conversation simple avec des dialogues | Microsoft Docs
description: Découvrez comment gérer un flux de conversation simple avec des dialogues dans le Kit de développement logiciel (SDK) Bot Builder pour Node.js.
keywords: flux de conversation simple, dialogues, invites, cascades, jeu de dialogues
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 8/2/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 77162601f542e6faa8908bc71abc971eb99fcc93
ms.sourcegitcommit: 1abc32353c20acd103e0383121db21b705e5eec3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/21/2018
ms.locfileid: "42756597"
---
# <a name="manage-simple-conversation-flow-with-dialogs"></a>Gérer un flux de conversation simple avec des dialogues

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Vous pouvez gérer des flux de conversation simples et complexes avec la bibliothèque de dialogues. Dans un flux de conversation simple, l’utilisateur commence à la première étape d’une *cascade*, puis continue jusqu’à la dernière étape où se termine l’échange conversationnel. Les dialogues peuvent également gérer des [flux de conversation complexes](~/v4sdk/bot-builder-dialog-manage-complex-conversation-flow.md), dans lesquels des parties du dialogue peuvent créer une branche et une boucle.

<!-- TODO: We need a dialogs conceptual topic to link to, so we can reference that here, in place of describing what they are and what their features are in a how-to topic. -->

<!-- TODO: This paragraph belongs in a conceptual topic. --> Un dialogue est comparable à une fonction dans un programme. Il est généralement conçu pour effectuer une opération spécifique, dans un ordre déterminé, et peut être appelé aussi souvent que nécessaire. L’utilisation de dialogues permet au développeur de bots de guider le flux conversationnel. Vous pouvez enchaîner plusieurs dialogues afin de gérer pratiquement tout flux de conversation que robot doit pouvoir gérer. La bibliothèque **Dialogs** du Kit de développement logiciel (SDK) Bot Builder comprend des fonctionnalités intégrées, telles que des _invites_ et des _dialogues en cascade_, conçues pour vous aider à gérer un flux de conversation. Vous pouvez utiliser les invites pour demander différents types d’informations aux utilisateurs. Vous pouvez recourir aux cascades pour combiner plusieurs étapes dans une séquence.

Dans cet article, nous allons utiliser des _jeux de dialogues_ pour créer un flux de conversation contenant à la fois des invites et des étapes en cascade. Nous disposons de deux exemples de dialogues. Le premier est un dialogue à une seule étape qui effectue une opération ne nécessitant aucune entrée utilisateur. Le second est un dialogue à plusieurs étapes qui invite l’utilisateur à fournir certaines informations.

## <a name="install-the-dialogs-library"></a>Installer la bibliothèque de dialogues

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Nous allons partir d’un modèle EchoBot de base. Pour obtenir des instructions, consultez le [démarrage rapide pour .NET](~/dotnet/bot-builder-dotnet-quickstart.md).

Pour utiliser des dialogues, vous devez installer le package NuGet `Microsoft.Bot.Builder.Dialogs` pour votre projet ou solution.
Référencez ensuite la bibliothèque de dialogues dans les instructions d’utilisation de vos fichiers de code.

```csharp
using Microsoft.Bot.Builder.Dialogs;
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Vous pouvez télécharger la bibliothèque `botbuilder-dialogs` à partir de NPM. Pour installer la bibliothèque `botbuilder-dialogs`, exécutez la commande NPM suivante :

```cmd
npm install --save botbuilder-dialogs@preview
```

Pour utiliser des **dialogues** dans votre bot, incluez le code ci-après dans votre fichier **app.js**.

```javascript
const botbuilder_dialogs = require('botbuilder-dialogs');
```

---

## <a name="create-a-dialog-stack"></a>Créer une pile de dialogues

Dans le cadre de ce premier exemple, nous allons créer un dialogue à une seule étape qui peut additionner deux nombres et afficher le résultat correspondant.

Pour utiliser des dialogues, vous devez commencer par créer un *jeu de dialogues*.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

La bibliothèque `Microsoft.Bot.Builder.Dialogs` fournit une classe `DialogSet`.
Créez une classe **AdditionDialog**, puis ajoutez les instructions using dont nous aurons besoin.
Vous pouvez ajouter des dialogues et ensembles de dialogues nommés à un jeu de dialogues, puis y accéder ultérieurement en utilisant leur nom.

```csharp
using Microsoft.Bot.Builder.Dialogs;
```

Dérivez la classe de **DialogSet**, puis définissez les ID et les clés que nous utiliserons pour identifier les dialogues et les informations d’entrée de ce jeu de dialogues.

```csharp
/// <summary>Defines a simple dialog for adding two numbers together.</summary>
public class AdditionDialog : DialogSet
{
    /// <summary>The ID of the main dialog in the set.</summary>
    public const string Main = "additionDialog";

    /// <summary>Defines the IDs of the input arguments.</summary>
    public struct Inputs
    {
        public const string First = "first";
        public const string Second = "second";
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

La bibliothèque `botbuilder-dialogs` fournit une classe `DialogSet`.
La classe **DialogSet** définit un **pile de dialogues**, et offre une interface simple pour gérer la pile.
Le Kit de développement logiciel (SDK) Bot Builder implémente la pile en tant que suite.

Pour créer un **DialogSet**, procédez comme suit :

```javascript
const dialogs = new botbuilder_dialogs.DialogSet();
```

L’appel ci-dessus crée un **DialogSet** avec une **pile de dialogues** par défaut nommée `dialogStack`.
Si vous souhaitez nommer votre pile, vous pouvez la passer en tant que paramètre à **DialogSet()**. Par exemple : 

```javascript
const dialogs = new botbuilder_dialogs.DialogSet("myStack");
```

---

La création d’un dialogue a uniquement pour effet d’ajouter la définition du dialogue au jeu. Le dialogue n’est pas exécuté tant qu’il n’est pas transmis à la pile de dialogues en appelant une méthode _begin_ ou _replace_.

Le nom du dialogue (par exemple, `addTwoNumbers`) doit être unique dans chaque jeu de dialogues. Vous pouvez définir autant de dialogues que nécessaire au sein de chaque jeu. Si vous souhaitez créer plusieurs jeux de dialogues et les combiner en toute transparence, consultez l’article expliquant comment [créer une logique de bot modulaire](bot-builder-compositcontrol.md).

La bibliothèque de dialogues définit les dialogues suivants :

* Un dialogue de type **invite** utilisant au moins deux fonctions, l’une pour inviter l’utilisateur à effectuer une entrée, et l’autre pour traiter celle-ci. Vous pouvez lier ces fonctions à l’aide du modèle **cascade**.
* Un dialogue de type **cascade** définit une séquence d’_étapes en cascade_ qui s’exécutent dans un ordre déterminé. Un dialogue de type cascade peut ne comprendre qu’une seule étape. Il peut alors être considéré comme un dialogue simple d’une étape.

## <a name="create-a-single-step-dialog"></a>Créer un dialogue d’une seule étape

Des dialogues d’une seule étape peuvent être utiles pour capturer des flux de conversation à un seul tour. Cet exemple crée un robot qui peut détecter si l’utilisateur dit quelque chose comme « 1 + 2 », et démarre un dialogue `addTwoNumbers` pour répondre par « 1 + 2 = 3 ».

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Les valeurs sont passées aux dialogues et retournées à partir de ceux-ci en tant que sacs de propriétés `IDictionary<string,object>`.

Pour créer un dialogue simple dans un jeu de dialogues, utilisez la méthode `Add`. Le code suivant ajoute une cascade d’une étape nommée `addTwoNumbers`.

Cette étape suppose que les arguments de dialogue transmis contiennent les propriétés `first` et `second` qui représentent les nombres à ajouter.

Ajoutez le constructeur ci-après à la classe **AdditionDialog**.

```csharp
/// <summary>Defines the steps of the dialog.</summary>
public AdditionDialog()
{
    Add(Main, new WaterfallStep[]
    {
        async (dc, args, next) =>
        {
            // Get the input from the arguments to the dialog and add them.
            var x =(double)args[Inputs.First];
            var y =(double)args[Inputs.Second];
            var sum = x + y;

            // Display the result to the user.
            await dc.Context.SendActivity($"{x} + {y} = {sum}");

            // End the dialog.
            await dc.End();
        }
    });
}
```

### <a name="pass-arguments-to-the-dialog"></a>Passer des arguments au dialogue

Dans le code de votre bot, mettez à jour vos instructions using.

```cs
using Microsoft.Bot;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Schema;
using System.Collections.Generic;
using System.Text.RegularExpressions;
using System.Threading.Tasks;
```

Ajoutez une propriété statique à la classe pour le dialogue d’addition.

```cs
private static AdditionDialog AddTwoNumbers { get; } = new AdditionDialog();
```

Pour appeler le dialogue à partir de la méthode `OnTurn` de votre robot, modifiez `OnTurn` pour contenir ce qui suit :

```cs
public async Task OnTurn(ITurnContext context)
{
    // Handle any message activity from the user.
    if (context.Activity.Type is ActivityTypes.Message)
    {
        // Get the conversation state from the turn context.
        var conversationState = context.GetConversationState<ConversationData>();

        // Generate a dialog context for the addition dialog.
        var dc = AddTwoNumbers.CreateContext(context, conversationState.DialogState);

        // Call a helper function that identifies if the user says something
        // like "2 + 3" or "1.25 + 3.28" and extract the numbers to add.
        if (TryParseAddingTwoNumbers(context.Activity.Text, out double first, out double second))
        {
            // Start the dialog, passing in the numbers to add.
            var args = new Dictionary<string, object>
            {
                [AdditionDialog.Inputs.First] = first,
                [AdditionDialog.Inputs.Second] = second
            };
            await dc.Begin(AdditionDialog.Main, args);
        }
        else
        {
            // Echo back to the user whatever they typed.
            await context.SendActivity($"You said '{context.Activity.Text}'");
        }
    }
}
```

Ajoutez une fonction d’assistance **TryParseAddingTwoNumbers** à la classe de bot. La fonction d’assistance utilise une expression régulière (regex) simple pour détecter si le message de l’utilisateur est une demande d’ajout de 2 nombres.

```cs
// Recognizes if the message is a request to add 2 numbers, in the form: number + number,
// where number may have optionally have a decimal point.: 1 + 1, 123.99 + 45, 0.4+7.
// For the sake of simplicity it doesn't handle negative numbers or numbers like 1,000 that contain a comma.
// If you need more robust number recognition, try System.Recognizers.Text
public static bool TryParseAddingTwoNumbers(string message, out double first, out double second)
{
    // captures a number with optional -/+ and optional decimal portion
    const string NUMBER_REGEXP = "([-+]?(?:[0-9]+(?:\\.[0-9]+)?|\\.[0-9]+))";

    // matches the plus sign with optional spaces before and after it
    const string PLUSSIGN_REGEXP = "(?:\\s*)\\+(?:\\s*)";

    const string ADD_TWO_NUMBERS_REGEXP = NUMBER_REGEXP + PLUSSIGN_REGEXP + NUMBER_REGEXP;

    var regex = new Regex(ADD_TWO_NUMBERS_REGEXP);
    var matches = regex.Matches(message);

    first = 0;
    second = 0;
    if (matches.Count > 0)
    {
        var matched = matches[0];
        if (double.TryParse(matched.Groups[1].Value, out first)
            && double.TryParse(matched.Groups[2].Value, out second))
        {
            return true;
        }
    }
    return false;
}
```

Si vous utilisez le modèle EchoBot, renommez la classe **EchoState** sous le nom **ConversationData** et modifiez-la en y incluant le code suivant.

```cs
using System.Collections.Generic;

/// <summary>
/// Class for storing conversation state.
/// </summary>
public class ConversationData
{
    /// <summary>Property for storing dialog state.</summary>
    public Dictionary<string, object> DialogState { get; set; } = new Dictionary<string, object>();
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Commencez avec le modèle JS décrit dans [Créer un robot avec le Kit de développement logiciel (SDK) Bot Builder v4](../javascript/bot-builder-javascript-quickstart.md). Dans **app.js**, ajoutez une instruction pour exiger `botbuilder-dialogs`.

```js
const {DialogSet} = require('botbuilder-dialogs');
```

Dans **app.js**, ajoutez le code suivant qui définit un dialogue simple nommé `addTwoNumbers` appartenantt au jeu `dialogs` :

```javascript
const dialogs = new DialogSet("myDialogStack");

// Show the sum of two numbers.
dialogs.add('addTwoNumbers', [async function (dc, numbers){
        var sum = Number.parseFloat(numbers[0]) + Number.parseFloat(numbers[1]);
        await dc.context.sendActivity(`${numbers[0]} + ${numbers[1]} = ${sum}`);
        await dc.end();
    }]
);
```

Remplacez le code dans **app.js** pour le traitement de l’activité entrante par le code suivant. Le robot appelle une fonction d’assistance pour vérifier si le message entrant ressemble à une demande d’ajout de deux nombres. Dans l’affirmative, il passe les nombres dans l’argument à `DialogContext.begin`.

```js
// Listen for incoming activity 
server.post('/api/messages', (req, res) => {
    // Route received activity to adapter for processing
    adapter.processActivity(req, res, async (context) => {
        const isMessage = context.activity.type === 'message';
        // State will store all of your information
        const convoState = conversationState.get(context);
        const dc = dialogs.createContext(context, convoState);

        if (isMessage) {
            // TryParseAddingTwoNumbers checks if the message matches a regular expression
            // and if it does, returns an array of the numbers to add
            var numbers = await TryParseAddingTwoNumbers(context.activity.text); 
            if (numbers != null && numbers.length >=2 )
            {
                await dc.begin('addTwoNumbers', numbers);
            }
            else {
                // Just echo back the user's message if they're not adding numbers
                const count = (convoState.count === undefined ? convoState.count = 0 : ++convoState.count);
                return context.sendActivity(`Turn ${count}: You said "${context.activity.text}"`);
            }
        }
        else {
            return context.sendActivity(`[${context.activity.type} event detected]`);
        }

        if (!context.responded) {
            await dc.continue();
            // if the dialog didn't send a response
            if (!context.responded && isMessage) {
                await dc.context.sendActivity(`Hi! I'm the add 2 numbers bot. Say something like "What's 2+3?"`);
            }
        }
    });
});

```

Ajoutez la fonction d’assistance à **app.js**. La fonction d’assistance utilise une expression régulière simple pour détecter si le message de l’utilisateur est une demande d’ajout de 2 nombres. Si l’expression régulière correspond, elle retourne un tableau contenant les nombres à ajouter.

```javascript
async function TryParseAddingTwoNumbers(message) {
    const ADD_NUMBERS_REGEXP = /([-+]?(?:[0-9]+(?:\.[0-9]+)?|\.[0-9]+))(?:\s*)\+(?:\s*)([-+]?(?:[0-9]+(?:\.[0-9]+)?|\.[0-9]+))/i;
    let matched = ADD_NUMBERS_REGEXP.exec(message);
    if (!matched) {
        // message wasn't a request to add 2 numbers
        return null;
    }
    else {
        var numbers = [matched[1], matched[2]];
        return numbers;
    }
}
```

---

### <a name="run-the-bot"></a>Exécuter le bot

Essayez d’exécuter le bot dans Bot Framework Emulator, et dites-lui quelque chose comme « Combien font 1 + 1 ? » .

![Exécuter le bot](./media/how-to-dialogs/bot-output-add-numbers.png)

## <a name="using-dialogs-to-guide-the-user-through-steps"></a>Utilisation de dialogues pour guider l’utilisateur dans des étapes

Dans l’exemple suivant, nous allons créer un dialogue à plusieurs étapes pour inviter l’utilisateur à fournir certaines informations.

### <a name="create-a-dialog-with-waterfall-steps"></a>Créer un dialogue avec des étapes en cascade

Une **cascade** est une implémentation spécifique d’un dialogue, utilisée le plus souvent pour collecter des informations de l’utilisateur ou guider celui-ci dans une série de tâches. Les tâches sont implémentées en tant que suite de fonctions, où le résultat de la première fonction est passé en tant qu’arguments à la fonction suivante, et ainsi de suite. Chaque fonction représente généralement une seule étape dans le processus global. À chaque étape, le robot [invite l’utilisateur à effectuer une entrée](bot-builder-prompts.md), attend une réponse, puis passe le résultat à l’étape suivante.

Par exemple, l’exemple de code ci-après définit trois fonctions dans un tableau qui représente les trois étapes d’une **cascade**. Après chaque invite, le bot accuse réception de l’entrée utilisateur, mais n’enregistre pas cette dernière. Si vous souhaitez conserver les entrées utilisateur, consultez l’article [Conserver les données utilisateur](bot-builder-tutorial-persist-user-inputs.md) pour plus d’informations.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Le code ci-après présente un constructeur pour un dialogue d’accueil, dans lequel **GreetingDialog** dérive de **DialogSet**, **Inputs.Text** contient l’ID que nous utilisons pour l’objet **TextPrompt**, et **Main** contient l’ID du dialogue d’accueil proprement dit.

```csharp
public GreetingDialog()
{
    // Include a text prompt.
    Add(Inputs.Text, new TextPrompt());

    // Define the dialog logic for greeting the user.
    Add(Main, new WaterfallStep[]
    {
        async (dc, args, next) =>
        {
            // Ask for their name.
            await dc.Prompt(Inputs.Text, "What is your name?");
        },
        async (dc, args, next) =>
        {
            // Get the prompt result.
            var name = args["Text"] as string;

            // Acknowledge their input.
            await dc.Context.SendActivity($"Hi, {name}!");

            // Ask where they work.
            await dc.Prompt(Inputs.Text, "Where do you work?");
        },
        async (dc, args, next) =>
        {
            // Get the prompt result.
            var work = args["Text"] as string;

            // Acknowledge their input.
            await dc.Context.SendActivity($"{work} is a fun place.");

            // End the dialog.
            await dc.End();
        }
    });
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// Greet user:
// Ask for the user name and then greet them by name.
// Ask them where they work.
dialogs.add('greetings',[
    async function (dc){
        await dc.prompt('textPrompt', 'What is your name?');
    },
    async function(dc, results){
        var userName = results;
        await dc.context.sendActivity(`Hi ${userName}!`);
        await dc.prompt('textPrompt', 'Where do you work?');
    },
    async function(dc, results){
        var workPlace = results;
        await dc.context.sendActivity(`${workPlace} is a fun place.`);
        await dc.end(); // Ends the dialog
    }
]);

dialogs.add('textPrompt', new botbuilder_dialogs.TextPrompt());
```

---

La signature d’une étape de **cascade** se présente comme suit :

| Paramètre | Description |
| :---- | :----- |
| `dc` | Contexte du dialogue. |
| `args` | (Facultatif) Contient les arguments transmis à l’étape. |
| `next` | (Facultatif) Méthode vous permettant de passer à l’étape suivante de la cascade sans aucune invite. Lorsque vous appelez cette méthode, vous pouvez fournir un argument *args*, ce qui vous permet de transmettre des arguments à l’étape suivante de la cascade. |

Chaque étape doit appeler l’une des méthodes ci-après avant de renvoyer une réponse : le délégué *next()* ou l’une des méthodes de contexte de dialogue *begin*, *end*, *prompt* ou *replace* ; dans le cas contraire, le bot restera bloqué à cette étape. Autrement dit, si une fonction ne s’achève pas par l’une de ces méthodes, toute entrée utilisateur entraîne la réexécution de cette étape chaque fois que l’utilisateur envoie un message au bot.

Lorsque vous avez atteint la fin de la cascade, une bonne pratique consiste à renvoyer la méthode _end_ afin que le dialogue puisse être dépilé. Pour plus d’informations, consultez la section [Terminer un dialogue](#end-a-dialog) dans la suite de cet article. De même, pour permettre le passage à l’étape suivante, une étape de la cascade doit se terminer par une invite ou appeler explicitement le délégué _next_ pour avancer dans la cascade.

## <a name="start-a-dialog"></a>Démarrer un dialogue

Pour démarrer un dialogue, transmettez l’ID *dialogId* du dialogue que vous souhaitez démarrer à la méthode _begin_, _prompt_ ou _replace_ du contexte de dialogue. La méthode _begin_ envoie (push) le dialogue en haut de la pile, tandis que la méthode _replace_ dépile le dialogue actuel et envoie le dialogue de remplacement dans la pile.

Pour démarrer un dialogue sans arguments :

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// Start the greetings dialog.
await dc.Begin("greetings");
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// Start the 'greetings' dialog.
await dc.begin('greetings');
```

---

Pour démarrer un dialogue avec des arguments :

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// Start the greetings dialog, passing in a property bag.
await dc.Begin("greetings", args);
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// Start the 'greetings' dialog with the 'userName' passed in.
await dc.begin('greetings', userName);
```

---

Pour démarrer un dialogue de type **invite** :

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Dans cet exemple, **Inputs.Text** contient l’ID d’une invite **TextPrompt** figurant dans le même jeu de dialogues.

```csharp
// Ask a user for their name.
await dc.Prompt(Inputs.Text, "What is your name?");
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// Ask a user for their name.
await dc.prompt('textPrompt', "What is your name?");
```

---

Selon le type d’invite que vous démarrez, la signature d’argument de l’invite peut varier. La méthode **DialogSet.prompt** est une méthode d’assistance. Cette méthode prend des arguments et construit les options appropriées pour l’invite. Ensuite, elle appelle la méthode **begin** pour démarrer le dialogue de type invite. Pour plus d’informations sur les invites, consultez [Inviter les utilisateurs à saisir une entrée](bot-builder-prompts.md).

## <a name="end-a-dialog"></a>Terminer un dialogue

La méthode _end_ met fin à un dialogue en dépilant ce dernier et renvoie un résultat facultatif au dialogue parent.

Il est recommandé d’appeler explicitement la méthode _end_ à la fin du dialogue. Toutefois, cette opération n’est pas obligatoire, car le dialogue est automatiquement dépilé lorsque vous atteignez la fin de la cascade.

Pour terminer un dialogue :

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// End the current dialog by popping it off the stack.
await dc.End();
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// End the current dialog by popping it off the stack
await dc.end();
```

---

Pour mettre fin à un dialogue et renvoyer des informations au dialogue parent, incluez un argument de conteneur des propriétés.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// End the current dialog and return information to the parent dialog.
await dc.end(new Dictionary<string, object>
    {
        ["property1"] = value1,
        ["property2"] = value2
    });
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// End the current dialog and pass a result to the parent dialog
await dc.end({
    "property1": value1,
    "property2": value2
});
```

---

## <a name="clear-the-dialog-stack"></a>Effacer la pile de dialogues

Si vous souhaitez dépiler tous les dialogues, vous pouvez effacer la pile de dialogues en appelant la méthode _end all_.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// Pop all dialogs from the current stack.
await dc.EndAll();
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// Pop all dialogs from the current stack.
await dc.endAll();
```

---

## <a name="repeat-a-dialog"></a>Répéter un dialogue

Pour répéter un dialogue, utilisez la méthode _replace_. La méthode *replace* du contexte de dialogue dépile le dialogue actuel, puis envoie le dialogue de remplacement en haut de la pile et démarre ce dialogue. Il s’agit là d’un excellent moyen de traiter les [flux de conversation complexes](~/v4sdk/bot-builder-dialog-manage-complex-conversation-flow.md) et de gérer les menus.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// End the current dialog and start the main menu dialog.
await dc.Replace("mainMenu");
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// End the current dialog and start the 'mainMenu' dialog.
await dc.replace('mainMenu');
```

---

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous avez appris à gérer des flux de conversation simples, examinons la manière dont vous pouvez tirer profit de la méthode _replace_ pour traiter des flux de conversation complexes.

> [!div class="nextstepaction"]
> [Gérer des flux de conversation complexes](bot-builder-dialog-manage-complex-conversation-flow.md)
