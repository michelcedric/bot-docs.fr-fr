---
title: Gérer les bots à l’aide des outils CLI
description: Les outils Bot Builder vous permettent de gérer vos ressources de bot directement à partir de la ligne de commande.
keywords: botbuilder templates, ludown, qna, luis, msbot, manage, cli, .bot, bot
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: tools
ms.date: 11/13/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 5ffaf9a946e1a540b82819b7f745200f47384819
ms.sourcegitcommit: 8b7bdbcbb01054f6aeb80d4a65b29177b30e1c20
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/14/2018
ms.locfileid: "51645659"
---
# <a name="manage-bots-using-cli-tools"></a>Gérer les bots à l’aide des outils CLI

Les outils Bot Builder couvrent tout le workflow de développement de bot : planification, compilation, test, publication, connexion et évaluation. Nous allons voir comment ces outils peuvent vous aider à chaque phase du cycle de développement.

## <a name="plan"></a>Planification

### <a name="create-mock-conversations-using-chatdown"></a>Créer des conversations fictives à l’aide de Chatdown

Chatdown est un générateur de transcription qui utilise un fichier .chat pour générer des transcriptions fictives. Les fichiers de transcription fictifs générés sont envoyés vers stdout.

Un bon bot, comme toute application ou tout site web réussi, nécessite de clarifier les scénarios pris en charge. La création de conversations fictives entre l’utilisateur et le bot est utile pour plusieurs raisons :

- Elle permet de délimiter les scénarios pris en charge par le bot.
- Les décideurs de l’entreprise peuvent effectuer des révisions et laisser des commentaires.
- La définition d’un « bon chemin » (ainsi que d’autres chemins d’accès) via les flux de conversation entre un utilisateur et un format de fichier .chat de bot vous permet de créer des conversations fictives entre un utilisateur et un bot. L’outil d’interface de ligne de commande Chatdown convertit les fichiers .chat en transcriptions de conversation (fichiers .transcript) qui peuvent être affichées dans [l’émulateur Bot Framework V4](https://github.com/microsoft/botframework-emulator).

Voici un exemple de fichier `.chat` :

```markdown
user=Joe
bot=LulaBot

bot: Hi!
user: yo!
bot: [Typing][Delay=3000]
Greetings!
What would you like to do?
* update - You can update your account
* List - You can list your data
* help - you can get help

user: I need the bot framework logo.

bot:
Here you go.
[Attachment=bot-framework.png]
[Attachment=http://yahoo.com/bot-framework.png]
[AttachmentLayout=carousel]

user: thanks
bot:
Here's a form for you
[Attachment=card.json adaptivecard]

```

### <a name="create-a-transcript-file-from-chat-file"></a>Créer un fichier de transcription à partir d’un fichier .chat
Une commande Chatdown se présente comme suit :

```bash
chatdown sample.chat > sample.transcript
```

L’entrée est `sample.chat`, et la sortie est `sample.transcript`. Pour plus d’informations, consultez la rubrique sur [l’interface de ligne de commande Chatdown][chatdown].

## <a name="build"></a>Créer
### <a name="create-a-luis-application-with-ludown"></a>Création une application LUIS avec LUDown
Vous pouvez utiliser l’outil LUDown pour créer des modèles .json pour LUIS et le service de questions/réponses.  
Vous pouvez définir des [intentions](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/add-intents) et des [entités](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/add-entities) pour une application LUIS comme vous le feriez à partir du portail LUIS.

« #\<nom_intention\> » décrit une nouvelle section de définition d’intention. Chaque ligne par la suite répertorie les [énoncés](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/add-example-utterances) qui décrivent cette intention.

Par exemple, vous pouvez créer plusieurs intentions LUIS dans un fichier .lu unique comme suit : 

```LUDown
# Greeting
- Hi
- Hello
- Good morning
- Good evening

# Help
- help
- I need help
- please help
```

### <a name="create-qna-pairs-with-ludown"></a>Créer des paires de questions/réponses avec LUDown

Le format de fichier .lu prend en charge également les paires de questions/réponses à l’aide de la notation suivante : 

```LUDown
> comment
### ? question ?
  ```markdown
    answer
  ```

L’outil LUDown sépare automatiquement les questions/réponses dans un fichier JSON qnamaker que vous pouvez ensuite utiliser pour créer votre base de connaissances [QnaMaker.ai](http://qnamaker.ai).

```LUDown
### ? How do I change the default message for QnA Maker?
  ```markdown
  You can change the default message if you use the QnAMakerDialog. 
  See [this link](https://docs.botframework.com/en-us/azure-bot-service/templates/qnamaker/#navtitle) for details. 
  ```

Vous pouvez également ajouter plusieurs questions à la même réponse en ajoutant simplement de nouvelles lignes de variantes de questions pour une réponse unique.

```LUDown
### ? What is your name?
- What should I call you?
  ```markdown
    I'm the echoBot! Nice to meet you.
  ```

### <a name="generate-json-models-with-ludown"></a>Générer des modèles .json avec LUDown

Après avoir défini les composants linguistiques LUIS ou de questions/réponses au format .lu, vous pouvez procéder à une publication dans un fichier LUIS .json, ou bien de questions/réponses .json ou .tsv. Quand il est exécuté, l’outil LUDown recherche tous les fichiers .lu dans le répertoire de travail à analyser. L’outil LUDown pouvant cibler LUIS ou le service de questions/réponses avec des fichiers .lu, nous devons simplement spécifier le service de langage pour lequel effectuer la génération, en utilisant la commande générale **ludown parse <Service> -- in <luFile>**. 

Dans notre exemple de répertoire de travail, nous avons deux fichiers .lu à analyser : « 1.lu » pour créer le modèle LUIS et « qna1.lu » pour créer une base de connaissances de questions/réponses.

#### <a name="generate-luis-json-models"></a>Générer des modèles LUIS .json

Pour générer un modèle LUIS à l’aide de LUDown, dans votre répertoire de travail actif, entrez simplement la commande suivante :

```shell
ludown parse ToLuis --in <luFile>
```

#### <a name="generate-qna-knowledge-base"></a>Générer une base de connaissances de questions/réponses

De même, pour créer une base de connaissances de questions/réponses, vous devez uniquement changer la cible d’analyse.

```shell
ludown parse ToQna --in <luFile> 
```

Les fichiers JSON obtenus peuvent être utilisés par LUIS et le service de questions/réponses via leurs portails respectifs, ou via les nouveaux outils CLI. Pour en savoir plus, consultez le référentiel GitHub sur [l’interface de ligne de commande LUdown][ludown].

### <a name="track-service-references-using-bot-file"></a>Suivre les références de service à l’aide d’un fichier .bot

Le nouvel outil [MSBot][msbotCli] vous permet de créer un fichier  **.bot** qui stocke, dans un même emplacement, les métadonnées relatives aux différents services utilisés par votre bot. Ce fichier permet également à votre bot de se connecter à ces services à partir de l’interface CLI. L’outil est disponible en tant que module npm ; pour l’installer, exécutez la commande suivante :

```shell
npm install -g msbot
```

Pour créer un fichier de bot, dans votre interface CLI, entrez **msbot init**, suivi du nom de votre bot et du point de terminaison de l’URL cible, par exemple :

```shell
msbot init --name TestBot --endpoint http://localhost:9499/api/messages
```
Pour connecter votre bot à un service, dans votre interface CLI, entrez **msbot connect** suivi du service approprié :

```shell
msbot connect [Service]
```

Pour obtenir la liste des services pris en charge, reportez-vous au fichier [readme][msbotCli].

### <a name="create-and-manage-luis-applications-using-luis-cli"></a>Créer et gérer les applications LUIS à l’aide de l’interface de ligne de commande LUIS

Le nouveau jeu d’outils comprend une [extension LUIS][luisCli] qui vous permet de gérer vos ressources LUIS. Il est disponible en tant que module npm que vous pouvez télécharger :

```shell
npm install -g luis-apis
```
L’utilisation de base de la commande pour l’outil LUIS à partir de l’interface CLI est la suivante :

```shell
luis <action> <resource> <args...>
```
Pour connecter votre bot à LUIS, vous devez créer un fichier **.luisrc**. Il s’agit d’un fichier de configuration qui provisionne l’ID d’application et le mot de passe LUIS sur le point de terminaison de service quand votre application effectue des appels sortants. Vous pouvez créer ce fichier en exécutant **luis init** comme suit :

```shell
luis init
```
Dans le terminal, vous devrez entrer votre clé de création LUIS, région et ID d’application pour que l’outil puisse générer le fichier.  

Une fois ce fichier généré, votre application est en mesure d’utiliser le fichier .json LUIS (généré à partir de LUDown) à l’aide de la commande suivante à partir de l’interface CLI.

```shell
luis import application --in luis-app.json | msbot connect luis --stdin
```
Pour en savoir plus, consultez le référentiel GitHub sur [l’interface de ligne de commande LUIS][luisCli].

### <a name="create-qna-maker-kb-using-qna-maker-cli"></a>Créer une base de connaissances QnA Maker à l’aide de l’interface de ligne de commande QnA Maker

Le nouveau jeu d’outils comprend une [extension de questions/réponses][qnaCli] qui vous permet de gérer vos ressources LUIS. Elle est disponible en tant que module npm que vous pouvez télécharger.

```shell
npm install -g qnamaker
```
Avec l’outil QnA Maker, vous pouvez créer, mettre à jour, publier, supprimer et former votre base de connaissances. Vous pouvez utiliser des fichiers générés via la commande [ludown parse toqna](#generate-qna-knowledge-base) pour créer/remplacer une base de connaissances.

```shell
qnamaker create --in qnaKB.json --msbot | msbot connect qna --stdin
```

Pour en savoir plus, consultez le référentiel GitHub sur [l’interface de ligne de commande QnA Maker][qnaCli].

### <a name="create-dispatch-model-using-dispatch-cli"></a>Créer un modèle de distribution à l’aide d’interface de ligne de commande Dispatch

Dispatch est un outil qui permet de créer et d’évaluer des modèles LUIS servant à distribuer une intention dans plusieurs modules de bot, tels que les modèles LUIS, les bases de connaissances de questions/réponses et autres (ajouté à Dispatch comme type de fichier).

Utilisez le modèle Dispatch dans les cas suivants :

- Votre bot est constitué de plusieurs modules, et vous avez besoin d’aide pour le routage des énoncés de l’utilisateur vers ces modules et l’évaluation de l’intégration du bot.
- Évaluer la qualité de la classification des intentions d’un modèle LUIS unique.
- Créer un modèle de classification de texte à partir de fichiers texte.

Une fois que vous avez assemblé votre fichier .bot avec les [applications LUIS][msbotCli-luis] et les [bases de connaissances QnA Maker][msbotCli-qna] desquelles dépend votre bot, vous pouvez générer simplement un modèle de répartition comme suit : 

```shell
dispatch create -b <YOUR-BOT-FILE> | msbot connect dispatch --stdin
```
Pour en savoir plus, consultez la rubrique sur [l’interface de ligne de commande Disptach][dispatchCli].

## <a name="test"></a>Test

L’[émulateur](bot-service-debug-emulator.md) Bot Framework est une application de bureau qui permet aux développeurs de bots de tester et déboguer leurs créations localement ou en les exécutant à distance via un tunnel.

## <a name="publish"></a>Publish

Vous pouvez utiliser l’interface de ligne de commande Azure pour créer, télécharger et publier votre bot dans Azure Bot Service. Installez l’extension de bot via : 
```shell
az extension add -n botservice
```

### <a name="create-azure-bot-service-bot"></a>Créer un bot Azure Bot Service

Remarque : vous devez utiliser la dernière version de `az cli`. Mettez-le à niveau, afin que az cli puisse fonctionner avec l’outil MSBot. 

Connectez-vous à votre compte Azure via : 
```shell
az login
```

Une fois que vous êtes connecté, vous pouvez créer un bot Azure Bot Service comme suit : 
```shell
az bot create [options]
```

Pour créer un bot et mettre à jour le fichier .bot avec la configuration de bot :  
```shell
az bot create [options] --msbot | msbot connect bot --stdin
```

Si vous avez déjà un bot :  
```shell
az bot show [options] --msbot | msbot connect bot --stdin
```

| Option                            | Description                                   |
|-----------------------------------|-----------------------------------------------|
| --kind -k [obligatoire]              | Type de bot.  Valeurs autorisées : function, registration, webapp.|
| --name -n [obligatoire]              | Nom de ressource du bot. |
| --appid                           | ID de compte msa à utiliser avec le bot.   |
| --location -l                     | Lieu. Vous pouvez configurer le lieu par défaut en utilisant `az configure --defaults location=<location>`.  Valeur par défaut : westus.|
| --msbot                           | Afficher la sortie au format JSON compatible avec un fichier .bot.  Valeurs autorisées : false, true.|
| --password -p                     | Mot de passe de compte msa du bot provenant du portail des développeurs. |
| --resource-group -g               | Nom du groupe de ressources. Vous pouvez configurer le groupe par défaut en utilisant `az configure --defaults group=<name>`.  Valeur par défaut : build2018. |
| --tags                            | Ensemble de balises à ajouter au bot. |


### <a name="configure-channels"></a>Configurer des canaux

Vous pouvez utiliser l’interface de ligne de commande Azure pour gérer les canaux de votre robot. 
```shell
>az bot -h
Group
   az bot: Manage Bot Services.
    Subgroups:
        directline: Manage Directline Channel on a Bot.
        email     : Manage Email Channel on a Bot.
        facebook  : Manage Facebook Channel on a Bot.
        kik       : Manage Kik Channel on a Bot.
        msteams   : Manage Msteams Channel on a Bot.
        skype     : Manage Skype Channel on a Bot.
        slack     : Manage Slack Channel on a Bot.
        sms       : Manage Sms Channel on a Bot.
        telegram  : Manage Telegram Channel on a Bot.
        webchat   : Manage Webchat Channel on a Bot.

    Commands:
        create    : Create a new Bot Service.
        delete    : Delete an existing Bot Service.
        download  : Download an existing Bot Service.
        publish   : Publish to an existing Bot Service.
        show      : Get an existing Bot Service.
        update    : Update an existing Bot Service.

```

## <a name="additional-information"></a>Informations supplémentaires
- [Outils Bot Builder sur GitHub][cliTools]

<!-- Footnote links -->

[cliTools]: https://aka.ms/botbuilder-tools-readme
[azureCli]: https://aka.ms/botbuilder-tools-azureCli
[msbotCli]: https://aka.ms/botbuilder-tools-msbot-readme
[msbotCli-luis]: https://aka.ms/botbuilder-tools-msbot-readme#connecting-to-luis-application
[msbotCli-qna]: https://aka.ms/botbuilder-tools-msbot-readme#connecting-to-qna-maker-knowledge-base
[chatdown]: https://aka.ms/botbuilder-tools-chatdown
[ludown]: https://aka.ms/botbuilder-ludown
[luisCli]: https://aka.ms/botbuilder-luis-cli
[qnaCli]: https://aka.ms/botbuilder-tools-qnaMaker
[dispatchCli]: https://aka.ms/botbuilder-tools-dispatch
