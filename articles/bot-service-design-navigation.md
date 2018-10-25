---
title: Concevoir la navigation d’un bot | Microsoft Docs
description: Découvrez comment concevoir une bonne structure de navigation pour votre bot et éviter les erreurs de conception les plus courantes.
keywords: navigation, vue d’ensemble, bot rebelle, bot stupide, bot mystérieux, bot Captain Obvious, bot qui n’oublie rien
author: matvelloso
ms.author: mateusv
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
ms.openlocfilehash: f2a97b35f7e83a825e533be528951e8c04c521a1
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2018
ms.locfileid: "49998079"
---
# <a name="design-bot-navigation"></a>Concevoir la navigation d’un bot

Les utilisateurs peuvent parcourir les sites web avec des barres de navigation, les applications avec des menus et les navigateurs web avec des boutons comme **Suivant** et **Retour**. Toutefois, aucune de ces techniques de navigation bien connues ne répond complètement aux exigences de navigation dans un bot. Comme [nous l’avons vu](~/bot-service-design-conversation-flow.md#handle-interruptions), les utilisateurs interagissent souvent avec les bots de façon non linéaire, ce qui rend difficile le travail de conception d’une structure de navigation offrant systématiquement une excellente expérience utilisateur. 

Prenez les dilemmes suivants :

- Comment faire en sorte que l’utilisateur ne se perde pas dans la conversation avec le bot ? 
- L’utilisateur peut-il faire « Retour » dans une conversation avec un bot ? 
- Comment l’utilisateur accède-t-il au « menu principal » lors d’une conversation avec un bot ? 
- Comment l’utilisateur peut-il « annuler » une opération lors d’une conversation avec un bot ? 

Les caractéristiques de conception de la navigation du bot dépendent en grande partie des fonctionnalités qu’il gère. Quel que soit le type de bot développé, il est essentiel d’éviter les pièges les plus courants des interfaces conversationnelles mal conçues. Cet article décrit ces pièges suivant cinq personnalités : le « bot rebelle », le « bot stupide », le « bot mystérieux », le « bot Captain Obvious » et le « bot qui n’oublie rien ». 

> [!TIP]
> Pour prévenir l’apparition de chacun de ces types de personnalités chez le bot, il faut généralement bien [gérer les interruptions des utilisateurs](v4sdk/bot-builder-howto-handle-user-interrupt.md).

## <a name="the-stubborn-bot"></a>Le « bot rebelle »

Le bot rebelle tient absolument à maintenir le fil de la conversation, même lorsque l’utilisateur tente de l’orienter dans une autre direction. 

Examinez le scénario suivant : 

![robot](~/media/bot-service-design-navigation/stubborn-bot-new.png)

Les utilisateurs changent souvent d’avis, décident d’annuler, voire souhaitent tout reprendre à zéro. 

> [!TIP]
> <b>Ce qu’il faut faire</b> : concevoir le bot de façon à prendre en compte le fait que l’utilisateur peut à tout moment tenter de modifier le cours de la conversation. 
>
> <b>Ce qu’il ne faut pas faire</b> : concevoir le bot de façon à ignorer les entrées utilisateur et à répéter toujours la même question dans une boucle sans fin. 

Il existe de nombreuses manières d’éviter ce piège, mais le moyen le plus simple d’empêcher un bot de poser la même question indéfiniment est peut-être de spécifier un nombre maximal de nouvelles tentatives pour chaque question. Un bot conçu de cette manière ne fait rien « d’intelligent » pour comprendre l’entrée de l’utilisateur et y réagir de façon appropriée, mais évite au moins de poser en boucle la même question. 

## <a name="the-clueless-bot"></a>Le « bot stupide »

Le bot stupide répond de manière absurde lorsqu’il ne comprend pas la tentative de l’utilisateur d’accéder à certaines fonctionnalités. Il arrive que l’utilisateur essaye des commandes par mot clé courantes comme « aide » ou « annuler » en partant du principe que le bot réagira certainement de manière appropriée.

Examinez le scénario suivant : 

![robot](~/media/bot-service-design-navigation/clueless-bot.png)

S’il est tentant de concevoir chaque dialogue du bot de façon à guetter et réagir convenablement à certains mots clés, cette approche est déconseillée. 

> [!TIP]
> <b>Ce qu’il faut faire</b> : implémenter un [intergiciel (middleware)](v4sdk/bot-builder-create-middleware.md) qui cherche les mots clés spécifiés (par exemple, « aide », « annuler », « recommencer », etc.) dans les entrées utilisateur et réagit de façon appropriée. 
> 
> <b>Ce qu’il ne faut pas faire</b> : concevoir chaque dialogue de façon à chercher les mots clés de la liste dans les entrées utilisateur. 

En définissant la logique dans votre **intergiciel (middleware)**, vous la rendez accessible à chaque échange avec l’utilisateur. Ainsi, les différents dialogues et invites peuvent être conçus de façon à ignorer sans risque les mots clés si nécessaire.

## <a name="the-mysterious-bot"></a>Le « bot mystérieux »

Le bot mystérieux n’indique en rien qu’il a reçu l’entrée de l’utilisateur. 

Examinez le scénario suivant : 

![robot](~/media/bot-service-design-navigation/mysterious-bot.png)

Dans certains cas, cette situation est le signe que le bot est en panne. Cependant, il peut simplement s’avérer qu’il est occupé à traiter l’entrée de l’utilisateur et n’a pas encore fini de compiler sa réponse. 

> [!TIP]
> <b>Ce qu’il faut faire</b> : concevoir le bot de façon à accuser immédiatement réception de l’entrée utilisateur, même dans les cas où il met un certain temps à compiler sa réponse. 
> 
> <b>Ce qu’il ne faut pas faire</b> : concevoir le bot de façon à n’accuser réception de l’entrée utilisateur qu’après avoir fini de compiler sa réponse.

En confirmant immédiatement l’entrée de l’utilisateur, vous éliminez tout risque de confusion quant à l’état du bot. Si votre réponse est très longue à compiler, vous pouvez envoyer un message du type « XXX est en train d’écrire… » pour indiquer que votre bot fonctionne, puis le compléter par un [message proactif](v4sdk/bot-builder-howto-proactive-message.md).

## <a name="the-captain-obvious-bot"></a>Le « bot Captain Obvious »

Le bot Captain Obvious donne des informations non sollicitées, complètement évidentes et par conséquent inutiles pour l’utilisateur. 

Examinez le scénario suivant :

![robot](~/media/bot-service-design-navigation/captainobvious-bot.png)

> [!TIP]
> <b>Ce qu’il faut faire</b> : concevoir le bot de façon à donner des informations utiles à l’utilisateur. 
> 
> <b>Ce qu’il ne faut pas faire</b> : concevoir le bot de façon à donner des informations non sollicitées qui ne seront probablement pas utiles à l’utilisateur.

En concevant votre bot de façon à donner des informations utiles, vous augmentez les chances que l’utilisateur interagisse avec lui.

## <a name="the-bot-that-cant-forget"></a>Le « bot qui n’oublie rien »

Le bot qui n’oublie rien intègre à tort des informations issues de conversations passées dans la conversation actuelle. 

Examinez le scénario suivant :

![robot](~/media/bot-service-design-navigation/rememberall-bot.png)

> [!TIP]
> <b>Ce qu’il faut faire</b> : concevoir le bot de façon à rester sur le même sujet de conversation, sauf si/jusqu'à ce que l’utilisateur exprime le désir de revenir à un ancien sujet. 
> 
> <b>Ce qu’il ne faut pas faire</b> : concevoir le bot de façon à introduire des informations provenant de conversations passées alors qu’elles ne sont pas pertinentes pour la conversation actuelle.

En gardant le même sujet de conversation, vous réduisez le risque de confusion et de mécontentement et augmentez les chances que l’utilisateur poursuive ses interactions avec le bot.

## <a name="next-steps"></a>Étapes suivantes

Concevoir le bot de façon à éviter les pièges courants des interfaces conversationnelles mal conçues, c’est faire un grand pas vers une expérience utilisateur de qualité. 

Renseignez-vous ensuite sur les [éléments d’expérience utilisateur](~/bot-service-design-user-experience.md) sur lesquels s’appuient généralement les bots pour échanger des informations avec les utilisateurs. 
