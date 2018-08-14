---
title: Aperçu des fonctionnalités du bot avec l’inspecteur de canaux | Microsoft Docs
description: Découvrez comment se présentent les différentes fonctionnalités du Bot Framework et comment elles fonctionnent sur différents canaux grâce à l’inspecteur de canaux.
keyword: bot, preview features, channel, display, Channel Inspector, Text formatting, markdown, XML
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: def3f811704af2e2a22612ba5755eb5dbd2d8044
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/18/2018
ms.locfileid: "39299172"
---
# <a name="preview-bot-features-with-the-channel-inspector"></a>Aperçu des fonctionnalités du bot avec l’inspecteur de canaux

Le Bot Framework vous permet de créer des bots incluant un large éventail de fonctionnalités : texte, boutons, images, cartes enrichies affichées sous forme de carrousel ou au format liste, etc. Toutefois, au final, chaque canal contrôle la façon dont les fonctionnalités sont affichées par ses clients de messagerie.

Même lorsqu’une fonctionnalité est prise en charge par plusieurs canaux, chaque canal peut afficher la fonctionnalité de façon légèrement différente. Lorsqu’un message contient des fonctionnalités non prises en charge de façon native par un canal, ce dernier peut tenter d’afficher une version réduite du contenu du message, sous forme de texte ou d’image statique, ce qui peut affecter considérablement l’apparence du message sur le client. Il arrive qu’une fonctionnalité ne soit pas du tout prise en charge par un canal. Par exemple, les clients GroupMe ne peuvent pas afficher d’indicateur de saisie.

## <a name="the-channel-inspector"></a>L’inspecteur de canaux

[L’inspecteur de canaux][inspector] a été conçu pour vous montrer comment se présentent les différentes fonctionnalités du Bot Framework et comment elles fonctionnent sur différents canaux. Si vous comprenez comment les fonctionnalités sont affichées par les différents canaux, vous pourrez concevoir votre bot de façon à offrir une expérience utilisateur exceptionnelles sur les canaux de communication concernés. L’inspecteur de canaux offre également un excellent moyen de découvrir les fonctionnalités du Bot Framework et de les explorer visuellement.

> [!NOTE]
> Les cartes enrichies s’imposent de plus en plus comme norme pour l’échange d’informations et assurent un affichage cohérent sur plusieurs canaux. Consultez la documentation sur le [.NET][netcard] ou [Node.js][nodecard] pour plus d’informations sur les cartes enrichies.

## <a name="text-formatting"></a>Mise en forme de texte

La mise en forme de texte peut améliorer l’apparence de vos messages de texte. Au-delà du texte **brut**, votre bot peut envoyer des messages de texte avec une mise en forme **Markdown** ou **xml** aux canaux pouvant la prendre en charge. Les tableaux suivants répertorient certaines des mises en forme les plus couramment utilisées avec **Markdown** et **xml**. Il se peut qu’un canal prenne en charge plus de mises en formes (ou moins) que celles indiquées ici. Vous pouvez utiliser [l’inspecteur de canaux][inspector] pour vérifier si une fonctionnalité que vous souhaitez utiliser est prise en charge par le canal ciblé.

### <a name="markdown-text-format"></a>Mise en forme de texte Markdown

Ces styles peuvent être pris en charge lorsque `textFormat` a la valeur **markdown** :  

| Style | Markdown | Exemples |
| ---- | ---- | ---- |
| gras | \*\*text\*\* | **text** |
| italique | \*text\* | *text* |
| en-tête (1-5) | # H1 | # H1 |
| barré | \~\~text\~\~ | ~~text~~ |
| ligne horizontale | ---- | ---- |
| liste non triée | \* texte |  <ul><li>texte</li></ul> |
| liste triée | 1. texte | 1. texte |
| texte déjà mis en forme | \`text\` | `text`  |
| blockquote | \> texte | <blockquote>texte</blockquote> |
| lien hypertexte | \[bing](http://www.bing.com) | [bing](http://www.bing.com) |
| lien vers image| !\[canard](http://aka.ms/Fo983c) | ![canard](http://aka.ms/Fo983c) |

> [!NOTE]
> Avec **Markdown**, les balises HTML ne sont pas prises en charge par les canaux Discussion Web du Bot Framework. Si vous devez utiliser des balises HTML dans votre **Markdown**, vous pouvez les afficher dans un canal [Direct Line](~/bot-service-channel-connect-directline.md) qui les prend en charge. Vous pouvez également utiliser les balises HTML ci-dessous en affectant à `textFormat` la valeur **xml** et connecter votre bot au canal Skype.

### <a name="xml-text-format"></a>Mise en forme de texte XML

Ces styles peuvent être pris en charge lorsque `textFormat` a la valeur **xml** :

|      Style      |                       XML                       |                   Exemples                   |
|-----------------|-------------------------------------------------|---------------------------------------------|
|      gras       |                 \<b\>text\</b\>                 |            <strong>text</strong>            |
|     italique      |                 \<i\>texte\</i\>                 |                <em>text</em>                |
|    souligné    |                 \<u\>texte\</u\>                 |                 <u>text</u>                 |
|  barré  |                 \<s\>texte\</s\>                 |                 <s>text</s>                 |
|    lien hypertexte    |   \<a href="http://www.bing.com"\>bing\</a\>    |   <a href="http://www.bing.com">bing</a>    |
|    paragraphe    |                 \<p\>text\</p\>                 |                 <p>texte</p>                 |
|   saut de ligne    |                     \<br/\>                     |             ligne 1 <br/>ligne 2              |
| ligne horizontale |                     \<hr/\>                     |                    <hr/>                    |
|  en-tête (1-4)   |                \<h1\>texte\</h1\>                |             <h1>Titre 1</h1>              |
|      code       |           \<code\>bloc de code \</code\>           |           <code>code block</code>           |
|      image      | \<img src="<http://aka.ms/Fo983c>" alt="Duck"\> | <img src="http://aka.ms/Fo983c" alt="Duck"> |

> [!NOTE]
> Le `textFormat` **xml** est pris en charge uniquement par le canal Skype.

## <a name="preview-features-across-various-channels"></a>Aperçu des fonctionnalités sur différents canaux

Pour voir comment un canal affiche une fonctionnalité particulière, accédez à [l’inspecteur de canaux][inspector], puis sélectionnez le canal dans la liste **Canal** et la fonctionnalité dans la liste **Fonctionnalité**. Par exemple, pour voir comment Skype affiche une bannière, définissez **Canal** sur *Skype* et **Fonctionnalité** sur *HeroCard*.

![Inspecteur de canaux avec le canal Skype et la fonctionnalité HeroCard](~/media/bot-service-channel-inspector.png)

L’inspecteur de canaux affiche un aperçu de la fonctionnalité sélectionnée, telle qu’elle sera affichée par le canal spécifié. La section **Notes** fournit des informations importantes sur les limitations de message et/ou les modifications de l’affichage. Par exemple, certains types de cartes enrichies prennent en charge une image uniquement, et certaines fonctionnalités peuvent être affichées sous forme de version réduite sur certains canaux.

> [!NOTE]
> Actuellement, l’inspecteur de canaux prend en charge les canaux suivants :
> * Cortana
> * Email
> * Facebook
> * GroupMe
> * Kik
> * Skype
> * Skype Entreprise
> * Slack
> * sms
> * Microsoft Teams
> * Telegram
> * WeChat
> * Discussion Web
>
> D’autres canaux peuvent être ajoutés à l’avenir.

## <a name="features-that-can-be-previewed"></a>Fonctionnalités pour lesquelles un aperçu est disponible

Actuellement, l’inspecteur de canal peut offrir un aperçu des fonctionnalités suivantes.

|Fonctionnalité | Description|
| --- | ----|
| Cartes adaptatives | Une carte pouvant inclure n’importe quelle combinaison de texte, données vocales, images, boutons et champs d’entrée. |
| Boutons| Boutons sur lesquels l’utilisateur peut cliquer. Les boutons s’affichent sur le canevas de conversation avec le message auquel ils appartiennent. |
| Carrousel| Une liste de cartes horizontale compacte, que l’utilisateur peut faire défiler. Pour une disposition verticale, utilisez le format Liste.|
| ChannelData| Une méthode pour passer des métadonnées pour accéder à une fonctionnalité spécifique à un canal, au-delà des cartes, textes et pièces jointes.|
| Codesample| Un bloc de texte sur plusieurs lignes, déjà mis en forme, conçu pour afficher du code informatique.|
| DirectMessage| Un message envoyé à un seul membre d’une conversation de groupe.
| Document| Pour envoyer et recevoir des pièces jointes non multimédias. |
| Emoji| Pour afficher un emoji pris en charge.
| GroupChat| Le bot envoie des messages pour participer aux conversations de groupe. |
| HeroCard| Une pièce jointe mise en forme contenant généralement une grande image unique, une action par appui et des boutons (facultatifs), ainsi qu’un contenu de texte descriptif. |
| Images| Pour afficher les images jointes. |
| Disposition en format liste| Une liste des cartes verticale. Pour une disposition horizontale que l’utilisateur peut faire défiler, utilisez la fonctionnalité Carrousel.|
| Lieu| Pour partager l’emplacement physique de l’utilisateur avec le bot. |
| Markdown| Pour afficher le texte mis en forme avec Markdown.|
| Members| Pour partager la liste des membres de la conversation avec le bot lors d’une conversation de groupe. |
| Carte de reçu| Pour afficher un reçu pour l’utilisateur. |
| Carte de connexion| Pour demander à l’utilisateur d’entrer ses informations d’authentification.|
| Actions suggérées | Actions présentées sous forme de boutons, sur lesquels l’utilisateur peut appuyer pour fournir une entrée. |
| Carte de miniature| Une pièce jointe mise en forme contenant une petite image unique (miniature), une action par appui et des boutons (facultatifs), ainsi qu’un contenu de texte descriptif. |
| Saisie| Pour afficher un indicateur de saisie. Cette fonctionnalité est utile pour informer l’utilisateur que le bot est toujours opérationnel, mais qu’il exécute certaines actions en arrière-plan.|
| Vidéo| Pour afficher les pièces jointes vidéo et les contrôles de lecture.|

## <a name="additional-resources"></a>Ressources supplémentaires

* [Channel Inspector][inspector] (Inspecteur de canaux)
* Cartes enrichies dans [Node.js][nodecard] et [.NET][netcard]
* Pièces jointes multimédias dans [Node.js][nodemedia] et [.NET][netmedia]
* Actions suggérées dans [Node.js][nodebutton] et [.NET][netbutton]

[inspector]: https://docs.botframework.com/en-us/channel-inspector/channels/Skype/

[syntax]: https://daringfireball.net/projects/markdown/syntax

[netcard]: ~/dotnet/bot-builder-dotnet-add-rich-card-attachments.md
[nodecard]: ~/nodejs/bot-builder-nodejs-send-rich-cards.md

[netmedia]: ~/dotnet/bot-builder-dotnet-add-media-attachments.md
[nodemedia]: ~/nodejs/bot-builder-nodejs-send-receive-attachments.md

[netbutton]: ~/dotnet/bot-builder-dotnet-add-suggested-actions.md
[nodebutton]: ~/nodejs/bot-builder-nodejs-send-suggested-actions.md
