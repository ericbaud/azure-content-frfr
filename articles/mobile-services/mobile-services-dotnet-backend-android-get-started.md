
<properties
	pageTitle="Prise en main d'Azure Mobile Services pour les applications Android"
	description="Suivez ce didacticiel pour commencer à utiliser Azure Mobile Services pour le développement Android."
	services="mobile-services"
	documentationCenter="android"
	authors="RickSaling"
	manager="erikre"
	editor=""/>

<tags
	ms.service="mobile-services"
	ms.workload="mobile"
	ms.tgt_pltfrm="mobile-android"
	ms.devlang="java"
	ms.topic="get-started-article"
	ms.date="03/05/2016"
	ms.author="ricksal"/>


# <a name="getting-started"> </a>Prise en main de Mobile Services

[AZURE.INCLUDE [mobile-services-selector-get-started](../../includes/mobile-services-selector-get-started.md)]

&nbsp;

[AZURE.INCLUDE [mobile-service-note-mobile-apps](../../includes/mobile-services-note-mobile-apps.md)]
> Pour la version Mobile Apps équivalente de cette rubrique, consultez l’article [Créer une application Android dans Azure Mobile Apps](../app-service-mobile/app-service-mobile-android-get-started.md).

Ce didacticiel présente l'ajout d'un service backend cloud à une application Android à l'aide d'Azure Mobile Services. Dans ce didacticiel, vous allez créer un service mobile et une simple application _To do list_ qui stocke les données d'application dans le nouveau service mobile. Le service mobile que vous allez créer utilise les langages .NET pris en charge à l'aide de Visual Studio pour la logique métier côté serveur et pour la gestion du service mobile. Pour créer un service mobile vous permettant d'écrire votre logique métier côté serveur en JavaScript, consultez la [version du backend JavaScript](mobile-services-android-get-started.md) de cette rubrique.

Voici une capture d'écran de l'application terminée :

![](./media/mobile-services-dotnet-backend-android-get-started/mobile-quickstart-completed-android.png)

Pour suivre ce didacticiel, vous avez besoin des [Outils de développement Android][Android Studio], qui incluent l'environnement de développement intégré Android Studio et la dernière plateforme Android. Android 4.2 ou une version ultérieure est nécessaire.

Le projet de démarrage rapide téléchargé contient le Kit de développement logiciel (SDK) Mobile Services pour Android.

> [AZURE.IMPORTANT] Pour suivre ce didacticiel, vous avez besoin d'un compte Azure. Si vous n'avez pas de compte, vous pouvez vous inscrire pour une évaluation d'Azure et obtenir jusqu'à 10 services mobiles gratuits que vous pourrez conserver après l'expiration de votre période d'évaluation. Pour plus d'informations, consultez la page [Version d'évaluation gratuite d'Azure](https://azure.microsoft.com/pricing/free-trial/?WT.mc_id=AE564AB28).


## <a name="create-new-service"> </a>Création d'un service mobile

[AZURE.INCLUDE [mobile-services-dotnet-backend-create-new-service](../../includes/mobile-services-dotnet-backend-create-new-service.md)]

## Téléchargement du service mobile sur votre ordinateur local

Maintenant que vous avez créé le service mobile, téléchargez votre projet de service mobile personnalisé afin de l'exécuter sur votre ordinateur local ou sur votre machine virtuelle.

1. Cliquez sur le service mobile que vous venez de créer, puis dans l'onglet de démarrage rapide, cliquez sur **Android** sous **Choisissez une plateforme** et développez **Créer une application Android**.

	![][1]

2. Si vous ne l'avez pas encore fait, téléchargez et installez [Visual Studio Professional 2013](https://go.microsoft.com/fwLink/p/?LinkID=391934) ou une version ultérieure.

3. À l'étape 2, cliquez sur **Télécharger** sous **Télécharger et publier votre service dans le cloud**.

	Cela permet de télécharger le projet Visual Studio qui implémente votre service mobile. Enregistrez le fichier projet compressé sur votre ordinateur local et notez l'emplacement où vous l'avez enregistré.

## Test du service mobile

[AZURE.INCLUDE [mobile-services-dotnet-backend-test-local-service](../../includes/mobile-services-dotnet-backend-test-local-service.md)]

## Publication de votre service mobile

[AZURE.INCLUDE [mobile-services-dotnet-backend-publish-service](../../includes/mobile-services-dotnet-backend-publish-service.md)]

## Création d'une application Android

Dans cette section, vous allez créer une application Android connectée à votre service mobile.

1. Dans le [portail Azure Classic], cliquez sur **Mobile Services**, puis sur le service mobile que vous venez de créer.

2. Dans l'onglet de démarrage rapide, cliquez sur **Android** sous **Choisissez une plateforme** et développez **Créer une application Android**.

	![][2]

3. Si ce n'est pas déjà fait, téléchargez et installez les [outils de développement Android][Android SDK] sur votre ordinateur local ou sur votre machine virtuelle.

4. Sous **Télécharger et exécuter l'application**, cliquez sur **Télécharger**.

  	Cette opération télécharge le projet de votre exemple d'application _To do list_ qui est connectée à votre service mobile. Enregistrez le fichier projet compressé sur votre ordinateur local et notez l'emplacement où vous l'avez enregistré.

## Exécution de votre application Android

[AZURE.INCLUDE [mobile-services-run-your-app](../../includes/mobile-services-android-get-started.md)]

## <a name="next-steps"> </a>Étapes suivantes
Vous avez terminé les étapes de démarrage rapide. Découvrez ensuite comment effectuer d’autres tâches importantes dans Mobile Services :

* [Ajouter les notifications push à votre application] <br/>En savoir plus sur l’envoi d’une notification Push très basique à votre application.

* [Ajouter une fonction d’authentification à votre application] <br/>Découvrez comment limiter l’accès à vos données principales à certains utilisateurs inscrits de votre application.

* [Résolution des problèmes d'un backend .NET Mobile Services ] <br/>Découvrez comment diagnostiquer et résoudre les problèmes qui peuvent se produire avec un backend .NET Mobile Services.

<!-- Anchors. -->
[Getting started with Mobile Services]: #getting-started
[Create a new mobile service]: #create-new-service
[Define the mobile service instance]: #define-mobile-service-instance
[Next Steps]: #next-steps

<!-- Images. -->
[0]: ./media/mobile-services-dotnet-backend-android-get-started/mobile-quickstart-completed-android.png
[1]: ./media/mobile-services-dotnet-backend-android-get-started/mobile-quickstart-steps-vs-AS.png
[2]: ./media/mobile-services-dotnet-backend-android-get-started/mobile-quickstart-steps-android-AS.png


[6]: ./media/mobile-services-dotnet-backend-android-get-started/mobile-portal-quickstart-android.png
[7]: ./media/mobile-services-dotnet-backend-android-get-started/mobile-quickstart-steps-android.png
[8]: ./media/mobile-services-dotnet-backend-android-get-started/mobile-eclipse-quickstart.png

[10]: ./media/mobile-services-dotnet-backend-android-get-started/mobile-quickstart-startup-android.png
[11]: ./media/mobile-services-dotnet-backend-android-get-started/mobile-data-tab.png
[12]: ./media/mobile-services-dotnet-backend-android-get-started/mobile-data-browse.png

[14]: ./media/mobile-services-dotnet-backend-android-get-started/mobile-services-import-android-workspace.png
[15]: ./media/mobile-services-dotnet-backend-android-get-started/mobile-services-import-android-project.png

<!-- URLs. -->
[Get started (Eclipse)]: mobile-services-dotnet-backend-android-get-started-ec.md
[Ajouter les notifications push à votre application]: mobile-services-dotnet-backend-android-get-started-push.md
[Ajouter une fonction d’authentification à votre application]: mobile-services-dotnet-backend-android-get-started-auth.md
[Android SDK]: https://go.microsoft.com/fwLink/p/?LinkID=280125
[Android Studio]: https://developer.android.com/sdk/index.html
[Mobile Services Android SDK]: https://go.microsoft.com/fwLink/p/?LinkID=266533
[Résolution des problèmes d'un backend .NET Mobile Services ]: mobile-services-dotnet-backend-how-to-troubleshoot.md

[portail Azure Classic]: https://manage.windowsazure.com/

<!-----HONumber=AcomDC_0309_2016-->