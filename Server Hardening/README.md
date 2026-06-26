# 🔒 Playbook Ansible - Sécurisation SSH

[![Playbook Status](https://img.shields.io/badge/Ansible-≥%202.12-green?style=flat-square&logo=ansible)](https://docs.ansible.com/)
[![Licence](https://img.shields.io/badge/Licence-MIT-blue?style=flat-square)](LICENSE)
![Support](https://img.shields.io/badge/Support-Debian%20|%20AlmaLinux%20|%20Ubuntu%20|%20Rocky%20|%20CentOS-orange?style=flat-square&logo=linux)

> Automatisez la sécurisation de vos serveurs SSH avec ce playbook Ansible. Modification du port par défaut, désactivation du tunneling/forwarding X11, et déploiement de Fail2ban.

---

## 📋 Table des matières

- [Description](#-description)
- [✨ Fonctionnalités](#-fonctionnalités)
- [📌 Prérequis](#-prérequis)
- [📁 Structure du projet](#-structure-du-projet)
- [🚀 Installation](#-installation)
- [⚙️ Configuration](#️-configuration)
- [📖 Utilisation](#-utilisation)
- [🔄 Workflow du playbook](#-workflow-du-playbook)
- [🐧 Comportement par distribution](#-comportement-par-distribution)
- [⚠️ Sécurité - Notes importantes](#️-sécurité---notes-importantes)
- [🔧 Dépannage](#-dépannage)
- [📄 Licence](#-licence)

---

## 📖 Description

Ce playbook Ansible permet de sécuriser automatiquement un serveur SSH sur les distributions Linux **Debian** et **AlmaLinux** (et dérivées). Il est conçu pour être exécuté en **localhost** et apply les meilleures pratiques de sécurité :

- ✏️ **Changement du port SSH** par défaut (22 → personnalisé)
- 🚫 **Désactivation** du X11 Forwarding, TCP Forwarding, Agent Forwarding et tunneling
- 🛡️ **Installation et configuration de Fail2ban** pour contrer les attaques par force brute

> **💡 Compatible localhost** : Ce playbook est conçu pour sécuriser directement la machine sur laquelle il est exécuté, sans nécessiter d'inventaire distant.

---

## ✨ Fonctionnalités

| Fonctionnalité | Description |
|----------------|-------------|
| 🔢 **Changement de port SSH** | Modifie le port par défaut (22) vers un port personnalisé (par défaut : 2222) |
| 🚫 **Désactivation X11 Forwarding** | Empêche le forwarding graphique à distance |
| 🚫 **Désactivation TCP Forwarding** | Bloque le tunneling TCP via SSH |
| 🚫 **Désactivation Agent Forwarding** | Désactive le forwarding d'agent SSH |
| 🚫 **Désactivation Tunneling** | Empêche la création de tunnels VPN via SSH |
| 🛡️ **Fail2ban** | Installe et configure Fail2ban avec des règles sshd personnalisées |
| 💾 **Backup automatique** | Sauvegarde la configuration SSH avant modification |
| ✅ **Validation** | Valide la configuration SSH avant application (`sshd -t`) |
| 🔥 **Firewall** | Configure firewalld sur RHEL/AlmaLinux automatiquement |
| 🔐 **SELinux** | Configure les ports SELinux sur les systèmes RHEL |
| 🧪 **Vérifications** | Teste l'ouverture du port et le bon fonctionnement des services |

---

## 📌 Prérequis

### 🖥️ Système

| Composant | Exigence |
|-----------|----------|
| **Système d'exploitation** | Debian 10+, Ubuntu 18.04+, AlmaLinux 8+, Rocky 8+, CentOS 8+, RedHat 8+ |
| **Python** | Python 3.x installé sur la machine cible |
| **Accès root** | Possibilité d'utiliser `become: true` (sudo) |
| **Connectivité** | Accès réseau pour installation des paquets |

### 🛠️ Ansible

| Composant | Version minimum |
|-----------|-----------------|
| **Ansible** | 2.12+ |
| **Python** | 3.6+ |

### 📦 Collections Ansible requise

| Collection | Commande d'installation |
|------------|--------------------------|
| `community.general` | `ansible-galaxy collection install community.general` |
| `ansible.posix` | `ansible-galaxy collection install ansible.posix` |

---

## 📁 Structure du projet

```
ansible-ssh-security/
├── playbook.yml              # Playbook principal
├── vars/
│   └── main.yml             # Variables de configuration
└── templates/
    ├── sshd_config.j2       # Template de configuration SSH
    └── jail.local.j2        # Template de configuration Fail2ban
```

```
ansible-ssh-security/
├── playbook.yml          # ⭐ Playbook principal (point d'entrée)
├── vars/
│   └── main.yml         # ⚙️ Variables de configuration (à adapter)
└── templates/
    ├── sshd_config.j2   # 🔧 Template configuration SSH
    └── jail.local.j2    # 🔧 Template configuration Fail2ban
```

| Fichier | Rôle |
|---------|------|
| `playbook.yml` | Orchestration principale des tâches |
| `vars/main.yml` | Variables centralisées (port, paramètres fail2ban) |
| `templates/sshd_config.j2` | Configuration SSH sécurisée (Jinja2) |
| `templates/jail.local.j2` | Configuration Fail2ban (Jinja2) |

---

## 🚀 Installation

### 1. Cloner ou télécharger le projet

```bash
git clone https://github.com/votre-repo/ansible-ssh-security.git
cd ansible-ssh-security
```

### 2. Installer Ansible (si nécessaire)

```bash
# Via pip
pip install ansible

# Via apt (Debian/Ubuntu)
sudo apt update
sudo apt install ansible

# Via dnf (AlmaLinux/RHEL)
sudo dnf install ansible
```

### 3. Installer les collections requises

```bash
ansible-galaxy collection install community.general
ansible-galaxy collection install ansible.posix
```

> ⚠️ **Note** : Les collections doivent être installées dans `~/.ansible/collections/ansible_collections/` ou dans le répertoire du projet.

---

## ⚙️ Configuration

Toutes les variables sont définies dans [`vars/main.yml`](vars/main.yml).

### Variables de configuration SSH

| Variable | Valeur par défaut | Description |
|----------|-------------------|-------------|
| `ssh_port` | `2222` | Nouveau port SSH (n'utilisez pas un port déjà utilisé) |
| `ssh_address` | `"0.0.0.0"` | Adresse d'écoute SSH |

### Variables de configuration Fail2ban

| Variable | Valeur par défaut | Description |
|----------|-------------------|-------------|
| `fail2ban_bantime` | `3600` | Durée du ban en secondes (1 heure) |
| `fail2ban_findtime` | `600` | Fenêtre de détection en secondes (10 minutes) |
| `fail2ban_maxretry` | `5` | Nombre maximum de tentatives avant ban |
| `fail2ban_ignoreip` | `["127.0.0.1/8", "::1"]` | IPs à ne jamais bannir |

### Exemple de personnalisation

```yaml
# vars/main.yml
---
ssh_port: 2244                    # Port SSH personnalisé
fail2ban_bantime: 86400          # Ban de 24 heures
fail2ban_findtime: 300           # Fenêtre de 5 minutes
fail2ban_maxretry: 3             # Max 3 tentatives
fail2ban_ignoreip:
  - "127.0.0.1/8"
  - "::1"
  - "192.168.1.0/24"             # Votre réseau local
```

---

## 📖 Utilisation

### Syntaxe de base

```bash
# Vérifier la syntaxe du playbook
ansible-playbook playbook.yml --syntax-check
```

### Mode "dry-run" (check)

```bash
# Tester sans appliquer les changements
ansible-playbook playbook.yml --check --diff
```

> 💡 **Astuce** : Utilisez `--check --diff` pour prévisualiser les modifications avant de les appliquer.

### Exécution normale

```bash
# Exécuter le playbook (applique toutes les tâches)
ansible-playbook playbook.yml
```

### Avec variables personnalisées

```bash
# Changer le port SSH via ligne de commande
ansible-playbook playbook.yml -e "ssh_port=2244"

# Combiner plusieurs variables
ansible-playbook playbook.yml -e "ssh_port=2244" -e "fail2ban_bantime=7200"

# Lister toutes les tâches avant exécution
ansible-playbook playbook.yml --list-tasks
```

### Exemples concrets

```bash
# === Exemple 1 : Exécution standard ===
ansible-playbook playbook.yml

# === Exemple 2 : Test préliminaire ===
ansible-playbook playbook.yml --check --diff

# === Exemple 3 : Port SSH personnalisé ===
ansible-playbook playbook.yml -e "ssh_port=2244"

# === Exemple 4 : Protection renforcée ===
ansible-playbook playbook.yml \
  -e "fail2ban_bantime=86400" \
  -e "fail2ban_maxretry=3"

# === Exemple 5 : Liste des tâches ===
ansible-playbook playbook.yml --list-tasks
```

---

## 🔄 Workflow du playbook

```
┌─────────────────────────────────────────────────────────────────┐
│                     WORKFLOW DU PLAYBOOK                        │
└─────────────────────────────────────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│  1️⃣  PRÉ-TÂCHES                                                │
│  • Affichage des variables de configuration                     │
└─────────────────────────────────────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│  2️⃣  MISE À JOUR DES PAQUETS                                  │
│  • apt update + upgrade (Debian)                                │
│  • dnf update (RHEL/AlmaLinux)                                  │
└─────────────────────────────────────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│  3️⃣  SAUVEGARDE SSH                                           │
│  • Création du répertoire /backup/ssh                          │
│  • Copie de /etc/ssh/sshd_config → /backup/ssh/                │
└─────────────────────────────────────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│  4️⃣  CONFIGURATION SSH                                        │
│  • Déploiement du template sshd_config.j2                       │
│  • Validation avec sshd -t                                      │
│  • Déclenchement du handler "Restart SSH"                       │
└─────────────────────────────────────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│  5️⃣  FIREWALL (RHEL uniquement)                               │
│  • Installation de firewalld                                    │
│  • Démarrage et activation du service                            │
│  • Ouverture du port SSH personnalisé                           │
└─────────────────────────────────────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│  6️⃣  SELINUX (RHEL uniquement)                                │
│  • Configuration du port SSH avec semanage                      │
└─────────────────────────────────────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│  7️⃣  INSTALLATION FAIL2BAN                                    │
│  • Installation du package fail2ban                             │
│  • Déploiement du template jail.local.j2                        │
│  • Déclenchement du handler "Restart Fail2ban"                   │
└─────────────────────────────────────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│  8️⃣  VÉRIFICATIONS FINALES                                   │
│  • Vérification du port SSH à l'écoute                         │
│  • Vérification du statut Fail2ban                             │
│  • Affichage du résumé de configuration                         │
└─────────────────────────────────────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│  ✅ PLAYBOOK TERMINÉ                                          │
│  • Affichage du banner récapitulatif                            │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🐧 Comportement par distribution

| Étape | Debian/Ubuntu | AlmaLinux/RHEL/Rocky |
|-------|---------------|----------------------|
| **Mise à jour** | `apt update && apt upgrade` | `dnf update` |
| **Paquet Fail2ban** | `fail2ban` | `fail2ban` + `fail2ban-firewalld` |
| **Firewall** | Non configuré (optionnel) | `firewalld` configuré |
| **SELinux** | N/A | Ports SSH configurés |
| **Logs SSH** | `/var/log/auth.log` | `/var/log/secure` |

### Détails des différences

**🐧 Debian/Ubuntu :**
- Utilise `apt` pour la gestion des paquets
- Logs dans `/var/log/auth.log`
- Aucune configuration firewall par défaut
- Fail2ban utilise `iptables-multiport`

**🔴 AlmaLinux/RHEL :**
- Utilise `dnf` pour la gestion des paquets
- Logs dans `/var/log/secure`
- `firewalld` installé et configuré automatiquement
- SELinux: ports SSH configurés via `semanage`
- Fail2ban utilise `firewalld` comme banaction

---

## ⚠️ Sécurité - Notes importantes

> ⚠️ **AVERTISSEMENT CRITIQUE** : Lisez attentivement cette section avant d'exécuter le playbook.

### 🛑 Règles de sécurité

| ⚠️ Règle |理由/Raison |
|----------|------------|
| **Gardez un terminal ouvert** | Après le changement de port, votre session SSH actuelle peut être coupée |
| **Méfiez-vous du port choisi** | N'utilisez pas un port déjà utilisé par un autre service |
| **Configurez votre client SSH** | Mettez à jour votre fichier `~/.ssh/config` avec le nouveau port |
| **Ne perdez pas le nouveau port !** | Notez-le quelque part en sécurité |

### 💾 Sauvegardes automatiques

Le playbook crée automatiquement une sauvegarde de votre configuration SSH :

```
/backup/ssh/sshd_config.backup.[timestamp_epoch]
```

Pour restaurer une sauvegarde :

```bash
# Lister les sauvegardes
ls -la /backup/ssh/

# Restaurer une sauvegarde
sudo cp /backup/ssh/sshd_config.backup.[timestamp] /etc/ssh/sshd_config
sudo systemctl restart sshd
```

### 🔐 Paramètres de sécurité appliqués

| Paramètre | Valeur | Description |
|-----------|--------|-------------|
| `Port` | Configurable (défaut: 2222) | Port non standard |
| `X11Forwarding` | `no` | Désactivé |
| `AllowTcpForwarding` | `no` | Désactivé |
| `AllowAgentForwarding` | `no` | Désactivé |
| `PermitTunnel` | `no` | Désactivé |
| `GatewayPorts` | `no` | Désactivé |
| `MaxAuthTries` | `3` | Limité à 3 tentatives |
| `ClientAliveInterval` | `180` | Déconnexion après 3 min d'inactivité |

---

## 🔧 Dépannage

### ❌ Problèmes courants

| Problème | Cause probable | Solution |
|---------|----------------|----------|
| **Erreur "port déjà utilisé"** | Le port choisi est occupé | Vérifiez avec `ss -tulpn \| grep <port>` |
| **Échec connection après modif** | Port incorrect ou firewall | Restore backup + vérifiez la config |
| **fail2ban ne démarre pas** | Erreur de syntaxe dans jail.local | Vérifiez avec `fail2ban-client -d` |
| **Permission denied** | Pas de droits sudo | Exécutez avec les bons privilèges |
| **SELinux error** | Contexte incorrect | `restorecon /etc/ssh/sshd_config` |

### 🔍 Commandes de diagnostic

```bash
# Vérifier le port SSH à l'écoute
ss -tulpn | grep sshd

# Tester la configuration SSH
sudo sshd -t

# Statut de Fail2ban
sudo fail2ban-client status

# Logs de Fail2ban
sudo tail -f /var/log/fail2ban.log

# Statut du firewall (RHEL)
sudo firewall-cmd --list-all

# Statut SELinux
sudo sestatus
sudo semanage port -l | grep ssh
```

### 🔄 Restauration d'urgence

```bash
# 1. Identifier la dernière sauvegarde
ls -lt /backup/ssh/

# 2. Restaurer la configuration
sudo cp /backup/ssh/sshd_config.backup.[timestamp] /etc/ssh/sshd_config

# 3. Redémarrer SSH
sudo systemctl restart sshd

# 4. Tester la connexion sur le port standard
ssh -p 22 user@localhost
```

### 🆘 Si vous êtes bloqué

Si vous êtes verrouillé hors du serveur après modification du port :

1. **Accédez physiquement** à la machine ou via console
2. ** Restaurez la sauvegarde** :
   ```bash
   sudo cp /backup/ssh/sshd_config.backup.* /etc/ssh/sshd_config
   sudo systemctl restart sshd
   ```
3. **Vérifiez le port** dans `/etc/ssh/sshd_config`
4. **Reconnectez-vous** sur le port configuré

---

## 📄 Licence

```
MIT License

Copyright (c) 2024 Ansible SSH Security Playbook
Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

---

<div align="center">

**⭐ N'oubliez pas de starifier ce projet si il vous a été utile !**

Fait avec ❤️ et Ansible

</div>