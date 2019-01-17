---
title: Connecter un bot au canal Discussion Web | Microsoft Docs
description: Découvrez comment utiliser le contrôle de discussion web dans votre page web pour un bot connecté au canal Discussion Web.
keywords: discussion web, canal de bot, page web, clé secrète, iframe, HTML
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/04/2018
ms.openlocfilehash: 20ee5a5a0849cef91e59aece7a87f8e9ac4e86ec
ms.sourcegitcommit: b94361234816e6b95459f142add936732fc40344
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/15/2019
ms.locfileid: "54317629"
---
# <a name="connect-a-bot-to-web-chat"></a>Connecter un bot au canal Discussion Web

[!INCLUDE [pre-release-label](./includes/pre-release-label.md)]

Lorsque vous [créez un bot](bot-service-quickstart.md) avec le service de bot, le canal Discussion Web est automatiquement configuré pour vous. Le canal Discussion Web inclut le contrôle de discussion web, qui offre aux utilisateurs la possibilité d’interagir avec votre bot directement dans une page web.

![Exemple de discussion web](./media/bot-service-channel-webchat/create-a-bot.png)

Le canal Discussion Web du portail Bot Framework contient tout ce dont vous avez besoin pour incorporer le contrôle de discussion web dans une page web. Pour utiliser le contrôle de discussion web, il vous suffit d’obtenir la clé secrète de votre bot et d’incorporer le contrôle dans une page web.

## <a id="step-1"></a> Obtenir la clé secrète de votre bot

1. Ouvrez votre bot dans le [Portail Azure](http://portal.azure.com) et cliquez sur le panneau **Canaux**.

2. Cliquez sur **Modifier** pour le canal **Web Chat** (Discussion Web).  
![Canal Discussion Web](./media/bot-service-channel-webchat/bot-service-channel-list.png)

3. Sous **Secret keys** (Clés secrètes), cliquez sur **Afficher** pour la première clé.  
![Clé secrète](./media/bot-service-channel-webchat/secret-key.png)

4. Copiez la **clé secrète** et le **code incorporé**.

5. Cliquez sur **Done**.

## <a name="embed-the-web-chat-control-in-your-website"></a>Incorporer le contrôle de discussion web dans votre site web

Vous pouvez incorporer le contrôle de discussion web dans votre site web en utilisant l’une des deux options ci-après.

### <a name="option-1---keep-your-secret-hidden-exchange-your-secret-for-a-token-and-generate-the-embed"></a>Option 1 : conserver votre secret masqué, l’échanger contre un jeton et générer l’incorporation

Utilisez cette option si vous pouvez exécuter une requête de serveur à serveur pour échanger votre secret de discussion web contre un jeton temporaire et si vous souhaitez compliquer la tâche des développeurs souhaitant incorporer votre bot dans leurs sites web. Bien que cette option n’empêche pas totalement les autres développeurs d’incorporer votre bot dans leurs sites web, elle rend la procédure plus compliquée.

Pour échanger votre secret contre un jeton et générer l’incorporation :

1. Émettez une requête **GET** vers `https://webchat.botframework.com/api/tokens` et passez votre secret de discussion web par le biais de l’en-tête `Authorization`. L’en-tête `Authorization` utilise le schéma `BotConnector` et inclut votre secret, comme illustré dans les exemples de requêtes ci-dessous.

2. La réponse à votre requête **GET** contient le jeton (entre guillemets) qui peut être utilisé pour démarrer une conversation en restituant le contrôle de discussion web au sein d’un **iframe**. Un jeton est valide pour une conversation uniquement. Pour démarrer une autre conversation, vous devez générer un nouveau jeton.

3. Dans le **code incorporé** `iframe` que vous avez copié à partir du canal Discussion Web dans le portail Bot Framework (comme décrit à l’étape [Obtenir la clé secrète de votre bot](#step-1) ci-dessus), remplacez le paramètre `s=` par `t=`et remplacez « YOUR_SECRET_HERE » par votre jeton.

> [!NOTE]
> Les jetons sont automatiquement renouvelés avant leur expiration. 

##### <a name="example-request"></a>Exemple de requête

```
requestGET https://webchat.botframework.com/api/tokens
Authorization: BotConnector YOUR_SECRET_HERE
```

##### <a name="example-response"></a>Exemple de réponse 

```response
"IIbSpLnn8sA.dBB.MQBhAFMAZwBXAHoANgBQAGcAZABKAEcAMwB2ADQASABjAFMAegBuAHYANwA.bbguxyOv0gE.cccJjH-TFDs.ruXQyivVZIcgvosGaFs_4jRj1AyPnDt1wk1HMBb5Fuw"
```

##### <a name="example-iframe-using-token"></a>Exemple d’Iframe (avec un jeton)

```html
<iframe src="https://webchat.botframework.com/embed/YOUR_BOT_ID?t=YOUR_TOKEN_HERE"></iframe>
```

##### <a name="example-html-code"></a>Exemple de code HTML
```html
<!DOCTYPE html>
<html>
<body>
  <iframe id="chat" style="width: 400px; height: 400px;" src=''></iframe>
</body>
<script>

    var xhr = new XMLHttpRequest();
    xhr.open('GET', "https://webchat.botframework.com/api/tokens", true);
    xhr.setRequestHeader('Authorization', 'BotConnector ' + 'YOUR SECRET HERE');
    xhr.send();
    xhr.onreadystatechange = processRequest;

    function processRequest(e) {
      if (xhr.readyState == 4  && xhr.status == 200) {
        var response = JSON.parse(xhr.responseText);
        document.getElementById("chat").src="https://webchat.botframework.com/embed/lucas-direct-line?t="+response
      }
    }

  </script>
</html>
```

### <a id="option-2"></a> Option 2 : incorporer le contrôle de discussion web dans votre site web en utilisant le secret

Utilisez cette option si vous souhaitez permettre aux autres développeurs d’incorporer facilement votre bot dans leurs sites Web. 

> [!WARNING]
> Si vous utilisez cette option, les autres développeurs peuvent incorporer votre bot dans leurs sites web en copiant simplement votre code incorporé.

Pour incorporer votre bot dans votre site web en spécifiant le secret dans la balise `iframe` :

1. Copiez le **code incorporé** `iframe` issu du canal Discussion Web, dans le portail Bot Framework (comme décrit à l’étape [Obtenir la clé secrète de votre bot](#step-1) ci-dessus).

2. Dans ce **code incorporé**, remplacez « YOUR_SECRET_HERE » par la valeur de la **clé secrète** que vous avez copiée sur la même page.

##### <a name="example-iframe-using-secret"></a>Exemple d’Iframe (avec un secret)

```html
<iframe src="https://webchat.botframework.com/embed/YOUR_BOT_ID?s=YOUR_SECRET_HERE"></iframe>
```

## <a name="style-the-web-chat-control"></a>Modification du style du contrôle de discussion web

Vous pouvez modifier la taille du contrôle de discussion web à l’aide de l’attribut `style` du `iframe` pour spécifier les valeurs `height` et `width`.

```html
<iframe style="height:480px; width:402px" src="... SEE ABOVE ..."></iframe>
```

![Client de contrôle de conversation](./media/chatwidget-client.png)

## <a name="additional-resources"></a>Ressources supplémentaires

Vous pouvez [télécharger le code source](https://aka.ms/BotFramework-WebChat-V4) du contrôle de discussion web sur GitHub.
