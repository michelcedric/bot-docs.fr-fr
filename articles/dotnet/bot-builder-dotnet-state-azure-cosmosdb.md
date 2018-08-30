---
title: Gérer les données d’état personnalisé avec Azure Cosmos DB | Microsoft Docs
description: Découvrir comment enregistrer et récupérer des données d’état avec Azure Cosmos DB par le biais du kit SDK Bot Builder pour .NET
author: kaiqb
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: cb64d25582589b7bcbbe715cb4288cf56ac93e1c
ms.sourcegitcommit: 2dc75701b169d822c9499e393439161bc87639d2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/24/2018
ms.locfileid: "42906077"
---
# <a name="manage-custom-state-data-with-azure-cosmos-db-for-net"></a>Gérer les données d’état personnalisé avec Azure Cosmos DB pour .NET

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

Dans cet article, vous implémentez Azure Cosmos DB pour stocker et gérer les données d’état de votre bot. Le service d’état de connecteur par défaut utilisé par les bots n’est pas destiné à l’environnement de production. Vous devez utiliser les [Extensions Azure](https://github.com/Microsoft/BotBuilder-Azure) disponibles sur GitHub ou implémenter un client avec un état personnalisé à l’aide de la plateforme de stockage de données de votre choix. Voici quelques-unes des raisons d’utiliser le stockage d’état personnalisé :
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
 - [Bot Framework Emulator](~/bot-service-debug-emulator.md)
 
## <a name="create-azure-account"></a>Créer un compte Azure
Si vous n’avez pas de compte Azure, cliquez [ici](https://azure.microsoft.com/en-us/free/) pour vous inscrire et bénéficier d’un compte gratuit.

## <a name="set-up-the-azure-cosmos-db-database"></a>Configurer la base de données Azure Cosmos DB
1. Une fois connecté au portail Azure, créez une base de données *Azure Cosmos DB* en cliquant sur **Nouveau**. 
2. Cliquez sur **Bases de données**. 
3. Localisez **Azure Cosmos DB** et cliquez sur **Créer**.
4. Renseignez les champs. Pour le champ **API**, sélectionnez **SQL (DocumentDB)**. Lorsque vous avez renseigné tous les champs, cliquez sur le bouton **Créer** en bas de l’écran pour déployer la nouvelle base de données. 
5. Accédez à votre nouvelle base de données dès que son déploiement est terminé. Cliquez sur **Clés d’accès** pour trouver les clés et les chaînes de connexion. Votre bot utilise ces informations pour appeler le service de stockage et enregistrer les données d’état.

## <a name="install-nuget-packages"></a>Installer les packages NuGet
1. Ouvrez un projet de bot C# existant, ou créez-en un avec le modèle Bot dans Visual Studio. 
2. Installez les packages NuGet suivants :
   - Microsoft.Bot.Builder.Azure
   - Autofac.WebApi2

## <a name="add-connection-string"></a>Ajouter une chaîne de connexion 
Ajoutez les entrées suivantes dans le fichier Web.config :
```XML
<add key="DocumentDbUrl" value="Your DocumentDB URI"/>
<add key="DocumentDbKey" value="Your DocumentDB Key"/>
```
Vous allez remplacer les valeurs par votre URI et votre Clé primaire qui se trouvent dans votre Azure Cosmos DB. Enregistrez le fichier Web.config.

## <a name="modify-your-bot-code"></a>Modifier le code de votre bot
Pour utiliser le stockage **Azure Cosmos DB**, ajoutez les lignes de code suivantes au fichier **Global.asax.cs** de votre bot, dans la méthode **Application_Start()**.

```cs
using System;
using Autofac;
using System.Web.Http;
using System.Configuration;
using Microsoft.Bot.Connector;
using Microsoft.Bot.Builder.Azure;
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Bot.Builder.Dialogs.Internals;

namespace SampleApp
{
    public class WebApiApplication : System.Web.HttpApplication
    {
        protected void Application_Start()
        {
            var uri = new Uri(ConfigurationManager.AppSettings["DocumentDbUrl"]);
            var key = ConfigurationManager.AppSettings["DocumentDbKey"];
            var store = new DocumentDbBotDataStore(uri, key);

            Conversation.UpdateContainer(
                        builder =>
                        {
                            builder.Register(c => store)
                                .Keyed<IBotDataStore<BotData>>(AzureModule.Key_DataStore)
                                .AsSelf()
                                .SingleInstance();

                            builder.Register(c => new CachingBotDataStore(store, CachingBotDataStoreConsistencyPolicy.ETagBasedConsistency))
                                .As<IBotDataStore<BotData>>()
                                .AsSelf()
                                .InstancePerLifetimeScope();

                        });

        }
    }
}
```

Enregistrez le fichier Global.asax.cs. Vous êtes maintenant prêt à tester le bot avec l’émulateur.

## <a name="run-your-bot-app"></a>Exécuter votre application de bot
Exécutez votre bot dans Visual Studio, le code que vous avez ajouté crée la table **botdata** personnalisée dans Azure.

## <a name="connect-your-bot-to-the-emulator"></a>Connecter votre bot à l’émulateur
À ce stade, votre bot s’exécute localement. Démarrez à présent l’émulateur, puis connectez-vous à votre bot dans l’émulateur :
1. Tapez http://localhost:port-number/api/messages dans la barre d’adresses, où port-number correspond au numéro de port affiché dans le navigateur dans lequel votre application est en cours d’exécution. Vous pouvez laisser les champs <strong>Microsoft App ID</strong> (ID d’application Microsoft) et <strong>Microsoft App Password</strong> (Mot de passe d’application Microsoft) vides pour l’instant. Vous obtiendrez ces informations ultérieurement lors de l’[inscription du bot](~/bot-service-quickstart-registration.md).
2. Cliquez sur **Connecter**. 
3. Testez votre bot en entrant quelques messages dans l’émulateur. 

## <a name="view-state-data-on-azure-portal"></a>Afficher les données d’état sur le portail Azure
Pour afficher les données d’état, connectez-vous au portail Azure et accédez à votre base de données. Cliquez sur **Explorateur de données (préversion)** pour vérifier que les informations d’état à partir de votre bot sont en cours d’enregistrement. 

## <a name="next-steps"></a>Étapes suivantes
Dans cet article, vous avez utilisé Cosmos DB pour enregistrer et gérer les données de votre bot. Découvrez maintenant comment modéliser le flux de la conversation avec des dialogues.

> [!div class="nextstepaction"]
> [Gérer le flux de la conversation](bot-builder-dotnet-manage-conversation-flow.md)

## <a name="additional-resources"></a>Ressources supplémentaires
Si vous n’êtes pas familiarisé avec les conteneurs d’inversion de contrôle et le modèle d’injection de dépendance utilisés dans le code ci-dessus, visitez le site [Autofac](http://autofac.readthedocs.io/en/latest/) pour de plus amples informations. 

Vous pouvez également télécharger un [exemple](https://github.com/Microsoft/BotBuilder-Azure/tree/master/CSharp/Samples/DocumentDb) à partir de GitHub pour en savoir plus sur l’utilisation de Cosmos DB dans la gestion de l’état. 
