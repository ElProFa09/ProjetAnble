# 🌐 Projet NetDevOps : Automatisation Réseau Multi-Vendeurs avec Ansible

> **Auteur :** NGOIE SHABANA GLODIE  
> **Contexte académique :** TP DevNet / Master 2 - DevOps et sécurité  
> **Statut du projet :** Production / Validé ✅

---

## 📖 Table des Matières
1. [Introduction et Vision du Projet](#-1-introduction-et-vision-du-projet)
2. [Prérequis et Installation de l'Environnement](#-2-prérequis-et-installation-de-lenvironnement-station-de-contrôle)
3. [Architecture et Explication Détaillée de l'Arborescence](#-3-architecture-et-explication-détaillée-de-larborescence)
4. [Comment adapter ce projet à votre propre infrastructure ?](#-4-comment-adapter-ce-projet-à-votre-propre-infrastructure)
5. [Guide d'Exécution Pas à Pas (Fichier par Fichier & Commandes)](#-5-guide-dexécution-pas-à-pas-fichier-par-fichier--commandes)
6. [Les Points Majeurs du TP (Points 8, 9 et 10)](#-6-les-points-majeurs-du-tp-points-8-9-et-10)
7. [Conclusion & Bonnes Pratiques](#-7-conclusion--bonnes-pratiques)

---

## 🎯 1. Introduction et Vision du Projet

Dans les infrastructures réseau traditionnelles, la configuration des routeurs et des commutateurs s'effectue manuellement, ligne par ligne, via des sessions SSH ou console. Cette méthode est lente, répétitive et représente une source majeure d'erreurs humaines.

Ce projet met en œuvre l'approche **NetDevOps** (ou *Infrastructure as Code - IaC*) grâce au moteur **Ansible**. L'objectif est de transformer la gestion d'un réseau hétérogène (composé d'équipements de différents constructeurs : Cisco IOS, IOS-XR et NX-OS) en un pipeline automatisé, reproductible, auditable et sécurisé.

---

## 🛠️ 2. Prérequis et Installation de l'Environnement (Station de Contrôle)

Si vous partez d'une machine Linux (Ubuntu/Debian) fraîchement installée pour piloter le réseau, suivez ces étapes préliminaires :

### A. Installer Ansible et les outils système nécessaires
```bash
sudo apt update
sudo apt install -y python3-pip git sshpass
pip3 install ansible paramiko
```

### B. Installer les collections Cisco officielles pour Ansible
Ansible utilise des collections spécifiques pour dialoguer avec les équipements Cisco IOS, IOS-XR et NX-OS. Installez-les via la commande :
```bash
ansible-galaxy collection install cisco.ios cisco.iosxr cisco.nxos
```

---

## 📂 3. Architecture et Explication Détaillée de l'Arborescence

Voici l'organisation exacte de ce dépôt et le rôle précis de chaque composant :

```text
.
├── ansible.cfg                 # Configuration globale d'Ansible (désactive le host_key_checking, définit les paramètres de l'inventaire)
├── inventory                   # Répertoire centralisant la topologie du réseau
│   ├── group_vars              
│   │   └── routers.yml         # Contient les identifiants (username, password) et ports communs à tous les routeurs
│   ├── hosts                   # Fichier d'inventaire statique listant les adresses IP et associant chaque équipement à son OS
│   └── host_vars               # Paramètres spécifiques à chaque routeur individuel
│       ├── 10.10.20.35.yml     # Variables propres à l'IOS-XRv
│       ├── 10.10.20.40.yml     # Variables propres au Nexus N9K
│       └── 10.10.20.48.yml     # Variables propres au CAT8Kv (IOS-XE)
├── playbooks                   # Cœur logique : scripts d'automatisation (Playbooks YAML)
│   ├── backup_config.yml       # Sauvegarde automatique des configurations courantes des routeurs
│   ├── configure_interfaces.yml# Paramétrage, adressage IP et activation des interfaces réseau
│   ├── configure_motd.yml      # Déploiement dynamique de la bannière de sécurité (via Jinja2)
│   ├── configure_ntp.yml       # Synchronisation temporelle (serveur NTP) pour la cohérence des logs
│   ├── configure_snmp.yml      # Uniformisation de la conformité SNMP (communauté, contact, localisation)
│   ├── deploy_vlans.yml        # Création et propagation de la nomenclature des VLANs
│   ├── get_facts.yml           # Inspection de l'état de santé et collecte des caractéristiques des équipements
│   └── manage_users.yml        # Gestion centralisée des comptes administrateurs et accès distants
├── templates                   # Modèles de fichiers dynamiques basés sur le moteur Jinja2
│   ├── motd.j2                 # Modèle de bannière intégrant dynamiquement le nom d'hôte de l'équipement
│   └── report.j2               # Modèle au format Markdown pour structurer le rapport d'exécution final
├── reports                     # Dossier de stockage des livrables d'audit
│   └── rapport_execution.md    # Rapport récapitulatif généré automatiquement attestant du succès du déploiement
└── backups                     # Dossier d'archivage sécurisé contenant les configurations sauvegardées
```

---

## ⚙️ 4. Comment adapter ce projet à votre propre infrastructure ?

Avant d'exécuter le moindre playbook sur votre réseau, vous devez configurer vos équipements dans l'inventaire :

1. **Modifier les adresses IP (`inventory/hosts`) :**
   Ouvrez le fichier `inventory/hosts` et remplacez les adresses IP de démonstration par celles de vos propres équipements (Cisco IOS, IOS-XR, NX-OS).
2. **Configurer les identifiants (`inventory/group_vars/routers.yml`) :**
   Modifiez le nom d'utilisateur (`ansible_user`) et le mot de passe (`ansible_password`) pour qu'ils correspondent aux identifiants administrateurs configurés sur vos routeurs.
3. **Ajuster les variables individuelles (`inventory/host_vars/`) :**
   Si certains équipements ont des ports spécifiques ou des exigences particulières, modifiez les fichiers YAML correspondants.

---

## 🚀 5. Guide d'Exécution Pas à Pas (Fichier par Fichier & Commandes)

Une fois le projet cloné et l'inventaire adapté, placez-vous dans le dossier du projet et suivez l'ordre d'exécution logique. **Chaque playbook dispose de sa commande dédiée.**

### Étape 0 : Cloner le dépôt et se positionner dedans
```bash
git clone <url-de-votre-depot-github>
cd Tp_DevNet
```

### Étape 1 : Valider la connectivité réseau (Test de liaison SSH)
Avant toute modification, testez la communication avec vos équipements :
```bash
ansible all -m ping
```
* *Pourquoi ?* Cette commande vérifie que la station de contrôle rejoint bien les adresses IP et s'authentifie correctement en SSH.

### Étape 2 : Inspecter l'état initial (`get_facts.yml`)
Récupérez les faits techniques et l'état de santé du parc :
```bash
ansible-playbook playbooks/get_facts.yml
```
* *Pourquoi ?* Interroge les routeurs pour s'assurer que les modules de chaque constructeur répondent correctement.

### Étape 3 : Déployer la segmentation et l'adressage (`deploy_vlans.yml` & `configure_interfaces.yml`)
* **Création des VLANs :**
  ```bash
  ansible-playbook playbooks/deploy_vlans.yml
  ```
* **Configuration des interfaces réseau :**
  ```bash
  ansible-playbook playbooks/configure_interfaces.yml
  ```

### Étape 4 : Appliquer les services fondamentaux (`configure_ntp.yml` & `manage_users.yml`)
* **Synchronisation temporelle (NTP) :**
  ```bash
  ansible-playbook playbooks/configure_ntp.yml
  ```
* **Gestion des accès administrateurs :**
  ```bash
  ansible-playbook playbooks/manage_users.yml
  ```

---

## 🎯 6. Les Points Majeurs du TP (Points 8, 9 et 10)

Les playbooks ci-dessous réalisent les actions avancées exigées par le projet :

### 📌 Point 8 : Bannière MOTD Dynamique (`configure_motd.yml`)
* **Rôle :** Utilise le moteur **Jinja2** (`templates/motd.j2`) pour injecter automatiquement le nom d'hôte (`inventory_hostname`) dans le message d'avertissement légal de chaque équipement.
* **Commande d'exécution :**
  ```bash
  ansible-playbook playbooks/configure_motd.yml
  ```

### 📌 Point 9 : Audit et Conformité SNMP (`configure_snmp.yml`)
* **Rôle :** Uniformise la supervision en configurant la communauté SNMP en lecture seule (`RO`), la localisation physique et le contact administrateur.
* **Commande d'exécution :**
  ```bash
  ansible-playbook playbooks/configure_snmp.yml
  ```

### 📌 Point 10 : Sauvegarde et Rapport d'État Final (`backup_config.yml` & `generate_report.yml`)
1. **Sauvegarder les configurations courantes (`running-config`) :**
   ```bash
   ansible-playbook playbooks/backup_config.yml
   ```
2. **Générer le Rapport d'État Final (Markdown) :**
   Exécute un playbook local pour compiler l'état des hôtes et générer un livrable propre.
   ```bash
   ansible-playbook playbooks/generate_report.yml
   ```
   *Le rapport compilé est consultable à tout moment dans :* `reports/rapport_execution.md`

---

## 🏁 7. Conclusion & Bonnes Pratiques

Ce dépôt fournit une solution clé en main d'automatisation **NetDevOps**. Grâce à ce guide complet, n'importe quel ingénieur ou débutant peut cloner le dépôt, adapter l'inventaire, et exécuter l'intégralité du pipeline de configuration réseau sans friction.
