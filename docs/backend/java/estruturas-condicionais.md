# Estruturas condicionais em Java

Aplicações backend raramente executam sempre o mesmo caminho. Um sistema precisa tomar decisões: liberar ou bloquear uma operação, aplicar uma regra de desconto, validar uma entrada, escolher uma estratégia de cobrança, classificar um conteúdo, rejeitar uma requisição inválida ou decidir qual mensagem retornar para uma API.

Em uma aplicação como **meucatalogomusical.com**, por exemplo, uma música pode receber selos diferentes conforme nota média, quantidade de avaliações, gênero, duração e tipo de usuário. Uma faixa de Rock Progressivo com mais de oito minutos pode aparecer como “faixa longa recomendada”, enquanto uma música com poucas avaliações talvez ainda não deva receber destaque.

Essas decisões são representadas em Java por estruturas condicionais e expressões condicionais: `if`, `else`, `else if`, `switch`, `switch expression` e operador ternário. A linguagem Java define formalmente essas estruturas na Java Language Specification; o `if` está na seção de statements, o operador ternário em expressões e o `switch` possui regras próprias de seleção, compatibilidade de tipos, exaustividade e execução. ([Oracle Docs][1])

## Antes de entrar no código: uma ideia simples

Uma estrutura condicional responde à pergunta:

> “Dado este estado, qual caminho o programa deve seguir?”

Exemplos:

```java
if (usuarioPremium) {
    liberarConteudoExclusivo();
}
```

```java
String selo = nota >= 9 ? "Destaque" : "Catálogo";
```

```java
String descricaoGenero = switch (genero) {
    case HEAVY_METAL -> "Metal";
    case HARD_ROCK -> "Hard Rock";
    case ROCK_PROGRESSIVO -> "Rock Progressivo";
};
```

A diferença principal entre essas formas está no tipo de decisão:

* `if`: melhor para condições booleanas livres.
* `else if`: melhor para faixas, prioridades e regras encadeadas.
* `switch`: melhor para escolher caminhos a partir de um mesmo valor.
* `switch expression`: melhor quando a decisão precisa produzir um valor.
* `operador ternário`: melhor quando uma condição simples escolhe entre dois valores.

## Definição técnica

Em Java, uma estrutura condicional é um mecanismo de controle de fluxo que permite executar blocos diferentes conforme uma expressão booleana ou conforme o valor de uma expressão seletora.

O `if` exige uma expressão compatível com `boolean` ou `Boolean`. Se a condição for verdadeira, o bloco associado é executado. Se houver `else`, ele será executado quando a condição do `if` for falsa. Em uma cadeia `else if`, o Java testa as condições na ordem e executa o primeiro bloco cuja condição for verdadeira.

O `switch` transfere o controle para um dos rótulos `case` ou `default`, de acordo com o valor da expressão seletora. No Java 25, sem habilitar recursos preview, a expressão seletora de `switch` pode ser `char`, `byte`, `short`, `int` ou tipo de referência. A própria JLS destaca que `boolean`, `long`, `float` e `double` não são tipos projetados para uso direto como seletor de `switch` padrão. ([Oracle Docs][2])

O operador ternário `? :` é uma expressão, não um statement. Ele usa o valor booleano da primeira expressão para escolher qual das outras duas expressões será avaliada. A expressão não escolhida não é avaliada naquela execução. ([Oracle Docs][3])

## Como funciona em Java

### `if`

O `if` é a forma mais direta de condicional.

```java
if (notaMedia >= 9) {
    System.out.println("Música em destaque.");
}
```

Aqui, o bloco só será executado se `notaMedia >= 9` resultar em `true`.

### `if` com `else`

O `else` representa o caminho alternativo.

```java
if (notaMedia >= 9) {
    System.out.println("Música em destaque.");
} else {
    System.out.println("Música comum no catálogo.");
}
```

Se a condição do `if` for falsa, o bloco do `else` será executado.

### `else if`

O `else if` organiza decisões mutuamente exclusivas.

```java
if (notaMedia >= 9) {
    System.out.println("Clássico essencial.");
} else if (notaMedia >= 7) {
    System.out.println("Boa recomendação.");
} else if (notaMedia >= 5) {
    System.out.println("Catálogo básico.");
} else {
    System.out.println("Baixa recomendação.");
}
```

Esse formato é diferente de escrever vários `if` separados. Em vários `if` independentes, todas as condições são testadas. Em uma cadeia `if / else if / else`, o Java para no primeiro bloco verdadeiro.

### `switch` tradicional

O `switch` tradicional é útil quando a decisão depende de um único valor.

```java
String descricao;

switch (nota) {
    case 1:
        descricao = "Muito ruim";
        break;
    case 2:
        descricao = "Ruim";
        break;
    case 3:
        descricao = "Razoável";
        break;
    case 4:
        descricao = "Muito bom";
        break;
    case 5:
        descricao = "Excelente";
        break;
    default:
        descricao = "Nota inválida";
}
```

O `break` encerra a execução do `switch`. Sem `break`, o fluxo continua nos próximos `case`, comportamento conhecido como **fall-through**. Esse comportamento existe na linguagem e pode ser intencional, mas costuma ser uma fonte de bugs quando acontece por esquecimento.

### `switch expression`

Desde o Java 14, por meio da JEP 361, `switch` também pode ser usado como expressão, ou seja, pode produzir um valor. Essa evolução introduziu os rótulos com seta (`case X ->`) e o uso de `yield` quando um bloco precisa devolver um valor. ([OpenJDK][4])

```java
String descricao = switch (nota) {
    case 1 -> "Muito ruim";
    case 2 -> "Ruim";
    case 3 -> "Razoável";
    case 4 -> "Muito bom";
    case 5 -> "Excelente";
    default -> "Nota inválida";
};
```

Esse formato reduz a chance de esquecer `break` e deixa mais claro que o objetivo da decisão é produzir um valor.

### Operador ternário

O operador ternário é adequado para escolhas simples entre dois valores.

```java
String tipoAcesso = usuarioPremium ? "Premium" : "Gratuito";
```

Ele deve ser usado com moderação. Quando a regra fica grande, o `if` ou um método bem nomeado costuma ser mais legível.

## Exemplo prático

Vamos usar um exemplo da aplicação fictícia **meucatalogomusical.com**. O objetivo é classificar uma música do catálogo com base em algumas regras:

* músicas com poucas avaliações não recebem selo definitivo;
* notas altas recebem destaque;
* Rock Progressivo com duração longa recebe uma descrição especial;
* músicas ao vivo podem receber um selo adicional para usuários premium;
* o gênero musical é traduzido com `switch expression`;
* uma condição simples usa operador ternário.

```java
public class ClassificadorMusical {

    enum Genero {
        HEAVY_METAL,
        HARD_ROCK,
        ROCK_PROGRESSIVO
    }

    record Musica(
            String titulo,
            String banda,
            Genero genero,
            int duracaoEmSegundos,
            int notaMedia,
            int quantidadeAvaliacoes,
            boolean aoVivo
    ) {
    }

    public static void main(String[] args) {
        Musica musica = new Musica(
                "Fear of the Dark",
                "Iron Maiden",
                Genero.HEAVY_METAL,
                438,
                9,
                2150,
                true
        );

        String classificacao = classificar(musica, true);

        System.out.println(classificacao);
    }

    static String classificar(Musica musica, boolean usuarioPremium) {
        if (musica.quantidadeAvaliacoes() < 10) {
            return "Aguardando mais avaliações para classificação.";
        }

        String descricaoGenero = switch (musica.genero()) {
            case HEAVY_METAL -> "Heavy Metal";
            case HARD_ROCK -> "Hard Rock";
            case ROCK_PROGRESSIVO -> "Rock Progressivo";
        };

        String seloNota;

        if (musica.notaMedia() >= 9) {
            seloNota = "Clássico essencial";
        } else if (musica.notaMedia() >= 7) {
            seloNota = "Boa recomendação";
        } else if (musica.notaMedia() >= 5) {
            seloNota = "Catálogo básico";
        } else {
            seloNota = "Baixa recomendação";
        }

        String formato = musica.aoVivo() ? "versão ao vivo" : "versão de estúdio";

        if (musica.genero() == Genero.ROCK_PROGRESSIVO && musica.duracaoEmSegundos() >= 480) {
            return "%s de %s: %s, faixa longa de %s."
                    .formatted(musica.titulo(), musica.banda(), seloNota, descricaoGenero);
        }

        if (usuarioPremium && musica.aoVivo()) {
            return "%s de %s: %s, %s disponível para usuários premium."
                    .formatted(musica.titulo(), musica.banda(), seloNota, formato);
        }

        return "%s de %s: %s no catálogo de %s."
                .formatted(musica.titulo(), musica.banda(), seloNota, descricaoGenero);
    }
}
```

## Analisando o código

O `enum Genero` limita os gêneros possíveis a três valores. Isso ajuda o compilador e melhora a modelagem: em vez de espalhar strings como `"metal"`, `"hard"` ou `"prog"` pelo código, o domínio trabalha com valores controlados.

O `record Musica` representa uma música de forma imutável e concisa. O exemplo usa recursos modernos da linguagem para manter o foco na regra de decisão.

A primeira condicional é uma **guarda**:

```java
if (musica.quantidadeAvaliacoes() < 10) {
    return "Aguardando mais avaliações para classificação.";
}
```

Esse padrão é comum em backend. Quando uma regra impede a continuação do fluxo, o método retorna cedo. Isso evita aninhar todo o restante do código dentro de um grande `else`.

Depois, o código usa `switch expression` para transformar o enum em uma descrição textual:

```java
String descricaoGenero = switch (musica.genero()) {
    case HEAVY_METAL -> "Heavy Metal";
    case HARD_ROCK -> "Hard Rock";
    case ROCK_PROGRESSIVO -> "Rock Progressivo";
};
```

Como todos os valores do enum foram tratados, o `default` não é necessário. Isso é útil porque, se um novo gênero for adicionado ao enum e o código for recompilado, o compilador pode exigir que a nova possibilidade seja tratada.

A classificação por nota usa `if / else if / else`, porque a regra depende de faixas numéricas:

```java
if (musica.notaMedia() >= 9) {
    seloNota = "Clássico essencial";
} else if (musica.notaMedia() >= 7) {
    seloNota = "Boa recomendação";
} else if (musica.notaMedia() >= 5) {
    seloNota = "Catálogo básico";
} else {
    seloNota = "Baixa recomendação";
}
```

Aqui a ordem importa. Se a condição `nota >= 5` viesse antes de `nota >= 9`, uma nota 9 cairia no bloco errado.

O operador ternário aparece em uma escolha simples:

```java
String formato = musica.aoVivo() ? "versão ao vivo" : "versão de estúdio";
```

Essa é uma boa aplicação do ternário: uma condição simples, dois valores simples, sem cálculo complexo dentro da expressão.

## O que acontece por baixo dos panos

### Escopo de variáveis

Uma variável declarada dentro de um bloco só existe naquele bloco.

```java
if (deveCobrarFrete) {
    double valorFrete = 12.90;
}

double total = valorDiscos + valorFrete; // não compila
```

Esse código não compila porque `valorFrete` foi declarada dentro do bloco do `if`. Fora do bloco, ela não está no escopo.

O correto é declarar a variável no menor escopo possível, mas amplo o suficiente para todos os usos necessários:

```java
double valorFrete = 0.0;

if (deveCobrarFrete) {
    valorFrete = 12.90;
}

double total = valorDiscos + valorFrete;
```

A JLS diferencia variáveis locais de campos. Campos de classe, campos de instância e componentes de array recebem valores padrão quando são criados. Variáveis locais, por outro lado, precisam receber valor explicitamente antes de serem usadas, e isso deve ser verificável pelo compilador por meio das regras de **definite assignment**. ([Oracle Docs][5])

Por isso este código compila:

```java
public class Pedido {
    static double valorFrete;

    public static void main(String[] args) {
        double total = 50.0 + valorFrete;
        System.out.println(total);
    }
}
```

`valorFrete` é um campo estático. Como é um campo do tipo `double`, recebe automaticamente o valor padrão `0.0`.

Mas este código não compila:

```java
public class Pedido {
    public static void main(String[] args) {
        double valorFrete;

        double total = 50.0 + valorFrete; // variável pode não ter sido inicializada
        System.out.println(total);
    }
}
```

A regra não muda por ser tipo primitivo ou referência. Uma variável local do tipo `String` também precisa estar definitivamente inicializada antes do uso:

```java
String descricao;

if (nota == 5) {
    descricao = "Excelente";
}

System.out.println(descricao); // não compila
```

Isso não gera `NullPointerException`. O problema acontece antes: é erro de compilação. `NullPointerException` aconteceria se a variável estivesse inicializada com `null` e o código tentasse acessar um membro dela:

```java
String descricao = null;

System.out.println(descricao.length()); // compila, mas lança NullPointerException em execução
```

Portanto, no caso de uma variável local como `String descricaoNota;`, ela pode ser declarada sem valor inicial **somente se o compilador conseguir provar que todos os caminhos até o uso atribuem algum valor a ela**.

### Campos estáticos não substituem variáveis locais

Embora um campo estático receba valor padrão, isso não significa que ele deva ser usado para evitar inicialização local.

```java
static double valorFrete;
```

Esse campo pertence à classe, não a uma execução específica do método. Em sistemas backend, campos estáticos mutáveis podem criar problemas sérios: estado compartilhado, dificuldade de teste, concorrência, comportamento inesperado entre requisições e acoplamento global.

Para uma informação temporária de cálculo, como frete de um pedido, prefira variável local:

```java
double valorFrete = deveCobrarFrete ? calcularFrete() : 0.0;
double total = valorProdutos + valorFrete;
```

### `switch`, `break` e fall-through

No `switch` tradicional, quando um `case` é encontrado, a execução continua até encontrar um `break`, `return`, `throw` ou o fim do bloco.

```java
switch (nota) {
    case 4:
        descricao = "Muito bom";
    case 5:
        descricao = "Excelente";
    default:
        descricao = "Opção inválida";
}
```

Se `nota` for `4`, o código entra no `case 4`, mas continua executando `case 5` e `default`. O resultado final será `"Opção inválida"`.

Com `break`:

```java
switch (nota) {
    case 4:
        descricao = "Muito bom";
        break;
    case 5:
        descricao = "Excelente";
        break;
    default:
        descricao = "Opção inválida";
}
```

Agora cada caso encerra corretamente sua execução.

O fall-through pode ser útil quando vários casos compartilham o mesmo bloco:

```java
switch (diaSemana) {
    case "ter":
    case "qua":
    case "qui":
    case "sex":
        horario = "08:00 às 18:00";
        break;
    case "sab":
    case "dom":
        horario = "08:00 às 12:00";
        break;
    default:
        horario = "Fechado";
}
```

Mas, em código moderno, normalmente é mais claro usar `switch expression`:

```java
String horario = switch (diaSemana) {
    case "ter", "qua", "qui", "sex" -> "08:00 às 18:00";
    case "sab", "dom" -> "08:00 às 12:00";
    default -> "Fechado";
};
```

### Tipos aceitos no `switch`

No Java 25 padrão, o seletor de `switch` aceita `char`, `byte`, `short`, `int` e tipos de referência. Isso inclui wrappers como `Character`, `Byte`, `Short`, `Integer`, `String`, enums e, desde Java 21, uso mais amplo com tipos de referência em pattern matching para `switch`. A JLS deixa explícito que `boolean`, `long`, `float` e `double` não são aceitos no `switch` padrão. ([Oracle Docs][2])

Resumo prático:

| Tipo                                             | `switch` padrão em Java 25? | Observação                                      |
| ------------------------------------------------ | --------------------------: | ----------------------------------------------- |
| `byte`                                           |                         Sim | Promovido/tratado de forma compatível com `int` |
| `short`                                          |                         Sim | Promovido/tratado de forma compatível com `int` |
| `char`                                           |                         Sim | Tipo integral                                   |
| `int`                                            |                         Sim | Caso clássico                                   |
| `long`                                           |                         Não | Não aceito no `switch` padrão                   |
| `float`                                          |                         Não | Não aceito no `switch` padrão                   |
| `double`                                         |                         Não | Não aceito no `switch` padrão                   |
| `boolean`                                        |                         Não | Use `if` ou ternário                            |
| `String`                                         |                         Sim | Suporte introduzido no Java 7                   |
| `enum`                                           |                         Sim | Muito recomendado para estados conhecidos       |
| wrappers (`Integer`, etc.)                       |                         Sim | Atenção a `null`                                |
| qualquer tipo de referência com pattern matching |          Sim, desde Java 21 | Regras de exaustividade e padrões               |

Existe uma exceção importante em Java 25: a JEP 507, **Primitive Types in Patterns, instanceof, and switch**, reintroduz em preview o suporte para tipos primitivos em contextos de pattern matching, `instanceof` e `switch`, incluindo a ideia de `switch` funcionar com todos os tipos primitivos. Como é preview, não deve ser tratado como recurso estável para código produtivo sem uma decisão explícita do time. ([OpenJDK][6])

### Evolução do `switch` até Java 25

| Versão       | Evolução                                                                 | JEP / referência                                        |
| ------------ | ------------------------------------------------------------------------ | ------------------------------------------------------- |
| Java inicial | `switch` clássico com `case`, `default`, `break` e fall-through          | JLS                                                     |
| Java 5       | Uso comum com enums                                                      | JLS                                                     |
| Java 7       | `String` passou a ser aceito em `switch`                                 | Documentação Oracle do Java Tutorial ([Oracle Docs][7]) |
| Java 12      | `switch expressions` em preview                                          | JEP 325                                                 |
| Java 13      | refinamento em preview                                                   | JEP 354                                                 |
| Java 14      | `switch expressions` finalizado                                          | JEP 361 ([OpenJDK][4])                                  |
| Java 17 a 20 | pattern matching for switch em preview                                   | JEPs 406, 420, 427, 433                                 |
| Java 21      | pattern matching for switch finalizado                                   | JEP 441 ([OpenJDK][8])                                  |
| Java 25      | primitive types em patterns, `instanceof` e `switch` em terceira preview | JEP 507 ([OpenJDK][6])                                  |

### Bytecode e performance

Em alguns casos, o compilador pode transformar um `switch` em instruções específicas da JVM, como `tableswitch` e `lookupswitch`. A especificação da JVM descreve `tableswitch` como acesso a uma tabela de saltos por índice e `lookupswitch` como acesso por correspondência de chave. Ambas trabalham com chave/index do tipo `int`. ([Oracle Docs][9])

Isso não significa que todo `switch` será sempre mais rápido que `if`. Em código de negócio, a decisão entre `if` e `switch` deve priorizar clareza, manutenibilidade e modelagem. Performance só deve guiar essa escolha quando houver medição real.

## Boas práticas

Prefira `if` quando a regra for uma condição booleana clara:

```java
if (usuarioPremium && musica.aoVivo()) {
    liberarVersaoEspecial();
}
```

Prefira `else if` quando houver faixas ou prioridades:

```java
if (nota >= 9) {
    selo = "Clássico";
} else if (nota >= 7) {
    selo = "Recomendado";
} else {
    selo = "Catálogo";
}
```

Prefira `switch expression` quando a decisão tiver como objetivo produzir um valor:

```java
String nomeGenero = switch (genero) {
    case HEAVY_METAL -> "Heavy Metal";
    case HARD_ROCK -> "Hard Rock";
    case ROCK_PROGRESSIVO -> "Rock Progressivo";
};
```

Use `enum` para estados conhecidos em vez de strings soltas:

```java
enum StatusPublicacao {
    RASCUNHO,
    PUBLICADO,
    ARQUIVADO
}
```

Evite campos estáticos mutáveis para dados temporários. Variáveis locais são melhores para cálculos de uma execução específica.

Use retornos antecipados para reduzir aninhamento:

```java
if (musica == null) {
    throw new IllegalArgumentException("Música é obrigatória.");
}

if (musica.quantidadeAvaliacoes() < 10) {
    return "Aguardando avaliações.";
}

return "Classificação disponível.";
```

Evite ternários aninhados:

```java
// Evite
String selo = nota >= 9 ? "Clássico" : nota >= 7 ? "Recomendado" : "Catálogo";
```

Prefira:

```java
String selo;

if (nota >= 9) {
    selo = "Clássico";
} else if (nota >= 7) {
    selo = "Recomendado";
} else {
    selo = "Catálogo";
}
```

## Erros comuns

### Esquecer `break` no `switch` tradicional

```java
switch (nota) {
    case 5:
        descricao = "Excelente";
    default:
        descricao = "Inválida";
}
```

Se `nota` for `5`, `descricao` terminará como `"Inválida"`.

### Usar vários `if` quando as condições são exclusivas

```java
if (nota >= 5) {
    selo = "Catálogo";
}

if (nota >= 7) {
    selo = "Recomendado";
}

if (nota >= 9) {
    selo = "Clássico";
}
```

Esse código até pode funcionar, mas avalia condições desnecessárias e depende da ordem das sobrescritas. Em regras exclusivas, `else if` comunica melhor a intenção.

### Declarar variável dentro de um bloco e tentar usar fora

```java
if (deveAplicarDesconto) {
    double desconto = 10.0;
}

double total = valor - desconto; // não compila
```

A variável `desconto` só existe dentro do bloco.

### Confundir variável local não inicializada com `NullPointerException`

```java
String descricao;
System.out.println(descricao); // erro de compilação
```

Isso não é `NullPointerException`. É erro de compilação por falta de inicialização definitiva.

### Usar `static` para fugir da inicialização local

```java
static double desconto;
```

Isso evita o erro de compilação, mas muda o significado do dado. Agora `desconto` é estado global da classe. Em backend, isso pode ser perigoso.

### Usar `switch` com `long` ou `double` no Java padrão

```java
long codigo = 10L;

switch (codigo) {
    case 10L -> System.out.println("Código 10");
}
```

No Java 25 padrão, isso não compila. Para `long`, `double`, `float` e `boolean`, use `if`, normalize para um tipo aceito ou avalie explicitamente recursos preview se isso fizer sentido para o projeto.

### Colocar regra de negócio complexa dentro de ternário

```java
double taxa = clienteVip && total > 500 && cupomValido ? 0.05 : calcularTaxaPadrao();
```

Ainda é legível para alguns casos, mas esse tipo de regra tende a crescer. Quando a condição tem muitas partes, um método nomeado melhora a leitura:

```java
double taxa = deveAplicarTaxaPromocional(cliente, pedido)
        ? 0.05
        : calcularTaxaPadrao();
```

## Decisões de engenharia

### Quando usar `if`

Use `if` quando a regra for naturalmente booleana:

* validar entrada;
* verificar permissão;
* bloquear fluxo;
* aplicar uma regra específica;
* retornar cedo em casos inválidos.

Exemplo:

```java
if (!usuarioPremium) {
    throw new IllegalStateException("Conteúdo disponível apenas para usuários premium.");
}
```

### Quando usar `else if`

Use `else if` quando houver uma ordem de prioridade ou faixas:

* classificação por nota;
* cálculo por faixa de preço;
* regras progressivas;
* validações que devem parar no primeiro caso encontrado.

Exemplo:

```java
if (nota >= 9) {
    return "Clássico essencial";
} else if (nota >= 7) {
    return "Boa recomendação";
} else {
    return "Catálogo básico";
}
```

### Quando usar `switch`

Use `switch` quando a decisão depende de um único valor discreto:

* `enum`;
* código de status interno;
* tipo de operação;
* categoria;
* estado de workflow.

Exemplo:

```java
return switch (status) {
    case RASCUNHO -> "Ainda não publicado";
    case PUBLICADO -> "Disponível no catálogo";
    case ARQUIVADO -> "Removido da navegação";
};
```

### Quando evitar `switch`

Evite `switch` quando:

* a lógica depende de muitas variáveis diferentes;
* cada caso cresce demais;
* os casos representam comportamentos polimórficos;
* a regra muda frequentemente por configuração;
* o código começa a virar uma grande central de decisões.

Em casos assim, alternativas podem ser melhores:

* polimorfismo;
* Strategy;
* Specification;
* Map de funções;
* regras configuráveis;
* validações separadas por classe;
* métodos pequenos e bem nomeados.

### Impacto em legibilidade

`switch expression` costuma ser mais legível que `switch` tradicional quando a decisão produz valor. O formato com `->` reduz ruído e evita `break`.

### Impacto em manutenção

Condicionais longas demais indicam que talvez exista uma abstração faltando. Quando uma classe concentra muitos `if` e `switch`, ela pode estar acumulando responsabilidade demais.

### Impacto em performance

Para a maioria das regras de negócio, a diferença de performance entre `if` e `switch` não importa. A JVM e o compilador já fazem várias otimizações. Só vale escolher por performance quando houver evidência por benchmark, profiling ou problema real em produção.

## Onde isso aparece em sistemas reais

Em APIs REST:

```java
if (request.titulo() == null || request.titulo().isBlank()) {
    return ResponseEntity.badRequest().body("Título é obrigatório.");
}
```

Em regras de autorização:

```java
if (!usuario.podeEditarCatalogo()) {
    throw new AcessoNegadoException("Usuário não pode editar o catálogo.");
}
```

Em workflows:

```java
String proximaAcao = switch (statusModeracao) {
    case PENDENTE -> "Enviar para revisão";
    case APROVADO -> "Publicar no catálogo";
    case REJEITADO -> "Solicitar ajustes";
};
```

Em classificação de dados:

```java
String faixaDuracao = musica.duracaoEmSegundos() >= 480
        ? "Faixa longa"
        : "Faixa curta ou média";
```

Em integração com outros serviços:

```java
if (resposta.statusCode() >= 500) {
    reagendarProcessamento();
} else if (resposta.statusCode() == 404) {
    registrarCatalogoNaoEncontrado();
} else {
    processarResposta();
}
```

Em sistemas reais, condicionais não servem apenas para “imprimir mensagens”. Elas expressam decisões de negócio. Por isso, devem ser escritas com clareza, testadas com cuidado e mantidas próximas da linguagem do domínio.

## Perguntas de revisão

1 - Qual problema as estruturas condicionais resolvem?

Resposta: elas permitem que o programa escolha caminhos diferentes durante a execução. Em vez de executar sempre a mesma sequência de comandos, o sistema pode validar dados, aplicar regras, bloquear operações, classificar resultados ou escolher respostas conforme o estado atual.

2 - Qual é a diferença entre `if` e `else`?

Resposta: o `if` executa um bloco quando a condição é verdadeira. O `else` executa um bloco alternativo quando a condição do `if` é falsa.

3 - Quando usar `else if`?

Resposta: use `else if` quando as condições são encadeadas e normalmente exclusivas. Ele é útil para faixas, prioridades e classificações, como nota maior ou igual a 9, depois maior ou igual a 7, depois maior ou igual a 5.

4 - Qual é a diferença entre vários `if` separados e uma cadeia `if / else if / else`?

Resposta: vários `if` separados são avaliados independentemente. Uma cadeia `if / else if / else` para no primeiro bloco cuja condição for verdadeira. Isso melhora a intenção quando apenas um caminho deve ser escolhido.

5 - O que é escopo de variável?

Resposta: escopo é a região do código onde uma variável pode ser acessada. Uma variável declarada dentro de um bloco `{ }` só existe dentro daquele bloco.

6 - Por que uma variável local precisa ser inicializada antes de ser usada?

Resposta: porque Java exige que variáveis locais estejam definitivamente atribuídas antes do uso. O compilador precisa conseguir provar que a variável recebeu valor em todos os caminhos possíveis até aquele ponto.

7 - Por que um campo estático `double valorFrete;` não precisa ser inicializado manualmente?

Resposta: porque campos de classe, campos de instância e componentes de array recebem valores padrão quando são criados. Para `double`, o padrão é `0.0`. Isso é diferente de variável local.

8 - Devo usar campo estático para evitar erro de variável local não inicializada?

Resposta: não. Campo estático representa estado da classe. Para valores temporários de cálculo, prefira variável local. Usar `static` sem necessidade pode gerar estado global, dificultar testes e causar problemas em aplicações concorrentes.

9 - Variáveis locais de referência, como `String`, precisam ser inicializadas?

Resposta: sim. A regra vale para tipos primitivos e tipos de referência. Uma variável local `String descricao;` só pode ser usada se o compilador conseguir provar que ela recebeu valor antes.

10 - Uma variável local `String` não inicializada causa `NullPointerException`?

Resposta: não. Ela causa erro de compilação. `NullPointerException` acontece em tempo de execução quando uma referência com valor `null` é usada para acessar membro, como `descricao.length()`.

11 - Quando usar `switch`?

Resposta: use `switch` quando a decisão depende de um único valor discreto, como um enum, uma string, um código inteiro ou um status.

12 - O que é fall-through no `switch` tradicional?

Resposta: é o comportamento em que, após entrar em um `case`, o fluxo continua executando os próximos `case` até encontrar `break`, `return`, `throw` ou o fim do bloco.

13 - O `break` é sempre obrigatório no `switch` tradicional?

Resposta: não é sempre obrigatório, mas geralmente é necessário para evitar fall-through acidental. Pode ser omitido de propósito quando vários `case` devem compartilhar o mesmo bloco.

14 - Qual é a vantagem de `switch expression`?

Resposta: ela permite que o `switch` produza um valor, reduz a necessidade de variável temporária, evita o fall-through com `->` e melhora a legibilidade quando a decisão é uma escolha de valor.

15 - Quando `switch expression` precisa de `default`?

Resposta: quando o compilador não consegue provar que todos os casos possíveis foram cobertos. Em um enum, se todos os valores forem tratados, o `default` pode não ser necessário.

16 - O `switch` aceita `long` em Java 25?

Resposta: no Java 25 padrão, não. `long`, `float`, `double` e `boolean` não são aceitos como seletores de `switch` padrão. A JEP 507 traz suporte ampliado em preview, mas isso exige habilitar recurso preview e não deve ser assumido como base estável sem decisão do projeto.

17 - O `switch` aceita `String`?

Resposta: sim. `String` passou a ser aceita em `switch` a partir do Java 7.

18 - O `double` funciona em `switch`?

Resposta: no Java padrão, não. Para decisões com `double`, normalmente use `if` com comparações por faixa ou normalize o valor para uma categoria representável por enum ou inteiro.

19 - Quando usar operador ternário?

Resposta: use ternário quando uma condição simples escolhe entre dois valores simples. Exemplo: `String tipo = premium ? "Premium" : "Gratuito";`.

20 - Por que evitar ternário aninhado?

Resposta: porque reduz a legibilidade. Quando existem várias condições, `if / else if / else`, `switch expression` ou métodos bem nomeados costumam deixar a regra mais clara.

21 - Condicionais afetam arquitetura?

Resposta: sim. Condicionais pequenas são normais. Mas condicionais enormes, repetidas ou espalhadas podem indicar falta de abstração, regra de negócio duplicada ou necessidade de polimorfismo, Strategy, Specification ou outro desenho melhor.

22 - O que um desenvolvedor pleno ou sênior deve observar em condicionais?

Resposta: deve observar clareza da regra, ordem das condições, cobertura de todos os cenários, testes para casos de borda, risco de estado global, duplicação de regras, legibilidade e facilidade de manutenção.

## Resumo final

Estruturas condicionais são fundamentais para expressar decisões em Java. O `if` resolve decisões booleanas diretas. O `else if` organiza faixas e prioridades. O `switch` é útil quando a decisão depende de um único valor. O `switch expression`, finalizado no Java 14, torna a escolha de valores mais concisa e segura. O operador ternário é uma alternativa enxuta para decisões simples entre dois valores.

Um ponto essencial é entender escopo e inicialização. Variáveis locais precisam ser explicitamente inicializadas antes do uso, enquanto campos recebem valores padrão. Essa diferença existe por desenho da linguagem e evita vários erros antes mesmo da execução.

Em sistemas backend, condicionais devem ser tratadas como parte da modelagem do domínio. Elas não são apenas sintaxe básica: representam regras de negócio. Quanto mais importante a regra, mais clareza, testes e organização ela exige.

## Referências

* Java Language Specification, Java SE 25 — capítulos sobre tipos, variáveis, escopo, statements, `if`, `switch`, expressões e definite assignment. ([Oracle Docs][1])
* Java Virtual Machine Specification, Java SE 25 — instruções `tableswitch` e `lookupswitch`. ([Oracle Docs][9])
* OpenJDK JEP 361 — Switch Expressions, recurso finalizado no Java 14. ([OpenJDK][4])
* OpenJDK JEP 441 — Pattern Matching for switch, finalizado no Java 21. ([OpenJDK][8])
* OpenJDK JEP 507 — Primitive Types in Patterns, instanceof, and switch, preview no Java 25. ([OpenJDK][6])
* Oracle Java Language Changes for Java SE 25 — histórico de mudanças da linguagem por versão. ([Oracle Docs][10])
* Oracle Java Tutorials — uso de `String` em `switch` a partir do Java SE 7. ([Oracle Docs][7])
* Joshua Bloch, *Effective Java* — boas práticas de clareza, imutabilidade, enums e desenho de APIs.
* Brian Goetz et al., *Java Concurrency in Practice* — referência para entender riscos de estado compartilhado e concorrência em aplicações Java.

[1]: https://docs.oracle.com/javase/specs/jls/se25/html/index.html "The Java® Language Specification"
[2]: https://docs.oracle.com/javase/specs/jls/se25/html/jls-14.html "Chapter 14. Blocks, Statements, and Patterns"
[3]: https://docs.oracle.com/javase/specs/jls/se25/html/jls-15.html "Chapter 15. Expressions"
[4]: https://openjdk.org/jeps/361 "JEP 361: Switch Expressions"
[5]: https://docs.oracle.com/javase/specs/jls/se25/html/jls-4.html "Chapter 4. Types, Values, and Variables"
[6]: https://openjdk.org/jeps/507 "JEP 507: Primitive Types in Patterns, instanceof, and ..."
[7]: https://docs.oracle.com/javase/tutorial/java/nutsandbolts/switch.html "The switch Statement (The Java™ Tutorials > Learning ..."
[8]: https://openjdk.org/jeps/441 "JEP 441: Pattern Matching for switch"
[9]: https://docs.oracle.com/javase/specs/jvms/se25/html/jvms-6.html "Chapter 6. The Java Virtual Machine Instruction Set"
[10]: https://docs.oracle.com/en/java/javase/25/language/java-language-changes-release.html "2 Java Language Changes by Release"
