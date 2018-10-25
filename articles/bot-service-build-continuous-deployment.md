---
title: Configurer le déploiement continu pour un service de bot | Microsoft Docs
description: Découvrez comment configurer le déploiement continu à partir du contrôle de code source pour un service de bot.
keywords: déploiement continu, publier, déployer, portail azure
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 09/18/2018
ms.openlocfilehash: 62cbbcc560e049776b8aa891c167b9a6eaba3264
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2018
ms.locfileid: "49997796"
---
# <a name="set-up-continuous-deployment"></a>Configurer un déploiement continu
Si votre code est archivé dans **GitHub** ou **Azure DevOps (anciennement Visual Studio Team Services)**, utilisez le déploiement continu pour déployer automatiquement les modifications de code à partir de votre référentiel source vers Azure. Dans cette rubrique, nous parlerons de la configuration du déploiement continu pour **GitHub** et **Azure DevOps**.

> [!NOTE]
> Une fois le déploiement continu configuré, l’éditeur de code en ligne dans le Portail Azure s’affiche en *LECTURE SEULE*.

## <a name="continuous-deployment-using-github"></a>Déploiement continu avec GitHub

Pour configurer le déploiement continu à l’aide de GitHub, procédez comme suit :

1. Utilisez le référentiel GitHub qui contient le code source que vous souhaitez déployer dans Azure. Vous pouvez soit [dupliquer](https://help.github.com/articles/fork-a-repo/) un référentiel existant soit créer le vôtre. Ensuite, chargez le code source associé dans votre référentiel GitHub.
2. À partir du [portail Azure](https://portal.azure.com), accédez au panneau **Générer** de votre bot et cliquez sur **Configurer le déploiement continu**. 
3. Cliquez sur **Setup**.
   
   ![Configurer un déploiement continu](~/media/azure-bot-build/continuous-deployment-setup.png)

4. Cliquez sur **Choisir une source** et sélectionnez **GitHub**.

   ![Choisissez GitHub](~/media/azure-bot-build/continuous-deployment-setup-github.png)

5. Cliquez sur **Autorisation**, puis sur le bouton **Autoriser**, et suivez les invites pour donner l’autorisation à Azure d’accéder à votre compte GitHub.

6. Cliquez sur **Choisir un projet** et sélectionnez un projet.

7. Cliquez sur **Choisir une branche** et sélectionnez une branche.

8. Cliquez sur **OK** pour terminer le processus de configuration.

Votre déploiement continu avec la configuration GitHub est terminé. Chaque fois que vous validez dans le référentiel de code source, vos modifications sont automatiquement déployées dans Azure Bot Service.

## <a name="continuous-deployment-using-azure-devops"></a>Déploiement continu avec Azure DevOps

1. À partir du [portail Azure](https://portal.azure.com), accédez au panneau **Générer** de votre bot et cliquez sur **Configurer le déploiement continu**. 
2. Cliquez sur **Setup**.
   
   ![Configurer un déploiement continu](~/media/azure-bot-build/continuous-deployment-setup.png)

3. Cliquez sur **Choisir une source** et sélectionnez **Visual Studio Team Services**. Gardez à l’esprit que Visual Studio Team Services s’appelle désormais Azure DevOps Services.

   ![Choisissez Visual Studio Team Services](~/media/azure-bot-build/continuous-deployment-setup-vs.png)

4. Cliquez sur **Choisir votre compte** et sélectionnez un compte.

> [!NOTE]
> Si vous ne voyez pas votre compte, assurez-vous qu’il est lié à votre abonnement Azure. Pour associer votre compte à votre abonnement Azure, accédez au portail Azure, puis à la section **organisations Azure DevOps Services (anciennement Team Services)**. La liste des organisations dont vous disposez dans Azure DevOps s’affiche. Cliquez sur celle qui contient le code source du bot que vous voulez déployer, puis sur le bouton **Connecter AAD**. Si l’organisation que vous sélectionnez n’est pas associée à votre abonnement Azure, ce bouton est actif. Cliquez donc sur ce bouton pour configurer la connexion. Vous devrez peut-être patienter quelques instants avant que l’opération s’effectue.

5. Cliquez sur **Choisir un projet** et sélectionnez un projet.

> [!NOTE]
> Seuls les projets VSTS Git sont pris en charge.

6. Cliquez sur **Choisir une branche** et sélectionnez une branche.
7. Cliquez sur **OK** pour terminer le processus de configuration.

   ![Configuration de Visual Studio](~/media/azure-bot-build/continuous-deployment-setup-vs-configuration.png)

Votre déploiement continu avec la configuration Azure DevOps est terminé. Chaque fois que vous validez, vos modifications sont automatiquement déployées sur Azure.

## <a name="disable-continuous-deployment"></a>Désactiver le déploiement continu

Même si votre bot est configuré pour le déploiement continu, vous ne pouvez pas utiliser l’éditeur de code en ligne pour apporter des modifications à votre bot. Si vous souhaitez utiliser l’éditeur de code en ligne, vous pouvez désactiver temporairement le déploiement continu.

Pour désactiver le déploiement continu, procédez comme suit :

1. À partir du panneau **Générer** de votre bot, cliquez sur **Configurer le déploiement continu**. 
2. Cliquez sur **Déconnecter** pour désactiver le déploiement continu. Pour réactiver le déploiement continu, répétez les étapes décrites dans les sections correspondantes ci-dessus.

## <a name="additional-information"></a>Informations supplémentaires
- [Azure DevOps](https://docs.microsoft.com/en-us/azure/devops/?view=vsts)
