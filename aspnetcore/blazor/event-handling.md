---
title: Gestion Blazor des événements ASP.net Core
author: guardrex
description: Découvrez les Blazorfonctionnalités de gestion des événements de, notamment les types d’arguments d’événement, les rappels d’événements et la gestion des événements de navigateur par défaut.
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 03/16/2020
no-loc:
- Blazor
- Identity
- Let's Encrypt
- Razor
- SignalR
uid: blazor/event-handling
ms.openlocfilehash: aa338bbe61eec14bc1e1b3606e11e26bfb0e6a09
ms.sourcegitcommit: 84b46594f57608f6ac4f0570172c7051df507520
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 05/08/2020
ms.locfileid: "82967465"
---
# <a name="aspnet-core-blazor-event-handling"></a>ASP.NET Core la gestion des événements éblouissants

Par [Luke Latham](https://github.com/guardrex) et [Daniel Roth](https://github.com/danroth27)

Les composants Razor fournissent des fonctionnalités de gestion des événements. Pour un attribut d’élément HTML [`@on{EVENT}`](xref:mvc/views/razor#onevent) nommé (par exemple `@onclick`,) avec une valeur de type délégué, un composant Razor traite la valeur de l’attribut en tant que gestionnaire d’événements.

Le code suivant appelle la `UpdateHeading` méthode lorsque le bouton est sélectionné dans l’interface utilisateur :

```razor
<button class="btn btn-primary" @onclick="UpdateHeading">
    Update heading
</button>

@code {
    private void UpdateHeading(MouseEventArgs e)
    {
        ...
    }
}
```

Le code suivant appelle la `CheckChanged` méthode lorsque la case à cocher est modifiée dans l’interface utilisateur :

```razor
<input type="checkbox" class="form-check-input" @onchange="CheckChanged" />

@code {
    private void CheckChanged()
    {
        ...
    }
}
```

Les gestionnaires d’événements peuvent également être asynchrones et <xref:System.Threading.Tasks.Task>retourner un. Il n’est pas nécessaire d’appeler manuellement [StateHasChanged](xref:blazor/lifecycle#state-changes). Les exceptions sont journalisées lorsqu’elles se produisent.

Dans l’exemple suivant, `UpdateHeading` est appelé de façon asynchrone quand le bouton est sélectionné :

```razor
<button class="btn btn-primary" @onclick="UpdateHeading">
    Update heading
</button>

@code {
    private async Task UpdateHeading(MouseEventArgs e)
    {
        ...
    }
}
```

## <a name="event-argument-types"></a>Types d’arguments d’événement

Pour certains événements, les types d’arguments d’événement sont autorisés. La spécification d’un type d’événement dans l’appel de méthode n’est nécessaire que si le type d’événement est utilisé dans la méthode.

Les `EventArgs` informations prises en charge sont indiquées dans le tableau suivant.

| Événement            | Classe                | Remarques et événements DOM |
| ---------------- | -------------------- | -------------------- |
| Presse-papiers        | `ClipboardEventArgs` | `oncut`, `oncopy`, `onpaste` |
| Glissement             | `DragEventArgs`      | `ondrag`, `ondragstart`, `ondragenter`, `ondragleave`, `ondragover`, `ondrop`, `ondragend`<br><br>`DataTransfer`et `DataTransferItem` contiennent des données d’élément glissées. |
| Error            | `ErrorEventArgs`     | `onerror` |
| Événement            | `EventArgs`          | *Général*<br>`onactivate`, `onbeforeactivate`, `onbeforedeactivate`, `ondeactivate`, `onended`, `onfullscreenchange`, `onfullscreenerror`, `onloadeddata`, `onloadedmetadata`, `onpointerlockchange`, `onpointerlockerror`, `onreadystatechange`, `onscroll`<br><br>*Presse-papiers*<br>`onbeforecut`, `onbeforecopy`, `onbeforepaste`<br><br>*Input*<br>`oninvalid`, `onreset`, `onselect`, `onselectionchange`, `onselectstart`, `onsubmit`<br><br>*Média*<br>`oncanplay`, `oncanplaythrough`, `oncuechange`, `ondurationchange`, `onemptied`, `onpause`, `onplay`, `onplaying`, `onratechange`, `onseeked`, `onseeking`, `onstalled`, `onstop`, `onsuspend`, `ontimeupdate`, `onvolumechange`, `onwaiting` |
| Focus            | `FocusEventArgs`     | `onfocus`, `onblur`, `onfocusin`, `onfocusout`<br><br>N’inclut pas la `relatedTarget`prise en charge de. |
| Entrée            | `ChangeEventArgs`    | `onchange`, `oninput` |
| Clavier         | `KeyboardEventArgs`  | `onkeydown`, `onkeypress`, `onkeyup` |
| Souris            | `MouseEventArgs`     | `onclick`, `oncontextmenu`, `ondblclick`, `onmousedown`, `onmouseup`, `onmouseover`, `onmousemove`, `onmouseout` |
| Pointeur de souris    | `PointerEventArgs`   | `onpointerdown`, `onpointerup`, `onpointercancel`, `onpointermove`, `onpointerover`, `onpointerout`, `onpointerenter`, `onpointerleave`, `ongotpointercapture`, `onlostpointercapture` |
| Roulette de la souris      | `WheelEventArgs`     | `onwheel`, `onmousewheel` |
| Progression         | `ProgressEventArgs`  | `onabort`, `onload`, `onloadend`, `onloadstart`, `onprogress`, `ontimeout` |
| Toucher            | `TouchEventArgs`     | `ontouchstart`, `ontouchend`, `ontouchmove`, `ontouchenter`, `ontouchleave`, `ontouchcancel`<br><br>`TouchPoint`représente un point de contact unique sur un appareil tactile. |

Pour plus d’informations, consultez les ressources suivantes :

* [Classes EventArgs dans la source de référence ASP.net Core (branche dotnet/aspnetcore ou version 3.1)](https://github.com/dotnet/aspnetcore/tree/release/3.1/src/Components/Web/src/Web).
* [MDN Web docs : GlobalEventHandlers](https://developer.mozilla.org/docs/Web/API/GlobalEventHandlers) &ndash; contient des informations sur les éléments HTML qui prennent en charge chaque événement DOM.

## <a name="lambda-expressions"></a>Expressions lambda

Les [expressions lambda](/dotnet/csharp/programming-guide/statements-expressions-operators/lambda-expressions) peuvent également être utilisées :

```razor
<button @onclick="@(e => Console.WriteLine("Hello, world!"))">Say hello</button>
```

Il est souvent pratique de se rapprocher de valeurs supplémentaires, par exemple lors de l’itération sur un ensemble d’éléments. L’exemple suivant crée trois boutons, chacun d’entre eux `UpdateHeading` qui passe un argument d'`MouseEventArgs`événement () et son numéro`buttonNumber`de bouton () lorsqu’ils sont sélectionnés dans l’interface utilisateur :

```razor
<h2>@message</h2>

@for (var i = 1; i < 4; i++)
{
    var buttonNumber = i;

    <button class="btn btn-primary"
            @onclick="@(e => UpdateHeading(e, buttonNumber))">
        Button #@i
    </button>
}

@code {
    private string message = "Select a button to learn its position.";

    private void UpdateHeading(MouseEventArgs e, int buttonNumber)
    {
        message = $"You selected Button #{buttonNumber} at " +
            $"mouse position: {e.ClientX} X {e.ClientY}.";
    }
}
```

> [!NOTE]
> N’utilisez **pas** la variable de boucle`i`() dans `for` une boucle directement dans une expression lambda. Dans le cas contraire, la même variable est utilisée par `i`toutes les expressions lambda, ce qui signifie que la valeur de est identique dans toutes les expressions lambda. Capturez toujours sa valeur dans une variable locale`buttonNumber` (dans l’exemple précédent), puis utilisez-la.

## <a name="eventcallback"></a>EventCallback suivante

Un scénario courant avec des composants imbriqués est le désir d’exécuter la méthode d’un composant parent lorsqu’un événement de&mdash;composant enfant se produit, `onclick` par exemple, lorsqu’un événement se produit dans l’enfant. Pour exposer des événements entre les composants, `EventCallback`utilisez un. Un composant parent peut affecter une méthode de rappel à un composant enfant `EventCallback`.

`ChildComponent` Dans l’exemple d’application (*Components/ChildComponent. Razor*), vous montre comment `onclick` un gestionnaire de bouton est configuré pour `EventCallback` recevoir un délégué de l' `ParentComponent`exemple de. Le `EventCallback` est typé avec `MouseEventArgs`, ce qui est approprié pour `onclick` un événement à partir d’un périphérique :

[!code-razor[](common/samples/3.x/BlazorWebAssemblySample/Components/ChildComponent.razor?highlight=5-7,17-18)]

`ParentComponent` Définit le (`OnClickCallback`) de `EventCallback<T>` l’enfant sur sa `ShowMessage` méthode.

*Pages/ParentComponent. Razor*:

```razor
@page "/ParentComponent"

<h1>Parent-child example</h1>

<ChildComponent Title="Panel Title from Parent"
                OnClickCallback="@ShowMessage">
    Content of the child component is supplied
    by the parent component.
</ChildComponent>

<p><b>@messageText</b></p>

@code {
    private string messageText;

    private void ShowMessage(MouseEventArgs e)
    {
        messageText = $"Blaze a new trail with Blazor! ({e.ScreenX}, {e.ScreenY})";
    }
}
```

Lorsque le bouton est sélectionné dans le `ChildComponent`:

* La `ParentComponent`méthode `ShowMessage` de est appelée. `messageText`est mis à jour et affiché `ParentComponent`dans le.
* Un appel à [StateHasChanged](xref:blazor/lifecycle#state-changes) n’est pas requis dans la méthode du`ShowMessage`rappel (). `StateHasChanged`est appelé automatiquement pour rerestituer `ParentComponent`le, tout comme les événements enfants déclenchent le rerendu des composants dans les gestionnaires d’événements qui s’exécutent dans l’enfant.

`EventCallback`et `EventCallback<T>` autorisent les délégués asynchrones. `EventCallback<T>`est fortement typé et requiert un type d’argument spécifique. `EventCallback`est faiblement typé et autorise tout type d’argument.

```razor
<ChildComponent 
    OnClickCallback="@(async () => { await Task.Yield(); messageText = "Blaze It!"; })" />
```

Appelez `EventCallback` ou `EventCallback<T>` avec et `InvokeAsync` en attente de <xref:System.Threading.Tasks.Task>:

```csharp
await callback.InvokeAsync(arg);
```

Utilisez `EventCallback` et `EventCallback<T>` pour la gestion des événements et les paramètres de composant de liaison.

Préférez le fortement typé `EventCallback<T>` `EventCallback`. `EventCallback<T>`fournit un meilleur retour d’erreur aux utilisateurs du composant. Comme pour d’autres gestionnaires d’événements d’interface utilisateur, la spécification du paramètre d’événement est facultative. Utilisez `EventCallback` quand aucune valeur n’est passée au rappel.

## <a name="prevent-default-actions"></a>Empêcher les actions par défaut

Utilisez l' [`@on{EVENT}:preventDefault`](xref:mvc/views/razor#oneventpreventdefault) attribut directive pour empêcher l’action par défaut d’un événement.

Quand une clé est sélectionnée sur un appareil d’entrée et que le focus de l’élément se trouve sur une zone de texte, un navigateur affiche normalement le caractère de la clé dans la zone de texte. Dans l’exemple suivant, le comportement par défaut est évité en spécifiant l' `@onkeypress:preventDefault` attribut directive. Le compteur est incrémenté et la **+** clé n’est pas capturée dans la `<input>` valeur de l’élément :

```razor
<input value="@count" @onkeypress="KeyHandler" @onkeypress:preventDefault />

@code {
    private int count = 0;

    private void KeyHandler(KeyboardEventArgs e)
    {
        if (e.Key == "+")
        {
            count++;
        }
    }
}
```

La spécification `@on{EVENT}:preventDefault` de l’attribut sans valeur équivaut à `@on{EVENT}:preventDefault="true"`.

La valeur de l’attribut peut également être une expression. Dans l’exemple suivant, `shouldPreventDefault` est un `bool` champ défini sur `true` ou `false`:

```razor
<input @onkeypress:preventDefault="shouldPreventDefault" />
```

Un gestionnaire d’événements n’est pas nécessaire pour empêcher l’action par défaut. Le gestionnaire d’événements et les scénarios d’action par défaut peuvent être utilisés indépendamment.

## <a name="stop-event-propagation"></a>Arrêter la propagation des événements

Utilisez l' [`@on{EVENT}:stopPropagation`](xref:mvc/views/razor#oneventstoppropagation) attribut directive pour arrêter la propagation des événements.

Dans l’exemple suivant, la sélection de la case à cocher empêche les événements `<div>` Click du deuxième enfant de se `<div>`propager vers le parent :

```razor
<label>
    <input @bind="stopPropagation" type="checkbox" />
    Stop Propagation
</label>

<div @onclick="OnSelectParentDiv">
    <h3>Parent div</h3>

    <div @onclick="OnSelectChildDiv">
        Child div that doesn't stop propagation when selected.
    </div>

    <div @onclick="OnSelectChildDiv" @onclick:stopPropagation="stopPropagation">
        Child div that stops propagation when selected.
    </div>
</div>

@code {
    private bool stopPropagation = false;

    private void OnSelectParentDiv() => 
        Console.WriteLine($"The parent div was selected. {DateTime.Now}");
    private void OnSelectChildDiv() => 
        Console.WriteLine($"A child div was selected. {DateTime.Now}");
}
```
