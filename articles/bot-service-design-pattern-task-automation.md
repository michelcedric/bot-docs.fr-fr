---
title: Créer des bots d’automatisation de tâche | Microsoft Docs
description: Découvrez comment concevoir des bots effectuant des tâches sans intervention humaine.
author: matvelloso
ms.author: mateusv
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 2/13/2018
ms.openlocfilehash: 1f14329958711cc7f9ef8832b267587e1232da2a
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39298929"
---
# <a name="create-task-automation-bots"></a>Créer des bots d’automatisation de tâche

Un bot d’automatisation de tâche permet à l’utilisateur d’effectuer une tâche ou un ensemble de tâches spécifique sans assistance humaine. Ce type de bot s’apparente souvent à une application ou un site web classique. Il communique avec l’utilisateur principalement par le biais de texte et de contrôles utilisateur enrichis. Il peut bénéficier de capacités de compréhension du langage naturel, ce qui permet d’enrichir les conversations avec les utilisateurs. 

## <a name="example-use-case-password-reset"></a>Exemple de cas d’usage : réinitialisation de mot de passe

Pour mieux comprendre la nature d’un bot de tâche, examinez l’exemple de cas d’usage suivant, portant sur la réinitialisation de mot de passe. 

Chaque jour, le support technique de la société Contoso reçoit plusieurs appels d’employés ayant besoin de réinitialiser leur mot de passe. Pour permettre aux agents du support technique de se consacrer à la résolution de problèmes plus complexes, Contoso souhaite automatiser cette tâche simple et reproductible qui consiste à réinitialiser le mot de passe d’un employé. 

John, un développeur expérimenté de Contoso, décide de créer un bot pour automatiser la tâche de réinitialisation de mot de passe. Il commence par rédiger une spécification de conception du bot, comme il le ferait pour créer une application ou un site web. 

::: moniker range="azure-bot-service-3.0"

### <a name="navigation-model"></a>Modèle de navigation

La spécification définit le modèle de navigation :

![Structure des boîtes de dialogue](~/media/bot-service-design-pattern-task-automation/simple-task1.png)

L’utilisateur accède d’abord à la boîte de dialogue `RootDialog`. Lorsqu’il demande une réinitialisation de mot de passe, il  
accède à la boîte de dialogue `ResetPasswordDialog`. Dans la boîte de dialogue `ResetPasswordDialog`, le bot demande à l’utilisateur deux informations : son numéro de téléphone et sa date de naissance. 

> [!IMPORTANT]
> La conception de bot décrite dans cet article est fournie uniquement à titre d’exemple. Dans un scénario réel, un bot de réinitialisation de mot de passe appliquerait probablement un processus de vérification d’identité plus robuste.

### <a name="dialogs"></a>Boîtes de dialogue

La spécification décrit ensuite l’apparence et les fonctionnalités de chaque boîte de dialogue. 

#### <a name="root-dialog"></a>Boîte de dialogue racine

La boîte de dialogue racine offre à l’utilisateur deux options : 

1. **Modifier le mot de passe** : pour les scénarios où l’utilisateur connaît son mot de passe actuel et veut simplement le modifier.
2. **Réinitialiser le mot de passe** : pour les scénarios où l’utilisateur a oublié ou égaré son mot de passe et doit donc en générer un nouveau.

> [!NOTE]
> Par souci de simplicité, cet article décrit uniquement le flux de **réinitialisation de mot de passe**.

La spécification décrit la boîte de dialogue racine comme illustré dans la capture d’écran suivante.

![Structure des boîtes de dialogue](~/media/bot-service-design-pattern-task-automation/simple-task2.png)

#### <a name="resetpassword-dialog"></a>Boîte de dialogue ResetPassword

Lorsque l’utilisateur choisit l’option **Réinitialiser le mot de passe** dans la boîte de dialogue racine, la boîte de dialogue `ResetPassword` est appelée. 
La boîte de dialogue `ResetPassword` appelle ensuite deux autres boîtes de dialogue. 
Tout d’abord, elle appelle la boîte de dialogue `PromptStringRegex` pour recueillir le numéro de téléphone de l’utilisateur. 
Puis, elle appelle la boîte de dialogue `PromptDate` pour recueillir sa date de naissance. 

> [!NOTE]
> Dans cet exemple, la logique implémentée par John pour recueillir le numéro de téléphone et la date de naissance de l’utilisateur repose sur deux boîtes de dialogue distinctes. Cette approche simplifie le code requis pour chaque boîte de dialogue. De plus, les boîtes de dialogue seront plus facilement réutilisables dans d’autres scénarios. 

La spécification décrit la boîte de dialogue `ResetPassword`.

![Structure des boîtes de dialogue](~/media/bot-service-design-pattern-task-automation/simple-task3.png)

#### <a name="promptstringregex-dialog"></a>Boîte de dialogue PromptStringRegex

La boîte de dialogue `PromptStringRegex` invite l’utilisateur à entrer son numéro de téléphone et vérifie que ce numéro respecte le format attendu. 
Elle gère également le scénario où l’utilisateur fournit une entrée non valide à plusieurs reprises. 
La spécification décrit la boîte de dialogue `PromptStringRegex`.

![Structure des boîtes de dialogue](~/media/bot-service-design-pattern-task-automation/simple-task4.png)

### <a name="prototype"></a>Prototype

Enfin, la spécification fournit l’exemple d’un utilisateur qui communique avec le bot pour effectuer correctement la tâche de réinitialisation de mot de passe.

![Structure des boîtes de dialogue](~/media/bot-service-design-pattern-task-automation/simple-task5.png)

::: moniker-end 

## <a name="bot-app-or-website"></a>Bot, application ou site web ?

Vous vous demandez peut-être pourquoi ne pas simplement créer une application ou un site web, dans la mesure où un bot d’automatisation de tâche s’y apparente étroitement. Selon votre scénario, il peut être judicieux de créer une application ou un site web plutôt qu’un bot. Vous pouvez même choisir d’incorporer votre bot dans une application en utilisant [l’API Direct Line du Bot Framework][directLineAPI] ou le <a href="https://github.com/Microsoft/BotFramework-WebChat" target="_blank">contrôle Discussion Web</a>. L’implémentation de votre bot dans le contexte d’une application s’avère doublement avantageuse dans la mesure où elle assure à la fois une expérience d’application riche et une expérience de conversation. 

Cependant, dans de nombreux cas, il peut être beaucoup plus complexe et beaucoup plus coûteux de créer une application ou un site web plutôt qu’un bot. Souvent, une application ou un site web doit prendre en charge plusieurs plateformes et clients. Les processus d’empaquetage et de déploiement peuvent être longs et fastidieux. Enfin, la nécessité pour l’utilisateur de télécharger et d’installer une application n’est pas idéale. Par conséquent, un bot peut souvent offrir un moyen beaucoup plus simple de résoudre un problème spécifique. 

En outre, les bots offrent une grande liberté en termes de développement et d’extension. Par exemple, un développeur peut choisir d’ajouter des capacités vocales et de langage naturel au bot de réinitialisation de mot de passe afin de le rendre accessible par appel audio. Il peut également ajouter la prise en prise en charge de messages texte. La société peut configurer des bornes dans l’ensemble du bâtiment et incorporer le bot de réinitialisation de mot de passe dans cette expérience.

::: moniker range="azure-bot-service-3.0"
## <a name="sample-code"></a>Exemple de code

Pour obtenir un exemple complet montrant comment implémenter une automatisation de tâche simple en utilisant le Kit de développement logiciel (SDK) Bot Builder pour .NET, consultez <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/capability-SimpleTaskAutomation" target="_blank">l’exemple d’automatisation de tâche simple</a> dans GitHub.

Pour obtenir un exemple complet montrant comment implémenter une automatisation de tâche simple en utilisant le Kit de développement logiciel (SDK) Bot Builder pour Node.js, consultez <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/capability-SimpleTaskAutomation" target="_blank">l’exemple d’automatisation de tâche simple</a> dans GitHub.

## <a name="additional-resources"></a>Ressources supplémentaires

- [Boîtes de dialogue](~/dotnet/bot-builder-dotnet-dialogs.md)
- [Gérer un flux de conversation avec des boîtes de dialogue (.NET)](~/dotnet/bot-builder-dotnet-manage-conversation-flow.md)
- [Gérer un flux de conversation avec des boîtes de dialogue (Node.js)](~/nodejs/bot-builder-nodejs-manage-conversation-flow.md)

::: moniker-end

[directLineAPI]: https://docs.botframework.com/en-us/restapi/directline3/#navtitle
