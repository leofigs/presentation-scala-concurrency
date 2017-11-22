@title[Título]

## Paralelismo, Concorrência e Scala

---
@title[Perfil: Leonardo]

### Quem é o Leonardo
- 99ner
- Programador Scala há 1 ano +-
- Antes disso Python, Java, .Net ...
- github/leofigs
- leofigs no gmail
- @leonardofigs no Twitter

---
@title[Conceitos]

## Conceitos

![Conceitos](assets/concepts.gif)

---
@title[O que é Paralelismo]

### O que é Paralelismo

![Paralelismo](assets/parallelism.gif)

Note:
- coisas acontecendo ao mesmo tempo
- propriedade do *Hardware* ou plataforma

---
@title[O que é Concorrência]

### O que é Concorrência

![Concorrência](assets/concurrency.gif)

Note:
- propriedade do Software
- programas não sequenciais / fragmentados / multi-tasking
- Ex: multiplas abas do browser e outros programmas rodando ao mesmo tempo
- lida com comunicação e coordenação de mensagens, threads e processos
- Pode ser concorrente sem ser paralelo

---
@title[Sincrono x Assincrono 1 ]

## Sincrono x Assincrono

Sincrono: Atividades ou processos que começam e terminam de forma sequencial ou coordenanda.

<img src="assets/sync.gif" alt="Sync" style="height: 400px;"/>

---
@title[Sincrono x Assincrono 2 ]

## Sincrono x Assincrono

Assincrono: Atividades ou processos executam de forma não sequencial.

![sync](assets/assync.gif)

---
@title[Problemas de concorrencia]

## Problemas de concorrencia

- Não determinismo
- Race condition
- Deadlocks
- Resource starvation

---
@title[Não determinismo]

## O que é Não determinismo

Execuções não retornam sempre o mesmo valor. Muitas vezes ocorre da tentativa de sincronizar processos não sincronizaveis, como callbacks.

Ocorre também quando processos que deveríam ser sincronos são codificados como assincronos.

Note:
não determinismo muitas vezes é esperado no processamento concorrente mas normalmente é inesperado no resultado do processamento

---
@title[Race conditions]

## O que são Race conditions

Acontece quando uma aplicação depende de que processos acontecam em sequencia. Principalmente no acesso a dados compartilhados.

Exemplo:
```
Thread 1 lê o valor compartilhado A
Thread 2 lê o valor compartilhado A
Thread 2 grava um novo valor
Thread 1 grava um novo valor

O valor gravado pela thread 2 é perdido.
```
---
@title[Deadlocks]

## O que são Deadlocks

Deadlock é a situação onde dois ou mais processos esperam um ao outro para continuar executando.


---
@title[Resource starvation]

## O que é Resource starvation

É quando um processo tem um recurso negado perpetuamente.

---
@title[Scala]

## Scala

![scala](assets/scala_ninja.gif)

Note:
- Lançada em 2003
- Criada por Martim Odersky
- Multi paradigmas ( Funcional + OO)
- Compilada para bytecode Java

---
@title[Porque Scala]

### Porque Scala?

- Paradigma Funcional = menos código e maior assertividade|
- Roda na JVM - Interoperabilidade com Java|
- Sistemas assincronos e performáticos|
- Muito legal|

---
@title[Uso real Scala - 99]
### E é utilizada mesmo?

![99 Scala](assets/99_active_repos.png)

Note:
- Ativos = criados ou modificados em 2017
- Empresa poliglota
- Scala é uma das linguagens principais
- Scala é utilizada desde 2013
---
@title[Concorrencia e Scala]

### Concorrencia e Scala

- Atomic Variables
- Software Transaction Memory (STM)
- Coleções paralelas
- Futures e Promises
- Actors
---
@title[Atomic Variables]

## Atomic Variables

Note:
lock-free thread-safe operations on single variables.

---
@title[Atomic Variables - Caracteristicas]

## Características

- Operações atômicas thread-safe em variáveis ( Building blocks ) |
- Implementadas a nível de máquina utilizando instruções do processador |
- Não usa locks / non-blocking (em teoria, dependendo da JVM) |
- Operações linearizáveis = instantâneas |
- compare-and-set or compare-and-swap (CAS) |

---
@title[CAS - Source example]

## Source do AtomicInteger

```java
public final int updateAndGet(IntUnaryOperator updateFunction) {
        int prev, next;
        do {
            prev = get();
            next = updateFunction.applyAsInt(prev);
        } while (!compareAndSet(prev, next));
        return next;
    }
```

---
@title[Atomic Variables - Scala]

## Exemplo de uso com Scala

```scala
@tailrec def getUniqueId(): Long = {
     val oldUid = uid.get
     val newUid = oldUid + 1
     if (uid.compareAndSet(oldUid, newUid)) newUid
     else getUniqueId()
}
```

---
@title[STM]

## Software Transaction Memory

Note:
Segue um principio parecido com transacoes de banco
Compoem melhores que variaveis atomicas

---
@title[STM - Conceitos]

### Conceitos

- Blocos atomicos/delimitados |
- Escreve em log para depois aplicar os valores |
- Rollback / Retry |

---
@title[STM - Exemplo Conceitual]

### Exemplo conceitual

```scala
def swap() = atomic {
  val tmp = a
  a= b
  b = tmp
}

def inc() = atomic { a = a + 1 }
```
---
@title[STM - Exemplo Conceitual Grafico]

### Transação STM

![STM](assets/stm.png)

---
@title[ScalaSTM]

### ScalaSTM (https://nbronson.github.io/scala-stm/)

- Biblioteca mais conhecida |
- Criada pelo Scala STM expert group (Martin Odersky, Jonas Bonér(Akka), ...) |
- API + Implementação de referência |

---
@title[ScalaSTM - Example]

### Exemplo

```scala
val urls = Ref[List[String]](Nil)
val clen = Ref(0)

def addUrl(url: String): Unit = atomic { implicit txn =>
     urls() = url :: urls()
     clen() = clen() + url.length + 1
}
```
---
@title[Collections paralelas]

## Collections Paralelas

---
@title[Collections paralelas]

### Collections Paralelas

Otimizam computações em collections utilizando multiplos processadores.

Varios fatores influenciam:

- JVM
- collections vs operação (sum, max, filter, foreach...)
- side effects ( concorrencia )
- gerenciamento de memoria ( GC )


---
@title[Collections paralelas 2]

### Collections paralelas

- Muito facil de usar (.par) |
- Existem versões para as collections mais comuns |
- Não é indicado para todos os usos ( teste sempre )


Note:
Algumas operações são fáceis de paralelizar

---
@title[Collections paralelas - Exemplo]

### Exemplo


```scala
class ParallelCollections {

  def SeqMaxSequential =  Seq.fill(10000)(nextLong).max

  def SeqMaxParallel   =  Seq.fill(10000)(nextLong).par.max

  def VectorMaxSequential =  Vector.fill(10000)(nextLong).max

  def VectorMaxParallel   =  Vector.fill(10000)(nextLong).par.max

}
```
@[3,7](Execução Sequencial)
@[5,9](Execução Paralela)


---
@title[Futures]

## Futures

![Future](assets/future.gif)

---
@title[Future - Definição]

Futures são objetos para valores que podem não existir ainda.

O valor de um future é disponibilizado concorrentemente. Permitindo um código mais rapido, assíncrono, non-blocking e paralelo.

---
@title[Future - Exemplo]

```scala
val inverseFuture : Future[Matrix] = Future {
  fatMatrix.inverse()
}
```

Note:
O programa continua executando e não bloqueia apos iniciar o processamento

---
@title[Future - Execution Context]

### E como funciona?

O Future precisa de um ExecutionContext para executar a rotina. ( que é similar ao Executor do Java)

O ExecutionContext pode executar na mesma thread, criar uma nova ou usar alguma do pool.

Note:
Scala possui um ExecutionContext padrão global, que pode ser extendido se necessário.



---
@title[Future - ExecutionContext Example]

### Explícito
```scala
val inverseFuture: Future[Matrix] = Future {
  fatMatrix.inverse()
}(executionContext)
```

### Implicito
```scala
implicit val ec: ExecutionContext = ...
val inverseFuture : Future[Matrix] = Future {
  fatMatrix.inverse()
}
```
---
@title[Future - ExecutionContext Global]

### Com o ExecutionContext Global

```scala
import scala.concurrent.ExecutionContext.Implicits.global
...
val inverseFuture : Future[Matrix] = Future {
  fatMatrix.inverse()
}
```

Note:
Possui um Thread Pool de Tamanho Fixo

---
@title[Future - Completo ou Incompleto]

### Completando um Future

Quando um Future ainda está processando ele esta **incompleto**. Quando finaliza esta **completo**.

Um Future pode finalizar com **sucesso** ou **falha**.

---
@title[Future - Callback]

### Callback

```scala
import scala.util.Random
import scala.util.{ Success, Failure}
import scala.concurrent.Future
import scala.concurrent.ExecutionContext.Implicits.global


val aFewNumbers : Future[List[Int]] = Future { List.fill(10)(Random.nextInt) }

aFewNumbers.onComplete {
  case Success(numbersList) => println("Os números são " + numbersList.toString)
  case Failure(err) 				=> println("Sem números para você :(")
}
```

---
@title[Future - Callbacks ForEach]

### Callback com foreach

```scala
​val aFewNumbers : Future[List[Int]] =
  Future { List.fill(5)(Random.nextInt) }
​
aFewNumbers.foreach(numList => numList map println)
​
aFewNumbers.failed.foreach(err => println(s"The house felt down. Error $err"))
```

Note:
como foreach não tem side-effect ( retorna Unit), ele se comporta como um callback
Codigo: println com parametros inferidos
Codigo: string interpolation

---
@title[Future - Map]

### Mapping the Future

```scala
import scala.util.Random
import scala.concurrent.Future
import scala.concurrent.ExecutionContext.Implicits.global


val aFewNumbers : Future[List[Int]] = Future { List.fill(5)(Random.nextInt) }

val onlyTens: Future[List[Int]] = aFewNumbers.map(numList => numList.map(num => 10))

onlyTens.foreach(numList => numList map println)
```
---
@title[Future - FlatMap 1]

### Future of Future

```scala
import scala.util.Random
import scala.concurrent.Future
import scala.concurrent.ExecutionContext.Implicits.global


def allTens(numList: List[Int]): Future[List[Int]] = Future{ numList.map(num => 10)}

val aFewNumbers : Future[List[Int]] = Future { List.fill(5)(Random.nextInt) }

val onlyTens = aFewNumbers.map(numList => allTens(numList))

```

@[10](O tipo de onlyTens é Future[Future[List[Int]]])
---
@title[Future - FlatMap 2]

### Flatmapping the Future

```scala
import scala.util.Random
import scala.concurrent.Future
import scala.concurrent.ExecutionContext.Implicits.global


def allTens(numList: List[Int]): Future[List[Int]] = Future{ numList.map(num => 10)}

val aFewNumbers : Future[List[Int]] = Future { List.fill(5)(Random.nextInt) }

val onlyTens = aFewNumbers.flatMap(numList => allTens(numList))
```

@[10](O tipo de onlyTens é Future[List[Int]])

---
@title[Future - For]

### Future composition com For

```scala


def allTens(numList: List[Int]): Future[List[Int]] =
  Future{ numList.map(num => 10)}

val aFewNumbers : Future[List[Int]] =
  Future { List.fill(5)(Random.nextInt) }

val onlyTens = aFewNumbers.flatMap(numList => allTens(numList))

val group = for {
                  listOne <- aFewNumbers
                  listTwo <- onlyTens
                  if listOne.length == listTwo.length
                } yield(listOne ++ listTwo)
```

---
@title[Future - Promises]

### Promise

Objeto que pode ser completado com um valor ou um erro.

Encapsula um future.

---
@title[Future - Promise - Example]

### Promise Example
```scala
object Government {
  def redeemCampaignPledge(): Future[TaxCut] = {
    val p = Promise[TaxCut]()
    Future {
      println("Starting the new legislative period.")
      Thread.sleep(2000)
      p.success(TaxCut(20))
      println("We reduced the taxes! You must reelect us!!!!1111")
    }
    p.future
  }
}
```


---
@title[Escaladores]


---
@title[Bibliografia / Sources]

### Fontes:

- Gifs by http://slimjimstudios.tumblr.com/

- [The Neophyte's Guide to Scala](http://danielwestheide.com/scala/neophytes.html)

- [Scala Docs](https://docs.scala-lang.org/overviews/)

- Learning Concurrent Programming in Scala - 2nd Edition
