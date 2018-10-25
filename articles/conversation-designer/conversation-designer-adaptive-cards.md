---
title: Configurer des cartes adaptatives | Microsoft Docs
description: Découvrez comment configurer des cartes adaptatives.
author: vkannan
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
ROBOTS: NoIndex, NoFollow
ms.openlocfilehash: d41c2c24ed38fffe76cd73a6bb8a685d3861ac55
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2018
ms.locfileid: "50000426"
---
# <a name="configure-adaptive-cards"></a>Configurer des cartes adaptatives
> [!IMPORTANT]
> Conversation Designer n’est pas encore accessible à tous les clients. Des informations supplémentaires sur la disponibilité de Conversation Designer seront communiquées plus tard cette année.

Les <a href="http://adaptivecards.io" target="_blank">cartes adaptatives</a> constituent un nouveau schéma qui définit des cartes d’interface utilisateur riches à utiliser sur différents points de terminaison, dont les canaux Microsoft Bot Framework. 

Conversation Designer fournit un environnement de création profondément intégré pour créer, prévisualiser et utiliser des cartes adaptatives dans vos robots. 

Des cartes adaptatives peuvent être définies dans différents emplacements clés.

- Réponse simple à une action pour une tâche.
- Dans un état de commentaire dans un dialogue.
- Dans des états d’invite dans un dialogue. Notez que les invites peuvent avoir des cartes distinctes : une pour la réponse et une autre pour une l’envoi d’une nouvelle invite.

Pour définir une carte adaptative, accédez à l’éditeur approprié. Recherchez et choisissez l’un des modèles de carte adaptative existants, ou créez votre propre modèle dans l’éditeur de code JSON. 

Lorsque vous créez une carte, un aperçu riche de celle-ci est rendu dans le portail de création.

> [!NOTE]
> Les fonctionnalités des cartes adaptatives sont en développement permanent. Tous les canaux ne prennent pas en charge toutes les fonctionnalités des cartes adaptatives pour le moment. Pour connaître les fonctionnalités que chaque canal prend en charge, consultez la section État de canal.

## <a name="input-form"></a>Formulaire d’entrée

Des cartes adaptatives peuvent contenir des formulaires d’entrée. Dans Conversation Designer, les formulaires sont intégrés avec des entités de tâche. Par exemple, si un champ a une `id` **myName** et que l’action `Submit` du formulaire est exécutée, une `taskEntity` nommée **myName** est créée, qui contient la valeur du champ. 

L’extrait de code ci-dessous montre comment l’entité **myName** est définie dans le code :

``javascript
{
   "type": "Input.Text",
   "id": "myName",
   "placeholder": "Last, First"
}
``

De plus, si un champ a un id `@task`, la valeur du champ est utilisée comme nom de tâche. Lorsque ce champ est déclenché (par exemple sur un clic de bouton), la tâche nommée est exécutée. 

Prenons, par exemple, cet extrait de code :

``javascript
{
  'type': 'Action.Submit',
  'title': 'Search',
  'speak': '<s>Search</s>',
  'data': {
    '@task': 'Hotel Search'
  }
}
``

Quand l’utilisateur clique sur ce bouton, une action d’envoi est déclenchée et le `context.sticky` est défini sur `Hotel Search`. Cela entraîne l’exécution de la tâche **Recherche d’hôtel**. Pour utiliser cette fonctionnalité, assurez-vous que le `@task` correspond à un nom de tâche que vous avez défini dans Conversation Designer.

## <a name="use-entities-and-language-generation-templates"></a>Utiliser des entités et des modèles de génération de langage
Les cartes adaptatives prennent en charge la résolution de génération de langage complète.

* `entityName` utilise des entités à l’intérieur de la carte.
* `responseTemplateName` utilise des modèles de réponses simples ou conditionnelles à l’intérieur de la carte.

Vous en apprendrez davantage sur les cartes adaptatives à l’emplacement suivant TODO : insérer un lien d’accès à la documentation relative au schéma des cartes adaptatives -->

## <a name="sample-adaptive-card-payload"></a>Exemple de charge utile de carte adaptative

Le JSON suivant montre la charge utile d’une carte adaptative.

```json
{
    "$schema": "https://microsoft.github.io/AdaptiveCards/schemas/adaptive-card.json",
    "type": "AdaptiveCard",
    "version": "1.0",
    "body": [
        {
            "speak": "<s>Serious Pie is a Pizza restaurant which is rated 9.3 by customers.</s>",
            "type": "ColumnSet",
            "columns": [
                {
                    "type": "Column",
                    "size": "2",
                    "items": [
                        {
                            "type": "TextBlock",
                            "text": "[Greeting], [TimeOfDayTemplate], You can eat in {location}",
                            "weight": "bolder",
                            "size": "extraLarge"
                        },
                        {
                            "type": "TextBlock",
                            "text": "9.3 · $$ · Pizza",
                            "isSubtle": true
                        },
                        {
                            "type": "TextBlock",
                            "text": "[builtin.feedback.display]",
                            "wrap": true
                        }
                    ]
                },
                {
                    "type": "Column",
                    "size": "1",
                    "items": [
                        {
                            "type": "Image",
                            "url": "http://res.cloudinary.com/sagacity/image/upload/c_crop,h_670,w_635,x_0,y_0/c_scale,w_640/v1397425743/Untitled-4_lviznp.jpg",
                            "size":"auto"
                        }
                    ]
                }
            ]
        }
    ],
    "actions": [
        {
            "type": "Action.Http",
            "method": "POST",
            "title": "More Info",
            "url": "http://foo.com"
        },
        {
            "type": "Action.Http",
            "method": "POST",
            "title": "View on Foursquare",
            "url": "http://foo.com"
        }
    ]
}
```

