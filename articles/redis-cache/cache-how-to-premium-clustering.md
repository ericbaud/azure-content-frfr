<properties 
	pageTitle="Comment configurer le clustering Redis pour un Cache Redis Azure Premium" 
	description="Découvrez comment créer et gérer le clustering Redis pour vos instances de Cache Redis Azure de niveau Premium" 
	services="redis-cache" 
	documentationCenter="" 
	authors="steved0x" 
	manager="erikre" 
	editor=""/>

<tags 
	ms.service="cache" 
	ms.workload="tbd" 
	ms.tgt_pltfrm="cache-redis" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="03/04/2016" 
	ms.author="sdanie"/>

# Comment configurer le clustering Redis pour un Cache Redis Azure Premium
Le Cache Redis Azure offre différents types de caches, permettant de choisir parmi plusieurs tailles et fonctionnalités de cache en toute flexibilité, y compris le nouveau niveau Premium.

Le niveau Premium de Cache Redis Azure comprend le clustering, la persistance et la prise en charge de réseau virtuel. Cet article décrit comment configurer le clustering dans une instance Premium de Cache Redis Azure.

Pour plus d’informations sur les autres fonctionnalités du cache Premium, consultez [Comment configurer la persistance pour un Cache Redis Azure Premium](cache-how-to-premium-persistence.md) et [Comment configurer la prise en charge de réseau virtuel pour un Cache Redis Azure Premium](cache-how-to-premium-vnet.md).

## Qu’est-ce que le cluster Redis ?
Le Cache Redis Azure propose le cluster Redis tel qu'[implémenté dans Redis](http://redis.io/topics/cluster-tutorial). Avec le cluster Redis, vous bénéficiez des avantages suivants.

-	Possibilité de fractionner automatiquement votre dataset parmi plusieurs nœuds. 
-	Continuité des opérations quand un sous-ensemble de nœuds rencontre des erreurs ou ne peut pas communiquer avec le reste du cluster. 
-	Débit supplémentaire : le débit augmente de façon linéaire à mesure que vous augmentez le nombre de partitions. 
-	Taille de mémoire supplémentaire : augmente de façon linéaire à mesure que vous augmentez le nombre de partitions.  

Pour plus d’informations sur la taille, le débit et la bande passante des caches Premium, consultez le [Forum aux questions sur le Cache Redis Azure](cache-faq.md#what-redis-cache-offering-and-size-should-i-use).

Dans Azure, le cluster Redis est proposé sous forme de modèle principal/réplica où chaque partition a une paire principal/réplica avec réplication gérée par le service Cache Redis Azure.

## Clustering
La configuration du clustering s'effectue dans le panneau **Nouveau cache Redis** lors de la création du cache. Pour créer un cache, connectez-vous au [portail Azure](https://portal.azure.com) et cliquez sur **Nouveau**->**Données + stockage**>**Cache Redis**.

![Création d’un cache Redis][redis-cache-new-cache-menu]

Pour configurer le clustering, sélectionnez d'abord l'un des caches **Premium** dans le panneau **Choisir un niveau tarifaire**.

![Choisir un niveau tarifaire][redis-cache-premium-pricing-tier]

La configuration du clustering s'effectue dans le panneau **Cluster Redis**.

![Clustering][redis-cache-clustering]

Vous pouvez avoir jusqu’à 10 partitions dans le cluster. Cliquez sur **Activé** et faites glisser le curseur ou tapez un nombre compris entre 1 et 10 pour **Nombre de partitions**, puis cliquez sur **OK**.

Chaque partition est une paire de cache principal/réplica gérée par Azure et la taille totale du cache est calculée en multipliant le nombre de partitions par la taille de cache sélectionnée dans le niveau tarifaire.

![Clustering][redis-cache-clustering-selected]

Une fois le cache créé, vous vous y connectez et l’utilisez tout comme un cache hors cluster. Redis distribuera les données parmi les partitions de Cache. Si les diagnostics sont [activés](cache-how-to-monitor.md#enable-cache-diagnostics), les mesures sont capturées séparément pour chaque partition et peuvent être [affichées](cache-how-to-monitor.md) dans le panneau du Cache Redis.

Pour accéder à un exemple de code relatif à l’utilisation du clustering avec le client StackExchange.Redis, consultez la partie [clustering.cs](https://github.com/rustd/RedisSamples/blob/master/HelloWorld/Clustering.cs) de l’exemple [Hello World](https://github.com/rustd/RedisSamples/tree/master/HelloWorld).

<a name="move-exceptions"></a>
>[AZURE.IMPORTANT] Lorsque vous vous connectez à un Cache Redis Azure avec le clustering activé à l’aide de StackExchange.Redis, vous pouvez rencontrer un problème et recevoir des exceptions `MOVE`. Cela se produit il faut un peu de temps au client de cache StackExchange.Redis pour collecter des informations sur les nœuds du cluster de cache. Ces exceptions peuvent se produire si vous vous connectez au cache pour la première fois et effectuez immédiatement des appels vers le cache avant que le client ait terminé la collecte de ces informations. Pour résoudre ce problème dans votre application, le plus simple consiste à vous connecter au cache et attendre une seconde avant de lancer des appels vers le cache. Pour cela, ajoutez un `Thread.Sleep(1000)` comme indiqué dans l’exemple de code suivant. Notez que le `Thread.Sleep(1000)` se produit uniquement lors de la connexion initiale au cache. Pour plus d’informations, consultez [StackExchange.Redis.RedisServerException - MOVED #248](https://github.com/StackExchange/StackExchange.Redis/issues/248). Un correctif pour résoudre ce problème est en cours de développement et les mises à jour seront publiées ici. **Mise à jour** : ce problème est résolu dans la dernière build [1\.1.572-alpha préliminaire](https://www.nuget.org/packages/StackExchange.Redis/1.1.572-alpha) de StackExchange.Redis. Consultez la [page StackExchange.Redis sur NuGet](https://www.nuget.org/packages/StackExchange.Redis/) pour connaître la build la plus récente.


	private static Lazy<ConnectionMultiplexer> lazyConnection = new Lazy<ConnectionMultiplexer>(() =>
	{
        // Connect to the Redis cache for the first time
	    var connection =  ConnectionMultiplexer.Connect("contoso5.redis.cache.windows.net,abortConnect=false,ssl=true,password=...");

		// Wait for 1 second
		Thread.Sleep(1000);

		// Return the connected ConnectionMultiplexer
		return connection;
	});
	
	public static ConnectionMultiplexer Connection
	{
	    get
	    {
	        return lazyConnection.Value;
	    }
	}

<a name="cluster-size"></a>
## Modifier la taille de cluster sur un cache de niveau Premium en cours d’exécution

Pour modifier la taille de cluster d’un cache de niveau Premium en cours d’exécution sur lequel le clustering est activé, cliquez sur **Taille du cluster Redis (version préliminaire)** dans le panneau **Paramètres**.

>[AZURE.NOTE] Notez que si le niveau Premium du cache Redis Azure a été publié en disponibilité générale, la fonctionnalité Taille du cluster Redis est actuellement en version préliminaire.

![Taille du cluster Redis][redis-cache-redis-cluster-size]

Pour modifier la taille du cluster, utilisez le curseur ou entrez un nombre compris entre 1 et 10 dans la zone de texte **Nombre de partitions**, puis cliquez sur **OK** pour enregistrer.

## Forum aux questions sur le clustering

La liste suivante présente différentes réponses aux questions les plus fréquemment posées sur le clustering du Cache Redis Azure.

## Dois-je apporter des modifications à mon application cliente pour utiliser le clustering ?

-	Lorsque le clustering est activé, seule la base de données 0 est disponible. Si votre application cliente utilise plusieurs bases de données et qu'elle essaie de lire ou d'écrire dans une base de données autre que 0, l'exception suivante est levée. `Unhandled Exception: StackExchange.Redis.RedisConnectionException: ProtocolFailure on GET --->` `StackExchange.Redis.RedisCommandException: Multiple databases are not supported on this server; cannot switch to database: 6`
-	Si vous utilisez [StackExchange.Redis](https://www.nuget.org/packages/StackExchange.Redis/), vous devez utiliser la version 1.0.481 ou une version ultérieure. Vous vous connectez au cache à l’aide des [points de terminaison, ports et clés](cache-configure.md#properties) que vous utilisez pour vous connecter à un cache pour lequel le clustering n’est pas activé. La seule différence est que toutes les lectures et les écritures doivent être effectuées sur la base de données 0.
	-	D’autres clients peuvent avoir des exigences différentes. Consultez [Tous les clients Redis prennent-ils en charge le clustering ?](#do-all-redis-clients-support-clustering).
-	Si votre application utilise plusieurs opérations sur les clés traitées par lot dans une seule commande, toutes les clés doivent se trouver dans la même partition. Pour ce faire, consultez [Comment les clés sont-elles distribuées dans un cluster ?](#how-are-keys-distributed-in-a-cluster).
-	Si vous utilisez un fournisseur d’état de session ASP.NET Redis, vous devez utiliser la version 2.0.1 ou une version ultérieure. Consultez [Puis-je utiliser le clustering avec les fournisseurs d’état de session ASP.NET Redis et de mise en cache de la sortie ?](#can-i-use-clustering-with-the-redis-aspnet-session-state-and-output-caching-providers).

## Comment les clés sont-elles distribuées dans un cluster ?

Selon la documentation Redis [Modèle de distribution de clés](http://redis.io/topics/cluster-spec#keys-distribution-model), l’espace de clé est fractionné en 16 384 emplacements. Chaque clé est hachée et affectée à l’un de ces emplacements, qui sont répartis entre les nœuds du cluster. Vous pouvez configurer la partie de la clé qui est hachée pour vous assurer que plusieurs clés se trouvent dans la même partition à l’aide de balises de hachage.

-	Clés avec une balise de hachage : si une partie de la clé est placée entre `{` et `}`, seule cette partie de la clé est hachée aux fins de détermination de l'emplacement de hachage d'une clé. Par exemple, les 3 clés suivantes se trouvent dans la même partition : `{key}1`, `{key}2` et `{key}3`, étant donné que seule la partie `key` du nom est hachée. Pour obtenir une liste complète des spécifications de balises de hachage de clés, consultez [Balises de hachage de clés](http://redis.io/topics/cluster-spec#keys-hash-tags).
-	Clés sans balise de hachage : le nom entier de la clé est utilisé pour le hachage. Il en résulte une distribution statistiquement uniforme sur les partitions du cache.

Pour optimiser les performances et le débit, nous vous recommandons de distribuer les clés uniformément. Si vous utilisez des clés avec une balise de hachage, il incombe à l’application de vérifier que les clés sont réparties uniformément.

Pour plus d’informations, consultez [Modèle de distribution de clés](http://redis.io/topics/cluster-spec#keys-distribution-model), [Partitionnement de données de cluster Redis](http://redis.io/topics/cluster-tutorial#redis-cluster-data-sharding) et [Balises de hachage de clés](http://redis.io/topics/cluster-spec#keys-hash-tags).

Pour accéder à un exemple de code relatif à l’utilisation du clustering et la localisation des clés dans une même partition avec le client StackExchange.Redis, consultez la partie [clustering.cs](https://github.com/rustd/RedisSamples/blob/master/HelloWorld/Clustering.cs) de l’exemple [Hello World](https://github.com/rustd/RedisSamples/tree/master/HelloWorld).

## Quelle est la taille de cache la plus grande que je peux créer ?

La plus grande taille de cache Premium est 53 Go. Vous pouvez créer jusqu’à 10 partitions, ce qui donne une taille maximale de 530 Go. Si vous avez besoin d’une plus grande taille, vous pouvez en [faire la demande](mailto:wapteams@microsoft.com?subject=Redis%20Cache%20quota%20increase). Pour plus d'informations, consultez [Tarification - Cache Redis Azure](https://azure.microsoft.com/pricing/details/cache/).

## Tous les clients Redis prennent-ils en charge le clustering ?

À l’heure actuelle, les clients ne prennent pas tous en charge le clustering Redis. StackExchange.Redis est l’un de ceux qui le prennent en charge. Pour plus d’informations sur d’autres clients, consultez la section [Playing with the cluster](http://redis.io/topics/cluster-tutorial#playing-with-the-cluster) du [didacticiel sur le cluster Redis](http://redis.io/topics/cluster-tutorial).

>[AZURE.NOTE] Si vous utilisez StackExchange.Redis comme client, assurez-vous d'utiliser la dernière version de [StackExchange.Redis](https://www.nuget.org/packages/StackExchange.Redis/) 1.0.481 ou une version ultérieure pour que le clustering fonctionne correctement. Si vous rencontrez des problèmes avec les exceptions MOVE, consultez la [section consacrée aux exceptions MOVE](#move-exceptions) pour plus d’informations.

## Comment puis-je me connecter à mon cache quand le clustering est activé ?

Vous pouvez vous connecter à votre cache à l’aide des [points de terminaison, ports et clés](cache-configure.md#properties) que vous utilisez pour vous connecter à un cache pour lequel le clustering n’est pas activé. Redis gère le clustering sur le serveur principal pour que vous n’ayez pas à le gérer à partir de votre client.

## Puis-je me connecter directement aux différentes partitions de mon cache ?

Cela n’est pas officiellement pris en charge. Ceci étant dit, chaque partition composée d’une paire de caches principal/réplica désignés collectivement sous le nom d’« instance de cache ». Vous pouvez vous connecter à ces instances de cache à l'aide de l'utilitaire redis-cli dans la branche [unstable](http://redis.io/download) du référentiel Redis sur GitHub. Cette version implémente la prise en charge de base lorsqu’elle est démarrée avec le commutateur `-c`. Pour plus d'informations, consultez [Playing with the cluster](http://redis.io/topics/cluster-tutorial#playing-with-the-cluster) sur [http://redis.io](http://redis.io) dans le [didacticiel de cluster Redis](http://redis.io/topics/cluster-tutorial).

Sans SSL, utilisez les commandes suivantes :

	Redis-cli.exe –h <<cachename>> -p 13000 (to connect to instance 0)
	Redis-cli.exe –h <<cachename>> -p 13001 (to connect to instance 1)
	Redis-cli.exe –h <<cachename>> -p 13002 (to connect to instance 2)
	...
	Redis-cli.exe –h <<cachename>> -p 1300N (to connect to instance N)

Avec SSL, remplacez `1300N` par `1500N`.

## Puis-je configurer le clustering pour un cache créé précédemment ?

Pour le moment, vous pouvez activer le clustering uniquement quand vous créez un cache. Vous pouvez modifier la taille du cluster une fois le cache créé, mais vous ne pouvez pas ajouter un clustering à un cache de niveau Premium ou supprimer le clustering d’un cache de niveau Premium une fois le cache créé. Un cache de niveau Premium avec clustering activé et une seule partition est différent d'un cache de niveau Premium de la même taille mais sans clustering.

## Puis-je configurer le clustering pour un cache De base ou Standard ?

Le clustering est disponible uniquement pour les caches de niveau Premium.

## Puis-je utiliser le clustering avec les fournisseurs d’état de session ASP.NET Redis et de mise en cache de la sortie ?

-	**Fournisseur de caches de sortie Redis** : aucune modification requise.
-	**Fournisseur d’état de session Redis** : pour utiliser le clustering, vous devez utiliser [RedisSessionStateProvider](https://www.nuget.org/packages/Microsoft.Web.RedisSessionStateProvider) version 2.0.1 ou ultérieure, sans quoi une exception est levée. Il s’agit d’une modification avec rupture. Pour plus d’informations, consultez [Détails de la modification avec rupture pour la version 2.0.0](https://github.com/Azure/aspnet-redis-providers/wiki/v2.0.0-Breaking-Change-Details).

## Étapes suivantes
Découvrez comment utiliser davantage de fonctionnalités de cache de niveau Premium.

-	[Comment configurer la persistance pour un Cache Redis Azure Premium](cache-how-to-premium-persistence.md)
-	[Comment configurer la prise en charge de réseau virtuel pour un Cache Redis Azure Premium](cache-how-to-premium-vnet.md)
  
<!-- IMAGES -->

[redis-cache-new-cache-menu]: ./media/cache-how-to-premium-clustering/redis-cache-new-cache-menu.png

[redis-cache-premium-pricing-tier]: ./media/cache-how-to-premium-clustering/redis-cache-premium-pricing-tier.png

[NewCacheMenu]: ./media/cache-how-to-premium-clustering/redis-cache-new-cache-menu.png

[CacheCreate]: ./media/cache-how-to-premium-clustering/redis-cache-cache-create.png

[redis-cache-premium-pricing-group]: ./media/cache-how-to-premium-clustering/redis-cache-premium-pricing-group.png

[redis-cache-premium-features]: ./media/cache-how-to-premium-clustering/redis-cache-premium-features.png

[redis-cache-clustering]: ./media/cache-how-to-premium-clustering/redis-cache-clustering.png

[redis-cache-clustering-selected]: ./media/cache-how-to-premium-clustering/redis-cache-clustering-selected.png

[redis-cache-redis-cluster-size]: ./media/cache-how-to-premium-clustering/redis-cache-redis-cluster-size.png

<!---------HONumber=AcomDC_0309_2016-->