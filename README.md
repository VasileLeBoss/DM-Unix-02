# Compte Rendu

## Secure Shell : SSH

### 1.1 Connection ssh root 

- Pour se connecter en tant qu'administrateur, il faut changer le paramètre : 
<pre>
  # PermitRootLogin off
</pre>
- Les argument posible sont :
  - Yes : Connexion root posible avec un mdp
    - Avantages : Connexion root facile
    - Incovénients: Permettre la connexion root avec un mot de passe rend le système vulnérable aux attaques par force brute.
  - No : Connexion root imposible
    - Avantages : Aucun risque de sécurité
    - Incovénients: Moins pratique
  - Prohibit-password (default) : Le mdp et le 'keyboard-interactive authentification' sont désactivé
    - Avantages : Seules les connexions SSH avec clés publiques sont autorisées
    - Incovénients : on doit configurer des clés SSH avant de pouvoir te connecter en tant que root
  - Forced-commands-only : Permet le login root avec une clef d'autentification public (Uniquement si l'option 'command' a été spécifiée)
    - Avantages : Accès root uniquement avec des clés SSH
    - Incovénients : La connexion root n'est possible que pour des commandes spécifiques, ce qui peut être restrictif pour des tâches d'administration générales
   
### 1.2 Authentification par clef / Génération de clefs

- Pour généré une clef privé et public sur la machine hote on utilise la commande :
<pre>
   ssh-keygen -t rsa  
</pre>
- Lors de (`Enter passphrase (empty for no passphrase):`) il faut créér une phrase de passe afin de renforcer la sécurité de la connexion au serveur
  - Si quelqu’un obtient un accès non autorisé à la clé privée, il doit toujours connaître la phrase de passe pour l’utiliser.  
- La clef va se généré dans le dossier saisie ou par default dans (`/root/.ssh`)
- Pour la transferer sur le serveur ssh on utilise la commande :
<pre>
    ssh-copy-id username@remote_server
</pre> 
### 1.3 Authentification par clef / Connection serveur
- Pour aller dans le dossier /.ssh/ on utilise la commande
<pre>
  cd ~/.ssh/
</pre>

### 1.4 Authentification par clef : depuis la machine hote
<pre>
  ssh -i ./maclef root@10.20.0.136
</pre>
  - `-i` pour specifié la clef privé avec laquelle on veux se connecter
  - `./maclef` le nom de la clef privé
### 1.5 Sécurisez

- Les attaques de type `brute-force ssh` consiste a esseyer tout les combinaison posible pour se connecter en tant qu'administrateur
- Pour éviter ces tentatives il faut changer le parametre `PermitRootLogin` et lui attribuer l'argument `Forced-commands-only`. Cela va permettre de ce connecter en tant qu'administrateur uniqument avec la clef privé.

  `OU`

- Utilisation de `fail2ban`
<pre>
  apt-get install fail2ban
</pre>
  - Editer : /etc/fail2ban/jail.conf
<pre>
  [ssh] enabled = true 
  port = ssh 
  filter = sshd 
  action = iptables[name=SSH, port=ssh, protocol=tcp] 
  logpath = /var/log/auth.log
  maxretry = 3 
  bantime = 900
</pre>
  - Et enfin redemarrer le service
<pre>
  service fail2ban restart
</pre>
**Autre techniques**
- Changer le port
  - C'est une modification très simple, qui permet d'éviter un bon nombre d'attaques, puisque certains robots ne     tentent de se connecter que sur le port par défaut.
- Nombre de tentatives d'authentification par connexion
  - La configuration par défaut du serveur OpenSSH autorise six tentatives d'authentification par connexion. Réduire cette valeur permet de limiter l'efficacité des attaques lors de l'utilisation de règles basées sur le nombre de connexions, puisque la connexion sera alors fermée automatiquement après un nombre réduit d'échecs.
- Liste blanche
  - Dans certains cas, il est possible d'établir une liste des adresses des machines ou réseaux autorisés, et d'interdire l'accès à tous les autres.
    
**Source :** https://blog.garamotte.net/posts/2018/01/07/fr-limit-brute-force-attacks-on-the-ssh-service.html#quelques-solutions-possibles

## Processus 

### 2.1 Etude des processus UNIX
- `ps -aux` pour afficher les processus tournant

<pre>USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root           1  0.0  0.1 168884 13568 ?        Ss   09:36   0:02 /sbin/init
root           2  0.0  0.0      0     0 ?        S    09:36   0:00 [kthreadd]
root           3  0.0  0.0      0     0 ?        I&lt;   09:36   0:00 [rcu_gp]
root           4  0.0  0.0      0     0 ?        I&lt;   09:36   0:00 [rcu_par_gp]
root           5  0.0  0.0      0     0 ?        I&lt;   09:36   0:00 [slub_flushwq]
root           6  0.0  0.0      0     0 ?        I&lt;   09:36   0:00 [netns]
root           8  0.0  0.0      0     0 ?        I&lt;   09:36   0:00 [kworker/0:0H-events_highpri]
root          10  0.0  0.0      0     0 ?        I&lt;   09:36   0:00 [mm_percpu_wq]
root          11  0.0  0.0      0     0 ?        I    09:36   0:00 [rcu_tasks_kthread]
</pre>
- L'information `TIME` idique le temps d'utilisation du processeur du processus ou du thread

**Source :** https://www.theunixschool.com/2012/09/ps-command-what-does-time-indicate.html

- Le processeur ayant le plus utilisé le processeur c'est le `PID 1` - aussi connu comme `init`
- Le premier processus lancé après le démarrage du système c'est le `/sbin/init `
- La machine a démarré à `09:36`
- Une autre commande permetant de trouver le temps depuis lequel la machine tourne est `uptime`
<pre>14:26:52 up  7:59,  3 users,  load average: 0,00, 0,00, 0,00
</pre>
- Pour trouver le nombre approximatif de processus créés depuis le démarrage on utilise la commande :
<pre>
  root@serveur-correction:~# ps -eaf | wc -l 
  157
</pre> 
- Trouver une option de la commande ps permettant d’afficher le PPID d’un processus.
<pre>
  ps -e | grep "NomDeProcessus"
</pre>
**Source :** https://www-tecmint-com.translate.goog/find-parent-process-ppid/?_x_tr_sl=en&_x_tr_tl=fr&_x_tr_hl=fr&_x_tr_pto=rq

- Installation `pstree`
<pre>
  apt update 
  apt install psmisc
</pre>
- `pstree`
<pre>systemd─┬─cron
        ├─dbus-daemon
        ├─dhclient
        ├─login───bash
        ├─sshd───sshd───bash───pstree
        ├─systemd───(sd-pam)
        ├─systemd-journal
        ├─systemd-logind
        ├─systemd-timesyn───{systemd-timesyn}
        └─systemd-udevd
</pre>
- `top`
<pre>top - 15:10:01 up  8:42,  2 users,  load average: 0,02, 0,03, 0,00
Tâches:<b> 154 </b>total,<b>   1 </b>en cours,<b> 153 </b>en veille,<b>   0 </b>arrêté,<b>   0 </b>zombie
%Cpu(s):<b>  0,0 </b>ut,<b>  0,0 </b>sy,<b>  0,0 </b>ni,<b> 99,9 </b>id,<b>  0,0 </b>wa,<b>  0,0 </b>hi,<b>  0,0 </b>si,<b>  0,0 </b>st 
MiB Mem :<b>   9942,8 </b>total,<b>   9421,4 </b>libr,<b>    395,8 </b>util,<b>    365,9 </b>tamp/cache     
MiB Éch :<b>   6173,0 </b>total,<b>   6173,0 </b>libr,<b>      0,0 </b>util.<b>   9547,0 </b>dispo Mem 

<span style="background-color:#000000"><span style="color:#FFFFFF">    PID UTIL.     PR  NI    VIRT    RES    SHR S  %CPU  %MEM    TEMPS+ COM.                             </span></span>
<b>   3155 root      20   0   11740   5548   3372 R   0,7   0,1   0:00.08 top                              </b>
    176 root      20   0       0      0      0 I   0,3   0,0   0:20.14 kworker/7:1-events_power_effici+ 
      1 root      20   0  168884  13568   9228 S   0,0   0,1   0:02.41 systemd                          
      2 root      20   0       0      0      0 S   0,0   0,0   0:00.01 kthreadd                         
      3 root       0 -20       0      0      0 I   0,0   0,0   0:00.00 rcu_gp                           
      4 root       0 -20       0      0      0 I   0,0   0,0   0:00.00 rcu_par_gp                       
      5 root       0 -20       0      0      0 I   0,0   0,0   0:00.00 slub_flushwq                     
      6 root       0 -20       0      0      0 I   0,0   0,0   0:00.00 netns                            
      8 root       0 -20       0      0      0 I   0,0   0,0   0:00.00 kworker/0:0H-events_highpri      
     10 root       0 -20       0      0      0 I   0,0   0,0   0:00.00 mm_percpu_wq  
</pre>
- La touche `Shift+M` trie les processeur par occupation mémoire decroissante.
- Le processus le plus `gournabnd` c'est le `systemd`
  - c'est un gestionnaire de système et de services pour les systèmes d'exploitation Linux
- La commande interactive permettant de passer a l'affichage en couleur est `z`
- Instalaltion htop :
<pre>
  apt install htop
</pre>
  - Les avantage :
    - une interface graphique plus atirante
   
  - Les inconvénients :
    - d

## Arret d'un processus 

- creation fishier :
<pre>
  touch date.sh
</pre>
- Modification :
<pre>
  nano date.sh
</pre>
- Execution :
<pre>
  ./date.sh
</pre>
- Le script affiche tout les decondes en boucle toto et l'heure actuele - 5h  
## Les tubes

### Quelle est la différence entre tee et cat 
- `cat` est utilisé principalement pour lire le contenu d'un fichier et l'afficher dans le terminal. Il permet aussi de concaténer plusieurs fichiers pour afficher leur contenu ensemble ou rediriger la sortie vers un autre fichier.

**Exemple**
<pre>
  cat fichier1.txt fichier2.txt > combiné.txt
</pre>

- `tee` lit depuis l'entrée standard et envoie simultanément le résultat à la sortie standard (l’écran) et dans un fichier.

**Exemple**
<pre>
  echo "Bonjour" | tee fichier.txt
</pre>

**Source :** https://linux.die.net/man/1/tee 


### Que font les commandes suivantes 
- `ls | cat` afficher simplement le résultat de la commande ls car `cat` ne modifie pas la sortie.

**output :**
<pre>
  fichier1.txt fichier2.txt
</pre>

- `ls -l | cat > liste` ls -l envoie sa sortie à cat. Ensuite, cette sortie est redirigée vers un fichier nommé liste.Si on regarde la liste, on obtien :

**output :** 
<pre>
  -rw-r--r-- 1 root root  1024 Oct 14 10:00 fichier1.txt
  -rw-r--r-- 1 root root  2048 Oct 14 10:02 fichier2.txt
</pre> 

- `ls -l | tee liste` envoie la sortie à deux endroits en même temps : elle l'affiche sur le terminal et l'écrit dans le fichier liste. 

**output :** 

Affichage à l'écran :
<pre>
  -rw-r--r-- 1 user user  1024 Oct 14 10:00 fichier1.txt
  -rw-r--r-- 1 user user  2048 Oct 14 10:02 fichier2.txt
</pre>

Contenu du fichier `liste`
<pre>
  -rw-r--r-- 1 user user  1024 Oct 14 10:00 fichier1.txt
  -rw-r--r-- 1 user user  2048 Oct 14 10:02 fichier2.txt
</pre>

- `ls -l | tee liste | wc -l` va lister les fishiers avec les détails > tee liste affiche cette sortie dans le terminal et l'enregistre dans le fichier liste  > wc -l affiche ke nombre total de lignes dans cette sortie

**output :** 
<pre>
    -rw-r--r-- 1 user user  1024 Oct 14 10:00 fichier1.txt
    -rw-r--r-- 1 user user  2048 Oct 14 10:02 fichier2.txt
    2
</pre>

## Journal système rsyslog

**rsyslog**
- Pour verifier si le service est lancé on utilise la commande `systemctl status rsyslog` et pour savoir son PID `pidof rsyslog`
- Les messages système standards sont souvent écrits dans le fichier `/var/log/syslog`
- Pour verifier ces fichiers on peut les ouvrir avec `tail -f /var/log/syslog`

**source :** https://linux.die.net/man/1/tail

**cron**
- `cron` permet d'exécuter automatiquement des scripts ou des commandes à des moments spécifiques, suivant un calendrier défini dans des fichiers de configuration appelés `crontabs`

**tail -f**
- permet d'afficher les dernières lignes d'un fichier et de mettre à jour dynamiquement l'affichage en temps réel 
- pour visualiser le contenu de `/var/log/messages` et temp réel :
<pre>
  tail -f /var/log/messages
</pre>
- redémarrer le service cron
<pre>
  sudo systemctl restart cron
</pre>

- Dans le terminal on verra les logs liées au redémarrage de cron

### À quoi sert le fichier /etc/logrotate.conf 
- Le fichier `/etc/logrotate.conf` sert à configurer le comportement de l'outil logrotate, qui est utilisé pour gérer la rotation, la compression et la suppression des fichiers de log. Il permet d'éviter que les fichiers de logs deviennent trop volumineux en les scindant périodiquement et en supprimant ou archivant les anciens fichiers.

**dmesg**
- La commande `dmesg` affiche les messages du buffer de la mémoire noyau, liés au matériel détecté lors du démarrage.
- Linux détecte le processeur de ma machine hôte
<pre>
  [0.000000] CPU: Intel(R) Core(R) i7-4770 CPU  @3.40GHz
</pre>
- Et la carte réseau émulées
<pre>
  [1.234567] virtio_net: Virtio network device
</pre>



