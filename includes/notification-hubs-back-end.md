
Cette section montre comment envoyer des notifications à partir de l’application console .NET (entre autres). Si vous utilisez Mobile Services, reportez-vous aux didacticiels [Prise en main des notifications Push](mobile-services-dotnet-backend-windows-store-dotnet-get-started-push.md). Si vous souhaitez utiliser Java ou PHP, consultez la rubrique [Utilisation de Notification Hubs depuis Java/PHP](../articles/notification-hubs/notification-hubs-java-backend-how-to.md). Vous pouvez envoyer des notifications à partir d’un serveur principal à l’aide de l’[interface REST de Notification Hub](http://msdn.microsoft.com/library/windowsazure/dn223264.aspx).

Le code suivant envoie des notifications aux appareils Windows Store, Windows Phone, iOS et Android.

Ignorez les étapes 1 à 3 si vous avez créé une application de console lorsque vous avez effectué la [prise en main des Notification Hubs][get-started].

1. Dans Visual Studio, créez une application console Visual C# : 

   	![][13]

2. Dans le menu principal de Visual Studio, cliquez sur **Outils**, **Gestionnaire de package de bibliothèques** et **Console du gestionnaire de package**, puis dans la fenêtre de la console, tapez la ligne suivante et appuyez sur **Entrée** :

        Install-Package Microsoft.Azure.NotificationHubs
 	
	Cette opération ajoute une référence au Kit de développement logiciel (SDK) Azure Notification Hubs à l’aide du <a href="http://www.nuget.org/packages/Microsoft.Azure.NotificationHubs/">package NuGet Microsoft.Azure.Notification Hubs</a>.

3. Ouvrez le fichier Program.cs et ajoutez l’instruction `using` suivante :

        using Microsoft.Azure.NotificationHubs;

4. Dans la classe `Program`, ajoutez la méthode suivante ou remplacez-la si elle existe déjà :

        private static async void SendNotificationAsync()
        {
			// Define the notification hub.
		    NotificationHubClient hub = 
				NotificationHubClient.CreateClientFromConnectionString(
					"<connection string with full access>", "<hub name>");
		
		    // Create an array of breaking news categories.
		    var categories = new string[] { "World", "Politics", "Business", 
		        "Technology", "Science", "Sports"};
		
            foreach (var category in categories)
            {
                try
                {
                    // Define a Windows Store toast.
                    var wnsToast = "<toast><visual><binding template="ToastText01">" 
                        + "<text id="1">Breaking " + category + " News!" 
                        + "</text></binding></visual></toast>";         
                    await hub.SendWindowsNativeNotificationAsync(wnsToast, category);

                    // Define a Windows Phone toast.
                    var mpnsToast =
                        "<?xml version="1.0" encoding="utf-8"?>" +
                        "<wp:Notification xmlns:wp="WPNotification">" +
                            "<wp:Toast>" +
                                "<wp:Text1>Breaking " + category + " News!</wp:Text1>" +
                            "</wp:Toast> " +
                        "</wp:Notification>";         
                    await hub.SendMpnsNativeNotificationAsync(mpnsToast, category);

                    // Define an iOS alert.
                    var alert = "{"aps":{"alert":"Breaking " + category + " News!"}}";
                    await hub.SendAppleNativeNotificationAsync(alert, category);

					// Define an Android notification.
                    var notification = "{"data":{"msg":"Breaking " + category + " News!"}}";
                    await hub.SendGcmNativeNotificationAsync(notification, category);
                }
                catch (ArgumentException)
                {
                    // An exception is raised when the notification hub hasn't been 
                    // registered for the iOS, Windows Store, or Windows Phone platform. 
                }
            }
		 }

	Ce code envoie des notifications à chacune des six balises du tableau de chaînes aux appareils iOS, Windows Store et Windows Phone. L’utilisation des balises permet d’envoyer les notifications uniquement aux appareils des catégories inscrites.
	
	> [AZURE.NOTE]Ce code principal prend en charge les clients Windows Store, Windows Phone, iOS et Android. Les méthodes d’envoi retournent une réponse d’erreur lorsque le hub de notification n’a pas encore été configuré pour une plateforme cliente particulière.

6. Dans le code ci-dessus, remplacez les espaces réservés `<hub name>` et `<connection string with full access>` par le nom de votre hub de notification et par la chaîne de connexion de la signature *DefaultFullSharedAccessSignature* obtenue précédemment.

7. Ajoutez les lignes suivantes à la méthode **Main** :

         SendNotificationAsync();
		 Console.ReadLine();

<!-- Anchors -->
[From a console app]: #console
[From Mobile Services]: #mobile-services
[Run the app and generate notifications]: #test-app

<!-- Images. -->
[13]: ./media/notification-hubs-back-end/notification-hub-create-console-app.png

[15]: ./media/notification-hubs-back-end/notification-hub-scheduler1.png
[16]: ./media/notification-hubs-back-end/notification-hub-scheduler2.png

<!-- URLs. -->
[get-started]: ../articles/notification-hubs/notification-hubs-windows-store-dotnet-get-started.md
[Use Notification Hubs to send notifications to users]: ../articles/tutorial-notify-users-mobileservices.md
[Get started with Mobile Services]: /develop/mobile/tutorials/get-started/#create-new-service
[wns object]: http://go.microsoft.com/fwlink/p/?LinkId=260591
[Notification Hubs Guidance]: http://msdn.microsoft.com/library/jj927170.aspx
[Notification Hubs How-To for Windows Store]: http://msdn.microsoft.com/library/jj927172.aspx
[Notification Hubs REST interface]: http://msdn.microsoft.com/library/windowsazure/dn223264.aspx

<!---HONumber=AcomDC_1210_2015-->