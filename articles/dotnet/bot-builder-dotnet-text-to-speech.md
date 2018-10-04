---
title: Ajouter des fonctionnalités vocales aux messages | Microsoft Docs
description: Découvrez comment ajouter du discours aux messages à l’aide du Kit SDK Bot Builder pour .NET.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 55663bb493808d5efce2f25699f9df5aca4db968
ms.sourcegitcommit: d4afc924b0e1907c4d6f7a6fc5ac1fe521aeef7e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/28/2018
ms.locfileid: "47447334"
---
# <a name="add-speech-to-messages"></a>Ajouter de la reconnaissance vocale aux messages

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-text-to-speech.md)
> - [Node.JS](../nodejs/bot-builder-nodejs-text-to-speech.md)
> - [REST](../rest-api/bot-framework-rest-connector-text-to-speech.md)

Si vous générez un bot pour un canal où sont activées les fonctionnalités vocales, tel que Cortana, vous pouvez créer des messages qui spécifient le texte que doit énoncer votre bot. Vous pouvez également tenter d’influencer l’état du microphone du client en spécifiant un [conseil de saisie](bot-builder-dotnet-add-input-hints.md) pour indiquer si votre bot accepte, attend ou ignore l’entrée de l’utilisateur.

## <a name="specify-text-to-be-spoken-by-your-bot"></a>Spécifier le texte à énoncer par votre bot

Utilisez le Kit SDK Bot Builder pour .NET. Il existe plusieurs manières de spécifier le texte à énoncer par votre bot sur un canal à fonctionnalités vocales. Vous pouvez définir la propriété `Speak` du [message][IMessageActivity], appeler la méthode `IDialogContext.SayAsync()`, ou spécifier les options d’invite `speak` et `retrySpeak` lors de l’envoi d’un message à l’aide d’une invite intégrée.

### <a id="message-speak"></a> IMessageActivity.Speak

Si vous créez un [message] [ IMessageActivity] et définissez ses propriétés individuelles, vous pouvez définir la propriété `Speak` de ce message pour spécifier le texte à énoncer par votre bot. L’exemple de code suivant crée un message qui spécifie le texte à afficher et le texte à énoncer et indique que le bot [accepte l’entrée utilisateur](bot-builder-dotnet-add-input-hints.md).

[!code-csharp[Set speak property](../includes/code/dotnet-text-to-speech.cs#Speak1)]

### <a id="say-async"></a> IDialogContext.SayAsync()

Si vous utilisez des [dialogues](bot-builder-dotnet-dialogs.md), vous pouvez appeler la méthode `SayAsync()` pour créer et envoyer un message qui spécifie le texte à énoncer, en plus du texte à afficher et d’autres options. L’exemple de code suivant crée un message qui spécifie le texte à afficher et le texte à énoncer.

[!code-csharp[Call SayAsync()](../includes/code/dotnet-text-to-speech.cs#Speak2)]

### <a id="prompt-options"></a> Options d’invite

En utilisant l’une des invites intégrées, vous pouvez définir les options `speak` et `retrySpeak` afin de spécifier le texte à énoncer par votre bot. L’exemple de code suivant crée une invite qui spécifie le texte à afficher, le texte à énoncer initialement et le texte à énoncer après avoir attendu l’entrée utilisateur un certain temps. Elle utilise le format [SSML](#ssml) pour indiquer que l’expression « bien sûr » doit être prononcée en y mettant l’accent.

[!code-csharp[Set Prompt options](../includes/code/dotnet-text-to-speech.cs#Speak3)]

## <a id="ssml"></a> Speech Synthesis Markup Language (SSML)

Pour spécifier le texte à énoncer par votre bot, vous pouvez utiliser une chaîne de texte brut ou une chaîne mise en forme selon Speech Synthesis Markup Language (SSML), un langage de balisage basé sur XML qui vous permet de contrôler diverses caractéristiques du discours de votre bot telles que la voix, le débit, le volume, la prononciation, la hauteur et bien plus encore. Pour plus d’informations sur SSML, consultez <a href="https://msdn.microsoft.com/en-us/library/hh378377(v=office.14).aspx" target="_blank">Speech Synthesis Markup Language Reference</a> (Références sur Speech Synthesis Markup Language).

## <a name="input-hints"></a>Conseils de saisie

Lorsque vous envoyez un message sur le canal de reconnaissance vocale, vous pouvez tenter d’influencer l’état du microphone du client en spécifiant un conseil de saisie pour indiquer si votre bot accepte, attend ou ignore l’entrée de l’utilisateur. Pour plus d’informations, consultez [Ajouter des conseils de saisie aux messages](bot-builder-dotnet-add-input-hints.md).

## <a name="sample-code"></a>Exemple de code 

Pour obtenir un exemple complet qui montre comment créer un bot à fonctionnalité vocale en utilisant le Kit SDK Bot Builder pour .NET, consultez l’<a href="https://github.com/Microsoft/BotBuilder-Samples/tree/v3-sdk-samples/CSharp" target="_blank">exemple de compétence Roller</a> dans GitHub.

## <a name="additional-resources"></a>Ressources supplémentaires

- [Créer des messages](bot-builder-dotnet-create-messages.md)
- [Ajouter des conseils de saisie aux messages](bot-builder-dotnet-add-input-hints.md)
- <a href="https://msdn.microsoft.com/en-us/library/hh378377(v=office.14).aspx" target="_blank">Speech Synthesis Markup Language (SSML)</a>
- <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/demo-RollerSkill" target="_blank">Exemple de compétence Roller (GitHub)</a>
- <a href="https://docs.botframework.com/en-us/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html" target="_blank">Classe d’activité</a>
- <a href="/dotnet/api/microsoft.bot.connector.imessageactivity" target="_blank">Interface IMessageActivity</a>
- <a href="/dotnet/api/microsoft.bot.builder.dialogs.internals.dialogcontext" target="_blank">Classe DialogContext</a>
- <a href="/dotnet/api/microsoft.bot.builder.dialogs.internals.prompt-2" target="_blank">Classe d’invite</a>

[IMessageActivity]: /dotnet/api/microsoft.bot.connector.imessageactivity

