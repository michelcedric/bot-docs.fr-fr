---
title: Créer un bot avec le Kit SDK Bot Builder pour .NET | Microsoft Docs
description: Créez un bot avec le kit SDK Bot Builder pour .NET, puissant framework de construction de bot.
keywords: SDK Bot Builder, créer un bot, créer un bot, guide de démarrage rapide, .NET, prise en main, bien démarrer, bot C#
author: kamrani
ms.author: kamrani
manager: kamrani
ms.topic: get-started-article
ms.prod: bot-framework
ms.date: 09/23/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 5f3a02783242697fccf267bef2d56ed453880c67
ms.sourcegitcommit: 3bf3dbb1a440b3d83e58499c6a2ac116fe04b2f6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/23/2018
ms.locfileid: "46707975"
---
# <a name="create-a-bot-with-the-bot-builder-sdk-for-net"></a>Créer un bot avec le Kit SDK Bot Builder pour .NET
[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Ce guide de démarrage rapide vous oriente tout au long de la création d’un bot à l’aide du modèle C#, puis lors du test avec Bot Framework Emulator. 

## <a name="prerequisites"></a>Prérequis
- Visual Studio [2017](https://www.visualstudio.com/downloads)
- Modèle Bot Builder SDK v4 pour [C#](https://botbuilder.myget.org/feed/aitemplates/package/vsix/BotBuilderV4.fbe0fc50-a6f1-4500-82a2-189314b7bea2)
- Bot Framework [Emulator](https://github.com/Microsoft/BotFramework-Emulator/releases)
- Connaissances d’[ASP.Net Core](https://docs.microsoft.com/aspnet/core/) et de la programmation asynchrone en [C#](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/async/index)

## <a name="create-a-bot"></a>Créer un bot
Installez le modèle BotBuilderVSIX.vsix que vous avez téléchargé à la section des prérequis. 

Dans Visual Studio, créez un projet de bot.

![Projet Visual Studio](../media/azure-bot-quickstarts/bot-builder-dotnet-project.png)

> [!TIP] 
> Si nécessaire, mettez à jour les [packages NuGet](https://docs.microsoft.com/en-us/nuget/quickstart/install-and-use-a-package-in-visual-studio).

Grâce au modèle, votre projet contient tout le code nécessaire pour créer le bot dans ce guide de démarrage rapide. En fait, vous n’avez pas besoin d’écrire du code supplémentaire.

## <a name="start-your-bot-in-visual-studio"></a>Démarrer votre bot dans Visual Studio

Lorsque vous cliquez sur le bouton d’exécution, Visual Studio génère l’application, la déploie sur localhost et lance le navigateur web pour afficher la page `default.htm` de l’application. À ce stade, votre bot s’exécute localement.

## <a name="start-the-emulator-and-connect-your-bot"></a>Démarrer l’émulateur et connecter votre robot

Démarrez à présent l’émulateur, puis connectez-vous à votre bot dans l’émulateur :

1. Cliquez sur le lien **Ouvrir le bot** de l’onglet de bienvenue de l’émulateur. 
2. Sélectionnez le fichier .bot situé dans le répertoire où vous avez créé le projet Visual Studio.

## <a name="interact-with-your-bot"></a>Interagir avec votre bot

Envoyez un message à votre bot, et le bot vous enverra un message à son tour.
![Émulateur en cours d’exécution](../media/emulator-v4/emulator-running.png)

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Fonctionnement des bots](../v4sdk/bot-builder-basics.md) 
