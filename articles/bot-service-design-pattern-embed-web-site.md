---
title: Incorporer un bot dans un site web | Microsoft Docs
description: Découvrez comment concevoir un bot à incorporer dans un site web.
author: matvelloso
ms.author: mateusv
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: 8f50c54c0841db5778c7966e30ec33f89938b376
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39299168"
---
# <a name="embed-a-bot-in-a-website"></a>Incorporer un bot dans un site web

Même si les bots se trouvent généralement en dehors des sites web, ils peuvent également être incorporés au sein d’un site web. Par exemple, vous pouvez incorporer un [bot de connaissance](~/bot-service-design-pattern-knowledge-base.md) au sein d’un site web pour permettre aux utilisateurs de trouver rapidement des informations qui pourraient être difficiles à rechercher dans des structures de site web complexes. Vous pouvez également incorporer un bot au sein d’un site web de support technique pour jouer le rôle de répondeur de première ligne pour les requêtes utilisateur entrantes. Le bot peut alors résoudre les problèmes simples de façon indépendante, puis [transférer](~/bot-service-design-pattern-handoff-human.md) les problèmes plus complexes à un agent humain. 

Cet article explore l’intégration de bots aux sites web et le processus d’utilisation du mécanisme de *backchannel* pour faciliter la communication privée entre une page web et un bot. 

Microsoft offre deux façons d’intégrer un bot à un site web : le contrôle web Skype et un contrôle web open source.

## <a name="skype-web-control"></a>Contrôle web Skype

Le contrôle web Skype est essentiellement un client Skype dans un contrôle web. L’authentification Skype intégrée permet au bot d’authentifier et de reconnaître les utilisateurs sans nécessiter l’écriture de code personnalisé de la part du développeur. Skype reconnaît automatiquement les comptes Microsoft utilisés dans son client web. 

Étant donné que le contrôle web Skype joue simplement le rôle de frontal pour Skype, le client Skype de l’utilisateur a automatiquement accès au contexte complet de toute conversation facilitée par le contrôle web. Même après la fermeture du navigateur web, l’utilisateur peut continuer à interagir avec le bot à l’aide du client Skype. 

## <a name="open-source-web-control"></a>Contrôle web open source

Le <a href="https://github.com/Microsoft/BotFramework-WebChat" target="_blank">contrôle de discussion web open source</a> repose sur ReactJS et utilise [l’API Direct Line][directLineAPI] pour communiquer avec le Bot Framework. Le contrôle de discussion web fournit un canevas vide pour l’implémentation de la discussion web, ce qui vous donne un contrôle total sur ses comportements et l’expérience utilisateur qu’il fournit. 

Grâce au mécanisme *backchannel*, la page web qui héberge le contrôle peut communiquer directement avec le bot, et ce, de façon totalement invisible pour l’utilisateur. Cette fonctionnalité ouvre la voie à de nombreux scénarios utiles : 

- La page web peut envoyer des données pertinentes au bot (par exemple, un emplacement GPS).
- La page web peut informer le bot sur les actions de l’utilisateur (par exemple, « l’utilisateur vient de sélectionner l’option A dans le menu déroulant »).
- La page web peut envoyer au bot le jeton d’authentification d’un utilisateur connecté.
- Le bot peut envoyer des données pertinentes à la page web (par exemple, la valeur actuelle du portefeuille de l’utilisateur).
- Le bot peut envoyer des « commandes » à la page web (par exemple, la modification de la couleur d’arrière-plan).

## <a name="using-the-backchannel-mechanism"></a>Utilisation du mécanisme de backchannel

[!INCLUDE [Introduction to backchannel mechanism](~/includes/snippet-backchannel.md)]

## <a name="sample-code"></a>Exemple de code

Le <a href="https://github.com/Microsoft/BotFramework-WebChat" target="_blank">contrôle de discussion web open source</a> est disponible dans GitHub. Pour plus d’informations sur la manière d’implémenter le mécanisme de backchannel à l’aide du contrôle de discussion web open source et du Kit de développement logiciel (SDK) Bot Builder pour Node.js, consultez [Utiliser le mécanisme de backchannel](~/nodejs/bot-builder-nodejs-backchannel.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [API Direct Line][directLineAPI]
- [TODO](~/dotnet/bot-builder-dotnet-activities.md)
- [Utiliser le mécanisme de backchannel](~/nodejs/bot-builder-nodejs-backchannel.md)

[directLineAPI]: https://docs.botframework.com/en-us/restapi/directline3/#navtitle
