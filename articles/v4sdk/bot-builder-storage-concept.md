---
redirect_url: /bot-framework/bot-builder-howto-v4-state
ms.openlocfilehash: e5da105e32ae748383d376f90afd9aebbf4c7aa5
ms.sourcegitcommit: 3bf3dbb1a440b3d83e58499c6a2ac116fe04b2f6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/23/2018
ms.locfileid: "46708115"
---
<a name="--"></a><!--
---
titre : État et stockage | Microsoft Docs description : Description du gestionnaire d’état, de l’état de la conversation et de l’état utilisateur au sein du SDK Bot Builder.
mots clés : LUIS, état de la conversation, état utilisateur, stockage, gérer l’état auteur : DeniseMak ms.author : v-demak responsable : kamrani ms.topic : article ms.prod : infrastructure bot ms.date : 15/02/2018 monikerRange : 'azure-bot-service-4.0'
---

# <a name="state-and-storage"></a>État et stockage
[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Une bonne conception de bot passe notamment par le suivi du contexte d’une conversation, afin que le bot se souvienne de choses telles que les réponses aux questions précédentes.
Selon ce pour quoi votre bot est utilisé, vous pouvez même être amené à effectuer le suivi de l’état ou à stocker des informations plus longtemps que la durée de vie de la conversation.
*L’état* d’un bot représente des informations dont il se souvient pour répondre correctement aux messages entrants. Le SDK Bot Builder fournit des classes pour stocker et récupérer des données d’état sous la forme d’un objet associé à un utilisateur ou une conversation.

* Les **propriétés de la conversation** permettent au bot d’effectuer le suivi de la conversation en cours entre lui-même et l’utilisateur. Si votre bot doit exécuter une séquence d’étapes ou basculer entre des sujets de conversation, vous pouvez utiliser les propriétés de la conversation pour gérer les étapes d’une séquence ou effectuer le suivi du sujet actuel. Étant donné que les propriétés de la conversation reflètent l’état de la conversation actuelle, vous les effacez généralement à la fin d’une conversation, quand le bot reçoit une activité _fin de conversation_.
* Les **propriétés utilisateur** peuvent servir à de nombreuses fins, par exemple pour déterminer l’endroit où une conversation précédente de l’utilisateur s’était arrêtée ou simplement pour accueillir un utilisateur régulier par son nom. Si vous stockez les préférences d’un utilisateur, vous pouvez utiliser ces informations pour personnaliser la prochaine conversation. Par exemple, vous pourriez avertir l’utilisateur de la publication d’un article sur un sujet qui l’intéresse ou bien l’informer de la disponibilité d’une date de rendez-vous. Vous devez les effacer si le bot reçoit une activité _supprimer les données utilisateur_.

Vous pouvez utiliser [Stockage](bot-builder-howto-v4-storage.md) pour lire et écrire dans un stockage persistant. Cela permet à votre bot d’effectuer des opérations telles que la mise à jour des ressources partagées, l’enregistrement des RSVP ou des votes, ou la lecture des données météorologiques historiques. De la même façon qu’une application utilise le stockage pour atteindre ses objectifs, votre bot peut faire de même au sein de la conversation avec votre utilisateur.

<!-- 
*Conversation state* pertains to the current conversation that the user is having with your bot. When the conversation ends, your bot deletes this data.

You can also store *user state* that persists after a conversation ends. For example, if you store a user's preferences, you can use that information to customize the conversation the next time you chat. For example, you might alert the user to a news article about a topic that interests her, or alert a user when an appointment becomes available. 
-->

<!-- You should generally avoid saving state using a global variable or function closures.
Doing so will create issues when you want to scale out your bot. Instead, use the conversation state and user state middleware that the BotBuilder SDK provides --> 

<!--
## Types of underlying storage

The SDK provides bot state manager middleware to persist conversation and user state. State can be accessed using the bot's context. This state manager can use Azure Table Storage, file storage, or memory storage as the underlying data storage. You can also create your own storage components for your bot.

Bots built using Azure Table Storage can be designed to be stateless and scalable across multiple compute nodes.

> [!NOTE] 
> File and memory storage won't scale across nodes.

## Writing directly to storage

You can also use the Bot Builder SDK to read and write data directly to storage, without using middleware or without using the bot context. This can be appropriate to data that your bot uses, that comes from a source outside your bot's conversation flow.

For example, let's say your bot allows the user to ask for the weather report, and your bot retrieves the weather report for a specified date, by reading it from an external database. The content of the weather database isn't dependent on user information or the conversation context, so you could just read it directly from storage instead of using the state manager.  See [How to write directly to storage](bot-builder-howto-v4-storage.md) for an example.

## Next steps

Next, lets get into how activities are processed, in depth, and how we respond to them.

> [!div class="nextstepaction"]
> [Activity Processing](bot-builder-concept-activity-processing.md)

## Additional resources

- [How to save state](bot-builder-howto-v4-state.md)
- [How to write directly to storage](bot-builder-howto-v4-storage.md)

-->
