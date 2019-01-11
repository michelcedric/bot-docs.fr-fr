---
title: Analyse de conversation à l’aide de Power BI | Microsoft Docs
description: Découvrez la façon dont le modèle de bot d’entreprise utilise Application Insights pour créer des insights via PowerBI
author: darrenj
ms.author: darrenj
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 09/18/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 88208a2f5b0eb88d3b2964e63a21585484166d73
ms.sourcegitcommit: 2d84d5d290359ac3cfb8c8f977164f799666f1ab
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/09/2019
ms.locfileid: "54152172"
---
# <a name="enterprise-bot-template---conversational-analytics-using-powerbi-dashboard-and-application-insights"></a>Modèle de bot d’entreprise - analyse de conversation à l’aide du tableau de bord PowerBI et d’Application Insights

> [!NOTE]
> Cet article s’applique à la version v4 du Kit de développement logiciel (SDK). 

Une fois que votre bot est déployé et qu’il commence à traiter les messages, vous pouvez voir des données de télémétrie circuler dans l’instance Application Insights au sein de votre groupe de ressources. 

Ces données de télémétrie peuvent être affichées dans le panneau Application Insights dans le Portail Azure et à l’aide de Log Analytics. En outre, les mêmes données de télémétrie peuvent être utilisées par Power BI pour fournir des insights métier plus générales sur l’utilisation de votre bot.

Un exemple de tableau de bord Power BI est fourni dans [Conversational AI Telemetry](https://aka.ms/botPowerBiTemplate). 

Il est fourni à titre d’exemple et montre comment vous pouvez commencer à créer vos propres insights. Au fil du temps, nous allons améliorer ces visualisations. 


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
    - FromName
    - ConversationId
    - ConversationName
    - Locale
    - Text
    - RecipientId
    - RecipientName
```
  
```
-BotMessageSent
    - ActivityId,
    - Channel
    - RecipientId
    - ConversationId
    - ConversationName
    - Locale
    - RecipientId
    - RecipientName
    - ReplyToId
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
    - DialogId
```

```
- QnAMaker
    - ActivityId
    - ConversationId
    - OriginalQuestion
    - FromName
    - ArticleFound
    - Question
    - Answer
    - Score
```