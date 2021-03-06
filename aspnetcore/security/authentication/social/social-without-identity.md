---
title: Authentification Facebook, Google et fournisseur externe sans ASP.NET CoreIdentity
author: rick-anderson
description: Explication de l’utilisation de l’authentification utilisateur Facebook, Google, Twitter, etc. sans IdentityASP.net core.
ms.author: riande
ms.date: 12/10/2019
no-loc:
- Blazor
- Identity
- Let's Encrypt
- Razor
- SignalR
uid: security/authentication/social/social-without-identity
ms.openlocfilehash: cc44eb83947540ca9a5a04ffad4fdb8522fab26a
ms.sourcegitcommit: 70e5f982c218db82aa54aa8b8d96b377cfc7283f
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 05/04/2020
ms.locfileid: "82775737"
---
# <a name="use-social-sign-in-provider-authentication-without-aspnet-core-identity"></a>Utiliser l’authentification du fournisseur de connexion sociale sans ASP.NET CoreIdentity

Par [Kirk Larkin](https://twitter.com/serpent5) et [Rick Anderson](https://twitter.com/RickAndMSFT)

::: moniker range=">= aspnetcore-3.0"

<xref:security/authentication/social/index>décrit comment permettre aux utilisateurs de se connecter à l’aide d’OAuth 2,0 avec les informations d’identification des fournisseurs d’authentification externes. L’approche décrite dans cette rubrique comprend des Identity ASP.net core en tant que fournisseur d’authentification.

Cet exemple montre comment utiliser un fournisseur d’authentification externe **sans** ASP.net Core Identity. Cela est utile pour les applications qui n’ont pas besoin de toutes les Identityfonctionnalités de ASP.net Core, mais nécessitent toujours une intégration avec un fournisseur d’authentification externe approuvé.

Cet exemple utilise [l’authentification Google](xref:security/authentication/google-logins) pour authentifier les utilisateurs. L’utilisation de l’authentification Google décale un grand nombre des complexités liées à la gestion du processus de connexion à Google. Pour une intégration avec un autre fournisseur d’authentification externe, consultez les rubriques suivantes :

* [Authentification Facebook](xref:security/authentication/facebook-logins)
* [Authentification Microsoft](xref:security/authentication/microsoft-logins)
* [Authentification Twitter](xref:security/authentication/twitter-logins)
* [Autres fournisseurs](xref:security/authentication/otherlogins)

## <a name="configuration"></a>Configuration

Dans la `ConfigureServices` méthode, configurez les schémas d’authentification de <xref:Microsoft.Extensions.DependencyInjection.AuthenticationServiceCollectionExtensions.AddAuthentication*>l' <xref:Microsoft.Extensions.DependencyInjection.CookieExtensions.AddCookie*>application avec <xref:Microsoft.Extensions.DependencyInjection.GoogleExtensions.AddGoogle*> les méthodes, et :

[!code-csharp[](social-without-identity/samples_snapshot/3.x/Startup.cs?name=snippet1)]

L’appel à <xref:Microsoft.Extensions.DependencyInjection.AuthenticationServiceCollectionExtensions.AddAuthentication*> définit le de <xref:Microsoft.AspNetCore.Authentication.AuthenticationOptions.DefaultScheme>l’application. `DefaultScheme` Est le schéma par défaut utilisé par les méthodes `HttpContext` d’extension d’authentification suivantes :

* <xref:Microsoft.AspNetCore.Authentication.AuthenticationHttpContextExtensions.AuthenticateAsync*>
* <xref:Microsoft.AspNetCore.Authentication.AuthenticationHttpContextExtensions.ChallengeAsync*>
* <xref:Microsoft.AspNetCore.Authentication.AuthenticationHttpContextExtensions.ForbidAsync*>
* <xref:Microsoft.AspNetCore.Authentication.AuthenticationHttpContextExtensions.SignInAsync*>
* <xref:Microsoft.AspNetCore.Authentication.AuthenticationHttpContextExtensions.SignOutAsync*>

La définition de l' `DefaultScheme` application sur [CookieAuthenticationDefaults. AuthenticationScheme](xref:Microsoft.AspNetCore.Authentication.Cookies.CookieAuthenticationDefaults.AuthenticationScheme) (« cookies ») configure l’application pour utiliser les cookies comme schéma par défaut pour ces méthodes d’extension. La définition de l' <xref:Microsoft.AspNetCore.Authentication.AuthenticationOptions.DefaultChallengeScheme> application sur [GoogleDefaults. AuthenticationScheme](xref:Microsoft.AspNetCore.Authentication.Google.GoogleDefaults.AuthenticationScheme) (« Google ») configure l’application pour utiliser Google comme modèle par défaut pour les appels `ChallengeAsync`à. `DefaultChallengeScheme`remplace `DefaultScheme`. Pour <xref:Microsoft.AspNetCore.Authentication.AuthenticationOptions> obtenir des propriétés supplémentaires qui remplacent `DefaultScheme` lorsqu’elles sont définies, consultez.

Dans `Startup.Configure`, appelez `UseAuthentication` et `UseAuthorization` entre appelant `UseRouting` et `UseEndpoints`. Cela définit la `HttpContext.User` propriété et exécute l’intergiciel (middleware) des autorisations pour les demandes :

[!code-csharp[](social-without-identity/samples_snapshot/3.x/Startup.cs?name=snippet2&highlight=3-4)]

Pour en savoir plus sur les schémas d’authentification, consultez [concepts d’authentification](xref:security/authentication/index#authentication-concepts). Pour en savoir plus sur l’authentification par <xref:security/authentication/cookie>cookie, consultez.

## <a name="apply-authorization"></a>Appliquer l’autorisation

Testez la configuration d’authentification de l’application `AuthorizeAttribute` en appliquant l’attribut à un contrôleur, une action ou une page. Le code suivant limite l’accès à la page de *confidentialité* aux utilisateurs qui ont été authentifiés :

[!code-csharp[](social-without-identity/samples_snapshot/3.x/Pages/Privacy.cshtml.cs?name=snippet&highlight=1)]

## <a name="sign-out"></a>Se déconnecter

Pour déconnecter l’utilisateur actuel et supprimer son cookie, appelez [SignOutAsync](xref:Microsoft.AspNetCore.Authentication.AuthenticationHttpContextExtensions.SignOutAsync*). Le code suivant ajoute un `Logout` gestionnaire de page à la page d' *index* :

[!code-csharp[](social-without-identity/samples_snapshot/3.x/Pages/Index.cshtml.cs?name=snippet&highlight=3-7)]

Notez que l’appel à `SignOutAsync` ne spécifie pas de schéma d’authentification. Le `DefaultScheme` de `CookieAuthenticationDefaults.AuthenticationScheme` l’application est utilisé comme un recul.

## <a name="additional-resources"></a>Ressources supplémentaires

* <xref:security/authorization/simple>
* <xref:security/authentication/social/additional-claims>

::: moniker-end
::: moniker range="< aspnetcore-3.0"

<xref:security/authentication/social/index>décrit comment permettre aux utilisateurs de se connecter à l’aide d’OAuth 2,0 avec les informations d’identification des fournisseurs d’authentification externes. L’approche décrite dans cette rubrique comprend des Identity ASP.net core en tant que fournisseur d’authentification.

Cet exemple montre comment utiliser un fournisseur d’authentification externe **sans** ASP.net Core Identity. Cela est utile pour les applications qui n’ont pas besoin de toutes les Identityfonctionnalités de ASP.net Core, mais nécessitent toujours une intégration avec un fournisseur d’authentification externe approuvé.

Cet exemple utilise [l’authentification Google](xref:security/authentication/google-logins) pour authentifier les utilisateurs. L’utilisation de l’authentification Google décale un grand nombre des complexités liées à la gestion du processus de connexion à Google. Pour une intégration avec un autre fournisseur d’authentification externe, consultez les rubriques suivantes :

* [Authentification Facebook](xref:security/authentication/facebook-logins)
* [Authentification Microsoft](xref:security/authentication/microsoft-logins)
* [Authentification Twitter](xref:security/authentication/twitter-logins)
* [Autres fournisseurs](xref:security/authentication/otherlogins)

## <a name="configuration"></a>Configuration

Dans la `ConfigureServices` méthode, configurez les schémas d’authentification de `AddAuthentication`l' `AddCookie`application avec `AddGoogle` les méthodes, et :

[!code-csharp[](social-without-identity/samples_snapshot/2.x/Startup.cs?name=snippet1)]

L’appel à [AddAuthentication](/dotnet/api/microsoft.extensions.dependencyinjection.authenticationservicecollectionextensions.addauthentication#Microsoft_Extensions_DependencyInjection_AuthenticationServiceCollectionExtensions_AddAuthentication_Microsoft_Extensions_DependencyInjection_IServiceCollection_System_Action_Microsoft_AspNetCore_Authentication_AuthenticationOptions__) définit le [DefaultScheme](xref:Microsoft.AspNetCore.Authentication.AuthenticationOptions.DefaultScheme)de l’application. `DefaultScheme` Est le schéma par défaut utilisé par les méthodes `HttpContext` d’extension d’authentification suivantes :

* <xref:Microsoft.AspNetCore.Authentication.AuthenticationHttpContextExtensions.AuthenticateAsync*>
* <xref:Microsoft.AspNetCore.Authentication.AuthenticationHttpContextExtensions.ChallengeAsync*>
* <xref:Microsoft.AspNetCore.Authentication.AuthenticationHttpContextExtensions.ForbidAsync*>
* <xref:Microsoft.AspNetCore.Authentication.AuthenticationHttpContextExtensions.SignInAsync*>
* <xref:Microsoft.AspNetCore.Authentication.AuthenticationHttpContextExtensions.SignOutAsync*>

La définition de l' `DefaultScheme` application sur [CookieAuthenticationDefaults. AuthenticationScheme](xref:Microsoft.AspNetCore.Authentication.Cookies.CookieAuthenticationDefaults.AuthenticationScheme) (« cookies ») configure l’application pour utiliser les cookies comme schéma par défaut pour ces méthodes d’extension. La définition de l' <xref:Microsoft.AspNetCore.Authentication.AuthenticationOptions.DefaultChallengeScheme> application sur [GoogleDefaults. AuthenticationScheme](xref:Microsoft.AspNetCore.Authentication.Google.GoogleDefaults.AuthenticationScheme) (« Google ») configure l’application pour utiliser Google comme modèle par défaut pour les appels `ChallengeAsync`à. `DefaultChallengeScheme`remplace `DefaultScheme`. Pour <xref:Microsoft.AspNetCore.Authentication.AuthenticationOptions> obtenir des propriétés supplémentaires qui remplacent `DefaultScheme` lorsqu’elles sont définies, consultez.

Dans la `Configure` méthode, appelez la `UseAuthentication` méthode pour appeler l’intergiciel (middleware) d’authentification `HttpContext.User` qui définit la propriété. Appelez la `UseAuthentication` méthode avant d' `UseMvcWithDefaultRoute` appeler `UseMvc`ou :

[!code-csharp[](social-without-identity/samples_snapshot/2.x/Startup.cs?name=snippet2)]

Pour en savoir plus sur les schémas d’authentification, consultez [concepts d’authentification](xref:security/authentication/index#authentication-concepts). Pour en savoir plus sur l’authentification par <xref:security/authentication/cookie>cookie, consultez.

## <a name="apply-authorization"></a>Appliquer l’autorisation

Testez la configuration d’authentification de l’application `AuthorizeAttribute` en appliquant l’attribut à un contrôleur, une action ou une page. Le code suivant limite l’accès à la page de *confidentialité* aux utilisateurs qui ont été authentifiés :

[!code-csharp[](social-without-identity/samples_snapshot/2.x/Pages/Privacy.cshtml.cs?name=snippet&highlight=1)]

## <a name="sign-out"></a>Se déconnecter

Pour déconnecter l’utilisateur actuel et supprimer son cookie, appelez [SignOutAsync](xref:Microsoft.AspNetCore.Authentication.AuthenticationHttpContextExtensions.SignOutAsync*). Le code suivant ajoute un `Logout` gestionnaire de page à la page d' *index* :

[!code-csharp[](social-without-identity/samples_snapshot/2.x/Pages/Index.cshtml.cs?name=snippet&highlight=3-7)]

Notez que l’appel à `SignOutAsync` ne spécifie pas de schéma d’authentification. Le `DefaultScheme` de `CookieAuthenticationDefaults.AuthenticationScheme` l’application est utilisé comme un recul.

## <a name="additional-resources"></a>Ressources supplémentaires

* <xref:security/authorization/simple>
* <xref:security/authentication/social/additional-claims>

::: moniker-end
