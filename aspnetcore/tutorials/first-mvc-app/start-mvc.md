---
title: Bien démarrer avec ASP.NET Core MVC
author: rick-anderson
description: Découvrez comment bien démarrer avec ASP.NET Core MVC.
ms.author: riande
ms.date: 10/16/2019
no-loc:
- Blazor
- Identity
- Let's Encrypt
- Razor
- SignalR
uid: tutorials/first-mvc-app/start-mvc
ms.openlocfilehash: 0b28e99d9c56d92e157e9010880e58ec5165196a
ms.sourcegitcommit: 70e5f982c218db82aa54aa8b8d96b377cfc7283f
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 05/04/2020
ms.locfileid: "82774983"
---
# <a name="get-started-with-aspnet-core-mvc"></a>Bien démarrer avec ASP.NET Core MVC

Par [Rick Anderson](https://twitter.com/RickAndMSFT)

::: moniker range=">= aspnetcore-3.0"

[!INCLUDE [consider RP](~/includes/razor.md)]

Ce didacticiel décrit les principes fondamentaux liés à la génération d’une application web dans ASP.NET Core MVC.

L’application gère une base de données de titres de films. Vous allez apprendre à effectuer les actions suivantes :

> [!div class="checklist"]
> * Créez une application web.
> * Ajouter et structurer un modèle.
> * Utiliser une base de données.
> * Ajouter une fonctionnalité de recherche et de validation.

À la fin, vous obtenez une application qui peut gérer et afficher des données de films.

[!INCLUDE[](~/includes/mvc-intro/download.md)]

## <a name="prerequisites"></a>Prérequis

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

[!INCLUDE[](~/includes/net-core-prereqs-vs-3.1.md)]

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

[!INCLUDE[](~/includes/net-core-prereqs-vsc-3.1.md)]

# <a name="visual-studio-for-mac"></a>[Visual Studio pour Mac](#tab/visual-studio-mac)

[!INCLUDE[](~/includes/net-core-prereqs-mac-3.1.md)]

---

## <a name="create-a-web-app"></a>Créer une application web

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

* Dans Visual Studio, sélectionnez **Créer un projet**.

* Sélectionnez **Application web ASP.NET Core**, puis **Suivant**.

![Nouvelle application web ASP.NET Core](start-mvc/_static/np_2.1.png)

* Nommez le projet **MvcMovie**, puis sélectionnez **Créer**. Il est important de nommer le projet **MvcMovie** pour que l’espace de noms corresponde quand vous copiez du code.

  ![Nouvelle application web ASP.NET Core](start-mvc/_static/config.png)

* Sélectionnez **Application web (modèle-vue-contrôleur)**, puis **Créer**.

![Boîte de dialogue Nouveau projet, .NET Core dans le volet gauche, web ASP.NET Core ](start-mvc/_static/new_project30.png)

Visual Studio a utilisé le modèle par défaut pour le projet MVC que vous venez de créer. Vous disposez maintenant d’une application fonctionnelle en entrant un nom de projet et en sélectionnant quelques options. Il s’agit d’un projet de démarrage de base.

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

Il part du principe que vous connaissez déjà VS Code. Pour plus d’informations, consultez [Bien démarrer avec VS Code](https://code.visualstudio.com/docs) et [Aide de Visual Studio Code](#visual-studio-code-help).

* Ouvrez le [terminal intégré](https://code.visualstudio.com/docs/editor/integrated-terminal).
* Accédez à un répertoire (`cd`) destiné à contenir le projet.
* Exécutez la commande suivante :

   ```dotnetcli
   dotnet new mvc -o MvcMovie
   code -r MvcMovie
   ```

  * Une boîte de dialogue s’affiche avec les **ressources requises pour la génération et le débogage dans’MvcMovie'. Ajoutez-les ?**  Sélectionnez **Oui**.

  * `dotnet new mvc -o MvcMovie` : crée un nouveau projet ASP.NET Core MVC dans le dossier *MvcMovie*.
  * `code -r MvcMovie`: Charge le fichier projet *MvcMovie. csproj* dans Visual Studio code.

# <a name="visual-studio-for-mac"></a>[Visual Studio pour Mac](#tab/visual-studio-mac)

* Sélectionnez **Fichier** > **Nouvelle solution**.

  ![macOS - Nouvelle solution](./start-mvc/_static/new_project_vsmac.png)

* Sélectionnez **.NET Core** > **Application** > **Application web (modèle-vue-contrôleur)** > **Suivant**.

  ![macOS - Boîte de dialogue Nouveau projet](./start-mvc/_static/new_project_mvc_vsmac.png)

* Dans la boîte de dialogue **configurer votre nouvelle API Web ASP.net Core** , définissez le **Framework cible** de **.net Core 3,1**.

  ![sélection de macOS .NET Core 3,1](./start-mvc/_static/new_project_31_vsmac.png)

* Nommez le projet **MvcMovie**, puis sélectionnez **Créer**.

---

### <a name="run-the-app"></a>Exécuter l’application

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

Sélectionnez **Ctrl-F5** pour exécuter l'application en mode non-débogage.

[!INCLUDE[](~/includes/trustCertVS.md)]

* Visual Studio démarre [IIS Express](/iis/extensions/introduction-to-iis-express/iis-express-overview) et exécute l’application. Notez que la barre d’adresse affiche `localhost:port#`, et non quelque chose comme `example.com`. En effet, `localhost` est le nom d’hôte standard de votre ordinateur local. Quand Visual Studio crée un projet web, un port aléatoire est utilisé pour le serveur web.
* Si vous lancez l’application avec Ctrl+F5 (mode sans débogage), vous pouvez apporter des modifications au code, enregistrer le fichier, actualiser le navigateur et examiner les modifications apportées au code. De nombreux développeurs préfèrent utiliser le mode non-débogage pour lancer rapidement l’application et voir les modifications.
* Vous pouvez lancer l’application en mode débogage ou non-débogage à partir de l’élément de menu **Déboguer** :

  ![Menu Déboguer](start-mvc/_static/debug_menu.png)

* Vous pouvez déboguer l’application en sélectionnant le bouton **IIS Express**

  ![IIS Express](start-mvc/_static/iis_express.png)

  L’image suivante montre l’application :

  ![Page d’accueil ou d’index](start-mvc/_static/home2.2.png)

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

Appuyez sur Ctrl+F5 pour exécuter sans le débogueur.

[!INCLUDE[](~/includes/trustCertVSC.md)]

  Visual Studio Code démarre [Kestrel](xref:fundamentals/servers/kestrel), lance un navigateur, puis accède à `https://localhost:5001`. La barre d’adresses affiche `localhost:port:5001` au lieu de quelque chose qui ressemble à `example.com`. La raison en est que `localhost` est le nom d’hôte standard de l’ordinateur local. Localhost traite uniquement les requêtes web de l’ordinateur local.

  Si vous lancez l’application avec Ctrl+F5 (mode sans débogage), vous pouvez apporter des modifications au code, enregistrer le fichier, actualiser le navigateur et examiner les modifications apportées au code. De nombreux développeurs préfèrent utiliser le mode sans débogage pour actualiser les modifications des pages et des vues.

  ![Page d’accueil ou d’index](start-mvc/_static/home2.2.png)

# <a name="visual-studio-for-mac"></a>[Visual Studio pour Mac](#tab/visual-studio-mac)

Sélectionnez **exécuter** > **Démarrer sans débogage** pour lancer l’application. Visual Studio pour Mac démarre le serveur [Kestrel](xref:fundamentals/servers/index#kestrel), lance un navigateur et accède à `http://localhost:port`, où *port* est un numéro de port choisi de façon aléatoire.

[!INCLUDE[](~/includes/trustCertMac.md)]

* La barre d’adresses affiche `localhost:port#` au lieu de quelque chose qui ressemble à `example.com`. En effet, `localhost` est le nom d’hôte standard de votre ordinateur local. Quand Visual Studio crée un projet web, un port aléatoire est utilisé pour le serveur web. Quand vous exécutez l’application, vous voyez un autre numéro de port.
* Vous pouvez lancer l’application en mode débogage ou non-débogage à partir du menu **Exécuter**.

  L’image suivante montre l’application :

  ![Page d’accueil ou d’index](./start-mvc/_static/output_macos.png)

---

[!INCLUDE[](~/includes/vs-vsc-vsmac-help.md)]

Dans la prochaine partie de ce didacticiel, vous allez découvrir MVC et commencer à écrire du code.

> [!div class="step-by-step"]
> [Suivant](adding-controller.md)

::: moniker-end

::: moniker range="< aspnetcore-3.0"

[!INCLUDE [consider RP](~/includes/razor.md)]

Ce didacticiel décrit les principes fondamentaux liés à la génération d’une application web dans ASP.NET Core MVC.

L’application gère une base de données de titres de films. Vous allez apprendre à effectuer les actions suivantes :

> [!div class="checklist"]
> * Créez une application web.
> * Ajouter et structurer un modèle.
> * Utiliser une base de données.
> * Ajouter une fonctionnalité de recherche et de validation.

À la fin, vous obtenez une application qui peut gérer et afficher des données de films.

[!INCLUDE[](~/includes/mvc-intro/download.md)]

## <a name="prerequisites"></a>Prérequis

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

[!INCLUDE[](~/includes/net-core-prereqs-vs2019-2.2.md)]

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

[!INCLUDE[](~/includes/net-core-prereqs-vsc-2.2.md)]

# <a name="visual-studio-for-mac"></a>[Visual Studio pour Mac](#tab/visual-studio-mac)

[!INCLUDE[](~/includes/net-core-prereqs-mac-2.2.md)]

---
## <a name="create-a-web-app"></a>Créer une application web

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

* Dans Visual Studio, sélectionnez **Créer un projet**.

* Sélectionnez **Application web ASP.NET Core**, puis **Suivant**.

![Nouvelle application web ASP.NET Core](start-mvc/_static/np_2.1.png)

* Nommez le projet **MvcMovie**, puis sélectionnez **Créer**. Il est important de nommer le projet **MvcMovie** pour que l’espace de noms corresponde quand vous copiez du code.

  ![Nouvelle application web ASP.NET Core](start-mvc/_static/config.png)


* Sélectionnez **Application web (modèle-vue-contrôleur)**, puis **Créer**.

![Boîte de dialogue Nouveau projet, .NET Core dans le volet gauche, web ASP.NET Core ](start-mvc/_static/new_project22-21.png)

Visual Studio a utilisé le modèle par défaut pour le projet MVC que vous venez de créer. Vous disposez maintenant d’une application fonctionnelle en entrant un nom de projet et en sélectionnant quelques options. Il s’agit d’un projet de démarrage de base qui constitue un bon point de départ.

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

Il part du principe que vous connaissez déjà VS Code. Pour plus d’informations, consultez [Bien démarrer avec VS Code](https://code.visualstudio.com/docs) et [Aide de Visual Studio Code](#visual-studio-code-help).

* Ouvrez le [terminal intégré](https://code.visualstudio.com/docs/editor/integrated-terminal).
* Accédez à un répertoire (`cd`) destiné à contenir le projet.
* Exécutez la commande suivante :

   ```dotnetcli
   dotnet new mvc -o MvcMovie
   code -r MvcMovie
   ```

  * Une boîte de dialogue s’affiche avec les **ressources requises pour la génération et le débogage dans’MvcMovie'. Ajoutez-les ?**  Sélectionnez **Oui**.

  * `dotnet new mvc -o MvcMovie` : crée un nouveau projet ASP.NET Core MVC dans le dossier *MvcMovie*.
  * `code -r MvcMovie`: Charge le fichier projet *MvcMovie. csproj* dans Visual Studio code.

# <a name="visual-studio-for-mac"></a>[Visual Studio pour Mac](#tab/visual-studio-mac)

* Sélectionnez **Fichier** > **Nouvelle solution**.

  ![macOS - Nouvelle solution](./start-mvc/_static/new_project_vsmac.png)

* Sélectionnez **.NET Core** > **Application** > **Application web (modèle-vue-contrôleur)** > **Suivant**.

  ![macOS - Boîte de dialogue Nouveau projet](./start-mvc/_static/new_project_mvc_vsmac.png)

* Dans la boîte de dialogue **configurer votre nouvelle API Web ASP.net Core** , acceptez le **Framework cible** par défaut de **.net Core 2,2**.

  ![sélection de .NET Core 2.2 pour macOS](./start-mvc/_static/new_project_22_vsmac.png)

* Nommez le projet **MvcMovie**, puis sélectionnez **Créer**.

---

### <a name="run-the-app"></a>Exécuter l’application

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

Sélectionnez **Ctrl-F5** pour exécuter l'application en mode non-débogage.

[!INCLUDE[](~/includes/trustCertVS.md)]

* Visual Studio démarre [IIS Express](/iis/extensions/introduction-to-iis-express/iis-express-overview) et exécute l’application. Notez que la barre d’adresse affiche `localhost:port#`, et non quelque chose comme `example.com`. En effet, `localhost` est le nom d’hôte standard de votre ordinateur local. Quand Visual Studio crée un projet web, un port aléatoire est utilisé pour le serveur web.
* Si vous lancez l’application avec Ctrl+F5 (mode sans débogage), vous pouvez apporter des modifications au code, enregistrer le fichier, actualiser le navigateur et examiner les modifications apportées au code. De nombreux développeurs préfèrent utiliser le mode non-débogage pour lancer rapidement l’application et voir les modifications.
* Vous pouvez lancer l’application en mode débogage ou non-débogage à partir de l’élément de menu **Déboguer** :

  ![Menu Déboguer](start-mvc/_static/debug_menu.png)

* Vous pouvez déboguer l’application en sélectionnant le bouton **IIS Express**

  ![IIS Express](start-mvc/_static/iis_express.png)

* Sélectionnez **Accepter** pour accepter le suivi. Cette application n’effectue pas le suivi d’informations personnelles. Le code généré par le modèle inclut des ressources qui aident à satisfaire au [Règlement général sur la protection des données (RGPD)](xref:security/gdpr).

  ![Page d’accueil ou d’index](start-mvc/_static/privacy.png)

  L’illustration suivante montre l’application une fois le suivi accepté :

  ![Page d’accueil ou d’index](start-mvc/_static/home2.2.png)

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

Appuyez sur Ctrl+F5 pour exécuter sans le débogueur.

[!INCLUDE[](~/includes/trustCertVSC.md)]

  Visual Studio Code démarre [Kestrel](xref:fundamentals/servers/kestrel), lance un navigateur, puis accède à `https://localhost:5001`. La barre d’adresses affiche `localhost:port:5001` au lieu de quelque chose qui ressemble à `example.com`. La raison en est que `localhost` est le nom d’hôte standard de l’ordinateur local. Localhost traite uniquement les requêtes web de l’ordinateur local.

  Si vous lancez l’application avec Ctrl+F5 (mode sans débogage), vous pouvez apporter des modifications au code, enregistrer le fichier, actualiser le navigateur et examiner les modifications apportées au code. De nombreux développeurs préfèrent utiliser le mode sans débogage pour actualiser les modifications des pages et des vues.

* Sélectionnez **Accepter** pour accepter le suivi. Cette application n’effectue pas le suivi d’informations personnelles. Le code généré par le modèle inclut des ressources qui aident à satisfaire au [Règlement général sur la protection des données (RGPD)](xref:security/gdpr).

  ![Page d’accueil ou d’index](start-mvc/_static/privacy.png)

  L’illustration suivante montre l’application une fois le suivi accepté :

  ![Page d’accueil ou d’index](start-mvc/_static/home2.2.png)

# <a name="visual-studio-for-mac"></a>[Visual Studio pour Mac](#tab/visual-studio-mac)

Sélectionnez **exécuter** > **Démarrer sans débogage** pour lancer l’application. Visual Studio pour Mac démarre le serveur [Kestrel](xref:fundamentals/servers/index#kestrel), lance un navigateur et accède à `http://localhost:port`, où *port* est un numéro de port choisi de façon aléatoire.

[!INCLUDE[](~/includes/trustCertMac.md)]

* La barre d’adresses affiche `localhost:port#` au lieu de quelque chose qui ressemble à `example.com`. En effet, `localhost` est le nom d’hôte standard de votre ordinateur local. Quand Visual Studio crée un projet web, un port aléatoire est utilisé pour le serveur web. Quand vous exécutez l’application, vous voyez un autre numéro de port.
* Vous pouvez lancer l’application en mode débogage ou non-débogage à partir du menu **Exécuter**.

* Sélectionnez **Accepter** pour accepter le suivi. Cette application n’effectue pas le suivi d’informations personnelles. Le code généré par le modèle inclut des ressources qui aident à satisfaire au [Règlement général sur la protection des données (RGPD)](xref:security/gdpr).

  ![Page d’accueil ou d’index](./start-mvc/_static/output_privacy_macos.png)

  L’illustration suivante montre l’application une fois le suivi accepté :

  ![Page d’accueil ou d’index](./start-mvc/_static/output_macos.png)

---

[!INCLUDE[](~/includes/vs-vsc-vsmac-help.md)]

Dans la prochaine partie de ce didacticiel, vous allez découvrir MVC et commencer à écrire du code.

> [!div class="step-by-step"]
> [Suivant](adding-controller.md)

::: moniker-end
