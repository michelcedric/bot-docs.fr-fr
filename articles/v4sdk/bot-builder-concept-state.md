---
title: Gestion d’un état | Microsoft Docs
description: Décrit le fonctionnement de l’état dans le Kit de développement logiciel (SDK) Bot Builder.
keywords: état, état de bot, état de conversation, état d’utilisateur
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 11/15/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 940dba389205ff339b80f741b8a8aec87ff54f1d
ms.sourcegitcommit: bcde20bd4ab830d749cb835c2edb35659324d926
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/27/2018
ms.locfileid: "52338562"
---
# <a name="managing-state"></a>Gestion de l’état

Un état au sein d’un bot suit les mêmes paradigmes que les applications web modernes et le Kit de développement logiciel de Bot Framework offre certaines abstractions pour faciliter la gestion d’un état.

Comme avec les applications web, un bot est fondamentalement sans état ; une autre instance de votre bot peut gérer n’importe quel tour de la conversation. Pour certains bots, cette simplicité est préférée : le robot peut fonctionner sans informations supplémentaires ou bien la présence des informations requises dans le message entrant est garantie. Pour d’autres, l’état (tel que là où nous en sommes dans la conversation ou bien des données relatives à l’utilisateur reçues précédemment) est nécessaire pour que le bot ait une conversation utile.

**Pourquoi ai-je besoin d’un état ?**

Le maintien d’un état permet à votre bot d’avoir des conversations plus pertinentes en mémorisant certaines choses concernant un utilisateur ou une conversation. Par exemple, si vous avez déjà parlé à un utilisateur, vous pouvez enregistrer les informations les concernant, pour ne pas être obligé de les demander à nouveau. L’état conserve également les données plus longtemps que le tour en cours, afin que votre bot puisse conserver des informations durant une conversation à plusieurs tours.

En ce qui concerne les bots, il existe quelques couches concernant l’utilisation de l’état que nous aborderons ici : la couche de stockage, la gestion d’état et les accesseurs de propriété d’état.

## <a name="storage-layer"></a>Couche de stockage

Notre *couche de stockage* commence sur le serveur principal; où les informations d’état sont réellement stockées. Cela peut être considéré comme notre stockage physique, comme un serveur en mémoire, Azure, ou tiers.

Le SDK Bot Framework inclut des implémentations pour la couche de stockage :

- **Stockage en mémoire** implémente le stockage en mémoire à des fins de test. Le stockage de données en mémoire est uniquement destiné aux tests locaux, car il est volatile et temporaire. Les données sont effacées chaque fois que le bot est redémarré.
- **Stockage Blob Azure** se connecte à une base de données d’objets Stockage Blob Azure.
- **Stockage Azure Cosmos DB** se connecte à une base de données NoSQL Cosmos DB.

Pour obtenir des instructions sur la façon de se connecter à d’autres options de stockage, consultez [Écrire directement dans le stockage](bot-builder-howto-v4-storage.md).

## <a name="state-management"></a>Gestion de l'état

La *gestion de l’état* automatise la lecture et l’écriture de l’état de votre bot sur la couche de stockage sous-jacente. L’état est stocké comme *propriétés d’état*, qui sont des paires clé-valeur que votre bot peut lire et écrire via l’objet de gestion d’état sans se préoccuper de l’implémentation sous-jacente spécifique. Ces propriétés d’état définissent la façon dont ces informations sont stockées. Par exemple, lorsque vous récupérez une propriété que vous avez définie comme une classe ou un objet spécifique, vous savez comment ces données sont structurées.

Ces propriétés d’état sont localisées dans des « compartiments » délimités, qui sont simplement des collections pour aider à organiser ces propriétés. Le Kit de développement logiciel inclut trois de ces « compartiments » :

- État utilisateur
- État des conversations
- État des conversations privées

Ces compartiments sont tous des sous-classes de la classe *bot state*, et peuvent être exploités pour définir d’autres types de compartiments avec des portées différentes.

Ces compartiments prédéfinis sont limités à une certaine visibilité, en fonction du compartiment :

- L’état utilisateur est disponible à chaque tour pendant lequel le bot est en conversation avec l’utilisateur sur ce canal, quelle que soit la conversation
- L’état des conversations est disponible à chaque tour d’une conversation spécifique, quel que soit l’utilisateur (par exemple, les conversations de groupe)
- L’état des conversations privées est limité à la fois la conversation spécifique et à un utilisateur spécifique

> [!TIP]
> L’état d’utilisateur et l’état de conversation sont limités par canal.
> Une même personne utilisant différents canaux pour accéder à votre bot entraîne l’apparition de plusieurs utilisateurs, un par canal, chacun ayant son propre état d’utilisateur.

Les clés utilisées pour chacun de ces compartiments prédéfinis sont spécifiques à l’utilisateur ou à la conversation, ou aux deux. Lorsque vous définissez la valeur de la propriété de votre état, la clé est définie pour vous en interne avec les informations contenues dans le contexte du tour pour assurer que chaque utilisateur ou conversation est placé dans le compartiment et la propriété corrects. Plus précisément, les clés sont définies comme suit :

- L’état utilisateur crée une clé à l’aide de l’*ID de canal* et de l’*ID d’origine*. Par exemple, _{Activity.ChannelId}/users/{Activity.From.Id}#YourPropertyName_
- L’état des conversations crée une clé à l’aide de l’*ID de canal* et de l’*ID de conversation*. Par exemple, _{Activity.ChannelId}/conversations/{Activity.Conversation.Id}#YourPropertyName_
- L’état des conversations privées crée une clé à l’aide de l’*ID de canal*, l’*ID d’origine* et de l’*ID de conversation*. Par exemple, _{Activity.ChannelId}/conversations/{Activity.Conversation.Id}/users/{Activity.From.Id}#YourPropertyName_

### <a name="when-to-use-each-type-of-state"></a>Quand utiliser chaque type d’état

L’état de conversation permet de suivre le contexte de la conversation, notamment :

- Le bot a-t-il posé une question à l’utilisateur et laquelle ?
- Quel est le sujet actuel de la conversation ou quel était le dernier ?

L’état d’utilisateur permet de suivre des informations sur l’utilisateur, notamment :

- Des informations non critiques sur l’utilisateur, comme son nom et ses préférences, un paramètre d’alarme ou une préférence d’alerte
- Des informations sur la dernière conversation avec le bot
  - Par exemple, un bot de support produit peut suivre les produits sur lesquels l’utilisateur s’est informé.

L’état de conversation privée convient aux canaux prenant en charge les conversations de groupe où vous souhaitez suivre des informations spécifiques à un utilisateur et à une conversation. Prenons l’exemple du bot d’un système de réponse en classe (ou « clicker ») :

- Le bot peut agréger et afficher les réponses des étudiants à une question donnée.
- Le bot peut agréger les performances de chaque étudiant et leur relayer en privé ces informations au terme de la session.

Pour plus d’informations sur l’utilisation de ces compartiments prédéfinis, consultez [l’article de procédure d’état](bot-builder-howto-v4-state.md).

### <a name="connecting-to-multiple-databases"></a>Connexion à plusieurs bases de données

Si votre bot doit se connecter à plusieurs bases de données, créez une couche de stockage pour chaque base de données.
Pour chaque couche de stockage, créez les objets de gestion d’état dont vous avez besoin pour prendre en charge vos propriétés d’état.

## <a name="state-property-accessors"></a>Accesseurs de propriété d’état

*Les accesseurs de propriété d’état* servent à lire ou écrire réellement l’une de vos propriétés d’état et fournissent les méthodes *get*, *set*, et *delete* pour accéder à vos propriétés d’état au sein d’un tour. Pour créer un accesseur, vous devez fournir le nom de la propriété, qui a généralement lieu lors de l’initialisation de votre bot. Ensuite, vous pouvez utiliser cet accesseur pour obtenir et manipuler cette propriété de l’état de votre bot.

Les accesseurs permettre au SDK d’obtenir l’état à partir du stockage sous-jacent et de mettre à jour le *cache d’état* du bot pour vous. Le cache d’état est un cache local géré par votre bot qui stocke l’objet d’état pour vous, autorisant des opérations de lecture et d’écriture sans accès au stockage sous-jacent. S’il n’est pas déjà dans le cache, l’appel de la méthode *get* de l’accesseur récupère l’état et le place également dedans. Une fois récupérée, la propriété d’état peut être manipulée comme une variable locale.

La méthode *delete* de l’accesseur supprime la propriété à partir du cache et la supprime également du stockage sous-jacent.

> [!IMPORTANT]
> Pour le premier appel à une méthode *get* de l’accesseur, vous devez fournir une méthode de fabrique pour créer l’objet s’il n’existe pas encore dans votre état. Si aucune méthode de fabrique n’est indiquée, vous obtiendrez une exception. Vous trouverez plus d’informations sur l’utilisation d’une méthode de fabrique dans l’[article de procédure d’état](bot-builder-howto-v4-state.md).

Pour rendre persistantes les modifications apportées à la propriété d’état que vous obtenez à partir de l’accesseur, vous devez mettre à jour la propriété dans le cache d’état. Vous pouvez le faire via un appel de la méthode *set* de l’accesseur, qui définit la valeur de votre propriété dans le cache et qui est disponible si une lecture ou une mise à jour est nécessaire plus tard dans le tour. Pour conserver réellement ces données sur le stockage sous-jacent (et les rendre ainsi disponibles après le tour actuel), vous devez [enregistrer votre état](#saving-state).

### <a name="how-the-state-property-accessor-methods-work"></a>Fonctionnement des méthodes de l’accesseur de propriété d’état

Les méthodes d’accesseur constituent le moyen principal pour que votre bot interagisse avec l’état. La façon dont chacune fonctionne et l’interaction des couches sous-jacentes sont comme suit :

- La méthode *get* de l’accesseur :
  - L’accesseur demande une propriété du cache d’état.
  - Si la propriété est dans le cache, elle est retournée. Sinon, obtenez-la à partir de l’objet de gestion d’état.
    Si elle n’est pas encore dans l’état, utilisez la méthode de fabrique fournie dans l’appel *get* de l’accesseur. -La méthode *set* de l’accesseur :
  - Mettez à jour le cache d’état avec la nouvelle valeur de propriété.
- La méthode *Enregistrer les changements* de l’objet de gestion d’état :
  - Vérifiez les modifications apportées à la propriété dans le cache d’état.
  - Écrivez cette propriété dans le stockage.

## <a name="saving-state"></a>Enregistrement de l’état

Lorsque vous appelez la méthode set de l’accesseur pour enregistrer l’état mis à jour, cette propriété d’état n’a pas encore été enregistrée dans votre stockage persistant, elle est uniquement enregistrée dans le cache d’état de votre bot. Pour enregistrer les modifications dans le cache d’état de votre état persistant, vous devez appeler la méthode *Enregistrer les modifications* de l’objet de gestion d’état, qui est disponible sur l’implémentation de la classe d’état de bot mentionnée ci-dessus (comme l’état utilisateur ou l’état des conversations).

L’appel de la méthode Enregistrer les modifications pour un objet de gestion d’état (par exemple, les compartiments mentionnés ci-dessus) enregistre toutes les propriétés dans le cache d’état définies pour pointer vers ce compartiment, mais pas pour les autres compartiments que vous pouvez avoir dans l’état de votre bot.

> [!TIP]
> L’état du bot implémente un comportement « la dernière écriture l’emporte », qui permet à la dernière écriture de remplacer l’état écrit précédemment. Cela peut fonctionner pour de nombreuses applications, mais avec des implications, en particulier pour les scénarios scale-out en cas de simultanéité ou de latence.

Si vous avez des middleware personnalisés pouvant mettre à jour un état une fois votre gestionnaire de tour terminé, envisagez la [gestion de l’état dans le middleware](bot-builder-concept-middleware.md#handling-state-in-middleware).

## <a name="additional-resources"></a>Ressources supplémentaires

- [État de dialogue](bot-builder-concept-dialog.md#dialog-state)
- [Écrire directement dans le stockage](bot-builder-howto-v4-storage.md)
- [Enregistrer les données de la conversation et de l’utilisateur](bot-builder-howto-v4-state.md)