<properties
	pageTitle="Implémentation de la synchronisation de mot de passe avec Azure AD Connect Sync | Microsoft Azure"
	description="Fournit des informations sur le fonctionnement de la synchronisation de mot de passe et explique comment l’activer."
	services="active-directory"
	documentationCenter=""
	authors="markusvi"
	manager="stevenpo"
	editor=""/>
<tags
	ms.service="active-directory"
	ms.workload="identity"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="get-started-article"
	ms.date="03/22/2016"
	ms.author="markusvi;andkjell"/>


# Implémentation de la synchronisation de mot de passe avec Azure AD Connect Sync

Avec la synchronisation de mot de passe, vous permettez à vos utilisateurs d’utiliser le mot de passe qu’ils utilisent pour se connecter à votre annuaire Active Directory local pour se connecter à Azure Active Directory.

L’objectif de cette rubrique est de vous fournir les informations nécessaires pour comprendre le fonctionnement de la synchronisation de mot de passe, comment l’activer et comment résoudre les problèmes liés dans votre environnement.

## Qu'est-ce que la synchronisation de mot de passe ?

La synchronisation de mot de passe est une fonctionnalité des services de synchronisation d’Azure Active Directory Connect (Azure AD Connect Sync) qui synchronise les mots de passe des utilisateurs de votre annuaire Active Directory local vers Azure Active Directory (Azure AD). Cette fonctionnalité permet à vos utilisateurs de se connecter à leurs services Azure Active Directory (comme Office 365, Microsoft Intune et CRM Online) à l’aide du mot de passe qu’ils utilisent pour se connecter à votre réseau local.

> [AZURE.NOTE] Pour plus d'informations sur les services de domaine Active Directory qui sont configurés pour la synchronisation de mot de passe et FIPS, consultez [Synchronisation de mot de passe et FIPS](#password-synchronization-and-fips).

### Disponibilité de la synchronisation de mot de passe

Tout client Azure Active Directory est autorisé à exécuter la synchronisation de mot de passe. Pour plus d’informations sur la compatibilité de la synchronisation de mot de passe et d’autres fonctionnalités telles que l’authentification fédérée, voir ci-dessous.

## Fonctionnement de la synchronisation de mot de passe

La synchronisation de mot de passe est une extension de la fonctionnalité de synchronisation d’annuaire implémentée par Azure AD Connect Sync. Ainsi, elle nécessite la configuration de la synchronisation d’annuaire entre votre annuaire local et votre annuaire Azure Active Directory.

Les services de domaine Active Directory stockent les mots de passe sous forme de valeur de hachage du mot de passe réel de l’utilisateur. Vous ne pouvez pas utiliser le hachage de mot de passe pour vous connecter à votre réseau local. De par sa conception, vous ne pouvez pas non plus l’inverser pour accéder au mot de passe en texte brut de l’utilisateur. Pour synchroniser un mot de passe, Azure AD Connect Sync extrait le hachage du mot de passe de l’utilisateur dans l’annuaire Active Directory local. Un traitement de sécurité supplémentaire est appliqué au hachage du mot de passe avant sa synchronisation avec le service d’authentification Azure Active Directory. Le flux de données du processus de synchronisation de mot de passe est similaire à la synchronisation des données utilisateur telles que le nom d’affichage ou les adresses de messagerie.

Les mots de passe sont synchronisés plus fréquemment que la fenêtre de synchronisation d’annuaire standard pour d’autres attributs. Les mots de passe sont synchronisés pour chaque utilisateur, généralement par ordre chronologique. Quand un mot de passe est synchronisé de l’annuaire Active Directory local vers le cloud, le mot de passe cloud existant est remplacé.

Quand vous activez pour la première fois la fonctionnalité de synchronisation de mot de passe, elle effectue une synchronisation initiale des mots de passe de tous les utilisateurs dans la portée de votre annuaire Active Directory local vers Azure Active Directory. Vous ne pouvez pas définir explicitement l’ensemble des utilisateurs dont les mots de passe seront synchronisés vers le cloud. Par la suite, quand un mot de passe a été modifié par un utilisateur local, la fonctionnalité de synchronisation de mot de passe détecte et synchronise le mot de passe modifié, le plus souvent en quelques minutes. La fonctionnalité de synchronisation de mot de passe tente automatiquement de réeffectuer une synchronisation de mot de passe utilisateur ayant échoué. Si une erreur se produit lors d’une tentative de synchronisation de mot de passe, l’erreur est enregistrée dans l’Observateur d’événements.

La synchronisation d’un mot de passe n’a aucun impact sur les utilisateurs actuellement connectés. Si un utilisateur connecté à un service cloud modifie également le mot de passe local, la session de service cloud se poursuit sans interruption. Toutefois, dès que le service cloud exige de l’utilisateur qu’il se réauthentifie, le nouveau mot de passe doit être fourni. À ce stade, l’utilisateur doit fournir le nouveau mot de passe (celui qui a récemment été synchronisé de l’annuaire Active Directory local vers le cloud).

> [AZURE.NOTE] La synchronisation de mot de passe est uniquement prise en charge pour l'utilisateur de type d'objet dans Active Directory. Elle n'est pas prise en charge pour le type d'objet iNetOrgPerson.

### Fonctionnement de la synchronisation de mot de passe avec les services de domaine Azure AD.

Si vous activez ce service dans Azure AD, l'option de synchronisation de mot de passe est requise pour obtenir une authentification unique. Lorsque ce service est activé, le comportement de synchronisation de mot de passe est modifié et les hachages de mots de passe sont également synchronisés en l'état depuis votre Active Directory local vers les services de domaine Azure AD. La fonctionnalité est semblable à l'outil ADMT (Active Directory Migration Tool) et permet aux services de domaine Azure AD d'être en mesure d'authentifier l'utilisateur avec toutes les méthodes disponibles dans l'Active Directory local.

### Sécurité

Lors de la synchronisation des mots de passe, la version en texte brut du mot de passe d’un utilisateur n’est exposée ni à la fonctionnalité de synchronisation de mot de passe, ni à Azure AD ou l’un des services associés.

En outre, il n’existe aucune spécification obligeant l’annuaire Active Directory local à stocker le mot de passe dans un format chiffré réversible. Un condensé du hachage de mot de passe Active Directory est utilisé pour la transmission entre l’annuaire Active Directory local et Azure Active Directory. Vous ne pouvez pas utiliser le condensé du hachage de mot de passe pour accéder aux ressources dans l’environnement local du client.

### Remarques sur les stratégies de mot de passe

Deux types de stratégies de mot de passe sont affectés par l’activation de la synchronisation de mot de passe :

1. la stratégie de complexité de mot de passe ;
2. la stratégie d’expiration de mot de passe.

**Stratégie de complexité de mot de passe**

Quand vous activez la synchronisation de mot de passe, les stratégies de complexité de mot de passe configurées dans l’annuaire Active Directory local remplacent les stratégies de complexité qui peuvent être définies dans le cloud pour les utilisateurs synchronisés. Cela signifie que tout mot de passe valide dans l’environnement Active Directory local du client peut être utilisé pour accéder aux services Azure AD.

> [AZURE.NOTE] Les mots de passe des utilisateurs créés directement dans le cloud sont toujours soumis aux stratégies de mot de passe définies dans le cloud.

**Stratégie d'expiration de mot de passe**

Si un utilisateur figure dans la portée de la synchronisation de mot de passe, le mot de passe de compte cloud a la valeur « *Ne jamais expirer* ». Cela signifie que le mot de passe d’un utilisateur peut expirer dans l’environnement local, mais que l’utilisateur peut continuer à se connecter aux services cloud à l’aide du nouveau mot de passe après le prochain cycle de synchronisation des mots de passe.

Le mot de passe cloud sera mis à jour la prochaine fois que l’utilisateur modifiera le mot de passe dans l’environnement local.

### Remplacement des mots de passe synchronisés

Un administrateur peut réinitialiser manuellement le mot de passe d’un utilisateur à l’aide d’Azure Active Directory PowerShell.

Dans ce cas, le nouveau mot de passe remplace le mot de passe synchronisé de l’utilisateur et toutes les stratégies de mot de passe définies dans le cloud s’appliquent au nouveau mot de passe.

Si l’utilisateur modifie de nouveau le mot de passe local, le nouveau mot de passe est synchronisé avec le cloud et il remplace le mot de passe mis à jour manuellement.


## Activation de la synchronisation de mot de passe

Pour activer la synchronisation de mot de passe, deux options s’offrent à vous :

- Si vous utilisez la configuration rapide lorsque vous installez Azure AD Connect, la synchronisation de mot de passe est activée par défaut.

- Si vous utilisez des paramètres personnalisés lors de l’installation d’Azure AD Connect, vous activez la synchronisation de mot de passe sur la page de connexion utilisateur.

<br> ![Activation de la synchronisation de mot de passe](./media/active-directory-aadconnectsync-implement-password-synchronization/usersignin.png) <br>

Si vous choisissez d’utiliser **Fédération avec AD FS**, vous pouvez éventuellement activer la synchronisation de mot de passe comme backup en cas d’échec de votre infrastructure AD FS. Vous pouvez également l'activer si vous prévoyez d'utiliser les Services de domaine Azure AD.

### Synchronisation du mot de passe et FIPS

Si votre serveur a été verrouillé en fonction de la norme FIPS (Federal Information Processing Standard), MD5 a été désactivé. Pour activer cette option pour la synchronisation de mot de passe, ajoutez la clé enforceFIPSPolicy dans miiserver.exe.config dans C:\\Program Files\\Azure AD Sync\\Bin.

```
<configuration>
    <runtime>
        <enforceFIPSPolicy enabled="false"/>
    </runtime>
</configuration>
```

Vous trouverez le nœud de configuration/d'exécution à la fin du fichier de configuration.

Pour plus d'informations sur la sécurité et FIPS, consultez [Synchronisation, cryptage et conformité à la norme FIPS du mot de passe AAD](http://blogs.technet.com/b/ad/archive/2014/06/28/aad-password-sync-encryption-and-and-fips-compliance.aspx)


## Dépannage de la synchronisation du mot de passe

**Pour résoudre les problèmes de synchronisation de mot de passe, procédez comme suit :**

1. Ouvrez **Synchronization Service Manager**.

2. Cliquez sur **Connecteurs**

3. Sélectionnez le connecteur Active Directory dans lequel se trouve l’utilisateur

4. Sélectionnez **Espace de connecteur de recherche**

5. Localisez l’utilisateur que vous recherchez.

6. Sélectionnez l’onglet **lignage** et assurez-vous qu’au moins une règle de synchronisation affiche pour la **Synchronisation de mot de passe** la valeur **True**. Dans la configuration par défaut, le nom de la règle de synchronisation est **In from AD - User AccountEnabled**.

    ![Informations de lignage sur un utilisateur](./media/active-directory-aadconnectsync-implement-password-synchronization/cspasswordsync.png)

7. Vous devez également [suivre l’utilisateur](active-directory-aadconnectsync-service-manager-ui-connectors.md#follow-an-object-and-its-data-through-the-system) dans le métaverse jusqu’à l’espace de connecteur Azure AD et vous assurer qu’il existe également une règle sortante dont l’option **Synchronisation de mot de passe** est définie sur **True**. Dans la configuration par défaut, le nom de la règle de synchronisation est **Out to AAD - User Join**.

    ![Propriétés d’espace de connecteur d’un utilisateur](./media/active-directory-aadconnectsync-implement-password-synchronization/cspasswordsync2.png)

8. Pour afficher les détails de synchronisation de mot de passe, cliquez sur le bouton **Journal...**.<br> La page affichera alors une vue de l’historique de l’état de synchronisation de mot de passe de l’utilisateur pour la semaine passée.

    ![Détails d’un journal objet](./media/active-directory-aadconnectsync-implement-password-synchronization/csobjectlog.png)

La colonne d'état peut avoir les valeurs suivantes et également indiquer le problème et la raison pour laquelle un mot de passe n'est pas synchronisé.

| Statut | Description |
| ---- | ----- |
| Succès | Le mot de passe a été correctement synchronisé. |
| FilteredByTarget | Le mot de passe est défini sur **L'utilisateur doit changer de mot de passe à la prochaine connexion**. Mot de passe n'a pas été synchronisé. |
| NoTargetConnection | Aucun objet dans le métaverse ou dans l'espace de connecteur Azure AD. |
| SourceConnectorNotPresent | Aucun objet trouvé dans l'espace de connecteur Active Directory local. |
| TargetNotExportedToDirectory | L'objet dans l'espace de connecteur Azure AD n'a pas encore été exporté. |
| MigratedCheckDetailsForMoreInfo | L'entrée de journal a été créée avant la version 1.0.9125.0 et est affichée dans son état hérité. |


## Déclenchement d’une synchronisation complète de tous les mots de passe

Dans de nombreux cas, il n’est pas nécessaire de forcer une synchronisation complète de tous les mots de passe.<br> Toutefois, si vous avez besoin de le faire, vous pouvez utiliser le script suivant :

    $adConnector = "<CASE SENSITIVE AD CONNECTOR NAME>"
    $aadConnector = "<CASE SENSITIVE AAD CONNECTOR NAME>"
    Import-Module adsync
    $c = Get-ADSyncConnector -Name $adConnector
    $p = New-Object Microsoft.IdentityManagement.PowerShell.ObjectModel.ConfigurationParameter “Microsoft.Synchronize.ForceFullPasswordSync”, String, ConnectorGlobal, $null, $null, $null
    $p.Value = 1
    $c.GlobalParameters.Remove($p.Name)
    $c.GlobalParameters.Add($p)
    $c = Add-ADSyncConnector -Connector $c
    Set-ADSyncAADPasswordSyncConfiguration -SourceConnector $adConnector -TargetConnector $aadConnector -Enable $false
    Set-ADSyncAADPasswordSyncConfiguration -SourceConnector $adConnector -TargetConnector $aadConnector -Enable $true




## Étapes suivantes

* [Azure AD Connect Sync : personnalisation des options de synchronisation](active-directory-aadconnectsync-whatis.md)
* [Intégration de vos identités locales avec Azure Active Directory](active-directory-aadconnect.md)

<!---HONumber=AcomDC_0330_2016-->