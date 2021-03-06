---
title: Kit de développement logiciel (SDK) Web ASP.NET Core
author: Rick-Anderson
description: Vue d’ensemble de Microsoft. NET. Sdk. Web.
ms.author: riande
ms.date: 01/25/2020
no-loc:
- Blazor
- Identity
- Let's Encrypt
- Razor
- SignalR
uid: razor-pages/web-sdk
ms.openlocfilehash: 2797f0b3003b8ad89093fe1115dee2acc8650c73
ms.sourcegitcommit: 70e5f982c218db82aa54aa8b8d96b377cfc7283f
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 05/04/2020
ms.locfileid: "82777161"
---
# <a name="aspnet-core-web-sdk"></a>Kit de développement logiciel (SDK) Web ASP.NET Core

### <a name="overview"></a>Vue d’ensemble

`Microsoft.NET.Sdk.Web`est un [Kit de développement logiciel (SDK) de projet MSBuild](https://docs.microsoft.com/visualstudio/msbuild/how-to-use-project-sdk) pour la génération d’applications ASP.net core. Il est possible de créer une application ASP.NET Core sans ce kit de développement logiciel (SDK), mais le kit de développement logiciel (SDK) Web est le suivant :

* Adapté à la fourniture d’une expérience de première classe.
* Cible recommandée pour la plupart des utilisateurs.

Utilisez le kit de développement logiciel (SDK) Web dans un projet :

  ```xml
  <Project Sdk="Microsoft.NET.Sdk.Web">
    <!-- omitted for brevity -->
  </Project>
  ```

Fonctionnalités activées à l’aide du kit de développement logiciel (SDK) Web :

* Les projets ciblant .NET Core 3,0 ou une version ultérieure référencent implicitement :

  * [Framework partagé ASP.net Core](xref:fundamentals/metapackage-app).
  * [Analyseurs](/visualstudio/extensibility/getting-started-with-roslyn-analyzers) conçus pour générer des applications ASP.net core.
* Le kit de développement logiciel (SDK) Web importe les cibles MSBuild qui permettent l’utilisation de profils de publication et la publication à l’aide de WebDeploy.

### <a name="properties"></a>Propriétés

| Propriété | Description |
| -------- | ----------- |
| `DisableImplicitFrameworkReferences` | Désactive la référence implicite à l' `Microsoft.AspNetCore.App` infrastructure partagée. |
| `DisableImplicitAspNetCoreAnalyzers` | Désactive la référence implicite aux analyseurs de ASP.NET Core. |
| `DisableImplicitComponentsAnalyzers` | Désactive la référence implicite aux Razor analyseurs de composants lors Blazor de la génération d’applications (serveur). |
