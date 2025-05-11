**Contexte** = début 2025, j'essaye de me setup une VM windows depuis un host linux.

* [dotnet](#dotnet)
   * [Connaître les versions et leur cycle de vie](#connaître-les-versions-et-leur-cycle-de-vie)
   * [Lister les versions de dotnet installées](#lister-les-versions-de-dotnet-installées)
* [Spécifique linux](#spécifique-linux)
   * [Installer dotnet sous linux](#installer-dotnet-sous-linux)
   * [Builder sous linux un exe pour windows](#builder-sous-linux-un-exe-pour-windows)
* [Spécifique windows](#spécifique-windows)
   * [Installer dotnet](#installer-dotnet)
   * [Les paquets dotnet dans chocolatey et leur signification](#les-paquets-dotnet-dans-chocolatey-et-leur-signification)


# dotnet

## Connaître les versions et leur cycle de vie

Le plus simple = https://endoflife.date/dotnet

En mai 2025, j'installe dotnet 8 LTS, qui arrivera en fin de vie en novembre 2026.

À noter qu'une version sur deux est LTS (Long-Term Support = 3 ans de support), l'autre est STS (Short-Term Support = 18 mois de support), [source](https://dotnet.microsoft.com/en-us/platform/support/policy/dotnet-core) :

- les versions paires sont LTS
- les versions impaires sont STS

## Lister les versions de dotnet installées

```sh
dotnet --list-sdks
dotnet --list-runtimes
```

# Spécifique linux

## Installer dotnet sous linux

```sh
wget https://dot.net/v1/dotnet-install.sh
chmod +x dotnet-install.sh
./dotnet-install.sh --channel 7.0
```

## Builder sous linux un exe pour windows

Il est apparemment possible de builder un exécutable windows depuis un vscode sous linux :

Sur le poste linux, créer le projet en C# :

```sh
dotnet new console -n MonProjet
cd MonProjet
```

Modifier le fichier `.csproj` pour indiquer qu'on compile pour la target `win-x64` :


```xml
<PropertyGroup>
  <OutputType>Exe</OutputType>
  <TargetFramework>net7.0</TargetFramework>
  <RuntimeIdentifier>win-x64</RuntimeIdentifier>
</PropertyGroup>
```

Compiler :

```
dotnet publish -c Release -r win-x64 --self-contained
```

Copier le binaire sur windows :
```
ls bin/Release/net7.0/win-x64/publish/
```

On peut exécuter sur windows un binaire linux \o/

# Spécifique windows

## Installer dotnet


**TL;DR** = il suffit d'installer le SDK, ça installe aussi les runtimes :

```
choco install dotnet-8.0-sdk
```

## Les paquets dotnet dans chocolatey et leur signification

Sous chocolatey, même si on se limite à dotnet-8.0, on a plusieurs packages disponibles, subtilement différents... Voici leur signification :

```sh
choco search dotnet | Select-String "dotnet" | Where-Object { $_ -notmatch "dotnetcore" }

# dotnet-8.0-runtime 8.0.15 [Approved]
#     le runtime principal

# dotnet-8.0-aspnetruntime 8.0.15 [Approved]
#     un sous-runtime pour faire tourner les applis ASP.NET (des applis web)

# dotnet-8.0-desktopruntime 8.0.15 [Approved]
#     un sous-runtime pour faire tourner les applis desktop (des IHM WPF, par exemple)

# dotnet-8.0-sdk 8.0.408 [Approved]
#     le SDK, qui inclut le runtime principal

# dotnet-8.0-sdk-1xx 8.0.115 [Approved] Downloads cached for licensed users
# dotnet-8.0-sdk-2xx 8.0.206 [Approved] Downloads cached for licensed users
# dotnet-8.0-sdk-3xx 8.0.311 [Approved] Downloads cached for licensed users
# dotnet-8.0-sdk-4xx 8.0.408 [Approved] Downloads cached for licensed users
#     ^ des versions précises du SDK ; pour mes besoins de débutant, inutile d'aller farfouiller là-dedans

# dotnet-8.0-windowshosting 8.0.15 [Approved]
#     un sous-runtime (?) pour hoster des applications dotnet
#     (de ce que j'en comprends : si je veux faire tourner un IIS qui sert des apps dotnet, j'ai besoin de ça)
```

Installer le SDK et suffisant. Et même si chocolatey n'installe pas explicitement les autres packages (e.g. le runtime asp.net), ils viennent tout de même avec le SDK :

```
choco install dotnet-8.0-sdk
# Installed:
# - dotnet-8.0-sdk v8.0.408
# - dotnet-8.0-sdk-4xx v8.0.408
# - KB2999226 v1.0.20181019
# - KB3033929 v1.0.5
# - KB3035131 v1.0.3
# - vcredist140 v14.44.35112.1

dotnet --list-sdks
# 8.0.408 [C:\Program Files\dotnet\sdk]

# NOTE = même si ça n'apparaît pas explicitement, installer le sdk est suffisant pour installer les runtimes :

dotnet --list-runtimes
# Microsoft.AspNetCore.App 8.0.15 [C:\Program Files\dotnet\shared\Microsoft.AspNetCore.App]
# Microsoft.NETCore.App 8.0.15 [C:\Program Files\dotnet\shared\Microsoft.NETCore.App]
# Microsoft.WindowsDesktop.App 8.0.15 [C:\Program Files\dotnet\shared\Microsoft.WindowsDesktop.App]
```
