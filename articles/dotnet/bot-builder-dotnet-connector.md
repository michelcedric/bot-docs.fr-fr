---
title: Envoyer et recevoir des activités | Microsoft Docs
description: Découvrez comment échanger des informations avec un utilisateur sur différents canaux en utilisant le service Connector via le kit SDK Bot Framework pour .NET.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 0407ec0d90c58e10aa14616e2aa9205bb8840d55
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/11/2019
ms.locfileid: "54225224"
---
# <a name="send-and-receive-activities"></a>Envoyer et recevoir des activités

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

Le connecteur d’infrastructure Bot fournit une API REST unique qui permet à un robot de communiquer sur plusieurs canaux tels que Skype, Email, Slack et bien plus encore. Il facilite la communication entre bot et utilisateur en relayant les messages du bot au canal et du canal au bot. 

Cet article explique comment utiliser le connecteur via le kit SDK Bot Framework pour .NET pour échanger des informations entre le bot et l’utilisateur sur un canal. 

> [!NOTE]
> Bien qu’il soit possible de construire un bot en utilisant exclusivement les techniques décrites dans cet article, le kit SDK Bot Framework propose des fonctionnalités supplémentaires, notamment les [dialogues](bot-builder-dotnet-dialogs.md) et [FormFlow](bot-builder-dotnet-formflow.md) qui peuvent simplifier la gestion du flux et de l’état des conversations et faciliter l’incorporation de services cognitifs comme la compréhension du langage.

## <a name="create-a-connector-client"></a>Créer une classe ConnectorClient

La classe [ConnectorClient][ConnectorClient] contient les méthodes qu’un bot utilise pour communiquer avec un utilisateur sur un canal. Lorsque votre robot reçoit un objet <a href="https://docs.botframework.com/en-us/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html" target="_blank">Activity</a> du Connecteur, il doit utiliser le `ServiceUrl` spécifié pour cette activité afin de créer le client de connecteur qu’il utilisera par la suite pour générer une réponse. 

[!code-csharp[Create connector client](../includes/code/dotnet-send-and-receive.cs#createConnectorClient)]

> [!TIP]
> Étant donné que le point de terminaison d’un canal peut ne pas être stable, votre bot doit transmettre les communications au point de terminaison indiqué par le connecteur dans l’objet `Activity` chaque fois que possible (plutôt que de s’appuyer sur un point de terminaison mis en cache). 
>
> Si votre robot doit lancer la conversation, il peut utiliser un point de terminaison mis en cache pour le canal spécifié (dans la mesure où il n’y aura aucun objet `Activity` entrant dans ce scénario), mais il doit actualiser régulièrement les points de terminaison mis en cache. 

## <a id="create-reply"></a> Créer une réponse

Le connecteur utilise un objet [Activity](bot-builder-dotnet-activities.md) pour faire passer des informations dans les deux sens entre le bot et le canal (l’utilisateur). Chaque activité contient des informations utilisées pour acheminer le message vers la destination appropriée, ainsi que des informations sur l’auteur du message (propriété `From`), le contexte du message et le destinataire du message (propriété `Recipient`).

Lorsque votre bot reçoit une activité de la part du connecteur, la propriété `Recipient` de l’activité entrante spécifie l’identité du bot dans cette conversation. Étant donné que certains canaux (par exemple, Slack) assignent au bot une nouvelle identité lorsqu’il est ajouté à une conversation, le bot devrait toujours utiliser la valeur de la propriété `Recipient` de l’activité entrante comme valeur de la propriété `From` dans sa réponse.

Bien que vous puissiez vous-même créer et initialiser l’objet `Activity` sortant à partir de zéro, le kit SDK Bot Framework offre un moyen plus simple de créer une réponse. À l’aide de la méthode `CreateReply` de l’activité entrante, spécifiez simplement le texte du message de réponse ; l’activité sortante est créée avec les propriétés `Recipient`, `From`, et `Conversation` renseignées automatiquement.

[!code-csharp[Create reply](../includes/code/dotnet-send-and-receive.cs#createReply)]

## <a name="send-a-reply"></a>Envoyer une réponse

Une fois que vous avez créé une réponse, vous pouvez l’envoyer en appelant la méthode `ReplyToActivity` du client de connecteur. Le connecteur fournira la réponse à l’aide de la sémantique de canal appropriée. 

[!code-csharp[Send reply](../includes/code/dotnet-send-and-receive.cs#sendReply)]

> [!TIP]
> Si votre robot répond au message d’un utilisateur, utilisez toujours la méthode `ReplyToActivity`.

## <a name="send-a-non-reply-message"></a>Envoyer un message (non-réponse) 

Si votre robot fait partie d’une conversation, il peut envoyer un message qui n’est pas une réponse directe à un message de l’utilisateur en appelant la méthode `SendToConversation`. 

[!code-csharp[Send non-reply message](../includes/code/dotnet-send-and-receive.cs#sendNonReplyMessage)]

Vous pouvez utiliser la méthode `CreateReply` pour initialiser le nouveau message (qui définirait automatiquement les propriétés `Recipient`, `From`, et `Conversation` pour le message). Vous pouvez également utiliser la méthode `CreateMessageActivity` pour créer le nouveau message et définir toutes les valeurs de la propriété vous-même.

> [!NOTE]
> L’infrastructure Bot n’impose aucune restriction concernant le nombre de messages qu’un robot est susceptible d’envoyer. Toutefois, la plupart des canaux appliquent des limitations de requêtes pour empêcher les bots d’envoyer un grand nombre de messages dans un court laps de temps. En outre, si le bot envoie plusieurs messages successifs rapidement, le canal risque de ne pas toujours afficher les messages dans le bon ordre.

## <a name="start-a-conversation"></a>Démarrer une conversation

Il peut arriver que votre robot ait besoin de démarrer une conversation avec un ou plusieurs utilisateurs. Vous pouvez démarrer une conversation en appelant la méthode `CreateDirectConversation` (pour une conversation privée avec un seul utilisateur) ou la méthode `CreateConversation` (pour une conversation de groupe avec plusieurs utilisateurs) afin de récupérer un objet `ConversationAccount`. Ensuite, créez le message et envoyez-le en appelant la méthode `SendToConversation`. Pour utiliser soit la méthode `CreateDirectConversation` ou la méthode `CreateConversation`, vous devez d’abord [créer le client du connecteur](#create-a-connector-client) à l’aide des URL de service du canal cible (que vous pouvez récupérer à partir du cache, si vous les avez conservées des précédents messages). 

> [!NOTE]
> Tous les canaux ne prennent pas en charge les conversations de groupe. Pour déterminer si un canal prend en charge les conversations de groupe, consultez la documentation du canal.

Cet exemple de code utilise la méthode `CreateDirectConversation` pour créer une conversation privée avec un seul utilisateur.

[!code-csharp[Start private conversation](../includes/code/dotnet-send-and-receive.cs#startPrivateConversation)]

Cet exemple de code utilise la méthode `CreateConversation` pour créer une conversation de groupe avec plusieurs utilisateurs.

[!code-csharp[Start group conversation](../includes/code/dotnet-send-and-receive.cs#startGroupConversation)]

## <a name="additional-resources"></a>Ressources supplémentaires

- [Vue d’ensemble des activités](bot-builder-dotnet-activities.md)
- [Créer des messages](bot-builder-dotnet-create-messages.md)
- <a href="/dotnet/api/?view=botbuilder-3.11.0" target="_blank">Informations de référence sur le kit SDK Bot Framework pour .NET</a>
- <a href="https://docs.botframework.com/en-us/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html" target="_blank">Classe Activité</a>
- <a href="/dotnet/api/microsoft.bot.connector.connectorclient" target="_blank">Classe ConnectorClient</a>

[ConnectorClient]: /dotnet/api/microsoft.bot.connector.connectorclient
