---
title: Fonctionnement de Service Bot | Microsoft Docs
description: Découvrez les fonctionnalités et capacités de Service Bot.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 85ef0fde39980bab1b891518e338fddbd56b275a
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/11/2019
ms.locfileid: "54224454"
---
# <a name="how-bot-service-works"></a>Fonctionnement de Service Bot

Bot Service fournit les principaux composants permettant de créer des bots, notamment le kit SDK Bot Framework pour le développement de bots et Bot Framework pour la connexion des bots aux canaux. Pour la création de vos robots, Bot Service vous offre un choix de cinq modèles prenant en charge .NET et Node.js.

> [!IMPORTANT]
> Pour pouvoir utiliser Bot Service, vous devez avoir un abonnement Microsoft Azure. Si vous n’avez pas encore d’abonnement, vous pouvez vous inscrire pour un <a href="https://azure.microsoft.com/en-us/free/" target="_blank">compte gratuit</a>.

## <a name="hosting-plans"></a>Plans d’hébergement
Bot Service fournit deux plans d’hébergement pour vos bots. Vous pouvez choisir un plan d’hébergement adapté à vos besoins.

### <a name="app-service-plan"></a>Plan App Service

Un bot qui utilise un plan App Service est une application web Azure standard que vous pouvez configurer pour allouer une capacité prédéfinie avec des coûts et une mise à l’échelle prévisibles. Dans le cas d’un bot utilisant ce plan d’hébergement, vous pouvez :

* modifier le code source du bot en ligne à l’aide d’un éditeur de code avancé dans le navigateur ;
* télécharger, déboguer et republier votre bot C# au moyen de Visual Studio ;
* configurer un déploiement continu en toute facilité pour Visual Studio Online et Github ;
* utiliser un exemple de code préparé pour le kit SDK Bot Framework.

### <a name="consumption-plan"></a>Plan de consommation
Un bot qui utilise un plan de consommation est un bot serverless qui s’exécute sur <a href="http://go.microsoft.com/fwlink/?linkID=747839" target="_blank">Azure Functions</a> et applique la tarification d’Azure Functions avec paiement à l’exécution. Un bot qui utilise ce plan d’hébergement peut faire l’objet d’une mise à l’échelle de manière à gérer les pics de trafic exceptionnels. Vous pouvez modifier le code source du bot en ligne à l’aide d’un éditeur de code de base dans le navigateur. Pour plus d’informations sur l’environnement d’exécution d’un bot utilisant un plan de consommation, consultez l’article <a target='_blank' href='/azure/azure-functions/functions-scale'>Plans Consommation et App Service d’Azure Functions</a>.

## <a name="templates"></a>Modèles

Bot Service vous permet de créer rapidement et facilement un robot en C# ou Node.js à l’aide d’un des cinq modèles disponibles.

[!INCLUDE [Bot Service templates](~/includes/snippet-abs-templates.md)]

[Apprenez-en davantage](bot-service-concept-templates.md) sur les différents modèles et la manière dont ils peuvent vous aider à créer des bots.

## <a name="develop-and-deploy"></a>Développer et déployer

Par défaut, Bot Service vous permet de développer votre bot directement dans le navigateur à l’aide de l’éditeur de code en ligne, sans avoir besoin d’une chaîne d’outils. 

Vous pouvez développer et déboguer votre bot localement avec le kit SDK Bot Framework et un IDE, comme Visual Studio 2017. Vous pouvez publier votre bot directement sur Azure à l’aide de Visual Studio 2017 ou d’Azure CLI. Vous pouvez également [configurer un déploiement continu](bot-service-continuous-deployment.md) avec le système de contrôle de code source de votre choix, tel que VSTS ou GitHub. Lorsque le déploiement continu est configuré, vous pouvez développer et déboguer dans un IDE sur votre ordinateur local. Les modifications de code que vous validez dans le contrôle de code source sont automatiquement déployées sur Azure.  

> [!TIP]
> Après avoir activé le déploiement continu, pour éviter tout conflit, veillez à modifier votre code uniquement via le déploiement continu et non par le biais d’autres mécanismes.

## <a name="manage-your-bot"></a>Gérer votre bot 

Pendant le processus de création d’un bot à l’aide de Bot Service, vous spécifiez un nom pour votre bot, configurez son plan d’hébergement, choisissez un niveau tarifaire, et configurez d’autres paramètres. Une fois votre bot créé, vous pouvez modifier ses paramètres, le configurer pour qu’il s’exécute sur un ou plusieurs canaux, et le tester dans Web Chat. 

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Créer un bot avec Bot Service](bot-service-quickstart.md)