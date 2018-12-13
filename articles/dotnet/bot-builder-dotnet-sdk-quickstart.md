---
title: Créer un bot avec le Kit SDK Bot Builder pour .NET | Microsoft Docs
description: Créez un bot avec le kit SDK Bot Builder pour .NET, puissant framework de construction de bot.
keywords: SDK Bot Builder, créer un bot, créer un bot, guide de démarrage rapide, .NET, prise en main, bien démarrer, bot C#
author: kamrani
ms.author: kamrani
manager: kamrani
ms.topic: get-started-article
ms.service: bot-service
ms.subservice: sdk
ms.date: 11/13/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 20a7dad4398874febfbd71024cd68763107f7bd8
ms.sourcegitcommit: 0b421ff71617f03faf55ea175fb91d1f9e348523
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/12/2018
ms.locfileid: "53286625"
---
# <a name="create-a-bot-with-the-bot-builder-sdk-for-net"></a>Créer un bot avec le Kit SDK Bot Builder pour .NET
[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Ce guide de démarrage rapide vous oriente tout au long de la création d’un bot à l’aide du modèle C#, puis lors du test avec Bot Framework Emulator.

## <a name="prerequisites"></a>Prérequis
- Visual Studio [2017](https://www.visualstudio.com/downloads)
- Modèle Bot Builder SDK v4 pour [C#](https://aka.ms/bot-vsix)
- Bot Framework [Emulator](https://aka.ms/Emulator-wiki-getting-started)
- Connaissances d’[ASP.Net Core](https://docs.microsoft.com/aspnet/core/) et de la programmation asynchrone en [C#](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/async/index)

## <a name="create-a-bot"></a>Créer un bot
Installez le modèle BotBuilderVSIX.vsix que vous avez téléchargé à la section des prérequis.

Dans Visual Studio, créez un nouveau projet de bot en utilisant le modèle Bot Builder Echo Bot V4.

![Projet Visual Studio](../media/azure-bot-quickstarts/bot-builder-dotnet-project.png)

> [!TIP] 
> Si nécessaire, modifiez le type de génération de projet sur ``.Net Core 2.1``. Le cas échéant, mettez à jour les [packages NuGet](https://docs.microsoft.com/en-us/nuget/quickstart/install-and-use-a-package-in-visual-studio).

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

> [!NOTE]
> Si vous voyez que le message ne peut pas être envoyé, vous devrez peut-être redémarrer votre ordinateur, car ngrok n’a pas encore obtenu les privilèges nécessaires sur votre système (à n’effectuer qu’une seule fois).

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Fonctionnement des bots](../v4sdk/bot-builder-basics.md) 
