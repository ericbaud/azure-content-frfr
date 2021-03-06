<properties
	pageTitle="Configurer l’intégration du coffre de clés Azure SQL Server sur des machines virtuelles (Resource Manager)"
	description="Apprenez à automatiser la configuration du chiffrement de SQL Server pour une utilisation avec Azure Key Vault. Cette rubrique explique comment utiliser l’intégration de coffre de clés Azure avec des machines virtuelles SQL Server créées avec Resource Manager."
	services="virtual-machines-windows"
	documentationCenter=""
	authors="rothja"
	manager="jeffreyg"
	editor=""
	tags="azure-service-management"/>

<tags
	ms.service="virtual-machines-windows"
	ms.devlang="na"
	ms.topic="article"
	ms.tgt_pltfrm="vm-windows-sql-server"
	ms.workload="infrastructure-services"
	ms.date="03/24/2016"
	ms.author="jroth"/>

# Configurer l’intégration du coffre de clés Azure SQL Server sur des machines virtuelles (Resource Manager)

> [AZURE.SELECTOR]
- [Gestionnaire de ressources](virtual-machines-windows-ps-sql-keyvault.md)
- [Classique](virtual-machines-windows-classic-ps-sql-keyvault.md)

## Vue d'ensemble
Il existe plusieurs fonctionnalités de chiffrement SQL Server, telles que [ le chiffrement transparent des données (TDE)](https://msdn.microsoft.com/library/bb934049.aspx), [le chiffrement au niveau des colonnes (CLE)](https://msdn.microsoft.com/library/ms173744.aspx) et [le chiffrement de sauvegarde](https://msdn.microsoft.com/library/dn449489.aspx). Ces types de chiffrement nécessitent que vous gériez et stockiez les clés de chiffrement que vous utilisez pour le chiffrement. Le service Azure Key Vault (coffre de clés Azure, AKV) est conçu pour optimiser la sécurité et la gestion de ces clés dans un emplacement sécurisé et hautement disponible. Le [connecteur SQL Server](http://www.microsoft.com/download/details.aspx?id=45344) permet à SQL Server d’utiliser ces clés depuis le coffre de clés Azure.

[AZURE.INCLUDE [learn-about-deployment-models](../../includes/learn-about-deployment-models-rm-include.md)]Modèle de déploiement classique

Si vous exécutez SQL Server sur des machines locales, vous devez [suivre la procédure d’accès au coffre de clés Azure à partir de votre machine SQL Server locale](https://msdn.microsoft.com/library/dn198405.aspx). Mais, pour SQL Server dans des machines virtuelles Azure, vous pouvez gagner du temps à l'aide de la fonctionnalité *Azure Key Vault Integration*.

Lorsque cette fonctionnalité est activée, elle installe automatiquement le connecteur SQL Server, configure le fournisseur EKM pour accéder à Azure Key Vault et crée les informations d'identification vous permettant d'accéder à votre coffre. Si vous avez examiné les étapes décrites dans la documentation locale mentionnée précédemment, vous pouvez voir que cette fonctionnalité automatise les étapes 2 et 3. La seule étape que vous devez encore exécuter manuellement consiste à créer le coffre de clé et des clés. À partir de là, la configuration complète de votre machine virtuelle SQL est automatisée. Une fois que cette fonctionnalité a terminé cette configuration, vous pouvez exécuter des instructions T-SQL pour crypter vos bases de données et vos sauvegardes comme vous y êtes habitué.

[AZURE.INCLUDE [Préparation pour l’intégration du coffre de clés Azure](../../includes/virtual-machines-sql-server-akv-prepare.md)]

## Activation de l’intégration d’AKV
Si vous approvisionnez une nouvelle machine virtuelle SQL Server avec Resource Manager, le portail Azure permet d’activer l’intégration d’Azure Key Vault.

![Intégration du coffre de clés Azure SQL ARM](./media/virtual-machines-windows-ps-sql-keyvault/azure-sql-arm-akv.png)

Consultez la procédure pas à pas détaillée de l’approvisionnement dans l’article [Provision a SQL Server virtual machine in the Azure Portal](virtual-machines-windows-portal-sql-server-provision.md) (Approvisionner une machine virtuelle SQL Server dans le portail Azure).

Si vous devez activer l’intégration d’AKV sur une machine virtuelle existante, vous pouvez utiliser un modèle. Pour plus d’informations, consultez l’article [Azure quickstart template for Azure Key Vault integration](https://github.com/Azure/azure-quickstart-templates/tree/master/101-vm-sql-keyvault-setup) (Modèle de démarrage rapide d’Azure pour l’intégration d’Azure Key Vault).


[AZURE.INCLUDE [Étapes suivantes de l’intégration du coffre de clés Azure](../../includes/virtual-machines-sql-server-akv-next-steps.md)]

<!---HONumber=AcomDC_0330_2016-->