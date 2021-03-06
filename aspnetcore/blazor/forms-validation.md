---
title: ASP.NET Core Blazor les formulaires et la validation
author: guardrex
description: Découvrez comment utiliser des formulaires et des scénarios de validation Blazorde champ dans.
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 03/17/2020
no-loc:
- Blazor
- Identity
- Let's Encrypt
- Razor
- SignalR
uid: blazor/forms-validation
ms.openlocfilehash: ec2bc2867acdd1c9be42f77cb38be36abb8c8108
ms.sourcegitcommit: 84b46594f57608f6ac4f0570172c7051df507520
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 05/08/2020
ms.locfileid: "82967478"
---
# <a name="aspnet-core-blazor-forms-and-validation"></a>ASP.NET Core des formulaires et des validations éblouissants

Par [Daniel Roth](https://github.com/danroth27) et [Luke Latham](https://github.com/guardrex)

Les formulaires et la validation sont pris en charge dans éblouissant à l’aide d' [Annotations de données](xref:mvc/models/validation).

Le type `ExampleModel` suivant définit la logique de validation à l’aide d’annotations de données :

```csharp
using System.ComponentModel.DataAnnotations;

public class ExampleModel
{
    [Required]
    [StringLength(10, ErrorMessage = "Name is too long.")]
    public string Name { get; set; }
}
```

Un formulaire est défini à l' `EditForm` aide du composant. Le formulaire suivant illustre des éléments typiques, des composants et du code Razor :

```razor
<EditForm Model="@exampleModel" OnValidSubmit="HandleValidSubmit">
    <DataAnnotationsValidator />
    <ValidationSummary />

    <InputText id="name" @bind-Value="exampleModel.Name" />

    <button type="submit">Submit</button>
</EditForm>

@code {
    private ExampleModel exampleModel = new ExampleModel();

    private void HandleValidSubmit()
    {
        Console.WriteLine("OnValidSubmit");
    }
}
```

Dans l'exemple précédent :

* Le formulaire valide l’entrée utilisateur dans le `name` champ à l’aide de la validation `ExampleModel` définie dans le type. Le modèle est créé dans le bloc du `@code` composant et conservé dans un champ privé (`exampleModel`). Le champ est assigné à l' `Model` attribut de l' `<EditForm>` élément.
* Liaisons `InputText` du `@bind-Value` composant :
  * Propriété de modèle (`exampleModel.Name`) de la `InputText` propriété du `Value` composant.
  * Délégué d’événement de modification à `InputText` la propriété `ValueChanged` du composant.
* Le `DataAnnotationsValidator` composant attache la prise en charge de la validation à l’aide d’annotations de données.
* Le `ValidationSummary` composant résume les messages de validation.
* `HandleValidSubmit`est déclenché quand le formulaire est envoyé avec succès (réussit la validation).

Un ensemble de composants d’entrée intégrés est disponible pour recevoir et valider les entrées utilisateur. Les entrées sont validées lorsqu’elles sont modifiées et lorsqu’un formulaire est envoyé. Les composants d’entrée disponibles sont répertoriés dans le tableau suivant.

| Composant d’entrée | Rendu comme&hellip;       |
| --------------- | ------------------------- |
| `InputText`     | `<input>`                 |
| `InputTextArea` | `<textarea>`              |
| `InputSelect`   | `<select>`                |
| `InputNumber`   | `<input type="number">`   |
| `InputCheckbox` | `<input type="checkbox">` |
| `InputDate`     | `<input type="date">`     |

Tous les composants d’entrée, y `EditForm`compris, prennent en charge les attributs arbitraires. Tout attribut qui ne correspond pas à un paramètre de composant est ajouté à l’élément HTML rendu.

Les composants d’entrée fournissent le comportement par défaut pour la validation lors de la modification et la modification de leur classe CSS pour refléter l’état du champ. Certains composants incluent une logique d’analyse utile. Par exemple, `InputDate` et `InputNumber` gèrent correctement les valeurs non analysables en les inscrivant en tant qu’erreurs de validation. Les types qui peuvent accepter des valeurs NULL prennent également en charge la possibilité de valeur null du `int?`champ cible (par exemple,).

Le type `Starship` suivant définit la logique de validation à l’aide d’un plus grand ensemble de propriétés et `ExampleModel`d’annotations de données que la précédente :

```csharp
using System;
using System.ComponentModel.DataAnnotations;

public class Starship
{
    [Required]
    [StringLength(16, ErrorMessage = "Identifier too long (16 character limit).")]
    public string Identifier { get; set; }

    public string Description { get; set; }

    [Required]
    public string Classification { get; set; }

    [Range(1, 100000, ErrorMessage = "Accommodation invalid (1-100000).")]
    public int MaximumAccommodation { get; set; }

    [Required]
    [Range(typeof(bool), "true", "true", 
        ErrorMessage = "This form disallows unapproved ships.")]
    public bool IsValidatedDesign { get; set; }

    [Required]
    public DateTime ProductionDate { get; set; }
}
```

Dans l’exemple précédent, `Description` est facultatif, car aucune annotation de données n’est présente.

La forme suivante valide l’entrée d’utilisateur à l’aide de la `Starship` validation définie dans le modèle :

```razor
@page "/FormsValidation"

<h1>Starfleet Starship Database</h1>

<h2>New Ship Entry Form</h2>

<EditForm Model="@starship" OnValidSubmit="HandleValidSubmit">
    <DataAnnotationsValidator />
    <ValidationSummary />

    <p>
        <label>
            Identifier:
            <InputText @bind-Value="starship.Identifier" />
        </label>
    </p>
    <p>
        <label>
            Description (optional):
            <InputTextArea @bind-Value="starship.Description" />
        </label>
    </p>
    <p>
        <label>
            Primary Classification:
            <InputSelect @bind-Value="starship.Classification">
                <option value="">Select classification ...</option>
                <option value="Exploration">Exploration</option>
                <option value="Diplomacy">Diplomacy</option>
                <option value="Defense">Defense</option>
            </InputSelect>
        </label>
    </p>
    <p>
        <label>
            Maximum Accommodation:
            <InputNumber @bind-Value="starship.MaximumAccommodation" />
        </label>
    </p>
    <p>
        <label>
            Engineering Approval:
            <InputCheckbox @bind-Value="starship.IsValidatedDesign" />
        </label>
    </p>
    <p>
        <label>
            Production Date:
            <InputDate @bind-Value="starship.ProductionDate" />
        </label>
    </p>

    <button type="submit">Submit</button>

    <p>
        <a href="http://www.startrek.com/">Star Trek</a>, 
        &copy;1966-2019 CBS Studios, Inc. and 
        <a href="https://www.paramount.com">Paramount Pictures</a>
    </p>
</EditForm>

@code {
    private Starship starship = new Starship();

    private void HandleValidSubmit()
    {
        Console.WriteLine("OnValidSubmit");
    }
}
```

`EditForm` Crée un `EditContext` comme valeur en [cascade](xref:blazor/components#cascading-values-and-parameters) qui effectue le suivi des métadonnées relatives au processus de modification, notamment les champs qui ont été modifiés et les messages de validation actuels. Le `EditForm` fournit également des événements pratiques pour les envois valides`OnValidSubmit`et `OnInvalidSubmit`non valides (,). Vous pouvez également utiliser `OnSubmit` pour déclencher la validation et vérifier les valeurs de champ avec un code de validation personnalisé.

Dans l’exemple suivant :

* La `HandleSubmit` méthode s’exécute lorsque le bouton **Envoyer** est sélectionné.
* Le formulaire est validé à l’aide de `EditContext`la.
* Le formulaire est validé en passant `EditContext` à la `ServerValidate` méthode qui appelle un point de terminaison d’API Web sur le serveur (*non affiché*).
* Du code supplémentaire est exécuté en fonction du résultat de la validation côté client et côté serveur en vérifiant `isValid`.

```razor
<EditForm EditContext="@editContext" OnSubmit="@HandleSubmit">

    ...

    <button type="submit">Submit</button>
</EditForm>

@code {
    private Starship starship = new Starship();
    private EditContext editContext;

    protected override void OnInitialized()
    {
        editContext = new EditContext(starship);
    }

    private async Task HandleSubmit()
    {
        var isValid = editContext.Validate() && 
            await ServerValidate(editContext);

        if (isValid)
        {
            ...
        }
        else
        {
            ...
        }
    }

    private async Task<bool> ServerValidate(EditContext editContext)
    {
        var serverChecksValid = ...

        return serverChecksValid;
    }
}
```

## <a name="inputtext-based-on-the-input-event"></a>InputText en fonction de l’événement d’entrée

Utilisez le `InputText` composant pour créer un composant personnalisé qui utilise l' `input` événement au lieu de `change` l’événement.

Créez un composant avec le balisage suivant et utilisez le composant de la `InputText` même façon que :

```razor
@inherits InputText

<input 
    @attributes="AdditionalAttributes" 
    class="@CssClass" 
    value="@CurrentValue" 
    @oninput="EventCallback.Factory.CreateBinder<string>(
        this, __value => CurrentValueAsString = __value, CurrentValueAsString)" />
```

## <a name="work-with-radio-buttons"></a>Utiliser des cases d’option

Lorsque vous utilisez des cases d’option dans un formulaire, la liaison de données est gérée différemment des autres éléments, car les cases d’option sont évaluées en tant que groupe. La valeur de chaque case d’option est fixe, mais la valeur du groupe de cases d’option est la valeur de la case d’option sélectionnée. L’exemple suivant montre comment :

* Gérer la liaison de données pour un groupe de cases d’option.
* Prendre en charge la validation `InputRadio` à l’aide d’un composant personnalisé.

```razor
@using System.Globalization
@typeparam TValue
@inherits InputBase<TValue>

<input @attributes="AdditionalAttributes" type="radio" value="@SelectedValue" 
       checked="@(SelectedValue.Equals(Value))" @onchange="OnChange" />

@code {
    [Parameter]
    public TValue SelectedValue { get; set; }

    private void OnChange(ChangeEventArgs args)
    {
        CurrentValueAsString = args.Value.ToString();
    }

    protected override bool TryParseValueFromString(string value, 
        out TValue result, out string errorMessage)
    {
        var success = BindConverter.TryConvertTo<TValue>(
            value, CultureInfo.CurrentCulture, out var parsedValue);
        if (success)
        {
            result = parsedValue;
            errorMessage = null;

            return true;
        }
        else
        {
            result = default;
            errorMessage = $"{FieldIdentifier.FieldName} field isn't valid.";

            return false;
        }
    }
}
```

L’exemple `EditForm` suivant utilise le `InputRadio` composant précédent pour obtenir et valider une évaluation de l’utilisateur :

```razor
@page "/RadioButtonExample"
@using System.ComponentModel.DataAnnotations

<h1>Radio Button Group Test</h1>

<EditForm Model="model" OnValidSubmit="HandleValidSubmit">
    <DataAnnotationsValidator />
    <ValidationSummary />

    @for (int i = 1; i <= 5; i++)
    {
        <label>
            <InputRadio name="rate" SelectedValue="i" @bind-Value="model.Rating" />
            @i
        </label>
    }

    <button type="submit">Submit</button>
</EditForm>

<p>You chose: @model.Rating</p>

@code {
    private Model model = new Model();

    private void HandleValidSubmit()
    {
        Console.WriteLine("valid");
    }

    public class Model
    {
        [Range(1, 5)]
        public int Rating { get; set; }
    }
}
```

## <a name="validation-support"></a>Prise en charge de la validation

Le `DataAnnotationsValidator` composant attache la prise en charge de la validation à l’aide d' `EditContext`annotations de données à la Cascaded. L’activation de la prise en charge de la validation à l’aide d’annotations de données requiert ce geste explicite. Pour utiliser un système de validation différent des annotations de données `DataAnnotationsValidator` , remplacez par une implémentation personnalisée. L’implémentation de ASP.net Core est disponible pour l’inspection dans la source de référence : [DataAnnotationsValidator](https://github.com/dotnet/AspNetCore/blob/master/src/Components/Forms/src/DataAnnotationsValidator.cs)/[AddDataAnnotationsValidation](https://github.com/dotnet/AspNetCore/blob/master/src/Components/Forms/src/EditContextDataAnnotationsExtensions.cs).

Blazoreffectue deux types de validation :

* La *validation de champ* est effectuée lorsque l’utilisateur quitte un champ. Pendant la validation de champ `DataAnnotationsValidator` , le composant associe tous les résultats de validation signalés au champ.
* La *validation du modèle* est effectuée lorsque l’utilisateur envoie le formulaire. Lors de la validation du `DataAnnotationsValidator` modèle, le composant tente de déterminer le champ en fonction du nom du membre que le résultat de validation signale. Les résultats de validation qui ne sont pas associés à un membre individuel sont associés au modèle plutôt qu’à un champ.

### <a name="validation-summary-and-validation-message-components"></a>Résumé des validations et composants de message de validation

Le `ValidationSummary` composant résume tous les messages de validation, ce qui est similaire au [tag Helper de résumé de validation](xref:mvc/views/working-with-forms#the-validation-summary-tag-helper):

```razor
<ValidationSummary />
```

Messages de validation de sortie pour un modèle spécifique `Model` avec le paramètre :
  
```razor
<ValidationSummary Model="@starship" />
```

Le `ValidationMessage` composant affiche des messages de validation pour un champ spécifique, qui est similaire au [tag Helper de message de validation](xref:mvc/views/working-with-forms#the-validation-message-tag-helper). Spécifiez le champ pour la validation `For` avec l’attribut et une expression lambda qui nomme la propriété de modèle :

```razor
<ValidationMessage For="@(() => starship.MaximumAccommodation)" />
```

Les `ValidationMessage` composants `ValidationSummary` et prennent en charge des attributs arbitraires. Tout attribut qui ne correspond pas à un paramètre de composant est ajouté `<div>` à `<ul>` l’élément ou généré.

### <a name="custom-validation-attributes"></a>Attributs de validation personnalisés

Pour vous assurer qu’un résultat de validation est correctement associé à un champ lors de l’utilisation d’un [attribut de validation personnalisé](xref:mvc/models/validation#custom-attributes), transmettez le contexte de validation lors de <xref:System.ComponentModel.DataAnnotations.ValidationContext.MemberName> la création du <xref:System.ComponentModel.DataAnnotations.ValidationResult>:

```csharp
using System;
using System.ComponentModel.DataAnnotations;

private class MyCustomValidator : ValidationAttribute
{
    protected override ValidationResult IsValid(object value, 
        ValidationContext validationContext)
    {
        ...

        return new ValidationResult("Validation message to user.",
            new[] { validationContext.MemberName });
    }
}
```

### <a name="blazor-data-annotations-validation-package"></a>Blazorpackage de validation des annotations de données

[Microsoft. AspNetCore. Components. DataAnnotations. validation](https://www.nuget.org/packages/Microsoft.AspNetCore.Components.DataAnnotations.Validation) est un package qui remplit les intervalles d’expérience de validation `DataAnnotationsValidator` à l’aide du composant. Le package est actuellement *expérimental*.

### <a name="compareproperty-attribute"></a>Attribut [CompareProperty]

Le <xref:System.ComponentModel.DataAnnotations.CompareAttribute> ne fonctionne pas correctement avec `DataAnnotationsValidator` le composant, car il n’associe pas le résultat de la validation à un membre spécifique. Cela peut entraîner un comportement incohérent entre la validation au niveau du champ et le moment où la totalité du modèle est validée sur une soumission. Le package *expérimental* [Microsoft. AspNetCore. Components. DataAnnotations. validation](https://www.nuget.org/packages/Microsoft.AspNetCore.Components.DataAnnotations.Validation) introduit un attribut de `ComparePropertyAttribute`validation supplémentaire,, qui contourne ces limitations. Dans une Blazor application, `[CompareProperty]` est un remplacement direct de l' `[Compare]` attribut.

### <a name="nested-models-collection-types-and-complex-types"></a>Modèles imbriqués, types de collection et types complexes

Blazorprend en charge la validation de l’entrée de formulaire à l’aide d’annotations de données avec le intégré `DataAnnotationsValidator`. Toutefois, l `DataAnnotationsValidator` 'valide uniquement les propriétés de niveau supérieur du modèle lié au formulaire qui ne sont pas des propriétés de type collection ou complexe.

Pour valider le graphique d’objets entier du modèle lié, y compris les propriétés de type collection et de type `ObjectGraphDataAnnotationsValidator` complexe, utilisez le fourni par le package *expérimental* [Microsoft. AspNetCore. Components. DataAnnotations. validation](https://www.nuget.org/packages/Microsoft.AspNetCore.Components.DataAnnotations.Validation) :

```razor
<EditForm Model="@model" OnValidSubmit="HandleValidSubmit">
    <ObjectGraphDataAnnotationsValidator />
    ...
</EditForm>
```

Annotez les propriétés `[ValidateComplexType]`de modèle avec. Dans les classes de modèle suivantes, `ShipDescription` la classe contient des annotations de données supplémentaires à valider lorsque le modèle est lié au formulaire :

*Starship.cs*:

```csharp
using System;
using System.ComponentModel.DataAnnotations;

public class Starship
{
    ...

    [ValidateComplexType]
    public ShipDescription ShipDescription { get; set; }

    ...
}
```

*ShipDescription.cs*:

```csharp
using System;
using System.ComponentModel.DataAnnotations;

public class ShipDescription
{
    [Required]
    [StringLength(40, ErrorMessage = "Description too long (40 char).")]
    public string ShortDescription { get; set; }
    
    [Required]
    [StringLength(240, ErrorMessage = "Description too long (240 char).")]
    public string LongDescription { get; set; }
}
```

### <a name="enable-the-submit-button-based-on-form-validation"></a>Activer le bouton Envoyer en fonction de la validation de formulaire

Pour activer et désactiver le bouton Envoyer en fonction de la validation de formulaire :

* Utilisez le du `EditContext` formulaire pour assigner le modèle lorsque le composant est initialisé.
* Validez le formulaire dans le rappel `OnFieldChanged` du contexte pour activer et désactiver le bouton Envoyer.
* Décrochez le gestionnaire d’événements dans `Dispose` la méthode. Pour plus d'informations, consultez <xref:blazor/lifecycle#component-disposal-with-idisposable>.

```razor
@implements IDisposable

<EditForm EditContext="@editContext">
    <DataAnnotationsValidator />
    <ValidationSummary />

    ...

    <button type="submit" disabled="@formInvalid">Submit</button>
</EditForm>

@code {
    private Starship starship = new Starship();
    private bool formInvalid = true;
    private EditContext editContext;

    protected override void OnInitialized()
    {
        editContext = new EditContext(starship);
        editContext.OnFieldChanged += HandleFieldChanged;
    }

    private void HandleFieldChanged(object sender, FieldChangedEventArgs e)
    {
        formInvalid = !editContext.Validate();
        StateHasChanged();
    }

    public void Dispose()
    {
        editContext.OnFieldChanged -= HandleFieldChanged;
    }
}
```

Dans l’exemple précédent, `formInvalid` affectez `false` à la valeur si :

* Le formulaire est préchargé avec des valeurs par défaut valides.
* Vous voulez que le bouton envoyer soit activé lors du chargement du formulaire.

L’un des effets secondaires de l’approche précédente est `ValidationSummary` qu’un composant est rempli avec des champs non valides une fois que l’utilisateur interagit avec un champ quelconque. Ce scénario peut être traité de l’une des manières suivantes :

* N’utilisez pas `ValidationSummary` un composant sur le formulaire.
* Rendre le `ValidationSummary` composant visible lorsque le bouton Envoyer est sélectionné (par exemple, dans une `HandleValidSubmit` méthode).

```razor
<EditForm EditContext="@editContext" OnValidSubmit="HandleValidSubmit">
    <DataAnnotationsValidator />
    <ValidationSummary style="@displaySummary" />

    ...

    <button type="submit" disabled="@formInvalid">Submit</button>
</EditForm>

@code {
    private string displaySummary = "display:none";

    ...

    private void HandleValidSubmit()
    {
        displaySummary = "display:block";
    }
}
```
