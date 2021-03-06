---
title: ASP.NET Core Blazor cycle de vie
author: guardrex
description: Découvrez comment utiliser les méthodes de cycle de vie des composants Razor dans ASP.NET Core applications Blazor.
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 03/17/2020
no-loc:
- Blazor
- SignalR
uid: blazor/lifecycle
ms.openlocfilehash: 831f575afa6ce11d06c016d43ecd1bb59d09eab6
ms.sourcegitcommit: 91dc1dd3d055b4c7d7298420927b3fd161067c64
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 03/24/2020
ms.locfileid: "80218906"
---
# <a name="aspnet-core-opno-locblazor-lifecycle"></a>ASP.NET Core Blazor cycle de vie

Par [Luke Latham](https://github.com/guardrex) et [Daniel Roth](https://github.com/danroth27)

L’infrastructure Blazor comprend des méthodes de cycle de vie synchrones et asynchrones. Substituez les méthodes de cycle de vie pour effectuer des opérations supplémentaires sur les composants lors de l’initialisation et du rendu des composants.

## <a name="lifecycle-methods"></a>Méthodes de cycle de vie

### <a name="component-initialization-methods"></a>Méthodes d’initialisation de composant

<xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync*> et <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitialized*> sont appelés lorsque le composant est initialisé après avoir reçu ses paramètres initiaux de son composant parent. Utilisez `OnInitializedAsync` lorsque le composant exécute une opération asynchrone et doit être actualisé à la fin de l’opération.

Pour une opération synchrone, remplacez `OnInitialized`:

```csharp
protected override void OnInitialized()
{
    ...
}
```

Pour effectuer une opération asynchrone, substituez `OnInitializedAsync` et utilisez le mot clé `await` sur l’opération :

```csharp
protected override async Task OnInitializedAsync()
{
    await ...
}
```

Blazor les applications serveur qui [préaffichent](xref:blazor/hosting-model-configuration#render-mode) l' `OnInitializedAsync` de l’appel de contenu à **_deux reprises_** :

* Une fois lorsque le composant est initialement restitué de manière statique dans le cadre de la page.
* Une deuxième fois lorsque le navigateur établit une connexion au serveur.

Pour empêcher l’exécution à deux reprises du code du développeur dans `OnInitializedAsync`, consultez la section [reconnexion avec état après le prérendu](#stateful-reconnection-after-prerendering) .

Bien qu’une application Blazor Server soit prérestitué, certaines actions, telles que l’appel en JavaScript, ne sont pas possibles, car une connexion avec le navigateur n’a pas été établie. Les composants peuvent avoir besoin d’être restitués différemment lorsqu’ils sont prérendus. Pour plus d’informations, consultez la section [détecter quand l’application est un prérendu](#detect-when-the-app-is-prerendering) .

Si des gestionnaires d’événements sont configurés, décrochez-les lors de la suppression. Pour plus d’informations, consultez la section [Suppression de composant avec IDisposable](#component-disposal-with-idisposable) .

### <a name="before-parameters-are-set"></a>Les paramètres Before sont définis

<xref:Microsoft.AspNetCore.Components.ComponentBase.SetParametersAsync*> définit les paramètres fournis par le parent du composant dans l’arborescence de rendu :

```csharp
public override async Task SetParametersAsync(ParameterView parameters)
{
    await ...

    await base.SetParametersAsync(parameters);
}
```

<xref:Microsoft.AspNetCore.Components.ParameterView> contient le jeu entier de valeurs de paramètre chaque fois que `SetParametersAsync` est appelé.

L’implémentation par défaut de `SetParametersAsync` définit la valeur de chaque propriété avec l’attribut `[Parameter]` ou `[CascadingParameter]` qui a une valeur correspondante dans le `ParameterView`. Les paramètres qui n’ont pas de valeur correspondante dans `ParameterView` restent inchangés.

Si `base.SetParametersAync` n’est pas appelé, le code personnalisé peut interpréter la valeur des paramètres entrants de la façon requise. Par exemple, il n’est pas nécessaire d’assigner les paramètres entrants aux propriétés de la classe.

Si des gestionnaires d’événements sont configurés, décrochez-les lors de la suppression. Pour plus d’informations, consultez la section [Suppression de composant avec IDisposable](#component-disposal-with-idisposable) .

### <a name="after-parameters-are-set"></a>Une fois les paramètres définis

<xref:Microsoft.AspNetCore.Components.ComponentBase.OnParametersSetAsync*> et <xref:Microsoft.AspNetCore.Components.ComponentBase.OnParametersSet*> sont appelées :

* Lorsque le composant est initialisé et a reçu son premier jeu de paramètres de son composant parent.
* Lorsque le composant parent est à nouveau rendu et fournit :
  * Seuls les types immuables primitifs connus dont au moins un paramètre a été modifié.
  * Tous les paramètres de type complexe. L’infrastructure ne peut pas savoir si les valeurs d’un paramètre de type complexe ont muté en interne, donc elle traite le jeu de paramètres comme étant modifié.

```csharp
protected override async Task OnParametersSetAsync()
{
    await ...
}
```

> [!NOTE]
> Un travail asynchrone lors de l’application de paramètres et de valeurs de propriété doit se produire pendant l’événement de cycle de vie `OnParametersSetAsync`.

```csharp
protected override void OnParametersSet()
{
    ...
}
```

Si des gestionnaires d’événements sont configurés, décrochez-les lors de la suppression. Pour plus d’informations, consultez la section [Suppression de composant avec IDisposable](#component-disposal-with-idisposable) .

### <a name="after-component-render"></a>Après le rendu du composant

<xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRenderAsync*> et <xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRender*> sont appelées après la fin du rendu d’un composant. Les références d’élément et de composant sont remplies à ce stade. Utilisez cette étape pour effectuer des étapes d’initialisation supplémentaires à l’aide du contenu rendu, par exemple l’activation de bibliothèques JavaScript tierces qui opèrent sur les éléments DOM rendus.

Le paramètre `firstRender` pour `OnAfterRenderAsync` et `OnAfterRender`:

* A la valeur `true` la première fois que l’instance de composant est restituée.
* Peut être utilisé pour s’assurer que le travail d’initialisation n’est effectué qu’une seule fois.

```csharp
protected override async Task OnAfterRenderAsync(bool firstRender)
{
    if (firstRender)
    {
        await ...
    }
}
```

> [!NOTE]
> Le travail asynchrone immédiatement après le rendu doit se produire pendant l’événement de cycle de vie `OnAfterRenderAsync`.
>
> Même si vous retournez un <xref:System.Threading.Tasks.Task> à partir d' `OnAfterRenderAsync`, l’infrastructure ne planifie pas un cycle de rendu supplémentaire pour votre composant une fois cette tâche terminée. Cela permet d’éviter une boucle de rendu infinie. Elle est différente des autres méthodes de cycle de vie, qui planifient un cycle de rendu supplémentaire une fois que la tâche retournée est terminée.

```csharp
protected override void OnAfterRender(bool firstRender)
{
    if (firstRender)
    {
        ...
    }
}
```

`OnAfterRender` et `OnAfterRenderAsync` *ne sont pas appelées lors du prérendu sur le serveur.*

Si des gestionnaires d’événements sont configurés, décrochez-les lors de la suppression. Pour plus d’informations, consultez la section [Suppression de composant avec IDisposable](#component-disposal-with-idisposable) .

### <a name="suppress-ui-refreshing"></a>Supprimer l’actualisation de l’interface utilisateur

Substituez <xref:Microsoft.AspNetCore.Components.ComponentBase.ShouldRender*> pour supprimer l’actualisation de l’interface utilisateur. Si l’implémentation retourne `true`, l’interface utilisateur est actualisée :

```csharp
protected override bool ShouldRender()
{
    var renderUI = true;

    return renderUI;
}
```

`ShouldRender` est appelée chaque fois que le composant est restitué.

Même si `ShouldRender` est substitué, le composant est toujours restitué initialement.

## <a name="state-changes"></a>Changements d'état

<xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged*> notifie le composant que son état a changé. Le cas échéant, l’appel de `StateHasChanged` entraîne le rerendu du composant.

## <a name="handle-incomplete-async-actions-at-render"></a>Gérer les actions asynchrones incomplètes au rendu

Les actions asynchrones exécutées dans des événements de cycle de vie peuvent ne pas être terminées avant le rendu du composant. Les objets peuvent être `null` ou remplis de façon incomplète avec des données pendant l’exécution de la méthode Lifecycle. Fournissez une logique de rendu pour confirmer que les objets sont initialisés. Affichez les éléments d’interface utilisateur d’espace réservé (par exemple, un message de chargement) pendant que les objets sont `null`.

Dans le composant `FetchData` des modèles Blazor, `OnInitializedAsync` est remplacé par asynchrone recevoir les données de prévision (`forecasts`). Lorsque `forecasts` est `null`, un message de chargement s’affiche pour l’utilisateur. Une fois la `Task` retournée par `OnInitializedAsync` terminée, le composant est rerendu avec l’État mis à jour.

*Pages/fetchData. Razor* dans le modèle Blazor Server :

[!code-razor[](lifecycle/samples_snapshot/3.x/FetchData.razor?highlight=9,21,25)]

## <a name="component-disposal-with-idisposable"></a>Suppression de composant avec IDisposable

Si un composant implémente <xref:System.IDisposable>, la [méthode dispose](/dotnet/standard/garbage-collection/implementing-dispose) est appelée lorsque le composant est supprimé de l’interface utilisateur. Le composant suivant utilise `@implements IDisposable` et la méthode `Dispose` :

```razor
@using System
@implements IDisposable

...

@code {
    public void Dispose()
    {
        ...
    }
}
```

> [!NOTE]
> L’appel de <xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged*> dans `Dispose` n’est pas pris en charge. `StateHasChanged` peut être appelé dans le cadre de la suppression du convertisseur, il n’est donc pas possible de demander des mises à jour de l’interface utilisateur à ce stade.

Annule l’abonnement des gestionnaires d’événements des événements .NET. Les exemples de [formulaire deBlazor](xref:blazor/forms-validation) suivants montrent comment décrocher un gestionnaire d’événements dans la méthode `Dispose` :

* Approche de champ privé et lambda

  [!code-razor[](lifecycle/samples_snapshot/3.x/event-handler-disposal-1.razor?highlight=23,28)]

* Approche de la méthode privée

  [!code-razor[](lifecycle/samples_snapshot/3.x/event-handler-disposal-2.razor?highlight=16,26)]

## <a name="handle-errors"></a>des erreurs

Pour plus d’informations sur la gestion des erreurs lors de l’exécution de la méthode de cycle de vie, consultez <xref:blazor/handle-errors#lifecycle-methods>.

## <a name="stateful-reconnection-after-prerendering"></a>Reconnexion avec état après le prérendu

Dans une application Blazor Server lorsque `RenderMode` est `ServerPrerendered`, le composant est initialement restitué de manière statique dans le cadre de la page. Une fois que le navigateur a établi une connexion au serveur, le composant est *à nouveau*rendu et le composant est maintenant interactif. Si la méthode de cycle de vie [OnInitialized {Async}](xref:blazor/lifecycle#component-initialization-methods) pour initialiser le composant est présente, la méthode est exécutée *deux fois*:

* Lorsque le composant est prérendu statiquement.
* Après l’établissement de la connexion au serveur.

Cela peut entraîner une modification notable des données affichées dans l’interface utilisateur lorsque le composant est finalement restitué.

Pour éviter le double rendu du scénario dans une application Blazor Server :

* Transmettez un identificateur qui peut être utilisé pour mettre en cache l’État pendant le prérendu et pour récupérer l’état après le redémarrage de l’application.
* Utilisez l’identificateur pendant le prérendu pour enregistrer l’état du composant.
* Utilisez l’identificateur après le prérendu pour récupérer l’État mis en cache.

Le code suivant illustre une `WeatherForecastService` mise à jour dans une application de serveur Blazor basée sur un modèle qui évite le double rendu :

```csharp
public class WeatherForecastService
{
    private static readonly string[] _summaries = new[]
    {
        "Freezing", "Bracing", "Chilly", "Cool", "Mild",
        "Warm", "Balmy", "Hot", "Sweltering", "Scorching"
    };
    
    public WeatherForecastService(IMemoryCache memoryCache)
    {
        MemoryCache = memoryCache;
    }
    
    public IMemoryCache MemoryCache { get; }

    public Task<WeatherForecast[]> GetForecastAsync(DateTime startDate)
    {
        return MemoryCache.GetOrCreateAsync(startDate, async e =>
        {
            e.SetOptions(new MemoryCacheEntryOptions
            {
                AbsoluteExpirationRelativeToNow = 
                    TimeSpan.FromSeconds(30)
            });

            var rng = new Random();

            await Task.Delay(TimeSpan.FromSeconds(10));

            return Enumerable.Range(1, 5).Select(index => new WeatherForecast
            {
                Date = startDate.AddDays(index),
                TemperatureC = rng.Next(-20, 55),
                Summary = _summaries[rng.Next(_summaries.Length)]
            }).ToArray();
        });
    }
}
```

Pour plus d’informations sur la `RenderMode`, consultez <xref:blazor/hosting-model-configuration#render-mode>.

## <a name="detect-when-the-app-is-prerendering"></a>Détecter quand l’application est prérendu

[!INCLUDE[](~/includes/blazor-prerendering.md)]
