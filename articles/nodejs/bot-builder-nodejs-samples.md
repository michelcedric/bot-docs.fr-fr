---
title: Exemples de robots pour le Kit de développement logiciel (SDK) Bot Builder pour Node.js | Microsoft Docs
description: Explorez une vaste sélection d’exemples de robots qui vous aideront à commencer à développer vos robots avec le Kit de développement logiciel (SDK) Bot Builder pour Node.js.
author: v-ducvo
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: f22e9e6c27b36955aa09953b7fc5d67f97674181
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2018
ms.locfileid: "49997756"
---
# <a name="bot-builder-sdk-for-nodejs-samples"></a>Exemples du Kit de développement logiciel (SDK) Bot Builder pour Node.js

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

Ces exemples montrent comment des robots centrés sur les tâches vous aident à tirer parti des fonctionnalités du Kit de développement logiciel (SDK) Bot Builder pour Node.js. Vous pouvez utiliser les exemples pour commencer rapidement à concevoir de puissants robots aux fonctionnalités enrichies.

## <a name="get-the-samples"></a>Obtenir les exemples
Pour obtenir les exemples, clonez le dépôt GitHub [BotBuilder-Samples](https://github.com/Microsoft/BotBuilder-Samples) à l’aide de Git.

```cmd
git clone https://github.com/Microsoft/BotBuilder-Samples.git
cd BotBuilder-Samples
```

Les exemples de robots générés avec le Kit de développement logiciel (SDK) Bot Builder Node.js sont organisés dans le répertoire **Node**.

Vous pouvez également afficher les exemples sur GitHub et les déployer directement sur Azure.

## <a name="core"></a>Principal
Ces exemples montrent les techniques de base pour créer des robots complets et efficaces.

Exemple | Description
------------ | ------------- 
[Envoyer une pièce jointe](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/core-SendAttachment) | Un exemple de bot qui envoie de simples pièces jointes multimédia (images) à l’utilisateur. 
[Recevoir une pièce jointe](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/core-ReceiveAttachment) | Exemple de bot qui reçoit les pièces jointes envoyées par l’utilisateur et les télécharge. 
[Créer une conversation](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/core-CreateNewConversation)  | Exemple de bot qui entame une conversation à l’aide d’une adresse d’utilisateur précédemment stockée.
[Obtenir les membres d’une conversation](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/core-GetConversationMembers) | Exemple de bot qui récupère la liste des membres de la conversation et détecte quand celle-ci change. 
[Direct Line](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/core-DirectLine) | Exemple de bot et de client personnalisé qui communiquent entre eux à l’aide de l’API Direct Line. 
[Direct Line (WebSockets)](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/core-DirectLineWebSockets) | Exemple de bot et de client personnalisé qui communiquent entre eux à l’aide de l’API Direct Line + WebSockets. 
[Plusieurs dialogues](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/core-MultiDialogs) | Exemple de robot qui affiche les différents types de dialogues.
[API State](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/core-State) | Exemple de bot sans état qui assure le suivi du contexte d’une conversation.
[API Custom State](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/core-CustomState) | Exemple de robot sans état qui assure le suivi du contexte d’une conversation à l’aide d’un fournisseur de stockage personnalisé.
[ChannelData](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/core-ChannelData) | Exemple de robot qui envoie des métadonnées natives à Facebook à l’aide de ChannelData.
[AppInsights](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/core-AppInsights) | Exemple de robot qui enregistre une télémétrie dans une instance Application Insights.

## <a name="search"></a>Recherche
Cet exemple montre comment tirer parti de Recherche Azure dans des robots pilotés par les données.

Exemple | Description
------------ | -------------
[Azure Search](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/demo-Search) | Deux exemples de robots qui aident l’utilisateur à accédez à de grandes quantités de contenu.


## <a name="cards"></a>Cartes
Ces exemples montrent comment envoyer des cartes riches dans Bot Framework.

Exemple | Description
------------ | -------------
[Cartes riches](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/cards-RichCards) | Exemple de robot qui envoie différents types de cartes riches.
[Carrousel de cartes](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/cards-CarouselCards) | Exemple de bot qui envoie plusieurs cartes riches dans un seul message en utilisant la mise en page du carrousel.

## <a name="intelligence"></a>Intelligence
Ces exemples montrent comment ajouter des fonctionnalités d’intelligence artificielle à un bot à l’aide d’API Bing et Microsoft Cognitive Services.

Exemple | Description
------------ | -------------
[LUIS](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/intelligence-LUIS) | Exemple de bot qui utilise LuisDialog pour intégrer avec une application LUIS.ai.
[Légende d’image](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/intelligence-ImageCaption) | Exemple de bot qui obtient une légende d’image à l’aide de l’API Vision de Microsoft Cognitive Services.
[Reconnaissance vocale](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/intelligence-SpeechToText)  | Exemple de bot qui obtient le texte de l’audio à l’aide de l’API Reconnaissance vocale Bing.
[Produits similaires](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/intelligence-SimilarProducts) | Exemple de bot qui recherche visuellement des produits similaires à l’aide de l’API Recherche d’images Bing. 
[Zummer](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/intelligence-Zummer) | Exemple de bot qui recherche des articles de Wikipédia à l’aide de l’API Recherche Bing.

## <a name="reference-implementation"></a>Implémentation de référence
Cet exemple est conçu pour illustrer un scénario de bout en bout. Il s’agit d’une source précieuse de fragments de code si vous cherchez à implémenter des fonctionnalités plus complexes dans votre robot.


Exemple | Description
------------ | -------------
[Contoso Flowers](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/demo-ContosoFlowers) | Exemple de robot qui utilise de nombreuses fonctionnalités de Bot Framework.

