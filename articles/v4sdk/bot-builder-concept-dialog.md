---
title: Dialogues dans le SDK Bot Builder | Microsoft Docs
description: Découvrez ce qu’est un dialogue et comment il fonctionne dans le SDK Bot Builder.
keywords: flux de conversation, reconnaître l’intention, tour unique, plusieurs tours, conversation de bot, dialogues, invites, cascades, ensemble de dialogues
author: johnataylor
ms.author: johtaylo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 9/22/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 97f280d1698e8670be81572a2891c18bc7bf47ab
ms.sourcegitcommit: a496714fb72550a743d738702f4f79e254c69d06
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/01/2018
ms.locfileid: "50736667"
---
# <a name="dialogs-library"></a>Bibliothèque des dialogues

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

La gestion des conversations grâce à une boîte de dialogue est une fonction centrale du kit de développement logiciel (SDK). Les objets de dialogue traitent les activités entrantes et génèrent des réponses sortantes. La logique métier du bot s’exécute directement ou indirectement dans les classes de dialogue.

Lors de l’exécution, les instances de dialogue sont organisées dans une pile. Le dialogue en haut de la pile est appelé ActiveDialog (dialogue actif). Le dialogue actuellement actif traite l’activité entrante. Entre chaque tour de la conversation (qui n’est pas limitée dans le temps et qui peut s’étendre sur plusieurs jours), la pile est conservée. 

Un dialogue implémente trois fonctions principales :
- BeginDialog
- ContinueDialog
- ResumeDialog

Lors de l’exécution, les dialogues et les classes DialogContext déterminent ensemble le dialogue approprié pour gérer l’activité. La classe DialogContext lie la pile de dialogues persistante, l’activité entrante et la classe DialogSet. DialogSet contient des dialogues que le bot peut appeler.

L’interface de DialogContext reflète la notion sous-jacente de début et de poursuite du dialogue. Le modèle général pour l’application est de toujours appeler ContinueDialog en premier. S’il n’existe aucune pile et donc aucun ActiveDialog, l’application doit commencer le dialogue qu’elle choisit en appelant BeginDialog sur DialogContext. L’entrée de dialogue correspondante dans DialogSet sera poussée sur la pile (techniquement, il s’agit de l’identifiant du dialogue qui est ajouté à la pile). Ensuite, elle délègue un appel à BeginDialog sur l’objet spécifique de dialogue. Si un ActiveDialog avait existé, il aurait simplement délégué l’appel au ContinueDialog du dialogue et aurait donné à ce dernier les propriétés persistantes associées.

Sachez que **BeginDialog du dialogue** correspond au code d’initialisation qui utilise les propriétés d’initialisation (appelées « options » dans le code). Par ailleurs, **ContinueDialog du dialogue** correspond au code qui est appliqué pour maintenir l’exécution lors de l’arrivée d’une activité après la persistance. Par exemple, imaginons un dialogue qui pose une question à l’utilisateur. La question est posée dans BeginDialog et la réponse est attendue dans ContinueDialog.

Pour prendre en charge l’imbrication des dialogues (quand un dialogue a des dialogues enfant), il existe un autre type de continuation qui s’appelle la reprise. DialogContext appelle la méthode ResumeDialog sur un dialogue parent lorsqu’un dialogue enfant est terminé.

Les invites et les cascades sont deux exemples concrets de dialogues fournis par le SDK. De nombreux scénarios sont créés en composant à l’aide de ces abstractions. Toutefois, en arrière-plan, la logique exécutée se base toujours sur le même début, c’est-à-dire sur le modèle de continuation et de reprise décrit ici. L’implémentation d’une classe de dialogue à partir de zéro est une opération assez avancée. Néanmoins, un exemple est inclus dans les [exemples](https://github.com/Microsoft/BotBuilder-samples).

La bibliothèque des **dialogues** du SDK Bot Builder contient des fonctionnalités intégrées comme les _invites_, les _dialogues en cascade_ et les _dialogues composants_ afin de vous aider à gérer la conversation de votre bot. Vous pouvez utiliser les invites pour demander aux utilisateurs différents types d’informations, une cascade pour combiner plusieurs étapes en une séquence, et les dialogues composants pour empaqueter votre logique de dialogue dans des classes séparées qui peuvent ensuite être intégrées dans d’autres bots.
## <a name="waterfall-dialogs-and-prompts"></a>Dialogues en cascade et invites

La bibliothèque des **dialogues** est équipée d’un ensemble de types d’invites que vous pouvez utiliser pour collecter différents types d’entrées utilisateur. Par exemple, pour demander à un utilisateur une entrée de texte, vous pouvez utiliser **TextPrompt** ; pour demander à un utilisateur un numéro, vous pouvez utiliser **NumberPrompt** ; pour demander une date et une heure, vous pouvez utiliser **DateTimePrompt**. Les invites sont un type de dialogue particulier. Pour utiliser l’invite d’un dialogue en cascade, ajoutez la cascade et l’invite au même ensemble de dialogue. 

En raison de la nature de l’interaction entre l’invite et la réponse, l’implémentation d’une invite requiert au moins deux étapes dans un dialogue en cascade : la première pour envoyer l’invite, et la deuxième pour capturer et traiter la réponse.  Si vous disposez d’une invite supplémentaire, vous pouvez parfois les combiner à l’aide d’une fonction unique. Cela vous permet, dans un premier temps, de traiter la réponse de l’utilisateur et, dans un deuxième temps, de démarrer la prochaine invite.

Le terme « `WaterfallDialog` » se réfère à une implémentation spécifique d’un dialogue, utilisée pour collecter des informations de l’utilisateur ou guider celui-ci dans une série de tâches. Les tâches sont implémentées en tant que suite de fonctions, où le résultat de la première fonction est passé en tant qu’argument à la fonction suivante, et ainsi de suite. Chaque fonction représente généralement une seule étape dans le processus global. À chaque étape, le bot invite l’utilisateur à effectuer une entrée, attend une réponse, puis passe les résultats à l’étape suivante. 

Les invites et les cascades sont tous deux des types de dialogues, comme le montre la hiérarchie de classes ci-dessous. 

![classes de dialogues](media/bot-builder-dialog-classes.png)

Un dialogue en cascade est composé d’une séquence d’étapes en cascade. Chaque étape est un délégué asynchrone correspondant à un paramètre de _contexte d’étape en cascade_ (`step`). Selon le modèle, la dernière opération d’une étape en cascade est soit de commencer un dialogue enfant (en général, une invite) soit de terminer la cascade en elle-même. Le diagramme suivant représente une séquence d’étapes en cascade et les opérations de pile qui se produisent.

![Concept de dialogue](media/bot-builder-dialog-concept.png)

Une valeur renvoyée par un dialogue peut-être gérée au sein d’une étape en cascade dans un dialogue, ou depuis le gestionnaire de tour de votre bot.
Dans une étape en cascade, le dialogue fournit la valeur renvoyée dans la propriété _résultat_ du contexte de l’étape en cascade.
En général, vous n’avez qu’à vérifier l’état du résultat du tour de dialogue dans la logique de tour de votre bot.

## <a name="repeating-a-dialog"></a>Répéter un dialogue

Pour répéter un dialogue, utilisez la méthode de *remplacement de dialogue*. La méthode de *remplacement de dialogue* du contexte de dialogue fait sortir le dialogue actuel de la pile, puis envoie le dialogue de remplacement en haut de la pile et démarre ce dialogue. Vous pouvez utiliser cette méthode pour créer une boucle en remplaçant un dialogue par lui-même. Notez que si vous avez besoin de conserver l’état interne du dialogue actuel, vous devez transmettre les informations à la nouvelle instance du dialogue lors de l’appel à la méthode de _remplacement de dialogue_. Ensuite, vous devez initialiser le dialogue de manière appropriée. Les options transférées dans le nouveau dialogue sont accessibles via la propriété _options_ du contexte de n’importe quelle étape du dialogue. Il s’agit là d’un excellent moyen de traiter les flux de conversation complexes et de gérer les menus.

## <a name="branch-a-conversation"></a>Créer une branche de conversation

Le contexte du dialogue maintient une _pile de dialogue_, et suit l’étape suivante de chaque dialogue sur la pile. Sa méthode de _démarrage de dialogue_ envoie un dialogue par push sur le dessus de la pile, et sa méthode _fin de dialogue_ fait sortir le dialogue du dessus de la pile.

Un dialogue peut démarrer un nouveau dialogue au sein du même ensemble en appelant la méthode de _démarrage de dialogue_ du contexte et en fournissant l’ID du nouveau dialogue. Le nouveau dialogue devient alors le dialogue actuellement actif. Le dialogue original est toujours sur la pile, mais les appels de la méthode de _poursuite du dialogue_ du contexte de dialogue ne sont envoyés qu’au dialogue du dessus de la pile, le _dialogue actif_. Lorsqu’un dialogue est sorti de la pile, le contexte du dialogue passe à la prochaine étape sur la cascade sur la pile, là où il a laissé le dialogue original.

Ainsi, vous pouvez créer une branche au sein de votre flux de conversation en incluant une étape dans un dialogue. Cette étape permet de choisir sous condition un dialogue à démarrer parmi l’ensemble des dialogues disponibles.

## <a name="component-dialog"></a>Dialogue composant
Il se peut que vous souhaitiez écrire un dialogue réutilisable dans différents scénarios. Par exemple, un dialogue relatif à l’adresse qui demande à l’utilisateur des informations telles que sa rue, sa ville et son code postal. 

Le type ComponentDialog (dialogue composant) fournit un certain niveau d’isolation, car il possède un DialogSet (ensemble de dialogues) séparé. Grâce à cet ensemble séparé, il évite les collisions de nom avec le parent contenant le dialogue. Par ailleurs, il crée son propre runtime de dialogue indépendant et interne (en créant son propre DialogContext), et il répartit l’activité sur ce runtime. Cette répartition secondaire signifie qu’il a eu l’opportunité d’intercepter l’activité. Cela peut être très utile si vous souhaitez implémenter des fonctionnalités telles que « aide » et « annuler ».  Consultez l’exemple de modèle de bot d’entreprise. 

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Utiliser une bibliothèque de dialogues pour collecter des entrées utilisateur](bot-builder-prompts.md)
