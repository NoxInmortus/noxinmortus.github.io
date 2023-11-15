---
layout: post
title: My Workstation 2022-02-13
categories: Workstation
date: 2022-02-13
---

Hello,

A l'heure où j'écris ces lignes, je travaille dans l'entreprise A. Mon travail est réalisé quasi exclusivement sur un PC portable (que je n'ai pas choisi). J'ai pris un certain nombre de notes sur sa configuration, notes que je vais synthétiser ici.

# Hardware
Il s'agit d'un HP 250 G7 Notebook PC:
- Processor	Intel(R) Core(TM) i7-8565U CPU @ 1.80GHz, 1992 Mhz, 4 Core(s), 8 Logical Processor(s)
- 16 GB DDR4 2400 Mhz
- SSD NVMe 240 GB

Côté BIOS j'ai configuré les options suivantes:
- Use UEFI
- Enable SecureBoot
- Clear TPM chip (before using Bitlocker encryption)

# Première installation sous KDE Neon 20.04

J'ai initialement installé KDE Neon 20.04 lorsque j'ai récupéré la machine, et ai pu constaté ne pas disposer de driver wifi pour ma carte Realtek RTL8821CE, j'ai dû exécuter les instructions suivantes afin d'obtenir un wifi fonctionnel:

```bash
sudo apt-get install --reinstall git dkms build-essential linux-headers-$(uname -r)
git clone https://github.com/tomaspinho/rtl8821ce
cd rtl8821ce
chmod +x dkms-install.sh
chmod +x dkms-remove.sh
sudo ./dkms-install.sh
```

Puis ces instructions pour signer le driver afin que celui-ci soit accepté par SecureBoot:
```bash
kmodsign sha512 \
    /var/lib/shim-signed/mok/MOK.priv \
    /var/lib/shim-signed/mok/MOK.der \
    /usr/lib/modules/$(uname -r)/kernel/drivers/net/wireless/8821ce.ko
```

Source: <https://askubuntu.com/questions/1071299/how-to-install-wi-fi-driver-for-realtek-rtl8821ce-on-ubuntu-18-04>

Je ne disposais également pas de Bluetooth, mais pour ça, je n'ai jamais réussi à régler le problème...

# Deuxième installation sous Windows 10 Entreprise
Malgré quelques problèmes dûs à l'utilisation d'un système d'exploitation non-Windows dans un environnement professionel principalement Microsoft, le gain de performance et d'ergonomie reste excellent en utilisant une distribution Linux personnalisée. Cependant j'ai été contraint de ré-installer mon poste de travail sous Windows.

## Chiffrement Bitlocker
Si l'activation du chiffrement Bitlocker est relativement aisé (voir <https://docs.microsoft.com/en-us/windows/security/information-protection/bitlocker/bitlocker-device-encryption-overview-windows-10>), attention à bien garder de côté votre clef de déchiffrement de secours. De plus, je n'ai pas compris pourquoi il n'est pas possible de configurer un mot de passe lors de la configuration initiale. Par défaut, le disque est déchiffré automatiquement au démarrage, sans avoir besoin de réaliser une action manuelle. Mais quel intérêt de chiffrer son disque dans ce cas ? Uniquement dans le cas où le disque et seulement le disque est volé ? Moui, je ne suis vraiment pas convaincu, je trouve ça assez aberrant. On parle de Windows où tout est censé être à la portée du premier venu...

Pour ma part j'ai dû ouvrir GPEdit (Windows+R > gpedit.msc) puis:
- Computer Configuration > Administrative Templates > Windows Components > BitLocker Drive Encryption > Operating System Drives (Group Policy window)
- Edit Require Additional Authentication at Startup
- Select "Enabled" at the top of the window here. Then, click the box under "Configure TPM Startup PIN" and select the "Require Startup PIN With TPM" option. Click "OK" to save your changes.
- Open Command Prompt and execute `manage-bde -protectors -add c: -TPMAndPIN` and then enter your password

Source: <https://www.howtogeek.com/262720/how-to-enable-a-pre-boot-bitlocker-pin-on-windows/>

## Software

### Windows Store
Il y a encore un an je n'avais rien à faire du Windows Store et je le considérais comme un autre bloatware, et j'ai beaucoup lu sur ce sujet avant de décider de l'utiliser. On le sait tous, Windows est un gruyère plein de trous, cependant Microsoft à initié la construction d'un véritable système de sécurité via les [UWP Apps](https://docs.microsoft.com/en-us/windows/uwp/get-started/universal-application-platform-guide), et il est donc recommandé d'installer les applications de préférence via le Windows Store.

Actuellement j'ai pu installer les outils suivants (d'autres ont pu être ajoutés depuis) :
- Firefox (internet browser - open-source)
- Bitwarden (password manager - open-source)
- Draw.io (diagram maker - open-source)
- Image Viewer (image viewer - NOT open-source)
- Microsoft Remote Desktop (RDP client - NOT open-source)
- Skype (VoIP, instant messaging - NOT open-source)
- VLC (multimedia player - open-source)
- [Windows Terminal Preview](https://docs.microsoft.com/en-us/windows/terminal/get-started) (terminal emulator - NOT open-source)

### Autres outils
- DriversCloud (find latests drivers - NOT open-source)
- 7zip (unarchiver - open-source)
- KeePass (password manager - open-source)
- MicroSip (voip client - open-source)
- MobaXterm (SSH/Telnet/RDP/VNC/FTP/SFTP and more clients - NOT open-source)
- Nextcloud desktop (sync and manage files from Nextcloud  - open-source)
- Notepad++ (editor - open-source)
- OpenVPN client (vpn client - open-source)
- ProcessHacker (Better task manager - open-source)
- Sumo Lite (Software Update Monitor - NOT open-source)
- TeamViewer (remote access client - NOT open-source
- VSCodium (community-driven, freely-licensed binary distribution of Microsoft’s editor VSCode)
- VMWare vSphere Client (NOT open-source)
- Wireguard client (vpn client - open-source)

### Outils Windows
- Remote Server Administration Tools for Windows 10 <https://www.microsoft.com/en-us/download/details.aspx?id=45520>
- Windows Subsystem Linux 2 <https://docs.microsoft.com/en-us/windows/wsl/install-win10>
- Windows Package Manager <https://docs.microsoft.com/en-us/windows/package-manager/winget/>
- Windows SysInternals <https://docs.microsoft.com/fr-fr/sysinternals/>
- Windows PowerToys <https://github.com/microsoft/PowerToys>
- Microsoft Security Compliance Toolkit <https://www.microsoft.com/en-us/download/details.aspx?id=55319>

### Windows 10 customization tools
- <https://github.com/builtbybel/privatezilla> (careful to run before installing any Windows Store apps if you want to remove pre-installed apps)
- <https://github.com/hellzerg/optimizer>
- <https://github.com/Sycnex/Windows10Debloater>
- <https://github.com/AndyFul/ConfigureDefender>

### [Windows Subsystem Linux 2 (WSL 2)](https://docs.microsoft.com/en-us/windows/wsl/setup/environment)
Une importante partie de mes activités se font dans un shell, et pour ne pas gaspiller de ressources dans une machine virtuelle j'ai testé pour la première fois WSL2. Une fois passé les 36 mises à jours pour obtenir la dernière version de Windows 10 et executé `wsl --install -d Debian`, on se retrouve avec notre subsystem Linux prêt. A noter qu'à l'écriture de ce post, on tombe sur Debian oldstable, et qu'il faut faire un petit coup d'upgrade pour être en Debian stable.

Pour pouvoir accéder à notre subsystem, on ouvre le [Windows Terminal](https://docs.microsoft.com/en-us/windows/wsl/setup/environment#set-up-windows-terminal), et dans le menu de selection on peut ouvrir un nouveau terminal sur notre subsystem. Remarque, dans les paramètres on peut pour se rendre la vie plus agréable:
- Dans le menu Startup : On configure notre subsystem Debian en `Default profile` (afin de lancer directement notre subsystem Debian)
- Dans le menu Profiles > Debian > Starting directory = "~" (afin de démarrer dans le home de son subsystem et non pas dans `/mnt/c/Users/$USER`)
- Dans le menu Profiles > Debian > Advanced > Bell notification style : Décocher audible (par défaut) pour ne plus avoir cette horrible cloche à chaque coup de tabulation...

Malgré donc ce WSL2 qui me permet de reproduire le même shell que sur une workstation sous Linux, il me manque un détail... L'interface dmenu que j'utilise pour Pass. Avec ça, en un raccourci clavier je peux accéder à mon pass-store et à ses secrets. Mais comment reproduire ça sous Windows et WSL2...

#### [Pass et dmenu](https://git.spartan.noxinmortus.fr/noxinmortus/dmenu-password-tool)
J'ai presque réussi à faire tomber le truc en marche (je me suis basé sur <https://medium.com/javarevisited/using-wsl-2-with-x-server-linux-on-windows-a372263533c3>):
- Installer VcXsrv (Serveur X pour Windows)
- Créer/modifier le raccourci pour VcXsrv afin d'avoir `Cible: "C:\Program Files\VcXsrv\xlaunch.exe" :0 -ac -terminate -lesspointer -multiwindow -clipboard -wgl -dpi auto`
- Copier le raccourci dans `C:\Users\$USER\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup`
- Dans notre subsystem Linux, installer un emulateur de terminal de son choix, `konsole` pour ma part
- On crée un petit script permettant de créer la variable DISPLAY (utilisée juste après), et on met ce script dans notre PATH:
```bash
#!/bin/bash
DISPLAY=$(cat /etc/resolv.conf | grep nameserver | awk '{print $2}'):0 konsole
```
- On ajoute dans son `~/.bashrc` (par exemple) notre raccourci (ici ALT+D):
```bash
## dmenu + pass (Alt+d is translated as \ed)
bind -x '"\ed":"/usr/local/bin/pass_dmenu.sh"'
```
- Dans les paramètres de Windows Terminal, on crée un nouveau profile (j'ai d'abord copié le profile Debian), nommé Konsole dans mon cas, afin d'exécuter notre script via WSL2 avec le paramètre `Ligne de commande: C:\Windows\System32\wsl.exe -d Debian display.sh`

Enfin, on lance notre nouveau profile "Konsole" depuis Windows Terminal, notre émulateur de terminal Konsole s'ouvre, on essaye le raccourci ALT+D... Et on a dmenu qui s'affiche avec nos secrets !

Il reste quelques détails à régler:
- Pour que Konsole soit à la même taille que Windows Terminal (ses valeurs par défaut en tous cas), il faut :
	- Se rendre dans Settings > Manage Profiles > General > Décocher `Remember window size` > apply
	- Se rendre dans Settings > Edit current profile > General > Initial terminal size > `155 columns` et `38 rows`
- Si problème de secrets convertis en qwerty, il manque le paquet `x11-xkb-utils`, ainsi que ce petit bout dans votre ~/.bashrc:
```bash
if [ -x /usr/bin/setxkbmap ]; then
  /usr/bin/setxkbmap fr
fi
```
- Quelques erreurs à l'ouverture de Konsole (pas encore trouvé de solution, cela ne semble pas poser problème pour autant):
```
QStandardPaths: XDG_RUNTIME_DIR not set, defaulting to '/tmp/runtime-${USER}'
libGL error: No matching fbConfigs or visuals found
libGL error: failed to load driver: swrast
```

Comme suggéré [ici](https://linuxfr.org/forums/linux-general/posts/pass-menu-without-gui-display), il devrait être possible de faire encore mieux avec [Rofi](https://github.com/davatorium/rofi), à voir ultérieurement.

#### Misc
Il existe une extension pour VSCode que je pense tester prochainement (à suivre): <https://docs.microsoft.com/en-us/windows/wsl/tutorials/wsl-vscode>

### Firefox Add-ons
- Auto-tab Discord : Reduce memory load when you have numerous open tabs
- Bitwarden : password manager
- Bypass Paywalls clean : Bypass paywalls of news sites
- ClearURLs : Remove tracking elements from URLs
- Consent-O-Matic : Automatic handling of GDPR consent forms
- Decentraleyes : Protect you against tracking through "free", centralized content delivery
- Disable Javascript : Adds the ability to disable JavaScript
- Enhanced GitHub : Display repo size, size of each file, download link and option to copy file contents
- Firefox Multi-Account Containers : Helps you keep all the parts of your online life contained in different tabs
- Flagfox : Display a flag depicting the location of the current server
- GitHub Dark Theme : A dark theme for all of GitHub
- HTTPS Everywhere : Automatically use HTTPS security when possible
- Privacy Badger : Automatically learns to block trackers
- Privacy Redirect : Redirect Twitter, YouTube, Instagram and more to privacy friendly alternatives
- Redirect AMP to HTML : Automatically redirect AMP pages to their canonical HTML equivalent
- Skip Redirect : Skip intermediary pages that some pages use before redirecting to a final pages
- SpanTree : Tree for Gitlab
- SponsorBlock : Skip sponsorships, subscriptions begging and more on YouTube videos
- uBlock Origin : Efficient pub blocker, easy on CPU & memory
- Undo Close Tab : Reopens the last closed tab
- User-Agent Switcher and Manager : Change User Agent whenever you want
- Zabbix Vue : Monitor Zabbix problems from your browser

### Notepad++ Plugins
- Auto Detect Indentation : Detects indention (tab or spaces) and auto adjust Tab key on-the-fly
- AutoEolFormat : A plugin to automatically set a document's EOL (End Of Line) format to your needs on loading, saving or renaming a document or activating its tab
- BetterMultiSelection : Provides better cursor movements when using multiple selections
- Compare : Shows the differences between 2 files (side by side)
- JSON Viewer : JSON viewer that displays the selected JSON string in a tree view
- Linter : Allows realtime code check against any checkstyle-compatible linter: jshint, eslint, jscs, phpcs, csslint, and many others
- RunMe : Execute the currently open file, based on its shell association
- Save as admin : Allows saving file as administrator with Windows UAC prompt
- Task list : Automatically scans the open document and adds all "TODO:*" items to your task list, a window pane docked on the right

### KeePass Plugins
- Enhance KeePass search functionality to search in all open databases <https://github.com/Rookiestyle/GlobalSearch>
- Adds multiple options to connect via RDP to the URL of an entry <https://github.com/iSnackyCracky/KeePassRDP>
- Adds connect capability to RDP/SSH/vSphere <https://github.com/cristianst85/QuickConnectPlugin>
- Adds "Password Quality" column in View > configure Columns <https://keepass.info/extensions/v2/qualitycolumn/QualityColumn-1.3.zip>
- Adds protection against accidental auto-typing of passwords into the wrong places <https://sourceforge.net/projects/checkpasswordbox/>
- Adds several KeePass 2.x dialogs resizable <https://keepass.info/extensions/v2/keeresize/>
- Provide an enhanced entry view <https://sourceforge.net/projects/kpenhentryview/>
- Provide bulk operations on fields <https://sourceforge.net/projects/kpfieldsadminconsole/>
- Count and show entries sharing a password <https://sourceforge.net/projects/keepasspasswordcounter/>

## Additional security stuff to read
- <https://docs.microsoft.com/en-us/windows-hardware/design/device-experiences/oem-secure-boot>
- <https://docs.microsoft.com/en-us/windows/security/information-protection/tpm/trusted-platform-module-overview>
- <https://docs.microsoft.com/en-us/microsoft-365/security/defender-endpoint/enable-controlled-folders?view=o365-worldwide>
- <https://docs.microsoft.com/en-us/windows/security/threat-protection/microsoft-defender-application-guard/install-md-app-guard>
- <https://wonderfall.space/windows-hardening/>
- <https://krebsonsecurity.com/2021/05/try-this-one-weird-trick-russian-hackers-hate/>

## Additional security stuff for entreprise to dig in
- <https://github.com/PaulSec/awesome-windows-domain-hardening>
- <https://github.com/0x6d69636b/windows_hardening>
- <https://github.com/S1ckB0y1337/Active-Directory-Exploitation-Cheat-Sheet>
- <https://github.com/beerisgood/Windows10_Hardening>
- <https://github.com/dev-sec/windows-baseline>
- <https://github.com/Sneakysecdoggo/Wynis>
- <https://github.com/securitywithoutborders/hardentools>
- <https://github.com/gentilkiwi/mimikatz/>
