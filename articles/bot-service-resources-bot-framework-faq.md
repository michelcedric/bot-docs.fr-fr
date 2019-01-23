---
title: 'Service de bot : Forum Aux Questions | Microsoft Docs'
description: Une liste de questions fréquemment posées sur les éléments du Bot Framework et la disponibilité de nouvelles fonctionnalités.
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 08/28/2018
ms.openlocfilehash: 9b77f05b77017b17ba63e83fa2a8b58e483f9bf8
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/11/2019
ms.locfileid: "54225804"
---
# <a name="bot-framework-frequently-asked-questions"></a>Bot Framework : Forum Aux Questions

Cet article répond à certaines questions fréquemment posées sur le Bot Framework.

## <a name="background-and-availability"></a>Contexte et disponibilité
### <a name="why-did-microsoft-develop-the-bot-framework"></a>Pourquoi Microsoft a-t-il développé le Bot Framework ?

Alors que l’interface utilisateur conversationnelle est déjà à notre portée, peu de développeurs disposent actuellement de l’expertise et des outils nécessaires pour créer de nouvelles expériences conversationnelles ou doter les applications et services existants d’une interface conversationnelle pour les utilisateurs. Nous avons créé le Bot Framework pour aider les développeurs à générer des bots exceptionnels et à les connecter avec les utilisateurs, et ce, quel que soit le lieu de la conversation, y compris sur les canaux de haute qualité Microsoft.

### <a name="what-is-the-v4-sdk"></a>Qu’est-ce que le Kit de développement logiciel (SDK) v4 ?
Le kit SDK Bot Framework v4 s’appuie sur les commentaires et les enseignements tirés des précédents kits SDK Bot Framework. Il offre des niveaux d’abstraction appropriés et assure une grande modularité des blocs de construction de bot. Vous pouvez commencer avec un bot simple et le sophistiquer progressivement à l’aide d’une infrastructure modulaire et extensible. Vous trouverez un [FAQ](https://github.com/Microsoft/botbuilder-dotnet/wiki/FAQ) sur le Kit de développement logiciel (SDK) sur GitHub.

## <a name="bot-framework-sdk-version-3-lifetime-support"></a>Prise en charge de la durée de vie du kit SDK version 3 du Bot Framework 
Les bots du SDK V3 continuent de s’exécuter et d’être pris en charge par Azure Bot Service.  Depuis la publication du SDK V4 du Bot Framework, comme avec d’autres frameworks, nous continuons de prendre en charge le SDK V3 en appliquant les correctifs de bogues de sécurité haute priorité et les mises à jour de la couche connecteur / protocole.  Les clients peuvent compter sur la prise en charge de v3 pendant toute l’année 2019.

### <a name="what-is-microsoft-plan-for-supporting-existing-v3-bots-what-happens-to-my-v3-bots-will-my-v3-bots-stop-working"></a>Qu’est-ce que prévoit Microsoft pour prendre en charge les bots V3 existants ? Qu’advient-il de mes bots V3 ? Mes bots V3 vont-ils s’arrêter de fonctionner ?
Les bots du SDK V3 continuent de s’exécuter et d’être pris en charge par Azure Bot Service.  Depuis la publication du SDK V4 du Bot Framework, comme avec d’autres frameworks, nous continuons de prendre en charge le SDK V3 en appliquant les correctifs de bogues de sécurité haute priorité et les mises à jour de la couche connecteur / protocole.  Les clients peuvent compter sur la prise en charge de v3 pendant toute l’année 2019.
- Azure Bot Service et Bot Framework V3 sont deux produits en disponibilité générale (GA) et sont entièrement pris en charge. Les bibliothèques de protocoles et de connecteurs Bot Framework sous-jacentes n’ont pas changé et sont partagées entre les SDK V3 et V4.  
- Les bots créés avec le SDK V3 du Bot Framework (BotBuilder) continuent d’être pris en charge tout au long de l’année 2019. 
- Les clients peuvent continuer à créer des bots V3 en utilisant le portail Azure ou les outils Azure CLI.

### <a name="what-happens-to-my-bot-written-to-rest--bot-framework-protocol-31"></a>Qu’advient-il de mes bots écrits avec REST et Bot Framework Protocol 3.1 ?
- Azure Bot Service et Bot Framework V3 sont deux produits en disponibilité générale (GA) et sont entièrement pris en charge.
- Le protocole Bot Framework n’a pas changé et est partagé entre les SDK V3 et V4.  

### <a name="will-there-be-more-updates-additional-development-for-the-v3-sdk-or-just-bugfixes"></a>D’autres mises à jour ou développements sont-ils prévus pour le SDK V3, ou juste des correctifs de bogues ?  
- Nous allons mettre à jour V3 avec des améliorations mineures, principalement au niveau de la couche connecteur, et avec des correctifs de bogues de sécurité haute priorité.  
- Les mises à jour pour V3 seront publiées deux fois par an selon les besoins, en fonction des correctifs de bogues et/ou des changements de protocole obligatoires. 
- Nous prévoyons actuellement de publier des versions correctives mineures de V3 sur NuGet et NPM pour nos kits SDK C# et JavaScript.

### <a name="why-v4-is-not-backwards-compatible-with-v3"></a>Pourquoi V4 n’est pas rétrocompatible avec V3 ?
- Au niveau protocole, la communication entre votre application de conversation (c’est-à-dire votre bot) et les différents canaux utilise le protocole Bot Framework Activity qui est le même entre V3 et V4. La même infrastructure Azure Bot Service (AZURE BOT SERVICE) sous-jacente prend en charge à la fois les bots V3 et les bots V4.
- Le SDK V4 du Bot Framework offre une expérience de développement de conversation avec une architecture de SDK modulaire et extensible, permettant aux développeurs de créer des applications de conversation robustes et sophistiquées. La conception extensible de V4 est basée sur le feedback des clients qui suggérait que les modèles et les primitives de dialogue du SDK V3 étaient trop rigides et limitaient l’extensibilité.  

### <a name="what-is-the-general-migration-strategy-i-have-a-v3-bot-how-can-i-migrate-it-to-v4-can-i-migrate-my-v3-bot-to-v4"></a>Quelle est la stratégie de migration générale ? J’ai un bot V3, comment le migrer vers V4 / puis-je migrer mon bot V3 vers V4 ?
- Actuellement, l’aide permettant de migrer des bots créés avec un SDK V3 vers le SDK V4 se présente sous forme de documentations et d’exemples. Nous n’avons pas prévu de fournir de couche de compatibilité de SDK V3 dans le SDK V4 qui permettrait aux bots créés avec V3 de fonctionner dans un bot V4. 
- Si vous avez déjà des bots du SDK V3 du Bot Framework en production, ne vous inquiétez pas, ils continueront de fonctionner de la même façon dans un avenir prévisible. 
- Le SDK V4 du Bot Framework est une évolution du très apprécié SDK V3. V4 est une version majeure contenant des changements cassants qui empêchent les bots V3 de s’exécuter sur le SDK V4 plus récent. 

### <a name="should-i-build-new-a-bot-using-v3-or-v4"></a>Est-il préférable de créer un bot avec V3 ou V4 ?
- Pour les nouvelles expériences de conversation, nous vous recommandons de commencer un nouveau bot avec le SDK V4 du Bot Framework.
- Si vous êtes déjà familiarisé avec le SDK V3 du Bot Framework, prenez le temps de découvrir la nouvelle version et les fonctionnalités offertes dans le nouveau [SDK V4 du Bot Framework](http://aka.ms/botframeowrkoverview).
- Si vous avez déjà des bots du SDK V3 du Bot Framework en production, ne vous inquiétez pas, ils continueront de fonctionner de la même façon dans un avenir prévisible.
- Vous pouvez créer des bots avec le SDK V4 du Bot Framework et avec le SDK V3 plus ancien dans le portail Azure et la ligne de commande Azure. 

## <a name="channels"></a>Canaux
### <a name="when-will-you-add-more-conversation-experiences-to-the-bot-framework"></a>Quand allez-vous ajouter des expériences de conversation au Bot Framework ?

Nous prévoyons d’apporter des améliorations continues au Bot Framework et notamment d’ajouter des canaux. Cependant, nous ne pouvons pas fournir de calendrier pour l’instant.  
Si vous souhaitez que nous ajoutions un canal spécifique à l’infrastructure, [indiquez-le nous][Support].

### <a name="i-have-a-communication-channel-id-like-to-be-configurable-with-bot-framework-can-i-work-with-microsoft-to-do-that"></a>J’aimerais pouvoir configurer un canal de communication spécifique avec le Bot Framework. Puis-je travailler avec Microsoft à cette fin ?

Nous n’avons pas fourni aux développeurs de mécanisme général pour ajouter de nouveaux canaux au Bot Framework. Cependant, vous pouvez connecter votre bot à votre application par le biais de [l’API Direct Line][DirectLineAPI]. Si vous développez un canal de communication et que vous souhaitez travailler avec nous pour activer votre canal dans le Bot Framework, [n’hésitez pas à nous contacter][Support].

### <a name="if-i-want-to-create-a-bot-for-skype-what-tools-and-services-should-i-use"></a>Quels outils et services dois-je utiliser pour créer un bot pour Skype ?

Le Bot Framework est conçu pour générer, connecter et déployer des bots réactifs, performants et évolutifs de haute qualité pour Skype et de nombreux autres canaux. Vous pouvez utiliser le Kit de développement logiciel (SDK) pour créer des bots prenant en charge le texte, les sms, les images, les boutons et les cartes (utilisés aujourd’hui pour la majorité des interactions de bot dans le cadre d’expériences conversationnelles), ainsi que des interactions de bot spécifiques à Skype, comme des expériences audio et vidéo enrichies.

Si vous disposez déjà d’un excellent bot et que vous souhaitez toucher les utilisateurs de Skype, vous pouvez le connecter facilement à Skype (ou à tout autre canal pris en charge) par le biais de l’API REST Bot Framework (dans la mesure où il dispose d’un point de terminaison REST accessible par Internet).

## <a name="security-and-privacy"></a>Sécurité et confidentialité
### <a name="do-the-bots-registered-with-the-bot-framework-collect-personal-information-if-yes-how-can-i-be-sure-the-data-is-safe-and-secure-what-about-privacy"></a>Les bots inscrits auprès du Bot Framework collectent-ils des informations personnelles ? Si c’est le cas, comment puis-je m’assurer qu’elles sont protégées et sécurisées ? Qu’en est-il de la confidentialité ?

Chaque bot est un service propre. Dans le cadre du Code de conduite du développeur, les développeurs de ces services sont tenus de fournir des conditions d’utilisation et des déclarations de confidentialité.  Vous pouvez accéder à ces informations à partir de la carte du bot dans le répertoire du bot.

Pour fournir le service d’E/S, le Bot Framework transmet votre message et le contenu du message (y compris votre ID) du service de conversation utilisé au bot.

### <a name="can-i-host-my-bot-on-my-own-servers"></a>Puis-je héberger mon bot sur mes propres serveurs ?
Oui. Votre bot peut être hébergé n’importe où sur Internet. Sur vos propres serveurs, dans Azure, ou dans n’importe quel autre centre de données. La seule exigence à respecter réside dans le fait que le bot doit exposer un point de terminaison HTTPS accessible publiquement.

### <a name="how-do-you-ban-or-remove-bots-from-the-service"></a>Comment excluez-vous ou supprimez-vous des bots du service ?

Les utilisateurs peuvent signaler la présence d’un bot inapproprié en se reportant à la carte de contact du bot dans le répertoire. Les développeurs doivent se conformer aux conditions d’utilisation de Microsoft pour participer au service.

### <a name="which-specific-urls-do-i-need-to-whitelist-in-my-corporate-firewall-to-access-bot-framework-services"></a>Quelles URL dois-je mettre sur la liste verte de mon pare-feu d’entreprise pour accéder aux services Bot Framework ?
Si vous disposez d’un pare-feu pour le trafic sortant qui bloque le trafic vers Internet en provenance de votre bot, vous devrez mettre en liste verte les URL ci-après dans ce pare-feu :
- login.botframework.com (authentification des bots)
- login.microsoftonline.com (authentification des bots)
- westus.api.cognitive.microsoft.com (pour l’intégration du système de traitement du langage naturel Luis.ai)
- state.botframework.com (stockage de l’état des bots pour le prototypage)
- cortanabfchanneleastus.azurewebsites.net (canal Cortana)
- cortanabfchannelwestus.azurewebsites.net (canal Cortana)
- *.botframework.com (canaux)

### <a name="can-i-block-all-traffic-to-my-bot-except-traffic-from-the-bot-connector-service"></a>Puis-je bloquer la totalité du trafic en direction de mon bot, à l’exception du trafic émanant du service Bot Connector ?
 Non. Ce type de mise en liste verte d’adresses IP ou de DNS est irréaliste. Le service Bot Framework Connector est hébergé dans les centres de données Azure du monde entier, et la liste des adresses IP Azure évolue constamment. La mise en liste verte de certaine adresses IP peut donc cesser de fonctionner du jour au lendemain lorsque les adresses IP Azure changent.
 
### <a name="what-keeps-my-bot-secure-from-clients-impersonating-the-bot-framework-connector-service"></a>De quelle manière mon bot est-il protégé contre les clients qui empruntent l’identité du service Bot Framework Connector ?
1. L’URL du service (ServiceUrl) est encodée dans le jeton de sécurité qui accompagne chaque requête adressée à votre bot ; autrement dit, même si un attaquant parvient à accéder au jeton, ils ne peut pas rediriger la conversation vers une nouvelle URL ServiceUrl. Cette approche est appliquée par toutes les implémentations du Kit de développement logiciel (SDK) et documentée dans nos articles de [référence](https://docs.microsoft.com/en-us/azure/bot-service/rest-api/bot-framework-rest-connector-authentication?view=azure-bot-service-3.0#bot-to-connector) concernant l’authentification.

2. Si le jeton entrant est manquant ou incorrect, le Kit de développement logiciel (SDK) Bot Framework ne génère aucun jeton en réponse. Cela limite les dommages potentiels en cas de configuration incorrecte du bot.
3. À l’intérieur du bot, vous pouvez vérifier manuellement l’URL ServiceUrl fournie dans le jeton. Cette opération fragilise le bot en cas d’apport de modifications à la topologie du service ; elle est donc possible, mais déconseillée.


Notez que ceci concerne les connexions sortantes à partir du bot vers Internet. Il n’existe aucune liste d’adresses IP ou de noms DNS destinée à être utilisée par le service Bot Framework Connector pour communiquer avec le bot. La mise en liste verte d’adresses IP entrantes n’est pas prise en charge.

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

[DirectLineAPI]: https://docs.microsoft.com/en-us/azure/bot-service/rest-api/bot-framework-rest-direct-line-3-0-concepts
[Support]: bot-service-resources-links-help.md
[WebChat]: bot-service-channel-connect-webchat.md

