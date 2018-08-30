---
title: Créer un bot avec le Kit de développement logiciel (SDK) Bot Builder pour Java | Microsoft Docs
description: Créez rapidement un bot à l’aide du Kit de développement logiciel (SDK) Bot Builder pour Java.
keywords: Kit de développement logiciel (SDK) Bot Builder, créer un bot, démarrage rapide, java, prise en main
author: jonathanfingold
ms.author: jonathanfingold
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 05/02/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 3b618bfb7cd1a462390aee4d564778c8ec0a7247
ms.sourcegitcommit: d486dd088b87a44fc8142f7a08877ff993861a42
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/26/2018
ms.locfileid: "42928428"
---
# <a name="create-a-bot-with-the-bot-builder-sdk-for-java"></a>Créer un bot avec le Kit de développement logiciel (SDK) Bot Builder pour Java
[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Le Kit de développement logiciel (SDK) Bot Builder pour Java permet aux développeurs Java d’écrire des bots dans un environnement qui leur est familier. Le Kit de développement logiciel (SDK) v4 est disponible en préversion. Pour plus d’informations, consultez le [référentiel GitHub](https://github.com/Microsoft/botbuilder-java) Java.

> [!NOTE]
> Nos exemples de code et documents sont actuellement adaptés à la version 1.8 de Java.

## <a name="getting-started"></a>Mise en route

Le Kit de développement logiciel (SDK) v4 inclut une série de [bibliothèques](https://github.com/Microsoft/botbuilder-java/tree/master/libraries). Pour les générer localement, consultez [Building the SDK](https://github.com/Microsoft/botbuilder-java/wiki/building-the-sdk) (Génération du SDK).

- Installez le [Bot Framework Emulator](https://github.com/Microsoft/BotFramework-Emulator/releases).

### <a name="create-echobot"></a>Créer l’EchoBot

Dans le fichier App.java, ajoutez ce qui suit :

```Java
package com.microsoft.bot.connector.sample;

import com.fasterxml.jackson.databind.DeserializationFeature;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.microsoft.aad.adal4j.AuthenticationException;
import com.microsoft.bot.connector.customizations.CredentialProvider;
import com.microsoft.bot.connector.customizations.CredentialProviderImpl;
import com.microsoft.bot.connector.customizations.JwtTokenValidation;
import com.microsoft.bot.connector.customizations.MicrosoftAppCredentials;
import com.microsoft.bot.connector.implementation.ConnectorClientImpl;
import com.microsoft.bot.schema.models.Activity;
import com.microsoft.bot.schema.models.ActivityTypes;
import com.microsoft.bot.schema.models.ResourceResponse;
import com.sun.net.httpserver.HttpExchange;
import com.sun.net.httpserver.HttpHandler;
import com.sun.net.httpserver.HttpServer;

import java.io.IOException;
import java.io.InputStream;
import java.net.InetSocketAddress;
import java.net.URLDecoder;
import java.util.logging.Level;
import java.util.logging.Logger;

public class App {
    private static final Logger LOGGER = Logger.getLogger( App.class.getName() );
    private static String appId = "";       // <-- app id -->
    private static String appPassword = ""; // <-- app password -->

    public static void main( String[] args ) throws IOException {
        CredentialProvider credentialProvider = new CredentialProviderImpl(appId, appPassword);
        HttpServer server = HttpServer.create(new InetSocketAddress(3978), 0);
        server.createContext("/api/messages", new MessageHandle(credentialProvider));
        server.setExecutor(null);
        server.start();
    }

    static class MessageHandle implements HttpHandler {
        private ObjectMapper objectMapper;
        private CredentialProvider credentialProvider;
        private MicrosoftAppCredentials credentials;

        MessageHandle(CredentialProvider credentialProvider) {
            this.objectMapper = new ObjectMapper()
                    .configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false)
                    .findAndRegisterModules();
            this.credentialProvider = credentialProvider;
            this.credentials = new MicrosoftAppCredentials(appId, appPassword);
        }

        public void handle(HttpExchange httpExchange) throws IOException {
            if (httpExchange.getRequestMethod().equalsIgnoreCase("POST")) {
                Activity activity = getActivity(httpExchange);
                String authHeader = httpExchange.getRequestHeaders().getFirst("Authorization");
                try {
                    JwtTokenValidation.assertValidActivity(activity, authHeader, credentialProvider);

                    // send ack to user activity
                    httpExchange.sendResponseHeaders(202, 0);
                    httpExchange.getResponseBody().close();

                    if (activity.type().equals(ActivityTypes.MESSAGE)) {
                        // reply activity with the same text
                        ConnectorClientImpl connector = new ConnectorClientImpl(activity.serviceUrl(), this.credentials);
                        ResourceResponse response = connector.conversations().sendToConversation(activity.conversation().id(),
                                new Activity()
                                        .withType(ActivityTypes.MESSAGE)
                                        .withText("Echo: " + activity.text())
                                        .withRecipient(activity.from())
                                        .withFrom(activity.recipient())
                        );
                    }
                } catch (AuthenticationException ex) {
                    httpExchange.sendResponseHeaders(401, 0);
                    httpExchange.getResponseBody().close();
                    LOGGER.log(Level.WARNING, "Auth failed!", ex);
                } catch (Exception ex) {
                    LOGGER.log(Level.WARNING, "Execution failed", ex);
                }
            }
        }

        private String getRequestBody(HttpExchange httpExchange) throws IOException {
            StringBuilder buffer = new StringBuilder();
            InputStream stream = httpExchange.getRequestBody();
            int rByte;
            while ((rByte = stream.read()) != -1) {
                buffer.append((char)rByte);
            }
            stream.close();
            if (buffer.length() > 0) {
                return URLDecoder.decode(buffer.toString(), "UTF-8");
            }
            return "";
        }

        private Activity getActivity(HttpExchange httpExchange) {
            try {
                String body = getRequestBody(httpExchange);
                LOGGER.log(Level.INFO, body);
                return objectMapper.readValue(body, Activity.class);
            } catch (Exception ex) {
                LOGGER.log(Level.WARNING, "Failed to get activity", ex);
                return null;
            }

        }
    }
}
```

Si vous utilisez Maven, vous pouvez copier le fichier pom.xml du dossier d’exemples dans ce référentiel. Une fois que vous avez lancé votre fichier exécutable, démarrez le Bot Framework Emulator.

### <a name="start-the-emulator-and-connect-your-bot"></a>Démarrer l’émulateur et connecter votre bot

À ce stade, votre bot s’exécute localement.
Démarrez à présent l’émulateur, puis connectez-vous à votre bot dans l’émulateur :

1. Cliquez sur le lien **create a new bot configuration** (créer une configuration de bot) sous l’onglet « Welcome » (Bienvenue) de l’émulateur. 

2. Renseignez le champ **Bot name** (Nom du bot) et indiquez le chemin d’accès au répertoire de votre code de bot. Le fichier de configuration du bot y sera enregistré.

3. Tapez `http://localhost:port-number/api/messages` dans le champ **URL du point de terminaison**, où *port-number* correspond au numéro de port affiché dans le navigateur où s’exécute votre application.

4. Cliquez sur **Se connecter** pour vous connecter à votre bot. Vous n’avez pas besoin de spécifier les valeurs **Microsoft App ID** (ID d’application Microsoft) et **Microsoft App Password** (Mot de passe d’application Microsoft). Vous pouvez laisser ces champs vides pour l’instant. Vous obtiendrez ces informations ultérieurement, lors de l’inscription du bot.

### <a name="interact-with-your-bot"></a>Interagir avec votre bot
Envoyez le mot « Bonjour » à votre bot. Il vous renverra le même message en retour.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Bot : concepts de base](../v4sdk/bot-builder-basics.md)
