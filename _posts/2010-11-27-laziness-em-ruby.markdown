Nos últimos posts tenho falado sobre programação funcional. Talvez
seja o momento
de mostrar algo útil, apenas para não parecer "isso é legal, mas nunca
vou utilizar na vida real".
Este exemplo foi feito por [Paul
Cantrell](http://innig.net/software/ruby/closures-in-ruby.rb)
(vale a pena dar uma olhada). Vou mostrar como utilizar as _closures_
que vimos para
implementar o conceito de _lazyness_.

### O que é lazyness? ###
Para variar, veja o [artigo](http://en.wikipedia.org/wiki/Lazy_evaluation) na
Wikipedia para maior referência. Explicando de maneira rápida, _lazy
evaluation_ é
adiar uma instrução/computação/execução até o momento em que o resultado é
requerido.

### E qual a utilidade? ###
Imagine que você precisa instanciar um objeto que vai consumir
bastante memória, por exemplo.
Ao utilizar _lazy loading_ (mesma coisa que _lazy evaluation_) você
pode "atrasar" esse
consumo de memória, deixando apenas para o momento que você precisar
do resultado. E mais,
caso você nem utilize o resultado, você acaba consumindo nenhum
recurso desnecessário, apesar
de supostamente ter "criado" o seu objeto. De certa forma, isto é o
que frameworks como o
Hibernate (para Java) e o Entity Framework (para C#) fazem para
prevenir chamadas desnecessárias
ao banco de dados.

### Implementando o _lazy loading_ ###
Vamos pensar na nossa implementação. Iremos criar um objeto **Lazy**, que vai
funcionar parecido com um
[proxy](http://en.wikipedia.org/wiki/Proxy_pattern) para
o nosso objeto real. A diferença é que o objeto Lazy só vai
executar/acessar o objeto
real quando for necessário, e pra isso vamos usar _closures_.

   #!ruby
   class LazyProc
     def initialize(&computation)
       @computation = computation
     end

     def method_missing(method, *args, &block)
       evaluate.send(method, *args, &block)
     end

     def evaluate
       @value = @computation.call unless @value
       @value
     end
   end

   def lazy(&block) # atalho para criar a proc
     LazyProc.new(&block)
   end

Vamos tentar entender o código acima. No **initialize** recebemos uma _Proc_ que
é guardada para execução posterior. Na primeira vez que for chamado um método,
a proc será executada e a chamada será encaminhada para o resultado da execução
desta proc. Vamos ver se funciona:

   #!ruby
   a = lazy { puts "Resolvendo..."; "Primeiro teste" }
   puts a.upcase
   # Resolvendo
   # PRIMEIRO TESTE
   # => nil

Parece que deu certo. Veja que aparece "Resolvendo", porque a nossa _Proc_ foi
executada apenas quando chamamos o método **upcase**. Mas será que não
existe nenhum
problema?
s
   #!ruby
   a = lazy { "Segundo teste" }
   a
   # => #<LazyProc:0x00000001082630>
   a.to_s
   # => #<LazyProc:0x00000001082630>
   a.respond_to?(:upcase)
   # => false

Quando chamamos o **to_s** ou o **respond_to?**
gostaríamos que a nossa proc fosse executada, mas isso não está acontecendo.
O problema é que a nossa classe **LazyProc** herda todos esses métodos
de **Object**.
Então como o método está presente, o **method_missing** não é chamado
nesses casos.
Nesse caso podemos sobrescrever estes métodos, e fazer a delegação diretamente.
Porém, devemos fazer isso para todos os métodos presentes em
**Object**. Uma solução
melhor, apesar de parecer radical, é remover todos os métodos na nossa classe
**LazyProc**. Vejamos como vai ficar:

   #!ruby
   class LazyProc
     # apagamos os métodos existentes
     instance_methods.each { |m| undef_method m unless m =~ /^__/ }

     def initialize(&computation)
       @computation = computation
     end

     def method_missing(method, *args, &block)
       evaluate.__send__(method, *args, &block)
     end

     def evaluate
       @value = @computation.call unless @value
       @value
     end
   end

Isso vai gerar um _warning_:

   warning: undefining `object_id' may cause serious problems

Mas por enquanto vamos ignorar isso. Veja que tem duas mudanças. A primeira é
que vamos eliminar a definição para os métodos de instância da nossa classe,
com exceção daqueles começados por dois _underlines_. A segunda é que mudamos
a chamada de **send** para **__send__**. Vamos à explicação. Você deve
se lembrar
que apagamos a definição do **send**, porém como ele é um método muito
importante,
por padrão é definido um _alias_, um apelido, que nesse caso é
**__send__**, assim
caso alguém sobrescreva o **send**, ainda pode utilizar o _alias_ para
se referir ao
método antigo. Vamos testar agora:

   #!ruby
   a = lazy { puts "Resolvendo..."; "Outro teste" }
   # Resolvendo
   # => "Outro teste"

Estranho! Só de executar a atribuição no irb, a nossa proc já foi
executada. Isso é a
mesma coisa que _eager evaluation_, não é o que a gente esperava. O que foi que
aconteceu? Vamos começar a debugar:

   #!ruby
   class LazyProc
     # apagamos os métodos existentes
     instance_methods.each { |m| undef_method m unless m =~ /^__/ }

     def initialize(&computation)
       @computation = computation
     end

     def method_missing(method, *args, &block)
       puts "chamando #{method}" # vamos descobrir que método está
sendo chamado
       evaluate.__send__(method, *args, &block)
     end

     def evaluate
       @value = @computation.call unless @value
       @value
     end
   end

E executando:

   #!ruby
   a = lazy { "Teste" }
   # chamando inspect
   # => "Teste"

Ah! A chamada para **inspect** está fazendo com que a nossa proc seja executada
logo de imediato. O jeito é definir o método **inspect** novamente. Além disso,
vamos definir também o método **respond_to?**:

   #!ruby
   class LazyProc
     alias __class__ class
     instance_methods.each { |m| undef_method m unless m =~ /^__/ or
m == :object_id }

     def initialize(&computation)
       @computation = computation
     end

     def inspect
       if @value
         @value.inspect
       else
         "#<#{__class__} computation=#{@computation.inspect}>"
       end
     end

     def respond_to?(method)
       method = method.to_sym
       method == :inspect or
       method == :evaluate or
       evaluate.respond_to?(method)
     end

     def method_missing(method, *args, &block)
       evaluate.__send__(method, *args, &block)
     end

     def evaluate
       @value = @computation.call unless @value
       @value
     end
   end

 Vamos ver quais foram as mudanças. Primeiro, criamos um _alias_ para
**class**, porque
 vamos utilizá-lo dentro do nosso **inspect**. Segundo, incluí o
**object_id** na
 exclusão dos métodos que serão removidos, apenas para que não seja
gerado o _warning_.
 Terceiro, o nosso **inspect** imprime a informação da **LazyProc**
caso não tenhamos
 executado ainda a nossa proc. Caso ela já tenha sido executada pelo
menos uma vez,
 o **inspect** é chamado pro resultado da execução. O **respond_to?** verifica
se o método chamado é o **inspect** ou o **evaluate**, do contrário passa a bola
para a nossa proc ser executada.

Será que funciona agora?

   #!ruby
   a = lazy { puts "Resolvendo..."; "Mais um teste" }
   # => #<LazyProc computation=#<Proc:0x0000000202eb38>
   a
   # => #<LazyProc computation=#<Proc:0x0000000202eb38>
   puts a
   # Resolvendo
   # Teste
   # => nil
   a
   # => "Teste"

Parece que funciona como planejado. Note que a primeira vez que digitamos "a"
no interpretador, aparece o **inspect** da **LazyProc**, enquanto na segunda
vez, é chamado o **inspect** da **String**. Isso ocorreu porque da segunda
vez a nossa proc já tinha sido executada.

### Exemplo mais elaborado ###

Antes eu disse que a vantagem de utilizar _lazy evaluation_ era para economizar
memória quando tivéssemos objetos muito grandes. Mas apenas dei exemplos com
strings simples, onde não há vantagem nenhuma utilizar _lazy evaluation_.
Então agora vou mostrar um exemplo mais elaborado. Você já deve ter visto a
sequência _Fibonacci_.

   1 1 2 3 5 8 13 21 ...

Cada número da sequência é formado pela soma dos dois números anteriores, sendo
que os dois primeiros números são 1. Vamos então criar uma estrutura de dados
que armazene a sequência _Fibonacci_. Mais do que isso, vamos criar
uma estrutura de
dados que armazene **todos** os números da sequência _Fibonacci_. Isso mesmo,
vamos criar uma lista infinita.

E como isso é possível? Como estamos utilizando _lazy evaluation_, cada
elemento só será criado quando for chamado, dessa forma não gastamos memória
desnecessariamente.

Vamos lá, primeiro vamos criar a nossa estrutura de dados que vai representar
a lista:

   #!ruby
   class LispyEnumerable
     include Enumerable

     def initialize(list)
       @list = list
     end

     def each
       while @list
         car,cdr = @list
         yield car
         @list = cdr
       end
     end
   end

Um pouco de contexto. O nome da classe é **LispyEnumerable** porque imita uma
lista em Lisp. Em Lisp, se tivermos uma lista, por exemplo, igual a (1 2 3 4),
temos que o car é 1, e o cdr é (2 3 4). O car é o primeiro elemento
da lista, enquanto o cdr é o restante, excluindo o primeiro.
Analisando o código,
o que fazemos é pegar o primeiro elemento da lista, e executar o bloco, enquanto
armazenamos o resto da lista para executar depois. Se o restante da lista for
uma **LazyProc**, já resolveu o nosso problema, pois ele só será executado na
próxima chamada, se houver. Dessa forma, podemos criar uma lista que dá apenas
a impressão de ser infinita, pois está sendo resolvida sobre demanda.

Agora só precisamos criar uma lista que tenha a estrutura desejada, ou seja,
tenha um primeiro elemento inteiro, e o restante da lista é uma **LazyProc**.
Vamos lá:

   #!ruby
   def fibo(a, b)
     lazy { [ a, fibo(b, a+b) ] } # => so nao temos recursao infinita
por causa do lazy
   end

Veja que a nossa lista é definida recursivamente. Se tivermos
paciência de executar
aos poucos teremos:

   lazy { [a, lazy { [b, lazy { [a+b, lazy { [ a+b+b, ...

Então veja que temos uma **LazyProc**, que ao ser executada, nos dará o primeiro
elemento da lista, e guarda o restante da lista em outra **LazyProc**,
que depois
vai ser executada, e assim por diante .É isso mesmo, temos um array,
dentro de um array,
dentro de um array, etc. Até parece Inception.

Vamos testar isso agora (cruzando os dedos):

   #!ruby
   fibo_list = LispyEnumerable.new(fibo(1,1))
   fibo_list.each do |i|
     puts i
     break if i > 100 # sem o break seria infinito
   end
   # 1
   # 1
   # 2
   # 3
   # 5
   # 8
   # 13
   # 21
   # 34
   # 55
   # 89
   # 144
   # => nil

Sim, funcionou. Pode não ser a maneira mais óbvia ou rápida de imprimir
os primeiros números da sequência _Fibonacci_, mas com certeza é uma das
mais legais. Isso se você considera uma maneira estranha como sendo legal.

### E agora? ###
Bem, se você quiser _lazyness_ em Ruby, você não precisa se dar a todo
esse trabalho,
porque já existem gems pra isso. Na verdade, a classe **LazyProc** é fortemente
inspirada na gem [lazy](http://moonbase.rydia.net/software/lazy.rb/). Para quem
quiser o código deste post, apenas para hackear e aprender, eu fiz um
[gist](https://gist.github.com/717937).
