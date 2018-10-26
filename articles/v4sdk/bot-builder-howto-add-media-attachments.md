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
ms.date: 09/10/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 35ec8968493eb024b2724d0729a8a2cd6e14ba82
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2018
ms.locfileid: "50000367"
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
reply.attachments = [card];

// Send hero card to the user.
await turnContext.sendActivity(reply);
```
---

## <a name="process-events-within-rich-cards"></a>Traiter des événements dans les cartes enrichies

Pour traiter les événements dans les cartes enrichies, utilisez les objets _card action_ pour spécifier ce qui doit se produire quand l’utilisateur clique sur un bouton ou appuie sur une section de la carte.

Pour fonctionner correctement, assignez un type d’action à chaque élément interactif de la carte. Ce tableau répertorie les valeurs valides pour la propriété type d’un objet card action et décrit le contenu attendu de la propriété value pour chaque type.

| type | Valeur |
| :---- | :---- |
| openUrl | URL à ouvrir dans le navigateur intégré. Répond à Appuyer ou Cliquer en ouvrant l’URL. |
| imBack | Texte du message à envoyer au bot (de la part de l’utilisateur qui a cliqué sur le bouton ou appuyé sur la carte). Ce message (de l’utilisateur au bot) sera visible par tous les participants à la conversation par le biais de l’application cliente qui héberge la conversation. |
| postBack | Texte du message à envoyer au bot (de la part de l’utilisateur qui a cliqué sur le bouton ou appuyé sur la carte). Certaines applications clientes peuvent afficher ce texte dans le flux de messages, où il sera visible par tous les participants à la conversation. |
| appel | Destination d’un appel téléphonique au format suivant : `tel:123123123123`. Répond à Appuyer ou Cliquer en lançant un appel.|
| playAudio | URL du fichier audio à lire. Répond à Appuyer ou Cliquer en lisant le fichier audio. |
| playVideo | URL du fichier vidéo à lire. Répond à Appuyer ou Cliquer en lisant le fichier vidéo. |
| showImage | URL de l’image à afficher. Répond à Appuyer ou Cliquer en affichant l’image. |
| downloadFile | URL du fichier à télécharger.  Répond à Appuyer ou Cliquer en téléchargeant le fichier. |
| signin | URL du flux OAuth à démarrer. Répond à Appuyer ou Cliquer en démarrant la connexion. |

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

## <a name="additional-resources"></a>Ressources supplémentaires
Vous trouverez ici le code source pour les cartes ([C#](https://aka.ms/bot-cards-sample-code)/[JS](https://aka.ms/bot-cards-js-sample-code)), les cartes adaptatives ([C#](https://aka.ms/bot-adaptive-cards-sample-code)/[JS](https://aka.ms/bot-adaptive-cards-js-sample-code)), les pièces jointes ([C#](https://aka.ms/bot-attachments-sample-code)/[JS](https://aka.ms/bot-attachments-sample-code-js)) et les actions suggérées ([C#](https://aka.ms/SuggestedActionsCSharp)/[JS](https://aka.ms/SuggestedActionsJS)). Pour obtenir plus d’exemples, consultez le référentiel d’exemples Bot Builder sur [GitHub](https://github.com/Microsoft/BotBuilder-Samples).
