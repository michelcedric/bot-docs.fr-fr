---
title: Présentation de l’assistant virtuel | Microsoft Docs
description: Découvrez comment créer votre propre assistant virtuel
author: darrenj
ms.author: darrenj
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 13/12/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 075c090571eb7a45efc57b915a4bc912fc9c55df
ms.sourcegitcommit: 87b5b0ca9b0d5e028ece9f7cc4948c5507062c2b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/24/2018
ms.locfileid: "47029843"
---
# <a name="virtual-assistant-solution-overview"></a>Présentation de la solution d’assistant virtuel

## <a name="overview"></a>Vue d’ensemble
Nous avons constaté que nos clients et partenaires souhaitaient ardemment être en mesure de fournir un assistant conversationnel adapté à leur marque et à leurs clients, et disponible sur un large éventail de canevas et d’appareils conversationnels. Dans la continuité de l’approche open source de Microsoft vis-à-vis du Kit de développement logiciel (SDK) Bot Framework, la solution d’assistant personnel personnalisé open source offre un contrôle total sur l’expérience de l’utilisateur final et repose sur un ensemble de fonctionnalités de base. En outre, l’expérience peut être enrichie d’informations sur l’utilisateur final et sur les appareils/écosystèmes pour une expérience véritablement intégrée et intelligente.

Nous avons la conviction que nos clients doivent entretenir et enrichir leurs relations et leurs insights clients. Par conséquent, tout assistant virtuel offre à nos clients et partenaires un contrôle complet sur l’expérience utilisateur via le code en open source sur GitHub. Le nom, la voix et la personnalité peuvent être modifiés pour répondre aux besoins de l’organisation. Notre solution d’assistant virtuel simplifie la création de votre assistant et vous permet de démarrer en quelques minutes. Vous pouvez par la suite continuer à le développer complètement grâce à nos outils spécifiques.

La portée de l’assistant virtuel est vaste et celui-ci offre généralement aux utilisateurs finals une large gamme de fonctionnalités. Pour accroître la productivité des développeurs et créer un écosystème dynamique d’expériences conversationnelles réutilisables, nous fournissons aux développeurs des exemples initiaux de compétences conversationnelles réutilisables. Ces compétences peuvent être ajoutées à l’application conversationnelle pour éclairer une expérience de conversation spécifique, comme la recherche d’un point d’intérêt, l’interaction avec le calendrier, les tâches, les e-mails et de nombreux autres scénarios. Les compétences sont entièrement personnalisables et se composent de différents modèles de langage, de dialogues et de code.

Nous exécutons actuellement une préversion initiale et travaillons en étroite collaboration avec des clients et partenaires initiaux au sein d’un référentiel open source pour créer les premières expériences et proposer cette version à un public plus large dans les mois à venir.

![Schéma d’assistant virtuel](./media/enterprise-template/customassistantdiagram.jpg)

## <a name="whats-in-the-box"></a>Fonctionnalités présentes 

L’Assistant virtuel repose sur les fonctionnalités du [modèle d’entreprise](./bot-builder-enterprise-template-overview.md), qui propose des fonctions basiques pour les expériences conversationnelles, y compris des intentions conversationnelles de base dans plusieurs langages, l’outil Dispatch, QnA et des insights conversationnelles. Les fonctionnalités suivantes de l’assistant sont proposées actuellement. Nous prévoyons d’en développer d’autres et allons travailler en étroite collaboration avec les clients et les partenaires pour établir la feuille de route.

Fonctionnalité | Description |
------------ | -------------
Mise en route | Un exemple de flux Mise en route permettant à votre assistant d’accueillir l’utilisateur et de recueillir des informations initiales.
Architecture d’événements | Les événements dans le contexte de l’assistant virtuel permettent à l’application cliente qui héberge l’assistant (dans un navigateur web ou sur un appareil tel qu’une voiture ou un haut-parleur) d’échanger des informations sur les événements de l’utilisateur ou de l’appareil tout en recevant également des événements pour effectuer des opérations sur l’appareil.
Comptes liés | Dans un scénario dirigé par la reconnaissance vocale, il n’est pas pratique pour un utilisateur d’entrer son nom d’utilisateur et son mot de passe pour les systèmes de prise en charge via des commandes vocales. Par conséquent, une expérience de compagnon distincte permet à l’utilisateur de se connecter et de donner l’autorisation à un assistant virtuel de récupérer des jetons pour une utilisation ultérieure.
Activation des compétences | Actuellement, un vaste ensemble de fonctionnalités courantes existe, que chaque développeur doit configurer. Notre solution d’assistant virtuel inclut une nouvelle fonctionnalité de compétence permettant à d’autres fonctionnalités d’être ajoutées à un assistant virtuel par configuration uniquement et de fournir un mécanisme d’authentification aux compétences pour qu’elles puissent demander des jetons pour les activités en aval.
Compétence Point d’intérêt | La compétence Point d’intérêt (POI) en préversion fournit un modèle de langage complet pour rechercher des points d’intérêt et demander des directions. La compétence propose actuellement l’intégration à Azure Maps.
Compétence Calendrier | La compétence Calendrier en préversion fournit un modèle de langage complet pour des activités associées au calendrier courantes. La compétence est actuellement intégrée à Microsoft Graph (Office 365/Outlook.com) avec une prise en charge des API Google prévue prochainement.
Compétence E-mail | La compétence E-mail en préversion fournit un modèle de langage complet pour des activités associées aux e-mails courantes. La compétence est actuellement intégrée à Microsoft Graph (Office 365/Outlook.com) avec une prise en charge des API Google prévue prochainement.
Compétence ToDo | La compétence ToDo en préversion fournit un modèle de langage complet pour des activités associées aux tâches courantes. La compétence est actuellement intégrée à OneNote avec une prise en charge de Microsoft Graph (outlookTask) prévue prochainement.
Intégration d’appareil | Nos Kits de développement logiciel (SDK) Azure Bot Service (Direct Line) et Kits de développement logiciel (SDK) de reconnaissance vocale et de carte adaptative permettent une intégration facile des plateformes aux appareils. Des exemples d’intégration d’appareils supplémentaires et une plateforme incluant Edge sont prévus.
Ateliers de test | En plus de l’émulateur Bot Framework, un atelier de test basé sur Web Chat est proposé, afin de pouvoir tester des scénarios d’authentification plus complexes. Un atelier de test basé sur une console simple démontre l’approche d’échange de messages pour aider à structurer la facilité d’intégration des appareils.
Déploiement automatisé | Toutes les ressources Azure requises pour votre assistant sont automatiquement déployées : inscription du bot, Azure App Service, LUIS, QnAMaker, Content Moderator, CosmosDB, Stockage Azure et Application Insights. En outre, des modèles LUIS pour toutes les compétences QnAMaker et des modèles Dispatch sont créés, formés et publiés pour permettre d’effectuer immédiatement des tests.
Modèle de langage automobile | Un modèle de langage automobile couvrant des domaines clés tels que les appels téléphoniques, la navigation et le contrôle des fonctionnalités intégrées à une voiture sera prochainement disponible.

## <a name="example-scenarios"></a>Exemples de scénarios

L’assistant virtuel couvre un grand nombre de scénarios industriels, dont certains sont présentés ci-dessous pour référence.

- Industrie automobile : l’assistant personnel activé par la voix intégré à la voiture offre aux utilisateurs finals la possibilité d’effectuer des opérations classiques au sein d’un véhicule (p. ex. navigation, radio). Il est également adapté aux scénarios orientés productivité comme par exemple pour décaler des réunions quand vous êtes en retard, ajouter des éléments à votre liste de tâches, et propose des expériences proactives où la voiture peut suggérer d’accomplir des tâches en fonction d’événements comme le démarrage du moteur, le trajet du retour à la maison ou l’activation du régulateur de vitesse. Des cartes adaptatives sont affichées dans l’intégration de l’unité principale et de reconnaissance vocale effectuée via les interactions de type Wake Word (mot déclencheur) ou Push-To-Talk (appuyer pour parler).

- Hôtellerie : assistant personnel activé par la voix intégré à un appareil dans une chambre d’hôtel, fournissant un large éventail de scénarios orientés hôtellerie (p. ex. prolongation du séjour, demander à libérer la chambre plus tard, service en chambre), y compris un service de conciergerie et la fonctionnalité de recherche de restaurants et de lieux à visiter. Des liens facultatifs vers vos comptes de productivité offrent des expériences encore plus personnalisées, comme les appels d’alarme suggérés, les avertissements en lien avec la météo et l’apprentissage des modèles entre les séjours. Une évolution de la personnalisation TV actuelle présente dans les chambres d’hôtel actuellement.

- Enterprise : expériences d’assistant d’employé de la société activé par la voix et le texte intégrées aux appareils de l’entreprise et aux canevas de conversation existants (p. ex équipes, chat web, Slack) permettant aux employés de gérer leurs calendriers, de rechercher des salles de réunion disponibles, de rechercher des personnes avec des compétences particulières ou d’effectuer des opérations RH.

## <a name="virtual-assistant-principles"></a>Principes de l’assistant virtuel

### <a name="your-data-your-brand-and-your-experience"></a>Vos données, votre marque et votre expérience
Vous détenez et contrôlez tous les aspects de l’expérience de l’utilisateur final. Cela inclut la personnalisation en fonction de la marque, le nom, la voix, la personnalité, les réponses et l’avatar. Le code source vers l’assistant virtuel et les compétences de prise en charge sont fournis en intégralité pour vous permettre d’effectuer les ajustements nécessaires.

Votre assistant virtuel est déployé dans votre abonnement Azure. Par conséquent, toutes les données générées par votre assistant (questions posées, comportement de l’utilisateur, etc.) se trouvent dans votre abonnement Azure. Consultez [Cognitive Services Azure Trusted Cloud](https://www.microsoft.com/en-us/trustcenter/cloudservices/cognitiveservices) (Cloud approuvé Azure Cognitive Services) et la section [Azure du Centre de confidentialité](https://www.microsoft.com/en-us/TrustCenter/CloudServices/Azure) plus spécifiquement, pour plus d’informations.

### <a name="write-it-once-embed-it-anywhere"></a>Écrivez-le une fois, intégrez-le partout
L’assistant virtuel s’appuie sur la plateforme d’intelligence artificielle conversationnelle Microsoft et par conséquent, peut être exposé via n’importe quel [canal](https://docs.microsoft.com/en-us/azure/bot-service/bot-service-manage-channels?view=azure-bot-service-4.0) Bot Framework, par exemple Web Chat, Facebook Messenger, Skype, etc. 

En outre, via le canal [Direct Line](https://docs.microsoft.com/en-us/azure/bot-service/rest-api/bot-framework-rest-direct-line-3-0-concepts?view=azure-bot-service-4.0), nous pouvons incorporer des expériences dans des applications bureau et mobiles, y compris des appareils tels que des voitures, des haut-parleurs, des réveils, etc.

### <a name="enterprise-grade-solutions"></a>Solutions professionnelles
La solution d’assistant virtuel repose sur Azure Bot Service, Language Understanding Cognitive Service, Unified Speech, ainsi qu’un vaste ensemble de composants Azure de prise en charge, ce qui signifie que vous bénéficiez de [l’infrastructure globale Azure](https://azure.microsoft.com/en-gb/global-infrastructure/), notamment ISO 27018, HIPPA, PCI DSS et des certifications SOC 1, 2 et 3.

En outre, la prise en charge de Language Understanding est fournie par LUIS Cognitive Service qui prend en charge un large ensemble de langages [répertoriés ici](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/luis-supported-languages). [Translator Cognitive Service](https://azure.microsoft.com/en-us/services/cognitive-services/translator-text-api/) propose des fonctionnalités de traduction machine pour étendre encore davantage la portée de votre assistant virtuel.

### <a name="integrated-and-context-aware"></a>Intégré et prenant en compte le contexte
Votre assistant virtuel peut être intégré à votre appareil et écosystème pour une expérience véritablement intégrée et intelligente. Cette prise en compte du contexte permet de développer des expériences plus intelligentes et d’offrir davantage de personnalisation.

### <a name="3rd-party-assistant-integration"></a>Intégration d’assistant tiers
L’assistant virtuel vous permet de proposer votre propre expérience unique, mais également de remettre aux utilisateurs finals l’assistant numérique choisi pour certains types de questions.

### <a name="flexible-integration"></a>Intégration flexible
Notre architecture d’assistant virtuel est flexible et peut être intégrée aux investissements existants que vous avez effectués dans des fonctionnalités de traitement en langage naturel ou de reconnaissance vocale basé sur des appareils, et bien sûr de l’intégrer à vos API et systèmes principaux existants.

### <a name="adaptive-cards"></a>Cartes adaptatives
Les [cartes adaptatives](https://adaptivecards.io/) offrent la possibilité à votre assistant virtuel de renvoyer des éléments de l’expérience utilisateur (p. ex. des cartes, des images, des boutons), ainsi que des réponses de type texte. Si le canevas de conversation ou l’appareil est doté d’un écran, ces cartes adaptatives peuvent être affichées sur une large gamme d’appareils et de plateformes fournissant une expérience utilisateur de prise en charge quand cela est nécessaire. Vous pouvez trouver des exemples de cartes adaptatives [ici](https://adaptivecards.io/samples/) avec plus d’informations sur les options d’affichage dans la documentation [ici](https://docs.microsoft.com/en-us/adaptive-cards/rendering-cards/getting-started).

### <a name="skills"></a>Compétences
En plus de l’assistant de base, il existe un large éventail de fonctionnalités courantes que chaque développeur doit configurer. La productivité est un excellent exemple dans lequel chaque organisation doit créer des modèles de langage (LUIS), des dialogues (code), l’intégration (code) et la génération de langage (réponses) pour activer les expériences populaires de calendrier, de tâche ou d’e-mail.

Cela se complique ensuite avec le besoin de prendre en charge plusieurs langages, et se traduit par une importante quantité de travail à fournir par les organisations créant leur propre assistant.

Notre solution d’assistant virtuel inclut une nouvelle fonctionnalité de compétence permettant d’ajouter d’autres fonctionnalités à un assistant personnalisé par configuration uniquement. 

Tous les aspects de chaque compétence (modèle de langage, dialogues, code d’intégration et génération de langages) sont entièrement personnalisables par les développeurs, puisque l’intégralité du code source est fournie sur GitHub avec l’assistant virtuel.

## <a name="getting-started"></a>Mise en route

Nous exécutons actuellement une préversion initiale et travaillons en étroite collaboration avec des clients et partenaires initiaux au sein d’un référentiel open source pour créer les premières expériences et proposer cette version à un public plus large dans les mois à venir. Pour faire part de votre intérêt et commencer, veuillez remplir [ce formulaire](https://aka.ms/customassistantpreviewform) pour que nous puissions vous contacter.

