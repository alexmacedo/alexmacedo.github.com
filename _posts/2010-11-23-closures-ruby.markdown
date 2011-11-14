---
layout: post
title: Closures em ruby
---

Que ruby &eacute; uma linguagem orientada a objetos todo mundo sabe (ou pelo
menos deveria
saber). Por&eacute;m ruby &eacute; uma linguagem multi-paradigma, ou seja, ela suporta outros
paradigmas de programa&ccedil;&atilde;o, al&eacute;m do orientado a objetos. E atualmente linguagens
que suportam o paradigma funcional tem se destacado. Erlang, Scala e F#, por
exemplo, se encaixam nesta categoria.

Se voc&ecirc; est&aacute; interessado em aprender mais sobre programa&ccedil;&atilde;o funcional, por&eacute;m se
desanima por n&atilde;o conhecer nenhuma das linguagens citadas acima, n&atilde;o se desanime.
Contanto que voc&ecirc; conhe&ccedil;a ruby, caso contr&aacute;rio pode se desanimar (se voc&ecirc; souber
python, ainda existem esperan&ccedil;as).

### O que &eacute; uma _closure_? 

Em programa&ccedil;&atilde;o funcional, a pe&ccedil;a chave &eacute; chamada de _closure_.
Qualquer linguagem
que se diga funcional, deve possuir suporte a _closures_. Mas o que &eacute;
uma _closure_?
Uma boa refer&ecirc;ncia (e mais completa) &eacute; olhar o
[artigo](http://en.wikipedia.org/wiki/Closure_\(computer_science\))
na Wikipedia. Para quem n&atilde;o quiser ler tudo, aqui eu fa&ccedil;o uma pequena s&iacute;ntese:

1. uma _closure_ &eacute; uma fun&ccedil;&atilde;o de primeira ordem. Resumidamente, &eacute; uma
fun&ccedil;&atilde;o que pode
  ser criada, armazenada em uma vari&aacute;vel e ser passada como par&acirc;metro
&agrave; outras fun&ccedil;&otilde;es, do
  mesmo jeito que se faz com qualquer vari&aacute;vel.
2. uma _closure_ "guarda" o contexto no qual ela foi criada. Apesar
desta frase parecer
  estranha, ela somente quer dizer que uma _closure_ se lembra dos
valores das vari&aacute;veis em escopo,
  no momento em que ela &eacute; criada, mesmo que que essas vari&aacute;veis
estejam fora do escopo no momento em que
  a _closure_ for chamada (que pode ser bem mais tarde).

Talvez o segundo item acima ainda tenha deixado um pouco de d&uacute;vidas.
N&atilde;o se preocupe
que mais tarde com um exemplo de c&oacute;digo, tudo vai parecer mais f&aacute;cil.
Outra coisa,
&agrave;s vezes as pessoas se referem &agrave; _closures_ como fun&ccedil;&otilde;es an&ocirc;nimas,
como se fossem
sin&ocirc;nimos, e isso n&atilde;o &eacute; verdade. Em geral, ao se definir _closures_
s&atilde;o utilizadas
fun&ccedil;&otilde;es an&ocirc;nimas, mas estritamente falando, uma fun&ccedil;&atilde;o an&ocirc;nima &eacute; apenas uma
fun&ccedil;&atilde;o que n&atilde;o possui nome, enquanto uma _closure_ deve possuir as
caracter&iacute;sticas ditas
acima.

### Blocks 
Vamos come&ccedil;ar de maneira bem simples, explicando o que s&atilde;o _blocks_.
S&atilde;o exatamente
blocos de c&oacute;digo, que voc&ecirc; pode passar para m&eacute;todos. Vejamos:

    {% highlight ruby %}
    def duas_vezes
      yield
      yield
    end

    duas_vezes { puts "Frase repetida!" }
    {% endhighlight %}

Tamb&eacute;m podemos passar par&acirc;metros para o bloco recebido pelo m&eacute;todo:

    {% highlight ruby %}
    def cumprimenta
      yield "Alexandre"
    end

    cumprimenta { |nome| puts "Ol&aacute; #{nome}!" }
    {% endhighlight %}

Isso &eacute; conhecido como _passagem impl&iacute;cita_ de bloco. Ao chamar
**yield**, voc&ecirc; est&aacute;
assumindo que foi passado algum bloco para o m&eacute;todo. Se nenhum bloco
for passado,
iremos obter um erro (*no_block_given*). Existe uma outra maneira de atingir
o mesmo resultado, s&oacute; que de forma diferente.

    {% highlight ruby %}
    def cumprimenta(&amp;cumprimento)
      cumprimento.call("Alexandre")
    end

    cumprimenta { |nome| puts "Como vai, #{nome}?" }
    {% endhighlight %}

Isso &eacute; o que chamamos de passagem _expl&iacute;cita_ de bloco. O bloco &eacute; a
nossa vari&aacute;vel
_cumprimento_, precedida de um _&amp;_. E assim chamamos o m&eacute;todo _call_
para executar
o c&oacute;digo do nosso bloco. Por&eacute;m h&aacute; limita&ccedil;&otilde;es, s&oacute; podemos passar um bloco para o
m&eacute;todo e a vari&aacute;vel que o representa sempre deve estar em &uacute;ltimo na
lista de par&acirc;metros.
Tudo bem, e qual &eacute; a vantagem da passagem expl&iacute;cita sobre a impl&iacute;cita?

### Procs e lambdas 

Com a passagem expl&iacute;cita, podemos armazenar o "bloco", e apenas
execut&aacute;-lo quando
for necess&aacute;rio, ou pass&aacute;-lo para algum outro m&eacute;todo, como exemplo.

    {% highlight ruby %}
    def execucao_postergada(&amp;bloco)
      @bloco = bloco # guardamos o bloco para executar depois
    end

    execucao_postergada { puts "Esperou &agrave; toa." }
    puts "Suponha que passou muito tempo, dias, meses ou anos."
    puts "S&oacute; vou executar o bloco agora:"
    @bloco.call # "Esperou &agrave; toa." s&oacute; aparece agora
    {% endhighlight %}

Por&eacute;m, ainda estamos restritos a passar apenas um bloco para o nosso m&eacute;todo.
E se quisess&eacute;mos passar v&aacute;rios blocos?

    {% highlight ruby %}
    def dois_blocos(bloco1, bloco2)
        bloco1.call
        bloco2.call
    end

    dois_blocos(Proc.new { puts "Primeiro bloco." }, Proc.new { puts "Segundo bloco." })
    {% endhighlight %}

Pronto! Descobrimos uma maneira de passar quantos blocos forem
necess&aacute;rios, e de certa
forma descobrimos como o exemplo anterior funciona. O _&amp;_ transforma o
bloco que passamos
para o m&eacute;todo em uma **Proc**. E adivinhem, a **Proc** funciona como
uma _closure_, isto &eacute;,
possui aquelas duas caracter&iacute;sticas do in&iacute;cio do post. Voc&ecirc; j&aacute;
percebeu que pudemos passar
v&aacute;rias **Procs** em par&acirc;metros ou atribuir **Procs** para vari&aacute;veis,
logo n&atilde;o resta
d&uacute;vidas da caracter&iacute;stica 1. E a 2, aquela sobre escopo de vari&aacute;veis?

    {% highlight ruby %}
    k = 10
    closure = Proc.new { |v| v * k }

    def meu_metodo(v, closure)
      k = 5 # variavel k no escopo do metodo
      closure.call(v)
    end

    meu_metodo(10, closure)   # 100 e nao 50
    closure.call(10)          # 100
    {% endhighlight %}

Ent&atilde;o, sabe aquela hist&oacute;ria sobre guardar contexto? &Eacute; isso que aconteceu quando
criamos a closure, por isso, apesar de redefinir o **k** em outro
escopo, ela se "lembrou"
do valor correto. E o inverso, funciona? Vamos ver outro exemplo:

    {% highlight ruby %}
    def somador
      soma = 0
      Proc.new { |item| soma += item }
    end

    s = somador
    soma = 100
    s.call(1)  # 1
    soma = 0
    s.call(10) # 11
    {% endhighlight %}

Veja que tamb&eacute;m funciona, a nossa closure ignora a vari&aacute;vel _soma_ fora do
escopo do m&eacute;todo. E o pr&oacute;ximo exemplo?

    {% highlight ruby %}
    k = 10

    multiplica = Proc.new { |v| k *= v }
    soma = Proc.new { |v| k += v }

    multiplica.call(2) # 20
    soma.call(10)      # 30
    {% endhighlight %}

O exemplo acima parece bobo, mas mostra uma importante caracter&iacute;stica
da linguagem
_ruby_. Para voc&ecirc; ter _closures_, a linguagem que deve dar suporte, e no caso de
diferentes linguagens, existem diferentes suportes. Basicamente, existem duas
abordagens poss&iacute;veis quando se trata de implementa&ccedil;&atilde;o de _closures_:

- uma _closure_ cria uma c&oacute;pia de todas as vari&aacute;veis de que necessita
no momento da sua cria&ccedil;&atilde;o.
  Se voc&ecirc; olhar o exemplo acima, ver&aacute; que este n&atilde;o &eacute; o caso, do
contr&aacute;rio os dois resultados
  seriam 20.
- uma _closure_ estende a "vida" das vari&aacute;veis de que necessita, e ao
inv&eacute;s de copiar o valor
  delas, apenas mant&eacute;m a refer&ecirc;ncia para elas. Foi o que aconteceu
acima, as duas closures
  est&atilde;o utilizando o mesmo **k**, e n&atilde;o uma c&oacute;pia dele.

&Eacute; importante fazer essa distin&ccedil;&atilde;o para que voc&ecirc; n&atilde;o tenha surpresas inesperadas.

Outra coisa, utilizar o **Proc.new** n&atilde;o &eacute; a &uacute;nica maneira de criar
_closures_ em ruby.
A verdade &eacute; que existem outras maneiras at&eacute; mais sucintas:

    {% highlight ruby %}
    k = 10
    multiplica = lambda { |v| k *= v } # utilizando lambda
    soma = proc { |v| k += v }         # e utilizando proc
    multiplica.call(2) # 20
    soma.call(10)      # 30
    {% endhighlight %}

Apesar de parecerem id&ecirc;nticos, existem diferen&ccedil;as.

### Diferen&ccedil;as entre _Procs_ e _lambdas_ 

Bem, aqui vai um pouco de confus&atilde;o. Se voc&ecirc; utiliza o ruby 1.8,
**proc** e **lambda** s&atilde;o
exatamente a mesma coisa. E ambas s&atilde;o diferentes de **Proc.new**.
Parece que algu&eacute;m
percebeu essa "inconsist&ecirc;ncia", e no ruby 1.9, **proc** e **Proc.new**
s&atilde;o a mesma coisa,
e ambas diferentes de **lambda**.

Mas e a&iacute;, qual a diferen&ccedil;a? Uma delas &eacute; sobre fluxo de execu&ccedil;&atilde;o. Veja o exemplo:

    #!ruby
    def testa_proc
      com_proc = Proc.new {   # com proc seria a mesma coisa
        puts "Retornando da proc."
        return
      }
      com_proc.call
      puts "Fim do teste."
    end

    def testa_lambda
      com_lambda = lambda {
        puts "Retornando do lambda."
        return
      }
      com_lambda.call
      puts "Fim do teste."
    end

    testa_proc   # "Retornando da proc."
    testa_lambda # "Retornando do lambda." "Fim do teste."

Veja que ao chamar o **return** de dentro da **Proc**, n&oacute;s encerramos
a execu&ccedil;&atilde;o do
m&eacute;todo. No caso do **lambda**, apenas encerramos a execu&ccedil;&atilde;o do pr&oacute;prio
**lambda**.

A outra diferen&ccedil;a &eacute; com rela&ccedil;&atilde;o aos par&acirc;metros. Veja outro exemplo:

    #!ruby
    com_lambda = lambda { |nome| puts "Voc&ecirc; passou: #{nome}" }
    com_proc = Proc.new { |nome| puts "Voc&ecirc; passou: #{nome}" }

    com_proc.call("Proc")     # Voc&ecirc; passou: Proc
    com_lambda.call("lambda") # Voc&ecirc; passou: lambda
    com_proc.call             # Voc&ecirc; passou:
    com_lambda.call           # ArgumentError

Veja que no caso da **Proc**, o par&acirc;metro &eacute; opcional, se n&atilde;o &eacute; passado
um par&acirc;metro,
a **Proc** utiliza **nil**. No caso do **lambda**, os par&acirc;metros s&atilde;o
obrigat&oacute;rios, e se
n&atilde;o forem passados, teremos um **ArgumentError**.

Uma maneira de diferenciar se voc&ecirc; est&aacute; lidando com uma **Proc** ou um
**lambda**, &eacute; utilizar
o m&eacute;todo **lambda?**:

    #!ruby
    lambda {}.class     # Proc
    lambda {}.lambda?   # true

    Proc.new {}.class   # Proc
    Proc.new {}.lambda? # false

    proc {}.class       # Proc
    proc {}.lambda?     # false

Por&eacute;m, o m&eacute;todo **lambda?** s&oacute; existe na vers&atilde;o 1.9 do _ruby_.

Em um pr&oacute;ximo post, devo continuar abordando o "ruby funcional".
Agora foi apenas um aquecimento, neste post nem cheguei a falar sobre
_currying_!

