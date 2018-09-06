---
title: Ajouter l’authentification à votre bot par le biais d’Azure Bot Service | Microsoft Docs
description: Découvrez comment utiliser les fonctionnalités d’authentification d’Azure Bot Service pour ajouter l’authentification unique à votre bot.
author: JonathanFingold
ms.author: JonathanFingold
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 7/2/2018
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 37189bb9efbee5a2ad5f40f58106d4d85eda4473
ms.sourcegitcommit: bff936a6a3dd5b1bd3ddfeed8bd1023e52929f08
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/30/2018
ms.locfileid: "43312473"
---
# <a name="add-authentication-to-your-bot-via-azure-bot-service"></a>Ajouter l’authentification à votre bot par le biais d’Azure Bot Service
Ce tutoriel utilise les nouvelles fonctionnalités d’authentification de bot d’Azure Bot Service, facilitant le développement d’un bot qui authentifie les utilisateurs auprès de divers fournisseurs d’identité tels qu’Azure AD (Azure Active Directory), GitHub ou Uber. En outre ces mises à jour favorisent l’amélioration de l’expérience utilisateur en éliminant la _vérification du code magique_ pour certains clients.

Auparavant, votre bot devait inclure des contrôleurs OAuth et des liens de connexion, stocker les ID et secrets clients cibles et effectuer la gestion des jetons d’utilisateur.
<!--
These capabilities were bundled in the BotAuth and AuthBot samples that are on GitHub.
-->

Désormais, les développeurs de bots n’ont plus besoin d’héberger des contrôleurs OAuth ou de gérer le cycle de vie des jetons, ces tâches pouvant maintenant être effectuées par Azure Bot Service.

Cette API offre les fonctionnalités suivantes :

- Des améliorations des canaux pour prendre en charge de nouvelles fonctionnalités d’authentification, telles que les nouvelles bibliothèques WebChat et DirectLineJS visant à éliminer la nécessité de la vérification du code magique à 6 chiffres.
- Des améliorations du portail Azure pour ajouter, supprimer et configurer les paramètres de connexion à divers fournisseurs d’identité OAuth.
- La prise en charge d’une variété de fournisseurs d’identité prêts à l’emploi, dont Azure AD (points de terminaison v1 et v2) et GitHub.
- Des mises à jour des SDK Bot Builder C# et Node.js permettant de récupérer des jetons, de créer des cartes OAuthCards et de gérer les événements TokenResponse.
- Des exemples montrant comment créer un bot qui s’authentifie auprès d’Azure AD (points de terminaison v1 et v2) et de GitHub.

Vous pouvez vous appuyer sur les étapes décrites dans cet article pour ajouter des fonctionnalités à un bot existant. Les exemples de bots suivants illustrent les nouvelles fonctionnalités d’authentification.

| Exemple | Version de Bot Builder | Description |
|:---|:---:|:---|
| [AadV1Bot](https://github.com/Microsoft/BotBuilder/tree/master/CSharp/Samples/AadV1Bot) | v3 | Illustre la prise en charge des cartes OAuthCards dans le SDK C# v3, à l’aide du point de terminaison Azure AD v1 |
| [AadV2Bot](https://github.com/Microsoft/BotBuilder/tree/master/CSharp/Samples/AadV2Bot) | v3 |  Illustre la prise en charge des cartes OAuthCards dans le SDK C# v3, à l’aide du point de terminaison Azure AD v2 |
| [GitHubBot](https://github.com/Microsoft/BotBuilder/tree/master/CSharp/Samples/GitHubBot) | v3 |  Illustre la prise en charge des cartes OAuthCards dans le SDK C# v3, à l’aide de GitHub |
| [BasicOAuth](https://github.com/Microsoft/BotBuilder/tree/master/CSharp/Samples/Microsoft.Bot.Sample.BasicOAuth) | v3 |  Illustre la prise en charge d’OAuth 2.0 dans le SDK C# v3 |

> [!NOTE]
> Les fonctionnalités d’authentification fonctionnent également avec Node.js avec Bot Builder v3. Toutefois, cet article traite uniquement l’exemple de code C#.

Pour obtenir de l’aide ou des informations supplémentaires, reportez-vous à [Ressources supplémentaires sur Bot Framework](https://docs.microsoft.com/azure/bot-service/bot-service-resources-links-help).

## <a name="overview"></a>Vue d’ensemble

Ce tutoriel crée un exemple de bot qui se connecte à Microsoft Graph à l’aide d’un jeton Azure AD v1 ou v2. <!--verify this info and fix wording--> Dans le cadre de ce processus, vous allez utiliser du code issu d’un dépôt GitHub ; ce tutoriel explique comment tout mettre en place, y compris l’application de bot.

- [Créer votre bot et une application d’authentification](#create-your-bot-and-an-authentication-application)
- [Préparer l’exemple de code du bot](#prepare-the-bot-sample-code)
- [Utiliser l’émulateur pour tester votre bot](#use-the-emulator-to-test-your-bot)

Pour effectuer ces étapes, vous aurez besoin de Visual Studio 2017, npm, node et git. Vous devez également avoir des connaissances sur Azure, OAuth 2.0 et le développement de bot.

Une fois que vous aurez terminé, vous aurez un bot pouvant répondre à quelques tâches simples par rapport à une application Azure AD, telles que la vérification et l’envoi d’un e-mail ou l’affichage de vos coordonnées et de celles de votre responsable. Pour ce faire, votre bot utilise un jeton à partir d’une application Azure AD par rapport à la bibliothèque Microsoft.Graph.

La dernière section décompose une partie du code du bot.

- [Remarques sur le flux de la récupération des jetons](#notes-on-the-token-retrieval-flow)

## <a name="create-your-bot-and-an-authentication-application"></a>Créer votre bot et une application d’authentification

Vous devez créer un bot d’inscription sur lequel vous allez publier le code de votre bot et vous devez créer une application Azure AD (v1 ou v2) pour autoriser votre bot à accéder à Office 365.

> [!NOTE]
> Ces fonctionnalités d’authentification fonctionnent avec d’autres types de bots. Toutefois, ce tutoriel utilise un bot d’inscription uniquement.

### <a name="register-an-application-in-azure-ad"></a>Inscrire une application dans Azure AD

Vous avez besoin d’une application Azure AD que votre bot puisse utiliser pour se connecter à l’API Microsoft Graph, à vos propres ressources protégées par Azure AD, etc.

Pour ce bot, vous pouvez utiliser des points de terminaison Azure AD v1 ou v2.
Pour plus d’informations sur les différences entre les points de terminaison v1 et v2, consultez la [comparaison v1-v2](https://docs.microsoft.com/azure/active-directory/develop/active-directory-v2-compare) et la [vue d’ensemble du point de terminaison Azure AD v2.0](https://docs.microsoft.com/azure/active-directory/develop/active-directory-appmodel-v2-overview).

#### <a name="to-create-an-azure-ad-v1-application"></a>Pour créer une application Azure AD v1

1. Accédez à [Azure AD dans le portail Azure](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/Overview).
1. Cliquez sur **Inscriptions des applications**.
1. Dans le panneau **Inscriptions des applications**, cliquez sur **Nouvelle inscription d’application**.
1. Renseignez les champs obligatoires et créez l’inscription d’application.
   1. Donnez un nom à votre application.
   1. Définissez **Type d’application** sur **Application web/API**.
   1. Définissez **l’URL de connexion** sur `https://token.botframework.com/.auth/web/redirect`.
   1. Cliquez sur **Créer**.
      - Une fois l’application créée, elle apparaît dans un volet **Application inscrite**.
      - Récupérez la valeur **ID de l’application**. Vous la fournirez plus loin en guise _d’ID client_.
1. Cliquez sur **Paramètres** pour configurer votre application.
1. Cliquez sur **Clés** pour ouvrir le panneau **Clés**.
   1. Sous **Mots de passe**, créez une clé `BotLogin`.
   1. Définissez sa **Durée** sur **N’expire jamais**.
   1. Cliquez sur **Enregistrer** et récupérez la valeur de la clé. Vous la fournirez plus tard en guise de _secret d’application_.
   1. Fermez le panneau **Clés**.
1. Cliquez sur **Autorisations requises** pour ouvrir le panneau **Autorisations requises**.
   1. Cliquez sur **Add**.
   1. Cliquez sur **Sélectionner une API**, puis sélectionnez **Microsoft Graph** et cliquez sur **Sélectionner**.
   1. Cliquez sur **Sélectionner les autorisations**. Choisissez les autorisations d’application que votre application doit utiliser.

      > [!NOTE]
      > Pour votre bot, évitez les autorisations qui sont marquées comme **Exige un administrateur**, car elles nécessitent la connexion d’un utilisateur et d’un administrateur de locataires.

      Sélectionnez les autorisations déléguées pour Microsoft Graph suivantes :
      - Accéder en lecture aux profils de base de tous les utilisateurs
      - Accéder en lecture aux e-mails utilisateur
      - Connecter et lire le profil utilisateur
      - Envoi de messages en tant qu’utilisateur
      - Afficher le profil de base des utilisateurs
      - Afficher l’adresse e-mail des utilisateurs

   1. Cliquez sur **Sélectionner**, puis sur **Terminé**.
   1. Fermez le panneau **Autorisations requises**.

Vous disposez maintenant d’une application Azure AD v1 configurée.

#### <a name="to-create-an-azure-ad-v2-application"></a>Pour créer une application Azure AD v2

1. Accédez au [portail d’inscription des applications de Microsoft](https://apps.dev.microsoft.com).
1. Cliquez sur **Ajouter une application**.
1. Donnez un nom à votre application Azure AD, puis cliquez sur **Créer**.

    Récupérez le GUID **ID de l’application**. Vous le fournirez plus tard en guise d’ID client pour votre paramètre de connexion.

1. Sous **Secrets de l’application**, cliquez sur **Générer un nouveau mot de passe**.

    Récupérez le mot de passe à partir de la fenêtre contextuelle. Vous le fournirez plus tard en guise de secret client pour votre paramètre de connexion.

1. Sous **Plateformes**, cliquez sur **Ajouter une plateforme**.
1. Dans la fenêtre contextuelle **Ajouter une plateforme**, cliquez sur **Web**.
    1. Laissez la case **Autoriser un flux implicite** cochée.
    1. Pour **URL de redirection**, entrez `https://token.botframework.com/.auth/web/redirect`.
    1. Laissez **URL de déconnexion** vide.
1. Sous **Autorisations pour Microsoft Graph**, vous pouvez ajouter des autorisations déléguées.
    - Pour ce tutoriel, vous devez ajouter les autorisations **Mail.Read**, **Mail.Send**, **openid**, **profile**, **User.Read** et **User.ReadBasic.All**.
      L’étendue du paramètre de connexion doit avoir **openid** et une ressource dans le graphe Azure AD, telle que **Mail.Read**.
    - Récupérez les autorisations que vous choisissez. Vous les fournirez plus tard en guise d’étendues pour votre paramètre de connexion.

1. Cliquez sur **Enregistrer** au bas de la page.

### <a name="create-your-bot-on-azure"></a>Créer votre bot sur Azure

Créez une inscription **Bot Channels Registration** à l’aide du [portail Azure](https://portal.azure.com/).

### <a name="register-your-azure-ad-application-with-your-bot"></a>Inscrire votre application Azure AD auprès de votre bot

L’étape suivante consiste à inscrire auprès de votre bot l’application Azure AD que vous venez de créer.

#### <a name="to-register-an-azure-ad-v1-application"></a>Pour inscrire une application Azure AD v1

1. Accédez à la page de ressources de votre bot sur le [portail Azure](http://portal.azure.com/).
1. Cliquez sur **Settings**.
1. Sous **Paramètres de connexion OAuth** près du bas de la page, cliquez sur **Ajouter un paramètre**.
1. Remplissez le formulaire comme suit :
    1. Pour **Nom**, entrez un nom pour votre connexion. Vous l’utiliserez dans le code de votre bot.
    1. Sous **Fournisseur de services**, sélectionnez **Azure Active Directory**. Une fois que vous avez sélectionné cette option, les champs propres à Azure AD sont affichés.
    1. Pour **Id client**, entrez l’ID d’application que vous avez récupéré pour votre application Azure AD v1.
    1. Pour **Secret du client**, entrez la clé que vous avez récupérée pour la clé `BotLogin` de votre application.
    1. Pour **Type d’autorisation**, entrez `authorization_code`.
    1. Pour **URL de connexion**, entrez `https://login.microsoftonline.com`.
    1. Pour **ID de locataire**, entrez l’ID de locataire pour Azure Active Directory, par exemple `microsoft.com` ou `common`.

       Il s’agit du locataire associé aux utilisateurs qui peuvent être authentifiés. Pour autoriser tout le monde à s’authentifier par le biais du bot, utilisez le locataire `common`.

    1. Pour **URL de la ressource**, entrez `https://graph.microsoft.com/`.
    1. Laissez **Étendues** vide.
1. Cliquez sur **Enregistrer**.

> [!NOTE]
> Ces valeurs permettent à votre application d’accéder aux données Office 365 via l’API Microsoft Graph.

Vous pouvez maintenant utiliser ce nom de connexion dans le code de votre bot pour récupérer des jetons utilisateur.

#### <a name="to-register-an-azure-ad-v2-application"></a>Pour inscrire une application Azure AD v2

1. Accédez à la page de l’inscription Bot Channels Registration de votre bot dans le [Portail Azure](http://portal.azure.com/).
1. Cliquez sur **Settings**.
1. Sous **Paramètres de connexion OAuth** près du bas de la page, cliquez sur **Ajouter un paramètre**.
1. Remplissez le formulaire comme suit :
    1. Pour **Nom**, entrez un nom pour votre connexion. Vous l’utiliserez dans le code de votre bot.
    1. Sous **Fournisseur de services**, sélectionnez **Azure Active Directory v2**. Une fois que vous avez sélectionné cette option, les champs propres à Azure AD sont affichés.
    1. Pour **Id client**, entrez l’ID d’application Azure AD v2 que vous avez récupéré au moment de l’inscription de l’application.
    1. Pour **Secret du client**, entrez le mot de passe d’application Azure AD v2 que vous avez récupéré au moment de l’inscription de l’application.
    1. Pour **ID de locataire**, entrez l’ID de locataire pour Azure Active Directory, par exemple `microsoft.com` ou `common`.

        Il s’agit du locataire associé aux utilisateurs qui peuvent être authentifiés. Pour autoriser tout le monde à s’authentifier par le biais du bot, utilisez le locataire `common`.

    1. Pour **Étendues**, entrez les noms des autorisations que vous avez choisies au moment de l’inscription de l’application : `Mail.Read Mail.Send openid profile User.Read User.ReadBasic.All`.

        > [!NOTE]
        > Pour Azure AD v2, **Étendues** prend une liste de valeurs séparées par des espaces et respectant la casse.

1. Cliquez sur **Enregistrer**.

> [!NOTE]
> Ces valeurs permettent à votre application d’accéder aux données Office 365 via l’API Microsoft Graph.

Vous pouvez maintenant utiliser ce nom de connexion dans le code de votre bot pour récupérer des jetons utilisateur.

#### <a name="to-test-your-connection"></a>Pour tester votre connexion

1. Ouvrez la connexion que vous venez de créer.
1. En haut du volet **Paramètre de connexion du fournisseur de service**, cliquez sur **Tester la connexion**.
1. La première fois, cette action doit ouvrir un nouvel onglet de navigateur répertoriant les autorisations que votre application demande et vous invite à accepter.
1. Cliquez sur **Accepter**.
1. Cette action doit vous rediriger vers une page indiquant que le **test de la connexion a réussi**.

## <a name="prepare-the-bot-sample-code"></a>Préparer l’exemple de code du bot

1. Clonez le dépôt à l’adresse https://github.com/Microsoft/BotBuilder.
1. Ouvrez et générez la solution `BotBuilder\CSharp\Microsoft.Bot.Builder.sln`.
1. Fermez cette solution et ouvrez `BotBuilder\CSharp\Samples\Microsoft.Bot.Builder.Samples.sln`.
1. Définissez le projet de démarrage.
    - Pour un bot qui utilise l’application Azure AD v1, utilisez le projet `Microsoft.Bot.Sample.AadV1Bot`.
    - Pour un bot qui utilise l’application Azure AD v2, utilisez le projet `Microsoft.Bot.Sample.AadV2Bot`.
1. Ouvrez le fichier `Web.config` et modifiez les paramètres d’application comme suit :
    1. Définissez `ConnectionName` sur la valeur que vous avez utilisée quand vous avez configuré le paramètre de connexion OAuth 2.0 de votre bot.
    1. Définissez `MicrosoftAppId` sur l’ID d’application de votre bot.
    1. Définissez `MicosoftAppPassword` sur le secret de votre bot.

    > [!IMPORTANT]
    > Selon les caractères de votre secret, vous pouvez être amené à placer le mot de passe dans une séquence d’échappement XML. Par exemple, les esperluettes (&) doivent être encodées sous la forme `&amp;`.

    ```xml
    <appSettings>
        <add key="ConnectionName" value="<your-AAD-connection-name>"/>
        <add key="MicrosoftAppId" value="<your-bot-appId>" />
        <add key="MicrosoftAppPassword" value="<your-bot-password>" />
    </appSettings>
    ```

    Si vous ne savez pas comment obtenir les valeurs de **l’ID d’application Microsoft** et du **mot de passe d’application Microsoft**, effectuez une recherche dans les **paramètres d’application** du service d’application Azure qui a été provisionné pour votre bot sur le portail Azure.

    > [!NOTE]
    > Maintenant, vous pouvez publier ce code de bot sur votre abonnement Azure (cliquez avec le bouton droit sur le projet et choisissez **Publier**), mais cela n’est pas nécessaire pour ce tutoriel. Vous devriez définir une configuration de publication qui utilise l’application et le plan d’hébergement que vous avez utilisés quand vous avez configuré le bot dans le portail Azure.

## <a name="use-the-emulator-to-test-your-bot"></a>Utiliser l’émulateur pour tester votre bot

Vous devez installer [l’émulateur de bot](https://github.com/Microsoft/BotFramework-Emulator) pour tester votre bot localement. Vous pouvez utiliser l’émulateur v3 ou v4.

1. Démarrez votre bot (avec ou sans débogage).
1. Notez le numéro de port localhost pour la page. Vous aurez besoin de ces informations pour interagir avec votre bot.
1. Démarrez l’émulateur.
1. Connectez-vous à votre bot.

   Si vous n’avez pas déjà configuré la connexion, indiquez l’adresse, ainsi que l’ID d’application Microsoft et le mot de passe de votre bot. Ajoutez `/api/messages` à l’URL du bot. Votre URL ressemble à ceci : `http://localhost:portNumber/api/messages`

1. Tapez `help` pour afficher la liste des commandes disponibles pour le bot, puis testez les fonctionnalités d’authentification.
1. Une fois connecté, vous n’avez pas besoin de refournir vos informations d’identification jusqu’à ce que vous vous déconnectiez.
1. Pour vous déconnecter et annuler votre authentification, tapez `signout`.

<!--To restart completely from scratch you also need to:
1. Navigate to the **AppData** folder for your account.
1. Go to the **Roaming/botframework-emulator** subfolder.
1. Delete the **Cookies** and **Coolies-journal** files.
-->

> [!NOTE]
> L’authentification de bot nécessite l’utilisation du service Bot Connector. Le service accède aux informations d’inscription de canaux de bot pour votre bot, raison pour laquelle vous devez définir le point de terminaison de messagerie de votre bot sur le portail. L’authentification requiert également l’utilisation du protocole HTTPS, raison pour laquelle vous avez dû créer une adresse de transfert HTTPS pour votre bot s’exécutant localement.

<!--The following is necessary for WebChat:
1. Use the **ngrok** command-line tool to get a forwarding HTTPS address for your bot.
   - For information on how to do this, see [Debug any Channel locally using ngrok](https://blog.botframework.com/2017/10/19/debug-channel-locally-using-ngrok/).
   - Any time you exit **ngrok**, you will need to redo this and the following step before starting the Emulator.
1. On the Azure Portal, go to the **Settings** blade for your bot.
   1. In the **Configuration** section, change the **Messaging endpoint** to the HTTPS forwarding address generated by **ngrok**.
   1. Click **Save** to save your change.
-->

## <a name="notes-on-the-token-retrieval-flow"></a>Remarques sur le flux de la récupération des jetons

Quand un utilisateur demande au bot de faire quelque chose qui nécessite sa connexion au bot, celui-ci peut utiliser `Microsoft.Bot.Builder.Dialogs.GetTokenDialog` pour lancer la récupération d’un jeton pour une connexion donnée. Les prochains extraits de code sont tirés de la classe `GetTokenDialog`.

### <a name="check-for-a-cached-token"></a>Rechercher un jeton mis en cache

Dans ce code, le bot commence par effectuer une vérification rapide pour déterminer si Azure Bot Service a déjà un jeton pour l’utilisateur (identifié par l’expéditeur de l’activité actuelle) et pour le nom de connexion (ConnectionName) donné (nom de connexion utilisé dans la configuration). Azure Bot Service a déjà un jeton mis en cache ou n’en a pas. L’appel à GetUserTokenAsync effectue cette « vérification rapide ». Si Azure Bot Service dispose d’un jeton et le retourne, le jeton peut être utilisé immédiatement. Si Azure Bot Service n’a pas de jeton, cette méthode retourne Null. Dans ce cas, le bot peut envoyer une carte OAuthCard personnalisée pour que l’utilisateur se connecte.

```csharp
// First ask Bot Service if it already has a token for this user
var token = await context.GetUserTokenAsync(ConnectionName).ConfigureAwait(false);
if (token != null)
{
    // use the token to do exciting things!
}
else
{
    // If Bot Service does not have a token, send an OAuth card to sign in
    await SendOAuthCardAsync(context, (Activity)context.Activity);
}
```

### <a name="send-an-oauthcard-to-the-user"></a>Envoyer une carte OAuthCard à l’utilisateur

Vous pouvez personnaliser la carte OAuthCard avec les texte et texte de bouton que vous souhaitez. Les points importants sont les suivants :

- Définissez `ContentType` sur `OAuthCard.ContentType`.
- Définissez la propriété `ConnectionName` sur le nom de la connexion que vous souhaitez utiliser.
- Incluez un bouton avec un `CardAction` ayant pour `Type` `ActionTypes.Signin` ; notez que vous n’avez pas besoin de spécifier de valeur pour le lien de connexion.

À la fin de cet appel, le bot doit « attendre le retour du jeton ». Cette attente s’effectue sur le flux d’activité principal, car l’utilisateur pourrait être amené à faire beaucoup de choses pour se connecter.

```csharp
private async Task SendOAuthCardAsync(IDialogContext context, Activity activity)
{
    await context.PostAsync($"To do this, you'll first need to sign in.");

    var reply = await context.Activity.CreateOAuthReplyAsync(_connectionName, _signInMessage, _buttonLabel).ConfigureAwait(false);
    await context.PostAsync(reply);

    context.Wait(WaitForToken);
}
```

### <a name="wait-for-a-tokenresponseevent"></a>Attendre un TokenResponseEvent

Dans ce code, la classe de dialogue du bot attend un `TokenResponseEvent` (nous abordons plus bas son acheminement vers la pile de dialogue). La méthode `WaitForToken` détermine d’abord si cet événement a été envoyé. S’il a été envoyé, il peut être utilisé par le bot. Sinon, la méthode `WaitForToken` prend le texte qui a été envoyé au bot et le transmet à `GetUserTokenAsync`. En effet, certains clients (tels qu’un webchat) n’ont pas besoin de code de vérification du code magique, ce qui leur permet d’envoyer directement le jeton dans le `TokenResponseEvent`. D’autres clients nécessitent toujours le code magique (tels que Facebook ou Slack). Azure Bot Service présente à ces clients un code magique à six chiffres et demande à l’utilisateur de le taper dans la fenêtre de conversation. Bien que cela ne soit pas l’idéal, il s’agit du comportement « de repli » ; ainsi, si `WaitForToke`n reçoit un code, le bot peut envoyer ce code à Azure Bot Service et obtenir un jeton. Si cet appel échoue également, vous pouvez décider de signaler une erreur ou faire autre chose. Cependant, dans la plupart des cas, le bot dispose désormais d’un jeton d’utilisateur.

Si vous consultez le fichier **MessageController.cs**, vous pouvez voir que les activités `Event` de ce type sont également acheminées vers la pile de dialogue.

```csharp
private async Task WaitForToken(IDialogContext context, IAwaitable<object> result)
{
    var activity = await result as Activity;

    var tokenResponse = activity.ReadTokenResponseContent();
    if (tokenResponse != null)
    {
        // Use the token to do exciting things!
    }
    else
    {
        if (!string.IsNullOrEmpty(activity.Text))
        {
            tokenResponse = await context.GetUserTokenAsync(ConnectionName,
                                                               activity.Text);
            if (tokenResponse != null)
            {
                // Use the token to do exciting things!
                return;
            }
        }
        await context.PostAsync($"Hmm. Something went wrong. Let's try again.");
        await SendOAuthCardAsync(context, activity);
    }
}
```

### <a name="message-controller"></a>Contrôleur de message

À l’occasion des appels suivants du bot, le jeton n’est jamais mis en cache par cet exemple de bot. En effet, le bot peut toujours demander le jeton à Azure Bot Service. Ainsi le bot n’a pas besoin d’effectuer des opérations telles que la gestion du cycle de vie des jetons ou l’actualisation du jeton, car Azure Bot Service s’en charge.

```csharp
else if(message.Type == ActivityTypes.Event)
{
    if(message.IsTokenResponseEvent())
    {
        await Conversation.SendAsync(message, () => new Dialogs.RootDialog());
    }
}
```
