# Guide Avancé du Protocole RIP (Routing Information Protocol)

**Auteur : Hackers_Tchad**  
**Version : 3.0**  
**Date : 2025**  
**Licence : Libre pour apprentissage et recherche**

---

## Table des matières

1. [Introduction](#1-introduction)
2. [Définition de RIP](#2-définition-de-rip)
3. [Historique du protocole RIP](#3-historique-du-protocole-rip)
4. [Principes fondamentaux du routage à vecteur de distance](#4-principes-fondamentaux-du-routage-à-vecteur-de-distance)
5. [Fonctionnement interne de RIP](#5-fonctionnement-interne-de-rip)
6. [Versions de RIP](#6-versions-de-rip)
7. [Comparaison RIP v1 et RIP v2](#7-comparaison-rip-v1-et-rip-v2)
8. [Format des messages RIP](#8-format-des-messages-rip)
9. [Métrique et comptage à l'infini](#9-métrique-et-comptage-à-linfini)
10. [Mécanismes de prévention des boucles](#10-mécanismes-de-prévention-des-boucles)
11. [Configuration avancée de RIP](#11-configuration-avancée-de-rip)
12. [Configuration sur Cisco IOS](#12-configuration-sur-cisco-ios)
13. [Configuration sur routeurs open-source](#13-configuration-sur-routeurs-open-source)
14. [Configuration sous Linux avec Quagga/FRRouting](#14-configuration-sous-linux-avec-quaggafrrouting)
15. [Configuration sous pfSense/OPNsense](#15-configuration-sous-pfsenseopnsense)
16. [Dépannage et commandes de vérification](#16-dépannage-et-commandes-de-vérification)
17. [Cas d'usage et déploiement](#17-cas-dusage-et-déploiement)
18. [Sécurité RIP](#18-sécurité-rip)
19. [Limites et alternatives](#19-limites-et-alternatives)
20. [RIP dans les certifications réseau](#20-rip-dans-les-certifications-réseau)
21. [Ressources et liens](#21-ressources-et-liens)
22. [Bibliographie et livres recommandés](#22-bibliographie-et-livres-recommandés)
23. [Glossaire](#23-glossaire)
24. [Conclusion](#24-conclusion)

---

## 1. Introduction

Le **Routing Information Protocol (RIP)** est l'un des plus anciens protocoles de routage dynamique utilisés dans les réseaux informatiques. Conçu à l'époque d'ARPANET, il a été normalisé pour permettre aux routeurs d'échanger automatiquement des informations de routage au sein d'un réseau. Bien que RIP soit aujourd'hui considéré comme un protocole hérité face à des solutions plus modernes telles que OSPF, EIGRP ou BGP, il demeure un outil pédagogique incontournable pour comprendre les bases du routage dynamique.

Ce guide, rédigé par **Hackers_Tchad**, propose une exploration approfondie du protocole RIP : sa définition, son fonctionnement, ses versions, sa configuration sur différentes plateformes, ainsi que des ressources complémentaires pour approfondir ses connaissances.

---

## 2. Définition de RIP

**RIP** est un protocole de routage **intérieur (IGP - Interior Gateway Protocol)** qui utilise l'algorithme de routage à **vecteur de distance**. Il permet à des routeurs situés au sein d'un même système autonome d'échanger des informations concernant les réseaux qu'ils connaissent et la distance qui les sépare de ces réseaux.

### Caractéristiques essentielles

- **Type de protocole** : IGP (Interior Gateway Protocol)
- **Algorithme** : Vecteur de distance (Bellman-Ford)
- **Métrique** : Nombre de sauts (hop count)
- **Métrique maximale** : 15 sauts (16 signifie inaccessible)
- **Protocole de transport** : UDP, port 520
- **Adresse de diffusion** : 255.255.255.255 (broadcast) ou 224.0.0.9 (multicast pour RIPv2)
- **Fréquence de mise à jour** : 30 secondes par défaut
- **Convergence** : Lente comparée aux protocoles à état de lien

RIP est particulièrement adapté aux petits réseaux homogènes où la complexité de configuration doit rester minimale.

---

## 3. Historique du protocole RIP

RIP trouve son origine dans le réseau **ARPANET** des années 1960. Il a été développé à partir de l'algorithme de **Bellman-Ford**, lui-même issu des travaux des mathématiciens Richard Bellman et Lester Ford.

### Chronologie

- **1969** : Premières implémentations expérimentales sur ARPANET.
- **1988** : RIP est formalisé dans le RFC 1058 par Charles Hedrick.
- **1994** : Publication de RIP version 2 dans le RFC 1723, puis mis à jour par le RFC 2453.
- **1997** : RIPng (RIP Next Generation) est défini dans le RFC 2080 pour supporter IPv6.

RIP a longtemps été le protocole de routage par défaut dans les réseaux locaux et les systèmes Unix grâce au démon **routed**. Aujourd'hui, il est encore enseigné dans les cursus réseau et présent dans de nombreux environnements hérités.

---

## 4. Principes fondamentaux du routage à vecteur de distance

Le routage à vecteur de distance repose sur une idée simple : chaque routeur connaît la distance qui le sépare de chaque destination et informe régulièrement ses voisins de ces distances.

### Concepts clés

- **Vecteur de distance** : une entrée de table de routage contient la destination et le coût pour l'atteindre.
- **Mise à jour périodique** : les routeurs envoient leur table complète à leurs voisins à intervalles réguliers.
- **Apprentissage par rumeur** : un routeur apprend les routes indirectement via ses voisins, sans connaître la topologie complète.

### Avantages

- Simplicité de configuration et de compréhension.
- Faible charge CPU sur les petits réseaux.
- Interopérabilité entre équipements de différents constructeurs.

### Inconvénients

- Lenteur de convergence en cas de changement topologique.
- Risque de boucles de routage.
- Limite de 15 sauts, inadapté aux grands réseaux.
- Consommation de bande passante due aux mises à jour périodiques.

---

## 5. Fonctionnement interne de RIP

RIP fonctionne selon un processus d'échange périodique de tables de routage entre routeurs voisins.

### Étapes principales

1. **Initialisation** : au démarrage, chaque routeur connaît uniquement les réseaux directement connectés.
2. **Échange de routes** : les routeurs envoient leurs tables de routage aux voisins via UDP port 520.
3. **Mise à jour des tables** : chaque routeur reçoit les tables de ses voisins, ajoute 1 au coût de chaque route et met à jour sa propre table si une meilleure route est trouvée.
4. **Périodicité** : l'échange se répète toutes les 30 secondes.
5. **Invalidation** : une route non rafraîchie après 180 secondes est marquée comme invalide.
6. **Suppression** : après 240 secondes, la route est retirée de la table.

### Exemple de propagation

Supposons trois routeurs A, B et C en ligne :

- A connaît le réseau 192.168.1.0/24 (coût 0).
- B apprend 192.168.1.0/24 depuis A avec un coût de 1.
- C apprend 192.168.1.0/24 depuis B avec un coût de 2.

Ainsi, la distance en nombre de sauts se propage progressivement à travers le réseau.

---

## 6. Versions de RIP

Il existe trois versions principales de RIP.

### RIP v1

- Défini dans le RFC 1058.
- Ne supporte que les masques de sous-réseau par classe (classful).
- Diffuse les mises à jour en broadcast (255.255.255.255).
- Ne transporte pas le masque de sous-réseau dans les mises à jour.
- Inadapté aux réseaux modernes utilisant le VLSM.

### RIP v2

- Défini dans les RFC 1723 et RFC 2453.
- Supporte le CIDR et le VLSM grâce à l'envoi du masque de sous-réseau.
- Utilise l'adresse multicast 224.0.0.9.
- Supporte l'authentification MD5 ou texte simple.
- Rétrocompatible avec RIP v1.

### RIPng

- Défini dans le RFC 2080.
- Extension de RIP v2 pour IPv6.
- Utilise UDP port 521.
- Adresse multicast FF02::9.
- Ne supporte pas l'authentification nativement ; s'appuie sur IPsec si besoin.

---

## 7. Comparaison RIP v1 et RIP v2

| Caractéristique        | RIP v1              | RIP v2                  |
|--------------------------|---------------------|-------------------------|
| RFC                      | 1058                | 1723, 2453              |
| Classful / Classless     | Classful            | Classless               |
| Masque dans les updates  | Non                 | Oui                     |
| Diffusion                | Broadcast           | Multicast 224.0.0.9     |
| Authentification         | Non                 | Texte ou MD5            |
| Tag de route             | Non                 | Oui                     |
| Compatibilité            | RIP v1 uniquement   | RIP v1 et v2            |

RIP v2 est recommandé pour tous les nouveaux déploiements car il supporte les réseaux modernes segmentés.

---

## 8. Format des messages RIP

### En-tête RIP

Chaque message RIP commence par un en-tête de 4 octets :

- **Commande** (1 octet) : 1 = Request, 2 = Response.
- **Version** (1 octet) : 1 pour RIPv1, 2 pour RIPv2.
- **Must Be Zero** (2 octets) : réservé, doit être à zéro.

### Entrée de route RIP v2

Chaque entrée de route occupe 20 octets :

- **AFI** (Address Family Identifier) : 2 pour IPv4.
- **Route Tag** : identifiant pour les routes redistribuées.
- **Adresse IP** : adresse du réseau de destination.
- **Masque de sous-réseau** : supporté uniquement en RIPv2.
- **Next Hop** : adresse du prochain saut.
- **Métrique** : nombre de sauts, de 1 à 15.

---

## 9. Métrique et comptage à l'infini

### Métrique RIP

La métrique utilisée par RIP est le **nombre de sauts (hop count)**. Chaque routeur traversé augmente le coût de 1. Le maximum acceptable est de 15 sauts. Une métrique de 16 signifie que le réseau est inaccessible.

### Comptage à l'infini

Le comptage à l'infini est un problème classique des protocoles à vecteur de distance. Lorsqu'un lien tombe, les routeurs peuvent continuer à s'annoncer mutuellement des routes invalides, faisant croître la métrique jusqu'à 16.

**Exemple** :

- A annonce 10.0.0.0/8 à B avec un coût de 1.
- B annonce 10.0.0.0/8 à A avec un coût de 2.
- Si le lien de A vers 10.0.0.0/8 tombe, A peut croire qu'il peut encore atteindre le réseau via B avec un coût de 3.
- La boucle se forme jusqu'à ce que la métrique atteigne 16.

---

## 10. Mécanismes de prévention des boucles

Plusieurs techniques ont été introduites pour limiter le comptage à l'infini.

### Split Horizon

Un routeur n'annonce pas une route sur l'interface par laquelle il l'a apprise. Cela réduit les boucles simples.

### Route Poisoning

Lorsqu'un route dévient inaccessible, le routeur l'annonce avec une métrique de 16 (poisoned route) pour informer rapidement ses voisins.

### Poison Reverse

Variante du split horizon : une route apprise via une interface est annoncée sur cette même interface avec une métrique de 16.

### Hold-Down Timer

Lorsqu'une route est marquée comme inaccessible, le routeur refuse toute mise à jour concernant cette route pendant une période donnée (par défaut 180 secondes), évitant ainsi l'acceptation d'informations erronées.

### Triggered Updates

En cas de changement topologique, le routeur envoie immédiatement une mise à jour sans attendre les 30 secondes réglementaires.

---

## 11. Configuration avancée de RIP

### Bonnes pratiques

- Utiliser RIPv2 ou RIPng plutôt que RIPv1.
- Désactiver l'auto-summarization sur les réseaux discontinus.
- Activer l'authentification pour sécuriser les échanges.
- Filtrer les interfaces qui ne doivent pas envoyer de mises à jour RIP.
- Ajuster les timers en fonction de la taille du réseau.

### Timers RIP

- **Update timer** : 30 secondes.
- **Invalid timer** : 180 secondes.
- **Hold-down timer** : 180 secondes.
- **Flush timer** : 240 secondes.

Ces valeurs peuvent être modifiées sur la plupart des équipements.

---

## 12. Configuration sur Cisco IOS

### Configuration de base RIPv2

```cisco
Router> enable
Router# configure terminal
Router(config)# router rip
Router(config-router)# version 2
Router(config-router)# no auto-summary
Router(config-router)# network 192.168.1.0
Router(config-router)# network 10.0.0.0
Router(config-router)# passive-interface GigabitEthernet0/0
Router(config-router)# exit
Router(config)# interface GigabitEthernet0/1
Router(config-if)# ip rip authentication mode md5
Router(config-if)# ip rip authentication key-chain RIP_KEY
```

### Vérification

```cisco
Router# show ip protocols
Router# show ip route rip
Router# show ip rip database
Router# debug ip rip
```

### Explication

- `version 2` active RIPv2.
- `no auto-summary` désactive la summarisation automatique par classe.
- `network` indique les réseaux directement connectés à inclure dans RIP.
- `passive-interface` empêche l'envoi de mises à jour sur une interface tout en continuant à l'écouter.

---

## 13. Configuration sur routeurs open-source

Plusieurs solutions open-source implémentent RIP, notamment **Quagga**, **FRRouting**, **BIRD** et **VyOS**.

### Quagga / FRRouting

Ces démons de routage offrent une interface en ligne de commande similaire à Cisco IOS.

### BIRD

BIRD utilise un langage de configuration différent mais supporte RIP via le protocole `rip`.

### VyOS

VyOS, fork de Vyatta, permet la configuration de RIP via une CLI structurée.

---

## 14. Configuration sous Linux avec Quagga/FRRouting

### Installation de FRRouting

```bash
sudo apt update
sudo apt install frr
```

### Activation du démon RIP

```bash
sudo sed -i 's/ripd=no/ripd=yes/' /etc/frr/daemons
sudo systemctl restart frr
```

### Configuration RIPv2

```bash
sudo vtysh

configure terminal
router rip
 version 2
 no auto-summary
 network 192.168.1.0/24
 network 10.0.0.0/24
 passive-interface eth0
 exit
interface eth1
 ip rip authentication mode md5
 ip rip authentication key-chain MYKEY
 exit
end
write memory
```

### Vérification

```bash
show ip rip
show ip rip status
show ip route rip
show running-config
```

---

## 15. Configuration sous pfSense/OPNsense

pfSense et OPNsense utilisent FRRouting pour le routage dynamique via le package `frr`.

### Étapes

1. Installer le package FRR dans `System > Package Manager`.
2. Naviguer vers `Services > FRR RIP`.
3. Activer RIP et choisir la version (v2 ou ng).
4. Ajouter les interfaces participantes.
5. Définir les réseaux à annoncer.
6. Configurer éventuellement l'authentification.

Ces distributions sont particulièrement adaptées aux petites et moyennes infrastructures.

---

## 16. Dépannage et commandes de vérification

### Commandes Cisco

| Commande                  | Utilisation                              |
|---------------------------|------------------------------------------|
| `show ip protocols`       | Affiche les protocoles de routage actifs |
| `show ip route`           | Affiche la table de routage              |
| `show ip route rip`       | Filtre les routes RIP                    |
| `show ip rip database`    | Affiche la base de données RIP           |
| `debug ip rip`            | Active le débogage RIP                   |
| `show interfaces`         | Vérifie l'état des interfaces            |

### Problèmes courants

- **Routes absentes** : vérifier les instructions `network`, les masques et l'état des interfaces.
- **Boucles de routage** : vérifier la cohérence des timers et la présence de split horizon.
- **Mises à jour non reçues** : vérifier les ACL, le multicast et l'authentification.
- **Métrique de 16** : indique une route inaccessible ou un réseau hors portée.

---

## 17. Cas d'usage et déploiement

### Quand utiliser RIP ?

- Petits réseaux avec moins de 15 sauts.
- Environnements hérités nécessitant la compatibilité.
- Réseaux homogènes avec peu de changements topologiques.
- Formation et laboratoires réseau.
- Équipements embarqués ou routeurs légers supportant uniquement RIP.

### Quand éviter RIP ?

- Grands réseaux d'entreprise.
- Topologies nécessitant une convergence rapide.
- Réseaux utilisant intensivement le VLSM et le CIDR.
- Environnements critiques nécessitant une haute disponibilité.

---

## 18. Sécurité RIP

### Risques

- **Usurpation de routes** : un attaquant peut injecter de fausses routes.
- **Écoute passive** : les mises à jour RIP circulent en clair par défaut.
- **Déni de service** : saturation par de faux messages RIP.

### Mesures de protection

- Activer l'authentification MD5 entre voisins.
- Limiter les interfaces participant à RIP.
- Utiliser des ACL pour filtrer les sources de mises à jour.
- Préférer OSPF ou EIGRP dans des environnements sensibles.

---

## 19. Limites et alternatives

### Limites de RIP

- Convergence lente.
- Limite de 15 sauts.
- Bande passante consommée par les mises à jour périodiques.
- Pas de prise en charge native du QoS ou du TE.

### Alternatives

- **OSPF** : protocole à état de lien, convergence rapide, supporte de grands réseaux.
- **EIGRP** : protocole hybride propriétaire Cisco, très performant.
- **IS-IS** : protocole à état de lien utilisé par les opérateurs.
- **BGP** : protocole de routage externe pour l'Internet.

---

## 20. RIP dans les certifications réseau

RIP est un sujet classique des certifications réseau.

- **Cisco CCNA** : configuration et dépannage de RIPv2.
- **CompTIA Network+** : principes des protocoles de routage.
- **Juniper JNCIA** : compréhension des IGP.
- **LPIC / Linux Foundation** : routage avec Quagga/FRRouting.

Maîtriser RIP est souvent une étape préalable à l'étude d'OSPF et de BGP.

---

## 21. Ressources et liens

### RFC officiels

- RFC 1058 — Routing Information Protocol : https://tools.ietf.org/html/rfc1058
- RFC 1723 — RIP Version 2 : https://tools.ietf.org/html/rfc1723
- RFC 2453 — RIP Version 2 (mise à jour) : https://tools.ietf.org/html/rfc2453
- RFC 2080 — RIPng for IPv6 : https://tools.ietf.org/html/rfc2080

### Documentation FRRouting

- https://frrouting.org/
- https://docs.frrouting.org/

### Documentation Cisco

- https://www.cisco.com/c/en/us/support/docs/ip/routing-information-protocol-rip/16448-rip-1.html
- https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/iproute_rip/configuration/xe-16/iri-xe-16-book/iri-cfg-rip.html

### Projets open-source

- Quagga : https://www.nongnu.org/quagga/
- FRRouting : https://frrouting.org/
- BIRD : https://bird.network.cz/
- VyOS : https://vyos.io/

---

## 22. Bibliographie et livres recommandés

### Ouvrages fondamentaux

1. **Andrew S. Tanenbaum, David J. Wetherall — Computer Networks**  
   Référence académique sur les réseaux, couvrant RIP et les algorithmes de routage.

2. **James F. Kurose, Keith W. Ross — Computer Networking: A Top-Down Approach**  
   Approche pédagogique du routage et des protocoles réseau.

3. **Larry L. Peterson, Bruce S. Davie — Computer Networks: A Systems Approach**  
   Analyse technique des protocoles de routage intérieurs.

4. **W. Richard Stevens — TCP/IP Illustrated, Volume 1**  
   Classique sur la pile TCP/IP, incluant les protocoles de routage.

5. **Cisco Press — Routing TCP/IP, Volume 1**  
   Guide avancé sur RIP, IGRP, EIGRP et OSPF.

6. **Todd Lammle — CCNA Routing and Switching Study Guide**  
   Préparation au CCNA avec chapitres dédiés à RIP et OSPF.

7. **Wendell Odom — CCNA 200-301 Official Cert Guide**  
   Référence officielle Cisco pour le CCNA.

### Articles et cours en ligne

- Cours MIT 6.829 : Computer Networks
- Cours Stanford CS144 : Computer Network
- Cours Réseaux de Pierre Loisel (Université de Sherbrooke)

---

## 23. Glossaire

- **IGP** : Interior Gateway Protocol, protocole de routage interne à un système autonome.
- **EGP** : Exterior Gateway Protocol, protocole de routage externe entre systèmes autonomes.
- **AS** : Autonomous System, ensemble de réseaux gérés par une entité unique.
- **CIDR** : Classless Inter-Domain Routing, routage sans classes.
- **VLSM** : Variable Length Subnet Masking, masquage de sous-réseau à longueur variable.
- **Hop count** : nombre de sauts, métrique utilisée par RIP.
- **Convergence** : état où tous les routeurs ont une vision cohérente du réseau.
- **Split horizon** : technique de prévention des boucles.
- **Route poisoning** : annonce d'une route avec une métrique infinie.

---

## 24. Conclusion

Le protocole RIP, malgré son âge, reste une brique fondamentale de la compréhension du routage dynamique. Sa simplicité en fait un excellent outil pédagogique et une solution viable pour les petits réseaux. Cependant, ses limites en termes de convergence, de métrique et de sécurité le rendent inadapté aux infrastructures modernes de grande taille.

Ce guide a été rédigé avec rigueur par **Hackers_Tchad** dans un but éducatif. Pour aller plus loin, il est recommandé d'expérimenter RIP dans un laboratoire avec Cisco Packet Tracer, GNS3, EVE-NG ou FRRouting, puis d'étudier OSPF et BGP pour comprendre les protocoles utilisés dans les réseaux professionnels.

---

**Fin du guide.**
