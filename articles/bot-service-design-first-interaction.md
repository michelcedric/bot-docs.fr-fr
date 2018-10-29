---
title: Concevoir une première interaction d’utilisateur avec un robot | Microsoft Docs
description: Découvrez ce qui fait une excellente première expérience utilisateur et comment concevoir vos robots pour réussir.
keywords: première impression, débuts, langage et menu
author: matvelloso
ms.author: mateusv
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
ms.openlocfilehash: f59a7acbdb7d580aebeef6ffe81d8b15505aed90
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2018
ms.locfileid: "49999806"
---
# <a name="design-a-bots-first-user-interaction"></a>Concevoir une première interaction d’utilisateur avec un robot

## <a name="first-impressions-matter"></a>Les premières impressions comptent

La toute première interaction entre l’utilisateur et un robot est déterminant pour l’expérience utilisateur. Lors de la conception de votre robot, gardez à l’esprit que ce premier message ne devrait se limiter à dire « bonjour ». Lorsque vous créez une application, vous concevez le premier écran pour fournir des indications [de navigation](bot-service-design-navigation.md) importantes. Les utilisateurs doivent comprendre de façon intuitive des éléments tels que l’emplacement et le fonctionnement du menu, où trouver de l’aide, la politique de confidentialité, etc. Lorsque vous concevez un robot, la première interaction de l’utilisateur avec le celui-ci doit fournir le même type d’informations. 

## <a name="language-versus-menus"></a>Langage et menus 

Considérons les deux conceptions suivantes :

### <a name="design-1"></a>Conception 1

![robot](~/media/bot-service-design-first-interaction/hello1.png)


### <a name="design-2"></a>Conception 2

![robot](~/media/bot-service-design-first-interaction/hello2.png)

Démarrer le bot avec une question ouverte telle que « Comment puis-je vous aider ? » n’est généralement pas recommandé. Si votre robot peut faire une centaine de choses différentes, il y a de fortes chances pour que les utilisateurs ne puissent pas deviner la nature de la plupart d’entre elles. Votre robot ne leur ayant pas dit ce qu’il peut faire, comment pourraient-ils le savoir ?

Les menus offrent une solution simple à ce problème. Tout d’abord, en répertoriant les options disponibles, votre robot informe l’utilisateur de ses capacités. Deuxièmement, les menus évitent à l’utilisateur de devoir trop saisir en lui permettant de simplement cliquer. Enfin, l’utilisation de menus peut simplifier considérablement vos modèles de langage naturel en limitant la portée de l’entrée que le robot peut recevoir de l’utilisateur. 

> [!TIP]
> Les menus constituent un outil précieux lors de la conception de robots pour une expérience utilisateur exceptionnelle. Ne les écartez pas sous prétexte qu’ils ne sont pas « assez intelligents ». Vous pouvez concevoir votre robot de façon à ce qu’il utilise des menus tout en prenant en charge la saisie sous forme libre. Si un utilisateur répond au menu initial en tapant plutôt qu’en sélectionnant une option de menu, votre robot peut tenter d’analyser l’entrée textuelle de l’utilisateur. 

Vous pouvez également poser des questions plus pointues pour orienter l’utilisateur si le robot a une fonction spécifique. Par exemple, si votre bot est chargé de prendre des commandes de sandwich, votre première interaction pourrait être « Bonjour ! Je suis ici pour prendre la commande de votre sandwich. Quel type de pain voulez-vous ? Nous avons du pain blanc, de froment, ou du pain de seigle. » De cette manière, l’utilisateur sait comment répondre, et reçoit des repères pour naviguer dans la conversation.

## <a name="other-considerations"></a>Autres points à considérer

En plus de fournir une première interaction intuitive et aisément navigable, un robot bien conçu permet à l’utilisateur d’accéder à des informations sur sa politique de confidentialité et ses conditions d’utilisation. 

> [!TIP]
> Si votre robot collecte des données personnelles de l’utilisateur, il est important de le faire savoir et de décrire l’usage qui sera fait de ces données.

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous êtes familiarisé avec certains principes de base de la conception de la première interaction entre un utilisateur et un robot, découvrez comment [concevoir le flux de conversation](~/bot-service-design-conversation-flow.md).
