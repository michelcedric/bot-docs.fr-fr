---
ms.openlocfilehash: 88732d2d5490962d7a899d936767e7dd148e94c5
ms.sourcegitcommit: bdb981c0b11ee99d128e30ae0462705b2dae8572
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/17/2019
ms.locfileid: "54360858"
---
Publiez votre bot local sur Azure. Cette étape peut prendre du temps.

```cmd
az bot publish --name <bot-resource-name> --proj-name "<project-file-name>" --resource-group <resource-group-name> --code-dir <directory-path> --verbose --version v4
```

| Option | Description |
|:---|:---|
| --name | Nom de ressource du bot dans Azure. |
| --proj-name | Pour C#, utilisez le nom du fichier projet de démarrage (sans .csproj) qui doit être publié. Par exemple : `EnterpriseBot`. Pour Node.js, utilisez le point d’entrée principal du bot. Par exemple : `index.js`. |
| --resource-group | Nom du groupe de ressources. |
| --code-dir | Répertoire depuis lequel charger le code du bot. |

Une fois cette opération terminée avec le message « Déploiement réussi ! », votre bot est déployé sur Azure.