<properties
	pageTitle="Utiliser un prédicat de filtre pour sélectionner les lignes à migrer (Stretch Database) | Microsoft Azure"
	description="Apprenez à utiliser un prédicat de filtre pour sélectionner les lignes à migrer."
	services="sql-server-stretch-database"
	documentationCenter=""
	authors="douglaslMS"
	manager=""
	editor=""/>

<tags
	ms.service="sql-server-stretch-database"
	ms.workload="data-management"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="02/26/2016"
	ms.author="douglasl"/>

# Utiliser un prédicat de filtre pour sélectionner les lignes à migrer (Stretch Database)

Si vous stockez des données historiques dans une table distincte, vous pouvez configurer Stretch Database pour migrer la totalité de la table. Si votre table contient à la fois des données historiques et des données actuelles, d’autre part, vous pouvez spécifier un prédicat de filtre pour sélectionner les lignes à transférer. Le prédicat de filtre doit appeler une fonction tabulaire inline. Cette rubrique explique comment écrire une fonction tabulaire inline pour sélectionner les lignes à migrer.

Dans la version CTP 3.1 via RC1, l’option permettant de spécifier un prédicat n’est pas disponible dans l’Assistant Activation de base de données pour Stretch. Vous devez utiliser l’instruction ALTER TABLE pour configurer l’extension de base de données avec cette option. Pour en savoir plus, consultez [ALTER TABLE (Transact-SQL)](https://msdn.microsoft.com/library/ms190273.aspx).

Si vous ne spécifiez pas de prédicat de filtre, la table entière est migrée.

>   [AZURE.NOTE] Si vous fournissez un prédicat de filtre qui fonctionne mal, la migration de données est elle aussi médiocre. Stretch Database applique le prédicat de filtre à la table à l’aide de l’opérateur CROSS APPLY.

## Exigences de base pour la fonction tabulaire inline
La fonction tabulaire inline requise pour une fonction de filtre Stretch Database ressemble à l’exemple qui suit.

```tsql
CREATE FUNCTION dbo.fn_stretchpredicate(@column1 datatype1, @column2 datatype2 [, ...n])
RETURNS TABLE
WITH SCHEMABINDING
AS
RETURN	SELECT 1 AS is_eligible
		WHERE <predicate>
```
Les paramètres de la fonction doivent être des identificateurs pour les colonnes de la table.

La liaison de schéma est nécessaire pour éviter que les colonnes utilisées par le prédicat de filtre soient supprimées ou modifiées.

### Valeur de retour
Si la fonction retourne un résultat non vide, la ligne est éligible à la migration ; autrement dit, si la fonction ne retourne aucune ligne, la ligne ne peut être retenue pour la migration.

### Conditions
Le &lt;*prédicat*&gt; peut comporter une condition ou plusieurs conditions reliées entre elles par l’opérateur logique AND.

```
<predicate> ::= <condition> [ AND <condition> ] [ ...n ]
```
Chaque condition peut à son tour se composer d’une condition primitive ou de plusieurs conditions primitives jointes avec l’opérateur logique OR.

```
<condition> ::= <primitive_condition> [ OR <primitive_condition> ] [ ...n ]
```

### Conditions primitives
Une condition primitive peut effectuer l’une des comparaisons suivantes.

```
<primitive_condition> ::=
{
<function_parameter> <comparison_operator> constant
| <function_parameter> { IS NULL | IS NOT NULL }
| <function_parameter> IN ( constant [ ,...n ] )
}
```

-   Comparer un paramètre de fonction à une expression constante. Par exemple, `@column1 < 1000`.

    Voici un exemple qui vérifie si la valeur de la colonne *date* est &lt; 1/1/2016.

    ```tsql
    CREATE FUNCTION dbo.fn_stretchpredicate(@column1 datetime)
    RETURNS TABLE
    WITH SCHEMABINDING
    AS
    RETURN	SELECT 1 AS is_eligible
    		WHERE @column1 < CONVERT(datetime, '1/1/2016', 101)
    GO

    ALTER TABLE stretch_table_name SET ( REMOTE_DATA_ARCHIVE = ON (
    	FILTER_PREDICATE = dbo.fn_stretchpredicate(date),
    	MIGRATION_STATE = OUTBOUND
    ) )
    ```

-   Appliquez l’opérateur IS NULL ou IS NOT NULL à un paramètre de fonction.

-   Utilisez l’opérateur IN pour comparer un paramètre de fonction à une liste de valeurs constantes.

    Voici un exemple qui vérifie si la valeur de la colonne *shipment\_status* est `IN (N'Completed', N'Returned', N'Cancelled')`.

    ```tsql
    CREATE FUNCTION dbo.fn_stretchpredicate(@column1 nvarchar(15))
    RETURNS TABLE
    WITH SCHEMABINDING
    AS
    RETURN	SELECT 1 AS is_eligible
    		WHERE @column1 IN (N'Completed', N'Returned', N'Cancelled')
    GO

    ALTER TABLE table1 SET ( REMOTE_DATA_ARCHIVE = ON (
    	FILTER_PREDICATE = dbo.fn_stretchpredicate(shipment_status),
    	MIGRATION_STATE = OUTBOUND
    ) )
    ```

### Opérateurs de comparaison
Les opérateurs de comparaison suivants sont pris en charge.

`<, <=, >, >=, =, <>, !=, !<, !>`

```
<comparison_operator> ::= { < | <= | > | >= | = | <> | != | !< | !> }
```

### Expressions constantes
Les constantes utilisées dans un prédicat de filtre peuvent être n’importe quelle expression déterministe pouvant être évaluée lors de la définition de la fonction. Les expressions constantes peuvent contenir les éléments suivants.

-   Littéraux. Par exemple, `N’abc’, 123`.

-   Expressions algébriques. Par exemple, `123 + 456`.

-   Fonctions déterministes. Par exemple, `SQRT(900)`.

-   Conversions déterministes utilisant CAST ou CONVERT. Par exemple, `CONVERT(datetime, '1/1/2016', 101)`.

### Autres expressions
Vous pouvez utiliser les opérateurs BETWEEN et NOT BETWEEN si le prédicat obtenu est conforme aux règles décrites ici une fois que vous avez remplacé les opérateurs BETWEEN et NOT BETWEEN par les expressions AND et OR équivalentes.

Vous ne pouvez pas utiliser de sous-requêtes ou de fonctions non déterministes comme RAND() ou GETDATE().

## Exemples de fonctions valides

-   L’exemple suivant combine deux conditions primitives en utilisant l’opérateur logique AND.

    ```tsql
    CREATE FUNCTION dbo.fn_stretchpredicate((@column1 datetime, @column2 nvarchar(15))
    RETURNS TABLE
    WITH SCHEMABINDING
    AS
    RETURN	SELECT 1 AS is_eligible
      WHERE @column1 < N'20150101' AND @column2 IN (N'Completed', N'Returned', N'Cancelled')
    GO

    ALTER TABLE table1 SET ( REMOTE_DATA_ARCHIVE = ON (
    	FILTER_PREDICATE = dbo.fn_stretchpredicate(date, shipment_status),
    	MIGRATION_STATE = OUTBOUND
    ) )
    ```

-   L’exemple suivant utilise plusieurs conditions et une conversion déterministe avec CONVERT.

    ```tsql
    CREATE FUNCTION dbo.fn_stretchpredicate_example1(@column1 datetime, @column2 int, @column3 nvarchar)
    RETURNS TABLE
    WITH SCHEMABINDING
    AS
    RETURN	SELECT 1 AS is_eligible
        WHERE @column1 < CONVERT(datetime, '1/1/2015', 101)AND (@column2 < -100 OR @column2 > 100 OR @column2 IS NULL)AND @column3 IN (N'Completed', N'Returned', N'Cancelled')
    GO
    ```

-   L’exemple suivant utilise des fonctions et des opérateurs mathématiques.

    ```tsql
    CREATE FUNCTION dbo.fn_stretchpredicate_example2(@column1 float)
    RETURNS TABLE
    WITH SCHEMABINDING
    AS
    RETURN	SELECT 1 AS is_eligible
    		WHERE @column1 < SQRT(400) + 10
    GO
    ```

-   L’exemple suivant utilise les opérateurs BETWEEN et NOT BETWEEN. Cette utilisation est valide parce que le prédicat obtenu est conforme aux règles décrites ici une fois que vous avez remplacé les opérateurs BETWEEN et NOT BETWEEN par les expressions AND et OR équivalentes.

    ```tsql
    CREATE FUNCTION dbo.fn_stretchpredicate_example3(@column1 int, @column2 int)
    RETURNS TABLE
    WITH SCHEMABINDING
    AS
    RETURN	SELECT 1 AS is_eligible
    		WHERE @column1 BETWEEN 0 AND 100
    			AND (@column2 NOT BETWEEN 200 AND 300 OR @column1 = 50)
    GO
    ```
    La fonction qui précède est équivalente à la fonction suivante une fois que vous avez remplacé les opérateurs BETWEEN et NOT BETWEEN par les expressions AND et OR équivalentes.

    ```tsql
    CREATE FUNCTION dbo.fn_stretchpredicate_example4(@column1 int, @column2 int)
    RETURNS TABLE
    WITH SCHEMABINDING
    AS
    RETURN	SELECT 1 AS is_eligible
    		WHERE @column1 >= 0 AND @column1 <= 100AND (@column2 < 200 OR @column2 > 300 OR @column1 = 50)
    GO
    ```

## Exemples de fonctions non valides

-   La fonction qui suit est non valide, car elle contient une conversion non déterministe.

    ```tsql
    CREATE FUNCTION dbo.fn_example5(@column1 datetime)
    RETURNS TABLE
    WITH SCHEMABINDING
    AS
    RETURN	SELECT 1 AS is_eligible
    		WHERE @column1 < CONVERT(datetime, '1/1/2016')
    GO
    ```

-   La fonction qui suit est non valide, car elle contient un appel de fonction non déterministe.

    ```tsql
    CREATE FUNCTION dbo.fn_example6(@column1 datetime)
    RETURNS TABLE
    WITH SCHEMABINDING
    AS
    RETURN	SELECT 1 AS is_eligible
    		WHERE @column1 < DATEADD(day, -60, GETDATE())
    GO
    ```

-   La fonction qui suit est non valide, car elle contient une sous-requête.

    ```tsql
    CREATE FUNCTION dbo.fn_example7(@column1 int)
    RETURNS TABLE
    WITH SCHEMABINDING
    AS
    RETURN	SELECT 1 AS is_eligible
    		WHERE @column1 IN (SELECT SupplierID FROM Supplier WHERE Status = 'Defunct'))
    GO
    ```

-   Les fonctions suivantes ne sont pas valides, car les expressions qui utilisent des opérateurs algébriques ou des fonctions intégrées doivent s’évaluer à une constante lorsque vous définissez la fonction. Vous ne pouvez pas inclure des références de colonne dans les expressions algébriques ou les appels de fonction.

    ```tsql
    CREATE FUNCTION dbo.fn_example8(@column1 int)
    RETURNS TABLE
    WITH SCHEMABINDING
    AS
    RETURN	SELECT 1 AS is_eligible
    		WHERE @column1 % 2 =  0
    GO

    CREATE FUNCTION dbo.fn_example9(@column1 int)
    RETURNS TABLE
    WITH SCHEMABINDING
    AS
    RETURN	SELECT 1 AS is_eligible
    		WHERE SQRT(@column1) = 30
    GO
    ```

-   La fonction suivante est non valide, car elle enfreint les règles décrites ici une fois que vous avez remplacé l’opérateur BETWEEN avec l’expression AND équivalente.

    ```tsql
    CREATE FUNCTION dbo.fn_example10(@column1 int, @column2 int)
    RETURNS TABLE
    WITH SCHEMABINDING
    AS
    RETURN	SELECT 1 AS is_eligible
    		WHERE (@column1 BETWEEN 1 AND 200 OR @column1 = 300) AND @column2 > 1000
    GO
    ```
    La fonction qui précède est équivalente à la fonction suivante une fois que vous avez remplacé l’opérateur BETWEEN par l’expression AND équivalente. Cette fonction est non valide parce que les conditions primitives ne peuvent utiliser que l’opérateur logique OR.

    ```tsql
    CREATE FUNCTION dbo.fn_example11(@column1 int, @column2 int)
    RETURNS TABLE
    WITH SCHEMABINDING
    AS
    RETURN	SELECT 1 AS is_eligible
    		WHERE (@column1 >= 1 AND @column1 <= 200 OR @column1 = 300) AND @column2 > 1000
    GO
    ```

## Comment Stretch Database applique-t-il le prédicat de filtre ?
Stretch Database applique le prédicat de filtre à la table et détermine les lignes éligibles à l’aide de l’opérateur CROSS APPLY. Par exemple :

```tsql
SELECT * FROM stretch_table_name CROSS APPLY fn_stretchpredicate(column1, column2)
```
Si la fonction retourne un résultat non vide pour la ligne, la ligne éligible à la migration.

## Ajouter un prédicat de filtre à une table
Ajoutez un prédicat de filtre à une table en exécutant l’instruction ALTER TABLE et en spécifiant une fonction tabulaire inline existante comme valeur du paramètre FILTER\_PREDICATE. Par exemple :

```tsql
ALTER TABLE stretch_table_name SET ( REMOTE_DATA_ARCHIVE = ON (
	FILTER_PREDICATE = dbo.fn_stretchpredicate(column1, column2),
	MIGRATION_STATE = <desired_migration_state>
) )
```
Après avoir lié la fonction à la table en tant que prédicat, les éléments qui suivent sont vrais.

-   Lors de la migration de données suivante, seules les lignes pour lesquelles la fonction retourne une valeur non vide sont migrées.

-   Les colonnes utilisées par la fonction sont liées par schéma. Vous ne pouvez pas modifier ces colonnes tant qu’une table utilise la fonction comme prédicat de filtre.

## Supprimer un prédicat de filtre d’une table
Pour migrer la table entière plutôt que des lignes sélectionnées, supprimez le FILTER\_PREDICATE existant en lui affectant la valeur null. Par exemple :

```tsql
ALTER TABLE stretch_table_name SET ( REMOTE_DATA_ARCHIVE = ON (
	FILTER_PREDICATE = NULL,
	MIGRATION_STATE = <desired_migration_state>
) )
```
Une fois que vous avez supprimé le prédicat de filtre, toutes les lignes de la table sont éligibles à la migration.

## Remplacer un prédicat de filtre existant
Vous pouvez remplacer le prédicat de filtre spécifié précédemment en exécutant à nouveau l’instruction ALTER TABLE et en spécifiant une nouvelle valeur pour le paramètre FILTER\_PREDICATE. Par exemple :

```tsql
ALTER TABLE stretch_table_name SET ( REMOTE_DATA_ARCHIVE = ON (
	FILTER_PREDICATE = dbo.fn_stretchpredicate2(column1, column2),
	MIGRATION_STATE = <desired_migration_state>
```
La nouvelle fonction tabulaire inline est configurée de la façon suivante.

-   La nouvelle fonction doit être moins restrictive que la fonction précédente.

-   Tous les opérateurs qui existaient dans l’ancienne fonction doivent exister dans la nouvelle fonction.

-   La nouvelle fonction ne peut pas contenir d’opérateur qui n’existe pas dans l’ancienne fonction.

-   L’ordre des arguments d’opérateur ne peut être modifié.

-   Seules les valeurs constantes qui font partie d’une comparaison `<, <=, >, >=` peuvent être modifiées de manière à rendre le prédicat moins restrictif.

### Exemple de remplacement valide
Supposons que la fonction suivante est le prédicat de filtre actuel.

```tsql
CREATE FUNCTION dbo.fn_stretchpredicate_old (@column1 datetime, @column2 int)
RETURNS TABLE
WITH SCHEMABINDING
AS
RETURN	SELECT 1 AS is_eligible
		WHERE @column1 < CONVERT(datetime, '1/1/2016', 101)
			AND (@column2 < -100 OR @column2 > 100)
GO
```
La fonction suivante est un remplacement valide, car la nouvelle constante de date (qui spécifie une date limite ultérieure) rend le prédicat moins restrictif.

```tsql
CREATE FUNCTION dbo.fn_stretchpredicate_new (@column1 datetime, @column2 int)
RETURNS TABLE
WITH SCHEMABINDING
AS
RETURN	SELECT 1 AS is_eligible
		WHERE @column1 < CONVERT(datetime, '2/1/2016', 101)
			AND (@column2 < -50 OR @column2 > 50)
GO
```

### Exemples de remplacement non valides
La fonction suivante n’est pas un remplacement valide, car la nouvelle date de constante (qui spécifie une date de coupure précédemment) ne rend pas le prédicat moins restrictif.

```tsql
CREATE FUNCTION dbo.fn_notvalidreplacement_1 (@column1 datetime, @column2 int)
RETURNS TABLE
WITH SCHEMABINDING
AS
RETURN	SELECT 1 AS is_eligible
		WHERE @column1 < CONVERT(datetime, '1/1/2015', 101)
			AND (@column2 < -100 OR @column2 > 100)
GO
```
La fonction suivante n’est pas un remplacement valide, car l’un des opérateurs de comparaison a été supprimé.

```tsql
CREATE FUNCTION dbo.fn_notvalidreplacement_2 (@column1 datetime, @column2 int)
RETURNS TABLE
WITH SCHEMABINDING
AS
RETURN	SELECT 1 AS is_eligible
		WHERE @column1 < CONVERT(datetime, '1/1/2016', 101)
			AND (@column2 < -50)
GO
```
La fonction suivante n’est pas un remplacement valide, car une nouvelle condition a été ajoutée avec l’opérateur logique AND.

```tsql
CREATE FUNCTION dbo.fn_notvalidreplacement_3 (@column1 datetime, @column2 int)
RETURNS TABLE
WITH SCHEMABINDING
AS
RETURN	SELECT 1 AS is_eligible
		WHERE @column1 < CONVERT(datetime, '1/1/2016', 101)
			AND (@column2 < -100 OR @column2 > 100)
			AND (@column2 <> 0)
GO
```

## Annuler un prédicat de filtre
Il est impossible de supprimer la fonction tabulaire inline tant qu’une table utilise la fonction en tant que prédicat de filtre.

## Vérifier le prédicat de filtre appliqué à une table
Pour vérifier le prédicat de filtre appliqué à une table, ouvrez la vue de catalogue **sys.remote\_data\_archive\_tables** et vérifiez la valeur de la colonne **filter\_predicate**. Si la valeur est null, la table entière est éligible à l’archivage. Pour plus d’informations, voir [sys.remote\_data\_archive\_tables (Transact-SQL)](https://msdn.microsoft.com/library/dn935003.aspx).

## Voir aussi

[ALTER TABLE (Transact-SQL)](https://msdn.microsoft.com/library/ms190273.aspx)

<!---HONumber=AcomDC_0330_2016-->