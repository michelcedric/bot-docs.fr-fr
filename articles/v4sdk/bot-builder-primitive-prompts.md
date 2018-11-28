---
title: Créer vos propres invites pour collecter des entrées utilisateur | Microsoft Docs
description: Découvrez comment gérer un flux de conversation avec des invites primitives dans le Kit de développement logiciel (SDK) Bot Builder.
keywords: flux de conversation, invites, état de conversation, état utilisateur, invites personnalisées
author: JonathanFingold
ms.author: v-jofing
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 11/20/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: e308457e9fb228dd141ec081ac3c5daa5fd54cac
ms.sourcegitcommit: 6cb37f43947273a58b2b7624579852b72b0e13ea
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/22/2018
ms.locfileid: "52288818"
---
# <a name="create-your-own-prompts-to-gather-user-input"></a>Créer vos propres invites pour collecter des entrées utilisateur

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Une conversation entre un bot et un utilisateur implique souvent une demande (invite) d’informations à l’utilisateur, l’analyse de sa réponse et l’action en fonction des informations obtenues.

Votre bot doit suivre le contexte d’une conversation pour pouvoir gérer son comportement et se rappeler des réponses aux questions précédentes. L’*état* d’un bot représente des informations qu’il suit pour répondre correctement aux messages entrants.

## <a name="prerequisites"></a>Prérequis

- Le code dans cet article est basé sur l’exemple **Demander aux utilisateurs d’effectuer une saisie**. Vous aurez besoin d’une copie de l’exemple en [ C# ](https://aka.ms/cs-primitive-prompt-sample) ou en [JS](https://aka.ms/js-primitive-prompt-sample).
- Connaissances de la [gestion de l’état](bot-builder-concept-state.md) et du fait d’[enregistrer les données d’utilisateur et de conversation](bot-builder-howto-v4-state.md).
- [Émulateur Bot Framework](https://aka.ms/Emulator-wiki-getting-started) pour tester le bot localement.

## <a name="about-the-sample-code"></a>Au sujet de l’exemple de code

Dans cet article, nous posons à l’utilisateur une série de questions, validons certaines des réponses et enregistrerons les entrées.
Nous utilisons le gestionnaire de tour du bot ainsi que les propriétés d’état de conversion et d’utilisateur pour gérer le flux de la conversation et la collection d’entrée.

1. Définir et configurer un état
1. Utilisez les propriétés d’état pour diriger la conversation
   1. Mettez à jour le gestionnaire de tour du bot.
   1. Implémentez une méthode d’assistance pour gérer la collecte de données de l’utilisateur.
   1. Implémentez des méthodes de validation pour l’entrée utilisateur.

## <a name="define-and-configure-state"></a>Définir et configurer un état

Nous devons configurer notre bot pour suivre les informations suivantes :

- le nom de l’utilisateur, l’âge et la date choisie, que nous allons définir dans l’état utilisateur.
- Ce que nous venons de demander à l’utilisateur, que nous définirons dans l’état de conversation.

Étant donné que nous ne prévoyons de déployer ce bot, nous allons configurer un état d’utilisateur et de conversation pour utiliser le _stockage mémoire_. Ci-dessous, nous décrivons certains aspects essentiels du code de configuration.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Nous définissons les types suivants.

- Une classe `UserProfile` pour les informations utilisateur qui seront collectées par le bot.
- Une classe `ConversationFlow` pour suivre les informations d’où nous en sommes dans la conversation.
- Une énumération `ConversationFlow.Question` interne pour le suivi d’où nous en sommes dans la conversation.
- Une classe `CustomPromptBotAccessors` dans laquelle regrouper les informations de gestion d’état.

La classe d’accesseurs de bot contient la gestion de notre état et les objets d’accesseur de propriété d’état et elle est envoyée au bot via une injection de dépendances dans ASP.NET Core. Dans notre bot, nous enregistrons les informations de l’accesseur de propriété d’état que vous recevez quand le bot créé chaque tour.

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Nous créons les objets de gestion d’état et nous les transférons lorsque nous créons notre bot.
Dans notre bot, nous définissons les identificateurs pour les propriétés d’état et pour le suivi d’où nous en sommes dans la conversation, nous enregistrons ensuite les objets de gestion d’état et nous créons nos accesseurs de propriété d’état dans le constructeur du bot.

---

## <a name="use-state-properties-to-direct-the-conversation"></a>Utilisez les propriétés d’état pour diriger la conversation

Une fois nos propriétés d’état configurées, nous pouvons les utiliser dans notre bot.

- Définissez notre [gestionnaire de tour](#the-bots-turn-handler) pour accéder à l’état et appeler notre méthode d’assistance.
- Implémentez une [méthode d’assistance](#filling-out-the-user-profile) pour gérer la collecte du profil utilisateur.
- Implémentez des [méthodes de validation](#parse-and-validate-input) pour analyser et valider l’entrée utilisateur.

### <a name="the-bots-turn-handler"></a>Le gestionnaire de tour du bot

Nous utilisons les accesseurs de propriété d’état pour obtenir les propriétés de notre état à partir du contexte du tour.
Si nous avons besoin de remplir le profil utilisateur, nous appelons notre méthode d’assistance, puis nous enregistrons les modifications d’état.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
{
   if (turnContext.Activity.Type == ActivityTypes.Message)
    {
        // Get the state properties from the turn context.
        ConversationFlow flow = await _accessors.ConversationFlowAccessor.GetAsync(turnContext, () => new ConversationFlow());
        UserProfile profile = await _accessors.UserProfileAccessor.GetAsync(turnContext, () => new UserProfile());

        await FillOutUserProfileAsync(flow, profile, turnContext);

        // Update state and save changes.
        await _accessors.ConversationFlowAccessor.SetAsync(turnContext, flow);
        await _accessors.ConversationState.SaveChangesAsync(turnContext);

        await _accessors.UserProfileAccessor.SetAsync(turnContext, profile);
        await _accessors.UserState.SaveChangesAsync(turnContext);
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// The bot's turn handler.
async onTurn(turnContext) {
    // This bot listens for message activities.
    if (turnContext.activity.type === ActivityTypes.Message) {
        // Get the state properties from the turn context.
        const flow = await this.conversationFlow.get(turnContext, { lastQuestionAsked: question.none });
        const profile = await this.userProfile.get(turnContext, {});

        await MyBot.fillOutUserProfile(flow, profile, turnContext);

        // Update state and save changes.
        await this.conversationFlow.set(turnContext, flow);
        await this.conversationState.saveChanges(turnContext);

        await this.userProfile.set(turnContext, profile);
        await this.userState.saveChanges(turnContext);
    }
}
```

---

### <a name="filling-out-the-user-profile"></a>Remplissage du profil utilisateur

Nous allons commencer par la collecte d’informations. Chacune d’elles fournit une interface similaire.

- La valeur de retour indique si l’entrée est une réponse valide pour cette question.
- Si la validation réussit, elle génère une valeur analysée et normalisée pour l’enregistrer.
- Si la validation échoue, elle génère un message avec lequel le bot peut redemander les informations.

 Dans la [section suivante](#parse-and-validate-input), nous allons définir les méthodes d’assistance pour l’analyse et la validation de l’entrée utilisateur.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
private static async Task FillOutUserProfileAsync(ConversationFlow flow, UserProfile profile, ITurnContext turnContext)
{
    string input = turnContext.Activity.Text?.Trim();
    string message;
    switch (flow.LastQuestionAsked)
    {
        case ConversationFlow.Question.None:
            await turnContext.SendActivityAsync("Let's get started. What is your name?");
            flow.LastQuestionAsked = ConversationFlow.Question.Name;
            break;
        case ConversationFlow.Question.Name:
            if (ValidateName(input, out string name, out message))
            {
                profile.Name = name;
                await turnContext.SendActivityAsync($"Hi {profile.Name}.");
                await turnContext.SendActivityAsync("How old are you?");
                flow.LastQuestionAsked = ConversationFlow.Question.Age;
                break;
            }
            else
            {
                await turnContext.SendActivityAsync(message ?? "I'm sorry, I didn't understand that.");
                break;
            }
        case ConversationFlow.Question.Age:
            if (ValidateAge(input, out int age, out message))
            {
                profile.Age = age;
                await turnContext.SendActivityAsync($"I have your age as {profile.Age}.");
                await turnContext.SendActivityAsync("When is your flight?");
                flow.LastQuestionAsked = ConversationFlow.Question.Date;
                break;
            }
            else
            {
                await turnContext.SendActivityAsync(message ?? "I'm sorry, I didn't understand that.");
                break;
            }
        case ConversationFlow.Question.Date:
            if (ValidateDate(input, out string date, out message))
            {
                profile.Date = date;
                await turnContext.SendActivityAsync($"Your cab ride to the airport is scheduled for {profile.Date}.");
                await turnContext.SendActivityAsync($"Thanks for completing the booking {profile.Name}.");
                await turnContext.SendActivityAsync($"Type anything to run the bot again.");
                flow.LastQuestionAsked = ConversationFlow.Question.None;
                profile = new UserProfile();
                break;
            }
            else
            {
                await turnContext.SendActivityAsync(message ?? "I'm sorry, I didn't understand that.");
                break;
            }
    }
}


```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// Manages the conversation flow for filling out the user's profile.
static async fillOutUserProfile(flow, profile, turnContext) {
    const input = turnContext.activity.text;
    let result;
    switch (flow.lastQuestionAsked) {
        // If we're just starting off, we haven't asked the user for any information yet.
        // Ask the user for their name and update the conversation flag.
        case question.none:
            await turnContext.sendActivity("Let's get started. What is your name?");
            flow.lastQuestionAsked = question.name;
            break;

        // If we last asked for their name, record their response, confirm that we got it.
        // Ask them for their age and update the conversation flag.
        case question.name:
            result = this.validateName(input);
            if (result.success) {
                profile.name = result.name;
                await turnContext.sendActivity(`I have your name as ${profile.name}.`);
                await turnContext.sendActivity('How old are you?');
                flow.lastQuestionAsked = question.age;
                break;
            } else {
                // If we couldn't interpret their input, ask them for it again.
                // Don't update the conversation flag, so that we repeat this step.
                await turnContext.sendActivity(
                    result.message || "I'm sorry, I didn't understand that.");
                break;
            }

        // If we last asked for their age, record their response, confirm that we got it.
        // Ask them for their date preference and update the conversation flag.
        case question.age:
            result = this.validateAge(input);
            if (result.success) {
                profile.age = result.age;
                await turnContext.sendActivity(`I have your age as ${profile.age}.`);
                await turnContext.sendActivity('When is your flight?');
                flow.lastQuestionAsked = question.date;
                break;
            } else {
                // If we couldn't interpret their input, ask them for it again.
                // Don't update the conversation flag, so that we repeat this step.
                await turnContext.sendActivity(
                    result.message || "I'm sorry, I didn't understand that.");
                break;
            }

        // If we last asked for a date, record their response, confirm that we got it,
        // let them know the process is complete, and update the conversation flag.
        case question.date:
            result = this.validateDate(input);
            if (result.success) {
                profile.date = result.date;
                await turnContext.sendActivity(`Your cab ride to the airport is scheduled for ${profile.date}.`);
                await turnContext.sendActivity(`Thanks for completing the booking ${profile.name}.`);
                await turnContext.sendActivity('Type anything to run the bot again.');
                flow.lastQuestionAsked = question.none;
                profile = {};
                break;
            } else {
                // If we couldn't interpret their input, ask them for it again.
                // Don't update the conversation flag, so that we repeat this step.
                await turnContext.sendActivity(
                    result.message || "I'm sorry, I didn't understand that.");
                break;
            }
    }
}
```

---

### <a name="parse-and-validate-input"></a>Analyser et valider l’entrée

Nous allons utiliser les critères suivants pour valider l’entrée.

- Le **nom** doit être une chaîne non vide. Nous allons le normaliser en supprimant les espaces.
- L’**âge** doit être une valeur comprise entre 18 et 120. Nous allons le normaliser en retournant un nombre entier.
- La **date** doit être de n’importe quelle date ou heure située au moins une heure dans le futur.
  Nous allons la normaliser en retournant uniquement la partie de date de l’entrée analysée.

> [!NOTE]
> Pour l’âge et la date d’entrée, nous utilisons les bibliothèques [Microsoft/Recognizers-Text](https://github.com/Microsoft/Recognizers-Text/) pour effectuer l’analyse initiale.
> Bien que nous fournissions des exemples de code, nous n’expliquons pas la façon dont les bibliothèques de modules de reconnaissance de texte fonctionnent et il s’agit juste d’une manière d’analyser l’entrée.
> Pour plus d’informations sur ces bibliothèques, consultez le fichier **LISEZ-MOI** du référentiel.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Ajoutez les méthodes de validation suivantes à votre bot.

```csharp
private static bool ValidateName(string input, out string name, out string message)
{
    name = null;
    message = null;

    if (string.IsNullOrWhiteSpace(input))
    {
        message = "Please enter a name that contains at least one character.";
    }
    else
    {
        name = input.Trim();
    }

    return message is null;
}

private static bool ValidateAge(string input, out int age, out string message)
{
    age = 0;
    message = null;

    // Try to recognize the input as a number. This works for responses such as "twelve" as well as "12".
    try
    {
        // Attempt to convert the Recognizer result to an integer. This works for "a dozen", "twelve", "12", and so on.
        // The recognizer returns a list of potential recognition results, if any.
        List<ModelResult> results = NumberRecognizer.RecognizeNumber(input, Culture.English);
        foreach (var result in results)
        {
            // result.Resolution is a dictionary, where the "value" entry contains the processed string.
            if (result.Resolution.TryGetValue("value", out object value))
            {
                age = Convert.ToInt32(value);
                if (age >= 18 && age <= 120)
                {
                    return true;
                }
            }
        }

        message = "Please enter an age between 18 and 120.";
    }
    catch
    {
        message = "I'm sorry, I could not interpret that as an age. Please enter an age between 18 and 120.";
    }

    return message is null;
}

private static bool ValidateDate(string input, out string date, out string message)
{
    date = null;
    message = null;

    // Try to recognize the input as a date-time. This works for responses such as "11/14/2018", "9pm", "tomorrow", "Sunday at 5pm", and so on.
    // The recognizer returns a list of potential recognition results, if any.
    try
    {
        List<ModelResult> results = DateTimeRecognizer.RecognizeDateTime(input, Culture.English);

        // Check whether any of the recognized date-times are appropriate,
        // and if so, return the first appropriate date-time. We're checking for a value at least an hour in the future.
        DateTime earliest = DateTime.Now.AddHours(1.0);
        foreach (ModelResult result in results)
        {
            // result.Resolution is a dictionary, where the "values" entry contains the processed input.
            var resolutions = result.Resolution["values"] as List<Dictionary<string, string>>;
            foreach (var resolution in resolutions)
            {
                // The processed input contains a "value" entry if it is a date-time value, or "start" and
                // "end" entries if it is a date-time range.
                if (resolution.TryGetValue("value", out string dateString)
                    || resolution.TryGetValue("start", out dateString))
                {
                    if (DateTime.TryParse(dateString, out var candidate)
                        && earliest < candidate)
                    {
                        date = candidate.ToShortDateString();
                        return true;
                    }
                }
            }
        }

        message = "I'm sorry, please enter a date at least an hour out.";
    }
    catch
    {
        message = "I'm sorry, I could not interpret that as an appropriate date. Please enter a date at least an hour out.";
    }

    return false;
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Ajoutez les méthodes de validation suivantes à votre bot.

```javascript
// Validates name input. Returns whether validation succeeded and either the parsed and normalized
// value or a message the bot can use to ask the user again.
static validateName(input) {
    const name = input && input.trim();
    return name != undefined
        ? { success: true, name: name }
        : { success: false, message: 'Please enter a name that contains at least one character.' };
};

// Validates age input. Returns whether validation succeeded and either the parsed and normalized
// value or a message the bot can use to ask the user again.
static validateAge(input) {

    // Try to recognize the input as a number. This works for responses such as "twelve" as well as "12".
    try {
        // Attempt to convert the Recognizer result to an integer. This works for "a dozen", "twelve", "12", and so on.
        // The recognizer returns a list of potential recognition results, if any.
        const results = Recognizers.recognizeNumber(input, Recognizers.Culture.English);
        let output;
        results.forEach(function (result) {
            // result.resolution is a dictionary, where the "value" entry contains the processed string.
            const value = result.resolution['value'];
            if (value) {
                const age = parseInt(value);
                if (!isNaN(age) && age >= 18 && age <= 120) {
                    output = { success: true, age: age };
                    return;
                }
            }
        });
        return output || { success: false, message: 'Please enter an age between 18 and 120.' };
    } catch (error) {
        return {
            success: false,
            message: "I'm sorry, I could not interpret that as an age. Please enter an age between 18 and 120."
        };
    }
}

// Validates date input. Returns whether validation succeeded and either the parsed and normalized
// value or a message the bot can use to ask the user again.
static validateDate(input) {
    // Try to recognize the input as a date-time. This works for responses such as "11/14/2018", "today at 9pm", "tomorrow", "Sunday at 5pm", and so on.
    // The recognizer returns a list of potential recognition results, if any.
    try {
        const results = Recognizers.recognizeDateTime(input, Recognizers.Culture.English);
        const now = new Date();
        const earliest = now.getTime() + (60 * 60 * 1000);
        let output;
        results.forEach(function (result) {
            // result.resolution is a dictionary, where the "values" entry contains the processed input.
            result.resolution['values'].forEach(function (resolution) {
                // The processed input contains a "value" entry if it is a date-time value, or "start" and
                // "end" entries if it is a date-time range.
                const datevalue = resolution['value'] || resolution['start'];
                // If only time is given, assume it's for today.
                const datetime = resolution['type'] === 'time'
                    ? new Date(`${now.toLocaleDateString()} ${datevalue}`)
                    : new Date(datevalue);
                if (datetime && earliest < datetime.getTime()) {
                    output = { success: true, date: datetime.toLocaleDateString() };
                    return;
                }
            });
        });
        return output || { success: false, message: "I'm sorry, please enter a date at least an hour out." };
    } catch (error) {
        return {
            success: false,
            message: "I'm sorry, I could not interpret that as an appropriate date. Please enter a date at least an hour out."
        };
    }
}
```

---

## <a name="test-the-bot-locally"></a>Testez le bot localement
1. Exécutez l’exemple en local sur votre machine. Si vous avez besoin d’instructions, consultez le fichier LISEZ-MOI pour l’exemple en [C# ](https://aka.ms/cs-primitive-prompt-sample) ou en [JS](https://aka.ms/js-primitive-prompt-sample).
1. Testez-le à l’aide de l’émulateur comme indiqué ci-dessous.

![primitive-prompts](media/primitive-prompts.png)

## <a name="additional-resources"></a>Ressources supplémentaires

La [bibliothèque de boîtes de dialogue](bot-builder-concept-dialog.md) fournit des classes qui automatisent de nombreux aspects de la gestion des conversations. 

## <a name="next-step"></a>Étape suivante

> [!div class="nextstepaction"]
> [Implémenter des flux de conversation séquentiels](bot-builder-dialog-manage-conversation-flow.md)
