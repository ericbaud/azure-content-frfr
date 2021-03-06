<properties
	pageTitle="Dépannage détaillé du Bureau à distance | Microsoft Azure"
	description="Étapes de dépannage détaillées des connexions RDP à une machine virtuelle Azure exécutant Windows."
	services="virtual-machines-windows"
	documentationCenter=""
	authors="dsk-2015"
	manager="timlt"
	editor=""
	tags="top-support-issue,azure-service-management,azure-resource-manager"/>

<tags
	ms.service="virtual-machines-windows"
	ms.workload="infrastructure-services"
	ms.tgt_pltfrm="vm-windows"
	ms.devlang="na"
	ms.topic="article"
	ms.date="01/08/2016"
	ms.author="dkshir"/>

# Dépannage de connexions du Bureau à distance à des machines virtuelles Azure Windows

[AZURE.INCLUDE [learn-about-deployment-models](../../includes/learn-about-deployment-models-both-include.md)]

Cet article décrit les étapes de dépannage détaillées pour diagnostiquer et résoudre les erreurs complexes du Bureau à distance pour les machines virtuelles basées Azure sur Windows.

> [AZURE.IMPORTANT] Pour éliminer les erreurs du Bureau à distance plus courantes, veillez à lire l’article sur [le dépannage de base pour le Bureau à distance](virtual-machines-windows-troubleshoot-rdp-connection.md) avant de continuer.

Si vous obtenez un message d’erreur du Bureau à distance qui ne ressemble à aucun des messages d’erreur spécifiques couverts dans [le guide de dépannage de base des problèmes du Bureau à distance](virtual-machines-windows-troubleshoot-rdp-connection.md), vous pouvez suivre ces étapes et essayer de déterminer pourquoi le client Bureau à distance (ou [RDP](https://en.wikipedia.org/wiki/Remote_Desktop_Protocol)) ne parvient pas à se connecter au service RDP sur la machine virtuelle Azure.

Si vous avez besoin d'aide supplémentaire concernant n'importe quel point de cet article, contactez les experts Azure sur les [forums MSDN Azure et Stack Overflow](https://azure.microsoft.com/support/forums/). Vous pouvez également signaler un incident au support Azure. Accédez au [site de support Azure](https://azure.microsoft.com/support/options/), puis cliquez sur **Obtenir un support**. Pour plus d’informations sur l’utilisation du support Azure, lisez la [FAQ du support Microsoft Azure](https://azure.microsoft.com/support/faq/).


## Composants d’une connexion Bureau à distance

Voici les composants impliqués dans une connexion RDP :

![](./media/virtual-machines-windows-detailed-troubleshoot-rdp/tshootrdp_0.png)

Avant de poursuivre, nous vous recommandons de réfléchir à tout ce qui a changé depuis que vous avez créé avec succès une connexion Bureau à distance à la machine virtuelle. Par exemple :

- Si l'adresse IP publique de la machine l'ordinateur virtuel ou du service cloud contenant la machine virtuelle (également appelée adresse IP virtuelle [VIP](https://en.wikipedia.org/wiki/Virtual_IP_address)) a changé, l’erreur RDP peut indiquer que le cache client DNS a toujours l’*ancienne adresse IP* enregistrée pour le nom DNS. Videz le cache client DNS et essayez de vous reconnecter à la machine virtuelle. Ou essayez de vous connecter directement avec la nouvelle adresse IP virtuelle.
- Si vous utilisez une application tierce pour gérer vos connexions Bureau à distance au lieu d’utiliser l’un des portails Azure, vérifiez que la configuration de l’application inclut bien le port TCP approprié pour le trafic de Bureau à distance. Vous pouvez vérifier ce port pour une machine virtuelle classique dans le [portail Azure](https://portal.azure.com), en cliquant sur Paramètres de la machine virtuelle > Points de terminaison.


## Étapes préliminaires

Avant de passer à la procédure de dépannage détaillé,

- vérifiez l’état de la machine virtuelle sur le portail Azure Classic ou le portail Azure pour identifier d’éventuels problèmes flagrants
- Suivez les [étapes du guide de dépannage de base pour corriger rapidement les erreurs RDP courantes](virtual-machines-windows-troubleshoot-rdp-connection.md#quickfixrdp)


Essayez de vous reconnecter à la machine virtuelle via le Bureau à distance après ces étapes.


## Dépannage détaillé

Le client Bureau à distance peut ne pas être en mesure d’atteindre le service Bureau à distance sur la machine virtuelle Azure en raison de problèmes au niveau des sources suivantes :

- ordinateur client de Bureau à distance ;
- périphérique de périmètre intranet de l’entreprise ;
- point de terminaison de service cloud et liste de contrôle d’accès (ACL) ;
- groupes de sécurité réseau ;
- machine virtuelle Azure Windows.

### Source 1 : Ordinateur client de Bureau à distance

Vérifiez que votre ordinateur peut établir des connexions Bureau à distance avec un autre ordinateur Windows local.

![](./media/virtual-machines-windows-detailed-troubleshoot-rdp/tshootrdp_1.png)

Si vous n’y parvenez pas, recherchez sur votre ordinateur :

- un paramètre de pare-feu local qui bloque le trafic de Bureau à distance ;
- un logiciel proxy client installé localement qui empêche les connexions Bureau à distance ;
- un logiciel de surveillance réseau installé localement qui empêche les connexions Bureau à distance ;
- d’autres types de logiciel de sécurité qui analysent le trafic ou autorisent/interdisent des types spécifiques de trafic empêchant les connexions Bureau à distance.

Dans tous les cas, désactivez temporairement le logiciel concerné et essayez d’établir une connexion avec un ordinateur local via Bureau à distance. Si vous ne parvenez pas à identifier l’origine du problème de cette façon, contactez votre administrateur réseau pour corriger les paramètres logiciels afin d’autoriser les connexions Bureau à distance.

### Source 2 : Périphérique de périmètre intranet de l’entreprise

Vérifiez qu’un ordinateur directement connecté à Internet peut établir des connexions Bureau à distance avec votre machine virtuelle Azure.

![](./media/virtual-machines-windows-detailed-troubleshoot-rdp/tshootrdp_2.png)

Si vous n’avez pas d’ordinateur directement connecté à Internet, créez et testez une nouvelle machine virtuelle Azure dans un groupe de ressources ou service cloud. Pour plus d’informations, consultez [Création d’une machine virtuelle exécutant Windows dans Azure](virtual-machines-windows-hero-tutorial.md). Une fois le test terminé, supprimez la machine virtuelle et le groupe de ressources ou le service cloud.

Si vous pouvez créer une connexion Bureau à distance avec un ordinateur directement connecté à Internet, recherchez sur votre périphérique de périmètre intranet d’entreprise :

- un pare-feu qui bloque les connexions HTTPS à Internet ;
- un serveur proxy qui empêche les connexions Bureau à distance ;
- un logiciel de détection d’intrusion ou de surveillance réseau s’exécutant sur les périphériques de votre réseau de périmètre et qui empêche les connexions Bureau à distance.

Contactez votre administrateur réseau pour corriger les paramètres de votre périphérique de périmètre intranet d’entreprise afin d’autoriser les connexions Bureau à distance basées sur HTTPS à Internet.

### Source 3 : Point de terminaison de service cloud et liste de contrôle d’accès

Pour des machines virtuelles créées à l’aide du modèle de déploiement classique, vérifiez qu’une autre machine virtuelle Azure du même service cloud ou réseau virtuel peut établir des connexions Bureau à distance avec votre machine virtuelle Azure.

![](./media/virtual-machines-windows-detailed-troubleshoot-rdp/tshootrdp_3.png)

> [AZURE.NOTE] Pour les machines virtuelles créées dans Resource Manager, passez à [Source 4 : groupes de sécurité réseau](#nsgs).

Si vous ne disposez pas d’une autre machine virtuelle dans le même service cloud ou réseau virtuel, vous pouvez en créer une en suivant les étapes de la rubrique [Création d’une machine virtuelle exécutant Windows dans Azure](virtual-machines-windows-hero-tutorial.md). Une fois le test terminé, supprimez la machine virtuelle supplémentaire.

Si vous pouvez vous connecter à une machine virtuelle via Bureau à distance dans le même service cloud ou réseau virtuel, vérifiez les paramètres suivants :

- La configuration du point de terminaison pour le trafic de Bureau à distance sur la machine virtuelle cible : le port TCP privé du point de terminaison doit correspondre au port TCP sur lequel le service Bureau à distance de la machine virtuelle procède à l’écoute (le port 3389, par défaut).
- La liste de contrôle d’accès du point de terminaison du trafic Bureau à distance sur la machine virtuelle cible : les listes de contrôle d’accès vous permettent de spécifier le trafic Internet entrant autorisé et interdit en fonction de l’adresse IP source. Une mauvaise configuration des listes de contrôle d’accès peut empêcher le trafic du Bureau à distance d’accéder au point de terminaison. Examinez vos listes de contrôle d’accès pour vous assurer que le trafic entrant provenant des adresses IP publiques de votre proxy ou d’un autre serveur Edge est autorisé. Pour plus d’informations, consultez [Qu’est-ce qu’une liste de contrôle d’accès (ACL) réseau ?](../virtual-network/virtual-networks-acl.md).

Pour vérifier si le point de terminaison est la source du problème, supprimez le point de terminaison actuel et créez un autre point en choisissant un port aléatoire dont le numéro externe se situe entre 49152 et 65535. Pour plus d’informations, consultez [Configuration des points de terminaison sur une machine virtuelle](virtual-machines-windows-classic-setup-endpoints.md).

### <a id="nsgs"></a>Source 4 : groupes de sécurité réseau

Les groupes de sécurité réseau vous permettent de contrôler plus précisément le trafic entrant et sortant autorisé. Vous pouvez créer des règles qui s’étendent aux sous-réseaux et aux services cloud d’un réseau virtuel Azure. Examinez les règles de votre groupe de sécurité réseau pour vous assurer que le trafic de Bureau à distance provenant d’Internet est autorisé.

Pour plus d’informations, consultez [Présentation du groupe de sécurité réseau](../virtual-network/virtual-networks-nsg.md).

### Source 5 : Machine virtuelle Azure Windows

![](./media/virtual-machines-windows-detailed-troubleshoot-rdp/tshootrdp_5.png)

Utilisez le [package de diagnostic Azure IaaS (Windows)](https://home.diagnostics.support.microsoft.com/SelfHelp?knowledgebaseArticleFilter=2976864) pour vérifier si le problème est dû à la machine virtuelle Azure elle-même. Si ce package de diagnostics ne peut pas résoudre le problème de **connectivité RDP à une machine virtuelle Azure (redémarrage requis)**, suivez les instructions de [cet article](virtual-machines-windows-reset-rdp.md) pour réinitialiser le service Bureau à distance sur la machine virtuelle. Avec cette opération, vous pouvez :

- activer la règle par défaut du pare-feu Windows Bureau à distance (port TCP 3389) ;
- activer les connexions Bureau à distance en définissant la valeur de registre HKLM\\System\\CurrentControlSet\\Control\\Terminal Server\\fDenyTSConnections sur 0.

Essayez une nouvelle fois de vous connecter à partir de votre ordinateur. Si vous ne réussissez toujours pas à vous connecter via Bureau à distance, cela peut être dû à l’une des raisons suivantes :

- Le service Bureau à distance ne fonctionne pas sur la machine virtuelle cible.
- Le service Bureau à distance n’est pas compatible avec l’écoute sur le port TCP 3389.
- Le pare-feu Windows ou un autre pare-feu local comporte une règle sortante qui empêche le trafic du Bureau à distance.
- Le logiciel de détection d’intrusion ou de surveillance réseau s’exécutant sur la machine virtuelle Azure empêche les connexions Bureau à distance.

Pour les machines virtuelles créées à l’aide du modèle de déploiement classique, vous pouvez utiliser une session Azure PowerShell distante vers la machine virtuelle Azure. Tout d’abord, vous devez installer un certificat pour le service cloud d’hébergement de la machine virtuelle. Accédez à [Configurer l’accès à distance sécurisé de PowerShell vers les machines virtuelles Azure](http://gallery.technet.microsoft.com/scriptcenter/Configures-Secure-Remote-b137f2fe) et téléchargez le fichier de script **InstallWinRMCertAzureVM.ps1** sur votre ordinateur local.

Installez ensuite Azure PowerShell si ce n’est pas déjà fait. Consultez [Installation et configuration d’Azure PowerShell](../powershell-install-configure.md).

Ouvrez ensuite une invite de commandes Azure PowerShell, puis remplacez le dossier actif par l’emplacement du fichier de script **InstallWinRMCertAzureVM.ps1**. Pour exécuter un script Azure PowerShell, vous devez définir la bonne stratégie d’exécution. Exécutez la commande **Get-ExecutionPolicy** afin de déterminer votre niveau de stratégie actuel. Pour plus d’informations sur la définition du niveau approprié, consultez [Set-ExecutionPolicy](https://technet.microsoft.com/library/hh849812.aspx).

Indiquez ensuite le nom de votre abonnement Azure, le nom du service cloud et le nom de votre machine virtuelle (en supprimant les caractères < and >), puis exécutez ces commandes.

	$subscr="<Name of your Azure subscription>"
	$serviceName="<Name of the cloud service that contains the target virtual machine>"
	$vmName="<Name of the target virtual machine>"
	.\InstallWinRMCertAzureVM.ps1 -SubscriptionName $subscr -ServiceName $serviceName -Name $vmName

Le nom d’abonnement correct apparaît dans la propriété **SubscriptionName** de l’affichage de la commande _Get-AzureSubscription_. Le nom du service cloud pour la machine virtuelle apparaît dans la colonne _ServiceName_ de l’affichage de la commande **Get-AzureVM**.

Vérifiez que vous disposez du nouveau certificat, ouvrez un composant logiciel enfichable Certificats pour l’utilisateur actuel, puis examinez le dossier **Autorités de certification racines de confiance\\Certificats**. Vous devriez voir un certificat portant le nom DNS de votre service cloud doit apparaître dans la colonne Issued To (exemple : cloudservice4testing.cloudapp.net).

Lancez ensuite une session Azure PowerShell distante à l’aide de ces commandes.

	$uri = Get-AzureWinRMUri -ServiceName $serviceName -Name $vmName
	$creds = Get-Credential
	Enter-PSSession -ConnectionUri $uri -Credential $creds

Une fois que vous avez entré les informations d’identification administrateur valides, l’invite de commande Azure PowerShell doit indiquer un message de ce type :

	[cloudservice4testing.cloudapp.net]: PS C:\Users\User1\Documents>

La première partie de l’invite de commande représente le nom de votre service cloud qui contient la machine virtuelle cible, qui peut être différent de « cloudservice4testing.cloudapp.net ». Vous pouvez maintenant émettre des commandes Azure PowerShell pour ce service cloud afin d’examiner les problèmes supplémentaires cités ci-dessus et corriger la configuration.

### Correction manuelle des Services Bureau à distance permettant l’écoute du port TCP

Si vous ne parvenez pas à exécuter le [package de diagnostic Azure IaaS (Windows)](https://home.diagnostics.support.microsoft.com/SelfHelp?knowledgebaseArticleFilter=2976864) pour résoudre le problème de **connectivité RDP à une machine virtuelle Azure (redémarrage requis)**, exécutez cette commande à l’invite de session Azure PowerShell distante.

	Get-ItemProperty -Path "HKLM:\System\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" -Name "PortNumber"

La propriété PortNumber indique le numéro de port actuel. Si nécessaire, utilisez cette commande pour modifier le numéro de port du Bureau à distance afin de revenir à sa valeur par défaut (3389).

	Set-ItemProperty -Path "HKLM:\System\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" -Name "PortNumber" -Value 3389

Vérifiez que le port affiche désormais la valeur 3389 à l’aide de cette commande.

	Get-ItemProperty -Path "HKLM:\System\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" -Name "PortNumber"

Quittez la session Azure PowerShell distante à l’aide de cette commande.

	Exit-PSSession

Vérifiez que le point de terminaison du Bureau à distance de la machine virtuelle Azure utilise également le port TCP 3398 comme port interne. Redémarrez la machine virtuelle Azure puis testez de nouveau la connexion Bureau à distance.


## Ressources supplémentaires

[Package de diagnostic Azure IaaS (Windows)](https://home.diagnostics.support.microsoft.com/SelfHelp?knowledgebaseArticleFilter=2976864)

[Réinitialisation d’un mot de passe ou du service Bureau à distance pour les machines virtuelles Windows](virtual-machines-windows-reset-rdp.md)

[Installation et configuration d’Azure PowerShell](../powershell-install-configure.md)

[Résolution des problèmes des connexions SSH avec une machine virtuelle Azure Linux](virtual-machines-linux-troubleshoot-ssh-connection.md)

[Résoudre les problèmes d’accès à une application exécutée sur une machine virtuelle Azure](virtual-machines-linux-troubleshoot-app-connection.md)

<!---HONumber=AcomDC_0323_2016-->