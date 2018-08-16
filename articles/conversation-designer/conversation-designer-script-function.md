---
title: Définir une fonction de script en tant qu’action Do | Microsoft Docs
description: Apprenez à configurer une fonction d’action de script en action Do.
author: vkannan
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ROBOTS: NoIndex, NoFollow
ms.openlocfilehash: 674781c2cb5a8700941feb59b2ce7ab902979d4f
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39300309"
---
# <a name="define-a-script-function-as-a-do-action"></a>Définir une fonction de script en tant qu’action Do

Les [fonctions d’action de script](conversation-designer-context-object.md#script-callback-functions) exécutent le script personnalisé que vous définissez pour aider à effectuer la tâche. Par exemple, appeler un service pour régler effectivement le thermostat à 19 degrés lorsque l’utilisateur indique de : « Régler mon thermostat à 19 degrés ». 

Pour utiliser le script personnalisé comme action, sélectionnez **Script action** (Action de script) en tant que type d’action **DO** dans l’éditeur de tâche. Ensuite, entrez le nom de la fonction qui implémente l’action. Cliquez sur **Edit** (Modifier) pour faire apparaître l’éditeur de **Scripts** et écrire votre implémentation de cette fonction. 

## <a name="script-action-function-parameter"></a>Paramètre de la fonction d’action de script

La fonction de rappel de l’action spécifiée est toujours appelée avec l’objet [`context`](conversation-designer-context-object.md).

L’objet `context` comprend `taskEntities` et `contextEntities`. Les entités de tâche sont des entités définies ou générées pour la tâche en question, et les entités de contexte sont un conteneur de propriétés dans lesquels il est possible de conserver les entités pendant les conversations avec l’utilisateur.

## <a name="return-value"></a>Valeur de retour
Les fonctions d’**action de script** doivent retourner une valeur booléenne.

## <a name="sample-script-action-function"></a>Exemple de fonction d’action de script
L’exemple de fonction suivant effectue un appel HTTP pour obtenir une réponse avant de retourner un déclencheur mis en correspondance.

```javascript
module.exports.NewTask_do_onRun = function(context) {
    var options =  {
        host: 'HOST',
        path: 'PATH',
        method: 'post',
        headers: {
            "HEADER1" : "VALUE"
        }, 
        body: {
            "BODY": VALUE
        }
    };

    return http.request(options).then(function(response) {
      // parse response payload
      // Send a message to user
      context.responses.push({text: "Done", type: "message"});
    });
} 
```

## <a name="next-step"></a>Étape suivante
> [!div class="nextstepaction"]
> [Action de dialogue](conversation-designer-dialogues.md)
