---
title: Gérer les données d’état personnalisé avec Azure Cosmos DB | Microsoft Docs
description: Découvrir comment enregistrer et récupérer des données d’état avec Azure Cosmos DB par le biais du kit SDK Bot Builder pour Node.js.
author: DucVo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 0c0d91a7ec9fd1d72c7c51c042b0f52e28798778
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2018
ms.locfileid: "49998116"
---
# <a name="manage-custom-state-data-with-azure-cosmos-db-for-nodejs"></a>Gérer les données d’état personnalisé avec Azure Cosmos DB pour Node.js

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

Dans cet article, vous allez implémenter le stockage Cosmos DB pour stocker et gérer les données d’état de votre bot. La valeur par défaut du service d’état des connecteurs utilisé par les bots n’est pas destinée à l’environnement de production. Vous devez utiliser les [Extensions Azure](https://www.npmjs.com/package/botbuilder-azure) disponibles sur GitHub ou implémenter un client avec un état personnalisé à l’aide de la plateforme de stockage de données de votre choix. Voici quelques-unes des raisons d’utiliser un stockage d’état personnalisé :

- Débit d’API d’état plus élevé (meilleur contrôle des performances)
- Latence inférieure pour la distribution géographique
- Contrôle de l’endroit où les données sont stockées (par exemple, USA Ouest ou USA Est)
- Accès aux données d’état réelles
- Base de données d’état non partagée avec d’autres robots
- Stockage de plus de 32 Ko

## <a name="prerequisites"></a>Prérequis

- [Node.js](https://nodejs.org/en/).
- [Bot Framework Emulator](~/bot-service-debug-emulator.md)
- Vous devez avoir un bot Node.js. Si vous n’en avez pas, accédez à [Créer un bot](bot-builder-nodejs-quickstart.md). 

## <a name="create-azure-account"></a>Créer un compte Azure
Si vous n’avez pas de compte Azure, cliquez [ici](https://azure.microsoft.com/en-us/free/) pour vous inscrire et bénéficier d’un compte gratuit.

## <a name="set-up-the-azure-cosmos-db-database"></a>Configurer la base de données Azure Cosmos DB
1. Une fois connecté au portail Azure, créez une base de données *Azure Cosmos DB* en cliquant sur **Nouveau**. 
2. Cliquez sur **Bases de données**. 
3. Localisez **Azure Cosmos DB** et cliquez sur **Créer**.
4. Renseignez les champs. Pour le champ **API**, sélectionnez **SQL (DocumentDB)**. Lorsque vous avez renseigné tous les champs, cliquez sur le bouton **Créer** en bas de l’écran pour déployer la nouvelle base de données. 
5. Accédez à votre nouvelle base de données dès que son déploiement est terminé. Cliquez sur **Clés d’accès** pour trouver les clés et les chaînes de connexion. Votre bot utilise ces informations pour appeler le service de stockage et enregistrer les données d’état.

## <a name="install-botbuilder-azure-module"></a>Installer le module botbuilder-azure

Pour installer le module `botbuilder-azure` à partir d’une invite de commande, accédez au répertoire du bot, puis exécutez la commande npm suivante :

```nodejs
npm install --save botbuilder-azure
```

## <a name="modify-your-bot-code"></a>Modifier le code de votre bot

Pour utiliser votre base de données **Azure Cosmos DB**, ajoutez les lignes de code suivantes au fichier **app.js** de votre bot.

1. Exigez le module nouvellement installé.

   ```javascript
   var azure = require('botbuilder-azure'); 
   ```

2. Configurez les paramètres de connexion pour se connecter à Azure.
   ```javascript
   var documentDbOptions = {
       host: 'Your-Azure-DocumentDB-URI', 
       masterKey: 'Your-Azure-DocumentDB-Key', 
       database: 'botdocs',   
       collection: 'botdata'
   };
   ```
   Les valeurs `host` et `masterKey` figurent dans le menu **Clés** de votre base de données. Si les entrées `database` et `collection` n’existent pas dans la base de données Azure, elles sont créées pour vous.

3. À l’aide du module `botbuilder-azure`, créez deux nouveaux objets à connecter à la base de données Azure. Tout d’abord, créez une instance de `DocumentDBClient` transmettant les paramètres de configuration de la connexion (définie comme `documentDbOptions` ci-dessus). Ensuite, créez une instance de `AzureBotStorage` transmettant l’objet `DocumentDBClient`. Par exemple : 
   ```javascript
   var docDbClient = new azure.DocumentDbClient(documentDbOptions);

   var cosmosStorage = new azure.AzureBotStorage({ gzipData: false }, docDbClient);
   ```

4. Indiquez que vous souhaitez utiliser votre base de données personnalisée au lieu du stockage en mémoire. Par exemple : 

   ```javascript
   var bot = new builder.UniversalBot(connector, function (session) {
        // ... Bot code ...
   })
   .set('storage', cosmosStorage);
   ```

Vous êtes à présent prêt à tester le robot avec l’émulateur.

## <a name="run-your-bot-app"></a>Exécuter votre application de bot

À partir d’une invite de commandes, accédez au répertoire de votre bot et exécutez celui-ci avec la commande suivante :

```nodejs
node app.js
```

## <a name="connect-your-bot-to-the-emulator"></a>Connecter votre bot à l’émulateur

À ce stade, votre bot s’exécute localement. Démarrez l’émulateur, puis connectez-vous à votre bot à partir de l’émulateur :

1. Tapez <strong>http://localhost:port-number/api/messages</strong> dans la barre d’adresse de l’émulateur, où port-number correspond au numéro de port affiché dans le navigateur où s’exécute votre application. Pour l’instant, vous pouvez laisser vides les champs <strong>ID d’application Microsoft</strong> et <strong>Mot de passe d’application Microsoft</strong>. Vous obtiendrez ces informations ultérieurement lors de l’[inscription du bot](~/bot-service-quickstart-registration.md).
2. Cliquez sur **Connecter**.
3. Testez votre bot en lui envoyant un message. Interagissez avec votre bot comme vous le feriez normalement. Lorsque vous avez terminé, accédez à l’**Explorateur Stockage** et affichez vos données d’état enregistrées.

## <a name="view-state-data-on-azure-portal"></a>Voir les données d’état dans le portail Azure

Pour voir les données d’état, connectez-vous au portail Azure et accédez à votre base de données. Cliquez sur **Explorateur de données (préversion)** pour vérifier que les informations d’état de votre bot sont en cours d’enregistrement.

## <a name="next-step"></a>Étape suivante

À présent que vous contrôlez totalement les données d’état de votre bot, voyons comment les utiliser pour mieux gérer le flux de la conversation.

> [!div class="nextstepaction"]
> [Gérer le flux de la conversation](bot-builder-nodejs-dialog-manage-conversation-flow.md)
