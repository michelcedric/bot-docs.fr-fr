---
title: Ajouter des médias aux messages | Microsoft Docs
description: Découvrez comment ajouter des médias à l’aide du kit de développement logiciel (SDK) Bot Builder.
keywords: médias, messages, images, audio, vidéo, fichiers, MessageFactory, cartes enrichies, messages, cartes adaptatives, carte de héros, actions suggérées
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/17/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: fd908335c69aab7c8b68925b8ecdece79e89ab4b
ms.sourcegitcommit: f7a8f05fc05ff4a7212a437d540485bf68831604
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/21/2018
ms.locfileid: "53735959"
---
# <a name="add-media-to-messages"></a>Ajouter des médias aux messages

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Les messages échangés entre l’utilisateur et le bot peuvent contenir des pièces jointes multimédia, comme des images, des vidéos, des pistes audio et des fichiers. Le Kit de développement logiciel (SDK) Bot Builder prend en charge l’envoi de messages enrichis à l’utilisateur. Pour déterminer le type des messages enrichis pris en charge sur un canal (Facebook, Skype, Slack, etc.), consultez la documentation associée pour en savoir plus sur les limitations. Reportez-vous à [l’expérience utilisateur de conception](../bot-service-design-user-experience.md) pour obtenir la liste des cartes disponibles. 

## <a name="send-attachments"></a>Envoyer des pièces jointes

Pour envoyer le contenu de l’utilisateur comme une image ou une vidéo, vous pouvez ajouter une pièce jointe ou une liste de pièces jointes à un message.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

La propriété `Attachments` de l’objet `Activity` contient un tableau d’objets `Attachment` qui représentent les pièces jointes multimédias et cartes enrichies attachées au message. Pour ajouter une pièce jointe multimédia à un message, créez un objet `Attachment` pour l’activité `message` et définissez les propriétés `ContentType`, `ContentUrl` et `Name`. La propriété `Attachments` de l’objet `Activity` contient un tableau d’objets `Attachment` qui représentent les pièces jointes multimédias et cartes enrichies attachées au message. Pour ajouter une pièce jointe multimédia à un message, utilisez la méthode `Attachment` afin de créer un objet `Attachment` pour l’activité `message`, puis définissez les propriétés `ContentType`, `ContentUrl` et `Name`. Le code source affiché ici repose sur l’exemple [Gestion des pièces jointes](https://aka.ms/bot-attachments-sample-code). 

```csharp
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Schema;

var reply = turnContext.Activity.CreateReply();

// Create an attachment.
var attachment = new Attachment
    {
        ContentUrl = "imageUrl.png",
        ContentType = "image/png",
        Name = "imageName",
    };

// Add the attachment to our reply.
reply.Attachments = new List<Attachment>() { attachment };

// Send the activity to the user.
await turnContext.SendActivityAsync(reply, cancellationToken);
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Le code source affiché ici repose sur l’exemple [Gestion des pièces jointes JS](https://aka.ms/bot-attachments-sample-code-js).
Pour envoyer à l’utilisateur un seul élément de contenu comme une image ou une vidéo, vous pouvez envoyer les médias contenus dans une URL :

```javascript
const { ActionTypes, ActivityTypes, CardFactory } = require('botbuilder');

// Call function to get an attachment.
const reply = { type: ActivityTypes.Message };
reply.attachments = [this.getInternetAttachment()];
reply.text = 'This is an internet attachment.';
// Send the activity to the user.
await turnContext.sendActivity(reply);

/* function getInternetAttachment - Returns an attachment to be sent to the user from a HTTPS URL */
getInternetAttachment() {
        return {
            name: 'imageName.png',
            contentType: 'image/png',
            contentUrl: 'imageUrl.png'}
}
```

---

Si la pièce jointe est une image, un contenu audio ou une vidéo, le service Connector communique les données de pièce jointe au canal de manière à permettre au [canal](bot-builder-channeldata.md) d’afficher cette pièce jointe dans la conversation. Si la pièce jointe est un fichier, l’URL du fichier s’affichera en tant que lien hypertexte dans la conversation.

## <a name="send-a-hero-card"></a>Envoyer une carte de héros

Outre les simples pièces jointes image ou vidéo, vous pouvez attacher une **carte de héros**, qui vous permet de combiner des images et des boutons dans un seul objet à envoyer à l’utilisateur.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Pour composer un message avec un bouton et une carte de héros, vous pouvez joindre `HeroCard` à un message. Le code source affiché ici repose sur l’exemple [Gestion des pièces jointes](https://aka.ms/bot-attachments-sample-code). 

```csharp
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Schema;

var reply = turnContext.Activity.CreateReply();

// Create a HeroCard with options for the user to choose to interact with the bot.
var card = new HeroCard
{
    Text = "You can upload an image or select one of the following choices",
    Buttons = new List<CardAction>()
    {
        new CardAction(ActionTypes.ImBack, title: "1. Inline Attachment", value: "1"),
        new CardAction(ActionTypes.ImBack, title: "2. Internet Attachment", value: "2"),
        new CardAction(ActionTypes.ImBack, title: "3. Uploaded Attachment", value: "3"),
    },
};

// Add the card to our reply.
reply.Attachments = new List<Attachment>() { card.ToAttachment() };

await turnContext.SendActivityAsync(reply, cancellationToken);
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Pour composer un message avec un bouton et une carte de héros, vous pouvez joindre `HeroCard` à un message. Le code source affiché ici repose sur l’exemple [Gestion des pièces jointes JS](https://aka.ms/bot-attachments-sample-code-js) :

```javascript
const { ActionTypes, ActivityTypes, CardFactory } = require('botbuilder');
// build buttons to display.
const buttons = [
            { type: ActionTypes.ImBack, title: '1. Inline Attachment', value: '1' },
            { type: ActionTypes.ImBack, title: '2. Internet Attachment', value: '2' },
            { type: ActionTypes.ImBack, title: '3. Uploaded Attachment', value: '3' }
];

// construct hero card.
const card = CardFactory.heroCard('', undefined,
buttons, { text: 'You can upload an image or select one of the following choices.' });

// add card to Activity.
const reply = { type: ActivityTypes.Message };
reply.attachments = [card];

// Send hero card to the user.
await turnContext.sendActivity(reply);
```

---

## <a name="process-events-within-rich-cards"></a>Traiter des événements dans les cartes enrichies

Pour traiter les événements dans les cartes enrichies, utilisez les objets _card action_ pour spécifier ce qui doit se produire quand l’utilisateur clique sur un bouton ou appuie sur une section de la carte. Chaque action de la carte a un _type_ et une _valeur_.

Pour fonctionner correctement, assignez un type d’action à chaque élément interactif de la carte. Ce tableau liste et décrit les types d’actions disponibles, et ce qui doit se trouver dans la propriété de valeur associée.

| type | Description | Valeur |
| :---- | :---- | :---- |
| openUrl | Ouvre une URL dans le navigateur intégré. | URL à ouvrir. |
| imBack | Envoie un message au bot et publie une réponse visible dans la conversation. | Texte du message à envoyer. |
| postBack | Envoie un message au bot et ne publie pas une réponse visible dans la conversation. | Texte du message à envoyer. |
| call | Procède à un appel téléphonique. | Destination d’un appel téléphonique au format suivant : `tel:123123123123`. |
| playAudio | Lit le contenu audio. | URL du contenu audio à lire. |
| playVideo | Lit une vidéo. | URL de la vidéo à lire. |
| showImage | Affiche une image. | URL de l’image à afficher. |
| downloadFile | Télécharge un fichier. | URL du fichier à télécharger. |
| signin | Lance un processus de connexion OAuth. | URL du flux OAuth à lancer. |

## <a name="hero-card-using-various-event-types"></a>Carte de héros utilisant différents types d’événements

Le code suivant montre des exemples d’utilisation de différents événements de carte enrichie.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Schema;

var reply = turnContext.Activity.CreateReply();

var card = new HeroCard
{
    Buttons = new List<CardAction>()
    {
        new CardAction(title: "Much Quieter", type: ActionTypes.PostBack, value: "Shh! My Bot friend hears me."),
        new CardAction(ActionTypes.OpenUrl, title: "Azure Bot Service", value: "https://azure.microsoft.com/en-us/services/bot-service/"),
    },
};

```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
const {ActionTypes} = require("botbuilder");

const hero = MessageFactory.attachment(
    CardFactory.heroCard(
        'Holler Back Buttons',
        ['https://example.com/whiteShirt.jpg'],
        [{
            type: ActionTypes.ImBack,
            title: 'ImBack',
            value: 'You can ALL hear me! Shout Out Loud'
        },
        {
            type: ActionTypes.PostBack,
            title: 'PostBack',
            value: 'Shh! My Bot friend hears me. Much Quieter'
        },
        {
            type: ActionTypes.OpenUrl,
            title: 'OpenUrl',
            value: 'https://en.wikipedia.org/wiki/{cardContent.Key}'
        }]
    )
);

await context.sendActivity(hero);

```

---

## <a name="send-an-adaptive-card"></a>Envoyer une carte adaptative
Vous pouvez utiliser aussi bien les cartes adaptatives que MessageFactory pour envoyer des messages enrichis contenant du texte, des images, des vidéos, du contenu audio et des fichiers afin de communiquer avec les utilisateurs. Toutefois, il existe certaines différences entre ces deux méthodes. 

Pour commencer, seuls certains canaux prennent en charge les cartes adaptatives et ne le font parfois que partiellement. Par exemple, si vous envoyez une carte adaptative dans Facebook, les boutons sont inopérants, alors que les textes et images fonctionnent parfaitement. MessageFactory est une simple classe d’assistance au sein du Kit de développement logiciel (SDK) Bot Builder qui est destinée à automatiser la procédure de création à votre intention et est prise en charge par la plupart des canaux. 

Ensuite, les cartes adaptatives remettent les messages au format carte, et le canal détermine la disposition de la carte. Le format des messages remis par MessageFactory dépend du canal et ne correspond pas nécessairement au format carte, sauf si la pièce jointe intègre une carte adaptative. 

Pour connaître les dernières informations sur la prise en charge par les canaux des cartes adaptatives, consultez <a href="http://adaptivecards.io/visualizer/">Adaptive Cards Visualizer</a>.

Pour utiliser des cartes adaptatives, veillez à ajouter le package NuGet `Microsoft.AdaptiveCards`. 


> [!NOTE]
> Vous devez tester cette fonctionnalité avec les canaux que votre bot utilise pour déterminer s’ils prennent en charge les cartes adaptatives.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Le code source affiché ici repose sur l’exemple [Utilisation des cartes adaptatives](https://aka.ms/bot-adaptive-cards-sample-code) :

```csharp
using AdaptiveCards;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Schema;

// Creates an attachment that contains an adaptive card
// filePath is the path to JSON file
private static Attachment CreateAdaptiveCardAttachment(string filePath)
{
    var adaptiveCardJson = File.ReadAllText(filePath);
    var adaptiveCardAttachment = new Attachment()
    {
        ContentType = "application/vnd.microsoft.card.adaptive",
        Content = JsonConvert.DeserializeObject(adaptiveCardJson),
    };
    return adaptiveCardAttachment;
}

// Create adaptive card and attach it to the message 
var cardAttachment = CreateAdaptiveCardAttachment(adaptiveCardJsonFilePath);
var reply = turnContext.Activity.CreateReply();
reply.Attachments = new List<Attachment>() { cardAttachment };

await turnContext.SendActivityAsync(reply, cancellationToken);
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Le code source affiché ici repose sur l’exemple [Utilisation des cartes adaptatives JS](https://aka.ms/bot-adaptive-cards-js-sample-code) :

```javascript
const { BotFrameworkAdapter } = require('botbuilder');

// Import AdaptiveCard content.
const FlightItineraryCard = require('./resources/FlightItineraryCard.json');
const ImageGalleryCard = require('./resources/ImageGalleryCard.json');
const LargeWeatherCard = require('./resources/LargeWeatherCard.json');
const RestaurantCard = require('./resources/RestaurantCard.json');
const SolitaireCard = require('./resources/SolitaireCard.json');

// Create array of AdaptiveCard content, this will be used to send a random card to the user.
const CARDS = [
    FlightItineraryCard,
    ImageGalleryCard,
    LargeWeatherCard,
    RestaurantCard,
    SolitaireCard
];
// Select a random card to send.
const randomlySelectedCard = CARDS[Math.floor((Math.random() * CARDS.length - 1) + 1)];
// Send adaptive card.
await context.sendActivity({
      text: 'Here is an Adaptive Card:',
       attachments: [CardFactory.adaptiveCard(randomlySelectedCard)]
});
```

---

## <a name="send-a-carousel-of-cards"></a>Envoyer un carrousel de cartes

Les messages peuvent également inclure plusieurs pièces jointes dans une disposition carrousel, qui place les pièces jointes côte à côte et permet à l’utilisateur de faire défiler latéralement.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Schema;

// Create the activity and attach a set of Hero cards.
var activity = MessageFactory.Carousel(
    new Attachment[]
    {
        new HeroCard(
            title: "title1",
            images: new CardImage[] { new CardImage(url: "imageUrl1.png") },
            buttons: new CardAction[]
            {
                new CardAction(title: "button1", type: ActionTypes.ImBack, value: "item1")
            })
        .ToAttachment(),
        new HeroCard(
            title: "title2",
            images: new CardImage[] { new CardImage(url: "imageUrl2.png") },
            buttons: new CardAction[]
            {
                new CardAction(title: "button2", type: ActionTypes.ImBack, value: "item2")
            })
        .ToAttachment(),
        new HeroCard(
            title: "title3",
            images: new CardImage[] { new CardImage(url: "imageUrl3.png") },
            buttons: new CardAction[]
            {
                new CardAction(title: "button3", type: ActionTypes.ImBack, value: "item3")
            })
        .ToAttachment()
    });

// Send the activity as a reply to the user.
await context.SendActivity(activity);
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// require MessageFactory and CardFactory from botbuilder.
const {MessageFactory, CardFactory} = require('botbuilder');

//  init message object
let messageWithCarouselOfCards = MessageFactory.carousel([
    CardFactory.heroCard('title1', ['imageUrl1'], ['button1']),
    CardFactory.heroCard('title2', ['imageUrl2'], ['button2']),
    CardFactory.heroCard('title3', ['imageUrl3'], ['button3'])
]);

await context.sendActivity(messageWithCarouselOfCards);
```

---

<!-- TODO: Add a media card, such as video or audion. Revisit which examples we put here and link to the 06 through 08 samples. -->

## <a name="additional-resources"></a>Ressources supplémentaires

Pour plus d’informations sur le schéma de la carte, consultez le [schéma de carte Bot Framework](https://aka.ms/botSpecs-cardSchema).

Un exemple de code est disponible ici pour les cartes : [C#](https://aka.ms/bot-cards-sample-code)/[JS](https://aka.ms/bot-cards-js-sample-code), cartes adaptatives : [C#](https://aka.ms/bot-adaptive-cards-sample-code)/[JS](https://aka.ms/bot-adaptive-cards-js-sample-code), pièces jointes : [C#](https://aka.ms/bot-attachments-sample-code)/[JS](https://aka.ms/bot-attachments-sample-code-js) et actions suggérées : [C#](https://aka.ms/SuggestedActionsCSharp)/[JS](https://aka.ms/SuggestedActionsJS).
Pour obtenir plus d’exemples, consultez le référentiel d’exemples Bot Builder sur [GitHub](https://aka.ms/bot-samples-readme).
