<properties 
    pageTitle="Meilleures pratiques pour améliorer les performances à l’aide de Service Bus | Microsoft Azure"
    description="Explique comment utiliser Azure Service Bus pour optimiser les performances lors de l’échange de messages répartis."
    services="service-bus"
    documentationCenter="na"
    authors="sethmanheim"
    manager="timlt"
    editor="" /> 
<tags 
    ms.service="service-bus"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="03/16/2016"
    ms.author="sethm" />

# Meilleures pratiques relatives aux améliorations de performances à l’aide de la messagerie répartie Service Bus

Cette rubrique décrit comment utiliser Azure Service Bus pour optimiser les performances lors de l’échange de messages répartis. La première partie de cette rubrique décrit les différents mécanismes qui sont proposés et qui permettent d’améliorer les performances. La deuxième partie fournit des conseils sur l’utilisation de Service Bus des services visant à offrir des performances optimales dans un scénario donné.

Dans cette rubrique, le terme « client » fait référence à une entité qui accède à Service Bus. Un client peut jouer le rôle d’un expéditeur ou d’un destinataire. Le terme « expéditeur » est utilisé pour un client de file d’attente ou de rubrique Service Bus qui envoie des messages à une file d’attente ou une rubrique Service Bus. Le terme « destinataire » fait référence à un client de file d’attente ou d’abonnement de Bus de Service qui reçoit des messages de la part d’une file d’attente ou d’un abonnement Service Bus.

Les sections suivantes présentent plusieurs concepts Service Bus utiles pour améliorer les performances.

## Protocoles

Service Bus permet aux clients d’envoyer et recevoir des messages via deux protocoles : le protocole client Service Bus et HTTP (REST). Le protocole client Service Bus est plus efficace, car il conserve la connexion au service de Bus des services tant que la structure de messagerie existe. Il permet également le traitement par lot et la lecture anticipée. Le protocole client Service Bus est disponible pour les applications .NET à l’aide de l’API .NET.

Sauf mention explicite, tout le contenu de cette rubrique suppose l’utilisation du protocole client Service Bus.

## Réutilisation de structures et de clients

Des objets client Service Bus, tels que [QueueClient][] ou [MessageSender][], sont créés via un objet [MessagingFactory][], qui fournit également la gestion interne des connexions. Vous ne devez pas fermer les structures de messagerie ou les files d’attente, les rubriques et les clients d’abonnement après avoir envoyé un message, et les recréer lorsque vous envoyez le message suivant. La fermeture d’une structure de messagerie supprime la connexion à Service Bus et une nouvelle connexion est établie au moment de la recréation de la structure. L’établissement d’une connexion est une opération coûteuse, que vous pouvez éviter en réutilisant les objets de structure et client dans plusieurs opérations. Vous pouvez utiliser en toute sécurité l’objet [QueueClient][] pour envoyer des messages à partir de plusieurs threads et d’opérations asynchrones simultanées.

## Opérations simultanées

L’exécution d’une opération (envoi, réception, suppression, etc.) prend un certain temps. Ce temps comprend le traitement de l’opération par le service Service Bus en plus de la latence de la demande et de la réponse. Pour augmenter le nombre d’opérations par période, les opérations doivent s’exécuter simultanément. Pour cela, vous pouvez procéder de plusieurs façons différentes :

-   **Opérations asynchrones** : le client planifie les opérations en effectuant des opérations asynchrones. La requête suivante démarre avant la fin de la demande précédente. Voici un exemple d’opération d’envoi asynchrone :

	```
	BrokeredMessage m1 = new BrokeredMessage(body);
	BrokeredMessage m2 = new BrokeredMessage(body);
	
	Task send1 = queueClient.SendAsync(m1).ContinueWith((t) => 
	  {
	    Console.WriteLine("Sent message #1");
	  });
	Task send2 = queueClient.SendAsync(m2).ContinueWith((t) => 
	  {
	    Console.WriteLine("Sent message #2");
	  });
	Task.WaitAll(send1, send2);
	Console.WriteLine("All messages sent");
	```

	Voici un exemple d’opération de réception asynchrone :
	
	```
	Task receive1 = queueClient.ReceiveAsync().ContinueWith(ProcessReceivedMessage);
	Task receive2 = queueClient.ReceiveAsync().ContinueWith(ProcessReceivedMessage);
	
	Task.WaitAll(receive1, receive2);
	Console.WriteLine("All messages received");
	
	async void ProcessReceivedMessage(Task<BrokeredMessage> t)
	{
	  BrokeredMessage m = t.Result;
	  Console.WriteLine("{0} received", m.Label);
	  await m.CompleteAsync();
	  Console.WriteLine("{0} complete", m.Label);
	}
	```

-   **Plusieurs structures** : tous les clients (expéditeurs et destinataires) sont créés par la même structure partagent une connexion TCP. Le débit maximal de messages est limité par le nombre d’opérations pouvant emprunter cette connexion TCP. Le débit qui peut être obtenu avec une seule structure varie sensiblement en fonction des durées de parcours aller-retour TCP et de la taille des messages. Pour obtenir des débits plus importants, vous devez utiliser plusieurs structures de messagerie.

## Mode de réception

Lorsque vous créez un client de file d’attente ou d’abonnement, vous pouvez spécifier un mode de réception : *verrouillage* ou *recevoir et supprimer*. Le mode par défaut est [Verrouillage][]. Lorsqu’il fonctionne dans ce mode, le client envoie une demande pour recevoir un message de la part de Service Bus. Une fois que le client a reçu le message, il envoie une demande pour compléter le message.

Lors de la définition du mode de réception [ReceiveAndDelete][], les deux étapes sont combinées dans une requête unique. Cela réduit le nombre total d’opérations et peut améliorer le débit global des messages. Ce gain de performances est fourni au risque de perdre des messages.

Service Bus ne gère pas les transactions des opérations de réception-suppression. En outre, la syntaxe de verrouillage est nécessaire pour tous les scénarios dans lesquels le client souhaite différer ou ignorer un message (lettre morte).

## Traitement par lots côté client

Le traitement par lot côté client permet à un client de file d’attente ou de rubrique de retarder l’envoi d’un message pendant une période donnée. Si le client envoie des messages supplémentaires pendant cette période, il transmet les messages dans un seul lot. Le traitement par lots côté client fait également en sorte qu’une file d’attente ou un abonnement client regroupe plusieurs requêtes **complètes** en une seule requête. Le traitement par lots n’est disponible que pour les opérations d’**envoi** et **complètes** asynchrones. Les opérations synchrones sont immédiatement envoyées au service Service Bus. Le traitement par lots ne peut pas concerner des opérations de verrouillage ou de réception, ou plusieurs clients.

Si le lot dépasse le volume de message maximal, le dernier message est supprimé du lot et le client envoie ce dernier immédiatement. Le dernier message devient le premier message du lot suivant. Par défaut, un client utilise un intervalle 20 ms entre les lots. Vous pouvez modifier l’intervalle de traitement par lots en définissant la propriété [BatchFlushInterval][] avant de créer la structure de messagerie. Ce paramètre affecte tous les clients créés par cette structure.

Pour désactiver le traitement par lot, définissez la propriété [BatchFlushInterval][] sur **TimeSpan.Zero**. Par exemple :

```
MessagingFactorySettings mfs = new MessagingFactorySettings();
mfs.TokenProvider = tokenProvider;
mfs.NetMessagingTransportSettings.BatchFlushInterval = TimeSpan.FromSeconds(0.05);
MessagingFactory messagingFactory = MessagingFactory.Create(namespaceUri, mfs);
```

Le traitement par lot n’affecte pas le nombre d’opérations de messagerie facturables et est disponible uniquement pour le protocole client Service Bus. Le protocole HTTP ne prend pas en charge le traitement par lot.

## Accès au dispositif de stockage de traitement par lot

Pour augmenter le débit d’une file d’attente/d’une rubrique/d’un abonnement, Service Bus regroupe plusieurs messages lorsqu’il écrit à son dispositif de stockage en interne. S’il est activé sur une file d’attente ou une rubrique, l’écriture de messages dans le dispositif de stockage est traitée par lot. S’il est activé sur une file d’attente ou un abonnement, l’écriture de messages depuis le dispositif de stockage est traitée par lot. Si l’accès au magasin par lot est activé pour une entité, Service Bus retarde l’opération d’écriture de dispositif de stockage concernant cette entité de 20 ms au maximum. Les opérations de stockage supplémentaires qui se produisent pendant cet intervalle sont ajoutées au lot. L’accès au dispositif de stockage par lot affecte seulement les opérations d’**envoi** et **Complètes** ; les opérations de réception ne sont pas affectées. L’accès au dispositif de stockage est une propriété d’entité. Le traitement par lot se produit sur toutes les entités qui permettent l’accès au stockage par lot.

Lorsque vous créez une file d’attente, une rubrique ou un abonnement, l’accès au stockage par lot est activé par défaut. Pour désactiver l’accès au stockage par lot, définissez la propriété [EnableBatchedOperations][] sur **false** avant de créer l’entité. Par exemple :

```
QueueDescription qd = new QueueDescription();
qd.EnableBatchedOperations = false;
Queue q = namespaceManager.CreateQueue(qd);
```

L’accès au stockage par lot n’affecte pas le nombre d’opérations de messagerie facturables et constitue une propriété de file d’attente, de rubrique ou d’abonnement. Il est indépendant du mode de réception et du protocole utilisé entre un client et le service Service Bus.

## Lecture anticipée

La lecture anticipée permet au client de la file d’attente ou de l’abonnement de charger des messages supplémentaires à partir du service lorsqu’il effectue une opération de réception. Le client stocke ces messages en mémoire cache. La taille du cache est déterminée par les propriétés [QueueClient.PrefetchCount][] et [SubscriptionClient.PrefetchCount][]. Chaque client qui permet la lecture anticipée gère son propre cache. Un cache n’est pas partagé par plusieurs clients. Si le client initie une opération de réception et que sa mémoire cache est vide, le service transmet un lot de messages. La taille du lot est égale à la taille du cache ou à 256 Ko, la plus faible l’emportant. Si le client initie une opération de réception et que le cache contient un message, ce dernier est extrait de la mémoire cache.

Lorsqu’un message est lu par anticipation, le service le verrouille. Ce faisant, le message lu par anticipation ne peut pas être reçu par un autre destinataire. Si le destinataire ne peut pas terminer le message avant expiration du verrouillage, le message devient disponible pour les autres destinataires. La copie lue par anticipation du message reste dans le cache. Le destinataire qui consomme la copie mise en cache expirée reçoit une exception lorsqu’il essaie de terminer le message. Par défaut, le verrouillage du message expire au bout de 60 secondes. Cette valeur peut être étendue à 5 minutes. Pour empêcher la consommation des messages arrivés à expiration, la taille du cache doit toujours être inférieure au nombre de messages qui peuvent être utilisés par un client au sein de l’intervalle de délai d’expiration de verrouillage.

Lorsque vous utilisez l’expiration de verrouillage par défaut (60 secondes), il est judicieux d’attribuer à [SubscriptionClient.PrefetchCount][] une valeur correspondant à 20 fois la vitesse de traitement de l’ensemble des destinataires présents dans la structure. Par exemple, une structure crée 3 destinataires, et chaque récepteur peut traiter jusqu’à 10 messages par seconde. Le nombre de lectures anticipées ne doit pas dépasser 20 * 3 * 10 = 600. Par défaut, [QueueClient.PrefetchCount][] est définie sur 0, ce qui signifie qu’aucun message supplémentaire n’est lu à partir du service.

La lecture anticipée de messages augmente le débit global d’un abonnement ou une file d’attente, car elle permet de réduire le nombre total d’opérations de messagerie, ou les allers-retours. La lecture anticipée du premier message, cependant, prend plus de temps (en raison de la taille du message accrue). La réception de message avec lecture anticipée sera plus rapide, car ces messages ont déjà été téléchargés par le client.

La propriété (TTL) de durée de vie d’un message est vérifiée par le serveur au moment où le serveur envoie le message au client. Le client ne vérifie pas la propriété TTL de durée de vie du message à la réception du message. Toutefois, le message peut être reçu même si la durée de vie du message a été dépassée pensant la mise en mémoire cache par le client.

La lecture anticipée n’affecte pas le nombre d’opérations de messagerie facturables et est disponible uniquement pour le protocole client Service Bus. Le protocole HTTP ne prend pas en charge la lecture anticipée. La lecture anticipée est disponible pour les opérations de réception synchrones et asynchrones.

## Files d’attente et les rubriques rapides

Les entités rapides permettent un débit élevé et réduisent les temps de latence. Avec les entités rapides, si un message est envoyé à une file d’attente ou à une rubrique, il n’est pas immédiatement stocké dans la banque de messagerie. Au lieu de cela, le message est mis en cache. Si un message reste dans la file d’attente pendant plus de quelques secondes, il est automatiquement écrit dans un stockage stable, afin d’éviter qu’il soit perdu en cas de panne. L’écriture du message en mémoire cache permet d’augmenter le débit et de réduire le temps de latence, car il n’a pas accès au stockage stable au moment où que le message est envoyé. Les messages consommés en quelques secondes ne sont pas écrits dans le magasin de messagerie. Dans l’exemple suivant, l’abonnement crée une rubrique rapide.

```
TopicDescription td = new TopicDescription(TopicName);
td.EnableExpress = true;
namespaceManager.CreateTopic(td);
```

Si un message contenant des informations sensibles ne devant pas être égarées est envoyé à une entité rapide, l’expéditeur peut forcer Service Bus à placer immédiatement le message dans un dispositif de stockage stable en définissant le [ForcePersistence][] à la propriété **true**.

## Utilisation de files d’attente partitionnées ou de rubriques

En interne, Service Bus utilise le même nœud et stockage de messagerie pour traiter et stocker tous les messages d’une entité de messagerie (file d’attente ou rubrique). Une file d’attente ou une rubrique partitionnée, elle, se répartit sur plusieurs nœuds et banques de messagerie. Les rubriques et files d’attente partitionnées donnent non seulement un débit plus élevé que les rubriques et files d’attente standard, mais ils présentent également une disponibilité supérieure. Pour créer une entité partitionnée, définissez la propriété [EnablePartitioning][] sur **true**, comme illustré dans l’exemple suivant. Pour plus d’informations sur les entités partitionnées, consultez [Entités de messagerie partitionnées][].

```
// Create partitioned queue.
QueueDescription qd = new QueueDescription(QueueName);
qd.EnablePartitioning = true;
namespaceManager.CreateQueue(qd);
```

## Utilisation de plusieurs files d’attente

S’il est impossible d’utiliser une file d’attente ou une rubrique partitionnée, la charge prévue ne peut pas être gérée par une seule file d’attente ou rubrique partitionnée. Vous devez utiliser plusieurs entités de messagerie. Lorsque vous utilisez plusieurs entités, créez un client dédié pour chacune d’elles au lieu d’utiliser le même client pour toutes les entités.

## Scénarios

Les sections suivantes décrivent les scénarios de messagerie classiques et soulignent les paramètres Service Bus favoris par défaut. Les débits sont classés dans la catégorie Petit (moins d’1 message/seconde), Modéré (1 message/s ou plus, mais moins de 100 messages par seconde) et Élevé (100 messages/seconde ou plus). Le nombre de clients est classé en tant que petit (5 ou moins), modéré (plus de 5, mais inférieur ou égal à 20) et grand (plus de 20).

### File d’attente à débit élevé

Objectif : maximiser le débit d’une file d’attente unique. Le nombre d’expéditeurs et de destinataires est faible.

-   Utilisez une file d’attente partitionnée pour améliorer les performances et la disponibilité.

-   Pour augmenter la vitesse de transmission globale dans la file d’attente, utilisez plusieurs structures de messages pour créer des expéditeurs. Pour chaque expéditeur, utilisez des opérations asynchrones ou plusieurs threads.

-   Pour augmenter la vitesse de réception depuis la file d’attente, utilisez plusieurs structures de messages pour créer des destinataires.

-   Utilisez des opérations asynchrones pour tirer parti du traitement par lot côté client.

-   Définissez l’intervalle de mise en lot de 50 ms pour réduire le nombre de transmissions de protocole client Service Bus. Si plusieurs expéditeurs sont utilisés, augmentez l’intervalle de traitement par lot à 100 ms.

-   Désactivez l’accès au magasin par lot. Cette opération augmente la cadence à laquelle les messages peuvent être écrits dans la file d’attente.

-   Définissez le nombre de lectures anticipées à 20 fois la vitesse de traitement maximale de la totalité des destinataires d’une structure. Cela réduit le nombre de transmissions de protocole client Service Bus.

### Plusieurs files d’attente haut débit

Objectif : Optimiser le débit global de plusieurs files d’attente. Le débit d’une file d’attente individuelle est modéré ou élevé.

Pour obtenir un débit maximal sur plusieurs files d’attente, utiliser les paramètres conseillés pour maximiser le débit d’une file d’attente unique. En outre, utiliser des structures différentes pour créer des clients procédant à des envois ou des réceptions de la part de différentes files d’attente.

### File d’attente à latence faible

Objectif : réduire la latence de bout en bout d’une file d’attente ou d’une rubrique. Le nombre d’expéditeurs et de destinataires est faible. Le débit de la file d’attente est faible ou modéré.

-   Utiliser une file d’attente partitionnée pour améliorer les performances et la disponibilité.

-   Désactiver le traitement par lots côté client. Le client envoie immédiatement un message.

-   Désactiver l’accès au magasin par lot. Le service écrit immédiatement le message pour le magasin.

-   Si vous utilisez un seul client, définissez le nombre de lectures anticipées à 20 fois la vitesse de traitement du destinataire. Si plusieurs messages arrivent à la file d’attente en même temps, le protocole client Service Bus les transmet tous simultanément. Lorsque le client reçoit le message suivant, ce message est déjà dans le cache local. Le cache doit être de petite taille.

-   Si vous utilisez plusieurs clients, définissez le nombre de lectures anticipées sur 0. Ce faisant, le deuxième client peut recevoir le deuxième message alors que le premier client traite toujours le premier message.

### File d’attente comportant un grand nombre d’expéditeurs

Objectif : maximiser le débit d’une file d’attente ou d’une rubrique comportant un grand nombre d’expéditeurs. Chaque expéditeur envoie des messages à une vitesse modérée. Le nombre de destinataires est faible.

Service Bus permet jusqu’à 1 000 connexions simultanées vers une entité de messagerie (ou 5 000 avec AMQP). Cette limite s’applique au niveau de l’espace de noms et les rubriques/files d’attente/abonnements sont limités par la limite de connexions simultanées par espace de noms. Pour les files d’attente, ce nombre est partagé entre les expéditeurs et les destinataires. Si les 1 000 connexions sont requises pour les expéditeurs, vous devez remplacer la file d’attente par une rubrique et un seul abonnement. Une rubrique accepte jusqu’à 1 000 connexions simultanées provenant d’expéditeurs, alors que l’abonnement accepte un 1 000 connexions simultanées destinataires. Si plus de 1 000 expéditeurs simultanés sont requis, les expéditeurs doivent envoyer leurs messages vers le protocole de Service Bus via HTTP.

Pour maximiser le débit, procédez comme suit :

-   Utilisez une file d’attente partitionnée pour améliorer les performances et la disponibilité.

-   Si chaque expéditeur réside dans un processus différent, utilisez uniquement une structure par processus.

-   Utilisez des opérations asynchrones pour tirer parti du traitement par lot côté client.

-   Utilisez la valeur par défaut de l’intervalle de 20 ms pour réduire le nombre de transmissions de protocole client Service Bus.

-   Désactivez l’accès au magasin par lot. Cette opération augmente la cadence à laquelle les messages peuvent être écrits dans la file d’attente ou la rubrique.

-   Définissez le nombre de lectures anticipées à 20 fois la vitesse de traitement maximale de la totalité des destinataires d’une structure. Cela réduit le nombre de transmissions de protocole client Service Bus.

### File d’attente comportant un grand nombre de destinataires

Objectif : optimiser la vitesse de réception d’une file d’attente ou d’abonnement comportant un grand nombre de destinataires. Chaque destinataire reçoit les messages à une vitesse modérée. Le nombre d’expéditeurs est faible.

Service Bus permet jusqu’à 1 000 connexions simultanées vers une entité. Si une file d’attente nécessite plus de 1 000 récepteurs, vous devez remplacer la file d’attente par une rubrique et plusieurs abonnements. Chaque abonnement peut prendre en charge jusqu’à 1 000 connexions simultanées. Les destinataires peuvent également accéder à la file d’attente via le protocole HTTP.

Pour maximiser le débit, procédez comme suit :

-   Utilisez une file d’attente partitionnée pour améliorer les performances et la disponibilité.

-   Si chaque destinataire réside dans un processus différent, utilisez uniquement une structure par processus.

-   Les récepteurs peuvent utiliser des opérations synchronisées ou asynchrones. Étant donné la vitesse de réception modérée d’un destinataire individuel, le traitement côté client de la demande complète n’affecte pas le débit du destinataire.

-   Désactivez l’accès au magasin par lot. Cela réduit la charge globale de l’entité. Cette opération réduit également la cadence générale à laquelle les messages peuvent être écrits dans la file d’attente ou la rubrique.

-   Définir le nombre de lectures anticipées sur une valeur faible (par exemple, PrefetchCount = 10). Cela empêche les destinataires de rester oisifs pendant que les autres destinataires mettent en cache un grand nombre de messages.

### Rubrique comportant un petit nombre d’abonnements

Objectif : maximiser le débit d’une rubrique comportant un petit nombre d’abonnements. Un message est reçu par un grand nombre d’abonnements, ce qui signifie que la vitesse de réception combinée sur l’ensemble des abonnements est supérieure à la vitesse d’envoi. Le nombre d’expéditeurs est faible. Le nombre de récepteurs par abonnement est faible.

Pour maximiser le débit, procédez comme suit :

-   Utilisez une rubrique partitionnée pour améliorer les performances et la disponibilité.

-   Pour augmenter la vitesse de transmission globale dans la rubrique, utilisez plusieurs structures de messages pour créer des expéditeurs. Pour chaque expéditeur, utilisez des opérations asynchrones ou plusieurs threads.

-   Pour augmenter la vitesse de réception d’ensemble d’un abonnement, utilisez plusieurs structures de message pour créer des destinataires. Pour chaque destinataire, utilisez des opérations asynchrones ou plusieurs threads.

-   Utilisez des opérations asynchrones pour tirer parti du traitement par lot côté client.

-   Utilisez la valeur par défaut de l’intervalle de 20 ms pour réduire le nombre de transmissions de protocole client Service Bus.

-   Désactivez l’accès au magasin par lot. Cette opération augmente la cadence à laquelle les messages peuvent être écrits dans la rubrique.

-   Définissez le nombre de lectures anticipées à 20 fois la vitesse de traitement maximale de la totalité des destinataires d’une structure. Cela réduit le nombre de transmissions de protocole client Service Bus.

### Rubrique comportant un grand nombre d’abonnements

Objectif : maximiser le débit d’une rubrique comportant un grand nombre d’abonnements. Un message est reçu par un grand nombre d’abonnements, ce qui signifie que la vitesse de réception associée à l’ensemble des abonnements est supérieure à la vitesse d’envoi. Le nombre d’expéditeurs est faible. Le nombre de récepteurs par abonnement est faible.

Les rubriques comportant un grand nombre d’abonnements affichent généralement un faible débit global si tous les messages sont acheminés vers tous les abonnements. Cela est dû au fait que chaque message est reçu plusieurs fois et tous les messages contenus dans une rubrique et tous les abonnements associés sont stockés dans le même magasin. Il est supposé que le nombre d’expéditeurs et nombre de récepteurs par abonnement est faible. Service Bus prend en charge jusqu’à 2 000 abonnements par rubrique.

Pour maximiser le débit, procédez comme suit :

-   Utilisez une rubrique partitionnée pour améliorer les performances et la disponibilité.

-   Utilisez des opérations asynchrones pour tirer parti du traitement par lot côté client.

-   Utilisez la valeur par défaut de l’intervalle de 20 ms pour réduire le nombre de transmissions de protocole client Service Bus.

-   Désactivez l’accès au magasin par lot. Cette opération augmente la cadence à laquelle les messages peuvent être écrits dans la rubrique.

-   Définissez le nombre de lectures anticipées à 20 fois la vitesse de réception prévue en secondes. Cela réduit le nombre de transmissions de protocole client Service Bus.

## Étapes suivantes

Pour en savoir plus sur l’optimisation des performances Service Bus, consultez [Entités de messagerie partitionnées][].

  [QueueClient]: https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.queueclient.aspx
  [MessageSender]: https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.messagesender.aspx
  [MessagingFactory]: https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.messagingfactory.aspx
  [Verrouillage]: https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.receivemode.aspx
  [ReceiveAndDelete]: https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.receivemode.aspx
  [BatchFlushInterval]: https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.netmessagingtransportsettings.batchflushinterval.aspx
  [EnableBatchedOperations]: https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.queuedescription.enablebatchedoperations.aspx
  [QueueClient.PrefetchCount]: https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.queueclient.prefetchcount.aspx
  [SubscriptionClient.PrefetchCount]: https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.subscriptionclient.prefetchcount.aspx
  [ForcePersistence]: https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.brokeredmessage.forcepersistence.aspx
  [EnablePartitioning]: https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.queuedescription.enablepartitioning.aspx
  [Entités de messagerie partitionnées]: service-bus-partitioning.md
  

<!---HONumber=AcomDC_0323_2016-->