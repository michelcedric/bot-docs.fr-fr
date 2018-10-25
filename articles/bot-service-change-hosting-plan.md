---
title: Migrer un bot C# d’un plan de consommation à un plan App Service | Microsoft Docs
description: Migrez un bot C# Bot Service d’un plan d’hébergement de consommation à un plan d’hébergement App Service.
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: abs
ms.date: 12/13/2017
ms.openlocfilehash: aafbfb2a38e2d5370cb2db5721dd7bc130497d74
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2018
ms.locfileid: "49999216"
---
# <a name="change-the-hosting-plan-for-your-bot-service"></a>Modifier le plan d’hébergement d’un service de bot

Cette rubrique explique comment migrer un bot de script C# avec un plan de consommation en un bot C# avec un plan App Service. 

## <a name="advantages-of-a-bot-on-an-app-service-plan"></a>Avantages des bots de type plan App Service

Les bots de type plan App Service s’exécutent en tant qu’applications web Azure. Ils peuvent effectuer des opérations auxquelles les bots de type plan de consommation n’ont pas accès :

- Un bot d’application web peut ajouter des définitions d’itinéraires personnalisés.
- Un bot d’application web peut activer un serveur WebSocket. 
- Un bot qui utilise un plan d’hébergement de consommation a les mêmes limitations que tout le code exécuté sur Azure Functions. Pour plus d’informations, voir <a target='_blank' href='/azure/azure-functions/functions-scale'>Plans de consommation Azure Functions et plans App Service</a>.

## <a name="download-your-existing-bot-source"></a>Télécharger le code source du bot

Suivez ces étapes pour télécharger le code source de votre bot déjà créé :

1. Dans votre bot Azure, cliquez sur l’onglet **Paramètres** et développez la section **Déploiement continu**.  
2. Cliquez sur le bouton bleu pour télécharger le fichier zip contenant le code source de votre bot.  
    ![Télécharger le fichier zip du bot](~/media/continuous-deployment-consumption-download.png)
3. Extrayez le contenu du fichier zip téléchargé dans un dossier local. 


## <a name="create-a-bot-template"></a>Créer un modèle de bot

Bot Service a les mêmes modèles pour tous les bots quel que soit leur plan (plan de consommation ou plan App Service). Pour migrer un bot de type plan de consommation, créez un bot de type plan App Service dans Bot Service sur le même modèle. Si le code sous-jacent peut différer entre les deux types d’hébergement pour un même modèle, la nouvelle application web a une structure et une configuration similaires à celles du bot initial.

## <a name="download-the-new-bot-source"></a>Télécharger le code source du nouveau bot

Suivez ces étapes pour télécharger le code source du nouveau bot :

1. Dans votre bot Azure, cliquez sur l’onglet **BUILD**, recherchez la section **Télécharger le code source**, puis cliquez sur **Télécharger le fichier zip**. 
2. Extrayez le contenu du fichier zip téléchargé dans le dossier local.

## <a name="add-source-files-to-new-solution"></a>Ajouter des fichiers sources à la nouvelle solution

Certains fichiers .csx sont susceptibles de se compiler et s’exécuter en tant que fichiers .cs dans la nouvelle solution. Créez un fichier .cs pour chaque fichier .csx de votre solution, sauf `run.csx`. Vous migrerez la logique `run.csx` manuellement. Dans vos fichiers .cs, vous devrez peut-être ajouter une déclaration de classe et une déclaration d’espace de noms facultative.

## <a name="migrate-runcsx-logic-into-your-project"></a>Migrer la logique run.csx dans un projet

Les projets de script C# ont une méthode `Run` dans laquelle différentes valeurs `ActivityTypes` sont gérées. Importer la logique de gestion des activités dans la méthode `MessageController.Post`, dans `MessageController.cs`.

## <a name="remove-compiler-keywords"></a>Supprimer des mots clés du compilateur

Les fichiers de script C# sont susceptibles d’intégrer un module de référence avec le mot clé `#r`. Supprimez ces lignes et ajoutez-les par ailleurs comme références à votre projet Visual Studio. Supprimez également les mots clés `#load`, qui insèrent d’autres fichiers de code source dans une compilation de fichiers. Ajoutez plutôt tous les fichiers `.csx` à votre projet comme code source `.cs`.

## <a name="add-references-from-projectjson"></a>Ajouter des références à partir de project.json

Si votre bot de type plan de consommation ajoute des références NuGet dans son fichier `project.json`, ajoutez ces références à votre nouvelle solution Visual Studio en cliquant avec le bouton droit sur le projet dans le volet Explorateur de solutions, puis en cliquant sur **Ajouter une référence**.

### <a name="add-references-that-were-implicit"></a>Ajouter des références qui étaient implicites

Un bot Bot Service de type plan de consommation inclut implicitement ces références dans tous les fichiers sources .csx. Lorsque le code source a été migré dans des fichiers sources .cs, il peut être nécessaire d’ajouter des références explicites à ces classes :

- `TraceWriter` dans le package NuGet `Microsoft.Azure.WebJobs` qui fournit les types de l’espace de noms `Microsoft.Azure.WebJobs.Host`. 
- Déclencheurs de minuteur dans le package NuGet `Microsoft.Azure.WebJobs.Extensions`
- `Newtonsoft.Json`, `Microsoft.ServiceBus` et d’autres assemblys référencés automatiquement
- `System.Threading.Tasks` et d’autres espaces de noms importés automatiquement

Pour obtenir de l’aide, voir *Convertir en fichiers de classe* dans <a target='_blank' href='https://blogs.msdn.microsoft.com/appserviceteam/2017/03/16/publishing-a-net-class-library-as-a-function-app/'>Publier une bibliothèque de classes .NET comme application de fonction</a>.

## <a name="debug-your-new-bot"></a>Déboguer le nouveau bot

Un bot de type plan App Service est beaucoup plus facile à déboguer localement qu’un bot de type plan de consommation. Vous pouvez utiliser [l’émulateur](bot-service-debug-emulator.md) pour déboguer localement votre code migré.

## <a name="publish-from-visual-studio-or-set-up-continuous-deployment"></a>Publier à partir de Visual Studio ou définir le déploiement continu

Enfin, publiez votre code source migré sur Bot Service soit en important son fichier `.PublishSettings` et en cliquant sur **Publier**, soit en [configurant le déploiement continu](bot-service-debug-bot.md).
