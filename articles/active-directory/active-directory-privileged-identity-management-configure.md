<properties
	pageTitle="Azure AD Privileged Identity Management"
	description="Une rubrique qui explique ce qu’est Azure AD Privileged Identity Management et comment le configurer."
	services="active-directory"
	documentationCenter=""
	authors="kgremban"
	manager="stevenpo"
	editor=""/>

<tags
	ms.service="active-directory"
	ms.workload="identity"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="01/21/2016"
	ms.author="kgremban"/>

# Azure AD Privileged Identity Management

Le service Active Directory (AD) Privileged Identity Management vous permet de gérer, de contrôler et de surveiller vos identités privilégiées et leur accès aux ressources dans Azure AD et dans d’autres services en ligne Microsoft tels qu’Office 365 ou Microsoft Intune.

Pour permettre aux utilisateurs d’effectuer des opérations privilégiées, les organisations doivent souvent attribuer à plusieurs de leurs utilisateurs un accès privilégié permanent à des ressources Azure AD, Azure ou Office 365 ou à d’autres applications SaaS. Pour de nombreux clients, cela pose un risque de sécurité croissant pour leurs ressources hébergées sur le cloud, car ils ne peuvent pas suffisamment contrôler ce que font les utilisateurs avec leurs privilèges d'administrateur. En outre, un compte d’utilisateur compromis qui dispose d’un accès privilégié peut affecter leur sécurité globale sur le cloud. Azure AD Privileged Identity Management contribue à minimiser ce risque.

Grâce à Azure AD Privileged Identity Management, vous pouvez :

- Identifier les utilisateurs qui ont un rôle d’administrateur dans Azure AD
- Activer un accès administratif immédiat et à la demande aux ressources des répertoires
- Obtenir des rapports sur l'historique des accès administrateur et sur les modifications apportées aux affectations de l'administrateur
- Recevoir des alertes sur l'accès à un rôle privilégié

Azure AD Privileged Identity Management peut gérer les rôles d'organisation intégrés dans Azure Active Directory :

- Administrateur général
- Administrateur de facturation
- Administrateur de services  
- Administrateur d'utilisateurs
- Administrateur de mots de passe

## Administrateur des accès immédiats

Jusqu'ici, vous pouviez affecter un utilisateur à un rôle d'administrateur via le portail de gestion Azure ou Windows PowerShell. Cela permet à cet utilisateur de devenir **administrateur permanent**, toujours actif dans le rôle qui lui a été affecté. Cette version préliminaire ajoute une prise en charge du rôle d’**administrateur temporaire** qui s’applique à un utilisateur qui doit terminer un processus d'activation pour le rôle qui lui a été affecté. Le processus d'activation permet d’activer l'affectation de l'utilisateur à un rôle dans Azure AD.

## Activer la gestion des identités privilégiées pour votre répertoire

Vous pouvez commencer à utiliser Azure AD Privileged Identity Management en accédant au [portail Azure](https://portal.azure.com/). Pour l’instant, Azure AD Privileged Identity Management est disponible uniquement sur le portail Azure. Il n’apparaît pas dans le portail classique. Vous devez être un administrateur global pour activer Azure AD Privileged Identity Management pour un répertoire.

![Capture d’écran du portail Azure : recherche des identités privilégiées][1]

Après avoir [initialisé cette extension](active-directory-privileged-identity-management-getting-started.md), vous devenez automatiquement le premier **administrateur de sécurité** du répertoire. Seul un administrateur de sécurité peut accéder à cette extension pour gérer l'accès des autres administrateurs.

Pendant l'initialisation, une mosaïque d’Azure AD Privileged Identity Management est ajoutée au panneau de démarrage du portail Azure.

## Tableau de bord de gestion des identités privilégiées

Azure AD Privileged Identity Manager fournit un tableau de bord qui contient des informations importantes telles que :

- Le nombre d'utilisateurs affectés à chaque rôle privilégié  
- Le nombre d'administrateurs temporaires et permanents
- L’historique des accès des administrateurs

![Capture d’écran du tableau de bord PIM][2]

## Gestion des rôles privilégiés

Avec Azure AD Privileged Identity Management, vous pouvez gérer les administrateurs en ajoutant ou en supprimant des administrateurs permanents ou temporaires pour chaque rôle.

![Capture d’écran d’ajout et de suppression d’administrateurs dans PIM][3]

## Configurer les paramètres d'activation de rôle

À l'aide des paramètres d'activation de rôle, vous pouvez configurer les propriétés d'activation d’un rôle temporaire, notamment :

- La durée de la période d’activation d’un rôle
- La notification d'activation d’un rôle
- Les informations qu'un utilisateur doit fournir au cours du processus d'activation du rôle  

![Capture d’écran de l’activation d’administrateur dans les paramètres PIM][4]

## Activation d’un rôle  

Pour activer un rôle, un administrateur temporaire doit demander une « activation » limitée dans le temps pour le rôle. L'activation peut être demandée à l'aide de l’option **Activate my role** dans Azure AD Privileged Identity Management.

Un administrateur qui souhaite activer un rôle doit initialiser Azure AD Privileged Identity Management sur le portail Azure.

Toutes les catégories d'administrateur peuvent utiliser Azure AD Privileged Identity Management pour activer leur rôle.

L'activation du rôle est limitée dans le temps. Dans les paramètres d’activation de rôle, vous pouvez configurer la durée de l’activation, ainsi que les informations obligatoires que l’administrateur doit fournir pour activer le rôle.

![Capture d’écran de demande d’activation de rôle d’administrateur dans PIM][5]

## Historique d’activation des rôles

Azure AD Privileged Identity Management vous permet également d’effectuer le suivi des modifications dans les attributions de rôles privilégiés et dans l’historique d’activation des rôles. Cette opération peut être effectuée à l’aide des options de journal d'audit :

![Capture d’écran de l’historique d’activation dans PIM][6]

## Étapes suivantes
[AZURE.INCLUDE [active-directory-privileged-identity-management-toc](../../includes/active-directory-privileged-identity-management-toc.md)]

<!--Image references-->
[1]: ./media/active-directory-privileged-identity-management-configure/Search_PIM.png
[2]: ./media/active-directory-privileged-identity-management-configure/PIM_Dash.png
[3]: ./media/active-directory-privileged-identity-management-configure/PIM_AddRemove.png
[4]: ./media/active-directory-privileged-identity-management-configure/PIM_RoleActivationSettings.png
[5]: ./media/active-directory-privileged-identity-management-configure/PIM_RequestActivation.png
[6]: ./media/active-directory-privileged-identity-management-configure/PIM_ActivationHistory.png

<!---HONumber=AcomDC_0128_2016-->