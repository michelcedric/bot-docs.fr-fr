---
title: Gérer les ressources de bot avec un fichier de bot | Microsoft Docs
description: Décrit l’objectif et l’utilisation du fichier de bot.
keywords: fichier de bot, .bot, fichier .bot, msbot, ressources de bot, gérer les ressources de bot
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 11/13/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 0ff3f0e68d58a8768bb785a88ee7664ab430e453
ms.sourcegitcommit: 8b7bdbcbb01054f6aeb80d4a65b29177b30e1c20
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/14/2018
ms.locfileid: "51645629"
---
# <a name="manage-bot-resources-with-a-bot-file"></a>Gérer les ressources de bot avec un fichier de bot

Généralement, les bots utilisent de nombreux services différents tels que [LUIS.ai](https://luis.ai) ou [QnaMaker.ai](https://qnamaker.ai). Lorsque vous développez un bot, il n’y a aucun emplacement uniforme pour stocker les métadonnées relatives aux services en cours d’utilisation.  Cela nous empêche de générer des outils qui examinent le bot dans sa globalité.

Pour résoudre ce problème, nous avons créé un **fichier .bot** qui doit servir d’emplacement de rassemblement des références de service afin d’activer les outils.  Par exemple, l’émulateur Bot Framework ([V4](https://aka.ms/Emulator-wiki-getting-started)) utilise un fichier .bot pour créer une vue unifiée sur les services connectés que votre bot utilise.  

Avec un fichier .bot, vous pouvez inscrire des services comme :

* **Localhost** - des points de terminaison de débogueur local
* [**Azure Bot Service**](https://azure.microsoft.com/en-us/services/bot-service/) - des inscriptions Azure Bot Service.
* [**LUIS.AI**](https://www.luis.ai/) - LUIS permet à votre bot de communiquer avec des utilisateurs en langage naturel. 
* [**QnA Maker**](https://qnamaker.ai/) - générez, formez et publiez en quelques minutes un bot de questions-réponses simple reposant sur des URL de FAQ, des documents structurés ou des contenus éditoriaux.
* [**Dispatch**](https://github.com/Microsoft/botbuilder-tools/tree/master/Dispatch) - des modèles pour une répartition sur plusieurs services.
* [**Azure Application Insights**](https://azure.microsoft.com/en-us/services/application-insights/) - pour des insights et analyses de bot.
* [**Stockage Blob Azure**](https://azure.microsoft.com/en-us/services/storage/blobs/) - pour la persistance de l’état du bot. 
* [**Azure Cosmos DB**](https://azure.microsoft.com/en-us/services/cosmos-db/) - un service de base de données multimodèle distribué globalement pour rendre persistant l’état du bot.

En dehors de ces éléments, votre bot peut s’appuyer sur d’autres services personnalisés. Vous pouvez utiliser la fonctionnalité de [service générique](https://github.com/Microsoft/botbuilder-tools/blob/master/packages/MSBot/docs/add-services.md) pour connecter une configuration de service générique.

## <a name="when-is-a-bot-file-created"></a>À quel moment un fichier .bot est-il créé ? 
- Si vous créez un bot avec [Azure Bot Service](https://ms.portal.azure.com/#blade/Microsoft_Azure_Marketplace/GalleryResultsListBlade/selectedSubMenuItemId/%7B%22menuItemId%22%3A%22gallery%2FCognitiveServices_MP%2FBotService%22%2C%22resourceGroupId%22%3A%22%22%2C%22resourceGroupLocation%22%3A%22%22%2C%22dontDiscardJourney%22%3Afalse%2C%22launchingContext%22%3A%7B%22source%22%3A%5B%22GalleryFeaturedMenuItemPart%22%5D%2C%22menuItemId%22%3A%22CognitiveServices_MP%22%2C%22subMenuItemId%22%3A%22BotService%22%7D%7D), un fichier .bot est automatiquement créé pour vous, accompagné d’une liste des services connectés approvisionnés. Le fichier .bot est chiffré par défaut.
- Si vous créez un bot à l’aide du [modèle](https://marketplace.visualstudio.com/items?itemName=BotBuilder.botbuilderv4) de Kit de développement logiciel (SDK) Bot Builder V4 pour Visual Studio ou du [générateur Yeoman](https://www.npmjs.com/package/generator-botbuilder) Bot Builder, un fichier .bot est automatiquement généré. Aucun service connecté n’est approvisionné dans ce flux, et le fichier de bot n’est pas chiffré.
- Si vous commencez avec [BotBuilder-samples](https://github.com/Microsoft/botbuilder-samples), chaque exemple du Kit de développement logiciel (SDK) Bot Builder V4 inclut un fichier .bot et celui-ci n’est pas chiffré. 
- Vous pouvez également créer un fichier de bot avec l’outil [MSBot](https://github.com/Microsoft/botbuilder-tools/blob/master/packages/MSBot/README.md).

## <a name="what-does-a-bot-file-look-like"></a>À quoi ressemble un fichier de bot ? 
Examinez un exemple de fichier [.bot](https://github.com/Microsoft/botbuilder-tools/blob/master/packages/MSBot/docs/sample-bot-file.json).
Pour apprendre à chiffrer et déchiffrer le fichier .bot, consultez [Bot Secrets](https://github.com/Microsoft/botbuilder-tools/blob/master/packages/MSBot/docs/bot-file-encryption.md) (Secrets de bot).
## <a name="why-do-i-need-a-bot-file"></a>Pourquoi ai-je besoin d’un fichier .bot ?

Un fichier .bot n’est **pas** obligatoire pour créer des bots avec le Kit de développement logiciel (SDK) Bot Builder. Vous pouvez continuer à utiliser appsettings.json, web.config, env, keyvault ou n’importe quel mécanisme souhaité pour suivre les références de service et les clés sur lesquelles reposent votre bot. Toutefois, pour tester le bot avec l’émulateur, vous avez besoin d’un fichier .bot. Et cet émulateur vous permet justement de créer un fichier .bot à des fins de test. Pour ce faire, démarrez l’émulateur et cliquez sur le lien de **création d’une configuration de bot** sur la page d’accueil. Dans la boîte de dialogue qui s’affiche, saisissez un **nom de bot** et une **URL de point de terminaison**. Ensuite, connectez-vous.

Vous trouverez ci-dessous les avantages liés à l’utilisation d’un fichier .bot :
- Offre un moyen standard de stockage des ressources, quel que soit le langage/la plateforme que vous utilisez.   
- Les outils CLI et l’émulateur Bot Framework fonctionnent correctement avec les services connectés de suivi dans un format cohérent (dans un fichier .bot) et reposent dessus 
- Des solutions de génération d’outils sophistiquées autour de la gestion et de la création de services sont plus complexes à mettre en place sans un schéma bien défini (fichier .bot).  


## <a name="using-bot-file-in-your-bot-builder-sdk-bot"></a>Utilisation du fichier .bot dans votre bot de Kit de développement logiciel (SDK) Bot Builder
Vous pouvez utiliser le fichier .bot pour obtenir des informations de configuration de service dans le code de votre bot. La bibliothèque BotFramework-Configuration disponible pour [C#](https://www.nuget.org/packages/Microsoft.Bot.Configuration) et [JS](https://www.npmjs.com/package/botframework-config) vous permet de charger un fichier de bot et prend en charge plusieurs méthodes pour interroger et obtenir les informations de configuration de service appropriées.

## <a name="additional-resources"></a>Ressources supplémentaires
Reportez-vous au fichier readme [MSBot](https://github.com/Microsoft/botbuilder-tools/blob/master/packages/MSBot/README.md) pour plus d’informations sur l’utilisation d’un fichier de bot.
