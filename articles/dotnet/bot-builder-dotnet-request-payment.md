---
title: Demander un paiement | Microsoft Docs
description: Découvrez comment envoyer une demande de paiement à l’aide du Kit SDK Bot Builder pour .NET.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/17
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 29359a2a0489863679158d87cdc4af2f3b52e232
ms.sourcegitcommit: 2dc75701b169d822c9499e393439161bc87639d2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/24/2018
ms.locfileid: "42904243"
---
# <a name="request-payment"></a>Demander un paiement

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-request-payment.md)
> - [Node.JS](../nodejs/bot-builder-nodejs-request-payment.md)

Si votre bot permet aux utilisateurs d’acheter des articles, il peut demander le paiement en incluant un type spécial de bouton dans une [carte enrichie](bot-builder-dotnet-add-rich-card-attachments.md). Cet article explique comment envoyer une demande de paiement à l’aide du Kit SDK Bot Builder pour .NET.

## <a name="prerequisites"></a>Prérequis

Avant de pouvoir envoyer une demande de paiement à l’aide du Kit SDK Bot Builder pour .NET, vous devez effectuer ces tâches préalables.

### <a name="update-webconfig"></a>Mettre à jour Web.config

Mettez à jour le fichier **Web.config** de votre bot pour définir `MicrosoftAppId` et `MicrosoftAppPassword` selon les valeurs d’ID et de mot de passe de l’application qui ont été générées pour votre bot lors du processus d’[inscription](~/bot-service-quickstart-registration.md). 

> [!NOTE]
> Pour trouver les paramètres **AppID** et **AppPassword** de votre bot, consultez la rubrique [MicrosoftAppID and MicrosoftAppPassword](~/bot-service-manage-overview.md#microsoftappid-and-microsoftapppassword) (MicrosoftAppID et MicrosoftAppPassword).

### <a name="create-and-configure-merchant-account"></a>Créer et configurer le compte du vendeur

1. <a href="https://dashboard.stripe.com/register" target="_blank">Créez et activez un compte Stripe si vous n’en avez pas encore un.</a>

2. <a href="https://seller.microsoft.com/en-us/dashboard/registration/seller/?accountprogram=botframework" target="_blank">Connectez-vous au Centre des vendeurs avec votre compte Microsoft.</a>

3. Dans le Centre des vendeurs, connectez votre compte avec Stripe.

4. Dans le Centre des vendeurs, accédez au tableau de bord et copiez la valeur **MerchantID**.

5. Mettez à jour le fichier **Web.config** de votre bot pour définir `MerchantId` à l’aide de la valeur que vous avez copiée à partir du tableau de bord du Centre des vendeurs. 

[!INCLUDE [Payment process overview](../includes/snippet-payment-process-overview.md)]

## <a name="payment-bot-sample"></a>Exemple de bot de paiement

Le modèle <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/sample-payments" target="_blank">Bot de paiement</a> fournit un exemple de bot qui envoie une demande de paiement à l’aide de .NET. Pour voir cet exemple de bot en action, vous pouvez <a href="https://webchat.botframework.com/embed/paymentsample?s=d39Bk7JOMzQ.cwA.Rig.dumLki9bs3uqfWFMjXPn5PFnQVmT2VAVR1Zl1iPi07k" target="_blank">le tester dans un chat web</a>, l’<a href="https://join.skype.com/bot/9fbc0f17-43eb-40fe-bf3b-af151e6ce45e" target="_blank">ajouter comme un contact Skype</a> ou télécharger l’exemple de bot de paiement et l’exécuter localement à l’aide de l’émulateur Bot Framework. 

> [!NOTE]
> Pour terminer le processus de paiement de bout en bout à l’aide de l’exemple **Bot de paiement** dans un chat web ou sur Skype, vous devez spécifier une carte de crédit valide ou une carte de débit dans votre compte Microsoft (par exemple, une carte valide émise par un établissement bancaire aux États-Unis). Votre carte ne sera pas débitée et le cryptogramme visuel de la carte n’est pas vérifié car l’exemple **Bot de paiement** s’exécute en mode test (par exemple, `LiveMode` a la valeur `false` dans **Web.config**).

Les sections suivantes de cet article décrivent les trois parties du processus de paiement, dans le contexte de l’exemple **Bot de paiement**.

## <a id="request-payment"></a> Demande de paiement

Votre bot peut demander le paiement d’un utilisateur en envoyant un message contenant une [pièce jointe de carte riche](bot-builder-dotnet-add-rich-card-attachments.md) avec un bouton qui spécifie une valeur `CardAction.Type` de « paiement ». Cet extrait de code de l’exemple **Bot de paiement** crée un message contenant une carte de bannière avec un bouton **Acheter** que l’utilisateur peut activer (en cliquant ou en appuyant dessus) pour lancer le processus de paiement. 

[!code-csharp[Request payment](../includes/code/dotnet-request-payment.cs#requestPayment)]

Dans cet exemple, le type du bouton est spécifié comme `PaymentRequest.PaymentActionType`, que la bibliothèque Bot Builder définit que « paiement ». La valeur du bouton est fournie par la méthode `BuildPaymentRequest`, qui retourne un objet `PaymentRequest` contenant des informations sur les modes de paiement pris en charge, les détails et les options. Pour plus d’informations sur les détails de l’implémentation, consultez **Dialogs/RootDialog.cs** dans l’exemple <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/sample-payments" target="_blank">Bot de paiement</a>.

Cette capture d’écran montre la carte de bannière (avec le bouton **Acheter**) générée par l’extrait de code ci-dessus. 
 
![Exemple de bot de paiement](../media/payments-bot-buy.png) 

> [!IMPORTANT]
> Tout utilisateur disposant d’un accès au bouton **Acheter** peut l’utiliser pour lancer le processus de paiement. Dans le contexte d’une conversation de groupe, il n’est pas possible de désigner un bouton dont seul un utilisateur spécifique pourra se servir. 

## <a id="user-experience"></a> Expérience utilisateur

Lorsqu’un utilisateur clique sur le bouton **Acheter**, il est dirigé vers une interface web de paiement afin d’y fournir les informations requises concernant le paiement, la livraison et les coordonnées via son compte Microsoft. 

![Paiement Microsoft](../media/microsoft-payment.png)

### <a name="http-callbacks"></a>Rappels HTTP

Des rappels HTTP seront envoyés à votre bot pour indiquer qu’il doit effectuer certaines opérations. Chaque rappel sera une [activité](bot-builder-dotnet-activities.md) contenant ces valeurs de propriété : 

| Propriété | Valeur |
|----|----|
| `Activity.Type` | invoke | 
| `Activity.Name` | Indique le type d’opération que le bot doit effectuer (par exemple, mettre à jour l’adresse de livraison, mettre à jour l’option de livraison ou finaliser le paiement). | 
| `Activity.Value` | Charge utile de la demande au format JSON. | 
| `Activity.RelatesTo` |  Décrit le canal et l’utilisateur qui sont associés à la demande de paiement. | 

> [!NOTE]
> `invoke` est un type spécial d’activité, réservé pour une utilisation par Microsoft Bot Framework. L’expéditeur d’une activité `invoke` s’attend à ce que votre bot accuse réception du rappel en envoyant une réponse HTTP.

## <a id="process-callbacks"></a> Traitement des rappels

[!INCLUDE [Process callbacks overview](../includes/snippet-payment-process-callbacks-overview.md)]

### <a name="shipping-address-update-and-shipping-option-update-callbacks"></a>Rappels de mise à jour de l’adresse de livraison et de mise à jour de l’option de livraison

[!INCLUDE [Process shipping address and shipping option callbacks](../includes/snippet-payment-process-callbacks-1.md)]

### <a name="payment-complete-callbacks"></a>Rappels de finalisation de paiement

Lors de la réception d’un rappel de finalisation de paiement, votre bot obtient une copie de la demande de paiement initiale, non modifiée, ainsi que les objets de la réponse de paiement dans la propriété `Activity.Value`. L’objet de la réponse de paiement contiendra les sélections finales effectuées par le client ainsi qu’un jeton de paiement. Votre bot devrait pouvoir recalculer la demande de paiement finale en fonction de la demande de paiement initiale et des sélections finales du client. En supposant que les sélections du client sont définies comme étant valides, le bot doit vérifier le montant et la monnaie locale dans l’en-tête du jeton de paiement pour s’assurer que ces valeurs correspondent à la demande de paiement finale.  Si le bot décide de facturer le client, il doit facturer uniquement le montant figurant dans l’en-tête du jeton de paiement, car il s’agit du prix confirmé par le client. En cas d’incohérence entre les valeurs attendues par le bot et les valeurs qu’il a reçues dans le rappel de finalisation du paiement, il peut annuler la demande de paiement en envoyant le code d’état HTTP `200 OK` et en affichant la valeur `failure` dans le champ de résultat.   

En plus de vérifier les détails du paiement, le bot doit également vérifier que la commande peut être honorée avant de lancer le traitement du paiement. Par exemple, il peut vérifier si les articles achetés sont toujours en stock. Si les valeurs sont correctes et que votre processeur de paiement a facturé avec succès le jeton de paiement, le bot doit répondre avec le code d’état HTTP `200 OK` en affichant la valeur `success` dans le champ de résultat, afin que l’interface web de paiement affiche la confirmation du paiement. Le jeton de paiement reçu par le bot ne peut être utilisé qu’une seule fois par le vendeur qui l’a demandé, et il doit être soumis à Stripe (le seul processeur de paiement actuellement pris en charge par Bot Framework). L’envoi d’un code d’état HTTP compris entre `400` et `500` entraînera une erreur générique pour le client.

La méthode `OnInvoke` dans l’exemple **Bot de paiement** traite les rappels reçus par le bot. 

[!code-csharp[Request payment](../includes/code/dotnet-request-payment.cs#processCallback)]

Dans cet exemple, le bot examine la propriété `Name` de l’activité entrante pour identifier le type d’opération à effectuer, puis appelle la méthode appropriée pour traiter le rappel. Pour plus d’informations sur les détails de l’implémentation, consultez **Controllers/MessagesControllers.cs** dans l’exemple <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/sample-payments" target="_blank">Bot de paiement</a>.

## <a name="testing-a-payment-bot"></a>Test d’un bot de paiement

[!INCLUDE [Test a payment bot](../includes/snippet-payment-test-bot.md)]

Dans l’exemple <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/sample-payments" target="_blank">Bot de paiement</a>, le paramètre de configuration `LiveMode` du fichier **Web.config** détermine si les rappels de finalisation du paiement contiendront des jetons de paiement émulés ou des jetons de paiement réels. Si le paramètre `LiveMode` est défini sur `false`, un en-tête est ajouté à la demande de paiement sortante du bot pour indiquer que le bot est en mode test, et le rappel de finalisation du paiement contiendra un jeton de paiement émulé qui ne peut pas être facturé. Si le paramètre `LiveMode` est défini sur `true`, l’en-tête qui indique que le bot est en mode test est omis de la demande de paiement sortante du bot et le rappel de finalisation du paiement contient un jeton de paiement effectif que le bot soumet à Stripe pour traiter le paiement. Il s’agit d’une vraie transaction qui entraîne des frais pour l’instrument de paiement spécifié. 

## <a name="additional-resources"></a>Ressources supplémentaires

- <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/sample-payments" target="_blank">Exemple de bot de paiement</a>
- [Vue d’ensemble des activités](bot-builder-dotnet-activities.md)
- [Ajouter des cartes détaillées aux messages](bot-builder-dotnet-add-rich-card-attachments.md)
- <a href="http://www.w3.org/Payments/" target="_blank">Paiements Web sur W3C</a> 
- <a href="/dotnet/api/?view=botbuilder-3.11.0" target="_blank">Référence sur le Kit SDK Bot Builder pour .NET</a>
