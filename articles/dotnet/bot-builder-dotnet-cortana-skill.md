---
title: Générer une compétence Cortana avec .NET | Microsoft Docs
description: Découvrez les concepts essentiels liés à la création d’une compétence Cortana dans le Kit SDK Bot Builder pour .NET.
keywords: Bot Framework, Compétence Cortana, reconnaissance vocale, .NET, Bot Builder, kit SDK, concepts clés, concepts de base
author: DeniseMak
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 030d17fa25a436ee8e8a1d093924e61f12e14e18
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2018
ms.locfileid: "49998304"
---
# <a name="build-a-speech-enabled-bot-with-cortana-skills"></a>Créer un bot à reconnaissance vocale avec des compétences Cortana

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-cortana-skill.md)
> - [Node.JS](../nodejs/bot-builder-nodejs-cortana-skill.md)


Le kit SDK Bot Builder pour .NET vous permet de créer un bot à reconnaissance vocale en le connectant par le biais du canal Cortana en tant que compétence Cortana. 


> [!TIP]
> Pour plus d’informations sur la définition d’une compétence et ses possibilités, consultez le [Kit de compétences Cortana][CortanaGetStarted].

La création d’une compétence Cortana avec Bot Framework nécessite très peu de connaissances particulières de Cortana, et se limite essentiellement à l’élaboration d’un bot. Une des différences majeures, probablement, par rapport aux autres bots que vous avez pu créer auparavant est la présence conjointe d’un composant visuel et d’un composant audio dans Cortana. Pour le composant visuel, Cortana dispose d’une zone de canevas permettant la restitution de contenus, tels que des cartes. Quant au composant audio, vous fournissez du texte ou des données SSML dans les messages de votre bot, et Cortana en fait la lecture à l’utilisateur, donnant ainsi une voix à votre bot. 

> [!NOTE]
> Cortana est disponible sur toutes sortes d’appareils. Certains sont équipés d’un écran, tandis que d’autres, comme une enceinte acoustique, n’en sont pas forcément pourvus. Vous devez vous assurer que votre bot est capable de gérer ces deux scénarios. Consultez les [entités propres à Cortana][CortanaSpecificEntities] pour savoir comment vérifier les informations des appareils.

## <a name="adding-speech-to-your-bot"></a>Ajout de la reconnaissance vocale à votre bot

Les messages parlés à partir de votre bot sont représentés en SSML (Speech Synthesis Markup Language). Le kit SDK Bot Builder vous permet d’inclure le SSML dans les réponses de votre bot afin de contrôler ce qu’il dit, en plus de ce qu’il affiche.  Vous pouvez également contrôler l’état du microphone de Cortana en précisant si votre bot accepte, attend ou ignore l’entrée utilisateur.

Définissez la propriété `Speak` de l’objet `IMessageActivity` pour indiquer un message à faire dire par Cortana. Si vous spécifiez le format texte brut, Cortana détermine la façon dont les mots sont prononcés. 

```cs
Activity reply = activity.CreateReply("This is the text that Cortana displays."); 
reply.Speak = "This is the text that Cortana will say.";
```

Si vous souhaitez davantage de contrôle sur la tonalité, le ton et l’accentuation, spécifiez la mise en forme de la propriété `Speak` en [SSML (Speech Synthesis Markup Language)](http://www.w3.org/TR/speech-synthesis/).  

L’exemple de code suivant précise que le mot « texte » doit être prononcé avec une accentuation moyenne :
```cs
Activity reply = activity.CreateReply("This is the text that will be displayed.");
reply.Speak = "<speak version=\"1.0\" xmlns=\"http://www.w3.org/2001/10/synthesis\" xml:lang=\"en-US\">This is the <emphasis level=\"moderate\">text</emphasis> that will be spoken.</speak>";
```


La propriété **InputHint** permet d’indiquer à Cortana si votre bot attend une entrée. La valeur par défaut est **ExpectingInput** pour une invite, et **AcceptingInput** pour d’autres types de réponses.


| Valeur | Description |
|------|------|
| **AcceptingInput** | Votre bot est passivement prêt pour l’entrée, mais il n’est pas en attente d’une réponse. Cortana accepte une entrée de l’utilisateur si celui-ci maintient le bouton du microphone appuyé.|
| **ExpectingInput** | Indique que le bot attend activement une réponse de l’utilisateur. Cortana est à l’écoute de l’utilisateur qui doit parler dans le microphone.  |
| **IgnoringInput** | Cortana ignore l’entrée. Votre bot peut envoyer cet indicateur s’il traite activement une demande ; il ignore les entrées des utilisateurs jusqu’à ce que cette requête soit traitée.  |

<!-- TODO: tip about time limit and batching -->

Cet exemple montre comment faire savoir à Cortana que l’entrée utilisateur est attendue. Le microphone est laissé ouvert.
```cs
// Add an InputHint to let Cortana know to expect user input
Activity reply = activity.CreateReply("This is the text that will be displayed."); 
reply.Speak = "This is the text that will be spoken.";
reply.InputHint = InputHints.ExpectingInput;
```



## <a name="display-cards-in-cortana"></a>Afficher les cartes dans Cortana

En plus des réponses oralisées, Cortana peut également afficher les cartes jointes. Cortana prend en charge les cartes enrichies suivantes :

| Type de carte | Description |
|----|----|
| [HeroCard][heroCard] | Carte contenant généralement une grande image, un ou plusieurs boutons et du texte. |
| [ThumbnailCard][thumbnailCard] | Carte contenant généralement une image miniature, un ou plusieurs boutons et du texte. |
| [ReceiptCard][receiptCard] | Carte permettant à un bot de fournir un reçu à l’utilisateur. Elle contient généralement la liste des articles à inclure sur le reçu, la taxe et le total, ainsi que du texte. |
| [SignInCard][signinCard] | Carte permettant à un bot de demander à un utilisateur de se connecter. Elle contient généralement du texte et un ou plusieurs boutons sur lesquels l’utilisateur peut cliquer pour lancer le processus de connexion. |


Consultez les [Bonnes pratiques de conception de cartes][CardDesign] et voyez à quoi ressemblent ces cartes dans Cortana. Pour obtenir un exemple illustrant l’utilisation d’une carte enrichie dans un bot, consultez [Ajouter des cartes enrichies en pièces jointes aux messages](bot-builder-dotnet-add-rich-card-attachments.md). 

<!--
The following code demonstrates how to add the `Speak` and `InputHint` properties to a message containing a `HeroCard`.
-->


## <a name="sample-rollerskill"></a>Exemple : RollerSkill
Le code des sections suivantes provient d’un exemple de compétence Cortana pour le lancement de dés. Téléchargez le code complet du bot à partir du [dépôt BotBuilder-Samples](https://github.com/Microsoft/BotBuilder-Samples/).

Vous appelez la compétence en disant son [nom d’appel][InvocationNameGuidelines] à Cortana. Pour la compétence de lancement de dés, après avoir [ajouté le bot au canal Cortana][CortanaChannel] et l’avoir inscrit en tant que compétence Cortana, vous pouvez l’appeler en disant à Cortana : « Demande Roller » ou « Demande à Roller de lancer les dés ».

### <a name="explore-the-code"></a>Explorer le code



Pour appeler les dialogues appropriés, les gestionnaires d’activités définis dans `RootDispatchDialog.cs` utilisent des expressions régulières pour faire correspondre l’entrée utilisateur. Par exemple, le gestionnaire de l’exemple suivant est déclenché si l’utilisateur dit quelque chose comme « Je souhaiterais lancer les dés ». Des synonymes sont inclus dans l’expression régulière, pour que des énoncés similaires déclenchent le dialogue.
```cs
        [RegexPattern("(roll|role|throw|shoot).*(dice|die|dye|bones)")]
        [RegexPattern("new game")]
        [ScorableGroup(1)]
        public async Task NewGame(IDialogContext context, IActivity activity)
        {
            context.Call(new CreateGameDialog(), AfterGameCreated);
        }
```

Le dialogue `CreateGameDialog` configure un jeu personnalisé pour que le bot joue. Il utilise `PromptDialog` pour demander à l’utilisateur combien de faces doivent avoir les dés, et combien de dés doivent être lancés. Notez que l’objet `PromptOptions` qui est utilisé pour initialiser l’invite contient une propriété `speak` pour la version parlée de l’invite.

```cs
    [Serializable]
    public class CreateGameDialog : IDialog<GameData>
    {
        public async Task StartAsync(IDialogContext context)
        {
            context.UserData.SetValue<GameData>(Utils.GameDataKey, new GameData());

            var descriptions = new List<string>() { "4 Sides", "6 Sides", "8 Sides", "10 Sides", "12 Sides", "20 Sides" };
            var choices = new Dictionary<string, IReadOnlyList<string>>()
             {
                { "4", new List<string> { "four", "for", "4 sided", "4 sides" } },
                { "6", new List<string> { "six", "sex", "6 sided", "6 sides" } },
                { "8", new List<string> { "eight", "8 sided", "8 sides" } },
                { "10", new List<string> { "ten", "10 sided", "10 sides" } },
                { "12", new List<string> { "twelve", "12 sided", "12 sides" } },
                { "20", new List<string> { "twenty", "20 sided", "20 sides" } }
            };

            var promptOptions = new PromptOptions<string>(
                Resources.ChooseSides,
                choices: choices,
                descriptions: descriptions,
                speak: SSMLHelper.Speak(Utils.RandomPick(Resources.ChooseSidesSSML))); // spoken prompt

            PromptDialog.Choice(context, this.DiceChoiceReceivedAsync, promptOptions);
        }

        private async Task DiceChoiceReceivedAsync(IDialogContext context, IAwaitable<string> result)
        {
            GameData game;
            if (context.UserData.TryGetValue<GameData>(Utils.GameDataKey, out game))
            {
                int sides;
                if (int.TryParse(await result, out sides))
                {
                    game.Sides = sides;
                    context.UserData.SetValue<GameData>(Utils.GameDataKey, game);
                }

                var promptText = string.Format(Resources.ChooseCount, sides);

                var promptOption = new PromptOptions<long>(promptText, choices: null, speak: SSMLHelper.Speak(Utils.RandomPick(Resources.ChooseCountSSML)));

                var prompt = new PromptDialog.PromptInt64(promptOption);
                context.Call<long>(prompt, this.DiceNumberReceivedAsync);
            }
        }

        private async Task DiceNumberReceivedAsync(IDialogContext context, IAwaitable<long> result)
        {
            GameData game;
            if (context.UserData.TryGetValue<GameData>(Utils.GameDataKey, out game))
            {
                game.Count = await result;
                context.UserData.SetValue<GameData>(Utils.GameDataKey, game);
            }

            context.Done(game);
        }
    }
```

`PlayGameDialog` restitue les résultats en les affichant dans une carte `HeroCard` et en élaborant un message vocal pour les annoncer à l’aide de la méthode `Speak`.

```cs
   [Serializable]
    public class PlayGameDialog : IDialog<object>
    {
        private const string RollAgainOptionValue = "roll again";

        private const string NewGameOptionValue = "new game";

        private GameData gameData;

        public PlayGameDialog(GameData gameData)
        {
            this.gameData = gameData;
        }

        public async Task StartAsync(IDialogContext context)
        {
            if (this.gameData == null)
            {
                if (!context.UserData.TryGetValue<GameData>(Utils.GameDataKey, out this.gameData))
                {
                    // User started session with "roll again" so let's just send them to
                    // the 'CreateGameDialog'
                    context.Done<object>(null);
                }
            }

            int total = 0;
            var randomGenerator = new Random();
            var rolls = new List<int>();

            // Generate Rolls
            for (int i = 0; i < this.gameData.Count; i++)
            {
                var roll = randomGenerator.Next(1, this.gameData.Sides);
                total += roll;
                rolls.Add(roll);
            }

            // Format rolls results
            var result = string.Join(" . ", rolls.ToArray());
            bool multiLine = rolls.Count > 5;

            var card = new HeroCard()
            {
                Subtitle = string.Format(
                    this.gameData.Count > 1 ? Resources.CardSubtitlePlural : Resources.CardSubtitleSingular,
                    this.gameData.Count,
                    this.gameData.Sides),
                Buttons = new List<CardAction>()
                {
                    new CardAction(ActionTypes.ImBack, "Roll Again", value: RollAgainOptionValue),
                    new CardAction(ActionTypes.ImBack, "New Game", value: NewGameOptionValue)
                }
            };

            if (multiLine)
            {
                card.Text = result;
            }
            else
            {
                card.Title = result;
            }

            var message = context.MakeMessage();
            message.Attachments = new List<Attachment>()
            {
                card.ToAttachment()
            };

            // Determine bots reaction for speech purposes
            string reaction = "normal";

            var min = this.gameData.Count;
            var max = this.gameData.Count * this.gameData.Sides;
            var score = total / max;
            if (score == 1)
            {
                reaction = "Best";
            }
            else if (score == 0)
            {
                reaction = "Worst";
            }
            else if (score <= 0.3)
            {
                reaction = "Bad";
            }
            else if (score >= 0.8)
            {
                reaction = "Good";
            }

            // Check for special craps rolls
            if (this.gameData.Type == "Craps")
            {
                switch (total)
                {
                    case 2:
                    case 3:
                    case 12:
                        reaction = "CrapsLose";
                        break;
                    case 7:
                        reaction = "CrapsSeven";
                        break;
                    case 11:
                        reaction = "CrapsEleven";
                        break;
                    default:
                        reaction = "CrapsRetry";
                        break;
                }
            }

            // Build up spoken response
            var spoken = string.Empty;
            if (this.gameData.Turns == 0)
            {
                spoken += Utils.RandomPick(Resources.ResourceManager.GetString($"Start{this.gameData.Type}GameSSML"));
            }

            spoken += Utils.RandomPick(Resources.ResourceManager.GetString($"{reaction}RollReactionSSML"));

            message.Speak = SSMLHelper.Speak(spoken);

            // Increment number of turns and store game to roll again
            this.gameData.Turns++;
            context.UserData.SetValue<GameData>(Utils.GameDataKey, this.gameData);

            // Send card and bots reaction to user.
            message.InputHint = InputHints.AcceptingInput;
            await context.PostAsync(message);

            context.Done<object>(null);
        }
    }
```
## <a name="next-steps"></a>Étapes suivantes

Si votre bot s’exécute localement ou qu’il est déployé dans le cloud, vous pouvez l’appeler à partir de Cortana. Consultez [Tester une compétence Cortana](../bot-service-debug-cortana-skill.md) pour connaître les étapes nécessaires au test de votre compétence Cortana.


## <a name="additional-resources"></a>Ressources supplémentaires
* [Kit de compétences Cortana][CortanaGetStarted]
* [Ajouter la reconnaissance vocale aux messages](bot-builder-dotnet-text-to-speech.md)
* [Référence SSML][SSMLRef]
* [Bonnes pratiques de conception vocale pour Cortana][VoiceDesign]
* [Bonnes pratiques de conception de cartes pour Cortana][CardDesign]
* [Centre de développement Java][CortanaDevCenter]
* [Bonnes pratiques de test et de débogage pour Cortana][Cortana-TestBestPractice]
* <a href="/dotnet/api/?view=botbuilder-3.11.0" target="_blank">Kit SDK Bot Builder pour .NET</a>

[CortanaGetStarted]: /cortana/getstarted
[BFPortal]: https://dev.botframework.com/

[SSMLRef]: https://aka.ms/cortana-ssml
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

[heroCard]: /dotnet/api/microsoft.bot.connector.herocard

[thumbnailCard]: /dotnet/api/microsoft.bot.connector.thumbnailcard 

[receiptCard]: /dotnet/api/microsoft.bot.connector.receiptcard 

[signinCard]: /dotnet/api/microsoft.bot.connector.signincard 


