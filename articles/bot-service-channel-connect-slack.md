---
title: Connecter un bot à Slack | Microsoft Docs
description: Découvrez comment configurer la connexion d’un bot à Slack.
keywords: connecter un bot, canal de bot, bot Slack, application de messagerie Slack
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: 56fe0af4d34e6e0aa4bc420112c541a410aa1301
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39299212"
---
# <a name="connect-a-bot-to-slack"></a>Connecter un bot à Slack

Vous pouvez configurer votre bot pour communiquer avec des personnes utilisant l’application de messagerie Slack.

## <a name="create-a-slack-application-for-your-bot"></a>Créer une application Slack pour votre bot

Connectez-vous à Slack et [créez une application Slack](https://api.slack.com/applications/new).

![Configurer le bot](~/media/channels/slack-NewApp.png)

## <a name="create-an-app-and-assign-a-development-slack-team"></a>Créer une application et assigner une équipe Slack de développement

Entrez un nom d’application et sélectionnez une équipe Slack de développement. Si vous n’êtes pas déjà membre d’une équipe Slack de développement, [créez-en une, ou rejoignez-en une](https://slack.com/).

![Créer une application](~/media/channels/slack-CreateApp.png)

Cliquez sur **Créer une application**. Slack crée votre application et génère un ID client et une clé secrète client.

## <a name="add-a-new-redirect-url"></a>Ajouter une nouvelle URL de redirection

Vous devez ensuite ajouter une nouvelle URL de redirection.

1. Sélectionnez l’onglet **OAuth & Permissions** (OAuth et autorisations).
2. Cliquez sur **Add a new Redirect URL** (Ajouter une nouvelle URL de redirection).
3. Entrez https://slack.botframework.com.
4. Cliquez sur **Add**.
5. Cliquez sur **Save URLs** (Enregistrer les URL).

![Ajouter une URL de redirection](~/media/channels/slack-RedirectURL.png)

## <a name="create-a-slack-bot-user"></a>Créer un utilisateur de bot Slack

L’ajout d’un utilisateur de bot vous permet d’assigner un nom d’utilisateur à votre bot et de choisir s’il doit toujours apparaître comme étant en ligne.

1. Cliquez sur l’onglet **Bot Users** (Utilisateurs de bot).
2. Cliquez sur **Add a Bot User** (Ajouter un utilisateur de bot).

![Créer un bot](~/media/channels/slack-CreateBot.png)

Cliquez sur **Add Bot User** (Ajouter un utilisateur de bot) pour valider vos paramètres. Définissez l’option **Always Show My Bot as Online** (Toujours afficher mon bot comme étant en ligne)sur **On** (Activé), puis cliquez sur **Save Changes** (Enregistrer les modifications).

![Créer un bot](~/media/channels/slack-CreateApp-AddBotUser.png)

## <a name="subscribe-to-bot-events"></a>S’abonner aux événements de bot

Suivez ces étapes pour vous abonner à six événements de bot spécifiques. Lorsque vous vous abonnez à des événements de bot, votre application est informée des activités de l’utilisateur à l’URL que vous spécifiez.

> [!TIP]
> Votre descripteur de bot est une propriété de votre bot. Pour rechercher le descripteur d’un bot, consultez la page [https://dev.botframework.com/bots](https://dev.botframework.com/bots), choisissez un bot, puis cliquez sur **SETTINGS** (Paramètres).

1. Sélectionnez l’onglet **Event Subscriptions** (Abonnements à des événements).
2. Définissez l’option **Enable Events** (Activer des événements) sur **On** (Activé).
3. Sous **Request URL** (URL de demande), entrez cette URL, mais remplacez `{YourBotHandle}` par le descripteur de votre bot.
        `https://slack.botframework.com/api/Events/{YourBotHandle}`
4. Sous **Subscribe to Bot Events** (S’abonner à des événements de bot), cliquez sur **Add Bot User Event** (Ajouter un événement d’utilisateur de bot).
5. Dans la liste des événements, cliquez sur **Add Bot User Event** (Ajouter un événement d’utilisateur de bot), puis sélectionnez ces six types d’événements :
    * `member_joined_channel`
    * `member_left_channel`
    * `message.channels`
    * `message.groups`
    * `message.im`
    * `message.mpim`
6. Cliquez sur **Enregistrer les modifications**.

![S’abonner aux événements](~/media/channels/slack-EnableEvents.png)

## <a name="add-and-configure-interactive-messages-optional"></a>Ajouter et configurer des messages interactifs (facultatif)

Si votre bot doit utiliser des fonctionnalités spécifiques à Slack comme des boutons, procédez comme suit :

1. Sélectionnez l’onglet **Interactive Components** (Composants interactifs) et cliquez sur **Enable Interactive Components** (Activer les composants interactifs).
2. Entrez `https://slack.botframework.com/api/Actions` sous **Request URL** (URL de demande).
3. Cliquez sur le bouton **Enable Interactive Messages** (Activer les messages interactifs), puis cliquez sur le bouton **Save changes** (Enregistrer les modifications).

![Activer les messages](~/media/channels/slack-MessageURL.png)

## <a name="gather-credentials"></a>Collecter les informations d’identification

Sélectionnez l’onglet **Basic Information** (Informations de base) et faites défiler jusqu’à la section **App Credentials** (Informations d’identification de l’application).
L’ID Client, la clé secrète client et le jeton de vérification requis pour la configuration de votre bot Slack sont affichés.

![Collecter les informations d’identification](~/media/channels/slack-AppCredentials.png)

## <a name="submit-credentials"></a>Envoyer les informations d’identification

Dans une fenêtre de navigateur distincte, revenez sur le site du Bot Framework à l’adresse `https://dev.botframework.com/`.

1. Sélectionnez **My bots** (Mes bots) et choisissez le bot que vous souhaitez connecter à Slack.
2. Dans la section **Add a channel** (Ajouter un canal), cliquez sur l’icône de Slack.
3. Dans la section **Enter your Slack credentials** (Entrez vos informations d’identification Slack), collez les informations d’identification de l’application issues du site web Slack dans les champs appropriés.
4. Le champ **Landing Page URL** (URL de la page d’accueil) est facultatif. Vous pouvez l’omettre ou le modifier.
5. Cliquez sur **Enregistrer**.

![Envoyer les informations d’identification](~/media/channels/slack-SubmitCredentials.png)

Suivez les instructions fournies pour permettre à votre équipe Slack de développement d’accéder à votre application Slack.

## <a name="enable-the-bot"></a>Activer le bot

Dans la page Configure Slack (Configurer Slack), vérifiez que le curseur en regard du bouton Enregistrer est bien positionné sur **Activé**.
Votre bot est configuré pour communiquer avec des utilisateurs dans Slack.

## <a name="create-an-add-to-slack-button"></a>Créer un bouton Add to Slack (Ajouter à Slack)

Slack fournit du code HTML que vous pouvez utiliser pour aider les utilisateurs de Slack à trouver votre bot dans la section *Add the Slack button* (Ajouter le bouton Slack) de [cette page](https://api.slack.com/docs/slack-button).
Pour utiliser ce code HTML avec votre bot, remplacez la valeur href (qui commence par `https://`) par l’URL figurant dans les paramètres de canal Slack de votre bot.
Suivez ces étapes pour obtenir l’URL de remplacement.

1. À la page [https://dev.botframework.com/bots](https://dev.botframework.com/bots), cliquez sur votre bot.
2. Cliquez sur **CHANNELS** (Canaux), cliquez avec le bouton droit sur l’entrée **Slack**, puis cliquez sur **Copier le lien**. Cette URL est désormais dans le Presse-papiers.
3. Collez cette URL à partir de votre Presse-papiers dans le code HTML fourni pour le bouton Slack. Cette URL remplace la valeur href fournie par Slack pour ce bot.

Les utilisateurs autorisés peuvent cliquer sur le bouton **Add to Slack** (Ajouter à Slack) fourni par ce code HTML pour trouver votre bot sur Slack.
