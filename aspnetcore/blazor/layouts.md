---
title: Dispositions Blazor de ASP.net Core
author: guardrex
description: Découvrez comment créer des composants de disposition réutilisables pour les Blazor applications.
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 02/12/2020
no-loc:
- Blazor
- Identity
- Let's Encrypt
- Razor
- SignalR
uid: blazor/layouts
ms.openlocfilehash: 5c6771dd7249bfb8280ba20e1ce75967f279971c
ms.sourcegitcommit: 70e5f982c218db82aa54aa8b8d96b377cfc7283f
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 05/04/2020
ms.locfileid: "82771583"
---
# <a name="aspnet-core-blazor-layouts"></a>Dispositions Blazor de ASP.net Core

Par [Rainer Stropek](https://www.timecockpit.com) et [Luke Latham](https://github.com/guardrex)

Certains éléments de l’application, tels que les menus, les messages de copyright et les logos de l’entreprise, font généralement partie de la mise en page globale de l’application et sont utilisés par chaque composant de l’application. La copie du code de ces éléments dans tous les composants d’une application n’est pas une&mdash;approche efficace chaque fois que l’un des éléments requiert une mise à jour, chaque composant doit être mis à jour. Une telle duplication est difficile à gérer et peut entraîner une incohérence du contenu au fil du temps. Les *dispositions* résolvent ce problème.

Techniquement, une disposition est simplement un autre composant. Une disposition est définie dans un Razor modèle ou dans du code C# et peut utiliser la [liaison de données](xref:blazor/data-binding), l' [injection de dépendances](xref:blazor/dependency-injection)et d’autres scénarios de composants.

Pour transformer un *composant* en une *disposition*, le composant :

* Hérite de `LayoutComponentBase`, qui définit une `Body` propriété pour le contenu rendu à l’intérieur de la disposition.
* Utilise la Razor syntaxe `@Body` pour spécifier l’emplacement dans la balise de mise en page où le contenu est restitué.

L’exemple de code suivant montre Razor le modèle d’un composant de disposition, *MainLayout. Razor*. La disposition hérite `LayoutComponentBase` de et définit `@Body` la entre la barre de navigation et le pied de page :

[!code-razor[](layouts/sample_snapshot/3.x/MainLayout.razor?highlight=1,13)]

Dans une application basée sur l’un des Blazor modèles d’application, `MainLayout` le composant (*MainLayout. Razor*) se trouve dans le dossier *partagé* de l’application.

## <a name="default-layout"></a>Disposition par défaut

Spécifiez la disposition de l’application `Router` par défaut dans le composant dans le fichier *app. Razor* de l’application. Le composant `Router` suivant, qui est fourni par les modèles Blazor par défaut, définit la disposition par défaut `MainLayout` sur le composant :

[!code-razor[](layouts/sample_snapshot/3.x/App1.razor?highlight=3)]

Pour fournir une disposition par défaut `NotFound` pour le contenu, `LayoutView` spécifiez un pour `NotFound` le contenu :

[!code-razor[](layouts/sample_snapshot/3.x/App2.razor?highlight=6-9)]

Pour plus d’informations sur `Router` le composant, <xref:blazor/routing>consultez.

La spécification de la disposition comme disposition par défaut dans le routeur est une pratique utile, car elle peut être remplacée par composant ou par dossier. Préférez utiliser le routeur pour définir la disposition par défaut de l’application, car il s’agit de la technique la plus générale.

## <a name="specify-a-layout-in-a-component"></a>Spécifier une disposition dans un composant

Utilisez la Razor directive `@layout` pour appliquer une disposition à un composant. Le compilateur convertit `@layout` en `LayoutAttribute`, qui est appliqué à la classe de composant.

Le contenu du composant suivant `MasterList` est inséré dans le `MasterLayout` à la position de : `@Body`

[!code-razor[](layouts/sample_snapshot/3.x/MasterList.razor?highlight=1)]

La spécification de la disposition directement dans un composant remplace un ensemble de *dispositions par défaut* dans le routeur `@layout` ou une directive importée à partir de *_Imports. Razor*.

## <a name="centralized-layout-selection"></a>Sélection de la disposition centralisée

Chaque dossier d’une application peut éventuellement contenir un fichier modèle nommé *_Imports. Razor*. Le compilateur comprend les directives spécifiées dans le fichier Imports dans Razor tous les modèles du même dossier et de manière récursive dans tous ses sous-dossiers. Par conséquent, un fichier *_Imports. Razor* contenant `@layout MyCoolLayout` s’assure que tous les composants d’un dossier `MyCoolLayout`utilisent. Il n’est pas nécessaire d’ajouter `@layout MyCoolLayout` à plusieurs reprises à tous les fichiers *. Razor* dans le dossier et les sous-dossiers. `@using`les directives sont également appliquées aux composants de la même façon.

Le fichier *_Imports. Razor* suivant importe les éléments suivants :

* `MyCoolLayout`.
* Tous Razor les composants du même dossier et de tous les sous-dossiers.
* Espace de noms `BlazorApp1.Data`.
 
[!code-razor[](layouts/sample_snapshot/3.x/_Imports.razor)]

Le fichier *_Imports. Razor* est semblable au [fichier _ViewImports. cshtml pour Razor les affichages et les pages,](xref:mvc/views/layout#importing-shared-directives) mais appliqué spécifiquement aux fichiers de Razor composants.

La spécification d’une disposition dans *_Imports. Razor* remplace une disposition spécifiée comme *disposition par défaut*du routeur.

## <a name="nested-layouts"></a>Dispositions imbriquées

Les applications peuvent se composer de dispositions imbriquées. Un composant peut faire référence à une disposition qui, à son tour, fait référence à une autre disposition. Par exemple, les dispositions d’imbrication sont utilisées pour créer une structure de menus à plusieurs niveaux.

L’exemple suivant montre comment utiliser des dispositions imbriquées. Le fichier *EpisodesComponent. Razor* est le composant à afficher. Le composant fait référence `MasterListLayout`à :

[!code-razor[](layouts/sample_snapshot/3.x/EpisodesComponent.razor?highlight=1)]

Le fichier *MasterListLayout. Razor* fournit le `MasterListLayout`. La disposition fait référence à une `MasterLayout`autre disposition,, où elle est rendue. `EpisodesComponent`l’emplacement `@Body` est affiché :

[!code-razor[](layouts/sample_snapshot/3.x/MasterListLayout.razor?highlight=1,9)]

Enfin, `MasterLayout` dans *MasterLayout. Razor* contient les éléments de disposition de niveau supérieur, tels que l’en-tête, le menu principal et le pied de page. `MasterListLayout`avec `EpisodesComponent` est rendu où `@Body` s’affiche :

[!code-razor[](layouts/sample_snapshot/3.x/MasterLayout.razor?highlight=6)]

## <a name="share-a-razor-pages-layout-with-integrated-components"></a>Partager une Razor disposition de pages avec des composants intégrés

Lorsque des composants routables sont intégrés à Razor une application pages, la disposition partagée de l’application peut être utilisée avec les composants. Pour plus d’informations, consultez <xref:blazor/integrate-components>.

## <a name="additional-resources"></a>Ressources supplémentaires

* <xref:mvc/views/layout>
