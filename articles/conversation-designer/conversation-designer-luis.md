---
title: Définir un module de reconnaissance LUIS comme déclencheur de tâche | Microsoft Docs
description: Apprenez comment configurer la compréhension de langage en tant que déclencheur de tâche à l’aide de LUIS.ai
author: vkannan
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ROBOTS: NoIndex, NoFollow
ms.openlocfilehash: 39fe222fb1d54346b33617c425b1fdf2d56daa0d
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39300352"
---
# <a name="define-a-luis-recognizer-as-task-trigger"></a>Définir un module de reconnaissance LUIS comme déclencheur de tâche
> [!IMPORTANT]
> Conversation Designer n’est pas encore accessible à tous les clients. Des informations supplémentaires sur la disponibilité de Conversation Designer seront communiquées plus tard cette année.

La plupart du temps, l’utilisateur exprime son intention d’effectuer une tâche en langage naturel. Avec Conversation Designer, vous pouvez facilement configurer un modèle de compréhension du langage naturel pour votre bot alimenté par <a href="https://luis.ai" target="_blank">LUIS.ai</a>.

Pour ce faire, sélectionnez le type de déclencheur **L’utilisateur dit ou saisit quelque chose**. Vous aurez l’option de spécifier le nom de l’**intention**. 

Vous pouvez rechercher des intentions existantes ou en créer dans le champ **Quelle intention de langage ?**.

## <a name="create-a-new-intent"></a>Créer une intention

Pour créer une intention, tapez le nom de l’intention, puis cliquez sur **Créer une intention**. Entrez des exemples d’énoncés des choses que vos utilisateurs pourraient dire et qui doivent déclencher cette tâche spécifique.

Par exemple, un bot de café doit être en mesure d’effectuer la tâche de recherche de cafés à proximité de l’utilisateur. Pour gérer ce scénario, sélectionnez **L’utilisateur dit ou saisit quelque chose** et définissez le **Nom de l’intention** sur « LocationNearMe ». Ensuite, fournissez des exemples d’énoncés pour cette intention. Par exemple :  
- *rechercher des emplacements à proximité*
- *rechercher des cafés à proximité*
- *le magasin Redmond est-il encore ouvert ?*
- *y-a-t-il un magasin à Seattle ?*
- *quels cafés sont encore ouverts ?*
- *où puis-je trouver quelque chose à manger ?*
- *J’aimerais avoir quelque chose à manger*
- *J'ai faim*

Entrez le plus d’énoncés possible dont vous pensez qu’ils peuvent aider à exprimer l’intention de l’utilisateur de déclencher cette tâche spécifique.

## <a name="default-intents-provisioned-for-your-bot"></a>Intentions par défaut approvisionnées pour votre bot

Par défaut, votre bot est approvisionné avec 4 intentions. 
- **Aucun** : c’est l’intention de secours (valeur par défaut) pour votre bot. Utilisez cette intention pour aider à saisir des choses auxquelles votre bot ne sait pas encore répondre.
- **Aide** : configurer avec des exemples d’énoncés qui permettent de déterminer si utilisateur a besoin d’aide. *À l’aide, j’ai besoin d’aide, que puis-je dire ?, je suis bloqué*, etc.
- **Message d’accueil** : configurer avec des exemples d’énoncés qui permettent de faire correspondre une intention de message d’accueil comme *Hi, Salut, Bonjour, Comment vas-tu bot*, etc.
- **Annuler** : configurer avec des exemples d’énoncés exprimant une intention d’annulation. *Stop, Annule ça, ne le fais-pas, rétablis*, etc.

## <a name="create-and-label-entities"></a>Créer et étiqueter des entités

En dehors de vous aider à déterminer l’intention de l’utilisateur, la compréhension du langage peut également vous aider à déterminer des entités spécifiques intéressantes pour la tâche. Par exemple, lorsque l’utilisateur dit *trouve des cafés près de Redmond*, vous pouvez souhaiter extraire *Redmond* en tant que valeur possible pour l’entité ***emplacement***. 

Pour configurer des entités pour la tâche, dans la chaîne **Énoncé**, sélectionnez la partie de l’énoncé qui doit être un exemple de valeur d’entité. Assignez-la à une entité existante ou créez-en une. Pour créer une entité, tapez le nom de l’entité dans le champ **Rechercher ou créer** et cliquez sur **Créer une entité**. 

# <a name="supported-entity-types"></a>Types d’entités prises en charge

La compréhension du langage vous permet de créer différents types d’entités. Lorsque vous créez une entité, vous devez spécifier un `type`. 

Les types disponibles sont :

- **Simple** : il s’agit de la valeur *par défaut*.
- **Liste** : utilisez ce type si votre entité a un ensemble fini de valeurs possibles. Exemple : *Couleur*, *Ville*.
- **Hiérarchique** : utilisez ce type pour créer des entités avec une relation parent-enfant. Exemple : *fromCity* et *toCity* ont toutes deux l’entité *Ville* en tant que parent.
- **Composite** : utilisez ce type pour créer des groupes de valeurs qui composent une unité explicite. Exemple : *Ville* et *État* constituent l’entité *Emplacement*.

<!-- # pre-built entity types TBD -->

# <a name="entities-in-use"></a>Entités utilisées

Quand vous créez et ajoutez des entités à la section de compréhension du langage, le tableau **Entités en cours d’utilisation** est mis à jour avec la liste des entités utilisées par cette tâche spécifique. Vous pouvez ajouter manuellement à la liste d’autres entités utilisées dans cette tâche. 

Options disponibles :

- **Code** : il s’agit d’une entité créée dans votre script personnalisé. Vous pouvez la spécifier ici pour qu’elle vous aide à créer des fonctionnalités telles qu’intellisense.

<!-- # Use as help tip TBD  -->

## <a name="next-step"></a>Étape suivante
> [!div class="nextstepaction"]
> [Module de reconnaissance de code](conversation-designer-code-recognizer.md)
