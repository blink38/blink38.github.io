---
layout: post
title: Synchroniser ses données de site Drupal via rsync avec snapshots
description: Synchroniser ses données de site Drupal via rsync avec snapshots
summary: Synchroniser ses données de site Drupal via rsync avec snapshots
comments: false
toc: true
tags: [drupal, rsync, snapshot, freenas]
---

Contexte : j'ai besoin de récupérer régulièrement les données de mes sites Drupal en production pour pouvoir les injecter sur mes sites de pré-production ou de développement. 

Les données de site Drupal dont j'ai besoin sont :

 * Les fichiers (répertoire files)
 * La base de données

J'ai besoin de pouvoir disposer de plusieurs versions des sauvegardes en cas de besoin. Je dispose par ailleurs d'un serveur Freenas qui utilise le système de fichier ZFS. Je peux donc m'appuyer dessus pour mettre en oeuvre un système de snapshot avec ZFS.

Pour la suite, je souhaite stocker les données de *site_internet* sur le Freenas.

### ZFS Dataset 

En premier lieu, nous allons créer sur Freenas un dataset ZFS sur lequel nous ferons les snapshots.

Dans **Storage** > **Pools**, sélectionner le pool dans lequel vous souhaitez créer un dataset. Cliquez sur les 3 petits points à droite, et choisissez **Add dataset**

![capture](/assets/images/2020-11-23/zfs-add-dataset.png "Add dataset")

### Utilisateur Freenas

Je vais ensuite crééer un utilisateur sur le freenas que j'utiliserais pour me connecter à distance sur le freenas en ssh (rsync utilise ssh pour communiquer).

Dans **Accounts** > **Users**, je crée un utilisateur que je nomme **synchro**

Je lui crée un répertoire personnel que j'utiliserais pour y stocker un script qui sera lancé après la synchronisation rsync.

* Full Name : Synchronisation des données
* Username : synchro
* email : sans
* Password : en choisir même si on peut faire sans.
* Home Directory : /mnt/pool01/home/synchro
* Authentication / SSH Public Key : y mettre la clé publique de l'utilisateur de votre serveur drupal qui lancera la synchronisation des données
* Shell : bash (*vous n'êtes pas obligé, c'est une préférence personnelle*)

### Script de création des snapshots

Il faut se connecter en ssh sur le Freenas avec l'utilisateur synchro. Soit vous avez mis votre clé SSH publique dans *Authentication / SSH Public Key** de l'utilisateur synchro, soit vous vous souvenez du mot de passe.

Créez le fichier **/mnt/pool01/home/synchro/snapshot-site-internet** (j'ai lu qu'il ne fallait pas mettre de point dans le nom du fichier).

```bash
#!/bin/bash

if [ $RSYNC_EXIT_STATUS -eq 0 ]
	zfs snapshot pool01/drupal/site_internet@rsync-`date "+%Y-%m-%d-%Hh%M"`
fi
```

Ce script, exécuté par rsync, exécutera le snaphost du dataset drupal/site_internet si l'exécution de rsync s'est déroulé sans accroc.

(en chantier)

---

Sources :

1. https://www.deltasight.fr/versioning-rsync-btrfs/
2. https://www.howtoforge.com/tutorial/how-to-use-snapshots-clones-and-replication-in-zfs-on-linux/
3. man page rsyncd.conf (pour avoir des infos sur RSYNC_EXIT_STATUS et les variables d'environnement à disposition lors du post-xfer)

---
