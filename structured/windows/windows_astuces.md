**Contexte** = début 2025, j'essaye de me setup une VM windows depuis un host linux.

* [connaître la version de windows](#connaître-la-version-de-windows)
* [PowerShell](#powershell)
   * [installation](#installation)
   * [lancer PowerShell en tant qu'adminstrateur :](#lancer-powershell-en-tant-quadminstrateur-)
   * [emplacement du profil PowerShell](#emplacement-du-profil-powershell)
   * [Connaître l'espace disque disponible](#connaître-lespace-disque-disponible)
   * [savoir si le shell est en mode Administrateur](#savoir-si-le-shell-est-en-mode-administrateur)
   * [modifier le PATH](#modifier-le-path)
   * [lister les utilisateurs d'un groupe](#lister-les-utilisateurs-dun-groupe)
   * [supprimer un répertoire non-vide](#supprimer-un-répertoire-non-vide)
   * [grepper le résultat d'une commande](#grepper-le-résultat-dune-commande)
   * [Afficher le contenu d'une variable](#afficher-le-contenu-dune-variable)
   * [Dumper l'output d'une commande dans un fichier](#dumper-loutput-dune-commande-dans-un-fichier)
   * [Équivalent de cat](#équivalent-de-cat)
   * [where pour savoir comment est résolu une commande](#where-pour-savoir-comment-est-résolu-une-commande)
   * [Comparer deux fichiers](#comparer-deux-fichiers)
   * [Renommer un fichier](#renommer-un-fichier)
* [Network](#network)
   * [ping](#ping)
   * [resolve dns](#resolve-dns)
* [chocolatey](#chocolatey)
* [Divers via l'IHM](#divers-via-lihm)
   * [raccourci pour accéder aux téléchargements de edge](#raccourci-pour-accéder-aux-téléchargements-de-edge)
   * [fonctionnalités windows spécifiques](#fonctionnalités-windows-spécifiques)
   * [APPDATA](#appdata)
   * [Exécuter un raccourci en mode administrateur](#exécuter-un-raccourci-en-mode-administrateur)

# connaître la version de windows

Pour connaître la version précise de windows (y compris le numéro de build) : dans un terminal, taper `winver`


# PowerShell

PowerShell semble être le shell textuel le plus utilisé par les devs sous windows.

## installation

cf. mes notes [sur l'installation de windows](./windows_install_on_vm.md).

## lancer PowerShell en tant qu'adminstrateur :

Dans "Taper ici pour rechercher" > PowerShell

Clic-droit dessus → `Exécuter en tant qu'administrateur`

## emplacement du profil PowerShell

```
C:\Users\phidra> $PROFILE
C:\Users\phidra\Documents\PowerShell\Microsoft.PowerShell_profile.ps1
```

## Connaître l'espace disque disponible

Connaître l'espace disque disponible en ligne de commande :

```
Get-PSDrive -PSProvider FileSystem
```

## savoir si le shell est en mode Administrateur

```
$identity = [System.Security.Principal.WindowsIdentity]::GetCurrent()
$principal = New-Object System.Security.Principal.WindowsPrincipal($identity)
if ($principal.IsInRole([System.Security.Principal.WindowsBuiltInRole]::Administrator)) {
    "PowerShell est exécuté en tant qu'administrateur."
} else {
    "PowerShell n'est pas exécuté en tant qu'administrateur."
}
```

## modifier le PATH

Dans "Taper ici pour rechercher" > `Modifier les variables d'environnements`

## lister les utilisateurs d'un groupe

Exemple ici, lister les utilisateurs docker :

```sh
Get-LocalGroupMember -Group "docker-users"
```

## supprimer un répertoire non-vide

```
rmdir /s myfolder
```

## grepper le résultat d'une commande

Grep direct avec `Select-String` :

```
choco search dotnet|Select-String "dotnet"
```

Reverse-grep (équivalent de `grep -v`) avec un truc alambiqué à base de `Where-Object` :

```
choco search dotnet | Where-Object { $_ -notmatch "dotnetcore" }
```

## Afficher le contenu d'une variable

Il suffit de mettre la variable sans commande (pas besoin de `echo`) :

```
$env:USERPROFILE
```

**NOTE** : d'une façon générale, il y a l'autocomplétion sur les variables.

## Dumper l'output d'une commande dans un fichier

Ça marche comme sous linux, avec `>` et `>>` :

```
choco list  > $env:USERPROFILE\choco_list_output.txt
choco list >> $env:USERPROFILE\choco_list_output.txt
```

## Équivalent de cat

On utilise `gc` (qui est un alias pour `Get-Content`) :

```
gc          $env:USERPROFILE\choco_list_output.txt
Get-Content $env:USERPROFILE\choco_list_output.txt
```

## where pour savoir comment est résolu une commande

```sh
where dotnet
C:\Program Files\dotnet\dotnet.exe
```

## Comparer deux fichiers

Il faut utiliser `fc.exe` car `fc` est reconnu pour une autre commande (Format-Custom) :

```
fc.exe file1 file2
```

## Renommer un fichier

On utilise `ren` (qui est un alias pour `Rename-Item`) :

```
ren  old_name  new_name
```

# Network

## ping

```
Test-Connection -ComputerName "my.target.domain" -Count 4

Destination: my.target.domain
Ping Source           Address                   Latency BufferSize Status
---- ------           -------                   ------- ---------- ------
   1 DESKTOP-SOURCE   *                               *          * TimedOut
   2 DESKTOP-SOURCE   *                               *          * TimedOut
   3 DESKTOP-SOURCE   *                               *          * TimedOut
```

## resolve dns

```
Resolve-DnsName -Name "my.target.domain"
```

# chocolatey

Évidemment, la commande pour installer choco est un truc imbittable... Depuis un PowerShell/Terminal en mode Administrateur :

```
Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))
```

Derrière, l'usage :

```sh
# lister les packages installables :
choco search dotnet|Select-String "dotnet"

# installer un package :
choco install dotnet

# détails sur un package :
choco info dotnet

# lister les packages installés :
choco list

# désinstaller :
choco uninstall dotnetcore-sdk  --removedependencies
# ATTENTION : ne pas oublier l'option, sinon on ne supprime que le package (et pas ses dépendances)
```

# Divers via l'IHM

## raccourci pour accéder aux téléchargements de edge

`Ctrl + J`

## fonctionnalités windows spécifiques

On peut activer des fonctionnalités windows spécifiques : taper `features` dans la barre de recherche.

## APPDATA

Entrer `%APPDATA%` dans la barre de chemin de l'explorateur de fichiers me redirige vers le répertoire où j'ai mes données.

## Exécuter un raccourci en mode administrateur

Quand on a un raccourci pour lancer rapidement une app (barre du bas), de façon contre-intuitive, pour la lancer il faut DEUX clic-droits (i.e. il faut faire un deuxième clic-droit... sur un item du menu contextuel ouvert par le premier clic-droit !) :

- premier clic-droit sur l'icône `Terminal` → ça liste des trucs, parmi lesquels je retrouve l'application `Terminal`
- deuxième clic-droit sur `Terminal` = permet d'ouvrir, de désépingler, ou... d'exécuter en tant qu'adminitrateur `\o/`
