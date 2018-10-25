---
title: Conversations dans le SDK Bot Builder | Microsoft Docs
description: Décrit ce qu’est une conversation dans le SDK Bot Builder.
keywords: flux de la conversation, reconnaître une intention, tour unique, plusieurs tours, conversation de bot
author: jonathanfingold
ms.author: jonathanfingold
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 09/01/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 55145b4728d325bd258cc2cd95f3a265aaa50b0a
ms.sourcegitcommit: b8bd66fa955217cc00b6650f5d591b2b73c3254b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/15/2018
ms.locfileid: "49326456"
---
# <a name="conversation-flow"></a>Flux de la conversation
[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Quand vous concevez le flux de conversation d’un bot, vous devez déterminer la façon dont celui-ci doit répondre à ce que lui dit l’utilisateur. Tout d’abord, un bot reconnaît la tâche ou le sujet de la conversation en fonction d’un message de l’utilisateur. Pour déterminer la tâche ou le sujet (connu sous le nom *d’intention*) associé au message d’un utilisateur, le bot peut rechercher des mots ou des modèles dans le texte du message ou il peut tirer parti de services tels que [Language Understanding](bot-builder-concept-luis.md) et [QnA Maker](https://docs.microsoft.com/en-us/azure/cognitive-services/qnamaker/overview/overview).

Une fois que le bot a reconnu l’intention de l’utilisateur, suivant le scénario, il peut répondre à la demande de l’utilisateur avec une réponse unique, effectuant la conversation en un seul tour, ou nécessiter une série de tours. Pour les flux de conversation à plusieurs tours, le SDK Bot Builder fournit la [gestion d’état](./bot-builder-howto-v4-state.md) pour le suivi d’une conversation, des [invites](bot-builder-prompts.md) pour les demandes d’informations et des [dialogues](bot-builder-dialog-manage-conversation-flow.md) pour l’encapsulation des flux de conversation.

Dans un bot complexe avec plusieurs sous-systèmes, il se peut que vous utilisiez plusieurs services pour reconnaître une intention, à raison d’un par sous-composant du bot. [L’outil Dispatch](bot-builder-tutorial-dispatch.md) obtient les résultats de plusieurs services au même endroit quand vous combinez des sous-systèmes conversationnels dans un même bot.

<!-- 
A conversation identifies a series of activities sent between a bot and a user on a specific channel and represents an interaction between one or more bots and either a _direct_ conversation with a specific user or a _group_ conversation with multiple users.
A bot communicates with a user on a channel by receiving activities from, and sending activities to the user.

- Each user has an ID that is unique per channel.
- Each conversation has an ID that is unique per channel.
- The channel sets the conversation ID when it starts the conversation.
- The bot cannot start a conversation; however, once it has a conversation ID, it can resume that conversation.
- Not all channels support group conversations.
-->

## <a name="single-turn-conversation"></a>Conversation à tour unique

Le flux conversationnel le plus simple est un tour unique. Dans un flux à tour unique, le bot termine sa tâche en un seul tour, composé d’un message de l’utilisateur et d’une réponse du bot.

<!-- The following isn't always true, it's a generalization -->

Le type le plus simple de bot à tour unique n’a pas besoin d’effectuer le suivi de l’état de la conversation. Chaque fois qu’il reçoit un message, il répond uniquement selon le contexte du message entrant actuel, sans avoir connaissance des tours conversationnels passés.

![Bot d’informations météorologiques à tour unique](./media/concept-conversation/weather-single-turn.png)

Un bot d’informations météorologiques peut avoir un flux à tour unique ; il communique simplement à l’utilisateur un bulletin météorologique, sans lui demander la ville ou la date. Toute la logique d’affichage du bulletin météorologique repose sur le message que le bot vient de recevoir. Dans chaque tour d’une conversation, le bot reçoit un [contexte de tour](bot-builder-concept-activity-processing.md#turn-context), qui lui permet de déterminer ce qu’il doit faire ensuite et le flux de la conversation.

## <a name="multiple-turns"></a>Plusieurs tours

Comme il est impossible d’effectuer la plupart des types de conversation en un seul tour, un bot peut également avoir un flux de conversation à plusieurs tours. Certains scénarios qui requièrent plusieurs tours conversationnels sont les suivants :

* Un bot qui invite l’utilisateur à fournir des informations supplémentaires nécessaires à l’accomplissement d’une tâche. Le bot doit savoir s’il dispose de tous les paramètres pour l’exécution de la tâche.
* Un bot qui guide l’utilisateur dans les étapes d’un processus, tel que le passage d’une commande. Le bot doit savoir où en est l’utilisateur dans la séquence d’étapes.

Par exemple, un bot d’informations météorologiques peut avoir un flux à plusieurs tours s’il répond à la question « quelle est la météo ? » en demandant la ville.

![Bot d’informations météorologiques à plusieurs tours](./media/concept-conversation/weather-multi-turn.png)

Quand l’utilisateur répond à l’invite du bot pour la ville en lui indiquant « Seattle », le bot doit disposer d’un contexte enregistré pour comprendre que le message actuel est la réponse à une invite précédente et qu’il fait partie d’une demande d’informations météorologiques. Les bots à plusieurs tours effectuent le suivi de l’état pour répondre correctement aux nouveaux messages.

Consultez [Gérer l’état de la conversation et l’état utilisateur](bot-builder-howto-v4-state.md) pour plus d’informations.

> [!NOTE]
> Les conversations à plusieurs tours avec des clients API REST doivent effectuer le suivi de leur propre état, par exemple dans un stockage de base de données ou de table.

## <a name="conversation-topics"></a>Sujets de la conversation

Vous pouvez concevoir votre bot pour gérer plusieurs types de tâche. Par exemple, vous pouvez avoir un bot qui fournit des flux de conversation différents pour l’accueil de l’utilisateur, le passage d’une commande, l’annulation d’une commande et l’obtention d’aide. Une façon de gérer le passage d’une conversation à l’autre suivant la tâche ou le sujet consiste à reconnaître l’intention (ce que l’utilisateur veut faire) à partir du message actuel.

### <a name="recognize-intent"></a>Reconnaître une intention

Le Kit de développement logiciel (SDK) Bot Builder fournit des _modules de reconnaissance_ qui peuvent traiter un message pour déterminer l’intention, afin que votre bot puisse démarrer le flux conversationnel approprié. Appelez la méthode asynchrone _recognize_ du module de reconnaissance pour déterminer l’intention de l’utilisateur à partir du contenu de son message. Vous pouvez ensuite appeler la méthode _get top scoring intent_ sur le résultat pour obtenir la prédiction principale du module de reconnaissance.

Un module de reconnaissance peut utiliser des expressions régulières, la compréhension langagière ou toute autre logique que vous développez. Voici quelques exemples de modules de reconnaissance possibles :

* Un module de reconnaissance qui utilise QnA Maker pour détecter lorsqu’un utilisateur pose une question traitée dans une base de connaissances.
* Un module de reconnaissance qui utilise Language Understanding (LUIS) pour former un service avec différentes façons dont l’utilisateur peut demander de l’aide et les associe à l’intention `Help`.
* Un module de reconnaissance personnalisé qui utilise des expressions régulières pour rechercher des commandes.
* Un module de reconnaissance personnalisé qui utilise un service pour traduire l’entrée.

### <a name="consider-how-to-interrupt-conversation-flow-or-change-topics"></a>Déterminer comment interrompre le flux de la conversation ou changer de sujet

Une façon d’effectuer le suivi de votre avancement dans une conversation consiste à utiliser [l’état de la conversation](bot-builder-howto-v4-state.md) pour enregistrer les informations sur le sujet actif ou sur les étapes d’une séquence ayant été effectuées.

Dans le cas d’un bot plus complexe, vous pouvez également imaginer une séquence de flux de conversation qui se produisent dans une pile ; par exemple, le bot appelle le flux de nouvelle commande, puis le flux de recherche de produit. Ensuite, l’utilisateur sélectionne un produit et confirme son choix, en suivant le flux de recherche de produit, puis passe la commande.

Toutefois, les conversations suivent rarement un chemin aussi logique et linéaire. Les utilisateurs ne communiquent sous forme de « piles », mais ont tendance à changer fréquemment d’avis. Considérez l'exemple suivant :

![L’utilisateur dit quelque chose d’inattendu](./media/concept-conversation/interruption.png)

Il se peut que votre bot construise logiquement une pile de flux, mais que l’utilisateur décide de faire quelque chose de complètement différent ou de poser une question qui n’est pas liée au sujet actuel. Dans l’exemple, l’utilisateur pose une question, alors que le flux l’invite à répondre par oui ou par non. Comment votre flux doit-il répondre ?

* Insister pour que l’utilisateur réponde d’abord à la question.
* Ignorer toutes les actions précédentes de l’utilisateur, réinitialiser toute la pile de flux et recommencer depuis le début en essayant de répondre à la question de l’utilisateur.
* Tenter de répondre à la question de l’utilisateur, puis revenir à la question oui/non et essayer de reprendre la procédure à ce stade.

Il n’existe aucune réponse correcte à cette question. En effet, la meilleure solution dépend des spécificités de votre scénario et des attentes raisonnables de l’utilisateur quant à la façon dont le bot va répondre. Consultez les articles relatifs à [l’utilisation des dialogues](bot-builder-dialog-manage-conversation-flow.md) et au [traitement des interruptions](bot-builder-howto-handle-user-interrupt.md) pour gérer le flux de conversation.

## <a name="conversation-lifetime"></a>Durée de vie de la conversation

<!-- Note: these activities are dependent on whether the channel actually sends them. Also, we should add links --> Un bot reçoit une activité de _mise à jour d’une conversation_ chaque fois qu’il est ajouté à une conversation, que d’autres membres ont été ajoutés ou supprimés d’une conversation ou que les métadonnées de la conversation ont changé.
Vous pouvez faire en sorte que votre bot réagisse aux activités de mise à jour d’une conversation en saluant l’utilisateur ou en se présentant.

Un bot reçoit une activité de _fin de conversation_ pour indiquer que l’utilisateur a mis fin à la conversation. Un bot peut envoyer une activité de _fin de conversation_ pour indiquer que la conversation va se terminer.
Si vous stockez des informations sur la conversation, vous pouvez les effacer quand celle-ci se termine.

<!--  Types of conversations -->

Votre bot peut prendre en charge les interactions avec plusieurs tours où il invite les utilisateurs à fournir différentes informations. Il peut se concentrer sur une tâche très spécifique ou prendre en charge plusieurs types de tâches.
Le Kit de développement logiciel (SDK) Bot Builder a une prise en charge intégrée pour la compréhension langagière (LUIS) et QnA Maker pour l’ajout de fonctionnalités de « questions et réponses » en langage naturel à votre bot.

## <a name="conversations-channels-and-users"></a>Conversations, canaux et utilisateurs

Une conversation peut être une conversation _directe_ avec un utilisateur spécifique ou une conversation de _groupe_ avec plusieurs utilisateurs.
Un bot communique avec un utilisateur sur un canal en recevant des activités de l’utilisateur ou en lui en envoyant.

* Chaque utilisateur a un ID unique par canal.
* Chaque conversation a un ID unique par canal.
* Le canal définit l’ID de conversation quand il démarre la conversation.
* Le bot ne peut pas démarrer une conversation ; toutefois, une fois qu’il a un ID de conversation, il peut reprendre cette conversation.
* Tous les canaux ne prennent pas en charge les conversations de groupe.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Gérer un flux de conversation simple avec des dialogues](bot-builder-dialog-manage-conversation-flow.md)

<!-- In addition, your bot can send activities back to the user, either _proactively_, in response to internal logic, or _reactively_, in response to an activity from the user or channel.-->
<!--TODO: Link to messaging how tos.-->
