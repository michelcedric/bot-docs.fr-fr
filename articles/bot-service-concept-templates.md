---
title: Modèles Bot Service | Microsoft Docs
description: Découvrez les différents modèles que vous pouvez utiliser pour créer un robot à l’aide de Bot Service.
author: robstand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 286d7057afb28983964ef992de2c11cebd74e0da
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39299673"
---
# <a name="bot-service-templates"></a>Modèles Bot Service
Bot Service comprend cinq modèles pour vous lancer dans la création de robots. Ces modèles fournissent un robot entièrement fonctionnel prêt à l’emploi vous permettant de démarrer rapidement. Lorsque vous [créer un robot](bot-service-quickstart.md), vous devez choisir un modèle et le langage du kit de développement logiciel de votre robot.

Chaque modèle offre un point de départ en fonction des fonctionnalités de base du robot. 

## <a name="basic-bot"></a>Robot de base
Pour créer un robot qui utilise des dialogues afin de répondre aux entrées de l’utilisateur, choisissez le modèle de base. Le **modèle de base** crée un robot avec le minimum de fichiers et de code pour commencer. Il renvoie à l’utilisateur ce qu’il saisit. Vous pouvez vous servir de ce modèle pour commencer à créer un flux de conversation dans votre robot.

## <a name="form-bot"></a>Robot de formulaire
Pour créer un robot qui recueille les données d’un utilisateur par le biais d’une conversation guidée, choisissez le **modèle de formulaire**. Un robot de formulaire est conçu pour recueillir un ensemble précis d’informations auprès de l’utilisateur. Par exemple, un robot conçu pour enregistrer la commande de sandwichs d’un utilisateur doit collecter des informations telles que le type de pain, le choix des accompagnements, la taille du sandwich, etc.

Les robots créés avec le modèle de formulaire en langage C# gèrent les formulaires à l’aide de [FormFlow](dotnet/bot-builder-dotnet-formflow.md) et les robots créés avec le modèle de formulaire en langage Node.js gèrent les formulaires à l’aide de [cascades](nodejs/bot-builder-nodejs-dialog-waterfall.md).

## <a name="language-understanding-bot"></a>Robot de compréhension du langage
Pour créer un robot qui utilise des modèles de langage naturel afin de comprendre l’intention de l’utilisateur, choisissez le **modèle de compréhension du langage**. Ce modèle s’appuie sur le module <a href="https://www.luis.ai" target="_blank">Language Understanding (LUIS)</a> permettant de comprendre le langage naturel.

Si un utilisateur envoie un message de type « donne-moi des infos sur les entreprises de réalité virtuelle », votre robot peut se servir de LUIS pour interpréter la signification du message. En utilisant LUIS, vous pouvez déployer rapidement un terminal HTTP qui interprète la saisie de l’utilisateur en fonction de l’intention qu’il exprime (trouver des nouvelles) et des entités clés en présence (entreprises de réalité virtuelle). LUIS vous permet d’indiquer l’ensemble des intentions et des entités pertinentes dans le pour votre application, puis vous guide tout au long du processus de création d’une application de compréhension du langage.

Lorsque vous créez un robot à l’aide du modèle de compréhension du langage, Bot Service crée une application LUIS correspondante qui est vide, c’est-à-dire qu’elle renvoie toujours `None`. Pour mettre à jour votre modèle d’application LUIS afin qu’il puisse interpréter les saisies des utilisateurs, vous devez vous connecter à <a href="https://www.luis.ai" target="_blank">LUIS</a>, cliquer sur **Mes applications**, sélectionner l’application que le service vous a créée, puis créer des intentions, définir des entités et entraîner l’application.

## <a name="question-and-answer-bot"></a>Robot de questions-réponses
Pour créer un bot qui synthétise des données semi-structurées telles que des couples questions-réponses sous forme de réponses distinctes et utiles, choisissez le **modèle de questions-réponses**. Ce modèle se sert du service <a href="https://qnamaker.ai">QnA Maker</a> pour analyser les questions et apporter des réponses. 

Lorsque vous créez un robot avec le modèle de questions-réponses, vous devez vous connecter à <a href="https://qnamaker.ai">QnA Maker</a> et créer un nouveau service QnA. Le service QnA vous fournira l’identifiant de la base de connaissances et la clé d’abonnement qui vous permettront de mettre à jour les valeurs des [paramètres d’application](bot-service-manage-settings.md) avec l’identifiant **QnAKnowldegebasedId** et la clé **QnASubscriptionKey**. Une fois ces valeurs définies, votre robot peut répondre à toutes les questions à disposition du service QnA dans sa base de connaissances.

## <a name="proactive-bot"></a>Robot proactif
Pour créer un robot capable d’envoyer des messages de façon proactive à l’utilisateur, choisissez le **modèle proactif**. En règle générale, chaque message envoyé à l’utilisateur par un robot se rapporte directement à la saisie précédente de l’utilisateur. Dans certains cas néanmoins, un robot peut avoir besoin d’envoyer à l’utilisateur des informations qui ne sont pas directement liées au message le plus récent de l’utilisateur. Il s’agit de **messages proactifs**. Les messages proactifs peuvent être utiles dans différents cas de figure. Par exemple, lorsqu’un bot définit une minuterie ou un rappel, il peut avoir besoin d’avertir l’utilisateur à leur arrivée. Ou encore, lorsqu’un robot reçoit une notification concernant un événement externe, il peut avoir besoin de communiquer cette information à l’utilisateur. 

Lorsque vous créez un robot à l’aide du modèle proactif, plusieurs ressources Azure sont automatiquement créées et ajoutées à votre groupe de ressources. Par défaut, ces ressources Azure sont déjà configurées pour un cas très simple de messagerie proactive. 

| Ressource | Description |
|----|----|
| Stockage Azure | Permet de créer la file d’attente. |
| Application Azure Function | Fonction `queueTrigger` Azure déclenchée chaque fois qu’un message se trouve dans la file d’attente. Elle communique avec Bot Service à l’aide de [Direct Line](https://docs.microsoft.com/bot-framework/rest-api/bot-framework-rest-direct-line-3-0-concepts). Cette fonction se sert de la liaison du robot pour envoyer le message comme élément de la charge utile du déclencheur. Notre fonction d’exemple transmet le message de l’utilisateur en l’état depuis la file d’attente.
| Service de robot | Votre robot. Contient la logique qui réceptionne le message de l’utilisateur, l’ajoute à la file d’attente Azure, reçoit les déclencheurs de la fonction Azure et renvoie le message reçu via la charge utile du déclencheur. |

Le diagramme suivant montre comment fonctionnent les événements déclenchés lorsque vous créez un robot à l’aide du modèle proactif.

![Vue d’ensemble de l’exemple de robot proactif](~/media/bot-proactive-diagram.png)

Le processus commence lorsque l’utilisateur envoie un message à votre robot via les serveurs Bot Framework (étape 1).

## <a name="next-steps"></a>Étapes suivantes
Maintenant que vous connaissez les différents modèles, découvrez les services cognitifs à utiliser dans les robots.

> [!div class="nextstepaction"]
> [Services cognitifs pour les robots](bot-service-concept-intelligence.md)
