> Prefira composição à herança.

Essa frase é um dos mantras da orientação a objetos. Me lembro de ter
visto este príncipio
pela primeira vez no conhecido _Design Patterns_ do _Gang of Four_.
Também é fonte de
confusão principalmente para iniciantes que podem achar que utilizar
herança é ruim, quando
simplesmente não é a melhor solução para determinados casos. De todo
jeito, não quero
discutir quando ou não usar herança, e sim apenas mostrar uma maneira de
(talvez) facilitar a composição em _ruby_.

=== Forwarding ===

Ao utilizar composição, em geral utilizamos uma técnica chamada
_forwarding_, isto é,
repassamos a chamada de um método para um outro objeto (o que foi
"composto") que vai tratar essa chamada.
É a mesma coisa que dizer que vamos delegar uma função para esse
objeto, por isso também
é bem comum se referirem à essa técnica como _delegation_.

Veja um exemplo abaixo de _forwarding_:

   #!ruby
   class Motor
     def liga
       puts "Ligando o motor."
     end

     def desliga
       puts "Desligando o motor."
     end
   end

   class Carro
     def initialize
       @motor = Motor.new
     end

     def liga_motor
       # delegamos a tarefa de ligar para o motor
       @motor.liga
     end

     def desliga_motor
       # delegamos a tarefa de desligar para o motor
       @motor.desliga
     end
   end

   c = Carro.new
   c.liga_motor
   # Ligando o motor.
   c.desliga_motor
   # Desligando o motor.

Esse exemplo é bem simples e apesar de não ser muito útil, serve para
ilustrar o conceito.
Compomos o objeto motor com o carro, e as chamadas dos métodos
*liga_motor* e *desliga_motor* são
encaminhadas para o motor "interno" ao carro. Não há nada de errado
com este exemplo, porém
podemos ter algumas inconveniências. A classe Motor tinha apenas dois
métodos, por isso não
dá muito trabalho escrever dois métodos na classe Carro que delegam a
responsabilidades ao
motor. Mas e se o Motor tivesse algumas dezenas de métodos? Teríamos
que escrever um monte
de métodos só para isso?

=== Usando o módulo Forwardable ===

Felizmente não. Em _ruby_ temos alguns atalhos quando queremos definir
_forwarding_, e para
isso precisamos do módulo **Forwardable**. Veja abaixo a classe Carro
refatorada para
utilizar este módulo:

   #!ruby
   require "forwardable"

   class Carro
     extend Forwardable

     def initialize
       @motor = Motor.new
     end

     def_delegator :@motor, :liga, :liga_motor

     def_delegator :@motor, :desliga, :desliga_motor
   end

   c = Carro.new
   c.liga_motor
   # Ligando o motor.
   c.desliga_motor
   # Desligando o motor.


O truque fica por conta do método **def_delegator**. Ele recebe até 3
parâmetros.
O primeiro é indicando o objeto para o qual vai ser encaminhado,
o segundo indica qual o método deve ser chamado, e o terceiro
(opcional) indica o _alias_
do método. Se não for especificado nenhum _alias_, o nome do método é
o mesmo que o
do objeto composto.

Caso você também não conheça, o **extend** é um método que pertence à
classe _Object_, logo
está disponível para todos os objetos. O que o _extend_ faz é incluir os métodos
do módulo _Forwardable_ na definição da nossa classe. Note que é
diferente de usar
**include**. O _include_ incluí os métodos do módulo para as
instâncias dos nossos objetos,
enquanto o _extend_ faz isso no nível da nossa classe, incluindo como
_singleton_.

E se tivermos muitos métodos? Não precisamos definir cada método
através do *def_delegator*.
Se quisermos definir vários métodos de uma vez, podemos usar o método
*def_delegators*, que recebe
uma lista com o nome de todos os métodos que vamos fazer _forwarding_.
Neste caso,
não podemos definir um _alias_ para os métodos.

   #!ruby
   require "forwardable"

   class Carro
     extend Forwardable

     def initialize
       @motor = Motor.new
     end

     def_delegators :@motor, :liga, :desliga
   end

   c = Carro.new
   c.liga
   # Ligando o motor.
   c.desliga
   # Desligando o motor.


E caso a gente queira mudar os nomes dos métodos? Bem, sempre podemos
fazer alguns truques.
Se os nomes que vão ser sobreescritos seguirem um determinado padrão,
ainda podemos usar
o _def_delegator_, e colocá-lo dentro de um loop. Veja o exemplo:

   #!ruby
   require "forwardable"

   class Carro
     extend Forwardable

     def initialize
       @motor = Motor.new
     end

     ['liga', 'desliga'].each do |nome|
       def_delegator :@motor, nome, nome.to_s + '_motor'
     end
   end

   c = Carro.new
   c.liga_motor
   # Ligando o motor.
   c.desliga_motor
   # Desligando o motor.

Nesse caso colocamos no array todos os métodos que vamos realizar
_forwarding_, e
informamos o _alias_ dentro do loop. Isso é claro se todos os novos
métodos seguirem
o mesmo padrão no nome. No nosso caso, o padrão é concatenar com o
sufixo "_motor", resultando
em _liga_motor_ e _desliga_motor_. Claro que se não houver um padrão a
ser seguido, não há
muita coisa a ser feita.

