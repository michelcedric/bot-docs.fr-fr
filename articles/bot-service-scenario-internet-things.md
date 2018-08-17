---
title: Scénario de bot Internet des objets | Microsoft Docs
description: Découvrez le scénario de bot Internet des objets avec Bot Framework.
author: BrianRandell
ms.author: v-brra
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 3b65f323427760fa43586f471aefb6811ef3e675
ms.sourcegitcommit: 67445b42796d90661afc643c6bb6533e9a662cbc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/06/2018
ms.locfileid: "39574518"
---
# <a name="internet-of-things-iot-bot-scenario"></a>Scénario de bot dans l’Internet des objets (IoT)

[!INCLUDE [pre-release-label](includes/pre-release-label-v3.md)]

Ce bot Internet des objets (IoT) vous permet de contrôler facilement les appareils de votre maison, comme l’éclairage Philips Hue, à l’aide de commandes de conversation interactive ou vocales.

Les gens adorent parler à leurs objets. Depuis les jours de la première télécommande TV, ils apprécient de ne pas avoir à se déplacer pour avoir une influence sur leur environnement. Ce bot IoT permet de gérer une ampoule Philips Hue par de simples commandes de conversation ou avec la voix. En outre, lors de l’utilisation de la conversation, une personne peut avoir le choix entre plusieurs couleurs.

![Diagramme d’un bot Internet des objets](~/media/scenarios/bot-service-scenario-iot-bot.png)

Voici le flux logique d’un bot IoT :

1. L’utilisateur se connecte à Skype et accède au bot IoT.
2. À l’aide de sa voix, il demande au bot d’allumer la lumière via l’appareil IoT.
3. La requête est transmise à un service tiers ayant accès au réseau de l’appareil IoT.
4. Les résultats de la commande sont retournés à l’utilisateur.
5. Application Insights collecte la télémétrie du runtime pour faciliter le développement à l’aide des informations sur les performances et l’utilisation du bot.

## <a name="sample-bot"></a>Exemple de bot
Le bot IoT vous permettra d’utiliser rapidement les commandes de conversation à partir de canaux comme Skype ou Slack pour contrôler votre Hue. Pour faciliter l’accès à distance, vous appellerez des applets IFTTT prédéfinies pour fonctionner avec Hue.

Vous pouvez télécharger ou cloner le code source pour cet exemple de bot à partir des [Exemples de scénarios Bot Framework courants](https://aka.ms/bot/scenarios).

## <a name="components-youll-use"></a>Composants que vous allez utiliser
Le bot Internet des objets (IoT) utilise les composants suivants :
-   Philips Hue
-   If This Then That (IFTTT)
-   Application Insights

### <a name="philips-hue"></a>Philips Hue
Les ampoules et le pont connectés Philips Hue vous permettre de maîtriser parfaitement votre éclairage. Quoi que vous vouliez faire avec votre éclairage, c’est possible avec Hue. Vous pouvez utiliser l’API Hue depuis votre réseau local. Toutefois, vous souhaitez pouvoir accéder à vos appareils et éclairages contrôlés par Hue où que vous soyez à l’aide d’une interface de bot conviviale. Vous accédez donc à Hue via IFTTT.

### <a name="ifttt"></a>IFTTT
IFTTT est un service web gratuit utilisé pour créer des chaînes d’instructions conditionnelles simples, appelées applets. Vous pouvez déclencher une applet à partir de votre bot pour qu’elle fasse quelque chose en votre nom. Il existe plusieurs applets Hue prédéfinies qui peuvent servir à allumer et à éteindre la lumière, à modifier l’ambiance, et bien plus encore.

### <a name="application-insights"></a>Application Insights
Application Insights vous aide à obtenir des informations exploitables grâce à la gestion des performances d’applications et à l’analyse instantanée. Vous bénéficiez de tableaux de bord simples mais complets pour surveiller les performances et gérer les alertes, ce qui vous permet de garantir que votre bot reste disponible et performant. Vous pouvez détecter rapidement les problèmes, puis lancer une analyse de la cause racine afin de les localiser et de les résoudre.
