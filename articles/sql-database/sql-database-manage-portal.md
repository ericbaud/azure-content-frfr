<properties
	pageTitle="Gérer les bases de données SQL Azure au moyen du portail Azure Classic"
	description="Découvrez comment utiliser le portail Azure Classic pour gérer une base de données relationnelle dans le cloud."
	services="sql-database"
	documentationCenter=""
	authors="stevestein"
	manager="jeffreyg"
	editor=""/>

<tags
	ms.service="sql-database"
	ms.devlang="NA"
	ms.workload="data-management"
	ms.topic="article"
	ms.tgt_pltfrm="NA"
	ms.date="01/22/2016"
	ms.author="sstein"/>


# Gestion des bases de données SQL Azure au moyen du portail Azure


> [AZURE.SELECTOR]
- [Portail Azure](sql-database-manage-portal.md)
- [SSMS](sql-database-manage-azure-ssms.md)
- [PowerShell](sql-database-command-line-tools.md)

Le [portail Azure](https://portal.azure.com/) permet de créer, de surveiller et de gérer des serveurs logiques et des bases de données SQL Azure. Cet article présente certaines des tâches les plus courantes.

![Vue d’ensemble des bases de données](./media/sql-database-manage-portal/sqldatabase_annotated.png)

## 1\. Actions de gestion des bases de données
![Actions de gestion des bases de données](./media/sql-database-manage-portal/sqldatabase_actions.png)

Le portail Azure fournit un ensemble d’actions communes à exécuter sur les bases de données, accessibles dans la partie supérieure d’un panneau de base de données. Vous pouvez restaurer une base de données à un moment donné, l’ouvrir dans Visual Studio, la copier vers un nouveau serveur et l’exporter vers un compte Microsoft Azure Storage.

- [Restauration d’une base de données SQL](sql-database-user-error-recovery.md)
- [Ouvrir une base de données SQL dans Visual Studio](sql-database-connect-query.md)
- [Exporter une base de données SQL](sql-database-export.md)

## 2\. Surveillance des bases de données
![Surveillance des bases de données](./media/sql-database-manage-portal/sqldatabase_monitoring.png)

Par défaut, les bases de données SQL Microsoft Azure proposent des graphiques de surveillance des unités de transaction de base de données (DTU), de la taille des bases de données et de l’intégrité des connexions. Ces graphiques de surveillance peuvent être personnalisés et étendus afin d’inclure le pourcentage de CPU, le pourcentage d’E/S des données, les blocages, le pourcentage d’E/S des fichiers journaux, voire le pourcentage des demandes bloquées par le pare-feu. Pour en savoir plus sur la personnalisation des graphiques de surveillance, cliquez [ici][Azure part monitoring].

En outre, des règles d’alerte peuvent être configurées pour surveiller une métrique spécifique et signaler à un administrateur ou à coadministrateur désigné qu’un seuil prédéfini a été atteint. Pour plus d’informations sur la configuration des règles d’alertes dans le portail Azure Classic, consultez [ce lien][Azure part monitoring].

## 3\. Audit et sécurité des bases de données
![Sécurité des bases de données](./media/sql-database-manage-portal/sqldatabase_security.png)

Vous pouvez configurer les bases de données SQL Microsoft Azure pour effectuer le suivi de l’ensemble des événements des bases de données et les écrire dans un fichier journal d’audit, dans votre compte Microsoft Azure Storage. Cette fonction peut vous aider à assurer la conformité réglementaire des bases de données, à identifier leurs activités et à obtenir des informations sur les écarts susceptibles de signaler des problèmes liés à l’activité ou des violations de sécurité éventuelles.

- [Audit de base de données SQL](sql-database-auditing-get-started.md)

Il est également possible de configurer les bases de données SQL Azure de manière à masquer les données sensibles aux utilisateurs non autorisés.

- [Dynamic Data Masking (masquage des données dynamiques)](sql-database-dynamic-data-masking-get-started.md)


## 4\. Géo-réplication
![Géo-réplication](./media/sql-database-manage-portal/sqldatabase_georeplication.png)

Vous pouvez configurer les bases de données SQL Microsoft Azure pour répliquer de manière asynchrone les transactions validées vers une base de données secondaire. La fonction de géo-réplication du portail Azure permet de sélectionner la région Azure dans laquelle vous voulez que la base de données secondaire réside.

- [Utiliser la géo-réplication](https://msdn.microsoft.com/library/azure/dn783447.aspx)





##Ressources supplémentaires
* [Base de données SQL](sql-database-technical-overview.md)   
* [Contrôle de la base de données SQL à l’aide de vues de gestion dynamiques][]   
* [Référence Transact-SQL (base de données SQL)][]

  [Azure Classic Portal Tour]: https://go.microsoft.com/fwlink/?LinkID=522341
  [Classic Portal]: https://portal.azure.com
  [Azure part monitoring]: ../documentdb-monitor-accounts.md
  [AzureDb management overview]: http://azure.microsoft.com/blog/2014/12/22/client-tooling-updates-for-azure-sql-database/
  [Introducing SQL Database]: http://azure.microsoft.com/services/sql-database
  [Database geo-replication]: http://azure.microsoft.com/blog/2014/07/12/spotlight-on-sql-database-active-geo-replication/
  [Managing Azure SQL Database using SQL Server Management Studio]: sql-database-manage-azure-ssms.md
  [Contrôle de la base de données SQL à l’aide de vues de gestion dynamiques]: http://msdn.microsoft.com/library/windowsazure/ff394114.aspx
  [Référence Transact-SQL (base de données SQL)]: http://msdn.microsoft.com/library/bb510741(v=sql.120).aspx
  [AzureDb Auditing]: http://azure.microsoft.com/documentation/articles/sql-database-auditing-get-started/
  [AzureDb datamasking]: http://azure.microsoft.com/documentation/articles/sql-database-dynamic-data-masking-get-started/

<!---HONumber=AcomDC_0302_2016-->