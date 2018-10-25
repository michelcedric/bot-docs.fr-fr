---
title: Kit SDK Bot Builder pour .NET | Microsoft Docs
description: Bien démarrer avec le Kit de développement logiciel (SDK) Bot Builder pour .NET, une infrastructure puissante et facile à utiliser permettant de créer des bots.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 801925e2c179392804d9707e62bfaef082c8e81b
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2018
ms.locfileid: "49996697"
---
# <a name="bot-builder-sdk-for-net"></a>Kit de développement logiciel Bot Builder pour .NET

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-overview.md)
> - [Node.JS](../nodejs/bot-builder-nodejs-overview.md)
> - [REST](../rest-api/bot-framework-rest-overview.md)

Le Kit de développement logiciel (SDK) Bot Builder pour .NET est une infrastructure puissante qui permet de créer des bots capables de gérer aussi bien des interactions à structure libre que des conversations plus encadrées dans lesquelles l'utilisateur choisit parmi plusieurs valeurs possibles. Facile d’emploi, il utilise le langage C# pour que les développeurs .NET puissent coder des bots dans un environnement qu'ils connaissent bien.

Le Kit SDK permet de créer des bots qui tirent parti des fonctionnalités suivantes : 

- un système puissant de dialogues isolés et modulables ;
- des invites intégrées pour les opérations simples comme Oui/Non, les chaînes, les nombres et les énumérations ;
- des dialogues prédéfinis qui utilisent de puissantes infrastructures d’intelligence artificielle comme <a href="http://luis.ai" target="_blank">LUIS</a> ;
- FormFlow pour générer automatiquement un bot (à partir d’une classe C#) qui guide l’utilisateur tout au long de la conversation, apportant de l’aide, des informations de navigation, des clarifications et des confirmations selon les besoins.

> [!IMPORTANT]
> Le 31 juillet 2017, des changements cassants ont été implémentés dans le protocole de sécurité de Bot Framework. Pour éviter que ces modifications aient un impact négatif sur votre bot, vérifiez que votre application utilise au moins la version 3.5 du Kit SDK Bot Builder. Si vous avez créé un bot à l’aide d’un Kit SDK antérieur au 5 janvier 2017 (date de sortie du Kit SDK Bot Builder v3.5), mettez votre Kit SDK Bot Builder à niveau vers la version 3.5 ou une version ultérieure.

## <a name="get-the-sdk"></a>Obtention du Kit de développement logiciel (SDK)

Le Kit SDK est proposé sous la forme d’un package NuGet et d’un package open source sur <a href="https://github.com/Microsoft/BotBuilder" target="_blank">GitHub</a>.

> [!IMPORTANT]
> Pour les besoins du Kit SDK Bot Builder pour .NET, la version 4.6 (ou une version ultérieure) de .NET Framework est requise. Si vous ajoutez le Kit SDK à un projet existant lié à une version antérieure de .NET Framework, vous devrez d'abord mettre à jour votre projet pour qu'il cible la version 4.6 de .NET Framework.

Pour installer le Kit SDK dans un projet Visual Studio, suivez les étapes ci-dessous :

1. Dans **l’Explorateur de solutions**, cliquez avec le bouton droit sur le nom du projet, puis sélectionnez **Gérer les packages NuGet…**.
2. Sur l’onglet **Parcourir**, saisissez « Microsoft.Bot.Builder » dans la zone de recherche.
3. Sélectionnez **Microsoft.Bot.Builder** dans la liste des résultats, cliquez sur **Installer** et acceptez les modifications.

## <a name="get-code-samples"></a>Obtenir des exemples de code

Ce Kit SDK comporte un [exemple de code source](bot-builder-dotnet-samples.md) qui utilise les fonctionnalités du Kit SDK Bot Builder pour .NET.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur la création de bots avec le Kit SDK Bot Builder pour .NET, lisez les articles tout au long de cette section, en commençant par :

- [Démarrage rapide](bot-builder-dotnet-quickstart.md) : créez et testez rapidement un bot simple en suivant les instructions de ce tutoriel pas à pas.
- [Concepts clés](bot-builder-dotnet-concepts.md) : découvrez les concepts clés du Kit SDK Bot Builder pour .NET.

Si vous rencontrez des problèmes ou avez des suggestions concernant le Kit SDK Bot Builder pour .NET, consultez la page [Support](../bot-service-resources-links-help.md) pour connaître la liste des ressources disponibles. 
