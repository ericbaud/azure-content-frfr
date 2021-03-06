<properties
   pageTitle="Prise en main de la configuration d’un équilibrage de charge accessible sur Internet à l’aide d’Azure Resource Manager | Microsoft Azure"
   description="Création de règles d’équilibrage de charge, de règles NAT et d’une sonde pour Azure Resource Manager. Présentation étape par étape de la procédure complète pour créer une ressource d’équilibrage de charge."
   services="load-balancer"
   documentationCenter="na"
   authors="joaoma"
   manager="carmonm"
   editor="tysonn" />
<tags
   ms.service="load-balancer"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="infrastructure-services"
   ms.date="02/09/2016"
   ms.author="joaoma" />

# Prise en main de la configuration d’un équilibrage de charge accessible sur Internet à l’aide d’Azure Resource Manager


> [AZURE.SELECTOR]
- [Étapes du portail Azure Classic](load-balancer-internet-getstarted.md)
- [Étapes PowerShell pour Azure Resource Manager](load-balancer-arm-powershell.md)


Les étapes ci-dessous expliquent comment créer un équilibrage de charge accessible sur Internet à l’aide d’Azure Resource Manager avec PowerShell. Avec Azure Resource Manager, les éléments pour créer un équilibrage de charge accessible sur Internet sont configurés individuellement, puis rassemblés pour créer une ressource.

Dans cette page, nous allons aborder la séquence de tâches individuelles qui doivent être exécutées pour créer un équilibrage de charge et expliquer en détail ce qui est effectué pour atteindre l’objectif : créer un équilibrage de charge.


## Ce qui est nécessaire pour créer un équilibrage de charge accessible sur Internet

Les éléments suivants doivent être configurés avant la création d’un équilibrage de charge :

- Configuration d’adresses IP frontales : ajoute une adresse IP publique au pool d’adresses IP frontales pour le trafic entrant du réseau d’équilibrage de charge. 

- Pool d'adresses principales : configure les interfaces réseau qui recevront le trafic d'équilibrage de charge provenant du pool d'adresses IP frontales.

- Règles d’équilibrage de charge : configure les ports locaux et source pour l’équilibrage de charge.

- Sondes : configure la sonde d'état d'intégrité pour les instances d’un ordinateur virtuel.

- Règles NAT entrantes : configure les règles de port pour accéder directement à l'une des instances d’un ordinateur virtuel.

Pour obtenir plus d’informations sur les composants de l’équilibrage de charge avec Azure Resource Manager, consultez la page [Support Azure Resource Manager pour l’équilibrage de charge](load-balancer-arm.md).

Les étapes suivantes montrent comment configurer un équilibreur de charge entre 2 machines virtuelles.


## Étape par étape à l’aide de powershell


### Création d’un groupe de ressources pour l’équilibrage de charge


### Étape 1
Veillez à passer en mode PowerShell pour utiliser les applets de commande ARM. Pour plus d’informations, consultez [Utilisation de Windows Powershell avec Azure Resource Manager](powershell-azure-resource-manager.md).


    PS C:\> Switch-AzureMode -Name AzureResourceManager

### Étape 2

Connectez-vous à votre compte Azure.


    PS C:\> Add-AzureAccount

Vous devez indiquer vos informations d’identification.


### Étape 3

Parmi vos abonnements Azure, choisissez celui que vous souhaitez utiliser.

    PS C:\> Select-AzureSubscription -SubscriptionName "MySubscription"

Pour afficher la liste des abonnements disponibles, utilisez l'applet de commande « Get-AzureSubscription ».


### Étape 4

Créez un groupe de ressources (ignorez cette étape si vous utilisez un groupe de ressources existant)

    PS C:\> New-AzureResourceGroup -Name NRP-RG -location "West US"

Azure Resource Manager requiert que tous les groupes de ressources spécifient un emplacement. Ce dernier est utilisé comme emplacement par défaut des ressources de ce groupe. Assurez-vous que toutes les commandes pour créer un équilibrage de charge utiliseront le même groupe de ressources.

Dans l'exemple ci-dessus, nous avons créé un groupe de ressources appelé « NRP-RG » et l’emplacement « West US ».

## Créez un réseau virtuel et une adresse IP publique pour le pool IP frontal


### Étape 1

Créez un réseau virtuel :

	$backendSubnet = New-AzureVirtualNetworkSubnetConfig -Name LB-Subnet-BE -AddressPrefix 10.0.2.0/24

Crée un sous-réseau pour le réseau virtuel et définit une affectation à $backendSubnet

	New-AzurevirtualNetwork -Name NRPVNet -ResourceGroupName NRP-RG -Location "West US" -AddressPrefix 10.0.0.0/16 -Subnet $backendSubnet

Crée le réseau virtuel et ajoute le sous-réseau lb-subnet-be au réseau virtuel NRPVNet.

### Étape 2

Créez une adresse IP publique à utiliser par le pool d’adresses IP de serveur frontal :

	$publicIP = New-AzurePublicIpAddress -Name PublicIp -ResourceGroupName NRP-RG -Location "West US" –AllocationMethod Dynamic -DomainNameLabel lbip 

>[AZURE.NOTE]La propriété étiquette de nom de domaine d'adresse IP publique correspond au préfixe du nom de domaine complet de l'équilibreur de charge.

## Création du pool d’adresses IP frontales et du pool d’adresses principales

Configuration d’un pool d’adresses IP frontales pour le trafic entrant du réseau d’équilibrage de charge et d’un pool d’adresses principales pour recevoir le trafic à charge équilibrée.

### Étape 1 

À l’aide de la variable IP publique ($publicIP), créez le pool d’adresses IP frontales.

	$frontendIP = New-AzureLoadBalancerFrontendIpConfig -Name LB-Frontend -PublicIpAddress $publicIP 


### Étape 2 

Configurez un pool d’adresses principales utilisé pour recevoir le trafic entrant à partir du pool d’adresses IP frontales :

	$beaddresspool= New-AzureLoadBalancerBackendAddressPoolConfig -Name "LB-backend"


## Création des règles d’équilibrage de charge, des règles NAT, de la sonde et de l’équilibrage de charge

Après avoir créé le pool d’adresses d’IP frontales et le pool d’adresses principales, vous devrez créer les règles qui appartiendront à la ressource d’équilibrage de charge :

### Étape 1

	$inboundNATRule1= New-AzureLoadBalancerInboundNatRuleConfig -Name "RDP1" -FrontendIpConfiguration $frontendIP -Protocol TCP -FrontendPort 3441 -BackendPort 3389

	$inboundNATRule2= New-AzureLoadBalancerInboundNatRuleConfig -Name "RDP2" -FrontendIpConfiguration $frontendIP -Protocol TCP -FrontendPort 3442 -BackendPort 3389

	$healthProbe = New-AzureLoadBalancerProbeConfig -Name "HealthProbe" -RequestPath "HealthProbe.aspx" -Protocol http -Port 80 -IntervalInSeconds 15 -ProbeCount 2

 	$lbrule = New-AzureLoadBalancerRuleConfig -Name "HTTP" -FrontendIpConfiguration $frontendIP -BackendAddressPool $beAddressPool -Probe $healthProbe -Protocol Tcp -FrontendPort 80 -BackendPort 80


L’exemple ci-dessus crée les éléments suivants :

- une règle NAT selon laquelle tout le trafic entrant au port 3441 passera au port 3389 ;
- une deuxième règle NAT selon laquelle tout le trafic entrant au port 3442 passera au port 3389 ;
- une règle d’équilibrage de charge qui équilibrera la charge pour tout le trafic entrant sur le port public 80 vers le port local 80 dans le pool d’adresses principales ;
- une règle d’analyse qui vérifiera l’état d’intégrité pour le chemin d’accès « HealthProbe.aspx ».



### Étape 2

Créez l’équilibrage de charge en ajoutant tous les objets (règles NAT, règles d’équilibrage de charge, configurations de sonde) :

	$NRPLB = New-AzureLoadBalancer -ResourceGroupName "NRP-RG" -Name "NRP-LB" -Location "West US" -FrontendIpConfiguration $frontendIP -InboundNatRule $inboundNATRule1,$inboundNatRule2 -LoadBalancingRule $lbrule -BackendAddressPool $beAddressPool -Probe $healthProbe 


## Création d’interfaces réseau

Après avoir créé l’équilibrage de charge, vous devez définir les interfaces réseau qui recevront le trafic entrant du réseau à charge équilibrée, les règles NAT et la sonde. Dans ce cas, l’interface réseau est configurée individuellement et peut affectée à un ordinateur virtuel par la suite.


### Étape 1 


Obtenez le réseau virtuel des ressources et le sous-réseau pour créer des interfaces réseau :

	$vnet = Get-AzureVirtualNetwork -Name NRPVNet -ResourceGroupName NRP-RG

	$backendSubnet = Get-AzureVirtualNetworkSubnetConfig -Name LB-Subnet-BE -VirtualNetwork $vnet 


Dans cette étape, nous créons une interface réseau qui appartiendra au pool principal de l’équilibrage de charge et associons la première règle NAT pour RDP à cette interface réseau :
	
	$backendnic1= New-AzureNetworkInterface -ResourceGroupName "NRP-RG" -Name lb-nic1-be -Location "West US" -PrivateIpAddress 10.0.2.6 -Subnet $backendSubnet -LoadBalancerBackendAddressPool $nrplb.BackendAddressPools[0] -LoadBalancerInboundNatRule $nrplb.InboundNatRules[0]

### Étape 2

Créez une deuxième interface réseau appelée LB-Nic2-BE :

Dans cette étape, nous créons une deuxième interface réseau, définissons une affectation au même pool principal de l’équilibrage de charge et associons la deuxième règle NAT créée pour RDP :

 	$backendnic2= New-AzureNetworkInterface -ResourceGroupName "NRP-RG" -Name lb-nic2-be -Location "West US" -PrivateIpAddress 10.0.2.7 -Subnet $backendSubnet -LoadBalancerBackendAddressPool $nrplb.BackendAddressPools[0] -LoadBalancerInboundNatRule $nrplb.InboundNatRules[1]


On obtient alors le résultat suivant :


PS C:\> $backendnic1


	Name                 : lb-nic1-be
	ResourceGroupName    : NRP-RG
	Location             : westus
	Id                   : /subscriptions/f50504a2-1865-4541-823a-b32842e3e0ee/resourceGroups/NRP-RG/providers/Microsoft.Network/networkInterfaces/lb-nic1-be
	Etag                 : W/"d448256a-e1df-413a-9103-a137e07276d1"
	ProvisioningState    : Succeeded
	Tags                 :
	VirtualMachine       : null
	IpConfigurations     : [
                         {
                           "PrivateIpAddress": "10.0.2.6",
                           "PrivateIpAllocationMethod": "Static",
                           "Subnet": {
                             "Id": "/subscriptions/f50504a2-1865-4541-823a-b32842e3e0ee/resourceGroups/NRP-RG/providers/Microsoft.Network/virtualNetworks/NRPVNet/subnets/LB-Subnet-BE"
                           },
                           "PublicIpAddress": {
                             "Id": null
                           },
                           "LoadBalancerBackendAddressPools": [
                             {
                               "Id": "/subscriptions/f50504a2-1865-4541-823a-b32842e3e0ee/resourceGroups/NRP-RG/providers/Microsoft.Network/loadBalancers/NRPlb/backendAddressPools/LB-backend"
                             }
                           ],
                           "LoadBalancerInboundNatRules": [
                             {
                               "Id": "/subscriptions/f50504a2-1865-4541-823a-b32842e3e0ee/resourceGroups/NRP-RG/providers/Microsoft.Network/loadBalancers/NRPlb/inboundNatRules/RDP1"
                             }
                           ],
                           "ProvisioningState": "Succeeded",
                           "Name": "ipconfig1",
                           "Etag": "W/"d448256a-e1df-413a-9103-a137e07276d1"",
                           "Id": "/subscriptions/f50504a2-1865-4541-823a-b32842e3e0ee/resourceGroups/NRP-RG/providers/Microsoft.Network/networkInterfaces/lb-nic1-be/ipConfigurations/ipconfig1"
                         }
                       ]
	DnsSettings          : {
                         "DnsServers": [],
                         "AppliedDnsServers": []
                       }
	AppliedDnsSettings   :
	NetworkSecurityGroup : null
	Primary              : False



### Étape 3 

Utilisez la commande Add-AzureVMNetworkInterface pour affecter la carte réseau à un ordinateur virtuel.

Pour la procédure détaillée à suivre afin de créer une machine virtuelle et définir une affectation à une carte réseau, consultez la documentation [Création et configuration d’une machine virtuelle Windows avec Resource Manager et Azure PowerShell](../virtual-machines/virtual-machines-windows-create-powershell.md#Example), option 4 ou 5.

## Mettre à jour un équilibreur de charge existant


### Étape 1 :

À l’aide de l’équilibreur de charge de l’exemple ci-dessus, attribuez un objet d’équilibreur de charge à la variable $slb à l’aide de Get-AzureLoadBalancer

	$slb=get-azureLoadBalancer -Name NRP-LB -ResourceGroupName NRP-RG

### Étape 2 :

Dans l’exemple suivant, vous allez ajouter une nouvelle règle NAT entrante en utilisant le port 81 dans le serveur frontal et le port 8181 pour le pool principal, à un équilibreur de charge existant.

	$slb | Add-AzureLoadBalancerInboundNatRuleConfig -Name NewRule -FrontendIpConfiguration $slb.FrontendIpConfigurations[0] -FrontendPort 81  -BackendPort 8181 -Protocol Tcp


### Étape 3

Enregistrez la nouvelle configuration à l’aide de Set-AzureLoadBalancer

	$slb | Set-AzureLoadBalancer

## Supprimer un équilibreur de charge

Utilisez la commande Remove-AzureLoadBalancer pour supprimer un équilibreur de charge créé précédemment appelé « NRP-LB » dans un groupe de ressources appelé « NRP-RG ».

	Remove-AzureLoadBalancer -Name NRP-LB -ResourceGroupName NRP-RG

>[AZURE.NOTE] Vous pouvez utiliser le commutateur facultatif -Force pour éviter l’invite relative à la suppression.


## Voir aussi

[Configuration d’un mode de distribution d’équilibrage de charge](load-balancer-distribution-mode.md)

[Configuration des paramètres de délai d’expiration TCP inactif pour votre équilibrage de charge](load-balancer-tcp-idle-timeout.md)
 

<!---HONumber=AcomDC_0323_2016-->