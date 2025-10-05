---
title: Montage OneDrive avec rclone
tags:
  - linux
  - ubuntu
  - onedrive
  - cloud
  - rclone
description: Configuration de OneDrive sur Linux avec rclone pour un accès sans synchronisation automatique
---

## Montage OneDrive sur Linux

rclone permet de monter OneDrive comme un système de fichiers virtuel sur Linux. Contrairement à une synchronisation classique, les fichiers ne sont téléchargés que lorsque vous les ouvrez, économisant ainsi de l'espace disque.

> [!info] Fonctionnement du montage virtuel
> Le dossier monté affiche tous vos fichiers OneDrive, mais ne les télécharge qu'à la demande. Seuls les fichiers récemment utilisés sont conservés temporairement en cache.

---

## Installation de rclone

```bash
# Installer rclone
sudo apt update
sudo apt install rclone

# Installer fuse (requis pour le montage)
sudo apt install fuse

# Vérifier l'installation
rclone version
```

---

## Configuration initiale

### Étape 1 : Lancer la configuration interactive

```bash
rclone config
```

Réponses à fournir :
- `n` → Nouveau remote
- Nom : `OneDrive` (ou autre nom de votre choix)
- Storage type : `30` (Microsoft OneDrive)
- Client ID : `Entrée` (laisser vide)
- Client Secret : `Entrée` (laisser vide)
- Region : `1` (Microsoft Cloud Global)
- Edit advanced config : `n`
- Use auto config : `n`

### Étape 2 : Générer le token d'authentification

Sur une machine avec navigateur (même PC ou autre) :

```bash
rclone authorize "onedrive"
```

**Processus :**
1. Le navigateur s'ouvre automatiquement
2. Connexion au compte Microsoft
3. Autorisation de rclone
4. Copie du JSON complet affiché dans le terminal

**Format attendu :**
```json
{
  "access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOi...",
  "token_type": "Bearer",
  "refresh_token": "M.C524_BAY.0.U.-CkNmdjPCzJ...",
  "expiry": "2025-10-05T14:32:45.614447619+02:00"
}
```

### Étape 3 : Finaliser la configuration

Dans `rclone config` :
- Coller le JSON complet à `config_token>`
- Type de connexion : `1` (OneDrive Personal or Business)
- Sélection du drive : `0` ou le numéro correspondant à votre OneDrive principal
- Confirmer : `y`
- Quitter : `q`

> [!warning] Format du token
> Le token doit être au format JSON complet avec les accolades `{}`. Ne pas coller seulement l'access_token brut.

---

## Utilisation du montage

### Montage manuel

```bash
# Créer le point de montage
mkdir ~/OneDrive-Rclone

# Monter OneDrive
rclone mount OneDrive: ~/OneDrive-Rclone --vfs-cache-mode full --daemon

# Vérifier le montage
ls ~/OneDrive-Rclone

# Démonter
fusermount -u ~/OneDrive-Rclone
```

**Options importantes :**
- `--vfs-cache-mode full` : Cache complet pour de meilleures performances
- `--daemon` : Exécution en arrière-plan

### Montage automatique au démarrage

```bash
# Éditer crontab
crontab -e

# Ajouter cette ligne (via vim ou autre)
@reboot rclone mount OneDrive: ~/OneDrive-Rclone --vfs-cache-mode full --daemon

# Vérifier la configuration
crontab -l
```

---

## Fonctionnement détaillé

### Comportement des fichiers

| Action | Résultat |
|--------|----------|
| Lister les fichiers (`ls`) | Affichage sans téléchargement |
| Ouvrir un fichier | Téléchargement temporaire |
| Copier vers dossier local | Téléchargement permanent |
| Glisser-déposer vers OneDrive | Upload vers le cloud |
| Fermer un fichier | Suppression possible du cache |

### Système de cache

**Emplacement :** `~/.cache/rclone/`

**Caractéristiques :**
- Gestion automatique de l'espace
- Suppression des fichiers peu utilisés
- Conservation temporaire des fichiers récents

```bash
# Vérifier l'espace utilisé par le cache
du -sh ~/.cache/rclone/

# Limiter la taille du cache (exemple : 1 Go)
rclone mount OneDrive: ~/OneDrive-Rclone \
  --vfs-cache-mode full \
  --vfs-cache-max-size 1G \
  --daemon
```

### Modes de cache disponibles

```bash
# Mode off : Pas de cache (streaming pur)
--vfs-cache-mode off

# Mode minimal : Cache metadata uniquement
--vfs-cache-mode minimal

# Mode writes : Cache les écritures
--vfs-cache-mode writes

# Mode full : Cache complet (recommandé)
--vfs-cache-mode full
```

> [!tip] Choix du mode de cache
> - **full** : Recommandé pour un usage bureautique normal
> - **off** : Pour économiser de l'espace (peut causer des lenteurs)
> - **writes** : Bon compromis entre performance et espace

---

## Commandes utiles

### Vérification et diagnostic

```bash
# Lister les dossiers OneDrive
rclone lsd OneDrive:

# Voir l'espace utilisé sur OneDrive
rclone size OneDrive:

# Afficher la configuration
rclone config show OneDrive

# Tester la connexion
rclone about OneDrive:

# Lister les montages actifs
mount | grep rclone
```

### Gestion des fichiers

```bash
# Copier un fichier local vers OneDrive
rclone copy /chemin/local/fichier.txt OneDrive:

# Copier un dossier
rclone copy /chemin/local/dossier OneDrive:dossier_cible

# Synchroniser (supprime les fichiers non présents dans la source)
rclone sync /chemin/local OneDrive:backup

# Supprimer un fichier sur OneDrive
rclone delete OneDrive:fichier.txt

# Vider la corbeille OneDrive
rclone cleanup OneDrive:
```

### Opérations avancées

```bash
# Afficher les transferts en temps réel
rclone mount OneDrive: ~/OneDrive-Rclone \
  --vfs-cache-mode full \
  --log-level INFO \
  --log-file ~/rclone.log

# Monter en lecture seule
rclone mount OneDrive: ~/OneDrive-Rclone \
  --vfs-cache-mode full \
  --read-only \
  --daemon

# Filtrer certains fichiers/dossiers
rclone mount OneDrive: ~/OneDrive-Rclone \
  --vfs-cache-mode full \
  --exclude "*.tmp" \
  --exclude ".DS_Store" \
  --daemon
```

---

## Résolution de problèmes

### Montage ne fonctionne pas au démarrage

```bash
# Vérifier les logs cron
journalctl -u cron -f

# Vérifier les logs rclone
journalctl | grep rclone

# Test manuel du script
rclone mount OneDrive: ~/OneDrive-Rclone --vfs-cache-mode full --daemon
```

### Erreurs d'authentification

```bash
# Régénérer le token
rclone config reconnect OneDrive:

# Supprimer et recréer la configuration
rclone config delete OneDrive
rclone config
```

### Permission denied lors du montage

```bash
# Vérifier que fuse est installé
dpkg -l | grep fuse

# Vérifier les permissions du point de montage
ls -ld ~/OneDrive-Rclone

# Ajouter l'utilisateur au groupe fuse
sudo usermod -a -G fuse $USER
```

### Problèmes de cache

```bash
# Vider le cache manuellement
rm -rf ~/.cache/rclone/*

# Désactiver le cache temporairement
fusermount -u ~/OneDrive-Rclone
rclone mount OneDrive: ~/OneDrive-Rclone --vfs-cache-mode off --daemon
```

> [!warning] Démontage forcé
> En cas de blocage, utiliser `fusermount -uz ~/OneDrive-Rclone` pour forcer le démontage.

---

## Nettoyage et désinstallation

### Désactiver le montage automatique

```bash
# Éditer crontab
crontab -e

# Supprimer la ligne @reboot
# Sauvegarder et quitter
```

### Supprimer la configuration

```bash
# Démonter OneDrive
fusermount -u ~/OneDrive-Rclone

# Supprimer le remote
rclone config delete OneDrive

# Supprimer le cache
rm -rf ~/.cache/rclone/

# Supprimer le point de montage
rmdir ~/OneDrive-Rclone

# Désinstaller rclone (optionnel)
sudo apt remove rclone
```

---

## Comparaison avec autres solutions

| Solution | Synchronisation | Espace disque | Accès hors-ligne |
|----------|----------------|---------------|------------------|
| **rclone mount** | Non | Minimal (cache) | Partiel (cache) |
| **onedriver** | Oui | Important | Oui |
| **onedrive (abraunegg)** | Oui | Important | Oui |
| **Accès web** | Non | Aucun | Non |

**Avantages de rclone mount :**
- Économie d'espace disque
- Accès transparent via explorateur de fichiers
- Pas de synchronisation bidirectionnelle automatique
- Contrôle total sur le cache

**Inconvénients :**
- Nécessite une connexion Internet
- Latence à l'ouverture des fichiers
- Pas de véritable mode hors-ligne

---
### Service systemd (alternative à cron)

```bash
# Créer le service
sudo nano /etc/systemd/system/onedrive-mount.service
```

```ini
[Unit]
Description=OneDrive rclone mount
After=network-online.target
Wants=network-online.target

[Service]
Type=notify
User=%u
ExecStart=/usr/bin/rclone mount OneDrive: %h/OneDrive-Rclone \
    --vfs-cache-mode full \
    --vfs-cache-max-size 2G \
    --log-level INFO
ExecStop=/bin/fusermount -u %h/OneDrive-Rclone
Restart=on-failure
RestartSec=10

[Install]
WantedBy=default.target
```

```bash
# Activer le service
sudo systemctl daemon-reload
sudo systemctl enable onedrive-mount.service
sudo systemctl start onedrive-mount.service

# Vérifier le statut
systemctl status onedrive-mount.service
```

---

## Ressources et documentation

### Documentation officielle

- [rclone OneDrive](https://rclone.org/onedrive/)
- [rclone mount](https://rclone.org/commands/rclone_mount/)
- [VFS caching](https://rclone.org/commands/rclone_mount/#vfs-virtual-file-system)

### Alternatives

- [onedriver](https://github.com/jstaf/onedriver) : Client OneDrive natif pour Linux
- [onedrive (abraunegg)](https://github.com/abraunegg/onedrive) : Client de synchronisation complet
- [rclone browser](https://github.com/kapitainsky/RcloneBrowser) : Interface graphique pour rclone

### Commandes de référence rapide

```bash
# Configuration
rclone config

# Montage
rclone mount OneDrive: ~/OneDrive-Rclone --vfs-cache-mode full --daemon

# Démontage
fusermount -u ~/OneDrive-Rclone

# Test
rclone lsd OneDrive:

# Nettoyage cache
rm -rf ~/.cache/rclone/
```

---

## Cas d'usage pratiques

### Accès ponctuel aux fichiers

```bash
# Monter temporairement
rclone mount OneDrive: ~/OneDrive-Rclone --vfs-cache-mode full --daemon

# Ouvrir un fichier
xdg-open ~/OneDrive-Rclone/Documents/fichier.pdf

# Démonter après utilisation
fusermount -u ~/OneDrive-Rclone
```

### Sauvegarde locale vers OneDrive

```bash
# Sauvegarder un dossier important
rclone copy ~/Documents OneDrive:Backup/Documents

# Synchroniser (attention : supprime les fichiers distants non présents localement)
rclone sync ~/Documents OneDrive:Backup/Documents --interactive

# Vérifier les différences avant sync
rclone check ~/Documents OneDrive:Backup/Documents
```

### Partage de fichiers

```bash
# Créer un lien de partage
rclone link OneDrive:fichier.txt

# Lister les fichiers partagés
rclone lsf OneDrive: --shared
```

> [!tip] Workflow recommandé
> 1. Monter OneDrive au démarrage via crontab ou systemd
> 2. Utiliser le dossier monté comme un disque normal
> 3. Limiter la taille du cache selon l'espace disponible
> 4. Démonter manuellement si besoin d'économiser de la RAM
