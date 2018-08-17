---
title: Créer un bot Conversation Designer | Microsoft Docs
description: Découvrez comment enregistrer et former le modèle de compréhension du langage et préparer la reconnaissance vocale dans un bot Conversation Designer.
author: vkannan
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ROBOTS: NoIndex, NoFollow
ms.openlocfilehash: 3a911158c379f879c0be604fb5e8ba30ab22ee44
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39300325"
---
# <a name="saving-your-conversation-designer-bot"></a>Enregistrer votre bot Conversation Designer
> [!IMPORTANT]
> Conversation Designer n’est pas encore accessible à tous les clients. Des informations supplémentaires sur la disponibilité de Conversation Designer seront communiquées plus tard cette année.

Lorsque vous travaillez dans Conversation Designer, tout votre travail est mis en cache dans la mémoire du navigateur. Pour valider vos modifications, cliquez sur le bouton **Enregistrer** situé dans le coin supérieur gauche du menu de navigation gauche. Pour éviter de perdre votre travail, il est recommandé de l’enregistrer souvent. Vous pouvez soit cliquer sur le bouton **Enregistrer**, soit enregistrer votre travail à l’aide du raccourci clavier **CTRL + S**.

## <a name="trains-luis-and-primes-speech-recognition"></a>Effectue l’apprentissage LUIS et prépare la reconnaissance vocale

Lorsque vous cliquez sur le bouton **Enregistrer**, les modifications sont enregistrées et quelques tâches supplémentaires effectuées. À la différence du raccourci clavier, cliquer sur le bouton **Enregistrer** équivaut également à demander à Conversation Designer d’effectuer les tâches suivantes :

- Effectue l’apprentissage de n’importe quelle nouvelle intention LUIS et des entités pour le bot et publie le modèle LUIS localement dans votre service Bot (si nécessaire). Ces intentions peuvent être ajoutées à Conversation Designer ou à l’application [LUIS](https://luis.ai) correspondante du bot.
- Met à jour le runtime pour utiliser le nouveau modèle LUIS.
- Préparation de la reconnaissance vocale en préparant et en envoyant vos exemples d’énoncés à Microsoft Cognitive Services, ce qui améliore considérablement la précision de la reconnaissance vocale pour ce bot.

## <a name="next-step"></a>Étape suivante
> [!div class="nextstepaction"]
> [Tester le bot](conversation-designer-debug-bot.md)
