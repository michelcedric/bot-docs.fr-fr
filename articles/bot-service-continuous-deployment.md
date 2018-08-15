---
title: Publier un Service Bot à partir d’un contrôle de code source ou de Visual Studio | Microsoft Docs
description: Découvrez comment publier un robot Service Bot une fois à partir de Visual Studio ou en continu à partir d’un contrôle de code source.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: 980ee1c6e4e54ff3f74ccfa618b0aeff7b507815
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39299789"
---
# <a name="publish-a-bot-to-bot-service"></a>Publier un robot sur Service Bot

Après avoir mis à jour votre code source de robot en C#, vous pouvez le publier dans un robot en cours d’exécution dans Service Bot à l’aide de Visual Studio. Vous pouvez également publier le code source du robot en C# ou Node.js écrit dans tout environnement de développement intégré (IDE) automatiquement chaque fois que vous vérifiez un fichier source dans un service de contrôle de code source.


## <a name="publish-a-bot-on-app-service-plan-from-the-online-code-editor"></a>Publier un robot sur un plan App Service à partir de l’éditeur de code en ligne

Si vous n’avez pas configuré le déploiement continu, vous pouvez modifier vos fichiers sources dans l’éditeur de code en ligne. Pour déployer votre source modifiée, procédez comme suit.

4. Cliquez sur l’icône Ouvrir la console.  
    ![Icône de console](~/media/azure-bot-service-console-icon.png)
2. Dans la fenêtre de la console, tapez **build.cmd**, puis appuyez sur la touche Entrée.


## <a name="publish-c-bot-on-app-service-plan-from-visual-studio"></a>Publier un robot en C# sur un plan App Service à partir de Visual Studio 

Pour configurer la publication à partir de Visual Studio à l’aide du fichier `.PublishSettings`, procédez comme suit :

1. Dans le portail Azure, cliquez sur votre Service Bot, sur l’onglet **BUILD**, puis sur **Télécharger le fichier zip**.
3. Extrayez le contenu du fichier zip téléchargé dans un dossier local.
4. Dans l’Explorateur, recherchez le fichier Solution Visual Studio (.sln) pour votre robot, puis double-cliquez dessus.
4. Dans Visual Studio, cliquez sur **Afficher**, puis sur **Explorateur de solutions**.
5. Dans le volet Explorateur de solutions, cliquez sur votre projet, puis cliquez sur **Publier...**. La fenêtre Publier s’ouvre. 
6. Dans la fenêtre Publier, cliquez sur **Créer un profil**, sur **Importer le profil**, puis sur **OK**.
7. Accédez au dossier de votre projet, accédez au dossier**PostDeployScripts**, sélectionnez le fichier se terminant par `.PublishSettings`, puis cliquez sur **Ouvrir**.

Vous avez maintenant configuré la publication pour ce projet. Pour publier votre code source local dans Service Bot, cliquez avec le bouton droit sur votre projet, cliquez sur **Publier...** , puis cliquez sur le bouton **Publier**. 

## <a name="set-up-continuous-deployment"></a>Configurer un déploiement continu

Par défaut, Service Bot vous permet de développer votre robot directement dans le navigateur à l’aide de l’éditeur Azure, sans avoir besoin d’un éditeur ou d’un contrôle de code source locaux. Toutefois, l’éditeur Azure ne vous permet pas gérer des fichiers (par exemple, ajouter, renommer ou supprimer des fichiers) au sein de votre application. Si vous souhaitez avoir la possibilité de gérer des fichiers au sein de votre application, vous pouvez configurer le déploiement continu et utiliser l’environnement de développement intégré (IDE) et le système de contrôle de code source de votre choix (par exemple, Visual Studio Team, GitHub, Bitbucket). Avec le déploiement continu configuré, toutes les modifications de code que vous validez dans le contrôle de code source sont automatiquement déployées sur Azure. Après avoir configuré le déploiement continu, vous pouvez [déboguer votre robot localement](bot-service-debug-bot.md).

> [!NOTE]
> Si vous activez le déploiement continu pour votre robot, vous devez archiver les changements du code dans votre service de contrôle de code source. Si vous souhaitez modifier à nouveau votre code dans l’éditeur Azure, vous devez [désactiver le déploiement continu](#disable-continuous-deployment).

Vous pouvez activer le déploiement continu pour votre application de robot en procédant comme suit.

## <a name="set-up-continuous-deployment-for-a-bot-on-an-app-service-plan"></a>Configurer le déploiement continu pour un robot sur un plan App Service

Cette section décrit comment activer le déploiement continu pour un robot que vous avez créé à l’aide de Service Bot, ayant un plan d’hébergement App Service.

1. Dans le portail Azure, recherchez votre robot Azure, cliquez sur l’onglet **BUILD**, puis recherchez la section **Déploiement continu à partir du contrôle de code source**.
2. Pour Visual Studio Online ou GitHub, fournissez un jeton d’accès émis pour vous sur ces sites web. Votre source sera extraite d’Azure dans votre dépôt source.
3. Pour les autres systèmes de contrôle de code source, sélectionnez **Autres**, puis suivez les étapes qui s’affichent. 
3. Cliquez sur **Enable**.  

### <a name="create-an-empty-repository-and-download-bot-source-code"></a>Créer un dépôt vide et télécharger le code source du robot

Suivez ces étapes si vous souhaitez utiliser un service de contrôle de code source *autre que* Visual Studio Online ou Github. Visual Studio Online et Github extrayant le code source pour votre robot à partir d’Azure, les utilisateurs de ces deux services peuvent ignorer ces étapes.

3. Pour un robot dans un plan App Service, recherchez la page de votre robot sur Azure, cliquez sur l’onglet **BUILD**, recherchez la section **Télécharger le code source**, puis cliquez sur **Télécharger le fichier zip**.
1. Créez un dépôt vide au sein d’un des systèmes de contrôle de code source pris en charge par Azure.

    ![Système de contrôle de code source](~/media/continuous-integration-sourcecontrolsystem.png)

3. Extrayez le contenu du fichier zip téléchargé dans le dossier local où vous prévoyez de synchroniser votre source de déploiement.
4. Cliquez sur **Configurer**, puis suivez les étapes qui s’affichent. 

## <a name="set-up-continuous-deployment-for-a-bot-on-a-consumption-plan"></a>Configurer le déploiement continu pour un robot sur un plan de consommation 

Choisissez la source de déploiement pour votre robot, puis connectez votre dépôt. 

1. Dans le portail Azure, dans votre robot Azure, cliquez sur l’onglet **PARAMÈTRES**, puis cliquez sur **Configurer** pour développer la section **Déploiement continu**.  
2. Suivez les étapes, puis cliquez sur la case à cocher pour confirmer que vous êtes prêt. 
3. Cliquez sur **Configurer**, sélectionnez la source de déploiement correspondant au système de contrôle de code source dans lequel vous avez créé précédemment le dépôt vide, puis accomplissez les étapes nécessaires pour le connecter.   


## <a name="disable-continuous-deployment"></a>Désactiver le déploiement continu 

Lorsque vous désactivez le déploiement continu, votre service de contrôle de code source continue de fonctionner, mais les modifications que vous archivez ne sont pas automatiquement publiées sur Azure. Pour désactiver le déploiement continu, procédez comme suit :

1. Si votre robot a un plan d’hébergement App Service, dans le portail Azure, recherchez votre robot Azure, cliquez sur l’onglet **BUILD**, puis recherchez la section **Déploiement continu à partir du contrôle de code source**, *ou...* 
2. Si votre robot dispose d’un plan de consommation, cliquez sur l’onglet **Paramètres**, développez la section **Déploiement continu**, puis cliquez sur **Configurer**.
3. Dans le volet **Déploiements**, sélectionnez le service de contrôle de code source dans lequel le déploiement continu est activé, puis cliquez sur **Déconnecter**.  


## <a name="additional-resources"></a>Ressources supplémentaires

Pour savoir comment déboguer votre robot localement après avoir configuré le déploiement continu, voir [Déboguer un robot Service Bot](bot-service-debug-bot.md).

Cet article a mis en évidence les fonctionnalités de déploiement continu spécifiques de Service Bot. Pour plus d’informations sur le déploiement continu en lien avec Azure App Services, voir <a href="https://azure.microsoft.com/en-us/documentation/articles/app-service-continuous-deployment/" target="_blank">Déploiement continu sur Azure App Service</a>.
