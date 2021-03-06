<properties
	pageTitle="Application hybride locale/dans le cloud (.NET) | Microsoft Azure"
	description="Découvrez comment créer une application hybride locale/de cloud .NET à l’aide d’Azure Service Bus Relay."
	services="service-bus"
	documentationCenter=".net"
	authors="sethmanheim"
	manager="timlt"
	editor=""/>

<tags
	ms.service="service-bus"
	ms.workload="tbd"
	ms.tgt_pltfrm="na"
	ms.devlang="dotnet"
	ms.topic="get-started-article"
	ms.date="10/07/2015"
	ms.author="sethm"/>

# Application hybride .NET locale/dans le cloud avec Azure Service Bus Relay

## Introduction

Le développement d'applications cloud hybride avec Microsoft Azure est simple grâce à Visual Studio 2013 et au Kit de développement logiciel (SDK) Azure gratuit pour .NET. Cet article part du principe que vous n’avez pas d’expérience en tant qu’utilisateur d’Azure. En moins de 30 minutes, vous disposez d’une application qui utilise plusieurs ressources Azure s’exécutant dans le cloud.

Vous apprendrez à effectuer les opérations suivantes :

-   créer ou adapter un service Web existant qui sera utilisé par une solution Web ;
-   utiliser Azure Service Bus Relay pour partager des données entre une application Azure et un service Web hébergé ailleurs.

[AZURE.INCLUDE [create-account-note](../../includes/create-account-note.md)]

## Avantages de l’utilisation de service bus relay pour les solutions hybrides

Les solutions d'entreprise se composent généralement d'une combinaison de code personnalisé écrit pour répondre aux exigences nouvelles et uniques de l'entreprise ainsi qu'aux fonctionnalités existantes offertes par des solutions et des systèmes déjà en place.

Les architectes de solutions commencent à utiliser le cloud, car celui-ci permet de gérer plus facilement les exigences de mise à l'échelle tout en offrant des coûts opérationnels faibles. Ainsi, ils découvrent que les ressources des services existants qu'ils souhaiteraient exploiter comme des blocs de construction pour leurs solutions se situent à l'intérieur du pare-feu d'entreprise et hors de portée pour une solution cloud. Bon nombre de services internes ne sont pas créés ni hébergés pour être facilement exposés dans le périmètre du réseau d'entreprise.

Service Bus Relay est conçu pour rendre accessibles les services Web WCF (Windows Communication Foundation) existants, d’une manière sécurisée, aux solutions qui résident à l’extérieur du périmètre de l’entreprise sans exiger de modifications intrusives de l’infrastructure de réseau d’entreprise. Ces services de relais Service Bus sont toujours hébergés dans leur environnement existant, mais ils délèguent l’écoute des sessions et demandes entrantes au Service Bus hébergé sur le cloud. Service Bus protège également ces services de tout accès non autorisé à l’aide de l’authentification par [signature d’accès partagé](https://msdn.microsoft.com/library/dn170478.aspx) (SAP).

## Scénario de la solution

Dans ce didacticiel, vous allez créer un site Web ASP.NET MVC qui vous permet de voir une liste de produits sur la page d’inventaire des produits.

![][0]

Ce didacticiel part du principe que vous disposez des informations sur les produits dans un système local existant, et utilise Service Bus Relay pour atteindre ce système. La simulation met en scène un service Web exécuté dans une simple application console et appuyé par un ensemble de produits en mémoire. Vous allez pouvoir exécuter cette application console sur votre propre ordinateur et déployer le rôle Web dans Azure. Ainsi, vous verrez comment procède le rôle Web en cours d'exécution dans le centre de données Azure pour appeler votre ordinateur, même si ce dernier réside certainement derrière au moins un pare-feu et une couche de traduction d'adresses réseau (NAT, network address translation).

La capture d'écran suivante montre la page d'accueil de l'application Web terminée.

![][1]

## Configuration de l’environnement de développement

Avant de commencer à développer votre application Azure, procurez-vous les outils et configurez votre environnement de développement.

1.  Installer le Kit de développement logiciel Azure pour .NET à [obtenir des outils et Kit de développement logiciel][].

2. 	Cliquez sur **Installer le Kit de développement logiciel (SDK)** correspondant à votre version de Visual Studio. Les étapes de ce didacticiel utilisent Visual Studio 2013.

	![][42]

4.  Lorsque vous êtes invité à exécuter ou à enregistrer le programme d’installation, cliquez sur **Exécuter**.

    ![][2]

5.  Dans **Web Platform Installer**, cliquez sur **Installer**, puis poursuivez l’installation.

    ![][3]

6.  Une fois l’installation terminée, vous disposez de tous les éléments nécessaires pour commencer le développement de l’application. Le Kit de développement logiciel (SDK) comprend des outils qui vous permettent de facilement développer des applications Azure dans Visual Studio. Si Visual Studio n’est pas installé, le Kit de développement logiciel (SDK) installe Visual Studio Express gratuitement.

## Création d'un espace de noms de service

Pour commencer à utiliser les fonctionnalités de Service Bus dans Azure, vous devez d'abord créer un espace de noms de service. Ce dernier fournit un conteneur d'étendue pour l'adressage des ressources Service Bus au sein de votre application.

Vous pouvez également gérer les espaces de noms et les entités de messagerie Service Bus à l’aide du [Portail Azure Classic][] ou de l’Explorateur de serveurs Visual Studio, mais vous ne pouvez créer d’espaces de noms que depuis le portail.

### Créer un espace de noms à l’aide du Portail Azure Classic :

1.  Connectez-vous au [Portail Azure Classic][].

2.  Dans le volet de navigation de gauche du portail, cliquez sur **Service Bus**.

3.  Dans le volet inférieur du portail, cliquez sur **Créer**.

    ![][5]

4.  Dans la boîte de dialogue **Ajouter un nouvel espace de noms**, saisissez un nom d’espace de noms. Le système vérifie immédiatement si le nom est disponible. ![][6]

5.  Après vous être assuré que le nom de l’espace de noms est disponible, choisissez le pays ou la région où votre espace de noms doit être hébergé (veillez à utiliser le même pays ou la même région que celui ou celle où vous déployez vos ressources de calcul).

    > [AZURE.IMPORTANT] choisissez la *même région* que celle que vous prévoyez de sélectionner pour le déploiement de votre application. Vous bénéficiez ainsi des meilleures performances.

6.	Laissez les autres champs de la boîte de dialogue avec leurs valeurs par défaut (**Messaging** et **Niveau Standard**), puis cliquez sur la coche. Le système crée l'espace de noms de service et l'active. Vous devrez peut-être attendre plusieurs minutes afin que le système approvisionne des ressources pour votre compte.

	![][38]

L’espace de noms créé s’affichera dans le Portail Azure Classic, même si son activation peut prendre un certain temps. Patientez jusqu'à ce que l'état soit **Actif** avant de continuer.

## Obtention d'informations d'identification de gestion par défaut pour l'espace de noms

Pour exécuter des opérations de gestion sur le nouvel espace de noms, comme la création d’entités de messagerie, vous devez obtenir les informations d’identification pour l’espace de noms.

1.  Dans la fenêtre principale, cliquez sur le nom de votre espace de noms de service.

	![][39]

2.  Cliquez sur **Connection Information**.

	![][40]

3.  Dans le volet **Accès aux informations de connexion**, recherchez la chaîne de connexion contenant la clé SAP et le nom de clé.

	![][45]

4.  Notez ces informations d'identification ou copiez-les dans le Presse-papiers.

## Création d’un serveur local

Vous créez d'abord un système local de catalogue de produits (fictif). Cela est assez simple : représentez-vous un système réel de catalogue de produits local avec une surface complète de services que nous essayons d'intégrer.

Ce projet démarre comme une application console Visual Studio. Il utilise le package NuGet Service Bus pour inclure les bibliothèques et les paramètres de configuration Service Bus. L'extension Visual Studio NuGet facilite l'installation et la mise à jour des bibliothèques et des outils de Visual Studio et Visual Studio Express. Le package NuGet Service Bus est le moyen le plus simple de se procurer l'API Service Bus et de configurer votre application avec toutes les dépendances Service Bus. Pour plus d'informations sur l'utilisation de NuGet et du package Service Bus, consultez la page [Utilisation du package NuGet Service Bus][].

### Création du projet

1.  Avec les privilèges d'administrateur, démarrez Microsoft Visual Studio 2013 ou Microsoft Visual Studio Express. Pour démarrer Visual Studio avec les privilèges d'administrateur, cliquez avec le bouton droit sur **Microsoft Visual Studio 2013 (ou Microsoft Visual Studio Express)**, puis cliquez sur **Exécuter en tant qu'administrateur**.

2.  Dans Visual Studio, dans le menu **Fichier**, cliquez sur **Nouveau**, puis sur **Projet**.

    ![][10]

3.  Dans **Modèles installés**, sous **Visual C#**, cliquez sur **Application console**. Dans la zone **Nom**, saisissez le nom **ProductsServer** :

    ![][11]

4.  Cliquez sur **OK** pour créer le projet **ProductsServer**.

5.  Dans l’Explorateur de solutions, cliquez avec le bouton droit sur **ProductsServer**, puis cliquez sur **Propriétés**.

6.  Cliquez sur l’onglet **Application** situé à gauche, puis assurez-vous que **.NET Framework 4** ou **.NET Framework 4.5** apparaît dans la liste déroulante **Version cible de .NET Framework** Dans le cas contraire, sélectionnez cette entrée dans la liste, puis cliquez sur **Oui** lorsque vous êtes invité à recharger le projet.

    ![][12]

7.  Si vous avez déjà installé le gestionnaire de package NuGet pour Visual Studio, passez à l'étape suivante. Sinon, accédez à [NuGet][], puis cliquez sur [Install NuGet](http://visualstudiogallery.msdn.microsoft.com/27077b70-9dad-4c64-adcf-c7cf6bc9970c). Suivez les invites pour installer le gestionnaire de package NuGet, puis redémarrez Visual Studio.

7.  Dans l'Explorateur de solutions, cliquez avec le bouton droit sur **References**, puis cliquez sur **Manage NuGet Packages**.

8.  Dans la colonne de gauche de la boîte de dialogue **NuGet**, cliquez sur **Online**.

9. 	Dans la colonne de droite, cliquez sur la zone **Rechercher**, saisissez « **Service Bus** », puis sélectionnez l’élément **Microsoft Azure Service Bus**. Cliquez sur **Installer** pour terminer l’installation, puis fermez cette boîte de dialogue.

    ![][13]

    Notez que les assemblys client nécessaires sont maintenant référencés.

9.  Ajoutez une nouvelle classe pour votre contrat de produit. Dans l’Explorateur de solutions, cliquez avec le bouton droit sur le projet **ProductsServer**, cliquez sur **Ajouter**, puis sur **Classe**.

    ![][14]

10. Dans la zone **Nom**, entrez le nom **ProductsContract.cs**. Cliquez ensuite sur **Add**.

11. Dans **ProductsContract.cs**, remplacez la définition d’espace de noms existante par le code suivant, qui définit le contrat du service.

        namespace ProductsServer
        {
            using System.Collections.Generic;
            using System.Runtime.Serialization;
            using System.ServiceModel;

            // Define the data contract for the service
            [DataContract]
            // Declare the serializable properties.
            public class ProductData
            {
                [DataMember]
                public string Id { get; set; }
                [DataMember]
                public string Name { get; set; }
                [DataMember]
                public string Quantity { get; set; }
            }

            // Define the service contract.
            [ServiceContract]
            interface IProducts
            {
                [OperationContract]
                IList<ProductData> GetProducts();

            }

            interface IProductsChannel : IProducts, IClientChannel
            {
            }
        }

12. Dans Program.cs, remplacez la définition d’espace de noms existante par le code suivant, qui ajoute le service de profil et l’hôte correspondant :

        namespace ProductsServer
        {
            using System;
            using System.Linq;
            using System.Collections.Generic;
            using System.ServiceModel;

            // Implement the IProducts interface.
            class ProductsService : IProducts
            {

                // Populate array of products for display on website.
                ProductData[] products =
                    new []
                        {
                            new ProductData{ Id = "1", Name = "Rock",
                                             Quantity = "1"},
                            new ProductData{ Id = "2", Name = "Paper",
                                             Quantity = "3"},
                            new ProductData{ Id = "3", Name = "Scissors",
                                             Quantity = "5"},
                            new ProductData{ Id = "4", Name = "Well",
                                             Quantity = "2500"},
                        };

                // Display a message in the service console application
                // when the list of products is retrieved.
                public IList<ProductData> GetProducts()
                {
                    Console.WriteLine("GetProducts called.");
                    return products;
                }

            }

            class Program
            {
                // Define the Main() function in the service application.
                static void Main(string[] args)
                {
                    var sh = new ServiceHost(typeof(ProductsService));
                    sh.Open();

                    Console.WriteLine("Press ENTER to close");
                    Console.ReadLine();

                    sh.Close();
                }
            }
        }

13. Dans l’Explorateur de solutions, double-cliquez sur le fichier **App.config** pour l’ouvrir dans l’éditeur de Visual Studio. Remplacez le contenu de **&lt;system.ServiceModel&gt;** par le code XML suivant. Assurez-vous de remplacer *yourServiceNamespace* par le nom de votre espace de noms de service, puis *yourKey* par la clé SAP que vous avez récupérée précédemment sur le Portail Azure Classic :

        <system.serviceModel>
          <extensions>
             <behaviorExtensions>
                <add name="transportClientEndpointBehavior" type="Microsoft.ServiceBus.Configuration.TransportClientEndpointBehaviorElement, Microsoft.ServiceBus, Version=2.6.0.0, Culture=neutral, PublicKeyToken=31bf3856ad364e35"/>
              </behaviorExtensions>
              <bindingExtensions>
                 <add name="netTcpRelayBinding" type="Microsoft.ServiceBus.Configuration.NetTcpRelayBindingCollectionElement, Microsoft.ServiceBus, Version=2.6.0.0, Culture=neutral, PublicKeyToken=31bf3856ad364e35"/>
              </bindingExtensions>
          </extensions>
          <services>
             <service name="ProductsServer.ProductsService">
               <endpoint address="sb://yourServiceNamespace.servicebus.windows.net/products" binding="netTcpRelayBinding" contract="ProductsServer.IProducts"
        behaviorConfiguration="products"/>
             </service>
          </services>
          <behaviors>
             <endpointBehaviors>
               <behavior name="products">
                 <transportClientEndpointBehavior>
                    <tokenProvider>
                       <sharedAccessSignature keyName="RootManageSharedAccessKey" key="yourKey" />
                    </tokenProvider>
                 </transportClientEndpointBehavior>
               </behavior>
             </endpointBehaviors>
          </behaviors>
        </system.serviceModel>

14. Appuyez sur F6 ou, dans le menu **Générer**, cliquez sur **Générer la solution** pour générer l’application afin de vérifier si votre travail est correct.

## Création d’une application ASP.NET MVC

Dans cette section, vous allez générer une application ASP.NET simple qui affiche des données récupérées de votre service de produit.

### Création du projet

1.  Vérifiez que Visual Studio fonctionne avec les privilèges d’administrateur. Dans le cas contraire, pour démarrer Visual Studio avec les privilèges d’administrateur, cliquez avec le bouton droit sur **Microsoft Visual Studio 2013 (ou Microsoft Visual Studio Express)**, puis cliquez sur **Exécuter en tant qu’administrateur**. L’émulateur de calcul Microsoft Azure, présenté plus loin dans cet article , nécessite que Visual Studio soit démarré avec les privilèges d’administrateur.

2.  Dans Visual Studio, dans le menu **Fichier**, cliquez sur **Nouveau**, puis sur **Projet**.

3.  Dans **Modèles installés**, sous **Visual C#**, cliquez sur **Application Web ASP.NET**. Nommez le projet **ProductsPortal**. Cliquez ensuite sur **OK**.

    ![][15]

4.  Dans la liste **Sélectionner un modèle**, cliquez sur **MVC**, puis sur **OK**.

    ![][16]

5.  Dans Explorateur de solutions, cliquez avec le bouton droit sur **Modèles**, cliquez sur **Ajouter**, puis sur **Classe**. Dans la zone **Nom**, entrez le nom **Products.cs**. Cliquez ensuite sur **Add**.

    ![][17]

### Modification de l’application Web

1.  Dans le fichier Product.cs dans Visual Studio, remplacez la définition d’espace de noms existante par le code suivant.

        // Declare properties for the products inventory.
        namespace ProductsWeb.Models
        {
            public class Product
            {
                public string Id { get; set; }
                public string Name { get; set; }
                public string Quantity { get; set; }
            }
        }

2.  Dans le fichier HomeController.cs dans Visual Studio, remplacez la définition d’espace de noms existante par le code suivant.

        namespace ProductsWeb.Controllers
        {
            using System.Collections.Generic;
            using System.Web.Mvc;
            using Models;

            public class HomeController : Controller
            {
                // Return a view of the products inventory.
                public ActionResult Index(string Identifier, string ProductName)
                {
                    var products = new List<Product>
                        {new Product {Id = Identifier, Name = ProductName}};
                    return View(products);
                }

            }
        }

3.  Dans l’Explorateur de solutions, développez le dossier Views\\Shared :

    ![][18]

4.  Double-cliquez ensuite **\_Layout.cshtml** pour l’ouvrir dans l’éditeur de Visual Studio.

5.  Remplacez toutes les occurrences de **Mon application ASP.NET** par **LITWARE's Products**.

6. Supprimez les liens **Home**, **About** et **Contact**. Dans l’exemple suivant, supprimez le code en surbrillance.

	![][41]

7.  Dans l’Explorateur de solutions, développez Views\\Home :

    ![][20]

8.  Double-cliquez ensuite sur **Index.cshtml** pour l’ouvrir dans l’éditeur de Visual Studio. Remplacez tout le contenu du fichier par le code suivant.

		@model IEnumerable<ProductsWeb.Models.Product>

		@{
    		ViewBag.Title = "Index";
		}

		<h2>Prod Inventory</h2>

		<table>
    		<tr>
        		<th>
            		@Html.DisplayNameFor(model => model.Name)
        		</th>
                <th></th>
        		<th>
            		@Html.DisplayNameFor(model => model.Quantity)
        		</th>
    		</tr>

		@foreach (var item in Model) {
    		<tr>
        		<td>
            		@Html.DisplayFor(modelItem => item.Name)
        		</td>
        		<td>
            		@Html.DisplayFor(modelItem => item.Quantity)
        		</td>
    		</tr>
		}

		</table>

9.  À ce stade, pour vérifier votre travail, appuyez sur **F6** ou **Ctrl+Maj+B** pour générer le projet.


### Exécution locale de l'application

Exécutez l'application afin de vérifier qu'elle fonctionne.

1.  Assurez-vous que **ProductsPortal** est le projet actif. Dans l’Explorateur de solutions, cliquez avec le bouton droit sur le nom du projet, puis sélectionnez **Définir comme projet de démarrage**.
2.  Dans **Visual Studio**, appuyez sur F5.
3.  Votre application doit s’exécuter dans un navigateur :

    ![][21]

## Préparation de votre application en vue de son déploiement sur azure

Vous pouvez déployer votre application vers un service cloud Azure ou un site web Azure. Pour plus d'informations sur les différences entre les sites web et les services cloud, consultez la page [Modèles d'exécution Azure][executionmodels]. Pour plus d’informations sur le déploiement de l’application vers un site web Azure, consultez la page [Déploiement d’une application web ASP.NET vers un site web Azure](https://azure.microsoft.com/develop/net/tutorials/get-started/). Cette section contient une procédure pas à pas pour déployer l’application sur un service cloud Azure.

Pour déployer votre application sur un service cloud, vous ajoutez un projet de déploiement de service cloud à la solution. Le projet de déploiement contient des informations de configuration qui sont nécessaires pour exécuter correctement l'application sur le cloud.

1.  Pour permettre le déploiement de votre application sur le cloud, cliquez avec le bouton droit sur le projet **ProductsPortal** dans l’Explorateur de solutions, puis cliquez sur **Convertir** et sur **Convertir en projet service cloud Microsoft Azure**.

    ![][22]

2.  Pour tester votre application, appuyez sur F5.

3.  L'émulateur de calcul Azure démarre alors. Il utilise l'ordinateur local pour émuler votre application qui s'exécute dans Azure. Vous savez que l’émulateur a démarré en consultant la zone de notification.

       ![][23]

4.  Un navigateur continue d'afficher votre application s'exécutant en local. Son apparence et son fonctionnement sont similaires à ceux de l'application ASP.NET MVC 4 classique que vous exécutiez précédemment.

## Assemblage des éléments

La prochaine étape consiste à raccorder le serveur de produits local et l'application ASP.NET MVC.

1.  S'il n'est pas déjà ouvert, dans Visual Studio rouvrez le projet **ProductsPortal** que vous avez créé dans la section « Création d'une application ASP.NET MVC ».

2.  Comme vous l'avez fait dans la section « Création d'un serveur local », ajoutez le package NuGet au projet Références. Dans l'Explorateur de solutions, cliquez avec le bouton droit sur **References**, puis cliquez sur **Manage NuGet Packages**.

3.  Recherchez « Service Bus » et sélectionnez l’élément **Microsoft Azure Service Bus**. Ensuite, terminez l’installation et fermez cette boîte de dialogue.

4.  Dans l'Explorateur de solutions, cliquez avec le bouton droit sur le projet **ProductsPortal**, cliquez sur **Ajouter**, puis sur **Élément existant**.

5.  Accédez au fichier **ProductsContract.cs** depuis le projet de console **ProductsServer**. Cliquez sur ProductsContract.cs afin de le mettre en surbrillance. Cliquez sur la flèche vers le bas en regard d’**Ajouter**, puis cliquez sur **Ajouter en tant que lien**.

	![][24]

6.  Ouvrez à présent le fichier **HomeController.cs** dans l'éditeur Visual Studio, puis remplacez la définition d'espace de noms existante par le code suivant. Assurez-vous de remplacer *yourServiceNamespace* par le nom de votre espace de noms de service, puis *yourKey* par votre clé SAP. Le client peut alors appeler le service local, en renvoyant le résultat de l'appel.

            namespace ProductsWeb.Controllers
            {
                using System.Linq;
                using System.ServiceModel;
                using System.Web.Mvc;
                using Microsoft.ServiceBus;
                using Models;
                using ProductsServer;

                public class HomeController : Controller
                {
                    // Declare the channel factory.
                    static ChannelFactory<IProductsChannel> channelFactory;

                    static HomeController()
                    {
                        // Create shared secret token credentials for authentication.
                        channelFactory = new ChannelFactory<IProductsChannel>(new NetTcpRelayBinding(),
                            "sb://yourServiceNamespace.servicebus.windows.net/products");
                        channelFactory.Endpoint.Behaviors.Add(new TransportClientEndpointBehavior {
                            TokenProvider = TokenProvider.CreateSharedAccessSignatureTokenProvider(
                                "RootManageSharedAccessKey", "yourKey") });
                    }

                    public ActionResult Index()
                    {
                        using (IProductsChannel channel = channelFactory.CreateChannel())
                        {
                            // Return a view of the products inventory.
                            return this.View(from prod in channel.GetProducts()
                                             select
                                                 new Product { Id = prod.Id, Name = prod.Name,
                                                     Quantity = prod.Quantity });
                        }
                    }
                }
            }
7.  Dans l’Explorateur de solutions, cliquez avec le bouton droit sur la solution **ProductsPortal**, cliquez sur **Ajouter**, puis sur **Projet existant**.

8.  Accédez au projet **ProductsServer**, puis double-cliquez sur le fichier solution **ProductsServer.csproj** pour l'ajouter.

9.  Dans l'Explorateur de solutions, cliquez avec le bouton droit sur la solution **ProductsPortal**, puis cliquez sur **Propriétés**.

10. Sur le côté gauche, cliquez sur **Projet de démarrage**. Sur le côté droit, cliquez sur **Plusieurs projets de démarrage**. Assurez-vous que **ProductsServer**, **ProductsPortal.Azure** et **ProductsPortal** apparaissent, dans cet ordre, avec **Démarrer** défini comme action pour **ProductsServer** et **ProductsPortal.Azure**, et **None** défini comme action pour **ProductsPortal**.

      ![][25]

11. Toujours dans la boîte de dialogue **Propriétés**, cliquez sur **ProjectDependencies** Sur le côté gauche.

12. Dans le menu déroulant **Projets**, cliquez sur **ProductsServer**. Assurez-vous que **ProductsPortal** n’est pas sélectionné et que **ProductsPortal.Azure** l’est. Cliquez ensuite sur **OK** :

    ![][26]

## Exécution de l'application

1.  Dans Visual Studio, dans le menu **Fichier**, cliquez sur **Enregistrer tout**.

2.  Appuyez sur F5 pour générer et exécuter l’application. Le serveur local (l’application console **ProductsServer**) doit démarrer en premier, puis l’application**ProductsWeb** doit démarrer dans une fenêtre de navigateur, comme illustré dans la capture d’écran suivant. À présent, vous voyez que l'inventaire de produits répertorie des données récupérées du système local de service de produit.

    ![][1]

## Déploiement de votre application dans Azure

1.  Dans l’Explorateur de solutions, cliquez avec le bouton droit sur le projet **ProductsPortal**, puis cliquez sur **Publier sur Microsoft Azure**.

2.  Vous devrez peut-être vous connecter pour voir tous vos abonnements.

    Cliquez sur **Sign in to see more subscriptions** :

    ![][27]

3.  Connectez-vous à l’aide de votre compte Microsoft.

8.  Cliquez sur **Next**. Si votre abonnement ne contient pas encore de service hébergé, vous êtes invité à en créer un. Le service hébergé se comporte comme un conteneur pour votre application au sein de votre abonnement Azure. Entrez un nom qui identifie votre application et choisissez la région pour laquelle l'application doit être optimisée. Les temps de chargement seront plus rapides pour les utilisateurs y accédant à partir de cette région.

9.  Sélectionnez le service hébergé vers lequel vous voulez publier votre application. Conservez les valeurs par défaut comme indiqué ci-dessous pour les autres paramètres. Cliquez sur **Next**.

    ![][33]

10. Dans la dernière page, cliquez sur **Publier** pour démarrer le processus de déploiement.

    ![][34] Cela prend environ 5 à 7 minutes. Comme il s'agit de votre première publication, Azure met en service une machine virtuelle, renforce la sécurité, crée un rôle Web sur la machine virtuelle pour héberger votre application, déploie votre code sur ce rôle Web, et enfin configure l'équilibrage de charge et la mise en réseau afin de rendre votre application accessible au public.

11. Lors de la publication, vous serez en mesure de surveiller l’activité dans la fenêtre **Journal des activités Azure**, qui est généralement ancrée au bas de Visual Studio ou de Visual Web Developer.

    ![][35]

12. Lorsque le déploiement est terminé, vous pouvez afficher votre site Web en cliquant sur le lien **URL du site Web** dans la fenêtre de contrôle.

    ![][36]

    Votre site Web reposant sur le serveur local, vous devez exécuter l’application **ProductsServer** en local afin que le site Web fonctionne correctement. En effectuant des demandes sur le site web du cloud, vous voyez des demandes s’afficher dans votre application console locale, comme indiqué dans la sortie « GetProducts called » visible dans la capture d’écran suivant.

    ![][37]

Pour plus d'informations sur les différences entre les sites web et les services cloud, consultez la page [Modèles d'exécution Azure][executionmodels].

## Arrêt et suppression de l’application

Après avoir déployé votre application, vous pouvez la désactiver afin de générer et de déployer d'autres applications pendant le temps serveur gratuit de 750 heures/mois (31 jours/mois).

Azure facture les instances de rôle Web par heure de serveur consommée. Une fois votre application déployée, elle consomme du temps de serveur, même si les instances ne sont pas exécutées et sont arrêtées. Un compte gratuit comprend 750 heures/mois (31 jours/mois) de temps de serveur pour une machine virtuelle dédiée permettant d'héberger ces instances de rôle Web.

La procédure suivante présente l'arrêt et la suppression de l'application.

1.  Connectez-vous au [Portail Azure Classic][], cliquez sur **Cloud Services**, puis sur le nom de votre service.

2.  Sélectionnez l’onglet **Tableau de bord**, puis cliquez sur **Arrêter** pour suspendre momentanément l’application. Vous pouvez la redémarrer en cliquant simplement sur **Démarrer**. Cliquez sur **Supprimer** pour supprimer complètement l'application d'Azure, sans aucune possibilité de la restaurer.

	![][43]

## Étapes suivantes  

Pour en savoir plus sur Service Bus, consultez les ressources suivantes :

* [Azure Service Bus][sbwacom]  
* [Utilisation des files d'attente Service Bus][sbwacomqhowto]  


  [0]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/hybrid.png
  [1]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/App2.png
  [obtenir des outils et Kit de développement logiciel]: http://go.microsoft.com/fwlink/?LinkId=271920
  [NuGet]: http://nuget.org
  [2]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/getting-started-3.png
  [3]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/getting-started-42-webpi.png


  [Portail Azure Classic]: http://manage.windowsazure.com
  [5]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/sb-queues-03.png
  [6]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/sb-queues-04.png



  [Utilisation du package NuGet Service Bus]: https://msdn.microsoft.com/library/azure/dn741354.aspx
  [10]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/hy-web-1.png
  [11]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/hy-con-1.png
  [12]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/hy-con-3.png
  [13]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/getting-started-multi-tier-13.png
  [14]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/hy-con-4.png
  [15]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/hy-web-2.png
  [16]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/hy-web-4.png
  [17]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/hy-web-7.jpg
  [18]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/hy-web-10.jpg

  [20]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/hy-web-11.png
  [21]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/App1.png
  [22]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/getting-started-hybrid-21.png
  [23]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/getting-started-hybrid-22.png
  [24]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/hy-web-12.png
  [25]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/hy-web-13.png
  [26]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/hy-web-14.png
  [27]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/getting-started-hybrid-33.png


  [30]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/getting-started-hybrid-36.png
  [31]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/getting-started-hybrid-37.png
  [32]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/getting-started-hybrid-38.png
  [33]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/getting-started-hybrid-39.png
  [34]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/getting-started-hybrid-40.png
  [35]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/getting-started-hybrid-41.png
  [36]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/App2.png
  [37]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/hy-service1.png
  [38]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/getting-started-multi-tier-27.png
  [39]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/sb-queues-09.png
  [40]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/sb-queues-06.png
  [41]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/getting-started-multi-tier-40.png
  [42]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/getting-started-41.png
  [43]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/getting-started-hybrid-43.png
  [45]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/hy-web-45.png

  [sbwacom]: /documentation/services/service-bus/
  [sbwacomqhowto]: service-bus-dotnet-how-to-use-queues.md
  [executionmodels]: ../cloud-services/fundamentals-application-models.md

<!---HONumber=AcomDC_0128_2016-->