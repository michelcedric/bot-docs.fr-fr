---
title: Créer des bots avec l’interface de ligne de commande Azure
description: Bot Builder Tools vous permet de gérer vos ressources bot directement à partir de la ligne de commande
author: matthewshim-ms
ms.author: v-shimma
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 04/25/2018
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 1eb47e76ef1bd6765d5ba93c27b97a8d9e6143db
ms.sourcegitcommit: 2dc75701b169d822c9499e393439161bc87639d2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/24/2018
ms.locfileid: "42905303"
---
# <a name="create-bots-with-azure-cli"></a>Créer des bots avec l’interface de ligne de commande Azure

[!INCLUDE [pre-release-label](./includes/pre-release-label-v3.md)]

[Bot Builder Tools](https://github.com/microsoft/botbuilder-tools) est un nouvel ensemble d’outils qui vous permet de gérer et d’interagir avec vos ressources bot directement à partir de la ligne de commande. 

Ce tutoriel vous explique comment :

- Activer l’extension bot dans Azure CLI
- Créer un bot avec Azure CLI 
- Télécharger une copie locale pour le développement
- Utiliser le nouvel outil MSBot pour stocker toutes les informations sur les ressources du bot
- Gérer, créer ou mettre à jour des modèles LUIS et QnA avec LUDown
- Se connecter aux services LUIS et QnA Maker à partir de l’interface CLI
- Déployer votre bot sur Azure à partir de l’interface CLI

## <a name="prerequisites"></a>Prérequis

Pour activer ces outils à partir de la ligne de commande, Node.js doit être installé sur votre ordinateur : 

- [Node.js (version 8.5 ou ultérieure)](https://nodejs.org/en/)

## <a name="1-enable-azure-cli"></a>1. Activer l’interface de ligne de commande Azure

Vous pouvez désormais gérer des bots à l’aide de l’[interface de ligne de commande Azure](https://docs.microsoft.com/cli/azure/?view=azure-cli-latest), comme pour toute autre ressource Azure. Pour activer l’interface de ligne de commande Azure, procédez comme suit :

1. [Téléchargez](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest) l’interface de ligne de commande Azure, le cas échéant. 

2. Entrez la commande suivante pour télécharger le package de distribution Azure Bot Extension.

```azurecli
az extension add -n botservice
```

>[!TIP]
> Actuellement, Azure Bot Extension prend uniquement en charge les bots v3.
  
3. [Connectez-vous](https://docs.microsoft.com/en-us/cli/azure/authenticate-azure-cli?view=azure-cli-latest) à l’interface de ligne de commande Azure en exécutant la commande suivante.

```azurecli
az login
```
Vous serez invité à saisir un code d’authentification temporaire unique. Pour vous connecter, utilisez un navigateur web et rendez-vous sur la page Microsoft de [connexion de l’appareil](https://microsoft.com/devicelogin), puis collez le code fourni par l’interface CLI pour continuer. 

![Connexion de l’appareil MS](media/bot-builder-tools/ms-device-login.png)

Une fois connecté, vous verrez l’écran d’accueil de l’interface de ligne de commande Azure ainsi qu’une liste des options disponibles pour gérer votre compte et vos ressources.

![Interface de ligne de commande Azure Bot](media/bot-builder-tools/az-cli-bot.png)


 Pour obtenir la liste complète des commandes de l’interface de ligne de commande Azure, [cliquez ici](https://docs.microsoft.com/cli/azure/reference-index?view=azure-cli-latest).


## <a name="2-create-a-new-bot-from-azure-cli"></a>2. Créer un bot à partir de l’interface de ligne de commande Azure

À l’aide d’Azure CLI et de la nouvelle extension bot, vous pouvez créer des bots entièrement à partir de la ligne de commande. 

```azurecli
az bot [command]
```
|Commandes|  |
|----|----|
| create      |ajouter une ressource|
| delete     |cloner une ressource|
| télécharger   | télécharger le code source du bit|
| Publier   |publier sur un service de bot existant|
| show |afficher les ressources bot existantes.|
| update| mettre à jour un service bot existant|

Pour créer un bot à partir de l’interface CLI, vous devez sélectionner un [groupe de ressources](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-overview) existant ou en créer un. 

```azurecli
az bot create --resource-group "my-resource-group" --name "my-bot-name" --kind "my-resource-type" --description "description-of-my-bot"
```
Après une demande réussie, un message de confirmation apparaît.
```
obtained msa app id and password. Provisioning bot now.
```

> [!TIP]
> Si vous recevez un message d’erreur indiquant que le **groupe de ressources est introuvable**, vous devrez peut-être définir votre [abonnement](https://docs.microsoft.com/en-us/azure/architecture/cloud-adoption-guide/adoption-intro/subscription-explainer) dans l’interface de ligne de commande Azure. L’abonnement Azure doit correspondre à celui entré lorsque vous avez créé le groupe de ressources. Pour le définir, entrez les informations suivantes.
> ```azurecli
> az account set --subscription "your-subscription-name"
> ```
> Pour afficher une liste des abonnements de votre compte, entrez la commande suivante.
> ```azurecli
> az account list
> ```

Par défaut, un nouveau bot .NET sera créé. Vous pouvez spécifier la plateforme SDK en indiquant la langue à l’aide de l’argument **--lang**. Actuellement, le package d’extension bot prend en charge C# et les Kits SDK bot Node.js. Par exemple, pour **créer un bot Node.js** :

```azurecli
az bot create --resource-group "my-resource-group" --name "my-bot-name" --kind "my-resource-type" --description "description-of-my-bot" --lang Node 
```
Votre nouveau bot echo sera provisionné sur votre groupe de ressources dans Azure. Pour le tester, il suffit de sélectionner **Tester dans le chat web** dans l’en-tête de gestion du bot de la vue Web App Bot. 

![Bot echo Azure](media/bot-builder-tools/az-echo-bot.png) 

## <a name="3-download-the-bot-locally"></a>3. Télécharger le bot localement

Il existe deux façons de télécharger le code source du nouveau bot :
- Télécharger à partir du portail Azure.
- Télécharger à l’aide de la nouvelle interface Azure CLI.

Pour télécharger le code source de votre bot à partir du portail, sélectionnez votre ressource de bot, puis **Build** dans la gestion du bot. Plusieurs options sont disponibles pour gérer ou extraire localement le code de source de votre bot. 

![Télécharger le bot depuis le portail Azure](media/bot-builder-tools/az-portal-manage-code.png)

Pour télécharger la source de votre bot à l’aide de l’interface CLI, entrez la commande suivante. Votre bot sera téléchargé vers un sous-répertoire. Si le sous-répertoire n’existe pas, la commande le crée pour vous.

```azurecli
az bot download --name "my-bot-name" --resource-group "my-resource-group"
```
Mais vous pouvez également spécifier le répertoire où télécharger le bot.
Par exemple : 

![Commande de téléchargement à l’aide de l’interface CLI](media/bot-builder-tools/cli-bot-download-command.png)

![Télécharger le bot à l’aide de l’interface CLI](media/bot-builder-tools/cli-bot-download.png)

La commande ci-dessus vous permet de télécharger le code source de votre bot directement vers l’emplacement spécifié, et ainsi de développer votre bot localement.


## <a name="4-store-your-bot-information-with-msbot"></a>4. Stocker les informations de votre bot avec MSBot

Le nouvel outil [MSBot](https://github.com/Microsoft/botbuilder-tools/tree/master/MSBot) vous permet de créer un fichier **.bot** qui stocke, dans un même emplacement, les métadonnées relatives aux différents services utilisés par votre bot. Ce fichier permet également à votre bot de se connecter à ces services à partir de l’interface CLI. L’outil est disponible en tant que module npm ; pour l’installer, exécutez la commande suivante :

```shell
npm install -g msbot 
```

Pour créer un fichier de bot, dans votre interface CLI, entrez **msbot init**, suivi du nom de votre bot et du point de terminaison de l’URL cible, par exemple :

```shell
msbot init --name name-of-my-bot --endpoint http://localhost:bot-port-number/api/messages
```
Pour connecter votre bot à un service, dans votre interface CLI, entrez **msbot connect** suivi du service approprié :

```shell
msbot connect service-type
```

| Type de service | Description |
| ------ | ----------- |
| azure  |Connecter votre bot à une inscription Azure Bot Service|
|endpoint| Connecter votre bot à un point de terminaison comme localhost|
|luis     | Connecter votre bot à une application LUIS |
| qna     |Connecter votre bot à une base de connaissances de questions/réponses|
|help [commande]  |Afficher l’aide de [commande]|

### <a name="connect-your-bot-to-abs-with-the-bot-file"></a>Connecter votre bot à ABS avec le fichier .bot

Une fois l’outil MSBot installé, vous pouvez facilement connecter votre bot à un groupe de ressources existant dans Azure Bot Service en exécutant la commande az bot **show**. 

```azurecli
az bot show -n my-bot-name -g my-resource-group --msbot | msbot connect azure --stdin
```

Cette commande récupère le point de terminaison actuel, ainsi que l’ID et le mot de passe d’application du groupe de ressources cible et met à jour les informations en conséquence dans votre fichier .bot. 


## <a name="5-manage-update-or-create-luis-and-qna-services-with--new-botbuilder-tools"></a>5. Gérer, mettre à jour ou créer des services LUIS et de questions/réponses avec les nouveaux outils Bot Builder

Les [outils Bot Builder](https://github.com/microsoft/botbuilder-tools) sont un nouvel ensemble d’outils vous permettant de gérer vos ressources de bot et d’interagir avec elles directement à partir de la ligne de commande. 

>[!TIP]
> Chaque outil Bot Builder inclut une commande d’aide globale, que vous pouvez exécuter à partir de la ligne de commande en entrant **-h** ou **--help**. Cette commande est disponible à tout moment à partir de n’importe quelle action, affichant les options disponibles, ainsi que leurs descriptions.

### <a name="ludown"></a>LUDown
[LUDown](https://github.com/Microsoft/botbuilder-tools/tree/master/Ludown) vous permet de décrire et de créer des composants linguistiques puissants pour les bots à l’aide de fichiers **.lu**. Le nouveau fichier .lu est un type de format Markdown qu’utilise l’outil LUDown et qui génère des fichiers .json propres au service cible. Vous pouvez utiliser des fichiers .lu pour créer une application [LUIS](https://docs.microsoft.com/azure/cognitive-services/luis/luis-get-started-create-app) ou une base de connaissances de [questions/réponses](https://qnamaker.ai/Documentation/CreateKb), en utilisant des formats différents pour chacune d’elles. LUDown est disponible en tant que module npm et vous pouvez l’installer globalement sur votre machine :

```shell
npm install -g ludown
```
Vous pouvez utiliser l’outil LUDown pour créer des modèles .json pour LUIS et le service de questions/réponses.  


### <a name="creating-a-luis-application-with-ludown"></a>Création d’une application LUIS avec LUDown

Vous pouvez définir des [intentions](https://docs.microsoft.com/azure/cognitive-services/luis/add-intents) et des [entités](https://docs.microsoft.com/azure/cognitive-services/luis/add-entities) pour une application LUIS comme vous le feriez à partir du portail LUIS. 

`# \<intent-name\>` décrit une nouvelle section de définition d’intention. Les lignes suivantes contiennent des [énoncés](https://docs.microsoft.com/azure/cognitive-services/luis/add-example-utterances) décrivant cette intention.

Par exemple, vous pouvez créer plusieurs intentions LUIS dans un fichier .lu unique comme suit : 

```ludown
# Greeting
Hi
Hello
Good morning
Good evening

# Help
help
I need help
please help
```

### <a name="qna-pairs-with-ludown"></a>Paires de questions/réponses avec LUDown

Le format de fichier .lu prend en charge également les paires de questions/réponses à l’aide de la notation suivante : 

  ```ludown
  > This is a comment. QnA definitions have the general format:
  ### ? this-is-the-question-string
  - this-is-an-alternate-form-of-the-same-question
  - this-is-another-one
    ```markdown
    this-is-the-answer
    ```
  ```
L’outil LUDown sépare automatiquement les questions/réponses dans un fichier JSON qnamaker que vous pouvez ensuite utiliser pour créer votre base de connaissances [QnaMaker.ai](http://qnamaker.ai).

  ```ludown
  ### ? How do I change the default message for QnA Maker?
    ```markdown
    You can change the default message if you use the QnAMakerDialog. 
    See [this link](https://docs.botframework.com/en-us/azure-bot-service/templates/qnamaker/#navtitle) for details. 
    ```
  ```

Vous pouvez également ajouter plusieurs questions à la même réponse en ajoutant simplement de nouvelles lignes de variantes de questions pour une réponse unique. 

  ```ludown
  ### ? What is your name?
  - What should I call you?
    ```markdown
    I'm the echoBot! Nice to meet you.
    ```
  ```

### <a name="generating-json-models-with-ludown"></a>Génération de modèles .json avec LUDown

Après avoir défini les composants linguistiques LUIS ou de questions/réponses au format .lu, vous pouvez procéder à une publication dans un fichier LUIS .json, ou bien de questions/réponses .json ou .tsv. Quand il est exécuté, l’outil LUDown recherche tous les fichiers .lu dans le répertoire de travail à analyser. L’outil LUDown pouvant cibler LUIS ou le service de questions/réponses avec des fichiers .lu, nous devons simplement spécifier le service de langage pour lequel effectuer la génération, en utilisant la commande générale **ludown parse <Service> -- in <luFile>**. 

Dans notre exemple de répertoire de travail, nous avons deux fichiers .lu à analyser : « luis-sample.lu » pour créer le modèle LUIS et « qna-sample.lu » pour créer une base de connaissances de questions/réponses.


#### <a name="generate-luis-json-models"></a>Générer des modèles LUIS .json

**luis-sample.lu** 
```ludown
# Greeting
- Hi
- Hello
- Good morning
- Good evening
```

Pour générer un modèle LUIS à l’aide de LUDown, dans votre répertoire de travail actif, entrez simplement la commande suivante :

```shell
ludown parse ToLuis --in ludown-file-name.lu
```

#### <a name="generate-qna-knowledge-base-json"></a>Générer le fichier .json de la base de connaissances de questions/réponses

**qna-sample.lu**
  ```ludown
  > This is a sample ludown file for QnA Maker.

  ### ? How do I change the default message
    ```markdown
    You can change the default message if you use the QnAMakerDialog. 
    See [this link](https://docs.botframework.com/en-us/azure-bot-service/templates/qnamaker/#navtitle) for details. 
    ```
  ```

De même, pour créer une base de connaissances de questions/réponses, vous devez uniquement changer la cible d’analyse. 

```shell
ludown parse ToQna --in ludown-file-name.lu
```

Les fichiers JSON obtenus peuvent être utilisés par LUIS et le service de questions/réponses via leurs portails respectifs, ou via les nouveaux outils CLI. 

## <a name="6-connect-to-luis-an-qna-maker-services-from-the-cli"></a>6. Se connecter aux services LUIS et QnA Maker à partir de l’interface CLI

### <a name="connect-to-luis-from-the-cli"></a>Se connecter à LUIS à partir de l’interface CLI 

Le nouveau jeu d’outils comprend une [extension LUIS](https://github.com/Microsoft/botbuilder-tools/tree/master/LUIS) qui vous permet de gérer vos ressources LUIS. Il est disponible en tant que module npm que vous pouvez télécharger :

```shell
npm install -g luis-apis
```
L’utilisation de base de la commande pour l’outil LUIS à partir de l’interface CLI est la suivante :

```shell
luis action-name resource-name arguments-list
```
Pour connecter votre bot à LUIS, vous devez créer un fichier **.luisrc**. Il s’agit d’un fichier de configuration qui provisionne l’ID d’application et le mot de passe LUIS sur le point de terminaison de service quand votre application effectue des appels sortants. Vous pouvez créer ce fichier en exécutant **luis init** comme suit :

```shell
luis init
```
Dans le terminal, vous devrez entrer votre clé de création LUIS, région et ID d’application pour que l’outil puisse générer le fichier.  

![LUIS init](media/bot-builder-tools/luis-init.png) 


Une fois ce fichier généré, votre application est en mesure d’utiliser le fichier .json LUIS (généré à partir de LUDown) à l’aide de la commande suivante à partir de l’interface CLI : 

```shell
luis import application --in luis-app.json | msbot connect luis --stdin
```

### <a name="connect-to-qna-from-the-cli"></a>Se connecter au service de questions/réponses à partir de l’interface CLI

Le nouveau jeu d’outils comprend une [extension de questions/réponses](https://github.com/Microsoft/botbuilder-tools/tree/master/QnAMaker) qui vous permet de gérer vos ressources LUIS. Il est disponible en tant que module npm que vous pouvez télécharger :

```shell
npm install -g qnamaker
```
Avec l’outil QnA Maker, vous pouvez créer, mettre à jour, publier, supprimer et former votre base de connaissances. Pour commencer, vous devez créer un fichier **.qnamakerrc** nécessaire pour activer le point de terminaison sur votre service. Vous pouvez facilement créer ce fichier en exécutant **qnamaker init** et en suivant les invites et l’ID de la base de connaissances QnA Maker. 

```shell
qnamaker init 
```
![QnaMaker init](media/bot-builder-tools/qnamaker-init.png)

Une fois votre fichier .qnamakerrc généré, vous pouvez vous connecter à votre base de connaissances de questions/réponses pour utiliser le fichier.json/.tsv de la base de connaissances (généré à partir de LUDown) à l’aide de la commande suivante :

```shell
qnamaker create --in qnaKB.json --msbot | msbot connect qna --stdin
```

## <a name="7-publish-to-azure-from-the-cli"></a>7. Publier sur Azure à partir de l’interface CLI

Après avoir apporté des modifications au code source de votre bot, vous pouvez en toute transparence publier ces modifications en exécutant la commande suivante :

```azurecli
az bot publish --name "my-bot-name" --resource-group "my-resource-group"
```

## <a name="references"></a>Références
- [Code source des outils Bot Builder](https://github.com/Microsoft/botbuilder-tools)
- [MSBot](https://github.com/Microsoft/botbuilder-tools/tree/master/MSBot)
- [ChatDown](https://github.com/Microsoft/botbuilder-tools/tree/master/Chatdown)
- [LUDown](https://github.com/Microsoft/botbuilder-tools/tree/master/ludown)
- [interface de ligne de commande Azure](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest)


