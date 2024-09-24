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

## Protocole secure erase sur SATA

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

## Protocole secure erase sur NVME

### Effacement

```bash
sudo nvme format /dev/nvmeXnY --ses=2 (Si le disque prend en charge crypto-erase)
```

```bash
sudo nvme format /dev/nvmeXnY --ses=1 (Si le disque ne prend pas en charge crypto-erase)
```

Vérification

```bash
sudo nvme list
```

Cette commande doit retourner une liste n'affichant pas le support effacé. L'absence du support dans la liste valide son effacement.

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

## L'exception sb[ ] 

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
