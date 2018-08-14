---
title: Reconnaître les intentions et les entités avec LUIS | Microsoft Docs
description: Découvrez comment permettre à votre bot de comprendre le langage naturel à l’aide des dialogues LUIS dans le Kit de développement logiciel (SDK) Bot Builder pour .NET.
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: b98eea6bdae097beec85e93301e5380a1de991c3
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39299885"
---
# <a name="recognize-intents-and-entities-with-luis"></a>Reconnaître les intentions et les entités avec LUIS 

Cet article utilise l’exemple d’un bot de prise de notes pour démontrer la manière dont le service Language Understanding ([LUIS][LUIS]) permet à votre bot de répondre de manière appropriée aux entrées en langage naturel. Un bot détecte ce qu’un utilisateur veut faire en identifiant son **intention**. Cette intention est déterminée à partir d’entrées orales/textuelles, ou **énoncés**. L’intention établit une correspondance entre les énoncés et les actions exécutées par le bot. Par exemple, un bot de prise de notes reconnaît une intention `Notes.Create` en appelant la fonctionnalité de création d’une note. Un bot peut également avoir besoin d’extraire des **entités** qui correspondent à des mots importants dans les énoncés. Dans l’exemple de bot de prise de notes, l’entité `Notes.Title` identifie le titre de chaque note.

## <a name="create-a-language-understanding-bot-with-bot-service"></a>Créer un bot Language Understanding avec Bot Service

1. Dans le [Portail Azure](https://portal.azure.com), sélectionnez **Créer une ressource** dans le panneau de menu, puis cliquez sur **Tout afficher**.<!-- Start with the steps in [Create a bot with Bot Service](../bot-service-quickstart.md) to start creating a new bot service.  -->

    ![Créer une ressource](../media/bot-builder-dotnet-use-luis/bot-service-creation.png)

2. Dans la zone de recherche, recherchez **Web App Bot**. 

    ![Créer une ressource](../media/bot-builder-dotnet-use-luis/bot-service-selection.png)

3. Dans le panneau **Bot Service** (Service de robot), fournissez les informations requises, puis cliquez sur **Créer**. Le service de bot et l’application LUIS sont alors déployés vers Azure. 
   * Dans **Nom de l’application**, entrez le nom de votre bot. Il sera utilisé comme sous-domaine lors du déploiement de votre bot sur le cloud (par exemple, mynotesbot.azurewebsites.net). Ce nom sert également de nom pour l’application LUIS associée à votre bot. Copiez-le pour l’utiliser ultérieurement afin de retrouver l’application LUIS associée au bot.
   * Sélectionnez l’abonnement, le [groupe de ressources](/azure/azure-resource-manager/resource-group-overview), le plan App Service et [l’emplacement](https://azure.microsoft.com/en-us/regions/).
   * Sélectionnez le modèle **Language Understanding (C#)** pour le champ **Modèle de bot**.

     ![Panneau Bot Service](../media/bot-builder-dotnet-use-luis/bot-service-setting-callout-template.png)

   * Cochez la case pour confirmer les conditions d’utilisation.

4. Vérifiez que le service bot a été déployé.
    * Sélectionnez Notifications (l’icône représentant une cloche qui se trouve dans la partie supérieure du Portail Azure). La notification passera de **Déploiement lancé** à **Déploiement réussi**.
    * Cliquez alors sur **Accéder à la ressource** sur la notification **Déploiement réussi**.

## <a name="try-the-bot"></a>Essayer le bot

Vérifiez que le bot a été déployé en consultant les **Notifications**. La notification passera de **Déploiement en cours…** à **Déploiement réussi**. Cliquez sur le bouton **Accéder à la ressource** pour ouvrir le panneau des ressources du bot.

Une fois le bot enregistré, cliquez sur **Test dans le chat web** pour ouvrir le volet de chat web. Tapez « Bonjour » dans la Discussion Web.

  ![Tester le bot dans la Discussion Web](../media/bot-builder-dotnet-use-luis/bot-service-web-chat.png)

Le bot répond : « Vous vous trouvez dans l’intention Salutations. Vous avez dit : "Bonjour". » Cela confirme que le bot a reçu votre message et l’a transmis à une application LUIS par défaut qu’il a créée. Celle-ci a détecté une intention de type Salutations.

## <a name="modify-the-luis-app"></a>Modifier l’application LUIS

Connectez-vous à [https://www.luis.ai](https://www.luis.ai) avec le même compte que celui que vous utilisez pour vous connecter à Azure. Cliquer sur **Mes applications**. Dans la liste des applications, recherchez l’application commençant par le **nom d’application** que vous avez indiqué dans le panneau **Bot Service** (Service de robot) lorsque vous avez créé le service de robot. 

L’application LUIS se lance avec 4 intentions : Cancel, Greeting, Help et None. <!-- picture -->

Les étapes ci-après ajoutent les intentions Note.Create, Note.ReadAloud et Note.Delete : 

1. Cliquez sur **Prebuilt Domains** (Domaines prédéfinis) dans le coin inférieur gauche de la page. Recherchez le domaine **Note**, puis cliquez sur **Ajouter un domaine**.

2. Ce didacticiel n’utilise pas toutes les intentions figurant dans le domaine prédéfini **Note**. Dans la page **Intents** (Intentions), cliquez sur chacun des noms d’intention ci-après, puis cliquez sur le bouton **Delete Intent** (Supprimer l’intention).
   * Note.ShowNext
   * Note.DeleteNoteItem
   * Note.Confirm
   * Note.Clear
   * Note.CheckOffItem
   * Note.AddToNote

   Les seules intentions qui devraient rester dans l’application LUIS sont les suivantes : 
   * Note.ReadAloud
   * Note.Create
   * Note.Delete
   * Aucun
   * Aide
   * Salutations
   * Annuler 

     ![Intentions affichées dans l’application LUIS](../media/bot-builder-dotnet-use-luis/luis-intent-list.png)

3. Cliquez sur le bouton **Former** en haut à droite pour effectuer l’apprentissage de votre application.
4. Cliquez sur **PUBLIER** dans la barre de navigation supérieure pour ouvrir la page **Publier**. Cliquez sur le bouton **Publish to production slot** (Publier à l’emplacement de production). Après avoir effectué la publication, copiez l’URL affichée dans la colonne **Point de terminaison** de la page **Publier l’application** sur la ligne qui commence par le nom de la ressource Starter_Key. Enregistrez cette URL afin de l’utiliser ultérieurement dans le code de votre bot. Le format de cette URL ressemble à ceci : `https://westus.api.cognitive.microsoft.com/luis/v2.0/apps/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx?subscription-key=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx&timezoneOffset=0&verbose=true&q=`

## <a name="modify-the-bot-code"></a>Modifier le code du bot

Cliquez sur **Générer**, puis sur **Ouvrir l’éditeur de code en ligne**.
    ![Ouvrir l’éditeur de code en ligne](../media/bot-builder-dotnet-use-luis/bot-service-build.png)

Dans l’éditeur de code, ouvrez `BasicLuisDialog.cs`. Ce fichier contient le code ci-après pour la gestion des intentions de l’application LUIS.
```cs
using System;
using System.Configuration;
using System.Threading.Tasks;

using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Bot.Builder.Luis;
using Microsoft.Bot.Builder.Luis.Models;

namespace Microsoft.Bot.Sample.LuisBot
{
    // For more information about this template visit http://aka.ms/azurebots-csharp-luis
    [Serializable]
    public class BasicLuisDialog : LuisDialog<object>
    {
        public BasicLuisDialog() : base(new LuisService(new LuisModelAttribute(
            ConfigurationManager.AppSettings["LuisAppId"], 
            ConfigurationManager.AppSettings["LuisAPIKey"], 
            domain: ConfigurationManager.AppSettings["LuisAPIHostName"])))
        {
        }

        [LuisIntent("None")]
        public async Task NoneIntent(IDialogContext context, LuisResult result)
        {
            await this.ShowLuisResult(context, result);
        }

        // Go to https://luis.ai and create a new intent, then train/publish your luis app.
        // Finally replace "Greeting" with the name of your newly created intent in the following handler
        [LuisIntent("Greeting")]
        public async Task GreetingIntent(IDialogContext context, LuisResult result)
        {
            await this.ShowLuisResult(context, result);
        }

        [LuisIntent("Cancel")]
        public async Task CancelIntent(IDialogContext context, LuisResult result)
        {
            await this.ShowLuisResult(context, result);
        }

        [LuisIntent("Help")]
        public async Task HelpIntent(IDialogContext context, LuisResult result)
        {
            await this.ShowLuisResult(context, result);
        }

        private async Task ShowLuisResult(IDialogContext context, LuisResult result) 
        {
            await context.PostAsync($"You have reached {result.Intents[0].Intent}. You said: {result.Query}");
            context.Wait(MessageReceived);
        }
    }
}
```
### <a name="create-a-class-for-storing-notes"></a>Créer une classe pour le stockage des notes

Ajoutez l’instruction `using` ci-après au fichier BasicLuisDialog.cs.

```cs
using System.Collections.Generic;
```

Ajoutez le code ci-après au sein de la classe `BasicLuisDialog`, après la définition du constructeur.

```cs
        // Store notes in a dictionary that uses the title as a key
        private readonly Dictionary<string, Note> noteByTitle = new Dictionary<string, Note>();
        
        [Serializable]
        public sealed class Note : IEquatable<Note>
        {

            public string Title { get; set; }
            public string Text { get; set; }

            public override string ToString()
            {
                return $"[{this.Title} : {this.Text}]";
            }

            public bool Equals(Note other)
            {
                return other != null
                    && this.Text == other.Text
                    && this.Title == other.Title;
            }

            public override bool Equals(object other)
            {
                return Equals(other as Note);
            }

            public override int GetHashCode()
            {
                return this.Title.GetHashCode();
            }
        }

        // CONSTANTS        
        // Name of note title entity
        public const string Entity_Note_Title = "Note.Title";
        // Default note title
        public const string DefaultNoteTitle = "default";
```

### <a name="handle-the-notecreate-intent"></a>Gérer l’intention Note.Create
Pour gérer l’intention Note.Create, ajoutez le code ci-après à la classe `BasicLuisDialog`.

```cs
        private Note noteToCreate;
        private string currentTitle;
        [LuisIntent("Note.Create")]
        public Task NoteCreateIntent(IDialogContext context, LuisResult result)
        {
            EntityRecommendation title;
            if (!result.TryFindEntity(Entity_Note_Title, out title))
            {
                // Prompt the user for a note title
                PromptDialog.Text(context, After_TitlePrompt, "What is the title of the note you want to create?");
            }
            else
            {
                var note = new Note() { Title = title.Entity };
                noteToCreate = this.noteByTitle[note.Title] = note;

                // Prompt the user for what they want to say in the note           
                PromptDialog.Text(context, After_TextPrompt, "What do you want to say in your note?");
            }
            
            return Task.CompletedTask;
        }
        
        
        private async Task After_TitlePrompt(IDialogContext context, IAwaitable<string> result)
        {
            EntityRecommendation title;
            // Set the title (used for creation, deletion, and reading)
            currentTitle = await result;
            if (currentTitle != null)
            {
                title = new EntityRecommendation(type: Entity_Note_Title) { Entity = currentTitle };
            }
            else
            {
                // Use the default note title
                title = new EntityRecommendation(type: Entity_Note_Title) { Entity = DefaultNoteTitle };
            }

            // Create a new note object 
            var note = new Note() { Title = title.Entity };
            // Add the new note to the list of notes and also save it in order to add text to it later
            noteToCreate = this.noteByTitle[note.Title] = note;

            // Prompt the user for what they want to say in the note           
            PromptDialog.Text(context, After_TextPrompt, "What do you want to say in your note?");

        }

        private async Task After_TextPrompt(IDialogContext context, IAwaitable<string> result)
        {
            // Set the text of the note
            noteToCreate.Text = await result;
            
            await context.PostAsync($"Created note **{this.noteToCreate.Title}** that says \"{this.noteToCreate.Text}\".");
            
            context.Wait(MessageReceived);
        }
```

### <a name="handle-the-notereadaloud-intent"></a>Gérer l’intention Note.ReadAloud
Le bot peut utiliser l’intention `Note.ReadAloud` pour afficher le contenu d’une note ou de toutes les notes si le titre de la note n’est pas détecté.

Collez le code ci-après dans la classe `BasicLuisDialog`.
```cs
        [LuisIntent("Note.ReadAloud")]
        public async Task NoteReadAloudIntent(IDialogContext context, LuisResult result)
        {
            Note note;
            if (TryFindNote(result, out note))
            {
                await context.PostAsync($"**{note.Title}**: {note.Text}.");
            }
            else
            {
                // Print out all the notes if no specific note name was detected
                string NoteList = "Here's the list of all notes: \n\n";
                foreach (KeyValuePair<string, Note> entry in noteByTitle)
                {
                    Note noteInList = entry.Value;
                    NoteList += $"**{noteInList.Title}**: {noteInList.Text}.\n\n";
                }
                await context.PostAsync(NoteList);
            }

            context.Wait(MessageReceived);
        }
        
        public bool TryFindNote(string noteTitle, out Note note)
        {
            bool foundNote = this.noteByTitle.TryGetValue(noteTitle, out note); // TryGetValue returns false if no match is found.
            return foundNote;
        }
        
        public bool TryFindNote(LuisResult result, out Note note)
        {
            note = null;

            string titleToFind;

            EntityRecommendation title;
            if (result.TryFindEntity(Entity_Note_Title, out title))
            {
                titleToFind = title.Entity;
            }
            else
            {
                titleToFind = DefaultNoteTitle;
            }

            return this.noteByTitle.TryGetValue(titleToFind, out note); // TryGetValue returns false if no match is found.
        }
```

### <a name="handle-the-notedelete-intent"></a>Gérer l’intention Note.Delete
Collez le code ci-après dans la classe `BasicLuisDialog`.

```cs
        [LuisIntent("Note.Delete")]
        public async Task NoteDeleteIntent(IDialogContext context, LuisResult result)
        {
            Note note;
            if (TryFindNote(result, out note))
            {
                this.noteByTitle.Remove(note.Title);
                await context.PostAsync($"Note {note.Title} deleted");
            }
            else
            {                             
                // Prompt the user for a note title
                PromptDialog.Text(context, After_DeleteTitlePrompt, "What is the title of the note you want to delete?");                         
            }           
        }

        private async Task After_DeleteTitlePrompt(IDialogContext context, IAwaitable<string> result)
        {
            Note note;
            string titleToDelete = await result;
            bool foundNote = this.noteByTitle.TryGetValue(titleToDelete, out note);

            if (foundNote)
            {
                this.noteByTitle.Remove(note.Title);
                await context.PostAsync($"Note {note.Title} deleted");
            }
            else
            {
                await context.PostAsync($"Did not find note named {titleToDelete}.");
            }

            context.Wait(MessageReceived);
        }
```

## <a name="build-the-bot"></a>Générer le bot
Cliquez avec le bouton droit sur **build.cmd** dans l’éditeur de code, puis choisissez **Run from Console** (Exécuter à partir de la console).

   ![Exécution de build.cmd](../media/bot-builder-dotnet-use-luis/bot-service-run-console.png)

## <a name="test-the-bot"></a>Tester le bot

Dans le Portail Azure, cliquez sur **Test dans le chat web** pour tester le bot. Essayez de taper des messages tels que « Créer une note », « lire mes notes » et « supprimer des notes ».
   ![Test du bot de prise de notes dans la Discussion Web](../media/bot-builder-dotnet-use-luis/bot-service-test-notebot.png)

> [!TIP]
> Si vous trouvez que votre bot ne reconnaît pas toujours la bonne intention ou les bonnes entités, améliorez les performances de votre application LUIS en lui donnant d’autres exemples d’énoncés pour l’entraîner. Ce nouvel apprentissage est possible sans aucune modification du code du bot. Voir [Ajouter des exemples d’énoncés](/azure/cognitive-services/LUIS/add-example-utterances) et [Entraîner et tester votre application LUIS](/azure/cognitive-services/LUIS/train-test).

> [!TIP]
> Si le code de votre bot rencontre un problème, procédez aux vérifications suivantes :
> * Vous avez [généré le bot](./bot-builder-dotnet-luis-dialogs.md#build-the-bot).
> * Le code de votre bot définit un gestionnaire pour chaque intention dans votre application LUIS.

## <a name="next-steps"></a>Étapes suivantes

Le test de votre bot vous permet de comprendre la manière dont les tâches sont appelées par une intention LUIS. Toutefois, cet exemple simple n’autorise pas l’interruption du dialogue actuellement actif. L’autorisation et la gestion des interruptions telles que « help » (aide) et « cancel » (annuler) offrent une conception flexible qui tient compte de ce que les utilisateurs font réellement. Découvrez-en davantage sur l’utilisation de dialogues scorables pour permettre à vos dialogues de gérer les interruptions.

> [!div class="nextstepaction"]
> [Gestionnaires de messages globaux à l’aide de dialogues scorables](bot-builder-dotnet-scorable-dialogs.md)

## <a name="additional-resources"></a>Ressources supplémentaires

- [Dialogues](bot-builder-dotnet-dialogs.md)
- [Gérer un flux de conversation avec des dialogues](bot-builder-dotnet-manage-conversation-flow.md)
- <a href="https://www.luis.ai" target="_blank">LUIS</a>
- <a href="/dotnet/api/?view=botbuilder-3.11.0" target="_blank">Référence sur le Kit SDK Bot Builder pour .NET</a>

[LUIS]: https://www.luis.ai/
[NotesSample]: https://github.com/Microsoft/BotFramework-Samples/tree/master/docs-samples/CSharp/Simple-LUIS-Notes-Sample
[NotesSampleJSON]: https://github.com/Microsoft/BotFramework-Samples/blob/master/docs-samples/CSharp/Simple-LUIS-Notes-Sample/Notes.json
