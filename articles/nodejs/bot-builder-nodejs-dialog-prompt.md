---
title: Demander par invite aux utilisateurs d’entrer des informations | Microsoft Docs
description: Découvrez comment utiliser les invites pour recueillir des entrées utilisateur avec le kit SDK Bot Builder pour Node.js.
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: aa20dc396b68ede3271d12a8deab2e673a79d1d1
ms.sourcegitcommit: 2dc75701b169d822c9499e393439161bc87639d2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/24/2018
ms.locfileid: "42904481"
---
# <a name="prompt-for-user-input"></a>Demander par invite aux utilisateurs d’entrer des informations

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

Le kit SDK Bot Builder pour Node.js fournit un ensemble d’invites intégrées pour simplifier la collecte d’entrées auprès d’un utilisateur. 

Une *invite* est utilisée chaque fois qu’un bot a besoin d’une entrée provenant de l’utilisateur. Vous pouvez utiliser les invites pour demander à un utilisateur toute une série d’entrées en chaînant ces invites en cascade. Vous pouvez vous servir des invites conjointement avec une [cascade](bot-builder-nodejs-dialog-waterfall.md) pour vous aider à [gérer le flux de la conversation](bot-builder-nodejs-manage-conversation-flow.md) dans votre bot. 

Cet article vous aide à comprendre le fonctionnement des invites, et la façon dont vous pouvez les utiliser pour collecter des informations auprès des utilisateurs.

## <a name="prompts-and-responses"></a>Invites et réponses

Chaque fois que vous avez besoin de l’entrée d’un utilisateur, vous pouvez envoyer une invite, attendre que l’utilisateur réponde par une entrée, puis traiter cette entrée et envoyer une réponse à l’utilisateur.

L’exemple de code suivant invite l’utilisateur à indiquer son nom, puis lui répond par un message de bienvenue.

```javascript
bot.dialog('greetings', [
    // Step 1
    function (session) {
        builder.Prompts.text(session, 'Hi! What is your name?');
    },
    // Step 2
    function (session, results) {
        session.endDialog(`Hello ${results.response}!`);
    }
]);
```

À l’aide de cette construction de base, vous pouvez modéliser votre flux de conversation en ajoutant autant d’invites et de réponses qui sont utiles à votre bot.

## <a name="prompt-results"></a>Résultats des invites 

Les invites intégrées sont implémentées en tant que [dialogues](bot-builder-nodejs-dialog-overview.md) retournant la réponse de l’utilisateur dans le champ `results.response`. Pour les objets JSON, les réponses sont retournées dans le champ `results.response.entity`. N’importe quel type de [Gestionnaire de boîte de dialogue](bot-builder-nodejs-dialog-overview.md#dialog-handlers) peut recevoir le résultat d’une invite. Lorsque le bot reçoit une réponse, il peut la consommer, ou la retransmettre au dialogue appelant par un appel à la méthode [`session.endDialogWithResult`][EndDialogWithResult].

L’exemple de code suivant montre comment retourner un résultat d’invite au dialogue appelant, par le biais de la méthode `session.endDialogWithResult`. Dans cet exemple, le dialogue `greetings` utilise le résultat de l’invite que le dialogue `askName` retourne pour accueillir l’utilisateur par son nom.

```javascript
// Ask the user for their name and greet them by name.
bot.dialog('greetings', [
    function (session) {
        session.beginDialog('askName');
    },
    function (session, results) {
        session.endDialog(`Hello ${results.response}!`);
    }
]);
bot.dialog('askName', [
    function (session) {
        builder.Prompts.text(session, 'Hi! What is your name?');
    },
    function (session, results) {
        session.endDialogWithResult(results);
    }
]);
```

## <a name="prompt-types"></a>Types d’invites
Le kit SDK Bot Builder pour Node.js inclut différents types d’invites intégrées. 

|**Type d’invite**     | **Description** |     
| ------------------ | --------------- |
|[Prompts.text](#promptstext) | Invite l’utilisateur à entrer une chaîne de texte. |     
|[Prompts.confirm](#promptsconfirm) | Invite l’utilisateur à confirmer une action.| 
|[Prompts.number](#promptsnumber) | Invite l’utilisateur à entrer un nombre.     |
|[Prompts.time](#promptstime) | Invite l’utilisateur à indiquer une heure ou une date/heure.      |
|[Prompts.choice](#promptschoice) | Invite l’utilisateur à choisir parmi une liste d’options.    |
|[Prompts.attachment](#promptsattachment) | Invite l’utilisateur à charger une image ou une vidéo.|       

Les sections suivantes fournissent des détails supplémentaires sur chaque type d’invite.

### <a name="promptstext"></a>Prompts.text

Utilisez la méthode [Prompts.text()][PromptsText] pour demander à l’utilisateur d’entrer une **chaîne de texte**. L’invite retourne la réponse de l’utilisateur en tant que [IPromptTextResult][IPromptTextResult].

```javascript
builder.Prompts.text(session, "What is your name?");
```

### <a name="promptsconfirm"></a>Prompts.confirm

Utilisez la méthode [Prompts.confirm()][PromptsConfirm] pour demander à l’utilisateur de confirmer une action avec une réponse **oui/non**. L’invite retourne la réponse de l’utilisateur en tant que [IPromptConfirmResult](http://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.ipromptconfirmresult.html).

```javascript
builder.Prompts.confirm(session, "Are you sure you wish to cancel your order?");
```

### <a name="promptsnumber"></a>Prompts.number

Utilisez la méthode [Prompts.number()][PromptsNumber] pour demander à l’utilisateur d’indiquer un **nombre**. L’invite retourne la réponse de l’utilisateur en tant que [IPromptNumberResult](http://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.ipromptnumberresult.html).

```javascript
builder.Prompts.number(session, "How many would you like to order?");
```

### <a name="promptstime"></a>Prompts.time

Utilisez la méthode [Prompts.time()][PromptsTime] pour demander à l’utilisateur de préciser une **heure** ou une **date/heure**. L’invite retourne la réponse de l’utilisateur en tant que [IPromptTimeResult](http://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.iprompttimeresult.html). L’infrastructure utilise la bibliothèque [Chrono](https://github.com/wanasit/chrono) pour analyser la réponse de l’utilisateur, et prend en charge les réponses relatives (par exemple, « dans 5 minutes ») et les réponses non relatives (par exemple, « 6 juin à 14 h 00 »).

Le champ [results.response](http://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.iprompttimeresult.html#response), qui représente la réponse de l’utilisateur, contient un objet [entity](http://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.ientity.html) qui spécifie la date et l’heure. Pour résoudre la date et l’heure dans un objet `Date` JavaScript, utilisez la méthode [EntityRecognizer.resolveTime()](http://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.entityrecognizer.html#resolvetime).

> [!TIP] 
> L’heure entrée par l’utilisateur est convertie en heure UTC en fonction du fuseau horaire auquel appartient le serveur qui héberge le bot. Étant donné que le serveur peut se trouver dans un autre fuseau horaire que celui de l’utilisateur, veillez à tenir compte des fuseaux horaires. Pour convertir la date et l’heure en heure locale de l’utilisateur, prévoyez de demander à l’utilisateur le fuseau horaire auquel il appartient.

```javascript
bot.dialog('createAlarm', [
    function (session) {
        session.dialogData.alarm = {};
        builder.Prompts.text(session, "What would you like to name this alarm?");
    },
    function (session, results, next) {
        if (results.response) {
            session.dialogData.name = results.response;
            builder.Prompts.time(session, "What time would you like to set an alarm for?");
        } else {
            next();
        }
    },
    function (session, results) {
        if (results.response) {
            session.dialogData.time = builder.EntityRecognizer.resolveTime([results.response]);
        }

        // Return alarm to caller  
        if (session.dialogData.name && session.dialogData.time) {
            session.endDialogWithResult({ 
                response: { name: session.dialogData.name, time: session.dialogData.time } 
            }); 
        } else {
            session.endDialogWithResult({
                resumed: builder.ResumeReason.notCompleted
            });
        }
    }
]);
```

### <a name="promptschoice"></a>Prompts.choice

Utilisez la méthode [Prompts.choice()][PromptsChoice] pour demander à l’utilisateur de **choisir parmi une liste d’options**. L’utilisateur peut transmettre sa sélection soit en entrant le numéro associé à l’option de son choix, soit en entrant le nom de l’option qu’il choisit. Les correspondances partielles ou totales du nom de l’option sont prises en charge. L’invite retourne la réponse de l’utilisateur en tant que [IPromptChoiceResult](http://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.ipromptchoiceresult.html). 

Pour spécifier le style de la liste qui est présentée à l’utilisateur, définissez la propriété [IPromptOptions.listStyle](http://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.ipromptoptions.html#liststyle). Le tableau suivant présente les valeurs d’énumération `ListStyle` de cette propriété.


Les valeurs enum `ListStyle` sont les suivantes :

| Index | NOM | Description |
| ---- | ---- | ---- |
| 0 | Aucun | Aucune liste n’est restituée. S’utilise lorsque la liste fait partie intégrante de l’invite. |
| 1 | inline | Les choix sont restitués sous la forme d’une liste incorporée du formulaire « 1. rouge, 2. vert ou 3. bleu. » |
| 2 | list | Les choix sont restitués sous la forme d’une liste numérotée. |
| 3 | button | Les choix sont restitués sous la forme de boutons pour les canaux qui prennent en charge des boutons. Pour les autres canaux, ils sont restitués sous forme de texte. |
| 4 | auto | Le style est sélectionné automatiquement en fonction du canal et du nombre d’options. | 

Vous pouvez accéder à cette énumération à partir de l’objet `builder`, ou vous pouvez fournir un index pour choisir un `ListStyle`. Par exemple, les deux instructions dans l’extrait de code suivant accomplissent la même chose.

```javascript
// ListStyle passed in as Enum
builder.Prompts.choice(session, "Which color?", "red|green|blue", { listStyle: builder.ListStyle.button });

// ListStyle passed in as index
builder.Prompts.choice(session, "Which color?", "red|green|blue", { listStyle: 3 });
```

Pour spécifier la liste des options, vous pouvez utiliser une chaîne (`|`) séparées par des barres verticales, un tableau de chaînes ou un mappage d’objets.

Une chaîne séparée par des barres verticales : 

```javascript
builder.Prompts.choice(session, "Which color?", "red|green|blue");
```

Un tableau de chaînes :

```javascript
builder.Prompts.choice(session, "Which color?", ["red","green","blue"]);
```

Un mappage d’objets : 

```javascript
var salesData = {
    "west": {
        units: 200,
        total: "$6,000"
    },
    "central": {
        units: 100,
        total: "$3,000"
    },
    "east": {
        units: 300,
        total: "$9,000"
    }
};

bot.dialog('getSalesData', [
    function (session) {
        builder.Prompts.choice(session, "Which region would you like sales for?", salesData); 
    },
    function (session, results) {
        if (results.response) {
            var region = salesData[results.response.entity];
            session.send(`We sold ${region.units} units for a total of ${region.total}.`); 
        } else {
            session.send("OK");
        }
    }
]);
```

### <a name="promptsattachment"></a>Prompts.attachment

Utilisez la méthode [Prompts.attachment()][PromptsAttachment] pour demander à l’utilisateur de charger un fichier, tel qu’une image ou une vidéo. L’invite retourne la réponse de l’utilisateur en tant que [IPromptChoiceResult](http://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.ipromptattachmentresult.html).

```javascript
builder.Prompts.attachment(session, "Upload a picture for me to transform.");
```

## <a name="next-steps"></a>Étapes suivantes

À présent que vous savez piloter les utilisateurs entre les différentes étapes d’une cascade, et les inviter à fournir des informations, voyons comment vous pouvez améliorer votre gestion du flux de la conversation.

> [!div class="nextstepaction"]
> [Gérer le flux de la conversation](bot-builder-nodejs-dialog-manage-conversation-flow.md)


[SendAttachments]: bot-builder-nodejs-send-receive-attachments.md
[SendCardWithButtons]: bot-builder-nodejs-send-rich-cards.md
[RecognizeUserIntent]: bot-builder-nodejs-recognize-intent-messages.md
[SaveUserData]: bot-builder-nodejs-save-user-data.md

[UniversalBot]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.universalbot.html
[ChatConnector]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.chatconnector.html
[Session]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session


[SendTyping]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session#sendtyping

[EndDialogWithResult]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session.html#enddialogwithresult

[IPromptResult]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.ipromptresult.html

[Result_Response]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.ipromptresult.html#reponse

[ResumeReason]: https://docs.botframework.com/en-us/node/builder/chat-reference/enums/_botbuilder_d_.resumereason.html

[Result_Resumed]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.ipromptresult.html#resumed

[entity]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.ientity.html

[ResolveTime]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.entityrecognizer.html#resolvetime

[PromptsRef]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.__global.iprompts.html

[PromptsText]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.__global.iprompts.html#text

[IPromptTextResult]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.iprompttextresult.html

[PromptsConfirm]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.__global.iprompts.html#confirm

[IPromptConfirmResult]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.ipromptconfirmresult.html

[PromptsNumber]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.__global.iprompts.html#number

[IPromptNumberResult]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.ipromptnumberresult.html

[PromptsTime]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.__global.iprompts.html#time

[IPromptTimeResult]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.iprompttimeresult.html

[PromptsChoice]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.__global.iprompts.html#choice

[IPromptChoiceResult]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.ipromptchoiceresult.html

[PromptsAttachment]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.__global.iprompts.html#attachment

[IPromptAttachmentResult]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.ipromptattachmentresult.html
