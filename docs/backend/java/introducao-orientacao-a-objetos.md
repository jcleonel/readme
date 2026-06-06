# Introdução à Programação Orientada a Objetos em Java

À medida que um sistema cresce, ele deixa de ser apenas uma sequência de comandos e passa a representar regras de negócio, estados, validações, fluxos, integrações e responsabilidades diferentes.

Em uma aplicação como `meucatalogomusical.com`, não basta ter variáveis soltas como `nomeBanda`, `nomeMusica`, `duracao`, `genero` e vários procedimentos manipulando tudo livremente. Com o tempo, surgem perguntas importantes:

* Quem garante que uma música sempre terá título?
* Quem decide se uma banda pode ser cadastrada?
* Onde fica a regra que impede uma duração inválida?
* Quem altera os dados do catálogo?
* Como evitar que qualquer parte do sistema mude informações de forma descontrolada?

A Programação Orientada a Objetos resolve esse problema organizando o sistema em unidades com **estado**, **comportamento** e **responsabilidade**. Em vez de pensar apenas em comandos executados em sequência, passamos a modelar partes do domínio como objetos que colaboram entre si.

No Java, esse modelo é especialmente importante porque a linguagem é fortemente baseada em classes, objetos, interfaces e hierarquias de tipos. A própria especificação da linguagem define classes, interfaces, arrays e objetos como parte central do sistema de tipos de referência. ([Oracle Docs][1])

## Antes de entrar no código: uma ideia simples

Um paradigma de programação é uma forma de pensar, organizar e expressar uma solução em código.

Alguns paradigmas comuns são:

| Paradigma           | Ideia central                                             | Exemplo de pensamento                                                           |
| ------------------- | --------------------------------------------------------- | ------------------------------------------------------------------------------- |
| Imperativo          | Dizer passo a passo o que o programa deve fazer           | “Faça isso, depois aquilo, depois atualize essa variável.”                      |
| Procedural          | Organizar o programa em procedimentos, funções ou rotinas | “Crie uma função para cadastrar música e outra para listar bandas.”             |
| Orientado a Objetos | Organizar o sistema em objetos com responsabilidades      | “Uma banda conhece suas músicas; um catálogo gerencia bandas.”                  |
| Funcional           | Descrever transformações de dados com funções             | “Filtre as músicas, transforme os dados e colete o resultado.”                  |
| Reativo             | Trabalhar com fluxos assíncronos de dados e eventos       | “Quando uma música for cadastrada, publique um evento para outros componentes.” |

A Programação Orientada a Objetos pertence ao grande grupo da programação imperativa, mas muda o foco. Em vez de pensar somente em uma lista de comandos, o programador pensa em **modelos**, **responsabilidades** e **colaboração entre partes do sistema**.

Java também permite combinar estilos. Desde o Java 8, por exemplo, a linguagem passou a ter suporte importante para expressões lambda e method references, aproximando alguns usos da programação funcional ao código Java cotidiano. ([OpenJDK][2])

## Definição técnica

Programação Orientada a Objetos é um paradigma que organiza o software em torno de objetos. Um objeto representa uma unidade do sistema que pode possuir:

* **estado**, representado por dados internos;
* **comportamento**, representado por operações;
* **identidade**, quando faz sentido distinguir uma instância de outra;
* **responsabilidade**, ou seja, uma razão clara para existir dentro do modelo.

Em Java, objetos normalmente são criados a partir de classes. Uma classe descreve a estrutura e o comportamento que seus objetos podem ter. A documentação da plataforma Java também destaca `java.lang.Object` como a raiz da hierarquia de classes, o que reforça a centralidade desse modelo na linguagem. ([Oracle Docs][3])

Mas é importante não simplificar demais: Java não é uma linguagem “100% orientada a objetos” no sentido absoluto. A linguagem possui tipos primitivos, membros `static`, métodos utilitários e recursos que permitem escrever código em estilo procedural. A própria especificação diferencia tipos primitivos de tipos de referência, e os objetos pertencem ao universo dos tipos de referência. ([Oracle Docs][4])

Portanto, uma definição mais madura seria:

> Programação Orientada a Objetos, em Java, é uma forma de estruturar sistemas usando tipos que combinam dados e comportamento, criando limites explícitos de responsabilidade e permitindo que diferentes partes do programa colaborem por meio de chamadas de métodos.

## Como funciona em Java

Em Java, a orientação a objetos aparece principalmente por meio de:

* classes;
* objetos;
* métodos;
* atributos;
* construtores;
* interfaces;
* herança;
* composição;
* encapsulamento;
* polimorfismo.

A ideia principal é: uma classe define um tipo; um objeto é uma instância concreta desse tipo; métodos representam comportamentos; atributos representam dados internos; e as partes do sistema interagem chamando métodos umas das outras.

Em `meucatalogomusical.com`, poderíamos pensar assim:

| Conceito do domínio | Possível representação em Java | Responsabilidade                     |
| ------------------- | ------------------------------ | ------------------------------------ |
| Banda               | `Banda`                        | Representar uma banda e suas músicas |
| Música              | `Musica`                       | Representar os dados de uma música   |
| Catálogo            | `CatalogoMusical`              | Gerenciar bandas cadastradas         |
| Busca               | `ServicoDeBusca`               | Pesquisar músicas e bandas           |
| API                 | `CatalogoController`           | Receber requisições HTTP             |

O ponto mais importante não é “criar classes porque Java exige classes”. O ponto é criar modelos que ajudem o sistema a ficar mais claro, seguro e fácil de evoluir.

### Comparação com versões anteriores do Java

A base da orientação a objetos existe desde as primeiras versões da linguagem. Classes, objetos, métodos, herança e interfaces fazem parte da identidade do Java há muito tempo.

O que mudou ao longo das versões foi a forma como Java permite combinar POO com outros estilos:

| Versão  | Recurso                                           | Impacto no desenho orientado a objetos                                                                                     |
| ------- | ------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------- |
| Java 8  | Lambdas e method references, associados à JEP 126 | Permitiu expressar comportamentos de forma mais concisa, especialmente com Collections e Streams. ([OpenJDK][2])           |
| Java 10 | `var`, associado à JEP 286                        | Reduziu verbosidade em variáveis locais quando o tipo pode ser inferido com clareza. ([OpenJDK][5])                        |
| Java 16 | Records, associados à JEP 395                     | Criou uma forma mais concisa de representar agregados de dados imutáveis ou quase imutáveis por convenção. ([OpenJDK][6])  |
| Java 17 | Sealed classes, associadas à JEP 409              | Permitiu restringir quais classes podem estender ou implementar um tipo. ([OpenJDK][7])                                    |
| Java 25 | LTS mais recente da plataforma Java SE            | Base moderna para projetos que querem suporte de longo prazo e recursos atuais da linguagem e da plataforma. ([Oracle][8]) |

Isso não muda o fundamento da POO, mas muda as ferramentas disponíveis para modelar melhor o domínio.

## Exemplo prático

O exemplo abaixo mostra uma modelagem simples para o `meucatalogomusical.com`.

A ideia não é se aprofundar ainda em todos os conceitos internos de classe, objeto, encapsulamento ou composição. O objetivo é perceber a mudança de mentalidade: em vez de variáveis soltas e funções manipulando tudo, criamos tipos com responsabilidades claras.

```java
import java.util.ArrayList;
import java.util.List;

public class CatalogoMusicalDemo {

    public static void main(String[] args) {
        var catalogo = new CatalogoMusical();

        var brothersOfMetal = new Banda("Brothers of Metal");
        brothersOfMetal.adicionarMusica(new Musica("Ride of the Valkyries", "Power Metal Viking", 276));
        brothersOfMetal.adicionarMusica(new Musica("Defenders of Valhalla", "Power Metal Viking", 265));

        var ironMaiden = new Banda("Iron Maiden");
        ironMaiden.adicionarMusica(new Musica("The Trooper", "Heavy Metal", 251));

        catalogo.cadastrar(brothersOfMetal);
        catalogo.cadastrar(ironMaiden);

        catalogo.listarBandas()
                .forEach(banda -> System.out.printf(
                        "%s: %d música(s)%n",
                        banda.nome(),
                        banda.quantidadeDeMusicas()
                ));
    }
}

record Musica(String titulo, String genero, int duracaoEmSegundos) {

    Musica {
        if (titulo == null || titulo.isBlank()) {
            throw new IllegalArgumentException("O título da música é obrigatório.");
        }

        if (genero == null || genero.isBlank()) {
            throw new IllegalArgumentException("O gênero musical é obrigatório.");
        }

        if (duracaoEmSegundos <= 0) {
            throw new IllegalArgumentException("A duração deve ser maior que zero.");
        }
    }
}

final class Banda {

    private final String nome;
    private final List<Musica> musicas = new ArrayList<>();

    Banda(String nome) {
        if (nome == null || nome.isBlank()) {
            throw new IllegalArgumentException("O nome da banda é obrigatório.");
        }

        this.nome = nome;
    }

    String nome() {
        return nome;
    }

    void adicionarMusica(Musica musica) {
        if (musica == null) {
            throw new IllegalArgumentException("A música não pode ser nula.");
        }

        musicas.add(musica);
    }

    int quantidadeDeMusicas() {
        return musicas.size();
    }

    List<Musica> musicas() {
        return List.copyOf(musicas);
    }
}

final class CatalogoMusical {

    private final List<Banda> bandas = new ArrayList<>();

    void cadastrar(Banda banda) {
        if (banda == null) {
            throw new IllegalArgumentException("A banda não pode ser nula.");
        }

        bandas.add(banda);
    }

    List<Banda> listarBandas() {
        return List.copyOf(bandas);
    }
}
```

## Analisando o código

A classe `CatalogoMusicalDemo` representa apenas o ponto de entrada do exemplo. Ela cria o catálogo, cadastra bandas e imprime um resumo.

A classe `CatalogoMusical` tem uma responsabilidade mais clara: guardar e listar bandas. Ela não deveria validar detalhes internos de uma música, calcular duração total de álbuns ou formatar resposta HTTP. Cada responsabilidade deve ficar no lugar mais adequado.

A classe `Banda` representa uma banda dentro do domínio. Ela possui um nome e uma coleção de músicas. Em vez de deixar qualquer parte do sistema manipular diretamente a lista interna, ela oferece o método `adicionarMusica`.

O `record Musica` representa uma música como dado de domínio. Records foram finalizados no Java 16 pela JEP 395 e são uma forma concisa de declarar classes voltadas a carregar dados, com membros como construtor, acessores, `equals`, `hashCode` e `toString` gerados de forma padronizada. ([OpenJDK][6])

O uso de `var` aparece em variáveis locais dentro do `main`. Esse recurso foi introduzido no Java 10 pela JEP 286 e permite que o compilador infira o tipo da variável a partir do inicializador. ([OpenJDK][5])

A expressão lambda usada em `.forEach(banda -> ...)` representa uma forma mais funcional de passar comportamento para uma operação. Esse estilo se tornou comum no Java moderno a partir do Java 8, com lambdas e method references. ([OpenJDK][2])

Mesmo com esses recursos modernos, o exemplo continua sendo orientado a objetos. O centro do desenho ainda está nos tipos `Banda`, `Musica` e `CatalogoMusical`.

## O que acontece por baixo dos panos

Quando escrevemos:

```java
var brothersOfMetal = new Banda("Brothers of Metal");
```

o código cria um novo objeto do tipo `Banda` e guarda uma referência para esse objeto na variável `brothersOfMetal`.

A variável não “é” o objeto em si. Ela referencia um objeto. Essa diferença é essencial em Java. A especificação da linguagem define que tipos de referência são implementados por objetos criados dinamicamente, e múltiplas referências podem apontar para o mesmo objeto. ([Oracle Docs][4])

Quando chamamos:

```java
brothersOfMetal.adicionarMusica(new Musica("Ride of the Valkyries", "Power Metal Viking", 276));
```

acontecem algumas coisas importantes:

1. um objeto `Musica` é criado;
2. o construtor compacto do `record` valida os dados;
3. o método `adicionarMusica` é chamado no objeto `brothersOfMetal`;
4. a música é adicionada à lista interna daquela banda.

Essa chamada de método é a forma mais comum de “mensagem” entre objetos em Java. Um objeto não precisa conhecer todos os detalhes internos do outro. Ele precisa conhecer apenas a operação pública ou acessível que pode chamar.

Outro ponto importante é que `Banda` e `CatalogoMusical` retornam cópias não modificáveis das listas usando `List.copyOf`. A intenção é proteger o estado interno. Sem isso, outro trecho do código poderia receber a lista e modificá-la diretamente, quebrando regras internas do objeto.

## Boas práticas

Modele objetos a partir de responsabilidades reais, não apenas a partir de tabelas do banco de dados. Uma tabela `bandas` pode virar uma classe `Banda`, mas isso não significa que toda classe deve ser uma cópia exata da estrutura relacional.

Evite criar objetos que só carregam dados e não protegem nenhuma regra. Em muitos sistemas, isso leva ao chamado modelo anêmico: os dados ficam nas classes, mas as regras importantes ficam espalhadas em services genéricos.

Use nomes que expressem o domínio. `Banda`, `Musica`, `CatalogoMusical` e `ServicoDeBusca` comunicam melhor a intenção do que nomes genéricos como `Manager`, `Processor`, `Handler` ou `Util`.

Proteja invariantes. Se uma música não pode existir sem título ou com duração negativa, essa validação deve estar próxima da criação da música. Isso reduz a chance de estados inválidos circularem pelo sistema.

Prefira composição antes de herança quando estiver modelando relações de negócio. Uma banda “possui músicas”; ela não “é uma música”. Herança deve representar uma relação conceitual forte, não apenas uma forma de reaproveitar código.

Use recursos modernos do Java com intenção. `record` é excelente para representar dados simples, mas nem todo objeto de domínio deve ser um record. Quando há identidade, ciclo de vida, mudança controlada de estado e regras comportamentais, uma classe comum pode ser mais adequada.

## Erros comuns

Um erro comum é acreditar que Programação Orientada a Objetos significa apenas “criar muitas classes”. Isso leva a sistemas cheios de arquivos, mas sem clareza de responsabilidade.

Outro erro é colocar tudo em uma única classe central. Por exemplo, uma classe `CatalogoService` que cadastra banda, valida música, formata JSON, acessa banco, envia evento e monta resposta HTTP tende a virar um ponto de acoplamento excessivo.

Também é comum confundir objeto com registro de banco. Um objeto pode até ser persistido, mas seu papel no código é representar comportamento e regras do domínio, não apenas linhas e colunas.

Outro problema recorrente é usar herança para reaproveitamento de código sem uma relação conceitual verdadeira. Isso cria hierarquias frágeis e difíceis de mudar.

Também existe o risco de usar POO de forma excessivamente abstrata. Interfaces, classes abstratas e padrões de projeto são ferramentas úteis, mas podem prejudicar a legibilidade quando aparecem antes de uma necessidade real.

## Decisões de engenharia

Use Programação Orientada a Objetos quando o problema possui entidades com regras, estados, comportamentos e relações importantes. Sistemas backend normalmente se beneficiam disso porque lidam com regras de negócio, validações, fluxos, integrações e persistência.

Evite forçar POO quando o problema é apenas uma transformação simples de dados. Por exemplo, filtrar uma lista de músicas por gênero pode ficar mais claro com Streams e funções do que com uma hierarquia complexa de classes.

A orientação a objetos melhora a manutenção quando as responsabilidades estão bem distribuídas. Porém, ela piora o sistema quando vira excesso de camadas, abstrações prematuras ou classes sem comportamento relevante.

Em performance, o custo de criar objetos geralmente não é o primeiro problema em aplicações backend comuns. Antes de otimizar a quantidade de objetos, normalmente faz mais sentido observar acesso a banco, chamadas HTTP, serialização, concorrência, uso de memória e algoritmos. Ainda assim, em trechos de alto volume, alocações excessivas podem impactar garbage collection e devem ser analisadas com métricas.

Alternativas e complementos:

| Abordagem                 | Quando considerar                                                                                     |
| ------------------------- | ----------------------------------------------------------------------------------------------------- |
| Procedural                | Scripts simples, automações pequenas ou fluxos lineares                                               |
| Funcional                 | Transformações de coleções, pipelines de dados e operações sem estado compartilhado                   |
| Orientação a Objetos      | Domínio com regras, estados, entidades e colaboração entre componentes                                |
| Data-oriented programming | Modelos em que a estrutura dos dados e os casos possíveis são mais importantes que identidade mutável |
| Reativo                   | Fluxos assíncronos, não bloqueantes e orientados a eventos                                            |

O ponto de maturidade não é defender um paradigma como se fosse sempre superior. O ponto é escolher o modelo que deixa o sistema mais simples, seguro e evolutivo.

## Onde isso aparece em sistemas reais

Em aplicações Java backend, POO aparece em praticamente todos os níveis.

Na camada de domínio, objetos representam conceitos de negócio como `Banda`, `Musica`, `Album`, `Usuario`, `Playlist` e `Assinatura`.

Na camada de aplicação, objetos representam casos de uso, comandos, resultados e regras de orquestração.

Na camada de infraestrutura, objetos representam repositórios, clients HTTP, produtores de mensagens, configurações, entidades de persistência e adaptadores externos.

Em APIs REST, objetos aparecem em requests, responses, controllers, services e validadores.

Em frameworks como Spring, a própria ideia de componentes, beans, services, repositories e controllers se apoia fortemente na criação, configuração e colaboração entre objetos.

Em entrevistas técnicas, entender POO ajuda a explicar temas como encapsulamento, coesão, acoplamento, SOLID, design patterns, Clean Architecture, DDD, testes unitários e modelagem de domínio.

## Perguntas de revisão

1 - O que é um paradigma de programação?
Resposta: é uma forma de organizar o pensamento e a estrutura do código. Paradigmas diferentes propõem formas diferentes de resolver problemas, como programação procedural, orientada a objetos, funcional e reativa.

2 - Qual problema a Programação Orientada a Objetos ajuda a resolver?
Resposta: ela ajuda a organizar sistemas complexos em partes com responsabilidades claras, reduzindo código espalhado, estado global descontrolado e regras de negócio duplicadas.

3 - Qual é a diferença entre programação procedural e programação orientada a objetos?
Resposta: na programação procedural, o foco está em procedimentos e passos executados sobre dados. Na orientação a objetos, o foco está em objetos que combinam dados, comportamento e responsabilidade.

4 - Java é uma linguagem totalmente orientada a objetos?
Resposta: não em sentido absoluto. Java é fortemente orientada a objetos, mas possui tipos primitivos, membros estáticos e recursos que permitem código procedural. A própria linguagem separa tipos primitivos de tipos de referência.

5 - O que é um objeto em uma visão inicial?
Resposta: é uma unidade do sistema que representa alguma coisa relevante e pode possuir estado, comportamento e responsabilidade.

6 - O que é uma classe em uma visão inicial?
Resposta: é a definição de um tipo. Ela descreve quais dados e comportamentos os objetos daquele tipo podem ter.

7 - O que significa dizer que objetos colaboram entre si?
Resposta: significa que um objeto pode chamar métodos de outro para realizar uma tarefa. Em Java, essa colaboração normalmente acontece por chamadas de métodos.

8 - Por que variáveis globais podem ser perigosas em sistemas grandes?
Resposta: porque podem ser lidas e alteradas por várias partes do programa, dificultando rastrear quem mudou o estado e por qual motivo.

9 - Por que encapsular estado interno é importante?
Resposta: porque impede que qualquer parte do sistema altere dados internos sem passar pelas regras definidas pelo próprio objeto.

10 - Quando um `record` pode ser útil em Java?
Resposta: quando queremos representar dados simples de forma concisa, especialmente valores que carregam informações e não exigem identidade mutável complexa.

11 - Quando uma classe comum pode ser melhor que um `record`?
Resposta: quando o objeto possui ciclo de vida, identidade, mudanças controladas de estado e comportamentos relevantes.

12 - O que é modelagem de domínio?
Resposta: é o processo de representar conceitos e regras importantes do negócio no código, usando nomes, estruturas e responsabilidades próximas da realidade do sistema.

13 - POO elimina a necessidade de programação funcional?
Resposta: não. Em Java moderno, os paradigmas podem ser combinados. Um sistema pode ter objetos bem modelados e, ao mesmo tempo, usar lambdas, Streams e funções para transformar dados.

14 - Qual é o principal risco de exagerar na orientação a objetos?
Resposta: criar abstrações demais, classes demais e hierarquias difíceis de entender, sem ganho real para o sistema.

15 - Qual é a principal habilidade ao aplicar POO em backend?
Resposta: distribuir responsabilidades corretamente, mantendo alta coesão, baixo acoplamento e regras de negócio protegidas em lugares adequados.

## Resumo final

Programação Orientada a Objetos é uma forma de organizar software em torno de objetos com responsabilidades claras.

Em Java, esse paradigma é central porque a linguagem trabalha fortemente com classes, objetos, interfaces e tipos de referência. Porém, Java moderno também permite combinar POO com recursos funcionais, como lambdas, além de recursos mais recentes como records e sealed classes.

O mais importante não é decorar termos como classe, objeto, atributo ou método. O mais importante é entender a mudança de mentalidade: sair de um código baseado apenas em comandos e variáveis soltas para um modelo em que cada parte do sistema tem uma responsabilidade compreensível.

Em sistemas backend reais, POO aparece na modelagem de domínio, nos casos de uso, nos services, nos controllers, nos repositories, nas integrações e nos testes. Quando bem aplicada, melhora clareza, manutenção e evolução. Quando mal aplicada, pode gerar excesso de abstração e complexidade desnecessária.

## Referências

* Oracle. **Java SE 25 API Documentation**. ([Oracle Docs][9])
* Oracle. **Java Language Specification — Java SE 25 Edition**. ([Oracle Docs][1])
* Oracle. **Java SE Downloads — JDK 25 como LTS mais recente**. ([Oracle][10])
* Oracle. **Java SE Support Roadmap**. ([Oracle][8])
* OpenJDK. **JEP 126: Lambda Expressions & Virtual Extension Methods**. ([OpenJDK][2])
* OpenJDK. **JEP 286: Local-Variable Type Inference**. ([OpenJDK][5])
* OpenJDK. **JEP 395: Records**. ([OpenJDK][6])
* OpenJDK. **JEP 409: Sealed Classes**. ([OpenJDK][7])
* Oracle. **java.lang.Object — Java SE 25 API**. ([Oracle Docs][3])
* Bloch, Joshua. **Effective Java**. Addison-Wesley.
* Goetz, Brian et al. **Java Concurrency in Practice**. Addison-Wesley.

[1]: https://docs.oracle.com/javase/specs/jls/se25/html/index.html "The Java® Language Specification"
[2]: https://openjdk.org/jeps/126 "JEP 126: Lambda Expressions & Virtual Extension Methods"
[3]: https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/Object.html "Object (Java SE 25 & JDK 25)"
[4]: https://docs.oracle.com/javase/specs/jls/se25/jls25.pdf "The Java® Language Specification"
[5]: https://openjdk.org/jeps/286 "JEP 286: Local-Variable Type Inference"
[6]: https://openjdk.org/jeps/395 "JEP 395: Records"
[7]: https://openjdk.org/jeps/409 "JEP 409: Sealed Classes"
[8]: https://www.oracle.com/java/technologies/java-se-support-roadmap.html "Java SE Support Roadmap"
[9]: https://docs.oracle.com/en/java/javase/25/docs/api/index.html "Overview (Java SE 25 & JDK 25)"
[10]: https://www.oracle.com/java/technologies/downloads/ "Java Downloads"
