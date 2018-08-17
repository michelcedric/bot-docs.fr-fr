---
title: Configurer un bot pour qu’il s’exécute sur un ou plusieurs canaux | Microsoft Docs
description: Découvrez comment configurer un bot pour l’exécuter sur un ou plusieurs canaux à l’aide du portail Bot Framework.
keywords: canaux de bots, configurer, cortana, facebook messenger, kik, slack, Skype, portail azure
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: cb682bf77f801c98d00deffa0fc63249962248cd
ms.sourcegitcommit: dcbc8ad992a3e242a11ebcdf0ee99714d919a877
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/30/2018
ms.locfileid: "39352918"
---
# <a name="connect-a-bot-to-channels"></a>Connecter un bot à des canaux

Un canal est une connexion entre Bot Framework et les applications de communication. Vous configurez un bot pour le connecter aux canaux sur lesquels vous souhaitez le rendre disponible. Par exemple, un bot connecté au canal Skype peut être ajouté à une liste de contacts afin que les utilisateurs puissent interagir avec lui dans Skype. 

Les canaux incluent de nombreux services populaires tels que [Cortana](bot-service-channel-connect-cortana.md), [Facebook Messenger](bot-service-channel-connect-facebook.md), [Kik](bot-service-channel-connect-kik.md) et [Slack](bot-service-channel-connect-slack.md), entre autres. [Skype](https://dev.skype.com/bots) et Web Chat sont préconfigurés pour vous. 

La connexion aux canaux est rapide et facile dans le [portail Azure](https://portal.azure.com).

## <a name="get-started"></a>Prise en main

Pour la plupart des canaux, vous devez fournir les informations de configuration de canal afin d’exécuter votre bot sur le canal. La plupart des canaux nécessitent que votre bot dispose d’un compte sur le canal, tandis que d’autres, par exemple Facebook Messenger, nécessitent que votre bot utilise également une application inscrite avec le canal.

Pour configurer votre bot afin de le connecter à un canal, procédez comme suit :

1. Connectez-vous au <a href="https://portal.azure.com" target="_blank">Portail Azure</a>.
1. Sélectionnez le bot à configurer.
3. Dans le panneau Bot Service, cliquez sur **Canaux** sous **Gestion du bot**.
4. Cliquez sur l’icône du canal que vous souhaitez ajouter à votre bot.

![Se connecter aux canaux](~/media/channels/connect-to-channels.png)

Une fois que vous avez configuré le canal, les utilisateurs sur ce canal peuvent commencer à utiliser votre bot.

## <a name="publish-a-bot"></a>Publier un bot

Le processus de publication est différent pour chaque canal.

[!INCLUDE [publishing](~/includes/snippet-publish-to-channel.md)]

