<properties
   pageTitle="Azure Active Directory B2B Collaboration | Microsoft Azure"
   description="La collaboration B2B Azure Active Directory permet aux partenaires professionnels d’accéder à vos applications d’entreprise. Chaque utilisateur est représenté par un compte Azure AD unique."
   services="active-directory"
   documentationCenter=""
   authors="curtand"
   manager="stevenpo"
   editor=""/>

<tags
   ms.service="active-directory"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="identity"
   ms.date="02/09/2016"
   ms.author="curtand"/>

# Azure Active Directory (Azure AD) B2B Collaboration

Azure AD B2B Collaboration vous permet d’activer l’accès à vos applications d’entreprise à partir des identités gérées par les partenaires. Vous pouvez créer des relations intersociétés en invitant et en autorisant des utilisateurs de sociétés partenaires à accéder à vos ressources. La complexité est réduite, car chaque entreprise se fédère une seule fois avec Azure Active Directory et chaque utilisateur est représenté par un seul compte Azure AD. La sécurité est renforcée car l’accès est refusé lorsque des utilisateurs du partenaire sont résiliés de leurs organisations et que l’accès involontaire via l’appartenance dans les répertoires internes est empêché. Pour les partenaires professionnels qui ne disposent pas déjà d’Azure AD, la collaboration B2B offre une expérience d’inscription rationalisée afin de fournir des comptes Azure AD à vos partenaires professionnels.

-   Vos partenaires professionnels utilisent leurs propres informations de connexion, ce qui vous libère de la gestion d’un répertoire de partenaires externes et de la nécessité de supprimer l’accès lorsque des utilisateurs quittent l’organisation partenaire.

-   Vous gérez l’accès à vos applications indépendamment du cycle de vie des comptes de vos partenaires professionnels. Par exemple, cela signifie que vous pouvez révoquer l’accès sans que le service informatique de votre partenaire professionnel ne fasse quoi que ce soit.

## Fonctionnalités

La collaboration B2B simplifie la gestion et améliore la sécurité de l’accès des partenaires aux ressources d’entreprise, y compris aux applications SaaS, comme Office 365, Salesforce et les services Azure et toutes les applications mobiles, cloud et locales prenant en charge les revendications. La collaboration B2B permet aux partenaires de gérer leurs propres comptes, et les entreprises peuvent appliquer des stratégies de sécurité pour l’accès des partenaires.

La collaboration B2B Azure Active Directory est facile à configurer avec une inscription simplifiée pour les partenaires de toutes tailles, même s’ils ne disposent pas de leur propre Azure Active Directory via un processus de vérification de messagerie. La gestion est également facile, sans répertoires externes ni configurations de fédérations de partenaires.

Le processus :

1. La collaboration B2B Azure AD permet à un administrateur d’entreprise d’inviter et d’autoriser un ensemble d’utilisateurs externes en téléchargeant un fichier de valeurs séparées par des virgules (CSV) ne dépassant pas 2 000 lignes sur le portail de collaboration B2B.

  ![Boîte de dialogue de téléchargement de fichier CSV](./media/active-directory-b2b-collaboration-overview/upload-csv.png)

2. Le portail enverra des invitations par e-mail à ces utilisateurs externes.

3. L’utilisateur invité se connectera à un compte professionnel existant avec Microsoft (géré dans Azure AD) ou obtiendra un nouveau compte professionnel dans Azure AD.

4. Une fois connecté, l’utilisateur sera redirigé vers l’application qui a été partagée avec lui.

Les invitations envoyées aux adresses électroniques grand public (par exemple, Gmail ou [*comcast.net*](http://comcast.net/)) ne sont pas prises en charge pour le moment.

Pour plus d’informations sur le fonctionnement de la collaboration B2B, regardez [cette vidéo](http://aka.ms/aadshowb2b).

## Format de fichier CSV

Le fichier CSV suit le format ci-dessous. Ajoutez toutes les virgules requises, même si vous ne spécifiez pas d’options.

**Email :** adresse e-mail de l’utilisateur invité.<br/> **DisplayName :** nom d’affichage de l’utilisateur invité (en général, nom et prénom).<br/> **InviteAppID :** l’ID de l’application à utiliser pour personnaliser l’invitation par e-mail et les pages d’acceptation.<br/> **InviteReplyURL :** l’URL vers laquelle diriger un utilisateur invité après acceptation de l’invitation. Il doit s’agir d’une URL spécifique à la société (par exemple, [*contoso.my.salesforce.com*](http://contoso.my.salesforce.com/)). Si ce champ facultatif n'est pas spécifié, l’URL du Panneau d’accès de la société à l’origine de l’invitation est générée (cette URL est au format `https://account.activedirectory.windowsazure.com/applications/default.aspx?tenantId=<TenantID>`).<br/> **InviteAppResources :** ID d’application auxquels les applications peuvent affecter des utilisateurs. Les ID d’application sont récupérables en appelant `Get-MsolServicePrincipal | fl DisplayName, AppPrincipalId`<br/> **InviteGroupResources :** ID d’objet pour les groupes auxquels ajouter des utilisateurs. Les ID d’objet sont récupérables en appelant `Get-MsolGroup | fl DisplayName, ObjectId`<br/> **InviteContactUsUrl :** URL Nous contacter à inclure dans les invitations par e-mail au cas où l’utilisateur invité souhaiterait contacter votre organisation.<br/>

## Exemple de fichier CSV
Voici un exemple de fichier CSV que vous pouvez modifier selon vos besoins. Enregistrez-le sous le nom de fichier de votre choix, mais assurez-vous que son extension de fichier est .csv.

```
Email,DisplayName,InviteAppID,InviteReplyUrl,InviteAppResources,InviteGroupResources,InviteContactUsUrl
wharp@contoso.com,Walter Harp,cd3ed3de-93ee-400b-8b19-b61ef44a0f29,http://azure.microsoft.com/services/active-directory/,,,http://azure.microsoft.com/services/active-directory/
jsmith@contoso.com,Jeff Smith,cd3ed3de-93ee-400b-8b19-b61ef44a0f29,http://azure.microsoft.com/services/active-directory/,,,http://azure.microsoft.com/services/active-directory/
bsmith@contoso.com,Ben Smith,cd3ed3de-93ee-400b-8b19-b61ef44a0f29,http://azure.microsoft.com/services/active-directory/,,,http://azure.microsoft.com/services/active-directory/
```
## Étapes suivantes
Consultez les autres articles sur Azure B2B Collaboration

- [Qu’est-ce qu’Azure AD B2B Collaboration ?](active-directory-b2b-what-is-azure-ad-b2b.md)
- [Fonctionnement](active-directory-b2b-how-it-works.md)
- [Procédure pas à pas](active-directory-b2b-detailed-walkthrough.md)
- [Référence du format de fichier CSV](active-directory-b2b-references-csv-file-format.md)
- [Format du jeton utilisateur externe](active-directory-b2b-references-external-user-token-format.md)
- [Modifications de l’attribut d’objet utilisateur externe](active-directory-b2b-references-external-user-object-attribute-changes.md)
- [Limites actuelles de la version préliminaire](active-directory-b2b-current-preview-limitations.md)
- [Index d’articles pour la gestion des applications dans Azure Active Directory](active-directory-apps-index.md)

<!---HONumber=AcomDC_0211_2016-->