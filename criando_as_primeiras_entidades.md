#Tutorial SimPy: criando as primeiras entidades


Algo elementar em qualquer pacote de simulação é uma função para criar entidades dentro do modelo. É o [“Alô mundo!”](http://pt.wikipedia.org/wiki/Programa_Ol%C3%A1_Mundo) dos pacotes de simulação.

Inicialmente, serão necessárias duas bibliotecas do Python: random – biblioteca de geração de números aleatórios – e o próprio SimPy.

Começaremos nosso primeiro programa em SimPy chamando as bibliotecas de interesse (adicionalmente, existe uma chamada para a future, mas isso é apenas para manter a função print, compatível com o Python 3):
```
# -*- coding: utf-8 -*-
from __future__ import print_function # para compatibilidade da função print com o Python 3
import random # gerador de números aleatórios
import simpy # biblioteca de simulação```

Tudo no SimPy gira em torno de **processos** criados pelo usuário e todos os processos vivem num environment, um “ambiente” de simulação. O programa principal começa com uma chamada ao SimPy, criando um *environment*  “env”:

```
# -*- coding: utf-8 -*-
from __future__ import print_function # para compatibilidade da função print com o Python 3
import random # gerador de números aleatórios
import simpy # biblioteca de simulação

env = simpy.Environment() # cria o environment do modelo
```
Se você executar o programa agora, nada acontece. No momento, você apenas criou um environment, mas não criou nenhum processo, portanto, nada acontece.

Vamos escrever uma função ```
geraChegadas```
 que cria entidades no sistema enquanto durar a simulação.

Inicialmente, precisamos de gerar intervalos de tempos aleatórios, exponencialmente distribuídos, para representar os tempos entre chegadas sucessivas das entidades. Para gerar chegadas com intervalos exponenciais, a biblioteca random, bem detalhada na [documentação](https://docs.python.org/2/library/random.html), possui a função:
```
random.expovariate(lambd)```

Onde ```
lambd```
 é a taxa de surgimento dos eventos ou, matematicamente, o inverso do tempo médio entre eventos sucessivos. No caso, se eu quero que as chegadas sejam entre intervalos de 2 min, a função ficaria:
```
random.expovariate (1/2)```

Temos agora um gerador de números aleatórios. Falta informar ao SimPy que queremos nossas entidades segundo essa distribuição. Isso é feito pela chamada da palavra reservada ```
yield```
 com a função do SimPy ```
env.timeout```, que nada mais é do que uma função que causa um atraso de tempo, um *delay* do tempo fornecido para a função no *environmet* ```
env```
 criado:

```
yield env.timeout(random.expovariate (1/2))
```

Ou seja: estamos chamando a função ```
yield env.timeout```
 para que ela retarde o processo num tempo aleatório gerado pela função random.expovariate.

Colocando tudo junto na função ```
geraChegadas```
e lembrando que temos de passar ```
env```
 como argumento da função, temos:
```
# -*- coding: utf-8 -*-
from __future__ import print_function # para compatibilidade da função print com o Python 3
import random # gerador de números aleatórios
import simpy  # biblioteca de simulação

def criaChegadas(env):
    #função que cria chegadas de entidades no sistema
    contaChegada = 0
    while True:
        yield env.timeout(random.expovariate(1/2))
        contaChegada += 1
        print("Cliente %i chega em: %.1f " % (contaChegada, env.now))

env = simpy.Environment() # cria o environment do modelo```

O código deve ser autoexplicativo. Note apenas, que dentro do print, existe uma chamada para a **hora atual de simulação** ```
env.now```.

Se você executar, nada acontece, pois falta chamarmos a função e informarmos ao SimPy qual o tempo de simulação. A chamada da função nos relembra que tudo em SimPy é gerar processos:

```
# -*- coding: utf-8 -*-
from __future__ import print_function # para compatibilidade da função print com o Python 3
import random # gerador de números aleatórios
import simpy  # biblioteca de simulação

def criaChegadas(env):
    #função que cria chegadas de entidades no sistema
    contaChegada = 0
    while True:
        yield env.timeout(random.expovariate(1/2))
        contaChegada += 1
        print("Cliente %i chega em: %.1f " % (contaChegada, env.now))

random.seed(1000)   # semente do gerador de números aleatórios
env = simpy.Environment() # cria o environment do modelo
env.process(criaChegadas(env))
env.run(until=10)
```

Agora sim!

## Conteúdos desta seção
| Conteúdo | Descrição |
| -- | -- |
| env = simpy.Environment() | cria um *environment* de simulação |
| random.expovariate | gera números aleatórios exponencialmente distribuidos |
| yield env.timeout(time) | gera um atraso dado por *time* |
| random.seed(seed) | define o gerador de sementes aleatórias para um mesmo valor a cada nova simulação |
| env.process(criaChegadas(env) | inicia a função criaChegadas como um *processo* em env |
| env.run(until=tempoSim) | executa a simulação pelo tempo tempoSim |

## Desafios
**Desafio 2:** é comum que os comandos de criação de entidades nos softwares proprietários tenham a opção de limitar o número máximo de entidades geradas durante a simulação. 
Modifique a função ```
geraChegadas```
 de modo que ela receba como parâmetro o ```
numeroMaxChegadas```
 e limite a criação de entidades a este número. (Solução no próximo post).

**Desafio 3:** modifique a função ```
geraChegadas```
 de modo que as chegadas entre entidades sejam distribuídas segundo uma triangular de moda 1, menor valor 0,1 e maior valor 1,1.



