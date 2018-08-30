---
title: Créer un middleware avec Cosmos DB | Microsoft Docs
description: Découvrez comment créer un middleware personnalisé se connectant à Cosmos DB.
keywords: middleware, cosmos db, base de données, azure, onTurn
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 03/21/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 3f36b00a91644859445676676374f7d321087800
ms.sourcegitcommit: 2dc75701b169d822c9499e393439161bc87639d2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/24/2018
ms.locfileid: "42906100"
---
# <a name="create-middleware-that-logs-in-cosmos-db"></a>Créer un middleware qui se connecte à Cosmos DB

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Le Kit de développement logiciel (SDK) offre des middleware très utiles. Cependant, dans certaines situations, vous devrez implémenter votre propre middleware pour atteindre l’objectif souhaité.

Dans le cadre de cet exemple, nous allons créer une couche de middleware se connectant à Cosmos DB pour enregistrer tous les messages reçus et les réponses que nous renvoyons. Le code que vous verrez ici est disponible en tant que code source complet dans le cadre de nos [exemples](../dotnet/bot-builder-dotnet-samples.md).

## <a name="set-up"></a>Installation

Nous devons configurer certaines choses avant d’aborder le code.

### <a name="create-your-database"></a>Créer votre base de données

Tout d’abord, nous devons disposer d’un emplacement pour stocker notre base de données.  Pour cela, vous pouvez utiliser un compte Azure. Vous pouvez également recourir à [l’émulateur Cosmos DB](https://docs.microsoft.com/en-us/azure/cosmos-db/local-emulator) à des fins de test. Quelle que soit la méthode choisie, nous avons besoin d’un URI de point de terminaison et d’une clé pour la base de données.

Pour l’exemple et le code fourni ici, nous utilisons l’émulateur. Le point de terminaison et la clé sont ceux fournis pour le compte de test Cosmos DB. Il s’agit de valeurs courantes fournies avec la documentation de l’émulateur, qui ne peuvent pas être utilisées dans un contexte de production.

Si vous souhaitez utiliser une véritable base de données Azure Cosmos DB, suivez l’étape 1 du [didacticiel sur la prise en main d’Azure Cosmos DB](https://docs.microsoft.com/en-us/azure/cosmos-db/sql-api-get-started). Ensuite, votre URI de point de terminaison et votre clé sont disponibles dans l’onglet **Clés** de vos paramètres de base de données. Ces valeurs seront requises pour notre fichier de configuration ci-dessous.

### <a name="build-a-basic-bot"></a>Créer un bot de base

Le reste de cette rubrique porte sur la création d’un bot de base, tel que le bot HelloBot créé dans le cadre de la présentation. Vous pouvez utiliser le [Bot Framework Emulator](https://github.com/Microsoft/BotFramework-Emulator) pour vous connecter à votre bot, communiquer avec votre bot et le tester.

Outre les références requises pour les bots standard utilisant le Bot Framework, vous devrez ajouter des références aux packages NuGet suivants :

* Microsoft.Azure.Documentdb.Core
* System.Configuration.ConfigurationManager

Ces bibliothèques sont utilisées pour la communication avec la base de données et pour permettre l’utilisation d’un fichier de configuration, respectivement.

### <a name="add-configuration-file"></a>Ajouter un fichier de configuration

L’utilisation d’un fichier de configuration est recommandée pour plusieurs raisons. Nous allons donc en utiliser un dans le cas présent. Nos données de configuration sont courtes et simples. Toutefois, vous pouvez ajouter d’autres paramètres de configuration à mesure que bot gagne en complexité.

# <a name="ctabcs"></a>[C#](#tab/cs)
1. Ajoutez un fichier à votre projet nommé `app.config`.
2. Enregistrez les informations suivantes dans ce fichier. Le point de terminaison et la clé sont définis pour l’émulateur local, mais peuvent être mis à jour pour votre instance de Cosmos DB.
    ```
    <?xml version="1.0" encoding="utf-8"?>
    <configuration>
        <appSettings>
            <add key="EmulatorDbUrl" value="https://localhost:8081"/>
            <add key="EmulatorDbKey" value="C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw=="/>
        </appSettings>
    </configuration>
    ```
# <a name="javascripttabjs"></a>[JavaScript](#tab/js)
1. Ajoutez un fichier à votre projet nommé `config.js`.
2. Enregistrez les informations suivantes dans ce fichier. Le point de terminaison et la clé sont définis pour l’émulateur local, mais peuvent être mis à jour pour votre instance de Cosmos DB.
    ```javascript
        var config = {}

        config.EmulatorServiceEndpoint = "https://localhost:8081";
        config.EmulatorAuthKey = "C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWE+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==";
        config.database = {
            "id": "Tasks"
        };
        config.collection = {
            "id": "Items"
        };

        module.exports = config;
    ```

---

Si vous utilisez une base de données Cosmos DB réelle, vous pouvez remplacer les valeurs des deux clés ci-dessus par les valeurs figurant dans vos paramètres Cosmos DB. Vous pouvez également ajouter deux autres clés à votre configuration, comme illustré ci-dessous. Vous pouvez ainsi basculer entre votre base de données Cosmos DB réelle et l’émulateur pour les tests.

# <a name="ctabcs"></a>[C#](#tab/cs)
```
    <add key="ActualDbUrl" value="<your database URI>"/>
    <add key="ActualDbKey" value="<your database key>"/>
```
 Si vous ajoutez deux autres clés et souhaitez utiliser la base de données réelle, veillez à spécifier la clé appropriée dans le constructeur ci-après à la place de `EmulatorDbUrl` et `EmulatorDbKey`.

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)
```
    config.ActualServiceEndpoint = "your database URI;
    config.ActualAuthKey = "your database key";
```

Si vous ajoutez deux autres clés et souhaitez utiliser la base de données réelle, veillez à spécifier la clé appropriée dans le constructeur ci-après à la place de `EmulatorServiceEndpoint` et `EmulatorAuthKey`.

---



## <a name="creating-your-middleware"></a>Créer votre middleware

# <a name="ctabcs"></a>[C#](#tab/cs)
Avant de commencer à écrire la logique réelle du middleware, créez une classe dans votre projet de bot pour le middleware. Une fois que vous disposez de cette classe, modifiez l’espace de noms selon ce que nous allons utiliser pour le reste de notre bot, `Microsoft.Bot.Samples`. Vous voyez ici que nous avons ajouté des références à de nouvelles bibliothèques :

* `Newtonsoft.Json;`
* `Microsoft.Azure.Documents;`
* `Microsoft.Azure.Documents.Client;`
* `Microsoft.Azure.Documents.Linq;`

Les différents `Microsoft.Azure.Documents.*` sont dédiés à différentes parties de la communication avec notre base de données, et `Newtonsoft.Json` nous aide à analyser le code JSON transitant vers et à partir de notre base de données.

Enfin, chaque élément du middleware implémente `IMiddleware`. Nous devons donc définir les descripteurs appropriés pour garantir un fonctionnement correct du middleware.

```cs
using System;
using System.Collections.Generic;
using System.Threading.Tasks;
using System.Configuration;
using Newtonsoft.Json;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Schema;
using Microsoft.Azure.Documents;
using Microsoft.Azure.Documents.Client;
using Microsoft.Azure.Documents.Linq;

namespace Microsoft.Bot.Samples
{
    public class CosmosMiddleware : IMiddleware
    {
        ...
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)
Avant de commencer à écrire la logique réelle du middleware, nous devons créer notre middleware personnalisé qui commence par `onTurn()`. 

```javascript
adapter.use({onTurn: async (context, next) =>{
    // Middleware logic here...
}})
    
```

---

#### <a name="defining-local-variables"></a>Définir des variables locales

# <a name="ctabcs"></a>[C#](#tab/cs)
Nous avons ensuite besoin de variables locales pour la manipulation de notre base de données et d’une classe pour stocker les informations que nous souhaitons enregistrer. Cette classe d’informations, que nous appelons `Log`, définit comment seront nommées les propriétés JSON associées à chaque membre. Nous y reviendrons ultérieurement.

```cs
    private string Endpoint;
    private string Key;
    private static readonly string Database = "BotData";
    private static readonly string Collection = "BotCollection";
    public DocumentClient docClient;

    public class Log
    {
        [JsonProperty("Time")]
        public string Time;

        [JsonProperty("MessageReceived")]
        public string Message;

        [JsonProperty("ReplySent")]
        public string Reply;
    }
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)
Ensuite, nous avons besoin d’une variable locale pour stocker les informations que nous souhaitons enregistrer. Dans le middleware, nous devons accéder à conversationState, dont le fournisseur de stockage est Cosmos DB. La variable `info` sera l’objet d’état que nous utiliserons pour la lecture et l’écriture. 

```javascript
// Add conversation state middleware
const conversationState = new ConversationState(storage);
adapter.use(conversationState);

adapter.use({onTurn: async (context, next) =>{

    const info = conversationState.get(context);
    // More middleware logic 
}})
```

--- 


#### <a name="initialize-database-connection"></a>Initialiser la connexion de base de données

# <a name="ctabcs"></a>[C#](#tab/cs)
Le constructeur de ce middleware gère l’extraction des informations nécessaires à partir du fichier de configuration que nous avons défini plus haut, ainsi que la création du `DocumentClient`, qui nous permet d’accéder à notre base de données en lecture et écriture. Nous nous assurons également que notre base de données et notre collection sont configurées.

L’utilisation correcte d’un nouveau `DocumentClient` implique sa suppression. Notre destructeur se charge de cette opération.

Pour créer notre base de données, nous devons effectuer une tentative de lecture simple sur notre base de données, puis sur la collection incluse dans la base de données. Si nous rencontrons une exception `NotFound`, nous l’interceptons et créons la base de données ou la collection au lieu de l’ajouter en haut de la pile. Dans la plupart des cas, elles sont déjà créées. Dans le cadre de cet exemple, cela nous permet toutefois de ne pas nous soucier de la création de cette base de données avant la première exécution. À des fins de simplicité, la base de données et la collection sont divisées en deux fonctions dans l’exemple de code. Cependant, elles ont été combinées dans l’extrait de code ci-dessous.

Pour plus d’informations sur CosmosDB, consultez le [site web](https://docs.microsoft.com/en-us/azure/cosmos-db/) correspondant. Dans le reste de cette rubrique, nous allons mettre de côté les détails sur Cosmos DB et nous concentrer sur ce dont votre bot à besoin pour l’utiliser.

Les définitions de `Key`, `Endpoint` et `docClient` étaient incluses dans un extrait de code précédent et ont été copiées ici uniquement à des fins de clarté.

```cs
    private string Endpoint;
    private string Key;
    // ...
    public DocumentClient docClient;
    // ...

    public CosmosMiddleware()
    {
        Endpoint = ConfigurationManager.AppSettings["EmulatorDbUrl"];
        Key = ConfigurationManager.AppSettings["EmulatorDbKey"];
        docClient = new DocumentClient(new Uri(Endpoint), Key);
        CreateDatabaseAndCollection().ConfigureAwait(false);
    }

    ~CosmosMiddleware()
    {
        docClient.Dispose();
    }

    private async Task CreateDatabaseAndCollection()
    {
        try
        {
            await docClient.ReadDatabaseAsync(UriFactory.CreateDatabaseUri(Database));
        }
        catch (DocumentClientException e)
        {
            if (e.StatusCode == System.Net.HttpStatusCode.NotFound)
            {
                await docClient.CreateDatabaseAsync(new Database { Id = Database });
            }
            else
            {
                throw;
            }
        }

        try
        {
            await docClient.ReadDocumentCollectionAsync(UriFactory.CreateDocumentCollectionUri
                (Database, Collection));
        }
        catch (DocumentClientException e)
        {
            if (e.StatusCode == System.Net.HttpStatusCode.NotFound)
            {
                await docClient.CreateDocumentCollectionAsync(
                    UriFactory.CreateDatabaseUri(Database),
                    new DocumentCollection { Id = Collection });
            }
            else
            {
                throw;
            }
        }
    }
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)
Pour créer notre base de données, nous devons effectuer une tentative de lecture simple sur notre base de données, puis sur la collection incluse dans la base de données. Si nous rencontrons `undefined`, nous l’interceptons et créons un objet appelé `log` qui sera défini en tant que tableau vide. Dans la plupart des cas, `log` est déjà créé. Dans le cadre de cet exemple, cela nous permet toutefois de ne pas nous soucier de la création de cette base de données avant la première exécution. Dans l’exemple de code, nous vérifions si `info.log` est indéfini. Si c’est le cas, définissez-le en tant que tableau vide.

```javascript
adapter.use({onTurn: async (context, next) =>{

    const info = conversationState.get(context);

    if(info.log == undefined){
        info.log = [];
    }

    // More bot logic below
}})
```
---

#### <a name="read-from-database-logic"></a>Lire à partir de la logique de base de données

La dernière fonction d’assistance lit à partir de notre base de données et retourne le nombre d’enregistrements spécifié le plus récent. Notez qu’il existe de meilleures façons de récupérer les données d’une base de données que celle que nous utilisons ici, notamment si votre banque de données est beaucoup plus importante.

# <a name="ctabcs"></a>[C#](#tab/cs)
```cs
    public async Task<string> ReadFromDatabase(int numberOfRecords)
    {
        var documents = docClient.CreateDocumentQuery<Log>(
                UriFactory.CreateDocumentCollectionUri(Database, Collection))
                .AsDocumentQuery();
        List<Log> messages = new List<Log>();
        while (documents.HasMoreResults)
        {
            messages.AddRange(await documents.ExecuteNextAsync<Log>());
        }

        // Create a sublist of messages containing the number of requested records.
        List<Log> messageSublist = messages.GetRange(messages.Count - numberOfRecords, numberOfRecords);

        string history = "";

        // Send the last 3 messages.
        foreach (Log logEntry in messageSublist)
        {
            history += ("Message was: " + logEntry.Message + " Reply was: " + logEntry.Reply + "  ");
        }

        return history;
    }
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

Le code ci-dessous est toujours inclus dans la fonction `onTurn()` et sera appelé si l’utilisateur entre la chaîne « history » (historique).

```javascript
adapter.use({onTurn: async (context, next) =>{

    const utterance = (context.activity.text || '').trim().toLowerCase();
    const isMessage = context.activity.type === 'message';
    const info = conversationState.get(context);

    if(info.log == undefined){
        info.log = [];
    }

    if(isMessage && utterance == 'history'){
        // Loop through the info.log array
        var logHistory = "";
        for(var i = 0; i < info.log.length; i++){
            logHistory += info.log[i] + " ";
        }
        await context.sendActivity(logHistory);
        // Short circuit the middleware
        return
    }
    //...
}})

```

---

#### <a name="define-onturn"></a>Définir OnTurn()

La méthode de middleware standard `OnTurn()` gère ensuite le reste du travail. Nous souhaitons effectuer l’enregistrement uniquement lorsque l’activité en cours est un message. Nous vérifions ceci avant et après l’appel de `next()`.

Nous vérifions d’abord s’il s’agit du message relatif au cas spécial (l’utilisateur demande l’historique le plus récent). Si c’est le cas, nous effectuons un appel de lecture à partir de notre base de données pour obtenir les enregistrements les plus récents et nous envoyons cela dans la conversation. Dans la mesure où ce processus gère totalement l’activité en cours, nous court-circuitons le pipeline au lieu de transmettre l’exécution.

Pour obtenir la réponse envoyée pas le bot, nous créons un descripteur chaque fois que nous recevons un message pour récupérer ces réponses, qui peuvent être vues dans la fonction lambda que nous fournissons à `OnSendActivity()`. Il génère une chaîne pour collecter tous les messages envoyés par le biais de `SendActivity()` pour cet objet de contexte.

Lorsque l’exécution revient au pipeline à partir de `next()`, nous assemblons nos données de journal et les écrivons dans la base de données. 

Si vous examinez l’explorateur de données de Cosmos DB après l’envoi de plusieurs messages à ce bot, vous devez voir vos données dans des enregistrements individuels. Lorsque vous examinez l’un de ces enregistrements, vous devez voir que les trois premiers éléments sont les trois valeurs de nos données de journal, nommées comme la chaîne que nous avons spécifiée pour leurs propriétés `JsonProperty` respectives.

# <a name="ctabcs"></a>[C#](#tab/cs)
```cs
    public async Task OnTurn
        (ITurnContext context, MiddlewareSet.NextDelegate next)
    {
        string botReply = "";

        if (context.Activity.Type == ActivityTypes.Message)
        {
            if (context.Activity.Text == "history")
            {
                // Read last 3 responses from the database, and short circuit future execution.
                await context.SendActivity(await ReadFromDatabase(3));
                return;
            }

            // Create a send activity handler to grab all response activities 
            // from the activity list.
            context.OnSendActivity(async (activityContext, activityList, activityNext) =>
            {
                foreach (Activity activity in activityList)
                {
                    botReply += (activity.Text + " ");
                }
                await activityNext();
            });
        }

        // Pass execution on to the next layer in the pipeline.
        await next();

        // Save logs for each conversational exchange only.
        if (context.Activity.Type == ActivityTypes.Message)
        {
            // Build a log object to write to the database.
            var logData = new Log
            {
                Time = DateTime.Now.ToString(),
                Message = context.Activity.Text,
                Reply = botReply
            };

            // Write our log to the database.
            try
            {
                var document = await docClient.CreateDocumentAsync(UriFactory.
                    CreateDocumentCollectionUri(Database, Collection), logData);
            }
            catch (Exception ex)
            {
                // More logic for what to do on a failed write can be added here
                throw ex;
            }
        }
    }
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

Code s’appuyant sur notre fonction `onTurn` décrite plus haut.

```javascript
adapter.use({onTurn: async (context, next) =>{

    const utterance = (context.activity.text || '').trim().toLowerCase();
    const isMessage = context.activity.type === 'message';
    const info = conversationState.get(context);

    if(info.log == undefined){
        info.log = [];
    }

    if(isMessage && utterance == 'history'){
        // Loop through info.log 
        var logHistory = "";
        for(var i = 0; i < info.log.length; i++){
            logHistory += info.log[i] + " ";
        }
        await context.sendActivity(logHistory);
        // Short circuit the middleware
        return
    } else if(isMessage){
        // Store the users response
        info.log.push(`Message was: ${context.activity.text}`); 

        await context.onSendActivities(async (handlerContext, activities, handlerNext) => 
        {
            // Store the bot's reply
            info.log.push(`Reply was: ${activities[0].text}`);
            
            await handlerNext(); 
        });
        await next();
    }
}})

```

---

## <a name="sample-output"></a>Exemple de sortie

Dans l’exemple complet, le bot utilise la bibliothèque d’invites pour demander le nom de l’utilisateur et le numéro favori. Pour plus d’informations sur la bibliothèque d’invites, reportez-vous à la rubrique [prompting users](~/v4sdk/bot-builder-prompts.md) (invites à l’attention des utilisateurs).

Voici un exemple de conversation dans lequel notre exemple de bot exécute le code fourni plus haut.

![Middleware Cosmos : exemple de conversation](./media/cosmos-middleware-conversation.png)

## <a name="additional-resources"></a>Ressources supplémentaires

* [Cosmos DB](https://docs.microsoft.com/en-us/azure/cosmos-db/)
* [Émulateur Cosmos DB](https://docs.microsoft.com/en-us/azure/cosmos-db/local-emulator)

