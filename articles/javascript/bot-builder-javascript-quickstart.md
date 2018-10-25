---
title: Créer un bot avec le Kit SDK Bot Builder pour JavaScript | Microsoft Docs
description: Créez rapidement un bot à l’aide du Kit SDK Bot Builder pour JavaScript.
keywords: démarrage rapide, kit sdk bot builder, mise en route
author: jonathanfingold
ms.author: jonathanfingold
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 10/15/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: aa13889cea2a26bf094a919f5d05905d65f7661f
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2018
ms.locfileid: "49998852"
---
# <a name="create-a-bot-with-the-bot-builder-sdk-for-javascript"></a>Créer un bot avec le Kit de développement logiciel (SDK) Bot Builder pour JavaScript

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Ce guide de démarrage rapide vous pilote tout au long de la création d’un bot, en utilisant le générateur Yeoman Bot Builder et le Kit SDK Bot Builder pour JavaScript, puis en testant votre bot avec l’émulateur Bot Framework. 

## <a name="prerequisites"></a>Prérequis

- [Visual Studio Code](https://www.visualstudio.com/downloads)
- [Node.JS](https://nodejs.org/)
- [Yeoman](http://yeoman.io/), qui peut utiliser un générateur afin de créer un bot pour vous
- [git](https://git-scm.com/)
- [Bot Framework Emulator](https://github.com/Microsoft/BotFramework-Emulator)
- Connaissances de [restify](http://restify.com/) et de la programmation asynchrone en JavaScript

> [!NOTE]
> Pour certaines installations, l’étape d’installation de restify génère une erreur liée à node-gyp.
> Si c’est le cas, essayez de lancer `npm install -g windows-build-tools`.

## <a name="create-a-bot"></a>Créer un bot

Ouvrez une invite de commandes avec élévation de privilèges, créez un répertoire, puis initialisez le package pour votre bot.

```bash
md myJsBots
cd myJsBots
```

Vérifiez que votre version de npm est à jour.
```bash
npm i npm
```

Installez maintenant Yeoman et le générateur pour JavaScript.

```bash
npm install -g yo
npm install -g generator-botbuilder
```

Ensuite, utilisez le générateur pour créer un bot d’écho.

```bash
yo botbuilder
```

Yeoman vous invite à entrer des informations afin de créer votre bot.

- Entrez un nom pour votre bot.
- Saisissez une description.
- Choisissez la langue de votre bot, `JavaScript` ou `TypeScript`.
- Choisissez le modèle `Echo`.

Grâce au modèle, votre projet contient tout le code nécessaire pour créer le bot dans ce guide de démarrage rapide. En fait, vous n’avez pas besoin d’écrire du code supplémentaire.

> [!NOTE]
> Pour un bot de base, vous aurez besoin d’un modèle de langage LUIS. Vous pouvez en créer un sur le portail [luis.ai](https://www.luis.ai). Après avoir créé le modèle, mettez à jour le fichier .bot. Votre fichier bot ressemble à [ceci](../v4sdk/bot-builder-service-file.md). 

## <a name="start-your-bot"></a>Démarrer votre robot

Dans Powershell/Bash, remplacez les répertoires par celui créé pour votre bot, puis démarrez-le avec `npm start`. À ce stade, votre bot s’exécute localement.

## <a name="start-the-emulator-and-connect-your-bot"></a>Démarrer l’émulateur et connecter votre robot
1. Démarrez l’émulateur.
2. Cliquez sur le lien **Ouvrir le bot** de l’onglet de bienvenue de l’émulateur.
3. Sélectionnez le fichier .bot situé dans le répertoire de création du projet.

Envoyez un message à votre bot, et le bot vous enverra un message à son tour.
![Émulateur en cours d’exécution](../media/emulator-v4/emulator-running.png)

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Fonctionnement des bots](../v4sdk/bot-builder-basics.md) 
