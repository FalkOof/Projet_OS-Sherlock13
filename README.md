# Projet OS : Jeu Sherlock13

## Introduction
Ce projet regroupe les concepts abordés lors des TPs réalisés avec M. PECHEUX ainsi que la réalisation du jeu multijoueur Sherlock13.

### But et Contenu du jeu 
Trouver quel personnage est le criminel, c’est-à-dire le seul personnage non visible par les joueurs (la seule carte cachée).

Le jeu est composé de :
- 13 cartes Personnage (chacun ayant des symboles spécifiques)
- Un bloc de feuilles d’enquête


### Mécaniques de base
Chaque joueur se voit attribué 4 cartes personnages mais une carte est cachée : c’est le criminel. En posant des questions aux autres joueurs, il faut déduire quel personnage est sur cette carte cachée.

Chaque tour, chaque joueur choisit, lors de son tour, une seule action parmi les 3 suivantes :

- Choisir un symbole (ex : loupe, crâne…).
- Demander aux autres qui possèdent ce symbole de le dire.
  On apprend alors combien de personnes possèdent ce symbole

- Choisir un joueur spécifique et un symbole.
  Ce joueur doit te dire combien de fois il voit ce symbole parmi ses cartes.

- Accuser un personnage que tu soupçonnes être le criminel.
  Si on a raison, on gagne.
  Si on a tort, on est éliminé mais on continue à répondre aux questions.


### Utilisation de la feuille d’enquête
Celle-ci est divisée en deux parties :
En haut, on coche les symboles vus et on fait des déductions.
En bas, on note les cartes que l'on a dans notre main, et ce que les autres révèlent petit à petit.


### Fin de la partie
La partie se termine dès qu’un joueur fait une accusation correct ou s'il ne reste plus qu'un joueur dans la partie (les autres sont éliminés).


# Compilation et Execution du Projet 

Pour compiler le Projet, un makefile était déjà fournis, il suffit de l'exécuter en faisant la commande : 
```bash
./cmd.sh
```
Suite à cela il vous faudra ouvrir un total de 5 terminaux, 1 pour le serveur et 1 pour chaque joueur. 
On peut lancer le serveur en l'executant et en indiquant un numero de port, par exemple : 
```bash
./server 1234
```
Et on peut lancer chaque joueur en executant le client et en indiquant sur quel port est le serveur, sur quel port communiquer et quel est le nom du joueur, par exemple :
```bash
./sh13 127.0.0.1 1234 127.0.0.1 12340 Test0
```

Pensez bien à changer le port sur lequel le client communique à chaque client.

Ensuite, pour chaque joueurs, il suffit alors de cliquer sur le bouton "Connect" qui apparait avec l'interface graphique lors de l'exécution du client.
Une fois que tous les joueurs sont connectés, le jeu peut actuellement commencer selon les règles décrites ci-dessus.



# Fonctionnement complet du code

## Code du Serveur

On rappelle que le but ici est de permettre au serveur de gérer les connexions de 4 clients, leur distribue des cartes, et s'occupe des tours de jeu jusqu’à la fin de la partie (accusation correcte ou élimination de tous sauf un joueur).

### Connexion et enregistrement des clients
L’objectif du serveur est de gérer une partie multijoueur entre 4 clients. Chaque client se connecte en envoyant un message de type C au serveur contenant son adresse IP, son port d’écoute et son pseudo.
Le serveur utilise différentes méthodes pour se faire :
- socket() pour créer une socket de communication TCP (AF_INET / SOCK_STREAM),
- bind() pour lier la socket à un port local,
- listen() pour écouter les connexions entrantes,
- accept() pour accepter ces mêmes connexions,
- read() pour lire un message reçu,
- write() pour envoyer un message vers un autre client en se connectant à son port.

Pour faire cela, on utilise une machine à états (sur la valeur de fsmServer) afin de gérer le déroulement du programme :
- fsmServer == 0 : le serveur attend les connexions des 4 joueurs.
- fsmServer == 1 : le jeu est en cours et on traite les envoies des joueurs.
- fsmServer == 2 : la partie est terminée.

Chaque joueur est stocké dans un tableau tcpClients[4] de structures contenant :
```bash
struct _client{
    char ipAddress[40];
    int port;
    char name[40];
} tcpClients[4];
```

Un message de type I id est envoyé au joueur nouvellement connecté pour lui donner son identifiant (id), et un message L nom1 nom2 nom3 nom4 est broadcasté à tous les joueurs pour annoncer la liste des joueurs connectés à chaque nouvelle connexion.


### Mélange et distribution des cartes
Une fois les 4 joueurs connectés, le deck est mélangé avec melangerDeck(), qui réalise une permutation aléatoire des 13 cartes. J'ai rajouté la bibliothèque <time.h> afin de faire une vraie permutation aléatoire seeder sur l'heure, sinon la permutation était toujours la même. Ensuite, les 12 premières cartes sont réparties équitablement entre les 4 joueurs (3 par joueur) et la carte restante (deck[12]) est le coupable.
Le tableau tableCartes[4][8] est ensuite construit pour associer à chaque joueur les symboles qu’il détient via ses cartes. Chaque carte est en effet représentée comme un ensemble de symboles (entre 0 et 7). Ce tableau est rempli en utilisant la fonction createTable().
Chaque joueur reçoit donc un message D c1 c2 c3 contenant les identifiants des 3 cartes qu’il possède, ainsi que 8 messages V id i j pour remplir la ligne i de sa vue sur les symboles (ligne tableCartes[i]).
Enfin, le serveur annonce le début du jeu en envoyant le message M 0, indiquant que c’est au joueur 0 de jouer.


### Gestion du jeu (état fsmServer == 1)
Pendant la partie, le serveur gère différentes actions des joueurs, identifiées par la première lettre du message reçu quand ils envoient un. Il faut donc faire attention lorsque l'on essaye de print quelque chose dans le terminal car si le message contient en première lettre une des lettres suivantes, cela peut casser le code. il y a différentes actions possibles dont voici la liste :
- G (accusation d'un coupable) : le serveur vérifie si la carte désignée est bien le coupable. Si oui alors le joueur a gagné et la partie est terminée, si non alors le joueur est éliminé, il ne jouera plus et son tour sera passé. Si tous les joueurs sauf un sont éliminés, alors la partie prend fin (état fsmServer = 2).
- O (demande globale de symbole) : le serveur compte combien de joueurs ont ce symbole et il répond R nb x si au moins un joueur possède le symbole, sinon il répond R nb 0.
- S (demande ciblée de symbole) : le serveur regarde combien de fois le joueur idCible possède ce symbole et répond R nbSymbole.

Enfin, à chaque fin de tour le serveur met à jour le joueur courant en sautant les joueurs éliminés et il envoie à tous un message M id pour indiquer qui joue ensuite.


## Code du Client


Le but du client est de se connecter au serveur, de recevoir les informations de jeu, et de permettre à l’utilisateur de jouer son tour ainsi que d'envoyer ses actions au serveur.

### Connexion au serveur et Réception des données
Le client commence par se connecter au serveur via une socket TCP en utilisant socket(), puis connect() pour établir la connexion. Une fois connecté, il envoie un message de type C contenant son IP, son port d’écoute (pour recevoir des messages du serveur ou d'autres clients) ainsi que son pseudo.
Le client crée également une socket d'écoute sur son propre port pour pouvoir recevoir des messages entrants, en utilisant bind() puis listen() et un accept() dans un thread séparé (puisqu’on veut pouvoir écouter et envoyer en même temps). Ce thread gère les messages entrants en continu.


Une fois connecté, le client reçoit différentes informations du serveur :
- un message de type I id pour indiquer au client son identifiant (entre 0 et 3).
- un message de type L nom1 nom2 nom3 nom4 pour annoncer les pseudos de tous les joueurs connectés.
- un message de type D c1 c2 c3, ce sont les identifiants des 3 cartes que possède le joueur.
- une série de messages V id i j pour compléter la table de symboles visibles du joueur (tableCartes).

Le client initialise alors son interface ou ses structures internes avec les infos de départ.

### Déroulement du jeu côté client

Quand c’est au tour du joueur, il reçoit un message M id avec id étant son propre identifiant. À ce moment, une boucle permet à l’utilisateur de choisir une action parmi les suivantes :
- faire une demande globale de symbole (message O s) : le joueur demande au serveur combien de joueurs possèdent un certain symbole s. Le serveur répond avec un message R nb x, où x vaut 0 si personne ne le possède.
- faire une demande ciblée de symbole (message S id s) : le joueur demande combien de fois le joueur id possède le symbole s. Le serveur répond avec un message R nbSymbole.
- faire une accusation (message G c) : le joueur accuse une carte d’être le coupable. Le serveur renvoie ensuite le résultat de l'accusation, met éventuellement à jour les états, et annonce si la partie est terminée.

Une fois l’action terminée, le joueur attend la fin de son tour. À la fin de chaque tour, un nouveau message M id est reçu pour indiquer qui joue ensuite.


## Lien avec les concepts techniques abordés en TP

L'application créée repose sur la communication client-serveur via des sockets TCP avec les fonctions usuelles socket(), bind(), listen(), accept() pour créer un serveur. On utilise aussi forcément connect() pour envoyer une réponse à un client. Le programme est mono-processus, il n'y a aucune création de processus enfants avec fork(). Enfin, le serveur est séquentiel : il traite chaque message reçu dans une boucle while principale.
