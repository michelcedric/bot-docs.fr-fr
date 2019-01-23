---
title: Créer une inscription Bot Channels Registration auprès de Bot Service | Microsoft Docs
description: Découvrez comment inscrire un bot déjà créé auprès de Bot Service.
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: abs
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: a5cb6431988e65a4fa4a889f3095404622d51626
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/11/2019
ms.locfileid: "54224444"
---
# <a name="register-a-bot-with-bot-service"></a>Inscrire un bot auprès de Bot Service



Si votre bot est hébergé ailleurs et que vous souhaitez utiliser Bot Service pour le connecter à d’autres canaux, vous devez l’inscrire auprès de Bot Service. Dans cette rubrique, découvrez comment inscrire votre bot auprès de Bot Service en créant un service de bot **Bot Channels Registration**.

> [!IMPORTANT] 
> L’inscription du bot n’est nécessaire que s’il n’est pas hébergé dans Azure. S’il a été [créé](bot-service-quickstart.md) avec le Portail Azure, il est déjà inscrit auprès de Bot Service.

## <a name="log-in-to-azure"></a>Connexion à Azure
Connectez-vous au [portail Azure](http://portal.azure.com).

> [!TIP]
> Si vous n’avez pas encore d’abonnement, vous pouvez vous inscrire pour un <a href="https://azure.microsoft.com/en-us/free/" target="_blank">compte gratuit</a>.

## <a name="create-a-bot-channels-registration"></a>Créer une inscription Bot Channels Registration
Le service de bot **Bot Channels Registration** est nécessaire pour pouvoir utiliser la fonctionnalité Bot Service. Un bot d’inscription permet de connecter le bot à des canaux.

Pour créer une inscription **Bot Channels Registration**, effectuez les actions suivantes :

1. Cliquez sur le bouton **Nouveau** en haut à gauche du Portail Azure, puis sélectionnez **IA + Cognitive Services > Bot Channels Registration**. 

2. Un nouveau panneau affiche des informations sur **Bot Channels Registration**. Cliquez sur le bouton **Créer** pour lancer le processus de création. 

3. Dans le panneau **Bot Service**, indiquez les informations demandées sur votre bot (voir le tableau sous l’image).  <br/>
   ![Panneau Créer un bot d’inscription](~/media/azure-bot-quickstarts/registration-create-bot-service-blade.png)


   |                    Paramètre                     |         Valeur suggérée         |                                                                                                  Description                                                                                                  |
   |------------------------------------------------|---------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
   |           <strong>Nom du bot</strong>            |     Nom d’affichage du bot     |                                                  Nom complet du bot qui s’affiche dans les canaux et les répertoires. Ce nom est modifiable à tout moment.                                                  |
   |         <strong>Abonnement</strong>          |        Votre abonnement        |                                                                                Sélectionnez l’abonnement Azure à utiliser.                                                                                 |
   |        <strong>Groupe de ressources</strong>         |         myResourceGroup         |                                 Vous pouvez créer un [groupe de ressources](/azure/azure-resource-manager/resource-group-overview#resource-groups) ou en choisir un.                                  |
   |                    Lieu                    |             USA Ouest             |                                                        Choisissez un emplacement proche de celui où est déployé votre bot ou proche d’autres services auxquels il accédera.                                                         |
   |         <strong>Niveau tarifaire</strong>          |               F0                |             Sélectionnez un niveau tarifaire. Vous pourrez mettre à jour le niveau tarifaire à tout moment. Pour plus d’informations, voir [Prix de Bot Service](https://azure.microsoft.com/en-us/pricing/details/bot-service/).              |
   |      <strong>Point de terminaison de messagerie</strong>       |               URL               |                                                                               Entrez l’URL du point de terminaison de messagerie de votre bot.                                                                                |
   |     <strong>Application Insights</strong>      |               Il en va                | <strong>Activez</strong> ou <strong>désactivez</strong> [Application Insights](bot-service-manage-analytics.md). Si vous sélectionnez <strong>Activé</strong>, spécifiez également un emplacement régional. |
   | <strong>ID d'application et mot de passe Microsoft</strong> | Création automatique de l’ID d’application et du mot de passe |              Utilisez cette option si vous voulez entrer manuellement un ID d’application et un mot de passe Microsoft. Sinon, ils seront créés automatiquement au cours du processus de création du bot.               |


4. Cliquez sur **Créer** pour créer le service et inscrire le point de terminaison de messagerie de votre bot.

Vérifiez que l’inscription a été créée en consultant les **Notifications**. La notification passera de **Déploiement en cours…** à **Déploiement réussi**. Cliquez sur le bouton **Accéder à la ressource** pour ouvrir le panneau des ressources du bot. 

## <a name="bot-channels-registration-password"></a>Mot de passe Bot Channels Registration

Le service de bot **Bot Channels Registration** n’est pas associé à un service d’application. C’est pourquoi il a juste un ID *MicrosoftAppID*. Le mot de passe doit être généré et enregistré manuellement. Vous en aurez besoin pour tester votre bot avec [l’émulateur](bot-service-debug-emulator.md).

Pour générer un mot de passe MicrosoftAppPassword, effectuez les étapes suivantes :

1. Dans le panneau **Paramètres**, cliquez sur **Gérer**. Ce lien apparaît à côté **d’ID d’application Microsoft**. Il ouvre une fenêtre permettant de générer un nouveau mot de passe. <br/>
  ![Lien Gérer dans le panneau Paramètres](~/media/azure-bot-quickstarts/registration-settings-manage-link.png)

2. Cliquez sur **Générer un nouveau mot de passe** pour générer un nouveau mot de passe pour votre bot. Copiez-le et enregistrez-le dans un fichier. C’est la seule fois où vous verrez ce mot de passe. Si vous n’enregistrez pas le mot de passe complet, vous devrez répéter le processus de création d’un mot de passe pour la suite. <br/>
  ![Générer un mot de passe d’application Microsoft](~/media/azure-bot-quickstarts/registration-generate-app-password.png)

## <a name="update-the-bot"></a>Mettre à jour le bot

Si vous utilisez le kit SDK Bot Framework pour Node.js, définissez les variables d’environnement suivantes :

* MICROSOFT_APP_ID
* MICROSOFT_APP_PASSWORD

Si vous utilisez le kit SDK Bot Framework pour .NET, définissez les valeurs de clé suivantes dans le fichier web.config :

* MicrosoftAppId
* MicrosoftAppPassword

## <a name="test-the-bot"></a>Tester le bot

Maintenant que votre service de bot est créé, [testez-le dans la Discussion Web](bot-service-manage-test-webchat.md). Entrez un message ; votre bot devrait répondre.

## <a name="next-steps"></a>Étapes suivantes

Dans cette rubrique, vous avez inscrit votre bot hébergé auprès de Bot Service. L’étape suivante consiste à apprendre à gérer votre Bot Service.

> [!div class="nextstepaction"]
> [Gérer un bot](bot-service-manage-overview.md)

