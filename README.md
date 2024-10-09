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
  
**Autre techniques**
- Changer le port
  - C'est une modification très simple, qui permet d'éviter un bon nombre d'attaques, puisque certains robots ne     tentent de se connecter que sur le port par défaut.
- Nombre de tentatives d'authentification par connexion
  - La configuration par défaut du serveur OpenSSH autorise six tentatives d'authentification par connexion. Réduire cette valeur permet de limiter l'efficacité des attaques lors de l'utilisation de règles basées sur le nombre de connexions, puisque la connexion sera alors fermée automatiquement après un nombre réduit d'échecs.
- Liste blanche
  - Dans certains cas, il est possible d'établir une liste des adresses des machines ou réseaux autorisés, et d'interdire l'accès à tous les autres.
    
**Source** : 

https://blog.garamotte.net/posts/2018/01/07/fr-limit-brute-force-attacks-on-the-ssh-service.html#quelques-solutions-possibles

## Processus 

### 2.1 Etude des processus UNIX
- `ps -eF` pour afficher les processus tournant
<pre>UID          PID    PPID  C    SZ   RSS PSR STIME TTY          TIME CMD
root           1       0  0 42221 13568   8 09:36 ?        00:00:02 /sbin/init
root           2       0  0     0     0   1 09:36 ?        00:00:00 [kthreadd]
root           3       2  0     0     0   0 09:36 ?        00:00:00 [rcu_gp]
root           4       2  0     0     0   0 09:36 ?        00:00:00 [rcu_par_gp]
root           5       2  0     0     0   0 09:36 ?        00:00:00 [slub_flushwq]
root           6       2  0     0     0   0 09:36 ?        00:00:00 [netns]
root           8       2  0     0     0   0 09:36 ?        00:00:00 [kworker/0:0H-events_highpri]
root          10       2  0     0     0   0 09:36 ?        00:00:00 [mm_percpu_wq]
root          11       2  0     0     0   0 09:36 ?        00:00:00 [rcu_tasks_kthread]
root          12       2  0     0     0   0 09:36 ?        00:00:00 [rcu_tasks_rude_kthread]
root          13       2  0     0     0   0 09:36 ?        00:00:00 [rcu_tasks_trace_kthread]
root          14       2  0     0     0   0 09:36 ?        00:00:00 [ksoftirqd/0]
root          15       2  0     0     0   4 09:36 ?        00:00:00 [rcu_preempt]
</pre>
