---
title: Ajouter des fonctionnalités vocales aux messages | Microsoft Docs
description: Découvrez comment ajouter des fonctionnalités vocales aux messages à l’aide du service Bot Connector.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
ms.openlocfilehash: 2aac000b7e8dd52b00659ffecde5184df6c29991
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2018
ms.locfileid: "49998686"
---
# <a name="add-speech-to-messages"></a>Ajouter de la reconnaissance vocale aux messages
> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-text-to-speech.md)
> - [Node.JS](../nodejs/bot-builder-nodejs-text-to-speech.md)
> - [REST](../rest-api/bot-framework-rest-connector-text-to-speech.md)

Si vous générez un bot pour un canal où sont activées les fonctionnalités vocales, tel que Cortana, vous pouvez créer des messages qui spécifient le texte que doit énoncer votre bot. Vous pouvez également tenter d’influencer l’état du microphone du client en spécifiant un [conseil de saisie](bot-framework-rest-connector-add-input-hints.md) pour indiquer si votre bot accepte, attend ou ignore l’entrée de l’utilisateur.

## <a name="specify-text-to-be-spoken-by-your-bot"></a>Spécifier le texte à énoncer par votre bot

Pour spécifier le texte que doit énoncer votre bot sur un canal où sont activées les fonctionnalités vocales, définissez la propriété `speak` dans l’objet [Activity][Activity] qui représente votre message. Vous pouvez utiliser la propriété `speak`, une chaîne de texte brut ou une chaîne au format <a href="https://msdn.microsoft.com/en-us/library/hh378377(v=office.14).aspx" target="_blank">SSML (Speech Synthesis Markup Language)</a>, qui est un langage de balisage XML qui vous permet de gérer diverses caractéristiques de votre bot, telles que la voix, le débit, le volume, la prononciation, le ton, etc. 

La requête suivante envoie un message qui spécifie le texte à afficher et le texte à énoncer, et indique que le bot [attend l’entrée utilisateur](bot-framework-rest-connector-add-input-hints.md). Elle spécifie la propriété `speak` à l’aide du format <a href="https://msdn.microsoft.com/en-us/library/hh378377(v=office.14).aspx" target="_blank">SSML</a> pour indiquer que l’expression « bien sûr » doit être prononcée en y mettant l’accent. Dans cet exemple de requête, `https://smba.trafficmanager.net/apis` représente l’URI de base, qui peut être différent de celui des requêtes émises par votre bot. Pour plus d’informations sur la définition de l’URI de base, consultez l’article [Informations de référence sur l’API](bot-framework-rest-connector-api-reference.md#base-uri).

```http
POST https://smba.trafficmanager.net/apis/v3/conversations/abcd1234/activities/5d5cdc723
Authorization: Bearer ACCESS_TOKEN
Content-Type: application/json
```

```json
{
    "type": "message",
    "from": {
        "id": "12345678",
        "name": "sender's name"
    },
    "conversation": {
        "id": "abcd1234",
        "name": "conversation's name"
   },
   "recipient": {
        "id": "1234abcd",
        "name": "recipient's name"
    },
    "text": "Are you sure that you want to cancel this transaction?",
    "speak": "Are you <emphasis level='moderate'>sure</emphasis> that you want to cancel this transaction?",
    "inputHint": "expectingInput",
    "replyToId": "5d5cdc723"
}
```

## <a name="input-hints"></a>Conseils de saisie

Lorsque vous envoyez un message sur le canal de reconnaissance vocale, vous pouvez tenter d’influencer l’état du microphone du client en spécifiant un conseil de saisie pour indiquer si votre bot accepte, attend ou ignore l’entrée de l’utilisateur. Pour plus d’informations, consultez [Ajouter des conseils de saisie aux messages](bot-framework-rest-connector-add-input-hints.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Créer des messages](bot-framework-rest-connector-create-messages.md)
- [Envoyer et recevoir des messages](bot-framework-rest-connector-send-and-receive-messages.md)
- [Ajouter des conseils de saisie aux messages](bot-framework-rest-connector-add-input-hints.md)
- <a href="https://msdn.microsoft.com/en-us/library/hh378377(v=office.14).aspx" target="_blank">Speech Synthesis Markup Language (SSML)</a>

[Activity]: bot-framework-rest-connector-api-reference.md#activity-object
