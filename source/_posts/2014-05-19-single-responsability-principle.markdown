---
layout: post
title: "Single Responsability Principle"
date: 2014-05-19 19:32
comments: true
categories:
published: false
---

S'il n'y avait qu'un principe de programation orienté objet a retenir, ça
serait celui-ci. Dans cet article, nous allons voir le S, des principes SOLID,
le Principe de Responsabilité Unique ou Single Responsability Principle (SRP)
en anglais.

<!-- more -->

## Le principe

Le Single Responsability Principle a été introduit par Robert C. Martin, alias Uncle Bob. D'après celui-ci, une classe doit avoir une, et une seule, responsabilité. Commençons par exemple ayant plus d'un responsabilité.

``` ruby
class User < ActiveRecord::Base
    attr_accessor :username, :email, :token, :password, :password_confirmation

    validates :username, presence: true

    before_create :generate_token

    def to_param
        token
    end

    def send_email_after_validation
        UserMailer.send_mail_after_validation(self).deliver
    end

    def send_email_to_remember_password
        UserMailer.send_email_to_remember_password(self).deliver
    end

    private

    def generate_token
        token = SecureRandom.urlsafe_base64
    end
end
```

On peut voir que cette classe est responsable de la persistence des données, de
la génération des token, de l'envoi d'email après la validation et pour le
rappel de mot de passe. En général, les modèles sont des aspirateur a
responsabilité. Ceux-ci est dû au principe "Fat model, skinny controller".

## Bénéfices

Pourquoi changer cette classes? Il existe plusieur bénéfice resultant de l'utilisation du Single Responsability Principle.

Premièrement, le fait de segmenter les fonctions en plusieurs classes permet
d'augmenter la clareté. Les classes vont être moins longue et donc, plus facile
a lire. Si l'on suit les principe de Sandi Metz, une classe ne devrait pas
dépasser les cent ligne. Plus c'est court, plus c'est facile a comprendre. De
plus, il va être plus facile de donner de classe plus representatif.

Ensuite, il sera plus facile d'instancier et d'utilser ces classes.
Normalement, une classe avec une seule responsabilité aura besoin de moins de
paramêtres en argument. Il s'agit d'un gros avantage quand viens le temps de
faire les tests.

Enfin, une classe avec une seule responsabilité favorise l'utilisation du Duck
Typing étant donnée qu'il peut y avoir une seule méthode. Il est possible de
nommer cette méthod de la même manière, `call` par exemple. Ce qui fait qu'il
sera toujours possible d'appeller la méthode ainsi `object.call`.

## Only on reason to change

Robert C. Martin décrit une responsabilté comme une raison de changer. Une
classe ou un module doit avoir seulement une raison de changer. Si l'on prend
l'exemple de la classe `User`, celle-ci devra changer si l'on décide de générer
des token différement, ou encore si l'on change les courriels à envoyer, ou si
l'on change le type de persistence des données. Bref, cette classe a
definitvement plus d'une raison de changer.

Un element sur le quel il faut faire attention, c'est la responsabilité qui
doit avoir une raison de changer et non le code. On peut imaginer dix milles
raisons de changer de code dans une classe, par exemple, changer le nom d'une
fonction ou d'un variable mais ces changement sont acceptable. Tandis que, si
l'on change la responsabilité de la classe, on change sa raison d'être et il
faut que ça soit justifier.
