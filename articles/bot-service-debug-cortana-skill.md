---
title: Tester une compétence Cortana | Microsoft Docs
description: Découvrez comment tester un bot Cortana en appelant une compétence Cortana.
keywords: Kit de développement logiciel (SDK) Bot Builder, inscrire votre bot, cortana
author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 05/01/18
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: ceeb854ace1388b6e0435aacc3acf9027763ee73
ms.sourcegitcommit: 3bf3dbb1a440b3d83e58499c6a2ac116fe04b2f6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/23/2018
ms.locfileid: "46707965"
---
# <a name="test-a-cortana-skill"></a>Tester une compétence Cortana

[!INCLUDE [pre-release-label](includes/pre-release-label-v3.md)]
 
Si vous avez créé une compétence Cortana à l’aide du Kit de développement logiciel (SDK) Bot Builder, vous pouvez la tester en l’appelant à partir de Cortana. Les instructions ci-après vous guident tout au long de la procédure requise pour tester votre compétence Cortana.

## <a name="register-your-bot"></a>Inscrire votre bot
Si vous avez [créé votre bot](~/bot-service-quickstart.md) à l’aide du service de robot dans Azure, votre bot est déjà inscrit, et vous pouvez donc ignorer cette étape.

Si vous avez déployé votre bot à un autre emplacement ou que vous souhaitez le tester localement, vous devez [inscrire](bot-service-quickstart-registration.md) votre bot afin de pouvoir le connecter à Cortana. Lors du processus d’inscription, vous devez fournir le **point de terminaison de messagerie** de votre bot. Si vous choisissez de tester votre bot localement, vous devez exécuter un logiciel de tunneling tel que [ngrok](http://ngrok.com) afin d’obtenir un point de terminaison pour votre bot local.

## <a name="get-messaging-endpoint-using-ngrok"></a>Obtenir un point de terminaison de messagerie à l’aide de ngrok

Si vous exécutez le bot au niveau local, vous pouvez obtenir un point de terminaison à utiliser pour le test en exécutant un logiciel de tunneling, tel que [ngrok](https://ngrok.com). Pour utiliser ngrok afin d’obtenir un point de terminaison, tapez la commande ci-après à partir d’une fenêtre de console : 

```cmd
 ngrok.exe http 3979 -host-header="localhost:3979"
``` 

Cette commande configure et affiche un lien de transfert ngrok qui transfère les requêtes vers votre bot, lequel s’exécute sur le port 3978. L’URL du lien de transfert doit ressembler à ceci : `https://0d6c4024.ngrok.io`.  Ajoutez `/api/messages` à la fin du lien pour former une URL de point de terminaison de messagerie présentant le format suivant : `https://0d6c4024.ngrok.io/api/messages`. 

Entrez cette URL de point de terminaison dans la section **Configuration** du panneau [Paramètres](~/bot-service-manage-settings.md) de votre bot.

## <a name="enable-speech-recognition-priming"></a>Activer la préparation de la reconnaissance vocale
Si votre bot utilise une application Language Understanding (LUIS), veillez à associer l’ID d’application LUIS à votre service de robot inscrit. Cette opération permet à votre bot d’identifier les énoncés parlés qui sont définis dans votre modèle LUIS. Pour plus d’informations, consultez l’article [Configurer la préparation vocale](~/bot-service-manage-speech-priming.md).

## <a name="add-the-cortana-channel"></a>Ajouter le canal Cortana
Pour ajouter Cortana en tant que canal, exécutez la procédure décrite dans l’article [Connecter un bot à Cortana](bot-service-channel-connect-cortana.md).

## <a name="test-using-web-chat-control"></a>Tester le bot à l’aide du contrôle Discussion Web

Pour tester votre bot à l’aide du contrôle Discussion Web intégré au service de robot, cliquez sur **Test in Web Chat** (Tester dans la Discussion Web), puis tapez un message pour vérifier que votre bot fonctionne.

## <a name="test-using-emulator"></a>Tester le bot à l’aide de l’émulateur

Pour tester votre bot à l’aide de [l’émulateur](~/bot-service-debug-emulator.md), procédez comme suit :

1. Exécutez le bot.
2. Ouvrez l’émulateur et renseignez les informations nécessaires. Pour trouver les paramètres **AppID** et **AppPassword** de votre bot, consultez la section [MicrosoftAppID and MicrosoftAppPassword](bot-service-manage-overview.md#microsoftappid-and-microsoftapppassword) (MicrosoftAppID et MicrosoftAppPassword). 
3. Cliquez sur **Connecter** pour connecter l’émulateur à votre bot.
4. Tapez un message pour vérifier que votre bot fonctionne.

## <a name="test-using-cortana"></a>Tester le bot à l’aide de Cortana
Vous pouvez appeler votre compétence Cortana en appelant Cortana à l’aide de la phrase de commande qui lui est associée. 
1. Ouvrez Cortana.
2. Ouvrez l’application Notebook dans Cortana, puis cliquez sur **À propos de moi** pour voir le compte que vous utilisez pour Cortana. Vérifiez que vous êtes connecté avec le même compte Microsoft que celui que vous avez utilisé pour inscrire votre bot. 
   ![Connexion à l’application Notebook de Cortana](~/media/cortana/cortana-notebook.png)
2. Cliquez sur le bouton du microphone dans l’application Cortana ou dans la zone de recherche « Entrez une question » dans Windows, et prononcez la [phrase de commande][InvocationNameGuidelines] de votre bot. La phrase de commande inclut un *nom d’appel*, qui identifie de manière unique la compétence à appeler. Par exemple, si le nom d’appel d’une compétence est « Northwind Photo », une phrase de commande appropriée pourrait inclure « Demander à Northwind Photo de... » ou « Dire à Northwind Photo que... ».

   Vous spécifiez le *Invocation Name* (Nom d’appel) de votre bot lorsque vous configurez ce dernier pour Cortana.
   ![Entrée du nom d’appel lors de la configuration du canal Cortana](~/media/cortana/cortana-invocation-name-callout.png)

3. Si Cortana reconnaît votre phrase de commande, votre bot se lance dans le canevas de Cortana. 

## <a name="troubleshoot"></a>Résolution des problèmes

Si le lancement de votre compétence Cortana échoue, procédez aux vérifications suivantes :
* Vérifiez que vous êtes connecté à Cortana avec le même compte Microsoft que celui que vous avez utilisé pour inscrire votre bot dans le portail Bot Framework.
* Vérifiez que le bot fonctionne en cliquant sur **Test in Web chat** (Tester dans la Discussion Web) pour ouvrir la fenêtre **Conversation** et en tapant un message dans cette dernière.
* Vérifiez que votre nom d’appel est conforme aux [instructions][InvocationNameGuidelines]. Si votre nom d’appel comporte plus de trois mots, des mots difficiles à prononcer ou des mots semblables à d’autres, Cortana peut avoir des difficultés à le reconnaître.
* Si votre compétence utilise un modèle LUIS, veillez à [activer la préparation de la reconnaissance vocale](~/bot-service-manage-speech-priming.md).

Pour obtenir des conseils supplémentaires en matière de résolution des problèmes et de plus amples informations sur l’activation du débogage de votre compétence dans le tableau de bord Cortana, consultez l’article [Enable Debugging of Cortana skills][Cortana-TestBestPractice] (Activer le débogage des compétences Cortana). 


## <a name="next-steps"></a>Étapes suivantes

Une fois que vous avez testé votre compétence Cortana et vérifié qu’elle fonctionne comme vous le souhaitez, vous pouvez la déployer auprès d’un groupe de testeurs de versions bêta ou la mettre à la disposition du public. Pour plus d’informations, consultez l’article [Publishing Cortana Skills][Cortana-Publish] (Publication de compétences Cortana).

## <a name="additional-resources"></a>Ressources supplémentaires
* [Kit de compétences Cortana][CortanaGetStarted]

[CortanaGetStarted]: /cortana/getstarted

[BFPortal]: https://dev.botframework.com/
[CortanaDevCenter]: https://developer.microsoft.com/en-us/cortana

[CortanaSpecificEntities]: https://aka.ms/lgvcto
[CortanaAuth]: https://aka.ms/vsdqcj

[InvocationNameGuidelines]: https://aka.ms/cortana-invocation-guidelines 


[Cortana-Debug]: https://aka.ms/cortana-enable-debug
[Cortana-TestBestPractice]: https://aka.ms/cortana-test-best-practice
[Cortana-Publish]: /cortana/skills/publish-skill
