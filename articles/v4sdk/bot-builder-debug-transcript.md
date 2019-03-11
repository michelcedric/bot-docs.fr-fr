---
title: Déboguer votre bot à l’aide de fichiers de transcription | Microsoft Docs
description: Comprendre comment utiliser un fichier de transcription pour vous aider à déboguer votre robot.
keywords: débogage, faq, fichier de transcription, émulateur
author: DanDev33
ms.author: v-dashel
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservices: sdk
ms.date: 2/26/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 997ad82e15a0fcd67d47b2fd6495c8e88a5ea127
ms.sourcegitcommit: cf3786c6e092adec5409d852849927dc1428e8a2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 03/01/2019
ms.locfileid: "57224817"
---
# <a name="debug-your-bot-using-transcript-files"></a>Déboguer votre bot à l’aide de fichiers de transcription
[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

L’une des clés de la réussite du test et du débogage d’un bot réside dans votre capacité à enregistrer et examiner l’ensemble des conditions qui surviennent lors de l’exécution de votre bot. Cet article décrit la création et l’utilisation d’un fichier de transcription de bot pour fournir un ensemble détaillé d’interactions utilisateur et de réponses de bot à des fins de test et de débogage.

## <a name="the-bot-transcript-file"></a>Fichier de transcription de bot
Un fichier de transcription de bot est un fichier JSON spécialisé qui conserve les interactions entre un utilisateur et votre bot. Il conserve non seulement le contenu d’un message, mais aussi les détails de l’interaction comme l’ID utilisateur, l’ID de canal, le type de canal, les fonctionnalités du canal, l’heure de l’interaction, etc. Toutes ces informations peuvent ensuite servir à identifier et à résoudre des problèmes lors du test ou du débogage de votre bot. 

## <a name="creatingstoring-a-bot-transcript-file"></a>Création/stockage d’un fichier de transcription de bot
Cet article explique comment créer des fichiers de transcription de bot à l’aide de Microsoft [Bot Framework Emulator](https://github.com/Microsoft/BotFramework-Emulator). Il est également possible de créer des fichiers de transcription par programmation. Vous trouverez plus d’informations sur cette approche [ici](./bot-builder-howto-v4-storage.md#blob-transcript-storage). Dans cet article, nous allons utiliser l’exemple de code Bot Framework pour [EchoBot with Counter (EchoBot avec compteur)](https://aka.ms/EchoBot-With-Counter) qui a été modifié pour demander le nom et le numéro de téléphone d’un utilisateur, mais tout code qui est accessible à l’aide Microsoft Bot Framework Emulator peut être utilisé pour créer un fichier de transcription.

Pour commencer ce processus, assurez-vous que le code du bot à tester s’exécute dans votre environnement de développement. Démarrez Bot Framework Emulator, sélectionnez le bouton _Open Bot (Ouvrir un bot)_ et connectez l’émulateur au fichier de _configuration de bot_ de votre code, comme indiqué dans l’image ci-dessous.

![connecter l’émulateur à votre code](./media/emulator_open_bot_configuration.png)

Après avoir connecté l’émulateur à votre code en cours d’exécution, testez votre code en envoyant des interactions utilisateur simulées au bot. Dans cet exemple, nous avons transmis le nom et le numéro de téléphone de l’utilisateur. Après avoir entré toutes les interactions d’utilisateur à conserver, utilisez Bot Framework Emulator pour créer et enregistrer un fichier de transcription contenant cette conversation. 

Dans l’onglet _Live Chat (Conversation en direct)_ (affiché ci-dessous), sélectionnez le bouton _Save Transcript (Enregistrer la transcription)_. 

![sélectionner Save Transcript (Enregistrer la transcription)](./media/emulator_transcript_save.png)

Choisissez l’emplacement et le nom de votre fichier de transcription, puis sélectionnez le bouton Save (Enregistrer).

![enregistrer la transcription sous ursula](./media/emulator_transcript_saveas_ursula.png)

Toutes les interactions d’utilisateur et les réponses de bot que vous avez entrées pour tester votre code avec l’émulateur sont maintenant enregistrées dans un fichier de transcription que vous pouvez recharger ultérieurement pour faciliter le débogage des interactions entre un utilisateur et votre bot.

## <a name="retrieving-a-bot-transcript-file"></a>Récupération d’un fichier de transcription de bot
Pour récupérer un fichier de transcription de bot à l’aide de Framework Bot Emulator, sélectionnez le contrôle de liste _TRANSCRIPTS (TRANSCRIPTIONS)_ dans la section _RESOURCES (RESSOURCES)_ dans le coin supérieur gauche de l’émulateur, comme indiqué ci-dessous. Ensuite, sélectionnez le fichier de transcription que vous souhaitez récupérer. Dans cet exemple, nous récupérons la transcription fichier nommé « Ursula_User_transcript ». La sélection d’un fichier de transcription charge automatiquement l’intégralité de la conservation conservée, dans un nouvel onglet intitulé _Transcript (Transcription)_.

![récupérer la transcription enregistrée](./media/emulator_transcript_retrieve.png)

## <a name="debug-using-transcript-file"></a>Déboguer à l’aide d’un fichier de transcription
Une fois votre fichier de transcription chargé, vous êtes prêt à déboguer les interactions que vous avez capturées entre un utilisateur et votre bot. Pour ce faire, il suffit de cliquer sur n’importe quel événement ou n’importe quelle activité dans la section _LOG (JOURNAL)_ affichée dans la partie inférieure droite de l’émulateur. Dans l’exemple ci-dessous, nous avons sélectionné la première interaction de l’utilisateur, quand il a envoyé le message « Hello ». Ce faisant, toutes les informations de votre fichier de transcription concernant cette interaction s’affichent dans la fenêtre _INSPECTOR (INSPECTEUR)_ au format JSON. En examinant certaines de ces valeurs du bas vers le haut, voici ce que nous voyons :
* Type d’interaction _message_.
* Heure d’envoi du message.
* Texte brut « Hello » envoyé.
* Message envoyé à notre bot.
* ID utilisateur et informations.
* ID du canal, fonctionnalités et informations.

![déboguer à l’aide du fichier de transcription](./media/emulator_transcript_debug.png)

Ce niveau détaillé d’informations vous permet de suivre par à pas les interactions entre l’entrée de l’utilisateur et la réponse de votre bot, ce qui permet de déboguer des situations où votre bot soit n’a pas répondu de la manière attendue, soit n’a pas répondu à l’utilisateur du tout. Ces valeurs et un enregistrement des étapes menant à l’interaction ayant échoué vous permettent de parcourir votre code, de trouver l’emplacement où votre bot ne répond pas comme prévu et de résoudre ces problèmes.

L’utilisation des fichiers de transcription et du Bot Framework Emulator n’est que l’un des nombreux outils à votre disposition pour vous aider à tester et déboguer le code de votre bot et les interactions utilisateur. Pour d’autres méthodes de test et de débogage de votre bot, consultez les ressources indiquées ci-dessous.

## <a name="additional-resources"></a>Ressources supplémentaires

Pour des informations complémentaires sur le test et le débogage, consultez :

* [Recommandations en matière de test et de débogage](./bot-builder-testing-debugging.md)
* [Déboguer avec l’émulateur](../bot-service-debug-emulator.md)
* [Résoudre les problèmes généraux](../bot-service-troubleshoot-bot-configuration.md) ainsi que les autres articles de résolution des problèmes de cette section.
* [Débogage dans Visual Studio](https://docs.microsoft.com/en-us/visualstudio/debugger/index)
