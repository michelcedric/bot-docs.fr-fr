---
title: Gérer un robot | Microsoft Docs
description: Découvrez comment gérer un robot via le portail Service Bot.
keywords: portail azure, gestion de robot, test dans discussion web, MicrosoftAppID, MicrosoftAppPassword, paramètres ’application
author: v-ducvo
ms.author: rstand
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: abs
ms.date: 12/13/2017
ms.openlocfilehash: e68db6c1fe9d3d136a8643652df034fb6df2858f
ms.sourcegitcommit: 8b7bdbcbb01054f6aeb80d4a65b29177b30e1c20
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/14/2018
ms.locfileid: "51645599"
---
# <a name="manage-a-bot"></a>Gérer un robot

Cette rubrique explique comment gérer un robot à l’aide du portail Azure.

## <a name="bot-settings-overview"></a>Vue d’ensemble des paramètres du robot

![Vue d’ensemble des paramètres du robot](~/media/azure-manage-a-bot/overview.png)

Le panneau **Vue d’ensemble** présente des informations de haut niveau sur votre robot. Par exemple, vous pouvez y voir l’**ID d’abonnement**, le **niveau tarifaire** et le **point de terminaison de messagerie** de votre robot.

## <a name="bot-management"></a>Gestion du robot

 La plupart des options de gestion de votre robot figurent dans la section **BOT MANAGEMENT** (GESTION DU ROBOT). Voici la liste des options pour vous aider à gérer votre robot.

![Gestion du robot](~/media/azure-manage-a-bot/bot-management.png)

| Option |  Description |
| ---- | ---- |
| **Créer** | L’onglet Créer fournit des options pour apporter des modifications à votre robot. Cette option n’est pas disponible pour le **Bot d’inscription uniquement**. |
| **Tester dans Discussion Web** | Utilisez le contrôle intégré Discussion Web pour vous aider à tester rapidement votre robot. |
| **Analyse** | Si l’analytique est activée pour votre robot, vous pouvez afficher les données d’analyse qu’Application Insights a collectées pour votre robot. |
| **Canaux** | Configurez les canaux que votre robot utilise pour communiquer avec les utilisateurs. |
| **Paramètres** | Gérer les divers paramètres de profil de robot, tels que le nom d’affichage, l’analytique et de point de terminaison de messagerie. |
| **Préparation vocale** | Gérer les connexions entre votre application LUIS et le service Reconnaissance vocale Bing... |
| **Tarification de Service Bot** | Gérer le niveau tarifaire pour le Service Bot. |

## <a name="app-service-settings"></a>Paramètres d’App Service

![Paramètres d’App Service](~/media/azure-manage-a-bot/app-service-settings.png)

Le panneau **Paramètres de l’application** contient des informations détaillées sur votre robot, telles que l’environnement du robot, l’ID, la clé Application Insights, l’ID d’application Microsoft et le mot de passe d’application Microsoft.

### <a name="microsoftappid-and-microsoftapppassword"></a>MicrosoftAppID et MicrosoftAppPassword

Vous pouvez trouver l’élément **MicrosoftAppID** de votre bot dans le panneau **Paramètres**. L’élément **MicrosoftAppPassword** de votre bot s’affiche uniquement lorsque vous créez votre bot.

![ID et mot de passe d’application Microsoft](~/media/azure-manage-a-bot/app-settings.png)

> [!NOTE]
> Le Service Bot **Enregistrement de canaux de robot** est fourni avec un *MicrosoftAppID* mais, aucun service d’application n’étant associé à ce type de service, il n’existe pas de panneau **Paramètres de l’application** dans lequel rechercher le *MicrosoftAppPassword*. Pour obtenir le mot de passe, vous devez en générer un. Pour générer le mot de passe pour une **Inscription de canaux de bot**, voir [Mot de passe d’inscription de canaux de robot](bot-service-quickstart-registration.md#bot-channels-registration-password).

## <a name="next-steps"></a>Étapes suivantes
À présent que vous avez exploré le panneau Service Bot dans le portail Azure, découvrez comment utiliser l’éditeur de code en ligne pour personnaliser votre robot.
> [!div class="nextstepaction"]
> [Utiliser l’éditeur de code en ligne](bot-service-build-online-code-editor.md)
