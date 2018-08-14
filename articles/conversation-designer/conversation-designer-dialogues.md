---
title: Définir un dialogue en tant qu’action Do | Microsoft Docs
description: Découvrez comment configurer un dialogue en tant qu’action Do.
author: vkannan
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ROBOTS: NoIndex, NoFollow
ms.openlocfilehash: d59df20821a7f63eb9ee5dea365597b1af839f75
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39299049"
---
# <a name="define-a-dialogue-as-a-do-action"></a>Définir un dialogue en tant qu’action Do
> [!IMPORTANT]
> Conversation Designer n’est pas encore accessible à tous les clients. Des informations supplémentaires sur la disponibilité de Conversation Designer seront communiquées plus tard cette année.

Un dialogue couvre le modèle conversationnel pour une tâche spécifique. Par exemple, un robot de café qui aide les utilisateurs à réserver une table pourrait définir une tâche nommée « Réserver une table ». L’action **Do** serait liée à un dialogue modélisant le flux de conversation entre le robot et un utilisateur. 

Les dialogues sont particulièrement utiles quand un robot s’engage dans une conversation avec un utilisateur pour aider celui-ci à accomplir une tâche.  Les utilisateurs expriment rarement toutes les valeurs requises pour accomplir une tâche dans un énoncé unique. 

La réservation d’une table nécessite des préférences d’emplacement, de nombre de convives, de date et d’heure. Un robot qui réserve des tables doit être capable de comprendre et traiter les diverses phrases que l’utilisateur pourrait prononcer. 

- *Réserver une table* : dans ce cas, aucune des entités requises n’a été capturée. Le robot doit à présent engager une conversation avec l’utilisateur.
- *Réserver une table pour 19 heures ce samedi* : dans ce cas, les préférences de date et d’heure sont spécifiées, mais le robot doit encore recueillir l’emplacement et le nombre de convives souhaités.

En cours de conversation, certaines entités peuvent changer en fonction de l’entrée de l’utilisateur. Par exemple, si l’utilisateur dit « Réserver une table à Toulouse pour ce dimanche à 19 heures », alors que l’emplacement à Toulouse ferme à 18 heures le dimanche, le dialogue du robot doit traiter de telles demandes non valides. 

Conversation Designer fournit un concepteur de dialogue utilisant la fonction glisser-déplacer pour vous aider à visualiser votre flux de conversation. Le concepteur de dialogue offre sept *états de dialogue* que vous pouvez utiliser pour modéliser votre flux de conversation.

## <a name="dialogue-states"></a>États de dialogue

Un dialogue est constitué d’états de conversation. Le dialogue proprement dit est modélisé comme un flux structuré et orienté, qui fournit au runtime de conversation une structure déterminant la manière d’exécuter le flux de conversation.

Les dialogues sont fournis avec de nombreux états intégrés que vous pouvez utiliser. Les états intégrés pris en charge sont les suivants :

- [**Start**](#start-state) : représente l’état de départ pour un flux de conversation. Tous les dialogues doivent avoir au moins un état Start défini.
- [**Return**](#return-state) : représente l’accomplissement du flux de conversation spécifique. Étant donné que les flux de conversation sont composables, l’état Return indique au runtime de conversation de retourner l’exécution à tout appelant possible de ce dialogue.
- [**Decision**](#decision-state) : représente un point de ramification dans le flux de conversation.
- [**Process**](#process-state) : représente un état dans lequel votre robot s’exécute une logique métier.
- [**Prompt**](#prompt-state) : représente un état dans lequel vous pouvez inviter l’utilisateur à effectuer une entrée. 
- [**Feedback**](#feedback-state) : représente un état dans lequel vous pouvez envoyer un commentaire ou une confirmation à l’utilisateur. Par exemple, un dialogue confirmant que la réservation a été effectuée.
- [**Module**](#module-state) : représente un appel à un autre dialogue. Dans la mesure où les flux de dialogue sont composables par défaut, cet état peut appeler un dialogue partagé ou un autre dialogue défini sous cette tâche.

Chaque état de conversation est connecté à un autre à l’aide de connecteurs de dialogue dans le concepteur de dialogue.

Chaque état de dialogue est associé à un éditeur d’état utilisé pour spécifier les propriétés de cet état, dont les noms de fonction de rappel pour les scripts personnalisés. **Éditeur d’état** situé comme un volet de redimensionnement au bas du port d’affichage principal du concepteur de dialogue. Pour afficher l’éditeur, double-cliquez sur un état dans le concepteur de dialogue. Un **éditeur d’état** affiche alors les propriétés pour cet état.
<!-- TODO: insert screenshot of the wrench in horizontal menu -->

Les sous-sections suivantes fournissent des détails supplémentaires sur chacun de ces états de dialogue intégrés.

### <a name="start-state"></a>État Start

L’état Start indique le point de départ d’un dialogue. La valeur obligatoire est le **nom** de l’état. Par défaut, le nom est « Start », mais vous pouvez le modifier pour renommer l’état.

### <a name="return-state"></a>État Return

L’état Return représente l’accomplissement de la branche spécifique du flux de conversation. Les dialogues étant composables, l’état donne également pour instruction au runtime de conversation de retourner l’exécution au dialogue appelant. La valeur obligatoire est le **nom** de l’état. Par défaut, le nom est « Return », mais vous pouvez le modifier pour renommer l’état.

### <a name="decision-state"></a>État Decision

Un état Decision représente une ramification dans le flux de conversation. Vous pouvez écrire un script personnalisé pour évaluer la ramification à suivre. En fonction de l’entrée de l’utilisateur et de la logique métier, le script retourne l’une des valeurs de transition possibles. Chaque valeur de transition invite le runtime de conversation à exécuter une ramification différente du dialogue.

Propriétés requises pour l’état Decision :
- **Name** : nom unique pour l’état Decision.
- **Code to execute on run** : nom de la fonction de rappel qui implémente votre logique métier pour déterminer la ramification de la conversation à prendre. 

#### <a name="example-code-for-decision-state"></a>Exemple de code pour l’état Decision

L’exemple suivant de fonction de rappel retourne une décision qui indique au runtime de conversation la ramification à exécuter.

```javascript
module.exports.fnDecisionState = function(context) {
    var a = context.taskEntities['a'];
    if (a[0].value === '0') {
        return 'yes';
    }
    else if (a[0].value === '1') {
        return 'no';
    }
}
```

### <a name="process-state"></a>État Process

L’état Process représente un point dans le dialogue où le robot fait avancer la conversation ou tente d’effectuer l’action finale d’accomplissement de la tâche. 

Propriétés requises pour l’état Process :
- **Name** : nom unique pour l’état Process.
- **Code to execute on run** : nom de la fonction de rappel qui implémente votre logique métier.

#### <a name="example-code-for-process-state"></a>Exemple de code pour l’état Process

L’exemple suivant de fonction de rappel obtient la météo et retourne les informations météorologiques à l’utilisateur.

```javascript
module.exports.fnGetWeather = function(context) {
    var options =  {
        host: 'mock',
        path: '/get?a=b',
        method: 'get'
    };
    return http.request(options).then(function(response) {
        context.contextEntities['x'].value= response.statusCode;
        var jsonBody = JSON.parse(response.body);
          // understand response
        if (response.statusCode != "200") {
            // error
        }
    });
}
```

### <a name="prompt-state"></a>État Prompt

L’état Prompt demande à l’utilisateur un élément d’information spécifique. Les états Prompt englobent un système de sous-dialogue. Il s’agit donc par définition d’états complexes. 

Dans un état Prompt, vous pouvez définir la réponse réelle à donner à l’utilisateur, et éventuellement inclure une carte adaptative. Vous pouvez ensuite spécifier un déclencheur pour analyser et comprendre la réponse de l’utilisateur. Ce déclencheur peut être LUIS ou un module de reconnaissance de code personnalisé utilisant une expression régulière.  

Si l’entrée de l’utilisateur n’est pas valide, le robot peut ré-inviter l’utilisateur à fournir les mêmes informations. Ce comportement peut également être défini dans l’éditeur d’état Prompt. 

#### <a name="prompting-the-user"></a>Invitation de l’utilisateur

La réponse à l’invite vous permet de spécifier le message à utiliser pour **inviter l’utilisateur** à effectuer une entrée. Par exemple, pour recueillir la date et l’heure, les réponses possibles peuvent être « Quand voulez-vous venir ? » ou « Quand voulez-vous dîner avec nous ? »

#### <a name="prompt-listening-for-user-input"></a>Invite à l’écoute de l’entrée utilisateur

Après que l’utilisateur a été invité à répondre, le runtime de conversation écoute automatiquement l’entrée utilisateur et essaie de comprendre ce qu’elle dit. Configurez un déclencheur basé sur LUIS ou un module de reconnaissance de code personnalisé pour essayer de comprendre l’entrée utilisateur et son intention. Cela est similaire au déclencheur de tâche.

#### <a name="re-prompting-the-user"></a>Nouvel envoi d’invite à l’utilisateur

Utilisez la section Re-prompt pour spécifier une réponse à chaque tentative. Chaque ligne dans la section Re-prompt correspond à la chaîne de nouvelle invite utilisée pour ce tour spécifique. La première réponse est utilisée pour la première nouvelle invite, la deuxième, pour la deuxième, et ainsi de suite. Par exemple : 

*Je suis désolé, je n ’ai pas compris. Quand voulez-vous venir ?*
*Pardonnez-moi, j’ai du mal à vous comprendre. Essayons à nouveau. Quand vouliez-vous venir ?*

#### <a name="prompt-callback-functions"></a>Fonctions de rappel d’invite

Vous pouvez spécifier deux fonctions de rappel différentes sur un état Prompt. 

1. **Before every prompt and reprompt** : exécuter cette fonction avant chaque invite ou nouvelle invite. Cette fonction de rappel attend une valeur de retour booléenne où true indique d’exécuter cette invite ou une nouvelle invite, et false de n’en exécuter aucune des deux. Vous pouvez utiliser `getTurnIndex()` pour obtenir l’index de tour actuel pour cette exécution d’invite.
2. **On responding** : exécuter cette fonction chaque fois qu’une invite a été générée, mais avant qu’elle ait été envoyée à l’utilisateur (en incluant la réponse à la nouvelle invite). Cela permet au script de modifier le message envoyé à l’utilisateur.

#### <a name="sample-code"></a>Exemple de code

Cet extrait de code présente un exemple de rappel **avant exécution**.

```javascript
module.exports.fnBeforeExecuting = function(context) {
    if(context.responses[0].text === "C") {
        return false;
    }
    return true;
}
```

Cet extrait de code présente un exemple de rappel **sur invitation**.

```javascript
module.exports.fnOnPrompting = function(context) {
    // include a hint card
    var activity = context.responses.slice(-1).pop();
    activity.attachments.push({
        "contentType": "application/vnd.microsoft.card.hero",
        "content": {
            "buttons": [
                {
                    "type": "imBack",
                    "title": "1",
                    "value": "1"
                },
                {
                    "type": "imBack",
                    "title": "2",
                    "value": "2"
                }
            ]
        }
    });
}
```

Cet extrait de code présente un exemple de rappel **avant nouvelle invitation**.

```javascript
module.exports.fnBeforeReprompting = function(context) {
    if(context.responses[0].text === "C") {
        return false;
    }
    return true;
}
```

### <a name="feedback-state"></a>État Feedback

Cet état permet de fournir une réponse à l’utilisateur. Des cas d’utilisation typiques de cet état sont l’envoi du résultat final après une tentative d’accomplissement de tâche, la fourniture d’une réponse à l’utilisateur dans des conditions d’échec, etc. 

Chaque état Feedback requiert un nom unique, certaines valeurs de réponse possible, et peut éventuellement inclure des définitions de cartes adaptatives. Apprenez-en davantage sur la [définition de cartes adaptatives](conversation-designer-adaptive-cards.md).

Chaque état Feedback permet d’utiliser une fonction de rappel **On responding** dans laquelle vous pouvez écrire votre script personnalisé pour modifier la charge utile d’activité si nécessaire avant son envoi à l’utilisateur. 


```javascript
module.exports.fnOnResponding = function(context) {
    // include a hint card
    var activity = context.responses.slice(-1).pop();
    activity.attachments.push({
        "contentType": "application/vnd.microsoft.card.hero",
        "content": {
            "buttons": [
                {
                    "type": "imBack",
                    "title": "1",
                    "value": "1"
                },
                {
                    "type": "imBack",
                    "title": "2",
                    "value": "2"
                }
            ]
        }
    });

}
```

### <a name="module-state"></a>État Module

Un état Module est utilisé pour ajouter une référence pour l’exécution d’un sous-dialogue. Utilisez cet état pour relier des dialogues. 

Chaque état Module requiert un nom unique et un pointeur vers un dialogue spécifique à exécuter. Les options possibles pour les dialogues à exécuter doivent figurer sous **Shared dialogues** (Dialogues partagés) ou sous des **tâches** spécifiques.

## <a name="multiple-dialogues-under-a-task"></a>Plusieurs dialogues sous une tâche

Une tâche peut comporter plusieurs dialogues. Pour ajouter un dialogue à une tâche, sélectionnez celle-ci, cliquez sur le bouton **Ajouter** dans le panneau d’arborescence à gauche, puis cliquez sur **Add dialogue** (Ajouter un dialogue). Cette opération ajoute un dialogue sous la tâche sélectionnée. 

Les dialogues étant composables, vous pouvez lier le flux de dialogue racine à la tâche qui appelle d’autres dialogues sous elle. Cela vous permet encapsuler des sous-tâches et de permettre leur réutilisation. Cela vous permet également d’enchaîner ces dialogues dans un flux de conversation à l’aide d’états *Module*.

## <a name="next-step"></a>Étape suivante
> [!div class="nextstepaction"]
> [Modèles de réponse](conversation-designer-response-templates.md)
