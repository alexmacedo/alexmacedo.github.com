---
layout: post
title: Introdu&ccedil;&atilde;o ao virtualenv
---

Uma das ferramentas que eu mais gosto do ecossistema  _ruby_ &eacute; o
[rvm](https://rvm.beginrescueend.com/). O objetivo do _rvm_ &eacute; permitir
que diversos interpretadores ruby, de diferentes vers&otilde;es, possam coexistir 
no mesmo ambiente. Dessa forma voc&ecirc; pode manter um projeto antigo usando a
vers&atilde;o 1.8, desenvolver um novo projeto usando a 1.9, e at&eacute; mesmo utilizar
jruby em outro. Outra _feature_ muito legal &eacute; a cria&ccedil;&atilde;o de _gemsets_, que
servem para isolar a instala&ccedil;&atilde;o de _gems_. Inclusive uma boa pr&aacute;tica ao 
usar o _rvm_ &eacute; criar uma _gemset_ por projeto, de forma a n&atilde;o misturar as
diversas _gems_ no sistema.

Ok, mas o objetivo n&atilde;o &eacute; falar sobre o _rvm_. Serve apenas como uma 
introdu&ccedil;&atilde;o, pois ao come&ccedil;ar a desenvolver em python, por causa do meu
trabalho atualmente, eu senti falta de uma ferramenta semelhante ao _rvm_.
No ecossistema _python_, a ferramenta mais pr&oacute;xima que eu encontrei,
pelo menos em rela&ccedil;&atilde;o &agrave;s _gemsets_, se chama
[virtualenv](http://www.virtualenv.org/en/latest/index.html). Apesar de n&atilde;o
ter exatamente o mesmo _modus operandi_, o _virtualenv_ tamb&eacute;m serve
para isolar o ambiente de desenvolvimento do resto do sistema.

### Por que usar o virtualenv? ###

Voc&ecirc; deve estar pensando: qual &eacute; a vantagem em ter um ambiente de 
desenvolvimento isolado? Tente imaginar a seguinte situa&ccedil;&atilde;o, voc&ecirc;
est&aacute; fazendo a manuten&ccedil;&atilde;o de um sistema que usa Django na vers&atilde;o 1.2.3,
por&eacute;m gostaria de iniciar um novo projeto (ou apenas fazer alguns
testes) na vers&atilde;o mais nova, a 1.3. Caso voc&ecirc; instale a nova vers&atilde;o 
globalmente, poder&aacute; ter conflitos no sistema anterior.

O ideal &eacute; que cada m&oacute;dulo seja instalado individualmente por projeto, dessa
forma diferentes vers&otilde;es n&atilde;o interferem em diferentes aplica&ccedil;&otilde;es.

### Instalando o virtualenv ###

A maneira mais f&aacute;cil de instalar o _virtualenv_ &eacute; utilizando o 
`easy_install`:

    $ easy_install virtualenv

Voc&ecirc; precisar&aacute; utilizar `sudo` caso precise de direitos administrativos
para instalar o _virtualenv_.
 
### Uso b&aacute;sico do virtualenv ###

Ap&oacute;s devidamente instalado, voc&ecirc; deve chamar o `virtualenv` passando como
par&acirc;metro o nome do ambiente a ser criado. Veja o exemplo:

    $ virtualenv environment

O comando acima vai criar uma nova pasta chamada `environment`. Vamos dar
uma olhada nesta pasta:

    $ cd environment
    $ ls
    bin include lib

Dentro da pasta, h&aacute; tr&ecirc;s outras. A pasta `bin` cont&eacute;m os execut&aacute;veis do
novo ambiente, caso voc&ecirc; d&ecirc; uma olhada, ver&aacute; `python`, `easy_install` e o
`pip`. Em vez de usar o comando `python` do sistema, voc&ecirc; deve passar a usar
o execut&aacute;vel presente nesta pasta, do mesmo jeito que o `easy_install` para
instalar novos m&oacute;dulos. Por exemplo, para instalar a vers&atilde;o 1.3 do Django:

    $ bin/easy_install Django==1.3

Depois disso, o Django ser&aacute; instalado na pasta `lib` do ambiente criado,
mais especificamente na pasta `lib/pythonX.X/site-packages`. Como o Django
tamb&eacute;m inclui um execut&aacute;vel para criar projetos e rodar scripts, na pasta
`bin` tamb&eacute;m ser&aacute; instalado o arquivo `django-admin.py`.

A pasta `lib`, como voc&ecirc; j&aacute; pode ter deduzido pelo par&aacute;grafo acima, guarda
os m&oacute;dulos (arquivos e links simb&oacute;licos). A pasta `include` inclue os 
_headers_ para os casos em que s&atilde;o instalados m&oacute;dulos com extens&otilde;es em
C, que precisam ser compilados.

Talvez voc&ecirc; ache meio chato ter que se lembrar de usar o caminho relativo
toda vez que for chamar o `python` ou o `easy_install`. Para isso o 
_virtualenv_ inclue uma conveni&ecirc;ncia. Na pasta `bin` h&aacute; um arquivo chamado
`activate` (para quem usa Windows, o `activate.bat`). Este arquivo 
altera o PATH do sistema, dessa forma toda vez que voc&ecirc; chamar o `python` 
ou `easy_install` pelo terminal, estar&aacute; se referindo aos bin&aacute;rios do 
ambiente criado pelo _ virtualenv_. Veja abaixo:

    $ source bin/activate

Para diminuir a confus&atilde;o, o `activate` por padr&atilde;o tamb&eacute;m altera o prompt
para incluir o nome do ambiente entre par&ecirc;nteses. Voc&ecirc; ter&aacute; algo parecido
com o exemplo abaixo:

    (environment) $ which python
    CAMINHO_QUALQUER/environment/bin/python

O `activate` apenas altera o o PATH e o _prompt_ do terminal ativo, caso 
voc&ecirc; queira usar outro terminal, ter&aacute; que rodar o `activate` no novo 
terminal tamb&eacute;m. Caso voc&ecirc; queira desfazer as altera&ccedil;&otilde;es do `activate` 
basta executar o seguinte comando:

    (environment) $ deactivate
    $ which python
    /usr/bin/python # talvez seja outro caminho

Dessa forma, o _prompt_ e o PATH s&atilde;o restaurados e tudo volta a ser como
antes.
 
### Como ter mais isolamento? ###

Uma boa pr&aacute;tica ao criar ambientes com o `virtualenv` &eacute; adicionar a op&ccedil;&atilde;o
_--no-site-packages_ como no exemplo abaixo:

    $ virtualenv --no-site-packages environment

Isto aumenta ainda mais o n&iacute;vel de isolamento, pois o _virtualenv_ n&atilde;o ir&aacute;
incluir os links simb&oacute;licos para o `site-packages` global, isolando
ainda mais o ambiente criado do restante do sistema.

