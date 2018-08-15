---
title: Scénario Robot Commerce | Microsoft Docs
description: Explorez le scénario Robot Commerce avec Bot Framework.
author: BrianRandell
ms.author: v-brra
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: b809e98ec971abaac98fd33c4fb2c285baca898f
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39300168"
---
# <a name="commerce-bot-scenario"></a>Scénario Robot Commerce
Le scénario [Robot Commerce](bot-service-scenario-commerce.md) décrit un robot qui remplace les interactions par e-mail et par téléphone que les utilisateurs ont généralement avec le service de conciergerie d’un hôtel. Le robot tire parti de Cognitive Services pour mieux traiter les demandes textuelles et vocales des clients avec un contexte obtenu grâce à une intégration avec des services principaux.

![Diagramme du robot d’application](~/media/scenarios/bot-service-scenario-commerce-bot.png)

Voici le flux logique d’un robot Commerce faisant office de concierge pour un hôtel :

1. Le client utilise l’application mobile Hôtel.
2. L’utilisateur s’authentifie à l’aide d’Azure AD B2C.
3. L’utilisateur demande des informations à l’aide du robot Application personnalisé. 
4. Les services Cognitive Services aident à traiter la demande en langage naturel.
5. La réponse est examinée par le client, qui peut affiner sa question en s’exprimant en langage naturel.
6. Quand l’utilisateur est satisfait des résultats, le robot Application met à jour la réservation du client.
7. Application Insights collecte la télémétrie du runtime pour faciliter le développement à l’aide des informations sur les performances et l’utilisation du robot.

## <a name="sample-bot"></a>Exemple de robot
L’exemple de robot Commerce est conçu autour d’un service fictif de conciergerie d’hôtel. Les clients accèdent au robot écrit en C# une fois qu’ils ont authentifié Azure AD B2C auprès d’un hôtel via l’application mobile des services membres de la chaîne. La chaîne stocke les réservations dans une base de données SQL Database. Un client peut poser des questions en langage naturel, telles que « Combien me coûterait la location d’une cabanon de piscine pendant la durée de mon séjour ? ». Le robot dispose à son tour d’un contexte sur l’hôtel et la durée du séjour du client. En outre, le service Language Understanding (LUIS) aide le robot à obtenir du contexte même à partir d’expressions aussi simples que « cabanon de piscine ». Le robot donne la réponse, puis peut proposer de réserver un cabanon pour le client, en proposant des choix concernant le nombre de jours et le type de cabanon. Une fois que le robot dispose de toutes les données nécessaires, il réserve la demande. Le client peut également utiliser sa voix pour formuler la même demande.

Vous pouvez télécharger ou cloner le code source pour cet exemple de robot à partir des [Exemples pour les scénarios Bot Framework courants](https://aka.ms/bot/scenarios).

## <a name="components-youll-use"></a>Composants que vous allez utiliser
Le robot Commerce utilise les composants suivants :
-   Azure AD pour l’authentification
-   Cognitive Services : LUIS
-   Application Insights

### <a name="azure-active-directory-azure-ad"></a>Azure Active Directory (Azure AD)
Azure Active Directory (Azure AD) est le service Microsoft de gestion des annuaires et des identités basé sur le cloud mutualisé. En votre qualité de développeur de robots, Azure AD vous permet de vous concentrer sur la génération de votre robot en simplifiant et en accélérant son intégration au sein d’une solution de gestion des identités de classe mondiale utilisée par des millions d’organisations à travers le monde. Azure AD prend en charge un connecteur B2C, ce qui vous permet d’identifier les personnes à l’aide d’ID externes comme un compte Google, Facebook ou Microsoft. Azure AD vous libère de la responsabilité de gérer les informations d’identification de l’utilisateur, en vous permettant de vous concentrer sur votre solution de robot, sachant que vous pouvez établir une corrélation entre l’utilisateur du robot et les données correctes exposées par l’application.

### <a name="cognitive-services-luis"></a>Cognitive Services : LUIS
En tant que membre de la famille de technologies Cognitive Services, Language Understanding (LUIS) apporte à vos applications la puissance de l’apprentissage automatique. Actuellement, LUIS prend en charge plusieurs langues qui permettent à votre robot de comprendre les attentes d’une personne. Lors de l’intégration avec LUIS, vous exprimez une intention et définissez les entités que votre robot comprend. Vous enseignez ensuite à votre robot comment comprendre ces intentions et entités en l’entraînant avec des exemples d’énoncés. Vous avez la possibilité d’affiner votre intégration à l’aide de listes de phrases et d’expressions régulières afin que votre robot soit aussi fluide que possible pour vos besoins de conversation particuliers.

### <a name="application-insights"></a>Application Insights
Application Insights vous aide à obtenir des informations exploitables grâce à la gestion des performances d’applications et à l’analytique instantanée. Vous bénéficiez de tableaux de bord simples mais complets pour surveiller les performances et gérer les alertes, ce qui vous permet de garantir que votre robot reste disponible et performant. Vous pouvez détecter rapidement les problèmes, puis lancer une analyse de la cause racine afin de les localiser et de les résoudre.

## <a name="next-steps"></a>Étapes suivantes
Ensuite, découvrez le scénario de robot Compétence Cortana.

> [!div class="nextstepaction"]
> [Scénario de robot Compétence Cortana](bot-service-scenario-cortana-skill.md)
