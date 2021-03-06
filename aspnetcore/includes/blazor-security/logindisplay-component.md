Le `LoginDisplay` composant (*Shared/LoginDisplay. Razor*) est affiché dans `MainLayout` le composant (*Shared/MainLayout. Razor*) et gère les comportements suivants :

* Pour les utilisateurs authentifiés :
  * Affiche le nom d’utilisateur actuel.
  * Offre un bouton permettant de se déconnecter de l’application.
* Pour les utilisateurs anonymes, offre la possibilité de se connecter.

```razor
@using Microsoft.AspNetCore.Components.Authorization
@using Microsoft.AspNetCore.Components.WebAssembly.Authentication
@inject NavigationManager Navigation
@inject SignOutSessionStateManager SignOutManager

<AuthorizeView>
    <Authorized>
        Hello, @context.User.Identity.Name!
        <button class="nav-link btn btn-link" @onclick="BeginSignOut">
            Log out
        </button>
    </Authorized>
    <NotAuthorized>
        <a href="authentication/login">Log in</a>
    </NotAuthorized>
</AuthorizeView>

@code {
    private async Task BeginSignOut(MouseEventArgs args)
    {
        await SignOutManager.SetSignOutState();
        Navigation.NavigateTo("authentication/logout");
    }
}
```
