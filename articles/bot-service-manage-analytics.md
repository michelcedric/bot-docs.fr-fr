---
title: Analyse de bot | Microsoft Docs
description: Découvrez comment utiliser la collecte de données et l’analyse pour améliorer votre bot grâce à des analyses dans Bot Framework.
keywords: analyse de bot, application insights, trafic, latence, intégrations, AppInsights
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: 503e9b2231b198346f5a7cd767a1e6a866e9e5b3
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39299205"
---
# <a name="bot-analytics"></a>Analyse de bot
Analytics est une extension d’[Application Insights](/azure/application-insights/app-insights-analytics). Application Insights fournit des données de **niveau de service** et d’instrumentation relatives au trafic, à la latence et aux intégrations. Analytics permet de créer des **rapports de conversation** à partir des données sur les utilisateurs, les messages et les canaux.

## <a name="view-analytics-for-a-bot"></a>Afficher l’analyse d’un bot
Pour accéder à Analytics, ouvrez le bot dans le portail des développeurs et cliquez sur **Analytics**.

Trop de données ? [Activer et configurer l’échantillonnage](/azure/application-insights/app-insights-sampling) afin de réduire les données de télémétrie, le trafic et stockage tout en conservant une analyse statistiquement correcte. 

### <a name="specify-channel"></a>Spécifier le canal
Choisissez les canaux qui apparaîtront dans les graphiques ci-dessous. Notez que si un bot n’est pas activé sur un canal, ce canal ne fournira pas de données.

![Sélectionnez le canal](~/media/analytics-channels.png)

* Cochez la case à cocher pour inclure un canal dans le graphique.
* Décochez la case à cocher pour supprimer un canal du graphique.

### <a name="specify-time-period"></a>Spécifier la période
L’analyse n’est disponible pour les 90 derniers jours. La collecte de données a commencé lorsqu’Application Insights a été activé.

![Sélectionnez la période](~/media/analytics-timepick.png)

Cliquez sur le menu déroulant, puis sur la période que les graphiques doivent afficher.
Notez que la modification de l’intervalle de temps global entraîne la modification en conséquence de l’incrément de temps (axe x) sur les graphiques.

### <a name="grand-totals"></a>Total général
Le nombre total d’utilisateurs actifs et de messages envoyés et reçus pendant le laps de temps spécifié.
Les tirets `--` indiquent qu’il n’y a aucune activité.

### <a name="retention"></a>Rétention
La rétention indique le nombre d’utilisateurs ayant envoyé un message qui sont revenus ultérieurement et en ont envoyé un autre.
Le graphique est une fenêtre cumulative de 10 jours ; les résultats ne sont pas affectés par la modification du laps de temps.

![Graphique de rétention](~/media/analytics-retention.png)

Notez que la date la plus récente possible est deux jours avant ; un utilisateur a envoyé des messages avant-hier, puis est *revenu* hier.

### <a name="user"></a>Utilisateur
Le graphique des utilisateurs indique le nombre d’utilisateurs ayant accédé au bot à l’aide de chaque canal pendant le laps de temps spécifié.

![Graphique des utilisateurs](~/media/analytics-users.png)

* Le graphique des pourcentages affiche le pourcentage d’utilisateurs ayant utilisé chaque canal.
* Le graphique linéaire indique le nombre d’utilisateurs qui avaient accès au bot à une heure donnée.
* La légende du graphique linéaire indique la couleur qui représente chaque canal et le nombre total d’utilisateurs pendant la période spécifiée.

### <a name="messages"></a>Messages
Le graphique des messages indique le nombre de messages envoyés et reçus dans chaque canal pendant le laps de temps spécifié.

![Graphique des messages](~/media/analytics-messages.png)

* Le graphique des pourcentages indique quel pourcentage de messages a été communiqué par chaque canal.
* Le graphique linéaire indique combien de messages ont été envoyés et reçus pendant le laps de temps spécifié.
* La légende du graphique linéaire indique la couleur de ligne qui représente chaque canal et le nombre total de messages envoyés et reçus sur ce canal pendant la période spécifiée. 

## <a name="enable-analytics"></a>Activer l'analyse
Analytics n’est disponible que lorsqu’Application Insights a été activé et configuré. Application Insights commence à collecter des données dès qu’il est activé. Par exemple, si Application Insights a été activée il y a une semaine pour un bot datant de six mois, il aura collecté les données d’une semaine.
> [!NOTE]
> Analytics nécessite un abonnement Azure et la [ressource](/azure/application-insights/app-insights-create-new-resource) Application Insights.
Pour accéder à Application Insights, ouvrez le bot dans le [portail Bot Framework](https://dev.botframework.com/) et cliquez sur **Paramètres**.

1. Créez une [ressource](/azure/application-insights/app-insights-create-new-resource) Application Insights.
2. Ouvrez le bot dans le tableau de bord. Cliquez sur **Paramètres** et faites défiler jusqu'à la section **Analytics**.
3. Entrez les informations pour connecter le bot à Application Insights. Tous les champs sont obligatoires.

![Connectez-vous à Insights](~/media/analytics-enable.png)

### <a name="appinsights-instrumentation-key"></a>Clé d'instrumentation AppInsights
Pour trouver cette valeur, ouvrez Application Insights et accédez à **Configurer** > **Propriétés**.

### <a name="appinsights-api-key"></a>Clé API AppInsights
Fournir une clé API Azure App Insights. Découvrez comment [générer une nouvelle clé API](https://dev.applicationinsights.io/documentation/Authorization/API-key-and-App-ID). Seule une autorisation de **lecture** est nécessaire.

### <a name="appinsights-application-id"></a>ID de l’application AppInsights
Pour trouver cette valeur, ouvrez Application Insights et accédez à **Configurer** > **Accès à l’API**.

Pour plus d’informations sur la façon de localiser ces valeurs, consultez [clés d’Application Insights](~/bot-service-resources-app-insights-keys.md).

## <a name="additional-resources"></a>Ressources supplémentaires
* [Clés Application Insights](~/bot-service-resources-app-insights-keys.md)