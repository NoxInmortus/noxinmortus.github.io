---
layout: post
title: Changer l'auteur de précédents commits
categories: divers
date: 2020-08-24
---

Hello,

Pour créer/maintenir ce blog tout en souhaitant protéger ma vie privée, il m'a fallu re-travailler l'historique de mes commits git. En effet, j'utilisais précédemment mon prénom voir mon nom complet ainsi qu'une addresse e-mail du type "prenom.nom" dans mes commits. Ce qui fait qu'en parcourant ce blog, on peut se rendre sur mon profil GitHub et ainsi voir d'anciens projets qui avaient un historique pas anonyme du tout (en plus de ne pas être forcément très propre).

J'ai passé un certain temps à essayer donc de nettoyer cela proprement. J'ai tout d'abord volontairement choisi de refaire complètement certains dépôts tellement ils étaient pourris et pour lesquels l'historique n'avait aucune valeur ajoutée.

Pour faire cela rapidement on peux faire comme ceci, ce qui a pour résulat de retrouver un dépôt avec un seul commit :
```
# Check out to a temporary branch:
git checkout --orphan TEMP_BRANCH

# Add all the files:
git add -A

# Commit the changes:
git commit -am "Initial commit"

# Delete the old branch:
git branch -D master

# Rename the temporary branch to master:
git branch -m master

# Finally, force update to our repository:
git push -f origin master
```

J'ai fini également par trouver une single-command qui permet de réinitialiser l'auteur de tous les commits précédents :
```
git rebase -i --root -x "git commit --amend --reset-author -CHEAD"

# et on force push sur master
git push -f origin master
```

Il me reste cependant un certain nombre de dépôts à nettoyer sur mon GitLab...

Sources :

- <https://blog.garamotte.net/posts/2018/07/08/fr-git-rework-commits-history.html>
- <https://medium.com/@catalinaturlea/clean-git-history-a-step-by-step-guide-eefc0ad8696d>
- <https://stackoverflow.com/questions/10911317/how-to-remove-the-first-commit-in-git>
- <https://stackoverflow.com/questions/5667884/how-to-squash-commits-in-git-after-they-have-been-pushed>

A+
