---
title: Connecter un bot à LINE | Microsoft Docs
description: Découvrez comment configurer la connexion d’un bot à LINE.
keywords: connecter un bot, canal de bot, bot LINE, informations d’identification, configurer, téléphone
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 1/7/2019
ms.openlocfilehash: f6ed40c5ad3999ea5a123bf051de849917f22b42
ms.sourcegitcommit: e41dabe407fdd7e6b1d6b6bf19bef5f7aae36e61
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 02/27/2019
ms.locfileid: "56893509"
---
# <a name="connect-a-bot-to-line"></a>Connecter un bot à LINE

Vous pouvez configurer votre bot pour communiquer avec des personnes via l’application LINE.

## <a name="log-into-the-line-console"></a>Se connecter à la console LINE

Connectez-vous à la [console développeur LINE](https://developers.line.biz/console/register/messaging-api/provider/) de votre compte LINE, en utilisant *Log in with Line*. 

> [!NOTE]
> Si vous ne l’avez pas déjà fait, [téléchargez LINE](https://line.me/), puis accédez à vos paramètres pour inscrire votre adresse e-mail.

### <a name="register-as-a-developer"></a>S’inscrire en tant que développeur

Si vous accédez pour la première fois à la console développeur de LINE, entrez votre nom et votre adresse e-mail pour créer un compte de développeur.

![Capture d’écran de LINE - Inscrire un développeur](./media/channels/LINE-screenshot-1.png)

## <a name="create-a-new-provider"></a>Créer un fournisseur

Créez d’abord un fournisseur pour votre bot si vous n’en avez pas déjà configuré un. Le fournisseur est l’entité (une personne ou une entreprise) qui offre votre application.

![Capture d’écran de LINE - Créer un fournisseur](./media/channels/LINE-screenshot-2.png)

## <a name="create-a-messaging-api-channel"></a>Créer un canal d’API de messagerie

Ensuite, créez un canal d’API de messagerie. 

![Capture d’écran de LINE - Type de canal](./media/channels/LINE-channel-type-selection.png)

Créez un canal d’API de messagerie en cliquant sur le carré vert.

![Capture d’écran de LINE - Créer un canal](./media/channels/LINE-create-channel.png)

Le nom ne peut pas inclure « LINE » ni une chaîne similaire. Remplissez les champs nécessaires et confirmez les paramètres de votre canal.

![Capture d’écran de LINE - Paramètres du canal](./media/channels/LINE-screenshot-4.png)

## <a name="get-necessary-values-from-your-channel-settings"></a>Obtenir les valeurs nécessaires des paramètres de votre canal

Une fois que vous avez confirmé les paramètres de votre canal, vous êtes dirigé vers une page similaire à celle-ci.

![Capture d’écran de LINE - Page du canal](./media/channels/LINE-screenshot-5.png)

Cliquez sur le canal que vous avez créé pour accéder aux paramètres de votre canal et faites défiler vers le bas jusqu’à trouver **Basic information > Channel secret**. Notez-les quelque part pour pouvoir les réutiliser. Vérifiez que les fonctionnalités disponibles incluent `PUSH_MESSAGE`.

![Capture d’écran de LINE - Secret du canal](./media/channels/LINE-screenshot-6.png)

Ensuite, faites défiler plus loin jusqu’à **Messaging settings** (Paramètres de messagerie). Là, vous voyez un champ **Channel access token** (Jeton d’accès du canal), avec un bouton *Issue* (Émettre). Cliquez sur ce bouton pour obtenir votre jeton d’accès et notez également celui-ci quelque part.

![Capture d’écran de LINE - Jeton du canal](./media/channels/LINE-screenshot-8.png)

## <a name="connect-your-line-channel-to-your-azure-bot"></a>Connecter votre canal LINE à votre bot Azure

Connectez-vous au [portail Azure](https://portal.azure.com/), recherchez votre bot et cliquez sur **Canaux**. 

![Capture d’écran de LINE - Paramètres Azure](./media/channels/LINE-channel-setting-2.png)

Là, sélectionnez le canal LINE et collez le secret et le jeton d’accès du canal que vous avez notés plus haut dans les champs appropriés. Veillez à enregistrer vos changements.

Copiez l’URL du webhook personnalisé que vous donne Azure.

![Capture d’écran de LINE - Paramètres Azure](./media/channels/LINE-channel-setting-1.png)

## <a name="configure-line-webhook-settings"></a>Configurer les paramètres du webhook LINE

Ensuite, revenez à la console développeur de LINE et collez l’URL du webhook d’Azure dans **Message settings > Webhook URL**, (Paramètres des messages > URL du webhook), puis cliquez sur **Verify** pour vérifier la connexion. Si vous venez de créer le canal dans Azure, quelques minutes peuvent être nécessaires pour qu’il soit pris en compte.

Ensuite, activez **Message settings > Use webhooks** (Paramètres des messages > Utiliser des webhooks).

> [!IMPORTANT]
> Dans la console développeur de LINE, vous devez d’abord définir l’URL du webhook, puis ensuite définir **Use webhooks = Enabled**. Si vous commencez par activer les webhooks avec une URL vide, l’état Activé n’est pas défini, même si l’interface utilisateur peut indiquer le contraire.

Après avoir ajouté une URL de webhook, puis activé les webhooks, veillez à recharger cette page et vérifiez que ces changements ont été correctement définis.

![Capture d’écran de LINE - Webhooks](./media/channels/LINE-screenshot-9.png)

## <a name="test-your-bot"></a>Tester votre robot

Une fois ces étapes effectuées, votre bot est correctement configuré pour communiquer avec les utilisateurs sur LINE et est prêt à être testé.

### <a name="add-your-bot-to-your-line-mobile-app"></a>Ajouter votre bot à votre application mobile LINE

Dans la console développeur de LINE, accédez à la page des paramètres : vous y voyez un code QR de votre bot. 

Dans l’application mobile LINE, accédez à l’onglet de navigation le plus à droite avec les trois points [**...**] et appuyez sur l’icône Code QR. 

![Capture d’écran de LINE - Application mobile](./media/channels/LINE-screenshot-12.jpg)

Pointez le lecteur de code QR sur le code QR dans votre console développeur. Vous devez maintenant être en mesure d’interagir avec votre bot dans votre application mobile LINE et de tester votre bot.

### <a name="automatic-messages"></a>Messages automatiques

Quand vous commencez à tester votre bot, vous pouvez noter que le bot envoie des messages inattendus qui ne sont pas ceux que vous avez spécifiés dans l’activité `conversationUpdate`.  Votre dialogue peut se présenter comme ceci :

![Capture d’écran de LINE - Conversation](./media/channels/LINE-screenshot-conversation.jpg)

Pour éviter l’envoi de ces messages, vous devez désactiver les messages de réponse automatique.

![Capture d’écran de LINE - Réponse automatique](./media/channels/LINE-screenshot-10.png)

Vous pouvez également choisir de conserver ces messages. Dans ce cas, il peut être judicieux de cliquer sur « Set message » (Définir le message) et de le modifier.

![Capture d’écran de LINE - Définir la réponse automatique](./media/channels/LINE-screenshot-11.png)

## <a name="troubleshooting"></a>Résolution de problèmes

* Dans le cas où votre bot ne répond pas à aucun de vos messages, accédez à votre bot dans le portail Azure, puis choisissez Tester dans Web Chat.  
    * Si le bot fonctionne ici mais qu’il ne répond pas dans LINE, rechargez votre page Console développeur de LINE et répétez les instructions ci-dessus relatives au webhook. Veillez à définir l’**URL du webhook** avant d’activer les webhooks.
    * Si le bot ne fonctionne pas dans Web Chat, déboguez le problème pour votre bot, puis revenez et terminez la configuration de votre canal LINE.

