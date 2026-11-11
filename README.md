# Audit WiFi — Capture de handshake WPA2 et test par dictionnaire

Laboratoire de sécurité sans fil réalisé dans le cadre du cours **Piratage et Sécurité II (420-5RH-HY)** au Cégep de Saint-Hyacinthe. L'objectif est de comprendre, de bout en bout, comment un réseau WPA2-PSK peut être audité : provoquer une reconnexion, capturer le *4-way handshake* et le tester hors ligne contre un dictionnaire.

> **Cadre.** Travail académique réalisé en classe sur un **routeur assigné par l'enseignant**, avec un **client autorisé**, en **environnement contrôlé**. Aucune action n'a visé un réseau tiers. Le mot de passe retrouvé n'est pas publié dans ce dépôt.

---

## Contexte

L'ESP32 est un microcontrôleur avec radio WiFi. Il n'est pas un outil d'attaque en soi : son comportement dépend du firmware téléversé. Ici, le firmware **ESP32-Deauther** exploite les trames de gestion non protégées du WiFi (802.11) pour provoquer une déconnexion. La reconnexion du client qui suit permet de capturer le handshake, que l'on teste ensuite hors ligne — le mot de passe n'est jamais transmis en clair sur le réseau.

## Matériel et outils

| Élément | Rôle |
| --- | --- |
| ESP32 + firmware ESP32-Deauther | Provoque une déconnexion contrôlée du client |
| Carte WiFi USB (RTL8821AU) sur Kali Linux | Passe en mode moniteur et capture les trames |
| Routeur RTR06 (WPA2-PSK) | Point d'accès cible, assigné par l'enseignant |
| airodump-ng, aircrack-ng, airmon-ng | Capture, validation et test par dictionnaire |

## Méthode

1. **Personnalisation du firmware** — compilation et téléversement du firmware ESP32-Deauther avec un SSID unique (PlatformIO / VS Code).
2. **Préparation de la carte** — vérification de l'interface, du pilote et du support du mode moniteur avant toute capture (`lsusb`, `iw list`, `airmon-ng`).
3. **Identification de la cible** — repérage du routeur assigné par son BSSID, son canal et le client associé (`airodump-ng`).
4. **Capture du handshake** — capture ciblée sur le bon canal et le bon BSSID, pendant une déconnexion brève provoquée par l'ESP32, suivie de la reconnexion du client.
5. **Test par dictionnaire** — validation du handshake capturé, puis test hors ligne des candidats du dictionnaire avec `aircrack-ng`.
6. **Remise en état** — retour de la carte en mode managed, aucun paramètre du routeur modifié.

## Le handshake WPA2 en bref

Le *4-way handshake* vérifie que le client possède le secret partagé **sans jamais le transmettre en clair**. Seuls des nonces, des MIC et des clés dérivées circulent. C'est précisément ce qui rend possible un test **hors ligne** : chaque candidat du dictionnaire est utilisé pour recalculer le MIC, que l'on compare à celui capturé.

## Résultat

Le handshake a été capturé et validé, puis un candidat du dictionnaire a reproduit le MIC capturé — ce qui confirme qu'il s'agit de la passphrase du routeur assigné. **Le mot de passe en clair n'est pas inclus dans ce dépôt.**

## Ce que ce lab démontre, côté défense

- Un réseau WPA2-PSK avec une passphrase faible ou présente dans un dictionnaire est vulnérable à une attaque hors ligne, même sans interaction avec le routeur.
- **PMF (802.11w)** protège les trames de gestion contre la désauthentification forgée.
- La vraie défense tient à la robustesse de la passphrase (longue, aléatoire) et, idéalement, au passage à WPA3.

## Portée et limites

Un test par dictionnaire qui échoue ne prouve pas qu'un réseau est inviolable : il indique seulement que la passphrase ne figurait pas dans le dictionnaire utilisé. Un dictionnaire plus large ou une autre méthode pourraient donner un résultat différent.

---

*Rapport technique complet (16 pages, captures à l'appui) disponible sur demande.*
