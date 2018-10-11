---
title: Configurer la préparation vocale | Microsoft Docs
description: Découvrez comment configurer la préparation vocale pour votre service de bot à l’aide du Portail Azure.
keywords: préparation vocale, reconnaissance vocale, LUIS
author: v-royhar
ms.author: v-royhar
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: ba2aec255cf160a72c11c3ddfda021baae304568
ms.sourcegitcommit: 3cb288cf2f09eaede317e1bc8d6255becf1aec61
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/27/2018
ms.locfileid: "47389638"
---
# <a name="configure-speech-priming"></a>Configurer la préparation vocale

La préparation vocale améliore la reconnaissance du texte parlé fréquemment utilisé dans votre bot. Pour les bots à reconnaissance vocale utilisant les canaux Discussion Web et Cortana, la préparation vocale utilise les exemples spécifiés dans les applications LUIS (Language Understanding Intelligent Service) pour améliorer la précision de la reconnaissance vocale des mots importants.

Votre bot est peut-être déjà intégré à une application LUIS. Vous pouvez également choisir de créer une application LUIS et l’associer à votre bot pour la préparation vocale. L’application LUIS contient des exemples d’expressions que les utilisateurs sont susceptibles de dire à votre bot. Les mots importants qui doivent être reconnus par le bot doivent être étiquetés en tant qu’entités. Par exemple, pour un bot de jeu d’échec, vous voudrez vous assurer que lorsque l’utilisateur dit « Déplacer la reine », la phrase ne sera pas interprétée comme « Déplacer l’arène ». L’application LUIS doit inclure des exemples dans lesquels le mot « reine » est étiqueté en tant qu’entité.

> [!NOTE]
> Pour utiliser la préparation vocale avec le canal Discussion Web, vous devez recourir au service Reconnaissance vocale Bing. Consultez la rubrique [Enable speech in the Web Chat channel](~/bot-service-channel-connect-webchat-speech.md) (Activer la reconnaissance vocale dans le canal Discussion Web) pour découvrir comment utiliser le service Reconnaissance vocale Bing.

> [!IMPORTANT]
> La préparation vocale s’applique uniquement aux bots configurés pour le canal Cortana ou le canal Discussion Web.

> [!IMPORTANT]
> La préparation n’est pas prise en charge depuis les applications LUIS régionales hors des États-Unis (y compris : eu.luis.ai et au.luis.ai).

## <a name="change-the-list-of-luis-apps-your-bot-uses"></a>Modifier la liste des applications LUIS utilisées par votre bot

Pour modifier la liste des applications LUIS utilisées par le service Reconnaissance vocale Bing avec votre bot, procédez comme suit :

1. Cliquez sur **Speech priming** (Préparation vocale) dans le panneau du service de bot. La liste des applications LUIS à votre disposition s’affiche.
2. Sélectionnez les applications LUIS que le service Reconnaissance vocale Bing doit utiliser.
 
    a. Pour sélectionner une application LUIS dans la liste, pointez sur le modèle LUIS jusqu’à ce qu’une case à cocher s’affiche, puis cochez la case.
     
    b. Pour sélectionner une application LUIS qui n’est pas dans la liste, faites défiler l’écran jusqu’en bas, puis entrez l’identificateur unique de l’application LUIS dans la zone de texte.
     
3. Cliquez sur **Enregistrer** pour enregistrer la liste des applications LUIS associées au service Reconnaissance vocale Bing pour votre bot.

![Panneau de préparation vocale](~/media/bot-service-manage-speech-priming/speech-priming.png)

## <a name="additional-resources"></a>Ressources supplémentaires

- Pour en savoir plus sur l’activation de la reconnaissance vocale dans le canal Discussion Web, consultez la rubrique [Enable speech in the Web Chat channel](~/bot-service-channel-connect-webchat-speech.md) (Activer la reconnaissance vocale dans le canal Discussion Web).
- Pour en savoir plus sur la préparation vocale, consultez l’article [Speech Support in Bot Framework – Web Chat to Directline, to Cortana](https://blog.botframework.com/2017/06/26/Speech-To-Text/) (Prise en charge de la reconnaissance vocale dans le Bot Framework : Discussion Web, Directline et Cortana).
- Pour en savoir plus sur les applications LUIS, consultez le site [Language Understanding Intelligent Service](https://www.luis.ai).
