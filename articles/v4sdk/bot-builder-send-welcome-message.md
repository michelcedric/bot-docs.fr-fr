---
title: Envoyer un message de bienvenue aux utilisateurs | Microsoft Docs
description: Découvrez comment développer votre bot pour offrir une expérience utilisateur conviviale.
keywords: vue d’ensemble, développer, expérience utilisateur, Accueil, expérience personnalisée, C#, JS, message de bienvenue, bot, salut, salutations
author: DanDev33
ms.author: v-dashel
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 02/19/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 354dcb1bf1e172609c1729690da76f3297201c0a
ms.sourcegitcommit: 05ddade244874b7d6e2fc91745131b99cc58b0d6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 02/21/2019
ms.locfileid: "56591140"
---
# <a name="send-welcome-message-to-users"></a>Envoyer un message de bienvenue aux utilisateurs

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

L’objectif principal lors de la création d’un bot est d’impliquer votre utilisateur dans une conversation utile. Pour atteindre cet objectif, l’une des meilleures méthodes consiste à s’assurer qu’à partir du moment où un utilisateur se connecte pour la première fois, il comprenne les fonctionnalités et l’utilité principales de votre bot, c’est-à-dire la raison pour laquelle il a été créé. Cet article fournit des exemples de code qui vous permettent d’accueillir les utilisateurs sur votre bot.

## <a name="prerequisites"></a>Prérequis

- Comprendre les [concepts de base des bots](bot-builder-basics.md). 
- Une copie de **l’exemple de bienvenue à l’utilisateur** en [C#](https://aka.ms/bot-welcome-sample-cs) ou [JS](https://aka.ms/bot-welcome-sample-js). Le code de l’exemple est utilisé pour expliquer comment envoyer des messages de bienvenue.

## <a name="same-welcome-for-different-channels"></a>Accueil similaire pour les différents canaux

Un message de bienvenue doit être généré lors de la première interaction de vos utilisateurs avec votre bot. Pour ce faire, vous pouvez surveiller les types d’activité de votre bot et observer les nouvelles connexions. Chaque nouvelle connexion peut générer jusqu’à deux activités de mise à jour de conversation selon le canal.

- Une lorsque le bot de l’utilisateur est connecté à la conversation.
- Une lorsque l’utilisateur rejoint la conversation.

Il est tentant de simplement générer un message de bienvenue chaque fois qu’une nouvelle mise à jour de conversation est détectée, mais cela peut produire des résultats inattendus lorsque l’accès à votre bot se fait par divers canaux.

Certains canaux créent une mise à jour de conversation lorsqu’un utilisateur se connecte initialement à ce canal et une mise à jour de conversation distincte uniquement après réception d’un message d’entrée initial de l’utilisateur. Les autres canaux génèrent ces deux activités lorsque l’utilisateur se connecte initialement au canal. Si vous surveillez simplement un événement de mise à jour de conversation et l’affichage d’un message de bienvenue sur un canal avec deux activités de mise à jour de conversation, votre utilisateur peut recevoir les éléments suivants :

![Message de bienvenue en double](./media/double_welcome_message.png)

Ce message en double peut être évité en générant un message de bienvenue initial pour le deuxième événement de mise à jour de conversation uniquement. Le deuxième événement peut être détecté quand :

- un événement de mise à jour de conversation s’est produit
- et quand un nouveau membre (utilisateur) a été ajouté à la conversation.

L’exemple de code ci-dessous surveille toute nouvelle *activité de mise à jour de conversation* et envoie un seul message de bienvenue par nouvel utilisateur rejoignant la conversation. Dès le premier événement _conversationUpdate_ de votre bot, ce dernier est ajouté en tant que _destinataire_ de l’activité pour votre canal. Le code vérifie ensuite si un membre ajouté == _turnContext.Activity.Recipient.Id_. Si la valeur est true, l’événement initial de mise à jour de conversation a été détecté et le code qui recherche les nouveaux utilisateurs connectés est ignoré.

[!INCLUDE [alert-await-send-activity](../includes/alert-await-send-activity.md)]

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

Dans l’exemple de code C#, **Startup.cs** a défini « WelcomeUserStateAccessors » comme service/singleton et a ajouté « UserState » à l’état de l’application. Nous allons maintenant les utiliser pour créer un objet d’état pour un utilisateur donné dans une conversation et son accesseur.

```csharp
/// The state object is used to keep track of various state related to a user in a conversation.
/// In this example, we are tracking if the bot has replied to customer first interaction.
public class WelcomeUserState
{
    public bool DidBotWelcomeUser { get; set; } = false;
}

/// Initializes a new instance of the <see cref="WelcomeUserStateAccessors"/> class.
public class WelcomeUserStateAccessors
{
    public WelcomeUserStateAccessors(UserState userState)
    {
        UserState = userState ?? throw new ArgumentNullException(nameof(userState));
    }

    public static string WelcomeUserName { get; } = $"{nameof(WelcomeUserStateAccessors)}.WelcomeUserState";

    public IStatePropertyAccessor<WelcomeUserState> WelcomeUserState { get; set; }

    public UserState UserState { get; }
}
```

Dans **WelcomeUserBot**, nous recherchons une mise à jour de l’activité pour voir si un nouvel utilisateur a été ajouté à la conversation, puis nous lui envoyons un message de bienvenue.

```csharp
// Messages sent to the user.
private const string WelcomeMessage = @"This is a simple Welcome Bot sample.This bot will introduce you
                                        to welcoming and greeting users. You can say 'intro' to see the
                                        introduction card. If you are running this bot in the Bot Framework
                                        Emulator, press the 'Start Over' button to simulate user joining
                                        a bot or a channel";

private const string InfoMessage = @"You are seeing this message because the bot received at least one
                                    'ConversationUpdate' event, indicating you (and possibly others)
                                    joined the conversation. If you are using the emulator, pressing
                                    the 'Start Over' button to trigger this event again. The specifics
                                    of the 'ConversationUpdate' event depends on the channel. You can
                                    read more information at:
                                        https://aka.ms/about-botframework-welcome-user";

private const string PatternMessage = @"It is a good pattern to use this event to send general greeting
                                        to user, explaining what your bot can do. In this example, the bot
                                        handles 'hello', 'hi', 'help' and 'intro. Try it now, type 'hi'";

// The bot state accessor object. Use this to access specific state properties.
private readonly WelcomeUserStateAccessors _welcomeUserStateAccessors;

// Initializes a new instance of the <see cref="WelcomeUserBot"/> class.
public WelcomeUserBot(WelcomeUserStateAccessors statePropertyAccessor)
{
    _welcomeUserStateAccessors = statePropertyAccessor ?? throw new System.ArgumentNullException("state accessor can't be null");
}

// Every conversation turn for our WelcomeUser Bot will call this method, including
// any type of activities such as ConversationUpdate or ContactRelationUpdate which
// are sent when a user joins a conversation.
// This bot doesn't use any dialogs; it's "single turn" processing, meaning a single
// request and response.
// This bot uses UserState to keep track of first message a user sends.
public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = new CancellationToken())
{
    // use state accessor to extract the didBotWelcomeUser flag
    var didBotWelcomeUser = await _welcomeUserStateAccessors.WelcomeUserState.GetAsync(turnContext, () => new WelcomeUserState());

    if (turnContext.Activity.Type == ActivityTypes.Message)
    {
        // Your bot should proactively send a welcome message to a personal chat the first time
        // (and only the first time) a user initiates a personal chat with your bot.
        if (didBotWelcomeUser.DidBotWelcomeUser == false)
        {
            didBotWelcomeUser.DidBotWelcomeUser = true;
            // Update user state flag to reflect bot handled first user interaction.
            await _welcomeUserStateAccessors.WelcomeUserState.SetAsync(turnContext, didBotWelcomeUser);
            await _welcomeUserStateAccessors.UserState.SaveChangesAsync(turnContext);

            // the channel should sends the user name in the 'From' object
            var userName = turnContext.Activity.From.Name;

            await turnContext.SendActivityAsync(
                $"You are seeing this message because this was your first message ever to this bot.",
                cancellationToken: cancellationToken);
            await turnContext.SendActivityAsync(
                $"It is a good practice to welcome the user and provide personal greeting. For example, welcome {userName}.",
                cancellationToken: cancellationToken);
        }
    }

    // Greet when users are added to the conversation.
    else if (turnContext.Activity.Type == ActivityTypes.ConversationUpdate)
    {
        if (turnContext.Activity.MembersAdded != null)
        {
            // Iterate over all new members added to the conversation
            foreach (var member in turnContext.Activity.MembersAdded)
            {
                // Greet anyone that was not the target (recipient) of this message
                // the 'bot' is the recipient for events from the channel,
                // turnContext.Activity.MembersAdded == turnContext.Activity.Recipient.Id indicates the
                // bot was added to the conversation.
                if (member.Id != turnContext.Activity.Recipient.Id)
                {
                    await turnContext.SendActivityAsync($"Hi there - {member.Name}. {WelcomeMessage}", cancellationToken: cancellationToken);
                    await turnContext.SendActivityAsync(InfoMessage, cancellationToken: cancellationToken);
                    await turnContext.SendActivityAsync(PatternMessage, cancellationToken: cancellationToken);
                }
            }
        }
    }
    else
    {
        // Default behavior for all other type of activities.
        await turnContext.SendActivityAsync($"{turnContext.Activity.Type} activity detected");
    }

    // save any state changes made to your state objects.
    await _welcomeUserStateAccessors.UserState.SaveChangesAsync(turnContext);
}
```

## <a name="javascripttabjs"></a>[JavaScript](#tab/js)

Ce code JavaScript envoie un message de bienvenue lorsqu’un utilisateur est ajouté. Pour effectuer cette opération, il faut surveiller l’activité de conversation et savoir quand un nouveau membre a été ajouté à la conversation.

```javascript
// Copyright (c) Microsoft Corporation. All rights reserved.
// Licensed under the MIT License.

// Import required Bot Framework classes.
const { ActivityTypes } = require('botbuilder');
const { CardFactory } = require('botbuilder');

// Adaptive Card content
const IntroCard = require('./resources/IntroCard.json');

// Welcomed User property name
const WELCOMED_USER = 'welcomedUserProperty';

class WelcomeBot {
    /**
     *
     * @param {UserState} User state to persist boolean flag to indicate
     *                    if the bot had already welcomed the user
     */
    constructor(userState) {
        // Creates a new user property accessor.
        // See https://aka.ms/about-bot-state-accessors to learn more about the bot state and state accessors.
        this.welcomedUserProperty = userState.createProperty(WELCOMED_USER);

        this.userState = userState;
    }
    /**
     *
     * @param {TurnContext} context on turn context object.
     */
    async onTurn(turnContext) {
        // See https://aka.ms/about-bot-activity-message to learn more about the message and other activity types.
        if (turnContext.activity.type === ActivityTypes.Message) {
            // Read UserState. If the 'DidBotWelcomedUser' does not exist (first time ever for a user)
            // set the default to false.
            const didBotWelcomedUser = await this.welcomedUserProperty.get(turnContext, false);

            // Your bot should proactively send a welcome message to a personal chat the first time
            // (and only the first time) a user initiates a personal chat with your bot.
            if (didBotWelcomedUser === false) {
                // The channel should send the user name in the 'From' object
                let userName = turnContext.activity.from.name;
                await turnContext.sendActivity('You are seeing this message because this was your first message ever sent to this bot.');
                await turnContext.sendActivity(`It is a good practice to welcome the user and provide personal greeting. For example, welcome ${ userName }.`);

                // Set the flag indicating the bot handled the user's first message.
                await this.welcomedUserProperty.set(turnContext, true);
            } else {
                // ...
            }
            // Save state changes
            await this.userState.saveChanges(turnContext);
        } else if (turnContext.activity.type === ActivityTypes.ConversationUpdate) {
            // Send greeting when users are added to the conversation.
            await this.sendWelcomeMessage(turnContext);
        } else {
            // Generic message for all other activities
            await turnContext.sendActivity(`[${ turnContext.activity.type } event detected]`);
        }
    }

    /**
     * Sends welcome messages to conversation members when they join the conversation.
     * Messages are only sent to conversation members who aren't the bot.
     * @param {TurnContext} turnContext
     */
    async sendWelcomeMessage(turnContext) {
        // Do we have any new members added to the conversation?
        if (turnContext.activity.membersAdded.length !== 0) {
            // Iterate over all new members added to the conversation
            for (let idx in turnContext.activity.membersAdded) {
                // Greet anyone that was not the target (recipient) of this message.
                // Since the bot is the recipient for events from the channel,
                // context.activity.membersAdded === context.activity.recipient.Id indicates the
                // bot was added to the conversation, and the opposite indicates this is a user.
                if (turnContext.activity.membersAdded[idx].id !== turnContext.activity.recipient.id) {
                    await turnContext.sendActivity(`Welcome to the 'Welcome User' Bot. This bot will introduce you to welcoming and greeting users.`);
                    await turnContext.sendActivity("You are seeing this message because the bot received at least one 'ConversationUpdate'" +
                                            'event,indicating you (and possibly others) joined the conversation. If you are using the emulator, ' +
                                            "pressing the 'Start Over' button to trigger this event again. The specifics of the 'ConversationUpdate' " +
                                            'event depends on the channel. You can read more information at https://aka.ms/about-botframework-welcome-user');
                    await turnContext.sendActivity(`It is a good pattern to use this event to send general greeting to user, explaining what your bot can do. ` +
                                            `In this example, the bot handles 'hello', 'hi', 'help' and 'intro. ` +
                                            `Try it now, type 'hi'`);
                }
            }
        }
    }
}

module.exports.WelcomeBot = WelcomeBot;
```

---

## <a name="discard-initial-user-input"></a>Ignorer l’entrée utilisateur initiale

Il est également important de déterminer si l’entrée utilisateur contient des informations utiles, ce qui peut également varier en fonction des canaux. Pour vérifier que votre utilisateur bénéficie d’une bonne expérience sur tous les canaux possibles, nous examinons l’indicateur d’état _didBotWelcomeUser_. Si la valeur est false, nous évitons de traiter cette entrée utilisateur initiale. Au lieu de cela, nous fournissons à l’utilisateur une invite initiale pour sa réponse. La variable _didBotWelcomeUser_ est ensuite définie avec la valeur true et notre code traite l’entrée utilisateur provenant de toutes les activités de message supplémentaires.

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = new CancellationToken())
{
    // use state accessor to extract the didBotWelcomeUser flag
    var didBotWelcomeUser = await _welcomeUserStateAccessors.WelcomeUserState.GetAsync(turnContext, () => new WelcomeUserState());

    if (turnContext.Activity.Type == ActivityTypes.Message)
    {
        if (didBotWelcomeUser.DidBotWelcomeUser == false)
        {
            // See previous code sample.
        }
        else
        {
            // This example hard-codes specific utterances. You should use LUIS or QnA for more advanced language understanding.
            var text = turnContext.Activity.Text.ToLowerInvariant();
            switch (text)
            {
                case "hello":
                case "hi":
                    await turnContext.SendActivityAsync($"You said {text}.", cancellationToken: cancellationToken);
                    break;
                case "intro":
                case "help":
                default:
                    await turnContext.SendActivityAsync(WelcomeMessage, cancellationToken: cancellationToken);
                    break;
            }
        }
    }

    // Greet when users are added to the conversation.
    // Note that all channels do not send the conversation update activity.
    // If you find that this bot works in the emulator, but does not in
    // another channel the reason is most likely that the channel does not
    // send this activity.
    else if (turnContext.Activity.Type == ActivityTypes.ConversationUpdate)
    {
        // See previous code sample.
    }

    // save any state changes made to your state objects.
    await _welcomeUserStateAccessors.UserState.SaveChangesAsync(turnContext);
}
```

## <a name="javascripttabjs"></a>[JavaScript](#tab/js)

```javascript
class MainDialog
{
    // Previous Code Sample
    async onTurn(turnContext)
    {
        // See https://aka.ms/about-bot-activity-message to learn more about the message and other activity types.
        if (turnContext.activity.type === ActivityTypes.Message) {
            // Previous Code Sample
            if (didBotWelcomeUser === false) {
                // Previous Code Sample
            } else  {
                // This example uses an exact match on user's input utterance.
                // Consider using LUIS or QnA for Natural Language Processing.
                let text = turnContext.activity.text.toLowerCase();
                switch (text) {
                case 'hello':
                case 'hi':
                    await turnContext.sendActivity(`You said "${ turnContext.activity.text }"`);
                    break;
                case 'intro':
                case 'help':
                    await turnContext.sendActivity({
                        text: 'Intro Adaptive Card',
                        attachments: [CardFactory.adaptiveCard(IntroCard)]
                    });
                    break;
                default :
                    await turnContext.sendActivity(`This is a simple Welcome Bot sample. You can say 'intro' to
                                                        see the introduction card. If you are running this bot in the Bot
                                                        Framework Emulator, press the 'Start Over' button to simulate user joining a bot or a channel`);
                }
            }
        }
       // Previous Sample Code
    }
}
```

---

## <a name="using-adaptive-card-greeting"></a>Utilisation d’un message de bienvenue sous forme de carte adaptative

L’accueil des utilisateurs peut également se faire à l’aide d’une carte adaptative. Pour en savoir plus sur les messages d’accueil sous forme de carte adaptative, consultez l’article [Ajouter des médias aux messages](./bot-builder-howto-add-media-attachments.md).

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// Sends an adaptive card greeting.
private static async Task SendIntroCardAsync(ITurnContext turnContext, CancellationToken cancellationToken)
{
    var response = turnContext.Activity.CreateReply();

    var card = new HeroCard();
    card.Title = "Welcome to Bot Framework!";
    card.Text = @"Welcome to Welcome Users bot sample! This Introduction card
                    is a great way to introduce your Bot to the user and suggest
                    some things to get them started. We use this opportunity to
                    recommend a few next steps for learning more creating and deploying bots.";
    card.Images = new List<CardImage>() { new CardImage("https://aka.ms/bf-welcome-card-image") };
    card.Buttons = new List<CardAction>()
    {
        new CardAction(ActionTypes.OpenUrl,
            "Get an overview", null, "Get an overview", "Get an overview",
            "https://docs.microsoft.com/en-us/azure/bot-service/?view=azure-bot-service-4.0"),
        new CardAction(ActionTypes.OpenUrl,
            "Ask a question", null, "Ask a question", "Ask a question",
            "https://stackoverflow.com/questions/tagged/botframework"),
        new CardAction(ActionTypes.OpenUrl,
            "Learn how to deploy", null, "Learn how to deploy", "Learn how to deploy",
            "https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-howto-deploy-azure?view=azure-bot-service-4.0"),
    };

    response.Attachments = new List<Attachment>() { card.ToAttachment() };
    await turnContext.SendActivityAsync(response, cancellationToken);
}
```

Ensuite, nous pouvons envoyer la carte à l’aide de la commande await ci-après. Entrons ce qui suit dans les bots : _switch (text) case "help"_.

```csharp
switch (text)
{
    case "hello":
    case "hi":
        await turnContext.SendActivityAsync($"You said {text}.", cancellationToken: cancellationToken);
        break;
    case "intro":
    case "help":
        await SendIntroCardAsync(turnContext, cancellationToken);
        break;
    default:
        await turnContext.SendActivityAsync(WelcomeMessage, cancellationToken: cancellationToken);
        break;
}
```

## <a name="javascripttabjs"></a>[JavaScript](#tab/js)

Tout d’abord, nous souhaitons ajouter notre carte adaptative au bot, en haut du fichier _index.js_, juste en dessous nos importations.

```javascript
// Adaptive Card content
const IntroCard = require('./Resources/IntroCard.json');
```

Ensuite, nous pouvons simplement utiliser le code ci-dessous dans la section _switch (text)_ _case "help"_ du bot pour répondre à l’invite des utilisateurs avec la carte adaptative.

```javascript
switch (text)
{
    case "hello":
    case "hi":
        await turnContext.sendActivity(`You said "${turnContext.activity.text}"`);
        break;
    case "intro":
    case "help":
        await turnContext.sendActivity({
            text: 'Intro Adaptive Card',
            attachments: [CardFactory.adaptiveCard(IntroCard)]
        });
        break;
    default :
        await turnContext.sendActivity(`This is a simple Welcome Bot sample. You can say 'intro' to 
                                        see the introduction card. If you are running this bot in the Bot 
                                        Framework Emulator, press the 'Start Over' button to simulate user joining a bot or a channel`);
}
```

---

## <a name="test-the-bot"></a>Tester le bot

Reportez-vous au fichier [LISEZ-MOI](https://aka.ms/bot-welcome-sample-cs) pour obtenir des instructions sur l’exécution et le test du bot.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Collecter les entrées utilisateur](bot-builder-prompts.md)
