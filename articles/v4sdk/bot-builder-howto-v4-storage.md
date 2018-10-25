---
title: Écrire directement dans le stockage | Microsoft Docs
description: Découvrez comment écrire directement dans le stockage avec le Kit de développement logiciel (SDK) Bot Builder pour .NET.
keywords: stockage, lecture et écriture, stockage de données, stockage de mémoire, étiquette d’entité
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 09/14/18
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 3c28ad1fd14fab90c558ece4736423be2cabd78b
ms.sourcegitcommit: b8bd66fa955217cc00b6650f5d591b2b73c3254b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/15/2018
ms.locfileid: "49326546"
---
# <a name="write-directly-to-storage"></a>Écrire directement dans le stockage

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Vous pouvez lire et écrire directement dans votre objet de stockage sans utiliser d’intergiciel ou d’objet de contexte. Cela peut convenir pour les données que votre bot utilise, qui proviennent d’une source en dehors du flux de la conversation de votre bot. Par exemple, supposons que votre bot permet à l’utilisateur de demander les prévisions météo et que votre bot les récupère pour une date spécifiée, en les lisant à partir d’une base de données externe. Le contenu de la base de données météo ne dépend pas des informations sur l’utilisateur ou du contexte de la conversation ; vous pouvez donc simplement le lire directement à partir du stockage au lieu d’utiliser le gestionnaire d’état. Les exemples de code dans cet article vous montrent comment lire et écrire des données dans le stockage à l’aide du **stockage mémoire**, de **Cosmos DB**, du **Stockage Blob** et du **magasin de transcription d’objets blob Azure**. 

## <a name="prerequisites"></a>Prérequis
- Si vous n’avez pas d’abonnement Azure, créez un compte [gratuit](https://azure.microsoft.com/en-us/free/) avant de commencer.
- Installez [l’émulateur](https://github.com/Microsoft/BotFramework-Emulator/releases) Bot Framework

## <a name="memory-storage"></a>Stockage mémoire

Nous allons commencer par créer un bot qui va lire et écrire des données dans le stockage mémoire. Le stockage mémoire est utilisé uniquement à des fins de test et n’est pas destiné à une utilisation en production. Veillez à définir le stockage sur Cosmos DB ou Stockage Blob avant de publier votre bot.

#### <a name="build-a-basic-bot"></a>Créer un bot de base

Le reste de cette rubrique s’appuie sur un bot Echo. Vous pouvez en créer un en [C#](../dotnet/bot-builder-dotnet-sdk-quickstart.md) ou [JS](../javascript/bot-builder-javascript-quickstart.md). Vous pouvez utiliser l’émulateur Bot Framework pour vous connecter à votre bot, communiquer avec votre bot et le tester. L’exemple suivant permet d’ajouter tous les messages de l’utilisateur à une liste. La structure de données contenant la liste est enregistrée dans votre stockage.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
using System;
using System.Threading.Tasks;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.TraceExtensions;
using Microsoft.Bot.Builder.Azure;
using Microsoft.Bot.Schema;
using System.Collections.Generic;
using System.Linq;
using System.Threading;

// Create local Memory Storage.
private static readonly MemoryStorage _myStorage = new MemoryStorage();

// Create cancellation token (used by Async Write operation).
public CancellationToken cancellationToken { get; private set; }

// Class for storing a log of utterances (text of messages) as a list.
public class UtteranceLog : IStoreItem
{
     // A list of things that users have said to the bot
     public List<string> UtteranceList { get; } = new List<string>();

     // The number of conversational turns that have occurred        
     public int TurnNumber { get; set; } = 0;

     // Create concurrency control where this is used.
     public string ETag { get; set; } = "*";
}

// Every Conversation turn for our Bot calls this method.
public async Task OnTurnAsync(ITurnContext context)
{
     
     var activityType = context.Activity.Type;
     // See if activity type for this turn is a message from the user.
     if (activityType == ActivityTypes.Message)
     {
         var utterance = context.Activity.Text;
         UtteranceLog logItems = null;
          
         // see if there are previous messages saved in sstorage.
         string[] utteranceList = { "UtteranceLog" };
         logItems = _myStorage.ReadAsync<UtteranceLog>(utteranceList).Result?.FirstOrDefault().Value;

         // If no stored messages were found, create and store a new entry.
         if (logItems is null)
         {
             logItems = new UtteranceLog();
         }
         
         // add new message to list of messages to display.
         logItems.UtteranceList.Add(utterance);
         // increment turn counter.
         logItems.TurnNumber++;
         
         // show user new list of saved messages.
         await context.SendActivityAsync($"The list is now: {string.Join(", ", logItems.UtteranceList)}");
         
         // Create Dictionary object to hold new list of messages.
         {
             changes.Add("UtteranceLog", logItems);
          };
          
          // Save new list to your Storage.
          await _myStorage.WriteAsync(changes,cancellationToken);
     }
     return;
}

```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
const { BotFrameworkAdapter, ConversationState, BotStateSet, MemoryStorage } = require('botbuilder');
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

// Add memory storage.
var storage = new MemoryStorage();

const conversationState = new ConversationState(storage);
adapter.use(conversationState);

// Listen for incoming activity .
server.post('/api/messages', (req, res) => {
    // Route received activity to adapter for processing.
    adapter.processActivity(req, res, async (context) => {
        if (context.activity.type === 'message') {
            const state = conversationState.get(context);
            const count = state.count === undefined ? state.count = 0 : ++state.count;

            await logMessageText(storage, context);

            await context.sendActivity(`${count}: You said "${context.activity.text}"`);
        } else {
            await context.sendActivity(`[${context.activity.type} event detected]`);
        }
    });
});

async function logMessageText(storage, context) {
    let utterance = context.activity.text;
    try {
        // Read from the storage.
        let storeItems = await storage.read(["UtteranceLog"])
        // Check the result.
        var utteranceLog = storeItems["UtteranceLog"];

        if (typeof (utteranceLog) != 'undefined') {
            // The log exists so we can write to it.
            storeItems["UtteranceLog"].UtteranceList.push(utterance);

            try {
                await storage.write(storeItems)
                context.sendActivity('Successful write to utterance log.');
            } catch (err) {
                context.sendActivity(`Write failed of UtteranceLog: ${err}`);
            }

         } else {
            context.sendActivity(`need to create new utterance log`);
            storeItems["UtteranceLog"] = { UtteranceList: [`${utterance}`], "eTag": "*" }

            try {
                await storage.write(storeItems)
                context.sendActivity('Successful write to log.');
            } catch (err) {
                context.sendActivity(`Write failed: ${err}`);
            }
        }
    } catch (err) {
        context.sendActivity(`Read rejected. ${err}`);
    };
}
```

---

### <a name="start-your-bot"></a>Démarrer votre robot
Exécutez votre bot en local.

### <a name="start-the-emulator-and-connect-your-bot"></a>Démarrer l’émulateur et connecter votre robot
Démarrez à présent l’émulateur, puis connectez-vous à votre bot dans l’émulateur :

1. Cliquez sur le lien **Ouvrir le bot** de l’onglet de bienvenue de l’émulateur. 
2. Sélectionnez le fichier .bot situé dans le répertoire de création du projet.

### <a name="interact-with-your-bot"></a>Interagir avec votre bot
Envoyez un message à votre bot, et le bot va répertorier les messages qu’il reçoit.
![Émulateur en cours d’exécution](../media/emulator-v4/emulator-running.png)

 
## <a name="using-cosmos-db"></a>Utilisation de Cosmos DB
Maintenant que vous avez utilisé le stockage mémoire, nous allons mettre à jour le code pour utiliser Azure Cosmos DB. Cosmos DB est un service de base de données multimodèle mondialement distribué de Microsoft. Azure Cosmos DB vous permet de faire évoluer en toute flexibilité et de façon indépendante le débit et le stockage sur n’importe quel nombre de régions géographiques Azure. Il offre des garanties en termes de débit, de latence, de disponibilité et de cohérence avec des contrats SLA complets. 

### <a name="set-up"></a>Installation
Pour utiliser Cosmos DB dans votre bot, vous devez configurer certaines choses avant d’aborder le code.

#### <a name="create-your-database-account"></a>Créer votre compte de base de données
1. Dans une nouvelle fenêtre du navigateur, connectez-vous au [portail Azure](http://portal.azure.com).
2. Cliquez sur **Créer une ressource > Bases de données > Azure Cosmos DB**
3. Sur la **page Nouveau compte**, indiquez un nom unique dans le champ **ID**. Pour **API**, sélectionnez **SQL**et fournissez les informations relatives à l’**abonnement**, à l’**emplacement** et au **groupe de ressources**.
4. Cliquez ensuite sur **Créer**.

La création du compte prend quelques minutes. Attendez que le portail affiche la page Congratulations! Your Azure Cosmos DB account was created (Félicitations ! Votre compte Azure Cosmos DB a été créé).

##### <a name="add-a-collection"></a>Ajouter une collection
1. Cliquez sur **Paramètres > Nouvelle collection**. La zone **Ajouter une collection** est affichée à l’extrême droite : il peut donc être nécessaire de faire défiler à droite pour l’afficher.

![Ajouter une collection Cosmos DB](./media/add_database_collection.png)

2. Votre nouvelle collection de bases de données, « bot-cosmos-sql-db » avec un ID de collection « bot-storage ». Nous allons utiliser ces valeurs dans notre exemple de code ci-après.

![Cosmos DB](./media/cosmos-db-sql-database.png)

3. L’URI de point de terminaison et la clé sont disponibles dans l’onglet **Clés** de vos paramètres de base de données. Ces valeurs sont nécessaires pour configurer votre code plus loin dans cet article. 

![Clés Cosmos DB](./media/comos-db-keys.png)

#### <a name="add-configuration-information"></a>Ajouter des informations de configuration
Nos données de configuration pour ajouter le stockage Cosmos DB sont simples et concises. Vous pouvez ajouter des paramètres de configuration supplémentaires à l’aide des mêmes méthodes à mesure que votre bot devient plus complexe. Cet exemple utilise les noms de collection et de base de données Cosmos DB de l’exemple ci-dessus.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
private const string CosmosServiceEndpoint = "<your-cosmos-db-URI>";
private const string CosmosDBKey = "<your-cosmos-db-account-key>";
private const string CosmosDBDatabaseName = "bot-cosmos-sql-db";
private const string CosmosDBCollectionName = "bot-storage";
```


# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Ajoutez les informations suivantes à votre fichier `.env`. 
 
```javascript
ACTUAL_SERVICE_ENDPOINT=<your database URI>
ACTUAL_AUTH_KEY=<your database key>
DATABASE=Tasks
COLLECTION=Items
```
---

#### <a name="installing-packages"></a>Installation des packages
Vérifiez que vous avez installé les packages nécessaires pour Cosmos DB

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```powershell
Install-Package Microsoft.Bot.Builder.Azure
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Vous pouvez ajouter des références à botbuilder-azure dans votre projet via npm :-->


```powershell
npm install --save botbuilder-azure 
```

Pour utiliser le fichier config .env, le bot doit inclure un package supplémentaire. Commencez par récupérer le package dotnet à partir de npm :

```powershell
npm install --save dotenv
```

---

### <a name="implementation"></a>Implémentation 

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

L’exemple de code suivant s’exécute avec le même code de bot que l’exemple de [stockage mémoire](#memory-storage) fourni ci-dessus.
L’extrait de code ci-dessous illustre une implémentation de stockage Cosmos DB pour « _myStorage_ » qui remplace le stockage mémoire local. 

```csharp
using Microsoft.Bot.Builder.Azure;

// Create access to Cosmos DB storage.
// Replaces Memory Storage with reference to Cosmos DB.
private static readonly CosmosDbStorage _myStorage = new CosmosDbStorage(new CosmosDbStorageOptions
{
   AuthKey = CosmosDBKey,
   CollectionId = CosmosDBCollectionName,
   CosmosDBEndpoint = new Uri(CosmosServiceEndpoint),
   DatabaseId = CosmosDBDatabaseName,
});
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

L’exemple de code suivant est semblable au [stockage mémoire](#memory-storage), à quelques petites différences près.

Demandez `CosmosDbStorage` issu de botbuilder-azure et configurez dotenv pour lire le fichier `.env`.

**app.js**
```javascript
const { CosmosDbStorage } = require("botbuilder-azure");
require('dotenv').config()
```

Remplacez le stockage mémoire par une référence à Cosmos DB.

```javascript
//Add CosmosDB 
const storage = new CosmosDbStorage({
    serviceEndpoint: process.env.ACTUAL_SERVICE_ENDPOINT, 
    authKey: process.env.ACTUAL_AUTH_KEY, 
    databaseId: process.env.DATABASE,
    collectionId: process.env.COLLECTION
})

const conversationState = new ConversationState(storage);
adapter.use(conversationState);
```

---

## <a name="start-your-bot"></a>Démarrer votre robot
Exécutez votre bot en local.

## <a name="start-the-emulator-and-connect-your-bot"></a>Démarrer l’émulateur et connecter votre robot
Démarrez à présent l’émulateur, puis connectez-vous à votre bot dans l’émulateur :

1. Cliquez sur le lien **Ouvrir le bot** de l’onglet de bienvenue de l’émulateur. 
2. Sélectionnez le fichier .bot situé dans le répertoire de création du projet.

## <a name="interact-with-your-bot"></a>Interagir avec votre bot
Envoyez un message à votre bot, et le bot va répertorier les messages qu’il reçoit.
![Émulateur en cours d’exécution](../media/emulator-v4/emulator-running.png)


### <a name="view-your-data"></a>Affichage de vos données
Une fois que vous avez exécuté votre bot et enregistré vos informations, nous pouvons les afficher dans le Portail Azure sous l’onglet **Explorateur de données**. 

![Exemple de l’Explorateur de données](./media/data_explorer.PNG)

### <a name="manage-concurrency-using-etags"></a>Gérer la concurrence à l’aide des étiquettes d’entité
Dans notre exemple de code de bot, nous définissons la propriété `eTag` de chaque `IStoreItem` sur `*`. L’élément `eTag` (étiquette d’entité) de votre objet de stockage est utilisé au sein de Cosmos DB pour gérer la concurrence. L’élément `eTag` indique à votre base de données ce qu’il faut faire si une autre instance du bot modifie l’objet de stockage sur lequel votre bot écrit. 

<!-- define optimistic concurrency -->

#### <a name="last-write-wins---allow-overwrites"></a>La dernière écriture prévaut : autorise les remplacements
Une valeur de propriété `eTag` de l’astérisque (`*`) indique que celui qui prévaut est le dernier qui écrit. Lorsque vous créez une nouvelle banque de données, vous pouvez définir l’élément `eTag` d’une propriété à `*` pour indiquer que vous n’avez pas précédemment enregistré les données que vous écrivez, ou que vous voulez que le dernier auteur remplace toutes les propriétés précédemment enregistrées. Si la concurrence ne constitue pas un problème pour votre bot, le remplacement est autorisé si vous définissez la propriété `eTag` sur `*` pour toutes les données que vous écrivez.

#### <a name="maintain-concurrency-and-prevent-overwrites"></a>Gérer la concurrence et empêcher les remplacements
Lorsque vous stockez vos données dans Cosmos DB, utilisez une valeur autre que `*` pour l’élément `eTag` si vous souhaitez empêcher l’accès simultané à une propriété et éviter de remplacer les modifications apportées par une autre instance du bot. Le bot reçoit une réponse d’erreur avec le message `etag conflict key=` quand il tente d’enregistrer des données d’état et que la valeur de l’élément `eTag` est différente de celle de l’élément `eTag` qui se trouve dans le stockage. <!-- To control concurrency of data that is stored using `IStorage`, the BotBuilder SDK checks the entity tag (ETag) for `Storage.Write()` requests. -->

Par défaut, le magasin Cosmos DB vérifie que la propriété `eTag` d’un objet de stockage reste la même chaque fois qu’un bot écrit dessus, puis la met à jour avec une nouvelle valeur unique après chaque écriture. Si la propriété `eTag` d’écriture ne correspond pas à la propriété `eTag` de stockage, un autre robot ou thread a donc modifié les données. 

Par exemple, si vous souhaitez que votre robot modifie une note enregistrée, mais que vous ne voulez pas qu’il remplace les modifications apportées par une autre instance du robot. Si une autre instance du robot a apporté des modifications, vous souhaitez que l’utilisateur modifie la version avec les dernières mises à jour.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Commencez par créer une classe qui implémente `IStoreItem`.

```csharp
public class Note : IStoreItem
{
    public string Name { get; set; }
    public string Contents { get; set; }
    public string ETag { get; set; }
}
```

Créez ensuite une première note avec un nouvel objet de stockage que vous a ajoutez à la banque.

```csharp
// create a note for the first time, with a non-null, non-* ETag.
var note = new Note { Name = "Shopping List", Contents = "eggs", ETag = "x" };

var changes = Dictionary<string, object>();
{
    changes.Add("Note", note);
};
await NoteStore.WriteAsync(changes, cancellationToken);
```

Accédez à la note et actualisez-la ultérieurement, en gardant le `eTag` que vous lisez dans la banque.

```csharp
var note = NoteStore.ReadAsync<Note>("Note").Result?.FirstOrDefault().Value;

if (note != null)
{
    note.Contents += ", bread";
    var changes = new Dictionary<string, object>();
    {
         changes.Add("Note1", note);
    };
    await NoteStore.WriteAsync(changes, cancellationToken);
}
```

Si la note a été actualisée dans la banque avant que vous n’écriviez vos modifications, l’appel à `Write` lance une exception.


# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Ajoutez une fonction d’assistance à la fin de votre bot, qui écrira une note de l’exemple dans un magasin de données.
Commencez par créer l’objet `myNoteData`.

```javascript
// Helper function for writing a sample note to a data store
async function createSampleNote(storage, context) {
    var myNoteData = {
        name: "Shopping List",
        contents: "eggs",
        // If any Note file is already stored, the eTag field
        // must be set to "*" in order to allow writing without first reading the stored eTag
        // otherwise you'll likely get an exception indicating an eTag conflict. 
        eTag: "*"
    }
}
```

Dans la fonction d’assistance `createSampleNote`, initialisez un objet `changes` et ajoutez vos *notes*, puis écrivez-le dans le stockage.

```javascript
// Write the note data to the "Note" key
var changes = {};
changes["Note"] = myNoteData;
// Creates a file named Note, if it doesn't already exist.
// specifying eTag= "*" will overwrite any existing contents.
// The act of writing to the file automatically updates the eTag property
// The first time you write to Note, the eTag is changed from *, and file contents will become:
//    {"name":"Shopping List","contents":"eggs","eTag":"1"}
try {
    await storage.write(changes);
    await context.sendActivity('Successful created a note.');
} catch (err) {
    await context.sendActivity(`Could not create note: ${err}`);
}
```

Pour accéder à la note et la mettre à jour plus tard, nous créons une autre fonction d’assistance accessible quand l’utilisateur tape « update note » (mettre à jour note).

```javascript
async function updateSampleNote(storage, context) {
    try {
        // Read in a note
        var note = await storage.read(["Note"]);
        console.log(`note.eTag=${note["Note"].eTag}\n note=${JSON.stringify(note)}`);
        // update the note that we just read
        note["Note"].contents += ", bread";
        console.log(`Updated note=${JSON.stringify(note)}`);

        try {
            await storage.write(note); // Write the changes back to storage
            await context.sendActivity('Successfully updated to note.');
        } catch (err) {            console.log(`Write failed: ${err}`);
        }
    }
    catch (err) {
        await context.sendActivity(`Unable to read the Note: ${err}`);
    }
}
```

Si la note a été actualisée dans la banque avant que vous n’écriviez vos modifications, l’appel à `write` lance une exception.

---

Pour gérer la concurrence, lisez toujours une propriété à partir du stockage, puis modifiez la propriété que vous lisez afin de conserver `eTag`. Si vous lisez les données de l’utilisateur dans la banque, la réponse contient la propriété de l’étiquette d’entité. Si vous modifiez les données et que vous enregistrez les nouvelles données dans la banque, votre requête doit inclure la propriété de l’étiquette d’entité qui indique la même valeur que celle que vous avez lue précédemment. Lorsque l’élément `eTag` d’un objet est défini à `*`, il est toutefois possible de l’enregistrer en remplaçant les autres modifications.

## <a name="using-blob-storage"></a>Utilisation du stockage Blob 
Le stockage Blob Azure est la solution de stockage d’objet de Microsoft pour le cloud. Le stockage Blob est optimisé pour stocker de grandes quantités de données non structurées, telles que des données texte ou binaires.

### <a name="create-your-blob-storage-account"></a>Créer votre compte de stockage Blob
Pour utiliser le stockage Blob dans votre bot, vous devez configurer certaines choses avant d’aborder le code.
1. Dans une nouvelle fenêtre du navigateur, connectez-vous au [portail Azure](http://portal.azure.com).
2. Cliquez sur **Créer une ressource > Stockage > Compte de stockage - blob, fichier, table, file d’attente**
3. Sur la **page Nouveau compte**, tapez le **nom** du compte de stockage, sélectionnez **Stockage d’objets blob** pour **Type de compte**, puis fournissez les informations relatives aux champs **Emplacement**, **Groupe de ressources** et **Abonnement**.  
4. Cliquez ensuite sur **Créer**.

![Créer un stockage Blob](./media/create-blob-storage.png)

#### <a name="add-configuration-information"></a>Ajouter des informations de configuration

Recherchez les clés de Stockage Blob dont vous avez besoin pour configurer le Stockage Blob de votre bot, comme indiqué ci-dessus :
1. Dans le Portail Azure, ouvrez votre compte Stockage Blob et sélectionnez **Paramètres > Clés d’accès**.

![Rechercher des clés de Stockage Blob](./media/find-blob-storage-keys.png)

Nous allons maintenant utiliser deux de ces clés pour fournir à notre code un accès à notre compte Stockage Blob.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
using Microsoft.Bot.Builder.Azure;
```
Mettez à jour la ligne de code qui pointe « _myStorage_ » vers votre compte Stockage Blob existant.

```csharp


private static readonly AzureBlobStorage _myStorage = new AzureBlobStorage("<your-blob-storage-account-string>", "<your-blob-storage-container-name>");
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)
```javascript
const mystorage = new BlobStorage({
   <youy_containerName>,
   <your_storageAccountOrConnectionString>,
   <your_storageAccessKey>
})
```
---

Une fois que « _myStorage_ » est défini pour pointer vers votre compte Stockage Blob, le code de votre bot va maintenant stocker et récupérer des données depuis Stockage Blob.

## <a name="start-your-bot"></a>Démarrer votre robot
Exécutez votre bot en local.

## <a name="start-the-emulator-and-connect-your-bot"></a>Démarrer l’émulateur et connecter votre robot
Démarrez à présent l’émulateur, puis connectez-vous à votre bot dans l’émulateur :

1. Cliquez sur le lien **Ouvrir le bot** de l’onglet de bienvenue de l’émulateur. 
2. Sélectionnez le fichier .bot situé dans le répertoire de création du projet.

## <a name="interact-with-your-bot"></a>Interagir avec votre bot
Envoyez un message à votre bot, et le bot va répertorier les messages qu’il reçoit.
![Émulateur en cours d’exécution](../media/emulator-v4/emulator-running.png)

### <a name="view-your-data"></a>Affichage de vos données
Une fois que vous avez exécuté votre bot et enregistré vos informations, nous pouvons les afficher sous l’onglet **Explorateur de données** du Portail Azure.

## <a name="blob-transcript-storage"></a>Stockage de transcriptions d’objets blob
Le stockage de transcriptions d’objets blob Azure fournit une option de stockage spécialisée qui vous permet d’enregistrer et de récupérer facilement des conversations d’utilisateur sous la forme d’une transcription enregistrée. Le stockage de transcriptions d’objets blob Azure est particulièrement utile pour capturer automatiquement les entrées utilisateur à examiner lors du débogage des performances de votre bot.

### <a name="set-up"></a>Installation
Le stockage de transcriptions d’objets blob Azure peut utiliser le même compte de stockage Blob créé en suivant les étapes détaillées dans les sections « _Créer votre compte de stockage Blob_ » et « _Ajouter des informations de configuration_ ». Pour cette discussion, nous avons ajouté un conteneur d’objets blob, « _mybottranscripts_ ». 

### <a name="implementation"></a>Implémentation 
Le code suivant connecte le pointeur du stockage de transcriptions « _transcriptStore_ » à votre nouveau compte de stockage de transcriptions d’objets blob Azure. Le code source permettant de stocker les conversations utilisateur indiqué ici se base sur l’exemple [Historique des conversations](https://aka.ms/bot-history-sample-code). 

```csharp
// In Startup.cs
using Microsoft.Bot.Builder.Azure;

// Enable the conversation transcript middleware.
blobStore = new AzureBlobTranscriptStore(blobStorageConfig.ConnectionString, storageContainer);
var transcriptMiddleware = new TranscriptLoggerMiddleware(blobStore);
options.Middleware.Add(transcriptMiddleware);

// In ConversationHistoryBot.cs
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Azure;
using Microsoft.Bot.Connector;
using Microsoft.Bot.Schema;

private readonly AzureBlobTranscriptStore _transcriptStore;

/// <param name="transcriptStore">Injected via ASP.NET dependency injection.</param>
public ConversationHistoryBot(AzureBlobTranscriptStore transcriptStore)
{
    _transcriptStore = transcriptStore ?? throw new ArgumentNullException(nameof(transcriptStore));
}

```

### <a name="store-user-conversations-in-azure-blob-transcripts"></a>Stocker les conversations utilisateur dans les transcriptions d’objets blob Azure
Une fois qu’un conteneur d’objets blob est disponible pour stocker les transcriptions, vous pouvez commencer à conserver les conversations de vos utilisateurs avec votre bot. Celles-ci peuvent être utilisées plus tard en tant qu’outil de débogage pour observer la façon dont les utilisateurs interagissent avec votre bot. Le code suivant conserve les entrées des conversations utilisateur lorsque l’élément activity.text reçoit le message d’entrée _!history_.


```csharp
/// <summary>
/// Every Conversation turn for our EchoBot will call this method. 
/// </summary>
/// <param name="turnContext">A <see cref="ITurnContext"/> containing all the data needed
/// for processing this conversation turn. </param>        
public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
{
    var activity = turnContext.Activity;
    if (activity.Type == ActivityTypes.Message)
    {
        if (activity.Text == "!history")
        {
           // Download the activities from the Transcript (blob store) when a request to upload history arrives.
           var connectorClient = turnContext.TurnState.Get<ConnectorClient>(typeof(IConnectorClient).FullName);
           // Get all the message type activities from the Transcript.
           string continuationToken = null;
           var count = 0;
           do
           {
               var pagedTranscript = await _transcriptStore.GetTranscriptActivitiesAsync(activity.ChannelId, activity.Conversation.Id);
               var activities = pagedTranscript.Items
                  .Where(a => a.Type == ActivityTypes.Message)
                  .Select(ia => (Activity)ia)
                  .ToList();
               
               var transcript = new Transcript(activities);

               await connectorClient.Conversations.SendConversationHistoryAsync(activity.Conversation.Id, transcript, cancellationToken: cancellationToken);

               continuationToken = pagedTranscript.ContinuationToken;
           }
           while (continuationToken != null);
```

### <a name="find-all-stored-transcripts-for-your-channel"></a>Rechercher toutes les transcriptions stockées de votre canal
Pour afficher les données que vous avez stockées, vous pouvez utiliser le code suivant pour rechercher par programme les éléments « _ConversationIDs_ » de toutes les transcriptions que vous avez stockées. Lorsque vous utilisez l’émulateur de bot pour tester votre code, sélectionner « _Start Over_ » (Recommencer) commence une nouvelle transcription avec un nouvel élément « _ConversationID_ ».

```csharp
List<string> storedTranscripts = new List<string>();
PagedResult<Transcript> pagedResult = null;
var pageSize = 0;
do
{
    pagedResult = await _transcriptStore.ListTranscriptsAsync("emulator", pagedResult?.ContinuationToken);
    
    // transcript item contains ChannelId, Created, Id.
    // save the converasationIds (Id) found by "ListTranscriptsAsync" to a local list.
    foreach (var item in pagedResult.Items)
    {
         // Make sure we store an unescaped conversationId string.
         var strConversationId = item.Id;
         storedTranscripts.Add(Uri.UnescapeDataString(strConversationId));
    }
} while (pagedResult.ContinuationToken != null);
```

### <a name="retrieve-user-conversations-from-azure-blob-transcript-storage"></a>Récupérer des conversations utilisateur à partir du stockage de transcriptions d’objets blob Azure
Une fois que les transcriptions d’interaction de bot ont été stockées dans votre magasin de transcriptions d’objets blob Azure, vous pouvez les récupérer par programme à des fins de test ou de débogage à l’aide de la méthode AzureBlobTranscriptStorage « _GetTranscriptActivities_ ». L’extrait de code suivant récupère toutes les transcriptions d’entrées utilisateur qui ont été reçues et stockées au cours des 24 heures précédentes à partir de chaque transcription stockée.


```csharp
var numTranscripts = storedTranscripts.Count();
for (int i = 0; i < numTranscripts; i++)
{
    PagedResult<IActivity> pagedActivities = null;
    do
    {
        string thisConversationId = storedTranscripts[i];
        // Find all inputs in the last 24 hours.
        DateTime yesterday = DateTime.Now.AddDays(-1);
        // Retrieve iActivities for this transcript.
        pagedActivities = await _myTranscripts.GetTranscriptActivitiesAsync("emulator", thisConversationId, pagedActivities?.ContinuationToken, yesterday);
        foreach (var item in pagedActivities.Items)
        {
            // View as message and find value for key "text" :
            var thisMessage = item.AsMessageActivity();
            var userInput = thisMessage.Text;
         }
    } while (pagedActivities.ContinuationToken != null);
}
```

### <a name="remove-stored-transcripts-from-azure-blob-transcript-storage"></a>Supprimer les transcriptions stockées du stockage de transcription d’objets blob Azure
Une fois que vous avez fini d’utiliser des données d’entrée utilisateur pour tester ou déboguer votre bot, les transcriptions stockées peuvent être supprimées par programme de votre magasin de transcription d’objets blob Azure. L’extrait de code suivant supprime toutes les transcriptions stockées de votre magasin de transcriptions de bot.


```csharp
for (int i = 0; i < numTranscripts; i++)
{
   // Remove all stored transcripts except the last one found.
   if (i > 0)
   {
       string thisConversationId = storedTranscripts[i];    
       await _transcriptStore.DeleteTranscriptAsync("emulator", thisConversationId);
    }
}
```


Le lien suivant fournit plus d’informations sur [le stockage de transcriptions d’objets blob Azure](https://docs.microsoft.com/en-us/dotnet/api/microsoft.bot.builder.azure.azureblobtranscriptstore)  

## <a name="next-steps"></a>Étapes suivantes
Maintenant que vous savez comment lire et écrire directement dans le stockage, voyons comment effectuer ces opérations à l’aide du gestionnaire d’état.

> [!div class="nextstepaction"]
> [Enregistrer l’état à l’aide des propriétés de la conversation et de l’utilisateur](bot-builder-howto-v4-state.md)

