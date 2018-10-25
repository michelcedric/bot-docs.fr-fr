---
title: Concevoir des bots de connaissances | Microsoft Docs
description: Découvrez les différentes façons de concevoir un bot de connaissances qui recherche et retourne des informations en réponse à l’entrée ou à la requête de l’utilisateur.
author: matvelloso
ms.author: mateusv
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: cognitive-services
ms.date: 12/13/2017
ms.openlocfilehash: e228209b4d239a05f9c76203e9fd2fb342c14d36
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2018
ms.locfileid: "49999286"
---
# <a name="design-knowledge-bots"></a>Concevoir des bots de connaissances

Un bot de connaissances peut être conçu pour fournir des informations sur pratiquement n’importe quel sujet. Par exemple, un bot de connaissances peut répondre à des questions sur des événements telles que « Quels sont les événements de bot présents à cette conférence ? », « Quand a lieu le prochain spectacle de reggae ? » ou « Qui est Tame Impala ? ». Un autre peut répondre à des questions d’ordre informatique telles que « Comment mettre à jour mon système d’exploitation ? » ou « Comment réinitialiser mon mot de passe ? ». Un autre encore peut répondre aux questions sur des contacts telles que « Qui est Jean Dupont ? » ou « Quel est l’e-mail de Jeanne Dupont ? ». 

Quel que soit le cas d’usage pour lequel un bot de connaissances est conçu, son objectif de base est toujours le même : trouver et retourner les informations que l’utilisateur a demandées en tirant parti d’un corpus de données, telles que des données relationnelles dans une base de données SQL, des données JSON dans un magasin non relationnel ou des fichiers PDF dans un magasin de documents. 

## <a name="search"></a>Recherche

La fonctionnalité de recherche peut être un outil précieux au sein d’un bot. 

Tout d’abord, la « recherche partielle » permet à un bot de retourner des informations qui sont susceptibles d’être pertinentes pour la question de l’utilisateur, sans que celui-ci doive fournir une entrée précise. Par exemple, si l’utilisateur demande à un bot de connaissances musicales des informations sur « impala » (au lieu de « Tame Impala »), le bot peut répondre avec des informations qui sont susceptibles d’être pertinentes pour cette entrée.

![Structure du dialogue](~/media/bot-service-design-pattern-knowledge-base/fuzzySearch2.png)

Les scores de recherche indiquent le niveau de confiance pour les résultats d’une recherche spécifique, ce qui permet à un bot d’ordonner ses résultats en conséquence ou même d’adapter sa communication en fonction du niveau de confiance. Par exemple, si le niveau de confiance est élevé, le bot peut répondre par « Voici l’événement qui correspond le mieux à votre recherche : ».

![Structure du dialogue](~/media/bot-service-design-pattern-knowledge-base/searchScore2.png)

Si le niveau de confiance est faible, le bot peut répondre par « Hmm... recherchiez-vous un de ces événements ? ».

![Structure du dialogue](~/media/bot-service-design-pattern-knowledge-base/searchScore1.png)

### <a name="using-search-to-guide-a-conversation"></a>Utilisation de Recherche pour guider une conversation

Si ce qui motive chez vous la génération d’un bot est l’activation de fonctionnalités de moteur de recherche de base, vous n’avez peut-être pas besoin d’un bot du tout. Qu’offre une interface de conversation par rapport à un moteur de recherche classique dans un navigateur web ? 

Les bots de connaissances sont généralement plus efficaces quand ils sont conçus pour guider la conversation. Une conversation est composée d’un échange entre un utilisateur et un bot, avec la possibilité pour celui-ci de poser des questions d’éclaircissement, de présenter des options et de valider les résultats selon un procédé qu’une recherche de base ne pourrait pas utiliser. Par exemple, le bot suivant guide l’utilisateur par le biais d’une conversation qui affine et filtre un jeu de données jusqu’à ce qu’il trouve les informations recherchées par l’utilisateur.

![Structure du dialogue](~/media/bot-service-design-pattern-knowledge-base/guidedConvo1.png)

![Structure du dialogue](~/media/bot-service-design-pattern-knowledge-base/guidedConvo2.png)

![Structure du dialogue](~/media/bot-service-design-pattern-knowledge-base/guidedConvo3.png)

![Structure du dialogue](~/media/bot-service-design-pattern-knowledge-base/guidedConvo4.png)

En traitant l’entrée de l’utilisateur à chaque étape et en présentant les options appropriées, le bot guide l’utilisateur vers les informations qu’il recherche. Outre ces informations, le bot peut même fournir des conseils sur la façon de trouver plus efficacement des informations similaires à l’avenir. 

![Structure du dialogue](~/media/bot-service-design-pattern-knowledge-base/Training.png)

### <a name="azure-search"></a>Recherche Azure

À l’aide de <a href="https://azure.microsoft.com/en-us/services/search/" target="_blank">Recherche Azure</a>, vous pouvez créer un index de recherche efficace qu’un bot peut facilement explorer, affiner et filtrer. Envisagez un index de recherche qui est créé à l’aide du portail Azure.

![Structure du dialogue](~/media/bot-service-design-pattern-knowledge-base/search3.PNG)

Comme vous souhaitez être en mesure d’accéder à toutes les propriétés du magasin de données, vous devez définir chaque propriété comme « Récupérable ». Comme vous souhaitez être en mesure de trouver les musiciens par nom, vous devez définir la propriété **Name** (Nom) en tant que « Possibilité de recherche ». Enfin, comme vous souhaitez être en mesure de filtrer les époques des musiciens en les affinant, vous marquez la propriété **Era** (Époque) en tant que « À choix multiples » et « Filtrable ». 

L’affinement détermine les valeurs qui existent dans le magasin de données pour une propriété donnée, ainsi que la grandeur de chaque valeur. Par exemple, cette capture d’écran montre qu’il existe 5 époques distinctes dans le magasin de données :

![Structure du dialogue](~/media/bot-service-design-pattern-knowledge-base/facet.png)

Le filtrage, à son tour, sélectionne uniquement les instances spécifiées d’une propriété donnée. Par exemple, vous pouvez filtrer le jeu de résultats ci-dessus pour qu’il ne contienne que les éléments où **Era** (Époque) est égal à « Romantic » (Romantique). 

> [!NOTE]
> Consultez <a href="https://github.com/ryanvolum/AzureSearchBot" target="_blank">un exemple de bot</a> pour obtenir un exemple complet de bot de connaissances qui est créé à l’aide d’Azure DocumentDB, de Recherche Azure et de Microsoft Bot Framework.
> 
> Par souci de simplicité, l’exemple ci-dessus montre un index de recherche qui est créé à l’aide du portail Azure. 
> Les index peuvent également être créés par programmation.

## <a name="qna-maker"></a>QnA Maker

Certains bots de connaissances peuvent être simplement destinés à répondre aux questions fréquentes (FAQ). 
<a href="https://www.microsoft.com/cognitive-services/en-us/qnamaker" target="_blank">QnA Maker</a> est un outil puissant conçu spécifiquement pour ce cas d’usage. QnA Maker a la capacité intégrée de capturer des questions et réponses à partir d’un site de FAQ existant, et vous permet également de configurer manuellement votre propre liste de questions et réponses. QnA Maker a des capacités de traitement automatique du langage naturel, ce qui lui permet même de fournir des réponses aux questions qui sont formulées légèrement différemment que prévu. Toutefois, il n’a pas de capacités de compréhension linguistique sémantique. Il ne peut pas déterminer qu’un chiot est un type de chien, par exemple. 

À l’aide de l’interface web de QnA Maker, vous pouvez configurer une base de connaissances avec trois paires de question et réponse : 

![Structure du dialogue](~/media/bot-service-design-pattern-knowledge-base/KnowledgeBaseConfig.png)

Ensuite, vous pouvez la tester en posant une série de questions : 

![Structure du dialogue](~/media/bot-service-design-pattern-knowledge-base/exampleQnAConvo.png)

Le bot répond correctement aux questions qui correspondent directement à celles qui ont été configurées dans la base de connaissances. Toutefois, il répond de façon incorrecte à la question « puis-je apporter mon thé ? », étant donné que cette question, de par sa structure, ressemble à la question « puis-je apporter ma vodka ? ». QnA Maker donne une réponse incorrecte car il ne comprend pas, par nature, la signification des mots. Il ne sait pas que « thé » est un type de boisson non alcoolisée. Ainsi, il répond : « l’alcool n’est pas autorisé ».

> [!TIP]
> Créez vos paires de question/réponse, puis testez et reformez votre bot en utilisant le bouton « Inspecter » sous la conversation afin de sélectionner une autre réponse pour chaque réponse incorrecte donnée. 

## <a name="luis"></a>LUIS

Certains bots de connaissances nécessitent des fonctionnalités de traitement automatique du langage naturel (TALN) pour analyser les messages d’un utilisateur afin de déterminer son intention. 
[Language Understanding (LUIS)](https://www.luis.ai) fournit un moyen rapide et efficace pour ajouter des fonctionnalités TALN aux bots. LUIS vous permet d’utiliser des modèles préconstruits existants à partir de Bing et Cortana chaque fois qu’ils répondent à vos besoins, ainsi que de créer vos propres modèles spécialisés. 

Quand vous utilisez de grands jeux de données, il n’est pas forcément possible de former un modèle TALN avec chaque variation d’une entité. Par exemple, dans un bot de lecture de musique, un utilisateur peut entrer le message « Jouer du reggae », « Jouer Bob Marley » ou « Jouer One Love ». Bien qu’un bot puisse mapper chacun de ces messages à l’intention « playMusic » (jouer de la musique), sans être formé avec chaque nom d’artiste, de genre et de chanson, un modèle TALN ne peut pas déterminer si l’entité est un genre, un artiste ou une chanson. En utilisant un modèle TALN pour identifier l’entité générique de type « musique », le bot peut rechercher cette entité dans son magasin de données et continuer à partir de là. 

## <a name="combining-search-qna-maker-andor-luis"></a>Combinaison de Recherche, QnA Maker et/ou LUIS

Recherche, QnA Maker et LUIS sont chacun des outils puissants en soi, mais vous pouvez également les combiner pour générer des bots de connaissances qui possèdent plusieurs de ces fonctionnalités.

### <a name="luis-and-search"></a>LUIS et Recherche

Dans l’exemple de bot de festivals de musique [décrit précédemment](#search), le bot guide la conversation en affichant des boutons qui représentent la formation. Toutefois, ce bot pourrait également intégrer la compréhension du langage naturel à l’aide de LUIS pour déterminer l’intention et des entités au sein des questions telles que « quel type de musique Romit Girdhar joue-t-il ? ». Le bot pourrait ensuite effectuer une recherche dans un index Recherche Azure à partir du nom de musicien. 
 
Il serait impossible de former le modèle avec chaque nom de musicien possible en raison du nombre élevé de valeurs potentielles, mais vous pourriez fournir suffisamment d’exemples représentatifs pour que LUIS identifie correctement l’entité à portée de main.  Par exemple, supposons que vous formez votre modèle en fournissant des exemples de musiciens : 

![Structure du dialogue](~/media/bot-service-design-pattern-knowledge-base/answerGenre.png)
![Structure du dialogue](~/media/bot-service-design-pattern-knowledge-base/answerGenreOneWord.png)

Quand vous testez ce modèle avec de nouveaux énoncés tels que « quel type de musique jouent les beatles ? », LUIS détermine correctement l’intention « answerGenre » (répondre un genre) et identifie l’entité « beatles ». Toutefois, si vous envoyez une question plus longue telle que « quel type de musique joue the devil makes three ? », LUIS identifie « devil » comme étant l’entité.

![Structure du dialogue](~/media/bot-service-design-pattern-knowledge-base/devilMakesThreeScore.png)

En formant le modèle avec des exemples d’entités qui sont représentatifs du jeu de données sous-jacent, vous pouvez augmenter la précision de la compréhension linguistique de votre bot. 

> [!TIP]
> En règle générale, il vaut mieux que le modèle se trompe en identifiant des mots en trop dans sa reconnaissance de l’entité, par exemple, « Jean Dupont s’il vous plait » à partir de l’énoncé « appelez Jean Dupont s’il vous plait », au lieu d’identifier trop peu de mots, par exemple, « Jean » à partir de l’énoncé « appelez Jean Dupont s’il vous plait ». L’index de recherche ignore les mots non pertinents tels que «s’il vous plait » dans l’expression « Jean Dupont s’il vous plait ». 

### <a name="luis-and-qna-maker"></a>LUIS et QnA Maker

Certains bots de connaissances peuvent utiliser QnA Maker pour répondre à des questions de base en association avec LUIS pour déterminer les intentions, extraire les entités et appeler des dialogues plus élaborés. Par exemple, considérez un simple bot d’assistance informatique. Ce bot peut utiliser QnA Maker pour répondre aux questions de base sur Windows ou Outlook, mais il peut également devoir faciliter des scénarios, tels que la réinitialisation de mot de passe, qui nécessitent la reconnaissance des intentions et une communication bidirectionnelle entre l’utilisateur et le bot. Un bot peut combiner LUIS et QnA Maker de plusieurs façons :

1. Appeler QnA Maker et LUIS en même temps et répondre à l’utilisateur à l’aide des informations du premier qui renvoie un score d’un seuil spécifique. 
2. Appeler LUIS d’abord, et si aucune intention ne répond à un score d’un seuil spécifique (dans ce cas l’intention « None » (aucune) est déclenchée), appeler QnA Maker. Vous pouvez également créer une intention LUIS pour QnA Maker, en alimentant votre modèle LUIS avec des exemples de question/réponse qui correspondent à « QnAIntent » (intention par rapport à une question/réponse). 
3. Appeler d’abord QnA Maker et si aucune réponse ne satisfait à un score de seuil spécifique, appeler LUIS. 

Le SDK Bot Builder prend en charge LUIS et QnA Maker. Cela vous permet de déclencher des dialogues ou de répondre automatiquement à des questions à l’aide de LUIS ou QnA Maker sans avoir à implémenter des appels personnalisés pour ces outils. Consultez le [didacticiel de l’outil Bot Builder Dispatch](https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-tutorial-dispatch?view=azure-bot-service-4.0) pour plus d’informations.

> [!TIP]
> Quand vous implémentez une combinaison de LUIS, QnA Maker et/ou Recherche Azure, testez les entrées avec chacun des outils afin de déterminer le score de seuil pour chacun de vos modèles. LUIS, QnA Maker et Recherche Azure génèrent chacun des scores en utilisant des critères de notation différents, ce qui empêche de comparer ces scores directement. En outre, LUIS et QnA Maker normalisent les scores. Un score donné peut être considéré comme « bon » dans un modèle LUIS, mais pas dans un autre. 

## <a name="sample-code"></a>Exemple de code

- Pour obtenir un exemple qui montre comment créer un bot de connaissances de base en utilisant le SDK Bot Builder pour .NET, consultez <a href="https://aka.ms/qna-with-appinsights" target="_blank">l’exemple de bot de connaissances</a> dans GitHub. 
<!-- TODO: Do not have a current bot sample to work with this
- For a sample that shows how to create more complex knowledge bots using the Bot Builder SDK for .NET, see the <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/demo-Search" target="_blank">Search-powered Bots sample</a> in GitHub.
-->

[qnamakerTemplate]: https://docs.botframework.com/en-us/azure-bot-service/templates/qnamaker/#navtitle
