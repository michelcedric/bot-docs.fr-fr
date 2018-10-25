---
title: Créer des expériences pilotées par les données avec Recherche Azure | Microsoft Docs
description: Découvrez comment créer des expériences pilotées par les données avec Recherche Azure, et permettre aux utilisateurs de faire naviguer de grandes quantités de contenu dans un bot au moyen du kit SDK Bot Builder pour .NET et de Recherche Azure.
author: matthewshim-ms
ms.author: v-shimma
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 8205f40053b2b3d0e62d9b9ce622f59432e059a4
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2018
ms.locfileid: "49999006"
---
# <a name="create-data-driven-experiences-with-azure-search"></a>Créer des expériences pilotées par les données avec Recherche Azure 

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-search-azure.md)
> - [Node.JS](../nodejs/bot-builder-nodejs-search-azure.md)

Vous pouvez ajouter [Recherche Azure](https://azure.microsoft.com/en-us/services/search/) à un bot pour permettre aux utilisateurs de faire naviguer de grandes quantités de contenu, et pour créer une expérience d’exploration pilotée par les données.

Recherche Azure est un service Azure qui offre des fonctionnalités, telles que la recherche par mot clé, une linguistique intégrée, la notation personnalisée, une navigation par facettes, et bien plus encore. Recherche Azure peut également indexer le contenu à partir de diverses sources, y compris Azure SQL DB, DocumentDB, Stockage Blob et Stockage Table. La solution prend en charge l’indexation par « push » pour d’autres sources de données, et peut ouvrir des fichiers PDF, des documents Office ainsi que d’autres formats contenant des données non structurées. Une fois collecté, le contenu est placé dans un index Recherche Azure, que le robot peut ensuite interroger.


## <a name="prerequisites"></a>Prérequis

Installez le package Nuget [Microsoft.Azure.Search](https://www.nuget.org/packages/Microsoft.Azure.Search/4.0.0-preview) dans votre projet de bot. 

Les trois projets C# suivants sont indispensables à votre solution de bot. Ces projets fournissent des fonctionnalités supplémentaires pour les bots et la Recherche Azure. Dupliquez (fork) les projets à partir de [GitHub](https://github.com/Microsoft/botBuilder-Samples/tree/master/CSharp/demo-Search) ou téléchargez le code source directement.

* [Search.Azure](https://github.com/Microsoft/botBuilder-Samples/tree/master/CSharp/demo-Search/Search.Azure) définit l’appel du service Azure. 
* [Search.Contracts](https://github.com/Microsoft/botBuilder-Samples/tree/master/CSharp/demo-Search/Search.Contracts) définit les interfaces génériques et les modèles de données pour traiter les données.
* [Search.Dialogs](https://github.com/Microsoft/botBuilder-Samples/tree/master/CSharp/demo-Search/Search.Dialogs) inclut les différents dialogues Bot Builder génériques, utilisés pour interroger la Recherche Azure.

## <a name="configure-azure-search-settings"></a>Configurer les paramètres de Recherche Azure 

Configurez les paramètres de Recherche Azure dans le fichier **Web.config** du projet avec vos propres informations d’identification de Recherche Azure dans les champs de valeur. Le constructeur dans la classe `AzureSearchClient` utilise ces paramètres pour inscrire et lier le bot au service Azure.

```xml
<appSettings>
    <add key="SearchDialogsServiceName" value="Azure-Search-Service-Name" /> <!-- replace value field with Azure Service Name --> 
    <add key="SearchDialogsServiceKey" value="Azure-Search-Service-Primary-Key" /> <!-- replace value field with Azure Service Key --> 
    <add key="SearchDialogsIndexName" value="Azure-Search-Service-Index" /> <!-- replace value field with your Azure Search Index --> 
</appSettings>
```

## <a name="create-a-search-dialog"></a>Créer un dialogue de recherche

Dans votre projet de bot, créez une classe `AzureSearchDialog` pour appeler le service Azure dans le bot. Cette nouvelle classe doit hériter de la classe `SearchDialog` à partir du projet **Search.Dialogs**, qui gère le plus gros du travail. Le remplacement `GetTopRefiners()` permet aux utilisateurs d’affiner/filtrer leurs résultats de recherche sans avoir à recommencer la recherche depuis le début, par la conservation de l’état de l’objet de recherche. Vous pouvez ajouter vos propres affinements personnalisés dans le tableau `TopRefiners` pour laisser vos utilisateurs filtrer ou affiner leurs résultats de recherche. 

```cs
[Serializable]
public class AzureSearchDialog : SearchDialog
{
    private static readonly string[] TopRefiners = { "refiner1", "refiner2", "refiner3" }; // define your own custom refiners 

    public AzureSearchDialog(ISearchClient searchClient) : base(searchClient, multipleSelection: true)
    {
    }

    protected override string[] GetTopRefiners()
    {
        return TopRefiners;
    }
}
```

## <a name="define-the-response-data-model"></a>Définir le modèle de données de la réponse

La classe **SearchHit.cs** dans le projet `Search.Contracts` définit les données pertinentes devant être analysées à partir de la réponse de Recherche Azure. Pour votre bot, les seules inclusions obligatoires sont la création et la déclaration IDictionary `PropertyBag` dans le constructeur. Vous pouvez définir toutes les autres propriétés de cette classe par rapport aux besoins de votre bot. 

```cs
[Serializable]
public class SearchHit
{
    public SearchHit()
    {
        this.PropertyBag = new Dictionary<string, object>();
    }

    public IDictionary<string, object> PropertyBag { get; set; }

    // customize the fields below as needed 
    public string Key { get; set; }

    public string Title { get; set; }

    public string PictureUrl { get; set; }

    public string Description { get; set; }
}
```

## <a name="after-azure-search-responds"></a>Après la réponse de Recherche Azure 

Suite à une requête réussie auprès du service Azure, le résultat de la recherche doit être analysé pour en extraire les données pertinentes à afficher par le bot pour l’utilisateur. Pour ce faire, vous devez créer une classe `SearchResultMapper`. L’objet `GenericSearchResult` créé dans le constructeur définit une liste et un dictionnaire pour stocker respectivement les résultats et les facettes après l’analyse de chaque résultat effectué par rapport aux modèles de données de votre bot. 

Synchronisez les propriétés dans la méthode `ToSearchHit` pour correspondre au modèle de données dans **SearchHit.cs**. La méthode `ToSearchHit` est exécutée et génère un nouvel objet `SearchHit` pour chaque résultat trouvé dans la réponse.  

```cs
public class SearchResultMapper : IMapper<DocumentSearchResult, GenericSearchResult>
{
    public GenericSearchResult Map(DocumentSearchResult documentSearchResult)
    {
        var searchResult = new GenericSearchResult();

        searchResult.Results = documentSearchResult.Results.Select(r => ToSearchHit(r)).ToList();
        searchResult.Facets = documentSearchResult.Facets?.ToDictionary(kv => kv.Key, kv => kv.Value.Select(f => ToFacet(f)));

        return searchResult;
    }

    private static GenericFacet ToFacet(FacetResult facetResult)
    {
        return new GenericFacet
        {
            Value = facetResult.Value,
            Count = facetResult.Count.Value
        };
    }

    private static SearchHit ToSearchHit(SearchResult hit)
    {
        return new SearchHit
        {
            // custom properties defined in SearchHit.cs 
            Key = (string)hit.Document["id"],
            Title = (string)hit.Document["title"],
            PictureUrl = (string)hit.Document["thumbnail"],
            Description = (string)hit.Document["description"]
        };
    }
}
```
Une fois les résultats analysés et stockés, il ne reste plus qu’à afficher les informations pour l’utilisateur. La classe `SearchHitStyler` doit être managée pour prendre en compte votre modèle de données issu de la classe `SearchHit`. Par exemple, les propriétés `Title`, `PictureUrl` et `Description` à partir de la classe **SearchHit.cs** sont utilisées dans l’exemple de code pour créer des pièces jointes de carte. Le code suivant crée une pièce jointe de carte pour chaque objet `SearchHit` de la liste des résultats `GenericSearchResult` devant être affiché pour l’utilisateur.   

```cs
[Serializable]
public class SearchHitStyler : PromptStyler
{
    public override void Apply<T>(ref IMessageActivity message, string prompt, IReadOnlyList<T> options, IReadOnlyList<string> descriptions = null)
    {
        var hits = options as IList<SearchHit>;
        if (hits != null)
        {
            var cards = hits.Select(h => new ThumbnailCard
            {
                Title = h.Title,
                Images = new[] { new CardImage(h.PictureUrl) },
                Buttons = new[] { new CardAction(ActionTypes.ImBack, "Pick this one", value: h.Key) },
                Text = h.Description
            });

            message.AttachmentLayout = AttachmentLayoutTypes.Carousel;
            message.Attachments = cards.Select(c => c.ToAttachment()).ToList();
            message.Text = prompt;
        }
        else
        {
            base.Apply<T>(ref message, prompt, options, descriptions);
        }
    }
}
```
Les résultats de la recherche sont affichés pour l’utilisateur, et vous venez d’ajouter la Recherche Azure à votre bot.

## <a name="samples"></a>Exemples

Pour voir deux exemples complets qui détaillent la prise en charge de Recherche Azure par des bots à l’aide du kit SDK Bot Builder pour .NET, consultez l’[exemple de bot Real Estate](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/demo-Search/RealEstateBot) (Immobilier) ou l’[exemple de bot Job Listing](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/demo-Search/JobListingBot) (Liste des tâches) dans GitHub. 

## <a name="additional-resources"></a>Ressources supplémentaires
* [Recherche Azure][search]
* [Vue d’ensemble des boîtes de dialogue](bot-builder-dotnet-dialogs.md)
* [Exemples de bot Recherche Azure](https://github.com/Microsoft/botBuilder-Samples/tree/master/CSharp/demo-Search)

[search]: /azure/search/search-what-is-azure-search
