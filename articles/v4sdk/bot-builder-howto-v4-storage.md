---
title: Stocker des données | Microsoft Docs
description: Découvrez comment écrire directement dans le stockage avec la version 4 du kit de développement logiciel Bot Builder pour .NET.
keywords: stockage, lecture et écriture, stockage de données, stockage de mémoire, étiquette d’entité
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 05/2/18
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 653ec6a1983dd59c485a91b2c08ea07d9f2a34c8
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39299680"
---
# <a name="save-data-directly-to-storage"></a>Enregistrer des données directement dans le stockage

<!--
 Note for V4: You can write directly to storage without using the state manager. Therefore, this topic isn't called "managing state". State is in a separate topic.
 ## Manage state data by writing directly to storage
-->
[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Vous pouvez écrire directement dans le stockage sans utiliser d’objet contextuel ou d’intergiciel. Il suffit de lire et d’écrire directement sur votre objet de stockage.

Cet exemple de code vous montre comment lire et écrire des données dans le stockage. Dans cet exemple, le stockage est un fichier, mais vous pouvez facilement modifier le code afin d’initialiser l’objet de stockage pour utiliser la mémoire de Stockage Table à la place. 

# <a name="ctabcsharpechorproperty"></a>[C#](#tab/csharpechorproperty)
Nous allons définir un objet et utiliser `IStorage.Write` et `IStorage.Read` pour sauvegarder et récupérer l’état. 

L’exemple suivant permet d’ajouter tous les messages de l’utilisateur à une liste. La structure de données de la liste est sauvegardée dans un fichier du répertoire que vous fournissez au constructeur `FileStorage`.

Commencez avec le modèle EchoBot Visual Studio dans le kit de développement logiciel BotBuilder V4.
Modifiez le fichier `EchoBot.cs` . Ajoutez ce qui suit à l’aide des instructions, puis remplacez la définition de la classe `EchoBot`.

```csharp
using System.Collections.Generic;
using System.Linq;
```

```csharp
// In the constructor initialize file storage
public class EchoBot : IBot
{
    private readonly FileStorage _myStorage;

    public EchoBot()
    {
        _myStorage = new FileStorage(System.IO.Path.GetTempPath());
    }

    // Add a class for storing a log of utterances (text of messages) as a list
    public class UtteranceLog : IStoreItem
    {
        // A list of things that users have said to the bot
        public List<string> UtteranceList { get; private set; } = new List<string>();

        // The number of conversational turns that have occurred        
        public int TurnNumber { get; set; } = 0;

        public string eTag { get; set; } = "*";
    }

    // Replace the OnTurn in EchoBot.cs with the following:
    public async Task OnTurn(ITurnContext context)
    {
        var activityType = context.Activity.Type;

        await context.SendActivity($"Activity type: {context.Activity.Type}.");

        if (activityType == ActivityTypes.Message)
        {
            // *********** begin (create or add to log of messages)
            var utterance = context.Activity.Text;
            bool restartList = false;

            if (utterance.Equals("restart"))
            {
                restartList = true;
            }

            // Attempt to read the existing property bag
            UtteranceLog logItems = null;
            try
            {
                logItems = _myStorage.Read<UtteranceLog>("UtteranceLog").Result?.FirstOrDefault().Value;
            }
            catch (System.Exception ex)
            {
                await context.SendActivity(ex.Message);
            }

            // If the property bag wasn't found, create a new one
            if (logItems is null)
            {
                try
                {
                    // add the current utterance to a new object.
                    logItems = new UtteranceLog();
                    logItems.UtteranceList.Add(utterance);

                    await context.SendActivity($"The list is now: {string.Join(", ",logItems.UtteranceList)}");

                    var changes = new KeyValuePair<string, object>[]
                    {
                        new KeyValuePair<string, object>("UtteranceLog", logItems)
                    };
                    await _myStorage.Write(changes);
                }
                catch (System.Exception ex)
                {
                    await context.SendActivity(ex.Message);
                }
            }
            // logItems.ContainsKey("UtteranceLog") == true, we were able to read a log from storage
            else
            {
                // Modify its property
                if (restartList)
                {
                    logItems.UtteranceList.Clear();
                }
                else
                {
                    logItems.UtteranceList.Add(utterance);
                    logItems.TurnNumber++;
                }

                await context.SendActivity($"The list is now: {string.Join(", ", logItems.UtteranceList)}");

                var changes = new KeyValuePair<string, object>[]
                {
                        new KeyValuePair<string, object>("UtteranceLog", logItems)
                };
                await _myStorage.Write(changes);
            }
        }

        return;
    }
}
```
# <a name="javascripttabjsechoproperty"></a>[JavaScript](#tab/jsechoproperty)

L’exemple suivant permet d’ajouter tous les messages de l’utilisateur à une liste. La structure de données de la liste est sauvegardée dans un fichier du répertoire que vous fournissez au constructeur `FileStorage`.

Collez ce qui suit dans `app.js`.
``` javascript
const { BotFrameworkAdapter, FileStorage, MemoryStorage, ConversationState, BotStateSet } = require('botbuilder');
const restify = require('restify');

// Create server.
let server = restify.createServer();
server.listen(process.env.port || process.env.PORT || 3978, function () {
    console.log(`${server.name} listening to ${server.url}`);
});

// Create adapter.
const adapter = new BotFrameworkAdapter({
    appId: process.env.MICROSOFT_APP_ID,
    appPassword: process.env.MICROSOFT_APP_PASSWORD
});

// Add storage.
var storage = new FileStorage("C:/temp");
const conversationState = new ConversationState(storage);
adapter.use(new BotStateSet(conversationState));

// Listen for incoming activities.
server.post('/api/messages', (req, res) => {
    // Route received activity to adapter for processing.
    adapter.processActivity(req, res, async (context) => {
        if (context.activity.type === 'message') {
            const state = conversationState.get(context);
            const count = state.count === undefined ? state.count = 0 : ++state.count;

            let utterance = context.activity.text;
            let storeItems = await storage.read(["UtteranceLog"])
            try {
                // check result
                var utteranceLog = storeItems["UtteranceLog"];
                
                if (typeof (utteranceLog) != 'undefined') {
                    // log exists so we can write to it
                    storeItems["UtteranceLog"].UtteranceList.push(utterance);
                    
                    await storage.write(storeItems)
                    try {
                        context.sendActivity('Successful write.');
                    } catch (err) {
                        context.sendActivity(`Srite failed of UtteranceLog: ${err}`);
                    }

                } else {
                    context.sendActivity(`need to create new utterance log`);
                    storeItems["UtteranceLog"] = { UtteranceList: [`${utterance}`], "eTag": "*" }
                    await storage.write(storeItems)
                    try {
                        context.sendActivity('Successful write.');
                    } catch (err) {
                        context.sendActivity(`Write failed: ${err}`);
                    }
                }
            } catch (err) {
                context.sendActivity(`Read rejected. ${err}`);
            };

            await context.sendActivity(`${count}: You said "${context.activity.text}"`);
        } else {
            await context.sendActivity(`[${context.activity.type} event detected]`);
        }
    });
});
```
---

## <a name="manage-concurrency-using-etags"></a>Gérer la concurrence à l’aide des étiquettes d’entité

Dans l’exemple précédent, vous avez défini la propriété `eTag` sur `*`. L’élément `eTag` (étiquette d’entité) de votre objet de stockage permet de gérer la concurrence. L’élément `eTag` indique ce qu’il faut faire si une autre instance du robot modifie l’objet de stockage sur lequel votre robot écrit. 

<!-- define optimistic concurrency -->

### <a name="last-write-wins---allow-overwrites"></a>La dernière écriture prévaut : autorise les remplacements

Définissez l’étiquette d’entité à `*` pour permettre à d’autres instances du robot de remplacer les données précédemment écrites. Une valeur de propriété `eTag` de l’astérisque (`*`) indique que celui qui prévaut est le dernier qui écrit. Lorsque vous créez une nouvelle banque de données, vous pouvez définir l’élément `eTag` d’une propriété à `*` pour indiquer que vous n’avez pas précédemment enregistré les données que vous écrivez, ou que vous voulez que le dernier auteur remplace toutes les propriétés précédemment enregistrées. Si la concurrence ne constitue pas un problème pour votre robot, le remplacement sera autorisé si vous définissez la propriété `eTag` à `*` pour toutes les données que vous écrivez.

Ce cas de figure est présenté dans l’exemple de code suivant :

# <a name="ctabcsetagoverwrite"></a>[C#](#tab/csetagoverwrite)
Utilisez la classe `UtteranceLog` définie précédemment.
```csharp
// Add the current utterance to a new object and save it to storage.
logItems = new UtteranceLog();
logItems.UtteranceList.Add(utterance);
logItems.eTag = "*";

var changes = new KeyValuePair<string, object>[]
{
    new KeyValuePair<string, object>("UtteranceLog", logItems)
};
await _myStorage.Write(changes);

```
# <a name="javascripttabjstagoverwrite"></a>[JavaScript](#tab/jstagoverwrite)
Ajouter un nouvel énoncé au journal et autoriser le remplacement.

```javascript
storeItems["UtteranceLog"] = { UtteranceList: [`${utterance}`], "eTag": "*" }
await storage.write(storeItems)
```
---

### <a name="maintain-concurrency-and-prevent-overwrites"></a>Gérer la concurrence et empêcher les remplacements
Si vous souhaitez empêcher l’accès simultané à une propriété, utilisez une valeur autre que `*` pour l’élément `eTag` afin d’éviter de remplacer les modifications apportées par une autre instance du robot. Lorsque le robot tente d’enregistrer des données d’état et que l’élément `eTag` est différent de ce qui se trouve dans le stockage, le robot reçoit une réponse d’erreur avec le message `etag conflict key=`. <!-- To control concurrency of data that is stored using `IStorage`, the BotBuilder SDK checks the entity tag (ETag) for `Storage.Write()` requests. -->

Par défaut, la banque vérifie que la propriété `eTag` d’un objet de stockage reste la même chaque fois qu’un robot écrit dessus, puis l’actualise à une nouvelle valeur unique après chaque écriture. Si la propriété `eTag` d’écriture ne correspond pas à la propriété `eTag` de stockage, un autre robot ou thread a donc modifié les données. 

Par exemple, si vous souhaitez que votre robot modifie une note enregistrée, mais que vous ne voulez pas qu’il remplace les modifications apportées par une autre instance du robot. Si une autre instance du robot a apporté des modifications, vous souhaitez que l’utilisateur modifie la version avec les dernières mises à jour.

# <a name="ctabcsetag"></a>[C#](#tab/csetag)
Commencez par créer une classe qui implémente `IStoreItem`.
```csharp
public class Note : IStoreItem
{
    public string Name { get; set; }
    public string Contents { get; set; }
    public string eTag { get; set; }
}
```
Créez ensuite une première note avec un nouvel objet de stockage que vous a ajoutez à la banque.
```csharp
// create a note for the first time, with a non-null, non-* eTag.
var note = new Note { Name = "Shopping List", Contents = "eggs", eTag = "x" };

var changes = new KeyValuePair<string, object>[]
{
    new KeyValuePair<string, object>("Note", note)
};
await NoteStore.Write(changes);
```
Accédez à la note et actualisez-la ultérieurement, en gardant le `eTag` que vous lisez dans la banque.
```csharp
var note = NoteStore.Read<Note>("Note").Result.FirstOrDefault().Value;

if (note != null)
{
    note.Contents += ", bread";
    var changes = new KeyValuePair<string, object>[]
    {
        new KeyValuePair<string, object>("Note1", note)
    };
    await NoteStore.Write(changes);
}
```
Si la note a été actualisée dans la banque avant que vous n’écriviez vos modifications, l’appel à `Write` lance une exception.

# <a name="javascripttabjsetag"></a>[JavaScript](#tab/jsetag)

Commencez par créer l’objet `myNote`.
```javascript
var myNote = {
    name: "Shopping List",
    contents: "eggs",
    eTag: "*"
}
```
Initialisez ensuite un objet `changes` pour y ajouter vos *notes*, puis enregistrez-le dans le stockage.

```javascript
// Write a note
var changes = {};
changes["Note"] = myNote;
await storage.write(changes);
```
Accédez à la note et actualisez-la ultérieurement, en gardant le `eTag` que vous lisez dans la banque.
```javascript
// Read in a note
var note = await storage.read(["Note"]);
try {
    // Someone has updated the note. Need to update our local instance to match
    if(myNote.eTag != note.eTag){
        myNote = note;
    }

    // Add any updates to the note and write back out
    myNote.contents += ", bread";   // Add to the note
    changes["Note"] = note;
    await storage.write(changes); // Write the changes back to storage
    try {
        context.sendActivity('Successful write.');
    } catch (err) {
        context.sendActivity(`Write failed: ${err}`);
    }
}
catch (err) {
    context.sendActivity(`Unable to read the Note: ${err}`);
}
```
Si la note a été actualisée dans la banque avant que vous n’écriviez vos modifications, l’appel à `Write` lance une exception.

---

Pour gérer la concurrence, lisez toujours une propriété à partir du stockage, puis modifiez la propriété que vous lisez afin de conserver `eTag`. Si vous lisez les données de l’utilisateur dans la banque, la réponse contient la propriété de l’étiquette d’entité. Si vous modifiez les données et que vous enregistrez les nouvelles données dans la banque, votre requête doit inclure la propriété de l’étiquette d’entité qui indique la même valeur que celle que vous avez lue précédemment. Lorsque l’élément `eTag` d’un objet est défini à `*`, il est toutefois possible de l’enregistrer en remplaçant les autres modifications.

<!-- If the ETag specified in your `Storage.Write()` request matches the current value in the store, the server will save the data and specify a new eTag value in the body of the response, that indicates that the data has been updated. If the ETag specified in your Storage.Write() request does not match the current value in the store, the bot responds with an error indicating an eTag conflict, to indicate that the user's data in the store has changed since you last saved or retrieved it. -->

<!-- TODO: new snippet -->

<!-- jf: I think this section can be cut entirely.

## Save a conversation reference using storage

You can use IStorage to save BotBuilder SDK objects as well as user-defined data. This code snippet uses `Storage.Write()` to save a ConversationReference object for use in sending a proactive message later.

# [C#](#tab/csharpwriteconvref)
```csharp
            var reference = context.ConversationReference;
            var userId = reference.User.Id;

            // save the ConversationReference to a global variable of this class
            conversationReference = reference;

            StoreItems storeItems = new StoreItems();
            StoreItem conversationReferenceToStore = new StoreItem();
            // set the eTag to "*" to indicate you're overwriting previous data
            conversationReferenceToStore.eTag = "*";
            conversationReferenceToStore.Add("ref", reference);
            storeItems[$"ConversationReference/{userId}"] = conversationReferenceToStore;

            await storage.Write(storeItems).ConfigureAwait(false);

            SubscribeUser(userId);

            await context.SendActivity("Thank You! We will message you shortly.");
        }

public void SubscribeUser(string userId)
{
    CreateContextForUserAsync(userId, async (ITurnContext context) =>
    {
        await context.SendActivity("You've been notified.");
        await Task.Delay(2000);
    });
                
}
```
# [JavaScript](#tab/jswriteconvref)
```javascript
const { MemoryStorage } = require('botbuilder');

const storage = new MemoryStorage();

// Listen for incoming activity 
server.post('/api/messages', (req, res) => {
    // Route received activity to adapter for processing
    adapter.processActivity(req, res, async (context) => {
        if (context.activity.type === 'message') {
            const utterances = (context.activity.text || '').trim().toLowerCase()
            if (utterances === 'subscribe') {
                const reference = context.activity;
                const userId = reference.id;
                const changes = {};
                changes['reference/' + userId] = reference;
                await storage.write(changes)
                await subscribeUser(userId)
                await context.sendActivity(`Thank You! We will message you shortly.`);
               
            } else{
                await context.sendActivity("Say 'subscribe'");
            }
    
        }
    });
});
```
---

To read the saved object from storage, call `Storage.Read()`.

# [C#](#tab/csharpread)
```csharp


        private async void CreateContextForUserAsync(String userId,Func<ITurnContext, Task> onReady)
        {
            var referenceKey = $"ConversationReference/{userId}";
            
            ConversationReference localRef = null;
            StoreItems conversationReferenceFromStorage = await storage.Read(referenceKey);
            foreach (var item in conversationReferenceFromStorage)
            {
                StoreItem value = item.Value as StoreItem;
                localRef = value["ref"];
            }
            
            await bot.CreateContext(this.conversationReference, onReady);

        }
```
# [JavaScript](#tab/jsread)
```javascript
async function subscribeUser(userId) {
    setTimeout(() => {
        createContextForUser(userId, (context) => {
            context.sendActivity("You have been notified");
        })
    }, 2000);
}

async function createContextForUser(userId, callback) {
    const referenceKey = 'reference/' + userId;
    var rows = await storage.read([referenceKey])
    var reference = await rows[referenceKey]
    await callback(adapter.createContext(reference))
          
}
```
---

-->

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous savez comment lire et écrire directement dans le stockage, voyons comment effectuer ces opérations à l’aide du gestionnaire d’état.

> [!div class="nextstepaction"]
> [Enregistrer l’état à l’aide des propriétés de la conversation et de l’utilisateur](bot-builder-howto-v4-state.md)

## <a name="additional-resources"></a>Ressources supplémentaires

- [Concept : Stockage dans le kit de développement logiciel Bot Builder](bot-builder-storage-concept.md)


