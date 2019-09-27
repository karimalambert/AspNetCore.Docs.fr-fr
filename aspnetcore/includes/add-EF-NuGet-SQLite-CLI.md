<span data-ttu-id="135eb-101">Exécutez les commandes CLI .NET Core suivantes :</span><span class="sxs-lookup"><span data-stu-id="135eb-101">Run the following .NET Core CLI commands:</span></span>

```dotnetcli
dotnet tool install --global dotnet-ef
dotnet add package Microsoft.EntityFrameworkCore.SQLite
dotnet add package Microsoft.VisualStudio.Web.CodeGeneration.Design
dotnet add package Microsoft.EntityFrameworkCore.Design
dotnet add package Microsoft.EntityFrameworkCore.SqlServer
```

<span data-ttu-id="135eb-102">Les commandes précédentes ajoutent :</span><span class="sxs-lookup"><span data-stu-id="135eb-102">The preceding commands add:</span></span>

* <span data-ttu-id="135eb-103">L' [outil de génération de modèles automatique ASPNET-CodeGenerator](xref:fundamentals/tools/dotnet-aspnet-codegenerator).</span><span class="sxs-lookup"><span data-stu-id="135eb-103">The [aspnet-codegenerator scaffolding tool](xref:fundamentals/tools/dotnet-aspnet-codegenerator).</span></span>
* <span data-ttu-id="135eb-104">Les outils de Entity Framework Core pour le CLI .NET Core.</span><span class="sxs-lookup"><span data-stu-id="135eb-104">The Entity Framework Core Tools for the .NET Core CLI.</span></span>
* <span data-ttu-id="135eb-105">Le fournisseur EF Core SQLite, qui installe le package EF Core comme une dépendance.</span><span class="sxs-lookup"><span data-stu-id="135eb-105">The EF Core SQLite provider, which installs the EF Core package as a dependency.</span></span>
* <span data-ttu-id="135eb-106">Les packages nécessaires à la génération de modèles automatique : `Microsoft.VisualStudio.Web.CodeGeneration.Design` et `Microsoft.EntityFrameworkCore.SqlServer`.</span><span class="sxs-lookup"><span data-stu-id="135eb-106">Packages needed for scaffolding: `Microsoft.VisualStudio.Web.CodeGeneration.Design` and `Microsoft.EntityFrameworkCore.SqlServer`.</span></span>