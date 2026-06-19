# Estruturas de repetição em Java

Em aplicações backend, quase nunca lidamos com apenas um único dado isolado. Um sistema real precisa percorrer listas de músicas, processar lotes de registros, validar itens de uma requisição, consumir páginas de uma API externa, recalcular estatísticas, aplicar regras em coleções, tentar uma operação novamente ou interromper um processamento quando uma condição de negócio é atingida.

Sem estruturas de repetição, o código ficaria duplicado:

```java
System.out.println("Processando música 1");
System.out.println("Processando música 2");
System.out.println("Processando música 3");
```

Isso não escala. Se amanhã forem 10, 100 ou 10.000 músicas, o código precisaria crescer manualmente. Estruturas de repetição resolvem esse problema permitindo executar o mesmo bloco de código várias vezes, de forma controlada.

No contexto do `meucatalogomusical.com`, imagine algumas necessidades reais:

* montar uma playlist com músicas bem avaliadas;
* percorrer o catálogo para encontrar músicas de Metal, Hard Rock ou Rock Progressivo;
* processar uma fila de músicas pendentes de revisão;
* consumir páginas de uma API de bandas;
* interromper o processamento quando a duração máxima da playlist for atingida;
* ignorar itens inválidos sem interromper o processamento inteiro.

Esses cenários são resolvidos com `for`, `while`, `do/while`, `break` e `continue`.

## Antes de entrar no código: uma ideia simples

Uma estrutura de repetição responde a perguntas como:

> “Enquanto ainda houver músicas pendentes, continue processando.”

> “Para cada música do catálogo, aplique esta regra.”

> “Execute esta validação pelo menos uma vez e repita enquanto o valor estiver inválido.”

> “Pare o processamento quando a playlist atingir o limite de duração.”

A ideia central é simples: repetir um bloco enquanto uma condição permitir.

Existem três formas principais de repetição em Java:

```java
for (...) {
    // repete com controle mais explícito
}
```

```java
while (...) {
    // repete enquanto a condição for verdadeira
}
```

```java
do {
    // executa pelo menos uma vez
} while (...);
```

Além disso, existem duas instruções importantes para alterar o fluxo normal do loop:

```java
break;    // interrompe o loop
continue; // pula a iteração atual
```

## Definição técnica

Em Java, estruturas de repetição são comandos de controle de fluxo que executam um bloco de código repetidamente de acordo com uma condição booleana ou uma sequência de elementos.

A Java Language Specification do Java SE 25 define formalmente os comandos `while`, `do`, `for`, `break` e `continue` nas seções de statements da linguagem. ([Oracle Docs][2])

De forma técnica:

* `for` é usado quando há uma estrutura clara de inicialização, condição e atualização;
* `while` é usado quando a repetição depende de uma condição avaliada antes de cada execução;
* `do/while` é usado quando o bloco precisa executar pelo menos uma vez;
* `break` causa uma saída antecipada do loop;
* `continue` pula o restante da iteração atual e segue para a próxima avaliação do loop.

## Como funciona em Java

### `for`

O `for` tradicional tem três partes principais:

```java
for (inicialização; condição; atualização) {
    // bloco repetido
}
```

Exemplo:

```java
for (var indice = 0; indice < 10; indice++) {
    System.out.println(indice);
}
```

A ordem de execução é:

1. executa a inicialização uma única vez;
2. avalia a condição;
3. se a condição for `true`, executa o bloco;
4. executa a atualização;
5. volta para a condição;
6. encerra quando a condição for `false`.

O `for` é muito útil quando existe um contador, índice ou limite bem definido.

Também existe o enhanced `for`, muito usado para percorrer arrays e coleções:

```java
for (var musica : musicas) {
    System.out.println(musica.titulo());
}
```

Esse formato é melhor quando você não precisa do índice, apenas do elemento atual.

### `for` com múltiplas variáveis de controle

O `for` tradicional também permite trabalhar com mais de uma variável na inicialização e na atualização do laço.

Esse recurso é útil quando duas informações precisam evoluir ao mesmo tempo. Por exemplo, uma variável pode avançar do início para o fim de uma lista, enquanto outra retrocede do fim para o início.

```java
for (int inicio = 0, fim = 9; inicio < 10; inicio++, fim--) {
    System.out.printf("%d - %d%n", inicio, fim);
}
```

Nesse exemplo, existem duas variáveis de controle:

* `inicio`, que começa em `0` e aumenta a cada iteração;
* `fim`, que começa em `9` e diminui a cada iteração.

A cada repetição, o bloco imprime os dois valores. Depois disso, o Java executa a parte de atualização:

```java
inicio++, fim--
```

Ou seja, primeiro incrementa `inicio` e depois decrementa `fim`.

A saída será:

```text
0 - 9
1 - 8
2 - 7
3 - 6
4 - 5
5 - 4
6 - 3
7 - 2
8 - 1
9 - 0
```

Esse tipo de `for` pode ser útil em algoritmos que precisam comparar elementos em posições opostas, como verificar pares do início e do fim de uma lista.

Imagine uma regra simples no `meucatalogomusical.com`: comparar músicas em posições opostas de uma playlist para identificar combinações entre abertura e encerramento.

```java
import java.util.List;

public class ComparacaoPlaylist {

    record Musica(String banda, String titulo) {
    }

    public static void main(String[] args) {
        var playlist = List.of(
                new Musica("Iron Maiden", "The Trooper"),
                new Musica("Metallica", "Orion"),
                new Musica("Rush", "Tom Sawyer"),
                new Musica("Pink Floyd", "Time")
        );

        for (int inicio = 0, fim = playlist.size() - 1;
             inicio < fim;
             inicio++, fim--) {

            var musicaInicial = playlist.get(inicio);
            var musicaFinal = playlist.get(fim);

            System.out.printf(
                    "Comparando: %s - %s / %s - %s%n",
                    musicaInicial.banda(),
                    musicaInicial.titulo(),
                    musicaFinal.banda(),
                    musicaFinal.titulo()
            );
        }
    }
}
```

Nesse exemplo, o loop começa comparando a primeira música com a última. Depois, compara a segunda com a penúltima. O loop continua enquanto `inicio < fim`.

A condição `inicio < fim` evita comparar a mesma posição duas vezes e também evita que os índices se cruzem.

Apesar de ser um recurso válido da linguagem, esse tipo de `for` deve ser usado com moderação. Ele é interessante quando as duas variáveis realmente representam partes do mesmo controle de repetição. Se a lógica começar a ficar difícil de entender, normalmente é melhor separar o código em métodos menores ou usar nomes mais descritivos.

Evite escrever loops assim apenas para “economizar linhas”. O objetivo deve ser expressar uma intenção clara, não compactar código.

```java
for (int i = 0, j = musicas.size() - 1; i < j; i++, j--) {
    // aceitável quando i e j fazem parte da mesma lógica
}
```

Mas evite misturar controles sem relação direta:

```java
for (int indice = 0, tentativas = 3; indice < musicas.size(); indice++, tentativas--) {
    // confuso se indice e tentativas não representam a mesma regra de repetição
}
```

Nesse segundo caso, `indice` e `tentativas` parecem representar conceitos diferentes. Isso pode prejudicar a legibilidade e aumentar o risco de erro.

### Regras importantes sobre tipos no `for` com múltiplas variáveis

Quando as variáveis são declaradas dentro do próprio cabeçalho do `for`, elas fazem parte de uma única declaração local.

Por isso, este código é válido:

```java
for (int inicio = 0, fim = 9; inicio < 10; inicio++, fim--) {
    System.out.printf("%d - %d%n", inicio, fim);
}
```

Nesse caso, `inicio` e `fim` são duas variáveis do tipo `int`.

O ponto importante é que o tipo `int` vale para todos os declaradores dessa mesma declaração. Portanto, não é possível declarar variáveis de tipos diferentes nessa mesma parte do `for` fazendo algo como:

```java
for (int inicio = 0, long fim = 9L; inicio < 10; inicio++, fim--) {
    System.out.printf("%d - %d%n", inicio, fim);
}
```

Esse código não compila.

Se você realmente precisar trabalhar com tipos diferentes, declare as variáveis antes do loop:

```java
int inicio = 0;
long fim = 9L;

for (; inicio < 10; inicio++, fim--) {
    System.out.printf("%d - %d%n", inicio, fim);
}
```

Nesse exemplo, a inicialização do `for` ficou vazia, porque as variáveis já foram declaradas antes. O loop continua usando a condição e a atualização normalmente.

Outro detalhe importante envolve o uso de `var`.

Embora `var` possa ser usado em variáveis locais desde o Java 10, ele não pode ser usado com múltiplos declaradores na mesma declaração. Portanto, este código não é válido:

```java
for (var inicio = 0, fim = 9; inicio < 10; inicio++, fim--) {
    System.out.printf("%d - %d%n", inicio, fim);
}
```

A forma correta, usando `var`, seria declarar as variáveis separadamente antes do `for`:

```java
var inicio = 0;
var fim = 9;

for (; inicio < 10; inicio++, fim--) {
    System.out.printf("%d - %d%n", inicio, fim);
}
```

Ainda assim, em muitos casos, usar o tipo explícito no cabeçalho do `for` pode ser mais direto e legível:

```java
for (int inicio = 0, fim = musicas.size() - 1;
     inicio < fim;
     inicio++, fim--) {

    var primeira = musicas.get(inicio);
    var ultima = musicas.get(fim);

    comparar(primeira, ultima);
}
```

A regra prática é: use múltiplas variáveis no `for` quando elas fizerem parte da mesma lógica de controle e forem naturalmente do mesmo tipo. Se os tipos forem diferentes ou se as variáveis representarem responsabilidades diferentes, prefira declará-las antes do loop ou reorganizar o código para preservar a legibilidade.


### Use múltiplas variáveis no `for` apenas quando elas fizerem parte da mesma regra

O `for` permite múltiplas variáveis de controle, mas isso não significa que sempre seja uma boa ideia.

Use esse recurso quando as variáveis caminharem juntas para representar uma mesma regra de iteração:

```java
for (int esquerda = 0, direita = musicas.size() - 1;
     esquerda < direita;
     esquerda++, direita--) {

    var primeira = musicas.get(esquerda);
    var ultima = musicas.get(direita);

    comparar(primeira, ultima);
}
```

Esse código é aceitável porque `esquerda` e `direita` fazem parte da mesma lógica: percorrer a lista pelas duas extremidades.

Evite usar múltiplas variáveis quando elas representarem responsabilidades diferentes. Nesses casos, o loop pode ficar compacto, mas menos legível.


### `while`

O `while` avalia a condição antes de executar o bloco:

```java
while (condição) {
    // bloco repetido
}
```

Exemplo:

```java
while (!fila.isEmpty()) {
    var musica = fila.removeFirst();
    System.out.println(musica.titulo());
}
```

Ele é comum quando você não sabe exatamente quantas repetições serão necessárias. Em vez de pensar “execute 10 vezes”, você pensa “execute enquanto ainda houver trabalho”.

### `do/while`

O `do/while` executa o bloco antes de avaliar a condição:

```java
do {
    // bloco executado pelo menos uma vez
} while (condição);
```

Isso é útil quando a primeira execução sempre precisa acontecer. Um caso comum é buscar a primeira página de uma integração externa e continuar enquanto houver próxima página.

### `break`

O `break` interrompe o loop imediatamente:

```java
for (var musica : musicas) {
    if (musica.duracaoSegundos() > 600) {
        break;
    }

    System.out.println(musica.titulo());
}
```

Depois do `break`, o programa continua na primeira instrução após o loop.

### `continue`

O `continue` pula o restante da iteração atual:

```java
for (var musica : musicas) {
    if (musica.notaCuradoria() < 8) {
        continue;
    }

    System.out.println("Música recomendada: " + musica.titulo());
}
```

Nesse exemplo, músicas com nota menor que 8 são ignoradas, mas o loop continua processando as próximas.

## Exemplo prático

O exemplo abaixo simula parte do backend do `meucatalogomusical.com`.

A aplicação possui um catálogo com bandas de Metal, Hard Rock e Rock Progressivo. O código monta uma playlist com músicas bem avaliadas, processa uma fila de revisão e simula uma sincronização paginada.

```java
import java.util.ArrayDeque;
import java.util.ArrayList;
import java.util.List;

public class CuradoriaCatalogoMusical {

    enum Genero {
        METAL,
        HARD_ROCK,
        ROCK_PROGRESSIVO
    }

    record Musica(
            String banda,
            String titulo,
            Genero genero,
            int duracaoSegundos,
            int notaCuradoria
    ) {
    }

    record Pagina<T>(List<T> itens, boolean temProxima) {
    }

    public static void main(String[] args) {
        var catalogo = List.of(
                new Musica("Iron Maiden", "The Trooper", Genero.METAL, 252, 9),
                new Musica("Metallica", "Orion", Genero.METAL, 507, 10),
                new Musica("Black Sabbath", "Paranoid", Genero.METAL, 168, 8),
                new Musica("Led Zeppelin", "Black Dog", Genero.HARD_ROCK, 296, 9),
                new Musica("Deep Purple", "Highway Star", Genero.HARD_ROCK, 368, 8),
                new Musica("Rush", "Tom Sawyer", Genero.ROCK_PROGRESSIVO, 277, 10),
                new Musica("Pink Floyd", "Time", Genero.ROCK_PROGRESSIVO, 412, 10),
                new Musica("Dream Theater", "Pull Me Under", Genero.ROCK_PROGRESSIVO, 493, 9)
        );

        var playlist = montarPlaylistDeDestaques(catalogo, 20 * 60, 8);

        System.out.println("Playlist de destaques:");
        exibirPlaylist(playlist);

        System.out.println();
        System.out.println("Processando fila de revisão:");
        processarFilaDeRevisao(catalogo);

        System.out.println();
        System.out.println("Sincronizando catálogo por páginas:");
        sincronizarCatalogoPorPaginas(catalogo, 3);
    }

    private static List<Musica> montarPlaylistDeDestaques(
            List<Musica> catalogo,
            int limiteDuracaoSegundos,
            int notaMinima
    ) {
        var selecionadas = new ArrayList<Musica>();
        var duracaoTotal = 0;

        for (var indice = 0; indice < catalogo.size(); indice++) {
            var musica = catalogo.get(indice);

            if (musica.notaCuradoria() < notaMinima) {
                continue;
            }

            if (duracaoTotal + musica.duracaoSegundos() > limiteDuracaoSegundos) {
                break;
            }

            selecionadas.add(musica);
            duracaoTotal += musica.duracaoSegundos();
        }

        return List.copyOf(selecionadas);
    }

    private static void exibirPlaylist(List<Musica> playlist) {
        for (var musica : playlist) {
            System.out.printf(
                    "- %s - %s (%s, %d segundos)%n",
                    musica.banda(),
                    musica.titulo(),
                    musica.genero(),
                    musica.duracaoSegundos()
            );
        }
    }

    private static void processarFilaDeRevisao(List<Musica> catalogo) {
        var fila = new ArrayDeque<>(catalogo);

        while (!fila.isEmpty()) {
            var musica = fila.removeFirst();

            if (musica.genero() == Genero.ROCK_PROGRESSIVO) {
                System.out.printf(
                        "Revisão detalhada: %s - %s%n",
                        musica.banda(),
                        musica.titulo()
                );
            } else {
                System.out.printf(
                        "Revisão padrão: %s - %s%n",
                        musica.banda(),
                        musica.titulo()
                );
            }
        }
    }

    private static void sincronizarCatalogoPorPaginas(
            List<Musica> catalogo,
            int tamanhoPagina
    ) {
        var paginaAtual = 1;
        Pagina<Musica> pagina;

        do {
            pagina = buscarPagina(catalogo, paginaAtual, tamanhoPagina);

            System.out.printf(
                    "Página %d: %d música(s)%n",
                    paginaAtual,
                    pagina.itens().size()
            );

            for (var musica : pagina.itens()) {
                System.out.printf(
                        "Enviando para índice de busca: %s - %s%n",
                        musica.banda(),
                        musica.titulo()
                );
            }

            paginaAtual++;
        } while (pagina.temProxima());
    }

    private static Pagina<Musica> buscarPagina(
            List<Musica> catalogo,
            int paginaAtual,
            int tamanhoPagina
    ) {
        var inicio = (paginaAtual - 1) * tamanhoPagina;
        var fim = Math.min(inicio + tamanhoPagina, catalogo.size());

        var itens = catalogo.subList(inicio, fim);
        var temProxima = fim < catalogo.size();

        return new Pagina<>(itens, temProxima);
    }
}
```

O exemplo usa `record`, recurso finalizado no Java 16 pela JEP 395, para representar dados simples de forma compacta. ([openjdk.org][3]) Também usa `var`, recurso introduzido no Java 10 pela JEP 286, permitido em variáveis locais, variáveis do `for` tradicional e variáveis do enhanced `for`. ([openjdk.org][4])

## Analisando o código

### Modelagem inicial

```java
enum Genero {
    METAL,
    HARD_ROCK,
    ROCK_PROGRESSIVO
}
```

O `enum` representa os gêneros aceitos no catálogo. Isso evita usar strings soltas como `"metal"` ou `"rock"`, reduzindo erro de digitação e melhorando a clareza das regras.

```java
record Musica(
        String banda,
        String titulo,
        Genero genero,
        int duracaoSegundos,
        int notaCuradoria
) {
}
```

O `record` representa uma música do catálogo. Ele é adequado porque `Musica` é um objeto de dados simples: banda, título, gênero, duração e nota de curadoria.

```java
record Pagina<T>(List<T> itens, boolean temProxima) {
}
```

O `record Pagina<T>` representa uma resposta paginada. Ele é genérico porque pode paginar músicas, bandas, álbuns ou qualquer outro tipo.

### Uso do `for` tradicional

```java
for (var indice = 0; indice < catalogo.size(); indice++) {
    var musica = catalogo.get(indice);

    if (musica.notaCuradoria() < notaMinima) {
        continue;
    }

    if (duracaoTotal + musica.duracaoSegundos() > limiteDuracaoSegundos) {
        break;
    }

    selecionadas.add(musica);
    duracaoTotal += musica.duracaoSegundos();
}
```

Esse trecho usa `for` tradicional porque existe uma navegação por índice.

A variável `indice` começa em `0`, continua enquanto for menor que `catalogo.size()` e é incrementada ao final de cada iteração.

O `continue` aparece quando a música não atinge a nota mínima. Nesse caso, a música é ignorada e o loop passa para a próxima.

O `break` aparece quando adicionar a música faria a playlist ultrapassar o limite de duração. Nesse caso, não faz sentido continuar montando a playlist, então o loop é encerrado.

Esse é um exemplo realista de regra de negócio: continuar enquanto os itens são válidos e parar quando um limite operacional for atingido.

### Uso do enhanced `for`

```java
for (var musica : playlist) {
    System.out.printf(
            "- %s - %s (%s, %d segundos)%n",
            musica.banda(),
            musica.titulo(),
            musica.genero(),
            musica.duracaoSegundos()
    );
}
```

Aqui não existe necessidade de índice. O objetivo é apenas percorrer cada música da playlist.

Nesse caso, o enhanced `for` deixa o código mais claro do que um `for` com contador.

Compare:

```java
for (var indice = 0; indice < playlist.size(); indice++) {
    var musica = playlist.get(indice);
}
```

Com:

```java
for (var musica : playlist) {
    // usa musica diretamente
}
```

A segunda versão comunica melhor a intenção: “para cada música da playlist”.

### Uso do `while`

```java
var fila = new ArrayDeque<>(catalogo);

while (!fila.isEmpty()) {
    var musica = fila.removeFirst();

    if (musica.genero() == Genero.ROCK_PROGRESSIVO) {
        System.out.printf(
                "Revisão detalhada: %s - %s%n",
                musica.banda(),
                musica.titulo()
        );
    } else {
        System.out.printf(
                "Revisão padrão: %s - %s%n",
                musica.banda(),
                musica.titulo()
        );
    }
}
```

Esse trecho usa `while` porque a repetição depende do estado da fila.

A pergunta aqui não é “quantas vezes devo repetir?”, mas sim:

> “Ainda existem músicas pendentes de revisão?”

Enquanto a fila não estiver vazia, o processamento continua. A cada iteração, uma música é removida da fila com `removeFirst()`.

Esse padrão aparece bastante em sistemas reais: filas em memória, filas de mensagens, tarefas pendentes, jobs assíncronos e processamento em lote.

### Uso do `do/while`

```java
do {
    pagina = buscarPagina(catalogo, paginaAtual, tamanhoPagina);

    System.out.printf(
            "Página %d: %d música(s)%n",
            paginaAtual,
            pagina.itens().size()
    );

    for (var musica : pagina.itens()) {
        System.out.printf(
                "Enviando para índice de busca: %s - %s%n",
                musica.banda(),
                musica.titulo()
        );
    }

    paginaAtual++;
} while (pagina.temProxima());
```

Esse trecho usa `do/while` porque a primeira página precisa ser buscada antes de saber se existe próxima página.

Esse é um cenário comum em integrações:

1. busca a primeira página;
2. processa os itens;
3. verifica se há próxima página;
4. repete enquanto houver mais dados.

A diferença para o `while` é importante. Com `while`, a condição é avaliada antes da primeira execução. Com `do/while`, a primeira execução é garantida.

## Comparação com versões anteriores do Java

As estruturas básicas `for`, `while`, `do/while`, `break` e `continue` fazem parte da linguagem Java desde suas primeiras versões. A forma essencial de uso continua a mesma em Java 25.

O que evoluiu ao redor delas foi a ergonomia da linguagem.

### Enhanced `for` — Java 5

O enhanced `for` foi introduzido no Java 5 dentro do conjunto de mudanças da JSR 201. A proposta descreve o enhanced `for` como uma forma conveniente de iterar sobre coleções sem declarar explicitamente um `Iterator`, reduzindo código repetitivo e oportunidades de erro. ([jcp.org][5])

Antes:

```java
for (int i = 0; i < musicas.size(); i++) {
    Musica musica = musicas.get(i);
    System.out.println(musica.titulo());
}
```

Depois:

```java
for (Musica musica : musicas) {
    System.out.println(musica.titulo());
}
```

Em Java 25, esse formato continua sendo a melhor escolha quando você precisa apenas dos elementos, sem índice.

### `var` em variáveis locais — Java 10, JEP 286

A partir do Java 10, a JEP 286 permitiu inferência de tipo para variáveis locais, incluindo variáveis declaradas no `for` tradicional e no enhanced `for`. ([openjdk.org][4])

Antes:

```java
for (Musica musica : musicas) {
    System.out.println(musica.titulo());
}
```

Depois:

```java
for (var musica : musicas) {
    System.out.println(musica.titulo());
}
```

Isso não torna Java uma linguagem dinamicamente tipada. O tipo continua sendo resolvido em tempo de compilação. A diferença é que o compilador infere o tipo a partir do contexto.

As diretrizes de estilo do OpenJDK reforçam que `var` deve ser usado com julgamento, porque pode melhorar ou piorar a legibilidade dependendo do nome da variável e da clareza do inicializador. ([openjdk.org][6])

## O que acontece por baixo dos panos

Loops são estruturas de alto nível da linguagem Java. Quando o código é compilado, o compilador transforma essas estruturas em bytecode executado pela JVM.

Conceitualmente, um loop vira uma combinação de:

* avaliação de condição;
* saltos condicionais;
* saltos para voltar ao início do loop;
* saída para o ponto posterior ao loop.

A JVM Specification do Java SE 25 documenta instruções de transferência de controle, que são a base para representar desvios de fluxo em bytecode. ([Oracle Docs][7])

Um `while`, por exemplo:

```java
while (contador < 10) {
    contador++;
}
```

Pode ser entendido conceitualmente como:

```text
verifica condição
se falso, sai
executa bloco
volta para verificar condição
```

O `break` altera esse fluxo mandando a execução para depois do loop. O `continue` manda a execução para o próximo ciclo do loop.

Em loops aninhados, `break` e `continue` afetam o loop mais interno por padrão:

```java
while (condicaoExterna) {
    for (var musica : musicas) {
        if (musica.notaCuradoria() < 5) {
            break;
        }
    }

    // O while continua.
}
```

Nesse exemplo, o `break` interrompe apenas o `for`, não o `while`.

Java também possui labels para `break` e `continue`, mas elas devem ser usadas com muito cuidado:

```java
externo:
for (var banda : bandas) {
    for (var musica : banda.musicas()) {
        if (musica.titulo().equals("Time")) {
            break externo;
        }
    }
}
```

Esse tipo de construção pode ser útil em casos específicos, mas frequentemente indica que o código deveria ser extraído para um método com `return`, tornando o fluxo mais simples.

## Boas práticas

### Escolha o loop pela intenção

Use `for` tradicional quando houver índice, contador ou limite numérico claro:

```java
for (var indice = 0; indice < musicas.size(); indice++) {
    // precisa do índice
}
```

Use enhanced `for` quando precisar apenas percorrer elementos:

```java
for (var musica : musicas) {
    // usa o elemento diretamente
}
```

Use `while` quando a repetição depender de uma condição de estado:

```java
while (!fila.isEmpty()) {
    // processa enquanto houver itens
}
```

Use `do/while` quando a primeira execução for obrigatória:

```java
do {
    // busca uma página
} while (temProximaPagina);
```

### Evite loops infinitos acidentais

Um loop infinito pode ser proposital:

```java
while (true) {
    // processo contínuo
}
```

Mas precisa ter uma condição clara de parada:

```java
while (true) {
    var evento = consumidor.receber();

    if (evento == null) {
        break;
    }

    processar(evento);
}
```

Em sistemas reais, loops infinitos sem controle podem consumir CPU, travar threads, impedir shutdown da aplicação ou causar indisponibilidade.

### Cuidado com alteração da coleção durante iteração

Este código pode causar problema:

```java
for (var musica : musicas) {
    if (musica.notaCuradoria() < 5) {
        musicas.remove(musica);
    }
}
```

Modificar uma coleção enquanto ela está sendo percorrida pode causar comportamento incorreto ou exceções, dependendo da coleção e da forma de iteração.

Alternativas melhores:

```java
musicas.removeIf(musica -> musica.notaCuradoria() < 5);
```

Ou criar uma nova lista filtrada.

### Evite lógica complexa demais dentro do loop

Loops muito grandes dificultam leitura, teste e manutenção.

Ruim:

```java
for (var musica : musicas) {
    // valida
    // transforma
    // acessa banco
    // chama API externa
    // monta resposta
    // calcula estatísticas
    // registra log
}
```

Melhor:

```java
for (var musica : musicas) {
    if (!deveProcessar(musica)) {
        continue;
    }

    processarMusica(musica);
}
```

### Evite concatenação excessiva de `String` dentro de loops

Em loops grandes, concatenação repetida de `String` pode gerar muitos objetos intermediários. Quando for montar texto progressivamente, prefira `StringBuilder`.

A documentação do Java SE 25 descreve `StringBuilder` como uma sequência mutável de caracteres, geralmente preferível a `StringBuffer` quando não há necessidade de sincronização. ([Oracle Docs][8])

```java
var relatorio = new StringBuilder();

for (var musica : musicas) {
    relatorio
            .append(musica.banda())
            .append(" - ")
            .append(musica.titulo())
            .append(System.lineSeparator());
}

System.out.println(relatorio);
```

## Erros comuns

### Esquecer de atualizar a variável de controle

```java
var indice = 0;

while (indice < musicas.size()) {
    System.out.println(musicas.get(indice).titulo());
}
```

Esse código nunca incrementa `indice`. Se a lista não estiver vazia, o loop será infinito.

Correção:

```java
var indice = 0;

while (indice < musicas.size()) {
    System.out.println(musicas.get(indice).titulo());
    indice++;
}
```

### Usar `<=` quando deveria usar `<`

```java
for (var indice = 0; indice <= musicas.size(); indice++) {
    System.out.println(musicas.get(indice));
}
```

Esse código tenta acessar o índice `musicas.size()`, que não existe. Em listas e arrays, os índices começam em `0` e vão até `size() - 1`.

Correção:

```java
for (var indice = 0; indice < musicas.size(); indice++) {
    System.out.println(musicas.get(indice));
}
```

### Usar `continue` antes de atualizar estado necessário

```java
var indice = 0;

while (indice < musicas.size()) {
    var musica = musicas.get(indice);

    if (musica.notaCuradoria() < 8) {
        continue;
    }

    indice++;
}
```

Se a primeira música tiver nota menor que 8, `continue` será executado antes do `indice++`, causando loop infinito.

Correção:

```java
var indice = 0;

while (indice < musicas.size()) {
    var musica = musicas.get(indice);
    indice++;

    if (musica.notaCuradoria() < 8) {
        continue;
    }

    System.out.println(musica.titulo());
}
```

### Usar `break` quando deveria usar `continue`

```java
for (var musica : musicas) {
    if (musica.notaCuradoria() < 8) {
        break;
    }

    System.out.println(musica.titulo());
}
```

Esse código para no primeiro item com nota menor que 8. Se a intenção era apenas ignorar essa música e continuar com as próximas, o correto seria:

```java
for (var musica : musicas) {
    if (musica.notaCuradoria() < 8) {
        continue;
    }

    System.out.println(musica.titulo());
}
```

### Usar `do/while` quando a execução não deveria acontecer nenhuma vez

```java
do {
    processarPagamento(pedido);
} while (pedido.precisaTentarNovamente());
```

Se existe a possibilidade de o pedido nem estar apto para pagamento, `do/while` pode ser perigoso, porque executa antes de validar a condição.

Nesse caso, `while` pode ser mais seguro:

```java
while (pedido.podeProcessarPagamento()) {
    processarPagamento(pedido);
}
```

## Decisões de engenharia

### Quando usar `for`

Use `for` quando a quantidade de repetições ou a progressão estiver bem definida.

Bom uso:

```java
for (var indice = 0; indice < musicas.size(); indice++) {
    var musica = musicas.get(indice);
    System.out.println(musica.titulo());
}
```

Use especialmente quando:

* você precisa do índice;
* existe início, fim e incremento claros;
* o loop representa uma contagem;
* você precisa comparar elementos vizinhos.

Evite quando o índice não agrega valor. Nesse caso, enhanced `for` é mais legível.

### Quando usar enhanced `for`

Use enhanced `for` quando você só precisa percorrer os elementos.

```java
for (var musica : musicas) {
    System.out.println(musica.titulo());
}
```

Ele melhora a legibilidade e reduz erros de índice.

Evite quando:

* você precisa do índice;
* precisa percorrer de trás para frente;
* precisa alterar posições específicas;
* precisa controlar o avanço manualmente.

### Quando usar `while`

Use `while` quando a repetição depende de um estado externo ou dinâmico.

```java
while (!fila.isEmpty()) {
    processar(fila.removeFirst());
}
```

É comum em:

* filas;
* consumidores;
* polling;
* leitura até fim de arquivo;
* tentativas com condição de parada;
* processos que dependem de estado mutável.

Evite quando um `for` comunicaria melhor a intenção.

### Quando usar `do/while`

Use `do/while` quando o bloco precisa executar pelo menos uma vez.

Bom exemplo:

```java
do {
    pagina = buscarPagina(numeroPagina);
    processar(pagina.itens());
    numeroPagina++;
} while (pagina.temProxima());
```

Evite quando a primeira execução depende de uma validação prévia.

### Quando usar `break`

Use `break` para parada antecipada quando continuar o loop não agrega mais valor.

Exemplos:

* encontrou o item procurado;
* atingiu um limite;
* detectou uma condição de erro;
* completou o objetivo do loop.

Evite `break` espalhado em loops longos. Se houver muitos pontos de saída, extraia o loop para um método e use `return`.

### Quando usar `continue`

Use `continue` para reduzir aninhamento e ignorar casos que não devem ser processados.

Sem `continue`:

```java
for (var musica : musicas) {
    if (musica.notaCuradoria() >= 8) {
        processar(musica);
    }
}
```

Com `continue`:

```java
for (var musica : musicas) {
    if (musica.notaCuradoria() < 8) {
        continue;
    }

    processar(musica);
}
```

A segunda forma costuma ficar melhor quando existem várias validações de descarte no início do loop.

### Alternativas aos loops

Nem todo processamento precisa ser feito com loop explícito.

Em Java moderno, você também pode usar:

```java
var destaques = musicas.stream()
        .filter(musica -> musica.notaCuradoria() >= 8)
        .toList();
```

Streams podem melhorar a legibilidade quando a intenção é transformar, filtrar ou agregar dados.

Mas loops explícitos ainda são ótimos quando:

* existe controle de fluxo com `break`;
* existe atualização de múltiplos estados;
* a regra é imperativa;
* performance e alocação precisam ser analisadas com cuidado;
* o código ficaria artificial usando Stream.

## Onde isso aparece em sistemas reais

Estruturas de repetição aparecem em praticamente qualquer backend Java.

Em APIs REST:

```java
for (var item : request.itens()) {
    validar(item);
}
```

Em importação de arquivos:

```java
while ((linha = leitor.readLine()) != null) {
    processarLinha(linha);
}
```

Em paginação:

```java
do {
    resposta = cliente.buscarPagina(pagina);
    salvar(resposta.itens());
    pagina++;
} while (resposta.temProxima());
```

Em filas:

```java
while (!fila.isEmpty()) {
    processar(fila.removeFirst());
}
```

Em regras de negócio:

```java
for (var musica : catalogo) {
    if (!musicaPodeSerPublicada(musica)) {
        continue;
    }

    publicar(musica);
}
```

Em busca com parada antecipada:

```java
for (var musica : catalogo) {
    if (musica.titulo().equalsIgnoreCase(tituloBuscado)) {
        encontrada = musica;
        break;
    }
}
```

Em sistemas reais, loops precisam ser pensados não apenas pela sintaxe, mas também por impacto operacional:

* quantos itens podem ser processados?
* há risco de loop infinito?
* existe timeout?
* há paginação?
* existe limite de memória?
* o loop chama banco ou API externa?
* o processamento precisa ser interrompível?
* o código é legível para manutenção futura?

## Perguntas de revisão

### 1 - O que são estruturas de repetição?

Resposta: são comandos da linguagem usados para executar um bloco de código várias vezes. Em Java, as principais estruturas são `for`, `while` e `do/while`.

### 2 - Qual problema real os loops resolvem?

Resposta: eles evitam duplicação de código quando precisamos processar vários itens ou repetir uma operação enquanto uma condição for verdadeira. Por exemplo, percorrer todas as músicas de um catálogo e aplicar uma regra de curadoria.

### 3 - Quando usar `for` tradicional?

Resposta: use `for` tradicional quando existe controle claro por índice, contador ou limite. Exemplo: percorrer uma lista usando posições ou executar uma operação exatamente 10 vezes.

### 4 - Quando usar enhanced `for`?

Resposta: use enhanced `for` quando você precisa apenas acessar cada elemento de uma coleção ou array, sem precisar do índice.

### 5 - Qual é a diferença entre `for` tradicional e enhanced `for`?

Resposta: o `for` tradicional dá controle sobre inicialização, condição e atualização. O enhanced `for` abstrai esse controle e entrega diretamente cada elemento da coleção.

### 6 - Quando usar `while`?

Resposta: use `while` quando a repetição depende de uma condição que pode mudar durante a execução. Exemplo: processar uma fila enquanto ela não estiver vazia.

### 7 - Quando usar `do/while`?

Resposta: use `do/while` quando o bloco precisa executar pelo menos uma vez antes da condição ser verificada. Um exemplo comum é buscar a primeira página de uma API e continuar enquanto houver próxima página.

### 8 - Qual é a diferença entre `while` e `do/while`?

Resposta: o `while` verifica a condição antes de executar o bloco. O `do/while` executa o bloco primeiro e só depois verifica a condição.

### 9 - O que o `break` faz?

Resposta: o `break` interrompe o loop imediatamente e continua a execução do programa após o bloco de repetição.

### 10 - O que o `continue` faz?

Resposta: o `continue` pula o restante da iteração atual e passa para a próxima avaliação do loop.

### 11 - Qual é a diferença entre `break` e `continue`?

Resposta: `break` encerra o loop inteiro. `continue` encerra apenas a iteração atual e permite que o loop continue.

### 12 - Por que loops infinitos são perigosos?

Resposta: porque podem consumir CPU indefinidamente, travar uma thread, impedir encerramento da aplicação e causar indisponibilidade. Loops infinitos só devem existir quando há controle explícito de parada, pausa, timeout ou interrupção.

### 13 - O que acontece se eu esquecer de incrementar uma variável em um `while`?

Resposta: o loop pode nunca terminar. Se a condição depende dessa variável e ela não muda, o programa fica preso repetindo o mesmo bloco.

### 14 - Por que `indice <= lista.size()` costuma estar errado?

Resposta: porque listas e arrays em Java começam no índice `0` e terminam em `size() - 1`. Usar `<= lista.size()` tenta acessar uma posição inexistente.

### 15 - `var` muda o tipo da variável em tempo de execução?

Resposta: não. `var` apenas permite que o compilador infira o tipo em tempo de compilação. Java continua sendo uma linguagem estaticamente tipada.

### 16 - Enhanced `for` substitui completamente o `for` tradicional?

Resposta: não. Enhanced `for` é melhor para percorrer elementos diretamente, mas o `for` tradicional continua necessário quando você precisa de índice, controle de direção, incremento customizado ou acesso a posições específicas.

### 17 - Streams substituem loops?

Resposta: não completamente. Streams são ótimas para operações declarativas como filtro, transformação e agregação. Loops explícitos ainda são melhores quando existe controle de fluxo com `break`, atualização de múltiplos estados ou lógica imperativa mais complexa.

### 18 - O `break` dentro de um loop aninhado interrompe todos os loops?

Resposta: não. Por padrão, ele interrompe apenas o loop mais interno. Para sair de múltiplos níveis, é possível usar labels, mas geralmente é melhor extrair a lógica para um método e usar `return`.

### 19 - É possível usar mais de uma variável no `for` tradicional?

Resposta: sim. O `for` tradicional permite múltiplas expressões na inicialização e na atualização, separadas por vírgula. Isso pode ser útil quando duas variáveis fazem parte da mesma lógica de controle, como uma avançando do início da lista e outra voltando do final.

Exemplo:

```java
for (int inicio = 0, fim = musicas.size() - 1;
     inicio < fim;
     inicio++, fim--) {

    comparar(musicas.get(inicio), musicas.get(fim));
}
```

Nesse caso, `inicio` e `fim` são declaradas na mesma declaração local e possuem o mesmo tipo: `int`.

Se as variáveis forem de tipos diferentes, elas devem ser declaradas separadamente antes do `for`:

```java
int inicio = 0;
long fim = musicas.size() - 1L;

for (; inicio < musicas.size(); inicio++, fim--) {
    // lógica do loop
}
```

Também é importante lembrar que `var` não pode ser usado com múltiplos declaradores na mesma declaração. Portanto, `for (var inicio = 0, fim = 9; ... )` não é válido em Java.


## Resumo final

Estruturas de repetição permitem executar código várias vezes sem duplicação manual.

O `for` tradicional é indicado quando existe contador, índice ou limite claro. O enhanced `for` é ideal para percorrer elementos de arrays e coleções com mais legibilidade. O `while` funciona bem quando a repetição depende de uma condição dinâmica. O `do/while` é útil quando a primeira execução precisa acontecer antes da verificação da condição.

O `break` encerra o loop antecipadamente. O `continue` pula apenas a iteração atual. Ambos são úteis, mas precisam ser usados com cuidado para não tornar o fluxo difícil de entender.

Em sistemas backend, loops aparecem em validações, paginação, filas, importações, integrações, processamento em lote e regras de negócio. Dominar essas estruturas não é apenas conhecer sintaxe: é saber escolher a forma mais clara, segura e eficiente para expressar uma repetição.

## Referências

* Java Language Specification, Java SE 25 Edition — seções sobre `while`, `do`, `for`, `break` e `continue`. ([Oracle Docs][2])
* Java Virtual Machine Specification, Java SE 25 Edition — instruções de transferência de controle. ([Oracle Docs][7])
* Java SE 25 API Documentation — `StringBuilder`. ([Oracle Docs][8])
* OpenJDK — JEP 286: Local-Variable Type Inference, Java 10. ([openjdk.org][4])
* OpenJDK — Local Variable Type Inference Style Guidelines. ([openjdk.org][6])
* Java Community Process — JSR 201, Enhanced `for` loop, Java 5. ([jcp.org][5])
* OpenJDK — JEP 395: Records, Java 16. ([openjdk.org][3])
* Joshua Bloch — *Effective Java*, boas práticas de clareza, controle de fluxo, imutabilidade e desenho de APIs.
* Brian Goetz et al. — *Java Concurrency in Practice*, especialmente para entender riscos de loops em código concorrente, threads e processamento contínuo.

[1]: https://www.oracle.com/java/technologies/downloads/ "Java Downloads"
[2]: https://docs.oracle.com/javase/specs/jls/se25/html/index.html "The Java® Language Specification"
[3]: https://openjdk.org/jeps/395 "JEP 395: Records"
[4]: https://openjdk.org/jeps/286 "JEP 286: Local-Variable Type Inference"
[5]: https://www.jcp.org/en/jsr/detail?id=201 "The Java Community Process(SM) Program - JSRs: Java Specification Requests - detail JSR# 201"
[6]: https://openjdk.org/projects/amber/guides/lvti-style-guide "Local Variable Type Inference: Style Guidelines"
[7]: https://docs.oracle.com/javase/specs/jvms/se25/html/index.html "The Java® Virtual Machine Specification"
[8]: https://docs.oracle.com/en/java/javase/25/docs/api//java.base/java/lang/StringBuilder.html "StringBuilder (Java SE 25 & JDK 25)"
