# 🛡️ ansible-ssh-security

> Playbook Ansible pour durcir SSH et surveiller les tentatives d'accès non autorisées sur Debian / AlmaLinux.

[![Ansible](https://img.shields.io/badge/Ansible-2.12+-EE0000?style=flat-square&logo=ansible&logoColor=white)](https://docs.ansible.com/)
[![Debian](https://img.shields.io/badge/Debian-12-AA0033?style=flat-square&logo=debian&logoColor=white)](https://www.debian.org/)
[![AlmaLinux](https://img.shields.io/badge/AlmaLinux-9-0F3D5B?style=flat-square&logo=almalinux&logoColor=white)](https://almalinux.org/)
[![MIT](https://img.shields.io/badge/License-MIT-green?style=flat-square)](https://opensource.org/licenses/MIT)

---

## 📖 Table des matières

- [🛡️ ansible-ssh-security](#-ansible-ssh-security)
- [📖 Table des matières](#-table-des-matières)
- [🚀 Fonctionnalités](#-fonctionnalités)
  - [🔐 Rôle SSH — Durcissement](#-rôle-ssh--durcissement)
  - [🚫 Rôle Fail2ban — Protection brute-force](#-rôle-fail2ban--protection-brute-force)
  - [📧 Notifications (optionnelles)](#-notifications-optionnelles)
- [📂 Arborescence](#-arborescence)
- [🛠 Prérequis](#-prérequis)
- [🚀 Installation rapide](#-installation-rapide)
- [📋 Variables disponibles](#-variables-disponibles)
  - [Configuration SSH](#-configuration-ssh)
  - [Configuration Fail2ban](#configuration-fail2ban)
  - [Notifications Mail (msmtp)](#notifications-mail-msmtp)
  - [Notifications Ntfy](#notifications-ntfy)
- [⚡ Utilisation](#-utilisation)
  - [Syntaxe only (pas d'exécution)](#syntaxe-only-pas- dexécution)
  - [Mode simulation (dry-run)](#mode-simulation-dry-run)
  - [Exécution classique](#exécution-classique)
  - [Avec notifications mail](#avec-notifications-mail)
  - [Avec notifications ntfy](#avec-notifications-ntfy)
  - [Les deuxnotifications simultanément](#les-deux-notifications-simultanément)
- [🔒 Gestion des secrets (ansible-vault)](#-gestion-des-secrets-ansible-vault)
- [⚠️ Points de vigilance](#️-points-de-vigilance)
- [🤝 Contribuer \& Tester](#-contribuer--tester)
- [📄 Licence](#-licence)

---

## 🚀 Fonctionnalités

### 🔐 Rôle SSH — Durcissement

| Fonctionnalité | Détail |
|---|---|
| **Port personnalisé** | Déplacement du port SSH par défaut (2222) pour réduire les scans automatisés |
| **Désactivation du forwarding** | TCP, X11, Agent forwarding désactivés |
| **Authentification** | Root login interdit, connexion par clé uniquement recommandée |
| **Validation** | Vérification de la config via `sshd -t` avant application |
| **Firewall auto** | Configuration automatique : UFW (Debian) / firewalld (AlmaLinux) |

### 🚫 Rôle Fail2ban — Protection brute-force

| Fonctionnalité | Détail |
|---|---|
| **Installation** | Fail2ban installé et activé au démarrage |
| **Jail SSH** | Protection contre les attaques brute-force sur SSH |
| **Paramètres ajustables** | Bantime, findtime, maxretry configurables |
| **Backend adaptatif** | Auto (Debian) / systemd (AlmaLinux) |

### 📧 Notifications (optionnelles)

| Méthode | Description |
|---|---|
| **📧 Mail (msmtp)** | Envoi d'emails via SMTP (Gmail, SMTP provider, etc.) |
| **📱 Ntfy** | Notifications push vers un topic [ntfy.sh](https://ntfy.sh/) auto-hébergé ou cloud |

> 💡 Les deux systèmes peuvent être activés simultanément ou indépendamment.

---

## 📂 Arborescence

```
ansible-ssh-security/
├── playbook.yml                  # Point d'entrée
├── inventory.ini                 # Inventaire localhost
├── README.md                     # Ce fichier
├── defaults/
│   └── main.yml                  # Variables par défaut (tout est là)
├── group_vars/
│   └── all.yml                   # Variables de groupe (surchargez vos secrets ici)
└── roles/
    ├── ssh/
    │   ├── tasks/
    │   │   └── main.yml          # Tâches SSH
    │   ├── templates/
    │   │   └── sshd_config.j2    # Template config SSHD
    │   └── handlers/
    │       └── main.yml          # Handler restart sshd
    └── fail2ban/
        ├── tasks/
        │   ├── main.yml          # Tâches principales fail2ban
        │   ├── mail.yml          # Installation & config msmtp
        │   └── ntfy.yml          # Installation & config ntfy
        ├── templates/
        │   ├── jail.local.j2     # Configuration jail fail2ban
        │   ├── msmtprc.j2        # Config msmtp (SMTP)
        │   ├── ntfy-action.conf.j2  # Action fail2ban ntfy
        │   └── ntfy-action.sh.j2    # Script notification ntfy
        └── handlers/
            └── main.yml          # Handler restart fail2ban
```

---

## 🛠 Prérequis

| Outil | Version | Rôle |
|---|---|---|
| **Python 3** | 3.8+ | Interpréteur côté contrôle |
| **Ansible** | 2.12+ | Moteur d'exécution |
| **Collections** | — | `ansible.posix`, `community.general` |
| **Accès sudo** | — | root ou sudo sur la machine cible |

### Distributions supportées

- ✅ **Debian** 12 (Bookworm)
- ✅ **AlmaLinux** 9.x
- ✅ Compatible **Ubuntu** (grâce aux tâches conditionnelles)

---

## 🚀 Installation rapide

```bash
# 1. Cloner / télécharger le playbook
cd ansible-ssh-security

# 2. Installer Ansible et les dépendances
python3 -m pip install --upgrade ansible ansible-lint

ansible-galaxy collection install ansible.posix community.general

# 3. Vérifier l'installation
ansible --version
```

---

## 📋 Variables disponibles

### Configuration SSH

| Variable | Défaut | Description |
|---|---|---|
| `ssh_port` | `2222` | Port SSH alternatif |
| `ssh_permit_root_login` | `"no"` | Interdire connexion root |
| `ssh_password_authentication` | `"no"` | Interdire auth par mot de passe |
| `ssh_pubkey_authentication` | `"yes"` | Authentification par clé |
| `ssh_tcp_forwarding` | `"no"` | Désactiver le tunneling TCP |
| `ssh_x11_forwarding` | `"no"` | Désactiver le renvoi X11 |
| `ssh_agent_forwarding` | `"no"` | Désactiver l'agent forwarding |

### Configuration Fail2ban

| Variable | Défaut | Description |
|---|---|---|
| `fail2ban_bantime` | `3600` | Durée du ban (secondes) |
| `fail2ban_findtime` | `600` | Fenêtre de détection (secondes) |
| `fail2ban_maxretry` | `5` | Nombre max de tentatives |

### Notifications Mail (msmtp)

| Variable | Défaut | Description |
|---|---|---|
| `notifications_mail_enabled` | `false` | Activer les notifications mail |
| `notifications_mail_smtp_host` | _(vide)_ | Hôte SMTP (ex: `smtp.gmail.com`) |
| `notifications_mail_smtp_port` | `587` | Port SMTP (TLS) |
| `notifications_mail_smtp_user` | _(vide)_ | Utilisateur SMTP |
| `notifications_mail_smtp_password` | _(vide)_ | Mot de passe SMTP (**utiliser vault!**) |
| `notifications_mail_smtp_tls` | `true` | Activer TLS |
| `notifications_mail_from` | _(vide)_ | Adresse expéditeur |
| `notifications_mail_to` | _(vide)_ | Adresse destinataire |

### Notifications Ntfy

| Variable | Défaut | Description |
|---|---|---|
| `notifications_ntfy_enabled` | `false` | Activer les notifications ntfy |
| `notifications_ntfy_url` | `https://ntfy.sh` | URL du serveur ntfy |
| `notifications_ntfy_topic` | _(vide)_ | Nom du topic (ex: `mon-serveur-alerts`) |
| `notifications_ntfy_token` | _(vide)_ | Token d'accès (optionnel, requis si topic privé) |
| `notifications_ntfy_priority` | `high` | Priorité du message |
| `notifications_ntfy_tags` | `warning` | Tags emoji (ex: `warning`, `no_entry`) |

---

## ⚡ Utilisation

### Syntaxe only (pas d'exécution)

```bash
ansible-playbook playbook.yml --syntax-check
```

### Mode simulation (dry-run)

```bash
ansible-playbook playbook.yml --check
```

### Exécution classique

```bash
# Configuration SSH + fail2ban basique
ansible-playbook playbook.yml
```

### Avec notifications mail

```bash
ansible-playbook playbook.yml \
  -e "notifications_mail_enabled=true" \
  -e "notifications_mail_smtp_host=smtp.gmail.com" \
  -e "notifications_mail_smtp_user=moi@gmail.com" \
  -e "notifications_mail_smtp_password='motdepasseapp'" \
  -e "notifications_mail_from=moi@gmail.com" \
  -e "notifications_mail_to=alertes@mon-domaine.com"
```

> ⚠️ **Ne jamais mettre de mot de passe en clair sur la ligne de commande** — voir la section [ansible-vault](#-gestion-des-secrets-ansible-vault).

### Avec notifications ntfy

```bash
ansible-playbook playbook.yml \
  -e "notifications_ntfy_enabled=true" \
  -e "notifications_ntfy_topic=mon-serveur-alerts"
```

Pour un topic privé (authentification requise) :

```bash
ansible-playbook playbook.yml \
  -e "notifications_ntfy_enabled=true" \
  -e "notifications_ntfy_topic=mon-serveur-alerts" \
  -e "notifications_ntfy_token=mntfytok_Véry_Long_Token_123"
```

### Les deux notifications simultanément

```bash
ansible-playbook playbook.yml \
  -e "notifications_mail_enabled=true" \
  -e "notifications_mail_smtp_host=smtp.gmail.com" \
  -e "notifications_mail_smtp_user=moi@gmail.com" \
  -e "notifications_mail_smtp_password='xxx'" \
  -e "notifications_mail_from=moi@gmail.com" \
  -e "notifications_mail_to=alertes@exemple.com" \
  -e "notifications_ntfy_enabled=true" \
  -e "notifications_ntfy_topic=mon-serveur-alerts"
```

---

## 🔒 Gestion des secrets (ansible-vault)

Toute variable contenant un secret (mot de passe SMTP, token ntfy, etc.) **doit être chiffrée avec `ansible-vault`**.

### Méthode 1 : fichier vault dédié

```bash
# Créer un fichier vault chiffré
ansible-vault create group_vars/secrets.yml

# Ajouter les variables :
#   notifications_mail_smtp_password: "motdepasse"
#   notifications_ntfy_token: "token_secret"

# Exécuter avec le vault
ansible-playbook playbook.yml --ask-vault-pass
```

### Méthode 2 : variable isolée (encrypt_string)

```bash
# Chiffrer une seule variable
ansible-vault encrypt_string 'motdepasse_smtp' --name 'notifications_mail_smtp_password'

# Copier la sortie dans group_vars/all.yml
```

### Méthode 3 : fichier vault sans demande interactive

```bash
# Créer un fichiervault_pwd.txt (mode 600!)
echo "mon_mot_de_passe_vault" > ~/.vault_pass.txt
chmod 600 ~/.vault_pass.txt

# Exécuter avec le fichier password
ansible-playbook playbook.yml --vault-password-file ~/.vault_pass.txt
```

---

## ⚠️ Points de vigilance

> 🚨 **Avant d'exécuter en production, vérifiez toujours :**

- [ ] Le nouveau port SSH (`ssh_port`) n'est **pas bloqué** par un firewall
- [ ] Vous avez un accès **alternatif** au serveur (console, IPMI, clé USB de secours)
- [ ] Le port SSH personnalisé est bien ouvert dans le firewall **avant** de redémarrer SSHD
- [ ] Vous êtes connecté en **sudo** ou root
- [ ] La syntaxe estOK : `ansible-playbook playbook.yml --syntax-check`
- [ ] La simulation passe : `ansible-playbook playbook.yml --check`
- [ ] Les secrets sont **chiffrés** avec `ansible-vault`, jamais en clair dans les lignes de commande

### En cas de perte d'accès SSH

Si vous perdez l'accès après modification du port ou du firewall :

1. Accéder au serveur via **console/IPMI/série**
2. Vérifier la config SSH : `sshd -t`
3. Vérifier le firewall : `ufw status` / `firewall-cmd --list-all`
4. Corriger ou restaurer `/etc/ssh/sshd_config`
5. Redémarrer SSHD : `systemctl restart sshd`

---

## 🤝 Contribuer & Tester

### Checklist avant exécution

```bash
# 1. Vérifier la syntaxe
ansible-playbook playbook.yml --syntax-check

# 2. Simuler sans appliquer
ansible-playbook playbook.yml --check

# 3. Linter le playbook
ansible-lint playbook.yml

# 4. Si tout est OK → exécuter pour de vrai
ansible-playbook playbook.yml
```

### Variables de test rapide

```bash
# Surcharger des variables facilement
ansible-playbook playbook.yml \
  -e "ssh_port=2222" \
  -e "fail2ban_bantime=3600"
```

---

## 📄 Licence

**MIT** — Utilisez, modifiez, distribuez librement.

---

> 📌 *Ce playbook est fourni tel quel. Testez toujours en environnement de staging avant production.*