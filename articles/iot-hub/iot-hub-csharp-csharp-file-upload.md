<properties
	pageTitle="Télécharger des fichiers à partir d’appareils à l’aide d’IoT Hub | Microsoft Azure"
	description="Suivez ce didacticiel pour découvrir comment télécharger des fichiers à partir d’appareils à l’aide d’Azure IoT Hub avec C#."
	services="iot-hub"
	documentationCenter=".net"
	authors="fsautomata"
	manager="timlt"
	editor=""/>

<tags
     ms.service="iot-hub"
     ms.devlang="dotnet"
     ms.topic="article"
     ms.tgt_pltfrm="na"
     ms.workload="na"
     ms.date="02/03/2016"
     ms.author="elioda"/>

# Didacticiel : télécharger des fichiers à partir d’appareils vers le cloud avec IoT Hub

## Introduction

Azure IoT Hub est un service entièrement géré qui autorise des communications bidirectionnelles fiables et sécurisées entre des millions d’appareils IoT et un serveur d’applications principal. Les didacticiels précédents ([Prise en main d’IoT Hub] et [Envoyer des messages cloud-à-appareil avec IoT Hub]) illustrent la fonctionnalité de base des messages appareil-à-cloud et cloud-à-appareil offerte par IoT Hub et comment y accéder à partir d’appareils et composants cloud. [Traiter les messages appareil-à-cloud] décrit un moyen de stocker en toute fiabilité les messages appareil-à-cloud dans le stockage d’objets blob Azure. Toutefois, dans certains cas les données provenant des appareils ne mappent pas facilement à des messages appareil-à-cloud relativement petits. Il s’agit par exemple de fichiers volumineux contenant des images, des vidéos, des exemples de données de vibration à haute fréquence ou contenant une forme quelconque de données prétraitées. Ces fichiers sont généralement traités par lot à l’aide d’outils tels que [Azure Data Factory] ou de la pile [Hadoop]. Quand le téléchargement d’un fichier à partir d’un appareil est préférable à l’envoi d’événements, il est toujours possible d’utiliser les fonctionnalités de fiabilité et de sécurité offertes par IoT Hub.

Ce didacticiel s’appuie sur le code présenté dans [Envoyer des messages cloud-à-appareil avec IoT Hub] pour montrer comment utiliser des messages cloud-à-appareil pour fournir à l’appareil, de façon sécurisée, un URI d’objet blob Azure à utiliser pour télécharger le fichier, et illustre comment utiliser des accusés de réception IoT Hub pour déclencher le traitement du fichier à partir de votre serveur d’applications principal. Les avantages de cette approche sont la réutilisation de l’identité de l’appareil IoT Hub et de l’accusé de réception des messages cloud-à-appareil pour signaler au serveur d’applications principal que le fichier a été téléchargé correctement.

> [AZURE.NOTE] Vous pouvez adopter la même approche pour faire en sorte que les appareils téléchargent des fichiers à partir du cloud en toute sécurité.

Vous trouverez des informations supplémentaires sur les messages cloud-à-appareil et sur la sécurité IoT Hub dans le [Guide du développeur d’IoT Hub].

À la fin de ce didacticiel, vous exécuterez deux applications de console Windows :

* **SimulatedDevice**, une version modifiée de l’application créée dans [Envoyer des messages cloud-à-appareil avec IoT Hub] qui se connecte à votre hub IoT et reçoit les messages cloud-à-appareil contenant des URI d’objets blob Azure. Pour chaque message cloud-à-appareil reçu, elle déclenche un téléchargement de fichier vers l’URI d’objet blob spécifié.
* **SendCloudToDevice**, qui crée un URI d’objet blob Azure (comme expliqué dans [Création et utilisation d’une signature d’accès partagé avec le service BLOB](../storage/storage-dotnet-shared-access-signature-part-2.md), l’envoie dans un message cloud-à-appareil simulé via IoT Hub et reçoit ensuite l’accusé de réception.

> [AZURE.NOTE] IoT Hub offre la prise en charge de plusieurs plateformes d’appareils et plusieurs langages (y compris C, Java et Javascript) via les Kits Azure IoT device SDK. Pour obtenir des instructions étape par étape expliquant comment connecter votre appareil au code de ce didacticiel, et généralement à Azure IoT Hub, consultez le [Centre de développement Azure IoT]. Les Kits Azure IoT service SDK pour Java et Node seront publiés prochainement.

Pour suivre ce didacticiel, vous avez besoin des éléments suivants :

+ Microsoft Visual Studio 2015

+ Un compte Azure actif. <br/>Si vous n’avez pas de compte, vous pouvez créer un compte gratuit en quelques minutes. Pour plus d’informations, consultez la page d’[essai gratuit d’Azure](https://azure.microsoft.com/pricing/free-trial/?WT.mc_id=A0E0E5C02&amp;returnurl=http%3A%2F%2Fazure.microsoft.com%2Ffr-FR%2Fdevelop%2Fiot%2Ftutorials%2Ffile-upload%2F target="\_blank").


[AZURE.INCLUDE [iot-hub-file-upload-cloud-csharp](../../includes/iot-hub-file-upload-cloud-csharp.md)]


[AZURE.INCLUDE [iot-hub-file-upload-device-csharp](../../includes/iot-hub-file-upload-device-csharp.md)]

## Exécution des applications

Vous êtes maintenant prêt à exécuter les applications.

1.  Dans Visual Studio, cliquez avec le bouton droit sur votre solution et sélectionnez **Définir les projets de démarrage**. Sélectionnez **Plusieurs projets de démarrage**, puis sélectionnez l’action **Démarrer** pour les applications **SimulatedDevice** et **SendCloudToDevice**.

2.  Appuyez sur **F5**. Toutes les applications doivent démarrer. Sélectionnez la fenêtre **SendCloudToDevice** et appuyez sur une touche. L’appareil simulé génère un message quand il a téléchargé le fichier et l’application **SendCloudToDevice** affiche l’accusé de réception. Vous pouvez utiliser le [portail Azure] ou l’Explorateur de serveurs Visual Studio pour vérifier la présence du fichier dans votre compte de stockage.

  ![][50]


## Étapes suivantes

Dans ce didacticiel, vous avez appris à tirer parti des messages cloud-à-appareil pour simplifier les chargements de fichiers à partir d’appareils. Vous pouvez continuer à explorer les scénarios et les fonctionnalités d’IoT Hub avec le didacticiel suivant :

- [Traiter les messages appareil-à-cloud] montre comment traiter de manière fiable des messages interactifs et de télémétrie provenant d’appareils.

Informations supplémentaires sur IoT Hub :

* [Vue d’ensemble d’IoT Hub]
* [Guide du développeur d’IoT Hub]
* [Conseils pour IoT Hub]
* [Langages et plateformes d’appareils pris en charge][Supported devices]
* [Centre de développement Azure IoT]

<!-- Images. -->

[50]: ./media/iot-hub-csharp-csharp-file-upload/run-apps1.png

<!-- Links -->

[Envoyer des messages du cloud vers des appareils avec IoT Hub]: iot-hub-csharp-csharp-c2d.md

[portail Azure]: https://portal.azure.com/

[Azure Data Factory]: https://azure.microsoft.com/documentation/services/data-factory/
[Hadoop]: https://azure.microsoft.com/documentation/services/hdinsight/

[Get started with IoT Hub]: iot-hub-csharp-csharp-getstarted.md
[Envoyer des messages cloud-à-appareil avec IoT Hub]: iot-hub-csharp-csharp-c2d.md
[Envoyer des messages cloud-à-appareil avec IoT Hub]: iot-hub-csharp-csharp-c2d.md
[Traiter les messages appareil-à-cloud]: iot-hub-csharp-csharp-process-d2c.md
[Uploading files from devices]: iot-hub-csharp-csharp-file-upload.md

[Vue d’ensemble d’IoT Hub]: iot-hub-what-is-iot-hub.md
[Conseils pour IoT Hub]: iot-hub-guidance.md
[Guide du développeur d’IoT Hub]: iot-hub-devguide.md
[IoT Hub Supported Devices]: iot-hub-supported-devices.md
[Prise en main d’IoT Hub]: iot-hub-csharp-csharp-getstarted.md
[Supported devices]: https://github.com/Azure/azure-iot-sdks/blob/master/doc/tested_configurations.md
[Centre de développement Azure IoT]: http://www.azure.com/develop/iot

<!----------HONumber=AcomDC_0309_2016-->