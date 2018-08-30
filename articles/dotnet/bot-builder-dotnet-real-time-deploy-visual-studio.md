---
title: Déployer un bot multimédia en temps réel Skype dans Azure | Microsoft Docs
description: Découvrez comment déployer un bot audio-vidéo en temps réel Skype dans Azure à l’aide de la fonctionnalité de publication intégrée de Visual Studio.
author: MalarGit
ms.author: malarch
manager: ssulzer
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/17
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: add83b0534ff950e9e7dd5c97a970d251b9c8fea
ms.sourcegitcommit: 2dc75701b169d822c9499e393439161bc87639d2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/24/2018
ms.locfileid: "42904115"
---
# <a name="deploy-a-real-time-media-bot-from-visual-studio-to-azure"></a>Déployer un bot multimédia en temps réel de Visual Studio vers Azure

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

Les bots multimédia en temps réel peuvent être hébergés dans une machine virtuelle Azure « IaaS » ou un service cloud Azure « classique ». Cet article explique comment déployer un bot, hébergé dans un rôle de travail de service cloud Azure à partir de Visual Studio à l’aide de sa fonctionnalité de publication intégrée.

## <a name="prerequisites"></a>Prérequis

Vous devez disposer d’un abonnement Microsoft Azure pour pouvoir déployer un bot dans Azure. Si vous n’avez pas encore d’abonnement, vous pouvez vous inscrire pour un <a href="https://azure.microsoft.com/en-us/free/" target="_blank">compte gratuit</a>. En outre, le processus décrit par cet article nécessite Visual Studio. Si vous n’avez pas Visual Studio, vous pouvez télécharger gratuitement <a href="https://www.visualstudio.com/downloads/" target="_blank">Visual Studio 2017 Community</a>.

### <a name="certificate-from-a-valid-certificate-authority"></a>Certificat délivré par une autorité de certification valide
Le bot doit être configuré avec un certificat valide délivré par une autorité de certificat (CA) approuvée. Le nom du sujet (SN) ou la dernière entrée du nom SAN (Subject Alternative) du certificat doit correspondre au nom du service cloud. Les certificats utilisant des caractères génériques ne sont pas pris en charge actuellement. Si un CNAME sert à pointer vers le service cloud, ce CNAME doit correspondre au SN ou à la dernière entrée SAN du certificat.

## <a name="configure-application-settings"></a>Configurer les paramètres de l’application
Pour que votre bot fonctionne correctement dans le cloud, vous devez vous assurer que ses paramètres d’application sont corrects. Plus précisément, définissez les valeurs clés suivantes dans le fichier app.config de votre rôle de travail :
> <ul><li>MicrosoftAppId</li><li>MicrosoftAppPassword</li></ul>

> [!NOTE]
> Pour trouver les paramètres **AppID** et **AppPassword** de votre bot, consultez la rubrique [MicrosoftAppID and MicrosoftAppPassword](~/bot-service-manage-overview.md#microsoftappid-and-microsoftapppassword) (MicrosoftAppID et MicrosoftAppPassword).

## <a name="create-worker-role-in-the-azure-portal"></a>Créer un rôle de travail dans le portail Azure
### <a name="step-1-create-cloud-serviceclassic"></a>Étape 1 : Créer un service cloud (classique)
Connectez-vous au <a href="https://portal.azure.com">portail Azure</a>. Cliquez sur **+** dans la partie gauche de l’écran, puis choisissez **Services cloud (classiques)**. Fournissez les informations requises dans le formulaire, puis cliquez sur **Créer**.

![Créer le service cloud](../media/real-time-media-bot-portal-service-creation.png)

> [!NOTE]
> Le nom DNS du bot doit être fourni dans l’URL pour l’inscription du bot.

### <a name="step-2-upload-the-certificate-for-the-bot"></a>Étape 2 : Charger le certificat du bot
Une fois le bot créé, chargez son certificat.

![Téléchargement d’un certificat](../media/real-time-media-bot-portal-certificates.png)

## <a name="modify-service-configuration-with-worker-role-details"></a>Modifier la configuration du service avec les détails du rôle de travail
Le nom de domaine complet (FQDN) du bot n’est pas disponible via les API RoleEnvironment Azure. Par conséquent, le bot doit être fourni avec son nom de domaine complet. Il doit également connaître le certificat pour le protocole HTTPS. Ces éléments peuvent être configurés dans le fichier de configuration (.cscfg) du service du rôle de travail.

> [!TIP]
> Si vous déployez un exemple à partir du dépôt Git BotBuilder-RealTimeMediaCalling,
> - remplacez $DnsName$ par le nom du service cloud ou le CNAME s’il est utilisé dans la configuration du service.
>   ```xml
>      <Setting name="ServiceDnsName" value="$DnsName$" />
>   ```
> 
> - Remplacez $CertThumbprint$ par l’empreinte numérique du certificat chargé sur le bot dans les lignes suivantes de la configuration.
>   ```xml
>      <Setting name="DefaultCertificate" value="$CertThumbprint$" />
>      <Certificate name="Default" thumbprint="$CertThumbprint$" thumbprintAlgorithm="sha1" />
>   ```

## <a name="publish-the-bot-from-visual-studio"></a>Publier le bot à partir de Visual Studio
### <a name="step-1-launch-the-microsoft-azure-publishing-wizard-in-visual-studio"></a>Étape 1 : Lancer l’Assistant Publication Microsoft Azure dans Visual Studio

Ouvrez votre projet dans Visual Studio. Dans l’Explorateur de solutions, cliquez avec le bouton droit sur le projet du service cloud, puis sélectionnez **Publier**. L’Assistant Publication Microsoft Azure démarre. Utilisez vos informations d’identification pour vous connecter à l’abonnement approprié.

![Cliquez avec le bouton droit sur le projet et choisissez Publier pour démarrer l’Assistant Publication Microsoft Azure](../media/real-time-media-bot-publish-signin.png)

### <a name="step-2-publish-the-bot"></a>Étape 2 : Publier le bot

Cliquez sur **Suivant**. L’onglet **Paramètres** s’ouvre. Spécifiez le service cloud, l’environnement, la configuration de build et la configuration du service pour le déploiement du bot.

![Cliquez sur Suivant et accédez à l’onglet Paramètres](../media/real-time-media-bot-publish-settings.png)

Vous pouvez éventuellement choisir l’option **Paramètres avancés** et spécifier le compte de stockage des journaux de déploiement (pour déboguer les problèmes).

![Cliquez sur l’onglet Paramètres avancés](../media/real-time-media-bot-publish-advanced-settings.png)

Vérifiez la configuration dans l’onglet **Résumé**, puis cliquez sur **Publier** pour déployer le bot sur Microsoft Azure.
