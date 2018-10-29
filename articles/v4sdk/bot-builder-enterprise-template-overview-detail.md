---
title: Présentation détaillée du modèle de bot d’entreprise | Microsoft Docs
description: Obtenez des informations sur la conception du modèle de bot d’entreprise
author: darrenj
ms.author: darrenj
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 09/18/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 2ee47d472518e54cf07b86648e270e40f8f015c0
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2018
ms.locfileid: "49997226"
---
# <a name="enterprise-template---detailed-overview"></a>Modèle d’entreprise – Présentation détaillée

> [!NOTE]
> Cet article s’applique à la version v4 du SDK. 

Le modèle de bot d’entreprise permet de rassembler un certain nombre de meilleures pratiques que nous avons identifiées lors de la création d’expériences conversationnelles. En outre, il automatise l’intégration des composants, ce qui est particulièrement avantageux pour les développeurs Azure Bot Service. Dans cette section, vous découvrirez dans quel contexte les décisions clés ont été prises afin de mieux comprendre le fonctionnement du modèle.

## <a name="introduction-card"></a>Carte de présentation

De nombreuses expériences conversationnelles font face à un problème de taille : les utilisateurs finaux ne savent pas comment commencer la discussion et posent des questions d’ordre général. De son côté, le bot n’est pas le mieux placé pour répondre à ce type de questions. La première impression est importante ! Avec une carte de présentation, le bot a la possibilité de présenter ses fonctionnalités à l’utilisateur final et il lui suggère quelques questions dont l’utilisateur peut se servir pour lancer la conversation. C’est également un excellent moyen de faire apparaître la personnalité de votre bot.

Vous disposez d’un exemple de carte de présentation simple qui est fourni de manière prédéfinie. Vous pouvez l’adapter à vos besoins.

## <a name="basic-language-understanding-luis-intents"></a>Intentions Language Understanding (LUIS) de base

Chaque bot doit disposer d’un niveau de base pour la compréhension du langage conversationnel. Par exemple, tous les bots doivent pouvoir traiter les messages d’accueil sans difficulté. En règle générale, les développeurs ont besoin de créer ces intentions de base et de fournir des données d’apprentissage initiales pour commencer. Le modèle de bot d’entreprise contient des exemples de fichiers LU pour vous aider à démarrer. Ainsi, vous n’avez pas besoin de les créer pour chaque projet et vous avez l’assurance de disposer, par défaut, d’un niveau de fonctionnalité de base.

Les fichiers LU fournissent les intentions suivantes en allemand, anglais, espagnol, français et italien.

> Greeting, Help, Cancel, Restart, Escalate, ConfirmYes, ConfirmNo, ConfirmMore, Next, Goodbye

Le format [LU](https://github.com/Microsoft/botbuilder-tools/blob/master/packages/Ludown/docs/lu-file-format.md) ressemble au format MarkDown. Par conséquent, la modification et le contrôle de la source sont facilités. L’outil [LuDown](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/Ludown) est ensuite utilisé pour convertir les fichiers .LU dans les modèles LUIS. Vous pouvez ensuite publier ces modèles dans votre abonnement LUIS via le portail ou l’outil de ligne de commande [LUIS](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/LUIS) associé.

## <a name="content-moderator"></a>Content Moderator

Content Moderator permet de détecter l’utilisation potentielle d’injures ainsi que la présence d’informations d’identification personnelle (PII). Grâce à cette fonctionnalité, les bots peuvent réagir aux obscénités ou au partage d’informations d’identification personnelle de l’utilisateur. Par exemple, un bot peut s’excuser et passer la main à un humain ou ne pas stocker les données de télémétrie s’il a détecté des informations d’identification personnelle.

Grâce au composant d’intergiciel fourni, les textes sont filtrés et analysés par le biais de ```TextModeratorResult``` sur l’objet TurnState.

## <a name="telemetry"></a>Télémétrie

Fournir des insights sur l’engagement des utilisateurs de votre bot s’est révélé être un avantage de taille. Ces insights peuvent vous aider à comprendre les niveaux d’engagement utilisateur, les fonctionnalités du bot qui sont sollicitées (intentions), ainsi que les questions auxquelles le bot ne peut pas répondre. Vous identifiez ainsi les lacunes du bot et pouvez les combler à l’aide de nouveaux articles QnAMaker par exemple.

L’intégration d’Application Insights apporte des informations opérationnelles et techniques importantes de manière prédéfinie. Cette fonctionnalité peut également servir à capturer des événements spécifiques associés au bot (des messages envoyés et reçus avec les opérations LUIS et QnAMaker).

Intrinsèquement, le niveau de télémétrie du bot est lié aux données de télémétrie techniques et opérationnelles, ce qui vous permet d’examiner la façon dont est traitée la question d’un utilisateur et vice versa.

Un composant d’intergiciel associé à une classe wrapper autour des classes SDK QnAMaker et LuisRecognizer fournit un bon moyen pour collecter un ensemble cohérent d’événements. Ces événements cohérents peuvent ensuite être utilisés par les outils Application Insights avec des outils tels que Power BI.

Chaque projet créé à l’aide du modèle de bot d’entreprise contient un exemple de tableau de bord Power BI. Pour en savoir plus, consultez la section [PowerBI](bot-builder-enterprise-template-powerbi.md).

## <a name="dispatcher"></a>Répartiteur

Pour obtenir un bon rendu lors de la première vague d’expériences conversationnelles, le modèle de conception clé tire parti de Language Understanding (LUIS) et de QnAMaker. L’apprentissage de LUIS est basé sur des tâches que votre bot peut effectuer pour un utilisateur final et l’apprentissage de QnAMaker est basé sur des connaissances plus générales.

Tous les énoncés entrants (questions) sont acheminés vers LUIS pour l’analyse. Si l’intention d’un énoncé donné n’a pas été identifiée, ce dernier ne porte aucune intention (None). Ensuite, QnAMaker est utilisé pour tenter de trouver une réponse à donner à l’utilisateur final.

Ce schéma fonctionne bien. Toutefois, il rencontre des problèmes dans deux scénarios principaux.

- Parfois, les énoncés du modèle LUIS et de QnAMaker se chevauchent légèrement. On remarque alors un comportement étrange où LUIS essaie de traiter une question qui aurait dû être dirigée vers QnaMaker.
- Lorsque deux modèles LUIS ou plus coexistent, le bot doit appeler chacun d’eux et effectuer une comparaison pour évaluer les intentions afin de déterminer où envoyer tel ou tel énoncé. Comme il n’existe aucune comparaison commune des scores de base entre les modèles, ce schéma ne fonctionne pas efficacement et offre une expérience utilisateur de qualité médiocre.

Le [répartiteur](https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-tutorial-dispatch?view=azure-bot-service-4.0&tabs=csaddref%2Ccsbotconfig) remédie efficacement à ce problème en extrayant des énoncés de chaque modèle LUIS configuré et des questions de QnAMaker, puis en créant un modèle LUIS de répartition centrale.

Ainsi, le bot peut rapidement identifier le modèle LUIS ou le composant qui doit gérer un énoncé donné et il garantit que les données QnAMaker sont placées au plus haut niveau d’intention, pas seulement une intention neutre (None) comme avant.

Cet outil de répartition offre également une fonctionnalité d’évaluation qui met en évidence les confusions et les chevauchements entre les modèles LUIS et les bases de connaissances QnAMaker. Ainsi, les problèmes sont détectés avant le déploiement.

Le répartiteur est utilisé au cœur de chaque projet créé à l’aide du modèle de bot d’entreprise. Le modèle de répartition est utilisé dans la classe `MainDialog` pour déterminer si la cible correspond à un modèle LUIS ou QnA. Pour LUIS, le modèle LUIS secondaire est appelé et renvoie les intentions et les entités à son habitude.

## <a name="qnamaker"></a>QnAMaker

[QnAMaker](https://www.qnamaker.ai/) offre la possibilité aux non-développeurs d’organiser des connaissances générales sous la forme de paires de questions et réponses. Ces connaissances peuvent être importées depuis des sources de données (FAQ, manuels), mais aussi de manière interactive au sein du portail QnaMaker.

Un exemple de jeu d’entrées QnA est fourni dans le format de fichier [LU](https://github.com/Microsoft/botbuilder-tools/blob/master/packages/Ludown/docs/lu-file-format.md) dans le dossier QnA de CogSvcModels. [LuDown](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/Ludown) est ensuite utilisé dans le cadre du script de déploiement pour créer un fichier JSON QnAMaker. L’outil de ligne de commande [QnAMaker](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/QnAMaker) l’utilise ensuite pour publier des éléments dans la base de connaissances QnAMaker.
