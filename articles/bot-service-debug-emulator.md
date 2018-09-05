---
title: Tester et déboguer des robots à l’aide de l’émulateur Bot Framework | Microsoft Docs
description: Découvrez comment inspecter, tester et déboguer des robots à l’aide de l’application de bureau Émulateur Bot Framework.
keywords: transcription, outil msbot, services linguistiques, reconnaissance vocale
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 04/30/2018
ms.openlocfilehash: 2a92281e9bbee09d8dfb00fbbc67fe6ac6b6c69b
ms.sourcegitcommit: 1abc32353c20acd103e0383121db21b705e5eec3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/21/2018
ms.locfileid: "42756728"
---
# <a name="debug-with-the-bot-framework-emulator"></a>Déboguer à l’aide de l’émulateur Bot Framework

L’émulateur Bot Framework est une application de bureau qui permet aux développeurs de robots de tester et déboguer leurs créations localement ou à distance. L’émulateur vous permet de converser avec le robot et d’inspecter les messages qu’il envoie et reçoit. L’émulateur affiche les messages tels qu’ils apparaîtraient dans une interface utilisateur de conversation web, et journalise les requêtes et réponses JSON à mesure que vous échangez des messages avec le robot. 

> [!TIP] 
> Avant de déployer votre robot dans le cloud, exécutez-le et testez-le localement à l’aide de l’émulateur. Vous pouvez tester votre robot à l’aide de l’émulateur, même si vous ne l’avez pas encore [inscrit](~/bot-service-quickstart-registration.md) auprès du Bot Framework ou configuré pour s’exécuter sur des canaux.

![Interface utilisateur de l’émulateur](media/emulator-v4/emulator-welcome.png)

## <a name="download-the-bot-framework-emulator"></a>Télécharger l’émulateur Bot Framework

Les packages pour Mac, Windows et Linux sont disponibles en téléchargement sur la [page des publications GitHub](https://github.com/Microsoft/BotFramework-Emulator/releases).

## <a name="connect-to-a-bot-running-on-local-host"></a>Se connecter à un robot en cours d’exécution sur un hôte local

![Points de terminaison de l’émulateur](media/emulator-v4/emulator-endpoint.png)

Pour vous connecter à un robot en cours d’exécution localement, sélectionnez l’onglet Bot Explorer (Explorateur de robots) en haut à gauche. Cliquez sur l’icône **+** sous l’onglet **Endpoint** (point de terminaison). Ici, vous pouvez spécifier comme point de terminaison le port sur lequel votre robot s’exécute localement afin de vous y connecter. Cliquez sur **Submit** (Envoyer) pour êtes redirigé vers une fenêtre de conversation en direct dans laquelle vous pouvez interagir avec votre robot.

## <a name="view-detailed-message-activity-with-the-inspector"></a>Afficher l’activité de message détaillée avec l’inspecteur

![Émulateur d’activité de message](media/emulator-v4/emulator-view-message-activity-02.png)

Vous pouvez cliquer sur toute bulle de message dans la fenêtre de conversation et inspecter l’activité JSON brute à l’aide de la fonctionnalité **INSPECTOR** (INSPECTEUR) à droite de la fenêtre. Lorsque cette option est sélectionnée, la bulle de message devient jaune et l’objet JSON d’activité s’affiche à gauche de la fenêtre de conversation. Ces informations JSON incluent des métadonnées clés, dont l’ID de canal, le type d’activité, l’ID de conversation, le message au format texte, l’URL du point de terminaison, etc. Vous pouvez afficher les activités d’inspection envoyées par l’utilisateur, ainsi que les activités avec lesquelles le robot répond. 

Vous pouvez également utiliser l’[inspecteur de canal](bot-service-channel-inspector.md) pour avoir un aperçu des fonctionnalités prises en charge sur les canaux spécifiques.


## <a name="save-and-load-conversations-with-bot-transcripts"></a>Enregistrer et charger des conversations avec des transcriptions de robot

L’activité de message dans l’émulateur peut être enregistrée sous la forme de transcriptions. Dans une fenêtre de conversation en direct, vous pouvez sélectionner **Save Transcript As** (Enregistrer la transcription sous) pour nommer et sélectionner l’emplacement du fichier de sortie de la transcription. 

>[!TIP]
> Vous pouvez utiliser le bouton **Start Over** (Recommencer) à tout moment pour effacer une conversation et rétablir une connexion au robot.  

![L’émulateur enregistre les transcriptions](media/emulator-v4/emulator-live-chat.png)

Pour charger les transcriptions, sélectionnez simplement **File (Fichier)** --> **Open Tranascript File (Ouvrir un fichier de transcription)**, puis sélectionnez la transcription. Une nouvelle fenêtre de transcription s’ouvre et affiche l’activité de message dans la fenêtre de sortie. 

![L’émulateur charge les transcriptions](media/emulator-v4/emulator-load-transcript.png)

## <a name="author-transcripts-with-chatdown"></a>Créer des transcriptions avec Chatdown

L’outil [Chatdown](https://github.com/Microsoft/botbuilder-tools/tree/master/Chatdown) est un générateur de transcription qui utilise un fichier [Markdown](https://daringfireball.net/projects/markdown/syntax) pour générer des transcriptions d’activité. Vous pouvez créer vos propres transcriptions au format Markdown, puis les enregistrer en tant que fichier **.chat** pour générer des transcriptions. Cela est utile pour créer rapidement des scénarios de conversation fictifs pendant le développement d’un robot.  

### <a name="prerequisites"></a>Prérequis

- [Émulateur Bot Framework](https://github.com/Microsoft/BotFramework-Emulator/releases) v4 ou version ultérieure 
- [Node.JS](https://nodejs.org/en/)
 
Chatdown est disponible en tant que module npm requérant Node.js. Pour installer Chatdown, installez-le globalement sur votre ordinateur. 

```
npm install -g chatdown
```
### <a name="create-and-load-transcript-transcript-files"></a>Créer et charger des fichiers de transcription ###

Voici un exemple de création de fichier **.chat**. Ces fichiers au format Markdown contiennent 2 parties :
- Un en-tête qui définit les participants à la conversation (utilisateur, robot)
- La conversation bidirectionnelle entre les participants

```
user=John Doe
bot=Bot

bot: Hello!
user: hey
bot: [Typing][Delay=3000]
What can I do for you?
user: Actually nevermind, goodbye.
bot: bye!
```
[Cliquez ici](https://github.com/Microsoft/botbuilder-tools/tree/master/Chatdown/Examples) pour afficher d’autres exemples de fichiers .chat. 

Pour générer le fichier de transcription, en utilisant la commande **chatdown** dans votre interface de ligne de commande, entrez le nom du fichier .chat, suivi de « > » et du nom du fichier de sortie de la transcription. 

```
chatdown sample.chat > sample.transcript
```
## <a name="manage-bot-resources-with-the-msbot-tool"></a>Gérer les ressources de robot avec l’outil MSBot

Le nouvel outil [MSBot](https://github.com/Microsoft/botbuilder-tools/tree/master/MSBot) vous permet de créer un fichier **.bot** qui stocke dans un seul emplacement les métadonnées relatives aux différents services que votre robot utilise. Ce fichier permet également à votre robot de se connecter à ces services à partir de l’interface de ligne de commande. L’outil est disponible en tant que module npm. Pour l’installer, exécutez la commande suivante :

```
npm install -g msbot 
```
![Fenêtre de l’interface de ligne de commande de MSBot](media/emulator-v4/msbot-cli-window.png)


Pour créer un fichier de robot, dans votre interface de ligne de commande, entrez **msbot init**, suivi du nom de votre robot et du point de terminaison de l’URL cible, par exemple :

```shell
msbot init --name az-cli-bot --endpoint http://localhost:3984/api/messages
```
![Botfile](media/emulator-v4/botfile-generated.png)

>**Remarque :** le robot utilisé dans ce guide est un robot Echo simple, généré à partir de l’extension de robot Azure CLI. Pour en savoir plus sur la création de robots avec Azure CLI, [cliquez ici](https://github.com/Microsoft/botbuilder-tools/tree/master/AzureCli). 

Le fichier .bot vous permet désormais de charger facilement votre robot sur  l’émulateur. Le fichier .bot est également requis pour enregistrer différents points de terminaison et composants de langage dans votre robot. 

![Robot-Fichier-Liste déroulante](media/emulator-v4/bot-file-dropdown.png)

## <a name="add-language-services"></a>Ajouter des services de langage 

Vous pouvez facilement inscrire une application LUIS ou une base de connaissances QnA dans votre fichier .bot directement à partir de l’émulateur. Lorsque le fichier .bot est chargé, sélectionnez le bouton de services à gauche de la fenêtre de l’émulateur. Vous verrez des options sous le menu **Services** pour ajouter LUIS, QnA Maker, Dispatch, des points de terminaison et Azure Bot Service. 

Pour ajouter une application LUIS, cliquez simplement sur le bouton **+** dans le menu LUIS, entrez vos informations d’identification d’application LUIS, puis cliquez sur **Submit** (Envoyer). Cette opération inscrit l’application LUIS dans le fichier .bot et connecte le service à votre application de robot. 

![Connexion à LUIS](media/emulator-v4/emulator-connect-luis-btn.png)

De même, pour ajouter une base de connaissances QnA, cliquez simplement sur le bouton **+** dans le menu QnA, entrez vos informations d’identification auprès de la base de connaissances QnA Maker, puis cliquez sur **Submit** (Envoyer). Votre base de connaissances sera désormais inscrite dans le fichier .bot et prête à être utilisée. 

![Connexion à QnA](media/emulator-v4/emulator-connect-qna-btn.png)

Quand un des services est connecté, vous pouvez revenir à une fenêtre de conversation en direct pour vérifier que vos services sont connectés et opérationnels. 

![QnA connecté](media/emulator-v4/emulator-view-message-activity.png)

## <a name="inspect-language-services"></a>Inspecter les services de langage

Avec le nouvel émulateur v4, vous pouvez également inspecter les réponses JSON à partir de LUIS et QnA. À l’aide d’un robot avec un service de langage connecté, vous pouvez sélectionner **trace** dans la fenêtre LOG (JOURNAL) en bas à droite. Ce nouvel outil offre également des fonctionnalités pour mettre à jour vos services de langage directement à partir de l’émulateur. 

![Inspecteur LUIS](media/emulator-v4/emulator-luis-inspector.png)

Avec un service LUIS connecté, vous remarquerez que le lien de trace spécifie **Luis Trace** (Trace LUIS). En suivant ce lien, vous verrez la réponse brute de votre service LUIS, qui inclut des intentions, des entités avec leurs scores spécifiés. Vous avez également la possibilité de réaffecter des intentions pour les énoncés de votre utilisateur. 

![Inspecteur de QnA](media/emulator-v4/emulator-qna-inspector.png)

Avec un service de QnA connecté, le journal affichera l’option **QnA Trace** (Trace QnA) qui permet de voir un aperçu de la paire question-réponse associée à cette activité, ainsi qu’un score de confiance. À partir de là, vous pouvez ajouter une autre formulation de question pour une réponse.

[!TIP]
> Ces fonctionnalités sont disponibles uniquement pour les robots du Kit de développement logiciel (SDK) v4 


## <a name="speech-recognition"></a>Reconnaissance vocale
L’émulateur Bot Framework prend en charge la reconnaissance vocale via l’[API Speech de Cognitive Services](/azure/cognitive-services/Speech/home). Cela vous permet d’exercer votre robot prenant en charge les fonctions vocales ou Cortana par le biais de la voix dans l’émulateur au cours du développement. L’émulateur Bot Framework fournit la reconnaissance vocale gratuitement pendant jusqu’à trois heures par robot et par jour. 

## <a id="ngrok"></a>Installer et configurer ngrok

Si vous utilisez Windows et exécutez l’émulateur Bot Framework derrière un pare-feu ou une autre limite réseau, et souhaitez vous connecter à un robot hébergé à distance, vous devez installer et configurer le logiciel de tunneling **ngrok**. L’émulateur Bot Framework s’intègre étroitement avec le logiciel de tunneling [ngrok][ngrokDownload] (développé par [inconshreveable][inconshreveable]), et vous pouvez le lancer automatiquement si nécessaire.

Pour installer **ngrok** sur Windows et configurer l’émulateur pour l’utiliser, procédez comme suit : 

1. Téléchargez l’exécutable [ngrok][ngrokDownload] exécutable sur votre ordinateur local.

2. Ouvrez la boîte de dialogue Paramètres de l’application de l’émulateur, entrez le chemin d’accès de ngrok, indiquez s’il faut ou non contourner ngrok pour les adresses locales, puis cliquez sur **Enregistrer**.

![chemin d’accès de ngrok](media/emulator-v4/emulator-ngrok-path.png)

## <a name="additional-resources"></a>Ressources supplémentaires

L’émulateur Bot Framework est open source. Vous pouvez [contribuer][EmulatorGithubContribute] au développement en [envoyant des bogues et suggestions][EmulatorGithubBugs].


[EmulatorGithub]: https://github.com/Microsoft/BotFramework-Emulator
[EmulatorGithubContribute]: https://github.com/Microsoft/BotFramework-Emulator/wiki/How-to-Contribute
[EmulatorGithubBugs]: https://github.com/Microsoft/BotFramework-Emulator/wiki/Submitting-Bugs-%26-Suggestions

[ngrokDownload]: https://ngrok.com/
[inconshreveable]: https://inconshreveable.com/
[BotFrameworkDevPortal]: https://dev.botframework.com/


[EmulatorConnectPicture]: ~/media/emulator/emulator-connect_localhost_credentials.png
[EmulatorNgrokPath]: ~/media/emulator/emulator-configure_ngrok_path.png
[EmulatorNgrokMonitor]: ~/media/emulator/emulator-testbot-ngrok-monitoring.png
[EmulatorUI]: ~/media/emulator/emulator-ui-new.png

[TroubleshootingGuide]: ~/bot-service-troubleshoot-general-problems.md
[TroubleshootingAuth]: ~/bot-service-troubleshoot-authentication-problems.md
[NodeGetStarted]: ~/nodejs/bot-builder-nodejs-quickstart.md
[CSGetStarted]: ~/dotnet/bot-builder-dotnet-quickstart.md
