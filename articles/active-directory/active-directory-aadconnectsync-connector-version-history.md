<properties
   pageTitle="Synchronisation d’Azure AD Connect : historique de publication des versions du connecteur | Microsoft Azure"
   description="Cette rubrique répertorie toutes les versions des connecteurs de Forefront Identity Manager (FIM) et Microsoft Identity Manager (MIM)."
   services="active-directory"
   documentationCenter=""
   authors="AndKjell"
   manager="stevenpo"
   editor=""/>

<tags
   ms.service="active-directory"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="identity"
   ms.date="03/08/2016"
   ms.author="andkjell"/>

# Synchronisation d’Azure AD Connect : historique de publication des versions du connecteur
Les connecteurs de Forefront Identity Manager (FIM) et Microsoft Identity Manager (MIM) sont fréquemment mis à jour.

Cet article est conçu pour vous aider à conserver la trace des versions qui ont été publiées, et à comprendre si vous devez ou non effectuer la mise jour vers la version la plus récente.

Liens connexes :

- [Télécharger les derniers connecteurs](http://go.microsoft.com/fwlink/?LinkId=717495)
- Documentation de référence sur le [connecteur LDAP générique](active-directory-aadconnectsync-connector-genericldap.md)
- Documentation de référence sur le [connecteur SQL générique](active-directory-aadconnectsync-connector-genericsql.md)
- Documentation de référence sur le [connecteur des services web](http://go.microsoft.com/fwlink/?LinkID=226245)
- Documentation de référence sur le [connecteur PowerShell](active-directory-aadconnectsync-connector-powershell.md)
- Documentation de référence sur le [connecteur Lotus Domino](active-directory-aadconnectsync-connector-domino.md)

## 1\.1.117.0
Publié : mars 2016

**Nouveau connecteur** Version initiale du [connecteur SQL générique](active-directory-aadconnectsync-connector-genericsql.md).

**Nouvelles fonctionnalités :**

- Connecteur LDAP générique :
    - Prise en charge supplémentaire pour l’importation delta avec Isode.
- Connecteur des services web :
    - Mise à jour de l’activité csEntryChangeResult et de l’activité setImportErrorCode pour autoriser le renvoi des erreurs au niveau de l’objet vers le moteur de synchronisation.
    - Mise à jour des modèles SAP6 et SAP6User pour utiliser les nouvelles fonctionnalités d’erreur au niveau de l’objet.
- Connecteur Lotus Domino :
    - Lors de l’exportation, vous avez besoin d’une autorité de certification par carnet d’adresses. Vous pouvez désormais utiliser le même mot de passe pour toutes les autorités de certification pour simplifier la gestion.

**Problèmes résolus :**

- Connecteur LDAP générique :
    - Pour IBM Tivoli DS, certains attributs de référence n’étaient pas été détectés correctement.
    - Pour Open LDAP lors d’une importation delta, les espaces blancs au début et à la fin des chaînes étaient tronqués.
    - Pour Novell et NetIQ, une exportation qui déplaçait un objet entre des unités d’organisation/conteneurs et renommait simultanément l’objet, échouait.
- Connecteur des services web :
    - Si le service web avait plusieurs points de terminaison pour la même liaison, alors le connecteur ne détectait pas correctement ces points de terminaison.
- Connecteur Lotus Domino :
    - Une exportation de l’attribut fullName vers une Base courrier en arrivée ne fonctionnait pas.
    - Une exportation qui ajoutait et supprimait un membre d’un groupe exportait uniquement les membres ajoutés.
    - Si un document Notes n’est pas valide (attribut isValid défini sur false), alors le connecteur échoue.

## Versions plus anciennes
Avant mars 2016, les connecteurs étaient publiés sous forme de rubriques de prise en charge.

**LDAP générique**

- [KB3078617](https://support.microsoft.com/kb/3078617) - 1.0.0597, septembre 2015
- [KB3044896](https://support.microsoft.com/kb/3044896) - 1.0.0549, mars 2015
- [KB3031009](https://support.microsoft.com/kb/3031009) - 1.0.0534, janvier 2015
- [KB3008177](https://support.microsoft.com/kb/3008177) - 1.0.0419, septembre 2014
- [KB2936070](https://support.microsoft.com/kb/2936070) - 4.3.1082, mars 2014

**WebServices**

- [KB3008178](https://support.microsoft.com/kb/3008178) - 1.0.0419, septembre 2014

**PowerShell**

- [KB3008179](https://support.microsoft.com/kb/3008179) - 1.0.0419, septembre 2014

**Lotus Domino**

- [KB3096533](https://support.microsoft.com/kb/3096533) - 1.0.0597, septembre 2015
- [KB3044895](https://support.microsoft.com/kb/3044895) - 1.0.0549, mars 2015
- [KB2977286](https://support.microsoft.com/kb/2977286) - 5.3.0712, août 2014
- [KB2932635](https://support.microsoft.com/kb/2932635) - 5.3.1003, février 2014  
- [KB2899874](https://support.microsoft.com/kb/2899874) - 5.3.0721, octobre 2013
- [KB2875551](https://support.microsoft.com/kb/2875551) - 5.3.0534, août 2013

## Étapes suivantes
En savoir plus sur la configuration de la [synchronisation Azure AD Connect](active-directory-aadconnectsync-whatis.md).

En savoir plus sur l’[intégration de vos identités locales avec Azure Active Directory](active-directory-aadconnect.md).

<!---------HONumber=AcomDC_0309_2016-->