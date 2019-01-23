---
title: Reconnaître une intention à partir du contenu d’un message | Microsoft Docs
description: Apprenez à reconnaître l’intention de l’utilisateur à l’aide des expressions régulières ou de la vérification du contenu du message.
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 211800211b422bb9c90c00705585be89737c77a9
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/11/2019
ms.locfileid: "54225554"
---
# <a name="recognize-user-intent-from-message-content"></a>Reconnaître l’intention de l’utilisateur à partir du contenu d’un message

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

Quand votre bot reçoit un message d’un utilisateur, il peut utiliser un **module de reconnaissance** pour examiner le message et déterminer l’intention. L’intention permet d’établir des correspondances entre les messages et les dialogues à appeler. Cet article explique comment reconnaître une intention à l’aide d’expressions régulières ou de l’inspection du contenu d’un message. Par exemple, un bot peut utiliser des expressions régulières pour vérifier si un message contient le mot « aide » et appeler un dialogue d’aide. Un bot peut également vérifier les propriétés du message utilisateur, par exemple pour voir si l’utilisateur envoie une image et non du texte et appeler un dialogue de traitement d’images. 

> [!NOTE]
> Pour plus d’informations sur la reconnaissance de l’intention avec LUIS, consultez [Reconnaître les intentions et les entités avec LUIS](bot-builder-nodejs-recognize-intent-luis.md) 


## <a name="use-the-built-in-regular-expression-recognizer"></a>Utiliser le module de reconnaissance d’expressions régulières intégré
Utilisez [RegExpRecognizer][RegExpRecognizer] pour détecter l’intention de l’utilisateur à l’aide d’une expression régulière. Vous pouvez passer plusieurs expressions au module de reconnaissance pour prendre en charge plusieurs langues. 

> [!TIP]
> Consultez [Prendre en charge la localisation](bot-builder-nodejs-localization.md) pour plus d’informations sur la localisation de votre bot.

Le code suivant crée un module de reconnaissance d’expressions régulières nommé `CancelIntent` et l’ajoute à votre bot. Le module de reconnaissance dans cet exemple fournit des expressions régulières pour les paramètres régionaux `en_us` et `ja_jp`. 

[!code-js[Add a regular expression recognizer (JavaScript)](../includes/code/node-regex-recognizer.js#addRegexRecognizer)]

Une fois que le module de reconnaissance est ajouté à votre bot, attachez un [triggerAction][triggerAction] au dialogue que vous souhaitez que le bot appelle quand le module de reconnaissance détecte l’intention. Utilisez l’option [matches][matches] pour spécifier le nom de l’intention, comme indiqué dans le code suivant :

[!code-js[Map the CancelIntent recognizer to a cancel dialog (JavaScript)](../includes/code/node-regex-recognizer.js#bindCancelDialogToRegexRecognizer)]

Les modules de reconnaissance de l’intention sont globaux, ce qui signifie que le module de reconnaissance sera exécuté pour chaque message reçu de l’utilisateur. Si un module de reconnaissance détecte une intention liée à un dialogue à l’aide d’un `triggerAction`, il peut déclencher l’interruption du dialogue actif. L’autorisation et la gestion des interruptions est une conception flexible qui tient compte de ce que les utilisateurs font réellement.

> [!TIP] 
> Pour savoir comment un `triggerAction` fonctionne avec les dialogues, consultez [Gérer le flux de la conversation](bot-builder-nodejs-manage-conversation-flow.md). Pour en savoir plus sur les différentes actions que vous pouvez associer à une intention reconnue, [Gérer les actions de l’utilisateur](bot-builder-nodejs-dialog-actions.md).

## <a name="register-a-custom-intent-recognizer"></a>Inscrire un module de reconnaissance de l’intention personnalisé
Vous pouvez également implémenter un module de reconnaissance personnalisé. Cet exemple ajoute un module de reconnaissance simple qui attend que l’utilisateur prononce « aide » ou « au revoir », mais vous pouvez facilement ajouter un module de reconnaissance qui effectue un traitement plus complexe, tel que celui qui reconnaît quand l’utilisateur envoie une image. 


[!code-js[Add a custom recognizer (JavaScript)](../includes/code/node-howto-recognize-intent.js#addCustomRecognizer)]

Une fois que vous avez inscrit un module de reconnaissance, vous pouvez l’associer à une action en utilisant une clause `matches`.

[!code-js[Bind intents to actions (JavaScript)](../includes/code/node-howto-recognize-intent.js#bindIntentsToActions)]

## <a name="disambiguate-between-multiple-intents"></a>Lever l’ambiguïté entre plusieurs intentions

Votre bot peut inscrire plusieurs modules de reconnaissance. Notez que l’exemple de module de reconnaissance personnalisé implique d’assigner un score à chaque intention. Sachant que votre bot peut avoir plusieurs modules de reconnaissance et que le kit SDK Bot Framework propose une logique intégrée, cela vise à lever toute ambiguïté entre les intentions retournées par plusieurs modules de reconnaissance. Le score attribué à une intention est généralement compris entre 0,0 et 1,0, mais un module de reconnaissance personnalisé peut définir une intention supérieure à 1,1 pour garantir que cette intention sera toujours choisie par la logique de levée d’ambiguïté du kit SDK Bot Framework. 

Par défaut, les modules de reconnaissance s’exécutent en parallèle, mais vous pouvez définir recognizeOrder dans [IIntentRecognizerSetOptions][IntentRecognizerSetOptions] pour que le processus s’arrête dès que votre bot en trouve une avec un score de 1,0.

Le kit SDK Bot Framework inclut un [exemple][DisambiguationSample] qui montre comment fournir une logique de levée d’ambiguïté personnalisée dans votre bot en implémentant [IDisambiguateRouteHandler][IDisambiguateRouteHandler].

## <a name="next-steps"></a>Étapes suivantes
La logique de l’utilisation d’expressions régulières et de l’inspection du contenu d’un message peut devenir complexe, en particulier si le flux de conversation de votre bot est très ouvert. Pour aider votre bot à gérer une plus grande variété d’entrées textuelles et parlées des utilisateurs, vous pouvez utiliser un service de reconnaissance des intentions comme [LUIS][LUIS] pour ajouter la compréhension du langage naturel à votre bot.

> [!div class="nextstepaction"]
> [Reconnaître les intentions et les entités avec LUIS](bot-builder-nodejs-recognize-intent-luis.md)


[LUIS]: https://www.luis.ai/

[triggerAction]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.dialog.html#triggeraction

[matches]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.itriggeractionoptions.html#matches

[node-js-bot-how-to]: bot-builder-nodejs-recognize-intent-luis.md

[LUISAzureDocs]: /azure/cognitive-services/LUIS/Home

[IMessage]: http://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.imessage

[IntentRecognizerSetOptions]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.iintentrecognizersetoptions.html

[LuisRecognizer]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.luisrecognizer

[LUISSample]: https://aka.ms/v3-js-luisSample

[LUISConcepts]: https://docs.botframework.com/en-us/node/builder/guides/understanding-natural-language/

[DisambiguationSample]: https://aka.ms/v3-js-onDisambiguateRoute

[IDisambiguateRouteHandler]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.idisambiguateroutehandler.html

[RegExpRecognizer]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.regexprecognizer.html

[AlarmBot]: https://aka.ms/v3-js-luisSample
