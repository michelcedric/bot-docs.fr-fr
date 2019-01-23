---
title: Concepts clés du kit SDK Bot Framework pour Node.js | Microsoft Docs
description: Assimilez les concepts clés et les outils permettant de créer et de déployer les bots conversationnels disponibles dans le kit SDK Bot Framework pour Node.js.
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: efd47cb1ae48c34d58d673eaea04feeb1869b640
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/11/2019
ms.locfileid: "54225444"
---
# <a name="key-concepts-in-the-bot-framework-sdk-for-nodejs"></a>Concepts clés du kit SDK Bot Framework pour Node.js

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-concepts.md)
> - [Node.JS](../nodejs/bot-builder-nodejs-concepts.md)

Cet article présente les concepts clés du kit SDK Bot Framework pour Node.js. Pour une présentation de Bot Framework, consultez [Vue d’ensemble de Bot Framework](../overview-introduction-bot-framework.md).

## <a name="connector"></a>Connecteur

Le connecteur de Bot Framework est un service qui connecte votre robot à plusieurs *canaux* qui sont des clients comme Skype, Facebook, Slack et SMS. Le connecteur facilite la communication entre le robot et l’utilisateur en transmettant les messages du robot au canal, et inversement. La logique de votre robot est hébergée comme un service web qui reçoit les messages des utilisateurs par le biais du service de connecteur. Les réponses de votre robot sont envoyées au connecteur à l’aide d’une PUBLICATION HTTPS. 

Le kit SDK Bot Framework pour Node.js propose les classes [UniversalBot][UniversalBot] et [ChatConnector][ChatConnector] pour configurer le bot de façon à envoyer et recevoir des messages via Bot Framework Connector. La classe `UniversalBot` constitue le cerveau de votre robot. Elle est chargée de gérer toutes les conversations de votre robot avec les utilisateurs. La classe `ChatConnector` connecte votre bot au service du connecteur de Bot Framework.
Pour obtenir un exemple qui montre l’utilisation de ces classes, consultez [Créer un bot avec le kit SDK Bot Framework pour Node.js](bot-builder-nodejs-quickstart.md).

Le connecteur normalise également les messages que le robot envoie sur les canaux afin que vous puissiez développer votre robot indépendamment de la plate-forme. La normalisation d’un message implique sa conversion depuis le schéma Bot Framework vers le schéma du canal. Lorsque le canal ne prend pas en charge tous les aspects du schéma du cadre, le connecteur tente de convertir le message dans un format pris en charge par le canal. Par exemple, si le robot envoie au canal SMS un message contenant une carte avec des boutons d’action, le connecteur peut présenter la carte sous forme d’image et inclure les actions sous forme de liens dans le texte du message. L’[inspecteur de canaux][ChannelInspector] est un outil Web qui vous montre comment le connecteur présente les messages sur les différents canaux.

L’élément `ChatConnector` nécessite la configuration d’un point de terminaison API dans votre robot. Dans le kit de développement logiciel Node.js, cela s’effectue généralement en installant le module Node.js `restify`. Il est également possible de créer des robots pour la console via [ConsoleConnector][ConsoleConnector] qui ne nécessite pas de terminal API.

## <a name="messages"></a>Messages

Les messages peuvent être constitués de texte à afficher, de texte à prononcer, de pièces jointes, de cartes enrichies et de suggestions d’actions. La méthode [session.send][SessionSend] vous permet d’envoyer des messages en réponse à un message de l’utilisateur. Votre robot peut appeler `send` autant de fois qu’il le souhaite en réponse à un message de l’utilisateur. Pour obtenir un exemple qui l’illustre, consultez [Répondez aux messages utilisateur][RespondMessages].

Pour obtenir un exemple qui illustre comment envoyer une carte graphique enrichie contenant des boutons interactifs sur lesquels l’utilisateur peut cliquer pour lancer une action, consultez [Ajouter des cartes enrichies aux messages](bot-builder-nodejs-send-rich-cards.md). Pour obtenir un exemple illustrant comment envoyer et recevoir des pièces jointes, consultez [Envoyer des pièces jointes](bot-builder-nodejs-send-receive-attachments.md). Pour obtenir un exemple illustrant comment envoyer un message qui précise le texte devant être prononcé par votre robot sur un canal vocal, consultez [Ajouter la synthèse vocale aux messages](bot-builder-nodejs-text-to-speech.md). Pour obtenir un exemple qui illustre comment envoyer des suggestions d’actions, consultez [Envoyer des suggestions d’actions](bot-builder-nodejs-send-suggested-actions.md).

## <a name="dialogs"></a>Dialogues
Les dialogues vous permettent d’organiser la logique conversationnelle de votre robot. Ils sont essentiels à la [conception du flux de conversation](../bot-service-design-conversation-flow.md). Pour obtenir une présentation des dialogues, consultez [Gérer une conversation à l’aide des dialogues](bot-builder-nodejs-dialog-manage-conversation.md).

## <a name="actions"></a>Actions
Vous souhaitez concevoir votre robot de manière à pouvoir gérer les interruptions à tout moment de la conversation, par exemple les requêtes d’annulation ou d’aide. Le kit SDK Bot Framework pour Node.js propose des gestionnaires généraux de messages qui déclenchent des actions comme l’annulation ou l’appel d’autres dialogues. Pour découvrir des exemples illustrant l’utilisation des gestionnaires [triggerAction][triggerAction], consultez l’article [Gérer les actions de l’utilisateur](bot-builder-nodejs-dialog-actions.md).
<!--[Handling cancel](bot-builder-nodejs-manage-conversation-flow.md#handling-cancel), [Confirming interruptions](bot-builder-nodejs-manage-conversation-flow.md#confirming-interruptions) and-->


## <a name="recognizers"></a>Modules de reconnaissance
Lorsque les utilisateurs demandent quelque chose à votre bot, par exemple « aider » ou « trouver des actualités », celui-ci a besoin de comprendre ce que l’utilisateur demande et de lancer l’action qui convient. Vous pouvez concevoir votre robot de manière à reconnaître les intentions en fonction de la saisie de l’utilisateur et les associer à des actions. 

Vous pouvez utiliser le module intégré de reconnaissance d’expressions régulières fourni par le kit SDK Bot Framework, faire appel à un service externe comme l’API LUIS, ou implémenter un module de reconnaissance personnalisé permettant de déterminer l’intention de l’utilisateur. Pour obtenir des exemples illustrant comment ajouter des modules de reconnaissance à votre robot et comment les utiliser pour déclencher des actions, consultez [Reconnaître l’intention de l’utilisateur](bot-builder-nodejs-recognize-intent-messages.md).


## <a name="saving-state"></a>Enregistrement de l’état

Pour bien concevoir un robot, il est notamment essentiel de suivre le contexte de la conversation afin que le robot se souvienne de certains éléments comme la dernière question posée par l’utilisateur. Les bots créés à l’aide du kit SDK Bot Framework sont conçus pour fonctionner sans état. Il est ainsi facile de les mettre à l’échelle pour les exécuter sur plusieurs nœuds de calcul. Bot Framework offre un système de stockage qui enregistre les données du robot pour que son service web puisse être mis à l’échelle. Pour cette raison, il est déconseillé d’enregistrer un état à l’aide d’une variable globale ou d’une fermeture de fonction. Sinon, des problèmes peuvent se produire au moment de mettre à l’échelle votre robot. Utilisez plutôt les propriétés suivantes de l’objet [session][Session] de votre robot pour enregistrer les données relatives à un utilisateur ou à une conversation :

* **userData** enregistre de façon globale les informations de l’utilisateur dans toutes les conversations.
* **conversationData** enregistre de façon globale les informations d’une seule conversation. Ces données sont accessibles à tout le monde au cours de la conversation. Vous devez donc faire preuve de prudence lorsque vous enregistrez des données dans cette propriété. Elle est activée par défaut et vous pouvez la désactiver à l’aide du paramètre [persistConversationData][PersistConversationData] du robot.
* **privateConversationData** enregistre de façon globale les informations d’une seule conversation, mais il s’agit de données privées propres à l’utilisateur actuel. Ces données s’étendent à tous les dialogues. Il est donc utile d’enregistrer l’état temporaire que vous souhaitez effacer à la fin de la conversation.
* **dialogData** conserve les informations d’une seule instance de dialogue. Cette propriété est indispensable pour enregistrer des informations temporaires entre les différentes étapes d’une [cascade](bot-builder-nodejs-dialog-waterfall.md) au cours d’un dialogue.

Pour obtenir des exemples illustrant comment utiliser ces propriétés pour enregistrer et récupérer des données, consultez [Gérer les données d’état](bot-builder-nodejs-state.md).

## <a name="natural-language-understanding"></a>Compréhension du langage naturel

Bot Builder vous permet d’utiliser LUIS pour intégrer un module de compréhension du langage naturel à votre robot à l’aide de la classe [LuisRecognizer][LuisRecognizer]. Vous pouvez ajouter une instance d’un module **LuisRecognizer** qui renvoie au modèle de langue que vous avez publié, puis ajouter des gestionnaires pour lancer des actions en fonction des énoncés de l’utilisateur. Pour découvrir le fonctionnement du module LUIS, suivez le tutoriel suivant qui dure une dizaine de minutes :

* [Tutoriel Microsoft LUIS][LUISVideo] (vidéo)

## <a name="next-steps"></a>Étapes suivantes
> [!div class="nextstepaction"]
> [Vue d’ensemble des dialogues](bot-builder-nodejs-dialog-overview.md)



[PersistConversationData]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.iuniversalbotsettings.html#persistconversationdata
[UniversalBot]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.universalbot.html
[ChatConnector]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.chatconnector.html
[ConsoleConnector]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.consoleconnector.html

[ChannelInspector]: ../bot-service-channel-inspector.md

[Session]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session.html
[SessionSend]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session#send

[triggerAction]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.dialog.html#triggeraction
[waterfall]: bot-builder-nodejs-prompts.md

[RespondMessages]:bot-builder-nodejs-use-default-message-handler.md

[LUISRecognizer]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.luisrecognizer
[LUISVideo]: https://vimeo.com/145499419
