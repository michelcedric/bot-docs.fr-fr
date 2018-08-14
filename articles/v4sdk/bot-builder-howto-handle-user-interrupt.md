---
title: Gérer les interruptions par l’utilisateur | Microsoft Docs
description: Découvrez comment gérer les interruptions par l’utilisateur et diriger le flux de conversation.
keywords: interruption, interruptions, changement de sujet, arrêt
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 04/17/2018
ms.reviewer: ''
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 651a7410893f7a66f5941121edc7b34055807ba7
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39298921"
---
# <a name="handle-user-interrupt"></a>Gérer les interruptions par l’utilisateur

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

La gestion des interruptions par l’utilisateur représente un aspect essentiel d’un bot efficace. Vous pensez peut-être que vos utilisateurs suivront pas à pas le flux de conversation que vous avez défini. Pourtant, il est probable qu’ils changent d’avis ou posent une question au milieu du processus au lieu de répondre à la question posée. Dans ce type de situation, comment votre bot gérera-t-il l’entrée utilisateur ? Quelle sera l’expérience utilisateur ? Comment gérerez-vous les données d’état de l’utilisateur ?

Il n’existe aucune réponse correcte à ces questions dans la mesure où chaque situation est spécifique au scénario que votre bot est supposé gérer. Dans cette rubrique, nous examinerons certaines méthodes courantes pour gérer les interruptions par l’utilisateur et suggérerons différentes façons de les implémenter dans votre bot.

## <a name="handle-expected-interruptions"></a>Gérer les interruptions attendues

Un flux de conversation procédural repose sur un ensemble d’étapes clés à travers lesquelles vous souhaitez guider l’utilisateur. Toute action de l’utilisateur ne correspondant pas à ces étapes représente une interruption potentielle. Dans le cadre d’un flux normal, vous pouvez anticiper certaines interruptions.

**Réservation de table** Dans un bot de réservation de table, les principales étapes peuvent consister à demander à l’utilisateur la date et l’heure de la réservation, le nombre d’invités et le nom auquel effectuer la réservation. Dans ce processus, vous pouvez anticiper certaines interruptions, notamment : 
 * `cancel` : pour quitter le processus.
 * `help` : pour fournir une assistance supplémentaire concernant ce processus.
 * `more info` : pour donner des conseils et des suggestions ou pour proposer d’autres moyens de réserver une table (par exemple, à l’aide d’une adresse e-mail ou d’un numéro de téléphone).
 * `show list of available tables` : le cas échéant, pour afficher une liste des tables disponibles à date et à l’heure souhaitées par l’utilisateur.

**Commande d’un dîner** Dans un bot de commande de dîner, les principales étapes consisteraient à fournir une liste de plats et à permettre à l’utilisateur d’en ajouter à son panier. Dans ce processus, vous pouvez anticiper certaines interruptions, notamment : 
 * `cancel` : pour quitter le processus de commande.
 * `more info` : pour fournir des informations diététiques sur chaque plat.
 * `help` : pour fournir une aide sur l’utilisation du système.
 * `process order` : pour traiter la commande.

Vous pouvez mettre ces options à la disposition de l’utilisateur sous la forme d’une liste **d’actions suggérées** ou de conseils, de sorte que l’utilisateur connaisse au moins les commandes qu’il peut envoyer et que le bot comprendra.

Par exemple, dans le flux de commande de dîner, vous pouvez inclure les interruptions attendues avec les plats. Dans ce cas, les plats sont envoyés sous forme de tableau de `choices`.

# <a name="ctabcsharptab"></a>[C#](#tab/csharptab)

```cs
public class dinnerItem
{
    public string Description;
    public double Price;
}

public class dinnerMenu
{
    static public Dictionary<string, dinnerItem> dinnerChoices = new Dictionary<string, dinnerItem>
    {
        { "potato salad", new dinnerItem { Description="Potato Salad", Price=5.99 } },
        { "tuna sandwich", new dinnerItem { Description="Tuna Sandwich", Price=6.89 } },
        { "clam chowder", new dinnerItem { Description="Clam Chowder", Price=4.50 } }
    };

    static public string[] choices = new string[] {"Potato Salad", "Tuna Sandwich", "Clam Chowder", "more info", "Process order", "help", "Cancel"};
}
```

# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)

```javascript
var dinnerMenu = {
    choices: ["Potato Salad - $5.99", "Tuna Sandwich - $6.89", "Clam Chowder - $4.50", 
            "more info", "Process order", "Cancel"],
    "Potato Salad - $5.99": {
        Description: "Potato Salad",
        Price: 5.99
    },
    "Tuna Sandwich - $6.89": {
        Description: "Tuna Sandwich",
        Price: 6.89
    },
    "Clam Chowder - $4.50": {
        Description: "Clam Chowder",
        Price: 4.50
    }
}
```

---

Dans votre logique de commande, vous pouvez les vérifier à l’aide de correspondances de chaînes ou d’expressions régulières.

# <a name="ctabcsharptab"></a>[C#](#tab/csharptab)

Tout d’abord, nous devons définir une assistance pour assurer le suivi de nos commandes.

```cs
// Helper class for storing the order in the dictionary
public class Orders
{
    public double total;
    public string order;
    public bool processOrder;

    // Initialize order values
    public Orders()
    {
        total = 0;
        order = "";
        processOrder = false;
    }
}
```

Vous devez ensuite ajouter la boîte de dialogue à votre bot.

```cs
dialogs.Add("orderPrompt", new WaterfallStep[]
{
    async (dc, args, next) =>
    {
        // Prompt the user
        await dc.Prompt("choicePrompt",
            "What would you like for dinner?",
            new ChoicePromptOptions
            {
                Choices = dinnerMenu.choices.Select( s => new Choice { Value = s }).ToList(),
                RetryPromptString = "I'm sorry, I didn't understand that. What would you " +
                    "like for dinner?"
            });
    },
    async(dc, args, next) =>
    {
        var convo = ConversationState<Dictionary<string,object>>.Get(dc.Context);

        // Get the user's choice from the previous prompt
        var response = (args["Value"] as FoundChoice).Value.ToLower();

        if(response == "process order")
        {
            try 
            {
                var order = convo["order"];

                await dc.Context.SendActivity("Order is on it's way!");
                
                // In production, you may want to store something more helpful, 
                // such as send order off to be made
                (order as Orders).processOrder = true;

                // Once it's submitted, clear the current order
                convo.Remove("order");
                await dc.End();
            }
            catch
            {
                await dc.Context.SendActivity("Your order is empty, please add your order choice");
                // Ask again
                await dc.Replace("orderPrompt");
            }
        }
        else if(response == "cancel" )
        {
            // Get rid of current order
            convo.Remove("order");
            await dc.Context.SendActivity("Your order has been canceled");
            await dc.End();
        }
        else if(response == "more info")
        {
            // Send more information about the options
            var msg = "More info: <br/>" +
                "Potato Salad: contains 330 calaries per serving. Cost: 5.99 <br/>"
                + "Tuna Sandwich: contains 700 calaries per serving. Cost: 6.89 <br/>"
                + "Clam Chowder: contains 650 calaries per serving. Cost: 4.50";
            await dc.Context.SendActivity(msg);

            // Ask again
            await dc.Replace("orderPrompt");
        }
        else if(response == "help")
        {
            // Provide help information
            await dc.Context.SendActivity("To make an order, add as many items to your cart as " +
                "you like then choose the \"Process order\" option to check out.");

            // Ask again
            await dc.Replace("orderPrompt");
        }
        else
        {
            // Unlikely to get past the prompt verification, but this will catch 
            // anything that isn't a valid menu choice
            if(!dinnerMenu.dinnerChoices.ContainsKey(response))
            {
                await dc.Context.SendActivity("Sorry, that is not a valid item. " +
                    "Please pick one from the menu.");
    
                // Ask again
                await dc.Replace("orderPrompt");
            }
            else {
                // Add the item to cart
                Orders currentOrder;

                // If there is a current order going, add to it. If not, start a new one
                try
                {
                    currentOrder = convo["order"] as Orders;
                }
                catch
                {
                    convo["order"] = new Orders();
                    currentOrder = convo["order"] as Orders;
                }

                // Add to the current order
                currentOrder.order += (dinnerMenu.dinnerChoices[$"{response}"].Description) + ", ";
                currentOrder.total += (double)dinnerMenu.dinnerChoices[$"{response}"].Price;

                // Save back to the conversation state
                convo["order"] = currentOrder;

                await dc.Context.SendActivity($"Added to cart. Current order: " +
                    $"{currentOrder.order} " +
                    $"<br/>Current total: ${currentOrder.total}");

                // Ask again to allow user to add more items or process order
                await dc.Replace("orderPrompt");
            }
        }
    }
});
```

# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)

```javascript
// Helper dialog to repeatedly prompt user for orders
dialogs.add('orderPrompt', [
    async function(dc){
        await dc.prompt('choicePrompt', "What would you like?", dinnerMenu.choices);
    },
    async function(dc, choice){
        if(choice.value.match(/process order/ig)){
            if(orderCart.orders.length > 0) {
                // Process the order
                dc.context.activity.conversation.dinnerOrder = orderCart;

                await dc.end();
            }
            else {
                await dc.context.sendActivity("Your cart was empty. Please add at least one item to the cart.");
                // Ask again
                await dc.replace('orderPrompt');
            }
        }
        else if(choice.value.match(/cancel/ig)){
            orderCart.clear(context);
            await dc.context.sendActivity("Your order has been canceled.");
            await dc.end(choice.value);
        }
        else if(choice.value.match(/more info/ig)){
            var msg = "More info: <br/>Potato Salad: contains 330 calaries per serving. <br/>"
                + "Tuna Sandwich: contains 700 calaries per serving. <br/>" 
                + "Clam Chowder: contains 650 calaries per serving."
            await dc.context.sendActivity(msg);
            
            // Ask again
            await dc.replace('orderPrompt');
        }
        else if(choice.value.match(/help/ig)){
            var msg = `Help: <br/>To make an order, add as many items to your cart as you like then choose the "Process order" option to check out.`
            await dc.context.sendActivity(msg);
            
            // Ask again
            await dc.replace('orderPrompt');
        }
        else {
            var choice = dinnerMenu[choice.value];

            // Only proceed if user chooses an item from the menu
            if(!choice){
                await dc.context.sendActivity("Sorry, that is not a valid item. Please pick one from the menu.");
                
                // Ask again
                await dc.replace('orderPrompt');
            }
            else {
                // Add the item to cart
                orderCart.orders.push(choice);
                orderCart.total += dinnerMenu[choice.value].Price;

                await dc.context.sendActivity(`Added to cart: ${choice.value}. <br/>Current total: $${orderCart.total}`);

                // Ask again
                await dc.replace('orderPrompt');
            }
        }
    }
]);
```

---

## <a name="handle-unexpected-interruptions"></a>Gérer les interruptions inattendues

Certaines interruptions ne correspondent pas au scénario pour lequel votre bot est conçu.
Même si vous ne pouvez pas anticiper toutes les interruptions, vous pouvez programmer votre bot pour gérer certains modèles d’interruption.

### <a name="switching-topic-of-conversations"></a>Changement de sujet de conversation
Que se passe-t-il si, au milieu d’une conversation, l’utilisateur souhaite passer à une autre conversation ? Par exemple, votre bot peut réserver une table et commander un dîner.
L’utilisateur suit le flux de _réservation de table_. Cependant, au lieu de répondre à la question « Combien de personnes participent au repas ? », il envoie le message « Commander un dîner ». Dans ce cas, l’utilisateur change d’avis et veut commencer une conversation sur la commande d’un dîner. Comment allez-vous gérer cette interruption ? 

Vous pouvez changer de sujet et basculer vers le flux de commande de dîner, ou vous pouvez obliger l’utilisateur à résoudre le problème en lui indiquant que vous attendez un nombre et en reposant la question. Si vous autorisez l’utilisateur à changer de sujet, vous devez décider si vous allez enregistrer la progression de la conversation afin qu’il puisse la reprendre au même stade plus tard ou si vous allez supprimer toutes les informations que vous avez collectées, de sorte qu’il reprenne l’ensemble du processus la prochaine fois qu’il souhaite réserver une table. Pour plus d’informations sur la gestion des données d’état de l’utilisateur, consultez la rubrique [Save state using conversation and user properties](bot-builder-howto-v4-state.md) (Enregistrer l’état à l’aide des propriétés de conversation et d’utilisateur).

### <a name="apply-artificial-intelligence"></a>Appliquer l’intelligence artificielle
Pour les interruptions ne correspondant pas au scénario, vous pouvez essayer de deviner l’intention de l’utilisateur. Pour cela, vous pouvez utiliser des services d’intelligence artificielle comme QnAMaker, LUIS ou votre logique personnalisée, puis proposer des suggestions quant aux intentions de l’utilisateur envisagées par le bot. 

Par exemple, au milieu du flux de réservation de table, l’utilisateur dit « Je veux commander un burger ». Le bot ne sait pas comment gérer cette situation dans le cadre de ce flux de conversation. Étant donné que le flux actuel n’a rien à voir avec une procédure de commande et que l’autre commande de conversation du bot est « Commander un dîner », le bot ne sait pas comment gérer cette entrée. Si vous appliquez le service LUIS, par exemple, vous pouvez former le modèle de sorte qu’il reconnaisse que l’utilisateur souhaite commander un repas (par exemple, LUIS peut retourner une intention « orderFood »). Par conséquent, le bot peut répondre « Vous souhaitez certainement commander un repas. Souhaitez-vous basculer vers notre processus de commande de dîner ? » Pour plus d’informations sur la formation de LUIS et la détection des intentions de l’utilisateur, consultez [Using LUIS for language understanding](bot-builder-howto-v4-luis.md) (Utiliser LUIS pour la reconnaissance vocale).

### <a name="default-response"></a>Réponse par défaut
Si tout le reste échoue, vous pouvez envoyer une réponse générique par défaut au lieu de ne rien faire et de laisser l’utilisateur dans l’interrogation. La réponse par défaut doit indiquer à l’utilisateur les commandes comprises par le bot, de sorte qu’il puisse revenir sur la bonne voie.

Vous pouvez consulter l’indicateur de contexte **responded** à la fin de la logique du bot pour voir si le bot a renvoyé quelque chose à l’utilisateur durant ce tour. Si le bot traite l’entrée de l’utilisateur mais ne répond pas, il est probable qu’il ne sache pas comment gérer l’entrée. Dans ce cas, vous pouvez l’intercepter et envoyer un message par défaut à l’utilisateur.

Avec ce bot, l’action par défaut consiste à présenter à l’utilisateur la boîte de dialogue `mainMenu`. C’est à vous de décider de l’expérience dont bénéficiera votre utilisateur dans cette situation avec votre bot.

# <a name="ctabcsharptab"></a>[C#](#tab/csharptab)

```cs
if(!context.Responded)
{
    await dc.EndAll().Begin("mainMenu");
}
```

# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)

```javascript
// Check to see if anyone replied. If not then clear all the stack and present the main menu
if (!context.responded) {
    await dc.endAll().begin('mainMenu');
}
```

---

