---
title: Définir un module de reconnaissance de code comme déclencheur de tâche | Microsoft Docs
description: Découvrez comment utiliser un module personnalisé de reconnaissance de code comme déclencheur de tâche.
author: vkannan
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ROBOTS: NoIndex, NoFollow
ms.openlocfilehash: 6d44cd299e46971b2218bb78d16e454d5df3c1d9
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39298795"
---
# <a name="define-code-recognizer-as-task-trigger"></a>Définir un module de reconnaissance de code comme déclencheur de tâche
> [!IMPORTANT]
> Conversation Designer n’est pas encore accessible à tous les clients. Des informations supplémentaires sur la disponibilité de Conversation Designer seront communiquées plus tard cette année.

Les modules de reconnaissance de code permettent d’écrire des scripts personnalisés afin de faciliter l’exécution d’une tâche. L’évaluation des expressions basées sur les expressions régulières ou l’appel d’autres services permettent de définir l’intention de l’utilisateur. Le script personnalisé demande au runtime de conversation de déclencher une tâche. 

## <a name="create-a-script-function"></a>Créer une fonction de script
Pour utiliser un script en tant que module de reconnaissance, accédez à l’éditeur de tâches, choisissez « Fonction de script » comme module de reconnaissance, précisez le nom de la fonction, puis cliquez sur **Créer/afficher la fonction** pour lancer l’édition d’un script. Vous pouvez également cliquer sur **Créer/afficher une fonction** sans indiquer de nom de script. Une fonction vide est alors créée avec un nom par défaut. 

## <a name="script-trigger-function-parameter"></a>Paramètre de la fonction de déclenchement de script

La fonction de rappel de script indiquée est toujours appelée avec l’objet [`context`](conversation-designer-context-object.md).

L’objet `context` comprend `taskEntities` et `contextEntities`. Les entités de tâche sont des entités définies ou générées pour la tâche en question. Les entités de contexte sont des conteneurs de propriétés dans lesquels il est possible de conserver les entités pendant les conversations avec l’utilisateur.

## <a name="return-value"></a>Valeur de retour

Les fonctions de **déclenchement de script** doivent retourner un booléen.

## <a name="sample-regex-based-recognizer"></a>Exemple de module de reconnaissance basé sur les expressions régulières
L’exemple de code suivant se sert des expressions régulières pour traiter la requête et retourne un booléen afin que le temps d’exécution de la conversation permette de déterminer quel déclenchement de script doit être exécuté.

```javascript
module.exports.fnFindPhoneTrigger = function(context) {
    if (context.request.text.includes("call") || context.request.text.includes("ring")) {
        return true;
    } else {
        return false;
    }
} 
```

## <a name="next-step"></a>Étape suivante
> [!div class="nextstepaction"]
> [Action de réponse](conversation-designer-reply.md)
