<properties 
	pageTitle="Comment autoriser des comptes de développeurs à l'aide de OAuth 2.0 dans Gestion des API Azure" 
	description="Apprenez à autoriser les utilisateurs à l'aide d'OAuth 2.0 dans Gestion des API." 
	services="api-management" 
	documentationCenter="" 
	authors="steved0x" 
	manager="erikre" 
	editor=""/>

<tags 
	ms.service="api-management" 
	ms.workload="mobile" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="03/04/2016" 
	ms.author="sdanie"/>

# Comment autoriser des comptes de développeurs à l'aide de OAuth 2.0 dans Gestion des API Azure

De nombreuses API prennent en charge [OAuth 2.0](http://oauth.net/2/) pour sécuriser les API et assurer que seuls les utilisateurs valides y ont accès et peuvent accéder uniquement aux ressources pour lesquelles ils y sont autorisés. Pour utiliser la console de développement interactive de Gestion des API Azure avec ces API, le service vous permet de configurer votre instance de service pour travailler avec votre API compatible OAuth 2.0.

## <a name="prerequisites"> </a>Configuration requise

Ce guide explique comment configurer votre instance de service Gestion des API pour utiliser l'autorisation OAuth 2.0 dans des comptes de développeur, mais n'explique pas comment configurer un fournisseur OAuth 2.0. La configuration de chaque fournisseur OAuth 2.0 est différente, bien que la procédure soit similaire et que les informations requises pour configurer OAuth 2.0 dans votre instance de service Gestion des API soient identiques. Cette rubrique inclut des exemples d'utilisation de Azure Active Directory en tant que fournisseur OAuth 2.0.

>[AZURE.NOTE] Pour plus d’informations sur la configuration de OAuth 2.0 à l’aide d’Azure Active Directory, consultez l’exemple [WebApp-GraphAPI-DotNet][].

## <a name="step1"> </a>Configuration du serveur d’autorisation OAuth 2.0 dans Gestion des API

Pour commencer, cliquez sur **Gérer** dans le portail Azure Classic de votre service Gestion des API. Vous accédez au portail des éditeurs Gestion des API.

![Portail des éditeurs][api-management-management-console]

>[AZURE.NOTE] Si vous n'avez pas encore créé une instance de service Gestion des API, consultez la page [Création d'une instance de service Gestion des API][] dans le didacticiel [Prise en main de Gestion des API Azure][].

Cliquez sur **Sécurité** dans le menu **Gestion des API** de gauche, cliquez sur **OAuth 2.0**, puis cliquez sur **Ajouter au serveur d'autorisation**.

![OAuth 2.0][api-management-oauth2]

Après avoir cliqué sur **Ajouter au serveur d'autorisation**, le formulaire de nouveau serveur d'autorisation s'affiche.

![Nouveau serveur][api-management-oauth2-server-1]

Entrez un nom et une description (facultatif) dans les champs **Nom** et **Description**.

>[AZURE.NOTE] Ces champs sont utilisés pour identifier le serveur d'autorisation OAuth 2.0 dans l'instance de service Gestion des API actuelle, et leurs valeurs ne proviennent pas du serveur OAuth 2.0.

Entrez l'**URL de la page d'enregistrement client**. Cette page est l'endroit où les utilisateurs créent et gèrent leurs comptes ; elle varie en fonction du fournisseur OAuth 2.0 utilisé. L’**URL de la page d’enregistrement client** pointe vers la page que les utilisateurs peuvent utiliser pour créer et configurer leurs propres comptes pour les fournisseurs OAuth 2.0 qui prennent en charge la gestion des comptes utilisateur. Certaines organisations ne configurent pas cette fonctionnalité ou ne l’utilisent pas, même si le fournisseur OAuth 2.0 la prend en charge. Si la gestion des utilisateurs de comptes de votre fournisseur OAuth 2.0 n’a pas été configurée, entrez un espace réservé d’URL ici, par exemple l’URL de votre société ou une URL comme `https://placeholder.contoso.com`.

La section suivante du formulaire inclut les paramètres **Types d'accès accordé aux codes d'autorisation**, **URL de point de terminaison d'autorisation** et **Méthode de demande d’autorisation**.

![Nouveau serveur][api-management-oauth2-server-2]

Spécifiez le paramètre **Types d'accès accordé aux codes d'autorisation** en sélectionnant les types souhaités. Le **code d'autorisation** est spécifié par défaut.

Entrez l'**URL de point de terminaison d'autorisation**. Pour Azure Active Directory, cette URL est similaire à l’URL suivante, où `<client_id>` est remplacé par l’ID client qui identifie votre application sur le serveur OAuth 2.0.

    https://login.windows.net/<client_id>/oauth2/authorize

La **Méthode de demande d’autorisation** spécifie comment la demande d'autorisation est envoyée au serveur OAuth 2.0. Par défaut, la méthode **GET** est sélectionnée.

La section suivante permet de spécifier les paramètres **URL de point de terminaison de jeton**, **Méthodes d’authentification du client**, **Méthode d'envoi des jetons d'accès** et **Étendue par défaut**.

![Nouveau serveur][api-management-oauth2-server-3]

Pour un serveur OAuth 2.0 Azure Active Directory, l’**URL de point de terminaison de jeton** a le format suivant, où `<APPID>` a le format `yourapp.onmicrosoft.com`.

    https://login.windows.net/<APPID>/oauth2/token

Le paramètre par défaut pour **Méthodes d’authentification du client** est **De base**, et le paramètre par défaut pour **Méthode d'envoi des jetons d'accès** est **En-tête d'autorisation**. Ces valeurs sont configurées dans cette section du formulaire, ainsi que le paramètre **Étendue par défaut**.

La section **Informations d'identification du client** inclut les paramètres **ID client** et **Clé secrète client**, qui sont obtenus lors du processus de création et de configuration de votre serveur OAuth 2.0. Une fois que les paramètres **ID client** et **Clé secrète client** ont été spécifiés, le **redirect\_uri** pour le **code d'autorisation** est généré. Cette URI est utilisée pour configurer l'URL de réponse dans la configuration de votre serveur OAuth 2.0.

![Nouveau serveur][api-management-oauth2-server-4]

Si le paramètre **Types d'accès accordés aux codes d'autorisation** est défini sur **Mot de passe du propriétaire des ressources**, la section **Informations d'identification du mot de passe du propriétaire des ressources** permet de spécifier ces informations d'identification ; sinon, vous pouvez la laisser vide.

![Nouveau serveur][api-management-oauth2-server-5]

Une fois le formulaire complété, cliquez sur **Enregistrer** pour enregistrer la configuration du serveur d'autorisation OAuth 2.0 de Gestion des API. Après l'enregistrement de la configuration du serveur, vous pouvez configurer les API pour utiliser cette configuration, tel qu'expliqué dans la section suivante.

## <a name="step2"> </a>Configuration d’une API pour utiliser l’autorisation utilisateur OAuth 2.0

Cliquez sur **API** dans le menu **Gestion des API** de gauche, cliquez sur le nom de l'API désirée, cliquez sur l'onglet **Sécurité**, puis cochez la case **OAuth 2.0**.

![Autorisation utilisateur][api-management-user-authorization]

Sélectionnez le **Serveur d'autorisation** souhaité dans la liste déroulante, puis cliquez sur **Enregistrer**.

![Autorisation utilisateur][api-management-user-authorization-save]

## <a name="step3"> </a>Tests de l’autorisation utilisateur OAuth 2.0 dans le portail de développement

Une fois que vous avez configuré votre serveur d'autorisation OAuth 2.0 et votre API pour utiliser ce serveur, vous pouvez le tester en accédant au portail de développement et en appelant une API. Cliquez sur **Portail de développement** dans le menu supérieur droit.

![Portail des développeurs][api-management-developer-portal-menu]

Cliquez sur **API** dans le menu supérieur et sélectionnez **API Echo**.

![API Echo][api-management-apis-echo-api]

>[AZURE.NOTE] Si vous n'avez qu'une API configurée ou visible dans votre compte, cliquez sur des API pour accéder directement aux opérations associées.

Sélectionnez l'opération **Ressource GET**, cliquez sur **Ouvrir la console**, puis sélectionnez **Code d'autorisation** dans la liste déroulante.

![Open console][api-management-open-console]

Lorsque **Code d'autorisation** est sélectionné, une fenêtre contextuelle s'affiche, avec le formulaire d'inscription du fournisseur OAuth 2.0. Dans cet exemple, le formulaire d'inscription est fourni par Azure Active Directory.

>[AZURE.NOTE] Si l'utilisation des fenêtres contextuelles est désactivée, vous êtes invité à l'activer par le navigateur. Après l'activation, sélectionnez à nouveau **Code d'autorisation**, et le formulaire d'inscription s'affiche.

![Connexion][api-management-oauth2-signin]

Une fois que vous êtes connecté, les **En-têtes de demandes** sont renseignés avec un en-tête `Authorization : Bearer` qui autorise la demande.

![Jeton d'en-tête de demande][api-management-request-header-token]

À présent, vous pouvez configurer les valeurs souhaitées pour les paramètres restants, puis soumettre la demande.

## Étapes suivantes

Pour plus d’informations sur l’utilisation d’OAuth 2.0 et la gestion des API, regardez la vidéo suivante et l’article qui l’[accompagne](api-management-howto-protect-backend-with-aad.md).

> [AZURE.VIDEO protecting-web-api-backend-with-azure-active-directory-and-api-management]

[api-management-management-console]: ./media/api-management-howto-oauth2/api-management-management-console.png
[api-management-oauth2]: ./media/api-management-howto-oauth2/api-management-oauth2.png
[api-management-user-authorization]: ./media/api-management-howto-oauth2/api-management-user-authorization.png
[api-management-user-authorization-save]: ./media/api-management-howto-oauth2/api-management-user-authorization-save.png
[api-management-oauth2-signin]: ./media/api-management-howto-oauth2/api-management-oauth2-signin.png
[api-management-request-header-token]: ./media/api-management-howto-oauth2/api-management-request-header-token.png
[api-management-developer-portal-menu]: ./media/api-management-howto-oauth2/api-management-developer-portal-menu.png
[api-management-open-console]: ./media/api-management-howto-oauth2/api-management-open-console.png
[api-management-oauth2-server-1]: ./media/api-management-howto-oauth2/api-management-oauth2-server-1.png
[api-management-oauth2-server-2]: ./media/api-management-howto-oauth2/api-management-oauth2-server-2.png
[api-management-oauth2-server-3]: ./media/api-management-howto-oauth2/api-management-oauth2-server-3.png
[api-management-oauth2-server-4]: ./media/api-management-howto-oauth2/api-management-oauth2-server-4.png
[api-management-oauth2-server-5]: ./media/api-management-howto-oauth2/api-management-oauth2-server-5.png
[api-management-apis-echo-api]: ./media/api-management-howto-oauth2/api-management-apis-echo-api.png


[How to add operations to an API]: api-management-howto-add-operations.md
[How to add and publish a product]: api-management-howto-add-products.md
[Monitoring and analytics]: api-management-monitoring.md
[Add APIs to a product]: api-management-howto-add-products.md#add-apis
[Publish a product]: api-management-howto-add-products.md#publish-product
[Prise en main de Gestion des API Azure]: api-management-get-started.md
[Get started with advanced API configuration]: api-management-get-started-advanced.md
[API Management policy reference]: api-management-policy-reference.md
[Caching policies]: api-management-policy-reference.md#caching-policies
[Création d'une instance de service Gestion des API]: api-management-get-started.md#create-service-instance

[http://oauth.net/2/]: http://oauth.net/2/
[WebApp-GraphAPI-DotNet]: https://github.com/AzureADSamples/WebApp-GraphAPI-DotNet

[Prerequisites]: #prerequisites
[Configure an OAuth 2.0 authorization server in API Management]: #step1
[Configure an API to use OAuth 2.0 user authorization]: #step2
[Test the OAuth 2.0 user authorization in the Developer Portal]: #step3
[Next steps]: #next-steps

<!---------HONumber=AcomDC_0309_2016-->