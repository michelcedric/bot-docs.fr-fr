---
title: 'Service de bot : Forum Aux Questions | Microsoft Docs'
description: Une liste de questions fréquemment posées sur les éléments du Bot Framework et la disponibilité de nouvelles fonctionnalités.
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 05/03/2018
ms.openlocfilehash: e59f9b10686b10ae821b8c4bf259a1fc301ac702
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39298908"
---
# <a name="bot-framework-frequently-asked-questions"></a>Bot Framework : Forum Aux Questions

Cet article répond à certaines questions fréquemment posées sur le Bot Framework.

## <a name="background-and-availability"></a>Contexte et disponibilité
### <a name="why-did-microsoft-develop-the-bot-framework"></a>Pourquoi Microsoft a-t-il développé le Bot Framework ?

Alors que l’interface utilisateur conversationnelle est déjà à notre portée, peu de développeurs disposent actuellement de l’expertise et des outils nécessaires pour créer de nouvelles expériences conversationnelles ou doter les applications et services existants d’une interface conversationnelle pour les utilisateurs. Nous avons créé le Bot Framework pour aider les développeurs à générer des bots exceptionnels et à les connecter avec les utilisateurs, et ce, quel que soit le lieu de la conversation, y compris sur les canaux de haute qualité Microsoft.

### <a name="what-is-the-v4-sdk"></a>Qu’est-ce que le Kit de développement logiciel (SDK) v4 ?
Le Kit de développement logiciel (SDK) Bot Builder v4 s’appuie sur les commentaires et les enseignements tirés des SDK Bot Builder précédents. Il offre des niveaux d’abstraction appropriés et assure une grande modularité des blocs de construction de bot. Vous pouvez commencer avec un bot simple et le sophistiquer progressivement à l’aide d’une infrastructure modulaire et extensible. Vous trouverez un [FAQ](https://github.com/Microsoft/botbuilder-dotnet/wiki/FAQ) sur le Kit de développement logiciel (SDK) sur GitHub.


## <a name="channels"></a>Canaux
### <a name="when-will-you-add-more-conversation-experiences-to-the-bot-framework"></a>Quand allez-vous ajouter des expériences de conversation au Bot Framework ?

Nous prévoyons d’apporter des améliorations continues au Bot Framework et notamment d’ajouter des canaux. Cependant, nous ne pouvons pas fournir de calendrier pour l’instant.  
Si vous souhaitez que nous ajoutions un canal spécifique à l’infrastructure, [indiquez-le nous][Support].

### <a name="i-have-a-communication-channel-id-like-to-be-configurable-with-bot-framework-can-i-work-with-microsoft-to-do-that"></a>J’aimerais pouvoir configurer un canal de communication spécifique avec le Bot Framework. Puis-je travailler avec Microsoft à cette fin ?

Nous n’avons pas fourni aux développeurs de mécanisme général pour ajouter de nouveaux canaux au Bot Framework. Cependant, vous pouvez connecter votre bot à votre application par le biais de [l’API Direct Line][DirectLineAPI]. Si vous développez un canal de communication et que vous souhaitez travailler avec nous pour activer votre canal dans le Bot Framework, [n’hésitez pas à nous contacter][Support].

### <a name="if-i-want-to-create-a-bot-for-skype-what-tools-and-services-should-i-use"></a>Quels outils et services dois-je utiliser pour créer un bot pour Skype ?

Le Bot Framework est conçu pour générer, connecter et déployer des bots réactifs, performants et évolutifs de haute qualité pour Skype et de nombreux autres canaux. Vous pouvez utiliser le Kit de développement logiciel (SDK) pour créer des bots prenant en charge le texte, les sms, les images, les boutons et les cartes (utilisés aujourd’hui pour la majorité des interactions de bot dans le cadre d’expériences conversationnelles), ainsi que des interactions de bot spécifiques à Skype, comme des expériences audio et vidéo enrichies.

Si vous disposez déjà d’un excellent bot et que vous souhaitez atteindre les utilisateurs de Skype, vous pouvez le connecter facilement à Skype (ou à tout autre canal pris en charge) par le biais du Bot Builder pour l’API REST (dans la mesure où il dispose d’un point de terminaison REST accessible par Internet).

## <a name="security-and-privacy"></a>Sécurité et confidentialité
### <a name="do-the-bots-registered-with-the-bot-framework-collect-personal-information-if-yes-how-can-i-be-sure-the-data-is-safe-and-secure-what-about-privacy"></a>Les bots inscrits auprès du Bot Framework collectent-ils des informations personnelles ? Si c’est le cas, comment puis-je m’assurer qu’elles sont protégées et sécurisées ? Qu’en est-il de la confidentialité ?

Chaque bot est un service propre. Dans le cadre du Code de conduite du développeur, les développeurs de ces services sont tenus de fournir des conditions d’utilisation et des déclarations de confidentialité.  Vous pouvez accéder à ces informations à partir de la carte du bot dans le répertoire du bot.

Pour fournir le service d’E/S, le Bot Framework transmet votre message et le contenu du message (y compris votre ID) du service de conversation utilisé au bot.

### <a name="how-do-you-ban-or-remove-bots-from-the-service"></a>Comment excluez-vous ou supprimez-vous des bots du service ?

Les utilisateurs peuvent signaler la présence d’un bot inapproprié en se reportant à la carte de contact du bot dans le répertoire. Les développeurs doivent se conformer aux conditions d’utilisation de Microsoft pour participer au service.

### <a name="which-specific-urls-do-i-need-to-whitelist-in-my-corporate-firewall-to-access-bot-services"></a>Quelles URL dois-je mettre sur la liste verte de mon pare-feu d’entreprise pour accéder aux services de bot ?

Vous devez mettre les URL suivantes sur la liste verte de votre pare-feu d’entreprise :
- login.botframework.com (authentification des bots)
- login.microsoftonline.com (authentification des bots)
- westus.api.cognitive.microsoft.com (pour l’intégration du système de traitement du langage naturel Luis.ai)
- state.botframework.com (stockage de l’état des bots pour le prototypage)
- cortanabfchanneleastus.azurewebsites.net (canal Cortana)
- cortanabfchannelwestus.azurewebsites.net (canal Cortana)
- *.botFramework.com (canaux)

## <a name="rate-limiting"></a>Limitation du débit
### <a name="what-is-rate-limiting"></a>Qu’est-ce que la limitation du débit ?
Le service Bot Framework doit protéger ses clients (et se protéger lui-même) des modèles d’appels abusifs (attaques par déni de service, par exemple), de sorte qu’aucun bot ne puisse nuire aux performances des autres bots. Pour assurer ce type de protection, nous avons ajouté des limitations de débit (ou limitations de bande passante) à tous les points de terminaison. En appliquant une limitation de débit, nous pouvons limiter la fréquence à laquelle un bot peut effectuer un appel de spécifique. Par exemple, lorsque la limitation de débit est activée, si un bot veut envoyer un grand nombre d’activités, il devra les espacer dans le temps. Notez que la limitation de débit ne vise pas à limiter le volume total pour un bot. Elle est conçue pour éviter toute utilisation abusive de l’infrastructure de conversation, ne respectant pas les modèles de conversation humains.

### <a name="how-will-i-know-if-im-impacted"></a>Comment savoir si je suis concerné ?
Il est peu probable que vous soyez concerné par la limitation de débit, même si vous traitez des volumes élevés. Généralement, la limitation de débit s’applique en cas d’envoi massif d’activités (à partir d’un bot ou d’un client), de test de charge extrême ou de bogue. Lorsqu’une demande est limitée, une réponse HTTP 429 (Demandes trop nombreuses) est renvoyée avec un en-tête Retry-After qui indique le délai (en secondes) à l’issue duquel une nouvelle tentative de requête pourra aboutir. Vous pouvez collecter ces informations en activant l’analyse pour votre bot via Azure Application Insights. Vous pouvez également ajouter du code dans votre bot pour enregistrer les messages. 

### <a name="how-does-rate-limiting-occur"></a>Dans quels cas la limitation de débit s’applique-t-elle ?
La limitation de débit s’applique dans les cas suivants :
-   Un bot envoie des messages trop fréquemment.
-   Le client d’un bot envoie des messages trop fréquemment.
-   Des clients Direct Line demandent un nouveau WebSocket trop fréquemment.

### <a name="what-are-the-rate-limits"></a>Quelles sont les limitations de débit ?
Nous ajustons en permanence les limitations de débit et les modérons autant que possible, tout en protégeant notre service et nos utilisateurs. Étant donné que les seuils changent occasionnellement, nous ne publions aucun chiffre pour l’instant. Si vous êtes affecté par la limitation de débit, n’hésitez pas à nous contacter à l’adresse [bf-reports@microsoft.com](mailto://bf-reports@microsoft.com).

## <a name="related-services"></a>Services connexes
### <a name="how-does-the-bot-framework-relate-to-cognitive-services"></a>Quel est le lien entre le Bot Framework et Cognitive Services ?

Le Bot Framework et [Cognitive Services](http://www.microsoft.com/cognitive) sont de nouvelles fonctionnalités introduites dans le cadre de [Microsoft Build 2016](http://build.microsoft.com), qui seront également intégrées à Cortana Intelligence Suite lors de sa mise à disposition générale. Ces deux services s’appuient sur des années de recherche et d’utilisation dans le cadre de produits Microsoft populaires. Grâce à ces fonctionnalités, combinées avec « Cortana Intelligence », toutes les organisations peuvent tirer parti de la puissance des données, du cloud et de l’intelligence pour générer leurs propres systèmes intelligents ; des systèmes ouvrant la voie à de nouvelles opportunités, accélérant leurs activités et leur permettant de dominer les secteurs dans lesquels elles servent leurs clients.

### <a name="what-is-cortana-intelligence"></a>Qu’est-ce que Cortana Intelligence ?

Cortana Intelligence est une suite entièrement gérée de traitement du Big Data et d’analyse avancée axée sur l’intelligence, qui convertit vos données en action intelligente.  
Cette suite complète regroupe des technologies issues de plusieurs années de recherche et d’innovation menées par Microsoft (couvrant l’analytique avancée, le Machine Learning, le stockage de Big Data et le traitement dans le cloud). Elle offre notamment les avantages suivants :

* Elle vous permet de collecter, gérer et stocker toutes vos données. Ces données peuvent gagner en volume de façon économique et transparente au fil du temps par le biais d’un processus évolutif et sécurisé.
* Elle fournit une analytique simple et actionnable s’appuyant sur le cloud, vous permettant de prévoir, prescrire et automatiser la prise de décision pour les problèmes les plus exigeants.
* Elle ouvre la voie à des systèmes intelligents reposant sur des agents et services cognitifs, qui vous permettent de voir, d’entendre, d’interpréter et de comprendre le monde qui vous entoure de façon plus naturelle et avec plus de contexte.

Avec Cortana Intelligence, notre ambition est d’aider nos clients d’entreprise à créer de nouvelles opportunités, à accélérer leurs activités et à s’imposer comme leaders dans leurs secteurs d’activité.

### <a name="what-is-the-direct-line-channel"></a>Qu’est-ce que le canal Direct Line ?

Direct Line est une API REST qui vous permet d’ajouter votre bot dans votre service, votre application mobile ou votre page web.

Vous pouvez écrire un client pour l’API Direct Line à l’aide de n’importe quel langage. Il vous suffit de coder vers le [protocole Direct Line][DirectLineAPI], de générer un secret dans la page de configuration Direct Line et de communiquer avec votre bot depuis n’importe quel emplacement où se trouve votre code.

Direct Line est adapté aux éléments suivants :

* Applications mobiles sur iOS, Android, Windows Phone et d’autres systèmes d’exploitation
* Applications de bureau sur Windows, OSX et d’autres systèmes d’exploitation
* Pages web requérant plus de capacités de personnalisation que celles offertes par le [canal Discussion Web incorporable][WebChat]
* Applications de service à service

[DirectLineAPI]: http://docs.botframework.com/en-us/restapi/directline/
[Support]: bot-service-resources-links-help.md
[WebChat]: bot-service-channel-connect-webchat.md

