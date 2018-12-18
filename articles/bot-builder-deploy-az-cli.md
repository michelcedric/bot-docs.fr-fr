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
ms.date: 12/11/2018
ms.openlocfilehash: a7cb9cb1e3df14f2f46bc5a4c3a633f5212b5dfd
ms.sourcegitcommit: 0b421ff71617f03faf55ea175fb91d1f9e348523
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/12/2018
ms.locfileid: "53286615"
---
# <a name="deploy-your-bot-using-azure-cli"></a>Déployer votre bot avec Azure CLI

[!INCLUDE [pre-release-label](./includes/pre-release-label.md)]

Après avoir créé votre bot et l’avoir testé localement, vous pouvez le déployer sur Azure pour le rendre accessible depuis n’importe où. Le déploiement de votre bot sur Azure implique de payer les services que vous utilisez. L’article sur la [gestion de la facturation et des coûts](https://docs.microsoft.com/en-us/azure/billing/) vous aide à comprendre la facturation Azure, à superviser votre utilisation et vos coûts, ainsi qu’à gérer votre compte et vos abonnements.

Dans cet article, nous allons vous montrer comment déployer des bots C# et JavaScript sur Azure à l’aide de l’outil `msbot`. Il est plus judicieux de lire cet article avant de suivre les étapes, afin de bien comprendre ce qu’implique le déploiement d’un bot.


## <a name="prerequisites"></a>Prérequis
- Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.
- Installez le [kit SDK .NET Core](https://dotnet.microsoft.com/download) version 2.2 ou supérieure. 
- Installez la dernière version de l’outil [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest).
- Installez la dernière extension `botservice` pour l’outil `az`. 
  - Tout d’abord, supprimez l’ancienne version à l’aide de la commande `az extension remove -n botservice`. Ensuite, utilisez la commande `az extension add -n botservice` pour installer la dernière version.
- Installez la dernière version de l’outil [MSBot](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/MSBot).
  - Vous avez besoin de [LUIS CLI](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/LUIS#installation) si l’opération de clonage inclut des ressources LUIS ou Dispatch.
  - Vous avez besoin de [QnA Maker CLI](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/QnAMaker#as-a-cli) si l’opération de clonage inclut des ressources QnA Maker.
- Installez [Bot Framework Emulator](https://aka.ms/Emulator-wiki-getting-started).
- Installez et configurez [ngrok](https://github.com/Microsoft/BotFramework-Emulator/wiki/Tunneling-%28ngrok%29).
- Familiarisez-vous avec ce qu’est un fichier [.bot](v4sdk/bot-file-basics.md).

## <a name="deploy-javascript-and-c-bots-using-az-cli"></a>Déployer des bots JavaScript et C# avec az cli
Vous avez déjà créé un bot et vous souhaitez maintenant le déployer sur Azure. Ces étapes impliquent d’avoir créé les ressources Azure nécessaires et d’avoir mis à jour les références de service dans le fichier .bot à l’aide de la commande `msbot connect` ou de Bot Framework Emulator. Si le fichier .bot n’est pas à jour, le processus de déploiement peut se terminer sans erreurs ni avertissements, mais le bot déployé ne fonctionnera pas.

Ouvrez une invite de commandes pour vous connecter au portail Azure.

```cmd
az login
```
Une nouvelle fenêtre de navigateur s’ouvre ; connectez-vous. 

### <a name="set-the-subscription"></a>Définir l’abonnement 
Définissez l’abonnement à l’aide de la commande suivante :

```cmd
az account set --subscription "<azure-subscription>"
``` 

Si vous n’êtes pas sûr de savoir quel abonnement utiliser pour déployer le bot, vous pouvez voir la liste des `subscriptions` de votre compte à l’aide de la commande `az account list`.

Accédez au dossier bot. 
```cmd 
cd <local-bot-folder>
```

### <a name="azure-subscription-account"></a>Compte d’abonnement Azure
Avant de continuer, lisez les instructions qui s’appliquent à votre cas en fonction du type de compte e-mail que vous utilisez pour vous connecter à Azure.

**Compte e-mail MSA**

Si vous utilisez un compte e-mail [MSA](https://en.wikipedia.org/wiki/Microsoft_account), vous devez créer les valeurs appId et appSecret à utiliser avec la commande `msbot clone services`. 

- Connectez-vous au [portail d’inscription des applications](https://apps.dev.microsoft.com/). Cliquez sur **Ajouter une application** pour inscrire votre application, créez un **ID d’application**, puis **générez un nouveau mot de passe**. 
- Enregistrez l’ID d’application et le nouveau mot de passe que vous venez de générer, pour pouvoir les utiliser avec la commande `msbot clone services`. 
- Pour effectuer le déploiement, utilisez la commande applicable à votre bot.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

`msbot clone services --folder deploymentScripts/msbotClone --location "<geographic-location>" --proj-file "<your.csproj>" --name "<bot-name>" --appid "xxxxxxxx" --password "xxxxxxx" --verbose`

[!INCLUDE [deployment note](./includes/deployment-note-cli.md)]

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

`msbot clone services --folder deploymentScripts/msbotClone --location "<geographic-location>"   --code-dir . --name "<bot-name>" --appid "xxxxxxxx" --password "xxxxxxx" --verbose`


[!INCLUDE [deployment note](./includes/deployment-note-cli.md)]

---

**Compte professionnel ou scolaire**

Si vous utilisez un compte e-mail fourni par votre entreprise ou votre établissement scolaire pour vous connecter à Azure, vous n’avez pas besoin de créer l’ID d’application et le mot de passe. Pour effectuer le déploiement, utilisez la commande applicable à votre bot.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

`msbot clone services --folder deploymentScripts/msbotClone --location "<geographic-location>" --verbose --proj-file "<your-project-file>" --name "<bot-name>"`

[!INCLUDE [deployment note](./includes/deployment-note-cli.md)]

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

`msbot clone services --folder deploymentScripts/msbotClone --location "<geographic-location>" --verbose --code-dir . --name "<bot-name>"`


[!INCLUDE [deployment note](./includes/deployment-note-cli.md)]

---

#### <a name="save-the-secret-used-to-decrypt-bot-file"></a>Enregistrer le secret utilisé pour déchiffrer le fichier .bot
Il est important de noter que le processus de déploiement crée un _fichier .bot et le chiffre à l’aide d’un secret_. Bien que le bot soit déployé, le message suivant apparaît dans la ligne de commande vous invitant à enregistrer le secret du fichier .bot. 

`The secret used to decrypt myAzBot.bot is:`
`hT6U1SIPQeXlebNgmhHYxcdseXWBZlmIc815PpK0WWA=`

`NOTE: This secret is not recoverable and you should save it in a safe place according to best security practices.
      Copy this secret and use it to open the <file.bot> the first time.`
      
Enregistrez le secret du fichier .bot pour l’utiliser ultérieurement. Le nouveau fichier .bot chiffré est utilisé dans le portail Azure avec botFileSecret. Si vous avez besoin de changer le nom du fichier bot ou le secret par la suite, accédez à la section **Paramètres App Service -> Paramètres de l’application** dans le portail. Notez que dans le fichier appsettings.json ou .env, le nom du fichier bot est mis à jour avec le dernier fichier bot qui a été créé.  

### <a name="test-your-bot"></a>Tester votre bot
Dans l’émulateur, utilisez le point de terminaison de production pour tester votre application. Si vous souhaitez la tester localement, vérifiez que votre bot est en cours d’exécution sur votre ordinateur local. 

### <a name="to-update-your-bot-code-in-azure"></a>Pour mettre à jour le code de votre bot dans Azure
N’utilisez PAS la commande `msbot clone services` pour mettre à jour le code de votre bot dans Azure. Vous devez utiliser la commande `az bot publish` comme indiqué ci-dessous :

```cmd
az bot publish --name "<your-azure-bot-name>" --proj-file "<your-proj-file>" --resource-group "<azure-resource-group>" --code-dir "<folder>" --verbose --version v4
```

| Arguments        | Description |
|----------------  |-------------|
| `name`      | Nom que vous avez utilisé lors du premier déploiement du bot sur Azure.|
| `proj-file` | Pour un bot C#, il s’agit du fichier .csproj. Pour un bot JS/TS, il s’agit du nom de fichier du projet de démarrage (par exemple, index.js ou index.ts) de votre bot local.|
| `resource-group` | Groupe de ressources Azure que la commande `msbot clone services` a utilisé.|
| `code-dir`  | Pointe vers le dossier du bot local.|



## <a name="additional-resources"></a>Ressources supplémentaires

Lorsque vous déployez un bot, ces ressources sont généralement créées dans le portail Azure :

| Ressources      | Description |
|----------------|-------------|
| Bot Web App | Bot Azure Bot Service déployé sur Azure App Service.|
| [App Service](https://docs.microsoft.com/en-us/azure/app-service/)| Permet de générer et héberger des applications web.|
| [Plan App Service](https://docs.microsoft.com/en-us/azure/app-service/azure-web-sites-web-hosting-plans-in-depth-overview)| Définit un ensemble de ressources de calcul nécessaires à l’exécution d’une application web.|
| [Application Insights](https://docs.microsoft.com/en-us/azure/application-insights/app-insights-overview)| Fournit des outils pour collecter et analyser des données de télémétrie.|
| [Compte de stockage](https://docs.microsoft.com/en-us/azure/storage/common/storage-introduction)| Fournit un stockage cloud hautement disponible, sécurisé, durable, scalable et redondant.|

Pour consulter la documentation sur les commandes `az bot`, reportez-vous à la rubrique de [référence](https://docs.microsoft.com/en-us/cli/azure/bot?view=azure-cli-latest).

Si vous ne connaissez pas le groupe de ressources Azure, consultez cette rubrique de [terminologie](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-overview#terminology).

## <a name="next-steps"></a>Étapes suivantes
> [!div class="nextstepaction"]
> [Configurer le déploiement continu](bot-service-build-continuous-deployment.md)
