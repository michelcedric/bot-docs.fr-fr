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
ms.openlocfilehash: 9986ac7d46acfa94694456d653b91dd66c1f55f0
ms.sourcegitcommit: f95702d27abbd242c902eeb218d55a72df56ce56
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/19/2018
ms.locfileid: "39300592"
---
# <a name="middleware"></a>Middlewares

[!INCLUDE [pre-release-label](~/includes/pre-release-label.md)]

Les middlewares sont simplement une classe qui se trouve entre l’adaptateur et la logique de votre bot, et qui est ajoutée à la collection de middlewares de votre adaptateur durant l’initialisation. Le SDK vous permet d’écrire vos propres middlewares ou d’ajouter des composants réutilisables de middlewares créés par d’autres. Que pouvez-vous faire dans les middlewares ? Pratiquement tout... Toute activité qui entre dans votre bot ou qui en sort transite par les middlewares.

L’adaptateur traite les activités entrantes et les dirige via le pipeline de middlewares de bot vers la logique de votre bot et inversement. Comme chaque activité entre et sort du bot, chaque middleware peut effectuer une inspection ou une action sur l’activité, avant comme après l’exécution de la logique de bot.

Avant d’aborder les middlewares, il est important de comprendre les [bots en général](~/v4sdk/bot-builder-basics.md) et [comment ils traitent les activités](~/v4sdk/bot-builder-concept-activity-processing.md).

## <a name="uses-for-middleware"></a>Utilisations des middlewares

La question récurrente est : que doivent traiter les middlewares par rapport à la logique de votre bot ? Les middlewares étant assez flexibles, tout dépend de vous. Nous allons aborder quelques bons usages des middlewares.

### <a name="looking-at-or-acting-on-every-activity"></a>Inspection et action sur chaque activité

Nombreux sont les cas où notre bot doit intervenir sur chaque activité, ou sur chaque activité d’un certain type. Par exemple, nous souhaiterons peut-être journaliser chaque activité de message que notre bot reçoit ou fournir une réponse de repli si le bot n’a pas généré de réponse à l’occasion de ce tour. Avec leur capacité à agir à la fois avant et après l’exécution du reste de la logique de bot, les middlewares sont parfaits pour cela.

### <a name="modifying-or-enhancing-the-turn-context"></a>Modification ou amélioration du contexte de tour

Certaines conversations peuvent être beaucoup plus fructueuses si le bot a plus d’informations que ce qui est fourni dans l’activité. Dans ce cas, les middlewares peuvent examiner les informations d’état de conversation dont ils disposent jusqu’ici, interroger une source de données externe et les ajouter à l’objet de contexte avant de passer l’exécution à la logique du bot.
Par exemple, les middlewares peuvent identifier les détails de la conversation, tels que l’ID et l’état de conversation, puis interroger un service d’annuaire pour obtenir des informations. Les middlewares peuvent ajouter l’objet utilisateur reçu de cette requête externe à l’objet de contexte, puis le transmettre, fournissant davantage de données sur l’utilisateur et permettant au bot de mieux gérer la demande.

Les middlewares peuvent parfaitement servir dans les deux utilisations ci-dessus, ou dans une autre utilisation ; tout dépend de la façon dont vous souhaitez que votre bot soit structuré et de la finalité de ce dernier.
Les middlewares peuvent établir ou conserver l’état, réagir aux demandes entrantes ou court-circuiter le pipeline.
Voici certaines situations propices à l’utilisation de middlewares :

- **Stockage ou persistance** : les middlewares permettent de stocker ou de conserver les valeurs après un tour ou de mettre à jour une valeur basée sur ce qui est arrivé pendant ce tour.
- **Erreur de gestion ou échec normal** : si une exception est levée, les middlewares peuvent envoyer un message convivial pour cette exception.
- **Traduction** : ces middlewares peuvent détecter et convertir les messages entrants et configurer les messages sortants afin qu’ils soient traduits dans la langue entrante détectée.

Vous pouvez définir vos propres middlewares, ou bien le SDK définit certains middlewares qui représentent des composants indépendants des applications, comme par exemple :

- Les middlewares d’état, qui conservent et restaurent les informations d’état sur l’objet de contexte. Le SDK fournit les middlewares de conversation et d’état utilisateur.
- Les middlewares de traduction, qui peuvent reconnaître la langue d’entrée et traduire dans une autre langue, telle qu’une langue comprise par votre application.
- Les middlewares de journalisation, qui enregistrent les activités entrantes et sortantes.

## <a name="the-bot-middleware-pipeline"></a>Pipeline de middlewares de bot

Pour chaque activité, l’adaptateur appelle les middlewares dans **l’ordre dans lequel vous les avez ajoutés**. L’adaptateur transmet l’objet de contexte pour le tour et un délégué _next_, puis le middleware appelle le délégué pour passer le contrôle au middleware suivant dans le pipeline. Les middlewares peuvent également effectuer des opérations une fois que le délégué _next_ a retourné le contrôle avant la fin de la méthode. Vous pouvez considérer que chaque objet de middleware a une occasion unique d’agir par rapport aux objets de middleware qui le suivent dans le pipeline.

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

Les premiers éléments de votre pipeline de middlewares doivent probablement être ceux qui prennent en charge les tâches de niveau inférieur systématiquement utilisées. Citons par exemple la journalisation, la gestion des exceptions, la gestion des états et la traduction. Leur classement peut varier en fonction de vos besoins, par exemple, si vous souhaitez que le message entrant soit d’abord traduit, avant que les exceptions ne soient gérées, ou si la gestion des exceptions doit intervenir en premier, ce qui peut signifier que les messages d’exception ne sont pas destinés à être traduits.

Les derniers éléments de votre pipeline de middlewares doivent être des middlewares propres au bot, c’est-à-dire des middlewares que vous implémentez pour traiter tout message envoyé à votre bot. Si vos middlewares utilisent des informations d’état ou d’autres informations définies dans le contexte de bot, ajoutez-les au pipeline de middlewares après les middlewares qui modifient l’état ou le contexte.

## <a name="short-circuiting"></a>Court-circuitage

Une idée importante autour des middlewares (et des [gestionnaires d’événements](~/v4sdk/bot-builder-concept-activity-processing.md#response-event-handlers)) est le _court-circuitage_. Un middleware (ou un gestionnaire) doit transmettre l’exécution en appelant son délégué _next_ si l’exécution doit se poursuivre à travers les couches qui le suivent.  Si le délégué next n’est pas appelé au sein de ce middleware (ou gestionnaire), le pipeline actuel est victime d’un court-circuitage, empêchant l’exécution des couches suivantes. Cela signifie que toute la logique de bot et tout middleware en aval dans le pipeline sont ignorés.

Pour les gestionnaires d’événements, le fait de ne pas appeler _next_ signifie que l’événement est annulé, résultat qui diffère considérablement de la logique ignorant les middlewares. En ne traitant pas le reste de l’événement, l’adaptateur ne l’envoie jamais.

> [!TIP]
> Si vous court-circuitez un événement, tel que `SendActivities`, assurez-vous qu’il s’agit du comportement que vous envisagez. Sinon, cela peut entraîner des bogues très déroutants.

## <a name="next-steps"></a>Étapes suivantes

Certains concepts clés d’un bot n’ayant plus de secret pour vous, découvrons la façon dont un bot peut envoyer des messages proactifs.

> [!div class="nextstepaction"]
> [Messagerie proactive](~/v4sdk/bot-builder-proactive-messages.md)
