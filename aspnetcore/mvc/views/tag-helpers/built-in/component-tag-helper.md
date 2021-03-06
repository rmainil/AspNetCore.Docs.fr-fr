---
title: Tag Helper Component dans ASP.NET Core
author: guardrex
ms.author: riande
description: Découvrez comment utiliser le tag Helper du composant ASP.NET Core pour afficher Razor les composants dans les pages et les vues.
ms.custom: mvc
ms.date: 04/15/2020
no-loc:
- Blazor
- Identity
- Let's Encrypt
- Razor
- SignalR
uid: mvc/views/tag-helpers/builtin-th/component-tag-helper
ms.openlocfilehash: 4e003e5ed5e7863d8a218c0f02bb37e214e31910
ms.sourcegitcommit: 70e5f982c218db82aa54aa8b8d96b377cfc7283f
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 05/04/2020
ms.locfileid: "82773927"
---
# <a name="component-tag-helper-in-aspnet-core"></a>Tag Helper Component dans ASP.NET Core

Par [Daniel Roth](https://github.com/danroth27) et [Luke Latham](https://github.com/guardrex)

Pour afficher un composant à partir d’une page ou d’une vue, utilisez le [tag Helper Component](xref:Microsoft.AspNetCore.Mvc.TagHelpers.ComponentTagHelper).

## <a name="prerequisites"></a>Prérequis

Suivez les instructions de la section *préparer l’application à utiliser les composants des pages et* des vues <xref:blazor/integrate-components#prepare-the-app> de l’article.

## <a name="component-tag-helper"></a>Tag Helper Component

Le tag Helper Component suivant restitue le `Counter` composant dans une page ou une vue :

```cshtml
@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers
@using {APP ASSEMBLY}.Pages

...

<component type="typeof(Counter)" render-mode="ServerPrerendered" />
```

L’exemple précédent suppose que le `Counter` composant se trouve dans le dossier *pages* de l’application.

Le tag Helper Component peut également transmettre des paramètres à des composants. Prenons le composant `ColorfulCheckbox` suivant qui définit la couleur et la taille de l’étiquette de case à cocher :

```razor
<label style="font-size:@(Size)px;color:@Color">
    <input @bind="Value"
           id="survey" 
           name="blazor" 
           type="checkbox" />
    Enjoying Blazor?
</label>

@code {
    [Parameter]
    public bool Value { get; set; }

    [Parameter]
    public int Size { get; set; } = 8;

    [Parameter]
    public string Color { get; set; }

    protected override void OnInitialized()
    {
        Size += 10;
    }
}
```

Les `Size` paramètres`int`du `Color` [composant](xref:blazor/components#component-parameters) (`string`) et () peuvent être définis par le tag Helper du composant :

```cshtml
@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers
@using {APP ASSEMBLY}.Shared

...

<component type="typeof(ColorfulCheckbox)" render-mode="ServerPrerendered" 
    param-Size="14" param-Color="@("blue")" />
```

L’exemple précédent suppose que le `ColorfulCheckbox` composant se trouve dans le dossier *partagé* de l’application.

Le code HTML suivant est affiché dans la page ou la vue :

```html
<label style="font-size:24px;color:blue">
    <input id="survey" name="blazor" type="checkbox">
    Enjoying Blazor?
</label>
```

Le passage d’une chaîne entre guillemets requiert une [expression Razor explicite](xref:mvc/views/razor#explicit-razor-expressions), comme illustré `param-Color` dans l’exemple précédent. Le comportement d’analyse Razor pour une `string` valeur de type ne s’applique `param-*` pas à un attribut, car `object` l’attribut est un type.

Le type de paramètre doit être sérialisable JSON, ce qui signifie généralement que le type doit avoir un constructeur par défaut et des propriétés définissables. Par exemple, vous pouvez spécifier une valeur pour `Size` et `Color` dans l’exemple précédent, car les types `Size` de `Color` et sont des types`int` primitifs (et `string`), qui sont pris en charge par le sérialiseur JSON.

Dans l’exemple suivant, un objet de classe est passé au composant :

*MyClass.cs*:

```csharp
public class MyClass
{
    public MyClass()
    {
    }

    public int MyInt { get; set; } = 999;
    public string MyString { get; set; } = "Initial value";
}
```

**La classe doit avoir un constructeur sans paramètre public.**

*Shared/MyComponent. Razor*:

```razor
<h2>MyComponent</h2>

<p>Int: @MyObject.MyInt</p>
<p>String: @MyObject.MyString</p>

@code
{
    [Parameter]
    public MyClass MyObject { get; set; }
}
```

*Pages/MyPage. cshtml*:

```cshtml
@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers
@using {APP ASSEMBLY}
@using {APP ASSEMBLY}.Shared

...

@{
    var myObject = new MyClass();
    myObject.MyInt = 7;
    myObject.MyString = "Set by MyPage";
}

<component type="typeof(MyComponent)" render-mode="ServerPrerendered" 
    param-MyObject="@myObject" />
```

L’exemple précédent suppose que le `MyComponent` composant se trouve dans le dossier *partagé* de l’application. `MyClass`se trouve dans l’espace de noms`{APP ASSEMBLY}`de l’application ().

<xref:Microsoft.AspNetCore.Mvc.Rendering.RenderMode>Configure si le composant :

* Est prérendu dans la page.
* Est rendu en HTML statique sur la page ou s’il contient les informations nécessaires pour démarrer une application éblouissant à partir de l’agent utilisateur.

| Mode de rendu | Description |
| ----------- | ----------- |
| <xref:Microsoft.AspNetCore.Mvc.Rendering.RenderMode.ServerPrerendered> | Génère le rendu du composant en HTML statique et comprend un marqueur Blazor pour une application serveur. Au démarrage de l’agent utilisateur, ce marqueur est utilisé pour démarrer Blazor une application. |
| <xref:Microsoft.AspNetCore.Mvc.Rendering.RenderMode.Server> | Restitue un marqueur pour Blazor une application serveur. La sortie du composant n’est pas incluse. Au démarrage de l’agent utilisateur, ce marqueur est utilisé pour démarrer Blazor une application. |
| <xref:Microsoft.AspNetCore.Mvc.Rendering.RenderMode.Static> | Génère le rendu du composant en HTML statique. |

Alors que les pages et les vues peuvent utiliser des composants, la réciproque n’est pas vraie. Les composants ne peuvent pas utiliser les fonctionnalités spécifiques aux vues et aux pages, telles que les vues partielles et les sections. Pour utiliser la logique d’une vue partielle dans un composant, factorisez la logique de la vue partielle dans un composant.

Le rendu des composants serveur à partir d’une page HTML statique n’est pas pris en charge.

## <a name="additional-resources"></a>Ressources supplémentaires

* <xref:Microsoft.AspNetCore.Mvc.TagHelpers.ComponentTagHelper>
* <xref:mvc/views/tag-helpers/intro>
* <xref:blazor/components>
