---
title: Connecter un bot à Facebook Messenger | Microsoft Docs
description: Découvrez comment configurer la connexion d’un bot à Facebook Messenger.
keywords: Facebook Messenger, bot de canal, application Facebook, ID d’application, clé secrète d’application, bot Facebook, informations d’identification
author: RobStand
ms.author: RobStand
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 10/12/2018
ms.openlocfilehash: 1424a1929afa7ab9bec48e038fc93a2f8262241a
ms.sourcegitcommit: 54ed5000c67a5b59e23b667547565dd96c7302f9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/13/2018
ms.locfileid: "49315195"
---
# <a name="connect-a-bot-to-facebook"></a>Connecter un bot à Facebook

Votre bot peut être connecté à Facebook Messenger et à Facebook Workplace, afin qu’il puisse communiquer avec les utilisateurs de ces deux plateformes. Le didacticiel suivant montre comment connecter un bot à ces deux canaux étape par étape.

> [!NOTE]
> L’interface utilisateur de Facebook peut être légèrement différente selon la version que vous utilisez.

## <a name="connect-a-bot-to-facebook-messenger"></a>Connecter un bot à Facebook Messenger

Pour en savoir plus sur le développement pour Facebook Messenger, consultez la [documentation de la plate-forme Messenger](https://developers.facebook.com/docs/messenger-platform). Si vous le souhaitez consultez les [directives de prélancement](https://developers.facebook.com/docs/messenger-platform/product-overview/launch#app_public), le [guide de démarrage rapide](https://developers.facebook.com/docs/messenger-platform/guides/quick-start) et le [guide de configuration](https://developers.facebook.com/docs/messenger-platform/guides/setup) Facebook.

Pour configurer un bot de sorte qu’il communique à l’aide de Facebook Messenger, activez Facebook Messenger sur une page Facebook, puis connectez le bot à l’application.

### <a name="copy-the-page-id"></a>Copier l’ID de la page

Le bot est accessible par le biais d’une page Facebook. [Créez une page Facebook](https://www.facebook.com/bookmarks/pages) ou accédez à une page existante.

* Ouvrez la page **About** (À propos de) de Facebook, puis copiez et enregistrez la valeur **Page ID** (ID de la page).

### <a name="create-a-facebook-app"></a>Créer une application Facebook

[Créez une application Facebook](https://developers.facebook.com/quickstarts/?platform=web) sur la page et générez un ID d’application et une clé secrète d’application.

![Créer un ID d’application](~/media/channels/FB-CreateAppId.png)

* Copiez et enregistrez les valeurs **App ID** (ID d’application) et **App Secret** (Clé secrète d’application).

![Enregistrer l’ID et la clé secrète d’application](~/media/channels/FB-get-appid.png)

Positionnez le curseur « Allow API Access to App Settings » (Autoriser l’accès de l’API aux paramètres d’application) sur « Yes » (Oui).

![Paramètres de l’application](~/media/bot-service-channel-connect-facebook/api_settings.png)

### <a name="enable-messenger"></a>Activer Messenger

Activez Facebook Messenger dans la nouvelle application Facebook.

![Activer Messenger](~/media/channels/FB-AddMessaging1.png)

### <a name="generate-a-page-access-token"></a>Générer un jeton d’accès à la page

Dans le volet **Token Generation** (Génération de jeton) de la section Messenger, sélectionnez la page cible. Un jeton d’accès à la page est généré.

* Copiez et enregistrez le jeton sous **Page Access Token** (Jeton d’accès à la page).

![Générer un jeton](~/media/channels/FB-generateToken.png)

### <a name="enable-webhooks"></a>Activer les webhooks

Cliquez sur **Set up Webhooks** (Configurer des webhooks) pour transférer les événements de messagerie de Facebook Messenger vers le bot.

![Activer le webhook](~/media/channels/FB-webhook.png)

### <a name="provide-webhook-callback-url-and-verify-token"></a>Fournir l’URL de rappel du webhook et vérifier le jeton

Dans le [Portail Azure](https://portal.azure.com/), ouvrez le bot, cliquez sur l’onglet **Canaux**, puis cliquez sur **Facebook Messenger**.

* Copiez les valeurs sous **URL de rappel** et **Vérifier le jeton** dans le portail.

![Copier les valeurs](~/media/channels/fb-callbackVerify.png)

1. Revenez à Facebook Messenger et collez les valeurs sous **Callback URL** (URL de rappel) et **Verify Token** (Vérifier le jeton).

2. Sous **Subscription Fields** (Champs d’abonnement), sélectionnez *message\_deliveries* (remise des messages), *messages*, *messaging\_options* (options de messagerie) et *messaging\_postbacks* (publications de messagerie).

3. Cliquez sur **Verify and Save** (Vérifier et enregistrer).

![Configurer le webhook](~/media/channels/FB-webhookConfig.png)

4. Abonnez le webhook à la page Facebook.

![Abonner le webhook](~/media/bot-service-channel-connect-facebook/subscribe-webhook.png)


### <a name="provide-facebook-credentials"></a>Fournir les informations d’identification Facebook

Dans le Portail Azure, collez les valeurs des champs **Facebook App ID** (ID d’application Facebook), **Facebook App Secret** (Secret d’application Facebook), **ID de la page** et **Page Access Token** (Jeton d’accès à la page) que vous avez précédemment copiées à partir de Facebook Messenger. Vous pouvez utiliser le même bot sur plusieurs pages Facebook en ajoutant des ID de page et jetons d’accès supplémentaires.

![Entrer les informations d’identification](~/media/channels/fb-credentials2.png)

### <a name="submit-for-review"></a>Envoyer pour vérification

Facebook requiert une URL de politique de confidentialité et une URL de conditions d’utilisation sur sa page de paramètres d’application de base. La page [Code of Conduct](https://investor.fb.com/corporate-governance/code-of-conduct/default.aspx) (Code de conduite) contient des liens vers les ressources tierces, aidant à créer une politique de confidentialité. La page [Terms of Use](https://www.facebook.com/terms.php) (Conditions d’utilisation) contient des exemples de conditions aidant à créer un document de conditions d’utilisation approprié.

Une fois le bot terminé, Facebook applique son propre [processus de vérification](https://developers.facebook.com/docs/messenger-platform/app-review) pour les applications publiées sur Messenger. Facebook teste le bot pour vérifier qu’il est conforme aux [politiques de la plate-forme](https://developers.facebook.com/docs/messenger-platform/policy-overview) Facebook.

### <a name="make-the-app-public-and-publish-the-page"></a>Rendre l’application publique et publier la page

> [!NOTE]
> Une application reste en [mode de développement](https://developers.facebook.com/docs/apps/managing-development-cycle) jusqu’à sa publication. Les fonctionnalités de plug-in et d’API fonctionnent uniquement pour les administrateurs, les développeurs et les testeurs.

Une fois la vérification effectuée, dans le tableau de bord de l’application, sous App Review (Vérification de l’application), définissez l’application sur Publique.
Assurez-vous que la page Facebook associée à ce bot est publiée. L’état s’affiche dans les paramètres des pages.

## <a name="connect-a-bot-to-facebook-workplace"></a>Connecter un bot à Facebook Workplace

Consultez le [centre d’aide Workplace](https://workplace.facebook.com/help/work/) pour en savoir plus sur Facebook Workplace, et la [documentation du développeur Workplace](https://developers.facebook.com/docs/workplace) pour obtenir des instructions sur le développement pour Facebook Workplace.

Pour configurer un bot afin qu’il communique avec Facebook Workplace, créez une intégration personnalisée et connectez-y le bot.

### <a name="create-a-facebook-workplace-premium-account"></a>Créer un compte Facebook Workplace Premium

Suivez ces [instructions](https://www.facebook.com/workplace) pour créer un compte Facebook Workplace Premium et vous définir comme administrateur système. Gardez à l’esprit que seul l’administrateur système d’un Workplace peut créer des intégrations personnalisées.

### <a name="create-a-custom-integration"></a>Créer une intégration personnalisée

Lorsque vous créez une intégration personnalisée, une application avec des autorisations définies et une page de type « Bot », visible uniquement au sein de votre communauté Workplace, sont créées.

Créez une [intégration personnalisée](https://developers.facebook.com/docs/workplace/custom-integrations-new) pour votre Workplace en suivant les étapes ci-dessous :

- Dans le panneau d’administration (**Admin Panel**), ouvrez l’onglet **Integrations** (Integrations).
- Cliquez sur le bouton **Create your own custom App** (Créer votre application personnalisée).

![Intégration avec Workplace](~/media/channels/fb-integration.png)

- Choisissez un nom d’affichage et une image de profil pour l’application. Ces informations seront partagées avec la page de type « Bot ».
- Positionnez le curseur **Allow API Access to App Settings** (Autoriser l’accès de l’API aux paramètres d’application) sur « Yes » (Oui).
- Copiez et stockez en toute sécurité les valeurs App ID (ID d’application), App Secret (Clé secrète d’application) et App Token (Jeton d’application) qui s’affichent.

![Clés Workplace](~/media/channels/fb-keys.png)

Vous avez créé une intégration personnalisée. Vous trouverez la page de type « Bot » dans votre communauté Workplace, comme indiqué ci-dessous.

![Page Workplace](~/media/channels/fb-page.png)

### <a name="provide-facebook-credentials"></a>Fournir les informations d’identification Facebook

Dans le Portail Azure, collez les valeurs des champs **Facebook App ID** (ID d’application Facebook), **Facebook App Secret** (Secret d’application Facebook), et **Page Access Token** (Jeton d’accès à la page) que vous avez précédemment copiées à partir de Facebook Workplace. Au lieu d’un ID de page traditionnel, utilisez les chiffres qui suivent le nom d’intégration sur la page **À propos de**. Comme pour la connexion d’un bot à Facebook Messenger, les webhooks peuvent être connectés à l’aide des informations d’identification figurant dans Azure.

### <a name="submit-for-review"></a>Envoyer pour vérification
Reportez-vous à la section **Connecter un bot à Facebook Messenger** et la [documentation du développeur Workplace](https://developers.facebook.com/docs/workplace) pour plus d’informations.

### <a name="make-the-app-public-and-publish-the-page"></a>Rendre l’application publique et publier la page
Reportez-vous à la section **Connecter un bot à Facebook Messenger** pour plus d’informations.

## <a name="sample-code"></a>Exemple de code

Pour référence ultérieure, l’exemple de bot <a href="https://aka.ms/facebook-events" target="_blank">Facebook-events</a> peut être utilisé pour explorer la communication du bot avec Facebook Messenger.
