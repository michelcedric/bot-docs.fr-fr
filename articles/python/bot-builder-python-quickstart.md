---
title: Créer un bot avec le kit SDK Bot Builder pour Python | Microsoft Docs
description: Créez rapidement un bot à l’aide du kit SDK Bot Builder pour Python.
keywords: SDK Bot Builder, créer un bot, démarrage rapide, python, prise en main
author: jonathanfingold
ms.author: jonathanfingold
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 02/21/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: ba16adebe6bbb9b79949cd9842e975e35c3f2aa6
ms.sourcegitcommit: d486dd088b87a44fc8142f7a08877ff993861a42
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/26/2018
ms.locfileid: "42928408"
---
# <a name="create-a-bot-with-the-bot-builder-sdk-for-python"></a>Créer un bot avec le kit SDK Bot Builder pour Python
[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Le kit SDK Bot Builder pour Python est un framework facile à utiliser pour développer des bots. Ce démarrage rapide vous guide tout au long de la création d’un bot et de son test avec Bot Framework Emulator. Le Kit de développement logiciel (SDK) v4 est disponible en préversion. Pour plus d’informations, consultez le [référentiel GitHub](https://github.com/Microsoft/botbuilder-python) Python. 

## <a name="pre-requisite"></a>Conditions préalables
- [Python 3.6.4](https://www.python.org/downloads/) 
- [Bot Framework Emulator](https://github.com/Microsoft/BotFramework-Emulator/releases)

# <a name="create-a-bot"></a>Créer un bot
Dans le fichier main.py, importez les modules standard suivants :

```python
import http.server
import json
import asyncio
```

Et les modules de kit SDK suivants :
```python
from botbuilder.schema import (Activity, ActivityTypes)
from botframework.connector import ConnectorClient
from botframework.connector.auth import (MicrosoftAppCredentials,
                                         JwtTokenValidation, SimpleCredentialProvider)
```
Ensuite, ajoutez le code suivant pour créer le bot à l’aide de ConnectorClient :
```python
APP_ID = ''
APP_PASSWORD = ''


class BotRequestHandler(http.server.BaseHTTPRequestHandler):

    @staticmethod
    def __create_reply_activity(request_activity, text):
        return Activity(
            type=ActivityTypes.message,
            channel_id=request_activity.channel_id,
            conversation=request_activity.conversation,
            recipient=request_activity.from_property,
            from_property=request_activity.recipient,
            text=text,
            service_url=request_activity.service_url)

    def __handle_conversation_update_activity(self, activity):
        self.send_response(202)
        self.end_headers()
        if activity.members_added[0].id != activity.recipient.id:
            credentials = MicrosoftAppCredentials(APP_ID, APP_PASSWORD)
            reply = BotRequestHandler.__create_reply_activity(activity, 'Hello and welcome to the echo bot!')
            connector = ConnectorClient(credentials, base_url=reply.service_url)
            connector.conversations.send_to_conversation(reply.conversation.id, reply)

    def __handle_message_activity(self, activity):
        self.send_response(200)
        self.end_headers()
        credentials = MicrosoftAppCredentials(APP_ID, APP_PASSWORD)
        connector = ConnectorClient(credentials, base_url=activity.service_url)
        reply = BotRequestHandler.__create_reply_activity(activity, 'You said: %s' % activity.text)
        connector.conversations.send_to_conversation(reply.conversation.id, reply)

    def __handle_authentication(self, activity):
        credential_provider = SimpleCredentialProvider(APP_ID, APP_PASSWORD)
        loop = asyncio.new_event_loop()
        try:
            loop.run_until_complete(JwtTokenValidation.assert_valid_activity(
                activity, self.headers.get("Authorization"), credential_provider))
            return True
        except Exception as ex:
            self.send_response(401, ex)
            self.end_headers()
            return False
        finally:
            loop.close()

    def __unhandled_activity(self):
        self.send_response(404)
        self.end_headers()

    def do_POST(self):
        body = self.rfile.read(int(self.headers['Content-Length']))
        data = json.loads(str(body, 'utf-8'))
        activity = Activity.deserialize(data)

        if not self.__handle_authentication(activity):
            return

        if activity.type == ActivityTypes.conversation_update.value:
            self.__handle_conversation_update_activity(activity)
        elif activity.type == ActivityTypes.message.value:
            self.__handle_message_activity(activity)
        else:
            self.__unhandled_activity()


try:
    SERVER = http.server.HTTPServer(('localhost', 9000), BotRequestHandler)
    print('Started http server on localhost:9000')
    SERVER.serve_forever()
except KeyboardInterrupt:
    print('^C received, shutting down server')
    SERVER.socket.close()
```


Enregistrez main.py. Pour exécuter l’exemple sur Windows, entrez la commande ci-après dans votre fenêtre de ligne de commande :
```
python main.py
```
Dans votre terminal local, vous devez voir un message de type « Serveur http démarré sur localhost:9000 »

### <a name="start-the-emulator-and-connect-your-bot"></a>Démarrer l’émulateur et connecter votre bot

Démarrez à présent l’émulateur, puis connectez-vous à votre bot dans l’émulateur :


1. Cliquez sur le lien **create a new bot configuration** (créer une configuration de bot) sous l’onglet « Welcome » (Bienvenue) de l’émulateur. 

2. Renseignez le champ **Bot name** (Nom du bot) et indiquez le chemin d’accès au répertoire de votre code de bot. Le fichier de configuration du bot y sera enregistré.

3. Tapez `http://localhost:port-number/api/messages` dans le champ **URL du point de terminaison**, où *port-number* correspond au numéro de port affiché dans le navigateur où s’exécute votre application.

4. Cliquez sur **Se connecter** pour vous connecter à votre bot. Vous n’avez pas besoin de spécifier les valeurs **Microsoft App ID** (ID d’application Microsoft) et **Microsoft App Password** (Mot de passe d’application Microsoft). Vous pouvez laisser ces champs vides pour l’instant. Vous obtiendrez ces informations ultérieurement, lors de l’inscription du bot.

Tapez **Bonjour** dans l’émulateur et le bot renverra **Vous avez dit « Bonjour »**.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Bot : concepts de base](../v4sdk/bot-builder-basics.md)
