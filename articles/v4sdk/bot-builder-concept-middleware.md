---
title: Middlewares | Microsoft Docs
description: Découvrez les middlewares (intergiciels) et leurs utilisations dans le SDK de bot.
keywords: middleware, pipeline de middlewares, court-circuit, utilisations de middlewares
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 05/24/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 38f1bced73251eea11be86a76963aeaf1ec0f718
ms.sourcegitcommit: 3cb288cf2f09eaede317e1bc8d6255becf1aec61
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/27/2018
ms.locfileid: "47389698"
---
# <a name="middleware"></a>Middlewares

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Les middlewares sont simplement une classe qui se trouve entre l’adaptateur et la logique de votre bot, et qui est ajoutée à la collection de middlewares de votre adaptateur durant l’initialisation. Le SDK vous permet d’écrire vos propres middlewares ou d’ajouter des composants réutilisables de middlewares créés par d’autres. Toute activité qui entre dans votre bot ou qui en sort transite par votre intergiciel.

L’adaptateur traite les activités entrantes et les dirige via le pipeline de middlewares de bot vers la logique de votre bot et inversement. Comme chaque activité entre et sort du bot, chaque middleware peut effectuer une inspection ou une action sur l’activité, avant comme après l’exécution de la logique de bot.

Avant d’aborder les middlewares, il est important de comprendre les [bots en général](~/v4sdk/bot-builder-basics.md) et [comment ils traitent les activités](~/v4sdk/bot-builder-concept-activity-processing.md).

## <a name="uses-for-middleware"></a>Utilisations des middlewares
La question suivante revient souvent : « Quand dois-je implémenter des actions avec mon intergiciel plutôt qu’en utilisant ma logique de bot normale ? » Un intergiciel vous offre des opportunités supplémentaires pour interagir avec le flux de conversation de vos utilisateurs avant et après le traitement de chaque _tour_ de la conversation. Un intergiciel vous permet également de stocker et de récupérer des informations concernant la conversation et d’appeler une logique de traitement supplémentaire quand cela est nécessaire. Vous trouverez ci-dessous des scénarios communs qui présentent les situations où un intergiciel peut être utile.

### <a name="looking-at-or-acting-on-every-activity"></a>Inspection et action sur chaque activité
Nombreux sont les cas où votre bot doit intervenir sur chaque activité, ou sur chaque activité d’un certain type. Par exemple, vous souhaitez peut-être journaliser chaque activité de message que votre bot reçoit ou fournir une réponse de repli si le bot n’a pas généré de réponse à l’occasion de ce tour. Avec leur capacité à agir à la fois avant et après l’exécution du reste de la logique de bot, les middlewares sont parfaits pour cela.

### <a name="modifying-or-enhancing-the-turn-context"></a>Modification ou amélioration du contexte de tour
Certaines conversations peuvent être beaucoup plus fructueuses si le bot a plus d’informations que ce qui est fourni dans l’activité. Dans ce cas, les intergiciels peuvent examiner les informations d’état de conversation dont ils disposent jusqu’ici, interroger une source de données externe et l’ajouter à l’objet de [contexte de tour](bot-builder-concept-activity-processing.md#turn-context) avant de transmettre l’exécution à la logique du bot. 

Le Kit de développement logiciel (SDK) définit un intergiciel de journalisation qui peut enregistrer des activités entrantes et sortantes, mais vous pouvez également définir votre propre intergiciel.

## <a name="the-bot-middleware-pipeline"></a>Pipeline de middlewares de bot
Pour chaque activité, l’adaptateur appelle les intergiciels dans l’ordre dans lequel vous les avez ajoutés. L’adaptateur transmet l’objet de contexte pour le tour et un délégué _next_, puis le middleware appelle le délégué pour passer le contrôle au middleware suivant dans le pipeline. Les middlewares peuvent également effectuer des opérations une fois que le délégué _next_ a retourné le contrôle avant la fin de la méthode. Vous pouvez considérer que chaque objet de middleware a une occasion unique d’agir par rapport aux objets de middleware qui le suivent dans le pipeline.

Par exemple : 

- Le gestionnaire de tour du 1er objet de middleware exécute du code avant d’appeler _next_.
  - Le gestionnaire de tour du 2e objet de middleware exécute du code avant d’appeler _next_.
    - Le gestionnaire de tour du bot s’exécute et retourne le contrôle.
  - Le gestionnaire de tour du 2e objet de middleware exécute tout code restant avant de retourner le contrôle.
- Le gestionnaire de tour du 1er objet de middleware exécute tout code restant avant de retourner le contrôle.

Si le middleware n’appelle pas le délégué next, l’adaptateur n’appelle aucun des gestionnaires de tour de middleware ou de bot suivants, ce qui entraîne un court-circuitage du pipeline.

Une fois le pipeline de middlewares de bot terminé, le tour prend fin et le contexte de tour se retrouve hors de l’étendue.

Les middlewares ou le bot peuvent générer des réponses et inscrire des gestionnaires d’événements de réponse, mais n’oubliez pas que les réponses sont gérées dans des processus distincts.

## <a name="order-of-middleware"></a>Ordre des middlewares
Étant donné que l’ordre dans lequel les middlewares sont ajoutés détermine l’ordre dans lequel ils traitent une activité, il est important de décider de la séquence d’ajout des middlewares.

> [!NOTE]
> Vous disposerez ainsi d’un modèle commun qui fonctionne pour la plupart des bots, mais veillez à prendre en compte la façon dont chaque middleware est susceptible d’interagir avec les autres en fonction de votre situation.

Les premiers éléments de votre pipeline de middlewares doivent probablement être ceux qui prennent en charge les tâches de niveau inférieur systématiquement utilisées. Citons par exemple la journalisation, la gestion des exceptions et la traduction. Leur classement peut varier en fonction de vos besoins, par exemple, si vous souhaitez que le message entrant soit d’abord traduit, avant que les messages ne soient stockés, ou si le stockage des messages doit intervenir en premier, ce qui peut signifier que les messages stockés ne sont pas destinés à être traduits.

Les derniers éléments de votre pipeline de middlewares doivent être des middlewares propres au bot, c’est-à-dire des middlewares que vous implémentez pour traiter tout message envoyé à votre bot. Si vos middlewares utilisent des informations d’état ou d’autres informations définies dans le contexte de bot, ajoutez-les au pipeline de middlewares après les middlewares qui modifient l’état ou le contexte.

## <a name="short-circuiting"></a>Court-circuitage
Une idée importante au sujet des intergiciels (et des [gestionnaires de réponses](./bot-builder-concept-activity-processing.md#response-event-handlers)) est le _court-circuitage_. Un intergiciel (ou un gestionnaire de réponses) doit transmettre l’exécution en appelant son délégué _next_ si l’exécution doit se poursuivre à travers les couches qui le suivent.  Si le délégué next n’est pas appelé au sein de cet intergiciel (ou gestionnaire de réponses), le pipeline associé est victime d’un court-circuitage, empêchant l’exécution des couches suivantes. Cela signifie que toute la logique de bot et tous les intergiciels dans le pipeline sont ignorés. Il existe une différence subtile entre le court-circuitage d’un tour par votre intergiciel ou par votre gestionnaire de réponses.

Quand l’intergiciel court-circuite un tour, votre gestionnaire de tours de bot n’est pas appelé, mais l’ensemble du code de l’intergiciel exécuté avant ce point dans le pipeline continue d’être exécuté jusqu’à la fin. 

Pour les gestionnaires d’événements, le fait de ne pas appeler _next_ signifie que l’événement est annulé, résultat qui diffère considérablement de la logique ignorant les middlewares. En ne traitant pas le reste de l’événement, l’adaptateur ne l’envoie jamais.

> [!TIP]
> Si vous court-circuitez un événement de réponse, tel que `SendActivities`, assurez-vous qu’il s’agit du comportement que vous envisagez. Sinon, cela peut compliquer la correction des bogues.

## <a name="additional-resources"></a>Ressources supplémentaires
Vous pouvez observer l’intergiciel d’enregistreur d’événements de transcription, tel qu’implémenté dans le Kit de développement logiciel (SDK) Bot Builder [[C#](https://github.com/Microsoft/botbuilder-dotnet/blob/master/libraries/Microsoft.Bot.Builder/TranscriptLoggerMiddleware.cs)|[JS](https://github.com/Microsoft/botbuilder-js/blob/master/libraries/botbuilder-core/src/transcriptLogger.ts)].
