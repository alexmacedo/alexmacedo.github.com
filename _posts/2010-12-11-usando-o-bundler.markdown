---
layout: post
title: Usando o Bundler
---

O [Bundler](http://gembundler.com/) é uma _gem_ com o propósito de
resolver facilmente
dependências em projetos _ruby_. Quem já criou um projeto utilizando a terceira
versão do _Rails_, já deve ter usado o _bundler_, pois ele é usado
como padrão para
gerenciar as dependências de gems. Porém o _bundler_ não está restrito
a projetos
com _Rails_, na verdade ele pode (e deve) ser usado com qualquer
projeto em _ruby_.

### O problema 

A comunidade _ruby_ é muito ágil e prolífica, o que gera uma grande quantidade
de _gems_ que são atualizadas constantemente. Isso é muito bom em geral, porém
traz também alguns problemas. Como as _gems_ mudam constantemente de
versão, você
não tem muitas garantias que o que estava funcionando em uma versão antiga vai
continuar funcionando na versão nova sem nenhum tipo de refatoração. Para se ter
certeza de que o seu script/aplicação vai funcionar como esperado, o indicado é
que se use o mesmo conjunto de _gems_. O problema é ainda maior se existem mais
desenvolvedores no projeto. Como cada um tem um ambiente diferente, pode ser
que cada um tenha instalado uma versão da _gem_ diferente.

Uma outra desvantagem (que na verdade é somente uma inconveniência) é o trabalho
entediante de instalar as gems necessárias para um projeto. Imagine
que você acabou
de clonar um repositório, porém o projeto utiliza várias gems que você
não tem instalado no seu ambiente. Você precisa descobrir primeiro
quais são as dependências,
para depois instalar as que são necessárias.

### Como o _bundler_ resolve o problema? 

O _bundler_ resolve o problema "registrando" todas
as dependências em um arquivo chamado **Gemfile**, que fica na pasta
raiz do projeto.
Para utilizar o _bundler_, precisamos antes instalar a sua gem:

   $ gem install bundler

Note que eu não usei _sudo_, porque estou usando o _rvm_ (talvez eu fale sobre o
_rvm_ no futuro). Caso você não utilize o _rvm_, precisará incluir o _sudo_. Com
o _bundler_ instalado, agora precisamos apenas escrever o arquivo _Gemfile_
(dica: você pode usar o comando _bundle init_ para gerar um Gemfile).
Suponha que vamos criar uma aplicação usando o _sinatra_ e mais algumas outras
gems.

   {% highlight ruby %}
   source "http://rubygems.org"

   gem "sinatra", ">= 1.0"
   gem "json", "1.4.6"
   gem "haml"
   gem "rack-contrib", :require => "rack/contrib"

   group :test do
     gem "rack-test", :require => "rack/test"
   end
   {% endhighlight %}

Dentro da pasta com o _Gemfile_ basta executar:

   $ bundle install

E pronto! Serão instaladas todas as gems que você não possui no sistema. Repare
que também não incluí _sudo_ no comando, e nesse caso não é
recomendado, independente
de utilizar ou não o _rvm_. Sempre use **bundle install** e não **sudo
bundle install**.
Se for necessário
a senha de administrador para instalar alguma gem, o bundle irá pedir
a senha através
do prompt. Após a execução do comando, a saída vai ser algo do tipo:

   Fetching source index for http://rubygems.org/
   Using haml (3.0.24)
   Using json (1.4.6)
   Using rack (1.2.1)
   Installing rack-contrib (1.1.0)
   Using rack-test (0.5.6)
   Using tilt (1.1)
   Using sinatra (1.1.0)
   Using bundler (1.0.7)
   Your bundle is complete! It was installed into #{pasta do seu sistema}

Veja que existem mais gems do que aquelas que especificamos. É porque uma gem
pode ter suas próprias dependências, e elas também precisam ser instaladas. No
caso veja que a única gem que foi instalada foi a _rack-contrib_,
todas as outras
já estavam instaladas no sistema.

Após a execução do _bundle install_, surge um arquivo chamado
**Gemfile.lock** na
mesma pasta. No caso acima, ele ficou com o seguinte conteúdo:

   GEM
     remote: http://rubygems.org/
     specs:
       haml (3.0.24)
       json (1.4.6)
       rack (1.2.1)
       rack-contrib (1.1.0)
         rack (>= 0.9.1)
       rack-test (0.5.6)
         rack (>= 1.0)
       sinatra (1.1.0)
         rack (~> 1.1)
         tilt (~> 1.1)
       tilt (1.1)

   PLATFORMS
     ruby

   DEPENDENCIES
     haml
     json (= 1.4.6)
     rack-contrib
     rack-test
     sinatra (>= 1.0)

Veja que ele possue praticamente as mesmas informações que o Gemfile,
organizadas
de uma outra maneira. Agora duas observações IMPORTANTES:

- não edite ou remova o **Gemfile.lock**. Ele nunca deve ser alterado
pelo usuário,
 apenas pelo próprio _bundle_. Se você precisar adicionar uma nova
gem, por exemplo,
 você deve editar o _Gemfile_ e rodar novamente o comando _bundle
install_. Isso é
 importante para manter a consistência do projeto.

- sempre inclua o _Gemfile.lock_ no seu controle de versão. Pode parecer
 estranho, você deve pensar que o _Gemfile_ seria o suficiente, mas o
_Gemfile.lock_ é
 igualmente importante (ou mais). No exemplo acima, você deve ter
notado que eu não
 especifiquei nenhuma versão para a gem _haml_. Por isso, o bundler utilizou a
 versão mais recente disponível, no caso a 3.0.24, e guardou essa
informação no _Gemfile.lock_.
 Se alguém clonar o projeto, e rodar _bundle install_, será utilizada
a versão informada
 no _Gemfile.lock_, mesmo que exista uma versão mais recente
disponível. Caso não incluíssemos
 o _lock_ no repositório, isso geraria uma inconsistência, com
projetos utilizando versões
 de gems diferentes. O _Gemfile.lock_ é a garantia de que todos estão
utilizando sempre
 a mesma versão de uma gem.

### Explicando o Gemfile 

Vamos ver em detalhes o _Gemfile_. No início temos o _source_,
informando a url que será
usada para as gems que precisam ser instaladas. Depois para cada
dependência, usamos:

   {% highlight ruby %}
   gem "nome", "versão"
   {% endhighlight %}

Note que ao informar a versão, podemos utilizar vários modificadores.
Por exemplo ">= 1.0",
"< 2.3", são auto-explicativos. Um modificador que pode confundir é o "~>". Se
temos algo como "~> 2.3.2", significa exatamente ">= 2.3.2" e "< 2.4". Se temos
algo como "~> 1.1" é idêntico a ">= 1.1" e "< 1.2". Veja também que a
versão é opcional,
e se não for especificada, o _bundler_ vai utilizar a versão mais
recente disponível.

Você também deve ter estranhado o _:require_, como no exemplo abaixo:

   {% highlight ruby %}
   gem "rack-contrib", :require => "rack/contrib"
   {% endhighlight %}

Isso é porque podemos (na verdade devemos) utilizar o bundler para adicionar as
gems no nosso projeto. Veja o exemplo abaixo:

   {% highlight ruby %}
   require "rubygems"
   require "bundler/setup"

   # require em outras gems
   require "sinatra"
   require "rack/contrib"
   # etc
   {% endhighlight %}

Note que primeiro devemos incluir o "bundler/setup" antes das demais gems, pois
ele que irá se encarregar de utilizar as versões corretas. Outra
maneira de atingir
os mesmos resultados seria:

   {% highlight ruby %}
   require "rubygems"
   require "bundler"

   Bundler.require

   # codigo
   {% endhighlight %}

Chamar o _Bundler.require_ vai incluir todas as gems especificadas no _Gemfile_.
O problema é que no _Gemfile_ o nome da gem é _rack-contrib_, o que faria o
_bundler_ tentar fazer o seguinte:

   {% highlight ruby %}
   require "rack-contrib" # o correto é "rack/contrib"
   {% endhighlight %}

Por isso precisamos indicar o nome a ser usado na hora em que a gem
for adicionada
ao projeto, através da opção _:require_.

Outra coisa interessante é que podemos agrupar as gems, como no caso:

   {% highlight ruby %}
   group :test do
     gem "rack-test", :require => "rack/test"
   end
   {% endhighlight %}

O nome do grupo pode ser qualquer um, mas o mais comum será nomes como
**:test**, **:production**
e **:development**, indicando o ambiente. A vantagem de definir grupos
é que podemos
restringir quais gems serão instaladas. Se estamos em um ambiente de
produção, não
estamos interessados em testes, então podemos fazer:

   $ bundle install --without test

Dessa forma, as gems relacionadas no grupo _:test_ serão ignoradas. No nosso
código também podemos utilizar os grupos da seguinte maneira:

   {% highlight ruby %}
   Bundler.require(:development)
   {% endhighlight %}

Para adicionar apenas as gems que estão no grupo _:development_.

### Atualizando uma gem

Se você quiser atualizar uma gem, precisa mudar a versão no _Gemfile_ e rodar
novamente o comando _bundle install_ que o _bundler_ irá resolver as novas
dependências e alterar o _Gemfile.lock_ corretamente. O _bundler_ não irá tentar
atualizar gems na qual você especificou uma versão, e que precisem ser
atualizadas
para satisfazer uma outra dependência. Nesse caso, ele irá mostrar pra você onde
ocorre o conflito, e caberá a você arrumar os conflitos no _Gemfile_, antes de
executar novamente o _bundle install_.

Porém também existem as gems na qual você não especificou uma versão. No nosso
exemplo, não especificamos nenhuma versão para a gem _haml_, e nem queremos
especificar. Porém o _Gemfile.lock_ sempre irá utilizar a versão 3.0.24, mesmo
que seja lançada depois uma nova versão. Nesse caso que queremos utilizar sempre
a versão mais nova de uma gem, existe o comando:

   $ bundle update GEM

Esse comando irá atualizar para a versão mais nova da GEM, e atualizar
corretamente
o _Gemfile.lock_ para utilizar essa versão. Novamente, o comando irá mostrar um
conflito caso não seja possível atualizar a gem.

### Deploy

Um último comando que pode ser muito interessante quando for um projeto web, e
estamos no ambiente de produção:

   $ bundle install --deployment

A flag --deployment irá mudar o comportamento habitual do _bundle
install_. Entre as
mudanças, temos:

- o comando só irá rodar se existir o _Gemfile.lock_, do contrário mostrará uma
 mensagem de erro;
- o _bundler_ não irá atualizar o _Gemfile.lock_ caso ele não esteja
sincronizado com
 o _Gemfile_.
- as gems serão instaladas na pasta _vendor/cache_ dentro da
aplicação, e não no sistema;
- só serão utilizadas as gems presentes em _vendor/cache_, mesmo que a
gem esteja disponível
 no sistema;

### Concluindo

O _bundler_ é uma ferramenta que se tornou um padrão para projetos em
_ruby_. Por isso,
independente de desenvolver um projeto Rails, uma nova Gem ou um
simples script, é bom
utilizá-lo, principalmente quando houver vários desenvolvedores envolvidos.

### Maiores referências

- [Site oficial do Bundler](http://gembundler.com/)
- [Screencast sobre o
bundler](http://railscasts.com/episodes/201-bundler) (um pouco
desatualizado)
