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
      context: ./WebApi
      dockerfile: Dockerfile
    ports:
      - 5001:80
````

## Index.php erstellen

````bash
touch index.php
````

... und folgenden Inhalt einfügen:

````html
<!DOCTYPE html >
<html >
<head >
<title >M347</title >
<meta charset ="utf-8" />
</head >
<body >
<h1>M347 - Webpage mit Dockefile</h1>
<?php
$vorname = "Damian";
$nachname = "Blatt";
echo "Autor: ${vorname} ${nachname}";
?> 
</body >
</html >
````

## Image erstellen

````bash
docker build -t m347-dockerfile
docker image ls
````

Nun sollten u. a. folgende Images aufgeführt werden:

````
REPOSITORY       TAG        IMAGE ID       CREATED              SIZE
m347-dockefile   latest     c5a049e80c58   About a minute ago   458MB
php              8-apache   af944036d594   About a minute ago	458MB
````

##  Container starten

````bash
docker run -d --name m347-dockerfile-container -p 8080:80 m347-dockefile
````

## Webpage aufrufen

Die Webpage kann nun unter _localhost:8080_ aufgerufen werden.
