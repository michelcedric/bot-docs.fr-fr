---
title: Vue d’ensemble des dialogues | Microsoft Docs
description: Découvrez comment utiliser des dialogues dans le Kit de développement logiciel (SDK) Bot Builder pour modéliser des conversations et gérer un flux de conversation.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: eb99be4699bba71a1fdc55bab19d035e4e31f536
ms.sourcegitcommit: 67445b42796d90661afc643c6bb6533e9a662cbc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/06/2018
ms.locfileid: "39574555"
---
# <a name="dialogs-in-the-bot-builder-sdk-for-net"></a>Dialogues dans le Kit de développement (SDK) Bot Builder pour .NET

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-dialogs.md)
> - [Node.JS](../nodejs/bot-builder-nodejs-dialog-overview.md)

Lorsque vous créez un robot en utilisant le Kit de développement logiciel (SDK) Bot Builder pour .NET, vous pouvez utiliser des dialogues pour modéliser une conversation et gérer un [flux de conversation](../bot-service-design-conversation-flow.md). Chaque dialogue est une abstraction qui encapsule son propre état dans une classe C# qui implémente `IDialog`. Un dialogue peut être composé avec d’autres dialogues pour maximiser la réutilisation, et un contexte de dialogue conserve la [pile des dialogues](../bot-service-design-conversation-flow.md#dialog-stack) actifs dans la conversation à un moment quelconque. 

Une conversation constituée de dialogues est portable sur divers ordinateurs, ce qui rend possible la mise à l’échelle de votre implémentation de robot. Lorsque vous utilisez des dialogues dans le Kit de développement logiciel (SDK) Bot Builder pour .NET, l’état de la conversation (la pile des dialogues et l’état de chacun d’eux dans la pile) est automatiquement stocké dans le stockage de [données d’état](bot-builder-dotnet-state.md) de votre choix. Cela permet au code de service de votre robot d’être sans état, comme une application web qui n’a pas besoin de stocker l’état de session dans la mémoire du serveur web. 

## <a name="echo-bot-example"></a>Exemple de robot echo

Considérez cet exemple de robot echo, qui explique comment modifier le robot créé dans le cadre du didacticiel [Démarrage rapide](bot-builder-dotnet-quickstart.md) afin qu’il utilise des dialogues pour échanger des messages avec l’utilisateur.

> [!TIP]
> Pour suivre cet exemple, utilisez les instructions fournies dans le didacticiel [Démarrage rapide](bot-builder-dotnet-quickstart.md) pour créer un robot, puis mettez à jour le fichier **MessagesController.cs** de celui-ci comme décrit ci-dessous.

### <a name="messagescontrollercs"></a>MessagesController.cs 

Dans le Kit de développement logiciel (SDK) Bot Builder pour .NET, la bibliothèque du [générateur][builderLibrary] vous permet d’implémenter des dialogues. Pour accéder aux classes pertinentes, importez l’espace de noms `Dialogs`.

[!code-csharp[Using statement](../includes/code/dotnet-dialogs.cs#usingStatement)]

Ajoutez ensuite cette `EchoDialog` au fichier **MessagesController.cs** pour représenter la conversation. 

[!code-csharp[EchoDialog class](../includes/code/dotnet-dialogs.cs#echobot1)]

Enfin, rattachez la classe `EchoDialog` à la `Post` méthode en appelant la méthode `Conversation.SendAsync`.

[!code-csharp[Post method](../includes/code/dotnet-dialogs.cs#echobot2)]

### <a name="implementation-details"></a>Informations d’implémentation 

La méthode `Post` est marquée `async` parce que Bot Builder utilise les fonctions C# pour gérer une communication asynchrone. Elle retourne un objet `Task` qui représente la tâche chargée d’envoyer des réponses au message transmis. S’il existe une exception, la `Task` qui retournée par la méthode contient des informations sur l’exception. 

La méthode `Conversation.SendAsync` est essentielle pour l’implémentation de dialogues avec le Kit de développement logiciel (SDK) Bot Builder pour .NET. Elle suit le <a href="https://en.wikipedia.org/wiki/Dependency_inversion_principle" target="_blank">principe d’inversion de dépendance</a> et effectue les opérations suivantes :

1. Elle instancie les composants requis.  
2. Elle désérialise l’état de la conversation (la pile de dialogues et l’état de chacun d’eux dans la pile) à partir de `IBotDataStore`.
3. Elle reprend le processus de conversation là où le robot l’a suspendu et attend un message.
4. Elle envoie les réponses.
5. Elle sérialise l’état de la conversation mis à jour et le réenregistre dans `IBotDataStore`.

Quand la conversation commence pour la première fois, la boîte de dialogue ne contient pas d’état. Ainsi, `Conversation.SendAsync` construit `EchoDialog` et appelle sa méthode `StartAsync`. La méthode `StartAsync` appelle `IDialogContext.Wait` avec le délégué de continuation pour spécifier la méthode à appeler los de la réception d’un nouveau message (`MessageReceivedAsync`). 

La méthode `MessageReceivedAsync` attend un message, publie une réponse et attend le message suivant. À chaque appel de `IDialogContext.Wait`, le robot entre en état suspendu et peut être redémarré sur tout ordinateur recevant le message. 

Un robot créé à l’aide des exemples de code ci-dessus répond à chaque message envoyé par l’utilisateur en renvoyant simplement ce message préfixé avec le texte « Vous avez dit : ». Le robot étant créé à l’aide de dialogues, il peut évoluer pour prendre en charge des conversations plus complexes sans avoir à gérer explicitement l’état.

## <a name="echo-bot-with-state-example"></a>Robot echo avec l’exemple d’état

L’exemple suivant repose sur le précédent en ajoutant la possibilité de suivre l’état du dialogue. Quand la classe `EchoDialog` est mise à jour comme dans l’exemple de code ci-dessous, le robot répond à chaque message que l’utilisateur envoie en renvoyant le message préfixé avec un nombre (`count`) suivi du texte « Vous avez dit : ». Le robot continue d’incrémenter `count` avec chaque réponse, jusqu’à ce que l’utilisateur choisisse de réinitialiser le nombre.

### <a name="messagescontrollercs"></a>MessagesController.cs 

[!code-csharp[EchoDialog class](../includes/code/dotnet-dialogs.cs#echobot3)]

### <a name="implementation-details"></a>Informations d’implémentation

Comme dans le premier exemple, la méthode `MessageReceivedAsync` est appelée à la réception d’un nouveau message. Cette fois cependant, la méthode `MessageReceivedAsync` évalue le message de l’utilisateur avant de répondre. Si le message de l’utilisateur est « réinitialiser », l’invite `PromptDialog.Confirm` intégrée génère un sous-dialogue qui invite l’utilisateur à confirmer la réinitialisation du nombre. Le sous-dialogue a son propre état privé qui n’interfère pas avec l’état du dialogue parent. Quand l’utilisateur répond à l’invite, le résultat du sous-dialogue est transmis à la méthode `AfterResetAsync` qui envoie un message à l’utilisateur pour indiquer si le nombre a été réinitialisé ou non, puis appelle `IDialogContext.Wait` avec une continuation vers la méthode `MessageReceivedAsync` sur le message suivant.

## <a name="dialog-context"></a>Contexte du dialogue.

L’interface `IDialogContext` transmise à chaque méthode de dialogue donne accès aux services dont un dialogue a besoin pour enregistrer l’état et communiquer avec le canal. L’interface `IDialogContext` comprend trois interfaces : [Internals.IBotData][iBotData], [Internals.IBotToUser][iBotToUser] et [ Internals.IDialogStack][iDialogStack]. 

### <a name="internalsibotdata"></a>Internals.IBotData

L’interface `Internals.IBotData` donne accès aux données d’état de conversation par utilisateur, par conversation, et privées conservées par le connecteur. Les données par utilisateur sont utiles pour stocker les données utilisateur non liées à une conversation spécifique, tandis que les données par conversation sont utiles pour stocker des données générales à propos d’une conversation, et que les données de conversation privées sont utiles pour stocker les données utilisateur associées à une conversation spécifique. 

### <a name="internalsibottouser"></a>Internals.IBotToUser

L’interface `Internals.IBotToUser` fournit des méthodes pour envoyer un message d’un robot à un utilisateur. Des messages peuvent être envoyés en fonction de la réponse à l’appel de méthode d’API web, ou directement à l’aide du [client de connecteur](bot-builder-dotnet-connector.md#create-a-connector-client). L’envoi et la réception de messages via le contexte du dialogue garantissent que l’état `Internals.IBotData` est transmis via le connecteur.

### <a name="internalsidialogstack"></a>Internals.IDialogStack

L’interface `Internals.IDialogStack` fournit des méthodes pour gérer la [pile de dialogues](../bot-service-design-conversation-flow.md#dialog-stack). La plupart du temps, la pile de dialogues est gérée automatiquement pour vous. Toutefois, il peut y avoir des cas où vous souhaitez gérer explicitement la pile. Par exemple, vous pouvez appeler un dialogue enfant et l’ajouter en haut de la pile de dialogues, marquer le dialogue actuel comme terminé (ce qui a pour effet de le supprimer de la pile de dialogues et de retourner le résultat au dialogue précédent dans la pile), suspendre le dialogue en cours jusqu’à ce qu’un message de l’utilisateur arrive, voire carrément réinitialiser la pile de dialogues.

## <a name="serialization"></a>Sérialisation

La pile de dialogues et l’état de tous les dialogues actifs sont sérialisés en [IBotDataBag][iBotDataBag] par utilisateur, par conversation. L’objet blob sérialisé est conservé dans les messages que le robot échange avec le [connecteur](bot-builder-dotnet-concepts.md#connector). Pour être sérialisée, une classe `Dialog` doit inclure l’attribut `[Serializable]`. Toutes les implémentations de `IDialog` dans la bibliothèque du [générateur][builderLibrary] sont marquées comme sérialisables. 

Les [méthodes Chain](#dialog-chains) fournissent une interface courante aux dialogues, qui est utilisable dans la syntaxe de requête LINQ. La forme compilée de la syntaxe de requête LINQ utilise souvent des méthodes anonymes. Si ces méthodes anonymes ne font pas référence à l’environnement de variables locales, ces méthodes anonymes n’ont pas d’état et sont sérialisables de façon triviale. Toutefois, si la méthode anonyme capture une variable locale dans l’environnement, l’objet fermeture qui en résulte (généré par le compilateur) n’est pas marqué comme sérialisable. Dans ce cas, Bot Builder lève une exception `ClosureCaptureException` pour identifier le problème.

Pour utiliser la réflexion pour sérialiser des classes qui ne sont pas marquées comme sérialisables, la bibliothèque du générateur inclut un substitut de sérialisation basé sur la réflexion, que vous pouvez utiliser pour vous inscrire auprès d’[Autofac][autofac].

[!code-csharp[Serialization](../includes/code/dotnet-dialogs.cs#serialization)]

## Chaînes de dialogue <a id="dialog-chains"></a>

Si vous pouvez gérer explicitement la pile de dialogues actifs en utilisant `IDialogStack.Call<R>` et `IDialogStack.Done<R>`, vous pouvez également la gérer implicitement en utilisant ces méthodes [Chain][chain] courantes.


|           Méthode            |  type   |                                 Notes                                  |
|-----------------------------|---------|------------------------------------------------------------------------|
|     Chain.Select<T, R>      |  LINQ   |           Prend en charge « select » et « let » dans la syntaxe de requête LINQ.            |
|  Chain.SelectMany<T, C, R>  |  LINQ   |            Prend en charge des « from » successifs dans la syntaxe de requête LINQ.            |
|       Chain.Where<T>        |  LINQ   |                 Prend en charge « where » dans la syntaxe de requête LINQ.                 |
|        Chain.From<T>        | Chains  |                Instancie une nouvelle instance d’un dialogue.                |
|       Chain.Return<T>       | Chains  |                Retourne une valeur constante dans la chaîne.                |
|         Chain.Do<T>         | Chains  |               Autorise des effets secondaires dans la chaîne.                |
|  Chain.ContinueWith<T, R>   | Chains  |                      Simple chaînage de dialogues.                       |
|       Chain.Unwrap<T>       | Chains  |                  Désencapsule un dialogue imbriqué dans un dialogue.                   |
| Chain.DefaultIfException<T> | Chains  | Ingère une exception du résultat précédent et retourne default(T). |
|        Chain.Loop<T>        | Branche  |                   Effectue une boucle de la chaîne entière de dialogues.                   |
|        Chain.Fold<T>        | Branche  |   Incorpore les résultats d’une énumération de dialogues dans un résultat unique.   |
|     Chain.Switch<T, R>      | Branche  |            Prend en charge la ramification en différentes chaînes de dialogue.            |
|     Chain.PostToUser<T>     | Message |                      Publie un message à l’adresse de l’utilisateur.                      |
|     Chain.WaitToBot<T>      | Message |                    Attend un message adressé au robot.                     |
|    Chain.PostToChain<T>     | Message |              Commence une chaîne par un message de l’utilisateur.              |

### <a name="examples"></a>Exemples 

La syntaxe de requête LINQ utilise la méthode `Chain.Select<T, R>`.

[!code-csharp[Chain.Select](../includes/code/dotnet-dialogs.cs#chain1)]

Ou la méthode `Chain.SelectMany<T, C, R>`.

[!code-csharp[Chain.SelectMany](../includes/code/dotnet-dialogs.cs#chain2)]

Les méthodes `Chain.PostToUser<T>` et `Chain.WaitToBot<T>` publient des messages du robot à l’utilisateur et inversement.

[!code-csharp[Chain.PostToUser](../includes/code/dotnet-dialogs.cs#chain3)]

La méthode `Chain.Switch<T, R>` ramifie le flux de dialogue de conversation.

[!code-csharp[Chain.Switch](../includes/code/dotnet-dialogs.cs#chain4)]

Si `Chain.Switch<T, R>` retourne un `IDialog<IDialog<T>>` imbriqué, le `IDialog<T>` interne peut être désencapsulé avec `Chain.Unwrap<T>`. Cela permet de ramifier les conversations vers différents chemins de dialogues enchaînés, éventuellement de longueurs inégales. Cet exemple présente un exemple plus complet de ramification de dialogues écrits dans le style de la chaîne courant avec une gestion de pile implicite.

[!code-csharp[Chain.Switch](../includes/code/dotnet-dialogs.cs#chain5)]

## <a name="next-steps"></a>Étapes suivantes

Les dialogues gèrent le flux de conversation entre un robot et un utilisateur. Un dialogue définit la manière d’interagir avec un utilisateur. Un robot peut utiliser de nombreux dialogues organisés en piles pour guider la conversation avec l’utilisateur. Dans la section suivante, découvrez comment la pile de dialogues augmente et diminue à mesure que vous créez et écartez des dialogues dans la pile.

> [!div class="nextstepaction"]
> [Gérer un flux de conversation avec des dialogues](bot-builder-dotnet-manage-conversation-flow.md)


[builderLibrary]: /dotnet/api/microsoft.bot.builder.dialogs

[iBotData]: /dotnet/api/microsoft.bot.builder.dialogs.internals.ibotdata

[iBotToUser]: /dotnet/api/microsoft.bot.builder.dialogs.internals.ibottouser

[iDialogStack]: /dotnet/api/microsoft.bot.builder.dialogs.internals.idialogstack

[iBotDataBag]: /dotnet/api/microsoft.bot.builder.dialogs.ibotdatabag

[autofac]: /dotnet/api/microsoft.bot.builder.autofac.base

[chain]: /dotnet/api/microsoft.bot.builder.dialogs.chain
