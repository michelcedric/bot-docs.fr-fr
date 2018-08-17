---
title: Exemples de bots pour le Kit SDK Bot Builder pour .NET | Microsoft Docs
description: Découvrez des exemples de bots qui vous aideront à commencer à développer vos bots avec le Kit SDK Bot Builder pour .NET.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 05/03/2018
ms.openlocfilehash: 7e19ec2c3e523003c0831544e42eeb88765d8317
ms.sourcegitcommit: dcbc8ad992a3e242a11ebcdf0ee99714d919a877
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/30/2018
ms.locfileid: "39352858"
---
::: moniker range="azure-bot-service-3.0"

# <a name="bot-builder-sdk-for-net-samples"></a>Exemples de kit SDK Bot Builder pour .NET

Ces exemples montrent comment les bots centrés sur les tâches vous aident à tirer parti des fonctionnalités du Kit SDK Bot Builder pour .NET. Vous pouvez utiliser les exemples pour commencer rapidement à concevoir de puissants bots aux fonctionnalités enrichies.

## <a name="get-the-samples"></a>Obtenir les exemples
Pour obtenir les exemples, clonez le référentiel GitHub [BotBuilder-Samples](https://github.com/Microsoft/BotBuilder-Samples) à l’aide de Git.

```cmd
git clone https://github.com/Microsoft/BotBuilder-Samples.git
cd BotBuilder-Samples
```

Les exemples de bots générés avec le Kit SDK Bot Builder pour .NET sont organisés dans le répertoire **CSharp**.

Vous pouvez également afficher les exemples sur GitHub et les déployer directement dans Azure.

## <a name="core"></a>Principal
Ces exemples montrent les techniques de base pour créer des bots complets et efficaces.

Exemple | Description
------------ | ------------- 
[Send Attachment](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-SendAttachment) | Un exemple de bot qui envoie de simples pièces jointes multimédia (images) à l’utilisateur. 
[Receive Attachment](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-ReceiveAttachment) | Exemple de bot qui reçoit les pièces jointes envoyées par l’utilisateur et les télécharge. 
[Create New Conversation](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-CreateNewConversation)  | Exemple de bot qui entame une conversation à l’aide d’une adresse d’utilisateur précédemment stockée.
[Get Members of a Conversation](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-GetConversationMembers) | Exemple de bot qui récupère la liste des membres de la conversation et détecte quand celle-ci change. 
[Direct Line](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-DirectLine) | Exemple de bot et de client personnalisé qui communiquent entre eux à l’aide de l’API Direct Line. 
[Direct Line (WebSockets)](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-DirectLineWebSockets) | Exemple de bot et de client personnalisé qui communiquent entre eux à l’aide de l’API Direct Line + WebSockets. 
[Multi Dialogs](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-MultiDialogs) | Exemple de robot qui affiche les différents types de dialogues.
[State API](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-State) | Exemple de bot sans état qui assure le suivi du contexte d’une conversation.
[Custom State API](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-CustomState) | Exemple de bot sans état qui assure le suivi du contexte d’une conversation à l’aide d’un fournisseur de stockage personnalisé.
[ChannelData](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-ChannelData) | Exemple de bot qui envoie des métadonnées natives à Facebook à l’aide de ChannelData.
[AppInsights](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-AppInsights) | Exemple de bot qui enregistre une télémétrie dans une instance Application Insights.

## <a name="search"></a>Recherche
Cet exemple montre comment tirer parti de Recherche Azure dans des bots pilotés par les données.

Exemple | Description
------------ | -------------
[Recherche Azure](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/demo-Search) | Deux exemples de bots qui permettent à l’utilisateur de parcourir de grandes quantités de contenu.


## <a name="cards"></a>Cartes
Ces exemples montrent comment envoyer des cartes riches dans Bot Framework.

Exemple | Description
------------ | -------------
[Rich Cards](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/cards-RichCards) | Exemple de bot qui envoie plusieurs types de cartes riches.
[Carousel of Cards](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/cards-CarouselCards) | Exemple de bot qui envoie plusieurs cartes riches dans un seul message en utilisant la mise en page du carrousel.

## <a name="intelligence"></a>Intelligence
Ces exemples montrent comment ajouter des fonctionnalités d’intelligence artificielle à un bot à l’aide d’API Bing et Microsoft Cognitive Services.

Exemple | Description
------------ | -------------
[LUIS](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/intelligence-LUIS) | Exemple de bot qui utilise LuisDialog pour intégrer avec une application LUIS.ai.
[Image Caption](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/intelligence-ImageCaption) | Exemple de bot qui obtient une légende d’image à l’aide de l’API Vision de Microsoft Cognitive Services.
[Speech To Text](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/intelligence-SpeechToText)  | Exemple de bot qui obtient le texte de l’audio à l’aide de l’API Reconnaissance vocale Bing.
[Similar Products](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/intelligence-SimilarProducts) | Exemple de bot qui recherche visuellement des produits similaires à l’aide de l’API Recherche d’images Bing. 
[Zummer](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/intelligence-Zummer) | Exemple de bot qui recherche des articles de Wikipédia à l’aide de l’API Recherche Bing.

## <a name="reference-implementation"></a>Implémentation de référence
Cet exemple est conçu pour illustrer un scénario de bout en bout. Il s’agit d’une source précieuse de fragments de code si vous cherchez à implémenter des fonctionnalités plus complexes dans votre bot.


Exemple | Description
------------ | -------------
[Contoso Flowers](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/demo-ContosoFlowers) | Exemple de bot qui utilise de nombreuses fonctionnalités de Bot Framework.

::: moniker-end

::: moniker range="azure-bot-service-4.0"
# <a name="bot-builder-sdk-v4-net-samples"></a>Exemples de kit SDK Bot Builder pour .NET v4
[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Ces exemples montrent comment les bots centrés sur les tâches vous aident à tirer parti des fonctionnalités du Kit SDK Bot Builder pour .NET. Vous pouvez utiliser les exemples pour commencer rapidement à concevoir de puissants bots aux fonctionnalités enrichies. 

Remarque : Le Kit SDK v4 est activement développé et doit donc être utilisé pour l’expérimentation uniquement. 

Pour obtenir les exemples, clonez le référentiel GitHub [botbuilder-dotnet](https://github.com/Microsoft/botbuilder-dotnet) à l’aide de Git.
```cmd
git clone https://github.com/Microsoft/botbuilder-dotnet.git
cd botbuilder-dotnet\samples-final
```
Les exemples de bots générés avec le Kit SDK Bot Builder pour .NET sont organisés dans le répertoire **samples-final**.


::: moniker-end

