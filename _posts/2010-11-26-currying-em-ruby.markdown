---
layout: post
title: Currying em ruby
---

Seguindo o [post anterior](http://alexandrebm.com/closures-em-ruby),
vou continuar
falando sobre caracter&iacute;sticas da programa&ccedil;&atilde;o funcional utilizando ruby.
E uma t&eacute;cnica muito usada em programa&ccedil;&atilde;o funcional &eacute; o _currying_. Novamente, se
voc&ecirc; estiver interessado em mais detalhes, veja o [artigo](http://en.wikipedia.org/wiki/Currying) na
_Wikipedia_ sobre _currying_. Para quem tiver mais pressa,
_currying_ &eacute; uma
t&eacute;cnica que transforma uma fun&ccedil;&atilde;o com v&aacute;rios argumentos em uma
sequ&ecirc;ncia de fun&ccedil;&otilde;es
que recebem apenas um argumento.

A defini&ccedil;&atilde;o ainda parece estranha? Talvez um exemplo com c&oacute;digo ajude.
Isto &eacute; o que vimos no post anterior:

    {% highlight ruby %}
    multiplica = lambda { |x, y| x * y }
    multiplica.call(21, 2) # =&gt; 42
    {% endhighlight %}

Ok, agora vamos utilizar o m&eacute;todo **curry**:

    {% highlight ruby %}
    multiplica = lambda { |x, y| x * y }.curry
    multiplica.call(21).call(2) # =&gt; 42
    {% endhighlight %}

Em vez de ter uma fun&ccedil;&atilde;o que recebe dois par&acirc;metros, temos duas
fun&ccedil;&otilde;es (note que o m&eacute;todo
**call** foi chamado duas vezes) que recebem um par&acirc;metro cada. Assim,
quando definimos
alguma fun&ccedil;&atilde;o dessa forma:

    {% highlight ruby %}
    lambda { |x, y| x * y }
    {% endhighlight %}

Ao utilizar o m&eacute;todo **curry** acabamos obtendo isso:

    {% highlight ruby %}
    lambda { |x| lambda { |y| x * y } }
    {% endhighlight %}

### E por que isso pode ser &uacute;til? 

Em alguns casos, quando voc&ecirc; tem uma fun&ccedil;&atilde;o que recebe m&uacute;ltiplos
par&acirc;metros, pode
ser &uacute;til fixar um deles. A partir disso voc&ecirc; acaba obtendo uma nova fun&ccedil;&atilde;o (&agrave;s
vezes referida como _fun&ccedil;&atilde;o parcial_). Veja o exemplo:

    {% highlight ruby %}
    multiplica = lambda { |x, y| x * y }.curry

    dobro = multiplica.call(2)  # =&gt; nova fun&ccedil;&atilde;o, com x = 2
    triplo = multiplica.call(3) # =&gt; nova fun&ccedil;&atilde;o, com x = 3

    dobro.call(21)  # =&gt; 42
    triplo.call(14) # =&gt; 42
    {% endhighlight %}

Estes exemplos funcionam apenas na vers&atilde;o 1.9 do ruby, na vers&atilde;o 1.8
n&atilde;o existe o
m&eacute;todo **Proc#curry**.

