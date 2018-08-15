## <a name="application-settings-and-messaging-endpoint"></a>Paramètres d’application et point de terminaison de messagerie

### <a name="verify-application-settings"></a>Vérifier les paramètres d’application

Pour que votre bot fonctionne correctement dans le cloud, vous devez vous assurer que ses paramètres d’application sont corrects. Si vous avez un **ID d’application** et un **mot de passe**, mettez à jour les valeurs `Microsoft AppId` et `Microsoft App Password` dans les paramètres de configuration de votre application dans le cadre du processus de déploiement. Pour trouver les paramètres **AppID** et **AppPassword** de votre robot, voir [MicrosoftAppID et MicrosoftAppPassword](~/bot-service-manage-overview.md#microsoftappid-and-microsoftapppassword).

> [!TIP]
> [!INCLUDE [Application configuration settings](~/includes/snippet-tip-bot-config-settings.md)]

Si vous n’avez pas encore inscrit votre robot auprès de Bot Framework (et n’avez donc pas encore d’**ID d’application** et de **mot de passe**), vous pouvez déployer votre robot avec des valeurs d’espace réservé temporaires pour ces paramètres.
Ensuite, après avoir [inscrit votre robot](~/bot-service-quickstart-registration.md), mettez à jour les paramètres de votre application déployée avec les valeurs d’**ID d’application** et de **mot de passe** générées pour votre robot lors de l’inscription.

### <a id="messagingEndpoint"></a> Vérifier le point de terminaison de messagerie

Votre robot déployé doit avoir un **point de terminaison de messagerie** pouvant recevoir des messages du service du connecteur Bot Framework.

> [!NOTE]
> Quand vous déployez votre bot sur Azure, le protocole SSL est automatiquement configuré pour votre application. Cela a pour effet s’activer le **Point de terminaison de messagerie** requis par le Bot Framework.
> Si vous effectuez le déploiement sur un autre service cloud, veillez à vérifier que votre application est configurée pour le protocole SSL, de sorte que le robot dispose d’un point de **terminaison de messagerie**.
