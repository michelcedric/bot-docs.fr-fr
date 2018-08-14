---
title: Exécution d’un bot du Kit de développement logiciel (SDK) .NET V3 dans le Kit SDK V4 | Microsoft Docs
description: Découvrez comment convertir un bot 3.x en bot 4.0 à l’aide du package NuGet classique.
keywords: migration, bot classique, convertir v3, v3 vers v4
author: v-royhar
ms.author: v-royhar
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 4/25/18
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: a808076e4865a181802b85cfc24ce342dbf23cba
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39299013"
---
# <a name="how-to-run-net-sdk-v3-bots-in-sdk-40"></a>Exécution de bots du Kit de développement logiciel (SDK) .NET V3 dans le Kit SDK 4.0

Le package NuGet **Microsoft.Bot.Builder.Classic** facilite la migration de vos bots de la version 3.x vers la version 4.0 de Microsoft Bot Framework.

**REMARQUE :** ce processus fonctionne uniquement pour les bots **Application web ASP.NET (.NET Framework)**. Il ne fonctionne pas sur les bots **Application web ASP.NET Core**.

## <a name="the-process"></a>Processus

Le processus est relativement simple :

- Ajoutez le package NuGet **Microsoft.Bot.Builder.Classic** au projet.
    - Vous devrez peut-être également mettre à jour le package NuGet **Autofac**.
- Mettez à jour les espaces de noms **Microsoft.Bot.Builder.Classic**.
- Appelez les dialogues à l’aide de la méthode **Conversation.SendAsync()** basée sur **ITurnContext** 4.0.

### <a name="add-the-microsoftbotbuilderclassic-nuget-package"></a>Ajouter le package NuGet Microsoft.Bot.Builder.Classic

Pour ajouter le package NuGet **Microsoft.Bot.Builder.Classic**, vous utilisez **Gérer les packages NuGet**.

### <a name="update-the-namespaces"></a>Mettre à jour les espaces de noms

Pour mettre à jour les espaces de noms, supprimez toutes les instructions `using` ci-après que vous trouverez :

```csharp
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Bot.Builder.Dialogs.Internals;
using Microsoft.Bot.Builder.FormFlow;
using Microsoft.Bot.Builder.Scorables;
```

Puis ajoutez les instructions `using` ci-après à la place :

```csharp
using Microsoft.Bot.Builder.Adapters;
using Microsoft.Bot.Builder.Classic.Dialogs;
using Microsoft.Bot.Builder.Classic.Dialogs.Internals;
using Microsoft.Bot.Builder.Classic.FormFlow;
using Microsoft.Bot.Builder.Classic.Scorables;
using Microsoft.Bot.Connector;
using Microsoft.Bot.Schema;
```

S’il existe une ligne `[BotAuthentication]`, supprimez-la ou mettez-la en commentaire.

### <a name="invoke-your-3x-dialog"></a>Appeler votre dialogue 3.x

Pour appeler votre dialogue 3.x, vous utilisez encore la méthode `Conversation.SendAsync`, à la seule différence près qu’elle utilise désormais un objet **ITurnContext** 4.0 au lieu d’un objet **Activity**.

```csharp
// invoke a Classic V3 IDialog 
await Conversation.SendAsync(turnContext, () => new EchoDialog());
```

Si vous ne disposez pas de l’objet **ITurnContext**, mais uniquement d’un objet **Activity**, vous pouvez obtenir l’objet **ITurnContext** à l’aide du code suivant :

```csharp
BotFrameworkAdapter adapter = new BotFrameworkAdapter("", "");

await adapter.ProcessActivity(this.Request.Headers.Authorization?.Parameter,
        activity,
        async (context) =>
        {
            // Do something with context here. For example, the body of your Post() method may go here.
        });
```

## <a name="fix-assembly-conflicts"></a>Résoudre les conflits d’assembly

Les bots créés avec le package NuGet classique présenteront des conflit avec leurs assemblys. Ces derniers apparaissent dans la fenêtre Liste d’erreurs dans Visual Studio après une génération.

### <a name="if-you-see-warning-found-conflicts-between-different-versions-of-the-same-dependent-assembly"></a>Si vous obtenez le message « Des conflits entre différentes versions du même assembly dépendant ont été rencontrés »

Si vous recevez un message d’avertissement commençant par le texte suivant : **Des conflits entre différentes versions du même assembly dépendant ont été rencontrés** :

- Double-cliquez sur le message d’avertissement. Une boîte de dialogue vous pose alors la question suivante : « Voulez-vous corriger ces conflits en ajoutant des enregistrements de redirection de liaison dans le fichier de configuration de l’application ? »
- Cliquez sur « Oui ».
- Régénérez le projet.

### <a name="if-you-see-error-missing-method-exception-on-startup"></a>Si vous obtenez le message « Error : Missing Method Exception on startup » (Erreur : exception de méthode manquante au démarrage)

Il existe un bogue avec .NET Standard, qui semble se produire lorsque vous mettez à niveau un ancien projet .NET 4.6 vers la version 4.6.1, puis que vous tentez d’utiliser une bibliothèque .NET standard avec ce projet. En fait, 2 assemblys System.Net.Http différents tentent de permuter dynamiquement. Pour contourner ce problème, vous pouvez ajouter une redirection de liaison à votre fichier Web.config pour System.Net.Http. 

Si vous recevez ce message d’erreur, ajoutez le code ci-après à votre fichier Web.config :

```xml
<dependentAssembly>
    <assemblyIdentity name="System.Net.Http" publicKeyToken="B03F5F7F11D50A3A" culture="neutral" />
    <bindingRedirect oldVersion="0.0.0.0-4.2.0.0" newVersion="4.2.0.0" />
</dependentAssembly>
```

Pour plus d’informations sur ce problème, consultez l’article [System.Net.Http v4.2.0.0 being copied/loaded from MSBuild tooling #25773](https://github.com/dotnet/corefx/issues/25773) (System.Net.Http v4.2.0.0 copié/chargé à partir des outils MSBuild - N° 25773).

## <a name="sample-of-a-converted-bot"></a>Exemple de bot converti

Dans le cas d’un bot déjà converti, vous pouvez consulter l’exemple [EchoBot-Classic](https://github.com/Microsoft/botbuilder-dotnet/tree/master/samples/Microsoft.Bot.Samples.EchoBot-Classic) qui présente un bot 3.x converti pour fonctionner avec 4.0.

## <a name="limitations"></a>Limites
Le package Microsoft.Bot.Builder.Classic est une bibliothèque .NET 4.61 uniquement ; il ne fonctionne pas sur .NET Core.
