---
title: Connecter un robot à GroupMe | Microsoft Docs
description: Découvrez comment configurer une connexion de robot à GroupMe.
keywords: canal de robot, GroupMe, créer GroupMe, informations d’identification
author: RobStand
ms.author: RobStand
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
ms.openlocfilehash: a2004293ff10cfbc7132f58b7c0c834a2012cfd1
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2018
ms.locfileid: "49998566"
---
# <a name="connect-a-bot-to-groupme"></a>Connecter un robot à GroupMe

Vous pouvez configurer un robot pour communiquer avec des personnes à l’aide de l’application de messagerie de groupe GroupMe.

[!INCLUDE [Channel Inspector intro](~/includes/snippet-channel-inspector.md)]

## <a name="sign-up-for-a-groupme-account"></a>Ouvrir compte GroupMe

Si vous n’avez pas de compte GroupMe, [créez un compte](https://web.groupme.com/signup).

## <a name="create-a-groupme-application"></a>Créer une application GroupMe

[Créez une application GroupMe](https://dev.groupme.com/applications/new) pour votre robot.

Utilisez cette URL de rappel : `https://groupme.botframework.com/Home/Login`

![Créer une application](~/media/channels/GM-StepApp.png)

## <a name="gather-credentials"></a>Collecter les informations d’identification

1. Dans le champ **URL de redirection**, copiez la valeur après **client_id =**.
2. Copiez la valeur **Jeton d’accès**.

![Copier l’ID client et le jeton d’accès](~/media/channels/GM-StepClientId.png)


## <a name="submit-credentials"></a>Envoyer les informations d’identification

1. Sur dev.botframework.com, collez la valeur **client_id** que vous venez de copier dans le champ **ID Client**.
2. Collez la valeur **Jeton d’accès** valeur dans le champ **Jeton d’accès**.
2. Cliquez sur **Enregistrer**.

![Entrer les informations d’identification](~/media/channels/GM-StepClientIDToken.png)
