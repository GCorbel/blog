---
layout: post
title: Mes trucs pour augmenter sa productivité
date: 2014-05-19 04:21
comments: true
categories:
---

Dans cet article, vous verrez quels outils j'utilise afin d'avoir un bon
rendement au travail en tant que développeur.

<!--more-->

## Système d'exploitation - Ubuntu

Le choix d'un système d'exploitation est une première étape obligatoire. Comme
beaucoup, j'ai commencé par Windows. Pour le développement, ce n'est
vraiment pas l'idéal. Premièrement, dans mon langage de programmation favori, Ruby,
beaucoup d'outils sont d'abord faits pour Linux et Mac avant d'être adaptés, ou
pas, pour Windows. Ensuite, le terminal est un outil ultra important.
L'interface graphique permet d'utiliser des fonctions qui sont en fait des
lignes de commandes dans le système. Utiliser l'interface, c'est se limiter a
ce qu'on a voulu de nous proposer. Linux et Mac sont vraiment plus orientés
pour l'utilisation du terminal.

Linux vs Mac : Il n'y a pas de bonnes ou de mauvaises réponses. Je n'utilise pas
Mac, car il ne correspond pas à ma philosophie. Je n'ai pas envie d'être
contrôlé par Apple et j'adore l'Open Source. Si vous préférez payer, libre a
vous.

Pourquoi Ubuntu? Tout simplement parce que c'est la distribution la plus connue.

## Terminal - ZSH

[Zsh](http://www.zsh.org/) est un remplaçant de Bash. Zsh offre
un meilleur affichage et une navigation plus rapide en ne tenant pas
compte des majuscules dans les noms de fichiers et de dossiers. Une meilleure
intégration avec git est disponible. Si votre répertoire est versionné, vous
verrez quelle est la branche courante.
[Oh-My-Zsh](https://github.com/robbyrussell/oh-my-zsh) offre
des configurations préfaites pour certains outils comme git ou rails.

Il est possible de configurer Zsh. Si l'on n'aime pas le fait d'ignorer la
casse, une seule ligne suffit dans le fichier `.zshrc` pour changer le
comportement.

## Éditeur de texte - VIM

Choisir un éditeur de texte est crucial. Certains préfèrent de gros IDE comme
Eclipse. D'autres préfèrent des outils plus simples comme Sublime Text ou
Notepad++. J'ai testé beaucoup de solutions avant de choisir. J'ai finalement
choisi [Vim](http://www.vim.org/) et j'en suis bien content.

J'encourage tout le monde à essayer Vim. Les premiers pas sont difficiles, mais
**le jeu en vaut la chandelle**. Cela fait plus d'un an que je n'utilise que
cet éditeur. Je pense avoir augmenté ma productivité, mais je sais qu'il me
reste encore des milliers de trucs à apprendre. Vim, c'est tout un monde.

La philosophie de Vim vise a éviter l'utilisation de la souris. En effet, le
temps passé à bouger la main entre la souris le clavier peut paraitre anodin,
mais on le fait des centaines de fois et il s'agit d'une totale perte de temps.
L'apprentissage de Vim demande de casser les habitudes.

Vim est un éditeur modal. Six modes sont disponibles :

* Normal : Offre des tonnes de raccourcis afin de naviguer, de supprimer, de
copier, etc.;
* Insertion : Permets d'écrire du texte;
* Commande : Permets d'effectuer des commandes dans le fichier ou sur l'OS;
* Mode Visuel : Permets d'effectuer des sélections;
* Mode Sélection : Permets d'éditer un texte sélectionné;
* Mode Ex : Comme le mode commande, mais ne retourne pas en mode normal après
  l'exécution de la commande;

Le fait d'utiliser des modes permet d'avoir plus de possibilités, car chaque mode
posséder ces propres raccourcis et fonctionnalités.

Il existe de nombreux plug-ins Vim. Voici une liste de mes préférés :

* [YouCompleteMe](https://github.com/Valloric/YouCompleteMe) : Auto complétion puissante;
* [ctrlp](https://github.com/kien/ctrlp.vim) : Fuzzyfinder permettant la navigation rapide entre les fichiers;
* [ag.vim](https://github.com/rking/ag.vim) : Permets de faire des recherches rapides;
* [ultisnips](https://github.com/SirVer/ultisnips) : Ajoute des fragments de code rapidement;

Vim est hautement personnalisable. L'édition du fichier `~/.vimrc` permet
d'ajouter de la configuration, mais également de faire vos propres bouts de
code. Il est également possible d'ajouter des raccourcis et de configurer les
combinaisons du clavier pour exécuter plus rapidement certaines actions.

## TMUX

[Tmux](http://tmux.sourceforge.net/) permets d'exploiter plusieurs terminaux au sein d'un seul écran. On peut
créer des panneaux, qui vont séparer l'écran en plusieurs parties, chaque
partie étant un terminal où il est possible d'exécuter des commandes. Il est
possible de regrouper les panneaux en fenêtres. La liste des fenêtres est
affichée en bas de l'écran. Il est également possible de créer des sessions
qui sont un regroupement de fenêtres. À tout instant, il est possible de
naviguer entre les différents éléments à l'aide de raccourcis clavier.

Mon truc préféré avec Tmux est de le faire parler avec Vim. J'ai ajouté le
plug-in [tslime](https://github.com/kikijump/tslime.vim/blob/master/tslime.vim), qui permet d'envoyer une commande dans un panneau.
Je l'utilise pour exécuter les tests. Lorsque j'appuie sur les touches `,t`,
Vim lance l'exécution de Rspec, avec le fichier et la ligne courante, et lance
le test dans un panneau séparé. Cela me permet d'exécuter les tests rapidement
tout en restant asynchrone et en me permettant d'afficher et de garder la trace
de l'erreur lorsque je modifie mon code.

J'utilise également [Tmuxinator](https://github.com/tmuxinator/tmuxinator), qui permet d'enregistrer des configurations Tmux
et de démarrer une session avec un certain affichage et exécuter certaines
commandes au démarrage. Je l'ai configuré pour accéder rapidement aux projets
sur lesquels je travaille. La commande `tmuxinator c`, me mettra dans le dossier
du projet et créera trois panneaux, le principal avec Vim, le second lancera le
serveur et le dernier est libre pour pouvoir exécuter des commandes. J'ai créé
un raccourci avec Zsh pour simplement taper `ti c`.

## Taper au clavier

Utiliser les bons outils ne sert à rien si l'on tape avec deux doigts. J'ai
suivi [un cours en ligne](http://www.typingweb.com/) pour apprendre à taper. Cela fait vraiment une grosse
différence. Le fait de taper de manière plus fluide permet non seulement
d'aller plus vite, mais de garder plus facilement le fil de ses idées. En
suivant des entrainements, même les personnes tapant déjà rapidement peuvent
s'améliorer.

Taper vite est utile quand on code, mais pas seulement. C'est également un
avantage lors de l'écriture d'un email, d'un document, ou d'un article de blog.

## Formation

Taper vite avec les bons outils ne sert à rien si l'on tape n'importe quoi. Je
suis inscrit à plusieurs sites permettant de m'améliorer dans ma pratique. J'ai
également lu des livres le développement. Enfin, écrire des articles de blog
sur des articles que je connais, en plus de me permettre de partager ma connaissance,
me donne une rigueur qui me permet d'approfondir certaines connaissances.

Le plus important est de ne jamais arrêter de coder. C'est avec la pratique
qu'on s'améliore. Il faut également prendre des temps pour regarder de plus
près ce que l'on fait et se poser les bonnes questions.

Petit conseil pour les unilingues francophones : N'ayez pas peur de l'anglais.
C'est dur au début, mais avec le temps ça va mieux. De plus, quand on lit des
livres sur le développement, c'est possible de comprendre le sens en lisant le
code.

## Conclusion

Les outils que j'utilise sont à mon goût et je suis certain que je vais en
découvrir d'autres avec le temps. Je crois que l'important est de tester et de
faire ses propres choix.

Vous pouvez voir mes fichiers de configuration sur [le repo GitHub](https://github.com/GCorbel/dotfiles).
