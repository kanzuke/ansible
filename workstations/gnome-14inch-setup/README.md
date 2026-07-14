# 🚀 Playbook Ansible : Configuration GNOME pour écran 14"

## 📋 Prérequis
- **Ansible** installé (`sudo dnf install ansible` / `sudo zypper install ansible`)
- **Python** (inclus par défaut sur Fedora/OpenSUSE)
- **GNOME** installé (par défaut sur Fedora/OpenSUSE Workstation)

## 🔧 Exécution

### 1️⃣ Cloner le dépôt (ou copier les fichiers)
```bash
git clone https://github.com/kanzuke/ansible/
cd workstations/gnome-14inch-setup
```

### 2️⃣ Exécuter le playbook
```bash
ansible-playbook playbook.yml -K
```

### 3️⃣ Redémarrer GNOME (optionnel)
Si les changements ne sont pas appliqués immédiatement :
```bash
gnome-shell --replace &
```