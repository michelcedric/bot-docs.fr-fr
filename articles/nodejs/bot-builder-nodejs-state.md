---
title: Gérer les données d’état | Microsoft Docs
description: Découvrez comment enregistrer et récupérer des données d’état avec le SDK Bot Framework pour Node.js.
author: DucVo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 6d653e47d2e906c6306134804c7731b374d830ba
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/11/2019
ms.locfileid: "54225004"
---
# <a name="manage-state-data"></a>Gérer les données d’état

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-state.md)
> - [Node.JS](../nodejs/bot-builder-nodejs-state.md)

[!INCLUDE [State concept overview](../includes/snippet-dotnet-concept-state.md)]

## <a name="in-memory-data-storage"></a>Stockage de données en mémoire

Le stockage de données en mémoire est uniquement destiné aux tests. Ce stockage est volatile et temporaire. Les données sont effacées chaque fois que le robot est redémarré. Pour utiliser le stockage en mémoire à des fins de test, vous devez procéder à deux opérations. Commencez par créer une nouvelle instance du stockage en mémoire :

```javascript
var inMemoryStorage = new builder.MemoryBotStorage();
```

Paramétrez-la ensuite pour le robot lorsque vous créez le **UniversalBot** :

```javascript
var inMemoryStorage = new builder.MemoryBotStorage();
var bot = new builder.UniversalBot(connector, [..waterfall steps..])
                    .set('storage', inMemoryStorage); // Register in-memory storage 
```

Vous pouvez utiliser cette méthode pour définir votre propre stockage de données personnalisé ou utiliser l’une des *extensions Azure*.

## <a name="manage-custom-data-storage"></a>Gérer le stockage de données personnalisé

Pour des raisons de performances et de sécurité dans l’environnement de production, vous pouvez implémenter votre propre stockage de données ou envisager de mettre en œuvre l’une des options de stockage de données suivantes :

1. [Gérer les données d’état avec Cosmos DB](bot-builder-nodejs-state-azure-cosmosdb.md)

2. [Gérer les données d’état avec le stockage de table](bot-builder-nodejs-state-azure-table-storage.md)

Avec l’une de ces options d’[extension Azure](https://www.npmjs.com/package/botbuilder-azure), le mécanisme de définition et de persistance des données par le biais du kit de développement logiciel Bot Framework pour Node.js reste identique à celui du stockage de données en mémoire.

## <a name="storage-containers"></a>Conteneurs de stockage

Dans le kit Bot Framework pour Node.js, l’objet `session` présente les propriétés de stockage de données d’état suivantes.

| Propriété | Étendue | Description |
| ---- | ---- | ---- |
| [`userData`][userDataURL] | Utilisateur | Contient les données sauvegardées pour l’utilisateur sur le canal spécifié. Ces données seront conservées dans plusieurs conversations. |
| [`privateConversationData`][privateConversationDataURL] | Conversation | Contient les données sauvegardées pour l’utilisateur dans le cadre d’une conversation donnée sur le canal spécifié. Ces données sont réservées à l’utilisateur actuel et ne seront conservées que pour la conversation en cours. La propriété est effacée en fin de conversation ou en cas d’appel explicite de `endConversation`. |
| [`conversationData`][conversationDataURL] | Conversation | Contient les données sauvegardées dans le cadre d’une conversation donnée sur le canal spécifié. Ces données sont partagées avec tous les utilisateurs participant à la conversation et ne seront conservées que pendant la conversation en cours. La propriété est effacée en fin de conversation ou en cas d’appel explicite de `endConversation`. |
| [`dialogData`][dialogDataURL] | Dialogue | Contient les données sauvegardées uniquement pour le dialogue en cours. Chaque dialogue conserve sa propre copie de cette propriété. La propriété est effacée lorsque le dialogue est effacé de la pile de dialogues. |

Ces quatre propriétés correspondent aux quatre conteneurs de stockage de données pouvant être utilisés pour stocker des données. Les propriétés utilisées pour stocker les données dépendent de l’étendue appropriée des données stockées, de leur nature et de la durée pendant laquelle elles doivent être conservées. Par exemple, si vous avez besoin de stocker des données utilisateur devant être accessibles dans plusieurs conversations, pensez à utiliser la propriété `userData`. Si vous avez besoin de stocker temporairement des valeurs de variables locales dans le cadre d’un dialogue, pensez à utiliser la propriété `dialogData`. Si vous avez besoin de stocker temporairement des données devant être accessibles dans plusieurs dialogues, pensez à utiliser la propriété `conversationData`.

## <a name="data-persistence"></a>Persistance des données

Par défaut, les données stockées à l’aide des propriétés `userData`, `privateConversationData` et `conversationData` sont définies de façon à être conservées après la fin de la conversation. Si vous ne souhaitez pas que les données soient conservées dans le conteneur `userData`, mettez l’indicateur `persistUserData` sur **Faux**. Si vous ne souhaitez pas que les données soient conservées dans le conteneur `conversationData`, mettez l’indicateur `persistConversationData` sur **Faux**. 

```javascript
// Do not persist userData
bot.set(`persistUserData`, false);

// Do not persist conversationData
bot.set(`persistConversationData`, false);
```

> [!NOTE]
> Il n’est pas possible de désactiver la conservation des données pour le conteneur `privateConversationData` ; elles sont toujours conservées.

## <a name="set-data"></a>Définir les données

Il est possible de stocker des objets JavaScript simples en les sauvegardant directement dans un conteneur de stockage. Pour un objet complexe comme `Date`, pensez à le convertir en `string`. En effet, les données d’état sont sérialisées et stockées au format JSON. Les exemples de code suivants montrent comment stocker des données primitives, un tableau, une carte d’objets et un objet complexe `Date`. 

**Stocker des données primitives**

```javascript
session.userData.userName = "Kumar Sarma";
session.userData.userAge = 37;
session.userData.hasChildren = true;
```

**Stocker un tableau**

```javascript
session.userData.profile = ["Kumar Sarma", "37", "true"];
```

**Stocker une carte d’objets**

```javascript
session.userData.about = {
    "Profile": {
        "Name": "Kumar Sarma",
        "Age": 37,
        "hasChildren": true
    },
    "Job": {
        "Company": "Contoso",
        "StartDate": "June 8th, 2010",
        "Title": "Developer"
    }
}
```
**Stocker la date et l’heure** 

Pour stocker un objet JavaScript complexe, convertissez-le en une chaîne de caractères avant de l’enregistrer dans un conteneur de stockage. 

```javascript 
var startDate = builder.EntityRecognizer.resolveTime([results.response]); 

// Date as string: "2017-08-23T05:00:00.000Z" 
session.userdata.start = startDate.toISOString(); 
``` 

### <a name="saving-data"></a>Enregistrement des données

Les données créées dans chaque conteneur de stockage restent en mémoire jusqu’à ce que le conteneur soit sauvegardé. Le kit SDK Bot Framework pour Node.js envoie par lots les données à enregistrer au service `ChatConnector` quand des messages doivent être envoyés. Pour enregistrer les données qui se trouvent dans les conteneurs de stockage sans envoyer de message, vous pouvez appeler manuellement la méthode [`save`](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session.html#save). Si vous n’appelez pas la méthode `save`, les données qui se trouvent dans les conteneurs de stockage sont conservées dans le cadre du traitement par lots.

```javascript
session.userData.favoriteColor = "Red";
session.userData.about.job.Title = "Senior Developer"; 
session.save();
```

## <a name="get-data"></a>Obtention des données

Pour accéder aux données enregistrées dans un conteneur de stockage donné, il suffit de faire référence à la propriété correspondante. Les exemples de code suivants montrent comment accéder à des données stockées antérieurement sous forme de données primitives, de tableau, de carte d’objets et d’objet Date complexe.

**Accéder à des données primitives**

```javascript
var userName = session.userData.userName;
var userAge = session.userData.userAge;
var hasChildren = session.userData.hasChildren;
```

**Accéder à un tableau**

```javascript
var userProfile = session.userData.userProfile;

session.send("User Profile:");
for(int i = 0; i < userProfile.length, i++){
    session.send(userProfile[i]);
}
```

**Accéder à une carte d’objets**

```javascript
var about = session.userData.about;

session.send("User %s works at %s.", about.Profile.Name, about.Job.Company);
```

**Accéder à un objet Date** 

Récupérez les données de date sous forme de chaîne de caractères, puis convertissez-les en objet Date de JavaScript. 

```javascript 
// startDate as a JavaScript Date object. 
var startDate = new Date(session.userdata.start); 
``` 

## <a name="delete-data"></a>Suppression de données

Par défaut, les données stockées dans le conteneur `dialogData` sont effacées lorsqu’un dialogue est supprimé de la pile de dialogues. De même, les données stockées dans les conteneurs `conversationData` et `privateConversationData` sont effacées lorsque la méthode `endConversation` est appelée. Cependant, pour supprimer les données stockées dans le conteneur `userData`, vous devez les effacer de façon explicite.

Pour ce faire, il suffit de réinitialiser le conteneur comme indiqué dans l’exemple de code suivant. 

```javascript
// Clears data stored in container.
session.userData = {}; 
session.privateConversationData = {};
session.conversationData = {};
session.dialogData = {};
```

Vous ne devez jamais définir un conteneur de données `null` ou le supprimer de l’objet `session`, car des erreurs se produiraient à la prochaine tentative d’accès au conteneur. Vous pouvez aussi appeler `session.save();` manuellement après avoir effacé manuellement un conteneur en mémoire afin de supprimer toutes les données correspondantes précédemment conservées.

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous savez comment gérer les données d’état utilisateur, voyons comment s’en servir pour mieux gérer le flux de la conversation.

> [!div class="nextstepaction"]
> [Gérer le flux de la conversation](bot-builder-nodejs-dialog-manage-conversation-flow.md)

## <a name="additional-resources"></a>Ressources supplémentaires
- [Inviter l’utilisateur à effectuer une entrée](bot-builder-nodejs-dialog-prompt.md)

[userDataURL]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session.html#userdata
[conversationDataURL]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session.html#conversationdata
[privateConversationDataURL]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session.html#privateconversationdata
[dialogDataURL]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session.html#dialogdata

[ChatConnector]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.chatconnector.html
