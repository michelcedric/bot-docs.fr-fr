---
title: Enregistrer les données d’utilisateur et de conversation | Microsoft Docs
description: Découvrez comment enregistrer et récupérer des données d’état avec le SDK Bot Builder.
keywords: état de conversation, état d’utilisateur, conversation, enregistrement de l’état, gestion de l’état d’un bot
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 11/26/18
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 8f979aed3bc1c4bb4c74629bcffb258e139ce77d
ms.sourcegitcommit: bcde20bd4ab830d749cb835c2edb35659324d926
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/27/2018
ms.locfileid: "52338552"
---
# <a name="save-user-and-conversation-data"></a>Enregistrer les données d’utilisateur et de conversation

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Un bot est par nature sans état. Une fois que votre bot est déployé, il ne peut pas s’exécuter dans le même processus ni sur le même ordinateur d’un tour à l’autre. Votre bot peut toutefois suivre le contexte d’une conversation pour pouvoir gérer son comportement et se rappeler des réponses aux questions précédentes. Les fonctionnalités d’état et de stockage du SDK permettent d’ajouter un état à votre bot.

## <a name="prerequisites"></a>Prérequis

- Connaissances des [concepts de base des bots](bot-builder-basics.md) et de la façon dont ils [gèrent l’état](bot-builder-concept-state.md).
- Le code de cet article est basé sur l’exemple **StateBot**. Vous aurez besoin d’une copie de l’exemple en [C# ](https://github.com/Microsoft/BotFramework-Samples/tree/master/SDKV4-Samples/dotnet_core/StateBot) ou en [JS]().
- [Bot Framework Emulator](https://aka.ms/Emulator-wiki-getting-started) pour tester le bot localement.

## <a name="about-the-sample-code"></a>Au sujet de l’exemple de code

Cet article décrit comment configurer la gestion de l’état d’un bot. Pour ajouter un état, nous devons configurer des propriétés d’état, la gestion de l’état et le stockage, puis les utiliser dans le bot.

- Chaque _propriété d’état_ contient des informations d’état sur le bot.
- Chaque accesseur de propriété d’état vous permet d’obtenir ou de définir la valeur de la propriété d’état associée.
- Chaque objet de gestion d’état automatise la lecture et l’écriture des informations d’état associées dans le stockage.
- La couche de stockage se connecte au stockage de sauvegarde de l’état, notamment le stockage en mémoire (pour les tests) ou Azure Cosmos DB (pour la production).

Nous devons configurer le bot avec des accesseurs de propriété d’état à l’aide desquels il peut obtenir et définir l’état au moment de l’exécution quand il traite une activité. Un accesseur de propriété d’état est créé à l’aide d’un objet de gestion d’état, lequel est créé à l’aide d’une couche de stockage. Nous allons donc débuter par la couche de stockage.

## <a name="configure-storage"></a>Configurer le stockage

Étant donné que nous ne prévoyons pas de déployer ce bot, nous allons utiliser le _stockage mémoire_ pour configurer un état d’utilisateur et un état de conversation à l’étape suivante.

### <a name="ctabcsharp"></a>[C#](#tab/csharp)

Dans **Startup.cs**, configurez la couche de stockage.

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // ...
    IStorage storage = new MemoryStorage();
    // ...
}
```

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Dans votre fichier **index.js**, configurez la couche de stockage.

```javascript
// Define state store for your bot.
const memoryStorage = new MemoryStorage();
```

---

## <a name="create-state-management-objects"></a>Créer des objets de gestion d’état

Nous suivons à la fois l’état d’_utilisateur_ et l’état de _conversation_ que nous utilisons pour créer les _accesseurs de propriété d’état_ à l’étape suivante.

### <a name="ctabcsharp"></a>[C#](#tab/csharp)

Dans **Startup.cs**, référencez la couche de stockage quand vous créez vos objets de gestion d’état.

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // ...
    ConversationState conversationState = new ConversationState(storage);
    UserState userState = new UserState(storage);
    // ...
}
```

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Dans votre fichier **index.js**, ajoutez `UserState` à votre instruction require.

```javascript
const { BotFrameworkAdapter, MemoryStorage, ConversationState, UserState } = require('botbuilder');
```

Ensuite, référencez la couche de stockage quand vous créez vos objets de gestion d’état de conversation et d’utilisateur.

```javascript
// Create conversation and user state with in-memory storage provider.
const conversationState = new ConversationState(memoryStorage);
const userState = new UserState(memoryStorage);
```

---

## <a name="create-state-property-accessors"></a>Créer des accesseurs de propriété d’état

Pour _déclarer_ une propriété d’état, commencez par créer un accesseur de propriété d’état à l’aide de l’un de nos objets de gestion d’état. Nous configurons le bot pour suivre les informations suivantes :

- Le nom de l’utilisateur, que nous allons définir dans l’état d’utilisateur.
- si, oui ou non, nous avons demandé à l’utilisateur son nom ainsi que des informations supplémentaires sur le message qu’il vient d’envoyer.

Le bot utilise l’accesseur pour obtenir la propriété d’état à partir du contexte de dialogue.

### <a name="ctabcsharp"></a>[C#](#tab/csharp)

Nous commençons par définir des classes pour héberger toutes les informations à gérer dans chaque type d’état.

- Une classe `UserProfile` pour les informations utilisateur qui seront collectées par le bot.
- Une classe `ConversationData` pour suivre les informations sur l’arrivée d’un message et son expéditeur.

```csharp
// Defines a state property used to track information about the user.
public class UserProfile
{
    public string Name { get; set; }
}
```

```csharp
// Defines a state property used to track conversation data.
public class ConversationData
{
    // The time-stamp of the most recent incoming message.
    public string Timestamp { get; set; }

    // The ID of the user's channel.
    public string ChannelId { get; set; }

    // Track whether we have already asked the user's name
    public bool PromptedUserForName { get; set; } = false;
}
```

Ensuite, nous définissons une classe qui contient les informations de gestion d’état dont nous aurons besoin pour configurer notre instance de bot.

```csharp
public class StateBotAccessors
{
    public StateBotAccessors(ConversationState conversationState, UserState userState)
    {
        ConversationState = conversationState ?? throw new ArgumentNullException(nameof(conversationState));
        UserState = userState ?? throw new ArgumentNullException(nameof(userState));
    }
  
    public static string UserProfileName { get; } = "UserProfile";

    public static string ConversationDataName { get; } = "ConversationData";

    public IStatePropertyAccessor<UserProfile> UserProfileAccessor { get; set; }

    public IStatePropertyAccessor<ConversationData> ConversationDataAccessor { get; set; }
  
    public ConversationState ConversationState { get; }
  
    public UserState UserState { get; }
}
```

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Nous passons les objets de gestion d’état directement au constructeur du bot et laissons le bot créer lui-même les accesseurs de propriété d’état.

Dans **index.js**, indiquez les objets de gestion d’état quand vous créez le bot.

```javascript
// Create the bot.
const myBot = new MyBot(conversationState, userState);
```

Dans **bot.js**, définissez les identificateurs dont vous aurez besoin pour gérer et suivre l’état.

```javascript
// The accessor names for the conversation data and user profile state property accessors.
const CONVERSATION_DATA_PROPERTY = 'conversationData';
const USER_PROFILE_PROPERTY = 'userProfile';
```

---

## <a name="configure-your-bot"></a>Configurer votre bot

Nous sommes maintenant prêts à définir les accesseurs de propriété d’état et à configurer notre bot.
Nous utilisons l’objet de gestion d’état de conversation pour l’accesseur de propriété d’état du flux conversationnel.
Quant à l’objet de gestion d’état de conversation, nous l’utilisons pour l’accesseur de propriété d’état du profil utilisateur.

### <a name="ctabcsharp"></a>[C#](#tab/csharp)

Dans **Startup.cs**, nous configurons ASP.NET pour fournir les objets de gestion et de propriété d’état groupés. Ils sont récupérés auprès du constructeur du bot au moyen du framework d’injection de dépendance dans ASP.NET Core.

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // ...
    services.AddSingleton<StateBotAccessors>(sp =>
    {
        // Create the custom state accessor.
        return new StateBotAccessors(conversationState, userState)
        {
            ConversationDataAccessor = conversationState.CreateProperty<ConversationData>(StateBotAccessors.ConversationDataName),
            UserProfileAccessor = userState.CreateProperty<UserProfile>(StateBotAccessors.UserProfileName),
        };
    });
}
```

Dans le constructeur du bot, l’objet `CustomPromptBotAccessors` est fourni quand ASP.NET crée le bot.

```csharp
// Defines a bot for filling a user profile.
public class CustomPromptBot : IBot
{
    private readonly StateBotAccessors _accessors;

    public StateBot(StateBotAccessors accessors, ILoggerFactory loggerFactory)
    {
        // ...
        accessors = accessors ?? throw new System.ArgumentNullException(nameof(accessors));
    }

    // The bot's turn handler and other supporting code...
}
```

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Dans le constructeur du bot (dans le fichier **bot.js**), nous créons les accesseurs de propriété d’état et les ajoutons au bot. Nous ajoutons également des références aux objets de gestion d’état, car nous en avons besoin pour enregistrer tout changement d’état.

```javascript
constructor(conversationState, userState) {
    // Create the state property accessors for the conversation data and user profile.
    this.conversationData = conversationState.createProperty(CONVERSATION_DATA_PROPERTY);
    this.userProfile = userState.createProperty(USER_PROFILE_PROPERTY);

    // The state management objects for the conversation and user state.
    this.conversationState = conversationState;
    this.userState = userState;
}
```

---

## <a name="access-state-from-your-bot"></a>Accéder à l’état à partir du bot

Dans les sections précédentes, nous avons suivi les étapes nécessaires au moment de l’initialisation pour ajouter des accesseurs de propriété d’état à notre bot.
Nous pouvons à présent utiliser ces accesseurs au moment de l’exécution pour lire et écrire des informations d’état.

1. Avant d’utiliser nos propriétés d’état, nous devons utiliser chaque accesseur pour charger la propriété à partir du stockage et la récupérer dans le cache d’état.
   - Chaque fois que vous obtenez une propriété d’état par le biais de son accesseur, vous devez fournir une valeur par défaut. Sinon, vous pouvez obtenir une erreur « Valeur null ».
1. Avant de quitter le gestionnaire de tour :
   1. Nous utilisons la méthode _set_ de l’accesseur pour envoyer (push) les changements à l’état du bot.
   1. Nous utilisons la méthode d’_enregistrement des changements_ des objets de gestion d’état pour écrire ces changements dans le stockage.

### <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// The bot's turn handler.
public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
{
    if (turnContext.Activity.Type == ActivityTypes.Message)
    {
        // Get the state properties from the turn context.
        UserProfile userProfile =
            await _accessors.UserProfileAccessor.GetAsync(turnContext, () => new UserProfile());
        ConversationData conversationData =
            await _accessors.ConversationDataAccessor.GetAsync(turnContext, () => new ConversationData());

        if (string.IsNullOrEmpty(userProfile.Name))
        {
            // First time around this is set to false, so we will prompt user for name.
            if (conversationData.PromptedUserForName)
            {
                // Set the name to what the user provided
                userProfile.Name = turnContext.Activity.Text?.Trim();

                // Acknowledge that we got their name.
                await turnContext.SendActivityAsync($"Thanks {userProfile.Name}.");

                // Reset the flag to allow the bot to go though the cycle again.
                conversationData.PromptedUserForName = false;
            }
            else
            {
                // Prompt the user for their name.
                await turnContext.SendActivityAsync($"What is your name?");

                // Set the flag to true, so we don't prompt in the next turn.
                conversationData.PromptedUserForName = true;
            }

            // Save user state and save changes.
            await _accessors.UserProfileAccessor.SetAsync(turnContext, userProfile);
            await _accessors.UserState.SaveChangesAsync(turnContext);
        }
        else
        {
            // Add message details to the conversation data.
            conversationData.Timestamp = turnContext.Activity.Timestamp.ToString();
            conversationData.ChannelId = turnContext.Activity.ChannelId.ToString();

            // Display state data
            await turnContext.SendActivityAsync($"{userProfile.Name} sent: {turnContext.Activity.Text}");
            await turnContext.SendActivityAsync($"Message received at: {conversationData.Timestamp}");
            await turnContext.SendActivityAsync($"Message received from: {conversationData.ChannelId}");
        }

        // Update conversation state and save changes.
        await _accessors.ConversationDataAccessor.SetAsync(turnContext, conversationData);
        await _accessors.ConversationState.SaveChangesAsync(turnContext);
    }
}
```

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// The bot's turn handler.
async onTurn(turnContext) {
    if (turnContext.activity.type === ActivityTypes.Message) {
        // Get the state properties from the turn context.
        const userProfile = await this.userProfile.get(turnContext, {});
        const conversationData = await this.conversationData.get(
            turnContext, { promptedForUserName: false });

        if (!userProfile.name) {
            // First time around this is undefined, so we will prompt user for name.
            if (conversationData.promptedForUserName) {
                // Set the name to what the user provided.
                userProfile.name = turnContext.activity.text;

                // Acknowledge that we got their name.
                await turnContext.sendActivity(`Thanks ${userProfile.name}.`);

                // Reset the flag to allow the bot to go though the cycle again.
                conversationData.promptedForUserName = false;
            } else {
                // Prompt the user for their name.
                await turnContext.sendActivity('What is your name?');

                // Set the flag to true, so we don't prompt in the next turn.
                conversationData.promptedForUserName = true;
            }
            // Save user state and save changes.
            await this.userProfile.set(turnContext, userProfile);
            await this.userState.saveChanges(turnContext);
        } else {
            // Add message details to the conversation data.
            conversationData.timestamp = turnContext.activity.timestamp.toLocaleString();
            conversationData.channelId = turnContext.activity.channelId;

            // Display state data.
            await turnContext.sendActivity(`${userProfile.name} sent: ${turnContext.activity.text}`);
            await turnContext.sendActivity(`Message received at: ${conversationData.timestamp}`);
            await turnContext.sendActivity(`Message received from: ${conversationData.channelId}`);
        }
        // Update conversation state and save changes.
        await this.conversationData.set(turnContext, conversationData);
        await this.conversationState.saveChanges(turnContext);
    }
}
```

---

## <a name="test-the-bot"></a>Tester le bot

1. Exécutez l’exemple en local sur votre machine. Si vous avez besoin d’instructions, consultez le fichier LISEZ-MOI pour l’exemple en [C# ](https://github.com/Microsoft/BotFramework-Samples/tree/master/SDKV4-Samples/dotnet_core/StateBot) ou en [JS](https://github.com/Microsoft/BotFramework-Samples/tree/master/SDKV4-Samples/js/stateBot).
1. Utilisez l’émulateur pour tester le bot comme indiqué ci-dessous.

![exemple de bot avec état à des fins de test](media/state-bot-testing-emulator.png)

## <a name="additional-resources"></a>Ressources supplémentaires

**Confidentialité** : si vous prévoyez de stocker les données personnelles de l’utilisateur, vous devez veiller au respect du [Règlement général sur la protection des données](https://blog.botframework.com/2018/04/23/general-data-protection-regulation-gdpr).

**Gestion d’état** : tous les appels de gestion d’état sont asynchrones et, par défaut, le dernier qui écrit gagne (last-writer-wins). Dans la pratique, vous devez obtenir, définir et enregistrer l’état dans votre bot de la façon la plus rapprochée possible.

**Données métier critiques** : utilisez l’état du bot pour stocker les préférences, le nom d’utilisateur ou le dernier élément commandé, mais ne l’utilisez pas pour stocker des données métiers critiques. Pour les données critiques, [créez vos propres composants de stockage](bot-builder-custom-storage.md) ou écrivez directement dans le [stockage](bot-builder-howto-v4-storage.md).

**Recognizers-Text** : l’exemple utilise les bibliothèques Microsoft/Recognizers-Text pour analyser et valider les entrées utilisateur. Pour plus d’informations, consultez la page de [présentation](https://github.com/Microsoft/Recognizers-Text#microsoft-recognizers-text-overview).

## <a name="next-step"></a>Étape suivante

Maintenant que vous savez comment configurer l’état pour vous aider à lire et à écrire les données d’un bot dans le stockage, examinons comment poser une série de questions à l’utilisateur, valider ses réponses et enregistrer ses entrées.

> [!div class="nextstepaction"]
> [Créez vos propres invites pour collecter les entrées utilisateur](bot-builder-primitive-prompts.md).
