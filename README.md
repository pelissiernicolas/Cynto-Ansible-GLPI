# Cynto - Déploiement GLPI avec LDAPs via Ansible

## Présentation

Ce projet propose un déploiement automatisé de la solution GLPI à l’aide d’Ansible.

**GLPI** est une solution open source de gestion de parc informatique et de service desk (ITIL), permettant la gestion des actifs, des tickets, des incidents et des demandes utilisateurs.

### Objectifs de l’automatisation

- Installation de GLPI
- Configuration du serveur web (Apache/PHP)
- Installation et configuration de MariaDB
- Configuration de PHP
- Intégration avec Active Directory (optionnelle)
- Gestion sécurisée des secrets avec Ansible Vault
## Prérequis

### Machine de contrôle
- Ansible installé
- Accès SSH au(x) serveur(s) cible(s)
### Serveur GLPI
- Debian ou Ubuntu recommandé
- Accès root ou sudo
- Connexion réseau vers Active Directory (si LDAP utilisé)
### Active Directory (optionnel)

- Déployé via le projet [Cynto-Ansible-AD](https://github.com/pelissiernicolas/Cynto-Ansible-AD)
- LDAP ou LDAPS accessible
## Préparation du serveur cible

Aucune installation préalable n’est requise sur le serveur cible : tout est automatisé par Ansible. Vérifier simplement l’accessibilité SSH :

```bash
ssh user@ip_du_serveur
```
## Structure du projet

```
Cynto-Ansible-GLPI/
├── inventories/
│   └── prod/
│       ├── group_vars/
│       │   ├── glpi/
│       │   │   └── vault.yml
│       │   └── ad/
│       │       └── vault.yml
├── playbooks/
├── roles/
└── README.md
```
## Gestion des secrets avec Ansible Vault

Les informations sensibles (mots de passe, comptes de service, etc.) sont stockées dans des fichiers Vault séparés pour chaque composant.
### 1. Création du Vault GLPI

```bash
ansible-vault create inventories/prod/group_vars/glpi/vault.yml
```

Exemple de contenu :

```yaml
## Variables sensibles pour GLPI ##
### var d'authentification SSH pour Ansible ###
vault_ansible_ssh_password: "xxx"
vault_ansible_become_password: "xxx"

#### var de la base de données ###
vault_mariadb_root_password: "xxx"
vault_glpi_db_password: "xxx"

### var postinstallation de GLPI ###
vault_glpi_admin_password: "xxx"
vault_glpi_postonly_password: "xxx"
vault_glpi_tech_password: "xxx"
vault_glpi_normal_password: "xxx"

### var pour l'intégration LDAP ##
vault_service_account_password: "xxx"
```
### 2. Création du Vault Active Directory

```bash
ansible-vault create inventories/prod/group_vars/ad/vault.yml
```

Exemple de contenu :

```yaml
vault_windows_admin_password: "xxx"
```
### 3. Consultation et modification des Vaults

```bash
ansible-vault view inventories/prod/group_vars/glpi/vault.yml
ansible-vault edit inventories/prod/group_vars/glpi/vault.yml
ansible-vault view inventories/prod/group_vars/ad/vault.yml
ansible-vault edit inventories/prod/group_vars/ad/vault.yml
```
## Configuration de l’inventaire Ansible

Exemple de fichier d’inventaire :

```ini
[glpi]
srv-glpi ansible_host=10.8.40.20

[ad]
srv-ad-01 ansible_host=10.8.40.10
srv-ad-02 ansible_host=10.8.40.11
```
## Vérification des connexions

- **Test SSH (GLPI) :**
  ```
  ansible glpi -m ping
  ```
- **Test WinRM (AD, si utilisé) :**
  ```
  ansible ad -m win_ping
  ```

## Déploiement

Lancer le déploiement complet :

```bash
ansible-playbook playbooks/00-site.yml --ask-vault-pass
```

## Fonctionnalités du playbook

- Installation et configuration automatique de GLPI
- Déploiement et sécurisation de la base de données MariaDB
- Installation et configuration du serveur web (Apache/PHP)
- Configuration de l’application GLPI
- Intégration LDAP/Active Directory (optionnelle)
### Fonctionnalités GLPI après déploiement
- Gestion des équipements IT
- Gestion des tickets (support)
- Gestion des incidents et changements
- Suivi des licences et inventaire
## Intégration Active Directory (optionnelle)

Si configurée, l’intégration AD permet :

- Authentification LDAP/AD
- Synchronisation des utilisateurs AD
- Gestion centralisée des comptes
**Pré-requis :**

- Compte de service LDAP dédié
- Accès réseau entre GLPI et AD
## Sécurité

- Ne jamais stocker de mots de passe en clair
- Utiliser systématiquement Ansible Vault
- En production :
    - Activer HTTPS sur le serveur web
    - Utiliser LDAPS pour l’annuaire
    - Restreindre les accès réseau (firewall)
    - Sécuriser MariaDB (comptes, droits, accès réseau)
## Maintenabilité

- Architecture modulaire basée sur des rôles Ansible
- Centralisation des variables et secrets
- Déploiement reproductible et versionné
- Facilité d’évolution (plugins, haute disponibilité, multi-site)

## Dépendances recommandées

- [Cynto-Ansible-AD](https://github.com/pelissiernicolas/Cynto-Ansible-AD) pour la gestion des identités et l’intégration Active Directory
## Auteur

Nicolas Pelissier
