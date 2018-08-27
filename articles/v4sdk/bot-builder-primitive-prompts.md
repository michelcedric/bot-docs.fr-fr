---
title: Gérer un flux de conversation avec invites primitives | Microsoft Docs
description: Découvrez comment gérer un flux de conversation avec des invites primitives dans le Kit de développement logiciel (SDK) Bot Builder.
keywords: flux de conversation, invites
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 7/20/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 1514e1bcafc87be9e8449382bca7aa156e512ed9
ms.sourcegitcommit: e8c513d3af5f0c514cadcbcd0a737a7393405afa
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/20/2018
ms.locfileid: "40228344"
---
# <a name="prompt-users-for-input-using-your-own-prompts"></a>Demander aux utilisateurs des entrées en utilisant vos propres invites
[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Une conversation entre un bot et un utilisateur implique souvent une demande (invite) d’informations à l’utilisateur, l’analyse de sa réponse et l’action en fonction des informations obtenues. Dans la rubrique [Demander aux utilisateurs avec la bibliothèque de dialogues](bot-builder-prompts.md), la marche à suivre pour demander une entrée aux utilisateurs avec la bibliothèque de **dialogues** est détaillée. Entre autres choses, la bibliothèque de **dialogues** se charge de suivre la conversation actuelle ainsi que la question actuelle posée. Elle fournit aussi des méthodes pour demander différents types d’informations telles que du texte, un nombre, une date, une heure, etc. 

Dans certains cas, les **dialogues** intégrés peuvent ne pas constituer la solution pour votre bot ; les **dialogues** peuvent surcharger le bot, être trop rigides ou empêcher le bot d’accomplir ce que vous voulez. Dans ces cas, vous pouvez ignorer la bibliothèque et implémenter votre propre logique d’invite. Cette rubrique vous montrera comment configurer votre *bot Echo* de base afin que vous puissiez gérer une conversation avec vos propres invites.

## <a name="track-prompt-states"></a>Suivre les états d’invite

Dans une conversation guidée par invite, vous devez suivre où vous vous trouvez dans la conversation, et les questions qui y sont posées. Dans le code, cela se traduit par la gestion de deux balises. 

Par exemple, vous pouvez créer deux propriétés que vous voulez suivre. 

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Nous avons ici une classe de profil utilisateur imbriquée dans nos informations d’invite, leur permettant d’être stockées dans l’état de notre [bot](bot-builder-howto-v4-state.md).

```csharp
// Where user information will be stored
public class UserProfile
{
    public string userName = null;
    public string workPlace = null;
    public int age = 0;
}

// Flags to keep track of our prompts, and the 
// user profile object for this conversation
public class PromptInformation
{
    public string topicTitle = null;
    public string prompt = null;
    public UserProfile userProfile = null;
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Ces états suivent juste les sujets et les invites dans lesquelles nous nous trouvons actuellement. Pour s’assurer que les balises fonctionnent comme prévu une fois déployées dans le cloud, nous les stockons dans l’[état de la conversation](bot-builder-howto-v4-state.md). Placez ce code dans la logique principale de votre bot.

**app.js**
```javascript
// Define a topicStates object if it doesn't exist in the convoState.
if(!convo.topicStates){
    convo.topicStates = {
        "topicTitle": undefined, // Current conversation topic in progress
        "prompt": undefined      // Current prompt state in progress - indicate what question is being asked.
    };
}
```

---

## <a name="manage-a-topic-of-conversation"></a>Gérer le sujet de conversation

Dans une conversation séquentielle, telle qu’une conversation où vous récoltez des informations de l’utilisateur, vous devez savoir quand l’utilisateur est entré dans la conversation et où il se trouve. Vous pouvez suivre cela dans le gestionnaire de tour de bot en configurant et en vérifiant les balises d’invite définies ci-dessus, et agir en conséquence. L’exemple ci-dessous montre comment vous pouvez utiliser ces balises pour récolter les informations du profil utilisateur dans la conversation.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// Get our current state, as defined above
var convoState = context.GetConversationState<PromptInformation>();

if (convoState.userProfile == null)
{
    await context.SendActivity("Welcome new user, please fill out your profile information.");
    convoState.topicTitle = "profileTopic"; // Start the userProfile topic
    convoState.userProfile = new UserProfile();
}

// Start or continue a conversation if we are in one
if (convoState.topicTitle == "profileTopic")
{
    if (convoState.userProfile.userName == null && convoState.prompt == null)
    {
        convoState.prompt = "askName";
        await context.SendActivity("What is your name?");
    }
    else if (convoState.prompt == "askName")
    {
        // Save the user's response
        convoState.userProfile.userName = context.Activity.Text;

        // Ask next question
        convoState.prompt = "askAge";
        await context.SendActivity("How old are you?");
    }
    else if (convoState.prompt == "askAge")
    {
        // Save user's response
        if (!Int32.TryParse(context.Activity.Text, out convoState.userProfile.age))
        {
            // Failed to convert to int
            await context.SendActivity("Failed to convert string to int");
        }
        else
        {
            // Ask next question
            convoState.prompt = "workPlace";
            await context.SendActivity("Where do you work?");
        }

    }
    else if (convoState.prompt == "workPlace")
    {
        // Save user's response
        convoState.userProfile.workPlace = context.Activity.Text;

        // Done
        convoState.topicTitle = null; // Reset conversation flag
        convoState.prompt = null;     // Reset the prompt flag
        
        await context.SendActivity("Thank you. Your profile is complete.");
    }
    else // Somehow our flags got inappropriately set
    {
        await context.SendActivity("Looks like something went wrong, let's start over");
        convoState.userProfile = null;
        convoState.prompt = null;
        convoState.topicTitle = null;
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**app.js**
```javascript
server.post('/api/messages', (req, res) => {
    adapter.processActivity(req, res, async (context) => {
        const isMessage = (context.activity.type === 'message');
        // State will store all of your information 
        const convo = conversationState.get(context);

        // Defined flags to manage the conversation flow and prompt states
        // convo.topicStates.topicTitle - Current conversation topic in progress
        // convo.topicStates.prompt - Current prompt state in progress - indicate what question is being asked.
        
        if (isMessage) {
            // Define a topicStates object if it doesn't exist in the convoState.
            if(!convo.topicStates){
                convo.topicStates = {
                    "topicTitle": undefined, // Current conversation topic in progress
                    "prompt": undefined      // Current prompt state in progress - indicate what question is being asked.
                };
            }
            
            // If user profile is not defined then define it.
            if(!convo.userProfile){
                
                await context.sendActivity(`Welcome new user, please fill out your profile information.`);
                convo.topicStates.topicTitle = "profileTopic"; // Start the userProfile topic
                convo.userProfile = { // Define the user's profile object
                    "userName": undefined,
                    "age": undefined,
                    "workPlace": undefined
                }; 
            }

            // Start or continue a conversation if we are in one
            if(convo.topicStates.topicTitle == "profileTopic"){
                if(!convo.userProfile.userName && !convo.topicStates.prompt){
                    convo.topicStates.prompt = "askName";
                    await context.sendActivity("What is your name?");
                }
                else if(convo.topicStates.prompt == "askName"){
                    // Save the user's response
                    convo.userProfile.userName = context.activity.text; 

                    // Ask next question
                    convo.topicStates.prompt = "askAge";
                    await context.sendActivity("How old are you?");
                }
                else if(convo.topicStates.prompt == "askAge"){
                    // Save user's response
                    convo.userProfile.age = context.activity.text;

                    // Ask next question
                    convo.topicStates.prompt = "workPlace";
                    await context.sendActivity("Where do you work?");
                }
                else if(convo.topicStates.prompt == "workPlace"){
                    // Save user's response
                    convo.userProfile.workPlace = context.activity.text;

                    // Done
                    convo.topicStates.topicTitle = undefined; // Reset conversation flag
                    convo.topicStates.prompt = undefined;     // Reset the prompt flag
                    await context.sendActivity("Thank you. Your profile is complete.");
                }
            }

            // Check for valid intents
            else if(context.activity.text && context.activity.text.match(/hi/ig)){
                await context.sendActivity(`Hi ${convo.userProfile.userName}.`);
            }
            else {
                // Default message
                await context.sendActivity("Hi. I'm the Contoso bot.");
            }
        }

    });
});

```

---

L’exemple de code ci-dessus vérifie qu’un profil utilisateur est défini dans la mémoire. Si ce n’est pas le cas et qu’il s’agit d’un nouvel utilisateur, il définit la balise pour démarrer le sujet automatiquement. Il montre ensuite comment progresser dans la conversation en définissant la balise `prompt` à la valeur de la question actuelle posée. Avec cette balise définie sur la bonne valeur, la clause conditionnelle cernera la réponse de l’utilisateur à chaque tour et la traitera en fonction. 

Enfin, vous devez réinitialiser ces balises lorsque vous avez terminé de demander des informations, afin de ne pas piéger votre bot dans une boucle. Vous pouvez étendre cette configuration de base pour gérer plus de flux de conversation complexes selon les besoins de votre bot en définissant simplement d’autres balises ou en créant une branche pour la conversation selon l’entrée de l’utilisateur.

## <a name="manage-multiple-topics-of-conversations"></a>Gérer plusieurs sujets de conversations

Une fois que vous êtes en mesure de gérer un sujet de conversation, vous pouvez étendre la logique de votre bot afin qu’il puisse gérer plusieurs sujets de conversations. Comme pour un seul sujet de conversation, plusieurs sujets peuvent être gérés en vérifiant simplement les conditions qui déclenchent un sujet particulier, puis en empruntant le chemin approprié.

Dans notre exemple ci-dessus, vous pouvez le refactoriser pour autoriser d’autres fonctions et sujets, comme la réservation d’une table ou la commande d’un repas. Des informations supplémentaires peuvent être ajoutées au profil utilisateur ou aux balises d’état de sujet si besoin pour suivre la conversation.

Pour gérer plus facilement plusieurs sujets de conversations, une méthode consiste à fournir un menu principal. Avec des [actions suggérées](bot-builder-howto-add-suggested-actions.md), vous pouvez donner à votre utilisateur un choix de sujet à engager, puis l’approfondir. Vous pouvez aussi trouver utile de diviser le code en fonctions indépendantes, facilitant ainsi le suivi du flux de conversation.

## <a name="validate-user-input"></a>Valider les entrées utilisateur

La bibliothèque de **dialogues** fournit des méthodes intégrées pour valider l’entrée de l’utilisateur, mais nous pouvons aussi le faire avec nos propres invites. Par exemple, si nous demandons l’âge de l’utilisateur, nous devons être sûrs d’obtenir un nombre, et pas une réponse du genre « Bob ».

L’analyse d’un nombre ou d’une date et heure est une tâche complexe qui dépasse l’objectif de cette rubrique. Heureusement, il existe une bibliothèque que nous pouvons exploiter. Pour analyser correctement ces informations, nous utilisons la bibliothèque [Reconnaissance de texte Microsoft](https://github.com/Microsoft/Recognizers-Text). Ce package est disponible via NuGet, ou en téléchargement depuis le référentiel. En outre, il est important de noter qu’il est inclus dans la bibliothèque de **dialogues**, même si nous ne l’utiliserons pas ici.

Cette bibliothèque est particulièrement utile pour l’analyse de langage commun ou de réponses complexes sur des éléments tels qu’une date, une heure ou des numéros de téléphone. Dans cet exemple, nous ne validons que le nombre de la taille d’un groupe pour une réservation, mais il est possible d’appliquer cette idée à d’autres validations plus profondes. Si vous ne connaissez pas bien cette bibliothèque, consultez le contenu disponible sur ce site pour voir ce qu’elle peut offrir.

Dans cet exemple, nous expliquons seulement l’utilisation de `RecognizeNumber`. Vous pouvez trouver des informations sur l’utilisation d’autres méthodes de reconnaissance de la bibliothèque dans cette [documentation de référentiel](https://github.com/Microsoft/Recognizers-Text/blob/master/README.md).

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Pour utiliser une bibliothèque de reconnaissance, ajoutez-la à vos instructions d’utilisation.

```csharp
using Microsoft.Recognizers.Text.Number;
using Microsoft.Recognizers.Text;
using System.Linq; // Used to get the first result from the recognizer
```

Ensuite, définissez une fonction qui s’occupe de la validation.

```csharp
private async Task<bool> ValidatePartySize(ITurnContext context, string value)
{
    try
    {
        // Recognize the input as a number. This works for responses such as
        // "twelve" as well as "12"
        var result = NumberRecognizer.RecognizeNumber(input, Culture.English);

        // Attempt to convert the Recognizer result to an integer
        Int32.TryParse(result.First().Text, out int partySize);
        
        if (partySize < 6)
        {
            throw new Exception("Party size too small.");
        }
        else if (partySize > 20)
        {
            throw new Exception("Party size too big.");
        }

        // If we got through this, the number is valid
        return true;
    }
    catch (Exception)
    {
        await context.SendActivity("Error with your party size. < br /> Please specify a number between 6 - 20.");
        return false;
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Pour utiliser la bibliothèque de reconnaissance, demandez-la dans **app.js** :

```javascript
// Required packages for this bot
var Recognizers = require('@microsoft/recognizers-text-suite');
```

Ensuite, définissez une fonction qui s’occupe de la validation.

```javascript
// Support party size between 6 and 20 only
async function validatePartySize(context, input){
    try {
        // Recognize the input as a number. This works for responses such as
        // "twelve" as well as "12"
        var result = Recognizers.recognizeNumber(input, Recognizers.Culture.English);
        var value = parseInt(results[0].resolution.value);

        if(value < 6) {
            throw new Error(`Party size too small.`);
        }
        else if(value > 20){
            throw new Error(`Party size too big.`);
        }
        return true; // Return the valid value
    }
    catch (err){
        await context.sendActivity(`${err.message} <br/>Please specify a number between 6 - 20.`);
        return false;
    }
}
```

---

Dans l’étape d’invite que nous voulons valider, appelez la fonction de validation avant de passer à l’invite suivante. Si la validation échoue, reposez la question.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
if (convoState.prompt == "partySize")
{
    if (await ValidatePartySize(context, context.Activity.Text))
    {
        // Save user's response in our state, ReservationInfo, which 
        // is a new class we've added to our state
        convoState.ReservationInfo.partySize = context.Activity.Text;

        // Ask next question
        convoState.prompt = "reserveName";
        await context.SendActivity("Who's name will this be under?");
    }
    else
    {
        // Ask again
        await context.SendActivity("How many people are in your party?");
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**app.js**

```javascript
// ...
if(convo.topicStates.prompt == "partySize"){
    if(await validatePartySize(context, context.activity.text)){
        // Save user's response
        convo.reservationInfo.partySize = context.activity.text;
        
        // Ask next question
        topicStates.prompt = "reserveName";
        await context.sendActivity("Who's name will this be under?");
    }
    else {
        // Ask again
        await context.sendActivity("How many people are in your party?");
    }
}
// ...
```

---

## <a name="next-step"></a>Étape suivante

Maintenant que vous savez comment gérer les états d’invite, voyons comment profiter de la bibliothèque de **dialogues** pour demander des entrées aux utilisateurs.

> [!div class="nextstepaction"]
> [Demander aux utilisateurs des entrées en utilisant la bibliothèque de dialogues](bot-builder-prompts.md)
