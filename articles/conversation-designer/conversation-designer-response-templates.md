---
title: Modèle de réponse | Microsoft Docs
description: Découvrez comment configurer le modèle de réponse pour les bots Conversation Designer.
author: vkannan
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ROBOTS: NoIndex, NoFollow
ms.openlocfilehash: b09b7fa5f5672c121711deb2edcdc9718a79983f
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39300357"
---
# <a name="response-template-for-conversation-designer-bots"></a>Modèle de réponse pour les bots Conversation Designer
> [!IMPORTANT]
> Conversation Designer n’est pas encore accessible à tous les clients. Des informations supplémentaires sur la disponibilité de Conversation Designer seront communiquées plus tard cette année.

La génération de langage permet à votre bot de communiquer avec l’utilisateur de façon naturelle et riche, au moyen de messages de réponses variables. Ces messages sont gérés par l’intermédiaire de modèles de réponse dans Conversation Designer.

Les modèles de réponse permettent la réutilisation des réponses, et vous aide à conserver un ton et un langage homogènes dans les réponses du bot. 

## <a name="create-response-templates"></a>Créer des modèles de réponse

Dans Conversation Designer, vous créez des modèles de réponse que vous pouvez ensuite réutiliser dans n’importe quelle situation où l’envoi d’une réponse à un utilisateur est voulu. 

Les modèles de réponse simple définissent une collection unique d’énoncés possibles, affichés et oralisés. Vous pouvez ensuite utiliser ces modèles dans les réponses de votre bot, mais aussi dans les états d’invite ou les cartes adaptatives pour reconstruire une chaîne entièrement résolue.

Pour ajouter un modèle de réponse simple, procédez de la façon suivante :
1. Dans le panneau gauche, cliquez sur **Add** (Ajouter). Un menu contextuel s’affiche.
2. Cliquez sur **Simple response** (Réponse simple). Une fenêtre de modification s’affiche dans le panneau de contenu principal.
3. Dans le champ **Simple response name** (Nom de réponse simple), entrez un nom pour cette réponse simple.
4. Dans le champ **Bot's response to user** (Réponse du bot à l’utilisateur), entrez la réponse, à raison d’une expression à la fois.
5. Dans le coin supérieur droit, cliquez sur **Save** (Enregistrer) lorsque vous avez terminé la configuration de la réponse simple. 

Par exemple, vous pouvez créer un modèle d’expression d’accusé de réception, que vous appelez « AckPhrase » (ExpressionAR), avec les éléments suivants :

- OK
- Bien sûr
- Et comment
- Je suis là pour vous aider
- C’est un plaisir de vous aider

À l’aide de ce modèle, le runtime de conversation résout de façon aléatoire sur une de ces chaînes, afin que les réponses du bot semblent plus naturelles, et moins ennuyeuses ou monotones.

## <a name="conditional-response-templates"></a>Modèles de réponse conditionnelle

Les modèles de réponse conditionnelle spécifient plusieurs réponses avec des conditions. La fonction de rappel que vous écrivez permet au moteur de génération de langage de décider de la chaîne de réponse à utiliser en fonction de la condition que vous avez indiquée. 

Par exemple, un modèle sur les heures de la journée doit résoudre en « bonjour » ou « bonsoir » selon l’heure réelle de la journée. 

Pour ajouter un modèle de réponse conditionnelle, suivez ces étapes :
1. Dans le volet gauche, cliquez sur **Add** (Ajouter). Un menu contextuel s’affiche.
2. Cliquez sur **Conditional response** (Réponse conditionnelle). Une fenêtre de modification s’affiche dans le panneau de contenu principal.
3. Dans le champ **Conditional response name** (Nom de réponse conditionnelle), entrez un nom pour ce modèle.
4. Dans le champ **Conditional response** (Réponse conditionnelle), entrez vos expressions, une à la fois. Par exemple, pour « timeOfDayTemplate », entrez « Bonjour » et « Bonsoir », comme deux réponses possibles dans le modèle.
5. Pour la réponse « Bonjour », indiquez « jour » comme **Condition name** (Nom de la condition). De même, pour la réponse « Bonsoir », indiquez « soir » comme **Condition name** (Nom de la condition). Le script personnalisé que vous allez écrire retournera l’une de ces conditions.
6. Dans le champ **Code to execute on run** (Code à exécuter à l’exécution), entrez un nom de fonction de rappel (par exemple : `fnResolveTimeOfDayTemplate`). Ensuite, cliquez sur **View code** (Afficher le code) pour charger l’éditeur de *Scripts**. À partir de là, vous pouvez définir l’implémentation de cette fonction de rappel.
7. Dans le coin supérieur droit, cliquez sur **Save** (Enregistrer) pour enregistrer vos modifications et créer ce modèle.

Exemple de code pour la fonction de rappel *fnResolveTimeOfDayTemplate*. Cette fonction retourne la chaîne qui correspond à la valeur de **Condition name** (Nom de la condition), « jour » ou « soir », spécifiée par la réponse.

```javascript
module.exports.fnTimeOfDayTemplate = function(context) {
    var currentTime = new Date().getHours();
    if(currentTime >= 12) {
        return "evening!";
    } else {
        return "morning!";
    }
}
```

## <a name="send-a-response-to-user"></a>Envoyer une réponse à l’utilisateur

Pour envoyer une réponse à un utilisateur à l’aide des modèles de réponse, entrez le nom de modèle de votre choix entre crochets. Par exemple, pour utiliser le modèle de réponse simple « AckPhrase » dans un état de commentaire, entrez l’expression de **Bot's response to user** (Réponse du bot à l’utilisateur) comme ceci `[AckPhrase], I will get that done for you right away`

Si vous souhaitez utiliser des crochets dans vos réponses, utilisez « \" » comme caractère d’échappement pour indiquer à l’exécution de la conversation qu’il faut ignorer la résolution pour les crochets.

Si des entités ont été définies, vous pouvez également les utiliser dans votre réponse à l’utilisateur. Vous faites référence à vos entités dans la génération de langage, en plaçant le nom de l’entité entre accolades. Par exemple, si votre bot est doté d’une entité `location`, vous pouvez y faire référence dans vos réponses de la façon suivante : `[AckPhrase], I can help you find a table at {location}.`

## <a name="nesting-response-templates"></a>Modèles de réponse imbriqués

Les modèles de réponse peuvent être « imbriqués » ; un modèle de réponse peut avoir des références à un autre modèle de réponse. Par exemple, vous pouvez utiliser le modèle de réponse simple `AckPhrase` et le modèle de réponse conditionnelle `timeOfDayTemplate` pour construire un nouveau modèle de réponse `timeBasedGreeting` qui utilise les deux modèles pour résoudre la réponse finale. 

> [!TIP]
> Bien que l’imbrication des modèles soit une puissante fonctionnalité, prenez soin de vérifier que vos modèles imbriqués ne provoquent pas de boucle infinie. Autrement dit, évitez les situations où `AckPhrase` appelle `timeOfDayTemplate` et `timeOfDayTemplate` rappelle `AckPhrase`.

Par exemple, créez un nom **Simple response** (Réponse Simple) `timeBasedGreeting` et entrez le texte suivant comme **Bot's response to user** (Réponse du bot à l’utilisateur) possible pour ce modèle : `[timeOfDayTemplate] [AckPhrase], ... `

## <a name="define-user-utterance-as-help-tips"></a>Définir l’énoncé de l’utilisateur comme info-bulle d’aide

Lors de la définition de **Utterance** (Énoncé) utilisateur, vous pouvez également définir un énoncé en tant que **Help Tip** (Info-bulle d’aide). Pour définir un énoncé en tant qu’info-bulle **d’aide**, cliquez sur `...` à droite de l’énoncé, et sélectionnez **Use as help tip** (Utiliser comme info-bulle d’aide). 

L’info-bulle d’aide est utilisée dans le modèle de génération de langage intégré `[builtin-tasktips]`, partout où vous définissez un champ **Bot's response to user** (Réponse du bot à l’utilisateur). Par exemple, vous pouvez composer une réponse indiquant que : `Sorry I did not understand that. Here are some things you can try - [builtin.tasktips]`

S’il n’existe aucune définition de **Help Tip** (Info-bulle d’aide) dans une tâche, `[builtin-tasktips]` est résolu en chaîne vide. Si une tâche comprend plusieurs définitions de **Help Tip** (Info-bulle d’aide), `[builtin-tasktips]` sélectionne l’une d’elles au hasard chaque fois qu’une réponse est donnée.

## <a name="next-step"></a>Étape suivante
> [!div class="nextstepaction"]
> [Cartes adaptatives](conversation-designer-adaptive-cards.md)
