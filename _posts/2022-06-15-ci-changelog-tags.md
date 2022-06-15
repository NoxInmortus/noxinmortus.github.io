---
layout: post
title: Tags et Changelog avec Gitlab-CI
categories: Divers
date: 2022-06-15
---

Hello,

Pour le contexte, il y a quelque temps,  je travaillais dans une certaine entreprise. Nous disposions d'une instance Gitlab sur laquelle était hébergée un certain nombre de roles Ansible. Leur maintenance était assez anarchique, et il était difficile de suivre l'évolution des rôles autrement qu'en consultant la liste des commits de chaque rôle, plus ou moins régulièrement.

Certains ne s'en embarrassaient pas et réutilisaient les mêmes tags de chaque rôle pour chaque nouveau projet. Pour ma part, cela n'était pas envisageable, et ce, pour plusieurs raisons :
- Au fil du temps, nos compétences, connaissances et besoins évoluent, et il est nécessaire d'adapter nos rôles en conséquence.
- Ce qui n'était pas géré par les rôles était donc géré par des tasks supplémentaires "standalone". Lorsqu'un rôle finit par pouvoir gérer ce besoin, il me semble plus propre d'abandonner les tasks "standalone" (débattable sans doute).
- Un rôle opérationnel à un instant T ne le sera pas forcément après un saut dans le temps si l'orchestrateur est différent et/ou si les cibles différentes. Il faut alors également adapter le rôle en conséquence.

À l'époque, j'avais proposé à ce que le nécessaire soit fait afin de pouvoir envoyer un récapitulatif hebdomadaire par mail des changements. Je ne suis plus dans cette entreprise, mais l'idée a eu le temps de murir, et j'ai fini par avoir un système d'intégration continue sur mon instance Gitlab personnelle qui répond à ce besoin, celui de réaliser un changelog à peu près propre. J'y ai par ailleurs ajouté la création automatique de tags, ce qui n'était pas forcément si simple à réaliser correctement. J'écris donc ce billet afin en tant que note technique de cette solution, je commence déjà à oublier... À noter que je ne suis ni expert Gitlab ni CI/CD de ce fait ce que j'écris plus bas peut très certainement être amélioré.

On va partir du principe qu'on dispose d'un projet git sur lequel on veut avoir une génération de changelog et de tags automatisés.

## Gitlab pré-requis
Sur notre projet, dans l'interface de Gitlab, on accède au menu `Settings > Access Token`. On génère ici un Token avec le droit `write_repository`, et on lui affecte le rôle désiré. Selon le rôle, les permissions ne seront pas les mêmes. Pour notre besoin, le rôle `Maintainer` par defaut est nécessaire pour push sur la branche Master/Main. Voir [ici](https://docs.gitlab.com/ee/user/permissions.html#gitlab-cicd-permissions) pour plus de détails. Et on enregistre le Token pour la suite.

Une fois notre token obtenu, on se rends dans Settings > CI/CD de notre groupe, et on va créer deux variables:
- `GITLAB_CI_USER`, dont la valeur sera un utilisateur que vous avez créé à l'avance pour cet usage. Par exemple j'ai créé un utilisateur `bot.gitlab-ci` et de type [external](https://docs.gitlab.com/ee/user/permissions.html#external-users). pour limiter ses accès par défaut.
- `GITLAB_CI_TOKEN`, dont la valeur sera le Token généré juste précédemment. On cochera les cases `Protect variable` et `Mask variable` pour l'imiter l'usage et l'affichage du Token.


## Tags
L'intérêt des tags est de pouvoir créer une version de notre projet à un instant T. Combiné au changelog, on va pouvoir facilement identifier une version du projet et les changements entre deux versions.

Sans plus attendre, voici son  `.gitlab-ci.yml`:

```yaml
---
stages:
  - tag
create_tag:
  stage: tag
  tags:
    - ansible-ci
  script:
    # Define TAG_DEFAULT_VERSION in case where there is no previous tag
    - TAG_DEFAULT_VERSION=1.0.0
    # Get latest tag
    - git pull origin master --tags
    # Get latest existing tag as TAG_LATEST_VERSION variable
    - TAG_LATEST_VERSION=$(git describe --tags --abbrev=0 `git rev-list --tags --max-count=1` || exit 0)
    # Set VERSION_INCREMENTER_ARG
    - if [[ "$(echo ${CI_COMMIT_MESSAGE} | grep -Ei 'major')" ]]; then VERSION_INCREMENTER_ARG=major;  elif [[ "$(echo ${CI_COMMIT_MESSAGE} | grep -Ei 'added|feat|change|remove')" ]]; then VERSION_INCREMENTER_ARG=feature; else VERSION_INCREMENTER_ARG=bug; fi
    # Set TAG_NEW_VERSION with version-incrementer.sh using TAG_LATEST_VERSION or if does not exist, use TAG_DEFAULT_VERSION
    - TAG_NEW_VERSION="$(version-incrementer.sh ${TAG_LATEST_VERSION:-${TAG_DEFAULT_VERSION}} ${VERSION_INCREMENTER_ARG})"
    # Create the new tag and push it
    - git tag "${TAG_NEW_VERSION}"
    - git push "https://${GITLAB_CI_USER}:${GITLAB_CI_TOKEN}@${CI_REPOSITORY_URL#*@}" "${TAG_NEW_VERSION}"
  only:
    refs:
      - master
```

Vous pouvez remarquer la définition d'une variable `VERSION_INCREMENTER_ARG` et l'appel au script `version-incrementer.sh` que voici (ceci est un fork, et je n'ai pas retrouvé la source de l'original):

```bash
#!/usr/bin/env bash
set -euo pipefail

version="${1}"
major=0
minor=0
bugfix=0

# break down the version number into it's components
regex="([0-9]+).([0-9]+).([0-9]+)"
if [[ ${version} =~ ${regex} ]]; then
  major="${BASH_REMATCH[1]}"
  minor="${BASH_REMATCH[2]}"
  bugfix="${BASH_REMATCH[3]}"
fi

# check parameter to see which number to increment
if [[ "${2}" == "feature" ]]; then
  minor=$(echo "${minor}" + 1 | bc)
elif [[ "${2}" == "bug" ]]; then
  bugfix=$(echo "${build}" + 1 | bc)
elif [[ "${2}" == "major" ]]; then
  major=$(echo "${major}" + 1 | bc)
else
  echo "usage: ./${0}.sh [version-string] [major/feature/bug]"
  exit 0
fi

# new version number
echo "${major}.${minor}.${bugfix}"
```

Le système procède dans cet ordre :
- On définit une version par défaut dans le cas où aucun tag précédent n'existe (ici `1.0.0`, mais vous pouvez définir ce que vous voulez)
- On récupère les tags existants et plus particulièrement le dernier, si aucun n'existe alors on utilisera la version par défaut pour la valeur de la variable `TAG_LATEST_VERSION`
- Ensuite, on va filtrer le message de commit pour rechercher un string particulier pour l'utiliser dans notre variable `VERSION_INCREMENTER_ARG`, soit `major`, soit `added` / `feat` / `change` / `remove` qu'on remplacera par `feature` dans notre variable. Si rien n'est identifié, alors à défaut on aura `bug` comme valeur.
- On appelle notre script `version-incrementer.sh` avec comme argument `TAG_LATEST_VERSION` et `VERSION_INCREMENTER_ARG`
- Dans notre script, on va décomposer `TAG_LATEST_VERSION` en trois parties (version `major`, version `minor` et version `bugfix`), et identifier via la valeur de `VERSION_INCREMENTER_ARG` qu'est-ce qu'on souhaite incrémenter, puis on envoie en sortie de script la nouvelle version.
- On crée alors le nouveau tag, et on push

Mon cas est particulièrement simple, n'ayant pas des gros besoins, mais l'ensemble est relativement KISS et facilement adaptable.

## CHANGELOG
### Conventional Commits
La génération du changelog va se faire grâce aux descriptions de nos commits git. Et pour cela on va suivre une certaine convention pour écrire les message de commits, j'essaye de suivre [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/#specification).

Exemple de ce que cela peut donner :
```
Fixed: Bad variable substitution resulting in bug referenced in issue #4
Changed: python3 package instead of python2 for python dependencies
Added: psmisc package to pkg_installation variable
Removed: workstation.bashrc file had nothing to do here
```

### Choisir son générateur de CHANGELOG
On pourrait certainement se débrouiller juste avec git et bash, mais avoir un petit binaire qui gère cette partie serait un vrai gain de temps.

Pour ma part j'ai choisi [git-chglog](https://github.com/git-chglog/git-chglog>), il s'agit tout simplement d'un binaire à placer quelque part dans le `PATH` de notre `gitlab-runner`.

### Intégration continue sur notre projet
Toujours dans notre projet git, on va créer le fichier `.gitlab-ci.yml` qui va contenir notre tâche de génération du changelog :

```yaml
---
stages:
  - changelog

create_changelog:
  stage: changelog
  tags:
    - ansible-ci
  script:
    - RANDOMSTRING=$(openssl rand -base64 20)
    # gitlab-runner runs on a detached HEAD, checkout a new branch
    - git checkout -b CHANGELOG-${RANDOMSTRING}
    # Generate changelog file
    - git-chglog -o CHANGELOG.md
    - git add CHANGELOG.md
    - git commit -m '[skip-ci] Auto-update CHANGELOG.md'
    # Set remote push URL and push to originating branch
    - git remote set-url --push origin "https://${GITLAB_CI_USER}:${GITLAB_CI_TOKEN}@${CI_REPOSITORY_URL#*@}"
    - git push origin CHANGELOG-${RANDOMSTRING}:${CI_COMMIT_REF_NAME}
  only:
    refs:
      - master
```

Ici avec `git-chglog`, il nous faut un dossier à la racine du projet nommé `.chglog`. Celui contient un fichier de template et à un autre de configuration, qui vont être utilisés par `git-chglog`. Deux exemples (sinon RTFM):
  - <https://github.com/git-chglog/git-chglog/tree/master/.chglog>
  - <https://git.tools01.noxinmortus.fr/sysadmins/ansible/role-zabbix/-/tree/master/.chglog>

Il ne nous reste plus qu'a combiner nos CI de création de tags avec celle-ci, et voilà.

A+
