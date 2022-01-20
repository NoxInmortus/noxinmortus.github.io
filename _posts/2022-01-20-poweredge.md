---
layout: post
title: Dell Poweredge T110 II
categories: Hardware
date: 2022-01-20
---

Hello,

J'ai ramené dernièrement une de mes machines qui trainait depuis longtemps dans un garage, un Dell Poweredge T110 II, un serveur en format tour de 2013: [SpecSheet](https://i.dell.com/sites/csdocuments/Shared-Content_data-Sheets_Documents/en/Dell-PowerEdge-T110II-SpecSheet.pdf).

Il s'agit d'un serveur doté d'un Intel Xeon processor E3-1200 (une version serveur du processeur Intel Core i3), pouvant aller jusqu'à 32GB de DDR3 ECC. J'ai à ma disposition une barrette de 4GB ainsi que 2x8GB (qui coûtaient 126€ à l'achat, une fortune) pour un total de 20GB. J'ai également doté la machine d'un disque SSD de 60GB ainsi que de trois disques SATA de 1TB que j'ai pu récupérés sur d'anciens NAS dont je n'ai plus l'usage.

# Mises à jour BIOS + BMC

Avant toute chose, j'ai souhaité réaliser une mise à jour du BIOS et de la BMC, passant de la version 2.0.5 à 2.10.0 pour le BIOS, et de 1.90 à 1.95 pour la BMC.

- Version 2.10.0 BIOS : <https://www.dell.com/support/home/fr-fr/drivers/driversdetails?driverid=5r0t5&oscode=ws8r2&productcode=poweredge-t110-2>
- Version 1.95 BMC : <https://www.dell.com/support/home/fr-fr/drivers/driversdetails?driverid=414g9>

Pour le BIOS, Dell ne fourni pas de binaire pour Linux, alors je suis parti pour faire la mise à jour avec le binaire DOS. Malheureusement impossible de faire fonctionner quoi que ce soit avec FreeDOS que j'utilise d'ordinaire, on obtient une erreur en boucle (que je n'ai plus sous la main) lorsqu'on lance la mise à jour. Vu que Dell met à disposition un binaire pour Windows pour BIOS et BMC, pour l'occasion j'ai testé un live-CD Windows 10 modifié et adapté a du diagnostic (<https://www.malekal.com/malekal-live-cd-reparer-depanner-pc-windows>), et j'ai pu réaliser la mise à jour sans plus de problèmes.

Pour l'occasion j'aurais appris la commande PowerShell `Get-FileHash` pour récupérer le hash d'un fichier.

# fwupd

J'ai plus ou moins par hasard découvert fwupd (<https://github.com/fwupd/fwupd>) en parralèle, qui est un outil qui permet de mettre à jour le firmware des composants matériels supportés.
Hélas, rien d'intérressant pour notre Poweredge T110 II !

# Wake-on-LAN

Pour avoir la possibilitée de démarrer le T110 II depuis n'importe quel hôte sur mon réseau, comme je le fais pour toutes mes machines physiques qui peuvent l'être, j'ai activé le Wake-On-Lan. Pour ce faire, il a tout d'abord fallu faire accéder au BIOS, se rendre sur `Integrated Devices`, Selectionner notre carte réseau (Embedded NIC1) puis Modifier l'entrée pour être sur `Enabled with PXE`. Save & Quit & Reboot.

Au démarrage du serveur on peut désormais accèder à un nouveau menu avec CTRL+S, le menu de configuration de notre carte réseau. On change ici l'entrée `Pre-boot Wake On Lan` à `enable`.

On peut ensuite redémarrer le serveur, et vérifier depuis l'OS que le Wake-On-LAN est bien actif. Si tout est bon, le packet `wakeonlan` peut être installé sur un autre périphérique, et suffit de lancer `wakeonlan [addresse-mac-du-server]` qui va broadcaster une requête UDP sur le port 9, et le serveur démarre.

Quelques liens qui m'ont servis :
- <https://www.dell.com/community/Mat%C3%A9riel-PowerEdge/wake-on-lan-DELL-poweredge-T110-II/td-p/5368082>
- <https://www.dell.com/community/Desktops-General-Read-Only/wake-on-lan-on-PowerEdge-T110/m-p/3498033>
- <https://wiki.archlinux.org/index.php/Wake-on-LAN#Enable_WoL_on_the_network_adapter>

# BMC / IPMI

BMC, ou Baseboard Management Controller, composant intégré à la carte mère, autonome et permet d'effectuer certaines actions même lorsque que le serveur lui-même est éteint.

Au démarrage, il faudra réaliser la combinaison CTRL+E pour rentrer dans le menu de configuration, avant que le système d'exploitation ne s'initialise. Une fois entré, il nous faut activer l'accès LAN, vérifier que l'on a les privilèges Admin avec l'accès distant, et configurer l'utilisateur/mot de passe d'accès. On sauvegarde, et on redémarre.

En parrallèle, on peut tester l'accès distant avec l'utilitaire `ipmitool`. Pour cela il nous faudra autoriser la sortie sur le port 623 en UDP. On test ensuite l'accès au shell ipmi :

```bash
ipmitool -H 192.168.x.x -U root -P root shell
ipmitool> power status
Chassis Power is on
```

Certains matériels requierent l'utiliser de la norme IPMI v2.0, pour cela il suffit de rajouter `-I lanplus` : `ipmitool -H 192.168.5.211 -U root -P root -C3 -I lanplus`

On peut également avoir besoin d'accéder au terminal KVM, mais je n'ai pas poussé son utilisation, celle-ci semble difficile... : `ipmitool -H 192.168.5.211 -U root -P root -C3 -I lanplus sol activate`

Une fois sur le shell IPMI, quelques commandes intérressantes :
```bash
ipmitool> power
chassis power Commands: status, on, off, cycle, reset, diag, soft

ipmitool> chassis
Chassis Commands:  status, power, identify, policy, restart_cause, poh, bootdev, bootparam, selftest

ipmitool> lan print
Set in Progress         : Set Complete
Auth Type Support       : NONE MD2 MD5 PASSWORD
Auth Type Enable        : Callback : MD2 MD5
                        : User     : MD2 MD5
                        : Operator : MD2 MD5
                        : Admin    : MD2 MD5
                        : OEM      :
IP Address Source       : DHCP Address
IP Address              : 192.168.x.x
Subnet Mask             : 255.255.255.0
MAC Address             : dd:dd:dd:dd:dd:dd
SNMP Community String   : public
IP Header               : TTL=0x40 Flags=0x40 Precedence=0x00 TOS=0x10
Default Gateway IP      : 192.168.x.x
Default Gateway MAC     : 00:00:00:00:00:00
Backup Gateway IP       : 0.0.0.0
Backup Gateway MAC      : 00:00:00:00:00:00
802.1q VLAN ID          : Disabled
802.1q VLAN Priority    : 0
RMCP+ Cipher Suites     : 0,1,2,3,4,5,6,7,8,9,10,11,12,13
Cipher Suite Priv Max   : aaaaaaaaaaaaaaX
                        :     X=Cipher Suite Unused
                        :     c=CALLBACK
                        :     u=USER
                        :     o=OPERATOR
                        :     a=ADMIN
                        :     O=OEM
Bad Password Threshold  : Not Available

ipmitool> lan
LAN Commands:
                   print [<channel number>]
                   set <channel number> <command> <parameter>
                   alert print <channel number> <alert destination>
                   alert set <channel number> <alert destination> <command> <parameter>
                   stats get [<channel number>]
                   stats clear [<channel number>]


ipmitool> delloem mac
System LOMs
NIC Number      MAC Address             Status
0               dd:dd:dd:dd:dd:dd       Enabled
BMC MAC Address dd:dd:dd:dd:dd:dd
```

Ce faisant je me rends compte qu'il n'est pas indispensable de configurer l'IPMI lors du démarrage du serveur, on aurait tout à fait pu le faire depuis le système d'exploitation.

Lors de mes recherches je suis également tombé sur ce dépôt non-officiel pour OpenManage sous Linux, a voir ultérieurement : <http://linux.dell.com/repo/community/ubuntu/>

# Carte RAID

Avec ce Poweredge, je dispose d'une carte Raid LSI `SAS2008 PCI-Express Fusion-MPT SAS-2 [Falcon]`. Pour l'identifier on peut utiliser `lspci` ou `lshw` :
```bash
lspci -nn|grep -i sas
01:00.0 Serial Attached SCSI controller [0107]: LSI Logic / Symbios Logic SAS2008 PCI-Express Fusion-MPT SAS-2 [Falcon] [1000:0072] (rev 03)

lshw |grep -i sas -B 5 -A 5
*-sas
     description: Serial Attached SCSI controller
     product: SAS2008 PCI-Express Fusion-MPT SAS-2 [Falcon]
     vendor: LSI Logic / Symbios Logic
     physical id: 0
     bus info: pci@0000:01:00.0
     logical name: scsi0
     version: 03
     width: 64 bits
     clock: 33MHz
     capabilities: sas pm pciexpress vpd msi msix bus_master cap_list
     configuration: driver=mpt3sas latency=0
     resources: irq:16 ioport:2000(size=256) memory:c1240000-c124ffff memory:c1200000-c123ffff
```

Une fois qu'on a identifié notre matériel, on va essayer d'en savoir un peu plus. Je ne suis pas arrivé à obtenir quoi que ce soit avec MegaCli, pour finalement trouver l'utilitaire `sas2ircu` qui permet de gérer ce type de carte Raid. Les détails peuvent se trouver ici :
- <https://hwraid.le-vert.net/wiki/LSIFusionMPTSAS2>
- <https://hwraid.le-vert.net/wiki/DebianPackages>
- <https://usermanual.wiki/Document/SAS2IRCUUserGuide.3989751573/html>
- <https://docs.broadcom.com/doc/12353380>

Une fois l'utilitaire installé, voici ce qu'on peut obtenir :
```bash
sas2ircu list
LSI Corporation SAS2 IR Configuration Utility.
Version 16.00.00.00 (2013.03.01)
Copyright (c) 2009-2013 LSI Corporation. All rights reserved.


         Adapter      Vendor  Device                       SubSys  SubSys
 Index    Type          ID      ID    Pci Address          Ven ID  Dev ID
 -----  ------------  ------  ------  -----------------    ------  ------
   0     SAS2008     1000h    72h   00h:01h:00h:00h      1028h   1f1ch
   

sas2ircu 0 display
LSI Corporation SAS2 IR Configuration Utility.
Version 16.00.00.00 (2013.03.01)
Copyright (c) 2009-2013 LSI Corporation. All rights reserved.

Read configuration has been initiated for controller 0
------------------------------------------------------------------------
Controller information
------------------------------------------------------------------------
  Controller type                         : SAS2008
  BIOS version                            : 7.11.10.00
  Firmware version                        : 7.15.08.00
  Channel description                     : 1 Serial Attached SCSI
  Initiator ID                            : 0
  Maximum physical devices                : 255
  Concurrent commands supported           : 1720
  Slot                                    : Unknown
  Segment                                 : 0
  Bus                                     : 1
  Device                                  : 0
  Function                                : 0
  RAID Support                            : Yes
```

Au vu du fait que je dispose de trois disques de 1TB pour ce projet, je souhaiterais faire du RAID1E:
```
RAID 1E is a type of nested RAID level that utilizes two-way mirroring on a minimum of two disks. It is similar to RAID level 1 but extends on its capabilities by supporting more physical disks.
RAID 1E is also known as striped mirroring, enhanced mirroring and hybrid mirroring.
Techopedia explains RAID 1E
RAID level 1E primarily combines drive mirroring and data striping capabilities within one level. The data is striped across all the drives within the array. It provides better drive redundancy and performance than RAID level 1.
RAID 1E requires a minimum of three drives to be constructed and can support up to 16 drives. RAID 1E only allows half of the capacity of the array to be used. If any of the drives fail, the read/write operations are transferred to other operational drives within the array.
```

Malheureusement, en plus d'avoir un firmware un peu viellot, il s'agit de la variante '2118 IT', qui ne permet pas de faire du RAID1E. On va donc essayer de flasher la carte raid pour avoir le firmware '2118 IR', dans sa dernière version.
En bref, j'ai perdu beaucoup de temps à comprendre comment faire ça correctement, j'ai par exemple effacé le firmware existant sans noter le SAS ID nécessaire, et il m'a fallu beaucoup de recherche pour comprendre que celui-ci était constitué de 16 chiffres hexadécimaux plus ou moins aléatoires, et que la perte de l'ID initial était donc sans réelle conséquence. Cependant, une fois qu'on a récupéré les bons fichiers pour flasher (je n'ai pas retrouvé les différentes sources que j'ai utilisé navré), démarré sur une clef usb sous DOS, le process est relativement simple : 
```bash
megarec -writesbr 0 sbrempty.bin
megarec -cleanflash 0
reboot
sas2flsh -o -f 6gbpsas.fw
reboot
sas2flsh -o -f 2118ir.bin -b mptsas2.rom
sas2flash -c 0 -o -sasadd 500605b00544bd30
```

J'ai cependant gardé les sources suivantes pour m'aider à flasher la carte :
- <https://www.truenas.com/community/resources/detailed-newcomers-guide-to-crossflashing-lsi-9211-9300-9305-9311-9400-94xx-hba-and-variants.54/>
- <https://www.vladan.fr/flash-dell-perc-h310-with-it-firmware/>
- <https://wiki.unraid.net/index.php/Crossflashing_Controllers#General_instructions_for_flashing>

# Créer le RAID1E

Enfin bref, une fois notre carte raid opérationnelle, on peut créer le RAID1E :
```bash
sas2ircu 0 create RAID1E MAX 1:0 1:1 1:2 hardraid3x1TB noprompt
```

Puis on regarde un peu ce qu'il en est :
```bash
sas2ircu 0 display
LSI Corporation SAS2 IR Configuration Utility.
Version 16.00.00.00 (2013.03.01)
Copyright (c) 2009-2013 LSI Corporation. All rights reserved.

Read configuration has been initiated for controller 0
------------------------------------------------------------------------
Controller information
------------------------------------------------------------------------
  Controller type                         : SAS2008
  BIOS version                            : 0.00.00.00
  Firmware version                        : 20.00.07.00
  Channel description                     : 1 Serial Attached SCSI
  Initiator ID                            : 0
  Maximum physical devices                : 255
  Concurrent commands supported           : 1720
  Slot                                    : Unknown
  Segment                                 : 0
  Bus                                     : 1
  Device                                  : 0
  Function                                : 0
  RAID Support                            : Yes
------------------------------------------------------------------------
IR Volume information
------------------------------------------------------------------------
IR volume 1
  Volume ID                               : 286
  Volume Name                             : hardraid3x1TB
  Status of volume                        : Okay (OKY)
  Volume wwid                             : 0beca8bce30ba982
  RAID level                              : RAID1E
  Size (in MB)                            : 1429080
  Physical hard disks                     :
  PHY[0] Enclosure#/Slot#                 : 1:4
  PHY[1] Enclosure#/Slot#                 : 1:5
  PHY[2] Enclosure#/Slot#                 : 1:6
------------------------------------------------------------------------
Physical device information
------------------------------------------------------------------------
Initiator at ID #0

Device is a Hard disk
  Enclosure #                             : 1
  Slot #                                  : 4
  SAS Address                             : 4433221-1-0700-0000
  State                                   : Optimal (OPT)
  Size (in MB)/(in sectors)               : 953869/1953525167
  Manufacturer                            : ATA
  Model Number                            : WDC WD10EZEX-00W
  Firmware Revision                       : 1A01
  Serial No                               : WDWMC6Y0N24579
  GUID                                    : 50014ee00456b628
  Protocol                                : SATA
  Drive Type                              : SATA_HDD

Device is a Hard disk
  Enclosure #                             : 1
  Slot #                                  : 5
  SAS Address                             : 4433221-1-0600-0000
  State                                   : Optimal (OPT)
  Size (in MB)/(in sectors)               : 953869/1953525167
  Manufacturer                            : ATA
  Model Number                            : WDC WD10EZEX-00W
  Firmware Revision                       : 1A01
  Serial No                               : WDWMC6Y0NAKX34
  GUID                                    : 50014ee00456b4c8
  Protocol                                : SATA
  Drive Type                              : SATA_HDD

Device is a Hard disk
  Enclosure #                             : 1
  Slot #                                  : 6
  SAS Address                             : 4433221-1-0500-0000
  State                                   : Optimal (OPT)
  Size (in MB)/(in sectors)               : 953868/1953523054
  Manufacturer                            : ATA
  Model Number                            : WDC WD10EZEX-08W
  Firmware Revision                       : 1A02
  Serial No                               : WDWCC6Y6AH22KR
  GUID                                    : 50014ee2ba4b6c61
  Protocol                                : SATA
  Drive Type                              : SATA_HDD
------------------------------------------------------------------------
Enclosure information
------------------------------------------------------------------------
  Enclosure#                              : 1
  Logical ID                              : 500605b0:0544bd30
  Numslots                                : 8
  StartSlot                               : 0
------------------------------------------------------------------------
SAS2IRCU: Command DISPLAY Completed Successfully.
SAS2IRCU: Utility Completed Successfully.
```

On constate également qu'on peut récupérer quelques informations avec smartctl :
```
root@poweredge:~ # smartctl -a /dev/sg0
smartctl 7.2 2020-12-30 r5155 [x86_64-linux-4.19.0-16-amd64] (local build)
Copyright (C) 2002-20, Bruce Allen, Christian Franke, www.smartmontools.org

=== START OF INFORMATION SECTION ===
Vendor:               LSI
Product:              Logical Volume
Revision:             3000
Compliance:           SPC-4
User Capacity:        1,498,498,990,080 bytes [1.49 TB]
Logical block size:   512 bytes
Physical block size:  4096 bytes
Logical Unit id:      0x600508e00000000082a90be3bca8ec0b
Serial number:        3809192322200059068
Device type:          disk
Local Time is:        Fri Dec 31 19:17:01 2021 CET
SMART support is:     Unavailable - device lacks SMART capability.

=== START OF READ SMART DATA SECTION ===
Current Drive Temperature:     0 C
Drive Trip Temperature:        0 C

Error Counter logging not supported

Device does not support Self Test logging
```

Et... Ce sera tout pour ces notes diverses.
