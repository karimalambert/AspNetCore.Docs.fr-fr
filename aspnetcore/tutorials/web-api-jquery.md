---
title: 'Tutoriel : Appeler une API web avec jQuery à l’aide d’ASP.NET Core'
author: rick-anderson
description: Découvrez comment appeler une API web ASP.NET Core avec jQuery.
ms.author: riande
ms.custom: mvc
ms.date: 07/20/2019
uid: tutorials/web-api-jquery
ms.openlocfilehash: eb8b2453fd037170a49f531fea4c3ef1c056292d
ms.sourcegitcommit: 0efb9e219fef481dee35f7b763165e488aa6cf9c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/29/2019
ms.locfileid: "68602568"
---
# <a name="tutorial-call-an-aspnet-core-web-api-with-jquery"></a><span data-ttu-id="f9c1c-103">Tutoriel : Appeler une API web ASP.NET Core avec jQuery</span><span class="sxs-lookup"><span data-stu-id="f9c1c-103">Tutorial: Call an ASP.NET Core web API with jQuery</span></span>

<span data-ttu-id="f9c1c-104">Par [Rick Anderson](https://twitter.com/RickAndMSFT)</span><span class="sxs-lookup"><span data-stu-id="f9c1c-104">By [Rick Anderson](https://twitter.com/RickAndMSFT)</span></span>

<span data-ttu-id="f9c1c-105">Ce tutoriel montre comment appeler une API web ASP.NET Core avec jQuery</span><span class="sxs-lookup"><span data-stu-id="f9c1c-105">This tutorial shows how to call an ASP.NET Core web API with jQuery</span></span>

::: moniker range="< aspnetcore-3.0"

<span data-ttu-id="f9c1c-106">Pour ASP.NET Core 2.2, consultez la version 2.2 de la section [Appeler l’API web avec jQuery](xref:tutorials/first-web-api#call-the-api-with-jquery).</span><span class="sxs-lookup"><span data-stu-id="f9c1c-106">For ASP.NET Core 2.2, see the 2.2 version of [Call the Web API with jQuery](xref:tutorials/first-web-api#call-the-api-with-jquery).</span></span>

::: moniker-end

::: moniker range=">= aspnetcore-3.0"

## <a name="prerequisites"></a><span data-ttu-id="f9c1c-107">Prérequis</span><span class="sxs-lookup"><span data-stu-id="f9c1c-107">Prerequisites</span></span>

<span data-ttu-id="f9c1c-108">Suivre le [Tutoriel : Créer une API web](xref:tutorials/first-web-api)</span><span class="sxs-lookup"><span data-stu-id="f9c1c-108">Complete [Tutorial: Create a web API](xref:tutorials/first-web-api)</span></span>

## <a name="call-the-api-with-jquery"></a><span data-ttu-id="f9c1c-109">Appeler l’API avec jQuery</span><span class="sxs-lookup"><span data-stu-id="f9c1c-109">Call the API with jQuery</span></span>

<span data-ttu-id="f9c1c-110">Dans cette section, une page HTML qui utilise jQuery pour appeler l’API web est ajoutée.</span><span class="sxs-lookup"><span data-stu-id="f9c1c-110">In this section, an HTML page is added that uses jQuery to call the web api.</span></span> <span data-ttu-id="f9c1c-111">jQuery lance la requête et met à jour la page avec les détails de la réponse de l’API.</span><span class="sxs-lookup"><span data-stu-id="f9c1c-111">jQuery initiates the request and updates the page with the details from the API's response.</span></span>

<span data-ttu-id="f9c1c-112">Configurez l’application pour [traiter les fichiers statiques](/dotnet/api/microsoft.aspnetcore.builder.staticfileextensions.usestaticfiles#Microsoft_AspNetCore_Builder_StaticFileExtensions_UseStaticFiles_Microsoft_AspNetCore_Builder_IApplicationBuilder_) et [activer le mappage de fichier par défaut](/dotnet/api/microsoft.aspnetcore.builder.defaultfilesextensions.usedefaultfiles#Microsoft_AspNetCore_Builder_DefaultFilesExtensions_UseDefaultFiles_Microsoft_AspNetCore_Builder_IApplicationBuilder_) en mettant à jour *Startup.cs* avec le code en surbrillance suivant :</span><span class="sxs-lookup"><span data-stu-id="f9c1c-112">Configure the app to [serve static files](/dotnet/api/microsoft.aspnetcore.builder.staticfileextensions.usestaticfiles#Microsoft_AspNetCore_Builder_StaticFileExtensions_UseStaticFiles_Microsoft_AspNetCore_Builder_IApplicationBuilder_) and [enable default file mapping](/dotnet/api/microsoft.aspnetcore.builder.defaultfilesextensions.usedefaultfiles#Microsoft_AspNetCore_Builder_DefaultFilesExtensions_UseDefaultFiles_Microsoft_AspNetCore_Builder_IApplicationBuilder_) by updating *Startup.cs* with the following highlighted code:</span></span>

[!code-csharp[](first-web-api/samples/3.0/TodoApi/StartupJquery.cs?highlight=8-9&name=snippet_configure)]

<span data-ttu-id="f9c1c-113">Créez un dossier *wwwroot* dans le répertoire du projet.</span><span class="sxs-lookup"><span data-stu-id="f9c1c-113">Create a *wwwroot* folder in the project directory.</span></span>

<span data-ttu-id="f9c1c-114">Ajoutez un fichier HTML nommé *index.html* au répertoire *wwwroot*.</span><span class="sxs-lookup"><span data-stu-id="f9c1c-114">Add an HTML file named *index.html* to the *wwwroot* directory.</span></span> <span data-ttu-id="f9c1c-115">Remplacez son contenu par le balisage suivant :</span><span class="sxs-lookup"><span data-stu-id="f9c1c-115">Replace its contents with the following markup:</span></span>

[!code-html[](first-web-api/samples/3.0/TodoApi/wwwroot/index.html)]

<span data-ttu-id="f9c1c-116">Ajoutez un fichier JavaScript nommé *site.js* au répertoire *wwwroot*.</span><span class="sxs-lookup"><span data-stu-id="f9c1c-116">Add a JavaScript file named *site.js* to the *wwwroot* directory.</span></span> <span data-ttu-id="f9c1c-117">Remplacez son contenu par le code suivant :</span><span class="sxs-lookup"><span data-stu-id="f9c1c-117">Replace its contents with the following code:</span></span>

[!code-javascript[](first-web-api/samples/3.0/TodoApi/wwwroot/site.js?name=snippet_SiteJs)]

<span data-ttu-id="f9c1c-118">Vous devrez peut-être changer les paramètres de lancement du projet ASP.NET Core pour tester la page HTML localement :</span><span class="sxs-lookup"><span data-stu-id="f9c1c-118">A change to the ASP.NET Core project's launch settings may be required to test the HTML page locally:</span></span>

* <span data-ttu-id="f9c1c-119">Ouvrez *Properties\launchSettings.json*.</span><span class="sxs-lookup"><span data-stu-id="f9c1c-119">Open *Properties\launchSettings.json*.</span></span>
* <span data-ttu-id="f9c1c-120">Supprimez la propriété `launchUrl` pour forcer l’utilisation du fichier *index.html* (fichier par défaut du projet) à l’ouverture de l’application.</span><span class="sxs-lookup"><span data-stu-id="f9c1c-120">Remove the `launchUrl` property to force the app to open at *index.html*&mdash;the project's default file.</span></span>

<span data-ttu-id="f9c1c-121">Il existe plusieurs façons d’obtenir jQuery.</span><span class="sxs-lookup"><span data-stu-id="f9c1c-121">There are several ways to get jQuery.</span></span> <span data-ttu-id="f9c1c-122">Dans l’extrait précédent, la bibliothèque est chargée à partir d’un CDN.</span><span class="sxs-lookup"><span data-stu-id="f9c1c-122">In the preceding snippet, the library is loaded from a CDN.</span></span>

<span data-ttu-id="f9c1c-123">Cet exemple appelle toutes les méthodes CRUD de l’API.</span><span class="sxs-lookup"><span data-stu-id="f9c1c-123">This sample calls all of the CRUD methods of the API.</span></span> <span data-ttu-id="f9c1c-124">Les explications suivantes traitent des appels à l’API.</span><span class="sxs-lookup"><span data-stu-id="f9c1c-124">Following are explanations of the calls to the API.</span></span>

### <a name="get-a-list-of-to-do-items"></a><span data-ttu-id="f9c1c-125">Obtenir une liste de tâches</span><span class="sxs-lookup"><span data-stu-id="f9c1c-125">Get a list of to-do items</span></span>

<span data-ttu-id="f9c1c-126">La fonction JQuery [ajax](https://api.jquery.com/jquery.ajax/) envoie une requête `GET` à l’API, qui retourne du code JSON représentant un tableau de tâches.</span><span class="sxs-lookup"><span data-stu-id="f9c1c-126">The jQuery [ajax](https://api.jquery.com/jquery.ajax/) function sends a `GET` request to the API, which returns JSON representing an array of to-do items.</span></span> <span data-ttu-id="f9c1c-127">La fonction de rappel `success` est appelée si la requête réussit.</span><span class="sxs-lookup"><span data-stu-id="f9c1c-127">The `success` callback function is invoked if the request succeeds.</span></span> <span data-ttu-id="f9c1c-128">Dans le rappel, le DOM est mis à jour avec les informations des tâches.</span><span class="sxs-lookup"><span data-stu-id="f9c1c-128">In the callback, the DOM is updated with the to-do information.</span></span>

[!code-javascript[](first-web-api/samples/3.0/TodoApi/wwwroot/site.js?name=snippet_GetData)]

### <a name="add-a-to-do-item"></a><span data-ttu-id="f9c1c-129">Ajouter une tâche</span><span class="sxs-lookup"><span data-stu-id="f9c1c-129">Add a to-do item</span></span>

<span data-ttu-id="f9c1c-130">La fonction [ajax](https://api.jquery.com/jquery.ajax/) envoie une requête `POST` dont le corps indique la tâche.</span><span class="sxs-lookup"><span data-stu-id="f9c1c-130">The [ajax](https://api.jquery.com/jquery.ajax/) function sends a `POST` request with the to-do item in the request body.</span></span> <span data-ttu-id="f9c1c-131">Les options `accepts` et `contentType` sont définies avec la valeur `application/json` pour spécifier le type de média qui est reçu et envoyé.</span><span class="sxs-lookup"><span data-stu-id="f9c1c-131">The `accepts` and `contentType` options are set to `application/json` to specify the media type being received and sent.</span></span> <span data-ttu-id="f9c1c-132">La tâche est convertie au format JSON à l’aide de [JSON.stringify](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/JSON/stringify).</span><span class="sxs-lookup"><span data-stu-id="f9c1c-132">The to-do item is converted to JSON by using [JSON.stringify](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/JSON/stringify).</span></span> <span data-ttu-id="f9c1c-133">Quand l’API retourne un code d’état de réussite, la fonction `getData` est appelée pour mettre à jour la table HTML.</span><span class="sxs-lookup"><span data-stu-id="f9c1c-133">When the API returns a successful status code, the `getData` function is invoked to update the HTML table.</span></span>

[!code-javascript[](first-web-api/samples/3.0/TodoApi/wwwroot/site.js?name=snippet_AddItem)]

### <a name="update-a-to-do-item"></a><span data-ttu-id="f9c1c-134">Mettre à jour une tâche</span><span class="sxs-lookup"><span data-stu-id="f9c1c-134">Update a to-do item</span></span>

<span data-ttu-id="f9c1c-135">La mise à jour d’une tâche est similaire à l’ajout d’une tâche.</span><span class="sxs-lookup"><span data-stu-id="f9c1c-135">Updating a to-do item is similar to adding one.</span></span> <span data-ttu-id="f9c1c-136">L’identificateur unique de la tâche est ajouté à l’`url` et le `type` est `PUT`.</span><span class="sxs-lookup"><span data-stu-id="f9c1c-136">The `url` changes to add the unique identifier of the item, and the `type` is `PUT`.</span></span>

[!code-javascript[](first-web-api/samples/3.0/TodoApi/wwwroot/site.js?name=snippet_AjaxPut)]

### <a name="delete-a-to-do-item"></a><span data-ttu-id="f9c1c-137">Supprimer une tâche</span><span class="sxs-lookup"><span data-stu-id="f9c1c-137">Delete a to-do item</span></span>

<span data-ttu-id="f9c1c-138">Pour supprimer une tâche, vous devez définir le `type` sur l’appel AJAX avec la valeur `DELETE` et spécifier l’identificateur unique de l’élément dans l’URL.</span><span class="sxs-lookup"><span data-stu-id="f9c1c-138">Deleting a to-do item is accomplished by setting the `type` on the AJAX call to `DELETE` and specifying the item's unique identifier in the URL.</span></span>

[!code-javascript[](first-web-api/samples/3.0/TodoApi/wwwroot/site.js?name=snippet_AjaxDelete)]

<span data-ttu-id="f9c1c-139">Passez au tutoriel suivant pour apprendre à générer des pages d’aide d’API :</span><span class="sxs-lookup"><span data-stu-id="f9c1c-139">Advance to the next tutorial to learn how to generate API help pages:</span></span>

> [!div class="nextstepaction"]
> <xref:tutorials/get-started-with-swashbuckle>
::: moniker-end