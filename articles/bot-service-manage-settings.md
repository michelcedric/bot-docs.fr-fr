---
title: Configurer les paramètres du bot| Microsoft Docs
description: Découvrez comment configurer les différentes options de votre bot sur le Portail Azure.
keywords: configurer les paramètres du bot, nom d’affichage, icône, Application Insights, panneau Paramètres
author: v-royhar
ms.author: v-royhar
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: 531e37f2186de2e315f11362dcefcc30a2ab6879
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39299196"
---
# <a name="configure-bot-settings"></a>Configurer les paramètres du bot

Les paramètres du bot, comme le nom d’affichage, l’icône et Application Insights, sont visibles et modifiables sur son panneau **Paramètres**.

![Panneau Paramètres du bot](~/media/bot-service-portal-configure-settings/bot-settings-blade.png)

Voici la liste des champs du panneau **Paramètres** :

| Champ | Description |
| :---  | :---        |
| Icône | Icône personnalisée permettant d’identifier visuellement le bot dans les canaux et servant d’icône pour Skype, Cortana et d’autres services. Cette icône doit être au format PNG et ne pas dépasser 30 K. Cette valeur peut être modifiée à tout moment. |
| Nom complet | Nom du bot dans les répertoires et les canaux. Cette valeur peut être modifiée à tout moment. Limitée à 35 caractères. |
| Descripteur du bot | Identificateur unique du bot. Cette valeur n’est plus modifiable une fois le bot créé avec Bot Service. |
| Point de terminaison de messagerie | Point de terminaison permettant de communiquer avec le bot. |
| ID d’application Microsoft | Identificateur unique du bot. Cette valeur ne peut pas être modifiée. Pour générer un nouveau mot de passe, cliquez sur le lien **Gérer**. |
| Clé d'instrumentation Application Insights | Clé unique de télémétrie du bot. Copiez votre clé Azure Application Insights dans ce champ si vous souhaitez recevoir les données de télémétrie du bot. Cette valeur est facultative. Cette clé est générée automatiquement pour les bots créés sur le Portail Azure. Pour plus d’informations sur ce champ, voir [Clés Application Insights](~/bot-service-resources-app-insights-keys.md). |
| Clé API Application Insights | Clé unique d’analyse du bot. Copiez votre clé API Azure Application Insights dans ce champ si vous souhaitez afficher des données d’analyse sur votre bot dans le Tableau de bord. Cette valeur est facultative. Pour plus d’informations sur ce champ, voir [Clés Application Insights](~/bot-service-resources-app-insights-keys.md). |
| ID d'application Application Insights | Clé unique d’analyse du bot. Copiez votre clé d’ID d’application Azure Application Insights dans ce champ si vous souhaitez afficher des données d’analyse sur votre bot dans le Tableau de bord. Cette valeur est facultative. Cette clé est générée automatiquement pour les bots créés sur le Portail Azure. Pour plus d’informations sur ce champ, voir [Clés Application Insights](~/bot-service-resources-app-insights-keys.md). |

> [!NOTE]
> Une fois que vous avez modifié les paramètres de votre bot, cliquez sur le bouton **Enregistrer** en haut du panneau pour les enregistrer.

## <a name="next-steps"></a>Étapes suivantes
Maintenant que vous avez appris à configurer les paramètres de votre service de bot, découvrez comment configurer la préparation vocale.
> [!div class="nextstepaction"]
> [Préparation vocale](bot-service-manage-speech-priming.md)