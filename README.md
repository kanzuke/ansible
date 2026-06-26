# 🧰 Ansible Playbooks — Infrastructure as Code

[![Ansible](https://img.shields.io/badge/Ansible-≥2.12-EE0000?logo=ansible&style=flat-square)](https://docs.ansible.com/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg?style=flat-square)](LICENSE)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg?style=flat-square)](CONTRIBUTING.md)
[![Maintained](https://img.shields.io/badge/Maintained-yes-brightgreen.svg?style=flat-square)]()

> Collection personnelle de playbooks Ansible pour automatiser et sécuriser l'administration système sur serveurs Linux (Debian / AlmaLinux / Rocky Linux / RHEL).

---

## 📖 Table des matières

- [Prérequis](#-prérequis)
- [Conventions & Structure](#-conventions--structure)
- [Utilisation](#-utilisation)
- [Bonnes pratiques](#-bonnes-pratiques)
- [Contribuer & Tester](#-contribuer--tester)
- [Licence](#-licence)

---

## 🛠 Prérequis

| Outil | Version minimale | Rôle |
|---|---|---|
| **Ansible** | ≥ 2.12 | Moteur d'exécution des playbooks |
| **Python 3** | 3.8+ | Interpréteur côté contrôle |
| **pip** | — | Installation des dépendances Python |
| **ansible-lint** *(optionnel)* | — | Linting statique des playbooks |

### 📦 Installation rapide

```bash
# 1. Installer Ansible et ses dépendances
python3 -m pip install --upgrade ansible ansible-lint

# 2. Installer la collection community.general (modules essentiels)
ansible-galaxy collection install community.general

# 3. Vérifier que tout est opérationnel
ansible --version
ansible-doc --list | head -20
```

> **Note** : certains playbooks peuvent nécessiter des collections supplémentaires. Reportez-vous au `README.md` propre à chaque playbook pour les dépendances spécifiques.

---

## 📂 Conventions & Structure

Chaque playbook vit dans son **propre répertoire** à la racine du dépôt. Cette isolation permet de comprendre, tester et faire évoluer chaque automation de manière indépendante.

### 🗂️ Structure type d'un playbook

```
mon-playbook/
├── playbook.yml          # Point d'entrée — déclaration des plays, tâches et handlers
├── vars/
│   └── main.yml          # Variables par défaut (surchargeables via -e)
├── templates/            # Fichiers Jinja2 déployés sur les cibles
├── handlers/
│   └── main.yml          # Handlers (ex : redémarrage d'un service)
├── files/                # Fichiers statiques copurés tels quels
└── README.md             # Documentation spécifique au playbook
```

> **Racine du dépôt** : inventory.ini, `ansible.cfg` ou tout fichier de configuration partagé peuvent être déposés à la racine. Chaque sous-répertoire est autonome et peut être exécuté indépendamment.

### ✏️ Conventions de nommage

| Élément | Convention | Exemple |
|---|---|---|
| Fichier playbook | `kebab-case` descriptif | `ssh-hardening.yml` |
| Répertoire | Même nom que le playbook | `ssh-hardening/` |
| Variables | `snake_case` | `ssh_port`, `fail2ban_bantime` |
| Tags | `snake_case` alignés sur le nom du playbook | `ssh`, `fail2ban` |
| Rôle / handlers | `main.yml` unique ou拆分 multiples | `handlers/main.yml` |

---

## 🚀 Utilisation

### Syntax check

Vérifier la syntaxe d'un playbook **sans exécuter** les tâches :

```bash
ansible-playbook mon-playbook/playbook.yml --syntax-check
```

### Mode dry-run (--check)

Simuler l'exécution **sans modifier** l'état des cibles :

```bash
ansible-playbook mon-playbook/playbook.yml --check
```

### Simuler + afficher les différences (--diff)

```bash
ansible-playbook mon-playbook/playbook.yml --check --diff
```

> 💡 **Astuce** : combinez `--check --diff` pour inspecter chaque modification avant de l'appliquer en production.

### Exécution classique

```bash
# Exécution sur localhost (playbook standard)
ansible-playbook mon-playbook/playbook.yml

# Exécution sur un inventaire spécifique
ansible-playbook -i inventories/prod.yml mon-playbook/playbook.yml
```

### Surcharger des variables (--extra-vars)

```bash
# Surcharger une variable (priorité haute)
ansible-playbook mon-playbook/playbook.yml -e "ssh_port=2222"

# Surcharger depuis un fichier YAML
ansible-playbook mon-playbook/playbook.yml -e "@vars/override.yml"

# Surcharger depuis un fichier JSON
ansible-playbook mon-playbook/playbook.yml -e "@vars/override.json"
```

---

## ✅ Bonnes pratiques

### 🔁 Idempotence

Chaque playbook est conçu pour être **ré-exécutable sans effet de bord** :
- Les services sont redémarrés uniquement via des **handlers** (déclenchés uniquement en cas de changement).
- Les modifications de configuration utilisent `changed_when: no` ou des conditions `when:` pour éviter les faux positifs.

### 🔩 Modules FQCN (Fully Qualified Collection Names)

Tous les modules sont appelés avec leur **espace de noms complet** :

```yaml
# ✅ Correct — FQCN explicite
- ansible.builtin.package:
    name: fail2ban
    state: present

# ❌ À éviter — forme courte (peut créer des conflits)
- package:
    name: fail2ban
    state: present
```

### 🔌 Connexion locale (`connection: local`)

Les playbooks conçus pour un **localhost** utilisent :

```yaml
- hosts: localhost
  connection: local
  become: true          # Élévation de privilèges (sudo/root)
```

### 🐧 Gestion multi-distributions

Détection de la distribution pour adapter les commandes :

```yaml
- name: Installer le paquet Fail2Ban
  ansible.builtin.package:
    name: fail2ban
    state: present
  when: ansible_distribution in ['Debian', 'Ubuntu']

- name: Installer EPEL puis Fail2Ban (AlmaLinux / Rocky / RHEL)
  ansible.builtin.package:
    name: epel-release
    state: present
  when: ansible_distribution in ['AlmaLinux', 'Rocky', 'RedHat']
```

### 🔄 Handlers pour les redémarrages

Les modifications de configuration qui nécessitent un redémarrage passent par des **handlers** déclarés dans `handlers/main.yml` et invoqués avec `notify:` :

```yaml
# Tâche
- name: Configurer SSHD
  ansible.builtin.template:
    src: sshd_config.j2
    dest: /etc/ssh/sshd_config
    validate: /usr/sbin/sshd -t
  notify: restart sshd

# Handler
- name: restart sshd
  ansible.builtin.service:
    name: sshd
    state: restarted
```

> **Important** : le validateur `/usr/sbin/sshd -t` est utilisé pour syntax-check le fichier de config SSH avant son déploiement, évitant un redémarrage raté.

### 🗃️ Variables centralisées

Toutes les valeurs configurable sont déclarées dans `vars/main.yml` à la racine du playbook :

```yaml
# vars/main.yml
---
ssh_port: 22
ssh_permitrootlogin: "no"
fail2ban_bantime: 600
fail2ban_findtime: 600
fail2ban_maxretry: 5
```

Cela permet de les surcharger simplement avec `-e` sans toucher au code.

---

## 🤝 Contribuer & Tester

### Linting avec ansible-lint

```bash
# Linter un playbook entier
ansible-lint mon-playbook/playbook.yml

# Linter avec correction automatique (si possible)
ansible-lint mon-playbook/playbook.yml --fix
```

> Linting est **requis** pour tout pull request. Les règles suivent le profile `default` d'`ansible-lint`.

### Tester localement avec --check

```bash
# 1. Syntaxe OK ?
ansible-playbook mon-playbook/playbook.yml --syntax-check

# 2. Simulation OK ?
ansible-playbook mon-playbook/playbook.yml --check --diff

# 3. Tout est vert → exécution réelle
ansible-playbook mon-playbook/playbook.yml
```

### 📋 Checklist avant PR

- [ ] `ansible-playbook --syntax-check` passe sans erreur
- [ ] `ansible-lint` passe sans avertissements bloquants
- [ ] `--check --diff` ne révèle aucune modification inattendue
- [ ] Un `README.md` documente le playbook (objectif, variables, usage)
- [ ] Les variables sont documentées dans `vars/main.yml` (commentaires en anglais)

---

## 📄 Licence

Ce projet est distribué sous la licence **MIT**. Voir le fichier [`LICENSE`](LICENSE) pour les détails.

---

*Généré automatiquement · Format : Markdown · Style : GitHub-friendly*
