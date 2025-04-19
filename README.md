# Projet OS : Jeu Sherlock13

## Introduction
Ce projet regroupe les concepts abordés lors des TPs réalisés avec M. PECHEUX ainsi que la réalisation du jeu multijoueur Sherlock13.

### But du jeu 
Trouver quel personnage est le criminel, c’est-à-dire le seul personnage non visible par les joueurs (la seule carte cachée).


### Contenu du jeu
- 13 cartes Personnage (chacun ayant des symboles spécifiques)
- Un bloc de feuilles d’enquête


### Mécaniques de base
Chaque joueur se voit attribué 4 cartes personnages mais une carte est cachée : c’est le criminel. En posant des questions aux autres joueurs, il faut déduire quel personnage est sur cette carte cachée.


### Déroulement d’un tour
Chaque joueur choisit, lors de son tour, une seule action parmi les 3 suivantes :

1 :
  - Choisir un symbole (ex : loupe, crâne…).
  - Demander aux autres qui possèdent ce symbole de le dire.
    On apprend alors combien de personnes possèdent ce symbole

2 :
  - Choisir un joueur spécifique et un symbole.
    Ce joueur doit te dire combien de fois il voit ce symbole parmi ses cartes.

3 : 
  - Accuser un personnage que tu soupçonnes être le criminel.
    Si tu as raison → tu gagnes !
    Si tu as tort → tu es éliminé, mais tu continues à répondre aux questions.


### Utilisation de la feuille d’enquête
Celle-ci est divisée en deux parties :
En haut, tu coches les symboles vus et fais des déductions.
En bas, tu notes les cartes que tu as vues (les tiennes), et ce que les autres révèlent petit à petit.


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

Pour chaque client, il suffit alors de cliquer sur le bouton "Connect" qui apparait avec l'interface graphique lors de l'exécution du client.
