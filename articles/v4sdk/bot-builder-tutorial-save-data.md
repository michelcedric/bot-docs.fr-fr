---
title: Didacticiel - Enregistrer les données d’état de l’utilisateur | Microsoft Docs
description: Découvrez comment enregistrer les données d’état de l’utilisateur dans le Kit de développement logiciel (SDK) Bot Builder.
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 4/23/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 86f70fd66f1bc2261339cbe0590061913b51ddbc
ms.sourcegitcommit: 2dc75701b169d822c9499e393439161bc87639d2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/24/2018
ms.locfileid: "42904437"
---
# <a name="save-user-state-data"></a>Enregistrer les données d’état de l’utilisateur

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Quand le robot invite l’utilisateur à entrer des informations, il est possible que vous souhaitiez conserver une partie de celles-ci sur un stockage sous une forme ou une autre. Le Kit de développement logiciel (SDK) Bot Builder vous permet de stocker des entrées utilisateur à l’aide d’un *stockage en mémoire*, d’un *stockage de fichiers*, ou d’une stockage de bases de données tel que *CosmosDB* ou *SQL*. 

Ce tutoriel vous explique comment définir votre objet de stockage dans l’intergiciel (middleware), et enregistrer les entrées utilisateur dans l’objet de stockage afin qu’elles puissent être conservées.

## <a name="prequisite"></a>Condition préalable 

Ce didacticiel s’appuie le didacticiel [Gérer un flux de conversation avec cascade](bot-builder-tutorial-waterfall.md).

## <a name="add-storage-to-middleware-layer"></a>Ajouter un stockage à la couche intergiciel (middleware)


## <a name="save-user-input-to-storage"></a>Enregistrer les entrées utilisateur dans un stockage

Pour gérer la conversation de réservation de table, vous devez définir un dialogue **en cascade** de quatre étapes. Dans cette conversation, vous utiliserez également un `DatetimePrompt` et un `NumberPrompt` en plus du `TextPrompt`.

Le dialogue `reserveTable` ressemblera à ceci :

```javascript
// Reserve a table:
// Help the user to reserve a table

var reservationInfo = {
    dateTime: '',
    partySize: '',
    reserveName: ''
}

dialogs.add('reserveTable', [
    async function(dc, args, next){
        await dc.context.sendActivity("Welcome to the reservation service.");
        reservationInfo = {}; // Clears any previous data
        await dc.prompt('dateTimePrompt', "Please provide a reservation date and time.");
    },
    async function(dc, result){
        reservationInfo.dateTime = result[0].value;

        // Ask for next info
        await dc.prompt('partySizePrompt', "How many people are in your party?");
    },
    async function(dc, result){
        reservationInfo.partySize = result;

        // Ask for next info
        await dc.prompt('textPrompt', "Who's name will this be under?");
    },
    async function(dc, result){
        reservationInfo.reserveName = result;

        // Confirm reservation
        var msg = `Reservation confirmed. Reservation details: 
            <br/>Date/Time: ${reservationInfo.dateTime} 
            <br/>Party size: ${reservationInfo.partySize} 
            <br/>Reservation name: ${reservationInfo.reserveName}`;
        await dc.context.sendActivity(msg);
        await dc.end();
    }
]);

```

Maintenant, vous êtes prêt à raccorder cela à la logique du robot.

## <a name="start-the-dialog"></a>Démarrer le dialogue

Modifiez la méthode `processActivity()` de votre robot pour qu’elle ressemble à ceci :

```javascript
// Listen for incoming activity 
server.post('/api/messages', (req, res) => {
    adapter.processActivity(req, res, async (context) => {
        if (context.activity.type === 'message') {
            // State will store all of your information 
            const state = conversationState.get(context);
            const dc = dialogs.createContext(context, state);

            if(context.activity.text.match(/hi/ig)){
                await dc.begin('greeting');
            }
            if(context.activity.text.match(/reserve table/ig)){
                await dc.begin('reserveTable');
            }
            else{
                // Continue executing the "current" dialog, if any.
                await dc.continue();
            }
        }
    });
});
```

Au moment de l’exécution, chaque fois que l’utilisateur envoie le message contenant la chaîne `reserve table`, le bot démarre la conversation `reserveTable`.

## <a name="next-steps"></a>Étapes suivantes

??? 

> [!div class="nextstepaction"]
> [Enregistrer les données d’état de l’utilisateur](bot-builder-tutorial-save-data.md)
