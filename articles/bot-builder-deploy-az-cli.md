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
ms.date: 01/07/2019
ms.openlocfilehash: 3ebc13cf9e2d111d716d081c36f125d28a441811
ms.sourcegitcommit: bdb981c0b11ee99d128e30ae0462705b2dae8572
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/17/2019
ms.locfileid: "54360739"
---
# <a name="deploy-your-bot-using-azure-cli"></a>Déployer votre bot avec Azure CLI

[!INCLUDE [pre-release-label](./includes/pre-release-label.md)]

Après avoir créé votre bot et l’avoir testé localement, vous pouvez le déployer sur Azure pour le rendre accessible depuis n’importe où. Le déploiement de votre bot sur Azure implique de payer les services que vous utilisez. L’article sur la [gestion de la facturation et des coûts](https://docs.microsoft.com/en-us/azure/billing/) vous aide à comprendre la facturation Azure, à superviser votre utilisation et vos coûts, ainsi qu’à gérer votre compte et vos abonnements.

Dans cet article, nous allons vous montrer comment déployer des bots C# et JavaScript sur Azure avec `az` et `msbot` cli. Il est plus judicieux de lire cet article avant de suivre les étapes, afin de bien comprendre ce qu’implique le déploiement d’un bot.

## <a name="prerequisites"></a>Prérequis

[!INCLUDE [prerequisite snippet](~/includes/deploy/snippet-prerequisite.md)]


## <a name="deploy-javascript-and-c-bots-using-az-cli"></a>Déployer des bots JavaScript et C# avec az cli

Vous avez déjà créé et testé un bot localement et vous souhaitez maintenant le déployer sur Azure. Ces étapes supposent que vous avez créé les ressources Azure nécessaires.

[!INCLUDE [az login snippet](~/includes/deploy/snippet-az-login.md)]

### <a name="create-a-web-app-bot"></a>Créer un bot d’application web

Si vous ne disposez pas déjà d’un groupe de ressources dans lequel publier votre bot, créez-en un :

[!INCLUDE [az create group snippet](~/includes/deploy/snippet-az-create-group.md)]

[!INCLUDE [az create web app snippet](~/includes/deploy/snippet-create-web-app.md)]

Avant de continuer, lisez les instructions qui s’appliquent à votre cas en fonction du type de compte e-mail que vous utilisez pour vous connecter à Azure.

#### <a name="msa-email-account"></a>Compte e-mail MSA

Si vous utilisez un compte e-mail [MSA](https://en.wikipedia.org/wiki/Microsoft_account), vous devez créer un ID et un mot de passe d’application sur le portail d’inscription des applications à utiliser avec la commande `az bot create`.

[!INCLUDE [create bot msa snippet](~/includes/deploy/snippet-create-bot-msa.md)]

#### <a name="business-or-school-account"></a>Compte professionnel ou scolaire

[!INCLUDE [create bot snippet](~/includes/deploy/snippet-create-bot.md)]

### <a name="download-the-bot-from-azure"></a>Télécharger le bot depuis Azure

Ensuite, téléchargez le bot que vous venez de créer. 
[!INCLUDE [download bot snippet](~/includes/deploy/snippet-download-bot.md)]

### <a name="decrypt-the-downloaded-bot-file-and-use-in-your-project"></a>Déchiffrer le fichier .bot téléchargé et l’utiliser dans votre projet

Les informations sensibles du fichier .bot sont chiffrées.

[!INCLUDE [decrypt bot snippet](~/includes/deploy/snippet-decrypt-bot.md)]

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

[!INCLUDE [publish snippet](~/includes/deploy/snippet-publish.md)]

<!-- TODO: If we tell them to re-encrypt, this step is not necessary. -->

[!INCLUDE [clear encryption snippet](~/includes/deploy/snippet-clear-encryption.md)]

## <a name="additional-resources"></a>Ressources supplémentaires

[!INCLUDE [additional resources snippet](~/includes/deploy/snippet-additional-resources.md)]

## <a name="next-steps"></a>Étapes suivantes
> [!div class="nextstepaction"]
> [Configurer le déploiement continu](bot-service-build-continuous-deployment.md)
