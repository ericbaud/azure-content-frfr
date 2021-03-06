<properties
pageTitle="Utiliser l'API Azure Service Bus dans vos applications logiques | Microsoft Azure"
description="Prendre en main l’utilisation de l’API Azure Service Bus (connecteur) dans vos applications logiques Microsoft Azure App Service"
services=""	
documentationCenter="" 	
authors="msftman"	
manager="erikre"	
editor=""
tags="connectors"/>

<tags
ms.service="multiple"
ms.devlang="na"
ms.topic="article"
ms.tgt_pltfrm="na"
ms.workload="na"
ms.date="03/16/2016"
ms.author="deonhe"/>

# Prise en main de l’API Azure Service Bus

Connectez-vous à Azure Service Bus pour envoyer et recevoir des messages. Vous pouvez effectuer des actions comme envoyer vers une file d'attente, envoyer vers une rubrique, recevoir d’une file d'attente, recevoir d'un abonnement, etc.

>[AZURE.NOTE] Cette version de l'article s'applique à la version de schéma 2015-08-01-preview des applications logiques. Pour la version du schéma 2014-12-01-preview, cliquez sur [Azure Service Bus](../app-service-logic/app-service-logic-connector-azureservicebus.md).

Avec Azure Service Bus, vous pouvez :

* Créer des applications logiques  

Pour ajouter une opération aux applications logiques, consultez [Créer une application logique](../app-service-logic/app-service-logic-create-a-logic-app.md).

## À propos des déclencheurs et des actions

L'API Azure Service Bus peut être utilisée en tant qu'action ; elle possède un ou plusieurs déclencheurs. Toutes les API prennent en charge les données aux formats JSON et XML.

 L'API Azure Service Bus met à votre disposition les actions et/ou les déclencheurs ci-après.

### Actions Azure Service Bus
Vous pouvez effectuer les actions suivantes :

|Action|Description|
|--- | ---|
|SendMessage|Envoie un message à la file d'attente Azure Service Bus ou à une rubrique.|
### Déclencheurs Azure Service Bus
Vous pouvez écouter les événements suivants :

|Déclencheur | Description|
|--- | ---|
|GetMessageFromQueue|Obtient un nouveau message de la file d'attente Azure Service Bus.|
|GetMessageFromTopic|Obtient un nouveau message de l'abonnement à la rubrique Azure Service Bus.|


## Créer une connexion à Azure Service Bus
Pour utiliser l'API Azure Service Bus, vous devez créer une **connexion**, puis fournir les détails de ces propriétés :

|Propriété| Requis|Description|
| ---|---|---|
|ConnectionString|Oui|Fournir une chaîne de connexion Azure Service Bus|  

Suivez ces étapes pour créer une **connexion** à Service Bus que vous pourrez ensuite utiliser dans votre application logique :

1. Sélectionnez **Périodicité**
2. Sélectionnez une **Fréquence** et entrez un **Intervalle** ![Configurer Service Bus][1] 
3. Sélectionnez **Ajouter une action** ![Configurer Service Bus][2]   
4. Entrez **Service Bus** dans la zone de recherche et attendez que la recherche renvoie toutes les entrées contenant Service Bus dans leur nom
5. Sélectionnez **Service Bus - Envoyer un message** ![Configurer Service Bus][3]
7. Entrez un **nom de connexion** et une **chaîne de connexion**, puis sélectionnez **Créer une connexion** : ![Configurer Service Bus][4]
7. Une fois la connexion créée, la boîte de dialogue **Envoyer un message** s’affiche. Entrez les informations nécessaires pour envoyer un message. ![Configurer Service Bus][5]
8. Utilisez le bouton **Enregistrer** dans le menu supérieur pour enregistrer votre travail.    

>[AZURE.TIP] Vous pouvez utiliser cette connexion dans d'autres applications logiques.

## Référence de l’API REST d’Azure Service Bus
#### Cette documentation concerne la version 1.0.


### Envoie un message à la file d'attente Azure Service Bus ou à une rubrique.
**```POST: /{entityName}/messages```**



| Nom| Type de données|Requis|Emplacement|Valeur par défaut|Description|
| ---|---|---|---|---|---|
|message| |yes|body|(aucun)|Message Service Bus|
|entityName|string|yes|path|(aucun)|Nom de la file d’attente ou de la rubrique|


### Voici les réponses possibles :

|Nom|Description|
|---|---|
|200|OK|
|default|L’opération a échoué.|
------



### Obtient un nouveau message de la file d'attente Azure Service Bus.
**```GET: /{queueName}/messages/head```**



| Nom| Type de données|Requis|Emplacement|Valeur par défaut|Description|
| ---|---|---|---|---|---|
|queueName|string|yes|path|(aucun)|Nom de la file d’attente.|


### Voici les réponses possibles :

|Nom|Description|
|---|---|
|200|OK|
|default|L’opération a échoué.|
------



### Obtient un nouveau message de l'abonnement à la rubrique Azure Service Bus.
**```GET: /{topicName}/subscriptions/{subscriptionName}/messages/head```**



| Nom| Type de données|Requis|Emplacement|Valeur par défaut|Description|
| ---|---|---|---|---|---|
|topicName|string|yes|path|(aucun)|Nom de la rubrique.|
|subscriptionName|string|yes|path|(aucun)|Nom de l'abonnement de rubrique.|


### Voici les réponses possibles :

|Nom|Description|
|---|---|
|200|OK|
|default|L’opération a échoué.|
------



## Définition(s) d'objet : 

 **ServiceBusMessage** : message composé d’un contenu et de propriétés

Propriétés requises pour ServiceBusMessage :

ContentTransferEncoding

**Toutes les propriétés** :


| Nom | Type de données |
|---|---|
|ContentData|string|
|ContentType|string|
|ContentTransferEncoding|string|
|Propriétés|objet|


## Étapes suivantes
[Créer une application logique](../app-service-logic/app-service-logic-create-a-logic-app.md).

[1]: ./media/connectors-create-api-servicebus/connectionconfig1.png
[2]: ./media/connectors-create-api-servicebus/connectionconfig2.png
[3]: ./media/connectors-create-api-servicebus/connectionconfig3.png
[4]: ./media/connectors-create-api-servicebus/connectionconfig4.png
[5]: ./media/connectors-create-api-servicebus/connectionconfig5.png
[6]: ./media/connectors-create-api-servicebus/connectionconfig6.png

<!---HONumber=AcomDC_0323_2016-->