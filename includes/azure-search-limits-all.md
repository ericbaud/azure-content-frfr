Ressource|Gratuit|De base (version préliminaire) <sup>6</sup>|S1|S2 <sup>7</sup>
---|---|---|---|----
Nombre maximum de services de recherche|N/A|---|12 par abonnement Azure|12 par abonnement Azure 
Taille de stockage maximum <sup>1</sup>|50 Mo ou 10 000 documents|2 Go par service|25 Go par partition ou 300 Go de documents par service|100 Go par partition ou 1,2 To par service 
Nombre maximum de documents hébergés|10 000 au total|1 million par service|15 millions par partition (jusqu’à 180 millions de documents par service)|60 millions par partition (Jusqu’à 720 millions de documents par service) 
Nombre maximum d’index|3|5|50|200 
Nombre maximum d’indexeurs|3|5|50|200 
Sources de données d’indexeur maximum|3|5|50|200 
Index : nombre maximum de champs par index|1000|100 <sup>5</sup>|1000|1000 
Index : nombre maximum de profils de score par index|16|16|16|16 
Index : nombre maximum de fonctions par profil|8|8|8|8 
Indexeurs : charge d’indexation maximum par appel|10 000 documents|Limité uniquement par le nombre maximum de documents|Limité uniquement par le nombre maximum de documents|Limité uniquement par le nombre maximum de documents 
Indexeurs : durée d’exécution maximum|3 minutes|24 heures|24 heures|24 heures 
Requêtes par seconde <sup>2</sup>|N/A|~3 par réplica|~15 par réplica|~60 par réplica 
Montée en charge : nombre maximum d’unités de recherche <sup>3</sup>|N/A|Jusqu’à 3 unités (3 réplicas et 1 partition)|36 unités|36 unités 
Tarification <sup>4</sup>|N/A|75 $ par unité de recherche par mois|250 $ par unité de recherche par mois|1 000 $ par unité de recherche par mois

<sup>1</sup> La taille de stockage est un volume fixe ou le nombre de documents par service, en fonction de la première occurrence.

<sup>2</sup> Le nombre de requêtes par seconde est une approximation basée sur une méthode heuristique. Il est calculé à l’aide de charges de travail client simulées et réelles pour obtenir des valeurs estimées. Le débit exact du nombre de requêtes par seconde varie en fonction de vos données et de la nature de la requête.

<sup>3</sup> Les unités de recherche sont l’unité facturable pour un réplica ou une partition. Vous avez besoin des deux pour les opérations de stockage, d’indexation et de requête. Consultez la [planification de la capacité](../articles/search/search-capacity-planning.md) pour obtenir des combinaisons valides de réplicas et des partitions qui vous permettront de rester dans la limite maximale de 3 ou 36 unités (pour les versions De base et Standard, respectivement).

<sup>4</sup> Le prix indiqué vaut pour le marché américain et illustre les coûts relatifs sur différents niveaux. Différents marchés ont différents prix. Reportez-vous à la [page de tarification](https://azure.microsoft.com/pricing/details/search/) pour connaître les frais dans d’autres devises. Les frais sont indiqués par unité de recherche (SU). Au niveau S1, une configuration de 3 unités de recherche (par exemple, 3 réplicas et 1 partition) coûterait 750 $ par mois en moyenne. Si vous descendez en puissance et réduisez le nombre d’unités de recherche au cours du moins, la facture et adaptée en conséquence et vous ne payez que ce que vous utilisez.

<sup>5</sup> Il ne s’agit pas d’une erreur typographique. Le niveau de base a une limite de 100 champs par index. Il s’agit de la seule couche dotée de cette limite inférieure.

<sup>6</sup> Le [niveau de base](http://aka.ms/azuresearchbasic) est disponible à un prix inférieur de 50 % du prix complet pendant la période de version préliminaire.

<sup>7</sup> S2 nécessite un ticket de service. Il ne peut pas être configuré dans le portail. Si vous avez besoin d’aide pour la prise en main, envoyez un courrier électronique à azuresearch_contact@microsoft.com.

<!-----HONumber=AcomDC_0316_2016-->