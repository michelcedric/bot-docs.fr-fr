---
title: Créer un bot avec le kit SDK Bot Builder pour .NET | Microsoft Docs
description: Créez un bot avec le kit SDK Bot Builder pour .NET, puissant framework de construction de bot.
keywords: Kit SDK Bot Builder, créer un bot, créer un robot, guide de démarrage rapide, .NET, prise en main, bien démarrer
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: get-started-article
ms.prod: bot-framework
ms.date: 04/27/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 98c6103f0cd38bf6d3f477735dafcf01952304d5
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39300340"
---
# <a name="create-a-bot-with-the-bot-builder-sdk-v4-for-net"></a>Créer un bot avec le kit SDK Bot Builder v4 pour .NET
[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Ce guide de démarrage rapide vous pilote tout au long de la création d’un bot, en utilisant le modèle Bot Application et le kit SDK Bot Builder pour .NET, puis en testant votre bot avec l’émulateur Bot Framework. Ce guide s’appuie sur le [kit SDK Microsoft Bot Builder v4](https://github.com/Microsoft/botbuilder-dotnet).

## <a name="prerequisites"></a>Prérequis
- Visual Studio [2017](https://www.visualstudio.com/downloads)
- Modèle Bot Builder SDK v4 pour [C#](https://marketplace.visualstudio.com/items?itemName=BotBuilder.botbuilderv4)
- [Bot Framework Emulator](https://github.com/Microsoft/BotFramework-Emulator/releases)
- Connaissances d’[ASP.Net Core](https://docs.microsoft.com/aspnet/core/) et de la programmation asynchrone en [C#](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/async/index)

## <a name="create-a-bot"></a>Créer un bot
Installez le modèle BotBuilderVSIX.vsix que vous avez téléchargé à la section des prérequis. 

Dans Visual Studio, créez un projet de bot.

![Projet Visual Studio](../media/azure-bot-quickstarts/bot-builder-dotnet-project.png)

> [!TIP] 
> Si nécessaire, mettez à jour les [packages NuGet](https://docs.microsoft.com/en-us/nuget/quickstart/install-and-use-a-package-in-visual-studio).

## <a name="explore-code"></a>Explorer le code
Ouvrez un fichier Startup.cs pour examiner le code dans la méthode `ConfigureServices(IServiceCollection services)`. Le middleware (intergiciel) `CatchExceptionMiddleware` est ajouté au pipeline de messagerie. Il gère toutes les exceptions levées par d’autres middlewares ou par la méthode OnTurn. 

```cs
options.Middleware.Add(new CatchExceptionMiddleware<Exception>(async (context, exception) =>
{
    await context.TraceActivity("EchoBot Exception", exception);
    await context.SendActivity("Sorry, it looks like something went wrong!");
}));
```

Le middleware d’état de la conversation utilise le stockage en mémoire. Il lit et écrit l’EchoState dans le stockage.  Le nombre de tours dans la classe EchoState assure le suivi du nombre de messages envoyés au bot. Vous pouvez utiliser une technique similaire pour conserver l’état entre les tours.

```cs
 IStorage dataStore = new MemoryStorage();
 options.Middleware.Add(new ConversationState<EchoState>(dataStore));
```

La méthode `Configure(IApplicationBuilder app, IHostingEnvironment env)` appelle `.UseBotFramework` pour router les activités entrantes vers l’adaptateur du bot. 

La méthode `OnTurn(ITurnContext context)` dans le fichier EchoBot.cs est utilisée pour vérifier le type d’activité entrante et envoyer une réponse à l’utilisateur. 

```cs
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
```
## <a name="start-your-bot"></a>Démarrer votre bot

- Votre page `default.html` s’affiche dans un navigateur.
- Notez le numéro de port localhost de la page. Vous avez besoin de ces informations pour interagir avec votre bot.

### <a name="start-the-emulator-and-connect-your-bot"></a>Démarrer l’émulateur et connecter votre bot

À ce stade, votre bot s’exécute localement.
Démarrez à présent l’émulateur, puis connectez-vous à votre bot dans l’émulateur :

1. Cliquez sur le lien **create a new bot configuration** (créer une configuration de bot) sous l’onglet « Welcome » (Bienvenue) de l’émulateur. 

2. Entrez un **Bot name** (Nom de bot) et le chemin du répertoire de votre code bot. Le fichier de configuration du bot est enregistré dans ce chemin.

3. Tapez `http://localhost:port-number/api/messages` dans le champ **Endpoint URL** (URL du point de terminaison), où *port-number* correspond au numéro de port affiché dans le navigateur dans lequel votre application est en cours d’exécution.

4. Cliquez sur **Connect** (Se connecter) pour vous connecter à votre bot. Inutile de spécifier les valeurs de **Microsoft App ID** (ID de l’application Microsoft) et **Microsoft App Password** (Mot de passe de l’application Microsoft). Vous pouvez laisser ces champs vides pour l’instant. Vous obtiendrez ces informations ultérieurement, lors de l’inscription du bot.

## <a name="interact-with-your-bot"></a>Interagir avec votre bot

Envoyez « Hi » (Bonjour) à votre bot, il répond au message par « Turn 1: You sent Hi » (Tour 1 : Vous avez envoyé Bonjour).

## <a name="next-steps"></a>Étapes suivantes

À présent, [déployez votre bot sur Azure](../bot-builder-howto-deploy-azure.md) ou passez aux concepts qui décrivent un bot et son fonctionnement.

> [!div class="nextstepaction"]
> [Concepts Bot de base](../v4sdk/bot-builder-basics.md)
