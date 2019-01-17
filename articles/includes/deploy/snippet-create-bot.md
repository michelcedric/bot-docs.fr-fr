---
ms.openlocfilehash: f95c8a37b1207a26dab0a714b86412a9dba2dcb4
ms.sourcegitcommit: bdb981c0b11ee99d128e30ae0462705b2dae8572
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/17/2019
ms.locfileid: "54360874"
---
```cmd
az bot create --kind webapp --name <bot-resource-name> --location <geographic-location> --version v4 --lang <language> --verbose --resource-group <resource-group-name>
```

| Option | Description |
|:---|:---|
| --name | Nom unique utilisé pour déployer le bot dans Azure. Il peut s’agir du même nom que celui de votre bot local. N’incluez PAS d’espaces ni de traits de soulignement dans le nom. |
| --location | Emplacement géographique utilisé pour créer les ressources de service du bot. Par exemple, `eastus`, `westus`, `westus2`, etc. |
| --lang | Langage à utiliser pour créer le bot : `Csharp` ou `Node` ; la valeur par défaut est `Csharp`. |
| --resource-group | Nom du groupe de ressources dans lequel créer le bot. Vous pouvez configurer le groupe par défaut en utilisant `az configure --defaults group=<name>`. |