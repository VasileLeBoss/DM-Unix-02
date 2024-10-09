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
  - La clef va se généré dans le dossier saisie ou par default dans (`/root/.ssh`)
- Pour la transferer sur le serveur ssh on utilise la commande :
<pre>
    ssh-copy-id username@remote_server
</pre> 
