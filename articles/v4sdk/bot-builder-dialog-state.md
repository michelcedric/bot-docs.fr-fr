---
title: État de dialogue | Microsoft Docs
description: Décrit le fonctionnement de l’état dans le Kit de développement logiciel (SDK) Bot Builder.
keywords: état, état de bot, état de conversation, état d’utilisateur
author: johnataylor
ms.author: johtaylo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 9/21/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 4da715650eb1c3e7b284aaa47aa5ce4ac7280f4c
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2018
ms.locfileid: "49998756"
---
# <a name="dialog-state"></a>État de dialogue

Les dialogues constituent une approche pour l’implémentation de la logique de conversation à plusieurs tours. En ce sens, il s’agit d’une fonctionnalité du Kit de développement logiciel (SDK) qui repose sur un état persistant. 

Un bot basé sur des dialogues comporte généralement une collection DialogSet en tant que variable de membre dans son implémentation de bot. DialogSet est créé avec un handle vers un objet appelé « accesseur ». 

L’accesseur est le concept central dans le modèle d’état du Kit de développement logiciel (SDK). Un accesseur implémente l’interface IStatePropertyAccessor. Il doit donc fournir des implémentations pour les méthodes Get, Set et Delete. Pour créer un accesseur, vous devez lui indiquer un nom de propriété. 

Le code utilisant l’accesseur peut appeler ses méthodes Get, Set et Delete, sachant que celles-ci se réfèrent à une propriété du même nom. Lorsqu’un composant de niveau supérieur a besoin d’un état persistant, il doit utiliser l’accesseur. De cette façon, différents scénarios peuvent apporter leurs implémentations de stockage d’état, tout en utilisant le même code de niveau supérieur. Par exemple, les dialogues ont recours à l’accesseur, et c’est leur seul moyen d’accéder à l’état persistant.

![état de dialogue](media/bot-builder-dialog-state.png)

Lors de l’appel de OnTurn, le bot initialise le sous-système de dialogue en appelant CreateContext sur DialogSet. La création de DialogContext nécessite l’état. DialogSet utilise donc son accesseur pour obtenir (Get) le JSON d’état de dialogue approprié. Le Kit de développement logiciel (SDK) fournit une implémentation de l’accesseur sous la forme de la classe BotState. Les applications qui prévoient d’utiliser l’implémentation d’état peuvent créer une sous-classe de BotState et hériter d’une implémentation de l’accesseur. Deux sous-classes de BotState sont incluses dans le Kit de développement logiciel (SDK) :

- UserState
- ConversationState

UserState et ConversationState utilisent des clés pour le stockage sous-jacent. La clé est transmise au stockage physique. Logiquement, la clé agit comme un espace de noms pour la propriété nommée par l’accesseur. En interne, l’implémentation BotState utilise l’activité entrante de TurnContext pour générer dynamiquement la clé de stockage dont elle a besoin.

- UserState crée une clé à l’aide de l’ID de canal et de l’ID d’origine. Par exemple, _{Activity.ChannelId}/conversations/{Activity.From.Id}#DialogState_
- ConversationState crée une clé à l’aide de l’ID de canal et de l’ID de conversation. Par exemple, _{Activity.ChannelId}/users/{Activity.Conversation.Id}#YourPropertyName_

Une application doit fournir un accesseur, ainsi que la liaison à la clé de stockage appropriée qui a été créée dynamiquement et le nom de propriété, tout cela en arrière-plan. L’implémentation de l’accesseur par BotState inclut quelques optimisations : 

- La première optimisation est un chargement différé en cache. Un chargement sur l’implémentation de stockage réelle est différé jusqu’à l’appel de la première action Get de l’accesseur, et le résultat de ce chargement est mis en cache. Une référence à ce cache est ajoutée à TurnContext, à l’aide de la clé fournie par la classe BotState correspondante. Ainsi, le cache correspondant à UserState est maintenu dans un champ nommé « UserState », et le cache correspondant à ConversationState est maintenu dans un champ nommé « ConversationState ». Les appels des divers objets d’accesseurs sont transmis à TurnContext, qui peut aller récupérer le cache approprié.

- La deuxième optimisation est que la classe BotState contient une logique pour déterminer si l’état a été modifié. L’opération d’enregistrement sur le stockage sous-jacent n’a lieu que si des modifications ont été apportées.

- La troisième optimisation de l’implémentation BotState concerne une classe nommée BotStateSet. BotStateSet est une collection d’objets BotState qui délègue un appel sur sa méthode SaveChanges à chaque membre de la collection en parallèle.

Il est important que l’appel de SaveChanges sur BotState ne fasse pas partie de l’interface IStatePropertyAccessor. En effet, SaveChanges est une optimisation particulière de l’implémentation BotState plutôt qu’une caractéristique clé du modèle. Plus précisément, le code tel que les dialogues n’a aucun lien avec BotState, et encore moins avec SaveChanges. En fait, le code des dialogues n’est lié qu’à l’accesseur. Le but est d’appeler la méthode SaveChanges hors du système des dialogues après l’exécution de ce dernier. Par exemple, comme illustré dans le schéma, elle peut être appelée dans la méthode OnTurn du bot.

Il est important de noter que l’implémentation BotState implique une sémantique particulière. Elle prend notamment en charge le comportement « la dernière écriture l’emporte », qui permet à la dernière écriture de remplacer l’état écrit précédemment. Cela peut très bien fonctionner pour de nombreuses applications, mais avec des implications, en particulier pour les scénarios scale-out en cas de simultanéité. En cas de problème, implémentez votre propre accesseur et transmettez-le au dialogue. Il existe plusieurs approches alternative : par exemple, une solution peut utiliser la condition eTag courante sur les services de stockages cloud comme le Stockage Azure. Dans ce cas, la solution implémente probablement d’autres parties importantes. Par exemple, elle peut mettre en tampon les activités sortantes et les envoyer uniquement après un enrgistrement réussi. L’important concernant ce comportement, c’est qu’il ne séagit pas de celui de l’implémentation BotState, mais de quelque chose fourni par une application et inséré au niveau de l’accesseur.

L’implémentation BotState en elle-même n’est pas un modèle de stockage pluggable. Elle suit un schéma chargement/enregistrement simple, et le Kit de développement logiciel (SDK) propose trois implémentations pour la production. Une pour le Stockage Azure, et l’autre pour CosmosDB, ainsi qu’une implémentation en mémoire à des fins de test. Notez ici que la sémantique de « la dernière écriture l’emporte » est dictée par l’implémentation BotState.

## <a name="handling-state-in-middleware"></a>Gestion de l’état dans le middleware
Le graphique ci-dessus illustre un appel de SaveAllChanges après une méthode OnTurn du bot. Voici le même graphique, avec un zoom sur l’appel.

![problèmes d’état dans le middleware](media/bot-builder-dialog-state-problem.png)

Le problème avec cette approche est que les états mis à jour à partir d’un middleware personnalisé après le retour de la méthode OnTurn du bot ne sont pas enregistrés dans un stockage durable. La solution consiste à décaler l’appel de SaveAllChanges après l’exécution du middleware personnalisé en ajoutant AutoSaveChangesMiddleware à la fin de la pile de middleware. L’exécution est illustrée ci-dessous.

![solution pour l’état dans le middleware](media/bot-builder-dialog-state-solution.png)

## <a name="additional-resources"></a>Ressources supplémentaires
Pour plus de détails, consultez le Kit de développement logiciel (SDK) Bot Builder sur GitHub [[C#](https://github.com/Microsoft/BotBuilder-dotnet) | [JavaScript](https://github.com/Microsoft/BotBuilder-js)].
