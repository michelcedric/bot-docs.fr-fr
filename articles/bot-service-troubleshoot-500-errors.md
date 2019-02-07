---
title: Résoudre les erreurs HTTP 500 de bot | Microsoft Docs
description: Comment résoudre les erreurs HTTP 500 dans un bot déployé.
keywords: résoudre les problèmes, dépanner, HTTP 500, problèmes.
author: jonathanFingold
ms.author: v-jofing
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/20/2018
ms.openlocfilehash: f86cacce5b25f60010f646cf5989123e3abf3bf2
ms.sourcegitcommit: 32615b88e4758004c8c99e9d564658a700c7d61f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 02/04/2019
ms.locfileid: "55711973"
---
# <a name="troubleshoot-http-500-errors"></a>Résoudre les erreurs HTTP 500

La première étape de résolution des erreurs 500 active Application Insights.

Les exemples luis-with-appinsights ([C#](https://aka.ms/cs-luis-with-appinsights-sample) / [JS](https://aka.ms/js-luis-with-appinsights-sample)) et qna-with-appinsights ([C#](https://aka.ms/qna-with-appinsights) / [JS](https://aka.ms/js-qna-with-appinsights-sample)) montrent des bots qui prennent en charge Azure Application Insights. Pour des informations sur la façon d’ajouter Application Insights à un bot existant, consultez [conversation analytics tememetry](https://aka.ms/botPowerBiTemplate).

## <a name="enable-application-insights-on-aspnet"></a>Activer Application Insights sur ASP.Net

Pour une prise en charge de base d’Application Insights, consultez [Configurer Application Insights pour votre site web ASP.NET](https://docs.microsoft.com/azure/application-insights/app-insights-asp-net). Le Bot Framework (à compter de v4.2) offre un niveau supérieur de télémétrie Application Insights, mais il n’est pas obligatoire pour diagnostiquer les erreurs HTTP 500.

## <a name="enable-application-insights-on-nodejs"></a>Activer Application Insights sur Node.js

Pour une prise en charge de base d’Application Insights, consultez [Surveiller vos services et applications Node.js avec Application Insights](https://docs.microsoft.com/azure/azure-monitor/learn/nodejs-quick-start). Le Bot Framework (à compter de v4.2) offre un niveau supérieur de télémétrie Application Insights, mais il n’est pas obligatoire pour diagnostiquer les erreurs HTTP 500.

## <a name="query-for-exceptions"></a>Rechercher les exceptions

La méthode la plus simple pour analyser des erreurs 500 de code d’état HTTP consiste à commencer par les exceptions.

Les requêtes suivantes vous donneront les exceptions les plus récentes :

```sql
exceptions
| order by timestamp desc
| project timestamp, operation_Id, appName
```

Avec la première requête, sélectionnez quelques-uns des ID d’opération et recherchez plus d’informations :

```sql
let my_operation_id = "d298f1385197fd438b520e617d58f4fb";
let union_all = () {
    union
    (traces | where operation_Id == my_operation_id),
    (customEvents | where operation_Id == my_operation_id),
    (requests | where operation_Id == my_operation_id),
    (dependencies | where operation_Id  == my_operation_id),
    (exceptions | where operation_Id == my_operation_id)
};

union_all
    | order by timestamp desc
```

Si vous avez seulement `exceptions`, analysez les détails et regardez s’ils correspondent aux lignes de votre code. Si vous voyez seulement des exceptions provenant du connecteur de canal (`Microsoft.Bot.ChannelConnector`), consultez [Aucun événement Application Insights](#no-application-insights-events) pour vérifier qu’Application Insights est correctement configuré et que votre code journalise les événements.

## <a name="no-application-insights-events"></a>Aucun événement Application Insights

Si vous recevez des erreurs 500 sans aucun autre événement de votre bot signalé dans Application Insights, vérifiez les points suivants :

### <a name="ensure-bot-runs-locally"></a>Vérifier que le bot s’exécute localement

Vérifiez que votre bot s’exécute d’abord localement avec l’émulateur.

### <a name="ensure-configuration-files-are-being-copied-net-only"></a>Assurez-vous que les fichiers de configuration sont en cours de copie (.NET uniquement)

Assurez-vous que votre fichier de configuration `.bot` et que votre fichier `appsettings.json` sont correctement empaquetés pendant le processus de déploiement.

#### <a name="application-assemblies"></a>Assemblys d’application

Vérifiez que les assemblys Application Insights sont correctement empaquetés pendant le processus de déploiement.

- Microsoft.ApplicationInsights
- Microsoft.ApplicationInsights.TraceListener
- Microsoft.AI.Web
- Microsoft.AI.WebServer
- Microsoft.AI.ServeTelemetryChannel
- Microsoft.AI.PerfCounterCollector
- Microsoft.AI.DependencyCollector
- Microsoft.AI.Agent.Intercept

Assurez-vous que votre fichier de configuration `.bot` et que votre fichier `appsettings.json` sont correctement empaquetés pendant le processus de déploiement.

#### <a name="appsettingsjson"></a>appsettings.json

Dans votre fichier `appsettings.json`, vérifiez que la clé d'instrumentation est définie.

## <a name="aspnet-web-apitabdotnetwebapi"></a>[API web ASP.NET](#tab/dotnetwebapi)

```json
{
    "Logging": {
        "IncludeScopes": false,
        "LogLevel": {
            "Default": "Debug",
            "System": "Information",
            "Microsoft": "Information"
        },
        "Console": {
            "IncludeScopes": "true"
        }
    }
}
```

## <a name="aspnet-coretabdotnetcore"></a>[ASP.NET Core](#tab/dotnetcore)

```json
{
    "botFilePath": "mybot.bot",
    "botFileSecret": "<my secret>",
    "ApplicationInsights": {
        "InstrumentationKey": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
    }
}
```

---

### <a name="verify-bot-config-file"></a>Vérifier le fichier de configuration .bot

Vérifiez qu’une clé Application Insights se trouve dans votre fichier .bot.

```json
    {
        "type": "appInsights",
        "tenantId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "subscriptionId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "resourceGroup": "my resource group",
        "name": "my appinsights name",
        "serviceName": "my service name",
        "instrumentationKey": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "applicationId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "apiKeys": {},
        "id": ""
    },
```

### <a name="check-logs"></a>Inspecter les journaux

ASP.Net et Node émettent des journaux au niveau serveur que vous pouvez inspecter.

#### <a name="set-up-a-browser-to-watch-your-logs"></a>Configurer un navigateur pour consulter vos journaux

1. Ouvrez votre bot dans le [portail Azure](http://portal.azure.com/).
1. Ouvrez la page **Paramètres App Service / Tous les paramètres App Service** pour voir tous les paramètres de service.
1. Ouvrez la page **Supervision / Journaux de diagnostic** du service d’application.
   - Vérifiez que **Journal des applications (Filesystem)** est activé. Veillez à cliquer sur **Enregistrer** si vous changez ce paramètre.
1. Basculez sur la page **Supervision / Flux de journaux**.
   - Sélectionnez **Journaux du serveur web** et vérifiez qu’un message vous dit que vous êtes connecté. La commande doit ressembler à ceci :

     ```bash
     Connecting...
     2018-11-14T17:24:51  Welcome, you are now connected to log-streaming service.
     ```

     Gardez cette fenêtre ouverte.

#### <a name="set-up-browser-to-restart-your-bot-service"></a>Configurer un navigateur pour redémarrer votre service de bot

1. À l’aide d’un autre navigateur, ouvrez votre bot dans le portail Azure.
1. Ouvrez la page **Paramètres App Service / Tous les paramètres App Service** pour voir tous les paramètres de service.
1. Basculez sur la page **Vue d’ensemble** du service d’application et cliquez sur **Redémarrer**.
   - Un message vous demande si vous êtes sûr, sélectionnez **Oui**.
1. Revenez dans la première fenêtre du navigateur et consultez les journaux.
1. Vérifiez que vous recevez de nouveaux journaux.
   - Si vous ne voyez aucune activité, redéployez votre bot.
   - Basculez ensuite sur la page **Journaux des applications** et recherchez les erreurs.
