---
title: Analyse de conversation à l’aide de Power BI | Microsoft Docs
description: Découvrez la façon dont le modèle de bot d’entreprise utilise Application Insights pour créer des insights via PowerBI
author: darrenj
ms.author: darrenj
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 09/18/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 18dcaeaf2967a90ca3ecb8ff32c1ef6399d5f498
ms.sourcegitcommit: 3bf3dbb1a440b3d83e58499c6a2ac116fe04b2f6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/23/2018
ms.locfileid: "46708594"
---
# <a name="enterprise-bot-template---conversational-analytics-using-powerbi-dashboard-and-application-insights"></a>Modèle de bot d’entreprise - analyse de conversation à l’aide du tableau de bord PowerBI et d’Application Insights

> [!NOTE]
> Cet article s’applique à la version v4 du Kit de développement logiciel (SDK). 

Une fois que votre bot est déployé et qu’il commence à traiter les messages, vous pouvez voir des données de télémétrie circuler dans l’instance Application Insights au sein de votre groupe de ressources. 

Ces données de télémétrie peuvent être affichées dans le panneau Application Insights dans le Portail Azure et à l’aide de Log Analytics. En outre, les mêmes données de télémétrie peuvent être utilisées par Power BI pour fournir des insights métier plus générales sur l’utilisation de votre bot.

Un exemple de tableau de bord PowerBI est fourni dans le dossier PowerBI du projet créé. Il est fourni à titre d’exemple et montre comment vous pouvez commencer à créer vos propres insights. Au fil du temps, nous allons améliorer ces visualisations. 

## <a name="getting-started"></a>Mise en route

- Téléchargez PowerBI Desktop [ici](https://powerbi.microsoft.com/en-us/desktop/).
 
- Récupérez un élément ```Application Id``` pour la ressource Application Insights utilisée par votre bot. Vous pouvez l’obtenir en accédant à la page d’accès à l’API sous la section de configuration du volet Application Insights du portail Azure.

Double-cliquez sur le fichier de modèle Power BI fourni, situé dans le dossier Power BI de votre Solution. Vous allez être invité à entrer l’élément ```Application Id``` que vous avez récupéré à l’étape précédente. Terminez l’authentification si vous y êtes invité, à l’aide de vos informations d’identification d’abonnement Azure. Vous aurez peut-être besoin de cliquer sur le paramètre de compte organisationnel pour vous connecter.

Le tableau de bord qui s’affiche est désormais associé à votre instance Application Insights et vous devez voir des insights initiales dans le tableau de bord si des messages ont été envoyés et reçus.

>Notez que la visualisation Sentiment n’affiche pas de données, puisque le script de déploiement actuel n’active pas Sentiment lors de la publication du modèle LUIS. Si vous [republiez](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/luis-how-to-publish-app) le modèle LUIS et activez Sentiment, cela fonctionnera.

## <a name="middleware-processing"></a>Traitement de l’intergiciel

Les wrappers de télémétrie autour des classes QnAMaker et LuisRecognizer sont fournis pour garantir une sortie des données de télémétrie cohérente indépendamment du scénario et permettre à un tableau de bord standard de fonctionner pour chaque projet.

```TelemetryLuisRecognizer``` et ```TelemetryQnAMaker``` fournissent des propriétés sur le constructeur permettant à un développeur de désactiver la journalisation des noms d’utilisateur et des messages d’origine. Toutefois, cela réduit la quantité d’insights disponibles.

## <a name="telemetry-captured"></a>Données de télémétrie capturées

4 événements de télémétrie distincts sont capturés via l’utilisation de ```TelemetryLuisRecognizer``` et ```TelemetryQnAMaker```, qui sont activés par défaut dans le modèle d’entreprise. 

Chaque intention LUIS utilisée par votre projet porte le préfixe LuisIntent. pour permettre une identification facile des intentions par le tableau de bord.

```
-BotMessageReceived
    - ActivityId
    - Channel
    - FromId
    - Conversationid
    - ConversationName
    - Locale
    - UserName
    - Text
```
  
```
-BotMessageSent
    - ActivityId,
    - Channel
    - RecipientId
    - Conversationid
    - ConversationName
    - Locale
    - ReceipientName
    - Text
```

```
- LuisIntent.*
    - ActivityId
    - Intent
    - IntentScore
    - SentimentLabel
    - SentimentScore
    - ConversationId
    - Question
```

```
- QnAMaker
    - ActivityId
    - ConversationId
    - OriginalQuestion
    - UserName
    - QnAItemFound
    - Question
    - Answer
    - Score
```