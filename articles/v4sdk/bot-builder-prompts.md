---
title: Inviter les utilisateurs à saisir une entrée à l’aide de la bibliothèque Dialogs | Microsoft Docs
description: Découvrez comment demander aux utilisateurs de saisir une entrée à l’aide de la bibliothèque Dialogs du Kit de développement logiciel (SDK) Bot Builder pour Node.js.
keywords: invites, dialogues, AttachmentPrompt, ChoicePrompt, ConfirmPrompt, DatetimePrompt, NumberPrompt, TextPrompt, inviter, validation
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 4/10/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 0b238ed510fd1d6fda82734af373f344b0dc28e3
ms.sourcegitcommit: 2dc75701b169d822c9499e393439161bc87639d2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/24/2018
ms.locfileid: "42905363"
---
# <a name="prompt-users-for-input-using-the-dialogs-library"></a>Inviter les utilisateurs à saisir une entrée à l’aide de la bibliothèque Dialogs

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Souvent, les bots collectent des informations via des questions posées à l’utilisateur. Vous pouvez simplement envoyer à l’utilisateur un message standard par le biais de la méthode _send activity_ de l’objet de [contexte de tour](bot-builder-concept-activity-processing.md#turn-context) pour demander une entrée sous forme de chaîne. Toutefois, le Kit de développement logiciel (SDK) Bot Builder fournit une bibliothèque de **dialogues** qui vous permet de demander différents types d’informations. Cette rubrique explique comment utiliser des **invites** pour demander à un utilisateur d’effectuer une saisie.

Cet article décrit comment utiliser des invites au sein d’un dialogue. Pour plus d’informations sur l’utilisation de dialogues en général, consultez l’article [Gérer un flux de conversation simple avec des dialogues](bot-builder-dialog-manage-conversation-flow.md).

## <a name="prompt-types"></a>Types d’invites

La bibliothèque de dialogues propose divers types d’invites, chacun demandant un type de réponse différent.

| Prompt | Description |
|:----|:----|
| **AttachmentPrompt** | Invite l’utilisateur à ajouter une pièce jointe comme un document ou une image. |
| **ChoicePrompt** | Invite l’utilisateur à choisir parmi un ensemble d’options. |
| **ConfirmPrompt** | Invite l’utilisateur à confirmer son action. |
| **DatetimePrompt** | Invite l’utilisateur à saisir une date/heure. Les utilisateurs puissent répondre à l’aide d’un langage naturel, par exemple « demain à 20 h 00 » ou « vendredi à 10 h ». Le Kit SDK Bot Framework utilise l’entité prédéfinie LUIS `builtin.datetimeV2`. Pour plus d’informations, consultez [builtin.datetimev2](https://docs.microsoft.com/azure/cognitive-services/luis/luis-reference-prebuilt-entities#builtindatetimev2). |
| **NumberPrompt** | Invite l’utilisateur à saisir une valeur numérique. L’utilisateur peut répondre par « 10 » ou « dix ». Si la réponse est « dix », par exemple, l’invite convertit la réponse en une valeur numérique et renvoie `10` comme résultat. |
| **TextPrompt** | Invite l’utilisateur à saisir une chaîne de texte. |

## <a name="add-references-to-prompt-library"></a>Ajouter des références à la bibliothèque d’invites

Vous pouvez accéder à la bibliothèque de **dialogues** en ajoutant le package **dialogs** à votre bot. Nous décrivons les dialogues dans l’article [Gérer un flux de conversation simple avec des dialogues](bot-builder-dialog-manage-conversation-flow.md), mais nous allons utiliser des dialogues pour nos invites.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Installez le package **Microsoft.Bot.Builder.Dialogs** à partir de NuGet.

Incluez ensuite une référence à la bibliothèque à partir du code de votre bot.

```cs
using Microsoft.Bot.Builder.Dialogs;
```

Vous pouvez définir un dialogue comme une classe ou en ligne comme une propriété dans le fichier du code de votre bot.

Le code dans cet article est écrit pour un dialogue défini comme une classe.
Les exemples suivants supposent que vous ajoutez du code au constructeur du dialogue.

Le flux principal de votre dialogue représente sa collection d’étapes et doit recevoir un ID. Comme votre bot utilise cet ID pour récupérer le dialogue, il est conseillé de l’exposer comme une constante.

```cs
public class MyDialog : DialogSet
{
    public const string Name = "mainDialog";

    public MyDialog()
    {
        // Define your dialog's prompts and steps here.
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Installez le package de dialogues à partir de NPM :

```cmd
npm install --save botbuilder-dialogs@preview
```

Pour utiliser des **dialogues** dans votre bot, incluez-les dans le code de celui-ci.

Dans le fichier app.js, ajoutez ce qui suit.

```javascript
const {DialogSet} = require("botbuilder-dialogs");
const dialogs = new DialogSet();
```

---

## <a name="prompt-the-user"></a>Inviter l'utilisateur

Pour inviter un utilisateur à effectuer une saisie, vous pouvez ajouter une invite à votre dialogue. Par exemple, vous pouvez définir une invite de type **TextPrompt** et lui attribuer un ID de dialogue de type **textPrompt** :

Quand un dialogue d’invite est ajouté, vous pouvez l’utiliser dans un simple dialogue en cascade à deux étapes, ou utilisez plusieurs invites dans une cascade à plusieurs étapes. Un dialogue *en cascade* est un moyen simple de définir une séquence d’étapes. Pour plus d’informations, consultez la section [Utilisation de dialogues](bot-builder-dialog-manage-conversation-flow.md#using-dialogs-to-guide-the-user-through-steps) de l’article [Gérer un flux de conversation simple avec des dialogues](bot-builder-dialog-manage-conversation-flow.md).

La première fois, le dialogue invite l’utilisateur à saisir son nom, et la deuxième fois, le dialogue traite l’entrée utilisateur comme une réponse à l’invite.

Par exemple, le dialogue invite l’utilisateur à saisir son nom, puis l’accueille en affichant ce nom :

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Chaque invite que vous utilisez dans votre dialogue reçoit également un nom, utilisé par le dialogue ou votre bot pour accéder à l’invite. Dans tous ces exemples, nous exposons les ID d’invites comme des constantes.

Un appel à la méthode **Prompt** ou **End** du contexte du dialogue lui signale la fin de l’étape. Sans ces instructions, le dialogue ne s’exécutera pas correctement.

```csharp
/// <summary>Defines a simple greeting dialog that asks for the user's name.</summary>
public class MyDialog : DialogSet
{
    /// <summary>The ID of the main dialog in the set.</summary>
    public const string Name = "mainDialog";

    /// <summary>Defines the IDs of the prompts in the set.</summary>
    public struct Inputs
    {
        /// <summary>The ID of the text prompt.</summary>
        public const string Text = "textPrompt";
    }

    /// <summary>Defines the prompts and steps of the dialog.</summary>
    public MyDialog()
    {
        Add(Inputs.Text, new TextPrompt());
        Add(Name, new WaterfallStep[]
        {
            // Each step takes in a dialog context, arguments, and the next delegate.
            async (dc, args, next) =>
            {
                // Prompt for the user's name.
                await dc.Prompt(Inputs.Text, "What is your name?");
            },
            async(dc, args, next) =>
            {
                var user = (string)args["Text"];
                await dc.Context.SendActivity($"Hi {user}!");
                await dc.End();
            }
        });
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
const {TextPrompt} = require("botbuilder-dialogs");
```

```javascript
// Greet user:
// Ask for the user name and then greet them by name.
dialogs.add('textPrompt', new TextPrompt());
dialogs.add('greetings', [
    async function (dc){
        await dc.prompt('textPrompt', 'What is your name?');
    },
    async function(dc, userName){
        await dc.context.sendActivity(`Hi ${userName}!`);
        await dc.end();
    }
]);
```

---

> [!NOTE]
> Pour démarrer un dialogue, obtenez un contexte de dialogue et utilisez sa méthode _begin_. Pour plus d’informations, consultez l’article [Gérer un flux de conversation simple avec des dialogues](./bot-builder-dialog-manage-conversation-flow.md).

## <a name="reusable-prompts"></a>Invites réutilisables

Une invite peut être réutilisée pour demander d’autres informations à l’aide du même type d’invite. Par exemple, l’exemple de code ci-dessus a défini un texte d’invite et l’a utilisé pour demander à l’utilisateur son nom. Si vous le souhaitez, vous pouvez également utiliser cette même invite pour demander à l’utilisateur une autre chaîne de texte, par exemple, « où travaillez-vous ? ».

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```cs
/// <summary>Defines a simple greeting dialog that asks for the user's name and place of work.</summary>
public class MyDialog : DialogSet
{
    /// <summary>The ID of the main dialog in the set.</summary>
    public const string Name = "mainDialog";

    /// <summary>Defines the IDs of the prompts in the set.</summary>
    public struct Inputs
    {
        /// <summary>The ID of the text prompt.</summary>
        public const string Text = "textPrompt";
    }

    /// <summary>Defines the prompts and steps of the dialog.</summary>
    public MyDialog()
    {
        Add(Inputs.Text, new TextPrompt());
        Add(Name, new WaterfallStep[]
        {
            async (dc, args, next) =>
            {
                // Prompt for the user's name.
                await dc.Prompt(Inputs.Text, "What is your name?");
            },
            async(dc, args, next) =>
            {
                var user = (string)args["Text"];

                // Ask them where they work.
                await dc.Prompt(Inputs.Text, $"Hi {user}! Where do you work?");
            },
            async(dc, args, next) =>
            {
                var workplace = (string)args["Text"];

                await dc.Context.SendActivity($"{workplace} is a cool place!");
                await dc.End();
            }
        });
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// Greet user:
// Ask for the user name and then greet them by name.
// Ask them where they work.
dialogs.add('textPrompt', new botbuilder_dialogs.TextPrompt());
dialogs.add('greetings',[
    async function (dc){
        await dc.prompt('textPrompt', 'What is your name?');
    },
    async function(dc, userName){
        await dc.context.sendActivity(`Hi ${userName}!`);

        // Ask them where they work.
        await dc.prompt('textPrompt', 'Where do you work?');
    },
    async function(dc, workPlace){
        await dc.context.sendActivity(`${workPlace} is a cool place!`);

        await dc.end();
    }
]);
```

---

Toutefois, si vous souhaitez associer l’invite à la valeur attendue qu’elle demande, vous pouvez attribuer à chaque invite une valeur *dialogId* unique. Un dialogue est ajouté avec un ID unique. En utilisant des ID différents, vous pouvez également créer plusieurs **invites** de même type. Par exemple, vous pouvez créer deux dialogues **TextPrompt** pour l’exemple ci-dessus :

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```cs
/// <summary>The ID of the main dialog in the set.</summary>
public const string Name = "mainDialog";

/// <summary>Defines the IDs of the prompts in the set.</summary>
public struct Inputs
{
    /// <summary>The ID of the name prompt.</summary>
    public const string Name = "namePrompt";

    /// <summary>The ID of the work prompt.</summary>
    public const string Work = "workPrompt";
}

/// <summary>Defines the prompts and steps of the dialog.</summary>
public MyDialog()
{
    Add(Inputs.Name, new TextPrompt());
    Add(Inputs.Work, new TextPrompt());
    Add(Name, new WaterfallStep[]
    {
        async (dc, args, next) =>
        {
            // Prompt for the user's name.
            await dc.Prompt(Inputs.Name, "What is your name?");
        },
        async(dc, args, next) =>
        {
            var user = (string)args["Text"];

            // Ask them where they work.
            await dc.Prompt(Inputs.Work, $"Hi {user}! Where do you work?");
        },
        async(dc, args, next) =>
        {
            var workplace = (string)args["Text"];

            await dc.Context.SendActivity($"{workplace} is a cool place!");
            await dc.End();
        }
    });
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
dialogs.add('namePrompt', new TextPrompt());
dialogs.add('workPlacePrompt', new TextPrompt());
```

---

Pour bien réutiliser le code, définissez une seule valeur `textPrompt` qui s’appliquera à toutes ces invites car elles demandent une chaîne de texte comme réponse. Mais la possibilité de nommer les dialogues s’avère utile lorsque vous devez valider l’entrée de l’invite. Dans ce cas, les invites peuvent utiliser une valeur **TextPrompt**, mais chacune recherchera un ensemble de valeurs différent. Voyons maintenant comment valider les réponses à des invites à l’aide d’une valeur `NumberPrompt`.

## <a name="specify-prompt-options"></a>Spécifier des options d’invite

Lorsque vous utilisez une invite au sein d’une étape de dialogue, vous pouvez également fournir des options d’invite, par exemple une chaîne de nouvelle invite.

Spécifier une chaîne de nouvelle invite est utile lorsque l’entrée utilisateur est incapable de répondre à une invite, soit parce que son format ne peut pas être analysé par l’invite, par exemple « demain » en réponse à une invite numérique, ou parce que l’entrée ne remplit pas un critère de validation.

> [!TIP]
> Lorsque vous créez une invite numérique, vous devez spécifier la culture d’entrée qui sera utilisée. L’invite numérique peut interpréter un large éventail d’entrées, par exemple « douze » ou « un quart », ainsi que « 12 » et « 0,25 ». La culture d’entrée aide l’invite à mieux interpréter l’entrée utilisateur.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Les cultures d’entrée sont définies dans une bibliothèque supplémentaire.

```csharp
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Recognizers.Text;
```

Le code suivant ajoute une invite numérique à un ensemble existant de dialogues, **dialogs**.

```csharp
dialogs.Add("numberPrompt", new NumberPrompt<int>(Culture.English));
```

Au sein d’une étape de dialogue, le code suivant invite l’utilisateur à effectuer une saisie et à fournir une chaîne de nouvelle invite si son entrée ne peut pas être interprétée comme une valeur numérique.

```csharp
await dc.Prompt("numberPrompt", "How many people are in your party?", new PromptOptions()
{
    RetryPromptString = "Sorry, please specify the number of people in your party."
});
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
const {NumberPrompt} = require("botbuilder-dialogs");
```

```javascript
await dc.prompt('numberPrompt', 'How many people in your party?', { retryPrompt: `Sorry, please specify the number of people in your party.` })
```

```javascript
dialogs.add('numberPrompt', new NumberPrompt());
```

---

En particulier, l’invite de choix nécessite des informations supplémentaires, la liste des choix offerts à l’utilisateur.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Cet exemple utilise des types provenant des espaces de noms suivants.

```csharp
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Bot.Builder.Prompts.Choices;
using Microsoft.Bot.Schema;
using Microsoft.Recognizers.Text;
using System.Collections.Generic;
```


Lorsque nous utilisons **ChoicePrompt** pour demander à l’utilisateur de choisir parmi un ensemble d’options, nous devons spécifier l’invite avec cet ensemble d’options, fournie dans un objet **ChoicePromptOptions**. Ici, nous utilisons **ChoiceFactory** pour convertir une liste d’options dans un format approprié.

Nous utilisons également une activité **SuggestedActions** en tant que nouvelle invite afin de fournir à nouveau les options d’entrée à l’utilisateur.


```csharp
/// <summary>Defines a dialog that asks for a choice of color.</summary>
public class MyDialog : DialogSet
{
    /// <summary>The ID of the main dialog in the set.</summary>
    public const string Name = "mainDialog";

    /// <summary>Defines the IDs of the prompts in the set.</summary>
    public struct Inputs
    {
        /// <summary>The ID of the color prompt.</summary>
        public const string Color = "colorPrompt";
    }

    /// <summary>The available colors to choose from.</summary>
    public List<string> Colors = new List<string> { "Green", "Blue" };

    /// <summary>Defines the prompts and steps of the dialog.</summary>
    public MyDialog()
    {
        Add(Inputs.Color, new ChoicePrompt(Culture.English));
        Add(Name, new WaterfallStep[]
        {
            async (dc, args, next) =>
            {
                // Prompt for a color. A choice prompt requires that you specify choice options.
                await dc.Prompt(Inputs.Color, "Please make a choice.", new ChoicePromptOptions()
                {
                    Choices = ChoiceFactory.ToChoices(Colors),
                    RetryPromptActivity =
                        MessageFactory.SuggestedActions(Colors, "Please choose a color.") as Activity
                });
            },
            async(dc, args, next) =>
            {
                var color = (FoundChoice)args["Value"];

                await dc.Context.SendActivity($"You chose {color.Value}.");
                await dc.End();
            }
        });
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
const {ChoicePrompt} = require("botbuilder-dialogs");
```

```javascript
dialogs.add('choicePrompt', new ChoicePrompt());
```

```javascript
// A choice prompt requires that you specify choice options.
const list = ['green', 'blue'];
await dc.prompt('choicePrompt', 'Please make a choice', list, {retryPrompt: 'Please choose a color.'});
```

---

## <a name="validate-a-prompt-response"></a>Valider la réponse à une invite

Vous pouvez valider la réponse à une invite avant de retourner la valeur valide à l’étape suivante de la **cascade**. Par exemple, pour valider une invite **NumberPrompt** au sein d’une plage de nombres compris entre **6** et **20**, vous pouvez utiliser une logique de validation similaire à ceci :

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```cs
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Recognizers.Text;
using PromptStatus = Microsoft.Bot.Builder.Prompts.PromptStatus;
```

```cs
/// <summary>Defines a dialog that asks for the number of people in a party.</summary>
public class MyDialog : DialogSet
{
    /// <summary>The ID of the main dialog in the set.</summary>
    public const string Name = "mainDialog";

    /// <summary>Defines the IDs of the prompts in the set.</summary>
    public struct Inputs
    {
        /// <summary>The ID of the party size prompt.</summary>
        public const string Size = "parytySize";
    }

    /// <summary>Defines the prompts and steps of the dialog.</summary>
    public MyDialog()
    {
        // Include a validation function for the party size prompt.
        Add(Inputs.Size, new NumberPrompt<int>(Culture.English, async (context, result) =>
        {
            if (result.Value < 6 || result.Value > 20)
            {
                result.Status = PromptStatus.OutOfRange;
            }
        }));
        Add(Name, new WaterfallStep[]
        {
            async (dc, args, next) =>
            {
                // Prompt for the party size.
                await dc.Prompt(Inputs.Size, "How many people are in your party?", new PromptOptions()
                {
                    RetryPromptString = "Please specify party size between 6 and 20."
                });
            },
            async(dc, args, next) =>
            {
                var size = (int)args["Value"];

                await dc.Context.SendActivity($"Okay, {size} people!");
                await dc.End();
            }
        });
    }
}
```

La validation peut également être encapsulée dans sa propre méthode privée puis ajoutée de cette façon.

```cs
/// <summary>Validates input for the partySize prompt.</summary>
/// <param name="context">The context object for the current turn of the bot.</param>
/// <param name="result">The recognition result from the prompt.</param>
/// <returns>An updated recognition result.</returns>
private static async Task PartySizeValidator(ITurnContext context, Int32Result result)
{
    if (result.Value < 6 || result.Value > 20)
    {
        result.Status = PromptStatus.OutOfRange;
    }
}
```

Au sein de votre dialogue, spécifiez la méthode à utiliser pour valider l’entrée.

```cs
// Include a validation function for the party size prompt.
Add(Inputs.Size, new NumberPrompt<int>(Culture.English, PartySizeValidator));
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// Customized prompts with validations
// A number prompt with validation for valid party size within a range.
dialogs.add('partySizePrompt', new botbuilder_dialogs.NumberPrompt( async (context, value) => {
    try {
        if(value < 6 ){
            throw new Error('Party size too small.');
        }
        else if(value > 20){
            throw new Error('Party size too big.')
        }
        else {
            return value; // Return the valid value
        }
    } catch (err) {
        await context.sendActivity(`${err.message} <br/>Please provide a valid number between 6 and 20.`);
        return undefined;
    }
}));
```

---

De même, si vous souhaitez valider une réponse **DatetimePrompt** pour une date et une heure ultérieures, vous pouvez utiliser une logique de validation similaire à ceci :

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```cs
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Recognizers.Text;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;
using DateTimeResult = Microsoft.Bot.Builder.Prompts.DateTimeResult;
using PromptStatus = Microsoft.Bot.Builder.Prompts.PromptStatus;
```

```cs
/// <summary>Validates input for the reservationTime prompt.</summary>
/// <param name="context">The context object for the current turn of the bot.</param>
/// <param name="result">The recognition result from the prompt.</param>
/// <returns>An updated recognition result.</returns>
private static async Task TimeValidator(ITurnContext context, DateTimeResult result)
{
    if (result.Resolution.Count == 0)
    {
        await context.SendActivity("Sorry, I did not recognize the time that you entered.");
        result.Status = PromptStatus.NotRecognized;
    }

    // Find any recognized time that is not in the past.
    var now = DateTime.Now;
    DateTime time = default(DateTime);
    var resolution = result.Resolution.FirstOrDefault(
        res => DateTime.TryParse(res.Value, out time) && time > now);

    if (resolution != null)
    {
        // If found, keep only that result.
        result.Resolution.Clear();
        result.Resolution.Add(resolution);
    }
    else
    {
        // Otherwise, flag the input as out of range.
        await context.SendActivity("Please enter a time in the future, such as \"tomorrow at 9am\"");
        result.Status = PromptStatus.OutOfRange;
    }
}
```

```csharp
Add(Inputs.Time, new DateTimePrompt(Culture.English, TimeValidator));
```

Vous trouverez d’autres exemples dans notre [référentiel d’exemples](https://github.com/Microsoft/botbuilder-dotnet).

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```JavaScript
// A date and time prompt with validation for date/time in the future.
dialogs.add('dateTimePrompt', new botbuilder_dialogs.DatetimePrompt( async (context, values) => {
    try {
        if (values.length < 0) { throw new Error('missing time') }
        if (values[0].type !== 'datetime') { throw new Error('unsupported type') }
        const value = new Date(values[0].value);
        if (value.getTime() < new Date().getTime()) { throw new Error('in the past') }
        return value;
    } catch (err) {
        await context.sendActivity(`Please enter a valid time in the future like "tomorrow at 9am".`);
        return undefined;
    }
}));
```

Vous trouverez d’autres exemples dans notre [référentiel d’exemples](https://github.com/Microsoft/botbuilder-js).

---

> [!TIP]
> Les invites de date/heure peuvent être résolues avec quelques dates différentes si l’utilisateur fournit une réponse ambiguë. Selon l’utilisation que vous comptez en faire, vous pouvez sélectionner toutes les résolutions fournies par le résultat de l’invite, au lieu de choisir simplement la première option.

Vous pouvez utiliser des techniques similaires pour valider les réponses à tout type d’invite.

## <a name="save-user-data"></a>Enregistrer les données utilisateur

Lorsque vous invitez l’utilisateur à effectuer une saisie, vous pouvez gérer cette saisie de plusieurs façons. Par exemple, vous pouvez utiliser et ignorer l’entrée, l’enregistrer dans une variable globale, dans un conteneur de stockage volatile ou en mémoire, dans un fichier, ou dans une base de données externe. Pour plus d’informations sur la façon d’enregistrer les données utilisateur, consultez [Gérer les données utilisateur](bot-builder-howto-v4-state.md).

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous savez comment inviter un utilisateur à effectuer une saisie, améliorons l’expérience utilisateur et le code du bot en gérant les différents flux de conversation via des dialogues.


