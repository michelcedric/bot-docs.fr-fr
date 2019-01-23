---
title: Créer un bot avec le service de robot | Microsoft Docs
description: Découvrez comment créer un bot avec le service de robot, un environnement de développement de bots dédié intégré.
keywords: Démarrage rapide, créer un bot, service de robot, bot d’application web
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: abs
ms.date: 01/08/2019
ms.openlocfilehash: da809023338847374715f7576481fc8d17a21ded
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/11/2019
ms.locfileid: "54225394"
---
::: moniker range="azure-bot-service-3.0"

# <a name="create-a-bot-with-azure-bot-service"></a>Créer un bot avec Azure Bot Service

[!INCLUDE [pre-release-label](includes/pre-release-label-v3.md)]

Bot Service fournit les principaux composants permettant de créer des bots, notamment le kit SDK Bot Framework pour le développement de bots et Bot Framework pour la connexion des bots aux canaux. Pour la création de vos robots, Bot Service vous offre un choix de cinq modèles prenant en charge .NET et Node.js. Dans cette rubrique, découvrez comment utiliser Bot Service pour créer un bot qui utilise le kit SDK Bot Framework.

## <a name="log-in-to-azure"></a>Connexion à Azure
Connectez-vous au [portail Azure](http://portal.azure.com).

> [!TIP]
> Si vous n’avez pas encore d’abonnement, vous pouvez vous inscrire pour un <a href="https://azure.microsoft.com/en-us/free/" target="_blank">compte gratuit</a>.

## <a name="create-a-new-bot-service"></a>Créer un service de robot

1. Cliquez sur le lien **Créer une ressource** dans le coin supérieur gauche du Portail Azure, puis sélectionnez **IA + Machine Learning > Web App bot (Web App Bot)**. 

2. Cette opération affiche un nouveau panneau fournissant des informations sur l’offre **Web App Bot**.  

3. Dans le panneau **Bot Service** (Service de robot), indiquez les informations demandées concernant votre bot, comme spécifié dans le tableau figurant sous l’image.  <br/>
   ![Panneau de création d’un bot Web App Bot](./media/azure-bot-quickstarts/sdk-create-bot-service-blade.png)

   | Paramètre | Valeur suggérée | Description |
   | ---- | ---- | ---- |
   | **Nom du bot** | Nom d’affichage du bot | Nom complet du bot qui s’affiche dans les canaux et les répertoires. Ce nom est modifiable à tout moment. |
   | **Abonnement** | Votre abonnement | Sélectionnez l’abonnement Azure à utiliser. |
   | **Groupe de ressources** | myResourceGroup | Vous pouvez créer un [groupe de ressources](/azure/azure-resource-manager/resource-group-overview#resource-groups) ou en choisir un. |
   | **Lieu** | Emplacement par défaut | Sélectionnez l’emplacement géographique de votre groupe de ressources. Vous pouvez choisir n’importe quel emplacement répertorié, même s’il est généralement préférable de sélectionner l’emplacement le plus proche de votre client. L’emplacement n’est plus modifiable une fois le bot créé. |
   | **Niveau tarifaire** | F0 | Sélectionnez un niveau tarifaire. Vous pourrez mettre à jour le niveau tarifaire à tout moment. Pour plus d’informations, consultez la page [Tarification Azure Bot Service](https://azure.microsoft.com/en-us/pricing/details/bot-service/). |
   | **Nom de l’application** | Un nom unique | Nom d’URL unique du bot. Par exemple, si vous nommez votre bot *myawesomebot*, l’URL de votre bot sera `http://myawesomebot.azurewebsites.net`. Ce nom doit uniquement comporter des caractères alphanumériques et des traits de soulignement. Ce champ est limité à 35 caractères. Le nom d’application n’est plus modifiable une fois le bot créé. |
   | **Modèle de bot** | De base | Choisissez **C#** ou **Node.js**, sélectionnez le modèle **De base** pour ce guide de démarrage rapide, puis cliquez sur **Sélectionner**. Le modèle De base crée un bot d’écho. [Apprenez-en davantage](bot-service-concept-templates.md) sur les modèles. |
   | **Plan/emplacement du service d’application** | Votre plan App Service  | Sélectionnez un emplacement de [plan App Service](https://azure.microsoft.com/en-us/pricing/details/app-service/plans/). Vous pouvez choisir n’importe quel emplacement répertorié, même s’il est généralement préférable de sélectionner l’emplacement le plus proche de votre client. (Non disponible pour l’offre Functions Bot.) |
   | **Stockage Azure** | Votre compte de stockage Azure | Vous pouvez créer un compte de stockage de données ou utiliser un compte existant. Par défaut, le bot utilisera [Stockage Table](/azure/storage/common/storage-introduction#table-storage). |
   | **Application Insights** | Il en va | **Activez** ou **désactivez** [Application Insights](/bot-framework/bot-service-manage-analytics). Si vous sélectionnez **Activé**, spécifiez également un emplacement régional. Vous pouvez choisir n’importe quel emplacement répertorié, même s’il est généralement préférable de sélectionner le même emplacement que celui des services du bot. |
   | **ID d’application et mot de passe Microsoft** | Création automatique de l’ID d’application et du mot de passe | Utilisez cette option si vous voulez entrer manuellement un ID d’application et un mot de passe Microsoft. Sinon, ces derniers seront créés automatiquement au cours du processus de création du bot. |

   > [!NOTE]
   > 
   > Si vous créez un bot **Functions Bot**, le champ **Plan App Service/Emplacement** n’apparaît pas. Le champ *Plan d’hébergement* s’affiche à la place. Dans ce cas, choisissez un [plan d’hébergement](bot-service-overview-readme.md#hosting-plans).

4. Cliquez sur **Créer** pour créer le service et déployer le bot dans le cloud. Ce processus peut prendre quelques minutes.

Vérifiez que le bot a été déployé en consultant les **Notifications**. La notification passera de **Déploiement en cours…** à **Déploiement réussi**. Cliquez sur le bouton **Accéder à la ressource** pour ouvrir le panneau des ressources du bot.

 > [!TIP] 
 > Pour des raisons de performances, l’offre **Functions Bot** exécutant des modèles Node.js a déjà été empaquetée à l’aide de l’outil *Azure Functions Pack*. L’outil *Azure Functions Pack* prend tous les modules Node.js et les combine en un seul fichier *.js.
 > Pour plus d’informations, consultez l’article [Azure Functions Pack](https://github.com/Azure/azure-functions-pack) (Azure Functions Pack).
 
## <a name="test-the-bot"></a>Tester le bot
Maintenant que votre bot est créé, testez-le dans la [Discussion Web](bot-service-manage-test-webchat.md). Entrez un message ; votre bot devrait répondre.

## <a name="next-steps"></a>Étapes suivantes

Cet article vous a appris à créer un bot Web App Bot/Functions Bot **De base** en utilisant le service de robot et à vérifier le bon fonctionnement du bot à l’aide du contrôle Discussion Web dans Azure. À présent, découvrez comment gérer votre bot et commencer à utiliser son code source.

> [!div class="nextstepaction"]
> [Gérer un bot](bot-service-manage-overview.md)

::: moniker-end

::: moniker range="azure-bot-service-4.0"

# <a name="create-a-bot-with-azure-bot-service"></a>Créer un bot avec Azure Bot Service

[!INCLUDE [pre-release-label](includes/pre-release-label.md)]

Azure Bot Service fournit les principaux composants permettant de créer des bots, notamment le kit SDK Bot Framework pour le développement de bots et Bot Service pour la connexion des bots aux canaux. Dans cette rubrique, vous allez pouvoir choisir entre le modèle .NET ou Node.js pour créer un bot à l’aide du kit SDK Bot Framework v4.

## <a name="prerequisites"></a>Prérequis
- Compte [Azure](http://portal.azure.com)

### <a name="create-a-new-bot-service"></a>Créer un service de robot

1. Connectez-vous au [portail Azure](http://portal.azure.com/).
1. Cliquez sur le lien **Créer une ressource** dans le coin supérieur gauche du Portail Azure, puis sélectionnez **IA + Machine Learning** > **Web App Bot**. 

![créer un bot](~/media/azure-bot-quickstarts/abs-create-blade.png)

2. Cette opération affiche un *nouveau panneau* fournissant des informations sur l’offre **Web App Bot**.  

3. Dans le panneau **Bot Service** (Service de robot), indiquez les informations demandées concernant votre bot, comme spécifié dans le tableau figurant sous l’image.  <br/>
 ![Panneau de création d’un bot Web App Bot](~/media/azure-bot-quickstarts/sdk-create-bot-service-blade.png)

 | Paramètre | Valeur suggérée | Description |
 | ---- | ---- | ---- |
 | **Nom du bot** | Nom d’affichage du bot | Nom complet du bot qui s’affiche dans les canaux et les répertoires. Ce nom est modifiable à tout moment. |
 | **Abonnement** | Votre abonnement | Sélectionnez l’abonnement Azure à utiliser. |
 | **Groupe de ressources** | myResourceGroup | Vous pouvez créer un [groupe de ressources](/azure/azure-resource-manager/resource-group-overview#resource-groups) ou en choisir un. |
 | **Lieu** | Emplacement par défaut | Sélectionnez l’emplacement géographique de votre groupe de ressources. Vous pouvez choisir n’importe quel emplacement répertorié, même s’il est généralement préférable de sélectionner l’emplacement le plus proche de votre client. L’emplacement n’est plus modifiable une fois le bot créé. |
 | **Niveau tarifaire** | F0 | Sélectionnez un niveau tarifaire. Vous pourrez mettre à jour le niveau tarifaire à tout moment. Pour plus d’informations, consultez la page [Tarification Azure Bot Service](https://azure.microsoft.com/en-us/pricing/details/bot-service/). |
 | **Nom de l’application** | Un nom unique | Nom d’URL unique du bot. Par exemple, si vous nommez votre bot *myawesomebot*, l’URL de votre bot sera `http://myawesomebot.azurewebsites.net`. Ce nom doit uniquement comporter des caractères alphanumériques et des traits de soulignement. Ce champ est limité à 35 caractères. Le nom d’application n’est plus modifiable une fois le bot créé. |
 | **Modèle de bot** | Bot d’écho | Choisissez **SDK v4** (Kit SDK v4). Pour ce guide de démarrage rapide, sélectionnez C# ou Node.js, puis cliquez sur **Sélectionner**.  
 | **Plan/emplacement du service d’application** | Votre plan App Service  | Sélectionnez un emplacement de [plan App Service](https://azure.microsoft.com/en-us/pricing/details/app-service/plans/). Vous pouvez choisir n’importe quel emplacement répertorié, même s’il est généralement préférable de sélectionner le même emplacement que celui des services du bot.Vous pouvez choisir n’importe quel emplacement répertorié, même s’il est généralement préférable de sélectionner le même emplacement que celui des services du bot. |
 | **Stockage Azure** | Votre compte de stockage Azure | Vous pouvez créer un compte de stockage de données ou utiliser un compte existant. Par défaut, le bot utilisera [Stockage Table](/azure/storage/common/storage-introduction#table-storage). |
 | **Application Insights** | Il en va | **Activez** ou **désactivez** [Application Insights](/bot-framework/bot-service-manage-analytics). Si vous sélectionnez **Activé**, spécifiez également un emplacement régional. Vous pouvez choisir n’importe quel emplacement répertorié, même s’il est généralement préférable de sélectionner le même emplacement que celui des services du bot.Vous pouvez choisir n’importe quel emplacement répertorié, même s’il est généralement préférable de sélectionner le même emplacement que celui des services du bot. |
 | **ID d’application et mot de passe Microsoft** | Création automatique de l’ID d’application et du mot de passe | Utilisez cette option si vous voulez entrer manuellement un ID d’application et un mot de passe Microsoft. Sinon, ces derniers seront créés automatiquement au cours du processus de création du bot. |

4. Cliquez sur **Créer** pour créer le service et déployer le bot dans le cloud. Ce processus peut prendre quelques minutes.

Vérifiez que le bot a été déployé en consultant les **Notifications**. La notification passera de **Déploiement en cours…** à **Déploiement réussi**. Cliquez sur le bouton **Accéder à la ressource** pour ouvrir le panneau des ressources du bot.

Maintenant que votre bot est créé, testez-le dans la discussion web. 

## <a name="test-the-bot"></a>Tester le bot
Dans la section **Gestion du bot**, cliquez sur **Tester dans la Discussion Web**. Azure Bot Service charge le contrôle Discussion Web et se connecte à votre bot. 

![Test de discussion web Azure](./media/azure-bot-quickstarts/azure-webchat-test.png)

Entrez un message ; votre bot devrait répondre.

## <a name="download-code"></a>Télécharger le code
Vous pouvez télécharger le code pour l’utiliser localement. 
1. Dans la section **Gestion du bot**, cliquez sur **Créer**. 
1. Cliquez sur le lien **Download Bot source code** (Télécharger le code source du bot) dans le volet droit. 
1. Suivez les invites pour télécharger le code, puis décompressez le dossier.

Le code que vous avez téléchargé utilise un [fichier .bot](./v4sdk/bot-file-basics.md) chiffré. Vous devrez mettre à jour les entrées `botFilePath` et `botFileSecret` dans le fichier appsettings.json ou .env. 
Pour ce faire, accédez au portail Azure. Sélectionnez votre bot dans le portail, puis sous **Paramètres App Service**, cliquez sur **Paramètres de l’application**. Dans le volet **Paramètres de l’application**, vous voyez les valeurs `botFilePath` et `botFileSecret`. Copiez ces valeurs et mettez à jour le fichier .env ou appsettings.json. 

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous avez appris à créer un bot d’application web **Écho** en utilisant Azure Bot Service et à vérifier le bon fonctionnement du bot à l’aide du contrôle Discussion Web. À présent, découvrez comment gérer votre bot et commencer à utiliser son code source.

> [!div class="nextstepaction"]
> [Fonctionnement des bots](~/v4sdk/bot-builder-basics.md)

::: moniker-end
