---
title: Créer un bot avec le Kit SDK Bot Builder pour JavaScript | Microsoft Docs
description: Créez rapidement un bot à l’aide du Kit SDK Bot Builder pour JavaScript.
keywords: démarrage rapide, kit sdk bot builder, mise en route
author: jonathanfingold
ms.author: jonathanfingold
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 07/12/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 21a9aa5b1d108b5d03b108278a81229e16b5bc99
ms.sourcegitcommit: a2f3d87c0f252e876b3e63d75047ad1e7e110b47
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/25/2018
ms.locfileid: "42928125"
---
# <a name="create-a-bot-with-the-bot-builder-sdk-v4-preview-for-javascript"></a>Créer un bot avec le Kit SDK Bot Builder v4 (préversion) pour JavaScript

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Ce guide de démarrage rapide vous pilote tout au long de la création d’un bot, en utilisant le générateur Yeoman Bot Builder et le Kit SDK Bot Builder pour JavaScript, puis en testant votre bot avec l’émulateur Bot Framework. Ce guide s’appuie sur le [kit SDK Microsoft Bot Builder v4](https://github.com/Microsoft/botbuilder-js).

## <a name="prerequisites"></a>Prérequis

- [Visual Studio Code](https://www.visualstudio.com/downloads)
- [Node.JS](https://nodejs.org/en/)
- [Yeoman](http://yeoman.io/), qui peut utiliser un générateur afin de créer un bot pour vous
- [Émulateur de bot](https://github.com/Microsoft/BotFramework-Emulator)
- Connaissances de [restify](http://restify.com/) et de la programmation asynchrone en JavaScript

> [!NOTE]
> Pour certaines installations, l’étape d’installation de restify génère une erreur liée à node-gyp.
> Si c’est le cas, essayez de lancer `npm install -g windows-build-tools`.

Le Kit SDK Bot Builder pour JavaScript se compose d’une série de [packages](https://github.com/Microsoft/botbuilder-js/tree/master/libraries) qui peuvent être installés à partir de NPM à l’aide d’une balise `@preview` spéciale.

## <a name="create-a-bot"></a>Créer un bot

Ouvrez une invite de commandes avec élévation de privilèges, créez un répertoire, puis initialisez le package pour votre bot.

```bash
md myJsBots
cd myJsBots
```

Installez maintenant Yeoman et le générateur pour JavaScript.

```bash
npm install -g yo
npm install -g generator-botbuilder@preview
```

Ensuite, utilisez le générateur pour créer un bot d’écho.

```bash
yo botbuilder
```

Yeoman vous invite à entrer des informations afin de créer votre bot.

- Entrez un nom pour votre bot.
- Saisissez une description.
- Choisissez la langue de votre bot, `JavaScript` ou `TypeScript`.
- Choisissez le modèle à utiliser. Actuellement, `Echo` est le seul modèle, mais d’autres modèles seront bientôt disponibles.

Yeoman crée votre bot dans un nouveau dossier.

## <a name="explore-code"></a>Explorer le code

Lorsque vous ouvrez le nouveau dossier de votre bot, un fichier `app.js` apparaît. Ce fichier `app.js` contient tout le code nécessaire pour exécuter une application de bot. Ce fichier contient un bot d’écho qui renverra tout ce qui a été entré, tout en incrémentant un compteur.

Dans le code suivant, le middleware d’état de la conversation utilise le stockage en mémoire. Il lit et écrit l’objet d’état dans le stockage. La variable numérique conserve une trace du nombre de messages envoyés au bot. Vous pouvez utiliser une technique similaire pour conserver l’état entre les tours.

**app.js**
```javascript
// Packages are installed for you
const { BotFrameworkAdapter, MemoryStorage, ConversationState } = require('botbuilder');
const restify = require('restify');

// Create server
let server = restify.createServer();
server.listen(process.env.port || process.env.PORT || 3978, function () {
    console.log(`${server.name} listening to ${server.url}`);
});

// Create adapter
const adapter = new BotFrameworkAdapter({
    appId: process.env.MICROSOFT_APP_ID,
    appPassword: process.env.MICROSOFT_APP_PASSWORD
});

// Add conversation state middleware
const conversationState = new ConversationState(new MemoryStorage());
adapter.use(conversationState);
```

Le code suivant est à l’écoute des requêtes entrantes et vérifie le type d’activité entrant avant d’envoyer une réponse à l’utilisateur.

```javascript
// Listen for incoming requests
server.post('/api/messages', (req, res) => {
    // Route received request to adapter for processing
    adapter.processActivity(req, res, (context) => {
        // This bot is only handling Messages
        if (context.activity.type === 'message') {

            // Get the conversation state
            const state = conversationState.get(context);

            // If state.count is undefined set it to 0, otherwise increment it by 1
            const count = state.count === undefined ? state.count = 0 : ++state.count;

            // Echo back to the user whatever they typed.
            return context.sendActivity(`${count}: You said "${context.activity.text}"`);
        } else {
            // Echo back the type of activity the bot detected if not of type message
            return context.sendActivity(`[${context.activity.type} event detected]`);
        }
    });
});
```

## <a name="start-your-bot"></a>Démarrer votre bot

Remplacez les répertoires par celui créé pour votre bot, puis démarrez-le.

```bash
cd <bot directory>
node app.js
```

## <a name="start-the-emulator-and-connect-your-bot"></a>Démarrer l’émulateur et se connecter votre bot

À ce stade, votre bot s’exécute localement. Démarrez à présent l’émulateur, puis connectez-vous à votre bot dans l’émulateur :

1. Cliquez sur le lien **create a new bot configuration** (créer une configuration de bot) sous l’onglet « Welcome » (Bienvenue) de l’émulateur.
1. Renseignez le champ **Bot name** (Nom du bot) et indiquez le chemin d’accès au répertoire de votre code de bot. Le fichier de configuration du bot y sera enregistré.
1. Tapez `http://localhost:port-number/api/messages` dans le champ **URL du point de terminaison**, où *port-number* correspond au numéro de port affiché dans le navigateur où s’exécute votre application.
1. Cliquez sur **Se connecter** pour vous connecter à votre bot. Vous n’avez pas besoin de spécifier les valeurs **Microsoft App ID** (ID d’application Microsoft) et **Microsoft App Password** (Mot de passe d’application Microsoft). Vous pouvez laisser ces champs vides pour l’instant. Vous obtiendrez ces informations ultérieurement, lors de l’inscription du bot.

Envoyez « Hi » (Bonjour) à votre bot, il répond au message par « 0: You said Hi » (0 : Vous avez dit Bonjour).

## <a name="next-steps"></a>Étapes suivantes

À présent, [déployez votre bot sur Azure](../bot-builder-howto-deploy-azure.md) ou passez aux concepts qui décrivent un bot et son fonctionnement.

> [!div class="nextstepaction"]
> [Concepts Bot de base](../v4sdk/bot-builder-basics.md)
