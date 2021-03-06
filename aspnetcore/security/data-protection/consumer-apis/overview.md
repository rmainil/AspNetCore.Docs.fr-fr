---
title: Vue d’ensemble des API de consommateur pour ASP.NET Core
author: rick-anderson
description: Recevez une brève présentation des différentes API de consommateur disponibles dans la bibliothèque de protection des données ASP.NET Core.
ms.author: riande
ms.date: 06/11/2019
no-loc:
- Blazor
- Identity
- Let's Encrypt
- Razor
- SignalR
uid: security/data-protection/consumer-apis/overview
ms.openlocfilehash: e840111d702882a45e59cf89d679b87fa537d833
ms.sourcegitcommit: 70e5f982c218db82aa54aa8b8d96b377cfc7283f
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 05/04/2020
ms.locfileid: "82767855"
---
# <a name="consumer-apis-overview-for-aspnet-core"></a>Vue d’ensemble des API de consommateur pour ASP.NET Core

Les `IDataProtectionProvider` interfaces `IDataProtector` et sont les interfaces de base par le biais desquelles les consommateurs utilisent le système de protection des données. Ils se trouvent dans le package [Microsoft. AspNetCore. dataprotection. Abstracts](https://www.nuget.org/packages/Microsoft.AspNetCore.DataProtection.Abstractions/) .

## <a name="idataprotectionprovider"></a>IDataProtectionProvider

L’interface du fournisseur représente la racine du système de protection des données. Il ne peut pas être utilisé directement pour protéger ou ôter la protection des données. Au lieu de cela, le consommateur doit obtenir une `IDataProtector` référence à `IDataProtectionProvider.CreateProtector(purpose)`un en appelant, où objet est une chaîne qui décrit le cas d’usage consommateur prévu. Pour plus d’informations sur l’intention de ce paramètre et sur la manière de choisir une valeur appropriée, consultez [chaînes d’objectif](xref:security/data-protection/consumer-apis/purpose-strings) .

## <a name="idataprotector"></a>IDataProtector

L’interface de protecteur est retournée par un `CreateProtector`appel à, et c’est cette interface que les consommateurs peuvent utiliser pour effectuer des opérations de protection et d’ôter la protection.

Pour protéger un élément de données, transmettez les données à `Protect` la méthode. L’interface de base définit une méthode qui convertit Byte []-> Byte [], mais il existe également une surcharge (fournie en tant que méthode d’extension) qui convertit la chaîne > chaîne. La sécurité offerte par les deux méthodes est identique ; le développeur doit choisir la surcharge la plus pratique pour le cas d’usage. Quelle que soit la surcharge choisie, la valeur retournée par la méthode Protect est désormais protégée (chiffrée et inviolable) et l’application peut l’envoyer à un client non approuvé.

Pour ôter la protection d’un élément de données précédemment protégé, transmettez les données protégées `Unprotect` à la méthode. (Il existe des surcharges basées sur un octet [] et une chaîne pour faciliter le développement.) Si la charge utile protégée a été générée par un appel `Protect` antérieur à sur `IDataProtector`ce même `Unprotect` , la méthode retourne la charge utile non protégée d’origine. Si la charge utile protégée a été falsifiée ou a été produite par un `IDataProtector`différent, `Unprotect` la méthode lèvera CryptographicException.

Le concept des mêmes différences par rapport `IDataProtector` à la notion de rôle est différent. Si deux `IDataProtector` instances ont été générées à partir `IDataProtectionProvider` de la même racine mais par le biais de `IDataProtectionProvider.CreateProtector`chaînes d’objectif différentes dans l’appel à, elles sont considérées comme des [protecteurs différents](xref:security/data-protection/consumer-apis/purpose-strings)et l’une ne peut pas ôter la protection des charges utiles générées par l’autre.

## <a name="consuming-these-interfaces"></a>Utilisation de ces interfaces

Pour un composant de DI-Aware, l’utilisation prévue est que le composant utilise `IDataProtectionProvider` un paramètre dans son constructeur et que le système di fournit automatiquement ce service lorsque le composant est instancié.

> [!NOTE]
> Certaines applications (telles que les applications console ou les applications ASP.NET 4. x) peuvent ne pas être compatibles avec la transprise en charge. par conséquent, il n’est pas possible d’utiliser le mécanisme décrit ici. Pour ces scénarios, consultez le document relatif aux [scénarios sans di-Aware](xref:security/data-protection/configuration/non-di-scenarios) pour plus d’informations sur `IDataProtection` l’obtention d’une instance d’un fournisseur sans passer par di.

L’exemple suivant illustre trois concepts :

1. [Ajoutez le système de protection des données](xref:security/data-protection/configuration/overview) au conteneur de service,

2. Utilisation de DI pour recevoir une instance d' `IDataProtectionProvider`un et

3. Créer un `IDataProtector` à partir `IDataProtectionProvider` d’un et l’utiliser pour protéger et ôter la protection des données.

[!code-csharp[](../using-data-protection/samples/protectunprotect.cs?highlight=26,34,35,36,37,38,39,40)]

Le package Microsoft. AspNetCore. DataProtection. Abstracts contient une méthode `IServiceProvider.GetDataProtector` d’extension en tant que développeur. Il encapsule comme une opération unique en extrayant un `IDataProtectionProvider` à partir du fournisseur de services et `IDataProtectionProvider.CreateProtector`en appelant. L’exemple suivant illustre son utilisation.

[!code-csharp[](./overview/samples/getdataprotector.cs?highlight=15)]

>[!TIP]
> Les instances `IDataProtectionProvider` de `IDataProtector` et de sont thread-safe pour plusieurs appelants. Il est prévu qu’une fois qu’un composant obtient une référence `IDataProtector` à un via un `CreateProtector`appel à, il utilise cette référence pour plusieurs appels `Protect` à `Unprotect`et. Un appel à `Unprotect` lèvera CryptographicException si la charge utile protégée ne peut pas être vérifiée ou déchiffrée. Certains composants peuvent souhaiter ignorer des erreurs lors de l’opération de non-protection. un composant qui lit les cookies d’authentification peut gérer cette erreur et traiter la demande comme s’il n’avait aucun cookie au lieu de faire échouer la demande. Les composants qui veulent ce comportement doivent intercepter spécifiquement CryptographicException au lieu d’absorber toutes les exceptions.
