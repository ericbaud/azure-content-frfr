<properties 
	pageTitle="Modélisation des données dans Azure DocumentDB | Microsoft Azure" 
	description="Obtenez plus d’informations sur la modélisation des données pour DocumentDB, une base de données de documents NoSQL." 
	keywords="modélisation des données"
	services="documentdb" 
	authors="ryancrawcour" 
	manager="jhubbard" 
	editor="mimig1" 
	documentationCenter=""/>

<tags 
	ms.service="documentdb" 
	ms.workload="data-services" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="02/16/2016" 
	ms.author="ryancraw"/>

#Modélisation des données dans DocumentDB#
Bien que les bases de données exemptes de schéma, comme DocumentDB, rendent très facile l'adoption des modifications apportées à votre modèle de données, vous devez quand même prendre le temps de réfléchir à vos données.

Comment les données seront-elles stockées ? Comment votre application va-t-elle récupérer et interroger des données ? Votre application exige-t-elle de nombreuses lectures (read heavy) ou de nombreuses écritures (write heavy) ?

Après avoir lu cet article, vous serez en mesure de répondre aux questions suivantes :

- Comment dois-je considérer un document dans une base de données de documents ?
- Qu'est-ce que la modélisation de données et pourquoi dois-je m'en soucier ? 
- En quoi la modélisation des données dans une base de données de documents et dans une base de données relationnelle diffère-t-elle ?
- Comment exprimer les relations entre les données dans une base de données non relationnelle ?
- Quand dois-je incorporer les données et quand dois-je créer un lien vers les données ?

##Incorporation de données##
Lorsque vous démarrez la modélisation des données dans une banque de documents telle que DocumentDB, essayez de traiter vos entités en tant que **documents autonomes** représentés dans JSON.

Avant d'aller trop loin, revenons quelques étapes en arrière et examinons comment nous pouvons modéliser un élément dans une base de données relationnelle. Beaucoup d'entre nous connaissent déjà le sujet. L'exemple suivant montre comment une personne peut être stockée dans une base de données relationnelle.

![Modèle de base de données relationnelle](./media/documentdb-modeling-data/relational-data-model.png)

Lorsqu'il s'agit de travailler avec des bases de données relationnelles, on nous a appris pendant des années qu'il fallait normaliser, normaliser, normaliser.

En général, la normalisation de vos données consiste à prendre une entité, une personne par exemple, et à la décomposer en éléments de données discrets. Dans l'exemple ci-dessus, une personne peut avoir plusieurs enregistrements de coordonnées, ainsi que plusieurs enregistrements d'adresse. Nous allons même plus loin et décomposons les coordonnées en extrayant des champs communs tels qu'un type. Même chose pour l’adresse : chaque enregistrement ici a un type, tel que *Home* ou *Business*.

Le principe directeur lors de la normalisation des données consiste à **éviter de stocker des données redondantes** dans chaque enregistrement et à faire plutôt référence aux données. Dans cet exemple, pour lire une personne, avec ses coordonnées et ses adresses, vous devez utiliser des jointures pour agréger efficacement vos données au moment de l'exécution.

	SELECT p.FirstName, p.LastName, a.City, cd.Detail
	FROM Person p
    JOIN ContactDetail cd ON cd.PersonId = p.Id
    JOIN ContactDetailType on cdt ON cdt.Id = cd.TypeId
    JOIN Address a ON a.PersonId = p.Id

La mise à jour d'une personne avec ses coordonnées et adresses nécessite des opérations d'écriture sur plusieurs tables individuelles.

Examinons à présent comment nous pourrions modéliser les mêmes données comme une entité autonome dans une base de données de documents.
		
	{
	    "id": "1",
	    "firstName": "Thomas",
	    "lastName": "Andersen",
	    "addresses": [
	        {            
	            "line1": "100 Some Street",
	            "line2": "Unit 1",
	            "city": "Seattle",
	            "state": "WA",
	            "zip": 98012
	        }
	    ],
	    "contactDetails": [
	        {"email: "thomas@andersen.com"},
	        {"phone": "+1 555 555-5555", "extension": 5555}
	    ] 
	}

Avec l’approche ci-dessus, nous avons maintenant **dénormalisé** l’enregistrement de la personne, où nous avons **incorporé** toutes les informations relatives à cette personne, telles que ses coordonnées et adresses, dans un seul document JSON. En outre, étant donné que nous ne sommes pas limités à un schéma fixe, nous avons la possibilité d'avoir des coordonnées de formes entièrement différentes.

La récupération d'un enregistrement complet de personne dans la base de données correspond désormais à une seule opération de lecture sur une collection unique et pour un document unique. La mise à jour d'un enregistrement de personne, avec ses coordonnées et adresses, correspond également à une seule opération d'écriture sur un document unique.

Avec la dénormalisation des données, votre application aura peut-être besoin d'émettre moins de requêtes et de mises à jour pour effectuer les opérations courantes.

###Quand utiliser l'incorporation

En général, utilisez des modèles de données incorporés dans les cas suivants :

- Il existe des relations de type **contient** entre des entités.
- Il existe des relations de type **un-à-plusieurs** entre des entités.
- Des données incorporées **changent rarement**.
- Des données incorporées ne croîtront pas **sans limite**.
- Des données incorporées sont une partie **intégrante** des données d’un document.

> [AZURE.NOTE] Normalement, les modèles de données dénormalisés offrent de meilleures performances en **lecture**.

###Quand éviter l'incorporation

Bien que la règle générale dans une base de données de documents soit de tout dénormaliser et d'incorporer toutes les données dans un seul document, cela peut déboucher sur des situations qui devraient être évitées.

Prenons cet extrait de code JSON.

	{
		"id": "1",
		"name": "What's new in the coolest Cloud",
		"summary": "A blog post by someone real famous",
		"comments": [
			{"id": 1, "author": "anon", "comment": "something useful, I'm sure"},
			{"id": 2, "author": "bob", "comment": "wisdom from the interwebs"},
			…
			{"id": 100001, "author": "jane", "comment": "and on we go ..."},
			…
			{"id": 1000000001, "author": "angry", "comment": "blah angry blah angry"},
			…
			{"id": ∞ + 1, "author": "bored", "comment": "oh man, will this ever end?"},
		]
	}

Une entité post avec commentaires incorporés pourrait avoir cet aspect si nous étions en train de modéliser un système de blog, ou CMS, classique. Dans cet exemple, le problème est que le tableau de commentaires est **illimité**, c’est-à-dire qu’il n’existe aucune limite (pratique) au nombre de commentaires possibles pour une publication. Cela posera un problème car la taille du document risque d'augmenter considérablement.

> [AZURE.TIP] Les documents dans DocumentDB ont une taille maximale. Pour plus d’informations, consultez la rubrique [Limites de DocumentDB](documentdb-limits.md).

L'augmentation de la taille du document a une incidence sur les possibilités de transmission des données par câble, ainsi que sur les possibilités de lecture et de mise à jour du document, à l'échelle.

Dans ce cas, il serait préférable de considérer le modèle suivant.
		
	Post document:
	{
		"id": 1,
		"name": "What's new in the coolest Cloud",
		"summary": "A blog post by someone real famous",
		"recentComments": [
			{"id": 1, "author": "anon", "comment": "something useful, I'm sure"},
			{"id": 2, "author": "bob", "comment": "wisdom from the interwebs"},
			{"id": 3, "author": "jane", "comment": "....."}
		]
	}

	Comment documents:
	{
		"postId": 1
		"comments": [
			{"id": 4, "author": "anon", "comment": "more goodness"},
			{"id": 5, "author": "bob", "comment": "tails from the field"},
			...
			{"id": 99, "author": "angry", "comment": "blah angry blah angry"}
		]
	},
	{
		"postId": 1
		"comments": [
			{"id": 100, "author": "anon", "comment": "yet more"},
			...
			{"id": 199, "author": "bored", "comment": "will this ever end?"}
		]
	}

Ce modèle présente les trois derniers commentaires incorporés dans la publication proprement dite, qui est un tableau avec une limite fixe cette fois-ci. Les autres commentaires sont regroupés par lots de 100 commentaires et stockés dans des documents distincts. 100 a été choisi comme taille de lot parce que notre application fictive permet à l'utilisateur de charger 100 commentaires à la fois.

Autre cas de figure où l'incorporation de données est déconseillée : lorsque les données incorporées sont souvent utilisées dans les documents et changent fréquemment.

Prenons cet extrait de code JSON.

	{
	    "id": "1",
	    "firstName": "Thomas",
	    "lastName": "Andersen",
	    "holdings": [
	        {
	            "numberHeld": 100,
	            "stock": { "symbol": "zaza", "open": 1, "high": 2, "low": 0.5 }
	        },
	        {
	            "numberHeld": 50,
	            "stock": { "symbol": "xcxc", "open": 89, "high": 93.24, "low": 88.87 }
	        }
	    ]
	}

Il pourrait représenter le portefeuille d'actions d'une personne. Nous avons choisi d'incorporer les informations boursières dans chaque document de portefeuille. Dans un environnement où les données associées changent fréquemment, comme une application de transactions boursières, incorporer les données qui changent fréquemment signifie que vous devez mettre à jour constamment chaque document de portefeuille, chaque fois que des actions sont échangées.

Des actions *zaza* peuvent être échangées des centaines de fois au cours d’une même journée, et des milliers d’utilisateurs peuvent posséder des actions *zaza* dans leur portefeuille. Avec un modèle de données comme le modèle ci-dessus, nous devons mettre à jour quotidiennement et à de nombreuses reprises des milliers de documents de portefeuille. Cela aboutit à un système qui n'est pas très évolutif.

##<a id="Refer"></a>Référencement des données##

Ainsi, l'incorporation de données fonctionne bien dans la plupart des cas, mais il est clair qu'il existe des scénarios où la dénormalisation de vos données provoque plus de problèmes qu'il n'en faudrait. Que faire, alors ?

Les bases de données relationnelles ne sont pas le seul endroit où vous pouvez créer des relations entre les entités. Dans une base de données de documents, vous pouvez avoir des informations dans un document qui sont en relation avec des données dans autres documents. Maintenant, je ne préconise absolument pas de créer des systèmes qui seraient mieux adaptés à une base de données relationnelle dans DocumentDB, ou toute autre base de données de documents, mais de simples relations conviennent et peuvent être très utiles.

Dans le code JSON ci-dessous, nous avons choisi d'utiliser l'exemple de portefeuille d'actions précédent, mais cette fois, nous faisons référence à l'action dans le portefeuille au lieu de l'incorporer. Ainsi, lorsque l'action change fréquemment au cours de la journée, le seul document à mettre à jour est le document d'action (stock).

    Person document:
    {
        "id": "1",
        "firstName": "Thomas",
        "lastName": "Andersen",
        "holdings": [
            { "numberHeld":  100, "stockId": 1},
            { "numberHeld":  50, "stockId": 2}
        ]
    }
	
    Stock documents:
    {
        "id": 1,
        "symbol": "zaza",
        "open": 1,
        "high": 2,
        "low": 0.5,
        "vol": 11970000,
        "mkt-cap": 42000000,
        "pe": 5.89
    },
    {
        "id": 2,
        "symbol": "xcxc",
        "open": 89,
        "high": 93.24,
        "low": 88.87,
        "vol": 2970200,
        "mkt-cap": 1005000,
        "pe": 75.82
    }
    

Cette approche présente cependant un inconvénient si votre application doit afficher des informations sur chaque action qui est conservée lors de l'affichage du portefeuille d'une personne ; dans ce cas, vous devez faire plusieurs aller et retour jusqu'à la base de données afin de charger les informations pour chaque document d'action. Ici, nous avons pris une décision pour améliorer l'efficacité des opérations d'écriture, qui ont lieu fréquemment pendant la journée, mais nous avons fait un compromis sur les opérations de lecture, qui ont potentiellement moins d'impact sur les performances de ce système.

> [AZURE.NOTE] Les modèles de données normalisés **peuvent nécessiter davantage d’aller-retour** jusqu’au serveur.

### Qu'en est-il des clés étrangères ?
Dans la mesure où il n'existe actuellement aucun concept d'une contrainte (clé étrangère ou autre), toutes les relations entre documents que vous avez dans les documents sont effectivement des « liens faibles » et elles ne sont pas vérifiées par la base de données. Si vous souhaitez vous assurer que les données auxquelles un document fait référence existent réellement, vous devez le faire dans votre application, ou en utilisant des déclencheurs côté serveur ou des procédures stockées sur DocumentDB.

###Quand utiliser des références
En général, utilisez des modèles de données normalisés dans les cas suivants :

- Représentation des relations **un-à-plusieurs**.
- Représentation des relations **plusieurs-à-plusieurs**.
- Les données associées **changent fréquemment**.
- Les données référencées peuvent être **illimitées**.

> [AZURE.NOTE] En général, la normalisation offre de meilleures performances en **écriture**.

###Où placer la relation ?
La croissance de la relation permet de déterminer dans quel document doit être stockée la référence.

Examinons le code JSON ci-dessous qui modélise des éditeurs et des livres :

	Publisher document:
	{
	    "id": "mspress",
	    "name": "Microsoft Press",
	    "books": [ 1, 2, 3, ..., 100, ..., 1000]
	}

	Book documents:
	{"id": 1, "name": "DocumentDB 101" }
	{"id": 2, "name": "DocumentDB for RDBMS Users" }
	{"id": 3, "name": "Taking over the world one JSON doc at a time" }
	...
	{"id": 100, "name": "Learn about Azure DocumentDB" }
	...
	{"id": 1000, "name": "Deep Dive in to DocumentDB" }

Si le nombre de livres par éditeur est peu élevé avec une croissance faible limitée, il peut être utile de stocker la référence du livre dans le document d'éditeur (publisher). Toutefois, si le nombre de livres par éditeur est illimité, ce modèle de données aboutira à des tableaux mutables, croissants, comme dans l'exemple de document d'éditeur ci-dessus.

Un petit changement donnera un modèle qui représente toujours les mêmes données, mais évite désormais ces grandes collections mutables.

	Publisher document: 
	{
	    "id": "mspress",
	    "name": "Microsoft Press"
	}
	
	Book documents: 
	{"id": 1,"name": "DocumentDB 101", "pub-id": "mspress"}
	{"id": 2,"name": "DocumentDB for RDBMS Users", "pub-id": "mspress"}
	{"id": 3,"name": "Taking over the world one JSON doc at a time"}
	...
	{"id": 100,"name": "Learn about Azure DocumentDB", "pub-id": "mspress"}
	...
	{"id": 1000,"name": "Deep Dive in to DocumentDB", "pub-id": "mspress"}

Dans l'exemple ci-dessus, nous avons supprimé la collection illimitée dans le document d'éditeur (publisher). Nous avons simplement une référence à l'éditeur dans chaque document de livre (book).

###Comment modéliser des relations plusieurs-à-plusieurs ?
Dans une base de données relationnelle *plusieurs-à-plusieurs*, les relations sont souvent modélisées avec des tables de jointure qui relient simplement les enregistrements d’autres tables.

![Tables de jointures](./media/documentdb-modeling-data/join-table.png)

Vous pouvez être tenté de répliquer la même chose à l'aide de documents et de générer un modèle de données qui ressemble à ce qui suit.

	Author documents: 
	{"id": 1, "name": "Thomas Andersen" }
	{"id": 2, "name": "William Wakefield" }
	
	Book documents:
	{"id": 1, "name": "DocumentDB 101" }
	{"id": 2, "name": "DocumentDB for RDBMS Users" }
	{"id": 3, "name": "Taking over the world one JSON doc at a time" }
	{"id": 4, "name": "Learn about Azure DocumentDB" }
	{"id": 5, "name": "Deep Dive in to DocumentDB" }
	
	Joining documents: 
	{"authorId": 1, "bookId": 1 }
	{"authorId": 2, "bookId": 1 }
	{"authorId": 1, "bookId": 2 }
	{"authorId": 1, "bookId": 3 }

Cette méthode fonctionne. Toutefois, le fait de charger soit un auteur avec ses livres soit un livre avec son auteur nécessite toujours au moins deux requêtes supplémentaires sur la base de données. Une requête pour le document de jointure (joining) et une autre requête pour extraire le document joint.

Si cette table de jointure ne fait rien d'autre que coller ensemble deux éléments de données, pourquoi ne pas la supprimer complètement ? Examinons le code suivant.

	Author documents:
	{"id": 1, "name": "Thomas Andersen", "books": [1, 2, 3]}
	{"id": 2, "name": "William Wakefield", "books": [1, 4]}
	
	Book documents: 
	{"id": 1, "name": "DocumentDB 101", "authors": [1, 2]}
	{"id": 2, "name": "DocumentDB for RDBMS Users", "authors": [1]}
	{"id": 3, "name": "Learn about Azure DocumentDB", "authors": [1]}
	{"id": 4, "name": "Deep Dive in to DocumentDB", "authors": [2]}

Maintenant, si j'ai un auteur, je saurai immédiatement quels livres il a écrits, et inversement, si j'ai un document de livre (book) chargé, je connaîtrai le ou les ID des auteurs. Cela permet de faire l'économie de cette requête intermédiaire sur la table de jointure en réduisant le nombre d'aller et retour jusqu'au serveur pour votre application.

##<a id="WrapUp"></a>Modèles de données hybrides##
Nous savons maintenant que l'incorporation (ou la dénormalisation) et le référencement (ou la normalisation) des données ont tous deux leurs avantages, mais qu'ils impliquent également des compromis.

Vous n'êtes pas toujours obligé de choisir soit l'un soit l'autre. N'ayez pas peur de combiner les deux.

En fonction des modèles d'utilisation et des charges de travail spécifiques de votre application, dans certains cas, la combinaison de données incorporées et de données référencées peut s'avérer intéressante et conduire à une logique d'application plus simple, avec moins d'aller et retour jusqu'au serveur, tout en préservant un bon niveau de performances.

Examinons le code JSON suivant.

	Author documents: 
	{
	    "id": 1,
	    "firstName": "Thomas",
	    "lastName": "Andersen",		
	    "countOfBooks": 3,
	 	"books": [1, 2, 3],
		"images": [
			{"thumbnail": "http://....png"}
			{"profile": "http://....png"}
			{"large": "http://....png"}
		]
	},
	{
	    "id": 2,
	    "firstName": "William",
	    "lastName": "Wakefield",
	    "countOfBooks": 1,
		"books": [1, 4, 5],
		"images": [
			{"thumbnail": "http://....png"}
		]
	}
	
	Book documents:
	{
		"id": 1,
		"name": "DocumentDB 101",
		"authors": [
			{"id": 1, "name": "Thomas Andersen", "thumbnailUrl": "http://....png"},
			{"id": 2, "name": "William Wakefield", "thumbnailUrl": "http://....png"}
		]
	},
	{
		"id": 2,
		"name": "DocumentDB for RDBMS Users",
		"authors": [
			{"id": 1, "name": "Thomas Andersen", "thumbnailUrl": "http://....png"},
		]
	}

Ici nous avons suivi (principalement) le modèle incorporé, où les données des autres entités sont incorporées dans le document de niveau supérieur, mais les autres données sont référencées.

Dans le document de livre (book), nous pouvons voir quelques champs intéressants lorsque nous examinons le tableau des auteurs. Il existe un champ *id* que nous utilisons pour faire référence à un document d’auteur (author), une pratique courante dans un modèle normalisé, mais nous avons également les champs *name* et *thumbnailUrl*. Nous aurions pu nous arrêter à l’*id* et laisser l’application obtenir les informations supplémentaires dont elle avait besoin à partir du document d’auteur (author) respectif à l’aide du « lien », mais comme notre application affiche le nom de l’auteur et une image miniature avec chaque livre, nous pouvons économiser un aller-retour par livre jusqu’au serveur en dénormalisant **certaines** données de l’auteur.

Bien sûr, si le nom de l'auteur changeait ou qu'il souhaitait mettre à jour sa photo, nous devrions procéder à une mise à jour sur chaque livre publié par lui ; mais pour notre application, si l'on se base sur l'hypothèse que les auteurs ne changent pas de nom très souvent, il s'agit d'une décision de conception acceptable.

Dans cet exemple, il existe des valeurs d’**agrégats précalculés** pour économiser un traitement coûteux sur une opération de lecture. Dans l'exemple, certaines données incorporées dans le document d'auteur (author) sont des données calculées au moment de l'exécution. À chaque publication d’un nouveau livre, un document de type livre est créé **et** le champ countOfBooks est défini sur une valeur calculée en fonction du nombre de documents de type livre existant pour un auteur particulier. Cette optimisation serait appropriée dans les systèmes qui exigent de nombreuses lectures (read heavy), où nous pouvons nous permettre d'effectuer des calculs sur les écritures afin d'optimiser les lectures.

L’existence d’un modèle avec des champs précalculés est possible, car DocumentDB prend en charge les **transactions multi-documents**. De nombreuses boutiques NoSQL ne peuvent pas effectuer des transactions à travers plusieurs documents et plaident par conséquent en faveur de décisions de conception, telles que « incorporer tout systématiquement », en raison de cette limitation. Avec DocumentDB, vous pouvez utiliser des déclencheurs côté serveur, ou des procédures stockées, qui insèrent des livres et mettent à jour les auteurs au sein d'une transaction ACID. Aujourd’hui, vous n’êtes pas **tenu** d’intégrer tous les éléments dans un document, simplement pour vous assurer que vos données restent cohérentes.

##<a name="NextSteps"></a>Étapes suivantes

Comprendre que la modélisation des données dans un monde sans schéma reste aussi importante que jamais, telle est la principale leçon à tirer de cet article.

De même qu'il existe plusieurs façons de représenter un élément de données sur un écran, il existe plusieurs manières de modéliser vos données. Vous devez comprendre votre application et comment elle produira, utilisera et traitera les données. Ensuite, en appliquant certaines des instructions présentées ici, vous pouvez entreprendre de créer un modèle qui répond aux besoins immédiats de votre application. Lorsque vos applications doivent changer, vous pouvez exploiter la flexibilité d'une base de données sans schéma pour adopter ce changement et développer facilement votre modèle de données.

Pour en savoir plus sur Azure DocumentDB, consultez la page de [documentation](https://azure.microsoft.com/documentation/services/documentdb/) du service.

Pour plus d’informations sur le paramétrage des index dans Azure DocumentDB, consultez l’article portant sur les [stratégies d’indexation](documentdb-indexing-policies.md).

Pour comprendre la répartition de vos données entre plusieurs partitions, consultez la rubrique [Partitionnement des données dans DocumentDB](documentdb-partition-data.md).

Et enfin, pour obtenir des conseils sur la modélisation des données et le partitionnement pour les applications mutualisées, consultez l’article [Mise à l’échelle d’une application mutualisée avec Azure DocumentDB](http://blogs.msdn.com/b/documentdb/archive/2014/12/03/scaling-a-multi-tenant-application-with-azure-documentdb.aspx).
 

<!---HONumber=AcomDC_0218_2016-->