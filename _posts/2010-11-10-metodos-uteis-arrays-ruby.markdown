Para estrear o blog, vamos com um post bem simples e que serve de introdu&ccedil;&atilde;o
para quem est&aacute; come&ccedil;ando com *ruby*. Ao aprender uma nova linguagem, 
temos a tend&ecirc;ncia de escrever o c&oacute;digo da mesma maneira que escrevemos na 
linguagem que j&aacute; estamos habituados.

Por exemplo, algu&eacute;m que esteja aprendendo *ruby*, e que precise iterar 
em algum *array*, provavelmente ir&aacute; fazer algo parecido com isso:

    #!ruby
    bandas = [ "Led Zeppelin", "The Who", "Grand Funk Railroad" ]
    for banda in bandas
        puts "Eu gosto de #{banda}"
    end

N&atilde;o h&aacute; nada de errado nesse exemplo (talvez voc&ecirc; n&atilde;o compartilhe do mesmo gosto
musical, mas a&iacute; o problema &eacute; outro), ele funciona exatamente da maneira esperada.    
Esse estilo &eacute; o mais parecido com as outras linguagens, praticamente toda 
linguagem possui um la&ccedil;o **for**. Por&eacute;m, essa n&atilde;o &eacute; a maneira mais "ruby" 
de se iterar em um array.
    
    #!ruby
    bandas = [ "Led Zeppelin", "The Who", "Grand Funk Railroad" ]
    bandas.each { |banda| puts "Eu gosto de #{banda}" }
    # ou de outra forma
    bandas.each do |banda|
        puts "Eu gosto de #{banda}"
    end

Agora temos um c&oacute;digo que faz exatamente o mesmo que o exemplo anterior.
A diferen&ccedil;a fica por conta do m&eacute;todo **each**, que recebe um bloco. Esse m&eacute;todo
ir&aacute; iterar em cada elemento do array, executando o c&oacute;digo do bloco que passamos.
Podemos dizer que essa &eacute; a maneira mais "ruby" de se iterar em uma cole&ccedil;&atilde;o.

O m&eacute;todo **each** n&atilde;o &eacute; o &uacute;nico que temos a nossa disposi&ccedil;&atilde;o para iterar em um
array. Dependendo da l&oacute;gica que vamos realizar, existem outros m&eacute;todos que
podem ser mais adequados.

Outro exemplo, digamos que voc&ecirc; precisa selecionar os elementos de um array
dada uma certa condi&ccedil;&atilde;o, como se fosse uma esp&eacute;cie de filtro. Novamente, se voc&ecirc; 
est&aacute; acostumado com outras linguagens, &eacute; poss&iacute;vel que escreva algo parecido
com o exemplo abaixo.

    #!ruby
    bandas = [ "The Beatles", "The Animals", "Guess Who" ]
    bandas_com_the = []
    for banda in bandas
        if banda.start_with?("The")
            bandas_com_the &lt;&lt; banda
        end
    end

De novo, temos um **for** para iterar no array, e tamb&eacute;m temos um **if** para
selecionar os elementos corretos e adicion&aacute;-los no array resultante. At&eacute; esse
ponto, nenhum segredo. Nesse caso poder&iacute;amos tamb&eacute;m substituir o **for** pelo
m&eacute;todo **each** e deixar com mais cara de "ruby". Mas nesse caso, quem se
encaixa melhor &eacute; o m&eacute;todo **select**.

    #!ruby
    bandas = [ "The Beatles", "The Animals", "Guess Who" ]
    bandas_com_the = bandas.select { |banda| banda.start_with?("The") }

O resultado &eacute; o mesmo que o c&oacute;digo anterior, apenas com um c&oacute;digo bem
mais enxuto. O bloco que passamos para o m&eacute;todo **select** deve retornar
*true* ou *false*, e dependendo do que ele retorna, o elemento &eacute; adicionado
ou n&atilde;o ao resultado. Tamb&eacute;m existe o m&eacute;todo **reject** que faz o inverso, ou
seja, se o bloco retorna *true* o elemento n&atilde;o &eacute; adicionado ao resultado.

    #!ruby
    bandas_sem_the = bandas.reject { |banda| banda.start_with?("The") }

Imagine agora que voc&ecirc; precisa alterar todos os elementos do array, seguindo uma mesma
regra. Exemplo: adicionar o prefixo "The " ao nome de cada banda presente no *array*.

    #!ruby
    bandas = [ "Beatles", "Who", "Clash" ]
    bandas_com_the = []
    for banda in bandas
        bandas_com_the &lt;&lt; "The " + banda
    end

Mais uma vez, um exemplo que funciona corretamente, mas n&atilde;o muito "ruby". Nesse 
caso, o m&eacute;todo mais indicado &eacute; o **map**.

    #!ruby
    bandas = [ "Beatles" , "Who", "Clash" ]
    bandas_com_the = bandas.map { |banda| "The " + banda }

Cada elemento do array vai ser "mapeado" num novo elemento do array 
resultante, usando o retorno do bloco que passamos. Se voc&ecirc; j&aacute; viu um
exemplo parecido, mas com o m&eacute;todo **collect**, n&atilde;o estranhe. Os dois
m&eacute;todos fazem exatamente a mesma coisa.

Para o &uacute;ltimo exemplo do post, imagine agora que voc&ecirc; precisa somar todos
os elementos de um array. Primeiro, a maneira habitual:

    #!ruby
    numeros = [ 1, 2, 4, 8 ]
    soma = 0
    for n in numeros
        soma += n
    end
    
Depois, a maneira "ruby", utilizando o m&eacute;todo **reduce**:

    #!ruby
    numeros = [ 1, 2, 4, 8 ]
    soma = numeros.reduce { |soma, n| soma += n }

O **reduce** &eacute; utilizado para fazer uma agrega&ccedil;&atilde;o. Ele &eacute; diferente dos
outros m&eacute;todos, pois o bloco que passamos para ele deve receber dois par&acirc;metros.
O primeiro &eacute; o valor que vai ser "agregado", enquanto o segundo &eacute; o elemento atual 
do array. No exemplo, fiz uma soma, mas podia ter sido qualquer outra opera&ccedil;&atilde;o, como
multipli&ccedil;&atilde;o, divis&atilde;o, etc. Para quem j&aacute; viu algo parecido, mas com o m&eacute;todo
**inject**, saiba que s&atilde;o a mesma coisa.

Nem precisamos nos restringir somente a n&uacute;meros, podemos fazer uma agrega&ccedil;&atilde;o 
em strings tamb&eacute;m:

    #!ruby
    ccr = [ "Creedence", "Clearwater", "Revival" ]
    ccr = ccr.reduce { |soma, n| soma += " " + n }
    
Claro que no exemplo acima &eacute; bem mais simples utilizar o **join**, essa n&atilde;o &eacute; a
maneira recomendada. Eu queria apenas ilustrar o exemplo de que o **reduce** n&atilde;o
&eacute; restrito &agrave; opera&ccedil;&otilde;es com n&uacute;meros.
