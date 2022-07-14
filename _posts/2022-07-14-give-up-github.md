---
layout: post
title: Give up Github
categories: Ethics
date: 2022-07-14
---

_Brief summary for english readers who lost themselves here:_ \
_Following the call to [Give Up GitHub: The Time Has Come!](https://sfconservancy.org/blog/2022/jun/30/give-up-github-launch/), I've decided to remove progressively my Github repositories, which were originally a mirror from my Gitlab instance._ \
_In consequence, my repositories will only be available on my [GitLab](https://git.tools01.noxinmortus.fr/public), which is not yet open to new users, but I'm thinking about enabling Gitlab/Github OAuth registration methods in the near future._

Hello,

Pour faire suite à l'article [Give Up GitHub: The Time Has Come!](https://sfconservancy.org/blog/2022/jun/30/give-up-github-launch/), et une de ses versions françaises [L’heure est venue d’abandonner GitHub !](https://lydra.fr/lheure-est-venue-dabandonner-github/), j'ai décidé de participer au mouvement.

Cela fait en effet longtemps que mon compte GitHub, et avec lui ce dilemme d'y rester ou non. Je suis profondémment pour l'open-source et le libre, et cette dissociation cognitive de rester sur GitHub ne pouvait plus durer.

## Overview
A titre personnel j'héberge très peu de code sur GitHub :

- Ce blog : <https://github.com/NoxInmortus/noxinmortus.github.io>
- Un ensemble de CI et de Dockerfile afin de réaliser un build multi-architecture de `pqchecker` : <https://github.com/NoxInmortus/docker-build-pqchecker>
- Un rôle ansible pour installer ISPConfig (qui n'est plus maintenu mais qui est le repository le plus consulté de mon profil): <https://github.com/NoxInmortus/role-ispconfig>
- Un rôle ansible pour déployer Zabbix : <https://github.com/NoxInmortus/role-zabbix>
- Un rôle ansible pour déployer Borg : <https://github.com/NoxInmortus/role-borgbackup>
- Une application en PHP permettant d'administrer les comptes d'un serveur privé World of Warcraft : <https://github.com/NoxInmortus/waccounts>
- Une image Docker trois-en-un du projet LDAP Tool Box : <https://github.com/NoxInmortus/docker-ldap-tool-box>
- Une image Docker BounCA : <https://github.com/NoxInmortus/docker-bounca>
- Une image Docker ManiaControl : <https://github.com/NoxInmortus/docker-maniacontrol>
- Une image Docker ManiaPlanet : <https://github.com/NoxInmortus/docker-maniaplanet>

Mis à part le blog, tous les autres sont des mirroirs de mon GitLab. Le seul projet utilisant les workflows Github est `docker-build-pqchecker`.

Ces mirroirs avaient étés mis en place initialement car je ne souhaitais pas partager le lien original de mon instance GitLab, et encore moins ouvrir l'authentification, mais souhaitais pouvoir recevoir des issues et pull requests.

## Next steps
Voici donc mon plan d'action :

- [x] (2022-07-14) Désactivation du mirroring des dépôts sur mon GitLab
- [x] (2022-07-14) Suppression des dépôts Github sans fork ni stars (je vais essayer d'indiquer l'URL GitLab du projet, la où j'ai pu partager dans le passé l'URL Github):
  - [x] NoxInmortus/role-borgbackup >> <https://git.tools01.noxinmortus.fr/sysadmins/ansible/role-borgbackup>
  - [x] NoxInmortus/waccounts >> <https://git.tools01.noxinmortus.fr/imperium/waccounts>
  - [x] NoxInmortus/role-zabbix >> <https://git.tools01.noxinmortus.fr/sysadmins/ansible/role-zabbix>
  - [x] NoxInmortus/docker-bounca >> <https://git.tools01.noxinmortus.fr/sysadmins/docker/docker-bounca>
  - [x] NoxInmortus/docker-maniacontrol >> <https://git.tools01.noxinmortus.fr/sysadmins/docker/docker-maniacontrol>
  - [x] NoxInmortus/docker-ldap-tool-box >> <https://git.tools01.noxinmortus.fr/sysadmins/docker/docker-ldap-tool-box>
- [ ] Suppression du code des dépôts Github avec fork et/ou stars avec une modification du README indiquant les raisons du changement et l'URL GitLab du projet
- [ ] Mise à jour des différents README afin de supprimer les URLs Github
- [ ] Remplacement du workflow Github par une CI gitlab du projet `docker-build-pqchecker`
- [ ] Migration du blog sur mon GitLab et création de sa CI de build
