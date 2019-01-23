---
title: Intercepter des messages | Microsoft Docs
description: Découvrez comment intercepter des messages entre un utilisateur et un bot à l’aide du kit SDK Bot Framework pour .NET.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 54ab447fac81ba17f1fa277347cf7a5aaaab8850
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/11/2019
ms.locfileid: "54223774"
---
# <a name="intercept-messages"></a>Intercepter des messages

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-middleware.md)
> - [Node.JS](../nodejs/bot-builder-nodejs-intercept-messages.md)

[!INCLUDE [Introducton to message logging](../includes/snippet-message-logging-intro.md)]

## <a name="intercept-and-log-messages"></a>Intercepter et journaliser des messages

L’exemple de code suivant montre comment intercepter les messages échangés entre un utilisateur et un bot en utilisant le concept d’**intergiciel (middleware)** dans le kit SDK Bot Framework pour .NET. 

Commencez par créer une classe `DebugActivityLogger` et par définir une méthode `LogAsync` pour spécifier la mesure à prendre pour chaque message intercepté. Cet exemple imprime seulement des informations sur chaque message.

```cs
public class DebugActivityLogger : IActivityLogger
{
    public async Task LogAsync(IActivity activity)
    {
        Debug.WriteLine($"From:{activity.From.Id} - To:{activity.Recipient.Id} - Message:{activity.AsMessageActivity()?.Text}");
    }
}
```

Ajoutez ensuite le code suivant à `Global.asax.cs`.  Chaque message échangé entre l’utilisateur et le bot (dans les deux sens) déclenche alors la méthode `LogAsync` dans la classe `DebugActivityLogger`. 

```cs
    public class WebApiApplication : System.Web.HttpApplication
    {
        protected void Application_Start()
        {
            var builder = new ContainerBuilder();
            builder.RegisterType<DebugActivityLogger>().AsImplementedInterfaces().InstancePerDependency();
            builder.Update(Conversation.Container);

            GlobalConfiguration.Configure(WebApiConfig.Register);
        }
    }
```

Bien que, dans cet exemple, seules des informations sur chaque message, vous pouvez mettre à jour la méthode `LogAsync` pour définir les mesures à prendre pour chaque message. 

## <a name="sample-code"></a>Exemple de code 

Pour obtenir un exemple complet qui montre comment intercepter et journaliser les messages à l’aide du kit SDK Bot Framework pour .NET, consultez l’<a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-Middleware" target="_blank">exemple Middleware</a> dans GitHub. 

## <a name="additional-resources"></a>Ressources supplémentaires

- <a href="/dotnet/api/?view=botbuilder-3.11.0" target="_blank">Informations de référence sur le kit SDK Bot Framework pour .NET</a>
- <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-Middleware" target="_blank">Exemple Middleware (GitHub)</a>
