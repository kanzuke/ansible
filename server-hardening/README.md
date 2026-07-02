# CrowdSec Ansible Playbook

> Playbook Ansible pour sécuriser des serveurs Linux avec CrowdSec (Debian/Ubuntu & AlmaLinux/RHEL).

**Ansible** ≥ 2.10 · **CrowdSec** latest · **Licence** MIT

---

## Table des matières

1. [Fonctionnalités](#fonctionnalités)
2. [Structure du projet](#structure-du-projet)
3. [Prérequis](#prérequis)
4. [Installation](#installation)
5. [Variables disponibles](#variables-disponibles)
6. [Utilisation](#utilisation)
7. [Commandes de vérification post-déploiement](#commandes-de-vérification-post-déploiement)
8. [Limitations connues](#limitations-connues)
9. [Roadmap](#roadmap)
10. [Licence](#licence)

---

## Fonctionnalités

- **Multiplateforme** : fonctionne sur Debian/Ubuntu (apt) et AlmaLinux/RHEL (dnf)
- **Détection automatique de l'OS** via `ansible_facts['os_family']`
- **Installation du moteur CrowdSec** via le script officiel `install.crowdsec.net`
- **Collections installées par défaut** :
  - `crowdsecurity/linux` — protection de base
  - `crowdsecurity/sshd` — protection SSH (si sshd détecté sur le système)
- **Collections supplémentaires optionnelles** via `crowdsec_extra_collections`
- **Bouncer firewall iptables** (`crowdsec-firewall-bouncer-iptables`)
- **Protection Nginx optionnelle** via `crowdsec_enable_nginx`
- **Durée de ban configurable** (`crowdsec_ban_duration`, défaut : `4h`)
- **Enrôlement console CrowdSec** optionnel (app.crowdsec.net)
- **Exécution en localhost** (`connection: local`)

---

## Structure du projet

```
crowdsec-ansible/
├── README.md
├── playbook.yml
├── group_vars/
│   └── all.yml
└── roles/
    └── crowdsec/
        ├── defaults/
        │   └── main.yml
        ├── handlers/
        │   └── main.yml
        ├── tasks/
        │   ├── main.yml
        │   ├── install-debian.yml
        │   ├── install-almalinux.yml
        │   ├── collections.yml
        │   └── bouncer-firewall.yml
        └── templates/
            └── profiles.yaml.j2
```

---

## Prérequis

- **Ansible** >= 2.10
- **Accès root/sudo** sur la machine cible
- **Accès internet sortant** (téléchargement depuis github.com, api.crowdsec.net)
- **Systèmes supportés** :
  - Debian 11 (Bullseye) / Debian 12 (Bookworm)
  - Ubuntu 20.04 LTS / 22.04 LTS
  - AlmaLinux 8 / AlmaLinux 9
  - RockyLinux 8 / 9

---

## Installation

1. **Cloner le dépôt**
   ```bash
   git clone <url-du-dépot>
   cd crowdsec-ansible
   ```

2. **Vérifier la configuration Ansible**
   ```bash
   ansible --version
   ```

3. **Lancer le playbook**
   ```bash
   ansible-playbook playbook.yml --ask-become-pass
   ```

---

## Variables disponibles

| Variable | Défaut | Description |
|---|---|---|
| `crowdsec_enable_nginx` | `false` | Installer la collection Nginx et le bouncer Nginx |
| `crowdsec_enroll_console` | `false` | Enrôler l'instance sur la console CrowdSec |
| `crowdsec_enroll_key` | `""` | Clé d'enrôlement (obtenue sur app.crowdsec.net) |
| `crowdsec_ban_duration` | `"4h"` | Durée par défaut des bans |
| `crowdsec_extra_collections` | `[]` | Liste de collections supplémentaires à installer |

### Exemple dans `group_vars/all.yml`

```yaml
---
crowdsec_enable_nginx: true
crowdsec_enroll_console: false
crowdsec_enroll_key: ""
crowdsec_ban_duration: "8h"
crowdsec_extra_collections:
  - crowdsecurity/http-cve
```

---

## Utilisation

### Installation standard (localhost)

```bash
ansible-playbook playbook.yml --ask-become-pass
```

### Activer la protection Nginx

```bash
ansible-playbook playbook.yml --ask-become-pass \
  -e crowdsec_enable_nginx=true
```

### Changer la durée de ban

```bash
ansible-playbook playbook.yml --ask-become-pass \
  -e crowdsec_ban_duration="24h"
```

### Enrôler sur la console CrowdSec

```bash
ansible-playbook playbook.yml --ask-become-pass \
  -e crowdsec_enroll_console=true \
  -e crowdsec_enroll_key="votre_cle_d_enrollment"
```

### Installer une collection supplémentaire

```bash
ansible-playbook playbook.yml --ask-become-pass \
  -e crowdsec_extra_collections='["crowdsecurity/http-cve", "crowdsecurity/mysql"]'
```

---

## Commandes de vérification post-déploiement

### Statuts des services

```bash
systemctl status crowdsec
systemctl status crowdsec-firewall-bouncer
```

### Métriques CrowdSec

```bash
cscli metrics
```

### Liste des collections installées

```bash
cscli collections list
```

### Liste des bouncers actifs

```bash
cscli bouncers list
```

### Liste des décisions (IPs bloquées)

```bash
cscli decisions list
```

### Vérifier les logs en cas de problème

```bash
journalctl -u crowdsec -f
journalctl -u crowdsec-firewall-bouncer -f
```

---

## Limitations connues

- **Bouncer firewall** : uniquement iptables pour l'instant. La détection automatique des backends (nftables, firewalld) n'est pas encore implémentée.
- **SELinux** : pas de gestion spécifique des politiques SELinux. Les contexts et booléens doivent être configurés manuellement si applicable.
- **Gestion des services** : le playbook ne redémarre pas automatiquement les services après modification des configurations — utiliser les handlers si besoin.

---

## Roadmap

- [ ] Détection automatique du backend firewall (iptables / nftables / firewalld)
- [ ] Gestion dynamique de firewalld
- [ ] Support nftables pour le bouncer
- [ ] Gestion des politiques SELinux
- [ ] Tests automatisés (Molecule / testinfra)
- [ ] Rôle Ansible Galaxy prêt à l'emploi

---

## Licence

> **MIT License** — Ce projet est fourni tel quel, sans garantie. Voir le fichier [LICENSE](LICENSE) pour les détails.
