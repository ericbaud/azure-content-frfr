<properties
   pageTitle="Présentation de Azure SQL Data Warehouse | Microsoft Azure"
   description="Base de données distribuée dédiée aux entreprises qui prend en charge le traitement de pétaoctets de données relationnelles et non relationnelles. Il s’agit du premier entrepôt de données cloud prenant en charge l’augmentation, la réduction et la pause en quelques secondes."
   services="sql-data-warehouse"
   documentationCenter="NA"
   authors="lodipalm"
   manager="barbkess"
   editor=""/>

<tags
   ms.service="sql-data-warehouse"
   ms.devlang="NA"
   ms.topic="get-started-article"
   ms.tgt_pltfrm="NA"
   ms.workload="data-services"
   ms.date="03/26/2016"
   ms.author="lodipalm;barbkess;mausher;jrj;sonyama;"/>


# En quoi consiste Azure SQL Data Warehouse ?

Azure SQL Data Warehouse est une base de données de mise à l’échelle basée sur le cloud qui prend en charge le traitement de grands volumes de données relationnelles et non relationnelles. Reposant sur notre architecture MPP (massively parallel processing), SQL Data Warehouse peut prendre en charge la charge de travail de votre entreprise.

SQL Data Warehouse :

- Combine notre base de données relationnelle SQL Server éprouvée avec nos capacités de mise à l’échelle du cloud Azure. Vous pouvez augmenter, diminuer, suspendre ou reprendre un calcul en quelques secondes. Cela vous permet de réduire les coûts liés à la mise à l’échelle de l’UC, si nécessaire, et de réduire l’utilisation pendant les heures creuses.
- S’appuie sur notre plateforme Azure. Il est facile à déployer, à entretenir et assure une tolérance complète aux pannes grâce à des sauvegardes automatiques. 
- Complète l’écosystème SQL Server. Permet le développement avec SQL Server T-SQL et ses outils qui vous sont familiers.

Poursuivez votre lecture pour en savoir plus sur les fonctionnalités clés de SQL Data Warehouse.

## Optimisation

### Architecture MPP (Massively Parallel Processing)

SQL Data Warehouse utilise l’architecture MPP conçue pour exécuter certains des plus grands entrepôts de données sur site.

Actuellement, notre architecture MPP répartit vos données sur 60 unités de traitement et de stockage sans partage. Les données sont stockées dans des objets blob géorépliqués redondants Azure Storage et liées aux nœuds de calcul pour l’exécution de la requête. Avec cette architecture, nous pouvons adopter une approche « Diviser pour mieux régner » pour l’exécution de requêtes T-SQL complexes. Lors du traitement, le nœud de contrôle analyse la requête, puis chaque nœud de calcul « conquiert » sa partie de données en parallèle.

En combinant notre architecture MPP et les capacités de stockage Azure, SQL Data Warehouse peut :

- Agrandir ou réduire le stockage indépendant de calcul.
- Augmenter ou réduire le calcul sans déplacement de données. 
- Suspendre le calcul tout en conservant les données.
- Reprendre le calcul à tout moment.

L’architecture est décrite en détail ci-dessous.

![Architecture de SQL Data Warehouse][1]


- **Nœud de contrôle :** le nœud de contrôle « contrôle » le système. Il s’agit du nœud frontal qui interagit avec toutes les applications et les connexions. Dans SQL Data Warehouse, le nœud de contrôle s’appuie sur une base de données SQL ; une fois la connexion établie, vous obtenez une interface et un fonctionnement similaire. Sous la surface, le nœud de contrôle coordonne tous les mouvements de données et calculs nécessaires pour exécuter des requêtes parallèles sur vos données distribuées. Lorsque vous envoyez une requête TSQL à SQL Data Warehouse, le nœud de contrôle la transforme en requêtes distinctes qui s’exécutent sur chaque nœud de calcul en parallèle.

- **Nœuds de calcul :** les nœuds de calcul alimentent SQL Data Warehouse. Il s’agit de bases de données SQL qui traitent les étapes de votre requête et gèrent vos données. Lors de l’ajout de données, SQL Data Warehouse distribue les lignes à l’aide de vos nœuds de calcul. Les nœuds de calcul sont également les travaux qui exécutent les requêtes parallèles sur vos données. Après traitement, ils renvoient les résultats au nœud de contrôle. Pour terminer la requête, le nœud de contrôle rassemble les résultats et renvoie le résultat final.


- **Stockage :** les données sont stockées dans des objets blob Azure Storage au format JSON. Lorsque les nœuds de calcul interagissent avec vos données, la lecture et l’écriture s’effectuent directement vers et depuis le stockage d’objets blob. Dans la mesure où le stockage Azure se développe de manière transparente et sans limite, SQL Data Warehouse peut faire de même. Étant donné que le calcul et le stockage sont indépendants, SQL Data Warehouse peut automatiquement mettre à l’échelle le stockage séparément du calcul de mise à l’échelle et inversement. Le stockage Azure offre également une tolérance complète aux pannes et rationalise le processus de sauvegarde et de restauration.
   

- **Service de déplacement de données :** DMS correspond à notre technologie de déplacement des données entre les nœuds. DMS permet aux nœuds de calcul d’accéder aux données dont ils ont besoin pour les jonctions et les agrégations. DMS n’est pas un service Azure. C’est un service Windows qui s’exécute en même temps que la base de données SQL sur tous les nœuds. Dans la mesure où DMS s’exécute en arrière-plan, aucune interaction directe avec ce service n’est possible. Toutefois, lorsque vous examinez les plans de requête, vous remarquerez qu’ils comprennent certaines opérations DMS, dans la mesure où le déplacement des données est nécessaire sous une certaine forme pour exécuter chaque requête en parallèle.


### Optimisation des performances des requêtes

En plus de la stratégie « Diviser pour mieux régner », l’approche MPP s’appuie sur un certain nombre d’optimisations de performances spécifiques à l’entreposage de données, parmi lesquelles :

- Un optimiseur de requêtes distribuées et un ensemble de statistiques complexes sur toutes les données. À partir des informations relatives à la distribution et à la taille des données, le service est en mesure d’optimiser les requêtes en évaluant le coût des opérations de requête distribuée spécifique.

- Des algorithmes et des techniques avancés intégrés au processus de déplacement de données pour déplacer efficacement les données entre les ressources de calcul dans le cadre de l’opération de requête. Ces opérations de mouvement de données sont intégrées et toutes les optimisations apportées à DMS s’effectuent automatiquement.

- Des index de stockage de colonnes cluster par défaut. En s’appuyant sur le stockage en colonnes, SQL Data Warehouse peut délivrer des performances de compression jusqu’à 5 fois supérieures au stockage en ligne, et des performances de requête jusqu’à 10 fois supérieures. Les requêtes d’analyse qui doivent analyser un grand nombre de lignes sont particulièrement adaptées aux index de stockage de colonnes cluster.

## Extensibilité

L’architecture de SQL Data Warehouse sépare le stockage du calcul, qui peuvent ainsi être mis à l’échelle indépendamment l’un de l’autre. La structure de déploiement simple et rapide de la base de données SQL permet de disposer instantanément de ressources de calcul supplémentaires. L’utilisation d’Azure Storage Blobs vient également enrichir cette structure. Les objets Blob garantissent un stockage stable et répliqué, tout en permettant une extension sans effort de l’infrastructure et ce, à moindre coût. Grâce à un stockage à l’échelle du cloud combiné à un calcul Azure, SQL Data Warehouse vous permet de payer le stockage de performances des requêtes dont vous avez besoin au moment où vous en avez besoin. Vous pouvez modifier la quantité de calcul en déplaçant simplement un curseur vers la gauche ou vers la droite dans le portail Azure, mais vous pouvez également planifier cette modification avec T-SQL et PowerShell.

Tout en offrant la possibilité de contrôler entièrement la quantité de ressources de calcul indépendamment du stockage, SQL Data Warehouse vous permet de suspendre totalement votre entrepôt de données. Sans modifier le stockage en place, toutes les ressources de calcul sont transférées dans le pool principal d’Azure, pour vous permettre de réaliser des économies immédiates. Si nécessaire, il vous suffit de reprendre le calcul et de mettre vos données et vos ressources de calcul à la disposition de votre charge de travail.

L’utilisation du calcul dans SQL Data Warehouse est évaluée et exprimée en SQL Data Warehouse Units (DWUs). Les DWU permettent de mesurer la puissance sous-jacente de votre entrepôt de données et sont conçues pour vous assurer qu’une quantité standard de performances reste associée à votre entrepôt à un moment donné. Plus précisément, nous utilisons les DWU pour :

- vous permettre d’étendre efficacement votre entrepôt de données sans que vous ayez à vous préoccuper du matériel ou des logiciels sous-jacents ;

- vous donner un aperçu des performances que vous obtiendrez en termes de DWU avant que vous ne modifiiez la taille de votre entrepôt de données ;

- vous permettre de modifier ou déplacer le matériel et les logiciels sous-jacents de votre instance sans incidence sur les performances de votre charge de travail ;

- pouvoir ajuster l’architecture sous-jacente du service sans affecter les performances de votre charge de travail ;

- améliorer rapidement les performances de SQL Data Warehouse d’une manière évolutive et qui impacte uniformément le système.

### Data Warehouse Units

Nous considérons plus particulièrement les Data Warehouse Units comme une mesure de trois indicateurs précis qui se trouvent être fortement corrélés aux performances de la charge de travail d’entreposage de données. En prévision de la disponibilité générale, nous cherchons à faire en sorte que ces mesures clés de la charge de travail évoluent de façon linéaire avec les DWU que vous avez choisies pour votre entrepôt de données.

**Analyse/agrégation :** cette mesure de la charge de travail utilise une requête d'entreposage de données standard qui analyse un grand nombre de lignes avant d'effectuer une agrégation complexe. Cette opération est très gourmande en E/S et en ressources processeur.

**Charge :** cette mesure évalue la capacité à injecter des données dans le service. Les charges sont complétées par PolyBase, conçu pour charger un dataset représentatif à partir d’un objet blob Azure Storage. Cette mesure est conçue pour souligner les aspects réseau et processeur du service.

**CREATE TABLE AS SELECT (CTAS) :** CTAS mesure la capacité à créer une copie d'une table. Cela implique la lecture des données sur le système de stockage, leur distribution entre les nœuds de l’appliance et leur réécriture dans le système de stockage. Cette opération est très gourmande en ressources processeur et réseau.

### Quand mettre à l’échelle ?

En général, nous tenons à simplifier au maximum les DWU. Lorsque vous avez besoin de résultats plus rapides, augmentez vos DWU et payez simplement un peu plus cher pour obtenir de meilleures performances. Lorsque vous avez besoin d’une puissance de traitement moindre, réduisez vos DWU et payez uniquement ce dont vous avez besoin. Voici quelques cas de figure qui pourraient vous inciter à vouloir modifier votre nombre de DWU :

- Lorsque vous n’avez pas besoin d’exécuter des requêtes, par exemple le soir ou le week-end, interrompez les ressources de calcul afin d’annuler l’ensemble des requêtes en cours et supprimez l’ensemble des unités DWU allouées à votre entrepôt de données.

- Lorsque vous exécutez une opération de chargement ou de transformation de données importante, vous pouvez souhaiter monter en charge afin que vos données soient disponibles plus rapidement.

- Pour obtenir la valeur DWU idéale, essayez d’augmenter et de réduire vos DWU et d’exécuter quelques requêtes après le chargement de vos données. Puisque la mise à l’échelle est rapide, vous pouvez essayer plusieurs différents niveaux de performances sans vous engager au-delà d’une heure.

> [AZURE.NOTE] Notez qu’en raison de l’architecture ou de SQL Data Warehouse, vous risquez de ne pas obtenir l’augmentation de performances attendue à des volumes de données inférieurs. Nous vous recommandons de commencer par des volumes de données d’au moins 1 To afin d’obtenir des résultats de tests de performances précis.

## Intégration

SQL Data Warehouse s’appuie sur le moteur de base de données relationnelle éprouvé de SQL Server et inclut les nombreuses fonctionnalités que vous pouvez attendre d’un entrepôt de données d’entreprise. Si vous connaissez déjà Transact-SQL, il vous suffit de transposer vos connaissances sur SQL Data Warehouse. Que vous ayez atteint un stade avancé ou que vous débutiez, les exemples fournis dans la documentation peuvent aider à prendre un bon départ. En règle générale, vous pouvez considérer la façon dont nous avons construit les éléments de langage de SQL Data Warehouse comme suit :

- SQL Data Warehouse utilise la syntaxe Transact-SQL (TSQL) de SQL Server pour de nombreuses opérations et prend en charge un large éventail de constructions SQL traditionnelles, telles que les procédures stockées, les fonctions définies par l’utilisateur, le partitionnement de tables, les index et les classements.

- SQL Data Warehouse contient également plusieurs fonctionnalités SQL Server avancées, notamment les index columnstore en cluster, l’intégration PolyBase et l’audit de données (complété par l’évaluation des menaces).

- Dans la mesure où SQL Data Warehouse est toujours en cours de développement, certains éléments de langage TSQL moins courants pour les charges de travail d’entreposage de données ou plus récemment ajoutés à SQL Server peuvent être indisponibles pour le moment. Pour plus d’informations à ce sujet, consultez notre documentation relative à la migration.

En profitant de Transact-SQL et de fonctionnalités identiques entre SQL Server, SQL Data Warehouse, la base de données SQL et Analytics Platform System, vous êtes en mesure de développer une solution adaptée à vos besoins. Vous pouvez définir l’emplacement de stockage de vos données, en fonction des exigences de performance, de sécurité et de mise à l’échelle, puis, en fonction des besoins, transférer vos données entre différents systèmes.

Outre l’adoption de la surface d’exposition TSQL de SQL Server, SQL Data Warehouse s’intègre également à de nombreux outils que les utilisateurs de SQL Server sont susceptibles de connaître. Nous nous sommes en particulier employés à intégrer quelques catégories d’outils avec SQL Data Warehouse, notamment :

**Les outils SQL Server traditionnels :** SQL Data Warehouse est totalement intégré à SQL Server Analysis Services, Integration Services et Reporting Services.

**Des outils basés sur le cloud :** SQL Data Warehouse peut être utilisé avec un certain nombre de nouveaux outils dans Azure, et offre une profonde intégration avec les composants Data Factory, Stream Analytics, Machine Learning et Power BI d'Azure.

**Des outils tiers :** de nombreux fournisseurs d'outils tiers ont certifié l'intégration de leurs outils avec SQL Data Warehouse. Consultez la liste complète.

## Hybride

L’utilisation de SQL Data Warehouse avec PolyBase permet aux utilisateurs de déplacer plus efficacement que jamais des données au sein de leur écosystème, et de configurer ainsi des scénarios hybrides sophistiqués avec des sources de données non relationnelles et sur site.

Simple d’utilisation, PolyBase vous permet de valoriser vos données de différentes sources en appliquant les commandes T-SQL que vous connaissez déjà. PolyBase vous permet d’interroger des données non relationnelles conservées dans le stockage d’objets Blob Microsoft Azure comme des tables standard. Utilisez PolyBase pour interroger des données non relationnelles ou pour importer des données non relationnelles dans SQL Data Warehouse.

- PolyBase accède aux données non relationnelles à l’aide de tables externes. Les définitions de table sont stockées dans SQL Data Warehouse et sont accessibles par SQL et par les outils de la même manière que vous accéderiez à des données relationnelles normales.

- PolyBase assure une intégration standard. La solution expose des fonctionnalités identiques à l’ensemble des sources qu’elle prend en charge. Les données lues par PolyBase peuvent se présenter sous différents formats, notamment les fichiers délimités ou ORC.

- PolyBase peut être utilisé pour accéder au stockage d’objets Blob qui est également utilisé comme espace de stockage pour un cluster HD Insight, ce qui vous permet d’accéder aux mêmes données avec des outils relationnels et non relationnels.


## Étapes suivantes

À présent que vous en savez un peu plus sur SQL Data Warehouse, renseignez-vous sur la [charge de travail de l'entrepôt de données] et sur le [provisionnement], puis chargez un [exemple de données] pour faire vos premiers pas.

>[AZURE.NOTE] Nous souhaitons améliorer cet article. Si vous choisissez de répondre « non » à la question « Cet article vous a-t-il été utile ? », veuillez ajouter une brève suggestion concernant ce qui manque ou comment cet article peut être amélioré. Merci d’avance !

<!--Image references-->
[1]: ./media/sql-data-warehouse-overview-what-is/dwarchitecture.png

<!--Article references-->
[charge de travail de l'entrepôt de données]: ./sql-data-warehouse-overview-workload.md
[exemple de données]: ./sql-data-warehouse-get-started-load-samples.md
[provisionnement]: ./sql-data-warehouse-get-started-provision.md

<!--MSDN references-->

<!--Other Web references-->

<!---HONumber=AcomDC_0330_2016-->