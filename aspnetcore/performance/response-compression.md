---
title: Compression des réponses en ASP.NET Core
author: rick-anderson
description: Découvrez ce qu’est la compression des réponses et comment utiliser le middleware de compression des réponses dans les applications ASP.NET Core.
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 02/07/2020
no-loc:
- Blazor
- Identity
- Let's Encrypt
- Razor
- SignalR
uid: performance/response-compression
ms.openlocfilehash: 12a39ccfefdcaec6251a9804011aefde3bbae7b2
ms.sourcegitcommit: 70e5f982c218db82aa54aa8b8d96b377cfc7283f
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 05/04/2020
ms.locfileid: "82776667"
---
# <a name="response-compression-in-aspnet-core"></a>Compression des réponses en ASP.NET Core

::: moniker range=">= aspnetcore-3.0"

La bande passante réseau est une ressource limitée. La réduction de la taille de la réponse augmente généralement la réactivité d’une application, souvent considérablement. Une façon de réduire les tailles de charge utile consiste à compresser les réponses d’une application.

[Afficher ou télécharger l’exemple de code](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/performance/response-compression/samples) ([procédure de téléchargement](xref:index#how-to-download-a-sample))

## <a name="when-to-use-response-compression-middleware"></a>Quand utiliser l’intergiciel (middleware) de compression des réponses

Utilisez les technologies de compression des réponses basées sur le serveur dans IIS, Apache ou nginx. Les performances de l’intergiciel (middleware) ne correspondront probablement pas à celles des modules de serveur. Le serveur de serveur [http. sys](xref:fundamentals/servers/httpsys) et le serveur [Kestrel](xref:fundamentals/servers/kestrel) n’offrent pas de prise en charge intégrée de la compression.

Utilisez l’intergiciel (middleware) de compression des réponses lorsque vous êtes :

* Impossible d’utiliser les technologies de compression basées sur le serveur suivantes :
  * [Module de compression dynamique IIS](https://www.iis.net/overview/reliability/dynamiccachingandcompression)
  * [Module Apache mod_deflate](https://httpd.apache.org/docs/current/mod/mod_deflate.html)
  * [Compression et décompression Nginx](https://www.nginx.com/resources/admin-guide/compression-and-decompression/)
* Hébergement direct :
  * [Serveur http. sys](xref:fundamentals/servers/httpsys) (anciennement appelé webListener)
  * [Serveur Kestrel](xref:fundamentals/servers/kestrel)

## <a name="response-compression"></a>Compression des réponses

En règle générale, toute réponse qui n’est pas compressée en mode natif peut tirer parti de la compression des réponses. Les réponses qui ne sont pas compressées en mode natif incluent généralement : CSS, JavaScript, HTML, XML et JSON. Vous ne devez pas compresser les ressources natives compressées, telles que les fichiers PNG. Si vous tentez de compresser davantage une réponse compressée en mode natif, toute petite réduction supplémentaire de taille et de temps de transmission sera probablement survenue du temps nécessaire au traitement de la compression. Ne compressez pas les fichiers d’une taille inférieure à environ 150-1000 octets (en fonction du contenu du fichier et de l’efficacité de la compression). La surcharge liée à la compression de petits fichiers peut entraîner la création d’un fichier compressé plus volumineux que le fichier non compressé.

Lorsqu’un client peut traiter du contenu compressé, le client doit informer le serveur de ses fonctionnalités en envoyant `Accept-Encoding` l’en-tête avec la demande. Lorsqu’un serveur envoie du contenu compressé, il doit inclure des informations `Content-Encoding` dans l’en-tête sur la manière dont la réponse compressée est encodée. Les désignations de codage de contenu prises en charge par l’intergiciel (middleware) sont indiquées dans le tableau suivant.

| `Accept-Encoding`valeurs d’en-tête | Intergiciel pris en charge | Description |
| ------------------------------- | :------------------: | ----------- |
| `br`                            | Oui (valeur par défaut)        | [Format de données compressées Brotli](https://tools.ietf.org/html/rfc7932) |
| `deflate`                       | Non                   | [Format de données compressées compressé](https://tools.ietf.org/html/rfc1951) |
| `exi`                           | Non                   | [Échange XML efficace W3C](https://tools.ietf.org/id/draft-varga-netconf-exi-capability-00.html) |
| `gzip`                          | Oui                  | [Format de fichier gzip](https://tools.ietf.org/html/rfc1952) |
| `identity`                      | Oui                  | Identificateur « no Encoding » : la réponse ne doit pas être encodée. |
| `pack200-gzip`                  | Non                   | [Format de transfert réseau pour les archives Java](https://jcp.org/aboutJava/communityprocess/review/jsr200/index.html) |
| `*`                             | Oui                  | Tout encodage de contenu disponible qui n’est pas explicitement demandé |

Pour plus d’informations, consultez la [liste de codage de contenu officielle IANA](https://www.iana.org/assignments/http-parameters/http-parameters.xml#http-content-coding-registry).

L’intergiciel (middleware) vous permet d’ajouter des fournisseurs de `Accept-Encoding` compression supplémentaires pour les valeurs d’en-tête personnalisées. Pour plus d’informations, consultez [fournisseurs personnalisés](#custom-providers) ci-dessous.

L’intergiciel (middleware `q`) peut réagir à la valeur de qualité (qvalue) pondérée lorsqu’il est envoyé par le client pour hiérarchiser les schémas de compression. Pour plus d’informations, consultez [RFC 7231 : Accept-Encoding](https://tools.ietf.org/html/rfc7231#section-5.3.4).

Les algorithmes de compression sont soumis à un compromis entre la vitesse de compression et l’efficacité de la compression. L' *efficacité* dans ce contexte fait référence à la taille de la sortie après compression. La plus petite taille est obtenue par la compression la plus *optimale* .

Les en-têtes impliqués dans la demande, l’envoi, la mise en cache et la réception de contenu compressé sont décrits dans le tableau ci-dessous.

| En-tête             | Role |
| ------------------ | ---- |
| `Accept-Encoding`  | Envoyée du client au serveur pour indiquer les schémas de codage de contenu acceptables pour le client. |
| `Content-Encoding` | Envoyé du serveur au client pour indiquer l’encodage du contenu dans la charge utile. |
| `Content-Length`   | En cas de compression, `Content-Length` l’en-tête est supprimé, car le contenu du corps change lorsque la réponse est compressée. |
| `Content-MD5`      | En cas de compression, `Content-MD5` l’en-tête est supprimé, car le contenu du corps a changé et le hachage n’est plus valide. |
| `Content-Type`     | Spécifie le type MIME du contenu. Chaque réponse doit spécifier son `Content-Type`. L’intergiciel vérifie cette valeur pour déterminer si la réponse doit être compressée. L’intergiciel (middleware) spécifie un ensemble de [types MIME par défaut](#mime-types) qu’il peut encoder, mais vous pouvez remplacer ou ajouter des types MIME. |
| `Vary`             | Lorsqu’il est envoyé par le serveur avec une `Accept-Encoding` valeur à clients et proxys, `Vary` l’en-tête indique au client ou au proxy qu’il doit mettre en cache (variation) les réponses `Accept-Encoding` en fonction de la valeur de l’en-tête de la demande. Le résultat de la restitution du contenu `Vary: Accept-Encoding` avec l’en-tête est que les réponses compressées et non compressées sont mises en cache séparément. |

Explorez les fonctionnalités de l’intergiciel de compression des réponses avec l' [exemple d’application](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/performance/response-compression/samples). L’exemple illustre les éléments suivants :

* La compression des réponses de l’application à l’aide de gzip et des fournisseurs de compression personnalisés.
* Comment ajouter un type MIME à la liste par défaut des types MIME pour la compression.

## <a name="package"></a>Package

L’intergiciel (middleware) de compression des réponses est fourni par le package [Microsoft. AspNetCore. ResponseCompression](https://www.nuget.org/packages/Microsoft.AspNetCore.ResponseCompression/) , qui est implicitement inclus dans les applications ASP.net core.

## <a name="configuration"></a>Configuration

Le code suivant montre comment activer l’intergiciel (middleware) de compression des réponses pour les types MIME et les fournisseurs de compression par défaut ([Brotli](#brotli-compression-provider) et [gzip](#gzip-compression-provider)) :

```csharp
public class Startup
{
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddResponseCompression();
    }

    public void Configure(IApplicationBuilder app, IHostingEnvironment env)
    {
        app.UseResponseCompression();
    }
}
```

Remarques :

* `app.UseResponseCompression`doit être appelé avant tout middleware qui compresse les réponses. Pour plus d’informations, consultez <xref:fundamentals/middleware/index#middleware-order>.
* Utilisez un outil tel que [Fiddler](https://www.telerik.com/fiddler), [Firebug](https://getfirebug.com/)ou [postal](https://www.getpostman.com/) pour définir l’en- `Accept-Encoding` tête de la demande et étudier les en-têtes, la taille et le corps de la réponse.

Envoyez une demande à l’exemple d’application sans `Accept-Encoding` l’en-tête et observez que la réponse n’est pas compressée. Les `Content-Encoding` en `Vary` -têtes et ne sont pas présents dans la réponse.

![Fenêtre Fiddler qui indique le résultat d’une demande sans l’en-tête d’encodage Accept. La réponse n’est pas compressée.](response-compression/_static/request-uncompressed.png)

Envoyez une demande à l’exemple d’application avec `Accept-Encoding: br` l’en-tête (compression Brotli) et observez que la réponse est compressée. Les `Content-Encoding` en `Vary` -têtes et sont présents dans la réponse.

![Fenêtre Fiddler présentant le résultat d’une demande avec l’en-tête Accept-Encoding et la valeur br. Les en-têtes Vary et encodage de contenu sont ajoutés à la réponse. La réponse est compressée.](response-compression/_static/request-compressed-br.png)

## <a name="providers"></a>Fournisseurs

### <a name="brotli-compression-provider"></a>Fournisseur de compression Brotli

Utilisez le <xref:Microsoft.AspNetCore.ResponseCompression.BrotliCompressionProvider> pour compresser les réponses avec le [format de données compressées Brotli](https://tools.ietf.org/html/rfc7932).

Si aucun fournisseur de compression n’est explicitement ajouté <xref:Microsoft.AspNetCore.ResponseCompression.CompressionProviderCollection>à :

* Le fournisseur de compression Brotli est ajouté par défaut au groupe de fournisseurs de compression avec le [fournisseur de compression gzip](#gzip-compression-provider).
* La compression prend par défaut la compression Brotli lorsque le format de données compressées Brotli est pris en charge par le client. Si Brotli n’est pas pris en charge par le client, la compression est définie par défaut sur gzip lorsque le client prend en charge la compression gzip.

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddResponseCompression();
}
```

Le fournisseur de compression Brotli doit être ajouté lors de l’ajout explicite de fournisseurs de compression :

[!code-csharp[](response-compression/samples/3.x/SampleApp/Startup.cs?name=snippet1&highlight=5)]

Définissez le niveau de compression <xref:Microsoft.AspNetCore.ResponseCompression.BrotliCompressionProviderOptions>avec. Par défaut, le fournisseur de compression Brotli a le niveau de compression le plus rapide ([CompressionLevel. plus rapide](xref:System.IO.Compression.CompressionLevel)), qui peut ne pas produire la compression la plus efficace. Si vous souhaitez obtenir la compression la plus efficace possible, configurez l’intergiciel (middleware) pour une compression optimale.

| Niveau de compression | Description |
| ----------------- | ----------- |
| [CompressionLevel. plus rapide](xref:System.IO.Compression.CompressionLevel) | La compression doit être effectuée aussi rapidement que possible, même si la sortie obtenue n’est pas compressée de manière optimale. |
| [CompressionLevel. NoCompression](xref:System.IO.Compression.CompressionLevel) | Aucune compression ne doit être effectuée. |
| [CompressionLevel. optimal](xref:System.IO.Compression.CompressionLevel) | Les réponses doivent être compressées de façon optimale, même si la compression prend plus de temps. |

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddResponseCompression();

    services.Configure<BrotliCompressionProviderOptions>(options => 
    {
        options.Level = CompressionLevel.Fastest;
    });
}
```

### <a name="gzip-compression-provider"></a>Fournisseur de compression gzip

Utilisez <xref:Microsoft.AspNetCore.ResponseCompression.GzipCompressionProvider> pour compresser les réponses avec le [format de fichier gzip](https://tools.ietf.org/html/rfc1952).

Si aucun fournisseur de compression n’est explicitement ajouté <xref:Microsoft.AspNetCore.ResponseCompression.CompressionProviderCollection>à :

* Le fournisseur de compression gzip est ajouté par défaut au tableau de fournisseurs de compression avec le [fournisseur de compression Brotli](#brotli-compression-provider).
* La compression prend par défaut la compression Brotli lorsque le format de données compressées Brotli est pris en charge par le client. Si Brotli n’est pas pris en charge par le client, la compression est définie par défaut sur gzip lorsque le client prend en charge la compression gzip.

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddResponseCompression();
}
```

Le fournisseur de compression gzip doit être ajouté lors de l’ajout explicite de fournisseurs de compression :

[!code-csharp[](response-compression/samples/3.x/SampleApp/Startup.cs?name=snippet1&highlight=6)]

Définissez le niveau de compression <xref:Microsoft.AspNetCore.ResponseCompression.GzipCompressionProviderOptions>avec. Par défaut, le fournisseur de compression gzip a le niveau de compression le plus rapide ([CompressionLevel. plus rapide](xref:System.IO.Compression.CompressionLevel)), qui peut ne pas produire la compression la plus efficace. Si vous souhaitez obtenir la compression la plus efficace possible, configurez l’intergiciel (middleware) pour une compression optimale.

| Niveau de compression | Description |
| ----------------- | ----------- |
| [CompressionLevel. plus rapide](xref:System.IO.Compression.CompressionLevel) | La compression doit être effectuée aussi rapidement que possible, même si la sortie obtenue n’est pas compressée de manière optimale. |
| [CompressionLevel. NoCompression](xref:System.IO.Compression.CompressionLevel) | Aucune compression ne doit être effectuée. |
| [CompressionLevel. optimal](xref:System.IO.Compression.CompressionLevel) | Les réponses doivent être compressées de façon optimale, même si la compression prend plus de temps. |

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddResponseCompression();

    services.Configure<GzipCompressionProviderOptions>(options => 
    {
        options.Level = CompressionLevel.Fastest;
    });
}
```

### <a name="custom-providers"></a>Fournisseurs personnalisés

Créer des implémentations de compression <xref:Microsoft.AspNetCore.ResponseCompression.ICompressionProvider>personnalisées avec. <xref:Microsoft.AspNetCore.ResponseCompression.ICompressionProvider.EncodingName*> Représente l’encodage de contenu que `ICompressionProvider` ce produit. L’intergiciel utilise ces informations pour choisir le fournisseur en fonction de la liste spécifiée dans l' `Accept-Encoding` en-tête de la demande.

À l’aide de l’exemple d’application, le client envoie une `Accept-Encoding: mycustomcompression` demande avec l’en-tête. L’intergiciel utilise l’implémentation de compression personnalisée et retourne la réponse avec un `Content-Encoding: mycustomcompression` en-tête. Le client doit être en mesure de décompresser l’encodage personnalisé pour qu’une implémentation de compression personnalisée fonctionne.

[!code-csharp[](response-compression/samples/3.x/SampleApp/Startup.cs?name=snippet1&highlight=7)]

[!code-csharp[](response-compression/samples/3.x/SampleApp/CustomCompressionProvider.cs?name=snippet1)]


Envoyez une demande à l’exemple d’application avec `Accept-Encoding: mycustomcompression` l’en-tête et observez les en-têtes de réponse. Les `Vary` en `Content-Encoding` -têtes et sont présents dans la réponse. Le corps de la réponse (non affiché) n’est pas compressé par l’exemple. Il n’existe pas d’implémentation de `CustomCompressionProvider` compression dans la classe de l’exemple. Toutefois, l’exemple illustre l’emplacement où vous implémentez un tel algorithme de compression.

![Fenêtre Fiddler présentant le résultat d’une demande avec l’en-tête Accept-Encoding et une valeur de mycustomcompression. Les en-têtes Vary et encodage de contenu sont ajoutés à la réponse.](response-compression/_static/request-custom-compression.png)

## <a name="mime-types"></a>types MIME

L’intergiciel (middleware) spécifie un ensemble de types MIME par défaut pour la compression :

* `application/javascript`
* `application/json`
* `application/xml`
* `text/css`
* `text/html`
* `text/json`
* `text/plain`
* `text/xml`

Remplacez ou ajoutez des types MIME par les options de l’intergiciel (middleware) de compression des réponses. Notez que les types MIME génériques, tels `text/*` que ne sont pas pris en charge. L’exemple d’application ajoute un type MIME `image/svg+xml` pour et compresse et sert le ASP.net Core image de bannière (*Banner. svg*).

[!code-csharp[](response-compression/samples/3.x/SampleApp/Startup.cs?name=snippet1&highlight=8-10)]

## <a name="compression-with-secure-protocol"></a>Compression avec protocole sécurisé

Les réponses compressées sur des connexions sécurisées `EnableForHttps` peuvent être contrôlées avec l’option, qui est désactivée par défaut. L’utilisation de la compression avec des pages générées dynamiquement peut entraîner des problèmes de sécurité tels que les attaques de [criminalité](https://wikipedia.org/wiki/CRIME_(security_exploit)) et de [violation](https://wikipedia.org/wiki/BREACH_(security_exploit)) .

## <a name="adding-the-vary-header"></a>Ajout de l’en-tête Vary

Lors de la compression des réponses basées sur `Accept-Encoding` l’en-tête, il existe potentiellement plusieurs versions compressées de la réponse et une version non compressée. Pour indiquer au client et aux caches proxy que plusieurs versions existent et doivent être stockées, l' `Vary` en-tête est ajouté `Accept-Encoding` avec une valeur. Dans ASP.NET Core 2,0 ou version ultérieure, l’intergiciel ajoute automatiquement `Vary` l’en-tête lorsque la réponse est compressée.

## <a name="middleware-issue-when-behind-an-nginx-reverse-proxy"></a>Problème d’intergiciel lors de l’arrière-plan d’un proxy inverse Nginx

Lorsqu’une demande est traitée par un proxy par Nginx `Accept-Encoding` , l’en-tête est supprimé. La suppression de `Accept-Encoding` l’en-tête empêche l’intergiciel de compresser la réponse. Pour plus d’informations, consultez [Nginx : compression et décompression](https://www.nginx.com/resources/admin-guide/compression-and-decompression/). Ce problème est suivi par la [compression directe pour Nginx (ASPNET/BasicMiddleware #123)](https://github.com/aspnet/BasicMiddleware/issues/123).

## <a name="working-with-iis-dynamic-compression"></a>Utilisation de la compression dynamique IIS

Si vous disposez d’un module de compression dynamique IIS configuré au niveau du serveur que vous souhaitez désactiver pour une application, désactivez le module avec un ajout au fichier *Web. config* . Pour plus d’informations, consultez [Désactivation de modules IIS](xref:host-and-deploy/iis/modules#disabling-iis-modules).

## <a name="troubleshooting"></a>Dépannage

Utilisez un outil tel que [Fiddler](https://www.telerik.com/fiddler), [Firebug](https://getfirebug.com/)ou [postal](https://www.getpostman.com/), qui vous permet de définir l’en `Accept-Encoding` -tête de la demande et d’étudier les en-têtes, la taille et le corps de la réponse. Par défaut, l’intergiciel (middleware) de compression des réponses compresse les réponses qui remplissent les conditions suivantes :

* L' `Accept-Encoding` en-tête est présent avec une `br`valeur `gzip`, `*`, ou un encodage personnalisé qui correspond à un fournisseur de compression personnalisé que vous avez établi. La valeur ne doit pas `identity` être ou avoir une valeur de qualité ( `q`qvalue,) égale à 0 (zéro).
* Le type MIME (`Content-Type`) doit être défini et doit correspondre à un type MIME configuré sur <xref:Microsoft.AspNetCore.ResponseCompression.ResponseCompressionOptions>le.
* La requête ne doit pas inclure `Content-Range` l’en-tête.
* La demande doit utiliser le protocole http, sauf si le protocole sécurisé (https) est configuré dans les options de l’intergiciel (middleware) de compression des réponses. *Notez le danger [décrit ci-dessus](#compression-with-secure-protocol) lors de l’activation de la compression de contenu sécurisé.*

## <a name="additional-resources"></a>Ressources supplémentaires

* <xref:fundamentals/startup>
* <xref:fundamentals/middleware/index>
* [Réseau des développeurs Mozilla : accepter-encodage](https://developer.mozilla.org/docs/Web/HTTP/Headers/Accept-Encoding)
* [RFC 7231 section 3.1.2.1 : codages de contenu](https://tools.ietf.org/html/rfc7231#section-3.1.2.1)
* [RFC 7230, section 4.2.3 : codage gzip](https://tools.ietf.org/html/rfc7230#section-4.2.3)
* [Spécification de format de fichier GZIP version 4,3](https://www.ietf.org/rfc/rfc1952.txt)

::: moniker-end

::: moniker range="= aspnetcore-2.2"

La bande passante réseau est une ressource limitée. La réduction de la taille de la réponse augmente généralement la réactivité d’une application, souvent considérablement. Une façon de réduire les tailles de charge utile consiste à compresser les réponses d’une application.

[Afficher ou télécharger l’exemple de code](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/performance/response-compression/samples) ([procédure de téléchargement](xref:index#how-to-download-a-sample))

## <a name="when-to-use-response-compression-middleware"></a>Quand utiliser l’intergiciel (middleware) de compression des réponses

Utilisez les technologies de compression des réponses basées sur le serveur dans IIS, Apache ou nginx. Les performances de l’intergiciel (middleware) ne correspondront probablement pas à celles des modules de serveur. Le serveur de serveur [http. sys](xref:fundamentals/servers/httpsys) et le serveur [Kestrel](xref:fundamentals/servers/kestrel) n’offrent pas de prise en charge intégrée de la compression.

Utilisez l’intergiciel (middleware) de compression des réponses lorsque vous êtes :

* Impossible d’utiliser les technologies de compression basées sur le serveur suivantes :
  * [Module de compression dynamique IIS](https://www.iis.net/overview/reliability/dynamiccachingandcompression)
  * [Module Apache mod_deflate](https://httpd.apache.org/docs/current/mod/mod_deflate.html)
  * [Compression et décompression Nginx](https://www.nginx.com/resources/admin-guide/compression-and-decompression/)
* Hébergement direct :
  * [Serveur http. sys](xref:fundamentals/servers/httpsys) (anciennement appelé webListener)
  * [Serveur Kestrel](xref:fundamentals/servers/kestrel)

## <a name="response-compression"></a>Compression des réponses

En règle générale, toute réponse qui n’est pas compressée en mode natif peut tirer parti de la compression des réponses. Les réponses qui ne sont pas compressées en mode natif incluent généralement : CSS, JavaScript, HTML, XML et JSON. Vous ne devez pas compresser les ressources natives compressées, telles que les fichiers PNG. Si vous tentez de compresser davantage une réponse compressée en mode natif, toute petite réduction supplémentaire de taille et de temps de transmission sera probablement survenue du temps nécessaire au traitement de la compression. Ne compressez pas les fichiers d’une taille inférieure à environ 150-1000 octets (en fonction du contenu du fichier et de l’efficacité de la compression). La surcharge liée à la compression de petits fichiers peut entraîner la création d’un fichier compressé plus volumineux que le fichier non compressé.

Lorsqu’un client peut traiter du contenu compressé, le client doit informer le serveur de ses fonctionnalités en envoyant `Accept-Encoding` l’en-tête avec la demande. Lorsqu’un serveur envoie du contenu compressé, il doit inclure des informations `Content-Encoding` dans l’en-tête sur la manière dont la réponse compressée est encodée. Les désignations de codage de contenu prises en charge par l’intergiciel (middleware) sont indiquées dans le tableau suivant.

| `Accept-Encoding`valeurs d’en-tête | Intergiciel pris en charge | Description |
| ------------------------------- | :------------------: | ----------- |
| `br`                            | Oui (valeur par défaut)        | [Format de données compressées Brotli](https://tools.ietf.org/html/rfc7932) |
| `deflate`                       | Non                   | [Format de données compressées compressé](https://tools.ietf.org/html/rfc1951) |
| `exi`                           | Non                   | [Échange XML efficace W3C](https://tools.ietf.org/id/draft-varga-netconf-exi-capability-00.html) |
| `gzip`                          | Oui                  | [Format de fichier gzip](https://tools.ietf.org/html/rfc1952) |
| `identity`                      | Oui                  | Identificateur « no Encoding » : la réponse ne doit pas être encodée. |
| `pack200-gzip`                  | Non                   | [Format de transfert réseau pour les archives Java](https://jcp.org/aboutJava/communityprocess/review/jsr200/index.html) |
| `*`                             | Oui                  | Tout encodage de contenu disponible qui n’est pas explicitement demandé |

Pour plus d’informations, consultez la [liste de codage de contenu officielle IANA](https://www.iana.org/assignments/http-parameters/http-parameters.xml#http-content-coding-registry).

L’intergiciel (middleware) vous permet d’ajouter des fournisseurs de `Accept-Encoding` compression supplémentaires pour les valeurs d’en-tête personnalisées. Pour plus d’informations, consultez [fournisseurs personnalisés](#custom-providers) ci-dessous.

L’intergiciel (middleware `q`) peut réagir à la valeur de qualité (qvalue) pondérée lorsqu’il est envoyé par le client pour hiérarchiser les schémas de compression. Pour plus d’informations, consultez [RFC 7231 : Accept-Encoding](https://tools.ietf.org/html/rfc7231#section-5.3.4).

Les algorithmes de compression sont soumis à un compromis entre la vitesse de compression et l’efficacité de la compression. L' *efficacité* dans ce contexte fait référence à la taille de la sortie après compression. La plus petite taille est obtenue par la compression la plus *optimale* .

Les en-têtes impliqués dans la demande, l’envoi, la mise en cache et la réception de contenu compressé sont décrits dans le tableau ci-dessous.

| En-tête             | Role |
| ------------------ | ---- |
| `Accept-Encoding`  | Envoyée du client au serveur pour indiquer les schémas de codage de contenu acceptables pour le client. |
| `Content-Encoding` | Envoyé du serveur au client pour indiquer l’encodage du contenu dans la charge utile. |
| `Content-Length`   | En cas de compression, `Content-Length` l’en-tête est supprimé, car le contenu du corps change lorsque la réponse est compressée. |
| `Content-MD5`      | En cas de compression, `Content-MD5` l’en-tête est supprimé, car le contenu du corps a changé et le hachage n’est plus valide. |
| `Content-Type`     | Spécifie le type MIME du contenu. Chaque réponse doit spécifier son `Content-Type`. L’intergiciel vérifie cette valeur pour déterminer si la réponse doit être compressée. L’intergiciel (middleware) spécifie un ensemble de [types MIME par défaut](#mime-types) qu’il peut encoder, mais vous pouvez remplacer ou ajouter des types MIME. |
| `Vary`             | Lorsqu’il est envoyé par le serveur avec une `Accept-Encoding` valeur à clients et proxys, `Vary` l’en-tête indique au client ou au proxy qu’il doit mettre en cache (variation) les réponses `Accept-Encoding` en fonction de la valeur de l’en-tête de la demande. Le résultat de la restitution du contenu `Vary: Accept-Encoding` avec l’en-tête est que les réponses compressées et non compressées sont mises en cache séparément. |

Explorez les fonctionnalités de l’intergiciel de compression des réponses avec l' [exemple d’application](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/performance/response-compression/samples). L’exemple illustre les éléments suivants :

* La compression des réponses de l’application à l’aide de gzip et des fournisseurs de compression personnalisés.
* Comment ajouter un type MIME à la liste par défaut des types MIME pour la compression.

## <a name="package"></a>Package

Pour inclure l’intergiciel (middleware) dans un projet, ajoutez une référence au AspNetCore [Microsoft. AspNetCore. app](xref:fundamentals/metapackage-app), qui inclut le package [Microsoft.. ResponseCompression](https://www.nuget.org/packages/Microsoft.AspNetCore.ResponseCompression/) .

## <a name="configuration"></a>Configuration

Le code suivant montre comment activer l’intergiciel (middleware) de compression des réponses pour les types MIME et les fournisseurs de compression par défaut ([Brotli](#brotli-compression-provider) et [gzip](#gzip-compression-provider)) :

```csharp
public class Startup
{
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddResponseCompression();
    }

    public void Configure(IApplicationBuilder app, IHostingEnvironment env)
    {
        app.UseResponseCompression();
    }
}
```

Remarques :

* `app.UseResponseCompression`doit être appelé avant tout middleware qui compresse les réponses. Pour plus d’informations, consultez <xref:fundamentals/middleware/index#middleware-order>.
* Utilisez un outil tel que [Fiddler](https://www.telerik.com/fiddler), [Firebug](https://getfirebug.com/)ou [postal](https://www.getpostman.com/) pour définir l’en- `Accept-Encoding` tête de la demande et étudier les en-têtes, la taille et le corps de la réponse.

Envoyez une demande à l’exemple d’application sans `Accept-Encoding` l’en-tête et observez que la réponse n’est pas compressée. Les `Content-Encoding` en `Vary` -têtes et ne sont pas présents dans la réponse.

![Fenêtre Fiddler qui indique le résultat d’une demande sans l’en-tête d’encodage Accept. La réponse n’est pas compressée.](response-compression/_static/request-uncompressed.png)

Envoyez une demande à l’exemple d’application avec `Accept-Encoding: br` l’en-tête (compression Brotli) et observez que la réponse est compressée. Les `Content-Encoding` en `Vary` -têtes et sont présents dans la réponse.

![Fenêtre Fiddler présentant le résultat d’une demande avec l’en-tête Accept-Encoding et la valeur br. Les en-têtes Vary et encodage de contenu sont ajoutés à la réponse. La réponse est compressée.](response-compression/_static/request-compressed-br.png)

## <a name="providers"></a>Fournisseurs

### <a name="brotli-compression-provider"></a>Fournisseur de compression Brotli

Utilisez le <xref:Microsoft.AspNetCore.ResponseCompression.BrotliCompressionProvider> pour compresser les réponses avec le [format de données compressées Brotli](https://tools.ietf.org/html/rfc7932).

Si aucun fournisseur de compression n’est explicitement ajouté <xref:Microsoft.AspNetCore.ResponseCompression.CompressionProviderCollection>à :

* Le fournisseur de compression Brotli est ajouté par défaut au groupe de fournisseurs de compression avec le [fournisseur de compression gzip](#gzip-compression-provider).
* La compression prend par défaut la compression Brotli lorsque le format de données compressées Brotli est pris en charge par le client. Si Brotli n’est pas pris en charge par le client, la compression est définie par défaut sur gzip lorsque le client prend en charge la compression gzip.

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddResponseCompression();
}
```

Le fournisseur de compression Brotli doit être ajouté lors de l’ajout explicite de fournisseurs de compression :

[!code-csharp[](response-compression/samples/2.x/SampleApp/Startup.cs?name=snippet1&highlight=5)]

Définissez le niveau de compression <xref:Microsoft.AspNetCore.ResponseCompression.BrotliCompressionProviderOptions>avec. Par défaut, le fournisseur de compression Brotli a le niveau de compression le plus rapide ([CompressionLevel. plus rapide](xref:System.IO.Compression.CompressionLevel)), qui peut ne pas produire la compression la plus efficace. Si vous souhaitez obtenir la compression la plus efficace possible, configurez l’intergiciel (middleware) pour une compression optimale.

| Niveau de compression | Description |
| ----------------- | ----------- |
| [CompressionLevel. plus rapide](xref:System.IO.Compression.CompressionLevel) | La compression doit être effectuée aussi rapidement que possible, même si la sortie obtenue n’est pas compressée de manière optimale. |
| [CompressionLevel. NoCompression](xref:System.IO.Compression.CompressionLevel) | Aucune compression ne doit être effectuée. |
| [CompressionLevel. optimal](xref:System.IO.Compression.CompressionLevel) | Les réponses doivent être compressées de façon optimale, même si la compression prend plus de temps. |

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddResponseCompression();

    services.Configure<BrotliCompressionProviderOptions>(options => 
    {
        options.Level = CompressionLevel.Fastest;
    });
}
```

### <a name="gzip-compression-provider"></a>Fournisseur de compression gzip

Utilisez <xref:Microsoft.AspNetCore.ResponseCompression.GzipCompressionProvider> pour compresser les réponses avec le [format de fichier gzip](https://tools.ietf.org/html/rfc1952).

Si aucun fournisseur de compression n’est explicitement ajouté <xref:Microsoft.AspNetCore.ResponseCompression.CompressionProviderCollection>à :

* Le fournisseur de compression gzip est ajouté par défaut au tableau de fournisseurs de compression avec le [fournisseur de compression Brotli](#brotli-compression-provider).
* La compression prend par défaut la compression Brotli lorsque le format de données compressées Brotli est pris en charge par le client. Si Brotli n’est pas pris en charge par le client, la compression est définie par défaut sur gzip lorsque le client prend en charge la compression gzip.

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddResponseCompression();
}
```

Le fournisseur de compression gzip doit être ajouté lors de l’ajout explicite de fournisseurs de compression :

[!code-csharp[](response-compression/samples/2.x/SampleApp/Startup.cs?name=snippet1&highlight=6)]

Définissez le niveau de compression <xref:Microsoft.AspNetCore.ResponseCompression.GzipCompressionProviderOptions>avec. Par défaut, le fournisseur de compression gzip a le niveau de compression le plus rapide ([CompressionLevel. plus rapide](xref:System.IO.Compression.CompressionLevel)), qui peut ne pas produire la compression la plus efficace. Si vous souhaitez obtenir la compression la plus efficace possible, configurez l’intergiciel (middleware) pour une compression optimale.

| Niveau de compression | Description |
| ----------------- | ----------- |
| [CompressionLevel. plus rapide](xref:System.IO.Compression.CompressionLevel) | La compression doit être effectuée aussi rapidement que possible, même si la sortie obtenue n’est pas compressée de manière optimale. |
| [CompressionLevel. NoCompression](xref:System.IO.Compression.CompressionLevel) | Aucune compression ne doit être effectuée. |
| [CompressionLevel. optimal](xref:System.IO.Compression.CompressionLevel) | Les réponses doivent être compressées de façon optimale, même si la compression prend plus de temps. |

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddResponseCompression();

    services.Configure<GzipCompressionProviderOptions>(options => 
    {
        options.Level = CompressionLevel.Fastest;
    });
}
```

### <a name="custom-providers"></a>Fournisseurs personnalisés

Créer des implémentations de compression <xref:Microsoft.AspNetCore.ResponseCompression.ICompressionProvider>personnalisées avec. <xref:Microsoft.AspNetCore.ResponseCompression.ICompressionProvider.EncodingName*> Représente l’encodage de contenu que `ICompressionProvider` ce produit. L’intergiciel utilise ces informations pour choisir le fournisseur en fonction de la liste spécifiée dans l' `Accept-Encoding` en-tête de la demande.

À l’aide de l’exemple d’application, le client envoie une `Accept-Encoding: mycustomcompression` demande avec l’en-tête. L’intergiciel utilise l’implémentation de compression personnalisée et retourne la réponse avec un `Content-Encoding: mycustomcompression` en-tête. Le client doit être en mesure de décompresser l’encodage personnalisé pour qu’une implémentation de compression personnalisée fonctionne.

[!code-csharp[](response-compression/samples/2.x/SampleApp/Startup.cs?name=snippet1&highlight=7)]

[!code-csharp[](response-compression/samples/2.x/SampleApp/CustomCompressionProvider.cs?name=snippet1)]

Envoyez une demande à l’exemple d’application avec `Accept-Encoding: mycustomcompression` l’en-tête et observez les en-têtes de réponse. Les `Vary` en `Content-Encoding` -têtes et sont présents dans la réponse. Le corps de la réponse (non affiché) n’est pas compressé par l’exemple. Il n’existe pas d’implémentation de `CustomCompressionProvider` compression dans la classe de l’exemple. Toutefois, l’exemple illustre l’emplacement où vous implémentez un tel algorithme de compression.

![Fenêtre Fiddler présentant le résultat d’une demande avec l’en-tête Accept-Encoding et une valeur de mycustomcompression. Les en-têtes Vary et encodage de contenu sont ajoutés à la réponse.](response-compression/_static/request-custom-compression.png)

## <a name="mime-types"></a>types MIME

L’intergiciel (middleware) spécifie un ensemble de types MIME par défaut pour la compression :

* `application/javascript`
* `application/json`
* `application/xml`
* `text/css`
* `text/html`
* `text/json`
* `text/plain`
* `text/xml`

Remplacez ou ajoutez des types MIME par les options de l’intergiciel (middleware) de compression des réponses. Notez que les types MIME génériques, tels `text/*` que ne sont pas pris en charge. L’exemple d’application ajoute un type MIME `image/svg+xml` pour et compresse et sert le ASP.net Core image de bannière (*Banner. svg*).

[!code-csharp[](response-compression/samples/2.x/SampleApp/Startup.cs?name=snippet1&highlight=8-10)]

## <a name="compression-with-secure-protocol"></a>Compression avec protocole sécurisé

Les réponses compressées sur des connexions sécurisées `EnableForHttps` peuvent être contrôlées avec l’option, qui est désactivée par défaut. L’utilisation de la compression avec des pages générées dynamiquement peut entraîner des problèmes de sécurité tels que les attaques de [criminalité](https://wikipedia.org/wiki/CRIME_(security_exploit)) et de [violation](https://wikipedia.org/wiki/BREACH_(security_exploit)) .

## <a name="adding-the-vary-header"></a>Ajout de l’en-tête Vary

Lors de la compression des réponses basées sur `Accept-Encoding` l’en-tête, il existe potentiellement plusieurs versions compressées de la réponse et une version non compressée. Pour indiquer au client et aux caches proxy que plusieurs versions existent et doivent être stockées, l' `Vary` en-tête est ajouté `Accept-Encoding` avec une valeur. Dans ASP.NET Core 2,0 ou version ultérieure, l’intergiciel ajoute automatiquement `Vary` l’en-tête lorsque la réponse est compressée.

## <a name="middleware-issue-when-behind-an-nginx-reverse-proxy"></a>Problème d’intergiciel lors de l’arrière-plan d’un proxy inverse Nginx

Lorsqu’une demande est traitée par un proxy par Nginx `Accept-Encoding` , l’en-tête est supprimé. La suppression de `Accept-Encoding` l’en-tête empêche l’intergiciel de compresser la réponse. Pour plus d’informations, consultez [Nginx : compression et décompression](https://www.nginx.com/resources/admin-guide/compression-and-decompression/). Ce problème est suivi par la [compression directe pour Nginx (ASPNET/BasicMiddleware #123)](https://github.com/aspnet/BasicMiddleware/issues/123).

## <a name="working-with-iis-dynamic-compression"></a>Utilisation de la compression dynamique IIS

Si vous disposez d’un module de compression dynamique IIS configuré au niveau du serveur que vous souhaitez désactiver pour une application, désactivez le module avec un ajout au fichier *Web. config* . Pour plus d’informations, consultez [Désactivation de modules IIS](xref:host-and-deploy/iis/modules#disabling-iis-modules).

## <a name="troubleshooting"></a>Dépannage

Utilisez un outil tel que [Fiddler](https://www.telerik.com/fiddler), [Firebug](https://getfirebug.com/)ou [postal](https://www.getpostman.com/), qui vous permet de définir l’en `Accept-Encoding` -tête de la demande et d’étudier les en-têtes, la taille et le corps de la réponse. Par défaut, l’intergiciel (middleware) de compression des réponses compresse les réponses qui remplissent les conditions suivantes :

* L' `Accept-Encoding` en-tête est présent avec une `br`valeur `gzip`, `*`, ou un encodage personnalisé qui correspond à un fournisseur de compression personnalisé que vous avez établi. La valeur ne doit pas `identity` être ou avoir une valeur de qualité ( `q`qvalue,) égale à 0 (zéro).
* Le type MIME (`Content-Type`) doit être défini et doit correspondre à un type MIME configuré sur <xref:Microsoft.AspNetCore.ResponseCompression.ResponseCompressionOptions>le.
* La requête ne doit pas inclure `Content-Range` l’en-tête.
* La demande doit utiliser le protocole http, sauf si le protocole sécurisé (https) est configuré dans les options de l’intergiciel (middleware) de compression des réponses. *Notez le danger [décrit ci-dessus](#compression-with-secure-protocol) lors de l’activation de la compression de contenu sécurisé.*

## <a name="additional-resources"></a>Ressources supplémentaires

* <xref:fundamentals/startup>
* <xref:fundamentals/middleware/index>
* [Réseau des développeurs Mozilla : accepter-encodage](https://developer.mozilla.org/docs/Web/HTTP/Headers/Accept-Encoding)
* [RFC 7231 section 3.1.2.1 : codages de contenu](https://tools.ietf.org/html/rfc7231#section-3.1.2.1)
* [RFC 7230, section 4.2.3 : codage gzip](https://tools.ietf.org/html/rfc7230#section-4.2.3)
* [Spécification de format de fichier GZIP version 4,3](https://www.ietf.org/rfc/rfc1952.txt)

::: moniker-end

::: moniker range="< aspnetcore-2.2"

La bande passante réseau est une ressource limitée. La réduction de la taille de la réponse augmente généralement la réactivité d’une application, souvent considérablement. Une façon de réduire les tailles de charge utile consiste à compresser les réponses d’une application.

[Afficher ou télécharger l’exemple de code](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/performance/response-compression/samples) ([procédure de téléchargement](xref:index#how-to-download-a-sample))

## <a name="when-to-use-response-compression-middleware"></a>Quand utiliser l’intergiciel (middleware) de compression des réponses

Utilisez les technologies de compression des réponses basées sur le serveur dans IIS, Apache ou nginx. Les performances de l’intergiciel (middleware) ne correspondront probablement pas à celles des modules de serveur. Le serveur de serveur [http. sys](xref:fundamentals/servers/httpsys) et le serveur [Kestrel](xref:fundamentals/servers/kestrel) n’offrent pas de prise en charge intégrée de la compression.

Utilisez l’intergiciel (middleware) de compression des réponses lorsque vous êtes :

* Impossible d’utiliser les technologies de compression basées sur le serveur suivantes :
  * [Module de compression dynamique IIS](https://www.iis.net/overview/reliability/dynamiccachingandcompression)
  * [Module Apache mod_deflate](https://httpd.apache.org/docs/current/mod/mod_deflate.html)
  * [Compression et décompression Nginx](https://www.nginx.com/resources/admin-guide/compression-and-decompression/)
* Hébergement direct :
  * [Serveur http. sys](xref:fundamentals/servers/httpsys) (anciennement appelé webListener)
  * [Serveur Kestrel](xref:fundamentals/servers/kestrel)

## <a name="response-compression"></a>Compression des réponses

En règle générale, toute réponse qui n’est pas compressée en mode natif peut tirer parti de la compression des réponses. Les réponses qui ne sont pas compressées en mode natif incluent généralement : CSS, JavaScript, HTML, XML et JSON. Vous ne devez pas compresser les ressources natives compressées, telles que les fichiers PNG. Si vous tentez de compresser davantage une réponse compressée en mode natif, toute petite réduction supplémentaire de taille et de temps de transmission sera probablement survenue du temps nécessaire au traitement de la compression. Ne compressez pas les fichiers d’une taille inférieure à environ 150-1000 octets (en fonction du contenu du fichier et de l’efficacité de la compression). La surcharge liée à la compression de petits fichiers peut entraîner la création d’un fichier compressé plus volumineux que le fichier non compressé.

Lorsqu’un client peut traiter du contenu compressé, le client doit informer le serveur de ses fonctionnalités en envoyant `Accept-Encoding` l’en-tête avec la demande. Lorsqu’un serveur envoie du contenu compressé, il doit inclure des informations `Content-Encoding` dans l’en-tête sur la manière dont la réponse compressée est encodée. Les désignations de codage de contenu prises en charge par l’intergiciel (middleware) sont indiquées dans le tableau suivant.

| `Accept-Encoding`valeurs d’en-tête | Intergiciel pris en charge | Description |
| ------------------------------- | :------------------: | ----------- |
| `br`                            | Non                   | [Format de données compressées Brotli](https://tools.ietf.org/html/rfc7932) |
| `deflate`                       | Non                   | [Format de données compressées compressé](https://tools.ietf.org/html/rfc1951) |
| `exi`                           | Non                   | [Échange XML efficace W3C](https://tools.ietf.org/id/draft-varga-netconf-exi-capability-00.html) |
| `gzip`                          | Oui (valeur par défaut)        | [Format de fichier gzip](https://tools.ietf.org/html/rfc1952) |
| `identity`                      | Oui                  | Identificateur « no Encoding » : la réponse ne doit pas être encodée. |
| `pack200-gzip`                  | Non                   | [Format de transfert réseau pour les archives Java](https://jcp.org/aboutJava/communityprocess/review/jsr200/index.html) |
| `*`                             | Oui                  | Tout encodage de contenu disponible qui n’est pas explicitement demandé |

Pour plus d’informations, consultez la [liste de codage de contenu officielle IANA](https://www.iana.org/assignments/http-parameters/http-parameters.xml#http-content-coding-registry).

L’intergiciel (middleware) vous permet d’ajouter des fournisseurs de `Accept-Encoding` compression supplémentaires pour les valeurs d’en-tête personnalisées. Pour plus d’informations, consultez [fournisseurs personnalisés](#custom-providers) ci-dessous.

L’intergiciel (middleware `q`) peut réagir à la valeur de qualité (qvalue) pondérée lorsqu’il est envoyé par le client pour hiérarchiser les schémas de compression. Pour plus d’informations, consultez [RFC 7231 : Accept-Encoding](https://tools.ietf.org/html/rfc7231#section-5.3.4).

Les algorithmes de compression sont soumis à un compromis entre la vitesse de compression et l’efficacité de la compression. L' *efficacité* dans ce contexte fait référence à la taille de la sortie après compression. La plus petite taille est obtenue par la compression la plus *optimale* .

Les en-têtes impliqués dans la demande, l’envoi, la mise en cache et la réception de contenu compressé sont décrits dans le tableau ci-dessous.

| En-tête             | Role |
| ------------------ | ---- |
| `Accept-Encoding`  | Envoyée du client au serveur pour indiquer les schémas de codage de contenu acceptables pour le client. |
| `Content-Encoding` | Envoyé du serveur au client pour indiquer l’encodage du contenu dans la charge utile. |
| `Content-Length`   | En cas de compression, `Content-Length` l’en-tête est supprimé, car le contenu du corps change lorsque la réponse est compressée. |
| `Content-MD5`      | En cas de compression, `Content-MD5` l’en-tête est supprimé, car le contenu du corps a changé et le hachage n’est plus valide. |
| `Content-Type`     | Spécifie le type MIME du contenu. Chaque réponse doit spécifier son `Content-Type`. L’intergiciel vérifie cette valeur pour déterminer si la réponse doit être compressée. L’intergiciel (middleware) spécifie un ensemble de [types MIME par défaut](#mime-types) qu’il peut encoder, mais vous pouvez remplacer ou ajouter des types MIME. |
| `Vary`             | Lorsqu’il est envoyé par le serveur avec une `Accept-Encoding` valeur à clients et proxys, `Vary` l’en-tête indique au client ou au proxy qu’il doit mettre en cache (variation) les réponses `Accept-Encoding` en fonction de la valeur de l’en-tête de la demande. Le résultat de la restitution du contenu `Vary: Accept-Encoding` avec l’en-tête est que les réponses compressées et non compressées sont mises en cache séparément. |

Explorez les fonctionnalités de l’intergiciel de compression des réponses avec l' [exemple d’application](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/performance/response-compression/samples). L’exemple illustre les éléments suivants :

* La compression des réponses de l’application à l’aide de gzip et des fournisseurs de compression personnalisés.
* Comment ajouter un type MIME à la liste par défaut des types MIME pour la compression.

## <a name="package"></a>Package

Pour inclure l’intergiciel (middleware) dans un projet, ajoutez une référence au AspNetCore [Microsoft. AspNetCore. app](xref:fundamentals/metapackage-app), qui inclut le package [Microsoft.. ResponseCompression](https://www.nuget.org/packages/Microsoft.AspNetCore.ResponseCompression/) .

## <a name="configuration"></a>Configuration

Le code suivant montre comment activer l’intergiciel (middleware) de compression des réponses pour les types MIME par défaut et le [fournisseur de compression gzip](#gzip-compression-provider):

```csharp
public class Startup
{
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddResponseCompression();
    }

    public void Configure(IApplicationBuilder app, IHostingEnvironment env)
    {
        app.UseResponseCompression();
    }
}
```

Remarques :

* `app.UseResponseCompression`doit être appelé avant tout middleware qui compresse les réponses. Pour plus d’informations, consultez <xref:fundamentals/middleware/index#middleware-order>.
* Utilisez un outil tel que [Fiddler](https://www.telerik.com/fiddler), [Firebug](https://getfirebug.com/)ou [postal](https://www.getpostman.com/) pour définir l’en- `Accept-Encoding` tête de la demande et étudier les en-têtes, la taille et le corps de la réponse.

Envoyez une demande à l’exemple d’application sans `Accept-Encoding` l’en-tête et observez que la réponse n’est pas compressée. Les `Content-Encoding` en `Vary` -têtes et ne sont pas présents dans la réponse.

![Fenêtre Fiddler qui indique le résultat d’une demande sans l’en-tête d’encodage Accept. La réponse n’est pas compressée.](response-compression/_static/request-uncompressed.png)

Envoyez une demande à l’exemple d’application avec `Accept-Encoding: gzip` l’en-tête et observez que la réponse est compressée. Les `Content-Encoding` en `Vary` -têtes et sont présents dans la réponse.

![Fenêtre Fiddler présentant le résultat d’une demande avec l’en-tête Accept-Encoding et une valeur de gzip. Les en-têtes Vary et encodage de contenu sont ajoutés à la réponse. La réponse est compressée.](response-compression/_static/request-compressed.png)

## <a name="providers"></a>Fournisseurs

### <a name="gzip-compression-provider"></a>Fournisseur de compression gzip

Utilisez <xref:Microsoft.AspNetCore.ResponseCompression.GzipCompressionProvider> pour compresser les réponses avec le [format de fichier gzip](https://tools.ietf.org/html/rfc1952).

Si aucun fournisseur de compression n’est explicitement ajouté <xref:Microsoft.AspNetCore.ResponseCompression.CompressionProviderCollection>à :

* Le fournisseur de compression gzip est ajouté par défaut au tableau de fournisseurs de compression.
* La compression est définie par défaut sur gzip lorsque le client prend en charge la compression gzip.

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddResponseCompression();
}
```

Le fournisseur de compression gzip doit être ajouté lors de l’ajout explicite de fournisseurs de compression :

[!code-csharp[](response-compression/samples/2.x/SampleApp/Startup.cs?name=snippet1&highlight=6)]

Définissez le niveau de compression <xref:Microsoft.AspNetCore.ResponseCompression.GzipCompressionProviderOptions>avec. Par défaut, le fournisseur de compression gzip a le niveau de compression le plus rapide ([CompressionLevel. plus rapide](xref:System.IO.Compression.CompressionLevel)), qui peut ne pas produire la compression la plus efficace. Si vous souhaitez obtenir la compression la plus efficace possible, configurez l’intergiciel (middleware) pour une compression optimale.

| Niveau de compression | Description |
| ----------------- | ----------- |
| [CompressionLevel. plus rapide](xref:System.IO.Compression.CompressionLevel) | La compression doit être effectuée aussi rapidement que possible, même si la sortie obtenue n’est pas compressée de manière optimale. |
| [CompressionLevel. NoCompression](xref:System.IO.Compression.CompressionLevel) | Aucune compression ne doit être effectuée. |
| [CompressionLevel. optimal](xref:System.IO.Compression.CompressionLevel) | Les réponses doivent être compressées de façon optimale, même si la compression prend plus de temps. |

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddResponseCompression();

    services.Configure<GzipCompressionProviderOptions>(options => 
    {
        options.Level = CompressionLevel.Fastest;
    });
}
```

### <a name="custom-providers"></a>Fournisseurs personnalisés

Créer des implémentations de compression <xref:Microsoft.AspNetCore.ResponseCompression.ICompressionProvider>personnalisées avec. <xref:Microsoft.AspNetCore.ResponseCompression.ICompressionProvider.EncodingName*> Représente l’encodage de contenu que `ICompressionProvider` ce produit. L’intergiciel utilise ces informations pour choisir le fournisseur en fonction de la liste spécifiée dans l' `Accept-Encoding` en-tête de la demande.

À l’aide de l’exemple d’application, le client envoie une `Accept-Encoding: mycustomcompression` demande avec l’en-tête. L’intergiciel utilise l’implémentation de compression personnalisée et retourne la réponse avec un `Content-Encoding: mycustomcompression` en-tête. Le client doit être en mesure de décompresser l’encodage personnalisé pour qu’une implémentation de compression personnalisée fonctionne.

[!code-csharp[](response-compression/samples/2.x/SampleApp/Startup.cs?name=snippet1&highlight=7)]

[!code-csharp[](response-compression/samples/2.x/SampleApp/CustomCompressionProvider.cs?name=snippet1)]

Envoyez une demande à l’exemple d’application avec `Accept-Encoding: mycustomcompression` l’en-tête et observez les en-têtes de réponse. Les `Vary` en `Content-Encoding` -têtes et sont présents dans la réponse. Le corps de la réponse (non affiché) n’est pas compressé par l’exemple. Il n’existe pas d’implémentation de `CustomCompressionProvider` compression dans la classe de l’exemple. Toutefois, l’exemple illustre l’emplacement où vous implémentez un tel algorithme de compression.

![Fenêtre Fiddler présentant le résultat d’une demande avec l’en-tête Accept-Encoding et une valeur de mycustomcompression. Les en-têtes Vary et encodage de contenu sont ajoutés à la réponse.](response-compression/_static/request-custom-compression.png)

## <a name="mime-types"></a>types MIME

L’intergiciel (middleware) spécifie un ensemble de types MIME par défaut pour la compression :

* `application/javascript`
* `application/json`
* `application/xml`
* `text/css`
* `text/html`
* `text/json`
* `text/plain`
* `text/xml`

Remplacez ou ajoutez des types MIME par les options de l’intergiciel (middleware) de compression des réponses. Notez que les types MIME génériques, tels `text/*` que ne sont pas pris en charge. L’exemple d’application ajoute un type MIME `image/svg+xml` pour et compresse et sert le ASP.net Core image de bannière (*Banner. svg*).

[!code-csharp[](response-compression/samples/2.x/SampleApp/Startup.cs?name=snippet1&highlight=8-10)]

## <a name="compression-with-secure-protocol"></a>Compression avec protocole sécurisé

Les réponses compressées sur des connexions sécurisées `EnableForHttps` peuvent être contrôlées avec l’option, qui est désactivée par défaut. L’utilisation de la compression avec des pages générées dynamiquement peut entraîner des problèmes de sécurité tels que les attaques de [criminalité](https://wikipedia.org/wiki/CRIME_(security_exploit)) et de [violation](https://wikipedia.org/wiki/BREACH_(security_exploit)) .

## <a name="adding-the-vary-header"></a>Ajout de l’en-tête Vary

Lors de la compression des réponses basées sur `Accept-Encoding` l’en-tête, il existe potentiellement plusieurs versions compressées de la réponse et une version non compressée. Pour indiquer au client et aux caches proxy que plusieurs versions existent et doivent être stockées, l' `Vary` en-tête est ajouté `Accept-Encoding` avec une valeur. Dans ASP.NET Core 2,0 ou version ultérieure, l’intergiciel ajoute automatiquement `Vary` l’en-tête lorsque la réponse est compressée.

## <a name="middleware-issue-when-behind-an-nginx-reverse-proxy"></a>Problème d’intergiciel lors de l’arrière-plan d’un proxy inverse Nginx

Lorsqu’une demande est traitée par un proxy par Nginx `Accept-Encoding` , l’en-tête est supprimé. La suppression de `Accept-Encoding` l’en-tête empêche l’intergiciel de compresser la réponse. Pour plus d’informations, consultez [Nginx : compression et décompression](https://www.nginx.com/resources/admin-guide/compression-and-decompression/). Ce problème est suivi par la [compression directe pour Nginx (ASPNET/BasicMiddleware #123)](https://github.com/aspnet/BasicMiddleware/issues/123).

## <a name="working-with-iis-dynamic-compression"></a>Utilisation de la compression dynamique IIS

Si vous disposez d’un module de compression dynamique IIS configuré au niveau du serveur que vous souhaitez désactiver pour une application, désactivez le module avec un ajout au fichier *Web. config* . Pour plus d’informations, consultez [Désactivation de modules IIS](xref:host-and-deploy/iis/modules#disabling-iis-modules).

## <a name="troubleshooting"></a>Dépannage

Utilisez un outil tel que [Fiddler](https://www.telerik.com/fiddler), [Firebug](https://getfirebug.com/)ou [postal](https://www.getpostman.com/), qui vous permet de définir l’en `Accept-Encoding` -tête de la demande et d’étudier les en-têtes, la taille et le corps de la réponse. Par défaut, l’intergiciel (middleware) de compression des réponses compresse les réponses qui remplissent les conditions suivantes :

* L' `Accept-Encoding` en-tête est présent avec une `gzip`valeur `*`, ou un encodage personnalisé qui correspond à un fournisseur de compression personnalisé que vous avez établi. La valeur ne doit pas `identity` être ou avoir une valeur de qualité ( `q`qvalue,) égale à 0 (zéro).
* Le type MIME (`Content-Type`) doit être défini et doit correspondre à un type MIME configuré sur <xref:Microsoft.AspNetCore.ResponseCompression.ResponseCompressionOptions>le.
* La requête ne doit pas inclure `Content-Range` l’en-tête.
* La demande doit utiliser le protocole http, sauf si le protocole sécurisé (https) est configuré dans les options de l’intergiciel (middleware) de compression des réponses. *Notez le danger [décrit ci-dessus](#compression-with-secure-protocol) lors de l’activation de la compression de contenu sécurisé.*

## <a name="additional-resources"></a>Ressources supplémentaires

* <xref:fundamentals/startup>
* <xref:fundamentals/middleware/index>
* [Réseau des développeurs Mozilla : accepter-encodage](https://developer.mozilla.org/docs/Web/HTTP/Headers/Accept-Encoding)
* [RFC 7231 section 3.1.2.1 : codages de contenu](https://tools.ietf.org/html/rfc7231#section-3.1.2.1)
* [RFC 7230, section 4.2.3 : codage gzip](https://tools.ietf.org/html/rfc7230#section-4.2.3)
* [Spécification de format de fichier GZIP version 4,3](https://www.ietf.org/rfc/rfc1952.txt)

::: moniker-end
