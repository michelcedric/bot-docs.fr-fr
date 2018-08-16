---
title: Canaux et le service Bot Connector | Microsoft Docs
description: Concepts clés du SDK Bot Builder.
keywords: activités, conversation
author: jonathanfingold
ms.author: jonathanfingold
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 03/28/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 99c9267fe41734a41243461fafa98c7116162984
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39300356"
---
# <a name="channels-and-the-bot-connector-service"></a>Canaux et le service Bot Connector

Les canaux sont le point de terminaison, tel que Facebook, Skype, Email ou Slack, à partir duquel un utilisateur se connecte à notre bot. Le service Bot Connector, configuré via le [portail Azure](https://portal.azure.com), connecte votre bot à ces canaux et facilite la communication entre votre bot et l’utilisateur. 

En plus des canaux standard fournis avec le service Bot Connector, vous pouvez utiliser [Direct Line](bot-builder-howto-direct-line.md) comme canal pour connecter votre bot à votre propre application cliente.

Le service Bot Connector vous permet de développer votre bot indépendamment du canal en normalisant les messages envoyés par le bot à un canal. Cela implique sa conversion depuis le schéma Bot Builder vers le schéma du canal. Toutefois, si le canal ne prend pas en charge tous les aspects du schéma Bot Builder, le service tente de convertir le message dans un format que le canal prend en charge. Par exemple, si le bot envoie au canal SMS un message qui contient une carte avec des boutons d’action, le connecteur peut envoyer la carte en tant qu’image et inclure les actions sous forme de liens dans le texte du message.

## <a name="activities-and-conversations"></a>Activités et conversations


Le service Bot Connector utilise JSON pour échanger des informations entre le bot et l’utilisateur, tandis que le SDK Bot Builder wrappe ces informations dans un objet d’activité spécifique au langage. Parmi les [types d’activités](../bot-service-activities-entities.md) évoqués dans la présentation de [l’interaction avec votre bot](bot-builder-basics.md#interaction-with-your-bot), celui le plus courant est un message, mais les autres sont également importants. Ceux-ci incluent la mise à jour d’une conversation, la mise à jour de la relation avec un contact, la suppression de données utilisateur, la fin de la conversation, la saisie, la réaction à un message et quelques autres activités spécifiques au bot que l’utilisateur est très peu susceptible de voir. Pour plus d’informations sur chacun d’entre eux et les moments auxquels ils se produisent, vous pouvez consulter le contenu de référence des activités.

Chaque tour et son activité associé appartiennent à une **conversation** logique, qui représente une interaction entre un ou plusieurs bots et un utilisateur spécifique ou un groupe d’utilisateurs. Une conversation est spécifique à un canal et a un ID unique pour ce canal.

## <a name="next-steps"></a>Étapes suivantes

Certains concepts clés d’un bot n’ayant plus de secret pour vous, plongeons-nous dans les différentes formes de conversation que notre bot peut utiliser.

> [!div class="nextstepaction"]
> [Formes de conversation](bot-builder-conversations.md)
