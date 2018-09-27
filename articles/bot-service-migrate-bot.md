---
title: Migrer votre bot vers Azure | Microsoft Docs
description: Découvrez comment migrer votre robot du portail d’infrastructure Bot hérité vers un service bot dans le portail Azure.
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 6/22/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 41f3355102147e0d403629f23de79a90ac301209
ms.sourcegitcommit: 3bf3dbb1a440b3d83e58499c6a2ac116fe04b2f6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/23/2018
ms.locfileid: "46707895"
---
# <a name="migrate-your-bot-to-azure"></a>Migrer votre bot vers Azure



Tous les bots **Azure Bot Service (préversion)** créés dans le [portail d’infrastructure Bot](http://dev.botframework.com) doivent migrer vers le nouveau Service Bot dans Azure. Le service a été mis à la disposition générale (GA) en décembre 2017. 

Notez que les bots d’inscription connectés uniquement aux canaux suivants ne nécessitent *pas* de migration : **Équipes**, **Skype**, ou **Cortana**. Par exemple, un bot d’inscription connecté à **Facebook** et à **Skype** *doit* migrer, mais un bot d’inscription connecté à **Skype** et **Cortana** *n’a pas besoin* de migrer.

> [!IMPORTANT]
> Avant de migrer un bot Functions créé avec Node.js, il est nécessaire d’utiliser le **Pack Azure Functions** pour maintenir ensemble les modules **node_modules**. Cela améliore les performances pendant la migration et l’exécution du bot Functions après sa migration. Pour empaqueter vos modules, consultez [Empaqueter un bot Functions avec Funcpack](#package-a-functions-bot-with-funcpack).

Pour migrer votre bot, procédez comme suit :

1. Connectez-vous au [portail d’infrastructure Bot](http://dev.botframework.com) et cliquez sur **My bots** (Mes bots).
2. Cliquez sur le bouton **Migrate** (Migrer) du bot que vous souhaitez migrer.
3. Accepter les **Conditions** et cliquez sur **Migrate** (Migrer) pour démarrer le processus de migration ou sur **Cancel** (Annuler) pour annuler cette action.

> [!IMPORTANT]
> Tant que la tâche de migration est en cours d’exécution, ne quittez la page et n’actualisez pas la page. Cela provoquerait l’arrêt prématuré de la tâche de migration et vous devrez recommencer. Pour vous assurer que la migration se termine avec succès, attendez l’affichage du message de confirmation.

Une fois le processus de migration terminée avec succès, le **Migration status** (état de la migration) indique que le bot a migré. Un bouton **Roll back migration** (restaurer la migration) sera disponible pendant une semaine après la date de migration en cas de problème.

Cliquer sur le nom d’un bot migré ouvrira le bot dans le [portail Azure](http://portal.azure.com).

## <a name="package-a-functions-bot-with-funcpack"></a>Empaqueter un bot Functions avec Funcpack

Les bots Functions créés avec Node.js doivent être empaquetés à l’aide de [Funcpack](https://github.com/Azure/azure-functions-pack) avant la migration. Pour utiliser Funcpack avec votre projet, procédez comme suit :

1.  [Téléchargez](bot-service-build-download-source-code.md) votre code en local si vous ne l’avez pas déjà fait.
2.  Mettez à jour tous les packages npm dans **packages.json** pour les versions les plus récentes, puis exécutez `npm install`.
3.  Ouvrez **messages/index.js** et modifiez `module.exports = { default: connector.listen() }` par `module.exports = connector.listen();`
4.  Installez Funcpack via npm : `npm install -g azure-functions-pack`
5.  Pour empaqueter le répertoire **node_modules**, exécutez la commande suivante : `funcpack pack ./`
6.  Testez votre bot en local en exécutant le bot Functions à l’aide de Bot Framework Channel Emulator. Plus d’informations sur la façon d’exécuter le bot *funcpack* [ici](https://github.com/Azure/azure-functions-pack#how-to-run). 
7.  Chargez votre code vers Azure. Assurez-vous que le répertoire `.funcpack` est chargé. Vous n’avez pas besoin de charger le répertoire **node_modules**.
8. Testez votre bot distant pour vous assurer qu’il répond comme prévu.
9. [Migrez votre bot](#migrate-your-bot-to-azure) à l’aide de la procédure ci-dessus.

## <a name="migration-under-the-hood"></a>La migration vue de l’intérieur

Selon le type de bot pour lequel vous effectuez une migration, la liste ci-dessous peut vous aider à mieux comprendre ce qui se passe vu de l’intérieur.

* **Web App Bot** ou **Functions Bot** : pour ces types de bots, le projet de code source est copié de l’ancien bot vers le nouveau bot. D’autres ressources telles que le stockage de votre bot, Application Insights, LUIS, etc., restent telles quelles. Dans ce cas, le nouveau bot contient une copie des ID/clés/mots de passe correspondant à ces ressources existantes. 
* **Bot Channels Registration** : pour ce type de bot, le processus de migration crée simplement un nouveau **Bot Channels Registration** et copie remplace son point de terminaison par celui de l’ancien bot. 
* Quel que soit le types de bot que vous migrez, le processus de migration ne modifie pas l’état du bot existant. Cela vous permet de restaurer en toute sécurité si vous en avez le besoin.
* Si un [déploiement continu](bot-service-build-continuous-deployment.md) est configuré, vous devrez le configurer à nouveau afin que votre contrôle de code source se connecte au nouveau robot.

## <a name="understanding-azure-resources-after-migration"></a>Compréhension des ressources Azure après migration
Une fois la migration effectuée, votre **groupe de ressources** contiendra un certain nombre de ressources Azure nécessaires pour que votre bot fonctionne. Le type et le nombre de ressources dépendent du type de bot que vous avez migré. Consultez les sections suivantes pour en savoir plus.

### <a name="registration-bot"></a>Bot Registration

Il s’agit du type le plus simple. Le groupe de ressources dans Azure contient uniquement une ressource de type « Bot Channel Registration ». Il s’agit de l’équivalent de l’enregistrement de bot précédent dans le portail des développeurs d’infrastructure Bot.

![Listes des bots Bot Channel Registration dans Azure](~/media/bot-service-migrate-bot/channel-registration-bot.png)

### <a name="web-app-bot"></a>Bot Web App
La migration de bot d’application Web approvisionnera une ressource de Service bot de type « Web App Bot » et une nouvelle application web App Service (en vert dans la capture d’écran ci-dessous). Le bot Azure Bot Service (préversion) précédent est toujours disponible et peut être supprimé (en rouge dans la capture d’écran ci-dessous).

![Listes de bots Web App dans Azure](~/media/bot-service-migrate-bot/web-app-bot.png)

### <a name="functions-bot"></a>Bot Functions
La migration de bot Functions approvisionnera une ressource de Service bot de type « Functions Bot » et une nouvelle application de fonctions App Service (en vert dans la capture d’écran ci-dessous). Le bot Azure Bot Service (préversion) précédent est toujours disponible et peut être supprimé (en rouge dans la capture d’écran ci-dessous).

![Listes de bots Functions dans Azure](~/media/bot-service-migrate-bot/functions-bot.png)


## <a name="roll-back-migration"></a>Restaurer la migration

Dans l’éventualité d’un problème avec le bot pendant ou après la migration, vous pouvez **restaurer la migration**. Pour restaurer une migration, procédez comme suit :

1. Connectez-vous au [portail d’infrastructure Bot](http://dev.botframework.com) et cliquez sur **My bots** (Mes bots).
2. Cliquez sur le bouton **Restaurer la migration** pour le bot que vous souhaitez restaurer. Une invite s’affiche.
3. Cliquez sur **Oui, restaurer** pour continuer ou **Annuler** pour annuler l’action de restauration.

> [!NOTE]
> La fonctionnalité de restauration sera disponible pendant une semaine après la migration et doit être utilisée uniquement si vous rencontrez des problèmes dans le bot migré.

## <a name="migration-troubleshootingknown-issues"></a>Dépannage/problèmes connus avec la migration
Mon bot node.js/functions a migré avec succès, mais il ne répond pas :

* **Vérifiez le point de terminaison**
  * Accédez au panneau **Paramètres** dans les ressources de votre bot et vérifiez que le point de terminaison du robot présente un paramètre de chaîne de requête « code » contenant une valeur. Si ce n’est pas le cas, vous devez l’ajouter.
* **Consultez le dossier de secrets dans kudu pour les copies de sauvegarde**
  * Dans certains cas rares, certains fichiers de sauvegarde de secrets peuvent créer un conflit. Accédez au dossier **home\data\Functions\secrets** dans Kudu et supprimez tout fichier **host.snapshot** (ou **host.backup**) dans cet emplacement. Il ne devrait y avoir qu’un seul fichier **host.json** et un fichier **messages.json**. Enfin, redémarrez App Service et discutez à nouveau avec votre bot.

Pour tout autre problème, merci de soumettre un CRI au support Azure ou signalez un problème dans [GitHub](https://github.com/MicrosoftDocs/bot-framework-docs/issues).


## <a name="next-steps"></a>Étapes suivantes

Maintenant que la migration de votre bot est effectuée, apprenez comment gérer votre bot à partir du portail Azure.

> [!div class="nextstepaction"]
> [Gérer un bot](bot-service-manage-overview.md)
