---
title: Créer un bot avec le Kit SDK Bot Builder pour JavaScript | Microsoft Docs
description: Créez rapidement un bot à l’aide du Kit SDK Bot Builder pour JavaScript.
keywords: démarrage rapide, kit sdk bot builder, mise en route
author: jonathanfingold
ms.author: v-jofing
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 10/30/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 1b111125ea240bf89f506106c948c6b5d3be7649
ms.sourcegitcommit: 15f7fa40b7e0a05507cdc66adf75bcfc9533e781
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/02/2018
ms.locfileid: "50916816"
---
# <a name="create-a-bot-with-the-bot-builder-sdk-for-javascript"></a>Créer un bot avec le Kit de développement logiciel (SDK) Bot Builder pour JavaScript

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Ce guide de démarrage rapide vous pilote tout au long de la création d’un bot, en utilisant le générateur Yeoman Bot Builder et le Kit SDK Bot Builder pour JavaScript, puis en testant votre bot avec l’émulateur Bot Framework.

## <a name="prerequisites"></a>Prérequis

- [Visual Studio Code](https://www.visualstudio.com/downloads)
- [Node.JS](https://nodejs.org/)
- [Yeoman](http://yeoman.io/), qui utilise un générateur afin de créer un bot pour vous
- [git](https://git-scm.com/)
- [Bot Framework Emulator](https://github.com/Microsoft/BotFramework-Emulator)
- Connaissances de [restify](http://restify.com/) et de la programmation asynchrone en JavaScript

> [!NOTE]
> Pour certaines installations, l’étape d’installation de restify génère une erreur liée à node-gyp.
> Si c’est le cas, essayez d’exécuter cette commande avec des autorisations élevées :
> ```bash
> npm install -g windows-build-tools
> ```

## <a name="create-a-bot"></a>Créer un bot

Pour créer votre bot et initialiser ses packages

1. Ouvrez un terminal ou une invite de commandes avec élévation de privilèges.
1. Si vous ne disposez pas d’un répertoire pour vos bots JavaScript, créez-le et remplacez les répertoires par ce dernier. (Nous créons un répertoire pour vos bots JavaScript en général, même si nous créons un seul bot dans ce didacticiel.)

   ```bash
   md myJsBots
   cd myJsBots
   ```

1. Vérifiez que votre version de npm est à jour.

   ```bash
   npm install -g npm
   ```

1. Installez maintenant Yeoman et le générateur pour JavaScript.

   ```bash
   npm install -g yo generator-botbuilder
   ```

1. Ensuite, utilisez le générateur pour créer un bot d’écho.

   ```bash
   yo botbuilder
   ```

Yeoman vous invite à entrer des informations afin de créer votre bot. Pour ce didacticiel, utilisez les valeurs par défaut.

- Entrez un nom pour votre bot. (myChatBot)
- Saisissez une description. (Illustrer les fonctionnalités principales de Microsoft Bot Framework)
- Choisissez la langue de votre bot. (JavaScript)
- Choisissez le modèle à utiliser. (Echo)

Grâce au modèle, votre projet contient tout le code nécessaire pour créer le bot dans ce guide de démarrage rapide. En fait, vous n’avez pas besoin d’écrire du code supplémentaire.

> [!NOTE]
> Si vous choisissez de créer un bot `Basic`, vous aurez besoin d’un modèle de langage LUIS. Vous pouvez en créer un sur le portail [luis.ai](https://www.luis.ai). Après avoir créé le modèle, mettez à jour le fichier .bot. Votre fichier bot ressemble à [ceci](../v4sdk/bot-builder-service-file.md).

## <a name="start-your-bot"></a>Démarrer votre robot

Dans un terminal ou une invite de commandes, remplacez les répertoires par celui créé pour votre bot, puis démarrez-le avec `npm start`. À ce stade, votre bot s’exécute localement.

## <a name="start-the-emulator-and-connect-your-bot"></a>Démarrer l’émulateur et connecter votre robot

1. Démarrez le Bot Framework Emulator.
2. Cliquez sur le lien **Ouvrir le bot** de l’onglet de bienvenue de l’émulateur.
3. Sélectionnez le fichier .bot situé dans le répertoire de création du projet.

Envoyez un message à votre bot, et le bot vous enverra un message à son tour.
![Émulateur en cours d’exécution](../media/emulator-v4/js-quickstart.png)

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Fonctionnement des bots](../v4sdk/bot-builder-basics.md)
