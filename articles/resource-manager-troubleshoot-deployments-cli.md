<properties
   pageTitle="Résolution des problèmes liés aux déploiements avec l’interface de ligne de commande Azure | Microsoft Azure"
   description="Décrit comment utiliser l'interface de ligne de commande Azure pour détecter et résoudre les problèmes de déploiement du Gestionnaire de ressources."
   services="azure-resource-manager,virtual-machines"
   documentationCenter=""
   tags="top-support-issue"
   authors="tfitzmac"
   manager="timlt"
   editor=""/>

<tags
   ms.service="azure-resource-manager"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="vm-multiple"
   ms.workload="infrastructure"
   ms.date="03/21/2016"
   ms.author="tomfitz"/>

# Résolution des problèmes liés aux déploiements de groupes de ressources avec l’interface de ligne de commande Azure

> [AZURE.SELECTOR]
- [Portail](resource-manager-troubleshoot-deployments-portal.md)
- [PowerShell](resource-manager-troubleshoot-deployments-powershell.md)
- [Interface de ligne de commande Azure](resource-manager-troubleshoot-deployments-cli.md)
- [API REST](resource-manager-troubleshoot-deployments-rest.md)

Si vous avez reçu une erreur lors du déploiement des ressources sur Azure, vous devez résoudre le problème. L’interface de ligne de commande Azure fournit les commandes qui vous permettent de rechercher les erreurs et de déterminer les corrections éventuelles.

Vous pouvez dépanner votre déploiement en examinant les journaux d'audit ou les opérations de déploiement. Cette rubrique illustre les deux méthodes.

Vous pouvez éviter certaines erreurs en validant votre modèle et votre infrastructure avant le déploiement. Pour plus d’informations, consultez la page [Déploiement d’un groupe de ressources avec un modèle Azure Resource Manager](resource-group-template-deploy.md).

## Utilisation des journaux d'audit pour résoudre les problèmes

[AZURE.INCLUDE [resource-manager-audit-limitations](../includes/resource-manager-audit-limitations.md)]

Pour afficher les erreurs d’un déploiement, procédez comme suit :

1. Pour afficher les journaux d'audit, exécutez la commande **azure group log show**. Vous pouvez utiliser l'option **--last-deployment** pour récupérer uniquement le journal du déploiement le plus récent.

        azure group log show ExampleGroup --last-deployment

2. La commande **azure group log show** risque de renvoyer un grand nombre d'informations. Pour le dépannage, vous devez généralement vous concentrer sur les opérations ayant échoué. Le script suivant utilise l’option **--json** et l’utilitaire JSON [jq](https://stedolan.github.io/jq/) pour rechercher les échecs de déploiement dans le journal.

        azure group log show ExampleGroup --json | jq '.[] | select(.status.value == "Failed")'
        
        {
        "claims": {
          "aud": "https://management.core.windows.net/",
          "iss": "https://sts.windows.net/<guid>/",
          "iat": "1442510510",
          "nbf": "1442510510",
          "exp": "1442514410",
          "ver": "1.0",
          "http://schemas.microsoft.com/identity/claims/tenantid": "{guid}",
          "http://schemas.microsoft.com/identity/claims/objectidentifier": "{guid}",
          "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress": "someone@example.com",
          "puid": "XXXXXXXXXXXXXXXX",
          "http://schemas.microsoft.com/identity/claims/identityprovider": "example.com",
          "altsecid": "1:example.com:XXXXXXXXXXX",
          "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/nameidentifier": "<hash string="">",
          "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname": "Tom",
          "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/surname": "FitzMacken",
          "name": "Tom FitzMacken",
          "http://schemas.microsoft.com/claims/authnmethodsreferences": "pwd",
          "groups": "{guid}",
          "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name": "example.com#someone@example.com",
          "wids": "{guid}",
          "appid": "{guid}",
          "appidacr": "0",
          "http://schemas.microsoft.com/identity/claims/scope": "user_impersonation",
          "http://schemas.microsoft.com/claims/authnclassreference": "1",
          "ipaddr": "000.000.000.000"
        },
        "properties": {
          "statusCode": "Conflict",
          "statusMessage": "{"Code":"Conflict","Message":"Website with given name mysite already exists.","Target":null,"Details":[{"Message":"Website with given name
            mysite already exists."},{"Code":"Conflict"},{"ErrorEntity":{"Code":"Conflict","Message":"Website with given name mysite already exists.","ExtendedCode":
            "54001","MessageTemplate":"Website with given name {0} already exists.","Parameters":["mysite"],"InnerErrors":null}}],"Innererror":null}"
        },
        ...

    Vous pouvez constater que la section **properties** inclut des informations concernant l'opération ayant échoué dans json.

    Vous pouvez utiliser les options **--verbose** et **- vv** pour afficher plus d'informations des journaux. Utilisez l'option **--verbose** pour afficher les étapes effectuées par les opérations dans `stdout`. Pour obtenir un historique complet des demandes, utilisez l'option **- vv**. Souvent, ces messages fournissent des indices essentiels sur les causes d'erreurs.

3. Pour vous concentrer sur le message d'état des entrées en échec, utilisez la commande suivante :

        azure group log show ExampleGroup --json | jq -r ".[] | select(.status.value == "Failed") | .properties.statusMessage"


## Utilisation des opérations de déploiement pour résoudre les problèmes

1. Récupérez l'état global d'un déploiement avec la commande **azure group deployment show**. Dans l'exemple ci-dessous, le déploiement a échoué.

        azure group deployment show --resource-group ExampleGroup --name ExampleDeployment
        
        info:    Executing command group deployment show
        + Getting deployments
        data:    DeploymentName     : ExampleDeployment
        data:    ResourceGroupName  : ExampleGroup
        data:    ProvisioningState  : Failed
        data:    Timestamp          : 2015-08-27T20:03:34.9178576Z
        data:    Mode               : Incremental
        data:    Name             Type    Value
        data:    ---------------  ------  ------------
        data:    siteName         String  ExampleSite
        data:    hostingPlanName  String  ExamplePlan
        data:    siteLocation     String  West US
        data:    sku              String  Free
        data:    workerSize       String  0
        info:    group deployment show command OK

2. Pour afficher le message des opérations ayant échoué pour un déploiement, utilisez :

        azure group deployment operation list --resource-group ExampleGroup --name ExampleDeployment --json  | jq ".[] | select(.properties.provisioningState == "Failed") | .properties.statusMessage.Message"


## Étapes suivantes

- Pour en savoir plus sur l'utilisation des journaux d'audit pour surveiller d'autres types d'actions, consultez [Auditer les opérations avec le Gestionnaire de ressources](resource-group-audit.md).
- Pour valider votre déploiement avant son exécution, consultez [Déployer un groupe de ressources avec le modèle Azure Resource Manager](resource-group-template-deploy.md).

<!---HONumber=AcomDC_0323_2016-->