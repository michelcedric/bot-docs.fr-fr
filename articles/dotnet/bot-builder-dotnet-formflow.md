---
title: Fonctionnalités de base de FormFlow | Microsoft Docs
description: Découvrez comment guider les flux de conversation en utilisant FormFlow dans le Kit de développement logiciel (SDK) Bot Builder pour .NET.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: cf3d69f7941d8c3177788bd00e4b58416a71cf5e
ms.sourcegitcommit: 2dc75701b169d822c9499e393439161bc87639d2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/24/2018
ms.locfileid: "42904623"
---
# <a name="basic-features-of-formflow"></a>Fonctionnalités de base de FormFlow

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

Si les [boîtes de dialogue](bot-builder-dotnet-dialogs.md) sont à la fois flexibles et très puissantes, la gestion d’une conversation guidée, par exemple pour commander un sandwich, peut nécessiter beaucoup d’efforts. Chaque stade de la conversation peut donner lieu à une multitude de possibilités pour la suite. Par exemple, vous devrez peut-être éclaircir une ambiguïté, fournir de l’aide, revenir en arrière ou afficher la progression. En utilisant **FormFlow** dans le Kit de développement logiciel (SDK) Bot Builder pour .NET, vous pouvez considérablement simplifier le processus de gestion d’une conversation guidée de ce type. 

FormFlow génère automatiquement les boîtes de dialogue nécessaires à la gestion d’une conversation guidée en fonction des instructions que vous spécifiez. L’utilisation de FormFlow se fait au détriment de la flexibilité dont vous bénéficiez lorsque vous créez et gérez vous-même les boîtes de dialogue. En revanche, FormFlow réduit considérablement le temps nécessaire au développement de votre bot pour la conception d’une conversation guidée. En outre, vous pouvez créer votre bot en combinant des boîtes de dialogue générées par FormFlow et d’autres types de boîtes de dialogue. Par exemple, une boîte de dialogue FormFlow peut guider l’utilisateur dans le processus de remplissage d’un formulaire, et une boîte de dialogue [LuisDialog][LuisDialog] peut évaluer l’entrée de l’utilisateur pour déterminer l’intention.

Cet article décrit comment créer un bot qui utilise les fonctionnalités de base de FormFlow pour collecter des informations auprès d’un utilisateur.

## <a id="forms-and-fields"></a>Formulaires et champs

Pour créer un bot à l’aide de FormFlow, vous devez spécifier les informations que le bot doit collecter auprès de l’utilisateur. Par exemple, si l’objectif du bot est d’obtenir une commande de sandwich auprès de l’utilisateur, vous devez définir un formulaire contenant des champs permettant de recueillir les données nécessaires au bot pour exécuter la commande. Vous pouvez définir le formulaire en créant une classe C# contenant une ou plusieurs propriétés publiques pour représenter les données que le bot va collecter auprès de l’utilisateur. Chaque propriété doit correspondre à l’un des types de données suivants :

- Intégral (sbyte, byte, short, ushort, int, uint, long, ulong)
- Virgule flottante (float, double)
- Chaîne
- Datetime
- Énumération
- Liste d’énumérations

Tous ces types de données peuvent être Nullable et utilisés pour spécifier que le champ n’a pas de valeur. Si un champ de formulaire est basé sur une propriété d’énumération qui n’est pas Nullable, la valeur **0** dans l’énumération représente **null** (c’est-à-dire qu’elle indique que le champ n’a pas de valeur), et vous devez commencer vos valeurs d’énumération par **1**. FormFlow ignore tous les autres types de propriétés et méthodes.

Pour les objets complexes, vous devez créer un formulaire pour la classe C# de niveau supérieur et un autre formulaire pour l’objet complexe. Vous pouvez composer les formulaires à l’aide de la sémantique habituelle des [boîtes de dialogue](bot-builder-dotnet-dialogs.md). Vous pouvez également définir un formulaire directement en implémentant [Advanced.IField][iField] ou en utilisant [Advanced.Field][field] et en remplissant les dictionnaires qu’il contient. 

> [!NOTE]
> Vous pouvez définir un formulaire à l’aide d’une classe C# ou d’un schéma JSON. Cet article décrit comment définir un formulaire à l’aide d’une classe C#. Pour plus d’informations sur l’utilisation d’un schéma JSON, consultez [Définir un formulaire à l’aide du schéma JSON](bot-builder-dotnet-formflow-json-schema.md).

## <a name="simple-sandwich-bot"></a>Bot simple pour la commande d’un sandwich

L’exemple suivant porte sur un bot simple conçu pour obtenir une commande de sandwich auprès d’un utilisateur. 

### <a id="create-class"></a> Création du formulaire

La classe `SandwichOrder` définit le formulaire, et les énumérations définissent les options relatives à la préparation d’un sandwich. La classe inclut également la méthode statique `BuildForm` qui utilise [FormBuilder][formBuilder] pour créer le formulaire et définir un simple message d’accueil. 

Pour utiliser FormFlow, vous devez d’abord importer l’espace de noms `Microsoft.Bot.Builder.FormFlow`.

[!code-csharp[Define form](../includes/code/dotnet-formflow.cs#defineForm)]

### <a name="connect-the-form-to-the-framework"></a>Connexion du formulaire à l’infrastructure 

Pour connecter le formulaire à l’infrastructure, vous devez l’ajouter au contrôleur. Dans cet exemple, la méthode `Conversation.SendAsync` appelle la méthode statique `MakeRootDialog` qui, qui à son tour, appelle la méthode `FormDialog.FromForm` pour créer le formulaire `SandwichOrder`. 

[!code-csharp[Connect form to framework](../includes/code/dotnet-formflow.cs#connectToFramework)]

### <a name="see-it-in-action"></a>Visualisation du processus en action

En définissant simplement le formulaire avec une classe C# et en le connectant à l’infrastructure, vous avez autorisé FormFlow à gérer automatiquement la conversation entre le bot et l’utilisateur. Les exemples d’interactions ci-après illustrent les capacités d’un bot créé à l’aide des fonctionnalités de base de FormFlow. Pour chaque interaction, un symbole **>** indique le stade auquel l’utilisateur entre une réponse. 

#### <a name="display-the-first-prompt"></a>Affichage de la première invite 

Ce formulaire renseigne la propriété `SandwichOrder.Sandwich`. Le formulaire génère automatiquement l’invite « Please select a sandwich » (Veuillez choisir un sandwich), où le mot « sandwich » dans l’invite est issu du nom de propriété `Sandwich`. L’énumération `SandwichOptions` définit les choix présentés à l’utilisateur, chaque valeur d’énumération étant automatiquement décomposée en mots en fonction des modifications de casse et des traits de soulignement.

```console
Please select a sandwich
1. BLT
2. Black Forest Ham
3. Buffalo Chicken
4. Chicken And Bacon Ranch Melt
5. Cold Cut Combo
6. Meatball Marinara
7. Oven Roasted Chicken
8. Roast Beef
9. Rotisserie Style Chicken
10. Spicy Italian
11. Steak And Cheese
12. Sweet Onion Teriyaki
13. Tuna
14. Turkey Breast
15. Veggie
>
```

#### <a name="provide-guidance"></a>Fourniture d’aide

L’utilisateur peut entrer le mot « help » (aide) à tout moment lors de la conversation pour obtenir de l’aide sur le remplissage du formulaire. Par exemple, si l’utilisateur entre « help » à l’invite relative au sandwich, le bot répondra avec l’aide correspondante. 

```console
> help
* You are filling in the sandwich field. Possible responses:
* You can enter a number 1-15 or words from the descriptions. (BLT, Black Forest Ham, Buffalo Chicken, Chicken And Bacon Ranch Melt, Cold Cut Combo, Meatball Marinara, Oven Roasted Chicken, Roast Beef, Rotisserie Style Chicken, Spicy Italian, Steak And Cheese, Sweet Onion Teriyaki, Tuna, Turkey Breast, and Veggie)
* Back: Go back to the previous question.
* Help: Show the kinds of responses you can enter.
* Quit: Quit the form without completing it.
* Reset: Start over filling in the form. (With defaults from your previous entries.)
* Status: Show your progress in filling in the form so far.
* You can switch to another field by entering its name. (Sandwich, Length, Bread, Cheese, Toppings, and Sauce).
```

#### <a name="advance-to-the-next-prompt"></a>Passage à l’invite suivante

Si l’utilisateur entre la valeur « 2 » en réponse à l’invite initiale relative au sandwich, le bot affiche une invite pour la propriété suivante définie par le formulaire : `SandwichOrder.Length`.

```console
Please select a sandwich
 1. BLT
 2. Black Forest Ham
 3. Buffalo Chicken
 4. Chicken And Bacon Ranch Melt
 5. Cold Cut Combo
 6. Meatball Marinara
 7. Oven Roasted Chicken
 8. Roast Beef
 9. Rotisserie Style Chicken
 10. Spicy Italian
 11. Steak And Cheese
 12. Sweet Onion Teriyaki
 13. Tuna
 14. Turkey Breast
 15. Veggie
> 2
Please select a length (1. Six Inch, 2. Foot Long)
> 
```

#### <a name="return-to-the-previous-prompt"></a>Retour à l’invite précédente 

Si l’utilisateur entre « back » (retour) à ce stade de la conversation, le bot renvoie l’invite précédente. L’invite affiche le choix actuel de l’utilisateur (« Black Forest Ham »). L’utilisateur peut modifier cette sélection en entrant un nombre différent ou la confirmant en entrant « c ».

```console
> back
Please select a sandwich(current choice: Black Forest Ham)
 1. BLT
 2. Black Forest Ham
 3. Buffalo Chicken
 4. Chicken And Bacon Ranch Melt
 5. Cold Cut Combo
 6. Meatball Marinara
 7. Oven Roasted Chicken
 8. Roast Beef
 9. Rotisserie Style Chicken
 10. Spicy Italian
 11. Steak And Cheese
 12. Sweet Onion Teriyaki
 13. Tuna
 14. Turkey Breast
 15. Veggie
> c
Please select a length (1. Six Inch, 2. Foot Long)
> 
```

#### <a name="clarify-user-input"></a>Clarification de l’entrée utilisateur

Si l’utilisateur répond avec du texte (au lieu d’un nombre) pour indiquer un choix et que l’entrée utilisateur correspond à plusieurs choix, le bot demandera automatiquement une clarification. 

```console
Please select a bread
 1. Nine Grain Wheat
 2. Nine Grain Honey Oat
 3. Italian
 4. Italian Herbs And Cheese
 5. Flatbread
> nine grain
By "nine grain" bread did you mean (1. Nine Grain Honey Oat, 2. Nine Grain Wheat)
> 1
```

Si l’entrée d’utilisateur ne correspond pas directement à l’un des choix valides, le bot demande automatiquement une clarification à l’utilisateur.

```console
Please select a cheese (1. American, 2. Monterey Cheddar, 3. Pepperjack)
> amercan
"amercan" is not a cheese option.
> american smoked
For cheese I understood American. "smoked" is not an option.
```

Si l’entrée utilisateur spécifie plusieurs choix pour une propriété et que le bot ne comprend aucun des choix spécifiés, il demande automatiquement une clarification à l’utilisateur.

```console
Please select one or more toppings
 1. Banana Peppers
 2. Cucumbers
 3. Green Bell Peppers
 4. Jalapenos
 5. Lettuce
 6. Olives
 7. Pickles
 8. Red Onion
 9. Spinach
 10. Tomatoes
> peppers, lettuce and tomato
By "peppers" toppings did you mean (1. Green Bell Peppers, 2. Banana Peppers)
> 1
```

#### <a name="show-current-status"></a>Affichage de l’état actuel

Si l’utilisateur entre le mot « status » (état) lors du processus de commande (à quelque moment que ce soit), la réponse du bot indique les valeurs qui ont déjà été spécifiées et celles qui doivent encore être spécifiées. 

```console
Please select one or more sauce
 1. Honey Mustard
 2. Light Mayonnaise
 3. Regular Mayonnaise
 4. Mustard
 5. Oil
 6. Pepper
 7. Ranch
 8. Sweet Onion
 9. Vinegar
> status
* Sandwich: Black Forest Ham
* Length: Six Inch
* Bread: Nine Grain Honey Oat
* Cheese: American
* Toppings: Lettuce, Tomatoes, and Green Bell Peppers
* Sauce: Unspecified  
```

#### <a name="confirm-selections"></a>Confirmation des sélections

Lorsque l’utilisateur arrive au bout du formulaire, le bot lui demande de confirmer ses sélections.

```console
Please select one or more sauce
 1. Honey Mustard
 2. Light Mayonnaise
 3. Regular Mayonnaise
 4. Mustard
 5. Oil
 6. Pepper
 7. Ranch
 8. Sweet Onion
 9. Vinegar
> 1
Is this your selection?
* Sandwich: Black Forest Ham
* Length: Six Inch
* Bread: Nine Grain Honey Oat
* Cheese: American
* Toppings: Lettuce, Tomatoes, and Green Bell Peppers
* Sauce: Honey Mustard
>
```

Si l’utilisateur répond en entrant « no » (non), le bot lui permet de mettre à jour les sélections précédentes de son choix. Si l’utilisateur répond en entrant « yes » (oui), le formulaire est terminé et le contrôle revient à la boîte de dialogue appelante. 

```console
Is this your selection?
* Sandwich: Black Forest Ham
* Length: Six Inch
* Bread: Nine Grain Honey Oat
* Cheese: American
* Toppings: Lettuce, Tomatoes, and Green Bell Peppers
* Sauce: Honey Mustard
> no
What do you want to change?
 1. Sandwich(Black Forest Ham)
 2. Length(Six Inch)
 3. Bread(Nine Grain Honey Oat)
 4. Cheese(American)
 5. Toppings(Lettuce, Tomatoes, and Green Bell Peppers)
 6. Sauce(Honey Mustard)
> 2
Please select a length (current choice: Six Inch) (1. Six Inch, 2. Foot Long)
> 2
Is this your selection?
* Sandwich: Black Forest Ham
* Length: Foot Long
* Bread: Nine Grain Honey Oat
* Cheese: American
* Toppings: Lettuce, Tomatoes, and Green Bell Peppers
* Sauce: Honey Mustard
> y
```

## <a name="handling-quit-and-exceptions"></a>Gestion des fermetures et exceptions

Si l’utilisateur entre « quit » (quitter) dans le formulaire ou qu’une exception se produit au cours de la conversation, votre bot devra connaître l’étape à laquelle l’événement s’est produit, l’état du formulaire à ce moment et les étapes du formulaire qui se sont déroulées correctement avant l’événement. Le formulaire renvoie ces informations par le biais de la classe `FormCanceledException<T>`. 

Cet exemple de code montre comment intercepter l’exception et afficher un message en fonction de l’événement. 

[!code-csharp[Handle exception or quit](../includes/code/dotnet-formflow.cs#handleExceptionOrQuit)]

## <a name="summary"></a>Résumé

Cet article décrit comment utiliser les fonctionnalités de base de FormFlow pour créer un bot capable d’effectuer les tâches suivantes :

- Générer et gérer automatiquement la conversation
- Fournir une aide et des conseils clairs
- Comprendre les nombres et les entrées de texte
- Indiquer à l’utilisateur ce qui a été compris et ce qui ne l’a pas été 
- Poser des questions de clarification si nécessaire 
- Permettre à l’utilisateur de naviguer entre les étapes

Même si les fonctionnalités de base de FormFlow sont suffisantes dans certains cas, réfléchissez aux avantages que pourrait apporter l’ajout de certaines fonctionnalités plus avancées de FormFlow à votre bot. Pour plus d’informations, consultez [Fonctionnalités avancées de FormFlow](bot-builder-dotnet-formflow-advanced.md) et [Personnaliser un formulaire à l’aide de FormBuilder](bot-builder-dotnet-formflow-formbuilder.md).

## <a name="sample-code"></a>Exemple de code

[!INCLUDE [Sample code](../includes/snippet-dotnet-formflow-samples.md)]

## <a name="next-steps"></a>Étapes suivantes

FormFlow simplifie le développement de boîtes de dialogue. Les fonctionnalités avancées de FormFlow vous permettent de personnaliser le comportement d’un objet FormFlow.

> [!div class="nextstepaction"]
> [Fonctionnalités avancées de FormFlow](bot-builder-dotnet-formflow-advanced.md)

## <a name="additional-resources"></a>Ressources supplémentaires

- [Personnaliser un formulaire à l’aide de FormBuilder](bot-builder-dotnet-formflow-formbuilder.md)
- [Localiser le contenu d’un formulaire](bot-builder-dotnet-formflow-localize.md)
- [Définir un formulaire à l’aide d’un schéma JSON](bot-builder-dotnet-formflow-json-schema.md)
- [Personnaliser l’expérience utilisateur avec le langage du modèle](bot-builder-dotnet-formflow-pattern-language.md)
- <a href="/dotnet/api/?view=botbuilder-3.11.0" target="_blank">Référence sur le Kit SDK Bot Builder pour .NET</a>

[LuisDialog]: /dotnet/api/microsoft.bot.builder.dialogs.luisdialog-1

[iField]: /dotnet/api/microsoft.bot.builder.formflow.advanced.ifield-1

[field]: /dotnet/api/microsoft.bot.builder.formflow.advanced.field-1

[formBuilder]: /dotnet/api/microsoft.bot.builder.formflow.formbuilder-1
