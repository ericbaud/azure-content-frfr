<properties 
	pageTitle="Gestion d’une base de données SQL avec SSMS | Microsoft Azure" 
	description="Découvrez comment utiliser SQL Server Management Studio pour gérer des serveurs et les bases de données de la base de données SQL." 
	services="sql-database" 
	documentationCenter=".net" 
	authors="jeffgoll" 
	manager="jeffreyg" 
	editor="tysonn"/>

<tags 
	ms.service="sql-database" 
	ms.workload="data-management" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="03/07/2016" 
	ms.author="jeffreyg"/>


# Gestion de la base de données SQL Azure au moyen de SQL Server Management Studio 


> [AZURE.SELECTOR]
- [Portail Azure](sql-database-manage-portal.md)
- [SSMS](sql-database-manage-azure-ssms.md)
- [PowerShell](sql-database-command-line-tools.md)

Vous pouvez utiliser SQL Server Management Studio (SSMS) pour administrer les serveurs logiques et les bases de données Azure SQL. Cette rubrique vous présente les tâches courantes effectuées avec SSMS. Avant de démarrer, vous devez déjà disposer d’un serveur logique et d’une base de données créés dans Azure SQL. Pour plus d’informations sur la façon de se connecter, puis d’exécuter une requête SELECT simple, consultez [Didacticiel sur la base de données SQL : Créer une base de données SQL en quelques minutes à l’aide d’exemples de données et du portail Azure](sql-database-get-started.md) et [Se connecter à la base de données SQL avec SQL Server Management Studio et exécuter un exemple de requête T-SQL](sql-database-connect-query-ssms.md).

Nous vous recommandons d’utiliser la dernière version de SSMS, quel que soit l’emplacement choisi pour utiliser la base de données Azure SQL. Pour l’obtenir, accédez à [Téléchargez SQL Server Management Studio](https://msdn.microsoft.com/library/mt238290.aspx).

## Créer et gérer les bases de données Azure SQL

Lorsque vous êtes connecté à la base de données **principale**, vous pouvez créer des bases de données sur le serveur et modifier ou supprimer les bases de données existantes. La procédure ci-dessous décrit la façon d’accomplir plusieurs tâches courantes de gestion de base de données par le biais de Management Studio. Pour effectuer ces tâches, vérifiez que vous êtes connecté à la base de données **principale** avec la connexion principale de niveau serveur que vous avez créée lors de la configuration de votre serveur.

Pour ouvrir une fenêtre de requête dans Management Studio, ouvrez le dossier Bases de données, développez **Bases de données système**, cliquez avec le bouton droit sur **master**, puis cliquez sur **Nouvelle requête**.

-   Utilisez l’instruction **CREATE DATABASE** pour créer une base de données. Pour plus d'informations, consultez la rubrique [CREATE DATABASE (Base de données SQL)](https://msdn.microsoft.com/library/dn268335.aspx). L’instruction ci-dessous crée une base de données appelée **myTestDB** et spécifie qu’il s’agit d’une base de données Standard S0 Edition d’une taille maximale de 250 Go.

        CREATE DATABASE myTestDB
        (EDITION='Standard',
         SERVICE_OBJECTIVE='S0');

Cliquez sur **Exécuter** pour exécuter la requête.

-   Utilisez l’instruction **ALTER DATABASE** pour modifier une base de données existante, par exemple pour modifier le nom ou l’édition de la base de données. Pour plus d'informations, consultez [ALTER DATABASE (Base de données SQL)](https://msdn.microsoft.com/library/ms174269.aspx). L’instruction ci-dessous modifie la base de données que vous avez créée à l’étape précédente pour changer l’édition en Standard S1.

        ALTER DATABASE myTestDB
        MODIFY
        (SERVICE_OBJECTIVE='S1');

-   Utilisez l’instruction **DROP DATABASE** pour supprimer une base de données existante. Pour plus d’informations, consultez [DROP DATABASE (Base de données SQL)](https://msdn.microsoft.com/library/ms178613.aspx). L’instruction ci-dessous supprime la base de données **maBDTest**, mais ne la supprimez pas maintenant car vous en aurez besoin pour créer des connexions à la prochaine étape.

        DROP DATABASE myTestBase;

-   La base de données principale dispose d’une vue **sys.databases** vous permettant d’afficher des informations sur toutes les bases de données. Pour afficher toutes les bases de données existantes, exécutez l’instruction suivante :

        SELECT * FROM sys.databases;

-   Dans la base de données SQL, l’instruction **USE** ne permet pas de passer d’une base de données à une autre. À la place, vous devez créer une connexion directe à la base de données cible.

>[AZURE.NOTE] Plusieurs des instructions Transact-SQL permettant de créer ou de modifier une base de données doivent être exécutées au sein de leur propre lot et ne peuvent pas être regroupées avec d’autres instructions Transact-SQL. Pour plus d'informations, consultez les informations spécifiques aux instructions, disponibles à partir des liens proposés ci-dessus.

## création et gestion de connexions

La base de données **master** effectue le suivi des connexions et indique celles qui sont autorisées à créer des bases de données ou d’autres connexions. Gérez les connexions en vous connectant à la base de données **principale** avec la connexion principale de niveau serveur que vous avez créée lors de la configuration de votre serveur. Vous pouvez utiliser les instructions **CREATE LOGIN**, **ALTER LOGIN** ou **DROP LOGIN** pour exécuter des requêtes dans la base de données en vue de gérer les connexions sur l’ensemble du serveur. Pour plus d'informations, consultez [Gestion des bases de données et des connexions dans la base de données SQL](http://msdn.microsoft.com/library/azure/ee336235.aspx).


-   Utilisez l’instruction **CREATE LOGIN** pour créer une connexion de niveau serveur. Pour plus d'informations, consultez [CREATE LOGIN (Base de données SQL)](https://msdn.microsoft.com/library/ms189751.aspx). L’instruction ci-dessous crée une connexion intitulée **connexion1**. Remplacez **motdepasse1** par le mot de passe de votre choix.

        CREATE LOGIN login1 WITH password='password1';

-   Utilisez l’instruction **CREATE USER** pour octroyer des autorisations au niveau de la base de données. Toutes les connexions doivent être créées dans la base de données **principale**, mais pour qu’une connexion permette d’accéder à une autre base de données, vous devez lui octroyer des autorisations de niveau de base de données en utilisant l’instruction **CREATE USER** dans cette base de données. Pour plus d'informations, consultez [CREATE USER (Base de données SQL)](https://msdn.microsoft.com/library/ms173463.aspx).

-   Pour fournir à connexion1 des autorisations d’accès à une base de données appelée **maBDTest**, procédez comme suit :

 1.  Pour actualiser l'Explorateur d'objets de façon à voir la base de données **myTestDB** que vous venez de créer, cliquez avec le bouton droit sur le nom du serveur dans l'Explorateur d'objets, puis cliquez sur **Actualiser**.  

     Si vous avez fermé la connexion, vous pouvez vous reconnecter en sélectionnant **Connecter l’Explorateur d’objets** dans le menu Fichier.

 2. Cliquez avec le bouton droit sur **maBDTest** et sélectionnez **Nouvelle requête**.

    3.  Appliquez l’instruction suivante à la base de données maBDTest pour créer un utilisateur de base de données nommé **utilisateur\_connexion1** correspondant à la connexion de niveau serveur **connexion1**.

            CREATE USER login1User FROM LOGIN login1;

-   Utilisez la procédure stockée **sp\_addrolemember** pour fournir au compte d’utilisateur le niveau approprié d’autorisations dans la base de données. Pour plus d’informations, consultez [sp\_addrolemember (Transact-SQL)](http://msdn.microsoft.com/library/ms187750.aspx). L’instruction ci-dessous fournit à **utilisateur\_connexion1** des autorisations d’accès en lecture seule à la base de données en ajoutant **utilisateur\_connexion1** au rôle **db\_datareader**.

        exec sp_addrolemember 'db_datareader', 'login1User';    

-   Utilisez l’instruction **ALTER LOGIN** pour modifier une connexion existante, par exemple si vous voulez modifier le mot de passe de la connexion. Pour plus d'informations, consultez [ALTER LOGIN (Base de données SQL)](https://msdn.microsoft.com/library/ms189828.aspx). L’instruction **ALTER LOGIN** doit être exécutée sur la base de données **principale**. Revenez à la fenêtre de requête qui est connectée à cette base de données.

    L’instruction ci-dessous modifie la connexion **connexion1** pour réinitialiser le mot de passe. Remplacez **nouveauMotPasse** par le mot de passe de votre choix et **ancienMotPasse** par le mot de passe actuel de la connexion.

        ALTER LOGIN login1
        WITH PASSWORD = 'newPassword'
        OLD_PASSWORD = 'oldPassword';

-   Utilisez l’instruction **DROP LOGIN** pour supprimer une connexion existante. La suppression d’une connexion au niveau du serveur a également pour effet de supprimer tous les comptes d’utilisateur de base de données associés. Pour plus d’informations, consultez [DROP DATABASE (Base de données SQL)](https://msdn.microsoft.com/library/ms178613.aspx). L’instruction **DROP LOGIN** doit être exécutée sur la base de données **principale**. L’instruction ci-dessous supprime la connexion **connexion1**.

        DROP LOGIN login1;

-   La base de données principale dispose d’une vue **sys.sql\_logins** vous permettant d’afficher les connexions. Pour afficher toutes les connexions existantes, exécutez l’instruction suivante :

        SELECT * FROM sys.sql_logins;

## surveillance de la base de données SQL au moyen de vues de gestion dynamique</h2>

La base de données SQL prend en charge plusieurs vues de gestion dynamique vous permettant de surveiller une base de données individuelle. Voici quelques exemples du type de données de surveillance que vous pouvez récupérer au moyen de ces vues. Pour plus d'informations et pour obtenir d'autres exemples d’utilisation, consultez [Surveillance d’une base de données SQL Azure à l’aide de vues de gestion dynamique](https://msdn.microsoft.com/library/azure/ff394114.aspx).

-   L’interrogation d’une vue de gestion dynamique nécessite des autorisations **VIEW DATABASE STATE**. Pour octroyer l’autorisation **VIEW DATABASE STATE** à un utilisateur de base de données spécifique, connectez-vous à la base de données que vous voulez gérer avec votre connexion de principe de niveau serveur et exécutez l’instruction suivante dans la base de données :

        GRANT VIEW DATABASE STATE TO login1User;

-   Calculez la taille de la base de données au moyen de la vue **sys.dm\_db\_partition\_stats**. La vue **sys.dm\_db\_partition\_stats** renvoie les informations de page et de nombre de lignes pour chaque partition de la base de données, vous permettant de calculer la taille de la base de données. La requête suivante renvoie la taille de votre base de données en mégaoctets :

        SELECT SUM(reserved_page_count)*8.0/1024
        FROM sys.dm_db_partition_stats;   

-   Utilisez les vues **sys.dm\_exec\_connections** et **sys.dm\_exec\_sessions** pour extraire les informations concernant les connexions utilisateur actuelles et les tâches internes associées à la base de données. La requête suivante renvoie des informations sur la connexion actuelle :

        SELECT
            e.connection_id,
            s.session_id,
            s.login_name,
            s.last_request_end_time,
            s.cpu_time
        FROM
            sys.dm_exec_sessions s
            INNER JOIN sys.dm_exec_connections e
              ON s.session_id = e.session_id;

-   Utilisez la vue **sys.dm\_exec\_query\_stats** pour extraire les statistiques de performance consolidées pour les plans de requête mise en cache. La requête suivante renvoie des informations relatives aux cinq requêtes principales classées sur la base du temps processeur moyen.

        SELECT TOP 5 query_stats.query_hash AS "Query Hash",
            SUM(query_stats.total_worker_time), SUM(query_stats.execution_count) AS "Avg CPU Time",
            MIN(query_stats.statement_text) AS "Statement Text"
        FROM
            (SELECT QS.*,
            SUBSTRING(ST.text, (QS.statement_start_offset/2) + 1,
            ((CASE statement_end_offset
                WHEN -1 THEN DATALENGTH(ST.text)
                ELSE QS.statement_end_offset END
                    - QS.statement_start_offset)/2) + 1) AS statement_text
             FROM sys.dm_exec_query_stats AS QS
             CROSS APPLY sys.dm_exec_sql_text(QS.sql_handle) as ST) as query_stats
        GROUP BY query_stats.query_hash
        ORDER BY 2 DESC;
 
 

<!---------HONumber=AcomDC_0309_2016-->