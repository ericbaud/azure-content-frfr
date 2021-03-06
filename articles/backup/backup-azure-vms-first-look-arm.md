<properties
	pageTitle="Protéger des machines virtuelles ARM avec Azure Backup | Microsoft Azure"
	description="Protégez des machines virtuelles ARM avec le service Azure Backup. Utilisez les sauvegardes de machines virtuelles ARM pour protéger vos données. Créez et enregistrez un archivage de Recovery Services. Enregistrez des machines virtuelles, créez une stratégie et protégez des machines virtuelles dans Azure."
	services="backup"
	documentationCenter=""
	authors="markgalioto"
	manager="jwhit"
	editor=""
	keyword="backups; vm backup"/>

<tags
	ms.service="backup"
	ms.workload="storage-backup-recovery"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="hero-article"
	ms.date="03/31/2016"
	ms.author="markgal; jimpark"/>


# Premier aperçu : sauvegarder les machines virtuelles ARM dans un archivage de Recovery Services

> [AZURE.SELECTOR]
- [Sauvegarder des machines virtuelles ARM](backup-azure-vms-first-look-arm.md)
- [Sauvegarder des machines virtuelles en mode Classique](backup-azure-vms-first-look.md)

Ce didacticiel détaille les procédures de création d'un archivage de Recovery Services et de sauvegarde d'une machine virtuelle Azure. Il s’adresse aux archivages de Recovery Services qui permettent de protéger des machines virtuelles IaaS v.2 ou ARM (Azure Resource Manager).

>[AZURE.NOTE] Dans ce didacticiel, nous partons du principe que vous disposez déjà d’une machine virtuelle dans votre abonnement Azure et que vous avez autorisé le service de sauvegarde à accéder à la machine virtuelle. Azure propose deux modèles de déploiement pour créer et utiliser des ressources : [Resource Manager et Classique](../resource-manager-deployment-model.md). Cet article concerne les machines virtuelles basées sur Resource manager et sur ARM.

Voici globalement les étapes que vous allez suivre.

1. Création d’un archivage de Recovery Services pour une machine virtuelle
2. Utilisez le portail Azure pour sélectionner un scénario, définir la stratégie et identifier les éléments à protéger.
3. Effectuez la sauvegarde initiale.



## Étape 1 - Créer un archivage de Recovery Services pour une machine virtuelle

Un archivage de Recovery Services est une entité qui stocke l’ensemble des sauvegardes et des points de récupération créés au fil du temps. L’archivage de Recovery Services contient également la stratégie de sauvegarde appliquée aux machines virtuelles protégées.

>[AZURE.NOTE] La sauvegarde de machines virtuelles est un processus local. Vous ne pouvez pas sauvegarder de machines virtuelles d’un emplacement dans un archivage de Recovery Services situé dans un autre emplacement. Donc, dans chaque emplacement Azure contenant des machines virtuelles à sauvegarder, il doit exister au moins un archivage de Recovery Services.


Pour créer un archivage de Recovery Services :

1. Connectez-vous au [portail Azure](https://portal.azure.com/).

2. Dans le menu Hub, cliquez sur **Parcourir** et, dans la liste des ressources, tapez **Recovery Services**. Au fur et à mesure des caractères saisis, la liste est filtrée. Cliquez sur **Archivage de Recovery Services**.

    ![Créer un archivage de Recovery Services - Étape 1](./media/backup-azure-vms-first-look-arm/browse-to-rs-vaults.png) <br/>

    La liste des archivages de Recovery Services est affichée.

3. Dans le menu **Archivages de Recovery Services**, cliquez sur **Ajouter**.

    ![Créer un archivage de Recovery Services - Étape 2](./media/backup-azure-vms-first-look-arm/rs-vault-menu.png)

    Dans le panneau de l’archivage de Recovery Services, renseignez les champs **Nom**, **Abonnement**, **Groupe de ressources** et **Emplacement**.

    ![Créer un archivage de Recovery Services - Étape 5](./media/backup-azure-vms-first-look-arm/rs-vault-attributes.png)

4. Sous **Nom**, entrez un nom convivial permettant d’identifier l’archivage. Le nom doit être unique pour l’abonnement Azure. Tapez un nom contenant entre 2 et 50 caractères. Il doit commencer par une lettre, et ne peut contenir que des lettres, des chiffres et des traits d’union.

5. Cliquez sur **Abonnement** pour afficher la liste des abonnements disponibles. Si vous n’êtes pas sûr de l’abonnement à utiliser, utilisez l’abonnement par défaut (ou suggéré). Vous ne disposez de plusieurs choix que si votre compte professionnel est associé à plusieurs abonnements Azure.

6. Cliquez sur **Groupe de ressources** pour afficher la liste des groupes de ressources disponibles ou sur **Nouveau** pour en créer un. Pour plus d’informations sur les groupes de ressources, consultez la section [Utilisation du portail Azure pour déployer et gérer vos ressources Azure](../azure-portal/resource-group-portal.md).

7. Dans **Emplacement**, sélectionnez la région géographique de l’archivage. L’archivage **doit** se trouver dans la même région que les machines virtuelles que vous souhaitez protéger.

    >[AZURE.IMPORTANT] Si vous ne savez pas où se trouve votre machine virtuelle, fermez la boîte de dialogue de création d’archivage et accédez à la liste des machines virtuelles dans le portail. Si vous avez des machines virtuelles dans plusieurs régions, vous devez créer un archivage de Recovery Services dans chaque région. Créez l’archivage dans le premier emplacement avant de passer à l'emplacement suivant. Il est inutile de spécifier des comptes de stockage dans lesquels héberger les données de sauvegarde : l’archivage de Recovery Services et le service Azure Backup gèrent cela automatiquement.

8. Cliquez sur **Créer**. La création de l’archivage de Recovery Services peut prendre un certain temps. Surveillez les notifications d'état dans l'angle supérieur droit du portail. Une fois votre archivage créé, il s'ouvre dans le portail.

9. Dans votre archivage, cliquez sur **Tous les paramètres** > **Configuration de la sauvegarde** pour afficher le **type de réplication de stockage**. Choisissez l’option de réplication de stockage correspondant à votre archivage.

    ![Liste des archivages de sauvegarde](./media/backup-azure-vms-first-look-arm/choose-storage-configuration.png)

    Par défaut, votre archivage utilise un stockage géo-redondant. Si vous utilisez Azure comme principal point de terminaison du stockage de sauvegarde, nous vous recommandons de continuer d’utiliser le stockage géo-redondant. Si vous utilisez Azure comme point de terminaison secondaire du stockage de sauvegarde, vous pouvez envisager de choisir un stockage redondant en local, de manière à réduire les coûts du stockage de données dans Azure. Pour en savoir plus sur les options de stockage [géo-redondant](../storage/storage-redundancy.md#geo-redundant-storage) et [localement redondant](../storage/storage-redundancy.md#locally-redundant-storage), consultez cette [présentation](../storage/storage-redundancy.md).

    Après avoir sélectionné l’option de stockage pour votre archivage, vous pouvez associer la machine virtuelle à l’archivage. Pour commencer l’association, vous devez découvrir et enregistrer les machines virtuelles Azure.

## Étape 2 - Sélectionner le scénario, définir la stratégie et définir les éléments à protéger
Avant d’enregistrer une machine virtuelle dans un archivage, lancez le processus de découverte pour vérifier que les nouvelles machines virtuelles ajoutées à l’abonnement sont bien identifiées. Le processus interroge Azure pour obtenir la liste des machines virtuelles de l’abonnement et des informations supplémentaires, comme le nom du service cloud et la région.

1. Si vous avez un archivage de Recovery Services ouvert, passez à l’étape 2. Si vous n'avez aucun archivage de Recovery Services ouvert mais que vous êtes dans le portail Azure, cliquez sur **Parcourir** dans le menu Hub.

    - Dans la liste des ressources, tapez **Recovery Services**.
    - Au fur et à mesure des caractères saisis, la liste est filtrée. Lorsque vous voyez **Archivages de Recovery Services**, cliquez dessus.

    ![Créer un archivage de Recovery Services - Étape 1](./media/backup-azure-vms-first-look-arm/browse-to-rs-vaults.png) <br/>

    La liste des archivages de Recovery Services s’affiche.

    - Dans la liste des archivages de Recovery Services, sélectionnez un archivage.

    Le tableau de bord de l’archivage sélectionné s'ouvre.

    ![Ouvrir le panneau de l’archivage](./media/backup-azure-vms-first-look-arm/vault-settings.png)

2. Dans le menu du tableau de bord de l’archivage, cliquez sur **Sauvegarder** pour ouvrir le panneau Sauvegarde.

    ![Ouvrir le panneau Sauvegarde](./media/backup-azure-vms-first-look-arm/backup-button.png)

    Lorsque le panneau s'ouvre, le service Backup recherche les nouvelles machines virtuelles dans l'abonnement.

    ![Détection des machines virtuelles](./media/backup-azure-vms-first-look-arm/discovering-new-vms.png)

3. Dans le panneau de sauvegarde, cliquez sur **Scénario** pour ouvrir le panneau Scénario.

    ![Ouvrir le panneau Scénario](./media/backup-azure-vms-first-look-arm/select-backup-scenario-one.png)

4. Sur le panneau Scénario, dans le menu **Type de sauvegarde**, sélectionnez **Sauvegarde de machine virtuelle Azure** et cliquez sur **OK**.

    ![Ouvrir le panneau Scénario](./media/backup-azure-vms-first-look-arm/select-rs-backup-scenario-two.png)

    Le panneau Scénario se ferme et le panneau Stratégie de sauvegarde s'ouvre.

5. Dans le panneau Sauvegarde, sélectionnez la stratégie de sauvegarde à appliquer à l’archivage et cliquez sur **OK**.

    ![Sélectionner la stratégie de sauvegarde](./media/backup-azure-vms-first-look-arm/setting-rs-backup-policy.png)

    La stratégie par défaut est indiquée dans les détails. Si vous souhaitez créer une stratégie, sélectionnez **Créer**. Pour des instructions sur la définition d'une stratégie de sauvegarde, consultez la section [Définition d'une stratégie de sauvegarde](backup-azure-vms-first-look-arm.md#defining-a-backup-policy). Lorsque vous cliquez sur OK, la stratégie de sauvegarde est associée à l’archivage. Ensuite, choisissez les machines virtuelles à associer à l'archivage.

6. Sélectionnez les machines virtuelles à associer à la stratégie spécifiée et cliquez sur **Sélectionner**.

    ![Sélectionner la charge de travail](./media/backup-azure-vms-first-look-arm/select-vms-to-backup.png)

    Si la machine virtuelle souhaitée n’apparaît pas dans la liste, cliquez sur **Actualiser**. Si vous ne voyez toujours pas la machine virtuelle souhaitée, vérifiez qu'elle existe dans le même emplacement Azure que l’archivage de Recovery Services.

7. Maintenant que vous avez défini tous les paramètres de l’archivage, dans le panneau Sauvegarde, cliquez sur **Activer la sauvegarde** en bas de la page. La stratégie est déployée dans l’archivage et les machines virtuelles.

    ![Activer la sauvegarde](./media/backup-azure-vms-first-look-arm/enable-backup-settings.png)


## Étape 3 - Sauvegarde initiale

Le déploiement d’une stratégie de sauvegarde sur la machine virtuelle ne signifie pas que les données ont été sauvegardées. Par défaut, la première sauvegarde planifiée (définie dans la stratégie de sauvegarde) est la sauvegarde initiale. Jusqu'à celle-ci, l’État de la dernière sauvegarde sur le panneau **Travaux de sauvegarde** est **Avertissement (sauvegarde initiale en attente)**.

![Sauvegarde en attente](./media/backup-azure-vms-first-look-arm/initial-backup-not-run.png)

Sauf si votre sauvegarde initiale est imminente, il est recommandé de cliquer sur **Sauvegarder maintenant**.

Pour exécuter **Sauvegarder maintenant** :

1. Sur le tableau de bord de l’archivage, dans la vignette **Sauvegarde**, cliquez sur **Machines virtuelles Azure** <br/> ![Icône Paramètres](./media/backup-azure-vms-first-look-arm/rs-vault-in-dashboard-backup-vms.png)

    Le panneau **Éléments de sauvegarde** s’ouvre.

2. Dans le panneau **Éléments de sauvegarde**, cliquez avec le bouton droit sur l'archivage à sauvegarder, puis cliquez sur **Sauvegarder maintenant**.

    ![Icône Paramètres](./media/backup-azure-vms-first-look-arm/back-up-now.png)

    Le travail de sauvegarde débute. <br/>

    ![Travail de sauvegarde déclenché](./media/backup-azure-vms-first-look-arm/backup-triggered.png)

3. Pour vérifier si votre sauvegarde initiale est terminée, dans la vignette **Travaux de sauvegarde** du tableau de bord de l’archivage, cliquez sur **Machines virtuelles Azure**.

    ![Vignette Travaux de sauvegarde](./media/backup-azure-vms-first-look-arm/open-backup-jobs.png)

    Le panneau Travaux de sauvegarde s’ouvre.

4. Le panneau Travaux de sauvegarde indique l’état de tous les travaux.

    ![Vignette Travaux de sauvegarde](./media/backup-azure-vms-first-look-arm/backup-jobs-in-jobs-view.png)

    >[AZURE.NOTE] Dans le cadre de l’opération de sauvegarde, le service Azure Backup émet une commande vers l’extension de sauvegarde de chaque machine virtuelle pour vider toutes les écritures et prendre un instantané cohérent.

    Lorsque le travail de sauvegarde est terminé, l’état affiché est *Terminé*.

## Définition d’une stratégie de sauvegarde

Une stratégie de sauvegarde définit quand les instantanés de données sont pris ainsi que leur durée de conservation. Lors de la définition de la stratégie de sauvegarde d'une machine virtuelle, vous pouvez déclencher un travail de sauvegarde *une fois par jour*. Lorsque vous créez une stratégie, elle est appliquée à l’archivage. L'interface de la stratégie de sauvegarde ressemble à ceci :

![Stratégie de sauvegarde](./media/backup-azure-vms-first-look-arm/backup-policy-daily-raw.png)

Pour créer une stratégie :

1. Dans **Nom de la stratégie**, indiquez le nom de la stratégie.

2. Les instantanés de vos données peuvent être pris à intervalles quotidiens ou hebdomadaires. Utilisez le menu déroulant **Fréquence de sauvegarde** pour déterminer la fréquence de prise des instantanés de données (tous les jours ou toutes les semaines.

    - Si vous choisissez un intervalle quotidien, utilisez le contrôle en surbrillance pour sélectionner l'heure du jour à laquelle prendre l'instantané. Pour modifier l'heure, désélectionnez-la et choisissez-en une autre.

    ![Stratégie de sauvegarde quotidienne](./media/backup-azure-vms-first-look-arm/backup-policy-daily.png) <br/>

    - Si vous choisissez un intervalle hebdomadaire, utilisez les contrôles en surbrillance pour sélectionner le(s) jour(s) de la semaine et l’heure auxquels prendre l’instantané. Dans le menu du jour, sélectionnez un ou plusieurs jours. Dans le menu de l'heure, sélectionnez une heure. Pour modifier l'heure, désélectionnez-la et choisissez-en une autre.

    ![Stratégie de sauvegarde hebdomadaire](./media/backup-azure-vms-first-look-arm/backup-policy-weekly.png)

3. Par défaut, toutes les options **Durée de rétention** sont sélectionnées. Désactivez la limite de plage de rétention que vous ne souhaitez pas utiliser.

    >[AZURE.NOTE] Lorsque vous protégez un ordinateur virtuel, un travail de sauvegarde s'exécute une fois par jour. L'heure d'exécution de la sauvegarde est le même pour chaque plage de rétention.

    Dans les contrôles correspondants, spécifiez le ou les intervalles à utiliser. Les durées de rétention mensuelle et annuelle vous permettent de prendre des instantanés selon un intervalle hebdomadaire ou quotidien.

4. Après avoir défini toutes les options de la stratégie, cliquez sur **OK** en bas du panneau.

    La nouvelle stratégie s’applique à l’archivage de Recovery Services, dès que ses paramètres sont définis. Revenir à l'étape 6 de la section [Sélectionner le scénario, définir la stratégie et définir les éléments à protéger](backup-azure-vms-first-look-arm.md#step-2---select-scenario-set-policy-and-define-items-to-protect)

## Installer l’agent de machine virtuelle sur la machine virtuelle

Ces informations sont fournies en cas de nécessité. L’agent de machine virtuelle Azure doit être installé sur la machine virtuelle Azure pour permettre la prise en charge de l’extension Backup. Cependant, si votre machine virtuelle a été créée à partir de la galerie Azure, l’agent y est déjà installé. L’agent de machine virtuelle n’est pas préinstallé sur les machines virtuelles qui ont été migrées à partir de centres de données locaux. Dans ce cas, il faut l’installer de manière explicite. Si vous rencontrez des problèmes de sauvegarde de la machine virtuelle Azure, vérifiez que son agent est correctement installé sur celle-ci (reportez-vous au tableau ci-dessous). Si vous créez une machine virtuelle personnalisée, [assurez-vous que la case **Installer l’agent de machine virtuelle** est bien cochée](../virtual-machines/virtual-machines-windows-classic-agents-and-extensions.md) avant de commencer à configurer la machine virtuelle.

En savoir plus sur l’[agent de machine virtuelle](https://go.microsoft.com/fwLink/?LinkID=390493&clcid=0x409) et [comment l’installer](../virtual-machines/virtual-machines-windows-classic-manage-extensions.md).

Le tableau suivant fournit des informations supplémentaires sur l’agent de machine virtuelle pour les machines virtuelles Windows et Linux.

| **Opération** | **Windows** | **Linux** |
| --- | --- | --- |
| Installation de l’agent de machine virtuelle | <li>Téléchargez et installez le fichier [MSI de l’agent](http://go.microsoft.com/fwlink/?LinkID=394789&clcid=0x409). Vous aurez besoin de privilèges d’administrateur pour terminer l’installation. <li>[Mettez à jour la propriété de la machine virtuelle](http://blogs.msdn.com/b/mast/archive/2014/04/08/install-the-vm-agent-on-an-existing-azure-vm.aspx) pour indiquer que l’agent est installé. | <li>Installez l’[agent Linux](https://github.com/Azure/WALinuxAgent) le plus récent à partir de GitHub. Vous aurez besoin de privilèges d’administrateur pour terminer l’installation. <li> [Mettez à jour la propriété de la machine virtuelle](http://blogs.msdn.com/b/mast/archive/2014/04/08/install-the-vm-agent-on-an-existing-azure-vm.aspx) pour indiquer que l’agent est installé. |
| Mise à jour de l’agent de machine virtuelle | La mise à jour de l’agent de machine virtuelle est aussi simple que la réinstallation des [fichiers binaires de l’agent de machine virtuelle](http://go.microsoft.com/fwlink/?LinkID=394789&clcid=0x409). <br>Vérifiez qu’aucune opération de sauvegarde n’est en cours pendant la mise à jour de l’agent de machine virtuelle. | Suivez les instructions fournies dans la section [Mise à jour de l’agent de machine virtuelle Linux](../virtual-machines-linux-update-agent.md). <br>Vérifiez qu’aucune opération de sauvegarde n’est en cours pendant la mise à jour de l’agent de machine virtuelle. |
| Validation de l’installation de l’agent de machine virtuelle | <li>Accédez au dossier *C:\\WindowsAzure\\Packages* sur la machine virtuelle Azure. <li>Le fichier WaAppAgent.exe doit être présent.<li> Cliquez avec le bouton droit sur le fichier, accédez à **Propriétés**, puis sélectionnez l’onglet **Détails**. Le champ Version du produit doit être défini sur 2.6.1198.718 ou une version ultérieure. | N/A |


### Extension de sauvegarde

Une fois l’agent de machine virtuelle installé sur la machine virtuelle, le service Azure Backup installe l’extension de sauvegarde vers l’agent de machine virtuelle. Le service Azure Backup met à niveau et corrige en toute transparence l'extension de sauvegarde sans intervention supplémentaire de l'utilisateur.

Le service Backup installe l’extension de sauvegarde que la machine virtuelle soit ou non en cours d’exécution. Une machine virtuelle en cours d’exécution offre le plus de chance d’obtenir un point de récupération d’application cohérent. Toutefois, le service Azure Backup poursuit la sauvegarde de la machine virtuelle, même si elle est éteinte et si l’extension n’a pas été installée, autrement dit si la machine virtuelle est hors connexion. Dans ce cas, le point de récupération sera *cohérent en cas d’incident*.



## Résolution de problèmes
Si vous rencontrez des problèmes pour accomplir certaines tâches décrites dans cet article, consultez la section [Conseils pour la résolution de problèmes](backup-azure-vms-troubleshoot.md).


## Des questions ?
Si vous avez des questions ou si vous souhaitez que certaines fonctionnalités soient incluses, [envoyez-nous vos commentaires](http://aka.ms/azurebackup_feedback).

<!---HONumber=AcomDC_0406_2016-->