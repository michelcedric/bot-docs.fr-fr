---
title: Tutoriel pour créer et déployer un bot de base | Microsoft Docs
description: Apprenez à créer un bot de base et à le déployer dans Azure.
keywords: bot echo, déployer, azure, tutoriel
author: Ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 1/9/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 7927ab97dc88657a198c8f1d8e56bcb1ddf0fabe
ms.sourcegitcommit: b2245df2f0a18c5a66a836ab24a573fd70c7d272
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 03/07/2019
ms.locfileid: "57568236"
---
# <a name="tutorial-create-and-deploy-a-basic-bot"></a>Didacticiel : Créer et déployer un bot de base

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Ce tutoriel vous aide à créer pas à pas un bot de base avec le kit SDK Bot Framework et à le déployer dans Azure. Si vous avez déjà créé un bot de base et qu’il s’exécute localement, passez directement à la section [Déployer votre bot](#deploy-your-bot).

Ce tutoriel vous montre comment effectuer les opérations suivantes :

> [!div class="checklist"]
> * Créer un bot Echo de base
> * L’exécuter et interagir avec lui localement
> * Publier votre bot

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

[!INCLUDE [dotnet quickstart](~/includes/quickstart-dotnet.md)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

[!INCLUDE [javascript quickstart](~/includes/quickstart-javascript.md)]

---

## <a name="deploy-your-bot"></a>Déployer votre bot

À présent, déployez votre bot dans Azure.

### <a name="prerequisites"></a>Prérequis

[!INCLUDE [prerequisite snippet](~/includes/deploy/snippet-prerequisite.md)]

### <a name="login-to-azure-cli-and-set-your-subscription"></a>Se connecter à Azure CLI et définir votre abonnement

Vous avez déjà créé et testé un bot localement et vous souhaitez maintenant le déployer sur Azure.

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

[!INCLUDE [download keys snippet](~/includes/snippet-abs-key-download.md)]

### <a name="decrypt-the-downloaded-bot-file-and-use-in-your-project"></a>Déchiffrer le fichier .bot téléchargé et l’utiliser dans votre projet

Les informations sensibles contenues dans le fichier .bot étant chiffrées, déchiffrons-les pour pouvoir les utiliser facilement. 

Tout d’abord, accédez au répertoire bot téléchargé.

[!INCLUDE [decrypt bot snippet](~/includes/deploy/snippet-decrypt-bot.md)]

### <a name="test-your-bot-locally"></a>Tester votre bot localement

À ce stade, votre bot doit fonctionner de la même façon qu’avec l’ancien fichier `.bot`. Vérifiez qu’il fonctionne normalement avec le nouveau fichier `.bot`.

Un point de terminaison *Production* doit à présent figurer dans l’émulateur. Si ce n’est pas le cas, vous utilisez probablement l’ancien fichier `.bot`.

### <a name="publish-your-bot-to-azure"></a>Publier votre bot sur Azure

<!-- TODO: re-encrypt your .bot file? -->

[!INCLUDE [publish snippet](~/includes/deploy/snippet-publish.md)]

<!-- TODO: If we tell them to re-encrypt, this step is not necessary. -->

[!INCLUDE [clear encryption snippet](~/includes/deploy/snippet-clear-encryption.md)]

À présent, vous devez pouvoir tester votre bot dans Web Chat.

## <a name="additional-resources"></a>Ressources supplémentaires

[!INCLUDE [additional resources snippet](~/includes/deploy/snippet-additional-resources.md)]

## <a name="next-steps"></a>Étapes suivantes
> [!div class="nextstepaction"]
> [Ajouter des services à votre bot](bot-builder-tutorial-add-qna.md)

