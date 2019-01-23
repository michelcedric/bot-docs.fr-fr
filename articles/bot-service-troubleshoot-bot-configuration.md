---
title: Résoudre les problèmes de configuration de bot | Microsoft Docs
description: Comment résoudre les problèmes de configuration dans un bot déployé.
keywords: résoudre les problèmes, dépanner, configuration, web chat, problèmes.
author: jonathanFingold
ms.author: v-jofing
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/20/2018
ms.openlocfilehash: 8a3ff4a30e3041937ba831efc237343c9aa27e62
ms.sourcegitcommit: 8161753641368567f239e24a35ad61768acccd8e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/10/2019
ms.locfileid: "54202545"
---
# <a name="troubleshoot-bot-configuration-issues"></a>Résoudre les problèmes de configuration de bot

La première étape de la résolution des problèmes d’un bot consiste à tester celui-ci dans Web Chat. Vous pourrez ainsi déterminer si le problème est spécifique à votre bot (le bot ne fonctionne dans aucun canal) ou à un canal particulier (le bot fonctionne dans certains canaux mais pas dans d’autres).

## <a name="test-in-web-chat"></a>Tester dans la Discussion Web

1. Ouvrez votre ressource bot dans le [portail Azure](http://portal.azure.com/).
1. Ouvrez le volet **Tester dans Web Chat**.
1. Envoyez un message à votre bot.

![Tester dans Web Chat](./media/test-in-webchat.png)

Si le bot ne répond pas avec la sortie attendue, accédez à [Le bot ne fonctionne pas dans Web Chat](#bot-does-not-work-in-web-chat). Dans le cas contraire, accédez à [Le bot fonctionne dans Web Chat, mais pas dans les autres canaux](#bot-works-in-web-chat-but-not-in-other-channels).

## <a name="bot-does-not-work-in-web-chat"></a>Le bot ne fonctionne pas dans Web Chat

Les raisons pour lesquelles un bot ne fonctionne pas peuvent être nombreuses. La plus probable est que l’application de bot est en panne et ne peut pas recevoir de messages, ou que le bot reçoit les messages mais ne répond pas.

Pour voir si le bot est en cours d’exécution :

1. Ouvrez le volet **Vue d’ensemble**.
1. Copiez le **point de terminaison de messagerie** et collez-le dans votre navigateur.

Si le point de terminaison retourne l’erreur HTTP 405, cela signifie que le bot est accessible et qu’il est capable de répondre aux messages. Vous devez chercher à savoir si votre bot [expire](https://github.com/daveta/analytics/blob/master/troubleshooting_timeout.md) ou [échoue avec l’erreur HTTP 5xx](bot-service-troubleshoot-500-errors.md).

Si le point de terminaison retourne une erreur « Impossible d’atteindre ce site » ou « Impossible d’atteindre cette page », cela signifie que votre bot est en panne et que vous devez le redéployer.

## <a name="bot-works-in-web-chat-but-not-in-other-channels"></a>Le bot fonctionne dans Web Chat, mais pas dans les autres canaux

Si le bot fonctionne comme prévu dans Web Chat mais ne fonctionne pas dans certains autres canaux, les raisons possibles sont :

- [Problèmes de configuration de canal](#channel-configuration-issues)
- [Comportement spécifique au canal](#channel-specific-behavior)
- [Panne de canal](#channel-outage)

### <a name="channel-configuration-issues"></a>Problèmes de configuration de canal

Il est possible que les paramètres de configuration de canal aient été mal définis ou changés en externe. Par exemple, un bot a configuré le canal Facebook pour une page particulière, mais la page a été supprimée par la suite. La solution la plus simple est de supprimer le canal et de recommencer la configuration du canal.

Les liens ci-dessous fournissent des instructions pour configurer les canaux pris en charge par le Bot Framework :

- [Cortana](bot-service-channel-connect-cortana.md)
- [DirectLine](bot-service-channel-connect-directline.md)
- [E-mail](bot-service-channel-connect-email.md)
- [Facebook](bot-service-channel-connect-facebook.md)
- [GroupMe](bot-service-channel-connect-groupme.md)
- [Kik](bot-service-channel-connect-kik.md)
- [Microsoft Teams](https://docs.microsoft.com/microsoftteams/platform/concepts/bots/bots-overview)
- [Skype](bot-service-channel-connect-skype.md)
- [Skype Entreprise](bot-service-channel-connect-skypeforbusiness.md)
- [Slack](bot-service-channel-connect-slack.md)
- [Telegram](bot-service-channel-connect-telegram.md)
- [Twilio](bot-service-channel-connect-twilio.md)

### <a name="channel-specific-behavior"></a>Comportement spécifique au canal

L’implémentation de certaines fonctionnalités peut différer selon le canal. Par exemple, les canaux ne prennent pas tous en charge les cartes adaptatives. La plupart des canaux prennent en charge les boutons, mais les restituent selon leur propre mode d’affichage. Si vous voyez des différences de fonctionnement de certains types de messages dans les différents canaux, consultez [Channel Inspector](https://docs.botframework.com/channel-inspector/channels/Skype).

Voici quelques liens supplémentaires qui peuvent aider sur chacun des canaux :

- [Ajouter des bots à des applications Microsoft Teams](https://docs.microsoft.com/microsoftteams/platform/concepts/bots/bots-overview)
- [Facebook : Introduction à la plate-forme Messenger](https://developers.facebook.com/docs/messenger-platform/introduction)
- [Principes de conception des compétences Cortana](https://docs.microsoft.com/cortana/skills/design-principles)
- [Skype pour les développeurs](https://dev.skype.com/bots)
- [Slack : Activation des interactions avec les bots](https://api.slack.com/bot-users)

### <a name="channel-outage"></a>Panne de canal

Parfois, certains canaux peuvent connaître une interruption de service. En règle générale, ces pannes ne durent pas longtemps. Toutefois, si vous soupçonnez une panne, consultez le réseau social ou le site web du canal.

Une autre façon de déterminer si un canal subit une panne consiste à créer un bot de test (par exemple, un bot Écho simple) et à ajouter un canal. Si le bot de test fonctionne avec certains canaux mais pas avec d’autres, cela indique que le problème n’est pas dans votre bot de production.
