---
title: Localisation de la prise en charge | Microsoft Docs
description: Découvrez comment déterminer l’endroit où se trouve l’utilisateur et comment activer les fonctionnalités de localisation avec le kit SDK Bot Builder pour Node.js.
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 5ac9fabcb0c6626e1b0133b7718b135a88d4c846
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2018
ms.locfileid: "49998061"
---
# <a name="support-localization"></a>Localisation de la prise en charge

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

Bot Builder comprend un système de localisation évolué, capable de créer des bots pouvant communiquer dans plusieurs langues avec l’utilisateur. Toutes les invites de votre bot peuvent être localisées à l’aide de fichiers JSON stockés dans la structure de répertoire de votre bot. Si vous utilisez un système tel que LUIS pour effectuer le traitement en langage naturel, vous pouvez configurer votre [LuisRecognizer][LUISRecognizer] au moyen d’un modèle distinct pour chaque langue prise en charge par votre bot, et le kit SDK sélectionnera automatiquement le modèle qui correspond aux paramètres régionaux par défaut de l’utilisateur.

## <a name="determine-the-locale-by-prompting-the-user"></a>Déterminer les paramètres régionaux par des invites à l’utilisateur
La première étape pour localiser votre bot consiste à ajouter la capacité d’identification de la langue par défaut utilisée par l’utilisateur. Le kit SDK fournit une méthode [session.preferredLocale()][preferredLocal] pour récupérer et enregistrer cette préférence en fonction de l’utilisateur. L’exemple suivant est un dialogue pour inviter l’utilisateur à indiquer la langue de son choix, puis à enregistrer son choix.

``` javascript
bot.dialog('/localePicker', [
    function (session) {
        // Prompt the user to select their preferred locale
        builder.Prompts.choice(session, "What's your preferred language?", 'English|Español|Italiano');
    },
    function (session, results) {
        // Update preferred locale
        var locale;
        switch (results.response.entity) {
            case 'English':
                locale = 'en';
                break;
            case 'Español':
                locale = 'es';
                break;
            case 'Italiano':
                locale = 'it';
                break;
        }
        session.preferredLocale(locale, function (err) {
            if (!err) {
                // Locale files loaded
                session.endDialog(`Your preferred language is now ${results.response.entity}`);
            } else {
                // Problem loading the selected locale
                session.error(err);
            }
        });
    }
]);
```

## <a name="determine-the-locale-by-using-analytics"></a>Déterminer les paramètres régionaux à l’aide de l’analytique
Un autre moyen de déterminer les paramètres régionaux de l’utilisateur consiste à utiliser un service tel que l’[API Analyse de texte](/azure/cognitive-services/cognitive-services-text-analytics-quick-start) qui détecte automatiquement la langue de l’utilisateur en fonction du texte du message qu’il a envoyé.

L’extrait de code ci-dessous illustre la manière dont vous pouvez intégrer ce service à votre propre bot.
``` javascript
var request = require('request');

bot.use({
    receive: function (event, next) {
        if (event.text && !event.textLocale) {
            var options = {
                method: 'POST',
                url: 'https://westus.api.cognitive.microsoft.com/text/analytics/v2.0/languages?numberOfLanguagesToDetect=1',
                body: { documents: [{ id: 'message', text: event.text }]},
                json: true,
                headers: {
                    'Ocp-Apim-Subscription-Key': '<YOUR API KEY>'
                }
            };
            request(options, function (error, response, body) {
                if (!error && body) {
                    if (body.documents && body.documents.length > 0) {
                        var languages = body.documents[0].detectedLanguages;
                        if (languages && languages.length > 0) {
                            event.textLocale = languages[0].iso6391Name;
                        }
                    }
                }
                next();
            });
        } else {
            next();
        }
    }
});
```

Une fois l’extrait de code ci-dessus ajouté à votre bot, l’appel à [session.preferredLocale()][preferredLocal] retourne automatiquement la langue détectée. L’ordre de recherche pour `preferredLocale()` est le suivant :
1. Les paramètres régionaux enregistrés en appelant `session.preferredLocale()`. Cette valeur est stockée dans `session.userData['BotBuilder.Data.PreferredLocale']`.
2. Les paramètres régionaux détectés, assignés à `session.message.textLocale`.
3. Les paramètres régionaux par défaut configurés pour le bot, par exemple : anglais (« en »).

Vous pouvez configurer les paramètres régionaux par défaut du bot à l’aide de son constructeur :

```javascript
var bot = new builder.UniversalBot(connector, {
    localizerSettings: { 
        defaultLocale: "es" 
    }
});
```

## <a name="localize-prompts"></a>Localiser les invites
Le système de localisation par défaut du kit SDK Bot Builder est basé sur des fichiers, et permet à un bot de prendre en charge plusieurs langues au moyen de fichiers JSON stockés sur le disque. Par défaut, le système de localisation recherche les invites du bot dans le fichier **./locale/<IETF TAG>/index.json**, où <IETF TAG> est une [balise de langue IETF][IEFT] valide représentant les paramètres régionaux par défaut pour lesquels rechercher des invites. 

La capture d’écran suivante montre la structure de répertoire d’un bot prenant en charge trois langues : l’anglais, l’italien et l’espagnol.

![Structure de répertoire pour les trois paramètres régionaux](../media/locale-dir.png)

La structure du fichier est un simple mappage JSON d’ID de message à des chaînes de texte localisé. Si la valeur est un tableau plutôt qu’une chaîne, une invite à partir du tableau est choisi au hasard lorsque cette valeur est récupérée au moyen de [session.localizer.gettext()][GetText]. 

Le bot récupère automatiquement la version localisée d’un message si vous transmettez l’ID de message dans un appel à [session.send()](http://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session#send) au lieu du texte propre à la langue :

```javascript
var bot = new builder.UniversalBot(connector, [
    function (session) {
        session.send("greeting");
        session.send("instructions");
        session.beginDialog('/localePicker');
    },
    function (session) {
        builder.Prompts.text(session, "text_prompt");
    }
]);
```

En interne, le kit SDK appelle [`session.preferredLocale()`][preferredLocale] pour obtenir les paramètres régionaux par défaut de l’utilisateur, afin de les utiliser ensuite dans un appel à [`session.localizer.gettext()`][GetText] pour mapper l’ID de message à sa chaîne de texte localisé.  Parfois, vous pouvez être amené à appeler manuellement le localisateur. Par exemple, les valeurs enum transmises à [`Prompts.choice()`][promptsChoice] ne sont jamais automatiquement localisées, vous pouvez donc avoir besoin de récupérer manuellement une liste localisée avant l’appel à l’invite :

```javascript
var options = session.localizer.gettext(session.preferredLocale(), "choice_options");
builder.Prompts.choice(session, "choice_prompt", options);
```

Le localisateur par défaut recherche un ID de message dans plusieurs fichiers, et s’il ne peut pas trouver d’ID (ou si aucun fichier de localisation n’a été fourni), il retourne simplement le texte de l’ID, rendant l’utilisation des fichiers de localisation transparente et facultative.  Les fichiers sont recherchés dans l’ordre suivant :

1. Le fichier **index.json**, sous les paramètres régionaux retournés par [`session.preferredLocale()`][preferredLocale], est recherché.
2. Si les paramètres régionaux comportait une sous-balise facultative, comme **en-US**, la balise racine de **en** est alors recherchée.
3. Les paramètres régionaux par défaut configurés du bot sont recherchés.

## <a name="use-namespaces-to-customize-and-localize-prompts"></a>Utiliser des espaces de noms pour personnaliser et localiser les invites
Le localisateur par défaut prend en charge l’attribution d’espaces de noms aux invites pour éviter les collisions entre les ID de message.  Votre bot peut écraser des invites à espace de noms pour personnaliser ou reformuler les invites provenant d’un autre espace de noms.  Vous pouvez tirer parti de cette fonctionnalité pour personnaliser les messages intégrés du kit SDK, ce qui vous permet soit d’ajouter une prise en charge de langues supplémentaires, soit de reformuler simplement les messages actuels du kit SDK.  Par exemple, vous pouvez modifier le message d’erreur par défaut du kit SDK en ajoutant juste un fichier nommé **BotBuilder.json** au répertoire des paramètres régionaux de votre bot, puis en ajoutant une entrée pour l’ID du message `default_error` :

![BotBuilder.json pour l’attribution d’espaces de noms au niveau des paramètres régionaux](../media/locale-namespacing.png)


## <a name="additional-resources"></a>Ressources supplémentaires

Pour savoir comment localiser un module de reconnaissance, consultez [Reconnaissance de l’intention](bot-builder-nodejs-recognize-intent-messages.md).


[LUIS]: https://www.luis.ai/
[IMessage]: http://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.imessage
[IntentRecognizerSetOptions]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.iintentrecognizersetoptions.html
[LUISRecognizer]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.luisrecognizer
[LUISSample]: https://aka.ms/v3-js-luisSample
[DisambiguationSample]: https://aka.ms/v3-js-onDisambiguateRoute
[preferredLocal]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session#preferredlocale
[preferredLocale]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session#preferredlocale
[promptsChoice]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.__global.iprompts.html#choice
[GetText]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.ilocalizer.html#gettext
[IEFT]: https://en.wikipedia.org/wiki/IETF_language_tag

