---
title: Créer un robot avec le Kit de développement logiciel (SDK) Bot Builder pour Node.js | Microsoft Docs
description: Créez un robot avec le Kit de développement logiciel (SDK) Bot Builder pour Node.js, une puissante infrastructure de construction de robot.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: get-started-article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 7259e1336965733b860a6129d51b3c2eb10362c7
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39299337"
---
# <a name="create-a-bot-with-the-bot-builder-sdk-for-nodejs"></a>Créer un robot avec le Kit de développement logiciel (SDK) Bot Builder pour Node.js
> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-quickstart.md)
> - [Node.JS](../nodejs/bot-builder-nodejs-quickstart.md)
> - [Service Bot](../bot-service-quickstart.md)
> - [REST](../rest-api/bot-framework-rest-connector-quickstart.md)

Le Kit de développement logiciel Bot Builder pour Node.js est une infrastructure pour le développement de robots. Il est facile à utiliser et modélise des infrastructures telles que Express & Restify pour fournir aux développeurs JavaScript un moyen familier d’écrire des robots.

Ce didacticiel vous guide tout au long de la création d’un robot à l’aide du Kit de développement logiciel (SDK) Bot Builder pour Node.js. Vous pouvez tester le robot dans une fenêtre de console et avec l’émulateur Bot Framework.

## <a name="prerequisites"></a>Prérequis
Commencez par accomplir les tâches préalables suivantes :

1. Installez [Node.js](https://nodejs.org).
2. Créez un dossier pour votre robot.
3. À partir d’un terminal ou d’une invite de commandes, accédez au dossier que vous venez de créer.
4. Exécutez la commande **npm** suivante :

```nodejs
npm init
```

Suivez les instructions à l’écran pour entrer des informations sur votre robot afin de permettre à npm de créer un fichier **package.json** contenant les informations fournies. 

## <a name="install-the-sdk"></a>Installer le Kit de développement logiciel (SDK)
Installez ensuite le kit de développement logiciel Bot Builder pour Node.js et en exécutant les commandes **npm** suivantes :

```nodejs
npm install --save botbuilder
```

Après avoir installé le Kit de développement logiciel (SDK), vous êtes prêt à écrire votre premier robot.

Pour votre première expérience, vous allez créer un robot qui renvoie en écho les entrées de l’utilisateur. Pour créer votre robot, procédez comme suit :

1. Dans le dossier que vous avez créé précédemment pour votre bot, créez un fichier nommé **app.js**.
2. Ouvrez **app.js** dans un éditeur de texte ou un environnement de développement intégré (IDE) de votre choix. Ajoutez le code suivant au fichier : 

   [!code-javascript[consolebot code sample Node.js](../includes/code/node-getstarted.js#consolebot)]

3. Enregistrez le fichier . Vous êtes maintenant prêt à exécuter et à tester votre robot.

### <a name="start-your-bot"></a>Démarrer votre robot

Accédez au répertoire de votre robot dans une fenêtre de console, puis démarrez votre robot :

```nodejs
node app.js
```

Le robot s’exécute à présent localement. Tester votre robot en tapant quelques messages dans la fenêtre de console.
Vous devriez voir que le robot répondre à chaque message que vous envoyez en renvoyant votre message précédé du texte *« Vous avez dit: »*.

## <a name="install-restify"></a>Installation de Restify

Les robots de console sont de bons clients basés sur du texte mais, pour utiliser l’un des canaux de Bot Framework (ou exécuter votre robot dans l’émulateur), votre robot doit s’exécuter sur un point de terminaison d’API. Installez <a href="http://restify.com/" target="_blank">restify</a> en exécutant la commande **npm** suivante :

```nodejs
npm install --save restify
```

Une fois Restify en place, vous êtes prêt à apporter des modifications à votre robot.

## <a name="edit-your-bot"></a>Modifier votre robot

Vous devez apporter des modifications à votre fichier **app.js**. 

1. Ajoutez une ligne pour exiger le module `restify`.
2. Remplacez `ConsoleConnector` par `ChatConnector`.
3. Incluez vos ID et mot de passe d’application Microsoft.
4. Faites en sorte que le connecteur écoute sur un point de terminaison d’API.

   [!code-javascript[echobot code sample Node.js](../includes/code/node-getstarted.js#echobot)]

5. Enregistrez le fichier . Vous êtes désormais prêt à exécuter et à tester votre robot dans l’émulateur.

> [!NOTE] 
> Vous n’avez pas besoin un **ID d’application Microsoft** ou d’un **Mot de passe d’application Microsoft** pour exécuter votre robot dans l’émulateur Bot Framework.

## <a name="test-your-bot"></a>Tester votre robot
Ensuite, testez votre robot à l’aide de l’[émulateur Bot Framework](../bot-service-debug-emulator.md) pour le voir en action. L’émulateur est une application de bureau qui vous permet de tester et déboguer votre robot sur un hôte local ou de l’exécuter à distance via un tunnel.

Commencez par [télécharger](https://emulator.botframework.com) et installer l’émulateur. Une fois le téléchargement terminé, lancez l’exécutable et accomplissez le processus d’installation.

### <a name="start-your-bot"></a>Démarrer votre robot

Une fois l’émulateur installé, accédez au répertoire de votre robot dans une fenêtre de console, puis démarrez le robot :

```nodejs
node app.js
```
   
Le robot s’exécute à présent localement.

### <a name="start-the-emulator-and-connect-your-bot"></a>Démarrer l’émulateur et connecter votre robot
Une fois votre robot démarré, démarrez l’émulateur; puis connectez votre robot :

1. Cliquez sur le lien **create a new bot configuration** (créer une configuration de robot) dans la fenêtre de l’émulateur. 

2. Tapez `http://localhost:port-number/api/messages` dans la barre d’adresses, où *port-number* correspond au numéro de port affiché dans le navigateur dans lequel votre application est en cours d’exécution.

3. Cliquez sur Enregistrer et se connecter. Inutile de spécifier les ID et mot de passe d’application Microsoft. Vous pouvez laisser ces champs vides pour l’instant. Vous obtiendrez ces informations ultérieurement, lors de l’inscription du robot.

### <a name="try-out-your-bot"></a>Tester votre robot

À présent que votre robot s’exécute localement et qu’il est connecté à l’émulateur, essayez-le en entrant quelques messages dans l’émulateur.
Vous devriez voir que le robot répondre à chaque message que vous envoyez en renvoyant votre message précédé du texte *« Vous avez dit: »*.

Vous avez créé votre premier robot à l’aide du Kit de développement logiciel (SDK) Bot Builder pour Node.js !

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Kit de développement logiciel Bot Builder pour Node.js](bot-builder-nodejs-overview.md)
