---
title: Connecter un bot à Facebook Messenger | Microsoft Docs
description: Découvrez comment configurer la connexion d’un bot à Facebook Messenger.
keywords: Facebook Messenger, bot de canal, application Facebook, ID d’application, clé secrète d’application, bot Facebook, informations d’identification
author: RobStand
ms.author: RobStand
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 09/16/2018
ms.openlocfilehash: 60e39bb652ab5b7ffeeb5ba53bdf4c82f936553e
ms.sourcegitcommit: 3bf3dbb1a440b3d83e58499c6a2ac116fe04b2f6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/23/2018
ms.locfileid: "46707865"
---
# <a name="connect-a-bot-to-facebook-messenger"></a>Connecter un bot à Facebook Messenger

Pour en savoir plus sur le développement pour Facebook Messenger, consultez la [documentation de la plate-forme Messenger](https://developers.facebook.com/docs/messenger-platform). Si vous le souhaitez consultez les [directives de prélancement](https://developers.facebook.com/docs/messenger-platform/product-overview/launch#app_public), le [guide de démarrage rapide](https://developers.facebook.com/docs/messenger-platform/guides/quick-start) et le [guide de configuration](https://developers.facebook.com/docs/messenger-platform/guides/setup) Facebook.

Pour configurer un bot de sorte qu’il communique à l’aide de Facebook Messenger, activez Facebook Messenger sur une page Facebook, puis connectez le bot à l’application.

> [!NOTE]
> L’interface utilisateur de Facebook peut être légèrement différente selon la version que vous utilisez.

## <a name="copy-the-page-id"></a>Copier l’ID de la page

Le bot est accessible par le biais d’une page Facebook. [Créez une page Facebook](https://www.facebook.com/bookmarks/pages) ou accédez à une page existante.

* Ouvrez la page **About** (À propos de) de Facebook, puis copiez et enregistrez la valeur **Page ID** (ID de la page).

## <a name="create-a-facebook-app"></a>Créer une application Facebook

[Créez une application Facebook](https://developers.facebook.com/quickstarts/?platform=web) sur la page et générez un ID d’application et une clé secrète d’application.

![Créer un ID d’application](~/media/channels/FB-CreateAppId.png)

* Copiez et enregistrez les valeurs **App ID** (ID d’application) et **App Secret** (Clé secrète d’application).

![Enregistrer l’ID et la clé secrète d’application](~/media/channels/FB-get-appid.png)

Positionnez le curseur « Allow API Access to App Settings » (Autoriser l’accès de l’API aux paramètres d’application) sur « Yes » (Oui).

![Paramètres de l’application](~/media/bot-service-channel-connect-facebook/api_settings.png)

## <a name="enable-messenger"></a>Activer Messenger


Activez Facebook Messenger dans la nouvelle application Facebook.

* Sur la page **Product Setup** (Configuration du produit) de l’application, cliquez sur **Get Started** (Prise en main), puis cliquez de nouveau sur **Get Started**.


![Activer Messenger](~/media/channels/FB-AddMessaging1.png)

## <a name="generate-a-page-access-token"></a>Générer un jeton d’accès à la page

Dans le volet **Token Generation** (Génération de jeton) de la section Messenger, sélectionnez la page cible. Un jeton d’accès à la page est généré.

* Copiez et enregistrez le jeton sous **Page Access Token** (Jeton d’accès à la page).

![Générer un jeton](~/media/channels/FB-generateToken.png)

## <a name="enable-webhooks"></a>Activer les webhooks

Cliquez sur **Set up Webhooks** (Configurer des webhooks) pour transférer les événements de messagerie de Facebook Messenger vers le bot.

![Activer le webhook](~/media/channels/FB-webhook.png)

## <a name="provide-webhook-callback-url-and-verify-token"></a>Fournir l’URL de rappel du webhook et vérifier le jeton

Dans le [Portail Azure](https://portal.azure.com/), ouvrez le bot, cliquez sur l’onglet **Canaux**, puis cliquez sur **Facebook Messenger**.

* Copiez les valeurs sous **URL de rappel** et **Vérifier le jeton** dans le portail.

![Copier les valeurs](~/media/channels/fb-callbackVerify.png)

1. Revenez à Facebook Messenger et collez les valeurs sous **Callback URL** (URL de rappel) et **Verify Token** (Vérifier le jeton).

2. Sous **Subscription Fields** (Champs d’abonnement), sélectionnez *message\_deliveries* (remise des messages), *messages*, *messaging\_options* (options de messagerie) et *messaging\_postbacks* (publications de messagerie).

3. Cliquez sur **Verify and Save** (Vérifier et enregistrer).

![Configurer le webhook](~/media/channels/FB-webhookConfig.png)

4. Abonnez le webhook à la page Facebook.

![Abonner le webhook](~/media/bot-service-channel-connect-facebook/subscribe-webhook.png)


## <a name="provide-facebook-credentials"></a>Fournir les informations d’identification Facebook

Dans le Portail Azure, collez les valeurs des champs **Facebook App ID** (ID d’application Facebook), **Facebook App Secret** (Secret d’application Facebook), **ID de la page** et **Page Access Token** (Jeton d’accès à la page) que vous avez précédemment copiées à partir de Facebook Messenger. Vous pouvez utiliser le même bot sur plusieurs pages Facebook en ajoutant des ID de page et jetons d’accès supplémentaires.

![Entrer les informations d’identification](~/media/channels/fb-credentials2.png)

## <a name="submit-for-review"></a>Envoyer pour vérification

Facebook requiert une URL de politique de confidentialité et une URL de conditions d’utilisation sur sa page de paramètres d’application de base. La page [Code of Conduct](https://aka.ms/bf-conduct) (Code de conduite) contient des liens vers les ressources tierces, aidant à créer une politique de confidentialité. La page [Terms of Use](https://aka.ms/bf-terms) (Conditions d’utilisation) contient des exemples de conditions aidant à créer un document de conditions d’utilisation approprié.

Une fois le bot terminé, Facebook applique son propre [processus de vérification](https://developers.facebook.com/docs/messenger-platform/app-review) pour les applications publiées sur Messenger. Facebook teste le bot pour vérifier qu’il est conforme aux [politiques de la plate-forme](https://developers.facebook.com/docs/messenger-platform/policy-overview) Facebook.

## <a name="make-the-app-public-and-publish-the-page"></a>Rendre l’application publique et publier la page

> [!NOTE]
> Une application reste en [mode de développement](https://developers.facebook.com/docs/apps/managing-development-cycle) jusqu’à sa publication. Les fonctionnalités de plug-in et d’API fonctionnent uniquement pour les administrateurs, les développeurs et les testeurs.

Une fois la vérification effectuée, dans le tableau de bord de l’application, sous App Review (Vérification de l’application), définissez l’application sur Publique.
Assurez-vous que la page Facebook associée à ce bot est publiée. L’état s’affiche dans les paramètres des pages.

> [!NOTE]
> Vous pouvez également utiliser la plateforme Facebook Workplace. Pour l’activer, créez une [intégration personnalisée](https://developers.facebook.com/docs/workplace/custom-integrations-new) pour votre Workplace et utilisez l’ID d’application, le secret d’application et le jeton d’accès correspondants. Au lieu d’un ID de page traditionnel, utilisez les chiffres qui suivent le nom d’intégration sur la page À propos de. Les Webhooks peuvent être connectés à l’aide des informations d’identification figurant dans Azure.
