<properties
   pageTitle="Créer une identité professionnelle ou scolaire dans AAD | Microsoft Azure"
   description="Découvrez comment créer une identité professionnelle ou scolaire dans Azure Active Directory à utiliser avec vos machines virtuelles Windows."
   services="virtual-machines-windows"
   documentationCenter=""
   authors="squillace"
   manager="timlt"
   editor=""
   tags="azure-service-management,azure-resource-manager"/>

<tags
   ms.service="virtual-machines-windows"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="vm-windows"
   ms.workload="infrastructure"
   ms.date="12/08/2015"
   ms.author="rasquill"/>

# Création d’une identité professionnelle ou scolaire dans Azure Active Directory à utiliser avec des machines virtuelles Windows

Si vous avez créé un compte Azure personnel ou si vous disposez d'un abonnement MSDN personnel et avez créé le compte Azure pour profiter des crédits Azure MSDN, alors vous avez utilisé une identité de *compte Microsoft* pour le créer. Pour fonctionner correctement, de nombreuses fonctionnalités d'Azure, parmi lesquelles les [modèles de groupes de ressources](../resource-group-overview.md), nécessitent un compte professionnel ou scolaire (une identité gérée par Azure Active Directory). Vous pouvez suivre les instructions ci-dessous pour créer un compte professionnel ou scolaire car, heureusement, l’un des atouts de votre compte personnel Azure est qu'il est fourni avec un domaine Azure Active Directory par défaut que vous pouvez utiliser pour créer un nouveau compte professionnel ou scolaire à utiliser avec les fonctionnalités Azure qui le nécessitent.

Toutefois, de récentes modifications permettent de gérer votre abonnement avec n'importe quel type de compte Azure grâce à la `azure login`méthode de connexion interactive décrite [ici](../xplat-cli-connect.md). Vous pouvez utiliser ce mécanisme ou suivre les instructions ci-dessous. Vous pouvez également [créer une identité professionnelle ou scolaire dans Azure Active Directory à utiliser avec des machines virtuelles Linux](virtual-machines-linux-create-aad-work-id.md).

[AZURE.INCLUDE [learn-about-deployment-models](../../includes/learn-about-deployment-models-both-include.md)]

[AZURE.INCLUDE [virtual-machines-common-create-aad-work-id](../../includes/virtual-machines-common-create-aad-work-id.md)]

<!---HONumber=AcomDC_0330_2016-->