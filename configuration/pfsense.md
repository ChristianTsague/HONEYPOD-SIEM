Le cas du pare-feu est quelque peut complexe car il dispose de trois carte reseaux (interface) et ca configuration ne se fait pas sur le une terterface graphique ou via un fichier netplan il se fait a*de la maniere suivante 

### ajouter les rois interface sur l'yperviseur

etant sur vmware, faire un clic droit sur pfsense et ajouter tout les vmnet precedement creer il est important de metre les meme carte atribuer au autre machine

## une fois avir mis les carte de maniere (physique) passer a l'assignation

il se fait apres avoir clique sur l'option 1 quand le pc est allumer et renseigner les carte dans mon cas choisir EM0:wan, EM1:LAN, OPT1:EM3, OPT2:EM4
puis valider 
 `NB: la WAN en DHCP`

# l'etape suivant consiste maintenant a configurer les ip 

choisi l'option 2 et selectionner la carte a attribuer les ip toujour a preciser que le wan doit etre en DHCP et les autre atribuer manuellement 
`les ip atribuer sur pfsense represente les passerelle par default des autre pc`

   ```bash 
        - 192.168.220.253 #(LAN)
        - 192.168.43.253 #(OPT 1)
        - 192.168.200.253 #(OPT 2)
        - DHCP #(WAN)
   ```
# configuration des regle de filtrage 

les regle de filtrage permete en gros de choisir qui peut avoir acces a qui sur notre reseaux ils définissent d'une certaine maniere le shémat que doit suivre le trafic sur le reseaux  pour ce faire il faut avoir acces a l'interface graphique de pfsense et si vous avez configurer comme indiquer 

    1. aller sur le navigateur de votre pc physique et entrez dans la bare d'adresse l'une des ip atribuer a pfsense (`192.168.220.253`)
    2. une page de login s'ouvre a vous entrez le nom d'utilisateur et le mot de passe de pfsense. par default le nom d'utilisateur est `admin` et le mot de passe est `pfsense`
    3. avant de configurer les regle on vous demandra peut etre de changer ce mot de passe si c'est le cas faite le

il a été marquer au tout début du cour que afin de bien suivre cette formation il est important d'avoir quelque base en administration systeme et bien que j'essayerais d'etre tres explicite lors sur cette docu je vous invite a faire quelque recherche si le domaine des pare-feu vous ai encore un peu flou si non `les regles apliquer sont present dans les fichier resources `

## installation de l'IDS Suricata

Documentation IDS – 

1. Qu’est-ce qu’un IDS ?

Un IDS (`Intrusion Detection System`): est un système de détection d’intrusions qui surveille le trafic réseau ou les activités sur un 
système pour identifier des comportements suspects, des tentatives d’attaques ou des violations de sécurité. 
Contrairement à un pare-feu, un IDS ne bloque pas forcément le trafic, mais alerte l’administrateur en cas d’anomalies.

2. Pourquoi Suricata ?

Suricata est un IDS open-source performant qui offre :
    a. Détection réseau en temps réel grâce à des règles mises à jour régulièrement.
    b. Compatibilité multi-plateforme (Linux, Windows, BSD).
    c. Haute performance grâce à la parallélisation et la prise en charge du multi-threading.
    d.Flexibilité : possibilité d’analyser le trafic réseau, détecter des malwares, scanner des paquets et générer des logs détaillés pour un SIEM comme Wazuh.
Nous l’avons choisi pour sa facilité d’intégration dans notre architecture VMware avec pfSense et sa capacité à surveiller différents 
segments du réseau (`LAN, WAN, DMZ`).

3. Rôle dans notre architecture

Dans notre architecture réseau :
Suricata est installé sur pfSense pour surveiller tout le trafic qui transite entre les VLAN et la DMZ.
Il permet de détecter des tentatives d’intrusion provenant de la DMZ (`machines publiques ou honeypots`) et des postes internes.
Les alertes générées sont envoyées au SIEM (Wazuh) pour corrélation, analyse et reporting.
Il contribue à la sécurité proactive en permettant d’anticiper les attaques avant qu’elles n’affectent nos serveurs critiques.

4. Mise en place

Voici les grandes étapes de configuration de Suricata sur pfSense :

Installation : via le package manager pfSense (System → Package Manager → Available Packages → Suricata).

Configuration des interfaces : activer Suricata sur les interfaces à surveiller (`LAN, WAN, DMZ`).

Définition des règles : utiliser les règles par défaut ou importer des règles spécifiques (Emerging Threats, Snort compatible).

Activation des logs : définir la journalisation pour qu’elle soit consultable depuis pfSense ou intégrée à Wazuh.

Test : vérifier que Suricata détecte correctement les flux réseau et génère des alertes pertinentes.

5. Conclusion

Suricata permet une détection proactive des intrusions et est un outil clé pour sécuriser notre infrastructure réseau segmentée. Il complète le pare-feu pfSense et le SIEM Wazuh pour offrir une vision complète de la sécurité réseau.


Après la mise en place de l’architecture réseau, de l’Active Directory et de l’IDS Suricata, la prochaine étape consiste à déployer
un SIEM (`security information and event management`) afin de centraliser, analyser et corréler les événements de sécurité provenant des différents composants de l’infrastructure. 
donc je vous recomande de vous rendre sur le fichier de configuration `wazuh.md` pour continuer
