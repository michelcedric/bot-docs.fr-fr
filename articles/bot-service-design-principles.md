---
title: Principes de conception des bots | Microsoft Docs
description: Découvrez comment créer un excellent bot conversationnel et comment planifier et concevoir des bots répondant à vos besoins tout en assurant la satisfaction de vos utilisateurs.
keywords: meilleures pratiques, conception de bot
author: matvelloso
ms.author: mateusv
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
ms.openlocfilehash: 3ac767e525c1082005f4521af9e9714dd8c39cff
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2018
ms.locfileid: "49998516"
---
# <a name="principles-of-bot-design"></a>Principes de conception des bots

Le Bot Framework permet aux développeurs de créer des expériences de bot attrayantes, pouvant résoudre une multitude de problèmes d’entreprise. En apprenant les concepts décrits dans cette section, vous serez en mesure de concevoir un bot conforme aux meilleures pratiques et tirant parti des leçons apprises jusqu’à présent dans ce domaine relativement nouveau. 

## <a name="designing-a-bot"></a>Conception d’un bot

Lorsque vous générez un bot, vous espérez généralement qu’il séduira des utilisateurs. Vous espérez aussi certainement que ces utilisateurs préféreront l’expérience offerte par votre bot pour répondre à leurs besoins spécifiques plutôt que d’autres expériences (applications, sites web, appels téléphoniques, etc.). En d’autres termes, votre bot est en concurrence avec d’autres systèmes comme des applications et des sites web aux yeux des utilisateurs. Alors, comment vous pouvez mettre toutes les chances de votre côté pour que votre bot atteigne son objectif final, à savoir séduire et fidéliser les utilisateurs ? En fait, il s’agit simplement de donner la priorité aux facteurs appropriés lors de la conception de votre bot.

## <a name="factors-that-do-not-guarantee-a-bots-success"></a>Facteurs qui ne garantissent pas la réussite d’un bot

Lorsque vous concevez votre bot, sachez qu’aucun des facteurs suivants ne garantit sa réussite : 

- **Degré d’« intelligence » du bot** : dans la plupart des cas, rendre votre bot plus intelligent ne garantit pas la satisfaction des utilisateurs ni l’adoption de votre plateforme. En réalité, de nombreux bots n’affichent que peu de capacités avancées en termes de Machine Learning et de langage naturel. Un bot peut bénéficier de telles capacités si elles sont nécessaires à la résolution des problèmes qu’il est supposé résoudre. Cependant, ne partez pas du principe qu’il existe une corrélation entre l’intelligence d’un bot et son adoption par les utilisateurs.

- **Degré de langage naturel pris en charge par le bot** : votre bot peut s’avérer excellent pour les conversations. Il peut bénéficier d’un vaste vocabulaire et peut même faire de très bonnes plaisanteries. Cependant, à moins qu’elles ne contribuent à résoudre les problèmes de vos utilisateurs, ces capacités ne favoriseront que très peu la réussite de votre bot. En fait, certains bots n’ont aucune capacité conversationnelle. Et dans de nombreux cas, cela ne pose aucun problème.

- **Voix** : l’ajout de capacités vocales à un bot n’ouvre pas systématiquement la voie à d’excellentes expériences utilisateur. Souvent, obliger les utilisateurs à utiliser la voix peut entraîner des expériences frustrantes. Lorsque vous concevez votre bot, demandez-vous toujours si la voix représente bien le canal approprié pour le problème donné. Serons-nous en présence d’un environnement bruyant ? La voix communiquera-t-elle les informations à partager avec l’utilisateur ? 

## <a name="factors-that-do-influence-a-bots-success"></a>Facteurs qui influencent la réussite d’un bot

Les applications et sites web connaissant le plus de succès ont au moins une chose en commun : une excellente expérience utilisateur. On observe la même chose pour les bots. Garantir une excellente expérience utilisateur doit donc être votre priorité absolue lorsque vous concevez un bot. Voici certains éléments clés à prendre en compte :

- Le bot résout-il facilement le problème de l’utilisateur, et ce, moyennant un nombre minimum d’étapes ?

- Le bot résout-il le problème de l’utilisateur plus efficacement/plus facilement/plus rapidement que toute autre expérience ?

- Le bot s’exécute-t-il sur les appareils et plateformes privilégiés par l’utilisateur ?

- Le bot est-il détectable ? Les utilisateurs savent-ils naturellement comment procéder lorsqu’ils l’utilisent ?

Aucune de ces questions n’est directement liée aux facteurs tels que le degré d’intelligence du bot, ses capacités en termes de langage naturel, l’utilisation ou non du Machine Learning, ou encore le langage de programmation utilisé pour le créer. Les utilisateurs sont peu susceptibles de se préoccuper de ces aspects dans la mesure où le bot résout le problème auquel ils sont confrontés et offre une excellente expérience utilisateur. Pour bénéficier d’une excellente expérience utilisateur avec le bot, les utilisateurs ne doivent pas avoir à saisir trop de données, à trop parler, à répéter la même chose plusieurs fois ou à expliquer des choses que le bot devrait déjà savoir.

> [!TIP]
> Quel que soit le type d’application que vous créez (bot, site web ou application), considérez l’expérience utilisateur comme une priorité absolue.

Le processus de conception d’un bot est similaire à celui appliqué pour concevoir une application ou un site web. Ainsi, les leçons tirées de la création d’interfaces et d’expériences utilisateur pour des applications et des sites web depuis des dizaines d’années continuent de s’appliquer lorsqu’il s’agit de concevoir des bots. 

Lorsque vous doutez de l’approche de conception à adopter pour votre bot, prenez du recul et posez-vous la question suivante : comment pourriez-vous résoudre ce problème avec une application ou un site web ? Il est probable que la même réponse s’applique à la conception d’un bot. 

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous êtes familiarisé avec certains principes fondamentaux de la conception de bot, vous allez en apprendre plus sur la conception de l’expérience utilisateur et les modèles courants en poursuivant cette section.