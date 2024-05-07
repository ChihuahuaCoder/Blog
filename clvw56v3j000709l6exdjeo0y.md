---
title: "Refit - Simple source code generated HTTP client library for .Net"
datePublished: Tue May 07 2024 08:42:14 GMT+0000 (Coordinated Universal Time)
cuid: clvw56v3j000709l6exdjeo0y
slug: refit-simple-source-code-generated-http-client-library-for-net
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1715069218007/57c26e18-23cc-4046-b4c8-e511a77019ea.jpeg
tags: csharp, net, client, refit

---

Refit is a library that turns the REST API into a living interface. With its help, I have created a PoC of a client that communicates with the REST API. As an example, I have used the [Wizard World API](https://wizard-world-api.herokuapp.com/swagger/index.html), which returns content about the Harry Potter universe.

## Installation

Installation is straightforward, just add [Refit](https://www.nuget.org/packages/Refit/) from Nuget. In case of complications, full documentation is available [here](https://github.com/reactiveui/refit).

```bash
dotnet add package Refit
```

## API file

Refit supports six HTTP methods: `Get`, `Post`, `Put`, `Delete`, `Patch`, and `Head`. In my POC I have included two of them: `Get` and `Post`.

```csharp
public interface IWizardWorldApi {
    [Get("/Elixirs/{id}")]
    Task<Elixir> GetElixir(Guid id);

    [Get("/Elixirs")]
    Task<ICollection<Elixir>> GetElixirs(ElixirsQueryParams elixirsQueryParams);

    [Post("/Feedback")]
    Task PostFeedback(FeedbackRequest feedbackRequest);
}
```

I have created an interface whose methods correspond to the endpoints I intend to consume. Each function has an attribute in which I specify the HTTP method and provide the path to the resource. `GetElixir` returns a specific elixir by id, which is given in the URL. The name of the parameter in the URL must match the name of the method argument, otherwise an alias should be used:

```csharp
[Get("/Elixirs/{id}")]
Task<Elixir> GetElixir([AliasAs("id")] Guid elixirId);
```

If parameters are not specified as a URL they will be automatically be used as query parameters. I have applied this to the `GetElixirs` method, which allows `ElixirsQueryParams` to filtering of the list of elixirs.

```csharp
public class ElixirsQueryParams {
    public string? Name { get; set; }
    public Difficulty? Difficulty { get; set; }
    public string? Ingredient { get; set; }
    public string? InventorFullName { get; set; }
    public string? Manufacturer { get; set; }
}
```

For a member to be considered a query parameter it must be property and its value must not be null. Otherwise, it will be ignored.

The `GetFeedback` method accepts `FeedbackRequest`:

```csharp
public class FeedbackRequest {
    [JsonPropertyName("feedbackType")]
    public FeedbackType? Type { get; set; }
    [JsonPropertyName("feedback")]
    public string? Message { get; set; }
    [JsonPropertyName("entityId")]
    public Guid? Id { get; set; }
}
```

Refit uses `System.Text.Json` by default to handle serialization, but it is possible to switch to `Newtonsoft.Json` using configuration.

## Usage

Usage is very straightforward.

```csharp
var client = RestService.For<IWizardWorldApi>("https://wizard-world-api.herokuapp.com");
var queryParams = new ElixirsQueryParams {
    Name = "Fire-Protection Potion"
};
var elixirs = await client.GetElixirs(queryParams);
```

In the first line, I create my service for WizarWorldApi. I pass the URL of the API as an argument. In the next step, I prepare my query parameters object and then pass it in as an argument. As a result, I get a list of potions, which are called Fire-Protection Potion.

And... that's it :) It is very easy to start playing with Refit.

## Three ways to debug Refit

Playing with Refit, I noticed that it is very... secretive. I found it hard to debug it. I found three ways to do this.

### Take a look at the generated code.

For me this is more of a curiosity, but could potentially be useful. The files with the generated code are located in the Dependencies of the project, in the .Net folder, further down SourceGenerators. In my case, the path looks like this:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1714473353378/077935a6-2de6-45ed-a4a9-9158ef3878df.png align="center")

### Use IApiResponse.

Refit provides an IApiResponse interface that will make the response contain a lot of useful information.

In the API interface, the returned type should be changed from `Task<T>` to `Task<IApiResponse>`. In this way, you will gain information about the HTTP status, potential error, request content, URL and much more.

```csharp
[Get("/Elixirs/{id}")]
Task<IApiResponse<Elixir>> GetElixir(Guid id);
```

### Add logger

It requires the most work but provides information and does not affect the type returned. Firstly, I created a simple logger:

```csharp
public class RefitHttpLogger : DelegatingHandler {
    private readonly ILogger _logger;

    public RefitHttpLogger(ILogger logger, HttpMessageHandler? innerHandler = null) : base(innerHandler ?? new HttpClientHandler()) {
        this._logger = logger;
    }

    protected override async Task<HttpResponseMessage> SendAsync(HttpRequestMessage request,
        CancellationToken cancellationToken) {
        _logger.LogInformation("Request: {request}", request);

        var response = await base.SendAsync(request, cancellationToken);

        _logger.LogInformation("Response: {response}", response);

        return response;
    }
```

To get the ILogger interface it was necessary to add [Microsoft.Extensions.Logging](https://www.nuget.org/packages/Microsoft.Extensions.Logging/) from Nuget. I then separated the creation of the WizardWorldApi service into a separate `WizardWorldApiFactory` class.

```csharp
public static class WizardWorldApiFactory {
    public static IWizardWorldApi Create() {
        var loggerFactory = LoggerFactory.Create(builder => builder.AddConsole());
        var logger = loggerFactory.CreateLogger<RefitHttpLogger>();
        return RestService.For<IWizardWorldApi>(new HttpClient(new RefitHttpLogger(logger)) {
            BaseAddress = new Uri("https://wizard-world-api.herokuapp.com")
        });
    }
}
```

I wanted to write logs to the console, so I added [Microsoft.Extensions.Logging.Console](https://www.nuget.org/packages/Microsoft.Extensions.Logging.Console/9.0.0-preview.3.24172.9) library to the project. In the code above, I create a logger and pass it to a new HTTP client, which I in turn pass to Refit.

After the changes made, the initialization of the client looks as follows:

```csharp
var client = WizardWorldApiFactory.Create();
```

## Bonus

The source code is available [here](https://github.com/ChihuahuaCoder/RefitPoc) :)