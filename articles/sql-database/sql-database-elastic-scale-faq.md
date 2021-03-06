<properties 
	pageTitle="FAQ sur l'infrastructure élastique d'Azure SQL | Microsoft Azure" 
	description="Frequently Asked Questions about Azure SQL Database Elastic Scale." 
	services="sql-database" 
	documentationCenter="" 
	manager="jeffreyg" 
	authors="sidneyh" 
	editor=""/>

<tags 
	ms.service="sql-database" 
	ms.workload="sql-database" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="02/01/2016" 
	ms.author="sidneyh"/>

# FAQ sur les outils de bases de données élastiques 

#### Si je possède un locataire unique par partition et aucune clé de partitionnement, comment remplir la clé de partitionnement pour les informations de schéma ?

L’objet d’informations de schéma est utilisé uniquement dans les scénarios de fractionnement/fusion. Si une application ne possède par nature qu’un seul locataire, elle n’a pas besoin de l’outil de fractionnement/fusion, et il n’est donc pas nécessaire de remplir l’objet d’informations de schéma.

#### J’ai configuré une base de données et dispose déjà d’un gestionnaire de cartes de partition, comment enregistrer cette nouvelle base de données en tant que partition ?

Consultez **[Ajout d’une partition à une application à l’aide de la bibliothèque cliente des bases de données élastiques](sql-database-elastic-scale-add-a-shard.md)**.

#### Combien coûtent les outils de bases de données élastiques ?

L’utilisation de la bibliothèque cliente des bases de données élastiques est gratuite. Les coûts sont uniquement liés à l’utilisation des bases de données Azure SQL pour les partitions et le gestionnaire des mappages de partition, ainsi qu’aux rôles web/de travail configurés pour le service de fractionnement/fusion.

#### Pourquoi mes informations d’identification ne fonctionnent pas quand j’ajoute une partition d’un autre serveur ?
N’utilisez pas les informations d’identification de type « ID d’utilisateur = nom\_utilisateur@nom\_serveur », à la place utilisez simplement « ID d’utilisateur = nom\_utilisateur ». Par ailleurs, assurez-vous que l’ID de connexion « nom\_utilisateur » dispose d’autorisations sur la partition.

#### Dois-je créer un gestionnaire de cartes de partition et remplir les partitions chaque fois que je démarre mes applications ?

Non, la création du gestionnaire de mappages de partitions (par exemple, **[ShardMapManagerFactory.CreateSqlShardMapManager](http://msdn.microsoft.com/library/azure/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmapmanagerfactory.createsqlshardmapmanager.aspx)**) est une opération unique. Votre application doit appeler **[ShardMapManagerFactory.TryGetSqlShardMapManager()](http://msdn.microsoft.com/library/azure/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmapmanagerfactory.trygetsqlshardmapmanager.aspx)** au démarrage de l’application. Un seul appel de ce type n’est possible par domaine d’application.

#### J’ai des questions sur l’utilisation des outils de bases de données élastique. Où trouver les réponses ? 

Contactez-nous via le [forum Azure SQL Database](https://social.msdn.microsoft.com/forums/azure/home?forum=ssdsgetstarted).

#### Quand j’obtiens une connexion de base de données à l’aide d’une clé de partitionnement, je peux encore interroger les données pour d’autres clés de partitionnement sur la même partition. Est-ce normal ?

Les API d’infrastructure élastique vous donnent une connexion à la base de données correspondant à votre clé de partitionnement, mais elles ne permettent pas de filtrer les clés de partitionnement. Ajoutez des clauses **WHERE** à votre requête pour restreindre l’étendue à la clé de partitionnement fournie, si nécessaire.

#### Puis-je utiliser une édition différente d’Azure SQL Database pour chaque partition de mon ensemble de partitions ?

Oui, une partition est une base de données individuelle, par conséquent, une partition peut utiliser une édition Premium alors qu’une autre utilise une édition Standard. Par ailleurs, l’édition d’une partition peut monter ou descendre en puissance plusieurs fois pendant la durée de vie de la partition.

#### Le service de fractionnement/fusion configure-t-il (ou supprime-t-il) une base de données pendant une opération de fractionnement ou de fusion ? 

Non. Pour les opérations de **fractionnement**, la base de données cible doit exister avec le schéma approprié et être enregistrée dans le gestionnaire de mappages de partition. Pour les opérations de **fusion**, vous devez supprimer la partition à partir du gestionnaire de mappages de partition, puis supprimer la base de données.

[AZURE.INCLUDE [elastic-scale-include](../../includes/elastic-scale-include.md)]
 

<!---HONumber=AcomDC_0204_2016-->