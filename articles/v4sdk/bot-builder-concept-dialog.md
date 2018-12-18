---
title: Dialogues dans le SDK Bot Builder | Microsoft Docs
description: Découvrez ce qu’est un dialogue et comment il fonctionne dans le SDK Bot Builder.
keywords: flux de conversation, invite, état de dialogue, reconnaître l’intention, tour unique, plusieurs tours, conversation de bot, dialogues, invites, cascades, ensemble de dialogues
author: johnataylor
ms.author: johtaylo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 11/28/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: a1187efd3280d9ec2d74af29d1c013e916b79e5b
ms.sourcegitcommit: 91156d0866316eda8d68454a0c4cd74be5060144
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2018
ms.locfileid: "53010604"
---
# <a name="dialogs-library"></a>Bibliothèque des dialogues

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Les *dialogues* constituent un concept central dans le SDK. Ils offrent un moyen utile de gérer une conversation avec l’utilisateur. Les dialogues sont des structures dans votre bot qui agissent comme des fonctions dans le programme de votre bot ; chaque dialogue est conçu pour effectuer une tâche spécifique, dans un ordre spécifique. Vous pouvez spécifier l’ordre des dialogues individuels pour guider la conversation, puis les appeler de différentes façons : parfois, en réponse à un utilisateur, parfois en réponse à des stimuli extérieurs ou à partir d’autres dialogues.

La bibliothèque de dialogues propose quelques fonctionnalités intégrées, comme les *invites* et les *dialogues en cascade* pour rendre la conversation de votre bot plus facile à gérer. Les [invites](#prompts) sont utilisées pour demander différents types d’informations, comme du texte, un nombre ou une date. Les [dialogues en cascade](#waterfall-dialogs) peuvent combiner plusieurs étapes en une séquence, ce qui permet à votre bot de suivre facilement cette séquence prédéfinie pour transmettre des informations à l’étape suivante.

<!-- When we have samples for building your own, add links and one liner about them -->

## <a name="dialogs-and-their-pieces"></a>Les dialogues et leurs éléments

La bibliothèque de dialogues comporte quelques autres éléments pour rendre les dialogues plus utiles. En plus des différents [types de dialogues](#dialog-types) décrits ci-dessous, la bibliothèque contient l’idée d’un *jeu de dialogues*, le *contexte du dialogue* et le *résultat du dialogue* .

Les *jeux de dialogues* sont, en termes simples, une collection de dialogues. Il peut s’agir d’invites, de dialogues en cascade ou de [dialogues composants](#component-dialog). Tous ces éléments sont les implémentations d’un dialogue, et tous sont ajoutés au jeu de dialogues avec un ID de chaîne spécifique. Quand votre bot souhaite démarrer un certain dialogue ou une certaine invite au sein du jeu de dialogues, il utilise cet ID de chaîne pour spécifier quel dialogue utiliser.

Le *contexte du dialogue* contient des informations sur le dialogue. Il sert à interagir avec un jeu de dialogues depuis le gestionnaire de tour de votre bot. Le contexte du dialogue inclut le contexte du tour actuel, le dialogue parent et l’[état du dialogue](#dialog-state), ce qui offre une méthode permettant de conserver les informations au sein du dialogue. Le contexte du dialogue vous permet de démarrer un dialogue avec son ID de chaîne ou de continuer le dialogue actuel (par exemple, un dialogue en cascade qui comporte plusieurs étapes).

Quand un dialogue se termine, il peut retourner un *résultat du dialogue* avec certaines informations obtenues à partir du dialogue. Ce résultat est retourné pour permettre à la méthode d’appel de voir ce qui s’est passé au cours du dialogue et d’enregistrer les informations à un emplacement persistant, si vous le souhaitez.

## <a name="dialog-state"></a>État de dialogue

Les dialogues constituent une approche pour l’implémentation de conversation à plusieurs tours. En ce sens, il s’agit d’une fonctionnalité du Kit de développement logiciel (SDK) qui repose sur un état persistant à plusieurs tours. Sans aucun état dans les dialogues, votre bot ne saurait pas où il se trouve dans le jeu de dialogues ni quelles informations il a déjà recueillies.

Un bot basé sur des dialogues comporte généralement une collection d’ensemble de boîtes de dialogue en tant que variable de membre dans son implémentation de bot. Cet ensemble de boîtes de dialogue est créé avec un descripteur vers un objet nommé accesseur qui fournit un accès à l’état persistant. Pour plus d’informations sur l’état au sein des bots, consultez [gestion de l’état](bot-builder-concept-state.md).

Au sein de son gestionnaire de tour, le bot initialise le sous-système de dialogue en appelant *create context* sur le jeu de dialogues, qui retourne le *contexte du dialogue*. Ce contexte de boîte de dialogue contient les informations nécessaires à la boîte de dialogue.

La création d’un contexte de boîte de dialogue requiert que l’état, accessible avec l’accesseur fourni lors de la création de l’ensemble de boîtes de dialogue. Avec cet accesseur, le jeu de dialogues peut obtenir l’état du dialogue approprié. Vous trouverez plus d’informations sur les accesseurs d’état dans [Enregistrer les données de la conversation et de l’utilisateur](bot-builder-howto-v4-state.md).

## <a name="dialog-types"></a>Types de dialogues

Les dialogues sont fournis dans différents types : invites, dialogues en cascade et dialogues composants, comme indiqué dans cette hiérarchie de classes.

![classes de dialogues](media/bot-builder-dialog-classes.png)

### <a name="prompts"></a>Invites

Les invites, au sein de la bibliothèque de dialogues, offrent un moyen simple de demander des informations à l’utilisateur et d’évaluer sa réponse. Par exemple, dans le cas d’une *invite numérique*, vous spécifiez la question posée ou les informations demandées, puis l’invite vérifie automatiquement si la réponse reçue est bien un nombre valide. Si c’est le cas, la conversation peut continuer ; sinon, l’utilisateur est de nouveau invité à donner une réponse valide.

Dans les coulisses, les invites constituent une boîte de dialogue en deux étapes. Tout d’abord, l’invite demande une entrée. Ensuite, elle retourne la valeur valide ou redémarre depuis le début avec une nouvelle invite.

Les invites comportent des *options* qui sont proposées lorsqu’elles sont appelées. Ces options vous permettent de spécifier le texte des invites, les invites de nouvelle tentative en cas d’échec de validation et les possibilités de réponse.

De plus, vous pouvez choisir d’ajouter une validation personnalisée pour votre invite au moment où vous la créez. Par exemple, supposons que nous voulions obtenir une taille, à l’aide d’une invite numérique, devant être supérieure à 2 mais inférieure à 12. L’invite commence par vérifier si elle a reçu un nombre valide, puis elle exécute la validation personnalisée si elle existe. Si la validation personnalisée échoue, l’utilisateur reçoit une nouvelle invite comme ci-dessus.

À la fin de l’invite, la valeur demandée obtenue est explicitement retournée. Lorsque cette valeur est retournée, nous pouvons être certains qu’elle est passé par la validation intégrée et toute autre éventuelle validation personnalisée.

Pour des exemples d’utilisation des diverses invites, intéressez-vous à la manière d’utiliser la [bibliothèque de dialogues pour recueillir une entrée utilisateur](bot-builder-prompts.md).

#### <a name="prompt-types"></a>Types d’invites

Dans les coulisses, les invites constituent une boîte de dialogue en deux étapes. Tout d’abord, l’invite demande une entrée. Ensuite, elle retourne la valeur valide, ou redémarre depuis le début avec une nouvelle invite. La bibliothèque de dialogues propose diverses invites de base, chacune étant utilisée pour recueillir un type de réponse différent. Les invites de base peuvent interpréter une entrée de langage naturel, comme « dix » ou « une dizaine » pour un nombre, ou « demain » ou un « vendredi à 10 h » pour une date-heure.

| Prompt | Description | Retours |
|:----|:----|:----|
| _Invite de pièce jointe_ | Demande une ou plusieurs pièces jointes, un document ou une image par exemple. | Une collection d’objets _Pièce jointe_. |
| _Invite de choix_ | Demande un choix à partir d’un ensemble d’options. | Un objet de _choix trouvé_. |
| _Invite de confirmation_ | Demande une confirmation. | Valeur booléenne. |
| _Invite de date et d’heure_ | Demande une date et une heure. | Une collection d’objets de _résolution de date / heure_. |
| _Invite de nombre_ | Demande un nombre. | Une valeur numérique. |
| _Invite de texte_ | Demande une saisie de texte générale. | Une chaîne. |

Pour inviter un utilisateur à saisir une entrée, définissez une invite à l’aide de l’une des classes intégrées, _text prompt_, par exemple, et ajoutez-la à votre ensemble de dialogues. Les invites ont des ID fixes devant être uniques au sein d’un ensemble de boîte de dialogue. Vous pouvez avoir un validateur personnalisé pour chaque invite, et pour certaines d’entre elles, vous pouvez spécifier des _paramètres régionaux par défaut_. 

#### <a name="prompt-locale"></a>Paramètres régionaux d’invite

Les paramètres régionaux sont utilisés pour déterminer le comportement spécifique à la langue des invites **choix**, **confirmer**, **date-heure**, et **nombre**. Pour toute entrée donnée de l’utilisateur, si le canal fournit une propriété _paramètres régionaux_ dans le message de l’utilisateur, elle est utilisée. Sinon, les _paramètres régionaux par défaut_ de l’invite sont utilisés s’ils sont définis, soit en les fournissant lors de l’appel du constructeur de l’invite ou bien en les définissant ultérieurement. Si aucun paramètre n’est fourni, l’anglais (« en-us ») est utilisé. Remarque : Les paramètres régionaux sont des codes ISO 639 de 2, 3 ou 4 caractères qui représentent une langue ou une famille de langues.

### <a name="waterfall-dialogs"></a>Dialogues en cascade

Un dialogue en cascade est une implémentation spécifique d’un dialogue, qui est couramment utilisée pour recueillir des informations auprès de l’utilisateur ou pour le guider dans une série de tâches. Chaque étape de la conversation est implémentée comme une fonction asynchrone qui prend un paramètre de *contexte d’étape en cascade* (`step`). À chaque étape, le bot [invite l’utilisateur à effectuer une entrée](bot-builder-prompts.md) (ou peut démarrer un dialogue enfant, mais il s’agit souvent d’une invite), attend une réponse, puis transmet le résultat à l’étape suivante. Le résultat de la première fonction est transmis en tant qu’argument à la fonction suivante, et ainsi de suite.

Le diagramme suivant représente une séquence d’étapes en cascade et les opérations de pile qui se produisent. L’utilisation de la pile de dialogues est décrite ci-dessous dans la section sur l’[utilisation des dialogues](#using-dialogs).

![Concept de dialogue](media/bot-builder-dialog-concept.png)

Dans le cadre des étapes en cascade, le contexte du dialogue en cascade est stocké dans son *contexte d’étape en cascade*. Celui-ci est similaire au contexte du dialogue car il fournit l’accès au contexte et à l’état du tour actuel. Utilisez l’objet de contexte d’étape en cascade pour interagir avec un ensemble de dialogues défini dans une étape en cascade.

Vous pouvez gérer une valeur de retour à partir d’un dialogue, soit dans le cadre d’une étape en cascade dans un dialogue, soit depuis le gestionnaire de tour de votre bot, même si vous avez généralement besoin de vérifier uniquement l’état du résultat du tour à partir de la logique de tour de votre bot.
Dans une étape en cascade, le dialogue fournit la valeur renvoyée dans la propriété _résultat_ du contexte de l’étape en cascade.

#### <a name="waterfall-step-context-properties"></a>Propriétés du contexte d’étape en cascade

Le contexte d’étape en cascade contient les éléments suivants :

* *Options* : contient les informations d’entrée pour le dialogue.
* *Valeurs* : contient les informations que vous pouvez ajouter au contexte, transmises aux étapes suivantes.
* *Résultat* : contient le résultat de l’étape précédente.

De plus, la méthode *next* passe à l’étape suivante du dialogue en cascade au sein du même tour, ce qui permet à votre bot d’ignorer une certaine étape si nécessaire.

### <a name="component-dialog"></a>Dialogue composant

Parfois, vous avez besoin d’écrire un dialogue réutilisable afin de l’exploiter dans différents scénarios, par exemple un dialogue d’adresse qui invite l’utilisateur à fournir des valeurs de rue, ville et code postal.

Le *dialogue composant* propose une stratégie qui permet de créer des dialogues indépendants pour gérer des scénarios spécifiques, en divisant un jeu de dialogues en éléments plus petits plus faciles à gérer. Chacun de ces éléments a son propre jeu de dialogues, ce qui permet d’éviter toute collision de nom avec le jeu de dialogues qui le contient. Consultez le [guide pratique sur les dialogues composants](bot-builder-compositcontrol.md) pour plus d’informations.

## <a name="using-dialogs"></a>Utilisation de dialogues

Vous pouvez utiliser le contexte du dialogue pour démarrer, continuer, remplacer ou terminer un dialogue. Vous pouvez également annuler tous les dialogues dans la pile de dialogues.

Les dialogues peuvent être considérés comme une pile programmatique, que nous appelons *pile de dialogues*, avec le gestionnaire de tour comme dirigeant et comme élément de secours si la pile est vide. L’élément le plus haut dans cette pile est considéré comme le *dialogue actif*. Le contexte du dialogue dirige toutes les entrées vers ce dialogue actif.

Dès qu’un dialogue commence, il est placé sur le dessus de la pile et devient le dialogue actif. Il reste dialogue actif jusqu’à ce qu’il se termine, qu’il soit supprimé par la méthode [replace dialog](#repeating-a-dialog) ou qu’un autre dialogue soit placé sur le dessus de la pile (par le gestionnaire de tour ou le dialogue lui-même) et devienne dialogue actif. Lorsque ce nouveau dialogue se termine, il est sorti de la pile (dépilé) et le dialogue situé juste en dessous redevient dialogue actif. Ainsi, il est possible de créer des [branches et des boucles](#looping-and-branching), comme décrit ci-dessous.

### <a name="create-the-dialog-context"></a>Créer le contexte du dialogue

Pour créer le contexte de votre dialogue, appelez la méthode *create context* de votre jeu de dialogues. Cette méthode obtient la propriété *dialog state* du jeu de dialogues et l’utilise pour créer le contexte du dialogue. Le contexte du dialogue est ensuite utilisé pour démarrer, continuer ou contrôler les dialogues inclus dans le jeu.

Le jeu de dialogues oblige à utiliser un *accesseur de propriété d’état* pour accéder à l’état du dialogue. L’accesseur est créé et utilisé de la même façon que les autres accesseurs d’état, mais il est créé comme sa propre propriété basée sur l’état de la conversation. Pour plus d’informations sur la gestion de l’état, consultez la [rubrique sur la gestion de l’état](bot-builder-concept-state.md). L’utilisation de l’état du dialogue est présentée dans le guide pratique sur le [flux de conversation séquentiel](bot-builder-dialog-manage-conversation-flow.md).

### <a name="to-start-a-dialog"></a>Pour démarrer un dialogue

Pour démarrer un dialogue, transmettez l’*ID du dialogue* à démarrer à la méthode *begin dialog*, *prompt* ou *replace dialog* du contexte du dialogue. La méthode begin dialog envoie (push) le dialogue en haut de la pile, tandis que la méthode replace dialog dépile le dialogue actuel et envoie le dialogue de remplacement dans la pile.

### <a name="to-continue-a-dialog"></a>Pour continuer un dialogue

Pour continuer un dialogue, appelez la méthode *continue dialog*. La méthode continue permet toujours de continuer le dialogue le plus haut dans la pile (dialogue actif), s’il existe. Si le dialogue continué se termine, le contrôle est transmis au contexte parent qui continue au sein du même tour.

### <a name="to-end-a-dialog"></a>Pour terminer un dialogue

La méthode *end dialog* met fin à un dialog en le sortant de la pile et retourne un résultat facultatif dans le contexte parent (par exemple, le dialogue qui l’a appelée ou le gestionnaire de tour du bot). Cette dernière est souvent appelée à partir du dialogue pour mettre fin à son instance actuelle.

Vous pouvez appeler la méthode end dialog depuis n’importe quel emplacement où vous avez un contexte de dialogue, mais pour le bot, elle aura été appelée à partir du dialogue actif actuel.

> [!TIP]
> Il est recommandé d’appeler explicitement la méthode *end dialog* à la fin du dialogue.

### <a name="to-clear-all-dialogs"></a>Pour effacer tous les dialogues

Si vous souhaitez dépiler tous les dialogues, vous pouvez effacer la pile de dialogues en appelant la méthode *cancel all dialogs* du contexte de dialogue.

### <a name="repeating-a-dialog"></a>Répéter un dialogue

Pour répéter un dialogue, utilisez la méthode de *remplacement de dialogue*. La méthode *replace dialog* du contexte de dialogue fait sortir le dialogue actif actuel de la pile (sans y mettre fin normalement), puis envoie le dialogue de remplacement en haut de la pile et démarre ce dialogue. Il s’agit là d’un excellent moyen de traiter des [interactions complexes](~/v4sdk/bot-builder-dialog-manage-complex-conversation-flow.md) et de gérer les menus. Vous pouvez utiliser cette méthode pour créer une boucle en remplaçant un dialogue par lui-même.

> [!NOTE]
> Si vous avez besoin de conserver l’état interne du dialogue actuel, vous devez transmettre les informations à la nouvelle instance du dialogue lors de l’appel à la méthode *replace dialog*. Ensuite, vous devez initialiser le dialogue de manière appropriée. Les options transférées dans le nouveau dialogue sont accessibles via la propriété *options* du contexte de n’importe quelle étape du dialogue.

### <a name="branch-a-conversation"></a>Créer une branche de conversation

Le contexte du dialogue maintient la pile de dialogue et suit l’étape suivante de chaque dialogue sur la pile. Sa méthode *begin dialog* crée un enfant et envoie un dialogue par push sur le dessus de la pile, et sa méthode *end dialog* fait sortir de la pile le dialogue du dessus. La méthode *end dialog* est généralement appelée dans la boîte de dialogue qui se termine.

Un dialogue peut démarrer un nouveau dialogue au sein du même ensemble en appelant la méthode de *démarrage de dialogue* du contexte et en fournissant l’ID du nouveau dialogue. Le nouveau dialogue devient alors le dialogue actuellement actif. Le dialogue original est toujours sur la pile, mais les appels de la méthode de *poursuite du dialogue* du contexte de dialogue ne sont envoyés qu’au dialogue du dessus de la pile, le *dialogue actif*. Lorsqu’un dialogue est sorti de la pile, le contexte du dialogue passe à la prochaine étape sur la cascade sur la pile, là où il a laissé le dialogue original.

Ainsi, vous pouvez créer une branche au sein de votre flux de conversation en incluant une étape dans un dialogue. Cette étape permet de choisir sous condition un dialogue à démarrer parmi l’ensemble des dialogues disponibles.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Utiliser une bibliothèque de dialogues pour collecter des entrées utilisateur](bot-builder-prompts.md)
