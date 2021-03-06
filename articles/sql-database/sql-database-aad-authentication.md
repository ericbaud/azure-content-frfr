<properties
   pageTitle="Connexion à la base de données SQL en utilisant l’authentification Azure Active Directory | Microsoft Azure"
   description="Apprenez comment vous connecter à la base de données SQL en utilisant l’authentification Azure Active Directory"
   services="sql-database"
   documentationCenter=""
   authors="BYHAM"
   manager="jeffreyg"
   editor="jeffreyg"
   tags=""/>

<tags
   ms.service="sql-database"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="data-management"
   ms.date="03/09/2016"
   ms.author="rick.byham@microsoft.com"/>

# Connexion à la base de données SQL avec l’authentification Azure Active Directory

L’authentification Azure Active Directory est un mécanisme servant à se connecter à la base de données SQL Microsoft Azure à l’aide d’identités dans Azure Active Directory (Azure AD). Avec l’authentification Azure Active Directory, vous pouvez gérer de manière centralisée les identités des utilisateurs de base de données et d’autres services Microsoft dans un emplacement centralisé. La gestion centralisée des ID fournissent un emplacement unique pour gérer les utilisateurs de base de données SQL et simplifient la gestion des autorisations. Les avantages suivants sont inclus :

- Il fournit une alternative à l’authentification SQL Server.
- Permet de bloquer la prolifération des identités utilisateur sur plusieurs serveurs de base de données.
- Permet la rotation de mot de passe à un emplacement unique
- Les clients peuvent gérer les autorisations de base de données à l’aide de groupes (AAD) externes.
- Il peut éliminer le stockage des mots de passe en activant l’authentification intégrée Windows et les autres formes d’authentification prises en charge par Azure Active Directory.
- L’authentification Azure Active Directory utilise les utilisateurs de base de données à relation contenant-contenu pour authentifier les identités au niveau de la base de données.

> [AZURE.IMPORTANT] L’authentification Azure Active Directory est une fonctionnalité préliminaire et est soumise aux conditions du contrat de licence (par exemple, l’accord Entreprise, l’accord Microsoft Azure ou le contrat d’abonnement Microsoft Online), ainsi qu’à toutes les [Conditions d’utilisation supplémentaires de la version préliminaire de Microsoft Azure applicable](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Les étapes de configuration incluent les procédures suivantes pour configurer et utiliser l’authentification Azure Active Directory.

1. Créer et renseigner un répertoire Azure Active Directory
2. Vérifier que votre base de données se trouve bien dans Azure SQL Database V12
3. Facultatif : associer ou modifier le répertoire actif actuellement associé à votre abonnement Azure
4. Créer un administrateur d’Azure Active Directory pour le serveur SQL Azure
5. Configurer vos ordinateurs clients
6. Créer des utilisateurs de base de données à relation contenant-contenu dans votre base de données mappés sur les identités Azure AD
7. Vous connecter à votre base de données en utilisant les identités Azure AD


## Architecture d’approbation

Le diagramme de niveau élevé suivant résume l’architecture de solution de l’utilisation de l’authentification Azure AD avec la base de données SQL Azure. Les flèches indiquent les voies de communication.

![diagramme autorisation aad][1]

Le diagramme suivant indique la fédération, l’approbation et les relations d’hébergement qui autorisent un client à se connecter à une base de données en soumettant un jeton authentifié par un Azure AD, et approuvé par la base de données. Il est important de comprendre que l’accès à une base de données à l’aide de l’authentification Azure AD exige que l’abonnement d’hébergement soit associé à Azure Active Directory.

![relation abonnement][2]

## Structure de l’administrateur

En cas d’utilisation de l’authentification Azure AD, il existe deux comptes administrateur pour le serveur de base de données SQL ; l’administrateur de SQL Server d’origine et l’administrateur Azure AD. Seul l’administrateur basé sur un compte Azure AD peut créer le premier utilisateur de la base de données Azure AD à relation contenant-contenu dans une base de données utilisateur. La connexion d’administrateur Azure AD peut être un utilisateur Azure AD ou un groupe Azure AD. Lorsque l’administrateur est un compte de groupe, il peut être utilisé par n’importe quel membre du groupe, ce qui permet à plusieurs administrateurs Azure AD pour l’instance de SQL Server. L’utilisation d’un compte de groupe en tant qu’administrateur facilite la gestion en vous permettant d’ajouter et de supprimer des membres du groupe dans Azure AD sans modifier les utilisateurs ou les autorisations de base de données SQL. Seul un administrateur Azure AD (utilisateur ou groupe) peut être configuré à tout moment.

![structure admin][3]

## Autorisations

Pour créer de nouveaux utilisateurs, vous devez disposer de l’autorisation **MODIFIER UN UTILISATEUR** dans la base de données. L’autorisation **MODIFIER UN UTILISATEUR** peut être octroyée à un utilisateur de base de données. L’autorisation **MODIFIER UN UTILISATEUR** est également détenue par les comptes d’administrateur de serveur et les utilisateurs de base de données avec les autorisations **CONTRÔLE DE BASE DE DONNÉES** ou **MODIFIER UNE BASE DE DONNÉES** pour cette base de données et par les membres du rôle de base de données **db\_owner**.

Pour créer un utilisateur de base de données à relation contenant-contenu dans la base de données SQL Azure, vous devez vous connecter à la base de données à l’aide d’une identité Azure AD. Pour créer le premier utilisateur de la base de données à relation contenant-contenu, vous devez vous connecter à la base de données à l’aide d’un administrateur Azure AD (le propriétaire de la base de données). Cette opération est illustrée dans les étapes 4 et 5 ci-dessous.

## Limitations et fonctionnalités azure AD

Les membres suivants de Microsoft Azure Active Directory peuvent être configurés dans le serveur Azure SQL Server :
- Membre natif : un membre créé dans le domaine géré ou le domaine client de Microsoft Azure AD. Pour plus d’informations, consultez [Ajout de votre propre nom de domaine à Azure AD]( https://azure.microsoft.com/documentation/articles/active-directory-add-domain/).
- Membre de domaine fédéré : un membre créé dans Azure AD avec un domaine fédéré. Pour plus d’informations, consultez [Microsoft Azure prend désormais en charge la fédération avec Windows Server Active Directory](https://azure.microsoft.com/blog/2012/11/28/windows-azure-now-supports-federation-with-windows-server-active-directory/).
- Membres importés à partir d’autres répertoires Azure Active Directory qui sont des membres natifs ou de domaine fédéré.
- Groupes Active Directory créés en tant que groupes de sécurité.

Les comptes Microsoft (par exemple outlook.com, hotmail.com, live.com) ou d’autres comptes d’invité (par exemple gmail.com, yahoo.com) ne sont pas pris en charge. Si vous pouvez vous connecter à [https://login.live.com](https://login.live.com) à l’aide du compte et du mot de passe, c’est que vous utilisez un compte Microsoft qui n’est pas pris en charge pour l’authentification Azure AD pour la base de données SQL Azure.

### Considérations supplémentaires

- Pour améliorer la facilité de gestion, nous vous conseillons de mettre en service un groupe Azure Active Directory dédié en tant qu’administrateur.
- Un seul utilisateur administrateur Azure AD (utilisateur ou groupe) peut être configuré pour un serveur Azure SQL Server à tout moment.
- Seul un administrateur d’Azure Active Directory peut se connecter initialement au serveur SQL Azure en utilisant un compte Azure Active Directory. L’administrateur Active Directory peut configurer les utilisateurs de base de données Azure Active Directory suivants.
- Nous vous conseillons de définir l’expiration du délai de connexion à 30 secondes.
- Certains outils tels que Business Intelligence et Excel ne sont pas pris en charge.
- L’authentification Azure Active Directory prend uniquement en charge **Fournisseur de données .NET Framework pour SqlServer** (au moins la version .NET Framework 4.6). Par conséquent, Management Studio (disponible avec SQL Server 2016) et les applications de couche Données (DAC et .bacpac) peuvent se connecter, mais **sqlcmd.exe** ne peut pas se connecter, car **sqlcmd** utilise le fournisseur ODBC.
- Authentification à deux facteurs ou autres formes d’authentification interactive non prise en charge.


## 1\. Créer et renseigner un répertoire Azure Active Directory

Créer un annuaire Azure Active Directory et le renseigner avec les utilisateurs et les groupes. notamment :

- Créer le domaine Azure AD géré initial.
- Fédérer les Services de domaine Active Directory local avec Azure Active Directory.
- À l’aide de l’outil **AD FS**, dans la section **Service**, **Points de terminaison**, activez **WS-Trust 1.3** pour le chemin d’URL **/adfs/services/trust/13/windowstransport**.

Pour plus d’informations, consultez [Ajouter votre propre nom de domaine à Azure AD]( https://azure.microsoft.com/documentation/articles/active-directory-add-domain/), [Microsoft Azure prend désormais en charge la fédération avec Windows Server Active Directory](https://azure.microsoft.com/blog/2012/11/28/windows-azure-now-supports-federation-with-windows-server-active-directory/), [Administration de votre annuaire Azure AD](https://msdn.microsoft.com/library/azure/hh967611.aspx), et [Gérer Azure Active Directory à l’aide de Windows PowerShell](https://msdn.microsoft.com/library/azure/jj151815.aspx).

## 2\. Vérifier que votre base de données se trouve bien dans Azure SQL Database V12

L’authentification Azure Active Directory prend en charge les bases de données SQL V12 les plus récentes. Pour plus d’informations sur SQL Database V12 et pour savoir si ce service est disponible dans votre région, consultez [Nouveautés dans la dernière mise à jour de SQL Database V12](sql-database-v12-whats-new.md).

Si vous avez déjà une base de données, vérifiez qu’elle est hébergée dans la base de données SQL V12 en se connectant à la base de données (par exemple, en utilisant SQL Server Management Studio) et en exécutant `SELECT @@VERSION;` Le résultat attendu pour une base de données SQL V12 est au moins **Microsoft SQL Azure (RTM) - 12.0**.

Si votre base de données n’est pas hébergée dans SQL Database V12, consultez [Planifier et préparer la mise à niveau vers SQL Database V12](sql-database-v12-plan-prepare-upgrade.md), puis visitez le portail Azure Classic pour migrer la base de données vers SQL Database V12.

Vous pouvez également créer une base de données dans SQL Database V12 en exécutant les opérations répertoriées dans [Créer votre première base de données SQL Azure](sql-database-get-started.md). **Conseil** : lisez l’étape suivante avant de sélectionner un abonnement pour votre nouvelle base de données.

## 3\. Facultatif : associer ou modifier le répertoire actif actuellement associé à votre abonnement Azure

Pour associer votre base de données à l’annuaire Azure AD de votre organisation, faites de l’annuaire un annulaire approuvé pour l’abonnement Azure qui héberge la base de données. Pour plus d’informations, consultez la page [Comment sont associés les abonnements Azure et Azure AD](https://msdn.microsoft.com/library/azure/dn629581.aspx).

**Additional information :** chaque abonnement Azure dispose d’une relation d’approbation avec une instance Azure AD. Cela signifie qu'il approuve ce répertoire pour authentifier les utilisateurs, les services et les appareils. Plusieurs abonnements peuvent approuver le même répertoire, mais un abonnement n’approuve qu’un seul répertoire. Vous pouvez découvrir quel répertoire est approuvé par votre abonnement dans l’onglet **Paramètres**, à l’adresse [https://manage.windowsazure.com/](https://manage.windowsazure.com/). Cette relation de confiance qu’un abonnement possède avec un répertoire est contraire à celle établie entre un abonnement et toutes les autres ressources Azure (sites Web, bases de données, etc.), qui se rapprochent plus des ressources enfants d'un abonnement. Lorsqu’un abonnement expire, les autres ressources associées à l'abonnement deviennent également inaccessibles. Mais le répertoire reste dans Azure, et vous pouvez associer un autre abonnement à ce répertoire et continuer à gérer les utilisateurs du répertoire. Pour plus d’informations sur les ressources, consultez [Comprendre l’accès aux ressources dans Azure](https://msdn.microsoft.com/library/azure/dn584083.aspx).

Les procédures suivantes fournissent des instructions étape par étape sur la façon de comment modifier l’annuaire associé à un abonnement donné.

1. Connectez-vous à votre [portail Azure Classic](https://manage.windowsazure.com/) à l’aide d’un administrateur d’abonnement Azure.
2. Dans la bannière de gauche, sélectionnez **PARAMÈTRES**.
3. Vos abonnements s’affichent dans l’écran Paramètres. Si l’abonnement souhaité n’apparaît pas, cliquez sur **abonnements** en haut, développez la liste déroulante le **FILTRER PAR ANNUAIRE** et sélectionnez le répertoire qui contient vos abonnements, puis cliquez sur **APPLIQUER**.

	![sélectionner l'abonnement][4]
4. Dans la zone de **paramètres**, cliquez sur votre abonnement, puis sur **MODIFIER L’ANNUAIRE** en bas de la page.

	![portail-paramètres-ad][5]
5. Dans la zone **MODIFIER L’ANNUAIRE**, sélectionnez Azure Active Directory associé à votre serveur SQL Server, puis cliquez sur la flèche pour le suivant.

	![edit-directory-select][6]
6. Dans la boîte de dialogue **Confirmer** le mappage d’annuaire, confirmez que « **Tous les coadministrateurs seront supprimés.** »

	![edit-directory-confirm][7]
7. Cliquez sur la coche pour recharger le portail.

> [AZURE.NOTE] Lorsque vous modifiez l’annuaire, l’accès à tous les coadministrateurs, les utilisateurs et les groupes Azure AD, et les utilisateurs de ressources reposant sur le répertoire des ressources est supprimé, ils n’ont plus accès à cet abonnement ou à ses ressources. Vous êtes le seul, en tant qu’administrateur du service, à pouvoir configurer aux serveurs principaux en fonction du nouveau répertoire. Cette modification peut prendre beaucoup de temps pour se propager à toutes les ressources. La modification du répertoire changera également l’administrateur Azure AD pour la base de données SQL et désactiver l’accès à la base de données SQL pour tous les utilisateurs Azure AD existants. L’administrateur Azure AD doit être réinitialisé (commet décrit ci-dessous) et de nouveaux utilisateurs Azure Active Directory doivent être créés.

## 4\. Créer un administrateur d’Azure Active Directory pour le serveur SQL Azure

Chaque serveur SQL Azure démarre avec un compte d’administrateur de serveur unique, qui est l’administrateur de l’ensemble du serveur SQL Azure. Un deuxième administrateur de serveur doit être créé. Il s’agit d’un compte Azure AD. Cet utilisateur principal est créé en tant qu’utilisateur de base de données à relation contenant-contenu dans la base de données master. En tant qu’administrateurs, les comptes d’administrateur de serveur sont des membres du rôle **db\_owner** de chaque base de données utilisateur, puis saisissez chaque base de données utilisateur en tant utilisateur **dbo**. Pour plus d’informations sur les comptes d’administrateur de serveur, consultez les sections [Gérer les bases de données et des connexions dans la base de données SQL Azure](sql-database-manage-logins.md) et **Connexions et utilisateurs** de [Consignes de sécurité et limites de base de données SQL Azure](sql-database-security-guidelines.md).

Lorsque vous utilisez Azure Active Directory avec la géo-réplication, le compte administrateur de Microsoft Azure Active Directory doit être configuré pour le serveur principal et le serveur secondaire. Si un serveur ne dispose pas d’un administrateur Azure Active Directory, les utilisateurs Azure Active Directory recevront un message d’erreur « Impossible de se connecter au serveur ».

> [AZURE.NOTE] Les utilisateurs qui ne sont pas basés sur un compte Azure AD (y compris le compte d’administrateur de serveur SQL Azure) ne peuvent pas créer des utilisateurs Azure AD, en car ils n’ont pas l’autorisation de valider les utilisateurs de base de données proposée avec Azure AD.

### Approvisionner un administrateur Active Directory d’Azure pour votre serveur de SQL Azure en utilisant le portail Azure

1. Dans le [Portail Azure](https://portal.azure.com/), dans le coin supérieur droit, cliquez sur votre connexion pour développer une liste déroulante de répertoires Active Directories. Choisissez l’annuaire Active Directory approprié en tant qu’Azure AD par défaut. Cette étape lie l’association de l’abonnement avec Active Directory et la base de données SQL Azure, ce qui garantit que c’est le même abonnement qui est utilisé à la fois pour Azure AD et pour SQL Server.

	![choose-ad][8]
2. Dans la bannière de gauche, sélectionnez **serveurs SQL**, sélectionnez votre **serveur SQL**, puis, en haut du panneau **Serveur SQL**, cliquez sur **paramètres**.

	![paramètre ad][9]
3. Dans le panneau **Paramètres**, cliquez sur **administrateur Active directory (aperçu)**, et acceptez la clause Preview.
4. Dans le panneau **Administrateur active directory (aperçu)**, cliquez pour examiner, puis cliquez sur **OK** pour accepter les termes de l’aperçu.
5. Dans le panneau **Administrateur Active directory (aperçu)**, cliquez sur **Administrateur Active directory**, puis en haut de la page, cliquez sur **Définir admin**.
6. Dans le panneau **Ajouter admin**, recherchez un utilisateur, sélectionnez l’utilisateur ou le groupe en tant qu’administrateur, puis cliquez sur **Sélectionner**. Le panneau d’administration Active Directory affiche tous les membres et les groupes présents dans Active Directory. Les utilisateurs ou les groupes grisés ne peuvent être sélectionnés, car ils ne sont pas pris en charge en tant qu’administrateurs Azure AD. (Voir la liste des administrateurs pris en charge dans **Fonctionnalités et limitations Azure AD** ci-dessus.) Le contrôle d'accès basé sur les rôles (RBAC) s'applique uniquement au portail et n'est pas propagé vers SQL Server.
7. En haut du panneau **Administrateur Active Directory**, cliquez sur **ENREGISTRER**. ![cliquer sur admin][10]

	La procédure de changement de l’administrateur peut prendre plusieurs minutes. Le nouvel administrateur apparaîtra dans la zone **Administrateur Active Directory**.

> [AZURE.NOTE] Lors de la configuration de l'administrateur Azure AD, le nouveau nom d'administrateur (utilisateur ou groupe) ne peut pas déjà être présent dans la base de données master en tant que connexion d'authentification SQL Server. Si tel est le cas, la configuration de l’administrateur d’Azure AD échoue, annulant sa création et indiquant que cet administrateur (ce nom) existe déjà. Dans la mesure où une connexion d'authentification SQL Server ne fait pas partie d’Azure AD, tout effort pour se connecter au serveur à l'aide de l'authentification Azure AD échouera.

Pour supprimer un administrateur, en haut du panneau **Administrateur Active Directory**, cliquez sur **Supprimer l’administrateur**.

### Configurez un administrateur Azure AD pour le serveur SQL Azure à l’aide de PowerShell



Pour exécuter les applets de commande PowerShell, Azure PowerShell doit être installé et en cours d’exécution. Pour plus de détails, consultez la rubrique [Installation et configuration d’Azure PowerShell](../powershell-install-configure.md).

Pour configurer un administrateur Azure AD, vous devez exécuter les commandes Azure PowerShell suivantes :

- Add-AzureRmAccount
- Select-AzureRmSubscription


Applets de commande utilisées pour configurer et gérer Azure AD admin :

| Nom de l’applet de commande | Description |
|---------------------------------------------------|-----------------------------------------------------------------------------------------------------------------|
| [Set-AzureRmSqlServerActiveDirectoryAdministrator](https://msdn.microsoft.com/library/azure/mt603544.aspx) | Approvisionne un administrateur d’Azure Active Directory pour Azure SQL Server (À partir de l’abonnement actuel) |
| [Remove-AzureRmSqlServerActiveDirectoryAdministrator](https://msdn.microsoft.com/library/azure/mt619340.aspx) | Retire un administrateur Azure Active Directory pour le serveur SQL Azure |
| [Get-AzureRmSqlServerActiveDirectoryAdministrator](https://msdn.microsoft.com/library/azure/mt603737.aspx) | Renvoie les informations sur un administrateur Azure Active Directory actuellement configuré pour Azure SQL Server. |

Utilisez la commande PowerShell get-help pour obtenir plus de détails sur chacune de ces commandes, par exemple ``get-help Set-AzureRmSqlServerActiveDirectoryAdministrator``.

Le script suivant configure un groupe d’administrateurs Azure AD nommé **DBA\_Group** (id d’objet `40b79501-b343-44ed-9ce7-da4c8cc7353f`) pour le serveur **demo\_server** d’un groupe de ressources nommé **groupe-23** :

```
Set-AzureRmSqlServerActiveDirectoryAdministrator –ResourceGroupName "Group-23"
–ServerName "demo_server" -DisplayName "DBA_Group"
```

Le paramètre d’entrée **DisplayName** accepte le nom d’affichage Azure AD ou le nom principal de l’utilisateur. Par exemple, ``DisplayName="John Smith"`` et ``DisplayName="johns@contoso.com"``. Pour les groupes Azure AD, seul le nom d’affichage est pris en charge.

> [AZURE.NOTE] La commande Azure PowerShell ```Set-AzureRmSqlServerActiveDirectoryAdministrator``` ne vous empêchera pas d’approvisionner les administrateurs admin Azure AD pour les utilisateurs non pris en charge. Un utilisateur non pris en charge peut être configuré, mais il ne sera pas en mesure de se connecter à une base de données. (Voir la liste des administrateurs pris en charge dans **Fonctionnalités et limitations Azure AD** ci-dessus.)

L'exemple suivant utilise l'**ObjectID** facultatif en option :

```
Set-AzureRmSqlServerActiveDirectoryAdministrator –ResourceGroupName "Group-23"
–ServerName "demo_server" -DisplayName "DBA_Group" -ObjectId "40b79501-b343-44ed-9ce7-da4c8cc7353f"
```

> [AZURE.NOTE] L’**ObjectID** Azure AD est requis lorsque le **DisplayName** n’est pas unique. Pour récupérer les valeurs **ObjectID** et **DisplayName**, utilisez la section Active Directory du portail Azure Classic et affichez les propriétés d’un utilisateur ou d’un groupe.

L’exemple suivant renvoie des informations sur l’administrateur Azure AD admin pour le serveur SQL Azure :

```
Get-AzureRmSqlServerActiveDirectoryAdministrator –ResourceGroupName "Group-23" –ServerName "demo_server" | Format-List
```

L’exemple suivant supprime un administrateur Azure AD :
```
Remove-AzureRmSqlServerActiveDirectoryAdministrator -ResourceGroupName "Group-23" –ServerName "demo_server"
```

## 5\. Configurer vos ordinateurs clients

Sur toutes les machines clientes à partir desquelles vos applications ou les utilisateurs se connectent à la base de données SQL Azure avec des identités Azure AD, vous devez installer les logiciels suivants :

- .NET Framework version 4.6 ou ultérieure de [https://msdn.microsoft.com/library/5a4x27ek.aspx](https://msdn.microsoft.com/library/5a4x27ek.aspx).
- La bibliothèque d’authentification Azure Active Directory pour SQL Server (**ADALSQL. DLL**) est disponible en plusieurs langues (x86 et amd64) à partir du centre de téléchargement de la [Bibliothèque d’authentification Microsoft Active Directory pour Microsoft SQL Server](http://www.microsoft.com/download/details.aspx?id=48742).

### Outils

- L’installation de [SQL Server 2016 Management Studio](https://msdn.microsoft.com/library/mt238290.aspx) ou de [SQL Server Data Tools pour Visual Studio 2015](https://msdn.microsoft.com/library/mt204009.aspx) est conforme à la configuration requise de .NET Framework 4.6.
- SSMS installe la version x86 de **ADALSQL. DLL**. (À ce stade, SSMS ne parvient pas à demander le redémarrage requis après l'installation. Ce problème devrait être résolu dans une prochaine version CTP.)
- SSDT installe la version amd64 de **ADALSQL. DLL**. L’authentification Azure AD n’est prise en charge que partiellement par SSDT.
- La dernière version de Visual Studio de [Téléchargements Visual Studio](https://www.visualstudio.com/downloads/download-visual-studio-vs) respecte la configuration requise de .NET Framework 4.6, mais n'installe pas la version requise amd64 de **ADALSQL.DLL**.

## 6\. Créer des utilisateurs de base de données à relation contenant-contenu dans votre base de données mappés sur les identités Azure AD

### À propos des utilisateurs de base de données à relation contenant-contenu

L’authentification Azure Active Directory nécessite que les utilisateurs de base de données soient créés en tant qu’utilisateurs de base de données à relation contenant-contenu. Un utilisateur de base de données à relation contenant-contenu sur une identité Azure AD est un utilisateur de base de données qui ne dispose pas de connexion dans la base de données master, et qui est mappé sur une identité située dans l’annuaire Azure AD associé à la base de données. L’identité Azure AD peut être un compte d’utilisateur individuel ou un groupe. Pour plus d'informations sur les utilisateurs de base de données à relation contenant-contenu, consultez [Utilisateurs de base de données - Rendre votre base de données portable](https://msdn.microsoft.com/library/ff929188.aspx). Les utilisateurs de base de données (à l’exception des administrateurs) ne peuvent pas être créés à l'aide du portail, et les rôles RBAC ne sont pas propagés à SQL Server.

### Se connecter à la base de données utilisateur à l’aide de SQL Server Management Studio

Pour vérifier que l’administrateur Azure AD est correctement configuré, connectez-vous à la base de données **master** en utilisant un compte d’administrateur Azure AD. Pour configurer un utilisateur de base de données à relation contenant-contenu Azure AD (autre que l’administrateur de serveur propriétaire de la base de données), connectez-vous à la base de données avec une identité Azure AD ayant accès à la base de données.

> [AZURE.IMPORTANT] La prise en charge de l'authentification Azure Active Directory est disponible avec [SQL Server 2016 Management Studio](https://msdn.microsoft.com/library/mt238290.aspx).

#### Connectez-vous à l’aide de l’authentification intégrée à Active Directory

Utilisez cette méthode si vous êtes connecté à Windows avec vos informations d’identification Azure Active Directory à partir d’un domaine fédéré.

1. Démarrez Management Studio et dans la boîte de dialogue **Se connecter au moteur de base de données** (ou **Se connecter à la base de données**), dans la zone **Authentification**, sélectionnez **Authentification intégrée Active Directory**. Aucun mot de passe n’est nécessaire ou ne peut être saisi, car les informations d’identification existantes seront présentées pour la connexion.
2. Cliquez sur le bouton **Options** puis, sur la page **Propriétés de connexion**, dans la zone **Se connecter à la base de données**, saisissez le nom de la base de données utilisateur à laquelle vous souhaitez vous connecter.

#### Connectez-vous à l’aide de l’authentification par mot de passe Active Directory

Utilisez cette méthode lors de la connexion avec un nom principal Azure AD à l’aide du domaine géré d’Azure AD. Vous pouvez également l’utiliser pour un compte fédéré sans accéder au domaine, par exemple lorsque vous travaillez à distance.

Utilisez cette méthode si vous êtes connecté à Windows à l’aide des informations d’identification d’un domaine qui n’est pas fédéré avec Azure, ou lorsque vous utilisez l’authentification Azure AD à l’aide d’Azure AD sur le domaine initial ou le domaine client.

1. Démarrez Management Studio et dans la boîte de dialogue **Se connecter au moteur de base de données** (ou **Se connecter à la base de données**), dans la zone **Authentification**, sélectionnez **Authentification par mot de passe Active Directory**.
2. Dans la zone **Nom d’utilisateur** saisissez votre nom d’utilisateur Azure Active Directory au format **username@domain.com**. Il soit s’agir d’un compte Azure Active Directory ou d’un compte de domaine fédéré avec Azure Active Directory.
3. Dans la zone **Mot de passe**, saisissez votre mot de passe utilisateur pour le compte Azure Active Directory ou le compte de domaine fédéré.
4. Cliquez sur le bouton **Options** puis, sur la page **Propriétés de connexion**, dans la zone **Se connecter à la base de données**, saisissez le nom de la base de données utilisateur à laquelle vous souhaitez vous connecter.


### Créer un utilisateur de base de données à relation contenant-contenu Azure AD dans une base de données utilisateur

Pour créer un utilisateur de base de données à relation contenant-contenu Azure AD (autre que l'administrateur du serveur propriétaire de la base de données), connectez-vous à la base de données avec une identité Azure AD (comme indiqué dans la procédure précédente) en tant qu'utilisateur avec au moins l'autorisation **MODIFIER UN UTILISATEUR**. Utilisez ensuite la syntaxe Transact-SQL suivante :

	CREATE USER Azure_AD_principal_name
	FROM EXTERNAL PROVIDER;


*Azure\_AD\_principal\_name* peut être le nom d’utilisateur principal d’un utilisateur Azure AD ou le nom d’affichage d’un groupe Azure AD.

**Exemples :** pour créer une base de données à relation contenant-contenu représentant un utilisateur de domaine fédéré ou géré Azure AD :

	CREATE USER [bob@contoso.com] FROM EXTERNAL PROVIDER;
	CREATE USER [alice@fabrikam.onmicrosoft.com] FROM EXTERNAL PROVIDER;

Pour créer un utilisateur de base de données à relation contenant-contenu représentant un groupe de domaine Azure AD ou fédéré, définissez le nom complet d’un groupe de sécurité :

	CREATE USER [ICU Nurses] FROM EXTERNAL PROVIDER;


Pour plus d’informations sur la création d’utilisateurs de base de données à relation contenant-contenu basés sur des identités Azure Active Directory, voir [CRÉER UN UTILISATEUR (Transact-SQL)](http://msdn.microsoft.com/library/ms173463.aspx).

Lorsque vous créez un utilisateur de base de données, il reçoit l’autorisation **CONNECT** et peut se connecter à cette base de données en tant que membre du rôle **PUBLIC**. À l'origine, les seules autorisations disponibles pour l'utilisateur sont celles qui sont octroyées au rôle **PUBLIC**, ou les autorisations accordées aux groupes Windows dont ils sont membres. Une fois que vous avez configuré un utilisateur de base de données Azure à relation contenant-contenu, vous pouvez octroyer à cet utilisateur des autorisations supplémentaires, de la même façon que vous accordez l’autorisation à un autre type d’utilisateur. En général, on accorde les autorisations aux rôles de base de données, puis on ajoute des utilisateurs aux rôles. Pour plus d’informations, consultez [Notions de base sur les autorisations de moteur de base de données](http://social.technet.microsoft.com/wiki/contents/articles/4433.database-engine-permission-basics.aspx). Pour plus d'informations sur les rôles de base de données SQL, consultez [Gestion des bases de données et des connexions dans la base de données SQL Azure](sql-database-manage-logins.md). Un utilisateur de domaine fédéré importé dans un domaine de gestion, doit utiliser l’identité de domaine géré.

> [AZURE.NOTE] Les utilisateurs AD Azure sont marqués dans les métadonnées de la base de données avec le type E (EXTERNAL\_USER) et pour les groupes avec le type X (EXTERNAL\_GROUPS). Pour plus d’informations, consultez [sys.database\_principals](https://msdn.microsoft.com/library/ms187328.aspx).


## 7\. Connectez-vous à votre base de données à l’aide d’identités Azure Active Directory

L’authentification Azure Active Directory prend en charge les méthodes suivantes de connexion à une base de données à l’aide d’identités Azure AD :

- À l’aide de l’authentification Windows.
- Avec un nom principal Azure AD et un mot de passe

### 7\.1. Connexion à l’aide de l’authentification intégrée (Windows).

Pour utiliser l’authentification Windows intégrée, l’annuaire Active Directory de votre domaine doit être fédéré avec Azure Active Directory et votre application cliente (ou un service) de connexion à la base de données doit être exécutée sur un ordinateur joint à un domaine avec les informations d’identification du domaine d’un utilisateur.

Pour vous connecter à une base de données à l’aide de l’authentification intégrée et d’une identité Azure AD, le mot clé d’authentification de la chaîne de connexion de base de données doit avoir la valeur Active Directory intégré. L’exemple de code C# suivant utilise ADO .NET.

	string ConnectionString =
	@"Data Source=n9lxnyuzhv.database.windows.net; Authentication=Active Directory Integrated;";
	SqlConnection conn = new SqlConnection(ConnectionString);
	conn.Open();

Notez que le mot clé de la chaîne de connexion ``Integrated Security=True`` n’est pas pris en charge pour la connexion à la base de données SQL Azure.

### 7\.2. La connexion avec un nom principal et un mot de passe Azure AD
Pour vous connecter à une base de données à l’aide de l’authentification intégrée et une identité Azure AD, le mot clé d’authentification doit être défini sur mot de passe Active Directory et la chaîne de connexion doit contenir les mots clés et les valeurs ID/UID et les mots clés de mot de passe/PWD de l’utilisateur. L’exemple de code C# suivant utilise ADO .NET.

	string ConnectionString =
	  @"Data Source=n9lxnyuzhv.database.windows.net; Authentication=Active Directory Password; UID=bob@contoso.onmicrosoft.com; PWD=MyPassWord!";
	SqlConnection conn = new SqlConnection(ConnectionString);
	conn.Open();

Pour obtenir des exemples de code spécifiques associés à l’authentification Azure AD, consultez la section [Blog de sécurité de SQL Server](http://blogs.msdn.com/b/sqlsecurity/) sur MSDN.

## Voir aussi

[Gestion des bases de données et des connexions dans la base de données Azure SQL](sql-database-manage-logins.md)

[Utilisateurs de base de données à relation contenant-contenu](https://msdn.microsoft.com/library/ff929071.aspx)

[CRÉER UN UTILISATEUR (Transact-SQL)](http://msdn.microsoft.com/library/ms173463.aspx)


<!--Image references-->

[1]: ./media/sql-database-aad-authentication/1aad-auth-diagram.png
[2]: ./media/sql-database-aad-authentication/2subscription-relationship.png
[3]: ./media/sql-database-aad-authentication/3admin-structure.png
[4]: ./media/sql-database-aad-authentication/4select-subscription.png
[5]: ./media/sql-database-aad-authentication/5ad-settings-portal.png
[6]: ./media/sql-database-aad-authentication/6edit-directory-select.png
[7]: ./media/sql-database-aad-authentication/7edit-directory-confirm.png
[8]: ./media/sql-database-aad-authentication/8choose-ad.png
[9]: ./media/sql-database-aad-authentication/9ad-settings.png
[10]: ./media/sql-database-aad-authentication/10choose-admin.png

<!----------HONumber=AcomDC_0309_2016-->