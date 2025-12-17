# Procédure de destruction de données sur support électronique

Par Adrien Ferron / lacapsule.org / Fédération régionale des reconditionneurs bretons OGO 2024

[TOC]

## Préambule

La plupart (tous?) Les SSD sont chiffrés par conception. Ce chiffrement n'est pas destiné à sauvegarder vos données au sens traditionnel du terme. Le seul but du chiffrement est de permettre un effacement de toutes les données en supprimant simplement la clé de chiffrement et en laissant derrière elle les données chiffrées. C'est pour cette raison que l'effacement est si rapide.

## Installation d'utilitaires

### Identifier le type du support

Il est avant tout nécessaire de connaître le type du support afin de choisir le bon utilitaire à utiliser pour le traiter.
Un bon utilitaire à utiliser pour se faire de par sa simplicité, sa rapidité et son efficacité, est `lsscsi` 

Pour l'installer :

```bash
sudo apt update
sudo apt upgrade
sudo apt install lsscsi
```

Une fois installé, nous pouvons tester sa bonne installation en tapant la commande suivante :

```bash
sudo lsscsi
```

Le retour de cette commande doit ressembler à peu près au suivant :

```bash
[0:0:0:0]    disk    ATA      ST1000LM024 HN-M 0001  		     /dev/sda 
[N:0:2:1]    disk    SAMSUNG MZVLW256HEHP-000L7__1               /dev/nvme0n1
```

Si ce n'est pas le cas, la commande a été mal orthographiée, le paquet n'est tout simplement pas présent ou encore aucun disque n'est détecté.

### Utilitaire pour interface SATA

Afin de manipuler les disques ATA/PATA/SATA, il sera nécessaire d'utiliser l'utilitaire [hdparm](https://doc.ubuntu-fr.org/hdparm)

Installer hdparm :

```bash
sudo apt update
sudo apt upgrade
sudo apt install hdparm
```

Vérifier la présence du paquet :

```bash
sudo hdparm -h
```

Le retour de cette commande doit être un ensemble d'options définies ligne par ligne ainsi qu'un exemple d'utilisation.
Si ce n'est pas le cas, la commande a été mal orthographiée ou le paquet n'est tout simplement pas présent.

### Utilitaire pour interface NVME

Afin de manipuler les disques NVME, il sera nécessaire d'utiliser l'utilitaire [nvme-cli](https://manpages.ubuntu.com/manpages/noble/man1/nvme.1.html)

Installer nvme-cli :

```bash
sudo apt update
sudo apt upgrade
sudo apt install nvme-cli
```

Vérifier la présence du paquet :

```bash
sudo nvme -h
```

Le retour de cette commande doit être un ensemble d'options définies ligne par ligne ainsi qu'un exemple d'utilisation.
Si ce n'est pas le cas, la commande a été mal orthographiée ou le paquet n'est tout simplement pas présent.

## Prise d'informations

### Obtenir les informations S.M.A.R.T.

Il sera nécessaire d'installer le paquet `smartmontools`

```bash
sudo apt update
sudo apt upgrade
sudo apt install smartmontools
```

Par la suite, il est possible d'obtenir l'état S.M.A.R.T. du support par la commande :

```bash
sudo smartctl -a /dev/xxx
```

Ce qui doit donner un résultat à peu près semblable au suivant :

```bash
=== START OF INFORMATION SECTION ===
Model Number:                       SAMSUNG MZVLW256HEHP-000L7
Serial Number:                      S35ENX0K349187
Firmware Version:                   4L7QCXB7
PCI Vendor/Subsystem ID:            0x144d
IEEE OUI Identifier:                0x002538
Total NVM Capacity:                 256 060 514 304 [256 GB]
Unallocated NVM Capacity:           0
Controller ID:                      2
NVMe Version:                       1.2
Number of Namespaces:               1
Namespace 1 Size/Capacity:          256 060 514 304 [256 GB]
Namespace 1 Utilization:            63 449 567 232 [63,4 GB]
Namespace 1 Formatted LBA Size:     512
```

### Obtenir les informations du support SATA

Sur disque SATA, la commande suivante retourne l'ensemble des informations du support :

```bash
sudo hdparm -I /dev/sdx
```

Exemple de retour :

```bash
Security: 
	Master password revision code = 65534
		supported
	not	enabled
	not	locked
	not	frozen
	not	expired: security count
		supported: enhanced erase
	4min for SECURITY ERASE UNIT. 8min for ENHANCED SECURITY ERASE UNIT.
```

### Obtenir les informations du support NVME

Sur disque NVME, la commande suivante retourne l'ensemble des informations du support :

```bash
sudo nvme id-ctrl /dev/nvme0nX
```

Exemple de retour :

```bash
NVME Identify Controller:
vid       : 0x144d
ssvid     : 0x144d
sn        : S35ENX0K349187      
mn        : SAMSUNG MZVLW256HEHP-000L7              
fr        : 4L7QCXB7
rab       : 2
ieee      : 002538
...
```

## Débloquer le support

### Obtenir les codes constructeurs

Le site [beta.bios-pw.org](https://beta.bios-pw.org/) est conçu pour aider à effacer les mots de passe inconnus des BIOS. 
Cela peut éviter d'avoir recours à un flashage intégral du microprogramme interne.

Lorsqu'un mot de passe bloque l'accès au support, un code est délivré dans le message d'erreur. Il sera alors nécessaire de reporter ce code dans le champs d'entrée présent sur le site  [beta.bios-pw.org](https://beta.bios-pw.org/).

Ceci fait, vous obtenez le code de déblocage à reporter dans le champs d'entrée de la machine bloquée.

NB: le clavier est en QWERTY et il sera préférable de valider la frappe avec ctrl+enter.

### Extraction de la pile setup

Dans certains cas, il sera nécessaire de débrancher la pile setup afin de vider la ROM de sa configuation et ainsi réinitialiser le mot de passe protégeant l'accès au BIOS.

### Effectuer un court-circuit

Dans d'autres cas, l'extraction de la pile ne suffit pas car la configuration est stocké dans une puce EEPROM. Il s'agit d'une **mémoire non volatile utilisée dans les dispositifs qui doivent contenir de petites quantités de données dans un circuit**.

Il sera donc nécessaire de réaliser un court-circuit entre deux pattes de la puce. Ceci peut être réalisé à l'aide d'un outil métallique fin, cependant, cette technique est à haut risque car en cas de dérapage, il est possible de 'briquer' le matériel. 

Cette technique nécessite donc certaines compétences en électronique et en programmation car en cas d'échec, il sera obligatoire de changer certains micro-composants, parfois même en effectuant une reprogrammation bas niveau.

### Reprogrammation bas niveau

Pour effectuer une reprogrammation bas niveau ou flash de la puce, il sera nécessaire d'utiliser un [CH341A](https://github.com/YTEC-info/CH341A-Softwares) en [direct sur la puce](https://www.youtube.com/watch?v=OzUh0jlOe-0) ou en [dessoudant celle-ci](https://www.youtube.com/watch?v=h0G4xFK-oAU) afin de [la positionner sur le socket du CH341A](https://www.youtube.com/watch?v=-no4N_B67QU).   

[Assurez-vous de bien choisir votre CH341A !](https://www.youtube.com/watch?v=MMyDvb_v4uc)

Il sera nécessaire :

- d'[installer les drivers](https://www.youtube.com/watch?v=5NYe21nFSDI)
- de sauvegarder la configuration actuelle
- de trouver la bonne version du BIOS
- de remplir la puce d'octets vides
- d'injecter la bonne version du BIOS dans la puce

### Sortir le support du mode frozen

Si le disque est frozen, vous devriez obtenir la sortie suivant via la commande `sudo hdparm -I /dev/sdx`

```bash
Security: 
	Master password revision code = 65534
		supported
	not	enabled
	not	locked
		frozen
	not	expired: security count
		supported: enhanced erase
	4min for SECURITY ERASE UNIT. 8min for ENHANCED SECURITY ERASE UNIT.
```

il faudra utiliser les commandes suivantes :

```bash
sudo chown $USER:$USER /sys/power/state
```

Cette commande permettra d'obtenir les droit sur `state` 

Il sera ensuite nécessaire de forcer la mise en veille pour sortir le support du mode`frozen` :

```bash
sudo echo -n mem > /sys/power/state
```

afin de sortir le système de veille, il sera souvent nécessaire d'appuyer sur le bouton d'alimentation de la machine. 

Si vous avez accès au disque et qu'il n'est pas gelé, nous pouvons alors lancer le protocole suivant :

## Protocole secure erase et sanitize sur SATA

### Effacement

```bash
sudo hdparm --user-master u --security-set-pass PASS /dev/sdX (ajouter un mot de passe)
```

```bash
sudo hdparm --user-master u --security-erase PASS /dev/sdX (effacer le mot de passe)
```

```bash
sudo hdparm --user-master u --security-erase-enhanced p /dev/sdX (si supporté par le disque)
```

```bash
sudo hdparm --user-master m --security-disable PASS /dev/sdX (désactiver le mot de passe)
```

Pour un SSD SATA, on utilise souvent hdparm avec une option de sanitize spécifique selon la version :

```bash
sudo hdparm --yes-i-know-what-i-am-doing --sanitize-block-erase /dev/sdX
```

ou pour un effacement cryptographique :

```bash
sudo hdparm --yes-i-know-what-i-am-doing --sanitize-crypto-scramble /dev/sdX
```

### Vérification

```bash
sudo dd if=/dev/sda bs=1M count=5 (lecture des premiers bits)
```

Nous pourrons valider l'effacement si les résultat de cette commande donnent tous 5+0 de cette manière :

```bash
5+0 records in
5+0 records out
5242880 bytes (5.2 MB, 5.0 MiB) copied, 0.000025 s, 209 MB/s
```

## Protocole secure erase et sanitize sur NVME

### Effacement

```bash
sudo nvme format /dev/nvmeXnY --ses=2 (Si le disque prend en charge crypto-erase)
```

```bash
sudo nvme format /dev/nvmeXnY --ses=1 (Si le disque ne prend pas en charge crypto-erase)
```

Pour un SSD NVMe, la commande Linux classique est via l’outil nvme-cli avec la commande :

```bash
sudo nvme sanitize /dev/nvmeXnY
```

Cette commande lance l’opération de sanitize qui efface toutes les données de manière sécurisée en utilisant la méthode prise en charge par le disque (block erase, crypto erase, etc.).

Vérification

```bash
sudo nvme list
```

Cette commande doit retourner une liste n'affichant pas le support effacé. L'absence du support dans la liste valide son effacement.

## Protocole TRIM sur SATA

### Vérification de TRIM

Il est impératif de vérifier que le SSD SATA supporte TRIM avant d'appliquer ce protocole.

```bash
sudo hdparm -I /dev/sdX | grep -i trim
```

Le retour doit indiquer :

```bash
Data Set Management TRIM supported (limit 1 block)
```

et idéalement :

```bash
Deterministic read data after TRIM
```

Si ces lignes sont absentes, ce protocole ne s'applique pas.

### Activation du support TRIM

Installer fstrim si nécessaire (Adapter à la branche Linux, ici Debian) :

```bash
sudo apt update
sudo apt install util-linux
```

Vérifier la présence :

```bash
fstrim -V
```

### Protocole TRIM (SSD déjà utilisé)

Ce protocole s'applique lorsque les commandes Security/Sanitize sont indisponibles.

Démonter toutes les partitions du SSD

Recréer une table de partitions couvrant tout le disque (GPT + 1 partition de données)

Créer un système de fichiers TRIM-compatible :

```bash
sudo mkfs.ext4 -E discard /dev/sdX1
```

Monter la partition :

```bash
sudo mkdir /mnt/ssd
sudo mount /dev/sdX1 /mnt/ssd
```

Lancer un TRIM global :

```bash
sudo fstrim -av
```

La sortie doit afficher les octets trimmed pour le SSD.

Démonter :

```bash
sudo umount /mnt/ssd
```

**Variante stricte (remplissage + TRIM) :**

```bash
sudo dd if=/dev/zero of=/mnt/ssd/fill.tmp bs=1M status=progress
rm /mnt/ssd/fill.tmp
sudo fstrim -av
```

**Protocole TRIM (SSD propre/affecté)**

Vérifier TRIM via 

```bash
hdparm -I /dev/sdX
```

Créer table de partitions + système de fichiers

```bash
sudo mkfs.ext4 -E discard /dev/sdX1
sudo mount /dev/sdX1 /mnt/tmp
```

Monter et lancer immédiatement :

```bash
sudo fstrim -av
```

Contrôler l'absence de données lisibles :

```bash
sudo dd if=/dev/sdX bs=1M count=5 | hexdump -C | head
```

**Les données doivent apparaître aléatoires/non interprétables.**

En cas d'échec de fstrim, utiliser l'alternative :

```bash
sudo blkdiscard /dev/sdX
```

(ATTENTION : destructif, disque non monté)

## HPA et DCO sur SATA

### Vérification de la HPA

Vérifions s'il y a une HPA (zone protégée par l'hôte).
Il s'agit d'une zone protégée qui ne sera pas effacée si nous écrasons l'ensemble du disque.

```bash
sudo hdparm -N /dev/sdX
```

Nous verrons quelque chose comme ceci si la HPA est désactivée:

```bash
/dev/sdX:
max sectors = 1565152896/1565152896, HPA is disabled
```

Sur le côté droit, nous avons la limite réelle du secteur matériel du disque, sur le côté gauche, nous voyons la valeur fixée pour l'HPA. Ici, les nombres sont les mêmes qui indiquent que HPA est désactivé.

Que faisons-nous donc si les chiffres ne correspondent pas ?

Nous modifions la valeur par rapport au nombre maximal réel de secteurs.

```bash
sudo hdparm –N 1565152896 /dev/sdx
```

Il est à noter que cela n'est pas permanent et qu'il sera restauré après le démarrage. 
Utilisation :

```bash
sudo hdparm –N p1565152896 
```

si vous voulez rendre cela permanent.

### Vérification du DCO

Pour voir le DCO, utilisez la commande HDPARM suivante.

```bash
sudo hdparm --dco-identify /dev/sdX
```

Le fabriquant utilise DCO pour définir les modes de transfert de données admissibles (MDMA, UDMA), la taille réelle du lecteur (secteurs max), et les commandes ATA/SATA qui peuvent être désactivées.

Si vous voulez essayer de remettre le DCO par défaut, vous pouvez utiliser la commande HDPARM suivante:

```bash
sudo hdparm --dco-restore /dev/sdX
```

Selon les instructions, vous ajoutez le commutateur suivant "J'accepte les conséquences":

```bash
sudo hdparm --yes-i-know-what-i-am-doing --dco-restore /dev/sdX
```

## Exceptions

### L'exception sb[ ] 

En cas d'erreur sb[]array, il convient de bien identifier le périphérique et de télécharger l'outil suivant :

```bash
sudo apt update
sudo apt upgrade
sudo apt install sg3-utils
```

Nous pourrons ensuite convertir le code héxa en anglais afin de mieux comprendre l'erreur et trouver de l'aide plus facilement :

```bash
sudo sg_decode_sense [le code hexa]
```

Ceci rendra le code erreur compréhensible.

### L'exception secure-erase indisponible

Sur certains modèles de SSD, le bloc "Security" est indisponible. Plusieurs firmware de SSD d'entrée de gamme n'exposent pas les commandes ATA Security de façon standard.
Dans ces cas, les commandes `--security-erase` de hdparm ne sont pas utilisables, même si le disque fonctionne normalement pour le stockage.

Sur SSD, si le contrôleur supporte TRIM et un mode de lecture  déterministe après TRIM (DRAT/DZAT), un effacement logique via TRIM  suffit généralement à rendre les anciennes données inaccessibles.
 Il est possible de vérifier la prise en charge de TRIM sur `/dev/sdx` avec une commande de type 

```bash
hdparm -I /dev/sdx | grep -i trim
```

Si TRIM est listé et que le contrôleur indique DRAT ou DZAT, un TRIM global (par exemple via formatage ou outil dédié) est considéré comme suffisant pour l’effacement et il suffit de suivre la procédure Protocole TRIM sur SATA disponible plus haut dans le document. 

### L'exception sanitize refusé

Le message « SANITIZE feature set is not supported » indique que le  firmware ne gère pas le jeu de commandes ATA Sanitize, même si certains SSD récents le proposent.

exemple : 

```bash
sudo hdparm --yes-i-know-what-i-am-doing --sanitize-block-erase /dev/sda
------------------------------------------------------------------------
/dev/sda:
SANITIZE feature set is not supported
```

Vérifier les capacités Security et TRIM 

```bash
sudo hdparm -I /dev/sdX
```

Le retour doit confirmer la présence du bloc `Security:` avec `supported`, `not enabled`, `not locked`, `not frozen`, `supported: enhanced erase`et la présence de `Data Set Management TRIM supported`

Vérifier que le disque n’est pas en mode frozen (sinon appliquer la  procédure « Sortir le support du mode frozen » déjà décrite plus haut dans le document).
Utiliser le protocole Security ERASE (classique) à la place de Sanitize.

Une fois l’opération terminée, vérifier que le bloc Security indique à nouveau `not enabled` et `not locked` dans `hdparm -I /dev/sdX`

Compléter par un effacement logique global via TRIM, afin de s’assurer que toutes les LBA utilisateur sont désallouées par le contrôleur, dans la mesure où TRIM est supporté.

### L'exception « sanitize supporté mais erreur en cours d’exécution »

Ce comportement est typique des SSD d’entrée de gamme (firmware limité mais Sanitize annoncé).

Cela se produit lorsque `hdparm -I /dev/sdX` annonce `*SANITIZE feature set` et `* BLOCK_ERASE_EXT command`, mais la commande `--sanitize-block-erase` démarre l’opération en arrière-plan (« Operation started in background ») puis rend le disque illisible (`dd: error reading '/dev/sda': Input/output error`).

**Cette exception s’applique lorsque :**

- Le SSD supporte Sanitize et indique des durées pour SECURITY ERASE/ENHANCED (ici 2min chacun).
- La commande sanitize débute mais bloque l’accès au disque pendant son exécution.
- `hdparm -I` retourne des erreurs SG_IO après lancement.

Vérifier le status sanitize en cours :

```bash
sudo hdparm --yes-i-know-what-i-am-doing --sanitize-status /dev/sdX
```

- Si « Operation in progress » ou pourcentage : **attendre la fin** (ici ~2min max selon firmware).
- Si « No operation in progress » mais disque bloqué : passer à l’étape suivante.

Si bloqué >10min, forcer interruption via veille :

```bash
sudo chown $USER:$USER /sys/power/state
sudo echo -n mem > /sys/power/state
```

(Réveiller via bouton power, puis re-vérifier `hdparm -I /dev/sdX`).

**Compléter par un Secure Erase si sanitize échoue**

Si sanitize ne se termine pas ou rend le disque inutilisable :

Vérifier que Security n’est pas frozen :

```bash
sudo hdparm -I /dev/sdX | grep -i frozen
```

(Si frozen, appliquer procédure « Sortir le support du mode frozen »).

Après Security Erase, vérifier accès et état :

```bash
sudo hdparm -I /dev/sdX
sudo dd if=/dev/sdX bs=1M count=5 | hexdump -C | head
```

`dd` doit réussir (données aléatoires/non-intelligibles).

Security doit être `not enabled/not locked/not frozen`.

Si accès toujours impossible :

- Redémarrer le système.
- Vérifier détection via `lsscsi` ou `lsblk`.
- Si disque « mort » : appliquer protocole TRIM ou écriture complète manuelle.

**Protocole de fallback (TRIM + remplissage)**

Si les deux méthodes (sanitize + security) échouent, créer table partitions + ext4 :

```bash
sudo mkfs.ext4 -E discard /dev/sdX1
sudo mount /dev/sdX1 /mnt/tmp
```

Remplir + TRIM :

```bash
sudo dd if=/dev/zero of=/mnt/tmp/fill.tmp bs=1G status=progress
sudo rm /mnt/tmp/fill.tmp
sudo fstrim -av /mnt/tmp
sudo umount /mnt/tmp
```

**Remarques importantes**

- **Toujours attendre** le sanitize background (ici TS240GSSD220S peut prendre >2min malgré annonce).
- **Ne jamais interrompre un sanitize en cours** sauf timeout >15min.
- Consigner : modèle exact (ex : TS240GSSD220S), firmware (ex : R0510A0), et méthode utilisée (ex : sanitize partieller + security fallback).

### L'exception « NVMe sans Sanitize/Format »

Certains modèles OEM sont connus pour refuser sanitize/format malgré NVMe 1.2, mais acceptent TRIM/Write Zeroes + chiffrement interne.

Ce cas concerne les SSD NVMe dont `sanicap = 0` (aucun type de sanitize supporté) et où `nvme format --ses=1/2` retourne `Access Denied (0x4286)`. Le SSD reste détecté mais refuse les commandes d'effacement standard.

Cette exception s'applique lorsque :

- `sudo nvme id-ctrl -H /dev/nvmeXnY` affiche `sanicap : 0` (Crypto/Block/Overwrite non supportés).
- `sudo nvme format --ses=1/2` échoue avec `Access Denied: Access to the namespace and/or LBA range is denied due to lack of access rights(0x4286)`.
- `sudo nvme sanitize /dev/nvmeXnY` retourne `Invalid Sanitize Action`.

**Vérifier les capacités NVMe**

Confirmer l'absence de sanitize et les capacités supportées :

```bash
sudo nvme id-ctrl -H /dev/nvme0n1 | grep -E "(sanicap|oacs|oncs)"
```

Résultat attendu : `sanicap : 0`, `oacs : 0x17` (Format NVM et Security Send/Receive supportés mais bloqués).

Vérifier TRIM/Write Zeroes (souvent les seuls disponibles) :

```bash
sudo nvme id-ctrl -H /dev/nvme0n1 | grep oncs
```

Chercher `Write Zeroes Supported` et `Data Set Management Supported`.

**Protocole TRIM + Write Zeroes (méthode principale)**

Puisque sanitize/format sont refusés, utiliser les commandes NVMe de base supportées :

Démonter toutes les partitions NVMe et identifier le namespace :

```bash
sudo umount /dev/nvme0n1p*
sudo nvme list
```

Lancer un effacement par zéros NVMe (équivalent write zeroes sur tout le namespace) :

```bash
sudo nvme write-zeroes /dev/nvme0n1 --force -s 0 -l 100
```

(Remplit toutes les LBA avec des zéros, force l'usage de toutes les zones utilisateur).

TRIM global sur le namespace non monté :

```bash
sudo nvme dsm-internal-log /dev/nvme0n1  # Vérifier DSM support
sudo nvme dataset-manage -t 1 /dev/nvme0n1  # TRIM toutes les zones libres
```

**Variante stricte (remplissage + TRIM)**

Si Write Zeroes n'est pas suffisant ou indisponible :

Créer un système de fichiers temporaire :

```bash
sudo parted /dev/nvme0n1 mklabel gpt
sudo parted /dev/nvme0n1 mkpart primary ext4 0% 100%
sudo mkfs.ext4 -E discard /dev/nvme0n1p1
sudo mkdir /mnt/nvme
sudo mount /dev/nvme0n1p1 /mnt/nvme
```

Remplir complètement + TRIM :

```bash
sudo dd if=/dev/zero of=/mnt/nvme/fill.tmp bs=1G status=progress
sudo rm /mnt/nvme/fill.tmp
sudo fstrim -av /mnt/nvme
sudo umount /mnt/nvme
```

**Vérification**

Contrôler l'absence de données lisibles :

```bash
sudo nvme read /dev/nvme0n1 --start-block=0 --block-count=10 --data-size=5120 | hexdump -C
```

Les données doivent être aléatoires ou nulles (comportement chiffré interne).

Vérifier SMART/usage après traitement :

```bash
sudo nvme smart-log /dev/nvme0n1
sudo nvme list
```

L'utilisation doit refléter un disque « propre » (~0% utilisé).

**Remarques importantes**

- Ne jamais tenter `nvme sanitize` ou `nvme format --ses` sur ces modèles (erreur systématique).
- Consigner : modèle exact (`PC401 NVMe SK hynix 256GB`), firmware (`80006E00`), et méthode utilisée (TRIM + Write Zeroes).
- Si le disque persiste à refuser l'accès après redémarrage, appliquer la procédure **« Variante stricte (remplissage + TRIM) »** de la section **"Protocole TRIM sur SATA"** comme ci-dessous :

```bash
sudo dd if=/dev/zero of=/mnt/ssd/fill.tmp bs=1M status=progress
```

