---
ms.openlocfilehash: eac6abae509d92ea4714bc01221f180ea575950f
ms.sourcegitcommit: bdb981c0b11ee99d128e30ae0462705b2dae8572
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/17/2019
ms.locfileid: "54360863"
---
Obtenez la clé de chiffrement.

1. Connectez-vous au [portail Azure](http://portal.azure.com/).
1. Ouvrez la ressource Web App Bot de votre bot.
1. Ouvrez les **Paramètres d’application** du bot.
1. Dans la fenêtre **Paramètres d’application**, faites défiler jusqu’à **Paramètres d’application**.
1. Recherchez le **botFileSecret** et copiez sa valeur.

Déchiffrez le fichier .bot.

```cmd
msbot secret --bot <name-of-bot-file> --secret "<bot-file-secret>" --clear
```

| Option | Description |
|:---|:---|
| --bot | Chemin relatif du fichier .bot téléchargé. |
| --secret | Clé de chiffrement. |

Copiez le fichier `.bot` déchiffré dans le répertoire qui contient votre projet de bot local, mettez à jour votre bot pour qu’il utilise ce nouveau fichier `.bot`, puis supprimez votre ancien fichier `.bot`.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Dans **appsettings.json**, mettez à jour la propriété **botFilePath** pour pointer vers le nouveau fichier `.bot` que vous avez ajouté à votre répertoire local.

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Dans **appsettings.json**, mettez à jour la propriété **botFilePath** pour pointer vers le nouveau fichier `.bot` que vous avez ajouté à votre répertoire local.

---

Une fois que votre bot a été mis à jour, supprimez le répertoire temporaire du bot téléchargé.