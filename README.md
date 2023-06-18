# M347 - Minimal API

## GIT

Für die Arbeit mit GIT sollte folgender Befehl verwendet werden:

````bash
dotnet new gitignore
````

Es sorgt dafür, dass nur relevanter Source-Code ins GIT-Repository übertragen wird (ohne Binaries, etc.).

## .NET 7 installieren

````bash
# Get Ubuntu version
declare repo_version=$(if command -v lsb_release &> /dev/null; then
lsb_release -r -s; else grep -oP '(?<=ˆVERSION_ID=).+' /etc/os-release
| tr -d '"'; fi)
,→
,→
# Download Microsoft signing key and repository
wget https://packages.microsoft.com/config/ubuntu/$repo_version/packagesmicrosoft-prod.deb -O
packages-microsoft-prod.deb
,→
,→
# Install Microsoft signing key and repository
sudo dpkg -i packages-microsoft-prod.deb
# Clean up
rm packages-microsoft-prod.deb
# Update packages
sudo apt update
# Install dotnet 7 sdk
sudo apt install dotnet-sdk-7.0
````

Zur Kontrolle:

````bash
dotnet --info
````

## Ordnerstruktur erstellen

````bash
mkdir m347-minimal-api
cd m347-minimal-api
# ...
````

## .NET Projekt erstellen

````bash
dotnet new web --name WebApi

# Nuget-Package installieren
dotnet add package MongoDB.Driver
````

## Dockerfile erstellen

````bash
touch Dockerfile
nano Dockerfile
````

... und folgenden Inhalt einfügen:

````dockerfile
FROM mcr.microsoft.com/dotnet/sdk:7.0 AS build
WORKDIR /app

COPY . .

RUN dotnet restore
RUN dotnet publish -c Release -o out

FROM mcr.microsoft.com/dotnet/aspnet:7.0 AS runtime
WORKDIR /app
EXPOSE 5001

COPY --from=build /app/out .

ENTRYPOINT ["dotnet", "Minimal-API.dll"]
````

## docker-compose.yml erstellen

````yaml
version: '3'
services:
  webapi:
    build:
      context: ./minimal-api-with-mongodb/WebApi
      dockerfile: Dockerfile
    ports:
      - 5001:80
    depends_on:
      - mongodb
    environment:
      - MoviesDatabaseSettings__ConnectionString=mongodb://gbs:geheim@mongodb:27017

  mongodb:
    image: mongo
    restart: always
    ports:
      - 27017:27017
    volumes:
      - mongodb_data:/data/db

volumes:
  mongodb_data:
````

## Program.cs erstellen

````bash
touch Program.cs
````

... und folgenden Inhalt einfügen:

````html
using Microsoft.AspNetCore.Builder;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;
using MongoDB.Driver;
using System;

var builder = WebApplication.CreateBuilder(args);

// ...

var app = builder.Build();

// ...

app.MapGet("/", () => {
    return "Minimal API Version 1.0";
});

app.MapGet("/check", () =>
{
    try
    {
        var mongoClient = new MongoClient("mongodb://gbs:geheim@localhost:27017");
        var databases = mongoClient.ListDatabases().ToList();

        string response = "Zugriff auf MongoDB OK.\n\nDatenbanken:\n";
        foreach (var db in databases)
        {
            response += $"{db["name"]}\n";
        }

        return response;
    }
    catch (Exception ex)
    {
        return $"Fehler beim Zugriff auf MongoDB: {ex.Message}";
    }
});

app.Run();
````

##  Service starten

Folgender Befehl muss im Verzeichnis ausgeführt werden, das die Datei docker-compose.yml enthält.

````bash
docker-compose up
````

## Webpage aufrufen

Die Webpage kann nun unter _localhost:5001_ aufgerufen werden.
