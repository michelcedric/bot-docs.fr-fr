---
title: Anatomie d’un bot | Microsoft Docs
description: Décomposition des différentes parties d’un bot et présentation de leur fonctionnement
keywords: ''
author: ivorb
ms.author: ivorb
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 06/25/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 238be22eff746edff30a5446d20991dfaedc945a
ms.sourcegitcommit: 44f100a588ffda19c275b118f4f97029f12d1449
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/26/2018
ms.locfileid: "42928287"
---
# <a name="anatomy-of-a-bot"></a>Anatomie d’un bot

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Au [sens premier du terme](bot-builder-basics.md), un bot est une application conversationnelle permettant aux utilisateurs de communiquer à l’aide de messages. Il respecte la structure de base d’une application web dans son langage respectif et s’appuie sur cette dernière avec le Kit de développement logiciel (SDK) Bot Framework.

Un bot comporte différents composants vous permettant [d’envoyer des messages](./bot-builder-howto-send-messages.md) et d’effectuer le suivi de [l’état](./bot-builder-storage-concept.md). Vous pouvez communiquer avec votre bot par le biais de [l’émulateur](../bot-service-debug-emulator.md), de [WebChat](../bot-service-manage-test-webchat.md) ou, en production, par l’intermédiaire de l’un des [canaux](bot-concepts.md). 

Dans cet article, nous allons examiner un bot Echo de base, étape par étape, et nous allons étudier chacun des composants qui contribuent à son fonctionnement.

## <a name="files-created-for-echo-bot"></a>Fichiers créés pour le bot Echo

Chaque langage de programmation crée des fichiers distincts pour le bot Echo, nommé **BasicEcho** dans cet exemple ; ces fichiers se répartissent en trois sections : la section système, les éléments d’assistance et le cœur du bot. La table ci-après répertorie les principaux fichiers pour C# et pour JavaScript.

| C# | JavaScript |
| --- | --- |
| `Properties > launchSettings.json` <br> `wwwroot > default.htm` <br> `appsettings.json` <br> `BasicEcho.bot` <br> `EchoBot.cs` <br> `EchoState.cs` <br> `Program.cs` <br> `readme.md` <br> `Startup.cs` | `.env` <br> `app.js` <br> `package.json` <br> `README.md` <br> `basicEcho.bot` |

Nous décrivons ci-après quelques fonctionnalités clés de ces fichiers par section. Certains de ces fichiers comportent leur propre ensemble d’instructions Include, qui intègrent des bibliothèques de programmation à la fois propres à un bot et générales. Ces instructions Include vous offrent un ensemble de fonctions qui vous sont nécessaires, ainsi que d’autres dont vous pourrez avoir l’utilité dans votre application. Bien que nous ne présentions pas les détails de ces instructions Include, n’hésitez pas à utiliser Visual Studio pour jeter un coup d’œil aux définitions et rechercher à quels espaces de noms elles appartiennent si ces questions vous intéressent.

## <a name="system-section"></a>Section système

La section système comporte les éléments dont vous avez besoin pour faire fonctionner votre bot comme une application standard. Cette section inclut les fichiers de configuration, l’adaptateur, ainsi que certains fichiers JSON. Ces éléments peuvent (et doivent le plus souvent) être conservés tels quels, à l’exception de certains fichiers de configuration pouvant nécessiter des informations qui vous sont propres.

# <a name="ctabcs"></a>[C#](#tab/cs)

Un bot est un type d’infrastructure d’application [web ASP.NET Core](https://docs.microsoft.com/en-us/aspnet/core/?view=aspnetcore-2.1). Si vous étudiez les [notions de base d’ASP.NET](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/index?view=aspnetcore-2.1&tabs=aspnetcore2x), vous découvrirez qu’il existe un code similaire dans les fichiers tels qu’appsettings.json, Program.cs et Startup.cs décrits ci-après. Ces fichiers sont requis pour toutes les applications web et ne sont pas propres à un bot. Le code de certains de ces fichiers n’est pas copié ici, mais vous le verrez lorsque vous exécuterez le bot.

### <a name="appsettingsjson"></a>appsettings.json

Ce fichier contient simplement les informations de paramètres concernant votre bot, principalement l’ID de l’application et le mot de passe, dans un format JSON de base.  Lorsque vous testez le bot avec [l’émulateur](../bot-service-debug-emulator.md), ces informations ne sont pas nécessaires et peuvent rester vides ; en revanche, elles sont requises en production.

### <a name="programcs"></a>Program.cs

Ce fichier est requis pour le fonctionnement de votre application web ASP.NET et spécifie la classe `Startup` à exécuter.

### <a name="startupcs"></a>Startup.cs

Le fichier **Startup.cs** intègre d’autres sections intéressantes que nous aborderons ultérieurement, mais nous allons ici nous concentrer sur ses sections système.

Le constructeur de `Startup` spécifie les paramètres d’application et les variables d’environnement, et crée la configuration de votre application web.

La méthode `Configure` achève la configuration de votre application en précisant que l’application utilise Bot Framework et quelques autres fichiers. Tous les bots qui utilisent Bot Framework nécessitent cet appel de configuration.

```cs
using System;
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Hosting;
using Microsoft.Bot.Builder.BotFramework;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Builder.Integration.AspNet.Core;
using Microsoft.Bot.Builder.TraceExtensions;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;

namespace BasicEcho
{
    public class Startup
    {
        // This method gets called by the runtime. Use this method to add services to the container.
        // For more information on how to configure your application, visit https://go.microsoft.com/fwlink/?LinkID=398940
        public Startup(IHostingEnvironment env)
        {
            var builder = new ConfigurationBuilder()
                .SetBasePath(env.ContentRootPath)
                .AddJsonFile("appsettings.json", optional: true, reloadOnChange: true)
                .AddJsonFile($"appsettings.{env.EnvironmentName}.json", optional: true)
                .AddEnvironmentVariables();

            Configuration = builder.Build();
        }

        public IConfiguration Configuration { get; }

        // ...
        // Definition of ConfigureServices covered in the next section
        // ...

        // This method gets called by the runtime. Use this method to configure the HTTP request pipeline.
        public void Configure(IApplicationBuilder app, IHostingEnvironment env)
        {
            if (env.IsDevelopment())
            {
                app.UseDeveloperExceptionPage();
            }

            app.UseDefaultFiles()
                .UseStaticFiles()
                .UseBotFramework();
        }
    }
}

```

### <a name="launchsettingsjson-and-readmemd"></a>launchSettings.json et readme.md

**launchSettings.json** contient uniquement certains paramètres de lancement pour les applications web, tandis que **readme.md** est une simple explication d’une seule ligne relative au bot Echo. Vous n’avez pas besoin de connaître le contenu de ces fichiers pour comprendre le fonctionnement de ce bot. 

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

La section système contient principalement les fichiers **package.json**, **.env**, **app.js** et **README.md**. Le code de certains fichiers n’est pas copié ici, mais vous le verrez lorsque vous exécuterez le bot.

### <a name="packagejson"></a>package.json

Le fichier **package.json** spécifie les dépendances et leurs versions associées pour votre bot. Toutes ces informations sont configurées par le modèle et par votre système.

### <a name="env-file"></a>Fichier .env

Le fichier **.env** spécifie les informations de configuration de votre bot, notamment le numéro de port, l’ID de l’application et le mot de passe. Si vous utilisez certaines technologies ou ce bot en production, vous devrez ajouter vos propres clés ou URL à cette configuration. Toutefois, pour les besoins de ce bot Echo, vous n’avez rien à faire à ce stade ; l’ID de l’application et le mot de passe peuvent rester non définis pour l’instant. 

Pour utiliser le fichier de configuration **.env**, le modèle doit inclure un package supplémentaire.  Commencez par obtenir le package `dotenv` à partir de npm :

`npm install dotenv`

Ensuite, ajoutez la ligne ci-après à votre bot avec les autres bibliothèques requises :

```javascript
const dotenv = require('dotenv');
```

### <a name="appjs"></a>app.js

Le début de votre fichier `app.js` spécifie le serveur et l’adaptateur qui permettent à votre bot de communiquer avec l’utilisateur et d’envoyer des réponses. Le serveur écoute le port spécifié à partir de la configuration du fichier **.env** ou revient au port 3978 pour la connexion à votre émulateur. L’adaptateur jouera le rôle de conducteur de votre bot en dirigeant les communications entrantes et sortantes, l’authentification, et ainsi de suite. 

```javascript
// Create server
let server = restify.createServer();
server.listen(process.env.port || process.env.PORT || 3978, function () {
    console.log(`${server.name} listening to ${server.url}`);
});

// Create adapter
const adapter = new BotFrameworkAdapter({ 
    appId: process.env.MICROSOFT_APP_ID, 
    appPassword: process.env.MICROSOFT_APP_PASSWORD 
});
``` 

### <a name="readmemd"></a>README.md

Le fichier **README.md** fournit des informations utiles sur certains aspects de la création d’un bot, tels que la structure de base du bot ou les *dialogues*, mais n’est pas nécessaire à la compréhension du bot.

---


## <a name="helper-items"></a>Éléments d’assistance

Comme dans le cas de n’importe quel programme, la présence de méthodes d’assistance et d’autres définitions peut simplifier notre code. Certaines parties de notre bot qui appartiennent à cette catégorie, comme le fichier `.bot`, sont importantes pour notre bot, mais ne font pas nécessairement partie de la logique fondamentale de celui-ci.

### <a name="bot-file"></a>Fichier .bot

Le fichier `.bot` contient différentes informations, telles que le point de terminaison, l’ID de l’application et le mot de passe, qui permettent aux [canaux](bot-concepts.md) de se connecter à votre bot. Ce fichier est créé à votre intention lorsque vous commencez à créer un bot à partir d’un modèle, mais vous pouvez créer votre propre fichier par le biais de l’émulateur ou d’autres outils.

Le contenu de votre fichier doit correspondre à celui présenté ici, sauf si vous attribuez à votre bot un nom autre que *BasicEcho*. Vous devrez peut-être modifier et mettre à jour ces informations en fonction du mode d’utilisation de votre bot, mais l’exécution de ce bot Echo ne nécessite aucune modification à ce stade.

```json
{
  "name": "BasicEcho",
  "secretKey": "",
  "services": [
    {
      "appId": "",
      "id": "http://localhost:3978/api/messages",
      "type": "endpoint",
      "appPassword": "",
      "endpoint": "http://localhost:3978/api/messages",
      "name": "BasicEcho"
    }
  ]
}
```

# <a name="ctabcs"></a>[C#](#tab/cs)

### <a name="echostatecs"></a>EchoState.cs

Le fichier **EchoState.cs** contient une classe simple que notre bot utilise pour conserver l’état actuel. Il comporte uniquement un élément `int` que nous utilisons pour incrémenter le compteur de tours. Nous étudierons plus en détail l’état à la section suivante, mais pour l’instant, il vous suffit de comprendre que l’élément `EchoState` correspond à notre classe contenant le nombre de tours.

```cs
namespace BasicEcho
{
    /// <summary>
    /// Class for storing conversation state.
    /// </summary>
    public class EchoState
    {
        public int TurnCount { get; set; } = 0;
    }
}
```

### <a name="defaulthtm"></a>default.htm

Le fichier **default.htm** correspond à la page web, écrite en `html`, qui s’affiche lorsque vous exécutez votre bot. Cette page fournit des informations utiles concernant la connexion à votre bot et la manière d’interagir avec ce dernier ; toutefois, son contenu n’a pas d’incidence sur le comportement de votre bot. Vous verrez le code s’afficher lorsque vous exécuterez le bot.

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

### <a name="required-libraries"></a>Bibliothèques requises

Le tout début de votre fichier `app.js` répertorie une série de modules ou de bibliothèques requis. Ces modules vous donnent accès à un ensemble de fonctions que vous souhaiterez peut-être inclure dans votre application. 

```javascript
const { BotFrameworkAdapter, MemoryStorage, ConversationState } = require('botbuilder');
const restify = require('restify');
```

---

## <a name="core-of-the-bot"></a>Cœur du bot

Cette dernière section est celle qui vous intéresse véritablement. Le cœur du bot définit la manière dont ce dernier interagit avec l’utilisateur, incluant l’intergiciel et la logique du bot.

# <a name="ctabcs"></a>[C#](#tab/cs)

### <a name="middleware"></a>Middlewares

Le fichier **Startup.cs** comporte une méthode appelée `ConfigureServices`. Cette méthode configure votre fournisseur d’informations d’identification et ajoute l’intergiciel.

[L’intergiciel](bot-builder-concept-middleware.md) ajouté ici accomplit deux tâches. La première consiste à gérer les exceptions afin que votre bot échoue de manière appropriée en cas de problème. Cette tâche est définie inline sous la forme d’une expression lambda, qui se limite à afficher l’exception sur le terminal et à informer l’utilisateur qu’un problème est survenu.

Le deuxième intergiciel conserve l’état à l’aide de l’élément `MemoryStorage` qui a été défini juste avant lui. Cet état est défini sous la forme `ConversationState`, ce qui signifie simplement que le bot conserve l’état de votre conversation. Il utilise `EchoState`, la classe que nous avons définie avec votre compteur de tours, pour stocker les informations qui vous intéressent.

``` cs
// This method gets called by the runtime. Use this method to add services to the container.
public void ConfigureServices(IServiceCollection services)
{
    services.AddBot<EchoBot>(options =>
    {
        options.CredentialProvider = new ConfigurationCredentialProvider(Configuration);

        // The CatchExceptionMiddleware provides a top-level exception handler for your bot.
        // Any exceptions thrown by other Middleware, or by your OnTurn method, will be 
        // caught here. To facilitate debugging, the exception is sent out, via Trace, 
        // to the emulator. Trace activities are NOT displayed to users, so in addition
        // an "Ooops" message is sent.
        options.Middleware.Add(new CatchExceptionMiddleware<Exception>(async (context, exception) =>
        {
            await context.TraceActivity("EchoBot Exception", exception);
            await context.SendActivity("Sorry, it looks like something went wrong!");
        }));

        // The Memory Storage used here is for local bot debugging only. When the bot
        // is restarted, anything stored in memory will be gone. 
        IStorage dataStore = new MemoryStorage();

        // The File data store, shown here, is suitable for bots that run on 
        // a single machine and need durable state across application restarts.

        // IStorage dataStore = new FileStorage(System.IO.Path.GetTempPath());

        // For production bots use the Azure Table Store, Azure Blob, or 
        // Azure CosmosDB storage provides, as seen below. To include any of 
        // the Azure based storage providers, add the Microsoft.Bot.Builder.Azure 
        // Nuget package to your solution. That package is found at:
        //      https://www.nuget.org/packages/Microsoft.Bot.Builder.Azure/

        // IStorage dataStore = new Microsoft.Bot.Builder.Azure.AzureTableStorage("AzureTablesConnectionString", "TableName");
        // IStorage dataStore = new Microsoft.Bot.Builder.Azure.AzureBlobStorage("AzureBlobConnectionString", "containerName");

        options.Middleware.Add(new ConversationState<EchoState>(dataStore));
    });
}
```

### <a name="bot-logic"></a>Logique du bot

La logique principale du bot est définie dans le gestionnaire `OnTurn` du bot. `OnTurn` reçoit une variable de [contexte](./bot-builder-concept-activity-processing.md#turn-context) en tant qu’argument, qui fournit des informations sur [l’activité](bot-builder-concept-activity-processing.md) entrante, la conversation, etc.

Les activités entrantes peuvent être de [différents types](../bot-service-activities-entities.md#activity-types) ; nous allons donc commencer par vérifier si votre bot a reçu un message. S’il s’agit d’un message, nous créons une variable d’état qui contient les informations de conversation actuelles de votre bot. Ensuite, nous incrémentons l’élément `int` qui effectue le suivi de votre compteur de tours avant de renvoyer le nombre de tours à l’utilisateur, ainsi que le message envoyé par ce dernier.

```cs
using System.Threading.Tasks;
using Microsoft.Bot;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Schema;

namespace BasicEcho
{
    public class EchoBot : IBot
    {
        /// <summary>
        /// Every Conversation turn for our EchoBot will call this method. In here
        /// the bot checks the Activty type to verify it's a message, bumps the 
        /// turn conversation 'Turn' count, and then echoes the users typing
        /// back to them. 
        /// </summary>
        /// <param name="context">Turn scoped context containing all the data needed
        /// for processing this conversation turn. </param>        
        public async Task OnTurn(ITurnContext context)
        {
            // This bot is only handling Messages
            if (context.Activity.Type == ActivityTypes.Message)
            {
                // Get the conversation state from the turn context
                var state = context.GetConversationState<EchoState>();

                // Bump the turn count. 
                state.TurnCount++;

                // Echo back to the user whatever they typed.
                await context.SendActivity($"Turn {state.TurnCount}: You sent '{context.Activity.Text}'");
            }
        }
    }    
}
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

### <a name="middleware"></a>Middlewares

Dans le fichier **app.js**, nous utilisons [l’intergiciel](bot-builder-concept-middleware.md) de ce bot pour conserver l’état, en utilisant l’élément `MemoryStorage` qui a été défini comme l’un de nos modules requis devant l’élément `botbuilder`. Cet état est défini sous la forme `ConversationState`, ce qui signifie simplement que le bot conserve l’état de votre conversation. `ConversationState` stocke en mémoire les informations qui vous intéressent, un simple compteur de tours dans le cas présent.

```javascript
// Add conversation state middleware
const conversationState = new ConversationState(new MemoryStorage());
adapter.use(conversationState);
```

### <a name="bot-logic"></a>Logique du bot

Le troisième paramètre de *processActivity* est un gestionnaire de fonctions qui est appelé pour exécuter la logique du bot une fois que [l’activité](bot-builder-concept-activity-processing.md) reçue a été prétraitée par l’adaptateur et acheminée par le biais d’un intergiciel. La variable de [contexte](./bot-builder-concept-activity-processing.md#turn-context), transmise sous la forme d’un argument au gestionnaire de fonctions, peut être utilisée pour fournir des informations sur l’activité entrante, l’expéditeur et le destinataire, le canal, la conversation, etc.

Nous commençons par vérifier si le bot a reçu un message. Si ce n’est pas le cas, nous renvoyons le type d’activité reçu. Ensuite, nous créons une variable d’état qui contient les informations de conversation de votre bot. Puis nous définissons votre variable de nombre de tours sur 0 si cette valeur n’est pas définie (ce qui est le cas lorsque vous démarrez le bot) ou nous l’incrémentons avec chaque nouveau message. Enfin, nous renvoyons le nombre de tours à l’utilisateur, ainsi que le message envoyé par ce dernier.

```javascript
// Listen for incoming requests 
server.post('/api/messages', (req, res) => {
    // Route received request to adapter for processing
    adapter.processActivity(req, res, (context) => {
        // This bot is only handling Messages
        if (context.activity.type === 'message') {

            // Get the conversation state
            const state = conversationState.get(context);

            // If state.count is undefined set it to 0, otherwise increment it by 1
            const count = state.count === undefined ? state.count = 0 : ++state.count;

            // Echo back to the user whatever they typed.
            return context.sendActivity(`${count}: You said "${context.activity.text}"`);
        } else {
            // Echo back the type of activity the bot detected if not of type message
            return context.sendActivity(`[${context.activity.type} event detected]`);
        }
    });
});
```

---
