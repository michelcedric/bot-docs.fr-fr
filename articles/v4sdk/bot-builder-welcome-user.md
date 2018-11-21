---
redirect_url: /bot-framework/bot-builder-send-welcome-message
ms.openlocfilehash: ef281fd1b6539484c06f68caffbd4e87ec8acc2b
ms.sourcegitcommit: cb0b70d7cf1081b08eaf1fddb69f7db3b95b1b09
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/09/2018
ms.locfileid: "51332903"
---
<a name="--"></a><!--
---
titre : Accueillir l’utilisateur | Microsoft Docs description : Découvrir comment concevoir votre bot pour qu’il propose une expérience utilisateur accueillante.
mots clés : vue d’ensemble, expérience utilisateur, Accueil, expérience personnalisée author: dashel ms.author: dashel manager: kamrani ms.topic: article ms.service: bot-service ms.subservice: sdk ms.date: 8/30/2018 monikerRange: « azure-bot-service-4.0 »
---

# <a name="welcoming-the-user"></a>Accueillir l’utilisateur

L’objectif principal lors de la création d’un bot est d’impliquer votre utilisateur dans une conversation utile. Pour atteindre cet objectif, l’une des meilleures méthodes consiste à s’assurer qu’à partir du moment où un utilisateur se connecte pour la première fois, il comprenne les fonctionnalités et l’utilité principales de votre bot, c’est-à-dire la raison pour laquelle il a été créé.

## <a name="show-your-purpose"></a>Montrer votre objectif

Imaginez pendant un instant l’expérience d’un utilisateur qui se connecte à votre bot nommé « Informations touristiques sur Chicago », dans l’espoir d’obtenir des renseignements sur des hôtels à Chicago. Ce qu’il ne sait pas, c’est que votre bot est en fait destiné aux passionnés de pizzas de Chicago et propose uniquement des informations sur des restaurants. Quand il finit par comprendre que les réponses ne correspondent pas à ses questions, cet utilisateur quitte le site en gardant le souvenir d’une mauvaise expérience. Ce genre de mauvaise expérience utilisateur peut être évité en accueillant vos utilisateurs avec suffisamment d’informations leur permettant de connaître les fonctionnalités et l’utilité principales de votre bot. 

![Message de bienvenue](./media/welcome_message.png)

Après avoir lu votre message de bienvenue, si votre bot ne fournit pas le type d’informations souhaité par l’utilisateur, il peut rapidement quitter le site et ainsi éviter toute frustration.
Un message de bienvenue doit être généré lors de la première interaction de vos utilisateurs avec votre bot. Pour ce faire, vous pouvez surveiller les types **Activité** de votre bot et observer les nouvelles connexions. Chaque nouvelle connexion peut générer jusqu’à deux activités de mise à jour de conversation selon le canal.

- Une lorsque le bot de l’utilisateur est connecté à la conversation.
- Une lorsque l’utilisateur rejoint la conversation.

Il est tentant de simplement générer un message de bienvenue chaque fois qu’une nouvelle mise à jour de conversation est détectée, mais cela peut produire des résultats inattendus lorsque l’accès à votre bot se fait par divers canaux.

## <a name="design-for-different-channels"></a>Conception pour différents canaux

Bien que vous ayez un contrôle total sur les informations présentées par votre bot, vous pouvez ne pas contrôler la façon dont les différents canaux implémentent la présentation de ces informations. Certains canaux créent une mise à jour de conversation lorsqu’un utilisateur se connecte initialement à ce canal et une mise à jour de conversation distincte uniquement après réception d’un message d’entrée initial de l’utilisateur. Les autres canaux génèrent ces deux activités lorsque l’utilisateur se connecte initialement au canal. Si vous surveillez simplement un événement de mise à jour de conversation et l’affichage d’un message de bienvenue sur un canal avec deux activités de mise à jour de conversation, votre utilisateur peut recevoir les éléments suivants :

![Message de bienvenue en double](./media/double_welcome_message.png)

Ce message en double peut être évité en générant un message de bienvenue initial pour le deuxième événement de mise à jour de conversation uniquement. Le deuxième événement peut être détecté quand :
- un événement de mise à jour de conversation s’est produit
- et quand un nouveau membre (utilisateur) a été ajouté à la conversation.

Il est également important de déterminer si l’entrée utilisateur contient des informations utiles, ce qui peut également varier en fonction des canaux. Si un canal génère deux activités de mise à jour de conversation après la connexion initiale à un bot, la première entrée utilisateur peut être correctement évaluée comme une réponse à l’invite de votre message de bienvenue comme la conversation indiquée ci-dessous.

![Réponse entrée unique](./media/single_input_response.png)

Toutefois, si un canal attend une entrée utilisateur initiale avant de générer un deuxième événement de mise à jour de conversation, alors le même code que celui utilisé ci-dessus crée l’expérience utilisateur suivante.

![Mauvaise réponse entrée unique](./media/single_input_wrong_response.png)

Pour vous assurer que vos utilisateurs ont une bonne expérience sur tous les canaux, une meilleure pratique consiste à ne pas attendre d’informations utiles dans l’entrée de conversation initiale d’un utilisateur. Imaginez plutôt que l’entrée initiale représente des données futiles, puis à leur réception, invitez l’utilisateur à vous donner les informations nécessaires pour poursuivre la conversation. Cette technique génère une expérience utilisateur cohérente, quel que soit le canal utilisé pour accéder initialement à votre bot.

![Première entrée futile](./media/no_first_input_response.png)

## <a name="personalize-the-user-experience"></a>Personnaliser l’expérience utilisateur

Être sans cesse obligé de fournir des informations déjà données auparavant peut sembler impersonnel et non convivial pour un utilisateur. Si votre bot conserve les informations des utilisateurs qui ont effectué une visite auparavant, il est conseillé de vérifier ces informations en premier lieu et d’accueillir votre utilisateur avec le nom que vous avez stocké lors de sa précédente visite, si disponible. 

![Nouveau message de bienvenue](./media/welcome_back.png)

Dans ce cas, votre logique de conversation ignore la demande de nom et passe à l’activité de conversation suivante.

Votre bot peut également personnaliser une expérience utilisateur avec des réponses rapides des entrées utilisateur inattendues, telles que des demandes pour mettre fin à une conversation.

![Répondre à une demande pour mettre fin à une conversation](./media/respond_to_exit.png)

Proposer des interactions rapides et en adéquation avec la conversation permet d’offrir aux utilisateurs une expérience plus conviviale et des interactions plus agréables avec votre bot.

## <a name="next-steps"></a>Étapes suivantes
> [!div class="nextstepaction"]
> [Envoyer un message de bienvenue aux utilisateurs](bot-builder-send-welcome-message.md)

-->
