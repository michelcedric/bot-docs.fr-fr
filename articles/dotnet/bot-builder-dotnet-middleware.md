---
title: Intercepter des messages | Microsoft Docs
description: Découvrez comment intercepter des messages entre un utilisateur et un bot à l’aide du kit de développement logiciel (SDK) Bot Builder pour .NET.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 21607dae5c2a8d08ed4b7bf1b6e6983cd9bf1196
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2018
ms.locfileid: "49999726"
---
# <a name="intercept-messages"></a>Intercepter des messages

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-middleware.md)
> - [Node.JS](../nodejs/bot-builder-nodejs-intercept-messages.md)

[!INCLUDE [Introducton to message logging](../includes/snippet-message-logging-intro.md)]

## <a name="intercept-and-log-messages"></a>Intercepter et journaliser des messages

L’exemple de code suivant montre comment intercepter des messages échangés entre un utilisateur et bot en utilisant le concept d’**intergiciel (middleware)** dans le Kit de développement logiciel (SDK) Bot Builder pour .NET. 

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

Pour obtenir un exemple complet montrant comment intercepter et journaliser les messages à l’aide du Kit de développement logiciel (SDK) Bot Builder pour .NET, voir l’<a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-Middleware" target="_blank">Exemple d’intergiciel</a> dans GitHub. 

## <a name="additional-resources"></a>Ressources supplémentaires

- <a href="/dotnet/api/?view=botbuilder-3.11.0" target="_blank">Documentation de référence concernant le Kit de développement logiciel (SDK) Bot Builder pour .NET</a>
- <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-Middleware" target="_blank">Exemple d’intergiciel (GitHub)</a>
