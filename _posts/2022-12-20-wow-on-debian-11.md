---
layout: post
title: World of Warcraft on Debian 11
categories: Gaming
date: 2022-12-20
---

Hello,

Petit mémo pour jouer à World of Warcraft sur Debian 11. Ceci est la version mise à jour du mémo sur [World of Warcraft sur Ubuntu 20.04](https://noxinmortus.github.io/gaming/2021/11/18/wow-on-ubuntu-20-04.html), j'ai en effet rencontré quelques déboires après avoir mis à jour sur Ubuntu 22.04, et j'ai donc décidé de refaire le setup.

Voici les specs du laptop sur lequel cette procédure à été réalisée:
```
[System]
OS:              GNOME 43 Flatpak runtime
Arch:            x86_64
Kernel:          5.10.0-20-amd64
Desktop:         KDE
Display Server:  x11

[CPU]
Vendor:          GenuineIntel
Model:           Intel(R) Core(TM) i5-6300U CPU @ 2.40GHz
Physical cores:  2
Logical cores:   4

[Memory]
RAM:             7.5 GB
Swap:            1.0 GB

[Graphics]
Vendor:          Intel
OpenGL Renderer: Mesa Intel(R) HD Graphics 520 (SKL GT2)
OpenGL Version:  4.6 (Compatibility Profile) Mesa 22.2.4 (git-80df10f902)
OpenGL Core:     4.6 (Core Profile) Mesa 22.2.4 (git-80df10f902)
OpenGL ES:       OpenGL ES 3.2 Mesa 22.2.4 (git-80df10f902)
Vulkan:          Supported
```

## Drivers

Avant toute chose il nous faut installer les drivers graphiques à jour pour notre matériel. Voici les notes pour NVIDIA et AMD/Intel Graphics (du 18/11/2021) (<https://github.com/lutris/docs/blob/master/InstallingDrivers.md>). Vous pouvez vérifier votre matériel ainsi :

```
$ sudo lshw -C display
*-display                 
     description: VGA compatible controller
     product: Skylake GT2 [HD Graphics 520]
     vendor: Intel Corporation
     physical id: 2
     bus info: pci@0000:00:02.0
     version: 07
     width: 64 bits
     clock: 33MHz
     capabilities: pciexpress msi pm vga_controller bus_master cap_list rom
     configuration: driver=i915 latency=0
     resources: irq:128 memory:e0000000-e0ffffff memory:d0000000-dfffffff ioport:3000(size=64) memory:c0000-dffff
```

Dans mon exemple j'ai donc juste une Intel Graphics, il nous faut juste les drivers Mesa:
```
sudo apt install mesa-utils libgl1-mesa-dri mesa-vulkan-drivers mesa-vulkan-drivers
```

## Lutris on flatpak

Install flatpak, optionally flatpak backend for Discover, and finally Lutris
```
sudo apt install flatpak plasma-discover-backend-flatpak
sudo flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo
flatpak install net.lutris.Lutris
```

## Création du launcher de World of Warcraft sur Lutris

Une fois tous nos pré-requis d'installés, on peut lancer Lutris, celui-ci devrait être disponible dans votre barre d'applications.

Lutris lancé, on clique sur le petit bouton (+) en haut à gauche de l'interface pour créer un launcher:
- Dans l'onglet `Game info` on lui donne un nom (WoW par exemple),
- Dans l'onglet `Game info` on selectionne le Runner `Wine (Runs Windows games)`,
- Dans l'onglet `Game options` on clique sur `Browse` et on selectionne le chemin de l'exécutable de WoW (`WoW.exe`),
- Dans `Runner options` on s'assure que `Enable DXVK` est bien actif

On peut ensuite valider et lancer le jeu. Au premier lancement WineHQ demandera si vous souhaitez installer des composants supplémentaires, il vous faut bien cliquer sur `Install` ! Une fois tous les composants installés, le jeu devrait alors se lancer.

## Tweaks

Référence: <https://github.com/lutris/docs/blob/master/Performance-Tweaks.md>

### [MangoHUD](https://github.com/flightlessmango/MangoHud)

Une fois en jeu, vous pouvez afficher le nombre de FPS avec CTRL+R par défaut. Si vous souhaitez avoir plus d'information sur les performances GPU, vous pouvez utiliser MangoHUD :
```
flatpak install flathub org.freedesktop.Platform.VulkanLayer.MangoHud//21.08
```

### Gamemode

Référence: <https://github.com/FeralInteractive/gamemode>

>Game Mode set your CPU governor to max performance while you are playing, and can improve FPS in some cases. It's automatically enabled for all your games when you have game mode installed on your system (for Lutris).

```
sudo apt install gamemode
```

### World of Warcraft Configuration

Lutris dispose d'une page wiki de configuration dédiée à World of Warcraft: <https://github.com/lutris/docs/blob/master/WorldOfWarcraft.md>

Pour ma part j'ai juste récupéré l'option à rajouter dans le dossier WoW/WTF/Config.wtf : `SET worldPreloadNonCritical "0"`

Une fois tout ceci installé/configuré, et vu que le laptop sur lequel cette procédure est effectué est TOUT sauf une bête de guerre, j'obtenais 20 FPS poussifs avec les réglages graphiques à fond.

J'ai dû faire quelques tweaks pour que ce soit fluide à 60 FPS (mais c'est tout de même bien moche):
- Multi-Echantillonage passé à `24 bits couleur 16 bits profondeur 1x multi-échantillon`
- Désactivation de Filtre trilinéaire
- Désactivation de Sync vertical
- Désactivation de tous les effets Shader
- Désactivation des ombres lisses

A+
