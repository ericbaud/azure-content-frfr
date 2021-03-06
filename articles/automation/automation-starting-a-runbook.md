<properties
   pageTitle="Démarrage d’un Runbook dans Azure Automation| Microsoft Azure"
   description="Résume les différentes méthodes qui peuvent être utilisées pour démarrer un Runbook dans Azure Automation et fournit des détails sur l'utilisation du portail Azure et de Windows PowerShell."
   services="automation"
   documentationCenter=""
   authors="mgoedtel"
   manager="stevenka"
   editor="tysonn" />
<tags 
   ms.service="automation"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="infrastructure-services"
   ms.date="02/23/2016"
   ms.author="magoedte;bwren"/>

# Démarrage d'un Runbook dans Azure Automation

Le tableau suivant vous aide à déterminer la méthode de démarrage d'un Runbook dans Azure Automation la plus appropriée à votre scénario. Cet article inclut des informations détaillées sur le démarrage d'un Runbook avec le portail Azure et avec Windows PowerShell. Des informations supplémentaires sur les autres méthodes sont fournies dans différentes documentations accessibles depuis les liens ci-dessous.

| **MÉTHODE** | **CARACTÉRISTIQUES** |
|-------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| [Portail Azure](#starting-a-runbook-with-the-azure-portal) | <li>Méthode la plus simple avec une interface utilisateur interactive.<br> <li>Formulaire pour fournir les valeurs de paramètres simples.<br> <li>Suivi aisé de l'état des tâches.<br> <li>Accès authentifié avec ouverture de session Azure. |
| [Windows PowerShell](https://msdn.microsoft.com/library/dn690259.aspx) | <li>Appel à partir de la ligne de commande avec les applets de commande Windows PowerShell.<br> <li>Possibilité d'inclusion dans une solution automatisée à plusieurs étapes.<br> <li>Demande authentifiée avec un certificat ou un principal du service / principal d'utilisateur OAuth.<br> <li>Fourniture de valeurs de paramètres simples et complexes.<br> <li>Suivi de l'état des tâches.<br> <li>Client requis pour prendre en charge les applets de commande PowerShell. |
| [API Azure Automation](http://msdn.microsoft.com/library/azure/mt163849.aspx) | <li>Méthode la plus souple, mais également la plus complexe.<br> <li>Appel à partir de n'importe quel code personnalisé qui peut envoyer des requêtes HTTP.<br> <li>Requête authentifiée avec un certificat ou un principal du service / principal d'utilisateur Oauth.<br> <li>Fourniture de valeurs de paramètres simples et complexes.<br> <li>Suivi de l'état des tâches. |
| [Webhooks](automation-webhooks.md) | <li>Démarrage d'un Runbook à partir d'une simple requête HTTP.<br> <li>Authentification avec un jeton de sécurité dans l'URL.<br> <li>Le client ne peut pas remplacer les valeurs de paramètre spécifiées lors de la création du Webhook. Le Runbook peut définir un paramètre unique qui est rempli avec les détails de la requête HTTP.<br> <li>Aucune possibilité de suivre l'état des tâches via l'URL du Webhook. |
| [Répondre à une alerte Azure](automation-webhooks.md) | <li>Démarrer un Runbook en réponse à une alerte Azure.<br> <li>Configurer webhook pour Runbook et lien vers l'alerte.<br> <li>Authentification avec un jeton de sécurité dans l'URL.<br> <li>Actuellement, prend en charge l'alerte sur Metrics uniquement. |
| [Planification](automation-scheduling-a-runbook.md) | <li>Démarrage automatique du Runbook selon une planification horaire, quotidienne ou hebdomadaire.<br> <li>Manipulation de la planification via le portail Azure, les applets de commande PowerShell ou les API Azure.<br> <li>Fourniture des valeurs de paramètres à utiliser avec la planification. |
| [À partir d'un autre Runbook](automation-child-runbooks.md) | <li>Utilisation d'un Runbook en tant qu'activité d'un autre Runbook.<br> <li>Utile pour les fonctionnalités utilisées par plusieurs Runbooks.<br> <li>Fourniture des valeurs de paramètres au Runbook enfant et utilisation de la sortie dans le Runbook parent. |

L’image suivante illustre le processus détaillé du cycle de vie d’un runbook. Elle présente différentes méthodes de démarrage d’un runbook dans Azure Automation, décrit les composants requis pour permettre à un ordinateur local d’exécuter les runbooks Azure Automation, et illustre les interactions entre les différents composants. Pour en savoir plus sur l’exécution des runbooks Automation dans votre centre de données, consultez l’article [Runbooks Workers hybrides Azure Automation](automation-hybrid-runbook-worker.md)

![Architecture de runbook](media/automation-starting-runbook/runbooks-architecture.png)

## Démarrage d'un Runbook avec le portail Azure

1.	Dans le portail Azure, sélectionnez **Automation**, puis cliquez sur le nom d'un compte Automation.
2.	Sélectionnez l'onglet **Runbooks**.
3.	Sélectionnez un Runbook, puis cliquez sur **Démarrer**.
4.	Si le Runbook possède des paramètres, vous devez fournir des valeurs avec une zone de texte pour chaque paramètre. Pour plus d'informations sur les paramètres, consultez [Paramètres du Runbook](#Runbook-parameters) ci-dessous.
5.	Sélectionnez **Afficher la tâche** à côté du message du Runbook **Démarrage** ou sélectionnez l'onglet **Tâches** pour que le Runbook affiche l'état de la tâche du Runbook.

## Démarrage d'un Runbook avec le portail Azure

1.	À partir de votre compte Automation, cliquez sur la partie **Runbooks** afin d'ouvrir le panneau **Runbooks**.
2.	Cliquez sur un Runbook pour ouvrir son panneau **Runbook**.
3.	Cliquez sur **Start**.
4.	Si le Runbook n'a aucun paramètre, vous devez confirmer si vous souhaitez le démarrer. Si le Runbook possède des paramètres, le panneau **Démarrer le Runbook** s'ouvre pour vous permettre de fournir les valeurs des paramètres. Pour plus d'informations sur les paramètres, consultez [Paramètres du Runbook](#Runbook-parameters) ci-dessous.
5.	Le panneau **Tâche** s'ouvre afin que vous puissiez suivre l'état de la tâche.

## Démarrage d'un Runbook avec Windows PowerShell

Vous pouvez utiliser l'applet de commande [Start-AzureAutomationRunbook](http://msdn.microsoft.com/library/azure/dn690259.aspx) pour démarrer un Runbook avec Windows PowerShell. L'exemple de code suivant démarre un Runbook appelé Test-Runbook.

```
Start-AzureAutomationRunbook –AutomationAccountName "MyAutomationAccount" –Name "Test-Runbook"
```

Start-AzureAutomationRunbook retourne un objet de traitement que vous pouvez utiliser pour suivre son état une fois que le Runbook a démarré. Vous pouvez ensuite utiliser cet objet de traitement avec [Get-AzureAutomationJob](http://msdn.microsoft.com/library/azure/dn690263.aspx) pour déterminer l'état de la tâche et avec [Get-AzureAutomationJobOutput](http://msdn.microsoft.com/library/azure/dn690268.aspx) pour obtenir sa sortie. L'exemple de code suivant démarre un Runbook appelé Test-Runbook, attend qu'il ait terminé, puis affiche sa sortie.

```
$job = Start-AzureAutomationRunbook –AutomationAccountName "MyAutomationAccount" –Name "Test-Runbook"

$doLoop = $true
While ($doLoop) {
   $job = Get-AzureAutomationJob –AutomationAccountName "MyAutomationAccount" -Id $job.Id
   $status = $job.Status
   $doLoop = (($status -ne "Completed") -and ($status -ne "Failed") -and ($status -ne "Suspended") -and ($status -ne "Stopped")
}

Get-AzureAutomationJobOutput –AutomationAccountName "MyAutomationAccount" -Id $job.Id –Stream Output
```

Si le Runbook requiert des paramètres, vous devez les fournir comme [table de hachage](http://technet.microsoft.com/library/hh847780.aspx) où la clé de la table de hachage correspond au nom de paramètre et la valeur à la valeur du paramètre. L'exemple suivant montre comment démarrer un Runbook avec deux paramètres de chaîne nommés FirstName et LastName, un entier nommé RepeatCount et un paramètre booléen nommé Show. Pour plus d'informations sur les paramètres, consultez [Paramètres du Runbook](#Runbook-parameters).

```
$params = @{"FirstName"="Joe";"LastName"="Smith";"RepeatCount"=2;"Show"=$true}
Start-AzureAutomationRunbook –AutomationAccountName "MyAutomationAccount" –Name "Test-Runbook" –Parameters $params
```

## Paramètres du Runbook

Lorsque vous démarrez un Runbook à l'aide du portail de gestion Azure ou de Windows PowerShell, l'instruction est envoyée via le service web Azure Automation. Ce service ne prend pas en charge les paramètres avec des types de données complexes. Si vous devez fournir une valeur pour un paramètre complexe, vous devez l’appeler en ligne à partir d’un autre Runbook, comme décrit dans [Runbooks enfants dans Azure Automation](automation-child-runbooks.md).

Le service web Azure Automation fournit des fonctionnalités spécifiques pour les paramètres en utilisant certains types de données comme décrit dans les sections suivantes.

### Valeurs nommées

Si le paramètre est un type de données [object], vous pouvez utiliser le format JSON suivant pour lui envoyer une liste de valeurs nommées : *{"Nom1":Valeur1, "Nom2":Valeur2, "Nom3":Valeur3}*. Ces valeurs doivent avoir des types simples. Le Runbook reçoit le paramètre comme [PSCustomObject](http://msdn.microsoft.com/library/azure/system.management.automation.pscustomobject(v=vs.85).aspx) avec des propriétés correspondant à chaque valeur nommée.

Considérez le Runbook de test suivant qui accepte un paramètre nommé user.

```
Workflow Test-Parameters
{
   param (
      [Parameter(Mandatory=$true)][object]$user
   )
    if ($user.Show) {
        foreach ($i in 1..$user.RepeatCount) {
            $user.FirstName
            $user.LastName
        }
    }
}
```

Le texte suivant peut être utilisé pour le paramètre user.

```
{"FirstName":"Joe","LastName":"Smith","RepeatCount":2,"Show":true}
```

Il s'ensuit la sortie suivante.

```
Joe
Smith
Joe
Smith
```

### Tableaux

Si le paramètre est un tableau comme [array] ou [string], vous pouvez utiliser le format JSON suivant pour lui envoyer une liste de valeurs : *[Valeur1,Valeur2,Valeur3]*. Ces valeurs doivent avoir des types simples.

Considérez le Runbook de test suivant qui accepte un paramètre nommé *user*.

```
Workflow Test-Parameters
{
   param (
      [Parameter(Mandatory=$true)][array]$user
   )
    if ($user[3]) {
        foreach ($i in 1..$user[2]) {
            $ user[0]
            $ user[1]
        }
    }
}
```

Le texte suivant peut être utilisé pour le paramètre user.

```
["Joe","Smith",2,true]
```

Il s'ensuit la sortie suivante.

```
Joe
Smith
Joe
Smith
```

### Informations d'identification

Si le paramètre est un type de données **PSCredential**, vous pouvez fournir le nom d'une [ressource d'informations d'identification](automation-credentials.md) Azure Automation. Le Runbook récupère la ressource d'informations d'identification portant le nom que vous spécifiez.

Considérez le Runbook de test suivant qui accepte un paramètre nommé credential.

```
Workflow Test-Parameters
{
   param (
      [Parameter(Mandatory=$true)][PSCredential]$credential
   )
   $credential.UserName
}
```

Le texte suivant peut être utilisé pour le paramètre user en supposant qu'il existe une ressource d'informations d'identification nommée *My Credential*.

```
My Credential
```

En supposant que le nom d'utilisateur des informations d'identification soit *jsmith*, il s'ensuit le résultat suivant.

```
jsmith
```

## Étapes suivantes

-	L’architecture de runbook présentée dans cet article fournit une description détaillée des runbooks hybrides. Pour plus d’informations, consultez [Runbooks enfants dans Azure Automation](automation-child-runbooks.md)

<!---HONumber=AcomDC_0302_2016-->