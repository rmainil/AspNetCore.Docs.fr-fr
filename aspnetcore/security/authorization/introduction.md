---
title: Présentation de l’autorisation dans ASP.NET Core
author: rick-anderson
description: Découvrez les principes de base de l’autorisation et le fonctionnement de l’autorisation dans les applications ASP.NET Core.
ms.author: riande
ms.date: 10/14/2016
no-loc:
- Blazor
- Identity
- Let's Encrypt
- Razor
- SignalR
uid: security/authorization/introduction
ms.openlocfilehash: 241ef8b00e9dcbd1983d32edcd9c1db2eaa5c687
ms.sourcegitcommit: 70e5f982c218db82aa54aa8b8d96b377cfc7283f
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 05/04/2020
ms.locfileid: "82777525"
---
# <a name="introduction-to-authorization-in-aspnet-core"></a>Présentation de l’autorisation dans ASP.NET Core

<a name="security-authorization-introduction"></a>

L’autorisation fait référence au processus qui détermine ce qu’un utilisateur est en mesure de faire. Par exemple, un utilisateur administratif est autorisé à créer une bibliothèque de documents, à ajouter des documents, à modifier des documents et à les supprimer. Un utilisateur non-administrateur travaillant avec la bibliothèque est autorisé uniquement à lire les documents.

L’autorisation est orthogonale et indépendante de l’authentification. Toutefois, l’autorisation nécessite un mécanisme d’authentification. L’authentification est le processus qui consiste à déterminer l’identité d’un utilisateur. L’authentification peut créer une ou plusieurs identités pour l’utilisateur actuel.

Pour plus d’informations sur l’authentification dans ASP.NET Core <xref:security/authentication/index>, consultez.

## <a name="authorization-types"></a>Types d’autorisation

ASP.NET Core autorisation fournit un [rôle](xref:security/authorization/roles) simple, déclaratif et un modèle riche [basé sur des stratégies](xref:security/authorization/policies) . L’autorisation est exprimée en spécifications et les gestionnaires évaluent les revendications d’un utilisateur par rapport aux exigences. Les vérifications impératives peuvent reposer sur des stratégies ou des stratégies simples qui évaluent à la fois l’identité de l’utilisateur et les propriétés de la ressource à laquelle l’utilisateur tente d’accéder.

## <a name="namespaces"></a>Espaces de noms

Les composants d’autorisation, `AuthorizeAttribute` y `AllowAnonymousAttribute` compris les attributs et, se `Microsoft.AspNetCore.Authorization` trouvent dans l’espace de noms.

Consultez la documentation sur l' [autorisation simple](xref:security/authorization/simple).
