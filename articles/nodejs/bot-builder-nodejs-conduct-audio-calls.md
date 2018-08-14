---
title: Passer des appels audio | Microsoft Docs
description: Découvrez comment passer des appels audio avec Skype dans un bot à l’aide de Node.js.
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/17
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: fbfe65526335b7a8797ab229871472d540735e20
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39298896"
---
# <a name="support-audio-calls-with-skype"></a>Prise en charge des appels audio avec Skype

Skype prend en charge une fonctionnalité riche, à savoir les bots d’appel.  Lorsque cette fonctionnalité est activée, les utilisateurs peuvent passer un appel vocal à votre bot et interagir avec lui à l’aide d’une réponse vocale interactive (RVI).  Le Kit de développement logiciel (SDK) Bot Builder pour Node.js inclut un [SDK d’appel][calling_sdk] spécial que les développeurs peuvent utiliser pour ajouter des fonctionnalités d’appel à leur bot conversationnel.   

Le SDK d’appel est très similaire au [SDK de conversation][chat_sdk]. Ils incluent des classes similaires et partagent des constructions communes. Vous pouvez même utiliser le SDK de conversation pour envoyer un message à l’utilisateur que vous êtes en train d’appeler.  Les deux Kits de développement logiciel (SDK) sont conçus pour s’exécuter côte à côte. Cependant, même s’ils sont similaires, ils présentent des différences significatives et vous devez généralement éviter de combiner des classes des deux bibliothèques.  

## <a name="create-a-calling-bot"></a>Créer un bot d’appel
L’exemple de code suivant montre comment le programme « Hello World » pour un bot d’appel s’apparente à un bot conversationnel classique. 

```javascript
var restify = require('restify');
var calling = require('botbuilder-calling');

// Setup Restify Server
var server = restify.createServer();
server.listen(process.env.port || process.env.PORT || 3978, function () {
   console.log(`${server.name} listening to ${server.url}`); 
});

// Create calling bot
var connector = new calling.CallConnector({
    callbackUrl: 'https://<your host>/api/calls',
    appId: '<your bots app id>',
    appPassword: '<your bots app password>'
});
var bot = new calling.UniversalCallBot(connector);
server.post('/api/calls', connector.listen());

// Add root dialog
bot.dialog('/', function (session) {
    session.send('Watson... come here!');
});
```

> [!NOTE]
> Pour trouver les paramètres **AppID** et **AppPassword** de votre bot, consultez la rubrique [MicrosoftAppID and MicrosoftAppPassword](~/bot-service-manage-overview.md#microsoftappid-and-microsoftapppassword) (MicrosoftAppID et MicrosoftAppPassword).

Actuellement, l’émulateur ne prend pas en charge le test des bots d’appel. Pour tester votre bot, vous devez suivre la plupart des étapes de publication du bot.  Vous devez également utiliser un client Skype pour interagir avec le bot. 

### <a name="enable-the-skype-channel"></a>Activer le canal Skype
[Inscrivez votre bot](../bot-service-quickstart-registration.md) et activez le canal Skype. Vous devez fournir un point de terminaison de messagerie lorsque vous inscrivez votre bot. Il est recommandé d’associer votre bot d’appel à un bot conversationnel. Le point de terminaison du bot conversationnel représente alors ce que vous indiqueriez normalement dans ce champ.  Si vous inscrivez uniquement un bot d’appel, vous pouvez simplement coller votre point de terminaison appelant dans ce champ.  

Pour activer la fonctionnalité d’appel réelle, vous devez accéder au canal Skype de votre bot et activer cette fonctionnalité. Un champ dans lequel vous pouvez copier votre point de terminaison appelant s’affiche alors. Assurez-vous d’utiliser le lien https ngrok pour la partie hôte de votre point de terminaison appelant.

Lors de l’inscription de votre bot, un ID d’application et un mot de passe vous sont affectés. Vous devez les coller dans les paramètres du connecteur pour votre bot « Hello World ». Vous devez également coller votre lien d’appel complet pour le paramètre callbackUrl.

### <a name="add-bot-to-contacts"></a>Ajouter un bot aux contacts
Dans le portail des développeurs, la page d’inscription de votre bot inclut un bouton **add to Skype** (ajouter à Skype) en regard du canal Skype de votre bot. Cliquez sur le bouton pour ajouter votre bot à votre liste de contacts dans Skype.  Une fois cette tâche effectuée, vous serez en mesure de communiquer avec le bot (de même que toutes les personnes auxquelles vous avez fourni le lien de participation).

### <a name="test-your-bot"></a>Tester votre bot
Vous pouvez tester votre bot à l’aide d’un client Skype. Vous voyez l’icône d’appel s’allumer lorsque vous cliquez sur l’entrée de contact de votre bot (vous devrez peut-être rechercher le bot pour l’afficher.)  Vous devrez peut-être attendre quelques minutes pour voir l’icône d’appel s’allumer si vous avez ajouté l’appel à un bot existant.  

Si vous cliquez sur le bouton d’appel, le système établit la communication avec votre bot, et vous devez entendre « Watson… come here! » (Watson, viens ici.) Puis, le bot raccroche.

## <a name="calling-basics"></a>Principes de base sur les appels
Les classes [UniversalCallBot](http://docs.botframework.com/en-us/node/builder/calling-reference/classes/_botbuilder_d_.universalcallbot) et [CallConnector](http://docs.botframework.com/en-us/node/builder/calling-reference/classes/_botbuilder_d_.callconnector) vous permettent de créer un bot d’appel comme vous le feriez pour un bot conversationnel, à quelques détails près. Vous ajoutez des boîtes de dialogue à votre bot, globalement identiques aux [boîtes de dialogue de conversation](bot-builder-nodejs-manage-conversation-flow.md). Vous pouvez ajouter des [cascades](bot-builder-nodejs-prompts.md) à votre bot. L’objet de session, la classe [CallSession](http://docs.botframework.com/en-us/node/builder/calling-reference/classes/_botbuilder_d_.callsession), qui contient les méthodes [answer()](http://docs.botframework.com/en-us/node/builder/calling-reference/classes/_botbuilder_d_.callsession#answer), [hangup()](http://docs.botframework.com/en-us/node/builder/calling-reference/classes/_botbuilder_d_.callsession#hangup), et [reject()](http://docs.botframework.com/en-us/node/builder/calling-reference/classes/_botbuilder_d_.callsession#reject) ajoutées pour la gestion de l’appel en cours. Cependant, en règle générale, vous n’avez pas à vous soucier de ces méthodes de contrôle d’appel. En effet, la classe CallSession possède la logique pour gérer automatiquement l’appel pour vous. La session répondra automatiquement à l’appel si vous exécutez une action comme l’envoi d’un message ou l’appel d’une invite intégrée. De plus, elle raccrochera/refusera l’appel automatiquement si vous appelez [endConversation()](http://docs.botframework.com/en-us/node/builder/calling-reference/classes/_botbuilder_d_.callsession#endconversation) ou si elle détecte que vous avez cessé de poser des questions à l’appelant (vous n’avez pas appelé d’invite intégrée.)

Les bots d’appel et bots conversationnels présentent une autre différence : généralement, les bots conversationnels envoient des messages, des cartes et des claviers à un utilisateur, alors qu’un bot d’appel est axé sur des actions et des résultats. Les bots d’appel Skype doivent créer des [flux de travail](http://docs.botframework.com/en-us/node/builder/calling-reference/interfaces/_botbuilder_d_.iworkflow) composés d’une ou plusieurs [actions](http://docs.botframework.com/en-us/node/builder/calling-reference/interfaces/_botbuilder_d_.iaction).  Dans la pratique, vous n’avez pas non plus à vous soucier de cet aspect. En effet, le Kit de développement logiciel (SDK) Bot Builder le gère principalement pour vous. La méthode [CallSession.send()](http://docs.botframework.com/en-us/node/builder/calling-reference/classes/_botbuilder_d_.callsession#send) vous permet de passer des actions ou des chaînes, qu’elle transforme en actions [PlayPromptActions](http://docs.botframework.com/en-us/node/builder/calling-reference/classes/_botbuilder_d_.playpromptaction).  La session contient une logique de traitement par lot automatique pour combiner plusieurs actions au sein d’un flux de travail unique soumis au service appelant. Ainsi, vous pouvez appeler send() plusieurs fois en toute sécurité.  De plus, vous devez vous appuyer sur les [invites](bot-builder-nodejs-prompts.md) intégrées du Kit de développement logiciel (SDK) pour collecter les entrées de l’utilisateur dans la mesure où elles traitent tous les résultats.  

[calling_sdk]: http://docs.botframework.com/en-us/node/builder/calling-reference/modules/_botbuilder_d_
[chat_sdk]: http://docs.botframework.com/en-us/node/builder/chat-reference/modules/_botbuilder_d_