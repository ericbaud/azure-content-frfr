<properties 
	pageTitle="Référence d’Analytics dans Application Insights" 
	description="Référence pour les instructions dans Analytics, le puissant outil de recherche d’Application Insights." 
	services="application-insights" 
    documentationCenter=""
	authors="alancameronwills" 
	manager="douge"/>

<tags 
	ms.service="application-insights" 
	ms.workload="tbd" 
	ms.tgt_pltfrm="ibiza" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="03/30/2016" 
	ms.author="awills"/>

# Référence pour Analytics

[Analytics](app-insights-analytics.md) est la puissante fonctionnalité de recherche d’[Application Insights](app-insights-overview.md). Ces pages décrivent le langage de requête Analytics.

*Utilisez la fonctionnalité de recherche de votre navigateur pour rechercher des éléments de langage dans cette page, qui combine le contenu des pages Requêtes, Agrégations et Valeurs scalaires.*

[AZURE.INCLUDE [app-insights-analytics-top-index](../../includes/app-insights-analytics-top-index.md)]



## Requêtes et opérateurs

Une requête sur vos données de télémétrie est constituée d’une référence à un flux source, suivie d’un pipeline de filtres. Par exemple :


```AIQL
requests
| where client_City == "London" and timestamp > ago(3d)
| count
```
    
Chaque filtre précédé de la barre verticale `|` est une instance d’un *opérateur*, assortie de certains paramètres. L’entrée de l’opérateur est la table résultant du pipeline précédent. Dans la plupart des cas, tous les paramètres sont des [expressions scalaires](##scalars) sur les colonnes de l’entrée. Dans certains cas, les paramètres correspondent aux noms des colonnes d’entrée et, parfois, le paramètre est une seconde table. Le résultat d’une requête est toujours une table, même si elle ne contient qu’une colonne et qu’une ligne.

Une requête peut être précédée d’une ou plusieurs [clauses Let](#let-clause), qui définissent les valeurs scalaires, les tables ou les fonctions pouvant être utilisées dans la requête.

```AIQL

    let interval = 3d ;
    let city = "London" ;
    let req = (city:string) {
      requests
      | where client_City == city and timestamp > ago(interval) };
    req(city) | count
```

> `T` est utilisé dans les exemples de requête ci-dessous pour désigner la table source ou le pipeline précédent.
> 

### opérateur count

L’opérateur `count` retourne le nombre d’enregistrements (lignes) dans le jeu d’enregistrements d’entrée.

**Syntaxe**

    T | count

**Arguments**

* *T* : données tabulaires dont les enregistrements doivent être comptabilisés.

**Retourne**

Cette fonction retourne une table contenant un seul enregistrement et une colonne de type `long`. La valeur de l’unique cellule correspond au nombre d’enregistrements dans *T*.

**Exemple**

```AIQL
requests | count
```



### opérateur extend

     T | extend duration = stopTime - startTime

Ajoute une ou plusieurs colonnes calculées à une table.


**Syntaxe**

    T | extend ColumnName = Expression [, ...]

**Arguments**

* *T :* table d’entrée.
* *ColumnName :* nom de la colonne à ajouter. 
* *Expression :* calcul sur les colonnes existantes.

**Retourne**

Une copie de la table d’entrée, avec les colonnes supplémentaires spécifiées.

**Conseils**

* Utilisez plutôt [`project`](#project-operator) si vous souhaitez également supprimer ou renommer certaines colonnes.
* N’utilisez pas `extend` si vous souhaitez simplement obtenir un nom plus court à utiliser dans une expression longue. `...| extend x = anonymous_user_id_from_client | ... func(x) ...` 

    Les colonnes d’origine de la table ont été indexées. Votre nouveau nom définit une colonne supplémentaire qui n’est pas indexée, c’est pourquoi la requête risque de s’exécuter plus lentement.

**Exemple**

```AIQL
traces
| extend
    Age = now() - timestamp
```


### opérateur join

    Table1 | join (Table2) on CommonColumn

Fusionne les lignes de deux tables en faisant correspondre les valeurs de la colonne spécifiée.


**Syntaxe**

    Table1 | join [kind=Kind] (Table2) on CommonColumn [, ...]

**Arguments**

* *Table1* : « côté gauche » de la jointure.
* *Table2* : « côté droit » de la jointure. Il peut s’agir d’une expression de requête imbriquée générant une table.
* *CommonColumn* : colonne portant le même nom dans les deux tables.
* *Kind* : spécifie la façon dont les lignes des deux tables doivent être mises en correspondance.

**Retourne**

Une table avec :

* Une colonne pour chaque colonne dans chacune des deux tables, y compris les clés correspondantes. Les colonnes du côté droit seront automatiquement renommées en cas de conflit de nom.
* Une ligne pour chaque correspondance entre les tables d’entrée. Une correspondance est une ligne sélectionnée dans une table ayant la même valeur pour tous les champs `on` qu’une ligne dans l’autre table. 

* `Kind` non spécifié

    Une seule ligne du côté gauche est mise en correspondance pour chaque valeur de la clé `on`. La sortie contient une ligne pour chaque correspondance de cette ligne avec des lignes du côté droit.

* `Kind=inner`
 
     La sortie contient une ligne pour chaque combinaison de lignes correspondantes des côtés gauche et droit.

* `kind=leftouter` (ou `kind=rightouter` ou `kind=fullouter`)

     Outre les correspondances internes, on trouve une ligne pour chaque ligne du côté gauche (et/ou droit), même s’il n’y a aucune correspondance. Dans ce cas, les cellules de sortie sans correspondance contiennent des valeurs null.

* `kind=leftanti`

     Retourne tous les enregistrements du côté gauche n’ayant pas de correspondance du côté droit. La table de résultats ne comporte que les colonnes du côté gauche.
 
Si plusieurs lignes comportent les mêmes valeurs pour ces champs, des lignes s’affichent pour toutes les combinaisons.

**Conseils**

Pour un résultat optimal :

* Utilisez `where` et `project` pour réduire le nombre de lignes et de colonnes dans les tables d’entrée, avant le `join`. 
* Si une table est toujours plus petite que l’autre, utilisez-la pour le côté gauche de la jointure.
* Les colonnes de la correspondance de jointure doivent avoir le même nom. Utilisez l’opérateur project si nécessaire pour renommer une colonne dans l’une des tables.

**Exemple**

Obtenez les activités étendues d’un journal dans lequel certaines entrées marquent le début et la fin d’une activité.

```AIQL
    let Events = MyLogTable | where type=="Event" ;
    Events
    | where Name == "Start"
    | project Name, City, ActivityId, StartTime=timestamp
    | join (Events
           | where Name == "Stop"
           | project StopTime=timestamp, ActivityId)
        on ActivityId
    | project City, ActivityId, StartTime, StopTime, Duration, StopTime, StartTime

```

### clause let

**Clause let tabulaire - nommer une table**

    let recentReqs = requests | where timestamp > ago(3d); 
    recentReqs | count

**Clause let scalaire - nommer une valeur**

    let interval = 3d; 
    requests | where timestamp > ago(interval)

**Clause let lambda - nommer une fonction**

    let Recent = 
       (interval:timespan) { requests | where timestamp > ago(interval) };
    Recent(3h) | count

Une clause let associe un nom à un résultat tabulaire, une valeur scalaire ou une fonction. La clause est un préfixe d’une requête tandis que la portée de la liaison correspond à cette requête. (La clause let ne permet pas de nommer des objets que vous pourrez utiliser ultérieurement dans votre session.)

**Syntaxe**

    let name = scalar_constant_expression ; query

    let name = query ; query

    let name = (parameterName : type [, ...]) { plain_query }; query

* *type :* `bool`, `int`, `long`, `double`, `string`, `timespan`, `datetime`, `guid`, [`dynamic`](#dynamic-type)
* *plain\_query :* requête non précédée d’une clause let.

**Exemples**




    let rows(n:long) = range steps from 1 to n step 1;
    rows(10) | ...


Jointure réflexive :

    let Recent = events | where timestamp > ago(7d);
    Recent | where name contains "session_started" 
    | project start = timestamp, session_id
    | join (Recent 
        | where name contains "session_ended" 
        | project stop = timestamp, session_id)
      on session_id
    | extend duration = stop - start 

### opérateur limit

     T | limit 5

Retourne au maximum le nombre spécifié de lignes de la table d’entrée. Il n’est pas possible de savoir quels enregistrements vont être retournés. (Pour retourner des enregistrements spécifiques, utilisez [`top`](#top-operator).)

**Alias** `take`

**Syntaxe**

    T | limit NumberOfRows


**Conseils**

`Take` est un moyen simple et efficace d’afficher un aperçu des résultats lorsque vous travaillez de manière interactive. N’oubliez pas qu’il ne garantit pas de produire des lignes spécifiques ou de les afficher dans un ordre particulier.

Il existe une limite implicite quant au nombre de lignes retournées au client, même si vous n’utilisez pas `take`. Pour lever cette limite, utilisez l’option de requête client `notruncation`.



### opérateur mvexpand

    T | mvexpand listColumn 

Développe une liste à partir d’une cellule de type dynamique (JSON) afin que chaque entrée comporte une ligne distincte. Toutes les autres cellules d’une ligne développée sont dupliquées.

(Voir aussi [`summarize makelist`](#summarize-operator) qui effectue la fonction inverse.)

**Exemple**

En partant du principe que la table d’entrée est :

|A:int|B:string|D:dynamic|
|---|---|---|
|1|"hello"|{"key":"value"}|
|2|"world"|[0,1,"k","v"]|

    mvexpand D

Voici le résultat :

|A:int|B:string|D:dynamic|
|---|---|---|
|1|"hello"|{"key":"value"}|
|2|"world"|0|
|2|"world"|1|
|2|"world"|"k"|
|2|"world"|"v"|


**Syntaxe**

    T | mvexpand  [bagexpansion=(bag | array)] ColumnName [limit Rowlimit]

    T | mvexpand  [bagexpansion=(bag | array)] [Name =] ArrayExpression [to typeof(Typename)] [limit Rowlimit]

**Arguments**

* *ColumnName :* dans le résultat, les tableaux dans la colonne nommée sont développés en plusieurs lignes. 
* *ArrayExpression :* expression produisant un tableau. Si ce formulaire est utilisé, une nouvelle colonne est ajoutée et la colonne existante est conservée.
* *Name :* nom de la nouvelle colonne.
* *Typename :* applique l’expression développée à un type spécifique.
* *RowLimit :* nombre maximal de lignes générées à partir de chaque ligne d’origine. La valeur par défaut est 128.

**Retourne**

Plusieurs lignes pour chacune des valeurs dans n’importe quel tableau dans la colonne nommée ou dans l’expression de tableau.

La colonne développée est toujours de type dynamique. Utilisez un transtypage tel que `todatetime()` ou `toint()` si vous souhaitez calculer ou agréger des valeurs.

Deux modes de développement de conteneurs de propriétés sont pris en charge :

* `bagexpansion=bag` : les conteneurs de propriétés sont développés en conteneurs de propriété à entrée unique. Il s’agit du développement par défaut.
* `bagexpansion=array` : les conteneurs de propriétés sont développés dans des structures de tableau `[`*clé*`,`*valeur*`]` à deux éléments, ce qui permet un accès uniforme aux clés et valeurs (comme, par exemple, effectuer une agrégation de comptage de valeurs sur les noms de propriété). 

**Exemples**


    exceptions | take 1 
    | mvexpand details[0]

Fractionne un enregistrement d’exception en plusieurs lignes pour chaque élément du champ de détails.



### opérateur parse

    T | parse "I am 63 next birthday" with "I am" Year:int "next birthday"

    T | parse kind=regex "My 62nd birthday" 
        with "My" Year:regex("[0..9]+") regex("..") "birthday"

Extrait les valeurs d’une chaîne. Peut utiliser une correspondance d’expression simple ou régulière.

Les éléments dans la clause `with` sont ensuite mis en correspondance avec la chaîne source. Chaque élément traite un fragment du texte source. S’il s’agit d’une chaîne simple, le curseur correspondant se déplace aussi loin que la correspondance. S’il s’agit d’une colonne avec un nom de type, le curseur se déplace suffisamment loin pour analyser le type spécifié. (Les correspondances de chaîne se déplacent jusqu’à ce qu’une correspondance pour l’élément suivant soit identifiée.) S’il s’agit d’une expression régulière, elle est mis en correspondance (et la colonne en résultant a toujours le type chaîne).

**Syntaxe**

    T | parse StringExpression with [SimpleMatch | Column:Type] ...

    T | parse kind=regex StringExpression 
        with [SimpleMatch | Column : regex("Regex")] ...

**Arguments**

* *T :* table d’entrée.
* *kind :* simple ou expression régulière. La valeur par défaut est simple.
* *StringExpression :* expression pouvant être convertie en chaîne ou en prendre la valeur.
* *SimpleMatch :* chaîne correspondant à la partie suivante du texte.
* *Column :* spécifie la nouvelle colonne à laquelle attribuer une correspondance.
* *Type :* détermine comment analyser la partie suivante de la chaîne source.
* *Regex :* expression régulière pour faire correspondre la partie suivante de la chaîne. 

**Retourne**

La table d’entrée, étendue en fonction de la liste des colonnes.


**Exemples**

L’opérateur `parse` offre un moyen simple de `extend` une table en utilisant plusieurs applications `extract` sur la même expression `string`. Il est particulièrement utile quand la table a une colonne `string` contenant plusieurs valeurs à répartir en différentes colonnes, par exemple une colonne qui a été générée par une instruction de trace de développeur (« `printf` »/« `Console.WriteLine` »).

Dans l’exemple suivant, partons du principe que la colonne `EventNarrative` de la table `StormEvents` contient des chaînes sous la forme `{0} at {1} crested at {2} feet around {3} on {4} {5}`. L’opération ci-dessous va étendre la table avec deux colonnes : `SwathSize` et `FellLocation`.


|EventNarrative|
|---|
|La rivière Green River à Brownsville est montée à 18,8 pieds aux alentours de 9 h 30 EST le 12 décembre. À Brownsville, la côte d’inondation est fixée à 18 pieds. À ce niveau, on parle d’une inondation mineure. La rivière sort de son lit et inonde une partie des berges basses ainsi que certaines terres agricoles.|
|La rivière Rolling Fork River à Boston est montée à 39,3 pieds aux alentours de 17 h 00 EST le 12 décembre. À Boston, la côte d’inondation est fixée à 35 pieds. À ce niveau, on parle d’une inondation mineure et certaines terres agricoles commencent à être immergées.|
|La rivière Green River à Woodbury est montée à 36,7 pieds aux alentours de 6 h 00 EST le 16 décembre. À Woodbury, la côte d’inondation est fixée à 33 pieds. À ce niveau, on parle d’une inondation mineure et certaines plaines autour de la ville de Woodbury commencent à être immergées.|
|La rivière Ohio River à Tell City est montée à 39,0 pieds aux alentours de 7 h 00 EST le 18 décembre. À Tell City, la côte d’inondation est fixée à 38 pieds. À ce niveau, la rivière commence à déborder et à inonder ses berges. L’autoroute Indiana Highway 66 est inondée entre Rome et Derby.|

```AIQL

StormEvents 
|  parse EventNarrative 
   with RiverName:string 
        "at" 
        Location:string 
        "crested at" 
        Height:double  
        "feet around" 
        Time:string 
        "on" 
        Month:string 
        " " 
        Day:long 
        "." 
        notImportant:string
| project RiverName , Location , Height , Time , Month , Day

```

|RiverName|Emplacement|Hauteur|Time|Mois|jour|
|---|---|---|---|---|---|
|Green River | Woodbury |36,7| 6 h 00 EST | Décembre|16|
|Rolling Fork River | Boston |39,3| 17 h 00 EST | Décembre|12|
|Green River | Brownsville |18,8| 9 h 30 EST | Décembre|12|
|Ohio River | Tell City |39| 7 h 00 EST | Décembre|18|

Il est également possible d’établir une correspondance à l’aide d’expressions régulières. Le résultat est le même mais toutes les colonnes de résultat ont le type chaîne :

```AIQL

StormEvents
| parse kind=regex EventNarrative 
  with RiverName:regex("(\\s?[a-zA-Z]+\\s?)+") 
  "at" Location:regex(".*") 
  "crested at " Height:regex("\\d+\\.\\d+") 
  " feet around" Time:regex(".*") 
  "on " Month:regex("(December|November|October)") 
   " " Day:regex("\\d+") 
   "." notImportant:regex(".*")
| project RiverName , Location , Height , Time , Month , Day
```


### opérateur project

    T | project cost=price*quantity, price

Sélectionnez les colonnes à inclure, renommer ou supprimer, puis insérez les nouvelles colonnes calculées. L’ordre des colonnes dans le résultat est déterminé par l’ordre des arguments. Seules les colonnes spécifiées dans les arguments sont incluses dans le résultat : les autres sont supprimées. (Voir aussi `extend`.)


**Syntaxe**

    T | project ColumnName [= Expression] [, ...]

**Arguments**

* *T :* table d’entrée.
* *ColumnName :* nom d’une colonne à afficher dans la sortie. S’il n’y a aucune *Expression*, une colonne portant ce nom doit apparaître dans l’entrée. 
* *Expression :* expression scalaire facultative faisant référence aux colonnes de l’entrée. 

    Il est possible de retourner une nouvelle colonne calculée portant le même nom qu’une colonne figurant dans l’entrée.

**Retourne**

Une table contenant les colonnes nommées en tant qu’arguments, et autant de lignes que la table d’entrée.

**Exemple**

L’exemple suivant présente plusieurs types de manipulations pouvant être effectuées à l’aide de l’opérateur `project`. La table d’entrée `T` a trois colonnes de type `int` : `A`, `B` et `C`.

```AIQL
T
| project
    X=C,                       // Rename column C to X
    A=2*B,                     // Calculate a new column A from the old B
    C=strcat("-",tostring(C)), // Calculate a new column C from the old C
    B=2*B                      // Calculate a new column B from the old B
```



### opérateur range

    range LastWeek from ago(7d) to now() step 1d

Génère une table de valeurs à une seule colonne. Notez qu’elle ne comporte pas d’entrée de pipeline.

|LastWeek|
|---|
|2015-12-05 09:10:04.627|
|2015-12-06 09:10:04.627|
|...|
|2015-12-12 09:10:04.627|



**Syntaxe**

    range ColumnName from Start to Stop step Step

**Arguments**

* *ColumnName :* nom de la colonne unique dans la table de sortie.
* *Start :* plus petite valeur de la sortie.
* *Stop :* valeur la plus élevée générée dans la sortie (ou une limite pour la valeur la plus élevée, si *Step* dépasse cette valeur).
* *Step :* différence entre deux valeurs consécutives. 

Les arguments doivent être des valeurs de type numérique, date ou durée. Ils ne peuvent pas faire référence aux colonnes d’une table. (Si vous souhaitez calculer la plage d’après une table d’entrée, utilisez la [fonction *range*](#range), éventuellement avec l’[opérateur mvexpand](#mvexpand-operator).)

**Retourne**

Une table comportant une seule colonne nommée *ColumnName*, dont les valeurs sont *Start*, *Start* + *Step*, ... jusqu’à *Stop* inclus.

**Exemple**

```AIQL
range Steps from 1 to 8 step 3
```

Une table comportant une seule colonne nommée `Steps` dont le type est `long` et dont les valeurs sont `1`, `4` et `7`.

**Exemple**

    range LastWeek from bin(ago(7d),1d) to now() step 1d

Table des occurrences de minuit au cours des sept derniers jours. La fonction d’emplacement (floor) diminue à chaque fois au début de la journée.

**Exemple**

```AIQL
range timestamp from ago(4h) to now() step 1m
| join kind=fullouter
  (traces
      | where timestamp > ago(4h)
      | summarize Count=count() by bin(timestamp, 1m)
  ) on timestamp
| project Count=iff(isnull(Count), 0, Count), timestamp
| render timechart  
```

Montre comment utiliser l’opérateur `range` pour créer une petite table de dimension ad hoc qui est ensuite utilisée pour intégrer des zéros là où les données sources ne contiennent aucune valeur.

### opérateur reduce

    exceptions | reduce by outerMessage

Tente de regrouper des enregistrements similaires. Pour chaque groupe, l’opérateur génère le `Pattern` décrivant le mieux ce groupe et le `Count` d’enregistrements dans ce groupe.


![](./media/app-insights-analytics-queries/reduce.png)

**Syntaxe**

    T | reduce by  ColumnName [ with threshold=Threshold ]

**Arguments**

* *ColumnName :* colonne à examiner. Cette valeur doit être de type chaîne.
* *Threshold :* valeur dans la plage {0..1}. La valeur par défaut est 0,001. Pour les entrées volumineuses, le seuil doit être bas. 

**Retourne**

Deux colonnes, `Pattern` et `Count`. Dans de nombreux cas, le modèle est une valeur complète de la colonne. Dans certains cas, il peut identifier des termes courants et remplacer les parties variables avec '*'.

Par exemple, le résultat de `reduce by city` peut inclure :

|Modèle | Nombre |
|---|---|
| San * | 5182 |
| Saint * | 2846 |
| Moscow | 3726 |
| * -on- * | 2730 |
| Paris | 27163 |


### directive render

    T | render [ table | timechart  | barchart | piechart ]

Render indique à la couche de présentation comment afficher la table. Il doit s’agir du dernier élément du canal. C’est une alternative pratique à l’utilisation des commandes affichées, ce qui vous permet d’enregistrer une requête avec une méthode de présentation particulière.


### opérateur sort 

    T | sort by country asc, price desc

Trie les lignes de la table d’entrée dans l’ordre d’après une ou plusieurs colonnes.

**Alias** `order`

**Syntaxe**

    T  | sort by Column [ asc | desc ] [ `,` ... ]

**Arguments**

* *T :* table d’entrée à trier.
* *Column :* colonne de *T* à laquelle appliquer le tri. Les valeurs doivent être de type numérique, date, heure ou chaîne.
* `asc` Tri par ordre croissant, de faible à élevé. La valeur par défaut est `desc`, par ordre décroissant allant d’élevé à faible.

**Exemple**

```AIQL
Traces
| where ActivityId == "479671d99b7b"
| sort by Timestamp asc
```
Toutes les lignes dans la table Traces ayant un `ActivityId` spécifique, triées d’après leur horodatage.

### opérateur summarize

Génère une table qui agrège le contenu de la table d’entrée.
 
    requests
	| summarize count(), avg(duration), makeset(client_City) 
      by client_CountryOrRegion

Une table qui indique le nombre, la durée moyenne des requêtes et un ensemble de villes dans chaque pays. La sortie comporte une ligne pour chaque pays. Les colonnes de sortie affichent le nombre, la durée moyenne, les villes et le pays. Toutes les autres colonnes d’entrée sont supprimées.


    T | summarize count() by price_range=bin(price, 10.0)

Une table indiquant le nombre d’éléments ayant un prix dans chaque intervalle [0,10.0], [10.0,20.0] et ainsi de suite. Cet exemple comprend une colonne pour le nombre et une autre pour la gamme de prix. Toutes les autres colonnes d’entrée sont supprimées.


**Syntaxe**

    T | summarize
         [  [ Column = ] Aggregation [ `,` ... ] ]
         [ by
            [ Column = ] GroupExpression [ `,` ... ] ]

**Arguments**

* *Column :* nom facultatif d’une colonne de résultats. Prend par défaut un nom dérivé de l’expression. 
* *Aggregation :* appel d’une fonction d’agrégation telle que `count()` ou `avg()`, avec des noms de colonnes comme arguments. Voir [Aggrégations](#aggregations).
* *GroupExpression :* expression sur les colonnes, qui fournit un ensemble de valeurs distinctes. En général, il s’agit d’un nom de colonne qui fournit déjà un ensemble restreint de valeurs, ou de `bin()` avec une colonne numérique ou de temps en tant qu’argument. 

Si vous fournissez une expression numérique ou de temps sans utiliser `bin()`, Analytics l’applique automatiquement avec un intervalle de `1h` pour les heures ou de `1.0` pour les nombres.

Si vous ne fournissez pas de *GroupExpression*, la table entière est résumée dans une ligne de sortie unique.



**Retourne**

Les lignes d’entrée sont organisées en groupes ayant les mêmes valeurs que les expressions `by`. Ensuite, les fonctions d’agrégation spécifiées sont calculées sur chaque groupe, générant une ligne pour chaque groupe. Le résultat contient les colonnes `by` et au moins une colonne pour chaque agrégation calculée. (Certaines fonctions d’agrégation retournent plusieurs colonnes.)

Le résultat contient autant de lignes qu’il existe de combinaisons de valeurs `by`. Si vous souhaitez générer une synthèse sur des plages de valeurs numériques, utilisez `bin()` pour limiter les plages à des valeurs discrètes.

**Remarque :**

Bien que vous puissiez fournir des expressions arbitraires pour les expressions d’agrégation et de regroupement, il est plus efficace d’utiliser des noms de colonnes simples ou d’appliquer `bin()` à une colonne numérique.



### opérateur take

Alias de [limite](#limit-operator)


### opérateur top

    T | top 5 by Name desc

Retourne les *N* premiers enregistrements triés d’après les colonnes spécifiées.


**Syntaxe**

    T | top NumberOfRows by Sort_expression [ `asc` | `desc` ] [, ... ]

**Arguments**

* *NumberOfRows :* nombre de lignes de *T* à retourner.
* *Sort\_expression :* expression utilisée pour trier les lignes. Il s’agit généralement juste d’un nom de colonne. Vous pouvez spécifier plusieurs expressions sort\_expression.
* `asc` ou `desc` (la valeur par défaut) peut permettre d’indiquer si la sélection est effectuée à partir du « bas » ou du « haut » de la plage.


**Conseils**

`top 5 by name` équivaut approximativement à `sort by name | take 5`. Toutefois, cet opérateur s’exécute plus rapidement et retourne toujours les résultats triés, contrairement à `take`.


### opérateur union

     Table1 | union Table2, Table3

Prend deux tables ou plus et retourne les lignes de toutes les tables.

**Syntaxe**

    T | union [ kind= inner | outer ] [ withsource = ColumnName ] Table2 [ , ...]  

    union [ kind= inner | outer ] [ withsource = ColumnName ] Table1, Table2 [ , ...]  

**Arguments**

* *Table1*, *Table2* ...
 *  Nom d’une table, tel que `events` ; ou
 *  Expression de requête, telle que `(events | where id==42)`
 *  Ensemble de tables spécifié par un caractère générique. Par exemple, `E*` assure l’union de toutes les tables dans la base de données dont le nom commence par `E`.
* `kind` : 
 * `inner` : le résultat comporte le sous-ensemble de colonnes qui sont communes à toutes les tables d’entrée.
 * `outer` : le résultat comporte toutes les colonnes qui apparaissent dans les entrées. Les cellules qui n’ont pas été définies par une ligne d’entrée sont définies sur `null`.
* `withsource=`*ColumnName :* si spécifiée, la sortie comprend une colonne nommée *ColumnName* dont la valeur indique la table source correspondant à chaque ligne.

**Retourne**

Une table contenant autant de lignes que dans l’ensemble des tables d’entrée.

**Exemple**

```AIQL

let ttrr = requests | where timestamp > ago(1h);
let ttee = exceptions | where timestamp > ago(1h);
union tt* | count
```
Union de toutes les tables dont le nom commence par « tt ».


**Exemple**

```AIQL

union withsource=SourceTable kind=outer Query, Command
| where Timestamp > ago(1d)
| summarize dcount(UserId)
```
Le nombre d’utilisateurs distincts ayant produit un événement `exceptions` ou `traces` au cours de la journée précédente. Dans le résultat, la colonne ’SourceTable’ indique « Requête » ou « Commande ».

```AIQL
exceptions
| where Timestamp > ago(1d)
| union withsource=SourceTable kind=outer 
   (Command | where Timestamp > ago(1d))
| summarize dcount(UserId)
```

Cette version plus efficace génère le même résultat. Il filtre chaque table avant la création de l’union.

### opérateur where

     T | where fruit=="apple"

Filtre une table d’après le sous-ensemble de lignes correspondant à un prédicat.

**Alias** `filter`

**Syntaxe**

    T | where Predicate

**Arguments**

* *T* : entrée tabulaire dont les enregistrements doivent être filtrés.
* *Predicate :* `boolean`[expression](#boolean) sur les colonnes de *T*. Elle est évaluée pour chaque ligne dans *T*.

**Retourne**

Les lignes de *T* dont le *prédicat* est `true`.

**Conseils**

Pour obtenir des performances optimales :

* **Utilisez des comparaisons simples** entre les noms de colonnes et les constantes. (« Constante » s’entend dans le sens de constante au fil de la table, de telle sorte que `now()` et `ago()` soient OK, tout comme les valeurs scalaires affectées à l’aide d’une [instruction `let`](#let-clause).)

    Par exemple, préférez `where Timestamp >= ago(1d)` à `where floor(Timestamp, 1d) == ago(1d)`.

* **Simplest terms first** : si vous avez plusieurs clauses unies avec `and`, insérez d’abord les clauses n’impliquant qu’une seule colonne. C’est pourquoi `Timestamp > ago(1d) and OpId == EventId` est plus adapté.


**Exemple**

```AIQL
Traces
| where Timestamp > ago(1h)
    and Source == "Kuskus"
    and ActivityId == SubActivityIt 
```

Enregistrements datant de moins de 1 heure et provenant de la source nommée « Kuskus » et ayant deux colonnes de la même valeur.

Notez que nous plaçons la comparaison entre deux colonnes à la fin, car elle ne peut pas utiliser l’index et force une analyse.



## Aggrégrations et synthèse

### any 

    any(Expression)

Sélectionne une ligne du groupe de façon aléatoire et retourne la valeur de l’expression spécifiée.

Cette fonction est utile, par exemple, quand une colonne a un grand nombre de valeurs similaires (par exemple, une colonne « texte d’erreur ») et que vous souhaitez échantillonner cette colonne une fois en fonction d’une valeur unique de la clé de groupe composée.

**Exemple**

```

traces 
| where timestamp > now(-15min)  
| summarize count(), any(message) by operation_Name 
| top 10 by count_level desc 
```

<a name="argmin"></a> <a name="argmax"></a>
### argmin, argmax

    argmin(ExprToMinimize, * | ExprToReturn  [ , ... ] )
    argmax(ExprToMaximize, * | ExprToReturn  [ , ... ] ) 

Recherche dans le groupe la ligne correspondant à la valeur maximale de *ExprToMaximize* ou à la valeur minimale de ExprToMinimize, et retourne la valeur de *ExprToReturn* (ou `*` pour retourner la ligne entière).

**Conseil** : les colonnes analysées sont automatiquement renommées. Pour vérifier que vous utilisez les noms corrects, examinez les résultats à l’aide de `take 5` avant de les transmettre à un autre opérateur.

**Exemples**

Pour chaque nom de requête, afficher à quel moment la requête la plus longue s’est produite :

    requests | summarize argmax(duration, timestamp) by name

Afficher tous les détails de la requête la plus longue, pas seulement l’horodatage :

    requests | summarize argmax(duration, *) by name


Trouver la plus petite valeur de chaque métrique, ainsi que son horodatage et d’autres données :

    metrics 
    | summarize minValue=argmin(value, *) 
      by name


![](./media/app-insights-analytics-aggregations/argmin.png)
 


### avg

    avg(Expression)

Calcule la moyenne de *Expression* dans le groupe.

### buildschema

    buildschema(DynamicExpression)

Retourne le schéma minimal qui admet toutes les valeurs de *DynamicExpression*.

Le type de la colonne de paramètre doit être `dynamic` (tableau ou conteneur de propriétés).

**Exemple**

    exceptions | summarize buildschema(details)

Résultat :

    { "`indexer`":
     {"id":"string",
       "parsedStack":
       { "`indexer`": 
         {  "level":"int",
            "assembly":"string",
            "fileName":"string",
            "method":"string",
            "line":"int"
         }},
      "outerId":"string",
      "message":"string",
      "type":"string",
      "rawStack":"string"
    }}

`indexer` indique où vous devez utiliser un index numérique. Pour ce schéma, voici certains chemins valides (en supposant que ces exemples d’index appartiennent à la plage) :

    details[0].parsedStack[2].level
    details[0].message
    arraylength(details)
    arraylength(details[0].parsedStack)

**Exemple**

Supposons que la colonne d’entrée a trois valeurs dynamiques :

| |
|---|
|`{"x":1, "y":3.5}`
|`{"x":"somevalue", "z":[1, 2, 3]}`
|`{"y":{"w":"zzz"}, "t":["aa", "bb"], "z":["foo"]}`


Le schéma résultant serait :

    { 
      "x":["int", "string"], 
      "y":["double", {"w": "string"}], 
      "z":{"`indexer`": ["int", "string"]}, 
      "t":{"`indexer`": "string"} 
    }

Le schéma fournit les informations suivantes :

* L’objet racine est un conteneur avec quatre propriétés nommées x, y, z et t.
* La propriété appelée « x » peut être de type « int » ou « string ».
* La propriété appelée « y » peut être de type « double », ou un autre conteneur avec une propriété appelée « w » de type « string ».
* Le mot clé ``indexer`` indique que « z » et « t » sont des tableaux.
* Chaque élément du tableau « z » est un entier ou une chaîne.
* « t » est un tableau de chaînes.
* Chaque propriété est implicitement facultative et tout tableau peut être vide.

##### Modèle de schéma

La syntaxe du schéma retourné est la suivante :

    Container ::= '{' Named-type* '}';
    Named-type ::= (name | '"`indexer`"') ':' Type;
	Type ::= Primitive-type | Union-type | Container;
    Union-type ::= '[' Type* ']';
    Primitive-type ::= "int" | "string" | ...;

Le schéma s’apparente à un sous-ensemble d’annotations de type TypeScript, encodées sous la forme d’une valeur dynamique. En Typescript, l’exemple de schéma serait le suivant :

    var someobject: 
    { 
      x?: (number | string), 
      y?: (number | { w?: string}), 
      z?: { [n:number] : (int | string)},
      t?: { [n:number]: string } 
    }


### count

    count([ Predicate ])

Retourne le nombre de lignes pour lesquelles *Predicate* est `true`. Si *Predicate* n’est pas spécifié, retourne le nombre total d’enregistrements dans le groupe.

**Conseil pour optimiser les performances** : utilisez `summarize count(filter)` plutôt que `where filter | summarize count()`.

> [AZURE.NOTE] Évitez d'utiliser count() pour rechercher le nombre de demandes, d’exceptions ou autres événements qui se sont produits. Quand l’[échantillonnage](app-insights-sampling.md) est en cours, le nombre de points de données est inférieur au nombre d'événements réels. Utilisez plutôt `summarize sum(itemCount)...`. La propriété itemCount reflète le nombre d'événements originaux qui sont représentés par chaque point de données conservé.
   

### dcount

    dcount( Expression [ ,  Accuracy ])

Retourne une estimation du nombre de valeurs distinctes de *Expr* dans le groupe. (Pour afficher les valeurs distinctes, utilisez [`makeset`](#makeset).)

Si *Accuracy* est spécifié, détermine le compromis entre vitesse et précision.

 * `0` : calcul le moins précis, mais le plus rapide.
 * `1` (valeur par défaut) : meilleur compromis entre précision et vitesse de calcul (environ 0,8 % d’erreur).
 * `2` : calcul le plus précis, mais le plus lent (environ 0,4 % d’erreur).

**Exemple**

    pageViews 
    | summarize countries=dcount(client_City) 
      by client_CountryOrRegion

![](./media/app-insights-analytics-aggregations/dcount.png)

### makelist

    makelist(Expr [ ,  MaxListSize ] )

Retourne un tableau (JSON) `dynamic` de toutes les valeurs de *Expr* dans le groupe.

* *MaxListSize* est une limite de nombre entier facultative sur le nombre maximal d’éléments retournés (la valeur par défaut est *128*).

### makeset

    makeset(Expression [ , MaxSetSize ] )

Retourne un tableau (JSON) `dynamic` du jeu de valeurs distinctes que *Expr* prend dans le groupe. (Conseil : pour comptabiliser uniquement les valeurs distinctes, utilisez [`dcount`](#dcount).)
  
*  *MaxSetSize* est une limite de nombre entier facultative sur le nombre maximal d’éléments retournés (la valeur par défaut est *128*).

**Exemple**

    pageViews 
    | summarize countries=makeset(client_City) 
      by client_CountryOrRegion

![](./media/app-insights-analytics-aggregations/makeset.png)

Consultez aussi l’[opérateur `mvexpand`](#mvexpand-operator) pour la fonction inverse.


### max, min

    max(Expr)

Calcule la valeur maximale de *Expr*.
    
    min(Expr)

Calcule la valeur minimale de *Expr*.

**Conseil** : ces fonctions ne donnent que des valeurs maximales ou minimales, par exemple le prix le plus élevé ou le plus bas. Pour que la ligne contienne d’autres colonnes, par exemple le nom du fournisseur proposant le prix le plus bas, utilisez [argmin ou argmax](#argmin-argmax).


<a name="percentile"></a> <a name="percentiles"></a>
### percentile, percentiles

    percentile(Expression, Percentile)

Retourne une estimation de *Expression* du centile spécifié dans le groupe. La précision dépend de la densité de population dans la région du centile.
    
    percentiles(Expression, Percentile1 [ , Percentile2 ] )

Similaire à `percentile()`, mais calcule un nombre de valeurs de centile (opération plus rapide que le calcul de chaque centile individuellement).

**Exemples**


Pour chaque nom de requête, valeur de `duration` supérieure à 95 % du jeu d’échantillon et inférieure à 5 % du jeu d’échantillon :

    request 
    | summarize percentile(duration, 95)
      by name

Omettez « by... » pour calculer la table entière.

Calculer simultanément plusieurs centiles pour différents noms de requête :

    
    requests 
    | summarize 
        percentiles(duration, 5, 20, 50, 80, 95) 
      by name

![](./media/app-insights-analytics-aggregations/percentiles.png)

Les résultats montrent que pour la requête /Events/Index, 5 % des requêtes reçoivent une réponse en moins de 2,44 s, la moitié en 3,52 s et 5 % en plus de 6,85 s.


Calculer plusieurs statistiques :

    requests 
    | summarize 
        count(), 
        avg(Duration),
        percentiles(Duration, 5, 50, 95)
      by name

##### Erreur d’estimation dans les centiles

L’agrégation de centiles fournit une valeur approximative au moyen de [T-Digest](https://github.com/tdunning/t-digest/blob/master/docs/t-digest-paper/histo.pdf).

Voici quelques points importants :

* Les limites de l’erreur d’estimation dépendent de la valeur du centile demandé. Les extrémités de l’échelle [0..100] offrent la meilleure précision, les centiles 0 et 100 étant les valeurs minimale et maximale exactes de la distribution. La précision diminue progressivement vers le milieu de l’échelle. Elle est la plus dégradée à la valeur médiane et est limitée à 1 %. 
* Les limites d’erreur sont observées sur le classement, et non sur la valeur. Supposons que percentile(X, 50) retourne la valeur Xm. L’estimation garantit qu’au moins 49 % et au plus 51 % des valeurs de X sont inférieures à Xm. Il n’existe aucune limite théorique quant à la différence entre Xm et la valeur médiane réelle de X.

### stdev

     stdev(Expr)

Retourne l’écart type de *Expr* sur le groupe.

### variance

    variance(Expr)

Retourne la variance de *Expr* sur le groupe.

### sum

    sum(Expr)

Retourne la somme de *Expr* sur le groupe.


## Valeurs scalaires

[casts](#casts) | [comparisons](#scalar-comparisons) <br/> [gettype](#gettype) | [hash](#hash) | [iff](#iff)| [isnull](#isnull) | [isnotnull](#isnotnull) | [notnull](#notnull)

Les types pris en charge sont :

| Type | Nom(s) supplémentaire(s) | Type .NET équivalent |
| --------- | -------------------- | -------------------- |
| `bool` | `boolean` | `System.Boolean` |
| `datetime`| `date` | `System.DateTime` |
| `dynamic` | | `System.Object` |
| `guid` | `uuid`, `uniqueid` | `System.Guid` |
| `int` | | `System.Int32` |
| `long` | | `System.Int64` |
| `double` | `real` | `System.Double` |
| `string` | | `System.String` |
| `timespan`| `time` | `System.TimeSpan` |

### Casts

Vous pouvez effectuer une conversion d’un type en un autre. En général, si la conversion a un sens, cela fonctionnera :

    todouble(10), todouble("10.6")
    toint(10.6) == 11
    floor(10.6) == 10
	toint("200")
    todatetime("2016-04-28 13:02")
    totimespan("1.5d"), totimespan("1.12:00:00")
    toguid("00000000-0000-0000-0000-000000000000")
    tostring(42.5)
    todynamic("{a:10, b:20}")

### Comparaisons scalaires

||
---|---
`<` |Inférieur à
`<=`|Inférieur ou égal à
`>` |Supérieur à
`>=`|Supérieur ou égal à
`<>`|Non égal à
`!=`|Non égal à 
`in`| L’opérande de droite est un tableau (dynamique) et l’opérande de gauche est égal à un de ses éléments.
`!in`| L’opérande de droite est un tableau (dynamique) et l’opérande de gauche n’est pas égal à un de ses éléments.




### gettype

**Retourne**

Une chaîne représentant le type de stockage sous-jacent de son argument unique. C’est particulièrement utile avec des valeurs de type `dynamic` : dans ce cas `gettype()` indique comment une valeur est encodée.

**Exemples**

|||
---|---
`gettype("a")` |`"string" `
`gettype(111)` |`"long" `
`gettype(1==1)` |`"int8" (*) `
`gettype(now())` |`"datetime" `
`gettype(1s)` |`"timespan" `
`gettype(parsejson('1'))` |`"int" `
`gettype(parsejson(' "abc" '))` |`"string" `
`gettype(parsejson(' {"abc":1} '))` |`"dictionary"` 
`gettype(parsejson(' [1, 2, 3] '))` |`"array"` 
`gettype(123.45)` |`"real" `
`gettype(guid(12e8b78d-55b4-46ae-b068-26d7a0080254))` |`"guid"` 
`gettype(parsejson(''))` |`"null"`



### Hachage

**Syntaxe**

    hash(source [, mod])

**Arguments**

* *source* : source scalaire à partir de laquelle le hachage est calculé.
* *mod* : valeur modulo à appliquer au résultat de hachage.

**Retourne**

La valeur xxhash (long) de la valeur scalaire donnée, modulo, la valeur mod donnée (si spécifiée).

**Exemples**

```
hash("World")                   // 1846988464401551951
hash("World", 100)              // 51 (1846988464401551951 % 100)
hash(datetime("2015-01-01"))    // 1380966698541616202
```
### iff

La fonction `iff()` évalue le premier argument (le prédicat) et retourne la valeur du deuxième ou troisième argument selon que le prédicat est `true` ou `false`. Les deuxième et troisième arguments doivent être de même type.

**Syntaxe**

    iff(predicate, ifTrue, ifFalse)


**Arguments**

* *predicate :* expression qui correspond à une valeur `boolean`.
* *ifTrue :* expression qui est évaluée et sa valeur retournée par la fonction si *predicate* est `true`.
* *ifFalse :* expression qui est évaluée et sa valeur retournée par la fonction si *predicate* est `false`.

**Retourne**

Cette fonction retourne la valeur de *ifTrue* si *predicate* est `true`, ou la valeur de *ifFalse* dans le cas contraire.

**Exemple**

```
iff(floor(timestamp, 1d)==floor(now(), 1d), "today", "anotherday")
```

<a name="isnull"/></a> <a name="isnotnull"/></a> <a name="notnull"/></a>
### isnull, isnotnull, notnull

    isnull(parsejson("")) == true

Accepte un argument unique et indique si la valeur est null.

**Syntaxe**


    isnull([value])


    isnotnull([value])


    notnull([value])  // alias for isnotnull

**Retourne**

True ou false selon si la valeur est null ou not null.


|x|isnull(x)
|---|---
| "" | false
|"x" | false
|parsejson("")|true
|parsejson("")|false
|parsejson("{}")|false

**Exemple**

    T | where isnotnull(PossiblyNull) | count

Notez qu’il existe d’autres façons d’obtenir cet effet :

    T | summarize count(PossiblyNull)




## Boolean 

### Littéraux booléens

	true == 1
    false == 0
    gettype(true) == "int8"
    typeof(bool) == typeof(int8)

### Opérateurs booléens

	and 
    or 

    

## Nombres

[bin](#bin) | [floor](#floor) | [rand](#rand) | [range](#range) | [sqrt](#sqrt) | [todouble](#todouble) | [toint](#toint) | [tolong](#tolong)

### Littéraux numériques

|||
|---|---
|`42`|`long`
|`42.0`|`real`

### Opérateurs arithmétiques

|| |
|---|-------------|
| + | Ajouter |
| - | Soustraire || * | Multiplier || / | Diviser || % | Modulo | || |`<` |Inférieur à |`<=`|Inférieur ou égal à |`>` |Supérieur à |`>=`|Supérieur ou égal à |`<>`|Non égal à |`!=`|Non égal à




### bin

Arrondit les valeurs à l’entier inférieur multiple d’une taille bin donnée. Très utilisé dans la requête [`summarize by`](#summarize-operator). Si vous avez un ensemble de valeurs dispersées, elles seront regroupées dans un plus petit ensemble de valeurs spécifiques.

Alias `floor`.

**Syntaxe**

     bin(value, roundTo)

**Arguments**

* *value:* nombre, date ou intervalle de temps. 
* *roundTo:* la « taille bin ». Nombre, date ou intervalle de temps qui divise *value*. 

**Retourne**

Le multiple le plus proche de *roundTo* en dessous de *value*.
 
    (toint((value/roundTo)-0.5)) * roundTo

**Exemples**

Expression | Résultat
---|---
`bin(4.5, 1)` | `4.0`
`bin(time(16d), 7d)` | `14d`
`bin(datetime(1953-04-15 22:25:07), 1d)`| `datetime(1953-04-15)`


L’expression suivante calcule un histogramme de durées, avec une taille de compartiment de 1 seconde :

```AIQL

    T | summarize Hits=count() by bin(Duration, 1s)
```

### floor

Alias pour [`bin()`](#bin).


### rand

Générateur de nombres aléatoires.

* `rand()` : nombre réel compris entre 0,0 et 1,0
* `rand(n)` : entier compris entre 0 et n-1




### sqrt

Fonction racine carrée.

**Syntaxe**

    sqrt(x)

**Arguments**

* *x :* nombre réel >= 0.

**Retourne**

* Un nombre positif tel que `sqrt(x) * sqrt(x) == x`
* `null` si l’argument est négatif ou s’il ne peut pas être converti en une valeur `real`. 




### toint

    toint(100)        // cast from long
    toint(20.7) == 21 // nearest int from double
    toint(20.4) == 20 // nearest int from double
    toint("  123  ")  // parse string
    toint(a[0])       // cast from dynamic
    toint(b.c)        // cast from dynamic

### tolong

    tolong(20.7) == 21 // conversion from double
    tolong(20.4) == 20 // conversion from double
    tolong("  123  ")  // parse string
    tolong(a[0])       // cast from dynamic
    tolong(b.c)        // cast from dynamic


### todouble

    todouble(20) == 20.0 // conversion from long or int
    todouble(" 12.34 ")  // parse string
    todouble(a[0])       // cast from dynamic
    todouble(b.c)        // cast from dynamic



## Date et heure


[ago](#ago) | [dayofweek](#dayofweek) | [getmonth](#getmonth)| [getyear](#getyear) | [now](#now) | [startofmonth](#startofmonth) | [startofyear](#startofyear) | [todatetime](#todatetime) | [totimespan](#totimespan)

### Littéraux de date et d’heure

|||
---|---
**datetime**|
`datetime("2015-12-31 23:59:59.9")`<br/>`datetime("2015-12-31")`|Les heures sont toujours exprimées en UTC. L’omission de la date entraîne la sélection de la date du jour.
`now()`|L’heure actuelle.
`now(`-*timespan*`)`|`now()-`*timespan*
`ago(`*timespan*`)`|`now()-`*timespan*
**timespan**|
`2d`|2 jours
`1.5h`|1,5 heure 
`30m`|30 minutes
`10s`|10 secondes
`0.1s`|0,1 seconde
`100ms`| 100 millisecondes
`10microsecond`|
`1tick`|100ns
`time("15 seconds")`|
`time("2")`| 2 jours
`time("0.12:34:56.7")`|`0d+12h+34m+56.7s`

### Expressions de date et d’heure

Expression |Résultat
---|---
`datetime("2015-01-02") - datetime("2015-01-01")`| `1d`
`datetime("2015-01-01") + 1d`| `datetime("2015-01-02")`
`datetime("2015-01-01") - 1d`| `datetime("2014-12-31")`
`2h * 24` | `2d`
`2d` / `2h` | `24`
`datetime("2015-04-15T22:33") % 1d` | `timespan("22:33")`
`bin(datetime("2015-04-15T22:33"), 1d)` | `datetime("2015-04-15T00:00")`
||
`<` |Inférieur à
`<=`|Inférieur ou égal à
`>` |Supérieur à
`>=`|Supérieur ou égal à
`<>`|Non égal à
`!=`|Non égal à 




### ago

Soustrait l’intervalle de temps donné de l’heure UTC actuelle. Comme `now()`, cette fonction peut être utilisée plusieurs fois dans une instruction, et l’heure UTC référencée est la même pour toutes les instanciations.

**Syntaxe**

    ago(a_timespan)

**Arguments**

* *a\_timespan* : intervalle à soustraire de l’heure UTC actuelle (`now()`).

**Retourne**

    now() - a_timespan

**Exemple**

Toutes les lignes de l’horodatage de la dernière heure :

```AIQL

    T | where timestamp > ago(1h)
```



### dayofweek

    dayofweek(datetime("2015-12-14")) == 1d  // Monday

Nombre entier de jours écoulés depuis le précédent dimanche, en tant que `timespan`.

**Syntaxe**

    dayofweek(a_date)

**Arguments**

* `a_date` : un `datetime`.

**Retourne**

Le `timespan` depuis minuit au début du précédent dimanche, arrondi au nombre entier inférieur de jours.

**Exemples**

```AIQL
dayofweek(1947-11-29 10:00:05)  // time(6.00:00:00), indicating Saturday
dayofweek(1970-05-11)           // time(1.00:00:00), indicating Monday
```

### getmonth

Obtient le numéro du mois (1 à 12) à partir d’une valeur datetime.

**Exemple**

    ... | extend month = getmonth(datetime(2015-10-12))

    --> month == 10

### getyear

Obtient l’année à partir d’une valeur datetime.

**Exemple**

    ... | extend year = getyear(datetime(2015-10-12))

    --> year == 2015

### now

    now()
    now(-2d)

L’heure UTC actuelle, avec un décalage éventuel selon un intervalle de temps donné. Cette fonction peut être utilisée plusieurs fois dans une instruction, et l’heure référencée est la même pour toutes les instances.

**Syntaxe**

    now([offset])

**Arguments**

* *offset :* un `timespan` ajouté à l’heure UTC actuelle. Valeur par défaut : 0.

**Retourne**

L’heure UTC actuelle en tant que `datetime`.

    now() + offset

**Exemple**

Détermine l’intervalle depuis l’événement identifié par le prédicat :

```AIQL
T | where ... | extend Elapsed=now() - timestamp
```

### startofmonth

    startofmonth(date)

Début du mois qui contient la date.

### startofyear

    startofyear(date)

Début de l’année qui contient la date.


### todatetime

Alias `datetime()`.

     todatetime("2016-03-28")
     todatetime("03/28/2016")
     todatetime("2016-03-28 14:34")
     todatetime("03/28/2016 2:34pm")
     todatetime("2016-03-28T14:34.5Z")
     todatetime(a[0])  // cast a dynamic type
     todatetime(b.c)   // cast a dynamic type

### totimespan

Alias `timespan()`.

    totimespan("21d")
    totimespan("21h")
    totimespan(request.duration)


## Chaîne

[countof](#countof) | [extract](#extract) | [extractjson](#extractjson) | [isempty](#isempty) | [isnotempty](#isnotempty) | [notempty](#notempty) | [replace](#replace) | [split](#split) | [strcat](#strcat) | [strlen](#strlen) | [substring](#substring) | [tolower](#tolower) | [tostring](#tostring) | [toupper](#toupper)


### Littéraux de chaîne

Les règles sont les mêmes que dans JavaScript.

Les chaînes peuvent être placées entre guillemets simples ou doubles.

La barre oblique inverse (``) est utilisée pour échapper les caractères tels que `\t` (tabulation), `\n` (renvoi à la ligne) et les instances de guillemets englobants.

* `'this is a "string" literal in single \' quotes'`
* `"this is a 'string' literal in double " quotes"`
* `@"C:\backslash\not\escaped\with @ prefix"`

### Littéraux de chaîne masqués

Les littéraux de chaîne masqués sont des chaînes qu’Analytics masque lors de la sortie de la chaîne (par exemple, lors d’un suivi). Le processus de masquage remplace tous les caractères masqués par un caractère de démarrage (`*`).

Pour créer un littéral de chaîne masqué, ajoutez `h` ou « H ». Par exemple :

```
h'hello'
h@'world' 
h"hello"
```

### Comparaisons de chaînes

Opérateur|Description|Respecte la casse|Exemple vrai
---|---|---|---
`==`|Égal à |Oui| `"aBc" == "aBc"`
`<>`|Non égal à|Oui| `"abc" <> "ABC"`
`=~`|Égal à |Non| `"abc" =~ "ABC"`
`!~`|Non égal à |Non| `"aBc" !~ "xyz"`
`has`|Le terme de droite est un terme entier dans le terme de gauche|Non| `"North America" has "america"`
`!has`|Le terme de droite n’est pas un terme entier dans le terme de gauche|Non|`"North America" !has "amer"` 
`contains` | Le terme de droite est une sous-séquence du terme de gauche|Non| `"FabriKam" contains "BRik"`
`!contains`| Le terme de droite n’est pas une sous-séquence du terme de gauche|Non| `"Fabrikam" !contains "xyz"`
`containscs` | Le terme de droite est une sous-séquence du terme de gauche|Oui| `"FabriKam" contains "Kam"`
`!containscs`| Le terme de droite n’est pas une sous-séquence du terme de gauche|Oui| `"Fabrikam" !contains "Kam"`
`startswith`|Le terme de droite est une sous-séquence initiale du terme de gauche|Non|`"Fabrikam" startswith "fab"`
`matches regex`|Le terme de gauche contient une correspondance du terme de droite|Oui| `"Fabrikam" matches regex "b.*k"`


Utilisez `has` ou `in` si vous testez la présence d’un terme lexical entier, c’est-à-dire un symbole ou un mot alphanumérique délimité par des caractères non alphanumériques ou le début ou la fin de champ. `has` effectue la recherche plus rapidement que `contains` ou `startswith`. La première de ces requêtes s’exécute plus rapidement :

    EventLog | where continent has "North" | count;
	EventLog | where continent contains "nor" | count





### countof

    countof("The cat sat on the mat", "at") == 3
    countof("The cat sat on the mat", @"\b.at\b", "regex") == 3

Compte les occurrences d’une sous-chaîne dans une chaîne. Les correspondances de chaînes simples peuvent se chevaucher, mais pas les correspondances d’expression régulière.

**Syntaxe**

    countof(text, search [, kind])

**Arguments**

* *text :* chaîne.
* *search :* chaîne simple ou [expression régulière](app-analytics-reference.md#regular-expressions) à faire correspondre à *text*.
* *kind :* `"normal"|"regex"`. Valeur par défaut : `normal`. 

**Retourne**

Le nombre de fois où la chaîne de recherche peut être mise en correspondance dans le conteneur. Les correspondances de chaînes simples peuvent se chevaucher, mais pas les correspondances d’expression régulière.

**Exemples**

|||
|---|---
|`countof("aaa", "a")`| 3 
|`countof("aaaa", "aa")`| 3 (pas 2 !)
|`countof("ababa", "ab", "normal")`| 2
|`countof("ababa", "aba")`| 2
|`countof("ababa", "aba", "regex")`| 1
|`countof("abcabc", "a.c", "regex")`| 2
    



### extract

    extract("x=([0-9.]+)", 1, "hello x=45.6|wo") == "45.6"

Obtient une correspondance pour une [expression régulière](#regular-expressions) à partir d’une chaîne de texte. Éventuellement, il convertit ensuite la sous-chaîne extraite dans le type indiqué.

**Syntaxe**

    extract(regex, captureGroup, text [, typeLiteral])

**Arguments**

* *regex :* [expression régulière](#regular-expressions).
* *captureGroup :* constante `int` positive qui indique le groupe de capture à extraire. Les valeurs sont 0 pour la correspondance entière, 1 pour la valeur mise en correspondance par la première '('parenthèse')' dans l’expression régulière, 2 ou plus pour les parenthèses suivantes.
* *text :* `string` à rechercher.
* *typeLiteral :* littéral de type facultatif (par exemple, `typeof(long)`). Si elle est fournie, la sous-chaîne extraite est convertie dans ce type. 

**Retourne**

Si *regex* trouve une correspondance dans *text* : la sous-chaîne mise en correspondance avec le groupe de capture indiqué *captureGroup*, éventuellement converti en *typeLiteral*.

Si aucune correspondance n’est trouvée, ou si la conversion de type échoue : `null`.

**Exemples**

Une définition de `Duration` est recherchée dans l’exemple de chaîne `Trace`. La correspondance est convertie en `real`, qui est ensuite multiplié par une constante de temps (`1s`) pour que `Duration` soit de type `timespan`. Dans cet exemple, elle est égale à 123,45 secondes :

```AIQL
...
| extend Trace="A=1, B=2, Duration=123.45, ..."
| extend Duration = extract("Duration=([0-9.]+)", 1, Trace, typeof(real)) * time(1s) 
```

Cet exemple est équivalent à `substring(Text, 2, 4)` :

```AIQL
extract("^.{2,2}(.{4,4})", 1, Text)
```

<a name="notempty"></a> <a name="isnotempty"></a> <a name="isempty"></a>
### isempty, isnotempty, notempty

    isempty("") == true

True si l’argument est une chaîne vide ou s’il a la valeur null. Voir aussi [isnull](#isnull).


**Syntaxe**

    isempty([value])


    isnotempty([value])


    notempty([value]) // alias of isnotempty

**Retourne**

Indique si l’argument est une chaîne vide ou s’il a la valeur isnull.

|x|isempty(x)
|---|---
| "" | true
|"x" | false
|parsejson("")|true
|parsejson("")|false
|parsejson("{}")|false


**Exemple**


    T | where isempty(fieldName) | count




### replace

Remplace toutes les correspondances d’expression régulière par une autre chaîne.

**Syntaxe**

    replace(regex, rewrite, text)

**Arguments**

* *regex :* [expression régulière](https://github.com/google/re2/wiki/Syntax) à rechercher dans *text*. Elle peut contenir des groupes de capture entre '('parenthèses')'. 
* *rewrite :* expression régulière de remplacement pour toute correspondance trouvée par *matchingRegex*. Utilisez `\0` pour faire référence à la correspondance complète, `\1` pour le premier groupe de capture, `\2` et ainsi de suite pour les groupes de capture suivants.
* *text :* chaîne.

**Retourne**

*text* après le remplacement de toutes les correspondances de *regex* par les évaluations de *rewrite*. Les correspondances ne se chevauchent pas.

**Exemple**

L’instruction suivante :

```AIQL
range x from 1 to 5 step 1
| extend str=strcat('Number is ', tostring(x))
| extend replaced=replace(@'is (\d+)', @'was: \1', str)
```

Donne les résultats suivants :

| x | str | replaced|
|---|---|---|
| 1 | Le nombre est 1.000000 | Le nombre était : 1.000000|
| 2 | Le nombre est 2.000000 | Le nombre était : 2.000000|
| 3 | Le nombre est 3.000000 | Le nombre était : 3.000000|
| 4 | Le nombre est 4.000000 | Le nombre était : 4.000000|
| 5 | Le nombre est 5.000000 | Le nombre était : 5.000000|
 



### split

    split("aaa_bbb_ccc", "_") == ["aaa","bbb","ccc"]

Fractionne une chaîne donnée en fonction d’un délimiteur donné et retourne un tableau de chaînes avec les sous-chaînes contenues. Éventuellement, une sous-chaîne spécifique peut être retournée si elle existe.

**Syntaxe**

    split(source, delimiter [, requestedIndex])

**Arguments**

* *source* : chaîne source qui sera fractionnée en fonction du délimiteur donné.
* *delimiter* : délimiteur qui sera utilisé pour fractionner la chaîne source.
* *requestedIndex* : index de base zéro facultatif `int`. S’il est fourni, le tableau de chaînes retourné contient la sous-chaîne demandée si elle existe. 

**Retourne**

Un tableau de chaînes qui contient les sous-chaînes de la chaîne source donnée séparées par le délimiteur donné.

**Exemples**

```
split("aa_bb", "_")           // ["aa","bb"]
split("aaa_bbb_ccc", "_", 1)  // ["bbb"]
split("", "_")                // [""]
split("a__b")                 // ["a","","b"]
split("aabbcc", "bb")         // ["aa","cc"]
```




### strcat

    strcat("hello", " ", "world")

Concatène entre 1 et 16 arguments, qui doivent être des chaînes.

### strlen

    strlen("hello") == 5

Longueur d’une chaîne.

### substring

    substring("abcdefg", 1, 2) == "bc"

Extrait une sous-chaîne d’une chaîne source donnée à partir d’un index donné. Éventuellement, la longueur de la sous-chaîne demandée peut être spécifiée.

**Syntaxe**

    substring(source, startingIndex [, length])

**Arguments**

* *source :* chaîne source dont la sous-chaîne est extraite.
* *startingIndex :* position du caractère de départ de base zéro de la sous-chaîne demandée.
* *length :* paramètre facultatif qui peut être utilisé pour spécifier le nombre de caractères demandé dans la sous-chaîne. 

**Retourne**

Une sous-chaîne de la chaîne donnée. La sous-chaîne commence à la position de caractère startingIndex (de base zéro) et continue jusqu’à la fin de la chaîne ou des caractères de longueur si elle est spécifiée.

**Exemples**

```
substring("123456", 1)        // 23456
substring("123456", 2, 2)     // 34
substring("ABCD", 0, 2)       // AB
```

### tolower

    tolower("HELLO") == "hello"

Convertit une chaîne en minuscules.

### toupper

    toupper("hello") == "HELLO"

Convertit une chaîne en majuscules.



## GUID

    guid(00000000-1111-2222-3333-055567f333de)


## Tableaux, objets et dynamiques

[literals](#dynamic-literals) | [casting](#casting-dynamic-objects) | [operators](#operators) | [clauses let](#dynamic-objects-in-let-clauses) <br/> [arraylength](#arraylength) | [extractjson](#extractjson) | [parsejson](#parsejson) | [range](#range) | [treepath](#treepath) | [todynamic](#todynamic)


Voici le résultat d’une requête sur une exception d’Application Insights. La valeur de `details` est un tableau.

![](./media/app-analytics-scalars/310.png)

**Indexing :** tableaux d’index et objets comme dans JavaScript :

    exceptions | take 1
    | extend 
        line = details[0].parsedStack[0].line,
        stackdepth = arraylength(details[0].parsedStack)

* Mais utilisez `arraylength` et d’autres fonctions Analytics (pas « .length »)

**Conversion** Dans certains cas, il est nécessaire de convertir un élément que vous extrayez à partir d’un objet, car son type peut varier. Par exemple, `summarize...to` a besoin d’un type spécifique :

    exceptions 
    | summarize count() 
      by toint(details[0].parsedStack[0].line)

    exceptions 
    | summarize count() 
      by tostring(details[0].parsedStack[0].assembly)

**Littéraux** Pour créer un tableau explicite ou un objet conteneur de propriétés, écrivez-le en tant que chaîne JSON et effectuez la conversion :

    todynamic('[{"x":"1", "y":"32"}, {"x":"6", "y":"44"}]')


**mvexpand :** pour décomposer les propriétés d’un objet dans des lignes distinctes, utilisez mvexpand :

    exceptions | take 1 
    | mvexpand details[0].parsedStack[0]


![](./media/app-analytics-scalars/410.png)


**treepath :** pour rechercher tous les chemins dans un objet complexe :

    exceptions | take 1 | project timestamp, details 
    | extend path = treepath(details) 
    | mvexpand path


![](./media/app-analytics-scalars/420.png)

**buildschema :** pour trouver le schéma minimal qui admet toutes les valeurs de l’expression dans la table :

    exceptions | summarize buildschema(details)

Résultat :

    { "`indexer`":
     {"id":"string",
       "parsedStack":
       { "`indexer`": 
         {  "level":"int",
            "assembly":"string",
            "fileName":"string",
            "method":"string",
            "line":"int"
         }},
      "outerId":"string",
      "message":"string",
      "type":"string",
      "rawStack":"string"
    }}

`indexer` indique où vous devez utiliser un index numérique. Pour ce schéma, voici certains chemins valides (en supposant que ces exemples d’index appartiennent à la plage) :

    details[0].parsedStack[2].level
    details[0].message
    arraylength(details)
    arraylength(details[0].parsedStack)



### Littéraux de tableau et d’objet

Pour créer un littéral dynamique, utilisez `parsejson` (alias `todynamic`) avec un argument de chaîne JSON :

* `parsejson('[43, 21, 65]')` : tableau de nombres
* `parsejson('{"name":"Alan", "age":21, "address":{"street":432,"postcode":"JLK32P"}}')` 
* `parsejson('21')` : valeur unique de type dynamique contenant un nombre
* `parsejson('"21"')` : valeur unique de type dynamique contenant une chaîne

Notez que, contrairement à JavaScript, JSON impose l’utilisation de guillemets doubles (`"`) autour des chaînes. Ainsi, il est généralement plus facile de placer les littéraux de chaîne encodés JSON à l’aide de guillemets simples (`'`).

Cet exemple crée une valeur dynamique, puis utilise ses champs :

```

T
| extend person = parsejson('{"name":"Alan", "age":21, "address":{"street":432,"postcode":"JLK32P"}}')
| extend n = person.name, add = person.address.street
```


### Opérateurs et fonctions sur des types dynamiques

|||
|---|---|
| *value* `in` *array*| True s’il existe un élément de *array* qui == *value*<br/>`where City in ('London', 'Paris', 'Rome')`
| *value* `!in` *array*| True s’il n’existe aucun élément de *array* qui == *value*
|[`arraylength(`array`)`](#arraylength)| Null si ce n’est pas un tableau
|[`extractjson(`path,object`)`](#extractjson)|Utilise le chemin pour accéder à l’objet.
|[`parsejson(`source`)`](#parsejson)| Convertit une chaîne JSON en un objet dynamique.
|[`range(`from,to,step`)`](#range)| Tableau de valeurs
|[`mvexpand` listColumn](app-analytics-queries.md#mvexpand-operator) | Réplique une ligne pour chaque valeur d’une liste dans une cellule spécifiée.
|[`summarize buildschema(`column`)`](app-analytics-queries.md#summarize-operator) |Déduit le schéma du type à partir du contenu de la colonne
|[`summarize makelist(`column`)` ](app-analytics-queries.md#summarize-operator)| Aplatit des groupes de lignes et place les valeurs de la colonne dans un tableau.
|[`summarize makeset(`column`)`](app-analytics-queries.md#summarize-operator) | Aplatit des groupes de lignes et place les valeurs de la colonne dans un tableau, sans duplication.

### Objets dynamiques dans les clauses let


Les [clauses let](app-analytics-queries.md#let-clause) stockent des valeurs dynamiques sous forme de chaînes, pour que ces deux clauses soient équivalentes, et elles nécessitent toutes deux `parsejson` (ou `todynamic`) avant d’être utilisées :

    let list1 = '{"a" : "somevalue"}';
    let list2 = parsejson('{"a" : "somevalue"}');

    T | project parsejson(list1).a, parsejson(list2).a




### arraylength

Nombre d’éléments dans un tableau dynamique.

**Syntaxe**

    arraylength(array)

**Arguments**

* *array :* valeur `dynamic`.

**Retourne**

Le nombre d’éléments dans *array*, ou `null` si *array* n’est pas un tableau.

**Exemples**

```
arraylength(parsejson('[1, 2, 3, "four"]')) == 4
arraylength(parsejson('[8]')) == 1
arraylength(parsejson('[{}]')) == 1
arraylength(parsejson('[]')) == 0
arraylength(parsejson('{}')) == null
arraylength(parsejson('21')) == null
```



### extractjson

    extractjson("$.hosts[1].AvailableMB", EventText, typeof(int))

Obtient un élément spécifié à partir d’un texte JSON à l’aide d’une expression de chemin. Convertit éventuellement la chaîne extraite un type spécifique.


**Syntaxe**

```

    string extractjson(jsonPath, dataSource)​​ 
    resulttype extractjson(jsonPath, dataSource, typeof(resulttype))​​
```


**Retourne**

Cette fonction effectue une requête JsonPath dans dataSource qui contient une chaîne JSON valide, convertissant éventuellement cette valeur en un autre type selon le troisième argument.



**Exemple**

La notation entre [crochets] et la notation sous forme de points sont équivalentes :

    ... | extend AvailableMB = extractjson("$.hosts[1].AvailableMB", EventText, typeof(int)) | ...

    ... | extend AvailableMD = extractjson("$['hosts'][1]['AvailableMB']", EventText, typeof(int)) | ...



**Conseils sur les performances**

* Appliquez les clauses where avant d’utiliser `extractjson()`
* Utilisez plutôt une correspondance d’expression régulière avec [extract](#extract). L’exécution peut être beaucoup plus rapide, et elle est efficace si le JSON est généré à partir d’un modèle.
* Utilisez `parsejson()` si vous avez besoin d’extraire plusieurs valeurs à partir du JSON.
* Envisagez d’analyser le JSON lors de l’ingestion en déclarant le type de la colonne comme étant dynamique.

### Expressions de chemin JSON

|||
|---|---|
|`$`|Objet racine|
|`@`|Objet en cours|
|`[0]`|Indice de tableau|
|`.` ou `[0]` | Enfant|

*(Actuellement, nous n’implémentons pas les caractères génériques, la récursivité, l’union ou les tranches.)*




### parsejson

Interprète une `string` comme une [valeur JSON](http://json.org/) et retourne la valeur en tant que `dynamic`. Son efficacité est supérieure à l’utilisation de `extractjson()` quand vous devez extraire plusieurs éléments d’un objet composé JSON.

**Syntaxe**

    parsejson(json)

**Arguments**

* *json :* document JSON.

**Retourne**

Un objet de type `dynamic` spécifié par *json*.

**Exemple**

Dans l’exemple suivant, quand `context_custom_metrics` est une `string`, le résultat ressemble à ceci :

```
{"duration":{"value":118.0,"count":5.0,"min":100.0,"max":150.0,"stdDev":0.0,"sampledValue":118.0,"sum":118.0}}
```

Ensuite, le fragment suivant récupère la valeur de l’emplacement `duration` dans l’objet et, à partir de cette valeur, il récupère deux emplacements, `duration.value` et `duration.min` (`118.0` et `110.0`, respectivement).

```AIQL
T
| ...
| extend d=parsejson(context_custom_metrics) 
| extend duration_value=d.duration.value, duration_min=d["duration"]["min"]
```



#### range

La fonction `range()` (à ne pas confondre avec l’opérateur `range`) génère un tableau dynamique contenant une série de valeurs espacées de façon égale.

**Syntaxe**

    range(start, stop, step)

**Arguments**

* *start :* valeur du premier élément dans le tableau résultant. 
* *stop :* valeur du dernier élément dans le tableau résultant, ou valeur minimale qui est supérieure au dernier élément du tableau obtenu et dans un multiple entier de *step* à partir de *start*.
* *step :* différence entre deux éléments consécutifs du tableau.

**Exemples**

L’exemple suivant retourne `[1, 4, 7]` :

```AIQL
range(1, 8, 3)
```

L’exemple suivant retourne un tableau contenant tous les jours de l’année 2015 :

```AIQL

    range(datetime(2015-01-01), datetime(2015-12-31), 1d)
```

### todynamic

    todynamic('{"a":"a1", "b":["b1", "b2"]}')

Convertit une chaîne en une valeur dynamique.

### treepath

    treepath(dynamic_object)

Énumère toutes les expressions de chemin qui identifient les feuilles dans un objet dynamique.

**Retourne**

Un tableau d’expressions de chemin.

**Exemples**

    treepath(parsejson('{"a":"b", "c":123}')) 
    =>       ["['a']","['c']"]
    treepath(parsejson('{"prop1":[1,2,3,4], "prop2":"value2"}'))
    =>       ["['prop1']","['prop1'][0]","['prop2']"]
    treepath(parsejson('{"listProperty":[100,200,300,"abcde",{"x":"y"}]}'))
    =>       ["['listProperty']","['listProperty'][0]","['listProperty'][0]['x']"]

Notez que « [0] » indique la présence d’un tableau, mais ne spécifie pas l’index utilisé par un chemin spécifique.



[AZURE.INCLUDE [app-insights-analytics-footer](../../includes/app-insights-analytics-footer.md)]

<!---HONumber=AcomDC_0330_2016-->