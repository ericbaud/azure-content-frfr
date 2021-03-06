1.	Connectez-vous au [portail Microsoft Azure](https://portal.azure.com/) en ligne.
2.	Dans la barre de lancement, cliquez sur **Nouveau**, sélectionnez **Données + stockage**, puis cliquez sur **DocumentDB**.
  
	![Capture d’écran du portail Azure pour créer une base de données, mise en surbrillance du bouton Nouveau, Données + stockage dans le panneau Créer et Azure DocumentDB dans le panneau Données + stockage](./media/documentdb-create-dbaccount/create-nosql-db-databases-json-tutorial-1.png)

3. Dans le panneau **Nouveau compte DocumentDB**, indiquez la configuration souhaitée pour le compte DocumentDB.
 
	![Capture d’écran du panneau Nouveau DocumentDB](./media/documentdb-create-dbaccount/create-nosql-db-databases-json-tutorial-2.png)


	- Dans la zone **ID**, entrez un nom pour identifier le compte DocumentDB. Lorsque l’**ID** est validé, une coche verte s’affiche dans la case **ID**. La valeur pour **ID** devient le nom d’hôte dans l’URI. Cet **ID** ne peut contenir que des minuscules, des chiffres, le caractère « - » et doit compter entre 3 et 50 caractères. Notez que *documents.azure.com* sera ajouté au nom du point de terminaison de votre choix. Celui-ci deviendra le point de terminaison de votre compte DocumentDB.
	

	
	- Dans **Abonnement**, sélectionnez l'abonnement Azure à utiliser avec le compte DocumentDB. Si votre compte ne comporte qu’un seul abonnement, ce compte sera sélectionné par défaut.

	- Dans **Groupe de ressources**, sélectionnez ou créez un groupe de ressources pour votre compte DocumentDB. Un groupe de ressources déjà attaché à l’abonnement Azure est sélectionné par défaut. Vous pouvez cependant choisir de sélectionner un nouveau groupe de ressources auquel ajouter votre compte DocumentDB. Pour plus d’informations, consultez [Utilisation du portail Azure pour gérer vos ressources Azure](resource-group-portal.md).
 
	- Utilisez **Emplacement** pour indiquer l’emplacement géographique de l’hébergement de votre compte DocumentDB.   

4.	Une fois les options du nouveau compte DocumentDB configurées, cliquez sur **Créer**.  La création du compte DocumentDB peut prendre plusieurs minutes. Pour vérifier l’état, vous pouvez suivre l’avancement sur le Tableau d’accueil.  
	![Capture d’écran de la vignette Création dans le Tableau d’accueil - créateur de base de données en ligne](./media/documentdb-create-dbaccount/create-nosql-db-databases-json-tutorial-3.png)
  
	Vous pouvez aussi surveiller l’avancement depuis le hub de notifications.  

	![Création rapide de bases de données - capture d’écran du hub de notifications, indiquant que le compte DocumentDB est en cours de création](./media/documentdb-create-dbaccount/create-nosql-db-databases-json-tutorial-4.png)

	![Capture d’écran du hub de notifications montrant le compte DocumentDB créé avec succès et déployé vers un groupe de ressources - notification du créateur de base de données en ligne](./media/documentdb-create-dbaccount/create-nosql-db-databases-json-tutorial-5.png)

5.	Une fois le compte DocumentDB créé, il est prêt à être utilisé avec les paramètres par défaut dans le portail en ligne. Notez que la cohérence par défaut du compte DocumentDB est définie sur **Par session**. Pour ajuster le paramètre de cohérence par défaut, cliquez sur l’icône **Paramètres** située sur la barre de commande supérieure, puis cliquez sur l’entrée **Cohérence par défaut** sous **Fonctionnalité** dans le panneau **Tous les paramètres**.

    ![Capture d’écran du panneau Groupe de ressources - commencer le développement d’applications](./media/documentdb-create-dbaccount/create-nosql-db-databases-json-tutorial-6.png)

    ![Capture d’écran du panneau Niveau de cohérence - Cohérence de session](./media/documentdb-create-dbaccount/create-nosql-db-databases-json-tutorial-7.png)

[How to: Create a DocumentDB account]: #Howto
[Next steps]: #NextSteps
[documentdb-manage]: ../articles/documentdb/documentdb-manage.md

<!---HONumber=AcomDC_0302_2016-->