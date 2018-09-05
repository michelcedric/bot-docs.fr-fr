---
title: Ajouter des pièces jointes de cartes enrichies aux messages | Microsoft Docs
description: Découvrez comment envoyer des cartes enrichies attractives et interactives à l’aide du Kit de développement logiciel (SDK) Bot Builder pour Node.js.
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 7f94ea05fcccfe7bdeb1dec187d735cef28b1d7c
ms.sourcegitcommit: 2dc75701b169d822c9499e393439161bc87639d2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/24/2018
ms.locfileid: "42905393"
---
# <a name="add-rich-card-attachments-to-messages"></a>Ajouter des pièces jointes de cartes enrichies aux messages

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]


> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-add-rich-card-attachments.md)
> - [Node.JS](../nodejs/bot-builder-nodejs-send-rich-cards.md)
> - [REST](../rest-api/bot-framework-rest-connector-add-rich-cards.md)

Plusieurs canaux, tels que Skype et Facebook, prennent en charge l’envoi aux utilisateurs de cartes graphiques enrichies comportant des boutons interactifs sur lesquels l’utilisateur doit cliquer pour exécuter une action. Le Kit de développement logiciel (SDK) fournit plusieurs classes de générateur de cartes et de messages permettant de créer et d’envoyer des cartes. Le service Bot Framework Connector affiche ces cartes à l’aide du schéma natif du canal, prenant ainsi en charge une communication multiplateforme. Si le canal ne prend pas en charge ces cartes, comme SMS, Bot Framework tentera d’offrir une expérience acceptable aux utilisateurs. 

## <a name="types-of-rich-cards"></a>Types de cartes enrichies 
Bot Framework prend actuellement en charge huit types de cartes enrichies : 

| Type de carte | Description |
|------|------|
| <a href="/adaptive-cards/get-started/bots">Carte adaptative</a> | Carte personnalisable pouvant inclure n’importe quelle combinaison de texte, données vocales, images, boutons et champs d’entrée.  Consultez l’article sur la [prise en charge de ces cartes par canal](/adaptive-cards/get-started/bots#channel-status). |
| [Carte d’animation][animationCard] | Carte pouvant lire des images GIF animées ou de courtes vidéos. |
| [Carte audio][audioCard] | Carte pouvant lire un fichier audio. |
| [Carte de bannière][heroCard] | Carte contenant généralement une image de grande taille, un ou plusieurs boutons, ainsi que du texte. |
| [Carte de miniature][thumbnailCard] | Carte contenant généralement une image miniature, un ou plusieurs boutons, ainsi que du texte.|
| [Carte de reçu][receiptCard] | Carte permettant à un robot de fournir un reçu à l’utilisateur. Elle contient généralement la liste des articles à inclure sur le reçu, la taxe et le total, ainsi que du texte. |
| [Carte de connexion][signinCard] | Carte permettant à un bot de demander à un utilisateur de se connecter. Elle contient généralement du texte et un ou plusieurs boutons sur lesquels l’utilisateur peut cliquer pour lancer le processus de connexion. |
| [Carte vidéo][videoCard] | Carte pouvant lire des vidéos. |

## <a name="send-a-carousel-of-hero-cards"></a>Envoyer un carrousel de cartes de bannière
L’exemple ci-après concerne un bot d’un fabricant de tee-shirts fictif et indique comment envoyer un carrousel de cartes en réponse à l’énoncé par l’utilisateur de la phrase « show shirts » (afficher les tee-shirts). 

```javascript
// Create your bot with a function to receive messages from the user
// Create bot and default message handler
var bot = new builder.UniversalBot(connector, function (session) {
    session.send("Hi... We sell shirts. Say 'show shirts' to see our products.");
});

// Add dialog to return list of shirts available
bot.dialog('showShirts', function (session) {
    var msg = new builder.Message(session);
    msg.attachmentLayout(builder.AttachmentLayout.carousel)
    msg.attachments([
        new builder.HeroCard(session)
            .title("Classic White T-Shirt")
            .subtitle("100% Soft and Luxurious Cotton")
            .text("Price is $25 and carried in sizes (S, M, L, and XL)")
            .images([builder.CardImage.create(session, 'http://petersapparel.parseapp.com/img/whiteshirt.png')])
            .buttons([
                builder.CardAction.imBack(session, "buy classic white t-shirt", "Buy")
            ]),
        new builder.HeroCard(session)
            .title("Classic Gray T-Shirt")
            .subtitle("100% Soft and Luxurious Cotton")
            .text("Price is $25 and carried in sizes (S, M, L, and XL)")
            .images([builder.CardImage.create(session, 'http://petersapparel.parseapp.com/img/grayshirt.png')])
            .buttons([
                builder.CardAction.imBack(session, "buy classic gray t-shirt", "Buy")
            ])
    ]);
    session.send(msg).endDialog();
}).triggerAction({ matches: /^(show|list)/i });
```
Cet exemple utilise la classe [Message][Message] pour générer un carrousel.  
Ce carrousel est composé d’une liste de classes [HeroCard][heroCard] qui contiennent une image, un texte et un bouton unique déclenchant l’achat de l’article.  
La sélection du bouton **Buy** (Acheter) déclenche l’envoi d’un message ; nous devons donc ajouter un second dialogue pour intercepter le clic sur le bouton. 

## <a name="handle-button-input"></a>Gérer l’entrée de bouton

Le dialogue `buyButtonClick` est déclenché chaque fois que le bot reçoit un message commençant par le terme « buy » (acheter) ou « add » (ajouter), suivi d’un élément contenant le mot « shirt ». Le dialogue commence par utiliser deux expressions régulières pour rechercher la couleur et la taille de tee-shirt facultative demandées par l’utilisateur.
Ce surcroît de flexibilité vous permet de prendre en charge à la fois les clics de bouton et les messages en langage naturel de la part de l’utilisateur, du type « ajouter un tee-shirt gris large à mon panier d’achat ».
Si la couleur est valide, mais que la taille est inconnue, le bot invite l’utilisateur à choisir une taille dans la liste avant d’ajouter l’article au panier d’achat. Une fois que le bot dispose de toutes les informations dont il a besoin, il place l’article dans un panier d’achat rendu persistant à l’aide de l’élément **session.userData**, puis envoie un message de confirmation à l’utilisateur.

```javascript
// Add dialog to handle 'Buy' button click
bot.dialog('buyButtonClick', [
    function (session, args, next) {
        // Get color and optional size from users utterance
        var utterance = args.intent.matched[0];
        var color = /(white|gray)/i.exec(utterance);
        var size = /\b(Extra Large|Large|Medium|Small)\b/i.exec(utterance);
        if (color) {
            // Initialize cart item
            var item = session.dialogData.item = { 
                product: "classic " + color[0].toLowerCase() + " t-shirt",
                size: size ? size[0].toLowerCase() : null,
                price: 25.0,
                qty: 1
            };
            if (!item.size) {
                // Prompt for size
                builder.Prompts.choice(session, "What size would you like?", "Small|Medium|Large|Extra Large");
            } else {
                //Skip to next waterfall step
                next();
            }
        } else {
            // Invalid product
            session.send("I'm sorry... That product wasn't found.").endDialog();
        }   
    },
    function (session, results) {
        // Save size if prompted
        var item = session.dialogData.item;
        if (results.response) {
            item.size = results.response.entity.toLowerCase();
        }

        // Add to cart
        if (!session.userData.cart) {
            session.userData.cart = [];
        }
        session.userData.cart.push(item);

        // Send confirmation to users
        session.send("A '%(size)s %(product)s' has been added to your cart.", item).endDialog();
    }
]).triggerAction({ matches: /(buy|add)\s.*shirt/i });
```

<!-- 

> [!NOTE]
> When sending a message that contains images, keep in mind that some channels download images before displaying a message to the user.   
> As a result, a message containing an image followed immediately by a message without images may sometimes be flipped in the user's feed.
> For information on how to avoid messages being sent out of order, see [Message ordering][MessageOrder].  

-->
## <a name="add-a-message-delay-for-image-downloads"></a>Ajouter un délai entre les messages pour les téléchargements d’images
Certains canaux tentent de télécharger les images avant d’afficher un message à l’intention de l’utilisateur, de sorte que si vous envoyez un message contenant une image, immédiatement suivi d’un message dépourvu d’images, ces messages vous apparaîtront parfois inversés dans le flux utilisateur. Pour réduire ce risque, vous pouvez essayer de vous assurer que vos images proviennent de réseaux de distribution de contenu (CDN) et éviter d’utiliser des images trop volumineuses. Dans les cas extrêmes, vous devrez peut-être même insérer un délai de 1 à 2 secondes entre le message contenant l’image et le message suivant. Vous pouvez faire en sorte que ce délai semble un peu plus naturel à l’utilisateur en appelant la méthode **session.sendTyping()** afin d’envoyer un indicateur de saisie avant l’initialisation de ce délai. 

<!-- 
To learn more about sending a typing indicator, see [How to send a typing indicator](bot-builder-nodejs-send-typing-indicator.md).
-->

Bot Framework implémente une traitement par lot pour tenter d’empêcher que les différents messages émanant du bot ne s’affichent dans le désordre. <!-- Unfortunately, not all channels can guarantee this. --> Lorsque votre bot envoie plusieurs réponses à l’utilisateur, ces différents messages sont automatiquement regroupés dans un lot et remis à l’utilisateur sous forme groupée dans le but de conserver l’ordre d’origine des messages. Ce traitement par lot automatique marque une pause de 250 ms par défaut après chaque appel de la méthode **session.send()** avant de procéder à l’appel suivant de la méthode **send()**.

Le délai du traitement par lot des messages est configurable. Pour désactiver la logique de traitement par lot automatique du Kit de développement logiciel (SDK), définissez le délai par défaut sur une valeur élevée, puis appelez manuellement la méthode **sendBatch()** avec un rappel à appeler une fois que le lot a été remis.

## <a name="send-an-adaptive-card"></a>Envoyer une carte adaptative

La carte adaptive peut inclure n’importe quelle combinaison de texte, données vocales, images, boutons et champs d’entrée. Les cartes adaptatives sont créées à l’aide du format JSON spécifié sur le site <a href="http://adaptivecards.io" target="_blank">Adaptive Cards</a> (Cartes adaptatives), ce qui vous donne un contrôle total sur le contenu et le format de la carte. 

Pour créer une carte adaptative à l’aide de Node.js, tirez profit des informations fournies par le site <a href="http://adaptivecards.io" target="_blank">Adaptive Cards</a> (Cartes adaptatives) pour comprendre le schéma de carte adaptative, explorer les éléments de carte adaptative et découvrir des exemples JSON qui permettent de créer des cartes présentant différents types de compositions et niveaux de complexité. En outre, vous pouvez utiliser le visualiseur interactif pour concevoir des charges utiles de carte adaptative et afficher un aperçu de la sortie des cartes.

Cet exemple de code indique comment créer un message contenant une carte adaptative pour un rappel du Calendrier : 

[!code-javascript[Add Adaptive Card attachment](../includes/code/node-send-card-buttons.js#addAdaptiveCardAttachment)]

La carte résultante contient trois blocs de texte, un champ d’entrée (liste de choix) et trois boutons :

![Carte adaptative de rappel du Calendrier](../media/adaptive-card-reminder.png)

## <a name="additional-resources"></a>Ressources supplémentaires

* [Aperçu des fonctionnalités du bot avec l’inspecteur de canaux][inspector]
* <a href="http://adaptivecards.io" target="_blank">Adaptive Cards</a> (Cartes adaptatives)
* [AnimationCard][animationCard]
* [AudioCard][audioCard]
* [HeroCard][heroCard]
* [ThumbnailCard][thumbnailCard]
* [ReceiptCard][receiptCard]
* [SigninCard][signinCard]
* [VideoCard][videoCard]
* [Message][Message]
* [Envoyer et recevoir des pièces jointes](bot-builder-nodejs-send-receive-attachments.md)

[MessageOrder]: bot-builder-nodejs-manage-conversation-flow.md#message-ordering
[Message]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.message
[IMessage]: http://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.imessage

[animationCard]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.animationcard.html 

[audioCard]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.audiocard.html 

[heroCard]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.herocard.html

[thumbnailCard]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.thumbnailcard.html 

[receiptCard]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.receiptcard.html 

[signinCard]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.signincard.html 

[videoCard]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.videocard.html

[inspector]: ../bot-service-channel-inspector.md
