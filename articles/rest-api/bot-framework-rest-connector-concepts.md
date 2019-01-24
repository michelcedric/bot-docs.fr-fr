---
title: Concepts clés des services Bot Connector et Bot State | Microsoft Docs
description: Comprenez les concepts clés des services Bot Connector et Bot State de Bot Framework.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 01/16/2019
ms.openlocfilehash: 7464e6f19ac1cd1a5744af845bd62c3a48cd2eb8
ms.sourcegitcommit: c6ce4c42fc56ce1e12b45358d2c747fb77eb74e2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/22/2019
ms.locfileid: "54453823"
---
# <a name="key-concepts"></a>Concepts clés

Vous pouvez utiliser les services Bot Connector et Bot State pour communiquer avec les utilisateurs via plusieurs canaux tels que Skype, Courrier, Slack, etc. Cet article présente les concepts clés des services Bot Connector et Bot State.

> [!IMPORTANT]
> L’API du service Bot Framework State n’est pas recommandée pour les environnements de production et pourra être déconseillée dans une version ultérieure. Nous vous recommandons de mettre à jour le code de votre bot pour qu’il utilise le stockage en mémoire à des fins de test ou pour qu’il utilise l’une des **extensions Azure** pour les bots de production. Pour plus d’informations, voir l’article **Gérer les données d’état** pour l’implémentation [.NET](~/dotnet/bot-builder-dotnet-state.md) ou [Node](~/nodejs/bot-builder-nodejs-state.md).

## <a name="bot-connector-service"></a>Service Bot Connector

Le service Bot Connector permet à votre robot d’échanger des messages avec des canaux configurés dans le <a href="https://dev.botframework.com/" target="_blank">Portail Bot Framework</a>. Il utilise des ressources REST et JSON standard sur HTTPS, et permet l’authentification avec des jetons de support JWT. Pour plus d’informations sur l’utilisation du service Bot Connector, voir [ Authentification](bot-framework-rest-connector-authentication.md) et les autres articles de cette section.

### <a name="activity"></a>Activité

Le service Bot Connector échange des informations entre robot et canal (utilisateur) en transmettant un objet [Activité][Activity]. Le type d’activité le plus courant est **message**, mais il existe d’autres types d’activité utilisables pour communiquer différents types d’informations à un robot ou à un canal. Pour plus d’informations sur les activités dans le service Bot Connector, voir [Vue d’ensemble des activités](bot-framework-rest-connector-activities.md).

## <a name="bot-state-service"></a>Service Bot State

Le service Bot State permet à votre robot de stocker et récupérer les données d’état qui sont associées à un utilisateur, à une conversation ou à un utilisateur spécifique dans le contexte d’une conversation donnée. Il utilise des ressources REST et JSON standard sur HTTPS, et permet l’authentification avec des jetons de support JWT. Toutes les données que vous enregistrez en utilisant le service Bot State seront stockées dans Azure et chiffrées au repos.

Le service Bot State n’est utile que conjointement avec le service Bot Connector. En d’autres termes, il ne peut être utilisé que pour stocker des données d’état liées à des conversations que votre robot conduit à l’aide du service Bot Connector. Pour plus d’informations sur l’utilisation du service Bot State, voir [Authentification](bot-framework-rest-connector-authentication.md) et [Gérer les données d’état](bot-framework-rest-state.md).

## <a name="authentication"></a>Authentification

Les services Bot Connector et Bot State permettent l’authentification à l’aide de jetons de support JWT. Pour plus d’informations sur l’authentification des demandes sortantes que votre robot envoie à Bot Framework, sur l’authentification des demandes entrantes que votre robot reçoit de Bot Framework, et bien plus, voir [ Authentification ](bot-framework-rest-connector-authentication.md). 

## <a name="client-libraries"></a>Bibliothèques clientes

Le Bot Framework fournit des bibliothèques clientes utilisables pour générer des robots en C# ou Node.js. 

- Pour créer un bot en C#, utilisez le kit [SDK Bot Framework pour C#](../dotnet/bot-builder-dotnet-overview.md). 
- Pour créer un bot en utilisant Node.js, utilisez le kit [SDK Bot Framework pour Node.js](../nodejs/index.md). 

En plus de la modélisation des services Bot Connector et Bot State, chaque kit SDK Bot Framework propose aussi un système puissant capable de générer des dialogues qui encapsulent une logique conversationnelle, des invites intégrées pour des choses aussi simples que Oui ou Non, des chaînes, des nombres, des énumérations, la prise en charge intégrée de puissantes infrastructures d’intelligence artificielle telles que <a href="https://www.luis.ai/" target="_blank"> LUIS </a>, et bien plus. 

> [!NOTE]
> Au lieu d’utiliser les kits de développement logiciel (SDK) C# ou Node.js, vous pouvez générer votre propre bibliothèque cliente dans la langue de votre choix en utilisant le <a href="https://aka.ms/connector-swagger-file" target="_blank">fichier Swagger de Bot Connector</a> et le <a href="https://aka.ms/state-swagger-file" target="_blank">fichier Swagger de Bot State</a>.

## <a name="additional-resources"></a>Ressources supplémentaires

Pour en savoir plus sur la création de robots à l’aide des services Bot Connector et Bot State, lisez les articles de cette section, à commencer par [Authentification](bot-framework-rest-connector-authentication.md). Si vous rencontrez des problèmes ou avez des suggestions concernant les services Bot Connector ou Bot State, consultez la page [Support](../bot-service-resources-links-help.md) pour accéder à la liste des ressources disponibles. 

[Activity]: bot-framework-rest-connector-api-reference.md#activity-object