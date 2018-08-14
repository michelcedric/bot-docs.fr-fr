---
title: Connecter un bot à Twilio | Microsoft Docs
description: Découvrez comment configurer la connexion d’un bot à Twilio.
keywords: Twilio, canaux de bot, SMS, application, téléphone, configurer Twilio, communication cloud, texte
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: 7e09126d50cfbebfc0aad0ee7fcb71b4e7551a7d
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39298904"
---
# <a name="connect-a-bot-to-twilio"></a>Connecter un bot à Twilio

Vous pouvez configurer votre bot pour communiquer avec des personnes utilisant la plateforme de communication cloud Twilio.

## <a name="log-in-to-or-create-a-twilio-account-for-sending-and-receiving-sms-messages"></a>Se connecter à un compte Twilio ou créer un compte Twilio pour envoyer et recevoir des messages SMS

Si vous n’avez pas de compte Twilio, <a href="https://www.twilio.com/try-twilio" target="_blank">créez-en un</a>.

## <a name="create-a-twiml-application"></a>Créer une application TwiML

<a href="https://www.twilio.com/user/account/messaging/dev-tools/twiml-apps/add" target="_blank">Créer une application TwiML</a>

![Créer une application](~/media/channels/twi-StepTwiml.png)

 Sous Messaging (Messagerie), l’URL de demande doit être **https://sms.botframework.com/api/sms**.

## <a name="select-or-add-a-phone-number"></a>Sélectionner ou ajouter un numéro de téléphone

<a href="https://www.twilio.com/user/account/phone-numbers/incoming" target="_blank">Sélectionnez ou ajoutez un numéro de téléphone</a>. Cliquez sur le numéro pour l’ajouter à l’application TwiML que vous avez créée.

![Définir un numéro de téléphone](~/media/channels/twi-StepPhone.png)

## <a name="specify-application-to-use-for-messaging"></a>Spécifier l’application à utiliser pour la messagerie
Dans la section **Messaging** (Messagerie), indiquez sous **TwiML App** (Application TwiML) le nom de l’application TwiML que vous venez de créer.
Copiez la valeur du champ **Phone number** (Numéro de téléphone) pour une utilisation ultérieure.

![Spécifier l’application](~/media/channels/twi-StepPhone2.png)

## <a name="gather-credentials"></a>Collecter les informations d’identification

<a href="https://www.twilio.com/user/account/settings" target="_blank">Collectez les informations d’identification</a>, puis cliquez sur l’icône en forme d’œil pour afficher le jeton d’authentification.

![Collecter les informations d’identification de l’application](~/media/channels/twi-StepAuth.png)

## <a name="submit-credentials"></a>Envoyer les informations d’identification

Entrez le numéro de téléphone, l’accountSID et le jeton d’authentification que vous avez copiés précédemment, puis cliquez sur **Submit Twilio Credentials** (Envoyer les informations d’identification Twilio).

## <a name="enable-the-bot"></a>Activer le bot
Cochez la case **Enable this bot on SMS** (Activer ce bot sur SMS). Puis, cliquez sur **I’m done configuring SMS** (J’ai terminé la configuration de SMS).

Une fois ces étapes effectuées, votre bot est correctement configuré pour communiquer avec les utilisateurs de Twilio.

