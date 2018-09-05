---
title: État et stockage | Microsoft Docs
description: Décrit ce que sont le gestionnaire d’état, l’état de la conversation et l’état utilisateur au sein du SDK Bot Builder.
keywords: LUIS, état de la conversation, état utilisateur, stockage, gérer l’état
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 02/15/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: e25ecec3aec1cdebe3b9eae4bff0d3c434cb610b
ms.sourcegitcommit: 1abc32353c20acd103e0383121db21b705e5eec3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/21/2018
ms.locfileid: "42756681"
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


## <a name="types-of-underlying-storage"></a>Types de stockage sous-jacent

Le SDK fournit le middleware (intergiciel) de gestion d’état du bot pour conserver l’état de la conversation et de l’utilisateur. L’état est accessible à l’aide du contexte du bot. Ce gestionnaire d’état peut utiliser Stockage Table Azure, le stockage de fichier ou le stockage en mémoire comme stockage de données sous-jacent. Vous pouvez également créer vos propres composants de stockage pour votre bot.

Les bots créés à l’aide du Stockage Table Azure peuvent être conçus pour être sans état et scalables sur plusieurs nœuds de calcul.

> [!NOTE] 
> Le stockage de fichier et le stockage en mémoire ne sont pas mis à l’échelle sur les nœuds.

## <a name="writing-directly-to-storage"></a>Écriture directe dans le stockage

Vous pouvez également utiliser le SDK Bot Builder pour lire et écrire des données directement dans le stockage, sans utiliser le middleware ou le contexte de bot. Cela peut convenir pour les données que votre bot utilise, qui proviennent d’une source en dehors du flux de la conversation de votre bot.

Par exemple, supposons que votre bot permet à l’utilisateur de demander les prévisions météo et que votre bot les récupère pour une date spécifiée, en les lisant à partir d’une base de données externe. Le contenu de la base de données météo ne dépend pas des informations sur l’utilisateur ou du contexte de la conversation ; vous pouvez donc simplement le lire directement à partir du stockage au lieu d’utiliser le gestionnaire d’état.  Consultez [Écrire directement dans le stockage](bot-builder-howto-v4-storage.md) pour obtenir un exemple.

## <a name="next-steps"></a>Étapes suivantes

À présent, découvrons en détail comment sont traitées les activités et comment y répondre.

> [!div class="nextstepaction"]
> [Traitement des activités](bot-builder-concept-activity-processing.md)

## <a name="additional-resources"></a>Ressources supplémentaires

- [Guide pratique pour enregistrer l’état](bot-builder-howto-v4-state.md)
- [Guide pratique pour écrire directement dans le stockage](bot-builder-howto-v4-storage.md)
