### configuratin de l'ip via netplan

On vien de configurer ubuntu server donc vous avez maintenant une connaissance sur la configuration des ip sur les systeme ubuntu maintenant on vas faire la meme chose sur ubuntu desktop 
si vous ne voulez pas le faire en ligne de code rendez vous dans parametre et reseaux et adresser via l'interface graphique dans mon cas je le fait en ligne de code
     A. accecer  au fichier se trouvant das netplan grace a la comande `cd /etc/netplan`
     rentrez la meme conficuration que celui de ubuntu server (si vous n'avez pas encore vue je vous invite a aller verifier dans le fichier de ubuntu server)
     B. rentre la meme config et changer d'ip ramplacer par l'ip de ubuntu desktop 

        ```yaml
            network:
            version: 2
            ethernets:
                ens33:
                    dhcp4: no
                        addresses:
                            - 192.168.200.3/24
                        gateway4: 192.168.200.253
                        nameservers:
                        addresses:
                            - 8.8.8.8
                            - 1.1.1.1
        ```
une fois fini si vous efectuez un ping vers votre paserelle vous verez que ca ne marcherat pas 

etant donner que ici, ce pc jourat le role de ma DMZ il ne serat pas relier a notre AD DS (consulté le fichier de windows server) car une DMZ serat exposé sur internet et donc serait vulnerable 

rendez vous maintenant sur le fichier de PF_sense pour voir les regle de filtrage apliquer cette DMZ afin de l'isoler et faire en sorte d'autoriser le ping


## installation et deployement de lagent wazuh

**aller sur le fichier wazuh.md ou vous verez comment installer notre siem wazuh avant de debuter l'installation de l'agent**

1. Ajouter le dépôt officiel Wazuh

```bash
sudo apt-get update
sudo apt-get install gnupg apt-transport-https -y
curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH \
  | gpg --no-default-keyring --keyring /usr/share/keyrings/wazuh.gpg --import
echo "deb [signed-by=/usr/share/keyrings/wazuh.gpg] \
  https://packages.wazuh.com/4.x/apt/ stable main" \
  | sudo tee /etc/apt/sources.list.d/wazuh.list
sudo apt-get update

```
2. Installation de l’agent

```bash
sudo WAZUH_MANAGER="192.168.220.5" apt-get install wazuh-agent -y
```

3. Activer et démarrer l’agent

```bash
sudo systemctl daemon-reload
sudo systemctl enable wazuh-agent
sudo systemctl start wazuh-agent
```
4. Vérifier le statut

```bash
sudo systemctl status wazuh-agent
```

**Agent Linux installation — Wazuh docs**

https://documentation.wazuh.com/current/installation-guide/wazuh-agent/wazuh-agent-package-linux.html

**Agent enrollment guide**

https://documentation.wazuh.com/current/user-manual/agent/agent-enrollment/index.html

## Difficultés rencontrées lors du déploiement de l’agent Wazuh en DMZ

Lors de l’intégration de l’agent Wazuh sur une machine Ubuntu placée en zone DMZ, plusieurs difficultés ont été rencontrées, empêchant le bon démarrage du service wazuh-agent.

Initialement, l’agent Wazuh était correctement installé et configuré :
    - les fichiers de configuration étaient présents,
    - les certificats TLS nécessaires à la communication sécurisée étaient générés,
    - la clé d’enregistrement de l’agent avait été importée avec succès,
    - le serveur Wazuh Manager était opérationnel dans le réseau interne (LAN).
Cepandant, malgré une configuration logicielle correcte, le service wazuh-agent ne parvenait pas à démarrer et retournait systématiquement une erreur indiquant que le processus s’était arrêté avec un code d’échec.

Après une analyse approfondie, il a été constaté que la cause du problème n’était pas liée à Wazuh lui-même, mais à la segmentation réseau mise en place via le pare-feu pfSense.
La machine Ubuntu surveillée étant placée en DMZ, les règles de filtrage appliquées bloquaient toute communication sortante vers le réseau interne, y compris vers le Wazuh Manager.

En particulier, les règles de sécurité empêchaient :
    - les requêtes ICMP (ping), utilisées pour vérifier la connectivité,
    - les connexions TCP sur le port 1514, utilisé par l’agent Wazuh pour envoyer les événements au manager,
    - les connexions TCP sur le port 1515, nécessaires à l’authentification et à l’enregistrement de l’agent.

Cette isolation stricte de la DMZ, bien que conforme aux bonnes pratiques de sécurité, empêchait l’agent Wazuh d’établir une connexion avec le serveur de supervision. En conséquence, l’agent détectait l’impossibilité de joindre le manager et s’arrêtait volontairement lors de son démarrage.

La résolution du problème a consisté à ajuster les règles du pare-feu pfSense afin d’autoriser de manière contrôlée la communication entre la DMZ et le Wazuh Manager, uniquement sur les ports nécessaires au fonctionnement du SIEM. Cette approche a permis de conserver le principe d’isolation de la DMZ tout en assurant la supervision de sécurité via Wazuh.

**sur interface de la DMZ(OPT1)**
1. pour l'icmp
    Action: PASS
    Interface: DMZ
    Protocol: ICMP
    Source: DMZ net
    Destination: WAZUH_SERVER_IP
    Description: DMZ → Wazuh ICMP
2. pour l'agent wazuh
    Action: PASS
    Interface: DMZ
    Protocol: TCP
    Source: DMZ net
    Destination: WAZUH_SERVER_IP
    Destination port: 1514
    Description: DMZ → Wazuh Agent

**Tests à faire immédiatement**

Sur l’Ubuntu DMZ :

```bash
ping 192.168.220.5
nc -vz 192.168.220.5 1514
```

**Résultat attendu**

```nginx
succeeded
```

**redemarage final**

```bash
sudo systemctl restart wazuh-agent
sudo systemctl status wazuh-agent
```
L’agent Wazuh en DMZ ne pouvait pas joindre le manager à cause des règles de filtrage pfSense.
Après ouverture contrôlée des ports 1514 et 1515 depuis la DMZ vers le LAN, l’agent s’est enregistré avec succès.
Cette segmentation renforce la sécurité tout en garantissant la supervision SIEM