##### instalation de notre SIEM : WAZUH 

ce passage est de loint ma preferer l'installation de notre siem est une etape crusial de notre projet car ces lui qui va collecter et coreller nos logs mais pas que. ont y intergrerat des framework comme `virus total` (c'est grace a lui que notre siem vas pouvoir annalyser les fichier et nous dire si il sont malveillant ou pas), puis `MITRE ATT&CK` qui est une base de donnée comportant les technique, tactique et procedure utiliser par les attaquant bref que des bonne chose 

---

**1. Pré-requis**

Avant de commencer l’installation de Wazuh, assure-toi que les conditions suivantes sont respectées :

**Version et sources officielles**

* Nous allons installer **la version 4.14**, qui est actuellement la **dernière version stable** de Wazuh a l'heure d'aujourd'hui.
* Toutes les commandes et instructions utilisées dans ce guide proviennent du site officiel : [https://documentation.wazuh.com/current/index.html](https://documentation.wazuh.com/current/index.html).

**Méthodes d’installation**

Wazuh peut être installé de deux manières :

    1. Mode tout en un : installe automatiquement le manager, les agents et l’API. Simple mais moins flexible pour une architecture complexe.
    2. Mode étape par étape (recommandé): installe chaque composant séparément (manager, API, agents) pour un contrôle total et une configuration fine.

* **Nous procéderons ici avec le mode étape par étape**, ce qui est idéal pour notre architecture multi-OS et future intégration avec Suricata.

**Serveur Wazuh (Ubuntu Server)**

* Ubuntu 22.04 LTS ou version plus récente.
* Accès root ou utilisateur avec `sudo`.
* Connexion réseau stable.
* Ports ouverts : (pas oubligatoire pour le moment)

  * TCP 1514 → pour la réception des logs des agents.
  * TCP 55000 → pour la communication avec les agents Wazuh.
* Paquets essentiels installés :

```bash
    sudo apt update
    sudo apt install curl wget apt-transport-https gnupg -y
```

**Agent Windows (Windows Server)** (pas necessaire pour le moment)

* Accès Administrateur.
* Firewall autorisant le trafic vers le serveur Wazuh (ports 1514 et 55000).
* Powershell configuré pour exécuter des scripts si nécessaire :

```powershell

  Set-ExecutionPolicy RemoteSigned

```

**Agent Linux (Ubuntu Desktop/DMZ)** (pas necessaire pour le moment)

* Accès root ou utilisateur avec `sudo`.
* Firewall autorisant le trafic vers le serveur Wazuh.

**Préparation pour Suricata (future intégration)**

* Suricata sera installé sur la DMZ ou segment réseau de test. (`deja fait`)
* Les logs de Suricata pourront ensuite être récupérés par Wazuh via syslog ou fichier JSON.


### Installation de Wazuh 4.14 – Mode étape par étape

L’installation du SIEM Wazuh a été réalisée en mode étape par étape, conformément à la documentation officielle. 
Cette méthode, bien que plus longue et fastidieuse, permet une meilleure maîtrise de chaque composant, une sécurisation fine des communications, ainsi qu’une adaptation à une architecture réseau segmentée.

L’installation a été effectuée dans l’ordre suivant :

**Étape 1 : Préparation du système**

    Mise à jour du système Ubuntu Server.

    Installation des dépendances requises (curl, gnupg, apt-transport-https, etc.).

    Vérification de la version du système et de la connectivité réseau.

**Étape 2 : Génération des certificats TLS**

    Téléchargement de l’outil officiel wazuh-certs-tool.sh.

    Configuration du fichier config.yml pour définir les nœuds (Indexer, Manager, Dashboard).

    Génération des certificats TLS nécessaires au chiffrement des communications.

    Distribution et placement des certificats selon les composants Wazuh.

**Étape 3 : Installation et configuration de Wazuh Indexer**

    Installation du paquet Wazuh Indexer.

    Configuration du cluster (node name, réseau, ports).

    Intégration des certificats TLS.

    Démarrage du service et validation du bon fonctionnement du cluster.

**Étape 4 : Installation de Wazuh Manager (Server)**

    Installation du Wazuh Manager.

    Configuration de la communication sécurisée avec Wazuh Indexer.

    Vérification de la connexion au cluster via les logs.

    Démarrage et validation du service Manager.

**Étape 5 : Installation de Wazuh Dashboard**

    Installation du Wazuh Dashboard.

    Configuration de la connexion sécurisée vers Wazuh Indexer à l’aide des certificats.

    Démarrage du service Dashboard.

    Accès à l’interface web pour validation du bon fonctionnement.

**Étape 6 : Vérifications finales**

    Vérification de l’état des services :

    azuh Indexer

    Wazuh Manager

    Wazuh Dashboard

    Validation de la communication sécurisée entre tous les composants.

    Confirmation de l’accès fonctionnel au Dashboard.

**✅ Résultat**

    À l’issue de ces étapes, le SIEM Wazuh est entièrement installé, sécurisé et opérationnel.
    L’infrastructure est désormais prête pour :

    le déploiement des agents Wazuh (Windows Server, DMZ),

    les tests de remontée de logs,

    et l’intégration future d’un IDS tel que Suricata.

en vous servant de ces etape et de la documentation officiel, l'installation serat j'en suis sur un jeu d'enfant pour vous !)

**NB**
    Lors de l’installation de Wazuh Dashboard en mode étape par étape, les certificats TLS ne sont pas copiés automatiquement. Il a donc été nécessaire d’extraire manuellement les certificats générés avec `wazuh-certs-tool.sh` et de les placer dans `/etc/wazuh-dashboard/certs`. Sans cette étape, le service wazuh-dashboard ne peut pas démarrer car les communications HTTPS sont obligatoires.

**📌 Note de configuration – Règle pare-feu pfSense (Accès Wazuh Dashboard)**
🔥 Objectif

Autoriser l’accès sécurisé au Wazuh Dashboard hébergé sur Ubuntu Server depuis le poste physique de l’administrateur, tout en conservant une segmentation réseau stricte.

**🧱 Contexte de l’architecture**

    Pare-feu principal : pfSense

    SIEM : Wazuh

    Serveur Wazuh Dashboard : Ubuntu Server

    Adresse IP du serveur Wazuh : 192.168.220.5

    Port du Dashboard : 5601/tcp

    Accès autorisé uniquement depuis le poste administrateur

**⚙️ Détails de la règle pfSense**
    Paramètre	Valeur
    Action	Pass
    Interface	LAN
    Protocole	TCP
    Source	IP du PC physique (administrateur)
    Destination	192.168.220.5
    Port de destination	5601
    Description	Accès sécurisé au Wazuh Dashboard
    🔐 Justification sécurité

    Le port 5601 n’est pas exposé au WAN

    L’accès est restreint à une IP spécifique

    Le Dashboard Wazuh contient des données sensibles (logs, alertes, topologie)

    Cette règle respecte le principe du moindre privilège

**🌐 Accès utilisateur**

    Depuis un navigateur web (HTTPS) :

    https://192.168.220.5:5601


    ⚠️ Certificat auto-signé accepté lors de la première connexion.

**🧪 Validation**

    Service wazuh-dashboard actif

    Port 5601 en écoute (ss -tulnp)

    Accès fonctionnel depuis le poste physique

    Aucun accès non autorisé depuis les autres segments réseau

**📝 Remarque**

Pour un environnement de production, l’accès au Dashboard doit être réalisé via :

un VPN (OpenVPN / WireGuard sur pfSense)

ou une machine bastion dédiée

avant de continuer assurer vous que vous vous trouvez a la meme étape

**✅ État actuel de ton projet (checkpoint)**

    Tu as déjà réalisé :

    ✔ Architecture réseau segmentée (LAN / DMZ / Service stagiaire / WAN)
    ✔ pfSense en pare-feu central
    ✔ Active Directory fonctionnel
    ✔ IDS (Suricata) en place
    ✔ Wazuh Server + Dashboard installés et fonctionnels
    ✔ Accès au dashboard depuis le PC physique via règle pfSense

Accès au Wazuh Dashboard
Le dashboard utilise un certificat SSL auto-signé. Lors de la première connexion, le navigateur affiche un avertissement de sécurité. Celui-ci est normal dans un environnement de laboratoire.
L’accès se fait via : https://IP_DASHBOARD:5601

maintenant je vous recomande de passez a l"installation des agent c'est par l'intermediere des agent que notre siem pourat surveiller les autre pc 
aller sur le fichier de windows-server puis de ubuntu afin de commancer l'installation

## installation des agent 

1. Contexte et objectif

        Dans le cadre de ce projet, nous avons mis en place un serveur Wazuh (version 4.14.x) et déployé des agents Wazuh version 4.13.x sur différents systèmes d’exploitation, conformément à la documentation officielle Wazuh.

        Afin de faciliter :

        la gestion centralisée des agents,

        la surveillance des fichiers critiques,

        l’intégration de services avancés tels que VirusTotal,

        nous avons créé des groupes d’agents distincts directement depuis le Wazuh Dashboard :

        un groupe Linux

        un groupe Windows

        Cette approche est recommandée par Wazuh pour appliquer des configurations spécifiques selon le type de système d’exploitation.

2. Création des groupes dans le Wazuh Dashboard **Étapes (selon la documentation officielle)**

        Accéder au Wazuh Dashboard

        Aller dans :

        Management → Groups


        Créer deux groupes :

        linux

        windows

        Ces groupes permettent d’associer automatiquement :

        des règles spécifiques,

        des politiques de sécurité adaptées,

        des configurations FIM (File Integrity Monitoring) propres à chaque OS.

        📌 Cette méthode est conforme au guide officiel sur la gestion des agents et des groupes.
        (Source : documentation Wazuh – Agent management)

3. Installation de l’agent Wazuh avec affectation au groupe **Principe général (documentation officielle)**

        Lors de l’installation d’un agent Wazuh, il est possible de :

        définir le Wazuh Manager

        assigner directement l’agent à un groupe spécifique

        Cela évite une reconfiguration manuelle après l’installation.

4. Installation de l’agent sur Linux (Ubuntu Desktop / Server)
        Version utilisée

        Agent : Wazuh 4.13.x

        Serveur : Wazuh ≥ 4.13 (4.14.x dans notre cas)
        ➡️ Compatibilité assurée selon la documentation officielle.

        Étapes d’installation (Linux)

        - Ajout du dépôt officiel Wazuh

        - Installation de l’agent avec la variable WAZUH_MANAGER

        - Démarrage et activation du service

        - Enregistrement automatique de l’agent

        Lors de l’enrôlement, l’agent est directement rattaché au groupe linux depuis le Dashboard.

        📌 Cela permet :

        - d’appliquer immédiatement les règles Linux,

        - de surveiller les fichiers critiques propres à Linux,

        - de préparer l’intégration de VirusTotal sur les chemins Linux sensibles.

5. Installation de l’agent sur Windows Server
        Version utilisée

        Agent : Wazuh 4.13.x (MSI)

        Serveur : Wazuh ≥ 4.13

        Étapes d’installation (Windows)

        - Téléchargement du package MSI 4.13.x

        - Installation via interface graphique ou ligne de commande

        - Configuration de l’adresse du Wazuh Manager

        - Démarrage du service WazuhSvc

        L’agent Windows est ensuite assigné au groupe windows depuis le Wazuh Dashboard.

        📌 Cette séparation permet :

        - l’application de règles spécifiques Windows,

        - la surveillance des fichiers systèmes critiques Windows,

        - une intégration ciblée avec VirusTotal pour les exécutables Windows.

6. Intérêt des groupes pour VirusTotal

        Selon la documentation officielle Wazuh, l’intégration avec VirusTotal repose principalement sur :

        la surveillance des fichiers (FIM),

        l’analyse des hashs de fichiers modifiés ou suspects.

        Avantages de la séparation Linux / Windows

        Grâce aux groupes :

        Chaque OS dispose de chemins de fichiers spécifiques à surveiller

        Les règles FIM peuvent être adaptées par groupe

        VirusTotal est utilisé de manière plus précise et efficace

    **Exemples**
        Groupe Linux

        - /etc/

        - /bin/

        - /usr/bin/

        - /var/www/

    **Groupe Windows**

        - C:\Windows\System32

        - C:\Program Files

        - C:\Users\

        - Fichiers .exe, .dll, .ps1

📌 Cette organisation est fortement recommandée par Wazuh pour éviter les faux positifs et améliorer la détection des menaces.

7. Intérêt des groupes pour la surveillance des fichiers critiques (FIM)

        La surveillance de l’intégrité des fichiers (FIM) est un composant clé de Wazuh.

        L’utilisation de groupes permet :

        - d’activer FIM uniquement sur les fichiers pertinents selon l’OS,

        - de réduire la charge inutile,

        - d’améliorer la lisibilité des alertes.

        - Chaque groupe dispose de :

        - sa propre configuration ossec.conf

        - ses propres chemins critiques

        - ses propres règles d’alerte

8. Conclusion
        La création de groupes Linux et Windows avant l’installation des agents :

        - respecte strictement la documentation officielle Wazuh,

        - facilite l’administration du SIEM,

        - prépare efficacement l’intégration de VirusTotal,

        - améliore la surveillance des fichiers critiques,

        - rend l’architecture plus scalable, claire et professionnelle.

        - Cette approche est essentielle dans un contexte réel de cybersécurité et SOC.

maintenant que tout est dit je vous recomande de vous diriger ver le fichier de windows server et de ubuntu desktop afin de voir en detaille les etapes l'installation

**Enregistrement (Enrollment) de l’agent**

Une fois l’agent installé, il faut l’enregistrer auprès du manager :

Depuis le Wazuh Dashboard :

Agents > Deploy new agent > Windows / Linux

Copier la ligne de commande générée

Coller dans l’agent Windows / Ubuntu

Alternativement, vous pouvez utiliser :

Registration password

Clés API
→ Ceci est décrit en détail dans le guide d’enrollment officiel. 

**Conseils pratiques**

Versions

Il est recommandé d’installer le même numéro de version mineure sur tous les agents si possible (ex : 4.13.1 pour tous), pour éviter des incompatibilités.

Mais un serveur en 4.14+ peut supporter des agents 4.13. 
documentation.wazuh.com

**Sécurité**

Assurez-vous que les ports nécessaires sont ouverts entre agents et serveur (par défaut 1514/UDP, 1515/TCP, 55000/TCP pour enrollment API si utilisé). 
documentation.wazuh.com


**Documentation officielle utile**

Agent Windows installation — Wazuh docs
https://documentation.wazuh.com/current/installation-guide/wazuh-agent/wazuh-agent-package-windows.html
 
Agent Linux installation — Wazuh docs
https://documentation.wazuh.com/current/installation-guide/wazuh-agent/wazuh-agent-package-linux.html

Agent enrollment guide
https://documentation.wazuh.com/current/user-manual/agent/agent-enrollment/index.html

## integration de syscheck (_FIM_) 

L’objectif de cette étape est de mettre en place le File Integrity Monitoring (FIM) à l’aide du module Syscheck de Wazuh, afin de surveiller en temps réel les modifications apportées aux fichiers et répertoires critiques du système.
Cette fonctionnalité permet de détecter toute activité suspecte telle que :

    - la modification non autorisée de fichiers système,
    - l’ajout de fichiers malveillants,
    - ou l’altération de configurations sensibles.

1. Principe de fonctionnement de Syscheck

Syscheck fonctionne en comparant l’état actuel des fichiers surveillés à un état de référence.
Pour chaque fichier analysé, Wazuh calcule des empreintes cryptographiques (hashs) et surveille :
    - les changements de contenu,
    - les permissions,
    - les propriétaires,
    - la date de modification.
Toute différence détectée est transmise au Wazuh Manager, analysée, puis indexée dans le Wazuh Indexer afin d’être visualisée sur le Dashboard
une fois cette intégration faite, notre SIEM serat capable d'éffectuer les taches suivantes:
    - Collecter les logs Syslog via un pare-feu

    - Surveiller l’intégrité des fichiers (Syscheck / FIM)

    - Centraliser les événements de sécurité

    - Enrichir les alertes via VirusTotal

    - Visualiser les données dans le Wazuh Dashboard

2. architecture générale

```less
[Equipements / Pare-feu / Serveurs ]
                |
                | (SYSLOG UPD 514)
                |
            [ Wazuh Manager ]
                |
    (Syscheck + Rules + VirusTotal)
                |
            [Wazuh Indexer ]
                |
            [Wazuh Dashboard]

```
cette architecture presente le plan global du fonctionnement de syscheck pour ce projet

3. installation et configuration de RSYSLOG

sur votre pc  ubuntu-serveur celle sur le quelle est a present installer votre siem wazuh taper les commande suivante

**installation de rsyslog**

```bash
sudo apt update
sudo apt install rsyslog -y
systemctl status rsyslog
```
maintenant que tout est installer place a la configuration

**configuration de rsyslog**

```bash
sudo nano /etc/rsyslog.conf
```
cette commande ouvre le fichier de configuration de syslog la prochaine etape est de décommenté d'ajouter les lignes suivant

```conf
module(load="imudp")
input(type="imudp" port="514")

module(load="imtcp")
input(type="imtcp" port="514")
```

une fois bon redémarer et verifier que sa ecoute bien sur le pour 514

```bash
sudo systemctl restart rsyslog
sudo ss -lunp | grep 514
```
4. configuration de UFW **(pare-feux interne a tout os linux)**

autorisé la communication sur ufw 

```bash
sudo ufw allow 514/udp
sudo ufw reload
```
si a la place vous utilisé comme pare-feux interne **iptables** cette commande est pour vous

```bash
sudo iptables -A INPUT -p udp --dport 514 -j ACCEPT
```
ces étapes on servis a installer et démarer syslog sur votre pc ubuntu server, maintenant il fut l'intégrer a votre manager 

5. configuration syslog coté wazuh-manager 

il se fera sur le fichier **ossec.conf** 

```bash
sudo nano /var/ossec/etc/ossec.conf
```
etant dans le fichier de chercher la section remote et rajoutez en une deuxiemme celle deja present represente la communication avec l'agent et la deuxiemme pour les mol

```xml
<remote>
  <connection>secure</connection>
  <port>1514</port>
  <protocol>tcp</protocol>
</remote>

<remote>
  <connection>syslog</connection>
  <port>514</port>
  <protocol>udp</protocol>
  <allowed-ips>192.168.220.0/24</allowed-ips><!-- ip du reseaux d'envoie -->
  <local_ip>192.168.220.5</local_ip> <!-- ip du pc qui recoit les log -->
</remote>
```

et maintenant on redemare le manager 

```bash
sudo systemctl restart wazuh-manager
```
c'est bon syslog est integrer maintenant il faut retourner dans le fichier ossec.conf pour changer les parametre par default de syscheck
NB _afin d'eviter tout erreur, suprimer toute votre config syscheck su fichier ossec et coller celle-ci a la place_

 ```xml
<syscheck>
  <disabled>no</disabled>
  <scan_on_start>yes</scan_on_start>
  <alert_new_files>yes</alert_new_files> <directories realtime="yes" check_all="yes">/etc,/usr/bin,/usr/sbin</directories>
  <directories realtime="yes" check_all="yes">/bin,/sbin,/boot</directories>
  <directories realtime="yes" check_all="yes">/root,/home,/tmp</directories>
</syscheck>
 ```

 par default wazuh analyse tout les 12 de temps ce qui nuit a nos analyse donc cette configuration vous garanti une annalyse en temp réel
 une fois bon redémarer 

```bash
sudo systemctl restart wazuh-manager
```

maintenant que les groupe on été précedemant creer nous allons renseigner les repertoire vulnerable sur ces groupe avnt e le refaire sur les différent fichier des agent 
les os linux et windows n'utilise pas les meme repertoire c'est pourquoi il est crusial de passer des groupes

sur le dashboard aller sur agent management puis sur groupe puis le le stylo (modifier) de chaque groupe et ajouter respectivement ceci

**fichier pour le groupe linux**

```xml
 <agent_config group="linux">
    <syscheck>
      <directories check_all="yes">/etc</directories>
      <directories check_all="yes">/usr/bin</directories>
      <directories check_all="yes">/usr/sbin</directories>
      <directories check_all="yes">/bin</directories>
      <directories check_all="yes">/var/www</directories>
      <directories check_all="yes">/home</directories>
      <directories check_all="yes">/root</directories>
    </syscheck>
  </agent_config>
```

**fichier pour le groupe windows**

```xml
  <agent_config group="windows">
    <syscheck>
      <directories check_all="yes">C:\Windows\System32</directories>
      <directories check_all="yes">C:\Program Files</directories>
      <directories check_all="yes">C:\Program Files (x86)</directories>
      <directories check_all="yes">C:\Users</directories>
      <directories check_all="yes">D:\Shared</directories>
    </syscheck>
  </agent_config>
```
et cliquer sur save pour sauvegarder

## instegration de virus total 

virus total ici vas permetre de detecter tout logiciel malveillant en le couplans a syscheck nous indiquons donc a virus total les repertoire critique (repertoire d'analyse) en temp réel

    - aller sur le site officiel de virus total 👉 https://www.virustotal.com/gui/join-us et creer un compte gratuit

    - recuperer votre clé API 👉 https://www.virustotal.com/gui/user/
    <username>/apikey

rendez vous a nouveaux sur le fichier ossec.conf et ajouter une integration pour virus total

```bash
sudo nano /var/ossec/etc/ossec.conf # éditer le fichier ossec.conf
```

```xml
<integration>
  <name>virustotal</name>
  <api_key>VOTRE_CLE_API</api_key>
  <rule_id>554,550</rule_id>
  <alert_format>json</alert_format>
</integration>
```

toujour penseé a redemarer le manager apres une integration 

```bash
sudo systemctl restart wazuh-manager
```
pour verifier les logs de virus total sur le terminal du server on procede de la maniere suivente 

```bash
sudo tail -f /var/ossec/logs/integrations.log
```

resultat attendu:

```lua
VirusTotal: Positive match
```

**remonter des log au dashboard**

filebeat comme nous l'avons vue lors de l'installation étape par étape de wazuh permet de relier tout les trois composants de wazuh (_indexer, manager, dashboard_) si filebeat est en panne ou est simleùent etaint les log ne remonterons pas 

    - vérification du status de filebeat

```bash
sudo systemctl status filebeat
```
    - redemarer si il est eteint et l'activer au redemarage 
```bash
sudo systemctl enable filebeat #activer au demarage de l'os
sudo systemctl restart filebeat # redemarer les services filebeat
```
    - verification de l'inderxer

```bash
curl -k -u admin:admin https://192.168.220.5:9200/_cat/indices
```
si tout est OK alors vos log remontera bien au niveaux du dashboard 
le test final pour verifier les logs 

**test syslog**

```bash
logger -n 192.168.220.5 -P 514 -d "TEST SYSLOG SOC"
```

**test syscheck**

```bash
touch /etc/test_fim.txt
```

**test virustotal**

```bash
curl -o /tmp/eicar.txt 
https://secure.eicar.org/eicar.com.txt 
```
ces commande vont vous permetre de tester chacune des configuration precedement fait et si tout est bon les log s'aficherons tous sur le dashbord de wazuh.

## surclassement de allerte 

**c'est quoi le le surclassement**

un grand classique avec wazuh ou OSSEC est que les logs remonte bien mais, il coresponde a une regle par défaut (`généralement le 5710 ou 5011 pour les ssh echoueés`) ce qui entraine un niveaux de severité faible (`niveaux 5`) 
le problemme dans ce cas precis c'est que si une attaque de type `brute force ssh` est en cour, le niveau d'alerte serais faible et donc se mélangera au evenement basic su systeme 
pour y remedier, nous allons ecrire une regle qui ecrase ou juste complete la regle standard et definir un niveaux d'alerte critique sur cette regle (`exemple: 12, 13, ...`)

l'objectif ici est de faire une règle `CHILD (`enfant`)` afin de surclasser les attaque de type brute force ssh sur notre systeme

**création de la règle**

ls règles se configure dans un fichi spécfique appelé `local_rules.xml` les étapes pour la création sont les suivante

1. accéder au fichier de creation 

```bash
    sudo su
    cd /var/osec/etc/rules/
    ls
    nano local_rles.xml
```
2. suprimer tout ls règle present par defaut car les nouveaux vont écrasé les anciens donc il n'est pas nécessaire de les gardé

3. ajouter les règle suivant en respectant la syntaxe

```XML
<!-- Example -->
<group name="local,syslog,sshd,">

  <!--
  Dec 10 01:02:02 host sshd[1234]: Failed none for root from 1.1.1.1 port 1066 ssh2
  -->
  <rule id="100001" level="12">
    <match>Failed password|authentification failed</match>
    <if_sid>5710,5711,5716,5760</if_sid>
    <description>PFA: Attaque brute force SSH détecteé - Seuille critique atteint</description>
  </rule>

  <rule id="100002" level="10">
    <if_sid>5710,5760</if_sid>
    <match>root</match>
    <description>PFA: tentative de connexion direct en ROOT</description>
  </rule>

  <rule id="100003" level="8">
    <if_sid>31108</if_sid>
    <match>nmap|sqlmap|nikto</match>
    <description>PFA: outil de reconnaissance (nmap|sqlmap|nikto) détecté sur le serveur </description>
  </rule>

  <rule id="100004" level="13">
    <if_sid>530</if_sid>
    <match>/etc/shadow|/etc/passwd</match>
    <description>PFA: ALERTE - Modification ou accès aux fichier de mot de passe système</description>
  </rule>

  <rule id="100005" level="15">
    <match>session opened</match>
    <if_sid>5501,5502</if_sid>
    <description>PFA: ATTENTION: connexion SSH reussit</description>
  </rule>

</group>

```
4. redémarer le manager

```bash
sudo systemctl restart wazuh-manager
```

## configuration alerte mail

Configurer le serveur Wazuh pour envoyer automatiquement des 
alertes par email lorsqu’un événement critique est détecté (attaque, malware, scan, etc.).

1. Prérequis

Avant de commencer, assure-toi que :

    - Wazuh Manager est installé et fonctionnel
    - Tu as accès au fichier de configuration /var/ossec/etc/ossec.conf
    - Tu as une adresse email (Gmail recommandé pour test)
    - Tu as généré un mot de passe d’application (pas ton vrai mot de passe)

2. Générer un mot de passe d’application (Gmail)
Active la validation en 2 étapes sur ton compte Gmail
    - Va dans : Sécurité → Mots de passe des applications
    - Génère un mot de passe (ex: "wazuh-mail")
    - Garde-le précieusement 

3. configuration a faire sur wazuh

ouvre le fichier ossec.conf

```bash
sudo nano /var/ossec/etc/ossec.conf
```

ajoute la configuration suivant dans la sectionn global 

```xml
<global>
  <email_notification>yes</email_notification>
  <email_to>tonmail@gmail.com</email_to>
  <smtp_server>smtp_server</smtp_server>
  <email_from>wazuh@tonlab.local</email_from>
  <email_maxperhour>12</email_maxperhour>
</global>

<email_alerts>
  <email_to>tonmail@gmail.com</email_to>
  <level>10</level>
</email_alerts>
``` 
**mail_notification** : active les alertes mail
**email_to** : adresse de réception
**smtp_server** : serveur SMTP (Gmail ici)
**level** : niveau d’alerte (10 = critique uniquement)
**email_maxperhour** : limite anti-spam

4. Configuration du service mail (Postfix)

Installe Postfix :

```bash
sudo apt install postfix mailutils -y
```
Choisis 
Internet Site
ici, postfix vas nous permetre de gerer les authentification via le serveur smtp de google 

5. Configurer Gmail SMTP

Édite postfix :

```bash
sudo nano /etc/postfix/main.cf
```
Ajoute à la fin :

```bash
relayhost = [smtp.gmail.com]:587
smtp_sasl_auth_enable = yes
smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd
smtp_sasl_security_options = noanonymous
smtp_tls_security_level = encrypt
smtp_tls_CAfile = /etc/ssl/certs/ca-certificates.crt
```

6. Ajouter les identifiants

```bash
sudo nano /etc/postfix/sasl_passwd
```
Ajoute :

```bash
[smtp.gmail.com]:587 tonmail@gmail.com:mot_de_passe_application
```
Puis sécurise :

```bash
sudo postmap /etc/postfix/sasl_passwd
sudo chmod 600 /etc/postfix/sasl_passwd
```

7. Redémarrer les services

```bash
sudo systemctl restart postfix
sudo systemctl restart wazuh-manager
```

8. Test d’envoi mail

Teste avec :

```bash
echo "Test Wazuh OK" | mail -s "Test Mail" tonmail@gmail.com
```

9. Vérification des logs

```bash
tail -f /var/log/mail.log
```

Et du côté de Wazuh :

```bash
tail -f /var/ossec/logs/ossec.log
```
nous pouvons aussi utiliser eicar pour tester l'envoie des mail 

```bash
curl -O https://secure.eicar.org/eicar.com
```

Si tout est OK, normalement Wazuh devrait détecter, les Alerte devrait etre générée et les Mail devrons etre reçu 