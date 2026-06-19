# Classes e objetos em Java

Sistemas backend existem para representar, validar e processar informações do mundo real: clientes, pedidos, pagamentos, produtos, usuários, contratos, bandas, músicas, permissões, assinaturas, documentos fiscais e muitos outros conceitos.

Sem uma forma organizada de modelar esses conceitos, o código tende a virar um conjunto de variáveis soltas, mapas genéricos, listas sem significado e funções difíceis de manter. Isso até pode funcionar em programas pequenos, mas começa a falhar quando o sistema cresce.

Imagine o **meucatalogomusical.com**, uma aplicação fictícia para catalogar bandas e músicas de Metal, Hard Rock e Rock Progressivo.

O sistema precisa representar coisas como:

* uma banda;
* uma música;
* um álbum;
* o gênero musical;
* a duração de uma música;
* a relação entre uma música e sua banda;
* regras para cadastrar, exibir e validar essas informações.

Classes e objetos resolvem exatamente esse problema: eles permitem transformar conceitos importantes do domínio em estruturas de código com **estado**, **identidade** e **comportamento**.

Em Java, um objeto é uma instância criada dinamicamente a partir de uma classe, e variáveis de tipos de referência guardam referências para objetos, não os objetos diretamente. A Java Language Specification define objeto como uma instância de classe ou array, e explica que valores de tipos de referência são referências para objetos. ([Oracle Docs][1])

## Antes de entrar no código: uma ideia simples

Uma **classe** é uma definição. Ela descreve como determinado tipo de coisa deve ser representado no sistema.

Um **objeto** é uma ocorrência concreta dessa definição durante a execução do programa.

No contexto do **meucatalogomusical.com**, podemos pensar assim:

```text
Classe: Banda

Objetos:
- Metallica
- Iron Maiden
- Dream Theater
- Black Sabbath
```

Todos esses objetos podem ser do mesmo tipo, `Banda`, mas cada um possui seus próprios valores.

A classe define que uma banda tem, por exemplo:

```text
nome
país de origem
ano de formação
```

Mas cada objeto terá valores diferentes:

```text
Banda 1:
nome = "Metallica"
paisOrigem = "Estados Unidos"
anoFormacao = 1981

Banda 2:
nome = "Iron Maiden"
paisOrigem = "Reino Unido"
anoFormacao = 1975
```

A classe é a estrutura. O objeto é a instância real em memória.

## Definição técnica

Em Java, uma classe é uma declaração que pode conter membros, como campos, métodos, construtores, classes internas e interfaces internas. A JLS descreve que o corpo de uma classe declara membros, inicializadores e construtores. ([Oracle Docs][2])

De forma prática:

* **classe** define um tipo;
* **objeto** é uma instância desse tipo;
* **campo** representa estado;
* **método** representa comportamento;
* **construtor** inicializa o objeto;
* **referência** é o valor guardado em uma variável para acessar um objeto;
* **composição** acontece quando um objeto referencia outro objeto.

Exemplo conceitual:

```java
Banda metallica = new Banda("Metallica", "Estados Unidos", 1981);
```

Nessa linha:

* `Banda` é o tipo da variável;
* `metallica` é a variável de referência;
* `new Banda(...)` cria um novo objeto;
* o objeto criado fica disponível na memória;
* a variável `metallica` guarda uma referência para esse objeto.

A expressão de criação de instância com `new` é o mecanismo usado para criar novos objetos que são instâncias de classes. A JLS especifica que uma class instance creation expression cria novos objetos que são instâncias de classes. ([Oracle Docs][3])

## Como funciona em Java

### Classe

Uma classe define a estrutura e o comportamento de um tipo.

```java
public class Banda {
    private String nome;
    private String paisOrigem;
    private int anoFormacao;
}
```

Essa classe ainda não cria uma banda concreta. Ela apenas define que objetos do tipo `Banda` terão esses campos.

### Objeto

Um objeto nasce quando a classe é instanciada.

```java
Banda banda = new Banda();
```

Nesse exemplo, `new Banda()` cria um objeto do tipo `Banda`.

A variável `banda` não é o objeto. Ela é uma referência para o objeto.

### Campo

Campos representam o estado interno do objeto.

```java
private String nome;
private String paisOrigem;
private int anoFormacao;
```

Cada objeto possui sua própria cópia dos campos de instância.

Isso significa que dois objetos do tipo `Banda` podem ter os mesmos campos, mas valores diferentes.

### Método

Métodos representam comportamentos.

```java
public String descricao() {
    return nome + " é uma banda formada em " + anoFormacao + ".";
}
```

Quando chamamos um método, estamos enviando uma mensagem para o objeto executar determinado comportamento.

Em Java, essa “mensagem” aparece como uma chamada de método:

```java
banda.descricao();
```

A variável `banda` referencia o objeto que receberá a chamada. O método `descricao()` é o comportamento solicitado.

### Comunicação entre objetos

Quando se diz que objetos “se comunicam por mensagens”, isso não significa necessariamente mensagem de rede, fila, Kafka, HTTP ou evento assíncrono.

No contexto básico de orientação a objetos, significa que um objeto interage com outro chamando seus métodos.

Exemplo:

```java
musica.descricaoCompleta();
```

Nesse caso, quem está usando `musica` pede ao objeto `musica` para executar o comportamento `descricaoCompleta`.

A comunicação entre objetos costuma envolver:

* um objeto chamando método de outro;
* passagem de argumentos;
* retorno de valores;
* alteração controlada de estado;
* delegação de responsabilidade.

Exemplo conceitual:

```java
catalogo.adicionarMusica(musica);
```

Aqui, um objeto `catalogo` recebe uma mensagem: “adicione esta música”.

Ele pode validar a música, armazená-la internamente, rejeitar duplicidade ou delegar a operação para outro objeto.

Essa ideia é fundamental: em orientação a objetos, não pensamos apenas em dados. Pensamos em objetos que colaboram entre si para cumprir uma regra do sistema.

## Exemplo prático

A seguir, vamos modelar parte do **meucatalogomusical.com** usando Java 25.

O exemplo usa classes tradicionais, campos privados, construtores e métodos. Isso é mais próximo do que se espera em aplicações backend reais, especialmente quando o objeto possui regras de negócio.

### Classe `Banda`

```java
import java.util.Objects;

public class Banda {

    private String nome;
    private String paisOrigem;
    private int anoFormacao;

    public Banda(String nome, String paisOrigem, int anoFormacao) {
        this.nome = Objects.requireNonNull(nome, "nome não pode ser nulo");
        this.paisOrigem = Objects.requireNonNull(paisOrigem, "paisOrigem não pode ser nulo");
        this.anoFormacao = anoFormacao;
    }

    public String nome() {
        return nome;
    }

    public String paisOrigem() {
        return paisOrigem;
    }

    public int anoFormacao() {
        return anoFormacao;
    }

    public String descricao() {
        return "%s, banda de %s formada em %d"
                .formatted(nome, paisOrigem, anoFormacao);
    }
}
```

### Classe `Musica`

```java
import java.time.Duration;
import java.util.Objects;

public class Musica {

    private String titulo;
    private Duration duracao;
    private Banda banda;

    public Musica(String titulo, Duration duracao, Banda banda) {
        this.titulo = Objects.requireNonNull(titulo, "titulo não pode ser nulo");
        this.duracao = Objects.requireNonNull(duracao, "duracao não pode ser nula");
        this.banda = Objects.requireNonNull(banda, "banda não pode ser nula");
    }

    public String titulo() {
        return titulo;
    }

    public Duration duracao() {
        return duracao;
    }

    public Banda banda() {
        return banda;
    }

    public String descricaoCompleta() {
        return "%s - %s (%d:%02d)"
                .formatted(
                        banda.nome(),
                        titulo,
                        duracao.toMinutes(),
                        duracao.toSecondsPart()
                );
    }
}
```

### Classe `CatalogoMusical`

```java
import java.util.ArrayList;
import java.util.List;
import java.util.Objects;

public class CatalogoMusical {

    private List<Musica> musicas = new ArrayList<>();

    public void adicionar(Musica musica) {
        Objects.requireNonNull(musica, "musica não pode ser nula");
        musicas.add(musica);
    }

    public void listar() {
        for (Musica musica : musicas) {
            System.out.println(musica.descricaoCompleta());
        }
    }
}
```

### Classe principal

```java
import java.time.Duration;

public class Principal {

    public static void main(String[] args) {
        Banda metallica = new Banda("Metallica", "Estados Unidos", 1981);
        Banda ironMaiden = new Banda("Iron Maiden", "Reino Unido", 1975);

        Musica enterSandman = new Musica(
                "Enter Sandman",
                Duration.ofMinutes(5).plusSeconds(31),
                metallica
        );

        Musica theTrooper = new Musica(
                "The Trooper",
                Duration.ofMinutes(4).plusSeconds(12),
                ironMaiden
        );

        CatalogoMusical catalogo = new CatalogoMusical();

        catalogo.adicionar(enterSandman);
        catalogo.adicionar(theTrooper);

        catalogo.listar();
    }
}
```

Saída esperada:

```text
Metallica - Enter Sandman (5:31)
Iron Maiden - The Trooper (4:12)
```

## Analisando o código

### A classe `Banda`

A classe `Banda` representa um conceito do domínio.

```java
public class Banda {
```

Ela define um tipo próprio da aplicação.

Os campos:

```java
private String nome;
private String paisOrigem;
private int anoFormacao;
```

representam o estado interno de cada objeto `Banda`.

Como são campos de instância, cada objeto possui seus próprios valores.

Exemplo:

```java
Banda metallica = new Banda("Metallica", "Estados Unidos", 1981);
Banda ironMaiden = new Banda("Iron Maiden", "Reino Unido", 1975);
```

`metallica` e `ironMaiden` são duas variáveis diferentes, referenciando dois objetos diferentes.

Ambos são do tipo `Banda`, mas possuem estados diferentes.

### O papel do construtor

```java
public Banda(String nome, String paisOrigem, int anoFormacao) {
    this.nome = Objects.requireNonNull(nome, "nome não pode ser nulo");
    this.paisOrigem = Objects.requireNonNull(paisOrigem, "paisOrigem não pode ser nulo");
    this.anoFormacao = anoFormacao;
}
```

O construtor inicializa o objeto em um estado válido.

Sem construtor, seria possível criar uma banda sem nome, sem país e com ano de formação zero. Isso até compila, mas provavelmente não representa uma banda válida no domínio da aplicação.

O método `Objects.requireNonNull` é útil para validar referências obrigatórias. A documentação oficial informa que ele verifica se uma referência é não nula e lança `NullPointerException` caso contrário. ([Oracle Docs][4])

### Métodos de consulta

```java
public String nome() {
    return nome;
}
```

Esse método permite consultar o nome da banda sem expor diretamente o campo.

Neste artigo, os métodos foram nomeados como `nome()`, `paisOrigem()` e `anoFormacao()`, seguindo um estilo mais conciso. Também seria comum encontrar `getNome()`, `getPaisOrigem()` e `getAnoFormacao()` em projetos Java tradicionais, especialmente quando frameworks esperam convenções JavaBeans.

### A classe `Musica` e a composição

```java
private Banda banda;
```

Esse é um exemplo de composição entre objetos.

Uma música tem uma banda associada.

A classe `Musica` não copia os dados da banda. Ela mantém uma referência para um objeto `Banda`.

```java
Musica enterSandman = new Musica(
        "Enter Sandman",
        Duration.ofMinutes(5).plusSeconds(31),
        metallica
);
```

Aqui, o objeto `enterSandman` recebe uma referência para o objeto `metallica`.

Isso significa que `Musica` e `Banda` passam a colaborar.

Quando `Musica` precisa montar sua descrição completa, ela chama um método da banda:

```java
banda.nome()
```

Esse é o ponto em que a comunicação entre objetos fica mais clara.

O objeto `Musica` envia uma mensagem para o objeto `Banda`: “me informe seu nome”.

### A classe `CatalogoMusical`

```java
private List<Musica> musicas = new ArrayList<>();
```

O catálogo possui uma lista de músicas.

Essa linha também mostra inicialização de variável de instância. Quando um objeto `CatalogoMusical` é criado, ele já começa com uma lista vazia pronta para uso.

Isso faz sentido porque um catálogo sem músicas ainda pode ter uma lista vazia. Nesse caso, inicializar a lista evita `null` desnecessário e representa bem o domínio.

### Objetos colaborando

Observe esta sequência:

```java
catalogo.adicionar(enterSandman);
catalogo.listar();
```

O código principal não precisa saber como o catálogo armazena músicas internamente.

Ele apenas envia mensagens para o objeto `catalogo`:

```text
adicione esta música
liste as músicas cadastradas
```

Dentro de `listar()`, o catálogo chama outro objeto:

```java
musica.descricaoCompleta()
```

Dentro de `descricaoCompleta()`, a música chama outro objeto:

```java
banda.nome()
```

Existe uma colaboração entre objetos:

```text
Principal -> CatalogoMusical -> Musica -> Banda
```

Isso é orientação a objetos na prática: objetos pequenos, com responsabilidades específicas, colaborando para executar um caso de uso.

## O que acontece por baixo dos panos

### `new` cria um novo objeto

Quando executamos:

```java
Banda metallica = new Banda("Metallica", "Estados Unidos", 1981);
```

o Java cria uma nova instância da classe `Banda`.

A especificação da linguagem descreve que, ao criar uma nova instância, espaço de memória é alocado para o objeto com espaço para todas as variáveis de instância declaradas na classe e nas superclasses. ([Oracle Docs][5])

A avaliação de uma expressão `new` também inicializa os campos com valores padrão antes da execução do construtor. A JLS especifica que o novo objeto contém novas instâncias dos campos e que cada campo é inicializado com seu valor padrão. ([Oracle Docs][3])

Depois disso, o construtor é executado para colocar o objeto no estado desejado.

### Valores padrão

Campos de instância recebem valores padrão quando o objeto é criado.

Exemplo:

```java
public class Exemplo {
    private String texto;
    private int numero;
    private boolean ativo;
    private Banda banda;
}
```

Se um objeto `Exemplo` for criado sem atribuições explícitas, os campos começam assim:

| Tipo do campo        | Valor padrão |
| -------------------- | ------------ |
| `String`             | `null`       |
| `int`                | `0`          |
| `boolean`            | `false`      |
| `Banda`              | `null`       |
| referências em geral | `null`       |

A JLS define que variáveis de classe, variáveis de instância e componentes de array são inicializados com valores padrão quando criados, incluindo zero para tipos numéricos, `false` para `boolean` e `null` para tipos de referência. ([Oracle Docs][1])

Isso explica por que um campo `int` não começa como `null`: `int` é tipo primitivo, não tipo de referência.

Já `String`, `Banda`, `Musica`, `List<Musica>` e outros tipos baseados em classes são tipos de referência. Portanto, se não forem inicializados, começam como `null`.

### `null` não é objeto

`null` representa ausência de referência.

Quando temos:

```java
private Banda banda;
```

e nenhum valor foi atribuído, o campo `banda` não aponta para objeto algum.

Por isso, este código pode falhar:

```java
System.out.println(musica.banda().nome());
```

Se `banda()` retornar `null`, a chamada `.nome()` tentará acessar um método em algo que não referencia objeto nenhum.

A documentação da classe `NullPointerException` informa que essa exceção é lançada quando a aplicação tenta usar `null` em um caso em que um objeto é necessário, como chamar método de um objeto nulo ou acessar campo de um objeto nulo. ([Oracle Docs][6])

### Variáveis de referência não guardam o objeto

Este ponto é essencial.

Considere:

```java
Banda banda = new Banda("Black Sabbath", "Reino Unido", 1968);
```

A variável `banda` não contém o objeto inteiro dentro dela.

Ela contém uma referência para o objeto.

Por isso, quando passamos um objeto para outro, não estamos copiando automaticamente o objeto inteiro.

Exemplo:

```java
Banda blackSabbath = new Banda("Black Sabbath", "Reino Unido", 1968);

Musica paranoid = new Musica(
        "Paranoid",
        Duration.ofMinutes(2).plusSeconds(48),
        blackSabbath
);
```

O objeto `paranoid` guarda uma referência para o mesmo objeto `blackSabbath`.

Se o objeto `Banda` fosse mutável e seu nome fosse alterado, quem possui referência para ele passaria a enxergar o novo estado.

Exemplo didático:

```java
public class BandaMutavel {

    private String nome;

    public BandaMutavel(String nome) {
        this.nome = nome;
    }

    public String nome() {
        return nome;
    }

    public void renomear(String novoNome) {
        this.nome = novoNome;
    }
}
```

Uso:

```java
BandaMutavel banda = new BandaMutavel("Nome antigo");

BandaMutavel outraReferencia = banda;

outraReferencia.renomear("Nome novo");

System.out.println(banda.nome());
```

Saída:

```text
Nome novo
```

Isso acontece porque `banda` e `outraReferencia` referenciam o mesmo objeto.

Não existem dois objetos. Existem duas variáveis apontando para a mesma instância.

### `==` em objetos compara referências

Quando usamos `==` entre objetos, comparamos se as duas variáveis apontam para o mesmo objeto.

```java
Banda a = new Banda("Rush", "Canadá", 1968);
Banda b = new Banda("Rush", "Canadá", 1968);

System.out.println(a == b);
```

Saída:

```text
false
```

Apesar dos dados serem iguais, são dois objetos diferentes.

A documentação de `Object.equals` informa que a implementação padrão de `equals` em `Object` retorna `true` se, e somente se, as referências apontam para o mesmo objeto. ([Oracle Docs][7])

Para comparar igualdade lógica, normalmente sobrescrevemos `equals` e `hashCode`, ou usamos `record` quando o objetivo for representar dados de forma transparente.

## Comparação com versões anteriores do Java

Classes, objetos, campos, métodos, construtores, `new`, referências e valores padrão fazem parte da base da linguagem Java desde o início.

O que mudou ao longo das versões foi a quantidade de recursos disponíveis para escrever modelos mais expressivos e menos verbosos.

### Java tradicional: classes comuns

Por muitos anos, a forma principal de modelar objetos foi a classe comum:

```java
public class Banda {
    private final String nome;

    public Banda(String nome) {
        this.nome = nome;
    }

    public String nome() {
        return nome;
    }
}
```

Essa abordagem continua sendo a mais flexível quando o objeto tem comportamento, invariantes, encapsulamento, regras de negócio e estado interno que não deve ser exposto diretamente.

### Java 10: `var` em variáveis locais

O Java 10 introduziu inferência de tipo para variáveis locais pela JEP 286. O recurso permite omitir o tipo explícito em variáveis locais com inicializador, mas não está disponível para campos, parâmetros de método ou tipos de retorno. ([OpenJDK][8])

Exemplo:

```java
var banda = new Banda("Deep Purple", "Reino Unido", 1968);
```

O compilador infere que `banda` é do tipo `Banda`.

Apesar disso, neste artigo os exemplos evitam `var` de propósito. Para estudar classes e objetos, escrever o tipo explicitamente ajuda a visualizar melhor a relação entre tipo, variável e objeto.

### Java 16: `record`

O Java 16 finalizou os `records` pela JEP 395. Records são uma forma especial de classe para representar agregados simples de dados com menos cerimônia. A própria JEP descreve records como uma nova espécie de classe para modelar dados como dados. ([OpenJDK][9])

Exemplo:

```java
public record BandaResumo(String nome, String paisOrigem, int anoFormacao) {
}
```

Um `record` gera automaticamente:

* campos privados e finais;
* construtor canônico;
* métodos de acesso;
* `equals`;
* `hashCode`;
* `toString`.

Mas `record` não substitui classe comum em todos os casos.

Use `record` quando o objetivo for representar dados imutáveis e transparentes.

Use classe comum quando precisar de mais controle sobre:

* identidade;
* ciclo de vida;
* encapsulamento;
* invariantes complexas;
* mutabilidade controlada;
* regras de negócio;
* integração com frameworks que exigem construtores ou proxies específicos.

## Boas práticas

### Modele classes a partir do problema real

Não crie classes apenas porque um substantivo apareceu no sistema.

Crie classes quando houver um conceito relevante para o domínio.

No **meucatalogomusical.com**, `Banda`, `Musica`, `Album` e `CatalogoMusical` fazem sentido.

Mas talvez `Texto`, `Dado`, `Informacao` ou `ObjetoGenerico` sejam nomes vagos demais.

### Use nomes específicos e claros

Prefira:

```java
private Banda banda;
```

em vez de:

```java
private Banda objeto;
```

Prefira:

```java
private Banda artistaPrincipal;
```

quando o papel da banda no relacionamento for mais específico.

O tipo informa o que o objeto é. O nome da variável informa o papel que ele exerce naquele contexto.

### Evite repetir o nome da classe nos campos

Dentro da classe `Banda`, isto é redundante:

```java
private String nomeBanda;
private String paisOrigemBanda;
```

Melhor:

```java
private String nome;
private String paisOrigem;
```

O contexto da classe já informa que os campos pertencem a uma banda.

### Inicialize objetos em estado válido

Prefira construir objetos já válidos:

```java
Banda banda = new Banda("Megadeth", "Estados Unidos", 1983);
```

em vez de criar objeto vazio e preencher depois:

```java
Banda banda = new Banda();
banda.nome = "Megadeth";
banda.paisOrigem = "Estados Unidos";
banda.anoFormacao = 1983;
```

A segunda abordagem facilita objetos incompletos em partes do sistema.

### Use composição para representar relacionamentos

Se uma música pertence a uma banda, represente isso com uma referência:

```java
private Banda banda;
```

Não espalhe os dados da banda dentro da música:

```java
private String nomeBanda;
private String paisBanda;
private int anoFormacaoBanda;
```

A segunda abordagem duplica dados, dificulta manutenção e enfraquece o modelo.

### Não inicialize tudo só para fugir de `NullPointerException`

Nem todo campo de referência precisa nascer com `new`.

Isto pode ser correto:

```java
private List<Musica> musicas = new ArrayList<>();
```

Porque uma lista vazia representa bem um catálogo sem músicas.

Mas isto pode ser estranho:

```java
private Banda banda = new Banda("Desconhecida", "Desconhecido", 0);
```

Se toda música precisa de uma banda real, criar uma banda falsa apenas para evitar `null` esconde erro de modelagem.

`null` não deve ser usado sem critério, mas também não deve ser escondido com objetos artificiais.

### Prefira objetos pequenos e coesos

Uma classe deve ter uma responsabilidade clara.

`Banda` não deve salvar a si mesma no banco de dados.

`Musica` não deve enviar e-mail.

`CatalogoMusical` não deve saber detalhes de HTTP.

Em sistemas backend, essa separação ajuda a manter domínio, aplicação e infraestrutura com responsabilidades mais limpas.

## Erros comuns

### Confundir classe com objeto

Classe:

```java
public class Banda {
}
```

Objeto:

```java
Banda banda = new Banda();
```

A classe é a definição. O objeto é a instância criada em tempo de execução.

### Achar que a variável é o objeto

```java
Banda banda = new Banda("Queen", "Reino Unido", 1970);
```

`banda` é uma variável de referência.

O objeto foi criado por `new Banda(...)`.

Essa diferença é essencial para entender composição, passagem de parâmetros, igualdade, mutabilidade e efeitos colaterais.

### Acessar campo ou método de referência nula

```java
Musica musica = null;

System.out.println(musica.titulo());
```

Esse código lança `NullPointerException`.

O problema não está no método `titulo()`. O problema é que `musica` não referencia objeto algum.

### Criar objetos sem estado válido

```java
Banda banda = new Banda();
```

Se a aplicação permite criar uma banda sem nome, sem país e sem ano de formação, talvez o objeto esteja sendo criado em um estado inválido.

Construtores ajudam a reduzir esse problema.

### Usar composição como simples agrupamento de campos

Composição não é apenas colocar uma classe dentro da outra.

Composição deve representar uma relação real no domínio.

Bom exemplo:

```java
private Banda banda;
```

em `Musica`, porque uma música tem uma relação com uma banda.

Exemplo duvidoso:

```java
private Relatorio relatorio;
```

dentro de `Banda`, se relatório for apenas uma saída gerada pela aplicação e não parte real do conceito de banda.

### Comparar objetos com `==` quando a intenção é igualdade lógica

```java
Banda a = new Banda("Rush", "Canadá", 1968);
Banda b = new Banda("Rush", "Canadá", 1968);

System.out.println(a == b);
```

O resultado é `false`, porque são referências para objetos diferentes.

Se a intenção for comparar conteúdo, implemente `equals` e `hashCode` ou use `record` quando fizer sentido.

## Decisões de engenharia

### Quando usar classes comuns

Use classes comuns quando o objeto precisa de:

* comportamento relevante;
* regras de negócio;
* validações;
* encapsulamento;
* mutabilidade controlada;
* identidade;
* composição com outros objetos;
* evolução interna sem expor todos os detalhes.

Exemplo:

```java
public class CatalogoMusical {
    private List<Musica> musicas = new ArrayList<>();

    public void adicionar(Musica musica) {
        Objects.requireNonNull(musica);
        musicas.add(musica);
    }
}
```

Aqui faz sentido usar classe comum porque existe comportamento: adicionar e listar músicas.

### Quando evitar criar uma classe

Evite criar classes quando elas não adicionam significado.

Exemplo ruim:

```java
public class NomeDaBanda {
    private String valor;
}
```

Essa classe pode fazer sentido se houver validação, normalização ou regra específica para nome de banda.

Mas, se for apenas um embrulho sem comportamento, talvez `String` seja suficiente naquele momento.

### Quando usar `record`

Use `record` para transportar dados imutáveis e transparentes.

Exemplo:

```java
public record BandaResumo(String nome, String paisOrigem, int anoFormacao) {
}
```

Isso pode ser útil para DTOs, projeções de consulta, respostas simples ou valores imutáveis.

Mas evite `record` quando precisar esconder representação interna, controlar mutabilidade ou construir um modelo rico de domínio.

### Quando usar composição

Use composição quando um objeto realmente possui ou se relaciona com outro objeto.

Exemplo:

```java
public class Musica {
    private Banda banda;
}
```

A música não precisa copiar os dados da banda. Ela se relaciona com uma banda.

Isso melhora:

* coesão;
* reutilização;
* clareza do modelo;
* redução de duplicidade;
* manutenção futura.

### Impacto em legibilidade

Um bom modelo orientado a objetos deixa o código mais próximo da linguagem do negócio.

Compare:

```java
catalogo.adicionar(musica);
```

com:

```java
lista.add(obj);
```

A primeira versão comunica melhor a intenção.

### Impacto em manutenção

Classes bem modeladas concentram regras no lugar certo.

Se a regra de descrição de música mudar, alteramos `Musica`.

Se a regra de cadastro no catálogo mudar, alteramos `CatalogoMusical`.

Isso evita espalhar lógica pelo sistema.

### Impacto em performance

Criar objetos tem custo, mas em sistemas backend comuns o maior problema raramente é a existência de objetos bem modelados.

Problemas de performance costumam vir mais de:

* consultas ineficientes;
* excesso de I/O;
* serialização pesada;
* loops desnecessários;
* estruturas de dados inadequadas;
* criação massiva de objetos em trechos críticos;
* uso incorreto de cache;
* má modelagem de transações.

Ainda assim, objetos devem ser criados com consciência. Em caminhos muito críticos, como processamento de milhões de registros, parsing intensivo ou aplicações de baixa latência, alocação excessiva pode pressionar o garbage collector.

A decisão não deve ser “evitar objetos”. Deve ser “modelar bem e medir quando houver suspeita real de gargalo”.

## Onde isso aparece em sistemas reais

### APIs REST

Em uma API REST, classes aparecem em várias camadas:

```text
Controller -> DTO -> Use Case -> Domain Object -> Repository -> Entity
```

Exemplo:

```java
public record CadastroMusicaRequest(
        String titulo,
        long duracaoEmSegundos,
        String nomeBanda
) {
}
```

Esse `record` pode representar os dados recebidos na requisição.

Já uma classe comum pode representar a regra de domínio:

```java
public class Musica {
    private String titulo;
    private Duration duracao;
    private Banda banda;
}
```

### Persistência

Em aplicações com banco de dados, objetos podem representar entidades persistidas.

Exemplo conceitual:

```java
public class BandaEntity {
    private Long id;
    private String nome;
    private String paisOrigem;
}
```

Mas é importante não confundir objeto de domínio com entidade de persistência. Em arquiteturas mais cuidadosas, como Clean Architecture, pode ser interessante separar o modelo de domínio do modelo usado pelo ORM.

### Mensageria

Em sistemas com Kafka, RabbitMQ ou SQS, objetos frequentemente representam mensagens.

Exemplo:

```java
public record MusicaCadastradaEvent(
        String titulo,
        String nomeBanda
) {
}
```

Esse objeto não precisa ter comportamento complexo. Ele pode ser apenas um registro imutável de um fato ocorrido.

### Testes automatizados

Testes também dependem de boa modelagem.

Criar objetos claros torna os testes mais legíveis:

```java
Banda banda = new Banda("Dream Theater", "Estados Unidos", 1985);

Musica musica = new Musica(
        "Pull Me Under",
        Duration.ofMinutes(8).plusSeconds(14),
        banda
);
```

A intenção do teste fica mais clara quando os objetos representam conceitos reais.

## Perguntas de revisão

### 1 - Qual é a diferença entre classe e objeto?

Resposta: classe é a definição de um tipo. Objeto é uma instância concreta criada a partir dessa classe durante a execução do programa. `Banda` é uma classe; `new Banda("Metallica", "Estados Unidos", 1981)` cria um objeto.

### 2 - O que significa dizer que uma variável referencia um objeto?

Resposta: significa que a variável não guarda o objeto inteiro. Ela guarda uma referência que permite acessar o objeto na memória. Por isso, duas variáveis podem apontar para o mesmo objeto.

### 3 - O que o operador `new` faz?

Resposta: `new` cria uma nova instância de uma classe, aloca espaço para seus campos, inicializa esses campos com valores padrão e executa o construtor correspondente.

### 4 - Todo objeto criado a partir da mesma classe tem os mesmos valores?

Resposta: não. Objetos da mesma classe possuem a mesma estrutura, mas cada instância pode ter valores próprios. Duas bandas podem ter os campos `nome`, `paisOrigem` e `anoFormacao`, mas com valores diferentes.

### 5 - O que são campos de instância?

Resposta: são variáveis declaradas na classe que pertencem a cada objeto criado. Cada instância possui sua própria cópia desses campos.

### 6 - O que são métodos?

Resposta: métodos representam comportamentos que um objeto pode executar. Em Java, objetos colaboram chamando métodos uns dos outros.

### 7 - O que significa dizer que objetos se comunicam por mensagens?

Resposta: significa que um objeto solicita comportamento de outro por meio de chamadas de método. Por exemplo, `catalogo.adicionar(musica)` é uma mensagem enviada ao objeto `catalogo` para adicionar uma música.

### 8 - O que é composição de objetos?

Resposta: composição acontece quando um objeto possui uma referência para outro objeto como parte de sua estrutura. Por exemplo, `Musica` pode ter um campo `Banda`, indicando que uma música está associada a uma banda.

### 9 - Composição copia o objeto referenciado?

Resposta: não. Quando um objeto recebe outro como campo, normalmente ele guarda uma referência para o objeto recebido. Ele não cria automaticamente uma cópia profunda.

### 10 - O que é `null`?

Resposta: `null` representa ausência de referência. Uma variável de referência com valor `null` não aponta para nenhum objeto.

### 11 - Por que acontece `NullPointerException`?

Resposta: acontece quando o programa tenta usar `null` em um contexto que exige um objeto, como chamar método ou acessar campo de uma referência nula.

### 12 - Quais são os valores padrão de campos de instância?

Resposta: campos numéricos primitivos começam com zero, `boolean` começa com `false`, `char` começa com `'\u0000'` e campos de referência começam com `null`.

### 13 - Por que `int` não pode ser `null`?

Resposta: porque `int` é tipo primitivo. Tipos primitivos armazenam valores diretamente e não referências. Apenas tipos de referência podem receber `null`.

### 14 - Qual é o risco de criar objetos vazios e preencher depois?

Resposta: o objeto pode circular pelo sistema em estado inválido ou incompleto. Construtores ajudam a criar objetos já consistentes.

### 15 - Quando faz sentido inicializar um campo de instância diretamente?

Resposta: quando o valor inicial representa corretamente o estado natural do objeto. Por exemplo, uma lista vazia em um catálogo faz sentido. Já criar uma banda fictícia apenas para evitar `null` pode esconder erro de modelagem.

### 16 - Qual é a diferença entre `==` e `equals` em objetos?

Resposta: `==` compara se duas referências apontam para o mesmo objeto. `equals`, quando sobrescrito corretamente, pode comparar igualdade lógica entre objetos.

### 17 - Quando usar classe comum em vez de `record`?

Resposta: use classe comum quando precisar de comportamento, encapsulamento, mutabilidade controlada, invariantes complexas ou identidade. Use `record` quando quiser representar dados imutáveis e transparentes.

### 18 - Por que evitar campos públicos?

Resposta: campos públicos expõem diretamente o estado interno do objeto, dificultando validação, manutenção e evolução da classe. Campos privados com métodos controlados preservam melhor o encapsulamento.

### 19 - O que um desenvolvedor backend deve observar ao modelar objetos?

Resposta: deve observar se a classe representa um conceito real do domínio, se possui responsabilidade clara, se nasce em estado válido, se evita duplicidade de dados e se suas relações com outros objetos fazem sentido.

### 20 - Objetos bem modelados melhoram performance?

Resposta: indiretamente, podem melhorar manutenção e reduzir erros, mas não garantem performance por si só. Em geral, a modelagem deve priorizar clareza e consistência. Otimizações de alocação devem ser feitas com medição real em trechos críticos.

## Resumo final

Classes e objetos são a base da programação orientada a objetos em Java.

Uma classe define um tipo. Um objeto é uma instância concreta desse tipo. Campos representam estado. Métodos representam comportamento. Construtores ajudam a inicializar objetos. Variáveis de referência apontam para objetos, mas não armazenam os objetos diretamente.

Composição permite que objetos se relacionem entre si. No **meucatalogomusical.com**, uma `Musica` pode ter uma `Banda`, e um `CatalogoMusical` pode ter várias músicas. Essa colaboração entre objetos acontece por chamadas de métodos, que são as mensagens da orientação a objetos em Java.

Entender bem classes, objetos, referências, `new`, valores padrão e `null` é essencial para avançar em temas mais profundos, como encapsulamento, imutabilidade, herança, polimorfismo, igualdade, coleções, persistência e modelagem de domínio.

## Referências

* Java Language Specification, Java SE 25, Chapter 4 — Types, Values, and Variables. ([Oracle Docs][1])
* Java Language Specification, Java SE 25, §4.12.5 — Initial Values of Variables. ([Oracle Docs][1])
* Java Language Specification, Java SE 25, Chapter 8 — Classes. ([Oracle Docs][2])
* Java Language Specification, Java SE 25, §12.5 — Creation of New Class Instances. ([Oracle Docs][5])
* Java Language Specification, Java SE 25, §15.9 — Class Instance Creation Expressions. ([Oracle Docs][3])
* Java SE 25 API Documentation — `java.lang.Object`. ([Oracle Docs][7])
* Java SE 25 API Documentation — `java.lang.NullPointerException`. ([Oracle Docs][6])
* Java SE 25 API Documentation — `java.util.Objects`. ([Oracle Docs][4])
* OpenJDK — JEP 286: Local-Variable Type Inference. ([OpenJDK][8])
* OpenJDK — JEP 395: Records. ([OpenJDK][9])
* Joshua Bloch — *Effective Java*.
* Brian Goetz et al. — *Java Concurrency in Practice*.

[1]: https://docs.oracle.com/javase/specs/jls/se25/html/jls-4.html "Chapter 4. Types, Values, and Variables"
[2]: https://docs.oracle.com/javase/specs/jls/se25/html/jls-8.html "Chapter 8. Classes"
[3]: https://docs.oracle.com/javase/specs/jls/se25/html/jls-15.html "Chapter 15. Expressions"
[4]: https://docs.oracle.com/en/java/javase/25/docs/api//java.base/java/util/Objects.html "Objects (Java SE 25 & JDK 25)"
[5]: https://docs.oracle.com/javase/specs/jls/se25/html/jls-12.html "Chapter 12. Execution"
[6]: https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/NullPointerException.html "NullPointerException (Java SE 25 & JDK 25)"
[7]: https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/Object.html "Object (Java SE 25 & JDK 25)"
[8]: https://openjdk.org/jeps/286 "JEP 286: Local-Variable Type Inference"
[9]: https://openjdk.org/jeps/395 "JEP 395: Records"
