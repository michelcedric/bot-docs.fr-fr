## <a name="prerequisites"></a>Prérequis
- Visual Studio [2017](https://www.visualstudio.com/downloads)
- Modèle du kit SDK Bot Framework v4 pour [C#](https://aka.ms/bot-vsix)
- Bot Framework [Emulator](https://aka.ms/Emulator-wiki-getting-started)
- Connaissances d’[ASP.Net Core](https://docs.microsoft.com/aspnet/core/) et de la programmation asynchrone en [C#](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/async/index)

## <a name="create-a-bot"></a>Créer un bot
Installez le modèle BotBuilderVSIX.vsix que vous avez téléchargé à la section des prérequis.

Dans Visual Studio, créez un projet de bot en utilisant le modèle **Bot Framework Echo Bot** V4.

![Projet Visual Studio](~/media/azure-bot-quickstarts/bot-builder-dotnet-project.png)

> [!TIP] 
> Si nécessaire, modifiez le type de build de projet en ``.Net Core 2.1``. De même, mettez éventuellement à jour les [packages NuGet](https://docs.microsoft.com/en-us/nuget/quickstart/install-and-use-a-package-in-visual-studio) `Microsoft.Bot.Builder`.

Grâce au modèle, votre projet contient tout le code nécessaire pour créer le bot dans ce guide de démarrage rapide. En fait, vous n’avez pas besoin d’écrire du code supplémentaire.

## <a name="start-your-bot-in-visual-studio"></a>Démarrer votre bot dans Visual Studio

Lorsque vous cliquez sur le bouton d’exécution, Visual Studio génère l’application, la déploie sur localhost et lance le navigateur web pour afficher la page `default.htm` de l’application. À ce stade, votre bot s’exécute localement.

## <a name="start-the-emulator-and-connect-your-bot"></a>Démarrer l’émulateur et connecter votre robot

Démarrez à présent l’émulateur, puis connectez-vous à votre bot dans l’émulateur :

1. Cliquez sur le lien **Ouvrir le bot** de l’onglet de bienvenue de l’émulateur. 
2. Sélectionnez le fichier .bot situé dans le répertoire où vous avez créé le projet Visual Studio.

## <a name="interact-with-your-bot"></a>Interagir avec votre bot

Envoyez un message à votre bot, et le bot vous enverra un message à son tour.

![Émulateur en cours d’exécution](~/media/emulator-v4/emulator-running.png)

> [!NOTE]
> Si vous voyez que le message ne peut pas être envoyé, vous devrez peut-être redémarrer votre ordinateur, car ngrok n’a pas encore obtenu les privilèges nécessaires sur votre système (à n’effectuer qu’une seule fois).
