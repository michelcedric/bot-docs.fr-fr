---
title: Déploiement d’un modèle de bot d’entreprise | Microsoft Docs
description: Découvrez comment déployer toutes les ressources Azure compatibles pour votre bot d’entreprise.
author: darrenj
ms.author: darrenj
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 09/18/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 622d5b4924a4739900659d6be8eebd1888191eab
ms.sourcegitcommit: ddc8c116887ada67642d49ee5166e7f1ae287263
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/08/2019
ms.locfileid: "54114883"
---
# <a name="enterprise-bot-template---deploying-your-bot"></a>Modèle de bot d’entreprise – Déployer votre bot

> [!NOTE]
> Cet article s’applique à la version v4 du Kit de développement logiciel (SDK). 

## <a name="prerequisites"></a>Prérequis

- Vérifiez que vous avez mis [.NET Core](https://www.microsoft.com/net/download) au niveau de la dernière version.

- Vérifiez que le [gestionnaire de package Node](https://nodejs.org/en/) est installé.

- Installez les outils de ligne de commande (CLI) Azure Bot Service. Même si vous les avez utilisés auparavant, installez-les de manière à disposer des dernières versions.

```shell
npm install -g ludown luis-apis qnamaker botdispatch msbot chatdown
```

- Installez les outils de ligne de commande (CLI) Azure à partir de [cette page](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli-windows?view=azure-cli-latest). Si vous avez déjà installé l’outil de ligne de commande (CLI) Azure Bot Service, assurez-vous de le mettre à jour vers la toute dernière version en désinstallant votre version actuelle, puis en installant la nouvelle.

> Avec msbot 4.3.2 et ultérieur, le seul prérequis AZ CLI est d’avoir une version AZ CLI > = 2.0.53. Si vous avez l’extension botservice également installée, supprimez-la via « az extension remove --name botservice ».

- Installer l’outil LUISGen

```shell
dotnet tool install -g luisgen
```

## <a name="configuration"></a>Configuration

- Récupérer votre clé de création LUIS
   - Consultez [cette page](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/luis-reference-regions) de documentation pour connaître le portail LUIS correspondant à la région de votre déploiement. Notez que www.luis.ai fait référence à la région des États-Unis et qu’une clé de création récupérée sur ce portail ne fonctionnera pas pour un déploiement en Europe.
   - Une fois connecté, cliquez sur votre nom dans l’angle supérieur droit.
   - Cliquez sur Paramètres et prenez note de la clé de création pour l’étape suivante.

## <a name="deployment"></a>Déploiement

> Si vous disposez de plusieurs abonnements Azure et souhaitez vous assurer que le bon abonnement est associé au déploiement, exécutez les commandes suivantes avant de continuer.

 Suivez le processus de connexion du navigateur dans votre compte Azure.
```shell
az login
az account list
az account set --subscription "YOUR_SUBSCRIPTION_NAME"
```

Les bots de modèle d’entreprise ont besoin des dépendances suivantes pour les opérations de bout en bout.
- Application web Azure
- Compte de stockage Azure (transcriptions)
- Azure Application Insights (données de télémétrie)
- Azure CosmosDb (état)
- Azure Cognitive Services – Language Understanding
- Azure Cognitive Services – QnA Maker (notamment Recherche Azure et Azure Web App)
- Azure Cognitive Services – Content Moderator (étape facultative manuelle)

Avec la configuration du déploiement de votre nouveau projet de bot, la commande `msbot clone services` peut automatiser le déploiement de tous les services ci-dessus dans votre abonnement Azure et vérifier que le fichier .bot de votre projet est mis à jour avec tous les services, y compris les clés nécessaires au bon fonctionnement de votre bot. Ella a également plusieurs options de configuration pour les langues suivantes : Allemand, anglais, chinois, espagnol, français et italien.

> Après le déploiement, vérifiez les niveaux tarifaires pour les services créés et ajustez-les en fonction de votre scénario.

Dans le projet que vous avez créé, le fichier README.md contient un exemple de ligne de commande `msbot clone services` qui est mis à jour avec le nom du bot créé. Une version générique est indiquée ci-dessous. Veillez à mettre à jour la clé de création de l’étape précédente et choisissez l’emplacement de centre de données Azure que vous souhaitez utiliser (par exemple : westus ou westeurope). Vérifiez que la clé de création LUIS récupérée à l’étape précédente correspond bien à la région spécifiée ci-dessous (par exemple, westus pour luis.ai ou westeurope pour eu.luis.ai). Enfin, référencez le dossier de la langue que vous souhaitez utiliser (par exemple, `DeploymentScripts\en`).

```shell
msbot clone services --name "YOUR_BOT_NAME" --luisAuthoringKey "YOUR_AUTHORING_KEY" --folder "DeploymentScripts\LOCALE_FOLDER" --location "REGION"
```

> Certains utilisateurs peuvent rencontrer un problème connu, dans lequel l’erreur suivante apparaît lors du déploiement `ERROR: Unable to provision MSA id automatically. Please pass them in as parameters and try again`. Dans ce cas, accédez à https://apps.dev.microsoft.com et créez manuellement une nouvelle application pour récupérer l’ID d’application et le mot de passe/secret. Exécutez la commande msbot clone services ci-dessus, mais indiquez les deux nouveaux arguments `appId` et `appSecret` avec les valeurs que vous venez de récupérer. Vérifiez que vous placez le secret entre guillemets pour éviter les problèmes d’analyse, par exemple : `-appSecret "YOUR_SECRET"`

L’outil msbot décrit le plan de déploiement, notamment l’emplacement et le SKU. Vérifiez ces informations avec de continuer.

![Confirmation du déploiement](./media/enterprise-template/EnterpriseBot-ConfirmDeployment.png)

>Au terme du déploiement, vous devez **impérativement** noter le secret du fichier .bot fourni, car vous en aurez besoin lors des étapes à venir. Vous pouvez également exécuter `msbot secret --clear --secret YOUR_BOT_SECRET` pour supprimer le secret de votre fichier bot et simplifier le développement jusqu’à ce que votre bot soit prêt pour la production. Exécutez `msbot secret --new` pour générer un nouveau secret.

- Mettez votre fichier `appsettings.json` à jour avec le nom du nouveau fichier .bot et le secret du fichier .bot.
- Exécutez la commande suivante, récupérez la clé d’instrumentation pour votre instance Application Insights et mettez à jour la clé d’instrumentation dans votre fichier `appsettings.json`.

`msbot list --bot YOUR_BOT_FILE.bot --secret "YOUR_BOT_SECRET"`

        {
          "botFilePath": ".\\YOUR_BOT_FILE.bot",
          "botFileSecret": "YOUR_BOT_SECRET",
          "ApplicationInsights": {
            "InstrumentationKey": "YOUR_INSTRUMENTATION_KEY"
          }
        }

## <a name="testing"></a>Test

Une fois terminé, exécutez votre projet de bot au sein de votre environnement de développement et ouvrez l’émulateur de Framework de bot. Dans l’émulateur, choisissez Ouvrir le bot dans le menu du fichier et accédez au fichier .bot dans votre répertoire.

Tapez ```hi``` pour vérifier que tout fonctionne.

En cas de problème avec l’émulateur Bot Framework, vérifiez d’abord que vous disposez de la toute dernière version de l’émulateur. Si votre ancienne version de l’émulateur ne se met pas à jour correctement, désinstallez-la et réinstallez l’émulateur.

## <a name="deploy-to-azure"></a>Déployer dans Azure

Le test peut être effectué localement de bout en bout. Lorsque vous êtes prêt à déployer votre bot dans Azure pour effectuer des tests supplémentaires, vous pouvez utiliser la commande suivante pour publier le code source. Vous pouvez l’exécuter dès que vous souhaitez envoyer des mises à jour du code source.

```shell
az bot publish -g YOUR_BOT_NAME -n YOUR_BOT_NAME --proj-file YOUR_BOT_NAME.csproj --sdk-version v4
```

## <a name="enabling-more-scenarios"></a>Scénarios supplémentaires

Votre projet de bot fournit des fonctionnalités supplémentaires que vous pouvez activer grâce aux étapes suivantes.

### <a name="authentication"></a>Authentification

Pour activer l’authentification, procédez comme suit après avoir configuré un nom de connexion d’authentification dans les paramètres de votre bot dans le Portail Azure. Vous trouverez des informations supplémentaires dans [la documentation](https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-authentication?view=azure-bot-service-4.0&tabs=csharp).

Inscrivez `AuthenticationDialog` dans le constructeur MainDialog :
    
`AddDialog(new AuthenticationDialog(_services.AuthConnectionName));`

Ajoutez les éléments suivants dans votre code, à l’emplacement souhaité, pour tester un flux de connexion simple :
    
`var authResult = await dc.BeginDialogAsync(nameof(AuthenticationDialog));`

### <a name="content-moderation"></a>Modération du contenu

La modération du contenu peut être utilisée pour identifier les informations d’identification personnelle (PII) et le contenu réservé aux adultes dans les messages envoyés au bot. Pour activer cette fonctionnalité, accédez au Portail Azure et créez un nouveau service modérateur de contenu. Recueillez votre clé d’abonnement et votre région pour configurer votre fichier .bot. 

> À l’avenir, cette étape sera automatisée.

Ajoutez le code suivant au bas de votre méthode service.AddBot<>() au démarrage pour activer la modération du contenu sur chaque tour. Le résultat de la modération du contenu est accessible via l’état du bot. 
    
```
    // Content Moderation Middleware (analyzes incoming messages for inappropriate content including PII, profanity, etc.)
    var moderatorService = botConfig.Services.Where(s => s.Name == ContentModeratorMiddleware.ServiceName).FirstOrDefault();
    if (moderatorService != null)
    {
        var moderator = moderatorService as GenericService;
        var moderatorKey = moderator.Configuration["subscriptionKey"];
        var moderatorRegion = moderator.Configuration["region"];
        var moderatorMiddleware = new ContentModeratorMiddleware(moderatorKey, moderatorRegion);
        options.Middleware.Add(moderatorMiddleware);
    }
```
Accédez au résultat de l’intergiciel en appelant la ligne suivante à partir de votre pile de dialogue.
```     
    var cm = dc.Context.TurnState.Get<Microsoft.CognitiveServices.ContentModerator.Models.Screen>(ContentModeratorMiddleware.TextModeratorResultKey);
```

## <a name="customize-your-bot"></a>Personnaliser votre bot

Après vous être assuré du bon déploiement du bot prédéfini, vous pouvez le personnaliser en fonction de votre scénario et de vos besoins. Poursuivre avec la [personnalisation du bot](bot-builder-enterprise-template-customize.md)
