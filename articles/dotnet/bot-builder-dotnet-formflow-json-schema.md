---
title: Définir un formulaire à l’aide du schéma JSON et de FormFlow | Microsoft Docs
description: Découvrez comment définir un formulaire à l’aide du schéma JSON et de FormFlow en utilisant le Kit SDK Bot Builder pour .NET.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 8bcc957dbe2d69790cdfa7c2d7c377ed28b5fa12
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2018
ms.locfileid: "50000326"
---
# <a name="define-a-form-using-json-schema"></a>Définir un formulaire à l’aide d’un schéma JSON

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

Si vous utilisez une [classe C#](bot-builder-dotnet-formflow.md#create-class) pour définir le formulaire lorsque vous créez un bot avec FormFlow, le formulaire extrait la définition statique de votre type en C#. Vous pouvez aussi définir le formulaire à l’aide du <a href="http://json-schema.org/documentation.html" target="_blank">schéma JSON</a>. Un formulaire défini à l’aide du schéma JSON est purement orienté données ; vous pouvez modifier le formulaire (et par conséquent, le comportement du bot) simplement en mettant à jour le schéma. 

Le schéma JSON décrit les champs de votre <a href="http://www.newtonsoft.com/json/help/html/T_Newtonsoft_Json_Linq_JObject.htm" target="_blank">JObject</a> et inclut des annotations qui contrôlent les invites, modèles et termes. Pour utiliser le schéma JSON avec FormFlow, vous devez ajouter le package NuGet `Microsoft.Bot.Builder.FormFlow.Json` à votre projet et importer l’espace de noms `Microsoft.Bot.Builder.FormFlow.Json`.

## <a name="standard-keywords"></a>Mots clés standard 

FormFlow prend en charge les mots clés de <a href="http://json-schema.org/documentation.html" target="_blank">schéma JSON</a> standard suivants :

| Mot clé | Description | 
|----|----|
| Type | Définit le type de données que contient le champ. |
| enum | Définit les valeurs valides pour le champ. |
| minimum | Définit la valeur numérique minimale autorisée pour le champ (comme décrit dans [NumericAttribute][numericAttribute]). |
| maximum | Définit la valeur numérique maximale autorisée pour le champ (comme décrit dans [NumericAttribute][numericAttribute]). |
| required | Définit les champs obligatoires. |
| modèle | Valide les valeurs de chaîne (comme décrit dans [PatternAttribute][patternAttribute]). |

## <a name="extensions-to-json-schema"></a>Extensions au schéma JSON

FormFlow étend le <a href="http://json-schema.org/documentation.html" target="_blank">schéma JSON</a> standard pour prendre en charge plusieurs autres propriétés.

### <a name="additional-properties-at-the-root-of-the-schema"></a>Propriétés supplémentaires à la racine du schéma

| Propriété | Valeur |
|----|----|
| OnCompletion | Script C# avec des arguments `(IDialogContext context, JObject state)` pour compléter le formulaire. |
| Références | Références à inclure dans les scripts. Par exemple : `[assemblyReference, ...]`. Les chemins d’accès doivent être absolus ou relatifs au répertoire actuel. Par défaut, le script inclut `Microsoft.Bot.Builder.dll`. |
| Importations | Importations à inclure dans les scripts. Par exemple : `[import, ...]`. Par défaut, le script inclut les espaces de noms `Microsoft.Bot.Builder`, `Microsoft.Bot.Builder.Dialogs`, `Microsoft.Bot.Builder.FormFlow`, `Microsoft.Bot.Builder.FormFlow.Advanced`, `System.Collections.Generic` et `System.Linq`. |

### <a name="additional-properties-at-the-root-of-the-schema-or-as-peers-of-the-type-property"></a>Propriétés supplémentaires à la racine du schéma, ou en tant qu’homologues de la propriété de type

| Propriété | Valeur |
|----|----|
| Modèles | `{ TemplateUsage: { Patterns: [string, ...], <args> }, ...}` |
| Prompt | `{ Patterns:[string, ...] <args>}` |

Pour spécifier des modèles et des invites dans le schéma JSON, utilisez le même vocabulaire tel que défini par [TemplateAttribute] [ templateAttribute] et [PromptAttribute] [ promptAttribute]. Les noms et valeurs du schéma doivent correspondre aux noms et valeurs de la propriété dans l’énumération C# sous-jacente. Par exemple, cet extrait de schéma définit un modèle qui remplace le modèle `TemplateUsage.NotUnderstood` et spécifie un `TemplateBaseAttribute.ChoiceStyle` : 

```json
"Templates":{ "NotUnderstood": { "Patterns": ["I don't get it"], "ChoiceStyle":"Auto"}}
```

### <a name="additional-properties-as-peers-of-the-type-property"></a>Propriétés supplémentaires en tant qu’homologues de la propriété de type

|   Propriété   |          Sommaire           |                                                   Description                                                    |
|--------------|-----------------------------|------------------------------------------------------------------------------------------------------------------|
|   Datetime   |            bool             |                                  Indique si le champ est un champ `DateTime`.                                  |
|   Describe   |      chaîne ou objet       |                  Description d’un champ, comme décrit dans [DescribeAttribute][describeAttribute].                  |
|    Termes     |       `[string,...]`        |                  Expressions régulières pour faire correspondre une valeur de champ comme décrit dans TermsAttribute.                  |
|  MaxPhrase   |             int             |                  Exécute les termes via `Language.GenerateTerms(string, int)` pour les développer.                   |
|    Valeurs    | `{ string: {Describe:string |                                  object, Terms:[string, ...], MaxPhrase}, ...}`                                  |
|    Actif    |           script            | Script C# avec des arguments `(JObject state)->bool` pour tester l’activité du champ, du message ou de la confirmation.  |
|   Valider   |           script            |      Script C# avec des arguments `(JObject state, object value)->ValidateResult` pour valider une valeur de champ.      |
|    Define    |           script            |        Script C# avec des arguments `(JObject state, Field<JObject> field)` pour définir de façon dynamique un champ.        |
|     Suivant     |           script            | Script C# avec des arguments `(object value, JObject state)` pour déterminer l’étape suivante après avoir rempli un champ. |
|    Avant    |          `[confirm          |                                                  message, ...]`                                                  |
|    Après     |          `[confirm          |                                                  message, ...]`                                                  |
| Les dépendances |        [string, ...]        |                           Champs dont dépend ce champ, ce message ou cette confirmation.                           |

Utilisez `{Confirm:script|[string, ...], ...templateArgs}` au sein de la valeur de la propriété **Before** ou **After** pour définir une confirmation à l’aide d’un script C# avec l’argument `(JObject state)` ou un ensemble de modèles qui sera sélectionné au hasard avec des arguments de modèle facultatifs.

Utilisez `{Message:script|[string, ...] ...templateArgs}` au sein de la valeur de la propriété **Before** ou **After** pour définir un message à l’aide d’un script C# avec l’argument `(JObject state)` ou un ensemble de modèles qui sera sélectionné au hasard avec des arguments de modèle facultatifs.

## <a name="scripts"></a>Scripts

Plusieurs des propriétés décrites ci-dessus contiennent un script en tant que valeur de propriété. Un script peut représenter tout extrait de code C# que vous trouveriez normalement dans le corps d’une méthode. Vous pouvez ajouter des références à l’aide de la propriété **References** et/ou de la propriété **Imports**. Les variables globales spéciales sont les suivantes :

| Variable | Description |
|----|----|
| choice | Distribution interne du script à exécuter. |
| state | État du formulaire `JObject` assigné à tous les scripts. |
| ifield | `IField<JObject>` pour autoriser le raisonnement sur le champ actuel pour tous les scripts à l’exception des générateurs d’invite de message/confirmation. |
| value | Valeur d’objet à valider pour **Validate**. |
| field | `Field<JObject>` pour autoriser la mise à jour dynamique d’un champ dans **Define**. |
| context | Contexte `IDialogContext` pour permettre la publication des résultats dans **OnCompletion**. |

Les champs définis par le biais d’un schéma JSON peuvent également étendre ou remplacer les définitions par programmation, comme n’importe quel autre champ. Ils peuvent également être localisés de la même façon.

## <a name="json-schema-example"></a>Exemple de schéma JSON

La façon la plus simple de définir un formulaire consiste à tout définir, y compris le code C#, directement dans le schéma JSON. Cet exemple montre le schéma JSON pour le bot sandwich annoté décrit dans la section [Personnaliser un formulaire à l’aide de FormBuilder](bot-builder-dotnet-formflow-formbuilder.md).

```json
{
  "References": [ "Microsoft.Bot.Sample.AnnotatedSandwichBot.dll" ],
  "Imports": [ "Microsoft.Bot.Sample.AnnotatedSandwichBot.Resource" ],
  "type": "object",
  "required": [
    "Sandwich",
    "Length",
    "Ingredients",
    "DeliveryAddress"
  ],
  "Templates": {
    "NotUnderstood": {
      "Patterns": [ "I do not understand \"{0}\".", "Try again, I don't get \"{0}\"." ]
    },
    "EnumSelectOne": {
      "Patterns": [ "What kind of {&} would you like on your sandwich? {||}" ],
      "ChoiceStyle": "Auto"
    }
  },
  "properties": {
    "Sandwich": {
      "Prompt": { "Patterns": [ "What kind of {&} would you like? {||}" ] },
      "Before": [ { "Message": [ "Welcome to the sandwich order bot!" ] } ],
      "Describe": { "Image": "https://placeholdit.imgix.net/~text?txtsize=16&txt=Sandwich&w=125&h=40&txttrack=0&txtclr=000&txtfont=bold" },
      "type": [
        "string",
        "null"
      ],
      "enum": [
        "BLT",
        "BlackForestHam",
        "BuffaloChicken",
        "ChickenAndBaconRanchMelt",
        "ColdCutCombo",
        "MeatballMarinara",
        "OvenRoastedChicken",
        "RoastBeef",
        "RotisserieStyleChicken",
        "SpicyItalian",
        "SteakAndCheese",
        "SweetOnionTeriyaki",
        "Tuna",
        "TurkeyBreast",
        "Veggie"
      ],
      "Values": {
        "RotisserieStyleChicken": {
          "Terms": [ "rotis\\w* style chicken" ],
          "MaxPhrase": 3
        }
      }
    },
    "Length": {
      "Prompt": {
        "Patterns": [ "What size of sandwich do you want? {||}" ]
      },
      "type": [
        "string",
        "null"
      ],
      "enum": [
        "SixInch",
        "FootLong"
      ]
    },
    "Ingredients": {
      "type": "object",
      "required": [ "Bread" ],
      "properties": {
        "Bread": {
          "type": [
            "string",
            "null"
          ],
          "Describe": {
            "Title": "Sandwich Bot",
            "SubTitle": "Bread Picker"
          },
          "enum": [
            "NineGrainWheat",
            "NineGrainHoneyOat",
            "Italian",
            "ItalianHerbsAndCheese",
            "Flatbread"
          ]
        },
        "Cheese": {
          "type": [
            "string",
            "null"
          ],
          "enum": [
            "American",
            "MontereyCheddar",
            "Pepperjack"
          ]
        },
        "Toppings": {
          "type": "array",
          "items": {
            "type": "integer",
            "enum": [
              "Everything",
              "Avocado",
              "BananaPeppers",
              "Cucumbers",
              "GreenBellPeppers",
              "Jalapenos",
              "Lettuce",
              "Olives",
              "Pickles",
              "RedOnion",
              "Spinach",
              "Tomatoes"
            ],
            "Values": {
              "Everything": { "Terms": [ "except", "but", "not", "no", "all", "everything" ] }
            }
          },
          "Validate": "var values = ((List<object>) value).OfType<string>(); var result = new ValidateResult {IsValid = true, Value = values} ; if (values != null && values.Contains(\"Everything\")) { result.Value = (from topping in new string[] {  \"Avocado\", \"BananaPeppers\", \"Cucumbers\", \"GreenBellPeppers\", \"Jalapenos\", \"Lettuce\", \"Olives\", \"Pickles\", \"RedOnion\", \"Spinach\", \"Tomatoes\"} where !values.Contains(topping) select topping).ToList();} return result;",
          "After": [ { "Message": [ "For sandwich toppings you have selected {Ingredients.Toppings}." ] } ]
        },
        "Sauces": {
          "type": [
            "array",
            "null"
          ],
          "items": {
            "type": "string",
            "enum": [
              "ChipotleSouthwest",
              "HoneyMustard",
              "LightMayonnaise",
              "RegularMayonnaise",
              "Mustard",
              "Oil",
              "Pepper",
              "Ranch",
              "SweetOnion",
              "Vinegar"
            ]
          }
        }
      }
    },
    "Specials": {
      "Templates": {
        "NoPreference": { "Patterns": [ "None" ] }
      },
      "type": [
        "string",
        "null"
      ],
      "Active": "return (string) state[\"Length\"] == \"FootLong\";",
      "Define": "field.SetType(null).AddDescription(\"cookie\", DynamicSandwich.FreeCookie).AddTerms(\"cookie\", Language.GenerateTerms(DynamicSandwich.FreeCookie, 2)).AddDescription(\"drink\", DynamicSandwich.FreeDrink).AddTerms(\"drink\", Language.GenerateTerms(DynamicSandwich.FreeDrink, 2)); return true;",
      "After": [ { "Confirm": "var cost = 0.0; switch ((string) state[\"Length\"]) { case \"SixInch\": cost = 5.0; break; case \"FootLong\": cost=6.50; break;} return new PromptAttribute($\"Total for your sandwich is {cost:C2} is that ok?\");" } ]
    },
    "DeliveryAddress": {
      "type": [
        "string",
        "null"
      ],
      "Validate": "var result = new ValidateResult{ IsValid = true, Value = value}; var address = (value as string).Trim(); if (address.Length > 0 && (address[0] < '0' || address[0] > '9')) {result.Feedback = DynamicSandwich.BadAddress; result.IsValid = false; } return result;"
    },
    "PhoneNumber": {
      "type": [ "string", "null" ],
      "pattern": "(\\(\\d{3}\\))?\\s*\\d{3}(-|\\s*)\\d{4}"
    },
    "DeliveryTime": {
      "Templates": {
        "StatusFormat": {
          "Patterns": [ "{&}: {:t}" ],
          "FieldCase": "None"
        }
      },
      "DateTime": true,
      "type": [
        "string",
        "null"
      ],
      "After": [ { "Confirm": [ "Do you want to order your {Length} {Sandwich} on {Ingredients.Bread} {&Ingredients.Bread} with {[{Ingredients.Cheese} {Ingredients.Toppings} {Ingredients.Sauces} to be sent to {DeliveryAddress} {?at {DeliveryTime}}?" ] } ]
    },
    "Rating": {
      "Describe": "your experience today",
      "type": [
        "number",
        "null"
      ],
      "minimum": 1,
      "maximum": 5,
      "After": [ { "Message": [ "Thanks for ordering your sandwich!" ] } ]
    }
  },
  "OnCompletion": "await context.PostAsync(\"We are currently processing your sandwich. We will message you the status.\");"
}
```

## <a name="implement-formflow-with-json-schema"></a>Implémenter FormFlow avec un schéma JSON

Pour implémenter FormFlow avec un schéma JSON, utilisez `FormBuilderJson`, qui prend en charge la même interface Fluent que `FormBuilder`. Ce code montre comment implémenter le schéma JSON pour le bot sandwich annoté décrit dans la section [Personnaliser un formulaire à l’aide de FormBuilder](bot-builder-dotnet-formflow-formbuilder.md).

[!code-csharp[Use JSON schema](../includes/code/dotnet-formflow-json-schema.cs#useSchema)]

## <a name="sample-code"></a>Exemple de code

[!INCLUDE [Sample code](../includes/snippet-dotnet-formflow-samples.md)]

## <a name="additional-resources"></a>Ressources supplémentaires

- [Fonctionnalités de base de FormFlow](bot-builder-dotnet-formflow.md)
- [Fonctionnalités avancées de FormFlow](bot-builder-dotnet-formflow-advanced.md)
- [Personnaliser un formulaire à l’aide de FormBuilder](bot-builder-dotnet-formflow-formbuilder.md)
- [Localiser le contenu d’un formulaire](bot-builder-dotnet-formflow-localize.md)
- [Personnaliser l’expérience utilisateur avec le langage du modèle](bot-builder-dotnet-formflow-pattern-language.md)
- <a href="/dotnet/api/?view=botbuilder-3.11.0" target="_blank">Référence sur le Kit SDK Bot Builder pour .NET</a>

[numericAttribute]: /dotnet/api/microsoft.bot.builder.formflow.numericattribute

[patternAttribute]: /dotnet/api/microsoft.bot.builder.formflow.patternattribute

[templateAttribute]: /dotnet/api/microsoft.bot.builder.formflow.templateattribute

[promptAttribute]: /dotnet/api/microsoft.bot.builder.formflow.promptattribute

[describeAttribute]: /dotnet/api/microsoft.bot.builder.formflow.describeattribute
