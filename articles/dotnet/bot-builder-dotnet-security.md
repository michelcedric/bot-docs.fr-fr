---
title: Sécuriser votre bot | Microsoft Docs
description: Découvrez comment sécuriser votre bot avec l’authentification Bot Framework et HTTPS.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/17
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 455ee87f31e70470d8d118f010c77327347c032d
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/11/2019
ms.locfileid: "54224884"
---
# <a name="secure-your-bot"></a>Sécuriser votre bot

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

Votre bot peut être connecté à de nombreux canaux de communication (Skype, SMS, e-mail, etc.) par le biais du service Bot Framework Connector. Cet article décrit comment sécuriser votre bot avec l’authentification Bot Framework et HTTPS.

## <a name="use-https-and-bot-framework-authentication"></a>Utiliser l’authentification Bot Framework et HTTPS

Pour vous assurer que le point de terminaison de votre bot est accessible uniquement par le service Bot Framework [Connector](bot-builder-dotnet-concepts.md#connector), configurez-le de sorte qu’il utilise seulement HTTPS et activez l’authentification Bot Framework en [inscrivant](~/bot-service-quickstart-registration.md) votre bot de sorte qu’il obtienne son appID et son mot de passe. 

## <a name="configure-authentication-for-your-bot"></a>Configurer l’authentification pour votre bot

Spécifiez l’appID et le mot de passe du bot dans le fichier web.config de votre bot. 

> [!NOTE]
> Pour trouver les paramètres **AppID** et **AppPassword** de votre bot, consultez la rubrique [MicrosoftAppID and MicrosoftAppPassword](~/bot-service-manage-overview.md#microsoftappid-and-microsoftapppassword) (MicrosoftAppID et MicrosoftAppPassword).

```xml
<appSettings>
    <add key="MicrosoftAppId" value="_appIdValue_" />
    <add key="MicrosoftAppPassword" value="_passwordValue_" />
</appSettings>
```

Ensuite, utilisez l’attribut `[BotAuthentication]` pour spécifier des informations d’authentification quand vous créez votre bot à l’aide du kit SDK Bot Framework pour .NET. 

Pour utiliser les informations d’authentification stockées dans le fichier web.config, spécifiez l’attribut `[BotAuthentication]` sans paramètres.

[!code-csharp[Use botAuthentication attribute](../includes/code/dotnet-security.cs#attribute1)]

Pour utiliser d’autres valeurs pour les informations d’authentification, spécifiez l’attribut `[BotAuthentication]` et passez ces valeurs.

[!code-csharp[Use botAuthentication attribute with parameters](../includes/code/dotnet-security.cs#attribute2)]

## <a name="additional-resources"></a>Ressources supplémentaires

- [SDK Bot Framework pour .NET](bot-builder-dotnet-overview.md)
- [Key concepts in the bot Builder SDK for .NET](bot-builder-dotnet-concepts.md) (Concepts clés du Kit de développement logiciel (SDK) Bot Builder pour .NET)
- [Register a bot with the Bot Framework](~/bot-service-quickstart-registration.md) (Inscrire un bot auprès du Bot Framework)
