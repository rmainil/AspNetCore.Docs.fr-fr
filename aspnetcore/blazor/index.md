---
title: Présentation de ASP.NET CoreBlazor
author: guardrex
description: Explorez ASP.NET Core Blazor, un moyen de créer une interface utilisateur Web interactive côté client avec .net dans une application ASP.net core.
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc, seoapril2019
ms.date: 03/25/2020
no-loc:
- Blazor
- Identity
- Let's Encrypt
- Razor
- SignalR
uid: blazor/index
ms.openlocfilehash: ced3e2cc0428fccf6f0b2eba7a3f045e07002234
ms.sourcegitcommit: 70e5f982c218db82aa54aa8b8d96b377cfc7283f
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 05/04/2020
ms.locfileid: "82771948"
---
# <a name="introduction-to-aspnet-core-blazor"></a>Présentation de ASP.NET CoreBlazor

Par [Daniel Roth](https://github.com/danroth27) et [Luke Latham](https://github.com/guardrex)

*Bienvenue Blazor!*

Blazorest une infrastructure permettant de créer une interface utilisateur Web interactive côté client avec .NET :

* Créez des interfaces utilisateur interactives riches à l’aide de C# plutôt que JavaScript.
* Partagez la logique d’application côté serveur et côté client écrite dans .NET.
* Affichez l’interface utilisateur en langage HTML et CSS pour une large prise en charge des navigateurs, y compris les navigateurs mobiles.
* Intégrez les plateformes d’hébergement modernes, telles que l' [amarrage](/dotnet/standard/microservices-architecture/container-docker-introduction/index).

L’utilisation de .NET dans le développement web côté client offre les avantages suivants :

* Écriture de code dans C# plutôt que dans JavaScript.
* Tirez parti de l’écosystème .NET existant des bibliothèques .NET.
* Partagez la logique de l’application sur le serveur et le client.
* Bénéficiez des performances, de la fiabilité et de la sécurité de .NET.
* Restez productif grâce à Visual Studio sur Windows, Linux et macOS.
* Développez avec un ensemble commun de langages, de frameworks et d’outils stables, riches en fonctionnalités et faciles à utiliser.

## <a name="components"></a>Components

Blazorles applications sont basées sur des *composants*. Un composant dans Blazor est un élément de l’interface utilisateur, tel qu’une page, une boîte de dialogue ou un formulaire de saisie de données.

Les composants sont des classes .NET intégrées dans des assemblys .NET qui :

* Définissent la logique de rendu de l’interface utilisateur flexible.
* Gèrent les événements de l’utilisateur.
* Peuvent être imbriqués et réutilisés.
* Peut être partagé et distribué en tant que [ Razor bibliothèques de classes](xref:razor-pages/ui-class) ou [packages NuGet](/nuget/what-is-nuget).

La classe Component est généralement écrite sous la forme d’une [Razor](xref:mvc/views/razor) page de balisage avec une extension de fichier *. Razor* . Les composants Blazor dans sont officiellement désignés sous * Razor *le terme de composants. Razorest une syntaxe pour combiner le balisage HTML avec du code C# conçu pour la productivité des développeurs. Razorvous permet de basculer entre le balisage HTML et C# dans le même fichier avec la prise en charge [IntelliSense](/visualstudio/ide/using-intellisense) . RazorLes pages et MVC utilisent Razorégalement. Contrairement Razor aux pages et MVC, qui sont générés autour d’un modèle de demande/réponse, les composants sont utilisés spécifiquement pour la logique et la composition de l’interface utilisateur côté client.

Le balisage suivant Razor illustre un composant (*Dialog. Razor*), qui peut être imbriqué dans un autre composant :

```razor
<div>
    <h1>@Title</h1>

    @ChildContent

    <button @onclick="OnYes">Yes!</button>
</div>

@code {
    [Parameter]
    public string Title { get; set; }

    [Parameter]
    public RenderFragment ChildContent { get; set; }

    private void OnYes()
    {
        Console.WriteLine("Write to the console in C#! 'Yes' button was selected.");
    }
}
```

Le contenu du corps (`ChildContent`) et le titre (`Title`) de la boîte de dialogue sont fournis par le composant qui utilise ce composant dans son interface utilisateur. `OnYes` est une méthode C# déclenchée par l’événement `onclick` du bouton.

Blazorutilise des balises HTML naturelles pour la composition de l’interface utilisateur. Les éléments HTML spécifient des composants et les attributs d’une balise passent les valeurs aux propriétés d’un composant.

Dans l’exemple suivant, le composant `Index` utilise le composant `Dialog`. `ChildContent` et `Title` sont définis par les attributs et le contenu de l’élément `<Dialog>`.

*Index.razor* :

```razor
@page "/"

<h1>Hello, world!</h1>

Welcome to your new app.

<Dialog Title="Blazor">
    Do you want to <i>learn more</i> about Blazor?
</Dialog>
```

La boîte de dialogue est affichée lorsque le parent (*Index.razor*) est accessible dans un navigateur :

![Composant de boîte de dialogue affiché dans le navigateur](index/_static/dialog.png)

Quand ce composant est utilisé dans l’application, IntelliSense dans [Visual Studio](/visualstudio/ide/using-intellisense)et [Visual Studio Code](https://code.visualstudio.com/docs/editor/intellisense) accélère le développement avec l’achèvement de la syntaxe et des paramètres.

Les composants s’affichent dans une représentation en mémoire du DOM (Document Object Model) du navigateur appelé *arborescence de rendu*, utilisée pour mettre à jour l’interface utilisateur de manière flexible et efficace.

## <a name="blazor-webassembly"></a>BlazorWebassembly

[!INCLUDE[](~/includes/blazorwasm-preview-notice.md)]

BlazorWebassembly est une infrastructure d’application à page unique pour la création d’applications Web interactives côté client avec .NET. BlazorWebassembly utilise des normes Web ouvertes sans plug-ins ou code transpilation et fonctionne dans tous les navigateurs Web modernes, y compris les navigateurs mobiles.

L’exécution de code .NET à l’intérieur des navigateurs web est rendue possible grâce à [WebAssembly](https://webassembly.org) (abrégé en *wasm*). WebAssembly est un format bytecode compact optimisé pour un téléchargement rapide et une vitesse d’exécution maximale. WebAssembly est un standard web ouvert pris en charge dans les navigateurs web sans plug-in.

Le code WebAssembly peut accéder à toutes les fonctionnalités du navigateur via JavaScript, appelé *interopérabilité JavaScript* (ou *interop JavaScript*). Le code .NET exécuté via WebAssembly dans le navigateur s’exécute dans le bac à sable JavaScript du navigateur avec les protections offertes par le bac à sable contre les actions malveillantes sur l’ordinateur client.

![BlazorWebassembly exécute du code .NET dans le navigateur avec webassembly.](index/_static/blazor-webassembly.png)

Quand une Blazor application webassembly est générée et exécutée dans un navigateur :

* Les fichiers de code Razor C# et les fichiers sont compilés dans des assemblys .net.
* Les assemblys et le runtime .NET sont téléchargés dans le navigateur.
* BlazorWebassembly amorce le Runtime .NET et configure le runtime pour charger les assemblys de l’application. Le Blazor Runtime webassembly utilise l’interopérabilité JavaScript pour gérer les appels de l’API de manipulation et du navigateur DOM.

La taille de l’application publiée, sa *taille de charge utile*, est un facteur de performance essentiel pour qu’une application soit facile à utiliser. Le téléchargement d’une application volumineuse dans un navigateur prend un certain temps, ce qui nuit à l’expérience utilisateur. BlazorWebassembly optimise la taille de la charge utile pour réduire les temps de téléchargement :

* Du code inutilisé est extrait de l’application lorsqu’elle est publiée par l’[éditeur de liens de langage intermédiaire (IL)](xref:host-and-deploy/blazor/configure-linker).
* Réponses HTTP compressées.
* Le runtime .NET et les assemblys sont mis en cache dans le navigateur.

## <a name="blazor-server"></a>BlazorServeurs

Blazordissocie la logique de rendu des composants de l’application des mises à jour de l’interface utilisateur. BlazorLe serveur prend en charge Razor l’hébergement de composants sur le serveur dans une application ASP.net core. Les mises à jour de l’interface utilisateur [SignalR](xref:signalr/introduction) sont gérées via une connexion.

Le runtime gère l’envoi des événements d’interface utilisateur du navigateur au serveur et applique les mises à jour de l’interface utilisateur renvoyées par le serveur dans le navigateur après avoir exécuté les composants.

La connexion utilisée par Blazor le serveur pour communiquer avec le navigateur est également utilisée pour gérer les appels Interop JavaScript.

![BlazorLe serveur exécute le code .NET sur le serveur et interagit avec le Document Object Model sur le client SignalR via une connexion](index/_static/blazor-server.png)

## <a name="javascript-interop"></a>Interopérabilité JavaScript

Pour les applications qui nécessitent des bibliothèques JavaScript tierces et l’accès à des API de navigateur, les composants interagissent avec JavaScript. Les composants peuvent utiliser les mêmes API ou bibliothèques que JavaScript. Le code C# peut appeler du code JavaScript, et le code JavaScript peut appeler du code C#. Pour plus d’informations, consultez les articles suivants :

* <xref:blazor/call-javascript-from-dotnet>
* <xref:blazor/call-dotnet-from-javascript>

## <a name="code-sharing-and-net-standard"></a>Partage de code et .NET Standard

Blazorimplémente [.NET Standard 2,0](/dotnet/standard/net-standard). .NET Standard est une spécification formelle d’API .NET qui sont communes aux implémentations .NET. .NET Standard bibliothèques de classes peuvent être partagées entre différentes plateformes .NET Blazor, telles que, .NET Framework, .net Core, Xamarin, mono et Unity.

Les API qui ne sont pas applicables à l’intérieur d’un navigateur web (par exemple l’accès au système de fichiers, l’ouverture d’un socket et le threading) lèvent une <xref:System.PlatformNotSupportedException>.

## <a name="additional-resources"></a>Ressources supplémentaires

* [WebAssembly](https://webassembly.org/)
* <xref:blazor/hosting-models>
* <xref:tutorials/signalr-blazor-webassembly>
* [Guide C#](/dotnet/csharp/)
* <xref:mvc/views/razor>
* [HTML](https://www.w3.org/html/)
* Liens de la communauté [Isard Blazor ](https://github.com/AdrienTorris/awesome-blazor)
