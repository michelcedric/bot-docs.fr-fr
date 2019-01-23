---
ms.openlocfilehash: 266cdc2bbeeb140e4b601c1bf0fb3fa8eb085dda
ms.sourcegitcommit: bdb981c0b11ee99d128e30ae0462705b2dae8572
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/17/2019
ms.locfileid: "54360889"
---
- Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.
- Installez la dernière version de l’outil [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest).
- Installez la dernière version de l’outil [MSBot](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/MSBot).
- Installez la dernière version publiée de l’[émulateur Bot Framework](https://aka.ms/Emulator-wiki-getting-started).
- Installez et configurez [ngrok](https://github.com/Microsoft/BotFramework-Emulator/wiki/Tunneling-%28ngrok%29).
- Familiarisez-vous avec ce qu’est un fichier [.bot](~/v4sdk/bot-file-basics.md).

Avec msbot version 4.3.2 et ultérieure, vous avez besoin d’Azure CLI version 2.0.54 ou ultérieure. Si vous avez installé l’extension botservice, supprimez-la à l’aide de cette commande.

```cmd
az extension remove --name botservice
```