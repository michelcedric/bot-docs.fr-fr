---
title: Configurer le déploiement continu pour un service de bot | Microsoft Docs
description: Découvrez comment configurer le déploiement continu à partir du contrôle de code source pour un service de bot.
keywords: déploiement continu, publier, déployer, portail azure
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 03/08/2018
ms.openlocfilehash: 596d264c4df72959c71ab353e5038175fc2bed31
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39299200"
---
# <a name="set-up-continuous-deployment"></a>Configurer un déploiement continu

Le déploiement continu vous permet de développer votre bot localement. Un tel déploiement est utile si votre bot est archivé dans un contrôle de code source comme **GitHub** ou **Visual Studio Team Services**. Quand vous archivez vos modifications dans votre référentiel source, elles sont automatiquement déployées dans Azure.

> [!NOTE]
> Une fois le déploiement continu configuré, l’[éditeur de code en ligne](bot-service-build-online-code-editor.md) devient *LECTURE SEULE*.

Cette rubrique vous explique comment configurer le déploiement continu pour **GitHub** et **Visual Studio Team Services**.

## <a name="continuous-deployment-using-github"></a>Déploiement continu avec GitHub

Pour configurer le déploiement continu à l’aide de GitHub, procédez comme suit :

1. [Dupliquez](https://help.github.com/articles/fork-a-repo/) le référentiel GitHub qui contient le code que vous souhaitez déployer dans Azure.
2. À partir du portail Azure, accédez au panneau **Générer** et cliquez sur **Configurer le déploiement continu**. 
3. Cliquez sur **Setup**.
   
   ![Configurer un déploiement continu](~/media/azure-bot-build/continuous-deployment-setup.png)

4. Cliquez sur **Choisir une source** et choisissez **GitHub**.

   ![Choisissez GitHub](~/media/azure-bot-build/continuous-deployment-setup-github.png)

5. Cliquez sur **Autorisation**, puis sur le bouton **Autoriser**, et suivez les invites pour donner l’autorisation à Azure d’accéder à votre compte GitHub.

Votre déploiement continu avec la configuration GitHub est terminé. Chaque fois que vous validez, vos modifications sont automatiquement déployées sur Azure.

## <a name="continuous-deployment-using-visual-studio"></a>Déploiement continu avec Visual Studio

1. À partir du panneau **Générer** de votre bot, cliquez sur **Configurer le déploiement continu**. 
2. Cliquez sur **Setup**.
   
   ![Configurer un déploiement continu](~/media/azure-bot-build/continuous-deployment-setup.png)

3. Cliquez sur **Choisir une source** et choisissez **Visual Studio Team Services**.

   ![Choisissez Visual Studio Team Services](~/media/azure-bot-build/continuous-deployment-setup-vs.png)

4. Cliquez sur **Choisir votre compte** et choisissez un compte.

> [!NOTE]
> Si vous ne voyez pas votre compte, assurez-vous qu’il est lié à votre abonnement Azure.
> Pour plus d’informations, consultez [Liaison de votre compte VSTS à votre abonnement Azure](https://github.com/projectkudu/kudu/wiki/Setting-up-a-VSTS-account-so-it-can-deploy-to-a-Web-App#linking-your-vsts-account-to-your-azure-subscription).

5. Cliquez sur **Choisir un projet** et choisissez un projet.

> [!NOTE]
> Seuls les projets VSTS Git sont pris en charge.

6. cliquez sur **Choisir la branche** et choisissez une branche à créer.
7. Cliquez sur **OK** pour terminer le processus de configuration.

   ![Configuration de Visual Studio](~/media/azure-bot-build/continuous-deployment-setup-vs-configuration.png)

Votre déploiement continu avec configuration de Visual Studio Team Services est terminé. Chaque fois que vous validez, vos modifications sont automatiquement déployées sur Azure.

## <a name="disable-continuous-deployment"></a>Désactiver le déploiement continu

Même si votre bot est configuré pour le déploiement continu, vous ne pouvez pas utiliser l’éditeur de code en ligne pour apporter des modifications à votre bot. Si vous souhaitez utiliser l’éditeur de code en ligne, vous pouvez désactiver temporairement le déploiement continu.

Pour désactiver le déploiement continu, procédez comme suit :

1. À partir du panneau **Générer** de votre bot, cliquez sur **Configurer le déploiement continu**. 
2. Cliquez sur **Déconnecter** pour désactiver le déploiement continu. Pour réactiver le déploiement continu, répétez les étapes décrites dans les sections correspondantes ci-dessus.

## <a name="next-steps"></a>Étapes suivantes
Maintenant que votre bot est configuré pour le déploiement continu, testez votre code à l’aide de la conversation web en ligne.

> [!div class="nextstepaction"]
> [Tester dans la discussion web](bot-service-manage-test-webchat.md)
