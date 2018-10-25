---
title: Présentation de l’assistant personnalisé | Microsoft Docs
description: Découvrez comment créer votre propre assistant personnalisé.
author: darrenj
ms.author: darrenj
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 13/12/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: dcef01d8ba07d44aebaeeeecb5637af691caccb9
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2018
ms.locfileid: "49997076"
---
## <a name="custom-assistant-overview"></a>Présentation de l’assistant personnalisé

## <a name="overview"></a>Vue d’ensemble

Nous avons constaté que nos clients et partenaires souhaitaient ardemment être en mesure de fournir un assistant conversationnel adapté à leur marque et à leurs clients, et disponible sur un large éventail de canevas et d’appareils conversationnels. Dans la continuité de l’approche open source de Microsoft vis-à-vis du SDK Bot Framework, l’assistant personnel personnalisé open source offre un contrôle total sur l’expérience de l’utilisateur final et repose sur un ensemble de fonctionnalités de base. En outre, l’expérience peut être enrichie d’informations sur l’utilisateur final et sur les appareils/écosystèmes pour une expérience véritablement intégrée et intelligente.

Nous avons la conviction que nos clients doivent entretenir et enrichir leurs relations et leurs insights clients. Par conséquent, tout assistant personnalisé offre à nos clients et partenaires un contrôle complet sur l'expérience utilisateur. Le nom, la voix et la personnalité peuvent être modifiés pour répondre aux besoins de l’organisation. Notre solution d'assistant personnalisé simplifie la création de votre propre assistant et vous permet de démarrer en quelques minutes. 

La portée de l’assistant personnel personnalisé est vaste et celui-ci offre généralement aux utilisateurs finaux une large gamme de fonctionnalités. Pour accroître la productivité des développeurs et créer un écosystème dynamique d'expériences conversationnelles réutilisables, nous fournissons aux développeurs des exemples initiaux de compétences conversationnelles réutilisables. Ces compétences peuvent être ajoutées à l'application conversationnelle pour éclairer une expérience de conversation spécifique, comme la recherche d'un point d'intérêt, l'interaction avec le calendrier, les tâches, le courrier électronique et de nombreux autres scénarios. Les compétences sont entièrement personnalisables et se composent de différents modèles de langage, de dialogues et de code.

Nous exécutons actuellement une préversion initiale et travaillons en étroite collaboration avec des clients et partenaires initiaux au sein d'un référentiel open source pour créer les premières expériences et proposer cette version à un public plus large dans les mois à venir. 

![Schéma d’assistant personnalisé](media/enterprise-template/CustomAssistantDiagram.jpg)

## <a name="complete-control-of-the-user-experience"></a>Contrôle total sur l’expérience utilisateur

Vous détenez et contrôlez tous les aspects de l’expérience de l’utilisateur final. Cela inclut la marque, le nom, la voix, la personnalité, les réponses et l’avatar. Le code source vers l’assistant personnalisé et la prise en charge des compétences sont fournis en intégralité pour vous permettre d’effectuer les ajustements nécessaires.

## <a name="complete-ownership-and-control-of-data"></a>Propriété et contrôle complets sur les données

Votre assistant personnalisé est déployé dans votre abonnement Azure. Par conséquent, toutes les données générées par votre assistant (questions posées, comportement de l’utilisateur, etc.) se trouvent dans votre abonnement Azure. Pour en savoir plus, consultez la page [Cloud approuvé Azure Cognitive Services](https://www.microsoft.com/en-us/trustcenter/cloudservices/cognitiveservices) et plus particulièrement la section [Azure du Centre de confidentialité](https://www.microsoft.com/en-us/TrustCenter/CloudServices/Azure).

## <a name="your-assistant-anywhere"></a>Votre assistant, n’importe où...

L’assistant personnalisé s’appuie sur la plateforme d’intelligence artificielle conversationnelle Microsoft et par conséquent, peut être exposé via n’importe quel [canal](https://docs.microsoft.com/en-us/azure/bot-service/bot-service-manage-channels?view=azure-bot-service-4.0) Bot Framework, par exemple un chat web, Facebook Messenger, Skype, etc. En outre, via le canal de [ligne directe](https://docs.microsoft.com/en-us/azure/bot-service/rest-api/bot-framework-rest-direct-line-3-0-concepts?view=azure-bot-service-4.0), nous pouvons incorporer des expériences dans des applications bureau et mobiles, y compris des appareils tels que des voitures, des haut-parleurs, des réveils, etc.

## <a name="built-on-enterprise-grade-technology"></a>Piloté par une technologie professionnelle

La solution d’assistant personnalisé repose sur Azure Bot Service, Language Understanding Cognitive Service, Unified Speech, ainsi qu’un vaste ensemble de composants Azure de prise en charge, ce qui signifie que vous bénéficiez de [l’infrastructure globale Azure](https://azure.microsoft.com/en-gb/global-infrastructure/).

En outre, la prise en charge de Language Understanding est fournie par LUIS Cognitive Service qui prend en charge un large ensemble de langages [répertoriés ici](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/luis-supported-languages). [Translator Cognitive Service](https://azure.microsoft.com/en-us/services/cognitive-services/translator-text-api/) propose des fonctionnalités de traduction machine pour étendre encore davantage la portée de votre assistant personnalisé.

## <a name="integrated-and-context-aware"></a>Intégré et prenant en charge le contexte

Votre assistant personnalisé peut être intégré à votre appareil et écosystème pour une expérience véritablement intégrée et intelligente. Cette prise en compte du contexte permet de développer des expériences plus intelligentes et d’offrir davantage de personnalisation.

## <a name="3rd-party-assistant-integration"></a>Intégration d’assistant tiers

L’assistant personnalisé vous permet de proposer votre propre expérience unique, mais également de remettre aux utilisateurs finaux l’assistant numérique choisi pour certains types de questions.

## <a name="flexible-integration"></a>Intégration flexible

Notre architecture d’assistant personnalisé est flexible et peut être intégrée aux investissements existants que vous avez effectués dans des fonctionnalités de traitement en langage naturel ou de reconnaissance vocale basé sur un appareil, et bien sûr de l’intégrer à vos API et systèmes principaux existants.

## <a name="adaptive-cards"></a>Cartes adaptatives

Les [cartes adaptatives](https://adaptivecards.io/) offrent la possibilité à votre assistant personnalisé de renvoyer des éléments de l’expérience utilisateur (p. ex. des cartes, des images, des boutons), ainsi que des réponses de type texte. Si le canevas de conversation ou l’appareil est doté d’un écran, ces cartes adaptatives peuvent être affichées sur une large gamme d’appareils et de plateformes fournissant l’expérience de prise en charge quand cela est nécessaire. Vous pouvez trouver des exemples de cartes adaptatives [ici](https://adaptivecards.io/samples/) avec plus d’informations sur les options d’affichage dans la documentation [ici](https://docs.microsoft.com/en-us/adaptive-cards/rendering-cards/getting-started).


## <a name="skills"></a>Compétences

En plus de l’assistant de base, il existe un large éventail de fonctionnalités courantes que chaque développeur doit configurer. La productivité est un excellent exemple dans lequel chaque organisation doit créer des modèles de langage (LUIS), des dialogues (code), l’intégration (code) et la génération de langage (réponses) pour activer les scénarios communs de point d'intérêt, de calendrier ou de tâche.

Cela se complique ensuite avec le besoin de prendre en charge plusieurs langages et se traduit par une importante quantité de travail à fournir par les organisations créant leur propre assistant.

Notre solution d’assistant personnalisé inclut une nouvelle fonctionnalité de compétence permettant d’ajouter d’autres fonctionnalités à un assistant personnalisé par configuration uniquement. 

Tous les aspects de chaque compétence (modèle de langage, dialogues, code d’intégration et génération de langages) sont entièrement personnalisables par les développeurs, puisque l’intégralité du code source est fournie sur GitHub avec l’assistant personnalisé.

## <a name="example-scenarios"></a>Exemples de scénarios

L’assistant personnalisé couvre un grand nombre de scénarios industriels, dont certains sont présentés ci-dessous pour référence.

- Industrie automobile : l’assistant personnel activé par la voix intégré à la voiture offre aux utilisateurs finaux la possibilité d’effectuer des opérations classiques au sein d’un véhicule (p. ex. navigation, radio), ainsi que des scénarios orientés productivité comme par exemple décaler des réunions quand vous êtes en retard, ajouter des éléments à votre liste de tâches et des expériences proactives où la voiture peut suggérer d’accomplir des tâches en fonction d’événements comme le démarrage du moteur, le retour à la maison ou l’activation du régulateur de vitesse. Des cartes adaptatives sont affichées dans l’intégration de l’unité principale et vocale effectuée via les interactions de type Wake Word (mot déclencheur) ou Push-To-Talk (Appuyer pour parler).

- Hôtellerie : assistant personnel activée par la voix intégré à un appareil dans une chambre d’hôtel, fournissant un large éventail de scénarios orientés hôtellerie (p. ex. prolongation du séjour, demander un checkout tardif, service en chambre), y compris conciergerie et la possibilité de rechercher des restaurants et des lieux à visiter. Des liens facultatifs vers vos comptes de productivité offrent des expériences encore plus personnalisées, comme les appels d’alarme suggérés, les avertissements en lien avec la météo et l’apprentissage des modèles entre les séjours. Une évolution de la personnalisation TV actuelle présente dans les chambres d’hôtel actuellement.

- Enterprise : expériences d’assistant d’employé de la société activé par la voix et le texte intégrées aux appareils de l’entreprise et aux canevas de conversation existants (p. ex équipes, chat web, Slack) permettant aux employés de gérer leurs calendriers, de rechercher des salles de réunion disponibles, de rechercher des personnes avec des compétences particulières ou d’effectuer des opérations RH. 

## <a name="getting-started"></a>Mise en route

Nous exécutons actuellement une préversion initiale et travaillons en étroite collaboration avec des clients et partenaires initiaux au sein d’un référentiel open source pour créer les premières expériences et proposer cette version à un public plus large dans les mois à venir. Pour faire part de votre intérêt et commencer, veuillez remplir [ce formulaire](https://aka.ms/customassistantpreviewform) pour que nous puissions vous contacter.

