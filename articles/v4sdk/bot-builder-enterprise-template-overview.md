---
title: Présentation du modèle de bot d’entreprise | Microsoft Docs
description: En savoir plus sur les fonctionnalités du modèle de bot d’entreprise
author: darrenj
ms.author: darrenj
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 09/18/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 43bc3c7606a12084690d71f8b6ea2dc3b2e5984d
ms.sourcegitcommit: 6c719b51c9e4e84f5642100a33fe346b21360e8a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/28/2018
ms.locfileid: "52451991"
---
# <a name="enterprise-bot-template"></a>Modèle de bot d’entreprise 

> [!NOTE]
> Cet article s’applique à la version v4 du Kit de développement logiciel (SDK). 

La création d’une expérience conversationnelle de haute qualité nécessite un ensemble de fonctionnalités de base. Pour vous aider à créer des expériences conversationnelles optimales, nous avons créé un modèle de bot d’entreprise. Ce modèle réunit toutes les meilleures pratiques et tous les composants de prise en charge que nous avons identifiés lors de la création d’expériences conversationnelles. 

Il simplifie considérablement la création d’un nouveau projet de bot. Le modèle fournit les fonctionnalités suivantes prêtes à l’emploi, grâce à l’utilisation du [SDK Bot Builder v4](https://github.com/Microsoft/botbuilder) et des [outils Bot Builder](https://github.com/Microsoft/botbuilder-tools).

Fonctionnalité | Description |
------------ | -------------
Message de présentation | Message de présentation avec une carte adaptative au début de la conversation. Il explique les fonctionnalités des bots, et est muni de boutons pour guider les questions initiales. Les développeurs peuvent ensuite personnaliser ce message comme ils le souhaitent.
Indicateurs de saisie automatisés  | Envoi d’indicateurs de saisie visuels lors des conversations et répétition pour les opérations dont l’exécution est longue.
Configuration pilotée du fichier .bot | Toutes les informations de configuration pour votre bot (par exemple, LUIS, points de terminaison du répartiteur, Application Insights) sont incluses dans le fichier .bot et utilisées pour piloter le démarrage de votre bot.
Intentions conversationnelles de base  | Intentions de base (message d’accueil, d’au revoir, d’aide, d’annulation, etc.) en anglais, français, italien, allemand, espagnol et chinois. Ces intentions se trouvent dans les fichiers .LU (Language Understanding) afin de faciliter leur modification.
Réponses conversationnelles de base  | Réponses aux intentions conversationnelles de base contenues dans des classes d’affichage séparées. Ces réponses seront déplacées dans les nouveaux fichiers LG (Language Generation) à l’avenir.
Détection de contenu inapproprié ou d’informations d’identification personnelle (PII)  |Détection de données inappropriées ou PII dans les conversations entrantes grâce à l’utilisation de [Content Moderator](https://azure.microsoft.com/en-us/services/cognitive-services/content-moderator/) dans un composant d’intergiciel.
Transcriptions  | Transcriptions de toutes les conversations stockées dans Stockage Azure
Répartiteur | Tout modèle [Dispatch](https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-tutorial-dispatch?view=azure-bot-service-4.0&tabs=csaddref%2Ccsbotconfig) intégré pour déterminer si un énoncé donné doit être traité par LUIS + Code ou passé à QnA Maker.
Intégration de QnA Maker  | Intégration avec [QnA Maker](https://www.qnamaker.ai) pour répondre à des questions générales à partir d’une base de connaissances qui peut exploiter des sources de données existantes (par exemple, des manuels PDF).
Insights conversationnels  | Intégration avec [Application Insights](https://azure.microsoft.com/en-gb/services/application-insights/) pour collecter la télémétrie de toutes les conversations et un exemple du tableau de bord PowerBI pour vous aider à démarrer avec les insights dans vos expériences conversationnelles.

De plus, toutes les ressources Azure nécessaires pour le bot sont automatiquement déployées : inscription du bot, Azure App Service, LUIS, QnA Maker, Content Moderator, CosmosDB, Stockage Azure et Application Insights. Par ailleurs, les modèles LUIS, QnA Maker et de répartition de base sont créés, entraînés et publiés pour activer immédiatement les tests des intentions de base et du routage.

Une fois que le modèle est créé et que les étapes de déploiement sont exécutées, vous pouvez appuyer sur F5 pour effectuer un test de bout en bout. Grâce à cela, vous bénéficiez d’une base solide pour commencer votre expérience conversationnelle. Ainsi, vous n’avez plus à passer plusieurs jours à travailler sur chaque projet et la qualité de l’expérience conversationnelle est optimisée.

Pour commencer, continuez la [création du projet](bot-builder-enterprise-template-create-project.md). Pour en savoir plus sur les meilleures pratiques et les connaissances qui ont conduit à ces fonctionnalités, consultez l’article sur les [détails du modèle](bot-builder-enterprise-template-overview-detail.md). 

Le code source complet pour le modèle de bot d’entreprise est disponible sur cet [emplacement GitHub](https://github.com/Microsoft/AI/tree/master/templates/Enterprise-Template)
