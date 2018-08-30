---
title: Personnaliser l’expérience utilisateur avec le langage du modèle | Microsoft Docs
description: Découvrez comment personnaliser les invites FormFlow et remplacer des modèles FormFlow à l’aide du langage du modèle avec le Kit SDK Bot Builder pour .NET.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: a192b69b2ffbac428d80b2fe7c3fd9180caacd4f
ms.sourcegitcommit: 2dc75701b169d822c9499e393439161bc87639d2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/24/2018
ms.locfileid: "42905581"
---
# <a name="customize-user-experience-with-pattern-language"></a>Personnaliser l’expérience utilisateur avec le langage du modèle

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

Lorsque vous personnalisez une invite ou remplacez un modèle par défaut, vous pouvez utiliser le langage du modèle pour spécifier le contenu et/ou le format de l’invite. 

## <a name="prompts-and-templates"></a>Invites et modèles

Une [invite][promptAttribute] définit le message envoyé à l’utilisateur pour demander une information ou une confirmation. Vous pouvez personnaliser une invite en utilisant l’[attribut Prompt](bot-builder-dotnet-formflow-advanced.md#customize-prompts-using-the-prompt-attribute) ou implicitement via [IFormBuilder<T>.Field][field]. 

Les formulaires utilisent des modèles pour construire automatiquement des invites et d’autres éléments tels que l’aide. Vous pouvez remplacer le modèle par défaut d’une classe ou d’un champ à l’aide de l’[attribut Template](bot-builder-dotnet-formflow-advanced.md#customize-prompts-using-the-template-attribute). 

> [!TIP]
> La classe [FormConfiguration.Templates][formConfiguration] définit un ensemble de modèles intégrés qui fournissent de bons exemples sur l’utilisation du langage du modèle.

## <a name="elements-of-pattern-language"></a>Éléments du langage du modèle

Le langage du modèle utilise des accolades (`{}`) pour identifier les éléments qui seront remplacés par des valeurs réelles lors de l’exécution. Ce tableau répertorie les éléments du langage du modèle.

| Élément | Description |
|----|----|
| `{<format>}` | Affiche la valeur du champ réel (champ auquel l’attribut s’applique). |
| `{&}` | Affiche la description du champ réel (sauf indication contraire, il s’agit du nom du champ). |
| `{<field><format>}` | Affiche la valeur du champ nommé. | 
| `{&<field>}` | Affiche la description du champ nommé. |
| <code>{&#124;&#124;}</code> | Affiche le choix actuel, qui pourrait être la valeur actuelle d’un champ, « aucune préférence » ou les valeurs d’une énumération. |
| `{[{<field><format>} ...]}` | Affiche une liste de valeurs des champs nommés à l’aide des attributs [Separator][separator] et [LastSeparator][lastSeparator] pour séparer les valeurs individuelles de la liste. |
| `{*}` | Affiche une ligne pour chaque champ actif, chaque ligne contenant la description du champ et sa valeur actuelle. | 
| `{*filled}` | Affiche une ligne pour chaque champ actif avec une valeur réelle, chaque ligne contenant la description du champ et sa valeur actuelle. |
| `{<nth><format>}` | Un spécificateur de format C# standard qui s’applique au nième argument d’un modèle. Pour obtenir la liste des arguments disponibles, consultez [TemplateUsage][templateUsage]. |
| `{?<textOrPatternElement>...}` | Substitution conditionnelle. Si tous les éléments de modèle ont des valeurs, les valeurs sont remplacées et l’expression entière est utilisée. |

Pour les éléments répertoriés ci-dessus :

- L’espace réservé `<field>` est le nom d’un champ dans votre classe de formulaire. Par exemple, si votre classe contient un champ portant le nom `Size`, vous pouvez spécifier `{Size}` comme élément de modèle. 

- Une ellipse (`"..."`) au sein d’un élément de modèle indique que l’élément peut contenir plusieurs valeurs.

- L’espace réservé `<format>` est un spécificateur de format C#. Par exemple, si la classe contient un champ portant le nom `Rating` et de type `double`, vous pouvez spécifier `{Rating:F2}` comme élément de modèle pour obtenir une précision à deux chiffres.

- L’espace réservé `<nth>` fait référence au nième argument d’un modèle.

### <a name="pattern-language-within-a-prompt-attribute"></a>Langage du modèle au sein d’un attribut Prompt

Cet exemple utilise l’élément `{&}` pour afficher la description du champ `Sandwich` et l’élément `{||}` pour afficher la liste des choix pour ce champ.

[!code-csharp[Patterns example](../includes/code/dotnet-formflow-pattern-language.cs#patterns1)]

Voici l’invite résultante :

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

## <a name="formatting-parameters"></a>Paramètres de mise en forme

Les invites et les modèles prennent en charge ces paramètres de mise en forme.

| Usage | Description |
|----|----|
| `AllowDefault` | S’applique aux éléments de modèle <code>{&#124;&#124;}</code>. Détermine si le formulaire doit afficher la valeur actuelle du champ comme un choix possible. Si `true`, la valeur actuelle apparaît comme une valeur possible. Par défaut, il s’agit de `true`. |
| `ChoiceCase` | S’applique aux éléments de modèle <code>{&#124;&#124;}</code>. Détermine si le texte de chaque choix est normalisé (par exemple, si la première lettre de chaque mot est une majuscule). Par défaut, il s’agit de `CaseNormalization.None`. Pour les valeurs possibles, consultez [CaseNormalization][caseNormalization]. |
| `ChoiceFormat` | S’applique aux éléments de modèle <code>{&#124;&#124;}</code>. Détermine s’il faut afficher une liste de choix sous forme d’une liste numérotée ou d’une liste à puces. Pour une liste numérotée, définissez `ChoiceFormat` sur `{0}` (valeur par défaut). Pour une liste à puces, définissez `ChoiceFormat` sur `{1}`. |
| `ChoiceLastSeparator` | S’applique aux éléments de modèle <code>{&#124;&#124;}</code>. Détermine si une liste incorporée de choix inclut un séparateur avant le dernier choix. |
| `ChoiceParens` | S’applique aux éléments de modèle <code>{&#124;&#124;}</code>. Détermine si une liste incorporée de choix s’affiche entre parenthèses. Si `true`, la liste de choix s’affiche entre parenthèses. Par défaut, il s’agit de `true`. |
| `ChoiceSeparator` | S’applique aux éléments de modèle <code>{&#124;&#124;}</code>. Détermine si une liste incorporée de choix inclut un séparateur avant chaque choix, à l’exception du dernier. | 
| `ChoiceStyle` | S’applique aux éléments de modèle <code>{&#124;&#124;}</code>. Détermine si la liste de choix s’affiche de façon incorporée ou par ligne. La valeur par défaut `ChoiceStyleOptions.Auto` détermine, lors de l’exécution, s’il faut afficher le choix de façon incorporée ou dans une liste. Pour connaître les valeurs possibles, consultez [ChoiceStyleOptions][choiceStyleOptions]. |
| `Feedback` | S’applique aux invites uniquement. Détermine si le formulaire renvoie le choix de l’utilisateur pour indiquer que le formulaire a compris la sélection. La valeur par défaut `FeedbackOptions.Auto` renvoie l’entrée utilisateur uniquement si une partie de celle-ci n’est pas comprise. Pour connaître les valeurs possibles, consultez [FeedbackOptions][feedbackOptions]. |
| `FieldCase` | Détermine si le texte de la description du champ est normalisé (par exemple, si la première lettre de chaque mot est une majuscule). La valeur par défaut `CaseNormalization.Lower` convertit la description en minuscules. Pour les valeurs possibles, consultez [CaseNormalization][caseNormalization]. |
| `LastSeparator` | S’applique aux éléments de modèle `{[]}`. Détermine si un tableau d’éléments inclut un séparateur avant le dernier élément. |
| `Separator` | S’applique aux éléments de modèle `{[]}`. Détermine si un tableau d’éléments inclut un séparateur avant chaque élément, à l’exception du dernier. |
| `ValueCase` | Détermine si le texte de la valeur du champ est normalisé (par exemple, si la première lettre de chaque mot est une majuscule). La valeur par défaut `CaseNormalization.InitialUpper` convertit la première lettre de chaque mot en majuscule. Pour les valeurs possibles, consultez [CaseNormalization][caseNormalization]. |

### <a name="prompt-attribute-with-formatting-parameter"></a>Attribut Prompt avec paramètre de mise en forme 

Cet exemple utilise le paramètre `ChoiceFormat` pour spécifier que la liste de choix doit être affichée sous forme de liste à puces.

[!code-csharp[Patterns example](../includes/code/dotnet-formflow-pattern-language.cs#patterns2)]

Voici l’invite résultante :

```console
What kind of sandwich would you like?
* BLT
* Black Forest Ham
* Buffalo Chicken
* Chicken And Bacon Ranch Melt
* Cold Cut Combo
* Meatball Marinara
* Oven Roasted Chicken
* Roast Beef
* Rotisserie Style Chicken
* Spicy Italian
* Steak And Cheese
* Sweet Onion Teriyaki
* Tuna
* Turkey Breast
* Veggie
>
```

## <a name="sample-code"></a>Exemple de code

[!INCLUDE [Sample code](../includes/snippet-dotnet-formflow-samples.md)]

## <a name="additional-resources"></a>Ressources supplémentaires

- [Fonctionnalités de base de FormFlow](bot-builder-dotnet-formflow.md)
- [Fonctionnalités avancées de FormFlow](bot-builder-dotnet-formflow-advanced.md)
- [Personnaliser un formulaire à l’aide de FormBuilder](bot-builder-dotnet-formflow-formbuilder.md)
- [Localiser le contenu d’un formulaire](bot-builder-dotnet-formflow-localize.md)
- [Définir un formulaire à l’aide d’un schéma JSON](bot-builder-dotnet-formflow-json-schema.md)
- <a href="/dotnet/api/?view=botbuilder-3.11.0" target="_blank">Référence sur le Kit SDK Bot Builder pour .NET</a>

[promptAttribute]: /dotnet/api/microsoft.bot.builder.formflow.promptattribute

[field]: /dotnet/api/microsoft.bot.builder.formflow.iformbuilder-1.field

[formConfiguration]: /dotnet/api/microsoft.bot.builder.formflow.formconfiguration

[separator]: /dotnet/api/microsoft.bot.builder.formflow.advanced.templatebaseattribute.separator

[lastSeparator]: /dotnet/api/microsoft.bot.builder.formflow.advanced.templatebaseattribute.lastseparator

[templateUsage]: /dotnet/api/microsoft.bot.builder.formflow.templateusage

[caseNormalization]: /dotnet/api/microsoft.bot.builder.formflow.casenormalization

[choiceStyleOptions]: /dotnet/api/microsoft.bot.builder.formflow.choicestyleoptions

[feedbackOptions]: /dotnet/api/microsoft.bot.builder.formflow.feedbackoptions
