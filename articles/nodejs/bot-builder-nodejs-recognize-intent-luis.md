---
title: Reconnaître les intentions et les entités avec LUIS | Microsoft Docs
description: Intégrez un robot avec LUIS afin de détecter les intentions de l’utilisateur et de réagir de façon appropriée en déclenchant des dialogues à l’aide du module Language Understanding Intelligent Service pour Node.js.
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 03/28/2018
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 5df1352241485bf95a46fa981b9b16c3cb7e3925
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2018
ms.locfileid: "49998696"
---
# <a name="recognize-intents-and-entities-with-luis"></a>Reconnaître les intentions et les entités avec LUIS 

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

Cet article prend l’exemple d’un robot servant à prendre des notes pour montrer comment le module Language Understanding ([LUIS][LUIS]) permet à votre robot de répondre de manière appropriée aux saisies en langage naturel. Un robot détecte ce qu’un utilisateur veut faire en identifiant son **intention**. Cette intention est déterminée à partir d’entrées orales ou textuelles, ou **énoncés**. L’intention établit la correspondance entre les énoncés et les actions prises par le bot, par exemple l’appel à un dialogue. Un robot peut également avoir besoin d’extraire des **entités** qui correspondent à des mots importants dans les énoncés. Il arrive que les entités doivent respecter une intention. Dans l’exemple d’un robot prenant des notes, l’entité `Notes.Title` identifie le titre de chaque note.

## <a name="create-a-language-understanding-bot-with-bot-service"></a>Créer un bot Language Understanding avec Bot Service

1. Sur le [Portail Azure](https://portal.azure.com), sélectionnez **Créer une ressource** dans le panneau de menu, puis cliquez sur **Tout afficher**.

    ![Créer une ressource](../media/bot-builder-nodejs-use-luis/bot-service-creation.png)

2. Dans la zone de recherche, recherchez **Web App Bot**. 

    ![Créer une ressource](../media/bot-builder-nodejs-use-luis/bot-service-selection.png)

3. Dans le panneau **Bot Service**, indiquez les informations requises, puis cliquez sur **Créer**. Le service de bot et l’application LUIS sont alors déployés vers Azure. 
   * Dans **Nom de l’application**, entrez le nom de votre bot. Il sera utilisé comme sous-domaine lors du déploiement de votre bot sur le cloud (par exemple, mynotesbot.azurewebsites.net). Ce nom sert également de nom pour l’application LUIS associée à votre robot. Copiez-le pour l’utiliser ultérieurement afin de retrouver l’application LUIS associée au robot.
   * Sélectionnez l’abonnement, le [groupe de ressources](/azure/azure-resource-manager/resource-group-overview), le plan App Service et [l’emplacement](https://azure.microsoft.com/en-us/regions/).
   * Sélectionnez le modèle **Language Understanding (Node.js)** pour le champ **Modèle de bot**.

     ![Panneau Bot Service](../media/bot-builder-nodejs-use-luis/bot-service-setting-callout-template.png)

   * Cochez la case pour confirmer les conditions d’utilisation.

4. Vérifiez que le service bot a été déployé.
    * Sélectionnez Notifications (l’icône représentant une cloche qui se trouve dans la partie supérieure du Portail Azure). La notification passera de **Déploiement lancé** à **Déploiement réussi**.
    * Cliquez alors sur **Accéder à la ressource** sur la notification **Déploiement réussi**.

## <a name="try-the-bot"></a>Essayer le bot

Vérifiez que le bot a été déployé en consultant les **Notifications**. La notification passera de **Déploiement en cours…** à **Déploiement réussi**. Cliquez sur le bouton **Accéder à la ressource** pour ouvrir le panneau des ressources du bot.

Une fois le bot enregistré, cliquez sur **Test dans le chat web** pour ouvrir le volet de chat web. Tapez « Bonjour » dans la Discussion Web.

  ![Tester le bot dans la Discussion Web](../media/bot-builder-nodejs-use-luis/bot-service-web-chat.png)

Le bot répond : « Vous vous trouvez dans l’intention Salutations. Vous avez dit : "Bonjour". » Cela confirme que le bot a reçu votre message et l’a transmis à une application LUIS par défaut qu’il a créée. Celle-ci a détecté une intention de type Salutations.

## <a name="modify-the-luis-app"></a>Modifier l’application LUIS

Connectez-vous à [https://www.luis.ai](https://www.luis.ai) avec le même compte que celui que vous utilisez pour vous connecter à Azure. Cliquer sur **Mes applications**. Dans la liste des applications, trouvez l’application commençant par le **nom d’application** indiqué dans le panneau **Bot Service** lorsque vous avez créé le Bot Service. 

L’application LUIS se lance avec 4 intentions : annulation, accueil, aide et aucun. <!-- picture -->

Les étapes suivantes permettent d’ajouter les intentions Note.Create, Note.ReadAloud et Note.Delete : 

1. Cliquez sur **Prebuilt Domains** (Domaines prédéfinis) dans le coin inférieur gauche de la page. Recherchez le domaine **Note**, puis cliquez sur **Ajouter un domaine**.

2. Ce didacticiel n’utilise pas toutes les intentions comprises dans le domaine prédéfini **Note**. Dans la page des **intentions**, cliquez sur chacun des noms d’intention suivants, puis cliquez sur le bouton **Supprimer l’intention**.
   * Note.ShowNext
   * Note.DeleteNoteItem
   * Note.Confirm
   * Note.Clear
   * Note.CheckOffItem
   * Note.AddToNote

   Voici les seules intentions qui devraient rester dans l’application LUIS : 
   * Note.ReadAloud
   * Note.Create
   * Note.Delete
   * Aucun
   * Aide
   * Salutations
   * Annuler

     ![intentions affichées dans l’application LUIS](../media/bot-builder-nodejs-use-luis/luis-intent-list.png)


3.  Cliquez sur le bouton **Former** en haut à droite pour effectuer l’apprentissage de votre application.
4.  Cliquez sur **PUBLIER** dans la barre de navigation supérieure pour ouvrir la page **Publier**. Cliquez sur le bouton **Publier à l’emplacement de production**. Après avoir effectué une publication, une application LUIS se déploie sur l’URL affichée dans la colonne **Point de terminaison** de la page **Publier l’application** et la ligne qui commence par le nom de la ressource Starter_Key. Le format de l’URL est similaire à l’exemple suivant : `https://westus.api.cognitive.microsoft.com/luis/v2.0/apps/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx?subscription-key=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx&timezoneOffset=0&verbose=true&q=`. L’identifiant d’application et la clé d’abonnement dans cette URL sont les mêmes que LuisAppId et LuisAPIKey dans ** Paramètres Azure App Service > Paramètres de l’application > Paramètres d’application > Paramètres d’application**


## <a name="modify-the-bot-code"></a>Modifier le code du bot

Cliquez sur **Générer**, puis sur **Ouvrir l’éditeur de code en ligne**.

   ![Ouvrir l’éditeur de code en ligne](../media/bot-builder-nodejs-use-luis/bot-service-build.png)

Dans l’éditeur de code, ouvrez `app.js`. Le fichier contient le code suivant :

```javascript
var restify = require('restify');
var builder = require('botbuilder');
var botbuilder_azure = require("botbuilder-azure");

// Setup Restify Server
var server = restify.createServer();
server.listen(process.env.port || process.env.PORT || 3978, function () {
   console.log('%s listening to %s', server.name, server.url); 
});
  
// Create chat connector for communicating with the Bot Framework Service
var connector = new builder.ChatConnector({
    appId: process.env.MicrosoftAppId,
    appPassword: process.env.MicrosoftAppPassword,
    openIdMetadata: process.env.BotOpenIdMetadata 
});

// Listen for messages from users 
server.post('/api/messages', connector.listen());

/*----------------------------------------------------------------------------------------
* Bot Storage: This is a great spot to register the private state storage for your bot. 
* We provide adapters for Azure Table, CosmosDb, SQL Azure, or you can implement your own!
* For samples and documentation, see: https://github.com/Microsoft/BotBuilder-Azure
* ---------------------------------------------------------------------------------------- */

var tableName = 'botdata';
var azureTableClient = new botbuilder_azure.AzureTableClient(tableName, process.env['AzureWebJobsStorage']);
var tableStorage = new botbuilder_azure.AzureBotStorage({ gzipData: false }, azureTableClient);

// Create your bot with a function to receive messages from the user
// This default message handler is invoked if the user's utterance doesn't
// match any intents handled by other dialogs.
var bot = new builder.UniversalBot(connector, function (session, args) {
    session.send('You reached the default message handler. You said \'%s\'.', session.message.text);
});

bot.set('storage', tableStorage);

// Make sure you add code to validate these fields
var luisAppId = process.env.LuisAppId;
var luisAPIKey = process.env.LuisAPIKey;
var luisAPIHostName = process.env.LuisAPIHostName || 'westus.api.cognitive.microsoft.com';

const LuisModelUrl = 'https://' + luisAPIHostName + '/luis/v2.0/apps/' + luisAppId + '?subscription-key=' + luisAPIKey;

// Create a recognizer that gets intents from LUIS, and add it to the bot
var recognizer = new builder.LuisRecognizer(LuisModelUrl);
bot.recognizer(recognizer);

// Add a dialog for each intent that the LUIS app recognizes.
// See https://docs.microsoft.com/en-us/bot-framework/nodejs/bot-builder-nodejs-recognize-intent-luis 
bot.dialog('GreetingDialog',
    (session) => {
        session.send('You reached the Greeting intent. You said \'%s\'.', session.message.text);
        session.endDialog();
    }
).triggerAction({
    matches: 'Greeting'
})

bot.dialog('HelpDialog',
    (session) => {
        session.send('You reached the Help intent. You said \'%s\'.', session.message.text);
        session.endDialog();
    }
).triggerAction({
    matches: 'Help'
})

bot.dialog('CancelDialog',
    (session) => {
        session.send('You reached the Cancel intent. You said \'%s\'.', session.message.text);
        session.endDialog();
    }
).triggerAction({
    matches: 'Cancel'
})

```


> [!TIP] 
> Vous pouvez également trouver le code exemple décrit dans cet article dans [l’exemple de robot de prise de notes][NotesSample].



## <a name="edit-the-default-message-handler"></a>Modifier le gestionnaire de messages par défaut
Le robot dispose d’un gestionnaire de messages par défaut. Vous pouvez le modifier pour qu’il corresponde à ce qui suit : 
```javascript
// Create your bot with a function to receive messages from the user.
// This default message handler is invoked if the user's utterance doesn't
// match any intents handled by other dialogs.
var bot = new builder.UniversalBot(connector, function (session, args) {
    session.send("Hi... I'm the note bot sample. I can create new notes, read saved notes to you and delete notes.");

   // If the object for storing notes in session.userData doesn't exist yet, initialize it
   if (!session.userData.notes) {
       session.userData.notes = {};
       console.log("initializing userData.notes in default message handler");
   }
});
```

## <a name="handle-the-notecreate-intent"></a>Gérer l’intention Note.Create

Copiez le code suivant et collez-le à la fin de l’app.js :

```javascript
// CreateNote dialog
bot.dialog('CreateNote', [
    function (session, args, next) {
        // Resolve and store any Note.Title entity passed from LUIS.
        var intent = args.intent;
        var title = builder.EntityRecognizer.findEntity(intent.entities, 'Note.Title');

        var note = session.dialogData.note = {
          title: title ? title.entity : null,
        };
        
        // Prompt for title
        if (!note.title) {
            builder.Prompts.text(session, 'What would you like to call your note?');
        } else {
            next();
        }
    },
    function (session, results, next) {
        var note = session.dialogData.note;
        if (results.response) {
            note.title = results.response;
        }

        // Prompt for the text of the note
        if (!note.text) {
            builder.Prompts.text(session, 'What would you like to say in your note?');
        } else {
            next();
        }
    },
    function (session, results) {
        var note = session.dialogData.note;
        if (results.response) {
            note.text = results.response;
        }
        
        // If the object for storing notes in session.userData doesn't exist yet, initialize it
        if (!session.userData.notes) {
            session.userData.notes = {};
            console.log("initializing session.userData.notes in CreateNote dialog");
        }
        // Save notes in the notes object
        session.userData.notes[note.title] = note;

        // Send confirmation to user
        session.endDialog('Creating note named "%s" with text "%s"',
            note.title, note.text);
    }
]).triggerAction({ 
    matches: 'Note.Create',
    confirmPrompt: "This will cancel the creation of the note you started. Are you sure?" 
}).cancelAction('cancelCreateNote', "Note canceled.", {
    matches: /^(cancel|nevermind)/i,
    confirmPrompt: "Are you sure?"
});
```

Toutes les entités de l’énoncé sont transmises au dialogue à l’aide du paramètre `args`. La première étape de la [cascade][waterfall] appelle [EntityRecognizer.findEntity][EntityRecognizer_findEntity] pour obtenir le titre de la note de toutes les entités `Note.Title` dans la réponse LUIS. Si l’application LUIS ne détecte pas d’entité `Note.Title`, le robot demande à l’utilisateur le nom de la note. La deuxième étape de la cascade lance une invite pour demander le texte à inclure dans la note. Une fois que le robot dispose du texte de la note, la troisième étape se sert de [session.userData][session_userData] pour enregistrer la note dans un objet `notes` en utilisant le titre comme clé. Pour plus d’informations sur `session.UserData`, consultez [Gérer les données d’état](./bot-builder-nodejs-state.md). 



## <a name="handle-the-notedelete-intent"></a>Gérer l’intention Note.Delete
Tout comme pour l’intention `Note.Create`, le robot analyse le paramètre `args` pour déterminer un titre. Si aucun titre n’est détecté, le robot invite l’utilisateur à le préciser. Le titre permet de rechercher la note à supprimer dans `session.userData.notes`. 



Copiez le code suivant et collez-le à la fin de l’app.js :
```javascript
// Delete note dialog
bot.dialog('DeleteNote', [
    function (session, args, next) {
        if (noteCount(session.userData.notes) > 0) {
            // Resolve and store any Note.Title entity passed from LUIS.
            var title;
            var intent = args.intent;
            var entity = builder.EntityRecognizer.findEntity(intent.entities, 'Note.Title');
            if (entity) {
                // Verify that the title is in our set of notes.
                title = builder.EntityRecognizer.findBestMatch(session.userData.notes, entity.entity);
            }
            
            // Prompt for note name
            if (!title) {
                builder.Prompts.choice(session, 'Which note would you like to delete?', session.userData.notes);
            } else {
                next({ response: title });
            }
        } else {
            session.endDialog("No notes to delete.");
        }
    },
    function (session, results) {
        delete session.userData.notes[results.response.entity];        
        session.endDialog("Deleted the '%s' note.", results.response.entity);
    }
]).triggerAction({
    matches: 'Note.Delete'
}).cancelAction('cancelDeleteNote', "Ok - canceled note deletion.", {
    matches: /^(cancel|nevermind)/i
});
```

Le code qui gère `Note.Delete` utilise la fonction `noteCount` pour déterminer si l’objet `notes` contient des notes. 

Collez la fonction d’aide `noteCount` à la fin de `app.js`.

[!code-js[Add a helper function that returns the number of notes (JavaScript)](../includes/code/node-basicNote.js#CountNotesHelper)]

## <a name="handle-the-notereadaloud-intent"></a>Gérer l’intention de Note.ReadAloud

Copiez le code suivant et collez-le dans `app.js` après le gestionnaire de `Note.Delete` :

```javascript
// Read note dialog
bot.dialog('ReadNote', [
    function (session, args, next) {
        if (noteCount(session.userData.notes) > 0) {
           
            // Resolve and store any Note.Title entity passed from LUIS.
            var title;
            var intent = args.intent;
            var entity = builder.EntityRecognizer.findEntity(intent.entities, 'Note.Title');
            if (entity) {
                // Verify it's in our set of notes.
                title = builder.EntityRecognizer.findBestMatch(session.userData.notes, entity.entity);
            }
            
            // Prompt for note name
            if (!title) {
                builder.Prompts.choice(session, 'Which note would you like to read?', session.userData.notes);
            } else {
                next({ response: title });
            }
        } else {
            session.endDialog("No notes to read.");
        }
    },
    function (session, results) {        
        session.endDialog("Here's the '%s' note: '%s'.", results.response.entity, session.userData.notes[results.response.entity].text);
    }
]).triggerAction({
    matches: 'Note.ReadAloud'
}).cancelAction('cancelReadNote', "Ok.", {
    matches: /^(cancel|nevermind)/i
});

```

L’objet `session.userData.notes` passe en tant que troisième argument à `builder.Prompts.choice` afin que l’invite affiche une liste de notes à l’utilisateur.

Maintenant que vous avez ajouté des gestionnaires pour les nouvelles intentions, le code complet de `app.js` contient ce qui suit :

```javascript
var restify = require('restify');
var builder = require('botbuilder');
var botbuilder_azure = require("botbuilder-azure");

// Setup Restify Server
var server = restify.createServer();
server.listen(process.env.port || process.env.PORT || 3978, function () {
   console.log('%s listening to %s', server.name, server.url); 
});
  
// Create chat connector for communicating with the Bot Framework Service
var connector = new builder.ChatConnector({
    appId: process.env.MicrosoftAppId,
    appPassword: process.env.MicrosoftAppPassword,
    openIdMetadata: process.env.BotOpenIdMetadata 
});

// Listen for messages from users 
server.post('/api/messages', connector.listen());

/*----------------------------------------------------------------------------------------
* Bot Storage: This is a great spot to register the private state storage for your bot. 
* We provide adapters for Azure Table, CosmosDb, SQL Azure, or you can implement your own!
* For samples and documentation, see: https://github.com/Microsoft/BotBuilder-Azure
* ---------------------------------------------------------------------------------------- */

var tableName = 'botdata';
var azureTableClient = new botbuilder_azure.AzureTableClient(tableName, process.env['AzureWebJobsStorage']);
var tableStorage = new botbuilder_azure.AzureBotStorage({ gzipData: false }, azureTableClient);

// Create your bot with a function to receive messages from the user.
// This default message handler is invoked if the user's utterance doesn't
// match any intents handled by other dialogs.
var bot = new builder.UniversalBot(connector, function (session, args) {
    session.send("Hi... I'm the note bot sample. I can create new notes, read saved notes to you and delete notes.");

   // If the object for storing notes in session.userData doesn't exist yet, initialize it
   if (!session.userData.notes) {
       session.userData.notes = {};
       console.log("initializing userData.notes in default message handler");
   }
});

bot.set('storage', tableStorage);

// Make sure you add code to validate these fields
var luisAppId = process.env.LuisAppId;
var luisAPIKey = process.env.LuisAPIKey;
var luisAPIHostName = process.env.LuisAPIHostName || 'westus.api.cognitive.microsoft.com';

const LuisModelUrl = 'https://' + luisAPIHostName + '/luis/v2.0/apps/' + luisAppId + '?subscription-key=' + luisAPIKey;

// Create a recognizer that gets intents from LUIS, and add it to the bot
var recognizer = new builder.LuisRecognizer(LuisModelUrl);
bot.recognizer(recognizer);

// CreateNote dialog
bot.dialog('CreateNote', [
    function (session, args, next) {
        // Resolve and store any Note.Title entity passed from LUIS.
        var intent = args.intent;
        var title = builder.EntityRecognizer.findEntity(intent.entities, 'Note.Title');

        var note = session.dialogData.note = {
          title: title ? title.entity : null,
        };
        
        // Prompt for title
        if (!note.title) {
            builder.Prompts.text(session, 'What would you like to call your note?');
        } else {
            next();
        }
    },
    function (session, results, next) {
        var note = session.dialogData.note;
        if (results.response) {
            note.title = results.response;
        }

        // Prompt for the text of the note
        if (!note.text) {
            builder.Prompts.text(session, 'What would you like to say in your note?');
        } else {
            next();
        }
    },
    function (session, results) {
        var note = session.dialogData.note;
        if (results.response) {
            note.text = results.response;
        }
        
        // If the object for storing notes in session.userData doesn't exist yet, initialize it
        if (!session.userData.notes) {
            session.userData.notes = {};
            console.log("initializing session.userData.notes in CreateNote dialog");
        }
        // Save notes in the notes object
        session.userData.notes[note.title] = note;

        // Send confirmation to user
        session.endDialog('Creating note named "%s" with text "%s"',
            note.title, note.text);
    }
]).triggerAction({ 
    matches: 'Note.Create',
    confirmPrompt: "This will cancel the creation of the note you started. Are you sure?" 
}).cancelAction('cancelCreateNote', "Note canceled.", {
    matches: /^(cancel|nevermind)/i,
    confirmPrompt: "Are you sure?"
});

// Delete note dialog
bot.dialog('DeleteNote', [
    function (session, args, next) {
        if (noteCount(session.userData.notes) > 0) {
            // Resolve and store any Note.Title entity passed from LUIS.
            var title;
            var intent = args.intent;
            var entity = builder.EntityRecognizer.findEntity(intent.entities, 'Note.Title');
            if (entity) {
                // Verify that the title is in our set of notes.
                title = builder.EntityRecognizer.findBestMatch(session.userData.notes, entity.entity);
            }
            
            // Prompt for note name
            if (!title) {
                builder.Prompts.choice(session, 'Which note would you like to delete?', session.userData.notes);
            } else {
                next({ response: title });
            }
        } else {
            session.endDialog("No notes to delete.");
        }
    },
    function (session, results) {
        delete session.userData.notes[results.response.entity];        
        session.endDialog("Deleted the '%s' note.", results.response.entity);
    }
]).triggerAction({
    matches: 'Note.Delete'
}).cancelAction('cancelDeleteNote', "Ok - canceled note deletion.", {
    matches: /^(cancel|nevermind)/i
});


// Read note dialog
bot.dialog('ReadNote', [
    function (session, args, next) {
        if (noteCount(session.userData.notes) > 0) {
           
            // Resolve and store any Note.Title entity passed from LUIS.
            var title;
            var intent = args.intent;
            var entity = builder.EntityRecognizer.findEntity(intent.entities, 'Note.Title');
            if (entity) {
                // Verify it's in our set of notes.
                title = builder.EntityRecognizer.findBestMatch(session.userData.notes, entity.entity);
            }
            
            // Prompt for note name
            if (!title) {
                builder.Prompts.choice(session, 'Which note would you like to read?', session.userData.notes);
            } else {
                next({ response: title });
            }
        } else {
            session.endDialog("No notes to read.");
        }
    },
    function (session, results) {        
        session.endDialog("Here's the '%s' note: '%s'.", results.response.entity, session.userData.notes[results.response.entity].text);
    }
]).triggerAction({
    matches: 'Note.ReadAloud'
}).cancelAction('cancelReadNote', "Ok.", {
    matches: /^(cancel|nevermind)/i
});


// Helper function to count the number of notes stored in session.userData.notes
function noteCount(notes) {

    var i = 0;
    for (var name in notes) {
        i++;
    }
    return i;
}
```

## <a name="test-the-bot"></a>Tester le bot

Dans le Portail Azure, cliquez sur **Test dans le chat web** pour tester le bot. Essayez de saisir des messages comme « Créer une note », « Lire mes notes » ou « Effacer des notes » pour appeler les intentions que vous venez d’ajouter.
   ![Test du bot de prise de notes dans la Discussion Web](../media/bot-builder-nodejs-use-luis/bot-service-test-notebot.png)

> [!TIP]
> Si vous trouvez que votre bot ne reconnaît pas toujours la bonne intention ou les bonnes entités, améliorez les performances de votre application LUIS en lui donnant d’autres exemples d’énoncés pour l’entraîner. Ce nouvel apprentissage est possible sans aucune modification du code du bot. Voir [Ajouter des exemples d’énoncés](/azure/cognitive-services/LUIS/add-example-utterances) et [Entraîner et tester votre application LUIS](/azure/cognitive-services/LUIS/train-test).


## <a name="next-steps"></a>Étapes suivantes

Le test du robot permet de vérifier que la reconnaissance est capable de déclencher l’interruption du dialogue en cours. L’autorisation et la gestion des interruptions correspondent à une conception flexible qui tient compte de ce que les utilisateurs font réellement. En savoir plus sur les différentes actions que vous pouvez associer à une intention reconnue.

> [!div class="nextstepaction"]
> [Gérer les actions de l’utilisateur](bot-builder-nodejs-dialog-actions.md)


[LUIS]: https://www.luis.ai/

[intentDialog]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.intentdialog.html

[intentDialog_matches]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.intentdialog.html#matches 

[NotesSample]: https://github.com/Microsoft/BotFramework-Samples/tree/master/docs-samples/Node/basics-naturalLanguage

[triggerAction]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.dialog.html#triggeraction

[confirmPrompt]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.itriggeractionoptions.html#confirmprompt

[waterfall]: bot-builder-nodejs-dialog-manage-conversation-flow.md#manage-conversation-flow-with-a-waterfall

[session_userData]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session.html#userdata

[EntityRecognizer_findEntity]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.entityrecognizer.html#findentity

[matches]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.itriggeractionoptions.html#matches

[LUISAzureDocs]: /azure/cognitive-services/LUIS/Home

[Dialog]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.dialog.html

[IntentRecognizerSetOptions]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.iintentrecognizersetoptions.html

[LuisRecognizer]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.luisrecognizer

[LUISConcepts]: https://docs.botframework.com/en-us/node/builder/guides/understanding-natural-language/

[DisambiguationSample]: https://aka.ms/v3-js-onDisambiguateRoute

[IDisambiguateRouteHandler]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.idisambiguateroutehandler.html

[RegExpRecognizer]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.regexprecognizer.html

[AlarmBot]: https://aka.ms/v3-js-luisSample

[UniversalBot]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.universalbot.html
