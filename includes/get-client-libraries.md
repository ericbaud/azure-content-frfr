### Installation via Composer

1. [Installez Git][install-git]. Notez que sous Windows, vous devez aussi ajouter l’exécutable Git à votre variable d’environnement PATH. 

2. Créez un fichier nommé **composer.json** à la racine de votre projet et ajoutez-y le code suivant :

	```
	{
	    "repositories": [
	        {
	            "type": "pear",
	            "url": "http://pear.php.net"
	        }
	    ],
	    "require": {
	        "pear-pear.php.net/mail_mime" : "*",
	        "pear-pear.php.net/http_request2" : "*",
	        "pear-pear.php.net/mail_mimedecode" : "*",
	        "microsoft/windowsazure": "*"
	    }
	}
	```

3. Téléchargez **[composer.phar][composer-phar]** à la racine du projet.

4. Ouvrez une invite de commande et exécutez la commande suivante à la racine du projet

	```
	php composer.phar install
	```

### Installation manuelle

Pour télécharger et installer manuellement les bibliothèques clientes PHP pour Azure, procédez comme suit :

> [AZURE.NOTE]Les bibliothèques clientes PHP pour Azure ont une dépendance sur les packages PEAR [HTTP\_Request2](http://pear.php.net/package/HTTP_Request2), [Mail\_mime](http://pear.php.net/package/Mail_mime) et [Mail\_mimeDecode](http://pear.php.net/package/Mail_mimeDecode). La méthode recommandée pour résoudre ces dépendances consiste à installer ces packages à l’aide du [Gestionnaire de package PEAR](http://pear.php.net/manual/en/installation.php).
 
1. Téléchargez une archive ZIP qui contient les bibliothèques de [GitHub][php-sdk-github]. Sinon, répliquez le répertoire et clonez-le sur votre ordinateur local. La deuxième option requiert un compte GitHub et l’installation locale de Git.
	
2. Copiez le répertoire `WindowsAzure` de l’archive téléchargée dans la structure de répertoires de votre application.

Pour plus d’informations sur l’installation des bibliothèques clientes PHP pour Azure (y compris des informations sur l’installation en tant que package PEAR), consultez la page [Télécharger le Kit de développement logiciel (SDK) Azure pour PHP][download-SDK-PHP].

[php-sdk-github]: http://go.microsoft.com/fwlink/?LinkId=252719
[install-git]: http://git-scm.com/book/en/Getting-Started-Installing-Git
[download-SDK-PHP]: ../articles/php-download-sdk.md
[composer-phar]: http://getcomposer.org/composer.phar

<!---HONumber=Oct15_HO3-->