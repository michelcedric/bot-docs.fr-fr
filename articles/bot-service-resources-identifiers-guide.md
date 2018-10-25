---
title: Guide des ID dans Bot Framework | Microsoft Docs
description: Ce guide décrit les caractéristiques des champs d’ID présents dans le protocole Bot Framework v3.
keywords: id, robots, protocole
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
ms.openlocfilehash: 3c4f8549f40740961feea24f73aa2e4b9b7bc82f
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2018
ms.locfileid: "49998886"
---
# <a name="id-fields-in-the-bot-framework"></a>Champs d’ID dans Bot Framework

Ce guide décrit les caractéristiques des champs d’ID dans Bot Framework.

## <a name="channel-id"></a>ID de canal

Chaque canal de Bot Framework est identifié par un ID unique.

Exemple : `"channelId": "skype"`

Les ID de canal font office d’espaces de noms pour d’autres ID. Les appels de runtime dans le protocole Bot Framework doivent intervenir dans le contexte d’un canal. Le canal donne un sens à la conversation et aux identifiants de compte utilisés lors de la communication.

Par convention, tous les ID de canal sont en minuscules. Les canaux garantissent que les ID de canal qu’ils émettent ont une casse cohérente, et que les robots peuvent donc utiliser des comparaisons ordinales pour établir des équivalences.

### <a name="rules-for-channel-ids"></a>Règles pour les ID de canal

- Les ID sont sensibles à la casse.

## <a name="bot-handle"></a>Descripteur de robot

Chaque robot inscrit auprès du Bot Framework a un descripteur.

Exemple : `FooBot`

Un descripteur de robot représente une inscription de robot auprès de Bot Framework en ligne. Cette inscription est associée à un point de terminaison de webhook HTTP et à des inscriptions auprès de canaux.

Le portail de développement de Bot Framework garantit l’unicité des descripteurs de robot. Le portail effectue une vérification de l’unicité ne tenant pas compte de la casse (ce qui signifie que les variations d’un descripteur de robot sont traitées comme s’il s’agissait d’un même descripteur), bien qu’il s’agisse d’une caractéristique du portail de développement, et pas nécessairement du descripteur de robot proprement dit.

### <a name="rules-for-bot-handles"></a>Règles pour les descripteurs de robot

* Les descripteurs de robot sont uniques (sans tenir compte de la casse) au sein du Bot Framework.

## <a name="app-id"></a>ID d'application

Chaque robot inscrit auprès du Bot Framework a un ID d’application.

> [!NOTE]
> Auparavant, les applications étaient communément appelées « Applications MSA » ou « Applications MSA/AAD ». Les applications sont désormais plus généralement appelées simplement « applications », mais il se peut que certains éléments de protocole continuent à faire référence aux applications en tant qu’« Applications MSA » à perpétuité.

Exemple : `"msaAppId": "353826a6-4557-45f8-8d88-6aa0526b8f77"`

Une application représente une inscription auprès du portail d’application de l’équipe Identité, et sert de mécanisme d’identité de service à service dans le protocole de runtime de Bot Framework. Les applications peuvent avoir des associations autres que de robots, tels que des sites web et des applications mobiles ou de bureau.

Chaque robot inscrit a exactement une application. Bien qu’il soit impossible pour le propriétaire d’un robot de modifier de façon indépendante l’application associée à son robot, l’équipe Bot Framework peut faire cela dans un petit nombre de cas exceptionnels.

Les robots et les canaux peuvent utiliser des ID d’application pour identifier de façon unique des robots inscrits.

Il est garanti que les ID d’application sont des GUID. Les ID d’application doivent être comparés sans respect de la casse.

### <a name="rules-for-app-ids"></a>Règles pour les ID d’application

* Les ID d’application sont uniques (comparaison de GUID) au sein de la plateforme d’application Microsoft.
* Chaque robot a exactement une application correspondante.
* La modification de l’application à laquelle un robot est associé nécessite l’assistance de l’équipe Bot Framework.

## <a name="channel-account"></a>Compte de canal

Chaque robot et utilisateur a un compte au sein de chaque canal. Le compte contient un identificateur (`id`) et d’autres données non structurelles informatives sur robot, en tant que nom facultatif.

Exemple : `"from": { "id": "john.doe@contoso.com", "name": "John Doe" }`

Ce compte décrit l’adresse à l’intérieur du canal dans lequel des messages peuvent être envoyés et reçus. Dans certains cas, ces inscriptions existent au sein d’un seul service (par exemple, Skype, Facebook). Dans d’autres cas, elles existent dans de nombreux systèmes (adresses e-mail, numéros de téléphone). Dans des canaux plus anonymes (par exemple, Discussion Web), l’inscription peut être éphémère.

Les comptes de canal sont imbriqués dans des canaux. Un compte Facebook, par exemple, n’est qu’un nombre. Ce nombre peut avoir une signification différente dans d’autres canaux, et n’a aucune signification en dehors des canaux.

La relation entre des comptes de canal et des utilisateurs (personnes réelles) dépend des conventions associées à chaque canal. Par exemple, un numéro de SMS fait généralement référence à une personne pendant une période de temps à l’issue de laquelle le numéro peut être transféré à quelqu’un d’autre. Inversement, un compte Facebook fait généralement référence à une personne à perpétuité, même s’il n’est pas rare que deux personnes partagent un même compte Facebook.

Dans la plupart des canaux, il convient de considérer un compte de canal comme une sorte de boîte aux lettres à laquelle des messages peuvent être remis. La plupart des canaux autorisent généralement le mappage de plusieurs adresses à une boîte aux lettres unique. Par exemple, « jdoe@contoso.com » et « john.doe@service.contoso.com » peuvent correspondre à la même boîte de réception. Certains canaux vont plus loin en modifiant l’adresse du compte en fonction du robot qui y accède. Par exemple, Skype et Facebook modifient les ID utilisateur de façon à ce que chaque robot ait une adresse distincte pour l’envoi et la réception de messages.

Bien qu’il soit possible dans certains cas d’établir une équivalence entre des adresses, l’établissement d’une équivalence entre des boîtes aux lettres ainsi qu’entre des personnes nécessite une connaissance des conventions au sein du canal, ce qui, dans de nombreux cas, n’est pas possible.

Un robot est informé de son adresse de compte de canal via le champ `recipient` sur les activités qui lui sont envoyées.

### <a name="rules-for-channel-accounts"></a>Règles pour les comptes de canal

* Les comptes de canal n’ont de signification qu’au sein du canal qui leur est associé.
* Plusieurs ID peuvent correspondre au même compte.
* Une comparaison ordinale permet d’établir que deux ID sont identiques.
* Il n’existe généralement pas de comparaison utilisable pour déterminer si deux ID distincts correspondent au même compte, au même robot ou à la même personne.
* La stabilité des associations entre des ID, comptes, boîtes aux lettres et personnes dépend du canal.

## <a name="conversation-id"></a>ID de conversation

Des messages sont envoyés et reçus dans le contexte d’une conversation identifiable par un ID.

Exemple : `"conversation": { "id": "1234" }`

Une conversation contient un échange de messages et d’autres activités. Chaque conversation comporte zéro ou plusieurs activités, et chaque activité n’apparaît que dans une seule conversation. Des conversations peuvent être perpétuelles, ou avoir des débuts et des fins distinctes. Le processus de création, de modification ou de terminaison d’une conversation se produit à l’intérieur du canal (c’est-à-dire qu’une conversation existe quand le canal en est informé) et les caractéristiques de ces processus sont établies par le canal.

Les activités au sein d’une conversation sont envoyées par des utilisateurs et des robots. La définition des utilisateurs qui « participent » à une conversation varie selon le canal, et peut inclure théoriquement des utilisateurs présents, des utilisateurs qui ont reçu un message, et des utilisateurs qui ont envoyé un message.

Quelques canaux (par exemple, SMS, Skype, voire d’autres) présentent la singularité que l’ID de conversation affecté à une conversation de 1 à 1 est l’ID de compte du canal distant. Cette singularité a deux effets secondaires :
1. L’ID de conversation est subjectif selon la personne qui le considère. Si des participants A et B discutent, le participant A voit l’ID de conversation « B », et le participant B l’ID de conversation « A ».
2. Si le robot compte plusieurs comptes de canal au sein de ce canal (par exemple, si le robot a deux numéros de SMS), l’ID de conversation n’est pas suffisant pour identifier de manière unique la conversation dans le champ de vision du bot.

Par conséquent, un ID de conversation n’identifie pas nécessairement de manière unique une conversation au sein d’un canal, même pour un seul robot.

### <a name="rules-for-conversation-ids"></a>Règles pour les ID de conversation

* Les conversations n’ont de signification qu’au sein du canal qui leur est associé.
* Plusieurs ID peut correspondre à une même conversation.
* L’égalité ordinale n’établit pas nécessairement que deux ID de conversation correspondent à la même conversation, même si c’est le plus souvent le cas.

## <a name="activity-id"></a>ID d’activité

Des activités sont envoyées et reçues dans le protocole Bot Framework, et elles sont parfois identifiables.

Exemple : `"id": "5678"`

Les ID d’activité sont facultatifs et utilisés par les canaux pour donner au robot un moyen référencer l’ID dans des appels d’API subséquents, s’ils sont disponibles :
* Réponse à une activité particulière
* Interrogation de la liste des participants au niveau d’une activité

Aucun autre cas d’utilisation n’étant établi, il n’existe pas d’autres règles pour le traitement des ID d’activité.
