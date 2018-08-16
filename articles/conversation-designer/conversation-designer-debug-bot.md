---
title: Tester un bot | Microsoft Docs
description: Test et débogage de votre bot Conversation Designer.
author: vkannan
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ROBOTS: NoIndex, NoFollow
ms.openlocfilehash: b08bd96bc7c413ff7cb6db4899c8c0d3ad613ab8
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39300373"
---
# <a name="test-your-conversation-designer-bot"></a>Tester votre bot Conversation Designer
> [!IMPORTANT]
> Conversation Designer n’est pas encore accessible à tous les clients. Des informations supplémentaires sur la disponibilité de Conversation Designer seront communiquées plus tard cette année.

Conversation Designer offre des outils de débogage pratiques par le biais de plusieurs fenêtres de sortie (il sont situés en bas de l’écran). Vous pouvez démarrer le test et le débogage d’un clic sur le bouton **Test** se trouvant dans le coin supérieur droit de la fenêtre. 

## <a name="validation-errors"></a>Erreurs de validation
Il existe quelques circonstances selon lesquelles les erreurs de validation se produisent. Par exemple :  
- il vous manque une réponse pour le déclencheur 
- des informations sont incomplètes sur les flux
- il manque des déclencheurs et du code-behind pour le modèle de réponse conditionnelle, les états de processus et de décision
- une boucle infinie ou récursive est détectée dans les références de modèle 
- votre dialogue a un état orphelin qui n’est pas connecté au flux.
- vous avez ajouté une tâche, mais il manque le déclencheur ou l’action 


## <a name="javascript-output"></a>Sortie JavaScript
Vous pouvez joindre un script à l’exécution d’une réponse pour fournir des fonctionnalités supplémentaires. Si une erreur existe dans le script, elle est affichée ici. Vous pouvez ajouter `console.log()` ou `error.log()` ou `debug.log()` à la logique métier de votre bot, ce qui a pour effet de lister les messages dans la fenêtre de sortie. Par exemple : 

``` javascript
module.exports.Respond_beforeResponse = function(context) {
    console.log(JSON.stringify(context));
}
```

## <a name="runtime-output"></a>Sortie du runtime
Les erreurs ou exceptions générées par le runtime sont affichées ici. Si votre bot ne peut pas répondre par exemple, examinez la sortie pour rechercher ce qui a provoqué l’exception ou l’erreur. S’il y a trop de messages dans la fenêtre de sortie, vous pouvez cliquer sur **Clear all** (Tout effacer), puis tester à nouveau votre bot en lui envoyant un message pour voir les détails de l’erreur. 

## <a name="next-step"></a>Étape suivante
> [!div class="nextstepaction"]
> [Importer et exporter un bot](conversation-designer-export-import-bot.md)
