---
title: Autorisation avec un schéma spécifique dans ASP.NET Core
author: rick-anderson
description: Cet article explique comment limiter l’identité à un schéma spécifique lors de l’utilisation de plusieurs méthodes d’authentification.
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.date: 11/08/2019
no-loc:
- Blazor
- Identity
- Let's Encrypt
- Razor
- SignalR
uid: security/authorization/limitingidentitybyscheme
ms.openlocfilehash: 69b6412f249355573faa785743b124a67ecb8b9e
ms.sourcegitcommit: 70e5f982c218db82aa54aa8b8d96b377cfc7283f
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 05/04/2020
ms.locfileid: "82777512"
---
# <a name="authorize-with-a-specific-scheme-in-aspnet-core"></a>Autorisation avec un schéma spécifique dans ASP.NET Core

Dans certains scénarios, tels que les applications à page unique (SPAs), il est courant d’utiliser plusieurs méthodes d’authentification. Par exemple, l’application peut utiliser l’authentification basée sur les cookies pour se connecter et l’authentification du porteur JWT pour les demandes JavaScript. Dans certains cas, l’application peut avoir plusieurs instances d’un gestionnaire d’authentification. Par exemple, deux gestionnaires de cookies où l’un contient une identité de base et l’autre est créé lorsqu’une authentification multifacteur (MFA) a été déclenchée. L’authentification multifacteur peut être déclenchée, car l’utilisateur a demandé une opération nécessitant une sécurité supplémentaire. Pour plus d’informations sur l’application de l’authentification MFA lorsqu’un utilisateur demande une ressource qui requiert l’authentification MFA, consultez la section GitHub issue [protection avec MFA](https://github.com/dotnet/AspNetCore.Docs/issues/15791#issuecomment-580464195).

Un schéma d’authentification est nommé lorsque le service d’authentification est configuré pendant l’authentification. Par exemple :

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // Code omitted for brevity

    services.AddAuthentication()
        .AddCookie(options => {
            options.LoginPath = "/Account/Unauthorized/";
            options.AccessDeniedPath = "/Account/Forbidden/";
        })
        .AddJwtBearer(options => {
            options.Audience = "http://localhost:5001/";
            options.Authority = "http://localhost:5000/";
        });
```

Dans le code précédent, deux gestionnaires d’authentification ont été ajoutés : un pour les cookies et un pour le porteur.

>[!NOTE]
>La spécification du schéma par défaut entraîne `HttpContext.User` la définition de la propriété sur cette identité. Si ce comportement n’est pas souhaité, désactivez-le en appelant la `AddAuthentication`forme sans paramètre de.

## <a name="selecting-the-scheme-with-the-authorize-attribute"></a>Sélection du schéma avec l’attribut Authorize

Au point d’autorisation, l’application indique le gestionnaire à utiliser. Sélectionnez le gestionnaire avec lequel l’application autorisera en passant une liste de schémas d’authentification délimités par des `[Authorize]`virgules à. L' `[Authorize]` attribut spécifie le ou les schémas d’authentification à utiliser, qu’une valeur par défaut soit configurée ou non. Par exemple :

```csharp
[Authorize(AuthenticationSchemes = AuthSchemes)]
public class MixedController : Controller
    // Requires the following imports:
    // using Microsoft.AspNetCore.Authentication.Cookies;
    // using Microsoft.AspNetCore.Authentication.JwtBearer;
    private const string AuthSchemes =
        CookieAuthenticationDefaults.AuthenticationScheme + "," +
        JwtBearerDefaults.AuthenticationScheme;
```

Dans l’exemple précédent, les gestionnaires de cookies et de porteur s’exécutent et ont la possibilité de créer et d’ajouter une identité pour l’utilisateur actuel. En spécifiant un seul schéma, le gestionnaire correspondant s’exécute.

```csharp
[Authorize(AuthenticationSchemes = 
    JwtBearerDefaults.AuthenticationScheme)]
public class MixedController : Controller
```

Dans le code précédent, seul le gestionnaire avec le schéma « Bearer » s’exécute. Toutes les identités basées sur les cookies sont ignorées.

## <a name="selecting-the-scheme-with-policies"></a>Sélection du schéma avec des stratégies

Si vous préférez spécifier les schémas souhaités dans la [stratégie](xref:security/authorization/policies), vous pouvez définir `AuthenticationSchemes` le regroupement lors de l’ajout de votre stratégie :

```csharp
services.AddAuthorization(options =>
{
    options.AddPolicy("Over18", policy =>
    {
        policy.AuthenticationSchemes.Add(JwtBearerDefaults.AuthenticationScheme);
        policy.RequireAuthenticatedUser();
        policy.Requirements.Add(new MinimumAgeRequirement());
    });
});
```

Dans l’exemple précédent, la stratégie « 18 ans » s’exécute uniquement sur l’identité créée par le gestionnaire « porteur ». Utilisez la stratégie en définissant `[Authorize]` la propriété `Policy` de l’attribut :

```csharp
[Authorize(Policy = "Over18")]
public class RegistrationController : Controller
```

::: moniker range=">= aspnetcore-2.0"

## <a name="use-multiple-authentication-schemes"></a>Utiliser plusieurs schémas d’authentification

Certaines applications peuvent avoir besoin de prendre en charge plusieurs types d’authentification. Par exemple, votre application peut authentifier les utilisateurs à partir de Azure Active Directory et à partir d’une base de données utilisateurs. Un autre exemple est une application qui authentifie les utilisateurs à la fois Services ADFS et Azure Active Directory B2C. Dans ce cas, l’application doit accepter un jeton de porteur JWT de plusieurs émetteurs.

Ajoutez tous les schémas d’authentification que vous souhaitez accepter. Par exemple, le code suivant dans `Startup.ConfigureServices` ajoute deux schémas d’authentification de porteur JWT avec différents émetteurs :

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // Code omitted for brevity

    services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
        .AddJwtBearer(options =>
        {
            options.Audience = "https://localhost:5000/";
            options.Authority = "https://localhost:5000/identity/";
        })
        .AddJwtBearer("AzureAD", options =>
        {
            options.Audience = "https://localhost:5000/";
            options.Authority = "https://login.microsoftonline.com/eb971100-6f99-4bdc-8611-1bc8edd7f436/";
        });
}
```

> [!NOTE]
> Une seule authentification du porteur JWT est enregistrée avec le schéma `JwtBearerDefaults.AuthenticationScheme`d’authentification par défaut. Une authentification supplémentaire doit être inscrite avec un schéma d’authentification unique.

L’étape suivante consiste à mettre à jour la stratégie d’autorisation par défaut pour accepter les deux schémas d’authentification. Par exemple :

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // Code omitted for brevity

    services.AddAuthorization(options =>
    {
        var defaultAuthorizationPolicyBuilder = new AuthorizationPolicyBuilder(
            JwtBearerDefaults.AuthenticationScheme,
            "AzureAD");
        defaultAuthorizationPolicyBuilder = 
            defaultAuthorizationPolicyBuilder.RequireAuthenticatedUser();
        options.DefaultPolicy = defaultAuthorizationPolicyBuilder.Build();
    });
}
```

Étant donné que la stratégie d’autorisation par défaut est remplacée, il est possible `[Authorize]` d’utiliser l’attribut dans les contrôleurs. Le contrôleur accepte ensuite les demandes avec le jeton JWT émis par le premier ou le deuxième émetteur.

::: moniker-end
