---
title: Gérer les données d’état | Microsoft Docs
description: Découvrez comment enregistrer et récupérer des données d’état avec le Kit de développement logiciel (SDK) Bot Builder pour .NET.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/17
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: deb8361a5cca2f37840abb1180c2de571ee08143
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2018
ms.locfileid: "49999756"
---
# <a name="manage-state-data"></a>Gérer les données d’état

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-state.md)
> - [Node.JS](../nodejs/bot-builder-nodejs-state.md)

[!INCLUDE [State concept overview](../includes/snippet-dotnet-concept-state.md)]

## <a name="in-memory-data-storage"></a>Stockage de données en mémoire

Le stockage de données en mémoire est uniquement destiné aux tests. Ce stockage est volatile et temporaire. Les données sont effacées chaque fois que le bot est redémarré. Pour utiliser le stockage en mémoire à des fins de test, vous devez procéder aux opérations suivantes : 

Installez les packages NuGet suivants : 
- Microsoft.Bot.Builder.Azure
- Autofac.WebApi2

Dans la méthode **Application_Start**, créez une autre instance du stockage en mémoire, puis inscrivez le nouveau magasin de données :

```cs
// Global.asax file

var store = new InMemoryDataStore();

Conversation.UpdateContainer(
           builder =>
           {
               builder.Register(c => store)
                         .Keyed<IBotDataStore<BotData>>(AzureModule.Key_DataStore)
                         .AsSelf()
                         .SingleInstance();

               builder.Register(c => new CachingBotDataStore(store,
                          CachingBotDataStoreConsistencyPolicy
                          .ETagBasedConsistency))
                          .As<IBotDataStore<BotData>>()
                          .AsSelf()
                          .InstancePerLifetimeScope();


           });
GlobalConfiguration.Configure(WebApiConfig.Register);

```

Vous pouvez utiliser cette méthode pour définir votre propre stockage de données personnalisé ou utiliser l’une des *extensions Azure*.

## <a name="manage-custom-data-storage"></a>Gérer le stockage de données personnalisé

Pour des raisons de performances et de sécurité dans l’environnement de production, vous pouvez implémenter votre propre stockage de données ou envisager de mettre en œuvre l’une des options de stockage de données suivantes :

1. [Gérer les données d’état avec Cosmos DB](bot-builder-dotnet-state-azure-cosmosdb.md)

2. [Gérer les données d’état avec le stockage de table](bot-builder-dotnet-state-azure-table-storage.md)

Avec l’une de ces options [d’extension Azure](https://www.nuget.org/packages/Microsoft.Bot.Builder.Azure/), le mécanisme de définition et de persistance des données par le biais du Kit de développement logiciel (SDK) Bot Framework pour .NET reste identique à celui du stockage de données en mémoire.

## <a name="bot-state-methods"></a>Méthodes d’état Bot

Ce tableau répertorie les méthodes que vous pouvez utiliser pour gérer les données d’état.

| Méthode | Étendue | Objectif |                                                
|----|----|----|
| `GetUserData` | Utilisateur | Obtenir les données d’état précédemment enregistrées pour l’utilisateur sur le canal spécifié |
| `GetConversationData` | Conversation | Obtenir les données d’état précédemment enregistrées pour la conversation sur le canal spécifié |
| `GetPrivateConversationData` | Utilisateur et conversation | Obtenir les données d’état précédemment enregistrées pour l’utilisateur dans la conversation sur le canal spécifié |
| `SetUserData` | Utilisateur | Enregistrer les données d’état pour l’utilisateur sur le canal spécifié |
| `SetConversationData` | Conversation | Enregistrer les données d’état pour la conversation sur le canal spécifié <br/><br/>**Remarque** : étant donné que la méthode `DeleteStateForUser` ne supprime pas les données qui ont été stockées à l’aide de la méthode `SetConversationData`, vous ne devez PAS utiliser cette méthode pour stocker les informations d’identification personnelle (PII) d’un utilisateur. |
| `SetPrivateConversationData` | Utilisateur et conversation | Enregistrer les données d’état pour l’utilisateur dans la conversation sur le canal spécifié |
| `DeleteStateForUser` | Utilisateur | Supprimer les données d’état de l’utilisateur qui ont été précédemment stockées par le biais de la méthode `SetUserData` ou de la méthode `SetPrivateConversationData` <br/><br/>**Remarque** : votre bot doit appeler cette méthode lorsqu’il reçoit une activité du type [deleteUserData](bot-builder-dotnet-activities.md#deleteuserdata) ou une activité du type [contactRelationUpdate](bot-builder-dotnet-activities.md#contactrelationupdate) indiquant que le bot a été supprimé de la liste de contacts de l’utilisateur. |

Si votre bot enregistre les données d’état en utilisant l’une des méthodes « **Set...Data** », les messages que votre bot recevra par la suite dans le même contexte contiendront ces données, auxquelles votre bot pourra accéder à l’aide de la méthode « **Get...Data** » correspondante.

## <a name="useful-properties-for-managing-state-data"></a>Propriétés utiles pour la gestion des données d’état

Chaque objet [Activity][Activity] contient des propriétés que vous utiliserez pour gérer les données d’état.

| Propriété | Description | Cas d’utilisation |
|----|----|----|
| `From` | Identifie de manière unique un utilisateur sur un canal | Stockage et récupération des données d’état qui sont associées à un utilisateur |
| `Conversation` | Identifie de manière unique une conversation | Stockage et récupération des données d’état qui sont associées à une conversation |
| `From` et `Conversation` | Identifie de manière unique un utilisateur et une conversation | Stockage et récupération des données d’état qui sont associées à un utilisateur spécifique dans le contexte d’une conversation donnée |

> [!NOTE]
> Vous pouvez utiliser ces valeurs de propriété en tant que clés, même si vous choisissez de stocker les données d’état dans votre propre base de données, plutôt que d’utiliser le magasin de données d’état Bot Framework.

## <a id="state-client"></a> Créer un client d’état

L’objet `StateClient` vous permet de gérer les données d’état en utilisant le Kit de développement logiciel (SDK) Bot Builder pour .NET. Si vous avez accès à un message appartenant au même contexte que celui dans lequel vous souhaitez gérer les données d’état, vous pouvez créer un client d’état en appelant la méthode `GetStateClient` sur l’objet `Activity`.

[!code-csharp[Get State client](../includes/code/dotnet-state.cs#getStateClient1)]

Si vous n’avez pas accès à un message appartenant au contexte dans lequel vous souhaitez gérer les données d’état, vous pouvez créer un client d’état en créant simplement une autre instance de la classe `StateClient`. Dans cet exemple, `microsoftAppId` et `microsoftAppPassword` correspondent aux informations d’identification d’authentification de Bot Framework dont vous faites l’acquisition pour votre bot lors du processus de [création du bot](../bot-service-quickstart.md).

> [!NOTE]
> Pour trouver les paramètres **AppID** et **AppPassword** de votre bot, consultez la section [MicrosoftAppID and MicrosoftAppPassword](~/bot-service-manage-overview.md#microsoftappid-and-microsoftapppassword) (MicrosoftAppID et MicrosoftAppPassword).

[!code-csharp[Get State client](../includes/code/dotnet-state.cs#getStateClient2)]

> [!NOTE]
> Le client d’état par défaut est stocké dans un service central. Dans le cas de certains canaux, vous souhaiterez utiliser une API d’état qui est hébergée dans le canal proprement dit, afin que les données d’état puissent être stockées dans un magasin conforme fourni par le canal.

## <a name="get-state-data"></a>Obtenir les données d’état

Chacune des méthodes « **Get...Data** » renvoie un objet `BotData` qui contient les données d’état pour l’utilisateur et/ou la conversation spécifiés. Pour obtenir une valeur de propriété spécifique à partir d’un objet `BotData`, appelez la méthode `GetProperty`. 

L’exemple de code ci-après indique comment obtenir une propriété typée à partir des données utilisateur. 

[!code-csharp[Get state property](../includes/code/dotnet-state.cs#getProperty1)]

L’exemple de code ci-après indique comment obtenir une propriété à partir d’un type complexe au sein des données utilisateur.

[!code-csharp[Get state property](../includes/code/dotnet-state.cs#getProperty2)]

Si aucune donnée d’état n’existe pour l’utilisateur et/ou la conversation spécifiés pour un appel de la méthode « **Get...Data** », l’objet `BotData` qui est renvoyé contient les valeurs de propriété suivantes : 
- `BotData.Data` = null
- `BotData.ETag` = "*"

## <a name="save-state-data"></a>Enregistrer les données d’état

Pour enregistrer les données d’état, commencez par obtenir l’objet `BotData` en appelant la méthode « **Get...Data** » appropriée, puis mettez-le à jour en appelant la méthode `SetProperty` pour chaque propriété que vous souhaitez mettre à jour, et enregistrez-le en appelant la méthode « **Set...Data** » adéquate. 

> [!NOTE]
> Vous pouvez stocker jusqu’à 32 Ko de données pour chaque utilisateur sur un canal, pour chaque conversation sur un canal et pour chaque utilisateur dans le contexte d’une conversation sur un canal. 

L’exemple de code ci-après indique comment enregistrer une propriété typée dans les données utilisateur.

[!code-csharp[Set state property](../includes/code/dotnet-state.cs#setProperty1)]

L’exemple de code ci-après indique comment enregistrer une propriété dans un type complexe au sein des données utilisateur. 

[!code-csharp[Set state property](../includes/code/dotnet-state.cs#setProperty2)]

## <a name="handle-concurrency-issues"></a>Gérer les problèmes de concurrence

Il est possible que votre bot reçoive une réponse d’erreur avec le code d’état HTTP **412 (Échec de la précondition)** s’il tente d’enregistrer des données d’état alors qu’une autre instance du bot a modifié les données. Vous pouvez concevoir votre bot de manière à prendre en compte ce scénario, comme indiqué dans l’exemple de code suivant.

[!code-csharp[Handle exception saving state](../includes/code/dotnet-state.cs#handleException)]

## <a name="additional-resources"></a>Ressources supplémentaires

- [Guide de résolution des problèmes généraux Bot Framework](../bot-service-troubleshoot-general-problems.md)
- <a href="/dotnet/api/?view=botbuilder-3.11.0" target="_blank">Référence concernant le Kit de développement logiciel (SDK) Bot Builder pour .NET</a>

[Activity]: https://docs.botframework.com/en-us/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html
