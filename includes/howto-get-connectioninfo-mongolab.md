Lors de la mise en service d’une base de données MongoLab, MongoLab transmet un URI de connexion à Azure en utilisant le format de la chaîne de connexion standard de MongoDB. Cette valeur est utilisée pour initier une connexion MongoDB via le pilote MongoDB de votre choix. Pour plus d’informations sur les chaînes de connexion, consultez la page [Connexions](http://www.mongodb.org/display/DOCS/Connections) sur mongodb.org.

**Cet URI contient votre nom d’utilisateur et votre mot de passe pour la base de données. Considérez ces informations comme étant sensibles, ne les partagez pas.**

Pour récupérer cet URI à partir du portail Azure, procédez comme suit :

1. Sélectionnez **Modules complémentaires**. ![BoutonModules][button-addons]
1. Recherchez le service MongoLab dans la liste de modules. ![EntréeMongoLab][entry-mongolabaddon]
1. Pour accéder à la page du module, cliquez sur son nom.
1. Cliquez sur **Connection Info**. ![ConnectionInfoButton][button-connectioninfo] Votre URI MongoLab s’affiche : ![ÉcranInformationsDeConnexion][screen-connectioninfo]  
1.  Cliquez sur le bouton Presse-papiers à droite de la valeur MONGOLAB\_URI pour la copier entièrement vers le Presse-papiers.

[entry-mongolabaddon]: ./media/howto-get-connectioninfo-mongolab/entry-mongolabaddon.png
[button-connectioninfo]: ./media/howto-get-connectioninfo-mongolab/button-connectioninfo.png
[screen-connectioninfo]: ./media/howto-get-connectioninfo-mongolab/dialog-mongolab_connectioninfo.png
[button-addons]: ./media/howto-get-connectioninfo-mongolab/button-addons.png

<!---HONumber=Oct15_HO3-->