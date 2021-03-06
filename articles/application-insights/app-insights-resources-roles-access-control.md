<properties 
	pageTitle="Contrôle d’accès, rôles et ressources dans Application Insights" 
	description="Propriétaires, collaborateurs et lecteurs des perspectives de votre organisation." 
	services="application-insights" 
    documentationCenter=""
	authors="alancameronwills" 
	manager="douge"/>

<tags 
	ms.service="application-insights" 
	ms.workload="tbd" 
	ms.tgt_pltfrm="ibiza" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="01/05/2016" 
	ms.author="awills"/>
 
# Contrôle d’accès, rôles et ressources dans Application Insights

Vous pouvez contrôler qui a lu et mis à jour l’accès à vos données dans Visual Studio [Application Insights][start], à l’aide du [Contrôle d’accès basé sur les rôles dans Microsoft Azure](../role-based-access-control-configure.md).

> [AZURE.IMPORTANT]Accordez l’accès aux utilisateurs dans le **groupe de ressources ou l’abonnement** auquel appartient votre ressource d’application et non dans la ressource elle-même. Affectez le rôle de **collaborateur de composants Application Insights**. Cela garantit un contrôle d’accès uniforme aux tests et aux alertes Web, ainsi qu’aux ressources de votre application. [En savoir plus](#access).


## Ressources, groupes et abonnements

Quelques définitions pour commencer :

* **Ressource** : une instance d’un service Microsoft Azure. Votre ressource Application Insights collecte, analyse et affiche les données de télémétrie envoyées par votre application. Les autres types de ressources Azure incluent des applications Web, des bases de données et des machines virtuelles. 

    Pour afficher toutes vos ressources, accédez au [portail Azure][portal], connectez-vous, puis cliquez sur Parcourir.

    ![Cliquez sur Parcourir, puis sur Tout ou Filtrer par Application Insights](./media/app-insights-resources-roles-access-control/10-browse.png)

<a name="resource-group"></a>

* [**Groupe de ressources**][group] : chaque ressource appartient à un groupe. Un groupe est un moyen pratique de gérer les ressources apparentées, en particulier pour le contrôle d’accès. Par exemple, vous pouvez placer dans un groupe de ressources une application Web, une ressource Application Insights pour surveiller l’application et une ressource de stockage pour conserver les données exportées.

    
    ![Cliquez sur Parcourir, Groupes de ressources, puis choisissez un groupe](./media/app-insights-resources-roles-access-control/11-group.png)

* [**Abonnement**](https://manage.windowsazure.com) : pour utiliser Application Insights ou d’autres ressources Azure, vous devez vous connecter à un abonnement Azure. Chaque groupe de ressources appartient à un abonnement Azure, où vous choisissez votre package de prix et, s’il s’agit d’un abonnement d’organisation, sélectionnez les membres et leurs autorisations d’accès.
* [**Compte Microsoft**][account] : le nom d’utilisateur et mot de passe que vous utilisez pour vous connecter à vos abonnements Microsoft Azure, XBox Live, Outlook.com et d’autres services Microsoft.


## <a name="access"></a> Contrôle de l’accès dans le groupe de ressources

Il est important de comprendre qu’en plus de la ressource que vous avez créée pour votre application, il existe également des ressources distinctes masquées pour les alertes et les tests Web. Elles sont associées au même [groupe de ressources](#resource-group) que votre application. Vous pouvez également placer d’autres services Azure ici, comme des sites Web ou du stockage.

![Ressources dans Application Insights](./media/app-insights-resources-roles-access-control/00-resources.png)

Pour contrôler l’accès à ces ressources, il est donc recommandé de :

* contrôler l’accès au niveau du **groupe de ressources ou de l’abonnement**.
* affecter le rôle de **collaborateur de composants Application Insights**. Cela leur permet de modifier les tests Web, les alertes et les ressources d’Application Insights, sans donner accès aux autres services dans le groupe. 

## Pour fournir l’accès à un autre utilisateur

Vous devez disposer des droits du propriétaire de l’abonnement ou du groupe de ressources.

L’utilisateur doit disposer d’un [compte Microsoft][account]. Vous pouvez fournir l’accès aux personnes et aux groupes d’utilisateurs définis dans Azure Active Directory.

#### Accédez au groupe de ressources

Ajoutez l’utilisateur à cet endroit.

![Dans le panneau de ressources de votre application, ouvrez Essentials, puis le groupe de ressources et sélectionnez Paramètres/Utilisateurs. Cliquez sur Ajouter.](./media/app-insights-resources-roles-access-control/01-add-user.png)

Vous pouvez également monter d’un niveau supplémentaire et ajouter l’utilisateur à l’abonnement.

#### Sélectionnez un rôle

![Sélectionnez un rôle pour le nouvel utilisateur](./media/app-insights-resources-roles-access-control/03-role.png)

Rôle | Dans le groupe de ressources
---|---
Propriétaire | Peut tout modifier, y compris l’accès utilisateur
Collaborateur | Peut tout modifier, y compris l’ensemble des ressources
Collaborateur de composants Application Insights | Peut modifier les ressources, les tests Web et les alertes Application Insights
Lecteur | Peut afficher les éléments, mais ne peut rien modifier

Une « modification » inclut la création, la suppression et la mise à jour des éléments suivants :

* Ressources
* Tests web
* Alertes
* Exportation continue

#### Sélectionnez l’utilisateur


![Tapez l’adresse e-mail d’un nouvel utilisateur. Sélectionnez l’utilisateur](./media/app-insights-resources-roles-access-control/04-user.png)

Si l’utilisateur n’est pas dans le répertoire, vous pouvez inviter toute personne disposant d’un compte Microsoft. (Si elle utilise des services comme Outlook.com, OneDrive, Windows Phone ou XBox Live, elle dispose d’un compte Microsoft.)



## Utilisateurs et rôles

* [Contrôle d’accès en fonction du rôle dans Azure](../role-based-access-control-configure.md)



<!--Link references-->

[account]: https://account.microsoft.com
[group]: ../azure-preview-portal-using-resource-groups.md
[portal]: http://portal.azure.com/
[start]: app-insights-overview.md

 

<!---HONumber=AcomDC_0107_2016-->