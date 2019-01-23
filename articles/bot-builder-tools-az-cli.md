---
title: Créer des bots avec l’interface de ligne de commande Azure
description: Les outils Bot Builder vous permettent de gérer vos ressources bot directement à partir de la ligne de commande
author: matthewshim-ms
ms.author: v-shimma
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: tools
ms.date: 10/31/2018
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 4b09ca152f99faa66d2da55ebeb93fb9cce090db
ms.sourcegitcommit: b94361234816e6b95459f142add936732fc40344
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/15/2019
ms.locfileid: "54317689"
---
# <a name="create-bots-with-azure-cli"></a>Créer des bots avec l’interface de ligne de commande Azure

[!INCLUDE [pre-release-label](./includes/pre-release-label-v3.md)]

Ce didacticiel vous montre comment effectuer les opérations suivantes : 
- Créer un bot avec Azure CLI 
- Télécharger une copie locale pour le développement
- Utiliser le nouvel outil MSBot pour stocker toutes les informations sur les ressources du bot
- Gérer, créer ou mettre à jour des modèles LUIS et QnA avec LUDown
- Se connecter aux services LUIS et QnA Maker à partir de l’interface CLI
- Déployer votre bot sur Azure à partir de l’interface CLI

## <a name="prerequisites"></a>Prérequis

Pour que vous puissiez utiliser ces outils à partir de la ligne de commande, Node.js doit être installé sur votre ordinateur : 
- [Node.js (version 8.5 ou ultérieure)](https://nodejs.org/en/)

## <a name="1-install-tools"></a>1. Installer des outils
1. [Installez](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest) la version la plus récente de l’interface de ligne de commande Azure.
2. [Installez](https://aka.ms/botbuilder-tools-readme) Bot Framework Tools.

Vous pouvez désormais gérer les bots à l’aide de l’interface de ligne de commande Azure comme n’importe quelle autre ressource Azure.

>[!TIP]
> Actuellement, Azure Bot Extension prend uniquement en charge les bots v3.
  
3. Connectez-vous à l’interface de ligne de commande Azure à l’aide de la commande suivante.

```azurecli
az login
```
Une nouvelle fenêtre de navigateur s’ouvre ; connectez-vous. Une fois connecté, vous obtenez le message suivant :

![Connexion de l’appareil MS](media/bot-builder-tools/az-browser-login.png)

Dans la fenêtre de ligne de commande, vous voyez les informations suivantes :

![Commande de connexion Azure](media/bot-builder-tools/az-login-command.png)

## <a name="2-create-a-new-bot-from-azure-cli"></a>2. Créer un bot à partir de l’interface de ligne de commande Azure

Utilisation de l’interface de ligne de commande Azure pour créer des bots uniquement à l’aide de cette méthode. 

```azurecli
az bot [command]
```
|Commandes|  |
|----|----|
| create      |Créer un bot|
| delete     |Supprimer un bot existant|
| download   |Télécharger un bot existant|
| publish   |Publier sur le service d’application associé du bot|
| show |Obtenir un bot existant|
| update|Mettre à jour un bot existant|

Pour créer un bot à partir de l’interface CLI, vous devez sélectionner un [groupe de ressources](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-overview) existant ou en créer un. 

```azurecli
az bot create --resource-group "my-resource-group" --name "my-bot-name" --kind "my-resource-type" --version v3 --description "description-of-my-bot" --lang "programming-language"
```
Les valeurs autorisées pour `--kind` sont : `function, registration, webapp`. Pour `--version`, ce sont : `v3, v4`.  Si vous ne spécifiez pas l’argument `--lang`, un bot .NET est créé. Pour créer un bot de nœud, utilisez `Node`.

Après une demande réussie, un message de confirmation apparaît.
```
Obtained msa app id and password. Provisioning bot now.
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

Votre nouveau bot echo sera provisionné sur votre groupe de ressources dans Azure. Pour le tester, il suffit de sélectionner **Tester dans le chat web** dans l’en-tête de gestion du bot de la vue Web App Bot. 

![Bot echo Azure](media/bot-builder-tools/az-echo-bot.png) 

## <a name="3-download-the-bot-locally"></a>3. Télécharger le bot localement

Vous pouvez télécharger le code source de deux façons :
- À partir du portail Azure.
- Via la nouvelle interface de ligne de commande Azure.

Pour télécharger de code source du bot à partir du [portail Microsoft Azure](http://portal.azure.com), il suffit de sélectionner la ressource de bot, puis **Build** sous la gestion de bot. Plusieurs options sont disponibles pour gérer ou extraire localement le code source de votre bot.

![Télécharger le bot depuis le portail Azure](media/bot-builder-tools/az-portal-manage-code.png)

Pour télécharger la source de votre bot à l’aide de l’interface CLI, entrez la commande suivante. Le bot est téléchargé en local.

```azurecli
az bot download --name "my-bot-name" --resource-group "my-resource-group"
```

![Commande de téléchargement à l’aide de l’interface CLI](media/bot-builder-tools/cli-bot-download-command.png)

## <a name="4-store-your-bot-information-with-msbot"></a>4. Stocker les informations de votre bot avec MSBot

Le nouvel outil MSBot permet de créer un fichier  **.bot** qui stocke les métadonnées des différents services utilisés par votre bot à un seul et même emplacement. Ce fichier permet également à votre bot de se connecter à ces services à partir de l’interface CLI. Les outils MSBot prennent en charge plusieurs commandes. Pour plus de détails, consultez le fichier [readme](https://aka.ms/botbuilder-tools-msbot-readme). 

Pour installer MSBot, exécutez :

```shell
npm install -g msbot
```

Pour créer un fichier de robot, dans votre interface de ligne de commande, entrez **msbot init**, suivi du nom de votre robot et du point de terminaison de l’URL cible, par exemple :

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

Consultez le fichier [readme](https://aka.ms/botbuilder-tools-msbot-readme) pour obtenir la liste complète des services pris en charge.

### <a name="connect-your-bot-to-abs-with-the-bot-file"></a>Connecter votre bot à ABS avec le fichier .bot

Une fois l’outil MSBot installé, vous pouvez facilement connecter votre bot à un groupe de ressources existant dans Azure Bot Service en exécutant la commande az bot **show**.

```azurecli
az bot show -n my-bot-name -g my-resource-group --msbot | msbot connect azure --stdin
```

Cette commande récupère le point de terminaison actuel, ainsi que l’ID et le mot de passe d’application du groupe de ressources cible et met à jour les informations en conséquence dans votre fichier .bot.


## <a name="5-manage-update-or-create-luis-and-qna-services-with--new-botbuilder-tools"></a>5. Gérer, mettre à jour ou créer des services LUIS et de questions/réponses avec les nouveaux outils Bot Builder

[Bot Framework Tools](https://aka.ms/botbuilder-tools) est un nouvel ensemble d’outils qui vous permet de gérer vos ressources de bot et d’interagir avec elles directement à partir de la ligne de commande.

>[!TIP]
> Chaque outil Bot Framework inclut une commande d’aide globale à laquelle vous pouvez accéder à partir de la ligne de commande en entrant **-h** ou **--help**. Cette commande est disponible à tout moment à partir de n’importe quelle action, affichant les options disponibles, ainsi que leurs descriptions.

### <a name="ludown"></a>LUDown

[LUDown](https://aka.ms/botbuilder-ludown) vous permet de décrire et de créer des composants linguistiques puissants pour les bots à l’aide de fichiers **.lu**. Le nouveau fichier .lu est un type de format Markdown qu’utilise l’outil LUDown et qui génère des fichiers .json propres au service cible. Vous pouvez utiliser des fichiers .lu pour créer une application [LUIS](https://docs.microsoft.com/azure/cognitive-services/luis/luis-get-started-create-app) ou une base de connaissances de [questions/réponses](https://qnamaker.ai/Documentation/CreateKb), en utilisant des formats différents pour chacune d’elles. LUDown est disponible en tant que module npm et vous pouvez l’installer globalement sur votre machine :

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

Le nouveau jeu d’outils comprend une [extension LUIS](https://aka.ms/botbuilder-luis-cli) qui vous permet de gérer vos ressources LUIS. Il est disponible en tant que module npm que vous pouvez télécharger :

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

Le nouveau jeu d’outils comprend une [extension de questions/réponses](https://aka.ms/botbuilder-tools-qnaMaker) qui vous permet de gérer vos ressources LUIS. Il est disponible en tant que module npm que vous pouvez télécharger :

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
- [Bot Framework Tools](https://aka.ms/botbuilder-tools-readme)
- [Interface de ligne de commande Azure](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest)
