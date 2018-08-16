---
title: Créer des bots avec des modèles Bot Builder
description: Les outils Bot Builder vous permettent de gérer vos ressources de bot directement à partir de la ligne de commande
keywords: modèles bot builder, node.js, python, java, .net, ludown, yeoman
author: matthewshim-ms
ms.author: v-shimma
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 04/25/2018
ms.openlocfilehash: 7cf05b3396099f1c65fce7abbceb143a3ad43e9a
ms.sourcegitcommit: 67445b42796d90661afc643c6bb6533e9a662cbc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/06/2018
ms.locfileid: "39574585"
---
# <a name="create-bots-with-botbuilder-templates"></a>Créer des bots avec des modèles Bot Builder

Vous disposez désormais de modèles pour créer des bots dans chacune des plateformes du SDK Bot Builder : 

- Node.js
- Python
- Java
- .NET

## <a name="prerequisites"></a>Prérequis

Les modèles Node.js, Java et Python créent tous un echo bot simple et sont tous accessibles via [Yeoman](http://yeoman.io/).

- [Node.js](https://nodejs.org/en/) version 8.5 ou ultérieure
- [Yeoman](http://yeoman.io/)

Si vous n’avez pas déjà installé Yeoman, vous devez l’installer globalement.

```shell
npm install -g yo
```

## <a name="nodejs-python-java-templates"></a>Modèles Node.js, Python, Java
À partir de la ligne de commande, accédez à un nouveau dossier de votre choix. Deux versions des modèles Node.js Bot Builder sont disponibles, qui ciblent les versions **v3** et **v4** du SDK respectivement. Les modèles Python et Java sont uniquement disponibles dans la version v4. 

Pour installer le générateur de modèles **Node.js Bot Builder v3** :

```shell
npm install generator-botbuilder
```

Pour installer le générateur de modèles **Node.js Bot Builder v4** :
```shell
npm install generator-botbuilder@preview
```

Les modèles Python et Java sont **uniquement disponibles dans la version v4**. 

Pour **Python** :
```shell
npm install generator-botbuilder-python
```

Pour **Java** :
```shell
npm install generator-botbuilder-java
```

Une fois un générateur installé, vous pouvez simplement exécuter la commande **yo** dans votre interface CLI pour afficher la liste des générateurs disponibles dans Yeoman.

![Interface de Yeoman](media/botbuilder-templates/yeoman-generator-botbuilder.png)

Basculez vers un répertoire de travail de votre choix, puis sélectionnez le générateur à utiliser. Vous serez invité à indiquer diverses options pour créer votre bot, telles que le nom et une description. Une fois toutes les invites terminées, votre modèle d’echo bot est créé dans le même dossier de travail.

![Modèle Yeoman Node.js](media/botbuilder-templates/new-template-node.png)


## <a name="net"></a>.NET

Deux modèles sont disponibles pour .NET, qui ciblent les versions **v3** et **v4** du SDK respectivement. Les deux sont disponibles en tant que packages [VSIX](https://docs.microsoft.com/en-us/visualstudio/extensibility/anatomy-of-a-vsix-package) ; pour les télécharger, cliquez sur un des liens suivants sur [Visual Studio Marketplace](https://marketplace.visualstudio.com/).

- [Modèle Bot Builder V3](https://marketplace.visualstudio.com/items?itemName=BotBuilder.BotBuilderV3)
- [Modèle Bot Builder V4](https://aka.ms/Ylcwxk)

#### <a name="prerequisites"></a>Prérequis

- [Visual Studio 2015 ou version ultérieure](https://www.visualstudio.com/downloads/)
- [Compte Azure](https://azure.microsoft.com/en-us/free/)

### <a name="install-the-templates"></a>Installer les modèles

À partir du répertoire enregistré, ouvrez simplement le package VSIX ; le modèle de générateur de bot est alors installé pour Visual Studio et sera disponible à sa prochaine ouverture.

![Package VSIX](media/botbuilder-templates/botbuilder-vsix-template.png)

Pour créer un projet de bot à l’aide du modèle, ouvrez simplement Visual Studio et sélectionnez **Fichier** > **Nouveau** > **Projet** et à partir de Visual C# sélectionnez **Bot Framework** > Application d’echo bot simple. Cette opération crée un projet d’echo bot localement que vous pouvez modifier comme vous le souhaitez. 

![Modèle de bot .NET](media/botbuilder-templates/new-template-dotnet.png)

## <a name="store-your-bot-information-with-msbot"></a>Stocker vos informations de bot avec MSBot

Le nouvel outil [MSBot](https://github.com/Microsoft/botbuilder-tools/tree/master/MSBot) vous permet de créer un fichier **.bot** qui stocke dans un seul emplacement les métadonnées relatives aux différents services que votre bot utilise. Ce fichier permet également à votre bot de se connecter à ces services à partir de l’interface CLI. L’outil est disponible en tant que module npm ; pour l’installer, exécutez la commande suivante :

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

| de diffusion en continu | Description |
| ------ | ----------- |
| azure  |Connecter votre bot à une inscription Azure Bot Service|
|localhost| Connecter votre bot à un point de terminaison localhost|
|luis     | Connecter votre bot à une application LUIS |
| qna     |Connecter votre bot à une base de connaissances de questions/réponses|
|help [commande]  |Afficher l’aide de [commande]|

### <a name="connect-your-bot-to-abs-with-the-bot-file"></a>Connecter votre bot à ABS avec le fichier .bot

Une fois l’outil MSBot installé, vous pouvez facilement connecter votre bot à un groupe de ressources existant dans Azure Bot Service en exécutant la commande az bot **show**. 

```azurecli
az bot show -n <botname> -g <resourcegroup> --msbot | msbot connect azure --stdin
```

Cette commande récupère le point de terminaison actuel, ainsi que l’ID et le mot de passe d’application du groupe de ressources cible et met à jour les informations en conséquence dans votre fichier .bot. 


## <a name="manage-update-or-create-luis-and-qna-services-with--new-botbuilder-tools"></a>Gérer, mettre à jour ou créer des services LUIS et de questions/réponses avec les nouveaux outils Bot Builder

Les [outils Bot Builder](https://github.com/microsoft/botbuilder-tools) sont un nouvel ensemble d’outils vous permettant de gérer vos ressources de bot et d’interagir avec elles directement à partir de la ligne de commande. 

>[!TIP]
> Chaque outil Bot Builder inclut une commande d’aide globale, que vous pouvez exécuter à partir de la ligne de commande en entrant **-h** ou **--help**. Cette commande est disponible à tout moment à partir de n’importe quelle action, affichant les options disponibles, ainsi que leurs descriptions. 

### <a name="ludown"></a>LUDown
[LUDown](https://github.com/Microsoft/botbuilder-tools/tree/master/Ludown) vous permet de décrire et de créer des composants linguistiques puissants pour les bots à l’aide de fichiers **.lu**. Le nouveau fichier .lu est un type de format Markdown qu’utilise l’outil LUDown et qui génère des fichiers .json propres au service cible. Vous pouvez utiliser des fichiers .lu pour créer une application [LUIS](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/luis-get-started-create-app) ou une base de connaissances de [questions/réponses](https://qnamaker.ai/Documentation/CreateKb), en utilisant des formats différents pour chacune d’elles. LUDown est disponible en tant que module npm et vous pouvez l’installer globalement sur votre machine :

```shell
npm install -g ludown
```
Vous pouvez utiliser l’outil LUDown pour créer des modèles .json pour LUIS et le service de questions/réponses.  


### <a name="creating-a-luis-application-with-ludown"></a>Création d’une application LUIS avec LUDown

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

### <a name="qna-pairs-with-ludown"></a>Paires de questions/réponses avec LUDown

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

### <a name="generating-json-models-with-ludown"></a>Génération de modèles .json avec LUDown

Après avoir défini les composants linguistiques LUIS ou de questions/réponses au format .lu, vous pouvez procéder à une publication dans un fichier LUIS .json, ou bien de questions/réponses .json ou .tsv. Quand il est exécuté, l’outil LUDown recherche tous les fichiers .lu dans le répertoire de travail à analyser. L’outil LUDown pouvant cibler LUIS ou le service de questions/réponses avec des fichiers .lu, nous devons simplement spécifier le service de langage pour lequel effectuer la génération, en utilisant la commande générale **ludown parse <Service> -- in <luFile>**. 

Dans notre exemple de répertoire de travail, nous avons deux fichiers .lu à analyser : « 1.lu » pour créer le modèle LUIS et « qna1.lu » pour créer une base de connaissances de questions/réponses.

#### <a name="generate-luis-json-models"></a>Générer des modèles LUIS .json

Pour générer un modèle LUIS à l’aide de LUDown, dans votre répertoire de travail actif, entrez simplement la commande suivante :

```shell
ludown parse ToLuis --in <luFile> 
```

#### <a name="generate-qna-knowledge-base-json"></a>Générer le fichier .json de la base de connaissances de questions/réponses

De même, pour créer une base de connaissances de questions/réponses, vous devez uniquement changer la cible d’analyse. 

```shell
ludown parse ToQna --in <luFile> 
```

Les fichiers JSON obtenus peuvent être utilisés par LUIS et le service de questions/réponses via leurs portails respectifs, ou via les nouveaux outils CLI. 

## <a name="connect-to-luis-an-qna-maker-services-from-the-cli"></a>Se connecter aux services LUIS et QnA Maker à partir de l’interface CLI

### <a name="connect-to-luis-from-the-cli"></a>Se connecter à LUIS à partir de l’interface CLI 

Le nouveau jeu d’outils comprend une [extension LUIS](https://github.com/Microsoft/botbuilder-tools/tree/master/LUIS) qui vous permet de gérer vos ressources LUIS. Il est disponible en tant que module npm que vous pouvez télécharger :

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

![LUIS init](media/bot-builder-tools/luis-init.png) 


Une fois ce fichier généré, votre application est en mesure d’utiliser le fichier .json LUIS (généré à partir de LUDown) à l’aide de la commande suivante à partir de l’interface CLI.

```shell
luis import application --in luis-app.json | msbot connect luis --stdin
```

### <a name="connect-to-qna-from-the-cli"></a>Se connecter au service de questions/réponses à partir de l’interface CLI

Le nouveau jeu d’outils comprend une [extension de questions/réponses](https://github.com/Microsoft/botbuilder-tools/tree/master/QnAMaker) qui vous permet de gérer vos ressources LUIS. Elle est disponible en tant que module npm que vous pouvez télécharger.

```shell
npm install -g qnamaker
```
Avec l’outil QnA Maker, vous pouvez créer, mettre à jour, publier, supprimer et former votre base de connaissances. Pour commencer, vous devez créer un fichier **.qnamakerrc** nécessaire pour activer le point de terminaison sur votre service. Vous pouvez facilement créer ce fichier en exécutant **qnamaker init** et en suivant les invites et provisionner l’ID de la base de connaissances QnA Maker. 

```shell
qnamaker init 
```
![QnaMaker init](media/bot-builder-tools/qnamaker-init.png)

Une fois votre fichier .qnamakerrc généré, vous pouvez vous connecter à votre base de connaissances de questions/réponses pour utiliser le fichier.json/.tsv de la base de connaissances (généré à partir de LUDown) à l’aide de la commande suivante.

```shell
qnamaker create --in qnaKB.json --msbot | msbot connect qna --stdin
```

## <a name="references"></a>Références
- [Code source des outils Bot Builder](https://github.com/Microsoft/botbuilder-tools)
- [MSBot](https://github.com/Microsoft/botbuilder-tools/tree/master/MSBot)
- [ChatDown](https://github.com/Microsoft/botbuilder-tools/tree/master/Chatdown)
- [LUDown](https://github.com/Microsoft/botbuilder-tools/tree/master/Ludown)
- [interface de ligne de commande Azure](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest)
