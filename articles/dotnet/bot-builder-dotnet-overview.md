---
title: Kit SDK Bot Framework pour .NET | Microsoft Docs
description: Démarrez avec le kit SDK Bot Framework pour .NET, framework de création de bots puissant et facile à utiliser.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: f54ea91bbe04f5b9b8a0701a3473ef7e76cacaeb
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/11/2019
ms.locfileid: "54224724"
---
# <a name="bot-framework-sdk-for-net"></a>SDK Bot Framework pour .NET

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-overview.md)
> - [Node.JS](../nodejs/bot-builder-nodejs-overview.md)
> - [REST](../rest-api/bot-framework-rest-overview.md)

Le kit SDK Bot Framework pour .NET est un framework puissant qui permet de créer des bots capables de gérer aussi bien les interactions à structure libre que les conversations plus encadrées dans lesquelles l’utilisateur choisit parmi plusieurs valeurs possibles. Facile d’emploi, il utilise le langage C# pour que les développeurs .NET puissent coder des bots dans un environnement qu'ils connaissent bien.

Le Kit SDK permet de créer des bots qui tirent parti des fonctionnalités suivantes : 

- un système puissant de dialogues isolés et modulables ;
- des invites intégrées pour les opérations simples comme Oui/Non, les chaînes, les nombres et les énumérations ;
- des dialogues prédéfinis qui utilisent de puissantes infrastructures d’intelligence artificielle comme <a href="http://luis.ai" target="_blank">LUIS</a> ;
- FormFlow pour générer automatiquement un bot (à partir d’une classe C#) qui guide l’utilisateur tout au long de la conversation, apportant de l’aide, des informations de navigation, des clarifications et des confirmations selon les besoins.

> [!IMPORTANT]
> Le 31 juillet 2017, des changements cassants ont été implémentés dans le protocole de sécurité de Bot Framework. Pour éviter que ces modifications aient un impact négatif sur votre bot, vérifiez que votre application utilise le kit SDK Bot Framework version 3.5 ou supérieure. Si vous avez créé un bot à l’aide d’un kit SDK obtenu avant le 5 janvier 2017 (date de sortie du kit SDK Bot Framework v3.5), veillez à mettre à niveau votre kit SDK Bot Framework vers la version 3.5 ou ultérieure.

## <a name="get-the-sdk"></a>Obtention du Kit de développement logiciel (SDK)

Le Kit SDK est proposé sous la forme d’un package NuGet et d’un package open source sur <a href="https://github.com/Microsoft/BotBuilder" target="_blank">GitHub</a>.

> [!IMPORTANT]
> Le kit SDK Bot Framework pour .NET nécessite .NET Framework version 4.6 ou plus récente. Si vous ajoutez le kit de développement logiciel à un projet existant destiné à une version inférieure de .NET Framework, vous devrez d’abord mettre à jour votre projet pour qu’il corresponde à la version 4.6 de .NET Framework.

Pour installer le Kit SDK dans un projet Visual Studio, suivez les étapes ci-dessous :

1. Dans **l’Explorateur de solutions**, cliquez avec le bouton droit sur le nom du projet, puis sélectionnez **Gérer les packages NuGet…**.
2. Sur l’onglet **Parcourir**, saisissez « Microsoft.Bot.Builder » dans la zone de recherche.
3. Sélectionnez **Microsoft.Bot.Builder** dans la liste des résultats, cliquez sur **Installer** et acceptez les modifications.

## <a name="get-code-samples"></a>Obtenir des exemples de code

Ce kit SDK comporte un [exemple de code source](bot-builder-dotnet-samples.md) qui utilise les fonctionnalités du kit SDK Bot Framework pour .NET.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur la création de bots à l’aide du kit SDK Bot Framework pour .NET, lisez les articles mentionnés dans cette section, en commençant par :

- [Démarrage rapide](bot-builder-dotnet-quickstart.md) : créez et testez rapidement un bot simple en suivant les instructions de ce tutoriel pas à pas.
- [Concepts clés](bot-builder-dotnet-concepts.md) : découvrez les concepts clés du kit SDK Bot Framework pour .NET.

Si vous rencontrez des problèmes ou avez des suggestions concernant le kit SDK Bot Framework pour .NET, consultez la page [Support](../bot-service-resources-links-help.md) pour obtenir la liste des ressources disponibles. 
