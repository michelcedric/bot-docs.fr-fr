---
ms.openlocfilehash: fed6222fb9cf2d7793776c5575dfbcda49b54224
ms.sourcegitcommit: bdb981c0b11ee99d128e30ae0462705b2dae8572
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/17/2019
ms.locfileid: "54360886"
---
1. Connectez-vous au [**portail d’inscription des applications**](https://apps.dev.microsoft.com/).
1. Cliquez sur **Ajouter une application** pour inscrire votre application, créez un **ID d’application**, puis **générez un nouveau mot de passe**. Si vous avez déjà une application et un mot de passe mais que vous avez oublié le mot de passe, vous devrez en générer un nouveau dans la section des secrets d’application.
1. Enregistrez l’ID d’application et le nouveau mot de passe que vous venez de générer afin de pouvoir les utiliser avec la commande `az bot create`.  

```cmd
az bot create --kind webapp --name <bot-resource-name> --location <geographic-location> --version v4 --lang <language> --verbose --resource-group <resource-group-name> --appid "<application-id>" --password "<application-password>" --verbose
```

| Option | Description |
|:---|:---|
| --name | Nom unique utilisé pour déployer le bot dans Azure. Il peut s’agir du même nom que celui de votre bot local. N’incluez PAS d’espaces ni de traits de soulignement dans le nom. |
| --location | Emplacement géographique utilisé pour créer les ressources de service du bot. Par exemple, `eastus`, `westus`, `westus2`, etc. |
| --lang | Langage à utiliser pour créer le bot : `Csharp` ou `Node` ; la valeur par défaut est `Csharp`. |
| --resource-group | Nom du groupe de ressources dans lequel créer le bot. Vous pouvez configurer le groupe par défaut en utilisant `az configure --defaults group=<name>`. |
| --appid | ID du compte Microsoft (ID MSA) à utiliser avec le bot. |
| --password | Mot de passe du compte Microsoft (MSA) pour le bot. |
