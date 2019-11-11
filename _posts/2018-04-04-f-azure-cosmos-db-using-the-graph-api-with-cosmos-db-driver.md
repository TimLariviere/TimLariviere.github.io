---
layout: post
title: "F# - Azure Cosmos DB - Using the Graph API with Cosmos DB driver"
date: "2018-04-04"
---

_The full sample can be found on Github : [https://github.com/TimLariviere/Sample-FSharp-AzureCosmosDb](https://github.com/TimLariviere/Sample-FSharp-AzureCosmosDb)_

Setting up a graph database is really easy thanks to Azure Cosmos DB. Azure Cosmos DB is a multi-model database hosted on Microsoft Azure with lots of cool features such as a Graph API (with support for the popular Gremlin language) and turn-key worldwide distribution. For more information on Azure Cosmos DB, [head to the documentation](https://docs.microsoft.com/en-us/azure/cosmos-db/introduction).

In this post, I will show you how to make a .NET Core app written in F# that will use the Graph API of Azure Cosmos DB.

### Azure Cosmos DB Graph SDK

Microsoft made an SDK to interact with the Graph API, available on NuGet : [Microsoft.Azure.Graphs](https://www.nuget.org/packages/Microsoft.Azure.Graphs) It's currently in preview, so to find it, you'll have to make sure to enable preview versions.

This SDK contains methods for most of the things we'll need for the graph database. It handles connecting to it, querying it, and even dynamically creating new graphs.

It comes with its own Gremlin driver (the thing that handles the connection). That's what I call the Cosmos DB driver. It has some limitations compared to the Gremlin.NET driver (for instance, named parameters are missing). I will explain how to use Gremlin.NET instead in a next post.

Before we're able to use the SDK, we'll need some configuration from the Azure portal.

### Getting the configuration parameters

Creating a new Azure Cosmos DB resource on the Azure portal is really simple. Just select "Azure Cosmos DB account", click "Create" and fill out the form (and make sure to select "Gremlin (graph)")

[![](images/CreateAzureCosmosDbAccount.png)](http://timothelariviere.com/wp-content/uploads/2018/03/CreateAzureCosmosDbAccount.png)

Wait until Azure is done creating the resource.

To use the SDK, we will need the following values :

- Endpoint
- AuthKey
- DatabaseName
- GraphName
- OfferThroughput

`Endpoint` and `AuthKey` can be found in the `Keys` tab. `Endpoint` is the URI field and `AuthKey` is either the Primary or Secondary Key. [![](images/Keys.png)](http://timothelariviere.com/wp-content/uploads/2018/03/Keys.png)

For the last 3 parameters, `DatabaseName` will be the name of the database (much like a database server in SQL Server) that will contain the graphs (the real databases). `GraphName` will be the name of our graph, and `OfferThroughput` is an indicator of the level of service you want for your graph (measured in Request Units per second, each request has its own RU cost).

For the sample, we'll go with DatabaseName = "sample-cosmosdb-database", GraphName = "sample-cosmosdb-graph" and OfferThroughput = 400 (the minimum). We'll store those settings in a json file named appsettings.json that our app will read.

_appsettings.json_ \[sourcecode language="javascript"\] { "AzureCosmosDb": { "Endpoint" : "https://\[YOUR-AZURECOSMOSDB-ACCOUNT-NAME\] .documents.azure.com:443/", "AuthKey" : "\[YOUR-PRIMARY-KEY\]", "DatabaseName" : "sample-cosmosdb-database", "GraphName" : "sample-cosmosdb-graph", "OfferThroughput" : 400 } } \[/sourcecode\]

### Creating the project

Making a .NET Core app in F# is as simple as running a single command line: (Of course, you can also create the project using Visual Studio) `dotnet new console -lang F# -n AzureCosmosDbSample`

This creates 2 files (`AzureCosmosDbSample.fsproj` and `Program.fs`) for a working Hello world console in F#/.NET Core.

Before going on, we will need a few NuGet packages:

- [Microsoft.Azure.Graphs](https://www.nuget.org/packages/Microsoft.Azure.Graphs): Obviously
- [FSharp.Control.AsyncSeq](https://www.nuget.org/packages/FSharp.Control.AsyncSeq): We will use it to asynchronously retrieve the query results
- [Microsoft.Extensions.Configuration.Json](https://www.nuget.org/packages/Microsoft.Extensions.Configuration.Json): To read our settings file
- [FSharp.Core](https://www.nuget.org/packages/FSharp.Core): [Because you should always reference it explicitly](https://fsharp.github.io/2015/04/18/fsharp-core-notes.html#always-reference-fsharpcore-via-the-nuget-package)

Add the latest version of each using the NuGet GUI of Visual Studio, or if you prefer command lines: `dotnet add package Microsoft.Azure.Graphs -v 0.3.1-preview dotnet add package FSharp.Control.AsyncSeq -v 2.0.21 dotnet add package Microsoft.Extensions.Configuration.Json -v 2.0.1 dotnet add package FSharp.Core -v 4.3.4`

Now we can start writing code in the `Program.fs` file.

### Using the Azure Cosmos DB SDK

The key class in the Azure Cosmos DB SDK is `DocumentClient`, so let's make a function to instantiate it. The constructor takes our previously defined `Endpoint` and `AuthKey`.

\[sourcecode language="fsharp"\] let createClient endpoint (authKey: string) = new DocumentClient( new Uri(endpoint), authKey, ConnectionPolicy.Default ) \[/sourcecode\]

Next, as we didn't created the database and graph manually, we need to ensure that the database is properly created before querying the graph. Lucky for us, `DocumentClient` has a `CreateDatabaseIfNotExistsAsync` method that takes only our configured `DatabaseName`. Note that the code below doesn't return the database. That's because the database instance is not required to run queries.

\[sourcecode language="fsharp"\] let createDatabaseAsync databaseName (client: DocumentClient) = new Database (Id = databaseName) |> client.CreateDatabaseIfNotExistsAsync |> Async.AwaitTask |> Async.Ignore \[/sourcecode\]

Same goes for the graph (using `CreateDocumentCollectionIfNotExistsAsync`). Except this time, we need to return the graph's instance to be able to run queries afterwards.

We provide our `DatabaseName`, `GraphName`, and `OfferThroughput`.

\[sourcecode language="fsharp"\] let createGraphAsync databaseName graphName offerThroughput (client: DocumentClient) = let throughput = Nullable<int>(offerThroughput)

client.CreateDocumentCollectionIfNotExistsAsync( UriFactory.CreateDatabaseUri(databaseName), new DocumentCollection (Id = graphName), new RequestOptions (OfferThroughput = throughput) ) |> Async.AwaitTask |> (fun asyncResult -> async { let! result = asyncResult return result.Resource }) \[/sourcecode\]

And now for the crucial part, running queries. Queries are made by calling `client.CreateGremlinQuery` with the previously created client, graph and the actual Gremlin request (as a string). This returns a Reader that exposes `ExecuteNextAsync` to get the next page of results. To ease its usage, the call to `ExecuteNextAsync` is wrapped into an asynchronous sequence. So enumerating that sequence will give all the query's results, in an asynchronous manner.

Note that `ExecuteNextAsync` is generic. Behind the scene, every results retrieved from Azure Cosmos DB (sent as a JSON payload, known as [GraphSON](http://tinkerpop.apache.org/docs/current/reference/#graphson-reader-writer)) will be converted to the given type via Newtonsoft.Json. The SDK already has some predefined types for that task, such as [Vertex and Edge](https://docs.microsoft.com/en-us/dotnet/api/microsoft.azure.graphs.elements).

\[sourcecode language="fsharp"\] let runGremlinQuery<'T> (dq: IDocumentQuery<'T>) = asyncSeq { while (dq.HasMoreResults) do let! items = dq.ExecuteNextAsync<'T>() |> Async.AwaitTask for item in items do yield item }

let runQueryWithClient<'T> (client: DocumentClient) graph query = client.CreateGremlinQuery<'T>(graph, query) |> runGremlinQuery<'T> \[/sourcecode\]

Last step, read our settings file using Microsoft.Extensions.Configuration.Json.

\[sourcecode language="fsharp"\] type AzureCosmosDbConfiguration = { Endpoint: string AuthKey: string DatabaseName: string GraphName: string OfferThroughput: int }

let getConfiguration() = let configurationBuilder = new ConfigurationBuilder() configurationBuilder .SetBasePath(Directory.GetCurrentDirectory()) .AddJsonFile("appsettings.json") .Build()

let readAzureCosmosDbConfiguration (configuration: IConfigurationRoot) = { Endpoint = configuration.\["AzureCosmosDb:Endpoint"\] AuthKey = configuration.\["AzureCosmosDb:AuthKey"\] DatabaseName = configuration.\["AzureCosmosDb:DatabaseName"\] GraphName = configuration.\["AzureCosmosDb:GraphName"\] OfferThroughput = (configuration .\["AzureCosmosDb:OfferThroughput"\] |> int) } \[/sourcecode\]

We have all our tooling ready, now we can move on and actually run the sample.

### Running the sample

First step is to initialize our environment.

\[sourcecode language="fsharp"\] // Get the configuration let configuration = getConfiguration() |> readAzureCosmosDbConfiguration

// Create the client let client = createClient configuration.Endpoint configuration.AuthKey

// Ensure that the database is created do! createDatabaseAsync configuration.DatabaseName client

// Ensure that the graph is created // and store its instance let! graph = createGraphAsync configuration.DatabaseName configuration.GraphName configuration.OfferThroughput client \[/sourcecode\]

To have a code easier to read, I partially applied the call to `runQueryWithClient` to always use the same client and graph.

\[sourcecode language="fsharp"\] // Prepare a query runner that won't // return results let executeQuery = runQueryWithClient<obj> client graph >> AsyncSeq.iter ignore

// Prepare a query runner that will // return a list of vertices let getVertices = runQueryWithClient<Vertex> client graph >> AsyncSeq.toListAsync

// Prepare a query runner that will // return a single vertex let getSingleVertex = getVertices >> List.head \[/sourcecode\]

Now we're all done, and can start running our queries.

\[sourcecode language="fsharp"\] // Add data do! executeQuery "g.addV('person') .property('id', 'thomas') .property('firstName', 'Thomas') .property('age', 44)"

// Retrieve data let! thomas = getSingleVertex "g.V('thomas')" \[/sourcecode\]
