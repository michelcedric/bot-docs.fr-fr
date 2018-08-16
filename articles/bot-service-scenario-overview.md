---
title: Vue d’ensemble des scénarios Bot Service | Microsoft Docs
description: Découvrez les scénarios clés dans lesquels vous pouvez tirer parti de la puissance des bots créés avec Bot Service.
author: BrianRandell
ms.author: v-brra
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 06be4330d34068bf86466b04686d6636971d0a5e
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39300540"
---
# <a name="bot-scenarios"></a>Scénarios impliquant des bots
Cette rubrique explore les scénarios clés dans lesquels vous pouvez tirer parti de la puissance des bots créés avec Bot Service.

Vous pouvez télécharger ou cloner le code source de tous les exemples de scénarios de bot ici : [Samples for Common Bot Framework Scenarios](https://aka.ms/bot/scenarios).

## <a name="commerce-bot-scenario"></a>Scénario impliquant le bot Commerce
Le scénario du [bot Commerce](bot-service-scenario-commerce.md) implique un bot qui remplace les interactions traditionnelles par e-mail et par téléphone que les clients des hôtels ont généralement avec la réception. Le bot utilise Cognitive Services pour mieux traiter les demandes des clients par le texte et la voix avec un contexte résultant de l’intégration de services backend.

Dans le scénario du bot Commerce, un client peut faire une demande auprès de la réception d’un hôtel. Il est authentifié via un point de terminaison d’authentification Azure Active Directory v2. Le bot peut parcourir les réservations du client et lui fournir différentes options de service. Par exemple, le client peut avoir réservé une cabine près de la piscine. Le bot utilise les services LUIS (Language Understanding Intelligent Services) pour analyser la demande, puis guide l’utilisateur dans la réservation d’une cabine dans le cadre de sa réservation.

## <a name="cortana-skill-bot-scenario"></a>Scénario impliquant le bot Compétence Cortana
Le scénario [bot Compétence Cortana](bot-service-scenario-cortana-skill.md) utilise la technologie Cortana. En utilisant l’interface naturelle que constitue votre voix et un bot Compétence Cortana personnalisé, vous pouvez demander à Cortana de parler à une organisation (par exemple, une entreprise proposant un service de nettoyage de véhicule) pour prendre rendez-vous en fonction du moment où vous passez l’appel. Le bot peut fournir la liste des services, les créneaux disponibles et la durée.

## <a name="enterprise-productivity-bot-scenario"></a>Scénario impliquant le bot Productivité d’entreprise
Le scénario du [bot Productivité d’entreprise](bot-service-scenario-enterprise-productivity.md) vous montre comment intégrer un bot à votre calendrier Office 365 et à d’autres services afin d’augmenter votre productivité.

Le bot s’intègre à Office 365 pour accélérer et faciliter la création d’une demande de réunion. Durant ce processus, vous pouvez accéder à des services supplémentaires tels que Dynamics CRM. Cet exemple fournit le code nécessaire à l’intégration à Office 365, avec une authentification via Azure Active Directory. Il fournit des points d’entrée factices pour des services externes pour que le lecteur puisse s’exercer.

## <a name="information-bot-scenario"></a>Scénario impliquant le bot Informations
Le [bot Informations](bot-service-scenario-informational.md) peut répondre à des questions définies dans une base de connaissances ou des questions fréquentes (FAQ) à l’aide du service QnA Maker de Cognitive Services. De plus, il peut répondre à des questions plus ouvertes à l’aide de la Recherche Azure.

Souvent, les informations sont enfouies dans des banques de données structurées telles que SQL Server, et peuvent être facilement récupérées au moyen d’une recherche. Imaginez-vous rechercher le statut d’une commande client à l’aide de commandes de conversation simples. Avec QnA Maker de Cognitive Services, l’utilisateur se voit proposer un ensemble d’options de recherche valides, comme la recherche d’un client, de sa commande la plus récente, etc. Lorsque le format QnA est défini, l’utilisateur peut facilement poser des questions, aidé par la Recherche Azure qui recherche les données stockées dans une base de données SQL.

## <a name="iot-bot-scenario"></a>Scénario impliquant le bot IoT
Le [bot IoT](bot-service-scenario-internet-things.md) vous permet de contrôler facilement les appareils de votre maison, comme l’éclairage Philips Hue, à l’aide des commandes de conversation interactive.

Avec ce simple bot et le service gratuit IFTTT, vous pouvez contrôler vos lumières Philips Hue. En tant qu’appareil IoT, Philips Hue peut être contrôlé localement par le biais de son API exposée. Toutefois, cette API n’est pas exposée pour l’accès général en dehors du réseau local. Le service IFTTT est « [compatible avec Hue](http://www2.meethue.com/en-us/friends-of-hue/ifttt/) ». Il a donc exposé plusieurs commandes de contrôle que vous pouvez exécuter, comme allumer et éteindre la lumière, ou régler la couleur ou l’intensité de la lumière.

## <a name="next-steps"></a>Étapes suivantes
Maintenant que vous avez eu une vue d’ensemble des scénarios, vous pouvez approfondir chacun d’eux.

> [!div class="nextstepaction"]
> [Bot Commerce](bot-service-scenario-commerce.md)
