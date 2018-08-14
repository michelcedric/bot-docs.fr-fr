---
title: Mettre à niveau votre robot vers l’API Bot Framework v3 | Microsoft Docs
description: Découvrez comment mettre à niveau votre robot vers l’API Bot Framework v3.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: ca54abf89f8967b2b109895326d13a601030fdec
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39299401"
---
# <a name="upgrade-your-bot-to-bot-framework-api-v3"></a>Mettre à niveau votre robot vers l’API Bot Framework v3

Dans le cadre de la Conférence Build 2016, Microsoft a annoncé Microsoft Bot Framework et son itération initiale de l’API robot Connector, de même que les kits de développement logiciel (SDK) Bot Builder et Bot Connector. Depuis lors, nous avons recueilli vos commentaires et travaillé activement à l’amélioration de l’API REST et des kits de développement logiciel (SDK).

En juillet 2016, l’API Bot Framework v3 a été publiée et l’API Bot Framework v1 déconseillée. Les robots qui utilisent l’API v1 ont cessé de fonctionner sur Skype en décembre 2016, et sur tous les autres canaux le 23 février 2017. Si vous avez créé un robot à l’aide de l’API v1 et souhaitez le rendre à nouveau fonctionnel, vous devez le mettre à niveau vers l’API v3 en suivant les instructions de cet article. Pour être certain de comprendre le processus de mise à niveau de bout en bout, lisez attentivement cet article avant de commencer. 

## <a name="step-1-get-your-app-id-and-password-from-the-bot-framework-portal"></a>Étape 1 : obtenir les identifiant et mot de passe de votre application sur le portail Bot Framework

Connectez-vous au [portail Bot Framework](https://dev.botframework.com/), cliquez sur **My bots** (Mes robots), puis sélectionnez votre robot pour ouvrir son tableau de bord. Cliquez ensuite sur le lien **SETTINGS** (PARAMÈTRES) dans l’angle supérieur droit de la page. 

Dans la section **Configuration** de la page des paramètres, examinez le contenu du champ **App ID** (Identifiant de l’application), puis passez aux étapes suivantes selon que le champ **App ID** est ou non déjà renseigné.

### <a name="case-1-app-id-field-is-already-populated"></a>Cas 1 : champ App ID (ID de l’application) déjà renseigné

Si le champ **App ID** (ID de l’application) est déjà renseigné, procédez comme suit :

1. Cliquez sur **Manage Microsoft App ID and password** (gérer l’ID et le mot de passe de de l’application Microsoft).  
![Configuration](~/media/upgrade/manage-app-id.png)

2. Cliquez sur **Generate New Password** (Générer un nouveau mot de passe).  
![Générer un nouveau mot de passe](~/media/upgrade/generate-new-password.png)

3. Copiez et enregistrez le nouveau mot de passe, ainsi que l’ID d’application MSA. Vous en aurez besoin par la suite.  
![Nouveau mot de passe](~/media/upgrade/new-password-generated.png)

### <a name="case-2-app-id-field-is-empty"></a>Cas 2 : le champ App ID (ID de l’application) est vide

Si le champ **App ID** (ID de l’application) est vide, procédez comme suit :

1. Cliquez sur **Create Microsoft App ID and password** (Créer l’ID et le mot de passe de l’application Microsoft).  
   ![Create Microsoft App ID and password](~/media/upgrade/generate-appid-and-password.png) (Créer l’ID et le mot de passe de l’application)
   > [!IMPORTANT]
   > N’activez pas encore la case d’option **Version 3.0**. Vous le ferez plus tard, après avoir [mis à jour le code de votre robot](#update-code).</div>

2. Cliquez sur **Generate a password to continue** (Générer un mot de passe pour continuer).  
   ![Générer le mot de passe de l’application](~/media/upgrade/generate-a-password-to-continue.png)

3. Copiez et enregistrez le nouveau mot de passe, ainsi que l’ID d’application MSA. Vous en aurez besoin par la suite.  
   ![Nouveau mot de passe](~/media/upgrade/new-password-generated.png)

4. Cliquez sur **Finish and go back to Bot Framework** (Terminer et revenir à Bot Framework).  
   ![Terminer et revenir au portail](~/media/upgrade/finish-and-go-back-to-bot-framework.png)

5. De retour à la page des paramètres du robot dans le portail Bot Framework, faites défiler vers le bas de la page, puis cliquez sur **Save changes** (Enregistrer les modifications).  
   ![Enregistrer les modifications](~/media/upgrade/save-changes.png)

## <a id="update-code"></a> Étape 2 : mettre à jour le code de votre robot vers la version 3.0

Pour mettre à jour le code de votre robot vers la version 3.0, procédez comme suit :

1. Mettez à jour vers la dernière version du [Kit de développement logiciel (SDK) Bot Builder](https://github.com/Microsoft/BotBuilder) le langage de votre robot.
2. Mettez à jour votre code pour appliquer les modifications nécessaires, en suivant les instructions ci-dessous.
3. Utilisez l’[émulateur Bot Framework](~/bot-service-debug-emulator.md) pour tester votre robot localement, puis dans le cloud.

Les sections suivantes décrivent les principales différences entre les API v1 et v3. Une fois votre code mis à jour vers l’API v3, vous pouvez terminer le processus de mise à niveau en [mettant à jour les paramètres de votre robot](#step-3) sur le portail Bot Framework.

### <a name="botbuilder-and-connector-are-now-one-sdk"></a>BotBuilder et le connecteur font désormais partie d’un même Kit de développement logiciel (SDK)

Au lieu de devoir installer des kits de développement logiciel (SDK) distincts pour Builder et le connecteur à l’aide de plusieurs packages NuGet (ou modules NPM), vous pouvez désormais obtenir les deux bibliothèques dans un Kit de développement logiciel (SDK) Bot Builder unique :

- Kit de développement logiciel (SDK) Bot Builder pour .NET : `Microsoft.Bot.Builder` package NuGet
- Kit de développement logiciel (SDK) Bot Builder pour Node.js : `botbuilder` module NPM

Le Kit de développement logiciel (SDK) `Microsoft.Bot.Connector` autonome est désormais déconseillé et n’est plus conservé.

### <a name="message-is-now-activity"></a>Le message est désormais Activité

L’objet `Message` a été remplacé par l’objet `Activity` dans l’API v3. Le type d’activité le plus courant est **message**, mais il existe d’autres types d’activité utilisables pour communiquer différents types d’informations à un robot ou à un canal. Pour plus d’informations sur les messages, voir [Créer des messages](~/dotnet/bot-builder-dotnet-create-messages.md) et [Envoyer et recevoir des activités](~/dotnet/bot-builder-dotnet-connector.md).

### <a name="activity-types--events"></a>Types d’activités et événements

Certains événements ont été renommés et/ou refactorisés dans l’API v3. En outre, une nouvelle énumération `ActivityTypes` a été ajoutée au connecteur pour éliminer la nécessité de mémoriser des types d’activités spécifiques.

- Le type d’activité `conversationUpdate` remplace la conversation Robot/Utilisateur Ajouté/Supprimé À/De par une méthode unique.
- Le nouveau type d’activité `typing` permet à votre robot d’indiquer qu’il compile une réponse et de savoir quand l’utilisateur tape une réponse.
- Le nouveau type d’activité `contactRelationUpdate` permet à votre robot de savoir s’il a été ajouté ou supprimé dans la liste de contacts de l’utilisateur.

Lorsque votre robot reçoit une activité `conversationUpdate`, les propriétés `MembersRemoved` et `MembersAdded` indiquent qui a été ajouté ou supprimé dans la conversation. Lorsque votre robot reçoit une activité `contactRelationUpdate`, la propriété `Action` indique si l’utilisateur a ajouté ou supprimé le robot dans sa liste de contacts. Pour plus d’informations sur les types d’activités, voir [Vue d’ensemble des activités](~/dotnet/bot-builder-dotnet-activities.md).

### <a name="addressing-messages"></a>Adressage de messages

L’emplacement où les informations sur l’expéditeur, le destinataire et le canal sont spécifiées dans un message a légèrement changé dans l’API v3 :

|Champ de l’API v1 | Champ de l’API v3|
|--------|--------|
| Object `From` | Object `From` |
| Object `To` | Object `Recipient` |
| Propriété `ChannelConversationID` | Object `Conversation`|
| Propriété `ChannelId` | Propriété `ChannelId` |

Pour plus d’informations sur l’adressage des messages, voir [Envoyer et recevoir des activités](~/dotnet/bot-builder-dotnet-connector.md).

### <a name="sending-replies"></a>Envoi de réponses

Dans l’API Bot Framework v3, toutes les réponses adressées à l’utilisateur sont envoyées de manière asynchrone sur une requête HTTP initiée séparément, plutôt qu’en ligne avec le HTTP POST pour le message entrant dans le robot. Comme aucun message n’est renvoyé en ligne à l’utilisateur via le connecteur, le type de retour de la méthode post de votre robot est `HttpResponseMessage`. Cela signifie que votre robot ne « retourne » pas de manière synchrone la chaîne que vous souhaitez envoyer à l’utilisateur, mais envoie un message de réponse à un moment quelconque dans votre code au lieu de devoir répondre sous la forme d’une réponse au POST entrant. Ces deux méthodes envoient un message à une conversation :

- `SendToConversation`
- `ReplyToConversation`

La méthode `SendToConversation` ajoute le message spécifié à la fin de la conversation, tandis que la méthode `ReplyToConversation` (pour les conversations qui la prennent en charge) ajoute le message spécifié en tant que réponse directe à un message précédent dans la conversation. Pour plus d’informations sur ces méthodes, voir [Envoyer et recevoir des activités](~/dotnet/bot-builder-dotnet-connector.md).

### <a name="starting-conversations"></a>Démarrage des conversations

Dans l’API Bot Framework v3, vous pouvez démarrer une conversation en utilisant la nouvelle méthode `CreateDirectConversation` pour démarrer une conversation privée avec un seul utilisateur, ou en utilisant la nouvelle méthode `CreateConversation` pour démarrer une conversation de groupe avec plusieurs utilisateurs. Pour plus d’informations sur le démarrage des conversations, voir [Envoyer et recevoir des activités](~/dotnet/bot-builder-dotnet-connector.md#start-a-conversation).

### <a name="attachments-and-options"></a>Pièces jointes et options

L’API Bot Framework v3 introduit une implémentation plus robuste des cartes et pièces jointes. Le type `Options` n’est plus pris en charge dans l’API v3. Il a été remplacé par des cartes. Pour plus d’informations sur l’ajout de pièces jointes à des messages à l’aide de .NET, voir [Ajouter des pièces jointes multimédia aux messages](~/dotnet/bot-builder-dotnet-add-media-attachments.md) et [Ajouter des pièces jointes de cartes enrichies aux messages](~/dotnet/bot-builder-dotnet-add-rich-card-attachments.md).

### <a name="bot-data-storage-bot-state"></a>Stockage de données de robot (Bot State)

Dans l’API Bot Framework v1, l’API de gestion des données d’état du robot a été intégrée dans l’API de messagerie. Dans l’API Bot Framework v3, ces API sont distinctes. Désormais, vous devez utiliser le service Bot State pour obtenir les données d’état (au lieu de supposer qu’elles seront incluses dans l’objet `Message`) et les stocker (au lieu de le transmettre dans l’objet `Message`). Pour plus d’informations sur la gestion des données d’état du robot à l’aide du service Bot State, voir [Gérer les données d’état](~/dotnet/bot-builder-dotnet-state.md).

> [!IMPORTANT]
> L’API du service Bot Framework State n’est pas recommandée pour les environnements de production et pourra être déconseillée dans une version ultérieure. Nous vous recommandons de mettre à jour votre code de robot pour qu’il utilise le stockage en mémoire à des fins de test ou pour qu’il utilise l’une des **extensions Azure** pour les robots de production. Pour plus d’informations, voir l’article **Gérer les données d’état** pour l’implémentation [.NET](~/dotnet/bot-builder-dotnet-state.md) ou [Node](~/nodejs/bot-builder-nodejs-state.md).

### <a name="webconfig-changes"></a>Modifications de Web.config

L’API Bot Framework v1 stockait les propriétés d’authentification avec ces clés dans **Web.Config** :

- `AppID`
- `AppSecret`

L’API Bot Framework v3 stocke les propriétés d’authentification avec ces clés dans **Web.Config** :

- `MicrosoftAppID`
- `MicrosoftAppPassword`

## <a id="step-3"></a> Étape 3 : mettre à jour les paramètres de votre robot dans le portail Bot Framework

Après avoir mis à niveau le code de votre robot vers l’API v3 et l’avoir déployé dans le cloud, achevez le processus de mise à niveau en procédant comme suit : 

1. Connectez-vous au [portail Bot Framework](https://dev.botframework.com/).

2. Cliquez sur **My bots** (Mes robots), puis sélectionnez votre robot pour ouvrir son tableau de bord. 

3. Cliquez sur le lien **SETTINGS** (PARAMÈTRES) à proximité de l’angle supérieur droit de la page. 

4. Dans la section **Configuration**, sous **Version 3.0**, collez le point de terminaison de votre robot dans le champ **Messaging endpoint** (Point de terminaison de messagerie).  
![Configuration de la version 3](~/media/upgrade/paste-new-v3-enpoint-url.png)

5. Activez la case d’option **Version 3.0**.  
![Sélectionner la version 3.0](~/media/upgrade/switch-to-v3-endpoint.png)

6. Faites défiler vers le bas de la page, puis cliquez sur **Enregistrer les modifications**.  
![Enregistrer les modifications](~/media/upgrade/save-changes.png)