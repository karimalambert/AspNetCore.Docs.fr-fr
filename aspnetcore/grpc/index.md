---
title: Présentation de gRPC sur .NET Core
author: juntaoluo
description: Découvrez les services gRPC avec le serveur Kestrel et la pile ASP.NET Core.
monikerRange: '>= aspnetcore-3.0'
ms.author: johluo
ms.date: 09/20/2019
uid: grpc/index
ms.openlocfilehash: d97eea1da28424680a3cfa38102637b1e20ff661
ms.sourcegitcommit: 9a129f5f3e31cc449742b164d5004894bfca90aa
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 03/06/2020
ms.locfileid: "78667313"
---
# <a name="introduction-to-grpc-on-net-core"></a>Présentation de gRPC sur .NET Core

Par [John Luo](https://github.com/juntaoluo) et [James Newton-King](https://twitter.com/jamesnk)

[gRPC](https://grpc.io/docs/guides/) est un framework d’appel de procédure distante (RPC) indépendant du langage et très performant.

Les principaux avantages de gRPC sont :
* Infrastructure RPC moderne, hautement performante et légère.
* Développement d’API « Contract-first », à l’aide de mémoires tampons de protocole par défaut, permettant des implémentations indépendantes du langage.
* Outils disponibles pour de nombreux langages afin de générer des clients et serveurs fortement typés.
* Prise en charge d’appels de streaming client, serveur et bidirectionnels.
* Utilisation réduite du réseau avec sérialisation binaire Protobuf.

Ces avantages font de gRPC la solution idéale pour :
* Les microservices légers où l’efficacité est essentielle.
* Les systèmes polyglottes où plusieurs langages sont nécessaires au développement.
* Les services en temps réel de point à point qui doivent gérer des demandes ou réponses de streaming.

## <a name="c-tooling-support-for-proto-files"></a>C#Prise en charge des outils pour les fichiers. proto

gRPC utilise une approche contrat d’abord pour le développement d’API. Les services et les messages sont définis dans les fichiers *\*. proto* :

```protobuf
syntax = "proto3";

service Greeter {
  rpc SayHello (HelloRequest) returns (HelloReply);
}

message HelloRequest {
  string name = 1;
}

message HelloReply {
  string message = 1;
}
```

Les types .NET pour les services, les clients et les messages sont générés automatiquement en incluant les fichiers *\*. proto* dans un projet :

* Ajoutez une référence de package au package [GRPC. Tools](https://www.nuget.org/packages/Grpc.Tools/) .
* Ajoutez *\*fichiers. proto* au groupe d’éléments `<Protobuf>`.

```xml
<ItemGroup>
  <Protobuf Include="Protos\greet.proto" />
</ItemGroup>
```

Pour plus d’informations sur la prise en charge des outils gRPC, consultez <xref:grpc/basics>.

## <a name="grpc-services-on-aspnet-core"></a>services gRPC sur ASP.NET Core

les services gRPC peuvent être hébergés sur ASP.NET Core. Les services ont une intégration complète aux fonctionnalités populaires de ASP.NET Core telles que la journalisation, l’injection de dépendances, l’authentification et l’autorisation.

Le modèle de projet de service gRPC fournit un service de démarrage :

```csharp
public class GreeterService : Greeter.GreeterBase
{
    private readonly ILogger<GreeterService> _logger;

    public GreeterService(ILogger<GreeterService> logger)
    {
        _logger = logger;
    }

    public override Task<HelloReply> SayHello(HelloRequest request,
        ServerCallContext context)
    {
        _logger.LogInformation("Saying hello to {Name}", request.Name);
        return Task.FromResult(new HelloReply 
        {
            Message = "Hello " + request.Name
        });
    }
}
```

`GreeterService` hérite du type de `GreeterBase`, qui est généré à partir du service `Greeter` dans le fichier *\*. proto* . Le service est rendu accessible aux clients dans *Startup.cs*:

```csharp
app.UseEndpoints(endpoints =>
{
    endpoints.MapGrpcService<GreeterService>();
});
```

Pour en savoir plus sur les services gRPC sur ASP.NET Core, consultez <xref:grpc/aspnetcore>.

## <a name="call-grpc-services-with-a-net-client"></a>Appeler gRPC services avec un client .NET

les clients gRPC sont des types de client concrets qui sont [générés à partir des fichiers *\*. proto* ](xref:grpc/basics#generated-c-assets). Le client gRPC concret possède des méthodes qui traduisent le service gRPC dans le fichier *\*. proto* .

```csharp
var channel = GrpcChannel.ForAddress("https://localhost:5001");
var client = new Greeter.GreeterClient(channel);

var response = await client.SayHelloAsync(
    new HelloRequest { Name = "World" });

Console.WriteLine(response.Message);
```

Un client gRPC est créé à l’aide d’un canal, qui représente une connexion à long terme à un service gRPC. Un canal peut être créé à l’aide de `GrpcChannel.ForAddress`.

Pour plus d’informations sur la création de clients et l’appel de différentes méthodes de service, consultez <xref:grpc/client>.

[!INCLUDE[](~/includes/gRPCazure.md)]

## <a name="additional-resources"></a>Ressources supplémentaires

* <xref:grpc/basics>
* <xref:grpc/aspnetcore>
* <xref:grpc/client>
* <xref:grpc/clientfactory>
* <xref:tutorials/grpc/grpc-start>
