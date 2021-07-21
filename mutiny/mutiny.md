# Studyng Smallrye Mutiny

## Primeiras Linhas de Código - Hello Mutiny!

Depois de instalado e configurado corretamente podemos fazer nosso primeiro hello world com mutiny.

```java
import io.smallrye.mutiny.Uni;

public class FirstProgram {

  public static void main(String[] args) {
    Uni.createFrom().item("hello")
      .onItem().transform(item -> item + " mutiny")
      .onItem().transform(String::toUpperCase)
      .subscribe().with(
        item -> System.out.println(item + "🔥"));
  }
}
```

>**Sysout:** HELLO MUTINY🔥

<br>

---

## Passso a passo do pipeline

O que é interessante é como essa mensagem é construída. Descrevemos um fluxo de processamento pegando um item, processando-o e finalmente consumindo-o.

Primeiro, nós criamos o **Uni**, junto ao **Multi** são os dois tipos que o **Mutiny** prove. Um **Uni** é uma stream emite um único item ou falha.

Aqui nós criamos um **Uni** e emitimos um item **"hello"**. Essa é a entrada do nosso fluxo. Então processamos o item:

1. Acrescentamos a String **"mutiny"**, então
2. Colocamos tudo em uppercase

Isso forma a parte de processamento do nosso pipeline, e então ao final realizamos o **subscribe**.

Essa última parte é essencial. Se você não fizer o **subscribe** nada vai acontecer! Os tipos do Mutiny são preguiçosos, se você não demonstrar interesse o fluxo não vai nem começar.

**Nota:** *Caso seu programa não tenha feito nada, verifique se você não esqueceu do subscribe!*