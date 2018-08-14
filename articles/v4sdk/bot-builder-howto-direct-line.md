---
title: Création d’un bot et d’un client Direct Line | Microsoft Docs
description: Découvrez comment créer un bot et un client Direct Line avec la version 4 du Kit de développement logiciel (SDK) Bot Builder pour .NET.
keywords: bot direct line, client direct line, canal personnalisé, application console, publication
author: v-royhar
ms.author: v-royhar
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 4/16/18
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: ac96e35763d690c91e6584ff9a840b490e0a32cd
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39298741"
---
# <a name="how-to-create-a-direct-line-bot-and-client"></a>Création d’un bot et d’un client Direct Line

Les bots Direct Line de Microsoft Bot Framework peuvent fonctionner avec un client que vous avez personnalisé. Les bots Direct Line sont très semblables aux bots standard. Ils n’ont simplement pas besoin d’utiliser les canaux fournis.

Les clients Direct Line peuvent être écrits pour jouer le rôle que vous souhaitez leur attribuer. Vous pouvez écrire un client Android, un client iOS ou même une application cliente console.

Dans cet article, vous allez apprendre à créer et déployer un bot Direct Line, et à créer et exécuter une application cliente console Direct Line.

## <a name="create-your-bot"></a>Créer votre bot

Chaque langage suit un chemin d’accès différent pour créer le bot. JavaScript crée le bot dans Azure, puis modifie le code, tandis que C# crée le bot localement, puis le publie sur Azure ; toutefois, ces deux méthodes sont valides et utilisables avec l’un ou l’autre langage. Pour plus d’informations sur la publication de votre bot sur Azure, consultez l’article [Déployer votre bot sur Azure](../bot-builder-howto-deploy-azure.md).

# <a name="ctabcscreatebot"></a>[C#](#tab/cscreatebot)

### <a name="create-the-solution-in-visual-studio"></a>Créer la solution dans Visual Studio

Pour créer la solution pour le bot Direct Line dans Visual Studio 2015 ou une version ultérieure :

1. Créez une **Application web ASP.NET Core** dans **Visual C#** > **.NET Core**.

1. Attribuez-lui le nom **DirectLineBotSample**, puis cliquez sur **OK**.

1. Assurez-vous que **.NET Core** et **ASP.NET Core 2.0** sont sélectionnés, choisissez le modèle de projet **Vide**, puis cliquez sur **OK**.

#### <a name="add-dependencies"></a>Ajout de dépendances

1. Dans **l’Explorateur de solutions**, cliquez avec le bouton droit sur **Dépendances**, puis choisissez **Gérer les packages NuGet**.

1. Cliquez sur **Parcourir**, puis assurez-vous que la case **Inclure la préversion** est cochée.

1. Recherchez et installez les packages NuGet suivants :
    - Microsoft.Bot.Builder
    - Microsoft.Bot.Builder.Core.Extensions
    - Microsoft.Bot.Builder.Integration.AspNet.Core
    - Newtonsoft.Json

### <a name="create-the-appsettingsjson-file"></a>Créer le fichier appsettings.json

Le fichier appsettings.json contient l’ID d’application Microsoft, le mot de passe d’application et la chaîne de connexion de données. Étant donné que ce bot ne stocke pas d’informations d’état, la chaîne de connexion de données reste vide. Et si vous utilisez uniquement l’application Bot Framework Emulator, toutes ces informations peuvent rester vides.

Pour créer un fichier **appsettings.json** :

1. Cliquez avec le bouton droit sur le projet **DirectLineBotSample**, choisissez **Ajouter** > **Nouvel élément**.

1. Dans **ASP.NET Core**, cliquez sur **Fichier de configuration ASP.NET**.

1. Entrez le nom **appsettings.json**, puis cliquez sur **Ajouter**.

Remplacez le contenu du fichier appsettings.json par le code suivant :

```json
{
    "Logging": {
        "IncludeScopes": false,
        "Debug": {
            "LogLevel": {
                "Default": "Warning"
            }
        },
        "Console": {
            "LogLevel": {
                "Default": "Warning"
            }
        }
    },

    "MicrosoftAppId": "",
    "MicrosoftAppPassword": "",
    "DataConnectionString": ""
}
```

### <a name="edit-startupcs"></a>Modifier le fichier Startup.cs

Remplacez le contenu du fichier Startup.cs par le code suivant :

**Remarque :** **DirectBot** sera défini à l’étape suivante ; vous pouvez donc ignorer le trait de soulignement rouge qui le recouvre.

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Hosting;
using Microsoft.AspNetCore.Http;
using Microsoft.Bot.Builder.BotFramework;
using Microsoft.Bot.Builder.Integration.AspNet.Core;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;

namespace DirectLineBotSample
{
    public class Startup
    {
        public IConfiguration Configuration { get; }

        public Startup(IHostingEnvironment env)
        {
            var builder = new ConfigurationBuilder()
                .SetBasePath(env.ContentRootPath)
                .AddJsonFile("appsettings.json", optional: true, reloadOnChange: true)
                .AddJsonFile($"appsettings.{env.EnvironmentName}.json", optional: true)
                .AddEnvironmentVariables();
            Configuration = builder.Build();
        }

        // This method gets called by the runtime. Use this method to add services to the container.
        public void ConfigureServices(IServiceCollection services)
        {
            services.AddSingleton(_ => Configuration);
            services.AddBot<DirectBot>(options =>
            {
                options.CredentialProvider = new ConfigurationCredentialProvider(Configuration);
            });
        }

        // This method gets called by the runtime. Use this method to configure the HTTP request pipeline.
        public void Configure(IApplicationBuilder app, IHostingEnvironment env)
        {
            if (env.IsDevelopment())
            {
                app.UseDeveloperExceptionPage();
            }

            app.UseDefaultFiles();
            app.UseStaticFiles();
            app.UseBotFramework();
        }
    }
}
```

### <a name="create-the-directbot-class"></a>Créer la classe DirectBot

La classe DirectBot contient la majeure partie de la logique de ce bot.

1. Cliquez avec le bouton droit sur **DirectLineBotSample** > **Ajouter** > **Nouvel élément**.

1. Dans **ASP.NET Core**, cliquez sur **Classe**.

1. Entrez le nom **DirectBot.cs**, puis cliquez sur **Ajouter**.

Remplacez le contenu du fichier DirectBot.cs par le code suivant :

```csharp
using System.Threading.Tasks;
using Microsoft.Bot;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Schema;

namespace DirectLineBotSample
{
    public class DirectBot : IBot
    {
        public Task OnTurn(ITurnContext context)
        {
            // Respond to the various activity types.
            switch (context.Activity.Type)
            {
                case ActivityTypes.Message:
                    // Respond to the incoming text message.
                    RespondToMessage(context);
                    break;

                case ActivityTypes.ConversationUpdate:
                    break;

                case ActivityTypes.ContactRelationUpdate:
                    break;

                case ActivityTypes.Typing:
                    break;

                case ActivityTypes.Ping:
                    break;

                case ActivityTypes.DeleteUserData:
                    break;
            }

            return Task.CompletedTask;
        }

        /// <summary>
        /// Responds to the incoming message by either sending a hero card, an image, 
        /// or echoing the user's message.
        /// </summary>
        /// <param name="context">The context of this conversation.</param>
        private void RespondToMessage(ITurnContext context)
        {
            switch (context.Activity.Text.Trim().ToLower())
            {
                case "hi":
                case "hello":
                case "help":
                    // Send the user an instruction message.
                    context.SendActivity("Welcome to the Bot to showcase the DirectLine API. " +
                    "Send \"Show me a hero card\" or \"Send me a BotFramework image\" to see how the " +
                    "DirectLine client supports custom channel data. Any other message will be echoed.");
                    break;

                case "show me a hero card":
                    // Create the hero card.
                    HeroCard heroCard = new HeroCard()
                    {
                        Title = "Sample Hero Card",
                        Text = "Displayed in the DirectLine client"
                    };

                    // Attach the hero card to a new activity.
                    context.SendActivity(MessageFactory.Attachment(heroCard.ToAttachment()));
                    break;

                case "send me a botframework image":
                    // Create the image attachment.
                    Attachment imageAttachment = new Attachment()
                    {
                        ContentType = "image/png",
                        ContentUrl = "https://docs.microsoft.com/en-us/bot-framework/media/how-it-works/architecture-resize.png",
                    };

                    // Attach the image attachment to a new activity.
                    context.SendActivity(MessageFactory.Attachment(imageAttachment));
                    break;

                default:
                    // No command was encountered. Echo the user's message.
                    context.SendActivity($"You said \"{context.Activity.Text}\"");
                    break;
            }
        }
    }
}
```

**Remarque :** vous n’avez pas besoin de modifier le fichier Program.cs existant.

Pour vérifier que tout fonctionne correctement, appuyez sur **F6** pour générer le projet. Aucun avertissement ni erreur ne doivent s’afficher.

### <a name="publish-the-bot-from-visual-studio"></a>Publier le bot à partir de Visual Studio

1. Dans Visual Studio, ouvrez l’Explorateur de solutions, cliquez avec le bouton droit sur le projet **DirectLineBotSample**, puis cliquez sur **Définir en tant que projet de démarrage**.

    ![Page de publication Visual Studio](media/bot-builder-howto-direct-line/visual-studio-publish-page.png)

1. Cliquez de nouveau avec le bouton droit sur **DirectLineBotSample**, puis cliquez sur **Publier**. La page **Publier** Visual Studio apparaît.

1. Si vous disposez déjà d’un profil de publication pour ce bot, sélectionnez-le, cliquez sur le bouton **Publier**, puis passez à la section suivante.

1. Pour créer un profil de publication, sélectionnez l’icône **Microsoft Azure App Service**.

1. Sélectionnez **Créer nouveau**.

1. Cliquez sur le bouton **Publier** . La boîte de dialogue **Créer App Service** apparaît.

    ![Boîte de dialogue Créer App Service](media/bot-builder-howto-direct-line/create-app-service-dialog.png)

    - Attribuez à l’application un nom que vous pourrez retrouver facilement par la suite. Par exemple, vous pouvez ajouter votre identifiant de messagerie au début du nom de l’application.

    - Vérifiez que vous utilisez l’abonnement approprié.

    - Veillez à utiliser le groupe de ressources approprié. Le groupe de ressources figure sur le panneau **Vue d’ensemble** de votre bot. **Remarque :** un groupe de ressources incorrect est difficile à corriger.

    - Vérifiez que vous utilisez le plan App Service correct.
    
1. Cliquez sur le bouton **Créer**. Visual Studio commence à déployer votre bot.

Une fois que votre bot a été publié, un navigateur s’affiche avec le point de terminaison d’URL de votre bot.

### <a name="create-bot-channels-registration-bot-on-microsoft-azure"></a>Créer un bot Bot Channels Registration sur Microsoft Azure

Le bot Direct Line peut être hébergé sur n’importe quelle plateforme. Dans cet exemple, le bot est hébergé sur Microsoft Azure. 

Pour créer le bot sur Microsoft Azure :

1. Dans le Portail Microsoft Azure, cliquez sur **Créer une ressource**, puis recherchez « Bot Channels Registration ».

1. Cliquez sur **Créer**. Le panneau Bot Channels Registration apparaît.

    ![Panneau Bot Channels Registration contenant notamment les champs de nom de bot, d’abonnement, de groupe de ressources, d’emplacement, de niveau tarifaire et de point de terminaison de messagerie](media/bot-builder-howto-direct-line/bot-service-registration-blade.png)

1. Dans le panneau Bot Channels Registration, renseignez les champs **Nom du robot**, **Abonnement**, **Groupe de ressources**, **Emplacement** et **Niveau tarifaire**.

1. Laissez le champ **Messaging endpoint** (Point de terminaison de messagerie) vide. Cette valeur sera renseignée ultérieurement.

1. Cliquez sur **Microsoft App ID and password** (ID et mot de passe d’application Microsoft), puis sur **Auto create App ID and password** (Créer automatiquement l’ID et le mot de passe d’application).

1. Cochez la case **Épingler au tableau de bord**.

1. Cliquez sur le bouton **Créer**.

Le déploiement nécessite quelques minutes, puis le bot apparaît dans votre tableau de bord.

### <a name="update-the-appsettingsjson-file"></a>Mettre à jour le fichier appsettings.json

1. Cliquez sur le bot de votre tableau de bord, ou accédez à votre nouvelle ressource en cliquant sur **Toutes les ressources** et en recherchant le nom de votre Bot Channels Registration.

1. Cliquez sur **Vue d’ensemble**.

1. Copiez le nom du groupe de ressources dans la chaîne **botId** du fichier **Program.cs** du projet **DirectLineClientSample**.

1. Cliquez sur **Paramètres**, puis sur **Gérer** en regard du champ **Microsoft App ID** (ID d’application Microsoft).

1. Copiez l’ID d’application et collez-le dans le champ **« MicrosoftAppId »** du fichier **appsettings.json**.

1. Cliquez sur le bouton **Générer un nouveau mot de passe**.

1. Copiez le nouveau mot de passe et collez-le dans le champ **« MicrosoftAppPassword »** du fichier **appsettings.json**.

### <a name="set-the-messaging-endpoint"></a>Définir le point de terminaison de messagerie

Pour ajouter le point de terminaison à votre bot :

1. Copiez l’adresse URL à partir du navigateur qui s’est affiché après la publication de votre bot.

    ![Panneau de paramètres de Bot Channels Registration mettant en évidence le point de terminaison de messagerie](media/bot-builder-howto-direct-line/bot-channels-registration-settings.png)

1. Affichez le panneau **Paramètres** de votre bot.

1. Collez l’adresse dans le champ **Messaging endpoint** (Point de terminaison de messagerie).

1. Modifiez l’adresse pour qu’elle commence par « https:// » et se termine par « /api/messages ». Par exemple, si l’adresse copiée à partir du navigateur est « http://v-royhar-dlbot-directlinebotsample20180329044602.azurewebsites.net », remplacez-la par « https://v-royhar-dlbot-directlinebotsample20180329044602.azurewebsites.net/api/messages ».

1. Cliquez sur le bouton **Enregistrer** du panneau Paramètres.

**Remarque :** si la publication échoue, vous devrez peut-être arrêter votre bot sur Azure avant de pouvoir publier des modifications pour ce bot.

1. Recherchez le nom de votre App Service (figurant entre « https:// » et « .azurewebsites.net ».

1. Recherchez cette ressource dans la zone « Toutes les ressources » du Portail Azure.

1. Cliquez sur la ressource App Service. Le panneau App Service s’affiche.

1. Cliquez sur le bouton **Arrêter** du panneau App Service.

1. Dans Visual Studio, publiez de nouveau votre bot.

1. Cliquez sur le bouton **Démarrer** du panneau App Service.

### <a name="test-your-bot-in-webchat"></a>Tester votre bot dans la Discussion Web

Pour vérifier que votre bot fonctionne, testez-le dans la Discussion Web :

1. Dans le panneau Bot Channels Registration de votre bot, cliquez sur **Paramètres**, puis sur **Test in Web Chat** (Tester dans la Discussion Web).

1. Tapez « Hi » (Bonjour). Le bot doit répondre par un message de bienvenue.

1. Tapez « show me a hero card » (montrez-moi une carte de bannière). Le bot doit afficher une carte de bannière.

1. Tapez « send me a botframework image » (envoyez-moi une image botframework). Le bot doit afficher une image provenant de la documentation de Bot Framework.

1. Tapez toute autre chaîne, et le bot doit vous répondre « You said » (Vous avez dit), suivi de votre message entre guillemets.

### <a name="troubleshooting"></a>Résolution de problèmes

Si votre bot ne fonctionne pas, vérifiez ou réentrez vos ID et mot de passe d’application Microsoft. Même si vous l’avez déjà copié auparavant, comparez l’ID d’application Microsoft figurant dans le panneau de paramètres de votre Bot Channels Registration à la valeur du champ **« MicrosoftAppId »** indiquée dans le fichier appsettings.json.

Vérifiez votre mot de passe ou créez et utilisez un nouveau mot de passe : 

1. Cliquez sur **Gérer** en regard du champ **Microsoft App ID** (ID d’application Microsoft) dans votre panneau Bot Channels Registration.

1. Connectez-vous au portail d’inscription des applications.

1. Vérifiez que les trois premières lettres de votre mot de passe correspondent à la valeur du champ **« MicrosoftAppPassword »** du fichier **appsettings.json**. 

1. Si les valeurs ne correspondent pas, générez un nouveau mot de passe et stockez cette valeur dans le champ **« MicrosoftAppPassword »** du fichier **appsettings.json**.

# <a name="javascripttabjscreatebot"></a>[JavaScript](#tab/jscreatebot)

### <a name="create-web-app-bot-on-microsoft-azure"></a>Créer un bot Web App Bot sur Microsoft Azure

Pour créer le bot sur Microsoft Azure : 

1. Dans le Portail Microsoft Azure, cliquez sur **Créer une ressource**, puis recherchez « Web App Bot ».

1. Cliquez sur **Créer**. Le panneau Web App Bot apparaît.

![Inscription dans Web App Bot](media/bot-builder-howto-direct-line/web-app-bot-registration.png)

1. Dans le panneau Web App Bot, renseignez les champs **Nom du robot**, **Abonnement**, **Groupe de ressources**, **Emplacement**, **Niveau tarifaire** et **Nom de l’application**.

1. Sélectionnez le modèle de bot que vous souhaitez utiliser. **Version du Kit de développement logiciel (SDK) : SDK v4** et **SDK language (Langage du Kit de développement logiciel (SDK)) : Node.js** 

1. Sélectionnez le modèle de préversion v4 de base. 

1. Choisissez vos valeurs **Plan App Service/Emplacement**, **Stockage Azure** et **Emplacement d’Application Insights**.

1. Cochez la case Épingler au tableau de bord.

1. Cliquez sur le bouton Créer.

1. Attendez que votre bot soit déployé. Étant donné que vous avez coché la case Épingler au tableau de bord, votre bot apparaît sur votre tableau de bord.

### <a name="edit-your-direct-line-bot"></a>Modifier votre bot Direct Line

À présent, ajoutons certaines autres fonctionnalités au bot d’écho.

1. Dans le panneau Web App Bot, cliquez sur Générer. 

1. Téléchargez le fichier zip. 

1. Extrayez le fichier zip dans un répertoire local.

1. Accédez au dossier extrait et ouvrez les fichiers sources dans votre environnement de développement intégré (IDE) favori.

1. Dans le fichier app.js, copiez et collez le code ci-après.

```javascript
// Copyright (c) Microsoft Corporation. All rights reserved.
// Licensed under the MIT License.

const { BotFrameworkAdapter, MessageFactory, CardFactory } = require('botbuilder');
const restify = require('restify');

// Create server
let server = restify.createServer();
server.listen(process.env.port || process.env.PORT || 3978, function () {
    console.log(`${server.name} listening to ${server.url}`);
});

// Create adapter
const adapter = new BotFrameworkAdapter({
    appId: process.env.MicrosoftAppId,
    appPassword: process.env.MicrosoftAppPassword
});

// Responds to the incoming message by either sending a hero card, an image, 
// or echoing the user's message.

// Listen for incoming requests 
server.post('/api/messages', (req, res) => {
    // Route received request to adapter for processing
    adapter.processActivity(req, res, (context) => {
        if (context.activity.type === 'message') {
            const text = (context.activity.text || '').trim().toLowerCase()

            switch(text){
                case 'hi':
                case "hello":
                case "help":
                    // Send the user an instruction message.
                    return context.sendActivity("Welcome to the Bot to showcase the DirectLine API. " +
                    "Send \"Show me a hero card\" or \"Send me a BotFramework image\" to see how the " +
                    "DirectLine client supports custom channel data. Any other message will be echoed.");
                    break;

                case 'show me a hero card':
                    // Create the hero card.
                    const message = MessageFactory.attachment(
                        CardFactory.heroCard(   
                        'Sample Hero Card', //cards title
                        'Displayed in the DirectLine client' //cards text
                        )
                    );
                    return context.sendActivity(message);
                    break;
                    
                case 'send me a botframework image':
                    // Create the image attachment.
                    const imageOrVideoMessage = MessageFactory.contentUrl('https://docs.microsoft.com/en-us/azure/bot-service/media/how-it-works/architecture-resize.png', 'image/png')
                    return context.sendActivity(imageOrVideoMessage);
                    break;
                
                default:
                    // No command was encountered. Echo the user's message.
                    return context.sendActivity(`You said ${context.activity.text}`);
                    break;
                    
            }
        }
    });
});

```

Lorsque vous êtes prêt, vous pouvez republier les sources sur Azure. [Pour découvrir comment republier votre bot sur Azure, exécutez cette procédure.](../bot-service-build-download-source-code.md#publish-node-bot-source-code-to-azure)

---


## <a name="create-the-console-client-app"></a>Créer l’application cliente console

# <a name="ctabcsclientapp"></a>[C#](#tab/csclientapp)

L’application cliente console fonctionne dans deux threads. Le thread principal accepte l’entrée utilisateur et envoie les messages au bot. Le thread secondaire interroge le bot une fois par seconde pour récupérer tous les messages émanant de ce dernier, puis affiche les messages reçus.

Pour créer le projet console :

1. Dans l’Explorateur de solutions, cliquez avec le bouton droit sur la **Solution « DirectLineBotSample »**.

1. Choisissez **Ajouter** > **Nouveau projet**.

1. Dans **Visual C#** > **Bureau classique Windows*, choisissez **Application console (.NET Framework)**.
 
    **Remarque :** ne choisissez pas **Application console (.NET Core)**.

1. Dans le champ de nom, entrez **DirectLineClientSample**.

1. Cliquez sur **OK**.

### <a name="add-the-nuget-packages-to-the-console-app"></a>Ajouter les packages NuGet à l’application console

1. Cliquez avec le bouton droit sur **Références**.

1. Cliquez sur **Gérer les packages NuGet...**

1. Cliquez sur **Parcourir**. Assurez-vous que la case **Inclure la préversion** est cochée.

1. Recherchez et installez les packages NuGet suivants :
    - Microsoft.Bot.Connector.DirectLine (v3.0.2)
    - Newtonsoft.Json

### <a name="edit-the-programcs-file"></a>Modifier le fichier Program.cs

Remplacez le contenu du fichier **Program.cs** du projet DirectLineClientSample par le code suivant :

```csharp
using System;
using System.Diagnostics;
using System.Linq;
using System.Threading.Tasks;
using Microsoft.Bot.Connector.DirectLine;
using Newtonsoft.Json;

namespace DirectLineClientSample
{
    class Program
    {
        // ************
        // Replace the following values with your Direct Line secret and the name of your bot resource ID.
        //*************
        private static string directLineSecret = "*** Replace with Direct Line secret ***";
        private static string botId = "*** Replace with the resource name of your bot ***";

        // This gives a name to the bot user.
        private static string fromUser = "DirectLineClientSampleUser";

        static void Main(string[] args)
        {
            StartBotConversation().Wait();
        }


        /// <summary>
        /// Drives the user's conversation with the bot.
        /// </summary>
        /// <returns></returns>
        private static async Task StartBotConversation()
        {
            // Create a new Direct Line client.
            DirectLineClient client = new DirectLineClient(directLineSecret);

            // Start the conversation.
            var conversation = await client.Conversations.StartConversationAsync();

            // Start the bot message reader in a separate thread.
            new System.Threading.Thread(async () => await ReadBotMessagesAsync(client, conversation.ConversationId)).Start();

            // Prompt the user to start talking to the bot.
            Console.Write("Type your message (or \"exit\" to end): ");

            // Loop until the user chooses to exit this loop.
            while (true)
            {
                // Accept the input from the user.
                string input = Console.ReadLine().Trim();

                // Check to see if the user wants to exit.
                if (input.ToLower() == "exit")
                {
                    // Exit the app if the user requests it.
                    break;
                }
                else
                {
                    if (input.Length > 0)
                    {
                        // Create a message activity with the text the user entered.
                        Activity userMessage = new Activity
                        {
                            From = new ChannelAccount(fromUser),
                            Text = input,
                            Type = ActivityTypes.Message
                        };

                        // Send the message activity to the bot.
                        await client.Conversations.PostActivityAsync(conversation.ConversationId, userMessage);
                    }
                }
            }
        }


        /// <summary>
        /// Polls the bot continuously and retrieves messages sent by the bot to the client.
        /// </summary>
        /// <param name="client">The Direct Line client.</param>
        /// <param name="conversationId">The conversation ID.</param>
        /// <returns></returns>
        private static async Task ReadBotMessagesAsync(DirectLineClient client, string conversationId)
        {
            string watermark = null;

            // Poll the bot for replies once per second.
            while (true)
            {
                // Retrieve the activity set from the bot.
                var activitySet = await client.Conversations.GetActivitiesAsync(conversationId, watermark);
                watermark = activitySet?.Watermark;

                // Extract the activies sent from our bot.
                var activities = from x in activitySet.Activities
                                 where x.From.Id == botId
                                 select x;

                // Analyze each activity in the activity set.
                foreach (Activity activity in activities)
                {
                    // Display the text of the activity.
                    Console.WriteLine(activity.Text);

                    // Are there any attachments?
                    if (activity.Attachments != null)
                    {
                        // Extract each attachment from the activity.
                        foreach (Attachment attachment in activity.Attachments)
                        {
                            switch (attachment.ContentType)
                            {
                                // Display a hero card.
                                case "application/vnd.microsoft.card.hero":
                                    RenderHeroCard(attachment);
                                    break;

                                // Display the image in a browser.
                                case "image/png":
                                    Console.WriteLine($"Opening the requested image '{attachment.ContentUrl}'");
                                    Process.Start(attachment.ContentUrl);
                                    break;
                            }
                        }
                    }

                    // Redisplay the user prompt.
                    Console.Write("\nType your message (\"exit\" to end): ");
                }

                // Wait for one second before polling the bot again.
                await Task.Delay(TimeSpan.FromSeconds(1)).ConfigureAwait(false);
            }
        }


        /// <summary>
        /// Displays the hero card on the console.
        /// </summary>
        /// <param name="attachment">The attachment that contains the hero card.</param>
        private static void RenderHeroCard(Attachment attachment)
        {
            const int Width = 70;
            // Function to center a string between asterisks.
            Func<string, string> contentLine = (content) => string.Format($"{{0, -{Width}}}", string.Format("{0," + ((Width + content.Length) / 2).ToString() + "}", content));

            // Extract the hero card data.
            var heroCard = JsonConvert.DeserializeObject<HeroCard>(attachment.Content.ToString());

            // Display the hero card.
            if (heroCard != null)
            {
                Console.WriteLine("/{0}", new string('*', Width + 1));
                Console.WriteLine("*{0}*", contentLine(heroCard.Title));
                Console.WriteLine("*{0}*", new string(' ', Width));
                Console.WriteLine("*{0}*", contentLine(heroCard.Text));
                Console.WriteLine("{0}/", new string('*', Width + 1));
            }
        }
    }
}
```

Pour vérifier que tout fonctionne correctement, appuyez sur **F6** pour générer le projet. Aucun avertissement ni erreur ne doivent s’afficher.

# <a name="javascripttabjsclientapp"></a>[JavaScript](#tab/jsclientapp)

### <a name="create-a-direct-line-client"></a>Créer un client Direct Line 

Maintenant que vous avez déployé votre bot Web App Bot, nous pouvons créer un client Direct Line.

```
    md DirectLineClient
    cd DirectLineClient
    npm init
```

Ensuite, installez les packages ci-après à partir de npm :

```
    npm install --save open
    npm install --save request
    npm install --save request-promise
    npm install --save swagger-client
```

Créez un fichier **app.js**. Copiez et collez le code ci-après dans ce fichier.

Nous obtiendrons la valeur directLineSecret à l’étape suivante.
```javascript
var Swagger = require('swagger-client');
var open = require('open');
var rp = require('request-promise');

// config items
var pollInterval = 1000;
// Change the Direct Line Secret to your own 
var directLineSecret = 'your secret here';
var directLineClientName = 'DirectLineClient';
var directLineSpecUrl = 'https://docs.botframework.com/en-us/restapi/directline3/swagger.json';

var directLineClient = rp(directLineSpecUrl)
    .then(function (spec) {
        // client
        return new Swagger({
            spec: JSON.parse(spec.trim()),
            usePromise: true
        });
    })
    .then(function (client) {
        // add authorization header to client
        client.clientAuthorizations.add('AuthorizationBotConnector', new Swagger.ApiKeyAuthorization('Authorization', 'Bearer ' + directLineSecret, 'header'));
        return client;
    })
    .catch(function (err) {
        console.error('Error initializing DirectLine client', err);
    });

// once the client is ready, create a new conversation
directLineClient.then(function (client) {
    client.Conversations.Conversations_StartConversation()                          // create conversation
        .then(function (response) {
            return response.obj.conversationId;
        })                            // obtain id
        .then(function (conversationId) {
            sendMessagesFromConsole(client, conversationId);                        // start watching console input for sending new messages to bot
            pollMessages(client, conversationId);                                   // start polling messages from bot
        })
        .catch(function (err) {
            console.error('Error starting conversation', err);
        });
});

// Read from console (stdin) and send input to conversation using DirectLine client
function sendMessagesFromConsole(client, conversationId) {
    var stdin = process.openStdin();
    process.stdout.write('Command> ');
    stdin.addListener('data', function (e) {
        var input = e.toString().trim();
        if (input) {
            // exit
            if (input.toLowerCase() === 'exit') {
                return process.exit();
            }

            // send message
            client.Conversations.Conversations_PostActivity(
                {
                    conversationId: conversationId,
                    activity: {
                        textFormat: 'plain',
                        text: input,
                        type: 'message',
                        from: {
                            id: directLineClientName,
                            name: directLineClientName
                        }
                    }
                }).catch(function (err) {
                    console.error('Error sending message:', err);
                });

            process.stdout.write('Command> ');
        }
    });
}

// Poll Messages from conversation using DirectLine client
function pollMessages(client, conversationId) {
    console.log('Starting polling message for conversationId: ' + conversationId);
    var watermark = null;
    setInterval(function () {
        client.Conversations.Conversations_GetActivities({ conversationId: conversationId, watermark: watermark })
            .then(function (response) {
                watermark = response.obj.watermark;                                 // use watermark so subsequent requests skip old messages
                return response.obj.activities;
            })
            .then(printMessages);
    }, pollInterval);
}

// Helpers methods
function printMessages(activities) {
    if (activities && activities.length) {
        // ignore own messages
        activities = activities.filter(function (m) { return m.from.id !== directLineClientName });

        if (activities.length) {
            process.stdout.clearLine();
            process.stdout.cursorTo(0);

            // print other messages
            activities.forEach(printMessage);

            process.stdout.write('Command> ');
        }
    }
}

function printMessage(activity) {
    if (activity.text) {
        console.log(activity.text);
    }

    if (activity.attachments) {
        activity.attachments.forEach(function (attachment) {
            switch (attachment.contentType) {
                case "application/vnd.microsoft.card.hero":
                    renderHeroCard(attachment);
                    break;

                case "image/png":
                    console.log('Opening the requested image ' + attachment.contentUrl);
                    open(attachment.contentUrl);
                    break;
            }
        });
    }
}

function renderHeroCard(attachment) {
    var width = 70;
    var contentLine = function (content) {
        return ' '.repeat((width - content.length) / 2) +
            content +
            ' '.repeat((width - content.length) / 2);
    }

    console.log('/' + '*'.repeat(width + 1));
    console.log('*' + contentLine(attachment.content.title) + '*');
    console.log('*' + ' '.repeat(width) + '*');
    console.log('*' + contentLine(attachment.content.text) + '*');
    console.log('*'.repeat(width + 1) + '/');
}
```

---

## <a name="configure-the-direct-line-channel"></a>Configurer le canal Direct Line

L’ajout du canal Direct Line convertit ce bot en bot Direct Line.

Pour configurer le canal Direct Line :

![Panneau de connexion aux canaux avec le bouton de canal Direct Line en surbrillance](media/bot-builder-howto-direct-line/direct-line-channel-button.png)

1. Dans le panneau **Bot Channels Registration**, cliquez sur **Channels** (Canaux). Le panneau **Connect to Channels** (Connecter aux canaux) apparaît.

    ![Panneau de configuration Direct Line présentant les deux clés secrètes](media/bot-builder-howto-direct-line/configure-direct-line.png)

1. Dans le panneau **Connect to Channels** (Connecter aux canaux), cliquez sur le bouton **Configure Direct Line channel** (Configurer le canal Direct Line). 

1. Si la case **3.0** n’est pas cochée, cochez-la.

1. Cliquez sur **Show** (Afficher) pour au moins une clé secrète.

1. Copiez l’une des clés secrètes et collez-la dans votre application cliente, dans la chaîne **directLineSecret**.

1. Cliquez sur Done.

## <a name="run-the-client-app"></a>Exécuter l’application cliente

# <a name="ctabcsrunclient"></a>[C#](#tab/csrunclient)

Votre bot est désormais prêt à communiquer avec l’application cliente console Direct Line. Pour exécuter l’application console, procédez comme suit :

1. Dans Visual Studio, cliquez avec le bouton droit sur le projet **DirectLineClientSample** et sélectionnez **Définir en tant que projet de démarrage**.

1. Examinez le fichier **Program.cs** du projet **DirectLineClientSample**.

    - Vérifiez que le code secret Direct Line figure dans la chaîne **directLineSecret**.

1. Si la valeur **directLineSecret** est incorrecte, accédez au Portail Azure, cliquez sur votre bot Direct Line, cliquez sur **Channels** (Canaux), cliquez sur **Configure Direct Line channel** (Configurer le canal Direct Line) (ou sur **Edit** (Modifier)), affichez une clé, puis copiez cette dernière dans la chaîne **directLineSecret**.

    - Vérifiez que le groupe de ressources figure dans la chaîne **botId**.

1. Dans le cas contraire, accédez au panneau **Vue d’ensemble**, puis copiez et collez le **Groupe de ressources** dans la chaîne **botId**.

1. Appuyez sur F5 ou sur Démarrer le débogage.

L’application cliente console démarre. Pour tester l’application : 

1. Entrez « Hi » (Bonjour). Le bot doit afficher « You said "Hi"» (Vous avec dit « Bonjour »).

1. Entrez « show me a hero card » (montrez-moi une carte de bannière). Le bot doit afficher une carte de bannière.

1. Entrez « send me a botframework image » (envoyez-moi une image botframework). Le bot lance un navigateur affichant une image provenant de la documentation de Bot Framework.

1. Entrez toute autre chaîne, et le bot doit vous répondre « You said » (Vous avez dit), suivi de votre message entre guillemets.

# <a name="javascripttabjsrunclient"></a>[JavaScript](#tab/jsrunclient)

Pour exécuter l’exemple, vous devez exécuter votre application DirectLineClient.

1. Ouvrez une console CMD et accédez au répertoire DirectLineClient.

1. Exécutez `node app.js`

Pour tester les messages personnalisés, tapez dans la console du client « show me a hero card » (montrez-moi une carte de bannière) ou « send me a botframework image » (envoyez-moi une image botframework), et vous devriez alors voir le résultat suivant.

![Résultat](media/bot-builder-howto-direct-line/outcome.png)

---


### <a name="next-steps"></a>Étapes suivantes
