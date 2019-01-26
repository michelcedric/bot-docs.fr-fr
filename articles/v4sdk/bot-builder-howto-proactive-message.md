---
title: Obtenir une notification de bots | Microsoft Docs
description: Comprendre comment envoyer des messages de notification
keywords: message proactif, message de notification, notification de bot,
author: jonathanfingold
ms.author: jonathanfingold
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 11/15/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 7a9a2e4f30d1e9b293e51a921afce57d243376d7
ms.sourcegitcommit: c6ce4c42fc56ce1e12b45358d2c747fb77eb74e2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/22/2019
ms.locfileid: "54453963"
---
# <a name="get-notification-from-bots"></a>Recevoir les notifications de bots

[!INCLUDE [pre-release-label](~/includes/pre-release-label.md)]

En règle générale, chaque message envoyé à l’utilisateur par un robot se rapporte directement à la saisie précédente de l’utilisateur.
Dans certains cas, il se peut qu’un bot ait besoin d’envoyer à l’utilisateur un message qui n’est pas directement associé au sujet de conversation actuel ou au dernier message envoyé par l’utilisateur. Il s’agit de _messages proactifs_.

## <a name="proactive-messages"></a>Messages proactifs

Les messages proactifs peuvent être utiles dans différents cas de figure. Lorsqu’un bot définit une minuterie ou un rappel, il peut avoir besoin d’avertir l’utilisateur lorsque le moment correspondant arrive. Ou encore, lorsqu’un bot reçoit une notification émanant d’un système externe, il peut avoir besoin de communiquer immédiatement cette information à l’utilisateur. Par exemple, si l’utilisateur a précédemment demandé au bot de surveiller le prix d’un produit, le bot peut alerter l’utilisateur dès que le prix du produit diminue de 20 %. En outre, si un bot a besoin de temps pour compiler une réponse à la question de l’utilisateur, il peut informer l’utilisateur de ce délai et autoriser la conversation à se poursuivre dans l’intervalle. Puis, dès que le bot a terminé de compiler la réponse à la question, il partage cette information avec l’utilisateur.

Lorsque vous implémentez des messages proactifs dans votre bot :

- N’envoyez pas plusieurs messages proactifs dans un court laps de temps. Certains canaux appliquent des restrictions concernant la fréquence à laquelle un bot peut envoyer des messages à l’utilisateur, et désactivent le bot si ce dernier enfreint ces restrictions.
- N’envoyez pas des messages proactifs aux utilisateurs qui ont n’a pas préalablement interagi avec le bot ou sollicité un contact avec le bot par un autre moyen, tel qu’un message électronique ou un SMS.

Un message proactif ad hoc constitue le type de message proactif le plus simple. Le bot injecte simplement le message dans la conversation chaque fois qu’il est déclenché, que l’utilisateur soit ou non déjà engagé dans une autre sujet de conversation avec le bot, et ne tentera pas de modifier la conversation d’une quelconque manière.

Pour gérer plus facilement les notifications, pensez à d’autres méthodes pour intégrer la notification dans le flux de messages. Par exemple, vous pouvez définir un indicateur dans l’état de la conversation ou ajouter la notification à une file d’attente.

## <a name="prerequisites"></a>Prérequis

- Comprendre les [concepts de base des bots](bot-builder-basics.md) et avoir des notions de la [gestion de l’état](bot-builder-concept-state.md).
- Une copie de l’**exemple de messages proactifs** en [C#](https://aka.ms/proactive-sample-cs) ou [JS](https://aka.ms/proactive-sample-js). Cet exemple est utilisé pour expliquer la messagerie proactive dans cet article. 

## <a name="about-the-sample-code"></a>Au sujet de l’exemple de code

L’exemple de messages proactifs modélise des tâches d’utilisateur dont la durée d’exécution reste indéterminée. Le bot stocke des informations sur la tâche, indique à l’utilisateur qu’il sera prévenu une fois la tâche terminée et laisse la conversation se dérouler. Une fois la tâche terminée, le bot envoie le message de confirmation de façon proactive dans la conversation d’origine.

## <a name="define-job-data-and-state"></a>Définir l’état et des données des travaux

Dans ce scénario, nous suivons des travaux arbitraires qui peuvent être créés par divers utilisateurs dans différentes conversations. Il faudra stocker des informations sur chaque travail, y compris la référence de la conversation et l’identificateur du travail. Ce dont nous avons besoin :

- La référence de la conversation pour pouvoir envoyer le message proactif à la bonne conversation.
- Un moyen d’identifier les travaux. Dans cet exemple, nous utilisons un horodatage.
- Pour stocker l’état du travail séparément de l’état de la conversation ou de l’utilisateur.

Nous étendrons l’_état du bot_ pour définir notre propre objet de gestion de l’état du bot. Le framework du bot utilise la _clé de stockage_ et le contexte du tour pour récupérer et conserver l’état. Pour plus d’informations, consultez [Gestion de l’état](bot-builder-concept-state.md).

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Nous devons définir des classes pour les données et l’état des travaux. Nous devons également inscrire notre bot et configurer un accesseur de propriété d’état pour le journal des travaux.

### <a name="define-a-class-for-job-data"></a>Définir une classe pour les données des travaux

La classe `JobLog` suit les données des travaux et les indexe par numéro de travail (horodatage). La classe `JobLog` suit tous les travaux en attente.  Chaque travail est identifié par une clé unique. `JobData` décrit l’état d’un travail et est défini comme une classe interne d’un dictionnaire.

```csharp
public class JobLog : Dictionary<long, JobLog.JobData>
{
    public class JobData
    {
        // Gets or sets the time-stamp for the job.
        public long TimeStamp { get; set; } = 0;

        // Gets or sets a value indicating whether indicates whether the job has completed.
        public bool Completed { get; set; } = false;

        // Gets or sets the conversation reference to which to send status updates.
        public ConversationReference Conversation { get; set; }
    }
}
```

### <a name="define-a-state-management-class"></a>Définir une classe de gestion de l’état

La classe `JobState` gérera l’état du travail, indépendamment de l’état de la conversation ou de l’utilisateur.

```csharp
using Microsoft.Bot.Builder;

/// A BotState for managing bot state for "bot jobs".
public class JobState : BotState
{
    // The key used to cache the state information in the turn context.
    private const string StorageKey = "ProactiveBot.JobState";

    // Initializes a new instance of the JobState class.
    public JobState(IStorage storage)
        : base(storage, StorageKey)
    {
    }

    // Gets the storage key for caching state information.
    protected override string GetStorageKey(ITurnContext turnContext) => StorageKey;
}
```

### <a name="register-the-bot-and-required-services"></a>Inscrire le bot et les services requis

Le fichier **Startup.cs** inscrit le bot et les services associés.

La méthode `ConfigureServices` inscrit le bot, et le service de point de terminaison y compris la gestion de l’état et des erreurs. Elle enregistre également l’accesseur d’état du travail.

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

    // ...      
    }
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Un bot a besoin d’un système de stockage d’état pour conserver l’état de boîte de dialogue et d’utilisateur entre les messages, qui dans ce cas est défini à l’aide du fournisseur de stockage en mémoire.

```javascript
// index.js

const memoryStorage = new MemoryStorage();
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

// ...
```

---

## <a name="define-the-bot"></a>Définit le bot

L’utilisateur peut demander au bot de créer et d’exécuter un travail à sa place. Un service de travail distinct peut avertir le bot une fois le travail terminé. Le bot est conçu pour les fonctions suivantes :

- Créer un travail en réponse à un message `run` ou `run job` de l’utilisateur
- Afficher tous les travaux inscrits en réponse à un message `show` ou `show jobs` de l’utilisateur
- Finaliser un travail en réponse à un événement de _travail terminé_ identifiant un travail comme tel
- Simuler un événement de travail terminé en réponse à un message `done <jobIdentifier>`
- Envoyer un message proactif à l’utilisateur, via la conversation d’origine, lorsque le travail est terminé.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Le bot a diversaspects :

- code d’initialisation
- gestionnaire de tour
- méthodes de création et de finalisation de travaux

### <a name="declare-the-class"></a>Déclarer la classe

Chaque interaction de l’utilisateur crée une instance de la classe `ProactiveBot`. Le processus de création d’un service chaque fois qu’il est nécessaire est appelé « service de durée de vie temporaire ». Les objets coûteux à construire ou ayant une durée de vie allant au-delà du tour unique, doivent être gérés soigneusement.

```csharp
namespace Microsoft.BotBuilderSamples
{
    public class ProactiveBot : IBot
    {
        // The name of events that signal that a job has completed.
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

    //...
}

```

### <a name="add-a-turn-handler"></a>Ajouter un gestionnaire de tour

L’adaptateur transmet les activités au gestionnaire de tour, qui inspecte le type `Activity`, puis appelle la méthode appropriée. Tous les bots doivent implémenter un gestionnaire de tour.

```csharp
public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
{
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
                // ...
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
```

### <a name="handle-non-message-activities"></a>Gérer les activités hors message

Lors d’un événement de travail terminé, marquez le travail comme terminé et informez l’utilisateur.

```csharp
private async Task OnSystemActivityAsync(ITurnContext turnContext)
{
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

Le code de finalisation de travail obtient l’état de départ dans le journal des travaux, puis marque le travail comme terminé et envoie un message proactif à l’aide de la méthode de `ContinueConversationAsync` de l’adaptateur.

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
```

### <a name="sends-a-proactive-message-to-the-user"></a>Envoie un message proactif à l’utilisateur

```csharp
private async Task CompleteJobAsync(
    BotAdapter adapter,
    string botId,
    JobLog.JobData jobInfo,
    CancellationToken cancellationToken = default(CancellationToken))
{
    await adapter.ContinueConversationAsync(botId, jobInfo.Conversation, CreateCallback(jobInfo), cancellationToken);
}
```

### <a name="creates-the-turn-logic-to-use-for-the-proactive-message"></a>Crée la logique de tour à utiliser pour le message proactif

```csharp
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

Générez et exécutez votre bot en local, puis ouvrez deux fenêtres d’émulateur. Si vous avez besoin d’obtenir des instructions détaillées, consultez le fichier [LISEZ-MOI](https://github.com/Microsoft/BotBuilder-Samples/blob/master/samples/csharp_dotnetcore/16.proactive-messages/README.md).

1. Notez que l’ID de conversation est différent dans les deux fenêtres.
1. Dans la première fenêtre, saisissez deux fois `run` pour démarrer quelques travaux.
1. Dans la seconde fenêtre, saisissez `show` pour afficher la liste des travaux dans le journal.
1. Dans la seconde fenêtre, saisissez `done <jobNumber>`, où `<jobNumber>` est un numéro de travail figurant dans le journal, sans parenthèses. (Le code du bot est conçu pour interpréter ceci comme un événement jobComplete.)
1. Notez que le bot envoie un message proactif à l’utilisateur dans la première fenêtre.

Votre conversation peut ressembler à ceci pour l’utilisateur :

![Session d’émulateur de l’utilisateur](~/v4sdk/media/how-to-proactive/user.png)

Elle ressemble à ceci du point de vue du système de travaux simulé :

![Session d’émulateur du système de travaux](~/v4sdk/media/how-to-proactive/job-system.png)

## <a name="additional-resources"></a>Ressources supplémentaires
Consultez des exemples supplémentaires en C# et en JS sur [GitHub](https://github.com/Microsoft/BotBuilder-Samples/blob/master/readme.md).
