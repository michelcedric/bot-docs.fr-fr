---
title: Déboguer un bot | Microsoft Docs
description: Découvrez comment déboguer un bot créé à l’aide de Bot Service.
author: v-ducvo
ms.author: v-ducvo
keywords: Kit SDK Bot Framework, déboguer un bot, tester un bot, émulateur de bot, émulateur
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 2/26/2019
ms.openlocfilehash: 1e806358ffdba90848f0c8124c8315fd4e2dec76
ms.sourcegitcommit: cf3786c6e092adec5409d852849927dc1428e8a2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 03/01/2019
ms.locfileid: "57224837"
---
# <a name="debug-a-bot"></a>Déboguer un bot

Cet article explique comment déboguer votre bot à l’aide d’un environnement de développement intégré (IDE) tels que Visual Studio ou Visual Studio Code et l’émulateur Bot Framework. Même si vous pouvez utiliser ces méthodes pour déboguer des bots localement, cet article utilise un bot [C# ](~/dotnet/bot-builder-dotnet-sdk-quickstart.md) et [JS](~/javascript/bot-builder-javascript-quickstart.md) créé dans le démarrage rapide.

## <a name="prerequisites"></a>Prérequis 
- Téléchargez et installez l’[émulateur Bot Framework](https://aka.ms/Emulator-wiki-getting-started).
- Téléchargez et installez [Visual Studio Code](https://code.visualstudio.com) ou [Visual Studio](https://www.visualstudio.com/downloads) (Community Edition ou version ultérieure).

### <a name="debug-a-javascript-bot-using-command-line-and-emulator"></a>Déboguer un bot JavaScript à l’aide de la ligne de commande et de l’émulateur

Pour exécuter un bot JavaScript en utilisant la ligne de commande et en le testant avec l’émulateur, procédez comme suit :
1. Depuis la ligne de commande, remplacez le répertoire par défaut par le répertoire de votre projet de bot.
1. Démarrez le bot en exécutant la commande **node app.js**.
1. Démarrez l’émulateur et connectez-vous au point de terminaison du bot (par exemple : **http://localhost:3978/api/messages**). Si vous exécutez le bot pour la première fois, cliquez sur **Fichier > Nouveau bot** et suivez les instructions à l’écran. Sinon, cliquez sur **Fichier > Ouvrir un bot** pour ouvrir un bot existant. Dans la mesure où ce bot est exécuté localement sur votre ordinateur, vous pouvez laisser vides les champs **ID d’application MSA** et **Mot de passe d’application MSA**. Pour plus d’informations, consultez [Déboguer avec l’émulateur](bot-service-debug-emulator.md).
1. À partir de l’émulateur, envoyez un message à votre bot (par exemple : le message « Salut »). 
1. Utilisez les panneaux **Inspector** et **Log** à droite de la fenêtre de l’émulateur pour déboguer votre bot. Par exemple, cliquer sur une bulle de messages (par exemple, la bulle du message « Hi » dans la capture d’écran ci-dessous) affiche les détails de ce message dans le panneau **Inspector**. Vous pouvez l’utiliser pour afficher les demandes et réponses à mesure que des messages sont échangés entre l’émulateur et le bot. Vous pouvez également cliquer sur le texte lié dans le panneau **Log** pour afficher les détails dans le panneau **Inspecteur**.


   ![Panneau Inspector sur l’émulateur](~/media/bot-service-debug-bot/emulator_inspector.png)

### <a name="debug-a-javascript-bot-using-breakpoints-in-visual-studio-code"></a>Déboguer un bot JavaScript à l’aide de points d’arrêt dans Visual Studio Code

Dans Visual Studio Code, vous pouvez définir des points d’arrêt et exécuter le bot en mode débogage pour parcourir votre code. Pour définir des points d’arrêt dans VS Code, procédez comme suit :

1. Lancez VS Code et ouvrez le dossier de votre projet de bot.
2. Dans la barre de menus, cliquez sur **Déboguer**, puis sur **Démarrer le débogage**. Si vous êtes invité à sélectionner un moteur d’exécution pour exécuter votre code, sélectionnez **Node.js**. À ce stade, le bot s’exécute localement. 
<!--
   > [!NOTE]
   > If you get the "Value cannot be null" error, check to make sure your **Table Storage** setting is valid.
   > The **EchoBot** is default to using **Table Storage**. To use Table Storage in your bot, you need the table *name* and *key*. If you do not have a Table Storage instance ready, you can create one or for testing purposes, you can comment out the code that uses **TableBotDataStore** and uncomment the line of code that uses **InMemoryDataStore**. The **InMemoryDataStore** is intended for testing and prototyping only.
-->
3. Définissez le point d’arrêt en fonction des besoins. Dans VS Code, vous pouvez définir des points d’arrêt en plaçant votre souris sur la colonne à gauche des numéros de ligne. Un petit point rouge s’affiche. Si vous cliquez sur ce point, le point d’arrêt est défini. Si vous cliquez à nouveau sur le point, le point d’arrêt est supprimé.

   ![Définir un point d’arrêt dans VS Code](~/media/bot-service-debug-bot/breakpoint-set.png)

4. Démarrez l’émulateur Bot Framework et connectez-vous à votre bot, comme décrit dans la section ci-dessus. 
5. À partir de l’émulateur, envoyez un message à votre bot (par exemple : le message « Salut »). L’exécution s’arrêtera à la ligne où vous placez le point d’arrêt.

   ![Déboguer dans VS Code](~/media/bot-service-debug-bot/breakpoint-caught.png)

### <a name="debug-a-c-bot-using-breakpoints-in-visual-studio"></a>Déboguer un bot C# à l’aide de points d’arrêt dans Visual Studio

Dans Visual Studio (VS), vous pouvez définir des points d’arrêt et exécuter le bot en mode débogage pour parcourir votre code. Pour définir des points d’arrêt dans VS, procédez comme suit :

1. Accédez au dossier de votre bot et ouvrez le fichier **.sln**. Cela permet d’ouvrir la solution dans Visual Studio.
2. Dans la barre de menus, cliquez sur **Générer** puis sur **Générer la solution**.
3. Dans l’**Explorateur de solutions**, cliquez sur **EchoWithCounterBot.cs**. Ce fichier définit le point d’arrêt logic.Set de votre bot principal comme nécessaire. Dans VS, vous pouvez définir des points d’arrêt en plaçant votre souris sur la colonne à gauche des numéros de ligne. Un petit point rouge s’affiche. Si vous cliquez sur ce point, le point d’arrêt est défini. Si vous cliquez à nouveau sur le point, le point d’arrêt est supprimé.
5. Dans la barre de menus, cliquez sur **Déboguer**, puis sur **Démarrer le débogage**. À ce stade, le bot s’exécute localement. 

<!--
   > [!NOTE]
   > If you get the "Value cannot be null" error, check to make sure your **Table Storage** setting is valid.
   > The **EchoBot** is default to using **Table Storage**. To use Table Storage in your bot, you need the table *name* and *key*. If you do not have a Table Storage instance ready, you can create one or for testing purposes, you can comment out the code that uses **TableBotDataStore** and uncomment the line of code that uses **InMemoryDataStore**. The **InMemoryDataStore** is intended for testing and prototyping only.
-->

   ![Définir le point d’arrêt dans Visual Studio](~/media/bot-service-debug-bot/breakpoint-set-vs.png)

7. Démarrez l’émulateur Bot Framework et connectez-vous à votre bot, comme décrit dans la section ci-dessus. 
8. À partir de l’émulateur, envoyez un message à votre bot (par exemple : le message « Salut »). L’exécution s’arrêtera à la ligne où vous placez le point d’arrêt.

   ![Débogage dans Visual Studio](~/media/bot-service-debug-bot/breakpoint-caught-vs.png)

::: moniker range="azure-bot-service-3.0" 

## <a id="debug-csharp-serverless"></a> Déboguer un bot Plan consommation C\# Functions

L’environnement de plan consommation sans serveur C\# de Bot Service partage plus d’éléments avec Node.js qu’une application C\# standard, car il nécessite un hôte de runtime, comme le moteur du nœud. Dans Azure, le runtime fait partie de l’environnement d’hébergement dans le cloud, mais vous devez répliquer cet environnement localement sur votre bureau. 

### <a name="prerequisites"></a>Prérequis

Avant de pouvoir déboguer votre bot C# Plan de consommation, vous devez effectuer ces tâches.

- Téléchargez le code source de votre bot (à partir d’Azure), comme décrit dans [Configurer un déploiement continu](bot-service-continuous-deployment.md).
- Téléchargez et installez l’[émulateur Bot Framework](https://aka.ms/Emulator-wiki-getting-started).
- Installez l’<a href="https://www.npmjs.com/package/azure-functions-cli" target="_blank">interface de ligne de commande Azure Functions</a>.
- Installez l’<a href="https://github.com/dotnet/cli" target="_blank">interface de ligne de commande DotNet</a>.
  
Si vous souhaitez être en mesure de déboguer votre code à l’aide de points d’arrêt dans Visual Studio 2017, vous devez également effectuer ces tâches.
  
- Téléchargez et installez <a href="https://www.visualstudio.com/downloads/" target="_blank">Visual Studio 2017</a> (Community Edition ou ultérieure).
- Téléchargez et installez l’<a href="https://visualstudiogallery.msdn.microsoft.com/e6bf6a3d-7411-4494-8a1e-28c1a8c4ce99" target="_blank">extension Visual Studio Command Task Runner</a>.

> [!NOTE]
> Visual Studio Code n’est pas pris en charge pour le moment.

### <a name="debug-a-consumption-plan-c-functions-bot-using-the-emulator"></a>Déboguer un bot Plan consommation C# Functions à l’aide de l’émulateur

La façon la plus simple de déboguer votre bot localement consiste à démarrer le bot puis à vous y connecter à partir de l’émulateur Bot Framework. 
Tout d’abord, ouvrez une invite de commandes et accédez au dossier où se trouve le fichier **project.json** dans votre référentiel. Exécutez ensuite la commande `dotnet restore` pour restaurer les différents packages référencés dans votre bot.

> [!NOTE]
> Visual Studio 2017 modifie la façon dont Visual Studio gère les dépendances. Bien que Visual Studio 2015 utilise **project.json** pour gérer les dépendances, Visual Studio 2017 utilise un modèle **.csproj** lors du chargement dans Visual Studio. Si vous utilisez Visual Studio 2017, <a href="https://aka.ms/bf-debug-project">téléchargez ce fichier **.csproj**</a> dans le dossier **/messages** de votre référentiel avant d’exécuter la commande `dotnet restore`.

![Invite de commande](~/media/bot-service-debug-bot/csharp-azureservice-debug-envconfig.png)

Exécutez maintenant `debughost.cmd` pour charger et démarrer votre bot. 

![Invite de commandes run debughost.cmd](~/media/bot-service-debug-bot/csharp-azureservice-debug-debughost.png)

À ce stade, le bot s’exécute localement. À partir de la fenêtre de console, copiez le point de terminaison sur lequel debughost est à l’écoute (dans cet exemple, `http://localhost:3978`). Ensuite, démarrez l’émulateur Bot Framework et collez le point de terminaison dans la barre d’adresse de l’émulateur. Pour cet exemple, vous devez également ajouter `/api/messages` au point de terminaison. Puisque vous n’avez pas besoin de sécurité pour le débogage local, vous pouvez laisser vides les champs **ID d’application Microsoft** et **Mot de passe d’application Microsoft**. Cliquez sur **CONNECT** (Connexion) pour établir une connexion à votre bot à l’aide du point de terminaison spécifié.

![Configurer l’émulateur](~/media/bot-service-debug-bot/mac-azureservice-emulator-config.png)

Une fois que vous avez connecté l’émulateur à votre bot, envoyez un message à votre bot en tapant un texte dans la zone de texte en bas de la fenêtre de l’émulateur (où **Tapez votre message...**  s’affiche dans le coin inférieur gauche). Dans les panneaux **Log** et **Inspector** sur le côté droit de la fenêtre de l’émulateur, vous pouvez visualiser les demandes et les réponses à mesure que des messages sont échangés entre l’émulateur et le bot.

![tester via l’émulateur](~/media/bot-service-debug-bot/mac-azureservice-debug-emulator.png)

En outre, vous pouvez afficher les détails du journal dans la fenêtre de console.

![Fenêtre de console](~/media/bot-service-debug-bot/csharp-azureservice-debug-debughostlogging.png)

::: moniker-end

## <a name="additional-resources"></a>Ressources supplémentaires

- Consultez [Résoudre les problèmes généraux](bot-service-troubleshoot-bot-configuration.md) ainsi que les autres articles de résolution des problèmes de cette section.
- Consultez le billet de blog [Debug any Channel locally using ngrok](https://blog.botframework.com/2017/10/19/debug-channel-locally-using-ngrok/).

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Déboguer votre bot à l’aide de fichiers de transcription](v4sdk/bot-builder-debug-transcript.md).
