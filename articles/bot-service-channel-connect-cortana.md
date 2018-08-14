---
title: Connecter un bot à Cortana | Microsoft Docs
description: Découvrez comment configurer un bot pour y accéder par le biais de l’interface de Cortana.
keywords: cortana, canal de bot, configurer cortana
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 04/30/2018
ms.openlocfilehash: 6e694ce8b54ebd2405d7496d333c2bb27eb344f1
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39299173"
---
# <a name="connect-a-bot-to-cortana"></a>Connecter un bot à Cortana

Cortana est un canal de reconnaissance vocale qui peut gérer une conversation textuelle, mais également envoyer et recevoir des messages vocaux. Un bot destiné à se connecter à Cortana doit être conçu pour la reconnaissance vocale et pour le texte. Une *compétence* Cortana est un bot qui peut être appelé à l’aide d’un client Cortana. La publication d’un bot entraîne l’ajout du bot à la liste des compétences disponibles.

Pour ajouter le canal Cortana, ouvrez le bot dans le [Portail Azure](https://portal.azure.com/), cliquez sur le panneau **Canaux**, puis cliquez sur **Cortana**.

![Ajouter un canal Cortana](~/media/channels/cortana-addchannel.png)

## <a name="configure-cortana"></a>Configurer Cortana

Lorsque vous connectez votre bot avec le canal Cortana, certaines informations de base sur votre bot sont préremplies dans le formulaire d’inscription. Vérifiez soigneusement ces informations. Le formulaire comprend les champs suivants.

| Champ | Description |
|------|------|
| **Icône de compétence** | Une icône qui s’affiche dans le canevas Cortana lorsque votre compétence est appelée. Elle est également utilisée lorsque les compétences sont détectables (Microsoft Store, par exemple). (32 Ko maximum, PNG uniquement)|
| **Nom complet** | Le nom de votre compétence Cortana est affiché pour l’utilisateur en haut de l’interface utilisateur visuelle. (Limite de 30 caractères) |
| **Nom d’appel** | Il s’agit du nom prononcé par les utilisateurs pour appeler une compétence. Il ne doit pas comporter plus de trois mots et doit être facile à prononcer. Consultez la rubrique [Invocation Name Guidelines][invocation] (Directives relatives aux noms d’appel) pour plus d’informations sur le choix de ce nom.|
| **Description** | Une description de votre compétence Cortana. Elle est utilisée lorsque les compétences sont détectables (Microsoft Store, par exemple). |
| **Brève description** | Une brève description des fonctionnalités de votre compétence, utilisée pour décrire la compétence dans le bloc-notes de Cortana. |

## <a name="general-bot-information"></a>Informations générales sur le bot

Dans la section **Manage user identity through connected services** (Gérer l’identité de l’utilisateur à l’aide des services connectés), cliquez sur l’option d’activation. Remplissez le formulaire.

Tous les champs accompagnés d’un astérisque (*) sont obligatoires. Vous devez publier les bots sur le Bot Framework pour pouvoir les connecter à Cortana.

![Fournir des informations générales](~/media/channels/cortana-details.png)

### <a name="sign-in-at-invocation"></a>Sign in at invocation (Connexion lors de l’appel)

Sélectionnez cette option si vous voulez que Cortana connecte l’utilisateur lorsqu’il appelle votre compétence.

### <a name="sign-in-when-required"></a>Sign in when required (Connexion si nécessaire)

Sélectionnez cette option si vous utilisez une carte de connexion de Bot Framework pour connecter l’utilisateur. En règle générale, vous utilisez cette option pour connecter l’utilisateur uniquement s’il utilise une fonctionnalité qui requiert une authentification. Lorsque votre compétence envoie un message contenant la carte de connexion sous forme de pièce jointe, Cortana ignore la carte de connexion et applique le flux d’autorisation en utilisant les paramètres du compte de connexion.

### <a name="connected-service-icon"></a>Icône de service connecté

L’icône que vous souhaitez afficher lorsque l’utilisateur se connecte à votre compétence.

### <a name="account-name"></a>Nom du compte

Le nom de votre compétence que vous souhaitez afficher lorsque l’utilisateur se connecte à votre compétence.

### <a name="client-id-for-third-party-services"></a>Client ID for third-party services (ID de client pour les services tiers)

L’ID d’application de votre bot. Vous recevez cet ID lors de l’inscription de votre bot.

### <a name="space-separated-list-of-scopes"></a>Space-separated list of scopes (Liste d’étendues séparées par des espaces)

Spécifiez les étendues nécessaires au service (voir la documentation du service).

### <a name="authorization-url"></a>URL d’autorisation

Défini sur `https://login.microsoftonline.com/common/oauth2/v2.0/authorize`.

### <a name="grant-type"></a>Type d’autorisation

Sélectionnez le code d’autorisation pour utiliser le flux d’octroi de code. Sélectionnez Implicite pour utiliser le flux implicite.

### <a name="token-url"></a>URL du jeton

Si vous sélectionnez le code d’autorisation, définissez l’URL `https://login.microsoftonline.com/common/oauth2/v2.0/token`.

### <a name="client-secretpassword-for-third-party-services"></a>Client secret/password for third party services (Clé secrète client/mot de passe pour les services tiers)

Le mot de passe du bot. Vous recevez ce mot de passe lors de l’inscription de votre bot.

### <a name="client-authentication-scheme"></a>Client authentication scheme (Schéma d’authentification client)

Sélectionnez l’option HTTP de base.

### <a name="token-options"></a>Token options (Options de jeton)

Sélectionnez POST.

### <a name="request-user-profile-data-optional"></a>Request user profile data (Demander des données de profil utilisateur) (facultatif)

Cortana permet d’accéder à différents types d’informations de profil utilisateur. Vous pouvez les utiliser pour personnaliser le bot pour l’utilisateur. Par exemple, si une compétence a accès au nom et à l’emplacement de l’utilisateur, elle peut offrir une réponse personnalisée comme « Bonjour Kamran, j’espère que vous passez un agréable séjour à Bellevue, à Washington ».

Cliquez sur le lien **Add a user profile request** (Ajouter une demande de profil utilisateur), puis sélectionnez les informations de profil utilisateur souhaitées dans la liste déroulante. Ajoutez le nom convivial à utiliser pour accéder à ces informations à partir du code de votre bot.

### <a name="save-skill"></a>Enregistrer la compétence

Lorsque vous avez rempli le formulaire d’inscription pour votre compétence Cortana, cliquez sur Enregistrer pour établir la connexion. Le système vous ramène au panneau Canaux de votre bot. Vous voyez que celui-ci est à présent connecté à Cortana.

À ce stade, votre bot a déjà été automatiquement déployé en tant que compétence Cortana dans votre compte.

## <a name="next-steps"></a>Étapes suivantes

* [Cortana Skills Kit](https://aka.ms/CortanaSkillsDocs) (Kit de compétences Cortana)
* [Activer le débogage](bot-service-debug-cortana-skill.md)
* [Publishing Cortana Skills][publish] (Publier des compétences Cortana)

[invocation]: https://docs.microsoft.com/en-us/cortana/skills/cortana-invocation-guidelines
[publish]: https://docs.microsoft.com/en-us/cortana/skills/publish-skill
[connected]: https://aka.ms/CortanaSkillsBotConnectedAccount
[CortanaEntity]: https://aka.ms/lgvcto
