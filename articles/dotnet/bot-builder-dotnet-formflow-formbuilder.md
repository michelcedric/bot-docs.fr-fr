---
title: Personnaliser un formulaire à l’aide de FormBuilder | Microsoft Docs
description: Découvrez comment modifier de façon dynamique et personnaliser le flux de la conversation et le contenu à l’aide de FormBuilder pour le Kit SDK Bot Builder pour .NET.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 1c4e60f76ecebfa01664500b8343d60ccff0064c
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2018
ms.locfileid: "49997696"
---
# <a name="customize-a-form-using-formbuilder"></a>Personnaliser un formulaire à l’aide de FormBuilder

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

La section [Fonctionnalités de base de FormFlow](bot-builder-dotnet-formflow.md) décrit une implémentation FormFlow de base offrant une expérience utilisateur assez générique, et la section [Fonctionnalités avancées de FormFlow](bot-builder-dotnet-formflow-advanced.md) explique comment personnaliser l’expérience utilisateur à l’aide d’une logique métier et d’attributs. Cet article décrit comment utiliser [FormBuilder] [ formBuilder] pour personnaliser encore davantage l’expérience utilisateur en spécifiant la séquence dans laquelle le formulaire exécute les étapes et en définissant de façon dynamique les valeurs de champ, les confirmations et les messages. 

## <a name="dynamically-define-field-values-confirmations-and-messages"></a>Définir de façon dynamique des valeurs de champ, des confirmations et des messages

À l’aide de FormBuilder, vous pouvez définir de façon dynamique des valeurs de champ, des confirmations et des messages.

### <a name="dynamically-define-field-values"></a>Définir de façon dynamique des valeurs de champ 

Un bot sandwich conçu pour ajouter une boisson ou un cookie offert pour toute commande d’un grand sandwich utilise le champ `Sandwich.Specials` pour stocker les données relatives à ces produits offerts. Dans ce cas, la valeur du champ `Sandwich.Specials` doit être de façon dynamique définie pour chaque commande selon que la commande contient ou non un grand sandwich. 

Le champ `Specials` est spécifié comme étant facultatif et la valeur texte « None » (aucune) s’affiche comme choix, indiquant qu’il n’y a aucune préférence.

[!code-csharp[Field definition](../includes/code/dotnet-formflow-formbuilder.cs#fieldDefinition)]

Cet exemple de code montre comment définir de façon dynamique la valeur du champ `Specials`. 

[!code-csharp[Define value](../includes/code/dotnet-formflow-formbuilder.cs#defineValue)]

Dans cet exemple, la méthode [Advanced.Field.SetType][setType] spécifie le type de champ (`null` représente un champ d’énumération). La méthode [Advanced.Field.SetActive][setActive] spécifie que le champ doit uniquement être si la longueur du sandwich est `Length.FootLong`. Enfin, la méthode [Advanced.Field.SetDefine][setDefine] spécifie un délégué asynchrone qui définit le champ. Le délégué reçoit l’objet d’état actuel et la valeur [Advanced.Field][field] définie de façon dynamique. Le délégué utilise les méthodes Fluent du champ pour définir de façon dynamique les valeurs. Dans cet exemple, les valeurs sont des chaînes et les méthodes `AddDescription` et `AddTerms` spécifient les descriptions et les termes de chaque valeur.

> [!NOTE]
> Pour définir de façon dynamique une valeur de champ, vous pouvez implémenter [Advanced.IField][iField] vous-même, ou simplifier le processus à l’aide de la classe [Advanced.FieldReflector][FieldReflector] comme indiqué dans l’exemple ci-dessus. 

### <a name="dynamically-define-messages-and-confirmations"></a>Définir de façon dynamique des messages et des confirmations

À l’aide de FormBuilder, vous pouvez également définir de façon dynamique des messages et des confirmations. Chaque message et chaque confirmation s’exécutent uniquement lorsque les étapes précédentes du formulaire sont inactives ou terminées. 

Cet exemple de code montre une confirmation, générée de façon dynamique, qui calcule le coût du sandwich. 

[!code-csharp[Define confirmation](../includes/code/dotnet-formflow-formbuilder.cs#defineConfirmation)]

## <a name="customize-a-form-using-formbuilder"></a>Personnaliser un formulaire à l’aide de FormBuilder

Cet exemple de code utilise FormBuilder pour définir les étapes du formulaire, [valider les sélections](bot-builder-dotnet-formflow-advanced.md#add-business-logic), et [définir de façon dynamique une valeur de champ et une confirmation](#dynamically-define-field-values-confirmations-and-messages). Par défaut, les étapes décrites dans le formulaire seront exécutées dans l’ordre dans lequel elles sont répertoriées. Toutefois, des étapes peuvent être ignorées pour les champs contenant déjà des valeurs ou si aucune navigation précise n’est spécifiée. 

[!code-csharp[FormBuilder form](../includes/code/dotnet-formflow-formbuilder.cs#formBuilderForm)]

Dans cet exemple, le formulaire effectue ces étapes :

- Affiche un message de bienvenue. 
- Renseigne `SandwichOrder.Sandwich`. 
- Renseigne `SandwichOrder.Length`. 
- Renseigne `SandwichOrder.Bread`. 
- Renseigne `SandwichOrder.Cheese`. 
- Renseigne `SandwichOrder.Toppings` et ajoute les valeurs manquantes, si l’utilisateur a sélectionné `ToppingOptions.Everything`. -. Affiche un message qui confirme les ingrédients sélectionnés. 
- Renseigne `SandwichOrder.Sauces`. 
- [Définit de façon dynamique](#dynamically-define-field-values) la valeur de champ pour `SandwichOrder.Specials`. 
- [Définit de manière dynamique](#dynamically-define-messages-and-confirmations) la confirmation du coût du sandwich. 
- Renseigne `SandwichOrder.DeliveryAddress` et [vérifie](bot-builder-dotnet-formflow-advanced.md#add-business-logic) la chaîne résultante. Si l’adresse ne commence pas par un chiffre, le formulaire renvoie un message. 
- Renseigne `SandwichOrder.DeliveryTime` à l’aide d’une invite personnalisée. 
- Confirme la commande. 
- Ajoute tous les champs restants qui ont été définis dans la classe, mais pas explicitement référencés par `Field`. (Si l’exemple n’a pas appelé la méthode `AddRemainingFields`, le formulaire n’inclut pas tous les champs non explicitement référencés.) 
- Affiche un message de remerciement. 
- Définit un gestionnaire `OnCompletionAsync` pour traiter la commande. 

## <a name="sample-code"></a>Exemple de code

[!INCLUDE [Sample code](../includes/snippet-dotnet-formflow-samples.md)]

## <a name="additional-resources"></a>Ressources supplémentaires

- [Fonctionnalités de base de FormFlow](bot-builder-dotnet-formflow.md)
- [Fonctionnalités avancées de FormFlow](bot-builder-dotnet-formflow-advanced.md)
- [Localiser le contenu d’un formulaire](bot-builder-dotnet-formflow-localize.md)
- [Définir un formulaire à l’aide d’un schéma JSON](bot-builder-dotnet-formflow-json-schema.md)
- [Personnaliser l’expérience utilisateur avec le langage du modèle](bot-builder-dotnet-formflow-pattern-language.md)
- <a href="/dotnet/api/?view=botbuilder-3.11.0" target="_blank">Référence sur le Kit SDK Bot Builder pour .NET</a>

[formBuilder]: /dotnet/api/microsoft.bot.builder.formflow.formbuilder-1

[setType]: /dotnet/api/microsoft.bot.builder.formflow.advanced.field-1.settype

[setActive]: /dotnet/api/microsoft.bot.builder.formflow.advanced.field-1.setactive

[setDefine]: /dotnet/api/microsoft.bot.builder.formflow.advanced.field-1.setdefine

[field]: /dotnet/api/microsoft.bot.builder.formflow.advanced.field-1

[iField]: /dotnet/api/microsoft.bot.builder.formflow.advanced.ifield-1

[FieldReflector]: /dotnet/api/microsoft.bot.builder.formflow.advanced.fieldreflector-1
