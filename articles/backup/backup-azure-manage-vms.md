
<properties
	pageTitle="Gestion et surveillance des sauvegardes de machines virtuelles Azure | Microsoft Azure"
	description="Découvrez comment gérer et surveiller les sauvegardes d'une machine virtuelle Azure"
	services="backup"
	documentationCenter=""
	authors="trinadhk"
	manager="shreeshd"
	editor=""/>

<tags
	ms.service="backup"
	ms.workload="storage-backup-recovery"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="01/25/2016"
	ms.author="trinadhk; jimpark; markgal;"/>

# Gestion et surveillance des sauvegardes de machines virtuelles Azure

## Gérer des machines virtuelles protégées

Pour gérer des machines virtuelles protégées :

1. Pour afficher et gérer les paramètres de sauvegarde pour une machine virtuelle, cliquez sur l’onglet **Éléments protégés**.

2. Cliquez sur le nom d’un élément protégé pour afficher l’onglet **Détails de la sauvegarde** qui vous présente des informations sur la dernière sauvegarde.

    ![Sauvegarde de machine virtuelle](./media/backup-azure-manage-vms/backup-vmdetails.png)

3. Pour afficher et gérer les paramètres de stratégie de sauvegarde pour une machine virtuelle, cliquez sur l’onglet **Stratégies**.

    ![Stratégie de machine virtuelle](./media/backup-azure-manage-vms/manage-policy-settings.png)

    L’onglet **Stratégies de sauvegarde** affiche la stratégie existante. Vous pouvez la modifier en fonction de vos besoins. Si vous devez créer une stratégie, cliquez sur **Créer** dans la page **Stratégies**. Notez que si vous voulez supprimer une stratégie, aucune machine virtuelle ne doit lui être associée.

    ![Stratégie de machine virtuelle](./media/backup-azure-manage-vms/backup-vmpolicy.png)

4. Vous pouvez obtenir plus d’informations sur l’état ou les actions d’une machine virtuelle dans la page **Tâches**. Cliquez sur une tâche dans la liste pour obtenir plus de détails, ou filtrez les tâches pour une machine virtuelle spécifique.

    ![Travaux](./media/backup-azure-manage-vms/backup-job.png)

## Sauvegarde à la demande d’une machine virtuelle
Vous pouvez effectuer une sauvegarde à la demande d’une machine virtuelle une fois que celle-ci est configurée pour la protection. Si la sauvegarde initiale est en attente pour la machine virtuelle, la sauvegarde à la demande crée une copie complète de la machine virtuelle dans l’archivage de sauvegarde Azure. Si la première sauvegarde est terminée, la sauvegarde à la demande envoie seulement les modifications apportées lors de la sauvegarde précédente à l’archivage de sauvegarde Azure.

Pour créer une sauvegarde à la demande d’une machine virtuelle :

1. Accédez à la page **Éléments protégés** et sélectionnez **Machine virtuelle Azure** comme **Type** (si elle n’est pas déjà sélectionnée), puis cliquez sur le bouton **Sélectionner**.

    ![Type de machine virtuelle](./media/backup-azure-manage-vms/vm-type.png)

2. Sélectionnez la machine virtuelle sur laquelle vous souhaitez effectuer la sauvegarde à la demande, puis cliquez sur le bouton **Sauvegarder maintenant** au bas de la page.

    ![Sauvegarder maintenant](./media/backup-azure-manage-vms/backup-now.png)

    Cette opération permet de créer un travail de sauvegarde sur la machine virtuelle sélectionnée. La plage de rétention de point de récupération créée via ce travail est identique à celle spécifiée dans la stratégie associée à la machine virtuelle.

    ![Création du travail de sauvegarde](./media/backup-azure-manage-vms/creating-job.png)

    >[AZURE.NOTE] Pour afficher la stratégie associée à une machine virtuelle, accédez à cette dernière dans la page **Éléments protégés**, puis à l’onglet Stratégie de sauvegarde.

3. Une fois que le travail est créé, vous pouvez cliquer sur le bouton **Afficher le travail** dans la barre de notification pour voir le travail correspondant dans la page Travaux.

    ![Travail de sauvegarde créé](./media/backup-azure-manage-vms/created-job.png)

4. Après l’achèvement réussi du travail, le point de récupération créé vous permet de restaurer la machine virtuelle. Cela permet également d’incrémenter la valeur de colonne de point de récupération de 1 dans la page **Éléments protégés**.

## Arrêt de la protection des machines virtuelles
Vous pouvez choisir d’arrêter les sauvegardes futures d’une machine virtuelle avec les options suivantes :

- Conserver les données de sauvegarde associées à la machine virtuelle dans l’archivage de sauvegarde Azure
- Supprimer les données de sauvegarde associées à la machine virtuelle

Si vous avez choisi de conserver des données de sauvegarde associées à la machine virtuelle, vous pouvez utiliser les données de sauvegarde pour restaurer la machine virtuelle. Pour connaître les détails de la tarification de ces machines virtuelles, cliquez [ici](https://azure.microsoft.com/pricing/details/backup/).

Pour arrêter la protection d’une machine virtuelle :

1. Accédez à la page **Éléments protégés** et sélectionnez **Machine virtuelle Azure** comme type de filtre (si ce n’est pas déjà fait), puis cliquez sur le bouton **Sélectionner**.

    ![Type de machine virtuelle](./media/backup-azure-manage-vms/vm-type.png)

2. Sélectionnez la machine virtuelle et cliquez sur **Arrêter la protection** en bas de la page.

    ![Arrêter la protection](./media/backup-azure-manage-vms/stop-protection.png)

3. Par défaut, Azure Backup ne supprime pas les données de sauvegarde associées à la machine virtuelle.

    ![Confirmation l’arrêt de la protection](./media/backup-azure-manage-vms/confirm-stop-protection.png)

    Si vous souhaitez supprimer les données de sauvegarde, sélectionnez la case à cocher.

    ![Case à cocher](./media/backup-azure-manage-vms/checkbox.png)

    Sélectionnez le motif de l’arrêt de la sauvegarde. Bien que cela soit facultatif, le fait d’indiquer un motif permet à Azure Backup de l’utiliser pour hiérarchiser les scénarios clients.

4. Cliquez sur le bouton **Envoyer** pour envoyer le travail **Arrêter la protection**. Cliquez sur **Afficher le travail** pour voir le travail correspondant dans la page **Travaux**.

    ![Arrêter la protection](./media/backup-azure-manage-vms/stop-protect-success.png)

    Si vous n’avez pas sélectionné l’option **Supprimer les données de sauvegarde associées** dans l’Assistant **Arrêter la protection**, à la fin du travail, l’état de la protection devient **Protection arrêtée**. Les données restent dans Azure Backup jusqu'à ce qu'elles soient explicitement supprimées. Vous pouvez toujours supprimer les données en sélectionnant la machine virtuelle sur la page **Éléments protégés** et en cliquant sur **Supprimer**.

    ![Protection arrêtée](./media/backup-azure-manage-vms/protection-stopped-status.png)

    Si vous avez sélectionné l’option **Supprimer les données de sauvegarde associées**, la machine virtuelle ne figure pas dans la page **Éléments protégés**.

## Application d’une nouvelle protection à la machine virtuelle
Si vous n’avez pas sélectionné l’option **Supprimer les données de sauvegarde associées** dans l’Assistant **Arrêter la protection**, vous pouvez à nouveau protéger la machine virtuelle en suivant les étapes similaires à celles de la sauvegarde de machines virtuelles inscrites. Une fois la protection effectuée, les données de cette machine virtuelle sont conservées avant l’arrêt de la protection et des points de récupération sont créés après l’application de la nouvelle protection.

Après l’application d’une nouvelle protection, l’état de protection de la machine virtuelle devient **Protégé** s’il existe des points de récupération antérieurs à l’opération **Arrêter la protection**.

  ![Machine virtuelle à nouveau protégée](./media/backup-azure-manage-vms/reprotected-status.png)

>[AZURE.NOTE] Lors de l’application d’une nouvelle protection à la machine virtuelle, vous pouvez choisir une autre stratégie que la stratégie avec laquelle la machine virtuelle a été initialement protégée.

## Annulation de l’inscription des machines virtuelles

Si vous voulez supprimer la machine virtuelle de l’archivage de sauvegarde :

1. Cliquez sur le bouton **ANNULER L’INSCRIPTION** en bas de la page.

    ![Désactiver la protection](./media/backup-azure-manage-vms/unregister-button.png)

    Une notification toast s’affiche en bas de l’écran pour demander confirmation. Cliquez sur **OUI** pour continuer.

    ![Désactiver la protection](./media/backup-azure-manage-vms/confirm-unregister.png)

## Suppression des données de sauvegarde
Vous pouvez supprimer les données de sauvegarde associées à une machine virtuelle, soit :

- Au cours du travail Arrêter la protection
- Après la fin du travail d’arrêt de la protection sur une machine virtuelle

Pour supprimer les données de sauvegarde sur une machine virtuelle à l’état *Protection arrêtée* à l’issue d’une tâche **Arrêter la sauvegarde** :

1. Accédez à la page **Éléments protégés** et sélectionnez **Machine virtuelle Azure** comme *type*, puis cliquez sur le bouton **Sélectionner**.

    ![Type de machine virtuelle](./media/backup-azure-manage-vms/vm-type.png)

2. Sélectionnez la machine virtuelle. La machine virtuelle apparaît avec l’état **Protection arrêtée**.

    ![Protection arrêtée](./media/backup-azure-manage-vms/protection-stopped-b.png)

3. Cliquez sur le bouton **SUPPRIMER** au bas de la page.

    ![Supprimer la sauvegarde](./media/backup-azure-manage-vms/delete-backup.png)

4. Dans l’Assistant **Supprimer les données de sauvegarde**, sélectionnez un motif de suppression des données de sauvegarde (hautement recommandé), puis cliquez sur **Soumettre**.

    ![Supprimer les données de sauvegarde](./media/backup-azure-manage-vms/delete-backup-data.png)

5. Cette opération crée un travail pour supprimer les données de sauvegarde de la machine virtuelle sélectionnée. Cliquez sur **Afficher le travail** pour afficher le travail correspondant dans la page Travaux.

    ![Suppression de données réussie](./media/backup-azure-manage-vms/delete-data-success.png)

    Une fois le travail terminé, l’entrée correspondant à la machine virtuelle est supprimée de la page **Éléments protégés**.

## Tableau de bord
Dans la page **Tableau de bord**, vous pouvez consulter les informations des machines virtuelles Azure, leur stockage et leurs tâches associées au cours des dernières 24 heures. Vous pouvez afficher l’état de la sauvegarde et les éventuelles erreurs de sauvegarde associées.

![Tableau de bord](./media/backup-azure-manage-vms/dashboard-protectedvms.png)

>[AZURE.NOTE] Les valeurs du tableau de bord sont actualisées toutes les 24 heures.

## Audit des opérations
La sauvegarde Azure fournit l’analyse des « journaux d’opérations » pour les opérations de sauvegarde déclenchées par le client, ce qui vous permet de savoir exactement quelles sont les opérations de gestion exécutées sur le coffre de sauvegarde. Les journaux d’opérations activent l’assistance post mortem et d’audit des opérations de sauvegarde.

Les opérations suivantes sont enregistrées dans les journaux des opérations :

- S’inscrire
- Annuler l’inscription
- Configurer la protection
- Sauvegarde (planifiée et à la demande via BackupNow)
- Restauration
- Arrêter la protection
- Supprimer les données de sauvegarde
- Add policy
- Supprimer la stratégie
- Mettre à jour la stratégie
- Annuler le travail

Pour afficher les journaux des opérations correspondant à un coffre de sauvegarde :

1. Accédez aux **Services de gestion** sur le portail Azure, puis cliquez sur l’onglet **Journaux des opérations**.

    ![Journaux des opérations](./media/backup-azure-manage-vms/ops-logs.png)

2. Dans les filtres, sélectionnez **Sauvegarde** comme *Type* et spécifiez le nom du coffre de sauvegarde dans *nom de service*, puis cliquez sur **Soumettre**.

    ![Filtre des journaux des opérations](./media/backup-azure-manage-vms/ops-logs-filter.png)

3. Dans les journaux des opérations, sélectionnez une opération, puis cliquez sur **Détails** pour afficher les détails correspondant à une opération.

    ![Détails de l’extraction des journaux d’opérations](./media/backup-azure-manage-vms/ops-logs-details.png)

    L’**Assistant Détails** contient des informations sur l’opération déclenchée, l’ID de tâche, la ressource sur laquelle cette opération est déclenchée et l’heure de début de l’opération.

    ![Détails de l'opération](./media/backup-azure-manage-vms/ops-logs-details-window.png)

## Notifications d’alerte
Vous pouvez obtenir des notifications d’alerte personnalisées pour les travaux du portail. Pour cela, vous devez définir des règles d’alerte basées sur PowerShell sur les événements de journaux des opérations.

Travail d’alertes basé sur les événements en mode ressource Azure. Basculer en mode de ressources Azure en exécutant l’applet de commande suivant en mode de commande élevé :

```
PS C:\> Switch-AzureMode AzureResourceManager
```

Pour définir une notification personnalisée et signaler les échecs de sauvegarde, un exemple de commande doit présenter :

```
PS C:\> Add-AlertRule -Operator GreaterThanOrEqual -Threshold 1 -ResourceId '/subscriptions/86eeac34-eth9a-4de3-84db-7a27d121967e/resourceGroups/RecoveryServices-DP2RCXUGWS3MLJF4LKPI3A3OMJ2DI4SRJK6HIJH22HFIHZVVELRQ-East-US/providers/microsoft.backupbvtd2/BackupVault/trinadhVault' -EventName Backup  -EventSource Administrative -Level Error -OperationName 'Microsoft.Backup/backupVault/Backup' -ResourceProvider Microsoft.Backup -Status Failed  -SubStatus Failed -RuleType Event -Location eastus -ResourceGroup RecoveryServices-DP2RCXUGWS3MLJF4LKPI3A3OMJ2DI4SRJK6HIJH22HFIHZVVELRQ-East-US -Name Backup-Failed -Description 'Backup failed for one of the VMs in vault trinadhkVault' -CustomEmails 'contoso@microsoft.com' -SendToServiceOwners
```

**ResourceId** : vous pouvez obtenir cela à partir de la fenêtre contextuelle Journaux des opérations, comme indiqué dans la section ci-dessus. L’élément ResourceUri de la fenêtre contextuelle de détails d’une opération est l’ID de ressource à fournir à cet applet de commande.

**EventName** : pour les alertes de sauvegarde de machines virtuelles IaaS, les valeurs prises en charge sont Register,Unregister,ConfigureProtection,Backup,Restore,StopProtection,DeleteBackupData,CreateProtectionPolicy,DeleteProtectionPolicy,UpdateProtectionPolicy.

**Level** : les valeurs prises en charge sont Informational, Error. Pour les alertes sur l’action ayant échoué, utilisez Error et les alertes sur les travaux terminés, utilisez Informational.

**OperationName** : il se présente sous le format « Microsoft.Backup/backupvault/<EventName> » où EventName est décrit ci-dessus.

**État** : les valeurs prises en charge sont Démarré, Réussi et Échec. Il est conseillé de conserver le niveau Informational pour l’état Succeeded.

**SubStatus** : identique à l’état des opérations de sauvegarde.

**RuleType** : conservez la valeur *Event*, car les alertes de sauvegarde sont basées sur les événements.

**ResourceGroup** : groupe de ressources auquel appartient la ressource sur laquelle l’opération est déclenchée. Vous pouvez l’obtenir à partir de la valeur ResourceId. La valeur entre les champs */resourceGroups/* et */providers/* dans la valeur ResourceId valeur correspond à la valeur de GroupeResource.

**Nom** : nom de la règle d’alerte.

**Description** : description de la règle d’alerte.

**CustomEmails** : spécifiez l’adresse de messagerie personnalisée à laquelle vous voulez envoyer des notifications d’alerte.

**SendToServiceOwners** : cette option envoie des notifications d’alerte à tous les administrateurs et coadministrateurs de l’abonnement.

Un message d’alerte exemple ressemble à ceci :

Exemple d’en-tête :

![En-tête d’alerte](./media/backup-azure-manage-vms/alert-header.png)

Exemple de corps de message d’alerte :

![Corps de l’alerte](./media/backup-azure-manage-vms/alert-body.png)

### Limitations sur les alertes
Les alertes basées sur des événements sont soumises aux limitations suivantes :

1. Des alertes sont déclenchées sur toutes les machines virtuelles dans le coffre de sauvegarde. Vous ne pouvez pas le personnaliser pour obtenir des alertes pour un ensemble spécifique de machines virtuelles à l’intérieur d’un coffre de sauvegarde.
2. Les alertes sont automatiquement résolues si aucun événement d’alerte déclenché dans la durée d’alerte suivante. Utilisez le paramètre *WindowSize* dans l’applet de commande Add-AlertRule pour définir la durée de déclenchement de l’alerte.

## Étapes suivantes

- [Restauration de machines virtuelles Azure](backup-azure-restore-vms.md)

<!---HONumber=AcomDC_0128_2016-->