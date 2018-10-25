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
ms.openlocfilehash: bdc86e5e64971e503157fe69a8b962e1d9b88542
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2018
ms.locfileid: "49998866"
---
# <a name="how-bot-service-works"></a>Fonctionnement de Service Bot

Service Bot fournit les principaux composants pour la création de robots, notamment le Kit de développement logiciel (SDK) Bot Builder pour le développement de robots, ainsi que Bot Framework pour la connexion des robots aux canaux. Pour la création de vos robots, Service Bot vous offre un choix de cinq modèles prenant en charge .NET et Node.js.

> [!IMPORTANT]
> Pour pouvoir utiliser Service Bot, vous devez avoir un abonnement Microsoft Azure. Si vous n’avez pas encore d’abonnement, vous pouvez vous inscrire pour un <a href="https://azure.microsoft.com/en-us/free/" target="_blank">compte gratuit</a>.

## <a name="hosting-plans"></a>Plans d’hébergement
Service Bot fournit deux plans d’hébergement pour vos robots. Vous pouvez choisir un plan d’hébergement adapté à vos besoins.

### <a name="app-service-plan"></a>Plan App Service

Un robot qui utilise un plan App Service est une application web Azure standard que vous pouvez configurer pour allouer une capacité prédéfinie avec des coûts et une mise à l’échelle prévisibles. Dans le cas d’un robot utilisant ce plan d’hébergement, vous pouvez :

* modifier le code source du robot en ligne à l’aide d’un éditeur de code avancé dans le navigateur ;
* télécharger, déboguer et republier votre robot C# au moyen de Visual Studio ;
* configurer un déploiement continu en toute facilité pour Visual Studio Online et Github ;
* utiliser un exemple de code préparé pour le Kit de développement logiciel (SDK) Bot Builder.

### <a name="consumption-plan"></a>Plan de consommation
Un robot qui utilise un plan de consommation est un robot sans serveur qui s’exécute sur <a href="http://go.microsoft.com/fwlink/?linkID=747839" target="_blank">Azure Functions</a> et applique la tarification d’Azure Functions avec paiement à l’exécution. Un robot qui utilise ce plan d’hébergement peut faire l’objet d’une mise à l’échelle de manière à gérer les pics de trafic exceptionnels. Vous pouvez modifier le code source du robot en ligne à l’aide d’un éditeur de code de base dans le navigateur. Pour plus d’informations sur l’environnement d’exécution d’un robot utilisant un plan de consommation, consultez l’article <a target='_blank' href='/azure/azure-functions/functions-scale'>Plans Consommation et App Service d’Azure Functions</a>.

## <a name="templates"></a>Modèles

Service Bot vous permet de créer rapidement et facilement un robot en C# ou Node.js à l’aide d’un des cinq modèles disponibles.

[!INCLUDE [Bot Service templates](~/includes/snippet-abs-templates.md)]

[Apprenez-en davantage](bot-service-concept-templates.md) sur les différents modèles et la manière dont ils peuvent vous aider à créer des robots.

## <a name="develop-and-deploy"></a>Développer et déployer

Par défaut, Service Bot vous permet de développer votre robot directement dans le navigateur à l’aide de l’éditeur de code en ligne, sans avoir besoin d’une chaîne d’outils. 

Vous pouvez développer et déboguer votre robot localement avec le Kit de développement logiciel (SDK) Bot Builder et un IDE tel que Visual Studio 2017. Vous pouvez publier votre robot directement sur Azure à l’aide de Visual Studio 2017 ou d’Azure CLI. Vous pouvez également [configurer un déploiement continu](bot-service-continuous-deployment.md) avec le système de contrôle de code source de votre choix, tel que VSTS ou GitHub. Lorsque le déploiement continu est configuré, vous pouvez développer et déboguer dans un IDE sur votre ordinateur local. Les modifications de code que vous validez dans le contrôle de code source sont automatiquement déployées sur Azure.  

> [!TIP]
> Après avoir activé le déploiement continu, pour éviter tout conflit, veillez à modifier votre code uniquement via le déploiement continu et non par le biais d’autres mécanismes.

## <a name="manage-your-bot"></a>Gérer votre robot 

Pendant le processus de création d’un robot à l’aide de Service Bot, vous spécifiez un nom pour votre bot, configurez son plan d’hébergement, choisissez un niveau tarifaire, et configurez d’autres paramètres. Une fois votre robot créé, vous pouvez modifier ses paramètres, le configurer pour qu’il s’exécute sur un ou plusieurs canaux, et le tester dans Discussion Web. 

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Créer un robot avec Service Bot](bot-service-quickstart.md)