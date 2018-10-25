---
title: Concepts clés du Kit de développement logiciel (SDK) Bot Builder pour NET | Microsoft Docs
description: Découvrez les concepts et outils clés en matière de création et de déploiement de bots conversationnels qui sont disponibles dans le Kit de développement logiciel (SDK) Bot Builder pour .NET.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 3b01a0e672b2c020462289384ddf68aafedf5314
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2018
ms.locfileid: "49997256"
---
# <a name="key-concepts-in-the-bot-builder-sdk-for-net"></a>Concepts clés du Kit de développement logiciel (SDK) Bot Builder pour .NET

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-concepts.md)
> - [Node.JS](../nodejs/bot-builder-nodejs-concepts.md)

Cet article présente les concepts clés du Kit de développement logiciel (SDK) Bot Builder pour .NET.

## <a name="connector"></a>Connecteur

Le service [Bot Framework Connector](bot-builder-dotnet-connector.md) fournit une API REST unique qui permet à un bot de communiquer sur plusieurs canaux tels que Skype, E-mail, Slack, etc. Il facilite la communication entre bot et utilisateur en relayant les messages du bot au canal et du canal au bot. 

Dans le Kit de développement logiciel Bot Builder pour .NET, la bibliothèque [Connector][connectorLibrary] permet d’accéder au service Connector. 

## <a name="activity"></a>Activité

[!INCLUDE [Activity concept overview](../includes/snippet-dotnet-concept-activity.md)]

Pour plus d’informations sur les activités disponibles dans le Kit de développement logiciel (SDK) Bot Builder pour .NET, consultez l’article [Vue d’ensemble des activités](bot-builder-dotnet-activities.md).

## <a name="dialog"></a>Dialogue

Lorsque vous créez un bot en utilisant le Kit de développement logiciel (SDK) Bot Builder pour .NET, vous pouvez utiliser des [dialogues](bot-builder-dotnet-dialogs.md) pour modéliser une conversation et gérer un [flux de conversation](../bot-service-design-conversation-flow.md#dialog-stack). Un dialogue peut se composer d’autres dialogues afin d’en optimiser la réutilisation, et un contexte de dialogue conserve la [pile des dialogues](../bot-service-design-conversation-flow.md) qui sont actifs dans la conversation à un instant donné. Une conversation constituée de dialogues est portable sur divers ordinateurs, permettant ainsi la mise à l’échelle de l’implémentation de votre bot. 

Dans le Kit de développement logiciel (SDK) Bot Builder pour .NET, la bibliothèque [Builder][builderLibrary] vous permet de gérer les dialogues.

## <a name="formflow"></a>FormFlow

Vous pouvez utiliser [FormFlow](bot-builder-dotnet-formflow.md) dans le Kit de développement logiciel Bot Builder pour .NET afin de rationaliser la génération d’un bot qui collecte des informations auprès de l’utilisateur. Par exemple, un bot prenant des commandes de sandwiches doit recueillir plusieurs types d’informations auprès de l’utilisateur, comme le type de pain, le choix de la garniture, la taille, etc. En fonction des instructions de base, FormFlow peut générer automatiquement les dialogues nécessaires pour gérer une conversation guidée de ce type.

## <a name="state"></a>État

[!INCLUDE [State concept overview](../includes/snippet-dotnet-concept-state.md)]

Pour plus d’informations sur la gestion de l’état à l’aide du Kit de développement logiciel (SDK) Bot Builder pour .NET, consultez l’article [Gérer les données d’état](bot-builder-dotnet-state.md).

## <a name="naming-conventions"></a>Conventions d’affectation de noms

Le Kit de développement logiciel (SDK) Bot Builder pour la bibliothèque .NET utilise des conventions d’affectation de noms en casse Pascal fortement typées. Toutefois, les messages JSON qui sont transportés dans les deux sens du réseau utilisent les conventions d’affectation de noms en casse mixte. Par exemple, la propriété C# **ReplyToId** est sérialisée sous la forme **replyToId** dans le message JSON qui est transporté sur le réseau.

## <a name="security"></a>Sécurité

Vous devez vous assurer que le point de terminaison de votre bot peut uniquement être appelé par le service Bot Framework Connector. Pour plus d’informations sur ce sujet, consultez l’article [Sécuriser votre bot](bot-builder-dotnet-security.md).

## <a name="next-steps"></a>Étapes suivantes

Vous connaissez désormais les concepts sur lesquels reposent tous les bots. Vous pouvez [générer rapidement un bot à l’aide de Visual Studio](bot-builder-dotnet-quickstart.md) en utilisant un modèle. Ensuite, étudiez en détail chacun des concepts clés, en commençant par les dialogues.

> [!div class="nextstepaction"]
> [Dialogues dans le Kit de développement logiciel (SDK) Bot Builder pour .NET](bot-builder-dotnet-dialogs.md)

[connectorLibrary]: /dotnet/api/microsoft.bot.connector

[builderLibrary]: /dotnet/api/microsoft.bot.builder.dialogs
