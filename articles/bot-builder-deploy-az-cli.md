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
ms.date: 02/07/2019
ms.openlocfilehash: b4c3b982bf061b3a24c6d240b05dc40b0cf07816
ms.sourcegitcommit: 8183bcb34cecbc17b356eadc425e9d3212547e27
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 02/09/2019
ms.locfileid: "55971422"
---
# <a name="deploy-your-bot"></a>Déployer votre bot 

[!INCLUDE [pre-release-label](./includes/pre-release-label.md)]

Après avoir créé votre bot et l’avoir testé localement, vous pouvez le déployer sur Azure pour le rendre accessible depuis n’importe où. Le déploiement de votre bot sur Azure implique de payer les services que vous utilisez. L’article sur la [gestion de la facturation et des coûts](https://docs.microsoft.com/en-us/azure/billing/) vous aide à comprendre la facturation Azure, à superviser votre utilisation et vos coûts, ainsi qu’à gérer votre compte et vos abonnements.

Dans cet article, nous allons vous montrer comment déployer des bots C# et JavaScript sur Azure. Il est plus judicieux de lire cet article avant de suivre les étapes, afin de bien comprendre ce qu’implique le déploiement d’un bot.

## <a name="prerequisites"></a>Prérequis
- Installez la dernière version de l’outil [MSBot](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/MSBot).
- Un bot [CSharp](./dotnet/bot-builder-dotnet-sdk-quickstart.md) ou [JavaScript](./javascript/bot-builder-javascript-quickstart.md) que vous avez développé sur votre ordinateur local. 

## <a name="create-a-web-app-bot-in-azure"></a>Créer un Web App Bot dans Azure
Cette section est facultative si vous avez déjà créé un bot dans Azure que vous souhaitez utiliser.

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Cliquez sur le lien **Créer une ressource** dans le coin supérieur gauche du Portail Azure, puis sélectionnez **IA + Machine Learning > Web App bot (Web App Bot)**.
1. Un nouveau panneau s’ouvre avec des informations sur le Web App Bot. 
1. Dans le panneau **Service Bot**, indiquez les informations requises sur votre bot.
1. Cliquez sur **Créer** pour créer le service et déployer le bot dans le cloud. Ce processus peut prendre quelques minutes.

## <a name="download-the-source-code"></a>Télécharger le code source
1. Dans la section **Gestion du bot**, cliquez sur **Créer**.
1. Cliquez sur le lien **Download Bot source code** (Télécharger le code source du bot) dans le volet droit.
1. Suivez les invites pour télécharger le code, puis décompressez le dossier.

## <a name="decrypt-the-bot-file"></a>Déchiffrer le fichier .bot
Le code source que vous avez téléchargé à partir du portail Azure inclut un fichier .bot chiffré. Vous devez le déchiffrer pour copier les valeurs dans votre fichier .bot local.  

1. Dans le portail Azure, ouvrez la ressource Web App Bot pour votre bot.
1. Ouvrez les **Paramètres d’application** du bot.
1. Faites défiler la fenêtre **Paramètres d’application** jusqu’à **Paramètres d’application**.
1. Recherchez le **botFileSecret** et copiez sa valeur.

Utilisez `msbot cli` pour déchiffrer le fichier.
```
msbot secret --bot <name-of-bot-file> --secret "<bot-file-secret>" --clear
```

## <a name="update-the-bot-file"></a>Mettre à jour le fichier bot
Ouvrez le fichier .bot que vous avez déchiffré. Copiez les entrées listées sous la section `services` et ajoutez-les à votre fichier .bot local. Par exemple : 

```
{
   "type": "endpoint",
   "name": "production",
   "endpoint": "https://<something>.azurewebsites.net/api/messages",
   "appId": "<App Id>",
   "appPassword": "<App Password>",
   "id": "2
}
```

Enregistrez le fichier .
 
## <a name="setup-a-repository"></a>Configurer un dépôt
Créez un dépôt git à l’aide de votre fournisseur de contrôle de code source git favori. Validez votre code dans le dépôt.
 
## <a name="update-app-settings-in-azure"></a>Mettre à jour les paramètres d’application dans Azure
1. Dans le portail Azure, ouvrez la ressource **Web App Bot** pour votre bot.
1. Ouvrez les **Paramètres d’application** du bot.
1. Faites défiler la fenêtre **Paramètres d’application** jusqu’à **Paramètres d’application**.
1. Recherchez le **botFileSecret** et supprimez-le.
1. Mettez à jour le nom du fichier bot pour qu’il corresponde à celui du fichier archivé dans le dépôt.
1. Enregistrez les changements.


## <a name="deploy-using-azure-deployment-center"></a>Déployer à l’aide du Centre de déploiement Azure
Vous devez maintenant connecter votre dépôt git avec Azure App Services. Suivez les instructions fournies pour [configurer un déploiement continu](https://docs.microsoft.com/en-us/azure/app-service/deploy-continuous-deployment). Notez qu’il est recommandé d’utiliser `App Service Kudu build server` pour la génération.

## <a name="test-your-deployment"></a>Tester votre déploiement
Patientez quelques secondes après un déploiement réussi et redémarrez éventuellement votre Web App pour effacer tous les caches. Revenez à votre panneau Web App Bot et testez le bot à l’aide du Web Chat fourni dans le portail Azure.

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
