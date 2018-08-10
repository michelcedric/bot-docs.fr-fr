---
title: Connecter un bot à Skype | Microsoft Docs
description: Découvrez comment configurer un bot pour qu’il soit accessible par le biais de l’interface de Skype.
keywords: Skype, Bot Channels, configurer Skype, publier, connecter à des canaux
author: v-ducvo
ms.author: RobStand
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 2/1/2018
ms.openlocfilehash: 5dc4063125855113f813b8873b01df84c90e197e
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39299193"
---
# <a name="connect-a-bot-to-skype"></a>Connecter un bot à Skype

Skype est un moyen de rester en contact avec des utilisateurs par le biais de la messagerie instantanée, du téléphone et des appels vidéo. Étendez cette fonctionnalité en créant des bots avec lesquels les utilisateurs peuvent interagir dans l’interface de Skype.

Pour ajouter le canal Skype, ouvrez le bot sur le [Portail Azure](https://portal.azure.com/), cliquez sur le panneau **Canaux**, puis cliquez sur **Skype**. La page de paramètres **Configurer Skype** s’ouvre. Remplissez toutes les informations nécessaires sur votre bot et cliquez sur **Enregistrer** pour connecter le canal Skype. Acceptez les **Conditions d’utilisation du service** pour ajouter le canal Skype à votre bot.

![Ajouter un canal Skype](~/media/channels/skype-addchannel.png)

## <a name="web-control"></a>Contrôle web

Pour obtenir le code permettant d’incorporer le bot dans votre site web, cliquez sur le bouton **Obtenir le code incorporé** de la section **Contrôle web**.

## <a name="messaging"></a>Messagerie

Cette section permet de configurer la manière dont le bot envoie et reçoit des messages dans Skype.

## <a name="calling"></a>Appel

Cette section permet de configurer la fonctionnalité d’appel de Skype dans le bot. Vous pouvez spécifier si **l’appel** est activé pour votre bot et, dans ce cas, si la fonctionnalité IVR et la fonctionnalité multimédia en temps réel doivent être utilisées.

## <a name="groups"></a>Groupes

Cette section permet de configurer si le bot peut être ajouté à un groupe et comment il se comporte dans le groupe pour la messagerie ; elle sert également à activer les appels vidéo de groupe pour les bots d’appel.

## <a name="publish"></a>Publish

Cette section permet de configurer les paramètres de publication du bot. Tous les champs marqués d’un * sont obligatoires.

Les bots en **Préversion** sont limités à 100 contacts. Si vous avez besoin de plus de 100 contacts, soumettez votre bot à une validation. Cliquez sur **Soumettre à validation** pour ajouter automatiquement à votre bot la possibilité de recherche dans Skype en cas d’acceptation. Si votre demande n’est pas approuvée, il sera précisé ce qu’il faut modifier pour qu’elle puisse l’être.

## <a name="next-steps"></a>Étapes suivantes

* [Skype Entreprise](bot-service-channel-connect-skypeforbusiness.md)
