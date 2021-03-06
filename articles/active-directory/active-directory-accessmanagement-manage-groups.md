<properties

	pageTitle="Managing security groups in Azure Active Directory | Microsoft Azure"
	description="How to create and manage security groups to manage Azure resource access using Azure Active Directory."
	services="active-directory"
	documentationCenter=""
	authors="curtand"
	manager="stevenpo"
	editor=""/>

<tags
	ms.service="active-directory"
	ms.workload="identity"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="get-started-article"
	ms.date="03/18/2016"
	ms.author="curtand"/>


# Gestion des groupes dans Azure Active Directory

L’une des principales fonctionnalités d’Azure Active Directory (Azure AD) est la possibilité de gérer l’accès aux ressources. Ces ressources peuvent être des objets dans le répertoire, des ressources externes au répertoire, comme les applications SaaS, les services Azure et les sites SharePoint, ou des ressources locales. En outre, le propriétaire d’une ressource peut attribuer l’accès à une ressource à un groupe Azure AD. Cette opération autorise les membres de ce groupe à accéder à la ressource. De son côté, le propriétaire du groupe gère l’appartenance au groupe. Le propriétaire de la ressource délègue au propriétaire du groupe l’autorisation d’affecter des utilisateurs à ses ressources.

## Comment créer un groupe de sécurité ?

**Pour créer un groupe**

Cette tâche peut être effectuée à l’aide du portail des comptes Office 365, du portail des comptes Windows Intune ou du portail Azure, selon les services auxquels votre organisation est abonnée. Pour plus d’informations sur l’utilisation des portails pour gérer Azure Active Directory, consultez [Administration de votre annuaire Azure AD](active-directory-administer.md).

1. Dans le [portail Azure Classic](https://manage.windowsazure.com), sélectionnez **Active Directory**, puis le nom du répertoire de votre organisation.

2. Sélectionnez l’onglet **Groupes**.

3. Sélectionnez **Ajouter un groupe**.

4. Dans la fenêtre **Ajouter un groupe**, spécifiez le nom et la description d’un groupe.


## Comment ajouter ou supprimer des utilisateurs individuels dans un groupe de sécurité ?

**Pour ajouter un utilisateur individuel à un groupe**

1. Dans le [portail Azure Classic](https://manage.windowsazure.com), sélectionnez **Active Directory**, puis le nom du répertoire de votre organisation.

2. Sélectionnez l’onglet **Groupes**.

3. Ouvrez le groupe auquel vous souhaitez ajouter des membres. Par défaut, l’onglet **Membres** du groupe sélectionné s’affiche.

4. Sélectionnez **Ajouter des membres**.

5. Dans la page **Ajouter des membres**, sélectionnez le nom de l’utilisateur ou du groupe à ajouter en tant que membre de ce groupe, puis vérifiez qu’il a été ajouté au volet **Sélectionné**.


**Pour supprimer un utilisateur individuel d’un groupe**

1. Dans le [portail Azure Classic](https://manage.windowsazure.com), sélectionnez **Active Directory**, puis le nom du répertoire de votre organisation.

2. Sélectionnez l’onglet **Groupes**.

3. Ouvrez le groupe duquel vous souhaitez supprimer des membres.

4. Sélectionnez l’onglet **Membres**, puis le nom du membre à supprimer de ce groupe, et enfin **Supprimer**.

6. À l’invite, confirmez que vous souhaitez supprimer ce membre du groupe.


## Comment puis-je gérer dynamiquement l’appartenance à un groupe ?

Dans Azure AD, vous pouvez très facilement définir une règle simple (une règle qui ne fait qu’une seule comparaison) pour déterminer les utilisateurs qui sont membres du groupe. Par exemple, si un groupe est affecté à une application SaaS et que vous définissez une règle pour ajouter des utilisateurs qui disposent de la fonction « Représentant », tous les utilisateurs dans votre répertoire Azure AD ayant cette fonction auront accès à cette application SaaS.

> [AZURE.NOTE] Vous pouvez définir une règle d’appartenance dynamique sur les groupes de sécurité ou Office 365. Les appartenances à des groupes imbriquées ne sont pas prises en charge pour l’affectation basée sur le groupe à des applications à ce stade. L’appartenance dynamique à des groupes nécessite qu’une licence Azure AD Premium soit affectée à
>- L’administrateur qui gère la règle sur un groupe
>- Tous les utilisateurs sélectionés par la règle pour être membres du groupe

**Pour activer l’appartenance dynamique pour un groupe**

1. Dans le [portail Azure Classic](https://manage.windowsazure.com), sélectionnez **Active Directory**, puis le nom du répertoire de votre organisation.

2. Sélectionnez l’onglet **Groupes**, puis ouvrez le groupe que vous souhaitez modifier.

3. Sélectionnez l’onglet **Configurer**, puis définissez **Activer les appartenances dynamiques** sur **Oui**.

4. Définissez une seule règle simple pour le groupe, qui contrôle la manière dont l’appartenance dynamique fonctionne pour ce groupe. Assurez-vous que l’option **Ajouter des utilisateurs où** est cochée, puis sélectionnez une propriété d’utilisateur dans la liste (par exemple, department, jobTitle, etc.).

5. Ensuite, sélectionnez une condition (Non égal à, Égal à, Ne commence pas par, Commence par, Ne contient pas, Contient, Ne correspond pas, Correspond), puis spécifiez une valeur pour la propriété d’utilisateur sélectionnée.

Pour en savoir plus sur la création de règles *avancées* (règles qui peuvent contenir plusieurs comparaisons) pour l’appartenance dynamique à un groupe, consultez la page [Utilisation d’attributs pour créer des règles avancées](active-directory-accessmanagement-groups-with-advanced-rules.md).

## Informations supplémentaires

Ces articles fournissent des informations supplémentaires sur Azure Active Directory.

* [Gestion de l’accès aux ressources avec les groupes Azure Active Directory](active-directory-manage-groups.md)

* [Index d’articles pour la gestion des applications dans Azure Active Directory](active-directory-apps-index.md)

* [Qu’est-ce qu’Azure Active Directory ?](active-directory-whatis.md)

* [Intégration des identités locales dans Azure Active Directory](active-directory-aadconnect.md)

<!---HONumber=AcomDC_0330_2016-->