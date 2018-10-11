---
title: Tester et déboguer des robots à l’aide de l’émulateur Bot Framework | Microsoft Docs
description: Découvrez comment inspecter, tester et déboguer des robots à l’aide de l’application de bureau Émulateur Bot Framework.
keywords: transcription, outil msbot, services linguistiques, reconnaissance vocale
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 09/20/2018
ms.openlocfilehash: 9fb42795d789f89d0bdc74d348d50dbc0ccaa7cb
ms.sourcegitcommit: 3cb288cf2f09eaede317e1bc8d6255becf1aec61
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/27/2018
ms.locfileid: "47389522"
---
# <a name="debug-with-the-emulator"></a>Déboguer avec l’émulateur

L’émulateur Bot Framework est une application de bureau qui permet aux développeurs de robots de tester et déboguer leurs créations localement ou à distance. L’émulateur vous permet de converser avec le robot et d’inspecter les messages qu’il envoie et reçoit. L’émulateur affiche les messages tels qu’ils apparaîtraient dans une interface utilisateur de conversation web, et journalise les requêtes et réponses JSON à mesure que vous échangez des messages avec le robot. Avant de déployer votre robot dans le cloud, exécutez-le et testez-le localement à l’aide de l’émulateur. Vous pouvez tester votre bot à l’aide de l’émulateur, même si vous ne l’avez pas encore [créé](./bot-service-quickstart.md) avec Azure Bot Service ou configuré pour s’exécuter sur des canaux.

## <a name="prerequisites"></a>Prérequis
- Installer [l’émulateur](https://github.com/Microsoft/BotFramework-Emulator/releases)
- Installer le logiciel de tunnelling [ngrok][ngrokDownload]

## <a name="connect-to-a-bot-running-on-localhost"></a>Se connecter à un bot s’exécutant sur un hôte local

Pour vous connecter à un bot s’exécutant en localement, cliquez sur **Open bot** (Ouvrir le bot) et sélectionnez le fichier .bot. 

![Interface utilisateur de l’émulateur](media/emulator-v4/emulator-welcome.png)

## <a name="view-detailed-message-activity-with-the-inspector"></a>Afficher l’activité de message détaillée avec l’inspecteur

Envoyez un message à votre bot, et ce dernier doit normalement vous répondre. Vous pouvez cliquer sur la bulle de message dans la fenêtre de conversation et inspecter l’activité JSON brute à l’aide de la fonctionnalité **INSPECTOR** (INSPECTEUR) à droite de la fenêtre. Lorsque cette option est sélectionnée, la bulle de message devient jaune et l’objet JSON d’activité s’affiche à gauche de la fenêtre de conversation. Ces informations JSON incluent des métadonnées clés, dont l’ID de canal, le type d’activité, l’ID de conversation, le message au format texte, l’URL du point de terminaison, etc. Vous pouvez inspecter les activités envoyées par l’utilisateur, ainsi que les activités avec lesquelles le bot répond. 

![Émulateur d’activité de message](media/emulator-v4/emulator-view-message-activity-02.png)

## <a name="save-and-load-conversations-with-bot-transcripts"></a>Enregistrer et charger des conversations avec des transcriptions de robot

Les activités dans l’émulateur peuvent être enregistrées sous la forme de transcriptions. Dans une fenêtre de conversation en direct ouverte, sélectionnez **Save Transcript As** (Enregistrer la transcription sous) dans le fichier de transcription. Vous pouvez utiliser le bouton **Start Over** (Recommencer) à tout moment pour effacer une conversation et rétablir une connexion au robot.  

![L’émulateur enregistre les transcriptions](media/emulator-v4/emulator-live-chat.png)

Pour charger les transcriptions, sélectionnez simplement **Fichier > Open Transcript File** (Ouvrir un fichier de transcription), puis sélectionnez la transcription. Une nouvelle fenêtre de transcription s’ouvre et affiche l’activité de message dans la fenêtre de sortie. 

![L’émulateur charge les transcriptions](media/emulator-v4/emulator-load-transcript.png)

## <a name="add-services"></a>Ajouter des services 

Vous pouvez facilement ajouter une application LUIS, une base de connaissances QnA ou un modèle dispatch dans votre fichier .bot directement à partir de l’émulateur. Lorsque le fichier .bot est chargé, sélectionnez le bouton de services à gauche de la fenêtre de l’émulateur. Vous pouvez voir des options sous le menu **Services** pour ajouter LUIS, QnA Maker et Dispatch. 

Pour ajouter une application de service, cliquez simplement sur le bouton **+** et sélectionnez le service que vous souhaitez ajouter. Vous êtes invité à vous connecter au Portail Azure pour ajouter le service au fichier bot et à connecter le service à votre application de bot. 

![Connexion à LUIS](media/emulator-v4/emulator-connect-luis-btn.png)

Quand un des services est connecté, vous pouvez revenir à une fenêtre de conversation en direct pour vérifier que vos services sont connectés et opérationnels. 

![QnA connecté](media/emulator-v4/emulator-view-message-activity.png)

## <a name="inspect-services"></a>Inspecter des services

Avec le nouvel émulateur v4, vous pouvez également inspecter les réponses JSON à partir de LUIS et QnA. À l’aide d’un robot avec un service de langage connecté, vous pouvez sélectionner **trace** dans la fenêtre LOG (JOURNAL) en bas à droite. Ce nouvel outil offre également des fonctionnalités pour mettre à jour vos services de langage directement à partir de l’émulateur. 

![Inspecteur LUIS](media/emulator-v4/emulator-luis-inspector.png)

Avec un service LUIS connecté, vous remarquerez que le lien de trace spécifie **Luis Trace** (Trace LUIS). En suivant ce lien, vous verrez la réponse brute de votre service LUIS, qui inclut des intentions, des entités avec leurs scores spécifiés. Vous avez également la possibilité de réaffecter des intentions pour les énoncés de votre utilisateur. 

![Inspecteur de QnA](media/emulator-v4/emulator-qna-inspector.png)

Avec un service de QnA connecté, le journal affichera l’option **QnA Trace** (Trace QnA) qui permet de voir un aperçu de la paire question-réponse associée à cette activité, ainsi qu’un score de confiance. À partir de là, vous pouvez ajouter une autre formulation de question pour une réponse.

## <a name="configure-ngrok"></a>Configurer ngrok

Si vous utilisez Windows et exécutez l’émulateur Bot Framework derrière un pare-feu ou une autre limite réseau, et souhaitez vous connecter à un robot hébergé à distance, vous devez installer et configurer le logiciel de tunneling **ngrok**. L’émulateur Bot Framework s’intègre étroitement au logiciel de tunneling ngrok (développé par [inconshreveable][inconshreveable]), et peut le lancer automatiquement si nécessaire.

Ouvrez les **paramètres de l’application de l’émulateur**, entrez le chemin d’accès de ngrok, indiquez s’il faut ou non contourner ngrok pour les adresses locales, puis cliquez sur **Enregistrer**.

![chemin d’accès de ngrok](media/emulator-v4/emulator-ngrok-path.png)

## <a name="additional-resources"></a>Ressources supplémentaires

L’émulateur Bot Framework est open source. Vous pouvez [contribuer][EmulatorGithubContribute] au développement en [envoyant des bogues et suggestions][EmulatorGithubBugs].



[EmulatorGithubContribute]: https://github.com/Microsoft/BotFramework-Emulator/wiki/How-to-Contribute
[EmulatorGithubBugs]: https://github.com/Microsoft/BotFramework-Emulator/wiki/Submitting-Bugs-%26-Suggestions

[ngrokDownload]: https://ngrok.com/
[inconshreveable]: https://inconshreveable.com/
