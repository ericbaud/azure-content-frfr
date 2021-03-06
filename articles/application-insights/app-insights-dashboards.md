<properties
	pageTitle="Tableaux de bord dans le portail Application Insights"
	description="Une fois que vous avez configuré votre application pour qu'elle envoie des données de télémétrie à Application Insights, ce guide vous montre comment vous repérer dans le portail."
	services="application-insights"
    documentationCenter=""
	authors="alancameronwills"
	manager="douge"/>

<tags
	ms.service="application-insights"
	ms.workload="tbd"
	ms.tgt_pltfrm="ibiza"
	ms.devlang="multiple"
	ms.topic="article" 
	ms.date="02/25/2016"
	ms.author="awills"/>

# Tableaux de bord et navigation dans le portail Application Insights

Une fois que vous avez [configuré Application Insights sur votre projet](app-insights-overview.md), les données de télémétrie relatives aux performances et à l’utilisation de votre application apparaissent dans les ressources Application Insights de votre projet dans le [portail Azure](https://portal.azure.com).

## Le tableau de bord

Lorsque vous vous connectez au [portail Azure](https://portal.azure.com), vous accédez immédiatement au tableau de bord. Vous pouvez le personnaliser ou le placer en mode plein écran. Cet exemple a été personnalisé pour afficher les graphiques principaux qui intéressent ses propriétaires.


![Un tableau de bord personnalisé.](./media/app-insights-dashboards/30.png)

1. Cliquez sur le coin supérieur à tout moment pour revenir au tableau de bord.
2. L’option **+ Nouveau** permet de créer une nouvelle ressource. Une [ressource Application Insights](app-insights-create-new-resource.md) est un emplacement de stockage et d’analyse des données de télémétrie de votre application.
3. La barre de navigation ouvre vos ressources existantes.
4. Modifiez et créez des tableaux de bord à l’aide de la barre d’outils du tableau de bord.

## Rechercher vos données de télémétrie

Connectez-vous au [portail Azure](https://portal.azure.com) et accédez à la ressource Application Insights que vous avez créée pour votre application.

![Cliquez sur Parcourir, sélectionnez Application Insights, puis sélectionnez votre application.](./media/app-insights-dashboards/00-start.png)

La page Vue d’ensemble fournit des données de télémétrie de base et offre une diversité de liens. Le contenu dépend du type de votre application et peut être personnalisé.



## Période

Vous pouvez modifier l’intervalle de temps sur lequel portent les graphiques et les grilles dans n’importe quel panneau.

![Ouvrez le panneau Vue d'ensemble de votre application dans le portail Azure](./media/app-insights-dashboards/03-range.png)


Si vous attendez des données qui ne sont pas encore affichées, cliquez sur Actualiser. Les graphiques s’actualisent régulièrement, mais plus les intervalles de temps sur lesquels ils portent sont étendus, plus les intervalles d’actualisation sont longs. Dans la version finale, les données peuvent mettre un certain temps pour passer du pipeline d'analyse au graphique.

Pour effectuer un zoom sur une partie d’un graphique, sélectionnez la partie souhaitée, puis cliquez sur l’icône de loupe :

![Sélectionnez une partie d'un graphique.](./media/app-insights-dashboards/12-drag.png)



## Valeurs de granularité et de point

Pointez votre souris sur le graphique pour afficher les valeurs des mesures à ce stade.

![Pointez la souris sur un graphique](./media/app-insights-dashboards/02-focus.png)

La valeur de la mesure à un moment donné est agrégée à l'intervalle d'échantillonnage précédent.

L’intervalle d’échantillonnage, ou « granularité », s’affiche en haut du panneau.

![En-tête du panneau.](./media/app-insights-dashboards/11-grain.png)

Vous pouvez ajuster le niveau de granularité dans le panneau Période :

![En-tête du panneau.](./media/app-insights-dashboards/grain.png)

Les niveaux de granularité disponibles dépendent de la période que vous sélectionnez. Les niveaux de granularité explicites sont des alternatives à la granularité « automatique » pour la période.

## Panneau Vue d’ensemble de l’application

Le panneau de vue d’ensemble (page) de votre application affiche un résumé des principales mesures de diagnostic de votre application et constitue une passerelle vers les autres fonctionnalités du portail.

Cliquez sur :

* **n'importe quel graphique** pour afficher plus de détails ;
* **Paramètres** pour accéder à des pages prédéfinies d’autres mesures ;
* **Metrics Explorer** pour créer des pages de mesures de votre choix ;
* **Rechercher** pour analyser des instances spécifiques d'événements tels que les demandes, les exceptions ou les suivis de journal.


![Procédures principales d'affichage de vos données de télémétrie](./media/app-insights-dashboards/010-oview.png)


### Personnalisation du panneau Vue d’ensemble 

Choisissez ce que vous souhaitez afficher sur le panneau Vue d’ensemble. Dans la zone de personnalisation, vous pouvez insérer des titres de section, faire glisser des vignettes et des graphiques, supprimer des éléments et ajouter de nouvelles vignettes et graphiques à partir de la galerie.

![Cliquez sur Modifier. Faites glisser les vignettes et les graphiques. Ajoutez des vignettes de la galerie. Cliquez ensuite sur Terminé.](./media/app-insights-dashboards/020-customize.png)

## Tableaux de bord

Le tableau de bord du portail Azure est la page d’accueil que vous voyez lors de votre première connexion [au portail](https://portal.azure.com). Vous pouvez y rassembler des graphiques et des vignettes (groupes de graphiques) à partir de plusieurs ressources.

![Cliquez sur Modifier. Faites glisser les vignettes et les graphiques. Ajoutez des vignettes de la galerie. Cliquez ensuite sur Terminé.](./media/app-insights-dashboards/30.png)

Lorsque vous examinez un panneau ou un graphique qui est particulièrement intéressant, vous pouvez l’épingler au tableau de bord. Celui-ci sera affiché lors de votre prochain accès au tableau de bord.

![Pour épingler un graphique, passez la souris sur celui-ci, puis cliquez sur « ... » dans l’en-tête.](./media/app-insights-dashboards/33.png)

Vous pouvez enregistrer plusieurs tableaux de bord et basculer entre ceux-ci. Lorsque vous épinglez un graphique ou un panneau, il est ajouté au tableau de bord actuel.

![Pour basculer entre les tableaux de bord, cliquez sur Tableau de bord et sélectionnez un tableau de bord enregistré. Pour créer et enregistrer un nouveau tableau de bord, cliquez sur Nouveau. Pour les réorganiser, cliquez sur Modifier.](./media/app-insights-dashboards/32.png)

Vous pouvez, par exemple, avoir un tableau de bord pour l’affichage en plein écran dans la salle de réunion et un autre pour le développement général.


Sur le tableau de bord, un panneau s’affiche sous forme de vignette : cliquez dessus pour accéder au panneau. Un graphique réplique le graphique dans son emplacement d’origine.


![](./media/app-insights-dashboards/35.png)


## Panneaux de mesures

Lorsque vous cliquez dans la vue d'ensemble pour obtenir plus de détails, vous êtes dans Metrics Explorer (même le nom est plus spécifique).

Vous pouvez également utiliser le bouton Metrics Explorer pour créer un nouveau panneau que vous pouvez modifier et enregistrer.


![Dans le panneau Vue d'ensemble, cliquez sur Métriques](./media/app-insights-dashboards/16-metrics.png)

### Modification des graphiques et des grilles

Pour ajouter un nouveau graphique au panneau :

![Dans Metrics Explorer, sélectionnez Ajouter un graphique](./media/app-insights-dashboards/04-add.png)

Sélectionnez un graphique existant ou un nouveau graphique pour modifier ce qu'il présente :

![Sélectionner une ou plusieurs mesures](./media/app-insights-dashboards/08-select.png)

Vous pouvez afficher plusieurs mesures dans un graphique, bien qu'il existe des restrictions sur les combinaisons affichables. Dès que vous sélectionnez une mesure, certaines autres sont désactivées.

Si vous avez ajouté des [mesures personnalisées](app-insights-api-custom-events-metrics.md#track-metric) au code de votre application (appels à TrackMetric et mesures liées à des appels TrackEvent), elles sont répertoriées ici.

### Segmenter vos données

Sélectionnez un graphique ou une grille, basculez vers le regroupement et choisissez une propriété de regroupement :

![Activez le regroupement, puis une propriété de regroupement](./media/app-insights-dashboards/15-segment.png)

Si vous avez ajouté des mesures personnalisées au code de votre application et qu'elles incluent des [valeurs de propriétés](app-insights-api-custom-events-metrics.md#properties), vous pourrez sélectionner la propriété dans la liste.

Le graphique est trop petit pour les données segmentées ? Ajustez la hauteur :

![Ajustez le curseur.](./media/app-insights-dashboards/18-height.png)

### Filtrer vos données

Pour afficher uniquement les mesures d'un jeu de valeurs de propriété sélectionné :

![Cliquez sur le filtre, développez une propriété et vérifiez les valeurs](./media/app-insights-dashboards/19-filter.png)

Si vous ne sélectionnez aucune valeur pour une propriété particulière, c'est la même chose que si vous aviez sélectionné toutes les valeurs : il n'existe aucun filtre sur cette propriété.

Notez le nombre d'événements en même temps que chaque valeur de propriété. Lorsque vous sélectionnez les valeurs d'une propriété, le nombre est modifié, en même temps que les autres valeurs de la propriété.

### Enregistrer votre panneau de mesures

Une fois que vous avez créé des graphiques, enregistrez-les en tant que favoris. Si vous travaillez avec un compte professionnel, vous pouvez choisir de les partager avec d'autres membres de l'équipe.

![Choisissez Favori](./media/app-insights-dashboards/21-favorite-save.png)

Pour réafficher le panneau, **allez dans le panneau Vue d'ensemble** et ouvrez Favoris :

![Dans le panneau Vue d'ensemble, cliquez sur Favoris](./media/app-insights-dashboards/22-favorite-get.png)

Si vous avez choisi une période relative lorsque vous avez enregistré, le panneau sera mis à jour avec les dernières mesures. Si vous avez choisi une période absolue, il affiche les mêmes données à chaque fois.

### Réinitialiser le panneau

Si vous modifiez un panneau, mais que vous souhaitez revenir à la configuration originale enregistrée, cliquez sur Réinitialiser.

![Dans les boutons en haut de Metrics Explorer](./media/app-insights-dashboards/17-reset.png)

## Search

La recherche affiche des événements individuels tels que les affichages de pages, les demandes, les exceptions, les suivis de journal et les événements personnalisés. Il n'affiche pas des mesures agrégées ou d'instances de l'appel TrackMetric().

> [AZURE.NOTE] Si votre application génère un volume important de télémétrie (et si vous utilisez le kit de développement logiciel ASP.NET version 2.0.0-beta3 ou ultérieure), le module d'échantillonnage adaptatiif réduit automatiquement le volume qui est envoyé vers le portail en envoyant uniquement une fraction représentative des événements. Cependant, les événements liés à la même demande seront activés ou désactivés en tant que groupe, afin que vous puissiez naviguer entre les événements connexes. [En savoir plus sur l'échantillonnage](app-insights-sampling.md).

Ouvrez la recherche de diagnostic :

![Open diagnostic search](./media/app-insights-dashboards/01-open-Diagnostic.png)

Ouvrez le panneau Filtre et choisissez les types d’événement que vous souhaitez afficher. (Si vous souhaitez restaurer plus tard les filtres avec lesquels vous avez ouvert le panneau, cliquez sur Réinitialiser).

![Choisissez le filtre et sélectionnez les types de télémétrie](./media/app-insights-dashboards/02-filter-req.png)

### Filtrer des valeurs de propriétés

Vous pouvez filtrer les événements en fonction des valeurs de leurs propriétés. Les propriétés disponibles varient en fonction des types d’événement que vous avez sélectionnés.

Par exemple, les demandes avec un code de réponse spécifique.

![Développez une propriété et choisissez une valeur](./media/app-insights-dashboards/03-response500.png)

Si vous ne choisissez aucune valeur pour une propriété, cela a le même effet que si vous sélectionniez toutes les valeurs : le filtrage sur cette propriété est désactivé.

> [AZURE.NOTE] Si votre application génère un volume important de télémétrie, le module d'échantillonnage adaptatiif réduit automatiquement le volume qui est envoyé vers le portail en envoyant uniquement une fraction représentative des événements. Les événements qui font partie de la même opération seront activés ou désactivés en tant que groupe, afin que vous puissiez naviguer entre les événements connexes. [En savoir plus sur l'échantillonnage.](app-insights-sampling.md)


### Affiner votre recherche

Notez que les nombres à droite des valeurs de filtre affichent le nombre d’occurrences dans le jeu actuellement filtré.

Dans cet exemple, il est clair que la demande `Reports/Employees` provoque la majorité des 500 erreurs :

![Développez une propriété et choisissez une valeur](./media/app-insights-dashboards/04-failingReq.png)

En outre, si vous voulez également voir les autres événements qui se sont produits pendant ce temps, vous pouvez vérifier **les événements dont les propriétés ne sont pas définies**.

### Enregistrer votre recherche

Une fois que vous avez défini tous les filtres que vous voulez, vous pouvez enregistrer la recherche dans vos recherches favorites. Si vous travaillez avec un compte professionnel, vous pouvez choisir de la partager avec d’autres membres de l’équipe.

![Cliquez sur Favori, définissez le nom et cliquez sur Enregistrer](./media/app-insights-dashboards/08-favorite-save.png)


Pour réafficher la recherche, **allez dans le panneau Vue d’ensemble** et ouvrez Favoris :

![Vignette des favoris](./media/app-insights-dashboards/22-favorite-get.png)

Si vous avez enregistré une période relative, le panneau rouvert comporte les données les plus récentes. Si vous avez enregistré une période absolue, vous voyez les mêmes données à chaque fois.

<!---------HONumber=AcomDC_0309_2016-->