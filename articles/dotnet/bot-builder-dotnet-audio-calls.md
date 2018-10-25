---
title: Passer des appels audio avec Skype | Microsoft Docs
description: Découvrez comment passer des appels audio avec Skype en utilisant le Kit de développement logiciel Bot Builder pour .NET.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 0d0489c23cd24a7323ba0160d5e8e5e914be3011
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2018
ms.locfileid: "49998848"
---
# <a name="conduct-audio-calls-with-skype"></a>Passer des appels audio avec Skype

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

[!INCLUDE [Introduction to conducting audio calls](../includes/snippet-audio-call-intro.md)]

L’architecture pour un bot qui prend en charge les appels audio est très similaire à celle d’un bot classique. Les exemples de code suivants montrent comment activer la prise en charge des appels audio via Skype avec le Kit de développement logiciel Bot Builder pour .NET. 

## <a name="enable-support-for-audio-calls"></a>Activer la prise en charge des appels audio

Pour activer un bot afin de prendre en charge les appels audio, définissez la classe `CallingController`.

```cs
[BotAuthentication]
[RoutePrefix("api/calling")]
public class CallingController : ApiController
{
    public CallingController() : base()
    {
        CallingConversation.RegisterCallingBot(callingBotService => new IVRBot(callingBotService));
    }

    [Route("callback")]
    public async Task<HttpResponseMessage> ProcessCallingEventAsync()
    {
        return await CallingConversation.SendAsync(this.Request, CallRequestType.CallingEvent);
    }

    [Route("call")]
    public async Task<HttpResponseMessage> ProcessIncomingCallAsync()
    {
        return await CallingConversation.SendAsync(this.Request, CallRequestType.IncomingCall);
    }
}
```

> [!NOTE]
> Outre la classe `CallingController` qui prend en charge les appels audio, un bot peut également contenir `MessagesController` pour prendre en charge les messages. En fournissant les deux options, les utilisateurs peuvent interagir avec le bot de la façon qu’ils préfèrent. <!-- docs on MessagesController are where? -->

##  <a name="answer-the-call"></a>Répondre à l’appel

La tâche `ProcessIncomingCallAsync` s’exécute chaque fois qu’un utilisateur lance un appel à ce bot de Skype.
Le constructeur enregistre la classe `IVRBot`, qui comprend un gestionnaire prédéfini pour l’évènement `incomingCallEvent`.

La première action dans le flux de travail doit déterminer si le bot répond à ou refuse l’appel entrant. Ce flux de travail indique au bot de répondre à l’appel entrant puis de lire un message d’accueil. 

```cs
private Task OnIncomingCallReceived(IncomingCallEvent incomingCallEvent)
{
    this.callStateMap[incomingCallEvent.IncomingCall.Id] = new CallState(incomingCallEvent.IncomingCall.Participants);

    incomingCallEvent.ResultingWorkflow.Actions = new List<ActionBase>
    {
        new Answer { OperationId = Guid.NewGuid().ToString() },
        GetPromptForText(WelcomeMessage)
    };

    return Task.FromResult(true);
}
```

## <a name="after-the-bot-answers"></a>Après que le bot a répondu

Si le bot répond à l’appel, les actions suivantes spécifiées dans le flux de travail indiqueront à la **plateforme d’appel de bot Skype** d’afficher des invites, d’enregistrer du contenu audio, procéder à une reconnaissance vocale ou de collecter des chiffres à partir d’un clavier de numérotation. La dernière action du flux de travail peut être de terminer l’appel. 

Cet exemple de code définit un gestionnaire qui configure un menu une fois le message d’accueil terminé.

```cs
private Task OnPlayPromptCompleted(PlayPromptOutcomeEvent playPromptOutcomeEvent)
{
    var callState = this.callStateMap[playPromptOutcomeEvent.ConversationResult.Id];
    SetupInitialMenu(playPromptOutcomeEvent.ResultingWorkflow);
    return Task.FromResult(true);
}
```

La méthode `CreateIvrOptions` définit le menu qui est montré à l’utilisateur.

```cs
private static Recognize CreateIvrOptions(string textToBeRead, int numberOfOptions, bool includeBack)
{
    if (numberOfOptions > 9)
    {
        throw new Exception("too many options specified");
    }

    var choices = new List<RecognitionOption>();

    for (int i = 1; i <= numberOfOptions; i++)
    {
        choices.Add(new RecognitionOption { Name = Convert.ToString(i), DtmfVariation = (char)('0' + i) });
    }

    if (includeBack)
    {
        choices.Add(new RecognitionOption { Name = "#", DtmfVariation = '#' });
    }

    var recognize = new Recognize
    {
        OperationId = Guid.NewGuid().ToString(),
        PlayPrompt = GetPromptForText(textToBeRead),
        BargeInAllowed = true,
        Choices = choices
    };

    return recognize;
}
```

La classe `RecognitionOption` définit la réponse prononcée ainsi que la variation multi-fréquence bitonale (DTMF) correspondante. La DTMF permet à l’utilisateur de répondre en tapant les chiffres correspondants sur le pavé au lieu de parler.

La méthode `OnRecognizeCompleted` traite la sélection de l’utilisateur et le paramètre d’entrée `recognizeOutcomeEvent` contient la valeur de la sélection de l’utilisateur.

```cs
private Task OnRecognizeCompleted(RecognizeOutcomeEvent recognizeOutcomeEvent)
{
    var callState = this.callStateMap[recognizeOutcomeEvent.ConversationResult.Id];
    ProcessMainMenuSelection(recognizeOutcomeEvent, callState);
    return Task.FromResult(true);
}
```

## <a name="support-natural-language"></a>Prise en charge du langage naturel
Le bot peut également être conçu pour prendre en charge les réponses en langage naturel. L’**API Reconnaissance vocale Bing** permet au bot de reconnaître des mots dans la réponse énoncée de l’utilisateur.

```cs
private async Task OnRecordCompleted(RecordOutcomeEvent recordOutcomeEvent)
{
    recordOutcomeEvent.ResultingWorkflow.Actions = new List<ActionBase>
    {
        GetPromptForText(EndingMessage),
        new Hangup { OperationId = Guid.NewGuid().ToString() }
    };

    // Convert the audio to text
    if (recordOutcomeEvent.RecordOutcome.Outcome == Outcome.Success)
    {
        var record = await recordOutcomeEvent.RecordedContent;
        string text = await this.GetTextFromAudioAsync(record);

        var callState = this.callStateMap[recordOutcomeEvent.ConversationResult.Id];

        await this.SendSTTResultToUser("We detected the following audio: " + text, callState.Participants);
    }

    recordOutcomeEvent.ResultingWorkflow.Links = null;
    this.callStateMap.Remove(recordOutcomeEvent.ConversationResult.Id);
}
```

## <a name="sample-code"></a>Exemple de code

Pour obtenir un exemple complet qui montre comment prendre en charge les appels audio avec Skype en utilisant le Kit de développement logiciel Bot Builder pour .NET, consultez la page <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/skype-CallingBot" target="_blank">Skype Calling Bot sample</a> (exemple de Bot Skype Calling) dans GitHub.

## <a name="additional-resources"></a>Ressources supplémentaires

- <a href="/dotnet/api/?view=botbuilder-3.11.0" target="_blank">Documentation de référence concernant le Kit de développement logiciel (SDK) Bot Builder pour .NET</a>
- <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/skype-CallingBot" target="_blank">Skype Calling Bot sample (GitHub)</a> (Exemple de Bot Skype Calling)
