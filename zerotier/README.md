# Playbook Ansible : Installation et configuration de ZeroTier

Ce playbook Ansible automatise l'installation et la configuration de [ZeroTier One](https://www.zerotier.com/) sur les distributions Linux suivantes :
- Raspberry Pi OS (32/64 bits)
- Debian / Ubuntu
- Fedora / AlmaLinux

## 📋 Prérequis

- **Ansible** installé sur votre machine de contrôle
- **Accès sudo** sur la machine cible
- **ID du réseau ZeroTier** que vous souhaitez rejoindre (disponible dans [my.zerotier.com](https://my.zerotier.com))

## 🚀 Utilisation

### 1. Cloner le dépôt (ou copier les fichiers)

```bash
git clone https://votre-depot.com/zerotier-ansible.git
cd zerotier-ansible