

## Mises à jour de préservation de la mémoire

Pour une classe de mises à jour dans Microsoft Azure, les clients ne verront aucun impact sur leurs machines virtuelles en cours d’exécution. La plupart de ces mises à jour sont des composants ou des services qui peuvent être mis à jour sans interférer avec l'instance en cours d'exécution. Certaines de ces mises à jour sont des mises à jour d’infrastructure de la plateforme sur le système d’exploitation hôte qui peuvent être appliquées sans requérir un redémarrage complet de la machine virtuelle.

Ces mises à jour sont réalisées avec la technologie qui permet la migration dynamique sur place (ou mise à jour de « conservation de la mémoire »). Lors de la mise à jour, la machine virtuelle est mise en pause, ce qui permet de préserver la mémoire RAM, pendant que le système d’exploitation hôte sous-jacent reçoit les correctifs et mises à jour nécessaires. La machine virtuelle redémarre après 30 secondes d’arrêt. Une fois redémarrée, l’horloge de la machine virtuelle est automatiquement synchronisée.

Les mises à jour ne peuvent pas toutes êtes déployées à l’aide de ce mécanisme, mais étant donné la courte interruption, ce type de déploiement des mises à jour réduit considérablement l’impact sur les machines virtuelles.

Les mises à jour multi-instance (pour les machines virtuelles d’un groupe à haute disponibilité) se voient appliquer un seul domaine de mise à jour à la fois.

## Configurations de machine virtuelle

Deux types de configuration de machine virtuelle sont disponibles : multi-instance et d’instance unique. Dans une configuration multi-instance, les machines virtuelles similaires sont placées dans un groupe à haute disponibilité.

La configuration multi-instance fournit la redondance entre les machines physiques, la puissance et le réseau. Elle est recommandée pour garantir la disponibilité de votre application. Toutes les machines virtuelles du groupe à haute disponibilité doivent servir le même objectif à votre application.

Pour plus d’informations sur la configuration des machines virtuelles pour une haute disponibilité, consultez les articles [Gestion de la disponibilité des machines virtuelles Windows](../articles/virtual-machines/virtual-machines-windows-manage-availability.md) ou [Gestion de la disponibilité des machines virtuelles Linux](../articles/virtual-machines/virtual-machines-linux-manage-availability.md).

En revanche, une configuration à instance unique est utilisée pour les machines virtuelles autonomes qui ne sont pas placées dans un groupe à haute disponibilité. Ces machines virtuelles ne répondent pas aux conditions du contrat de niveau de service (SLA), qui requiert le déploiement de deux machines virtuelles au minimum sous le même groupe à haute disponibilité.

Pour plus d’informations sur les contrats de niveau de service, consultez la section « Services cloud, machines virtuelles et réseau virtuel » de la rubrique [Contrats de niveau de service](https://azure.microsoft.com/support/legal/sla/).


## Mises à jour de configuration multi-instance

Pendant les maintenances planifiées, la plateforme Azure met tout d’abord à jour l’ensemble des machines virtuelles qui sont hébergées dans une configuration multi-instance. Il s’ensuit un redémarrage des machines virtuelles avec environ 15 minutes d’interruption de service.

Lors d’une mise à jour de configuration multi-instance, les machines virtuelles sont mises à jour de façon à préserver la disponibilité tout au long du processus, en considérant que chaque machine virtuelle obéit à une fonction similaire à celle des autres machines du groupe.

Chaque machine virtuelle de votre groupe à haute disponibilité se voit attribuer un domaine de mise à jour et un domaine d’erreur par la plateforme Azure sous-jacente. Chaque domaine de mise à jour est un groupe de machines virtuelles qui sera redémarré dans la même fenêtre de temps. Chaque domaine d’erreur est un groupe de machines virtuelles qui partagent une source d’alimentation et un commutateur réseau communs.

Pour plus d’informations sur les domaines de mise à jour et les domaines d’erreur, consultez [Configurer plusieurs machines virtuelles dans un groupe à haute disponibilité pour la redondance](../articles/virtual-machines/virtual-machines-windows-manage-availability.md#configure-multiple-virtual-machines-in-an-availability-set-for-redundancy).

Pour empêcher les domaines de mise à jour de passer en mode hors connexion en même temps, la maintenance s’effectue de la façon suivante : arrêt de chaque machine virtuelle dans un domaine de mise à jour, application de la mise à jour aux ordinateurs hôtes, redémarrage des machines virtuelles et passage au domaine de mise à jour suivant. L’événement de maintenance planifiée se termine une fois tous les domaines mis à jour.

Les domaines de mise à jour ne sont pas nécessairement redémarrés de manière séquentielle au cours de la maintenance planifiée, mais un seul domaine de mise à jour est redémarré à la fois. Aujourd’hui, Azure fournit une notification 1 semaine à l’avance pour la maintenance planifiée des machines virtuelles dans la configuration multi-instance.

Après restauration d’une machine virtuelle, voici un exemple de ce que l’Observateur d’événements Windows peut afficher :

<!--Image reference-->
![][image2]

Utilisez l'observateur pour déterminer quelles machines virtuelles sont configurées dans une configuration à plusieurs instances à l'aide du portail Azure, Azure PowerShell ou l'interface de ligne de commande Azure. Par exemple, pour déterminer les machines virtuelles qui sont dans une configuration à plusieurs instances, vous pouvez parcourir la liste des machines virtuelles avec la colonne Groupe à haute disponibilité ajoutée à la boîte de dialogue des machines virtuelles. Dans l'exemple suivant, les machines virtuelles Exemple-VM1 et Exemple-VM2 sont dans une configuration à plusieurs instances :

<!--Image reference-->
![][image4]

## Mises à jour de configuration d’instance unique

Une fois les mises à jour de configuration multi-instance effectuées, Azure procède aux mises à jour de configuration d’instance unique. Cette mise à jour entraîne également le redémarrage des machines virtuelles qui ne sont pas exécutées dans des groupes à haute disponibilité.

Même si vous n’avez qu’une seule instance en cours d’exécution dans un groupe à haute disponibilité, la plateforme Azure la traite comme une mise à jour de configuration multi-instance.

Pour les machines virtuelles d’une configuration à instance unique, les machines virtuelles sont mises à jour en étant arrêtées, en appliquant la mise à jour à la machine hôte et en redémarrant les machines virtuelles, soit 15 minutes environ d’interruption de service. Ces mises à jour sont exécutées sur toutes les machines virtuelles d’une région dans une seule fenêtre de maintenance.

Cet événement de maintenance planifiée aura un impact sur la disponibilité de votre application pour ce type de configuration de machine virtuelle. Aujourd'hui, Azure vous fournit une notification une semaine à l'avance de la maintenance planifiée des machines virtuelles dans la configuration à instance unique.

### Notification par courrier électronique

Pour les configurations de machine virtuelle multi-instance et à instance unique uniquement, Azure envoie à l’avance un message électronique pour vous informer de la prochaine maintenance planifiée (une semaine à l’avance). Ce courrier électronique sera envoyé aux comptes de messagerie de l’administrateur et du coadministrateur du compte fournis dans l’abonnement. Voici un exemple de ce type de courrier électronique :

<!--Image reference-->
![][image1]

## Paires de régions

Lors de l’exécution de maintenance, Azure met à jour uniquement les instances de machine virtuelle d’une seule région de la paire. Par exemple, lors de la mise à jour des ordinateurs virtuels de l’Amérique du Nord central, Azure ne met à jour dans le même temps aucune machine virtuelle de l’Amérique du Nord méridionale. La mise à jour est planifiée à un autre moment, en activant le basculement ou l’équilibrage de charge entre les régions. En revanche, les autres régions, l’Europe du Nord par exemple, peuvent faire l’objet d’une maintenance en même temps que la région Est des États-Unis.

Pour plus d’informations sur les paires de régions actuelles, reportez-vous au tableau ci-dessous :

Région 1 | Région 2
:----- | ------:
Nord du centre des États-Unis | Sud du centre des États-Unis
Est des États-Unis | Ouest des États-Unis
Est des États-Unis 2 | Centre des États-Unis
Europe du Nord | Europe de l'Ouest
Asie du Sud-Est | Asie de l'Est
Chine orientale | Chine du Nord
Est du Japon | Ouest du Japon
Sud du Brésil | Sud du centre des États-Unis
Sud-est de l’Australie | Est de l’Australie
Gouvernement américain - Iowa | Gouvernement américain - Virginie

<!--Anchors-->
[image1]: ./media/virtual-machines-common-planned-maintenance/vmplanned1.png
[image2]: ./media/virtual-machines-common-planned-maintenance/EventViewerPostReboot.png
[image3]: ./media/virtual-machines-planned-maintenance/RegionPairs.PNG
[image4]: ./media/virtual-machines-common-planned-maintenance/AvailabilitySetExample.png


<!--Link references-->
[Virtual Machines Manage Availability]: ../articles/virtual-machines/virtual-machines-windows-hero-tutorial.md

[Understand planned versus unplanned maintenance]: ../articles/virtual-machines/virtual-machines-windows-manage-availability.md#Understand-planned-versus-unplanned-maintenance/

<!---HONumber=AcomDC_0330_2016-->