---
title: Clés Application Insights | Microsoft Docs
description: Découvrez comment obtenir des clés Application Insights pour ajouter des données de télémétrie à un bot.
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: abs
ms.date: 12/13/2017
ms.openlocfilehash: 87cdd27a3a413ae200273090d4a79b5edbc55dc1
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2018
ms.locfileid: "49996906"
---
# <a name="application-insights-keys"></a>Clés Application Insights

Azure **Application Insights** affiche les données relatives à votre application dans une ressource Microsoft Azure. Pour ajouter des données de télémétrie à votre bot, vous avez besoin d’un abonnement Azure et d’une ressource Application Insights créée pour votre bot. À partir de cette ressource, vous pouvez obtenir les trois clés pour configurer votre bot :

1. Clé d’instrumentation
2. ID de l'application
3. Clé API

Cette rubrique vous montre comment créer ces clés Application Insights.

> [!NOTE]
> Pendant le processus de création ou d’inscription du bot, vous avez pu choisir entre *activer* ou *désactiver* **Application Insights**. Si vous l’avez *activé*, votre bot a déjà toutes les clés Application Insights nécessaires. Toutefois, si vous l’avez *désactivé*, vous pouvez suivre les instructions de cette rubrique pour savoir comment créer ces clés manuellement.

## <a name="instrumentation-key"></a>Clé d’instrumentation

Pour obtenir la clé d’instrumentation, effectuez les étapes suivantes :
1. À partir du [portail Azure](http://portal.azure.com), sous la section « Surveiller », créez une ressource **Application Insights** (ou utilisez-en une existante).
![Capture d’écran du portail listant les ressources Application Insights](~/media/portal-app-insights-add-new.png)

2. Dans la liste des ressources Application Insights, cliquez sur celle que vous venez de créer.

3. Cliquez sur **Overview**.

4. Développez le bloc **Bases** et recherchez la **Clé d’instrumentation**. 
![Capture d’écran du portail montrant la vue d’ensemble](~/media/portal-app-insights-instrumentation-key-dropdown.png)
![Capture d’écran du portail montrant la clé d’instrumentation](~/media/portal-app-insights-instrumentation-key.png)

5. Copiez la **Clé d’instrumentation** et collez-la dans le champ **Clé d’instrumentation Application Insights** des paramètres de votre bot.

## <a name="application-id"></a>ID de l'application

Pour obtenir l’ID d’application, effectuez les étapes suivantes :
1. À partir de votre ressource Application Insights, cliquez sur **Accès API**.

2. Copiez **l’ID d’application** et collez-le dans le champ **ID d’application Application Insights** des paramètres de votre bot. 
![Capture d’écran du portail montrant l’ID d’application](~/media/portal-app-insights-appid.png)

## <a name="api-key"></a>Clé API

Pour obtenir la clé API, effectuez les étapes suivantes :
1. À partir de la ressource Application Insights, cliquez sur **Accès API**.

2. Cliquez sur **Créer une clé API**.

3. Entrez une brève description, cochez l’option **Lire les données de télémétrie**, puis cliquez sur le bouton **Générer la clé**.
![Capture d’écran du portail montrant l’ID d’application et la clé API](~/media/portal-app-insights-appid-apikey.png)

   > [!WARNING]
   > Copiez cette **clé API** et enregistrez-la, car vous ne la verrez plus s’afficher. Si vous perdez cette clé, vous devez en créer une autre.

4. Copiez la clé API dans le champ **Clé API Application Insights** des paramètres de votre bot.

## <a name="additional-resources"></a>Ressources supplémentaires
Pour plus d’informations sur la façon de connecter ces champs aux paramètres de votre bot, consultez [Activer l’analytique](~/bot-service-manage-analytics.md#enable-analytics).