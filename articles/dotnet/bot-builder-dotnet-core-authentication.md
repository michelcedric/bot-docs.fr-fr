---
title: Authentifier les activités avec .NET Core | Microsoft Docs
description: Découvrez comment authentifier les activités des bots avec .NET Core.
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/17
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 6df7923caa708ac2b10af37d860dfac317e113a0
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39299201"
---
# <a name="authenticating-activities-using-net-core"></a>Authentifier les activités avec .NET Core

Si vous choisissez de développer votre bot avec [.NET Core](/dotnet/core/index), vous pouvez utiliser [Bot Framework Connector](bot-builder-dotnet-connector.md) pour échanger des messages [d’activité](https://docs.botframework.com/en-us/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html) avec votre bot. Pour pouvoir utiliser le service Connector, vous devez définir le modèle d’authentification adapté à la version du framework ciblée.

Bot Framework Connector.AspNetCore prend en charge les versions suivantes d’ASP.NET :
* Pour .NET Core v1.1/AspNetCore1.x, utiliser **Microsoft.Bot.Connector.AspNetCore 1.x**.
* Pour .NET Core v2.0/AspNetCore2.x, utiliser **Microsoft.Bot.Connector.AspNetCore 2.x**.

Cet article explique comment configurer le modèle d’authentification pour le framework ciblé par le bot.

## <a name="prerequisites"></a>Prérequis

* [Visual Studio 2017](https://www.visualstudio.com/downloads/)
* [.NET Core](https://www.microsoft.com/net/download/windows). Installez la version de .NET Core que vous ciblez (par exemple, .NET Core v1.1 ou .NET Core v 2.0).
* [Inscrire un bot](~/bot-service-quickstart-registration.md). Inscrivez votre bot pour obtenir un ID d’application et un mot de passe, nécessaires dans le cadre du processus d’authentification.

## <a name="create-a-net-core-project"></a>Créer un projet .NET Core

Pour créer votre projet .NET Core, effectuez les actions suivantes :

1. Ouvrez Visual Studio 2017 et cliquez sur **Fichier > Nouveau > Projet…**.
2. Développez le nœud **Visual C#** et cliquez sur **.NET Core**.
3. Choisissez le type de projet **Application web ASP.NET Core** et renseignez les informations du projet (par exemple, les champs Nom, Emplacement et Nom de la solution).
4. Cliquez sur **OK**.
5. Vérifiez que le projet cible *.NET Core* et la version *d’ASP.NET Core* souhaitée. Par exemple, la capture d’écran ci-dessous porte sur **.NET Core** et **ASP.NET Core 2.0** :

![Créer un projet ASP.NET Core v2.0](~/media/dotnet-core-authentication/create-asp-net-core-2x-project.png)
 
  > [!NOTE]
  > Si vous ciblez **ASP.NET Core 2.0**, veillez à ce que la version installée sur votre ordinateur soit au moins la version 2.x.

6. Choisissez le type de projet **API web**.
7. Cliquez sur **OK** pour créer le projet.

## <a name="download-the-nuget-package"></a>Télécharger le package NuGet

Pour utiliser **Bot Framework Connector**, installez le package NuGet adapté à votre version de .NET Core. Pour cela, effectuez les actions suivantes :

1. Dans **l’Explorateur de solutions**, cliquez avec le bouton droit sur le nom du projet, puis sur **Gérer les packages NuGet…**.
2. Cliquez sur **Parcourir** et recherchez **Connector.ASPNetCore**. 
3. Choisissez la version que vous souhaitez cibler. Par exemple, dans la capture d’écran ci-dessous, la version sélectionnée est **2.0.0.3**. Pour installer la version 1.1.3.2, cliquez sur la liste déroulante **Version** et choisissez la version **1.1.3.2**.
![Package NuGet pour ASP.NET Core version 2.0.0.3](~/media/dotnet-core-authentication/nuget-package-net-core-version.png)
4. Cliquez sur **Installer**.

## <a name="update-the-appsettingsjson"></a>Mettre à jour appsettings.json

Avec Bot Framework Connector, l’ID d’application et le mot de passe doivent authentifier le bot. Vous pouvez définir ces valeurs dans le fichier **appsettings.json** de votre projet d’application web.

> [!NOTE]
> Pour trouver les paramètres **AppID** et **AppPassword** du bot, voir [MicrosoftAppID et MicrosoftAppPassword](~/bot-service-manage-overview.md#microsoftappid-and-microsoftapppassword).

### <a name="appsettingsjson-for-net-core-v11"></a>appsettings.json pour .NET Core v1.1 :

```json
{
  "Logging": {
    "IncludeScopes": false,
    "LogLevel": {
      "Default": "Warning"
    }
  },
  "MicrosoftAppId": "<MicrosoftAppId>",
  "MicrosoftAppPassword": "<MicrosoftAppPassword>"
}
```

### <a name="appsettingsjson-for-net-core-v20"></a>appsettings.json pour .NET Core v2.0 :

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
  "MicrosoftAppId": "<MicrosoftAppId>",
  "MicrosoftAppPassword": "<MicrosoftAppPassword>"
}
```

## <a name="update-the-startupcs-class"></a>Mettre à jour la classe startup.cs

Mettez à jour la classe **startup.cs**. Mettez à jour le fragment de code correspondant à la version de votre projet pour que l’authentification fonctionne.

### <a name="startupcs-for-net-core-v11"></a>startup.cs pour .NET Core v1.1 :

```cs
public class Startup
    {
        public Startup(IHostingEnvironment env)
        {
            var builder = new ConfigurationBuilder()
                .SetBasePath(env.ContentRootPath)
                .AddJsonFile("appsettings.json", optional: true, reloadOnChange: true)
                .AddJsonFile($"appsettings.{env.EnvironmentName}.json", optional: true)
                .AddEnvironmentVariables();
            Configuration = builder.Build();
        }

        public IConfigurationRoot Configuration { get; }

        // This method gets called by the runtime. Use this method to add services to the container.
        public void ConfigureServices(IServiceCollection services)
        {
            services.AddSingleton(_ => Configuration);

            services.AddSingleton(typeof(ICredentialProvider), new StaticCredentialProvider(
                Configuration.GetSection(MicrosoftAppCredentials.MicrosoftAppIdKey)?.Value,
                Configuration.GetSection(MicrosoftAppCredentials.MicrosoftAppPasswordKey)?.Value));

            // Add framework services.
            services.AddMvc(options =>
            {
                options.Filters.Add(typeof(TrustServiceUrlAttribute));
            });
        }

        // This method gets called by the runtime. Use this method to configure the HTTP request pipeline.
        public void Configure(IApplicationBuilder app, IServiceProvider serviceProvider, IHostingEnvironment env, ILoggerFactory loggerFactory)
        {
            loggerFactory.AddConsole(Configuration.GetSection("Logging"));
            loggerFactory.AddDebug();

            app.UseStaticFiles();

            ICredentialProvider credentialProvider = serviceProvider.GetService<ICredentialProvider>();

            app.UseBotAuthentication(credentialProvider);

            app.UseMvc();
        }
    }
```

### <a name="startupcs-for-net-core-v20"></a>startup.cs pour .NET Core v2.0 :

```cs
public class Startup
    {
        public Startup(IHostingEnvironment env)
        {
            var builder = new ConfigurationBuilder()
                .SetBasePath(env.ContentRootPath)
                .AddJsonFile("appsettings.json", optional: true, reloadOnChange: true)
                .AddJsonFile($"appsettings.{env.EnvironmentName}.json", optional: true)
                .AddEnvironmentVariables();
            Configuration = builder.Build();
        }

        public IConfiguration Configuration { get; }
        // This method gets called by the runtime. Use this method to add services to the container.
        public void ConfigureServices(IServiceCollection services)
        {
            services.AddSingleton(_ => Configuration);

            var credentialProvider = new StaticCredentialProvider(Configuration.GetSection(MicrosoftAppCredentials.MicrosoftAppIdKey)?.Value,
                Configuration.GetSection(MicrosoftAppCredentials.MicrosoftAppPasswordKey)?.Value);

            services.AddAuthentication(
                    options =>
                    {
                        options.DefaultAuthenticateScheme = JwtBearerDefaults.AuthenticationScheme;
                        options.DefaultChallengeScheme = JwtBearerDefaults.AuthenticationScheme;
                    }
                )
                .AddBotAuthentication(credentialProvider);

            services.AddSingleton(typeof(ICredentialProvider), credentialProvider);

            services.AddMvc(options =>
            {
                options.Filters.Add(typeof(TrustServiceUrlAttribute));
            });
        }


        // This method gets called by the runtime. Use this method to configure the HTTP request pipeline.
        public void Configure(IApplicationBuilder app, IHostingEnvironment env)
        {
            app.UseStaticFiles();
            app.UseAuthentication();
            app.UseMvc();
        }
    }
```

## <a name="create-the-messagescontrollercs-class"></a>Créer la classe MessagesController.cs

Dans **l’Explorateur de solutions**, ajoutez une nouvelle classe vide nommée **MessagesController.cs**. Dans la classe **MessagesController.cs**, mettez à jour la méthode **Post** en utilisant le code ci-dessous. Le bot pourra ainsi envoyer et recevoir des messages pour l’utilisateur.

```cs
private IConfiguration configuration;

public MessagesController(IConfiguration configuration)
{
    this.configuration = configuration;
}


[Authorize(Roles = "Bot")]
[HttpPost]
public async Task<OkResult> Post([FromBody] Activity activity)
{
    if (activity.Type == ActivityTypes.Message)
    {
        //MicrosoftAppCredentials.TrustServiceUrl(activity.ServiceUrl);
        var appCredentials = new MicrosoftAppCredentials(configuration);
        var connector = new ConnectorClient(new Uri(activity.ServiceUrl), appCredentials);

        // return our reply to the user
        var reply = activity.CreateReply("HelloWorld");
        await connector.Conversations.ReplyToActivityAsync(reply);
    }
    else
    {
        //HandleSystemMessage(activity);
    }
    return Ok();
}
```
