1. configurer une ip via netplan  

#### configuration du fichier ip par default 
comme sur toute les machine une fois l'os installer configurer les ip sur les pc etant donner que nous somme sur ubuntu server, la configuration de l'ip se fait via netplan
la verification de l'ip grace a la commande ip config n'a pas marcher donc j'ai mis mas carte reseaux en briged (acces par pont) et installer le paquet net-tols grace a la commande `sudo apt install net-tools`"
maintenant, configurer les ip via netplan grace au commande suivante 
accedez au fichier `etc/netplan` ouvrez lz fichier `50-cloud_unit` et ajouter la configuration suivante la commande et `nano 50-cloud_unit `

```yaml
network:
  version: 2
  ethernets: 
    ens33:
      dhcp4: false
      addresses: 
	      -192.168.220.5/24
	gateway4: 192.168.20.253
      nameservers: 
        addresses: 
          - 8.8.8.8
          - 1.1.1.1 
```
puis effectuer la commande `sudo netplan generate` puis la comande `sudo netplan apply` afin de valider la configuration 

### creation du nouveau fichier 

une fois cette configuration faite j'ai remarquer que quand on redemarais le pc toute la configurationetais automatiquement effacé donc commesodution j'ai
    A. creer un nouveau fichier de non de `01-net_cfg` (attention le nom est facultative)
    B. editer le fichier grace a la commande `nano 01-net_cgf`
    C. ajouté le configuration de le l'ip suivant

        ```yaml
        network:
          version: 2
          ethernets: 
            ens33:
              dhcp4: false
              addresses: 
                -192.168.220.5/24
              gateway4: 192.168.20.253
              nameservers: 
                addresses: 
                  - 8.8.8.8
                  - 1.1.1.1 
        ```
valider avec la commande `sudo netplan generate` puis `sudo netplan apply` et en redamarant le systeme vous remarquerez que deux ip apraraira 
    - une pour le fichier dhcp4 
    - et une pour la configuration static precedement effectuer
prochaine etape desactiver le service cloud init pour avoir juste une addresse ip fonctionelle (`celle du DHCP`)

      ```bash
      1. desactiver cloud init proprement (`sudo touch /etc/cloud/cloud-init.disable`)
      2. desactiver tout les service cloud init
          (`sudo systemctl disable cloud-init
            sudo systemctl disable cloud-init-local
            sudo systemctl disable cloud-config
            sudo systemctl disable cloud-final
          `)
      3. redemarer (`sudo reboot`)
      ```
et voila vous avez regler le probleme des ip double ;) 


## Prochaine etape relier ce pc a notre active directory

si vous n'avez pas encore creer l'AD DS(`active directory domain server`) je vous conseille d'aller lire le fichier de config de windows server avant de continuer 
car  sur ce fichier je montre j'explique en detaille comment creer notre servers active directory

# joindre au domaine active diectory

 nous nous trouvons actuellement sur ubuntu_server et le but a ce stade est de le relier a notre domaine AD DS `Active Directory Domain Server` 
 la procedure est extremement simple mais avant vous devez faire une petite modification sur votre ficher `/etc/netplan/01_netcfg` en ajoutant l'adresse et le nom de domaine de notre serveur

      ```yaml
          network:
        version: 2
        ethernets: 
          ens33:
            dhcp4: false
            addresses: 
              -192.168.220.5/24
        gateway4: 192.168.20.253
            nameservers: 
              search: 
                - S_serveur.local
              addresses: 
                - 192.168.220.3
                - 8.8.8.8 
      ```
(`ici, la 8.8.8.8 est l'adresse de google elle joue le role de serveur secondaire`) elle vas nous permetre d'avoir un acces a internet vue que le trafic passe par le parefeux logiquement on aurais pas besoin de l'inscrit mais en faisant sans j'ai remarquer que sa ne marchais pas le pc ne pouvais pas surfer donc inscriver le 
valider avec un `ctrl o` pour sauvegarder puis un `ctrl x` pour quiter 
puis faire un `sudo netplan generate` et un `sudo netplan apply`

la premiere etape est fait maintenant on le renseigne sur samba (c'est une aplication linux qui a plusieurs fonction mais cele qui nous interesse est de pouvoir relier a notre AD DS) ou sssd nous on vas utiliser sssd elle est plus simple 

 fait un (`sudo nano /etc/resolv.conf`) et ajoute l'ip de ton serveur a la place de celui present puis relie avec realm

    ```bash
      sudo apt install realmd sssd adcli -y
      sudo real discover S_serveur.local
      sudo join -U Administrator S_serveur.local
    ```
une fois fait verifier avec un `realm list` et efectuer un ping avec votre DNS pour voir le resultat

     ```bash

     ping S_serveur.local

     ```
vous pouvez maintenant vous conecter a tout les comptes présent sur windows server étant sur votre machine ubuntu server c'est ce principe qui est utiliser en entreprise

    ```bash


    ```
et sa sera le cas pour tout les autre pc avec L'ADDS configurer dessus

