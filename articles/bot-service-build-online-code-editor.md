---
title: Créer un robot avec l’éditeur de code en ligne Azure | Microsoft Docs
description: Découvrez comment créer votre robot à l’aide de l’éditeur de code en ligne dans Service Bot.
keywords: éditeur de code en ligne, portail azure, robot de fonctions
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 03/08/2018
ms.openlocfilehash: bdb287e26c31a784bf6f53ad1601d586781c5ef3
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39299532"
---
# <a name="edit-a-bot-with-online-code-editor"></a>Modifier un robot avec l’éditeur de code en ligne

Vous pouvez utiliser l’éditeur de code en ligne pour créer votre robot sans environnement de développement intégré (IDE). Cette rubrique montre comment ouvrir le code de votre robot dans l’éditeur de code en ligne. 

## <a name="edit-bot-source-code-in-online-code-editor"></a>Modifier le code source d’un robot dans l’éditeur de code en ligne

Pour modifier le code source d’un robot dans l’éditeur de code en ligne, procédez comme suit pour le type spécifique que vous avez.

### <a name="web-app-bot"></a>Robot Web App
1. Connectez-vous au [portail Azure](http://portal.azure.com) et ouvrez le panneau pour le robot.
2. Dans la section **BOT MANAGEMENT** (GESTION DU ROBOT), cliquez sur **Build**.
3. Sélectionnez **Open online code editor** (Ouvrir l’éditeur de code en ligne). Cela a pour effet d’ouvrir le code du robot dans une nouvelle fenêtre de navigateur. 

   ![Ouvrir l’éditeur de code en ligne](~/media/azure-bot-build/open-online-code-editor.png)

   La structure de fichiers sous le répertoire **WWWRoot** varie selon le langage du robot. Par exemple, si vous avez un robot C#, votre répertoire **WWWRoot** peut ressembler à ceci :

   ![Structure de fichiers C#](~/media/azure-bot-build/cs-wwwroot-structure.png)

   Si vous avez un robot Node.js, votre répertoire **WWWRoot** peut ressembler à ceci :

   ![Structure de fichiers Node.js](~/media/azure-bot-build/node-wwwroot-structure.png)

4. Modifiez le code. Par exemple, pour des robots C#, vous pouvez commencer par le fichier **Dialogs/EchoDialog.cs**. Pour des robots Node.js, vous pouvez commencer par le fichier **App.js**.

   > [!NOTE]
   > Si l’éditeur de code en ligne vous permet modifier le code des fichiers sources actuels dans le projet, il ne permet pas de créer des fichiers sources. Pour ajouter de nouveaux fichiers sources au robot, vous devez [télécharger le projet source](bot-service-build-download-source-code.md), ajouter vos fichiers, puis publier les modifications sur Azure.

5. Enregistrez vos modifications. Pour des robots C# figurant dans un **plan de consommation** et tous les robots Node.js, le robot est automatiquement mis à jour une fois le code source enregistré en cliquant sur le bouton **Enregistrer**. 

   ![Structure de fichiers Node.js](~/media/azure-bot-build/node-save-file.png)

   Pour des robots C# sur un plan **App service**, ouvrez le panneau **Console**, puis envoyez la commande **build.cmd**. 

   ![Créer un projet dans le panneau de la console](~/media/azure-bot-build/cs-console-build-cmd.png)
 
   > [!NOTE]
   > Si cette commande échoue, essayez de redémarrer le service d’application de votre robot, puis réessayez de créer le projet. Pour redémarrer votre service d’application, dans le panneau de votre robot, cliquez sur **All App service settings** (Tous les paramètres App service), puis cliquez sur le bouton **Redémarrer**.
   > ![Redémarrer une application web](~/media/azure-bot-build/open-online-code-editor-restart-appservice.png)

6. Revenez au portail Azure, puis cliquez sur **Test in Web Chat** (Tester dans Discussion Web) pour tester vos modifications. Si Discussion Web est déjà ouvert pour ce robot, cliquez sur **Start over** (Recommencer) pour voir les nouvelles modifications.

### <a name="functions-bot"></a>Robot Functions

1. Connectez-vous au [portail Azure](http://portal.azure.com) et ouvrez le panneau pour le robot.
2. Dans la section **BOT MANAGEMENT** (GESTION DU ROBOT), cliquez sur **Build**.
3. Cliquez sur **Open this bot in Azure Functions** (Ouvrir ce robot dans Azure Functions). Cela a pour effet d’ouvrir le robot dans l’interface utilisateur d’<a href="http://go.microsoft.com/fwlink/?linkID=747839" target="_blank">Azure Functions</a>. 
4. Modifiez le code. Par exemple, mettez à jour le code des messages de la fonction. La capture d’écran ci-dessous montre le code Messages pour un robot Functions Node.js.

   ![Éditeur de code Messages de Robot Functions](~/media/azure-bot-build/functions-messages-code.png)

5. Enregistrez vos modifications du code.
6. Revenez au portail Azure, puis cliquez sur **Test in Web Chat** (Tester dans Discussion Web) pour tester vos modifications. Si Discussion Web est déjà ouvert pour ce robot, cliquez sur **Start over** (Recommencer) pour voir les nouvelles modifications.

## <a name="next-steps"></a>Étapes suivantes
À présent que vous savez comment modifier le code de votre robot à l’aide de l’éditeur de code en ligne, vous pouvez également créer votre robot localement à l’aide de l’environnement de développement intégré (IDE) de votre choix.

> [!div class="nextstepaction"]
> [Télécharger le code source du robot](bot-service-build-download-source-code.md)
