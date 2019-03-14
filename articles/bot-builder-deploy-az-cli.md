---
title: Déployer votre bot | Microsoft Docs
description: Déployez votre bot sur le cloud Azure.
keywords: déployer bot, déployer bot azure, publier bot
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: get-started-article
ms.service: bot-service
ms.subservice: abs
ms.date: 02/13/2019
ms.openlocfilehash: 9cd2ed67110aa1611c41c33c31874f103e24b14d
ms.sourcegitcommit: b2245df2f0a18c5a66a836ab24a573fd70c7d272
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 03/07/2019
ms.locfileid: "57568196"
---
# <a name="deploy-your-bot"></a>Déployer votre bot

[!INCLUDE [pre-release-label](./includes/pre-release-label.md)]

Après avoir créé votre bot et l’avoir testé localement, vous pouvez le déployer sur Azure pour le rendre accessible depuis n’importe où. Le déploiement de votre bot sur Azure implique de payer les services que vous utilisez. L’article sur la [gestion de la facturation et des coûts](https://docs.microsoft.com/en-us/azure/billing/) vous aide à comprendre la facturation Azure, à superviser votre utilisation et vos coûts, ainsi qu’à gérer votre compte et vos abonnements.

Dans cet article, nous allons vous montrer comment déployer des bots C# et JavaScript sur Azure. Il est plus judicieux de lire cet article avant de suivre les étapes, afin de bien comprendre ce qu’implique le déploiement d’un bot.

## <a name="prerequisites"></a>Prérequis

- Installez la dernière version de l’outil [msbot](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/MSBot).
- Un bot [CSharp](./dotnet/bot-builder-dotnet-sdk-quickstart.md) ou [JavaScript](./javascript/bot-builder-javascript-quickstart.md) que vous avez développé sur votre ordinateur local.

## <a name="1-prepare-for-deployment"></a>1. Préparation du déploiement
Le processus de déploiement nécessite un bot Web App cible dans Azure pour que votre bot local puisse y être déployé. Le bot Web App cible et les ressources qui sont provisionnées avec lui dans Azure sont utilisés par votre bot local pour le déploiement. C’est nécessaire, car votre bot local n’a pas toutes les ressources Azure nécessaires provisionnées. Quand vous créez un bot Web App cible, les ressources suivantes sont provisionnées pour vous :
-   Bot Web App : vous utilisez ce bot pour y déployer votre bot local.
-   Plan App Service : fournit les ressources nécessaires à l’exécution d’une application App Service.
-   App Service : service pour l’hébergement d’applications web
-   Compte de stockage : contient tous vos objets de données de Stockage Azure : objets blob, fichiers, files d’attente, tables et disques.

Lors de la création du bot Web App cible, un ID et un mot de passe d’application sont également générés pour votre bot. Dans Azure, l’ID et le mot de passe d’application prennent en charge [l’authentification et les autorisations du service](https://docs.microsoft.com/azure/app-service/overview-authentication-authorization). Vous allez récupérer certaines de ces informations pour les utiliser dans le code de votre bot local. 

> [!IMPORTANT]
> Le langage du modèle de bot pour le service doit correspondre au langage dans lequel votre bot est écrit.

La création d’un bot Web App est facultative si vous avez déjà créé un bot dans Azure que vous souhaitez utiliser.

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Cliquez sur le lien **Créer une ressource** dans le coin supérieur gauche du Portail Azure, puis sélectionnez **IA + Machine Learning > Web App bot (Web App Bot)**.
1. Un nouveau panneau s’ouvre avec des informations sur le Web App Bot. 
1. Dans le panneau **Service Bot**, indiquez les informations requises sur votre bot.
1. Cliquez sur **Créer** pour créer le service et déployer le bot dans le cloud. Ce processus peut prendre quelques minutes.

### <a name="download-the-source-code"></a>Télécharger le code source
Après avoir créé le bot Web App cible, vous devez télécharger le code du bot à partir du portail Azure sur votre machine locale. La raison du téléchargement du code est que vous devez obtenir les références des services qui se trouvent dans le [fichier .bot](./v4sdk/bot-file-basics.md). Ces références des services concernent : Bot Web App, Plan App Service, App Service et Compte de stockage. 

1. Dans la section **Gestion du bot**, cliquez sur **Créer**.
1. Cliquez sur le lien **Download Bot source code** (Télécharger le code source du bot) dans le volet droit.
1. Suivez les invites pour télécharger le code, puis décompressez le dossier.
    1. [!INCLUDE [download keys snippet](~/includes/snippet-abs-key-download.md)]

### <a name="decrypt-the-bot-file"></a>Déchiffrer le fichier .bot

Le code source que vous avez téléchargé à partir du portail Azure inclut un fichier .bot chiffré. Vous devez le déchiffrer pour copier les valeurs dans votre fichier .bot local. Cette étape est nécessaire pour pouvoir copier les références réelles des services et non pas les références chiffrées.  

1. Dans le portail Azure, ouvrez la ressource Web App Bot pour votre bot.
1. Ouvrez les **Paramètres d’application** du bot.
1. Faites défiler la fenêtre **Paramètres d’application** jusqu’à **Paramètres d’application**.
1. Recherchez le **botFileSecret** et copiez sa valeur.
1. Utilisez `msbot cli` pour déchiffrer le fichier.

    ```cmd
    msbot secret --bot <name-of-bot-file> --secret "<bot-file-secret>" --clear
    ```

### <a name="update-your-local-bot-file"></a>Mettre à jour votre fichier .bot local

Ouvrez le fichier .bot que vous avez déchiffré. Copiez **toutes** les entrées listées sous la section `services` et ajoutez-les à votre fichier .bot local. Résolvez les éventuelles entrées de service en double ou les ID de service en double. Conservez les autres références des services dont dépend votre robot. Par exemple : 

```json
"services": [
    {
        "type": "abs",
        "tenantId": "<tenant-id>",
        "subscriptionId": "<subscription-id>",
        "resourceGroup": "<resource-group-name>",
        "serviceName": "<bot-service-name>",
        "name": "<friendly-service-name>",
        "id": "1",
        "appId": "<app-id>"
    },
    {
        "type": "blob",
        "connectionString": "<connection-string>",
        "tenantId": "<tenant-id>",
        "subscriptionId": "<subscription-id>",
        "resourceGroup": "<resource-group-name>",
        "serviceName": "<blob-service-name>",
        "id": "2"
    },
    {
        "type": "endpoint",
        "appId": "",
        "appPassword": "",
        "endpoint": "<local-endpoint-url>",
        "name": "development",
        "id": "3"
    },
    {
        "type": "endpoint",
        "appId": "<app-id>",
        "appPassword": "<app-password>",
        "endpoint": "<hosted-endpoint-url>",
        "name": "production",
        "id": "4"
    },
    {
        "type": "appInsights",
        "instrumentationKey": "<instrumentation-key>",
        "applicationId": "<appinsights-app-id>",
        "apiKeys": {},
        "tenantId": "<tenant-id>",
        "subscriptionId": "<subscription-id>",
        "resourceGroup": "<resource-group>",
        "serviceName": "<appinsights-service-name>",
        "id": "5"
    }
],
```

Enregistrez le fichier .

Vous pouvez utiliser l’outil msbot pour générer un nouveau secret et chiffrer le fichier .bot avant de publier. Si vous rechiffrez votre fichier .bot, mettez à jour le **botFileSecret** du bot dans le portail Azure pour qu’il contienne le nouveau secret.

```cmd
msbot secret --bot <name-of-bot-file> --new
```

### <a name="setup-a-repository"></a>Configurer un dépôt

Pour prendre en charge le déploiement continu, créez un dépôt Git en utilisant votre fournisseur de contrôle de code source Git favori. Validez votre code dans le dépôt.

Vérifiez que la racine de votre dépôt contient les fichiers corrects, comme décrit sous [Préparer votre dépôt](https://docs.microsoft.com/azure/app-service/deploy-continuous-deployment#prepare-your-repository).

### <a name="update-app-settings-in-azure"></a>Mettre à jour les paramètres d’application dans Azure
Le bot local n’utilise pas de fichier .bot chiffré, mais le portail Azure est configuré pour en utiliser un. Vous pouvez résoudre ce problème en supprimant le **botFileSecret** qui est stocké dans les paramètres du bot Azure.
1. Dans le portail Azure, ouvrez la ressource **Web App Bot** pour votre bot.
1. Ouvrez les **Paramètres d’application** du bot.
1. Faites défiler la fenêtre **Paramètres d’application** jusqu’à **Paramètres d’application**.
1. Recherchez le **botFileSecret** et supprimez-le. Si vous rechiffrez votre fichier .bot, vérifiez que le **botFileSecret** contient le nouveau secret et **ne supprimez pas** le paramètre.
1. Mettez à jour le nom du fichier bot pour qu’il corresponde à celui du fichier archivé dans le dépôt.
1. Enregistrez les changements.

## <a name="2-deploy-using-azure-deployment-center"></a>2. Déployer à l’aide du Centre de déploiement Azure

Maintenant, vous devez charger le code de votre bot dans Azure. Suivez les instructions fournies dans la rubrique [Déploiement continu vers Azure App Service](https://docs.microsoft.com/azure/app-service/deploy-continuous-deployment).

Notez qu’il est recommandé d’utiliser `App Service Kudu build server` pour la génération.

Une fois que vous avez configuré le déploiement continu, les changements que vous commitez dans votre dépôt sont publiées. Cependant, si vous ajoutez des services à votre bot, vous devez ajouter des entrées pour ceux-ci à votre fichier .bot.

## <a name="3-test-your-deployment"></a>3. Tester votre déploiement

Attendez quelques secondes après un déploiement réussi et redémarrez éventuellement votre application Web App pour effacer tous les caches. Revenez à votre panneau Web App Bot et testez le bot à l’aide du Web Chat fourni dans le portail Azure.

## <a name="additional-resources"></a>Ressources supplémentaires

- [How to investigate common issues with continuous deployment (Examen des problèmes courants liés au déploiement continu)](https://github.com/projectkudu/kudu/wiki/Investigating-continuous-deployment)

<!--

## Prerequisites

[!INCLUDE [prerequisite snippet](~/includes/deploy/snippet-prerequisite.md)]


## Deploy JavaScript and C# bots using az cli

You've already created and tested a bot locally, and now you want to deploy it to Azure. These steps assume that you have created the required Azure resources.

[!INCLUDE [az login snippet](~/includes/deploy/snippet-az-login.md)]

### Create a Web App Bot

If you don't already have a resource group to which to publish your bot, create one:

[!INCLUDE [az create group snippet](~/includes/deploy/snippet-az-create-group.md)]

[!INCLUDE [az create web app snippet](~/includes/deploy/snippet-create-web-app.md)]

Before proceeding, read the instructions that apply to you based on the type of email account you use to log in to Azure.

#### MSA email account

If you are using an [MSA](https://en.wikipedia.org/wiki/Microsoft_account) email account, you will need to create the app ID and app password on the Application Registration Portal to use with `az bot create` command.

[!INCLUDE [create bot msa snippet](~/includes/deploy/snippet-create-bot-msa.md)]

#### Business or school account

[!INCLUDE [create bot snippet](~/includes/deploy/snippet-create-bot.md)]

### Download the bot from Azure

Next, download the bot you just created. 
[!INCLUDE [download bot snippet](~/includes/deploy/snippet-download-bot.md)]

[!INCLUDE [download keys snippet](~/includes/snippet-abs-key-download.md)]

### Decrypt the downloaded .bot file and use in your project

The sensitive information in the .bot file is encrypted.

[!INCLUDE [decrypt bot snippet](~/includes/deploy/snippet-decrypt-bot.md)]

### Update the .bot file

If your bot uses LUIS, QnA Maker, or Dispatch services, you will need to add references to them to your .bot file. Otherwise, you can skip this step.

1. Open your bot in the BotFramework Emulator, using the new .bot file. The bot does not need to be running locally.
1. In the **BOT EXPLORER** panel, expand the **SERVICES** section.
1. To add references to LUIS apps, click the plus-sign (+) to the right of **SERVICES**.
   1. Select **Add Language Understanding (LUIS)**.
   1. If it prompts you to log into your Azure account, do so.
   1. It presents a list of LUIS applications you have access to. Select the ones for your bot.
1. To add references to a QnA Maker knowledge base, click the plus-sign (+) to the right of **SERVICES**.
   1. Select **Add QnA Maker**.
   1. If it prompts you to log into your Azure account, do so.
   1. It presents a list of knowledge bases you have access to. Select the ones for your bot.
1. To add references to Dispatch models, click the plus-sign (+) to the right of **SERVICES**.
   1. Select **Add Dispatch**.
   1. If it prompts you to log into your Azure account, do so.
   1. It presents a list of Dispatch models you have access to. Select the ones for your bot.

### Test your bot locally

At this point, your bot should work the same way it did with the old .bot file. Make sure that it works as expected with the new .bot file.

### Publish your bot to Azure

[!INCLUDE [publish snippet](~/includes/deploy/snippet-publish.md)]


[!INCLUDE [clear encryption snippet](~/includes/deploy/snippet-clear-encryption.md)]

## Additional resources

[!INCLUDE [additional resources snippet](~/includes/deploy/snippet-additional-resources.md)]

## Next steps
> [!div class="nextstepaction"]
> [Set up continous deployment](bot-service-build-continuous-deployment.md)

-->
