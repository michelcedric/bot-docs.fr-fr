---
title: Traitement des activités | Microsoft Docs
description: Découvrez le traitement des activités dans le SDK de bot.
keywords: adaptateur de bot, middleware (intergiciel) personnalisé, court-circuit, repli, gestionnaires d’événements
author: jonathanfingold
ms.author: jonathanfingold
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 09/13/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 6ba140324fb6b50c2d6696aae6e4bd3e8824fd96
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2018
ms.locfileid: "49997566"
---
# <a name="activity-processing"></a>Traitement des activités

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Le bot et l’utilisateur interagissent et échangent des informations par le biais des activités. Chaque activité reçue par votre application de bot est passée à un adaptateur de bot, qui transmet les informations de l’activité à la logique de votre bot, avant d’envoyer les réponses à l’utilisateur. La réception d’une activité, puis son traitement par l’intermédiaire de votre bot, s’appelle un tour. Il s’agit d’un cycle complet du bot. Un tour s’achève lorsque l’exécution est effectuée dans son intégralité, l’activité entièrement traitée et toutes les couches du bot terminées.

Les activités, en particulier celles qui sont [envoyées à partir d’un bot](#generating-responses) pendant un tour de bot, sont traitées de façon asynchrone. C’est une partie nécessaire de la génération d’un bot ; si vous souhaitez rafraîchir vos connaissances sur la façon dont tout cela fonctionne, consultez [Programmation asynchrone pour .NET](https://docs.microsoft.com/en-us/dotnet/csharp/async) ou [Programmation asynchrone pour JavaScript](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function) selon le langage de votre choix.

## <a name="the-bot-adapter"></a>Adaptateur de bot

L’adaptateur de bot encapsule le processus d’authentification et envoie les activités au service Bot Connector et en reçoit de celui-ci. Quand votre bot reçoit une activité, l’adaptateur rassemble tout ce qui concerne cette activité, crée un [objet de contexte](#turn-context) pour le tour, le transmet à la logique d’application de votre bot et envoie les réponses générées par celui-ci au canal de l’utilisateur.

## <a name="authentication"></a>Authentification

L’adaptateur authentifie chaque activité entrante que reçoit l’application, à l’aide des informations issues de l’activité et de l’en-tête `Authentication` de la demande REST. L’adaptateur utilise un objet de connecteur et les informations d’identification de votre application pour authentifier les activités sortantes à destination de l’utilisateur.

L’authentification du service Bot Connector utilise des jetons `Bearer` JWT (JSON Web Token), ainsi que **l’ID d’application Microsoft** et le **mot de passe d’application Microsoft** qu’Azure crée pour vous quand vous créez un service de bot ou inscrivez votre bot. Votre application a besoin de ces informations d’identification au moment de l’initialisation, pour permettre à l’adaptateur d’authentifier le trafic.

> [!NOTE]
> Si vous exécutez ou testez votre bot localement, par exemple, à l’aide de l’émulateur Bot Framework, vous pouvez effectuer cela sans configurer l’adaptateur pour authentifier le trafic vers et depuis votre bot.

## <a name="turn-context"></a>Contexte de tour

Quand un adaptateur reçoit une activité, il génère un objet de **contexte de tour**, qui fournit des informations sur l’activité entrante, l’expéditeur et le destinataire, le canal, la conversation et d’autres données nécessaires pour traiter l’activité. L’adaptateur transmet ensuite cet objet de contexte au bot. L’objet de contexte est conservé tout au long d’un tour et fournit des informations sur les éléments suivants :

* Conversation : identifie la conversation et inclut des informations sur le bot et l’utilisateur participant à la conversation.
* Activité : les demandes et réponses dans une conversation sont toutes des types d’activités. Ce contexte fournit des informations sur l’activité entrante, y compris les informations de routage, des informations sur le canal, la conversation, l’expéditeur et le destinataire.
* Informations personnalisées : si vous étendez votre bot en implémentant des middlewares (intergiciels) ou au sein de la logique de votre bot, vous pouvez mettre à disposition des informations supplémentaires à chaque tour.

Vous pouvez également utiliser l’objet de contexte pour envoyer une réponse à l’utilisateur et obtenir une référence à l’adaptateur<!-- to create a new conversation or continue an existing one-->.

> [!NOTE]
> Votre application et l’adaptateur gèrent les demandes de façon asynchrone ; toutefois, votre logique métier n’a pas besoin d’être contrôlée par un procédé de requête-réponse.

## <a name="middleware"></a>Middlewares

Vous pouvez ajouter des middlewares à l’adaptateur. Les middlewares et la logique de bot utilisent l’objet de contexte pour récupérer les informations sur l’activité et agir en conséquence. Les middlewares et le bot peuvent également mettre à jour ou ajouter des informations à l’objet de contexte, par exemple pour effectuer le suivi de l’état pour un tour, une conversation ou une autre étendue. Pour plus d’informations sur les intergiciels, consultez [l’article consacré aux intergiciels](~/v4sdk/bot-builder-concept-middleware.md).

## <a name="generating-responses"></a>Génération de réponses

L’objet de contexte fournit des méthodes de réponse aux activités pour permettre au code de répondre à une activité :

* Les méthodes _d’envoi d’une activité_ et _d’envoi d’activités_ envoient une ou plusieurs activités à la conversation.
* Si le canal la prend en charge, la méthode de _mise à jour d’une activité_ met à jour une activité au sein de la conversation.
* Si le canal la prend en charge, la méthode de _suppression d’une activité_ supprime une activité de la conversation.

Chaque méthode de réponse s’exécute dans un processus asynchrone. Quand elle est appelée, la méthode de réponse à une activité clone la liste de [gestionnaires d’événements](#response-event-handlers) associée avant de commencer à appeler les gestionnaires, ce qui signifie qu’elle contient chaque gestionnaire ajouté à ce stade, mais rien qui est ajouté après le démarrage du processus.

Cela signifie également que l’ordre de vos réponses n’est pas garanti, en particulier quand une tâche est plus complexe qu’une autre. Si votre bot peut générer plusieurs réponses à une activité entrante, vérifiez qu’elles ont du sens quel que soit l’ordre dans lequel l’utilisateur les reçoit.

> [!IMPORTANT]
> Le thread qui traite le tour de bot principal se charge de supprimer l’objet de contexte quand il prend fin. Si une réponse (y compris ses gestionnaires) prend un certain temps et essaie d’agir sur l’objet de contexte, elle risque de recevoir une erreur `Context was disposed`. **Veillez à exécuter une opération `await` sur tous les appels d’activité** afin que le thread principal attende l’activité générée avant de terminer son traitement et de supprimer le contexte de tour.

## <a name="response-event-handlers"></a>Gestionnaires d’événements de réponse

En plus de la logique de bot et de middleware, vous pouvez ajouter des gestionnaires de réponse (parfois également appelés gestionnaires d’événements ou gestionnaires d’événements d’activité) à l’objet de contexte. Ces gestionnaires sont appelés quand la [réponse](#generating-responses) associée se produit sur l’objet de contexte actuel, avant l’exécution de la réponse proprement dite. Ces gestionnaires sont utiles quand vous savez que vous souhaitez faire quelque chose, avant ou après l’événement réel, pour toute activité de ce type pour le reste de la réponse actuelle.

> [!WARNING]
> Veillez à ne pas appeler une méthode de réponse d’activité à partir de son propre gestionnaire d’événements de réponse, par exemple, en appelant la méthode d’envoi d’activité depuis un gestionnaire _écoutant les envois d’activité_. Cela peut générer une boucle infinie.

Chaque nouvelle activité obtient un nouveau thread sur lequel s’exécuter. Quand le thread destiné à traiter l’activité est créé, la liste des gestionnaires de cette activité est copiée sur ce nouveau thread. Aucun gestionnaire ajouté après ce point n’est exécuté pour cet événement d’activité spécifique.

Les gestionnaires inscrits sur un objet de contexte sont traités de façon très semblable à la façon dont l’adaptateur gère le [pipeline d’intergiciels](~/v4sdk/bot-builder-concept-middleware.md#the-bot-middleware-pipeline). Concrètement, les gestionnaires sont appelés dans l’ordre dans lequel ils sont ajoutés, et l’appel du délégué _next_ transmet le contrôle au gestionnaire d’événements inscrit suivant. Si un gestionnaire n’appelle pas le délégué next, aucun des gestionnaires d’événements suivants n’est appelé ; l’événement est victime d’un [court-circuitage](~/v4sdk/bot-builder-concept-middleware.md#short-circuiting), ce qui empêche l’adaptateur d’envoyer la réponse au canal.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Middleware](~/v4sdk/bot-builder-concept-middleware.md)
