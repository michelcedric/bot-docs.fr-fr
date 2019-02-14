---
title: Personnalisation de bot d’entreprise | Microsoft Docs
description: Découvrez comment personnaliser le bot créé par le modèle d’entreprise
author: darrenj
ms.author: darrenj
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 02/7/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: d472cbe7c0235862f8dcff1bcc2d53d977bb7657
ms.sourcegitcommit: 8183bcb34cecbc17b356eadc425e9d3212547e27
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 02/09/2019
ms.locfileid: "55971489"
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
    | - DeploymentScripts   // msbot clone recipes for deployment
        | - de              // Deployment files for German
        | - en              // Deployment files for English        
        | - es              // Deployment files for Spanish
        | - fr              // Deployment files for French
        | - it              // Deployment files for Italian
        | - zh              // Deployment files for Chinese
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

Deux modèles cognitifs sont inclus par défaut dans le modèle d’entreprise, un exemple de base de connaissances QnA Maker FAQ et un modèle LUIS pour les intentions générales (salutations, aide, annulation, etc.). Ces modèles peuvent être personnalisés pour répondre à vos besoins. Vous pouvez également ajouter des modèles LUIS et des bases de connaissances QnA Maker pour étendre les capacités de votre bot.

### <a name="updating-an-existing-luis-model"></a>Mise à jour d’un modèle LUIS existant
Afin de mettre à jour un modèle LUIS existant pour le modèle d’entreprise, procédez comme suit :
1. Modifiez le modèle LUIS dans le [portail LUIS](http://luis.ai) ou via les outils de CLI [LuDown](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/Ludown) et [Luis](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/LUIS). 
2. Exécutez la commande suivante pour mettre à jour votre modèle Dispatch pour qu’il reflète vos modifications (garantit un routage des messages approprié) :
```shell
    dispatch refresh --bot "YOUR_BOT.bot" --secret YOUR_SECRET
```
3. Exécutez la commande suivante à partir de la racine de votre projet pour chaque modèle mis à jour, afin de mettre à jour leurs classes LuisGen associées : 
```shell
    luis export version --appId [LUIS_APP_ID] --versionId [LUIS_APP_VERSION] --authoringKey [YOUR_LUIS_AUTHORING_KEY] | luisgen --cs [CS_FILE_NAME] -o "\Dialogs\Shared\Resources"
```

### <a name="updating-an-existing-qna-maker-knowledge-base"></a>Mise à jour d’une base de connaissances QnA Maker existante
Pour mettre à jour une base de connaissances QnA Maker existante, effectuez les étapes suivantes :
1. Modifiez votre base de connaissances QnA Maker par le biais des outils de CLI [LuDown](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/Ludown) et [QnA Maker](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/QnAMaker) ou le [portail QnA Maker](https://qnamaker.ai).
2. Exécutez la commande suivante pour mettre à jour votre modèle Dispatch pour qu’il reflète vos modifications (garantit un routage des messages approprié) :
```shell
    dispatch refresh --bot "YOUR_BOT.bot" --secret YOUR_SECRET
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
    dispatch add -t luis -id LUIS_APP_ID -bot "YOUR_BOT.bot" --secret YOURSECRET
```
4. Actualisez le modèle dispatch pour qu’il reflète les modifications du modèle LUIS via la commande suivante
```shell
    dispatch refresh -bot "YOUR_BOT.bot" --secret YOUR_SECRET
```

### <a name="adding-an-additional-qna-maker-knowledge-base"></a>Ajout d’une base de connaissances QnA Maker supplémentaire

Dans certains scénarios, vous souhaiterez ajouter une base de connaissances QnA Maker supplémentaire à votre bot. Cette opération peut être effectuée en suivant la procédure ci-après.

1. Créer une base de connaissances QnA Maker à partir d’un fichier JSON à l’aide de la commande suivante, exécutée dans le répertoire de votre assistant
```shell
qnamaker create kb --in <KB.json> --msbot | msbot connect qna --stdin --bot "YOUR_BOT.bot" --secret YOURSECRET
```
2. Exécuter la commande suivante pour mettre à jour votre modèle Dispatch pour qu’il reflète vos modifications
```shell
dispatch refresh --bot "YOUR_BOT.bot" --secret YOUR_SECRET
```
3. Mettre à jour la classe Dispatch fortement typée pour refléter la nouvelle source de QnA
```shell
msbot get dispatch --bot "YOUR_BOT.bot" | luis export version --stdin > dispatch.json
luisgen dispatch.json -cs Dispatch -o Dialogs\Shared
```
4.  Mettez à jour le fichier `Dialogs\Main\MainDialog.cs` pour inclure l’intention Dispatch correspondant à votre nouvelle source QnA en suivant l’exemple fourni.

Vous devez maintenant être en mesure d’utiliser plusieurs sources QnA dans votre bot.

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
    AddDialog(new TextPrompt(DialogIds.NamePrompt));
    AddDialog(new TextPrompt(DialogIds.EmailPrompt));
    AddDialog(new TextPrompt(DialogIds.LocationPrompt));
```

Ensuite, vous devez créer le gestionnaire de modèles pour gérer les réponses. Créez une classe et dérivez à partir de TemplateManager, un exemple est fourni dans le fichier OnboardingResponses.cs et un extrait est illustré ci-dessous.

```    
 ["default"] = new TemplateIdMap
            {
                { ResponseIds.EmailPrompt,
                    (context, data) =>
                    MessageFactory.Text(
                        text: OnboardingStrings.EMAIL_PROMPT,
                        ssml: OnboardingStrings.EMAIL_PROMPT,
                        inputHint: InputHints.ExpectingInput)
                },
                { ResponseIds.HaveEmailMessage,
                    (context, data) =>
                    MessageFactory.Text(
                        text: string.Format(OnboardingStrings.HAVE_EMAIL, data.email),
                        ssml: string.Format(OnboardingStrings.HAVE_EMAIL, data.email),
                        inputHint: InputHints.IgnoringInput)
                },
                { ResponseIds.HaveLocationMessage,
                    (context, data) =>
                    MessageFactory.Text(
                        text: string.Format(OnboardingStrings.HAVE_LOCATION, data.Name, data.Location),
                        ssml: string.Format(OnboardingStrings.HAVE_LOCATION, data.Name, data.Location),
                        inputHint: InputHints.IgnoringInput)
                },
                
                ...
```

Afin d’afficher les réponses, vous pouvez utiliser une instance du gestionnaire de modèles pour accéder à ces réponses via `ReplyWith` ou `RenderTemplate` pour les invites. Des exemples sont présentés ci-dessous.

```
Prompt = await _responder.RenderTemplate(sc.Context, sc.Context.Activity.Locale, OnboardingResponses.ResponseIds.NamePrompt)
await _responder.ReplyWith(sc.Context, OnboardingResponses.ResponseIds.HaveNameMessage, new { name });
```

La dernière partie de l’infrastructure Dialogue est la création d’une classe d’état limitée à votre dialogue uniquement. Créez une classe et assurez-vous qu’elle dérive de `DialogState`

Une fois que votre dialogue est terminé, vous devez ajouter le dialogue à votre composant `MainDialog` à l’aide de `AddDialog`. Pour utiliser votre nouveau dialogue, appelez `dc.BeginDialogAsync()` depuis votre méthode `RouteAsync`, qui se déclenche avec l’intention LUIS si vous le souhaitez.

## <a name="conversational-insights-using-powerbi-dashboard-and-application-insights"></a>Insights conversationnelles à l’aide du tableau de bord Power BI et d’Application Insights
- Pour commencer à obtenir des insights conversationnelles, poursuivez avec l’article [Enterprise Bot Template - Conversational Analytics using PowerBI Dashboard and Application Insights](bot-builder-enterprise-template-powerbi.md) (Modèle de bot d’entreprise - Analytique de conversation à l’aide du tableau de bord PowerBI et d’Application Insights).
