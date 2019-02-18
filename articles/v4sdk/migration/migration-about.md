---
title: Différences entre les kits SDK v3 et v4 | Microsoft Docs
description: Décrit les différences entre les kits SDK v3 et v4.
keywords: migration de bot, formflow, dialogues, état
author: JonathanFingold
ms.author: v-jofing
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 02/11/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 7e5440e7d47d88b7ff6827359e7eb621bce53e3c
ms.sourcegitcommit: 7f418bed4d0d8d398f824e951ac464c7c82b8c3e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 02/14/2019
ms.locfileid: "56240487"
---
# <a name="differences-between-the-v3-and-v4-net-sdk"></a>Différences entre les kits SDK .NET v3 et v4

La v4 du kit SDK Bot Framework prend en charge le même service Bot Framework sous-jacent que la v3. Toutefois, la v4 est une refactorisation de la version précédente du kit SDK qui offre aux développeurs davantage de flexibilité et de contrôle sur leurs bots. Parmi les principaux changements apportés au kit SDK, citons les suivants :

- Introduction d’un adaptateur de bot. L’adaptateur fait partie de la pile de traitement d’activité.
  - L’adaptateur gère l’authentification du Bot Framework.
  - L’adaptateur gère le trafic entrant et sortant entre un canal et le gestionnaire de tour de votre bot, encapsulant les appels au Bot Framework Connector.
  - L’adaptateur initialise le contexte pour chaque tour.
  - Pour plus d’informations, consultez la rubrique sur le [fonctionnement des bots](../bot-builder-basics.md).
- Gestion de l’état refactorisée.
  - Les données d’état ne sont plus automatiquement disponibles au sein d’un bot.
  - L’état est désormais géré par le biais d’objets de gestion d’état et d’accesseurs de propriété.
  - Pour plus d’informations, consultez la rubrique sur la [gestion de l’état](../bot-builder-concept-state.md).
- Nouvelle bibliothèque de dialogues.
  - Les dialogues v3 doivent être réécrits pour la nouvelle bibliothèque de dialogues.
  - Les dialogues scorables n’existent plus. Vous pouvez rechercher des commandes « globales » dans le gestionnaire de tour, avant de passer le contrôle à vos dialogues.
  - Pour plus d’informations, consultez la rubrique sur la [bibliothèque de dialogues](../bot-builder-concept-dialog.md).
- Prise en charge d’ASP.NET Core.
  - Les modèles pour la création de bots C# ciblent le framework ASP.NET Core.
  - Vous pouvez toujours utiliser ASP.NET pour vos bots, mais notre objectif pour la v4 est de prendre en charge le framework ASP.NET Core.
  - Pour plus d’informations sur ce framework, consultez l’[introduction à ASP.NET Core](https://docs.microsoft.com/aspnet/core/).

## <a name="activity-processing"></a>Traitement des activités

Quand vous créez l’adaptateur pour votre bot, vous fournissez également un délégué de gestionnaire de tour qui reçoit les activités entrantes des canaux et des utilisateurs. L’adaptateur crée un objet de contexte de tour pour chaque activité reçue. Il transmet l’objet de contexte de tour au gestionnaire de tour, puis supprime l’objet une fois le tour terminé.

Le gestionnaire de tour peut recevoir de nombreux types d’activités. En général, vous ne transférez que les activités liées aux _messages_ vers les dialogues de votre bot. Pour obtenir des informations détaillées sur les types d’activités, consultez la rubrique sur le [schéma d’activité](https://aka.ms/botSpecs-activitySchema).

### <a name="handling-turns"></a>Gestion des tours

Votre gestionnaire de tour doit correspondre à la signature d’un `BotCallbackHandler` :

```csharp
public delegate Task BotCallbackHandler(
    ITurnContext turnContext,
    CancellationToken cancellationToken);
```

Lors du traitement d’un tour, utilisez le contexte de tour pour obtenir des informations sur l’activité entrante et envoyer des activités à l’utilisateur :

| | |
|-|-|
| Pour obtenir l’activité entrante | Obtenez la propriété `Activity` du contexte de tour. |
| Pour créer une activité et l’envoyer à l’utilisateur | Appelez la méthode `SendActivityAsync` du contexte de tour.<br/>Pour plus d’informations, consultez les rubriques sur l’[envoi et la réception d’un message texte](../bot-builder-howto-send-messages.md) et sur l’[ajout de médias à des messages](../bot-builder-howto-add-media-attachments.md) |

La classe `MessageFactory` fournit des méthodes d’assistance pour créer et mettre en forme des activités.

### <a name="scorables-is-gone"></a>Disparition des dialogues scorables

Gérez ces éléments dans la boucle de messages du bot. Pour la procédure à suivre avec les dialogues v4, consultez la rubrique sur la [gestion des interruptions de l’utilisateur](../bot-builder-howto-handle-user-interrupt.md).

Les arborescences de dispatch scorables composables et les dialogues de chaînes composables, comme l’_exception par défaut_, ont également disparu. Un moyen de reproduire cette fonctionnalité consiste à l’implémenter dans le gestionnaire de tour de votre bot.

## <a name="state-management"></a>Gestion de l'état

La v4 n’utilise ni les propriétés `UserData`, `ConversationData` et `PrivateConversationData` ni les conteneurs de données pour gérer l’état.
L’état est désormais géré par le biais d’objets de gestion d’état et d’accesseurs de propriété, comme décrit dans la [gestion de l’état](../bot-builder-concept-state.md).

La v4 définit des classes `UserState`, `ConversationState` et `PrivateConversationState` qui gèrent les données d’état pour le bot. Vous devez créer un accesseur de propriété d’état pour chaque propriété à rendre persistante, au lieu de simplement lire et écrire dans un conteneur de données prédéfini.

### <a name="setting-up-state"></a>Configuration de l’état

L’état doit être configuré sous la forme de singletons dans la mesure du possible, dans **Startup.cs** pour .NET Core ou dans **Global.asax.cs** pour le .NET Framework.

1. Initialisez un ou plusieurs objets `IStorage`. Cela représente le magasin de stockage pour les données de votre bot.
    Le kit SDK v4 fournit quelques [couches de stockage](../bot-builder-concept-state.md#storage-layer).
    Vous pouvez également implémenter votre propre couche pour vous connecter à un autre type de magasin.
1. Ensuite, créez et inscrivez des objets de [gestion d’état](../bot-builder-concept-state.md#state-management) en fonction des besoins.
    Vous disposez des mêmes étendues que dans la v3 et vous pouvez en créer d’autres si nécessaire.
1. Ensuite, créez et inscrivez des [accesseurs de propriété d’état](../bot-builder-concept-state.md#state-property-accessors) pour les propriétés dont votre bot a besoin.
    Au sein d’un objet de gestion d’état, chaque accesseur de propriété nécessite un nom unique.

### <a name="using-state"></a>Utilisation de l’état

Vous pouvez utiliser l’injection de dépendances pour accéder à ces éléments chaque fois que votre bot est créé.
(Dans ASP.NET, une instance de votre bot ou contrôleur de message est créée pour chaque tour). Utilisez les accesseurs de propriété d’état pour obtenir et mettre à jour vos propriétés, et utilisez les objets de gestion d’état pour écrire tous les changements dans le stockage. Sachant que vous devez prendre en compte les problèmes de concurrence, voici comment accomplir quelques tâches courantes.

| | |
|-|-|
| Créer un accesseur de propriété d’état | Appelez `BotState.CreateProperty<T>`.<br/>`BotState` est la classe de base abstraite pour la conversation, la conversation privée et l’état utilisateur. |
| Obtenir la valeur actuelle d’une propriété | Appelez `IStatePropertyAccessor<T>.GetAsync`.<br/>Si aucune valeur n’a été définie précédemment, le paramètre d’usine par défaut est utilisé pour générer une valeur. |
| Mettre à jour la valeur actuelle mise en cache d’une propriété | Appelez `IStatePropertyAccessor<T>.SetAsync`.<br/>Seul le cache est mis à jour (pas la couche de stockage de sauvegarde). |
| Rendre les changements d’état apportés au stockage persistants | Appelez `BotState.SaveChangesAsync` pour tout objet de gestion d’état dans lequel l’état a changé avant la fermeture du gestionnaire de tour. |

Pour plus d’informations, consultez la rubrique sur la [gestion de l’état](../bot-builder-concept-state.md#saving-state).

### <a name="managing-concurrency"></a>Gérer l'accès concurrentiel

Votre bot peut avoir besoin de gérer la concurrence de l’état. Pour plus d’informations, consultez la section [Enregistrement de l’état](../bot-builder-concept-state.md#saving-state) dans **Gestion de l’état** et la section [Gérer la concurrence à l’aide des étiquettes d’entité](../bot-builder-howto-v4-storage.md#manage-concurrency-using-etags) dans **Écrire directement dans le stockage**.

## <a name="dialogs-library"></a>Bibliothèque des dialogues

Voici quelques-uns des principaux changements apportés aux dialogues :

- La bibliothèque de dialogues est désormais un package NuGet distinct : **Microsoft.Bot.Builder.Dialogs**.
- Les classes de dialogue n’ont plus besoin d’être sérialisables. L’état du dialogue est géré par le biais d’un accesseur de propriété d’état `DialogState`.
  - La propriété d’état de dialogue est maintenant persistante entre les tours, contrairement à l’objet de dialogue.
- L’interface `IDialogContext` est remplacée par la classe `DialogContext`. Au cours d’un tour, vous créez un contexte de dialogue pour un _jeu de dialogues_.
  - Ce contexte de dialogue encapsule la pile de dialogues (l’ancien frame de pile). Cette information est persistante dans la propriété d’état du dialogue.
- L’interface `IDialog` est remplacée par la classe `Dialog` abstraite.

### <a name="defining-dialogs"></a>Définition des dialogues

Vous disposez à présent de plusieurs options pour définir des dialogues :

- Dialogue en cascade (instance de la classe `WaterfallDialog`).

  Il est conçu pour fonctionner avec les dialogues d’invite, qui invitent l’utilisateur à saisir différents types d’entrées et les valident. Consultez la rubrique sur l’[invite à saisir une entrée](../bot-builder-prompts.md).

  Ceci automatise la majeure partie du processus pour vous, mais impose une certaine forme à votre code de dialogue (consultez la rubrique sur les [flux de conversation séquentiels](../bot-builder-dialog-manage-conversation-flow.md)). Toutefois, vous pouvez créer d’autres flux de contrôle en ajoutant plusieurs dialogues à un jeu de dialogues (consultez la rubrique sur les [flux de conversation avancés](../bot-builder-dialog-manage-complex-conversation-flow.md)).

- Un dialogue de composant (dérivé de la classe `ComponentDialog`).

  Cela vous permet d’encapsuler le code du dialogue sans conflits de nommage avec les contextes externes. Consultez la rubrique sur la [réutilisation des dialogues](../bot-builder-compositcontrol.md).

- Dialogue personnalisé (dérivé de la classe abstraite `Dialog`).

  Cette option vous donne le plus de flexibilité dans la gestion du comportement de vos dialogues, mais vous oblige également à en savoir plus sur la manière dont la pile de dialogues est implémentée.

Pour accéder à un dialogue, vous devez placer une instance de celui-ci dans un _jeu de dialogues_, puis générer un _contexte de dialogue_ pour ce jeu.

Vous devez fournir un accesseur de propriété d’état de dialogue quand vous créez un jeu de dialogues. Le framework peut ainsi rendre l’état du dialogue persistant d’un tour à l’autre. [Gestion de l’état](../bot-builder-concept-state.md) décrit la façon dont l’état est géré dans la v4.

### <a name="using-dialogs"></a>Utilisation de dialogues

Voici une liste des opérations courantes dans la v3 et la procédure à suivre pour les accomplir dans un dialogue en cascade. Notez que chaque étape d’un dialogue en cascade est censée retourner une valeur `DialogTurnResult`. Si ce n’est pas le cas, la cascade peut se terminer plus tôt.

| Opération | v3 | v4 |
|:---|:---|:---|
| Gérer le début de votre dialogue | Implémentez `IDialog.StartAsync` | Définissez ceci comme première étape d’un dialogue en cascade. |
| Envoyer une activité | Appelez `IDialogContext.PostAsync`. | Appelez `ITurnContext.SendActivityAsync`.<br/>Utilisez la propriété `Context` du contexte d’étape pour obtenir le contexte de tour.  |
| Attendre la réponse d’un utilisateur | Utiliser un paramètre `IAwaitable<IMessageActivity>` et appeler `IDialogContext.Wait` | Utilisez return await avec `ITurnContext.PromptAsync` pour commencer un dialogue d’invite. Ensuite, récupérez le résultat à l’étape suivante de la cascade. |
| Gérer la continuation de votre dialogue | Appelez `IDialogContext.Wait`. | Ajoutez des étapes supplémentaires à un dialogue en cascade ou implémentez `Dialog.ContinueDialogAsync` |
| Signaler la fin du traitement jusqu’au prochain message de l’utilisateur | Appelez `IDialogContext.Wait`. | Retournez `Dialog.EndOfTurn`. |
| Commencer un dialogue enfant | Appelez `IDialogContext.Call`. | Utilisez return await avec la méthode `BeginDialogAsync` du contexte d’étape.<br/>Si le dialogue enfant retourne une valeur, celle-ci est disponible à l’étape suivante de la cascade par le biais de la propriété `Result` du contexte d’étape. |
| Remplacer le dialogue actuel par un nouveau dialogue | Appelez `IDialogContext.Forward`. | Utilisez return await avec `ITurnContext.ReplaceDialogAsync`. |
| Signaler que le dialogue en cours est terminé | Appelez `IDialogContext.Done`. | Utilisez return await avec la méthode `EndDialogAsync` du contexte d’étape. |
| Échec d’un dialogue. | Appelez `IDialogContext.Fail`. | Levez une exception à intercepter à un autre niveau du bot, terminez l’étape avec l’état `Cancelled` ou appelez la méthode `CancelAllDialogsAsync` du contexte d’étape ou de dialogue.<br/>Notez que dans la v4, les exceptions au sein d’un dialogue sont propagées le long de la pile C# et non le long de la pile de dialogues. |

Autres remarques concernant le code v4 :

- Les différentes classes dérivées de `Prompt` dans la v4 implémentent les invites utilisateur en tant que dialogues à deux étapes distinctes. Découvrez comment [collecter les entrées utilisateur avec une invite de dialogue](../bot-builder-prompts.md).
- Utilisez `DialogSet.CreateContextAsync` pour créer un contexte de dialogue pour le tour actuel.
- Au sein d’un dialogue, utilisez la propriété `DialogContext.Context` pour obtenir le contexte de tour actuel.
- Les étapes en cascade ont un paramètre `WaterfallStepContext`, qui dérive de `DialogContext`.
- Toutes les classes de dialogue et d’invite concrètes dérivent de la classe `Dialog` abstraite.
- Vous affectez un ID quand vous créez un dialogue composant. Chaque dialogue d’un jeu de dialogues doit recevoir un ID unique dans ce jeu.

### <a name="passing-state-between-and-within-dialogs"></a>Transmission de l’état entre dialogues et au sein d’entre eux

Les sections [État de dialogue](../bot-builder-concept-dialog.md#dialog-state), [Propriétés du contexte d’étape d’un dialogue en cascade](../bot-builder-concept-dialog.md#waterfall-step-context-properties) et [Utilisation de dialogues](../bot-builder-concept-dialog.md#using-dialogs) de l’article **Bibliothèque de dialogues** décrivent comment gérer l’état de dialogue dans la v4.

### <a name="iawaitable-is-gone"></a>IAwaitable n’est plus disponible

Pour obtenir l’activité de l’utilisateur dans un tour, obtenez-la à partir du contexte de tour.

Pour inviter l’utilisateur et recevoir le résultat d’une invite :

- Ajoutez une instance d’invite appropriée à votre jeu de dialogues.
- Appelez l’invite à partir d’une étape dans un dialogue en cascade.
- Récupérez le résultat à partir de la propriété `Result` du contexte d’étape à l’étape suivante.

### <a name="formflow"></a>Formflow

Dans la v3, Formflow faisait partie du kit SDK C#, mais pas du kit SDK JavaScript. Il ne fait pas partie du kit SDK v4, mais une version de communauté existe pour C#.

| Nom du package NuGet | Dépôt GitHub de communauté |
|-|-|
| Bot.Builder.Community.Dialogs.Formflow | [BotBuilderCommunity/botbuilder-community-dotnet/libraries/Bot.Builder.Community.Dialogs.FormFlow](https://github.com/BotBuilderCommunity/botbuilder-community-dotnet/tree/master/libraries/Bot.Builder.Community.Dialogs.FormFlow) |

## <a name="additional-resources"></a>Ressources supplémentaires

- [Migrer un bot du kit SDK .NET v3 vers v4](conversion-framework.md)