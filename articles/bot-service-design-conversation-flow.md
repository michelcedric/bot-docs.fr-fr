---
title: Concevoir et contrôler un flux de conversation | Microsoft Docs
description: Découvrez comment concevoir et contrôler un flux de conversation dans votre bot pour offrir une excellente expérience utilisateur.
keywords: conception, contrôle, flux de conversation, gestion des interruptions, vue d’ensemble
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 4/8/2018
ms.openlocfilehash: 82503407c527bee4d56edc0cb58fa507239350e3
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2018
ms.locfileid: "49998448"
---
::: moniker range="azure-bot-service-3.0"

# <a name="design-and-control-conversation-flow"></a>Concevoir et contrôler un flux de conversation

[!INCLUDE [pre-release-label](./includes/pre-release-label-v3.md)]

Dans une application classique, l’interface utilisateur (IU) se compose d’une série d’écrans. 
Une application ou un site web peut utiliser un ou plusieurs écrans selon les besoins du processus en termes d’échange d’informations avec l’utilisateur. 
La plupart des applications commencent par afficher un écran principal assurant l’accueil des utilisateurs et leur permettant de naviguer vers d’autres écrans. Ceux-ci peuvent offrir diverses fonctions comme la création d’une nouvelle commande, la consultation de produits ou la recherche d’aide.

Comme les applications et les sites web, les bots sont dotés d’une interface utilisateur. Toutefois, celle-ci ne se compose pas d’écrans mais de **boîtes de dialogue**. Les dialogues vous aident à préserver votre position dans une conversation, à inviter les utilisateurs à fournir les informations nécessaires et à valider les entrées. Ils sont utiles pour la gestion des conversations à plusieurs tours et des collectes simples d’informations basées sur des formulaires pour l’accomplissement de tâches telles que la réservation d’un vol.

Les boîtes de dialogue permettent au développeur de bot de séparer logiquement les différentes zones de fonctionnalités du bot et de guider le flux de conversation. Par exemple, vous pouvez concevoir une boîte de dialogue contenant la logique qui permet à l’utilisateur de consulter les produits et une boîte de dialogue distincte contenant la logique qui permet à l’utilisateur de créer une nouvelle commande. 

Les boîtes de dialogue peuvent éventuellement comporter des interfaces graphiques. Elles peuvent inclure des boutons, du texte et d’autres éléments ou être entièrement basées sur la voix. Les boîtes de dialogue intègrent également des actions permettant d’effectuer des tâches telles que l’appel d’autres boîtes de dialogue ou le traitement d’entrées utilisateur.

## <a name="using-dialogs-to-manage-conversation-flow"></a>Utilisation de boîtes de dialogue pour gérer le flux de conversation

[!INCLUDE [Dialog flow example](./includes/snippet-dotnet-manage-conversation-flow-intro.md)]

Pour obtenir une description détaillée de la gestion du flux de conversation à l’aide de boîtes de dialogue et du Kit de développement logiciel (SDK) Bot Builder, consultez :

- [Gérer un flux de conversation avec des boîtes de dialogue (.NET)](./dotnet/bot-builder-dotnet-manage-conversation-flow.md)
- [Gérer un flux de conversation avec des boîtes de dialogue (Node.js)](./nodejs/bot-builder-nodejs-manage-conversation-flow.md)

## <a name="dialog-stack"></a>Pile de boîtes de dialogue

Lorsqu’une boîte de dialogue en appelle une autre, le Bot Builder ajoute la nouvelle boîte de dialogue en haut de la pile de boîtes de dialogue. 
La boîte de dialogue qui se trouve en haut de la pile contrôle la conversation. 
Chaque nouveau message envoyé par l’utilisateur est traité par cette boîte de dialogue jusqu’à ce qu’elle se ferme ou redirige l’utilisateur vers une autre boîte de dialogue. 
Lorsqu’une boîte de dialogue se ferme, elle est supprimée de la pile, et la boîte de dialogue qui la précède dans la pile prend le contrôle de la conversation. 

> [!IMPORTANT]
> Pour concevoir efficacement le flux de conversation d’un bot, il est essentiel de comprendre comment le Bot Builder construit et déconstruit la pile de boîtes de dialogue lorsqu’une boîte de dialogue se ferme, en appelle une autre, etc. 

## <a name="dialogs-stacks-and-humans"></a>Boîtes de dialogue, piles et humains

Il peut être tentant de supposer que les utilisateurs accéderont aux différentes boîtes de dialogue, créant ainsi une pile de boîtes de dialogue, puis, à un moment donné, navigueront dans le sens opposé, supprimant les boîtes de dialogue de la pile une à une de façon claire et ordonnée. 
Par exemple, l’utilisateur accédera d’abord à la boîte de dialogue racine, appellera la boîte de dialogue de nouvelle commande, puis appellera la boîte de dialogue de recherche de produit. 
Puis, il sélectionnera un produit, confirmera et quittera ainsi la boîte de dialogue de recherche de produit. Ensuite, il terminera la commande et quittera ainsi la boîte de dialogue de nouvelle commande. Il reviendra alors à la boîte de dialogue racine. 

Il serait très pratique que les utilisateurs naviguent toujours de façon logique et linéaire. Cependant, cela se produit rarement. 
Les humains ne communiquent pas par « piles ». Ils ont tendance à changer fréquemment d’avis. 
Considérez l'exemple suivant : 

![robot](./media/bot-service-design-conversation-flow/stack-issue.png)

Il se peut que votre bot construise logiquement une pile de boîtes de dialogue, mais que l’utilisateur décide de faire quelque chose de complètement différent ou de poser une question qui n’est pas liée au sujet actuel. 
Dans l’exemple, l’utilisateur pose une question, alors que la boîte de dialogue l’invite à répondre par oui ou par non. 
Comment votre boîte de dialogue doit-elle répondre ?

- Insister pour que l’utilisateur réponde d’abord à la question. 
- Ignorer toutes les actions précédentes de l’utilisateur, réinitialiser toute la pile de boîtes de dialogue et recommencer depuis le début en essayant de répondre à la question de l’utilisateur. 
- Tenter de répondre à la question de l’utilisateur, puis revenir à la question oui/non et essayer de reprendre la procédure à ce stade. 

Il n’existe pas qu’une seule réponse *correcte* à cette question. En effet, la meilleure solution dépend des spécificités de votre scénario et des attentes raisonnables de l’utilisateur quant à la façon dont le bot va répondre. Toutefois, plus votre conversation devient complexe, plus les **dialogues** sont difficiles à gérer. Pour les cas de création de branches complexes, il peut être plus facile de créer votre propre flux de logique de contrôle pour suivre la conversation de votre utilisateur.

## <a name="next-steps"></a>Étapes suivantes

La gestion de la navigation de l’utilisateur dans les boîtes de dialogue et la conception du flux de conversation d’une manière qui permette aux utilisateurs d’atteindre leurs objectifs (y compris de façon non linéaire) constituent des enjeux majeurs de la conception de bot. 
[L’article suivant](./bot-service-design-navigation.md) passe en revue les pièges fréquents inhérents à une navigation mal conçue et aborde les stratégies permettant de les éviter. 

::: moniker-end

::: moniker range="azure-bot-service-4.0"
# <a name="design-and-control-conversation-flow"></a>Conception et contrôle d’un flux de conversation

Dans une application classique, l’interface utilisateur (IU) se compose d’une série d’écrans. Une application ou un site web peut utiliser un ou plusieurs écrans selon les besoins du processus en termes d’échange d’informations avec l’utilisateur. 
La plupart des applications commencent par afficher un écran principal assurant l’accueil des utilisateurs et leur permettant de naviguer vers d’autres écrans. Ceux-ci peuvent offrir diverses fonctions comme la création d’une nouvelle commande, la consultation de produits ou la recherche d’aide.

Comme les applications et les sites web, les bots sont dotés d’une interface utilisateur. Toutefois, celle-ci ne se compose pas d’écrans mais de **messages**. Les messages peuvent inclure des boutons, du texte et d’autres éléments ou être entièrement basés sur la voix. 

Alors qu’une application ou un site web classique peut inviter l’utilisateur à entrer une multitude d’informations sur un même écran, un bot recueillera la même quantité d’informations à l’aide de plusieurs messages. Ainsi, le processus de collecte d’informations auprès de l’utilisateur repose sur une expérience active : l’utilisateur tient une conversation active avec le bot. 

Un bot bien conçu offrira un flux de conversation naturel. Le bot doit être en mesure de gérer la conversation principale de façon transparente et de gérer correctement les interruptions et changements de sujet de conversation. 

## <a name="procedural-conversation-flow"></a>Flux de conversation procédural

La conversation avec un bot est généralement axée sur la tâche que le bot essaie d’effectuer (flux procédural). Dans ce contexte, le bot pose à l’utilisateur une série de questions pour recueillir toutes les informations dont il a besoin pour traiter la tâche.

Dans le cadre d’un flux de conversation procédural, vous définissez l’ordre des questions, et le bot pose les questions dans cet ordre. Vous pouvez organiser les questions en *modules* logiques pour conserver le code centralisé tout en restant focalisé sur le guidage du flux de conversation. Par exemple, vous pouvez concevoir un module contenant la logique qui permet à l’utilisateur de consulter les produits et un module distinct contenant la logique qui permet à l’utilisateur de créer une nouvelle commande. 

Vous pouvez structurer ces modules selon le type de flux souhaité (libre, séquentiel, etc.). Le Kit de développement logiciel (SDK) Bot Builder offre plusieurs bibliothèques qui vous permettent de construire n’importe quel flux de conversation requis par votre bot. Par exemple, la bibliothèque `prompts` vous permet de demander une entrée aux utilisateurs, la bibliothèque `waterfall` vous permet de définir une séquence de paires question/réponse, la bibliothèque `dialog control` vous permet de modulariser la logique de votre flux de conversation, etc. Toutes ces bibliothèques sont liées entre elles par un objet `dialogs`. Examinons plus en détail l’implémentation des modules sous forme de `dialogs` pour concevoir et gérer des flux de conversation, et voyons en quoi ce flux s’apparente au flux d’application classique.

![bot](./media/designing-bots/core/dialogs-screens.png)

Dans une application classique, tout commence par **l’écran principal**.
**L’écran principal** appelle **l’écran de nouvelle commande**.
**L’écran de nouvelle commande** conserve le contrôle jusqu’à ce qu’il se ferme ou appelle d’autres écrans, comme **l’écran de recherche de produit**. 
Si **l’écran de nouvelle commande** se ferme, l’utilisateur est renvoyé à **l’écran principal**.

Dans un bot, tout commence par le **dialogue racine**. 
La **boîte de dialogue racine** appelle la **boîte de dialogue de nouvelle commande**. 
À ce stade, la **boîte de dialogue de nouvelle commande** prend le contrôle de la conversation et le conserve jusqu’à ce qu’elle se ferme ou appelle d’autres boîtes de dialogue, comme la **boîte de dialogue de recherche de produit**. 
Si la **boîte de dialogue de nouvelle commande** se ferme, le contrôle de la conversation revient à **la boîte de dialogue racine**.

Consultez la rubrique conceptuelle sur le [flux de conversation](v4sdk/bot-builder-conversations.md) pour en savoir plus sur les types de conversation.

## <a name="handle-interruptions"></a>Gestion des interruptions

Il peut être tentant de supposer que les utilisateurs effectueront les tâches procédurales une par une de manière claire et ordonnée. 
Par exemple, dans le cadre d’un flux de conversation procédural utilisant des `dialogs`, l’utilisateur accédera d’abord à la boîte de dialogue racine, appellera la boîte de dialogue de nouvelle commande, puis appellera la boîte de dialogue de recherche de produit. Puis, il sélectionnera un produit, confirmera et quittera ainsi la boîte de dialogue de recherche de produit. Ensuite, il terminera la commande et quittera ainsi la boîte de dialogue de nouvelle commande. Il reviendra alors à la boîte de dialogue racine. 

Il serait très pratique que les utilisateurs naviguent toujours de façon logique et linéaire. Cependant, cela se produit rarement. 
Les humains ne communiquent pas avec des `dialogs` séquentielles. Ils ont tendance à changer fréquemment d’avis. 
Considérez l'exemple suivant : 

![robot](./media/bot-service-design-conversation-flow/stack-issue.png)

Il se peut que votre bot soit basé sur une approche procédurale, mais que l’utilisateur décide de faire quelque chose de complètement différent ou de poser une question qui n’est pas liée au sujet actuel. 
Dans l’exemple plus haut, l’utilisateur pose une question, alors que le bot l’invite à répondre par oui ou par non. 
Comment votre bot doit-il répondre ?

- Insister pour que l’utilisateur réponde d’abord à la question. 
- Ignorer toutes les actions précédentes de l’utilisateur, réinitialiser toute la pile de boîtes de dialogue et recommencer depuis le début en essayant de répondre à la question de l’utilisateur. 
- Tenter de répondre à la question de l’utilisateur, puis revenir à la question oui/non et essayer de reprendre la procédure à ce stade. 

Il n’existe aucune réponse *correcte* à cette question. En effet, la meilleure solution dépend des spécificités de votre scénario et des attentes raisonnables de l’utilisateur quant à la façon dont le bot va répondre. Pour plus d’informations, consultez l’article [Gérer les interruptions de l’utilisateur](v4sdk/bot-builder-howto-handle-user-interrupt.md).

## <a name="next-steps"></a>Étapes suivantes

La gestion de la navigation de l’utilisateur dans les boîtes de dialogue et la conception du flux de conversation d’une manière qui permette aux utilisateurs d’atteindre leurs objectifs (y compris de façon non linéaire) constituent des enjeux majeurs de la conception de bot. 
[L’article suivant](~/bot-service-design-navigation.md) passe en revue les pièges fréquents inhérents à une navigation mal conçue et aborde les stratégies permettant de les éviter. 

::: moniker-end
