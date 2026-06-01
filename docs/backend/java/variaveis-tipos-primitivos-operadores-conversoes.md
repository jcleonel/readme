# Fundamentos de Java: variáveis, tipos primitivos, operadores e conversões

Todo sistema backend manipula dados: identificadores, quantidades, preços, datas, flags, códigos, médias, limites e resultados de cálculos. Em uma aplicação como o **meucatalogomusical.com**, por exemplo, o código precisa representar a duração de uma música, a quantidade de reproduções de uma banda, o preço de um álbum digital, a posição de uma faixa no catálogo e se um item está ativo ou não.

O problema é que cada uma dessas informações tem uma natureza diferente. Uma duração em segundos não é a mesma coisa que um preço. Uma flag de ativação não é a mesma coisa que uma nota média. Um código de caractere não é a mesma coisa que um texto completo.

Os fundamentos de Java existem para dar precisão a essas decisões. Variáveis, tipos primitivos, operadores aritméticos, conversões e promoção aritmética formam a base para escrever código que calcula corretamente, evita perda de informação e deixa claro o que cada dado representa.

Em sistemas reais, erros nesses fundamentos aparecem como bugs silenciosos: divisão inteira retornando `1` em vez de `1.5`, valores monetários com arredondamento indevido, `int` estourando em contadores grandes, `byte` e `short` sendo promovidos para `int` sem o desenvolvedor perceber, ou conversões explícitas descartando dados importantes.

Os exemplos deste artigo usam **Java 25**, versão que alcançou disponibilidade geral em 16 de setembro de 2025 e é tratada como LTS por grande parte dos fornecedores. ([OpenJDK][1])

## Antes de entrar no código: uma ideia simples

Uma variável pode ser vista como um nome dado a uma informação que o programa precisa usar.

Por exemplo:

```java
int duracaoEmSegundos = 515;
```

Nesse caso:

* `int` define o tipo da informação;
* `duracaoEmSegundos` é o nome da variável;
* `515` é o valor armazenado;
* `=` atribui o valor à variável.

A ideia mais importante é: **em Java, o tipo faz parte do contrato da variável**.

Depois que uma variável local é declarada como `int`, ela continua sendo `int`. O valor pode mudar, mas o tipo não muda:

```java
int totalFaixas = 8;

totalFaixas = 9; // ok: continua sendo int
// totalFaixas = 9.5; // não compila: 9.5 não é int
```

Isso ajuda o compilador a encontrar erros antes da aplicação rodar. Em um backend, esse tipo de proteção é essencial porque muitos erros de cálculo só apareceriam em produção se a linguagem não validasse tipos em tempo de compilação.

## Definição técnica

A especificação da linguagem Java divide os tipos em duas grandes categorias: **tipos primitivos** e **tipos de referência**. Os tipos primitivos incluem `boolean` e os tipos numéricos. Os numéricos se dividem em integrais (`byte`, `short`, `int`, `long` e `char`) e ponto flutuante (`float` e `double`). Já os tipos de referência incluem classes, interfaces e arrays. ([Oracle Docs][2])

Os tipos primitivos representam valores diretamente. Eles não possuem identidade de objeto e não compartilham estado com outros valores primitivos. A própria especificação define que valores primitivos não compartilham estado entre si. ([Oracle Docs][2])

| Tipo      |         Categoria |                                                Tamanho / faixa principal | Uso típico em backend                                |
| --------- | ----------------: | -----------------------------------------------------------------------: | ---------------------------------------------------- |
| `boolean` |            lógico | valores `true` ou `false`; a linguagem não fixa tamanho de armazenamento | flags, validações, estados simples                   |
| `byte`    |           inteiro |                                                8 bits, de `-128` a `127` | dados binários, protocolos, buffers                  |
| `short`   |           inteiro |                                           16 bits, de `-32768` a `32767` | integração legada, formatos compactos                |
| `int`     |           inteiro |                                 32 bits, de `-2147483648` a `2147483647` | contadores, quantidades, cálculos inteiros comuns    |
| `long`    |           inteiro |               64 bits, de `-9223372036854775808` a `9223372036854775807` | IDs, timestamps, contadores grandes                  |
| `char`    | inteiro sem sinal |       16 bits, de `0` a `65535`; representa uma unidade de código UTF-16 | caracteres simples e códigos textuais baixos         |
| `float`   |   ponto flutuante |                                               32 bits, IEEE 754 binary32 | medições aproximadas, uso raro em backend comum      |
| `double`  |   ponto flutuante |                                               64 bits, IEEE 754 binary64 | cálculos aproximados, médias, métricas, estatísticas |

As faixas numéricas dos tipos integrais e a natureza dos tipos `float` e `double` seguem a Java Language Specification. Ela também define que `char` é um inteiro sem sinal de 16 bits que representa uma unidade de código UTF-16, não necessariamente um “caractere visual completo” como o usuário enxerga na tela. ([Oracle Docs][2])

Essa observação sobre `char` é importante: alguns símbolos modernos, como emojis ou certos caracteres fora do Plano Multilíngue Básico Unicode, podem precisar de mais de uma unidade UTF-16. Para textos reais de usuário, prefira `String`.

## Como funciona em Java

### Declaração e atribuição de variáveis

Em Java, uma variável local precisa ser declarada antes de ser usada:

```java
int totalAlbuns = 12;
double notaMedia = 4.7;
boolean bandaAtiva = true;
char classificacao = 'A';
```

O valor pode ser alterado depois:

```java
int totalAlbuns = 12;
totalAlbuns = 13;
```

Mas o tipo permanece o mesmo:

```java
int totalAlbuns = 12;

// totalAlbuns = 13.5; // não compila
```

O compilador usa o tipo declarado para validar as operações possíveis. Isso é parte do motivo pelo qual Java é uma linguagem estaticamente tipada.

### Nomes de variáveis

Em código Java profissional, nomes de variáveis locais normalmente seguem `lowerCamelCase`:

```java
int duracaoTotalEmSegundos = 0;
boolean albumDisponivel = true;
long totalReproducoes = 2_500_000L;
```

Boas variáveis descrevem intenção, não apenas formato. Por exemplo, `duracaoTotalEmSegundos` comunica mais do que `x`, e `precoAlbum` comunica mais do que `valor`.

### Literais numéricos com underscore

Desde Java 7, é possível usar `_` entre dígitos em literais numéricos para melhorar a leitura. Esse recurso veio pelo Project Coin / JSR 334, não por uma JEP individual. ([OpenJDK CR][3])

```java
int populacao = 699_097;
long totalReproducoes = 2_147_483_648L;
```

O `_` não muda o valor. Ele só melhora a legibilidade no código-fonte.

### Sufixos `L`, `f` e `d`

Por padrão, literais inteiros comuns são tratados como `int`, e literais de ponto flutuante comuns são tratados como `double`. A especificação define que literais inteiros sem sufixo especial são `int`, e literais de ponto flutuante sem `F`/`f` são `double`. ([Oracle Docs][4])

Por isso:

```java
long totalReproducoes = 2_147_483_648L;
float taxaPopularidade = 98.5f;
double notaMedia = 4.85d; // o d é opcional
```

O `L` indica que o literal é `long`. O `f` indica que o literal é `float`. O `d` indica `double`, mas quase sempre é omitido porque `double` já é o padrão.

## Exemplo prático

Imagine uma rotina simples do **meucatalogomusical.com** para calcular informações de um álbum no catálogo.

```java
import java.math.BigDecimal;
import java.math.RoundingMode;

public class CalculoCatalogoMusical {

    public static void main(String[] args) {
        String banda = "Rush";
        String album = "Moving Pictures";

        int duracaoTomSawyer = 276;
        int duracaoLimelight = 259;
        int duracaoYyz = 266;

        int totalFaixas = 3;
        int duracaoTotal = duracaoTomSawyer + duracaoLimelight + duracaoYyz;
        double duracaoMedia = duracaoTotal / (double) totalFaixas;

        long totalReproducoes = 2_147_483_648L;
        totalReproducoes += 50_000L;

        byte prioridadeCuradoria = 5;
        prioridadeCuradoria++;

        byte prioridadeAjustada = (byte) (prioridadeCuradoria + 1);

        char seloCuradoria = 'A';
        int codigoSeloCuradoria = seloCuradoria;

        boolean albumClassico = duracaoTotal > 700;

        BigDecimal precoAlbum = new BigDecimal("39.90");
        BigDecimal desconto = new BigDecimal("5.00");
        BigDecimal precoPromocional = precoAlbum.subtract(desconto);

        BigDecimal precoPorFaixa = precoPromocional.divide(
                BigDecimal.valueOf(totalFaixas),
                2,
                RoundingMode.HALF_UP
        );

        System.out.println("Banda: " + banda);
        System.out.println("Álbum: " + album);
        System.out.println("Duração total: " + duracaoTotal + " segundos");
        System.out.println("Duração média: " + duracaoMedia + " segundos");
        System.out.println("Total de reproduções: " + totalReproducoes);
        System.out.println("Prioridade ajustada: " + prioridadeAjustada);
        System.out.println("Código do selo: " + codigoSeloCuradoria);
        System.out.println("Álbum clássico? " + albumClassico);
        System.out.println("Preço promocional: R$ " + precoPromocional);
        System.out.println("Preço por faixa: R$ " + precoPorFaixa);
    }
}
```

## Analisando o código

`String banda` e `String album` são tipos de referência, não tipos primitivos. Eles aparecem no exemplo porque nomes de bandas e álbuns são textos, e textos em Java são representados pela classe `String`.

As variáveis `duracaoTomSawyer`, `duracaoLimelight` e `duracaoYyz` foram declaradas como `int` porque durações em segundos cabem confortavelmente em um inteiro de 32 bits. Para esse tipo de cálculo, `int` é legível, eficiente e suficiente.

```java
int duracaoTotal = duracaoTomSawyer + duracaoLimelight + duracaoYyz;
```

Aqui ocorre uma soma entre inteiros. O resultado também é `int`.

```java
double duracaoMedia = duracaoTotal / (double) totalFaixas;
```

Esse trecho é importante. Se fosse escrito assim:

```java
double duracaoMedia = duracaoTotal / totalFaixas;
```

a divisão seria feita como divisão inteira, porque os dois operandos são `int`. Só depois o resultado inteiro seria atribuído ao `double`. Ao converter um dos operandos para `double`, a operação passa a ser uma divisão de ponto flutuante.

```java
long totalReproducoes = 2_147_483_648L;
```

Esse valor ultrapassa o limite máximo de `int`, por isso precisa ser `long`. O sufixo `L` deixa claro que o literal é do tipo `long`.

```java
totalReproducoes += 50_000L;
```

Esse é um operador de atribuição composto. Ele equivale, conceitualmente, a:

```java
totalReproducoes = totalReproducoes + 50_000L;
```

Mas existe um detalhe técnico importante: operadores como `+=`, `-=`, `*=`, `/=` e `%=` fazem uma conversão implícita para o tipo da variável da esquerda. A JLS define que `E1 op= E2` equivale a `E1 = (T) ((E1) op (E2))`, onde `T` é o tipo de `E1`, com a diferença de que `E1` é avaliado apenas uma vez. ([Oracle Docs][4])

Isso pode ser útil, mas também perigoso:

```java
short posicao = 3;

posicao += 1; // compila
// posicao = posicao + 1; // não compila, porque posicao + 1 vira int
```

A segunda forma não compila porque operações aritméticas com `byte`, `short` e `char` geralmente são promovidas para `int`.

```java
byte prioridadeCuradoria = 5;
prioridadeCuradoria++;
```

O operador `++` incrementa a variável em uma unidade. Para pós-incremento, o valor da expressão é o valor anterior ao armazenamento do novo valor; para pré-incremento, o valor da expressão é o valor depois do armazenamento. A mesma lógica vale para `--`. ([Oracle Docs][4])

```java
byte prioridadeAjustada = (byte) (prioridadeCuradoria + 1);
```

Esse trecho mostra uma pegadinha real: `prioridadeCuradoria + 1` não resulta em `byte`, mas em `int`. Por isso o cast explícito para `byte` é necessário.

```java
char seloCuradoria = 'A';
int codigoSeloCuradoria = seloCuradoria;
```

`char` pode ser convertido implicitamente para `int`, porque é uma conversão de ampliação (`widening`). O valor de `'A'` corresponde ao código numérico dessa unidade UTF-16.

```java
BigDecimal precoAlbum = new BigDecimal("39.90");
```

`BigDecimal` não é tipo primitivo. Ele é uma classe da API Java usada quando precisamos de controle decimal e arredondamento explícito, especialmente em valores monetários. A documentação oficial define `BigDecimal` como uma classe para aritmética decimal, manipulação de escala, arredondamento, comparação e conversão de formato. ([Oracle Docs][5])

## O que acontece por baixo dos panos

### Tipagem em tempo de compilação

Quando o compilador encontra:

```java
int totalFaixas = 3;
```

ele registra que `totalFaixas` é uma variável do tipo `int`. A partir desse ponto, operações envolvendo essa variável passam por validação estática.

Por isso este código não compila:

```java
int totalFaixas = 3;
// totalFaixas = 3.5;
```

O compilador sabe que `3.5` é um literal `double`, e uma conversão de `double` para `int` pode perder informação. Java exige cast explícito nesse caso:

```java
double valor = 3.5;
int inteiro = (int) valor; // resultado: 3
```

A conversão descarta a parte decimal. Isso não é arredondamento; é truncamento em direção a zero.

### Conversões de ampliação

Conversões de ampliação são aquelas em que o destino consegue representar uma faixa maior de valores ou uma categoria mais ampla. Exemplos:

```java
byte codigoPequeno = 10;
short codigoMedio = codigoPequeno;
int codigoInteiro = codigoMedio;
long codigoLongo = codigoInteiro;

char letra = 'A';
int codigoLetra = letra;

float notaFloat = 4.5f;
double notaDouble = notaFloat;
```

A JLS lista como conversões de ampliação, entre outras, `byte` para `short`, `int`, `long`, `float` ou `double`; `short` para `int`, `long`, `float` ou `double`; `char` para `int`, `long`, `float` ou `double`; `int` para `long`, `float` ou `double`; `long` para `float` ou `double`; e `float` para `double`. ([Oracle Docs][6])

Um detalhe de nível pleno/sênior: ampliação nem sempre significa preservação exata de precisão. Por exemplo, `long` para `double` ou `int` para `float` podem perder precisão, mesmo sendo conversões permitidas implicitamente. A especificação permite isso e afirma que conversões de ampliação nunca geram exceção em tempo de execução. ([Oracle Docs][6])

### Conversões de estreitamento

Conversões de estreitamento exigem cast explícito porque podem perder informação:

```java
long reproducoes = 9_300_000_035L;
int reproducoesComoInt = (int) reproducoes;

System.out.println(reproducoesComoInt); // valor inesperado
```

Isso acontece porque um `long` de 64 bits não cabe necessariamente em um `int` de 32 bits.

O mesmo vale para `double` para `int`:

```java
double nota = 4.95;
int notaInteira = (int) nota;

System.out.println(notaInteira); // 4
```

A JLS define que conversões de estreitamento podem perder magnitude, precisão e faixa. Para inteiros, bits mais significativos podem ser descartados; para ponto flutuante convertido para inteiro, a parte fracionária é removida por arredondamento em direção a zero. ([Oracle Docs][6])

### Conversões com `byte`, `short` e `char`

Esses tipos causam muitas dúvidas porque, embora sejam inteiros menores, as operações aritméticas comuns promovem seus valores para `int`.

```java
byte a = 10;
byte b = 20;

// byte resultado = a + b; // não compila
int resultado = a + b;     // correto
```

O mesmo acontece com `short`:

```java
short faixasLadoA = 4;
short faixasLadoB = 5;

// short total = faixasLadoA + faixasLadoB; // não compila
int total = faixasLadoA + faixasLadoB;
```

E com `char`:

```java
char selo = 'A';

int proximoCodigo = selo + 1;
char proximoSelo = (char) (selo + 1);
```

A operação `selo + 1` resulta em `int`, não em `char`.

Também existe uma conversão especial de `byte` para `char`, que combina ampliação e estreitamento: primeiro o `byte` é convertido para `int`, depois o `int` é convertido para `char`. Isso precisa ser tratado com cuidado porque `byte` é assinado e `char` é sem sinal. ([Oracle Docs][6])

```java
byte codigo = 65;

char caractere = (char) codigo;
System.out.println(caractere); // A
```

Agora veja um caso perigoso:

```java
byte codigoNegativo = -1;
char caractere = (char) codigoNegativo;

System.out.println((int) caractere); // 65535
```

Isso acontece porque `char` não representa números negativos. O cast força a conversão, mas o significado do valor muda.

### Promoção aritmética

Promoção aritmética é a regra que Java usa para definir o tipo final de uma operação numérica.

Em operações binárias, Java promove os operandos para um tipo comum. A regra geral em contexto aritmético é:

1. se houver `double`, o resultado tende a `double`;
2. senão, se houver `float`, o resultado tende a `float`;
3. senão, se houver `long`, o resultado tende a `long`;
4. caso contrário, o resultado tende a `int`.

A especificação define essa ordem de promoção numérica: `double`, depois `float`, depois `long`; quando nenhum deles aparece em contexto aritmético, o tipo promovido é `int`. ([Oracle Docs][6])

Exemplo:

```java
int x = 10;
long y = 5;

long resultado = x * y;
```

O `int` é promovido para `long`, então o resultado da multiplicação é `long`.

Outro exemplo:

```java
long reproducoes = 10_000L;
float fatorPopularidade = 1.5f;

float resultado = reproducoes * fatorPopularidade;
```

Mesmo `long` tendo 64 bits, `float` é ponto flutuante. A operação é promovida para `float`.

A divisão inteira é um dos pontos mais importantes:

```java
int totalSegundos = 3;
int totalFaixas = 2;

double mediaErrada = totalSegundos / totalFaixas;
double mediaCorreta = totalSegundos / (double) totalFaixas;

System.out.println(mediaErrada);   // 1.0
System.out.println(mediaCorreta);  // 1.5
```

Na primeira conta, a divisão acontece entre dois `int`, então o resultado é `int`. Só depois ele é convertido para `double`.

Na segunda conta, um dos operandos foi convertido para `double`, então a divisão já nasce como operação de ponto flutuante.

### Operadores aritméticos

Java possui os operadores aritméticos básicos:

```java
int soma = 10 + 5;
int subtracao = 10 - 5;
int multiplicacao = 10 * 5;
int divisaoInteira = 10 / 3;
int resto = 10 % 3;
```

Os operadores `*`, `/` e `%` têm precedência maior que `+` e `-`. A JLS define `*`, `/` e `%` como operadores multiplicativos, e `+` e `-` como operadores aditivos; dentro de cada grupo, a avaliação é associativa à esquerda. ([Oracle Docs][4])

Por isso:

```java
int mediaErrada = 40 + 20 + 30 / 3;
int mediaCorreta = (40 + 20 + 30) / 3;
```

A primeira expressão calcula primeiro `30 / 3`. A segunda força a soma antes da divisão.

O operador `+` também pode ser usado para concatenação de `String`:

```java
int totalFaixas = 8;

System.out.println("Total de faixas: " + totalFaixas);
```

Quando um dos operandos do `+` é `String`, a operação vira concatenação, não soma numérica. A JLS define que, se qualquer operando do `+` for `String`, a operação é concatenação de strings. ([Oracle Docs][4])

### Introdução a `BigDecimal` para valores monetários

`float` e `double` são ótimos para cálculos aproximados, mas não são a melhor escolha para dinheiro. Eles seguem representação binária de ponto flutuante, e muitos valores decimais comuns não são representados exatamente em binário.

Exemplo clássico:

```java
double preco = 0.10 + 0.20;

System.out.println(preco); // pode imprimir algo como 0.30000000000000004
```

Para valores monetários, prefira `BigDecimal`:

```java
BigDecimal preco = new BigDecimal("0.10");
BigDecimal taxa = new BigDecimal("0.20");

BigDecimal total = preco.add(taxa);

System.out.println(total); // 0.30
```

Evite:

```java
BigDecimal valor = new BigDecimal(0.10); // não recomendado
```

Prefira:

```java
BigDecimal valor = new BigDecimal("0.10");
```

ou:

```java
BigDecimal valor = BigDecimal.valueOf(0.10);
```

Em sistemas reais, além de usar `BigDecimal`, você precisa definir regras de escala e arredondamento:

```java
BigDecimal total = new BigDecimal("34.90");
BigDecimal quantidade = BigDecimal.valueOf(3);

BigDecimal precoUnitario = total.divide(
        quantidade,
        2,
        RoundingMode.HALF_UP
);

System.out.println(precoUnitario); // 11.63
```

`BigDecimal` dá controle explícito sobre arredondamento. A documentação oficial afirma que, se nenhum modo de arredondamento for especificado e o resultado exato não puder ser representado, uma `ArithmeticException` pode ser lançada. ([Oracle Docs][5])

## Comparação com versões anteriores do Java

Os fundamentos de variáveis, tipos primitivos, operadores aritméticos, casting e promoção aritmética são muito estáveis em Java. Um `int`, um `long`, uma divisão inteira e uma promoção de `byte` para `int` continuam seguindo as mesmas ideias centrais em Java 25.

O que mudou ao longo das versões foi principalmente a ergonomia da linguagem.

Em Java 7, o Project Coin / JSR 334 adicionou melhorias como literais binários e underscores em literais numéricos. Antes disso, números longos precisavam ser escritos sem separadores visuais:

```java
long valorAntigo = 2147483648L;
```

Com Java 7 ou superior:

```java
long valorAtual = 2_147_483_648L;
```

Em Java 10, a JEP 286 introduziu inferência de tipo para variáveis locais com `var`. Isso permite omitir o tipo explícito em algumas declarações locais com inicializador, mantendo a segurança estática de tipos. ([OpenJDK][7])

```java
var totalFaixas = 8;      // inferido como int
var banda = "Metallica";  // inferido como String
```

Mas `var` não transforma Java em linguagem dinâmica:

```java
var totalFaixas = 8;

// totalFaixas = "oito"; // não compila
```

O tipo continua fixo; ele apenas foi inferido pelo compilador.

Em Java 25, a JEP 512 finalizou arquivos de origem compactos e métodos `main` de instância, permitindo escrever pequenos programas com menos cerimônia. Isso ajuda estudos, scripts e exemplos pequenos, mas não altera as regras fundamentais de tipos, operadores ou conversões. ([OpenJDK][8])

Exemplo compacto em Java 25:

```java
void main() {
    int totalFaixas = 8;
    IO.println("Total de faixas: " + totalFaixas);
}
```

Para artigos técnicos e código de backend profissional, ainda é comum usar classes explícitas, pacotes, testes e organização tradicional.

## Boas práticas

Prefira `int` para contadores e quantidades comuns. Use `long` quando o valor puder ultrapassar o limite de `int`, como IDs, timestamps e contadores acumulados de alto volume.

Evite `byte` e `short` para “economizar memória” em variáveis locais comuns. Na prática, eles frequentemente são promovidos para `int` em operações, e podem deixar o código mais confuso. Use-os quando houver motivo real: protocolo binário, arquivo, buffer, interoperabilidade ou estrutura compacta.

Use `boolean` para estados claramente binários:

```java
boolean albumAtivo = true;
boolean assinaturaExpirada = false;
```

Evite representar booleanos com `0` e `1`. Em Java, isso não é idiomático e nem compila diretamente para `boolean`.

Use `char` apenas para caracteres simples ou códigos controlados. Para nomes, títulos, descrições, slugs, mensagens e conteúdo de usuário, use `String`.

Use `double` para cálculos aproximados, como estatísticas, médias, notas e métricas. Para dinheiro, use `BigDecimal`.

Use parênteses quando a expressão puder gerar dúvida:

```java
int media = (idadeA + idadeB + idadeC) / 3;
```

Mesmo quando a precedência da linguagem resolveria corretamente, parênteses podem melhorar a leitura.

Evite misturar cálculo e concatenação complexa na mesma linha:

```java
System.out.println("Média: " + (total / (double) quantidade));
```

Prefira nomes que indiquem unidade:

```java
int duracaoEmSegundos = 515;
long tamanhoEmBytes = 5_242_880L;
BigDecimal precoEmReais = new BigDecimal("39.90");
```

Unidades explícitas evitam bugs muito comuns em sistemas distribuídos.

## Erros comuns

### Achar que divisão inteira arredonda

```java
int x = 3;
int y = 2;

double resultado = x / y;

System.out.println(resultado); // 1.0
```

O resultado não é `1.5` porque a divisão aconteceu entre inteiros.

Correção:

```java
double resultado = x / (double) y;
```

### Usar `double` para dinheiro

```java
double total = 0.10 + 0.20;
```

Para dinheiro:

```java
BigDecimal total = new BigDecimal("0.10")
        .add(new BigDecimal("0.20"));
```

### Esquecer o `L` em literais `long`

```java
// long valor = 2_147_483_648; // não compila
long valor = 2_147_483_648L;
```

O literal sem `L` é interpretado como `int`, e esse valor não cabe em `int`.

### Achar que `byte + byte` resulta em `byte`

```java
byte a = 10;
byte b = 20;

// byte c = a + b; // não compila
int c = a + b;
```

Operações com `byte`, `short` e `char` geralmente são promovidas para `int`.

### Fazer cast para “calar o compilador”

```java
long reproducoes = 9_300_000_035L;
int valor = (int) reproducoes;
```

Esse código compila, mas o valor pode ficar incorreto. Cast explícito deve ser uma decisão consciente, não uma forma de ignorar o compilador.

### Confundir `char` com texto

```java
char inicial = 'R';
// char banda = 'Rush'; // não compila
String banda = "Rush";
```

`char` representa uma unidade UTF-16. Texto deve ser `String`.

### Abusar de incremento dentro de expressões

```java
int total = 10;
int resultado = total++ + ++total;
```

Mesmo sendo válido, esse tipo de expressão reduz legibilidade. Em código profissional, prefira operações separadas.

## Decisões de engenharia

### Quando usar tipos primitivos

Use tipos primitivos quando você precisa representar valores simples, sem identidade de objeto, com baixo custo e sem necessidade de `null`.

Exemplos:

```java
int quantidadeFaixas;
long totalReproducoes;
boolean albumAtivo;
double notaMedia;
```

Eles são diretos, eficientes e expressam bem dados básicos.

### Quando evitar tipos primitivos

Evite primitivos quando `null` tiver significado no modelo ou quando você estiver trabalhando com APIs que exigem objetos, como algumas estruturas genéricas:

```java
Integer quantidadeOpcional;
Long idExterno;
Boolean aceiteTermos;
```

Mas cuidado: wrappers (`Integer`, `Long`, `Boolean`) podem ser `null`, e isso introduz risco de `NullPointerException` por unboxing automático.

### Quando usar `int`

Use `int` para quantidades e cálculos inteiros comuns:

```java
int totalFaixas = 10;
int duracaoEmSegundos = 515;
```

Evite `int` quando o valor pode crescer muito, como total global de reproduções, IDs externos ou timestamps.

### Quando usar `long`

Use `long` para valores inteiros grandes:

```java
long totalReproducoes = 9_000_000_000L;
long timestampEvento = System.currentTimeMillis();
```

O impacto de performance costuma ser irrelevante para a maioria dos backends. A decisão deve ser guiada por faixa de valor e clareza.

### Quando usar `float` e `double`

Use `double` para valores aproximados:

```java
double notaMedia = 4.75;
double percentualPopularidade = 98.3;
```

Evite `float` em backend comum, a menos que exista motivo específico, como economia de memória em grande volume de dados numéricos aproximados ou compatibilidade com APIs específicas.

### Quando usar `BigDecimal`

Use `BigDecimal` para dinheiro, taxas financeiras e cálculos decimais nos quais arredondamento precisa ser controlado:

```java
BigDecimal preco = new BigDecimal("39.90");
```

O custo é maior que `double`, mas a previsibilidade compensa em domínios financeiros. Em backend, normalmente é melhor pagar esse custo do que introduzir erros monetários.

### Quando evitar casts

Evite casts quando eles estiverem apenas escondendo uma modelagem errada:

```java
int total = (int) valorLongo;
```

Antes de fazer isso, pergunte:

* o valor realmente cabe no tipo menor?
* o truncamento é aceitável?
* existe teste cobrindo o limite?
* o tipo da variável de destino deveria ser maior?

### Alternativas

Para valores monetários persistidos em banco, uma alternativa comum é armazenar em menor unidade monetária usando inteiro, por exemplo centavos em `long`:

```java
long precoEmCentavos = 3990L;
```

Essa estratégia pode ser boa em domínios financeiros controlados, mas exige disciplina: todas as operações precisam respeitar a unidade. Para regras de negócio com escala decimal, impostos, descontos e arredondamento, `BigDecimal` costuma ser mais expressivo.

## Onde isso aparece em sistemas reais

Em APIs REST, tipos aparecem em DTOs, validações e contratos JSON. Um campo `quantidade` pode ser `int`, mas um campo `id` frequentemente é `long` ou `UUID`.

Em persistência, escolhas de tipos precisam conversar com o banco. Um `BigDecimal` em Java normalmente se relaciona melhor com colunas `NUMERIC`/`DECIMAL` do que com `FLOAT`/`DOUBLE`.

Em mensageria, tipos pequenos aparecem em protocolos, flags, códigos e serialização. Nesses casos, `byte` pode fazer sentido.

Em observabilidade, métricas e percentuais podem usar `double`, desde que sejam aproximados e não monetários.

Em regras de negócio, a decisão de tipo comunica intenção. Um método que recebe `BigDecimal preco` transmite uma ideia diferente de um método que recebe `double preco`.

```java
public BigDecimal calcularPrecoPromocional(
        BigDecimal precoOriginal,
        BigDecimal desconto
) {
    return precoOriginal.subtract(desconto);
}
```

Esse método deixa claro que a operação envolve valor decimal controlado, não aproximação binária.

## Perguntas de revisão

### 1. Qual é a diferença entre declarar uma variável e atribuir um valor a ela?

**Resposta:** Declarar uma variável é informar ao compilador que aquele nome existe e qual tipo de valor ele pode armazenar. Atribuir um valor é colocar efetivamente uma informação dentro dessa variável.

```java
int totalFaixas;      // declaração
totalFaixas = 8;      // atribuição
```

Também é possível declarar e atribuir na mesma linha:

```java
int totalFaixas = 8;
```

---

### 2. Por que Java não permite trocar o tipo de uma variável depois que ela foi declarada?

**Resposta:** Porque Java é uma linguagem estaticamente tipada. O tipo da variável faz parte do contrato validado pelo compilador. Se uma variável foi declarada como `int`, ela deve continuar recebendo valores compatíveis com `int`. Isso evita muitos erros antes mesmo da aplicação executar. A Java Language Specification define os tipos primitivos e os contextos de conversão que determinam quando uma atribuição é válida ou não. ([Oracle Docs][9])

```java
int totalFaixas = 8;

// totalFaixas = "oito"; // não compila
```

---

### 3. Qual é a diferença entre `int` e `long`?

**Resposta:** `int` é um tipo inteiro de 32 bits, enquanto `long` é um tipo inteiro de 64 bits. Na prática, `int` é usado para quantidades comuns, contadores e cálculos inteiros do dia a dia. `long` é usado quando o valor pode ser muito grande, como IDs, timestamps e contadores globais de alta escala. ([Oracle Docs][9])

```java
int totalFaixas = 12;
long totalReproducoes = 9_500_000_000L;
```

---

### 4. Por que `2_147_483_648` precisa do sufixo `L` para ser tratado como `long`?

**Resposta:** Porque literais inteiros sem sufixo são interpretados inicialmente como `int`. O valor `2_147_483_648` ultrapassa o limite máximo de `int`, que é `2_147_483_647`. Por isso, é necessário usar `L` para indicar que o literal deve ser tratado como `long`. ([Oracle Docs][9])

```java
// long valor = 2_147_483_648; // não compila

long valor = 2_147_483_648L; // correto
```

---

### 5. Por que `3 / 2` resulta em `1`, mas `3 / 2.0` resulta em `1.5`?

**Resposta:** Porque `3 / 2` é uma divisão entre dois valores inteiros. Nesse caso, Java faz divisão inteira e descarta a parte decimal. Já `3 / 2.0` envolve um valor de ponto flutuante (`double`), então a operação é promovida para `double` e o resultado preserva a parte decimal. As regras de promoção numérica são definidas pela JLS. ([Oracle Docs][9])

```java
System.out.println(3 / 2);     // 1
System.out.println(3 / 2.0);   // 1.5
```

---

### 6. Por que `byte + byte` resulta em `int`?

**Resposta:** Porque Java aplica promoção numérica em operações aritméticas. Tipos menores que `int`, como `byte`, `short` e `char`, normalmente são promovidos para `int` antes da operação. Por isso, o resultado de `byte + byte` é `int`, não `byte`. ([Oracle Docs][9])

```java
byte a = 10;
byte b = 20;

// byte resultado = a + b; // não compila

int resultado = a + b; // correto
```

---

### 7. O que acontece quando um `double` é convertido para `int`?

**Resposta:** A parte decimal é descartada. Isso não é arredondamento; é truncamento em direção a zero. Por isso, a conversão de `double` para `int` exige cast explícito, porque pode haver perda de informação. ([Oracle Docs][9])

```java
double nota = 4.95;

int notaInteira = (int) nota;

System.out.println(notaInteira); // 4
```

---

### 8. Por que `char` não deve ser tratado como texto completo?

**Resposta:** Porque `char` representa uma unidade de código UTF-16 de 16 bits, não necessariamente um texto completo ou um caractere visual completo. Para textos, nomes, títulos e mensagens, o tipo adequado é `String`. ([Oracle Docs][9])

```java
char inicial = 'R';

String banda = "Rush";
String album = "Moving Pictures";
```

---

### 9. Qual é o risco de usar cast explícito sem validar a faixa do valor?

**Resposta:** O risco é perder informação ou gerar um valor inesperado. Um cast explícito diz ao compilador: “eu sei o que estou fazendo”. Mas, se o valor não couber no tipo de destino, o resultado pode ficar incorreto. A JLS trata essas conversões como conversões de estreitamento, que podem perder magnitude ou precisão. ([Oracle Docs][9])

```java
long totalReproducoes = 9_300_000_035L;

int valorConvertido = (int) totalReproducoes;

System.out.println(valorConvertido); // valor inesperado
```

---

### 10. Por que `BigDecimal` é preferível a `double` para valores monetários?

**Resposta:** Porque `double` usa ponto flutuante binário e pode representar alguns valores decimais de forma aproximada. Para dinheiro, isso pode gerar pequenas diferenças indesejadas. `BigDecimal` permite trabalhar com aritmética decimal, escala e arredondamento controlado. A documentação oficial destaca que `BigDecimal` dá controle completo sobre o comportamento de arredondamento. ([Oracle Docs][5])

```java
BigDecimal preco = new BigDecimal("39.90");
BigDecimal desconto = new BigDecimal("5.00");

BigDecimal precoFinal = preco.subtract(desconto);

System.out.println(precoFinal); // 34.90
```

---

### 11. Qual é a diferença entre `total++` e `++total`?

**Resposta:** Os dois incrementam a variável em 1, mas retornam valores diferentes quando usados dentro de uma expressão. `total++` é pós-incremento: primeiro usa o valor atual e depois incrementa. `++total` é pré-incremento: primeiro incrementa e depois usa o novo valor. ([Oracle Docs][9])

```java
int total = 10;

int a = total++; // a recebe 10, total vira 11
int b = ++total; // total vira 12, b recebe 12
```

Em código profissional, é melhor evitar incremento dentro de expressões complexas, porque isso reduz a legibilidade.

---

### 12. Em quais situações `byte` e `short` fazem sentido em aplicações backend?

**Resposta:** Eles fazem sentido quando existe uma necessidade real de representar dados pequenos ou compatíveis com formatos externos, como arquivos binários, protocolos, buffers, integrações legadas ou estruturas compactas. Para variáveis comuns de regra de negócio, `int` costuma ser mais simples e legível.

```java
byte tipoEvento = 1;
short codigoLegado = 32000;
int quantidadeFaixas = 12;
```

Na maioria dos cálculos, `byte` e `short` acabam sendo promovidos para `int`, então usá-los apenas para “economizar memória” em variáveis locais geralmente não traz benefício prático.

---

### 13. Quando `var`, introduzido no Java 10 pela JEP 286, melhora a legibilidade e quando pode piorá-la?

**Resposta:** `var` melhora a legibilidade quando o tipo é óbvio pelo lado direito da atribuição e repetir o tipo deixaria o código mais poluído. Ele pode piorar a leitura quando esconde informação importante sobre o domínio ou torna o tipo menos evidente. A JEP 286 introduziu inferência de tipo para variáveis locais com inicializador, mas isso não torna Java uma linguagem dinâmica: o tipo continua sendo definido em tempo de compilação. ([OpenJDK][7])

Bom uso:

```java
var preco = new BigDecimal("39.90");
var bandas = List.of("Rush", "Metallica", "Pink Floyd");
```

Uso que pode prejudicar a leitura:

```java
var resultado = buscarDados();
```

Nesse caso, `resultado` não comunica claramente se é uma lista, um mapa, um DTO, uma entidade ou outro tipo.

---

### 14. O que significa promoção aritmética?

**Resposta:** Promoção aritmética é o processo pelo qual Java ajusta os tipos dos operandos antes de realizar uma operação numérica. Isso define o tipo final da expressão. Em geral, `double` tem prioridade sobre `float`, `float` sobre `long`, `long` sobre `int`, e tipos menores como `byte`, `short` e `char` são promovidos para `int`. ([Oracle Docs][9])

```java
int x = 10;
long y = 5;

long resultado = x * y; // x é promovido para long
```

Outro exemplo:

```java
int a = 3;
int b = 2;

double resultado = a / (double) b; // resultado 1.5
```

---

### 15. Por que operadores compostos como `+=` podem esconder uma conversão implícita?

**Resposta:** Porque operadores compostos fazem uma conversão automática para o tipo da variável à esquerda. Em termos conceituais, `x += y` se comporta como `x = (tipoDeX) (x + y)`. Isso pode ser conveniente, mas também pode esconder perda de informação em alguns casos. A JLS define esse comportamento para operadores de atribuição compostos. ([Oracle Docs][9])

```java
short quantidade = 10;

quantidade += 1; // compila

// quantidade = quantidade + 1; // não compila
```

O segundo caso não compila porque `quantidade + 1` resulta em `int`. O primeiro compila porque o operador `+=` aplica a conversão implícita de volta para `short`.

## Resumo final

Variáveis são nomes dados a valores que o programa manipula. Em Java, cada variável tem um tipo, e esse tipo define o que pode ser armazenado, quais operações são permitidas e quais conversões são seguras.

Os tipos primitivos formam a base da linguagem: `boolean` para valores lógicos, `byte`, `short`, `int`, `long` e `char` para valores integrais, e `float` e `double` para valores aproximados de ponto flutuante.

Operadores aritméticos permitem somar, subtrair, multiplicar, dividir e obter resto de divisão. Mas eles seguem regras de precedência, concatenação com `String`, promoção aritmética e divisão inteira.

Conversões de ampliação costumam ser automáticas, mas ainda podem perder precisão em alguns casos. Conversões de estreitamento exigem cast e podem perder informação de forma significativa.

`byte`, `short` e `char` merecem atenção especial porque operações aritméticas com eles geralmente resultam em `int`.

Para valores monetários, `BigDecimal` é a escolha mais segura porque oferece aritmética decimal e controle explícito de escala e arredondamento.

Dominar esses fundamentos é essencial para escrever código Java confiável. Eles parecem simples, mas afetam diretamente legibilidade, segurança, performance, persistência, integrações e regras de negócio em sistemas backend reais.

## Referências

* **Java Language Specification, Java SE 25 — Chapter 4: Types, Values, and Variables**. Base para tipos primitivos, tipos de referência, faixas dos tipos integrais, `char`, `float`, `double` e `boolean`. ([Oracle Docs][2])
* **Java Language Specification, Java SE 25 — Chapter 5: Conversions and Contexts**. Base para conversões primitivas, casting, widening, narrowing e promoção numérica. ([Oracle Docs][6])
* **Java Language Specification, Java SE 25 — Chapter 15: Expressions**. Base para operadores aritméticos, concatenação com `String`, incremento/decremento e operadores de atribuição compostos. ([Oracle Docs][4])
* **Java SE 25 API Documentation — `java.math.BigDecimal`**. Referência para aritmética decimal, escala, arredondamento e comportamento de operações exatas. ([Oracle Docs][5])
* **OpenJDK — JDK 25 Project Page**. Referência sobre disponibilidade geral do JDK 25 e relação com Java SE 25 / JSR 400. ([OpenJDK][1])
* **OpenJDK — JEP 286: Local-Variable Type Inference**. Referência sobre `var`, introduzido no Java 10. ([OpenJDK][7])
* **OpenJDK — JEP 512: Compact Source Files and Instance Main Methods**. Referência sobre arquivos compactos e métodos `main` de instância finalizados no JDK 25. ([OpenJDK][8])
* **Project Coin / JSR 334**. Referência histórica para underscores em literais numéricos e outras pequenas melhorias de linguagem introduzidas no Java 7. ([OpenJDK CR][3])

[1]: https://openjdk.org/projects/jdk/25/ "JDK 25"
[2]: https://docs.oracle.com/javase/specs/jls/se25/html/jls-4.html "Chapter 4. Types, Values, and Variables"
[3]: https://cr.openjdk.org/~darcy/ProjectCoin/ProjectCoin-Documentation-v0.83.html "Project Coin/JSR 334 Documentation v0.83"
[4]: https://docs.oracle.com/javase/specs/jls/se25/html/jls-15.html "Chapter 15. Expressions"
[5]: https://docs.oracle.com/en/java/javase/25/docs/api//java.base/java/math/BigDecimal.html "BigDecimal (Java SE 25 & JDK 25)"
[6]: https://docs.oracle.com/javase/specs/jls/se25/html/jls-5.html "Chapter 5. Conversions and Contexts"
[7]: https://openjdk.org/jeps/286 "JEP 286: Local-Variable Type Inference"
[8]: https://openjdk.org/jeps/512 "JEP 512: Compact Source Files and Instance Main Methods"
[9]: https://docs.oracle.com/javase/specs/jls/se25/html/index.html "The Java® Language Specification"
