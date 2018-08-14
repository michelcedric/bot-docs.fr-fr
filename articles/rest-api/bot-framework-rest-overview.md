---
title: API REST Bot Builder | Microsoft Docs
description: Commencez avec les API REST Bot Framework permettant des créer des robots et des clients qui se connectent à des robots.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: d6d83edb390933cb8895b26efeb9775cafdb1acb
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39299340"
---
# <a name="bot-framework-rest-apis"></a>API REST Bot Framework
> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-overview.md)
> - [Node.JS](../nodejs/bot-builder-nodejs-overview.md)
> - [REST](../rest-api/bot-framework-rest-overview.md)

Les API REST Bot Framework vous permettent de créer des robots qui échangent des messages avec des canaux configurés dans le <a href="https://dev.botframework.com/" target="_blank">Portail Bot Framework</a>, stockent et récupèrent des données d’état, et connectent vos propres applications clientes à vos robots. Tous les services Bot Framework utilisent les ressources REST et JSON standard sur HTTPS.

## <a name="build-a-bot"></a>Créez un robot

Vous pouvez créer un robot avec n’importe quel langage de programmation en utilisant le service Bot Connector pour échanger des messages avec des canaux configurés dans le portail Bot Framework. 

> [!TIP]
> Le Bot Framework fournit des bibliothèques clientes utilisables pour générer des robots en C# ou Node.js. Pour générer un robot à l’aide de C#, utilisez le [Kit de développement logiciel (SDK) Bot Builder pour C#](../dotnet/bot-builder-dotnet-overview.md). Pour générer un robot à l’aide de Node.js, utilisez le [Kit de développement logiciel (SDK) Bot Builder pour Node.js](../nodejs/index.md). 

Pour en savoir plus sur la création de robots à l’aide du service Bot Connector, voir [Concepts clés](bot-framework-rest-connector-concepts.md).

## <a name="build-a-client"></a>Créer un client

Vous pouvez autoriser votre propre application cliente à communiquer avec votre robot à l’aide de l’API Direct Line. L’API Direct Line implémente un mécanisme d’authentification qui utilise des modèles de secret/jeton standard et fournit un schéma stable, même si votre robot change sa version de protocole. Pour en savoir plus sur l’utilisation de l’API Direct Line pour permettre la communication entre un client et votre robot, voir [Concepts clés](bot-framework-rest-direct-line-3-0-concepts.md). 