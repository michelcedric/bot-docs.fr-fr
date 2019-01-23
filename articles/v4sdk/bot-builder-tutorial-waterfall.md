---
redirect_url: /bot-framework/bot-builder-prompts
ms.openlocfilehash: d45ec888a0082ee17718c93fc34a3df99431a254
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/11/2019
ms.locfileid: "54225414"
---
<a name="--"></a><!--
---
titre : Poser des questions à l’utilisateur | Microsoft Docs Description : Apprenez à utiliser le modèle en cascade pour demander à un utilisateur plusieurs entrées dans le kit SDK Bot Framework.
mots clés : cascades, dialogues, poser une question, auteur d’invites : v-ducvo ms.author: v-ducvo manager: kamrani ms.topic: article ms.service: bot-service ms.subservice: sdk ms.date: 5/10/2018 monikerRange : « azure-bot-service-4.0'
---

# <a name="ask-the-user-questions"></a>Poser des questions à l’utilisateur

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Fondamentalement, un bot repose sur la conversation avec un utilisateur. Une conversation peut prendre de [nombreuses formes](bot-builder-conversations.md) ; elle peut être courte ou plus complexe, poser des questions ou répondre à des questions. La forme que prend la conversation dépend de plusieurs facteurs, mais ils impliquent tous une conversation.

Ce tutoriel vous guide tout au long de la création d’une conversation, d’une simple question posée à un bot à plusieurs tours. Notre exemple concernera la réservation d’une table, mais vous pouvez imaginer une conversation à plusieurs tours où un bot fait des choses variées, comme par exemple passer une commande, répondre à des questions dans une FAQ, faire des réservations, etc.

Un bot conversationnel interactif peut répondre à une entrée utilisateur ou lui demander une entrée spécifique. Ce tutoriel vous explique comment poser une question à un utilisateur à l’aide de la bibliothèque `Prompts`, qui fait partie de `Dialogs`. Les [ dialogues](../bot-service-design-conversation-flow.md) peuvent être considérés comme le conteneur qui définit la structure d’une conversation, et les invites des dialogues sont traitées plus en détail dans leur propre [article de procédure](bot-builder-prompts.md).

## <a name="prerequisite"></a>Configuration requise

Le code de ce tutoriel reposera sur le **bot de base** créé par le biais de l’expérience de [prise en main](~/bot-service-quickstart.md).

## <a name="get-the-package"></a>Obtenir le package

# <a name="ctabcstab"></a>[C#](#tab/cstab)

Installez le package **Microsoft.Bot.Builder.Dialogs** à partir du gestionnaire de paquet NuGet.

# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)
Accédez à votre dossier de projet de bot et installez le package `botbuilder-dialogs` à partir de NPM :

```cmd
npm install --save botbuilder-dialogs@preview
```

---

## <a name="import-package-to-bot"></a>Importez le package dans le bot

# <a name="ctabcstab"></a>[C#](#tab/cstab)

Ajoutez une référence aussi bien aux dialogues qu’aux invites de votre code de bot.

```cs
// ...
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Bot.Builder.Prompts;
// ...
```


# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)

Ouvrez le fichier **app.js** et incluez la bibliothèque `botbuilder-dialogs` dans le code de bot.

```javascript
const botbuilder_dialogs = require('botbuilder-dialogs');
```

---

Cela vous donnera accès aux bibliothèques `DialogSet` et `Prompts`, que vous utiliserez pour poser des questions à l’utilisateur. `DialogSet` est simplement une collection de dialogues que nous structurons dans un modèle **en cascade**. Cela signifie simplement qu’un dialogue en suit une autre.

## <a name="instantiate-a-dialogs-object"></a>Instancier un objet de dialogue

Instancier un objet `dialogs`. Vous utiliserez cet objet de dialogue pour gérer la question et le processus de réponse.

# <a name="ctabcstab"></a>[C#](#tab/cstab)
Déclarez une variable de membre dans votre classe de bot et initialisez-la dans le constructeur de votre bot. 
```cs
public class MyBot : IBot
{
    private readonly DialogSet dialogs;
    public MyBot()
    {
        dialogs = new DialogSet();
    }
    // The rest of the class definition is omitted here
}
```

# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)

```javascript
const dialogs = new botbuilder_dialogs.DialogSet();
```
---

## <a name="define-a-waterfall-dialog"></a>Définir un dialogue en cascade

Pour poser une question, vous aurez besoin d’au moins un dialogue **en cascade** en deux étapes. Pour cet exemple, vous construirez un dialogue **en cascade** en deux étapes, où l’on demandera son nom à l’utilisateur lors de la première étape, et où on lui souhaitera la bienvenue par son nom lors de la deuxième étape. 

# <a name="ctabcstab"></a>[C#](#tab/cstab)

Modifiez le constructeur de votre bot pour ajouter le dialogue :
```csharp
public MyBot()
{
    dialogs = new DialogSet();
    dialogs.Add("greetings", new WaterfallStep[]
    {
        async (dc, args, next) =>
        {
            // Prompt for the guest's name.
            await dc.Prompt("textPrompt","What is your name?");
        },
        async(dc, args, next) =>
        {
            await dc.Context.SendActivity($"Hi {args["Text"]}!");
            await dc.End();
        }
    });
}
```

# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)

```javascript
// Greet user:
// Ask for the user name and then greet them by name.
dialogs.add('greetings',[
    async function (dc){
        await dc.prompt('textPrompt', 'What is your name?');
    },
    async function(dc, results){
        var userName = results;
        await dc.context.sendActivity(`Hello ${userName}!`);
        await dc.end(); // Ends the dialog
    }
]);
```

---

La question est posée à l’aide d’une méthode `textPrompt` fournie avec la bibliothèque `Prompts`. La bibliothèque `Prompts` propose un ensemble d’invites qui vous permettent de demander différents types d’informations aux utilisateurs. Pour plus d’informations sur les autres types d’invites, consultez [Inviter l’utilisateur à effectuer une saisie](~/v4sdk/bot-builder-prompts.md).

Pour que les invites fonctionnent, vous devez ajouter une invite à l’objet `dialogs` avec l’identifiant dialogId `textPrompt` et la créer avec le constructeur `TextPrompt()`.

# <a name="ctabcstab"></a>[C#](#tab/cstab)

```cs
public MyBot()
{
    dialogs = new DialogSet();
    dialogs.Add("greetings", new WaterfallStep[]
    {
        async (dc, args, next) =>
        {
            // Prompt for the guest's name.
            await dc.Prompt("textPrompt","What is your name?");
        },
        async(dc, args, next) =>
        {
            await dc.Context.SendActivity($"Hi {args["Text"]}!");
            await dc.End();
        }
    });
    // add the prompt, of type TextPrompt
    dialogs.Add("textPrompt", new Builder.Dialogs.TextPrompt());
}

```
Une fois que l’utilisateur répond à la question, vous pouvez trouver la réponse dans le paramètre `args` de l’étape 2.

# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)

```javascript
dialogs.add('textPrompt', new botbuilder_dialogs.TextPrompt());
```
Une fois que l’utilisateur répond à la question, vous pouvez trouver la réponse dans le paramètre `results` de l’étape 2. Dans ce cas, le paramètre `results` est affecté à une variable locale `userName`. Dans un prochain tutoriel, nous allons vous montrer comment conserver les entrées des utilisateurs dans un stockage de votre choix.

---


Maintenant que vous avez défini votre `dialogs` pour poser une question, vous devez appeler le dialogue pour démarrer le processus d’invite.

## <a name="start-the-dialog"></a>Démarrer le dialogue

# <a name="ctabcstab"></a>[C#](#tab/cstab)

Modifiez la logique de votre bot à peu près comme suit :

```cs

public async Task OnTurn(ITurnContext context)
{
    // We'll cover state later, in the next tutorial
    var state = ConversationState<Dictionary<string, object>>.Get(context);
    var dc = dialogs.CreateContext(context, state);
    if (context.Activity.Type == ActivityTypes.Message)
    {
        await dc.Continue();
        
        if(!context.Responded)
        {
            await dc.Begin("greetings");
        }
    }
}
```

La logique du bot rentre dans la méthode `OnTurn()`. Une fois que l’utilisateur dit « Salut », le bot démarre le dialogue `greetings`. La première étape du dialogue `greetings` demande son nom à l’utilisateur. L’utilisateur envoie une réponse avec son nom comme activité de message, et le résultat est envoyé à l’étape 2 de la cascade via la méthode `dc.Continue()`. La deuxième étape de la cascade telle que vous l’avez définie souhaite la bienvenue à l’utilisateur par son nom et termine le dialogue. 

# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)

Modifiez votre méthode `processActivity()` de bot **de base** pour qu’elle ressemble à ceci :

```javascript
// Listen for incoming activity 
server.post('/api/messages', (req, res) => {
    adapter.processActivity(req, res, async (context) => {
        const isMessage = (context.activity.type === 'message');
        // State will store all of your information 
        const convo = conversationState.get(context);
        const dc = dialogs.createContext(context, convo);

        if (isMessage) {
            // Check for valid intents
            if(context.activity.text.match(/hi/ig)){
                await dc.begin('greetings');
            }
        }

        if(!context.responded){
            // Continue executing the "current" dialog, if any.
            await dc.continue();

            if(!context.responded && isMessage){
                // Default message
                await context.sendActivity("Hi! I'm a simple bot. Please say 'Hi'.");
            }
        }
    });
});
```

La logique du bot rentre dans la méthode `processActivity()`. Une fois que l’utilisateur dit « Salut », le bot démarre le dialogue `greetings`. La première étape du dialogue `greetings` demande son nom à l’utilisateur. L’utilisateur envoie une réponse avec son nom en tant que message de l’activité `text`. Étant donné que le message ne correspond à aucune intention attendue et que le bot n’a envoyé aucune réponse à l’utilisateur, le résultat est envoyé à l’étape 2 de la cascade via la méthode `dc.continue()`. La deuxième étape de la cascade telle que vous l’avez définie souhaite la bienvenue à l’utilisateur par son nom et termine le dialogue. Si, par exemple, l’étape 2 n’a pas envoyé le message de bienvenue à l’utilisateur, la méthode `processActivity` se termine par le *message par défaut* envoyé à l’utilisateur.

---



## <a name="define-a-more-complex-waterfall-dialog"></a>Définir un dialogue en cascade plus complexe

Maintenant que nous avons traité le fonctionnement d’un dialogue en cascade et la manière d’en créer un, faisons un essai avec un dialogue plus complexe destiné à la réservation d’une table.

Pour gérer la requête de réservation de table, vous devez définir un dialogue **en cascade** de quatre étapes. Dans cette conversation, vous utiliserez également une `DatetimePrompt` et une `NumberPrompt` en plus de l’`TextPrompt`.



# <a name="ctabcstab"></a>[C#](#tab/cstab)

Commencez par le modèle Echo Bot et renommez votre bot CafeBot. Ajoutez un `DialogSet` et des variables de membre statique.

```cs

namespace CafeBot
{
    public class CafeBot : IBot
    {
        private readonly DialogSet dialogs;

        // Usually, we would save the dialog answers to our state object, which will be covered in a later tutorial.
        // For purpose of this example, let's use the three static variables to store our reservation information.
        static DateTime reservationDate;
        static int partySize;
        static string reservationName;

        // the rest of the class definition is omitted here
        // but is discussed in the rest of this article
    }
}
```

Définissez ensuite votre dialogue `reserveTable`. Vous pouvez ajouter le dialogue au constructeur de classe de bot.
```cs
public CafeBot()
{
    dialogs = new DialogSet();

    // Define our dialog
    dialogs.Add("reserveTable", new WaterfallStep[]
    {
        async (dc, args, next) =>
        {
            // Prompt for the guest's name.
            await dc.Context.SendActivity("Welcome to the reservation service.");

            await dc.Prompt("dateTimePrompt", "Please provide a reservation date and time.");
        },
        async(dc, args, next) =>
        {
            var dateTimeResult = ((DateTimeResult)args).Resolution.First();

            reservationDate = Convert.ToDateTime(dateTimeResult.Value);
            
            // Ask for next info
            await dc.Prompt("partySizePrompt", "How many people are in your party?");

        },
        async(dc, args, next) =>
        {
            partySize = (int)args["Value"];

            // Ask for next info
            await dc.Prompt("textPrompt", "Whose name will this be under?");
        },
        async(dc, args, next) =>
        {
            reservationName = args["Text"];
            string msg = "Reservation confirmed. Reservation details - " +
            $"\nDate/Time: {reservationDate.ToString()} " +
            $"\nParty size: {partySize.ToString()} " +
            $"\nReservation name: {reservationName}";
            await dc.Context.SendActivity(msg);
            await dc.End();
        }
    });

     // Add a prompt for the reservation date
     dialogs.Add("dateTimePrompt", new Microsoft.Bot.Builder.Dialogs.DateTimePrompt(Culture.English));
     // Add a prompt for the party size
     dialogs.Add("partySizePrompt", new Microsoft.Bot.Builder.Dialogs.NumberPrompt<int>(Culture.English));
     // Add a prompt for the user's name
     dialogs.Add("textPrompt", new Microsoft.Bot.Builder.Dialogs.TextPrompt());
}
```


# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)

Le dialogue `reserveTable` ressemblera à ceci :

```javascript
// Reserve a table:
// Help the user to reserve a table
var reservationInfo = {};

dialogs.add('reserveTable', [
    async function(dc, args, next){
        await dc.context.sendActivity("Welcome to the reservation service.");

        reservationInfo = {}; // Clears any previous data
        await dc.prompt('dateTimePrompt', "Please provide a reservation date and time.");
    },
    async function(dc, result){
        reservationInfo.dateTime = result[0].value;

        // Ask for next info
        await dc.prompt('partySizePrompt', "How many people are in your party?");
    },
    async function(dc, result){
        reservationInfo.partySize = result;

        // Ask for next info
        await dc.prompt('textPrompt', "Whose name will this be under?");
    },
    async function(dc, result){
        reservationInfo.reserveName = result;
        
        // Reservation confirmation
        var msg = `Reservation confirmed. Reservation details: 
            <br/>Date/Time: ${reservationInfo.dateTime} 
            <br/>Party size: ${reservationInfo.partySize} 
            <br/>Reservation name: ${reservationInfo.reserveName}`;
        await dc.context.sendActivity(msg);
        await dc.end();
    }
]);
```

---

Le flux de la conversation du dialogue `reserveTable` pose à l’utilisateur 3 questions via les trois premières étapes de la cascade. L’étape 4 traite la réponse à la dernière question et envoie la confirmation de la réservation à l’utilisateur.



# <a name="ctabcstab"></a>[C#](#tab/cstab)
Chaque étape en cascade du dialogue `reserveTable` utilise une invite pour demander des informations à l’utilisateur. Le code suivant a été utilisé pour ajouter les invites à l’ensemble de dialogues.

```cs
dialogs.Add("dateTimePrompt", new Microsoft.Bot.Builder.Dialogs.DateTimePrompt(Culture.English));
dialogs.Add("partySizePrompt", new Microsoft.Bot.Builder.Dialogs.NumberPrompt<int>(Culture.English));
dialogs.Add("textPrompt", new Microsoft.Bot.Builder.Dialogs.TextPrompt());
```

# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)

Pour que cette cascade fonctionne, vous devez également ajouter ces invites à l’objet `dialogs` :

```javascript
// Define prompts
// Generic prompts
dialogs.add('textPrompt', new botbuilder_dialogs.TextPrompt());
dialogs.add('dateTimePrompt', new botbuilder_dialogs.DatetimePrompt());
dialogs.add('partySizePrompt', new botbuilder_dialogs.NumberPrompt());
```

---

Maintenant, vous êtes prêt à raccorder cela à la logique du bot.

## <a name="start-the-dialog"></a>Démarrer le dialogue

# <a name="ctabcstab"></a>[C#](#tab/cstab)
Modifiez `OnTurn` dans votre bot pour qu’il contienne le code suivant :
```cs
public async Task OnTurn(ITurnContext context)
{
    if (context.Activity.Type == ActivityTypes.Message)
    {
        // The type parameter PropertyBag inherits from 
        // Dictionary<string,object>
        var state = ConversationState<Dictionary<string, object>>.Get(context);
        var dc = dialogs.CreateContext(context, state);
        await dc.Continue();

        // Additional logic can be added to enter each dialog depending on the message received
        
        if(!context.Responded)
        {
            if (context.Activity.Text.ToLowerInvariant().Contains("reserve table"))
            {
                await dc.Begin("reserveTable");
            }
            else
            {
                await context.SendActivity($"You said '{context.Activity.Text}'");
            }
        }
    }
}
```


Dans **Startup.cs**, modifiez l’initialisation de l’intergiciel ConversationState pour utiliser une classe dérivée de `Dictionary<string,object>` au lieu de `EchoState`.

Par exemple, dans `Configure()` :
```cs
options.Middleware.Add(new ConversationState<Dictionary<string, object>>(dataStore));
```


# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)

Pour saisir l’intention de l’utilisateur concernant cette requête, ajoutez quelques lignes de code à la méthode `processActivity()`. Modifiez la méthode `processActivity()` de votre bot pour qu’elle ressemble à ceci :

```javascript
// Listen for incoming activity 
server.post('/api/messages', (req, res) => {
    adapter.processActivity(req, res, async (context) => {
        const isMessage = (context.activity.type === 'message');
        // State will store all of your information 
        const convo = conversationState.get(context);
        const dc = dialogs.createContext(context, convo);

        if (isMessage) {
            // Check for valid intents
            if(context.activity.text.match(/hi/ig)){
                await dc.begin('greetings');
            }
            else if(context.activity.text.match(/reserve table/ig)){
                await dc.begin('reserveTable');
            }
        }

        if(!context.responded){
            // Continue executing the "current" dialog, if any.
            await dc.continue();

            if(!context.responded && isMessage){
                // Default message
                await context.sendActivity("Hi! I'm a simple bot. Please say 'Hi' or 'Reserve table'.");
            }
        }
    });
});
```

Au moment de l’exécution, chaque fois que l’utilisateur envoie le message contenant la chaîne `reserve table`, le bot démarre la conversation `reserveTable`.

---



## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, le bot enregistre l’entrée de l’utilisateur dans des variables au sein de notre bot. Si vous souhaitez stocker ou conserver ces informations, vous devez ajouter un état à la couche d’intergiciel. Examinons plus en détail comment conserver des données d’état des utilisateurs dans le tutoriel suivant. 

> [!div class="nextstepaction"]
> [Conserver les données utilisateur](bot-builder-tutorial-persist-user-inputs.md)

-->
