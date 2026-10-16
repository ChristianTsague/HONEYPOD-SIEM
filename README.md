# 🛡️ Mise en place d'un Honeypot sécurisé combiné à un SIEM

**Projet de fin d'année (PFA) — Réseaux et Cybersécurité**

## 📌 Présentation du projet

Ce projet consiste à mettre en place une infrastructure de cybersécurité combinant un **honeypot et un SIEM (Security Information and Event Management)** afin de détecter, collecter, centraliser et analyser les événements liés aux activités suspectes sur un réseau.

L'objectif principal est de déployer un environnement capable d'attirer des tentatives d'intrusion dans un espace contrôlé, de collecter les traces laissées par les attaquants et de corréler les événements de sécurité afin de faciliter leur détection et leur analyse.

La solution repose sur une architecture virtualisée intégrant plusieurs composants complémentaires : un pare-feu, un système de détection d'intrusion, un honeypot et une plateforme de supervision de sécurité.

Ce travail a été réalisé dans le cadre de mon projet de fin d'année et a fait l'objet d'une soutenance académique.

## 🎯 Objectifs

- Déployer une infrastructure réseau segmentée et sécurisée.
- Mettre en place un honeypot pour observer les tentatives d'intrusion.
- Centraliser et analyser les événements de sécurité à l'aide d'un SIEM.
- Surveiller les activités suspectes provenant du réseau.
- Détecter les comportements potentiellement malveillants grâce aux règles et alertes de sécurité.
- Renforcer la visibilité sur les événements affectant les systèmes surveillés.
- Explorer l'intégration de plusieurs outils open source dans une architecture de supervision de sécurité.

## 🏗️ Architecture technique

L'environnement a été conçu autour de plusieurs machines virtuelles assurant des fonctions distinctes.

| Composant | Fonction |
|---|---|
| **pfSense** | Pare-feu, filtrage du trafic et segmentation réseau |
| **Suricata** | Détection des activités réseau suspectes et des signatures d'attaque |
| **Cowrie** | Honeypot destiné à observer les tentatives d'accès et les interactions avec les services SSH/Telnet simulés |
| **Wazuh Manager** | Collecte, analyse et gestion des événements de sécurité |
| **Wazuh Indexer** | Indexation et stockage des données de sécurité |
| **Wazuh Dashboard** | Visualisation des événements, alertes et données de supervision |
| **Ubuntu Server** | Hébergement des composants de supervision |
| **Windows Server / Active Directory** | Environnement Windows intégré à l'infrastructure |
| **Kali Linux** | Machine utilisée pour les tests de sécurité dans le laboratoire |

L'infrastructure a été déployée sous VMware, avec une segmentation réseau permettant de séparer les différents rôles et de contrôler les communications entre les zones.

## ⚙️ Fonctionnalités mises en œuvre

### 1. Détection et observation des intrusions

Le honeypot Cowrie a été déployé dans la zone démilitarisée (DMZ) afin d'exposer un service leurre et d'observer les tentatives de connexion ainsi que les interactions avec celui-ci.

### 2. Supervision centralisée avec Wazuh

La plateforme Wazuh a été déployée pour centraliser et analyser les événements provenant des systèmes surveillés, notamment à travers des agents Linux et Windows.

### 3. Surveillance du réseau avec Suricata

Suricata a été intégré à l'infrastructure pour contribuer à la détection des activités réseau suspectes au niveau de la DMZ.

### 4. Intégration des journaux du pare-feu

La collecte des journaux de pfSense a été configurée afin d'améliorer la visibilité sur les événements réseau et les communications autorisées ou bloquées.

### 5. Détection des modifications de fichiers

La surveillance d'intégrité des fichiers (FIM) de Wazuh a été configurée pour suivre les modifications de répertoires sensibles.

### 6. Enrichissement des alertes

Une intégration avec VirusTotal a été explorée pour enrichir certaines investigations à partir de renseignements sur les fichiers ou indicateurs suspects.

### 7. Notifications de sécurité

Un mécanisme de notification par courrier électronique a été configuré pour transmettre certaines alertes de sécurité.

## 🔬 Environnement de test

Le laboratoire a permis de tester la visibilité du honeypot depuis une machine de test, la collecte des événements, la remontée des journaux et la supervision des systèmes intégrés.

Les expérimentations ont été effectuées dans un environnement virtualisé contrôlé, à des fins pédagogiques et de validation technique.

## 🧰 Technologies utilisées

- **Virtualisation :** VMware
- **Pare-feu :** pfSense
- **Détection réseau :** Suricata
- **Honeypot :** Cowrie
- **SIEM / XDR :** Wazuh
- **Systèmes :** Ubuntu Server, Windows Server, Kali Linux
- **Analyse et supervision :** Wazuh Dashboard
- **Renseignement sur les menaces :** VirusTotal
- **Annuaire et gestion des identités :** Active Directory

## 📈 Résultats et apports

Ce projet m'a permis de travailler sur plusieurs aspects concrets de la cybersécurité :

- La conception et la segmentation d'une infrastructure réseau.
- Le déploiement et la configuration d'un SIEM open source.
- La collecte et l'analyse des journaux de sécurité.
- L'intégration de mécanismes de détection réseau et système.
- La compréhension du rôle des honeypots dans l'observation des comportements malveillants.
- L'intégration de différents composants de sécurité dans une architecture cohérente.

## 🚧 Limites et perspectives d'amélioration

Le projet a atteint une étape fonctionnelle significative et a été soutenu dans le cadre de mon parcours académique. Toutefois, certaines améliorations restent envisageables :

- Automatiser davantage le déploiement et la configuration des composants.
- Approfondir la corrélation entre les événements Cowrie, Suricata, pfSense et Wazuh.
- Ajouter des tableaux de bord personnalisés pour faciliter l'analyse des incidents.
- Améliorer l'enrichissement des alertes avec des sources de renseignement sur les menaces.
- Étudier l'intégration de notifications supplémentaires, notamment par SMS.
- Renforcer les tests de validation et documenter davantage les résultats expérimentaux.
- Évaluer les possibilités d'automatisation de certaines tâches de réponse aux incidents.

Ces perspectives constituent des pistes d'évolution et ne doivent pas être interprétées comme des fonctionnalités déjà implémentées.

## 🎓 Contexte académique

Ce travail a été réalisé dans le cadre de mon projet de fin d'année en réseaux et sécurité informatique.

Il m'a permis de mettre en pratique des connaissances en administration système, sécurité réseau, détection d'intrusion et supervision des événements de sécurité.

## ⚠️ Avertissement

Ce projet est destiné à l'apprentissage, à la recherche et à l'expérimentation en cybersécurité.

Toute reproduction de l'infrastructure ou réalisation de tests doit s'effectuer dans un environnement contrôlé et avec les autorisations nécessaires.

## 👨‍💻 Auteur

**Christian Tsague @ G1M1**

Étudiant et praticien en réseaux, systèmes et cybersécurité.

Centres d'intérêt :
- Sécurité des réseaux et des systèmes
- Détection et analyse des incidents
- SIEM et supervision de sécurité
- Honeypots et threat intelligence
- Infrastructure informatique et sécurité défensive

*Projet académique — Documentation publiée à des fins de partage de connaissances et de valorisation des travaux pratiques en cybersécurité.*
