---
title: Configuration de la protection des données dans ASP.NET Core
author: rick-anderson
description: Consultez les rubriques qui expliquent comment configurer la protection des données dans ASP.NET Core.
ms.author: riande
ms.date: 10/12/2017
no-loc:
- Blazor
- Identity
- Let's Encrypt
- Razor
- SignalR
uid: security/data-protection/configuration/index
ms.openlocfilehash: 2d9330867357cecbf58be808cdfcb01b6bb25eb1
ms.sourcegitcommit: 70e5f982c218db82aa54aa8b8d96b377cfc7283f
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 05/04/2020
ms.locfileid: "82777434"
---
# <a name="data-protection-configuration-in-aspnet-core"></a>Configuration de la protection des données dans ASP.NET Core

Consultez les rubriques suivantes pour en savoir plus sur la configuration de la protection des données dans ASP.NET Core :

* [Configurer la protection des données ASP.NET Core](xref:security/data-protection/configuration/overview)  
  Vue d’ensemble de la configuration de la protection des données ASP.NET Core.

* [Gestion et durée de vie des clés de protection des données](xref:security/data-protection/configuration/default-settings)  
  Informations sur la gestion et la durée de vie des clés de protection des données.

* [Prise en charge de la stratégie au niveau de l’ordinateur de protection des données](xref:security/data-protection/configuration/machine-wide-policy)  
  Informations sur la définition d’une stratégie par défaut au niveau de l’ordinateur pour toutes les applications qui utilisent la protection des données.

* [Scénarios non compatibles avec l’injection de dépendances pour la protection des données dans ASP.NET Core](xref:security/data-protection/configuration/non-di-scenarios)  
  Indique comment utiliser le type concret [DataProtectionProvider](/dotnet/api/Microsoft.AspNetCore.DataProtection.DataProtectionProvider) pour utiliser la protection des données sans passer par des chemins de code spécifiques à l’injection de dépendances.
