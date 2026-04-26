# Fondamentaux Réseau : Comment les données circulent

> Mes notes de formation Jedha. Écrit pour être relu dans 3 mois sans tout reconstruire de zéro. Si quelque chose est flou à la relecture, c'est que je n'avais pas encore bien compris quand j'ai écrit.

## Sommaire

1. [Partie 1 : Comment les données circulent dans le réseau](#partie-1--comment-les-données-circulent-dans-le-réseau)
   * NIC
   * IP
   * DHCP
   * DNS
   * Modèle OSI
2. [Partie 2 : ARP, Routing et NAT](#partie-2--arp-routing-et-nat)
   * ARP, ARP Cache, ARP Table
   * Routing
   * NAT

---

## Partie 1 : Comment les données circulent dans le réseau

### Pourquoi je commence par là

Avant de toucher quoi que ce soit en sécu, je dois comprendre ce qui circule sur un réseau et comment. L'attaquant et le défenseur jouent tous les deux sur ce terrain. Si je ne sais pas comment un paquet voyage, je ne comprendrai jamais pourquoi certaines attaques fonctionnent.

Le scénario que je garde en tête tout au long : je tape une URL dans mon navigateur. Que se passe-t-il vraiment ?

---

### NIC : ma carte réseau

La **NIC** (*Network Interface Card*) c'est le composant physique (ou virtuel) qui me connecte à un réseau. J'en ai potentiellement plusieurs sur ma machine : une pour le wifi, une pour l'ethernet, parfois une virtuelle.

Ce que je retiens absolument : chaque NIC a une **adresse MAC** (*Media Access Control*), gravée par le fabricant, théoriquement unique au monde. Format : `A4:5E:60:C1:23:F7` (six paires hexadécimales).

La MAC c'est mon **identité physique** sur le réseau local. Elle ne change pas quand je change de réseau. Elle suit le matériel.

Distinction à ne jamais confondre :
* **MAC** = identité physique, locale
* **IP** = identité logique, positionnelle

Je peux changer d'IP en changeant de réseau. Ma MAC reste la même (sauf *MAC spoofing*, technique qu'on verra plus tard).

---

### IP : mon adresse sur le réseau

**IP** = *Internet Protocol*. L'adresse IP c'est l'adresse postale de ma machine sur un réseau.

Deux versions :
* **IPv4** : `192.168.1.42` (quatre nombres de 0 à 255)
* **IPv6** : beaucoup plus long, créé pour pallier la pénurie d'IPv4

#### IP privée vs IP publique

C'est un point fondamental que j'ai besoin d'avoir ancré.

**IP privée** : l'adresse de ma machine à l'intérieur de mon réseau local. Attribuée par ma box. Plages réservées :
* `192.168.x.x`
* `10.x.x.x`
* `172.16.x.x` à `172.31.x.x`

Ces IP ne sortent pas sur internet. Elles n'existent qu'en local.

**IP publique** : l'adresse que ma box présente à internet. Tous mes appareils partagent la même IP publique vue de l'extérieur.

L'analogie que je garde : mon IP publique c'est l'adresse de l'immeuble. Mon IP privée c'est mon numéro d'appartement. Internet voit l'immeuble, pas l'appart. La box fait l'intermédiaire. C'est le **NAT** (partie 2).

---

### DHCP : comment j'obtiens mon IP automatiquement

**DHCP** = *Dynamic Host Configuration Protocol*.

Quand je connecte mon téléphone à un wifi, il obtient une IP sans que je fasse quoi que ce soit. C'est DHCP qui s'en charge.

#### L'échange DORA

L'attribution se fait en 4 étapes, je retiens l'acronyme **DORA** :

1. **D**iscover : mon appareil crie sur le réseau "y'a un serveur DHCP ici ?"
2. **O**ffer : le serveur DHCP (ma box) répond "oui, je te propose l'IP `192.168.1.42`"
3. **R**equest : mon appareil confirme "ok je prends cette IP"
4. **A**cknowledge : le serveur valide "c'est bon, elle est à toi"

À l'issue du DORA, mon appareil a reçu :
* Son IP privée
* Le masque de sous-réseau
* L'IP de la passerelle (ma box)
* L'IP du serveur DNS

**Notion de bail (lease)** : l'IP est prêtée pour une durée limitée (souvent 24h). Si j'éteins ma machine une semaine, mon IP peut être réattribuée à un autre appareil.

#### Pourquoi c'est important en sécu

Un faux serveur DHCP sur un réseau peut distribuer de mauvaises configurations à toutes les machines : fausse passerelle, faux DNS. Toutes les connexions passent alors par l'attaquant sans que personne s'en rende compte. C'est du **DHCP spoofing**.

---

### DNS : l'annuaire qui traduit les noms en IP

**DNS** = *Domain Name System*.

Je tape `google.com`. Mon ordi ne sait pas communiquer avec un nom, il lui faut une IP. Le DNS fait la traduction.

#### Comment ça se passe

1. Je tape `google.com`
2. Mon ordi demande au serveur DNS configuré : "c'est quoi l'IP de google.com ?"
3. Réponse : `142.250.74.110`
4. Mon ordi peut établir la connexion

En réalité la résolution est hiérarchique si le DNS local ne connaît pas la réponse :
* Il interroge un **serveur racine** (qui connaît les serveurs des extensions)
* Qui pointe vers le **serveur TLD** (.com, .fr, .org...)
* Qui pointe vers le **serveur autoritaire** du domaine concerné
* Qui donne la réponse définitive

**Le cache DNS** : pour ne pas refaire ce chemin à chaque fois, les réponses sont mises en cache. Mon ordi, ma box, mon FAI, tous gardent une copie temporaire.

#### Pourquoi c'est important en sécu

Si un attaquant corrompt les réponses DNS (**DNS spoofing** ou **DNS poisoning**), il peut rediriger `mabanque.fr` vers un site pirate visuellement identique. Je donne mes identifiants, il les récupère. Le DNS est une cible de choix précisément parce que je lui fais confiance implicitement.

---

### Le modèle OSI : la carte mentale du réseau

**OSI** = *Open Systems Interconnection*.

Modèle théorique en 7 couches qui décrit comment les données voyagent d'une machine à une autre. En production on utilise plutôt TCP/IP (plus simple), mais OSI reste la référence pour comprendre **où** se passent les choses. Quand j'analyserai du trafic avec Wireshark, je me référerai constamment à ces couches.

#### Les 7 couches

| Couche | Nom | Rôle | Exemples |
|--------|-----|------|---------|
| 7 | Application | Ce que voit l'utilisateur | HTTP, HTTPS, FTP, DNS |
| 6 | Présentation | Format et chiffrement | TLS/SSL, encodage |
| 5 | Session | Ouverture/fermeture des dialogues | Maintien de session |
| 4 | Transport | Découpage, fiabilité | TCP, UDP |
| 3 | Réseau | Adressage logique, routage | IP, ICMP |
| 2 | Liaison | Adressage physique local | MAC, Ethernet, Wifi |
| 1 | Physique | Le signal brut | Câble, fibre, ondes |

#### Comment je lis ce modèle

Quand j'envoie une donnée, elle **descend** les couches chez moi (chaque couche ajoute son enveloppe), traverse le réseau, puis **remonte** les couches chez le destinataire (chaque couche retire son enveloppe).

Analogie que je garde : une lettre. J'écris le message (couche 7), je le mets dans une enveloppe adressée (couche 3), la poste le met dans un sac régional (couche 2), le sac part dans un camion (couche 1). À l'arrivée : déballage dans l'ordre inverse.

#### Moyen mnémotechnique (couche 7 à 1)

*All People Seem To Need Data Processing*
(Application, Presentation, Session, Transport, Network, Data link, Physical)

#### Ce que je dois retenir absolument

* Adresse **MAC** = couche **2**
* Adresse **IP** = couche **3**
* **TCP/UDP** = couche **4**
* **HTTP, DNS, FTP** = couche **7**

---

## Partie 2 : ARP, Routing et NAT

Je sais maintenant qui parle (NIC, IP) et comment les noms se résolvent (DNS). Mais comment un paquet trouve concrètement son chemin de A à B ?

---

### ARP : faire le lien entre IP et MAC

**ARP** = *Address Resolution Protocol*.

#### Le problème qu'ARP résout

Sur internet, on utilise des **IP** (couche 3). Mais sur le réseau local, un paquet doit être livré physiquement à une carte réseau, donc à une **MAC** (couche 2).

Quand mon ordi veut parler à `192.168.1.1` (ma box), il connaît l'IP. Mais pour livrer le paquet sur le réseau local, il lui faut la MAC. ARP répond à cette question : *quelle MAC correspond à cette IP ?*

#### Comment ARP fonctionne

Mon ordi veut parler à `192.168.1.1` mais ne connaît pas sa MAC :

1. Il envoie un message à **tout le monde** sur le réseau local (broadcast) : "qui a l'IP `192.168.1.1` ? Réponds avec ta MAC." C'est une **ARP request**.
2. La machine concernée répond : "c'est moi, ma MAC est `A4:5E:60:C1:23:F7`." C'est une **ARP reply**.
3. Mon ordi peut maintenant adresser son paquet à la bonne MAC.

Simple. Mais refaire ça à chaque paquet serait catastrophique en termes de performance.

#### ARP Cache et ARP Table

Mon OS garde en mémoire les correspondances IP/MAC déjà découvertes.

* **ARP Cache** : la mémoire temporaire de ces correspondances
* **ARP Table** : la structure qui contient les entrées (les deux termes sont souvent interchangeables)

Je peux voir ma table :
```bash
# Windows
arp -a

# Linux / Mac
arp -a
# ou
ip neigh
```

Résultat typique :
```
192.168.1.1     a4:5e:60:c1:23:f7    dynamic
192.168.1.42    b8:27:eb:11:22:33    dynamic
```

Les entrées `dynamic` ont été apprises par ARP. Elles expirent après quelques minutes car un appareil peut quitter le réseau à tout moment.

#### Pourquoi c'est important en sécu

**ARP poisoning** (ou ARP spoofing) : j'envoie de fausses réponses ARP sur le réseau pour faire croire à une machine que la MAC de la passerelle c'est en fait la mienne. Tout le trafic destiné à la passerelle passe alors par moi. Je peux le lire, le modifier, le bloquer.

C'est une attaque **Man-in-the-Middle** classique sur réseau local. Je vais forcément la croiser en pratique très rapidement.

---

### Routing : faire voyager les paquets entre réseaux

ARP gère la comm **à l'intérieur** d'un réseau local. Pour passer d'un réseau à un autre, de chez moi vers Google par exemple, il faut du **routing**.

#### La passerelle par défaut

Chaque réseau local a une **passerelle par défaut** (*default gateway*), c'est la porte de sortie. Chez moi, c'est ma box.

Quand mon ordi veut envoyer un paquet vers une IP qui n'est **pas dans son réseau local**, il l'envoie à la passerelle. À charge pour elle de savoir quoi en faire.

Comment mon ordi sait si une IP est locale ou pas : grâce au **masque de sous-réseau**. Il fait une comparaison entre l'IP de destination et son propre réseau. Même réseau = ARP direct. Réseau différent = envoie à la passerelle.

#### La table de routage

Un routeur consulte sa **table de routage** pour décider où envoyer chaque paquet. Elle contient des règles comme :
* "Pour aller vers le réseau X, passe par tel voisin"
* "Pour tout le reste, utilise la route par défaut"

#### Le voyage complet d'un paquet

Je veux joindre `142.250.74.110` (Google) :

1. Mon ordi : "cette IP n'est pas dans mon réseau, j'envoie à ma box"
2. Ma box : "je ne connais pas cette route, j'envoie à mon routeur supérieur (FAI)"
3. De routeur en routeur, le paquet progresse vers Google
4. La réponse fait le chemin inverse

Chaque saut s'appelle un **hop**. Je peux les observer :
```bash
traceroute google.com   # Linux / Mac
tracert google.com      # Windows
```

**Point conceptuel important** : à chaque saut, l'IP source et destination **ne changent pas** (identité globale de bout en bout). Mais la MAC source et destination **change à chaque saut** (identité locale, valable uniquement pour le saut en cours).

IP = qui je suis dans le monde. MAC = qui je suis pour mon voisin direct.

---

### NAT : la traduction d'adresses

**NAT** = *Network Address Translation*.

#### Le problème

Mes appareils ont des IP privées qui ne sont pas routables sur internet. Comment font-ils pour communiquer quand même avec des serveurs extérieurs ?

#### Comment ça fonctionne

Ma box a deux faces :
* **Face interne** : IP privée `192.168.1.1`, parle à mes appareils
* **Face externe** : IP publique attribuée par mon FAI, parle à internet

Mon ordi (`192.168.1.42`) envoie un paquet vers Google :

1. Paquet arrive à la box : source = `192.168.1.42`, destination = `142.250.74.110`
2. La box **réécrit la source** : `192.168.1.42` devient mon IP publique (ex : `82.66.10.3`)
3. La box **note dans une table** que cet échange concerne `192.168.1.42`
4. Le paquet part sur internet avec l'IP publique
5. Google répond à l'IP publique
6. La box consulte sa table, retrouve que c'était pour `192.168.1.42`, réécrit la destination, livre le paquet

#### PAT : la précision technique

Ce que j'ai décrit est en réalité du **PAT** (*Port Address Translation*) ou NAT overload. La box utilise les IP **et** les numéros de port pour distinguer les connexions.

Table de la box (simplifiée) :

| IP privée + port | IP publique + port |
|------------------|--------------------|
| 192.168.1.42:50123 | 82.66.10.3:50123 |
| 192.168.1.55:50123 | 82.66.10.3:50124 |

Même port utilisé en interne par deux appareils différents : la box attribue des ports externes distincts pour les démêler.

#### Pourquoi NAT existe

* **Pénurie IPv4** : pas assez d'adresses pour un appareil = une IP publique. NAT compresse.
* **Sécurité par effet de bord** : depuis l'extérieur, impossible de joindre directement une IP privée. Protection naturelle, mais pas une vraie sécurité en soi.
* **Flexibilité** : je change d'opérateur, mon réseau interne reste identique.

---

## Synthèse : le scénario complet

Je tape `google.com` dans mon navigateur. Voilà exactement ce qui se passe :

1. Mon ordi a une **IP privée** obtenue par **DHCP** (échange DORA) à la connexion au réseau
2. Le navigateur doit résoudre `google.com` : requête **DNS**, réponse `142.250.74.110`
3. Cette IP n'est pas locale : mon ordi envoie le paquet à sa **passerelle** (ma box)
4. Pour envoyer le paquet à la box, il lui faut sa **MAC** : il consulte son **ARP cache**, ou envoie une **ARP request** si besoin
5. Le paquet quitte mon ordi via la **NIC** (couches 1 et 2 du modèle **OSI**) : IP source = mon IP privée, IP destination = Google, MAC source = ma MAC, MAC destination = MAC de la box
6. La box applique le **NAT** : remplace mon IP privée par son IP publique, note la correspondance
7. De **routeur en routeur** (routing), le paquet arrive chez Google
8. Google répond, le paquet remonte, la box inverse le NAT, mon ordi reçoit la réponse
9. La page s'affiche

Tout ça en quelques dizaines de millisecondes.

---

## Questions de vérification

Si je peux répondre sans relire, c'est intégré :

1. Différence entre MAC et IP ?
2. Les 4 étapes du DORA ?
3. À quelle couche OSI vit l'IP ? Et la MAC ?
4. Quel protocole trouve la MAC correspondant à une IP sur le réseau local ?
5. Pourquoi NAT existe ?
6. Différence entre ARP et DNS ?

---

## Commandes à connaître par cœur

```bash
ip a                        # Voir mes interfaces et IP (Linux)
ipconfig                    # Idem Windows
arp -a                      # Voir mon ARP cache
ip neigh                    # Idem Linux (plus précis)
ping 8.8.8.8                # Tester la connectivité
traceroute google.com       # Voir les sauts jusqu'à une destination
nslookup google.com         # Résolution DNS
dig google.com              # Résolution DNS détaillé (Linux)
```

Manipuler ces commandes sur ma machine, pas juste les lire. C'est en voyant les vraies sorties que ça s'ancre.
