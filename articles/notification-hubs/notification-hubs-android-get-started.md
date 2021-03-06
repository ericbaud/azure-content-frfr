<properties
	pageTitle="Envoi de notifications Push vers Android avec Azure Notification Hubs | Microsoft Azure"
	description="Dans ce didacticiel, vous découvrirez comment utiliser Azure Notification Hubs pour envoyer des notifications Push à une application Android."
	services="notification-hubs"
	documentationCenter="android"
	keywords="notifications push,notification push,notification push android"
	authors="wesmc7777"
	manager="dwrede"
	editor=""/>
<tags
	ms.service="notification-hubs"
	ms.workload="mobile"
	ms.tgt_pltfrm="mobile-android"
	ms.devlang="java"
	ms.topic="hero-article"
	ms.date="03/15/2016"
	ms.author="wesmc"/>

# Envoi de notifications Push vers Android avec Azure Notification Hubs

[AZURE.INCLUDE [notification-hubs-selector-get-started](../../includes/notification-hubs-selector-get-started.md)]

##Vue d'ensemble

> [AZURE.IMPORTANT] Pour suivre ce didacticiel, vous avez besoin d'un compte Azure actif. Si vous ne possédez pas de compte, vous pouvez créer un compte d'évaluation gratuit en quelques minutes. Pour plus d'informations, consultez la page [Version d'évaluation gratuite d'Azure](https://azure.microsoft.com/pricing/free-trial/?WT.mc_id=A643EE910&amp;returnurl=http%3A%2F%2Fazure.microsoft.com%2Ffr-FR%2Fdocumentation%2Farticles%2Fnotification-hubs-android-get-started).

Ce didacticiel montre comment utiliser Azure Notification Hubs pour envoyer des notifications Push vers une application Android. Vous allez créer une application Android vide qui reçoit des notifications Push à l’aide de Google Cloud Messaging (GCM).

Veillez à suivre le [didacticiel de balisage](./notification-hubs-routing-tag-expressions.md) pour savoir comment utiliser Notification Hubs pour envoyer des notifications Push ciblées.


## Avant de commencer

[AZURE.INCLUDE [notification-hubs-hero-slug](../../includes/notification-hubs-hero-slug.md)]

Le code complet de ce didacticiel est disponible sur GitHub [ici](https://github.com/Azure/azure-notificationhubs-samples/tree/master/Android/GetStarted).


##Composants requis

En plus du compte Azure actif mentionné ci-dessus, ce didacticiel requiert uniquement la dernière version d’[Android Studio](http://go.microsoft.com/fwlink/?LinkId=389797).

Vous devez suivre ce didacticiel avant de pouvoir suivre tous les autres didacticiels Notification Hubs pour les applications Android.

##Création d’un projet prenant en charge Google Cloud Messaging

[AZURE.INCLUDE [mobile-services-enable-Google-cloud-messaging](../../includes/mobile-services-enable-google-cloud-messaging.md)]


##Configurer un nouveau hub de notification


[AZURE.INCLUDE [notification-hubs-portal-create-new-hub](../../includes/notification-hubs-portal-create-new-hub.md)]


&emsp;&emsp;7. Dans le panneau **Paramètres**, sélectionnez **Notification Services**, puis **Google (GCM)**. Entrez la clé d’API et enregistrez.

&emsp;&emsp;![Azure Notification Hubs - Google (GCM)](./media/notification-hubs-android-get-started/notification-hubs-gcm-api.png)

Votre hub de notification est à présent configuré pour GCM, et vous disposez des chaînes de connexion vous permettant d’inscrire votre application pour la réception et l’envoi de notifications Push.

##<a id="connecting-app"></a>Connexion de votre application au hub de notification

###Créer un projet Android

1. Dans Android Studio, démarrez un nouveau projet Android Studio.

   	![Android Studio - nouveau projet][13]

2. Choisissez le format **Phone and Tablet** et le kit de développement logiciel (SDK) minimal (**Minimum SDK**) que vous souhaitez prendre en charge. Cliquez ensuite sur **Next**.

   	![Android Studio - workflow de création de projet][14]

3. Choisissez **Activité vide** comme activité principale, cliquez sur **Suivant**, puis cliquez sur **Terminer**.

###Ajout de services Google Play au projet

[AZURE.INCLUDE [Ajout de services Google Play](../../includes/notification-hubs-android-studio-add-google-play-services.md)]

###Ajout de code

1. Téléchargez le fichier `notification-hubs-0.4.jar` à partir de l’onglet **Fichiers** du [SDK Notification-Hubs-Android sur Bintray](https://bintray.com/microsoftazuremobile/SDK/Notification-Hubs-Android-SDK/0.4). Faites glisser les fichiers directement dans le dossier **libs** dans la fenêtre Project View d’Android Studio. Cliquez ensuite avec le bouton droit sur le fichier, puis cliquez sur **Ajouter en tant que bibliothèque**.

2. Dans le fichier `Build.Gradle` de l’**application**, dans la section **dépendances**, ajoutez la ligne qui suit.

	    compile 'com.microsoft.azure:azure-notifications-handler:1.0.1@aar'

	Ajoutez le référentiel suivant après la section **dépendances**.

		repositories {
		    maven {
		        url "http://dl.bintray.com/microsoftazuremobile/SDK"
		    }
		}

3. Configurez l’application afin d’obtenir un ID d’enregistrement depuis GCM, et utilisez-la pour inscrire l’instance d’application auprès du hub de notification.

	Dans le fichier `AndroidManifest.xml`, ajoutez les autorisations suivantes juste sous la balise `</application>`. Veillez à remplacer `<your package>` par le nom du package qui apparaît en haut du fichier `AndroidManifest.xml` (`com.example.testnotificationhubs` dans cet exemple).

		<uses-permission android:name="android.permission.INTERNET"/>
		<uses-permission android:name="android.permission.GET_ACCOUNTS"/>
		<uses-permission android:name="android.permission.WAKE_LOCK"/>
		<uses-permission android:name="com.google.android.c2dm.permission.RECEIVE" />

		<permission android:name="<your package>.permission.C2D_MESSAGE" android:protectionLevel="signature" />
		<uses-permission android:name="<your package>.permission.C2D_MESSAGE"/>

3. Dans la classe **MainActivity**, ajoutez les instructions `import` suivantes au-dessus de la déclaration de la classe.

		import android.app.AlertDialog;
		import android.content.DialogInterface;
		import android.os.AsyncTask;
		import com.google.android.gms.gcm.*;
		import com.microsoft.windowsazure.messaging.*;
		import com.microsoft.windowsazure.notifications.NotificationsManager;
		import android.widget.Toast;



4. Ajoutez les membres privés suivants dans la partie supérieure de la classe. Nous les utilisons pour configurer le canal de notifications Push entre votre application et les services cloud.

		private String SENDER_ID = "<your project number>";
		private GoogleCloudMessaging gcm;
		private NotificationHub hub;
    	private String HubName = "<Enter Your Hub Name>";
		private String HubListenConnectionString = "<Your default listen connection string>";
	    private static Boolean isVisible = false;


	Veillez à mettre à jour les trois espaces réservés :
	* **SENDER\_ID** : définissez `SENDER_ID` sur le numéro de projet que vous avez obtenu précédemment dans la [Google Cloud Console](http://cloud.google.com/console).
	* **HubListenConnectionString** : définissez `HubListenConnectionString` sur la chaîne de connexion **DefaultListenAccessSignature** de votre hub. Vous pouvez copier cette chaîne de connexion en cliquant sur **Stratégies d’accès** dans le panneau **Paramètres** de votre hub sur le [portail Azure].
	* **HubName** : utilisez le nom de votre hub de notification qui s’affiche dans le panneau Hub sur le [portail Azure].


5. Dans la méthode `OnCreate` de la classe `MainActivity`, ajoutez le code suivant pour effectuer l’inscription auprès du hub de notification lorsque l’activité est créée.

        MyHandler.mainActivity = this;
        NotificationsManager.handleNotifications(this, SENDER_ID, MyHandler.class);
        gcm = GoogleCloudMessaging.getInstance(this);
        hub = new NotificationHub(HubName, HubListenConnectionString, this);
        registerWithNotificationHubs();

6. Dans `MainActivity.java`, ajoutez le code ci-dessous pour la méthode `registerWithNotificationHubs()`. Cette méthode permet de signaler que l’inscription auprès du hub de notification et de Google Cloud Messaging a réussi :

    	@SuppressWarnings("unchecked")
    	private void registerWithNotificationHubs() {
        	new AsyncTask() {
            	@Override
            	protected Object doInBackground(Object... params) {
                	try {
                    	String regid = gcm.register(SENDER_ID);
                    ToastNotify("Registered Successfully - RegId : " +
						hub.register(regid).getRegistrationId());
                	} catch (Exception e) {
                    	ToastNotify("Registration Exception Message - " + e.getMessage());
                    	return e;
                	}
                	return null;
            	}
        	}.execute(null, null, null);
    	}


7. Ajoutez la méthode `ToastNotify` à l’activité pour afficher la notification lorsque l’application est en cours d’exécution et visible. Substituez également `onStart`, `onPause`, `onResume` et `onStop` afin de déterminer si l’activité est visible pour afficher le toast.

	    @Override
	    protected void onStart() {
	        super.onStart();
	        isVisible = true;
	    }

	    @Override
	    protected void onPause() {
	        super.onPause();
	        isVisible = false;
	    }

	    @Override
	    protected void onResume() {
	        super.onResume();
	        isVisible = true;
	    }

	    @Override
	    protected void onStop() {
	        super.onStop();
	        isVisible = false;
	    }

	    public void ToastNotify(final String notificationMessage)
	    {
	        if (isVisible == true)
	            runOnUiThread(new Runnable() {
	                @Override
	                public void run() {
	                    Toast.makeText(MainActivity.this, notificationMessage, Toast.LENGTH_LONG).show();
	                }
	            });
	    }

8. Android ne prenant pas en charge par défaut l’affichage des notifications, vous devez écrire votre propre récepteur. Dans `AndroidManifest.xml`, ajoutez l’élément ci-après dans l’élément `<application>`.

	> [AZURE.NOTE] Remplacez l’espace réservé par votre nom de package.

        <receiver android:name="com.microsoft.windowsazure.notifications.NotificationsBroadcastReceiver"
            android:permission="com.google.android.c2dm.permission.SEND">
            <intent-filter>
                <action android:name="com.google.android.c2dm.intent.RECEIVE" />
                <category android:name="<your package name>" />
            </intent-filter>
        </receiver>


9. Dans la vue du projet, développez **app** -> **src** -> **main** -> **java**. Cliquez avec le bouton droit sur le dossier de votre package sous **java**, cliquez sur **Nouveau**, puis cliquez sur **Classe Java**.

	![Android Studio - nouvelle classe Java][6]

10. Dans le champ **Nom** associé à la nouvelle classe, saisissez **MyHandler**, puis cliquez sur **OK**.


11. Ajoutez les instructions import suivantes au-dessus de `MyHandler.java` :

		import android.app.NotificationManager;
		import android.app.PendingIntent;
		import android.content.Context;
		import android.content.Intent;
		import android.os.Bundle;
		import android.support.v4.app.NotificationCompat;
		import com.microsoft.windowsazure.notifications.NotificationsHandler;


12. Mettez à jour la déclaration de la classe comme suit pour faire de `MyHandler` une sous-classe de `com.microsoft.windowsazure.notifications.NotificationsHandler` comme indiqué ci-dessous.

		public class MyHandler extends NotificationsHandler {


13. Ajoutez le code suivant pour la classe `MyHandler`.

	Comme ce code remplace la méthode `OnReceive`, le gestionnaire affiche un toast pour répertorier les notifications reçues. Le gestionnaire envoie également la notification Push au gestionnaire de notifications Android en utilisant la méthode `sendNotification()`.

    	public static final int NOTIFICATION_ID = 1;
    	private NotificationManager mNotificationManager;
    	NotificationCompat.Builder builder;
    	Context ctx;

    	static public MainActivity mainActivity;

    	@Override
    	public void onReceive(Context context, Bundle bundle) {
        	ctx = context;
        	String nhMessage = bundle.getString("message");

        	sendNotification(nhMessage);
	        mainActivity.ToastNotify(nhMessage);
    	}

    	private void sendNotification(String msg) {
        	mNotificationManager = (NotificationManager)
                ctx.getSystemService(Context.NOTIFICATION_SERVICE);

        	PendingIntent contentIntent = PendingIntent.getActivity(ctx, 0,
                new Intent(ctx, MainActivity.class), 0);

			NotificationCompat.Builder mBuilder =
				new NotificationCompat.Builder(ctx)
					.setSmallIcon(R.mipmap.ic_launcher)
					.setContentTitle("Notification Hub Demo")
					.setStyle(new NotificationCompat.BigTextStyle()
					.bigText(msg))
					.setContentText(msg);

			mBuilder.setContentIntent(contentIntent);
			mNotificationManager.notify(NOTIFICATION_ID, mBuilder.build());
		}

14. Dans Android Studio, sur la barre de menus, cliquez sur **Build** -> **Rebuild Project** pour vous assurer que votre code ne contient aucune erreur.

##Envoi de notifications Push

Vous pouvez tester la réception de notifications dans votre application en les envoyant via le [portail Azure] (observez la section **Dépannage** dans le panneau Hub, comme illustré ci-dessus).

![Azure Notification Hubs - Test d’envoi](./media/notification-hubs-android-get-started/notification-hubs-test-send.png)

[AZURE.INCLUDE [notification-hubs-sending-notifications-from-the-portal](../../includes/notification-hubs-sending-notifications-from-the-portal.md)]

## (Facultatif) Envoyer des notifications Push directement depuis l’application

Dans la plupart des cas de test, vous chercherez peut-être à envoyer des notifications Push directement à partir de l’application que vous développez. Cette section explique comment implémenter correctement ce scénario.

1. Dans la vue de projet Android Studio, développez **App** -> **src** -> **main** -> **res** -> **layout**. Ouvrez le fichier de disposition `activity_main.xml` et cliquez sur l’onglet **Texte** pour mettre à jour le texte du fichier. Mettez-le à jour avec le code suivant, qui ajoute deux nouveaux contrôles `Button` et `EditText` pour l’envoi des messages de notification Push au hub de notification. Ajoutez ce code en bas, juste avant `</RelativeLayout>`.

	    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="@string/send_button"
        android:id="@+id/sendbutton"
        android:layout_centerVertical="true"
        android:layout_centerHorizontal="true"
        android:onClick="sendNotificationButtonOnClick" />

	    <EditText
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:id="@+id/editTextNotificationMessage"
        android:layout_above="@+id/sendbutton"
        android:layout_centerHorizontal="true"
        android:layout_marginBottom="42dp"
        android:hint="@string/notification_message_hint" />

2. Ajoutez cette ligne au fichier `build.gradle` sous `android`

		useLibrary 'org.apache.http.legacy'

3. Dans la vue de projet Android Studio, développez **App** -> **src** -> **main** -> **res** -> **values**. Ouvrez le fichier `strings.xml` et ajoutez les valeurs de chaîne référencées par les nouveaux contrôles `Button` et `EditText`. Ajoutez-les en bas du fichier juste avant `</resources>`.

        <string name="send_button">Send Notification</string>
        <string name="notification_message_hint">Enter notification message text</string>


4. Dans votre fichier `MainActivity.java`, ajoutez les instructions `import` suivantes au-dessus de la classe `MainActivity`.

		import java.net.URLEncoder;
		import javax.crypto.Mac;
		import javax.crypto.spec.SecretKeySpec;

		import android.util.Base64;
		import android.view.View;
		import android.widget.EditText;

		import org.apache.http.HttpResponse;
		import org.apache.http.client.HttpClient;
		import org.apache.http.client.methods.HttpPost;
		import org.apache.http.entity.StringEntity;
		import org.apache.http.impl.client.DefaultHttpClient;


5. Dans votre fichier `MainActivity.java`, ajoutez les membres suivants au-dessus de la classe `MainActivity`.

	Mettez à jour `HubFullAccess` avec la chaîne de connexion **DefaultFullSharedAccessSignature** correspondant à votre hub. Vous pouvez copier cette chaîne de connexion depuis le [portail Azure] en cliquant sur **Stratégies d’accès** dans le panneau **Paramètres** de votre hub de notification.

	    private String HubEndpoint = null;
	    private String HubSasKeyName = null;
	    private String HubSasKeyValue = null;
		private String HubFullAccess = "<Enter Your DefaultFullSharedAccess Connection string>";

6. Votre activité conserve le nom du hub et la chaîne de connexion d'accès partagé complet pour le hub. Vous devez créer un jeton SaS (Software Access Signature) pour authentifier une demande POST d'envoi de messages à votre hub de notification. Cette opération est effectuée grâce à l’analyse des données clés de la chaîne de connexion, puis en créant le jeton SaS, comme indiqué sur la page [Concepts courants](http://msdn.microsoft.com/library/azure/dn495627.aspx) des informations de référence sur l’API REST.

	Dans `MainActivity.java`, ajoutez la méthode suivante à la classe `MainActivity` pour analyser votre chaîne de connexion.

	    /**
    	 * Example code from http://msdn.microsoft.com/library/azure/dn495627.aspx
    	 * to parse the connection string so a SaS authentication token can be
    	 * constructed.
    	 *
    	 * @param connectionString This must be the DefaultFullSharedAccess connection
    	 *                         string for this example.
	     */
	    private void ParseConnectionString(String connectionString)
	    {
	        String[] parts = connectionString.split(";");
	        if (parts.length != 3)
	            throw new RuntimeException("Error parsing connection string: "
	                    + connectionString);

	        for (int i = 0; i < parts.length; i++) {
	            if (parts[i].startsWith("Endpoint")) {
	                this.HubEndpoint = "https" + parts[i].substring(11);
	            } else if (parts[i].startsWith("SharedAccessKeyName")) {
	                this.HubSasKeyName = parts[i].substring(20);
	            } else if (parts[i].startsWith("SharedAccessKey")) {
	                this.HubSasKeyValue = parts[i].substring(16);
	            }
	        }
	    }

7. Dans `MainActivity.java`, ajoutez la méthode suivante à la classe `MainActivity` pour créer un jeton d’authentification SaS.

        /**
         * Example code from http://msdn.microsoft.com/library/azure/dn495627.aspx to
         * construct a SaS token from the access key to authenticate a request.
         *
         * @param uri The unencoded resource URI string for this operation. The resource
         *            URI is the full URI of the Service Bus resource to which access is
         *            claimed. For example,
         *            "http://<namespace>.servicebus.windows.net/<hubName>"
         */
        private String generateSasToken(String uri) {

            String targetUri;
            try {
                targetUri = URLEncoder
                        .encode(uri.toString().toLowerCase(), "UTF-8")
                        .toLowerCase();

                long expiresOnDate = System.currentTimeMillis();
                int expiresInMins = 60; // 1 hour
                expiresOnDate += expiresInMins * 60 * 1000;
                long expires = expiresOnDate / 1000;
                String toSign = targetUri + "\n" + expires;

                // Get an hmac_sha1 key from the raw key bytes
                byte[] keyBytes = HubSasKeyValue.getBytes("UTF-8");
                SecretKeySpec signingKey = new SecretKeySpec(keyBytes, "HmacSHA256");

                // Get an hmac_sha1 Mac instance and initialize with the signing key
                Mac mac = Mac.getInstance("HmacSHA256");
                mac.init(signingKey);

                // Compute the hmac on input data bytes
                byte[] rawHmac = mac.doFinal(toSign.getBytes("UTF-8"));

            	// Using android.util.Base64 for Android Studio instead of
	            // Apache commons codec
                String signature = URLEncoder.encode(
                        Base64.encodeToString(rawHmac, Base64.NO_WRAP).toString(), "UTF-8");

                // Construct authorization string
                String token = "SharedAccessSignature sr=" + targetUri + "&sig="
                        + signature + "&se=" + expires + "&skn=" + HubSasKeyName;
                return token;
            } catch (Exception e) {
                DialogNotify("Exception Generating SaS",e.getMessage().toString());
            }

            return null;
        }


8. Dans `MainActivity.java`, ajoutez la méthode suivante à la classe `MainActivity` pour gérer le clic sur le bouton **Envoyer une notification** et envoyer le message de notification Push au hub à l’aide de l’API REST intégrée.

        /**
         * Send Notification button click handler. This method parses the
         * DefaultFullSharedAccess connection string and generates a SaS token. The
         * token is added to the Authorization header on the POST request to the
         * notification hub. The text in the editTextNotificationMessage control
         * is added as the JSON body for the request to add a GCM message to the hub.
         *
         * @param v
         */
        public void sendNotificationButtonOnClick(View v) {
            EditText notificationText = (EditText) findViewById(R.id.editTextNotificationMessage);
            final String json = "{"data":{"message":"" + notificationText.getText().toString() + ""}}";

            new Thread()
            {
                public void run()
                {
                    try
                    {
                        HttpClient client = new DefaultHttpClient();

                        // Based on reference documentation...
                        // http://msdn.microsoft.com/library/azure/dn223273.aspx
                        ParseConnectionString(HubFullAccess);
                        String url = HubEndpoint + HubName + "/messages/?api-version=2015-01";
                        HttpPost post = new HttpPost(url);

                        // Authenticate the POST request with the SaS token
                        post.setHeader("Authorization", generateSasToken(url));

                        // JSON content for GCM
                        post.setHeader("Content-Type", "application/json;charset=utf-8");

                        // Notification format should be GCM
                        post.setHeader("ServiceBusNotification-Format", "gcm");
                        post.setEntity(new StringEntity(json));

                        HttpResponse response = client.execute(post);
                    }
                    catch(Exception e)
                    {
                        DialogNotify("Exception",e.getMessage().toString());
                    }
                }
            }.start();
        }



##Test de votre application

####Notifications Push dans l’émulateur

Si vous souhaitez effectuer les tests de notifications Push sur un émulateur, vérifiez que votre image d’émulateur prend en charge le niveau d’API Google que vous avez choisi pour votre application. Si votre image ne prend pas en charge les API Google natives, l’exception **SERVICE\_NOT\_AVAILABLE** est levée.

Parallèlement à cela, assurez-vous que vous avez ajouté votre compte Google à l’émulateur en cours d’exécution sous **Paramètres**->**Comptes**. Sinon, vos tentatives d'inscription auprès de GCM peuvent entraîner la levée de l'exception **AUTHENTICATION\_FAILED**.

####Exécution de l'application

1. Exécutez l’application et notez qu’un ID d’inscription apparaît quand l’inscription réussit.

   	![Tests sur Android - Inscription de canal][18]

2. Entrez le message de notification à envoyer à tous les appareils Android inscrits auprès du hub.

   	![Tests sur Android - envoi d’un message][19]

3. Appuyez sur **Envoyer une notification**. Tous les appareils sur lesquels l’application est en cours d’exécution affichent une instance `AlertDialog` comportant le message de notification Push. Les appareils sur lesquels l’application n’est pas en cours d’exécution, mais qui ont déjà été inscrits aux notifications Push, reçoivent une notification dans le gestionnaire de notifications Android. Vous pouvez afficher ces notifications en effectuant un balayage vers le bas depuis l’angle supérieur gauche.

   	![Tests sur Android - notifications][21]

##Étapes suivantes

Nous vous recommandons de consulter le didacticiel [Utiliser Notification Hubs pour envoyer des notifications Push aux utilisateurs] comme prochaine étape. Il vous expliquera comment envoyer des notifications à partir d’un serveur principal ASP.NET en utilisant des balises pour cibler des utilisateurs spécifiques.

Pour segmenter vos utilisateurs par groupes d’intérêt, consultez le didacticiel [Utilisation des Notification Hubs pour diffuser les dernières nouvelles].

Pour obtenir des informations générales sur Notification Hubs, consultez nos [Recommandations relatives à Notification Hubs].

<!-- Images. -->
[6]: ./media/notification-hubs-android-get-started/notification-hub-android-new-class.png

[12]: ./media/notification-hubs-android-get-started/notification-hub-connection-strings.png

[13]: ./media/notification-hubs-android-get-started/notification-hubs-android-studio-new-project.png
[14]: ./media/notification-hubs-android-get-started/notification-hubs-android-studio-choose-form-factor.png
[15]: ./media/notification-hubs-android-get-started/notification-hub-create-android-app4.png
[16]: ./media/notification-hubs-android-get-started/notification-hub-create-android-app5.png
[17]: ./media/notification-hubs-android-get-started/notification-hub-create-android-app6.png

[18]: ./media/notification-hubs-android-get-started/notification-hubs-android-studio-registered.png
[19]: ./media/notification-hubs-android-get-started/notification-hubs-android-studio-set-message.png

[20]: ./media/notification-hubs-android-get-started/notification-hub-create-console-app.png
[21]: ./media/notification-hubs-android-get-started/notification-hubs-android-studio-received-message.png
[22]: ./media/notification-hubs-android-get-started/notification-hub-scheduler1.png
[23]: ./media/notification-hubs-android-get-started/notification-hub-scheduler2.png
[29]: ./media/mobile-services-android-get-started-push/mobile-eclipse-import-Play-library.png

[30]: ./media/notification-hubs-android-get-started/notification-hubs-debug-hub-gcm.png

[31]: ./media/notification-hubs-android-get-started/notification-hubs-android-studio-add-ui.png


<!-- URLs. -->
[Get started with push notifications in Mobile Services]: ../mobile-services-javascript-backend-android-get-started-push.md
[Mobile Services Android SDK]: https://go.microsoft.com/fwLink/?LinkID=280126&clcid=0x409
[Referencing a library project]: http://go.microsoft.com/fwlink/?LinkId=389800
[Azure Classic Portal]: https://manage.windowsazure.com/
[Recommandations relatives à Notification Hubs]: http://msdn.microsoft.com/library/jj927170.aspx
[Utiliser Notification Hubs pour envoyer des notifications Push aux utilisateurs]: notification-hubs-aspnet-backend-android-notify-users.md
[Utilisation des Notification Hubs pour diffuser les dernières nouvelles]: notification-hubs-aspnet-backend-android-breaking-news.md
[portail Azure]: https://portal.azure.com

<!----HONumber=AcomDC_0316_2016-->
