---
title: Fichiers d'installation yml | txt | ...
tags:
  - python
  - configuration
  - yml
  - yaml
  - requirements
  - toml
description: Guide des différents formats de fichiers de configuration pour les projets Python
---

> [!warning] Information
> Cette notes est une compilation d'explication déjà contenu dans les notes [[Gestion des environnements]], [[Packages Python]] et [[pyproject.toml]] et présente des exemples d'utilisation / construction de fichier d'installation

## 🐍 environment.yml - Environnements conda

### 📋 Structure complète

```yaml
name: mon_projet_geo
channels:
  - conda-forge
  - defaults
dependencies:
  # Version Python
  - python=3.11
  
  # Packages scientifiques de base
  - numpy>=1.24.0
  - pandas>=2.0.0
  - matplotlib>=3.7.0
  
  # Packages géospatiaux (meilleurs avec conda)
  - geopandas>=0.13.0
  - rasterio>=1.3.0
  - fiona>=1.9.0
  - shapely>=2.0.0
  - gdal>=3.6.0
  
  # Outils de développement
  - jupyter
  - ipykernel
  
  # Gestionnaire pip
  - pip
  
  # Packages uniquement disponibles via pip
  - pip:
    - streamlit>=1.25.0
    - plotly>=5.15.0
    - kaleido  # pour export plotly
    - git+https://github.com/user/dev_package.git
    - -e ./local_package

# Variables d'environnement (optionnel)
variables:
  - GDAL_DATA: $CONDA_PREFIX/share/gdal
  - PROJ_LIB: $CONDA_PREFIX/share/proj
  - PYTHONPATH: $CONDA_PREFIX/lib/python3.11/site-packages

# Préfixe personnalisé (optionnel)
prefix: /opt/conda/envs/mon_projet_geo
```

### 🎯 Cas d'usage

**Utilisez environment.yml pour :**
- Projets avec packages scientifiques/géospatiaux
- Reproduction exacte d'environnements
- Dépendances système complexes (GDAL, OpenCV, etc.)
- Collaboration en équipe

---

## 📄 requirements.txt - Dépendances pip

> [!info] Installation avec pip
> Pour les détails d'installation et gestion des packages pip, voir [[Packages Python#Installation avec Pip]]

### 📝 Formats et syntaxe

```txt
# requirements.txt basique
numpy>=1.24.0
pandas>=2.0.0,<3.0.0
matplotlib==3.7.2
requests~=2.31.0

# Avec commentaires
# Base scientifique
numpy>=1.24.0    # Arrays
pandas>=2.0.0    # DataFrames

# Visualisation
matplotlib>=3.7.0
plotly>=5.15.0

# Web
fastapi>=0.100.0
uvicorn[standard]>=0.23.0

# Développement
pytest>=7.4.0
black>=23.0.0
mypy>=1.5.0
```

### 🔗 Sources diverses

```txt
# Depuis PyPI (défaut)
requests>=2.31.0

# Depuis Git
git+https://github.com/user/repo.git
git+https://github.com/user/repo.git@v1.0.0
git+https://github.com/user/repo.git@main

# Package local en mode éditable
-e .
-e ./path/to/package

# Depuis URL directe
https://github.com/user/repo/archive/main.zip

# Depuis fichier local
./packages/my_package-1.0.0-py3-none-any.whl
```

### 📚 Organisation modulaire

```bash
# Structure organisée
requirements/
├── base.txt          # Dépendances communes
├── dev.txt           # Développement
├── prod.txt          # Production
├── test.txt          # Tests
└── docs.txt          # Documentation
```

```txt
# requirements/base.txt
numpy>=1.24.0
pandas>=2.0.0
requests>=2.31.0

# requirements/dev.txt
-r base.txt
pytest>=7.4.0
black>=23.0.0
mypy>=1.5.0
jupyter>=1.0.0

# requirements/prod.txt
-r base.txt
gunicorn>=21.0.0
psycopg2-binary>=2.9.0
```

### 🎯 Cas d'usage

**Utilisez requirements.txt pour :**
- Projets Python pur sans dépendances complexes
- Déploiement en production
- CI/CD et conteneurs Docker
- Packages disponibles uniquement sur PyPI

---

## 🔗 Comparatif des formats

| Aspect | environment.yml | requirements.txt | pyproject.toml |
|--------|----------------|------------------|----------------|
| **Gestionnaire** | conda/mamba | pip | pip + outils modernes |
| **Environnements** | ✅ Gestion complète | ❌ Packages seulement | ❌ Packages seulement |
| **Dépendances système** | ✅ Binaires pré-compilés | ❌ Compilation locale | ❌ Compilation locale |
| **Multi-langage** | ✅ Python, R, C++, etc. | ❌ Python seulement | ❌ Python seulement |
| **Métadonnées projet** | ❌ | ❌ | ✅ Complètes |
| **Standard moderne** | ❌ | ❌ | ✅ PEP 518/621 |

---

## 🔗 Intégration et workflow

### 🏗️ Workflow recommandé

```bash
# 1. Développement avec environment.yml
conda env create -f environment.yml
conda activate mon_projet

# 2. Configuration du projet avec pyproject.toml
pip install -e .[dev]
```

---

## 📚 Ressources

### 📖 Documentation

- [Conda environment.yml](https://docs.conda.io/projects/conda/en/latest/user-guide/tasks/manage-environments.html#creating-an-environment-file-manually)
- [Pip requirements](https://pip.pypa.io/en/stable/reference/requirements-file-format/)