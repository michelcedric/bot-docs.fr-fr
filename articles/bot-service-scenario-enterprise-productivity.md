---
title: Scénario de robot de productivité d’entreprise | Microsoft Docs
description: Explorez le scénario de robot de productivité d’entreprise avec Bot Framework.
author: BrianRandell
ms.author: v-brra
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 1fe68144662be3de349d05ea861a230641ae1efb
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2018
ms.locfileid: "49996707"
---
# <a name="enterprise-productivity-bot-scenario"></a>Scénario de robot de productivité d’entreprise

[!INCLUDE [pre-release-label](includes/pre-release-label-v3.md)]

Le robot d’entreprise montre comment vous pouvez augmenter votre productivité en intégrant un robot avec votre calendrier Office 365 et d’autres services.

L’accès rapide aux informations client sans avoir à ouvrir un grand nombre de fenêtres est l’objectif du robot de productivité d’entreprise. À l’aide de commandes de conversation simples, un représentant commercial peut rechercher un client et vérifier son prochain rendez-vous via l’API Graph et Office 365. À partir de là, ils peut accéder aux informations spécifiques du client stockées dans Dynamics CRM, par exemple, pour récupérer un cas ou en créer un.

![Diagramme de robot d’entreprise](~/media/scenarios/bot-service-scenario-enterprise-bot.png)

Voici le flux logique d’un robot de productivité d’entreprise :

1. L’employé accède au robot de productivité d’entreprise.
2. Azure Active Directory valide l’identité de l’employé.
3. Le robot de productivité d’entreprise est capable d’interroger le calendrier Office 365 de l’employé via Microsoft Azure AD Graph.
4. À l’aide des données extraites du calendrier, le robot accède aux informations spécifiques dans Dynamics CRM.
5. Les informations sont retournées à l’employé qui peut filtrer les données sans quitter le robot.
6. Application Insights collecte la télémétrie d’exécution pour faciliter le développement en fournissant des informations sur les performances et l’utilisation du robot.

Vous pouvez télécharger ou cloner le code source pour cet exemple de robot à partir des [Exemples pour les scénarios Bot Framework courants](https://aka.ms/bot/scenarios).

## <a name="sample-bot"></a>Exemple de robot
Étant donné que les robots sont accessibles à partir de divers canaux, vous pouvez les utiliser sur votre bureau à partir d’un portail d’entreprise ou de Skype lors de vos déplacements. Il suffit que vous soyez authentifié. Avec l’intégration d’Azure AD, votre robot de productivité d’entreprise sait que, si vous pouvez y accéder, vous avez été authentifié par Azure AD. À partir de là, vous pouvez demander au robot de vérifier votre prochain rendez-vous avec un client particulier. Le robot obtient ces informations en interrogeant Office 365 via l’API Graph. Ensuite, si un rendez-vous est planifié au cours des sept prochains jours, le robot interroge CRM afin de rechercher des cas récents pour le client. Le Bot répond soit sans aucun cas trouvé, ou avec le nombre de cas ouverts et fermés. A partir de là, vous pouvez demander au robot de répertorier les cas par type et de sonder des cas individuels.

## <a name="components-youll-use"></a>Composants que vous allez utiliser
Le robot de productivité d’entreprise utilise les composants suivants :
-   Azure AD pour l’authentification
-   API Graph pour Office 365
-   Dynamics CRM
-   Application Insights

### <a name="azure-active-directory-azure-ad"></a>Azure Active Directory (Azure AD)
Azure Active Directory (Azure AD) est le service Microsoft de gestion des annuaires et des identités basé sur le cloud mutualisé. En votre qualité de développeur de robots, Azure AD vous permet de vous concentrer sur la génération de votre robot en simplifiant et en accélérant son intégration au sein d’une solution de gestion des identités de classe mondiale utilisée par des millions d’organisations à travers le monde. En définissant une application Azure AD, vous pouvez contrôler qui a accès à votre robot et les données présentées par ce dernier, sans avoir à implémenter votre propre système complexe d’authentification et d’autorisation.

### <a name="graph-api-to-office-365"></a>API Graph pour Office 365
Microsoft Graph expose plusieurs API à partir d’Office 365 et d’autres services de cloud computing Microsoft via un point de terminaison unique à l’adresse https://graph.microsoft.com. Microsoft Graph facilite l’exécution de requêtes pour vous et le bot. L’API expose des données de plusieurs services de cloud computing Microsoft, dont Exchange Online dans le cadre d’Office 365, Azure Active Directory, SharePoint, etc. Vous pouvez utiliser l’API pour naviguer entre les entités et relations. Vous pouvez utiliser l’API à partir de vos robots en vous servant des points de terminaison SDK ou REST, ainsi qu’à partir de vos autres applications avec prise en charge en mode natif d’Android, d’iOS, de Ruby, d’UWP, de Xamarin et bien plus.

### <a name="dynamics-crm"></a>Dynamics CRM
Dynamics CRM est une plateforme d’engagement client. À l’aide d’API de robots et de CRM, vous pouvez créer des robots interactifs capables d’accéder aux données riches stockées dans CRM. La puissance de Dynamics CRM est à la disposition de votre robot pour créer des cas, vérifier des statuts, des recherches en matière de gestion des connaissances, et bien plus.

### <a name="application-insights"></a>Application Insights
Application Insights vous aide à obtenir des informations exploitables grâce à la gestion des performances d’applications et à l’analyse instantanée. Vous bénéficiez de tableaux de bord simples mais complets pour surveiller les performances et gérer les alertes, ce qui vous permet de garantir que votre bot reste disponible et performant. Vous pouvez détecter rapidement les problèmes, puis lancer une analyse de la cause racine afin de les localiser et de les résoudre.

## <a name="next-steps"></a>Étapes suivantes
Ensuite, découvrez le scénario de robot d’information.

> [!div class="nextstepaction"]
> [Scénario de robot d’information](bot-service-scenario-informational.md)
