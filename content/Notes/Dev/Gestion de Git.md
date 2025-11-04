---
title: Gestion de Git
tags:
  - git
  - dev
  - versioning
  - gitlab
  - github
description: Guide pratique pour cloner, travailler avec des branches, synchroniser un dépôt, gérer les tags, publier des releases, maintenir un changelog et utiliser Git LFS
---

## Objectif

Note de référence pour les usages courants de Git. Les exemples utilisent un dépôt avec un serveur distant nommé `origin` et une branche principale `main` ou `master` selon le contexte du projet.

> [!info] Raccourcis utiles
> `HEAD` désigne le dernier commit sur la branche courante. `origin/nom_branche` désigne la branche distante. `upstream` est souvent le dépôt source d'un fork.

---

## Configuration minimale

```bash
# Vérifier l'installation
git --version

# Identité
git config --global user.name "Nom Prénom"
git config --global user.email "email@example.com"

# Branche par défaut
git config --global init.defaultBranch main

# Style de pull
git config --global pull.rebase false   # merge
# git config --global pull.rebase true  # rebase
```

> [!info] Portée de la configuration
> `--global` applique la configuration à l'utilisateur. Sans `--global`, la configuration s'applique au dépôt courant dans `.git/config`.

---

## Cloner un dépôt

```bash
# Clone standard
git clone https://example.com/owner/repo.git

# Cloner une branche spécifique
git clone --branch dev https://example.com/owner/repo.git

# Clone léger
git clone --depth 1 https://example.com/owner/repo.git

# Inclure les sous-modules
git clone --recurse-submodules https://example.com/owner/repo.git
```

> [!warning] Clone léger
> Un clone avec `--depth 1` ne contient pas tout l'historique. Certaines opérations comme `git blame` ou des rebase complexes peuvent échouer.

---

## Cycle de base

```bash
# État du dépôt
git status

# Ajouter des fichiers
git add fichier.txt
git add .

# Committer
git commit -m "Message clair et concis"

# Modifier le dernier commit
git commit --amend

# Historique
git log --oneline --graph --decorate --all
```

> [!info] Messages de commit
> Un bon message résume l'intention. Utiliser l'impératif court. Exemple "Ajoute la commande X".

---

## Concepts clés

### Espace de travail et index

- Arbre de travail. fichiers présents dans le dossier du projet
- Index. zone de préparation qui liste ce qui sera inclus dans le prochain commit
- Commit. instantané de l'index avec un message et des métadonnées

```bash
git status           # vue d'ensemble
git diff             # différences non indexées
git diff --staged    # différences indexées
```

### HEAD et branches

- HEAD. pointeur vers le dernier commit de la branche actuelle
- Branche. pointeur qui avance avec les commits
- Detached HEAD. HEAD pointe directement sur un commit sans branche

```bash
git checkout <hash>      # detached HEAD
git switch -c fix/x      # crée une branche et rattache HEAD
```

### Distants et suivi

- origin. nom par défaut du dépôt distant principal
- upstream. dépôt source d'un fork si utilisé
- Branche de suivi. branche locale liée à une branche distante. exemple `dev` qui suit `origin/dev`

```bash
git remote -v
git branch -vv           # montre les branches locales et leur suivi
```

### Fetch et pull

- fetch. récupère les mises à jour du distant sans modifier la branche locale
- pull. équivaut à fetch puis merge ou rebase selon la configuration

```bash
git fetch origin
git pull --ff-only       # refuse si merge nécessaire
git pull --rebase        # rejoue les commits locaux après la mise à jour
```

### Merge et fast forward

- Merge. combine deux historiques
- Fast forward. avance simple du pointeur si aucun commit local ne diverge

```bash
git merge dev            # merge auto si fast forward possible
git merge --no-ff dev    # force un commit de merge
git merge --ff-only dev  # refuse si non fast forward
```

### Rebase

- Rebase. rejoue les commits de la branche courante par dessus une autre base
- Réécrit l'historique local. les identifiants de commit changent
- Utile pour obtenir une histoire linéaire et propre

```bash
git rebase dev           # place vos commits au sommet de dev
git rebase --abort       # annule le rebase en cours
git rebase --continue    # après résolution de conflits

# Nettoyer l'historique récent
git rebase -i HEAD~3     # reword, squash, fixup
```

> [!warning] Rebase de branches publiques
> Ne pas rebase une branche déjà partagée sans coordination. Préférer rebase avant le premier push ou sur des branches personnelles.

### Reset et revert

- reset. déplace la branche courante vers un commit antérieur
  - soft. garde index et fichiers
  - mixed. garde les fichiers. vide l'index
  - hard. remet index et fichiers à l'état du commit ciblé
- revert. crée un nouveau commit qui inverse un commit existant

```bash
git reset --soft <hash>
git reset --mixed <hash>
git reset --hard <hash>
git revert <hash>
```

> [!info] Choisir reset ou revert
> `reset` modifie l'histoire de la branche. `revert` garde l'histoire et ajoute un commit d'annulation. Pour une branche partagée, préférer `revert`.

### Cherry pick

Appliquer un commit particulier sur la branche courante sans fusionner tout le reste.

```bash
git cherry-pick <hash>
```

### Force push et sécurité

- push --force. écrase l'historique distant
- push --force-with-lease. refuse d'écraser si le distant a bougé depuis le dernier fetch

```bash
git push --force-with-lease
```

## Branches

```bash
# Créer et basculer
git switch -c feature/ma-fonction

# Basculer vers une branche existante
git switch dev

# Équivalent avec checkout
git checkout -b fix/bug-123
git checkout dev

# Suivre une branche distante
git switch -t origin/dev

# Renommer la branche courante
git branch -m nouveau_nom

# Supprimer une branche locale
git branch -d feature/ma-fonction      # refuse si non fusionnée
git branch -D feature/ma-fonction      # suppression forcée
```

> [!info] `switch` ou `checkout`
> `git switch` est plus simple pour les branches. `git checkout` reste utile pour d'autres opérations comme changer de commit ou de fichier.

---

## Distant et synchronisation

```bash
# Voir les distants
git remote -v

# Définir le distant origin si absent
git remote add origin https://example.com/owner/repo.git

# Récupérer sans fusion
git fetch origin

# Récupérer et fusionner
git pull              # merge par défaut selon la config
git pull --rebase     # rebase si souhaité

# Pousser une branche et définir l'upstream
git push -u origin dev

# Pousser des tags
git push origin --tags
```

> [!warning] Force push
> `git push --force` réécrit l'historique distant. Préférer `--force-with-lease` et réserver aux branches privées ou en coordination explicite.

---

## Revenir en arrière et corriger

```bash
# Voir les changements non indexés
git diff

# Annuler des changements non indexés vers l'état du HEAD
git restore fichier.txt

# Retirer un fichier de l'index
git restore --staged fichier.txt

# Revenir par commit de correction
git revert <hash_ou_tag>

# Déplacer la branche courante vers un commit
git reset --soft <hash>
git reset --mixed <hash>   # par défaut
git reset --hard <hash>

# Mettre de côté son travail
git stash push -m "travail en cours"
git stash list
git stash pop
```

> [!warning] `reset --hard`
> Efface les modifications locales non commitées. Vérifier avec `git status` et sauvegarder au besoin avec `git stash`.

---

## Fusion et rebase

```bash
# Fusionner dev dans la branche courante
git merge dev

# Rejouer la branche courante par dessus dev
git rebase dev

# Résoudre des conflits
git status
git add fichiers_concernés
git rebase --continue   # ou git merge --continue
```

> [!info] Choisir merge ou rebase
> Merge crée un commit de jonction quand il y a divergence. Rebase réécrit vos commits pour les placer après la base visée. Rebase donne une histoire linéaire. Merge conserve le contexte d'intégration. Aligner le choix sur les conventions de l'équipe.

### Cas pratiques

```bash
# Créer un commit de merge même si fast forward possible
git merge --no-ff dev

# Refuser un merge implicite et exiger un fast forward
git pull --ff-only

# Nettoyer et condenser l'historique local avant de pousser
git rebase -i origin/dev

# Fusion à plat. regroupe les changements sans garder l'historique de la branche feature
git merge --squash feature/x
git commit -m "Intègre feature/x"
```

> [!warning] Conflits
> Un conflit apparaît quand des lignes modifiées se chevauchent. Résoudre dans les fichiers puis marquer comme résolus avec `git add`. Continuer avec `--continue`.

> [!tip] Quand préférer rebase
> Avant le premier push d'une branche. Pour synchroniser une branche locale sur dev. Pour nettoyer des commits intermédiaires avec `rebase -i`.

---

## Tags

```bash
# Lister les tags
git tag
git tag -n             # avec message

# Créer un tag léger au HEAD
git tag v1.2.3

# Créer un tag annoté au HEAD
git tag -a v1.2.3 -m "Version 1.2.3"

# Tagger un commit spécifique
git tag -a v1.2.3 <hash_commit>

# Pousser un tag
git push origin v1.2.3

# Supprimer un tag local
git tag -d v1.2.3

# Supprimer un tag distant
git push --delete origin v1.2.3

# Vérifier les tags distants
git ls-remote --tags origin
```

> [!info] Tag léger ou annoté
> Un tag annoté stocke l'auteur, la date et un message. Recommandé pour les versions publiées. Un tag léger est un simple pointeur.

> [!tip] Renommer un tag
> Créer le nouveau tag sur le même commit puis supprimer l'ancien. Exemple
> `git tag v0.1.0 0.1^{}` puis `git tag -d 0.1` et `git push origin v0.1.0` suivi de `git push --delete origin 0.1`.

---

## Releases

Différence entre tag et release. Un tag identifie un commit. Une release ajoute un titre, une description, des ressources et un numéro de version sémantique.

### GitLab et GitHub interface

Étapes courantes

1. Créer et pousser un tag
2. Ouvrir la page Releases du projet
3. Créer une release depuis ce tag
4. Renseigner le titre, la description et le changelog
5. Ajouter des assets si nécessaire

> [!info] Sources zip et tar.gz
> Les archives source sont générées à la demande par la forge. Elles ne grossissent pas le dépôt Git. Les fichiers ajoutés comme assets téléversés sont stockés côté serveur et comptent dans les quotas du projet.

> [!info] Collecte de preuves GitLab
> La collecte de preuves enregistre un instantané des métadonnées de release. Option utile pour la traçabilité. Elle n'affecte pas le contenu du dépôt.

> [!warning] Visibilité des releases
> Si une release n'est pas visible en navigation privée, vérifier la visibilité du projet et du dépôt. Projet, dépôt et pages de releases doivent être publics. Vérifier aussi que le tag a été poussé.

---

## Changelog

Rôle. Documenter les changements par version et faciliter le suivi. Format recommandé Keep a Changelog et versions sémantiques SemVer.

> [!info] SemVer en bref
> MAJOR.minor.patch
> MAJOR. rupture de compatibilité
> minor. ajout rétrocompatible
> patch. correction sans changement d'API
> Le préfixe `v` dans les tags est courant et pratique

### Mise en place

Créer `CHANGELOG.md` à la racine avec une section Unreleased et une entrée par tag.

```markdown
# Changelog

Toutes les modifications notables sont documentées ici.

Format inspiré de Keep a Changelog
Versionnage suivant SemVer

## [Unreleased]
### Added
-
### Changed
-
### Fixed
-

## [v0.1.0] - 2024-10-01
### Added
- Première release officielle du projet

[Unreleased]: https://example.com/compare/v0.1.0...dev
[v0.1.0]: https://example.com/releases/tag/v0.1.0
```

> [!info] Liens de comparaison
> `Unreleased` peut comparer la branche de développement au dernier tag. Adapter l'URL à la forge utilisée. Sur GitLab motif `/-/compare/v0.1.0...dev`.

> [!tip] Release et changelog
> Le texte du changelog pour une version peut être copié dans la description de la release. Garder le fichier dans le dépôt pour l'historique.

---

## Git LFS et binaires

Git LFS gère les gros fichiers binaires. Utile pour données volumineuses ou exécutables. Alternative possible. Publier les binaires en assets de release plutôt que dans le dépôt si les fichiers changent rarement.

```bash
# Installation locale
git lfs install

# Traquer des motifs
git lfs track "*.zip"
git lfs track "bin/*"

# Le suivi crée ou modifie .gitattributes
git add .gitattributes
git commit -m "Active LFS pour les binaires"
```

> [!warning] Quotas et clones
> LFS consomme du stockage et de la bande passante dédiés. Vérifier les limites du serveur utilisé. Un clone sans LFS récupère des pointeurs et non les gros fichiers.

> [!tip] Choix pratique pour exécutables
> Pour des binaires stables et rarement mis à jour, préférer des assets de release par système d'exploitation. Le dépôt reste léger et l'installation côté utilisateur est simple.

---

## Référence rapide

```bash
# Cloner et créer une branche
git clone https://example.com/owner/repo.git
cd repo
git switch -c feature/x

# Cycle local
git add .
git commit -m "Implémente x"

# Synchroniser
git fetch origin
git pull --rebase
git push -u origin feature/x

# Tag et release
git tag -a v0.1.0 -m "v0.1.0"
git push origin v0.1.0

# Nettoyer
git branch -d feature/x
```

---

## Bonnes pratiques

- Unité de travail claire par branche
- Tag annoté pour les versions publiées
- Changelog mis à jour à chaque release
- Rebase avant push si l'équipe le demande
- Préférer des assets de release pour les gros binaires
- Messages de commit courts et explicites (en anglais si possible)
