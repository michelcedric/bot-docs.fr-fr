Si vous utilisez des services tels que LUIS, vous devez également passer `luisAuthoringKey`. Si vous souhaitez utiliser un groupe de ressources existant dans Azure, utilisez l’argument `groupName` avec la commande ci-dessus.

Il est vivement recommandé d’utiliser l’option `verbose` permettant de résoudre les problèmes qui peuvent survenir lors du déploiement du bot. Les autres options utilisées avec la commande `msbot clone services` sont décrites ci-dessous :

| Arguments    | Description |
|--------------|-------------|
| `folder`     | Emplacement du fichier `bot.receipe`. Par défaut, le fichier receipe est créé dans le `DeploymentsScript/MSBotClone`. NE MODIFIEZ PAS ce fichier.|
| `location`   | Emplacement géographique utilisé pour créer les ressources de service du bot. Par exemple, eastus, westus, westus2, etc.|
| `proj-file`  | Pour un bot C#, il s’agit du fichier .csproj. Pour un bot JS, il s’agit du nom de fichier du projet de démarrage (par exemple, index.js) de votre bot local.|
| `name`       | Nom unique utilisé pour déployer le bot dans Azure. Il peut s’agir du même nom que celui de votre bot local. N’incluez PAS d’espaces dans le nom.|

Pour pouvoir créer des ressources Azure, vous devrez d’abord vous authentifier. Suivez les instructions qui s’affichent à l’écran pour effectuer cette étape.

Notez que l’étape ci-dessus dure _quelques secondes ou quelques minutes_ et que les ressources créées dans Azure ont leurs noms tronqués. Pour en savoir plus sur les noms tronqués, consultez le [problème n° 796](https://github.com/Microsoft/botbuilder-tools/issues/796) dans le dépôt GitHub.
