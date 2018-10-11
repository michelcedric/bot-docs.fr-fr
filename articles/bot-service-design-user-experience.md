---
title: Concevoir l’expérience utilisateur | Microsoft Docs
description: Apprenez à concevoir votre robot pour offrir une expérience utilisateur engageante, en utilisant des contrôles utilisateur riches, ainsi que la reconnaissance et la synthèse vocales.
keywords: vue d’ensemble, conception, expérience utilisateur, contrôle utilisateur riche
author: matvelloso
ms.author: mateusv
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 09/20/2018
ms.openlocfilehash: 94882202eca48a4c662f0ffa32a80065953f13fa
ms.sourcegitcommit: 3cb288cf2f09eaede317e1bc8d6255becf1aec61
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/27/2018
ms.locfileid: "47389788"
---
# <a name="design-the-user-experience"></a>Concevoir l’expérience utilisateur

Vous pouvez créer des bots incluant un large éventail de fonctionnalités : texte, boutons, images, cartes enrichies affichées sous forme de carrousel ou de liste, etc. Toutefois, au final, chaque canal (tel que Facebook, Slack, Skype, etc.) contrôle la façon dont ses clients de messagerie affichent les fonctionnalités. Même lorsque plusieurs canaux prennent en charge une fonctionnalité, il est possible que chaque canal affiche cette fonctionnalité de façon légèrement différente. Lorsqu’un message contient des fonctionnalités non prises en charge de façon native par un canal, ce dernier peut tenter d’afficher une version réduite du contenu du message, sous forme de texte ou d’image statique, ce qui peut affecter considérablement l’apparence du message sur le client. Il arrive qu’une fonctionnalité ne soit pas du tout prise en charge par un canal. Par exemple, les clients GroupMe ne peuvent pas afficher d’indicateur de saisie.

## <a name="rich-user-controls"></a>Contrôles utilisateur riches

Les **contrôles utilisateur riches** sont des contrôles d’interface utilisateur communs tels que des boutons, des images, des carrousels et autres menus que le robot présente à l’utilisateur, avec lesquels celui-ci interagit pour communiquer ses choix et intentions. Un robot peut utiliser une série de contrôles d’interface utilisateur pour imiter une application, voire s’exécuter incorporé à l’intérieur d’une application. Quand un robot est incorporé dans une application ou un site web, il peut représenter pratiquement tout contrôle d’interface utilisateur en exploitant les fonctionnalités de l’application qui l’héberge. 

Depuis des décennies, les développeurs d’applications et de sites web s’appuient sur des contrôles d’interface utilisateur pour permettre aux utilisateurs d’interagir avec leurs applications. Ces mêmes contrôles d’interface utilisateur peuvent également être très efficaces dans des robots. Par exemple, les boutons constituent un excellent moyen de présenter à l’utilisateur des choix simples. Permettre à l’utilisateur à communiquer « Hôtels » en cliquant sur un bouton étiqueté **Hôtels** est plus facile et plus rapide que de le forcer à taper « Hôtels ». Cela en particulièrement vrai pour les appareils mobiles sur lesquels il est plus facile de cliquer que de taper.

## <a name="cards"></a>Cartes

Les cartes vous permettent de présenter à vos utilisateurs une variété de messages visuels, audio ou sélectionnables, et de faciliter le flux de conversation. Si un utilisateur doit choisir parmi un ensemble fixe d’éléments, vous pouvez afficher un carrousel de cartes, contenant chacune une image, une description textuelle et un bouton de sélection unique. Si un utilisateur dispose d’un ensemble de choix pour un élément, vous pouvez présenter une image unique plus petite et une collection de boutons avec différentes options parmi lesquelles choisir. Ont-ils demandé des informations supplémentaires sur un sujet ? Les cartes peuvent fournir des informations détaillées à l’aide d’une sortie audio ou vidéo, ou un reçu détaillant l’expérience d’achat. Il existe un éventail incroyablement vaste d’utilisations des cartes pour guider la conversation entre votre utilisateur et votre robot. Le type de carte que vous utilisez est déterminé par les besoins de votre application. Examinons de plus près les cartes, leurs actions et certaines utilisations recommandées. 

Les cartes du service Microsoft Bot sont des objets programmables contenant des collections standardisées de contrôles utilisateur riches, reconnus sur un vaste éventail de canaux. Le tableau suivant décrit la liste des cartes disponibles et les meilleures pratiques d’utilisation suggérées pour chaque type de carte.

| Type de carte | Exemples | Description |
| ---- | ---- | ---- |
| AdaptiveCard | ![Image de carte adaptative](./media/adaptive-card.png) | Format d’échange de carte ouvert restitué sous la forme d’un objet JSON. Généralement utilisé pour le déploiement de cartes via plusieurs canaux. Les cartes s’adaptent à l’apparence de chaque canal hôte. |
| AnimationCard | ![Image de carte d’animation](./media/animation-card1.png) | Carte pouvant lire des GIF animés ou des vidéos courtes. |
| AudioCard | ![Image de carte audio](./media/audio-card.png) | Carte pouvant lire un fichier audio. |
| HeroCard | ![Image de carte de bannière](./media/hero-card1.png) | Carte contenant une image de grande taille, un ou plusieurs boutons, et du texte. Généralement utilisée pour mettre en évidence une sélection d’utilisateur potentielle. |
| ThumbnailCard | ![Image de carte de miniature](./media/thumbnail-card.png) | Carte contenant une image miniature, un ou plusieurs boutons, et du texte. Généralement utilisée pour mettre en évidence les boutons destinés à une sélection d’utilisateur potentielle. |
| ReceiptCard | ![Image de carte de reçu](./media/receipt-card1.png) | Carte permettant à un robot de fournir un reçu à l’utilisateur. Elle contient généralement la liste des postes à inclure sur le reçu, la taxe et le total, ainsi que du texte. |
| SignInCard | ![Image de carte de connexion](./media/sign-in-card.png) | Carte permettant à un robot de demander à un utilisateur de se connecter. Elle contient généralement du texte et un ou plusieurs boutons sur lesquels l’utilisateur peut cliquer pour lancer le processus de connexion. |
| SuggestedAction | ![Image de carte d’actions suggérées](./media/suggested-actions.png) | Présente à l’utilisateur un ensemble de CardActions représentant un choix d’utilisateur. Cette carte disparaît après sélection de l’une des actions suggérées. |
| VideoCard | ![Image de carte de vidéo](./media/video-card.png) | Carte pouvant lire des vidéos. Généralement utilisée pour ouvrir une URL et diffuser en continu une vidéo disponible. |
| CardCarousel | ![Image de carrousel de cartes](./media/card-carousel.png) | Collection de cartes à défilement horizontal permettant à l’utilisateur de visualiser facilement une série de choix d’utilisateur possibles.|

Les cartes vous permettent de concevoir votre robot, puis de le faire fonctionner sur différents canaux. Cependant, certains types de cartes ne sont pas totalement pris en charge sur tous les canaux disponibles. 

Vous trouverez des instructions détaillées sur l’ajout de cartes à votre robot dans les sections [Ajouter des pièces jointes de média de carte riches](v4sdk/bot-builder-howto-add-media-attachments.md) et [Ajouter des actions suggérées aux messages](v4sdk/bot-builder-howto-add-suggested-actions.md). Vous pouvez également trouver des exemples de code pour les cartes ([C#](https://aka.ms/bot-cards-sample-code-cs)/[JS](https://aka.ms/bot-cards-sample-code-js)), pour les cartes adaptatives ([C#](https://aka.ms/bot-adaptive-cards-sample-code)/[JS](https://aka.ms/bot-adaptive-cards-js-sample-code)), les pièces jointes ([C#](https://aka.ms/bot-attachments-sample-code)/[JS](https://aka.ms/bot-attachments-js-sample-code)) et les actions suggérées ([C#](https://aka.ms/bot-suggested-actions-code)/[JS](https://aka.ms/bot-suggested-actions-js-code)).



Lors de la conception de votre robot, n’écartez pas automatiquement les éléments d’interface utilisateur courants comme n’étant pas « suffisamment intelligents ». Comme indiqué [précédemment](~/bot-service-design-principles.md#designing-a-bot), votre robot doit être conçu pour résoudre le problème de l’utilisateur de la manière la meilleure, la plus rapide et la plus simple possible. Ne cédez pas la tentation de commencer par incorporer la compréhension du langage naturel, car elle s’avère souvent superflue et introduit une complexité injustifiée.

> [!TIP]
> Commencez par utiliser les contrôles d’interface utilisateur minimaux, qui permettent au robot de résoudre le problème de l’utilisateur, puis ajoutez des éléments par la suite si ces contrôles s’avèrent insuffisants.


## <a name="text-and-natural-language-understanding"></a>Compréhension du texte et du langage naturel

Un robot peut accepter une entrée de **texte** de l’utilisateur, et tenter d’analyser cette entrée sur la base d’une mise en correspondance d’expression régulière ou à l’aide d’API de **compréhension du langage naturel**, telles que <a href="https://www.luis.ai" target="_blank">LUIS</a>. Selon le type d’entrée de l’utilisateur, la compréhension du langage naturel peut être ou non une bonne solution.

Dans certains cas, un utilisateur peut **répondre à une question très spécifique**. Par exemple, si le robot demande « Quel est votre nom ? », l’utilisateur peut répondre avec un texte spécifiant uniquement le nom (« Jean »), ou avec une phrase (« Mon nom est Jean »).

Poser des questions spécifiques permet de réduire l’étendue des réponses que le robot pourrait raisonnablement recevoir, et donc la complexité de la logique nécessaire pour analyser et comprendre la réponse. Par exemple, considérez la question ouverte et large : « Comment vous sentez-vous ? ». Comprendre les nombreuses permutations possibles des réponses possibles à cette question est une tâche très complexe.

En revanche, des questions spécifiques telles que « Ressentez-vous de la douleur ? Oui/Non » et « Où ressentez-vous de la douleur ? Poitrine/Tête/Bras/Jambe » invitent probablement à fournir des réponses plus spécifiques qu’un robot peut analyser et comprendre sans que vous deviez implémenter la compréhension du langage naturel. 

> [!TIP]
> Autant que possible, posez des questions spécifiques appelant des réponses dont l’analyse ne nécessite pas de fonctionnalités de compréhension du langage naturel. Cela simplifiera votre robot et augmentera sa capacité à comprendre l’utilisateur.

  
Dans d’autres cas, un utilisateur peut **taper une commande spécifique**. Par exemple, un robot de DevOps qui permet aux développeurs de gérer des machines virtuelles peut être conçu pour accepter des commandes spécifiques telles que « /ARRÊTER LA MACHINE VIRTUELLE XYZ » ou « /DÉMARRER LA MACHINE VIRTUELLE XYZ ». Concevoir un robot de façon à ce qu’il accepte des commandes spécifiques telles que celle-ci permet une bonne expérience utilisateur, car la syntaxe est facile à apprendre, et le résultat attendu de chaque commande est clair. En outre, le robot n’a pas besoin de fonctionnalités de compréhension du langage naturel, car l’entrée de l’utilisateur peut être aisément analysée à l’aide d’expressions régulières. 

> [!TIP]
> Concevoir un robot de façon à ce qu’il exige des commandes spécifiques de l’utilisateur permet souvent d’offrir une bonne expérience utilisateur, tout en éliminant la nécessité pour le robot d’être capable de comprendre le langage naturel.

  
Dans le cas d’un robot de *base de connaissances* ou de *questions et réponses*, l’utilisateur peut **poser des questions générales**. Par exemple, imaginez un robot capable de répondre à des questions basées sur le contenu de milliers de documents. <a href="https://qnamaker.ai" target="_blank">QnA Maker</a> et <a href="https://azure.microsoft.com/en-us/services/search/" target="_blank">Recherche Azure</a> sont deux technologies spécifiquement conçues pour ce type de scénario. Pour plus d’informations, voir [concevoir des robots de base de connaissances](bot-service-design-pattern-knowledge-base.md).

> [!TIP]
> Si vous concevez un robot destiné à répondre à des questions sur la base de données, structurées ou non, extraites de bases de données, de pages web ou de documents, songez à utiliser des technologies spécifiquement conçues pour ce type de scénario au lieu d’essayer de résoudre le problème avec la fonctionnalité de compréhension du langage naturel.

  
Dans d’autres scénarios, un utilisateur peut **taper des demandes simples formulées en langage naturel**. Par exemple, un utilisateur peut taper « Je veux une pizza pepperoni » ou « Y a-t-il des restaurants végétariens ouverts à moins de 3 km de chez moi ? ». Des API de compréhension du langage naturel telles que [LUIS.ai](https://www.luis.ai) conviennent parfaitement pour de tels scénarios. 

Les API permettent à votre robot d’extraire les composants clés du texte de l’utilisateur pour identifier l’intention de celui-ci. Lors de l’implémentation de fonctionnalités de compréhension du langage naturel dans votre robot, définissez des attentes réalistes en ce qui concerne le niveau de détail que les utilisateurs sont susceptibles de fournir en entrée. 

![manière dont les utilisateurs s’expriment](./media/bot-service-design-user-experience/buy-house.png)

> [!TIP]
> Lors de la création de modèles de langage naturel, ne supposez pas que les utilisateurs fourniront toutes les informations requises dans leur requête initiale. Concevez votre robot de façon à ce qu’il demande spécifiquement les informations dont il a besoin, en amenant l’utilisateur à les lui fournir au travers d’une série de questions, si nécessaire. 

  
## <a name="speech"></a>Speech

Un robot peut utiliser des entrées ou sorties **vocales** pour communiquer avec les utilisateurs. Dans les cas où un robot est conçu pour prendre en charge des appareils dépourvus de clavier ou d’écran, la voix est le seul moyen de communiquer avec l’utilisateur. 

## <a name="choosing-between-rich-user-controls-text-and-natural-language-and-speech"></a>Choix entre contrôles utilisateur riches, texte et langage naturel, et voix

Tout comme les gens communiquent entre eux à l’aide d’une combinaison de gestes, de voix et de symboles, les robots peuvent communiquer avec les utilisateurs à l’aide d’une combinaison de contrôles utilisateur riches, de texte (parfois en langage naturel) et de voix. Ces modes de communication peuvent être utilisés ensemble. Vous n’avez pas besoin d’en choisir un au détriment des autres. 

Par exemple, imaginez un « robot cuisinier » conçu pour aider les utilisateurs à réaliser des recettes, qui soit capable de donner des instructions en lisant des vidéos ou en affichant une série d’images pour expliquer comment procéder. Certains utilisateurs préfèrent tourner les pages de la recette ou poser des questions vocales au robot tandis qu’ils confectionnent leur plat. D’autres préfèrent toucher l’écran d’un appareil au lieu d’interagir vocalement avec le robot. Lors de la conception de votre robot, intégrez des éléments d’expérience utilisateur prenant en charge les manières dont les utilisateurs préféreront probablement interagir avec votre robot, compte tenu des cas d’utilisation spécifiques auxquels il est destiné. 

