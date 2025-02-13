Introduction:
-------------
Ce tutoriel a pour but de compiler DPDK pour pfSense.

Pre-requis:
-----------
Pour pouvoir compiler quoi que ce soit pour pfSense il faut :

- Une machine sous FreeBSD avec un kernel d'une version égale ou supérieure à celle de pfSense
- Cette machine doit avoir suffisamment d'espace disque (32Go recommandé), de RAM (8Go recommandé) et le plus de puissance CPU possible car c'est elle qui va cross compiler les programmes
- Il faut avoir git, wget, poudrière, vim, portmaster, portsnap, screen : pkg update && pkg install git wget poudriere vim portmaster portsnap screen
- Se mettre en root sur le FreeBSD

Pour connaître la version du kernel de pfSense on peut regarder sur ce lien.
Installation des sources de pfSense:
------------------------------------
1) Télécharger les sources de pfSense

Il faut tout d'abord récupérer les sources de pfSense.

Pour cela il faut se rendre sur la page Github des sources de pfSense et regarder la liste des branches. Par exemple pour pfSense 2.7.2 il faut sélectionner RELENG_2_7_2.

SI le dossier /usr/src n'est pas vide : cd /usr && rm -R /usr/src/

Il faut ensuite télécharger ce sources avec git clone --branch RELENG_2_7_2 https://github.com/pfsense/FreeBSD-src /usr/src

2)Compiler l'ensemble des programmes de pfSense

On va devoir compiler l'ensemble des binaires des sources de pfSense pour pouvoir créer un environnement de compilation pour nos programmes ou modules kernel.

Pour ordre d'idée la compilation prend 45 min sur 8 coeurs de Intel i9-9900K.

Pour cela :

- cd /usr/src/
- make buildworld -j nombredecoeurCPU

Création de l'environnement de compilation:
-------------------------------------------
Pour compiler nos binaires on va créer un environnement de compilation dans une Jail FreeBSD en utilisant Poudrière.
1)Configuration de poudrière

On va créer le fichier de configuration de poudrière avec cp /usr/local/etc/poudriere.conf.sample /usr/local/etc/poudriere.conf

Puis on le modifie de façon à ce que :

- Si on utilise pas ZFS il faut cocher NO_ZFS=yes
- Si on utilise ZFS on peut récupérer la liste des pools ZFS avec zpool list puis il faut modifier le fichier de configuration avec ZPOOL=nondupool et ZROOTFS=/poudriere
- FREEBSD_HOST=ftp://ftp.freebsd.org

2) Création de l'environnement

Il faut choisir un nom pour notre environnement. Par exemple pf272amd64. Ce nom ne doit pas comporter de point.

Comme notre environnement va compiler des binaires pour un kernel qui n'est pas le dernier à jour (pfSense 2.7.2) et qui n'est donc plus supporté, il faut créer un fichier de configuration avec : vim /usr/local/etc/poudriere.d/nomdelenvironnement-make.conf et mettre dans ce fichier :

ALLOW_UNSUPPORTED_SYSTEM=yes

On va créer une version du port avec : poudriere ports -c -p HEAD

On s'assure de retirer les dépendances inutiles avec pkg autoremove

On va créer notre environnement dans une jail en installant l'ensemble des binaires qu'on a préalablement compilés depuis les sources de pfSense avec poudriere jail -c -j pf272amd64 -a amd64 -m src=/usr/src -p HEAD

Récupération des sources des ports:
-----------------------------------
Pour récupérer la liste des ports on peut portsnap fetch update

On peut vérifier qu'on a bien la liste en allant dans /usr/ports et parcourant tous les ports de programme disponibles à la compilation.

(git clone https://github.com/freebsd/freebsd-ports /usr/ports)

Compiler le port qu'on veut:
----------------------------
1) Création de la liste de port qu'on veut compiler

On va créer un ficher comprenant le chemin de fichier depuis /usr/ports jusqu'à la racine du dossier du port qu'on veut compiler. Par exemple echo net/dpdk > pkglist.txt

2) Compiler le port

Si on est sur une session SSH on ouvrir une session screen avec screen

A tout moment on peut détacher cette session avec CTRL-a d et pour revenir à la session screen -x

On lance la compilation avec poudriere bulk -f pkglist.txt -j pf272amd64 -p HEAD

On peut voir l'avancée avec CTRL - t

3) Récupérer les packages compilés

Les packages compilés se trouvent dans /usr/local/poudriere/data/packages/pf272amd64-HEAD/All/

Installer les packages :
-----------------------
Pour installer ces packages, il faut les récupérer tous d'une même compilation (car il s'agit de dépendances), et exécuter pkg add nomdupackagequ'onveutinstaller.pkg

