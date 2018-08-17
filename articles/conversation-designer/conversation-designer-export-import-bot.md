---
title: Importer et exporter un bot Conversation Designer | Microsoft Docs
description: Découvrez comment importer et exporter des bots Conversation Designer.
author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ROBOTS: NoIndex, NoFollow
ms.openlocfilehash: ed055cb0f75d148e6a0dd3248366851901d9a675
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39300360"
---
# <a name="export-and-import-a-conversation-designer-bot"></a>Exporter et importer un bot Conversation Designer

Les bots contenus dans Conversation Designer peuvent être exportés sur votre ordinateur sous forme de fichier .zip. Ceci vous permet de faire une sauvegarde de votre bot. Vous pourrez ultérieurement restaurer votre bot à un état antérieur à l’aide de n’importe lequel des fichiers .zip exportés. 

## <a name="export-a-conversation-designer-bot"></a>Exporter un bot Conversation Designer

L’exportation vous permet d’enregistrer l’état actuel de votre bot Conversation Designer sur votre ordinateur local. 

Pour exporter un bot Conversation Designer, procédez comme suit :
1. Dans le coin supérieur droit du panneau de navigation, cliquez sur le bouton de sélection (**...** ).
2. Cliquez sur **Exporter**. Le serveur recueille les informations nécessaires et retourne la possibilité d’enregistrer le fichier .zip.
3. Enregistrez le fichier .zip exporté sur votre ordinateur local.

Votre bot est désormais enregistré dans un fichier .zip sur votre ordinateur. Vous pourrez restaurer ultérieurement le bot à cet état en [le réimportant](#import-a-conversation-designer-bot) dans le bot.

## <a name="import-a-conversation-designer-bot"></a>Importer un bot Conversation Designer

L’importation vous permet de restaurer le bot Conversation Designer à un état antérieur. L’importation remplace le bot actuel par le bot importé. Si vous ne souhaitez pas perdre ce que vous avez actuellement dans le bot actuel, veillez à [exporter](#export-a-conversation-designer-bot) le bot actuel avant d’effectuer une opération d’importation.

Pour importer un bot Conversation Designer, procédez comme suit :
1. Dans le coin supérieur droit du panneau de navigation, cliquez sur le bouton de sélection (**...** ).
2. Cliquez sur **Importer**. 
3. Si vous souhaitez enregistrer votre travail dans le robot actif, choisissez l’option **Backup and import** (Sauvegarder et importer). Cette option enregistre le bot actuel sur votre ordinateur local, puis vous demande l’emplacement du fichier .zip à importer. Autrement, choisissez **Import without backing up** (importer sans sauvegarder).

Votre bot est maintenant importé.

> [!NOTE]
> Vous ne pouvez importer que des bots exportés par Conversation Designer.

## <a name="import-a-luis-app-into-a-conversation-designer-bot"></a>Importer une application LUIS dans un bot Conversation Designer

Si vous avez une application LUIS que vous souhaitez utiliser dans votre bot Conversation Designer, vous pouvez importer votre application LUIS dans votre bot Conversation Designer. En principe, ce processus nécessite que vous exportiez votre application LUIS et le bot Conversation Designer, puis que vous échangiez le contenu du fichier **luis.model** de votre bot avec votre fichier **luis.json**. Ensuite, réimportez vos modifications dans votre bot Conversation Designer. En fait, vous remplacez les intentions LUIS de votre bot par celles de votre application LUIS. Pour cette raison, il est conseillé d’effectuer cette opération d’importation avant de commencer à personnaliser les intentions de votre bot LUIS. Sinon, tout votre travail sera remplacé par cette opération d’importation.

> [!NOTE]
> Si vous apportez des modifications à une application [LUIS](https://luis.ai) associée à votre bot (chaque bot Conversation Designer a une application LUIS correspondante), vous n’avez pas besoin d’effectuer ces étapes. Au lieu de cela, il vous suffit d’accéder à votre bot Conversation Designer et de cliquer sur [**Enregistrer**](conversation-designer-save-bot.md).

Pour importer votre application LUIS dans votre bot Conversation Designer, procédez comme suit :

1. À partir de [LUIS.ai](https://luis.ai), ouvrez votre application LUIS, puis cliquez sur **PARAMÈTRES**.
2. Choisissez les **versions** de l’application que vous souhaitez utiliser et cliquez sur l’icône d’action **{ }**. Cette action télécharge le fichier **luis.json** sur votre ordinateur local. 
3. À partir du Concepteur de Conversation Designer, [créez un bot](conversation-designer-create-bot.md#create-a-conversation-designer-bot) ou ouvrez un bot existant.
4. [Exporter](#export-a-conversation-designer-bot) votre bot. Votre bot sera exporté en tant que fichier .zip sur votre ordinateur.
5. Accédez à votre fichier .zip exporté et extrayez-le.
6. Ouvrez le fichier **luis.model** dans un éditeur de texte et remplacez le contenu de ce fichier par le contenu du fichier **luis.json**. Enregistrez le fichier .
7. Zippez le dossier de votre bot Conversation Designer.
8. [Réimportez](#import-a-conversation-designer-bot) votre bot dans votre bot Conversation Designer.

Votre bot peut désormais utiliser les nouvelles intentions LUIS que vous venez d’importer.

## <a name="next-step"></a>Étape suivante
> [!div class="nextstepaction"]
> [Tâches :](conversation-designer-tasks.md)
