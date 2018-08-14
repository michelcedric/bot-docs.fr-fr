---
title: Scénario impliquant le bot Informations | Microsoft Docs
description: Découvrez le scénario impliquant le bot Informations avec Bot Framework.
author: BrianRandell
ms.author: v-brra
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 7c85227d062f32f9f1e8096f1dad83c09ed85a9f
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39299993"
---
# <a name="information-bot-scenario"></a>Scénario impliquant le bot Informations
Ce bot Informations peut répondre à des questions définies dans une base de connaissances ou une source de données FAQ au moyen de la fonctionnalité QnA Maker de Cognitive Services. En outre, il peut répondre à des questions plus ouvertes à l’aide du service Recherche Azure.

Les informations sont souvent enfouies dans des banques de données structurées telles que SQL Server, et peuvent être facilement récupérées au moyen d’une recherche. Considérons l’exemple d’un utilisateur qui recherche le statut d’une commande client à l’aide de commandes conversationnelles simples. Grâce à la fonctionnalité QnA Maker de Cognitive Services, l’utilisateur se voit proposer un ensemble d’options de recherche valides, comme la recherche d’un client, l’examen de sa commande la plus récente, etc. À l’aide du format QnA défini, l’utilisateur peut facilement poser des questions, soutenu par le service Recherche Azure qui peut rechercher les données stockées dans un service SQL Database.

![Diagramme du bot Informations](~/media/scenarios/bot-service-scenario-informational-bot.png)

Voici le flux logique d’un bot Informations :

1. L’employé démarre le bot Informations.
2. Azure Active Directory valide l’identité de l’employé.
3. L’employé peut demander au bot les types de requêtes qui sont pris en charge.
4. Les services Cognitive Services renvoient un bot de FAQ généré avec QnA Maker.
5. L’employé définit une requête valide.
6. Le bot soumet la requête au service Recherche Azure qui renvoie des informations sur les données d’application.
7. Application Insights collecte la télémétrie d’exécution pour faciliter le développement en fournissant des informations sur les performances et l’utilisation du bot.

## <a name="sample-bot"></a>Exemple de bot
L’exemple de bot, écrit en C#, s’exécute dans Microsoft Azure utilisant les données indexées par le service Recherche Azure à partir d’une instance SQL Database. Le bot présente une liste de questions possibles, avec des informations sur la façon de formuler la question (la réponse) à l’aide de la fonctionnalité QnA Maker de Cognitive Services. L’utilisateur du bot peut alors taper une requête qui recherche les données par le biais du service Recherche Azure dans une zone générale ou spécifique de la base de données indexée. L’exemple fournit une base de données simple contenant des clients et des informations sur les commandes. Application Insights effectue le suivi de l’utilisation du bot et vous permet de surveiller si des exceptions se produisent dans le bot. Le bot est publié sous une application Azure AD, ce qui vous permet de restreindre la liste des utilisateurs ayant accès aux informations.

Vous pouvez télécharger ou cloner le code source pour cet exemple de bot à partir du [référentiel d’exemples pour les scénarios Bot Framework courants](https://aka.ms/bot/scenarios).

## <a name="components-youll-use"></a>Composants utilisés
Le bot Informations utilise les composants suivants :
-   Azure AD pour l’authentification
-   Cognitive Services : QnA Maker
-   Recherche Azure
-   Application Insights

### <a name="azure-active-directory-azure-ad"></a>Azure Active Directory (Azure AD)
Azure Active Directory (Azure AD) est le service Microsoft de gestion des annuaires et des identités basé sur le cloud mutualisé. En votre qualité de développeur de bots, Azure AD vous permet de vous concentrer sur la génération de votre bot en simplifiant et en accélérant son intégration au sein d’une solution de gestion des identités de classe mondiale utilisée par des millions d’organisations à travers le monde. En définissant une application Azure AD, vous pouvez contrôler qui a accès à votre bot et les données présentées par ce dernier, sans avoir à implémenter votre propre système complexe d’authentification et d’autorisation.

### <a name="cognitive-services-qna-maker"></a>Cognitive Services : QnA Maker
La fonctionnalité QnA Maker de Cognitive Services vous aide à fournir une source de données FAQ que vos utilisateurs peuvent interroger à partir de votre bot. En cas d’utilisation de grandes quantités d’informations stockées dans différents systèmes, il peut être utile d’aider les utilisateurs à filtrer la source et l’ensemble d’informations. Si une base de données SQL comporte d’énormes quantités d’informations, une recherche en forme libre renvoie un volume d’informations excessif. En commençant par utiliser QnA Maker, vous pouvez définir une feuille de route pour les utilisateurs de votre bot afin de leur expliquer comment poser des questions intelligentes qui sont ensuite récupérables par le biais du service Recherche Azure.

### <a name="azure-search"></a>Recherche Azure
Le service Recherche Azure est un service de recherche cloud pour les applications vous permettant de rechercher rapidement les index opérationnels. En l’exécutant sur Microsoft Azure, vous pouvez facilement effectuer une montée ou descente en puissance selon l’évolution de votre utilisation. Vous pouvez connecter les résultats de la recherche aux objectifs métiers en bénéficiant d’un contrôle accru sur le classement des recherches en fonction de la pertinence et en exposant les données cachées dans vos bases de données.

### <a name="application-insights"></a>Application Insights
Application Insights vous aide à obtenir des insights actionnables grâce à la gestion des performances d’applications et à l’analytique instantanée. Vous bénéficiez de tableaux de bord simples mais complets pour surveiller les performances et gérer les alertes, ce qui vous permet de garantir que votre bot reste disponible et performant. Vous pouvez déceler rapidement les problèmes, puis lancer une analyse de la cause racine afin de localiser et résoudre les problèmes.

## <a name="next-steps"></a>Étapes suivantes
Découvrez à présent le scénario impliquant le bot Internet des objets.

> [!div class="nextstepaction"]
> [Scénario impliquant le bot Internet des objets](bot-service-scenario-internet-things.md)
