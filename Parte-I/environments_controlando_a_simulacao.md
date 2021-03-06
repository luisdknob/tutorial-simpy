#Environments: controlando a simulação

Em SimPy, o `Environment`é quem coordena a execução do seu programa. Ele avança o relógio de simulação, planeja a ordem de execução dos eventos e executa cada evento planejado pelo programa no instante correto.

##Controle de execução com `env.run(until=fim_da_simulação)` 

A maneira mais usual de controle de execução de um modelo de simulação é fornecendo a duração do experimento de simulação. O SimPy, como veremos, vai além e permite alguns outros modos de controle.

Incialmente, vamos trabalhar com um modelo simples, que gera chegadas de eventos em intervalos constantes entre si:

```python
import simpy

def geraChegada(env, p):
    while True:
        yield env.timeout(1)
        print("%3.1f nova chegada" %(env.now))
        
env = simpy.Environment()
chegadas = env.process(geraChegada(env, "p1"))
env.run(until=5)    # executa até o instante 5
```

Quando executado, o modelo anterior fornece:

```python
1.0 nova chegada
2.0 nova chegada
3.0 nova chegada
4.0 nova chegada
```

No modelo anterior, a última linha informa ao SimPy que a simulação deve ser executada até o instante 5 \(implicitamente o SimPy assume que o instante inicial é 0\). Esta é a maneira mais usual: o instante final de simulação é um parâmetro de entrada.

O interessante, no modelo anterior, é que se quisermos continuar a execução do instante atual (5, no caso) até o instante 10, por exemplo, podemos simplesmente acrescentar mais uma linha `env.run(until=10)` informando que a execução **continua de onde está **\(instante 5\) e termina em 10.
Isto pode ser útil em situações em que precisamos modificar algum parâmetro de entrada do modelo ao longo da própria simulação. 

Vamos modificar o modelo anterior de modo que nos primeiros 5 minutos o intervalo entre geração de chegadas seja de 1 minuto e, depois, até o instante 10, o intervalo seja de 2 minutos. Para isso, criamos uma variável global `intervalo `que armazenará o intervalo entre chegadas, como mostra o código a seguir:

```python
import simpy

intervalo = 1       # valor inicial para o intervalo entre chegada sucessivas

def geraChegada(env, p):
    global intervalo
    
    while True:
        yield env.timeout(intervalo)
        print("%3.1f nova chegada" %(env.now))

env = simpy.Environment()
chegadas = env.process(geraChegada(env, "p1"))
env.run(until=5)    # executa até o instante 5

print("\nModificando o intervalo entre chegadas para 2 min")
intervalo = 2       # novo intervalo entre chegadas sucessivas
env.run(until=10)   # executa até o instante 10
```

Depois de executado, o modelo anterior fornece:

```python
1.0 nova chegada
2.0 nova chegada
3.0 nova chegada
4.0 nova chegada

Modificando o intervalo entre chegadas para 2 min
5.0 nova chegada
7.0 nova chegada
9.0 nova chegada
```
A segunda chamada do `run`, `env.run(until=10)`, executou do instante atual (no caso, 5) até o instante 10. Assim, a opção `until `não representa a duração da simulação, mas *até que instante queremos executá-la*. Isto implica também, que uma nova chamada para `env.run` não reinicializa o tempo de simulação, isto é, não retorna o relógio do simulador para o instante 0.

Para reinicializar a relógio de um modelo em execução, o que seria equivalente a reinicializar a simulação, uma alternativa possível é acrescentar uma nova linha de criação do `environment` (e, nas linhas seguintes, realizar as chamadas de processo e do `env.run)`. Por exemplo: 
```python
...
env = simpy.Environment()
chegadas = env.process(geraChegada(env, "p1"))
env.run(until=5)                                # executa até o instante 5

print("\nModificando o intervalo entre chegadas para 2 min")
intervalo = 2                                   # novo intervalo entre chegadas sucessivas
env = simpy.Environment()                       # reinicializa o environment
chegadas = env.process(geraChegada(env, "p1"))  # nova chamada do processo chegadas

env.run(until=5)                                # reexecuta até o instante 5
```
Agora, o modelo reinicializa o relógio, como pode-se verificar pela sua saída:
```python
1.0 nova chegada
2.0 nova chegada
3.0 nova chegada
4.0 nova chegada

Modificando o intervalo entre chegadas para 2 min
2.0 nova chegada
4.0 nova chegada
```

## Parada por execução de todos os processo programados

Quando não se fornece o tempo de simulação \(ou ele não é conhecido a priori\), podemos interromper a simulação pela própria extição do processo. No programa anterior, por exemplo, podemos substituir o comando `while True` por um laço `for`e executar a simulação com um número fixo de entidades pré estabelecido:

```python
import simpy

def geraChegada(env, numEntidades):
    for i in range(0,numEntidades):
        yield env.timeout(1)
        print("%3.1f nova chegada" %(env.now))

env = simpy.Environment()
chegadas = env.process(geraChegada(env, 5)) # gere apenas 5 entidades
env.run()           # executa até o fim de todos os processos do modelo
```
Quando executado, o modelo anterior fornce como saída:
```python
1.0 nova chegada
2.0 nova chegada
3.0 nova chegada
4.0 nova chegada
5.0 nova chegada
```
Um modelo de simulação pode possuir diversos processos ocorrendo ao mesmo tempo, de modo que o término da simulação só é garantido quando *todos* os processos programados terminarem.

O próximo modelo amplia o exemplo anterior, de modo que dois processos são executados ao mesmo tempo, um com 3 entidades e outro com 5 entidades. Note que os processos também podem se armazendos em uma lista:

```python
import simpy

def geraChegada(env, p, numEntidades):
    for i in range(0,numEntidades):
        yield env.timeout(1)
        print("%3.1f nova chegada para o processo %s" %(env.now, p))
        
env = simpy.Environment()
# chegadas é uma lista que armazena os processos em execução
chegadas = [env.process(geraChegada(env, "p1", 5)), env.process(geraChegada(env, "p2", 3))]
env.run()           # executa até o fim de todos os processos do modelo
```
Quando executado, o modelos anterior fornece como saída:
```python
1.0 nova chegada para o processo p1
1.0 nova chegada para o processo p2
2.0 nova chegada para o processo p1
2.0 nova chegada para o processo p2
3.0 nova chegada para o processo p1
3.0 nova chegada para o processo p2
4.0 nova chegada para o processo p1
5.0 nova chegada para o processo p1
```

Neste último exemplo, a simulação terminou apenas quando o processo "p1", de 5 entidades, foi exaurido.

## Parada por fim de execução de processo específico por `env.run(until=processo)`

Uma outra alternativa de controle de execução é pelo término do próprio processo de execução. Partindo do exemplo anterior, podemos parar a simulação quando o processo que gera 3 entidades termina. Isto é possível com a opção `env.run(until=processo)`:

```python
import simpy

def geraChegada(env, p, numEntidades):
    for i in range(0,numEntidades):
        yield env.timeout(1)
        print("%3.1f nova chegada para o processo %s" %(env.now, p))

env = simpy.Environment()
# chegadas é uma lista que armazena os processos em execução
chegadas = [env.process(geraChegada(env, "p1", 5)), env.process(geraChegada(env, "p2", 3))]
env.run(chegadas[1])           # executa até o fim do processo armazenado em chegadas[1]
```

Quando executado, o programa anterior fornece:

```python
1.0 nova chegada para o processo p1
1.0 nova chegada para o processo p2
2.0 nova chegada para o processo p1
2.0 nova chegada para o processo p2
3.0 nova chegada para o processo p1
3.0 nova chegada para o processo p2
```

No programa anterior, a linha `env.run(until=chegadas[1])` determina que o programa seja executado até que o processo `chegadas[1]` esteja concluído. Note que na lista:

```python
chegadas = [env.process(geraChegada(env, "p1", 5)), env.process(geraChegada(env, "p2", 3))]
```

`chegadas[1]`é o processo `env.process(geraChegadas(env, "p2", 3))` que deve terminar após 3 entidades criadas. Verifique na saída do programa que, neste caso, de fato o programa parou após 3 entidades do tipo "p2" geradas.

## Simulação passo a passo: `peek` & `step`

O SimPy permite a simulação passo a passo por meio de dois comandos:

* `peek()`: retorna o instante de execução do próximo evento programado. Caso não existam mais eventos programados, retorna infinito `(float('inf'));`
* `step()`: processa o próximo evento. Caso não existam mais eventos, ele retorna um exceção interna `EmptySchedule.`

Um uso interessante da simulação passo a passo é na representação de barras de progresso. O exemplo a seguir faz uso da biblioteca [pyprind](https://github.com/rasbt/pyprind) para gerar uma barra de progresso simples \(talvez você tenha de instalar a biblioteca pyprind - veja no link como proceder\):

```python
import simpy
import pyprind

def geraChegada(env, p):
# gera chegadas em intervalos ctes            
    while True:
        yield env.timeout(1)

env = simpy.Environment()
chegadas = env.process(geraChegada(env, "p1"))

until = 1000000

# cria barra de tamanho until     
pbar = pyprind.ProgBar(until, monitor = True)

while env.peek() < until:            
   delay = env.now
   env.step()
   delay = env.now - delay
   # atualiza a barra pelo intervalo de tempo processado
   pbar.update(delay)

# imprime estatísticas da CPU   
print(pbar)  

```

    0%                          100%
    [##############################] | ETA: 00:00:00
    Title:
     Started: 10/07/2016 09:58:23
     Finished: 10/07/2016 09:58:32
     Total time elapsed: 00:00:00
     CPU %: 98.80
     Memory %: 0.32`

Existem outras possibilidades de uso do `peek()` &`step().` Por exemplo, o Spyder \(IDE sugerida para desenvolvimento dos programas deste livro\) possui opções de controle de execução passo-a-passo para [_debugging _](https://pythonhosted.org/spyder/debugging.html) no menu "Debug". Assim, podemos colocar um *breakpoint* na linha `env.step()` do programa e acompanhar melhor sua execução - coisa boa quando o modelo está com algum bug.

## Conteúdos desta seção

| **Conteúdo** | **Descrição** |
| --- | --- |
| `env.run(until=tempo_de_simulação)` | solicita que o modelo seja executado pelo tempo definido em 'tempo_de_simulação'. Note que a unidade é instrinseca ao problema simulado. |
| `env.run(until=processo)` | solicita que o modelo seja simulado até que o `processo` termine sua execução. (Caso `processo` tenha um laço infinito, o programa será simulado indefinidamente ou até que Ctrl+C seja acionado no teclado). |
| `peek()` | retorna o instante de execução do próximo evento programado. Caso não existam mais eventos programados, retorna infinito `(float('inf'));` |
| `step()` | processa o próximo evento. Caso não existam mais eventos, ele retorna um exceção interna `EmptySchedule.` |

## Desafios

>**Desafio 9**: Considere que cada entidade gerada no primeiro exemplo desta seção tem um peso em gramas dado por uma distribuição normal de média 10 e desvio padrão igual a 3. Crie um critério de parada para quando a média dos pesos das entidades geradas esteja no intervalo entre 9,5 e 10,5.

>**Desafio 10**: Modifique o critério anterior para que a parada ocorra quando a média for 10 com um intervalo de confiança de amplitude 0,5 e nível de significância igual a 95%. Dica: utilize a biblioteca `numpy` para isso \(consulte o stackoverflow!\).

