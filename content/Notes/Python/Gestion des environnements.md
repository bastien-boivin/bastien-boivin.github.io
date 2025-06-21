---
title: Gestion des environnements Python
tags:
  - python
  - conda
  - mamba
  - environnements
description: Guide complet pour la gestion des environnements Python avec conda et mamba
---

## 🐍 Gestion des environnements Python

La gestion des environnements Python est cruciale pour maintenir des projets isolés et reproductibles. Plusieurs solutions existent : `venv`, `virtualenv`, `pipenv`, `poetry`, `conda`, `mamba`...

Dans cette note, je me concentre sur **conda** et **mamba** qui sont particulièrement adaptés pour mon utilisation (_datasciences_) et les projets nécessitant des dépendances complexes.

> [!info] Note complémentaire
> Pour la gestion des packages dans les environnements, voir : [[Packages Python]]

---

## 📦 Conda : Les fondamentaux

🔗 [Documentation officielle de conda](https://docs.conda.io/)

### 🧪 Gestion des environnements

#### Créer un environnement

```bash
# Environnement basique
conda create -n mon_env python=3.13.2

# Avec des packages spécifiques
conda create -n mon_env python=3.13.2 numpy pandas matplotlib

# Depuis un canal spécifique
conda create -n mon_env -c conda-forge python=3.13.2 geopandas

# Depuis un dossier spécifique
conda create -p ./envs/mon_env python=3.11

# Cloner un environnement existant
conda create -n clone_env --clone mon_env
````

#### Activer/Désactiver

```bash
# Activer
conda activate mon_env

# Désactiver
conda deactivate

# Lister les environnements
conda env list
# ou
conda info --envs

# Afficher l'environnement actuel
echo $CONDA_DEFAULT_ENV
```

#### Informations sur l'environnement

```bash
# Détails de l'environnement actif
conda info

# Packages installés dans l'environnement
conda list

# Historique des modifications
conda list --revisions

# Revenir à une révision précédente
conda install --revision 2
```

#### Supprimer un environnement

```bash
conda env remove -n mon_env
# ou
conda remove -n mon_env --all

# Supprimer un environnement par chemin
conda env remove -p ./envs/mon_env
```

### 📋 Environnements reproductibles

#### Créer depuis un fichier YAML

```bash
# Créer depuis environment.yml
conda env create -f environment.yml

# Spécifier un nom différent
conda env create -f environment.yml -n nouveau_nom

# Mettre à jour un environnement existant
conda env update -f environment.yml --prune
```

#### Structure d'un fichier environment.yml

```yaml
name: mon_projet
channels:
  - conda-forge
  - defaults
dependencies:
  - python=3.11
  - numpy>=1.24
  - pandas
  - matplotlib
  - geopandas
  - pip
  - pip:
    - requests
    - some-pip-only-package
variables:
  - VAR_NAME: value
prefix: /path/to/env  # optionnel
```

#### Exporter un environnement

```bash
# Export complet avec hash (reproductibilité stricte)
conda env export > environment.yml

# Export sans hash (multi-plateforme)
conda env export --no-builds > environment.yml

# Export historique des packages installés
conda env export --from-history > environment.yml

# Export avec nom personnalisé
conda env export -n mon_env > mon_projet.yml

# Export type pip
conda list --export > requirements.txt
```

> [!warning]- Hash et reproductibilité
> 
> - **Avec hash** : reproductibilité stricte, mais plateforme unique
> - **Sans hash** : plus souple, multi-plateforme
> - **--from-history** : liste uniquement les packages explicitement installés

### 🏷️ Gestion avancée

#### Renommer un environnement

```bash
# Pas de renommage direct : cloner puis supprimer l'ancien
conda create --name nouveau_nom --clone ancien_nom
conda env remove --name ancien_nom
```

#### Variables d'environnement

```bash
# Définir des variables pour un environnement
conda env config vars set VAR_NAME=value -n mon_env

# Lister les variables
conda env config vars list -n mon_env

# Supprimer une variable
conda env config vars unset VAR_NAME -n mon_env
```

### 🧹 Nettoyage et maintenance

```bash
# Nettoyer les packages non utilisés dans les caches
conda clean --all

# Nettoyer les paquets tarballs
conda clean --tarballs

# Nettoyer les caches de packages téléchargés
conda clean --packages

# Supprimer les index inutilisés
conda clean --index-cache
```

> [!info]- 🧹 Détails sur `conda clean`
> 
> - `--all` : supprime **tout ce qui est nettoyable** (à utiliser avec précaution)
> - `--tarballs` : supprime les archives `.tar.bz2` téléchargées
>     - ✅ Gagne de la place disque
>     - ⚠️ Re-téléchargement nécessaire si réinstallation
> - `--packages` : supprime les paquets inutilisés dans le cache
>     - ✅ Utile après des tests temporaires
>     - ⚠️ Ne pas utiliser en plein développement
> - `--index-cache` : supprime les caches des index de paquets
>     - ✅ À utiliser si conda semble bloqué
>     - ⚠️ Allonge la prochaine résolution

### ⚙️ Optimiser conda : changer le solveur

```bash
# Installer libmamba
conda install -n base conda-libmamba-solver

# L'utiliser par défaut
conda config --set solver libmamba

# Vérifier la configuration
conda config --show solver
```

> [!success] Performance Libmamba est **10 à 100 fois plus rapide** que le solveur par défaut de conda.

---

## 🚄 Mamba : L'alternative rapide

🔗 [Documentation officielle de Mamba](https://mamba.readthedocs.io/)

### 🧱 Installation avec Miniforge

**Miniforge** : distribution conda minimale avec mamba natif et conda-forge par défaut.

```bash
# Linux/macOS
wget https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-Linux-x86_64.sh
bash Miniforge3-Linux-x86_64.sh

# Ou directement avec curl
curl -L -O https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-Linux-x86_64.sh
bash Miniforge3-Linux-x86_64.sh
```

> [!tip] Installation recommandée Je recommande **fortement** Miniforge plutôt que Anaconda ou Miniconda pour :
> 
> - Mamba intégré nativement
> - Conda-forge par défaut
> - Pas de limitations commerciales
> - Plus léger qu'Anaconda

> [!info] Liens utiles
> 
> - [Miniforge sur GitHub](https://github.com/conda-forge/miniforge)
> - [Guide d'installation détaillé](https://github.com/conda-forge/miniforge#install)

### 🔍 Comparatif Mamba vs Conda

#### Avantages techniques de Mamba

|Aspect|Conda|Mamba|
|---|---|---|
|**Solveur**|Python (lent)|C++ (jusqu'à 100x plus rapide)|
|**Téléchargements**|Séquentiel|Parallèle|
|**RAM**|Usage élevé|Optimisé|
|**Interface**|Basique|Plus lisible avec barres de progression|
|**Résolution conflits**|Parfois lente|Beaucoup plus rapide|

#### Compatibilité complète

Mamba est un **drop-in replacement** de conda :

```bash
# Remplacez simplement 'conda' par 'mamba'
mamba create -n mon_env python=3.11
mamba install -c conda-forge geopandas
mamba env export > environment.yml
mamba activate mon_env
```

#### Limites actuelles

- Quelques options très avancées peuvent manquer
- Base d'utilisateurs plus petite (mais en croissance rapide)
- Résolution parfois légèrement différente (généralement meilleure)
- Moins de documentation disponible en ligne

### 🛠️ Configuration recommandée

```bash
# Définir conda-forge en priorité
mamba config --add channels conda-forge
mamba config --set channel_priority strict

# Configuration SSL si problèmes réseau
mamba config --set ssl_verify false  # à éviter en production

# Vérifier la configuration
mamba config --show channels
mamba config --show
```

---

## 🎯 Workflows recommandés

### 🚀 Nouveau projet (workflow optimal)

```bash
# 1. Créer l'environnement avec version Python spécifique
mamba create -n projet_2025 python=3.11

# 2. Activer
mamba activate projet_2025

# 3. Installer les dépendances de base
mamba install -c conda-forge numpy pandas matplotlib

# 4. Exporter pour la reproductibilité
mamba env export --no-builds > environment.yml

# 5. Ajouter au contrôle de version
git add environment.yml
```

### 🔄 Reproduire un environnement

```bash
# Créer depuis le fichier
mamba env create -f environment.yml

# Ou mettre à jour un environnement existant
mamba env update -f environment.yml --prune

# Activer
mamba activate nom_du_projet
```

### 🔧 Maintenance régulière

```bash
# Mettre à jour l'environnement
mamba update --all

# Exporter les changements
mamba env export --no-builds > environment.yml

# Nettoyer périodiquement
mamba clean --tarballs
```

> [!tip] Bonnes pratiques
> 
> - **Un environnement = un projet**
> - **Noms clairs** et explicites (`projet_nom_version`)
> - **Export régulier** de `environment.yml`
> - **Toujours utiliser conda-forge** comme canal principal
> - **Utiliser Mamba** pour optimiser les performances
> - **Versionner** les fichiers `environment.yml`

---

## 🔧 Résolution de problèmes courants

### Environnement corrompu

```bash
# Recréer un environnement depuis l'export
mamba env create -f environment.yml --force

# Ou nettoyer et réinstaller
mamba clean --all
mamba env remove -n problematic_env
mamba env create -f environment.yml
```

### Conflits de dépendances

```bash
# Forcer la résolution avec mamba (plus efficace)
mamba install package_name --force-reinstall

# Ou installer depuis un canal spécifique
mamba install -c conda-forge package_name
```

### Problèmes de performance

```bash
# Passer à libmamba si vous utilisez encore conda
conda config --set solver libmamba

# Ou migrer vers mamba/miniforge (recommandé)
```

---

## 📚 Ressources supplémentaires

### Documentation officielle

- [Documentation Conda](https://docs.conda.io/)
- [Documentation Mamba](https://mamba.readthedocs.io/)
- [Guide Conda-forge](https://conda-forge.org/docs/)

### Guides et cheat sheets

- [Cheat sheet Conda](https://docs.conda.io/projects/conda/en/4.6.0/_downloads/52a95608c49671267e40c689e0bc00ca/conda-cheatsheet.pdf)
- [Guide migration vers Mamba](https://mamba.readthedocs.io/en/latest/user_guide/mamba.html)

### Notes connexes

- [[Packages Python]] - Installation et gestion des librairies