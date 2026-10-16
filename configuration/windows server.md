cette configiration ce fait sur window server un systeme ideal pour la configuration des serveur 
il ferat office de DMZ (`un genre de serveur web`) mais pas que nous allons configurer un ADDS dessus (`active dyrectory domain server`)

#### maintenant que tout est dit passont a la configuration des ip 

sur windows les ip se configure de la meme maniere nous allons toujour le faire en static et pour ceux qui ignore comment faire voila les etape
        1. acceder a votre panneaux de configuration, reseaux internet puis centre reseaux et partage
        2. cliquer sur modifier les parametre de la carte puis choisir la carte qui nous interesse (`ethernet dans notre cas`)
        3. cliquer sur propriéter puis choisisser le protocole ipv4 et rentrez votre ip (`192.168.220.2`) le masque par default (`255.255.255.0`) selectionner valider en quitant et refermer

### creation du domaine active directory 

se rendre sur windows server et cliquer sur gestion des server 
        1. choisir l'option 2 (`ajouter des role et fonctionaliter`) cliquer sur suivant
        2. cliquer sur la premiere option (`installation baser sur un role ou une fonctionnaliter`)
        3. sur filtre laisser la section vide  
        dans le tableau plus bas choisser le serveur par default celui qui porte l'ip que nous avons precedament fixer 
        4. en apuiant sur suivant prenez service AD DS comme role de serveurs et installer le
        5. une fois l'installation terminer retour sur le gestionnaire de serveur et cliquer le petit drapeau jaune des notification puis sur `promouvoir le serveur`
        6. sur l'assistant de configuration clique sur `ajouter une nouvelle foret` puis entre un nom de domaine racine dans mon cas je rentre `S_serveur.local
        7. laisse les option DNS par default sur les option de controleur de domaine et entre juste un mot de passe admin pour continuer 
        *NB* le mot de passe doit contenir des letre, chifre, et caractere speciaux
        8. prochaine etape le `NetBios name` laisser par defaul le nom proposer ou modifier a votre preference
        9. laisser le chemain de fichier par defaul et cliquer sur suivant
        10. apres verification si tout a ete parfaitement executer, clique sur installer 

le domaine est créer maintenant retour sur les autre systeme pour relier a ce domaine. aller dans le fichier de ubuntu server pour voir comment les relier a notre nom de domaine

## installation de lagent wazuh

**aller sur le fichier wazuh.md ou vous verez comment installer notre siem wazuh avant de debuter l'installation de l'agent**

1. Téléchargement du package MSI

Depuis le Wazuh Dashboard (Agents > Deploy new agent), sélectionnez :

Windows

Version 4.13.x

Architecture (32-bit ou 64-bit)

Vous obtiendrez :

Un fichier .msi (ex : wazuh-agent-4.13.1-1.msi)

La ligne de commande préconfigurée (avec adresse du manager et paramètres)

**C’est la méthode recommandée pour éviter les erreurs de configuration.**

2. Installation par GUI

Double-cliquez sur le fichier .msi téléchargé.

Suivez l’assistant d’installation.

Entrez :

IP/Hostname du Wazuh Manager (serveur)

Nom de l’agent (facultatif)

Groupe / clé d’enregistrement si utilisé

Validez.

3. Installation par ligne de commande (CLI)

**si vous vous trouver sur le cmd**

```bash
msiexec /i wazuh-agent-4.13.1-1.msi /qn WAZUH_MANAGER="192.168.220.5"
NET START WazuhSvc
```
**si vous vous trouver sur le shell**

```bash
Start-Process msiexec.exe -ArgumentList "/i wazuh-agent-4.13.1-1.msi /qn WAZUH_MANAGER=`"192.168.220.5`"" -Wait
Start-Service WazuhSvc

```
4. Vérification

Vérifiez que le service WazuhSvc est en cours d’exécution sur le shell :

```bash
Get-Service WazuhSvc
```

Si agent ne démarre pas ou se bloque :

- Revérifiez l’adresse du manager

- Assurez-vous que le pare-feu Windows autorise le port 1514/UDP (agent → manager) si utilisé

**Note** : Certains utilisateurs ont signalé que l’installateur MSI 4.13.1 peut parfois bloquer à l’installation sur Windows Server 2019 (résolu via log ou debug)

**Agent Windows installation — Wazuh docs**

```lien
https://documentation.wazuh.com/current/installation-guide/wazuh-agent/wazuh-agent-package-windows.html
```