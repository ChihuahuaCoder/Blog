---
title: "How to deploy your .Net API on Heroku with Docker and Github Actions"
datePublished: Thu Jun 06 2024 07:50:48 GMT+0000 (Coordinated Universal Time)
cuid: clx2yk9n000010ak29dav1imr
slug: how-to-deploy-your-net-api-on-heroku-with-docker-and-github-actions
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1717660123569/eccf27dc-e2a0-459e-8904-8eea678f4e84.jpeg
tags: docker, net, heroku, github-actions-1

---

Heroku is a cheap and simple hosting platform to host your apps. Basic hosting costs around 5$/month for many hours of active app. That means that you can host multiple applications for that cost.

## Create your app on Heroku

Click "New" and choose "Create new app".

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1717585758576/93f143ee-b9d5-413f-b7ea-5fde769fdf05.jpeg align="center")

Add "App name", choose a region, and click "Create app".

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1717585781630/7bb43257-0d78-49c8-bba0-47c2db8a82ea.jpeg align="center")

### **Add a database (if you need it)**

In your app "Overview" click "Configure Add-ons".

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1717585941009/1d94def5-967f-401e-8c62-2a2bf88a8240.jpeg align="center")

Postgres is the only official database on Heroku so further instructions will be given based on it. Choose "Heroku Postgres".

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1717585982621/1f93d5a9-dc0f-4f16-84a2-4e503b0681b2.jpeg align="center")

### **Configure database connection in your project**

The easiest way to configure a connection with the database is by using the library [HerokuDbConnector](https://www.nuget.org/packages/HerokuDbConnector/). It will build a database connection string for you.

```bash
dotnet add package HerokuDbConnector
```

In your `Program.cs`, before you add the db context, configure the connection string depending on the environment. If it is production, use the HerokuDbConnector builder.

```csharp
    string connectionString;
    if (builder.Environment.IsProduction()) {
        connectionString = new HerokuDbConnector.HerokuDbConnector().Build();
    }
    else {
        connectionString = builder.Configuration.GetConnectionString("Default")!;
    }
        
    builder.Services.AddDbContext<YourApplicationDbContext>(o => o.UseNpgsql(connectionString));
```

## Create Dockerfile

Your Dockerfile will consist of three parts: build ASP.NET API, add support for ASPNETCORE\_ENVIRONMENT in Dockerfile, and set PORT variable for Heroku hosting.

```dockerfile
# Build ASP.NET API
# Match your .Net version, for me it was .Net 8
FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS base
WORKDIR /app
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src
COPY ["YourApi/YourApi.csproj", "YourApi/"]
RUN dotnet restore "YourApi/YourApi.csproj"
COPY . .
WORKDIR "/src/YourApi"
RUN dotnet build "YourApi.csproj" -c Release -o /app/build
FROM build AS publish
RUN dotnet publish "YourApi.csproj" -c Release -o /app/publish /p:UseAppHost=false
FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
# Adds support for ASPNETCORE_ENVIRONMENT in Dockerfile
ARG ENVIRONMENT
ENV ASPNETCORE_ENVIRONMENT ${ENVIRONMENT}
# For Heroku hosting:
# Dockerfile cannot have EXPOSE
# Application must be using $PORT variable given by Heroku
ARG PORT
CMD ASPNETCORE_URLS=http://*:$PORT dotnet YourApi.dll
```

## Create a Github Action workflow

First, you have to get your Heroku API Key. Go to your "Account Settings" in Heroku.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1717586166324/1aa884f4-c1b7-4346-bb05-b3cdf7f06c58.jpeg align="center")

Then reveal your API Key.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1717586185925/d3d42e7b-df09-422b-bf7f-8293c947bdc2.jpeg align="center")

Next, you have to add a repository secret.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1717502033495/5444c995-a064-4246-bd31-19fee04199c1.png align="center")

Go to your API repository, next "Settings", then "Action" in "Secrets and variables". Click "New repository secret".

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1717502485533/41ddba23-e70b-4f27-8581-847cdd281b9f.png align="center")

The name is up to you. In the Secret field paste your Heroku API Key. Click "Add secret".

Now, you can create a Github Action file.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1717503120674/fa4892d5-9d6f-40c3-9964-3f683fc7c422.png align="center")

Go to "Action" and choose "Set up a workflow yourself".

```yaml
name: Deploy - API

# Action will be trigger manually (worflow_dispatch) or by pushing on main brach 
on:
  push:
    branches: 
      - main
    
  workflow_dispatch:

jobs:
  build-api:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Deploy to Heroku
        uses: akhileshns/heroku-deploy@v3.12.12 
        with:
          # Name of the secret you have added in your repo
          heroku_api_key: ${{secrets.HEROKU_KEY}}
          heroku_app_name: your-heroku-app-name 
          heroku_email: your-heroku-account@email.com
          usedocker: true
          appdir: path/to/your/Dockerfile/directory
          docker_build_args: |
            ENVIRONMENT
        env:
          ENVIRONMENT: Production
```

And... that's it :) After deployment, your API (and database) will be hosted on Heroku.