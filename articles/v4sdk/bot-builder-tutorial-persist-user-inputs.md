---
title: Conserver les données utilisateur | Microsoft Docs
description: Découvrez comment enregistrer les données d’état utilisateur sur un stockage dans le SDK Bot Builder.
keywords: conserver les données utilisateur, stockage, données de conversation
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 4/23/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 539e9e1cd772495d849ce106ee7d6a157fc1a9c0
ms.sourcegitcommit: 9a38d76afb0e82fdccc1f36f9b1a65042671e538
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/04/2018
ms.locfileid: "39515079"
---
# <a name="persist-user-data"></a>Conserver les données utilisateur

[!INCLUDE [pre-release-label](~/includes/pre-release-label.md)]

Quand le bot invite l’utilisateur à entrer des informations, il est possible que vous souhaitiez conserver une partie de ces informations sur un stockage sous une forme ou une autre. Le SDK Bot Builder vous permet de stocker les entrées d’utilisateur à l’aide d’un *stockage en mémoire*, d’un *stockage de fichier* ou d’un stockage de base de données tel que *CosmosDB* ou *SQL* ; les types de stockage local sont principalement utilisés à des fins de test ou de prototypage, tandis que les derniers types de stockage sont appropriés pour les bots de production.

Ce tutoriel vous explique comment définir votre objet de stockage et enregistrer les entrées utilisateur dans l’objet de stockage afin qu’elles soient rendues persistantes. 

> [!NOTE]
> Quel que soit le type de stockage que vous choisissez d’utiliser, le processus de traitement et de conservation des données est le même. Ce tutoriel utilise `FileStorage` comme stockage pour conserver les données.
> Pour plus d’informations sur l’état et d’autres types de stockage, consultez [Gérer l’état de la conversation et de l’utilisateur](bot-builder-howto-v4-state.md).

## <a name="prequisite"></a>Prérequis 

Ce tutoriel s’appuie sur le tutoriel [Réserver une table](bot-builder-tutorial-waterfall.md). Dans ce tutoriel, vous générez un bot qui invite l’utilisateur à indiquer trois éléments d’informations sur la réservation de sa table auprès de votre restaurant. Toutefois, l’entrée utilisateur n’a pas été rendue persistante. Ce tutoriel ajoute la persistance du stockage de données à ce bot.

## <a name="add-storage-to-middleware-layer"></a>Ajouter le stockage à la couche du middleware (intergiciel)

Le SDK Bot Builder V4 gère l’état et le stockage par le biais d’un middleware de gestion d’état. Le middleware fournit une couche d’abstraction qui vous permet d’accéder aux propriétés à l’aide d’un magasin clé-valeur simple, indépendamment du type de stockage sous-jacent. Le gestionnaire d’état s’occupe de l’écriture des données dans le stockage et de la gestion de la concurrence, quel que soit le type de stockage sous-jacent (en mémoire, fichier ou table Azure). Dans ce tutoriel, nous allons utiliser `FileStorage` pour conserver les entrées utilisateur.

Le fournisseur `FileStorage` est fourni avec le package `bot-builder`. L’exemple avec lequel vous avez démarré utilise le fournisseur `MemoryStorage`. Ce type de stockage utilise le support *en mémoire* volatil du bot, qui est détruit au redémarrage de ce dernier. La bibliothèque `FileStorage`, par contre, se comporte comme une base de données. Concrètement, elle écrit les informations de stockage dans un fichier sur votre ordinateur local. Vous pouvez spécifier où placer ce fichier de stockage afin de pouvoir l’inspecter plus tard.

Pour utiliser `FileStorage`, recherchez dans votre bot l’instruction où l’objet `conversationState` est défini et mettez-la à jour pour créer un `new botbuilder.FileStorage("c:/temp")`. En outre, vous pouvez définir l’emplacement auquel ce fichier de stockage doit être écrit. De cette façon, vous pouvez facilement le retrouver pour inspecter le contenu de ce qui a été rendu persistant.

# <a name="ctabcstab"></a>[C#](#tab/cstab)
```cs
var storage = new FileStorage("c:/temp");

// These two classes are simply Dictionaries to store state
options.Middleware.Add(new ConversationState<MyBot.convoState>(storage));
options.Middleware.Add(new UserState<MyBot.userState>(storage));
```

Le SDK Bot Builder fournit trois objets d’état avec des étendues différentes parmi lesquelles vous pouvez choisir.

| État | Étendue | Description |
| ---- | ---- | ---- |
| `dc.ActiveDialog.State` | Dialogue | État disponible pour les étapes du dialogue en cascade. |
| `ConversationState` | Conversation | État disponible pour la conversation actuelle. |
| `UserState` | user | État disponible sur plusieurs conversations. |

# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)

**app.js**
```javascript
// Storage
const storage = new FileStorage("c:/temp");
const convoState = new ConversationState(storage);
const userState  = new UserState(storage);
adapter.use(new BotStateSet(convoState, userState));
```

`BotStateSet` peut gérer `ConversationState` et `UserState` en même temps. Au moment d’enregistrer les données de l’utilisateur, vous pouvez choisir. Le SDK Bot Builder fournit trois objets d’état avec des étendues différentes parmi lesquelles vous pouvez choisir.

| État | Étendue | Description |
| ---- | ---- | ---- |
| `dc.activeDialog.state` | Dialogue | État disponible pour les étapes du dialogue en cascade. |
| `ConversationState` | Conversation | État disponible pour la conversation actuelle. |
| `UserState` | user | État disponible sur plusieurs conversations. |

---
 

## <a name="persist-state"></a>Conserver l’état

Pour votre bot, vous pouvez écrire dans n’importe lequel de ces trois emplacements d’état, selon ce que vous enregistrez et la durée pendant laquelle vous devez le conserver.  

# <a name="ctabcstab"></a>[C#](#tab/cstab)

Le *middleware de gestion d’état* gère automatiquement l’écriture de l’état à la fin de chaque tour. Ainsi, dans votre bot, vous devez simplement assigner les données à l’objet d’état de votre choix. Dans cet exemple, nous utilisons `dc.ActiveDialog.State` pour effectuer le suivi de l’entrée utilisateur pour notre réservation. Ainsi, au lieu d’enregistrer l’entrée utilisateur dans une variable globale, vous pouvez l’enregistrer dans une étendue d’objet d’état temporaire au sein du dialogue. Cet objet existe uniquement tant que le dialogue est actif ; si vous souhaitez le conserver plus de temps, vous devez le transférer à un des autres objets d’état. Dans ce cas, nous assignons le `msg` de réservation à l’état de conversation à la dernière étape de la cascade.

```cs
dialogs.Add("reserveTable", new WaterfallStep[]
{
    async (dc, args, next) =>
    {
        // Prompt for the guest's name.
        await dc.Context.SendActivity("Welcome to the reservation service.");

        dc.ActiveDialog.State = new Dictionary<string, object>();

        await dc.Prompt("dateTimePrompt", "Please provide a reservation date and time.");
    },
    async(dc, args, next) =>
    {
        var dateTimeResult = ((DateTimeResult)args).Resolution.First();

        dc.ActiveDialog.State["date"] = Convert.ToDateTime(dateTimeResult.Value);
        
        // Ask for next info
        await dc.Prompt("partySizePrompt", "How many people are in your party?");

    },
    async(dc, args, next) =>
    {
        dc.ActiveDialog.State["partySize"] = (int)args["Value"];

        // Ask for next info
        await dc.Prompt("textPrompt", "Who's name will this be under?");
    },
    async(dc, args, next) =>
    {
        dc.ActiveDialog.State["name"] = args["Text"];
        string msg = "Reservation confirmed. Reservation details - " +
        $"\nDate/Time: {dc.ActiveDialog.State["date"].ToString()} " +
        $"\nParty size: {dc.ActiveDialog.State["partySize"].ToString()} " +
        $"\nReservation name: {dc.ActiveDialog.State["name"]}";

        var convo = ConversationState<convoState>.Get(dc.Context);

        // In production, you may want to store something more helpful
        convo[$"{dc.ActiveDialog.State["name"]} reservation"] = msg;

        await dc.Context.SendActivity(msg);
        await dc.End();
    }
});
```

# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)

Le *middleware de gestion d’état* gère automatiquement l’écriture de l’état dans le fichier à la fin de chaque tour. Ainsi, dans votre bot, vous devez simplement assigner les données à l’objet d’état de votre choix. Dans cet exemple, nous utilisons `dc.activeDialog.state` pour effectuer le suivi de l’entrée utilisateur dans un objet `reservervationInfo`. Ainsi, au lieu d’enregistrer l’entrée utilisateur dans une variable globale, vous pouvez l’enregistrer dans une étendue d’objet d’état temporaire au sein du dialogue. Cet objet existant uniquement tant que le dialogue est actif, si vous souhaitez le conserver, vous devez le transférer à un des autres objets d’état. Dans ce cas, nous assignons `reservationInfo` à l’état `convo` à la dernière étape de la cascade.

```javascript
// Reserve a table:
// Help the user to reserve a table

dialogs.add('reserveTable', [
    async function(dc, args, next){
        await dc.context.sendActivity("Welcome to the reservation service.");

        dc.activeDialog.state.reservationInfo = {}; // Clears any previous data
        await dc.prompt('dateTimePrompt', "Please provide a reservation date and time.");
    },
    async function(dc, result){
        dc.activeDialog.state.reservationInfo.dateTime = result[0].value;

        // Ask for next info
        await dc.prompt('partySizePrompt', "How many people are in your party?");
    },
    async function(dc, result){
        dc.activeDialog.state.reservationInfo.partySize = result;

        // Ask for next info
        await dc.prompt('textPrompt', "Who's name will this be under?");
    },
    async function(dc, result){
        dc.activeDialog.state.reservationInfo.reserveName = result;
        
        // Persist data
        var convo = conversationState.get(dc.context);; // conversationState.get(dc.context);
        convo.reservationInfo = dc.activeDialog.state.reservationInfo;

        // Confirm reservation
        var msg = `Reservation confirmed. Reservation details: 
            <br/>Date/Time: ${dc.activeDialog.state.reservationInfo.dateTime} 
            <br/>Party size: ${dc.activeDialog.state.reservationInfo.partySize} 
            <br/>Reservation name: ${dc.activeDialog.state.reservationInfo.reserveName}`;
            
        await dc.context.sendActivity(msg);
        await dc.end();
    }
]);
```

---

Maintenant, vous êtes prêt à intégrer ce code à la logique du bot.

## <a name="start-the-dialog"></a>Démarrer le dialogue

Aucune modification de code n’est nécessaire ici. Exécutez simplement le bot et envoyez le message pour démarrer la conversation `reserveTable`.

## <a name="check-file-storage-content"></a>Vérifier le contenu du stockage de fichier

Après avoir exécuté le bot et suivi la conversation `reserveTable`, recherchez les informations enregistrées dans un fichier à l’emplacement que vous avez spécifié (par exemple : « C:/temp »). Le nom de fichier est précédé de « conversation! » ou « user! ». Il peut être utile de trier les fichiers par date afin de trouver le dernier plus facilement.

## <a name="next-steps"></a>Étapes suivantes

L’enregistrement des entrées utilisateur n’ayant plus de secret pour vous, jetons un œil au type d’entrée que vous pouvez demander à l’utilisateur à l’aide de la bibliothèque d’invites.

> [!div class="nextstepaction"]
> [Demander à l’utilisateur d’effectuer une saisie](~/v4sdk/bot-builder-prompts.md)
