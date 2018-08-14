---
title: Gérer un flux de conversation avec des dialogues | Microsoft Docs
description: Découvrez comment gérer un flux de conversation avec des dialogues dans le Kit de développement logiciel (SDK) Bot Builder pour Node.js.
keywords: flux de conversation, dialogues, invites, cascades, jeu de dialogues
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 5/8/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 99184ba71072c159c598c7f68289c42a51926795
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39299368"
---
# <a name="manage-conversation-flow-with-dialogs"></a>Gérer un flux de conversation avec des dialogues
[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]


La gestion de flux de conversation est une tâche essentielle dans la création de robots. Le Kit de développement logiciel (SDK) Bot Builder vous permet de gérer un flux de conversation à l’aide de **dialogues**.

Un dialogue est comparable à une fonction dans un programme. Il est généralement conçu pour effectuer une opération spécifique, et peut être appelé aussi souvent que nécessaire. Vous pouvez enchaîner plusieurs dialogues afin de gérer pratiquement tout flux de conversation que robot doit pouvoir gérer. Le bibliothèque de **dialogues** du Kit de développement logiciel (SDK) Bot Builder comprend des fonctionnalités intégrées, telles que des **invites** et **cascades**, destinées à vous aider à gérer un flux de conversation via des dialogues. La bibliothèque d’invites comprend diverses invites que vous pouvez utiliser pour demander aux utilisateurs différents types d’informations. Les cascades permettent de combiner plusieurs étapes dans une séquence.

Cet article montre comment créer un objet dialogues, et ajouter des invites et des étapes de cascade à un jeu de dialogues pour gérer des flux de conversation tant simples que complexes. 

## <a name="install-the-dialogs-library"></a>Installer la bibliothèque de dialogues

# <a name="ctabcsharp"></a>[C#](#tab/csharp)
Pour utiliser des dialogues, vous devez installer le package NuGet `Microsoft.Bot.Builder.Dialogs` pour votre projet ou solution.
Référencez ensuite la bibliothèque de dialogues dans les instructions d’utilisation de vos fichiers de code. Par exemple : 

```csharp
using Microsoft.Bot.Builder.Dialogs;
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)
Vous pouvez télécharger la bibliothèque `botbuilder-dialogs` à partir de NPM. Pour installer la bibliothèque `botbuilder-dialogs`, exécutez la commande NPM suivante :

```cmd
npm install --save botbuilder-dialogs
```

Pour utiliser des **dialogues** dans votre robot, incluez-les dans le code de celui-ci. Par exemple : 

**app.js**

```javascript
const botbuilder_dialogs = require('botbuilder-dialogs');
```
---

## <a name="create-a-dialog-stack"></a>Créer une pile de dialogues

Pour utiliser des dialogues, vous devez commencer par créer un *jeu de dialogues*.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

La bibliothèque `Microsoft.Bot.Builder.Dialogs` fournit une classe `DialogSet`.
Vous pouvez ajouter à un jeu de dialogues des dialogues et ensembles de dialogues nommés, puis y accéder ultérieurement en utilisant leur nom.

```csharp
IDialog dialog = null;
// Initialize dialog.

DialogSet dialogs = new DialogSet();
dialogs.Add("dialog name", dialog);
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

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

Le nom du dialogue (par exemple, `addTwoNumbers`) doit être unique dans chaque jeu de dialogues. Vous pouvez définir autant de dialogues que nécessaire au sein de chaque jeu.

La bibliothèque de dialogues définit les dialogues suivants :
-   Un dialogue de type **invite** utilisant au moins deux fonctions, l’une pour inviter l’utilisateur à effectuer une entrée, et l’autre pour traiter celle-ci.
    Vous pouvez lier ces fonctions à l’aide du modèle **cascade**.
-   Un dialogue de type **cascade** définit une séquence d’_étapes en cascade_ qui s’exécutent dans un ordre déterminé.
    Un dialogue de type cascade peut ne comprendre qu’une seule étape. Il peut alors être considéré comme un dialogue simple d’une étape.

## <a name="create-a-single-step-dialog"></a>Créer un dialogue d’une seule étape

Des dialogues d’une seule étape peuvent être utiles pour capturer des flux de conversation à un seul tour. Cet exemple crée un robot qui peut détecter si l’utilisateur dit quelque chose comme « 1 + 2 », et démarre un dialogue `addTwoNumbers` pour répondre par « 1 + 2 = 3 ». 

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Les valeurs sont passées aux dialogues et retournées à partir de ceux-ci en tant que sacs de propriétés `IDictionary<string,object>`.

Pour créer un dialogue simple dans un jeu de dialogues, utilisez la méthode `Add`. Le code suivant ajoute une cascade d’une étape nommée `addTwoNumbers`.

Cette étape suppose que les arguments de dialogue transmis contiennent les propriétés `first` et `second` qui représentent les nombres à ajouter.

Commencez avec le modèle EchoBot. Ajoutez ensuite le code à votre classe de robot pour ajouter le dialogue dans le constructeur.
```csharp
public class EchoBot : IBot
{
    private DialogSet _dialogs;

    public EchoBot()
    {
        _dialogs = new DialogSet();
        _dialogs.Add("addTwoNumbers", new WaterfallStep[]
        {              
            async (dc, args, next) =>
            {
                double sum = (double)args["first"] + (double)args["second"];
                await dc.Context.SendActivity($"{args["first"]} + {args["second"]} = {sum}");
                await dc.End();
            }
        });
    }

    // The rest of the class definition is omitted here but would include OnTurn()
}

```

### <a name="pass-arguments-to-the-dialog"></a>Passer des arguments au dialogue

Pour appeler le dialogue à partir de la méthode `OnTurn` de votre robot, modifiez `OnTurn` pour contenir ce qui suit :
```cs
public async Task OnTurn(ITurnContext context)
{
    // This bot is only handling Messages
    if (context.Activity.Type == ActivityTypes.Message)
    {
        // Get the conversation state from the turn context
        var state = context.GetConversationState<EchoState>();

        // create a dialog context
        var dialogCtx = _dialogs.CreateContext(context, state);

        // Bump the turn count. 
        state.TurnCount++;

        await dialogCtx.Continue();
        if (!context.Responded)
        {
            // Call a helper function that identifies if the user says something 
            // like "2 + 3" or "1.25 + 3.28" and extract the numbers to add            
            if (TryParseAddingTwoNumbers(context.Activity.Text, out double first, out double second))
            { 
                var dialogArgs = new Dictionary<string, object>
                {
                    ["first"] = first,
                    ["second"] = second
                };                        
                await dialogCtx.Begin("addTwoNumbers", dialogArgs);
            }
            else
            {
                // Echo back to the user whatever they typed.
                await context.SendActivity($"Turn: {state.TurnCount}. You said '{context.Activity.Text}'");
            }
        }
    }
}
```

Ajoutez la fonction d’assistance à la classe de robot. La fonction d’assistance utilise une expression régulière (regex) simple pour détecter si le message de l’utilisateur est une demande d’ajout de 2 nombres.

```cs
// Recognizes if the message is a request to add 2 numbers, in the form: number + number, 
// where number may have optionally have a decimal point.: 1 + 1, 123.99 + 45, 0.4+7. 
// For the sake of simplicity it doesn't handle negative numbers or numbers like 1,000 that contain a comma.
// If you need more robust number recognition, try System.Recognizers.Text
public bool TryParseAddingTwoNumbers(string message, out double first, out double second)
{
    // captures a number with optional -/+ and optional decimal portion
    const string NUMBER_REGEXP = "([-+]?(?:[0-9]+(?:\\.[0-9]+)?|\\.[0-9]+))";
    // matches the plus sign with optional spaces before and after it
    const string PLUSSIGN_REGEXP = "(?:\\s*)\\+(?:\\s*)";
    const string ADD_TWO_NUMBERS_REGEXP = NUMBER_REGEXP + PLUSSIGN_REGEXP + NUMBER_REGEXP;
    var regex = new Regex(ADD_TWO_NUMBERS_REGEXP);
    var matches = regex.Matches(message);
    var succeeded = false;
    first = 0;
    second = 0;
    if (matches.Count == 0)
    {
        succeeded = false;
    }
    else
    {
        var matched = matches[0];
        if ( System.Double.TryParse(matched.Groups[1].Value, out first) 
            && System.Double.TryParse(matched.Groups[2].Value, out second))
        {
            succeeded = true;
        } 
    }
    return succeeded;
}
```

Si vous utilisez le modèle EchoBot, modifiez la classe `EchoState` dans **EchoState.cs** comme suit :

```cs
/// <summary>
/// Class for storing conversation state.
/// This bot only stores the turn count in order to echo it to the user
/// </summary>
public class EchoState: Dictionary<string, object>
{
    private const string TurnCountKey = "TurnCount";
    public EchoState()
    {
        this[TurnCountKey] = 0;            
    }

    public int TurnCount
    {
        get { return (int)this[TurnCountKey]; }
        set { this[TurnCountKey] = value; }
    }
}
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

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
        if (isMessage) {
            const state = conversationState.get(context);
            const count = state.count === undefined ? state.count = 0 : ++state.count;

            // create a dialog context
            const dc = dialogs.createContext(context, state);

            // MatchesAdd2Numbers checks if the message matches a regular expression
            // and if it does, returns an array of the numbers to add
            var numbers = await MatchesAdd2Numbers(context.activity.text); 
            if (numbers != null && numbers.length >=2 )
            {    
                await dc.begin('addTwoNumbers', numbers);
            }
            else {
                // Just echo back the user's message if they're not adding numbers
                return context.sendActivity(`Turn ${count}: You said "${context.activity.text}"`); 
            }           
        } else {
            return context.sendActivity(`[${context.activity.type} event detected]`);
        }
        if (!context.responded) {
            await dc.continue();
            // if the dialog didn't send a response
            if (!context.responded && isMessage) {
                await dc.context.sendActivity(`Hi! I'm the add 2 numbers bot. Say something like "what's 1+2?"`);
            }
        }
    });
});
```

Ajoutez la fonction d’assistance à **app.js**. La fonction d’assistance utilise une expression régulière simple pour détecter si le message de l’utilisateur est une demande d’ajout de 2 nombres. Si l’expression régulière correspond, elle retourne un tableau contenant les nombres à ajouter.

```javascript
async function MatchesAdd2Numbers(message) {
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

### <a name="run-the-bot"></a>Exécuter le robot

Essayez d’exécuter le robot dans l’émulateur Bot Framework, et dites quelque chose comme « Que font 1 + 1 » ? à ce dernier.

![exécuter le bot](./media/how-to-dialogs/bot-output-add-numbers.png)



## <a name="using-dialogs-to-guide-the-user-through-steps"></a>Utilisation de dialogues pour guider l’utilisateur dans des étapes

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

### <a name="create-a-composite-dialog"></a>Créer un dialogue composite

Les extraits de code suivants proviennent de l’exemple de code [Microsoft.Bot.Samples.Dialog.Prompts](https://github.com/Microsoft/botbuilder-dotnet/tree/master/samples/MIcrosoft.Bot.Samples.Dialog.Prompts) dans le référentiel dotnet-botbuilder.

Dans Startup.cs :
1.  Renommez votre robot en `DialogContainerBot`.
1.  Utilisez un dictionnaire simple en guide de sac de propriétés pour l’état de la conversation pour le robot.

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddBot<DialogContainerBot>(options =>
    {
        options.CredentialProvider = new ConfigurationCredentialProvider(Configuration);
        options.Middleware.Add(new ConversationState<Dictionary<string, object>>(new MemoryStorage()));
    });
}
```

Renommer votre `EchoBot` en `DialogContainerBot`.

Dans `DialogContainerBot.cs`, définissez une classe pour un dialogue de profil.

```csharp
public class ProfileControl : DialogContainer
{
    public ProfileControl()
        : base("fillProfile")
    {
        Dialogs.Add("fillProfile", 
            new WaterfallStep[]
            {
                async (dc, args, next) =>
                {
                    dc.ActiveDialog.State = new Dictionary<string, object>();
                    await dc.Prompt("textPrompt", "What's your name?");
                },
                async (dc, args, next) =>
                {
                    dc.ActiveDialog.State["name"] = args["Value"];
                    await dc.Prompt("textPrompt", "What's your phone number?");
                },
                async (dc, args, next) =>
                {
                    dc.ActiveDialog.State["phone"] = args["Value"];
                    await dc.End(dc.ActiveDialog.State);
                }
            }
        );
        Dialogs.Add("textPrompt", new Builder.Dialogs.TextPrompt());
    }
}
```

Ensuite, dans la définition du robot, déclarez un champ pour le dialogue principal du robot, puis initialisez-le dans le constructeur du robot.
Le dialogue principal du robot inclut le dialogue de profil.

```csharp
private DialogSet _dialogs;

public DialogContainerBot()
{
    _dialogs = new DialogSet();

    _dialogs.Add("getProfile", new ProfileControl());
    _dialogs.Add("firstRun",
        new WaterfallStep[]
        {
            async (dc, args, next) =>
            {
                    await dc.Context.SendActivity("Welcome! We need to ask a few questions to get started.");
                    await dc.Begin("getProfile");
            },
            async (dc, args, next) =>
            {
                await dc.Context.SendActivity($"Thanks {args["name"]} I have your phone number as {args["phone"]}!");
                await dc.End();
            }
        }
    );
}
```

Dans la méthode `OnTurn` du robot :
-   Accueillez l’utilisateur au démarrage de la conversation.
-   Initialisez et _continuez_ le dialogue principal chaque fois que vous recevez un message de l’utilisateur.

    Si le dialogue n’a pas généré de réponse, supposez qu’il exécuté précédemment ou qu’il n’a pas encore démarré, et _entamez_-le en spécifiant le nom du dialogue dans le jeu avec lequel commencer.

```csharp
public async Task OnTurn(ITurnContext turnContext)
{
    try
    {
        switch (turnContext.Activity.Type)
        {
            case ActivityTypes.ConversationUpdate:
                foreach (var newMember in turnContext.Activity.MembersAdded)
                {
                    if (newMember.Id != turnContext.Activity.Recipient.Id)
                    {
                        await turnContext.SendActivity("Hello and welcome to the Composite Control bot.");
                    }
                }
                break;

            case ActivityTypes.Message:
                var state = ConversationState<Dictionary<string, object>>.Get(turnContext);
                var dc = _dialogs.CreateContext(turnContext, state);

                await dc.Continue();

                if (!turnContext.Responded)
                {
                    await dc.Begin("firstRun");
                }

                break;
        }
    }
    catch (Exception e)
    {
        await turnContext.SendActivity($"Exception: {e.Message}");
    }
}

```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

### <a name="create-a-dialog-with-waterfall-steps"></a>Créer un dialogue avec des étapes en cascade

Une conversation se compose d’une série de messages échangés entre un utilisateur et un robot. Quand l’objectif du robot est de guider l’utilisateur dans une série d’étapes, vous pouvez utiliser un modèle **de cascade** pour définir les étapes de la conversation.

Une **cascade** est une implémentation spécifique d’un dialogue, utilisée le plus souvent pour collecter des informations de l’utilisateur ou guider celui-ci dans une série de tâches. Les tâches sont implémentées en tant que suite de fonctions, où le résultat de la première fonction est passé en tant qu’arguments à la fonction suivante, et ainsi de suite. Chaque fonction représente généralement une seule étape dans le processus global. À chaque étape, le robot [invite l’utilisateur à effectuer une entrée](bot-builder-prompts.md), attend une réponse, puis passe le résultat à l’étape suivante.

Par exemple, l’exemple de code suivant définit trois fonctions dans une série qui représente les trois étapes d’une **cascade** :

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

La signature d’une étape de **cascade** se présente comme suit :

| Paramètre | Description |
| ---- | ----- |
| `context` | Contexte du dialogue. |
| `args` | Facultatif, contient un ou plusieurs arguments passés dans l’étape. |
| `next` | Facultatif, il s’agit d’une méthode qui vous permet de passer à l’étape suivante de la cascade. Lorsque vous appelez cette méthode, vous pouvez fournir un argument *args*, ce qui vous permet de passer un ou plusieurs arguments à l’étape suivante dans la cascade. |

Chaque étape doit appeler l’une des méthodes suivantes avant de retourner une réponse : *next()*, *dialogs.prompt()*, *dialogs.end()*, *dialogs.begin()* ou *Promise.resolve()*. Autrement, le robot reste bloqué à cette étape. Autrement dit, si une fonction ne retourne aucune de ces méthodes, toute entrée d’utilisateur entraîne la réexécution de cette étape chaque fois que l’utilisateur envoie un message au robot.

Lorsque vous atteint la fin de la cascade, il est recommandé de retourner avec la méthode `end()` de façon à ce que le dialogue soit retiré de la pile. Pour plus d’informations, voir la section [Terminer un dialogue](#end-a-dialog). De même, pour passer d’une étape à la suivante, l’étape de la cascade doit se terminer par une invite ou appeler explicitement la fonction `next()` pour avancer dans la cascade. 

### <a name="start-a-dialog"></a>Démarrer un dialogue

Pour démarrer un dialogue, passez le *dialogId* à démarrer dans les méthodes `begin()`, `prompt()` ou `replace()`. La méthode **begin** pousse le dialogue en haut de la pile, tandis que la méthode **replace** retire le dialogue actif de la pile et pousse le dialogue de remplacement sur la pile.

Pour démarrer un dialogue sans arguments :

```javascript
// Start the 'greetings' dialog.
await dc.begin('greetings');
```

Pour démarrer un dialogue avec des arguments :

```javascript
// Start the 'greetings' dialog with the 'userName' passed in. 
await dc.begin('greetings', userName);
```

Pour démarrer un dialogue de type **invite** :

```javascript
// Start a 'choicePrompt' dialog with choices passed in as an array of colors to choose from.
await dc.prompt('choicePrompt', `choice: select a color`, ['red', 'green', 'blue']);
```

Selon le type d’invite que vous démarrez, la signature d’argument de l’invite peut varier. La méthode **DialogSet.prompt** est une méthode d’assistance. Cette méthode prend des arguments et construit les options appropriées pour l’invite. Ensuite, elle appelle la méthode **begin** pour démarrer le dialogue de type invite.

Pour remplacer un dialogue sur la pile :

```javascript
// End the current dialog and start the 'mainMenu' dialog.
await dc.replace('mainMenu'); // Can optionally passed in an 'args' as the second argument.
```

Pour plus d’informations sur l’utilisation de la méthode **replace()**, voir les sections [Répéter un dialogue](#repeat-a-dialog) et [Boucles de dialogue](#dialog-loops) ci-dessous.

## <a name="end-a-dialog"></a>Terminer un dialogue

Pour terminer un dialogue, retirez-le de la pile et retournez un résultat facultatif au dialogue parent. La méthode **Dialog.resume()** du dialogue parent est appelée avec tout résultat retourné.

Il est recommandé d’appeler explicitement la méthode `end()`à la fin du dialogue. Ce n’est cependant pas obligatoire, car le dialogue est automatiquement retiré de la pile lorsque vous atteignez la fin de la cascade.

Pour terminer un dialogue :

```javascript
// End the current dialog by popping it off the stack
await dc.end();
```

Pour terminer un dialogue avec un ou plusieurs arguments facultatifs passés au dialogue parent :

```javascript
// End the current dialog and pass a result to the parent dialog
await dc.end(result);
```

Vous pouvez également terminer le dialogue en retournant une promesse résolue :

```javascript
await Promise.resolve();
```

L’appel de `Promise.resolve()` entraîne la fin du dialogue et son retrait de la pile. Toutefois, cette méthode n’appelle pas le dialogue parent pour reprendre l’exécution. Après l’appel de `Promise.resolve()`, l’exécution s’arrête, et le robot reprend là où le dialogue parent s’était arrêté lorsque l’utilisateur envoie un message au robot. Terminer un dialogue n’offre peut-être pas l’expérience utilisateur idéale. Envisagez de terminer un dialogue avec `end()` ou `replace()`, afin que votre robot puisse continuer à interagir avec l’utilisateur.

### <a name="clear-the-dialog-stack"></a>Effacer la pile de dialogues

Si vous souhaitez retirer tous les dialogues de la pile, vous pouvez effacer la pile de dialogues en appelant la méthode `dc.endAll()`.

```javascript
// Pop all dialogs from the current stack.
await dc.endAll();
```

### <a name="repeat-a-dialog"></a>Répéter un dialogue

Pour répéter un dialogue, utilisez la méthode `dialogs.replace()`.

```javascript
// End the current dialog and start the 'mainMenu' dialog.
await dc.replace('mainMenu'); 
```

Si vous souhaitez afficher le menu principal par défaut, vous pouvez créer un dialogue `mainMenu` avec les étapes suivantes :

```javascript
// Display a menu and ask user to choose a menu item. Direct user to the item selected.
dialogs.add('mainMenu', [
    async function(dc){
        await dc.context.sendActivity("Welcome to Contoso Hotel and Resort.");
        await dc.prompt('choicePrompt', "How may we serve you today?", ['Order Dinner', 'Reserve a table']);
    },
    async function(dc, result){
        if(result.value.match(/order dinner/ig)){
            await dc.begin('orderDinner');
        }
        else if(result.value.match(/reserve a table/ig)){
            await dc.begin('reserveTable');
        }
        else {
            // Repeat the menu
            await dc.replace('mainMenu');
        }
    },
    async function(dc, result){
        // Start over
        await dc.endAll().begin('mainMenu');
    }
]);

dialogs.add('choicePrompt', new botbuilder_dialogs.ChoicePrompt());
```

Ce dialogue utilise un `ChoicePrompt` pour afficher le menu et attend que l’utilisateur choisisse une option. Lorsque l’utilisateur choisit `Order Dinner` ou `Reserve a table`, il démarre le dialogue pour le choix approprié, puis, une fois cette tâche effectuée, au lieu de simplement terminer le dialogue de la dernière étape, ce dialogue se répète.

### <a name="dialog-loops"></a>Boucles de dialogue

Une autre façon d’utiliser la méthode `replace()` consiste à émuler des boucles. Prenez, par exemple, ce scénario. Si vous souhaitez autoriser l’utilisateur à ajouter plusieurs articles à un panier, vous pouvez afficher en boucle les choix du menu jusqu’à ce que l’utilisateur ait fini de commander.

```javascript
// Order dinner:
// Help user order dinner from a menu

var dinnerMenu = {
    choices: ["Potato Salad - $5.99", "Tuna Sandwich - $6.89", "Clam Chowder - $4.50", 
        "More info", "Process order", "Cancel", "Help"],
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

// The order cart
var orderCart = {
    orders: [],
    total: 0,
    clear: function(dc) {
        this.orders = [];
        this.total = 0;
        dc.context.activity.conversation.orderCart = null;
    }
};

dialogs.add('orderDinner', [
    async function (dc){
        await dc.context.sendActivity("Welcome to our Dinner order service.");
        orderCart.clear(dc); // Clears the cart.

        await dc.begin('orderPrompt'); // Prompt for orders
    },
    async function (dc, result) {
        if(result == "Cancel"){
            await dc.end();
        }
        else { 
            await dc.prompt('numberPrompt', "What is your room number?");
        }
    },
    async function(dc, result){
        await dc.context.sendActivity(`Thank you. Your order will be delivered to room ${result} within 45 minutes.`);
        await dc.end();
    }
]);

// Helper dialog to repeatedly prompt user for orders
dialogs.add('orderPrompt', [
    async function(dc){
        await dc.prompt('choicePrompt', "What would you like?", dinnerMenu.choices);
    },
    async function(dc, choice){
        if(choice.value.match(/process order/ig)){
            if(orderCart.orders.length > 0) {
                // Process the order
                // ...
                await dc.end();
            }
            else {
                await dc.context.sendActivity("Your cart was empty. Please add at least one item to the cart.");
                // Ask again
                await dc.replace('orderPrompt');
            }
        }
        else if(choice.value.match(/cancel/ig)){
            orderCart.clear(context);
            await dc.context.sendActivity("Your order has been canceled.");
            await dc.end(choice.value);
        }
        else if(choice.value.match(/more info/ig)){
            var msg = "More info: <br/>Potato Salad: contains 330 calaries per serving. <br/>"
                + "Tuna Sandwich: contains 700 calaries per serving. <br/>" 
                + "Clam Chowder: contains 650 calaries per serving."
            await dc.context.sendActivity(msg);
            
            // Ask again
            await dc.replace('orderPrompt');
        }
        else if(choice.value.match(/help/ig)){
            var msg = `Help: <br/>To make an order, add as many items to your cart as you like then choose the "Process order" option to check out.`
            await dc.context.sendActivity(msg);
            
            // Ask again
            await dc.replace('orderPrompt');
        }
        else {
            var choice = dinnerMenu[choice.value];

            // Only proceed if user chooses an item from the menu
            if(!choice){
                await dc.context.sendActivity("Sorry, that is not a valid item. Please pick one from the menu.");
                
                // Ask again
                await dc.replace('orderPrompt');
            }
            else {
                // Add the item to cart
                orderCart.orders.push(choice);
                orderCart.total += dinnerMenu[choice.value].Price;

                await dc.context.sendActivity(`Added to cart: ${choice.value}. <br/>Current total: $${orderCart.total}`);

                // Ask again
                await dc.replace('orderPrompt');
            }
        }
    }
]);

// Define prompts
// Generic prompts
dialogs.add('textPrompt', new botbuilder_dialogs.TextPrompt());
dialogs.add('numberPrompt', new botbuilder_dialogs.NumberPrompt());
dialogs.add('dateTimePrompt', new botbuilder_dialogs.DatetimePrompt());
dialogs.add('choicePrompt', new botbuilder_dialogs.ChoicePrompt());

```

L’exemple de code ci-dessus montre que le dialogue `orderDinner` principal utilise un dialogue d’assistance nommé `orderPrompt` pour traiter les choix de l’utilisateur. Le dialogue `orderPrompt` affiche le menu, invite l’utilisateur à choisir un article, ajoute celui-ci au panier, puis invite à nouveau l’utilisateur. Cela permet à l’utilisateur d’ajouter plusieurs articles à sa commande. Le dialogue se répète en boucle jusqu’à ce que l’utilisateur choisisse `Process order` ou `Cancel`. À ce moment-là, l’exécution est rendue au dialogue parent (par exemple, `orderDinner`). Le dialogue `orderDinner` fait un ménage de dernière minute si l’utilisateur souhaite traiter la commande. Autrement, il se termine et rend l’exécution à son dialogue parent (par exemple, `mainMenu`). Le dialogue `mainMenu` continue à son tour l’exécution de la dernière étape qui consiste simplement à réafficher les choix du menu principal.

---

## <a name="next-steps"></a>Étapes suivantes

À présent que vous avez appris à utiliser des **dialogues**, des **invites** et des **cascades** pour gérer un flux de conversation, voyons comment décomposer les dialogues en tâches modulaires au lieu de les regrouper tous dans l’objet `dialogs` de la logique principale du robot.

> [!div class="nextstepaction"]
> [Créer une logique modulaire de robot avec contrôle composite](bot-builder-compositcontrol.md)