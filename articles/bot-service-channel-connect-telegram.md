---
title: Créer un bot pour Telegram | Microsoft Docs
description: Découvrez comment configurer la connexion d’un bot à Telegram.
keywords: configurer un bot, Telegram, canal de bot, bot de Telegram, jeton d’accès
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
ms.openlocfilehash: efe38392117fb871b2b98e3f1d8d798bfaef0c41
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2018
ms.locfileid: "49996506"
---
# <a name="connect-a-bot-to-telegram"></a>Connecter un bot à Telegram

Vous pouvez configurer votre bot pour communiquer avec des personnes utilisant l’application de messagerie Telegram.

[!INCLUDE [Channel Inspector intro](~/includes/snippet-channel-inspector.md)]

## <a name="visit-the-bot-father-to-create-a-new-telegram-bot"></a>Visitez Bot Father pour créer un nouveau bot Telegram

<a href="https://telegram.me/botfather" target="_blank">Créer un bot Telegram</a> avec Bot Father.

![Visitez Bot Father](~/media/channels/tg-StepVisitBotFather.png)

## <a name="create-a-new-telegram-bot"></a>Créez un bot Telegram
Pour créer un bot Telegram, envoyez la commande `/newbot`.

![Créez un bot](~/media/channels/tg-StepNewBot.png)

### <a name="specify-a-friendly-name"></a>Donnez-lui un nom convivial

Donnez un nom convivial au bot Telegram.

![Donnez un nom convivial au bot](~/media/channels/tg-StepNameBot.png)

### <a name="specify-a-username"></a>Spécifiez un nom d’utilisateur

Donnez un nom d’utilisateur unique au bot Telegram.

![Donnez un nom unique au bot](~/media/channels/tg-StepUsername.png)

### <a name="copy-the-access-token"></a>Copiez le jeton d’accès

Copiez le jeton d’accès au bot Telegram.

![Copiez le jeton d'accès](~/media/channels/tg-StepBotCreated.png)

## <a name="enter-the-telegram-bots-access-token"></a>Entrez le jeton d'accès au bot Telegram

Collez le jeton que vous avez copié précédemment dans le champ **Jeton d’accès** et cliquez sur **Envoyer**.

## <a name="enable-the-bot"></a>Activer le bot
Cochez **Enable this bot on Telegram** (Activer ce robot sur Telegram). Cliquez ensuite sur **I'm done configuring Telegram** (J’ai terminé la configuration de Telegram).

Une fois ces étapes effectuées, votre bot est correctement configuré pour communiquer avec les utilisateurs dans Telegram.
