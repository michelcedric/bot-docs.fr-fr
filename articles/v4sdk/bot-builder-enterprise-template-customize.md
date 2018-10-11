---
title: Personnalisation de bot d’entreprise | Microsoft Docs
description: Découvrez comment personnaliser le bot créé par le modèle d’entreprise
author: darrenj
ms.author: darrenj
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 09/18/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 6fc7f73d406c1bbbc2b2671c9336df6bda10ade6
ms.sourcegitcommit: 87b5b0ca9b0d5e028ece9f7cc4948c5507062c2b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/24/2018
ms.locfileid: "47029757"
---
# <a name="enterprise-bot-template---customize-your-bot"></a>Modèle de bot d’entreprise - Personnaliser votre bot

> [!NOTE]
> Cet article s’applique à la version v4 du Kit de développement logiciel (SDK). 

## <a name="net"></a>.NET
Après avoir déployé le modèle de bot d’entreprise et vérifié qu’il fonctionne complètement, comme indiqué dans les instructions [ici](bot-builder-enterprise-template-deployment.md), vous pouvez facilement personnaliser votre bot en fonction de votre scénario et de vos besoins. L’objectif du modèle consiste à fournir une base solide sur laquelle créer votre expérience de conversation.

## <a name="project-structure"></a>Structure du projet

La structure du dossier de votre bot est indiquée ci-dessous et représente nos meilleures pratiques recommandées pour structurer votre projet de bot et traiter les messages entrants.

    | - YourBot.bot         // The .bot file containing all of your Bot configuration including dependencies
    | - README.md           // README file containing links to documentation
    | - Program.cs          // Default Program.cs file
    | - Startup.cs          // Core Bot Initialisation including Bot Configuration LUIS, Dispatcher, etc. 
    | - <BOTNAME>State.cs   // The Root State class for your Bot
    | - appsettings.json    // References above .bot file for Configuration information. App Insights key
    | - CognitiveModels     
        | - LUIS            // .LU file containing base conversational intents (Greeting, Help, Cancel)
        | - QnA             // .LU file containing example QnA items
    | - DeploymentScripts   // msbot clone recipe for deployment
    | - Dialogs             // All Bot dialogs sit under this folder
        | - Main            // Root Dialog for all messages
            | - MainDialog.cs       // Dialog Logic
            | - MainResponses.cs    // Dialog responses
            | - Resources           // Adaptive Card JSON, Resource File
        | - Onboarding
            | - OnboardingDialog.cs       // Onboarding dialog Logic
            | - OnboardingResponses.cs    // Onboarding dialog responses
            | - OnboardingState.cs        // Localised dialog state
            | - Resources                 Resource File
        | - Cancel
        | - Escalate
        | - Signin
    | - Middleware          // Telemetry, Content Moderator
    | - ServiceClients      // SDK libraries, example GraphClient provided for Auth example
   
## <a name="update-introduction-message"></a>Mettre à jour le message de présentation

Le message de présentation utilise une [carte adaptative](https://www.adaptivecards.io). Pour personnaliser cette présentation de votre bot, vous pouvez rechercher le fichier JSON nommé ```Intro.json``` dans le dossier Dialogues/Principal/Ressources. Utilisez le [visualiseur de carte adaptative](http://adaptivecards.io/visualizer) pour modifier la carte adaptative en fonction des besoins de votre bot.

## <a name="update-bot-responses"></a>Mettre à jour les réponses de bot

Chaque dialogue dans le projet possède un ensemble de réponses stocké au sein des fichiers de ressources de prise en charge (.resx). Ils peuvent se trouver dans le dossier Ressources sous chaque dialogue.

Vous pouvez modifier les réponses dans l’éditeur de ressources Visual Studio, comme indiqué ci-dessous, pour ajuster la façon dont votre bot répond.

![Personnalisation des réponses de bot](media/enterprise-template/EnterpriseBot-CustomisingResponses.png)

Cette approche prend en charge les réponses multilingues à l’aide de l’approche de localisation des fichiers de ressources standard. Des informations supplémentaires sont disponibles [ici.](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/localization?view=aspnetcore-2.1)

## <a name="updating-your-cognitive-models"></a>Mise à jour de vos modèles cognitifs

Deux modèles cognitifs sont inclus par défaut dans le modèle d’entreprise, un exemple de base de connaissances QnAMaker FAQ et un modèle LUIS pour les intentions générales (salutations, aide, annulation, etc.). Ces modèles peuvent être personnalisés pour répondre à vos besoins. Vous pouvez également ajouter des modèles LUIS et des bases de connaissances QnAMaker pour étendre les capacités de votre bot.

### <a name="updating-an-existing-luis-model"></a>Mise à jour d’un modèle LUIS existant
Afin de mettre à jour un modèle LUIS existant pour le modèle d’entreprise, procédez comme suit :
1. Modifiez le modèle LUIS dans le [portail LUIS](http://luis.ai) ou via les outils de CLI [LuDown](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/Ludown) et [Luis](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/LUIS). 
2. Exécutez la commande suivante pour mettre à jour votre modèle Dispatch pour qu’il reflète vos modifications (garantit un routage des messages approprié) :
```shell
    dispatch refresh --bot "YOURBOT.bot" --secret YOURSECRET
```
3. Exécutez la commande suivante à partir de la racine de votre projet pour chaque modèle mis à jour, afin de mettre à jour leurs classes LuisGen associées : 
```shell
    luis export version --appId [LUIS_APP_ID] --versionId [LUIS_APP_VERSION] --authoringKey [YOUR_LUIS_AUTHORING_KEY] | luisgen - -cs [CS_FILE_NAME] -o "\Dialogs\Shared\Resources"
```

### <a name="updating-an-existing-qnamaker-knowledge-base"></a>Mise à jour d’une base de connaissances QnAMaker existante
Pour mettre à jour une base de connaissances QnAMaker existante, procédez comme suit :
1. Modifiez votre base de connaissances QnAMaker via les outils de CLI [LuDown](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/Ludown) et [QnAMaker](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/QnAMaker) ou le [portail QnAMaker](https://qnamaker.ai).
2. Exécutez la commande suivante pour mettre à jour votre modèle Dispatch pour qu’il reflète vos modifications (garantit un routage des messages approprié) :
```shell
    dispatch refresh --bot "YOURBOT.bot" --secret YOURSECRET
```

### <a name="adding-a-new-luis-model"></a>Ajout d’un modèle LUIS

Dans les scénarios où vous souhaitez ajouter un modèle LUIS à votre projet, vous devez mettre à jour la configuration du bot et Dispatcher pour vous assurer qu’ils prennent en compte le nouveau modèle. 
1. Créez votre modèle LUIS via les outils de CLI LuDown/LUIS ou via le portail LUIS
2. Exécutez la commande suivante pour connecter votre nouvelle application LUIS à votre fichier .bot :
```shell
    msbot connect luis --appId [LUIS_APP_ID] --authoringKey [LUIS_AUTHORING_KEY] --subscriptionKey [LUIS_SUBSCRIPTION_KEY] 
```
3. Ajoutez ce nouveau modèle LUIS à Dispatcher via la commande suivante
```shell
    dispatch add -t luis -id YOUR_LUIS_APPID -bot "YOURBOT.bot" -secret YOURSECRET
```
4. Actualisez le modèle dispatch pour qu’il reflète les modifications du modèle LUIS via la commande suivante
```shell
    dispatch refresh -bot "YOURBOT.bot" -secret YOURSECRET
```

## <a name="adding-a-new-dialog"></a>Ajout d’un dialogue 

Pour ajouter un dialogue à votre bot, vous devez d’abord créer un dossier sous Dialogues et vous assurer que cette classe dérive de `EnterpriseDialog`. Vous devez ensuite raccorder l’infrastructure Dialogue. Le dialogue Mise en route est un exemple simple auquel vous pouvez vous référer, et vous en trouverez un extrait ci-après avec une présentation des étapes.

- Ajoutez un dialogue en cascade à votre constructeur
- Définissez les étapes de votre cascade
- Créez vos étapes en cascade
- Appelez AddDialog, qui transmet votre cascade
- Appeler AddDialog, qui transmet les invites que vous utilisez dans votre cascade
- Définissez l’élément InitialDialogId pour le premier dialogue que le composant doit exécuter

```
InitialDialogId = nameof(OnboardingDialog);

var onboarding = new WaterfallStep[]
{
    AskForName,
    AskForEmail,
    AskForLocation,
    FinishOnboardingDialog,
};

AddDialog(new WaterfallDialog(InitialDialogId, onboarding));
AddDialog(new TextPrompt(NamePrompt));
AddDialog(new TextPrompt(EmailPrompt));
AddDialog(new TextPrompt(LocationPrompt));
```

Ensuite, vous devez créer le gestionnaire de modèles pour gérer les réponses. Créez une classe et dérivez à partir de TemplateManager, un exemple est fourni dans le fichier OnboardingResponses.cs et un extrait est illustré ci-dessous.

```
public const string _namePrompt = "namePrompt";
public const string _haveName = "haveName";
public const string _emailPrompt = "emailPrompt";
      
private static LanguageTemplateDictionary _responseTemplates = new LanguageTemplateDictionary
{
    ["default"] = new TemplateIdMap
    {
        {
            _namePrompt,
            (context, data) => OnboardingStrings.NAME_PROMPT
        },
        {
            _haveName,
            (context, data) => string.Format(OnboardingStrings.HAVE_NAME, data.name)
        },
        {
            _emailPrompt,
            (context, data) => OnboardingStrings.EMAIL_PROMPT
        },
```

Afin d’afficher les réponses, vous pouvez utiliser une instance du gestionnaire de modèles pour accéder à ces réponses via `ReplyWith` ou `RenderTemplate` pour les invites. Des exemples sont présentés ci-dessous.

```
Prompt = await _responder.RenderTemplate(sc.Context, "en", OnboardingResponses._namePrompt),
await _responder.ReplyWith(sc.Context, OnboardingResponses._haveName, new { name });
```

La dernière partie de l’infrastructure Dialogue est la création d’une classe d’état limitée à votre dialogue uniquement. Créez une classe et assurez-vous qu’elle dérive de `DialogState`

Une fois que votre dialogue est terminé, vous devez ajouter le dialogue à votre composant `MainDialog` à l’aide de `AddDialog`. Pour utiliser votre nouveau dialogue, appelez `dc.BeginDialogAsync()` depuis votre méthode `RouteAsync`, qui se déclenche avec l’intention LUIS si vous le souhaitez.

## <a name="conversational-insights-using-powerbi-dashboard-and-application-insights"></a>Insights conversationnelles à l’aide du tableau de bord Power BI et d’Application Insights
- Pour commencer à obtenir des insights conversationnelles, poursuivez avec l’article [Enterprise Bot Template - Conversational Analytics using PowerBI Dashboard and Application Insights](bot-builder-enterprise-template-powerbi.md) (Modèle de bot d’entreprise - Analytique de conversation à l’aide du tableau de bord PowerBI et d’Application Insights).

