---
title: Envoyer des messages proactifs | Microsoft Docs
description: Découvrez comment des messages proactifs à l’aide du kit de développement logiciel (SDK) Bot Builder pour .NET.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: f5cc520704c4bebef6250195fd3e6cbdc3b0be3f
ms.sourcegitcommit: 67445b42796d90661afc643c6bb6533e9a662cbc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/06/2018
ms.locfileid: "39574955"
---
# <a name="send-proactive-messages"></a>Envoyer des messages proactifs

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-proactive-messages.md)
> - [Node.JS](../nodejs/bot-builder-nodejs-proactive-messages.md)

[!INCLUDE [Introduction to proactive messages - part 1](../includes/snippet-proactive-messages-intro-1.md)]

## <a name="types-of-proactive-messages"></a>Types de messages proactifs 

[!INCLUDE [Introduction to proactive messages - part 2](../includes/snippet-proactive-messages-intro-2.md)]

## <a name="send-an-ad-hoc-proactive-message"></a>Envoyer un message proactif ad hoc

Les exemples de code suivants montrent comment envoyer un message proactif ad hoc à l’aide du Kit de développement logiciel (SDK) Bot Builder pour .NET.

Pour pouvoir envoyer un message ad hoc à un utilisateur, le robot doit d’abord collecter et stocker des informations sur l’utilisateur à partir de la conversation en cours. 

```cs
public virtual async Task MessageReceivedAsync(IDialogContext context, IAwaitable<IMessageActivity> result)
{
    var message = await result;
    
    // Extract data from the user's message that the bot will need later to send an ad hoc message to the user. 
    // Store the extracted data in a custom class "ConversationStarter" (not shown here).

    ConversationStarter.toId = message.From.Id;
    ConversationStarter.toName = message.From.Name;
    ConversationStarter.fromId = message.Recipient.Id;
    ConversationStarter.fromName = message.Recipient.Name;
    ConversationStarter.serviceUrl = message.ServiceUrl;
    ConversationStarter.channelId = message.ChannelId;
    ConversationStarter.conversationId = message.Conversation.Id;

    // (Save this information somewhere that it can be accessed later, such as in a database.)

    await context.PostAsync("Hello user, good to meet you! I now know your address and can send you notifications in the future.");
    context.Wait(MessageReceivedAsync);
}
```
> [!NOTE]
> Par souci de simplicité, cet exemple ne spécifie pas comment stocker les données utilisateur. La manière dont les données sont stockées est sans importance pour autant que le robot puisse les récupérer ultérieurement.

À présent que les données ont été stockées, le robot peut simplement les récupérer, créer le message proactif ad hoc et l’envoyer. 

```cs
// Use the data stored previously to create the required objects.
var userAccount = new ChannelAccount(toId,toName);
var botAccount = new ChannelAccount(fromId, fromName);
var connector = new ConnectorClient(new Uri(serviceUrl));

// Create a new message.
IMessageActivity message = Activity.CreateMessageActivity();
if (!string.IsNullOrEmpty(conversationId) && !string.IsNullOrEmpty(channelId))  
{
    // If conversation ID and channel ID was stored previously, use it.
    message.ChannelId = channelId;
}
else
{
    // Conversation ID was not stored previously, so create a conversation. 
    // Note: If the user has an existing conversation in a channel, this will likely create a new conversation window.
    conversationId = (await connector.Conversations.CreateDirectConversationAsync( botAccount, userAccount)).Id;
}

// Set the address-related properties in the message and send the message.
message.From = botAccount;
message.Recipient = userAccount;
message.Conversation = new ConversationAccount(id: conversationId);
message.Text = "Hello, this is a notification";
message.Locale = "en-us";
await connector.Conversations.SendToConversationAsync((Activity)message);
```

> [!NOTE]
> Si le bot spécifie un ID de conversation stocké précédemment, le message sera probablement remis à l’utilisateur dans la fenêtre de conversation existante sur client. Si le robot génère un nouvel identifiant de conversation, le message sera remis à l’utilisateur dans une nouvelle fenêtre de conversation sur le client, à condition que le client prenne en charge plusieurs fenêtres de conversation. 

## <a name="send-a-dialog-based-proactive-message"></a>Envoyer un message proactif basé sur un dialogue

Les exemples de code suivants montrent comment envoyer un message proactif basé sur un dialogue à l’aide du Kit de développement logiciel (SDK) Bot Builder pour .NET.

Pour pouvoir envoyer un message proactif basé sur un dialogue à un utilisateur, le robot doit d’abord collecter et enregistrer les informations de la conversation en cours. 

```cs
public virtual async Task MessageReceivedAsync(IDialogContext context, IAwaitable<IMessageActivity> result)
{
    var message = await result;
    
    // Store information about this specific point the conversation, so that the bot can resume this conversation later.
    var conversationReference = message.ToConversationReference();
    ConversationStarter.conversationReference = JsonConvert.SerializeObject(conversationReference);

    await context.PostAsync("Greetings, user! I now know how to start a proactive message to you."); 
    context.Wait(MessageReceivedAsync);
}
```

Quand il est temps d’envoyer le message, le robot crée un dialogue et l’ajoute en haut de la pile de dialogues. Le nouveau dialogue prend le contrôle de la conversation, délivre le message proactif, se ferme, puis rend le contrôle au dialogue précédent dans la pile. 

```cs
// This will interrupt the conversation and send the user to SurveyDialog, then wait until that's done 
public static async Task Resume() 
{
    // Recreate the message from the conversation reference that was saved previously.
    var message = JsonConvert.DeserializeObject<ConversationReference>(conversationReference).GetPostToBotMessage(); 
    var client = new ConnectorClient(new Uri(message.ServiceUrl));

    // Create a scope that can be used to work with state from bot framework.
    using (var scope = DialogModule.BeginLifetimeScope(Conversation.Container, message))
    {
        var botData = scope.Resolve<IBotData>();
        await botData.LoadAsync(CancellationToken.None);

        // This is our dialog stack.
        var task = scope.Resolve<IDialogTask>();

        // Create the new dialog and add it to the stack.
        var dialog = new SurveyDialog();
        // interrupt the stack. This means that we're stopping whatever conversation that is currently happening with the user
        // Then adding this stack to run and once it's finished, we will be back to the original conversation
        task.Call(dialog.Void<object, IMessageActivity>(), null);
        
        await task.PollAsync(CancellationToken.None);

        // Flush the dialog stack back to its state store.
        await botData.FlushAsync(CancellationToken.None);        
    }
}
```
Le `SurveyDialog` contrôle la conversation jusqu’à la fin. Une fois sa tâche terminée, il appelle `context.Done` et se ferme, en rendant le contrôle au dialogue précédent. 

```cs
[Serializable]
public class SurveyDialog : IDialog<object>
{
    public async Task StartAsync(IDialogContext context)
    {
        await context.PostAsync("Hello, I'm the survey dialog. I'm interrupting your conversation to ask you a question. Type \"done\" to resume");

        context.Wait(this.MessageReceivedAsync);
    }
    public virtual async Task MessageReceivedAsync(IDialogContext context, IAwaitable<IMessageActivity> result)
    {
        if ((await result).Text == "done")
        {
            await context.PostAsync("Great, back to the original conversation!");
            context.Done(String.Empty); // Finish this dialog.
        }
        else
        {
            await context.PostAsync("I'm still on the survey until you type \"done\"");
            context.Wait(MessageReceivedAsync); // Not done yet.
        }
    }
}
```

## <a name="sample-code"></a>Exemple de code

Pour obtenir un exemple complet montrant comment envoyer des messages proactifs à l’aide du Kit de développement logiciel (SDK) Bot Builder pour .NET, voir les <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-proactiveMessages" target="_blank">exemples de messages proactifs</a> dans GitHub. Dans les exemples de messages proactifs, <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-proactiveMessages/simpleSendMessage" target="_blank">simpleSendMessage</a> montre comment envoyer un message proactif ad hoc, et <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-proactiveMessages/startNewDialog" target="_blank"> startNewDialog </a> comment envoyer un message proactif basé sur un dialogue. 

## <a name="additional-resources"></a>Ressources supplémentaires

- [Concevoir et contrôler un flux de conversation](../bot-service-design-conversation-flow.md)
- <a href="/dotnet/api/?view=botbuilder-3.11.0" target="_blank">Documentation de référence concernant le Kit de développement logiciel (SDK) Bot Builder pour .NET</a>
- <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-proactiveMessages" target="_blank">Exemples de messages proactifs (GitHub)</a>

