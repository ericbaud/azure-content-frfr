<properties
	pageTitle="Comment utiliser le plug-in Apache Cordova pour Azure Mobile Apps"
	description="Comment utiliser le plug-in Apache Cordova pour Azure Mobile Apps"
	services="app-service\mobile"
	documentationCenter="javascript"
	authors="adrianhall"
	manager="dwrede"
	editor=""/>

<tags
	ms.service="app-service-mobile"
	ms.workload="mobile"
	ms.tgt_pltfrm="mobile-html"
	ms.devlang="javascript"
	ms.topic="article"
	ms.date="03/07/2016"
	ms.author="adrianha"/>

# Comment utiliser la bibliothèque cliente Apache Cordova pour Azure Mobile Apps

[AZURE.INCLUDE [app-service-mobile-selector-client-library](../../includes/app-service-mobile-selector-client-library.md)]

Ce guide indique le déroulement de scénarios courants dans le cadre de l’utilisation du dernier [plug-in Apache Cordova pour Azure Mobile Apps]. Si vous ne connaissez pas Azure Mobile Apps, consultez d’abord la section [Démarrage rapide d’Azure Mobile Apps] pour créer un serveur principal, créer une table et télécharger un projet Apache Cordova prédéfini. Dans ce guide, nous nous concentrons sur le plug-in Apache Cordova côté client.

##<a name="Setup"></a>Configuration et conditions préalables

Ce guide part du principe que vous avez créé un serveur principal avec une table. Ce guide suppose que la table a le même schéma que les tables dans ces didacticiels. Ce guide suppose également que vous avez ajouté le plug-in Apache Cordova à votre code. Si ce n’est pas le cas, vous pouvez l’ajouter à votre projet depuis la ligne de commande :

```
cordova plugin add cordova-plugin-ms-azure-mobile-apps
```

Pour plus d’informations sur la création de [votre première application Apache Cordova], consultez la documentation officielle.

[AZURE.INCLUDE [app-service-mobile-html-js-library.md](../../includes/app-service-mobile-html-js-library.md)]

##<a name="auth"></a>Procédure : authentification des utilisateurs

Azure App Service prend en charge l’authentification et l’autorisation des utilisateurs d’applications par le biais de divers fournisseurs d’identité externes : Facebook, Google, compte Microsoft et Twitter. Vous pouvez définir des autorisations sur les tables pour limiter l'accès à certaines opérations aux seuls utilisateurs authentifiés. Vous pouvez également utiliser l'identité des utilisateurs authentifiés pour implémenter des règles d'autorisation dans les scripts serveur. Pour plus d'informations, consultez la page [Prise en main de l'authentification].

Quand vous utilisez l’authentification dans une application Apache Cordova, les plug-ins Cordova suivants doivent être réunis :

* [cordova-plugin-device]
* [cordova-plugin-inappbrowser]

Deux flux d’authentification sont pris en charge : un flux serveur et un flux client. Le flux serveur fournit l'authentification la plus simple, car il repose sur l'interface d'authentification Web du fournisseur. Le flux client permet une intégration approfondie avec les fonctionnalités propres aux appareils, telles que l'authentification unique, car il repose sur des Kits de développement logiciel (SDK) propres aux appareils et aux fournisseurs.

[AZURE.INCLUDE [app-service-mobile-html-js-auth-library.md](../../includes/app-service-mobile-html-js-auth-library.md)]

###<a name="configure-external-redirect-urls"></a>Configurer votre Mobile App Service pour les URL de redirection externes

Plusieurs types d’applications Apache Cordova utilisent une fonctionnalité de bouclage pour gérer les flux d’interface utilisateur OAuth. Cela pose des problèmes, car le service d’authentification sait uniquement comment utiliser votre service par défaut. L’utilisation de l’émulateur Ripple, l’exécution de votre service localement ou dans un autre Azure App Service avec une redirection vers Azure App Service pour l’authentification, ou Live Reload avec Ionic, en sont des exemples. Suivez ces instructions pour ajouter vos paramètres régionaux à la configuration :

1. Connectez-vous au [portail Azure].
2. Sélectionnez **Toutes les ressources** ou **App Services**, puis cliquez sur le nom de votre application mobile.
3. Cliquez sur **Outils**.
4. Cliquez sur **Explorateur de ressources** dans le menu OBSERVER, puis cliquez sur **Atteindre**. Une nouvelle fenêtre ou un nouvel onglet s’ouvre.
5. Développez les nœuds **config**, **authsettings** pour votre site dans le volet de navigation de gauche.
6. Cliquez sur **Modifier**.
7. Recherchez l’élément "allowedExternalRedirectUrls". Il est défini sur Null. Modifiez-le comme suit :

         "allowedExternalRedirectUrls": [
             "http://localhost:3000",
             "https://localhost:3000"
         ],

    Remplacez les URL par les URL de votre service. Par exemple, "http://localhost:3000" (pour le service Node.js) ou "http://localhost:4400" (pour le service Ripple). Il s’agit seulement d’exemples. Votre situation, y compris pour les services mentionnés dans les exemples, peut être différente.
8. Cliquez sur le bouton **Lecture/Écriture** dans le coin supérieur droit de l’écran.
9. Cliquez sur le bouton vert **PUT**.

Les paramètres sont alors enregistrés. Ne fermez pas la fenêtre du navigateur avant la fin de l’enregistrement des paramètres. Vous devez également ajouter ces URL de bouclage aux paramètres de CORS :

1. Connectez-vous au [portail Azure].
2. Sélectionnez **Toutes les ressources** ou **App Services**, puis cliquez sur le nom de votre application mobile.
3. Le panneau Paramètres s’ouvre automatiquement. Si ce n’est pas le cas, cliquez sur **Tous les paramètres**.
4. Cliquez sur **CORS** sous le menu de l’API.
5. Entrez l’URL à ajouter dans la zone correspondante et appuyez sur Entrée.
6. Entrez des URL supplémentaires, si nécessaire.
7. Cliquez sur **Enregistrer** pour enregistrer les paramètres.

L’application des nouveaux paramètres prend environ 10 à 15 secondes.

##<a name="register-for-push"></a>Procédure : inscription aux notifications Push

Installez le plug-in [phonegap-plugin-push] pour gérer les notifications Push. Vous pouvez l’ajouter facilement en exécutant la commande `cordova plugin add` sur la ligne de commande, ou par le biais du programme d’installation de plug-in Git dans Visual Studio. Le code suivant dans votre application Apache Cordova inscrit votre appareil aux notifications Push :

```
var pushOptions = {
    android: {
        senderId: '<from-gcm-console>'
    },
    ios: {
        alert: true,
        badge: true,
        sound: true
    },
    windows: {
    }
};
pushHandler = PushNotification.init(pushOptions);

pushHandler.on('registration', function (data) {
    registrationId = data.registrationId;
    // For cross-platform, you can use the device plugin to determine the device
    // Best is to use device.platform
    var name = 'gcm'; // For android - default
    if (device.platform.toLowerCase() === 'ios')
        name = 'apns';
    if (device.platform.toLowerCase().substring(0, 3) === 'win')
        name = 'wns';
    client.push.register(name, registrationId);
});

pushHandler.on('notification', function (data) {
    // data is an object and is whatever is sent by the PNS - check the format
    // for your particular PNS
});

pushHandler.on('error', function (error) {
    // Handle errors
});
```

Utilisez le Kit de développement logiciel (SDK) Notification Hubs pour envoyer des notifications Push à partir du serveur. Vous ne devez jamais envoyer de notifications Push directement à partir de clients, car cela peut favoriser le déclenchement d’une attaque par déni de service contre Notification Hubs ou le PNS.

<!-- URLs. -->
[portail Azure]: https://portal.azure.com
[Démarrage rapide d’Azure Mobile Apps]: app-service-mobile-cordova-get-started.md
[Prise en main de l'authentification]: app-service-mobile-cordova-get-started-users.md
[Ajout de l’authentification à votre application]: app-service-mobile-cordova-get-started-users.md

[plug-in Apache Cordova pour Azure Mobile Apps]: https://www.npmjs.com/package/cordova-plugin-ms-azure-mobile-apps
[votre première application Apache Cordova]: http://cordova.apache.org/#getstarted
[phonegap-facebook-plugin]: https://github.com/wizcorp/phonegap-facebook-plugin
[phonegap-plugin-push]: https://www.npmjs.com/package/phonegap-plugin-push
[cordova-plugin-device]: https://www.npmjs.com/package/cordova-plugin-device
[cordova-plugin-inappbrowser]: https://www.npmjs.com/package/cordova-plugin-inappbrowser
[documentation de l’objet Query]: https://msdn.microsoft.com/fr-FR/library/azure/jj613353.aspx

<!-----HONumber=AcomDC_0316_2016-->