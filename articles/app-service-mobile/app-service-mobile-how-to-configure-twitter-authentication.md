<properties
	pageTitle="Comment configurer l'authentification Twitter pour votre application App Services"
	description="Découvrez comment configurer l'authentification Twitter pour votre application App Services."
	services="app-service\mobile"
	documentationCenter=""
	authors="mattchenderson"
	manager="dwrede"
	editor=""/>

<tags
	ms.service="app-service-mobile"
	ms.workload="mobile"
	ms.tgt_pltfrm="na"
	ms.devlang="multiple"
	ms.topic="article"
	ms.date="02/04/2016"
	ms.author="mahender"/>

# Comment configurer votre application App Service de manière à utiliser la connexion via Twitter

[AZURE.INCLUDE [app-service-mobile-selector-authentication](../../includes/app-service-mobile-selector-authentication.md)]

Cette rubrique montre comment configurer Azure App Service pour utiliser Twitter comme fournisseur d’authentification.

Pour effectuer la procédure décrite dans cette rubrique, vous devez disposer d'un compte Twitter avec une adresse électronique vérifiée. Pour créer un compte Twitter, consultez la page <a href="http://go.microsoft.com/fwlink/p/?LinkID=268287" target="_blank">twitter.com</a>.


> [AZURE.NOTE]
Cette rubrique décrit l'utilisation de la fonctionnalité Authentification/autorisation d'App Service. Elle remplace la passerelle App Service pour la plupart des applications. Les différences qui s'appliquent à l'utilisation de la passerelle sont signalées dans des notes tout au long de cette rubrique.


## <a name="register"> </a>Inscription de votre application avec Twitter


1. Connectez-vous au [portail Azure] et accédez à votre application. Copiez votre **URL**. Vous l’utiliserez pour configurer votre application Twitter.

2. Accédez au site web [Twitter Developers], connectez-vous avec vos identifiants Twitter, puis cliquez sur **Créer une application**.

3. Entrez le **Nom** et une **Description** pour votre nouvelle application. Collez l’**URL** de votre application en guise de **Site web**. Ensuite, pour l’**URL de rappel**, collez l’**URL de rappel** que vous avez copiée précédemment. Il s’agit de la passerelle de votre application Mobile App suivie du chemin _/.auth/login/twitter/callback_. Par exemple, `https://contoso.azurewebsites.net/.auth/login/twitter/callback`. Assurez-vous d'utiliser le schéma HTTPS.

> [AZURE.NOTE]
Si vous utilisez la passerelle App Service au lieu de la fonction d’authentification/autorisation d’App Service, votre URL de redirection utilise à la place l’URL de la passerelle avec le chemin d’accès _/signin-twitter_.

3.  Au bas de la page, lisez et acceptez les termes du contrat. Ensuite, cliquez sur **Créer votre application Twitter**. Cette opération inscrit l'application et affiche les détails de la demande.

4. Cliquez sur l’onglet **Paramètres**, activez l’option **Autoriser la connexion à Twitter via cette application**, puis cliquez sur **Mettre à jour les paramètres**.

5. Sélectionnez l’onglet **Clés et jetons d’accès**. Prenez note des valeurs de **Clé de consommateur (clé API)** et de **Secret de consommateur (clé secrète API)**.

    > [AZURE.NOTE] La clé secrète consommateur est une information d'identification de sécurité importante. Ne partagez pas cette clé secrète avec quiconque et ne la distribuez pas avec votre application.


## <a name="secrets"> </a>Ajout des informations Twitter à votre application

> [AZURE.NOTE]
Si vous utilisez la passerelle App Service, ignorez cette section et accédez à votre passerelle dans le portail. Sélectionnez **Paramètres**, **Identité**, puis **Twitter**. Collez les valeurs obtenues précédemment et cliquez sur **Enregistrer**.


13. Revenez au [portail Azure] et accédez à votre application. Cliquez sur **Paramètres**, puis sur **Authentification / Autorisation**.

14. Si la fonctionnalité Authentification / Autorisation n’est pas activée, positionnez le commutateur sur **On**.

15. Cliquez sur **Twitter**. Collez-y les valeurs d'ID d'application et de clé secrète d'application que vous avez obtenues précédemment. Cliquez ensuite sur **OK**.

    ![][1]

	Par défaut, App Service fournit une authentification, mais ne restreint pas l'accès autorisé à votre contenu et aux API de votre site. Vous devez autoriser les utilisateurs dans votre code d'application.

17. (Facultatif) Pour restreindre l'accès à votre site aux seuls utilisateurs authentifiés par Twitter, définissez **Action à exécuter lorsque la demande n'est pas authentifiée** sur **Twitter**. Cela implique que toutes les demandes soient authentifiées. Toutes les demandes non authentifiées sont redirigées vers Twitter pour être authentifiées.

17. Cliquez sur **Save**.

Vous êtes maintenant prêt à utiliser Twitter pour l'authentification dans votre application.

## <a name="related-content"> </a>Contenu connexe

[AZURE.INCLUDE [app-service-mobile-related-content-get-started-users](../../includes/app-service-mobile-related-content-get-started-users.md)]



<!-- Images. -->

[0]: ./media/app-service-mobile-how-to-configure-twitter-authentication/app-service-twitter-redirect.png
[1]: ./media/app-service-mobile-how-to-configure-twitter-authentication/mobile-app-twitter-settings.png

<!-- URLs. -->

[Twitter Developers]: http://go.microsoft.com/fwlink/p/?LinkId=268300
[portail Azure]: https://portal.azure.com/
[xamarin]: ../app-services-mobile-app-xamarin-ios-get-started-users.md

<!---HONumber=AcomDC_0211_2016-->