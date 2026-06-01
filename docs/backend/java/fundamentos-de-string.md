# Fundamentos de String em Java

Praticamente todo sistema backend manipula texto o tempo inteiro: nomes de usuários, e-mails, mensagens de erro, logs, respostas HTTP, payloads JSON, comandos SQL, templates de notificação, parâmetros de busca, valores vindos de formulários e dados recebidos de outros sistemas.

Em uma aplicação fictícia como **meucatalogomusical.com**, por exemplo, textos aparecem em vários pontos:

- nome de uma banda;
- título de uma música;
- descrição de um álbum;
- mensagem exibida em uma API;
- log de uma operação;
- JSON retornado para um frontend;
- entrada digitada em um terminal durante um teste simples.

Por isso, entender `String` em Java não é apenas saber “colocar texto entre aspas”. É entender como o Java representa texto, como concatena valores, como interpreta caracteres especiais, como formata saídas e quais armadilhas podem aparecer em código real.

A documentação oficial define `String` como a classe que representa sequências de caracteres, e todos os literais de string escritos no código, como `"abc"`, são instâncias dessa classe. Também é importante saber que objetos `String` são imutáveis: depois de criados, seus valores não mudam. :contentReference[oaicite:0]{index=0}

## Antes de entrar no código: uma ideia simples

Uma `String` é um texto tratado como valor dentro do programa.

Quando escrevemos:

```java
String banda = "Dream Theater";
```

estamos dizendo:

> A variável `banda` referencia um objeto do tipo `String` que representa o texto `Dream Theater`.

Isso é diferente de tipos primitivos como `int`, `double`, `boolean` e `char`. `String` começa com letra maiúscula porque é uma classe, não um tipo primitivo.

De forma simples:

```java
int ano = 1986;
String banda = "Europe";
```

`ano` guarda um valor numérico primitivo.
`banda` guarda uma referência para um objeto que representa texto.

## Definição técnica

Em Java, `String` é uma classe final do pacote `java.lang`, usada para representar sequências de caracteres. Por estar em `java.lang`, ela não precisa ser importada manualmente.

Um literal de string é qualquer sequência de caracteres entre aspas duplas:

```java
String genero = "Rock Progressivo";
```

Pela especificação da linguagem, um literal de string consiste em zero ou mais caracteres entre aspas duplas e é sempre do tipo `String`. A JLS também define que quebras de linha diretas não podem aparecer dentro de um literal comum de string; quando uma quebra de linha é necessária, usamos uma sequência de escape, como `\n`, ou um text block. ([Oracle Docs][1])

Tecnicamente, isso significa que:

```java
String texto = "Metal";
```

não cria um valor primitivo. Cria, ou reutiliza, um objeto `String`.

Além disso, literais de string são internados. Isso significa que strings literais iguais podem compartilhar a mesma instância no pool de strings mantido pela JVM. A especificação define que literais de string e expressões constantes de string são internados, funcionando como se `String.intern()` tivesse sido usado. ([Oracle Docs][1])

## Como funciona em Java

### Criando strings

A forma mais comum de criar uma `String` é usando aspas duplas:

```java
String banda = "Black Sabbath";
String musica = "Paranoid";
```

Também é possível criar uma `String` com `new`, mas normalmente isso deve ser evitado:

```java
String banda = new String("Black Sabbath");
```

Esse segundo formato força a criação de um novo objeto desnecessário na maioria dos casos. Em código real, prefira literais ou valores vindos de entrada, banco, APIs e serviços.

### Concatenando strings

O operador `+` pode somar números ou concatenar textos.

```java
int musicasCadastradas = 10;
int musicasImportadas = 5;

System.out.println("Total: " + musicasCadastradas + musicasImportadas);
```

A saída será:

```text
Total: 105
```

Isso acontece porque a expressão é avaliada da esquerda para a direita. Quando o Java encontra uma `String` primeiro, o operador `+` passa a trabalhar como concatenação. A JLS define que o operador `+` é associativo à esquerda; por isso, `a + b + c` é interpretado como `(a + b) + c`. ([Oracle Docs][2])

Para obter a soma antes da concatenação, use parênteses:

```java
System.out.println("Total: " + (musicasCadastradas + musicasImportadas));
```

Saída:

```text
Total: 15
```

Também funcionaria assim:

```java
System.out.println(musicasCadastradas + musicasImportadas + " músicas no total");
```

Saída:

```text
15 músicas no total
```

Nesse caso, os dois primeiros operandos são numéricos, então o Java soma antes de encontrar a `String`.

### Sequências de escape

Sequências de escape são formas de representar caracteres especiais dentro de literais de texto.

Exemplo:

```java
String mensagem = "Banda \"Rush\" cadastrada com sucesso.";
```

A saída será:

```text
Banda "Rush" cadastrada com sucesso.
```

Sem a barra invertida antes das aspas internas, o compilador entenderia que a string terminou antes da hora.

A JLS define as sequências de escape para literais de caractere, literais de string e text blocks. Elas incluem `\b`, `\s`, `\t`, `\n`, `\f`, `\r`, `\"`, `\'`, `\\`, escapes octais e continuação de linha com `\` seguido de terminador de linha. ([Oracle Docs][1])

Tabela prática:

| Escape                | Significado           | Uso comum                                                                       |
| --------------------- | --------------------- | ------------------------------------------------------------------------------- |
| `\"`                  | aspas duplas          | escrever aspas dentro de uma string                                             |
| `\'`                  | aspas simples         | mais comum em `char`, mas válido em string                                      |
| `\\`                  | barra invertida       | caminhos, regex, textos com `\`                                                 |
| `\n`                  | nova linha, line feed | quebra de linha lógica                                                          |
| `\r`                  | carriage return       | compatibilidade com formatos antigos/Windows                                    |
| `\t`                  | tabulação horizontal  | alinhamento simples                                                             |
| `\b`                  | backspace             | raro em aplicações backend                                                      |
| `\f`                  | form feed             | raro, usado historicamente em impressão                                         |
| `\s`                  | espaço                | útil especialmente em text blocks, introduzido junto aos ajustes de text blocks |
| `\` + quebra de linha | continuação de linha  | evita inserir a quebra no valor final                                           |
| `\0` até `\377`       | escape octal          | raro em código moderno                                                          |

O `\\` não “vira uma barra” por mágica. Ele representa um único caractere de barra invertida no valor final da string. O primeiro `\` informa ao compilador que o próximo caractere faz parte de uma sequência especial; o segundo `\` é o caractere desejado.

Exemplo:

```java
String caminho = "C:\\catalogo\\bandas\\sabbath.json";
System.out.println(caminho);
```

Saída:

```text
C:\catalogo\bandas\sabbath.json
```

### Text blocks

A partir do Java 15, text blocks se tornaram um recurso padrão da linguagem por meio da JEP 378. Eles permitem escrever strings multilinha com `"""`, evitando muitas concatenações e escapes desnecessários. ([OpenJDK][3])

Exemplo:

```java
String json = """
        {
          "banda": "Iron Maiden",
          "genero": "Heavy Metal",
          "ativa": true
        }
        """;
```

Isso é muito útil para exemplos de JSON, SQL, HTML, mensagens longas e fixtures de teste.

Text blocks também são do tipo `String`. A JLS define que um text block pode ser usado onde uma expressão `String` é aceita, e que seu conteúdo passa por processamento de normalização de quebras de linha, remoção de indentação incidental e interpretação de escapes. ([Oracle Docs][1])

### `print`, `println`, `printf` e `format`

`System.out` é um `PrintStream`. Essa classe possui métodos para imprimir valores de forma conveniente, incluindo `print`, `println`, `printf` e `format`. A documentação oficial mostra sobrecargas para tipos como `boolean`, `char`, `int`, `long`, `float`, `double`, `String` e `Object`. ([Oracle Docs][4])

Principais diferenças:

| Método                   | Quebra linha automaticamente? | Formata com `%s`, `%d`, `%f`? | Uso comum                               |
| ------------------------ | ----------------------------: | ----------------------------: | --------------------------------------- |
| `print`                  |                           Não |                           Não | imprimir partes de uma linha            |
| `println`                |                           Sim |                           Não | imprimir uma linha simples              |
| `printf`                 |                           Não |                           Sim | imprimir saída formatada                |
| `format`                 |                           Não |                           Sim | equivalente prático ao `printf`         |
| `String.format`          |                   Não imprime |                           Sim | criar uma string formatada              |
| `"texto".formatted(...)` |                   Não imprime |                           Sim | forma fluente de criar string formatada |

`println` imprime o valor e termina a linha. A documentação informa que `println()` escreve o separador de linha definido pelo sistema, que não necessariamente é apenas `\n`. ([Oracle Docs][4])

`printf` é um método de conveniência para escrever uma string formatada e funciona como `format`. A documentação mostra que `out.printf(format, args)` se comporta como `out.format(format, args)`. ([Oracle Docs][4])

### Conversões do `printf`

O `printf` usa a infraestrutura de `java.util.Formatter`, que interpreta strings de formatação no estilo `printf`. O `Formatter` suporta alinhamento, largura, formatos numéricos, strings, datas, horas e saída dependente de `Locale`. ([Oracle Docs][5])

Sintaxe geral:

```text
%[índice_do_argumento$][flags][largura][.precisão]conversão
```

A documentação oficial descreve essa estrutura com partes opcionais como índice do argumento, flags, largura, precisão e conversão. ([Oracle Docs][5])

Conversões mais úteis:

| Conversão         | Significado                        | Exemplo                             |
| ----------------- | ---------------------------------- | ----------------------------------- |
| `%s`              | string                             | nome, descrição, status             |
| `%S`              | string em maiúsculas               | saída textual destacada             |
| `%d`              | inteiro decimal                    | quantidade, código, ano             |
| `%f`              | ponto flutuante decimal            | peso, média, percentual             |
| `%.2f`            | ponto flutuante com 2 casas        | valores monetários simples, medidas |
| `%10.2f`          | largura mínima 10, 2 casas         | alinhamento em relatórios simples   |
| `%b`              | booleano                           | `true` ou `false`                   |
| `%c`              | caractere                          | um caractere Unicode                |
| `%x` / `%X`       | inteiro em hexadecimal             | debug técnico                       |
| `%o`              | inteiro em octal                   | raro                                |
| `%e` / `%E`       | notação científica                 | números muito grandes/pequenos      |
| `%g` / `%G`       | formato geral para ponto flutuante | saída compacta                      |
| `%t...` / `%T...` | data/hora                          | exige sufixo, como `%tF`            |
| `%%`              | caractere `%` literal              | imprimir porcentagem                |
| `%n`              | separador de linha da plataforma   | quebra de linha portável            |

A documentação divide as conversões em categorias: gerais, caractere, numéricas, data/hora, percentual e separador de linha. Também define `%n` como o separador de linha específico da plataforma. ([Oracle Docs][5]) ([Oracle Docs][5])

Exemplo:

```java
import java.util.Locale;

public class RelatorioCatalogoMusical {

    public static void main(String[] args) {
        Locale locale = Locale.US;

        String banda = "Metallica";
        String album = "Master of Puppets";
        int anoLancamento = 1986;
        int faixas = 8;
        double avaliacaoMedia = 9.75;
        boolean classico = true;

        System.out.printf(locale,
                "Banda: %s%nÁlbum: %s%nAno: %d%nFaixas: %d%nAvaliação: %.2f%nClássico: %b%n",
                banda, album, anoLancamento, faixas, avaliacaoMedia, classico);
    }
}
```

Saída:

```text
Banda: Metallica
Álbum: Master of Puppets
Ano: 1986
Faixas: 8
Avaliação: 9.75
Clássico: true
```

O uso de `Locale.US` evita surpresas na representação decimal. Em alguns ambientes, a localidade padrão pode trocar ponto por vírgula em números formatados.

### `Scanner` e entrada de dados

`Scanner` é uma classe simples para ler e converter texto em tokens. A documentação define `Scanner` como um leitor capaz de analisar tipos primitivos e strings usando expressões regulares. Por padrão, ele separa tokens usando espaços em branco. ([Oracle Docs][6])

Principais métodos:

| Método              | O que lê                                                   |
| ------------------- | ---------------------------------------------------------- |
| `next()`            | próximo token como `String`                                |
| `nextLine()`        | restante da linha atual                                    |
| `nextInt()`         | próximo token como `int`                                   |
| `nextLong()`        | próximo token como `long`                                  |
| `nextDouble()`      | próximo token como `double`                                |
| `nextFloat()`       | próximo token como `float`                                 |
| `nextBoolean()`     | próximo token como `boolean`                               |
| `nextByte()`        | próximo token como `byte`                                  |
| `nextShort()`       | próximo token como `short`                                 |
| `nextBigInteger()`  | próximo token como `BigInteger`                            |
| `nextBigDecimal()`  | próximo token como `BigDecimal`                            |
| `hasNext...()`      | verifica se o próximo token pode ser lido no tipo desejado |
| `useDelimiter(...)` | altera o delimitador                                       |
| `findInLine(...)`   | procura padrão na linha atual                              |
| `skip(...)`         | avança ignorando um padrão                                 |

A documentação lista métodos como `nextInt`, `nextLong`, `nextFloat`, `nextDouble`, `nextBigInteger`, `nextBigDecimal` e suas versões `hasNext...`. ([Oracle Docs][6]) ([Oracle Docs][6])

Um detalhe importante: `nextLine()` lê o restante da linha atual, enquanto métodos como `nextInt()` e `nextDouble()` trabalham por token. A documentação descreve que `nextLine()` avança o scanner até o fim da linha e retorna o conteúdo ignorado, excluindo o separador de linha. ([Oracle Docs][6])

Por isso, misturar `nextInt()` com `nextLine()` pode causar surpresa:

```java
import java.util.Scanner;

public class CadastroTerminal {

    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);

        System.out.print("Ano de lançamento: ");
        int ano = scanner.nextInt();

        scanner.nextLine();

        System.out.print("Nome da banda: ");
        String banda = scanner.nextLine();

        System.out.printf("Banda cadastrada: %s (%d)%n", banda, ano);
    }
}
```

A chamada extra a `scanner.nextLine()` consome o restante da linha deixado após a leitura do número.

Em aplicações backend reais, `Scanner` é mais comum em programas de terminal, scripts, exercícios, ferramentas internas simples e exemplos didáticos. Em APIs web, os dados normalmente chegam via HTTP, JSON, parâmetros de rota, query parameters, headers, filas ou eventos.

## Exemplo prático

O exemplo abaixo usa `String`, concatenação, sequências de escape, text block, `String.formatted`, `printf` e `Scanner` em um cenário simples do **meucatalogomusical.com**.

```java
import java.math.BigDecimal;
import java.util.Locale;
import java.util.Scanner;

public class CadastroMusicaTerminal {

    public static void main(String[] args) {
        Locale locale = Locale.US;
        Scanner scanner = new Scanner(System.in);

        System.out.print("Banda: ");
        String banda = scanner.nextLine();

        System.out.print("Música: ");
        String musica = scanner.nextLine();

        System.out.print("Ano de lançamento: ");
        int anoLancamento = scanner.nextInt();

        System.out.print("Duração em minutos: ");
        double duracaoMinutos = scanner.nextDouble();

        System.out.print("Preço digital: ");
        BigDecimal precoDigital = scanner.nextBigDecimal();

        scanner.nextLine();

        String resumo = "Cadastro recebido: " + banda + " - \"" + musica + "\"";

        String mensagemFormatada = """
                Música cadastrada no meucatalogomusical.com
                Banda: %s
                Música: "%s"
                Ano: %d
                Duração: %.2f minutos
                Preço digital: R$ %s
                """.formatted(banda, musica, anoLancamento, duracaoMinutos, precoDigital);

        System.out.println();
        System.out.println(resumo);
        System.out.println();

        System.out.printf(locale,
                "Resumo técnico: banda=%s | musica=%s | ano=%d | duracao=%.2f%n",
                banda, musica, anoLancamento, duracaoMinutos);

        System.out.println();
        System.out.println(mensagemFormatada);
    }
}
```

## Analisando o código

O programa começa importando três classes:

```java
import java.math.BigDecimal;
import java.util.Locale;
import java.util.Scanner;
```

`BigDecimal` representa valores decimais com maior controle, especialmente útil quando o assunto envolve dinheiro. `Locale` controla regras de formatação regional. `Scanner` lê dados do terminal.

Depois, o programa cria o `Scanner`:

```java
Scanner scanner = new Scanner(System.in);
```

`System.in` representa a entrada padrão do processo. Em um terminal, normalmente é o teclado.

As leituras de texto usam `nextLine()`:

```java
String banda = scanner.nextLine();
String musica = scanner.nextLine();
```

Isso permite ler valores com espaços, como `Dream Theater` ou `Smoke on the Water`.

As leituras numéricas usam métodos específicos:

```java
int anoLancamento = scanner.nextInt();
double duracaoMinutos = scanner.nextDouble();
BigDecimal precoDigital = scanner.nextBigDecimal();
```

Aqui, cada método tenta converter o próximo token para o tipo esperado. Se a entrada não for compatível, o `Scanner` pode lançar `InputMismatchException`.

Depois da última leitura por token, aparece:

```java
scanner.nextLine();
```

Essa linha consome o restante da linha atual, evitando problemas caso outra leitura com `nextLine()` seja adicionada depois.

A variável `resumo` mostra concatenação:

```java
String resumo = "Cadastro recebido: " + banda + " - \"" + musica + "\"";
```

As aspas ao redor do nome da música usam `\"`, porque aspas duplas têm significado especial dentro de literais de string.

Depois, o programa usa text block com `formatted`:

```java
String mensagemFormatada = """
        Música cadastrada no meucatalogomusical.com
        Banda: %s
        Música: "%s"
        Ano: %d
        Duração: %.2f minutos
        Preço digital: R$ %s
        """.formatted(banda, musica, anoLancamento, duracaoMinutos, precoDigital);
```

O método `String.formatted`, disponível desde Java 15, formata a própria string usando os argumentos informados. A documentação informa que ele é equivalente a `String.format(this, args)`. ([Oracle Docs][7])

Por fim, o código usa `printf`:

```java
System.out.printf(locale,
        "Resumo técnico: banda=%s | musica=%s | ano=%d | duracao=%.2f%n",
        banda, musica, anoLancamento, duracaoMinutos);
```

O `Locale.US` ajuda a manter o ponto como separador decimal na saída. O `%n` quebra a linha de forma portável.

## O que acontece por baixo dos panos

### `String` é imutável

Quando você escreve:

```java
String nome = "Rush";
nome = nome + " - Progressivo";
```

o texto `"Rush"` não é alterado. O que acontece é que uma nova `String` é produzida com o conteúdo `"Rush - Progressivo"`, e a variável `nome` passa a referenciar essa nova string.

A imutabilidade é uma das características mais importantes de `String`. Ela facilita compartilhamento, segurança, uso em mapas, uso em caches e previsibilidade em código concorrente.

### Literais e pool de strings

Strings literais iguais podem ser compartilhadas no pool de strings:

```java
String a = "Queen";
String b = "Queen";

System.out.println(a == b);      // true, em geral por interning de literal
System.out.println(a.equals(b)); // true
```

Mas strings criadas em tempo de execução podem ser objetos diferentes, mesmo com o mesmo conteúdo:

```java
String a = "Queen";
String b = "Que" + "en";
String c = "Que";
String d = c + "en";

System.out.println(a == b);      // true, expressão constante
System.out.println(a == d);      // false, calculada em runtime
System.out.println(a.equals(d)); // true
```

A JLS explica que expressões constantes de string podem ser computadas em tempo de compilação e tratadas como literais, enquanto strings calculadas em tempo de execução são recém-criadas e podem ser distintas. ([Oracle Docs][1])

Regra prática:

* use `equals` para comparar conteúdo;
* não use `==` para comparar texto, exceto quando você realmente quer comparar identidade de objeto.

### Representação interna e Compact Strings

Desde o Java 9, a JEP 254 mudou a representação interna de `String` de um array de `char` para um array de `byte` mais uma marca de codificação. A ideia é armazenar caracteres em Latin-1 com um byte quando possível, ou UTF-16 quando necessário, sem alterar a API pública. ([OpenJDK][8])

Isso importa porque strings costumam ocupar uma parte relevante da heap em aplicações reais. Logs, nomes, mensagens, payloads, chaves de cache e dados textuais em geral podem gerar muitas instâncias de `String`.

### Concatenação e `invokedynamic`

Antes do Java 9, era comum explicar concatenação dizendo que o compilador transformava `+` em operações com `StringBuilder`. Essa explicação ainda ajuda como modelo mental, mas ficou incompleta para versões modernas.

A partir do Java 9, a JEP 280 alterou a forma como o `javac` gera bytecode para concatenação de strings, passando a usar `invokedynamic` e funções de biblioteca do JDK. O objetivo foi permitir otimizações futuras sem exigir mudanças frequentes no compilador. ([OpenJDK][9])

Na prática:

```java
String mensagem = "Banda: " + banda + ", música: " + musica;
```

continua sendo simples e aceitável para concatenações pequenas e legíveis. Para concatenações repetidas dentro de loops grandes, `StringBuilder` ainda pode ser melhor.

### `length()` conta unidades UTF-16, não necessariamente “caracteres visuais”

A documentação de `String.length()` informa que o tamanho retornado é o número de unidades de código Unicode na string. ([Oracle Docs][7])

Isso é importante porque nem todo símbolo visual cabe em uma única unidade UTF-16.

Em sistemas reais, isso afeta validações de tamanho, campos com emojis, normalização Unicode e integrações que limitam bytes ou caracteres de forma diferente.

### `printf` usa `Formatter`

Quando usamos:

```java
System.out.printf("Banda: %s%n", banda);
```

o Java usa a infraestrutura de `Formatter`. O formato pode lançar exceções se a string de formatação estiver inválida, se faltar argumento ou se uma conversão for incompatível com o tipo passado. A documentação de `Formatter` informa que métodos de formatação podem lançar `IllegalFormatException` nessas situações. ([Oracle Docs][5])

Exemplo de erro:

```java
System.out.printf("Ano: %d%n", "1986");
```

`%d` espera um tipo numérico inteiro compatível, não uma `String`.

## Comparação com versões anteriores do Java

### Java 1.0: `String` já existia

`String` é uma classe fundamental desde o início da plataforma Java. Literais de string, concatenação com `+`, `print` e `println` fazem parte da base histórica da linguagem.

### Java 1.5: `Formatter`, `printf` e `String.format`

A API de formatação no estilo `printf`, incluindo `Formatter`, `PrintStream.printf` e `String.format`, existe desde Java 5. A documentação marca `Formatter` e métodos relacionados como disponíveis desde 1.5. ([Oracle Docs][5])

Antes disso, era mais comum ver concatenações manuais ou uso de classes como `NumberFormat` e `MessageFormat`.

### Java 9: Compact Strings e nova concatenação

No Java 9, duas mudanças relevantes aconteceram:

* JEP 254: Compact Strings, mudando a representação interna para economizar memória em muitos cenários;
* JEP 280: Indify String Concatenation, mudando a estratégia de bytecode para concatenação com `+`. ([OpenJDK][8]) ([OpenJDK][9])

Essas mudanças não alteraram como você escreve código no dia a dia, mas mudaram detalhes importantes de performance e implementação.

### Java 15: Text blocks e `String.formatted`

Text blocks viraram recurso padrão no Java 15 pela JEP 378, ajudando a escrever strings multilinha com menos ruído. ([OpenJDK][3])

Também desde Java 15, `String.formatted` permite formatar a própria string de maneira fluente:

```java
String mensagem = "Banda %s cadastrada".formatted("Rush");
```

A documentação oficial informa que `formatted` existe desde Java 15. ([Oracle Docs][7])

### Java 21 e 22: String Templates em preview

String Templates apareceram como preview no Java 21 pela JEP 430 e foram reavaliados no Java 22 pela JEP 459. Porém, a documentação de mudanças da linguagem informa que o recurso foi retirado antes do Java 23, por falta de consenso sobre o desenho adequado. Portanto, em Java 25, não trate String Templates como recurso padrão da linguagem. ([Oracle Docs][10])

Na prática, para Java 25, continue usando:

* concatenação simples com `+`;
* `StringBuilder` para montagem repetitiva;
* `String.format`;
* `String.formatted`;
* `MessageFormat` quando precisar de mensagens internacionalizadas;
* template engines em camadas de apresentação, quando fizer sentido.

## Boas práticas

### Prefira clareza em concatenações simples

Para mensagens pequenas, concatenação é aceitável:

```java
String mensagem = "Banda " + banda + " cadastrada com sucesso";
```

Evite transformar tudo em `printf` ou `formatted` quando a concatenação for mais clara.

### Use `formatted` ou `String.format` para mensagens com muitos valores

Quando a string tem vários valores interpolados, formatação pode melhorar a leitura:

```java
String mensagem = "Banda %s cadastrada com %d músicas".formatted(banda, totalMusicas);
```

### Use `StringBuilder` em concatenações repetidas

Evite isto em loops grandes:

```java
String resultado = "";

for (String banda : bandas) {
    resultado += banda + "\n";
}
```

Prefira:

```java
StringBuilder builder = new StringBuilder();

for (String banda : bandas) {
    builder.append(banda).append(System.lineSeparator());
}

String resultado = builder.toString();
```

### Use `%n` em vez de `\n` em formatações

Em strings de formatação, prefira:

```java
System.out.printf("Banda: %s%n", banda);
```

em vez de:

```java
System.out.printf("Banda: %s\n", banda);
```

`%n` usa o separador de linha da plataforma, enquanto `\n` representa line feed.

### Defina `Locale` quando formatar números

Para logs técnicos, testes e formatos estáveis, use um `Locale` explícito:

```java
System.out.printf(Locale.US, "Duração: %.2f%n", duracao);
```

Sem isso, a saída pode variar conforme a configuração regional do ambiente.

### Use `equals` para comparar conteúdo

Correto:

```java
if ("Metal".equals(genero)) {
    System.out.println("Gênero encontrado");
}
```

Evite:

```java
if (genero == "Metal") {
    System.out.println("Gênero encontrado");
}
```

O operador `==` compara referências. `equals` compara conteúdo.

### Valide entradas antes de converter

Com `Scanner`, prefira verificar antes:

```java
if (scanner.hasNextInt()) {
    int ano = scanner.nextInt();
} else {
    System.out.println("Ano inválido");
}
```

Os métodos `hasNext...` permitem verificar se o próximo token pode ser interpretado no tipo desejado sem avançar a entrada. ([Oracle Docs][6])

## Erros comuns

### Achar que `String` é tipo primitivo

`String` é uma classe. Isso explica por que ela tem métodos:

```java
String banda = "Rush";

System.out.println(banda.length());
System.out.println(banda.toUpperCase());
System.out.println(banda.contains("sh"));
```

### Usar `==` para comparar textos

Errado:

```java
if (banda == "Rush") {
    System.out.println("Encontrada");
}
```

Correto:

```java
if ("Rush".equals(banda)) {
    System.out.println("Encontrada");
}
```

### Esquecer a ordem da concatenação

Possível erro:

```java
int aprovadas = 10;
int pendentes = 5;

System.out.println("Total: " + aprovadas + pendentes);
```

Saída:

```text
Total: 105
```

Correto:

```java
System.out.println("Total: " + (aprovadas + pendentes));
```

### Usar especificador incompatível no `printf`

Errado:

```java
String ano = "1986";
System.out.printf("Ano: %d%n", ano);
```

Correto:

```java
int ano = 1986;
System.out.printf("Ano: %d%n", ano);
```

### Esquecer que `printf` não quebra linha sozinho

```java
System.out.printf("Banda: %s", "Rush");
System.out.printf("Gênero: %s", "Rock Progressivo");
```

Saída colada:

```text
Banda: RushGênero: Rock Progressivo
```

Melhor:

```java
System.out.printf("Banda: %s%n", "Rush");
System.out.printf("Gênero: %s%n", "Rock Progressivo");
```

### Misturar `nextInt()` e `nextLine()` sem consumir a linha

Problema comum:

```java
int ano = scanner.nextInt();
String banda = scanner.nextLine();
```

`banda` pode receber apenas o restante vazio da linha atual.

Solução simples:

```java
int ano = scanner.nextInt();
scanner.nextLine();
String banda = scanner.nextLine();
```

### Escapar barras em excesso ou de menos

Para representar:

```text
C:\catalogo\bandas
```

a string Java precisa ser:

```java
String caminho = "C:\\catalogo\\bandas";
```

### Usar `Scanner` como se fosse parser robusto de produção

`Scanner` é simples e útil, mas não é a melhor escolha para parsing de alto desempenho, protocolos de rede, JSON, CSV complexo ou payloads HTTP. Nesses casos, use bibliotecas e APIs específicas.

## Decisões de engenharia

### Quando usar concatenação com `+`

Use quando:

* a mensagem é curta;
* há poucos valores;
* a legibilidade fica melhor que com formato;
* não há loop grande montando texto repetidamente.

Exemplo:

```java
String mensagem = "Banda " + banda + " cadastrada";
```

Impacto:

* boa legibilidade em casos simples;
* performance adequada na maioria dos trechos pequenos;
* pode ficar confuso com muitos valores.

### Quando usar `String.formatted`

Use quando:

* a mensagem tem vários valores;
* você quer separar template de valores;
* a string final precisa ser guardada em variável;
* a leitura fica melhor que concatenação.

Exemplo:

```java
String mensagem = "Banda %s cadastrada com %d músicas"
        .formatted(banda, totalMusicas);
```

Impacto:

* melhora a organização;
* exige atenção aos especificadores;
* pode lançar exceções se o formato estiver incorreto.

### Quando usar `printf`

Use quando:

* a intenção é imprimir diretamente no console;
* você precisa formatar números, largura, precisão ou datas;
* está escrevendo uma ferramenta de terminal.

Exemplo:

```java
System.out.printf(Locale.US, "Avaliação média: %.2f%n", avaliacao);
```

Impacto:

* bom para saída textual formatada;
* não adiciona quebra de linha sozinho;
* não deve substituir logs profissionais em aplicações backend.

### Quando usar `StringBuilder`

Use quando:

* há concatenação dentro de loops;
* você monta textos grandes progressivamente;
* há muitas partes opcionais;
* a construção do texto é incremental.

Exemplo:

```java
StringBuilder builder = new StringBuilder();

for (String banda : bandas) {
    builder.append("- ").append(banda).append(System.lineSeparator());
}

String relatorio = builder.toString();
```

Impacto:

* evita criação excessiva de strings intermediárias;
* pode melhorar performance em loops;
* exige código um pouco mais verboso.

### Quando usar text blocks

Use quando:

* o texto é multilinha;
* você precisa representar JSON, SQL, HTML, XML ou mensagens longas;
* concatenação com `\n` deixaria o código ilegível.

Exemplo:

```java
String sql = """
        select b.id, b.nome
        from bandas b
        where b.genero = ?
        order by b.nome
        """;
```

Impacto:

* melhora muito a legibilidade;
* exige atenção à indentação;
* está disponível como recurso padrão desde Java 15.

### Quando evitar `Scanner`

Evite em:

* APIs web;
* parsing de arquivos grandes;
* leitura de JSON;
* leitura de CSV real com aspas, delimitadores e escapes;
* aplicações de alta performance.

Alternativas:

* `BufferedReader` para leitura linha a linha;
* bibliotecas JSON, como Jackson;
* bibliotecas CSV, como Apache Commons CSV;
* parâmetros HTTP em controllers Spring;
* DTOs de request em APIs REST.

### Quando evitar `printf`

Evite para logs de aplicação:

```java
System.out.printf("Erro ao cadastrar banda: %s%n", erro);
```

Em aplicações backend, prefira logging estruturado:

```java
log.info("Banda cadastrada com sucesso: {}", banda);
```

Motivo: frameworks de logging integram níveis de log, timestamps, correlação, appenders, observabilidade e coleta por ferramentas externas.

## Onde isso aparece em sistemas reais

### APIs REST

Ao retornar mensagens:

```java
String mensagem = "Banda %s cadastrada com sucesso".formatted(nomeBanda);
```

### Logs

Ao registrar eventos:

```java
log.info("Música cadastrada: banda={}, musica={}", banda, musica);
```

Mesmo quando você não usa `String` diretamente, os valores textuais continuam sendo parte central do log.

### JSON

Text blocks são úteis em testes:

```java
String payload = """
        {
          "banda": "Led Zeppelin",
          "musica": "Kashmir",
          "genero": "Hard Rock"
        }
        """;
```

### SQL

Text blocks melhoram consultas longas:

```java
String consulta = """
        select m.id, m.titulo, b.nome
        from musicas m
        join bandas b on b.id = m.banda_id
        where b.nome = ?
        order by m.titulo
        """;
```

### Mensagens para usuários

Formatação ajuda a montar respostas consistentes:

```java
String resposta = "A banda %s possui %d músicas cadastradas"
        .formatted(banda, quantidade);
```

### Ferramentas internas

`Scanner`, `printf` e `println` são úteis em CLIs simples:

```java
System.out.print("Digite o nome da banda: ");
String banda = scanner.nextLine();
System.out.printf("Buscando discografia de %s...%n", banda);
```

## Perguntas de revisão

1 - O que é uma `String` em Java?
Resposta: É uma classe usada para representar sequências de caracteres. Todo literal textual escrito entre aspas duplas, como `"Metallica"`, é uma instância de `String`.

2 - `String` é tipo primitivo?
Resposta: Não. `String` é uma classe do pacote `java.lang`. Ela é muito usada e tem suporte especial da linguagem, mas não é um tipo primitivo.

3 - Por que `String` começa com letra maiúscula?
Resposta: Porque é o nome de uma classe. Em Java, classes normalmente usam `UpperCamelCase`, enquanto tipos primitivos usam palavras reservadas minúsculas, como `int`, `double` e `boolean`.

4 - O que significa dizer que `String` é imutável?
Resposta: Significa que o conteúdo de uma `String` não muda depois que o objeto é criado. Operações como concatenação, `toUpperCase` ou `replace` retornam novas strings em vez de alterar a original.

5 - Qual é a diferença entre `==` e `equals` ao comparar strings?
Resposta: `==` compara se duas referências apontam para o mesmo objeto. `equals` compara o conteúdo textual. Para comparar textos, normalmente use `equals`.

6 - Por que `"Total: " + 10 + 5` resulta em `"Total: 105"`?
Resposta: Porque o operador `+` é avaliado da esquerda para a direita. Depois que a primeira parte vira string, os próximos valores são concatenados como texto.

7 - Como forçar a soma antes da concatenação?
Resposta: Use parênteses: `"Total: " + (10 + 5)`. Assim, o Java calcula `10 + 5` antes de concatenar com o texto.

8 - Para que servem sequências de escape?
Resposta: Servem para representar caracteres especiais dentro de literais, como aspas, barra invertida, quebra de linha, tabulação e caracteres de controle.

9 - O que `\"` representa?
Resposta: Representa uma aspas dupla dentro de uma string, sem encerrar o literal textual.

10 - O que `\\` representa?
Resposta: Representa uma única barra invertida no valor final da string.

11 - O que `\n` representa?
Resposta: Representa uma quebra de linha do tipo line feed. Em strings de formatação, `%n` costuma ser preferível por usar o separador de linha da plataforma.

12 - O que é um text block?
Resposta: É uma forma de escrever strings multilinha usando `"""`, disponível como recurso padrão desde Java 15. É útil para JSON, SQL, HTML e textos longos.

13 - Qual é a diferença entre `print` e `println`?
Resposta: `print` escreve o valor sem encerrar a linha. `println` escreve o valor e depois encerra a linha.

14 - Qual é a diferença entre `println` e `printf`?
Resposta: `println` imprime valores simples e quebra linha. `printf` usa uma string de formatação com especificadores como `%s`, `%d`, `%f` e não quebra linha automaticamente.

15 - O que `%s` faz no `printf`?
Resposta: Formata um argumento como string, normalmente usando `toString()` quando o argumento não é uma `String`.

16 - O que `%d` faz?
Resposta: Formata um número inteiro decimal, como `int`, `long`, `short`, `byte` ou `BigInteger`.

17 - O que `%f` faz?
Resposta: Formata número de ponto flutuante, como `float`, `double` ou `BigDecimal`.

18 - O que significa `%.2f`?
Resposta: Significa formatar um número de ponto flutuante com duas casas decimais.

19 - O que significa `%10.2f`?
Resposta: Significa formatar um número com largura mínima de 10 caracteres e duas casas decimais. Se o valor ocupar menos espaço, o Java preenche com espaços à esquerda.

20 - O que `%n` faz?
Resposta: Insere o separador de linha da plataforma. É mais portável que usar `\n` dentro de strings de formatação.

21 - Como imprimir o caractere `%` usando `printf`?
Resposta: Use `%%`. Exemplo: `System.out.printf("Progresso: 80%%%n");`.

22 - O que é `String.formatted`?
Resposta: É um método disponível desde Java 15 que formata a própria string usando os argumentos informados, funcionando de forma semelhante a `String.format`.

23 - O que é `Scanner`?
Resposta: É uma classe usada para ler entrada textual e converter tokens em tipos como `String`, `int`, `double`, `boolean`, `BigDecimal` e outros.

24 - Qual é a diferença entre `next()` e `nextLine()`?
Resposta: `next()` lê o próximo token, normalmente até um espaço em branco. `nextLine()` lê o restante da linha atual.

25 - Por que misturar `nextInt()` e `nextLine()` pode causar problema?
Resposta: Porque `nextInt()` consome apenas o número, mas pode deixar a quebra de linha pendente. O próximo `nextLine()` pode ler essa sobra em vez de esperar uma nova entrada.

26 - Como resolver o problema entre `nextInt()` e `nextLine()`?
Resposta: Depois de `nextInt()`, chame um `nextLine()` extra para consumir o restante da linha antes de ler o próximo texto completo.

27 - Quais outros métodos de leitura existem no `Scanner`?
Resposta: Existem métodos como `nextLong`, `nextFloat`, `nextDouble`, `nextBoolean`, `nextByte`, `nextShort`, `nextBigInteger`, `nextBigDecimal` e suas versões `hasNext...`.

28 - Quando usar `StringBuilder`?
Resposta: Use quando precisar montar strings progressivamente, especialmente dentro de loops ou com muitas partes opcionais.

29 - O que mudou em `String` no Java 9?
Resposta: O Java 9 trouxe Compact Strings pela JEP 254, mudando a representação interna para economizar memória em muitos casos, e a JEP 280, que mudou a estratégia de concatenação para usar `invokedynamic`.

30 - String Templates existem como recurso padrão no Java 25?
Resposta: Não. String Templates foram preview no Java 21 e Java 22, mas foram retirados antes do Java 23. Em Java 25, use concatenação, `StringBuilder`, `String.format`, `String.formatted`, `MessageFormat` ou templates externos conforme o caso.

## Resumo final

`String` é uma das classes mais importantes de Java. Ela representa texto, é imutável, tem tratamento especial pela linguagem e aparece em praticamente qualquer aplicação backend.

Para usar bem `String`, é importante entender:

* literais de string são objetos `String`;
* `String` não é tipo primitivo;
* concatenação com `+` depende da ordem de avaliação;
* use parênteses quando quiser calcular antes de concatenar;
* sequências de escape representam caracteres especiais;
* `\\` representa uma barra invertida real;
* `\n` representa quebra de linha, mas `%n` é melhor em `printf`;
* `print`, `println`, `printf` e `format` têm propósitos diferentes;
* `printf` usa especificadores como `%s`, `%d`, `%f`, `%%` e `%n`;
* `Scanner` lê tokens e linhas, mas exige cuidado ao misturar métodos;
* `StringBuilder` é melhor para montagem repetitiva de texto;
* text blocks, desde Java 15, melhoram strings multilinha;
* Java 9 trouxe mudanças internas importantes com Compact Strings e concatenação via `invokedynamic`.

Em sistemas reais, strings aparecem em logs, APIs, JSON, SQL, mensagens, validações, integrações e ferramentas internas. Dominar `String` é dominar uma parte essencial da comunicação entre o programa, o usuário, outros sistemas e a própria JVM.

## Referências

* Java Language Specification, Java SE 25 Edition — literais, text blocks, escapes, ordem de avaliação e concatenação. ([Oracle Docs][11])
* Java SE 25 API Documentation — `java.lang.String`. ([Oracle Docs][7])
* Java SE 25 API Documentation — `java.io.PrintStream`. ([Oracle Docs][4])
* Java SE 25 API Documentation — `java.util.Formatter`. ([Oracle Docs][5])
* Java SE 25 API Documentation — `java.util.Scanner`. ([Oracle Docs][6])
* OpenJDK JEP 254 — Compact Strings. ([OpenJDK][8])
* OpenJDK JEP 280 — Indify String Concatenation. ([OpenJDK][9])
* OpenJDK JEP 378 — Text Blocks. ([OpenJDK][3])
* Oracle Java Language Changes by Release — histórico de String Templates e mudanças da linguagem. ([Oracle Docs][10])
* Effective Java, Joshua Bloch — boas práticas gerais de uso de objetos, imutabilidade, legibilidade e APIs.


[1]: https://docs.oracle.com/javase/specs/jls/se25/html/jls-3.html "Chapter 3. Lexical Structure"
[2]: https://docs.oracle.com/javase/specs/jls/se25/html/jls-15.html "Chapter 15. Expressions"
[3]: https://openjdk.org/jeps/378 "JEP 378: Text Blocks"
[4]: https://docs.oracle.com/en/java/javase/25/docs/api//java.base/java/io/PrintStream.html "PrintStream (Java SE 25 & JDK 25)"
[5]: https://docs.oracle.com/en/java/javase/25/docs//api/java.base/java/util/Formatter.html "Formatter (Java SE 25 & JDK 25)"
[6]: https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/util/Scanner.html "Scanner (Java SE 25 & JDK 25)"
[7]: https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/String.html "String (Java SE 25 & JDK 25)"
[8]: https://openjdk.org/jeps/254 "JEP 254: Compact Strings"
[9]: https://openjdk.org/jeps/280 "JEP 280: Indify String Concatenation"
[10]: https://docs.oracle.com/en/java/javase/25/language/java-language-changes-release.html "Java Language Changes by Release"
[11]: https://docs.oracle.com/javase/specs/jls/se25/html/index.html "The Java® Language Specification"
