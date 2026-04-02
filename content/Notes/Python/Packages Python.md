---
title: Packages Python
tags:
  - python
  - conda
  - mamba
  - pip
  - packages
  - librairies
description: Installation et gestion des packages Python avec conda, mamba et pip
---

## Gestion des packages Python

Cette note complète le guide sur la [[Gestion des environnements]] en détaillant l'installation et la gestion des packages dans les environnements conda/mamba.

---

## Conda vs Pip : Comprendre les différences

### Architecture et fonctionnement

| Aspect           | Conda                                 | Pip                         |
| ---------------- | ------------------------------------- | --------------------------- |
| **Gestionnaire** | Packages + environnements             | Packages uniquement         |
| **Dépendances**  | Binaires pré-compilées                | Source + compilation        |
| **Langages**     | Python, R, C++, Fortran...            | Python uniquement           |
| **Installation** | Binaires optimisés                    | Compilation locale possible |

### Quand utiliser quoi ?

#### Préférer Conda/Mamba pour :
- **Packages scientifiques** (numpy, scipy, pandas, matplotlib)
- **Librairies géospatiales** (GDAL, GEOS, PROJ)
- **Dépendances complexes** (OpenCV, TensorFlow, PyTorch)
- **Packages avec extensions C** (lxml, Pillow, psycopg2)

#### Utiliser Pip pour :
- **Packages Python pur** sans dépendances complexes
- **Packages non disponibles** sur conda-forge
- **Versions de développement** depuis Git
- **Packages locaux** en mode éditable

---

## Installation avec Conda/Mamba

>[!tip] Possible de remplacer `conda` par `mamba` comme expliqué dans [[Gestion des environnements#Compatibilité complète]] **à lire**
### Installation basique

```bash
# Package simple
conda install numpy

# Version spécifique
conda install numpy=1.24.0
conda install "numpy>=1.24,<1.25"

# Canal spécifique (recommandé : conda-forge)
conda install -c conda-forge geopandas

# Plusieurs packages
conda install -c conda-forge numpy pandas matplotlib geopandas
```

### Gestion des canaux
>[!tip] Possible de le géré par défaut, voir [[Gestion des environnements]]

```bash
# Ajouter un canal
conda config --add channels conda-forge

# Priorité des canaux
conda config --set channel_priority strict

# Lister les canaux configurés
conda config --show channels

# Installer depuis un canal spécifique sans l'ajouter
conda install -c conda-forge package_name

# Ordre de priorité des canaux (par défaut)
conda config --add channels defaults
conda config --add channels conda-forge  # Plus prioritaire
```

> [!tip]- Canaux recommandés
> 
> - **conda-forge** : canal communautaire de référence
> - **defaults** : canal officiel Anaconda
> - **pytorch** : pour PyTorch et dépendances

### Mise à jour et suppression

```bash
# Mettre à jour un package
conda update numpy

# Mettre à jour tous les packages
conda update --all

# Supprimer un package
conda remove numpy

# Supprimer avec dépendances inutilisées
conda remove numpy --force-remove

# Lister les packages installés
conda list
conda list numpy  # recherche spécifique
```

### Recherche de packages

```bash
# Rechercher un package
conda search numpy

# Rechercher dans un canal spécifique
conda search -c conda-forge gdal

# Informations détaillées sur un package
conda info numpy

# Voir les dépendances d'un package
conda depends numpy
```

---

## Installation avec Pip

### Installation depuis PyPI

```bash
# Installation basique
pip install requests

# Version spécifique
pip install numpy==1.24.0
pip install "numpy>=1.24,<1.25"

# Plusieurs packages
pip install requests beautifulsoup4 lxml

# Depuis requirements.txt
pip install -r requirements.txt
```

### Installation depuis Git

```bash
# Depuis une branche spécifique
pip install git+https://github.com/user/repo.git@branch_name

# Depuis un tag
pip install git+https://github.com/user/repo.git@v1.0.0

# Depuis un commit spécifique
pip install git+https://github.com/user/repo.git@abc123

# Avec SSH (si configuré)
pip install git+ssh://git@github.com/user/repo.git

# Depuis GitLab ou autres
pip install git+https://gitlab.com/user/repo.git
```

### Installation en mode développement

```bash
# Installation locale éditable (mode développement)
pip install -e .

# Avec des dépendances optionnelles
pip install -e .[dev]
pip install -e .[test,docs]

# Depuis un autre répertoire
pip install -e ./path/to/package

# Désinstaller un package en mode éditable
pip uninstall package_name
```

### Options avancées de Pip

```bash
# Installation sans cache
pip install --no-cache-dir package_name

# Installation sans dépendances
pip install --no-deps package_name

# Installation dans un répertoire spécifique
pip install --target ./lib package_name

# Forcer la réinstallation
pip install --force-reinstall package_name

# Installation depuis un fichier wheel local
pip install ./package-1.0-py3-none-any.whl

# Télécharger sans installer
pip download package_name
```

---

## Packages géospatiaux : cas particuliers

### GDAL et écosystème géospatial

Les packages géospatiaux ont des dépendances complexes (GDAL, GEOS, PROJ, etc.). **Conda est fortement recommandé** :

```bash
# Installation recommandée avec mamba
conda install -c conda-forge gdal geos proj

# Package géospatiaux populaires
conda install -c conda-forge geopandas rasterio fiona shapely
```

> [!warning] Installer GDAL avec pip peut être **très problématique** :
> 
> - Compilation complexe
> - Dépendances système requises
> - Versions incompatibles fréquentes
> 
> **Utilisez toujours conda/mamba pour GDAL !**
---

## Stratégies de gestion hybride Conda + Pip

### Ordre d'installation recommandé

```bash
# 1. D'abord, installer les packages scientifiques/géospatiaux avec conda
conda install -c conda-forge numpy pandas geopandas matplotlib

# 2. Ensuite, compléter avec pip pour les packages purs Python [ou conda]
pip install requests beautifulsoup4 streamlit

# 3. Packages de développement depuis Git
pip install git+https://github.com/user/dev_package.git
```

### Fichier environment.yml hybride

```yaml
name: projet_geo
channels:
  - conda-forge
  - defaults
dependencies:
  # Packages conda (prioritaires)
  - python=3.11
  - numpy
  - pandas
  - geopandas
  - matplotlib
  - jupyter
  - streamlit
  - requests
  # Packages pip
  - pip
  - pip:
    - git+https://github.com/user/custom_package.git
    - -e ./local_package  # package local en mode éditable
```

> [!tip] Bonnes pratiques hybrides
> 
> 1. **Toujours installer conda packages en premier**
> 2. **Pip en complément** pour les packages non disponibles
> 3. **Éviter** de mélanger conda et pip pour le même package
> 4. **Exporter régulièrement** avec `conda env export`

---

## Résolution de conflits et problèmes

### Conflits de dépendances

```bash
# Diagnostiquer les conflits
conda list --show-channel-urls

# Forcer la résolution avec mamba
conda install package_name --force-reinstall

# Spécifier explicitement les canaux
conda install -c conda-forge -c defaults package_name

# En dernier recours : nettoyer et recréer
conda env remove -n env_name
conda env create -f environment.yml
```

### Nettoyage des packages

```bash
# Conda/Mamba
conda clean --packages --tarballs

# Pip
pip cache purge

# Voir l'espace utilisé
pip cache info
```

---

## Monitoring et audit des packages

### Informations sur les packages

```bash
# Détails d'un package conda
conda info package_name

# Historique des installations
conda list --revisions

# Packages pip avec leurs dépendances
pip show package_name

# Arbre des dépendances (avec pipdeptree)
pip install pipdeptree
pipdeptree
```

### Sécurité et vulnérabilités

```bash
# Audit de sécurité avec pip-audit
pip install pip-audit
pip-audit

# Vérification des packages obsolètes
pip list --outdated

# Mise à jour sécurisée
pip install --upgrade package_name
```

---

## Ressources et documentation

### Documentation officielle

- [Conda packages](https://docs.conda.io/projects/conda/en/latest/user-guide/tasks/manage-pkgs.html)
- [Pip user guide](https://pip.pypa.io/en/stable/user_guide/)
- [Conda-forge documentation](https://conda-forge.org/docs/)

### Outils utiles

- [conda-tree](https://github.com/rvalieris/conda-tree) : visualiser l'arbre des dépendances
- [pipdeptree](https://github.com/tox-dev/pipdeptree) : arbre des dépendances pip
- [pip-audit](https://github.com/pypa/pip-audit) : audit de sécurité