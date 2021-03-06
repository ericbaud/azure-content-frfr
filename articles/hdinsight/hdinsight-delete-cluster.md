<properties
pageTitle="Suppression d’un cluster HDInsight | Azure"
description="Informations sur les différentes méthodes que vous pouvez utiliser pour supprimer un cluster HDInsight."
services="hdinsight"
documentationCenter=""
authors="Blackmist"
manager="paulettm"
editor="cgronlun"/>

<tags
ms.service="hdinsight"
ms.devlang="na"
ms.topic="article"
ms.tgt_pltfrm="na"
ms.workload="big-data"
ms.date="03/07/2016"
ms.author="larryfr"/>

#Suppression d’un cluster HDInsight

Les clusters HDInsight sont facturés à l’heure. Vous devez donc toujours supprimer votre cluster lorsqu’il n’est plus utilisé. Dans ce document, vous allez apprendre à supprimer un cluster à l’aide du portail Azure, d’Azure PowerShell et de l’interface de ligne de commande (CLI) Azure.

> [AZURE.IMPORTANT] La suppression d’un cluster HDInsight ne supprime pas les comptes de stockage Azure associés au cluster. Cela vous permet de conserver et de réutiliser les données stockées par le cluster.

##Portail Azure

1. Connectez-vous au [portail Azure](https://portal.azure.com) et sélectionnez votre cluster HDInsight. Si votre cluster HDInsight n’est pas épinglé au tableau de bord, vous pouvez le rechercher par son nom dans le champ de recherche (icône en forme de loupe) sur le côté droit de la barre de navigation.

    ![recherche dans le portail](./media/hdinsight-delete-cluster/navbar.png)

2. Lorsque le panneau s’ouvre pour le cluster, sélectionnez l’icône __Supprimer__. Lorsque vous y êtes invité, sélectionnez __Oui__ pour supprimer le cluster.

    ![icône Supprimer](./media/hdinsight-delete-cluster/deletecluster.png)

##Azure PowerShell

> [AZURE.NOTE] Si vous n’avez pas installé, ni configuré Azure PowerShell, utilisez les étapes du document [Installer et configurer Azure PowerShell](../powershell-install-configure.md).

À partir d’une invite PowerShell, utilisez la commande suivante pour supprimer le cluster :

    Remove-AzureRmHDInsightCluster -ClusterName CLUSTERNAME

Remplacez __CLUSTERNAME__ par le nom de votre cluster HDInsight.

##Interface de ligne de commande Azure

> [AZURE.NOTE] Si vous n’avez pas installé, ni configuré l’interface de ligne de commande (CLI) Azure, utilisez les étapes du document [Installer et configurer l’interface de ligne de commande Azure](../xplat-cli-install.md).

À partir d’une invite, utilisez la commande suivante pour supprimer le cluster :

    azure hdinsight cluster delete CLUSTERNAME
    
Remplacez __CLUSTERNAME__ par le nom de votre cluster HDInsight.

<!---------HONumber=AcomDC_0309_2016-->