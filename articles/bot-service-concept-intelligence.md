---
title: Ajouter l’intelligence aux bots avec Cognitive Services | Microsoft Docs
description: Découvrez comment ajouter l’intelligence artificielle à vos bots avec Microsoft Cognitive Services, pour les rendre plus utiles et plus attrayants.
keywords: compréhension du langage naturel, extraction de connaissances, reconnaissance vocale, recherche web
author: RobStand
ms.author: rstand
manager: rstand
ms.topic: article
ms.prod: bot-framework
ms.date: 12/17/2017
ms.openlocfilehash: 2a558694cb96dd199894627146947ae1e7345183
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39300312"
---
# <a name="add-intelligence-to-bots-with-cognitive-services"></a>Ajouter l’intelligence aux bots avec Cognitive Services

Microsoft Cognitive Services vous permet d’exploiter une gamme de puissants algorithmes d’intelligence artificielle, développée par des experts en vision par ordinateur, reconnaissance vocale, traitement automatique du langage naturel, extraction de connaissances et recherche sur le web. Ces services simplifient de nombreuses tâches basées sur l’intelligence artificielle, et permettent d’ajouter rapidement à vos bots des technologies d’intelligence de pointe, avec seulement quelques lignes de code. Les API sont intégrées dans la plupart des langages et des plateformes modernes. Avec l’apprentissage automatique, les API s’améliorent constamment et deviennent de plus en plus intelligentes. Vous êtes donc toujours à la pointe. 

Les bots intelligents répondent comme s’ils pouvaient voir le monde comme des humains. Ils découvrent des informations et extraient des connaissances à partir de différentes sources, afin de fournir des réponses utiles. Mieux encore, ils apprennent au fur et à mesure de leur expérience et s’améliorent constamment. 

## <a name="language-understanding"></a>Compréhension de la langue

Les interactions entre les utilisateurs et les bots ne sont pas codifiées. De fait, les bots doivent comprendre le langage naturellement, en s’appuyant sur le contexte. Les API de langage naturel Cognitive Services fournissent des modèles de langage puissants permettant de déterminer ce que souhaitent les utilisateurs, et d’identifier les concepts et les entités d’une phrase donnée, pour que les bots puissent répondre avec l’action appropriée. Les cinq API prennent en charge plusieurs fonctionnalités d’analyse de texte, telles que la vérification orthographique, la détection du sentiment, la modélisation linguistique et l’extraction d’informations précises à partir d’un texte. 

Cognitive Services fournit les API suivantes pour la compréhension du langage naturel :

- Le <a href="https://www.microsoft.com/cognitive-services/en-us/language-understanding-intelligent-service-luis" target="_blank">service LUIS (Language Understanding Intelligent Service)</a> est capable de traiter le langage naturel à l’aide de modèles de langage prédéfinis ou ayant suivi un apprentissage personnalisé. Pour plus d’informations, consultez [Compréhension du langage naturel pour les bots](v4sdk/bot-builder-concept-luis.md).

- L’API <a href="https://www.microsoft.com/cognitive-services/en-us/text-analytics-api" target="_blank">Analyse de texte</a> détecte le sentiment, les expressions clés, les thèmes et la langue d’un texte.

- L’API <a href="https://www.microsoft.com/cognitive-services/en-us/bing-spell-check-api" target="_blank">Vérification orthographique Bing</a> fournit des fonctionnalités puissantes de vérification orthographique, et est capable de faire la différence entre les noms, les noms de marques et l’argot.

- Les <a href="https://docs.microsoft.com/en-us/azure/machine-learning/studio/text-analytics-module-tutorial" target ="_blank">modèles d’analyse de texte d’Azure Machine Learning Studio</a> permettent de créer et de rendre opérationnels des modèles d’analyse de texte, tels que la lemmatisation et le prétraitement de texte. Ces modèles peuvent vous aider à résoudre des problèmes d’analyse du sentiment ou de classification des documents.

En savoir plus sur la [compréhension du langage naturel][language] avec Microsoft Cognitive Services

## <a name="knowledge-extraction"></a>Extraction de connaissances

Cognitive Services propose quatre API de connaissances qui vous permettent d’identifier des entités nommées ou des expressions dans un texte non structuré, d’ajouter des recommandations personnalisées, de fournir des suggestions de saisie semi-automatique en fonction de l’interprétation naturelle des requêtes utilisateur, et d’effectuer des recherches dans des publications universitaires ou dans un service personnalisé de questions fréquentes (FAQ).

- L’API <a href="https://www.microsoft.com/cognitive-services/en-us/entity-linking-intelligence-service" target="_blank">Entity Linking Intelligence Service</a> annote le texte non structuré avec les entités pertinentes mentionnées dans le texte. Selon le contexte, un même mot ou une même expression peuvent avoir un sens différent. Ce service comprend le contexte du texte fourni et identifie chaque entité qui s’y trouve.    

- L’API <a href="https://www.microsoft.com/cognitive-services/en-us/knowledge-exploration-service" target="_blank">Service d’exploration des connaissances</a> interprète le langage naturel des requêtes utilisateur, et retourne des interprétations annotées offrant des expériences enrichies de recherche et de saisie semi-automatique, qui anticipent la saisie utilisateur. Les suggestions instantanées de saisie semi-automatique et l’affinement de la prédiction des requêtes sont basés sur vos propres données et sur les grammaires propres à chaque application, ce qui accélère les requêtes.    

- L’API <a href="https://www.microsoft.com/cognitive-services/en-us/academic-knowledge-api" target="_blank">Connaissances universitaires</a> est capable de retourner des recherches universitaires, des auteurs, des revues, des conférences, des domaines et des universités à partir de <a href="https://www.microsoft.com/en-us/research/project/microsoft-academic-graph/" target="_blank">Microsoft Academic Graph</a>. L’API Connaissances universitaires est une version spécialisée du Service d’exploration des connaissances. Elle fournit une base de connaissances qui utilise une boîte de dialogue de type graphe, permettant d’effectuer des recherches parmi des centaines de millions d’entités relatives à la recherche universitaire. Effectuez une recherche sur un domaine, un professeur, une université ou une conférence, et l’API retournera les publications et les entités associées. La grammaire prend également en charge les requêtes en langage naturel du type « Publications de Michael Jordan sur l’apprentissage automatique après 2010 ».

- <a href="https://qnamaker.ai" target="_blank">QnA Maker</a> est une API REST gratuite et facile à utiliser, ainsi qu’un service web qui effectue l’apprentissage de l’intelligence artificielle en vue de répondre aux questions des utilisateurs de manière naturelle, comme lors d’une conversation. Avec sa logique optimisée d’apprentissage automatique et sa capacité à intégrer un traitement du langage de pointe, QnA Maker condense des données semi-structurées, telles que des paires question-réponse, et retourne des réponses claires et utiles.

En savoir plus sur [l’extraction de connaissances][knowledge] avec Microsoft Cognitive Services

## <a name="speech-recognition-and-conversion"></a>Reconnaissance vocale et synthèse vocale

Utilisez les API de traitement vocal pour ajouter à votre bot des fonctionnalités vocales avancées, qui tirent parti d’algorithmes de pointe pour la reconnaissance de l’orateur, ainsi que pour la conversion parole-texte et la synthèse vocale. Les API de traitement vocal utilisent des modèles linguistiques et acoustiques intégrés, qui couvrent un large éventail de scénarios avec une haute précision. 

Pour les applications qui nécessitent davantage de personnalisation, vous pouvez utiliser Custom Recognition Intelligent Service. Vous pourrez ainsi ajuster les modèles linguistiques et acoustiques du moteur de reconnaissance vocale, en les adaptant au vocabulaire de l’application et au style d’élocution de vos utilisateurs.

Trois API sont disponibles dans Cognitive Services pour le traitement vocal et la synthèse vocale :

- L’API <a href="https://www.microsoft.com/cognitive-services/en-us/speech-api" target="_blank">Reconnaissance vocale Bing</a> fournit des fonctionnalités de conversion parole-texte et de synthèse vocale.
- <a href="https://www.microsoft.com/cognitive-services/en-us/custom-recognition-intelligent-service-cris" target="_blank">Custom Recognition Intelligent Service (CRIS)</a> permet de créer des modèles de reconnaissance vocale personnalisés pour adapter la conversion parole-texte au vocabulaire de l’application ou au style d’élocution de l’utilisateur.
- L’API <a href="https://www.microsoft.com/cognitive-services/en-us/speaker-recognition-api" target="_blank">Reconnaissance de l’orateur</a> permet l’identification de l’orateur et sa vérification par la voix.

Les ressources suivantes fournissent des informations supplémentaires sur l’ajout de la reconnaissance vocale à votre bot.

* [Bot Conversations for Apps video overview](https://channel9.msdn.com/events/Build/2017/P4114)
* [Microsoft.Bot.Client library for UWP or Xamarin apps](https://aka.ms/BotClient)
* [Bot Client Library Sample](https://aka.ms/BotClientSample)
* [Speech-enabled WebChat Client](https://aka.ms/BFWebChat)

En savoir plus sur la [reconnaissance vocale et la conversion parole-texte][speech] avec Microsoft Cognitive Services

## <a name="web-search"></a>Recherche Web

Les API Recherche Bing permettent d’ajouter à vos bots des fonctionnalités intelligentes de recherche web. Avec quelques lignes de code, vous pouvez accéder à des milliards de pages web, d’images, de vidéos, d’actualités et autres types de résultats. Vous pouvez configurer les API de manière à retourner des résultats selon la zone géographique, le marché ou la langue, pour une plus grande pertinence. Vous pouvez personnaliser davantage votre recherche en utilisant les paramètres de recherche pris en charge, tels que Safesearch pour filtrer le contenu adulte, ou Freshness pour retourner les résultats d’une date donnée.

Cinq API Recherche Bing sont disponibles dans Cognitive Services.

- L’API <a href="https://www.microsoft.com/cognitive-services/en-us/bing-web-search-api" target="_blank">Recherche Web</a> retourne des pages web, des images, des vidéos, des actualités, et autres résultats de recherche associés, avec un seul appel d’API.

- L’API <a href="https://www.microsoft.com/cognitive-services/en-us/bing-image-search-api" target="_blank">Recherche d’images</a> retourne des images avec des métadonnées améliorées (couleur dominante, type, etc.) et prend en charge plusieurs filtres d’image pour personnaliser les résultats.

- L’API <a href="https://www.microsoft.com/cognitive-services/en-us/bing-video-search-api" target="_blank">Recherche de vidéos</a> récupère des vidéos avec des métadonnées complètes (taille de la vidéo, qualité, prix, etc.) et des aperçus vidéo. De plus, elle prend en charge plusieurs filtres vidéo pour personnaliser les résultats.

- L’API <a href="https://www.microsoft.com/cognitive-services/en-us/bing-news-search-api" target="_blank">Recherche d’actualités</a> recherche les articles d’actualité du monde entier qui correspondent à votre requête ou qui correspondent à une tendance actuelle sur Internet.

- L’API <a href="https://www.microsoft.com/cognitive-services/en-us/bing-autosuggest-api" target="_blank">Suggestion automatique</a> propose des suggestions instantanées de saisie semi-automatique pour obtenir des résultats plus rapidement et avec moins d’efforts. 

En savoir plus sur la [recherche web][search] avec Microsoft Cognitive Services

## <a name="image-and-video-understanding"></a>Compréhension des images et des vidéos

Les API de vision apportent à vos bots des fonctionnalités avancées de compréhension des images et des vidéos. Leurs algorithmes de pointe permettent de traiter des images et des vidéos, et d’obtenir des informations sur lesquelles vous pouvez baser vos actions. Par exemple, vous pouvez les utiliser pour reconnaître des objets, le visage d’une personne, son âge, son sexe et même ses sentiments. 

Les API de vision prennent en charge plusieurs fonctionnalités de compréhension des images. Elles peuvent identifier le contenu adulte ou explicite, déterminer l’accentuation des couleurs, classer le contenu des images par catégorie, réaliser une reconnaissance optique de caractères et décrire une image avec des phrases complètes. Les API de vision prennent également en charge plusieurs fonctionnalités de traitement des images et des vidéos, telles que la génération intelligente de miniatures, ou la stabilisation de la sortie d’une vidéo.

Cognitive Services fournit quatre API, que vous pouvez utiliser pour traiter des images ou des vidéos :

- L’API <a href="https://www.microsoft.com/cognitive-services/en-us/computer-vision-api" target="_blank">Vision par ordinateur</a> extrait des informations détaillées sur les images (comme les objets ou les personnes qui s’y trouvent), détermine si l’image contient du contenu explicite ou adulte, et traite le texte (à l’aide de la reconnaissance optique de caractères) en images.

- L’API <a href="https://www.microsoft.com/cognitive-services/en-us/emotion-api" target="_blank">Émotion</a> analyse les visages humains et reconnaît les émotions parmi les huit catégories d’émotions humaines.

- L’API <a href="https://www.microsoft.com/cognitive-services/en-us/face-api" target="_blank">Visage</a> détecte les visages humains, les compare à des visages ressemblants, et peut même regrouper des personnes en fonction d’une similarité visuelle.

- L’API <a href="https://www.microsoft.com/cognitive-services/en-us/video-api" target="_blank">Vidéo</a> analyse et traite les vidéos pour stabiliser la sortie vidéo, détecte les mouvements, suit les visages et peut générer un récapitulatif de miniature de mouvement pour une vidéo.

En savoir plus sur la [compréhension des images et des vidéos][vision] avec Microsoft Cognitive Services

## <a name="additional-resources"></a>Ressources supplémentaires

Pour obtenir une documentation complète sur chaque produit et les références d’API associées, consultez la <a href="https://docs.microsoft.com/azure/cognitive-services" target="_blank">documentation Cognitive Services</a>.

[language]: https://docs.microsoft.com/en-us/azure/cognitive-services/luis/home
[search]: https://docs.microsoft.com/en-us/azure/cognitive-services/bing-web-search/search-the-web
[vision]: https://docs.microsoft.com/en-us/azure/cognitive-services/computer-vision/home
[knowledge]: https://docs.microsoft.com/en-us/azure/cognitive-services/kes/overview
[speech]: https://docs.microsoft.com/en-us/azure/cognitive-services/speech/home
[location]: https://docs.microsoft.com/en-us/azure/cognitive-services/
