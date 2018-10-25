---
title: Télécharger et redéployer le code source d’un bot | Microsoft Docs
description: Découvrez comment télécharger et publier un service de robot.
keywords: télécharger le code source, redéployer, déployer, fichier zip, publier
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 09/26/2018
ms.openlocfilehash: afb1c4a0e766df7ac2d122b3c7ca4e7959871dbb
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2018
ms.locfileid: "49997276"
---
# <a name="download-and-redeploy-bot-code"></a>Télécharger et redéployer le code d’un bot
Azure Bot Service vous permet de télécharger l’intégralité du projet source de votre bot afin que vous puissiez travailler localement dans l’environnement de développement intégré de votre choix. Une fois que vous avez terminé la mise à jour du code, vous pouvez publier vos modifications dans le portail Azure. Nous allons vous montrer comment télécharger le code à l’aide du portail Azure et de la CLI `az`. Nous expliquerons également comment redéployer le code de votre bot mis à jour à l’aide de Visual Studio et de l’outil CLI `az`. Vous pouvez choisir la méthode qui vous convient le mieux.

## <a name="prerequisites"></a>Prérequis
- Installer [l’interface de ligne de commande Azure](https://docs.microsoft.com/en-us/cli/azure/?view=azure-cli-latest)
- Installer l’extension az botservice à l’aide de la commande `az extension add -n botservice`

### <a name="download-code-using-the-azure-portal"></a>Télécharger le code à l’aide du portail Azure
Pour télécharger le code à partir du [portail Azure](https://portal.azure.com), procédez comme suit :
1. Ouvrez le panneau du bot.
1. Dans la section **Bot management** (Gestion du bot), cliquez sur **Build** (Générer).
1. Sous **Download source code** (Télécharger le code source), cliquez sur **Download zip file** (Télécharger le fichier ZIP).
1. Attendez qu’Azure prépare votre URI de téléchargement, puis cliquez sur **Download zip file** (Télécharger le fichier ZIP) dans la notification.
1. Enregistrez et extrayez le fichier .zip dans un répertoire local.

Si vous avez un bot C#, mettez à jour le fichier `appsettings.json` pour y inclure les informations sur le fichier .bot, comme indiqué ci-dessous :

```
{
  "botFilePath": "yourbasicBot.bot",
  "botFileSecret": "ukxxxxxxxxxxxs="
}
```
`botFilePath` fait référence au nom de votre bot. Remplacez simplement « yourbasicBot.bot » par le nom de votre propre bot. Pour obtenir la clé `botFileSecret`, reportez-vous à l’article sur le [chiffrement des fichiers Bot](https://aka.ms/bot-file-encryption) qui explique comment générer une clé pour votre bot.


Si vous avez un bot node.js, ajoutez un fichier `.env` avec les entrées suivantes :
```
botFilePath=yourbasicBot.bot
botFileSecret=ukxxxxxxxxxxxxs=
```

Ensuite, modifiez vos sources en apportant des modifications aux fichiers source existants ou ajoutez-en des nouveaux à votre projet. Testez votre code à l’aide de l’émulateur. Lorsque vous êtes prêt à redéployer le code modifié sur le portail Azure, suivez les instructions ci-dessous.

### <a name="publish-code-using-visual-studio"></a>Publier un code à l’aide de Visual Studio
1. Dans Visual Studio, cliquez avec le bouton droit sur le nom de votre projet, puis cliquez sur **Publier...**. La fenêtre **Publier** s’affiche.

![Publication dans Azure](~/media/azure-bot-build/azure-csharp-publish.png)

2. Sélectionnez le profil de votre projet.
3. Copiez le mot de passe répertorié dans le fichier _publish.cmd_ de votre projet.
4. Cliquez sur **Publier**.
5. Lorsque vous y êtes invité, entrez le mot de passe que vous avez copié à l’étape 3.   

Une fois votre projet configuré, les modifications que vous y avez apportées sont publiées dans Azure. 

Nous allons ensuite aborder le téléchargement et le redéploiement de code à l’aide de la CLI `az`.

### <a name="download-code-using-azure-cli"></a>Télécharger du code à l’aide de l’interface de ligne de commande Azure

Tout d’abord, connectez-vous au portail Azure à l’aide de l’outil az cli.

```azcli
az login
```

Vous serez invité à saisir un code d’authentification temporaire unique. Pour vous connecter, utilisez un navigateur web et rendez-vous sur la page Microsoft de [connexion de l’appareil](https://microsoft.com/devicelogin), puis collez le code fourni par l’interface CLI pour continuer.

Pour télécharger du code à l’aide de l’interface de ligne de commande `az`, utilisez la commande suivante :
```azcli
az bot download --name "my-bot-name" --resource-group "my-resource-group"`
```
Une fois le code téléchargé, procédez comme suit :
- Pour un bot C#, mettez à jour le fichier appsettings.json pour y inclure les informations sur le fichier .bot, comme indiqué ci-dessous :

```
{
  "botFilePath": "yourbasicBot.bot",
  "botFileSecret": "ukxxxxxxxxxxxs="
}
```

- Pour un bot node.js, ajoutez un fichier .env avec les entrées suivantes :

```
botFilePath=yourbasicBot.bot
botFileSecret=ukxxxxxxxxxxxxs=
```

Ensuite, modifiez vos sources en apportant des modifications aux fichiers source existants ou ajoutez-en des nouveaux à votre projet. Testez votre code à l’aide de l’émulateur. Lorsque vous êtes prêt à redéployer le code modifié sur le portail Azure, suivez les instructions ci-dessous.

### <a name="login-to-azure-cli-by-running-the-following-command"></a>Connectez-vous à l’interface de ligne de commande Azure en exécutant la commande suivante.
Vous pouvez ignorer cette étape si vous déjà connecté.

```azcli
az login
```
Vous serez invité à saisir un code d’authentification temporaire unique. Pour vous connecter, utilisez un navigateur web et rendez-vous sur la page Microsoft de [connexion de l’appareil](https://microsoft.com/devicelogin), puis collez le code fourni par l’interface CLI pour continuer.

### <a name="publish-code-using-azure-cli"></a>Publier du code à l’aide de l’interface de ligne de commande Azure
Pour publier du code dans Azure à l’aide de l’interface de ligne de commande `az`, utilisez la commande suivante :
```azcli
az bot publish --name "my-bot-name" --resource-group "my-resource-group" --code-dir <path to directory> 
```

Vous pouvez utiliser l’option `code-dir` pour indiquer le répertoire à utiliser. S’il n’est pas indiqué, la commande `az bot publish` utilise le répertoire local pour la publication.

## <a name="next-steps"></a>Étapes suivantes
Maintenant que vous savez comment charger les modifications dans Azure, vous pouvez configurer le déploiement continu de votre bot.

> [!div class="nextstepaction"]
> [Configurer un déploiement continu](bot-service-build-continuous-deployment.md)
