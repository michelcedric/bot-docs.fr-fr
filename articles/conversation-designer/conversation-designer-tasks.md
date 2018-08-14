---
title: Vue d’ensemble des tâches | Microsoft Docs
description: Découvrez les tâches dans le robot Conversation Designer.
author: vkannan
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ROBOTS: NoIndex, NoFollow
ms.openlocfilehash: 685a0296f1bfa5452c28f4ec4d2e4e439f09baca
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39299828"
---
# <a name="tasks-in-conversation-designer-bots"></a>Tâches dans les robots Conversation Designer
> [!IMPORTANT]
> Conversation Designer n’est pas encore accessible à tous les clients. Des informations supplémentaires sur la disponibilité de Conversation Designer seront communiquées plus tard cette année.

Les robots dans Conversation Designer sont composés d’un groupe de tâches connexes. Une **tâche** est une action que votre robot effectue en réponse à une demande ou à une activité spécifiques d’un utilisateur. Par exemple, un robot de café peut inclure les tâches suivantes : « Rechercher des emplacements », « Réserver une table », « Commander de la nourriture » et « Gérer les réservations ». Chaque tâche correspond à un objectif distinct de l’utilisateur. 

## <a name="triggers-and-actions"></a>Déclencheurs et actions
Une tâche effectue une action quand une condition de déclenchement est remplie. Toutes les tâches suivent le modèle suivant : **SI déclenchement**, **FAIRE action**.

Un **déclencheur** peut être de deux types :
1. **Un utilisateur rejoint ou quitte une conversation** (initié par une activité) : une tâche déclenchée par « Un utilisateur rejoint ou quitte une conversation » exécute une action quand un utilisateur entame ou termine une conversation avec le robot. Ce déclencheur est utile pour envoyer un message d’introduction à l’utilisateur. 
2. **Un utilisateur énonce ou saisit quelque chose à l’adresse du robot** (initié par un message) : une tâche déclenchée par « Un utilisateur énonce ou saisit quelque chose à l’adresse du robot » exécute une action en réponse au message de l’utilisateur. Le message de l’utilisateur est interprété par un **module de reconnaissance**.

Quand une condition de déclenchement est remplie, une tâche effectue l’une des actions suivantes :
- **donner une réponse** : une réponse peut être une combinaison de texte affiché, de texte parlé ou de carte riche. Avec cette action, le robot envoie une réponse à l’utilisateur et accomplit la tâche. Une réponse est une conversation à un tour, ce qui signifie que la tâche n’attend pas de messages de suivi de l’utilisateur.
- **démarrer un dialogue** : un dialogue est une conversation à plusieurs tous entre le robot et l’utilisateur. Les dialogues sont particulièrement utiles lorsque le robot doit engager une conversation bidirectionnelle avec un utilisateur afin de collecter les informations nécessaires pour accomplir une tâche.
- **exécuter une fonction de script** : votre robot peut exécuter un script personnalisé que vous écrivez pour appeler un service principal afin d’accomplir une tâche.

## <a name="tips-for-modeling-tasks"></a>Conseils pour la modélisation des tâches

1. Chaque robot doit être conçu pour effectuer plusieurs tâches différentes pour votre client. Vous devriez soigneusement réfléchir à la liste de ces fonctions, et créer une tâche pour chacune d’elles.
2. Pour configurer des déclencheurs, réfléchissez à la manière dont vous pouvez détecter l’intention du client en lien avec l’exécution d’une tâche. Comment le robot va-t-il *savoir* ce que le client veut faire ?
3. Si vous utilisez Language Understanding pour la reconnaissance, veillez à inclure suffisamment d’exemples pour les différentes manières dont un utilisateur peut exprimer son intention d’exécuter une tâche. « Réserver une table » devrait déclencher la même action que « Faire une réservation » ou « Retenir une table ».
4. Envisagez d’ajouter à votre intention Language Understanding des exemples grammaticalement corrects.
5. Si vous prévoyez de publier votre robot en tant que Compétence Cortana, envisagez d’ajouter des exemples de phrases qui fonctionnent bien avec les déclencheurs Cortana. « Demandez à votre robot de faire quelque chose ». 
6. Pour les modules de reconnaissance de code, écrivez des modèles d’expressions régulières (regex) pour déterminer l’intention de l’utilisateur. Les modules de reconnaissance de code peuvent également retourner des entités utilisables dans votre tâche.
7. Si **donner une réponse** est l’*action* de la tâche, elle peut éventuellement inclure une carte adaptative pour le rendu sur les canaux pris en charge.

## <a name="next-step"></a>Étape suivante
> [!div class="nextstepaction"]
> [Module de reconnaissance LUIS](conversation-designer-luis.md)
