---
title: Migrer un bot existant au sein du même projet .NET Framework | Microsoft Docs
description: Nous prenons un bot v3 existant que nous migrons vers le kit SDK v4, à l’aide du même projet.
keywords: migration de bot, formflow, dialogues, bot v3
author: JonathanFingold
ms.author: v-jofing
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 02/11/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 1904bb09d8bd387cc5cec0d85f82df24d1f6ec9d
ms.sourcegitcommit: 7f418bed4d0d8d398f824e951ac464c7c82b8c3e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 02/14/2019
ms.locfileid: "56240175"
---
# <a name="migrate-a-net-sdk-v3-bot-to-v4"></a>Migrer un bot du SDK .NET v3 vers v4

Dans cet article, nous allons convertir [ContosoHelpdeskChatBot](https://github.com/Microsoft/intelligent-apps/tree/master/ContosoHelpdeskChatBot/ContosoHelpdeskChatBot) v3 en bot v4 _sans convertir le type de projet_. Il restera un projet .NET Framework.
Cette conversion est décomposée en plusieurs étapes :

1. Mettre à jour et installer les packages NuGet
1. Mettre à jour votre fichier Global.asax.cs
1. Mettre à jour votre classe MessagesController
1. Convertir vos dialogues

<!--TODO: Link to the converted bot...[ContosoHelpdeskChatBot](https://github.com/EricDahlvang/intelligent-apps/tree/v4netframework/ContosoHelpdeskChatBot).-->

Le kit SDK Bot Framework v4 est basé sur la même API REST sous-jacente que le kit SDK v3. Toutefois, le kit SDK v4 est une refactorisation de la version précédente du kit pour offrir aux développeurs plus de flexibilité et de contrôle sur leurs bots. Les principaux changements du kit SDK sont notamment les suivants :

- L’état est géré via des objets de gestion d’état et des accesseurs de propriété.
- La configuration du gestionnaire de tour et la transmission des activités à ce dernier ont changé.
- Les dialogues scorables n’existent plus. Vous pouvez rechercher des commandes « globales » dans le gestionnaire de tour, avant de passer le contrôle à vos dialogues.
- Nouvelle bibliothèque de dialogues qui est très différente de celle de la version précédente. Vous devez convertir les anciens dialogues dans le nouveau système de dialogues à l’aide de dialogues composants et en cascade ainsi que de l’implémentation de la communauté de dialogues Formflow pour v4.

Pour plus d’informations sur des changements spécifiques, consultez [Différences entre le SDK .NET v3 et v4](migration-about.md).

## <a name="update-and-install-nuget-packages"></a>Mettre à jour et installer les packages NuGet

1. Effectuez la mise à jour de **Microsoft.Bot.Builder.Azure** vers la dernière version stable.

    Vous mettez ainsi également à jour les packages **Microsoft.Bot.Builder** et **Microsoft.Bot.Connector**, dans la mesure où ce sont des dépendances.

1. Supprimez le package **Microsoft.Bot.Builder.History**. Il ne fait pas partie du kit SDK v4.
1. Ajoutez **Autofac.WebApi2**

    Nous l’utiliserons pour faciliter l’injection de dépendances dans ASP.NET.

1. Ajoutez **Bot.Builder.Community.Dialogs.Formflow**

    Il s’agit d’une bibliothèque de communauté pour la génération de dialogues v4 à partir de fichiers de définition Formflow v3. Comme **Microsoft.Bot.Builder.Dialogs** est l’une de ses dépendances, elle est également installée automatiquement.

Si vous procédez à la génération à ce stade, vous obtenez des erreurs du compilateur. Vous pouvez les ignorer. Une fois la conversion terminée, nous obtenons un code fonctionnel.

<!--
## Add a BotDataBag class

This file will contain wrappers to add a v3-style **IBotDataBag** to make dialog conversion simpler.

```csharp
using System.Collections.Generic;

namespace ContosoHelpdeskChatBot
{
    public class BotDataBag : Dictionary<string, object>, IBotDataBag
    {
        public bool RemoveValue(string key)
        {
            return base.Remove(key);
        }

        public void SetValue<T>(string key, T value)
        {
            this[key] = value;
        }

        public bool TryGetValue<T>(string key, out T value)
        {
            if (!ContainsKey(key))
            {
                value = default(T);
                return false;
            }

            value = (T)this[key];

            return true;
        }
    }

    public interface IBotDataBag
    {
        int Count { get; }

        void Clear();

        bool ContainsKey(string key);

        bool RemoveValue(string key);

        void SetValue<T>(string key, T value);

        bool TryGetValue<T>(string key, out T value);
    }
}
```
-->

## <a name="update-your-globalasaxcs-file"></a>Mettre à jour votre fichier Global.asax.cs

Une partie de la génération de modèles automatique a changé et nous devons configurer des composants de l’infrastructure de [gestion d’état](/articles/v4sdk/bot-builder-concept-state.md) nous-mêmes dans v4. Par exemple, v4 utilise un adaptateur de bot pour gérer l’authentification et transférer des activités au code de votre bot, et nous devons déclarer les propriétés d’état en amont.

Nous allons créer une propriété d’état pour `DialogState`, dont nous avons maintenant besoin pour la prise en charge des dialogues dans v4. Nous allons utiliser l’injection de dépendances pour faire parvenir les informations nécessaires au contrôleur et au code de bot.

Dans **Global.asax.cs** :

1. Mettez à jour les instructions `using` :
    ```csharp
    using System.Configuration;
    using System.Reflection;
    using System.Web.Http;
    using Autofac;
    using Autofac.Integration.WebApi;
    using Microsoft.Bot.Builder;
    using Microsoft.Bot.Builder.Dialogs;
    using Microsoft.Bot.Connector.Authentication;
    ```
1. Supprimez ces lignes de la méthode **Application_Start**
    ```csharp
    BotConfig.UpdateConversationContainer();
    this.RegisterBotModules();
    ```
    Insérez ensuite cette ligne.
    ```csharp
    GlobalConfiguration.Configure(BotConfig.Register);
    ```
1. Supprimez la méthode **RegisterBotModules** qui n’est plus référencée.
1. Remplacez la méthode **BotConfig.UpdateConversationContainer** par cette méthode **BotConfig.Register**, où nous allons inscrire les objets nécessaires pour prendre en charge l’injection de dépendances.
    > [!NOTE]
    > Ce bot n’utilise pas l’état _utilisateur_ ni des _conversations privées_. Les lignes pour les inclure sont commentées ici.
    ```csharp
    public static void Register(HttpConfiguration config)
    {
        ContainerBuilder builder = new ContainerBuilder();
        builder.RegisterApiControllers(Assembly.GetExecutingAssembly());

        SimpleCredentialProvider credentialProvider = new SimpleCredentialProvider(
            ConfigurationManager.AppSettings[MicrosoftAppCredentials.MicrosoftAppIdKey],
            ConfigurationManager.AppSettings[MicrosoftAppCredentials.MicrosoftAppPasswordKey]);

        builder.RegisterInstance(credentialProvider).As<ICredentialProvider>();

        // The Memory Storage used here is for local bot debugging only. When the bot
        // is restarted, everything stored in memory will be gone.
        IStorage dataStore = new MemoryStorage();

        // Create Conversation State object.
        // The Conversation State object is where we persist anything at the conversation-scope.
        ConversationState conversationState = new ConversationState(dataStore);
        builder.RegisterInstance(conversationState).As<ConversationState>();

        //var userState = new UserState(dataStore);
        //var privateConversationState = new PrivateConversationState(dataStore);

        // Create the dialog state property acccessor.
        IStatePropertyAccessor<DialogState> dialogStateAccessor
            = conversationState.CreateProperty<DialogState>(nameof(DialogState));
        builder.RegisterInstance(dialogStateAccessor).As<IStatePropertyAccessor<DialogState>>();

        IContainer container = builder.Build();
        AutofacWebApiDependencyResolver resolver = new AutofacWebApiDependencyResolver(container);
        config.DependencyResolver = resolver;
    }
    ```

## <a name="update-your-messagescontroller-class"></a>Mettre à jour votre classe MessagesController

C’est là où le gestionnaire de tour intervient dans v4 et les changements sont donc nombreux. Sauf pour le gestionnaire de tour proprement dit, la majeure partie peut être considérée comme réutilisable. Dans votre fichier **Controllers\MessagesController.cs** :

1. Mettez à jour les instructions `using` :
    ```csharp
    using System;
    using System.Net;
    using System.Net.Http;
    using System.Threading;
    using System.Threading.Tasks;
    using System.Web.Http;
    using Bot.Builder.Community.Dialogs.FormFlow;
    using ContosoHelpdeskChatBot.Dialogs;
    using Microsoft.Bot.Builder;
    using Microsoft.Bot.Builder.Dialogs;
    using Microsoft.Bot.Connector.Authentication;
    using Microsoft.Bot.Schema;
    ```
1. Supprimez l’attribut `[BotAuthentication]` de la classe. Dans v4, l’adaptateur du bot gère l’authentification.
1. Ajoutez ces champs. **ConversationState** gère l’état limité à la conversation et **IStatePropertyAccessor\<DialogState>** est nécessaire pour prendre en charge les dialogues dans v4.
    ```csharp
    private readonly ConversationState _conversationState;
    private readonly ICredentialProvider _credentialProvider;
    private readonly IStatePropertyAccessor<DialogState> _dialogData;

    private readonly DialogSet _dialogs;
    ```
1. Ajoutez un constructeur pour :
    - Initialiser les champs d’instance.
    - Utiliser l’injection de dépendances dans ASP.NET pour obtenir les valeurs de paramètres. (Nous avons inscrit des instances de ces classes dans **Global.asax.cs** pour assurer cette prise en charge.)
    - Créer et initialiser un jeu de dialogues, à partir duquel nous pouvons créer un contexte de dialogue. (Nous devons le faire explicitement dans v4.)
    ```csharp
    public MessagesController(
        ConversationState conversationState,
        ICredentialProvider credentialProvider,
        IStatePropertyAccessor<DialogState> dialogData)
    {
        _conversationState = conversationState;
        _dialogData = dialogData;
        _credentialProvider = credentialProvider;

        _dialogs = new DialogSet(dialogData);
        _dialogs.Add(new RootDialog(nameof(RootDialog)));
    }
    ```
1. Remplacez le corps de la méthode **Post**. Il s’agit de l’endroit où nous allons créer l’adaptateur et l’utiliser pour appeler notre boucle de messages (gestionnaire de tour). Nous utilisons `SaveChangesAsync` à la fin de chaque tour pour enregistrer les changements que le bot a apportés à l’état.

    ```csharp
    public async Task<HttpResponseMessage> Post([FromBody]Activity activity)
    {

        var botFrameworkAdapter = new BotFrameworkAdapter(_credentialProvider);

        var invokeResponse = await botFrameworkAdapter.ProcessActivityAsync(
            Request.Headers.Authorization?.ToString(),
            activity,
            OnTurnAsync,
            default(CancellationToken));

        if (invokeResponse == null)
        {
            return Request.CreateResponse(HttpStatusCode.OK);
        }
        else
        {
            return Request.CreateResponse(invokeResponse.Status);
        }
    }
    ```
1. Ajoutez une méthode **OnTurnAsync** qui contient le code de [gestionnaire de tour](/articles/v4sdk/bot-builder-basics.md#the-activity-processing-stack) du bot.
    > [!NOTE]
    > Les dialogues scorables n’existent pas en tant que tels dans v4. Nous recherchons un message `cancel` de l’utilisateur dans le gestionnaire de tour du bot avant de poursuivre tout dialogue actif.
    ```csharp
    protected async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken)
    {
        // We're only handling message activities in this bot.
        if (turnContext.Activity.Type == ActivityTypes.Message)
        {
            // Create the dialog context for our dialog set.
            DialogContext dc = await _dialogs.CreateContextAsync(turnContext, cancellationToken);

            // Globally interrupt the dialog stack if the user sent 'cancel'.
            if (turnContext.Activity.Text.Equals("cancel", StringComparison.InvariantCultureIgnoreCase))
            {
                Activity reply = turnContext.Activity.CreateReply($"Ok restarting conversation.");
                await turnContext.SendActivityAsync(reply);
                await dc.CancelAllDialogsAsync();
            }

            try
            {
                // Continue the active dialog, if any. If we just cancelled all dialog, the
                // dialog stack will be empty, and this will return DialogTurnResult.Empty.
                DialogTurnResult dialogResult = await dc.ContinueDialogAsync();
                switch (dialogResult.Status)
                {
                    case DialogTurnStatus.Empty:
                        // There was no active dialog in the dialog stack; start the root dialog.
                        await dc.BeginDialogAsync(nameof(RootDialog));
                        break;

                    case DialogTurnStatus.Complete:
                        // The last dialog on the stack completed and the stack is empty.
                        await dc.EndDialogAsync();
                        break;

                    case DialogTurnStatus.Waiting:
                    case DialogTurnStatus.Cancelled:
                        // The active dialog is waiting for a response from the user, or all
                        // dialogs were cancelled and the stack is empty. In either case, we
                        // don't need to do anything here.
                        break;
                }
            }
            catch (FormCanceledException)
            {
                // One of the dialogs threw an exception to clear the dialog stack.
                await turnContext.SendActivityAsync("Cancelled.");
                await dc.CancelAllDialogsAsync();
                await dc.BeginDialogAsync(nameof(RootDialog));
            }
        }
    }
    ```
1. Comme nous gérons uniquement les activités de _message_, nous pouvons supprimer la méthode **HandleSystemMessage**.

### <a name="delete-the-cancelscorable-and-globalmessagehandlersbotmodule-classes"></a>Supprimer les classes CancelScorable et GlobalMessageHandlersBotModule

Comme les dialogues scorables n’existent pas dans v4 et que nous avons mis à jour le gestionnaire de tour pour réagir à un message `cancel`, nous pouvons supprimer les classes **CancelScorable** (dans **Dialogs\CancelScorable.cs**) et **GlobalMessageHandlersBotModule**.

## <a name="convert-your-dialogs"></a>Convertir vos dialogues

Nous allons apporter de nombreux changements aux dialogues d’origine pour les migrer vers le kit SDK v4. Ne vous souciez pas des erreurs du compilateur pour l’instant. Elles seront résolues une fois la conversion terminée.
Dans l’intérêt de ne pas modifier le code d’origine plus que nécessaire, il restera des avertissements du compilateur une fois la migration terminée.

Tous nos dialogues dérivent de `ComponentDialog`, au lieu d’implémenter l’interface `IDialog<object>` de v3.

Ce bot contient quatre dialogues que nous devons convertir :

| | |
|---|---|
| [RootDialog](#update-the-root-dialog) | Présente les options et démarre les autres dialogues. |
| [InstallAppDialog](#update-the-install-app-dialog) | Gère les demandes d’installation d’une application sur un ordinateur. |
| [LocalAdminDialog](#update-the-local-admin-dialog) | Gère les demandes de droits d’administrateur sur l’ordinateur local. |
| [ResetPasswordDialog](#update-the-reset-password-dialog) | Gère les demandes de réinitialisation de votre mot de passe. |

Ces dialogues recueillent des entrées, mais n’effectuent aucune de ces opérations sur votre ordinateur.

### <a name="make-solution-wide-dialog-changes"></a>Apporter des changements de dialogue à l’échelle de la solution

1. Pour l’ensemble de la solution, remplacez toutes les occurrences de `IDialog<object>` par `ComponentDialog`.
1. Pour l’ensemble de la solution, remplacez toutes les occurrences de `IDialogContext` par `DialogContext`.
1. Pour chaque classe de dialogue, supprimez l’attribut `[Serializable]`.

Le flux de contrôle et la messagerie au sein des dialogues n’étant plus gérés de la même façon, nous allons devoir examiner ce point lors de la conversion de chaque dialogue.

| Opération | Code v3 | Code v4 |
| :--- | :--- | :--- |
| Gérer le début de votre dialogue | Implémentez `IDialog.StartAsync` | Définissez ceci comme première étape d’un dialogue en cascade ou implémentez `Dialog.BeginDialogAsync` |
| Gérer la continuation de votre dialogue | Appelez `IDialogContext.Wait` | Ajoutez des étapes supplémentaires à un dialogue en cascade ou implémentez `Dialog.ContinueDialogAsync` |
| Envoyer un message à l’utilisateur | Appelez `IDialogContext.PostAsync` | Appelez `ITurnContext.SendActivityAsync` |
| Démarrer un dialogue enfant | Appelez `IDialogContext.Call` | Appelez `DialogContext.BeginDialogAsync` |
| Signaler que le dialogue en cours est terminé | Appelez `IDialogContext.Done` | Appelez `DialogContext.EndDialogAsync` |
| Obtenir l’entrée de l’utilisateur | Utilisez un paramètre `IAwaitable<IMessageActivity>` | Utilisez une invite à partir d’une cascade, ou `ITurnContext.Activity` |

Remarques concernant le code v4 :

- Utilisez la propriété `DialogContext.Context` pour obtenir le contexte de tour actuel.
- Les étapes en cascade ont un paramètre `WaterfallStepContext`, qui dérive de `DialogContext`.
- Toutes les classes de dialogue et d’invite concrètes dérivent de la classe `Dialog` abstraite.
- Vous affectez un ID quand vous créez un dialogue composant. Chaque dialogue d’un jeu de dialogues doit recevoir un ID unique dans ce jeu.

### <a name="update-the-root-dialog"></a>Mettre à jour le dialogue racine

Dans ce bot, le dialogue racine invite l’utilisateur à choisir parmi un ensemble d’options, puis démarre un dialogue enfant en fonction de ce choix. Celui-ci effectue ensuite une boucle pendant la durée de vie de la conversation.

- Nous pouvons définir le flux principal opérationnel comme dialogue en cascade, ce qui est un nouveau concept dans le kit SDK v4. Il s’exécute via un ensemble fixe d’étapes dans l’ordre. Pour plus d’informations, consultez [Implémenter des flux de conversation séquentiels](/articles/v4sdk/bot-builder-dialog-manage-conversation-flow).
- L’invite est désormais gérée par le biais de classes d’invite, qui sont des dialogues enfants courts qui demandent une entrée, effectuent un traitement minimal et une validation, et retournent une valeur. Pour plus d’informations, consultez [Collecter les entrées utilisateur avec une invite de dialogue](/articles/v4sdk/bot-builder-prompts.md).

Dans le fichier **Dialogs/RootDialog.cs** :

1. Mettez à jour les instructions `using` :
    ```csharp
    using System;
    using System.Collections.Generic;
    using System.Threading;
    using System.Threading.Tasks;
    using Microsoft.Bot.Builder;
    using Microsoft.Bot.Builder.Dialogs;
    using Microsoft.Bot.Builder.Dialogs.Choices;
    ```
1. Nous devons convertir les options `HelpdeskOptions` à partir d’une liste de chaînes vers une liste de choix. Celle-ci sera utilisée avec une invite de choix, qui accepte le numéro du choix (dans la liste), la valeur du choix ou l’un des synonymes du choix en tant qu’entrée valide.
    ```csharp
    private static List<Choice> HelpdeskOptions = new List<Choice>()
    {
        new Choice(InstallAppOption) { Synonyms = new List<string>(){ "install" } },
        new Choice(ResetPasswordOption) { Synonyms = new List<string>(){ "password" } },
        new Choice(LocalAdminOption)  { Synonyms = new List<string>(){ "admin" } }
    };
    ```
1. Ajoutez un constructeur. Ce code effectue les actions suivantes :
   - Un ID est affecté à chaque instance d’un dialogue lors de sa création. L’ID de dialogue fait partie du jeu de dialogues auquel le dialogue est ajouté. Rappelez-vous que le bot possède un jeu de dialogues qui a été initialisé dans la classe **MessageController**. Chaque `ComponentDialog` a son propre jeu de dialogues interne, avec son propre ensemble d’ID de dialogue.
   - Il ajoute les autres dialogues, dont l’invite de choix, comme dialogues enfants. Ici, nous utilisons simplement le nom de classe pour chaque ID de dialogue.
   - Il définit un dialogue en cascade en trois étapes. Nous allons les implémenter dans un instant.
     - Le dialogue invite tout d’abord l’utilisateur à choisir une tâche à effectuer.
     - Il démarre ensuite le dialogue enfant associé à ce choix.
     - Pour finir, il redémarre.
   - Chaque étape de la cascade est un délégué et nous allons les implémenter ensuite, en prenant le code existant du dialogue d’origine là où nous pouvons.
   - Quand vous démarrez un dialogue composant, celui-ci démarre son _dialogue initial_. Par défaut, il s’agit du premier dialogue enfant ajouté à un dialogue composant. Pour affecter un autre enfant en tant que dialogue initial, vous devez définir manuellement la propriété `InitialDialogId` du composant.
    ```csharp
    public RootDialog(string id)
        : base(id)
    {
        AddDialog(new WaterfallDialog("choiceswaterfall", new WaterfallStep[]
            {
                PromptForOptionsAsync,
                ShowChildDialogAsync,
                ResumeAfterAsync,
            }));
        AddDialog(new InstallAppDialog(nameof(InstallAppDialog)));
        AddDialog(new LocalAdminDialog(nameof(LocalAdminDialog)));
        AddDialog(new ResetPasswordDialog(nameof(ResetPasswordDialog)));
        AddDialog(new ChoicePrompt("options"));
    }
    ```
1. Nous pouvons supprimer la méthode **StartAsync**. Quand un dialogue composant commence, il démarre automatiquement son dialogue _initial_. Dans ce cas, il s’agit du dialogue en cascade que nous avons défini dans le constructeur. Il démarre aussi automatiquement à sa première étape.
1. Nous allons supprimer les méthodes **MessageReceivedAsync** et **ShowOptions**, puis les remplacer par la première étape de notre dialogue en cascade. Ces deux méthodes accueillent l’utilisateur et lui demandent de choisir l’une des options disponibles.
   - Vous pouvez voir ici que la liste de choix ainsi que les messages d’accueil et d’erreur sont fournis en tant qu’options dans l’appel à l’invite de nos choix.
   - Nous n’avons pas à spécifier la méthode suivante à appeler dans le dialogue, car la cascade passe à l’étape suivante à l’issue de l’invite de choix.
   - L’invite de choix effectue une boucle jusqu’à ce qu’elle reçoive une entrée valide ou que toute la pile de dialogues soit annulée.
    ```csharp
    private async Task<DialogTurnResult> PromptForOptionsAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        // Prompt the user for a response using our choice prompt.
        return await stepContext.PromptAsync(
            "options",
            new PromptOptions()
            {
                Choices = HelpdeskOptions,
                Prompt = MessageFactory.Text(GreetMessage),
                RetryPrompt = MessageFactory.Text(ErrorMessage)
            },
            cancellationToken);
    }
    ```
1. Nous pouvons remplacer **OnOptionSelected** par la deuxième étape de notre cascade. Nous commençons toujours un dialogue enfant basé sur l’entrée de l’utilisateur.
   - L’invite de choix retourne une valeur `FoundChoice`. Cela s’affiche dans la propriété `Result` du contexte d’étape. La pile de dialogues traite toutes les valeurs de retour comme des objets. Si la valeur de retour provient d’un de vos dialogues, vous connaissez le type de valeur de l’objet. Consultez [Types d’invites](/articles/v4sdk/bot-builder-concept-dialog.md#prompt-types) pour obtenir une liste ce que chaque type d’invite retourne.
   - Étant donné que l’invite de choix ne va pas lever d’exception, le bloc try-catch peut être supprimé.
   - Nous devons ajouter un passage afin que cette méthode retourne toujours une valeur appropriée. Ce code ne doit jamais être atteint mais, si c’est le cas, il permettra au dialogue d’« échouer de manière appropriée ».
    ```csharp
    private async Task<DialogTurnResult> ShowChildDialogAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        // string optionSelected = await userReply;
        string optionSelected = (stepContext.Result as FoundChoice).Value;

        switch (optionSelected)
        {
            case InstallAppOption:
                //context.Call(new InstallAppDialog(), this.ResumeAfterOptionDialog);
                //break;
                return await stepContext.BeginDialogAsync(
                    nameof(InstallAppDialog),
                    cancellationToken);
            case ResetPasswordOption:
                //context.Call(new ResetPasswordDialog(), this.ResumeAfterOptionDialog);
                //break;
                return await stepContext.BeginDialogAsync(
                    nameof(ResetPasswordDialog),
                    cancellationToken);
            case LocalAdminOption:
                //context.Call(new LocalAdminDialog(), this.ResumeAfterOptionDialog);
                //break;
                return await stepContext.BeginDialogAsync(
                    nameof(LocalAdminDialog),
                    cancellationToken);
        }

        // We shouldn't get here, but fail gracefully if we do.
        await stepContext.Context.SendActivityAsync(
            "I don't recognize that option.",
            cancellationToken: cancellationToken);
        // Continue through to the next step without starting a child dialog.
        return await stepContext.NextAsync(cancellationToken: cancellationToken);
    }
    ```
1. Enfin, remplacez l’ancienne méthode **ResumeAfterOptionDialog** par la dernière étape de notre cascade.
    - Au lieu de mettre fin au dialogue et de retourner le numéro de ticket comme nous l’avons fait dans le dialogue d’origine, nous redémarrons la cascade en remplaçant l’instance d’origine sur la pile par une nouvelle instance de lui-même. Cette opération est possible, car l’application d’origine a toujours ignoré la valeur de retour (le numéro de ticket) et redémarré le dialogue racine.
    ```csharp
    private async Task<DialogTurnResult> ResumeAfterAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        try
        {
            //var message = await userReply;
            var message = stepContext.Context.Activity;

            int ticketNumber = new Random().Next(0, 20000);
            //await context.PostAsync($"Thank you for using the Helpdesk Bot. Your ticket number is {ticketNumber}.");
            await stepContext.Context.SendActivityAsync(
                $"Thank you for using the Helpdesk Bot. Your ticket number is {ticketNumber}.",
                cancellationToken: cancellationToken);

            //context.Done(ticketNumber);
        }
        catch (Exception ex)
        {
            // await context.PostAsync($"Failed with message: {ex.Message}");
            await stepContext.Context.SendActivityAsync(
                $"Failed with message: {ex.Message}",
                cancellationToken: cancellationToken);

            // In general resume from task after calling a child dialog is a good place to handle exceptions
            // try catch will capture exceptions from the bot framework awaitable object which is essentially "userReply"
            logger.Error(ex);
        }

        // Replace on the stack the current instance of the waterfall with a new instance,
        // and start from the top.
        return await stepContext.ReplaceDialogAsync(
            "choiceswaterfall",
            cancellationToken: cancellationToken);
    }
    ```

### <a name="update-the-install-app-dialog"></a>Mettre à jour le dialogue d’installation d’une application

Le dialogue d’installation d’une application effectue quelques tâches logiques que nous allons configurer comme un dialogue en cascade en 4 étapes. La façon dont vous factorisez le code existant en étapes en cascade est un exercice logique pour chaque dialogue. Pour chaque étape, la méthode d’origine de laquelle provient le code est indiquée.

1. Demande à l’utilisateur une chaîne à rechercher.
1. Recherche des correspondances potentielles dans une base de données.
   - S’il existe une correspondance, il la sélectionne et continue.
   - S’il existe plusieurs correspondances, il invite l’utilisateur à en choisir une.
   - S’il n’existe aucune correspondance, le dialogue s’arrête.
1. Demande à l’utilisateur un ordinateur sur lequel installer l’application.
1. Écrit les informations dans une base de données et envoie un message de confirmation.

Dans le fichier **Dialogs/InstallAppDialog.cs** :

1. Mettez à jour les instructions `using` :
    ```csharp
    using System.Collections.Generic;
    using System.Linq;
    using System.Threading;
    using System.Threading.Tasks;
    using ContosoHelpdeskChatBot.Models;
    using Microsoft.Bot.Builder;
    using Microsoft.Bot.Builder.Dialogs;
    using Microsoft.Bot.Builder.Dialogs.Choices;
    ```
1. Définissez des constantes pour les ID que nous allons utiliser pour les invites et les dialogues. Cela facilite la gestion du code de dialogue, car la chaîne à utiliser est définie à un seul emplacement.
    ```csharp
    // Set up our dialog and prompt IDs as constants.
    private const string MainId = "mainDialog";
    private const string TextId = "textPrompt";
    private const string ChoiceId = "choicePrompt";
    ```
1. Définissez des constantes pour les clés que nous allons utiliser pour suivre l’état du dialogue.
    ```csharp
    // Set up keys for managing collected information.
    private const string InstallInfo = "installInfo";
    ```
1. Ajoutez un constructeur et initialisez le jeu de dialogues du composant. Nous définissons explicitement la propriété `InitialDialogId` cette fois, ce qui signifie que le dialogue en cascade principal n’a pas à être le premier que vous ajoutez au jeu. Par exemple, si vous préférez ajouter d’abord les invites, cela vous permet de le faire sans provoquer un problème d’exécution.
    ```csharp
    public InstallAppDialog(string id)
        : base(id)
    {
        // Initialize our dialogs and prompts.
        InitialDialogId = MainId;
        AddDialog(new WaterfallDialog(MainId, new WaterfallStep[] {
            GetSearchTermAsync,
            ResolveAppNameAsync,
            GetMachineNameAsync,
            SubmitRequestAsync,
        }));
        AddDialog(new TextPrompt(TextId));
        AddDialog(new ChoicePrompt(ChoiceId));
    }
    ```
1. Nous pouvons remplacer **StartAsync** par la première étape de notre dialogue en cascade.
    - Comme nous devons gérer l’état nous-mêmes, nous allons suivre l’objet d’installation d’une application dans l’état du dialogue.
    - Le message demandant à l’utilisateur de fournir une entrée devient une option dans l’appel à l’invite.
    ```csharp
    private async Task<DialogTurnResult> GetSearchTermAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        // Create an object in dialog state in which to track our collected information.
        stepContext.Values[InstallInfo] = new InstallApp();

        // Ask for the search term.
        return await stepContext.PromptAsync(
            TextId,
            new PromptOptions
            {
                Prompt = MessageFactory.Text("Ok let's get started. What is the name of the application? "),
            },
            cancellationToken);
    }
    ```
1. Nous pouvons remplacer **appNameAsync** et **multipleAppsAsync** par la deuxième étape de notre cascade.
    - Nous obtenons le résultat de l’invite maintenant, au lieu de regarder simplement le dernier message de l’utilisateur.
    - La requête de base de données et les instructions if sont organisées de la même façon que dans **appNameAsync**. Le code dans chaque bloc de l’instruction if a été mis à jour pour fonctionner avec les dialogues v4.
        - S’il existe une correspondance, nous allons mettre à jour l’état du dialogue et passer à l’étape suivante.
        - S’il existe plusieurs correspondances, nous allons utiliser l’invite de nos choix pour demander à l’utilisateur de choisir parmi la liste des options. Cela signifie que nous pouvons simplement supprimer **multipleAppsAsync**.
        - S’il n’existe aucune correspondance, nous allons mettre fin à ce dialogue et retourner la valeur null au dialogue racine.
    ```csharp
    private async Task<DialogTurnResult> ResolveAppNameAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        // Get the result from the text prompt.
        var appname = stepContext.Result as string;

        // Query the database for matches.
        var names = await this.getAppsAsync(appname);

        if (names.Count == 1)
        {
            // Get our tracking information from dialog state and add the app name.
            var install = stepContext.Values[InstallInfo] as InstallApp;
            install.AppName = names.First();

            return await stepContext.NextAsync();
        }
        else if (names.Count > 1)
        {
            // Ask the user to choose from the list of matches.
            return await stepContext.PromptAsync(
                ChoiceId,
                new PromptOptions
                {
                    Prompt = MessageFactory.Text("I found the following applications. Please choose one:"),
                    Choices = ChoiceFactory.ToChoices(names),
                },
                cancellationToken);
        }
        else
        {
            // If no matches, exit this dialog.
            await stepContext.Context.SendActivityAsync(
                $"Sorry, I did not find any application with the name '{appname}'.",
                cancellationToken: cancellationToken);

            return await stepContext.EndDialogAsync(null, cancellationToken);
        }
    }
    ```
1. **appNameAsync** demande également à l’utilisateur le nom de son ordinateur une fois la requête résolue. Nous allons capturer cette partie de la logique à l’étape suivante de la cascade.
    - Là encore, dans v4, nous devons gérer l’état nous-mêmes. La seule difficulté ici est que nous pouvons parvenir à cette étape via deux branches logiques différentes de l’étape précédente.
    - Nous allons demander à l’utilisateur un nom d’ordinateur à l’aide de la même invite de texte que précédemment, en fournissant simplement différentes options cette fois.
    ```csharp
    private async Task<DialogTurnResult> GetMachineNameAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        // Get the tracking info. If we don't already have an app name,
        // Then we used the choice prompt to get it in the previous step.
        var install = stepContext.Values[InstallInfo] as InstallApp;
        if (install.AppName is null)
        {
            install.AppName = (stepContext.Result as FoundChoice).Value;
        }

        // We now need the machine name, so prompt for it.
        return await stepContext.PromptAsync(
            TextId,
            new PromptOptions
            {
                Prompt = MessageFactory.Text(
                    $"Found {install.AppName}. What is the name of the machine to install application?"),
            },
            cancellationToken);
    }
    ```
1. La logique de **machineNameAsync** est encapsulée dans la dernière étape de notre cascade.
    - Nous récupérons le nom de l’ordinateur à partir du résultat de l’invite de texte et mettons à jour l’état du dialogue.
    - Nous supprimons l’appel pour mettre à jour la base de données, comme le code de prise en charge est dans un autre projet.
    - Nous envoyons ensuite le message de réussite à l’utilisateur et mettons fin au dialogue.
    ```csharp
    private async Task<DialogTurnResult> SubmitRequestAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        if (stepContext.Reason != DialogReason.CancelCalled)
        {
            // Get the tracking info and add the machine name.
            var install = stepContext.Values[InstallInfo] as InstallApp;
            install.MachineName = stepContext.Context.Activity.Text;

            //TODO: Save to this information to the database.
        }

        await stepContext.Context.SendActivityAsync(
            $"Great, your request to install {install.AppName} on {install.MachineName} has been scheduled.",
            cancellationToken: cancellationToken);

        return await stepContext.EndDialogAsync(null, cancellationToken);
    }
    ```
1. Pour simuler la base de données, j’ai mis à jour **getAppsAsync** pour interroger une liste statique au lieu de la base de données.
    ```csharp
    private async Task<List<string>> getAppsAsync(string Name)
    {
        List<string> names = new List<string>();

        // Simulate querying the database for applications that match.
        return (from app in AppMsis
            where app.ToLower().Contains(Name.ToLower())
            select app).ToList();
    }

    // Example list of app names in the database.
    private static readonly List<string> AppMsis = new List<string>
    {
        "µTorrent 3.5.0.44178",
        "7-Zip 17.1",
        "Ad-Aware 9.0",
        "Adobe AIR 2.5.1.17730",
        "Adobe Flash Player (IE) 28.0.0.105",
        "Adobe Flash Player (Non-IE) 27.0.0.130",
        "Adobe Reader 11.0.14",
        "Adobe Shockwave Player 12.3.1.201",
        "Advanced SystemCare Personal 11.0.3",
        "Auslogics Disk Defrag 3.6",
        "avast! 4 Home Edition 4.8.1351",
        "AVG Anti-Virus Free Edition 9.0.0.698",
        "Bonjour 3.1.0.1",
        "CCleaner 5.24.5839",
        "Chmod Calculator 20132.4",
        "CyberLink PowerDVD 17.0.2101.62",
        "DAEMON Tools Lite 4.46.1.328",
        "FileZilla Client 3.5",
        "Firefox 57.0",
        "Foxit Reader 4.1.1.805",
        "Google Chrome 66.143.49260",
        "Google Earth 7.3.0.3832",
        "Google Toolbar (IE) 7.5.8231.2252",
        "GSpot 2701.0",
        "Internet Explorer 903235.0",
        "iTunes 12.7.0.166",
        "Java Runtime Environment 6 Update 17",
        "K-Lite Codec Pack 12.1",
        "Malwarebytes Anti-Malware 2.2.1.1043",
        "Media Player Classic 6.4.9.0",
        "Microsoft Silverlight 5.1.50907",
        "Mozilla Thunderbird 57.0",
        "Nero Burning ROM 19.1.1005",
        "OpenOffice.org 3.1.1 Build 9420",
        "Opera 12.18.1873",
        "Paint.NET 4.0.19",
        "Picasa 3.9.141.259",
        "QuickTime 7.79.80.95",
        "RealPlayer SP 12.0.0.319",
        "Revo Uninstaller 1.95",
        "Skype 7.40.151",
        "Spybot - Search & Destroy 1.6.2.46",
        "SpywareBlaster 4.6",
        "TuneUp Utilities 2009 14.0.1000.353",
        "Unlocker 1.9.2",
        "VLC media player 1.1.6",
        "Winamp 5.56 Build 2512",
        "Windows Live Messenger 2009 16.4.3528.331",
        "WinPatrol 2010 31.0.2014",
        "WinRAR 5.0",
    };
    ```

### <a name="update-the-local-admin-dialog"></a>Mettre à jour le dialogue d’administrateur local

Dans v3, ce dialogue accueille l’utilisateur, démarre le dialogue Formflow, puis enregistre le résultat dans une base de données. Cela se traduit facilement en une cascade en deux étapes.

1. Mettez à jour les instructions `using`. Notez que ce dialogue inclut un dialogue Formflow v3. Dans v4, nous pouvons utiliser la bibliothèque Formflow de communauté.
    ```csharp
    using System.Threading;
    using System.Threading.Tasks;
    using Bot.Builder.Community.Dialogs.FormFlow;
    using ContosoHelpdeskChatBot.Models;
    using Microsoft.Bot.Builder.Dialogs;
    ```
1. Nous pouvons supprimer la propriété d’instance pour `LocalAdmin`, comme le résultat sera disponible dans l’état du dialogue.
1. Définissez des constantes pour les ID que nous allons utiliser pour les dialogues. Dans la bibliothèque de communauté, l’ID de dialogue construit est toujours défini sur le nom de la classe produite par le dialogue.
    ```csharp
    // Set up our dialog and prompt IDs as constants.
    private const string MainDialog = "mainDialog";
    private static string AdminDialog { get; } = nameof(LocalAdminPrompt);
    ```
1. Ajoutez un constructeur et initialisez le jeu de dialogues du composant. Le dialogue Formflow est créé de la même façon. Nous allons simplement l’ajouter au jeu de dialogues de notre composant dans le constructeur.
    ```csharp
    public LocalAdminDialog(string dialogId) : base(dialogId)
    {
        InitialDialogId = MainDialog;

        AddDialog(new WaterfallDialog(MainDialog, new WaterfallStep[]
        {
            BeginFormflowAsync,
            SaveResultAsync,
        }));
        AddDialog(FormDialog.FromForm(BuildLocalAdminForm, FormOptions.PromptInStart));
    }
    ```
1. Nous pouvons remplacer **StartAsync** par la première étape de notre dialogue en cascade. Nous avons déjà créé le Formflow dans le constructeur et les deux autres instructions sont traduites comme suit.
    ```csharp
    private async Task<DialogTurnResult> BeginFormflowAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        await stepContext.Context.SendActivityAsync("Great I will help you request local machine admin.");

        // Begin the Formflow dialog.
        return await stepContext.BeginDialogAsync(AdminDialog, cancellationToken: cancellationToken);
    }
    ```
1. Nous pouvons remplacer **ResumeAfterLocalAdminFormDialog** par la deuxième étape de notre cascade. Nous devons obtenir la valeur de retour à partir du contexte d’étape et non à partir d’une propriété d’instance.
    ```csharp
    private async Task<DialogTurnResult> SaveResultAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        // Get the result from the Formflow dialog when it ends.
        if (stepContext.Reason != DialogReason.CancelCalled)
        {
            var admin = stepContext.Result as LocalAdminPrompt;

            //TODO: Save to this information to the database.
        }

        return await stepContext.EndDialogAsync(null, cancellationToken);
    }
    ```
1. **BuildLocalAdminForm** reste en grande partie identique, à ceci près que Formflow ne met pas à jour la propriété d’instance.
    ```csharp
    // Nearly the same as before.
    private IForm<LocalAdminPrompt> BuildLocalAdminForm()
    {
        //here's an example of how validation can be used in form builder
        return new FormBuilder<LocalAdminPrompt>()
            .Field(nameof(LocalAdminPrompt.MachineName),
            validate: async (state, value) =>
            {
                ValidateResult result = new ValidateResult { IsValid = true, Value = value };
                //add validation here

                //this.admin.MachineName = (string)value;
                return result;
            })
            .Field(nameof(LocalAdminPrompt.AdminDuration),
            validate: async (state, value) =>
            {
                ValidateResult result = new ValidateResult { IsValid = true, Value = value };
                //add validation here

                //this.admin.AdminDuration = Convert.ToInt32((long)value) as int?;
                return result;
            })
            .Build();
    }
    ```

### <a name="update-the-reset-password-dialog"></a>Mettre à jour le dialogue de réinitialisation du mot de passe

Dans v3, ce dialogue accueille l’utilisateur, autorise l’utilisateur avec un code d’accès, échoue ou démarre le dialogue Formflow, puis réinitialise le mot de passe. Cela se traduit néanmoins bien en une cascade.

1. Mettez à jour les instructions `using`. Notez que ce dialogue inclut un dialogue Formflow v3. Dans v4, nous pouvons utiliser la bibliothèque Formflow de communauté.
    ```csharp
    using System;
    using System.Threading;
    using System.Threading.Tasks;
    using Bot.Builder.Community.Dialogs.FormFlow;
    using ContosoHelpdeskChatBot.Models;
    using Microsoft.Bot.Builder.Dialogs;
    ```
1. Définissez des constantes pour les ID que nous allons utiliser pour les dialogues. Dans la bibliothèque de communauté, l’ID de dialogue construit est toujours défini sur le nom de la classe produite par le dialogue.
    ```csharp
    // Set up our dialog and prompt IDs as constants.
    private const string MainDialog = "mainDialog";
    private static string ResetDialog { get; } = nameof(ResetPasswordPrompt);
    ```
1. Ajoutez un constructeur et initialisez le jeu de dialogues du composant. Le dialogue Formflow est créé de la même façon. Nous allons simplement l’ajouter au jeu de dialogues de notre composant dans le constructeur.
    ```csharp
    public ResetPasswordDialog(string dialogId) : base(dialogId)
    {
        InitialDialogId = MainDialog;

        AddDialog(new WaterfallDialog(MainDialog, new WaterfallStep[]
        {
            BeginFormflowAsync,
            ProcessRequestAsync,
        }));
        AddDialog(FormDialog.FromForm(BuildResetPasswordForm, FormOptions.PromptInStart));
    }
    ```
1. Nous pouvons remplacer **StartAsync** par la première étape de notre dialogue en cascade. Nous avons déjà créé le Formflow dans le constructeur. Sinon, nous conservons la même logique, en traduisant uniquement les appels v3 en leurs équivalents v4.
    ```csharp
    private async Task<DialogTurnResult> BeginFormflowAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        await stepContext.Context.SendActivityAsync("Alright I will help you create a temp password.");

        // Check the passcode and fail out or begin the Formflow dialog.
        if (sendPassCode(stepContext))
        {
            return await stepContext.BeginDialogAsync(ResetDialog, cancellationToken: cancellationToken);
        }
        else
        {
            //here we can simply fail the current dialog because we have root dialog handling all exceptions
            throw new Exception("Failed to send SMS. Make sure email & phone number has been added to database.");
        }
    }
    ```
1. **sendPassCode** est considéré principalement comme un exercice. Le code d’origine est commenté et la méthode retourne simplement la valeur true. En outre, nous pouvons là encore supprimer l’adresse e-mail, car elle n’a pas été utilisée dans le bot d’origine.
    ```csharp
    private bool sendPassCode(DialogContext context)
    {
        //bool result = false;

        //Recipient Id varies depending on channel
        //refer ChannelAccount class https://docs.botframework.com/en-us/csharp/builder/sdkreference/dd/def/class_microsoft_1_1_bot_1_1_connector_1_1_channel_account.html#a0b89cf01fdd73cbc00a524dce9e2ad1a
        //as well as Activity class https://docs.botframework.com/en-us/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html
        //int passcode = new Random().Next(1000, 9999);
        //Int64? smsNumber = 0;
        //string smsMessage = "Your Contoso Pass Code is ";
        //string countryDialPrefix = "+1";

        // TODO: save PassCode to database
        //using (var db = new ContosoHelpdeskContext())
        //{
        //    var reset = db.ResetPasswords.Where(r => r.EmailAddress == email).ToList();
        //    if (reset.Count >= 1)
        //    {
        //        reset.First().PassCode = passcode;
        //        smsNumber = reset.First().MobileNumber;
        //        result = true;
        //    }

        //    db.SaveChanges();
        //}

        // TODO: send passcode to user via SMS.
        //if (result)
        //{
        //    result = Helper.SendSms($"{countryDialPrefix}{smsNumber.ToString()}", $"{smsMessage} {passcode}");
        //}

        //return result;
        return true;
    }
    ```
1. **BuildResetPasswordForm** ne contient aucun changement.
1. Nous pouvons remplacer **ResumeAfterLocalAdminFormDialog** par la deuxième étape de notre cascade et nous allons obtenir la valeur de retour à partir du contexte d’étape. Nous avons supprimé l’adresse e-mail que le dialogue d’origine n’a pas utilisée et nous avons fourni un résultat factice au lieu d’interroger la base de données. Nous conservons la même logique, en traduisant uniquement les appels v3 en leurs équivalents v4.
    ```csharp
    private async Task<DialogTurnResult> ProcessRequestAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        // Get the result from the Formflow dialog when it ends.
        if (stepContext.Reason != DialogReason.CancelCalled)
        {
            var prompt = stepContext.Result as ResetPasswordPrompt;
            int? passcode;

            // TODO: Retrieve the passcode from the database.
            passcode = 1111;

            if (prompt.PassCode == passcode)
            {
                string temppwd = "TempPwd" + new Random().Next(0, 5000);
                await stepContext.Context.SendActivityAsync(
                    $"Your temp password is {temppwd}",
                    cancellationToken: cancellationToken);
            }
        }

        return await stepContext.EndDialogAsync(null, cancellationToken);
    }
    ```

### <a name="update-models-as-necessary"></a>Mettre à jour les modèles en fonction des besoins

Nous devons mettre à jour les instructions `using` dans certains des modèles qui font référence à la bibliothèque Formflow.

1. Dans `LocalAdminPrompt`, remplacez-les par ceci :
    ```csharp
    using Bot.Builder.Community.Dialogs.FormFlow;
    ```
1. Dans `ResetPasswordPrompt`, remplacez-les par ceci :
    ```csharp
    using Bot.Builder.Community.Dialogs.FormFlow;
    using System;
    ```

## <a name="update-webconfig"></a>Mettre à jour Web.config

Commentez les clés de configuration pour **MicrosoftAppId** et **MicrosoftAppPassword**. Cela vous permettra de déboguer le bot localement sans avoir à fournir ces valeurs à l’émulateur.

## <a name="run-and-test-your-bot-in-the-emulator"></a>Exécuter et tester votre bot dans l’émulateur

À ce stade, nous devrions être en mesure d’exécuter le bot localement dans IIS et de s’y attacher avec l’émulateur.

1. Exécutez le bot dans IIS.
1. Démarrez l’émulateur et connectez-vous au point de terminaison du bot (par exemple, **http://localhost:3978/api/messages**).
    - Si vous exécutez le bot pour la première fois, cliquez sur **Fichier > Nouveau bot** et suivez les instructions à l’écran. Sinon, cliquez sur **Fichier > Ouvrir un bot** pour ouvrir un bot existant.
    - Vérifiez bien vos paramètres de port dans la configuration. Par exemple, si le bot est ouvert dans votre navigateur sur `http://localhost:3979/`, définissez dans l’émulateur le point de terminaison du bot sur `http://localhost:3979/api/messages`.
1. Les quatre dialogues doivent tous fonctionner et vous pouvez définir des points d’arrêt dans les étapes en cascade pour vérifier le contexte et l’état des dialogues à ces points.

## <a name="additional-resources"></a>Ressources supplémentaires

Rubriques conceptuelles v4 :

- [Fonctionnement des bots](../bot-builder-basics.md)
- [Gestion de l’état](../bot-builder-concept-state.md)
- [Bibliothèque de boîtes de dialogues](../bot-builder-concept-dialog.md)

Rubriques de procédures v4 :

- [Envoyer et recevoir des SMS](../bot-builder-howto-send-messages.md)
- [Enregistrer les données d’utilisateur et de conversation](../bot-builder-howto-v4-state.md)
- [Implémenter des flux de conversation séquentiels](../bot-builder-dialog-manage-conversation-flow.md)
- [Collecter les entrées utilisateur avec une invite de boîte de dialogue](../bot-builder-prompts.md)

<!-- TODO:
- The conceptual piece
- The migration to a .NET Core project
-->
