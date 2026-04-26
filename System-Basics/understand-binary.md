# 💾 Comprendre le Binaire

**Formation** : Jedha Cybersécurité — Essentials  
**Module** : How Computers Work  
**Statut** : ✅ Complété

---

## 1. Computers Are Just 1s and 0s

Tout ce qu'un ordinateur fait afficher une image, jouer de la musique, envoyer un email, exécuter un malware, e résume à une seule réalité physique : de l'électricité qui passe ou qui ne passe pas.

Un transistor est un minuscule composant électronique qui agit comme un interrupteur :

* **Courant qui passe** → état `1` (HIGH)
* **Courant qui ne passe pas** → état `0` (LOW)

C'est tout. Il n'y a pas de troisième état. Pas de "peut-être". Juste 1 ou 0.

Un CPU moderne contient des **milliards** de ces transistors. En les combinant et en les faisant commuter des milliards de fois par seconde, un ordinateur peut représenter et traiter n'importe quelle information : texte, image, son, vidéo, code.

### Le bit et l'octet

La plus petite unité d'information est le **bit** (binary digit), un seul 1 ou 0.

En pratique, les bits sont regroupés en **octets** (bytes) de 8 bits :

```
Un bit     :  1
Un octet   :  1 0 1 1 0 1 0 0
```

Avec 8 bits, on peut représenter 2⁸ = **256 combinaisons différentes** (de 0 à 255).

### Pourquoi le binaire et pas autre chose ?

On aurait pu concevoir des ordinateurs avec 3 états (ternaire) ou 10 états (décimal). En pratique, 2 états c'est ce qu'il y a de plus simple et de plus fiable à implémenter physiquement. Un transistor fiable à 2 états est trivial à fabriquer. Un composant fiable à 10 états distincts est exponentiellement plus complexe et plus sujet aux erreurs.

Le binaire n'est pas un choix arbitraire, c'est la solution d'ingénierie la plus robuste possible.

---

## 2. Binary Translations

Si les ordinateurs ne parlent que 1 et 0, comment font-ils pour afficher du texte, des couleurs, ou jouer de la musique ? La réponse : **des tables de correspondance** (encodages) que tout le monde s'est mis d'accord pour respecter.

### Texte — ASCII et Unicode

**ASCII** (American Standard Code for Information Interchange) est la première table de correspondance standardisée entre nombres binaires et caractères.

Chaque caractère est associé à un nombre décimal, lui-même représenté en binaire :

| Caractère | Décimal | Binaire |
|---|---|---|
| A | 65 | 01000001 |
| B | 66 | 01000010 |
| a | 97 | 01100001 |
| 0 | 48 | 00110000 |
| espace | 32 | 00100000 |
| ! | 33 | 00100001 |

Quand tu tapes `A` sur ton clavier, le système envoie `01000001` au CPU. Quand le CPU veut afficher `A` à l'écran, il envoie `01000001` à la carte graphique qui sait que ça correspond au glyphe "A".

**Unicode** est l'extension moderne d'ASCII qui couvre plus de 140 000 caractères : alphabets du monde entier, emojis, symboles mathématiques. L'encodage le plus utilisé est **UTF-8**, rétrocompatible avec ASCII.

### Couleurs — RGB

Les couleurs sur un écran sont représentées par trois canaux : **Rouge (R), Vert (G), Bleu (B)**.

Chaque canal prend une valeur de 0 à 255 (un octet = 8 bits).

| Couleur | R | G | B | Représentation binaire |
|---|---|---|---|---|
| Noir | 0 | 0 | 0 | 00000000 00000000 00000000 |
| Blanc | 255 | 255 | 255 | 11111111 11111111 11111111 |
| Rouge pur | 255 | 0 | 0 | 11111111 00000000 00000000 |
| Bleu pur | 0 | 0 | 255 | 00000000 00000000 11111111 |

Une image HD de 1920×1080 pixels contient 2 073 600 pixels. Chaque pixel = 3 octets RGB = environ **6 Mo** de données binaires brutes avant compression.

### Son — Échantillonnage

Le son est une onde continue. Pour le stocker en binaire, on l'**échantillonne** : on mesure l'amplitude de l'onde à intervalles réguliers et on encode chaque mesure en binaire.

* **Fréquence d'échantillonnage** : nombre de mesures par seconde (44 100 Hz pour un CD)
* **Profondeur de bits** : précision de chaque mesure (16 bits pour un CD = 65 536 valeurs possibles)

Un fichier audio CD de 1 minute : 44 100 × 16 bits × 2 canaux (stéréo) × 60 secondes = environ **10 Mo** non compressé.

---

## 3. Use Case : Create a File

Pour rendre concret le lien entre binaire et fichiers, créons un fichier texte et observons ce qui se passe réellement.

### Ce qui se passe quand tu crées un fichier texte

Imagine que tu ouvres un éditeur de texte et que tu écris :

```
Hi
```

En apparence, tu as écrit deux caractères. En réalité, voici ce que l'OS stocke sur le disque :

| Caractère | Décimal (ASCII) | Binaire |
|---|---|---|
| H | 72 | 01001000 |
| i | 105 | 01101001 |
| \n (retour ligne) | 10 | 00001010 |

Le fichier sur le disque contient exactement ces 3 octets (ou 2 si pas de retour ligne) : `01001000 01101001 00001010`.

### Observation pratique sur Linux

```bash
# Créer un fichier
echo "Hi" > hello.txt

# Voir la taille en octets
ls -l hello.txt

# Voir le contenu en hexadécimal (représentation lisible du binaire)
xxd hello.txt
```

Sortie de `xxd hello.txt` :
```
00000000: 4869 0a                                  Hi.
```

`48` = H en hexadécimal (72 en décimal)  
`69` = i en hexadécimal (105 en décimal)  
`0a` = \n en hexadécimal (10 en décimal)

### Pourquoi c'est important en cybersécurité

Tout fichier est une séquence d'octets. Un malware est une séquence d'octets. Un fichier PDF infecté est un PDF normal avec une séquence d'octets malveillants ajoutée. Les antivirus fonctionnent en cherchant des **signatures** (des séquences d'octets caractéristiques) dans les fichiers. C'est pour ça qu'on peut analyser un fichier sans l'exécuter : on lit directement ses octets.

---

## 4. Side Note on Coding

Un point souvent mal compris en début de parcours : **le code que tu écris n'est pas ce que le CPU exécute**.

### La chaîne de transformation

```
Code source (Python, C, Java...)
        ↓
Compilateur / Interpréteur
        ↓
Langage machine (instructions binaires)
        ↓
CPU qui exécute les 1 et les 0
```

### Langages compilés vs interprétés

**Langages compilés** (C, C++, Rust) :  
Le code source est transformé **une fois** en langage machine avant l'exécution. Le résultat est un fichier binaire exécutable directement par le CPU. Plus rapide à l'exécution.

```
code.c  →  compilateur  →  programme.exe (binaire)  →  CPU
```

**Langages interprétés** (Python, JavaScript, Ruby) :  
Le code source est traduit **à la volée** pendant l'exécution par un interpréteur. Plus flexible, plus lent.

```
script.py  →  interpréteur Python  →  CPU
```

**Langages à bytecode** (Java, C#) :  
Compilés vers un format intermédiaire (bytecode), ensuite interprété par une machine virtuelle (JVM, .NET CLR).

```
code.java  →  compilateur  →  bytecode  →  JVM  →  CPU
```

### Ce que ça signifie pour la sécurité

* Un **reverse engineer** prend un exécutable binaire et remonte vers un code lisible,c'est ce que fait l'analyse de malwares
* Un **décompilateur** comme Ghidra ou IDA Pro lit les bytes d'un programme et reconstruit une représentation du code original
* Les **CVE de type buffer overflow** exploitent directement la façon dont le code machine gère la mémoire, compréhension du binaire indispensable

---

## 5. Binary Translation : Base 10 to Base 2

### Comprendre les bases numériques

Le système décimal (base 10) qu'on utilise au quotidien n'a rien de naturel, c'est une convention liée au fait qu'on a 10 doigts. Un système en base N utilise N symboles distincts et fonctionne par puissances de N.

**Base 10 (décimal)** : symboles 0–9, puissances de 10

Prenons le nombre **347** :

```
347 = 3 × 10² + 4 × 10¹ + 7 × 10⁰
    = 3 × 100 + 4 × 10 + 7 × 1
    = 300 + 40 + 7
```

**Base 2 (binaire)** : symboles 0–1, puissances de 2

Chaque position représente une puissance de 2 :

| Position | 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 |
|---|---|---|---|---|---|---|---|---|
| Valeur | 128 | 64 | 32 | 16 | 8 | 4 | 2 | 1 |

Prenons `10110100` en binaire :

```
10110100 = 1×128 + 0×64 + 1×32 + 1×16 + 0×8 + 1×4 + 0×2 + 0×1
         = 128 + 0 + 32 + 16 + 0 + 4 + 0 + 0
         = 180
```

### Table de correspondance rapide

| Décimal | Binaire | Décimal | Binaire |
|---|---|---|---|
| 0 | 00000000 | 8 | 00001000 |
| 1 | 00000001 | 9 | 00001001 |
| 2 | 00000010 | 10 | 00001010 |
| 3 | 00000011 | 16 | 00010000 |
| 4 | 00000100 | 32 | 00100000 |
| 5 | 00000101 | 64 | 01000000 |
| 6 | 00000110 | 128 | 10000000 |
| 7 | 00000111 | 255 | 11111111 |

### Note sur l'hexadécimal (Base 16)

En pratique, le binaire brut est difficile à lire pour un humain. On utilise souvent l'**hexadécimal** (base 16) comme raccourci : 4 bits = 1 chiffre hexadécimal.

| Binaire | Hex | Décimal |
|---|---|---|
| 0000 | 0 | 0 |
| 1010 | A | 10 |
| 1111 | F | 15 |
| 11111111 | FF | 255 |

Un octet s'écrit donc en 2 chiffres hexadécimaux : `FF` = `11111111` = 255. C'est pour ça que les adresses mémoire, les hashes, les adresses MAC et les couleurs web s'écrivent en hex.

---

## 6. How to Convert to Base 2

### Méthode des divisions successives

L'algorithme pour convertir un nombre décimal en binaire :

1. Diviser le nombre par 2
2. Noter le reste (0 ou 1)
3. Prendre le quotient et recommencer
4. Continuer jusqu'à ce que le quotient soit 0
5. Lire les restes **de bas en haut**

### Exemple : convertir 43 en binaire

```
43 ÷ 2 = 21  reste 1  ← bit de poids faible (LSB)
21 ÷ 2 = 10  reste 1
10 ÷ 2 = 5   reste 0
 5 ÷ 2 = 2   reste 1
 2 ÷ 2 = 1   reste 0
 1 ÷ 2 = 0   reste 1  ← bit de poids fort (MSB)
```

On lit les restes de bas en haut : **101011**

Vérification : 1×32 + 0×16 + 1×8 + 0×4 + 1×2 + 1×1 = 32 + 8 + 2 + 1 = **43** ✅

### Exemple : convertir 200 en binaire

```
200 ÷ 2 = 100  reste 0
100 ÷ 2 = 50   reste 0
 50 ÷ 2 = 25   reste 0
 25 ÷ 2 = 12   reste 1
 12 ÷ 2 = 6    reste 0
  6 ÷ 2 = 3    reste 0
  3 ÷ 2 = 1    reste 1
  1 ÷ 2 = 0    reste 1
```

Résultat : **11001000**

Vérification : 128 + 64 + 0 + 0 + 8 + 0 + 0 + 0 = **200** ✅

### Méthode alternative : soustraction par puissances de 2

Plus rapide mentalement une fois qu'on connaît les puissances de 2.

Puissances de 2 à connaître par cœur :

```
2⁰ = 1    2¹ = 2    2² = 4    2³ = 8
2⁴ = 16   2⁵ = 32   2⁶ = 64   2⁷ = 128
```

Pour convertir **107** :

```
107 ≥ 64 ? Oui  → bit = 1, reste 107 - 64 = 43
 43 ≥ 32 ? Oui  → bit = 1, reste 43 - 32 = 11
 11 ≥ 16 ? Non  → bit = 0
 11 ≥ 8  ? Oui  → bit = 1, reste 11 - 8 = 3
  3 ≥ 4  ? Non  → bit = 0
  3 ≥ 2  ? Oui  → bit = 1, reste 3 - 2 = 1
  1 ≥ 1  ? Oui  → bit = 1, reste 0
```

Résultat : **1101011**

Vérification : 64 + 32 + 8 + 2 + 1 = **107** ✅

### Exercices d'entraînement

| Décimal | Binaire (à calculer) | Réponse |
|---|---|---|
| 12 | ? | 00001100 |
| 57 | ? | 00111001 |
| 128 | ? | 10000000 |
| 255 | ? | 11111111 |
| 192 | ? | 11000000 |

> **Astuce** : 255 = `11111111` (8 bits tous à 1) et 0 = `00000000` (8 bits tous à 0). Ce sont les valeurs minimale et maximale d'un octet. Elles reviennent constamment en réseau (masques de sous-réseau, adresses IP, valeurs RGB).

---

## 🛡️ Applications en cybersécurité

| Concept | Usage sécurité |
|---|---|
| Binaire / octets | Analyse de malwares, lecture de fichiers bruts, forensics |
| ASCII / Unicode | Injection de caractères, encodage d'exploits, obfuscation |
| Hexadécimal | Shellcodes, hashes, adresses mémoire, signatures AV |
| Conversion base 2 | Calcul de masques réseau, subnetting, flags TCP/IP |
| RGB / encodages | Stéganographie — cacher des données dans des images |
| Compilé vs interprété | Reverse engineering, analyse statique de binaires |

---

## 📋 Références

* [ASCII Table](https://www.asciitable.com)
* [Unicode Character Table](https://unicode-table.com)
* [Ghidra — Reverse Engineering Tool (NSA)](https://ghidra-sre.org)
* [CyberChef — Convertisseur universel en ligne](https://gchq.github.io/CyberChef)
