---
title: Définir une réponse en tant qu’action Do | Microsoft Docs
description: Découvrez comment définir une réponse sous la forme d’une action Do.
author: vkannan
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ROBOTS: NoIndex, NoFollow
ms.openlocfilehash: 344a771a55cf729587863a70398ef944863ac738
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39299876"
---
# <a name="define-a-reply-as-a-do-action"></a>Définir une réponse en tant qu’action Do

Une réponse peut correspondre à n’importe quelle combinaison de texte affiché, de texte parlé ou de carte enrichie. Avec cette action, le bot envoie une réponse à l’utilisateur et exécute la tâche. Une réponse est une conversation à un seul tour, ce qui signifie que la tâche n’attend pas de messages de suivi de la part de l’utilisateur. Pour configurer une réponse en tant qu’action *reply*, sélectionnez **Give a reply** (Donner une réponse) dans la liste déroulante d’actions **DO**. Vous pouvez ensuite entrer des réponses simples dans le champ **Bot’s response to user** (Réponse du bot à l’utilisateur).

Vous pouvez éventuellement inclure une [carte adaptative](conversation-designer-adaptive-cards.md) avec la réponse simple. Si vous le souhaitez, vous pouvez également fournir une fonction de script personnalisé à exécuter avant l’envoi de la réponse à l’utilisateur. Ces options vous sont accessibles lorsque vous créez ou modifiez une tâche. 

## <a name="next-step"></a>Étape suivante
> [!div class="nextstepaction"]
> [Action de script](conversation-designer-script-function.md)
