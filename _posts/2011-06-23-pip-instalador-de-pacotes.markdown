---
layout: post
title: "Pip: instalador de pacotes"
---

No post anterior, falei sobre o *virtualenv*, por&eacute;m deixei de falar sobre o 
[pip](http://www.pip-installer.org/en/latest/index.html). Ao instalar o 
`virtualenv`, voc&ecirc; acaba instalando automaticamente o `pip`, que &eacute; um
instalador de pacotes ao estilo do `easy_install`, por&eacute;m com uma
abordagem mais moderna.

### pip + virtualenv 

O `virtualenv` e o `pip` andam sempre juntos. Ao criar um novo ambiente:

    $ virtualenv env

Voc&ecirc; j&aacute; ter&aacute; na pasta `env/bin` o execut&aacute;vel `pip` e poder&aacute; utilizar ele
ao inv&eacute;s do `easy_install` para instalar os pacotes necess&aacute;rios do novo
ambiente. Para instalar algum pacote com o pip, basta executar o comando:

    $ bin/pip install NomeDoPacote

Onde `NomeDoPacote` &eacute; um nome igual ao que se encontra no 
[PyPi](http://pypi.python.org/pypi). Note que no caso, estou usando o
execut&aacute;vel presente na pasta `env/bin`. Por&eacute;m, nada impede que voc&ecirc;
utilize o `pip` para instalar pacotes globalmente no sistema. Nesse
caso, ser&aacute; necess&aacute;rio que o `pip` esteja instalado globalmente no
sistema, o que pode ser feito com o `easy_install`:

    $ easy_install pip

### Vantagens do pip sobre o easy_install 

O `pip` permite desinstalar pacotes de maneira f&aacute;cil, ao contr&aacute;rio do
`easy_install`:

    $ pip uninstall NomeDoPacote

Outra grande vantagem &eacute; que o `pip` permite instalar pacotes `tar.gz` ou `zip`
diretamente de `urls`. Por exemplo, para instalar o cliente `gdata` das 
APIs do *Google*:

    $ bin/pip install http://gdata-python-client.googlecode.com/files/gdata-2.0.14.zip

E al&eacute;m disso, o `pip` possui suporte nativo a controladores de vers&atilde;o, como 
`git`, `mercurial` e `svn`. Dessa forma, podemos instalar um pacote diretamente
de um reposit&oacute;rio. Para instalar o pacote do exemplo anterior atrav&eacute;s do seu
reposit&oacute;rio, basta executar:

    $ bin/pip install hg+https://gdata-python-client.googlecode.com/hg/#egg=gdata-python-client

&Eacute; importante notar:
* O `hg+` no in&iacute;cio da `url` foi adicionado para indicar que &eacute; um reposit&oacute;rio 
`mercurial`. Caso 
fosse um reposit&oacute;rio `git`, seria `git+`, se fosse `subversion`, `svn+` e assim por
diante;

* O `#egg=nome-do-pacote` no final da `url` foi adicionado para indicar o nome 
com o qual o pacote ser&aacute; instalado, dessa forma o usu&aacute;rio pode escolher um 
nome que achar apropriado.

&Agrave;s vezes, pode ser que voc&ecirc; necessite fazer altera&ccedil;&otilde;es no c&oacute;digo do pacote 
que voc&ecirc; instalou de um reposit&oacute;rio. Nesse caso, &eacute; interessante
incluir a op&ccedil;&atilde;o `-e` para indicar que o m&oacute;dulo &eacute; edit&aacute;vel. Veja um exemplo
com outro pacote do Google, o `gviz_api` usado para facilitar
a cria&ccedil;&atilde;o de _DataSources_ ao usar o 
[Google Chart Tools](http://code.google.com/apis/chart/):

    $ bin/pip install -e svn+http://google-visualization-python.googlecode.com/svn/trunk/#egg=gviz

Ap&oacute;s o t&eacute;rmino do comando, teremos uma nova pasta `src` na raiz do ambiente.
Dentro desta pasta, vamos encontrar a pasta `gviz` que foi adicionada pelo comando
acima (lembre-se que o `#egg` indica o nome da pasta). Nesta pasta, fica
o c&oacute;digo fonte dos m&oacute;dulos, que podem ser editados conforme a necessidade.

### Requirements File

Outro ponto muito interessante do `pip` &eacute; a cria&ccedil;&atilde;o de _requirements files_.
&Eacute; uma boa maneira de indicar e instalar depend&ecirc;ncias de forma r&aacute;pida.
O comum &eacute; criar um arquivo chamado `requirements.txt` (que na verdade pode ter qualquer nome)
na raiz de um projeto, para indicar quais pacotes s&atilde;o obrigat&oacute;rios. O formato deste
arquivo &eacute; bem simples:

    Django==1.3
    South==0.7.3

&Eacute; apenas um arquivo texto plano, com o nome do pacote em cada linha. Apesar de n&atilde;o ser
obrigat&oacute;rio, &eacute; bom tamb&eacute;m indicar qual a vers&atilde;o, por isso coloquei o `==versao`.
Tamb&eacute;m pode se usar `&gt;=` para indicar um vers&atilde;o igual ou superior. Caso n&atilde;o seja detalhado
qual a vers&atilde;o, o `pip` sempre instalar&aacute; a vers&atilde;o mais recente dispon&iacute;vel. Um atalho para
criar um _requirement file_ &eacute; utilizar o comando:

    $ bin/pip freeze &gt; requirements.txt

O `pip` inclue todos os pacotes instalados no `environment` no arquivo 
`requirements.txt`. Voc&ecirc; pode editar depois este mesmo arquivo, caso queira
retirar alguma depend&ecirc;ncia manualmente.

Para instalar os pacotes descritos em um _requirement file_ &eacute; bem simples. Basta passar
o arquivo com a op&ccedil;&atilde;o `-r` para o `pip`:

    $ bin/pip install -r requirements.txt

Dessa forma, todos os pacotes presentes no arquivo ser&atilde;o instalados. Isto facilita
a cria&ccedil;&atilde;o de novos ambientes, e tamb&eacute;m ajuda a garantir que todos utilizem uma
mesma vers&atilde;o de um _framework_, por exemplo.

### Refer&ecirc;ncias

* [Site oficial](http://www.pip-installer.org/en/latest/index.html)
* [P&aacute;gina no PyPi](http://pypi.python.org/pypi/pip/1.0.1)

