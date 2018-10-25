---
title: Extraire des résultats de LUIS typés | Microsoft Docs
description: Apprenez à utiliser LUIS pour extraire des entités avec le Kit de développement logiciel (SDK) Bot Builder.
keywords: intentions, entités, LUISGen, extraction
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 5/16/17
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: f0e428ca0aa1b0208538e2de7fc0a293763062a1
ms.sourcegitcommit: 54ed5000c67a5b59e23b667547565dd96c7302f9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/13/2018
ms.locfileid: "49315205"
---
# <a name="extract-intents-and-entities-using-luisgen"></a>Extraire des intentions et entités à l’aide de LUISGen

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

En plus de reconnaître une intention, une application LUIS peut extraire des entités qui sont des mots importants pour répondre à la demande d’un utilisateur. Par exemple, dans l’exemple d’une réservation de restaurant, l’application LUIS pourrait extraire le nombre de convives, la date de réservation ou l’emplacement du restaurant du message de l’utilisateur. 


L’[outil LUISGen](https://aka.ms/botbuilder-tools-luisgen) permet de générer des classes qui facilitent l’extraction d’entités de LUIS dans le code de votre robot.

À partir d’une ligne de commande Node.js, installez `luisgen` dans le chemin d’accès global.
```
npm install -g luisgen
```

# <a name="ctabcs"></a>[C#](#tab/cs)

## <a name="generate-a-luis-results-class"></a>Générer une classe de résultats de LUIS

Téléchargez l’[exemple LUIS CafeBot](https://aka.ms/contosocafebot-luis), puis, dans son dossier racine, exécutez LUISGen :

```
luisgen Assets\LU\models\LUIS\cafeLUISModel.json -cs ContosoCafeBot.CafeLUISModel
```

## <a name="examine-the-generated-code"></a>Examiner le code généré
Cette opération génère le fichier **cafeLUISModel.cs** que vous pouvez ajouter à votre projet. Il fournit une classe `cafeLuisModel` pour obtenir des résultats fortement typés à partir de LUIS.

Cette classe comprend une énumération (enum) pour l’obtention des intentions définies dans l’application LUIS.
```cs
public enum Intent {
    Book_Table, 
    Greeting, 
    None, 
    Who_are_you_intent
};
```
Elle possède également une propriété `Entities`. Dans la mesure où il peut y avoir plusieurs occurrences d’une entité dans le message d’un utilisateur, la classe `_Entities` définit un tableau pour chaque type d’entité. 
```cs
public class _Entities
{
    // Simple entities
    public string[] partySize;

    // Built-in entities
    public Microsoft.Bot.Builder.Ai.Luis.DateTimeSpec[] datetime;
    public double[] number;

    // Lists
    public string[][] cafeLocation;

    // Instance
    public class _Instance
    {
        public Microsoft.Bot.Builder.Ai.Luis.InstanceData[] partySize;
        public Microsoft.Bot.Builder.Ai.Luis.InstanceData[] datetime;
        public Microsoft.Bot.Builder.Ai.Luis.InstanceData[] number;
        public Microsoft.Bot.Builder.Ai.Luis.InstanceData[] cafeLocation;
    }
    [JsonProperty("$instance")]
    public _Instance _instance;
}
public _Entities Entities;
```

> [!NOTE]
> Tous les types d’entités sont des tableaux, car LUIS peut détecter plusieurs entités d’un type spécifié dans l’énoncé d’un utilisateur. Par exemple, si l’utilisateur dit « faire les réservations pour 17 heures demain et 21 heures samedi prochain », « 17 heures demain » et « 21 heures samedi prochain » sont retournés dans les résultats `datetime`.
>

|Entité | type | Exemples | Notes |
|-------|-----|------|---|
|partySize| string[]| `four` convives| Une entité simple reconnaît les chaînes. Dans cet exemple, la valeur d’Entities.partySize[0] est `"four"`.
|Datetime| [DateTimeSpec](https://docs.microsoft.com/en-us/dotnet/api/microsoft.bot.builder.ai.luis.datetimespec?view=botbuilder-4.0.0-alpha)[]| Réservation pour `9pm tomorrow`| Chaque objet **DateTimeSpec** comporte un champ Timex contenant la valeur d’heure spécifiée au format **timex**. Vous trouverez plus d’informations sur timex ici : http://www.timeml.org/publications/timeMLdocs/timeml_1.2.1.html#timex3      Vous trouverez plus d’informations sur la bibliothèque qui effectue la reconnaissance ici : https://github.com/Microsoft/Recognizers-Text
|number| double[]| `four` convives dont `2` enfants | `number` identifie tous les nombres, pas seulement le nombre de convives. <br/> Dans l’énoncé « Quatre convives dont deux enfants », `Entities.number[0]` est 4, et `Entities.number[1]` est 2.
|cafelocation| string[][] | Réservation à l’emplacement `Seattle`.| cafeLocation est une entité de liste, ce qui signifie qu’elle contient des membres reconnus de listes. Il s’agit d’un tableau de tableaux, au cas où une entité reconnue est membre de plusieurs listes. Par exemple, « réservation à Washington » peut correspondre à une liste pour l’état de Washington ou pour Washington D.C.

La propriété `_Instance` fournit [InstanceData](https://docs.microsoft.com/en-us/dotnet/api/microsoft.bot.builder.ai.luis.instancedata?view=botbuilder-4.0.0-alpha) pour plus de détails sur chaque entité reconnue.

## <a name="check-intents-in-your-bot"></a>Vérifier les intentions dans votre robot
Dans **CafeBot.cs**, examinez le code dans `OnTurn`. Vous pouvez voir où le robot appelle LUIS et vérifie les intentions pour décider du dialogue par lequel commencer. Les résultats de LUIS suite à l’appel de [`Recognize`](https://docs.microsoft.com/en-us/dotnet/api/microsoft.bot.builder.ai.luis.luisrecognizer?view=botbuilder-4.0.0-alpha#methods) sont passés en tant qu’argument au dialogue `BookTable`.



```cs
if(!context.Responded)
{
    // call LUIS and get results
    LuisRecognizer lRecognizer = createLUISRecognizer();
    // Use the generated class as the type parameter to Recognize()
    cafeLUISModel lResult = await lRecognizer.Recognize<cafeLUISModel>(utterance, ct);
    Dictionary<string,object> lD = new Dictionary<string,object>();
    if(lResult != null) {
        lD.Add("luisResult", lResult);
    }
    
    // top level dispatch
    switch (lResult.TopIntent().intent)
    {
        case cafeLUISModel.Intent.Greeting:
            await context.SendActivity("Hello!");
            if (userState.sendCards) await context.SendActivity(CreateCardResponse(context.Activity, createWelcomeCardAttachment()));
            break;

        case cafeLUISModel.Intent.Book_Table:
            await dc.Begin("BookTable", lD);
            break;

        case cafeLUISModel.Intent.Who_are_you_intent:
            await context.sendActivity("I'm the Contoso Cafe bot.");
            break;

        case cafeLUISModel.Intent.None:
        default:
            await getQnAResult(context);
            break;
    }
}
```

## <a name="extract-entities-in-a-dialog"></a>Extraire les entités dans un dialogue

Examinons à présent `Dialogs/BookTable.cs`. Le dialogue `BookTable` contient une séquence d’étapes en cascade, dont chacune contrôle la présente d’une entité dans les résultats de LUIS passés à `args`. Si l’entité n’est pas trouvée, le dialogue ignore l’invitation en appelant `next()`. Si l’entité est trouvée, le dialogue la demande, et la réponse de l’utilisateur à l’invite est reçue dans l’étape suivante de la cascade.

```cs
    Dialogs.Add("BookTable",
        new WaterfallStep[]
        {
            async (dc, args, next) =>
            {
                dc.ActiveDialog.State = new Dictionary<string, object>();
                IDictionary<string,object> state = dc.ActiveDialog.State;

                // add any LUIS entities to active dialog state.
                if(args.ContainsKey("luisResult")) {
                    cafeLUISModel lResult = (cafeLUISModel)args["luisResult"];
                    updateContextWithLUIS(lResult, ref state);
                }
                
                // prompt if we do not already have cafelocation
                if(state.ContainsKey("cafeLocation")) {
                    state["bookingLocation"] = state["cafeLocation"];
                    await next();
                } else {
                    await dc.Prompt("choicePrompt", "Which of our locations would you like?", promptOptions);
                }
            },
            async (dc, args, next) =>
            {
                var state = dc.ActiveDialog.State;
                if(!state.ContainsKey("cafeLocation")) {
                    var choiceResult = (FoundChoice)args["Value"];
                    state["bookingLocation"] = choiceResult.Value;
                }
                bool promptForDateTime = true;
                if(state.ContainsKey("datetime")) {
                    // validate timex
                    var inputdatetime = new string[] {(string)state["datetime"]};
                    var results = evaluateTimeX((string[])inputdatetime);
                    if(results.Count != 0) {
                        var timexResolution = results.First().TimexValue;
                        var timexProperty = new TimexProperty(timexResolution.ToString());
                        var bookingDateTime = $"{timexProperty.ToNaturalLanguage(DateTime.Now)}";
                        state["bookingDateTime"] = bookingDateTime;
                        promptForDateTime = false;
                    }
                }
                // prompt if we do not already have date and time
                if(promptForDateTime) {
                    await dc.Prompt("timexPrompt", "When would you like to arrive? (We open at 4PM.)",
                                    new PromptOptions { RetryPromptString = "We only accept reservations for the next 2 weeks and in the evenings between 4PM - 8PM" });
                } else {
                    await next();
                }                       
                
            },
            async (dc, args, next) =>
            {
                var state = dc.ActiveDialog.State;
                if(!state.ContainsKey("datetime")) { 
                    var timexResult = (TimexResult)args;
                    var timexResolution = timexResult.Resolutions.First();
                    var timexProperty = new TimexProperty(timexResolution.ToString());
                    var bookingDateTime = $"{timexProperty.ToNaturalLanguage(DateTime.Now)}";
                    state["bookingDateTime"] = bookingDateTime;
                }
                // prompt if we already do not have party size
                if(state.ContainsKey("partySize")) {
                    state["bookingGuestCount"] = state["partySize"];
                    await next();
                } else {
                    await dc.Prompt("numberPrompt", "How many in your party?");
                }
            },
            async (dc, args, next) =>
            {
                var state = dc.ActiveDialog.State;
                if(!state.ContainsKey("partySize")) {
                    state["bookingGuestCount"] = args["Value"];
                }

                await dc.Prompt("confirmationPrompt", $"Thanks, Should I go ahead and book a table for {state["bookingGuestCount"].ToString()} guests at our {state["bookingLocation"].ToString()} location for {state["bookingDateTime"].ToString()} ?");
            },
            async (dc, args, next) =>
            {
                var dialogState = dc.ActiveDialog.State;

                // TODO: Verify user said yes to confirmation prompt

                // TODO: book the table! 

                await dc.Context.SendActivity($"Thanks, I have {dialogState["bookingGuestCount"].ToString()} guests booked for our {dialogState["bookingLocation"].ToString()} location for {dialogState["bookingDateTime"].ToString()}.");
            }
        }
    );
}

// This helper method updates dialog state with any LUIS results
private void updateContextWithLUIS(cafeLUISModel lResult, ref IDictionary<string,object> dialogContext) {
    if(lResult.Entities.cafeLocation != null && lResult.Entities.cafeLocation.GetLength(0) > 0) {
        dialogContext.Add("cafeLocation", lResult.Entities.cafeLocation[0][0]);
    }
    if(lResult.Entities.partySize != null && lResult.Entities.partySize.GetLength(0) > 0) {
        dialogContext.Add("partySize", lResult.Entities.partySize[0][0]);
    } else {
        if(lResult.Entities.number != null && lResult.Entities.number.GetLength(0) > 0) {
            dialogContext.Add("partySize", lResult.Entities.number[0]);
        }
    }
    if(lResult.Entities.datetime != null && lResult.Entities.datetime.GetLength(0) > 0) {
        dialogContext.Add("datetime", lResult.Entities.datetime[0].Expressions[0]);
    }
}
```
## <a name="run-the-sample"></a>Exécution de l'exemple

Ouvrez `ContosoCafeBot.sln` dans Visual Studio 2017, puis exécutez le robot. Utilisez l’[émulateur Bot Framework](https://docs.microsoft.com/en-us/azure/bot-service/bot-service-debug-emulator) pour vous connecter à l’exemple de robot.

Dans l’émulateur, dites `reserve a table` pour démarrer le dialogue de réservation.

![exécuter le bot](media/how-to-luisgen/run-bot.png)

# <a name="typescripttabjs"></a>[TypeScript](#tab/js)

Téléchargez l’[exemple CafeBot_LUIS](https://aka.ms/contosocafebot-typescript-luis-dialogs), puis, dans son dossier racine, exécutez LUISGen :

```
luisgen cafeLUISModel.json -ts CafeLUISModel
```

Cette opération génère **CafeLUISModel.ts**, que vous pouvez ajouter à votre projet. Vous pouvez obtenir un résultat typé à partir du module de reconnaissance de LUIS en utilisant les types dans le fichier généré.


```typescript
// call LUIS and get typed results
await luisRec.recognize(context).then(async (res : any) => 
{    
    // get a typed result
    var typedresult = res as CafeLUISModel;   
    
```

## <a name="pass-the-typed-result-to-a-dialog"></a>Passer le résultat typé à un dialogue

Examinez le code dans **luisbot.ts**. Dans le gestionnaire `processActivity`, le robot passe le résultat typé à un dialogue.

```typescript
// Listen for incoming requests 
server.post('/api/messages', (req, res) => {
    // Route received request to adapter for processing
    adapter.processActivity(req, res, async (context) => {
        const isMessage = context.activity.type === 'message';

        // Create dialog context 
        const state = conversationState.get(context);
        const dc = dialogs.createContext(context, state);
            
        if (!isMessage) {
            await context.sendActivity(`[${context.activity.type} event detected]`);
        }

        // Check to see if anyone replied. 
        if (!context.responded) {
            await dc.continue();
            // if the dialog didn't send a response
            if (!context.responded && isMessage) {

                
                await luisRec.recognize(context).then(async (res : any) => 
                {    
                    var typedresult = res as CafeLUISModel;                
                    let topIntent = LuisRecognizer.topIntent(res);    
                    switch (topIntent)
                    {
                        case Intents.Book_Table: {
                            await context.sendActivity("Top intent is Book_Table ");                          
                            await dc.begin('reserveTable', typedresult);
                            break;
                        }
                        
                        case Intents.Greeting: {
                            await context.sendActivity("Top intent is Greeting");
                            break;
                        }
    
                        case Intents.Who_are_you_intent: {
                            await context.sendActivity("Top intent is Who_are_you_intent");
                            break;
                        }
                        default: {
                            await dc.begin('default', topIntent);
                            break;
                        }
                    }
    
                }, (err) => {
                    // there was some error
                    console.log(err);
                }
                );                                
            }
        }
    });
});
```

## <a name="check-for-existing-entities-in-a-dialog"></a>Vérifier des entités existantes dans un dialogue

Dans **luisbot.ts**, le dialogue `reserveTable` appelle une fonction d’assistance `SaveEntities` pour vérifier la présence d’entités détectées par l’application LUIS. Si les entités sont trouvées, elles sont enregistrées à l’état du dialogue. Chaque étape de la cascade dans la boîte de dialogue vérifie si une entité a été enregistrée dans l’état du dialogue et, à défaut, vous invite à l’entrer.

```typescript
dialogs.add('reserveTable', [
    async function(dc, args, next){
        var typedresult = args as CafeLUISModel;

        // Call a helper function to save the entities in the LUIS result
        // to dialog state
        await SaveEntities(dc, typedresult);

        await dc.context.sendActivity("Welcome to the reservation service.");
        
        if (dc.activeDialog.state.dateTime) {
            await next();     
        }
        else {
            await dc.prompt('dateTimePrompt', "Please provide a reservation date and time.");
        }
    },
    async function(dc, result, next){
        if (!dc.activeDialog.state.dateTime) {
            // Save the dateTimePrompt result to dialog state
            dc.activeDialog.state.dateTime = result[0].value;
        }

        // If we don't have party size, ask for it next
        if (!dc.activeDialog.state.partySize) {
            await dc.prompt('textPrompt', "How many people are in your party?");
        } else {
            await next();
        }
    },
    async function(dc, result, next){
        if (!dc.activeDialog.state.partySize) {
            dc.activeDialog.state.partySize = result;
        }
        // Ask for the reservation name next
        await dc.prompt('textPrompt', "Whose name will this be under?");
    },
    async function(dc, result){
        dc.activeDialog.state.Name = result;

        // Save data to conversation state
        var state = conversationState.get(dc.context);

        // Copy the dialog state to the conversation state
        state = dc.activeDialog.state;

        // TODO: Add in <br/>Location: ${state.cafeLocation}
        var msg = `Reservation confirmed. Reservation details:             
            <br/>Date/Time: ${state.dateTime} 
            <br/>Party size: ${state.partySize} 
            <br/>Reservation name: ${state.Name}`;
            
        await dc.context.sendActivity(msg);
        await dc.end();
    }
]);
```

La fonction d’assistance `SaveEntities` vérifie la présence de `datetime` et d’entités `partysize`. L’entité `datetime` est une [entité prédéfinie](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/luis-reference-prebuilt-entities#builtindatetimev2).

```typescript
// Helper function that saves any entities found in the LUIS result
// to the dialog state
async function SaveEntities( dc: DialogContext<TurnContext>, typedresult) {
    // Resolve entities returned from LUIS, and save these to state
    if (typedresult.entities)
    {
        console.log(`Entities found.`);
        let datetime = typedresult.entities.datetime;

        if (datetime) {
            // Use the first date or time found in the utterance
            if (datetime[0].timex) {
                timexValues = datetime[0].timex;
                // Datetime results from LUIS are represented in timex format:
                // http://www.timeml.org/publications/timeMLdocs/timeml_1.2.1.html#timex3                                
                // More information on the library which does the recognition can be found here: 
                // https://github.com/Microsoft/Recognizers-Text

                if (datetime[0].type === "datetime") {
                    // To parse timex, here you use the resolve and creator from
                    // @microsoft/recognizers-text-data-types-timex-expression
                    // The second parameter is an array of constraints the results must satisfy
                    var resolution = Resolver.evaluate(
                        // array of timex values to evaluate. There may be more than one: "today at 6" can be 6AM or 6PM.
                        timexValues,
                        // constrain results to times between 4pm and 8pm                        
                        [Creator.evening]);
                    if (resolution[0]) {
                        // toNaturalLanguage takes the current date into account to create a friendly string
                        dc.activeDialog.state.dateTime = resolution[0].toNaturalLanguage(new Date());
                        // You can also use resolution.toString() to format the date/time.
                    } else {
                        // time didn't satisfy constraint.
                        dc.activeDialog.state.dateTime = null;
                    }
                } 
                else  {
                    console.log(`Type ${datetime[0].type} is not yet supported. Provide both the date and the time.`);
                }
            }                                                
        }
        let partysize = typedresult.entities.partySize;
        if (partysize) {
            console.log(`partysize entity detected.${partysize}`);
            // use first partySize entity that was found in utterance
            dc.activeDialog.state.partySize = partysize[0];
        }
        let cafelocation = typedresult.entities.cafeLocation;

        if (cafelocation) {
            console.log(`location entity detected.${cafelocation}`);
            // use first cafeLocation entity that was found in utterance
            dc.activeDialog.state.cafeLocation = cafelocation[0][0];
        }
    } 
}
```

## <a name="run-the-sample"></a>Exécution de l'exemple

1. Si vous n’avez pas le compilateur TypeScript installé, installez-le à l’aide de la commande suivante :

```
npm install --global typescript
```

2. Installez les dépendances avant d’exécuter le robot, en exécutant `npm install` dans le répertoire racine de l’exemple :

```
npm install
```

3. À partir du répertoire racine, générez l’exemple en utilisant `tsc`. Cela a pour effet de générer `luisbot.js`.

4. Exécutez `luisbot.js` dans le répertoire `lib`.

5. Utilisez l’[émulateur Bot Framework](https://docs.microsoft.com/en-us/azure/bot-service/bot-service-debug-emulator) pour exécuter l’exemple.

6. Dans l’émulateur, dites `reserve a table` pour démarrer le dialogue de réservation.

![Exécuter le bot](media/how-to-luisgen/run-bot.png)

---


## <a name="additional-resources"></a>Ressources supplémentaires

Pour plus de contexte sur LUIS, voir [Language Understanding](./bot-builder-concept-luis.md).


## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Combiner LUIS et QnA à l’aide de l’outil de répartition](./bot-builder-tutorial-dispatch.md)


