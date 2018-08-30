---
title: Gérer les données d’état personnalisé avec le stockage de table Azure | Microsoft Docs
description: Découvrez comment enregistrer et récupérer des données d’état avec le stockage de table Azure par le biais du kit SDK Bot Builder pour .NET.
author: kaiqb
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: e5ff23caa1bdb1158ab19fa7c66e1fe4f6899f49
ms.sourcegitcommit: 2dc75701b169d822c9499e393439161bc87639d2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/24/2018
ms.locfileid: "42905112"
---
# <a name="manage-custom-state-data-with-azure-table-storage-for-net"></a>Gérer les données d’état personnalisé avec le stockage de table Azure pour .NET

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

Dans cet article, vous implémentez le stockage de table Azure pour stocker et gérer les données d’état de votre bot. Le service d’état du connecteur par défaut utilisé par les bots n’est pas destiné à l’environnement de production. Vous devez utiliser les [Extensions Azure](https://github.com/Microsoft/BotBuilder-Azure) disponibles sur GitHub ou implémenter un client avec un état personnalisé à l’aide de la plateforme de stockage de données de votre choix. Voici quelques-unes des raisons d’utiliser le stockage d’état personnalisé :
 - Débit d’API d’état plus élevé (meilleur contrôle des performances)
 - Latence inférieure pour la géodistribution
 - Contrôle de l’emplacement où les données sont stockées
 - Accès aux données d’état réelles
 - Stockage de plus de 32 ko de données

## <a name="prerequisites"></a>Prérequis
Vous devez disposer des éléments suivants :
 - [Compte Microsoft Azure](https://azure.microsoft.com/en-us/free/)
 - [Visual Studio 2015 ou ultérieur](https://www.visualstudio.com/)
 - [Package NuGet Bot Builder Azure](https://www.nuget.org/packages/Microsoft.Bot.Builder.Azure/)
 - [Package NuGet Autofac Web Api2](https://www.nuget.org/packages/Autofac.WebApi2/)
 - [Bot Framework Emulator](https://emulator.botframework.com/)
 - [Azure Storage Explorer](http://storageexplorer.com/)
 
## <a name="create-azure-account"></a>Créer un compte Azure
Si vous n’avez pas encore de compte Azure, cliquez [ici](https://azure.microsoft.com/en-us/free/) pour vous inscrire et bénéficier d’un compte gratuit.

## <a name="set-up-the-azure-table-storage-service"></a>Configurer le service de stockage de table Azure
1. Une fois connecté au portail Azure, créez un service de stockage de table Azure en cliquant sur **Nouveau**. 
2. Recherchez un **compte de stockage** qui implémente la table Azure. 
3. Renseignez les champs, puis cliquez sur le bouton **Créer** en bas de l’écran pour déployer le nouveau service de stockage. Une fois déployé, le nouveau service de stockage affichera les fonctionnalités et options dont vous disposez.
4. Sélectionnez l’onglet **Clés d’accès** sur la gauche, puis copiez la chaîne de connexion pour une utilisation ultérieure. Votre bot utilisera cette chaîne de connexion pour appeler le service de stockage afin d’enregistrer les données d’état.

## <a name="install-nuget-packages"></a>Installer les packages NuGet
1. Ouvrez un projet de bot C# existant ou créez-en un en utilisant le modèle Bot C# dans Visual Studio. 
2. Installez les packages NuGet suivants :
   - Microsoft.Bot.Builder.Azure
   - Autofac.WebApi2

## <a name="add-connection-string"></a>Ajouter une chaîne de connexion 
Ajoutez l’entrée suivante dans votre fichier Web.config : 
```XML
  <connectionStrings>
    <add name="StorageConnectionString"
    connectionString="YourConnectionString"/>
  </connectionStrings>
```
Remplacez « YourConnectionString » par la chaîne de connexion au service de stockage de table Azure que vous avez enregistrée précédemment. Enregistrez le fichier Web.config.

## <a name="modify-your-bot-code"></a>Modifier le code de votre bot
Dans le fichier Global.asax.cs, ajoutez les instructions `using` :
```cs
using Autofac;
using System.Configuration;
using Microsoft.Bot.Connector;
using Microsoft.Bot.Builder.Azure;
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Bot.Builder.Dialogs.Internals;
```
Dans la méthode `Application_Start()`, créez une instance de la classe `TableBotDataStore`. La classe `TableBotDataStore` implémente l’interface `IBotDataStore<BotData>`. L’interface `IBotDataStore` vous permet de remplacer la connexion au service d’état du connecteur par défaut.
 ```cs
 var store = new TableBotDataStore(ConfigurationManager.ConnectionStrings["StorageConnectionString"].ConnectionString);
 ```
Inscrivez le service, comme indiqué ci-dessous :
 ```cs
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
 ```
Enregistrez le fichier Global.asax.cs.

## <a name="run-your-bot-app"></a>Exécuter votre application de bot
Exécutez votre bot dans Visual Studio, le code que vous avez ajouté crée la table **botdata** personnalisée dans Azure.

## <a name="connect-your-bot-to-the-emulator"></a>Connecter votre bot à l’émulateur
À ce stade, votre bot s’exécute localement. Démarrez à présent l’émulateur, puis connectez-vous à votre bot dans l’émulateur :
1. Tapez http://localhost:port-number/api/messages dans la barre d’adresses, où port-number correspond au numéro de port affiché dans le navigateur dans lequel votre application est en cours d’exécution. Vous pouvez laisser les champs <strong>Microsoft App ID</strong> (ID d’application Microsoft) et <strong>Microsoft App Password</strong> (Mot de passe d’application Microsoft) vides pour l’instant. Vous obtiendrez ces informations ultérieurement lors de l’[inscription du bot](~/bot-service-quickstart-registration.md).
2. Cliquez sur **Connecter**. 
3. Testez votre bot en entrant quelques messages dans l’émulateur. 

## <a name="view-data-in-azure-table-storage"></a>Afficher les données dans le stockage de table Azure
Pour afficher les données d’état, ouvrez l’**Explorateur Stockage** et connectez-vous à Azure à l’aide de vos informations d’identification sur le portail Azure, ou connectez-vous directement à la table en utilisant le nom et la clé de stockage, puis accédez au nom de votre table.  

## <a name="next-steps"></a>Étapes suivantes
Dans cet article, vous avez implémenté le stockage de table Azure pour enregistrer et gérer les données de votre bot. Découvrez maintenant comment modéliser le flux de la conversation avec des dialogues.

> [!div class="nextstepaction"]
> [Gérer le flux de la conversation](bot-builder-dotnet-manage-conversation-flow.md)


## <a name="additional-resources"></a>Ressources supplémentaires

Si vous n’êtes pas familiarisé avec les conteneurs Inversion de contrôle et le modèle d’injection de dépendance utilisés dans le code ci-dessus, consultez le site [Autofac](http://autofac.readthedocs.io/en/latest/) pour plus d’informations. 

Vous pouvez également télécharger un exemple de [stockage de table Azure](https://github.com/Microsoft/BotBuilder-Azure/tree/master/CSharp/Samples/AzureTable) complet à partir de GitHub.
