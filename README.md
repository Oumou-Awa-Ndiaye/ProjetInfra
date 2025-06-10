# ProjetInfra
Projet de Maquette Réseau avec pfSense

## Introduction
Ce projet présente une maquette réseau complète utilisant pfSense comme firewall principal, avec segmentation en zones LAN et DMZ. La configuration est optimisée pour la sécurité tout en assurant la connectivité nécessaire entre les différents composants.

## Plan d'Adressage IP

| Équipement     | Interface   | Adresse IP       | Masque           | Passerelle     |
|----------------|-------------|------------------|------------------|----------------|
| pfSense        | em0 (WAN)   | 10.0.2.4         | 255.255.255.0    | 10.0.2.1       |
| pfSense        | em1 (LAN)   | 192.168.1.1      | 255.255.255.0    | -              |
| pfSense        | em2 (DMZ)   | 192.168.3.1      | 255.255.255.0    | -              |
| PC Admin       | Ethernet0   | 192.168.1.100    | 255.255.255.0    | 192.168.1.1    |
| Serveur Web    | Ethernet0   | 192.168.3.100    | 255.255.255.0    | 192.168.3.1    |

## Configuration des Équipements

### pfSense (ASA 5506-X)


! Configuration des interfaces
interface GigabitEthernet0/0  # WAN
 nameif WAN
 ip address 10.0.2.4 255.255.255.0
 security-level 0
 no shutdown

interface GigabitEthernet0/1  # LAN
 nameif LAN
 ip address 192.168.1.1 255.255.255.0
 security-level 100
 no shutdown

interface GigabitEthernet0/2  # DMZ
 nameif DMZ
 ip address 192.168.3.1 255.255.255.0
 security-level 50
 no shutdown

! Règles firewall
access-list DMZ_in extended permit tcp any host 192.168.3.100 eq www
access-list LAN_in extended permit icmp any any  # Autoriser ping pour tests

! NAT pour la DMZ
nat (DMZ) 0 access-list DMZ_in
global (WAN) 1 interface

## Serveur Web  
**OS**: Ubuntu Server 22.04 LTS  
**Services**: Apache2, SSH  

**Configuration IP statique :**  

network:
  version: 2
  ethernets:
    eth0:
      addresses: [192.168.3.100/24]
      routes:
        - to: default
          via: 192.168.3.1
      nameservers:
        addresses: [1.1.1.1]

        ## Bonnes Pratiques de Sécurité

- **Isolation des zones** : Séparation stricte LAN/DMZ  
- **Politique firewall** : "Deny by default" avec exceptions minimales  
- **Accès administrateur** : SSH uniquement depuis le LAN  
- **Mises à jour** : Automatisées via cron  
- **Monitoring** : Alertes sur activité suspecte  

---

## Services Mise en Œuvre

| Service | Hôte        | Ports | Accessibilité   |
|---------|-------------|-------|-----------------|
| HTTP    | Serveur Web | 80    | Internet        |
| SSH     | pfSense     | 22    | LAN seulement   |
| DNS     | Cloudflare  | 53    | Tous            |