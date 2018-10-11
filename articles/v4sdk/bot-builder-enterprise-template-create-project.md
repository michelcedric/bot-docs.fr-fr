---
title: Bot d’entreprise pour la création d’un nouveau projet | Microsoft Docs
description: Découvrez comment créer un bot en vous basant sur le modèle d’un bot d’entreprise
author: darrenj
ms.author: darrenj
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 09/18/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 2c6736cc8aa607da73c392b04ea894a19c86ff29
ms.sourcegitcommit: 87b5b0ca9b0d5e028ece9f7cc4948c5507062c2b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/24/2018
ms.locfileid: "47029777"
---
# <a name="enterprise-bot-template---creating-a-new-project"></a>Modèle de bot d’entreprise - Création d’un projet

> [!NOTE]
> Cet article s’applique à la version v4 du Kit de développement logiciel (SDK). 

Le modèle de bot d’entreprise réunit toutes les meilleures pratiques et tous les composants de prise en charge que nous avons identifiés lors de la création d’expériences conversationnelles. Le modèle est disponible dans les plateformes de Kit de développement logiciel (SDK) Bot Builder suivantes :

- .NET
- Node.js (prochainement)

## <a name="net"></a>.NET

Le modèle de bot d’entreprise est disponible pour la plateforme .NET, ciblant les versions **V4** du Kit de développement logiciel (SDK). Il est disponible en tant que package [VSIX](https://docs.microsoft.com/en-us/visualstudio/extensibility/anatomy-of-a-vsix-package). Pour le télécharger, cliquez sur le lien suivant :

- [Modèle de bot d’entreprise Kit de développement logiciel (SDK) Bot Builder V4](https://aka.ms/GetEnterpriseBotTemplate)

#### <a name="prerequisites"></a>Prérequis

- [Visual Studio 2017 ou version ultérieure](https://www.visualstudio.com/downloads/)
- [Compte Azure](https://azure.microsoft.com/en-us/free/)
- [Azure PowerShell](https://docs.microsoft.com/en-us/powershell/azure/overview?view=azurermps-6.8.1)

### <a name="install-the-template"></a>Installer le modèle

À partir du répertoire enregistré, ouvrez simplement le package VSIX ; le modèle de bot d’entreprise est alors installé dans Visual Studio et sera disponible à sa prochaine ouverture.

Pour créer un projet de bot à l’aide du modèle, ouvrez simplement Visual Studio et sélectionnez **Fichier** > **Nouveau** > **Projet** et à partir de Visual C# sélectionnez **Bot Framework** > Enterprise Bot Template (Modèle de bot d’entreprise). Cette opération crée un projet de bot localement que vous pouvez modifier comme vous le souhaitez. 

![Fichier - Nouveau - Projet - Modèle](media/enterprise-template/EnterpriseBot-NewProject.png)

## <a name="deploy-your-bot"></a>Déployer votre bot

Maintenant que votre projet est créé, la prochaine étape consiste à créer l’infrastructure de prise en charge Azure et à effectuer des opérations de configuration/déploiement pour permettre au bot d’être tout de suite opérationnel. Poursuivez avec l’article [Enterprise Bot Template - Deploying your Bot](bot-builder-enterprise-template-deployment.md) (Modèle de bot d’entreprise - Déploiement de votre bot).

> Vous devez effectuer cette étape, sinon l’initialisation du bot (AppInsights) et les dépendances LUIS ne seront pas disponibles.
## <a name="customize-your-bot"></a>Personnaliser votre bot

Après vous être assuré du bon déploiement du bot prédéfini, vous pouvez le personnaliser en fonction de votre scénario et de vos besoins. Poursuivez avec l’article [Enterprise Bot Template - Customize your Bot](bot-builder-enterprise-template-customize.md) (Modèle de bot d’entreprise - Personnaliser votre bot).
