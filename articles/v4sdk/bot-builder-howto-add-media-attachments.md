---
title: Ajouter des médias aux messages | Microsoft Docs
description: Découvrez comment ajouter des médias à l’aide du kit de développement logiciel (SDK) Bot Builder.
keywords: média, messages, images, audio, vidéo, fichiers, MessageFactory, messages enrichis
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 08/24/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 5883b31df95da26fa0432f4cfe195f12fc3089ad
ms.sourcegitcommit: 86ddf3ebe6cc3385d1c4d30b971ac9c3e1fc5a77
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/27/2018
ms.locfileid: "43055993"
---
# <a name="add-media-to-messages"></a>Ajouter des médias aux messages

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Un échange de messages entre l’utilisateur et le bot peut contenir des médias joints, tels que des images, des vidéo, de l’audio et des fichiers. Le Kit de développement logiciel Bot Builder inclut une nouvelle classe `Microsoft.Bot.Builder.MessageFactory`, conçue pour faciliter la tâche d’envoi des messages enrichis à l’utilisateur.

## <a name="send-attachments"></a>Envoyer des pièces jointes

Pour envoyer le contenu de l’utilisateur comme une image ou une vidéo, vous pouvez ajouter une pièce jointe ou une liste de pièces jointes à un message.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

La propriété `Attachments` de l’objet `Activity` contient un tableau d’objets `Attachment` qui représentent les pièces jointes multimédias et cartes enrichies attachées au message. Pour ajouter une pièce jointe multimédia à un message, créez un objet `Attachment` pour l’activité `message` et définissez les propriétés `ContentType`, `ContentUrl`, et `Name`. 

```csharp
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Schema;

// Create the activity and add an attachment.
var activity = MessageFactory.Attachment(
    new Attachment { ContentUrl = "imageUrl.png", ContentType = "image/png" }
);

// Send the activity to the user.
await context.SendActivity(activity);
```

La méthode `Attachment` de fabrique de messages peut également envoyer une liste de pièces jointes, empilées les unes sur les autres.

```csharp
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Schema;

// Create the activity and add an attachment.
var activity = MessageFactory.Attachment(new Attachment[]
{
    new Attachment { ContentUrl = "imageUrl1.png", ContentType = "image/png" },
    new Attachment { ContentUrl = "imageUrl2.png", ContentType = "image/png" },
    new Attachment { ContentUrl = "imageUrl3.png", ContentType = "image/png" }
});

// Send the activity to the user.
await context.SendActivity(activity);
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Pour envoyer à l’utilisateur un seul élément de contenu comme une image ou une vidéo, vous pouvez envoyer les médias contenus dans une URL :

```javascript
const {MessageFactory} = require('botbuilder');
let imageOrVideoMessage = MessageFactory.contentUrl('imageUrl.png', 'image/jpeg')

// Send the activity to the user.
await context.sendActivity(imageOrVideoMessage);
```

Pour envoyer une liste de pièces jointes, empilées les unes sur les autres : <!-- TODO: Convert the hero cards to image attachments in this example. -->

```javascript
// require MessageFactory and CardFactory from botbuilder.
const {MessageFactory, CardFactory} = require('botbuilder');

let messageWithCarouselOfCards = MessageFactory.list([
    CardFactory.heroCard('title1', ['imageUrl1'], ['button1']),
    CardFactory.heroCard('title2', ['imageUrl2'], ['button2']),
    CardFactory.heroCard('title3', ['imageUrl3'], ['button3'])
]);

await context.sendActivity(messageWithCarouselOfCards);
```

---

Si la pièce jointe est une image ou un fichier audio ou vidéo, le service du connecteur communiquera les données de pièce jointe au canal d’une façon qui permet au [canal](~/v4sdk/bot-builder-channeldata.md) d’afficher cette pièce jointe dans la conversation. Si la pièce jointe est un fichier, l’URL du fichier s’affichera en tant que lien hypertexte dans la conversation.

## <a name="send-a-hero-card"></a>Envoyer une carte de héros

Outre les simples pièces jointes image ou vidéo, vous pouvez attacher une **carte de héros**, qui vous permet de combiner des images et des boutons dans un seul objet à envoyer à l’utilisateur.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Pour composer un message avec une carte de héros et un bouton, vous pouvez attacher une `HeroCard` à un message :

```csharp
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Schema;

// Create the activity and attach a Hero card.
var activity = MessageFactory.Attachment(
    new HeroCard(
        title: "White T-Shirt",
        images: new CardImage[] { new CardImage(url: "imageUrl.png") },
        buttons: new CardAction[]
        {
            new CardAction(title: "buy", type: ActionTypes.ImBack, value: "buy")
        })
    .ToAttachment());

// Send the activity as a reply to the user.
await context.SendActivity(activity);
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Pour envoyer une carte et un bouton à l’utilisateur, vous pouvez attacher une `heroCard` au message :

```javascript
// require MessageFactory and CardFactory from botbuilder.
const {MessageFactory} = require('botbuilder');
const {CardFactory} = require('botbuilder');

const message = MessageFactory.attachment(
     CardFactory.heroCard(
        'White T-Shirt',
        ['https://example.com/whiteShirt.jpg'],
        ['buy']
    )
 );

await context.sendActivity(message);
```

---

<!--Lifted from the RESP API documentation-->

Une carte enrichie comporte un titre, une description, un lien et des images. Un message peut contenir plusieurs cartes enrichies, affichées au format liste ou au format carrousel. Le Kit de développement logiciel Bot Builder prend actuellement en charge un large éventail de cartes enrichies. Pour obtenir une liste des cartes enrichies et des canaux dans lesquels elles sont prises en charge, consultez [Concevoir des éléments UX](../bot-service-design-user-experience.md).

> [!TIP]
> Pour déterminer le type de cartes enrichies prises en charge par un canal et voir comment le canal affiche les cartes enrichies, consultez l’article concernant l’[Inspecteur de canal](https://docs.microsoft.com/en-us/azure/bot-service/bot-service-channel-inspector?view=azure-bot-service-4.0). Consultez la documentation du canal pour plus d’informations sur les limitations des contenus des cartes (par exemple, le nombre maximal de boutons ou la longueur maximale du titre).

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

// Create the activity and attach a Hero card.
var activity = MessageFactory.Attachment(
    new HeroCard(
        title: "Holler Back Buttons",
        images: new CardImage[] { new CardImage(url: "imageUrl.png") },
        buttons: new CardAction[]
        {
            new CardAction(title: "Shout Out Loud", type: ActionTypes.imBack, value: "You can ALL hear me!"),
            new CardAction(title: "Much Quieter", type: ActionTypes.postBack, value: "Shh! My Bot friend hears me."),
            new CardAction(title: "Show me how to Holler", type: ActionTypes.openURL, value: $"https://en.wikipedia.org/wiki/{cardContent.Key}")
        })
    .ToAttachment());

// Send the activity as a reply to the user.
await context.SendActivity(activity);
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
const {ActionTypes} = require("botbuilder");
```

```javascript
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

# <a name="ctabcsharp"></a>[C#](#tab/csharp)
Pour utiliser des cartes adaptatives, veillez à ajouter le package NuGet `Microsoft.AdaptiveCards`.


> [!NOTE]
> Vous devez tester cette fonctionnalité avec les canaux que votre bot utilise pour déterminer s’ils prennent en charge les cartes adaptatives.

```csharp
using AdaptiveCards;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Schema;

// Create the activity and attach an Adaptive Card.
var card = new AdaptiveCard();
card.Body.Add(new TextBlock()
{
    Text = "<title>",
    Size = TextSize.Large,
    Wrap = true,
    Weight = TextWeight.Bolder
});
card.Body.Add(new TextBlock() { Text = "<message text>", Wrap = true });
card.Body.Add(new TextInput()
{
    Id = "Title",
    Value = string.Empty,
    Style = TextInputStyle.Text,
    Placeholder = "Title",
    IsRequired = true,
    MaxLength = 50
});
card.Actions.Add(new SubmitAction() { Title = "Submit", DataJson = "{ Action:'Submit' }" });
card.Actions.Add(new SubmitAction() { Title = "Cancel", DataJson = "{ Action:'Cancel'}" });

var activity = MessageFactory.Attachment(new Attachment(AdaptiveCard.ContentType, content: card));

// Send the activity as a reply to the user.
await context.SendActivity(activity);
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
const {CardFactory} = require("botbuilder");

const message = CardFactory.adaptiveCard({
    "$schema": "http://adaptivecards.io/schemas/adaptive-card.json",
    "version": "1.0",
    "type": "AdaptiveCard",
    "speak": "Your flight is confirmed for you from San Francisco to Amsterdam on Friday, October 10 8:30 AM",
    "body": [
        {
            "type": "TextBlock",
            "text": "Passenger",
            "weight": "bolder",
            "isSubtle": false
        },
        {
            "type": "TextBlock",
            "text": "Sarah Hum",
            "separator": true
        },
        {
            "type": "TextBlock",
            "text": "1 Stop",
            "weight": "bolder",
            "spacing": "medium"
        },
        {
            "type": "TextBlock",
            "text": "Fri, October 10 8:30 AM",
            "weight": "bolder",
            "spacing": "none"
        },
        {
            "type": "ColumnSet",
            "separator": true,
            "columns": [
                {
                    "type": "Column",
                    "width": 1,
                    "items": [
                        {
                            "type": "TextBlock",
                            "text": "San Francisco",
                            "isSubtle": true
                        },
                        {
                            "type": "TextBlock",
                            "size": "extraLarge",
                            "color": "accent",
                            "text": "SFO",
                            "spacing": "none"
                        }
                    ]
                },
                {
                    "type": "Column",
                    "width": "auto",
                    "items": [
                        {
                            "type": "TextBlock",
                            "text": " "
                        },
                        {
                            "type": "Image",
                            "url": "http://messagecardplayground.azurewebsites.net/assets/airplane.png",
                            "size": "small",
                            "spacing": "none"
                        }
                    ]
                },
                {
                    "type": "Column",
                    "width": 1,
                    "items": [
                        {
                            "type": "TextBlock",
                            "horizontalAlignment": "right",
                            "text": "Amsterdam",
                            "isSubtle": true
                        },
                        {
                            "type": "TextBlock",
                            "horizontalAlignment": "right",
                            "size": "extraLarge",
                            "color": "accent",
                            "text": "AMS",
                            "spacing": "none"
                        }
                    ]
                }
            ]
        },
        {
            "type": "ColumnSet",
            "spacing": "medium",
            "columns": [
                {
                    "type": "Column",
                    "width": "1",
                    "items": [
                        {
                            "type": "TextBlock",
                            "text": "Total",
                            "size": "medium",
                            "isSubtle": true
                        }
                    ]
                },
                {
                    "type": "Column",
                    "width": 1,
                    "items": [
                        {
                            "type": "TextBlock",
                            "horizontalAlignment": "right",
                            "text": "$1,032.54",
                            "size": "medium",
                            "weight": "bolder"
                        }
                    ]
                }
            ]
        }
    ]
});

// send adaptive card as attachment 
await context.sendActivity({ attachments: [message] })
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

[Aperçu des fonctionnalités avec l’inspecteur de canaux](../bot-service-channel-inspector.md)

---
