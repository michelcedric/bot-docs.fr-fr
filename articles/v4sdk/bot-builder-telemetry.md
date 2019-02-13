---
title: Ajouter des données de télémétrie à votre bot | Microsoft Docs
description: Découvrez comment intégrer votre bot aux nouvelles fonctionnalités de télémétrie.
keywords: télémétrie, appinsights, bot monitor
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 02/06/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 4c268bc40b7dc3315232d8f695bdb79343b15e21
ms.sourcegitcommit: c7d2e939ec71f46f48383c750fddaf6627b6489d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 02/07/2019
ms.locfileid: "55795585"
---
# <a name="add-telemetry-to-your-bot"></a>Ajouter des données de télémétrie à votre bot
Dans la version 4.2 du SDK Bot Framework, la journalisation des données de télémétrie a été ajoutée au produit.  Elle permet aux applications bot d’envoyer des données d’événement à des services comme Application Insights.

Ce document explique comment intégrer votre bot aux nouvelles fonctionnalités de télémétrie.  

## <a name="using-bot-configuration-option-1-of-2"></a>Utilisation de la configuration du bot (Option 1 sur 2)
Il existe deux méthodes pour configurer votre bot.  La première part du principe que vous effectuez une intégration à Application Insights.

Le fichier de configuration du bot contient des métadonnées sur les services externes que le bot utilise quand il est en cours d’exécution.  Par exemple, CosmosDB, Application Insights ainsi que la connexion de service et les métadonnées de Language Understanding (LUIS) y sont stockés.   

Si vous voulez « stocker » Application Insights, sans qu’aucune configuration supplémentaire spécifique à Application Insights ne soit nécessaire (par exemple, des initialiseurs de télémétrie), transmettez l’objet de configuration du bot pendant l’initialisation.   Il s’agit de la méthode la plus simple pour initialiser et configurer Application Insights en vue de commencer le suivi des requêtes et des appels externes à d’autres services, ainsi que la corrélation des événements entre les services.

```csharp
// ASP.Net Core - Startup.cs

public void ConfigureServices(IServiceCollection services)
{
     ...
     // Add Application Insights - pass in the bot configuration
     services.AddBotApplicationInsights(botConfig);
     ...
}

public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
     app.UseBotApplicationInsights()
                 ...
                .UseDefaultFiles()
                .UseStaticFiles()
                .UseBotFramework();
                ...
}
```

```JavaScript
const appInsightsClient = new ApplicationInsightsTelemetryClient(botConfig);
```

**Application Insights ne figure pas dans la configuration du bot** Que se passe-t-il si la configuration de votre bot ne contient pas Application Insights ?  Pas de problème. Un client null qui n’agit pas sur les appels de méthode est utilisé par défaut.

**Plusieurs sections Application Insights** Vous avez plusieurs sections Application Insight dans la configuration de votre bot ?  Vous pouvez désigner quelle instance du service Application Insights vous voulez utiliser dans la configuration de votre bot.

```csharp
// ASP.Net Core - Startup.cs

public void ConfigureServices(IServiceCollection services)
{
     // Add Application Insights
     services.AddBotApplicationInsights(botConfig, "myAppInsights");
     ...
}
```

```JavaScript
const appInsightsClient = new ApplicationInsightsTelemetryClient(botConfig);
```

## <a name="overriding-the-telemetry-client-option-2-of-2"></a>Remplacement du client de télémétrie (Option 2 sur 2)

Si vous voulez personnaliser votre client Application Insights ou vous connecter à un service totalement distinct, vous devez configurer le système différemment.

**Modifier la configuration d’Application Insights**

```csharp

public void ConfigureServices(IServiceCollection services)
{
     ...
     // Create Application Insight Telemetry Client
     // with custom configuration.
     var telemetryClient = TelemetryClient(myCustomConfiguration)
     
     // Add Application Insights
     services.AddBotApplicationInsights(new BotTelemetryClient(telemetryClient), "InstrumentationKey");
```

```JavaScript
const appInsightsClient = new ApplicationInsightsTelemetryClient(botConfig);
```

**Utiliser une télémétrie personnalisée** Si vous voulez journaliser les événements de télémétrie générés par Bot Framework dans un système totalement distinct, créez une classe dérivée de l’interface de base et effectuez la configuration.  

```csharp
public void ConfigureServices(IServiceCollection services)
{
     ...
     // Create my IBotTelemetryClient-based logger
     var myTelemetryClient = MyTelemetryLogger();
     
     // Add Application Insights
     services.AddBotApplicationInsights(myTelemetryClient);
     ...
}
```

```JavaScript
const appInsightsClient = new ApplicationInsightsTelemetryClient(botConfig);
```

## <a name="add-custom-logging-to-your-bot"></a>Ajouter une journalisation personnalisée à votre bot

Une fois la nouvelle prise en charge de la journalisation des données de télémétrie configurée pour le bot, vous pouvez commencer à ajouter des données de télémétrie à votre bot.  `BotTelemetryClient` (en C#, `IBotTelemetryClient`) dispose de plusieurs méthodes pour journaliser différents types d’événements.  Si vous choisissez le type d’événement approprié, vous pouvez tirer parti des rapports Application Insights existants (si vous utilisez Application Insights).  Pour les scénarios généraux, `TraceEvent` est habituellement utilisé.  Les données journalisées à l’aide de `TraceEvent` arrivent dans la table `CustomEvent` dans Kusto.

Si vous utilisez un dialogue dans votre bot, chaque objet basé sur un dialogue (notamment les invites) contient une nouvelle propriété `TelemetryClient`.  C’est `BotTelemetryClient` qui vous permet d’effectuer la journalisation.  Ce n’est pas uniquement pour des raisons pratiques. Nous verrons plus loin dans cet article que si cette propriété est définie, `WaterfallDialogs` génère des événements.

### <a name="identifiers-and-custom-events"></a>Identificateurs et événements personnalisés

Lors de la journalisation d’événements dans Application Insights, les événements générés contiennent des propriétés par défaut que vous n’avez pas besoin de renseigner.  Par exemple, les propriétés `user_id` et `session_id` sont contenues dans chaque événement personnalisé (généré avec l’API `TraceEvent`).  Par ailleurs, `activitiId`, `activityType` et `channelId` sont également ajoutées.

>Remarque : Ces valeurs ne sont pas fournies aux clients de télémétrie personnalisée.

Propriété |Type | Détails
--- | --- | ---
`user_id`| `string` | [ChannelID](https://github.com/Microsoft/botframework-obi/blob/master/botframework-activity/botframework-activity.md#channel-id) + [From.Id](https://github.com/Microsoft/botframework-obi/blob/master/botframework-activity/botframework-activity.md#from)
`session_id`| `string`|  [ConversationID](https://github.com/Microsoft/botframework-obi/blob/master/botframework-activity/botframework-activity.md#conversation)
`customDimensions.activityId`| `string` | [ID d’activité du bot](https://github.com/Microsoft/botframework-obi/blob/master/botframework-activity/botframework-activity.md#id)
`customDimensions.activityType` | `string` | [Type d’activité du bot](https://github.com/Microsoft/botframework-obi/blob/master/botframework-activity/botframework-activity.md#channel-id)
`customDimensions.channelId` | `string` |  [ID de canal d’activité du bot](https://github.com/Microsoft/botframework-obi/blob/master/botframework-activity/botframework-activity.md#channel-id)

## <a name="waterfalldialog-events"></a>Événements WaterfallDialog

En plus de la génération de vos propres événements, l’objet `WaterfallDialog` du SDK génère maintenant des événements. La section suivante décrit les événements générés à partir de Bot Framework. Si la propriété `TelemetryClient` est définie sur `WaterfallDialog`, ces événements sont stockés.

Voici un exemple de la modification d’un échantillon (BasicBot) qui utilise `WaterfallDialog` pour journaliser des événements de télémétrie.  BasicBot utilise un modèle courant utilisé dans lequel un `WaterfallDialog` est placé dans un `ComponentDialog` (`GreetingDialog`).

```csharp
// IBotTelemetryClient is direct injected into our Bot
public BasicBot(BotServices services, UserState userState, ConversationState conversationState, IBotTelemetryClient telemetryClient)
...

// The IBotTelemetryClient passed to the GreetingDialog
...
Dialogs = new DialogSet(_dialogStateAccessor);
Dialogs.Add(new GreetingDialog(_greetingStateAccessor, telemetryClient));
...

// The IBotTelemetryClient to the WaterfallDialog
...
AddDialog(new WaterfallDialog(ProfileDialog, waterfallSteps) { TelemetryClient = telemetryClient });
...

```

Une fois que `WaterfallDialog` a configuré `IBotTelemetryClient`, il commence à journaliser des événements.

### <a name="customevent-waterfallstart"></a>CustomEvent : « WaterfallStart » 

Quand un WaterfallDialog commence, un événement `WaterfallStart` est journalisé.

- `user_id` ([provenant de l’initialiseur de télémétrie](https://aka.ms/telemetry-initializer))
- `session_id` ([provenant de l’initialiseur de télémétrie](https://aka.ms/telemetry-initializer))
- `customDimensions.activityId` ([provenant de l’initialiseur de télémétrie](https://aka.ms/telemetry-initializer))
- `customDimensions.activityType` ([provenant de l’initialiseur de télémétrie](https://aka.ms/telemetry-initializer))
- `customDimensions.channelId` ([provenant de l’initialiseur de télémétrie](https://aka.ms/telemetry-initializer))
- `customDimensions.DialogId` (Il s’agit de la valeur dialogId (string) passée dans votre cascade.  Vous pouvez la considérer comme le « type de cascade »)
- `customDimensions.InstanceID` (unique pour chaque instance de dialogue)

### <a name="customevent-waterfallstep"></a>CustomEvent : « WaterfallStep » 

Journalise les étapes individuelles d’un dialogue en cascade.

- `user_id` ([provenant de l’initialiseur de télémétrie](https://aka.ms/telemetry-initializer))
- `session_id` ([provenant de l’initialiseur de télémétrie](https://aka.ms/telemetry-initializer))
- `customDimensions.activityId` ([provenant de l’initialiseur de télémétrie](https://aka.ms/telemetry-initializer))
- `customDimensions.activityType` ([provenant de l’initialiseur de télémétrie](https://aka.ms/telemetry-initializer))
- `customDimensions.channelId` ([provenant de l’initialiseur de télémétrie](https://aka.ms/telemetry-initializer))
- `customDimensions.DialogId` (Il s’agit de la valeur dialogId (string) passée dans votre cascade.  Vous pouvez la considérer comme le « type de cascade »)
- `customDimensions.StepName` (un nom de méthode ou `StepXofY` en cas d’expression lambda)
- `customDimensions.InstanceID` (unique pour chaque instance de dialogue)

### <a name="customevent-waterfalldialogcomplete"></a>CustomEvent : « WaterfallDialogComplete »

Journalise quand un dialogue en cascade se termine.

- `user_id` ([provenant de l’initialiseur de télémétrie](https://aka.ms/telemetry-initializer))
- `session_id` ([provenant de l’initialiseur de télémétrie](https://aka.ms/telemetry-initializer))
- `customDimensions.activityId` ([provenant de l’initialiseur de télémétrie](https://aka.ms/telemetry-initializer))
- `customDimensions.activityType` ([provenant de l’initialiseur de télémétrie](https://aka.ms/telemetry-initializer))
- `customDimensions.channelId` ([provenant de l’initialiseur de télémétrie](https://aka.ms/telemetry-initializer))
- `customDimensions.DialogId` (Il s’agit de la valeur dialogId (string) passée dans votre cascade.  Vous pouvez la considérer comme le « type de cascade »)
- `customDimensions.InstanceID` (unique pour chaque instance de dialogue)

### <a name="customevent-waterfalldialogcancel"></a>CustomEvent : « WaterfallDialogCancel » 

Journalise quand un dialogue en cascade est annulé.

- `user_id` ([provenant de l’initialiseur de télémétrie](https://aka.ms/telemetry-initializer))
- `session_id` ([provenant de l’initialiseur de télémétrie](https://aka.ms/telemetry-initializer))
- `customDimensions.activityId` ([provenant de l’initialiseur de télémétrie](https://aka.ms/telemetry-initializer))
- `customDimensions.activityType` ([provenant de l’initialiseur de télémétrie](https://aka.ms/telemetry-initializer))
- `customDimensions.channelId` ([provenant de l’initialiseur de télémétrie](https://aka.ms/telemetry-initializer))
- `customDimensions.DialogId` (Il s’agit de la valeur dialogId (string) passée dans votre cascade.  Vous pouvez la considérer comme le « type de cascade »)
- `customDimensions.StepName` (un nom de méthode ou `StepXofY` en cas d’expression lambda)
- `customDimensions.InstanceID` (unique pour chaque instance de dialogue)


## <a name="events-generated-by-the-bot-framework-service"></a>Événements générés par le service Bot Framework

En plus de `WaterfallDialog`, qui génère des événements à partir de votre code de bot, le service Bot Framework Channel journalise également des événements.  Cela vous permet de diagnostiquer les problèmes liés aux canaux ou l’ensemble des défaillances du bot.

### <a name="customevent-activity"></a>CustomEvent : « Activity »
**Journalisé à partir de :** Service Channel   Journalisé par le service Channel quand un message est reçu.

### <a name="exception-bot-errors"></a>Exception : « Bot Errors »
**Journalisé à partir de :** Service Channel   Journalisé par le canal quand un appel au bot retourne une réponse HTTP non 2XX.

## <a name="additional-events"></a>Événements supplémentaires

Le [Modèle d’entreprise](https://github.com/Microsoft/AI/tree/master/templates/Enterprise-Template) est le code open source qui peut être copié librement.  Il contient plusieurs composants qui peuvent être réutilisés et modifiés pour répondre à vos besoins en terme de rapport.

### <a name="customevent-botmessagereceived"></a>CustomEvent : BotMessageReceived 
**Journalisé à partir de :** TelemetryLoggerMiddleware (**Exemple Entreprise**)

Journalisé quand le bot reçoit un nouveau message.

- UserID ([provenant de l’initialiseur de télémétrie](https://aka.ms/telemetry-initializer))
- ConversationID ([provenant de l’initialiseur de télémétrie](https://aka.ms/telemetry-initializer))
- ActivityID ([provenant de l’initialiseur de télémétrie](https://aka.ms/telemetry-initializer))
- Channel ([provenant de l’initialiseur de télémétrie](https://aka.ms/telemetry-initializer))
- ActivityType ([provenant de l’initialiseur de télémétrie](https://aka.ms/telemetry-initializer))
- Text (facultatif pour PII)
- FromId
- FromName
- RecipientId
- RecipientName
- ConversationId
- ConversationName
- Locale

### <a name="customevent-botmessagesend"></a>CustomEvent : BotMessageSend 
**Journalisé à partir de :** TelemetryLoggerMiddleware (**Exemple Entreprise**)

Journalisé quand le bot envoie un message.

- UserID ([provenant de l’initialiseur de télémétrie](https://aka.ms/telemetry-initializer))
- ConversationID ([provenant de l’initialiseur de télémétrie](https://aka.ms/telemetry-initializer))
- ActivityID ([provenant de l’initialiseur de télémétrie](https://aka.ms/telemetry-initializer))
- Channel ([provenant de l’initialiseur de télémétrie](https://aka.ms/telemetry-initializer))
- ActivityType ([provenant de l’initialiseur de télémétrie](https://aka.ms/telemetry-initializer))
- ReplyToID
- Channel (Canal source ; par exemple, Skype, Cortana, Teams)
- RecipientId
- ConversationName
- Paramètres régionaux
- Text (facultatif pour PII)
- RecipientName (facultatif pour PII)

### <a name="customevent-botmessageupdate"></a>CustomEvent : BotMessageUpdate
**Journalisé à partir de :** TelemetryLoggerMiddleware   Journalisé quand un message est mis à jour par le bot (cas rare)

### <a name="customevent-botmessagedelete"></a>CustomEvent : BotMessageDelete
**Journalisé à partir de :** TelemetryLoggerMiddleware   Journalisé quand un message est supprimé par le bot (cas rare)

### <a name="customevent-luisintentinentname"></a>CustomEvent : LuisIntent.INENTName 
**Journalisé à partir de :** TelemetryLuisRecognizer (**Exemple Entreprise**)

Journalise les résultats du service LUIS.

- UserID ([provenant de l’initialiseur de télémétrie](https://aka.ms/telemetry-initializer))
- ConversationID ([provenant de l’initialiseur de télémétrie](https://aka.ms/telemetry-initializer))
- ActivityID ([provenant de l’initialiseur de télémétrie](https://aka.ms/telemetry-initializer))
- Channel ([provenant de l’initialiseur de télémétrie](https://aka.ms/telemetry-initializer))
- ActivityType ([provenant de l’initialiseur de télémétrie](https://aka.ms/telemetry-initializer))
- Intention
- IntentScore
- Question
- ConversationId
- SentimentLabel
- SentimentScore
- *Entités LUIS*
- **NOUVEAU** DialogId

### <a name="customevent-qnamessage"></a>CustomEvent : QnAMessage
**Journalisé à partir de :** TelemetryQnaMaker (**Exemple Entreprise**)

Journalise les résultats du service QnA Maker.

- UserID ([provenant de l’initialiseur de télémétrie](https://aka.ms/telemetry-initializer))
- ConversationID ([provenant de l’initialiseur de télémétrie](https://aka.ms/telemetry-initializer))
- ActivityID ([provenant de l’initialiseur de télémétrie](https://aka.ms/telemetry-initializer))
- Channel ([provenant de l’initialiseur de télémétrie](https://aka.ms/telemetry-initializer))
- ActivityType ([provenant de l’initialiseur de télémétrie](https://aka.ms/telemetry-initializer))
- Nom d’utilisateur
- ConversationId
- OriginalQuestion
- Question
- Réponse
- Score (*Facultatif* : si nous avons trouvé des connaissances)

## <a name="querying-the-data"></a>Interrogation des données
Quand vous utilisez Application Insights, toutes les données sont corrélées (même entre les services).  Nous pouvons le constater en interrogeant une requête réussie et voir tous les événements associés pour cette requête.  
Les requêtes suivantes vous indiquent les requêtes les plus récentes :
```sql
requests 
| where timestamp > ago(3d) 
| where resultCode == 200
| order by timestamp desc
| project timestamp, operation_Id, appName
| limit 10
```

Avec la première requête, sélectionnez quelques-unes des valeurs `operation_Id` et recherchez plus d’informations :

```sql
let my_operation_id = "<OPERATION_ID>";
let union_all = () {
    union
    (traces | where operation_Id == my_operation_id),
    (customEvents | where operation_Id == my_operation_id),
    (requests | where operation_Id == my_operation_id),
    (dependencies | where operation_Id  == my_operation_id),
    (exceptions | where operation_Id == my_operation_id)
};
union_all
    | order by timestamp asc
    | project itemType, name, performanceBucket
```

Cela vous donne la répartition chronologique d’une requête unique, avec le compartiment de durée de chaque appel.
![Exemple d’appel](media/performance_query.png)

> Remarque : L’horodatage d’événement `customEvent` « Activity » est hors service, car ces événements sont journalisés de façon asynchrone.

## <a name="create-a-dashboard"></a>Création d’un tableau de bord

La façon la plus simple d’effectuer un test consiste à créer un tableau de bord à l’aide de la [page de déploiement de modèle du portail Azure](https://portal.azure.com/#create/Microsoft.Template).  
- Cliquer sur ["Créer votre propre modèle dans l’éditeur"]
- Copiez et collez l’un des fichiers .json suivants qui sont fournis pour vous aider à créer le tableau de bord :
  - [Tableau de bord d’intégrité du système](https://aka.ms/system-health-appinsights)
  - [Tableau de bord d’intégrité de conversation](https://aka.ms/conversation-health-appinsights)
- Cliquez sur « Enregistrer »
- Remplir `Basics` : 
   - Abonnement : <your test subscription>
   - Groupe de ressources : <a test resource group>
   - Lieu : <such as West US>
- Remplir `Settings` :
   - Nom du composant Insights : <like `core672so2hw`>
   - Groupe de ressources de composant Insights : <like `core67`>
   - Nom du tableau de bord : <like `'ConversationHealth'` ou `SystemHealth`>
- Cliquez sur `I agree to the terms and conditions stated above`
- Cliquez sur `Purchase`
- Valider
   - Cliquez sur [`Resource Groups`](https://ms.portal.azure.com/#blade/HubsExtension/Resources/resourceType/Microsoft.Resources%2Fsubscriptions%2FresourceGroups)
   - Sélectionnez votre groupe de ressources ci-dessus (comme `core67`).
   - Si vous ne voyez pas de nouvelle ressource, regardez « Déploiements » pour voir s’il y a des échecs.
   - Voici ce que vous voyez généralement concernant les échecs :
     
```json
{"code":"DeploymentFailed","message":"At least one resource deployment operation failed. Please list deployment operations for details. Please see https://aka.ms/arm-debug for usage details.","details":[{"code":"BadRequest","message":"{\r\n \"error\": {\r\n \"code\": \"InvalidTemplate\",\r\n \"message\": \"Unable to process template language expressions for resource '/subscriptions/45d8a30e-3363-4e0e-849a-4bb0bbf71a7b/resourceGroups/core67/providers/Microsoft.Portal/dashboards/Bot Analytics Dashboard' at line '34' and column '9'. 'The template parameter 'virtualMachineName' is not found. Please see https://aka.ms/arm-template/#parameters for usage details.'\"\r\n }\r\n}"}]}
```

Pour afficher les données, accédez au portail Azure. Cliquez sur **Tableau de bord** à gauche, puis sélectionnez le tableau de bord que vous avez créé dans la liste déroulante.

## <a name="additional-resources"></a>Ressources supplémentaires
Vous pouvez vous référer à ces exemples qui implémentent des mesures de télémétrie :
- C#
  - [LUIS avec AppInsights](https://aka.ms/luis-with-appinsights-cs)
  - [QnA avec AppInsights](https://aka.ms/qna-with-appinsights-cs)
- JS
  - [LUIS avec AppInsights](https://aka.ms/luis-with-appinsights-js)
  - [QnA avec AppInsights](https://aka.ms/qna-with-appinsights-js)

