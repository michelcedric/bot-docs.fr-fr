---
title: Scénario de robot Compétence Cortana | Microsoft Docs
description: Explorez le scénario de robot Compétence Cortana avec Bot Framework.
author: BrianRandell
ms.author: v-brra
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 2483366c6d325e5c18a77c2a89b63cc0350aef85
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39299048"
---
# <a name="cortana-skills-bot-scenario"></a>Scénario de robot Compétence Cortana
Le robot Compétence Cortana étend Cortana pour faciliter la réservation d’un rendez-vous d’entretien auto mobile à l’aide de la voix avec le contexte de votre calendrier.

Cortana est votre assistant personnel. En utilisant l’interface naturelle que constitue votre voix, et un robot Compétence Cortana personnalisé, vous pouvez demander à Cortana de parler à une organisation (par exemple, un atelier automobile), afin de prendre un rendez-vous. Le service peut fournir la liste des services, les heures disponibles et la durée. Cortana peut consulter votre calendrier pour voir s’il y a un éventuel conflit d’heure et, s’il n’y en pas, créer le rendez-vous et l’ajouter à votre calendrier.

![Diagramme de robot Compétence Cortana](~/media/scenarios/bot-service-scenario-cortana-skill.png)

Voici le flux logique d’un robot Compétence Cortana pour un atelier automobile :

1. L’utilisateur accède à Cortana à partir d’un PC ou d’un appareil mobile.
2. À l’aide de commandes textuelles ou vocales, l’utilisateur demande un rendez-vous pour l’entretien d’une automobile.
3. Le robot étant intégré avec Cortana, il a accès au calendrier de l’utilisateur et applique une logique à la demande.
4. Avec ces informations, le robot peut interroger le service automobile pour connaître les rendez-vous possibles.
5. Lorsque des options contextuelles lui sont présentées, l’utilisateur peut réserver le rendez-vous.
6. Application Insights collecte la télémétrie du runtime pour faciliter le développement à l’aide des informations sur les performances et l’utilisation du robot.

## <a name="sample-bot"></a>Exemple de robot
Avec un robot Compétence Cortana, tout est affaire de contexte personnel. Cortana vous permet d’utiliser votre voix pour demander que le « Service d’entretien mobile de Bob » vienne travailler sur votre voiture en fonction de votre localisation. En utilisant les informations personnelles exposées via Cortana, votre robot peut confirmer l’emplacement en fonction de la localisation de l’utilisateur au moment où il parle au robot.

Vous pouvez télécharger ou cloner le code source pour cet exemple de robot à partir des [Exemples pour les scénarios Bot Framework courants](https://aka.ms/bot/scenarios).

## <a name="components-youll-use"></a>Composants que vous allez utiliser
Le robot Cortana utilise les composants suivants :
-   Cortana
-   Application Insights

### <a name="cortana"></a>Cortana
Vous pouvez maintenant ajouter un support à votre robot en créant une Compétence Cortana. Vous utilisez le kit Compétence Cortana pour créer de nouvelles fonctionnalités (appelées compétences) pour Cortana. Une compétence est une construction qui permet à Cortana d’en faire plus. Vous créez des compétences à intégrer avec vos robots, qui autorisent Cortana à accomplir des tâches. Dans le cadre du processus d’innovation, Cortana peut (avec le consentement de l’utilisateur) transmettre des informations sur l’utilisateur à une compétence au moment du runtime, afin que la compétence puisse personnaliser son expérience en conséquence. La connaissance contextuelle de Cortana permet à votre robot d’être utile, voire plus intelligent pour eux. Une fois invoqués, certains types de compétences peuvent manipuler l’interface de Cortana pour avoir une conversation entre la compétence et l’utilisateur final. Une fois publiés, les utilisateurs peuvent voir et utiliser votre compétence sur Cortana pour Windows 10 Anniversary Update+ (Desktop et Mobile), iOS et Android.

### <a name="application-insights"></a>Application Insights
Application Insights vous aide à obtenir des informations exploitables grâce à la gestion des performances d’applications et à l’analytique instantanée. Vous bénéficiez de tableaux de bord simples mais complets pour surveiller les performances et gérer les alertes, ce qui vous permet de garantir que votre robot reste disponible et performant. Vous pouvez détecter rapidement les problèmes, puis lancer une analyse de la cause racine afin de les localiser et de les résoudre.

## <a name="next-steps"></a>Étapes suivantes
Découvrez ensuite le scénario de robot de productivité d’entreprise.

> [!div class="nextstepaction"]
> [Scénario de robot de productivité d’entreprise](bot-service-scenario-enterprise-productivity.md)
