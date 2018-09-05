---
title: Authentifier les requêtes | Microsoft Docs
description: Découvrez comment authentifier les requêtes API dans les API Bot Connector et Bot State.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: 998586820a0489bc4cca1d25b53cb6ac8162c452
ms.sourcegitcommit: 0b2be801e55f6baa048b49c7211944480e83ba95
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/28/2018
ms.locfileid: "43115044"
---
# <a name="authentication"></a>Authentification

Votre robot communique avec le service Bot Connector en utilisant le protocole HTTP sur un canal sécurisé (SSL/TLS). Lorsque votre robot envoie une requête au service du connecteur, il doit inclure des informations lui permettant de vérifier son identité. De même, lorsque le service du connecteur envoie une requête à votre robot, il doit inclure des informations lui permettant vérifier son identité. Cet article décrit les technologies d’authentification et les exigences en matière d’authentification au niveau du service entre un robot et le service Bot Connector. Si vous créez votre propre code d’authentification, vous devez appliquer les procédures de sécurité décrites dans cet article pour que votre robot puisse échanger des messages avec le service Bot Connector.

> [!IMPORTANT]
> Si vous créez votre propre code d’authentification, il est essentiel que vous appliquiez correctement toutes les procédures de sécurité. En appliquant toutes les étapes de cet article, vous pouvez réduire le risque qu’un pirate puisse lire les messages envoyés à votre robot, envoyer des messages qui usurpent l’identité de votre robot et voler des clés secrètes. 

Si vous utilisez le [kit de développement logiciel pour .NET](../dotnet/bot-builder-dotnet-overview.md) ou le [kit de développement logiciel pour Node.js](../nodejs/index.md), vous n’avez pas besoin d’appliquer les procédures de sécurité décrites dans cet article, car le kit de développement logiciel le fait automatiquement pour vous. Il vous suffit de configurer votre projet avec l’identifiant de l’application et le mot de passe obtenus pour votre robot lors de [l’enregistrement](../bot-service-quickstart-registration.md). Le kit de développement logiciel s’occupe du reste.

> [!WARNING]
> En décembre 2016, la version 3.1 du protocole de sécurité Bot Framework a modifié plusieurs valeurs utilisées lors de la génération et de la validation des jetons. À la fin de l’automne 2017, la version 3.2 du protocole de sécurité Bot Framework a apporté des modifications aux valeurs utilisées lors de la génération et de la validation des jetons.
> Pour plus d’informations, consultez [Modifications du protocole de sécurité](#security-protocol-changes).

## <a name="authentication-technologies"></a>Technologies d’authentification

Quatre technologies d’authentification sont utilisées pour assurer la fiabilité entre un robot et Bot Connector :

| Technology | Description |
|----|----|
| **SSL/TLS** | La technologie SSL/TLS est utilisée pour toutes les connexions de service à service. Les certificats `X.509v3` permettent de déterminer l’identité de tous les services HTTPS. **Les clients doivent toujours vérifier les certificats de service pour s’assurer de leur fiabilité et de leur validité.** (Ce programme N’utilise PAS de certificats de clients.) |
| **OAuth 2.0** | Le service de connexion OAuth 2.0 au compte Microsoft (MSA)/AAD v2 permet de générer un jeton sécurisé pouvant être utilisé par un robot pour envoyer des messages. Il s’agit d’un jeton de service à service ; aucune connexion utilisateur n’est nécessaire. |
| **JSON Web Token (JWT)** | Les JWT permettent d’encoder les jetons qui sont envoyés vers et depuis le robot. **Les clients doivent vérifier tous les jetons JWT qu’ils reçoivent**, conformément aux exigences décrites dans cet article. |
| **Métadonnées OpenID** | Le service Bot Connector publie une liste des jetons valides qu’il utilise pour signer ses propres jetons JWT sur les métadonnées OpenID à un point de terminaison statique bien défini. |

Cet article décrit comment utiliser ces technologies à l’aide des standards HTTPS et JSON. Aucun kit de développement logiciel spécial n’est requis, bien que vous puissiez trouver de l’utilité aux aides pour OpenID, etc.

## <a id="bot-to-connector"></a> Authentifier les requêtes de votre robot auprès du service Bot Connector

Pour communiquer avec le service Bot Connector, vous devez indiquer un jeton d’accès dans l’en-tête `Authorization` de chaque requête API, au format suivant : 

```http
Authorization: Bearer ACCESS_TOKEN
```

Ce diagramme montre les étapes de l’authentification du robot vers le connecteur :

![Authentifiez-vous auprès du service de connexion MSA, puis auprès du robot](../media/connector/auth_bot_to_bot_connector.png)

> [!IMPORTANT]
> Si vous ne l’avez pas déjà fait, vous devez [enregistrer votre robot](../bot-service-quickstart-registration.md) auprès de Bot Framework pour obtenir son identifiant d’application et son mot de passe. Pour demander un jeton d’accès, vous devez disposer de l’identifiant d’application et du mot de passe du robot.

### <a name="step-1-request-an-access-token-from-the-msaaad-v2-login-service"></a>Étape 1 : Demander un jeton d’accès au service de connexion MSA/AAD v2

Pour demander un jeton d’accès au service de connexion MSA/AAD v2, envoyez la requête suivante en remplaçant **MICROSOFT-APP-ID** et **MICROSOFT-APP-PASSWORD** par l’identifiant d’application et le mot de passe que vous avez obtenus lorsque vous avez [enregistré](../bot-service-quickstart-registration.md) votre robot avec Bot Framework.

```http
POST https://login.microsoftonline.com/botframework.com/oauth2/v2.0/token
Host: login.microsoftonline.com
Content-Type: application/x-www-form-urlencoded

grant_type=client_credentials&client_id=MICROSOFT-APP-ID&client_secret=MICROSOFT-APP-PASSWORD&scope=https%3A%2F%2Fapi.botframework.com%2F.default
```

### <a name="step-2-obtain-the-jwt-token-from-the-msaaad-v2-login-service-response"></a>Étape 2 : Récupérer le jeton JWT dans la réponse du service de connexion MSA/AAD v2

Si votre application est autorisée par le service de connexion MSA/AAD v2, le corps de réponse JSON indiquera votre clé d’accès, son type et son délai d’expiration (en secondes). 

Lorsque vous ajoutez le jeton à l’en-tête `Authorization` d’une requête, vous devez utiliser la valeur exacte indiquée dans cette réponse (ne pas échapper ou encoder la valeur du jeton). Le jeton d’accès est valide jusqu’à son expiration. Pour éviter que l’expiration des jetons n’ait une incidence sur le fonctionnement de votre robot, vous pouvez choisir de les mettre en cache et de les actualiser de façon proactive.

Cet exemple montre une réponse du service de connexion MSA/AAD v2 :

```http
HTTP/1.1 200 OK
... (other headers) 
```

```json
{
    "token_type":"Bearer",
    "expires_in":3600,
    "ext_expires_in":3600,
    "access_token":"eyJhbGciOiJIUzI1Ni..."
}
```

### <a name="step-3-specify-the-jwt-token-in-the-authorization-header-of-requests"></a>Étape 3 : Indiquer le jeton JWT dans l’en-tête d’autorisation des requêtes

Lorsque vous envoyez une requête API au service Bot Connector, indiquez le jeton d’accès dans l’en-tête `Authorization` de la requête, au format suivant :

```http
Authorization: Bearer ACCESS_TOKEN
```

Toutes les requêtes que vous envoyez au service Bot Connector doivent inclure le jeton d’accès dans l’en-tête `Authorization`. Si le jeton est correctement formé, qu’il n’est pas expiré et qu’il a été créé par le service de connexion MSA/AAD v2, le service Bot Connector autorise la requête. Des vérifications supplémentaires sont effectuées pour s’assurer que le jeton appartient au robot qui a envoyé la requête.

L’exemple suivant montre comment indiquer le jeton d’accès dans l’en-tête `Authorization` de la requête. 

```http
POST https://smba.trafficmanager.net/apis/v3/conversations/12345/activities 
Authorization: Bearer eyJhbGciOiJIUzI1Ni...
    
(JSON-serialized Activity message goes here)
```

> [!IMPORTANT]
> Indiquez le jeton JWT uniquement dans l’en-tête `Authorization` des requêtes que vous envoyez au service Bot Connector. N’envoyez PAS le jeton sur des canaux non sécurisés et NE l’incluez PAS dans les requêtes HTTP que vous envoyez à d’autres services. Le jeton JWT que vous obtenez du service de connexion MSA/AAD v2 est comparable à un mot de passe. Il doit être manipulé avec le plus grand soin. Toutes les personnes qui disposent du jeton peuvent s’en servir pour effectuer des opérations au nom de votre robot. 

#### <a name="bot-to-connector-example-jwt-components"></a>Robot vers connecteur : exemple de composants JWT

```json
header:
{
  typ: "JWT",
  alg: "RS256",
  x5t: "<SIGNING KEY ID>",
  kid: "<SIGNING KEY ID>"
},
payload:
{
  aud: "https://api.botframework.com",
  iss: "https://sts.windows.net/d6d49420-f39b-4df7-a1dc-d59a935871db/",
  nbf: 1481049243,
  exp: 1481053143,
  appid: "<YOUR MICROSOFT APP ID>",
  ... other fields follow
}
```

> [!NOTE]
> En pratique, les champs réels peuvent être différents. Créez et validez tous les jetons JWT comme indiqué ci-dessus.

## <a id="connector-to-bot"></a> Authentifier les requêtes du service Bot Connector auprès de votre robot

Lorsque le service Bot Connector envoie une requête à votre robot, il indique un jeton JWT signé dans l’en-tête `Authorization` de la requête. Votre robot peut authentifier les appels du service Bot Connector en vérifiant l’authenticité du jeton JWT signé. 

Ce diagramme montre la procédure d’authentification du connecteur vers le robot :

![Authentifier les appels de Bot Connector à votre robot](../media/connector/auth_bot_connector_to_bot.png)

### <a id="openid-metadata-document"></a> Étape 2 : Récupérer le document de métadonnées OpenID

Le document de métadonnées OpenID indique l’emplacement d’un second document qui énumère les clés de signature valides du service Bot Connector. Pour récupérer le document de métadonnées OpenID, envoyez la requête suivante via HTTPS :

```http
GET https://login.botframework.com/v1/.well-known/openidconfiguration
```

> [!TIP]
> Il s’agit d’une URL statique que vous pouvez coder en dur dans votre application. 

L’exemple suivant montre un document de métadonnées OpenID qui est renvoyé en réponse à la requête `GET`. La propriété `jwks_uri` indique l’emplacement du document qui contient les clés de signature valides du service Bot Connector.

```json
{
    "issuer": "https://api.botframework.com",
    "authorization_endpoint": "https://invalid.botframework.com",
    "jwks_uri": "https://login.botframework.com/v1/.well-known/keys",
    "id_token_signing_alg_values_supported": [
      "RS256"
    ],
    "token_endpoint_auth_methods_supported": [
      "private_key_jwt"
    ]
}
```

### <a id="connector-to-bot-step-3"></a> Étape 3 : Récupérer la liste des clés de signature valides

Pour obtenir la liste des clés de signature valides, envoyez une requête `GET` via HTTPS à l’URL indiquée par la propriété `jwks_uri` dans le document de métadonnées OpenID. Par exemple : 

```http
GET https://login.botframework.com/v1/.well-known/keys
```

Le corps de la réponse indique le document au format [JWK](https://tools.ietf.org/html/rfc7517). Il inclut également une propriété supplémentaire pour chaque clé : `endorsements`. La liste des clés est relativement stable. Il est possible de la mettre en cache pendant de longues périodes de temps (par défaut, 5 jours dans le kit de développement logiciel Bot Builder).

La propriété `endorsements` de chaque clé contient une ou plusieurs chaînes d’approbation que vous pouvez utiliser pour vérifier l’authenticité de l’identifiant de canal indiqué dans la propriété `channelId` dans l’objet [Activité][Activity] de la requête entrante. La liste des identifiants de canal nécessitant une approbation est configurable dans chaque robot. Par défaut, la liste contient tous les identifiants des canaux publiés. Les développeurs de robots peuvent néanmoins remplacer les valeurs des identifiants des canaux sélectionnés. Si l’identifiant du canal nécessite une approbation :

- Vous devez exiger que tous les objets [Activité][Activity] envoyés à votre robot avec cet identifiant de canal soient accompagnés d’un jeton JWT signé avec une approbation du canal. 
- S’il manque l’approbation, votre robot doit rejeter la requête en renvoyant un code d’état **HTTP 403 (interdit)**.

### <a name="step-4-verify-the-jwt-token"></a>Étape 4 : Vérifier le jeton JWT

Pour vérifier l’authenticité du jeton envoyé par le service Bot Connector, vous devez extraire le jeton de l’en-tête `Authorization` de la requête, l’analyser, vérifier son contenu et vérifier sa signature. 

Les bibliothèques d’analyse JWT sont disponibles pour de nombreuses plates-formes et la plupart d’entre elles effectuent une analyse fiable et sécurisée des jetons JWT, bien que vous deviez généralement les configurer pour exiger que certains éléments du jeton (émetteur, public, etc.) contiennent des valeurs correctes. Lorsque vous analysez le jeton, vous devez configurer la bibliothèque d’analyse ou inscrire votre propre validation pour vous assurer qu’il respecte ces exigences :

1. Le jeton a été envoyé dans l’en-tête HTTP `Authorization` avec le schéma « porteur ».
2. Le jeton est un JSON valide conforme à la norme [JWT ](http://openid.net/specs/draft-jones-json-web-token-07.html).
3. Le jeton contient une revendication « émetteur » ayant une valeur de `https://api.botframework.com`.
4. Le jeton contient une revendication « public » ayant une valeur égale à l’identifiant de l’application Microsoft du robot.
5. Le jeton se trouve dans sa période de validité. La durée standard dans le secteur est de 5 minutes.
6. La signature chiffrée du jeton est valide et la clé est répertoriée dans le document des clés OpenID récupéré à l’[étape 3](#connector-to-bot-step-3) à l’aide de l’algorithme de signature indiqué dans la propriété `id_token_signing_alg_values_supported` du document Open ID Metadata récupéré à l’[étape 2](#openid-metadata-document).
7. Le jeton contient une revendication « serviceUrl » dont la valeur correspond à la propriété `servieUrl` à la racine de l’objet [Activité][Activity] de la requête entrante. 

Si le jeton ne respecte pas toutes ces exigences, votre robot doit rejeter la requête en renvoyant un code d’état **HTTP 403 (interdit)**.

> [!IMPORTANT]
> Toutes ces exigences sont importantes, notamment les exigences 4 et 6. Si vous n’appliquez pas TOUTES ces exigences de vérification, le robot sera exposé à des attaques susceptibles d’entraîner la divulgation de son jeton JWT.

Les implémenteurs doivent éviter que la validation du jeton JWT envoyé au robot puisse être désactivée.

#### <a name="connector-to-bot-example-jwt-components"></a>Connecteur vers Robot : exemple de composants JWT

```json
header:
{
  typ: "JWT",
  alg: "RS256",
  x5t: "<SIGNING KEY ID>",
  kid: "<SIGNING KEY ID>"
},
payload:
{
  aud: "<YOU MICROSOFT APP ID>",
  iss: "https://api.botframework.com",
  nbf: 1481049243,
  exp: 1481053143,
  ... other fields follow
}
```

> [!NOTE]
> En pratique, les champs réels peuvent être différents. Créez et validez tous les jetons JWT comme indiqué ci-dessus.

## <a id="emulator-to-bot"></a> Authentifier les requêtes de l’émulateur de Bot Framework auprès de votre robot

> [!WARNING]
> À la fin de l’automne 2017, la version 3.2 du protocole de sécurité Bot Framework a été mise en place. Cette nouvelle version inclut une nouvelle valeur « émetteur » dans les jetons échangés entre Bot Framework Eumaltor et votre robot. Pour préparer cette modification, les étapes ci-dessous indiquent comment vérifier les valeurs de l’émetteur dans les versions v3.1 et v3.2. 

L’[émulateur de Bot Framework](../bot-service-debug-emulator.md) est un outil de bureau dont vous pouvez vous servir pour tester les fonctionnalités de votre robot. Bien que l’émulateur de Bot Framework utilise les mêmes [ technologies d’authentification](#authentication-technologies) que celles décrites ci-dessus, il ne peut pas se faire passer pour le véritable service Bot Connector. À la place, il utilise l’identifiant et du mot de passe de l’application Microsoft que vous indiquez lorsque vous connectez l’émulateur à votre robot pour créer des jetons identiques à ceux créés par le robot. Lorsque l’émulateur envoie une requête à votre robot, il indique le jeton JWT dans l’en-tête `Authorization` de la requête. En fait, il utilise ses propres informations d’identification pour authentifier la requête. 

Pour accepter les requêtes de l’émulateur de Bot Framework en implémentant une bibliothèque d’authentification, vous devez ajouter ce chemin de vérification supplémentaire. La structure du chemin est similaire à celle du chemin de vérification [Connecteur -> Robot](#connector-to-bot). Par contre, il fait appel au document OpenID de MSA au lieu du document OpenID de Bot Connector.

Ce diagramme montre la procédure d’authentification de l’émulateur vers le robot :

![Authentifier les appels de l’émulateur de Bot Framework vers votre robot](../media/connector/auth_bot_framework_emulator_to_bot.png)

---
### <a name="step-2-get-the-msa-openid-metadata-document"></a>Étape 2 : Récupérer le document de métadonnées OpenID de MSA

Le document de métadonnées OpenID indique l’emplacement d’un second document qui énumère les clés de signature valides. Pour récupérer le document de métadonnées OpenID de MSA, envoyez la requête suivante via HTTPS :

```http
GET https://login.microsoftonline.com/botframework.com/v2.0/.well-known/openid-configuration
```

L’exemple suivant montre un document de métadonnées OpenID qui est renvoyé en réponse à la requête `GET`. La propriété `jwks_uri` indique l’emplacement du document qui contient les clés de signature valides.

```json
{
    "authorization_endpoint":"https://login.microsoftonline.com/common/oauth2/v2.0/authorize",
    "token_endpoint":"https://login.microsoftonline.com/common/oauth2/v2.0/token",
    "token_endpoint_auth_methods_supported":["client_secret_post","private_key_jwt"],
    "jwks_uri":"https://login.microsoftonline.com/common/discovery/v2.0/keys",
    ...
}
```

### <a id="emulator-to-bot-step-3"></a> Étape 3 : Récupérer la liste des clés de signature valides

Pour obtenir la liste des clés de signature valides, envoyez une requête `GET` via HTTPS à l’URL indiquée par la propriété `jwks_uri` dans le document de métadonnées OpenID. Par exemple : 

```http
GET https://login.microsoftonline.com/common/discovery/v2.0/keys 
Host: login.microsoftonline.com
```

Le corps de la réponse indique le document au format [JWK ](https://tools.ietf.org/html/rfc7517). 

### <a name="step-4-verify-the-jwt-token"></a>Étape 4 : Vérifier le jeton JWT

Pour vérifier l’authenticité du jeton envoyé par l’émulateur, vous devez extraire le jeton de l’en-tête `Authorization` de la requête, l’analyser, vérifier son contenu et vérifier sa signature. 

Les bibliothèques d’analyse JWT sont disponibles pour de nombreuses plates-formes et la plupart d’entre elles effectuent une analyse fiable et sécurisée des jetons JWT, bien que vous deviez généralement les configurer pour exiger que certains éléments du jeton (émetteur, public, etc.) contiennent des valeurs correctes. Lorsque vous analysez le jeton, vous devez configurer la bibliothèque d’analyse ou inscrire votre propre validation pour vous assurer qu’il respecte ces exigences :

1. Le jeton a été envoyé dans l’en-tête HTTP `Authorization` avec le schéma « porteur ».
2. Le jeton est un JSON valide conforme à la norme [JWT ](http://openid.net/specs/draft-jones-json-web-token-07.html).
3. Le jeton contient une revendication « émetteur » ayant une valeur de `https://sts.windows.net/d6d49420-f39b-4df7-a1dc-d59a935871db/` ou `https://sts.windows.net/f8cdef31-a31e-4b4a-93e4-5f571e91255a/`. (Le fait de vérifier les deux valeurs de l’émetteur vous permet de contrôler les valeurs d’émetteur des protocoles de sécurité des versions v3.1 et v3.2.)
4. Le jeton contient une revendication « public » ayant une valeur égale à l’identifiant de l’application Microsoft du robot.
5. Le jeton contient une revendication « appid » dont la valeur est égale à l’identifiant de l’application Microsoft du robot.
6. Le jeton se trouve dans sa période de validité. La durée standard dans le secteur est de 5 minutes.
7. Le jeton a une signature chiffrée valide avec une clé répertoriée dans le document des clés OpenID récupéré à l’[étape 3](#emulator-to-bot-step-3).

> [!NOTE]
> L’exigence 5 est spécifique au chemin de vérification de l’émulateur. 

Si le jeton ne respecte pas toutes ces exigences, votre robot doit mettre fin à la requête en renvoyant un code d’état **HTTP 403 (interdit)**.

> [!IMPORTANT]
> Toutes ces exigences sont importantes, notamment les exigences 4 et 7. Si vous n’appliquez pas TOUTES ces exigences de vérification, le robot sera exposé à des attaques susceptibles d’entraîner la divulgation de son jeton JWT.

#### <a name="emulator-to-bot-example-jwt-components"></a>Émulateur vers Robot : exemple de composants JWT

```json
header:
{
  typ: "JWT",
  alg: "RS256",
  x5t: "<SIGNING KEY ID>",
  kid: "<SIGNING KEY ID>"
},
payload:
{
  aud: "<YOUR MICROSOFT APP ID>",
  iss: "https://sts.windows.net/d6d49420-f39b-4df7-a1dc-d59a935871db/",
  nbf: 1481049243,
  exp: 1481053143,
  ... other fields follow
}
```

> [!NOTE]
> En pratique, les champs réels peuvent être différents. Créez et validez tous les jetons JWT comme indiqué ci-dessus.

## <a name="security-protocol-changes"></a>Modifications du protocole de sécurité

> [!WARNING]
> La version 3.0 du protocole de sécurité n’est plus prise en charge depuis le **31 juillet 2017**. Si vous avez créé votre propre code d’authentification (c’est-à-dire que vous n’avez pas utilisé le kit de développement logiciel Bot Builder pour créer votre robot), vous devez effectuer une mise à niveau vers la version 3.1 du protocole de sécurité en mettant à jour votre application afin d’utiliser les valeurs de la version v3.1 énumérées ci-dessous. 

### <a name="bot-to-connector-authenticationbot-to-connector"></a>[Authentification du robot vers le connecteur](#bot-to-connector)

#### <a name="oauth-login-url"></a>URL de connexion OAuth

| Version du protocole | Valeur valide |
|----|----|
| v3.1 et v3.2 | `https://login.microsoftonline.com/botframework.com/oauth2/v2.0/token` |

#### <a name="oauth-scope"></a>Étendue OAuth

| Version du protocole | Valeur valide |
|----|----|
| v3.1 et v3.2 |  `https://api.botframework.com/.default` |

### <a name="connector-to-bot-authenticationconnector-to-bot"></a>[Authentification du connecteur vers le robot](#connector-to-bot)

#### <a name="openid-metadata-document"></a>Document de métadonnées OpenID

| Version du protocole | Valeur valide |
|----|----|
| v3.1 et v3.2 | `https://login.botframework.com/v1/.well-known/openidconfiguration` |

#### <a name="jwt-issuer"></a>Émetteur JWT

| Version du protocole | Valeur valide |
|----|----|
| v3.1 et v3.2 | `https://api.botframework.com` |

### <a name="emulator-to-bot-authenticationemulator-to-bot"></a>[Authentification de l’émulateur vers le robot](#emulator-to-bot)

#### <a name="oauth-login-url"></a>URL de connexion OAuth

| Version du protocole | Valeur valide |
|----|----|
| v3.1 et v3.2 | `https://login.microsoftonline.com/botframework.com/oauth2/v2.0/token` |

#### <a name="oauth-scope"></a>Étendue OAuth

| Version du protocole | Valeur valide |
|----|----|
| v3.1 et v3.2 |  Identificateur de l’application Microsoft de votre robot + `/.default` |

#### <a name="jwt-audience"></a>Public JWT

| Version du protocole | Valeur valide |
|----|----|
| v3.1 et v3.2 | Identificateur de l’application Microsoft de votre robot |

#### <a name="jwt-issuer"></a>Émetteur JWT

| Version du protocole | Valeur valide |
|----|----|
| v3.1 | `https://sts.windows.net/d6d49420-f39b-4df7-a1dc-d59a935871db/` |
| v3.2 | `https://sts.windows.net/f8cdef31-a31e-4b4a-93e4-5f571e91255a/` |

#### <a name="openid-metadata-document"></a>Document de métadonnées OpenID

| Version du protocole | Valeur valide |
|----|----|
| v3.1 et v3.2 | `https://login.microsoftonline.com/botframework.com/v2.0/.well-known/openid-configuration` |

## <a name="additional-resources"></a>Ressources supplémentaires

- [Résoudre les problèmes d’authentification de Bot Framework](../bot-service-troubleshoot-authentication-problems.md)
- [JSON Web Token (JWT) draft-jones-json-web-token-07](http://openid.net/specs/draft-jones-json-web-token-07.html)
- [JSON Web Signature (JWS) draft-jones-json-web-signature-04](https://tools.ietf.org/html/draft-jones-json-web-signature-04)
- [JSON Web Key (JWK) RFC 7517](https://tools.ietf.org/html/rfc7517)

[Activity]: bot-framework-rest-connector-api-reference.md#activity-object
