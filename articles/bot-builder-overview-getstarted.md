---
title: Démarrer la création de robots avec Bot Builder | Microsoft Docs
description: Démarrez la création de puissants robots avec Bot Builder.
author: robstand
ms.author: kamrani
manager: kamrani
ms.topic: get-started-article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 21186c5d3b0769311e4703ca1dab2f48a0a0081a
ms.sourcegitcommit: 67445b42796d90661afc643c6bb6533e9a662cbc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/06/2018
ms.locfileid: "39574885"
---
# <a name="develop-bots-with-bot-builder"></a>Développer des robots avec Bot Builder

[!INCLUDE [pre-release-label](includes/pre-release-label-v3.md)]

Bot Builder fournit un kit de développement logiciel, des bibliothèques, des exemples et des outils permettant de créer et de déboguer des robots. Lorsque vous créez un robot avec Bot Service, celui-ci est sauvegardé par le kit de développement logiciel Bot Builder. Vous pouvez également vous servir du kit de développement logiciel Bot Builder pour créer un robot en partant de zéro à l’aide de C# ou de Node.js. Bot Builder comprend l’émulateur de Bot Framework permettant de tester les robots et l’inspecteur de canaux permettant de prévisualiser l’expérience utilisateur assurée par le robot sur différents canaux.

Il est possible d’utiliser l’API REST pour créer un robot à l’aide de n’importe quel langage de programmation.

## <a name="bot-builder-sdk-for-net"></a>Kit de développement logiciel Bot Builder pour .NET
Le kit de développement logiciel Bot Builder pour .NET utilise le C# pour que les développeurs .NET puissent coder des robots dans un environnement qu’ils connaissent bien. C’est une infrastructure puissante qui permet de créer des robots capables de gérer aussi bien des interactions libres que des conversations plus ciblées dans lesquelles l’utilisateur choisit parmi plusieurs valeurs possibles. 

Le [démarrage rapide .NET](~/dotnet/bot-builder-dotnet-quickstart.md) permet de guider dans la création d’un robot avec le kit de développement logiciel Bot Builder pour .NET.

Le kit de développement logiciel Bot Builder pour .NET est disponible sous forme de package [NuGet](https://www.nuget.org/packages/Microsoft.Bot.Builder/).

> [!IMPORTANT]
> Le kit de développement logiciel Bot Builder pour .NET nécessite la version 4.6 ou une version ultérieure de .NET Framework. Si vous ajoutez le kit de développement logiciel à un projet existant destiné à une version inférieure de .NET Framework, vous devrez d’abord mettre à jour votre projet pour qu’il corresponde à la version 4.6 de .NET Framework.

Pour installer le kit de développement logiciel dans un projet Visual Studio existant, procédez comme suit :

1. Dans l’**Explorateur de solutions**, cliquez avec le bouton droit sur le nom du projet, puis sélectionnez **Gérer les packages NuGet...**.
2. Sur l’onglet **Parcourir**, saisissez « Microsoft.Bot.Builder » dans la zone de recherche.
3. Sélectionnez **Microsoft.Bot.Builder** dans la liste des résultats, cliquez sur **Installer** et acceptez les modifications.

Le kit de développement logiciel Bot Builder pour le [code source](https://github.com/Microsoft/BotBuilder/tree/master/CSharp) .NET est également téléchargeable sur GitHub.

### <a name="visual-studio-project-templates"></a>Modèles de projet Visual Studio
Pour développer des robots plus rapidement, téléchargez des modèles de projet Visual Studio.

* [Modèle de robot pour Visual Studio][bot-template] pour développer des robots en C#
* [Modèle de compétences Cortana pour Visual Studio][cortana-template] pour développer des compétences Cortana en C#

> [!TIP]
> <a href="/visualstudio/ide/how-to-locate-and-organize-project-and-item-templates" target="_blank">En savoir plus</a> sur l’installation des modèles de projet Visual Studio 2017.

## <a name="bot-builder-sdk-for-nodejs"></a>kit de développement logiciel Bot Builder pour Node.js
Le kit de développement logiciel Bot Builder pour Node.js permet aux développeurs de Node.js de coder des robots dans un environnement qu’ils connaissent bien. Il permet de créer une large palette d’interfaces utilisateur conversationnelles, allant de simples invites à des conversations libres. Le kit de développement logiciel Bot Builder pour Node.js utilise restify pour créer le serveur web du robot. Il s’agit d’une infrastructure courante pour la création de services web. Le kit de développement logiciel est également compatible avec Express. 

Le [démarrage rapide Node.js](~/nodejs/bot-builder-nodejs-quickstart.md) permet de guider dans la création d’un robot avec le kit de développement logiciel Bot Builder pour Node.js. 

Le kit de développement logiciel Bot Builder pour Node.js est disponible sous forme de package npm. Pour installer le kit de développement logiciel pour Node.js et ses dépendances, commencez par créer un dossier pour le robot, accédez à ce dossier, puis exécutez la commande **npm** suivante :

```nodejs
npm init
```

Installez ensuite le kit de développement logiciel Bot Builder pour Node.js et les modules <a href="http://restify.com/" target="_blank">Restify</a>en exécutant les commandes **npm** suivantes :

```nodejs
npm install --save botbuilder
npm install --save restify
```

Le kit de développement logiciel Bot Builder pour le [code source](https://github.com/Microsoft/BotBuilder/tree/master/Node) Node.js est également téléchargeable sur GitHub.

## <a name="rest-api"></a>API REST

Il est possible de créer un robot avec n’importe quel langage de programmation à l’aide de l’API REST du Bot Framework. Le [démarrage rapide REST](rest-api/bot-framework-rest-connector-quickstart.md) permet de guider dans la création d’un robot avec REST.

Bot Framework contient trois API REST :

 - L’[API REST Bot Connector][connectorAPI] permet au robot d’envoyer et de recevoir des messages sur les canaux configurés dans le [portail Bot Framework](https://dev.botframework.com/). 

- L’[API REST Bot State][stateAPI] permet au robot de stocker et de récupérer l’état associé aux conversations menées par l’API REST Bot Connector.

- L’[API REST Direct Line][directLineAPI] permet de connecter sa propre application, comme une application client, un contrôle de conversation Web ou une application mobile, directement à un seul robot.

## <a name="bot-framework-emulator"></a>Émulateur de Bot Framework
L’émulateur de Bot Framework est une application de bureau qui permet de tester et de déboguer les robots, en local ou à distance. L’émulateur permet de dialoguer avec le robot et d’inspecter les messages qu’il envoie et reçoit. [En savoir plus sur l’émulateur de Bot Framework](~/bot-service-debug-emulator.md) et [télécharger l’émulateur](http://emulator.botframework.com) correspondant à votre plate-forme.

## <a name="bot-framework-channel-inspector"></a>Inspecteur de canaux de Bot Framework
Visualisez en un clin d’œil à quoi ressemblent les fonctionnalités du robot sur différents canaux à l’aide de l’[inspecteur de canaux](bot-service-channel-inspector.md) de Bot Framework.

[bot-template]: http://aka.ms/bf-bc-vstemplate
[cortana-template]: https://aka.ms/bf-cortanaskill-template


[connectorAPI]: https://docs.botframework.com/en-us/restapi/connector/#navtitle
 
[stateAPI]: https://docs.botframework.com/en-us/restapi/state/#navtitle

[directLineAPI]: https://docs.botframework.com/en-us/restapi/directline3/#navtitle
