---
title: Appel multimédia en temps réel avec Skype | Microsoft Docs
description: Comprenez les concepts clés de la création d’un robot capable de conduire des appels audio et vidéo en temps réel avec Skype en utilisant le Kit de développement logiciel (SDK) Bot Builder pour .NET.
author: ssulzer
ms.author: ssulzer
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 255248449ab7adb6512db93ece51389d65c01dee
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39299248"
---
# <a name="real-time-media-calling-with-skype"></a>Appel multimédia en temps réel avec Skype

La plateforme multimédia en temps réel pour robots ajoute une dimension à la façon dont les robots peuvent interagir avec des utilisateurs en activant des modalités de voix, de vidéo et de partage d’écran en temps réel. Cela permet de créer des robots de divertissement, éducatifs et d’assistance à la fois captivants et interactifs. Les utilisateurs communiquent avec des robots multimédia en temps réel à l’aide de Skype.

Il s’agit d’une fonctionnalité avancée qui permet à un robot d’envoyer et recevoir des contenus vocaux et vidéo *trame par trame*. Le robot dispose d’un accès « brut » aux modalités de voix, vidéo et de partage d’écran en temps réel. Par exemple, quand l’utilisateur parle, le robot reçoit 50 trames audio par seconde, chacune d’elles contenant 20 millisecondes (ms) de contenu audio. Le robot peut effectuer la reconnaissance vocale en temps réel à mesure qu’il reçoit des trames audio, au lieu de devoir attendre un enregistrement après que l’utilisateur a cessé de parlé. Le robot peut également envoyer à l’utilisateur une vidéo haute résolution de 30 trames par seconde, en même temps que l’audio.

La plateforme multimédia en temps réel pour robots fonctionne avec le Cloud Appel Skype pour la configuration d’appel et l’établissement de session multimédia, ce qui permet au robot de s’engager dans une conversation vocale et (éventuellement) vidéo avec un appelant Skype. La plateforme fournit une simple API de type « socket » pour permettre au robot d’envoyer et recevoir du contenu multimédia, et gère l’encodage et le décodage en temps réel de ce contenu en utilisant des codecs tels que SILK pour l’audio et H.264 pour la vidéo. Un robot multimédia en temps réel peut également participer à des appels vidéo de groupe Skype impliquant plusieurs participants.

Cet article présente les concepts clés liés à la création d’un robot capable de conduire des appels audio et vidéo en temps réel avec Skype, et fournit des liens vers des ressources pertinentes pour les développeurs.

## <a name="media-session"></a>Session multimédia
Quand un robot multimédia en temps réel répond à un appel Skype entrant, il décide de prendre en charge les modalités audio et vidéo, ou uniquement l’audio. Pour chaque modalité prise en charge, le robot peut envoyer et recevoir du contenu multimédia, uniquement en recevoir ou uniquement en envoyer. Par exemple, un robot peut souhaiter envoyer et recevoir du contenu audio, mais envoyer uniquement du contenu vidéo (parce qu’il ne souhaite pas recevoir de contenu vidéo de l’appelant Skype). L’ensemble des modalités audio et vidéo établies entre l’appelant Skype et le robot est appelé **session multimédia**.

Deux modalités vidéo sont prises en charge : la « vidéo principale » et le « partage d’écran ». La vidéo principale transmet la vidéo que le robot génère ou lit à l’appelant, et la vidéo de l’appelant (généralement de sa webcam) au robot. La modalité de partage d’écran permet à l’appelant de partager son écran (sous forme de vidéo) avec le robot. Le robot ne peut pas envoyer de vidéo de partage d’écran à l’utilisateur.

Lorsqu’il est associé à un **appel vidéo de groupe** Skype impliquant plusieurs interlocuteurs, le robot multimédia en temps réel peut prendre en charge la réception simultanée de plusieurs flux vidéo principaux. Cela permet au robot de « voir » plusieurs participants dans l’appel vidéo de groupe.

## <a name="frames-and-frame-rate"></a>Trames et fréquence d’images
Un robot multimédia en temps réel interagit directement avec les modalités audio et vidéo d’un appel Skype. Cela signifie que le robot envoie et/ou reçoit du contenu multimédia sous la forme d’une séquence de **trames**, chacune d’elles représentant une unité de contenu. Une seconde d’audio peut être transmise sous la forme d’une séquence de 50 trames, chacune d’elles comprenant 20 millisecondes (ms), soit 1/50e de seconde de contenu. Une seconde de vidéo peut être découpée en une séquence de 30 trames fixes, chacune devant être visualisée pendant seulement 33 ms (1/30e de seconde) avant que la trame vidéo suivante soit affichée. Le nombre de trames transmises ou restituées par seconde est appelé **fréquence d’images**. « 30 ips » indique 30 images par seconde.

## <a name="audio-format"></a>Format audio
Chaque seconde d’audio est représentée par 16 000 **échantillons**, chacun stockant 16 bits de données. Une trame audio de 20 ms audio 320 échantillons (640 octets de données).

## <a name="video-format"></a>Format vidéo
Plusieurs formats vidéo sont pris en charge. Les deux principales propriétés d’un format vidéo sont la **taille de trame** et le **format de couleur**. Les tailles de trame prises en charge sont 640 x 360 (« 360p »), 1280 x 720 (« 720p ») et 1920 x 1080 (« 1080p »). Les formats de couleur pris en charge sont NV12 (12 bits par pixel) et RVB24 (24 bits par pixel).

Une trame vidéo « 720p » contient 921 600 pixels (1280 x 720). Dans le format de couleur RVB24, chaque pixel est représenté par 3 octets (24 bits) correspondant aux trois composants de couleur rouge, vert et bleu. Par conséquent, une trame vidéo 720p RVB24 nécessite 2 764 800 octets de données (921 600 pixels fois 3 octets/pixel). À une fréquence d’images de 30 IPS, l’envoi de trames vidéo 720p RVB24 implique le traitement d’environ 80 Mo/s de contenu (sensiblement compressé par le codec vidéo H.264 avant transmission réseau).

## <a name="active-and-dominant-speakers"></a>Orateurs actifs et dominants
Associé à un appel vidéo de groupe Skype réunissant plusieurs personnes, le robot peut identifier les participants en train de parler. Les **orateurs actifs** sont les participants audibles dans chaque trame audio reçue. Les **orateurs dominants** sont les participants actuellement les plus actifs (ou « dominants ») dans la conversation de groupe, même si leur voix n’est pas audible dans chaque trame audio. Le groupe des orateurs dominants peut changer à mesure que différents participants prennent la parole.

## <a name="video-subscription"></a>Abonnement vidéo
Dans le cadre d’un appel avec un seul appelant Skype, le robot reçoit automatiquement la vidéo de l’appelant (si le robot est activé pour recevoir la vidéo principale). Dans le cadre d’un appel vidéo de groupe Skype avec plusieurs participants, le robot doit indiquer à la plateforme multimédia en temps réel les participants qu’il souhaite voir. Un abonnement vidéo est une demande du robot pour recevoir la vidéo d’un participant. À mesure que les participants à un appel vidéo de groupe mènent leur conversation, un robot peut modifier ses abonnements vidéo souhaités en fonction des mises à jour du groupe d’orateurs dominants.

## <a name="developer-resources"></a>Ressources pour les développeurs 

### <a name="sdks"></a>Kits de développement logiciel (SDK)

Pour développer un robot multimédia en temps réel, vous devez installer les packages NuGet suivants dans votre projet Visual Studio :

- [Kit de développement logiciel Bot Builder pour .NET](bot-builder-dotnet-overview.md)
- [Appel multimédia en temps réel Bot Builder pour .NET](https://www.nuget.org/packages?q=Bot.Builder.RealTimeMediaCalling)
- [Bibliothèque .NET Microsoft.Skype.Bots.Media](https://www.nuget.org/packages?q=Microsoft.Skype.Bots.Media)

### <a name="code-samples"></a>Exemples de code

Le dépôt GitHub [BotBuilder-RealTimeMediaCalling](https://github.com/Microsoft/BotBuilder-RealTimeMediaCalling) contient des exemples qui montrent comment créer des robots multimédia en temps réel pour Skype.
