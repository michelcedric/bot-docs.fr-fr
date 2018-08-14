---
title: Traduire l’entrée utilisateur pour rendre votre bot multilingue | Microsoft Docs
description: Découvrez comment traduire automatiquement l’entrée utilisateur dans la langue native de votre bot et la retraduire dans la langue de l’utilisateur.
keywords: traduction, traduire, multilingue, microsoft translator
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 04/06/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 13139755989afccd85b2e09267dc42619ec1f83c
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39299024"
---
# <a name="translate-user-input-to-make-your-bot-multilingual"></a>Traduire l’entrée utilisateur pour rendre votre bot multilingue

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Votre bot peut utiliser [Microsoft Translator](https://www.microsoft.com/en-us/translator/) pour traduire automatiquement des messages dans la langue qu’il comprend et éventuellement retraduire les réponses du bot dans la langue de l’utilisateur. L’ajout d’une traduction à votre bot vous permet d’atteindre un public plus large sans apporter de modifications majeures à la programmation centrale de votre bot.
<!-- 
- [Get a Text Services key](#get-a-text-services-key)
- [Installing Packages](#installing-packages)
- [Combine translation with LUIS or QnA Maker](#using-luis)
- [Combine translation with QnA Maker](#using-qna-maker)
-->

Dans ce didacticiel, nous allons utiliser le service Microsoft Translator pour la traduction, puis ajouter cette dernière à un bot simple afin de comprendre la manière dont elle fonctionne.

## <a name="get-a-text-services-key"></a>Obtenir une clé de services de texte

Pour commencer, l’utilisation du service Traduction de texte Translator Text nécessite une clé. Vous pouvez obtenir une [clé d’essai gratuit](https://www.microsoft.com/en-us/translator/trial.aspx#get-started) dans le Portail Azure.

## <a name="installing-packages"></a>Installation des packages

Assurez-vous que vous avez installé les packages nécessaires pour ajouter une traduction à votre bot.

# <a name="ctabcs"></a>[C#](#tab/cs)

[Ajoutez une référence](https://docs.microsoft.com/en-us/nuget/tools/package-manager-ui) à la préversion des packages NuGet suivants :

* `Microsoft.Bot.Builder.Integration.AspNet.Core`
* `Microsoft.Bot.Builder.Ai.Translation` (requis pour la traduction)

Si vous voulez combiner une traduction avec Language Understanding (LUIS), ajoutez également une référence à :

* `Microsoft.Bot.Builder.Ai.Luis` (requis pour LUIS)

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

L’un de ces services peut être ajouté à votre bot à l’aide du package botbuilder-ai. Vous pouvez ajouter ce package à votre projet par le biais de npm :
* `npm install --save botbuilder@preview`
* `npm install --save botbuilder-ai@preview`

---

## <a name="configure-translation"></a>Configurer la traduction

Ensuite, vous pouvez configurer votre bot afin d’appeler le traducteur pour chaque message reçu d’un utilisateur, en l’ajoutant simplement à la pile de l’intergiciel (middleware) de votre bot. L’intergiciel utilise la traduction obtenue pour modifier le message de l’utilisateur à l’aide de l’objet de contexte.


# <a name="ctabcs"></a>[C#](#tab/cs)

Démarrez avec l’exemple EchoBot du Kit de développement logiciel (SDK) v4 et mettez à jour la méthode `ConfigureServices` de votre fichier `Startup.cs` afin d’ajouter `TranslationMiddleware` au bot. Cette opération configure votre bot afin que chaque message reçu d’un utilisateur soit traduit. <!--, by simply adding it to your bot's middleware set. The middleware stores the translation results on the context object. -->
-   Mettez à jour vos instructions using.
-   Mettez à jour votre méthode `ConfigureServices` pour y inclure l’intergiciel de traduction.

    Nous avons simplifié l’extrait de code présenté ici en supprimant la plupart des commentaires, ainsi que l’intergiciel de gestion des exceptions.

**Startup.cs**
```csharp
using System;
using System.Collections.Generic;
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Hosting;
using Microsoft.Bot.Builder.Ai.Translation;
using Microsoft.Bot.Builder.BotFramework;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Builder.Integration.AspNet.Core;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Logging;
```
```csharp
// This method gets called by the runtime. Use this method to add services to the container.
public void ConfigureServices(IServiceCollection services)
{
    services.AddBot<EchoBot>(options =>
    {
        options.CredentialProvider = new ConfigurationCredentialProvider(Configuration);
        var middleware = options.Middleware;
        // Add translation middleware
        // The first parameter is a list of languages the bot recognizes
        middleware.Add(new TranslationMiddleware(new string[] { "en" }, "<YOUR MICROSOFT TRANSLATOR TEXT API KEY>", false));

    });
}


```

> [!TIP] 
> Le Kit de développement logiciel (SDK) Bot Builder détecte automatiquement la langue de l’utilisateur en fonction du message que ce dernier vient d’envoyer. Pour remplacer cette fonctionnalité, vous pouvez fournir des paramètres de rappel supplémentaires afin d’ajouter votre propre logique de détection et de changement de la langue de l’utilisateur.  



Examinons le code du fichier `EchoBot.cs`, qui envoie le texte « You sent » (Vous avez envoyé) suivi du message de l’utilisateur :

```cs
using Microsoft.Bot.Builder;
using Microsoft.Bot.Schema;
using System.Threading.Tasks;

namespace Microsoft.Bot.Samples
{
    public class EchoBot : IBot
    {
        public EchoBot() { }

        public async Task OnTurn(ITurnContext context)
        {
            switch (context.Activity.Type)
            {
                case ActivityTypes.Message:
                // echo back the user's input.
                    await context.SendActivity($"You sent '{context.Activity.Text}'");


                case ActivityTypes.ConversationUpdate:
                    foreach (var newMember in context.Activity.MembersAdded)
                    {
                        if (newMember.Id != context.Activity.Recipient.Id)
                        {
                            await context.SendActivity("Hello and welcome to the echo bot.");
                        }
                    }
                    break;
            }
        }
        
    }
}
```

Lorsque vous ajoutez un intergiciel (middleware) de traduction, un paramètre facultatif spécifie s’il faut retraduire les réponses dans la langue de l’utilisateur. Dans `Startup.cs`, nous avons spécifié `false` pour simplement traduire les messages de l’utilisateur dans la langue du bot.

```cs
// The first parameter is a list of languages the bot recognizes
// The second parameter is your API key
// The third parameter specifies whether to translate the bot's replies back to the user's language
middleware.Add(new TranslationMiddleware(new string[] { "en" }, "<YOUR MICROSOFT TRANSLATOR TEXT API KEY>", false));
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

Pour configurer un intergiciel (middleware) de traduction avec un bot d’écho, collez le code ci-après dans app.js.

```javascript
const { BotFrameworkAdapter, MemoryStorage, ConversationState } = require('botbuilder');
const { LanguageTranslator, LocaleConverter } = require('botbuilder-ai');
const restify = require('restify');

// Create server
let server = restify.createServer();
server.listen(process.env.port || process.env.PORT || 3978, function () {
    console.log(`${server.name} listening to ${server.url}`);
});

// Create adapter
const adapter = new BotFrameworkAdapter({ 
    appId: process.env.MICROSOFT_APP_ID, 
    appPassword: process.env.MICROSOFT_APP_PASSWORD 
});

// Add language translator middleware
const languageTranslator = new LanguageTranslator({
    translatorKey: "xxxxxx",
    noTranslatePatterns: new Set(),
    nativeLanguages: ['en'] 
});
adapter.use(languageTranslator);


// Listen for incoming requests 
server.post('/api/messages', (req, res) => {
    // Route received request to adapter for processing
    adapter.processActivity(req, res, async (context) => {
        if (context.activity.type === 'message') {
            await context.sendActivity(`You said "${context.activity.text}"`);
        } else {
            await context.sendActivity(`[${context.activity.type} event detected]`);
        }
    });
});
```

---

## <a name="run-the-bot-and-see-translated-input"></a>Exécuter le bot et afficher les entrées traduites

Exécutez le bot et tapez quelques messages dans d’autres langues. Vous verrez que le bot a traduit le message de l’utilisateur et indique la traduction dans sa réponse.

![Le bot détecte la langue et traduit l’entrée](./media/how-to-bot-translate/bot-detects-language-translates-input.png)




## <a name="invoke-logic-in-the-bots-native-language"></a>Appeler une logique dans la langue native du bot

Maintenant, ajoutez une logique qui recherche les termes anglais. Si l’utilisateur tape « help » ou « cancel » dans une autre langue, le bot traduit le texte en anglais, et la logique qui vérifie les mots anglais « help » ou « cancel » est appelée.

# <a name="ctabcs"></a>[C#](#tab/cs)
Dans `EchoBot.cs`, mettez à jour l’instruction `case` pour les activités de message dans la méthode `OnTurn` de votre bot.
```cs
case ActivityTypes.Message:
    // check the message text before calling context.SendActivity
    var messagetext = context.Activity.Text.Trim().ToLower();
    if (string.Equals("help", messagetext))
    {
        await context.SendActivity("You asked for help.");
    }
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)
```javascript
if (context.activity.type === 'message') {
    // check the message text before calling context.sendActivity
    var messagetext = context.activity.text.trim().toLowerCase();
    if(messagetext === "help"){
        await context.sendActivity("you said help");
    }
}
```

---

![Le bot détecte le mot help en français (aide)](./media/how-to-bot-translate/bot-detects-help-french.png)



## <a name="translate-replies-back-to-the-users-language"></a>Retraduire les réponses dans la langue de l’utilisateur

Vous pouvez également retraduire des réponses dans la langue de l’utilisateur, en définissant le dernier paramètre du constructeur sur `true`.

# <a name="ctabcs"></a>[C#](#tab/cs)
Dans le fichier `Startup.cs`, mettez à jour la ligne ci-après de la méthode `ConfigureServices`.
```cs
// Use language recognition to detect the user's language from their message, instead of providing helper callbacks.
// Last parameter indicates that we'll translate replies back to the user's language
middleware.Add(new TranslationMiddleware(new string[] { "en" }, "TRANSLATION-SUBSCRIPTION-KEY", true));
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)
```javascript
// Use language recognition to detect the user's language from their message, instead of providing helper callbacks.
// Last parameter indicates that we'll translate replies back to the user's language
const languageTranslator = new LanguageTranslator({
    translatorKey: "xxxxxx",
    noTranslatePatterns: new Set(),
    nativeLanguages: ['en'],
    translateBackToUserLanguage: true
});
adapter.use(languageTranslator);
```

---

## <a name="run-the-bot-to-see-replies-in-the-users-language"></a>Exécuter le bot pour voir les réponses dans la langue de l’utilisateur

Exécutez le bot et tapez quelques messages dans d’autres langues. Vous verrez qu’il détecte la langue de l’utilisateur et traduit la réponse.

![Le bot détecte la langue et traduit la réponse](./media/how-to-bot-translate/bot-detects-language-translates-response.png)


## <a name="adding-logic-for-detecting-or-changing-the-user-language"></a>Ajout d’une logique de détection ou de modification de la langue de l’utilisateur

Au lieu de laisser le Kit de développement logiciel (SDK) Bot Builder détecter automatiquement la langue de l’utilisateur, vous pouvez fournir un rappel pour ajouter votre propre logique afin d’identifier la langue de l’utilisateur ou de déterminer si la langue de l’utilisateur a changé.


Dans l’exemple ci-après, le rappel `CheckUserChangedLanguage` recherche un message utilisateur spécifique afin de modifier la langue. 

# <a name="ctabcs"></a>[C#](#tab/cs)
Dans `Startup.cs`, ajoutez un rappel à l’intergiciel de traduction.
```csharp
middleware.Add(new TranslationMiddleware(new string[] { "en" },
     "<YOUR MICROSOFT TRANSLATOR TEXT API KEY>", null,
     TranslatorLocaleHelper.GetActiveLanguage,
     TranslatorLocaleHelper.CheckUserChangedLanguage));
```
Ajoutez un fichier `TranslatorLocalHelper.cs`, puis ajoutez le code ci-après à la définition de classe TranslatorLocalHelper.
```csharp
    class CurrentUserState
    {
        public string Language { get; set; }
    }

    public static void SetLanguage(ITurnContext context, string language) =>
        context.GetConversationState<CurrentUserState>().Language = language;

    public static bool IsSupportedLanguage(string language) =>
        _supportedLanguages.Contains(language);

    public static async Task<bool> CheckUserChangedLanguage(ITurnContext context)
    {
        bool changeLang = false;
        
        // use a specific message from user to change language
        if (context.Activity.Type == ActivityTypes.Message)
        {
            var messageActivity = context.Activity.AsMessageActivity();
            if (messageActivity.Text.ToLower().StartsWith("set my language to"))
            {
                changeLang = true;
            }
            if (changeLang)
            {
                var newLang = messageActivity.Text.ToLower().Replace("set my language to", "").Trim();
                if (!string.IsNullOrWhiteSpace(newLang)
                        && IsSupportedLanguage(newLang))
                {
                    SetLanguage(context, newLang);
                    await context.SendActivity($@"Changing your language to {newLang}");
                }
                else
                {
                    await context.SendActivity($@"{newLang} is not a supported language.");
                }
                // return true to intercept message from further processesing
                return true;
            }
        }

        return false;
    }

    public static string GetActiveLanguage(ITurnContext context)
    {

        if (context.Activity.Type == ActivityTypes.Message
            && context.GetConversationState<CurrentUserState>() != null
            && context.GetConversationState<CurrentUserState>().Language != null)
        {
            return context.GetConversationState<CurrentUserState>().Language;
        }

        return "en";
    }

```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)
```javascript
// When the user inputs 'set my language to fr'
// The bot will automatically change all text to French
async function setUserLanguage(context) {
    let state = conversationState.get(context)
    if (context.activity.text.toLowerCase().startsWith('set my language to')) {
        state.language = context.activity.text.toLowerCase().replace('set my language to', '').trim();
        await context.sendActivity(`Setting your language to ${state.language}`);
        return Promise.resolve(true);
    } else {
        return Promise.resolve(false);
    }
}

// Add language translator middleware
const languageTranslator = new LanguageTranslator({
    translatorKey: "<YOUR MICROSOFT TRANSLATOR TEXT API KEY>",
    noTranslatePatterns: new Set(),
    nativeLanguages: ['en'],
    setUserLanguage: setUserLanguage,
    translateBackToUserLanguage: true
});
adapter.use(languageTranslator);

// Listen for incoming activity 
server.post('/api/messages', (req, res) => {
    // Route received activity to adapter for processing
    adapter.processActivity(req, res, async (context) => {
        if (context.activity.type != 'message') {
            await context.sendActivity(`[${context.activity.type} event detected]`);
        } else {
            await context.sendActivity(`You said: ${context.activity.text}`)
        }
    });
});

```

---

## <a name="combining-luis-or-qna-with-translation"></a>Combinaison de LUIS ou QnA avec une traduction

Si vous combinez une traduction avec d’autres services dans votre bot, par exemple LUIS ou QnA, commencez par ajouter l’intergiciel de traduction afin de traduire les messages avant de les transmettre à un autre intergiciel qui attend la langue native du bot.

# <a name="ctabcs"></a>[C#](#tab/cs)
```cs
public void ConfigureServices(IServiceCollection services)
{
    services.AddSingleton(_ => Configuration);
    services.AddBot<AiBot>(options =>
    {
        options.CredentialProvider = new ConfigurationCredentialProvider(Configuration);
        var middleware = options.Middleware;

        // Add translation middleware
        // The first parameter is a list of languages the bot recognizes. Input in these language won't be translated.
        middleware.Add(new TranslationMiddleware(new string[] { "en" }, "<YOUR MICROSOFT TRANSLATOR TEXT API KEY>", false));

        // Add LUIS middleware after translation middleware to pass translated messages to LUIS
        middleware.Add(
            new LuisRecognizerMiddleware(
                new LuisModel("luisAppId", "subscriptionId", new Uri("luisModelBaseUrl"))));
    });
}
```

Dans le code de votre bot, les résultats LUIS reposent sur l’entrée qui a déjà été traduite dans la langue native du bot. Essayez de modifier le code du bot pour vérifier les résultats d’une application LUIS :

```cs
public async Task OnTurn(ITurnContext context)
{
    switch (context.Activity.Type)
    {
        case ActivityTypes.Message:
            // check LUIS results
            var luisResult = context.Services.Get<RecognizerResult>(LuisRecognizerMiddleware.LuisRecognizerResultKey);
            (string key, double score) topItem = luisResult.GetTopScoringIntent();
            await context.SendActivity($"The top intent was: '{topItem.key}', with score {topItem.score}");

            await context.SendActivity($"You sent '{context.Activity.Text}'");

        case ActivityTypes.ConversationUpdate:
            foreach (var newMember in context.Activity.MembersAdded)
            {
                if (newMember.Id != context.Activity.Recipient.Id)
                {
                    await context.SendActivity("Hello and welcome to the echo bot.");
                }
            }
            break;
        }
    }  
}          
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)
```javascript
// Add language translator middleware
const languageTranslator = new LanguageTranslator({
    translatorKey: "xxxxxx",
    noTranslatePatterns: new Set(),
    nativeLanguages: ['en'],
    translateBackToUserLanguage: false
});
adapter.use(languageTranslator);

// Add Luis recognizer middleware
const luisRecognizer = new LuisRecognizer({
    appId: "xxxxxx",
    subscriptionKey: "xxxxxxx"
});
adapter.use(luisRecognizer);


// Listen for incoming requests 
server.post('/api/messages', (req, res) => {
    // Route received request to adapter for processing
    adapter.processActivity(req, res, async (context) => {
        if (context.activity.type != 'message') {
            await context.sendActivity(`[${context.activity.type} event detected]`);
        } else {
            let results = luisRecognizer.get(context);
            for (const intent in results.intents) {
                await context.sendActivity(`intent: ${intent.toString()} : ${results.intents[intent.toString()]}`)
            }
        }
    });
});
```

---

## <a name="bypass-translation-for-specified-patterns"></a>Ignorer la traduction pour les modèles spécifiés
Il existe peut-être des termes que votre bot ne devrait pas traduire, par exemple des noms propres. Vous pouvez fournir des expressions régulières pour indiquer les modèles qui ne doivent pas être traduits. Par exemple, si l’utilisateur tape « Mon nom est... » dans une langue non native pour votre bot, vous pouvez utiliser un modèle pour éviter que son nom soit traduit.

# <a name="ctabcs"></a>[C#](#tab/cs)
Dans Startup.cs
```cs
// Pattern representing input to not translate
Dictionary<string, List<string>> patterns = new Dictionary<string, List<string>>();
// single pattern for fr language, to avoid translating what follows "my name is"
patterns.Add("fr", new List<string> { "mon nom est (.+)" });

middleware.Add(new TranslationMiddleware(new string[] { "en" },
    "<YOUR API KEY>", patterns,
    TranslatorLocaleHelper.GetActiveLanguage,
    TranslatorLocaleHelper.CheckUserChangedLanguage));

```
<!-- TODO: ADD more explanation (both of these callbacks are run every time), fix image by debugging regex for l'etat -->

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)
```javascript
// Add language translator middleware
const languageTranslator = new LanguageTranslator({
    translatorKey: "<YOUR API KEY>",
    // single pattern for fr language, to avoid translating what follows "my name is"
    noTranslatePatterns: new Set("fr", "/mon nom est (.+)/"),
    nativeLanguages: ['en'],
    translateBackToUserLanguage: false,
    
});
adapter.use(languageTranslator);

```

---

![Le bot ignore la traduction d’un modèle](./media/how-to-bot-translate/bot-no-translate-name-fr.png)

## <a name="localize-dates"></a>Localiser des dates

Si vous devez localiser des dates, vous pouvez ajouter `LocaleConverterMiddleware`. Par exemple, si vous savez que votre bot attend des dates au format `MM/DD/YYYY`, et que les utilisateurs d’autres pays peuvent entrer des dates au format `DD/MM/YYYY`, l’intergiciel de conversion des paramètres régionaux peut convertir automatiquement les dates au format attendu par votre bot.

> [!NOTE]
> L’intergiciel de conversion des paramètres régionaux est uniquement destiné à convertir des dates. Il n’a aucune connaissance des résultats de l’intergiciel de traduction. Si vous utilisez un intergiciel de traduction, soyez prudent lorsque vous l’associez à un convertisseur de paramètres régionaux. L’intergiciel de traduction traduira certaines dates au format texte et d’autres entrées texte, mais non les dates.

Par exemple, l’image ci-après illustre un bot qui renvoie l’entrée utilisateur après l’avoir traduite de l’anglais vers le français. Le bot utilise `TranslationMiddleware`, mais non `LocaleConverterMiddleware`.

![Bot traduisant des dates sans les convertir](./media/how-to-bot-translate/locale-date-before.png)

L’exemple ci-après illustre le même bot si `LocaleConverterMiddleware` est ajouté.

![Bot traduisant des dates sans les convertir](./media/how-to-bot-translate/locale-date-after.png)

Les convertisseurs de paramètres régionaux peuvent prendre en charge les paramètres régionaux en anglais, français, allemand et chinois. <!-- TODO: ADD DETAIL ABOUT SUPPORTED LOCALES -->

# <a name="ctabcs"></a>[C#](#tab/cs)
Dans Startup.cs
```cs
// Add locale converter middleware
middleware.Add(new LocaleConverterMiddleware(
    TranslatorLocaleHelper.GetActiveLocale,
    TranslatorLocaleHelper.CheckUserChangedLocale,
     "en-us", LocaleConverter.Converter));
```
Dans TranslatorLocaleHelper.cs
```cs
// TranslatorLocaleHelper.cs
public static async Task<bool> CheckUserChangedLocale(ITurnContext context)
{
    bool changeLocale = false;//logic implemented by developper to make a signal for language changing 
    //use a specific message from user to change language
    if (context.Activity.Type == ActivityTypes.Message)
    {
        var messageActivity = context.Activity.AsMessageActivity();
        if (messageActivity.Text.ToLower().StartsWith("set my locale to"))
        {
            changeLocale = true;
        }
        if (changeLocale)
        {
            var newLocale = messageActivity.Text.ToLower().Replace("set my locale to", "").Trim(); //extracted by the user using user state 
            if (!string.IsNullOrWhiteSpace(newLocale)
                    && IsSupportedLanguage(newLocale))
            {
                SetLocale(context, newLocale);
                await context.SendActivity($@"Changing your language to {newLocale}");
            }
            else
            {
                await context.SendActivity($@"{newLocale} is not a supported locale.");
            }
            //intercepts message
            return true;
        }
    }

    return false;
}
public static string GetActiveLocale(ITurnContext context)
{
    if (currentLocale != null)
    {
        //the user has specified a different locale so update the bot state
        if (context.GetConversationState<CurrentUserState>() != null
            && currentLocale != context.GetConversationState<CurrentUserState>().Locale)
        {
            SetLocale(context, currentLocale);
        }
    }
    if (context.Activity.Type == ActivityTypes.Message
        && context.GetConversationState<CurrentUserState>() != null && context.GetConversationState<CurrentUserState>().Locale != null)
    {
        return context.GetConversationState<CurrentUserState>().Locale;
    }

    return "en-us";
}
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

<!-- this snippet only works if the user doesn't actually try to change their locale.  Emailed Mostafa about the issue 
It should change the locale after you type in 'set my locale to....' -->

```javascript
// Delegates for getting and setting user locale
function getUserLocale(context) {
    const state = conversationState.get(context)
    if (state.locale == undefined) {
        return 'en-us';
    } else {
        return state.locale;
    }
}

// Function to change the user's locale.
async function setUserLocale(context) {
    let state = conversationState.get(context)
    if (context.activity.text.toLowerCase().startsWith('set my locale to')) {        
        state.locale = context.activity.text.toLowerCase().replace('set my locale to', '').trim();
        await context.sendActivity(`Setting your locale to ${state.locale}`);
        return Promise.resolve(true);
    } else {
        return Promise.resolve(false);
    }
}

// Add locale converter middleware
// Will convert input to fr-fr
const localeConverter = new LocaleConverter({
    toLocale: 'fr-fr',
    setUserLocale: setUserLocale,
    getUserLocale: getUserLocale
});
adapter.use(localeConverter);


// Listen for incoming requests 
server.post('/api/messages', (req, res) => {
    // Route received request to adapter for processing
    adapter.processActivity(req, res, async (context) => {
        if (context.activity.type != 'message') {
            await context.sendActivity(`[${context.activity.type} event detected]`);
        } else {
            await context.sendActivity(`The message that you sent was:  ${context.activity.text}`)
        }
    });
});

```

---
