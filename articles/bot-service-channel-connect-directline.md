---
title: Connecter un bot à Direct Line | Microsoft Docs
description: Découvrez comment configurer la connexion d’un bot à Direct Line.
keywords: direct Line, canaux de bots, client personnalisé, connecter à des canaux, configurer
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 10/11/2018
ms.openlocfilehash: 9383e15590569458e795e9a0603df21f63609001
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2018
ms.locfileid: "49997736"
---
# <a name="connect-a-bot-to-direct-line"></a>Connecter un bot à Direct Line

Vous pouvez autoriser votre propre application cliente à communiquer avec votre bot à l’aide du canal Direct Line. 

## <a name="add-the-direct-line-channel"></a>Ajouter le canal Direct Line

Pour ajouter le canal Direct Line, ouvrez le bot dans le [Portail Azure](https://portal.azure.com/), cliquez sur le panneau **Canaux**, puis cliquez sur **Direct Line**.

![Ajouter le canal Direct Line](~/media/bot-service-channel-connect-directline/directline-addchannel.png)

## <a name="add-new-site"></a>Ajouter un nouveau site

Ensuite, ajoutez un nouveau site qui représente l’application cliente que vous souhaitez connecter à votre bot. Cliquez sur **Ajouter un nouveau site**, entrez un nom pour votre site, puis cliquez sur **Terminé**.

![Ajouter un site Direct Line](~/media/bot-service-channel-connect-directline/directline-addsite.png)

## <a name="manage-secret-keys"></a>Gérer les clés secrètes

Quand votre site est créé, Bot Framework génère des clés secrètes que votre application cliente peut utiliser pour [authentifier](~/rest-api/bot-framework-rest-direct-line-3-0-authentication.md) les demandes de l’API Direct Line qu’elle émet pour communiquer avec votre bot. Pour afficher une clé en texte brut, cliquez sur **Afficher** pour la clé correspondante.

![Afficher une clé Direct Line](~/media/bot-service-channel-connect-directline/directline-showkey.png)

Copiez et stockez de manière sécurisée la clé qui s’affiche. Ensuite, utilisez la clé pour [authentifier](~/rest-api/bot-framework-rest-direct-line-3-0-authentication.md) les demandes de l’API Direct Line que votre client émet pour communiquer avec votre bot.
Vous pouvez également utiliser l’API Direct Line pour [échanger la clé contre un jeton](~/rest-api/bot-framework-rest-direct-line-3-0-authentication.md#generate-token) que votre client peut utiliser pour authentifier ses demandes suivantes dans l’étendue d’une même conversation.

![Copier la clé Direct Line](~/media/bot-service-channel-connect-directline/directline-copykey.png)

## <a name="configure-settings"></a>Configurer les paramètres

Enfin, configurez les paramètres du site.

- Sélectionnez la version du protocole Direct Line que votre application cliente doit utiliser pour communiquer avec votre bot.

> [!TIP]
> Si vous créez une connexion entre votre application cliente et votre bot, utilisez l’API Direct Line 3.0.

Quand vous avez terminé, cliquez sur **Terminé** pour enregistrer la configuration du site. Vous pouvez répéter ce processus, en commençant par [Ajouter un nouveau site](#add-new-site), pour chaque application cliente que vous souhaitez connecter à votre bot.
