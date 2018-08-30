---
title: Fonctionnalités avancées de FormFlow | Microsoft Docs
description: Découvrez comment personnaliser l’expérience utilisateur avec FormFlow et le Kit SDK Bot Builder pour .NET.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 6637876b016b8680fe722602f530a0c6b0ddfc5a
ms.sourcegitcommit: 2dc75701b169d822c9499e393439161bc87639d2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/24/2018
ms.locfileid: "42905403"
---
# <a name="advanced-features-of-formflow"></a>Fonctionnalités avancées de FormFlow

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

L’article sur les [fonctionnalités de base de FormFlow](bot-builder-dotnet-formflow.md) décrit une implémentation de FormFlow simple qui offre une expérience utilisateur plutôt générale. Pour fournir une expérience utilisateur plus personnalisée au moyen de FormFlow, vous pouvez spécifier l’état du formulaire initial, ajouter une logique métier permettant de gérer les interdépendances entre les champs et les entrées utilisateur du processus, et utiliser des attributs pour personnaliser les invites, remplacer les modèles, désigner les champs facultatifs, faire correspondre l’entrée utilisateur et valider l’entrée utilisateur. 

## <a name="specify-initial-form-state-and-entities"></a>Spécifier les entités et l’état du formulaire initial

Lorsque vous lancez [FormDialog][formDialog], vous pouvez le cas échéant passer une instance de votre état. Si vous décidez de passer une instance de votre état, par défaut FormFlow ignore les étapes de tous les champs qui contiennent déjà des valeurs ; l’utilisateur n’est pas invité à renseigner ces champs. Pour forcer le formulaire à proposer à l’utilisateur de remplir tous les champs (y compris les champs qui contiennent déjà des valeurs dans l’état initial), passez dans [FormOptions.PromptFieldsWithValues][promptFieldsWithValues] lorsque vous lancez `FormDialog`. Si un champ contient une valeur initiale, l’invite utilise cette valeur comme valeur par défaut.

Vous pouvez également transmettre des entités [LUIS](https://luis.ai/) à lier à l’état. Si `EntityRecommendation.Type` est le chemin d’un champ dans votre classe C#, `EntityRecommendation.Entity` est transmis via le module de reconnaissance pour se lier à votre champ. FormFlow ignore les étapes des champs qui sont liés à une entité ; l’utilisateur n’est donc pas invité à renseigner ces champs-là. 

## <a name="add-business-logic"></a>Ajouter une logique métier 

Pour gérer les interdépendances entre les champs de formulaire, ou appliquer une logique spécifique au cours du processus d’obtention ou de définition d’une valeur de champ, vous pouvez spécifier une logique métier dans une fonction de validation. Une fonction de validation vous permet de manipuler l’état et de retourner un objet [ValidateResult][validateResult] qui peut contenir : 

- une chaîne de commentaires décrivant la raison pour laquelle une valeur n’est pas valide ;
- une valeur transformée ;
- un ensemble de choix pour clarifier une valeur.

Cet exemple de code illustre une fonction de validation pour le champ `Toppings`. Si l’entrée du champ contient la valeur d’énumération `ToppingOptions.Everything`, la fonction garantit que la valeur du champ `Toppings` contient la liste complète des ingrédients.

[!code-csharp[Validation function](../includes/code/dotnet-formflow-advanced.cs#validationFunction)]

En plus de la fonction de validation, vous pouvez ajouter l’attribut [Terms](#match-user-input-using-the-terms-attribute) pour faire correspondre les expressions utilisateur, telles que « tout » ou « pas de ».

[!code-csharp[Terms for Toppings](../includes/code/dotnet-formflow-advanced.cs#toppingsTerms)]

À l’aide de la fonction de validation illustrée ci-dessus, cet extrait de code montre l’interaction entre le bot et l’utilisateur, lorsque l’utilisateur demande « everything but Jalapenos (tout sauf des Jalapenos) ». 

```console
Please select one or more toppings (current choice: No Preference)
 1. Everything
 2. Avocado
 3. Banana Peppers
 4. Cucumbers
 5. Green Bell Peppers
 6. Jalapenos
 7. Lettuce
 8. Olives
 9. Pickles
 10. Red Onion
 11. Spinach
 12. Tomatoes
> everything but jalapenos
For sandwich toppings you have selected Avocado, Banana Peppers, Cucumbers, Green Bell Peppers, Lettuce, Olives, Pickles, Red Onion, Spinach, and Tomatoes.
```

## <a name="formflow-attributes"></a>Attributs FormFlow

Vous pouvez ajouter ces attributs C# à votre classe pour personnaliser le comportement d’un dialogue FormFlow.

| Attribut | Objectif |
|----|----| 
| [Describe][describeAttribute] | Modifier la façon dont une valeur ou un champ est affiché dans un modèle ou une carte |
| [Numeric][numericAttribute] | Restreindre les valeurs acceptées d’un champ numérique |
| [Optional][optionalAttribute] | Marquer un champ comme étant facultatif |
| [Pattern][patternAttribute] | Définir une expression régulière pour valider un champ de chaîne |
| [Prompt][promptAttribute] | Définir l’invite d’un champ |
| [Template][templateAttribute] | Définir le modèle à utiliser pour générer des invites ou des valeurs dans les invites |
| [Terms][termsAttribute] | Définir les termes d’entrée qui correspondent à un champ ou à une valeur |

## <a name="customize-prompts-using-the-prompt-attribute"></a>Personnaliser les invites avec l’attribut Prompt

Les invites par défaut sont automatiquement générées pour chaque champ dans votre formulaire, mais vous pouvez spécifier une invite personnalisée pour un champ au moyen de l’attribut `Prompt`. Par exemple, si l’invite par défaut du champ `SandwichOrder.Sandwich` est « Veuillez sélectionner un sandwich », vous pouvez ajouter l’attribut `Prompt` permettant de spécifier une invite personnalisée pour ce champ.

[!code-csharp[Prompt attribute](../includes/code/dotnet-formflow-advanced.cs#promptAttribute)]

Cet exemple utilise le [langage du modèle](bot-builder-dotnet-formflow-pattern-language.md) pour remplir dynamiquement l’invite avec des données de formulaire lors de l’exécution : `{&}` est remplacé par la description du champ, et `{||}` par la liste de choix dans l’énumération. 

> [!NOTE]
> Par défaut, la description d’un champ est générée à partir du nom du champ. Pour spécifier une description personnalisée d’un champ, ajoutez l’attribut `Describe`.

Cet extrait de code montre l’invite personnalisée qui est spécifiée par l’exemple ci-dessus.

```console
What kind of sandwich would you like?
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

Un attribut `Prompt` peut également préciser des paramètres qui affectent la façon dont le formulaire affiche l’invite. Par exemple, le paramètre `ChoiceFormat` détermine la façon dont le formulaire présente la liste de choix.

[!code-csharp[Prompt attribute ChoiceFormat parameter](../includes/code/dotnet-formflow-advanced.cs#promptChoice)]

Dans cet exemple, la valeur du paramètre `ChoiceFormat` indique que les choix doivent être affichés sous forme de liste à puces (au lieu d’une liste numérotée).

```console
What kind of sandwich would you like?
- BLT
- Black Forest Ham
- Buffalo Chicken
- Chicken And Bacon Ranch Melt
- Cold Cut Combo
- Meatball Marinara
- Oven Roasted Chicken
- Roast Beef
- Rotisserie Style Chicken
- Spicy Italian
- Steak And Cheese
- Sweet Onion Teriyaki
- Tuna
- Turkey Breast
- Veggie
>
```

## <a name="customize-prompts-using-the-template-attribute"></a>Personnaliser les invites avec l’attribut Template

Tandis que l’attribut `Prompt` vous permet de personnaliser l’invite d’un champ unique, l’attribut `Template` offre la possibilité de remplacer les modèles par défaut que FormFlow utilise pour générer automatiquement des invites. Cet exemple de code utilise l’attribut `Template` pour redéfinir la façon dont le formulaire gère tous les champs d’énumération. L’attribut indique que l’utilisateur ne peut sélectionner qu’un élément, il définit le texte d’invite au moyen du [langage du modèle](bot-builder-dotnet-formflow-pattern-language.md) et précise que le formulaire doit afficher un seul élément par ligne. 

[!code-csharp[Template attribute](../includes/code/dotnet-formflow-advanced.cs#templateAttribute)]

Cet extrait de code montre les invites obtenues pour le champ `Bread` et le champ `Cheese`.

```console
What kind of bread would you like on your sandwich?
 1. Nine Grain Wheat
 2. Nine Grain Honey Oat
 3. Italian
 4. Italian Herbs And Cheese
 5. Flatbread
> 

What kind of cheese would you like on your sandwich? 
 1. American
 2. Monterey Cheddar
 3. Pepperjack
> 
```

Si vous utilisez l’attribut `Template` pour remplacer les modèles par défaut utilisés par FormFlow pour générer des invites, vous avez peut-être envie d’apporter quelques variantes dans les invites et les messages que le formulaire génère. Ainsi, vous pouvez définir plusieurs chaînes de texte à l’aide du [langage du modèle](bot-builder-dotnet-formflow-pattern-language.md), pour que le formulaire choisisse au hasard parmi les options disponibles chaque fois qu’il a besoin d’afficher une invite ou un message.

Cet exemple de code redéfinit le modèle [TemplateUsage.NotUnderstood][notUnderstood] pour préciser deux autres variantes du message. Lorsque le bot doit faire savoir qu’il ne comprend pas une entrée d’utilisateur, il détermine le contenu du message par une sélection aléatoire de l’une des deux chaînes de texte. 

[!code-csharp[Template variations of message](../includes/code/dotnet-formflow-advanced.cs#templateMessages)]

Cet extrait de code montre un exemple de l’interaction résultante entre le bot et l’utilisateur. 

```console
What size of sandwich do you want? (1. Six Inch, 2. Foot Long)
> two feet
I do not understand "two feet".
> two feet
Try again, I don't get "two feet"
> 
```

## <a name="designate-a-field-as-optional-using-the-optional-attribute"></a>Désigner un champ comme étant facultatif à l’aide de l’attribut Optional

La désignation d’un champ en tant que facultatif se fait au moyen de l’attribut `Optional`. Cet exemple de code précise que le champ `Cheese` est facultatif.

[!code-csharp[Optional attribute](../includes/code/dotnet-formflow-advanced.cs#optionalAttribute)]

Si un champ est facultatif et qu’aucune valeur n’a été indiquée, le choix actuel s’affiche comme étant « Aucune préférence ».

```console
What kind of cheese would you like on your sandwich? (current choice: No Preference)
  1. American
  2. Monterey Cheddar
  3. Pepperjack
 >
```

Si un champ est facultatif et que l’utilisateur a précisé une valeur, « Aucune préférence » s’affiche en tant que dernier choix dans la liste.

```console
What kind of cheese would you like on your sandwich? (current choice: American)
 1. American
 2. Monterey Cheddar
 3. Pepperjack
 4. No Preference
>
```

## <a name="match-user-input-using-the-terms-attribute"></a>Faire correspondre l’entrée utilisateur au moyen de l’attribut Termes

Lorsqu’un utilisateur envoie un message à un bot créé à l’aide de FormFlow, le bot tente d’identifier la signification de l’entrée de l’utilisateur en faisant correspondre cette entrée avec une liste de termes. Par défaut, la liste de termes est générée en appliquant les étapes qui suivent au champ ou à la valeur : 

1. S’arrêter sur les modifications de casse et les traits de soulignement (_).
2. Générer chaque <a href="https://en.wikipedia.org/wiki/N-gram" target="_blank">n-gramme</a> jusqu’à une longueur maximale.
3. Ajouter « s? » à la fin de chaque mot (pour prendre en charge les pluriels). 

Par exemple, la valeur « PizzaAilEtBœufAngus » génère ces termes : 

- 'pizzas?'
- 'ails?'
- 'bœufs?'
- 'angus?'
- 'pizzas? ails?'
- 'bœufs? angus?'
- 'pizza ail et bœuf angus'

Pour remplacer ce comportement par défaut et définir la liste des termes à utiliser pour établir des correspondances entre les entrées d’utilisateur et un champ ou une valeur de champ, utilisez l’attribut `Terms`. Par exemple, vous pouvez utiliser l’attribut `Terms` (avec une expression régulière) pour prendre en compte le fait que les utilisateurs sont susceptibles de mal orthographier le mot « rotisserie (rôtisserie) ». 

[!code-csharp[Terms attribute](../includes/code/dotnet-formflow-advanced.cs#termsAttribute)]

En utilisant l’attribut `Terms`, vous augmentez vos chances de pouvoir faire correspondre les entrées d’utilisateur avec l’un des choix valides. Le paramètre `Terms.MaxPhrase` dans cet exemple oblige `Language.GenerateTerms` à générer des variantes de termes supplémentaires. 

Cet extrait de code montre l’interaction résultante entre le bot et l’utilisateur lorsque l’utilisateur a mal orthographié « Rotisserie. »

```console
What kind of sandwich would you like?
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
> rotissary checkin
For sandwich I understood Rotisserie Style Chicken. "checkin" is not an option.
```

## <a name="validate-user-input-using-the-numeric-attribute-or-pattern-attribute"></a>Valider les entrées utilisateur au moyen de l’attribut Numeric ou Pattern

Pour restreindre la plage des valeurs autorisées pour un champ numérique, utilisez l’attribut `Numeric`. Cet exemple de code utilise l’attribut `Numeric` pour spécifier que l’entrée du champ `Rating` doit être un nombre compris entre 1 et 5. 

[!code-csharp[Numeric attribute](../includes/code/dotnet-formflow-advanced.cs#numericAttribute)]

Pour indiquer le format voulu pour la valeur d’un champ particulier, utilisez l’attribut `Pattern`. Cet exemple de code utilise l’attribut `Pattern` permettant de préciser le format voulu pour la valeur du champ `PhoneNumber`.

[!code-csharp[Pattern attribute](../includes/code/dotnet-formflow-advanced.cs#patternAttribute)]

## <a name="summary"></a>Résumé

Cet article a traité en détail l’apport d’une expérience utilisateur personnalisée avec FormFlow, en spécifiant l’état du formulaire initial,en ajoutant une logique métier qui permet de gérer les interdépendances entre les champs et les entrées utilisateur du processus, et en utilisant des attributs pour personnaliser les invites, remplacer les modèles, désigner les champs facultatifs, faire correspondre l’entrée utilisateur et valider l’entrée utilisateur. Pour obtenir des informations sur les autres moyens de personnaliser l’expérience utilisateur avec FormFlow, consultez [Personnaliser un formulaire à l’aide de FormBuilder](bot-builder-dotnet-formflow-formbuilder.md).

## <a name="sample-code"></a>Exemple de code

[!INCLUDE [Sample code](../includes/snippet-dotnet-formflow-samples.md)]

## <a name="additional-resources"></a>Ressources supplémentaires

- [Fonctionnalités de base de FormFlow](bot-builder-dotnet-formflow.md)
- [Personnaliser un formulaire à l’aide de FormBuilder](bot-builder-dotnet-formflow-formbuilder.md)
- [Localiser le contenu d’un formulaire](bot-builder-dotnet-formflow-localize.md)
- [Définir un formulaire à l’aide d’un schéma JSON](bot-builder-dotnet-formflow-json-schema.md)
- [Personnaliser l’expérience utilisateur avec le langage du modèle](bot-builder-dotnet-formflow-pattern-language.md)
- <a href="/dotnet/api/?view=botbuilder-3.11.0" target="_blank">Référence sur le Kit SDK Bot Builder pour .NET</a>

[formDialog]: /dotnet/api/microsoft.bot.builder.formflow.formdialog

[promptFieldsWithValues]: /dotnet/api/microsoft.bot.builder.formflow.formoptions.promptfieldswithvalues

[validateResult]: /dotnet/api/microsoft.bot.builder.formflow.validateresult

[describeAttribute]: /dotnet/api/microsoft.bot.builder.formflow.describeattribute

[numericAttribute]: /dotnet/api/microsoft.bot.builder.formflow.numericattribute

[optionalAttribute]: /dotnet/api/microsoft.bot.builder.formflow.optionalattribute

[patternAttribute]: /dotnet/api/microsoft.bot.builder.formflow.patternattribute

[promptAttribute]: /dotnet/api/microsoft.bot.builder.formflow.promptattribute

[templateAttribute]: /dotnet/api/microsoft.bot.builder.formflow.templateattribute

[termsAttribute]: /dotnet/api/microsoft.bot.builder.formflow.termsattribute

[notUnderstood]: /dotnet/api/microsoft.bot.builder.formflow.templateusage.notunderstood
