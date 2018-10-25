---
title: Concevoir un bot multimédia en temps réel pour Skype | Microsoft Docs
description: Découvrez comment créer un robot qui effectue des appels audio/vidéo en temps réel avec Skype, à l’aide du kit de développement logiciel (SDK) Bot Builder pour .NET et le kit de développement logiciel (SDK) Bot Builder-RealTimeMediaCalling pour .NET.
author: MalarGit
ms.author: malarch
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/17
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 6ceeca9adc9cad9e60a73c1c7c91bea43b97fdd9
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2018
ms.locfileid: "49997918"
---
# <a name="build-a-real-time-media-bot-for-skype"></a>Concevoir un bot multimédia en temps réel pour Skype

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

La plateforme multimédia en temps réel pour bots est une fonctionnalité avancée qui permet à un bot d’envoyer et recevoir des contenus vocaux et vidéo image par image. Le bot dispose d’un accès « brut » aux modalités de voix, vidéo et partage d’écran en temps réel. Cet article présente la conception d’un bot d’appel audio/vidéo et l’accès aux modalités en temps réel.

Dans cet article, le robot s’exécute dans un Service Cloud Azure, en tant que rôle Web ou rôle de travail qui auto-héberge l’infrastructure API Web ASP.NET.

> [!IMPORTANT]
> Il s’agit d’un article préliminaire ; reportez-vous au dossier Exemples dans le référentiel GitHub <a href="https://github.com/Microsoft/BotBuilder-RealTimeMediaCalling">BotBuilder-RealTimeMediaCalling</a> pour obtenir l’intégralité du code d’exemples de bots multimédia en temps réel.

## <a name="configure-the-service-hosting-the-real-time-media-bot"></a>Configurer le service qui héberge le bot multimédia en temps réel

Pour pouvoir utiliser la plateforme multimédia en temps réel, les configurations de service suivantes sont nécessaires.

* Le robot doit connaître le nom de domaine complet (FQDN) de son service. Ce dernier n’est pas fourni par l’API Azure RoleEnvironment, mais doit être stocké dans la configuration de Service Cloud du bot et être lu lors du démarrage du service.

* Le Service Bot doit disposer d’un certificat émis par une autorité de certification reconnue. L’empreinte numérique du certificat doit être stockée dans la configuration de Service Cloud du bot et être lu lors du démarrage du service.

* Un <a href="/azure/cloud-services/cloud-services-enable-communication-role-instances#instance-input-endpoint">point de terminaison externe d’instance</a> public doit être approvisionné. Cela assigne un port public unique à chaque instance de machine virtuelle (VM) du service du bot. Ce port est utilisé par la plateforme multimédia en temps réel pour communiquer avec Skype Calling Cloud.
  ```xml
  <InstanceInputEndpoint name="InstanceMediaControlEndpoint" protocol="tcp" localPort="20100">
    <AllocatePublicPortFrom>
    <FixedPortRange max="20200" min="20101" />
    </AllocatePublicPortFrom>
  </InstanceInputEndpoint>
  ```

  Il est également utile pour créer un autre point de terminaison externe d’instance pour les rappels et les notifications liés aux appels. L’utilisation d’un point de terminaison externe d’instance garantit que les rappels et les notifications sont remis à la même instance de machine virtuelle dans le déploiement de service qui héberge la session multimédia de support en temps réel pour l’appel.
  ```xml
  <InstanceInputEndpoint name="InstanceCallControlEndpoint" protocol="tcp" localPort="10100">
    <AllocatePublicPortFrom>
    <FixedPortRange max="10200" min="10101" />
    </AllocatePublicPortFrom>
  </InstanceInputEndpoint>
  ```

* Chaque instance de machine virtuelle doit disposer d’une adresse IP publique de niveau d'instance (ILPIP). Lors du démarrage, le bot doit découvrir l’adresse ILPIP assignée à chaque instance de service. Consultez <a href="/azure/virtual-network/virtual-networks-instance-level-public-ip">ILPIP</a> pour plus d’informations sur l’obtention et la configuration d’une adresse ILPIP.
  ```xml
  <NetworkConfiguration>
  <AddressAssignments>
    <InstanceAddress roleName="WorkerRole">
    <PublicIPs>
        <PublicIP name="InstancePublicIP" domainNameLabel="InstancePublicIP" />
    </PublicIPs>
    </InstanceAddress>
  </AddressAssignments>
  </NetworkConfiguration>
  ```

* Pendant le démarrage de l’instance de service, le script `MediaPlatformStartupScript.bat` (fourni dans le cadre du package Nuget) doit être exécuté en tant que tâche de démarrage avec des privilèges élevés. L’exécution du script doit se terminer avant l’appel de la méthode d’initialisation de la plateforme. 

```xml
<Startup>
<Task commandLine="MediaPlatformStartupScript.bat" executionContext="elevated" taskType="simple" />      
</Startup> 
```

## <a name="initialize-the-media-platform-on-service-startup"></a>Initialiser la plateforme multimédia au démarrage du service

Lors du démarrage de l’instance de service, la plateforme multimédia en temps réel doit être initialisée. Cela doit être effectué une seule fois avant que le bot puisse accepter les appels Skype audio/vidéo sur cette instance. L’initialisation de la plateforme multimédia exige de fournir différents paramètres de configuration, y compris le nom de domaine complet du service, l’adresse ILPIP publique, la valeur de port du point de terminaison externe d’instance et l’**ID d’application Microsoft** du bot.

> [!NOTE]
> Pour trouver les paramètres **AppID** et **AppPassword** de votre robot, consultez la rubrique [MicrosoftAppID and MicrosoftAppPassword](~/bot-service-manage-overview.md#microsoftappid-and-microsoftapppassword) (MicrosoftAppID et MicrosoftAppPassword).

```cs
var mediaPlatformSettings = new MediaPlatformSettings()
{
    MediaPlatformInstanceSettings = new MediaPlatformInstanceSettings()
    {
        CertificateThumbprint = certificateThumbprint,
        InstanceInternalPort = instanceMediaControlEndpointInternalPort,
        InstancePublicIPAddress = instancePublicIPAddress,
        InstancePublicPort = instanceMediaControlEndpointPublicPort,
        ServiceFqdn = serviceFqdn
    },

    ApplicationId = MicrosoftAppId
};

MediaPlatform.Initialize(mediaPlatformSettings);            
```

## <a name="register-to-receive-incoming-call-requests"></a>Inscription pour recevoir des requêtes d’appels entrants

Définissez une classe `CallController`. Cela permet au Service Bot de s’inscrire pour les appels entrants Skype et d’assurer que les requêtes de rappel et de demandes de notification sont transmises vers l’objet `RealTimeMediaCall` approprié.

```cs
[BotAuthentication]
[RoutePrefix("api/calling")]
public class CallController : ApiController
{
    static CallController()
    {
        RealTimeMediaCalling.RegisterRealTimeMediaCallingBot(
            c => { return new RealTimeMediaCall(c); },
            new RealTimeMediaCallingBotServiceSettings()
        );
    }

    [Route("call")]
    public async Task<HttpResponseMessage> OnIncomingCallAsync()
    {
        // forwards the incoming call to the associated RealTimeMediaCall object
        return await RealTimeMediaCalling.SendAsync(this.Request, RealTimeMediaCallRequestType.IncomingCall);
    }

    [Route("callback")]
    public async Task<HttpResponseMessage> OnCallbackAsync()
    {
        // forwards the incoming callback to the associated RealTimeMediaCall object
        return await RealTimeMediaCalling.SendAsync(this.Request, RealTimeMediaCallRequestType.CallingEvent);
    }

    [Route("notification")]
    public async Task<HttpResponseMessage> OnNotificationAsync()
    {
        // forwards the incoming notification to the associated RealTimeMediaCall object
        return await RealTimeMediaCalling.SendAsync(this.Request, RealTimeMediaCallRequestType.NotificationEvent);
    }
}
```

`RealTimeMediaCallingBotServiceSettings` implémente `IRealTimeMediaCallServiceSettings` et fournit des liens webhook pour les événements de rappel et de notification.

## <a name="register-for-incoming-events-for-the-call"></a>S’inscrire aux événements entrants pour l’appel

Créez une classe `RealTimeMediaCall` qui implémente `IRealTimeMediaCall`. Pour chaque appel reçu par le bot, une instance de `RealTimeMediaCall` est créée par l’infrastructure de bot. L’objet `IRealTimeMediaCallService` passé au constructeur permet au robot de s’inscrire aux événements afin de gérer les événements associés à l’appel multimédia en temps réel.

```cs
internal class RealTimeMediaCall : IRealTimeMediaCall
{
     public RealTimeMediaCall(IRealTimeMediaCallService callService)
     {
         if (callService == null)
             throw new ArgumentNullException(nameof(callService));

         CallService = callService;
         CorrelationId = callService.CorrelationId;
         CallId = CorrelationId + ":" + Guid.NewGuid().ToString();

         // Register for the call events
         CallService.OnIncomingCallReceived += OnIncomingCallReceived;
         CallService.OnAnswerAppHostedMediaCompleted += OnAnswerAppHostedMediaCompleted;
         CallService.OnCallStateChangeNotification += OnCallStateChangeNotification;
         CallService.OnRosterUpdateNotification += OnRosterUpdateNotification;
         CallService.OnCallCleanup += OnCallCleanup;
     }
}
```

## <a name="create-audio-and-video-sockets"></a>Créer des sockets audio et vidéo
Avant d’accepter un appel entrant audio/vidéo Skype, le bot doit créer des objets `AudioSocket` et `VideoSocket` pour prendre en charge l’envoi et la réception de médias en temps réel. (Si le bot ne souhaite pas prendre en charge la vidéo, il suffit de créer uniquement un AudioSocket.)

Le robot doit décider à l’avance des modalités qu’il souhaite prendre en charge et créer les objets AudioSocket et VideoSocket appropriés. Il ne peut pas modifier les modalités prises en charge pour l’appel après l’acceptation de l’appel entrant.

Pour chaque AudioSocket et VideoSocket, le robot spécifie si le socket de média sert à la prise en charge de l’envoi et de la réception de média, ou uniquement l’un ou l’autre. Par exemple, le bot peut souhaiter envoyer et recevoir les données audio (« Sendrecv »), mais envoyer uniquement pour les données vidéo (« Sendonly »). Le bot doit également spécifier quels formats de média sont pris en charge pour chaque socket de média. Pour un AudioSocket, le format actuellement pris en charge est « Pcm16K » : encodage PCM 16 bits (signé), fréquence d’échantillonnage de 16 KHz. Pour un VideoSocket, les formats multimédia d’envoi et de réception sont spécifiés séparément. Seul le format « NV12 » est pris en charge pour la réception de vidéo, tandis que différents formats sont pris en charge pour l’envoi.

```cs
_audioSocket = new AudioSocket(new AudioSocketSettings
{
    StreamDirections = StreamDirection.Sendrecv,
    SupportedAudioFormat = AudioFormat.Pcm16K,
    CallId = correlationId
});

_videoSocket = new VideoSocket(new VideoSocketSettings
{
    StreamDirections = StreamDirection.Sendrecv,
    ReceiveColorFormat = VideoColorFormat.NV12,
    SupportedSendVideoFormats = new VideoFormat[]
    {
        VideoFormat.Yuy2_1280x720_30Fps,
        VideoFormat.Yuy2_720x1280_30Fps,
    },
    CallId = correlationId
});

_audioSocket.AudioMediaReceived += OnAudioMediaReceived;
_audioSocket.AudioSendStatusChanged += OnAudioSendStatusChanged;
_audioSocket.DominantSpeakerChanged += OnDominantSpeakerChanged;
_videoSocket.VideoMediaReceived += OnVideoMediaReceived;
_videoSocket.VideoSendStatusChanged += OnVideoSendStatusChanged;
```                             

## <a name="create-a-mediaconfiguration"></a>Créer un objet MediaConfiguration
Une fois que les sockets de média ont été créés, le robot doit créer un objet `MediaConfiguration`, nécessaire pour associer les sockets de média à un appel entrant audio/vidéo Skype.

```cs
var mediaConfiguration = MediaPlatform.CreateMediaConfiguration(_audioSocket, _videoSocket);
```

##  <a name="answer-an-incoming-audiovideo-call"></a>Répondre à un appel audio/vidéo entrant
L’événement `OnIncomingCallReceived` est appelé pour permettre au robot d’accepter l’appel entrant audio/vidéo Skype. Pour ce faire, le bot crée un objet `AnswerAppHostedMedia` avec l’objet `MediaConfiguration`. Le bot s’inscrit aux notifications associées à cet appel Skype.

```cs
private Task OnIncomingCallReceived(RealTimeMediaIncomingCallEvent incomingCallEvent)
{
    // ... create Audio/VideoSocket objects and MediaConfiguration ...

    incomingCallEvent.RealTimeMediaWorkflow.Actions = new ActionBase[]
    {
        new AnswerAppHostedMedia
        {
            MediaConfiguration = mediaConfiguration,
            OperationId = Guid.NewGuid().ToString()
        }
    };

    // subscribe for roster and call state changes
    incomingCallEvent.RealTimeMediaWorkflow.NotificationSubscriptions = new NotificationType[]
    {
        NotificationType.CallStateChange,
        NotificationType.RosterUpdate
    };
}
```

## <a name="outcome-of-the-call"></a>Résultat de l’appel
`OnAnswerAppHostedMediaCompleted` est déclenché lorsque l’action `AnswerAppHostedMedia` se termine. La propriété `Outcome` dans l’évènement `AnswerAppHostedMediaOutcomeEvent` indique la réussite ou l’échec. Si l’appel ne peut pas être établi, le robot doit supprimer les objets AudioSocket et VideoSocket qu’il a créés pour l’appel.

## <a name="receive-audio-media"></a>Recevoir des médias audio
Si `AudioSocket` a été créé avec la possibilité de recevoir les données audio, l’événement `AudioMediaReceived` sera appelé chaque fois qu’une trame audio est reçue. Le robot doit s’attendre à gérer cet événement environ 50 fois par seconde, quel que soit le pair qui pourrait approvisionner le contenu audio (étant donné que des tampons de bruit de confort sont générés localement en l’absence de réception du contenu audio du pair). Chaque paquet de contenu audio est remis dans un objet `AudioMediaBuffer`. Cet objet contient un pointeur vers une mémoire tampon native allouée sur le tas qui contient le contenu audio décodé. 

```cs
void OnAudioMediaReceived(
            object sender,
            AudioMediaReceivedEventArgs args)
{
   var buffer = args.Buffer;

   // native heap-allocated memory containing decoded content
   IntPtr rawData = buffer.Data;            
}
```

Le Gestionnaire d’événements doit effectuer un retour rapidement. Il est recommandé de traiter de façon asynchrone la file d’attente de l’application de `AudioMediaBuffer`. Les événements `OnAudioMediaReceived` doivent être sérialisés par la plateforme multimédia en temps réel (autrement dit, l’événement suivant n’est pas déclenché avant que celui en cours ait été retourné). Une fois `AudioMediaBuffer` utilisé, l’application doit appeler méthode Dispose de la mémoire tampon afin que la mémoire non managée sous-jacente puisse être récupérée par la plateforme multimédia. 

```cs
   // release/dispose buffer when done 
   buffer.Dispose();
```

> [!IMPORTANT]
> Le bot ne doit pas appeler la méthode Dispose de la mémoire tampon s’il n’a pas terminé d’accéder à la mémoire tampon.

## <a name="receive-video-media"></a>Recevoir des média vidéo
La réception des données vidéo est similaire à la réception des données audio à ceci près que le nombre de tampons par seconde varie selon la valeur de la fréquence d’images. `VideoMediaBuffer` dispose de propriétés `VideoFormat` et `OriginalVideoFormat`. `OriginalVideoFormat` représente le format d’origine de la mémoire tampon lorsqu’il a été obtenu. Il est disponible uniquement lors de la réception des mémoires tampons vidéo via le Gestionnaire d’événements `IVideoSocket.VideoMediaReceived`. Si la mémoire tampon a été redimensionnée avant d’être transmise, la propriété `OriginalVideoFormat` disposera de la largeur et la hauteur d’origine, tandis que `VideoFormat` disposera des dimensions actuelles après redimensionnement. Si les propriétés Width et Height de `OriginalVideoFormat` diffèrent de la propriété `VideoFormat`, le consommateur de `VideoMediaBuffer` déclenché lors de l’événement `VideoMediaReceived` doit redimensionner la mémoire tampon pour correspondre à la taille du paramètre `OriginalVideoFormat`. Actuellement, seul le format NV12 est pris en charge pour la réception.

## <a name="send-audio-media"></a>Envoyer un média audio
Si le `AudioSocket` est configuré pour envoyer des média, le robot doit s’inscrire au Gestionnaire d’événements `AudioSendStatusChanged` sur `AudioSocket` pour obtenir des notifications sur les modifications d’état d’envoi. Le robot doit commencer à envoyer l’audio uniquement après que les modifications d’état d’envoi deviennent Active.

```cs
void AudioSocket_OnSendStatusChanged(
             object sender,
             AudioSendStatusChangedEventArgs args)
{
    switch (args.MediaSendStatus)
    {
    case MediaSendStatus.Active:
        // notify bot to begin sending audio 
        break;
     
    case MediaSendStatus.Inactive:
        // notify bot to stop sending audio
        break;
    }
}
```

Pour envoyer un média audio, il est supposé que le contenu audio PCM est contenu dans une mémoire tampon native allouée sur le tas. Le bot doit dériver de la classe abstraite `AudioMediaBuffer`. Les média sont envoyés de manière asynchrone par la méthode `Send` d’AudioSocket et la plateforme appellera `Dispose` sur `AudioMediaBuffer` lorsque l’envoi est terminé. Le bot ne doit pas libérer (ou renvoyer à un allocateur de pool) les ressources non managées lorsque `Send` est renvoyé. Il doit attendre l’appel de `Dispose`.

## <a name="send-video-media"></a>Envoyer les médias vidéo
L’envoi de médias vidéo est similaire à l’envoi de médias audio. Le bot doit s’inscrire à l’événement `VideoSendStatusChanged` et attendre que `MediaSendStatus` soit à l’état `Active`. Le bot ne doit pas libérer ou récupérer les ressources non managées de la mémoire tampon avant l’appel de la méthode `Dispose`. Les formats de couleur RGB24, NV12 et Yuy2 sont pris en charge.

Bien que plusieurs `VideoFormat` sont pris en charge pour l’envoi de vidéos, le `VideoFormat` actuellement préféré est communiqué au robot via l’événement `VideoSendStatusChanged`. Le `VideoFormat` préféré pour l’envoi de vidéo peut changer en raison des conditions de la bande passante du réseau, ou un redimensionnement de la fenêtre de la vidéo du client pair.

```cs
void VideoSocket_OnSendStatusChanged(
            object sender,
            VideoSendStatusChangedEventArgs args)
{
    VideoFormat preferredVideoFormat;

    switch (args.MediaSendStatus)
    {
    case MediaSendStatus.Active:
        // notify bot to begin sending audio 
        // bot is recommended to use this format for sourcing video content.
        preferredVideoFormat = args.PreferredVideoSourceFormat;
        break;
     
    case MediaSendStatus.Inactive:
        // notify bot to stop sending audio
        break;
    }
}
```

Chaque `VideoMediaBuffer` présente une propriété VideoFormat qui indique le format du contenu vidéo pour cette mémoire tampon en particulier. Alors que la propriété `VideoFormat` ne doit pas forcément correspondre à la propriété `PreferredVideoSourceFormat` indiquée dans l’événement `VideoSendStatusChanged`, il est fortement recommandé d’utiliser `PreferredVideoSourceFormat` afin d’éviter les cycles de processeur inutiles dus au redimensionnement de l’image vidéo.

## <a name="call-notifications"></a>Notifications d’appel
### <a name="call-state-changes"></a>Changements d'état d’appel
Le robot peut obtenir les notifications de changement d’état d’appel en s’abonnant à `NotificationType.CallStateChange` dans `NotificationSubscriptions` du flux de travail `RealTimeMediaIncomingCallEvent.RealTimeMediaWorkflow`.

```cs
private Task OnCallStateChangeNotification(CallStateChangeNotification callStateChangeNotification)
{
    if (callStateChangeNotification.CurrentState == CallState.Terminated)
    {   
        // stop sending media and dispose the media sockets
        _audioSocket.Dispose();
        _videoSocket.Dispose();
    }

    return Task.CompletedTask;
}
 ```

### <a name="roster-update"></a>Mise à jour de la liste
Si le bot est ajouté à une conférence, il peut écouter la liste de la conférence en s’abonnant à `NotificationType.RosterUpdate` dans `NotificationSubscriptions` du flux de travail `RealTimeMediaIncomingCallEvent.RealTimeMediaWorkflow`. `RosterUpdateNotification` contient les détails de tous les participants à la conférence. Le bot peut choisir d’attendre une notification comportant un participant envoyant une vidéo, puis s’abonner (`Subscribe`) à la vidéo du participant sur l’un de ses objets `VideoSocket`.

```cs
private async Task OnRosterUpdateNotification(RosterUpdateNotification rosterUpdateNotification)
{
    // Find a video source in the conference to subscribe
    foreach (RosterParticipant participant in rosterUpdateNotification.Participants)
    {
        if (participant.MediaType == ModalityType.Video
            && (participant.MediaStreamDirection == "sendonly" || participant.MediaStreamDirection == "sendrecv")
            )
        {
            var videoSubscription = new VideoSubscription
            {
                ParticipantIdentity = participant.Identity,
                OperationId = Guid.NewGuid().ToString(),
                SocketId = _videoSocket.SocketId,
                VideoModality = ModalityType.Video,
                VideoResolution = ResolutionFormat.Hd720p
            };

            await CallService.Subscribe(videoSubscription).ConfigureAwait(false);
            break;
        }
    }  
}
```

## <a name="end-the-call"></a>Terminer l'appel

### <a name="handle-call-termination-from-the-caller"></a>Gérer la clôture d’appel de l’appelant
Lorsque l’utilisateur se déconnecte de l’appel avec le bot dans une conversation de pair à pair, ou lorsque le robot est supprimé de la conférence, le bot obtient une notification de modification d’état d’appel avec la CallState au statut Terminated. Le bot doit supprimer tous les sockets media qu'il a créés.

### <a name="terminate-the-call-from-the-bot"></a>Mettre fin à l’appel à partir du bot
Le bot peut choisir de terminer l’appel en appelant `EndCall` sur `IRealTimeMediaCallService`.

### <a name="handle-call-clean-up-by-the-bot-framework"></a>Gérer le nettoyage des appels par l’infrastructure du bot
S’il existe une condition d’erreur (par exemple, si `AnswerAppHostedMediaOutcomeEvent` n’est pas reçu dans un délai raisonnable), l’infrastructure du bot peut mettre fin à l’appel. Le robot doit s’inscrire aux événements `OnCallCleanup` et supprimer les sockets média.

