<properties 
   pageTitle="Query Performance Insight pour base de données SQL Azure" 
   description="La surveillance des performances des requêtes identifie les requêtes consommant le plus d’UC pour une base de données SQL Azure." 
   services="sql-database" 
   documentationCenter="" 
   authors="stevestein" 
   manager="jeffreyg" 
   editor="monicar"/>

<tags
   ms.service="sql-database"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="data-management" 
   ms.date="02/03/2016"
   ms.author="sstein"/>

# Query Performance Insight pour base de données SQL Azure


La gestion et le réglage des performances des bases de données relationnelles est une tâche complexe qui nécessite une réelle expertise et un investissement en temps. Query Performance Insight vous permet de passer moins de temps à résoudre les problèmes de performances des bases de données en fournissant :

- Une meilleure compréhension de la consommation des ressources des bases de données (DTU). 
- Les requêtes principales consommatrices d’UC peuvent être réglées pour améliorer les performances. 
- La possibilité d’examiner les détails d’une requête.

## Composants requis

- Query Performance Insight est uniquement disponible avec la base de données SQL Azure V12.
- Query Performance Insight qui nécessite que le [magasin de requêtes](https://msdn.microsoft.com/library/dn817826.aspx) soit en cours d’exécution sur votre base de données. Le portail vous invite à activer le magasin de requêtes s’il ne fonctionne pas encore.

 
## Autorisations

Les autorisations [de contrôle d’accès en fonction du rôle](../active-directory/role-based-access-control-configure.md) suivantes sont requises pour utiliser Query Performance Insight :

- Les autorisations **Reader**, **Owner**, **Contributor**, **SQL DB Contributor** ou **SQL Server Contributor** sont requises pour afficher les requêtes et graphiques consommant le plus de ressources. 
- Les autorisations **Owner**, **Contributor**, **SQL DB Contributor** ou **SQL Server Contributor** sont requises pour afficher le texte de requête.



## Utilisation de Query Performance Insight

Query Performance Insight est simple d’utilisation :

- Passez en revue la liste des requêtes consommant le plus de ressources. 
- Sélectionnez une requête pour en afficher les détails.
- Cliquez sur **Paramètres** pour personnaliser l’affichage des données ou pour afficher une autre période.
- Ouvrez [Index Advisor](sql-database-index-advisor.md), puis vérifiez si des recommandations sont disponibles.



> [AZURE.NOTE] Quelques heures de données doivent être capturées par Query Store pour que la base de données SQL fournisse des informations sur les performances des requêtes. Si la base de données n’a aucune activité ou que Query Store est resté inactif pendant une certaine période, les graphiques seront vides lors de l’affichage de cette période. Vous pouvez activer Query Store à tout moment s’il n’est pas en cours d’exécution.



## Consultez les requêtes principales consommatrices d’UC

Dans le [portail](http://portal.azure.com), procédez comme suit :

1. Accédez à une base de données SQL et cliquez sur **Query Performance Insight**. 

    ![Query Performance Insight][1]

    La vue des principales requêtes s’ouvre, affichant une liste des requêtes consommant le plus d’UC.

1. Cliquez sur le graphique pour plus d’informations.<br>La première ligne affiche le pourcentage de DTU global pour la base de données, tandis que les barres affichent le pourcentage d’UC consommé par les requêtes sélectionnées pendant l’intervalle sélectionné (par exemple, si **Semaine dernière** est sélectionné, chaque barre représente un jour).

    ![principales requêtes][2]

    Le tableau du bas contient des informations agrégées concernant les requêtes visibles.

    -	Temps processeur moyen par requête pendant l’intervalle observable. 
    -	Durée totale par requête.
    -	Nombre total d’exécutions pour une requête particulière.


	Sélectionnez ou effacez des requêtes individuelles pour les inclure ou les exclure du graphique.


1. Si vos données deviennent obsolètes, cliquez sur le bouton **Actualiser**.
1. Vous pouvez également cliquer sur **Paramètres** pour personnaliser l’affichage des données de consommation d’UC ou pour afficher une autre période.

    ![paramètres](./media/sql-database-query-performance/settings.png)

## Affichage des détails d’une requête individuelle

Pour afficher les détails d’une requête :

1. Cliquez sur n’importe quelle requête dans la liste des principales requêtes.

    ![détails](./media/sql-database-query-performance/details.png)

4. La vue détaillée s’ouvre et la consommation d’UC de requêtes est répartie dans le temps.
3. Cliquez sur le graphique pour plus d’informations.<br>La première ligne affiche le pourcentage de DTU global et les barres affichent le pourcentage d’UC consommé par la requête sélectionnée.
4. Passez en revue les données pour afficher des mesures détaillées, notamment la durée, le nombre d’exécutions et le pourcentage d’utilisation des ressources pour chaque intervalle que la requête exécutait.
    
    ![détails de la requête][3]

1. Vous pouvez également cliquer sur **Paramètres** pour personnaliser l’affichage des données de consommation d’UC ou pour afficher une autre période.


## 	Optimisation de la configuration du magasin de requêtes pour Query Performance Insight

Pendant l’utilisation de Query Performance Insight, vous pouvez rencontrer les messages suivants du magasin de requêtes :

- « Le magasin de requêtes a atteint sa capacité maximale et ne peut plus collecter de données. »
- « Le magasin de requêtes de cette base de données est en lecture seule et ne peut pas collecter de données d’analyse de performances ».
- « Les paramètres du magasin de requêtes ne sont pas définis de manière optimale pour Query Performance Insight. »

Ces messages s’affichent généralement lorsque le magasin de requêtes ne peut plus collecter de données supplémentaires. Pour résoudre ce problème, vous avez trois options :

-	Modifier la stratégie de rétention et de capture du magasin de requêtes
-	Augmenter la taille du magasin de requêtes 
-	Supprimer le contenu du magasin de requêtes

### Stratégie de rétention et de capture recommandée

Il existe deux types de stratégies de rétention :

- Basée sur la taille : si la valeur AUTO est définie, les données sont automatiquement supprimées quand la taille maximale est proche.
- Basée sur le temps : la période par défaut est de 30 jours. Ainsi, si le magasin de requêtes manque d’espace, il supprimera les informations liées aux requêtes de plus de 30 jours. 

La stratégie de capture peut avoir les valeurs suivantes :

- **Tout**: toutes les requêtes sont capturées. Il s'agit de l'option par défaut.
- **Auto**: les requêtes peu fréquentes et les requêtes avec une durée de compilation et d’exécution insignifiante sont ignorées. Les seuils concernant le nombre d’exécutions, et la durée de compilation et d’exécution, sont déterminés en interne.
- **Aucun**: le magasin de requêtes capture de nouvelles requêtes.
	
Nous vous recommandons de définir toutes les stratégies sur Auto, et de définir la stratégie de suppression des anciennes requêtes sur 30 jours :

    ALTER DATABASE [YourDB] 
    SET QUERY_STORE (SIZE_BASED_CLEANUP_MODE = AUTO);
    	
    ALTER DATABASE [YourDB] 
    SET QUERY_STORE (CLEANUP_POLICY = (STALE_QUERY_THRESHOLD_DAYS = 30));
    
    ALTER DATABASE [YourDB] 
    SET QUERY_STORE (QUERY_CAPTURE_MODE = AUTO);

Augmentez la taille du magasin de requêtes. Pour cela, connectez-vous à une base de données et exécutez la requête suivante :

    ALTER DATABASE [YourDB]
    SET QUERY_STORE (MAX_STORAGE_SIZE_MB = 1024);

Supprimez le contenu du magasin de requêtes. Cette action entraînera la suppression de l’ensemble des informations du magasin de requêtes :

    ALTER DATABASE [YourDB] SET QUERY_STORE CLEAR;


## Résumé

Query Performance Insight vous permet de comprendre l’impact de votre charge de travail de requêtes et la relation de celle-ci avec la consommation des ressources de base de données. Cette fonctionnalité vous permettra d’en savoir plus sur les requêtes les plus gourmandes et d’identifier facilement les requêtes à corriger avant qu’elles ne deviennent un problème. Cliquez sur **Query Performance Insight** dans une base de données pour afficher les requêtes qui consomment le plus de ressources (UC).




## Étapes suivantes

Les charges de travail d’une base de données sont dynamiques et évoluent en permanence. Surveillez vos requêtes et continuez à les régler pour améliorer les performances.

Pour obtenir d’autres recommandations concernant l’amélioration des performances de votre base de données SQL, cliquez sur [Index Advisor](sql-database-index-advisor.md) dans le panneau **Query Performance Insight**.

![Assistant Index](./media/sql-database-query-performance/ia.png)


<!--Image references-->
[1]: ./media/sql-database-query-performance/tile.png
[2]: ./media/sql-database-query-performance/top-queries.png
[3]: ./media/sql-database-query-performance/query-details.png

<!---HONumber=AcomDC_0224_2016-->