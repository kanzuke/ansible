# 📡 Playbook Ansible — Installation et Configuration de ZeroTier One

Playbook Ansible pour installer et configurer ZeroTier One sur diverses distributions Linux, incluant Raspberry Pi, Fedora, AlmaLinux, Debian et Ubuntu.

---

## 📋 Description

Ce playbook automatise l'installation de **ZeroTier One**, un logiciel de réseau privé virtuel (VPN) overlay, sur différentes distributions Linux. Il détecte automatiquement la distribution et l'architecture du système, puis configure le service pour rejoindre un réseau ZeroTier spécifié par l'utilisateur.

**Fonctionnalités principales :**
- Installation automatique sur plusieurs distributions Linux
- Support natif des architectures x86_64 et ARM (aarch64/arm64)
- Configuration interactive via prompt
- Démarrage et validation du service

---

## ✅ Prérequis

### Système hôte (machine qui exécute le playbook)

| Composant | Version minimale |
|-----------|------------------|
| Python | 3.x |
| Ansible | 2.9+ |
| SSH | OpenSSH |

### Système cible (machine où ZeroTier sera installé)

| Composant | Support |
|-----------|---------|
| **Distribitions supportées** | Raspberry Pi OS, Debian, Ubuntu, Fedora, AlmaLinux |
| **Architectures** | x86_64 (amd64), aarch64/arm64 |
| **Accès** | Utilisateur avec droits sudo |

---

## 🚀 Installation

### 1. Cloner ou télécharger le playbook

```bash
# Copier le fichier du playbook
# install_zerotier.yml

# Rendre le playbook exécutable (optionnel)
chmod +x install_zerotier.yml
```

### 2. Vérifier la connectivité SSH

```bash
# Tester la connexion vers l'hôte cible
ansible all -i <hostname>, -m ping -u <user>
```

### 3. Exécuter le playbook

```bash
# Exécution locale (localhost)
ansible-playbook install_zerotier.yml

# Exécution distante (inventory file)
ansible-playbook install_zerotier.yml -i inventory.ini
```

---

## 🎯 Fonctionnalités

| Fonctionnalité | Description |
|----------------|-------------|
| **Détection automatique** | Identifie la distribution et l'architecture |
| **Multi-distributions** | Support de Raspberry Pi OS, Debian, Ubuntu, Fedora, AlmaLinux |
| **Multi-architectures** | Compatible x86_64 et ARM (Raspberry Pi) |
| **Installation silencieuse** | Pas d'interaction requise pendant l'installation |
| **Configuration réseau** | Joint automatiquement le réseau ZeroTier spécifié |
| **Validation** | Vérifie le statut du service et la joinée au réseau |

---

## 📖 Exemple d'utilisation

### Commande d'exécution

```bash
ansible-playbook install_zerotier.yml
```

### Exemple de sortie

```
PLAY [Installer ZeroTier One sur Linux] ****************************************

TASK [Gathering Facts] *********************************************************
ok: [localhost]

TASK [Détecter la distribution] ***********************************************
ok: [localhost]

TASK [Définir l'architecture du système] **************************************
ok: [localhost]

TASK [Installer ZeroTier One (Debian)] ****************************************
ok: [localhost]

TASK [Redémarrer le service ZeroTier] *****************************************
changed: [localhost]

TASK [Rejoindre le réseau ZeroTier] *******************************************
changed: [localhost]

PLAY RECAP *********************************************************************
localhost                  : ok=6    changed=2    unreachable=0    failed=0
```

---

## 📝 Notes importantes

### ⚠️ Limitations

- **Distribution** : Ce playbook supporte uniquement Raspberry Pi OS, Debian, Ubuntu, Fedora et AlmaLinux. Les autres distributions nécessitent une adaptation manuelle.
- **Réseau** : Vous devez disposer d'un réseau ZeroTier valide (Network ID).
- **Autorisation** : Après installation, le nœud doit être autorisé dans l'interface d'administration ZeroTier Central.

### 💡 Conseils

1. **Obtenir un Network ID** : Créez un compte sur [zerotier.com](https://www.zerotier.com) et créez un réseau pour obtenir votre Network ID.

2. **Autorisation** : Après installation, connectez-vous à ZeroTier Central, allez dans votre réseau, et approuvez le nœud dans la section "Members".

3. **Vérifier le statut** : Vous pouvez vérifier manuellement l'état de ZeroTier avec :

   ```bash
   sudo zerotier-cli status
   sudo zerotier-cli listnetworks
   ```

4. **Logs** : Les journaux sont disponibles via :

   ```bash
   sudo journalctl -u zerotier
   ```

5. **Désinstaller** : Pour supprimer ZeroTier :

   ```bash
   # Debian/Ubuntu
   sudo apt remove zerotier-one
   
   # Fedora/AlmaLinux
   sudo dnf remove zerotier-one
   ```

---

## 🔧 Résolution de problèmes

### Erreur "ModuleNotFoundError"

Si vous rencontrez une erreur liée à `module` ou `dnf`, assurez-vous que le module `dnf` est disponible sur votre système Fedora/AlmaLinux :

```bash
ansible localhost -m dnf -a "name=example"
```

### Échec de jointure au réseau

1. Vérifiez le Network ID saisi
2. Assurez-vous que le nœud est autorisé dans ZeroTier Central
3. Vérifiez la connectivité réseau :

```bash
sudo zerotier-cli status
sudo zerotier-cli listnetworks
```

---

## 📄 Licence

**MIT License**

Copyright (c) 2024

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

---

## 👤 Auteur

**Mistral AI**

---

## 🔗 Ressources

- [Documentation ZeroTier](https://docs.zerotier.com/)
- [ZeroTier Central](https://my.zerotier.com/)
- [Ansible Documentation](https://docs.ansible.com/)