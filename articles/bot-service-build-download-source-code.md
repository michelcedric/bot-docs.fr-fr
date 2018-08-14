---
title: Télécharger et redéployer le code source d’un service de robot | Microsoft Docs
description: Découvrez comment télécharger et publier un service de robot.
keywords: télécharger le code source, redéployer, déployer, fichier zip, publier
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 03/08/2018
ms.openlocfilehash: 6d76388712ffeff94c56ba89b4bf4f4831caf45c
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39299925"
---
# <a name="download-and-redeploy-bot-service-source-code"></a>Télécharger et redéployer le code source d’un service de robot

Le service de robot vous permet de télécharger la totalité du projet source de votre bot. Cette opération vous offre la possibilité de travailler sur votre bot localement à l’aide de l’environnement de développement intégré (IDE) de votre choix. Une fois que vous avez terminé vos modifications, vous pouvez republier ces dernières sur Azure. 

Cet article vous explique comment télécharger le code source de votre bot, puis comment republier les modifications sur Azure. 

## <a name="download-bot-source-code"></a>Télécharger le code source du bot

Pour développer votre bot localement, procédez comme suit :

1. Dans le Portail Azure, ouvrez le panneau du bot.
2. Sous la section **BOT MANAGEMENT** (GESTION DU BOT), cliquez sur **Générer**.
3. Cliquez sur **Télécharger le fichier zip**. 

   ![Téléchargement du code source](~/media/azure-bot-build/download-zip-file.png)

4. Extrayez le fichier zip dans un répertoire local.
5. Accédez au dossier extrait et ouvrez les fichiers sources dans votre IDE favori.
6. Modifiez les fichiers sources. Vous pouvez modifier les fichiers sources existants ou ajouter de nouveaux fichiers à votre projet.

Lorsque vous avez terminé, vous pouvez republier les sources sur Azure.

## <a name="publish-node-bot-source-code-to-azure"></a>Publier un code source de bot Node sur Azure

Pour installer ces packages, accédez au répertoire de votre projet à partir d’une invite de commandes, puis exécutez les commandes NPM suivantes.

**Remarque :** ces packages ne doivent être ajoutés qu’une seule fois.

```console
npm install --save fs
npm install --save path
npm install --save request
npm install --save zip-folder
```

Vous voici prêt à publier votre projet sur Microsoft Azure. Pour publier votre projet sur Microsoft Azure, exécutez la commande NPM ci-après au niveau de l’invite de commandes :

```console
npm run azure-publish
```

> [!NOTE]
> Si vous rencontrez une erreur après avoir entré cette commande NPM, vous devrez peut-être ajouter la chaîne `"scripts": {"azure-publish": "node publish.js"}` à votre fichier `package.json`, puis réexécuter la commande.

## <a name="publish-c-bot-source-code-to-azure"></a>Publier un code source de bot C# sur Azure

La publication de code C# sur Azure à l’aide de Visual Studio est un processus en deux étapes : vous devez commencer par configurer les paramètres de publication. Vous pouvez ensuite publier vos modifications.

Pour configurer la publication à partir de Visual Studio, procédez comme suit :

1. Dans Visual Studio, cliquez sur **Explorateur de solutions**.
2. Cliquez avec le bouton droit sur le nom de votre projet, puis cliquez sur **Publier...**. La fenêtre **Publier** s’affiche.
3. Cliquez sur **Créer un profil**, sur **Importer le profil**, puis sur **OK**.
4. Accédez au dossier de votre projet, accédez au dossier **PostDeployScripts**, puis sélectionnez le fichier se terminant par **.PublishSettings**. Cliquez sur **Ouvrir**.

Votre projet est désormais configuré pour la publication des modifications sur Azure.

Une fois votre projet configuré, vous pouvez republier le code source de votre bot sur Azure en procédant comme suit :

1. Dans Visual Studio, cliquez sur **Explorateur de solutions**.
2. Cliquez avec le bouton droit sur le nom de votre projet, puis cliquez sur **Publier...**.
3. Cliquez sur le bouton **Publier** pour publier vos modifications sur Azure.

## <a name="next-steps"></a>Étapes suivantes
Maintenant que vous avez appris à générer votre bot localement, vous pouvez configurer le déploiement continu de votre bot.

> [!div class="nextstepaction"]
> [Configurer un déploiement continu](bot-service-build-continuous-deployment.md)
