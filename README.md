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
    - Incovénients:   
  - No : Connexion root imposible
    - Avantages : Aucun risque de sécurité
    - Incovénients:
  - Prohibit-password (default) : Le mdp et le 'keyboard-interactive authentification' sont désactivé
    - Avantages : 
    - Incovénients :  
  - Forced-commands-only : Permet le login root avec une clef d'autentification public (Uniquement si l'option 'command' a été spécifiée)
    - Avantages :
    - Incovénients :
   
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
