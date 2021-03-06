---
title: Intégrer des composants ASP.NET Core Razor dans des applications Razor Pages et MVC
author: guardrex
description: En savoir plus sur les scénarios de liaison de données pour les composants et les éléments DOM dans Blazor applications.
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 03/17/2020
no-loc:
- Blazor
- SignalR
uid: blazor/integrate-components
ms.openlocfilehash: cf6056e0985d5433bddecac8dd183ca3f4c2af5b
ms.sourcegitcommit: 91dc1dd3d055b4c7d7298420927b3fd161067c64
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 03/24/2020
ms.locfileid: "80218932"
---
# <a name="integrate-aspnet-core-razor-components-into-razor-pages-and-mvc-apps"></a>Intégrer des composants ASP.NET Core Razor dans des applications Razor Pages et MVC

Par [Luke Latham](https://github.com/guardrex) et [Daniel Roth](https://github.com/danroth27)

Les composants Razor peuvent être intégrés dans des applications Razor Pages et MVC. Lorsque la page ou la vue est restituée, les composants peuvent être prérendus en même temps.

## <a name="prepare-the-app-to-use-components-in-pages-and-views"></a>Préparer l’application pour utiliser des composants dans les pages et les vues

Une Razor Pages ou une application MVC existante peut intégrer des composants Razor dans des pages et des vues :

1. Dans le fichier de disposition de l’application ( *_Layout. cshtml*) :

   * Ajoutez la balise `<base>` suivante à l’élément `<head>` :

     ```html
     <base href="~/" />
     ```

     La valeur `href` (le *chemin d’accès de base*de l’application) dans l’exemple précédent suppose que l’application se trouve dans le chemin d’URL racine (`/`). Si l’application est une sous-application, suivez les instructions de la section *chemin d’accès* à la base de l’application de l’article <xref:host-and-deploy/blazor/index#app-base-path>.

     Le fichier *_Layout. cshtml* se trouve dans le dossier *pages/Shared* d’une application Razor pages ou d’un dossier *Views/Shared* dans une application MVC.

   * Ajoutez une balise `<script>` pour le script *éblouissant. Server. js* immédiatement avant la balise de fermeture `</body>` :

     ```html
     <script src="_framework/blazor.server.js"></script>
     ```

     L’infrastructure ajoute le script *éblouissant. Server. js* à l’application. Il n’est pas nécessaire d’ajouter manuellement le script à l’application.

1. Ajoutez un fichier *_Imports. Razor* au dossier racine du projet avec le contenu suivant (modifiez le dernier espace de noms, `MyAppNamespace`, en lui attribuant l’espace de noms de l’application) :

   ```razor
   @using System.Net.Http
   @using Microsoft.AspNetCore.Authorization
   @using Microsoft.AspNetCore.Components.Authorization
   @using Microsoft.AspNetCore.Components.Forms
   @using Microsoft.AspNetCore.Components.Routing
   @using Microsoft.AspNetCore.Components.Web
   @using Microsoft.JSInterop
   @using MyAppNamespace
   ```

1. Dans `Startup.ConfigureServices`, inscrivez le service Blazor Server :

   ```csharp
   services.AddServerSideBlazor();
   ```

1. Dans `Startup.Configure`, ajoutez le point de terminaison Blazor Hub à `app.UseEndpoints`:

   ```csharp
   endpoints.MapBlazorHub();
   ```

1. Intégrer des composants dans n’importe quelle page ou vue. Pour plus d’informations, consultez la section [rendre les composants à partir d’une page ou d’une vue](#render-components-from-a-page-or-view) .

## <a name="use-routable-components-in-a-razor-pages-app"></a>Utiliser des composants routables dans une application Razor Pages

*Cette section concerne l’ajout de composants qui sont directement routables à partir des demandes des utilisateurs.*

Pour prendre en charge les composants Razor routables dans les applications Razor Pages :

1. Suivez les instructions de la section [préparer l’application à utiliser les composants des pages et des vues](#prepare-the-app-to-use-components-in-pages-and-views) .

1. Ajoutez un fichier *app. Razor* à la racine du projet avec le contenu suivant :

   ```razor
   @using Microsoft.AspNetCore.Components.Routing

   <Router AppAssembly="typeof(Program).Assembly">
       <Found Context="routeData">
           <RouteView RouteData="routeData" />
       </Found>
       <NotFound>
           <h1>Page not found</h1>
           <p>Sorry, but there's nothing here!</p>
       </NotFound>
   </Router>
   ```

1. Ajoutez un fichier *_Host. cshtml* au dossier *pages* avec le contenu suivant :

   ```cshtml
   @page "/blazor"
   @{
       Layout = "_Layout";
   }

   <app>
       <component type="typeof(App)" render-mode="ServerPrerendered" />
   </app>
   ```

   Les composants utilisent le fichier *_Layout. cshtml* partagé pour leur disposition.

1. Ajoutez un itinéraire de priorité basse pour la page *_Host. cshtml* à la configuration du point de terminaison dans `Startup.Configure`:

   ```csharp
   app.UseEndpoints(endpoints =>
   {
       ...

       endpoints.MapFallbackToPage("/_Host");
   });
   ```

1. Ajoutez des composants routables à l’application. Par exemple :

   ```razor
   @page "/counter"

   <h1>Counter</h1>

   ...
   ```

   Pour plus d’informations sur les espaces de noms, consultez la section [espaces de noms de composants](#component-namespaces) .

## <a name="use-routable-components-in-an-mvc-app"></a>Utiliser des composants routables dans une application MVC

*Cette section concerne l’ajout de composants qui sont directement routables à partir des demandes des utilisateurs.*

Pour prendre en charge les composants Razor routables dans les applications MVC :

1. Suivez les instructions de la section [préparer l’application à utiliser les composants des pages et des vues](#prepare-the-app-to-use-components-in-pages-and-views) .

1. Ajoutez un fichier *app. Razor* à la racine du projet avec le contenu suivant :

   ```razor
   @using Microsoft.AspNetCore.Components.Routing

   <Router AppAssembly="typeof(Program).Assembly">
       <Found Context="routeData">
           <RouteView RouteData="routeData" />
       </Found>
       <NotFound>
           <h1>Page not found</h1>
           <p>Sorry, but there's nothing here!</p>
       </NotFound>
   </Router>
   ```

1. Ajoutez un fichier *_Host. cshtml* au dossier *views/de démarrage* avec le contenu suivant :

   ```cshtml
   @{
       Layout = "_Layout";
   }

   <app>
       <component type="typeof(App)" render-mode="ServerPrerendered" />
   </app>
   ```

   Les composants utilisent le fichier *_Layout. cshtml* partagé pour leur disposition.

1. Ajoutez une action au contrôleur d’hébergement :

   ```csharp
   public IActionResult Blazor()
   {
      return View("_Host");
   }
   ```

1. Ajoutez un itinéraire de faible priorité pour l’action de contrôleur qui retourne la vue *_Host. cshtml* à la configuration du point de terminaison dans `Startup.Configure`:

   ```csharp
   app.UseEndpoints(endpoints =>
   {
       ...

       endpoints.MapFallbackToController("Blazor", "Home");
   });
   ```

1. Créez un dossier *pages* et ajoutez des composants routables à l’application. Par exemple :

   ```razor
   @page "/counter"

   <h1>Counter</h1>

   ...
   ```

   Pour plus d’informations sur les espaces de noms, consultez la section [espaces de noms de composants](#component-namespaces) .

## <a name="component-namespaces"></a>Espaces de noms de composants

Lorsque vous utilisez un dossier personnalisé pour stocker les composants de l’application, ajoutez l’espace de noms qui représente le dossier à la page/la vue ou au fichier *_ViewImports. cshtml* . Dans l’exemple suivant :

* Remplacez `MyAppNamespace` par l’espace de noms de l’application.
* Si un dossier nommé *Components* n’est pas utilisé pour contenir les composants, remplacez `Components` par le dossier dans lequel se trouvent les composants.

```cshtml
@using MyAppNamespace.Components
```

Le fichier *_ViewImports. cshtml* se trouve dans le dossier *pages* d’une application Razor pages ou du dossier *views* d’une application MVC.

Pour plus d’informations, consultez <xref:blazor/components#import-components>.

## <a name="render-components-from-a-page-or-view"></a>Rendre les composants à partir d’une page ou d’une vue

*Cette section se rapporte à l’ajout de composants à des pages ou à des vues, où les composants ne sont pas directement routés à partir des demandes de l’utilisateur.*

Pour afficher un composant à partir d’une page ou d’une vue, utilisez le [tag Helper Component](xref:mvc/views/tag-helpers/builtin-th/component-tag-helper).

Pour plus d’informations sur la façon dont les composants sont rendus, l’état des composants et le tag Helper `Component`, consultez les articles suivants :

* <xref:blazor/hosting-models>
* <xref:blazor/hosting-model-configuration>
* <xref:mvc/views/tag-helpers/builtin-th/component-tag-helper>
