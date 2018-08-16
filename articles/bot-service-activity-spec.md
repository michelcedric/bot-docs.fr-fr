---
title: Spécification de Bot Framework | Microsoft Docs
description: Spécification de Bot Framework
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 03/07/2018
ms.openlocfilehash: 0406d489f7d1e27131b4b01411e86850ca4a17b8
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39298619"
---
# <a name="bot-framework----activity"></a>Bot Framework -- Activité

## <a name="abstract"></a>Résumé

Le schéma d’activité de Bot Framework est une représentation au niveau de l’application d’actions conversationnelles effectuées par des humains et des logiciels automatisés. Il inclut des dispositions pour la communication d’actions multimédias, textuelles et sans contenu, telles que des interactions sociales et des indicateurs de saisie.

Ce schéma est utilisé dans le protocole Bot Framework et est implémenté par les systèmes de conversation Microsoft et par les clients et bots interopérables de nombreuses sources.

## <a name="table-of-contents"></a>Sommaire

1. [Introduction](#introduction)
2. [Structure de base de l’activité](#basic-activity-structure)
3. [Activité de message](#message-activity)
4. [Activité de mise à jour de relation de contact](#contact-relation-update-activity)
5. [Activité de mise à jour de conversation](#conversation-update-activity)
6. [Activité de fin de conversation](#end-of-conversation-activity)
7. [Activité d’événement](#event-activity)
8. [Activité d’appel](#invoke-activity)
9. [Activité de mise à jour d’installation](#installation-update-activity)
10. [Activité de suppression de message](#message-delete-activity)
11. [Activité de mise à jour de message](#message-update-activity)
12. [Activité de réaction à un message](#message-reaction-activity)
13. [Activité de saisie](#typing-activity)
14. [Types complexes](#complex-types)
15. [Informations de référence](#references)
16. [Annexe I - Changements](#appendix-i---changes)
17. [Annexe II - Types d’entités non-IRI](#appendix-ii---non-iri-entity-types)
18. [Annexe III - Protocoles utilisant l’activité invoke](#appendix-iii---protocols-using-the-invoke-activity)

## <a name="introduction"></a>Introduction

### <a name="overview"></a>Vue d’ensemble

Le schéma d’activité de Bot Framework représente des comportements conversationnels entre des humains et des logiciels automatisés dans des applications de conversation, de messagerie et d’autres programmes d’interaction textuelle. Chaque objet d’activité comprend un champ de type et représente une action unique : en règle générale l’envoi de contenu textuel, mais également l’inclusion de pièces jointes multimédias et des comportements sans contenu comme un bouton « J’aime » ou un indicateur de saisie.

Ce document explique la signification de chaque type d’activité et décrit les champs obligatoires et facultatifs qui peuvent être inclus. Il définit également les rôles du client et du serveur, et fournit des conseils concernant les champs qui sont contrôlés par chaque participant et ceux qui peuvent être ignorés.

Il existe trois rôles important dans cette spécification : les clients (qui envoient et reçoivent des activités pour le compte d’utilisateurs), les bots (qui envoient et reçoivent des activités et sont généralement automatisés) et le canal (qui stocke et transfère les activités entre les clients et les bots).

Bien que cette spécification nécessite la transmission d’activités entre les rôles, la nature exacte de cette transmission n’est pas décrite ici.

Par souci de compacité, les cartes interactives visuelles ne sont pas définies dans cette spécification. Pour plus d’informations à ce sujet, consultez la spécification [Cartes adaptatives](https://adaptivecards.io) [[11](#references)]. Cette carte, ainsi que d’autres types de cartes non définis, peuvent être inclus en tant que pièces jointes dans les activités Bot Framework.

### <a name="requirements"></a>Configuration requise

Les mots clés « DOIT », « NE DOIT PAS », « OBLIGATOIRE », « DEVRA », « NE DEVRA PAS », « DEVRAIT », « NE DEVRAIT PAS », « RECOMMANDÉ », « NON RECOMMANDÉ », « PEUT » et « FACULTATIF » figurant dans ce document doivent être interprétés de la manière décrite dans le document [RFC 2119](https://tools.ietf.org/html/rfc2119) [[1](#references)].

Une implémentation n’est pas conforme si elle ne répond pas à une ou plusieurs des exigences de niveau DOIT ou OBLIGATOIRE pour les protocoles qu’elle implémente. Une implémentation répondant à toutes les exigences de niveau DOIT ou OBLIGATOIRE et à toutes les exigences de niveau DEVRAIT pour ses protocoles est dite « conforme sans condition ». Une implémentation répondant à toutes les exigences de niveau DOIT mais pas à toutes les exigences de niveau DEVRAIT pour ses protocoles est dite « conforme sous condition ».

### <a name="numbered-requirements"></a>Exigences numérotées

Les lignes commençant par des marqueurs au format `RXXXX` sont des exigences spécifiques conçues pour être référencées par un chiffre dans des discussions en dehors de ce document. Vous ne devez pas leur accorder plus, ni moins, de poids que les déclarations normatives effectuées en dehors des lignes `RXXXX`.

`R1000` : les éditeurs de cette spécification PEUVENT ajouter de nouvelles exigences `RXXXX`. Ils DEVRAIENT trouver des valeurs `RXXXX` numériques qui préservent le flux du document.

`R1001` : les éditeurs NE DOIVENT PAS renuméroter les exigences `RXXXX` existantes.

`R1002` : les éditeurs PEUVENT supprimer ou réviser des exigences `RXXXX`. En cas de révision, les éditeurs DEVRAIENT conserver la valeur `RXXXX` existante si la rubrique de l’exigence demeure en grande partie intacte.

`R1003` : les éditeurs NE DEVRAIENT PAS réutiliser de valeurs `RXXXX` retirées. Une liste de valeurs supprimées PEUT être tenue à jour à la fin de ce document.

### <a name="terminology"></a>Terminologie

activity
> Action exprimée par un bot, un canal ou un client qui est conforme au schéma d’activité.

channel
> Logiciel qui envoie et reçoit des activités, et les transforme vers et à partir de comportements de conversation ou d’application. Les canaux sont le magasin faisant autorité pour les données d’activités.

bot
> Logiciel qui envoie et reçoit des activités, et génère des réponses automatisées, semi-automatisées ou entièrement manuelles. Les bots ont des points de terminaison qui sont inscrits auprès de canaux.

client
> Logiciel qui envoie et reçoit des activités, généralement pour le compte d’utilisateurs humains. Les clients n’ont pas de points de terminaison.

expéditeur
> Logiciel qui transmet une activité.

récepteur
> Logiciel qui accepte une activité.

endpoint
> Emplacement adressable par programmation où un bot ou un canal peut recevoir des activités.

address
> Identificateur ou adresse où un utilisateur ou un bot peut être contacté.

field
> Valeur nommée dans un objet d’activité ou un objet imbriqué.

### <a name="overall-organization"></a>Organisation globale

L’objet d’activité est une liste plate de paires nom/valeur, certains d’entre eux étant des objets primitifs et d’autres des objets complexes (imbriqués). L’objet d’activité est souvent exprimé au format JSON, mais peut également être projeté dans des structures de données en mémoire en .Net ou JavaScript.

Le champ `type` d’activité contrôle la signification de l’activité et les champs qu’elle contient. En fonction du rôle assumé par un participant (client, bot ou canal), chaque champ est obligatoire, facultatif ou ignoré. Par exemple, le champ `id` est contrôlé par le canal, et il est obligatoire dans certains cas, mais ignoré s’il est envoyé par un bot.

Les champs qui décrivent l’identité de l’activité et les participants, tels que les champs `type` et `from`, sont partagés par toutes les activités. Dans de nombreux langages de programmation, il est plus commode d’organiser ces champs sur un type de base principal à partir duquel d’autres types d’activités plus précis dérivent.

Lors du stockage ou de la transmission d’activités, certains champs peuvent être dupliqués dans le mécanisme de transport. Par exemple, si une activité est transmise par le biais de HTTP POST à une URL qui inclut l’ID de conversation, le récepteur peut déduire sa valeur sans exiger sa présence dans le corps de l’activité. Ce document décrit simplement les exigences abstraites pour ces champs, et c’est au protocole de contrôle d’établir si les valeurs doivent être déclarées explicitement ou si les valeurs implicites ou déduites sont autorisées.

Quand un bot ou un client envoie une activité à un canal, il s’agit généralement d’une demande pour que l’activité soit enregistrée. Quand un canal envoie une activité à un bot ou à un client, il s’agit généralement d’une notification indiquant que l’activité a déjà été enregistrée.

## <a name="basic-activity-structure"></a>Structure de base de l’activité

Cette section définit les exigences pour la structure de base de l’objet d’activité.

Les objets d’activités incluent une liste plate de paires nom/valeur, appelées champs. Les champs peuvent être des types primitifs. JSON est utilisé comme format d’échange commun, et bien que toutes les activités ne doivent pas toujours et obligatoirement être sérialisées au format JSON, elles doivent être sérialisables. Cela permet aux implémentations de s’appuyer sur un ensemble simple de conventions pour la gestion des champs d’activités connus et inconnus.

`R2001` : les activités DOIVENT être sérialisables au format JSON, notamment quant au respect des contraintes d’unicité de champ.

`R2002` : les récepteurs PEUVENT autoriser les noms de champs de casse incorrecte, bien que cela ne soit pas obligatoire. Les récepteurs PEUVENT rejeter les activités qui n’incluent pas des champs de la casse appropriée.

`R2003` : les récepteurs PEUVENT rejeter les activités qui contiennent des valeurs de champs dont les types ne correspondent pas aux types de valeurs décrits dans cette spécification.

`R2004` : sauf indication contraire, les expéditeurs NE DEVRAIENT PAS inclure de valeurs de chaînes vides pour les champs de chaînes.

`R2005` : sauf indication contraire, les expéditeurs PEUVENT inclure des champs supplémentaires au sein de l’activité ou de tout objet complexe imbriqué. Les récepteurs DOIVENT accepter les champs qu’ils ne comprennent pas.

`R2006` : les récepteurs DEVRAIENT accepter les événements de types qu’ils ne comprennent pas.

### <a name="type"></a>type

Le champ `type` contrôle la signification de chaque activité. Il s’agit par convention d’une chaîne courte (par exemple, « `message` »). Les expéditeurs peuvent définir leurs propres types de couche application, bien qu’ils soient invités à choisir des valeurs qui sont peu susceptibles d’entrer en conflit avec des valeurs futures bien définies. Si des expéditeurs utilisent des URI comme valeurs de type, ils NE DEVRAIENT PAS implémenter de comparaisons d’échelle d’URI pour établir une équivalence.

`R2010` : les activités DOIVENT inclure un champ `type`, avec type de valeur de chaîne.

`R2011` : deux valeurs `type` sont équivalentes uniquement si elles sont identiques du point de vue ordinal.

`R2012` : un expéditeur PEUT générer des valeurs `type` d’activité non définies dans ce document.

`R2013` : un canal DEVRAIT rejeter les activités de type qu’il ne comprend pas.

`R2014` : un client ou un bot DEVRAIT ignorer les activités de type qu’il ne comprend pas.

### <a name="channel-id"></a>ID de canal

Le champ `channelId` établit le canal et le magasin faisant autorité pour l’activité. La valeur du champ `channelId` est de type chaîne.

`R2020` : les activités DOIVENT inclure un champ `channelId`, avec type de valeur de chaîne.

`R2021` : deux valeurs `channelId` sont équivalentes uniquement si elles sont identiques du point de vue ordinal.

`R2022` : un canal PEUT ignorer ou rejeter toute activité qu’il reçoit sans valeur `channelId` attendue.

### <a name="id"></a>ID

Le champ `id` établit l’identité de l’activité une fois qu’elle a été enregistrée dans le canal. Les activités en cours qui n’ont pas encore été enregistrées n’ont pas d’identités. Des identités ne sont pas affectées à toutes les activités (par exemple, un `id` ne peut jamais être attribué à une [activité de saisie](#typing-activity).) La valeur du champ `id` est de type chaîne.

`R2030` : les canaux DEVRAIENT inclure un champ `id` s’il est disponible pour cette activité.

`R2031` : les clients et les bots NE DEVRAIENT PAS inclure de champ `id` dans les activités qu’ils génèrent.

Pour faciliter l’implémentation, il doit être supposé que les autres participants n’ont pas une connaissance sophistiquée des ID d’activités, et qu’ils utiliseront uniquement une comparaison ordinale pour établir l’équivalence.

Par exemple, un canal peut utiliser des GUID codés en hexadécimal pour chaque ID d’activité. Bien que les GUID encodés en majuscules soient logiquement équivalents aux GUID encodés en minuscules, dans la mesure du possible les expéditeurs NE DEVRAIENT PAS utiliser ces autres encodages. La version normalisée de chaque ID est établie par le magasin faisant autorité, le canal.

`R2032` : lors de la génération de valeurs `id`, les expéditeurs DEVRAIENT choisir des valeurs dont l’équivalence peut être établie par la comparaison ordinale. Toutefois, les expéditeurs et les récepteurs PEUVENT autoriser une équivalence logique de deux valeurs qui ne sont pas ordinalement équivalentes s’ils ont une connaissance spéciale des circonstances.

Le champ `id` est conçu pour autoriser la déduplication, mais ceci est prohibitif dans la plupart des applications.

`R2033` : les récepteurs PEUVENT dédupliquer les activités par ID, mais les expéditeurs NE DEVRAIENT PAS se fier aux récepteurs pour l’exécution de cette déduplication.

### <a name="timestamp"></a>Timestamp

Le champ `timestamp` enregistre l’heure UTC exacte à laquelle l’activité s’est produite. En raison de la nature distribuée des systèmes informatiques, l’heure importante est celle à laquelle le canal (le magasin faisant autorité) enregistre l’activité. L’heure à laquelle un client ou un bot a démarré une activité peut être transmise séparément dans le champ `localTimestamp`. La valeur du champ `timestamp` est une date/heure encodée au format [ISO 8601](https://www.iso.org/iso-8601-date-and-time-format.html) [[2](#references)] dans une chaîne.

`R2040` : les canaux DEVRAIENT inclure un champ `timestamp` s’il est disponible pour cette activité.

`R2041` : les clients et les bots NE DEVRAIENT PAS inclure de champ `timestamp` dans les activités qu’ils génèrent.

`R2042` : les clients et les bots NE DEVRAIENT PAS utiliser `timestamp` pour rejeter des activités, car elles peuvent apparaître dans le désordre. En revanche, ils PEUVENT utiliser `timestamp` pour mettre en ordre des activités dans une interface utilisateur ou pour le traitement en aval.

`R2043` : les expéditeurs DEVRAIENT toujours encoder la valeur de `timestamp` au format UTC, et ils DEVRAIENT toujours inclure Z comme marque UTC explicite dans la valeur.

### <a name="local-timestamp"></a>Horodateur local

Le champ `localTimestamp` exprime la date/heure et le décalage de fuseau horaire où l’activité a été générée. Il peut être différent du `timestamp` UTC où l’activité a été enregistrée. La valeur du champ `localTimestamp` est une date/heure encodée au format ISO 8601 [[3](#references)] dans une chaîne.

`R2050` : les clients et les bots PEUVENT inclure le champ `localTimestamp` dans leurs activités. Ils DEVRAIENT répertorier explicitement le décalage de fuseau horaire dans la valeur encodée.

`R2051` : les canaux DEVRAIENT préserver `localTimestamp` lors du transfert des activités d’un expéditeur au(x) récepteur(s).

### <a name="from"></a>À partir

Le champ `from` décrit quel client, bot, ou canal a généré une activité. La valeur du champ `from` est un objet complexe du type [Compte de canal](#channel-account).

Le champ `from.id` identifie qui a généré une activité. Il s’agit le plus souvent d’un autre utilisateur ou bot au sein du système. Dans certains cas, le champ `from` identifie le canal lui-même.

`R2060` : les canaux DOIVENT inclure les champs `from` et `from.id` lors de la génération d’une activité.

`R2061` : les clients et les bots DEVRAIENT inclure les champs `from` et `from.id` lors de la génération d’une activité. Un canal PEUT rejeter une activité à cause de l’absence des champs `from` et `from.id`.

Le champ `from.name` est facultatif et représente le nom d’affichage du compte dans le canal. Les canaux DEVRAIENT inclure cette valeur afin que les clients et les bots puissent peupler leurs interfaces utilisateur et systèmes backend. Les bots et les clients NE DEVRAIENT PAS envoyer cette valeur à des canaux ayant un enregistrement central de ce magasin, mais ils PEUVENT envoyer cette valeur à des canaux qui peuplent la valeur sur chaque activité (par exemple, un canal par e-mail).

`R2062` : les canaux DEVRAIENT inclure le champ `from.name` si le champ `from` est présent et que `from.name` est disponible.

`R2063` : les bots et les clients NE DEVRAIENT PAS inclure le champ `from.name`, sauf s’il est sémantiquement précieux dans le canal.

### <a name="recipient"></a>Recipient

Le champ `recipient` décrit quel client ou bot reçoit cette activité. Ce champ n’est significatif que quand une activité est transmise à exactement un seul récepteur ; il n’est pas significatif quand il est diffusé à plusieurs récepteurs (comme c’est le cas quand une activité est envoyée à un canal). L’objectif de ce champ est de permettre au récepteur de s’identifier. Ceci est utile quand un client ou un bot a plus d’une identité dans le canal. La valeur du champ `recipient` est un objet complexe du type [Compte de canal](#channel-account).

`R2070` : les canaux doivent inclure les champs `recipient` et `recipient.id` lors de la transmission d’une activité à un seul récepteur.

`R2071` : les bots et les clients NE DEVRAIENT PAS inclure le champ `recipient` lors de la génération d’une activité.

Le champ `recipient.name` est facultatif et représente le nom d’affichage du compte dans le canal. Les canaux DEVRAIENT inclure cette valeur afin que les clients et les bots puissent peupler leurs interfaces utilisateur et systèmes backend.

`R2072` : les canaux DEVRAIENT inclure le champ `recipient.name` si le champ `recipient` est présent et que `recipient.name` est disponible.

### <a name="conversation"></a>Conversation

Le champ `conversation` décrit la conversation dans laquelle l’activité existe. La valeur du champ `conversation` est un objet complexe du type [Compte de conversation](#conversation-account).

`R2080` : les canaux, les bots et les clients DOIVENT inclure les champs `conversation` et `conversation.id` lors de la génération d’une activité.

Le champ `conversation.name` est facultatif et représente le nom d’affichage de la conversation s’il existe et est disponible.

`R2081` : les canaux doivent inclure les champs `conversation.name` et `conversation.isGroup` s’ils sont disponibles.

`R2082` : les bots et les clients NE DEVRAIENT PAS inclure le champ `conversation.name`, sauf s’il est sémantiquement précieux dans le canal.

`R2083` : les bots et les clients NE DEVRAIENT PAS inclure le champ `conversation.isGroup` dans les activités qu’ils génèrent.

### <a name="reply-to-id"></a>Réponse à l’ID

Le champ `replyToId` identifie l’activité précédente à laquelle l’activité actuelle est une réponse. Ce champ autorise la communication des fils de conversation et de l’imbrication des commentaires parmi les participants. `replyToId` est valide uniquement dans la conversation active. (Pour les références à d’autres conversations, consultez [relatesTo](#relates-to).) La valeur du champ `replyToId` est une chaîne.

`R2090` : les expéditeurs DEVRAIENT inclure `replyToId` sur une activité quand il s’agit d’une réponse à une autre activité.

`R2091` : un canal PEUT rejeter une activité si son `replyToId` ne référence pas une activité valide au sein de la conversation.

`R2092` : les bots et les clients PEUVENT omettre `replyToId` s’ils savent que le canal n’utilise pas le champ, même si l’activité envoyée est une réponse à une autre activité.

### <a name="entities"></a>Entités

Le champ `entities` contient une liste plate d’objets de métadonnées se rapportant à cette activité. Contrairement aux pièces jointes (voir le champ [attachments](#attachments)), les entités ne se manifestent pas nécessairement en tant qu’éléments de contenu avec interaction utilisateur, et elles sont destinées à être ignorées si elles ne sont pas comprises. Les expéditeurs peuvent inclure des entités qu’ils pensent être utiles à un récepteur, même s’ils ne sont pas certains que le récepteur peut les accepter. La valeur de chaque élément de liste `entities` est un objet complexe du type [Entité](#entity).

`R2100` : les expéditeurs DEVRAIENT omettre le champ `entities` s’il ne contient aucun élément.

`R2101` : les expéditeurs PEUVENT envoyer plusieurs entités du même type, à condition qu’elles aient une signification distincte.

`R2102` : les expéditeurs NE DOIVENT PAS inclure plusieurs entités avec des types et du contenu identiques.

`R2103` : les expéditeurs et les récepteurs NE DEVRAIENT PAS se reposer sur un ordonnancement spécifique des entités incluses dans une activité.

### <a name="channel-data"></a>Données de canal

Les données d’extensibilité dans le schéma d’activité sont organisées principalement dans le champ `channelData`. Cela simplifie la plomberie dans les SDK qui implémentent le protocole. Le format de l’objet `channelData` est défini par le canal qui envoie ou reçoit l’activité.

`R2200` : les canaux NE DEVRAIENT PAS définir de formats `channelData` qui sont des primitives JSON (par exemple des chaînes ou des entiers). Au lieu de cela, ils DEVRAIENT définir `channelData` en tant que type complexe, ou le laisser non défini.

`R2201` : si le format `channelData` n’est pas défini pour le canal actif, les récepteurs DEVRAIENT ignorer le contenu de `channelData`.

### <a name="service-url"></a>URL du service

Les activités sont fréquemment envoyées de manière asynchrone, avec des connexions de transport distinctes pour envoyer et recevoir le trafic. Le champ `serviceUrl` est utilisé par les canaux pour désigner l’URL où les réponses à l’activité en cours peuvent être envoyées. La valeur du champ `serviceUrl` est de type chaîne.

`R2300` : les canaux DOIVENT inclure le champ `serviceUrl` dans toutes les activités qu’ils envoient aux bots.

`R2301` : les canaux NE DEVRAIENT PAS envoyer le champ `serviceUrl` aux clients qui démontrent qu’ils connaissent déjà le point de terminaison du canal.

`R2302` : les bots et les clients NE DEVRAIENT PAS renseigner le champ `serviceUrl` dans les activités qu’ils génèrent.

`R2302` : les canaux DOIVENT ignorer la valeur de `serviceUrl` dans les activités envoyées par les clients et les bots.

`R2304` : les canaux DEVRAIENT utiliser des valeurs stables pour le champ `serviceUrl`, car les bots peuvent les rendre persistantes pendant de longues périodes.

## <a name="message-activity"></a>Activité de message

Les activités de message représentent le contenu destiné à être présenté dans une interface de conversation. Elles peuvent contenir du texte, de la parole, des cartes interactives et des pièces jointes binaires ou inconnues. En général, les canaux exigent au plus un de ces éléments pour que l’activité de message soit construite correctement.

Les activités de message sont identifiées par une valeur `type` `message`.

### <a name="text"></a>Texte

Le champ `text` contient du texte, au format Markdown, XML, ou en tant que texte brut. Le format est contrôlé par le champ [`textFormat`](#text-format) comme c’est normal s’il est non spécifié ou ambigu. La valeur du champ `text` est de type chaîne.

`R3000` : le champ `text` PEUT contenir une chaîne vide pour indiquer l’envoi de texte sans contenu.

`R3001` : les canaux DEVRAIENT gérer le texte au format `markdown` d’une manière qui se dégrade normalement pour ce canal.

### <a name="text-format"></a>Format Texte

Le champ `textFormat` indique si le champ [`text`](#text) doit être interprété comme [Markdown](https://daringfireball.net/projects/markdown/) [[4](#references)], texte brut ou XML. La valeur du champ `textFormat` est de type chaîne, avec des valeurs définies `markdown`, `plain` et `xml`. La valeur par défaut est `plain`. Ce champ n’est pas conçu pour être étendu avec des valeurs arbitraires.

Le champ `textFormat` contrôle des champs supplémentaires dans les pièces jointes, et ainsi de suite. Cette relation est décrite dans ces champs, ailleurs dans ce document.

`R3010` : si un expéditeur inclut le champ `textFormat`, il DEVRAIT envoyer uniquement des valeurs définies.

`R3011` : les expéditeurs DEVRAIENT omettre `textFormat` si la valeur est `plain`.

`R3012` : les récepteurs DEVRAIENT interpréter les valeurs non définies en tant que `plain`.

`R3013` : les bots et les clients NE DEVRAIENT PAS envoyer la valeur `xml`, sauf s’ils savent auparavant que le canal prend en charge cette valeur et les caractéristiques du dialecte XML pris en charge.

`R3014` : les canaux NE DEVRAIENT PAS envoyer de contenu `markdown` ou `xml` aux bots.

`R3015` : les canaux DEVRAIENT au moins accepter les valeurs `textformat` `plain` et `markdown`.

`R3016` : les canaux PEUVENT rejeter `textformat` de valeur `xml`.

### <a name="locale"></a>Paramètres régionaux

Le champ `locale` communique le code de langue du champ [`text`](#text). La valeur du champ `locale` est un code au format [ISO 639](https://www.iso.org/iso-639-language-codes.html) [[5](#references)] dans une chaîne.

`R3020` : les récepteurs DEVRAIENT traiter les valeurs manquantes et inconnues du champ `locale` comme inconnues.

`R3021` : les récepteurs NE DEVRAIENT PAS rejeter les activités dont les paramètres régionaux sont inconnus.

### <a name="speak"></a>Speak

Le champ `speak` indique la façon dont l’activité doit être prononcée par le biais d’un système de synthèse vocale. Il est utilisé uniquement pour personnaliser le rendu de reconnaissance vocale quand la valeur par défaut est considérée comme inadéquate. Il remplace la synthèse vocale pour tout contenu dans l’activité, notamment le texte, les pièces jointes et les résumés. La valeur du champ `speak` est codée au format [SSML](https://www.w3.org/TR/speech-synthesis/) [[6](#references)] dans une chaîne. Le texte non encapsulé est autorisé et est automatiquement mis à niveau au format SSML nu.

`R3030` : le champ `speak` PEUT contenir une chaîne vide pour indiquer qu’aucune parole ne doit être générée.

`R3031` : les récepteurs incapables de générer de la parole DEVRAIENT ignorer le champ `speak`.

`R3032` : si aucun élément SSML racine n’est trouvé, les récepteurs DOIVENT considérer le texte inclus comme étant le contenu interne d’une balise `<speak>` SSML, précédé d’un [Prologue XML](https://www.w3.org/TR/xml/#sec-prolog-dtd) [[7](#references)] valide. Sinon, il doit être mis à niveau pour être un document SSML valide.

`R3033` : les récepteurs NE DEVRAIENT PAS utiliser de résolution de schéma ou DTD XML pour inclure des ressources distantes situées à l’extérieur du fragment XML communiqué.

`R3034` : les canaux NE DEVRAIENT PAS envoyer le champ `speak` aux bots.

### <a name="input-hint"></a>Indicateur d’entrée

Le champ `inputHint` indique si le générateur de l’activité attend une réponse. Ce champ est utilisé principalement dans les canaux qui ont des interfaces utilisateur modales. Il n’est généralement pas utilisé dans les canaux avec des flux de conversation continus. La valeur du champ `inputHint` est de type chaîne, avec des valeurs définies `accepting`, `expecting` et `ignoring`. La valeur par défaut est `accepting`.

`R3040` : si un expéditeur inclut le champ `inputHint`, il DEVRAIT envoyer uniquement des valeurs définies.

`R3041` : en cas d’envoi d’une activité à un canal où `inputHint` est utilisé, les bots DEVRAIENT inclure le champ, même quand la valeur est `accepting`.

`R3042` : les récepteurs DEVRAIENT interpréter les valeurs non définies en tant que `accepting`.

### <a name="attachments"></a>Pièces jointes

Le champ `attachments` contient une liste plate d’objets à afficher dans le cadre de cette activité. La valeur de chaque élément de liste `attachments` est un objet complexe du type [Pièce jointe](#attachment).

`R3050` : les expéditeurs DEVRAIENT omettre le champ `attachments` s’il ne contient aucun élément.

`R3051` : les expéditeurs PEUVENT envoyer plusieurs entités du même type.

`R3052` : les récepteurs PEUVENT traiter les pièces jointes de types inconnus en tant que documents téléchargeables.

`R3053` : les récepteurs DEVRAIENT conserver l’ordre des pièces jointes lors du traitement du contenu, sauf quand les limites de rendu imposent des modifications, par exemple le regroupement de documents après des images.

### <a name="attachment-layout"></a>Disposition des pièces jointes

Le champ `attachmentLayout` indique aux convertisseurs d’interface utilisateur comment présenter le contenu inclus dans le champ [`attachments`](#attachments). La valeur du champ `attachmentLayout` est de type chaîne, avec des valeurs définies `list` et `carousel`. La valeur par défaut est `list`.

`R3060` : si un expéditeur inclut le champ `attachmentLayout`, il DEVRAIT envoyer uniquement des valeurs définies.

`R3061` : les récepteurs DEVRAIENT interpréter les valeurs non définies en tant que `list`.

### <a name="summary"></a>Résumé

Le champ `summary` contient du texte utilisé pour remplacer les [`attachments`](#attachments) sur les canaux qui ne les prennent pas en charge. La valeur du champ `summary` est de type chaîne.

`R3070` : les récepteurs DEVRAIENT considérer le champ `summary` comment suivant logiquement le champ `text`.

`R3071` : les canaux NE DEVRAIENT PAS envoyer le champ `summary` aux bots.

`R3072` : les canaux capables de traiter toutes les pièces jointes dans une activité doivent ignorer le champ `summary`.

### <a name="suggested-actions"></a>Actions suggérées

Le champ `suggestedActions` contient une charge utile d’actions interactives qui peuvent être présentées à l’utilisateur. La prise en charge des `suggestedActions` et de leur manifestation dépend en grande partie du canal. La valeur du champ `suggestedActions` est un objet complexe du type [Actions suggérées](#suggested-actions-2).

### <a name="value"></a>Valeur

Le champ `value` contient une charge utile de programmation propre à l’activité envoyée. Sa signification et son format sont définis dans d’autres sections de ce document décrivant son utilisation.

`R3080` : les expéditeurs NE DEVRAIENT PAS inclure de champs `value` de types primitifs (par exemple, string, int). Les champs `value` DEVRAIENT être des types complexes ou omis.

### <a name="expiration"></a>Expiration

Le champ `expiration` contient une heure à laquelle l’activité doit être considérée comme ayant « expiré » et ne doit pas être présentée au récepteur. La valeur du champ `expiration` est une date/heure encodée au format [ISO 8601](https://www.iso.org/iso-8601-date-and-time-format.html) [[2](#references)] dans une chaîne.

`R3090` : les expéditeurs DEVRAIENT toujours encoder la valeur de `expiration` au format UTC, et ils DEVRAIENT toujours inclure Z comme marque UTC explicite dans la valeur.

### <a name="importance"></a>importance

Le champ `importance` contient un jeu énuméré de valeurs pour signaler au récepteur l’importance relative de l’activité.  Il incombe au récepteur de mapper ces indicateurs d’importance à l’expérience utilisateur. La valeur du champ `importance` est de type chaîne, avec des valeurs définies `low`, `normal` et `high`. La valeur par défaut est `normal`.

`R3100` : si un expéditeur inclut le champ `importance`, il DEVRAIT envoyer uniquement des valeurs définies.

`R3101` : les récepteurs DEVRAIENT interpréter les valeurs non définies en tant que `normal`.

### <a name="deliverymode"></a>Mode de remise

Le champ `deliveryMode` contient un jeu énuméré de valeurs pour signaler au récepteur d’autres chemins de remise de l’activité.  La valeur du champ `DeliveryMode` est de type chaîne, avec des valeurs définies `normal` et `notification`. La valeur par défaut est `normal`.

`R3110` : si un expéditeur inclut le champ `deliveryMode`, il DEVRAIT envoyer uniquement des valeurs définies.

`R3111` : les récepteurs DEVRAIENT interpréter les valeurs non définies en tant que `normal`.

## <a name="contact-relation-update-activity"></a>Activité de mise à jour de relation de contact

Les activités de mise à jour de relation de contact signalent une modification dans la relation entre le récepteur et un utilisateur dans le canal. En général, elles ne contiennent pas de contenu généré par l’utilisateur. La mise à jour de relation décrite par une activité de mise à jour de relation de contact existe entre l’utilisateur dans le champ `from` (souvent, mais pas toujours, l’utilisateur qui procède à la mise à jour) et l’utilisateur ou le bot dans le champ `recipient`.

Les activités de mise à jour de relation de contact sont identifiées par une valeur `type` `contactRelationUpdate`.

### <a name="action"></a>Action

Le champ `action` décrit la signification de l’activité de mise à jour de relation de contact. La valeur du champ `action` est une chaîne. Seules les valeurs de `add` et `remove` sont définies, ce qui indique une relation entre les utilisateurs/bots dans les champs `from` et `recipient`.

## <a name="conversation-update-activity"></a>Activité de mise à jour de conversation

Les activités de mise à jour de conversation décrivent un changement des membres, de la description, de l’existence, ou autres, d’une conversation. En général, elles ne contiennent pas de contenu généré par l’utilisateur. La conversation soumise à la mise à jour est décrite dans le champ `conversation`.

Les activités de mise à jour de conversation sont identifiées par une valeur `type` `conversationUpdate`.

`R4100` : les expéditeurs PEUVENT inclure zéro, un ou plusieurs des champs `membersAdded`, `membersRemoved`, `topicName` et `historyDisclosed` dans une activité de mise à jour de conversation.

`R4101` : chaque `channelAccount` (identifié par le champ `id`) doit apparaître au maximum une fois dans les champs `membersAdded` et `membersRemoved`. Un ID NE DEVRAIT PAS apparaître dans les deux champs. Un ID NE DEVRAIT PAS apparaître en double dans l’un ou l’autre champ.

`R4102` : les canaux NE DEVRAIENT PAS utilisent d’activités de mise à jour de conversation pour indiquer des modifications apportées aux champs d’un compte de canal (par exemple, `name`) si le compte de canal n’a été pas ajouté ou supprimé de la conversation.

`R4103` : les canaux NE DEVRAIENT PAS envoyer les champs `topicName` ou `historyDisclosed` si l’activité ne signale pas une modification de la valeur de l’un ou l’autre champ.

### <a name="members-added"></a>Membres ajoutés

Le champ `membersAdded` contient une liste de participants au canal (bots ou utilisateurs) ajoutés à la conversation. La valeur du `membersAdded` champ est un tableau de type [`channelAccount`](#channel-account).

### <a name="members-removed"></a>Membres supprimés

Le champ `membersRemoved` contient une liste de participants au canal (bots ou utilisateurs) supprimés de la conversation. La valeur du `membersRemoved` champ est un tableau de type [`channelAccount`](#channel-account).

### <a name="topic-name"></a>Nom de la rubrique

Le champ `topicName` contient la rubrique de texte ou la description de la conversation. La valeur du champ `topicName` est de type chaîne.

### <a name="history-disclosed"></a>Historique divulgué

Le champ `historyDisclosed` est déprécié.

`R4110` : les expéditeurs NE DEVRAIENT PAS inclure le champ `historyDisclosed`.

## <a name="end-of-conversation-activity"></a>Activité de fin de conversation

Les activités de fin de conversation signalent la fin d’une conversation du point de vue du récepteur. Cela peut être dû au fait que la conversation a été complètement terminée, ou au fait que le récepteur a été supprimé de la conversation d’une manière qui est indiscernable de la fin de conversation. La conversation qui prend fin est décrite dans le champ `conversation`.

Les activités de fin de conversation sont identifiées par une valeur `type` `endOfConversation`.

Les champs `code` et `text` sont tous deux facultatifs.

### <a name="code"></a>Code

Le champ `code` contient une valeur de programmation décrivant pourquoi ou comment la conversation a été terminée. La valeur du champ `code` est de type chaîne et sa signification est définie par le canal qui envoie l’activité.

### <a name="text"></a>Texte

Le champ `text` contient du contenu textuel facultatif à communiquer à un utilisateur. La valeur du champ `text` est de type chaîne, et son format est texte brut.

## <a name="event-activity"></a>Activité d’événement

Les activités d’événements communiquent des informations de programmation d’un client ou un canal à un bot. La signification d’une activité d’événement est définie par le champ `name`, qui est significatif dans l’étendue d’un canal. Les activités d’événements sont conçues pour transporter à la fois des informations interactives (telles que des clics de bouton) et des informations non interactives (telles qu’une notification signalant qu’un client a effectué une mise à jour automatique d’un modèle de reconnaissance vocale incorporé).

Les activités d’événements sont l’équivalent asynchrone des [activités d’appel](#invoke-activity). Contrairement à invoke, event est conçu pour être étendu par les extensions d’applications clientes.

Les activités d’événements sont identifiées par une valeur `type` `event` et des valeurs spécifiques du champ `name`.

`R5000` : les canaux PEUVENT autoriser les messages d’événement définis par l’application entre les clients et les bots, si les clients autorisent la personnalisation de l’application.

### <a name="name"></a>NOM

Le champ `name` contrôle la signification de l’événement et le schéma du champ `value`. La valeur du champ `name` est de type chaîne.

`R5001` : les activités d’événements DOIVENT contenir un champ `name`.

`R5002` : les récepteurs DOIVENT ignorer les activités d’événements avec des champs `name` qu’ils ne comprennent pas.

### <a name="value"></a>Valeur

Le champ `value` contient des paramètres propres à cet événement, tels que définis par le nom de l’événement. La valeur du champ `value` est un type complexe.

`R5100` : le champ `value` PEUT être manquant ou vide, s’il est défini par le nom d’événement.

`R5101` : les extensions de l’activité d’événement NE DEVRAIENT PAS exiger des récepteurs qu’ils utilisent d’autres informations que le `type` d’activité et les champs `name` pour comprendre le schéma du champ `value`.

### <a name="relates-to"></a>Associé à

Le champ `relatesTo` référence une autre conversation et éventuellement une activité spécifique au sein de cette activité. La valeur du champ `relatesTo` est un objet complexe du type de référence Conversation.

`R5200` : `relatesTo` NE DEVRAIT PAS référencer une activité au sein de la conversation identifiée par le champ `conversation`.

## <a name="invoke-activity"></a>Activité d’appel

Les activités d’appel communiquent des informations de programmation d’un client ou un canal à un bot, et elles ont une charge utile de retour correspondante pour une utilisation dans le canal. La signification d’une activité d’appel est définie par le champ `name`, qui est significatif dans l’étendue d’un canal. 

Les activités d’appel sont l’équivalent synchrone des [activités d’événements](#event-activity). Les activités d’événements sont conçues pour être extensibles. Les activités d’appel diffèrent uniquement par leur capacité à retourner des charges utiles de réponse au canal. Étant donné que le canal doit décider où et comment traiter ces charges utiles de réponse, Invoke est utile uniquement dans les cas où la prise en charge explicite de chaque nom d’appel a été ajoutée au canal. Par conséquent, invoke n’est pas conçue pour être un mécanisme d’extensibilité d’application générique.

Les activités d’appel sont identifiées par une valeur `type` `invoke` et des valeurs spécifiques du champ `name`.

La liste des activités d’appel définies est incluse dans l’[Annexe III](#appendix-iii---protocols-using-the-invoke-activity).

`R5301` : les canaux NE DEVRAIENT PAS autoriser les messages d’appel définis par l’application entre les clients et les bots.

### <a name="name"></a>NOM

Le champ `name` contrôle la signification de l’appel et le schéma du champ `value`. La valeur du champ `name` est de type chaîne.

`R5401` : les activités d’appel DOIVENT contenir un champ `name`.

`R5402` : les récepteurs DOIVENT ignorer les activités d’événements avec des champs `name` qu’ils ne comprennent pas.

### <a name="value"></a>Valeur

Le champ `value` contient des paramètres propres à cet événement, tels que définis par le nom de l’événement. La valeur du champ `value` est un type complexe.

`R5500` : le champ `value` PEUT être manquant ou vide, s’il est défini par le nom d’événement.

`R5501` : les extensions de l’activité d’événement NE DEVRAIENT PAS exiger des récepteurs qu’ils utilisent d’autres informations que le `type` d’activité et les champs `name` pour comprendre le schéma du champ `value`.

### <a name="relates-to"></a>Associé à

Le champ `relatesTo` référence une autre conversation et éventuellement une activité spécifique au sein de cette activité. La valeur du champ `relatesTo` est un objet complexe du type de [référence Conversation](#conversation-reference).

`R5600` : `relatesTo` NE DEVRAIT PAS référencer une activité au sein de la conversation identifiée par le champ `conversation`.

## <a name="installation-update-activity"></a>Activité de mise à jour d’installation

Les activités de mise à jour d’installation représentent une installation ou désinstallation d’un bot au sein d’une unité d’organisation (par exemple un locataire client ou une « équipe ») d’un canal. En général, elles ne représentent pas l’ajout ou la suppression d’un canal.

Les activités de mise à jour d’installation sont identifiées par une valeur `type` `installationUpdate`.

`R5700` : les canaux PEUVENT envoyer des activités d’installation quand un bot est ajouté ou supprimé d’un locataire, d’une équipe ou autre unité d’organisation dans le canal.

`R5701` : les canaux NE DEVRAIENT PAS envoyer d’activités d’installation quand le bot est installé ou supprimé d’un canal.

### <a name="action"></a>Action

Le champ `action` décrit la signification de l’activité de mise à jour d’installation. La valeur du champ `action` est une chaîne. Seules les valeurs de `add` et `remove` sont définies.

## <a name="message-delete-activity"></a>Activité de suppression de message

Les activités de suppression de message représentent la suppression d’une activité de message existante dans une conversation. L’activité supprimée est référencée par les champs `id` et `conversation` au sein de l’activité.

Les activités de suppression de message sont identifiées par une valeur `type` `messageDelete`.

`R5800` : les canaux PEUVENT choisir d’envoyer des activités de suppression de message pour toutes les suppressions dans une conversation, pour un sous-ensemble de suppressions dans une conversation (par exemple, seules les suppressions par certains utilisateurs), ou pour aucune activité dans la conversation.

`R5801` : les canaux NE DEVRAIENT PAS envoyer d’activités de suppression de message pour les conversations ou les activités que le bot n’a pas observées.

`R5802` : si un bot déclenche une suppression, le canal NE DEVRAIT PAS envoyer d’activité de suppression de message à ce bot.

`R5803` : les canaux NE DEVRAIENT PAS envoyer d’activités de suppression de message correspondant à des activités dont le type n’est pas `message`.

## <a name="message-update-activity"></a>Activité de mise à jour de message

Les activités de mise à jour de message représentent la mise à jour d’une activité de message existante dans une conversation. L’activité de mise à jour est identifiée par les champs `id` et `conversation` au sein de l’activité, et l’activité de mise à jour de message contient tous les champs de l’activité de message révisée.

Les activités de mise à jour de message sont identifiées par une valeur `type` `messageUpdate`.

`R5900` : les canaux PEUVENT choisir d’envoyer des activités de mise à jour de message pour toutes les mises à jour dans une conversation, pour un sous-ensemble de mises à jour dans une conversation (par exemple, seules les mises à jour par certains utilisateurs), ou pour aucune activité dans la conversation.

`R5901` : si un bot déclenche une mise à jour, le canal NE DEVRAIT PAS envoyer d’activité de mise à jour de message à ce bot.

`R5902` : les canaux NE DEVRAIENT PAS envoyer d’activités de mise à jour de message correspondant à des activités dont le type n’est pas `message`.

## <a name="message-reaction-activity"></a>Activité de réaction à un message

Les activités de réaction à un message représentent une interaction sociale sur une activité de message existante dans une conversation. L’activité d’origine est référencée par les champs `id` et `conversation` au sein de l’activité. Le champ `from` représente la source de la réaction (par exemple l’utilisateur qui a réagi au message).

Les activités de réaction à un message sont identifiées par une valeur `type` `messageReaction`.

### <a name="reactions-added"></a>Réactions ajoutées

Le champ `reactionsAdded` contient une liste des réactions ajoutées à cette activité. La valeur du `reactionsAdded` champ est un tableau de type [`messageReaction`](#message-reaction).

### <a name="reactions-removed"></a>Réactions supprimées

Le champ `reactionsRemoved` contient une liste des réactions supprimées de cette activité. La valeur du `reactionsRemoved` champ est un tableau de type [`messageReaction`](#message-reaction).

## <a name="typing-activity"></a>Activité de saisie

Les activités de saisie représentent l’entrée en cours provenant d’un utilisateur ou d’un bot. Cette activité est souvent envoyée quand des séquences de touches sont entrées par un utilisateur, bien qu’elle soit également utilisée par les bots pour indiquer qu’ils « réfléchissent », et qu’elle puisse également être utilisée pour indiquer par exemple la collecte audio à partir d’utilisateurs.

Les activités de saisie sont conçues pour être persistantes dans les interfaces utilisateur pendant trois secondes.

Les activités de saisie sont identifiées par une valeur `type` `typing`.

`R6000` : s’ils le peuvent, les clients DEVRAIENT afficher les indicateurs de saisie pendant trois secondes à compter de la réception d’une activité de saisie.

`R6001` : à moins que cela soit connu pour le canal, les expéditeurs Ne DEVRAIENT PAS envoyer d’activités de saisie plus fréquemment qu’une toutes les trois secondes. (Les expéditeurs PEUVENT envoyer des activités de saisie toutes les deux secondes afin d’empêcher l’apparition d’écarts.)

`R6002` : si un canal attribue un [`id`](#id) à une activité de saisie, il PEUT autoriser les bots et les clients à supprimer l’activité de saisie avant son expiration.

`R6003` : s’ils le peuvent, les canaux DEVRAIENT envoyer des activités de saisie aux bots.

## <a name="complex-types"></a>Types complexes

Cette section définit les types complexes utilisés dans le schéma d’activité décrit ci-dessus.

### <a name="attachment"></a>Pièce jointe

Les pièces jointes sont du contenu inclus dans une [activité de message](#message-activity) : cartes, documents binaires et autre contenu interactif. Elles sont destinées à être affichées en conjonction avec du contenu textuel. Le contenu peut être envoyé par le biais d’un URI de données dans le champ `contentUrl` ou inline dans le champ `content`.

`R7100` : les expéditeurs NE DEVRAIENT PAS inclure à la fois les champs `content` et `contentUrl` dans une même pièce jointe.

#### <a name="content-type"></a>Type de contenu

Le champ `contentType` décrit le [type de média MIME](https://www.iana.org/assignments/media-types/media-types.xhtml) [[8](#references)] du contenu de la pièce jointe. La valeur du champ `contentType` est de type chaîne.

#### <a name="content"></a>Contenu

Quand il est présent, le champ `content` contient un objet JSON structuré en guise de pièce jointe. La valeur du champ `content` est un type complexe défini par le champ `contentType`.

`R7110` : les expéditeurs NE DEVRAIENT PAS inclure de primitives JSON dans le champ `content` d’une pièce jointe.

#### <a name="content-url"></a>URL de contenu

Quand il est présent, le champ `contentUrl` contient une URL vers le contenu de la pièce jointe. Les URI de données, tels que définis dans [RFC 2397](https://tools.ietf.org/html/rfc2397) [[9](#references)] sont généralement prises en charge par les canaux. La valeur du champ `contentUrl` est de type chaîne.

`R7120` : les récepteurs DEVRAIENT accepter les URL HTTPS.

`R7121` : les récepteurs PEUVENT accepter les URL HTTP.

`R7122` : les canaux DEVRAIENT accepter les URI de données.

`R7123` : les canaux NE DEVRAIENT PAS envoyer d’URI de données aux clients ou aux bots.

#### <a name="name"></a>NOM

Le champ `name` contient un nom facultatif ou un nom de fichier pour la pièce jointe. La valeur du champ `name` est de type chaîne.

#### <a name="thumbnail-url"></a>URL de miniature

Certains clients ont la possibilité d’afficher des miniatures personnalisées pour les pièces jointes non interactives ou en tant qu’espaces réservés pour les pièces jointes interactives. Le champ `thumbnailUrl` identifie la source pour cette miniature. Les URI de données, tels que définis dans [RFC 2397](https://tools.ietf.org/html/rfc2397) [[9](#references)] sont généralement aussi autorisées.

`R7140` : les récepteurs DEVRAIENT accepter les URL HTTPS.

`R7141` : les récepteurs PEUVENT accepter les URL HTTP.

`R7142` : les canaux DEVRAIENT accepter les URI de données.

`R7143` : les canaux NE DEVRAIENT PAS envoyer de champs `thumbnailUrl` aux bots.

### <a name="card-action"></a>Action de carte

Une action de carte représente un bouton interactif utilisable dans des cartes ou en tant qu’[actions suggérées](#suggested-actions). Elle sert à demander une entrée utilisateur. En dépit de leur nom, les actions de carte ne se limitent pas à une utilisation sur les cartes.

Elles sont significatives uniquement quand elles sont envoyées à des canaux.

Les canaux décident comment chaque action se manifeste dans l’expérience utilisateur. Dans la plupart des cas, les cartes sont interactives, c’est-à-dire que l’on peut cliquer dessus. Dans d’autres cas, elles peuvent être sélectionnées par la saisie vocale. Dans les cas où le canal n’offre pas d’expérience d’activation interactive (par exemple lors de l’interaction par SMS), il est possible que le canal ne prenne en charge aucune activation. La décision concernant le rendu des actions est contrôlée par des exigences normatives mentionnées ailleurs dans ce document (par exemple dans le format de la carte ou dans la définition d’[actions suggérées](#suggested-actions)).

#### <a name="type"></a>type

Le champ `type` décrit la signification du bouton et le comportement quand le bouton est activé. Les valeurs du champ `type` sont par convention des chaînes courtes (par exemple, « `openUrl` »). Pour plus d’informations sur les exigences spécifiques de chaque type d’action, consultez les sections suivantes.

* Une action de type `messageBack` représente une réponse textuelle à envoyer par le biais du système de conversation.
* Une action de type `imBack` représente une réponse textuelle qui est ajoutée au flux de conversation.
* Une action de type `postBack` représente une réponse textuelle qui n’est pas ajoutée au flux de conversation.
* Une action de type `openUrl` représente un lien hypertexte devant être géré par le client.
* Une action de type `downloadFile` représente un lien hypertexte à télécharger.
* Une action de type `showImage` représente une image qui peut être affichée.
* Une action de type `signin` représente un lien hypertexte devant être géré par le système de connexion du client.
* Une action de type `playAudio` représente du contenu audio pouvant être lu.
* Une action de type `playVideo` représente du contenu vidéo pouvant être lu.
* Une action de type `call` représente un numéro de téléphone qui peut être appelé.
* Une action de type `payment` représente une demande de paiement.

#### <a name="title"></a>Intitulé

Le champ `title` contient le texte à afficher sur la face du bouton. La valeur du champ `title` est de type chaîne et ne contient pas de balisage.

Ce champ s’applique aux actions de tous les types.

`R7210` : les canaux NE DEVRAIENT PAS traiter de balisage à l’intérieur du champ `title` (par exemple, Markdown).

#### <a name="image"></a>Image

Le champ `image` contient une URL référençant une image à afficher sur la face du bouton. Les URI de données, tels que définis dans [RFC 2397](https://tools.ietf.org/html/rfc2397) [[9](#references)] sont généralement prises en charge par les canaux. La valeur du champ `image` est de type chaîne.

Ce champ s’applique aux actions de tous les types.

`R7220` : les canaux DEVRAIENT accepter les URL HTTPS.

`R7221` : les canaux PEUVENT accepter les URL HTTPS.

`R7222` : les canaux DEVRAIENT accepter les URI de données.

#### <a name="text"></a>Texte

Le champ `text` contient du texte à envoyer à un bot et inclus dans le flux de conversation quand l’utilisateur clique sur le bouton. Le contenu du champ `text` peut être visible ou non, en fonction du type de bouton. Le champ `text` peut contenir du balisage, contrôlé par le champ [`textFormat`](#text-format) dans la racine de l’activité. La valeur du champ `text` est de type chaîne.

Ce champ est utilisé uniquement sur les actions des types sélectionnés. Vous trouverez des détails sur chaque type d’action plus loin dans ce document.

`R7230` : le champ `text` PEUT contenir une chaîne vide pour indiquer l’envoi de texte sans contenu.

`R7231` : les canaux DEVRAIENT traiter le contenu du champ `text` conformément au champ [`textFormat`](#text-format) dans la racine de l’activité.

#### <a name="display-text"></a>Texte affiché

Le champ `displayText` contient du texte à inclure dans le flux de conversation quand l’utilisateur clique sur le bouton. Le contenu du champ `displayText` DEVRAIT toujours être affiché, si cela est techniquement possible dans le canal. Le champ `displayText` peut contenir du balisage, contrôlé par le champ [`textFormat`](#text-format) dans la racine de l’activité. La valeur du champ `displayText` est de type chaîne.

Ce champ est utilisé uniquement sur les actions des types sélectionnés. Vous trouverez des détails sur chaque type d’action plus loin dans ce document.

`R7240` : le champ `displayText` PEUT contenir une chaîne vide pour indiquer l’envoi de texte sans contenu.

`R7241` : les canaux DEVRAIENT traiter le contenu du champ `displayText` conformément au champ [`textFormat`](#text-format) dans la racine de l’activité.

#### <a name="value"></a>Valeur

Le champ `value` contient du contenu de programmation à envoyer à un bot quand le bouton est activé. Le contenu du champ `value` est de tout type primitif ou complexe, bien que certains types d’activités limitent ce champ.

Ce champ est utilisé uniquement sur les actions des types sélectionnés. Vous trouverez des détails sur chaque type d’action plus loin dans ce document.

#### <a name="message-back"></a>Réponse message

Une action `messageBack` représente une réponse textuelle à envoyer par le biais du système de conversation. Elle utilise les champs suivants :
* `type` ("`messageBack`")
* `title`
* `image`
* `text`
* `displayText`
* `value` (de n’importe quel type)

`R7350` : les expéditeurs NE DEVRAIENT PAS inclure de champs `value` de types primitifs (par exemple, string, int). Les champs `value` DEVRAIENT être des types complexes ou omis.

`R7351` : les canaux PEUVENT rejeter ou supprimer les champs `value` dont le type n’est pas complexe.

`R7352` : en cas d’activation, les canaux DOIVENT envoyer une activité de type `message` à tous les récepteurs concernés.

`R7353` : si le canal prend en charge le stockage et la transmission de texte, le contenu du champ `text` de l’action doit être conservé et transmis dans le champ `text` de l’activité de message générée.

`R7352` : si le canal prend en charge le stockage et la transmission de valeurs de programmation supplémentaires, le contenu du champ `value` doit être conservé et transmis dans le champ `value` de l’activité de message générée.

`R7353` : si le canal prend en charge la conservation dans le flux de conversation d’une valeur différente de celle envoyée aux bots, il doit inclure le champ `displayText` dans l’historique de conversation.

`R7354` : si le canal ne prend pas en charge `R7353` mais prend en charge l’enregistrement de texte dans le flux de conversation, il DOIT inclure le champ `text` dans l’historique de conversation.

#### <a name="im-back"></a>Retour messagerie instantanée

Une action `imBack` représente une réponse textuelle qui est ajoutée au flux de conversation. Elle utilise les champs suivants :
* `type` ("`imBack`")
* `title`
* `image`
* `value` (de type chaîne)

`R7360` : en cas d’activation, les canaux DOIVENT envoyer une activité de type `message` à tous les récepteurs concernés.

`R7361` : si le canal prend en charge le stockage et la transmission de texte, le contenu du champ `title` doit être conservé et transmis dans le champ `text` de l’activité de message générée.

`R7362` : si le champ `title` sur une action est manquant et que le champ `value` est de type chaîne, le canal PEUT transmettre le contenu du champ `value` dans le champ `text` de l’activité de message générée.

`R7363` : si le canal prend en charge l’enregistrement de texte dans le flux de conversation, il DOIT inclure le contenu du champ `title` dans l’historique de conversation.

#### <a name="post-back"></a>Publication

Une action `postBack` représente une réponse textuelle qui n’est pas ajoutée au flux de conversation. Elle utilise les champs suivants :
* `type` ("`postBack`")
* `title`
* `image`
* `value` (de type chaîne)

`R7370` : en cas d’activation, les canaux DOIVENT envoyer une activité de type `message` à tous les récepteurs concernés.

`R7371` : les canaux NE DEVRAIENT PAS inclure de texte dans l’historique de conversation quand une action postBack est activée.

`R7372` : les canaux DOIVENT rejeter ou supprimer les champs `value` qui ne sont pas de type chaîne.

`R7373` : si le canal prend en charge le stockage et la transmission de texte, le contenu du champ `value` doit être conservé et transmis dans le champ `text` de l’activité de message générée.

`R7374` : si le canal ne peut pas prendre en charge la transmission au bot sans inclure l’historique dans le flux de conversation, il doit utiliser le champ `title` en tant que texte d’affichage.

#### <a name="open-url-actions"></a>Actions d’ouverture d’URL

Une action `openUrl` représente un lien hypertexte devant être géré par le client. Elle utilise les champs suivants :
* `type` ("`openUrl`")
* `title`
* `image`
* `value` (de type chaîne)

`R7380` : les expéditeurs DOIVENT inclure une URL dans le champ `value` d’une action `openUrl`.

`R7381` : les récepteurs PEUVENT rejeter une action `openUrl` dont le champ `value` est manquant ou n’est pas une chaîne.

`R7382` : les récepteurs DEVRAIENT rejeter ou supprimer les actions `openUrl` dont le champ `value` contient un URI de données, tel que défini dans [RFC 2397](https://tools.ietf.org/html/rfc2397) [[9](#references)].

`R7383` : les destinataires NE DEVRAIENT PAS rejeter les actions `openUrl` dont l’URI `value` a une valeur ou un schéma d’URI inattendu.

`R7384` : les clients ayant connaissance de schémas d’URI particuliers (par exemple, HTTP) PEUVENT gérer les actions `openUrl` au sein d’un convertisseur incorporé (par exemple un contrôle de navigateur).

`R7385` : si possible, les clients DEVRAIENT déléguer la gestion des actions `openUrl` non gérées par `R7354` au gestionnaire d’URI au niveau du système d’exploitation ou du shell.

#### <a name="download-file-actions"></a>Actions de téléchargement de fichier

Une action `downloadFile` représente un lien hypertexte à télécharger. Elle utilise les champs suivants :
* `type` ("`downloadFile`")
* `title`
* `image`
* `value` (de type chaîne)

`R7390` : les expéditeurs DOIVENT inclure une URL dans le champ `value` d’une action `downloadFile`.

`R7391` : les récepteurs PEUVENT rejeter une action `downloadFile` dont le champ `value` est manquant ou n’est pas une chaîne.

`R7392` : les récepteurs DEVRAIENT rejeter ou supprimer les actions `downloadFile` dont le champ `value` contient un URI de données, tel que défini dans [RFC 2397](https://tools.ietf.org/html/rfc2397) [[9](#references)].

#### <a name="show-image-file-actions"></a>Actions d’affichage de fichier image

Une action `showImage` représente une image qui peut être affichée. Elle utilise les champs suivants :
* `type` ("`showImage`")
* `title`
* `image`
* `value` (de type chaîne)

`R7400` : les expéditeurs DOIVENT inclure une URL dans le champ `value` d’une action `showImage`.

`R7401` : les récepteurs PEUVENT rejeter une action `showImage` dont le champ `value` est manquant ou n’est pas une chaîne.

`R7402` : les récepteurs PEUVENT rejeter une action `showImage` dont le champ `value` est un URI de données, tel que défini dans [RFC 2397](https://tools.ietf.org/html/rfc2397) [[9](#references)].

#### <a name="signin"></a>Connexion

Une action `signin` représente un lien hypertexte devant être géré par le système de connexion du client. Il utilise les champs suivants :
* `type` ("`signin`")
* `title`
* `image`
* `value` (de type chaîne)

`R7410` : les expéditeurs DOIVENT inclure une URL dans le champ `value` d’une action `signin`.

`R7411` : les récepteurs PEUVENT rejeter une action `signin` dont le champ `value` est manquant ou n’est pas une chaîne.

`R7412` : les récepteurs DOIVENT rejeter ou supprimer les actions `signin` dont le champ `value` contient un URI de données, tel que défini dans [RFC 2397](https://tools.ietf.org/html/rfc2397) [[9](#references)].

#### <a name="play-audio"></a>Lecture audio

Une action `playAudio` représente du contenu audio pouvant être lu. Elle utilise les champs suivants :
* `type` ("`playAudio`")
* `title`
* `image`
* `value` (de type chaîne)

`R7420` : en cas d’activation, les canaux PEUVENT envoyer des événements multimédias.

`R7421` : les canaux DOIVENT rejeter ou supprimer les champs `value` qui ne sont pas de type chaîne.

`R7422` : les expéditeurs NE DEVRAIENT PAS envoyer d’URI de données, tel que défini dans [RFC 2397](https://tools.ietf.org/html/rfc2397) [[9](#references)], sans connaissance préalable de leur prise en charge par le canal.

#### <a name="play-video"></a>Lecture vidéo

Une action `playAudio` représente du contenu vidéo pouvant être lu. Elle utilise les champs suivants :
* `type` ("`playVideo`")
* `title`
* `image`
* `value` (de type chaîne)

`R7430` : en cas d’activation, les canaux PEUVENT envoyer des événements multimédias.

`R7431` : les canaux DOIVENT rejeter ou supprimer les champs `value` qui ne sont pas de type chaîne.

`R7432` : les expéditeurs NE DEVRAIENT PAS envoyer d’URI de données, tel que défini dans [RFC 2397](https://tools.ietf.org/html/rfc2397) [[9](#references)], sans connaissance préalable de leur prise en charge par le canal.

#### <a name="call"></a>Appel

Une action `call` représente un numéro de téléphone qui peut être appelé. Il utilise les champs suivants :
* `type` ("`call`")
* `title`
* `image`
* `value` (de type chaîne)

`R7440` : les expéditeurs DOIVENT inclure une URL de schéma `tel` dans le champ `value` d’une action `signin`.

`R7441` : les récepteurs DOIVENT rejeter une action `signin` dont le champ `value` est manquant ou est un URI d’un schéma autre que `tel`.

#### <a name="payment"></a>Paiement

Une action `payment` représente une demande de paiement. Elle utilise les champs suivants :
* `type` ("`payment`")
* `title`
* `image`
* `value` (de type complexe [Demande de paiement](#payment-request))

`R7450` : les expéditeurs DOIVENT inclure un objet complexe de type [Demande de paiement](#payment-request) dans le champ `value` d’une action `payment`.

`R7451`: Les canaux DOIVENT rejeter une action `payment` dont le champ `value` est manquant ou n’est pas un objet complexe de type [Demande de paiement](#payment-request).

### <a name="channel-account"></a>Compte de canal

Comptes de canal représentent des identités au sein d’un canal. Le compte de canal comprend un ID qui peut être utilisé pour identifier et contacter le compte au sein de ce canal. Parfois, ces ID existent dans un seul espace de noms (par exemple les ID Skype). Parfois, ils sont fédérés entre de nombreux serveurs (par exemple les adresses e-mail). Outre l’ID, les comptes de canal incluent des noms d’affichage et des ID d’objets AAD (Azure Active Directory).

#### <a name="channel-account-id"></a>ID de compte de canal

Le champ `id` est l’identificateur et l’adresse dans le canal. La valeur du champ `id` est une chaîne. « name@example.com » est un exemple de `id` au sein d’un canal qui utilise des adresses e-mail.

`R7510` : les canaux DEVRAIENT utiliser les mêmes valeurs et conventions pour les ID de comptes, quelle que soit leur position dans le schéma (`from.id`, `recipient.id`, `membersAdded`, et ainsi de suite). Ainsi, les bots et les clients peuvent utiliser des comparaisons de chaînes ordinales pour savoir par exemple quand ils sont décrits dans le champ `membersAdded` d’une activité `conversationUpdate`.

#### <a name="channel-account-name"></a>Nom du compte de canal

Le champ `name` est un nom convivial facultatif dans le canal. La valeur du champ `name` est une chaîne. « John Doe » est un exemple de `name` au sein d’un canal.

#### <a name="channel-account-aad-object-id"></a>ID d’objet AAD de compte de canal

Le champ `aadObjectId` est un ID facultatif correspondant à l’ID d’objet du compte dans AAD (Azure Active Directory). La valeur du champ `aadObjectId` est une chaîne.

### <a name="conversation-account"></a>Compte de conversation

Les comptes de conversation représentent l’identité de conversations au sein d’un canal. Dans les canaux qui ne prennent en charge qu’une seule conversation entre deux comptes (par exemple, SMS), le compte de conversation est persistant et n’a pas de début ou de fin prédéterminé(e). Dans les canaux qui prennent en charge plusieurs conversations parallèles (par exemple, e-mail), chaque conversation aura probablement un ID unique.

#### <a name="conversation-account-id"></a>ID de compte de conversation

Le champ `id` est l’identificateur au sein du canal. Le format de cet ID est défini par le canal et est utilisé comme chaîne opaque dans l’ensemble du protocole.

Les canaux DEVRAIENT choisir des valeurs `id` qui sont stables pour tous les participants à une conversation. (Par exemple, un exemple médiocre de champ `id` pour une conversation 1:1 consiste à utiliser l’ID de l’autre participant comme valeur `id`. Cela génère un `id` différent du point de vue de chaque participant. Un meilleur choix consiste à trier les ID des deux participants et à les concaténer. On obtient alors la même valeur pour les deux parties.)

#### <a name="conversation-account-name"></a>Nom du compte de conversation

Le champ `name` est un nom convivial facultatif pour la conversation dans le canal. La valeur du champ `name` est une chaîne.

#### <a name="conversation-account-aad-object-id"></a>ID d’objet AAD de compte de conversation

Le champ `aadObjectId` est un ID facultatif correspondant à l’ID d’objet de la conversation dans AAD (Azure Active Directory). La valeur du champ `aadObjectId` est une chaîne.

#### <a name="conversation-account-is-group"></a>isGroup

Le champ `isGroup` indique si la conversation contient plus de deux participants au moment où l’activité est générée. La valeur du champ `isGroup` est une valeur booléenne. S’il est omis, la valeur par défaut est `false`. Ce champ contrôle généralement le comportement @mention pour les participants dans le canal. Il DEVRAIT être défini sur `true` si et seulement si plus de deux participants ont la capacité à envoyer et à recevoir des activités au sein de la conversation.

### <a name="entity"></a>Entité

Les entités transmettent des métadonnées relatives à une activité ou une conversation. La signification et la forme de chaque entité sont définies par le champ `type`, et des champs supplémentaires propres au type existent en tant qu’homologues du champ `type`.

`R7600` : les destinataires DOIVENT ignorer les entités dont ils ne comprennent pas les types.

`R7601` : les récepteurs DEVRAIENT ignorer les entités dont ils comprennent le type, mais qu’ils sont incapables de traiter à cause par exemple d’erreurs syntaxiques.

Nous conseillons aux parties projetant un type d’entité existant dans le format de l’entité d’activité de résoudre les conflits avec le nom de champ `type` et les incompatibilités avec l’exigence de sérialisation `R2001` dans le cadre de la liaison entre l’IRI et le schéma d’entité.

#### <a name="type"></a>Type

Le champ `type` est obligatoire. Il définit la signification et la forme de l’entité. `type` est destiné à contenir des [IRI](https://tools.ietf.org/html/rfc3987) [[3](#references)], bien qu’un petit nombre de types d’entités non-IRI soient définis dans l’[Annexe I](#appendix-ii---non-iri-entity-types).

`R7610` : les expéditeurs DEVRAIENT utiliser des noms de types non-IRI uniquement pour les types décrits dans l’[Annexe II](#appendix-ii---non-iri-entity-types).

`R7611` : les expéditeurs PEUVENT envoyer des types IRI pour les types décrits dans l’[Annexe II](#appendix-ii---non-iri-entity-types) s’ils savent que le récepteur les comprend.

`R7612` : les expéditeurs DEVRAIENT utiliser ou établir des IRI pour les types d’entités non définis dans l’[Annexe II](#appendix-ii---non-iri-entity-types).

### <a name="suggested-actions"></a>Actions suggérées

Les actions suggérées peuvent être envoyées dans le contenu du message afin de créer des éléments d’action interactive au sein d’une interface utilisateur cliente.

`R7700` : les clients qui ne prennent pas en charge l’interface utilisateur capable d’afficher les actions suggérées DEVRAIENT ignorer le champ `suggestedActions`.

`R7701` : les expéditeurs DEVRAIENT omettre le champ `suggestedActions` si le champ `actions` est vide.

#### <a name="to"></a>Destination

Le champ `to` contient les ID de compte de canal auxquels les actions suggérées doivent être présentées. Vous pouvez l’utiliser pour filtrer les actions à un sous-ensemble de participants au sein de la conversation.

`R7710` : si le champ `to` est manquant ou vide, le client DEVRAIT présenter les actions suggérées à tous les participants à la conversation.

`R7711` : si le champ `to` contient des ID non valides, ces valeurs DEVRAIENT être ignorées.

#### <a name="actions"></a>Actions

Le champ `actions` contient une liste plate d’actions à afficher. La valeur de chaque élément de liste `actions` est un objet complexe de type `cardAction`.

### <a name="message-reaction"></a>Réaction à un message

Les réactions aux messages représentent une interaction sociale (« j’aime », « + 1 », et ainsi de suite). Actuellement, elles ne comportent qu’un seul champ : `type`.

#### <a name="type"></a>Type

Le champ `type` décrit le type d’interaction sociale. La valeur du champ `type` est une chaîne, et sa signification est définie par le canal dans lequel l’interaction se produit. `like` et `+1` sont des valeurs courantes, bien qu’elles soient uniformes par convention et non par règle.

## <a name="references"></a>Références

1. [RFC 2119](https://tools.ietf.org/html/rfc2119)
2. [ISO 8601](https://www.iso.org/iso-8601-date-and-time-format.html)
3. [RFC 3987](https://tools.ietf.org/html/rfc3987)
4. [Markdown](https://daringfireball.net/projects/markdown/)
5. [ISO 639](https://www.iso.org/iso-639-language-codes.html)
6. [SSML](https://www.w3.org/TR/speech-synthesis/)
7. [XML](https://www.w3.org/TR/xml/)
8. [Types de média MIME](https://www.iana.org/assignments/media-types/media-types.xhtml)
9. [RFC 2397](https://tools.ietf.org/html/rfc2397)
10. [ISO 3166-1](https://www.iso.org/iso-3166-country-codes.html)
11. [Cartes adaptatives](https://adaptivecards.io)

# <a name="appendix-i---changes"></a>Annexe I - Changements

## <a name="2018-02-07"></a>07-02-2018

* Première ébauche

# <a name="appendix-ii---non-iri-entity-types"></a>Annexe II - Types d’entités non-IRI

Les [entités](#entity) d’activités communiquent des métadonnées supplémentaires concernant l’activité, telles que l’emplacement d’un utilisateur ou la version de l’application de messagerie qu’il utilise. Les types d’activités sont censés être des IRI, mais quelques noms non-IRI sont couramment utilisés. Cette annexe contient une liste exhaustive des types d’entités non-IRI pris en charge.

| Ttype           | URI équivalent                          | Description               |
| -------------- | --------------------------------------- | ------------------------- |
| GeoCoordinates | https://schema.org/GeoCoordinates       | Géo-coordonnées Schema.org |
| Mention        | https://botframework.com/schema/mention | @-mention                 |
| Place          | https://schema.org/Place                | Emplacement Schema.org          |
| Thing          | https://schema.org/Thing                | Chose Schema.org          |
| clientInfo     | N/A                                     | Informations sur le client Skype         |

### <a name="clientinfo"></a>clientInfo

L’entité clientInfo contient des informations étendues sur le logiciel client utilisé pour envoyer le message d’un utilisateur. Il contient trois propriétés, toutes facultatives.

`R9201` : les bots NE DEVRAIENT PAS envoyer l’entité `clientInfo`.

`R9202` : les expéditeurs DEVRAIENT inclure l’entité `clientInfo` uniquement quand un ou plusieurs champs sont renseignés.

#### <a name="locale-deprecated"></a>Paramètres régionaux (déprécié)

Le champ `locale` contient les paramètres régionaux de l’utilisateur. Ce champ duplique le champ [`locale`](#locale) dans la racine de l’activité. La valeur du champ `locale` est un code au format [ISO 639](https://www.iso.org/iso-639-language-codes.html) [[5](#references)] dans une chaîne.

Le champ `locale` dans `clientInfo` est déprécié.

`R9211` : les récepteurs NE DEVRAIENT PAS utiliser le champ `locale` dans l’objet `clientInfo`.

`R9212` : les expéditeurs PEUVENT renseigner le champ `locale` dans `clientInfo` pour des raisons de compatibilité. Si la compatibilité avec les récepteurs antérieurs n’est pas obligatoire, les expéditeurs NE DEVRAIENT PAS envoyer la propriété `locale`.

#### <a name="country"></a>Pays

Le champ `country` contient l’emplacement détecté de l’utilisateur. Cette valeur peut différer des données [`locale`](#locale), car `country` est détecté tandis que `locale` est généralement un paramètre utilisateur ou un paramètre d’application. La valeur du champ `country` est un code de pays à deux ou trois lettres [ISO 3166-1](https://www.iso.org/iso-3166-country-codes.html) [[10](#references)].

`R9220` : les canaux NE DEVRAIENT PAS autoriser les clients à spécifier des valeurs arbitraires pour le champ `country`. Les canaux DEVRAIENT utiliser un mécanisme tel que GPS, API d’emplacement ou détection d’adresse IP pour déterminer le pays générant une demande.

#### <a name="platform"></a>Plateforme

Le champ `platform` décrit la plateforme cliente de messagerie utilisée pour générer l’activité. La valeur du champ `platform` est une chaîne, et la liste des valeurs possibles et leur signification sont définies par le canal qui les envoie.

Notez que sur les canaux avec un flux de conversation permanent, `platform` est généralement utile uniquement pour décider du contenu à inclure, et non du format de ce contenu. Par exemple, si un utilisateur sur un appareil mobile demande l’aide du support produit, un bot peut générer une aide propre à son appareil mobile. Toutefois, l’utilisateur pourrait alors rouvrir le flux de conversation sur son PC afin de pouvoir le lire sur cet écran tout en apportant des modifications à son appareil mobile. Dans cette situation, le champ `platform` est destiné à établir le contenu, mais celui-ci doit être visible sur d’autres appareils.

`R9230` : les bots NE DEVRAIENT PAS utiliser le champ `platform` pour contrôler la mise en forme des données de réponse, sauf s’ils savent expressément que le contenu qu’ils envoient ne peut être vu que sur l’appareil en question.

# <a name="appendix-iii---protocols-using-the-invoke-activity"></a>Annexe III - Protocoles utilisant l’activité invoke

L’[activité invoke](#invoke-activity) est conçue pour être utilisée uniquement dans les protocoles pris en charge par les canaux Bot Framework (autrement dit, il ne s’agit pas d’un mécanisme d’extensibilité générique). Cette annexe contient une liste de tous les protocoles Bot Framework utilisant cette activité.

## <a name="payments"></a>Paiements

Le protocole de paiements Bot Framework utilise invoke pour calculer les frais d’expédition et le taux d’imposition, et pour communiquer une forte confirmation des paiements effectués.

Le protocole de paiements définit trois opérations (définies dans le champ `name` d’une activité invoke) :
* `payment/shippingAddressChange`
* `payment/shppingOptionsChange`
* `payment/paymentResponse`

Vous trouverez plus de détails dans la page [Demande de paiement](https://docs.microsoft.com/en-us/bot-framework/dotnet/bot-builder-dotnet-request-payment).

## <a name="teams-compose-extension"></a>Extension de composition Teams

Le canal Microsoft Teams utilise invoke pour les [extensions de composition](https://docs.microsoft.com/en-us/microsoftteams/platform/concepts/messaging-extensions). Cette utilisation d’invoke est propre à Microsoft Teams.



