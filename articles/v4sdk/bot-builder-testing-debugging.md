---
title: Recommandations en matière de test et de débogage | Microsoft Docs
description: Découvrez comment tester et déboguer votre robot.
keywords: principes de test, éléments fictifs, faq, niveaux de test
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 2/26/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: e0fbf2fe505866ec28385b2be3b9fcd7988be890
ms.sourcegitcommit: cf3786c6e092adec5409d852849927dc1428e8a2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 03/01/2019
ms.locfileid: "57224927"
---
# <a name="testing-and-debugging-guidelines"></a>Recommandations en matière de test et de débogage

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Les bots sont des applications complexes composées de nombreux éléments différents qui fonctionnent ensemble. Comme pour toutes les applications complexes, des bogues intéressants ou des comportements inattendus peuvent se produire.

Il peut parfois s’avérer difficile de tester, puis de déboguer votre robot. Chaque développeur effectue cette tâche comme il le souhaite. Les conseils que nous vous présentons ci-dessous sont des recommandations d’utilisation qui s’appliquent à la plupart des robots.

## <a name="testing-your-bot"></a>Test de votre robot

Les recommandations ci-dessous se présentent sous trois **niveaux** différents.  Chaque niveau ajoute de la complexité et des fonctionnalités aux tests. Nous vous recommandons donc qu’un niveau vous convienne avant de passer au suivant. Vous pourrez ainsi isoler et corriger les problèmes de plus bas niveau avant d’ajouter de la complexité.

Les bonnes pratiques en matière de tests couvriront plusieurs approches, le cas échéant. Il peut s’agir de sécurité, d’intégration, d’URL mal formées, d’exploitation de validation, de codes d’état HTTP, de charges utiles JSON, de valeurs nulles, etc. Si votre robot traite des informations sensibles pour la confidentialité des utilisateurs, ce point est particulièrement important.

### <a name="level-1-use-mock-elements"></a>Niveau 1 : Utiliser des éléments fictifs

Le premier niveau de test consiste à s’assurer que chaque petit composant de votre application, ou de notre robot dans le cas présent, fonctionne exactement comme prévu. Pour ce faire, vous pouvez utiliser des éléments fictifs pour simuler tout ce que vous n’êtes pas en train de tester. À titre de référence, ce niveau peut généralement être assimilé à un test d’unité et d’intégration.

#### <a name="use-mock-elements-to-test-individual-sections"></a>Utiliser des éléments fictifs pour tester des sections individuelles

En simulant autant d’éléments que vous le souhaitez, vous pouvez mieux isoler la partie à tester. Les éléments fictifs possibles sont le stockage, l’adaptateur, l’intergiciel, le pipeline d’activités, les canaux et tout ce qui ne fait pas directement partie de votre robot. Il est également possible de simuler la suppression temporaire de certains éléments, par exemple un intergiciel non concerné par ce que vous testez dans votre robot, et ce, afin d’isoler chaque partie. Si vous testez votre intergiciel, vous pouvez cependant vouloir simuler votre robot à la place.

Les éléments fictifs peuvent prendre toute une série de formes, du remplacement d’un élément par un autre objet connu à la mise en place d’une fonctionnalité de type hello world. L’élément peut aussi être simplement retiré s’il n’est pas nécessaire ou être forcé de rester inactif. 

Ce niveau doit employer des méthodes et des fonctions individuelles au sein de votre robot. Il est possible de tester des méthodes individuelles au moyen de tests d’unité intégrés (recommandés) à l’aide de votre propre application de test ou suite de tests, ou encore manuellement dans votre IDE. 

#### <a name="use-mock-elements-to-test-larger-features"></a>Utiliser des éléments fictifs pour tester des fonctionnalités plus importantes

Une fois que le comportement de chaque méthode vous convient, utilisez ces éléments fictifs pour tester des fonctionnalités plus poussées dans votre robot. Voici une illustration de la façon dont quelques couches interagissent pour discuter avec l’utilisateur. 

Toute une série d’outils est fournie à cette fin. Par exemple, [Azure Bot Framework Emulator](https://github.com/Microsoft/BotFramework-Emulator) permet de communiquer avec votre robot à l’aide d’un canal émulé. L’utilisation de l’émulateur crée une situation plus complexe que les tests d’unité et d’intégration. Elle s’étend donc au niveau de test suivant.

### <a name="level-2-use-a-direct-line-client"></a>Niveau 2 : Utiliser un client Direct Line

Après avoir vérifié que votre robot semble fonctionner comme prévu, l’étape suivante consiste à le connecter à un canal. Pour ce faire, vous pouvez déployer votre bot sur un serveur intermédiaire et créer votre propre client DirectLine sur lequel votre bot peut se connecter.
<!--IBTODO [Direct Line client](bot-builder-howto-direct-line.md)-->

La création de votre propre client vous permet de définir le fonctionnement interne du canal et de tester de façon précise la réponse de votre robot à certains échanges d’activités. Une fois que la connexion à votre client est établie, exécutez vos tests pour configurer l’état de votre robot et vérifier vos fonctionnalités. Si votre robot utilise une fonctionnalité comme des données vocales, il est possible de la vérifier à l’aide de ces canaux.

L’utilisation de l’émulateur et de la discussion en ligne via le portail Azure permet de mieux comprendre le fonctionnement de votre robot lorsqu’il interagit avec différents canaux.

### <a name="level-3-channel-tests"></a>Niveau 3 : Tests de canaux

Une fois que vous êtes certain du fonctionnement indépendant de votre robot, il est important de voir comment il se comporte avec les différents canaux sur lesquels il sera accessible. 

Il existe de nombreuses façons de procéder, de l’utilisation séparée de différents canaux et navigateurs à l’utilisation d’un outil tiers comme [Selenium](https://docs.seleniumhq.org/) permettant d’interagir par le biais d’un canal et de récupérer les réponses de votre robot.

### <a name="other-testing"></a>Autres tests

Il est possible d’effectuer différents types de tests avec les niveaux ci-dessus ou selon différentes approches, par exemple les tests de stress, les tests de performance ou le profilage de l’activité du robot. Visual Studio fournit des méthodes permettant de les effectuer localement ainsi qu’une [suite d’outils](https://azure.microsoft.com/en-us/solutions/dev-test/) pour tester votre application. Le [portail Azure](https://portal.azure.com) fournit des informations sur le fonctionnement de votre robot.

## <a name="debugging"></a>Débogage

Le débogage de votre robot fonctionne de la même manière que les autres applications multithreads en permettant de définir des points d’arrêt ou d’utiliser des fonctions telles que la fenêtre immédiate. 

Les robots suivent un paradigme de programmation par événement qui peut être difficile à rationaliser si vous ne le connaissez pas bien. Des bogues inattendus peuvent se produire lorsque votre robot est sans état, en multithreads et qu’il traite des appels async/await. Bien que le débogage de votre robot fonctionne de la même manière que les autres applications multithreads, nous vous proposerons quelques recommandations, outils et ressources pour vous aider.

### <a name="understanding-bot-activities-with-the-emulator"></a>Comprendre les activités du bot à l’aide de l’émulateur

Votre robot traite différents types [d’activités](bot-builder-basics.md#the-activity-processing-stack)en plus de l’activité normale de _message_. L’[émulateur](../bot-service-debug-emulator.md) vous permettra de savoir quelles sont ces activités, quand elles se produisent et quelles sont les informations qu’elles contiennent. Le fait de comprendre ces activités vous aidera à coder votre robot de façon efficace et vous permettra de vérifier que les activités envoyées et reçues par votre robot correspondent bien à ce qu’elles devraient être.

### <a name="saving-and-retrieving-user-interactions-with-transcripts"></a>Enregistrer et récupérer des interactions utilisateur avec les transcriptions

Le stockage de transcriptions blob Azure fournit une ressource spécialisée dans laquelle vous pouvez à la fois [stocker et récupérer des transcriptions](bot-builder-howto-v4-storage.md) qui contiennent des interactions entre vos utilisateurs et votre bot.  

De plus, une fois que les interactions d’entrée de l’utilisateur ont été stockées, vous pouvez utiliser l’_explorateur du stockage_ Azure pour afficher manuellement les données contenues dans les transcriptions qui sont stockées dans votre magasin d’objets blob. L’exemple suivant ouvre l’_explorateur du stockage_ à partir des paramètres pour _mynewtestblobstorage_. Pour ouvrir une entrée utilisateur enregistrée, sélectionnez :    Conteneur d’objets Blob > ChannelId > TranscriptId > ConversationId

![Examine_stored_transcript_text](./media/examine_transcript_text_in_azure.png)

Cette opération ouvre l’entrée de la conversation utilisateur stockée au format JSON. L’entrée de l’utilisateur est conservée avec la clé _texte_.

### <a name="how-middleware-works"></a>Fonctionnement du middleware

L’[intergiciel](bot-builder-concept-middleware.md) peut ne pas être intuitif lors de la première tentative d’utilisation, en particulier en ce qui concerne la poursuite ou le court-circuitage de l’exécution. L’intergiciel peut s’exécuter en début ou en fin de tour en appelant le délégué `next()` qui détermine le moment où l’exécution est transmise à la logique du robot. 

Si vous utilisez plusieurs intergiciels, le délégué peut transmettre l’exécution à un autre intergiciel si votre pipeline est orienté de la sorte. Des détails sur [le pipeline de l’intergiciel du robot](bot-builder-concept-middleware.md#the-bot-middleware-pipeline) permettent de clarifier cette idée.

Lorsque le délégué `next()` n’est pas appelé, il s’agit d’un [routage en court-circuit ](bot-builder-concept-middleware.md#short-circuiting). Cette situation se produit lorsque l’intergiciel répond à l’activité en cours et détermine qu’il n’est pas nécessaire de transmettre l’exécution. 

Le fait de comprendre quand et pourquoi l’intergiciel effectue des courts-circuits permet de savoir quelle partie de l’intergiciel doit se trouver en premier dans votre pipeline. De plus, il est particulièrement important de savoir à quoi s’attendre avec l’intergiciel intégré fourni par le kit de développement logiciel ou d’autres développeurs. Certaines personnes trouvent utile de créer leur propre intergiciel afin de gagner en pratique avant de se lancer avec l’intergiciel intégré.

<!-- Snip: QnA was once implemented as middleware.
For example [QnA maker](bot-builder-howto-qna.md) is designed to handle certain interactions and short-circuit the pipeline when it does, which can be confusing when first learning how to use it.
-->

### <a name="understanding-state"></a>Comprendre l’état

La conservation de la trace de l’état est une partie importante de votre robot, en particulier pour les tâches complexes. En règle générale, la bonne pratique consiste à traiter les activités le plus rapidement possible et à laisser le traitement se terminer afin de conserver l’état. Les activités peuvent être envoyées à votre robot presque en même temps, ce qui peut provoquer des bogues très déroutants en raison de l’architecture asynchrone.

Plus important encore, assurez-vous que la conservation de l’état correspond à vos attentes. Selon l’endroit où se trouve votre état conservé, les émulateurs de stockage pour [Cosmos DB](https://docs.microsoft.com/en-us/azure/cosmos-db/local-emulator)et [Azure Table Storage](https://docs.microsoft.com/en-us/azure/storage/common/storage-use-emulator) vous permettent de le vérifier avant d’utiliser le stockage en production.

### <a name="how-to-use-activity-handlers"></a>Comment utiliser les gestionnaires d’activités

Les gestionnaires d’activités peuvent ajouter une couche de complexité supplémentaire, notamment parce que chaque activité s’exécute sur un thread indépendant (ou web worker selon le langage utilisé). Selon les activités de vos gestionnaires, des problèmes peuvent se produire si l’état actuel ne correspond pas à l’état attendu.

L’état intégré est enregistré en fin de tour, mais toutes les activités générées par ce tour sont exécutées indépendamment de son pipeline. Cela ne nous affecte généralement pas, mais si un gestionnaire d’activité change d’état, nous avons besoin de l’état enregistré pour maîtriser cette modification. Dans ce cas, le pipeline du tour peut rester en attente sur l’activité pour que le traitement se termine avant la fin de façon à s’assurer qu’il enregistre le bon état pour ce tour.

La méthode _send activity_ et ses gestionnaires posent un problème unique. Le simple fait d’appeler _send activity_ à partir d’un gestionnaire _on send activities_ provoque une duplication infinie des threads. Il est possible de contourner ce problème, notamment en ajoutant des messages aux données sortantes ou en les écrivant dans un autre emplacement, tel que la console ou un fichier, afin d’éviter que votre bot ne plante.

## <a name="additional-resources"></a>Ressources supplémentaires

* [Débogage dans Visual Studio](https://docs.microsoft.com/en-us/visualstudio/debugger/index)
* [Débogage, traçage et profilage](https://docs.microsoft.com/en-us/dotnet/framework/debug-trace-profile/) de l’infrastructure du robot
* Utilisez l’attribut [ConditionalAttribute](https://docs.microsoft.com/en-us/dotnet/api/system.diagnostics.conditionalattribute?view=netcore-2.0) pour les méthodes que vous ne souhaitez pas inclure dans le code de production
* Utilisez des outils comme [Fiddler](https://www.telerik.com/fiddler) pour afficher le trafic réseau
* [Référentiel d’outils de robots](https://github.com/Microsoft/botbuilder-tools)
* Les infrastructures peuvent faciliter les tests, par exemple [Moq](https://github.com/moq/moq4)
* [Résoudre les problèmes généraux](../bot-service-troubleshoot-bot-configuration.md) ainsi que les autres articles de résolution des problèmes de cette section.
