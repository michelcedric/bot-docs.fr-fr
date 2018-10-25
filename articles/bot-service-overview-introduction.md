---
title: Présentation d’Azure Bot Service | Microsoft Docs
description: Découvrez Bot Service, qui permet de créer, connecter, tester, déployer, surveiller et gérer des bots.
keywords: vue d’ensemble, introduction, SDK, présentation
author: Kaiqb
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 10/08/2018
ms.openlocfilehash: 3ca80439a44ac7e715d19f8e47683ac9b5a5721a
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2018
ms.locfileid: "49998876"
---
::: moniker range="azure-bot-service-3.0"

# <a name="about-azure-bot-service"></a>À propos d’Azure Bot Service

[!INCLUDE [pre-release-label](includes/pre-release-label-v3.md)]

Azure Bot Service fournit des outils pour créer, tester, déployer et gérer des bots intelligents, le tout en un même endroit. Dans le framework modulable et extensible fourni par le SDK, les développeurs peuvent utiliser des modèles pour créer des bots qui proposent des fonctionnalités de voix, de compréhension du langage naturel, de questions-réponses, et bien plus encore.  

## <a name="what-is-a-bot"></a>Qu’est-ce qu’un bot ?
Un bot est une application avec laquelle les utilisateurs interagissent par le biais d’une conversation textuelle, graphique (cartes) ou vocale. Il peut s’agir d’un simple dialogue de questions-réponses, ou d’un bot sophistiqué permettant aux utilisateurs d’interagir avec les services de manière intelligente, à l’aide de critères spéciaux, d’un suivi de l’état et de techniques d’intelligence artificielle bien intégrées aux services professionnels existants. 

## <a name="building-a-bot"></a>Création d’un bot 
Vous pouvez choisir d’utiliser votre environnement de développement ou vos outils en ligne de commande favoris pour créer votre bot en C# ou en Node.js. Nous fournissons des outils pour les différentes étapes du développement de bots, qui vous permettront de bien démarrer avec la création de bots.    

![Présentation des bots](media/bot-service-overview.png) 

## <a name="plan"></a>Planification 
Avant d’écrire votre code, consultez les [instructions relatives à la conception](bot-service-design-principles.md)  des bots, pour connaître les bonnes pratiques et déterminer les besoins de votre bot. Vous pouvez créer un bot simple ou inclure des fonctionnalités plus sophistiquées, telles que la voix, la compréhension du langage naturel, les questions-réponses, ou la capacité à extraire des informations de plusieurs sources pour fournir des réponses intelligentes.  

> [!TIP]
> Créez un compte [Azure](https://portal.azure.com). 

## <a name="build-your-bot"></a>Créer votre bot 
Votre bot est un service web qui implémente une interface de conversation et communique avec Bot Service. Vous pouvez créer cette solution dans autant d’environnements et de langages que vous le souhaitez. D’ailleurs, pour bien démarrer, nous vous proposons des outils faciles à utiliser pour Visual Studio et Yeoman. Vous pouvez également créer la solution directement dans le portail Azure. Vous trouverez ci-dessous certains des outils et services que vous pouvez utiliser.

> [!TIP]
> Créez un bot à l’aide du [portail Azure](bot-service-quickstart.md). Si nécessaire, ajoutez des composants, tels que : 
> - Language Understanding [LUIS](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/home). 
> - Base de connaissances [QnA Maker](https://docs.microsoft.com/en-us/azure/cognitive-services/qnamaker/home) pour répondre aux questions posées par les utilisateurs.  

## <a name="test-your-bot"></a>Tester votre robot 
Les bots sont des applications complexes composées de nombreux éléments différents qui fonctionnent ensemble. Comme pour toutes les applications complexes, des bogues intéressants ou des comportements inattendus peuvent se produire. Avant de le publier, testez votre bot.

> [!TIP] 
> - Tester le bot dans [Web Chat](bot-service-manage-test-webchat.md) ou tester le bot localement avec l’[émulateur](bot-service-debug-emulator.md)

## <a name="publish"></a>Publish 
Lorsque vous êtes prêt, publiez votre bot dans Azure ou dans votre propre centre de données ou service web. Vous pouvez configurer un déploiement continu pour développer votre bot localement. Un tel déploiement est utile si votre bot est archivé dans un contrôle de code source comme GitHub ou Visual Studio Team Services. Quand vous archivez vos modifications dans votre référentiel source, elles sont automatiquement déployées dans Azure.

> [!Tip]
> - [Télécharger et redéployer le code dans Azure](bot-service-build-download-source-code.md)

## <a name="connect"></a>Connecter          
Connecter votre bot aux canaux tels que Facebook, Messenger, Kik, Skype, Slack, Microsoft Teams, Telegram, SMS, Twilio, Cortana et Skype, pour augmenter les interactions et atteindre davantage de clients.  
  
> [!TIP]
> - [Choisir les canaux à ajouter](bot-service-manage-channels.md)


## <a name="evaluate"></a>Évaluer 
Utilisez les données collectées dans le portail Azure pour identifier les opportunités d’améliorer les fonctionnalités et les performances de votre bot. Vous pouvez obtenir des données relatives au service et à l’instrumentation, comme le trafic, la latence et les intégrations. L’outil d’analyse permet également de créer des rapports de conversation à partir des données sur les utilisateurs, les messages et les canaux.

> [!Tip]
> - [Collecter des données d’analyse](bot-service-manage-analytics.md) 


::: moniker-end

::: moniker range="azure-bot-service-4.0"

# <a name="about-azure-bot-service"></a>À propos d’Azure Bot Service

[!INCLUDE [pre-release-label](includes/pre-release-label.md)]

Azure Bot Service fournit des outils pour créer, tester, déployer et gérer des bots intelligents, le tout en un même endroit. L’infrastructure modulable et extensible fournie par le Kit de développement logiciel (SDK), les outils, les modèles et les services d’intelligence artificielle permettent aux développeurs de créer des bots en mesure d’offrir des fonctionnalités vocales, de comprendre le langage naturel, de traiter les questions-réponses, et bien davantage.

## <a name="what-is-a-bot"></a>Qu’est-ce qu’un bot ?
Les bots offrent une expérience qui donne l’impression aux utilisateurs d’avoir moins à faire à un ordinateur, et plus à une personne, ou du moins à un robot intelligent. Les bots peuvent être utilisés pour faire passer des tâches répétitives et simples, comme la réservation de table ou le recueil d’informations de profil, vers des systèmes automatisés qui ne nécessitent plus d’intervention humaine directe. Les utilisateurs conversent avec le bot à l’aide de texte, de cartes interactives et avec la voix. Une interaction avec un bot peut se composer d’une question-réponse rapide, ou il peut s’agir d’une conversation plus sophistiquée fournissant un accès à des services de manière plus intelligente.

Les bots sont comparables à des applications web modernes. Ils résident sur Internet et utilisent des API pour envoyer et recevoir des messages. Le contenu d’un bot varie considérablement selon son type. Les logiciels de bots modernes s’appuient sur une pile de technologies et d’outils qui offrent des expériences de plus en plus complexes sur un large éventail de plateformes. Toutefois, un bot simple peut se contenter de recevoir un message et de répondre à l’utilisateur avec très peu de code. 

Les bots peuvent effectuer les mêmes opérations que d’autres types de logiciels : lire et écrire des fichiers, utiliser des bases de données et des API, et réaliser des tâches de calcul classiques. Ce qui rend les bots uniques, c’est leur utilisation de mécanismes généralement réservés à la communication entre humains. 

Les bots comprennent généralement les composants suivants :
* Un serveur web, dans la plupart des cas disponible sur l’Internet public
* Le SDK et les outils Bot Builder qui fournissent une interface pour le développement de bots
* Azure Cognitive Service 
* Stockage Azure

## <a name="building-a-bot"></a>Création d’un bot 

Azure Bot Service propose un ensemble intégré d’outils et de services permettant de faciliter ce processus. Choisissez votre environnement de développement ou vos outils en ligne de commande favoris pour créer votre bot en C#, en JavaScript ou en Typescript. (Java et Python prochainement) Nous fournissons des outils pour les différentes étapes du développement de bots, qui vous permettront de bien démarrer avec la création de bots.

![Présentation des bots](media/bot-service-overview.png) 

### <a name="plan"></a>Planification
Comme avec n’importe quel type de logiciel, il est important pour le processus de création d’un bot réussi de disposer d’une compréhension approfondie des objectifs, des processus et des besoins des utilisateurs. Avant d’écrire votre code, consultez les [instructions relatives à la conception](bot-service-design-principles.md)  des bots, pour connaître les bonnes pratiques et déterminer les besoins de votre bot. Vous pouvez créer un bot simple ou inclure des fonctionnalités plus élaborées, telles que des fonctions vocales, la compréhension du langage naturel ou la capacité de répondre aux questions.

### <a name="build"></a>Créer
Votre bot est un service web qui implémente une interface de conversation et communique avec Bot Framework Service pour envoyer et recevoir des messages et des événements. Vous pouvez créer des bots dans divers environnements et langages. Vous pouvez commencer le développement de votre bot dans le [portail Azure](bot-service-quickstart.md), ou utiliser des modèles [[C#](dotnet/bot-builder-dotnet-sdk-quickstart.md) | [JavaScript](javascript/bot-builder-javascript-quickstart.md)] pour un développement local.

Dans le cadre d’Azure Bot Service, nous proposons des composants supplémentaires que vous pouvez utiliser pour étendre les fonctionnalités de votre bot

| Fonctionnalité | Description | Lien |
| --- | --- | --- |
| Ajouter un traitement en langage naturel | Activez votre bot pour la compréhension du langage naturel, la détection des fautes d’orthographe, l’utilisation de la voix et la reconnaissance de l’intention de l’utilisateur | Utilisation de [LUIS](~/v4sdk/bot-builder-howto-v4-luis.md) 
| Répondre aux questions | Ajoutez une base de connaissances pour répondre aux questions des utilisateurs de manière plus naturelle et conversationnelle | Utilisation de [QnA Maker](~/v4sdk/bot-builder-howto-qna.md) 
| Gérer plusieurs modèles | Si vous utilisez plusieurs modèles, tels que LUIS et QnA Maker, déterminez de manière judicieuse le modèle qu’il convient d’utiliser au cours de la conversation avec votre bot | Outil [Dispatch](~/v4sdk/bot-builder-tutorial-dispatch.md)|
| Ajouter des cartes et des boutons | Améliorez l’expérience utilisateur à l’aide de médias autres que du texte, tels que des graphiques, des menus et des cartes | [Ajout de cartes](v4sdk/bot-builder-howto-add-media-attachments.md) |

> [!NOTE]
> Le tableau ci-dessus n’est pas exhaustif. Pour plus d’informations sur les fonctionnalités des bots, explorez les articles sur la gauche, en commençant par [Envoi de messages](~/v4sdk/bot-builder-howto-send-messages.md).

En outre, nous fournissons des outils en ligne de commande pour vous aider à créer, gérer et tester les ressources de bot. Ces outils peuvent gérer un fichier de configuration de bot, configurer des applications LUIS, créer une base de connaissances QnA, simuler une conversation, et bien davantage. Vous trouverez plus d’informations dans le fichier [readme](https://aka.ms/botbuilder-tools-readme) des outils en ligne de commande.

Vous avez également accès à un large éventail d’[exemples](https://github.com/microsoft/botbuilder-samples) qui présentent un bon nombre de fonctionnalités disponibles avec le Kit de développement logiciel (SDK). Ils sont intéressants pour les développeurs qui souhaitent démarrer avec un plus grand ensemble de fonctionnalités.

### <a name="test"></a>Test 
Les bots sont des applications complexes composées de nombreux éléments différents qui fonctionnent ensemble. Comme pour toutes les applications complexes, des bogues intéressants ou des comportements inattendus peuvent se produire. Avant de le publier, testez votre bot. Nous proposons plusieurs moyens de tester les bots avant leur publication :

- Testez votre bot en local avec l’[émulateur](bot-service-debug-emulator.md). L’émulateur Bot Framework Bot est une application autonome qui fournit non seulement une interface de conversation, mais également des outils de débogage et d’interrogation qui permettent de comprendre le comportement de votre bot.  L’émulateur peut être exécuté localement en même temps que votre application de bot en développement. 
 
- Testez votre bot sur le [Web](bot-service-manage-test-webchat.md). Une fois configuré via le portail Azure, votre bot peut également être accessible via une interface de conversation web. L’interface de conversation web constitue un excellent moyen d’accorder l’accès à votre bot à des testeurs et à d’autres personnes ne disposant pas d’un accès direct au code d’exécution du bot.

### <a name="publish"></a>Publish 
Lorsque vous êtes prêt à rendre votre bot accessible sur le Web, publiez-le dans [Azure](bot-builder-howto-deploy-azure.md) ou dans votre propre centre de données ou service web. La possession d’une adresse sur l’Internet public est la première étape pour que votre bot prenne vie sur votre site, ou dans des canaux de conversation.

### <a name="connect"></a>Connecter          
Connectez votre bot aux canaux tels que Facebook, Messenger, Kik, Skype, Slack, Microsoft Teams, Telegram, SMS, Twilio, Cortana et Skype. Bot Framework effectue la plupart des opérations nécessaires pour envoyer et recevoir des messages de l’ensemble de ces différentes plateformes : l’application de votre bot reçoit un flux unifié et normalisé de messages, quels que soient le nombre et le type de canaux auxquels il est connecté. Pour plus d’informations sur l’ajout de canaux, consultez la rubrique [Canaux](bot-service-manage-channels.md).

### <a name="evaluate"></a>Évaluer 
Utilisez les données collectées dans le portail Azure pour identifier les opportunités d’améliorer les fonctionnalités et les performances de votre bot. Vous pouvez obtenir des données relatives au service et à l’instrumentation, comme le trafic, la latence et les intégrations. L’outil d’analyse permet également de créer des rapports de conversation à partir des données sur les utilisateurs, les messages et les canaux. Pour plus d’informations, consultez la rubrique expliquant [comment rassembler les données d’analyse](bot-service-manage-analytics.md).


## <a name="next-steps"></a>Étapes suivantes
Découvrez des [études de cas](https://azure.microsoft.com/services/bot-service/) sur des bots ou cliquez sur le lien ci-dessous pour créer un bot.
> [!div class="nextstepaction"]
> [Créer un robot](bot-service-quickstart.md)

::: moniker-end
