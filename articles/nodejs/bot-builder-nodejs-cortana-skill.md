---
title: Créer un bot à fonctionnalité vocale avec des compétences Cortana | Microsoft Docs
description: Découvrez comment créer un bot à fonctionnalité vocale avec des compétences Cortana et le Kit SDK Bot Builder pour Node.js.
author: DeniseMak
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: a5936f3a622bdc91084ba9a636d8be3b782f9a11
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39300204"
---
# <a name="build-a-speech-enabled-bot-with-cortana-skills"></a>Créer un bot à fonctionnalité vocale avec des compétences Cortana
> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-cortana-skill.md)
> - [Node.JS](../nodejs/bot-builder-nodejs-cortana-skill.md)

Le Kit SDK Bot Builder pour Node.js vous permet de créer un bot à fonctionnalité vocale en le connectant au canal Cortana en tant que compétence Cortana. Les compétences Cortana vous permettent de fournir des fonctionnalités via Cortana en réponse à l’entrée vocale d’un utilisateur.

> [!TIP]
> Pour plus d’informations sur la définition d’une compétence et ses possibilités, consultez le [Kit de compétences Cortana][CortanaGetStarted].

La création d’une compétence Cortana avec Bot Framework nécessite très peu de connaissances particulières de Cortana, et se limite essentiellement à l’élaboration d’un bot. Une des différences majeures, par rapport aux autres bots que vous avez pu créer, est la présence conjointe de composants visuels et de composants audio dans Cortana. Pour le composant visuel, Cortana dispose d’une zone de canevas permettant la restitution de contenus, tels que des cartes. Quant au composant audio, vous fournissez du texte ou des données SSML dans les messages de votre bot, et Cortana en fait la lecture à l’utilisateur, donnant ainsi une voix à votre bot. 

> [!NOTE]
> Cortana est disponible sur toutes sortes d’appareils. Certains sont équipés d’un écran, tandis que d’autres, comme une enceinte acoustique, n’en sont pas forcément pourvus. Vous devez vous assurer que votre bot est capable de gérer ces deux scénarios. Consultez les [entités propres à Cortana][CortanaSpecificEntities] pour savoir comment vérifier les informations des appareils.

## <a name="adding-speech-to-your-bot"></a>Ajout de la reconnaissance vocale à votre bot

Les messages parlés à partir de votre bot sont représentés en SSML (Speech Synthesis Markup Language, SSML). Le Kit SDK Bot Builder vous permet d’inclure le SSML dans les réponses de votre bot, afin de contrôler ce qu’il dit, en plus de ce qu’il affiche.

### <a name="sessionsay"></a>session.say

Votre bot utilise la méthode **session.say** pour dialoguer avec l’utilisateur, au lieu de la méthode **session.send**. Cela inclut des paramètres facultatifs pour l’envoi d’une sortie SSML, ainsi que des pièces jointes telles que des cartes. 

Voici le format de cette méthode :

```session.say(displayText: string, speechText: string, options?: object)```

| Paramètre | Description |
|------|------|
| **displayText** | Message texte à afficher dans l’interface utilisateur Cortana.|
| **speechText** | Texte ou SSML que Cortana lit à l’utilisateur. |
| **options** | Un objet [IMessage][IMessage] qui peut contenir une pièce jointe ou un conseil de saisie. Les conseils de saisie indiquent si le bot accepte, attend ou ignore une entrée. Les pièces jointes de cartes sont affichées dans le canevas de Cortana, sous les informations **displayText**.   |

La propriété **inputHint** permet d’indiquer à Cortana si votre bot attend une entrée. Si vous utilisez une invite intégrée, cette propriété affiche automatiquement la valeur par défaut **expectingInput**.


| Valeur | Description |
|------|------|
| **acceptingInput** | Votre bot est passivement prêt pour l’entrée, mais il n’est pas en attente d’une réponse. Cortana accepte une entrée de l’utilisateur si celui-ci maintient le bouton du microphone appuyé.|
| **expectingInput** | Indique que le bot attend activement une réponse de l’utilisateur. Cortana est à l’écoute de l’utilisateur qui doit parler dans le microphone.  |
| **ignoringInput** | Cortana ignore l’entrée. Votre bot peut envoyer cet indicateur s’il traite activement une demande ; il ignore les entrées des utilisateurs jusqu’à ce que cette requête soit traitée.  |


L’exemple suivant montre comment Cortana lit le texte brut ou code SSML :

```javascript
// Have Cortana read plain text
session.say('This is the text that Cortana displays', 'This is the text that is spoken by Cortana.');

// Have Cortana read SSML
session.say('This is the text that Cortana displays', '<speak version="1.0" xmlns="http://www.w3.org/2001/10/synthesis" xml:lang="en-US">This is the text that is spoken by Cortana.</speak>');
```

Cet exemple montre comment faire savoir à Cortana que l’entrée utilisateur est attendue. Le microphone est laissé ouvert.
```javascript
// Add an InputHint to let Cortana know to expect user input
session.say('Hi there', 'Hi, what’s your name?', {
    inputHint: builder.InputHint.expectingInput
});
```
<!-- TODO: tip about time limit and batching -->


### <a name="prompts"></a>Invites

Outre la méthode **session.say()**, vous pouvez également passer le texte ou code SSML à des invites intégrées à l’aide des options **speak** et **retrySpeak**.  

```javascript
builder.Prompts.text(session, 'text based prompt', {                                    
    speak: 'Cortana reads this out initially',                                               
    retrySpeak: 'This message is repeated by Cortana after waiting a while for user input',  
    inputHint: builder.InputHint.expectingInput                                              
});
```



<!-- TODO: Link to SSML library -->

Pour offrir à l’utilisateur une liste de choix, utilisez **Prompts.choice**. L’option **synonyms** permet plus de souplesse dans la reconnaissance des énoncés de l’utilisateur. L’option **value** est retournée dans **results.response.entity**. L’option **action** spécifie l’étiquette que votre bot affiche pour le choix.

**Prompts.Choice** prend en charge les choix ordinaux. Cela signifie que l’utilisateur peut dire « le premier », « le deuxième » ou « le troisième » pour choisir un élément dans une liste. Par exemple, avec l’invite suivante, si l’utilisateur a demandé à Cortana « la deuxième option », l’invite retournera la valeur 8.

```javascript
        var choices = [
            { value: '4', action: { title: '4 Sides' }, synonyms: 'four|for|4 sided|4 sides' },
            { value: '8', action: { title: '8 Sides' }, synonyms: 'eight|ate|8 sided|8 sides' },
            { value: '12', action: { title: '12 Sides' }, synonyms: 'twelve|12 sided|12 sides' },
            { value: '20', action: { title: '20 Sides' }, synonyms: 'twenty|20 sided|20 sides' },
        ];
        builder.Prompts.choice(session, 'choose_sides', choices, { 
            speak: speak(session, 'choose_sides_ssml') // use helper function to format SSML
        });
```

Dans l’exemple précédent, le code SSML pour la propriété **speak** de l’invite est mis en forme à l’aide de chaînes stockées dans un fichier d’invites localisées au format suivant. 

```json
{
    "choose_sides": "__Number of Sides__",
    "choose_sides_ssml": [
        "How many sides on the dice?",
        "Pick your poison.",
        "All the standard sizes are supported."
    ]
}
```


Une fonction d’assistance crée ensuite l’élément racine requis d’un document Speech Synthesis Markup Language (SSML). 

```javascript

module.exports.speak = function (template, params, options) {
    options = options || {};
    var output = '<speak xmlns="http://www.w3.org/2001/10/synthesis" ' +
        'version="' + (options.version || '1.0') + '" ' +
        'xml:lang="' + (options.lang || 'en-US') + '">';
    output += module.exports.vsprintf(template, params);
    output += '</speak>';
    return output;
}
```

> [!TIP]
> Vous pouvez rechercher un petit module utilitaire (ssml.js) pour créer les réponses SSML de votre bot dans l’[exemple de compétence Roller](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/demo-RollerSkill).
> Plusieurs bibliothèques SSML utiles sont également disponibles via [npm](https://www.npmjs.com/search?q=ssml), facilitant la création d’éléments SSML correctement formatés.

## <a name="display-cards-in-cortana"></a>Afficher les cartes dans Cortana

Outre les réponses oralisées, Cortana peut également afficher les cartes jointes. Cortana prend en charge les cartes enrichies suivantes :
* [HeroCard](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.herocard.html)
* [ReceiptCard](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.receiptcard.html)
* [ThumbnailCard](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.thumbnailcard.html)

Consultez les [Bonnes pratiques de conception de cartes][CardDesign] et voyez à quoi ressemblent ces cartes dans Cortana. Pour obtenir un exemple montrant comment ajouter une carte riche à un bot, consultez [Envoyer des cartes riches](bot-builder-nodejs-send-rich-cards.md). 

Le code suivant montre comment ajouter les propriétés **speak** et **inputHint** à un message contenant une carte de bannière.

```javascript 

bot.dialog('HelpDialog', function (session) {
    var card = new builder.HeroCard(session)
        .title('help_title')
        .buttons([
            builder.CardAction.imBack(session, 'roll some dice', 'Roll Dice'),
            builder.CardAction.imBack(session, 'play yahtzee', 'Play Yahtzee')
        ]);
    var msg = new builder.Message(session)
        .speak(speak(session, 'I\'m roller, the dice rolling bot. You can say \'roll some dice\''))
        .addAttachment(card)
        .inputHint(builder.InputHint.acceptingInput); // Tell Cortana to accept input
    session.send(msg).endDialog();
}).triggerAction({ matches: /help/i });


/** This helper function builds the required root element of a Speech Synthesis Markup Language (SSML) document. */
module.exports.speak = function (template, params, options) {
    options = options || {};
    var output = '<speak xmlns="http://www.w3.org/2001/10/synthesis" ' +
        'version="' + (options.version || '1.0') + '" ' +
        'xml:lang="' + (options.lang || 'en-US') + '">';
    output += module.exports.vsprintf(template, params);
    output += '</speak>';
    return output;
}

```
## <a name="sample-rollerskill"></a>Exemple : RollerSkill
Le code des sections suivantes provient d’un exemple de compétence Cortana pour le lancement de dés. Téléchargez le code complet du bot à partir du [dépôt BotBuilder-Samples](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/demo-RollerSkill).

Vous appelez la compétence en disant son [nom d’appel][InvocationNameGuidelines] à Cortana. Pour la compétence de lancement de dés, après avoir [ajouté le bot au canal Cortana][CortanaChannel] et l’avoir inscrit en tant que compétence Cortana, vous pouvez l’appeler en disant à Cortana : « Demande Roller » ou « Demande à Roller de lancer les dés ».

### <a name="explore-the-code"></a>Explorer le code

L’exemple RollerSkill commence par ouvrir une carte avec quelques boutons pour montrer à l’utilisateur les options dont il dispose.

```javascript
/**
 *   Create your bot with a default message handler that receive messages from the user.
 * - This function is be called anytime the user's utterance isn't
 *   recognized by the other handlers in the bot.
 */
var bot = new builder.UniversalBot(connector, function (session) {
    // Just redirect to our 'HelpDialog'.
    session.replaceDialog('HelpDialog');
});

//...

bot.dialog('HelpDialog', function (session) {
    var card = new builder.HeroCard(session)
        .title('help_title')
        .buttons([
            builder.CardAction.imBack(session, 'roll some dice', 'Roll Dice'),
            builder.CardAction.imBack(session, 'play craps', 'Play Craps')
        ]);
    var msg = new builder.Message(session)
        .speak(speak(session, 'help_ssml'))
        .addAttachment(card)
        .inputHint(builder.InputHint.acceptingInput);
    session.send(msg).endDialog();
}).triggerAction({ matches: /help/i });
```

### <a name="prompt-the-user-for-input"></a>Demander à l’utilisateur d’effectuer une saisie

Le dialogue suivant configure un jeu personnalisé que le bot jouera.  Elle demande à l’utilisateur combien de facettes doivent posséder les dés, et combien de dés doivent être lancés. Après avoir créé la structure du jeu, elle la passe à un élément 'PlayGameDialog' distinct.

Pour démarrer le dialogue, le gestionnaire **triggerAction()** de ce dialogue permet à un utilisateur de dire quelque chose comme « Je souhaiterais lancer des dés ». Il utilise une expression régulière pour faire correspondre l’entrée utilisateur, mais vous pouvez tout aussi facilement utiliser une [intention LUIS](./bot-builder-nodejs-recognize-intent-luis.md). 


```javascript


bot.dialog('CreateGameDialog', [
    function (session) {
        // Initialize game structure.
        // - dialogData gives us temporary storage of this data in between
        //   turns with the user.
        var game = session.dialogData.game = { 
            type: 'custom', 
            sides: null, 
            count: null,
            turns: 0
        };

        var choices = [
            { value: '4', action: { title: '4 Sides' }, synonyms: 'four|for|4 sided|4 sides' },
            { value: '6', action: { title: '6 Sides' }, synonyms: 'six|sex|6 sided|6 sides' },
            { value: '8', action: { title: '8 Sides' }, synonyms: 'eight|8 sided|8 sides' },
            { value: '10', action: { title: '10 Sides' }, synonyms: 'ten|10 sided|10 sides' },
            { value: '12', action: { title: '12 Sides' }, synonyms: 'twelve|12 sided|12 sides' },
            { value: '20', action: { title: '20 Sides' }, synonyms: 'twenty|20 sided|20 sides' },
        ];
        builder.Prompts.choice(session, 'choose_sides', choices, { 
            speak: speak(session, 'choose_sides_ssml') 
        });
    },
    function (session, results) {
        // Store users input
        // - The response comes back as a find result with index & entity value matched.
        var game = session.dialogData.game;
        game.sides = Number(results.response.entity);

        /**
         * Ask for number of dice.
         */
        var prompt = session.gettext('choose_count', game.sides);
        builder.Prompts.number(session, prompt, {
            speak: speak(session, 'choose_count_ssml'),
            minValue: 1,
            maxValue: 100,
            integerOnly: true
        });
    },
    function (session, results) {
        // Store users input
        // - The response is already a number.
        var game = session.dialogData.game;
        game.count = results.response;

        /**
         * Play the game we just created.
         * 
         * replaceDialog() ends the current dialog and start a new
         * one in its place. We can pass arguments to dialogs so we'll pass the
         * 'PlayGameDialog' the game we created.
         */
        session.replaceDialog('PlayGameDialog', { game: game });
    }
]).triggerAction({ matches: [
    /(roll|role|throw|shoot).*(dice|die|dye|bones)/i,
    /new game/i
 ]});
```

### <a name="render-results"></a>Afficher les résultats

 Ce dialogue est notre principale boucle de jeu. Le bot stocke la structure du jeu dans **session.conversationData**. Ainsi, si l’utilisateur dit « relancer les dés », il nous suffit de relancer le même jeu de dés.

```javascript

bot.dialog('PlayGameDialog', function (session, args) {
    // Get current or new game structure.
    var game = args.game || session.conversationData.game;
    if (game) {
        // Generate rolls
        var total = 0;
        var rolls = [];
        for (var i = 0; i < game.count; i++) {
            var roll = Math.floor(Math.random() * game.sides) + 1;
            if (roll > game.sides) {
                // Accounts for 1 in a million chance random() generated a 1.0
                roll = game.sides;
            }
            total += roll;
            rolls.push(roll);
        }

        // Format roll results
        var results = '';
        var multiLine = rolls.length > 5;
        for (var i = 0; i < rolls.length; i++) {
            if (i > 0) {
                results += ' . ';
            }
            results += rolls[i];
        }

        // Render results using a card
        var card = new builder.HeroCard(session)
            .subtitle(game.count > 1 ? 'card_subtitle_plural' : 'card_subtitle_singular', game)
            .buttons([
                builder.CardAction.imBack(session, 'roll again', 'Roll Again'),
                builder.CardAction.imBack(session, 'new game', 'New Game')
            ]);
        if (multiLine) {
            //card.title('card_title').text('\n\n' + results + '\n\n');
            card.text(results);
        } else {
            card.title(results);
        }
        var msg = new builder.Message(session).addAttachment(card);

        // Determine bots reaction for speech purposes
        var reaction = 'normal';
        var min = game.count;
        var max = game.count * game.sides;
        var score = total/max;
        if (score == 1.0) {
            reaction = 'best';
        } else if (score == 0) {
            reaction = 'worst';
        } else if (score <= 0.3) {
            reaction = 'bad';
        } else if (score >= 0.8) {
            reaction = 'good';
        }

        // Check for special craps rolls
        if (game.type == 'craps') {
            switch (total) {
                case 2:
                case 3:
                case 12:
                    reaction = 'craps_lose';
                    break;
                case 7:
                    reaction = 'craps_seven';
                    break;
                case 11:
                    reaction = 'craps_eleven';
                    break;
                default:
                    reaction = 'craps_retry';
                    break;
            }
        }

        // Build up spoken response
        var spoken = '';
        if (game.turn == 0) {
            spoken += session.gettext('start_' + game.type + '_game_ssml') + ' ';
        } 
        spoken += session.gettext(reaction + '_roll_reaction_ssml');
        msg.speak(ssml.speak(spoken));

        // Increment number of turns and store game to roll again
        game.turn++;
        session.conversationData.game = game;

        /**
         * Send card and bot's reaction to user. 
         */

        msg.inputHint(builder.InputHint.acceptingInput);
        session.send(msg).endDialog();
    } else {
        // User started session with "roll again" so let's just send them to
        // the 'CreateGameDialog'
        session.replaceDialog('CreateGameDialog');
    }
}).triggerAction({ matches: /(roll|role|throw|shoot) again/i });
```

## <a name="next-steps"></a>Étapes suivantes
Si votre bot s’exécute localement ou qu’il est déployé dans le cloud, vous pouvez l’appeler à partir de Cortana. Consultez [Tester une compétence Cortana](../bot-service-debug-cortana-skill.md) pour connaître les étapes nécessaires pour tester votre compétence Cortana.


## <a name="additional-resources"></a>Ressources supplémentaires
* [Kit de compétences Cortana][CortanaGetStarted]
* [Ajouter de la reconnaissance vocale aux messages](bot-builder-nodejs-text-to-speech.md)
* [Référence SSML][SSMLRef]
* [Bonnes pratiques de conception vocale pour Cortana][VoiceDesign]
* [Bonnes pratiques de conception de cartes pour Cortana][CardDesign]
* [Centre de développement Java][CortanaDevCenter]
* [Bonnes pratiques de test et de débogage pour Cortana][Cortana-TestBestPractice]


[CortanaGetStarted]: /cortana/getstarted
[BFPortal]: https://dev.botframework.com/


[SSMLRef]: https://aka.ms/cortana-ssml
[IMessage]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.imessage.html
[Send]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session#send
[CortanaDevCenter]: https://developer.microsoft.com/en-us/cortana

[CortanaSpecificEntities]: https://aka.ms/lgvcto
[CortanaAuth]: https://aka.ms/vsdqcj

[InvocationNameGuidelines]: https://aka.ms/cortana-invocation-guidelines
[VoiceDesign]: https://aka.ms/cortana-design-voice
[CardDesign]: https://aka.ms/cortana-design-card
[Cortana-Debug]: https://aka.ms/cortana-enable-debug
[Cortana-Publish]: https://aka.ms/cortana-publish


[CortanaTry]: https://aka.ms/try-cortana-bot
[CortanaChannel]: https://aka.ms/bot-cortana-channel
[Cortana-TestBestPractice]: https://aka.ms/cortana-test-best-practice
