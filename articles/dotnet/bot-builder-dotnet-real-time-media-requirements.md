---
title: Exigences et considérations relatives aux bots multimédias en temps réel| Microsoft Docs
description: Appréhendez les exigences et les considérations importantes en matière de création de bots multimédias en temps réel pour Skype en utilisant le kit SDK Bot Framework pour .NET.
author: ssulzer
ms.author: ssulzer
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/17
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: d8cd326a3027fe5fcb440d9b205ba7d32a8b1640
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/11/2019
ms.locfileid: "54224934"
---
# <a name="requirements-and-considerations-for-real-time-media-bots"></a>Exigences et considérations relatives aux bots multimédias en temps réel

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

Certains conseils applicables au développement de bots d’appel de système de réponse vocale interactive (IVR) et de messagerie ne concernent pas la création de bots multimédias en temps réel. Cet article décrit quelques-unes des exigences et considérations importantes relatives au développement de bots multimédias en temps réel. 

> [!NOTE]
> Étant donné que la plateforme multimédia en temps réel pour bots est une technologie disponible en préversion, les conseils fournis dans cet article sont susceptibles d’évoluer.

## <a name="real-time-media-bot-development-requires-cnet-and-windows-server"></a>Le développement de bots multimédias en temps réel nécessite C#/.NET et Windows Server

- Le développement d’un bot multimédia en temps réel requiert la bibliothèque .NET `Microsoft.Skype.Bots.Media` (disponible par le biais de <a href="https://www.nuget.org/" target="_blank">NuGet</a>) pour l’accès aux éléments multimédias audio et vidéo, et le bot doit être déployé sur une machine Windows Server (ou sur un système d’exploitation invité Windows Server dans Azure). Par conséquent, le bot doit être développé à l’aide de C# et du composant .NET Framework standard, et déployé dans Azure. Vous ne pouvez pas utiliser les API C++ et Node.js pour accéder aux éléments multimédias en temps réel.

- Le bot multimédia en temps réel doit s’exécuter sur une version récente de la bibliothèque .NET `Microsoft.Skype.Bots.Media`. Le bot doit utiliser soit la dernière version disponible du package <a href="https://www.nuget.org/" target="_blank">NuGet</a>, soit une version datant de moins de trois mois. Les versions antérieures de la bibliothèque sont déconseillées et cesseront de fonctionner au bout de quelques mois.

## <a name="real-time-media-calls-stay-on-the-machine-where-they-were-created"></a>Les appels multimédias en temps réel restent sur la machine sur laquelle ils ont été créés

- Les bots multimédias en temps réel sont avec état. L’appel multimédia en temps réel est rattaché à l’instance de machine virtuelle ayant accepté l’appel entrant : les éléments multimédias audio et vidéo émanant de l’appelant Skype sont acheminés vers cette instance de machine virtuelle, et les éléments multimédias que le bot renvoie à l’appelant doivent également provenir de cette machine virtuelle.

- Si des appels multimédias en temps réel sont en cours lorsque la machine virtuelle est arrêtée, ces appels sont brutalement interrompus. Si le bot peut être informé de l’arrêt imminent de la machine virtuelle, il peut tenter de terminer l’appel de manière appropriée.

## <a name="real-time-media-bots-must-be-directly-accessible-on-the-internet"></a>Les bots multimédias en temps réel doivent être directement accessibles sur Internet

- Chaque instance de machine virtuelle qui héberge un bot multimédia en temps réel doit être directement accessible à partir d’Internet. Pour les bots multimédias en temps réel hébergés sur Azure, chaque instance de machine virtuelle doit disposer d’une adresse IP publique de niveau d’instance (ILPIP). Pour plus d’informations sur l’obtention et la configuration d’une adresse ILPIP, consultez l’article <a href="/azure/virtual-network/virtual-networks-instance-level-public-ip" target="_blank">Vue d’ensemble des adresses IP publiques de niveau d’instance (classique)</a>. Par défaut, un abonnement Azure peut obtenir 5 adresses ILPIP ; si vous souhaitez augmenter le quota de votre abonnement, contactez le support Azure.

- Du fait de cette exigence relative aux adresses IP publiques, les bots multimédias en temps réel doivent être hébergés sur une machine virtuelle Azure « IaaS » (infrastructure as a service) ou sur un service cloud Azure « classique ». Les autres environnements d’exécution, tels que le service de robot ou Service Fabric avec VM Scale Sets, ne sont pas pris en charge, car ils ne gèrent pas les adresses ILPIP.

- Les bots multimédias en temps réel ne sont pas non plus pris en charge par l’application [Bot Framework Emulator](../bot-service-debug-emulator.md).

## <a name="scalability-and-performance-considerations"></a>Éléments à prendre en compte en matière d’extensibilité et de niveau de performance

- Les bots multimédias en temps réel nécessitent une capacité de calcul et réseau (bande passante) plus importante que les bots de messagerie et peuvent donc accroître les coûts opérationnels de manière significative. Un développeur de bots multimédias en temps réel doit évaluer soigneusement l’extensibilité des bots et s’assurer que ces derniers n’acceptent pas plus d’appels simultanés qu’ils ne peuvent en gérer. Un bot prenant en charge les éléments vidéo ne supporte parfois qu’un ou deux appels multimédias en temps réel simultanés par cœur de processeur.

- La préversion actuelle de la plateforme multimédia en temps réel pour bots fait l’objet de certaines limites en matière d’extensibilité que le développeur de bots doit connaître (ces aspects seront éventuellement améliorés dans les versions ultérieures) : 
  1. Il est impossible de créer plus de 10 sockets vidéo à la fois pour une même instance de machine virtuelle.
  2. La plateforme multimédia en temps réel ne tire profit d’aucun processeur graphique (GPU) disponible sur la machine virtuelle pour se décharger du codage/décodage vidéo H.264. À la place, le codage et le décodage vidéo s’effectuent dans le logiciel sur l’unité centrale. Si un GPU est disponible, le bot peut en tirer avantage pour son propre rendu graphique (par exemple, si le bot utilise un moteur graphique 3D).

- L’instance de machine virtuelle qui héberge le bot multimédia en temps réel doit comporter au moins 2 cœurs de processeur. Pour Azure, l’utilisation d’une machine virtuelle série Dv2 est recommandée. Vous trouverez des informations détaillées sur les types de machines virtuelles Azure dans la <a href="/azure/virtual-machines/windows/sizes-general" target="_blank">documentation Azure</a>. 
