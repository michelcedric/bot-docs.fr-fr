---
title: Gestionnaires de messages globaux à l’aide de dialogues scorables
description: Créez des dialogues plus flexibles à l’aide de dialogues scorables dans le Kit de développement logiciel (SDK) Bot Builder pour .NET.
author: matthewshim-ms
ms.author: v-shimma
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: a6a69bd1001c51b44ad42824a0da6a079c5ac5a0
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2018
ms.locfileid: "49999516"
---
# <a name="global-message-handlers-using-scorables"></a>Gestionnaires de messages globaux à l’aide de dialogues scorables

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

Les utilisateurs essaient d’accéder à certaines fonctionnalités au sein d’un bot en utilisant des mots tels que « help » (aide), « cancel » (annuler) ou « start over » (recommencer) au milieu d’une conversation alors que le bot attend une réponse différente. Vous pouvez concevoir votre bot pour qu’il traite ces requêtes de manière appropriée à l’aide de dialogues scorables.

Les dialogues scorables surveillent tous les messages entrants et déterminent si un message est actionnable d’une quelconque manière. Chaque dialogue scorable attribue aux messages qui sont scorables un score compris entre [0 et 1]. Le dialogue scorable qui détermine le score le plus élevé est ajouté au sommet de la pile de dialogues, puis transmet la réponse à l’utilisateur. Une fois que le dialogue scorable s’est exécuté, la conversation reprend à l’endroit où elle s’était interrompue.

Les dialogues scorables contribuent à créer des conversations plus flexibles en permettant aux utilisateurs d’interrompre le flux de conversation normal propre aux dialogues standard.

## <a name="create-a-scorable-dialog"></a>Créer un dialogue scorable

Commencez par définir un nouveau [dialogue](bot-builder-dotnet-dialogs.md). Le code ci-après utilise un dialogue dérivé de l’interface `IDialog`.

```cs
public class SampleDialog : IDialog<object>
{
    public async Task StartAsync(IDialogContext context)
    {
        await context.PostAsync("This is a Sample Dialog which is Scorable. Reply with anything to return to the prior prior dialog.");

        context.Wait(this.MessageReceived);
    }

    private async Task MessageReceived(IDialogContext context, IAwaitable<IMessageActivity> result)
    {
        var message = await result;

        if ((message.Text != null) && (message.Text.Trim().Length > 0))
        {
            context.Done<object>(null);
        }
        else
        {
            context.Fail(new Exception("Message was not a string or was an empty string."));
        }
    }
}
```
Pour rendre ce dialogue scorable, créez une classe qui hérite de la classe abstraite `ScorableBase`. Le code ci-après présente une classe `SampleScorable`.

```cs
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Bot.Builder.Dialogs.Internals;
using Microsoft.Bot.Builder.Internals.Fibers;
using Microsoft.Bot.Builder.Scorables.Internals;

public class SampleScorable : ScorableBase<IActivity, string, double>
{
    private readonly IDialogTask task;

    public SampleScorable(IDialogTask task)
    {
        SetField.NotNull(out this.task, nameof(task), task);
    }
}
```
La classe abstraite `ScorableBase` hérite de l’interface `IScorable`. Vous devez implémenter les méthodes `IScorable` ci-après dans votre classe :

- `PrepareAsync` est la première méthode appelée dans l’instance scorable. Elle accepte l’activité des messages entrants, puis analyse et définit l’état du dialogue, qui est alors transmis à toutes les autres méthodes de l’interface `IScorable`.

```cs
protected override async Task<string> PrepareAsync(IActivity item, CancellationToken token)
{
        // TODO: insert your code here
}
```

- La méthode `HasScore` vérifie la propriété « state » pour déterminer si le dialogue scorable doit fournir un score pour le message. Si elle renvoie la valeur false, le dialogue scorable ignore le message.

```cs
protected override bool HasScore(IActivity item, string state)
{
        // TODO: insert your code here
}
```

- `GetScore` ne se déclenche que si `HasScore` renvoie la valeur true. Vous approvisionnerez la logique de cette méthode afin de déterminer le score d’un message compris entre 0 et 1.

```cs
protected override double GetScore(IActivity item, string state)
{
        // TODO: insert your code here
}
```
- Dans la méthode `PostAsync`, définissez les actions principales à exécuter pour la classe scorable. Tous les dialogues scorables surveilleront les messages entrants et attribueront des scores aux messages valides en fonction de la méthode GetScore des dialogues scorables. La classe scorable qui détermine le score le plus élevé (entre 0 et 1) déclenchera alors la méthode `PostAsync` de ce dialogue scorable.

```cs
protected override Task PostAsync(IActivity item, string state, CancellationToken token)
{
        //TODO: insert your code here
}
```

- La méthode `DoneAsync` est appelée à l’issue du processus de scoring. Utilisez cette méthode pour supprimer toutes les ressources incluses dans l’étendue.

```cs
protected override Task DoneAsync(IActivity item, string state, CancellationToken token)
{
        //TODO: insert your code here
}
```

## <a name="create-a-module-to-register-the-iscorable-service"></a>Créer un module pour inscrire le service IScorable

Ensuite, définissez un élément `Module` qui inscrira la classe `SampleScorable` en tant que composant. Cette opération approvisionne le service `IScorable`.

```cs
public class GlobalMessageHandlersBotModule : Module
{
    protected override void Load(ContainerBuilder builder)
    {
        base.Load(builder);

        builder
            .Register(c => new SampleScorable(c.Resolve<IDialogTask>()))
            .As<IScorable<IActivity, double>>()
            .InstancePerLifetimeScope();
    }
}
```
## <a name="register-the-module"></a>Inscrire le module  

La dernière étape du processus consiste à appliquer la classe `SampleScorable` au conteneur de conversation du bot. Cette opération inscrit le service scorable dans le pipeline de gestion des messages de Bot Framework. Le code ci-après indique comment mettre à jour l’élément `Conversation.Container` dans le cadre de l’initialisation de l’application bot dans le fichier **Global.asax.cs** :

```cs
public class WebApiApplication : System.Web.HttpApplication
{
    protected void Application_Start()
    {
        this.RegisterBotModules();
        GlobalConfiguration.Configure(WebApiConfig.Register);
    }

    private void RegisterBotModules()
    {
        var builder = new ContainerBuilder();
        builder.RegisterModule(new ReflectionSurrogateModule());

        //Register the module within the Conversation container
        builder.RegisterModule<GlobalMessageHandlersBotModule>();

        builder.Update(Conversation.Container);
    }
}
```

## <a name="additional-resources"></a>Ressources supplémentaires
* [Exemple de gestionnaires de messages globaux](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-GlobalMessageHandlers)
* [Exemple de bot scorable simple](https://github.com/Microsoft/BotFramework-Samples/tree/master/blog-samples/CSharp/ScorableBotSample)
* [Vue d’ensemble des dialogues](bot-builder-dotnet-dialogs.md)
* [Autofac](https://autofac.org/)
