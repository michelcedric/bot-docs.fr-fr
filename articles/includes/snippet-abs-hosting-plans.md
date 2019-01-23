---
ms.openlocfilehash: 8eb125b2d0f0c9981cafb763ba809c7d042d8f77
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/11/2019
ms.locfileid: "54225984"
---
Le service de robot offre deux plans d’hébergement pour les bots. La conversion du code source des bots d’un plan vers l’autre est un processus manuel.   

## <a name="app-service-plan"></a>Plan App Service

Un robot qui utilise un plan App Service est une application web Azure standard que vous pouvez configurer pour allouer une capacité prédéfinie avec des coûts et une mise à l’échelle prévisibles. Dans le cas d’un robot utilisant ce plan d’hébergement, vous pouvez :

* modifier le code source du robot en ligne à l’aide d’un éditeur de code avancé dans le navigateur ;
* télécharger, déboguer et republier votre robot C# au moyen de Visual Studio ;
* configurer un déploiement continu en toute facilité pour Visual Studio Online et Github ;
* utiliser un exemple de code préparé pour le kit SDK Bot Framework.

## <a name="consumption-plan"></a>Plan de consommation

Un robot qui utilise un plan de consommation est un robot sans serveur qui s’exécute sur <a href="http://go.microsoft.com/fwlink/?linkID=747839" target="_blank">Azure Functions</a> et applique la tarification d’Azure Functions avec paiement à l’exécution. Un robot qui utilise ce plan d’hébergement peut faire l’objet d’une mise à l’échelle de manière à gérer les pics de trafic exceptionnels. Vous pouvez modifier le code source du robot en ligne à l’aide d’un éditeur de code de base dans le navigateur. Pour plus d’informations sur l’environnement d’exécution d’un bot utilisant un plan de consommation, consultez l’article <a target='_blank' href='/azure/azure-functions/functions-scale'>Plans Consommation et App Service d’Azure Functions</a>.
