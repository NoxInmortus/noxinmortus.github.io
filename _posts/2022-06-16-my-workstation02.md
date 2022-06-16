---
layout: post
title: My Workstation 2022-06-16
categories: Workstation
date: 2022-06-16
---

Hello,

A l'heure où j'écris ces lignes, je travaille désormais pour une association ! Mon travail est réalisé quasi exclusivement sur un PC portable (que j'ai choisi !). J'ai pris un certain nombre de notes sur sa configuration que je vais essayer de synthétiser ici.

## Hardware
Il s'agit d'un HP EliteBook Folio 1040 G3 de 2016 acheté d'occasion chez [Ecodair](https://www.ordinateur-occasion.com/) pour la modique somme de 390€ TTC, dont voici les caractéristiques principales:
- Memory: 8 GB DDR4 2133 MHZ
- CPU: Intel Core i5-6300U Skylake 2.40GHz
- Graphics: Intel Skylake GT2 [HD Graphics 520]
- Disk: Toshiba HG6 Series SSD M2 240 Go

### Fingerprint Reader
Petite note sur le lecteur d'empreinte digitale:
```bash
$ lsusb |grep -i fingerprint
Bus 001 Device 003: ID 138a:003f Validity Sensors, Inc. VFS495 Fingerprint Reader
```

Pas moyen de le faire fonctionner, la lib [libfprint](https://gitlab.freedesktop.org/libfprint/libfprint) qui gère ça [n'aura pas les drivers propriétaires](https://bugs.freedesktop.org/show_bug.cgi?id=97346).

J'ai également essayé le projet [python-validity](https://github.com/uunicorn/python-validity) [sans succès](https://github.com/uunicorn/python-validity/issues/117)...

Il semble possible de build libfprint avec les drivers propriétaires de Validity, voir [ici](https://github.com/Gnate/libfprint/tree/master/libfprint/drivers/validity)

Mais après avoir lu [ceci](https://forum.kde.org/viewtopic.php?f=309&t=174078&p=452834&hilit=fingerprint#p452834), je doute vraiment sur le gain de temps et de sécurité quant à l'utilisateur du fingerprint reader... Peut-être lorsque le sujet sera beaucoup plus mûr...

## BIOS
Dans le BIOS j'ai réalisé les opérations suivantes:
- Mise à jour du BIOS en dernière version
- Activation de [BIOS SureStart](https://www.hp.com/us-en/hp-news/blog/enterprise-computing/office-of-the-future.html)
- Activation de [Intel Software Guard extensions (Intel SGX)](https://phoenixnap.com/kb/intel-sgx)
- Activation de [VTx (Virtualization Technology VTx)](https://en.wikipedia.org/wiki/X86_virtualization)
- Activation de [VT-d](https://en.wikipedia.org/wiki/X86_virtualization#I/O_MMU_virtualization_(AMD-Vi_and_Intel_VT-d))
- Activation de [Intel Trusted Execution Technology (Intel TXT)](https://www.intel.com/content/www/us/en/support/articles/000025873/technologies.html)

# Installation sous KDE Neon 20.04
Concernant le système d'exploitation, je suis resté sur KDE Neon 20.04 avec l'intégralité des filesystems (boot, partition root et swap) chiffrés avec LUKS, et avec du EXT4, rien d'extraordinaire. Il faut vraiment que j'essaye BTRFS la prochaine fois...
