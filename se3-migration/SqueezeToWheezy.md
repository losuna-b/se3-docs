# Migration de se3-squeeze vers se3-wheezy

* [Présentation](#présentation)
* [Préparation du `se3 Squeeze`](#préparation-du-se3-squeeze)
    * [Configurer `apt`](#configurer-apt)
    * [Mise à jour du `se3` ?](#mise-à-jour-du-se3-)
    * [Supprimer les profiles ?](#supprimer-les-profiles-)
    * [Supprimer les montages de disques externes](#supprimer-les-montages-de-disques-externes)
    * [Dernière précaution](#dernière-précaution)
    * [Prévenir les collègues…](#prévenir-les-collègues)
    * [Temps à prévoir](#temps-à-prévoir)
* [Migration vers un `se3-Wheezy`](#migration-vers-un-se3-wheezy)
    * [Utilisation d'une session `screen`](#utilisation-dune-session-screen)
    * [Utilisation du script de migration](#utilisation-du-script-de-migration)
    * [Redémarrer à la fin ?](#redémarrer-à-la-fin-)
* [Réparer `Grub` ?](#réparer-grub-)
    * [Télécharger et graver `boot-repair`](#télécharger-et-graver-boot-repair)
    * [Démarrer le `se3` sur le DVD gravé](#démarrer-le-se3-sur-le-dvd-gravé)
    * [Autre solution](#autre-solution)
    * [Configurer l'onduleur](#configurer-londuleur)
* [Post-migration](#post-migration)
    * [Les modules](#les-modules)
    * [Remettre en place les disques de sauvegarde](#remettre-en-place-les-disques-de-sauvegarde)
* [Utiliser les scripts de sauvegarde/restauration](#utiliser-les-scripts-de-sauvegarderestauration)



## Présentation

Cet article propose une procédure qui vous permettra de migrer, en toute sénérité, votre `se3-squeeze` vers un `se3-wheezy`.

Cet article est une synthèse réalisée à partir d'échanges collaboratifs parus sur la liste de dicussion de l'académie de Cæn `l-samba-edu@ac-caen.fr` : il est donc en évolution en fonction des questions, des réponses et précisions apparues lors de la discussion sur cette liste.

**Remarque 1 :** il se peut que les infos des derniers échanges ne soient pas encore intégrées ; cela prend un certain temps…

Si des intérrogations surgissent au cours de la lecture de cet article, n'hésitez pas à les partager sur la liste.

**Remarque 2 :** Si vous avez encore un `se3-lenny` ou une version antérieure, le mieux est d'utiliser les script de sauvegarde/restauration : cela est détaillé ci-dessous avec la méthode alternative qui est valable aussi pour migrer d'un `se3-squeeze` vers un `se3-wheezy`.

**Remarque 3 :** Vous pourrez vous entraîner sur un réseau `se3 virtuel`, ce qui vous permettra de le faire ensuite plus sereinement sur votre précieux `se3` si cela vous semble nécessaire. Bien sûr, ce n'est pas obligatoire et le script fonctionne très bien maintenant. L'article ci-dessous, un peu ancien et il faudrait le mettre à jour, vous guidera pour la mise en place d'un réseau virtuel.
> http://wiki.dane.ac-versailles.fr/index.php?title=Installer_un_r%C3%A9seau_SE3_avec_VirtualBox


## Préparation du `se3 Squeeze`

### Configurer `apt`
→ avoir corrigé les divers sources.list : normalement la mise à jour récente du se3 devrait le faire.

Sur un `se3-squeeze` à jour (*Version 2.4.9299* à la date de rédaction de cet article), **/etc/apt/sources.list** contient :
```sh
deb http://archive.debian.org/debian squeeze main contrib non-free
deb http://archive.debian.org/debian squeeze-lts main contrib non-free
```

…et **/etc/apt/sources.list.d/se3.list** contient :
```sh
# sources pour se3
deb http://wawadeb.crdp.ac-caen.fr/debian squeeze se3

#### Sources testing desactivee en prod ####
#deb http://wawadeb.crdp.ac-caen.fr/debian squeeze se3testing

#### Sources XP desactivee en prod ####
#deb http://wawadeb.crdp.ac-caen.fr/debian squeeze se3XP
```

→ créer le fichier **/etc/apt/apt.conf** (normalement, il n'existe pas)
```sh
touch /etc/apt/apt.conf
```
→ l'éditer (avec `nano` par exemple)
```sh
nano /etc/apt/apt.conf
```
…et y coller la ligne suivante :
```sh
Acquire::Check-Valid-Until false;
```

### Mise à jour du `se3` ?
Inutile, cela sera fait par le script de migration ; cependant, pour les angoissés, voici quelques commandes utiles pour cette mise à jour :
```sh
aptitude update
aptitude safe-upgrade
aptitude -P full-upgrade # selon la réponse, validez
bash /usr/share/se3/scripts/se3_update_system.sh
```

**Remarque :** si le `se3-squeeze` est à jour, vous disposerez, de ce fait, de la dernière version du script. D'ailleurs, le script de migration vérifie que le `se3` est bien à jour et, sinon, il le met à jour ; c'est pour cette raison que vous devez le relancer pour bénéficier de la version la plus à jour du script de migration.


### Supprimer les profiles ?
*Franck : "le script supprime immédiatement celui du compte admin et les autres sont supprimés en arrière plan. Cela étant je vais peut être modifier cela pour être certain qu'il ne traîne plus rien à la fin du script."*
Il n'est pas interdit de le faire préventivement.
```sh
rm -rf /home/profiles/*
```


> En faisant ça, les raccourcis firefox  dégagent, non ?
Si cela fait partie de l'arborescence `/home/profiles/`, oui, en effet. Cependant, il me semble qu'il est plutôt dans le `/home` de l'utilisateur : il n'est donc pas touché. Précision de Franck : seuls, les favoris `IE` passent à la trappe.

Sur les données utilisateurs, vous perdrez peu de choses : les favoris
Windows, les favoris internet explorer (IE), tout ce qui est dans le profil V2.


### Supprimer les montages de disques externes
En accord avec une bonne pratique de sauvegarde des données, vous avez mis en place les sauvegardes `backuppc` et `sauveserveur` (pour cette dernière, la plus importante, voir ci-dessous la solution alternative). Il vaut mieux les démonter pour éviter des surprises lors de l'exécution du script de migration.
Le script démonte le disque monté sur `/var/lib/backuppc` mais pas celui monté sur `/sauveserveur`.
```sh
umount /var/lib/backuppc
umount /sauveserveur
```
…et on débranche pour éviter un remontage automatique. Voir ci-dessous. 


### Dernière précaution
Si cela n'est déjà en place, nous vous recommendons chaudement de procéder à la sauvegarde du type `sauveserveur` dont nous avons parlé ci-dessus et qui est documentée dans la solution alternative ci-dessous, avant de lancer le script de sauvegarde ; vous ferez plus sereinement la migration.


### Prévenir les collègues…
De toute façon, il est prudent de les prévenir en leur demandant de procéder à une sauvegarde de leurs données… Ce qu'ils devraient toujours faire.

Par ailleurs, il faut que le `se3` ne soit pas utilisé pendant la migration.

Le mieux est de couper le `se3` du reste du réseau (si cela est possible) pour éviter que des utilisateurs tentent de s'y connecter ; je pense cela préférable vu qu'à la fin du script de migration s'exécute en arrière plan un script qui efface tous les profils et réencode les home en `UTF-8`, c'est très long !


### Temps à prévoir
Il vous faut une journée pour être sur que tout soit bien fait.


## Migration vers un `se3-Wheezy`

### Utilisation d'une session `screen`
`screen` est une session particulière en ce sens que vous pouvez la quitter sans que le processus soit arrêté, contrairement à une session normale. Vous trouverez sans doute de la doc sur la toile. Sinon, il faudrait que je retrouve un mémo écrit par François Lafont.
L'utilisation de `screen` est d'ailleurs suggérée par le script.
```sh
screen
```

**Remarque :** il se peut que le paquet ne soit pas installé. Dans ce cas, il faut l'installer :
```sh
aptitude install screen
```

### Utilisation du script de migration
```sh
bash /usr/share/se3/sbin/se3_upgrade_wheezy.sh
```

Une première utilisation de ce script a lieu puis il faudra le relancer pour poursuivre la migration : les messages donnés à l'écran précisent tout cela très clairement.

Pendant cette première utilisation, le script vérifie qu'il a la dernière version stable avec cette `url` :
> http://wawadeb.crdp.ac-caen.fr/majse3/se3_upgrade_wheezy.sh
C'est une des raisons pour lesquelles le script doit être relancé. Avec la 2ème utilisation du script, on sera donc avec la version stable de ce script.

Il existe une version de dev' (qui sera bientôt transférée en version stable) et qui est, quant à elle, désormais sur le `github` :
> https://github.com/SambaEdu/maintscripts/blob/master/migration/se3_upgrade_wheezy.sh

Cette version de dev' ajoute quelques fonctions au script comme la possibilité d'utiliser des options afin de se trouver dans différents modes : debug, pas de téléchargement ou encore préchargement des paquets dans le cache `apt`.

Les voici (ces options ne servant que pour la mise au point du script, vous n'avez pas à les utiliser) :
* --download | -d : préparer la migration sans la lancer en téléchargeant uniquement les paquets nécessaires
* --no-update     : ne pas vérifier la mise à jour du script de migration sur le serveur centrale mais utiliser la version locale
* --debug         : lancer le script en outrepassant les tests de taille et de place libre des partitions. À NE PAS UTILISER EN PRODUCTION


### Redémarrer à la fin ?
Il n'est pas nécessaire de redémarrer… Mais ce n'est pas interdit ;-)
```sh
reboot
```

## Réparer `Grub` ?
Il se peut que `Grub` soit cassé à la suite de la migration ; il suffit de le réparer.

Cependant, il faudrait voir dans quel cas cela arrive pour palier ce problème.


### Télécharger et graver `boot-repair`
Récupérer l'archive `iso` de `boot-repair` :
> https://sourceforge.net/p/boot-repair/home/fr/

Graver un DVD (pas fait, je n'en ai pas eu besoin) avec cette archive `iso`.


### Démarrer le `se3` sur le DVD gravé
> La réparation est automatique ou y-a-t-il des options à choisir ? **Nicolas, un retour ?**


### Autre solution
> *Marc : "`grub-repair` n'avait pas marché (deux jours de chaos au lycée). J'avais réussi à réinstaller `grub` en suivant la procédure en `chroot` du site ci-dessous (j'avais fait les deux techniques de cette partie… Les deux m'indiquaient un message d'échec mais c'était reparti !)"*

> https://wikiUtiliser les scripts de sauvegarde/restauration.debian-fr.xyz/R%C3%A9installer_Grub2


## Configurer l'onduleur

Normalement, la migration ne doit pas modifier la configuration de l'onduleur.

D'ailleurs, si cette configuration n'a pas était faite sur votre `se3-squeeze`,une fois la migration effectuée il sera plus que temps de la mettre en chantier, en vous aidant des indications de l'article suivant. Là, vous n'aurez pas d'excuses en cas de pépin…
> http://www.samba-edu.ac-versailles.fr/Sauvegarde-et-restauration-SE3

Cependant, il vaudra mieux configurer l'onduleur **avant la migration** car s'il y a un problème sur l'alimentation électrique ou des micro-coupures, ce sera plus chaud pour vous ;-) Mais vous aurez pris, de toute façon, la précaution d'avoir une sauvegarde de type `sauveserveur` à jour (voir la remarque 2 de la présentation ci-dessus ou bien la solution alternative ci-dessous) avant de passer aux choses sérieuses…


## Post-migration

### Les modules
Presque tous les modules ont été reportés sur `Wheezy`. Mais `se3-unattended` a disparu (voir ci-dessous) et `se3-internet` est en testing pour l'instant.
> http://wawadeb.crdp.ac-caen.fr/versions-paquets-se3.html

**Module `se3-pla` :** c'est le nouveau paquet permettant l'exploration de l'annuaire Ldap. À vous de l'installer :
```sh
aptitude install se3-pla
```

**Module `se3-ocs` :** il est désinstallé et purgé lors de la migration. Il est ré-installable ensuite. `se3-ocs`, version `Wheezy`, sera vierge comme l'a précisé Franck. Tout est effacé du passage de `se3-squeeze` à `se3-wheezy` pour éviter des problèmes de base de donnée.

**Module `se3-maintenance` :** il a disparu (peut-être sera-t-il remis).

**Module `se3-unattended` :** ce module n'est pas supprimé lors de la migration contrairement à `se3-ocs`. Par conséquent si vous l'avez installé sous `se3-squeeze` vous conserverez cette version sous `se3-wheezy`. Simplement le module n'étant plus maintenu, il n'a pas été reporté sous `Wheezy` dans une nouvelle version.

**Le paquet wpkg `wsuoffline` :** *Laurent : "pour ce paquet wpkg, j'ai l'impression qu'il ne fait pas les mises à jour correctement sur W7 de manière automatique. Il faudrait que je jette un œil sur un poste pour en être sûr".*


**Remarque :** ne vous embêtez pas à mettre votre `se3-Wheezy` en testing, la branche stable est suffisamment bien peuplé.


### Remettre en place les disques de sauvegarde
Il suffit de rebrancher les disques et deux solutions se présentent :

* soit redémarrer le serveur si des entrées dans le fichier `/etc/fstab` concernent ces disques,
* soit de les monter conformément aux indications de la documentation.


## Utiliser les scripts de sauvegarde/restauration

**Une solution alternative** à l'utilisation du script ci-dessus est la suivante :
* sauvegarder votre serveur (script `sauve_serveur.sh`)
* installer un `se3-wheezy`, par la méthode de votre choix
* configurer les modules adaptés à votre situation
* restaurer votre serveur (script `restaure_serveur.sh`)

Cette méthode sera à utiliser si vous avez une version antérieure à celle de `se3-squeeze`.

Ces deux scripts (`sauve_serveur.sh` et `restaure_serveur.sh`) sont proposés sur le site ci-dessous, ainsi que la documentation d'utilisation.

Il est à noter qu'ils sont aussi sur votre `se3-squeeze`, s'il est à jour bien entendu. Cependant, les versions les plus récentes de ces scripts se trouveront sur le site suivant :
> http://www.samba-edu.ac-versailles.fr/Sauvegarde-et-restauration-SE3

**Remarque :** une chose à prendre en compte sur ces scripts : sur `se3-wheezy` on est full `utf8`, et `samba 4` n'aime pas du tout les noms de fichiers avec des accents codés en `iso`. Cela fait quelques versions que nous sommes en `utf8` côté `samba` mais, pour autant, s'il y a eu des versions successives, il y a de fortes chances que de le codage `iso` traîne encore dans `/home` et `/var/se3`.

**La solution à ce problème**, c'est de réencoder avec `/usr/bin/convmv`. Cela fonctionne très bien mais demande d'analyser tous les fichiers.

Voici les deux commandes pour cela :
```sh
/usr/bin/convmv --notest -f iso-8859-15 -t utf-8 -r /home 2&>1 | grep -v Skipping >> $fichier_log
/usr/bin/convmv --notest -f iso-8859-15 -t utf-8 -r /var/se3 2&>1 | grep -v Skipping >> $fichier_log
```

