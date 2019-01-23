---
title: Création d’une compétence Cortana à l’aide de Node.js | Microsoft Docs
description: Découvrez les concepts de base liés à la création d’une compétence Cortana dans le kit SDK Bot Framework pour Node.js.
keywords: Bot Framework, compétence Cortana, voix, Node.js, Bot Builder, Kit de développement logiciel (SDK), concepts clés, concepts fodamentaux
author: DeniseMak
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 909294243abe00ac95e8f5d89d6babc2edc4f994
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/11/2019
ms.locfileid: "54225681"
---
# <a name="key-concepts-for-building-a-bot-for-cortana-skills-using-nodejs"></a>Concepts clés pour la création d’un robot pour Compétence Cortana à l’aide de Node.js
 
[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!NOTE]
> Cet article est un contenu préliminaire qui sera mis à jour.

Cet article présente les concepts clés de création d’une compétence Cortana dans le kit SDK Bot Framework pour Node.js. 

## <a name="what-is-a-cortana-skill"></a>Qu’est qu’une compétence Cortana ?
Une compétence Cortana est un robot que vous pouvez appeler à l’aide d’un client Cortana, tel que celui intégré à Windows 10. L’utilisateur lance le robot en disant des mots clés ou des expressions associées au robot. Vous utilisez le portail Bot Framework pour configurer les mots clés utilisés pour lancer votre robot. 

On peut se représenter Cortana comme un canal de reconnaissance vocale capable d’envoyer et recevoir des messages vocaux en plus d’une conversation textuelle. Un robot publié en tant que compétence Cortana doit être conçu tant pour la voix que pour le texte. Bot Framework fournit des méthodes permettant de spécifier Speech Synthesis Markup Language (SSML) pour définir des messages parlés que votre robot envoie.

## <a name="acknowledge-user-utterances"></a>Accuser réception d’énoncés d’utilisateur 

<!-- Establishing conversational understanding -->
<!-- Placeholder: In this section, describe how you have to write your speech to sound natural -->


Lorsque vous créez un robot de reconnaissance vocale, vous devez tenter d’établir un fondement commun et une compréhension mutuelle dans la conversation. Le robot doit « fonder » les énoncés de l’utilisateur en indiquant que celui-ci a été entendu et compris.

Les utilisateurs sont plongés dans la confusion quand un système ne parvient pas fonder leurs énoncés. Par exemple, la conversation suivante peut être une source de confusion quand le robot demande « Et ensuite ? » :

```
Agent: Did you want to review some more of your profile?

User: No.

Agent: What's next?
```

Si le robot ajoute « OK » en tant qu’accusé de réception, c’est plus convivial pour l’utilisateur :

```
Agent: Did you want to review some more of your profile?

User: No.

Agent: **Okay**, what's next?
```


Degrés de fondement, du plus faible au plus fort :
1. Attention continue
2. Prochaine contribution pertinente
3. Accusé de réception : réponse minimale : « Oui », « OK », « d’accord », « super »
4. Démonstration : marquer la compréhension par une reformulation, une finalisation.
5. Affichage : répéter en tout ou partie.

#### <a name="acknowledgement-and-next-relevant-contribution"></a>Accusé de réception et contribution pertinents suivante
Utilisateur : ... Je dois voyager en mai.
Agent : **Et**, quel jour de mai vouliez-vous voyager ?
Utilisateur : OK, je dois être là-bas du 12 au 15 ?
Agent : **Et**, quelle est votre destination ?

#### <a name="grounding-by-demonstration"></a>Fondement par démonstration
Utilisateur : ... Je dois voyager en mai.
Agent : Et, **quel jour** de mai vouliez-vous voyager ?
Utilisateur : OK, je dois être là-bas du 12 au 15 ?
Agent : **Et**, quelle est votre destination ?


### <a name="closure"></a>Fermeture

Le robot effectuant une action doit présenter une preuve de son accomplissement.
Il est également important d’indiquer l’échec ou la compréhension. 
* Fermeture non vocale : si vous appuyez sur un bouton d’ascenseur, son témoin s’allume.
Processus en deux étapes :
* Présentation 
* Acceptation


### <a name="differences-in-content-presentation"></a>Différences de présentation du contenu
Lors de la conception de votre robot à fonctions vocale, gardez à l’esprit que le dialogue parlé diffère souvent des messages textuels que votre robot envoie.
<!-- If there are differences in what the bot will say, in the text vs the speak fields of a prompt or in a waterfall, for example, discuss them here.

## Speech

You bot uses the **session.say** method to speak to the user. The speak method has three overloads:
* If you pass only one parameter to **session.say**, it can be a text parameter.
* If you pass two parameters to **session.say**, it can take text and SSML.
* If you pass three parameters, the third parameter takes an options structure that specifies all the options you can pass to build an **IMessage** object.

```javascript
var bot = new builder.UniversalBot(connector, function (session) {
    session.say("Hello... I'm a decision making bot.'.", 
        ssml.speak("Hello. I can help you answer all of life's tough questions."));
    session.replaceDialog('rootMenu');
});

```
## Speech in messages

The **IMessage** object provides a **speak** property for SSML. It can be used to play a .wav file.

The **inputHint** property helps indicate to Cortana whether your bot is expecting input. If you're using a built-in prompt, this value is automatically set to the default of **expectingInput**.

The **inputHint** property can take the following values: 
* **expectingInput**: Indicates that the bot is actively expecting a response from the user. Cortana listens for the user to speak into the microphone.
* **acceptingInput**: Indicates that the bot is passively ready for input but is not waiting on a response. Cortana accepts input from the user if the user holds down the microphone button.
* **ignoringInput**: Cortana is ignoring input. Your bot may send this hint if it is actively processing a request and will ignore input from users until the request is complete.

Prompts can take a `speak:` or `retrySpeak` option.

```javascript
        builder.Prompts.choice(session, "Decision Options", choices, {
            listStyle: builder.ListStyle.button,
            speak: ssml.speak("How would you like me to decide?")
        });
```

Prompts.number has *ordinal support*, meaning that you can say "the last", "the first", "the next-to-last" to choose an item in a list.




## Using synonyms

<!-- Axl Rose example -->     
```javascript   
         var choices = [
            { 
                value: 'flipCoinDialog',
                action: { title: "Flip A Coin" },
                synonyms: 'toss coin|flip quarter|toss quarter'
            },
            {
                value: 'rollDiceDialog',
                action: { title: "Roll Dice" },
                synonyms: 'roll die|shoot dice|shoot die'
            },
            {
                value: 'magicBallDialog',
                action: { title: "Magic 8-Ball" },
                synonyms: 'shake ball'
            },
            {
                value: 'quit',
                action: { title: "Quit" },
                synonyms: 'exit|stop|end'
            }
        ];
        builder.Prompts.choice(session, "Decision Options", choices, {
            listStyle: builder.ListStyle.button,
            speak: ssml.speak("How would you like me to decide?")
        });
```


## <a name="configuring-your-bot"></a>Configuration de votre robot

## <a name="prompts"></a>Invites


## <a name="additional-resources"></a>Ressources supplémentaires

[CortanaGetstarted]: /cortana/getstarted
[SSMLRef]: https://msdn.microsoft.com/en-us/library/hh378377(v=office.14).aspx
