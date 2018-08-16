---
title: À propos de Bot Service | Microsoft Docs
description: Découvrez Bot Service, qui permet de créer, connecter, tester, déployer, surveiller et gérer des bots.
keywords: vue d’ensemble, introduction, SDK, présentation
author: Kaiqb
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 05/03/2018
ms.openlocfilehash: b6326ac152112ff1df01470db1f525d4bf241af4
ms.sourcegitcommit: 67445b42796d90661afc643c6bb6533e9a662cbc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/06/2018
ms.locfileid: "39574605"
---
::: moniker range="azure-bot-service-3.0"

# <a name="azure-bot-service"></a>Azure Bot Service

[!INCLUDE [pre-release-label](includes/pre-release-label-v3.md)]

Azure Bot Service fournit des outils pour créer, tester, déployer et gérer des bots intelligents, le tout en un même endroit. Dans le framework modulable et extensible fourni par le SDK, les développeurs peuvent utiliser des modèles pour créer des bots qui proposent des fonctionnalités de voix, de compréhension du langage naturel, de questions-réponses, et bien plus encore.  

## <a name="what-is-a-bot"></a>Qu’est-ce qu’un bot ?
Un bot est une application avec laquelle les utilisateurs interagissent par le biais d’une conversation textuelle, graphique (cartes) ou vocale. Il peut s’agir d’un simple dialogue de questions-réponses, ou d’un bot sophistiqué permettant aux utilisateurs d’interagir avec les services de manière intelligente, à l’aide de critères spéciaux, d’un suivi de l’état et de techniques d’intelligence artificielle bien intégrées aux services professionnels existants. Découvrez des [études de cas](https://azure.microsoft.com/services/bot-service/) sur les bots.  

## <a name="building-a-bot"></a>Création d’un bot 
Vous pouvez choisir d’utiliser votre environnement de développement ou vos outils en ligne de commande favoris pour créer votre bot en C# ou en Node.js. Nous fournissons des outils pour les différentes étapes du développement de bots, qui vous permettront de bien démarrer avec la création de bots.    

![Présentation des bots](media/bot-service-overview.png) 

## <a name="plan"></a>Planification 
Avant d’écrire votre code, consultez les [instructions relatives à la conception](bot-service-design-principles.md)  des bots, pour connaître les bonnes pratiques et déterminer les besoins de votre bot. Vous pouvez créer un bot simple ou inclure des fonctionnalités plus sophistiquées, telles que la voix, la compréhension du langage naturel, les questions-réponses, ou la capacité à extraire des informations de plusieurs sources pour fournir des réponses intelligentes.  

> [!TIP] 
>
> Installer le modèle dont vous avez besoin :
>  - [Modèle VSIX](https://marketplace.visualstudio.com/items?itemName=BotBuilder.BotBuilderV3) du SDK .NET Bot Builder v3 
>  - [Modèle Yeoman](https://www.npmjs.com/package/generator-botbuilder) du SDK Node.js Bot Builder v3 
>
> Installez les outils :
> - Téléchargez les [outils CLI](https://github.com/Microsoft/botbuilder-tools) pour créer et gérer les ressources du bot. Ces outils vous permettent de gérer le fichier de configuration du bot, l’application LUIS, la base de connaissances QnA, etc. à partir de la ligne de commande. Vous trouverez plus d’informations dans le fichier [Lisez-moi](https://github.com/Microsoft/botbuilder-tools/blob/master/README.md).
> - [Emulator](https://github.com/Microsoft/BotFramework-Emulator/releases) pour tester votre bot
>
> Si nécessaire, utilisez les composants de bot, tels que :  
> - [LUIS](https://www.luis.ai/) pour ajouter la compréhension du langage naturel
> - [QnA Maker](https://qnamaker.ai/) pour répondre aux questions de l’utilisateur de manière plus naturelle, comme dans une conversation
> - [Speech](https://azure.microsoft.com/services/cognitive-services/speech/) pour convertir de l’audio en texte, comprendre l’intention et reconvertir le texte en audio  
> - [Spelling](https://azure.microsoft.com/services/cognitive-services/spell-check/) pour corriger les fautes d’orthographe, faire la différence entre les noms, les noms de marques et l’argot 
> - [Cognitive Services](bot-service-concept-intelligence.md) pour tirer parti d’autres composants intelligents 


## <a name="build-your-bot"></a>Créer votre bot 
Votre bot est un service web qui implémente une interface de conversation et communique avec Bot Service. Vous pouvez créer cette solution dans autant d’environnements et de langages que vous le souhaitez. D’ailleurs, pour bien démarrer, nous vous proposons des outils faciles à utiliser pour Visual Studio et Yeoman. Vous pouvez également créer la solution directement dans le portail Azure. Vous trouverez ci-dessous certains des outils et services que vous pouvez utiliser.

> [!TIP]
>
> Vous pouvez créer un bot à l’aide du [SDK](~/dotnet/bot-builder-dotnet-quickstart.md), du [portail Azure](bot-service-quickstart.md) ou des [outils CLI](~/bot-builder-create-templates.md).
>
> Ajoutez des composants : 
> - Ajoutez le modèle de compréhension du langage naturel [LUIS](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/home). 
> - Ajoutez la base de connaissances [QnA Maker](https://docs.microsoft.com/en-us/azure/cognitive-services/qnamaker/home) pour répondre aux questions posées par les utilisateurs.  
> - Améliorez l’expérience utilisateur avec les cartes, la reconnaissance vocale et la traduction. 
> - Ajoutez une logique à votre bot à l’aide du SDK Bot Builder.   

## <a name="test-your-bot"></a>Tester votre bot 
Les bots sont des applications complexes composées de nombreux éléments différents qui fonctionnent ensemble. Comme pour toutes les applications complexes, des bogues intéressants ou des comportements inattendus peuvent se produire. Avant de le publier, testez votre bot.

> [!TIP] 
>
> - [Tester le bot avec l’émulateur](bot-service-debug-emulator.md)
> - [Tester le bot dans Web Chat](bot-service-manage-test-webchat.md)

## <a name="publish"></a>Publish 
Lorsque vous êtes prêt, publiez votre bot dans Azure ou dans votre propre centre de données ou service web. Vous pouvez configurer un déploiement continu pour développer votre bot localement. Un tel déploiement est utile si votre bot est archivé dans un contrôle de code source comme GitHub ou Visual Studio Team Services. Quand vous archivez vos modifications dans votre référentiel source, vos modifications sont automatiquement déployées dans Azure.

> [!Tip]
>
> - [Déployer dans Azure](bot-service-build-continuous-deployment.md)

## <a name="connect"></a>Connecter          
Connecter votre bot aux canaux tels que Facebook, Messenger, Kik, Skype, Slack, Microsoft Teams, Telegram, SMS, Twilio, Cortana et Skype, pour augmenter les interactions et atteindre davantage de clients.  
  
> [!TIP]
>
> - [Choisir les canaux à ajouter](bot-service-manage-channels.md)


## <a name="evaluate"></a>Évaluer 
Utilisez les données collectées dans le portail Azure pour identifier les opportunités d’améliorer les fonctionnalités et les performances de votre bot. Vous pouvez obtenir des données relatives au service et à l’instrumentation, comme le trafic, la latence et les intégrations. L’outil d’analyse permet également de créer des rapports de conversation à partir des données sur les utilisateurs, les messages et les canaux.

> [!Tip]
>
> - [Collecter des données d’analyse](bot-service-manage-analytics.md) 


::: moniker-end

::: moniker range="azure-bot-service-4.0"

# <a name="azure-bot-service"></a>Azure Bot Service

[!INCLUDE [pre-release-label](includes/pre-release-label.md)]

Azure Bot Service fournit des outils pour créer, tester, déployer et gérer des bots intelligents, le tout en un même endroit. Dans le framework modulable et extensible fourni par le SDK, les développeurs peuvent utiliser des modèles pour créer des bots qui proposent des fonctionnalités de voix, de compréhension du langage naturel, de questions-réponses, et bien plus encore.  

## <a name="what-is-a-bot"></a>Qu’est-ce qu’un bot ?
Un bot est une application avec laquelle les utilisateurs interagissent par le biais d’une conversation textuelle, graphique (cartes) ou vocale. Il peut s’agir d’un simple dialogue de questions-réponses, ou d’un bot sophistiqué permettant aux utilisateurs d’interagir avec les services de manière intelligente, à l’aide de critères spéciaux, d’un suivi de l’état et de techniques d’intelligence artificielle bien intégrées aux services professionnels existants. Découvrez des [études de cas](https://azure.microsoft.com/services/bot-service/) sur les bots.  

## <a name="building-a-bot"></a>Création d’un bot 
Vous pouvez choisir d’utiliser votre environnement de développement ou vos outils en ligne de commande favoris pour créer votre bot en C#, JavaScript, Java ou Python. Nous fournissons des outils pour les différentes étapes du développement de bots, qui vous permettront de bien démarrer avec la création de bots.    

![Présentation des bots](media/bot-service-overview.png) 

## <a name="plan"></a>Planification 
Avant d’écrire votre code, consultez les [instructions relatives à la conception](bot-service-design-principles.md)  des bots, pour connaître les bonnes pratiques et déterminer les besoins de votre bot. Vous pouvez créer un bot simple ou inclure des fonctionnalités plus sophistiquées, telles que la voix, la compréhension du langage naturel, les questions-réponses, ou la capacité à extraire des informations de plusieurs sources pour fournir des réponses intelligentes. Pour commencer, vous pouvez utiliser les [outils CLI](~/bot-builder-create-templates.md), le [portail Azure](bot-service-quickstart.md), ou les modèles ci-dessous.

**Choisissez le modèle dont vous avez besoin**

| .NET | JavaScript | 
| --- | --- | 
| [Modèle VSIX](https://marketplace.visualstudio.com/items?itemName=BotBuilder.botbuilderv4) | [Modèle Yeoman](https://www.npmjs.com/package/generator-botbuilder). Utilisez @preview pour obtenir le modèle v4. |

**Explorez ou installez les outils, si nécessaire**

- Téléchargez les [outils CLI](https://github.com/Microsoft/botbuilder-tools) pour créer et gérer les ressources du bot. Ces outils vous permettent de gérer le fichier de configuration du bot, l’application LUIS, la base de connaissances QnA, etc. à partir de la ligne de commande. Vous trouverez plus d’informations dans le fichier [Lisez-moi](https://github.com/Microsoft/botbuilder-tools/blob/master/README.md).
- [Emulator](https://github.com/Microsoft/BotFramework-Emulator/releases) pour tester votre bot
- [LUIS](https://www.luis.ai/) pour ajouter le langage naturel aux bots
- [QnA Maker](https://qnamaker.ai/) pour répondre aux questions de l’utilisateur de manière plus naturelle, comme dans une conversation


## <a name="build-your-bot"></a>Créer votre bot 
Votre bot est un service web qui implémente une interface de conversation et communique avec Bot Service. Vous pouvez créer cette solution dans autant d’environnements et de langages que vous le souhaitez. D’ailleurs, pour bien démarrer, nous vous proposons des outils faciles à utiliser pour Visual Studio et Yeoman. Vous pouvez également créer la solution directement dans le portail Azure. Vous trouverez ci-dessous certains des outils et services que vous pouvez utiliser.

Voici certains des nombreux composants disponibles :

| Composants | Description |
| --- | --- |
| [LUIS](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/home) | Pour ajouter la reconnaissance vocale |
| [QnA Maker](https://docs.microsoft.com/en-us/azure/cognitive-services/qnamaker/home)  | Pour ajouter la base de connaissances en vue de répondre aux questions posées par les utilisateurs |
| [Outil Répartition](~/v4sdk/bot-builder-tutorial-dispatch.md) | Si vous utilisez plusieurs modèles, il détermine intelligemment à quel moment utiliser chacun d’eux |
| [Multimédia enrichi](v4sdk/bot-builder-howto-add-media-attachments.md) | Pour améliorer l’expérience utilisateur avec des cartes multimédias ou la voix |
| [Speech](https://azure.microsoft.com/services/cognitive-services/speech/) | Pour convertir de l’audio en texte, comprendre l’intention et reconvertir le texte en audio |
| [Spelling](https://azure.microsoft.com/services/cognitive-services/spell-check/) | Pour corriger les fautes d’orthographe, faire la différence entre les noms, les noms de marques et l’argot |

> [!NOTE]
> Le tableau ci-dessus n’est pas exhaustif. Pour plus d’informations sur les fonctionnalités des bots, explorez les articles sur la gauche, en commençant par [Envoi de messages](v4sdk/bot-builder-howto-send-messages.md).

## <a name="test-your-bot"></a>Tester votre bot 
Les bots sont des applications complexes composées de nombreux éléments différents qui fonctionnent ensemble. Comme pour toutes les applications complexes, des bogues intéressants ou des comportements inattendus peuvent se produire. Avant de le publier, testez votre bot. 

[Testez le bot avec l’émulateur](bot-service-debug-emulator.md) ou [testez le bot dans Web Chat](bot-service-manage-test-webchat.md).

## <a name="publish"></a>Publish 
Lorsque vous êtes prêt, publiez votre bot dans Azure ou dans votre propre centre de données ou service web. Vous pouvez configurer un déploiement continu pour développer votre bot localement. Un tel déploiement est utile si votre bot est archivé dans un contrôle de code source comme GitHub ou Visual Studio Team Services. Quand vous archivez vos modifications dans votre référentiel source, vos modifications sont automatiquement déployées dans Azure.

[Déployez dans Azure](bot-service-build-continuous-deployment.md) à l’aide d’un déploiement continu.

## <a name="connect"></a>Connecter          
Connecter votre bot aux canaux tels que Facebook, Messenger, Kik, Skype, Slack, Microsoft Teams, Telegram, SMS, Twilio, Cortana et Skype, pour augmenter les interactions et atteindre davantage de clients.

[Choisissez les canaux à ajouter](bot-service-manage-channels.md).


## <a name="evaluate"></a>Évaluer 
Utilisez les données collectées dans le portail Azure pour identifier les opportunités d’améliorer les fonctionnalités et les performances de votre bot. Vous pouvez obtenir des données relatives au service et à l’instrumentation, comme le trafic, la latence et les intégrations. L’outil d’analyse permet également de créer des rapports de conversation à partir des données sur les utilisateurs, les messages et les canaux. 

Découvrez comment [Collecter des données d’analyse](bot-service-manage-analytics.md).

::: moniker-end
