<properties 
	pageTitle="Créer et charger des données dans des tables Hive à partir d'un stockage Blob | Microsoft Azure" 
	description="Créer des tables Hive et charger des données d’un blob dans des tables Hive" 
	services="machine-learning,storage" 
	documentationCenter="" 
	authors="hangzh-msft" 
	manager="jacob.spoelstra" 
	editor="cgronlun"  />

<tags 
	ms.service="machine-learning" 
	ms.workload="data-services" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="02/08/2016" 
	ms.author="hangzh;bradsev" />

 
#Créer et charger des données dans des tables Hive à partir d’un stockage Azure Blob

## Introduction
Ce **document** présente des requêtes Hive génériques qui créent des tables Hive et chargent des données à partir d’un stockage Azure Blob. Il donne également quelques conseils sur le partitionnement des tables Hive et sur l’utilisation du format ORC (Optimized Row Columnar) pour améliorer les performances des requêtes.

Ce **menu** pointe vers des rubriques qui décrivent comment recevoir les données dans les environnements cibles où les données peuvent être stockées et traitées pendant le processus d’analyse Cortana (CAP).

[AZURE.INCLUDE [cap-ingest-data-selector](../../includes/cap-ingest-data-selector.md)]


## Configuration requise
Cet article suppose que vous avez :
 
* Créé un compte Azure Storage. Si vous avez besoin d'aide, consultez [Créer un compte Azure Storage](../hdinsight-get-started.md#storage). 
* Approvisionné un cluster Hadoop personnalisé avec le service HDInsight. Si vous avez besoin d'aide, consultez [Personnaliser des clusters Hadoop Azure HDInsight pour l'analyse avancée](machine-learning-data-science-customize-hadoop-cluster.md).
* Activé l’accès à distance au cluster, saisi les identifiants appropriés et ouvert la console de ligne de commande Hadoop. Si vous avez besoin d'aide, consultez [Accéder au nœud principal du cluster Hadoop](machine-learning-data-science-customize-hadoop-cluster.md#headnode). 

## Téléchargement de données vers le stockage d’objets blob Azure
Si vous avez créé une machine virtuelle Azure en suivant les instructions de l’article [Configurer une machine virtuelle Azure pour l’analyse avancée](machine-learning-data-science-setup-virtual-machine.md), ce fichier de script doit avoir été téléchargé dans le répertoire *C:\\Users<nom\_utilisateur>\\Documents\\Data Science Scripts* de la machine virtuelle. Pour pouvoir être envoyées, ces requêtes Hive nécessitent simplement que votre schéma de données et la configuration de votre stockage Azure Blob soient déclarés dans les champs appropriés.

Nous partons du principe que les données des tables Hive ont un format tabulaire **non compressé** et qu’elles ont été chargées dans le conteneur par défaut (ou un conteneur supplémentaire) du compte de stockage utilisé par le cluster Hadoop.

Si vous souhaitez vous exercer sur les _données NYC Taxi Trip_, vous devez d'abord télécharger les 24 fichiers de <a href="http://www.andresmh.com/nyctaxitrips/" target="_blank">données NYC Taxi Trip</a> (12 fichiers Trip et 12 fichiers Fare), **décompresser** ces fichiers pour obtenir des fichiers .csv, puis les charger dans le conteneur par défaut (ou le conteneur approprié) du compte Azure Storage utilisé par la procédure dans la rubrique [Personnaliser les clusters Hadoop Azure HDInsight pour le processus et la technologie d’analyse avancée](machine-learning-data-science-customize-hadoop-cluster.md). Pour découvrir le processus qui vous permet de télécharger les fichiers .csv du conteneur par défaut sur le compte de stockage, consultez cette [page](machine-learning-data-science-process-hive-walkthrough/#upload).

## <a name="submit"></a>Comment envoyer des requêtes Hive
Pour envoyer des requêtes Hive, utilisez au choix :

* la ligne de commande Hadoop sur le nœud principal du cluster ;
* l’interpréteur IPython Notebook ;
* l’éditeur Hive ;
* des scripts Azure PowerShell.

Des requêtes Hive sont similaires à SQL. Les utilisateurs maîtrisant SQL peuvent trouver l'<a href="http://hortonworks.com/wp-content/uploads/downloads/2013/08/Hortonworks.CheatSheet.SQLtoHive.pdf" target="_blank">aide-mémoire SQL-Hive</a> utile.

Lors de l’envoi d’une requête Hive, vous pouvez également contrôler la destination de sa sortie : écran, fichier local sur le nœud principal ou blob Azure.

### Avec la console de ligne de commande Hadoop dans le nœud principal du cluster Hadoop

Une requête Hive complexe envoyée directement au nœud principal du cluster Hadoop est traitée plus rapidement qu’avec un éditeur Hive ou des scripts Azure PowerShell.

Connectez-vous au nœud principal du cluster Hadoop, ouvrez la ligne de commande Hadoop sur le bureau du nœud principal et saisissez la commande :

    cd %hive_home%\bin

Les utilisateurs disposent de trois possibilités pour envoyer des requêtes Hive dans la console de ligne de commande Hadoop :

* directement depuis la ligne de commande Hadoop ;
* à l’aide de fichiers HQL ;
* à l’aide de la console de commande Hive.

#### Envoyer des requêtes Hive directement depuis la ligne de commande Hadoop

Les utilisateurs peuvent exécuter une commande du type

	hive -e "<your hive query>;

pour envoyer des requêtes Hive simples directement dans la ligne de commande Hadoop. Voici un exemple, où l’encadré rouge indique la commande qui envoie la requête Hive et l’encadré vert, la sortie de la requête Hive.

![Create workspace](./media/machine-learning-data-science-process-hive-tables/run-hive-queries-1.png)

#### Envoyer des requêtes Hive dans des fichiers HQL

Lorsque la requête Hive est plus complexe et comporte plusieurs lignes, sa modification dans la ligne de commande Hadoop ou la console de commande Hive n’est pas pratique. L’alternative consiste à utiliser un éditeur de texte dans le nœud principal du cluster Hadoop pour enregistrer la requête Hive dans un fichier HQL situé dans un répertoire local du nœud principal. Ensuite, celle-ci peut être envoyée à l'aide de l'argument `-f` dans la commande `hive`, comme indiqué ci-dessous :

	`hive -f "<path to the .hql file>"`


#### Supprimer l’affichage de l’état d’avancement des requêtes Hive

Par défaut, après l’envoi d’une requête Hive dans la console de ligne de commande Hadoop, l’état d’avancement de la tâche Map/Reduce s’affiche à l’écran. Pour supprimer l'affichage de l'état d'avancement de la tâche Map/Reduce, déclarez l'argument `-S` (en respectant la casse) dans la ligne de commande, comme indiqué ci-dessous :

	hive -S -f "<path to the .hql file>"
	hive -S -e "<Hive queries>"

#### Envoyer des requêtes Hive dans la console de commande Hive

Les utilisateurs peuvent également ouvrir la console de commande Hive en exécutant la commande `hive` dans la ligne de commande Hadoop, puis ils peuvent envoyer les requêtes Hive depuis la console de commande Hive, à l'invite **hive>**. Voici un exemple.

![Create workspace](./media/machine-learning-data-science-process-hive-tables/run-hive-queries-2.png)

Ici, les deux encadrés rouges indiquent les commandes utilisées pour ouvrir la console de commande Hive et envoyer la requête Hive dans cette console. L’encadré vert montre la sortie de la requête Hive.

Les exemples précédents affichent directement les résultats de la requête à l’écran. Les utilisateurs peuvent également consigner la sortie dans un fichier local sur le nœud principal ou dans un blob Azure. Puis, ils peuvent utiliser d’autres outils pour analyser plus finement la sortie de la requête Hive.

#### Enregistrer les résultats d’une requête Hive dans un fichier local

Pour enregistrer les résultats d’une requête Hive dans un répertoire local du nœud principal, les utilisateurs doivent envoyer celle-ci dans la ligne de commande Hadoop, comme indiqué ci-dessous :

	`hive -e "<hive query>" > <local path in the head node>`


#### Enregistrer les résultats d’une requête Hive dans un blob Azure

Les utilisateurs peuvent également enregistrer les résultats d’une requête Hive dans un blob Azure situé dans le conteneur par défaut du cluster Hadoop. La requête Hive doit être similaire à celle-ci :

	insert overwrite directory wasb:///<directory within the default container> <select clause from ...>

Dans l'exemple suivant, la sortie de la requête Hive est consignée dans le répertoire de blob `queryoutputdir` situé dans le conteneur par défaut du cluster Hadoop. Ici, il suffit d’indiquer le nom du répertoire, sans celui du blob. Le fait de déclarer le nom du répertoire et celui du blob, comme dans *wasb:///queryoutputdir/queryoutput.txt*, déclenche une erreur.

![Create workspace](./media/machine-learning-data-science-process-hive-tables/output-hive-results-2.png)

Pour visualiser la sortie de la requête Hive dans le stockage de blob, ouvrez le conteneur par défaut du cluster Hadoop avec Azure Storage Explorer (ou un outil équivalent). Vous pouvez utiliser le filtre (encadré rouge) pour retrouver un blob dont le nom comporte les lettres spécifiées.

![Create workspace](./media/machine-learning-data-science-process-hive-tables/output-hive-results-3.png)

### Avec l’éditeur Hive ou des commandes Azure PowerShell

Les utilisateurs peuvent également utiliser Query Console (un éditeur Hive) en saisissant l’URL au format

*https://&#60;Hadoop cluster name>.azurehdinsight.net/Home/HiveEditor*

dans un navigateur Web. Notez que vous devrez entrer les informations d'identification du cluster Hadoop pour vous connecter.

Vous pouvez également [exécuter des requêtes Hive à l'aide de PowerShell](../hdinsight/hdinsight-hadoop-use-hive-powershell.md).


## <a name="create-tables"></a>Création de la base de données et des tables Hive

Les requêtes Hive sont disponibles en téléchargement dans le [référentiel Github](https://github.com/Azure/Azure-MachineLearning-DataScience/tree/master/Misc/DataScienceProcess/DataScienceScripts/sample_hive_create_db_tbls_load_data_generic.hql).

Voici la requête Hive qui crée une table Hive.

    create database if not exists <database name>;
	CREATE EXTERNAL TABLE if not exists <database name>.<table name>
	(
		field1 string, 
		field2 int, 
		field3 float, 
		field4 double, 
		...,
		fieldN string
	) 
	ROW FORMAT DELIMITED FIELDS TERMINATED BY '<field separator>' lines terminated by '<line separator>' 
	STORED AS TEXTFILE LOCATION '<storage location>' TBLPROPERTIES("skip.header.line.count"="1");

Voici les descriptions des champs que les utilisateurs doivent renseigner et d’autres opérations de configuration :

- **&#60;database name>** : nom de la base de données que les utilisateurs souhaitent créer. S'ils veulent utiliser la base de données par défaut, la requête *create database...* peut être omise. 
- **&#60;table name>** : nom de la table que les utilisateurs veulent créer dans la base de données spécifiée. S'ils veulent utiliser la base de données par défaut, la table peut être désignée directement par *&#60;table name>* sans &#60;database name>.
- **&#60;field separator>** : séparateur qui délimite les champs dans le fichier de données à charger dans la table Hive. 
- **&#60;line separator>** : séparateur qui délimite les lignes dans le fichier de données. 
- **&#60;storage location>** : emplacement Azure où enregistrer les données des tables Hive. Si les utilisateurs ne spécifient pas *LOCATION &#60;storage location>*, la base de données et les tables sont stockées dans le répertoire *hive/warehouse/* du conteneur par défaut du cluster Hive par défaut. Si un utilisateur souhaite spécifier l’emplacement de stockage, ce dernier doit se trouver dans le conteneur par défaut de la base de données et des tables. Cet emplacement doit être désigné comme emplacement relatif du conteneur par défaut du cluster au format *’wasb:///&#60;directory 1>/’* ou *’wasb:///&#60;directory 1>/&#60;directory 2>/’*, etc. Une fois la requête exécutée, les répertoires relatifs seront créés dans le conteneur par défaut. 
- **TBLPROPERTIES("skip.header.line.count"="1")** : si le fichier de données contient une ligne d'en-tête, les utilisateurs doivent ajouter cette propriété **à la fin** de la requête *create table*. Sinon, cette ligne d’en-tête est chargée comme un enregistrement dans la table. Si le fichier de données ne contient aucune ligne d’en-tête, cette configuration peut être omise dans la requête. 

## <a name="load-data"></a>Chargement des données dans des tables Hive
Voici la requête Hive qui charge les données dans une table Hive.

    LOAD DATA INPATH '<path to blob data>' INTO TABLE <database name>.<table name>;

- **&#60;path to blob data>** : si le fichier blob à charger dans la table Hive se trouve dans le conteneur par défaut du cluster Hadoop HDInsight, le chemin *&#60;path to blob data>* doit être au format *’wasb:///&#60;directory in this container>/&#60;blob file name>’*. Le fichier blob peut également se trouver dans un autre conteneur du cluster Hadoop HDInsight. Dans ce cas, *&#60;path to blob data>* doit être au format *’wasb://&#60;container name>@&#60;storage account name>.blob.core.windows.net/&#60;blob file name>’*.

	>[AZURE.NOTE] Les données blob à charger dans la table Hive doivent se trouver dans le conteneur par défaut ou un autre conteneur du compte de stockage du cluster Hadoop. Sinon, la requête *LOAD DATA* ne pourra pas s'exécuter car elle n'aura pas accès aux données.


## <a name="partition-orc"></a>Rubriques avancées : Table partitionnée et Stocker des données Hive au format ORC

Si les données sont volumineuses, le partitionnement de la table est avantageux pour les requêtes qui doivent n’en balayer que quelques partitions. Par exemple, il est raisonnable de partitionner les données journalisées d’un site Web par dates.

Outre le partitionnement des tables Hive, il est également judicieux de stocker les données Hive au format ORC (Optimized Row Columnar). Pour plus d'informations sur le format ORC, consultez l'article <a href="https://cwiki.apache.org/confluence/display/Hive/LanguageManual+ORC#LanguageManualORC-ORCFiles" target="_blank">L'utilisation de fichiers ORC améliore les performances lorsque Hive lit, écrit et traite des données</a>.

### Table partitionnée
Voici la requête Hive qui crée une table partitionnée et charge les données dans celle-ci.

    CREATE EXTERNAL TABLE IF NOT EXISTS <database name>.<table name>
	(field1 string,
	...
	fieldN string
	)
    PARTITIONED BY (<partitionfieldname> vartype) ROW FORMAT DELIMITED FIELDS TERMINATED BY '<field separator>'
		 lines terminated by '<line separator>' TBLPROPERTIES("skip.header.line.count"="1");
	LOAD DATA INPATH '<path to the source file>' INTO TABLE <database name>.<partitioned table name> 
		PARTITION (<partitionfieldname>=<partitionfieldvalue>);

Lors de l'interrogation de tables partitionnées, il est recommandé d'ajouter la condition de partition au **début** de la clause `where`, car cela améliore considérablement l'efficacité de la recherche.

    select 
		field1, field2, ..., fieldN
	from <database name>.<partitioned table name> 
	where <partitionfieldname>=<partitionfieldvalue> and ...;

### <a name="orc"></a>Stocker des données Hive au format ORC

Les utilisateurs ne peuvent pas charger directement des données au format ORC depuis le stockage blob dans des tables Hive. Voici les étapes que les utilisateurs doivent suivre pour charger des données au format ORC depuis des blobs Azure dans des tables Hive.

1. Créez une table externe **STORED AS TEXTFILE** et chargez les données du stockage blob dedans.

		CREATE EXTERNAL TABLE IF NOT EXISTS <database name>.<external textfile table name>
		(
			field1 string,
			field2 int,
			...
			fieldN date
		)
		ROW FORMAT DELIMITED FIELDS TERMINATED BY '<field separator>' 
			lines terminated by '<line separator>' STORED AS TEXTFILE 
			LOCATION 'wasb:///<directory in Azure blob>' TBLPROPERTIES("skip.header.line.count"="1");

		LOAD DATA INPATH '<path to the source file>' INTO TABLE <database name>.<table name>;

2. Créez une table interne avec le même schéma que la table externe à l’étape 1 et le même délimiteur de champ. Puis, stockez-y les données Hive au format ORC.

		CREATE TABLE IF NOT EXISTS <database name>.<ORC table name> 
		(
			field1 string,
			field2 int,
			...
			fieldN date
		) 
		ROW FORMAT DELIMITED FIELDS TERMINATED BY '<field separator>' STORED AS ORC;

3. Sélectionnez les données de la table externe à l’étape 1 et insérez-les dans la table ORC.

		INSERT OVERWRITE TABLE <database name>.<ORC table name>
            SELECT * FROM <database name>.<external textfile table name>;

	>[AZURE.NOTE] Si la table TEXTFILE *&#60;database name>.&#60;external textfile table name>* a des partitions, à l'Étape 3, la commande `SELECT * FROM <database name>.<external textfile table name>` sélectionne la variable de partition comme champ dans le jeu de données renvoyé. Le fait de l'insérer dans *&#60;database name>.&#60;ORC table name>* échouera car *&#60;database name>.&#60;ORC table name>* ne dispose pas de la variable de partition comme champ dans le schéma de la table. Dans ce cas, les utilisateurs doivent sélectionner explicitement les champs à insérer dans *&#60;database name>.&#60;ORC table name>* comme suit :

		INSERT OVERWRITE TABLE <database name>.<ORC table name> PARTITION (<partition variable>=<partition value>)
		   SELECT field1, field2, ..., fieldN
		   FROM <database name>.<external textfile table name> 
		   WHERE <partition variable>=<partition value>;

4. Pour plus de sécurité, lorsque vous utilisez la requête suivante, il est recommandé de déplacer la table *&#60;external textfile table name>* une fois toutes les données insérées dans la table *&#60;database name>.&#60;ORC table name>* :

		DROP TABLE IF EXISTS <database name>.<external textfile table name>;

À l’issue de cette procédure, vous devez obtenir une table immédiatement exploitable et contenant des données au format ORC.


##Le paramétrage des sections doit être placé ici

La section finale présente les paramètres que les utilisateurs peuvent ajuster pour accélérer le traitement des requêtes Hive.

<!-------HONumber=AcomDC_0211_2016-->