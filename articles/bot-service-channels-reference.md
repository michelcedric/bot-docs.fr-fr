---
title: Informations de référence sur les canaux
description: Informations de référence sur les canaux Bot Framework
keywords: informations de référence sur les canaux, canaux bot builder, canaux bot framework
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: tools
ms.date: 03/01/2019
ms.openlocfilehash: 0fb0f650b44d320d78a0ada5d46105048019964c
ms.sourcegitcommit: cf3786c6e092adec5409d852849927dc1428e8a2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 03/01/2019
ms.locfileid: "57224957"
---
# <a name="categorized-activities-by-channel"></a>Activités classées par canal

Les tableaux suivants indiquent quels événements (ou activités) peuvent provenir des canaux.

Voici l’explication des symboles des tableaux :

Symbole              | Signification
:------------------:|:------------------------------------------------
:white_check_mark:  |Le bot doit s’attendre à recevoir cette activité
:x:                 |Le bot ne doit **jamais** s’attendre à recevoir cette activité
:white_large_square:|La possibilité pour ce bot de recevoir cette activité reste indéterminée

Les activités peuvent être réparties en diverses catégories. Pour chaque catégorie, nous avons un tableau présentant les activités possibles.

<a name="conversational"></a>Conversationnel
--------------

 \                      | Cortana            | Ligne directe        | Direct Line (discussion web) | Email              | Facebook           | GroupMe            | Kik                | Teams              | Slack              | Skype   | Skype Entreprise | Telegram | Twilio  
:---------------------- | :-----:            | :----------------: | :--------------------: |:----:              | :------:           | :-----:            | :-----:            | :---:              | :---:              | :---:   | :------------: | :------: | :----:  
Message                 | :white_check_mark: | :white_check_mark: | :white_check_mark:     | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark:        | :white_check_mark:  | :white_check_mark: 
MessageReaction         | :x:                | :x:                | :x:                    | :x:                | :x:                | :x:                | :x:                | :white_check_mark: | :x:                | :x:      | :x:             | :x:       | :x:      

- Tous les canaux envoient des activités de message.
- Lorsque vous utilisez un dialogue, vous devez toujours passer les activités de message au dialogue.
- Ce n’est probablement pas vrai pour les réactions aux messages (MessageReaction), même si elles font partie de la conversation.
- Il existe logiquement deux types de MessageReaction : Added et Removed (Ajout et Suppression)


> [!TIP]
> Les réactions aux messages sont, par exemple, des approbations du type « pouce vers le haut » pour un commentaire. Leur affichage pouvant être aléatoire, il arrive que l’utilisateur les confonde avec un bouton. Cette activité est actuellement envoyée par le canal Teams.  


<a name="welcome"></a>Bienvenue
-------

 \                         | Cortana            | Ligne directe        | Direct Line (Web Chat) | Email   | Facebook             | GroupMe | Kik     | Teams   | Slack   | Skype   | Skype Entreprise | Telegram | Twilio  
:----------------------    | :-----:            | :---------:        | :--------------------: |:----:   | :------:             | :-----: | :-----: | :---:   | :---:   | :---:   | :------------: | :------: | :----:  
ConversationUpdate         | :white_check_mark: | :white_check_mark: | :white_check_mark:     | :x:     | :white_large_square: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :x:      | :x:             | :white_check_mark:  | :x:     
ContactRelationUpdate      | :x:                | :x:                | :x:                    | :x:     | :x:                  | :x:      | :x:      | :x:      | :x:      | :white_check_mark: | :white_check_mark:        | :x:       | :x:     

- Il est courant que les canaux envoient des activités ConversationUpdate.
- Il existe logiquement deux types de MessageReaction : Added et Removed (Ajout et Suppression)
- Il est très tentant de penser que le comportement d’accueil du bot peut être implémenté simplement à l’aide de ConversationUpdate.Added, et il arrive que cela fonctionne.
- Cependant, cette méthode est trop simple. Pour obtenir un comportement d’accueil fiable, l’implémentation du bot doit également utiliser l’état.


<a name="application-extensibility"></a>Extensibilité des applications
-------------------------

 \                      | Cortana            | Ligne directe        | Direct Line (Web Chat) | Email   | Facebook | GroupMe | Kik     | Teams   | Slack   | Skype   | Skype Entreprise | Telegram | Twilio  
:---------------------- | :-----:            | :---------:        | :--------------------: |:----:   | :------: | :-----: | :-----: | :---:   | :---:   | :---:   | :------------: | :------: | :----:  
Event.*                    | :white_large_square: | :white_check_mark: | :white_check_mark:    | :white_large_square: | :white_large_square:  | :white_large_square: | :white_large_square: | :white_large_square:  | :white_large_square:  | :white_large_square:  | :white_large_square:           | :white_large_square:     | :white_large_square:  
Event.CreateConversation   | :white_large_square: | :white_large_square: | :white_large_square:   | :white_large_square: | :white_large_square:  | :white_large_square: | :white_large_square: | :white_large_square:  | :white_large_square:  | :white_large_square:  | :white_large_square:           | :white_large_square:     | :white_large_square:  
Event.ContinueConversation | :white_large_square: | :white_large_square: | :white_large_square:   | :white_large_square: | :white_large_square:  | :white_large_square: | :white_large_square: | :white_large_square:  | :white_large_square:  | :white_large_square:  | :white_large_square:           | :white_large_square:     | :white_large_square:  

- Dans _Direct Line_, les activités d’événement constituent un mécanisme d’extensibilité.
- Une application qui est propriétaire à la fois du client et du serveur peut choisir de « tunneler » ses propres événements via le service à l’aide de cette activité d’événement.


<a name="microsoft-teams"></a>Microsoft Teams
------------------

 \                      | Cortana            | Ligne directe        | Direct Line (Web Chat) | Email   | Facebook | GroupMe | Kik     | Teams   | Slack   | Skype   | Skype Entreprise | Telegram | Twilio  
:---------------------- | :-----:            | :---------:        | :--------------------: |:----:   | :------: | :-----: | :-----: | :---:   | :---:   | :---:   | :------------: | :------: | :----:  
Invoke.TeamsVerification   | :x:      | :x:          | :x: | :x:   | :x:       | :x:      | :x:      | :white_check_mark: | :x:    | :x:    | :x:             | :x:       | :x:     
Invoke.ComposeResponse     | :x:      | :x:          | :x: | :x:   | :x:       | :x:      | :x:      | :white_check_mark: | :x:    | :x:    | :x:             | :x:       | :x:     

- En plus d’un certain nombre d’autres activités typées, Microsoft Teams définit certaines activités d’appel propres à Teams.
- Les activités d’appel sont propres à une application et ne sont pas définies par un client.
- Il n’existe pas de notion générale d’appel, seulement des sous-types d’activité.
- Les appels sont les seules activités qui déclenchent un comportement de type « demande-réponse » chez le bot.

Important : Si vous utilisez des dialogues pour l’invite OAuth, vous devez transférer l’activité Invoke.TeamsVerification au dialogue.


<a name="message-update"></a>Mise à jour des messages
--------------

 \                      | Cortana            | Ligne directe        | Direct Line (Web Chat) | Email   | Facebook | GroupMe | Kik     | Teams   | Slack   | Skype   | Skype Entreprise | Telegram | Twilio  
:---------------------- | :-----:            | :---------:        | :--------------------: |:----:   | :------: | :-----: | :-----: | :---:   | :---:   | :---:   | :------------: | :------: | :----:  
MessageUpdate | :x:      | :x:          | :x:    | :x: | :x:      | :x:      | :x:  | :white_check_mark: | :white_large_square:  | :x:    | :x:             | :x:       | :x:     
MessageDelete | :x:      | :x:          | :x:    | :x: | :x:      | :x:      | :x:  | :white_check_mark: | :white_large_square:  | :x:    | :x:             | :x:       | :x:     

- La mise à jour des messages est prise en charge par Teams.


<a name="oauth"></a>OAuth
-------

 \                      | Cortana            | Ligne directe        | Direct Line (Web Chat) | Email   | Facebook | GroupMe | Kik     | Teams   | Slack   | Skype   | Skype Entreprise | Telegram | Twilio  
:---------------------- | :-----:            | :---------:        | :--------------------: |:----:   | :------: | :-----: | :-----: | :---:   | :---:   | :---:   | :------------: | :------: | :----:  
Event.TokenResponse| :white_large_square:  | :white_check_mark:   | :white_check_mark:    | :x:    | :white_large_square: | :white_large_square: | :white_large_square: | :x:    | :white_large_square: | :white_large_square: | :white_large_square:       | :white_large_square: | :white_large_square: 

Important : Si vous utilisez des dialogues pour l’invite OAuth, vous devez transférer l’activité Event.TokenResponse au dialogue.


<a name="uncategorized"></a>Sans catégorie 
-------------

 \                      | Cortana  | Ligne directe        | Direct Line (Web Chat) | Email | Facebook | GroupMe | Kik     | Teams | Slack | Skype | Skype Entreprise | Telegram | Twilio  
:---------------------- | :-----:  | :---------:        | :--------------------: |:----: | :------: | :-----: | :-----: | :---: | :---: | :---: | :------------: | :------: | :----:  
EndOfConversation       | :x:      | :white_check_mark: | :white_check_mark:     | :x:   | :x:      | :x:     | :x:     | :x:   | :x:   | :x:   | :x:            | :x:      | :x:     
InstallationUpdate      | :x:      | :white_check_mark: | :white_check_mark:     | :x:   | :x:      | :x:     | :x:     | :x:   | :x:   | :x:   | :x:            | :x:      | :x:     
Saisie                  | :x:      | :white_check_mark: | :white_check_mark:     | :x:   | :x:      | :x:     | :x:     | :x:   | :x:   | :x:   | :x:            | :x:      | :x:     
handoffHandoff                 | :x:      | :x:                | :x:                    | :x:   | :x:      | :x:     | :x:     | :x:   | :x:   | :x:   | :x:            | :x:      | :x:     


<a name="out-of-use-includes-payment-specific-invoke"></a>Absence d’utilisation (inclut les appels liés au paiement)
---------------------------------------------
- DeleteUserData 
- Invoke.PaymentRequest  
- Invoke.Address
- Ping

---

## <a name="summary-of-activities-supported-per-channel"></a>Récapitulatif des activités prises en charge par canal

<a name="cortana"></a>Cortana
-------
- Message
- ConversationUpdate
- _Event.TokenResponse_
- _EndOfConversation (lorsque la fenêtre se ferme ?)_

<a name="direct-line"></a>Ligne directe
--------
- Message
- ConversationUpdate
- Event.TokenResponse
- Event.*
- _Event.CreateConversation_
- _Event.ContinueConversation_

<a name="email"></a>Email
-----
- Message

<a name="facebook"></a>Facebook
--------
- Message
- _Event.TokenResponse_

<a name="groupme"></a>GroupMe
-------
- Message
- ConversationUpdate
- _Event.TokenResponse_

<a name="kik"></a>Kik
---
- Message
- ConversationUpdate
- _Event.TokenResponse_

<a name="teams"></a>Teams
-----
- Message
- ConversationUpdate
- MessageReaction
- MessageUpdate
- MessageDelete
- Invoke.TeamsVerification
- Invoke.ComposeResponse


<a name="slack"></a>Slack
-----
- Message
- ConversationUpdate
- _Event.TokenResponse_


<a name="skype"></a>Skype
-----
- Message
- ContactRelationUpdate
- _Event.TokenResponse_


<a name="skype-business"></a>Skype Entreprise
--------------
- Message
- ContactRelationUpdate 
- _Event.TokenResponse_


<a name="telegram"></a>Telegram
--------
- Message
- ConversationUpdate
- _Event.TokenResponse_


<a name="twilio"></a>Twilio
------
- Message

## <a name="summary-table-all-activities-to-all-channels"></a>Tableau récapitulatif de toutes les activités de tous les canaux

 \                         | Cortana              | Ligne directe          | Direct Line (Web Chat) | Email                | Facebook             | GroupMe | Kik     | Teams   | Slack   | Skype   | Skype Entreprise | Telegram | Twilio  
:----------------------    | :-----:              | :---------:          | :--------------------: |:----:                | :------:             | :-----: | :-----: | :---:   | :---:   | :---:   | :------------: | :------: | :----:  
Message                    | :white_check_mark:   | :white_check_mark:   | :white_check_mark:     | :white_check_mark:   | :white_check_mark:   | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark:        | :white_check_mark:  | :white_check_mark: 
MessageReaction            | :x:                  | :x:                  | :x:                    | :x:                  | :x:                  | :x:      | :x:      | :white_check_mark: | :x:      | :x:      | :x:             | :x:       | :x:      
ConversationUpdate         | :white_check_mark:   | :white_check_mark:   | :white_check_mark:     | :x:                  | :white_large_square: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :x:      | :x:             | :white_check_mark:  | :x:     
ContactRelationUpdate      | :x:                  | :x:                  | :x:                    | :x:                  | :x:                  | :x:      | :x:      | :x:      | :x:      | :white_check_mark: | :white_check_mark:        | :x:       | :x:     
Event.*                    | :white_large_square: | :white_check_mark:   | :white_check_mark:     | :white_large_square: | :white_large_square: | :white_large_square: | :white_large_square: | :white_large_square:  | :white_large_square:  | :white_large_square:  | :white_large_square:           | :white_large_square:     | :white_large_square:  
Event.CreateConversation   | :white_large_square: | :white_large_square: | :white_large_square:   | :white_large_square: | :white_large_square: | :white_large_square: | :white_large_square: | :white_large_square:  | :white_large_square:  | :white_large_square:  | :white_large_square:           | :white_large_square:     | :white_large_square:  
Event.ContinueConversation | :white_large_square: | :white_large_square: | :white_large_square:   | :white_large_square: | :white_large_square: | :white_large_square: | :white_large_square: | :white_large_square:  | :white_large_square:  | :white_large_square:  | :white_large_square:           | :white_large_square:     | :white_large_square:  
Invoke.TeamsVerification   | :x:                  | :x:                  | :x:                    | :x:                  | :x:                  | :x:      | :x:      | :white_check_mark: | :x:    | :x:    | :x:             | :x:       | :x:     
Invoke.ComposeResponse     | :x:                  | :x:                  | :x:                    | :x:                  | :x:                  | :x:      | :x:      | :white_check_mark: | :x:    | :x:    | :x:             | :x:       | :x:     
MessageUpdate              | :x:                  | :x:                  | :x:                    | :x:                  | :x:                  | :x:      | :x:  | :white_check_mark: | :white_large_square:  | :x:    | :x:             | :x:       | :x:     
MessageDelete              | :x:                  | :x:                  | :x:                    | :x:                  | :x:                  | :x:      | :x:  | :white_check_mark: | :white_large_square:  | :x:    | :x:             | :x:       | :x:     
Event.TokenResponse        | :white_large_square: | :white_check_mark:   | :white_check_mark:     | :x:                  | :white_large_square: | :white_large_square: | :white_large_square: | :x:    | :white_large_square: | :white_large_square: | :white_large_square:       | :white_large_square: | :white_large_square: 
EndOfConversation          | :x:                  | :white_check_mark:   | :white_check_mark:     | :x:                  | :x:      | :x:     | :x:     | :x:   | :x:   | :x:   | :x:            | :x:      | :x:     
InstallationUpdate         | :x:                  | :white_check_mark:   | :white_check_mark:     | :x:                  | :x:      | :x:     | :x:     | :x:   | :x:   | :x:   | :x:            | :x:      | :x:     
Saisie                     | :x:                  | :white_check_mark:   | :white_check_mark:     | :x:                  | :x:      | :x:     | :x:     | :x:   | :x:   | :x:   | :x:            | :x:      | :x:     
Handoff                    | :x:                  | :x:                  | :x:                    | :x:                  | :x:      | :x:     | :x:     | :x:   | :x:   | :x:   | :x:            | :x:      | :x:     

## <a name="web-chat"></a>Web Chat 
Web Chat envoie :
- « message » : avec « text » et/ou « attachments »
- « event » : avec « name » et « value » (au format JSON/chaîne)
- « typing » : si l’utilisateur a défini l’option « sendTypingIndicator », Web Chat n’envoie pas « contactRelationUpdate ». Web Chat ne prend pas en charge les réactions aux messages (« messageReaction »), car il ne nous a jamais été demandé explicitement de prendre en charge cette fonctionnalité.

Par défaut, Web Chat affiche :
- « message » : s’affiche au format carrousel ou empilé, selon l’option de l’activité
- « typing » : s’affiche pendant 5 secondes, puis est masqué (ou jusqu’à l’activité suivante)
- « conversationUpdate » : est masqué
- « event» : est masqué
- « others» : affiche un avertissement (jamais visible dans la version de production). Vous pouvez modifier ce pipeline de rendu pour ajouter, supprimer ou remplacer un rendu personnalisé.

Vous pouvez utiliser Web Chat pour envoyer n’importe quel type d’activité ou de charge utile. Toutefois, cette fonctionnalité n’est ni documentée ni recommandée. Vous devez utiliser une activité d’événement à la place.
