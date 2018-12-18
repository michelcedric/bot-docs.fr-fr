---
title: Configurer le déploiement continu pour un service de bot | Microsoft Docs
description: Découvrez comment configurer le déploiement continu à partir du contrôle de code source pour un service de bot.
keywords: déploiement continu, publier, déployer, portail azure
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/06/2018
ms.openlocfilehash: ffbc3ef83c8fe1cd6f04697a3fff9e229df9956f
ms.sourcegitcommit: 080b9633925ffe381f2c3cf11c8f8ca4b37e2046
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2018
ms.locfileid: "53068712"
---
# <a name="set-up-continuous-deployment"></a>Configurer un déploiement continu
Si votre code est archivé dans **GitHub** ou **Azure DevOps (anciennement Visual Studio Team Services)**, utilisez le déploiement continu pour déployer automatiquement les modifications de code à partir de votre référentiel source vers Azure. Dans cette rubrique, nous parlerons de la configuration du déploiement continu pour **GitHub** et **Azure DevOps**.

> [!NOTE]
> Le scénario décrit dans cet article suppose que vous avez déployé votre bot sur Azure et que vous souhaitez maintenant activer le déploiement continu pour ce bot. De plus, sachez qu’une fois que le déploiement continu est configuré, l’éditeur de code en ligne dans le portail Azure s’affiche en lecture seule.

## <a name="continuous-deployment-using-github"></a>Déploiement continu avec GitHub

Pour configurer un déploiement continu à l’aide du dépôt GitHub qui contient le code source à déployer sur Azure, effectuez les étapes suivantes :

1. Dans le [portail Azure](https://portal.azure.com), accédez au panneau de **tous les paramètres App Service** et cliquez sur **Options de déploiement (classiques)**. 

1. Cliquez sur **Choisir une source** et sélectionnez **GitHub**.

   ![Choisissez GitHub](~/media/azure-bot-build/continuous-deployment-setup-github.png)

1. Cliquez sur **Autorisation**, puis sur le bouton **Autoriser**, et suivez les invites pour donner l’autorisation à Azure d’accéder à votre compte GitHub.

1. Cliquez sur **Choisir un projet** et sélectionnez un projet.

1. Cliquez sur **Choisir une branche** et sélectionnez une branche.

1. Cliquez sur **OK** pour terminer le processus de configuration.

Votre déploiement continu avec la configuration GitHub est terminé. Chaque fois que vous validez dans le référentiel de code source, vos modifications sont automatiquement déployées dans Azure Bot Service.

## <a name="continuous-deployment-using-azure-devops"></a>Déploiement continu avec Azure DevOps

1. Dans le [portail Azure](https://portal.azure.com), accédez au panneau de **tous les paramètres App Service** et cliquez sur **Options de déploiement (classiques)**. 
2. Cliquez sur **Choisir une source** et sélectionnez **Visual Studio Team Services**. Gardez à l’esprit que Visual Studio Team Services s’appelle désormais Azure DevOps Services.

   ![Choisissez Visual Studio Team Services](~/media/azure-bot-build/continuous-deployment-setup-vs.png)

3. Cliquez sur **Choisir votre compte** et sélectionnez un compte.

> [!NOTE]
> Si votre compte n’est pas listé, vous devrez [le lier à votre abonnement Azure](https://docs.microsoft.com/en-us/azure/devops/organizations/accounts/connect-organization-to-azure-ad?view=vsts&tabs=new-nav). Notez que seuls les projets VSTS Git sont pris en charge.

4. Cliquez sur **Choisir un projet** et sélectionnez un projet.
5. Cliquez sur **Choisir une branche** et sélectionnez une branche.
6. Cliquez sur **OK** pour terminer le processus de configuration.

   ![Configuration de Visual Studio](~/media/azure-bot-build/continuous-deployment-setup-vs-configuration.png)

Votre déploiement continu avec la configuration Azure DevOps est terminé. Chaque fois que vous validez, vos modifications sont automatiquement déployées sur Azure.

## <a name="disable-continuous-deployment"></a>Désactiver le déploiement continu

Même si votre bot est configuré pour le déploiement continu, vous ne pouvez pas utiliser l’éditeur de code en ligne pour apporter des modifications à votre bot. Si vous souhaitez utiliser l’éditeur de code en ligne, vous pouvez désactiver temporairement le déploiement continu.

Pour désactiver le déploiement continu, procédez comme suit :
1. Dans le [portail Azure](https://portal.azure.com), accédez au panneau de **tous les paramètres App Service** et cliquez sur **Options de déploiement (classiques)**. 
2. Cliquez sur **Déconnecter** pour désactiver le déploiement continu. Pour réactiver le déploiement continu, répétez les étapes décrites dans les sections correspondantes ci-dessus.

## <a name="additional-information"></a>Informations supplémentaires
- Visual Studio Team Services s’appelle désormais [Azure DevOps Services](https://docs.microsoft.com/en-us/azure/devops/?view=vsts)
