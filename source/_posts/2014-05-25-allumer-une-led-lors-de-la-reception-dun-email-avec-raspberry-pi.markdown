---
layout: post
title: Allumer une LED lors de la réception d'un email avec Raspberry Pi
date: 2014-05-25 10:01
comments: true
categories:
---

L'article a pour but d'expliquer, en détail, les principes de programmation avec le
[Raspberry Pi](http://www.raspberrypi.org/). Comme exemple, je vais indiquer
les différentes étapes pour faire un petit programme en Python permettant
d'allumer une LED lorsqu'un courriel arrive sur un compte GMail.

<!--more-->

## Raspberry Pi
Commençons par la base. Le Raspberry Pi est un petit ordinateur possédant un
processeur ARM. Dans la même catégorie, il existe [Arduino](http://www.arduino.cc/) et [BeagleBone](http://beagleboard.org/bone). Chacun
possède ses forces comme indiqué dans [cet article](http://makezine.com/2013/04/15/arduino-uno-vs-beaglebone-vs-raspberry-pi/).
Avec le Raspberry Pi, il est possible de choisir un OS à installer. Le plus
utilisé est Raspbian, une version dérivée de Debian pour Raspberry. C'est celui
que j'utilise dans cet article.

Dans mon cas, j'ai acheté le [Raspberry Pi Ultimate Starter Kit](http://www.amazon.ca/CanaKit-Raspberry-Ultimate-Starter-Components/dp/B00GWTNYJW/ref=sr_1_1?ie=UTF8&qid=1401027372&sr=8-1).

## Le circuit
Mes notions d'électroniques remontent au collège et avaient besoin d'être
dépoussiérées. Le principe est simple, l'électricité est émise par le Raspberry
et suit un chemin du positif vers le négatif. Entre ces deux étapes, j'ai
placé une LED, pour la lumière, et une résistance, pour s'assurer que
le voltage ne dépasse pas une certaine puissance.

Le Raspberry Pi possède différents ports d'entrées et de sorties. Les signaux
électriques sont envoyés par des GPIOs. Un GPIO (General Purpose Input/Output)
est un port sans but prédéfini. Il est possible de choisir s'il s'agit d'un
port d'entrée ou de sortie ainsi que la puissance émise par ce port.

Il est facile de se perdre dans le numéro des GPIO et des ports. Les deux
possèdent une numérotation différente et sont souvent utilisées dans les
articles. Voici une image indiquant la liste des ports et les GPIOs
correspondants :

![](http://img11.hostingpics.net/pics/720336RaspberryPiGPIOLayoutRevision1.png)

Le circuit électrique que j'ai fait est celui de [cet article](https://projects.drogon.net/raspberry-pi/gpio-examples/tux-crossing/gpio-examples-1-a-single-led/).

Voici le schéma suivi et adapter pour utiliser les pièces du kit :

|[](http://img15.hostingpics.net/pics/2000521ledgpiobb1267x300.jpg)

Tout d'abord, pour vérifier que le circuit fonctionne correctement, je l'ai
connecté au port du 3.3v comme montré sur cette image :

![](http://img11.hostingpics.net/pics/861306photo1.jpg)

Le chemin doit about à la mise à la terre (ground). Par convention, le positif est représenté par des fils rouges et le négatif est représenté par des fils noirs. Ça fonctionne, je peux passer à la création du programme.

## Choix du langage
Le premier choix à faire lors du commencement d'un programme est celui du langage.
J'ai toujours pensé qu'un projet avec une composante électrique devait avoir
un langage comme le C, ou plus bas niveau.

Étant donné que Raspberry Pi fonctionne avec un OS normal, il est possible
d'installer n'importe quel langage, mais je crois qu'il est bon de rester sur
les deux installés de base tout simplement parce que ce sont ceux qu'utilise la
communauté Raspberry. Ces deux langages sont les C et le Python.

Après un comparatif de ces langages, **j'ai finalement choisi le Python**. Bien que le
langage en lui-même est plus lent, il est tellement plus rapide de programmer
avec celui-ci qu'il reste une option intéressante. Je crois qu'une bonne pratique
est de faire un programme en Python et, si la rapidité devient un point
problématique, refaire le même programme en C.

De toute façon de nombreux autres éléments peuvent influencer de manière plus
importante la rapidité, comme la vitesse du réseau par exemple.

## Allumer une LED via un GPIO
Dans cet exemple, nous allons utiliser le GPIO 17. Le choix est totalement
arbitraire et n'a aucune importance. Avant de faire un programme permettant de
se connecter à GMail, j'ai commencé par faire clignoter la LED. Voici le circuit avec le GPIO changé :

![](http://img11.hostingpics.net/pics/250456photo2.jpg)

Et voici le programme :

```python
import RPIO
import time

gpio = 17
on = True

RPIO.setmode(RPIO.BCM)
RPIO.setup(gpio, RPIO.OUT)

while True:
    RPIO.output(gpio, on)
    on = not on
    time.sleep(.1)
```

Comme vous pouvez le constater, j'ai choisi d'utiliser
[RPIO](https://pypi.python.org/pypi/RPIO). Une bibliothèque simple
et efficace.

Deux modes permettent d'indiquer le type de numérotation que l'on souhaite.
Si l'on choisit `RPIO.BOARD`, il faudra indiquer le numéro du port et pour `RPIO.BCM` celui
du GPIO. Les deux fonctionnent aussi bien et le choix est à votre convenance.
`RPIO.setmode` permet de choisir un mode ou l'autre.

`RPIO.setup` permet de configurer le port pour faire en sorte qu'il s'agisse
d'une entrée ou d'une sortie. Dans ce cas, il s'agit d'une sortie.

`RPIO.output` permet d'indiquer le signal à envoyer au port. Il est possible de
lui envoyer un booléen, 0, `GPIO.LOW:`, 1 ou `GPIO.HIGH`.

## Connecter le programme avec GMail

Pour connecter le programme avec GMail, j'ai utilisé [imaplib](https://docs.python.org/2/library/imaplib.html).
Voici le résultat final :

```python
import RPIO
import time
import imaplib

gmail = imaplib.IMAP4_SSL('imap.gmail.com', '993')
gmail.login('username', 'password')
gmail.select()

gpio = 17

RPIO.setmode(RPIO.BCM)
RPIO.setup(gpio, RPIO.OUT)

while True:
    _, data = gmail.search(None, 'UnSeen')
    if len(data[0].split()) == 0:
        RPIO.output(gpio, False)
    else:
        RPIO.output(gpio, True)
```

Et voilà! Quand un nouveau email arrive, la LED s'allume. Si le courriel
n'est plus marqué comme lu, celle-ci s'éteind.

## Conclusion

L'application utilisée comme exemple n'est pas la plus utile du monde, mais
permet de voir les concepts de base du développement avec Raspberry Pi.
