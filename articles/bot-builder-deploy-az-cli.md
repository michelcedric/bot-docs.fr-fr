---
title: Déployer votre bot avec Azure CLI | Microsoft Docs
description: Déployez votre bot sur le cloud Azure.
keywords: déployer bot, azure déployer, publier bot, az déployer bot, visual studio déployer bot, msbot publier, clone msbot
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: get-started-article
ms.service: bot-service
ms.subservice: abs
ms.date: 12/14/2018
ms.openlocfilehash: 19960940a40fa291534bc1f88290bc6a7da109e0
ms.sourcegitcommit: 8c10aa7372754596a3aa7303a3a893dd4939f7e9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/20/2018
ms.locfileid: "53654334"
---
# <a name="deploy-your-bot-using-azure-cli"></a>Déployer votre bot avec Azure CLI

[!INCLUDE [pre-release-label](./includes/pre-release-label.md)]

Après avoir créé votre bot et l’avoir testé localement, vous pouvez le déployer sur Azure pour le rendre accessible depuis n’importe où. Le déploiement de votre bot sur Azure implique de payer les services que vous utilisez. L’article sur la [gestion de la facturation et des coûts](https://docs.microsoft.com/en-us/azure/billing/) vous aide à comprendre la facturation Azure, à superviser votre utilisation et vos coûts, ainsi qu’à gérer votre compte et vos abonnements.

Dans cet article, nous allons vous montrer comment déployer des bots C# et JavaScript sur Azure avec `az` et `msbot` cli. Il est plus judicieux de lire cet article avant de suivre les étapes, afin de bien comprendre ce qu’implique le déploiement d’un bot.


## <a name="prerequisites"></a>Prérequis
- Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.
- Installez la dernière version de l’outil [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest).
- Installez la dernière extension `botservice` pour l’outil `az`.
  - Tout d’abord, supprimez l’ancienne version à l’aide de la commande `az extension remove -n botservice`. Ensuite, utilisez la commande `az extension add -n botservice` pour installer la dernière version.
- Installez la dernière version de l’outil [MSBot](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/MSBot).
- Installez la dernière version publiée de l’[émulateur Bot Framework](https://aka.ms/Emulator-wiki-getting-started).
- Installez et configurez [ngrok](https://github.com/Microsoft/BotFramework-Emulator/wiki/Tunneling-%28ngrok%29).
- Familiarisez-vous avec ce qu’est un fichier [.bot](v4sdk/bot-file-basics.md).

## <a name="deploy-javascript-and-c-bots-using-az-cli"></a>Déployer des bots JavaScript et C# avec az cli

Vous avez déjà créé et testé un bot localement et vous souhaitez maintenant le déployer sur Azure. Ces étapes supposent que vous avez créé les ressources Azure nécessaires.

Ouvrez une invite de commandes pour vous connecter au portail Azure.

```cmd
az login
```

Une nouvelle fenêtre de navigateur s’ouvre ; connectez-vous.

### <a name="set-the-subscription"></a>Définir l’abonnement

Définissez l’abonnement par défaut à utiliser.

```cmd
az account set --subscription "<azure-subscription>"
```

Si vous n’êtes pas sûr de savoir quel abonnement utiliser pour déployer le bot, vous pouvez voir la liste des `subscriptions` de votre compte à l’aide de la commande `az account list`.

Accédez au dossier bot.

```cmd
cd <local-bot-folder>
```

### <a name="create-a-web-app-bot"></a>Créer un bot d’application web

Créez la ressource de bot dans laquelle vous allez publier votre bot.

Avant de continuer, lisez les instructions qui s’appliquent à votre cas en fonction du type de compte e-mail que vous utilisez pour vous connecter à Azure.

#### <a name="msa-email-account"></a>Compte e-mail MSA

Si vous utilisez un compte e-mail [MSA](https://en.wikipedia.org/wiki/Microsoft_account), vous devez créer un ID et un mot de passe d’application sur le portail d’inscription des applications à utiliser avec la commande `az bot create`.

1. Connectez-vous au [**portail d’inscription des applications**](https://apps.dev.microsoft.com/).
1. Cliquez sur **Ajouter une application** pour inscrire votre application, créez un **ID d’application**, puis **générez un nouveau mot de passe**. Si vous avez déjà une application et un mot de passe mais que vous avez oublié le mot de passe, vous devrez en générer un nouveau dans la section des secrets d’application.
1. Enregistrez l’ID d’application et le nouveau mot de passe que vous venez de générer afin de pouvoir les utiliser avec la commande `az bot create`.  

```cmd
az bot create --kind webapp --name <bot-resource-name> --location <geographic-location> --version v4 --lang <language> --verbose --resource-group <resource-group-name> --appid "<application-id>" --password "<application-password>" --verbose
```

| Option | Description |
|:---|:---|
| --name | Nom unique utilisé pour déployer le bot dans Azure. Il peut s’agir du même nom que celui de votre bot local. N’incluez PAS d’espaces ni de traits de soulignement dans le nom. |
| --location | Emplacement géographique utilisé pour créer les ressources de service du bot. Par exemple, `eastus`, `westus`, `westus2`, etc. |
| --lang | Langage à utiliser pour créer le bot : `Csharp` ou `Node` ; la valeur par défaut est `Csharp`. |
| --resource-group | Nom du groupe de ressources dans lequel créer le bot. Vous pouvez configurer le groupe par défaut en utilisant `az configure --defaults group=<name>`. |
| --appid | ID du compte Microsoft (ID MSA) à utiliser avec le bot. |
| --password | Mot de passe du compte Microsoft (MSA) pour le bot. |

#### <a name="business-or-school-account"></a>Compte professionnel ou scolaire

```cmd
az bot create --kind webapp --name <bot-resource-name> --location <geographic-location> --version v4 --lang <language> --verbose --resource-group <resource-group-name>
```

| Option | Description |
|:---|:---|
| --name | Nom unique utilisé pour déployer le bot dans Azure. Il peut s’agir du même nom que celui de votre bot local. N’incluez PAS d’espaces ni de traits de soulignement dans le nom. |
| --location | Emplacement géographique utilisé pour créer les ressources de service du bot. Par exemple, `eastus`, `westus`, `westus2`, etc. |
| --lang | Langage à utiliser pour créer le bot : `Csharp` ou `Node` ; la valeur par défaut est `Csharp`. |
| --resource-group | Nom du groupe de ressources dans lequel créer le bot. Vous pouvez configurer le groupe par défaut en utilisant `az configure --defaults group=<name>`. |

### <a name="download-the-bot-from-azure"></a>Télécharger le bot depuis Azure

Ensuite, téléchargez le bot que vous venez de créer. Cette commande crée un sous-répertoire sous save-path ; toutefois, le chemin spécifié doit déjà exister.

```cmd
az bot download --name <bot-resource-name> --resource-group <resource-group-name> --save-path "<path>"
```

| Option | Description |
|:---|:---|
| --name | Nom du bot dans Azure. |
| --resource-group | Nom du groupe de ressources dans lequel se trouve le bot. |
| --save-path | Répertoire existant sur lequel télécharger le code du bot. |

### <a name="decrypt-the-downloaded-bot-file"></a>Déchiffrer le fichier .bot téléchargé

Les informations sensibles du fichier .bot sont chiffrées.

Obtenez la clé de chiffrement.

1. Connectez-vous au [portail Azure](http://portal.azure.com/).
1. Ouvrez la ressource Web App Bot de votre bot.
1. Ouvrez les **Paramètres d’application** du bot.
1. Dans la fenêtre **Paramètres d’application**, faites défiler jusqu’à **Paramètres d’application**.
1. Recherchez le **botFileSecret** et copiez sa valeur.

Déchiffrez le fichier .bot.

```cmd
msbot secret --bot <name-of-bot-file> --secret "<bot-file-secret>" --clear
```

| Option | Description |
|:---|:---|
| --bot | Chemin relatif du fichier .bot téléchargé. |
| --secret | Clé de chiffrement. |

### <a name="use-the-downloaded-bot-file-in-your-project"></a>Utiliser le fichier .bot téléchargé dans votre projet

Copiez le fichier .bot déchiffré dans le répertoire contenant votre projet de bot local.

Mettez à jour votre bot pour utiliser ce nouveau fichier .bot.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Dans **appsettings.json**, mettez à jour la propriété **botFilePath** pour pointer vers le nouveau fichier .bot.

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Dans **.env**, mettez à jour la propriété **botFilePath** pour pointer vers le nouveau fichier .bot.

---

### <a name="update-the-bot-file"></a>Mettre à jour le fichier bot

Si votre bot utilise LUIS, QnA Maker ou les services Dispatch, vous devez les référencer dans votre fichier .bot. Dans le cas contraire, vous pouvez ignorer cette étape.

1. Ouvrez votre bot dans l’émulateur BotFramework à l’aide du nouveau fichier .bot. Le bot n’a pas besoin d’être exécuté localement.
1. Dans le volet **BOT EXPLORER**, développez la section **SERVICES**.
1. Pour ajouter des références aux applications LUIS, cliquez sur le signe plus (+) à droite de **SERVICES**.
   1. Sélectionnez **Add Language Understanding (LUIS)**.
   1. Si vous êtes invité à vous connecter avec votre compte Azure, faites-le.
   1. Une liste des applications LUIS auxquelles vous avez accès vous est présentée. Sélectionnez celles pour votre bot.
1. Pour ajouter des références à une base de connaissances QnA Maker, cliquez sur le signe plus (+) à droite de **SERVICES**.
   1. Sélectionnez **Add QnA Maker**.
   1. Si vous êtes invité à vous connecter avec votre compte Azure, faites-le.
   1. Une liste des bases de connaissances auxquelles vous avez accès vous est présentée. Sélectionnez celles pour votre bot.
1. Pour ajouter des références aux modèles Dispatch, cliquez sur le signe plus (+) à droite de **SERVICES**.
   1. Sélectionnez **Add Dispatch**.
   1. Si vous êtes invité à vous connecter avec votre compte Azure, faites-le.
   1. Une liste des modèles Dispatch auxquels vous avez accès vous est présentée. Sélectionnez celles pour votre bot.

### <a name="test-your-bot-locally"></a>Tester votre bot localement

À ce stade, votre bot devrait fonctionner de la même façon qu’avec l’ancien fichier .bot. Assurez-vous qu’il fonctionne comme prévu avec le nouveau fichier .bot.

### <a name="publish-your-bot-to-azure"></a>Publier votre bot sur Azure

<!-- TODO: re-encrypt your .bot file? -->

Publiez votre bot local sur Azure. Cette étape peut prendre du temps.

```cmd
az bot publish --name <bot-resource-name> --proj-file "<project-file-name>" --resource-group <resource-group-name> --code-dir <directory-path> --verbose --version v4
```

<!-- Question: What should --proj-file be for a Node project? -->

| Option | Description |
|:---|:---|
| --name | Nom de ressource du bot dans Azure. |
| --proj-file | Nom du fichier projet de démarrage (sans .csproj) qui doit être publié. Par exemple :  EnterpriseBot. |
| --resource-group | Nom du groupe de ressources. |
| --code-dir | Répertoire depuis lequel charger le code du bot. |

Une fois cette opération terminée avec le message « Déploiement réussi ! », votre bot est déployé sur Azure.

<!-- TODO: If we tell them to re-encrypt, this step is not necessary. -->

Effacez le paramètre de clé de chiffrement.

1. Connectez-vous au [portail Azure](http://portal.azure.com/).
1. Ouvrez la ressource Web App Bot de votre bot.
1. Ouvrez les **Paramètres d’application** du bot.
1. Dans la fenêtre **Paramètres d’application**, faites défiler jusqu’à **Paramètres d’application**.
1. Recherchez le **botFileSecret** et supprimez-le.

## <a name="additional-resources"></a>Ressources supplémentaires

Lorsque vous déployez un bot, ces ressources sont généralement créées dans le portail Azure :

| Ressources      | Description |
|----------------|-------------|
| Robot Web App | Bot Azure Bot Service déployé sur Azure App Service.|
| [App Service](https://docs.microsoft.com/en-us/azure/app-service/)| Permet de générer et héberger des applications web.|
| [Plan App Service](https://docs.microsoft.com/en-us/azure/app-service/azure-web-sites-web-hosting-plans-in-depth-overview)| Définit un ensemble de ressources de calcul nécessaires à l’exécution d’une application web.|
| [Application Insights](https://docs.microsoft.com/en-us/azure/application-insights/app-insights-overview)| Fournit des outils pour collecter et analyser des données de télémétrie.|
| [Compte de stockage](https://docs.microsoft.com/en-us/azure/storage/common/storage-introduction)| Fournit un stockage cloud hautement disponible, sécurisé, durable, scalable et redondant.|

Pour consulter la documentation sur les commandes `az bot`, reportez-vous à la rubrique de [référence](https://docs.microsoft.com/en-us/cli/azure/bot?view=azure-cli-latest).

Si vous ne connaissez pas le groupe de ressources Azure, consultez cette rubrique de [terminologie](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-overview#terminology).

## <a name="next-steps"></a>Étapes suivantes
> [!div class="nextstepaction"]
> [Configurer le déploiement continu](bot-service-build-continuous-deployment.md)
