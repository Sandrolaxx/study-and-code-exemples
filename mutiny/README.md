# Estudando Smallrye Mutiny

## Primeiras Linhas de Código - Olá Mutiny!

Depois de instalado e configurado corretamente podemos fazer nosso primeiro hello world com mutiny.

```java
import io.smallrye.mutiny.Uni;

public class FirstProgram {

  public static void main(String[] args) {
    Uni.createFrom().item("Olá")
      .onItem().transform(item -> item + " mutiny")
      .onItem().transform(String::toUpperCase)
      .subscribe().with(
        item -> System.out.println(item + "🔥"));
  }
  
}
```

>**Sysout:** OLÁ MUTINY🔥

<br>

---

## Passso a passo do fluxo

O que é interessante é como essa mensagem é construída. Descrevemos um fluxo de processamento pegando um item, processando-o e finalmente consumindo-o.

Primeiro, nós criamos o **Uni**, junto ao **Multi** são os dois tipos que o **Mutiny** prove. Um **Uni** é uma stream emite um único item ou falha.

Aqui nós criamos um **Uni** e emitimos um item **"hello"**. Essa é a entrada do nosso fluxo. Então processamos o item:

1. Acrescentamos a String **"mutiny"**, então
2. Colocamos tudo em uppercase

Isso forma a parte de processamento do nosso pipeline, e então ao final realizamos o **subscribe**.

Essa última parte é essencial. Se você não fizer o **subscribe** nada vai acontecer! Os tipos do Mutiny são preguiçosos, se você não demonstrar interesse o fluxo não vai nem começar.

```
NOTA: Caso seu programa não tenha feito nada, verifique se você não esqueceu do subscribe!
```
<br>

# Mutiny usa builder API

Outro aspecto importante do fluxo de construção é que a cada estágio adicionado ele retorna um novo **Uni**.

O programa anterior equivale a:

```java
Uni<String> uni1 = Uni.createFrom().item("Olá");
Uni<String> uni2 = uni1.onItem().transform(item -> item + " mutiny");
Uni<String> uni3 = uni2.onItem().transform(String::toUpperCase);

uni3.subscribe().with(item -> System.out.println(">> " + item)); 
```

É importante intender que aquele programa não equivale a:

```java
Uni<String> uni = Uni.createFrom().item("Olá");

uni.onItem().transform(item -> item + " mutiny");
uni.onItem().transform(String::toUpperCase);

uni.subscribe().with(item -> System.out.println(">> " + item));
```

O programa acima apenas printa ">> hello", como ele não usa os estágios adicionados e o assinante(subscribe) final consome apenas o primeiro item.

```
NOTA: APIs do Mutiny não são fluentes e a cada estágio de computação retornam um novo objeto.
```

<br>

# Criando Unis

Um Uni representa um fluxo que só pode emitir um item ou um evento de falha. Raramente você cria instâncias de Uni, mas, em vez disso, usa um cliente reativo que expõe uma API Mutiny que fornece Unis. Dito isso, às vezes pode ser útil.

---

<br>

## O tipo Uni
Um **Uni\<T>** é um fluxo especializado que emite apenas um item ou falha. Normalmente, **Uni\<T>** são ótimos para representar ações assíncronas, como uma chamada de procedimento remoto, uma solicitação HTTP ou uma operação que produz um único resultado.

**Uni\<T>** fornece muitos operadores que criam, transformam e orquestram sequências Uni.

Como disse, **Uni\<T>** emite um item ou uma falha. Observe que o item pode ser nulo e a API Uni possui métodos específicos para esse caso.

Normalmente, um **Uni\<Void>**  sempre emite nulo como evento de item ou uma falha se a operação representada falhar. Você pode considerar o evento do item como um sinal de conclusão, indicando o sucesso da operação.

Os operadores oferecidos podem ser usados ​​para definir um fluxo de processamento. O evento, seja o item ou a falha, flui neste fluxo e cada operador pode processar ou transformar o evento. Unis são preguiçosos por natureza.

Para acionar um processamento, você deve ter um assinante(subscribe) final indicando seu interesse. O snippet a seguir fornece um exemplo simples de fluxo usando Uni:

```java
Uni.createFrom().item(1)
        .onItem().transform(i -> "Olá-" + i)
        .onItem().delayIt().by(Duration.ofMillis(100))
        .subscribe().with(System.out::println);
```

Você pode ver o javadoc do **Uni** [aqui](https://javadoc.io/static/io.smallrye.reactive/mutiny/0.19.2/index.html?io/smallrye/mutiny/Uni.html).

---

<br>

## Inscrevendo-se(Subscribing) em um Uni.

Lembre-se: se você não se inscrever(subscribe), nada vai acontecer. Além do mais, o fluxo é materializado para cada assinatura.

Ao assinar um **Uni**, você pode passar um retorno de chamada de item (invocado quando o item é emitido), ou dois retornos de chamada (um recebendo o item e outro recebendo a falha):

```java
Cancellable cancellable = uni
        .subscribe().with(
                item -> System.out.println(item),
                failure -> System.out.println("Falha por " + failure));
```

Observe que ele retorna uma **Cancellable**: este objeto permite cancelar a operação se necessário.

---

<br>

## Criando Unis com base em um item

Existem muitas maneiras de criar instâncias Uni. Use **Uni.createFrom()** para ver todas as possibilidades. 

Você pode, por exemplo, criar um Uni a partir de um valor conhecido:

```java
Uni<Integer> uni = Uni.createFrom().item(1);
```

Todo assinante recebe o item 1 logo após a assinatura. 

Você também pode passar um **Supplier**:

```java
AtomicInteger counter = new AtomicInteger();
Uni<Integer> uni = Uni.createFrom().item(() -> counter.getAndIncrement());
```
O **Supplier** é chamado para cada inscrição. Portanto, cada um deles terá um valor diferente.

---

<br>

## Criando Unis com falha

As operações representadas pelo Unis também podem emitir um evento de falha, indicando que a operação falhou. 

Você pode criar instâncias Uni com falha com:
```java
// Passando a exception diretamente:
Uni<Integer> failed1 = Uni.createFrom().failure(new Exception("boom"));

// Passando um supplier chamado em cada inscrição:
Uni<Integer> failed2 = Uni.createFrom().failure(() -> new Exception("boom"));
```
---

<br>

## Criando Uni\<Void>

Quando a operação representada não produz um resultado, você ainda precisa de uma maneira de indicar a conclusão da operação. Para isso, você precisa emitir um **null** item:
```java
Uni<Void> uni = Uni.createFrom().nullItem();
```
---

<br>

## Criando Unis para usar em um emitter(avançado)

Você pode criar um Uni usando um emitter. Essa abordagem é útil ao integrar APIs baseadas em callback:

```java
Uni<String> uni = Uni.createFrom().emitter(em -> {
    // Quando o resultado estiver disponível ele emitira ele
    em.complete(result);
});
```
O emitter também pode enviar uma falha. Ele também pode ser notificado sobre o cancelamento para, por exemplo, interromper o trabalho em andamento.

---

<br>

## Criando Unis a partir de CompletionStage(avançado)

Você também pode criar objetos **Uni** de **CompletionStage** / **CompletableFuture**. Isso é útil na integração com APIs baseadas nestes tipos:
```java
Uni<String> uni = Uni.createFrom().completionStage(stage);
```

SUBSCRIBE COMO COMPLETIONSTAGE <BR>
Você também pode criar um CompletionStage a partir de um Uni usando uni.subscribe().AsCompletionStage()