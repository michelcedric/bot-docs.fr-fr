---
title: Activer les fonctions vocales dans le canal Discussion Web | Microsoft Docs
description: Découvrez comment activer les fonctions vocales dans le contrôle de discussion web pour un robot connecté au canal Discussion Web.
keywords: fonctions vocales, discussion web, voix, microphone, audio
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: 37e18f49eb55fcb7d0bf94e96051479753eec8aa
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39299057"
---
# <a name="how-to-enable-speech-in-web-chat"></a>Comment activer les fonctions vocales dans Discussion Web
Vous pouvez activer une interface vocale dans le contrôle Discussion Web. Les utilisateurs interagissent avec l’interface vocale à l’aide du microphone dans le contrôle Discussion Web.

![Exemple de fonction vocale de discussion web](~/media/bot-service-channel-webchat/webchat-sample-speech.png)

Si l’utilisateur tape une réponse au lieu de la prononcer, Discussion Web désactive la fonctionnalité vocale et le robot fournit uniquement une réponse textuelle au lieu de la prononcer. Pour réactiver la réponse vocale, l’utilisateur peut utiliser le microphone pour répondre au robot la prochaine fois. Si le microphone accepte l’entrée, il apparaît sombre ou rempli. S’il est grisé, l’utilisateur clique dessus pour l’activer.

## <a name="prerequisites"></a>Prérequis

  Avant d’exécuter l’exemple, vous devez avoir un secret ou un jeton Direct Line pour le robot que vous souhaitez exécuter à l’aide du contrôle Discussion Web. 
  * Pour plus d’informations sur l’obtention d’un secret Direct Line associé à votre robot, voir [connecter un robot à Direct Line](bot-service-channel-connect-directline.md).
  * Pour plus d’informations sur l’échange du secret contre un jeton, voir [Générer un jeton Direct Line](rest-api/bot-framework-rest-direct-line-3-0-authentication.md).

## <a name="customizing-web-chat-for-speech"></a>Personnalisation de Discussion Web pour les fonctions vocales
Pour activer la fonctionnalité vocale dans Discussion Web, vous devez personnaliser le code JavaScript qui appelle le contrôle Discussion Web. Vous pouvez essayer Discussion Web localement en procédant comme suit.

1. Téléchargez l’[exemple index.html](https://aka.ms/web-chat-speech-sample). <!-- this aka.ms link needs to be updated if the sample location changes -->
2. Modifiez le code dans `index.html` en fonction du type de prise en charge vocale à ajouter. Les types d’implémentations vocales sont décrits dans [Activer les services vocaux](#enable-speech-services). 
3. Démarrez un serveur web. Une façon de le faire consiste à utiliser `npm http-server` en réponse à une invite de commandes Node.js.

   * Pour installer `http-server` globalement afin qu’il puisse être exécuté à partir de la ligne de commande, exécutez la commande suivante :

     ```
     npm install http-server -g
     ```

   * Pour démarrer un serveur web à l’aide du port 8000, à partir du répertoire contenant `index.html`, exécutez la commande suivante :

     ```
     http-server -p 8000
     ```
4. Dirigez votre navigateur vers `http://localhost:8000/samples?parameters`. Par exemple, `http://localhost:8000/samples?s=YOURDIRECTLINESECRET` appelle le robot en utilisant un secret Direct Line. Les paramètres qui peuvent être définis dans la chaîne de requête sont décrits dans le tableau suivant :

   | Paramètre | Description |
   |-----------|-------------|
   | s | Secret Direct Line. Pour plus d’informations sur l’obtention d’un secret Direct Line, voir [Connecter un robot à Direct Line](bot-service-channel-connect-directline.md). |
   | t | Jeton Direct Line. Pour plus d’informations sur la façon de générer ce jeton, voir [Générer un jeton Direct Line](rest-api/bot-framework-rest-direct-line-3-0-authentication.md). |
   | domaine | facultatif. URL d’un autre point de terminaison Direct Line.  |
   | webSocket | facultatif. Défini sur « true » afin d’utiliser WebSocket pour recevoir des messages. La valeur par défaut est `false`. |
   | userId | facultatif. ID de l’utilisateur du robot.  |
   | username | facultatif. Nom d’utilisateur de l’utilisateur du robot.  |
   | botid | facultatif. ID du robot. |
   | botname | facultatif. Nom du robot. |


## <a name="enable-speech-services"></a>Activer les services vocaux
La personnalisation vous permet d’ajouter une fonctionnalité vocale de l’une des manières suivantes :

* **Fonction vocale fournie par le navigateur** : utiliser la fonctionnalité vocale intégrée dans le navigateur web. Pour l’instant, cette fonctionnalité est disponible uniquement sur le navigateur Chrome.
* **Utiliser le service vocal Bing** : vous pouvez utiliser le service vocal Bing pour la reconnaissance et la synthèse vocales. Ce mode d’accès à la fonctionnalité vocale est pris en charge par plusieurs navigateurs. Dans ce cas, le traitement est effectué sur un serveur plutôt que dans le navigateur.
* **Créer un service vocal personnalisé** : vous pouvez créer vos propres composants personnalisés de synthèse et de reconnaissance vocales.

### <a name="browser-provided-speech"></a>Fonctions vocales fournie par le navigateur

Le code suivant instancie les composants de reconnaissance et de synthèse vocales intégrés dans le navigateur. Cette méthode d’ajout de fonctions vocales n’est pas prise en charge par tous les navigateurs. 

> [!NOTE] 
> Google Chrome prend en charge la reconnaissance vocale par le navigateur. Toutefois, Chrome peut bloquer le microphone dans les cas suivants :
> * Si l’URL de la page contenant Discussion Web commence par `http://` au lieu de `https://`.
> * Si l’URL est un fichier local utilisant le protocole `file://` au lieu de `http://localhost:8000`.

[!code-js[Specify speech options to use in-browser speech (JavaScript)](./includes/code/bot-service-channel-connect-webchat-speech.js#BrowserSpeech)]

### <a name="bing-speech-service"></a>Service vocal Bing

Le code suivant instancie les composants de reconnaissance et de synthèse vocales qui utilisent le service vocal Bing. La reconnaissance et la génération de voix sont effectuées sur le serveur. Ce mécanisme est pris en charge dans plusieurs navigateurs. 

> [!TIP]
> Vous pouvez utiliser une préparation de la reconnaissance vocale pour améliorer la précision vocale de votre robot si vous utilisez le service vocal Bing. Pour plus d’informations, voir le billet de blog [Prise en charge de la reconnaissance vocale dans Bot Framework](https://blog.botframework.com/2017/06/26/Speech-To-Text).

[!code-js[Specify speech options to use the Bing Speech API (JavaScript)](./includes/code/bot-service-channel-connect-webchat-speech.js#BingSpeech)]

#### <a name="use-the-bing-speech-service-with-a-token"></a>Utiliser le service Reconnaissance vocale Bing avec un jeton

Vous avez également la possibilité d’activer la reconnaissance vocale de Cognitive Services à l’aide d’un jeton. Le jeton est généré dans un back end sécurisé à l’aide de votre clé API.

L’exemple de code suivant montre comment l’extraction du jeton est effectuée à partir d’un back end sécurisé pour éviter d’exposer la clé API.

[!code-js[Fetch a token to use with the Bing Speech API (JavaScript)](./includes/code/bot-service-channel-connect-webchat-speech.js#FetchToken)]

### <a name="custom-speech-service"></a>Custom Speech Service

Vous pouvez également fournir votre propre reconnaissance vocale personnalisée implémentant ISpeechRecognizer, ou synthèse vocale implémentant ISpeechSynthesis. 

[!code-js[Fetch a token to use with a custom speech service (JavaScript)](./includes/code/bot-service-channel-connect-webchat-speech.js#CustomSpeechService)]

### <a name="pass-the-speech-options-to-web-chat"></a>Passer les options vocales à Discussion web

Le code suivant passe les options vocales au contrôle Discussion Web :

[!code-js[Pass speech options to Web Chat (JavaScript)](./includes/code/bot-service-channel-connect-webchat-speech.js#PassSpeechOptionsToWebChat)]

## <a name="next-steps"></a>Étapes suivantes
À présent que vous pouvez activer l’interaction vocale avec Discussion Web, découvrez comment votre robot génère des messages parlés et ajuste l’état du microphone :
* [Ajouter des fonctions vocales aux messages (C#)](dotnet/bot-builder-dotnet-text-to-speech.md)
* [Ajouter des fonctions vocales aux messages (Node.js)](nodejs/bot-builder-nodejs-text-to-speech.md)

## <a name="additional-resources"></a>Ressources supplémentaires

* Vous pouvez [télécharger le code source](https://github.com/Microsoft/BotFramework-WebChat) pour le contrôle de Discussion Web sur GitHub.
* La [documentation de l’API Reconnaissance vocale Bing](https://docs.microsoft.com/azure/cognitive-services/speech/home) fournit des informations sur l’API Reconnaissance vocale Bing.

