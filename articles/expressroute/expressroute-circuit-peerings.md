<properties 
   pageTitle="Circuits ExpressRoute et domaines de routage | Microsoft Azure"
   description="Cette page fournit une vue d'ensemble des circuits ExpressRoute et des domaines de routage."
   documentationCenter="na"
   services="expressroute"
   authors="cherylmc"
   manager="carolz"
   editor=""/>
<tags 
   ms.service="expressroute"
   ms.devlang="na"
   ms.topic="article" 
   ms.tgt_pltfrm="na"
   ms.workload="infrastructure-services" 
   ms.date="03/21/2016"
   ms.author="cherylmc"/>

# Circuits ExpressRoute et domaines de routage

 Vous devez commander un *circuit ExpressRoute* pour connecter votre réseau local à Microsoft via un fournisseur de connectivité. La figure ci-dessous fournit une représentation logique de la connectivité entre votre réseau étendu et Microsoft.

![](./media/expressroute-circuit-peerings/expressroute-basic.png)

## Circuits ExpressRoute

Un *circuit ExpressRoute* représente une connexion logique entre votre infrastructure local et les services de cloud Microsoft via un fournisseur de connectivité. Vous pouvez commander plusieurs circuits ExpressRoute. Chaque circuit peut figurer dans des régions identiques ou différentes, et peut être connecté à leur site via des fournisseurs de connectivité.

Les circuits ExpressRoute ne sont mappés à aucune entité physique. Un circuit est identifié par un GUID standard appelé clé de service (s-key). La clé de service est la seule information échangée entre Microsoft, le fournisseur de connectivité, et vous. La clé de service n'est pas secrète pour des raisons de sécurité. Il existe un mappage 1:1 entre un circuit ExpressRoute et la clé de service.

Un circuit ExpressRoute peut avoir jusqu'à trois homologations indépendantes : public Azure, privé Azure et Microsoft. Chaque homologation est une paire de sessions BGP indépendantes, chacune configurée de manière redondante pour offrir une disponibilité optimale. Il existe un mappage 1:N (1 <= N <= 3) entre un circuit ExpressRoute et des domaines de routage. Un circuit ExpressRoute peut avoir une, deux ou trois homologations activées par circuit ExpressRoute.
 
Chaque circuit offre une bande passante fixe (50 Mbps, 100 Mbps, 200 Mbps, 500 Mbps, 1 Gbps, 10 Gbps) et est mappé à un fournisseur de connectivité et un emplacement d’homologation. La bande passante que vous sélectionnez sera partagée par toutes les homologations pour le circuit.

### Quotas, limites et limitations

Des limites et quotas par défaut s'appliquent à chaque circuit ExpressRoute. Consultez la page [Limites, quotas et contraintes applicables à l’abonnement et au service Azure](../azure-subscription-service-limits.md) pour obtenir des informations actualisées sur les quotas.

## Domaines de routage ExpressRoute

Un circuit ExpressRoute est associé à plusieurs domaines de routage : public Azure, privé Azure et Microsoft. Chaque domaine de routage est configuré de manière identique sur une paire de routeurs (en configuration actif-actif ou de répartition de la charge) pour la haute disponibilité. Les services Azure sont classés en *public Azure* et *privé Azure* pour représenter les schémas d'adressage IP.


![](./media/expressroute-circuit-peerings/expressroute-peerings.png)


### Homologation privée

Les services de calcul Azure, à savoir les machines virtuelles (IaaS) et les services cloud (PaaS) déployés au sein d’un réseau virtuel peuvent être connectés via le domaine d’homologation privée. Celui-ci est considéré comme une extension de confiance de votre réseau de base dans Microsoft Azure. Vous pouvez configurer une connectivité bidirectionnelle entre votre réseau de base et les réseaux virtuels Azure. Cela vous permet de vous connecter aux machines virtuelles et services cloud directement sur leurs adresses IP privées.

Vous pouvez connecter plusieurs réseaux virtuels au domaine d’homologation privée. Pour plus d’informations sur les limitations, consultez le [Forum Aux Questions](expressroute-faqs.md). Vous pouvez consulter la page [Limites, quotas et contraintes applicables à l’abonnement et au service Azure](../azure-subscription-service-limits.md) pour obtenir des informations actualisées sur les limites. Reportez-vous à la page [Routage](expressroute-routing.md) pour plus d'informations sur la configuration du routage.

### Homologation publique

Les services tels qu’Azure Storage, Base de données SQL et Sites Web sont proposés sur des adresses IP publiques. En privé, vous pouvez vous connecter à des services hébergés sur des adresses IP publiques (y compris les adresses IP virtuelles de vos services cloud) via le domaine de routage d’homologation publique. Vous pouvez connecter le domaine d’homologation publique à votre zone DMZ et vous connecter à tous les services Azure sur leurs adresses IP publiques à partir de votre réseau étendu, sans avoir à vous connecter via Internet.

La connectivité est toujours initiée de votre réseau étendu vers les services Microsoft Azure. Ces derniers ne sont pas en mesure d’initier des connexions à votre réseau via ce domaine de routage. Une fois l’homologation publique activée, vous êtes en mesure de vous connecter à tous les services Azure. Nous ne vous permettons pas de sélectionner les services pour lesquels nous publions les itinéraires. Vous pouvez consulter la liste des préfixes que nous publions via cette homologation à la page [Plages d’adresses IP des centres de données Microsoft Azure](http://www.microsoft.com/download/details.aspx?id=41653). La page est actualisée chaque semaine.

Vous pouvez définir des filtres d’itinéraires personnalisés au sein de votre réseau pour utiliser uniquement les itinéraires dont vous avez besoin. Reportez-vous à la page [Routage](expressroute-routing.md) pour plus d'informations sur la configuration du routage. Vous pouvez définir des filtres d’itinéraires personnalisés au sein de votre réseau pour utiliser uniquement les itinéraires dont vous avez besoin.

Pour plus d’informations sur les services pris en charge via le domaine de routage d’homologation publique, consultez le [Forum Aux Questions](expressroute-faqs.md).
 
### Homologation Microsoft

La connectivité à toutes les autres services en ligne Microsoft (tels qu’Office 365) a lieu via l’homologation Microsoft. Nous activons la connectivité bidirectionnelle entre votre réseau étendu et les services cloud Microsoft via le domaine de routage d’homologation Microsoft. Vous devez vous connecter aux services cloud Microsoft uniquement via des adresses IP publiques qui sont détenues par vous ou votre fournisseur de connectivité. Vous devez également respecter toutes les règles définies. Pour plus d’informations, consultez la page [Conditions préalables d’ExpressRoute](expressroute-prerequisites.md).

Pour plus d’informations sur les services pris en charge, les coûts et les détails de configuration, consultez le [Forum Aux Questions](expressroute-faqs.md). Pour plus d’informations sur la liste des fournisseurs de connectivité offrant une prise en charge de l’homologation Microsoft, consultez la page [Emplacements ExpressRoute](expressroute-locations.md)

## Comparaison des domaines de routage

Le tableau ci-dessous compare les trois domaines de routage.

||**Homologation privée**|**Homologation publique**|**Homologation Microsoft**|
|---|---|---|---|
|**Nb max. de préfixes pris en charge par homologation**|4 000 par défaut, 10 000 avec ExpressRoute Premium|200|200|
|**Plages d’adresses IP prises en charge**|Toute adresse IPv4 valide au sein de votre réseau étendu.|Adresses IPv4 publiques détenues par vous ou par votre fournisseur de connectivité.|Adresses IPv4 publiques détenues par vous ou par votre fournisseur de connectivité.|
|**Exigences en matière de numéros AS**|Numéros AS publics et privés Vous devez posséder un numéro AS public. | Numéros AS publics et privés Toutefois, vous devez fournir des numéros AS publics pour valider la propriété des adresses IP publiques.| Numéros AS publics et privés Toutefois, vous devez fournir des numéros AS publics pour valider la propriété des adresses IP publiques.|
|**Adresses IP de l’interface de routage**|RFC1918 et adresses IP publiques|Adresses IP publiques enregistrées auprès de vous dans les registres de routage.| Adresses IP publiques enregistrées auprès de vous dans les registres de routage.|
|**Prise en charge du hachage MD5**| Oui|Oui|Oui|

Vous pouvez choisir d’activer un ou plusieurs domaines de routage dans le cadre de leur circuit ExpressRoute. Vous pouvez choisir de placer tous les domaines de routage sur le même VPN si vous souhaitez les combiner dans un domaine de routage unique. Vous pouvez également les placer dans différents domaines de routage comme indiqué dans le schéma. Nous vous recommandons de connecter l’homologation privée directement à votre réseau principal, et les homologations publiques et Microsoft à votre zone DMZ.
 
Si vous choisissez d’avoir les trois sessions d’homologation, vous devez disposer de trois paires de sessions BGP (une paire pour chaque type d’homologation). Les paires de session BGP fournissent un lien hautement disponible. Si vous vous connectez via des fournisseurs de couche 2, il vous incombe de configurer et de gérer le routage. Pour en savoir plus, passez en revue les [workflows](expressroute-workflows.md) d’ExpressRoute.

## Étapes suivantes

- Recherchez un fournisseur de services. Consultez la rubrique [Emplacements et fournisseurs de services ExpressRoute](expressroute-locations.md).
- Vérifiez que toutes les conditions préalables sont remplies. Consultez la page [Configuration requise pour ExpressRoute](expressroute-prerequisites.md).
- Configurez votre connexion ExpressRoute.
	- [Création d’un circuit ExpressRoute](expressroute-howto-circuit-classic.md)
	- [Configuration d’un routage (homologations de circuit)](expressroute-howto-routing-classic.md)
	- [Liaison d’un réseau virtuel à un circuit ExpressRoute](expressroute-howto-linkvnet-classic.md)

<!---HONumber=AcomDC_0323_2016-->