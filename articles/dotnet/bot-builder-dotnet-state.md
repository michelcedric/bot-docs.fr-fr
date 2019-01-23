---
title: Gérer les données d’état | Microsoft Docs
description: Découvrez comment enregistrer et récupérer des données d’état avec le kit SDK Bot Framework pour .NET.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/17
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 3ee3af72d1c03faf485a64adb8d9fa2548f5d99d
ms.sourcegitcommit: 3cc768a8e676246d774a2b62fb9c688bbd677700
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/16/2019
ms.locfileid: "54323665"
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
| `SetConversationData` | Conversation | Enregistrer les données d’état pour la conversation sur le canal spécifié <br/><br/>**Remarque**: sachant que la méthode `DeleteStateForUser` ne supprime pas les données qui ont été stockées à l’aide de la méthode `SetConversationData`, vous ne devez PAS utiliser cette méthode pour stocker les informations d’identification personnelle (PII) d’un utilisateur. |
| `SetPrivateConversationData` | Utilisateur et conversation | Enregistrer les données d’état pour l’utilisateur dans la conversation sur le canal spécifié |
| `DeleteStateForUser` | Utilisateur | Supprimer les données d’état de l’utilisateur qui ont été précédemment stockées par le biais de la méthode `SetUserData` ou de la méthode `SetPrivateConversationData` <br/><br/>**Remarque**: votre bot doit appeler cette méthode quand il reçoit une activité de type [deleteUserData](bot-builder-dotnet-activities.md#deleteuserdata) ou une activité de type [contactRelationUpdate](bot-builder-dotnet-activities.md#contactrelationupdate) qui indique que le bot a été supprimé de la liste de contacts de l’utilisateur. |

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

## <a name="handle-concurrency-issues"></a>Gérer les problèmes de concurrence

Il est possible que votre bot reçoive une réponse d’erreur avec le code d’état HTTP **412 (Échec de la précondition)** s’il tente d’enregistrer des données d’état alors qu’une autre instance du bot a modifié les données. Vous pouvez concevoir votre bot de manière à prendre en compte ce scénario, comme indiqué dans l’exemple de code suivant.

[!code-csharp[Handle exception saving state](../includes/code/dotnet-state.cs#handleException)]

## <a name="additional-resources"></a>Ressources supplémentaires

- [Guide de résolution des problèmes généraux Bot Framework](../bot-service-troubleshoot-general-problems.md)
- <a href="/dotnet/api/?view=botbuilder-3.11.0" target="_blank">Informations de référence sur le kit SDK Bot Framework pour .NET</a>

[Activity]: https://docs.botframework.com/en-us/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html
