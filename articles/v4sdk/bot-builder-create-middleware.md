---
title: Créer votre propre intergiciel | Microsoft Docs
description: Comprenez comment écrire votre propre intergiciel (middleware).
keywords: intergiciel (middleware), intergiciel (middleware) personnalisé, court-circuit, de secours, gestionnaires d’activités
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 03/21/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 78c0afd6095cf9a10e04e1b0131d66ce6964f369
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2018
ms.locfileid: "50000196"
---
# <a name="create-your-own-middleware"></a>Créer votre propre intergiciel

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Un intergiciel (middleware) vous permet d’écrire des plug-ins riches pour vos robots, que d’autres personnes peuvent également utiliser par la suite. Nous allons montrer ici comment ajouter et implémenter un intergiciel (middleware), et comment celui-ci fonctionne. Le Kit de développement logiciel (SDK) v4 fournit un intergiciel (middleware) pour effectuer des opérations telles que la gestion des états, LUIS, QnAMaker et la traduction. Pour plus d’informations, examinons le Kit de développement logiciel (SDK) Bot Builder pour [.NET](https://github.com/Microsoft/botbuilder-dotnet) ou [JavaScript](https://github.com/Microsoft/botbuilder-js).

## <a name="adding-middleware"></a>Ajout d’un intergiciel (middleware)

Dans l’exemple ci-après, basé sur notre exemple de bot de base créé dans le cadre de l’expérience de [démarrage rapide](~/bot-service-quickstart.md), deux intergiciels différents sont ajoutés à nos services avec une nouvelle instance de chacune de ces classes.

> [!IMPORTANT]
> Rappelez-vous que l’ordre dans lequel ils sont ajoutés aux options détermine l’ordre dans lequel ils sont exécutés. Si vous utilisez plus d’un intergiciel (middleware), réfléchissez bien à la manière dont cela fonctionnera.

**Startup.cs**

# <a name="ctabcsaddmiddleware"></a>[C#](#tab/csaddmiddleware)

Ajoutez des appels de la méthode `options.Middleware.Add(new MyMiddleware());` aux options de service de votre robot pour chaque intergiciel (middleware) que vous souhaitez ajouter.

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddSingleton(_ => Configuration);
    services.AddBot<HelloBot>(options =>
    {
        options.CredentialProvider = new ConfigurationCredentialProvider(Configuration);
        options.Middleware.Add(new MyMiddleware());
        options.Middleware.Add(new MyOtherMiddleware());
    });
}
```
# <a name="javascripttabjsaddmiddleware"></a>[JavaScript](#tab/jsaddmiddleware)

Ajoutez `adapter.use(MyMiddleware());` à votre adaptateur pour chaque intergiciel (middleware) que vous souhaitez ajouter.

```javascript
// Create adapter
const adapter = new botbuilder.BotFrameworkAdapter({
    appId: process.env.MICROSOFT_APP_ID, 
    appPassword: process.env.MICROSOFT_APP_PASSWORD 
});

adapter.use(MyMiddleware());
adapter.use(MyOtherMiddleware());
```

---


## <a name="implementing-your-middleware"></a>Implémentation de votre intergiciel (middleware)

Chaque intergiciel (middleware) hérite d’une interface d’intergiciel (middleware), et implémente toujours son gestionnaire de traitement exécuté sur chaque activité envoyée à votre robot. Pour chaque intergiciel (middleware) ajouté, le gestionnaire de traitement a la possibilité de modifier l’objet de contexte ou d’exécuter une tâche, telle qu’une journalisation, avant d’autoriser d’autres intergiciels (middleware) ou une logique de robot à interagir avec l’objet de contexte tandis que celui-ci progresse dans le pipeline.

# <a name="ctabcsetagoverwrite"></a>[C#](#tab/csetagoverwrite)

Chaque intergiciel (middleware) hérite de `IMiddleware` et implémente toujours `OnTurn()`.

**ExampleMiddleware.cs**
```csharp
public class MyMiddleware : IMiddleware
{
    public async Task OnTurn(ITurnContext context, MiddlewareSet.NextDelegate next)
    {            
        // This simple middleware reports the request type and if we responded
        await context.SendActivity($"Request type: {context.Activity.Type}");
        
        await next();            

        // Report if any responses were recorded
        string response = context.Responded ? "yes" : "no";
        await context.SendActivity($"Responded?  {response}");
    }
}

public class MyOtherMiddleware : IMiddleware
{
    public async Task OnTurn(ITurnContext context, MiddlewareSet.NextDelegate next)
    {
        // simple middleware to add an additional send activity
        await context.SendActivity($"My other middleware just saying Hi before the bot logic");

        await next();
    }
}

```

# <a name="javascripttabjsimplementmiddleware"></a>[JavaScript](#tab/jsimplementmiddleware)

Chaque intergiciel (middleware) hérite de `MiddlewareSet` et implémente toujours `onTurn()`.

**ExampleMiddleware.js**
```js
adapter.use({onTurn: async (context, next) =>{

    // This simple middleware reports the activity type and if we responded
    await context.sendActivity(`Activity type: ${context.activity.type}`); 
    await next();            

    // Report if any responses were recorded
    const response = context.activity.text ? "yes" : "no";
    await context.sendActivity(`Responded?  ${response}`);

}}, {onTurn: async (context, next) => {

     // simple middleware to add an additional send activity
     await context.sendActivity("My other middleware just saying Hi before the bot logic");

     await next();
}})
```

---

L’appel de `next()` a pour effet que l’exécution continue vers l’intergiciel (middleware) suivant. La possibilité de choisir quand l’exécution est transmise vous permet d’écrire du code qui s’exécute **après** exécution du reste de la pile de l’intergiciel (middleware). Nous pouvons agir sur la « périphérie » du gestionnaire de traitement, après exécution de la logique du robot et des autres intergiciels (middleware).  Dans l’exemple ci-dessus, le premier intergiciel (middleware) implémenté a fait exactement cela, en signalant si nous répondions pour cet objet de contexte, avant de re-transmettre l’exécution au pipeline.

## <a name="short-circuit-routing"></a>Routage de court-circuit

Dans certains cas, vous souhaiterez peut-être arrêter tout traitement ultérieur de l’activité reçue, ce que nous appelons court-circuiter. Ceci est utile quand l’intergiciel (middleware) prend complètement en charge une demande, fournit une réponse facile à des commandes spécifiques, ou peut gérer une demande entrante sans que la logique du robot ait besoin de la voir.

Créons un intergiciel (middleware) qui enverra une réponse et empêchera tout routage supplémentaire de la demande chaque fois que l’utilisateur dira « ping »:

# <a name="ctabcsmiddlewareshortcircuit"></a>[C#](#tab/csmiddlewareshortcircuit)
```cs
public class ExampleMiddleware : IMiddleware
{
    public async Task OnTurn(ITurnContext context, MiddlewareSet.NextDelegate next)
    {
        var utterance = context.Activity?.AsMessageActivity()?.Text.Trim().ToLower();

        if (utterance == "ping") 
        {
            context.SendActivity("pong");
            return;
        } 
        else 
        {
            await next();
        }
    }
}
```
# <a name="javascripttabjsmiddlewareshortcircuit"></a>[JavaScript](#tab/jsmiddlewareshortcircuit)
```JavaScript
adapter.use({onTurn: async (context, next) =>{
    const utterance = (context.activity.text || '').trim().toLowerCase();
        if (utterance == "ping") 
        {
            await context.sendActivity("pong");
            return;
        } 
        else 
        {
            await next();
        }

}})

```

---

## <a name="fallback-processing"></a>Traitement de secours

Une autre chose que vous pourriez avoir à faire est de répondre à une demande à laquelle vous n’avez pas encore répondu. Cela se fait facilement à l’aide de la périphérie du gestionnaire de traitement en vérifiant la propriété `context.Responded`. Créons un intergiciel (middleware) simple qui dit automatiquement « I didn’t understand » (je n’ai pas compris) si le robot ne parvient pas à gérer la demade :

# <a name="ctabcsfallback"></a>[C#](#tab/csfallback)
```cs
public async Task OnTurn(ITurnContext context, MiddlewareSet.NextDelegate next)
{
    await next();

    if (!context.Responded) 
    {
        context.SendActivity("I didn't understand.");
    }
}
```
# <a name="javascripttabjsfallback"></a>[JavaScript](#tab/jsfallback)
```JavaScript
adapter.use({onTurn: async (context, next) =>{
    await next();

    if (!context.responded) 
    {
       await context.sendActivity("I didn't understand.");
    }

}})
```

---

> [!NOTE] 
> Il se peut que cela ne fonctionne pas dans tous les cas, par exemple, quand d’autres intergiciels (middleware) peuvent répondre à l’utilisateur ou quand le robot reçoit un message correctement mais ne répond pas. Répondre en disant « je ne comprends pas » serait trompeur pour notre utilisateur.


