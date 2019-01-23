---
title: Implémenter des gestionnaires de messages globaux | Microsoft Docs
description: Découvrez comment permettre à votre bot d’écouter et de gérer les entrées utilisateur contenant certains mots clés à l’aide du kit SDK Bot Framework pour .NET.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 86964bc39a95a23f397af649cfac6e2784dd588a
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/11/2019
ms.locfileid: "54224364"
---
# <a name="implement-global-message-handlers"></a>Implémenter des gestionnaires de messages globaux

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

[!INCLUDE [Introduction to global message handlers](../includes/snippet-global-handlers-intro.md)]

## <a name="listen-for-keywords-in-user-input"></a>Écouter des mots clés dans les entrées utilisateur

La procédure pas à pas suivante montre comment implémenter des gestionnaires de messages globaux à l’aide du kit SDK Bot Framework pour .NET.

Tout d’abord, `Global.asax.cs` inscrit `GlobalMessageHandlersBotModule`, qui est implémentée comme indiqué ici. Dans cet exemple, le module enregistre deux dialogues scorables : l’un pour gérer une demande de modification de paramètres (`SettingsScorable`), et un autre pour gérer une demande d’annulation (`CancelScoreable`).

```cs
public class GlobalMessageHandlersBotModule : Module
{
    protected override void Load(ContainerBuilder builder)
    {
        base.Load(builder);

        builder
            .Register(c => new SettingsScorable(c.Resolve<IDialogTask>()))
            .As<IScorable<IActivity, double>>()
            .InstancePerLifetimeScope();

        builder
            .Register(c => new CancelScorable(c.Resolve<IDialogTask>()))
            .As<IScorable<IActivity, double>>()
            .InstancePerLifetimeScope();
    }
}
```

Le `CancelScorable` contient une méthode `PrepareAsync` qui définit le déclencheur : si le texte du message est « annuler », ce dialogue scorable est déclenché.

```cs
protected override async Task<string> PrepareAsync(IActivity activity, CancellationToken token)
{
    var message = activity as IMessageActivity;
    if (message != null && !string.IsNullOrWhiteSpace(message.Text))
    {
        if (message.Text.Equals("cancel", StringComparison.InvariantCultureIgnoreCase))
        {
            return message.Text;
        }
    }
    return null;
}
```

Quand une demande « annuler » est reçue, la méthode `PostAsync` dans `CancelScoreable` réinitialise la pile de dialogues. 

```cs
protected override async Task PostAsync(IActivity item, string state, CancellationToken token)
{
    this.task.Reset();
}
```

Lors de la réception d’une demande « modifier les paramètres », la méthode `PostAsync` dans `SettingsScorable` appelle le dialogue `SettingsDialog` (en transmettant la demande à ce dialogue), ce qui a pour effet d’ajouter le dialogue `SettingsDialog` en haut de la pile de dialogues, et de lui confier le contrôle de la conversation.

```cs
protected override async Task PostAsync(IActivity item, string state, CancellationToken token)
{
    var message = item as IMessageActivity;
    if (message != null)
    {
        var settingsDialog = new SettingsDialog();
        var interruption = settingsDialog.Void<object, IMessageActivity>();
        this.task.Call(interruption, null);
        await this.task.PollAsync(token);
    }
}
```

## <a name="sample-code"></a>Exemple de code

Pour obtenir un exemple complet qui montre comment implémenter des gestionnaires de messages globaux à l’aide du kit SDK Bot Framework pour .NET, consultez l’<a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-GlobalMessageHandlers" target="_blank">exemple Global Message Handlers</a> dans GitHub.

## <a name="additional-resources"></a>Ressources supplémentaires

- [Concevoir et contrôler un flux de conversation](../bot-service-design-conversation-flow.md)
- <a href="/dotnet/api/?view=botbuilder-3.12.2.4" target="_blank">Informations de référence sur le kit SDK Bot Framework pour .NET</a>
- <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-GlobalMessageHandlers" target="_blank">Exemples de gestionnaires de messages globaux (GitHub)</a>
