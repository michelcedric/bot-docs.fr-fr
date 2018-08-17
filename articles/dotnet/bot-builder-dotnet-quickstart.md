---
title: Créer un bot avec le Kit SDK Bot Builder pour .NET | Microsoft Docs
description: Créez un bot avec le Kit SDK Bot Builder pour .NET, une puissante infrastructure de construction de bot.
keywords: SDK Bot Builder, créer un bot, démarrage rapide, prise en main
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: get-started-article
ms.prod: bot-framework
ms.date: 04/27/2018
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: fcb21fa38750c09f110a3c71f763a941c4437979
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39300301"
---
# <a name="create-a-bot-with-the-bot-builder-sdk-for-net"></a>Créer un bot avec le Kit SDK Bot Builder pour .NET
> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-quickstart.md)
> - [Node.JS](../nodejs/bot-builder-nodejs-quickstart.md)
> - [Service de bot](../bot-service-quickstart.md)
> - [REST](../rest-api/bot-framework-rest-connector-quickstart.md)

Le <a href="https://github.com/Microsoft/BotBuilder" target="_blank">Kit SDK Bot Builder pour .NET</a> est une infrastructure facile à utiliser pour développer des bots à l’aide de Visual Studio et Windows. Le SDK s’appuie sur le langage C# pour permettre aux développeurs .NET de créer de puissants bots dans un environnement familier.


Ce tutoriel vous guide tout au long de la création d’un bot en utilisant le modèle Application de bot et le Kit SDK pour .NET, puis en le testant avec l’émulateur Bot Framework.

## <a name="prerequisites"></a>Prérequis
1. Visual Studio [2017](https://www.visualstudio.com/).

2. Dans Visual Studio, mettez à jour toutes les [extensions](https://docs.microsoft.com/en-us/visualstudio/extensibility/how-to-update-a-visual-studio-extension) avec leurs versions les plus récentes.

3. Modèle de bot pour [C#](https://marketplace.visualstudio.com/items?itemName=BotBuilder.BotBuilderV3).

> [!TIP]
> Le répertoire de modèles de projet Visual Studio 2017 est généralement situé dans `%USERPROFILE%\Documents\Visual Studio 2017\Templates\ProjectTemplates\Visual C#\`, et le répertoire des modèles d’éléments dans `%USERPROFILE%\Documents\Visual Studio 2017\Templates\ItemTemplates\Visual C#\`

## <a name="create-your-bot"></a>Créer votre bot

Ouvrez Visual Studio et créez un projet C#. Choisissez le modèle **Simple Echo Bot Application** pour votre nouveau projet.

![Créer un projet Visual Studio](../media/connector-getstarted-create-project.png)

> [!NOTE]
> Visual Studio peut vous demander de télécharger et d’installer [IIS Express](https://www.microsoft.com/en-us/download/details.aspx?id=48264). 

Grâce au modèle Application de bot, votre projet contient tout le code nécessaire pour créer le bot dans ce tutoriel. En fait, vous n’avez pas besoin d’écrire du code supplémentaire. Mais avant de passer au test de votre bot, examinons rapidement le code fourni par le modèle Application de bot.

> [!TIP] 
> Si nécessaire, mettez à jour les [packages NuGet](https://docs.microsoft.com/en-us/nuget/quickstart/install-and-use-a-package-in-visual-studio).

## <a name="explore-the-code"></a>Explorer le code

Tout d’abord, la méthode `Post` au sein de **Controllers\MessagesController.cs** reçoit le message de l’utilisateur et appelle le dialogue racine.

```csharp
public async Task<HttpResponseMessage> Post([FromBody]Activity activity)
{
    if (activity.GetActivityType() == ActivityTypes.Message)
    {
        await Conversation.SendAsync(activity, () => new Dialogs.RootDialog());
    }
    else
    {
        HandleSystemMessage(activity);
    }
    var response = Request.CreateResponse(HttpStatusCode.OK);
    return response;
}

```

Le dialogue racine traite le message et génère une réponse. La méthode `MessageReceivedAsync` au sein de **Dialogs\RootDialog.cs** envoie une réponse qui reprend le message de l’utilisateur, précédé du 'You sent' (Vous avez envoyé) et se terminant par le texte 'which was *##* characters' (qui contenait X caractères), où *##* représente le nombre de caractères du message de l’utilisateur.

```csharp
public class RootDialog : IDialog<object>
{
    public Task StartAsync(IDialogContext context)
    {
        context.Wait(MessageReceivedAsync);

        return Task.CompletedTask;
    }

    private async Task MessageReceivedAsync(IDialogContext context, IAwaitable<object> result)
    {
        var activity = await result as Activity;

        // Calculate something for us to return
        int length = (activity.Text ?? string.Empty).Length;

        // Return our reply to the user
        await context.PostAsync($"You sent {activity.Text} which was {length} characters");

        context.Wait(MessageReceivedAsync);
    }
}
```

## <a name="test-your-bot"></a>Tester votre bot

[!INCLUDE [Get started test your bot](../includes/snippet-getstarted-test-bot.md)]

### <a name="start-your-bot"></a>Démarrer votre bot

Après avoir installé l’émulateur, démarrez votre bot dans Visual Studio en utilisant un navigateur comme hôte d’application.
Cette capture d’écran de Visual Studio indique que le bot se lancera dans Microsoft Edge lorsque l’utilisateur clique sur le bouton d’exécution.

![Nouveau projet Visual Studio](../media/connector-getstarted-start-bot-locally.png)

Lorsque vous cliquez sur le bouton d’exécution, Visual Studio générera l’application, la déploiera sur localhost et lancera le navigateur web pour afficher la page **default.htm** de l’application.
Par exemple, voici la page **default.htm** de l’application dans Microsoft Edge :

![Bot Studio Visual exécuté sur localhost](../media/connector-getstarted-bot-running-localhost.png)

> [!NOTE]
> Vous pouvez modifier le fichier **default.htm** au sein de votre projet pour spécifier le nom et la description de votre application de bot.

### <a name="start-the-emulator-and-connect-your-bot"></a>Démarrer l’émulateur et se connecter votre bot

À ce stade, votre bot s’exécute localement.
Démarrez à présent l’émulateur, puis connectez-vous à votre bot dans l’émulateur :

1. Créez une nouvelle configuration de bot. Tapez `http://localhost:port-number/api/messages` dans la barre d’adresses, où *port-number* correspond au numéro de port affiché dans le navigateur dans lequel votre application est en cours d’exécution.

2. Cliquez sur **Enregistrer et se connecter**. Inutile de spécifier les valeurs **Microsoft App ID** et **Microsoft App Password**. Vous pouvez laisser ces champs vides pour l’instant. Vous obtiendrez ces informations ultérieurement, lors de l’[inscription du bot](~/bot-service-quickstart-registration.md).

### <a name="test-your-bot"></a>Tester votre bot

Maintenant que votre bot s’exécute localement et qu’il est connecté à l’émulateur, testez votre bot en entrant quelques messages dans l’émulateur.
Vous devriez constater que le bot répond à chaque message que vous envoyez en renvoyant votre message, précédé du texte 'You sent' (Vous avez envoyé) et se terminant par le texte 'which was *##* characters' (qui contenait X caractères), où *##* correspond au nombre total de caractères du message que vous avez envoyé.


> [!TIP]
> Dans l’émulateur, cliquez sur n’importe quel bulle dans votre conversation. Les détails du message s’afficheront dans le volet Détails, au format JSON.

Vous avez créé avec succès un bot à l’aide du modèle Application de bot du Kit SDK Bot Builder pour .NET.

## <a name="next-steps"></a>Étapes suivantes

Dans ce démarrage rapide, vous avez créé un simple bot à l’aide du modèle Application de bot et le Kit SDK Bot Builder pour .NET, puis vérifié les fonctionnalités du bot à l’aide de l’émulateur Bot Framework.

Examinons à présent les concepts clés du Kit SDK Bot Builder pour .NET.

> [!div class="nextstepaction"]
> [Concepts clés du Kit SDK Bot Builder pour .NET](bot-builder-dotnet-concepts.md)
