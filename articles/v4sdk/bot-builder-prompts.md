---
title: Collecter les entrées utilisateur avec une invite de dialogue | Microsoft Docs
description: Découvrez comment demander aux utilisateurs de saisir une entrée à l’aide de la bibliothèque de dialogues du kit de développement logiciel (SDK) Bot Builder.
keywords: invites, invite, entrée utilisateur, dialogues, AttachmentPrompt, ChoicePrompt, ConfirmPrompt, DatetimePrompt, NumberPrompt, TextPrompt, réinviter, validation
author: JonathanFingold
ms.author: v-jofing
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 11/02/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: ec0cc5e942ed66c8683f8b0cc92ba7df2e36db42
ms.sourcegitcommit: 8b7bdbcbb01054f6aeb80d4a65b29177b30e1c20
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/14/2018
ms.locfileid: "51645669"
---
# <a name="gather-user-input-using-a-dialog-prompt"></a>Collecter les entrées utilisateur avec une invite de dialogue

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Pour interagir avec les utilisateurs, un bot collecte généralement des informations à travers des questions. Pour ce faire, vous pouvez utiliser directement la méthode d’_envoi d’activité_ de l’objet [contexte de tour](~/v4sdk/bot-builder-basics.md#defining-a-turn), puis traiter le message entrant suivant en tant que réponse. Toutefois, le SDK Bot Builder offre une bibliothèque de [dialogues](bot-builder-concept-dialog.md) qui fournit des méthodes permettant de poser des questions plus facilement et de s’assurer que la réponse correspond à un type de données spécifique ou respecte des règles de validation personnalisées. Cette rubrique explique comment y parvenir à l’aide d’objets d’invite pour demander à un utilisateur d’effectuer une saisie.

## <a name="prompt-types"></a>Types d’invites

Dans les coulisses, les invites constituent une boîte de dialogue en deux étapes. Tout d’abord, l’invite demande une entrée. Ensuite, elle retourne la valeur valide, ou redémarre depuis le début avec une nouvelle invite.

La bibliothèque de dialogues propose diverses invites de base, chacune étant utilisée pour recueillir un type de réponse différent.

| Prompt | Description | Retours |
|:----|:----|:----|
| _Invite de pièce jointe_ | Demande une ou plusieurs pièces jointes, un document ou une image par exemple. | Une collection d’objets _Pièce jointe_. |
| _Invite de choix_ | Demande un choix à partir d’un ensemble d’options. | Un objet de _choix trouvé_. |
| _Invite de confirmation_ | Demande une confirmation. | Valeur booléenne. |
| _Invite de date et d’heure_ | Demande une date et une heure. | Une collection d’objets de _résolution de date / heure_. |
| _Invite de nombre_ | Demande un nombre. | Une valeur numérique. |
| _Invite de texte_ | Demande une saisie de texte générale. | Une chaîne. |

La bibliothèque inclut également une _invite OAuth_ pour obtenir un _jeton OAuth_ permettant d’accéder à une autre application pour le compte de l’utilisateur. Pour plus d’informations sur l’authentification, consultez Comment [ajouter l’authentification à votre bot](bot-builder-authentication.md).

Les invites de base peuvent interpréter une entrée de langage naturel, comme « dix » ou « une dizaine » pour un nombre, ou « demain » ou un « vendredi à 10 h » pour une date-heure.

## <a name="using-prompts"></a>Utilisation des invites

Une boîte de dialogue peut utiliser une invite de commandes uniquement si la boîte de dialogue et l’invite de commandes se situent dans le même ensemble de boîte de dialogue.

1. Définissez un accesseur de propriété d’état pour l’état de votre boîte de dialogue.
1. Créez un jeu de dialogues.
1. Créez vos invites et ajoutez-les à l’ensemble de boîte de dialogue.
1. Créez un dialogue utilisant vos invites, puis ajoutez-le à l’ensemble de dialogues.
1. Dans la boîte de dialogue, ajoutez les appels pour les invites et pour récupérer les résultats des invites.

Cet article explique comment créer vos invites et comment les appeler à partir d’une boîte de dialogue en cascade.
Pour plus d’informations sur les boîtes de dialogue en général, consultez l’article [Bibliothèque de boîtes de dialogue](bot-builder-concept-dialog.md).
Pour une description d’un bot complet utilisant des invites et des boîtes de dialogue, consultez comment [utiliser des boîtes de dialogue pour gérer des flux de conversations simples](bot-builder-dialog-manage-conversation-flow.md).

Vous pouvez utiliser la même invite à plusieurs étapes au sein d’une boîte de dialogue et dans plusieurs boîtes de dialogue au sein du même ensemble de boîte de dialogue.
Toutefois, vous associez une validation personnalisée à une invite de commandes au moment de l’initialisation.
Par conséquent, si vous avez besoin d’une validation différente pour le même type d’invite, vous avez besoin de plusieurs instances du type d’invite, chacun avec son propre code de validation.

### <a name="create-a-prompt"></a>Créer une invite

Pour inviter un utilisateur à saisir une entrée, définissez une invite à l’aide de l’une des classes intégrées, _text prompt_, par exemple, et ajoutez-la à votre ensemble de dialogues.

* L’invite a un ID fixe. (Les identificateurs doivent être uniques au sein d’un ensemble de boîte de dialogue.)
* L’invite peut avoir un validateur personnalisé. (Voir [validation personnalisée](#custom-validation).)
* Pour certaines invites, vous pouvez spécifier des _paramètres régionaux par défaut_.

En règle générale, créez et ajoutez des invites et des boîtes de dialogue à votre ensemble de boîte de dialogue lorsque vous initialisez votre bot. L’ensemble de boîte de dialogue peut ensuite résoudre les ID de l’invite lorsque le bot reçoit une entrée de la part de l’utilisateur.

Par exemple, le code suivant crée deux invites de texte et les ajoute à un ensemble de boîte de dialogue existant. La seconde invite de texte fait référence à une méthode de validation qui n’est pas affichée ici.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Ici, `_dialogs` contient un ensemble existant de boîte de dialogue, et `NameValidator` est une méthode de validation.

```csharp
_dialogs.Add(new TextPrompt("nickNamePrompt"));
_dialogs.Add(new TextPrompt("namePrompt", NameValidator));
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Ici, `this.dialogs` contient un ensemble existant de boîte de dialogue, et `NameValidator` est une fonction de validation.

```javascript
this.dialogs.add(new TextPrompt('nickNamePrompt'));
this.dialogs.add(new TextPrompt('namePrompt', NameValidator));
```

---

#### <a name="locales"></a>Locales

Les paramètres régionaux sont utilisés pour déterminer le comportement spécifique à la langue des invites **choix**, **confirmer**, **date-heure**, et **nombre**. Pour une entrée venant de l’utilisateur :

* Si le canal fournit une propriété _paramètres régionaux_ dans le message de l’utilisateur, elle est utilisée.
* Sinon, les _paramètres régionaux par défaut_ de l’invite sont utilisés s’ils sont définis, soit en les fournissant lors de l’appel du constructeur de l’invite ou bien en les définissant ultérieurement.
* Sinon, l’anglais (« en-us ») est utilisé comme paramètre régional.

> [!NOTE]
> Les paramètres régionaux sont des codes ISO 639 de 2, 3 ou 4 caractères qui représentent une langue ou une famille de langues.

### <a name="call-a-prompt-from-a-waterfall-dialog"></a>Appeler une invite de commandes à partir d’une boîte de dialogue en cascade

Une fois qu’une invite est ajoutée, appelez-la dans une seule étape d’une boîte de dialogue en cascade et obtenez le résultat de l’invite à l’étape suivante de la boîte de dialogue.
Pour appeler une invite à partir d’une étape en cascade, appelez la méthode _invite_ du _contexte de l’étape en cascade_ de l’objet. Le premier paramètre est l’ID de l’invite à utiliser, et le deuxième paramètre contient les options pour l’invite, comme le texte utilisé pour demander une entrée à l’utilisateur.

Supposons que l’utilisateur interagit avec un bot, le bot possède une boîte de dialogue en cascade active, et l’étape suivante dans la boîte de dialogue utilise une invite.

1. Lorsque l’utilisateur envoie un message au bot, il effectue les opérations suivantes :
   1. Le gestionnaire de tour du bot crée le contexte d’une boîte de dialogue et appelle sa méthode _continue_.
   1. Le contrôle passe à l’étape suivante dans la boîte de dialogue active, votre boîte de dialogue en cascade dans ce cas précis.
   1. L’étape appelle la méthode _prompt_ du contexte d’étape en cascade pour demander une entrée à l’utilisateur.
   1. Le contexte d’étape en cascade pousse l’invite dans la pile et la démarre.
   1. L’invite envoie une activité à l’utilisateur pour demander son entrée.
1. Lorsque l’utilisateur envoie son prochain message au bot, il effectue les opérations suivantes :
   1. Le gestionnaire de tour du bot crée le contexte d’une boîte de dialogue et appelle sa méthode _continue_.
   1. Le contrôle passe à l’étape suivante dans la boîte de dialogue active, le second tour de l’invite.
   1. L’invite valide l’entrée de l’utilisateur.
      * Si son entrée n’est pas valide, l’invite est redémarrée, provoquant une nouvelle demande d’entrée, et cet ensemble d’étapes se répète au prochain tour.
      * Sinon, l’invite se termine et retourne un objet _dialog turn result_ à la boîte de dialogue parente. Le contrôle passe à l’étape suivante de votre boîte de dialogue en cascade, avec le résultat de l’invite de commandes disponible dans la propriété _résultat_ du contexte d’étape en cascade.

<!--
> [!NOTE]
> A waterfall step delegate takes a _waterfall step context_ parameter and returns a _dialog turn result_.
> A prompt's result is contained within the prompt's return value (a dialog turn result object) when it ends.
> The waterfall dialog provides the return value in the waterfall step context parameter when it calls the next waterfall step.
-->

Quand une invite de commandes est retournée, la propriété _résultat_ du contexte d’étape en cascade est définie sur la valeur renvoyée de l’invite.

Cet exemple illustre les parties de deux étapes consécutives en cascade. La première utilise l’invite pour demander le nom de l’utilisateur. La seconde obtient la valeur renvoyée de l’invite.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Ici, `name` est l’ID d’une invite de texte, `NameStepAsync` et `GreetingStepAsync` sont deux délégués d’étapes consécutives d’une boîte de dialogue en cascade.

```csharp
private static async Task<DialogTurnResult> NameStepAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
{
    // ...

    // Prompt for the user's name.
    return await stepContext.PromptAsync(
        "name",
         new PromptOptions { Prompt = MessageFactory.Text("Please enter your name.") },
         cancellationToken);
}

private static async Task<DialogTurnResult> GreetingStepAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
{
    // Get the user's name from the prompt result.
    string name = (string)stepContext.Result;
    await stepContext.Context.SendActivityAsync(
        MessageFactory.Text($"Pleased to meet you, {name}."),
         cancellationToken);

    // ...
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Ici, `name` est l’ID d’une invite de texte, `nameStep` et `greetingStep` sont deux fonctions d’étapes consécutives d’une boîte de dialogue en cascade.

```javascript
async nameStep(step) {
    // ...

    return await step.prompt('name', 'Please enter your name.');
}

async greetingStep(step) {
    // Get the user's name from the prompt result.
    const name = step.result;
    await step.context.sendActivity(`Pleased to meet you, ${name}.`);

    // ...
}
```

---

### <a name="call-a-prompt-from-the-bots-turn-handler"></a>Appeler une invite de commandes à partir du gestionnaire de tour du bot

Il est possible d’appeler une invite directement à partir de votre gestionnaire de tour, à l’aide de la méthode _prompt_ de contexte de boîte de dialogue.
Vous devez appeler la méthode _continue dialog_ de contexte de boîte de dialogue lors du prochain tour et examiner sa valeur renvoyée, un objet _dialog turn result_. Pour obtenir un exemple de la procédure à suivre, consultez l’exemple de validations d’invite ([C#](https://aka.ms/cs-prompt-validation-sample) | [JS](https://aka.ms/js-prompt-validation-sample)), ou examinez comment [demander à l’utilisateur d’effectuer une saisie à l’aide de vos propres invites](bot-builder-primitive-prompts.md) pour une autre approche.

## <a name="prompt-options"></a>Options d’invite

Le deuxième paramètre de la méthode _prompt_ prend un objet _prompt options_, qui a les propriétés suivantes.

| Propriété | Description |
| :--- | :--- |
| _prompt_ | L’activité initiale à envoyer à l’utilisateur, pour demander son entrée. |
| _retry prompt_ | L’activité à envoyer à l’utilisateur si sa première entrée n’a pas été validée. |
| _choix_ | Une liste des choix que l’utilisateur peut sélectionner, pour une utilisation avec une invite de choix. |

En règle générale, les propriétés prompt et retry prompt sont des activités, bien qu’il existe des variantes sur la façon dont cela est géré par les différents langages de programmation.

Vous devez toujours spécifier l’activité prompt initiale à envoyer à l’utilisateur.

Spécifier une nouvelle invite est utile lorsque l’entrée de l’utilisateur est incapable de valider, soit parce que son format ne peut pas être analysé par l’invite, par exemple « demain » en réponse à une invite numérique, ou parce que l’entrée ne remplit pas un critère de validation. Dans ce cas, si aucune nouvelle invite n’a été fournie, l’invite utilisera l’activité prompt initiale pour redemander une saisie à l’utilisateur.

Pour une invite de choix, vous devez toujours fournir la liste des choix disponibles.

Cet exemple montre comment utiliser une invite de choix, en fournissant les trois propriétés. La méthode _favorite color_ est utilisée comme une étape dans une boîte de dialogue en cascade et notre ensemble de boîte de dialogue contient la boîte de dialogue en cascade et une invite de choix avec un ID de `colorChoice`.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
private static async Task<DialogTurnResult> FavoriteColorAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
{
    // ...

    return await stepContext.PromptAsync(
        "colorChoice",
        new PromptOptions {
            Prompt = MessageFactory.Text("Please choose a color."),
            RetryPrompt = MessageFactory.Text("Sorry, please choose a color from the list."),
            Choices = ChoiceFactory.ToChoices(new List<string> { "blue", "green", "red" }),
        },
        cancellationToken);
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Dans le kit de développement logiciel (SDK) JavaScript, vous pouvez fournir une chaîne pour les propriétés `prompt` et `retryPrompt`. L’invite les convertit en activités de message pour vous.

```javascript
async favoriteColor(step) {
    // ...

    return await step.prompt('colorChoice', {
        prompt: 'Please choose a color:',
        retryPrompt: 'Sorry, please choose a color from the list.',
        choices: [ 'red', 'green', 'blue' ]
    });
}
```

---

## <a name="custom-validation"></a>Validation personnalisée

Vous pouvez valider la réponse à une invite avant de retourner la valeur à l’étape suivante de la **cascade**. Une fonction du validateur possède un paramètre de _contexte de validateur d’invite_ et retourne une valeur booléenne indiquant si l’entrée passe la validation.

Le contexte de validateur d’invite inclut les propriétés suivantes :

| Propriété | Description |
| :--- | :--- |
| _Contexte_ | Le contexte de tour actuel pour le bot. |
| _Reconnu_ | Un _résultat du module de reconnaissance de l’invite_ qui contient des informations sur l’entrée de l’utilisateur, comme traitées par le module de reconnaissance. |

Le résultat du module de reconnaissance de l’invite a les propriétés suivantes :

| Propriété | Description |
| :--- | :--- |
| _Réussi_ | Indique si le module de reconnaissance a été en mesure d’analyser l’entrée. |
| _Valeur_ | La valeur renvoyée du module de reconnaissance. Si nécessaire, le code de validation peut modifier cette valeur. |

### <a name="setup"></a>Paramétrage

Nous devons faire une légère configuration avant d’ajouter notre code de validation.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Dans le fichier **.cs** de votre bot, définissez une classe interne pour les informations de réservation.

```csharp
public class Reservation
{
    public int Size { get; set; }

    public string Date { get; set; }
}
```

Dans **BotAccessors.cs**, ajoutez un accesseur de propriété d’état pour les données de réservation.

```csharp
public class BotAccessors
{
    public BotAccessors(ConversationState conversationState)
    {
        ConversationState = conversationState ?? throw new ArgumentNullException(nameof(conversationState));
    }

    public static string DialogStateAccessorKey { get; } = "BotAccessors.DialogState";
    public static string ReservationAccessorKey { get; } = "BotAccessors.Reservation";

    public IStatePropertyAccessor<DialogState> DialogStateAccessor { get; set; }
    public IStatePropertyAccessor<ReservationBot.Reservation> ReservationAccessor { get; set; }

    public ConversationState ConversationState { get; }
}
```

Dans **Startup.cs**, mettez à jour `ConfigureServices` pour définir les accesseurs.

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // ...

    // Create and register state accesssors.
    // Acessors created here are passed into the IBot-derived class on every turn.
    services.AddSingleton<BotAccessors>(sp =>
    {
        // ...

        // Create the custom state accessor.
        // State accessors enable other components to read and write individual properties of state.
        var accessors = new BotAccessors(conversationState)
        {
            DialogStateAccessor = conversationState.CreateProperty<DialogState>(BotAccessors.DialogStateAccessorKey),
            ReservationAccessor = conversationState.CreateProperty<ReservationBot.Reservation>(BotAccessors.ReservationAccessorKey),
        };

        return accessors;
    });
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Aucune modification du code du service HTTP n’est exigée pour JavaScript, nous pouvons laisser notre fichier **index.js** en l’état.

Dans **bot.js**, mettez à jour les instructions obligatoires et ajoutez des identificateurs pour les accesseurs de la propriété d’état.

```javascript
const { ActivityTypes } = require('botbuilder');
const { DialogSet, WaterfallDialog, NumberPrompt, DateTimePrompt, DialogTurnStatus } = require('botbuilder-dialogs');

// Define identifiers for our state property accessors.
const DIALOG_STATE_ACCESSOR = 'dialogStateAccessor';
const RESERVATION_ACCESSOR = 'reservationAccessor';
```

---

Dans le fichier de votre bot, ajoutez des identificateurs pour nos boîtes de dialogue et nos invites.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// Define identifiers for our dialogs and prompts.
private const string ReservationDialog = "reservationDialog";
private const string PartySizePrompt = "partyPrompt";
private const string ReservationDatePrompt = "reservationDatePrompt";
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// Define identifiers for our dialogs and prompts.
const RESERVATION_DIALOG = 'reservationDialog';
const PARTY_SIZE_PROMPT = 'partySizePrompt';
const RESERVATION_DATE_PROMPT = 'reservationDatePrompt';
```

---

### <a name="define-the-prompts-and-dialogs"></a>Définir les invites et les boîtes de dialogue

Dans le code du constructeur du bot, créez l’ensemble de boîte de dialogue, ajoutez les invites et ajoutez la boîte de dialogue de réservation.
Nous incluons la validation personnalisée lorsque nous créons les invites. Nous implémenterons ensuite les fonctions de validation.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
public ReservationBot(BotAccessors accessors, ILoggerFactory loggerFactory)
{
    // ...
    _accessors = accessors ?? throw new System.ArgumentNullException(nameof(accessors));

    // Create the dialog set and add the prompts, including custom validation.
    _dialogSet = new DialogSet(_accessors.DialogStateAccessor);
    _dialogSet.Add(new NumberPrompt<int>(PartySizePrompt, PartySizeValidatorAsync));
    _dialogSet.Add(new DateTimePrompt(ReservationDatePrompt, DateValidatorAsync));

    // Define the steps of the waterfall dialog and add it to the set.
    WaterfallStep[] steps = new WaterfallStep[]
    {
        PromptForPartySizeAsync,
        PromptForReservationDateAsync,
        AcknowledgeReservationAsync,
    };
    _dialogSet.Add(new WaterfallDialog(ReservationDialog, steps));
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
constructor(conversationState) {
    // Creates our state accessor properties.
    // See https://aka.ms/about-bot-state-accessors to learn more about the bot state and state accessors.
    this.dialogStateAccessor = conversationState.createProperty(DIALOG_STATE_ACCESSOR);
    this.reservationAccessor = conversationState.createProperty(RESERVATION_ACCESSOR);
    this.conversationState = conversationState;

    // Create the dialog set and add the prompts, including custom validation.
    this.dialogSet = new DialogSet(this.dialogStateAccessor);
    this.dialogSet.add(new NumberPrompt(PARTY_SIZE_PROMPT, partySizeValidator));
    this.dialogSet.add(new DateTimePrompt(RESERVATION_DATE_PROMPT, dateValidator));

    // Define the steps of the waterfall dialog and add it to the set.
    this.dialogSet.add(new WaterfallDialog(RESERVATION_DIALOG, [
        this.promptForPartySize.bind(this),
        this.promptForReservationDate.bind(this),
        this.acknowledgeReservation.bind(this),
    ]));
}
```

---

### <a name="implement-validation-code"></a>Implémenter le code de validation

Implémentez le validateur de taille de tiers. Nous allons limiter les réservations aux parties de 6 à 20 personnes.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
private async Task<bool> PartySizeValidatorAsync(
    PromptValidatorContext<int> promptContext,
    CancellationToken cancellationToken)
{
    // Check whether the input could be recognized as an integer.
    if (!promptContext.Recognized.Succeeded)
    {
        await promptContext.Context.SendActivityAsync(
            "I'm sorry, I do not understand. Please enter the number of people in your party.",
            cancellationToken: cancellationToken);
        return false;
    }

    // Check whether the party size is appropriate.
    int size = promptContext.Recognized.Value;
    if (size < 6 || size > 20)
    {
        await promptContext.Context.SendActivityAsync(
            "Sorry, we can only take reservations for parties of 6 to 20.",
            cancellationToken: cancellationToken);
        return false;
    }

    return true;
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
async partySizeValidator(promptContext) {
    // Check whether the input could be recognized as date.
    if (!promptContext.recognized.succeeded) {
        await promptContext.context.sendActivity(
            "I'm sorry, I do not understand. Please enter the number of people in your party.");
        return false;
    }
    if (promptContext.recognized.value % 1 != 0) {
        await promptContext.context.sendActivity(
            "I'm sorry, I don't understand fractional people.");
        return false;
    }
    // Check whether the party size is appropriate.
    var size = promptContext.recognized.value;
    if (size < 6 || size > 20) {
        await promptContext.context.sendActivity(
            'Sorry, we can only take reservations for parties of 6 to 20.');
        return false;
    }

    return true;
}
```

---

L’invite de date-heure retourne une liste ou un tableau des _résolutions de date-heure_ possibles correspondant à l’entrée de l’utilisateur. Par exemple, 9:00 peut vouloir dire 9 h 00 ou 21 h 00, et dimanche est également ambigu. En outre, une résolution de date-heure peut représenter une date, une heure, une date-heure ou une plage. L’invite de date-heure utilise [Microsoft/Recognizers-Text](https://github.com/Microsoft/Recognizers-Text) pour analyser l’entrée de l’utilisateur.

Implémentez le validateur de date de réservation. Nous allons limiter les réservations à une heure ou plus à partir de l’heure actuelle. Nous gardons la première résolution correspondant à nos critères et nous effaçons le reste.

Ce code de validation n’est pas exhaustif. Il convient mieux pour l’entrée qui analyse à une date et une heure. Il montre certaines des options pour la validation d’une invite de date-heure, et votre implémentation dépendra des informations que vous essayez de collecter par l’intermédiaire de l’utilisateur.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// Validates whether the reservation date is appropriate.
// Reservations must be made at least an hour in advance.
private async Task<bool> DateValidatorAsync(
    PromptValidatorContext<IList<DateTimeResolution>> promptContext,
    CancellationToken cancellationToken = default(CancellationToken))
{
    // Check whether the input could be recognized as an integer.
    if (!promptContext.Recognized.Succeeded)
    {
        await promptContext.Context.SendActivityAsync(
            "I'm sorry, I do not understand. Please enter the date or time for your reservation.",
            cancellationToken: cancellationToken);
        return false;
    }

    // Check whether any of the recognized date-times are appropriate,
    // and if so, return the first appropriate date-time.
    DateTime earliest = DateTime.Now.AddHours(1.0);
    DateTimeResolution value = promptContext.Recognized.Value.FirstOrDefault(v =>
        DateTime.TryParse(v.Value ?? v.Start, out DateTime time) && DateTime.Compare(earliest,time) <= 0);
    if (value != null)
    {
        promptContext.Recognized.Value.Clear();
        promptContext.Recognized.Value.Add(value);
        return true;
    }

    await promptContext.Context.SendActivityAsync(
            "I'm sorry, we can't take reservations earlier than an hour from now.",
            cancellationToken: cancellationToken);
    return false;
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
async dateValidator(promptContext) {
// Check whether the input could be recognized as an integer.
if (!promptContext.recognized.succeeded) {
    await promptContext.context.sendActivity(
        "I'm sorry, I do not understand. Please enter the date or time for your reservation.");
    return false;
}

// Check whether any of the recognized date-times are appropriate,
// and if so, return the first appropriate date-time.
const earliest = Date.now() + (60 * 60 * 1000);
let value = null;
promptContext.recognized.value.forEach(candidate => {
    // TODO: update validation to account for time vs date vs date-time vs range.
    const time = new Date(candidate.value || candidate.start);
    if (earliest < time.getTime()) {
        value = candidate;
    }
});
if (value) {
    promptContext.recognized.value = [value];
    return true;
}

await promptContext.context.sendActivity(
    "I'm sorry, we can't take reservations earlier than an hour from now.");
return false;
}
```

---

### <a name="implement-the-dialog-steps"></a>Implémenter les étapes de boîte de dialogue

Utilisez les invites que nous avons ajoutées à l’ensemble de boîte de dialogue. Nous avons ajouté la validation aux invites quand nous les avons créées dans le constructeur du bot. La première fois que l’invite demande une saisie de l’utilisateur, il envoie l’activité _prompt_ parmi les options fournies. Si la validation échoue, il envoie l’activité _retry prompt_ pour demander une entrée différente à l’utilisateur.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// First step of the main dialog: prompt for party size.
private async Task<DialogTurnResult> PromptForPartySizeAsync(
    WaterfallStepContext stepContext,
    CancellationToken cancellationToken = default(CancellationToken))
{
    // Prompt for the party size. The result of the prompt is returned to the next step of the waterfall.
    return await stepContext.PromptAsync(
        PartySizePrompt,
        new PromptOptions
        {
            Prompt = MessageFactory.Text("How many people is the reservation for?"),
            RetryPrompt = MessageFactory.Text("How large is your party?"),
        },
        cancellationToken);
}

// Second step of the main dialog: record the party size and prompt for the
// reservation date.
private async Task<DialogTurnResult> PromptForReservationDateAsync(
    WaterfallStepContext stepContext,
    CancellationToken cancellationToken = default(CancellationToken))
{
    // Record the party size information in the current dialog state.
    int size = (int)stepContext.Result;
    stepContext.Values["size"] = size;

    // Prompt for the reservation date. The result of the prompt is returned to the next step of the waterfall.
    return await stepContext.PromptAsync(
        ReservationDatePrompt,
        new PromptOptions
        {
            Prompt = MessageFactory.Text("Great. When will the reservation be for?"),
            RetryPrompt = MessageFactory.Text("What time should we make your reservation for?"),
        },
        cancellationToken);
}

// Third step of the main dialog: return the collected party size and reservation date.
private async Task<DialogTurnResult> AcknowledgeReservationAsync(
    WaterfallStepContext stepContext,
    CancellationToken cancellationToken = default(CancellationToken))
{
    // Retrieve the reservation date.
    DateTimeResolution resolution = (stepContext.Result as IList<DateTimeResolution>).First();
    string time = resolution.Value ?? resolution.Start;

    // Send an acknowledgement to the user.
    await stepContext.Context.SendActivityAsync(
        "Thank you. We will confirm your reservation shortly.",
        cancellationToken: cancellationToken);

    // Return the collected information to the parent context.
    Reservation reservation = new Reservation
    {
        Date = time,
        Size = (int)stepContext.Values["size"],
    };
    return await stepContext.EndDialogAsync(reservation, cancellationToken);
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
async promptForPartySize(stepContext) {
    // Prompt for the party size. The result of the prompt is returned to the next step of the waterfall.
    return await stepContext.prompt(
        PARTY_SIZE_PROMPT, {
            prompt: 'How many people is the reservation for?',
            retryPrompt: 'How large is your party?'
        });
}

async promptForReservationDate(stepContext) {
    // Record the party size information in the current dialog state.
    stepContext.values.size = stepContext.result;

    // Prompt for the reservation date. The result of the prompt is returned to the next step of the waterfall.
    return await stepContext.prompt(
        RESERVATION_DATE_PROMPT, {
            prompt: 'Great. When will the reservation be for?',
            retryPrompt: 'What time should we make your reservation for?'
        });
}

async acknowledgeReservation(stepContext) {
    // Retrieve the reservation date.
    const resolution = stepContext.result[0];
    const time = resolution.value || resolution.start;

    // Send an acknowledgement to the user.
    await stepContext.context.sendActivity(
        'Thank you. We will confirm your reservation shortly.');

    // Return the collected information to the parent context.
    return await stepContext.endDialog({ date: time, size: stepContext.values.size });
}
```

---

### <a name="update-the-turn-handler"></a>Mettre à jour le gestionnaire de tour

Mettez à jour le gestionnaire de tour du bot pour démarrer la boîte de dialogue et acceptez une valeur renvoyée à partir de la boîte de dialogue lorsqu’elle est terminée.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
{
    switch (turnContext.Activity.Type)
    {
        // On a message from the user:
        case ActivityTypes.Message:

            // Get the current reservation info from state.
            Reservation reservation = await _accessors.ReservationAccessor.GetAsync(
                turnContext, () => null, cancellationToken);

            // Generate a dialog context for our dialog set.
            DialogContext dc = await _dialogSet.CreateContextAsync(turnContext, cancellationToken);

            if (dc.ActiveDialog is null)
            {
                // If there is no active dialog, check whether we have a reservation yet.
                if (reservation is null)
                {
                    // If not, start the dialog.
                    await dc.BeginDialogAsync(ReservationDialog, null, cancellationToken);
                }
                else
                {
                    // Otherwise, send a status message.
                    await turnContext.SendActivityAsync(
                        $"We'll see you {reservation.Date}.",
                        cancellationToken: cancellationToken);
                }
            }
            else
            {
                // Continue the dialog.
                DialogTurnResult dialogTurnResult = await dc.ContinueDialogAsync(cancellationToken);

                // If the dialog completed this turn, record the reservation info.
                if (dialogTurnResult.Status is DialogTurnStatus.Complete)
                {
                    reservation = (Reservation)dialogTurnResult.Result;
                    await _accessors.ReservationAccessor.SetAsync(
                        turnContext,
                        reservation,
                        cancellationToken);

                    // Send a confirmation message to the user.
                    await turnContext.SendActivityAsync(
                        $"Your party of {reservation.Size} is confirmed for {reservation.Date}.",
                        cancellationToken: cancellationToken);
                }
            }

            // Save the updated dialog state into the conversation state.
            await _accessors.ConversationState.SaveChangesAsync(turnContext, false, cancellationToken);
            break;

        // Handle other incoming activity types as appropriate to your bot.
        default:
            await turnContext.SendActivityAsync($"{turnContext.Activity.Type} event detected");
            break;
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
async onTurn(turnContext) {
    switch (turnContext.activity.type) {
        case ActivityTypes.Message:
            // Get the current reservation info from state.
            const reservation = await this.reservationAccessor.get(turnContext, null);

            // Generate a dialog context for our dialog set.
            const dc = await this.dialogSet.createContext(turnContext);

            if (!dc.activeDialog) {
                // If there is no active dialog, check whether we have a reservation yet.
                if (!reservation) {
                    // If not, start the dialog.
                    await dc.beginDialog(RESERVATION_DIALOG);
                }
                else {
                    // Otherwise, send a status message.
                    await turnContext.sendActivity(
                        `We'll see you ${reservation.date}.`);
                }
            }
            else {
                // Continue the dialog.
                const dialogTurnResult = await dc.continueDialog();

                // If the dialog completed this turn, record the reservation info.
                if (dialogTurnResult.status === DialogTurnStatus.complete) {
                    await this.reservationAccessor.set(
                        turnContext,
                        dialogTurnResult.result);

                    // Send a confirmation message to the user.
                    await turnContext.sendActivity(
                        `Your party of ${dialogTurnResult.result.size} is ` +
                        `confirmed for ${dialogTurnResult.result.date}.`);
                }
            }

            // Save the updated dialog state into the conversation state.
            await this.conversationState.saveChanges(turnContext, false);
            break;
        default:
            break;
    }
}
```

---

Vous pouvez utiliser des techniques similaires pour valider les réponses à tout type d’invite.

## <a name="handling-prompt-results"></a>Gestion des résultats des invites

L’action que vous effectuez avec le résultat de l’invite dépend de la raison pour laquelle vous avez demandé les informations à l’utilisateur. Options disponibles :

* Utilisez les informations pour contrôler le flux de votre boîte de dialogue, par exemple lorsque l’utilisateur répond à une invite de confirmation ou de choix.
* Mettez en cache les informations contenues dans l’état de la boîte de dialogue, comme la définition d’une valeur dans la propriété _valeurs_ du contexte d’étape en cascade, puis renvoyez ensuite les informations collectées lorsque la boîte de dialogue se termine.
* Enregistrez les informations dans l’état du bot. Cela vous oblige à concevoir votre boîte de dialogue de façon à avoir accès aux accesseurs de la propriété d’état du bot.

## <a name="additional-resources"></a>Ressources supplémentaires
Pour en savoir plus sur les invites multiples, consultez les exemples en [[C#](https://aka.ms/cs-multi-prompts-sample) ou [JS](https://aka.ms/js-multi-prompts-sample)].

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Implémenter des flux de conversation séquentiels de base](bot-builder-dialog-manage-conversation-flow.md)
