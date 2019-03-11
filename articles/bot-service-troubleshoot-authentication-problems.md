---
title: Résoudre les problèmes d’authentification du Bot Framework | Microsoft Docs
description: Découvrez comment résoudre les erreurs d’authentification avec votre bot.
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 02/26/2019
ms.openlocfilehash: 780dcf4d9db48f9ef7f5a92180dc13c41cc63305
ms.sourcegitcommit: cf3786c6e092adec5409d852849927dc1428e8a2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 03/01/2019
ms.locfileid: "57224937"
---
# <a name="troubleshooting-bot-framework-authentication"></a>Résoudre les problèmes d’authentification du Bot Framework

Ce guide peut vous aider à résoudre les problèmes d’authentification rencontrés avec votre bot en évaluant une série de scénarios pour déterminer où se produit le problème. 

> [!NOTE]
> Pour suivre toutes les étapes de ce guide, vous devez télécharger et utiliser le [Bot Framework Emulator][Emulator]. Vous devez également avoir accès aux paramètres d’inscription du bot dans le <a href="https://dev.botframework.com" target="_blank">portail Bot Framework</a>.

## <a id="PW"></a> ID et mot de passe d’application

La sécurité du bot est configurée par **l’ID d’application Microsoft** et le **mot de passe d’application Microsoft** que vous obtenez lorsque vous inscrivez votre bot auprès du Bot Framework. Ces valeurs sont généralement spécifiées dans le fichier de configuration du bot et utilisées pour récupérer les jetons d’accès à partir du service de compte Microsoft. 

Si ce n’est déjà fait, [déployez votre bot sur Azure](~/bot-builder-howto-deploy-azure.md) pour obtenir un **ID d’application Microsoft** et un **mot de passe d’application Microsoft** qu’il peut utiliser pour l’authentification. 

> [!NOTE]
> Pour trouver les paramètres **AppID** et **AppPassword** d’un bot déjà déployé, consultez la rubrique [MicrosoftAppID et MicrosoftAppPassword](bot-service-manage-overview.md#microsoftappid-and-microsoftapppassword).

## <a name="step-1-disable-security-and-test-on-localhost"></a>Étape 1 : Désactiver la sécurité et tester le bot sur localhost

Cette étape consiste à vérifier que votre bot est accessible et fonctionnel sur localhost lorsque la sécurité est désactivée. 

> [!WARNING]
> Lorsque vous désactivez la sécurité pour votre bot, des attaquants inconnus peuvent emprunter l’identité d’utilisateurs. Implémentez la procédure suivante uniquement si vous travaillez dans un environnement de débogage protégé.

### <a id="disable-security-localhost"></a> Désactiver la sécurité

Pour désactiver la sécurité pour votre bot, modifiez ses paramètres de configuration pour supprimer les valeurs de mot de passe et d’ID d’application. 

::: moniker range="azure-bot-service-3.0"

Si vous utilisez le kit SDK Bot Framework pour .NET, modifiez ces paramètres dans votre fichier Web.config : 

```xml
<appSettings>
  <add key="MicrosoftAppId" value="" />
  <add key="MicrosoftAppPassword" value="" />
</appSettings>
```

Si vous utilisez le kit SDK Bot Framework pour Node.js, modifiez ces valeurs (ou mettez à jour les variables d’environnement correspondantes) :

```javascript
var connector = new builder.ChatConnector({
  appId: null,
  appPassword: null
});
```

::: moniker-end

::: moniker range="azure-bot-service-4.0"

Si vous utilisez le kit SDK Bot Framework pour .NET, modifiez ces paramètres dans votre fichier `.bot` :

```json
"services": [
  {
    "appId": "<your app ID>",
    "appPassword": "<your app password>",
  }
]
```

Si vous utilisez le kit SDK Bot Framework pour Node.js, modifiez ces valeurs (ou mettez à jour les variables d’environnement correspondantes) :

```javascript
const adapter = new BotFrameworkAdapter({
    appId: null,
    appPassword: null
});
```

Si vous utilisez le fichier de configuration `.bot`, vous pouvez mettre à jour les valeurs `appId` et `appPassword` sur `""`.

::: moniker-end

### <a name="test-your-bot-on-localhost"></a>Tester votre bot sur localhost 

Ensuite, testez votre bot sur localhost à l’aide du Bot Framework Emulator.

1. Démarrez votre bot sur localhost.
2. Démarrez le Bot Framework Emulator.
3. Connectez-vous à votre bot à l’aide de l’émulateur.
    - Tapez `http://localhost:port-number/api/messages` dans la barre d’adresse de l’émulateur, où **port-number** correspond au numéro de port affiché dans le navigateur où s’exécute votre application. 
    - Assurez-vous que les champs **Microsoft App ID** (ID d’application Microsoft) et **Microsoft App Password** (Mot de passe d’application Microsoft) sont vides.
    - Cliquez sur **Connecter**.
4. Pour tester la connectivité avec votre bot, tapez du texte dans l’émulateur et appuyez sur Entrée.

Si le bot répond à l’entrée et qu’aucune erreur n’apparaît dans la fenêtre de conversation, vous avez vérifié que votre bot est accessible et fonctionnel sur localhost lorsque la sécurité est désactivée. Passez à [l’étape 2](#step-2).

Si une ou plusieurs erreurs sont indiquées dans la fenêtre de conversation, cliquez dessus pour obtenir plus d’informations. Voici quelques problèmes courants :

* Les paramètres de l’émulateur spécifient un point de terminaison incorrect pour le bot. Assurez-vous que vous avez indiqué le numéro de port approprié dans l’URL et le chemin d’accès approprié à la fin de l’URL (par exemple, `/api/messages`).
* Les paramètres de l’émulateur spécifient un point de terminaison de bot qui commence par `https`. Sur localhost, le point de terminaison doit commencer par `http`.
* Les paramètres de l’émulateur spécifient une valeur pour les champs **Microsoft App ID** (ID d’application Microsoft) et/ou **Microsoft App Password** (Mot de passe d’application Microsoft). Ces deux champs doivent être vides.
* La sécurité n’a pas été désactivée pour le bot. [Vérifiez](#disable-security-localhost) que le bot ne spécifie pas une valeur pour l’ID ou le mot de passe d’application.

## <a id="step-2"></a> Étape 2 : Vérifier l’ID et le mot de passe d’application de votre bot

Cette étape consiste à vérifier que le l’ID et le mot de passe d’application utilisés par votre bot pour l’authentification sont valides. (Si vous ne connaissez pas ces valeurs, [récupérez-les](#PW) maintenant.) 

> [!WARNING]
> Les instructions suivantes décrivent comment désactiver la vérification SSL pour `login.microsoftonline.com`. Appliquez cette procédure uniquement sur un réseau sécurisé et pensez à changer le mot de passe de votre application par la suite.

### <a name="issue-an-http-request-to-the-microsoft-login-service"></a>Émettre une requête HTTP au service de connexion Microsoft

Ces instructions expliquent comment utiliser [cURL](https://curl.haxx.se/download.html) pour émettre une requête HTTP au service de connexion de Microsoft. Vous pouvez utiliser un autre outil tel que Postman. Assurez-vous simplement que la requête est conforme au [protocole d’authentification](~/rest-api/bot-framework-rest-connector-authentication.md) du Bot Framework.

Pour vérifier que l’ID et le mot de passe d’application de votre bot sont valides, émettez la requête suivante à l’aide de **cURL** en remplaçant `APP_ID` et `APP_PASSWORD` par l’ID et le mot de passe d’application de votre bot.

> [!TIP]
> Il se peut que votre mot de passe contienne des caractères spéciaux qui rendent l’appel suivant non valide. Dans ce cas, essayez de convertir votre mot de passe en encodage d’URL.

```cmd
curl -k -X POST https://login.microsoftonline.com/botframework.com/oauth2/v2.0/token -d "grant_type=client_credentials&client_id=APP_ID&client_secret=APP_PASSWORD&scope=https%3A%2F%2Fapi.botframework.com%2F.default"
```

Cette requête tente d’échanger l’ID et le mot de passe d’application de votre bot contre un jeton d’accès. Si la requête aboutit, vous recevez une charge utile JSON qui contient notamment une propriété `access_token`. 

```json
{"token_type":"Bearer","expires_in":3599,"ext_expires_in":0,"access_token":"eyJ0eXAJKV1Q..."}
```

Si la requête aboutit, vous avez vérifié que l’ID et le mot de passe d’application que vous avez spécifiés dans la requête sont valides. Passez à [l’étape 3](#step-3).

Si vous recevez une erreur en réponse à la requête, examinez la réponse pour identifier la cause de l’erreur. Si la réponse indique que l’ID ou le mot de passe d’application n’est pas valide, [récupérez les valeurs correctes](#PW) à partir du portail Bot Framework et émettez une nouvelle fois la requête avec les nouvelles valeurs pour vérifier leur validité. 

## Étape 3 : Activer la sécurité et tester le bot sur localhost <a id="step-3"></a>

À ce stade, vous avez vérifié que votre bot est accessible et fonctionnel sur localhost lorsque la sécurité est désactivée et que l’ID et le mot de passe d’application utilisés par votre bot pour l’authentification sont valides. Cette étape consiste à vérifier que votre bot est accessible et fonctionnel sur localhost lorsque la sécurité est activée.

### <a id="enable-security-localhost"></a> Activer la sécurité

La sécurité de votre bot s’appuie sur les services Microsoft, même lorsque votre bot s’exécute uniquement sur localhost. Pour activer la sécurité pour votre bot, modifiez ses paramètres de configuration pour renseigner l’ID et le mot de passe d’application avec les valeurs que vous avez vérifiées à [l’étape 2](#step-2).  Par ailleurs, vérifiez que vos packages sont à jour, en particulier `System.IdentityModel.Tokens.Jwt` et `Microsoft.IdentityModel.Tokens`.

Si vous utilisez le kit SDK Bot Framework pour .NET, renseignez ces paramètres dans votre fichier `appsettings.config` ou les valeurs correspondantes dans votre fichier `.bot` :

```xml
<appSettings>
  <add key="MicrosoftAppId" value="APP_ID" />
  <add key="MicrosoftAppPassword" value="PASSWORD" />
</appSettings>
```

Si vous utilisez le kit SDK Bot Framework pour Node.js, renseignez ces paramètres (ou mettez à jour les variables d’environnement correspondantes) :

```javascript
var connector = new builder.ChatConnector({
  appId: 'APP_ID',
  appPassword: 'PASSWORD'
});
```

> [!NOTE]
> Pour trouver les paramètres **AppID** et **AppPassword** de votre bot, consultez la rubrique [MicrosoftAppID and MicrosoftAppPassword](bot-service-manage-overview.md#microsoftappid-and-microsoftapppassword) (MicrosoftAppID et MicrosoftAppPassword).

### <a name="test-your-bot-on-localhost"></a>Tester votre bot sur localhost 

Ensuite, testez votre bot sur localhost à l’aide du Bot Framework Emulator.

1. Démarrez votre bot sur localhost.
2. Démarrez le Bot Framework Emulator.
3. Connectez-vous à votre bot à l’aide de l’émulateur.
    - Tapez `http://localhost:port-number/api/messages` dans la barre d’adresse de l’émulateur, où **port-number** correspond au numéro de port affiché dans le navigateur où s’exécute votre application. 
    - Entrez l’ID d’application de votre bot dans le champ **Microsoft App ID** (ID d’application Microsoft).
    - Entrez le mot de passe de votre bot dans le champ **Microsoft App Password** (mot de passe d’application Microsoft).
    - Cliquez sur **Connecter**.
4. Pour tester la connectivité avec votre bot, tapez du texte dans l’émulateur et appuyez sur Entrée.

Si le bot répond à l’entrée et qu’aucune erreur n’apparaît dans la fenêtre de conversation, vous avez vérifié que votre bot est accessible et fonctionnel sur localhost lorsque la sécurité est activée.  Passez à [l’étape 4](#step-4).

Si une ou plusieurs erreurs sont indiquées dans la fenêtre de conversation, cliquez dessus pour obtenir plus d’informations. Voici quelques problèmes courants :

* Les paramètres de l’émulateur spécifient un point de terminaison incorrect pour le bot. Assurez-vous que vous avez indiqué le numéro de port approprié dans l’URL et le chemin d’accès approprié à la fin de l’URL (par exemple, `/api/messages`).
* Les paramètres de l’émulateur spécifient un point de terminaison de bot qui commence par `https`. Sur localhost, le point de terminaison doit commencer par `http`.
* Dans les paramètres de l’émulateur, les champs **Microsoft App ID** (ID d’application Microsoft) et/ou **Microsoft App Password** (Mot de passe d’application Microsoft) ne contiennent pas de valeurs valides. Les deux champs doivent être renseignés et chaque champ doit contenir la valeur correspondante que vous avez vérifiée à [l’étape 2](#step-2).
* La sécurité n’a pas été activée pour le bot. [Vérifiez](#enable-security-localhost) que les paramètres de configuration du bot spécifient des valeurs pour l’ID et le mot de passe d’application.

## Étape 4 : Tester votre bot dans le cloud <a id="step-4"></a>

À ce stade, vous avez vérifié que votre bot est accessible et fonctionnel sur localhost lorsque la sécurité est désactivée et que l’ID et le mot de passe d’application de votre bot sont valides. Vous avez également vérifié que votre bot est accessible et fonctionnel sur localhost lorsque la sécurité est activée. Cette étape consiste à déployer votre bot dans le cloud et à vérifier qu’il est accessible et fonctionnel lorsque la sécurité est activée. 

### <a name="deploy-your-bot-to-the-cloud"></a>Déployer votre bot dans le cloud

Le Bot Framework implique que les bots soient accessibles sur Internet. Par conséquent, vous devez déployer votre bot sur une plateforme d’hébergement cloud comme Azure. Veillez à activer la sécurité pour votre bot avant le déploiement, comme décrit à [l’étape 3](#step-3).

> [!NOTE]
> Si vous n’avez pas encore de fournisseur d’hébergement cloud, vous pouvez vous inscrire pour obtenir un <a href="https://azure.microsoft.com/en-us/free/" target="_blank">compte gratuit</a>. 

Si vous déployez votre bot sur Azure, le protocole SSL sera automatiquement configuré pour votre application. Le point de terminaison **HTTPS** requis par le Bot Framework sera ainsi disponible. Si vous effectuez le déploiement sur un autre fournisseur d’hébergement cloud, veillez à vérifier que votre application est configurée pour le protocole SSL, de sorte que le bot dispose d’un point de terminaison **HTTPS**.

### <a name="test-your-bot"></a>Tester votre bot 

Pour tester votre bot dans le cloud avec la sécurité activée, procédez comme suit.

1. Assurez-vous que votre bot a été correctement déployé et qu’il est en cours d’exécution. 
2. Connectez-vous au <a href="https://dev.botframework.com" target="_blank">portail Bot Framework</a>.
3. Cliquez sur **My Bots** (Mes bots).
4. Sélectionnez le bot à tester.
5. Cliquez sur **Test** pour ouvrir le bot dans un contrôle de discussion web intégré.
6. Pour tester la connectivité avec votre bot, tapez du texte dans le contrôle de discussion web et appuyez sur Entrée.

Si une erreur est indiquée dans la fenêtre de conversation, examinez le message d’erreur pour déterminer la cause de l’erreur. Voici quelques problèmes courants : 

* Le **point de terminaison de messagerie** spécifié dans la page **Paramètres** de votre bot dans le portail Bot Framework est incorrect. Assurez-vous que vous avez indiqué le chemin d’accès approprié à la fin de l’URL (par exemple, `/api/messages`).
* Le **point de terminaison de messagerie** spécifié dans la page **Paramètres** de votre bot dans le portail Bot Framework ne commence pas par `https` ou n’est pas approuvé par le Bot Framework. Votre bot doit disposer d’un certificat valide approuvé chaîne.
* Le bot est configuré avec des valeurs d’ID ou de mot de passe d’application manquantes ou incorrectes. [Vérifiez](#enable-security-localhost) que les paramètres de configuration du bot spécifient des valeurs valides pour l’ID et le mot de passe d’application.

Si le bot répond correctement à l’entrée, vous avez vérifié que votre bot est accessible et fonctionnel dans le cloud lorsque la sécurité est activée. À ce stade, votre bot est prêt à [se connecter à un canal](~/bot-service-manage-channels.md) en toute sécurité, notamment à Skype, Facebook Messenger ou Direct Line.

## <a name="additional-resources"></a>Ressources supplémentaires

Si vous rencontrez toujours des problèmes après avoir suivi les étapes ci-dessus, vous pouvez :

* Consultez la procédure [Déboguer un bot](bot-service-debug-bot.md) ainsi que les autres articles de débogage de cette section.
* [Déboguer votre bot dans le cloud](~/bot-service-debug-emulator.md) en utilisant le Bot Framework Emulator et <a href="https://ngrok.com/" target="_blank">ngrok</a>
* Utiliser un outil de proxy comme [Fiddler](https://www.telerik.com/fiddler) pour inspecter le trafic HTTPS vers votre bot et à partir de votre bot (*Fiddler n’est pas un produit Microsoft.*)
* Consulter le [guide d’authentification de Bot Connector][BotConnectorAuthGuide] pour en savoir plus sur les technologies d’authentification utilisées par le Bot Framework.
* Demander de l’aide à d’autres personnes en vous reportant aux ressources de [support][Support] du Bot Framework. 

[BotConnectorAuthGuide]: ~/rest-api/bot-framework-rest-connector-authentication.md
[Support]: bot-service-resources-links-help.md
[Emulator]: bot-service-debug-emulator.md
