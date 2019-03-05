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
ms.openlocfilehash: fef82d27c6dd4fb61c9a0cf864e76a88128847d7
ms.sourcegitcommit: 05ddade244874b7d6e2fc91745131b99cc58b0d6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 02/21/2019
ms.locfileid: "56591160"
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

Le panneau **Paramètres de l’application** contient des informations détaillées sur votre bot, comme l’environnement du bot, les paramètres de débogage et les clés des paramètres de l’application, comme botFilePath et botFileSecret.

### <a name="microsoftappid-and-microsoftapppassword"></a>MicrosoftAppID et MicrosoftAppPassword

**MicrosoftAppID** et **MicrosoftAppPassword** sont conservés dans le fichier `.bot` du bot. Pour les récupérer, téléchargez le fichier du bot et déchiffrez-le, ce qui peut être nécessaire pour le tester localement avec l’ID et le mot de passe.

### <a name="bot-file-path-and-secret"></a>Chemin et secret du fichier du bot

Vous trouverez **botFilePath** et **botFileSecret** pour votre bot dans le panneau **Paramètres d’application**.

![Chemin et secret du fichier du bot Microsoft](~/media/azure-manage-a-bot/app-settings.png)

> [!NOTE]
> Le Service Bot **Enregistrement de canaux de robot** est fourni avec un *MicrosoftAppID* mais, aucun service d’application n’étant associé à ce type de service, il n’existe pas de panneau **Paramètres de l’application** dans lequel rechercher le *MicrosoftAppPassword*. Pour obtenir le mot de passe, vous devez en générer un. Pour générer le mot de passe pour une **Inscription de canaux de bot**, voir [Mot de passe d’inscription de canaux de robot](bot-service-quickstart-registration.md#bot-channels-registration-password).

## <a name="next-steps"></a>Étapes suivantes
À présent que vous avez exploré le panneau Service Bot dans le portail Azure, découvrez comment utiliser l’éditeur de code en ligne pour personnaliser votre robot.
> [!div class="nextstepaction"]
> [Utiliser l’éditeur de code en ligne](bot-service-build-online-code-editor.md)
