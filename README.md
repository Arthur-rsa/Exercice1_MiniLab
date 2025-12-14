# Exercice 01 : MiniLab Packet Tracer
---

**Configuration réseau complète avec VLAN, DHCP et accès Internet**

Ce projet consiste à créer un *mini-laboratoire réseau* sous **Cisco Packet Tracer**, comprenant la configuration de VLANs, la mise en place du routage inter-VLAN, un serveur DHCP centralisé, ainsi que la vérification de la connectivité interne et l’accès Internet.

---
## Objectifs du projet

- Configurer un réseau multi-VLAN fonctionnel.
- Gérer le Routage inter-VLAN via un **Routeur Cisco 1941 (Router-on-a-Stick)**.
- Déployer un **serveur DHCP** pour chaque VLAN.
- Assurer l’accès Internet pour tous les équipements.
- Tester la connectivité entre les postes des différents VLAN.
- Organiser physiquement le réseau en **3 bureaux identiques**.
  
---

## Équipements utilisés

**1 Routeur Cisco 1941**
**3 Switch PT**
**3 Points d’accès Wi-Fi (PT-AC)**
**3 PC portables**
**6 PC fixes**
**Câbles ethernet/fibre selon les besoins**

 > Les téléphones IP sont à configurer uniquement sur leur VLAN - **pas d’IPBX à gérer**.

---

## Organisation physique des bureaux

Chaque bureau contient :

- 1 switch
- 1 point d’accès Wi-Fi
- 1 PC portable
- 2 PC fixes
- 1 téléphone IP

---

## Configuration des VLAN sur chaque switch

| Ports        | Usage                   | VLAN      |
| ------------ | ----------------------- | --------- |
| Port 8       | Administration          | **30**    |
| Ports 6-7    | PC fixes                | **20**    |
| Ports 4–5    | Wi-Fi                   | **10**    |
| Ports 2–3    | Téléphones IP           | **1**     |
| Ports 1 et 9 | Uplink (Ethernet/Fibre) | **TRUNK** |

---
    Dans un contexte réel, le VLAN 1 n'est jamais utilisé pour du trafic utilisateur. Ici, il est imposé par le sujet.

## Plan d’adressage & DHCP

Le **routeur 1941** assure :

  - la passerelle par VLAN
  - le routage inter-VLAN (sous-interfaces)
  - le DHCP pour chaque VLAN

### Tableau des réseaux

| VLAN    | Usage          | Réseau          | Plage DHCP                    |
| ------- | -------------- | --------------- | ----------------------------- |
| VLAN 1  | VoIP           | 192.168.0.0/24  | 192.168.0.10 -> 192.168.0.50   |
| VLAN 10 | Wi-Fi          | 192.168.10.0/24 | 192.168.10.10 -> 192.168.10.50 |
| VLAN 20 | PC fixes       | 192.168.20.0/24 | 192.168.20.10 -> 192.168.20.50 |
| VLAN 30 | Administration | 192.168.30.0/24 | 192.168.30.10 -> 192.168.30.50 |

---

## Étapes principales de la configuration

### 1. *Création des VLANs sur chaque switch*

    vlan 1
    vlan 10
    vlan 20
    vlan 30

### 2. *Affectation des ports*

     Ports 2–3    -->    VLAN 1
     Ports 4–5    -->    VLAN 10
     Ports 6–7    -->    VLAN 20
     Port 8       -->    VLAN 30
     Ports 1 et 9 -->    TRUNK

### 3. *Configuration du Router-on-a-Stick*

- Création des sous-interfaces :

      interface g0/0.1
      encapsulation dot1Q 1
      ip address 192.168.0.1 255.255.255.0
  
      interface g0/0.10
      encapsulation dot1Q 10
      ip address 192.168.10.1 255.255.255.0

      interface g0/0.20
      encapsulation dot1Q 20
      ip address 192.168.20.1 255.255.255.0

      interface g0/0.30
      encapsulation dot1Q 30
      ip address 192.168.30.1 255.255.255.0

### 4. *Configuration des pools DHCP*

    enable
    configure terminal

    - Exclusions DHCP
  
    ip dhcp excluded-address 192.168.0.1 192.168.0.9
    ip dhcp excluded-address 192.168.10.1 192.168.10.9
    ip dhcp excluded-address 192.168.20.1 192.168.20.9
    ip dhcp excluded-address 192.168.30.1 192.168.30.9

    - DHCP pools

    ip dhcp pool VLAN1
    network 192.168.0.0 255.255.255.0
    default-router 192.168.0.1
    dns-server 8.8.8.8

    ip dhcp pool VLAN10
    network 192.168.10.0 255.255.255.0
    default-router 192.168.10.1
    dns-server 8.8.8.8

    ip dhcp pool VLAN20
    network 192.168.20.0 255.255.255.0
    default-router 192.168.20.1
    dns-server 8.8.8.8
 
    ip dhcp pool VLAN30
    network 192.168.30.0 255.255.255.0
    default-router 192.168.30.1
    dns-server 8.8.8.8
  
  ---
### 4. *Simulation d’un accès Internet*
      Afin de permettre aux VLANs internes de “sortir” vers Internet, un deuxième routeur a été ajouté pour simuler un fournisseur d'accès.

- Simulation d'Internet avec Deux Routeurs
Architecture Wan:

      VLAN internes -> Routeur Principal -> Routeur Internet Simulé
      192.168.x.x        200.0.0.2/24          200.0.0.1/24
   
### 1. Configuration des Routeurs: 
- Routeur Internet (Simulation)

      hostname INTERNET
      int g0/0
      ip address 200.0.0.1 255.255.255.0
      no sh


      interface Loopback0
      ip address 8.8.8.8 255.255.255.255
      description Google-DNS-Simule

      ip route 0.0.0.0 0.0.0.0 Null0
      ip route 192.168.0.0 255.255.0.0 200.0.0.2

- Routeur Principal (Passerelle)

      hostname R-PRINCIPAL
      int g0/1
      ip address 200.0.0.2 255.255.255.0
      ip nat outside
      no sh

      ip nat inside source list 100 interface    GigabitEthernet0/1 overload
      access-list 100 permit ip 192.168.0.0 0.0.255.255 any

      ip route 0.0.0.0 0.0.0.0 200.0.0.1

      ip name-server 8.8.8.8
---
### 5. Test de connectivité

* Ping entre VLANs
* Ping vers Internet
* Test IP des téléphones
* Test Wi-Fi
---
##  Résultats obtenus

* Tous les équipements reçoivent une adresse IP via DHCP.
* Les VLANs communiquent grâce au routeur.
* Chaque bureau est autonome et bien organisé.
* Internet est accessible pour tous les réseaux.

---
## Captures d'écran
* topologie_minilab.PNG
* adressage.PNG
* Les client obtiennent des IP via DHCP Server
* Configuration des IP-Phones
* Configuration des AP
* ping des VLAN semblables
* ping entre VLAN différents
* test de l'acces internet: ping vers 8.8.8.8
* table de routage
* table arp

---
##  Contenu disponible sur le repo

 **README.md** (ce fichier)   
 **Fichier Packet Tracer (.pkt)**   
 **Exports des configurations des équipements**  
 **Captures d’écran** des tests et topologies

---
**Auteur :** Arthur FOGUE (Laplateforme_Test_Msc_Cyber)  
**Date :** 10/12/2025  
**Version :** 1.0  
**Statut :** Complété