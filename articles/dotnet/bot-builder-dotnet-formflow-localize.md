---
title: Localiser le contenu d’un formulaire | Microsoft Docs
description: Découvrez comment localiser le contenu d’un formulaire avec FormFlow et le Kit SDK Bot Builder pour .NET.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: bb0ac4b8e3fa34ec8863bb323ae968db37972a6f
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39299204"
---
# <a name="localize-form-content"></a>Localiser le contenu d’un formulaire

La langue de localisation d’un formulaire est déterminée par les valeurs [CurrentUICulture](https://msdn.microsoft.com/en-us/library/system.threading.thread.currentuiculture(v=vs.110).aspx) et [CurrentCulture](https://msdn.microsoft.com/en-us/library/system.threading.thread.currentculture(v=vs.110).aspx) du thread actuel. Par défaut, la culture est dérivée du champ des **paramètres régionaux** du message actuel, mais vous pouvez remplacer ce comportement par défaut. Selon la façon dont votre bot est construit, les informations localisées peuvent provenir de trois sources différentes :

- la localisation intégrée pour les valeurs **PromptDialog** et **FormFlow**
- un fichier de ressources que vous générez pour les chaînes statiques de votre formulaire
- un fichier de ressources que vous créez avec des chaînes pour les champs, messages ou confirmations définis de manière dynamique

## <a name="generate-a-resource-file-for-the-static-strings-in-your-form"></a>Générer un fichier de ressources pour les chaînes statiques dans votre formulaire

Les chaînes statiques d’un formulaire incluent les chaînes permettant de générer le formulaire à partir des informations de votre classe C#, et les chaînes que vous spécifiez en tant qu’invites, modèles, messages ou confirmations. Les chaînes générées à partir de modèles intégrés ne sont pas considérées comme des chaînes statiques, car ces chaînes sont déjà localisées. Comme la plupart des chaînes d’un formulaire sont automatiquement générées, vous ne pouvez pas utiliser directement des chaînes de ressources C# standard. Au lieu de cela, vous pouvez générer un fichier de ressources pour les chaînes statiques de votre formulaire en appelant `IFormBuilder.SaveResources` ou à l’aide de l’outil **RView** inclus avec le Kit SDK Bot Builder pour .NET.

### <a name="use-iformbuildersaveresources"></a>Utiliser IFormBuilder.SaveResources

Vous pouvez générer un fichier de ressources en appelant [IFormBuilder.SaveResources] [ saveResources] sur votre formulaire pour enregistrer les chaînes dans un fichier .resx.

### <a name="use-rview"></a>Utiliser RView

Vous pouvez également générer un fichier de ressources qui dépend de votre fichier .dll ou .exe à l’aide de l’outil <a href="https://github.com/Microsoft/BotBuilder/tree/master/CSharp/Tools/RView" target="_blank">RView</a> inclus dans le Kit SDK Bot Builder pour .NET. Pour générer le fichier .resx, exécutez **rview** et spécifiez l’assembly qui contient votre méthode de création de formulaires statiques ainsi que le chemin d’accès à cette méthode. Cet extrait de code montre comment générer le fichier de ressources `Microsoft.Bot.Sample.AnnotatedSandwichBot.SandwichOrder.resx` à l’aide de l’outil **RView**. 

```csharp
rview -g Microsoft.Bot.Sample.AnnotatedSandwichBot.dll Microsoft.Bot.Sample.AnnotatedSandwichBot.SandwichOrder.BuildForm
```

Cet extrait montre une partie du fichier .resx généré par l’exécution de cette commande **rview**.

```xml
<data name="Specials_description;VALUE" xml:space="preserve">
<value>Specials</value>
</data>
<data name="DeliveryAddress_description;VALUE" xml:space="preserve">
<value>Delivery Address</value>
</data>
<data name="DeliveryTime_description;VALUE" xml:space="preserve">
<value>Delivery Time</value>
</data>
<data name="PhoneNumber_description;VALUE" xml:space="preserve">
<value>Phone Number</value>
</data>
<data name="Rating_description;VALUE" xml:space="preserve">
<value>your experience today</value>
</data>
<data name="message0;LIST" xml:space="preserve">
<value>Welcome to the sandwich order bot!</value>
</data>
<data name="Sandwich_terms;LIST" xml:space="preserve">
<value>sandwichs?</value>
</data>
```

## <a name="configure-your-project"></a>Configurer votre projet

Une fois que vous avez généré un fichier de ressources, ajoutez-le à votre projet, puis définissez la langue neutre en procédant comme suit : 

1. Cliquez avec le bouton droit sur votre projet et sélectionnez **Application**.
2. Cliquez sur **Informations de l’assembly**.
3. Sélectionnez la valeur **Langue neutre** qui correspond à la langue dans laquelle vous avez développé votre bot.

Lors de la création de votre formulaire, la méthode [IFormBuilder.Build] [ build] recherchera automatiquement les ressources contenant le nom de votre type de formulaire et s’en servira pour localiser les chaînes statiques de votre formulaire. 

> [!NOTE]
> Les champs calculés de manière dynamique et définis à l’aide de [Advanced.Field.SetDefine] [ setDefine] (comme décrit dans [Utilisation des champs dynamiques](bot-builder-dotnet-formflow-formbuilder.md#dynamically-define-field-values-confirmations-and-messages)) ne peuvent pas être localisés de la même manière que les champs statiques, étant donné que les chaînes des champs calculés de manière dynamique sont construites au moment où le formulaire est rempli. Vous pouvez cependant localiser des champs calculés de façon dynamique à l’aide de mécanismes de localisation C# standard.

### <a name="localize-resource-files"></a>Localiser des fichiers de ressources 

Une fois que vous avez ajouté des fichiers de ressources à votre projet, vous pouvez les localiser à l’aide de l’outil <a href="https://developer.microsoft.com/en-us/windows/develop/multilingual-app-toolkit" target="_blank">Multilingual App Toolkit (MAT)</a>. Installez **MAT**, puis activez-le pour votre projet en procédant comme suit :

1. Sélectionnez votre projet dans l’Explorateur de solutions Visual Studio.
2. Cliquez sur **Outils**, **Multilingual App Toolkit**, puis **Activer**.
3. Cliquez avec le bouton droit sur le projet, puis sélectionnez **Multilingual App Toolkit**, **Ajouter des traductions** pour sélectionner les traductions. Vous créez ainsi des fichiers <a href="https://en.wikipedia.org/wiki/XLIFF" target="_blank">XLF</a> standard que vous pouvez traduire automatiquement ou manuellement.

> [!NOTE]
> Même si cet article décrit comment utiliser Multilingual App Toolkit pour localiser le contenu, vous pouvez implémenter la localisation via une variété d’autres moyens.

## <a name="see-it-in-action"></a>Voir en action

Cet exemple de code s’appuie sur celui de la section [Personnaliser un formulaire à l’aide de FormBuilder](bot-builder-dotnet-formflow-formbuilder.md) pour implémenter la localisation, comme décrit ci-dessus. Dans cet exemple, la classe `DynamicSandwich` (non illustrée ici) contient des informations de localisation pour les champs, messages et confirmations générés de manière dynamique.

[!code-csharp[Build localized form](../includes/code/dotnet-formflow-localize.cs#buildLocalizedForm)]

Cet extrait de code montre l’interaction générée entre le bot et l’utilisateur lorsque le paramètre `CurrentUICulture` est défini sur **Français**.

```console
Bienvenue sur le bot d'ordre "sandwich" !
Quel genre de "sandwich" vous souhaitez sur votre "sandwich"?
 1. BLT
 2. Jambon Forêt Noire
 3. Poulet Buffalo
 4. Faire fondre le poulet et Bacon Ranch
 5. Combo de coupe à froid
 6. Boulette de viande Marinara
 7. Poulet rôti au four
 8. Rôti de boeuf
 9. Rotisserie poulet
 10. Italienne piquante
 11. Bifteck et fromage
 12. Oignon doux Teriyaki
 13. Thon
 14. Poitrine de dinde
 15. Veggie
> 2

Quel genre de longueur vous souhaitez sur votre "sandwich"?
 1. Six pouces
 2. Pied Long
> ?
* Vous renseignez le champ longueur.Réponses possibles:
* Vous pouvez saisir un numéro 1-2 ou des mots de la description. (Six pouces, ou Pied Long)
* Retourner à la question précédente.
* Assistance: Montrez les réponses possibles.
* Abandonner: Abandonner sans finir
* Recommencer remplir le formulaire. (Vos réponses précédentes sont enregistrées.)
* Statut: Montrer le progrès en remplissant le formulaire jusqu'à présent.
* Vous pouvez passer à un autre champ en entrant son nom. ("Sandwich", Longueur, Pain, Fromage, Nappages, Sauces, Adresse de remise, Délai de livraison, ou votre expérience aujourd'hui).
Quel genre de longueur vous souhaitez sur votre "sandwich"?
 1. Six pouces
 2. Pied Long
> 1

Quel genre de pain vous souhaitez sur votre "sandwich"?
 1. Neuf grains de blé
 2. Neuf grains miel avoine
 3. Italien
 4. Fromage et herbes italiennes
 5. Pain plat
> neuf
Par pain "neuf" vouliez-vous dire (1. Neuf grains miel avoine, ou 2. Neuf grains de blé)
```

## <a name="sample-code"></a>Exemple de code

[!INCLUDE [Sample code](../includes/snippet-dotnet-formflow-samples.md)]

## <a name="additional-resources"></a>Ressources supplémentaires

- [Fonctionnalités de base de FormFlow](bot-builder-dotnet-formflow.md)
- [Fonctionnalités avancées de FormFlow](bot-builder-dotnet-formflow-advanced.md)
- [Personnaliser un formulaire à l’aide de FormBuilder](bot-builder-dotnet-formflow-formbuilder.md)
- [Définir un formulaire à l’aide d’un schéma JSON](bot-builder-dotnet-formflow-json-schema.md)
- [Personnaliser l’expérience utilisateur avec le langage du modèle](bot-builder-dotnet-formflow-pattern-language.md)
- <a href="/dotnet/api/?view=botbuilder-3.11.0" target="_blank">Référence sur le Kit SDK Bot Builder pour .NET</a>

[build]: /dotnet/api/microsoft.bot.builder.formflow.formbuilder-1.build 

[setDefine]: /dotnet/api/microsoft.bot.builder.formflow.advanced.field-1.setdefine

[saveResources]: /dotnet/api/microsoft.bot.builder.formflow.iform-1.saveresources
