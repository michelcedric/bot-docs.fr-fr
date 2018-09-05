---
title: Envoyer des messages proactifs | Microsoft Docs
description: Comprendre comment envoyer un message de façon proactive avec votre robot.
keywords: message proactif
author: jonathanfingold
ms.author: jonathanfingold
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 05/01/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: c22ce6a35d4d49506360a78a439f15137c429d9d
ms.sourcegitcommit: 2dc75701b169d822c9499e393439161bc87639d2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/24/2018
ms.locfileid: "42905133"
---
# <a name="send-proactive-messages"></a>Envoyer des messages proactifs 

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]


Souvent, les robots envoient des _messages réactifs_ , mais il arrive que nous devions également pouvoir envoyer un [message proactif](bot-builder-proactive-messages.md). 

Un cas courant de messagerie proactive est quand notre robot effectue une tâche qui peut prendre un certain temps. Dans ce cas, vous pouvez stocker les informations sur la tâche, et indiquer à l’utilisateur que le robot reviendra vers lui lorsque la tâche sera terminée, et laissera la conversation se poursuivre. Une fois la tâche terminée, le robot peut reprendre la conversation en envoyant le message de confirmation de manière proactive.

# <a name="ctabcs"></a>[C#](#tab/cs)

## <a name="notes-about-this-sample"></a>À propos de cet exemple

Nous allons modifier l’exemple EchoBot de base.
- Nous utilisons `Microsoft.Samples.Proactive` en tant qu’espace de noms.
- Nous remplaçons le fichier d’état par un fichier `JobData.cs`.
- Nous remplaçons le fichier du robot par un fichier `ProactiveBot.cs`.

> [!NOTE]
> La messagerie proactive requiert actuellement que votre robot ait un ID d’application et un mot de passe valides.


## <a name="define-task-data"></a>Définir des données de tâche

Dans ce scénario, nous suivons des tâches arbitraires qui peuvent être créées par divers utilisateurs dans différentes conversations. Par conséquent, nous utilisons un intergiciel (middleware) général d’état du robot, plutôt qu’un intergiciel (middleware) d’état de l’utilisateur ou de la conversation.

La classe suivante définit la structure de données que nous allons utiliser pour des travaux individuels.


```csharp
using Microsoft.Bot.Schema;
using System.Collections.Generic;

namespace Microsoft.Samples.Proactive
{
    /// <summary>
    /// Class for storing job state. 
    /// </summary>
    public class JobData
    {
        /// <summary>
        /// The name to use to read and write this bot state object to storage.
        /// </summary>
        public readonly static string PropertyName = $"BotState:{typeof(Dictionary<int, JobData>).FullName}";

        public int JobNumber { get; set; } = 0;
        public bool Completed { get; set; } = false;

        /// <summary>
        /// The conversation reference to which to send status updates.
        /// </summary>
        public ConversationReference Conversation { get; set; }
    }
}
```


Nous devons également ajouter l’intergiciel (middleware) d’état à notre code de démarrage.


Dans le fichier `StartUp.cs`, mettez à jour la méthode `ConfigureServices` pour ajouter un dictionnaire de travaux à notre état de robot. Dans le code suivant, il s’agit du dernier appel à `options.Middleware.Add`.
```csharp
// This method gets called by the runtime. Use this method to add services to the container.
public void ConfigureServices(IServiceCollection services)
{
    services.AddBot<ProactiveBot>(options =>
    {
        options.CredentialProvider = new ConfigurationCredentialProvider(Configuration);

        // The CatchExceptionMiddleware provides a top-level exception handler for your bot. 
        // Any exceptions thrown by other Middleware, or by your OnTurn method, will be 
        // caught here. To facillitate debugging, the exception is sent out, via Trace, 
        // to the emulator. Trace activities are NOT displayed to users, so in addition
        // an "Ooops" message is sent. 
        options.Middleware.Add(new CatchExceptionMiddleware<Exception>(async (context, exception) =>
        {
            await context.TraceActivity($"{nameof(ProactiveBot)} Exception", exception);
            await context.SendActivity("Sorry, it looks like something went wrong!");
        }));

        // The Memory Storage used here is for local bot debugging only. When the bot
        // is restarted, anything stored in memory will be gone. 
        IStorage dataStore = new MemoryStorage();

        // Using the base BotState here, since the job log is not necessarily tied to a
        // specific user or conversation.
        options.Middleware.Add(
            new BotState<Dictionary<int, JobData>>(
                dataStore, JobData.PropertyName, (context) => $"jobs/{typeof(Dictionary<int, JobData>)}"));
    });
}
```


## <a name="update-your-bot-to-create-and-run-jobs"></a>Mettre à jour votre robot pour créer et exécuter des travaux

À chaque tour, nous allons laisser un utilisateur créer un travail en tapant `run` ou `run job`.

En réponse, notre robot effectuera les opérations suivantes à chaque tour :
- Créer le travail.
- Enregistrer des informations sur la conversation actuelle afin que nous puissions envoyer le message proactif ultérieurement.
- Informer l’utilisateur que nous allons commencer à exécuter son travail et que nous l’informerons quand celui-ci sera terminé.
- Démarrer le travail asynchrone.
- Laisser le tour s’achever.

Le travail que nous commençons est un simple minuteur de 5 secondes qui se termine par l’envoi du message proactif.
- L’appel de la méthode continue conversation de l’adaptateur crée un tour initié par le robot.
- Ce tour dispose de son propre [contexte de tour](bot-builder-concept-activity-processing.md#turn-context) à partir duquel nous récupérons les informations d’état.
- Nous utilisons ce contexte pour envoyer le message proactif à l’utilisateur.



> [!NOTE]
> La méthode `GetAppId` est un contournement pour activer la messagerie proactive dans le Kit de développement logiciel (SDK) .NET.

```csharp
using Microsoft.Bot;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Connector.Authentication;
using Microsoft.Bot.Schema;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Security.Claims;
using System.Security.Principal;
using System.Threading;
using System.Threading.Tasks;

namespace Microsoft.Samples.Proactive
{
    public class ProactiveBot : IBot
    {
        /// <summary>
        /// Random number generator for job numbers.
        /// </summary>
        private static Random NumberGenerator = new Random();

        /// <summary>
        /// Gets the job log from the bot state.
        /// </summary>
        /// <param name="context">The current turn context.</param>
        /// <returns>The job log.</returns>
        private static Dictionary<int, JobData> GetJobLog(ITurnContext context)
        {
            return context.Services.Get<Dictionary<int, JobData>>(JobData.PropertyName);
        }

        /// <summary>
        /// Workaround to get the bot's app ID.
        /// </summary>
        /// <param name="context">The current turn context.</param>
        /// <returns>The application ID for the bot.</returns>
        private static string GetAppId(ITurnContext context)
        {
            // The BotFrameworkAdapter sets the identity provider on the context object.
            var claimsIdentity = context.Services.Get<IIdentity>("BotIdentity") as ClaimsIdentity;

            // For requests from a channel, the app ID is in the Audience claim of the JWT token.
            // For requests from the emulator, it is in the AppId claim.
            // For unauthenticated requests, we have anonymouse identity provided auth is disabled.
            // For Activities coming from Emulator AppId claim contains the Bot's AAD AppId.
            var botAppIdClaim =
                (claimsIdentity.Claims?.SingleOrDefault(claim => claim.Type == AuthenticationConstants.AudienceClaim)
                ?? claimsIdentity.Claims?.SingleOrDefault(claim => claim.Type == AuthenticationConstants.AppIdClaim));

            return botAppIdClaim?.Value;
        }

        /// <summary>
        /// Every Conversation turn calls this method.
        /// When the user types "run" or "run job", the bot starts a "job".
        /// When the job finishes, the bot proactively notifies the user.
        /// </summary>
        /// <param name="context">The turn context.</param>
        /// <remarks>When our virtual job finishes, it sends a proactive message
        /// to notify the user that the job completed.</remarks>
        public async Task OnTurn(ITurnContext context)
        {
            // This bot is only handling Messages
            if (context.Activity.Type is ActivityTypes.Message)
            {
                var text = context.Activity.AsMessageActivity()?.Text?.Trim().ToLower();
                switch (text)
                {
                    case "run":
                    case "run job":

                        var jobLog = GetJobLog(context);
                        var job = CreateJob(context, jobLog);
                        var appId = GetAppId(context);
                        var conversation = TurnContext.GetConversationReference(context.Activity);

                        await context.SendActivity($"We're starting job {job.JobNumber} for you. We'll notify you when it's complete.");

                        // Since the context is disposed at the end of the turn, extract and send the
                        // information we need to send the proactive message later.
                        var adapter = context.Adapter;
                        Task.Run(() =>
                        {
                            // Simulate a separate process to complete the user's job.
                            Thread.Sleep(5000);

                            // Perform bookkeeping and send the proactive message.
                            CompleteJob(adapter, appId, conversation, job.JobNumber);
                        });

                        break;

                    default:

                        await context.SendActivity("Type 'run' or 'run job' to start a new job.");

                        break;
                }
            }
        }

        /// <summary>
        /// Creates a simulated job and updates the job log.
        /// </summary>
        /// <param name="context">The current turn context.</param>
        /// <param name="jobLog">The job log.</param>
        /// <returns>A new job.</returns>
        private JobData CreateJob(ITurnContext context, Dictionary<int, JobData> jobLog)
        {
            // Generate a non-duplicate job number;
            int number;
            while (jobLog.ContainsKey(number = NumberGenerator.Next())) { }

            // Simulate creaing the job and logging it.
            var job = new JobData
            {
                JobNumber = number,
                Conversation = TurnContext.GetConversationReference(context.Activity)
            };
            jobLog.Add(job.JobNumber, job);

            // Return the created job.
            return job;
        }

        /// <summary>
        /// Performs bookkeeping and proactively notifies the user that their job completed.
        /// </summary>
        /// <param name="adapter">The bot adapter with which to send the message.</param>
        /// <param name="appId">The app ID of the bot to send the message from.</param>
        /// <param name="conversation">The conversation in which to put the message.</param>
        /// <param name="jobNumber">The number of the job that completed.</param>
        private async void CompleteJob(BotAdapter adapter, string appId, ConversationReference conversation, int jobNumber)
        {
            await adapter.ContinueConversation(appId, conversation, async context =>
            {
                // Get the job log from state, and retrieve the job.
                var jobLog = GetJobLog(context);
                var job = jobLog[jobNumber];

                // Perform bookkeeping.
                job.Completed = true;

                // Send the user a proactive confirmation message.
                await context.SendActivity($"Job {job.JobNumber} is complete.");
            });
        }
    }
}
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

Avant de pouvoir envoyer un message proactif à un utilisateur, il faut que celui-ci envoie au moins un message de style réactif à votre robot. 

Vous devez envoyer un message au robot dans la mesure où celui-ci doit obtenir une référence à l’objet activité et l’enregistrer quelque part en vue d’une utilisation ultérieure. Vous pouvez considérer l’objet activité comme l’adresse des utilisateurs, car il contient des informations sur le canal via lequel ils ont accédé, leur ID utilisateur, l’ID de conversation et même le serveur qui devrait recevoir les messages futurs. Cet objet est un simple JSON qui doit être enregistré entièrement sans altération.

Commençons par un court extrait de code montrant comment enregistrer la référence de conversation chaque fois que l’utilisateur dit « s’abonner » :
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
                var userId = await saveReference(TurnContext.getConversationReference(context.activity));
                await subscribeUser(userId)
                await context.sendActivity(`Thank You! We will message you shortly.`);
               
            } else{
                await context.sendActivity("Say 'subscribe' to start proactive message");
            }
    
        }
    });
});
```
L’extrait de code ci-dessus appelle la fonction `saveReference()` qui enregistrera la référence de l’utilisateur avec `MemoryStorage`, et retourne `userId`. Une fois la référence correctement enregistrée, nous appelons `subscribeUser()` qui informe l’utilisateur qu’il a été abonné. 

La fonction `subscribeUser()` configure l’abonnement réel. Jetons un coup d’œil à une implémentation simple qui démarre un minuteur de 2 secondes, et envoie de façon proactive un message à l’utilisateur une fois le temps du minuteur écoulé :

```javascript
// Persist info to storage
async function saveReference(reference){
    const userId = reference.activityId
    const changes = {};
    changes['reference/' + userId] = reference;
    await storage.write(changes); // Write reference info to persisted storage
    return userId;
}

// Subscribe user to a proactive call. In this case, we are using a setTimeOut() to trigger the proactive call
async function subscribeUser(userId) {
    setTimeout(async () => {
        const reference = await findReference(userId);
        if (reference) {
            await adapter.continueConversation(reference, async (context) => {
                await context.sendActivity("You have been notified");
            });
            
        }
    }, 2000); // Trigger after 2 secs
}

// Read the stored reference info from storage
async function findReference(userId){
    const referenceKey = 'reference/' + userId;
    var rows = await storage.read([referenceKey])
    var reference = await rows[referenceKey]

    return reference;
}
```

La fonction `subscribeUser()` configure un minuteur qui localise l’objet de référence en le lisant dans le stockage. Si l’objet de référence est trouvé, nous pouvons poursuivre la conversation avec l’utilisateur. La méthode `continueConversation` permet au robot d’envoyer de manière proactive des messages à une conversation ou à l’utilisateur avec lequel il a déjà communiqué.

---

## <a name="test-your-bot"></a>Tester votre robot

Pour tester votre robot, déployez-le sur Azure en tant que robot dédié à l’inscription, puis testez-le dans Discussion Web, ou localement à l’aide de l’émulateur.
