<properties
	pageTitle="Mapper le stockage dans Azure Site Recovery pour la réplication de machines virtuelles Hyper-V entre centres de données locaux | Microsoft Azure"
	description="Préparez le mappage de stockage pour la réplication de machines virtuelles Hyper-V entre deux centres de données locaux avec Azure Site Recovery."
	services="site-recovery"
	documentationCenter=""
	authors="rayne-wiselman"
	manager="jwhit"
	editor=""/>

<tags
	ms.service="site-recovery"
	ms.devlang="na"
	ms.topic="article"
	ms.tgt_pltfrm="na"
	ms.workload="storage-backup-recovery"
	ms.date="02/22/2016"
	ms.author="raynew"/>


# Préparer le mappage de stockage pour la réplication de machines virtuelles Hyper-V entre deux centres de données locaux avec Azure Site Recovery


Azure Site Recovery contribue à mettre en œuvre la stratégie de continuité des activités et de récupération d'urgence de votre entreprise en coordonnant la réplication, le basculement et la récupération de machines virtuelles et de serveurs physiques. Cet article décrit le mappage de stockage, qui vous permet d’utiliser le stockage de façon optimale lorsque vous utilisez Site Recovery pour répliquer des machines virtuelles Hyper-V entre deux centres de données VMM locaux.

Publiez des commentaires ou des questions au bas de cet article ou sur le [Forum Azure Recovery Services](https://social.msdn.microsoft.com/forums/azure/home?forum=hypervrecovmgr).

## Vue d'ensemble

Le mappage de stockage est uniquement pertinent lorsque vous répliquez des machines virtuelles Hyper-V qui se trouvent dans des clouds VMM à partir d’un centre de données principal vers un centre de données secondaire à l’aide de Réplica Hyper-V ou de la réplication SAN, comme suit :


- **Réplication d’un site local vers un autre avec Réplica Hyper-V**: configurez le mappage de stockage en mappant les classifications de stockage sur des serveurs VMM source et cible pour effectuer les opérations suivantes :

	- **Identifier le stockage cible des ordinateurs virtuels de réplica**: les ordinateurs virtuels seront répliqués sur une cible de stockage (partage de SMB ou volumes partagés de cluster (VPC)) que vous choisissez.
	- **Placement d’ordinateurs virtuels de réplica**: le mappage de stockage permet d’optimiser le positionnement d’ordinateurs virtuels de réplica sur des serveurs hôtes Hyper-V. Les ordinateurs virtuels de réplica seront placés sur des hôtes ayant accès à la classification de stockage mappé.
	- **Aucun mappage de stockage**: si vous ne configurez aucun mappage de stockage, les machines virtuelles seront répliquées sur l’emplacement de stockage par défaut spécifié sur le serveur hôte Hyper-V associé à la machine virtuelle de réplica.

- **Réplication d’un site local vers un autre avec SAN**: configurez le mappage de stockage en mappant les pools de groupes de stockage sur des serveurs VMM source et cible.
	- **Spécifier le pool**: spécifie le pool de stockage secondaire qui doit recevoir les données de réplication du pool principal.
	- **Identifier les pools de stockage cible**: garantit que les numéros d’unités logiques dans un groupe de réplication source sont répliqués dans le pool de stockage cible mappé de votre choix.

## Configurer des classifications de stockage pour la réplication Hyper-V

Lorsque vous utilisez Réplica Hyper-V pour la réplication avec Site Recovery, vous créez un mappage entre les classifications de stockage sur les serveurs VMM source et cible ou sur un seul serveur VMM si deux sites sont gérés par le même serveur VMM. Notez les points suivants :

- Lorsque le mappage est configuré correctement et que la réplication est activée, le disque dur virtuel d’un ordinateur virtuel à l’emplacement primaire est répliqué sur le stockage dans l’emplacement cible mappé.
- Les classifications de stockage doivent être accessibles aux groupes hôtes situés dans les clouds source et cible.
- Il n'est pas obligatoire qu'elles aient le même type de stockage. Par exemple, vous pouvez mapper une classification source contenant des partages SMB à une classification cible contenant des volumes partagés de cluster.
- Pour en savoir plus, consultez [Comment créer des classifications de stockage dans VMM](https://technet.microsoft.com/library/gg610685.aspx).

## exemples

Si les classifications sont correctement configurées dans VMM lorsque vous sélectionnez les serveurs VMM source et cible lors du mappage de stockage, les classifications source et cible s’affichent. Voici un exemple de partages de fichiers de stockage et de classifications pour une entreprise ayant deux sites à New York et Chicago.

**Emplacement** | **Serveur VMM** | **Partage de fichiers (source)** | **Classification (source)** | **Mappé à** | **Partage de fichiers (cible)**
---|---|--- |---|---|---
New York | VMM\_Source| SourceShare1 | GOLD | GOLD\_TARGET | TargetShare1
 | | SourceShare2 | SILVER | SILVER\_TARGET | TargetShare2
 | | SourceShare3 | BRONZE | BRONZE\_TARGET | TargetShare3
Chicago | VMM\_Target | | GOLD\_TARGET | Non mappé |
| | | SILVER\_TARGET | Non mappé |
 | | | BRONZE\_TARGET | Non mappé

Vous pouvez les configurer dans l’onglet **Serveur de stockage** de la page **Ressources** du portail Site Recovery.

![configurer le mappage de stockage](./media/site-recovery-storage-mapping/storage-mapping1.png)

Dans cet exemple : - Lorsqu'un ordinateur virtuel de réplica est créé pour un ordinateur virtuel sur le stockage GOLD (SourceShare1), il est répliqué sur un stockage GOLD\_TARGET (TargetShare1). -Lorsqu'un ordinateur virtuel de réplica est créé pour un ordinateur virtuel sur un stockage SILVER (SourceShare2), il est répliqué sur un stockage SILVER\_TARGET (TargetShare2), et ainsi de suite.

Les partages de fichiers et leurs classifications affectées dans VMM apparaissent dans la capture d’écran suivante.

![Classifications de stockage dans VMM](./media/site-recovery-storage-mapping/storage-mapping2.png)

## Emplacements de stockage multiples

Si la classification cible est affectée à plusieurs partages SMB ou VPC, l’emplacement de stockage optimal est sélectionné automatiquement lorsque l’ordinateur virtuel est protégé. Si aucun stockage cible approprié n'est disponible avec la classification spécifiée, l'emplacement de stockage par défaut spécifié sur l'hôte Hyper-V est utilisé pour placer les disques durs virtuels de réplica.

Le tableau suivant affiche la classification de stockage et les volumes partagés de cluster sont configurés dans notre exemple.

**Emplacement** | **Classification** | **Stockage associé**
---|---|---
New York | GOLD | <p>C:\\ClusterStorage\\SourceVolume1</p><p>\\FileServer\\SourceShare1</p>
 | SILVER | <p>C:\\ClusterStorage\\SourceVolume2</p><p>\\FileServer\\SourceShare2</p>
Chicago | GOLD\_TARGET | <p>C:\\ClusterStorage\\TargetVolume1</p><p>\\FileServer\\TargetShare1</p>
 | SILVER\_TARGET| <p>C:\\ClusterStorage\\TargetVolume2</p><p>\\FileServer\\TargetShare2</p>

Ce tableau récapitule le comportement lorsque vous activez la protection des ordinateurs virtuels (VM1 à VM5) dans cet exemple d'environnement.

**Ordinateur virtuel** | **Stockage source** | **Classification source** | **Stockage cible mappé**
---|---|---|---
MV1 | C:\\ClusterStorage\\SourceVolume1 | GOLD | <p>C:\\ClusterStorage\\SourceVolume1</p><p>\\\FileServer\\SourceShare1</p><p>Both GOLD\_TARGET</p>
MV2 | \\FileServer\\SourceShare1 | GOLD | <p>C:\\ClusterStorage\\SourceVolume1</p><p>\\FileServer\\SourceShare1</p> <p>Both GOLD\_TARGET</p>
MV3 | C:\\ClusterStorage\\SourceVolume2 | SILVER | <p>C:\\ClusterStorage\\SourceVolume2</p><p>\\FileServer\\SourceShare2</p>
MV4 | \\FileServer\\SourceShare2 | SILVER |<p>C:\\ClusterStorage\\SourceVolume2</p><p>\\FileServer\\SourceShare2</p><p>Both SILVER\_TARGET</p>
MV5 | C:\\ClusterStorage\\SourceVolume3 | N/A | Aucun mappage. Donc, l’emplacement de stockage par défaut de l’hôte Hyper-V est utilisé.

## Étapes suivantes

Maintenant que vous comprenez mieux le mappage de stockage, consultez [Meilleures pratiques du déploiement de Site Recovery](site-recovery-best-practices.md).

<!---HONumber=AcomDC_0224_2016-->