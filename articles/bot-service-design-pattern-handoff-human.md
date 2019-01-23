---
title: Conversations de transition de robot à humain | Microsoft Docs
description: Apprenez à concevoir des situations dans lesquelles un utilisateur entame une conversation avec un robot qui doit ensuite la transférer à un humain.
author: matvelloso
ms.author: mateusv
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
ms.openlocfilehash: f18b375a1e4ebcf06d00d045e383db8b05fb5111
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/11/2019
ms.locfileid: "54225734"
---
# <a name="transition-conversations-from-bot-to-human"></a>Conversations de transition de robot à humain

Indépendamment de l’intelligence artificielle qu’un robot possède, il peut toujours arriver qu’il doive transférer la conversation à un être humain. Le robot doit reconnaître le moment où il a besoin de passer la main en offrant à l’utilisateur une transition claire et en douceur.

## <a name="scenarios-that-require-human-involvement"></a>Scénarios qui requièrent une intervention humaine

Nombreux sont les scénarios pouvant nécessiter qu’un robot transfère le contrôle de la conversation à un humain. Parmi ces scénarios figurent le *triage*, la *réaffectation* et la *supervision*. 

### <a name="triage"></a>Tri

Un appel classique au support technique commence par des questions très élémentaires auxquelles un robot peut facilement répondre. En tant que premier intervenant à répondre aux demandes entrantes des utilisateurs, un robot peut collecter le nom, l’adresse, la description du problème ou toute autre information pertinente de l’utilisateur, puis transférer le contrôle de la conversation à un agent. L’utilisation d’un robot pour trier les demandes entrantes permet aux agents de consacrer leur temps à la résolution de problèmes plutôt qu’à la collecte d’informations.

### <a name="escalation"></a>Escalade

Dans le cadre de support technique, en plus de collecter des informations, un robot peut être en mesure de répondre à des questions élémentaires ainsi que de résoudre certains problèmes simples, tels que la réinitialisation du mot de passe d’un utilisateur. En revanche, si une conversation indique que le problème d’un utilisateur est suffisamment complexe pour nécessiter une intervention humaine, le robot doit réaffecter le problème à un agent humain. Pour implémenter ce type de scénario, un robot doit être capable de différencier les problèmes qu’il est capable de résoudre de façon autonome de ceux qu’il doit réaffecter à un être humain. Un robot peut déterminer qu’il doit transférer le contrôle de la conversation à un humain de plusieurs façons. 

#### <a name="user-driven-menus"></a>Menus pilotés par l’utilisateur

Le moyen le plus simple pour un robot de gérer ce dilemme est peut-être de présenter à l’utilisateur un menu d’options. Les tâches que le robot peut gérer de façon autonome apparaissent dans le menu situé au-dessus d’un lien libellé « Parler à un agent ». Ce type d’implémentation ne nécessite ni d’apprentissage automatique avancé, ni compréhension du langage naturel. Le robot transfère simplement le contrôle de la conversation à un agent humain quand l’utilisateur sélectionne l’option « Parler à un agent ». 

#### <a name="scenario-driven"></a>Piloté par scénario

Le robot peut décider de transférer ou non le contrôle de la conversation selon qu’il détermine qu’il est capable ou non de gérer le scénario en cours. Le robot collecte des informations sur la demande de l’utilisateur, puis interroge sa liste interne de fonctionnalités pour déterminer s’il est capable de répondre à cette demande. Si le robot détermine qu’il est capable de répondre à la demande, il le fait, mais s’il détermine que la demande va au-delà des problèmes qu’il est capable de résoudre, il transfère le contrôle de la conversation à un agent humain.

#### <a name="natural-language"></a>Langage naturel

La compréhension du langage naturel et l’analyse des sentiments aident le robot à décider quand il doit transférer le contrôle de la conversation à un agent humain. Cela est particulièrement précieux pour déterminer si l’utilisateur est frustré ou souhaite parler à un agent humain. 
 
Le robot analyse le contenu des messages de l’utilisateur en utilisant l’<a href="https://www.microsoft.com/cognitive-services/en-us/text-analytics-api" target="blank">API Analyse de texte</a> pour déduire des sentiments, ou en utilisant l’<a href="https://www.luis.ai" target="_blank">API LUIS</a>. 


> [!TIP]
> La compréhension du langage naturel n’est pas forcément toujours être la meilleure méthode pour déterminer quand un robot doit transférer le contrôle de la conversation à un être humain. Comme les humains, les robots ne devinent pas toujours correctement, et des réponses non valides ont pour effet de frustrer l’utilisateur. Toutefois, si l’utilisateur opère une sélection dans un menu de choix valides, le robot répondra toujours de manière appropriée à cette entrée. 

### <a name="supervision"></a>Surveillance

Dans certains cas, l’agent humain voudra surveiller la conversation au lieu d’en prendre le contrôle.

Par exemple, considérons un scénario de support technique où un robot communique avec un utilisateur pour diagnostiquer des problèmes informatiques. Un modèle d’apprentissage automatique aide le robot à déterminer la cause la plus probable du problème. Cependant, avant de conseiller à l’utilisateur de prendre une mesure spécifique, le robot peut en privé confirmer le diagnostic et le remède avec l’agent humain, et lui demander l’autorisation de poursuivre. L’agent clique alors sur un bouton, le robot présente la solution à l’utilisateur, et le problème est résolu. Le robot effectue toujours l’essentiel du travail, mais l’agent conserve le contrôle de la décision finale. 

## <a name="transitioning-control-of-the-conversation"></a>Transition du contrôle de la conversation 

Quand un robot décide de transférer le contrôle d’une conversation à un humain, il peut informer l’utilisateur que ce transfert est en cours et mettre la conversation en état d’« attente » jusqu’à ce que qu’un agent soit disponible. 

Lorsque le robot attend qu’un humain soit disponible, il peut répondre automatiquement à tous les messages entrants de l’utilisateur avec une réponse par défaut telle que « en file d’attente ». Vous pouvez également faire en sorte que le robot supprime la conversation de l’état « attente » si l’utilisateur envoie des messages tels que « peu importe » ou « annuler ».

Vous spécifiez la manière dont les agents sont affectés aux utilisateurs en attente lorsque vous concevez votre robot. Par exemple, le robot peut implémenter un système de file d’attente simple : premier entré, premier sorti. Une logique plus complexe aurait affecter des utilisateurs à des agents sur la base de l’emplacement géographique, de la langue ou de tout autre facteur. Le robot pourrait également présenter à l’agent un certain type d’interface utilisateur permettant de sélectionner un utilisateur. Quand un agent devient disponible, il se connecte au robot et rejoint la conversation.

> [!IMPORTANT]
> Même après l’implication de l’agent, le robot reste en coulisses l’animateur de la conversation. L’utilisateur et l’agent ne communiquent jamais directement entre eux. Les messages sont routés via le robot. 

## <a name="routing-messages-between-user-and-agent"></a>Routage de messages entre utilisateur et agent

Une fois que l’agent connecté au robot, celui-ci commence à router les messages entre l’utilisateur et l’agent. Même s’il peut sembler à l’utilisateur et à l’agent qu’ils discutent directement entre eux, ils échangent en réalité des messages via le robot. Celui-ci reçoit un message de l’utilisateur et l’envoie à l’agent, puis reçoit un message de l’agent et l’envoie à l’utilisateur. 

> [!NOTE]
> Dans des scénarios plus avancés, le robot peut assumer une responsabilité au-delà du simple routage des messages entre l’utilisateur et l’agent. Par exemple, le robot peut déterminer quelle réponse est appropriée et simplement demander à l’agent de confirmer avant de continuer.

## <a name="sample-code"></a>Exemple de code

Pour une présentation complète de la manière de transférer des conversations de bot à humain à l’aide du kit SDK Bot Framework pour Node.js, consultez l’exemple <a href="https://github.com/palindromed/Bot-HandOff" target="_blank">Bot-HandOff</a> dans GitHub.

## <a name="additional-resources"></a>Ressources supplémentaires

::: moniker range="azure-bot-service-4.0"

- [Dialogues](v4sdk/bot-builder-dialog-manage-conversation-flow.md)
- <a href="https://www.microsoft.com/cognitive-services/en-us/text-analytics-api" target="blank">API Analyse de texte</a>

::: moniker-end

::: moniker range="azure-bot-service-3.0"

- [Gérer un flux de conversation avec des dialogues (.NET)](~/dotnet/bot-builder-dotnet-manage-conversation-flow.md)
- [Gérer un flux de conversation avec des dialogues (Node.js)](~/nodejs/bot-builder-nodejs-manage-conversation-flow.md)
- <a href="https://www.microsoft.com/cognitive-services/en-us/text-analytics-api" target="blank">API Analyse de texte</a>


::: moniker-end

