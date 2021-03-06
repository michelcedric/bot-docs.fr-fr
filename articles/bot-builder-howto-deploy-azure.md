---
redirect_url: /bot-framework/bot-builder-deploy-az-cli
ms.openlocfilehash: a300d6602a59c5e7d7cebdf14bb4f720a30ecbf8
ms.sourcegitcommit: 8183bcb34cecbc17b356eadc425e9d3212547e27
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 02/09/2019
ms.locfileid: "55971469"
---
<a name="--"></a><!--
---
titre : Déployer votre bot C# avec Visual Studio | Microsoft Docs description: Déployez votre bot sur le cloud Azure.
keywords: déployer bot, déployer azure, publier bot, déployer bot az, déployer bot visual studio, publier msbot, cloner msbot author: ivorb ms.author: v-ivorb manager: kamrani ms.topic: get-started-article ms.service: bot-service ms.subservice: abs ms.date: 07/02/2019
---

# <a name="deploy-your-c-bot-using-visual-studio"></a>Déployer votre bot C# avec Visual Studio

[!INCLUDE [pre-release-label](./includes/pre-release-label.md)]

Après avoir créé votre bot et l’avoir testé localement, vous pouvez le déployer sur Azure pour le rendre accessible depuis n’importe où. Le déploiement de votre bot sur Azure implique de payer les services que vous utilisez. L’article sur la [gestion de la facturation et des coûts](https://docs.microsoft.com/en-us/azure/billing/) vous aide à comprendre la facturation Azure, à superviser votre utilisation et vos coûts, ainsi qu’à gérer votre compte et vos abonnements.

Dans cet article, nous allons vous montrer comment déployer un bot C# à l’aide de Visual Studio et du portail Azure. Il est plus judicieux de lire cet article avant de suivre les étapes, afin de bien comprendre ce qu’implique le déploiement d’un bot.

## <a name="prerequisites"></a>Prérequis
- Installez [Bot Framework Emulator](https://aka.ms/Emulator-wiki-getting-started).
- Familiarisez-vous avec ce qu’est un fichier [.bot](v4sdk/bot-file-basics.md).


## <a name="update-bot-file-properties"></a>Mettre à jour les propriétés du fichier .bot

Avant de commencer le processus de déploiement, mettez à jour les propriétés du fichier .bot suivantes dans Visual Studio :
- **Action de build : Contenu**
- **Copier dans le répertoire de sortie : Toujours copier**


## <a name="deploy-your-bot-in-app-service"></a>Déployer votre bot dans App Service

Vous allez tout d’abord déployer le bot sur Azure App Service à partir de Visual Studio. Ensuite, vous allez configurer votre bot avec Azure Bot Service à l’aide de Bot Channels Registration.

**Remarque : Si le nom de votre projet Visual Studio comporte des espaces, les étapes de déploiement décrites ci-dessous ne fonctionnent pas.**

Dans la fenêtre de l’Explorateur de solutions, cliquez avec le bouton droit sur le nœud de votre projet, puis sélectionnez Publier.

![paramètre de publication](media/azure-bot-quickstarts/getting-started-publish-setting.png)

2. Dans le dialogue Choisir une cible de publication, vérifiez que l’option **App Service** est sélectionnée sur la gauche et que l’option **Créer** est sélectionnée sur la droite.

3. Cliquez sur le bouton Publier.

4. Dans le coin supérieur droit du dialogue, vérifiez que le dialogue affiche l’ID utilisateur correct pour votre abonnement Azure.

![publier principal](media/azure-bot-quickstarts/getting-started-publish-main.png)

5. Spécifiez le nom de l’application, l’abonnement, le groupe de ressources et le plan d’hébergement.

6. Une fois que vous êtes prêt, cliquez sur Créer. L’exécution de ce processus peut prendre plusieurs minutes.

7. Une fois l’opération terminée, un navigateur web s’ouvre, affichant l’URL publique de votre bot.

8. Effectuez une copie de cette URL (semblable à https://<yourbotname>.azurewebsites.net/).

> [!NOTE] 
> Vous devez utiliser la version HTTPS de l’URL lors de l’inscription de votre bot. Azure prend en charge SSL avec Azure App Service.

## <a name="create-your-bot-channels-registration"></a>Créer l’inscription aux canaux pour votre bot
Une fois votre bot déployé dans Azure, vous devez l’inscrire auprès d’Azure Bot Service.

1. Accédez au portail Azure à l’adresse https://portal.azure.com.

2. Connectez-vous à l’aide de la même identité que vous avez utilisée précédemment dans Visual Studio pour publier votre bot.

3. Cliquez sur Créer une ressource.

4. Dans le champ Rechercher dans la Place de marché, tapez Bot Channels Registration et appuyez sur Entrée.

5. Dans la liste renvoyée, choisissez Bot Channels Registration :

![Publier](media/azure-bot-quickstarts/getting-started-bot-registration.png)

6. Cliquez sur Créer dans le panneau qui s’ouvre.

7. Entrez un nom pour votre bot.

8. Choisissez le même abonnement que celui où vous avez déployé le code de votre bot.

9. Choisissez votre groupe de ressources existant qui définira l’emplacement.

10. Vous pouvez choisir le niveau de tarification F0 pour le développement et le test.

11. Entrer l’URL de votre bot. Assurez-vous que vous démarrez avec HTTPS et que vous ajoutez /api/messages, par exemple https://yourbotname.azurewebsites.net/api/messages

12. Désactivez Application Insights pour l’instant.

13. Cliquez sur ID d'application et mot de passe Microsoft

14. Dans le nouveau panneau, cliquez sur Créer.

15. Dans le nouveau panneau s’ouvre à droite, cliquez sur « Create App ID in the App Registration Portal » (créer un ID d’application dans le portail d’inscription des applications) pour ouvrir un nouvel onglet de navigateur.

![bot msa](media/azure-bot-quickstarts/getting-started-msa.png)

16. Dans le nouvel onglet, effectuez une copie de l’ID d’application et enregistrez-le quelque part. 

17. Cliquez sur le bouton Générer un mot de passe d'application pour continuer.

18. Un dialogue de navigateur s’ouvre et vous fournit le mot de passe de votre application : ce sera l’unique fois où vous l’obtiendrez. Copiez et enregistrez ce mot de passe quelque part pour pouvoir le récupérer plus tard.

19. Cliquez sur OK une fois que vous avez enregistré le mot de passe.

20. Fermez l’onglet du navigateur et revenez dans l’onglet du portail Azure.

21. Collez votre ID d’application et votre mot de passe dans les champs appropriés, puis cliquez sur OK.

22. Cliquez à présent sur Créer pour configurer votre inscription de canal. Cette opération peut prendre quelques secondes ou minutes.

## <a name="update-your-bots-application-settings"></a>Mettre à jour les paramètres d’application de votre bot
Pour permettre l’authentification de votre bot auprès d’Azure Bot Service, vous devez ajouter deux paramètres à la section Paramètres d'application de votre bot dans Azure App Service. 

1. Cliquez sur App Services. Tapez le nom de votre bot dans la zone de texte Abonnements. Cliquez ensuite sur le nom de votre bot dans la liste.

![app service](media/azure-bot-quickstarts/getting-started-app-service.png)

2. Dans la liste des options sur la gauche, dans les options de votre bot, recherchez l’option Paramètres d’application dans la section Paramètres et cliquez dessus.

![id bot](media/azure-bot-quickstarts/getting-started-app-settings-1.png)

3. Faites défiler jusqu'à la section Paramètres d’application.

![bot msa](media/azure-bot-quickstarts/getting-started-app-settings-2.png)

4. Cliquez sur Ajouter un nouveau paramètre.

5. Tapez le nom **MicrosoftAppId** et votre ID d’application comme valeur.

6. Cliquez sur Ajouter un nouveau paramètre

7. Tapez le nom **MicrosoftAppPassword** et votre mot de passe comme valeur.

8. Cliquez sur le bouton Enregistrer en haut de la fenêtre.

## <a name="test-your-bot-in-production"></a>Tester votre bot en production
À ce stade, vous pouvez tester votre bot depuis Azure à l’aide du client de chat web intégré.

1. Retournez dans votre groupe de ressources sur le portail Azure

2. Ouvrez votre bot.

3. Sous **Gestion du bot**, sélectionnez **Tester dans le chat web**.

![tester dans le chat web](media/azure-bot-quickstarts/getting-started-test-webchat.png)

4. Tapez un message comme `Hi`, puis appuyez sur Entrée. Le bot renverra `Turn 1: You sent Hi`.

---

## <a name="additional-resources"></a>Ressources supplémentaires

Lorsque vous déployez un bot, ces ressources sont généralement créées dans le portail Azure :

| Ressources      | Description |
|----------------|-------------|
| Robot Web App | Bot Azure Bot Service déployé sur Azure App Service.|
| [App Service](https://docs.microsoft.com/en-us/azure/app-service/)| Permet de générer et héberger des applications web.|
| [Plan App Service](https://docs.microsoft.com/en-us/azure/app-service/azure-web-sites-web-hosting-plans-in-depth-overview)| Définit un ensemble de ressources de calcul nécessaires à l’exécution d’une application web.|
| [Application Insights](https://docs.microsoft.com/en-us/azure/application-insights/app-insights-overview)| Fournit des outils pour collecter et analyser des données de télémétrie.|
| [Compte de stockage](https://docs.microsoft.com/en-us/azure/storage/common/storage-introduction)| Fournit un stockage cloud hautement disponible, sécurisé, durable, scalable et redondant.|

Si vous ne connaissez pas le groupe de ressources Azure, consultez cette rubrique de [terminologie](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-overview#terminology).

## <a name="next-steps"></a>Étapes suivantes
> [!div class="nextstepaction"]
> [Configurer le déploiement continu](bot-service-build-continuous-deployment.md)
-->
