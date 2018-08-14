## <a name="next-steps"></a>Étapes suivantes
Une fois que vous avez déployé votre bot dans le cloud et vérifié la réussite de ce déploiement en testant le bot à l’aide de l’application Bot Framework Emulator, l’étape suivante du processus de publication du bot dépend de si vous avez déjà inscrit ou non votre bot auprès de Bot Framework.

### <a name="if-you-have-already-registered-your-bot-with-the-bot-framework"></a>Si vous avez déjà inscrit votre bot auprès de Bot Framework :

1. Revenez au <a href="https://dev.botframework.com" target="_blank">portail Bot Framework</a> et [mettez à jour les données de paramètres de votre bot](~/bot-service-manage-settings.md) pour spécifier le **point de terminaison de messagerie** du bot.

2. [Configurez le bot pour qu’il s’exécute sur un ou plusieurs canaux](~/bot-service-manage-channels.md).

### <a name="if-you-have-not-yet-registered-your-bot-with-the-bot-framework"></a>Si vous n’avez pas encore inscrit votre bot auprès de Bot Framework :

1. [Inscrivez votre bot auprès de Bot Framework](~/bot-service-quickstart-registration.md).

2. Mettez à jour les valeurs d’ID et de mot de passe d’application Microsoft dans les paramètres de configuration de votre application déployée afin de spécifier les valeurs **appID** et **password** qui ont été générées pour votre bot lors du processus d’inscription. Pour trouver les paramètres **AppID** et **AppPassword** de votre bot, consultez la section [MicrosoftAppID and MicrosoftAppPassword](~/bot-service-manage-overview.md#microsoftappid-and-microsoftapppassword) (MicrosoftAppID et MicrosoftAppPassword).

3. [Configurez le bot pour qu’il s’exécute sur un ou plusieurs canaux](~/bot-service-manage-channels.md).