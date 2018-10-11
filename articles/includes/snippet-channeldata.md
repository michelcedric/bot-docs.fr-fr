# <a name="implement-channel-specific-functionality"></a>Implémenter une fonctionnalité spécifique du canal

Certains canaux fournissent des fonctionnalités qui ne peuvent pas être implémentées uniquement à l’aide de SMS et de pièces jointes. Pour implémenter une fonctionnalité spécifique à un canal, vous pouvez transmettre des métadonnées natives à un canal dans la propriété des _données du canal_ de l’objet d’activité. Par exemple, votre bot peut utiliser la propriété des données du canal pour indiquer à Telegram d’envoyer un autocollant, ou pour demander à Office 365 d’envoyer un e-mail.

Cet article explique comment utiliser une propriété des données du canal de l’activité de message pour implémenter cette fonctionnalité propre au canal :

| Canal | Fonctionnalités |
|----|----|
| Email | Envoyer et recevoir un e-mail qui contient un corps, un objet et des métadonnées d’importance |
| Slack | Envoyer des messages Slack de fidélité |
| Facebook | Envoyer des notifications Facebook en mode natif |
| Telegram | Effectuer des actions spécifiques à Telegram, comme le partage d’un mémo vocal ou d’un autocollant |
| Kik | Envoyer et recevoir des messages Kik natifs |

> [!NOTE]
> La valeur de la propriété des données du canal d’un objet d’activité est un objet JSON.
> Par conséquent, les exemples de cet article montrent le format attendu de la propriété JSON `channelData` dans divers scénarios.
> Pour créer un objet JSON à l’aide de .NET, utilisez la classe (.NET) `JObject`.

## <a name="create-a-custom-email-message"></a>Créer un e-mail personnalisé

Pour créer un e-mail, définissez la propriété des données du canal de l’objet d’activité sur un objet JSON contenant ces propriétés :

| Propriété | Description |
|----|----|
| bccRecipients | Chaîne d’adresses e-mail séparées par un point-virgule (;) à ajouter au champ Cci (copie carbone invisible) du message. |
| ccRecipients | Chaîne d’adresses e-mail séparées par un point-virgule (;) à ajouter au champ Cc (copie carbone) du message. |
| htmlBody | Document HTML qui spécifie le corps de l’e-mail. Consultez la documentation du canal pour obtenir plus d’informations sur les éléments et attributs HTML pris en charge. |
| importance | Niveau d’importance de l’e-mail. Les valeurs valides sont **high** (haute), **normal** et **low** (faible). La valeur par défaut est **normal**. |
| subject | Objet de l’e-mail. Consultez la documentation du canal pour obtenir plus d’informations sur la configuration nécessaire du champ. |
| toRecipients | Chaîne d’adresses e-mail séparées par un point-virgule (;) à ajouter au champ À du message. |

> [!NOTE]
> Les messages que votre bot reçoit de la part des utilisateurs via le canal E-mail peuvent contenir une propriété des données du canal qui est renseignée par un objet JSON similaire à celui ci-dessus.

Cet extrait de code montre un exemple de la propriété `channelData` d’un e-mail personnalisé.

```json
"channelData": {
    "type": "message",
    "locale": "en-Us",
    "channelID": "email",
    "from": { "id": "mybot@mydomain.com", "name": "My bot"},
    "recipient": { "id": "joe@otherdomain.com", "name": "Joe Doe"},
    "conversation": { "id": "123123123123", "topic": "awesome chat" },
    "channelData":
    {
        "htmlBody": "<html><body style = /"font-family: Calibri; font-size: 11pt;/" >This is more than awesome.</body></html>",
        "subject": "Super awesome message subject",
        "importance": "high",
        "ccRecipients": "Yasemin@adatum.com;Temel@adventure-works.com"
    }
}
```

## <a name="create-a-full-fidelity-slack-message"></a>Créer un message Slack de fidélité optimale

Pour créer un message Slack haute fidélité, définissez la propriété des données du canal de l’objet d’activité sur un objet JSON qui spécifie <a href="https://api.slack.com/docs/messages" target="_blank">Messages Slack</a>, <a href="https://api.slack.com/docs/message-attachments" target="_blank">Pièces jointes Slack</a> et/ou <a href="https://api.slack.com/docs/message-buttons" target="_blank">Boutons Slack</a>.

> [!NOTE]
> Pour prendre en charge des boutons dans les messages Slack, vous devez activer **Interactive Messages** (Messages interactifs) lorsque vous [connectez votre bot](../bot-service-manage-channels.md) au canal Slack.

Cet extrait de code montre un exemple de la propriété `channelData` pour un message Slack personnalisé.

```json
"channelData": {
   "text": "Now back in stock! :tada:",
   "attachments": [
        {
            "title": "The Further Adventures of Slackbot",
            "author_name": "Stanford S. Strickland",
            "author_icon": "https://api.slack.com/img/api/homepage_custom_integrations-2x.png",
            "image_url": "http://i.imgur.com/OJkaVOI.jpg?1"
        },
        {
            "fields": [
                {
                    "title": "Volume",
                    "value": "1",
                    "short": true
                },
                {
                    "title": "Issue",
                    "value": "3",
                    "short": true
                }
            ]
        },
        {
            "title": "Synopsis",
            "text": "After @episod pushed exciting changes to a devious new branch back in Issue 1, Slackbot notifies @don about an unexpected deploy..."
        },
        {
            "fallback": "Would you recommend it to customers?",
            "title": "Would you recommend it to customers?",
            "callback_id": "comic_1234_xyz",
            "color": "#3AA3E3",
            "attachment_type": "default",
            "actions": [
                {
                    "name": "recommend",
                    "text": "Recommend",
                    "type": "button",
                    "value": "recommend"
                },
                {
                    "name": "no",
                    "text": "No",
                    "type": "button",
                    "value": "bad"
                }
            ]
        }
    ]
}
```

Lorsqu’un utilisateur clique sur un bouton dans un message Slack, votre bot reçoit un message de réponse dans lequel la propriété des données du canal est renseignée par un objet JSON `payload`.
L’objet `payload` précise le contenu du message d’origine, il identifie le bouton qui a été cliqué et identifie l’utilisateur qui a cliqué sur le bouton.

Cet extrait de code montre un exemple de la propriété `channelData` dans le message reçu par un bot lorsqu’un utilisateur clique sur un bouton dans le message Slack.

```json
"channelData": {
    "payload": {
        "actions": [
            {
                "name": "recommend",
                "value": "yes"
            }
        ],
        . . .
        "original_message": "{…}",
        "response_url": "https://hooks.slack.com/actions/..."
    }
}
```

Votre bot peut répondre à ce message normalement, ou il peut publier sa réponse directement sur le point de terminaison spécifié par la propriété `response_url` de l’objet `payload`.
Pour plus d’informations sur le moment et la façon de publier une réponse sur `response_url`, consultez <a href="https://api.slack.com/docs/message-buttons" target="_blank">Slack Buttons</a>.

Vous pouvez créer des boutons dynamiques à l’aide du code suivant :
```cs
private async Task DemoButtonsAsync(IDialogContext context)
        {
            var reply = context.MakeMessage();

            string s = @"{
                ""text"": ""Would you like to play a game ? "",
                ""attachments"": [
                    {
                        ""text"": ""Choose a game to play!"",
                        ""fallback"": ""You are unable to choose a game"",
                        ""callback_id"": ""wopr_game"",
                        ""color"": ""#3AA3E3"",
                        ""attachment_type"": ""default"",
                        ""actions"": [
                            {
                                ""name"": ""game"",
                                ""text"": ""Chess"",
                                ""type"": ""button"",
                                ""value"": ""chess""
                            },
                            {
                                ""name"": ""game"",
                                ""text"": ""Falken's Maze"",
                                ""type"": ""button"",
                                ""value"": ""maze""
                            },
                            {
                                ""name"": ""game"",
                                ""text"": ""Thermonuclear War"",
                                ""style"": ""danger"",
                                ""type"": ""button"",
                                ""value"": ""war"",
                                ""confirm"": {
                                    ""title"": ""Are you sure?"",
                                    ""text"": ""Wouldn't you prefer a good game of chess?"",
                                    ""ok_text"": ""Yes"",
                                    ""dismiss_text"": ""No""
                                }
                            }
                        ]
                    }
                ]
            }";

            reply.Text = null;
            reply.ChannelData = JObject.Parse(s);
            await context.PostAsync(reply);
            context.Wait(MessageReceivedAsync);
        }
```

Pour créer des menus interactifs, utilisez le code suivant :
```cs
private async Task DemoMenuAsync(IDialogContext context)
        {
            var reply = context.MakeMessage();

            string s = @"{
                ""text"": ""Would you like to play a game ? "",
                ""response_type"": ""in_channel"",
                ""attachments"": [
                    {
                        ""text"": ""Choose a game to play"",
                        ""fallback"": ""If you could read this message, you'd be choosing something fun to do right now."",
                        ""color"": ""#3AA3E3"",
                        ""attachment_type"": ""default"",
                        ""callback_id"": ""game_selection"",
                        ""actions"": [
                            {
                                ""name"": ""games_list"",
                                ""text"": ""Pick a game..."",
                                ""type"": ""select"",
                                ""options"": [
                                    {
                                        ""text"": ""Hearts"",
                                        ""value"": ""menu_id_hearts""
                                    },
                                    {
                                        ""text"": ""Bridge"",
                                        ""value"": ""menu_id_bridge""
                                    },
                                    {
                                        ""text"": ""Checkers"",
                                        ""value"": ""menu_id_checkers""
                                    },
                                    {
                                        ""text"": ""Chess"",
                                        ""value"": ""menu_id_chess""
                                    },
                                    {
                                        ""text"": ""Poker"",
                                        ""value"": ""menu_id_poker""
                                    },
                                    {
                                        ""text"": ""Falken's Maze"",
                                        ""value"": ""menu_id_maze""
                                    },
                                    {
                                        ""text"": ""Global Thermonuclear War"",
                                        ""value"": ""menu_id_war""
                                    }
                                ]
                            }
                        ]
                    }
                ]
            }";

            reply.Text = null;
            reply.ChannelData = JObject.Parse(s);
            await context.PostAsync(reply);
            context.Wait(MessageReceivedAsync);
        }
```

## <a name="create-a-facebook-notification"></a>Créer une notification Facebook

Pour créer une notification Facebook, définissez la propriété des données du canal de l’objet d’activité sur un objet JSON spécifiant ces propriétés :

| Propriété | Description |
|----|----|
| notification_type | Type de notification (par exemple, **REGULAR**, **SILENT_PUSH**, **NO_PUSH**).
| attachment | Pièce jointe qui spécifie une image, une vidéo ou un autre type multimédia, ou encore une pièce jointe basée sur un modèle, comme un accusé de réception. |

> [!NOTE]
> Pour plus d’informations sur le format et le contenu de la propriété `notification_type` et de la propriété `attachment`, consultez la <a href="https://developers.facebook.com/docs/messenger-platform/send-api-reference#guidelines" target="_blank">documentation de l’API Facebook</a>. 

Cet extrait de code montre un exemple de la propriété `channelData` pour une pièce jointe d’accusé de réception de Facebook.

```json
"channelData": {
    "notification_type": "NO_PUSH",
    "attachment": {
        "type": "template"
        "payload": {
            "template_type": "receipt",
            . . .
        }
    }
}
```

## <a name="create-a-telegram-message"></a>Créer un message Telegram

Pour créer un message qui implémente des actions spécifiques de Telegram, telles que le partage d’un mémo vocal ou d’un autocollant, définissez la propriété des données du canal de l’objet d’activité sur un objet JSON qui spécifie ces propriétés : 

| Propriété | Description |
|----|----|
| method | Méthode de l’API Bot Telegram à appeler. |
| parameters | Paramètres de la méthode spécifiée. |

Les méthodes Telegram prises en charge sont les suivantes : 

- answerInlineQuery
- editMessageCaption
- editMessageReplyMarkup
- editMessageText
- forwardMessage
- kickChatMember
- SendVideo
- sendChatAction
- sendContact
- sendDocument
- sendLocation
- SendMessage
- sendPhoto
- sendSticker
- sendVenue
- sendVideo
- sendVoice
- unbanChateMember

Pour plus d’informations sur ces méthodes Telegram et leurs paramètres, consultez la <a href="https://core.telegram.org/bots/api#available-methods" target="_blank">documentation de l’API Bot Telegram</a>.

> [!NOTE]
> <ul><li>Le paramètre <code>chat_id</code> est commun à toutes les méthodes Telegram. Si vous ne spécifiez pas <code>chat_id</code> en tant que paramètre, l’infrastructure fournit l’ID pour vous.</li>
> <li>Au lieu de passer le contenu de fichier inline, spécifiez le fichier à l’aide d’une URL et d’un type de média, comme indiqué dans l’exemple ci-dessous.</li>
> <li>Dans chaque message que votre bot reçoit du canal Telegram, la propriété <code>ChannelData</code> inclut le message précédemment envoyé par votre bot.</li></ul>

Cet extrait de code montre l’exemple d’une propriété `channelData` qui spécifie une méthode Telegram unique.

```json
"channelData": {
    "method": "sendSticker",
    "parameters": {
        "sticker": {
            "url": "https://domain.com/path/gif",
            "mediaType": "image/gif",
        }
    }
}
```

Cet extrait de code montre l’exemple d’une propriété `channelData` qui spécifie un tableau de méthodes Telegram.

```json
"channelData": [
    {
        "method": "sendSticker",
        "parameters": {
            "sticker": {
                "url": "https://domain.com/path/gif",
                "mediaType": "image/gif",
            }
        }
    },
    {
        "method": "sendMessage",
        "parameters": {
            "text": "<b>This message is HTML formatted.</b>",
            "parse_mode": "HTML"
        }
    }
]
```

## <a name="create-a-native-kik-message"></a>Créer un message Kik natif

Pour créer un message Kik natif, définissez la propriété des données du canal de l’objet d’activité sur un objet JSON spécifiant cette propriété :

| Propriété | Description |
|----|----|
| Cloud vers appareil | Un tableau de messages Kik. Pour obtenir des informations détaillées sur le format des messages Kik, consultez <a href="https://dev.kik.com/#/docs/messaging#message-formats" target="_blank">Formats des messages Kik</a>. |

Cet extrait de code montre un exemple de la propriété `channelData` pour un message Kik natif.

```json
"channelData": {
    "messages": [
        {
            "chatId": "c6dd8165…",
            "type": "link",
            "to": "kikhandle",
            "title": "My Webpage",
            "text": "Some text to display",
            "url": "http://botframework.com",
            "picUrl": "http://lorempixel.com/400/200/",
            "attribution": {
                "name": "My App",
                "iconUrl": "http://lorempixel.com/50/50/"
            },
            "noForward": true,
            "kikJsData": {
                    "key": "value"
                }
        }
    ]
}
```

## <a name="additional-resources"></a>Ressources supplémentaires

- [Entités et types d’activités](../bot-service-activities-entities.md)
- [Schéma d’activité Bot Framework](https://github.com/Microsoft/BotBuilder/blob/hub/specs/botframework-activity/botframework-activity.md)
