---
title: Connecter un bot à Twilio | Microsoft Docs
description: Découvrez comment configurer la connexion d’un bot à Twilio.
keywords: Twilio, canaux de bot, SMS, application, téléphone, configurer Twilio, communication cloud, texte
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 10/9/2018
ms.openlocfilehash: 7d7416940ccad4e62c98f4a386dac43189301b56
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2018
ms.locfileid: "49998302"
---
# <a name="connect-a-bot-to-twilio"></a>Connecter un bot à Twilio

Vous pouvez configurer votre bot pour communiquer avec des personnes utilisant la plateforme de communication cloud Twilio.

## <a name="log-in-to-or-create-a-twilio-account-for-sending-and-receiving-sms-messages"></a>Se connecter à un compte Twilio ou créer un compte Twilio pour envoyer et recevoir des messages SMS

Si vous n’avez pas de compte Twilio, <a href="https://www.twilio.com/try-twilio" target="_blank">créez-en un</a>.

## <a name="create-a-twiml-application"></a>Créer une application TwiML

<a href="https://support.twilio.com/hc/en-us/articles/223180928-How-Do-I-Create-a-TwiML-App-" target="_blank">Créer une application TwiML</a> en suivant les instructions fournies.

![Créer une application](~/media/channels/twi-StepTwiml.png)

Sous **Properties** (Propriétés), entrez un **NOM CONVIVIAL**. Dans ce didacticiel, nous utilisons « Mon application TwiML » comme exemple. L’**URL DE DEMANDE** sous Voice (Voix) peut être laissée vide. Sous **Messaging** (Messagerie), l’**URL de demande** doit être `https://sms.botframework.com/api/sms`.

## <a name="select-or-add-a-phone-number"></a>Sélectionner ou ajouter un numéro de téléphone

Suivez les instructions indiquées <a href = "https://support.twilio.com/hc/en-us/articles/223180048-Adding-a-Verified-Phone-Number-or-Caller-ID-with-Twilio" target="_blank">ici</a> pour ajouter un ID d’appelant vérifié via le site de la console. Une fois que vous avez terminé, vous verrez votre numéro vérifié dans **Active Numbers** (Numéros actifs) sous **Manage Numbers** (Gestion des numéros).

![Définir un numéro de téléphone](~/media/channels/twi-StepPhone.png)

## <a name="specify-application-to-use-for-voice-and-messaging"></a>Spécifier l’application à utiliser pour la voix et la messagerie

Cliquez sur le numéro et accédez à la configuration (**Configure**). Sous Voice (Voix) et Messaging (Messagerie), définissez les options **CONFIGURE WITH** (Configurer avec) sur l’application TwiML et **TWIML APP** sur My TwiML app (Mon application TwiML). Une fois que vous avez terminé, cliquez sur **Save** (Enregistrer).

![Spécifier l’application](~/media/channels/twi-StepPhone2.png)

Revenez à la page de gestion des numéros (**Manage Numbers**), et vous verrez que la voix et de la messagerie ont bien été définies sur l’application TwiML.

![Numéro spécifié](~/media/channels/twi-StepPhone3.png)


## <a name="gather-credentials"></a>Collecter les informations d’identification

Revenez à la [page d’accueil de la console](https://www.twilio.com/console/), vous verrez votre SID de compte et un jeton d’authentification sur le tableau de bord du projet, comme indiqué ci-dessous.

![Collecter les informations d’identification de l’application](~/media/channels/twi-StepAuth.png)

## <a name="submit-credentials"></a>Envoyer les informations d’identification

Dans une fenêtre distincte, revenez sur le site du Bot Framework à l’adresse https://dev.botframework.com/. 

- Sélectionnez **My bots** (Mes bots) et choisissez le bot que vous souhaitez connecter à Twilio. Ceci vous dirigera vers le portail Azure.
- Sélectionnez **Canaux** sous **Gestion du bot**. Cliquez sur l’icône de Twilio (SMS).
- Entrez le numéro de téléphone, le SID de compte et le jeton d’authentification enregistrés précédemment. Une fois que vous avez terminé, cliquez sur **Save** (Enregistrer).

![Envoyer les informations d’identification](~/media/channels/twi-StepSubmit.png)

Une fois ces étapes effectuées, votre bot est correctement configuré pour communiquer avec les utilisateurs de Twilio.