---
title: Connecter un bot à Skype | Microsoft Docs
description: Découvrez comment configurer un bot pour qu’il soit accessible par le biais de l’interface de Skype.
keywords: Skype, Bot Channels, configurer Skype, publier, connecter à des canaux
author: v-ducvo
ms.author: RobStand
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 10/11/2018
ms.openlocfilehash: f52c8fd668299486eccda3920181df971f475a90
ms.sourcegitcommit: 75f32b3325dd0fc4d8128dee6c22ebf91e5785b3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2018
ms.locfileid: "53120656"
---
# <a name="connect-a-bot-to-skype"></a>Connecter un bot à Skype

Skype est un moyen de rester en contact avec des utilisateurs par le biais de la messagerie instantanée, du téléphone et des appels vidéo. Étendez cette fonctionnalité en créant des bots avec lesquels les utilisateurs peuvent interagir dans l’interface de Skype.

Pour ajouter le canal Skype, ouvrez le bot sur le [Portail Azure](https://portal.azure.com/), cliquez sur le panneau **Canaux**, puis cliquez sur **Skype**.

![Ajouter un canal Skype](~/media/channels/skype-addchannel.png)

La page de paramètres **Configurer Skype** s’ouvre.

![Configurer le canal Skype](~/media/channels/skype_configure.png)

Vous devez configurer les paramètres des sections **Contrôle Web**, **Messagerie**, **Appel**, **Groupes** et **Publish**. Décrivons-les un par un.

## <a name="web-control"></a>Contrôle web

Pour incorporer le bot dans votre site web, cliquez sur le bouton **Obtenir le code incorporé** de la section **Contrôle web**. Ceci vous dirigera vers la page Skype pour les développeurs. Suivez les instructions qui s’y trouvent pour obtenir le code incorporé.

## <a name="messaging"></a>Messagerie

Cette section permet de configurer la manière dont le bot envoie et reçoit des messages dans Skype.

## <a name="calling"></a>Appel

Cette section permet de configurer la fonctionnalité d’appel de Skype dans le bot. Vous pouvez spécifier si **l’appel** est activé pour votre bot et, dans ce cas, si la fonctionnalité IVR et la fonctionnalité multimédia en temps réel doivent être utilisées.

## <a name="groups"></a>Groupes

Cette section permet de configurer si le bot peut être ajouté à un groupe et comment il se comporte dans le groupe pour la messagerie ; elle sert également à activer les appels vidéo de groupe pour les bots d’appel.

## <a name="publish"></a>Publish

Cette section permet de configurer les paramètres de publication du bot. Tous les champs marqués d’un * sont obligatoires.

Les bots en **Préversion** sont limités à 100 contacts. Si vous avez besoin de plus de 100 contacts, soumettez votre bot à une validation. Cliquez sur **Soumettre à validation** pour ajouter automatiquement à votre bot la possibilité de recherche dans Skype en cas d’acceptation. Si votre demande n’est pas approuvée, il sera précisé ce qu’il faut modifier pour qu’elle puisse l’être.

> [!TIP]
> Si vous voulez soumettre votre bot à une validation, n’oubliez pas qu’il doit respecter la [liste de vérification de certification Skype](https://github.com/Microsoft/skype-dev-bots/blob/master/certification/CHECKLIST.md) pour être accepté.

Après avoir terminé la configuration, cliquez sur **Enregistrer** et acceptez les **Conditions d’utilisation du service**. Le canal Skype est désormais ajouté à votre bot.

## <a name="next-steps"></a>Étapes suivantes

* [Skype Entreprise](bot-service-channel-connect-skypeforbusiness.md)
