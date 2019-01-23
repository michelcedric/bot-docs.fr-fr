---
title: Kit SDK Bot Framework pour Node.js | Microsoft Docs
description: Explorez le kit SDK Bot Framework pour Node.js, framework de génération de bots puissant et facile à utiliser.
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 2e25237b616810f5ef10442fec41834568afcb59
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/11/2019
ms.locfileid: "54224737"
---
# <a name="bot-framework-sdk-for-nodejs"></a>SDK Bot Framework pour Node.js

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-overview.md)
> - [Node.JS](../nodejs/bot-builder-nodejs-overview.md)
> - [REST](../rest-api/bot-framework-rest-overview.md)

Le kit SDK Bot Framework pour Node.js est un framework puissant et facile à utiliser qui permet aux développeurs Node.js de coder des bots dans un environnement qui leur est familier.
Il permet de créer une large palette d’interfaces utilisateur conversationnelles, allant de simples invites à des conversations libres.

La logique de conversation de votre bot est hébergée en tant que service web. Le kit SDK Bot Framework utilise <a href="http://restify.com">restify</a>, framework populaire permettant de générer des services web et de créer le serveur web du bot. Le kit SDK est également compatible avec <a href="http://expressjs.com/">Express</a> et l’utilisation d’autres frameworks d’applications web est possible avec certaines adaptations. 

Le kit SDK vous permet de tirer parti des fonctionnalités du SDK suivantes : 

- Système puissant pour créer des dialogues servant à encapsuler la logique de conversation.
- Invites intégrées pour les opérations simples comme Oui/Non, les chaînes, les nombres et les énumérations, ainsi que la prise en charge des messages contenant des images et des pièces jointes, et des cartes enrichies contenant des boutons.
- Prise en charge intégrée de puissants frameworks d’intelligence artificielle comme <a href="http://luis.ai" target="_blank">LUIS</a>.
- Gestionnaires d’événements et modules de reconnaissance intégrés qui guident l’utilisateur tout au long de la conversation, apportant de l’aide, des informations de navigation, des clarifications et des confirmations selon les besoins.

## <a name="get-started"></a>Prise en main

Si vous débutez dans la rédaction de bots, [créez votre premier bot avec Node.js](bot-builder-nodejs-quickstart.md) en suivant des instructions détaillées qui vous aideront à configurer votre projet, installer le kit SDK et exécuter votre premier bot. 

Si vous débutez avec le kit SDK Bot Framework pour Node.js, vous pouvez commencer par explorer les concepts clés, ce qui vous aidera à comprendre les principaux composants du kit SDK Bot Framework ; pour cela, consultez [Concepts clés](bot-builder-nodejs-concepts.md).

Pour vérifier que votre bot répond aux principaux scénarios utilisateur, passez en revue les [principes de conception](../bot-service-design-principles.md) et [modèles d’exploration](../bot-service-design-pattern-task-automation.md) pour obtenir des conseils.

## <a name="get-samples"></a>Obtenir les exemples

Les [exemples du kit SDK Bot Framework pour Node.js](bot-builder-nodejs-samples.md) présentent des bots centrés sur les tâches qui montrent comment tirer parti des fonctionnalités du kit SDK Bot Framework pour Node.js. Vous pouvez utiliser les exemples pour commencer rapidement à concevoir de puissants robots aux fonctionnalités enrichies.

## <a name="next-steps"></a>Étapes suivantes
> [!div class="nextstepaction"]
> [Concepts clés](bot-builder-nodejs-concepts.md)

## <a name="additional-resources"></a>Ressources supplémentaires

Les procédures centrées sur les tâches suivantes illustrent diverses fonctionnalités du kit SDK Bot Framework pour Node.js.

* [Répondre aux messages](bot-builder-nodejs-use-default-message-handler.md)
* [Gérer les actions de l’utilisateur](bot-builder-nodejs-dialog-actions.md)
* [Reconnaître l’intention de l’utilisateur](bot-builder-nodejs-recognize-intent-messages.md)
* [Envoyer une carte enrichie](bot-builder-nodejs-send-rich-cards.md)
* [Envoyer des pièces jointes](bot-builder-nodejs-send-receive-attachments.md)
* [Enregistrer les données utilisateur](bot-builder-nodejs-save-user-data.md)


Si vous rencontrez des problèmes ou avez des suggestions concernant le kit SDK Bot Framework pour Node.js, consultez la page [Support](../bot-service-resources-links-help.md) pour obtenir la liste des ressources disponibles. 


[DesignGuide]: ../bot-service-design-principles.md 
[DesignPatterns]: ../bot-service-design-pattern-task-automation.md 
[HowTo]: bot-builder-nodejs-use-default-message-handler.md 
