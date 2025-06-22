---
title: Configuration de pyproject.toml
tags:
  - python
  - pyproject
  - toml
  - configuration
  - projet
description: Guide pour configurer un projet Python avec pyproject.toml
---

## 🚀 Configuration avec pyproject.toml

Le fichier `pyproject.toml` est le standard moderne pour configurer un projet Python (PEP 518, 621, 660). Il remplace `setup.py` par une approche déclarative plus sûre et lisible.

>[!warning] Cette note nécessite une bonne compréhension des environnements Python, de la gestion de versions, des paquets, de la gestion de dépôts, etc. Elle liste l'ensemble des possibilités présentes dans les fichiers .toml (que j'utilise personnellement). La maîtrise de [[Packages Python]], [[Fichier d'installation]] est requise.

---

## 🏗️ Structure fondamentale

### 📦 Section [build-system]

```toml
[build-system]
requires = [
    "setuptools>=64",
    "wheel",
    'tomli; python_version < "3.11"'  # support TOML pour Python < 3.11
]
build-backend = "setuptools.build_meta"
````

### 🏷️ Section [project] - Métadonnées

```toml
[project]
name = "mon_projet_geo"
version = "0.1.0"
description = "Projet de géosciences avec Python"
readme = "README.md"
license = { file = "LICENSE" }
requires-python = ">=3.11"

authors = [
    { name = "Bastien Boivin", email = "bastien.boivin@univ-rennes.fr" },
]

keywords = [
    "géosciences", "python", "data analysis",
    "GIS", "hydrology"
]

classifiers = [
    "Programming Language :: Python :: 3",
    "Programming Language :: Python :: 3.11",
    "License :: OSI Approved :: MIT License",
    "Operating System :: OS Independent",
    "Intended Audience :: Science/Research",
    "Topic :: Scientific/Engineering :: GIS"
]
```

> [!info]- Classifiers
> 
> ## Qu'est-ce que les classifiers ?
> 
> Les **classifiers** sont des métadonnées standardisées qui décrivent un package Python sur PyPI. Ils fonctionnent comme des "étiquettes" qui aident à :
> 
> - Catégoriser le projet
> - Faciliter la recherche et le filtrage
> - Informer les utilisateurs sur la compatibilité
> - Améliorer la visibilité du package
> 
> ## Structure des classifiers
> 
> Les classifiers suivent une hiérarchie à plusieurs niveaux séparés par `::` :
> 
> ```
> Catégorie :: Sous-catégorie :: Détail
> ```
> 
> ## Principales catégories
> 
> ### **Programming Language**
> 
> ```toml
> "Programming Language :: Python :: 3"
> "Programming Language :: Python :: 3.11"
> "Programming Language :: Python :: 3.12"
> ```
> 
> Spécifie les versions Python supportées
> 
> ### **License**
> 
> ```toml
> "License :: OSI Approved :: MIT License"
> "License :: OSI Approved :: Apache Software License"
> "License :: OSI Approved :: GNU General Public License v3 (GPLv3)"
> ```
> 
> ### **Operating System**
> 
> ```toml
> "Operating System :: OS Independent"
> "Operating System :: Microsoft :: Windows"
> "Operating System :: POSIX :: Linux"
> "Operating System :: MacOS"
> ```
> 
> ### **Intended Audience**
> 
> ```toml
> "Intended Audience :: Developers"
> "Intended Audience :: Science/Research"
> "Intended Audience :: Education"
> "Intended Audience :: End Users/Desktop"
> ```
> 
> ### **Topic**
> 
> ```toml
> "Topic :: Scientific/Engineering :: GIS"
> "Topic :: Software Development :: Libraries :: Python Modules"
> "Topic :: Internet :: WWW/HTTP :: Dynamic Content"
> "Topic :: Database"
> ```
> 
> ### **Development Status**
> 
> ```toml
> "Development Status :: 3 - Alpha"
> "Development Status :: 4 - Beta"
> "Development Status :: 5 - Production/Stable"
> ```
> 
> ## Comment choisir les classifiers
> 
> 1. **Consulter la liste officielle** : [Liste complète des classifiers PyPI](https://pypi.org/classifiers/)
> 2. **Être précis** : Choisir les classifiers qui décrivent réellement le projet
> 3. **Penser à l'audience** : Identifier qui va utiliser le package
> 4. **Inclure les versions Python** : Tester et spécifier toutes les versions supportées
> 
> ## Exemple complet
> 
> ```toml
> classifiers = [
>    "Development Status :: 4 - Beta",
>    "Programming Language :: Python :: 3",
>    "Programming Language :: Python :: 3.11",
>    "Programming Language :: Python :: 3.12",
>    "License :: OSI Approved :: MIT License",
>    "Operating System :: OS Independent",
>    "Intended Audience :: Developers",
>    "Intended Audience :: Science/Research",
>    "Topic :: Scientific/Engineering :: GIS",
>    "Topic :: Software Development :: Libraries :: Python Modules"
> ]
> ```

---

## 📚 Gestion des dépendances

### 🔢 Syntaxe des versions

#### Opérateurs de version

```toml
dependencies = [
    # Version exacte (rigide, non recommandé sauf cas particulier)
    "numpy==1.24.3",
    
    # Version minimale (recommandé pour stabilité)
    "pandas>=2.0.0",
    
    # Fourchette de versions (sécurisé pour breaking changes)
    "matplotlib>=3.7.0,<4.0.0",
    
    # Version compatible (tilde) : accepte les correctifs
    "requests~=2.31.0",  # équivalent à ">=2.31.0,<2.32.0"
    
    # Version compatible (caret) : accepte mises à jour mineures
    "fastapi^=0.100.0",  # équivalent à ">=0.100.0,<1.0.0"
    
    # Excluure une version problématique
    "scipy>=1.9.0,!=1.10.1",
    
    # Sans contrainte (à éviter en production)
    "plotly",
]
```

#### Bonnes pratiques de versioning

```toml
dependencies = [
    # ✅ Version minimale stable pour packages matures
    "numpy>=1.21.0",
    
    # ✅ Fourchette pour éviter breaking changes
    "pandas>=2.0.0,<3.0.0",
    
    # ✅ Version compatible pour packages qui suivent semver
    "pydantic~=2.5.0",
    
    # ❌ Version exacte (trop rigide)
    # "requests==2.31.0",
    
    # ❌ Sans contrainte (risqué)
    # "matplotlib",
]
```

### 🎯 Dependencies vs Optional Dependencies

#### **Dependencies** : Dépendances obligatoires

```toml
[project]
dependencies = [
    # Packages ESSENTIELS au fonctionnement du projet
    "numpy>=1.21.0",        # calculs numériques de base
    "pandas>=2.0.0",        # manipulation de données
]
```

- **Installées automatiquement** avec `pip install mon_projet`
- **Indispensables** pour que le package fonctionne
- Doivent être **minimales** et **stables**

#### **Optional Dependencies** : Dépendances optionnelles

```toml
[project.optional-dependencies]
# Groupe de développement
dev = [
    "pytest>=7.0.0",        # tests
    "black>=22.0.0",        # formatage
    "mypy>=1.0.0",          # vérification types
    "pre-commit>=2.20.0",   # hooks git
]

# Groupe de documentation
docs = [
    "sphinx>=5.0.0",
    "sphinx-rtd-theme>=1.0.0",
    "numpydoc",
]

# Fonctionnalités spécialisées
geo = [
    "geopandas>=0.12.0",    # données géospatiales
    "folium>=0.14.0",       # cartes interactives
    "cartopy>=0.21.0",      # projections cartographiques
]
```

- **Installation optionnelle** : `pip install mon_projet[dev]`
- **Fonctionnalités spécialisées** non requises pour tous les utilisateurs
- **Outils de développement** séparés du code de production

#### Exemple d'utilisation des groupes

```bash
# Installation de base (dependencies seulement)
pip install mon_projet

# Installation avec développement
pip install mon_projet[dev]

# Installation avec plusieurs groupes
pip install mon_projet[dev,docs,geo]

# Installation de tous les groupes optionnels
pip install mon_projet[dev,docs,geo,ml]

# En mode éditable pour développement
pip install -e .[dev,test]
```

### 🌐 URLs du projet

```toml
[project.urls]
homepage = "https://github.com/bastien-boivin/mon_projet"
repository = "https://github.com/bastien-boivin/mon_projet.git"
documentation = "https://mon-projet.readthedocs.io"
"Bug Tracker" = "https://github.com/bastien-boivin/mon_projet/issues"
docker = "https://hub.docker.com/r/bastien/mon_projet"
```

---

## 🛠️ Configuration setuptools

### 📦 Découverte des packages

```toml
[tool.setuptools.packages.find]
where = ["."]
include = ["mon_projet*"]
exclude = ["tests*", "docs*"]

# Ou spécifier explicitement
[tool.setuptools.packages]
mon_projet = "src/mon_projet"
```

### 📂 Répertoire des packages

```toml
[tool.setuptools.package-dir]
"" = "src"  # packages dans src/
# ou
"" = "."   # packages à la racine
```

### 📄 Fichiers de données

```toml
[tool.setuptools.package-data]
mon_projet = [
    "data/*.csv",
    "templates/*.html",
    "static/css/*.css",
]

# Ou globalement
[tool.setuptools]
include-package-data = true
```

---

## 🚀 Utilisation pratique

### 🔧 Installation en mode développement

```bash
# Installation éditable avec dépendances de base
pip install -e .

# Avec dépendances optionnelles
pip install -e .[dev]
pip install -e .[dev,docs,test]

# Toutes les dépendances optionnelles
pip install -e .[dev,docs,test,geo]
```

### 🔄 Intégration avec conda

```yaml
# environment.yml
name: mon_projet
channels:
  - conda-forge
dependencies:
  - python=3.11
  - numpy>=1.24.0
  - pandas>=2.0.0
  - geopandas>=0.13.0
  - pip
  - pip:
    - -e .  # installe le projet local via pyproject.toml
```

---

## 🔧 Résolution de problèmes

### 🚨 Erreurs courantes

```bash
# Vérifier la syntaxe TOML
python -c "import tomli; tomli.load(open('pyproject.toml', 'rb'))"

# Vérifier la configuration
python -m setuptools check

# Problème avec les dépendances dynamiques
pip install -e . --force-reinstall
```

### 🔍 Validation et debugging

```bash
# Installer des outils de validation
pip install validate-pyproject check-manifest

# Valider le pyproject.toml
validate-pyproject pyproject.toml

# Vérifier les fichiers inclus
check-manifest
```

---

## 📚 Ressources

### 📖 Standards PEP

- [PEP 518](https://peps.python.org/pep-0518/) - pyproject.toml
- [PEP 621](https://peps.python.org/pep-0621/) - Métadonnées projet
- [PEP 660](https://peps.python.org/pep-0660/) - Installation éditable

### 🛠️ Documentation

- [Setuptools et pyproject.toml](https://setuptools.pypa.io/en/latest/userguide/pyproject_config.html)
- [Packaging Python](https://packaging.python.org/en/latest/)
- [TOML specification](https://toml.io/en/)