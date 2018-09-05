---
title: Incorporer un bot dans une application | Microsoft Docs
description: Découvrez comment concevoir un bot à incorporer dans une autre application.
author: matvelloso
ms.author: mateusv
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 08/15/2018
ms.openlocfilehash: 68d2d4f7a19aecfcb2c630e5e9e55ca5b8a21d89
ms.sourcegitcommit: e8c513d3af5f0c514cadcbcd0a737a7393405afa
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/20/2018
ms.locfileid: "42756528"
---
# <a name="embed-a-bot-in-an-app"></a>Incorporer un bot dans une application

Bien que les bots existent généralement en dehors des applications, ils peuvent également y être intégrés. Par exemple, vous pouvez incorporer un [bot de connaissance](~/bot-service-design-pattern-knowledge-base.md) au sein d’une application pour permettre aux utilisateurs de trouver des informations qui pourraient être difficiles à rechercher dans des structures d’applications complexes. Vous pouvez incorporer un bot au sein d’une application de support technique, comme premier système de réponse aux demandes utilisateur entrantes. Le bot peut alors résoudre les problèmes simples de façon indépendante puis [transférer](~/bot-service-design-pattern-handoff-human.md) les problèmes plus complexes à un agent humain. 

## <a name="integrating-bot-with-app"></a>Intégration d’un bot au sein d’une application

La manière d’intégrer un bot au sein d’une application varie selon le type d’application. 

### <a name="native-mobile-app"></a>Application mobile native

Une application créée en code natif peut communiquer avec Bot Framework à l’aide de l’[API Direct Line][directLineAPI], via REST ou WebSockets.

### <a name="web-based-mobile-app"></a>Application mobile basée sur le Web

Une application mobile générée à l’aide d’un langage web et d’infrastructures telles que <a href="https://cordova.apache.org/" target="_blank">Cordova</a> peut communiquer avec Bot Framework en utilisant les mêmes composants qu’un [bot incorporé au sein d’un site Web](~/bot-service-design-pattern-embed-web-site.md). Dans ce cas, elle est encapsulée au sein de l’interpréteur de commandes de l’application native.

### <a name="iot-app"></a>Application IoT

Une application IoT peut communiquer avec Bot Framework à l’aide de l’[API Direct Line][directLineAPI]. Dans certains scénarios, elle peut également utiliser <a href="https://www.microsoft.com/cognitive-services/" target="_blank">Microsoft Cognitive Services</a> pour activer des fonctionnalités telles que la reconnaissance d’image et la synthèse vocale.

### <a name="other-types-of-apps-and-games"></a>Autres types d’applications et de jeux

D’autres types d’applications et de jeux peuvent communiquer avec Bot Framework à l’aide de l’[API Direct Line][directLineAPI]. 

## <a name="creating-a-cross-platform-mobile-app-that-runs-a-bot"></a>Création d’une application mobile multiplateforme qui exécute un bot

Cet exemple de création d’une application mobile qui exécute un bot utilise <a href="https://www.xamarin.com/" target="_blank">Xamarin</a>, un outil populaire pour créer des applications mobiles multiplateformes. 

Tout d’abord, créez un composant de vue web simple et utilisez-le pour héberger un <a href="https://github.com/Microsoft/BotFramework-WebChat" target="_blank">contrôle de chat web</a>. Ensuite, à l’aide du Portail Azure, ajoutez le canal Discussion Web. 

Spécifiez ensuite l’URL du chat web inscrit comme source pour le contrôle de la vue web dans l’application Xamarin :

```cs
public class WebPage : ContentPage
{
    public WebPage()
    {
        var browser = new WebView();
        browser.Source = "https://webchat.botframework.com/embed/<YOUR SECRET KEY HERE>";
        this.Content = browser;
    }
}
```

Ce processus vous permet de créer une application mobile multiplateforme qui affiche la vue web incorporée avec le contrôle de chat web.

![Canal arrière](~/media/bot-service-design-pattern-embed-app/xamarin-apps.png)

## <a name="sample-code"></a>Exemple de code

Pour obtenir un exemple complet montrant comment créer une application mobile multiplateforme qui exécute un bot (comme décrit dans cet article), consultez l’<a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/capability-BotInApps" target="_blank">exemple de bot dans des applications</a> dans GitHub.

## <a name="additional-resources"></a>Ressources supplémentaires

- [API Direct Line][directLineAPI]
- <a href="https://www.microsoft.com/cognitive-services/" target="_blank">Microsoft Cognitive Services</a>

[directLineAPI]: https://docs.botframework.com/en-us/restapi/directline3/#navtitle
