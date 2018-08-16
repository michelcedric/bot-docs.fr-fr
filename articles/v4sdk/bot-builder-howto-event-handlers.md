---
title: Gestionnaires d’événements | Microsoft Docs
description: Découvrez comment utiliser les gestionnaires d’événements.
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 06/14/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 0c054b8f4209004806e4564be45e83bdf3a8ec25
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39300085"
---
# <a name="event-handlers"></a>Gestionnaires d’événements

Les gestionnaires d’événements sont des fonctions que nous pouvons ajouter aux événements des activités futures au sein d’un [tour](bot-builder-basics.md#defining-a-turn). Ces activités sont `SendActivity`, `UpdateActivity` et `DeleteActivity`, chacune ayant son propre gestionnaire. Ces gestionnaires sont utiles quand vous avez besoin de faire quelque chose sur chaque activité future de ce type pour l’objet de contexte actuel.

Vous pouvez ajouter plusieurs gestionnaires d’événements à chaque événement d’activité ; leur traitement pour chaque événement ayant lieu **après** leur ajout, l’emplacement auquel vous les ajoutez dans le code a son importance.

> [!NOTE]
> Dans cet article, les *gestionnaires d’événements* ne répondent pas aux définitions traditionnelles spécifiques au langage des *événements* et des *gestionnaires*. Ils font plutôt référence à l’idée de programmation plus conceptuelle *d’événements* et de *gestionnaires*.

## <a name="using-the-next-delegate"></a>Utilisation du délégué *next*

En appelant le délégué *next*, chaque gestionnaire passe l’exécution au gestionnaire suivant dans l’ordre dans lequel les gestionnaires ont été ajoutés. Le dernière délégué *next* est un appel à l’adaptateur pour effectivement envoyer, mettre à jour ou supprimer l’activité.

Dans la mesure du possible, il est important d’appeler *next()* au sein de votre gestionnaire pour éviter de [court-circuiter](bot-builder-create-middleware.md#short-circuit-routing) l’activité. La différence entre le court-circuitage d’une activité et les middlewares (intergiciels) est que le court-circuitage annule complètement l’activité, tandis que les middlewares changent ce que le code est autorisé à exécuter, mais sans interrompre l’exécution de l’activité.

Pour les événements d’envoi d’activité, en cas de réussite, le délégué *next* retourne les ID assignés aux messages envoyés.

## <a name="expected-return-value"></a>Valeur de retour attendue

Le gestionnaire d’événements s’attend à ce que la valeur de retour soit le résultat du délégué next. Si nécessaire, ce résultat peut être affiché et stocké avant d’être retourné, ou être retourné directement, comme illustré ci-dessous.

La valeur de retour du gestionnaire `SendActivity` est la valeur de retour qui est transmise le long de la chaîne en tant que valeur de retour du délégué next correspondant.

# <a name="ctabcseventhandler"></a>[C#](#tab/cseventhandler)

Les trois types de gestionnaires d’événements fournis sont `OnSendActivities()`, `OnUpdateActivity()` et `OnDeleteActivity()`. Ici, nous ajoutons un gestionnaire `OnSendActivities()` au code du bot, mais les gestionnaires peuvent aussi être ajoutés à un middleware.

Les gestionnaires sont souvent ajoutés en tant qu’expressions lambda, comme vous le verrez dans cet exemple. Ici, nous écoutons l’activité d’entrée de l’utilisateur quand **help** (aide) est écrit.

```cs
public Task OnTurn(ITurnContext context)
{
    if (context.Activity.Type == ActivityTypes.Message)
    {
        // Get the conversation state from the turn context
        
        context.OnSendActivities(async (handlerContext, activities, handlerNext) =>
        {
            if (handlerContext.Activity.Text == "help")
            {
                Console.WriteLine("help!");
                // Do whatever logging you want to do for this help message
            }

            return await handlerNext();
        });
        // ...
    }
}
```

# <a name="javascripttabjseventhandler"></a>[JavaScript](#tab/jseventhandler)

Les trois types de gestionnaires d’événements fournis sont `onSendActivities()`, `onUpdateActivity()` et `onDeleteActivity()`. Ici, nous ajoutons un gestionnaire `onSendActivities()` au code du bot, mais les gestionnaires peuvent aussi être ajoutés à un middleware.

Cet exemple écoute l’activité d’entrée de l’utilisateur quand **help** (aide) est écrit.

```js
adapter.processActivity(req, res, async (context) => {

    if (context.activity.type === 'message') {

        context.onSendActivities(async (handlerContext, activities, handlerNext) => { 
            
            if(handlerContext.activity.text === 'help'){
                console.log('help!')
                // Do whatever logging you want to do for this help message
            }
            // Add handler logic here
        
            await handlerNext(); 
        });
        await context.sendActivity(`you said ${context.activity.text}`);
    }
});
```

---

Il est important de faire la distinction entre un événement *d’envoi d’activité* et un événement de *mise à jour ou suppression d’activité*, le premier créant entièrement un événement d’activité, tandis que le second agit sur une activité passée. De plus, les canaux ne prennent pas tous en charge la *mise à jour* ou la *suppression* des activités. Nous vous conseillons d’ajouter l’exception appropriée autour des appels à ces dernières et à leurs gestionnaires pour prendre en compte cette possibilité.

