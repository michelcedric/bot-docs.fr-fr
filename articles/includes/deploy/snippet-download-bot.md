---
ms.openlocfilehash: 867e65b25878f810e3247eb3cace95f4d31e11db
ms.sourcegitcommit: bdb981c0b11ee99d128e30ae0462705b2dae8572
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/17/2019
ms.locfileid: "54360871"
---
Utilisez un répertoire temporaire en dehors de votre répertoire de projet actuel. 

Cette commande crée un sous-répertoire sous save-path ; toutefois, le chemin spécifié doit déjà exister.

```cmd
az bot download --name <bot-resource-name> --resource-group <resource-group-name> --save-path "<path>"
```

| Option | Description |
|:---|:---|
| --name | Nom du bot dans Azure. |
| --resource-group | Nom du groupe de ressources dans lequel se trouve le bot. |
| --save-path | Répertoire existant sur lequel télécharger le code du bot. |