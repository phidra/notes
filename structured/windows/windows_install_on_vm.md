**Contexte** = début 2025, j'essaye de me setup une VM windows depuis un host linux.

* [Installation de windows](#installation-de-windows)
* [À installer sur une VM fraîche](#à-installer-sur-une-vm-fraîche)
   * [guest-additions](#guest-additions)
   * [Powershell](#powershell)
   * [Installation de Windows-Terminal](#installation-de-windows-terminal)
   * [WSL](#wsl)
   * [chocolatey](#chocolatey)
   * [Connecter un lecteur réseau samba](#connecter-un-lecteur-réseau-samba)
   * [docker](#docker)
   * [nuget](#nuget)
   * [dotnet](#dotnet)

# Installation de windows

Je télécharge windows 10 sur le site microsoft : https://www.microsoft.com/fr-fr/software-download/windows10iso

(windows 11 nécessite un compte utilisateur alors que windows10 peut être utilisé en tant qu'anonyme)

Ne pas hésiter à viser grand sur la taille du disque-dur : avec qemu, seul l'espace réellement consommé compte + les partitions windows sont plus difficiles à resizer que sous linux.

Dans le process d'installation :

- je n'entre pas de clé de produit
- je choisis "Windows 10 professionnel N" (le `N` correspond à l'absence de media player)
- configurer pour une utilisation personnelle
- compte hors-connexion
- expérience limitée (pour ne pas connecter mes comptes)
- je refuse toutes les autorisations spéciales

# À installer sur une VM fraîche

## guest-additions

Si applicable (i.e. si la VM est qemu), télécharger et installer `spice-guest-tools-latest.exe` pour pouvoir copier/coller depuis la VM QEMU : https://www.spice-space.org/download.html

(il y a l'équivalent pour VirtualVox)


## Powershell

(possiblement inutile car PowerShell viendrait avec Windows-Terminal ? À tester la prochaine fois = commencer par installer d'abord Windows-Terminal)

Je recommande d'installer via le MSI ([doc](https://learn.microsoft.com/fr-fr/powershell/scripting/install/installing-powershell-on-windows?view=powershell-7.5#installing-the-msi-package))

Pour cela, télécharger et exécuter le MSI depuis [la page des release github](https://github.com/PowerShell/PowerShell/releases) (exemple : https://github.com/PowerShell/PowerShell/releases/download/v7.5.0/PowerShell-7.5.0-win-x64.msi )


DEPRECATED : j'avais aussi testé [l'install via winget](https://learn.microsoft.com/fr-fr/powershell/scripting/install/installing-powershell-on-windows?view=powershell-7.5#winget), mais j'ai finalement préféré revenir sur le MSI, je n'ai pas noté pourquoi :

```
winget search Microsoft.PowerShell
winget install --id Microsoft.PowerShell --source winget
```

## Installation de Windows-Terminal

[Cette page](https://learn.microsoft.com/fr-fr/windows/terminal/install) a un bouton `Installer` qui me redirige vers [le store microsoft](https://apps.microsoft.com/detail/9n0dx20hk701?hl=fr-FR&gl=FR) depuis lequel je télécharge un installer `Windows Terminal Installer.exe`.

Windows-Terminal semble venir accompagné de PowerShell (et truc cool, il peut afficher des icônes UTF-8 out-of-the-box).

## WSL

WSL = [Windows Subsystem for Linux](https://fr.wikipedia.org/wiki/Windows_Subsystem_for_Linux) = couche de compatibilité pour que windows puisse exécuter des binaires linux.

Note préliminaire : pour savoir si wsl est installé, c'est un peu trompeur : même si wsl n'est PAS installé, la commande suivante n'échouera pas et renverra quelque chose :

```
wsl --version
wsl --status
```

Pour installer wsl :

```
wsl --install --web-download
```

Une fois wsl correctement installé, `wsl --version` renvoie quelque chose du genre :

```
Version WSL : 2.4.13.0
Version du noyau : 5.15.167.4-1
Version WSLg : 1.0.65
Version MSRDC : 1.2.5716
Version direct3D : 1.611.1-81528511
Version de DXCore : 10.0.26100.1-240331-1435.ge-release
Version de Windows : 10.0.19045.5796
```

## chocolatey

Depuis un PowerShell/Terminal **en mode Administrateur** :

```
Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))
```

À ce stade, je reboote par précaution... (je fais pas confiance à Windows)

## Connecter un lecteur réseau samba

cf. [mes notes samba](./windows_samba.md)

## docker

(si besoin, [doc sur comment préparer windows pour les containers](https://learn.microsoft.com/en-us/virtualization/windowscontainers/quick-start/set-up-environment?tabs=dockerce)).

Depuis un PowerShell/Terminal **en mode Administrateur** :

```
choco install Microsoft-Hyper-V -source windowsFeatures
choco install Containers -source windowsFeatures
choco install docker-desktop  # assez long
```

Switcher docker vers des containers windows :

- clic-droit sur l'icône de docker dans le systray
- cliquer sur `Switch to windows containers`
- j'ai eu l'erreur suivante lors du switch :
    > The Privileged helper service is not running. The service runs in the background with SYSTEM privileges.
    > Docker Desktop needs the service to interact with privileged parts of Windows.
    >
    > Would you like to start the service? Windows will ask you for elevated access.
- j'ai cliqué sur `Start Service` → pas de souci

Vérifier si mon utilisateur est dans le groupe docker :

```
Get-LocalGroupMember -Group "docker-users"
```

À confirmer = il faut alors me déconnecter + me reconnecter pour que mon user soit dans le groupe docker.

À noter qu'a priori, DockerDesktop n'est pas lancé au démarrage -> à lancer à la main.

## nuget

(nuget = package manager dotnet, équivalent de pip ou npm)

```
choco install nuget.commandline
```

## dotnet

cf. [mes notes dotnet](./windows_dotnet.md).
