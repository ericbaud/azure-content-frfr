<properties
   pageTitle="Intégration de Data Lake Store à d'autres services Azure | Azure"
   description="Présentation de l'intégration de Data Lake Store à d'autres services Azure"
   documentationCenter=""
   services="data-lake-store" 
   authors="nitinme"
   manager="paulettm"
   editor="cgronlun"/>

<tags
   ms.service="data-lake-store"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="big-data"
   ms.date="01/04/2016"
   ms.author="nitinme"/>

# Intégration de Data Lake Store à d'autres services Azure

Azure Data Lake Store peut être utilisé conjointement avec d'autres services Azure afin d'augmenter le nombre de scénarios. L'article suivant liste les services auxquels Data Lake Store peut être intégré.

## Utiliser Data Lake Store avec Azure HDInsight

Vous pouvez approvisionner un cluster [Azure HDInsight](https://azure.microsoft.com/documentation/learning-paths/hdinsight-self-guided-hadoop-training/) qui utilise Data Lake Store comme stockage compatible HDFS. Pour cette version, pour les clusters Hadoop et Storm sous Windows et Linux, vous ne pouvez utiliser Data Lake Store que comme stockage supplémentaire. Ces clusters utilisent toujours Azure Storage (WASB) comme stockage par défaut. Toutefois, pour les clusters HBase sous Windows et Linux, vous pouvez utiliser Data Lake Store comme stockage par défaut, comme stockage supplémentaire ou les deux.

Pour savoir comment approvisionner un cluster HDInsight avec Data Lake Store, consultez :

* [Approvisionner un cluster HDInsight avec Data Lake Store à l’aide du portail Azure](data-lake-store-hdinsight-hadoop-use-portal.md)
* [Approvisionner un cluster HDInsight avec Data Lake Store à l'aide d'Azure PowerShell](data-lake-store-hdinsight-hadoop-use-powershell.md)


## Utiliser Data Lake Store avec Azure Data Lake Analytics

[Azure Data Lake Analytics](data-lake-analytics/data-lake-analytics-overview.md) vous permet de travailler avec le Big Data à l'échelle du cloud. Il approvisionne dynamiquement des ressources et vous permet d'effectuer des analyses sur des téraoctets, voire des exaoctets, de données stockées dans plusieurs sources de données prises en charge, parmi lesquelles Data Lake Store. Data Lake Analytics est spécialement optimisé pour fonctionner avec Azure Data Lake Store, fournissant ainsi le plus haut niveau de performances, de débit et de parallélisation pour vos charges de travail de Big Data.

Pour obtenir des instructions sur l'utilisation de Data Lake Analytics avec Data Lake Store, consultez [Prise en main de Data Lake Analytics avec Data Lake Store](data-lake-analytics/data-lake-analytics-get-started-portal.md).


## Utiliser Data Lake Store avec Azure Data Factory

Vous pouvez utiliser [Azure Data Factory](https://azure.microsoft.com/services/data-factory/) pour recevoir des données à partir de tables Azure, de bases de données SQL Azure, d'Azure SQL DataWarehouse, d'objets Blob Azure Storage et de bases de données locales. Jouissant d'un statut de premier ordre dans l'écosystème Azure, Azure Data Factory peut être utilisé pour orchestrer l'ingestion de données de ces sources vers Azure Data Lake Store.

Pour obtenir des instructions sur l'utilisation d'Azure Data Factory avec Data Lake Store, consultez [Déplacer des données vers et depuis Data Lake Store avec Data Factory](data-factory/data-factory-azure-datalake-connector.md).

## Copier des données d’objets blob Azure Storage dans Data Lake Store

Azure Data Lake Store fournit un outil en ligne de commande, AdlCopy, qui vous permet de copier les données d’objets blob Azure Storage vers un compte Data Lake Store. Pour plus d’informations, consultez [Copier des données d’objets blob Azure Storage vers Data Lake Store](data-lake-store-copy-data-azure-storage-blob.md).


## Voir aussi

- [Présentation d'Azure Data Lake Store](data-lake-store-overview.md)
- [Prise en main de Data Lake Store avec le portail](data-lake-store-get-started-portal.md)
- [Prise en main de Data Lake Store avec PowerShell](data-lake-store-get-started-powershell.md)  

<!---HONumber=AcomDC_0107_2016-->