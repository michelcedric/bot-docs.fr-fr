---
title: Gérer les données d’état | Microsoft Docs
description: Découvrez comment stocker et récupérer les données d’état à l’aide du service Bot State.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: 1557941d4e5413108ea3ce788f7d5d684252b657
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39298747"
---
# <a name="manage-state-data"></a>Gérer les données d’état

Le service Bot State permet à votre bot de stocker et récupérer les données d’état qui sont associées à un utilisateur, à une conversation ou à un utilisateur spécifique dans le contexte d’une conversation donnée. Vous pouvez stocker jusqu’à 32 Ko de données pour chaque utilisateur sur un canal, pour chaque conversation sur un canal et pour chaque utilisateur dans le contexte d’une conversation sur un canal. Les données d’état peuvent servir à de nombreuses fins, par exemple pour déterminer l’endroit où une conversation précédente s’était arrêtée ou simplement pour accueillir un utilisateur régulier par son nom. Si vous stockez les préférences d’un utilisateur, vous pouvez utiliser ces informations pour personnaliser la prochaine conversation. Par exemple, vous pouvez avertir l’utilisateur de la publication d’un article sur un sujet qui l’intéresse ou bien l’informer de la disponibilité d’une date de rendez-vous. 

> [!IMPORTANT]
> L’API du service Bot Framework State n’est pas recommandée pour les environnements de production et peut être déconseillée dans une version ultérieure. Nous vous recommandons de mettre à jour le code de votre bot pour qu’il utilise le stockage en mémoire à des fins de test ou pour qu’il utilise l’une des **extensions Azure** pour les bots de production. Pour plus d’informations, consultez l’article **Gérer les données d’état** de l’implémentation [.NET](~/dotnet/bot-builder-dotnet-state.md) ou [Node](~/nodejs/bot-builder-nodejs-state.md).

## <a id="concurrency"></a> Concurrence des données

Pour contrôler la concurrence des données qui sont stockées à l’aide du service Bot State, l’infrastructure utilise l’étiquette d’entité (ETag) pour les requêtes `POST`. L’infrastructure n’utilise pas les en-têtes standard pour les ETags. À la place, le corps de la requête et de la réponse spécifie l’ETag à l’aide de la propriété `eTag`. 

Par exemple, si vous émettez une requête `GET` pour récupérer des données utilisateur à partir du magasin, la réponse contiendra la propriété `eTag`. Si vous modifiez les données et émettez une requête `POST` pour enregistrer les données mises à jour dans le magasin, votre requête peut inclure la propriété `eTag` qui spécifie la même valeur que celle que vous avez reçue précédemment dans la réponse `GET`. Si l’ETag spécifié dans votre requête `POST` correspond à la valeur actuelle dans le magasin, le serveur enregistre les données de l’utilisateur, répond par le message **HTTP 200 (Succès)** et spécifie une nouvelle valeur `eTag` dans le corps de la réponse. Si l’ETag spécifié dans votre requête `POST` ne correspond pas à la valeur actuelle dans le magasin, le serveur répond par le message **HTTP 412 (Échec de la précondition)** pour indiquer que les données de l’utilisateur dans le magasin ont changé depuis que vous les avez enregistrées ou récupérées pour la dernière fois.

> [!TIP]
> Une réponse `GET` inclut toujours une propriété `eTag`, mais vous n’avez pas besoin de spécifier la propriété `eTag` dans les requêtes `GET`. La définition de la propriété `eTag` sur un astérisque (`*`) signale que vous n’avez pas encore enregistré les données pour la combinaison canal-utilisateur-conversation spécifiée.
>
> La spécification de la propriété `eTag` dans les requêtes `POST` est facultative. 
> Si la concurrence n’est pas un problème pour votre bot, n’incluez pas la propriété `eTag` dans les requêtes `POST`. 

## <a id="save-user-data"></a> Enregistrer les données utilisateur

Pour enregistrer les données d’état relatives à un utilisateur sur un canal spécifique, émettez la requête suivante :

```http
POST /v3/botstate/{channelId}/users/{userId}
```

Dans cet URI de requête, remplacez **{channelId}** par l’ID du canal et remplacez **{userId}** par l’ID d’utilisateur sur ce canal. Les propriétés `channelId` et `from` figurant dans tous les messages que votre bot a précédemment reçus de l’utilisateur contiennent ces ID. Vous pouvez également choisir de mettre en cache ces valeurs dans un emplacement sécurisé pour être en mesure d’accéder par la suite aux données de l’utilisateur sans avoir à les extraire d’un message.

Définissez le corps de la requête sur un objet [BotData][BotData], dans lequel la propriété `data` spécifie les données que vous souhaitez enregistrer. Si vous utilisez des étiquettes d’entité pour le [contrôle d’accès concurrentiel](#concurrency), définissez la propriété `eTag` sur l’ETag que vous avez reçu dans la réponse la dernière fois que vous avez enregistré ou récupéré les données de l’utilisateur (selon l’opération la plus récente des deux). Si vous n’utilisez pas d’étiquettes d’entité pour la concurrence, n’incluez pas la propriété `eTag` dans la requête.

> [!NOTE]
> Cette requête `POST` ne remplace les données utilisateur dans le magasin que si l’ETag spécifié correspond à l’ETag du serveur, ou si aucun ETag n’est spécifié dans la requête.

### <a name="request"></a>Requête 

L’exemple ci-après présente une requête qui enregistre les données relatives à un utilisateur sur un canal spécifique. Dans cet exemple de requête, `https://smba.trafficmanager.net/apis` représente l’URI de base, qui peut être différent de celui des requêtes émises par votre bot. Pour plus d’informations sur la définition de l’URI de base, consultez l’article [API Reference](bot-framework-rest-connector-api-reference.md#base-uri) (Informations de référence sur l’API).

```http
POST https://smba.trafficmanager.net/apis/v3/botstate/abcd1234/users/12345678
Authorization: Bearer ACCESS_TOKEN
Content-Type: application/json
```

```json
{
    "data": [
        {
            "trail": "Lake Serene",
            "miles": 8.2,
            "difficulty": "Difficult",
        },
        {
            "trail": "Rainbow Falls",
            "miles": 6.3,
            "difficulty": "Moderate",
        }
    ],
    "eTag": "a1b2c3d4"
}
```

### <a name="response"></a>response 

La réponse contient un objet [BotData][BotData] avec une nouvelle valeur `eTag`.

## <a name="get-user-data"></a>Obtenir les données utilisateur

Pour obtenir les données d’état précédemment enregistrées pour l’utilisateur sur un canal spécifique, émettez la requête suivante :

```http
GET /v3/botstate/{channelId}/users/{userId}
```

Dans cet URI de requête, remplacez **{channelId}** par l’ID du canal et remplacez **{userId}** par l’ID d’utilisateur sur ce canal. Les propriétés `channelId` et `from` figurant dans tous les messages que votre bot a précédemment reçus de l’utilisateur contiennent ces ID. Vous pouvez également choisir de mettre en cache ces valeurs dans un emplacement sécurisé pour être en mesure d’accéder par la suite aux données de l’utilisateur sans avoir à les extraire d’un message.

### <a name="request"></a>Requête

L’exemple ci-après présente une requête qui obtient les données précédemment enregistrées pour l’utilisateur. Dans cet exemple de requête, `https://smba.trafficmanager.net/apis` représente l’URI de base, qui peut être différent de celui des requêtes émises par votre bot. Pour plus d’informations sur la définition de l’URI de base, consultez l’article [API Reference](bot-framework-rest-connector-api-reference.md#base-uri) (Informations de référence sur l’API).

```http
GET https://smba.trafficmanager.net/apis/v3/botstate/abcd1234/users/12345678
Authorization: Bearer ACCESS_TOKEN
Content-Type: application/json
```

### <a name="response"></a>response

La réponse contient un objet [BotData][BotData], dans lequel la propriété `data` contient les données que vous avez précédemment enregistrées pour l’utilisateur, et où la propriété `eTag` contient l’ETag que vous pouvez utiliser dans une requête ultérieure pour enregistrer les données de l’utilisateur. Si vous n’avez pas encore enregistré les données de l’utilisateur, la propriété `data` présente la valeur `null`, et la propriété `eTag` contient un astérisque (`*`).

Cet exemple présente la réponse à la requête `GET` :

```json
{
    "data": [
        {
            "trail": "Lake Serene",
            "miles": 8.2,
            "difficulty": "Difficult",
        },
        {
            "trail": "Rainbow Falls",
            "miles": 6.3,
            "difficulty": "Moderate",
        }
    ],
    "eTag": "xyz12345"
}
```

## <a name="save-conversation-data"></a>Enregistrer les données de conversation

Pour enregistrer les données d’état relatives à une conversation sur un canal spécifique, émettez la requête suivante :

```http
POST /v3/botstate/{channelId}/conversations/{conversationId}
```

Dans cet URI de requête, remplacez **{channelId}** par l’ID du canal et remplacez **{conversationId}** par l’ID de la conversation. Les propriétés `channelId` et `conversation` figurant dans tous les messages que votre bot a précédemment envoyés ou reçus dans le contexte de la conversation contiennent ces ID. Vous pouvez également choisir de mettre en cache ces valeurs dans un emplacement sécurisé pour être en mesure d’accéder par la suite aux données de la conversation sans avoir à les extraire d’un message.

Définissez le corps de la requête sur un objet [BotData][BotData], tel que décrit à la section [Enregistrer les données utilisateur](#save-user-data).

> [!IMPORTANT]
> Étant donné que l’opération [Supprimer des données utilisateur](#delete-user-data) ne supprime pas les données qui ont été stockées à l’aide de l’opération **Enregistrer les données de conversation**, vous ne devez PAS utiliser cette opération pour stocker les informations d’identification personnelle (PII) d’un utilisateur.

### <a name="response"></a>response

La réponse contient un objet [BotData][BotData] avec une nouvelle valeur `eTag`.

## <a name="get-conversation-data"></a>Obtenir les données de conversation

Pour obtenir les données d’état précédemment enregistrées pour une conversation sur un canal spécifique, émettez la requête suivante :

```http
GET /v3/botstate/{channelId}/conversations/{conversationId}
```

Dans cet URI de requête, remplacez **{channelId}** par l’ID du canal et remplacez **{conversationId}** par l’ID de la conversation. Les propriétés `channelId` et `conversation` figurant dans tous les messages que votre bot a précédemment envoyés ou reçus dans le contexte de la conversation contiennent ces ID. Vous pouvez également choisir de mettre en cache ces valeurs dans un emplacement sécurisé pour être en mesure d’accéder par la suite aux données de la conversation sans avoir à les extraire d’un message.

### <a name="response"></a>response

La réponse contient un objet [BotData][BotData] avec une nouvelle valeur `eTag`.

## <a name="save-private-conversation-data"></a>Enregistrer les données de conversation privées

Pour enregistrer les données d’état d’un utilisateur dans le contexte d’une conversation spécifique, émettez la requête suivante :

```http
POST /v3/botstate/{channelId}/conversations/{conversationId}/users/{userId}
```

Dans cet URI de requête, remplacez **{channelId}** par l’ID du canal, remplacez **{conversationId}** par l’ID de la conversation et remplacez **{userId}** par l’ID de l’utilisateur sur ce canal. Les propriétés `channelId`, `conversation` et `from` figurant dans tous les messages que votre bot a précédemment reçus de l’utilisateur dans le contexte de la conversation contiennent ces ID. Vous pouvez également choisir de mettre en cache ces valeurs dans un emplacement sécurisé pour être en mesure d’accéder par la suite aux données de la conversation sans avoir à les extraire d’un message.

Définissez le corps de la requête sur un objet [BotData][BotData], tel que décrit à la section [Enregistrer les données utilisateur](#save-user-data).

### <a name="response"></a>response

La réponse contient un objet [BotData][BotData] avec une nouvelle valeur `eTag`.

## <a name="get-private-conversation-data"></a>Obtenir les données de conversation privées

Pour obtenir les données d’état précédemment enregistrées pour un utilisateur dans le contexte d’une conversation spécifique, émettez la requête suivante : 

```http
GET /v3/botstate/{channelId}/conversations/{conversationId}/users/{userId}
```

Dans cet URI de requête, remplacez **{channelId}** par l’ID du canal, remplacez **{conversationId}** par l’ID de la conversation et remplacez **{userId}** par l’ID de l’utilisateur sur ce canal. Les propriétés `channelId`, `conversation` et `from` figurant dans tous les messages que votre bot a précédemment reçus de l’utilisateur dans le contexte de la conversation contiennent ces ID. Vous pouvez également choisir de mettre en cache ces valeurs dans un emplacement sécurisé pour être en mesure d’accéder par la suite aux données de la conversation sans avoir à les extraire d’un message.

### <a name="response"></a>response

La réponse contient un objet [BotData][BotData] avec une nouvelle valeur `eTag`.

## <a name="delete-user-data"></a>Supprimer des données utilisateur

Pour supprimer les données d’état relatives à un utilisateur sur un canal spécifique, émettez la requête suivante :

```http
DELETE /v3/botstate/{channelId}/users/{userId}
```
Dans cet URI de requête, remplacez **{channelId}** par l’ID du canal et remplacez **{userId}** par l’ID d’utilisateur sur ce canal. Les propriétés `channelId` et `from` figurant dans tous les messages que votre bot a précédemment reçus de l’utilisateur contiennent ces ID. Vous pouvez également choisir de mettre en cache ces valeurs dans un emplacement sécurisé pour être en mesure d’accéder par la suite aux données de l’utilisateur sans avoir à les extraire d’un message.

> [!IMPORTANT]
> Cette opération supprime les données qui ont été précédemment stockées pour l’utilisateur par le biais de l’opération [Enregistrer les données utilisateur](#save-user-data) ou de l’opération [Enregistrer les données de conversation privées](#save-private-conversation-data). Elle ne supprime PAS les données qui ont été précédemment stockées à l’aide de l’opération [Enregistrer les données de conversation](#save-conversation-data). Par conséquent, vous ne devez PAS utiliser l’opération **Enregistrer les données de conversation** pour stocker les informations d’identification personnelle (PII) d’un utilisateur.

Votre bot doit exécuter l’opération **Supprimer des données utilisateur** lorsqu’il reçoit une [Activité][Activity] du type [deleteUserData](bot-framework-rest-connector-activities.md#deleteuserdata) ou une activité du type [contactRelationUpdate](bot-framework-rest-connector-activities.md#contactrelationupdate) indiquant que le bot a été supprimé de la liste de contacts de l’utilisateur.

## <a name="additional-resources"></a>Ressources supplémentaires

- [Concepts clés](bot-framework-rest-connector-concepts.md)
- [Vue d’ensemble des activités](bot-framework-rest-connector-activities.md)

[BotData]: bot-framework-rest-connector-api-reference.md#botdata-object
[Activity]: bot-framework-rest-connector-api-reference.md#activity-object
