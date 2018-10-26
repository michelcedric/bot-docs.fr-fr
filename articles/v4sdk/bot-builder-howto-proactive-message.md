---
title: Comment utiliser une messagerie proactive | Microsoft Docs
description: Comprendre comment envoyer des messages proactifs avec votre bot.
keywords: message proactif
author: jonathanfingold
ms.author: jonathanfingold
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 09/27/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 032d7027db3ce83c54bbacf913c2021a22c3f356
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2018
ms.locfileid: "49997586"
---
# <a name="how-to-use-proactive-messaging"></a>Comment utiliser une messagerie proactive

[!INCLUDE [pre-release-label](~/includes/pre-release-label.md)]

En règle générale, chaque message envoyé à l’utilisateur par un robot se rapporte directement à la saisie précédente de l’utilisateur.
Dans certains cas, il se peut qu’un bot ait besoin d’envoyer à l’utilisateur un message qui n’est pas directement associé au sujet de conversation actuel ou au dernier message envoyé par l’utilisateur. Il s’agit de _messages proactifs_.

## <a name="uses"></a>Utilisations

Les messages proactifs peuvent être utiles dans différents cas de figure. Lorsqu’un bot définit une minuterie ou un rappel, il peut avoir besoin d’avertir l’utilisateur lorsque le moment correspondant arrive. Ou encore, lorsqu’un bot reçoit une notification émanant d’un système externe, il peut avoir besoin de communiquer immédiatement cette information à l’utilisateur. Par exemple, si l’utilisateur a précédemment demandé au bot de surveiller le prix d’un produit, le bot peut alerter l’utilisateur dès que le prix du produit diminue de 20 %. En outre, si un bot a besoin de temps pour compiler une réponse à la question de l’utilisateur, il peut informer l’utilisateur de ce délai et autoriser la conversation à se poursuivre dans l’intervalle. Puis, dès que le bot a terminé de compiler la réponse à la question, il partage cette information avec l’utilisateur.

Lorsque vous implémentez des messages proactifs dans votre bot :

- N’envoyez pas plusieurs messages proactifs dans un court laps de temps. Certains canaux appliquent des restrictions concernant la fréquence à laquelle un bot peut envoyer des messages à l’utilisateur, et désactivent le bot si ce dernier enfreint ces restrictions.
- N’envoyez pas des messages proactifs aux utilisateurs qui ont n’a pas préalablement interagi avec le bot ou sollicité un contact avec le bot par un autre moyen, tel qu’un message électronique ou un SMS.

Un **message proactif ad hoc** constitue le type de message proactif le plus simple.
Le bot injecte simplement le message dans la conversation chaque fois qu’il est déclenché, que l’utilisateur soit ou non déjà engagé dans une autre sujet de conversation avec le bot, et ne tentera pas de modifier la conversation d’une quelconque manière.

Pour gérer plus facilement les notifications, pensez à d’autres méthodes pour intégrer la notification dans le flux de messages. Par exemple, vous pouvez définir un indicateur dans l’état de la conversation ou ajouter la notification à une file d’attente.

## <a name="prerequisites"></a>Prérequis

Pour envoyer un message proactif, votre bot doit avoir un ID d’application et un mot de passe valide. Toutefois, pour les tests en local dans l’émulateur, vous pouvez utiliser une valeur de réserve comme ID d’application.

Pour obtenir un ID d’application et un mot de passe à utiliser avec votre bot, vous pouvez vous connecter au [portail Microsoft Azure](https://portal.azure.com) et créer une ressource **Bot Channels Registration**. Pour les tests, vous pouvez utiliser cet ID d’application et ce mot de passe en local pour le bot, sans déploiement vers Azure.

> [!TIP]
> Si vous n’avez pas encore d’abonnement, vous pouvez vous inscrire pour un <a href="https://azure.microsoft.com/en-us/free/" target="_blank">compte gratuit</a>.

### <a name="required-libraries"></a>Bibliothèques requises

Si vous partez d’un des modèles BotBuilder, les bibliothèques requises sont installées pour vous. Voici les bibliothèques BotBuilder requises pour les messages proactifs :

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Package NuGet **Microsoft.Bot.Builder.Integration.AspNet.Core**. (Cette installation inclut également celle du package **Microsoft.Bot.Builder**.)

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Package npm **Microsoft.Bot.Builder**.

---

## <a name="notes-on-the-sample-code"></a>Remarques sur l’exemple de code

Le code de cet article est issu des exemples de messages proactifs [[C#](https://aka.ms/proactive-sample-cs) | [JS](https://aka.ms/proactive-sample-js)].

Cet exemple modélise des tâches d’utilisateur dont la durée d’exécution reste indéterminée. Le bot stocke des informations sur la tâche, indique à l’utilisateur qu’il sera prévenu une fois la tâche terminée et laisse la conversation se dérouler. Une fois la tâche terminée, le bot envoie le message de confirmation de façon proactive dans la conversation d’origine.

## <a name="define-job-data-and-state"></a>Définir l’état et des données des travaux

Dans ce scénario, nous suivons des travaux arbitraires qui peuvent être créés par divers utilisateurs dans différentes conversations. Il faudra stocker des informations sur chaque travail, y compris la référence de la conversation et l’identificateur du travail.

- La référence de la conversation servira à envoyer le message proactif à la bonne conversation.
- Nous avons besoin d’un moyen pour identifier les travaux. Dans cet exemple, nous utilisons un horodatage.
- Nous devrons stocker l’état des travaux séparément de l’état de la conversation ou de l’utilisateur.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Nous devons définir des classes pour les données et l’état des travaux. Nous devons également inscrire notre bot et configurer un accesseur de propriété d’état pour le journal des travaux.

### <a name="define-a-class-for-job-data"></a>Définir une classe pour les données des travaux

La classe **JobLog** suit les données des travaux et les indexe par numéro de travail (horodatage). Les données des travaux sont définies en tant que classe interne d’un dictionnaire.

```csharp
/// <summary>Contains a dictionary of job data, indexed by job number.</summary>
/// <remarks>The JobLog class tracks all the outstanding jobs.  Each job is
/// identified by a unique key.</remarks>
public class JobLog : Dictionary<long, JobLog.JobData>
{
    /// <summary>Describes the state of a job.</summary>
    public class JobData
    {
        /// <summary>Gets or sets the time-stamp for the job.</summary>
        /// <value>
        /// The time-stamp for the job when the job needs to fire.
        /// </value>
        public long TimeStamp { get; set; } = 0;

        /// <summary>Gets or sets a value indicating whether indicates whether the job has completed.</summary>
        /// <value>
        /// A value indicating whether indicates whether the job has completed.
        /// </value>
        public bool Completed { get; set; } = false;

        /// <summary>
        /// Gets or sets the conversation reference to which to send status updates.
        /// </summary>
        /// <value>
        /// The conversation reference to which to send status updates.
        /// </value>
        public ConversationReference Conversation { get; set; }
    }
}
```

### <a name="define-a-state-middleware-class"></a>Définir une classe de middleware d’état

La classe **JobState** gère l’état du travail, indépendamment de l’état de la conversation ou de l’utilisateur.

```csharp
using Microsoft.Bot.Builder;

/// <summary>A <see cref="BotState"/> for managing bot state for "bot jobs".</summary>
/// <remarks>Independent from both <see cref="UserState"/> and <see cref="ConversationState"/> because
/// the process of running the jobs and notifying the user interacts with the
/// bot as a distinct user on a separate conversation.</remarks>
public class JobState : BotState
{
    /// <summary>The key used to cache the state information in the turn context.</summary>
    private const string StorageKey = "ProactiveBot.JobState";

    /// <summary>
    /// Initializes a new instance of the <see cref="JobState"/> class.</summary>
    /// <param name="storage">The storage provider to use.</param>
    public JobState(IStorage storage)
        : base(storage, StorageKey)
    {
    }

    /// <summary>Gets the storage key for caching state information.</summary>
    /// <param name="turnContext">A <see cref="ITurnContext"/> containing all the data needed
    /// for processing this conversation turn.</param>
    /// <returns>The storage key.</returns>
    protected override string GetStorageKey(ITurnContext turnContext) => StorageKey;
}
```

### <a name="register-the-bot-and-required-services"></a>Inscrire le bot et les services requis

Le fichier **Startup.cs** inscrit le bot et les services associés.

1. L’ensemble des instructions est étendu pour référencer les espaces de noms suivants :

    ```csharp
    using System;
    using System.Linq;
    using Microsoft.AspNetCore.Builder;
    using Microsoft.AspNetCore.Hosting;
    using Microsoft.Bot.Builder;
    using Microsoft.Bot.Builder.Integration;
    using Microsoft.Bot.Builder.Integration.AspNet.Core;
    using Microsoft.Bot.Configuration;
    using Microsoft.Bot.Connector.Authentication;
    using Microsoft.Extensions.Configuration;
    using Microsoft.Extensions.DependencyInjection;
    using Microsoft.Extensions.Logging;
    using Microsoft.Extensions.Options;
    ```

1. La méthode `ConfigureServices` inscrit le bot, y compris la gestion de l’état et des erreurs. Elle inscrit également le service de point de terminaison du bot et l’accesseur d’état du travail.

    ```csharp
    public void ConfigureServices(IServiceCollection services)
    {
        // The Memory Storage used here is for local bot debugging only. When the bot
        // is restarted, everything stored in memory will be gone.
        IStorage dataStore = new MemoryStorage();

        // ...

        // Create Job State object.
        // The Job State object is where we persist anything at the job-scope.
        // Note: It's independent of any user or conversation.
        var jobState = new JobState(dataStore);

        // Make it available to our bot
        services.AddSingleton(sp => jobState);

        // Register the proactive bot.
        services.AddBot<ProactiveBot>(options =>
        {
            var secretKey = Configuration.GetSection("botFileSecret")?.Value;
            var botFilePath = Configuration.GetSection("botFilePath")?.Value;

            // Loads .bot configuration file and adds a singleton that your Bot can access through dependency injection.
            var botConfig = BotConfiguration.Load(botFilePath ?? @".\BotConfiguration.bot", secretKey);
            services.AddSingleton(sp => botConfig ?? throw new InvalidOperationException($"The .bot config file could not be loaded. ({botConfig})"));

            // Retrieve current endpoint.
            var environment = _isProduction ? "production" : "development";
            var service = botConfig.Services.Where(s => s.Type == "endpoint" && s.Name == environment).FirstOrDefault();
            if (!(service is EndpointService endpointService))
            {
                throw new InvalidOperationException($"The .bot file does not contain an endpoint with name '{environment}'.");
            }

            options.CredentialProvider = new SimpleCredentialProvider(endpointService.AppId, endpointService.AppPassword);

            // Creates a logger for the application to use.
            ILogger logger = _loggerFactory.CreateLogger<ProactiveBot>();

            // Catches any errors that occur during a conversation turn and logs them.
            options.OnTurnError = async (context, exception) =>
            {
                logger.LogError($"Exception caught : {exception}");
                await context.SendActivityAsync("Sorry, it looks like something went wrong.");
            };

        });

        services.AddSingleton(sp =>
        {
            var config = BotConfiguration.Load(@".\BotConfiguration.bot");
            var endpointService = (EndpointService)config.Services.First(s => s.Type == "endpoint")
                                    ?? throw new InvalidOperationException(".bot file 'endpoint' must be configured prior to running.");

            return endpointService;
        });
    }
    ```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

### <a name="set-up-the-server-code"></a>Configurer le code du serveur

Le fichier **index.js** a les attributs suivants :

- Il inclut les packages et les services requis.
- Il référence la classe de bot et le fichier **.bot**.
- Il crée le serveur HTTP.
- Il crée l’adaptateur de bot et les objets de stockage.
- Il crée le bot et démarre le serveur, en transmettant les activités au bot.

```javascript
// Copyright (c) Microsoft Corporation. All rights reserved.
// Licensed under the MIT License.

const restify = require('restify');
const path = require('path');

// Import required bot services. See https://aka.ms/bot-services to learn more about the different part of a bot.
const { BotFrameworkAdapter, BotState, MemoryStorage } = require('botbuilder');
const { BotConfiguration } = require('botframework-config');

const { ProactiveBot } = require('./bot');

// Read botFilePath and botFileSecret from .env file.
// Note: Ensure you have a .env file and include botFilePath and botFileSecret.
const ENV_FILE = path.join(__dirname, '.env');
require('dotenv').config({ path: ENV_FILE });

// Create HTTP server.
let server = restify.createServer();
server.listen(process.env.port || process.env.PORT || 3978, function() {
    console.log(`\n${ server.name } listening to ${ server.url }.`);
    console.log(`\nGet Bot Framework Emulator: https://aka.ms/botframework-emulator.`);
    console.log(`\nTo talk to your bot, open proactive-messages.bot file in the Emulator.`);
});

// .bot file path
const BOT_FILE = path.join(__dirname, (process.env.botFilePath || ''));

// Read the bot's configuration from a .bot file identified by BOT_FILE.
// This includes information about the bot's endpoints and configuration.
let botConfig;
try {
    botConfig = BotConfiguration.loadSync(BOT_FILE, process.env.botFileSecret);
} catch (err) {
    console.error(`\nError reading bot file. Please ensure you have valid botFilePath and botFileSecret set for your environment.`);
    console.error(`\n - The botFileSecret is available under appsettings for your Azure Bot Service bot.`);
    console.error(`\n - If you are running this bot locally, consider adding a .env file with botFilePath and botFileSecret.\n\n`);
    process.exit();
}

const DEV_ENVIRONMENT = 'development';

// Define the name of the bot, as specified in .bot file.
// See https://aka.ms/about-bot-file to learn more about .bot files.
const BOT_CONFIGURATION = (process.env.NODE_ENV || DEV_ENVIRONMENT);

// Load the configuration profile specific to this bot identity.
const endpointConfig = botConfig.findServiceByNameOrId(BOT_CONFIGURATION);

// Create the adapter. See https://aka.ms/about-bot-adapter to learn more about using information from
// the .bot file when configuring your adapter.
const adapter = new BotFrameworkAdapter({
    appId: endpointConfig.appId || process.env.MicrosoftAppId,
    appPassword: endpointConfig.appPassword || process.env.MicrosoftAppPassword
});

// Define the state store for your bot. See https://aka.ms/about-bot-state to learn more about using MemoryStorage.
// A bot requires a state storage system to persist the dialog and user state between messages.
const memoryStorage = new MemoryStorage();

// Create state manager with in-memory storage provider.
const botState = new BotState(memoryStorage, () => 'proactiveBot.botState');

// Create the main dialog, which serves as the bot's main handler.
const bot = new ProactiveBot(botState, adapter);

// Listen for incoming requests.
server.post('/api/messages', (req, res) => {
    adapter.processActivity(req, res, async (turnContext) => {
        // Route the message to the bot's main handler.
        await bot.onTurn(turnContext);
    });
});

// Catch-all for errors.
adapter.onTurnError = async (context, error) => {
    // This check writes out errors to console log .vs. app insights.
    console.error(`\n [onTurnError]: ${ error }`);
    // Send a message to the user
    context.sendActivity(`Oops. Something went wrong!`);
};
```

---

<!--TODO: (Post-Ignite) -- link to a second topic on how to write a job completion DirectLine client that will generate appropriate job completed event activities.-->

## <a name="define-the-bot"></a>Définit le bot

L’utilisateur peut demander au bot de créer et d’exécuter un travail à sa place. Un service de travail distinct peut avertir le bot une fois le travail terminé.

Le bot est conçu pour les fonctions suivantes :

- Créer un travail en réponse à un message `run` ou `run job` de l’utilisateur
- Afficher tous les travaux inscrits en réponse à un message `show` ou `show jobs` de l’utilisateur
- Finaliser un travail en réponse à un événement de _travail terminé_ identifiant un travail comme tel
- Simuler un événement de travail terminé en réponse à un message `done <jobIdentifier>`
- Envoyer un message proactif à l’utilisateur, via la conversation d’origine, lorsque le travail est terminé.

Nous ne montrons pas comment implémenter un système capable d’envoyer des activités d’événement à notre bot.
<!--TODO: DirectLine--Add back in once the DirectLine topic is added back to the TOC.
See [how to create a Direct Line bot and client](bot-builder-howto-direct-line.md) for information on how to do so.
-->

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Le bot a diversaspects :

- code d’initialisation
- gestionnaire de tour
- méthodes de création et de finalisation de travaux

### <a name="declare-the-class"></a>Déclarer la classe

```csharp
using System;
using System.Linq;
using System.Threading;
using System.Threading.Tasks;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Configuration;
using Microsoft.Bot.Schema;

namespace Microsoft.BotBuilderSamples
{
    /// <summary>
    /// For each interaction from the user, an instance of this class is called.
    /// This is a Transient lifetime service.  Transient lifetime services are created
    /// each time they're requested. For each Activity received, a new instance of this
    /// class is created. Objects that are expensive to construct, or have a lifetime
    /// beyond the single Turn, should be carefully managed.
    /// </summary>
    public class ProactiveBot : IBot
    {
        /// <summary>The name of events that signal that a job has completed.</summary>
        public const string JobCompleteEventName = "jobComplete";

        public const string WelcomeText = "Type 'run' or 'run job' to start a new job.\r\n" +
                                          "Type 'show' or 'show jobs' to display the job log.\r\n" +
                                          "Type 'done <jobNumber>' to complete a job.";
    }
}
```

### <a name="add-initialization-code"></a>Ajouter le code d’initialisation

```csharp
private readonly JobState _jobState;
private readonly IStatePropertyAccessor<JobLog> _jobLogPropertyAccessor;

public ProactiveBot(JobState jobState, EndpointService endpointService)
{
    _jobState = jobState ?? throw new ArgumentNullException(nameof(jobState));
    _jobLogPropertyAccessor = _jobState.CreateProperty<JobLog>(nameof(JobLog));

    // Validate AppId.
    // Note: For local testing, .bot AppId is empty for the Bot Framework Emulator.
    AppId = string.IsNullOrWhiteSpace(endpointService.AppId) ? "1" : endpointService.AppId;
}

private string AppId { get; }
```

### <a name="add-a-turn-handler"></a>Ajouter un gestionnaire de tour

Tous les bots doivent implémenter un gestionnaire de tour. L’adaptateur transfère les activités vers cette méthode.

```csharp
public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
{
    // See https://aka.ms/about-bot-activity-message to learn more about the message and other activity types.
    if (turnContext.Activity.Type != ActivityTypes.Message)
    {
        // Handle non-message activities.
        await OnSystemActivityAsync(turnContext);
    }
    else
    {
        // Get the job log.
        // The job log is a dictionary of all outstanding jobs in the system.
        JobLog jobLog = await _jobLogPropertyAccessor.GetAsync(turnContext, () => new JobLog());

        // Get the user's text input for the message.
        var text = turnContext.Activity.Text.Trim().ToLowerInvariant();
        switch (text)
        {
            case "run":
            case "run job":

                // Start a virtual job for the user.
                JobLog.JobData job = CreateJob(turnContext, jobLog);

                // Set the new property
                await _jobLogPropertyAccessor.SetAsync(turnContext, jobLog);

                // Now save it into the JobState
                await _jobState.SaveChangesAsync(turnContext);

                await turnContext.SendActivityAsync(
                    $"We're starting job {job.TimeStamp} for you. We'll notify you when it's complete.");

                break;

            case "show":
            case "show jobs":

                // Display information for all jobs in the log.
                if (jobLog.Count > 0)
                {
                    await turnContext.SendActivityAsync(
                        "| Job number &nbsp; | Conversation ID &nbsp; | Completed |<br>" +
                        "| :--- | :---: | :---: |<br>" +
                        string.Join("<br>", jobLog.Values.Select(j =>
                            $"| {j.TimeStamp} &nbsp; | {j.Conversation.Conversation.Id.Split('|')[0]} &nbsp; | {j.Completed} |")));
                }
                else
                {
                    await turnContext.SendActivityAsync("The job log is empty.");
                }

                break;

            default:
                // Check whether this is simulating a job completed event.
                string[] parts = text?.Split(' ', StringSplitOptions.RemoveEmptyEntries);
                if (parts != null && parts.Length == 2
                    && parts[0].Equals("done", StringComparison.InvariantCultureIgnoreCase)
                    && long.TryParse(parts[1], out long jobNumber))
                {
                    if (!jobLog.TryGetValue(jobNumber, out JobLog.JobData jobInfo))
                    {
                        await turnContext.SendActivityAsync($"The log does not contain a job {jobInfo.TimeStamp}.");
                    }
                    else if (jobInfo.Completed)
                    {
                        await turnContext.SendActivityAsync($"Job {jobInfo.TimeStamp} is already complete.");
                    }
                    else
                    {
                        await turnContext.SendActivityAsync($"Completing job {jobInfo.TimeStamp}.");

                        // Send the proactive message.
                        await CompleteJobAsync(turnContext.Adapter, AppId, jobInfo);
                    }
                }

                break;
        }

        if (!turnContext.Responded)
        {
            await turnContext.SendActivityAsync(WelcomeText);
        }
    }
}

private static async Task SendWelcomeMessageAsync(ITurnContext turnContext)
{
    foreach (var member in turnContext.Activity.MembersAdded)
    {
        if (member.Id != turnContext.Activity.Recipient.Id)
        {
            await turnContext.SendActivityAsync($"Welcome to SuggestedActionsBot {member.Name}.\r\n{WelcomeText}");
        }
    }
}

// Handles non-message activities.
private async Task OnSystemActivityAsync(ITurnContext turnContext)
{
    // On a job completed event, mark the job as complete and notify the user.
    if (turnContext.Activity.Type is ActivityTypes.Event)
    {
        var jobLog = await _jobLogPropertyAccessor.GetAsync(turnContext, () => new JobLog());
        var activity = turnContext.Activity.AsEventActivity();
        if (activity.Name == JobCompleteEventName
            && activity.Value is long timestamp
            && jobLog.ContainsKey(timestamp)
            && !jobLog[timestamp].Completed)
        {
            await CompleteJobAsync(turnContext.Adapter, AppId, jobLog[timestamp]);
        }
    }
    else if (turnContext.Activity.Type is ActivityTypes.ConversationUpdate)
    {
        if (turnContext.Activity.MembersAdded.Any())
        {
            await SendWelcomeMessageAsync(turnContext);
        }
    }
}
```

### <a name="add-job-creation-and-completion-methods"></a>Ajouter des méthodes de création et de finalisation de travaux

Pour démarrer un travail, le bot crée le travail, puis enregistre des informations à son sujet et sur la conversation en cours, dans le journal des travaux. Lorsque le bot reçoit un événement de travail terminé dans une conversation, il valide l’ID de travail avant d’appeler le code qui permet de finaliser le travail.

Le code de finalisation de travail obtient l’état de départ dans le journal des travaux, puis marque le travail comme terminé et envoie un message proactif à l’aide de la méthode de _conversation continue_ de l’adaptateur.

- L’appel de la conversation continue invite le canal à commencer un tour indépendant de l’utilisateur.
- L’adaptateur exécute le callback associé à la place du comportement normal du bot dans le gestionnaire de tour. Ce tour a son propre contexte, qui permet de récupérer les informations d’état et d’envoyer un message proactif à l’utilisateur.

```csharp
// Creates and "starts" a new job.
private JobLog.JobData CreateJob(ITurnContext turnContext, JobLog jobLog)
{
    JobLog.JobData jobInfo = new JobLog.JobData
    {
        TimeStamp = DateTime.Now.ToBinary(),
        Conversation = turnContext.Activity.GetConversationReference(),
    };

    jobLog[jobInfo.TimeStamp] = jobInfo;

    return jobInfo;
}

// Sends a proactive message to the user.
private async Task CompleteJobAsync(
    BotAdapter adapter,
    string botId,
    JobLog.JobData jobInfo,
    CancellationToken cancellationToken = default(CancellationToken))
{
    await adapter.ContinueConversationAsync(botId, jobInfo.Conversation, CreateCallback(jobInfo), cancellationToken);
}

// Creates the turn logic to use for the proactive message.
private BotCallbackHandler CreateCallback(JobLog.JobData jobInfo)
{
    return async (turnContext, token) =>
    {
        // Get the job log from state, and retrieve the job.
        JobLog jobLog = await _jobLogPropertyAccessor.GetAsync(turnContext, () => new JobLog());

        // Perform bookkeeping.
        jobLog[jobInfo.TimeStamp].Completed = true;

        // Set the new property
        await _jobLogPropertyAccessor.SetAsync(turnContext, jobLog);

        // Now save it into the JobState
        await _jobState.SaveChangesAsync(turnContext);

        // Send the user a proactive confirmation message.
        await turnContext.SendActivityAsync($"Job {jobInfo.TimeStamp} is complete.");
    };
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Le bot est défini dans **bot.js** et possède les caractéristiques suivantes :

- code d’initialisation
- gestionnaire de tour
- méthodes de création et de finalisation de travaux

### <a name="declare-the-class-and-add-initialization-code"></a>Déclarer la classe et ajouter le code d’initialisation

```javascript
const { ActivityTypes, TurnContext } = require('botbuilder');

const JOBS_LIST = 'jobs';

class ProactiveBot {
    constructor(botState, adapter) {
        this.botState = botState;
        this.adapter = adapter;

        this.jobsList = this.botState.createProperty(JOBS_LIST);
    }

    // ...
};

// Helper function to check if object is empty.
function isEmpty(obj) {
    for (var key in obj) {
        if (obj.hasOwnProperty(key)) {
            return false;
        }
    }
    return true;
};

module.exports.ProactiveBot = ProactiveBot;
```

### <a name="the-turn-handler"></a>Gestionnaire de tour

Les méthodes `onTurn` et `showJobs` sont définies dans la classe `ProactiveBot`. `onTurn` gère l’entrée saisie par les utilisateurs. Il reçoit également les activités d’événement d’un éventuel système d’exécution de travaux. `showJobs` met en forme et envoie le journal des travaux.

```javascript
/**
    *
    * @param {TurnContext} turnContext A TurnContext object representing an incoming message to be handled by the bot.
    */
async onTurn(turnContext) {
    // See https://aka.ms/about-bot-activity-message to learn more about the message and other activity types.
    if (turnContext.activity.type === ActivityTypes.Message) {
        const utterance = (turnContext.activity.text || '').trim().toLowerCase();
        var jobIdNumber;

        // If user types in run, create a new job.
        if (utterance === 'run') {
            await this.createJob(turnContext);
        } else if (utterance === 'show') {
            await this.showJobs(turnContext);
        } else {
            const words = utterance.split(' ');

            // If the user types done and a Job Id Number,
            // we check if the second word input is a number.
            if (words[0] === 'done' && !isNaN(parseInt(words[1]))) {
                jobIdNumber = words[1];
                await this.completeJob(turnContext, jobIdNumber);
            } else if (words[0] === 'done' && (words.length < 2 || isNaN(parseInt(words[1])))) {
                await turnContext.sendActivity('Enter the job ID number after "done".');
            }
        }

        if (!turnContext.responded) {
            await turnContext.sendActivity(`Say "run" to start a job, or "done <job>" to complete one.`);
        }
    } else if (turnContext.activity.type === ActivityTypes.Event && turnContext.activity.name === 'jobCompleted') {
        jobIdNumber = turnContext.activity.value;
        if (!isNaN(parseInt(jobIdNumber))) {
            await this.completeJob(turnContext, jobIdNumber);
        }
    }

    await this.botState.saveChanges(turnContext);
}

// Show a list of the pending jobs
async showJobs(turnContext) {
    const jobs = await this.jobsList.get(turnContext, {});
    if (Object.keys(jobs).length) {
        await turnContext.sendActivity(
            '| Job number &nbsp; | Conversation ID &nbsp; | Completed |<br>' +
            '| :--- | :---: | :---: |<br>' +
            Object.keys(jobs).map((key) => {
                return `${ key } &nbsp; | ${ jobs[key].reference.conversation.id.split('|')[0] } &nbsp; | ${ jobs[key].completed }`;
            }).join('<br>'));
    } else {
        await turnContext.sendActivity('The job log is empty.');
    }
}
```

### <a name="logic-to-start-a-job"></a>Logique pour le démarrage d’un travail

La méthode `createJob` est définie dans la classe `ProactiveBot`. Elle crée et consigne les nouveaux travaux de l’utilisateur. En théorie, elle transfère aussi ces informations au système d’exécution des travaux.

```javascript
// Save job ID and conversation reference.
async createJob(turnContext) {
    // Create a unique job ID.
    var date = new Date();
    var jobIdNumber = date.getTime();

    // Get the conversation reference.
    const reference = TurnContext.getConversationReference(turnContext.activity);

    // Get the list of jobs. Default it to {} if it is empty.
    const jobs = await this.jobsList.get(turnContext, {});

    // Try to find previous information about the saved job.
    const jobInfo = jobs[jobIdNumber];

    try {
        if (isEmpty(jobInfo)) {
            // Job object is empty so we have to create it
            await turnContext.sendActivity(`Need to create new job ID: ${ jobIdNumber }`);

            // Update jobInfo with new info
            jobs[jobIdNumber] = { completed: false, reference: reference };

            try {
                // Save to storage
                await this.jobsList.set(turnContext, jobs);
                // Notify the user that the job has been processed
                await turnContext.sendActivity('Successful write to log.');
            } catch (err) {
                await turnContext.sendActivity(`Write failed: ${ err.message }`);
            }
        }
    } catch (err) {
        await turnContext.sendActivity(`Read rejected. ${ err.message }`);
    }
}
```

### <a name="logic-to-complete-a-job"></a>Logique pour terminer un travail

La méthode `completeJob` est définie dans la classe `ProactiveBot`. Elle joue un rôle de registre et envoie le message proactif à l’utilisateur (dans sa conversation d’origine) pour l’informer que le travail est terminé.

```javascript
async completeJob(turnContext, jobIdNumber) {
    // Get the list of jobs from the bot's state property accessor.
    const jobs = await this.jobsList.get(turnContext, {});

    // Find the appropriate job in the list of jobs.
    let jobInfo = jobs[jobIdNumber];

    // If no job was found, notify the user of this error state.
    if (isEmpty(jobInfo)) {
        await turnContext.sendActivity(`Sorry no job with ID ${ jobIdNumber }.`);
    } else {
        // Found a job with the ID passed in.
        const reference = jobInfo.reference;
        const completed = jobInfo.completed;

        // If the job is not yet completed and conversation reference exists,
        // use the adapter to continue the conversation with the job's original creator.
        if (reference && !completed) {
            // Since we are going to proactively send a message to the user who started the job,
            // we need to create the turnContext based on the stored reference value.
            await this.adapter.continueConversation(reference, async (proactiveTurnContext) => {
                // Complete the job.
                jobInfo.completed = true;
                // Save the updated job.
                await this.jobsList.set(turnContext, jobs);
                // Notify the user that the job is complete.
                await proactiveTurnContext.sendActivity(`Your queued job ${ jobIdNumber } just completed.`);
            });

            // Send a message to the person who completed the job.
            await turnContext.sendActivity('Job completed. Notification sent.');
        } else if (completed) { // The job has already been completed.
            await turnContext.sendActivity('This job is already completed, please start a new job.');
        };
    };
};
```

---

## <a name="test-your-bot"></a>Tester votre robot

Générez et exécutez votre bot en local, puis ouvrez deux fenêtres d’émulateur.

1. Notez que l’ID de conversation est différent dans les deux fenêtres.
1. Dans la première fenêtre, saisissez deux fois `run` pour démarrer quelques travaux.
1. Dans la seconde fenêtre, saisissez `show` pour afficher la liste des travaux dans le journal.
1. Dans la seconde fenêtre, saisissez `done <jobNumber>`, où `<jobNumber>` est un numéro de travail figurant dans le journal, sans parenthèses. (Le code du bot est conçu pour interpréter ceci comme un événement jobComplete.)
1. Notez que le bot envoie un message proactif à l’utilisateur dans la première fenêtre.

<!--TODO: Recreate the screen shots once we're happy with both the C# and JS versions of the code.-->

Votre conversation peut ressembler à ceci pour l’utilisateur :

![Session d’émulateur de l’utilisateur](~/v4sdk/media/how-to-proactive/user.png)

Elle ressemble à ceci du point de vue du système de travaux simulé :

![Session d’émulateur du système de travaux](~/v4sdk/media/how-to-proactive/job-system.png)

<!-- Add a next steps section. -->
