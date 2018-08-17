---
title: Intégrer votre bot dans un navigateur web | Microsoft Docs
description: Découvrez comment concevoir une transition d’utilisateur continue du bot vers le navigateur web et inversement.
author: matvelloso
ms.author: mateusv
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: 7f3a6ace5e3ef8122cca32baf8ec4c9a1f250dfa
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39299184"
---
# <a name="integrate-your-bot-with-a-web-browser"></a>Intégrer votre bot dans un navigateur web

Dans certains scénarios, il faut plus qu’un bot pour satisfaire à une exigence. Un bot peut devoir envoyer l’utilisateur vers un navigateur web pour effectuer une tâche, puis reprendre la conversation avec lui une fois la tâche terminée. 

## <a name="authentication-and-authorization"></a>Authentification et autorisation
Si un bot souhaite pouvoir lire le calendrier de l’utilisateur dans Office 365, ou peut-être même créer des rendez-vous au nom de cet utilisateur, ce dernier doit tout d’abord s’authentifier auprès de Microsoft Azure Active Directory et autoriser le bot à accéder aux données de calendrier de l’utilisateur. Le bot redirigera l’utilisateur vers un navigateur web pour effectuer les tâches d’authentification et d’autorisation, puis reprendra la conversation avec l’utilisateur. 

## <a name="security-and-compliance"></a>Sécurité et conformité
Les exigences de conformité et de sécurité limitent souvent le type d’informations qu’un bot peut échanger avec un utilisateur. Dans certains cas, il peut être nécessaire pour l’utilisateur d’envoyer/de recevoir des données en dehors de la conversation actuelle. Par exemple, si un utilisateur souhaite effectuer un paiement à l’aide d’un fournisseur de paiement tiers, un numéro de carte de crédit ne devrait pas être spécifié dans le contexte de la conversation. Au lieu de cela, le bot dirigera l’utilisateur vers un navigateur web pour effectuer le processus de paiement, puis reprendra la conversation avec l’utilisateur.

Cet article explore le processus qui facilite la transition d’un utilisateur d’un bot vers le navigateur web et inversement. 

> [!NOTE]
> Une transition de la conversation vers le navigateur web et inversement n’est pas idéale, car le passage d’une application à une autre peut facilement perturber un utilisateur. Pour fournir une meilleure expérience, plusieurs canaux sont dotés de fenêtres HTML intégrées qu’un bot peut utiliser pour présenter des applications qui, sinon, s’afficheraient dans un navigateur web. Cette technique permet à l’utilisateur de rester dans la conversation tout en ayant encore accès à des ressources externes. Cette approche est conceptuellement semblable aux flux d’autorisation de gestion d’applications mobiles à l’aide d’OAuth dans des vues web incorporées.

## <a name="bot-to-web-browser-and-back-again"></a>Du bot au navigateur web et inversement

Ce diagramme illustre le flux de haut niveau pour l’intégration entre un bot et un navigateur web. 

![Interaction entre le bot et le web](~/media/bot-service-design-pattern-integrate-browser/bot-to-web1.png)

Prenez en compte chaque étape du flux :

1. <a id="generate-hyperlink"></a>Le bot génère et affiche un lien hypertexte qui redirigera l’utilisateur vers un site web. 
   Le lien hypertexte inclut généralement des données par le biais de paramètres de chaîne de requête sur l’URL cible, qui spécifient des informations sur le contexte de la conversation actuelle, telles que l’ID de conversation, l’ID de canal et l’ID d’utilisateur dans le canal. 

2. L’utilisateur clique sur le lien hypertexte et est redirigé vers l’URL cible au sein d’un navigateur web. 

3. Le bot passe à un état d’attente de communication depuis le site web pour indiquer que le flux du site Web est terminé.  
   > [!TIP]
   > Concevez ce flux de conception de telle manière que le bot ne reste pas définitivement dans l’état « en attente » si l’utilisateur ne termine jamais le flux du site web. En d’autres termes, si l’utilisateur abandonne le navigateur web et commence à nouveau à communiquer avec le bot, ce dernier doit reconnaître, et non [ignorer](~/bot-service-design-navigation.md#the-mysterious-bot), cette entrée.

4. L’utilisateur termine la ou les tâches nécessaires via le navigateur web. 
   Il peut s’agir d’un flux OAuth ou de n’importe quelle séquence d’événements requis par le scénario disponible. 

5. <a id="generate-magic-number"></a>Quand l’utilisateur termine le flux du site web, le site web génère un « [nombre magique](#verify-identity) » qu’il demande à l’utilisateur de copier et de coller dans la conversation avec le bot. 

6. <a id="signal-to-bot"></a>Le site web [signale au bot](#website-signal-to-bot) que l’utilisateur a terminé le flux du site web. 
   Il communique le « nombre magique » au bot et fournit toutes les autres données pertinentes.
   Par exemple, dans le cas d’un flux OAuth, le site web fournit un jeton d’accès au bot.

7. L’utilisateur retourne au bot et colle le « nombre magique » dans la conversation. 
   Le bot confirme que le « nombre magique » fourni par l’utilisateur correspond à la valeur attendue et vérifie que l’utilisateur actuel est bien le même que celui qui a préalablement cliqué sur le lien hypertexte pour initier le flux de site web. 

### <a id="verify-identity"></a> Vérification de l’identité de l’utilisateur à l’aide du « nombre magique »

La génération d’un « nombre magique » pendant le flux du bot vers le site web ([étape 5](#generate-magic-number) ci-dessus) permet au bot de vérifier ensuite que l’utilisateur qui a initié le flux du site web est effectivement l’utilisateur auquel il était destiné. Par exemple, si un bot dirige une conversation de groupe avec plusieurs utilisateurs, n’importe lequel d’entre eux peut avoir cliqué sur le lien hypertexte afin d’initier le flux de site web. Sans le processus de validation par « nombre magique », le bot n’a aucun moyen de savoir quel utilisateur a terminé le flux. Un utilisateur pourrait s’authentifier et injecter des jetons d’accès dans la session d’un autre utilisateur. 

> [!WARNING] 
> Ce risque n’existe pas que dans les conversations de groupe. Sans le processus de validation par « nombre magique », toute personne qui obtient le lien hypertexte pour lancer le flux du site web peut usurper l’identité d’un utilisateur. 

Le nombre magique doit être un nombre aléatoire généré à l’aide d’une bibliothèque de chiffrement fort. Pour voir un exemple du processus de génération en C#, consultez <a href="https://github.com/MicrosoftDX/botauth/tree/master/CSharp" target="_blank">ce code</a> au sein de la bibliothèque <a href="https://www.nuget.org/packages/BotAuth" target="_blank">BotAuth</a>. BotAuth permet aux bots créés dans Microsoft Bot Framework d’implémenter le flux du bot vers le site web pour authentifier un utilisateur dans un site web, puis d’utiliser le jeton d’accès qui a été généré à partir du processus d’authentification. Étant donné que BotAuth ne fait pas d’hypothèses concernant les fonctionnalités du canal, ces flux devraient fonctionner correctement avec la plupart des canaux. 

> [!NOTE]
> Le processus de validation par « nombre magique » devrait être déconseillé, car les canaux créent leurs propres vues web incorporées.

### <a id="website-signal-to-bot"></a> Comment le site web « signale-t-il » le bot ?

Lorsque le bot [génère le lien hypertexte](#generate-hyperlink) sur lequel l’utilisateur cliquera pour initier le flux du site web, il contient, par le biais de paramètres de chaîne de requête dans l’URL cible, des informations sur le contexte de la conversation actuelle, telles que l’ID de conversation, l’ID de canal et l’ID d’utilisateur dans le canal. Le site web peut ensuite utiliser ces informations afin de lire et d’écrire des variables d’état pour cet utilisateur ou cette conversation à l’aide du kit de développement logiciel (SDK) Bot Builder ou des API REST. Consultez l’[étape 6](#signal-to-bot) ci-dessus pour avoir un exemple de la manière dont le site web « signale » au bot que le flux du site Web est terminé.

## <a name="sample-code"></a>Exemple de code

Comme il est décrit dans cet article, la bibliothèque <a href="https://github.com/MicrosoftDX/botauth" target="_blank">BotAuth</a> permet aux flux OAuth d’être liés à des bots créés à l’aide de .NET et de Node dans Microsoft Bot Framework.

## <a name="additional-resources"></a>Ressources supplémentaires

- [Dialogues](~/dotnet/bot-builder-dotnet-dialogs.md)
- [Gérer un flux de conversation avec des dialogues (.NET)](~/dotnet/bot-builder-dotnet-manage-conversation-flow.md)
- [Gérer un flux de conversation avec des dialogues (Node.js)](~/nodejs/bot-builder-nodejs-manage-conversation-flow.md)
