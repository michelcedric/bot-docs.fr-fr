---
title: Référence API pour l’objet de contexte | Microsoft Docs
description: Découvrez comment référencer l’objet de contexte dans votre robot Conversation Designer.
author: vkannan
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ROBOTS: NoIndex, NoFollow
ms.openlocfilehash: 5ba70189c16815539524d3c9046da03ed0344b9b
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39298794"
---
# <a name="api-reference"></a>Référence API
> [!IMPORTANT]
> Conversation Designer n’est pas encore accessible à tous les clients. Des informations supplémentaires sur la disponibilité de Conversation Designer seront communiquées plus tard cette année.

Conversation Designer vous permet d’ajouter une logique métier personnalisée à votre robot. Ces fonctions de script sont implémentées dans l’éditeur de **Scripts**. Les fonctions sont connectées à différents endroits comme les modèles de réponse conditionnelle, les *déclencheurs* ou les *actions* de la tâche, ou encore les états de dialogue. Elles reçoivent toutes un `context`objet de type **[IConversationContext]** dans leurs paramètres de fonction.

L’exemple de code suivant montre la signature d’une fonction de réponse conditionnelle lorsque l’objet de **contexte** est passé.

```javascript
/**
* @param {IConversationContext} context
*/
module.exports.NewConditionalResponse_onRun = function(context) {
    // Business logic here
    return true; // Returns a boolean
}
```

L’objet `context` vous permet d’accéder aux informations concernant la conversation entre l’utilisateur et le robot.

## <a name="script-callback-functions"></a>Fonctions de rappel de script

Vous pouvez créer des fonctions personnalisées de rappel de script sous différentes formes. Bien que vous puissiez leur attribuer des noms différents, elles prennent l’une des formes suivantes sur le plan fonctionnel.

| Forme | Paramètre | Type de retour | Description |
| ---- | ---- | ---- | ---- |
| Fonction d’avant réponse | context | **void** ou **promise** | Fonction exécutée avant qu’une réponse ne soit fournie. |
| Fonction de processus | context | **void** ou **promise** | Fonction exécutant la logique métier. |
| Fonction de décision | context | **string** ou **promise** | Fonction prenant des décisions basées sur la logique métier. La chaîne de retour doit correspondre à une condition d’un bloc de [décision](conversation-designer-dialogues.md#decision-state). |
| Fonction de reconnaissance de code | context | **boolean** ou **promise** | Logique métier personnalisée qui s’exécute en cas de **déclenchement de script**. Retournez `true` pour indiquer une correspondance. Sinon, retournez `false` pour annuler la correspondance. |
| Fonction onRecognize | context | **boolean** ou **promise** | Fonction qui ne s’exécute en cas de correspondance avec un module de reconnaissance LUIS. Cette fonction de rappel permet de traiter les entités LUIS et de retourner une valeur **booléenne** appropriée. Retournez `true` pour indiquer une correspondance. Sinon, retournez `false` pour annuler la correspondance. |

## <a name="iconversationcontext-interface"></a>Interface IConversationContext

L’interface `IConversationContext` assure un suivi des informations de conversation entre l’utilisateur et le robot. Toutes les fonctions personnalisées connectées via Conversation Designer reçoivent un objet `context` comme argument de paramètre.

## <a name="context-properties"></a>Propriétés de contexte
L’objet `context` présente les propriétés suivantes.

| NOM |  Code | Description |
| ---- | ---- | ---- |
| `request` | `context.request` | Récupérer l’objet de requête contenant l’activité du robot.  |
| | `context.request.attachment` | Activité de pièce jointe pouvant contenir une carte adaptable. |
| | `context.request.text` | Activité de texte contenant le message texte entrant du client. |
| | `context.request.speak` | Activité de parole contenant le texte parlé du client (si disponible). |
| | `context.request.type` | Indique le type d’activité (par défaut : « message »). |
| `responses` | `context.responses` | Conserve un tableau des activités devant être renvoyées au client à la fin de l’état actuel ou du code sous-jacent à l’exécution. |
| | `context.responses.push` | Ajouter une activité à la réponse. |
| `global` | `context.global` | Objet JavaScript contenant les données conversationnelles que vous avez définies. Cet objet est conservé tout au long de la conversation. |
| `local` | `context.local` | Objet JavaScript contenant les données de tâche que vous avez définies. Cet objet est conservé pendant toute la durée d’une tâche donnée. LUIS essaye toujours de revenir au contexte local. Si vous souhaitez conserver les résultats de LUIS, pensez à les copier dans le contexte `context.global`. |
| | `context.local['@description']` | Renvoie les entités brutes reçues de LUIS. |
| `sticky` | `context.sticky` | Indique le nom de la tâche en cours |
| `currentTemplate` | `context.currentTemplate` | [Modèle de réponse conditionnelle](conversation-designer-response-templates.md#conditional-response-templates) appelé pour les évaluations d’affichage et de parole. Cet objet contient trois propriétés : <br/>1. **name** : nom du modèle actuel. <br/>2. **modalityDisplay**: valeur booléenne indiquant le mode d’association d’une évaluation d’affichage. <br/>3. **modalitySpeak**: valeur booléenne indiquant le mode d’association d’une évaluation de parole. |

## <a name="context-methods"></a>Méthodes contextuelles
L’objet `context` présente les méthodes suivantes.

| NOM | Type de retour | Code | Description |
| ---- | ---- | ---- | ---- |
| `getCurrentTurn` | **number** | `context.getCurrentTurn();` | Obtenir le tour à partir du cadre en haut de la pile en cas d’exécution d’une nouvelle invite. |

## <a name="next-step"></a>Étape suivante
> [!div class="nextstepaction"]
> [Créer un robot](conversation-designer-create-bot.md)
