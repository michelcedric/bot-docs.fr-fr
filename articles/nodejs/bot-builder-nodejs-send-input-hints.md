---
title: Ajouter des conseils de saisie aux messages | Microsoft Docs
description: Découvrez comment ajouter des conseils de saisie aux messages à l’aide du kit de développement logiciel (SDK) Bot Builder pour .NET.
author: v-ducvo
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: b5efb024c01437867b6ab1cf99b2f544077eee0c
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39299660"
---
# <a name="add-input-hints-to-messages"></a>Ajouter des conseils de saisie aux messages
> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-add-input-hints.md)
> - [Node.JS](../nodejs/bot-builder-nodejs-send-input-hints.md)
> - [REST](../rest-api/bot-framework-rest-connector-add-input-hints.md)

En spécifiant un conseil de saisie pour un message, vous pouvez indiquer si votre robot accepte, attend ou ignore l’entrée utilisateur une fois le message remis au client. Pour de nombreux canaux, cela permet aux clients de définir l’état des contrôles d’entrée utilisateur en conséquence. Par exemple, si le conseil de saisie d’un message indique que le bot ignore l’entrée utilisateur, le client peut fermer le microphone et désactiver la zone de saisie pour empêcher l’utilisateur de fournir une entrée.

## <a name="accepting-input"></a>Acceptation d’entrées

Pour indiquer que votre bot est passivement prêt pour l’entrée sans être en attente d’une réponse de l’utilisateur, définissez le conseil de saisie du message sur `builder.InputHint.acceptingInput`. Sur la plupart des canaux, cela entraîne l’activation de la zone de saisie du client, ainsi que la coupure du microphone tout en restant accessible à l’utilisateur. Par exemple, Cortana ouvre le microphone afin d’accepter une entrée de l’utilisateur si l’utilisateur maintient le bouton du microphone enfoncé. L’exemple de code suivant crée un message qui indique que le bot accepte l’entrée utilisateur.

[!code-javascript[IMessage.speak](../includes/code/node-input-hints.js#InputHintAcceptingInput)]

## <a name="expecting-input"></a>Attente d’entrée

Pour indiquer que votre bot attend une réponse de l’utilisateur, définissez le conseil de saisie du message sur `builder.InputHint.expectingInput`. Sur la plupart des canaux, cela entraîne l’activation de la zone de saisie du client, ainsi que l’activation du microphone. L’exemple de code suivant crée une invite qui indique que le bot attend l’entrée utilisateur.

[!code-javascript[Prompt](../includes/code/node-input-hints.js#InputHintExpectingInput)]

## <a name="ignoring-input"></a>Ignorer l’entrée

Pour indiquer que votre bot n’est pas prêt à recevoir une entrée de l’utilisateur, définissez le conseil de saisie du message sur `builder.InputHint.ignoringInput`. Sur la plupart des canaux, cela entraîne la désactivation de la zone de saisie du client ainsi que la coupure du microphone. L’exemple de code suivant utilise la méthode `session.say()` pour envoyer un message qui indique que le bot ignore l’entrée utilisateur.

[!code-javascript[Session.say()](../includes/code/node-input-hints.js#InputHintIgnoringInput)]

## <a name="default-values-for-input-hint"></a>Valeurs par défaut pour le conseil de saisie

Si vous ne définissez pas de conseil de saisie pour un message, le kit de développement logiciel (SDK) Bot Builder le fera automatiquement pour vous en suivant cette logique : 

- Si votre bot envoie une invite, le conseil de saisie pour le message spécifie que votre bot **attend une entrée**.</li>
- Si votre bot envoie un message simple, le conseil de saisie pour le message spécifie que votre bot **accepte une entrée**.</li>
- Si votre robot envoie une série de messages consécutifs, le conseil de saisie pour tous les messages sauf le dernier de la série spécifiera que votre bot **ignorant les entrées**, et le conseil de saisie pour le message final de la série spécifie que votre bot **accepte une entrée**.

## <a name="additional-resources"></a>Ressources supplémentaires

- [Ajouter de la reconnaissance vocale aux messages](bot-builder-nodejs-text-to-speech.md)
- [Kit de développement logiciel (SDK) Bot Builder pour Node.js][SDKReference]

[SDKReference]: https://docs.botframework.com/en-us/node/builder/chat-reference/modules/_botbuilder_d_.html

