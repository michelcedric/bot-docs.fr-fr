---
title: Envoyer et recevoir des pièces jointes | Microsoft Docs
description: Découvrez comment envoyer et recevoir des messages contenant des pièces jointes à l’aide du kit SDK Bot Builder pour Node.js.
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 4203e9d7a9c5c8e6ab068def879747a4c6158367
ms.sourcegitcommit: 2dc75701b169d822c9499e393439161bc87639d2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/24/2018
ms.locfileid: "42904043"
---
# <a name="send-and-receive-attachments"></a>Envoyer et recevoir des pièces jointes

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-add-media-attachments.md)
> - [Node.JS](../nodejs/bot-builder-nodejs-send-receive-attachments.md)
> - [REST](../rest-api/bot-framework-rest-connector-add-media-attachments.md)

Un échange de messages entre l’utilisateur et le bot peut contenir des pièces jointes multimédias, telles que des images, des vidéos, de l’audio et des fichiers. Les types de pièces jointes qui peuvent être envoyés varient selon le canal, mais voici les types de base :

* **Multimédia et fichiers** : vous pouvez envoyer des fichiers tels que des images, de l’audio et des vidéos en affectant à **contentType** le type MIME de [l’objet IAttachment][IAttachment], puis en passant un lien vers le fichier dans **contentUrl**.
* **Cartes** : vous pouvez envoyer un ensemble complet de cartes visuelles <!-- and custom keyboards --> en affectant à **contentType** le type de carte souhaité, puis en passant le JSON pour la carte. Si vous utilisez l’une des classes du générateur de cartes enrichies comme **HeroCard**, la pièce jointe est automatiquement renseignée pour vous. Consultez [Envoyer une carte enrichie](bot-builder-nodejs-send-rich-cards.md) pour obtenir un exemple.

## <a name="add-a-media-attachment"></a>Ajouter une pièce jointe multimédia
L’objet du message est censé être une instance d’un [IMessage][IMessage] et il est particulièrement utile d’envoyer à l’utilisateur un message en tant qu’objet quand vous souhaitez inclure une pièce jointe comme une image. Utilisez la méthode [session.send()][SessionSend] pour envoyer des messages sous la forme d’un objet JSON. 

## <a name="example"></a>Exemples

L’exemple suivant vérifie si l’utilisateur a envoyé une pièce jointe et, le cas échéant, il renvoie n’importe quelle image contenue dans la pièce jointe. Vous pouvez le tester avec Bot Framework Emulator en envoyant une image à votre bot.

```javascript
// Create your bot with a function to receive messages from the user
var bot = new builder.UniversalBot(connector, function (session) {
    var msg = session.message;
    if (msg.attachments && msg.attachments.length > 0) {
     // Echo back attachment
     var attachment = msg.attachments[0];
        session.send({
            text: "You sent:",
            attachments: [
                {
                    contentType: attachment.contentType,
                    contentUrl: attachment.contentUrl,
                    name: attachment.name
                }
            ]
        });
    } else {
        // Echo back users text
        session.send("You said: %s", session.message.text);
    }
});
```
## <a name="additional-resources"></a>Ressources supplémentaires

* [Aperçu des fonctionnalités du bot avec l’inspecteur de canaux][inspector]
* [IMessage][IMessage]
* [Envoyer une carte enrichie][SendRichCard]
* [session.send][SessionSend]

[IMessage]: http://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.imessage
[SendRichCard]: bot-builder-nodejs-send-rich-cards.md
[SessionSend]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session.html#send
[IAttachment]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.iattachment.html
[inspector]: ../bot-service-channel-inspector.md
