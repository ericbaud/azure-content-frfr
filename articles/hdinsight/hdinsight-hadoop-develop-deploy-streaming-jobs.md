
<properties
	pageTitle="Développement de programmes de diffusion en continu Hadoop en C# pour HDInsight | Microsoft Azure"
	description="Découvrez comment développer des programmes MapReduce de diffusion en continu Hadoop en C# et les déployer sur Azure HDInsight."
	services="hdinsight"
	documentationCenter=""
	tags="azure-portal"
	authors="mumian"
	manager="paulettm"
	editor="cgronlun"/>

<tags
	ms.service="hdinsight"
	ms.workload="big-data"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="01/28/2016"
	ms.author="jgao"/>



# Développement de programmes de diffusion en continu Hadoop en C# pour HDInsight

Hadoop fournit une API de diffusion en continu pour MapReduce qui vous permet d'écrire des fonctions de mappage et de réduction dans d'autres langages que Java. Ce didacticiel vous familiarise avec la création d’un programme de comptage de mots en C#, qui compte les occurrences d’un mot spécifique dans les données d’entrée que vous fournissez. L’illustration suivante indique comment l’infrastructure MapReduce effectue un comptage des mots :

![HDI.WordCountDiagram][image-hdi-wordcountdiagram]

> [AZURE.NOTE] Les étapes de cet article s’appliquent uniquement aux clusters Azure HDInsight basés sur Windows. Pour obtenir un exemple de diffusion en continu pour HDInsight basé sur Linux, consultez la page [Développement de programmes de diffusion en continu Python pour HDInsight](hdinsight-hadoop-streaming-python.md).

Ce didacticiel vous explique les procédures suivantes :

- Développement d’un programme MapReduce de diffusion en continu Hadoop en utilisant C# 
- Exécution du programme MapReduce sur Azure HDInsight
- Extraction des résultats de la tâche MapReduce

###Configuration requise

Avant de commencer ce didacticiel, vous devez avoir effectué les tâches suivantes :

- Une station de travail avec [Azure PowerShell][powershell-install] et [Microsoft Visual Studio](https://www.visualstudio.com/).
- Obtenez un abonnement Azure. Pour obtenir des instructions, consultez les pages traitant des [options d'achat][azure-purchase-options], des [offres spéciales membres][azure-member-offers] ou de la [version d'évaluation gratuite][azure-free-trial].


##Développement d’un programme de diffusion en continu Hadoop pour le comptage de mots en C&#35;

La solution de comptage de mots contient deux projets d’application console : mappeur et raccord de réduction. L’application mappeur diffuse chaque mot dans la console et l’application raccord de réduction compte le nombre de mots diffusés depuis un document. Le mappeur et le raccord de réduction peuvent tous les deux lire les caractères à partir du flux d’entrée standard (stdin), puis écrire dans le flux de sortie standard (stdout).

**Pour créer le programme mappeur**

1. Ouvrez Visual Studio et créez une application console C# nommée **WordCountMapper**.
2. Dans l'Explorateur de solutions, renommez **Program.cs** en **WordCountMapper.cs**. Cliquez sur **Oui** pour confirmer les changement de noms de toutes les références.
3. Remplacez le code dans WordCountMapper.cs par ce qui suit :

        using System;
        using System.IO;

        namespace WordCountMapper
        {
            class WordCountMapper
            {
                static void Main(string[] args)
                {
                    if (args.Length > 0)
                    {
                        Console.SetIn(new StreamReader(args[0]));
                    }

                    string line;
                    string[] words;

                    while ((line = Console.ReadLine()) != null)
                    {
                        words = line.Split(' ');

                        foreach (string word in words)
                            Console.WriteLine(word.ToLower());
                    }
                }
            }
        }

4. Générez la solution et assurez-vous qu'il n'existe aucune erreur de compilation.

**Pour créer le programme raccord de réduction**

1. Ajoutez une autre application console C# à la solution appelée **WordCountReducer**". Emplacement|C:\\Tutorials\\WordCount
2. Dans l'Explorateur de solutions, renommez **Program.cs** en **WordCountReducer.cs**. Cliquez sur **Oui** pour confirmer les changement de noms de toutes les références.
3. Remplacez le code dans WordCountReducer.cs par ce qui suit :

        using System;
        using System.IO;

        namespace WordCountReducer
        {
            class WordCountReducer
            {
                static void Main(string[] args)
                {
                    string word, lastWord = null;
                    int count = 0;

                    if (args.Length > 0)
                    {
                        Console.SetIn(new StreamReader(args[0]));
                    }

                    while ((word = Console.ReadLine()) != null)
                    {
                        if (word != lastWord)
                        {
                            if (lastWord != null)
                                Console.WriteLine("{0}[{1}]", lastWord, count);

                            count = 1;
                            lastWord = word;
                        }
                        else
                        {
                            count += 1;
                        }
                    }
                    Console.WriteLine(count);
                }
            }
        }

4. Générez la solution et assurez-vous qu'il n'existe aucune erreur de compilation.

Vous devriez obtenir les exécutables de mappeur et du raccord de réduction :

- ..\\WordCountMapper\\bin\\Debug\\WordCountMapper.exe
- ..\\WordCountReducer\\bin\\Debug\\WordCountReducer.exe


##Téléchargement de données vers le stockage d'objets blob Azure
Azure HDInsight utilise le stockage d'objets blob Azure comme système de fichiers par défaut. Vous pouvez configurer un cluster HDInsight pour utiliser un autre stockage d'objets blob pour les fichiers de données. Dans cette section, vous allez créer un compte de stockage Azure, puis télécharger les fichiers de données vers le stockage d’objets blob. Les fichiers de données sont des fichiers .txt dans le répertoire %hadoop\_home%\\share\\doc\\hadoop\\common.


**Pour créer un compte de stockage et un conteneur**

1. Ouvrez Azure PowerShell.
2. Définissez les variables, puis exécutez ces commandes :

		$subscriptionName = "<AzureSubscriptionName>"
		$storageAccountName = "<AzureStorageAccountName>"  
		$containerName = "<ContainerName>"
		$location = "<MicrosoftDataCenter>"  # For example, "East US"

3. Exécutez les commandes suivantes pour créer un compte de stockage et un conteneur de stockage d’objets blob sur le compte :

		# Select an Azure subscription
		Select-AzureSubscription $subscriptionName

		# Create a Storage account
		New-AzureStorageAccount -StorageAccountName $storageAccountName -location $location

		# Create a Blob storage container
		$storageAccountKey = Get-AzureStorageKey $storageAccountName | %{ $_.Primary }
		$destContext = New-AzureStorageContext –StorageAccountName $storageAccountName –StorageAccountKey $storageAccountKey  
		New-AzureStorageContainer -Name $containerName -Context $destContext

4. Exécutez les commandes suivantes pour vérifier le compte de stockage et le conteneur :

		Get-AzureStorageAccount -StorageAccountName $storageAccountName
		Get-AzureStorageContainer -Context $destContext

**Pour télécharger les fichiers de données**

1. Dans la fenêtre Azure PowerShell, définissez les valeurs pour le dossier local et les dossiers de destination :

		$localFolder = "C:\hdp\hadoop-2.4.0.2.1.3.0-1981\share\doc\hadoop\common"
		$destFolder = "WordCount/Input"

	Notez que le dossier des fichiers sources locaux est **C:\\hdp\\hadoop-2.4.0.2.1.3.0-1981\\share\\doc\\hadoop\\common** et que le dossier de destination est **WordCount/Input**. L'emplacement source correspond à l'emplacement des fichiers .txt sur l'émulateur HDInsight. La destination est la structure de dossiers qui sera reflétée sous le conteneur d'objets blob Azure.

3. Exécutez les commandes suivantes pour obtenir une liste des fichiers .txt contenus dans le dossier des fichiers sources :

		# Get a list of the .txt files
		$filesAll = Get-ChildItem $localFolder
		$filesTxt = $filesAll | where {$_.Extension -eq ".txt"}

5. Exécutez l'extrait de code suivant pour copier les fichiers :

		# Copy the files from the local workstation to the Blob container
		foreach ($file in $filesTxt){

		    $fileName = "$localFolder\$file"
		    $blobName = "$destFolder/$file"

		    write-host "Copying $fileName to $blobName"

		    Set-AzureStorageBlobContent -File $fileName -Container $containerName -Blob $blobName -Context $destContext
		}

6. Utilisez la commande suivante pour répertorier les fichiers téléchargés :

		# List the uploaded files in the Blob storage container
		Get-AzureStorageBlob -Container $containerName  -Context $destContext -Prefix $destFolder


**Pour télécharger des applications de comptage de mots**

1. Dans la fenêtre Azure PowerShell, définissez les variables suivantes :

		$mapperFile = "C:\Tutorials\WordCount\WordCountMapper\bin\Debug\WordCountMapper.exe"
		$reducerFile = "C:\Tutorials\WordCount\WordCountReducer\bin\Debug\WordCountReducer.exe"
		$blobFolder = "WordCount/Apps"

	Notez que le dossier de destination est **WordCount/Apps**, ce qui correspond à la structure qui sera reflétée dans le conteneur d'objets blob Azure.

2. Exécutez les commandes suivantes pour copier les applications :

		Set-AzureStorageBlobContent -File $mapperFile -Container $containerName -Blob "$blobFolder/WordCountMapper.exe" -Context $destContext
		Set-AzureStorageBlobContent -File $reducerFile -Container $containerName -Blob "$blobFolder/WordCountReducer.exe" -Context $destContext

3. Utilisez la commande suivante pour répertorier les fichiers téléchargés :

		# List the uploaded files in the Blob storage container
		Get-AzureStorageBlob -Container $containerName  -Context $destContext -Prefix $blobFolder

	Les deux fichiers d'application doivent être répertoriés ici.


##Exécution du programme MapReduce sur Azure HDInsight

Cette section fournit un script Azure PowerShell qui effectue toutes les tâches liées à l’exécution d’une tâche MapReduce. Liste des tâches incluses :

1. Approvisionnement d'un cluster HDInsight

	1. Création d’un compte de stockage utilisé en tant que système de fichiers de cluster HDInsight par défaut
	2. Création d'un conteneur d'objets blob
	3. Création d'un cluster HDInsight

2. Envoi de la tâche MapReduce

	1. Création d'une définition de tâche MapReduce de diffusion en continu
	2. Envoi d'une tâche MapReduce
	3. Attente de l’arrêt du travail
	4. Affichage de l'erreur standard
	5. Affichage du résultat standard

3. Suppression du cluster

	1. Suppression du cluster HDInsight
	2. Suppression du compte de stockage utilisé en tant que système de fichiers du cluster HDInsight


**Pour exécuter le script Azure PowerShell**

1. Ouvrez le Bloc-notes.
2. Copiez et collez le code suivant :

		# ====== STORAGE ACCOUNT AND HDINSIGHT CLUSTER VARIABLES ======
		$subscriptionName = "<AzureSubscriptionName>"
		$stringPrefix = "<StringForPrefix>"     ### Prefix to cluster, Storage account, and container names
		$storageAccountName_Data = "<TheDataStorageAccountName>"
		$containerName_Data = "<TheDataBlobStorageContainerName>"
		$location = "<MicrosoftDataCenter>"     ### Must match the data storage account location
		$clusterNodes = 1

		$clusterName = $stringPrefix + "hdicluster"

		$storageAccountName_Default = $stringPrefix + "hdistore"
		$containerName_Default =  $stringPrefix + "hdicluster"

		# ====== THE STREAMING MAPREDUCE JOB VARIABLES ======
		$mrMapper = "WordCountMapper.exe"
		$mrReducer = "WordCountReducer.exe"
		$mrMapperFile = "wasb://$containerName_Data@$storageAccountName_Data.blob.core.windows.net/WordCount/Apps/WordCountMapper.exe"
		$mrReducerFile = "wasb://$containerName_Data@$storageAccountName_Data.blob.core.windows.net/WordCount/Apps/WordCountReducer.exe"
		$mrInput = "wasb://$containerName_Data@$storageAccountName_Data.blob.core.windows.net/WordCount/Input/"
		$mrOutput = "wasb://$containerName_Data@$storageAccountName_Data.blob.core.windows.net/WordCount/Output/"
		$mrStatusOutput = "wasb://$containerName_Data@$storageAccountName_Data.blob.core.windows.net/WordCount/MRStatusOutput/"

		Select-AzureSubscription $subscriptionName

		#====== CREATE A STORAGE ACCOUNT ======
		Write-Host "Create a storage account" -ForegroundColor Green
		New-AzureStorageAccount -StorageAccountName $storageAccountName_Default -location $location

		#====== CREATE A BLOB STORAGE CONTAINER ======
		Write-Host "Create a Blob storage container" -ForegroundColor Green
		$storageAccountKey_Default = Get-AzureStorageKey $storageAccountName_Default | %{ $_.Primary }
		$destContext = New-AzureStorageContext –StorageAccountName $storageAccountName_Default –StorageAccountKey $storageAccountKey_Default

		New-AzureStorageContainer -Name $containerName_Default -Context $destContext

		#====== CREATE AN HDINSIGHT CLUSTER ======
		Write-Host "Create an HDInsight cluster" -ForegroundColor Green
		$storageAccountKey_Data = Get-AzureStorageKey $storageAccountName_Data | %{ $_.Primary }

		$config = New-AzureHDInsightClusterConfig -ClusterSizeInNodes $clusterNodes |
		    Set-AzureHDInsightDefaultStorage -StorageAccountName "$storageAccountName_Default.blob.core.windows.net" -StorageAccountKey $storageAccountKey_Default -StorageContainerName $containerName_Default |
		    Add-AzureHDInsightStorage -StorageAccountName "$storageAccountName_Data.blob.core.windows.net" -StorageAccountKey $storageAccountKey_Data

		Select-AzureSubscription $subscriptionName
		New-AzureHDInsightCluster -Name $clusterName -Location $location -Config $config

		#====== CREATE A STREAMING MAPREDUCE JOB DEFINITION ======
		Write-Host "Create a streaming MapReduce job definition" -ForegroundColor Green

		$mrJobDef = New-AzureHDInsightStreamingMapReduceJobDefinition -JobName mrWordCountStreamingJob -StatusFolder $mrStatusOutput -Mapper $mrMapper -Reducer $mrReducer -InputPath $mrInput -OutputPath $mrOutput
		$mrJobDef.Files.Add($mrMapperFile)
		$mrJobDef.Files.Add($mrReducerFile)

		#====== RUN A STREAMING MAPREDUCE JOB ======
		Write-Host "Run a streaming MapReduce job" -ForegroundColor Green
		Select-AzureSubscription $subscriptionName
		$mrJob = Start-AzureHDInsightJob -Cluster $clusterName -JobDefinition $mrJobDef
		Wait-AzureHDInsightJob -Job $mrJob -WaitTimeoutInSeconds 3600

		Get-AzureHDInsightJobOutput -Cluster $clusterName -JobId $mrJob.JobId -StandardError
		Get-AzureHDInsightJobOutput -Cluster $clusterName -JobId $mrJob.JobId -StandardOutput

		#====== DELETE THE HDINSIGHT CLUSTER ======
		Write-Host "Delete the HDInsight cluster" -ForegroundColor Green
		Select-AzureSubscription $subscriptionName
		Remove-AzureHDInsightCluster -Name $clusterName

		#====== DELETE THE STORAGE ACCOUNT ======
		Write-Host "Delete the storage account" -ForegroundColor Green
		Remove-AzureStorageAccount -StorageAccountName $storageAccountName_Default

3. Définissez les quatre premières variables du script. La variable **$stringPrefix** permet d’ajouter la chaîne spécifiée sous forme de préfixe au nom du cluster HDInsight, au nom du compte de stockage et au nom du conteneur de stockage d’objets blob. Étant donné que ces noms doivent comprendre entre 3 et 24 caractères, vérifiez que la chaîne que vous spécifiez et que les noms utilisés par ce script n’excèdent pas ensemble la limite de caractères. Vous devez uniquement utiliser des minuscules pour **$stringPrefix**.

	Les variables **$storageAccountName\_Data** et **$containerName\_Data** correspondent au compte de stockage et au conteneur que vous avez déjà créés aux étapes précédentes. Vous devez dès lors fournir les noms de ceux-ci. Ils sont utilisés pour le stockage des fichiers de données et des applications. La variable **$location** doit correspondre à l’emplacement du compte de stockage des données.

4. Consultez le reste des variables.
5. Enregistrez le fichier de script.
6. Ouvrez Azure PowerShell.
7. Exécutez la commande suivante pour définir la stratégie d’exécution sur RemoteSigned :

		PowerShell -File <FileName> -ExecutionPolicy RemoteSigned

8. Lorsque vous y êtes invité, entrez le nom d’utilisateur et le mot de passe du cluster HDInsight. Veillez à ce que le mot de passe contienne au moins 10 caractères, dont une lettre majuscule, une lettre minuscule, un chiffre et un caractère spécial. Si vous ne voulez pas être invité à saisir les informations d’identification, consultez la page [Utilisation des mots de passe, des chaînes sécurisées et des informations d’identification dans Windows PowerShell][powershell-PSCredential].

Pour accéder à un exemple de Kit de développement logiciel (SDK) .NET HDInsight lors de l’envoi de tâches de diffusion en continu Hadoop, consultez la rubrique [Envoi de tâches Hadoop par programme][hdinsight-submit-jobs].


##Extraction du résultat de la tâche MapReduce
Cette section montre comment télécharger et afficher le résultat. Pour obtenir des informations sur l’affichage des résultats dans Excel, consultez les rubriques [Connexion d’Excel à HDInsight avec le pilote ODBC Microsoft Hive][hdinsight-ODBC] et [Connexion d’Excel à HDInsight avec Power Query][hdinsight-power-query].


**Pour récupérer le résultat**

1. Ouvrez la fenêtre Azure PowerShell.
2. Définissez les valeurs, puis exécutez ces commandes :

		$subscriptionName = "<AzureSubscriptionName>"
		$storageAccountName = "<TheDataStorageAccountName>"
		$containerName = "<TheDataBlobStorageContainerName>"
		$blobName = "WordCount/Output/part-00000"

3. Exécutez les commandes suivantes pour créer un objet de contexte de stockage Azure :

		Select-AzureSubscription $subscriptionName
		$storageAccountKey = Get-AzureStorageKey $storageAccountName | %{ $_.Primary }
		$storageContext = New-AzureStorageContext –StorageAccountName $storageAccountName –StorageAccountKey $storageAccountKey  

4. Exécutez les commandes suivantes pour télécharger et afficher le résultat :

		Get-AzureStorageBlobContent -Container $ContainerName -Blob $blobName -Context $storageContext -Force
		cat "./$blobName" | findstr "there"



##Étapes suivantes
Dans ce didacticiel, vous avez appris à développer un travail MapReduce en Hadoop, à tester l’application sur l’émulateur HDInsight et à écrire un script Azure PowerShell pour approvisionner un cluster HDInsight et exécuter un travail MapReduce sur le cluster. Pour en savoir plus, consultez les articles suivants :

- [Prise en main d’Azure HDInsight](hdinsight-hadoop-linux-tutorial-get-started.md)
- [Prise en main de l’émulateur HDInsight][hdinsight-get-started-emulator]
- [Développement de programmes MapReduce en Java pour HDInsight][hdinsight-develop-mapreduce]
- [Utilisation du stockage d’objets blob Azure avec HDInsight][hdinsight-storage]
- [Administration de HDInsight à l’aide d’Azure PowerShell][hdinsight-admin-powershell]
- [Téléchargement de données vers HDInsight][hdinsight-upload-data]
- [Utilisation de Hive avec HDInsight][hdinsight-use-hive]
- [Utilisation de Pig avec HDInsight][hdinsight-use-pig]

[azure-purchase-options]: http://azure.microsoft.com/pricing/purchase-options/
[azure-member-offers]: http://azure.microsoft.com/pricing/member-offers/
[azure-free-trial]: http://azure.microsoft.com/pricing/free-trial/

[hdinsight-develop-mapreduce]: hdinsight-develop-deploy-java-mapreduce.md
[hdinsight-submit-jobs]: hdinsight-submit-hadoop-jobs-programmatically.md

[hdinsight-get-started-emulator]: ../hdinsight-get-started-emulator.md
[hdinsight-emulator-wasb]: ../hdinsight-get-started-emulator.md#blobstorage
[hdinsight-upload-data]: hdinsight-upload-data.md
[hdinsight-storage]: ../hdinsight-hadoop-use-blob-storage.md
[hdinsight-admin-powershell]: hdinsight-administer-use-powershell.md

[hdinsight-use-hive]: hdinsight-use-hive.md
[hdinsight-use-pig]: hdinsight-use-pig.md
[hdinsight-ODBC]: hdinsight-connect-excel-hive-ODBC-driver.md
[hdinsight-power-query]: hdinsight-connect-excel-power-query.md

[powershell-PSCredential]: http://social.technet.microsoft.com/wiki/contents/articles/4546.working-with-passwords-secure-strings-and-credentials-in-windows-powershell.aspx
[powershell-install]: ../powershell-install-configure.md

[image-hdi-wordcountdiagram]: ./media/hdinsight-hadoop-develop-deploy-streaming-jobs/HDI.WordCountDiagram.gif "Flux de l’application de comptage de mots MapReduce"

<!---HONumber=AcomDC_0218_2016-->