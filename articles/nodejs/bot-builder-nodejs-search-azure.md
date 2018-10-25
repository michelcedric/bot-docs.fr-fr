---
title: Créer des expériences pilotées par les données avec Recherche Azure | Microsoft Docs
description: Découvrez comment créer des expériences pilotées par les données avec Recherche Azure, et permettre aux utilisateurs naviguer dans de grandes quantités de contenu dans un robot au moyen du Kit de développement logiciel (SDK) Bot Builder pour Node.js et de Recherche Azure.
author: matthewshim-ms
ms.author: v-shimma
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 204fb5d8f4838c78d771bfad5c0ed6511b27932b
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2018
ms.locfileid: "49999996"
---
# <a name="create-data-driven-experiences-with-azure-search"></a>Créer des expériences pilotées par les données avec Recherche Azure 

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-search-azure.md)
> - [Node.JS](../nodejs/bot-builder-nodejs-search-azure.md)

Vous pouvez ajouter [Recherche Azure][search] à votre robot pour permettre à l’utilisateur de naviguer dans de grandes quantités de contenu, et pour créer une expérience d’exploration pilotée par les données pour les utilisateurs de votre robot.

Recherche Azure est un service Azure qui offre des fonctionnalités, telles que la recherche par mot clé, une linguistique intégrée, la notation personnalisée, une navigation par facettes, et bien plus encore. Recherche Azure peut également indexer le contenu à partir de diverses sources, dont Azure SQL DB, DocumentDB, Stockage Blob et Stockage Table. La solution prend en charge l’indexation par « push » pour d’autres sources de données, et peut ouvrir des fichiers PDF, des documents Office ainsi que d’autres formats contenant des données non structurées. Une fois collecté, le contenu est placé dans un index Recherche Azure, que le robot peut ensuite interroger.

## <a name="install-dependencies"></a>Installer des dépendances

À partir d’une invite de commandes, accédez au répertoire de projet de votre robot, puis installez les modules suivants avec le gestionnaire de package de nœud (NPM) :

* [bluebird](https://www.npmjs.com/package/bluebird)
* [lodash](https://www.npmjs.com/package/lodash)
* [requête](https://www.npmjs.com/package/request)

## <a name="prerequisites"></a>Prérequis

Ce qui suit est **obligatoire** : 
- Avoir un abonnement Azure et une clé primaire de Recherche Azure. Vous le trouverez sur le portail Azure.
- Copiez la bibliothèque [SearchDialogLibrary](https://github.com/Microsoft/botBuilder-Samples/tree/master/Node/demo-Search/SearchDialogLibrary) vers le répertoire de projet de votre robot. Cette bibliothèque contient des dialogues généraux dans lesquels l’utilisateur peut chercher, mais peut être personnalisée en fonction des besoins de votre robot. 

- Copiez la bibliothèque [SearchProviders](https://github.com/Microsoft/botBuilder-Samples/tree/master/Node/demo-Search/SearchProviders) vers le répertoire de projet de votre robot. Cette bibliothèque contient tous les composants requis pour créer une demande et l’envoyer à Recherche Azure.

## <a name="connect-to-the-azure-service"></a>Se connecter à Azure Service 

Dans le fichier programme principal de votre robot (par exemple, app.js), créer les chemins d’accès de référence aux deux bibliothèques que vous avez installées précédemment. 

```javascript
var SearchLibrary = require('./SearchDialogLibrary');
var AzureSearch = require('./SearchProviders/azure-search');
```

Ajoutez l’exemple de code suivant à votre robot. Dans l’objet `AzureSearch`, transmettez vos propres paramètres de Recherche Azure à la méthode `.create`. Au moment de l’exécution, celle-ci associera votre robot au service Recherche Azure et attendra une requête utilisateur terminée sous la forme d’une `Promise`.  

```javascript
// Azure Search
var azureSearchClient = AzureSearch.create('Your-Azure-Search-Service-Name', 'Your-Azure-Search-Primary-Key', 'Your-Azure-Search-Service-Index');
var ResultsMapper = SearchLibrary.defaultResultsMapper(ToSearchHit);
```

 La référence `azureSearchClient` crée le modèle Recherche Azure qui transmet les paramètres d’autorisation d’Azure Service à partir des paramètres `.env` du robot. 
 `ResultsMapper` analyse l’objet Response Azure et mappe les données comme nous le définissons dans la méthode `ToSearchHit`. Pour obtenir un exemple d’implémentation de cette méthode, voir [Après la réponse de Recherche Azure](#after-azure-search-responds).

## <a name="register-the-search-library"></a>Inscrire la bibliothèque de recherche
Vous pouvez personnaliser vos dialogues de recherche directement dans le module `SearchLibrary`. La `SearchLibrary` effectue le gros du travail, dont l’appel à Recherche Azure. 

Ajoutez le code suivant dans le fichier programme principal de votre robot pour inscrire la bibliothèque de dialogues de recherche auprès de votre robot. 

```javascript
bot.library(SearchLibrary.create({
    multipleSelection: true,
    search: function (query) { return azureSearchClient.search(query).then(ResultsMapper); },
    refiners: ['refiner1', 'refiner2', 'refiner3'], // customize your own refiners 
    refineFormatter: function (refiners) {
        return _.zipObject(
            refiners.map(function (r) { return 'By ' + _.capitalize(r); }),
            refiners);
    }
}));
```
En plus de stocker tous les dialogues associés votre recherche, la `SearchLibrary` prend la requête utilisateur à envoyer à Recherche Azure. Vous devrez définir vos propres affinements dans le tableau `refiners` afin de spécifier les entités de votre choix pour autoriser votre utilisateur à limiter ou à filtrer ses résultats de recherche.  

## <a name="create-a-search-dialog"></a>Créer une dialogue de recherche

Vous pouvez choisir de structurer vos dialogues comme vous le souhaitez. La seule exigence pour configurer un dialogue de Recherche Azure est d’appeler la méthode `.begin` à partir de l’objet `SearchLibrary`, en passant l’objet `session` généré par le Kit de développement logiciel (SDK) Bot Builder. 

```javascript
function (session) {
        // Trigger Azure Search dialogs 
        SearchLibrary.begin(session);
    },
    function (session, args) {
        // Process selected search results
        session.send(
            'Search Completed!',
            args.selection.map(  ); // format your response 
    }
```
Pour plus d’informations sur les dialogues, voir [Gérer une conversation à l’aide de dialogues](bot-builder-nodejs-dialog-manage-conversation.md).

## <a name="after-azure-search-responds"></a>Après la réponse de Recherche Azure 

Une fois qu’une recherche Azure est correctement résolue, vous devez stocker les données de votre choix à partir de l’objet Response, et les afficher de manière significative à l’utilisateur.

> [!TIP]
> Envisagez d’inclure le [module util][NodeUtil]. Il vous permettra de mettre en forme et de mapper la réponse à partir de Recherche Azure.

Dans le fichier programme principal de votre robot, créez une méthode `ToSearchHit`. Cette méthode retourne un objet qui met en forme les données pertinentes d’Azure Response dont vous avez besoin. Le code suivant montre comment vous pouvez définir vos propres paramètres dans la méthode `ToSearchHit`. 
 
 ```javascript
 function ToSearchHit(azureResponse) {
     return {
         // define your own parameters 
         key: azureResponse.id,
         title: azureResponse.title,
         description: azureResponse.description,
         imageUrl: azureResponse.thumbnail
     };
 }
```
Cela fait, il vous suffit d’afficher les données à l’utilisateur. 

 Dans le fichier **index.js** du projet **SearchDialogLibrary**, la méthode `searchHitAsCard` analyse chaque réponse de Recherche Azure, et crée un objet Card à afficher à l’utilisateur. Les champs que vous avez définis dans la méthode `ToSearchHit` à partir du fichier programme principal de votre robot doivent être synchronisés avec les propriétés de la méthode `searchHitAsCard`. 

L’exemple suivant montre comment et où vos paramètres définis à partir de la méthode `ToSearchHit` sont utilisés pour créer une interface utilisateur de pièce jointe carte à afficher à l’utilisateur. 

```javascript
function searchHitAsCard(showSave, searchHit) {
    var buttons = showSave
        ? [new builder.CardAction().type('imBack').title('Save').value(searchHit.key)]
        : [];

    var card = new builder.HeroCard()
        .title(searchHit.title) 
        .buttons(buttons);

    if (searchHit.description) {
        card.subtitle(searchHit.description);
    }

    if (searchHit.imageUrl) {
        card.images([new builder.CardImage().url(searchHit.imageUrl)]);
    }

    return card;
}
```

## <a name="sample-code"></a>Exemple de code

Pour voir deux exemples complets qui montrent comment prendre en charge Recherche Azure avec des robots à l’aide du Kit de développement logiciel (SDK) Bot Builder pour Node.js, voir l’[exemple de robot Real Estate](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/demo-Search/RealEstateBot) (Immobilier) ou l’[exemple de robot Job Listing](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/demo-Search/JobListingBot) (Liste des tâches) dans GitHub. 

## <a name="additional-resources"></a>Ressources supplémentaires

* [Recherche Azure][search]
* [Nœud Util][NodeUtil]
* [Dialogues](bot-builder-nodejs-dialog-manage-conversation.md)

[NodeUtil]: https://nodejs.org/api/util.html
[search]: /azure/search/search-what-is-azure-search
