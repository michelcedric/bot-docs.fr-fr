---
title: Créer un bot Conversation Designer | Microsoft Docs
description: Découvrez comment créer un bot à l’aide de Conversation Designer.
author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ROBOTS: NoIndex, NoFollow
ms.openlocfilehash: 1b33d08e56bf8ae473b28cf1cf3a26d8138ce861
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39299017"
---
# <a name="create-a-new-conversation-designer-bot"></a>Créer un bot Conversation Designer
> [!IMPORTANT]
> Conversation Designer n’est pas encore accessible à tous les clients. Des informations supplémentaires sur la disponibilité de Conversation Designer seront communiquées plus tard cette année.

Ce didacticiel vous guide tout au long de la procédure détaillée de création d’un bot Conversation Designer. 

## <a name="prerequisites"></a>Prérequis

- Conversation Designer nécessite un abonnement Azure. Vous pouvez commencer <a href="https://azure.microsoft.com/en-us/" target="_blank">ici</a>.
- Si vous ne l’avez pas déjà fait, vérifiez que vous vous êtes connecté au [portail LUIS](https://luis.ai) au moins une fois après avoir créé un compte.
- Vous devez connaître la programmation en JavaScript. Les fonctions de script personnalisé sont rédigées en JavaScript.
- Microsoft Edge ou Google Chrome

## <a name="create-a-conversation-designer-bot"></a>Créer un bot Conversation Designer

Pour créer un bot Conversation Designer, procédez comme suit :
1. Accédez à https://dev.botframework.com/ et connectez-vous. Utilisez l’adresse e-mail que vous avez fournie à Microsoft Corporation pour participer à cette préversion privée.
2. Cliquez sur **Créer un bot** dans le panneau de navigation supérieur droit. 
3. Cliquez sur **Créer** pour *créer un bot avec Conversation Designer*.
4. Sélectionnez l’un des nombreux [exemples de bots](conversation-designer-sample-bots.md) avec lequel vous souhaitez commencer. Cliquez sur **Suivant**. Si vous ne savez pas quel **exemple de bot** utiliser, choisissez simplement celui qui vous semble le plus proche du bot que vous souhaitez créer. Vous pourrez basculer vers un autre **exemple de bot** par la suite.
5. Renseignez tous les champs, puis cliquez sur **Créer un bot** ; l’approvisionnement du bot nécessite environ 2 minutes. 

## <a name="bot-provisioning"></a>Approvisionnement du bot

Lorsque vous créez un bot Conversation Designer, les fonctionnalités Azure ci-après sont automatiquement approvisionnées : 

1. Groupe de ressources Azure avec le nom de bot que vous avez spécifié
2. Azure App Service
3. Plan Azure App Service 
4. Compte de Stockage Azure
5. Application Insights 
6. Abonnement Cognitive Services pour [LUIS.ai](https://luis.ai). Une application LUIS est créée avec le **descripteur Bot** (ainsi qu’une chaîne générée de manière aléatoire) en tant que nom d’application.
7. Application de compte Microsoft unique [En savoir plus](https://apps.dev.microsoft.com/#/appList)

## <a name="welcome-message"></a>Message de bienvenue

Une fois que le bot a été approvisionné, Conversation Designer ouvre la page **Générer** du bot. Un message de bienvenue s’affiche avec des informations conçues pour vous aider à démarrer. Explorez ces options ou fermez le message et commencez à travailler sur votre bot. Vous pouvez revenir au message de bienvenue en cliquant sur le bouton de sélection (**...**) dans le panneau de navigation supérieur gauche, puis en sélectionnant l’option **Bienvenue...** .

## <a name="next-step"></a>Étape suivante
> [!div class="nextstepaction"]
> [Enregistrer le bot](conversation-designer-save-bot.md)

## <a name="additional-resources"></a>Ressources supplémentaires
* En savoir plus sur les [tâches](conversation-designer-tasks.md)
* En savoir plus sur le [Kit de développement logiciel (SDK) Bot Builder pour Node](../nodejs/index.md) 
