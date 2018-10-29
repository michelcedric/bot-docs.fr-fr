---
title: Gérer l’état de la conversation et l’état utilisateur | Microsoft Docs
description: Découvrez comment enregistrer et récupérer des données d’état avec le Kit de développement logiciel (SDK) Bot Builder pour .NET.
keywords: état de la conversation, état utilisateur, flux de la conversation
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 09/18/18
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 972df2a12ffa7901ed4e4ecf14ce99233293c5a2
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2018
ms.locfileid: "49997706"
---
# <a name="manage-conversation-and-user-state"></a>Gérer l’état de la conversation et l’état utilisateur

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Une bonne conception de bot passe notamment par le suivi du contexte d’une conversation, afin que le bot se souvienne de choses telles que les réponses aux questions précédentes. Selon ce pour quoi votre bot est utilisé, vous pouvez même être amené à effectuer le suivi de l’état ou à stocker des informations plus longtemps que la durée de vie de la conversation. *L’état* d’un bot représente des informations dont il se souvient pour répondre correctement aux messages entrants. Le SDK Bot Builder fournit deux classes pour stocker et récupérer des données d’état sous la forme d’un objet associé à un utilisateur ou une conversation.

- L’**état de la conversation** permet au bot d’effectuer le suivi de la conversation en cours entre lui-même et l’utilisateur. Si votre bot doit exécuter une séquence d’étapes ou basculer entre des sujets de conversation, vous pouvez utiliser les propriétés de la conversation pour gérer les étapes d’une séquence ou effectuer le suivi du sujet actuel. 

- L’**état de l’utilisateur** peut servir à de nombreuses fins, par exemple pour déterminer l’endroit où une conversation précédente de l’utilisateur s’était arrêtée ou simplement pour accueillir un utilisateur régulier par son nom. Si vous stockez les préférences d’un utilisateur, vous pouvez utiliser ces informations pour personnaliser la prochaine conversation. Par exemple, vous pouvez avertir l’utilisateur de la publication d’un article sur un sujet qui l’intéresse ou bien l’informer de la disponibilité d’une date de rendez-vous. 

`ConversationState` et `UserState` sont des classes d’état, qui correspondent à des spécialisations de la classe `BotState`. Ces classes d’état comportent des stratégies permettant de contrôler la durée de vie et la portée des objets qu’elles stockent. Les composants qui doivent stocker l’état créent et enregistrent une propriété avec un type, et utilisent l’accesseur de propriétés pour accéder à l’état. Le gestionnaire d’état du bot peut utiliser le stockage en mémoire, CosmosDB et le stockage Blob. 

> [!NOTE] 
> Utilisez le gestionnaire d’état du bot pour stocker les préférences, le nom d’utilisateur ou le dernier élément commandé. En revanche, n’utilisez pas le gestionnaire pour stocker des données critiques. Pour les données critiques, créez vos propres composants de stockage ou écrivez directement sur le [stockage](bot-builder-howto-v4-storage.md).
> Le stockage de données en mémoire est uniquement destiné aux tests. Ce stockage est volatile et temporaire. Les données sont effacées chaque fois que le bot est redémarré.

## <a name="using-conversation-state-and-user-state-to-direct-conversation-flow"></a>Utiliser l’état de la conversation et l’état de l’utilisateur pour diriger le flux de la conversation
Lorsque vous concevez un flux de conversation, il est utile de définir un indicateur d’état afin de diriger le flux de la conversation. L’indicateur peut être un type booléen simple ou un type incluant le nom de la rubrique active. L’indicateur peut vous aider à vous situer dans une conversation. Par exemple, un indicateur de type booléen peut vous indiquer si vous vous trouvez ou non dans une conversation. Une propriété de nom de rubrique, quant à elle, peut vous indiquer dans quelle conversation vous vous trouvez actuellement.



# <a name="ctabcsharp"></a>[C#](#tab/csharp)
### <a name="conversation-and-user-state"></a>État de la conversation et de l’utilisateur
Vous pouvez utiliser l’[exemple de bot Écho avec compteur](https://aka.ms/EchoBot-With-Counter) en tant que point de départ pour cette procédure. Tout d’abord, créez la classe `TopicState` pour gérer la rubrique actuelle de conversation dans `TopicState.cs`, comme indiqué ci-dessous :

```csharp
public class TopicState
{
   public string Prompt { get; set; } = "askName";
}
``` 
Créez ensuite la classe `UserProfile` dans `UserProfile.cs` pour gérer l’état utilisateur.
```csharp
public class UserProfile
{
    public string UserName { get; set; }
    public string TelephoneNumber { get; set; }
}
``` 
La classe `TopicState` dispose d’un indicateur pour effectuer le suivi de l’état de la conversation. Elle stocke cet état. L’invite est initialisée en tant que « askName » pour lancer la conversation. Une fois que le bot reçoit la réponse de l’utilisateur, l’invite est redéfinie en tant que « askNumber » pour lancer la conversation suivante. La classe `UserProfile` identifie le nom d’utilisateur et le numéro de téléphone, et les stocke dans l’état utilisateur.

### <a name="property-accessors"></a>Accesseurs de propriété
La classe `EchoBotAccessors` de notre exemple est créée en tant que singleton, et est transférée au constructeur `class EchoWithCounterBot : IBot` via l’injection de dépendances. La classe `EchoBotAccessors` contient les éléments `ConversationState`, `UserState`, ainsi que l’élément `IStatePropertyAccessor` associé. L’objet `conversationState` stocke l’état du sujet, et l’objet `userState` stocke les informations relatives au profil utilisateur. Les objets `ConversationState` et `UserState` seront créés ultérieurement dans le fichier Startup.cs. Les objets d’état utilisateur et de conversation nous permettent de conserver toutes les données relatives à la portée de la conversation et de l’utilisateur. 

Mise à jour du constructeur afin d’inclure `UserState` comme indiqué ci-dessous :
```csharp
using EchoBotWithCounter;

public EchoBotAccessors(ConversationState conversationState, UserState userState)
{
    ConversationState = conversationState ?? throw new ArgumentNullException(nameof(conversationState));
    UserState = userState ?? throw new ArgumentNullException(nameof(userState));
}
```
Créez des noms uniques pour les accesseurs `TopicState` et `UserProfile`.
```csharp
public static string UserProfileName { get; } = $"{nameof(EchoBotAccessors)}.UserProfile";
public static string TopicStateName { get; } = $"{nameof(EchoBotAccessors)}.TopicState";
```
Ensuite, définissez deux accesseurs. Le premier stocke le sujet de la conversation, et le deuxième stocke le nom d’utilisateur et le numéro de téléphone.
```csharp
public IStatePropertyAccessor<TopicState> TopicState { get; set; }
public IStatePropertyAccessor<UserProfile> UserProfile { get; set; }
```

Les propriétés permettant d’obtenir ConversationState (état de la conversation) sont déjà définies, mais vous devez ajouter `UserState` comme indiqué ci-dessous :
```csharp
public ConversationState ConversationState { get; }
public UserState UserState { get; }
```
Enregistrez le fichier après avoir apporté les modifications. Ensuite, nous mettons à jour la classe Startup pour créer l’objet `UserState` afin de conserver toutes les informations au niveau de l’utilisateur. La classe `ConversationState` existe déjà. 
```csharp

services.AddBot<EchoWithCounterBot>(options =>
{
    ...

    IStorage dataStore = new MemoryStorage();
    
    var conversationState = new ConversationState(dataStore);
    options.State.Add(conversationState);
        
    var userState = new UserState(dataStore);  
    options.State.Add(userState);
});
```
Les lignes `options.State.Add(ConversationState);` et `options.State.Add(userState);` permettent d’ajouter respectivement l’état de la conversation et l’état de l’utilisateur. Ensuite, créez et enregistrez des accesseurs d’état. Les accesseurs créés ici sont transmis dans la classe dérivée de IBot à chaque tour. Modifiez le code afin d’inclure l’état de l’utilisateur comme indiqué ci-dessous :
```csharp
services.AddSingleton<EchoBotAccessors>(sp =>
{
   ...
    var userState = options.State.OfType<UserState>().FirstOrDefault();
    if (userState == null)
    {
        throw new InvalidOperationException("UserState must be defined and added before adding user-scoped state accessors.");
    }
   ...
 });
```

Ensuite, créez les deux accesseurs à l’aide de `TopicState` et `UserProfile`, et transmettez-les à la classe `class EchoWithCounterBot : IBot` via l’injection de dépendances.
```csharp
services.AddSingleton<EchoBotAccessors>(sp =>
{
   ...
    var accessors = new EchoBotAccessors(conversationState, userState)
    {
        TopicState = conversationState.CreateProperty<TopicState>(EchoBotAccessors.TopicStateName),
        UserProfile = userState.CreateProperty<UserProfile>(EchoBotAccessors.UserProfileName),
     };

     return accessors;
 });
```

L’état de la conversation et l’état utilisateur sont liés à un singleton via le bloc de code `services.AddSingleton`. Ils sont enregistrés via un accesseur de magasin d’état dans le code qui commence par `var accessors = new EchoBotAccessor(conversationState, userState)`.

### <a name="use-conversation-and-user-state-properties"></a>Utiliser les propriétés d’état de la conversation et d’état utilisateur 
Dans le gestionnaire `OnTurnAsync` de la classe `EchoWithCounterBot : IBot`, modifiez le code pour demander à l’utilisateur son nom d’utilisateur et numéro de téléphone. Pour savoir là où nous nous trouvons dans la conversation, nous utilisons la propriété d’invite définie dans TopicState (état du sujet). Cette propriété était initialisée sur « askName ». Une fois que nous obtenons le nom d’utilisateur, nous définissons la propriété sur « askNumber » et récupérons le nom que l’utilisateur a saisi. Une fois que vous avez reçu le numéro de téléphone, vous envoyez un message de confirmation et définissez l’invite sur « confirmation », car vous touchez à la fin de la conversation.

```csharp
using EchoBotWithCounter;

if (turnContext.Activity.Type == ActivityTypes.Message)
{
    // Get the conversation state from the turn context.
    var convo = await _accessors.TopicState.GetAsync(turnContext, () => new TopicState());
    
    // Get the user state from the turn context.
    var user = await _accessors.UserProfile.GetAsync(turnContext, () => new UserProfile());
    
    // Ask user name. The Prompt was initialiazed as "askName" in the TopicState.cs file.
    if (convo.Prompt == "askName")
    {
        await turnContext.SendActivityAsync("What is your name?");
        
        // Set the Prompt to ask the next question for this conversation
        convo.Prompt = "askNumber";
        
        // Set the property using the accessor
        await _accessors.TopicState.SetAsync(turnContext, convo);
        
        //Save the new prompt into the conversation state.
        await _accessors.ConversationState.SaveChangesAsync(turnContext);
    }
    else if (convo.Prompt == "askNumber")
    {
        // Set the UserName that is defined in the UserProfile class
        user.UserName = turnContext.Activity.Text;
        
        // Use the user name to prompt the user for phone number
        await turnContext.SendActivityAsync($"Hello, {user.UserName}. What's your telephone number?");
        
        // Set the Prompt now that we have collected all the data
        convo.Prompt = "confirmation";
                 
        await _accessors.TopicState.SetAsync(turnContext, convo);
        await _accessors.ConversationState.SaveChangesAsync(turnContext);

        await _accessors.UserProfile.SetAsync(turnContext, user);
        await _accessors.UserState.SaveChangesAsync(turnContext);
    }
    else if (convo.Prompt == "confirmation")
    { 
        // Set the TelephoneNumber that is defined in the UserProfile class
        user.TelephoneNumber = turnContext.Activity.Text;

        await turnContext.SendActivityAsync($"Got it, {user.UserName}. I'll call you later.");

        // reset initial prompt state
        convo.Prompt = "askName"; // Reset for a new conversation.
        
        await _accessors.TopicState.SetAsync(turnContext, convo);
        await _accessors.ConversationState.SaveChangesAsync(turnContext);
    }
}
```   

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

### <a name="conversation-and-user-state"></a>État de la conversation et de l’utilisateur

Vous pouvez utiliser l’[exemple de bot Écho avec compteur](https://aka.ms/EchoBot-With-Counter-JS) en tant que point de départ pour cette procédure. Cet exemple utilise déjà `ConversationState` pour stocker le nombre de messages. Nous devons ajouter un objet `TopicStates` pour effectuer le suivi de la conversation et un objet `UserState` pour effectuer le suivi des informations utilisateur dans un objet `userProfile`. 

Dans le fichier `index.js` principal du bot, ajoutez `UserState` à la liste d’exigences :

**index.js**

```javascript
// Import required bot services. See https://aka.ms/bot-services to learn more about the different parts of a bot.
const { BotFrameworkAdapter, MemoryStorage, ConversationState, UserState } = require('botbuilder');
```

Ensuite, créez `UserState` à l’aide de `MemoryStorage` en tant que fournisseur de stockage, puis transmettez l’objet en tant que second argument de la classe `MainDialog`.

**index.js**

```javascript
// Create conversation state with in-memory storage provider. 
const conversationState = new ConversationState(memoryStorage);
const userState = new UserState(memoryStorage);
// Create the main bot.
const bot = new EchBot(conversationState, userState);
```

Dans votre fichier `bot.js`, mettez à jour le constructeur pour accepter `userState` en tant que second argument. Créez ensuite une propriété `topicState` à partir de `conversationState` et créez une propriété `userProfile` à partir de `userState`.

**bot.js**

```javascript
const TOPIC_STATE = 'topic';
const USER_PROFILE = 'user';

constructor (conversationState, userState) {
    // creates a new state accessor property.see https://aka.ms/about-bot-state-accessors to learn more about the bot state and state accessors 
    this.conversationState = conversationState;
    this.topicState = this.conversationState.createProperty(TOPIC_STATE);

    // User state
    this.userState = userState;
    this.userProfile = this.userState.createProperty(USER_PROFILE);
}
```

### <a name="use-conversation-and-user-state-properties"></a>Utiliser les propriétés d’état de la conversation et d’état utilisateur

Dans le gestionnaire `onTurn` de la classe `MainDialog`, modifiez le code pour demander à l’utilisateur son nom d’utilisateur et numéro de téléphone. Pour savoir là où nous nous trouvons dans la conversation, nous utilisons la propriété `prompt` définie dans `topicState`. Cette propriété est initialisée sur « askName ». Une fois que nous obtenons le nom d’utilisateur, nous définissons la propriété sur « askNumber » et récupérons le nom que l’utilisateur a saisi. Une fois que vous avez reçu le numéro de téléphone, vous envoyez un message de confirmation et définissez l’invite sur `undefined`, car vous touchez à la fin de la conversation.

**dialogs/mainDialog/index.js**

```javascript
// see https://aka.ms/about-bot-activity-message to learn more about the message and other activity types
if (turnContext.activity.type === 'message') {
    // read from state and set default object if object does not exist in storage.
    let topicState = await this.topicState.get(turnContext, {
        //Define the topic state object
        prompt: "askName"
    });
    let userProfile = await this.userProfile.get(turnContext, {  
        // Define the user's profile object
        "userName": "",
        "telephoneNumber": ""
    });

    if(topicState.prompt == "askName"){
        await turnContext.sendActivity("What is your name?");

        // Set next prompt state
        topicState.prompt = "askNumber";

        // Update state
        await this.topicState.set(turnContext, topicState);
    }
    else if(topicState.prompt == "askNumber"){
        // Set the UserName that is defined in the UserProfile class
        userProfile.userName = turnContext.activity.text;

        // Use the user name to prompt the user for phone number
        await turnContext.sendActivity(`Hello, ${userProfile.userName}. What's your telephone number?`);

        // Set next prompt state
        topicState.prompt = "confirmation";

        // Update states
        await this.topicState.set(turnContext, topicState);
        await this.userProfile.set(turnContext, userProfile);
    }
    else if(topicState.prompt == "confirmation"){
        // Set the phone number
        userProfile.telephoneNumber = turnContext.activity.text;

        // Sent confirmation
        await turnContext.sendActivity(`Got it, ${userProfile.userName}. I'll call you later.`)

        // reset initial prompt state
        topicState.prompt = "askName"; // Reset for a new conversation

        // Update states
        await this.topicState.set(turnContext, topicState);
        await this.userProfile.set(turnContext, userProfile);
    }
    
    // Save state changes to storage
    await this.conversationState.saveChanges(turnContext);
    await this.userState.saveChanges(turnContext);
    
}
else {
    await turnContext.sendActivity(`[${context.activity.type} event detected]`);
}
```

---

## <a name="start-your-bot"></a>Démarrer votre robot
Exécutez votre bot en local.

### <a name="start-the-emulator-and-connect-your-bot"></a>Démarrer l’émulateur et connecter votre robot
Démarrez à présent l’émulateur, puis connectez-vous à votre bot dans l’émulateur :

1. Cliquez sur le lien **Ouvrir le bot** de l’onglet de bienvenue de l’émulateur. 
2. Sélectionnez le fichier .bot situé dans le répertoire où vous avez créé le projet Visual Studio.

### <a name="interact-with-your-bot"></a>Interagir avec votre bot

Envoyez un message à votre bot, et le bot vous enverra un message à son tour.
![Émulateur en cours d’exécution](../media/emulator-v4/emulator-running.png)

Si vous décidez de gérer l’état vous-même, consultez l’article relatif à la [gestion du flux de conversation en utilisant vos propres invites](bot-builder-primitive-prompts.md). Une alternative consiste à utiliser le dialogue en cascade. Le dialogue suit l’état de la conversation pour vous, ce qui vous évite d’avoir à créer des indicateurs pour suivre votre état. Pour plus d’informations, consultez l’article [Gérer un flux de conversation simple avec des dialogues](bot-builder-dialog-manage-conversation-flow.md).

## <a name="next-steps"></a>Étapes suivantes
Maintenant que vous savez comment utiliser l’état pour vous aider à lire et à enregistrer des données bot dans le stockage, voyons comment lire et enregistrer ces données directement dans le stockage.

> [!div class="nextstepaction"]
> [Guide pratique pour écrire directement dans le stockage](bot-builder-howto-v4-storage.md).
