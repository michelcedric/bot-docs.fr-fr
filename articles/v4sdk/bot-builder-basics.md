---
title: Principes de base de Bot Builder | Microsoft Docs
description: Structure de base du SDK Bot Builder.
keywords: contexte de tour, structure de bot, réception des entrées
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 04/18/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 48c32a1dd1c43bbb84c0f771be7dff691f0996ec
ms.sourcegitcommit: f95702d27abbd242c902eeb218d55a72df56ce56
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/19/2018
ms.locfileid: "39300581"
---
# <a name="basic-bot-structure"></a>Structure de bot de base

[!INCLUDE [pre-release-label](~/includes/pre-release-label.md)]

Azure Bot Service et le SDK Bot Builder fournissent des bibliothèques, des exemples et des outils permettant de générer et de déboguer des bots. Toutefois, avant de nous pencher dessus, vous devez comprendre la structure de base d’un bot et la façon dont tous les éléments fonctionnent ensemble. Ces concepts s’appliquent quel que soit le langage de programmation que vous choisissez. Des liens vers du contenu plus détaillé sont fournis tout au long de cet article, qui fournit une base pour comprendre le fonctionnement d’un bot.

Nous allons explorer progressivement la structure de base d’un bot.

## <a name="creation-of-your-bot"></a>Création de votre bot

Vous pouvez créer un bot de différentes façons, par exemple dans le [portail Azure](~/bot-service-quickstart.md), dans [Visual Studio](~/dotnet/bot-builder-dotnet-sdk-quickstart.md), ou par le biais d’outils en ligne de commande pour [JavaScript](~/javascript/bot-builder-javascript-quickstart.md), [Java](~/java/bot-builder-java-quickstart.md) ou [Python](~/python/bot-builder-python-quickstart.md). Une fois créés, les bots peuvent être exécutés sur un ordinateur local, sur Azure ou sur un autre service cloud. Ils fonctionnent tous de manière très similaire, quels que soient l’endroit où ils sont exécutés ou la façon dont ils sont générés.

## <a name="interaction-with-your-bot"></a>Interaction avec votre bot

Un bot, par nature, n’a aucune interface utilisateur visible, contrairement à un site web ou une application ; l’utilisateur interagit avec lui par le biais d’une [conversation](~/v4sdk/bot-concepts.md#activities-and-conversations). En fonction de l’application utilisée pour la connexion au bot (que nous appelons [canal](~/v4sdk/bot-concepts.md), mais que nous n’aborderons pas ici), certaines informations sont échangées entre l’utilisateur et votre bot.

Chaque unité d’information est appelée **activité** au sein de notre bot ; une activité peut se présenter sous différentes formes. Les activités incluent la communication issue de l’utilisateur, appelée **message**, ou les informations supplémentaires rassemblées dans un certain nombre d’autres [types d’activités](~/bot-service-activities-entities.md). Ces informations supplémentaires peuvent indiquer à quel moment un nouveau tiers rejoint ou quitte la conversation, une conversation se termine, etc. Ces types de communication à partir de la connexion de l’utilisateur sont envoyés par le système sous-jacent, sans intervention de l’utilisateur.

Le bot reçoit la communication et la wrappe dans un objet d’activité, avec le type correct, afin de la donner à votre code de bot. Tous les autres types d’activité fournissent des informations utiles ; toutefois, l’activité la plus intéressante et la plus courante est l’activité de **message** à partir de l’utilisateur.

Chaque activité que reçoit le bot commence un tour, que nous décrirons plus en détail un peu plus loin.

## <a name="receiving-user-input"></a>Réception d’une entrée utilisateur

Quand nous recevons une activité de message de l’utilisateur, nous devons comprendre ce qu’il nous envoie. La façon la plus simple consiste à faire correspondre le texte du message entrant à une chaîne. Selon la nature de la chaîne et l’objectif de votre bot, nous pouvons choisir de faire quelque chose. Il peut s’agir de répondre à l’utilisateur, de mettre à jour une variable ou une ressource, de l’enregistrer sur un [stockage](~/v4sdk/bot-builder-storage-concept.md) ou d’effectuer un traitement similaire.

Il existe des façons plus complexes de reconnaître l’entrée utilisateur, telles que l’utilisation de [LUIS](~/v4sdk/bot-builder-concept-luis.md) ou [QnA Maker](~/v4sdk/bot-builder-howto-qna.md), mais la mise en correspondance de chaîne est la plus simple.

## <a name="defining-a-turn"></a>Définition d’un tour

[!INCLUDE [Define a turn](~/includes/snippet-definition-turn.md)]

Le traitement des activités est géré par **l’adaptateur**, comme décrit plus en détail dans [Traitement des activités](~/v4sdk/bot-builder-concept-activity-processing.md). Quand l’adaptateur reçoit une activité, il crée un **contexte de tour** pour fournir des informations sur l’activité et donner du contexte au tour en cours de traitement. Ce contexte de tour existe pendant la durée du tour, puis est éliminé, ce qui marque la fin du tour.

Le [contexte de tour](~/v4sdk/bot-builder-concept-activity-processing.md#turn-context) contient un certain nombre d’informations, et le même objet est utilisé à travers toutes les couches de votre bot. Cela est utile, car cet objet de contexte de tour peut être, et doit être, utilisé pour stocker des informations pouvant s’avérer nécessaires par la suite dans le tour.

> [!IMPORTANT]
> Tous les **tours** sont indépendants les uns des autres, s’exécutent tout seuls et sont susceptibles de se chevaucher. Le bot peut traiter plusieurs tours à la fois, à partir de différents utilisateurs sur différents canaux. Chaque tour a son propre contexte de tour, mais il est intéressant de prendre en compte la complexité introduite dans certaines situations.

## <a name="where-to-go-from-here"></a>Où aller à partir d’ici

Cet article a évité un grand nombre de détails, tels que la façon dont les [activités sont traitées](~/v4sdk/bot-builder-concept-activity-processing.md), les différents [types de conversation](~/v4sdk/bot-builder-conversations.md) ou le suivi de [l’état de votre conversation](~/v4sdk/bot-builder-storage-concept.md) pour des conversations plus intelligentes. Le reste des rubriques conceptuelles s’appuie sur les principes de base et couvre le reste des idées nécessaires à la compréhension des bots et d’Azure Bot Service. Vous pouvez suivre la section Étapes suivantes pour approfondir vos connaissances ou explorer les thèmes qui aiguisent votre curiosité, essayer le [guide de démarrage rapide](~/bot-service-quickstart.md) pour générer votre premier bot ou découvrir comment [développer](~/v4sdk/bot-builder-howto-send-messages.md) des bots.

## <a name="next-steps"></a>Étapes suivantes

Découvrez à présent dans quelle mesure le service Bot Connector permet à votre bot de communiquer avec des utilisateurs sur différentes plateformes.

> [!div class="nextstepaction"]
> [Canaux et le service Bot Connector](~/v4sdk/bot-concepts.md)
