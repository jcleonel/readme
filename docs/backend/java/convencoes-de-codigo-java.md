# Convenções de código em Java

Em um projeto Java real, o maior custo raramente está em fazer o código compilar pela primeira vez. O custo maior aparece depois: quando outra pessoa precisa entender, revisar, alterar, corrigir, testar ou evoluir aquele código meses depois.

Convenções de código resolvem um problema de comunicação entre desenvolvedores. Elas reduzem o esforço mental necessário para ler o código porque criam um padrão previsível para nomes, organização, espaçamento, quebras de linha, chaves, comentários e estrutura dos arquivos.

Em uma aplicação como `meucatalogomusical.com`, por exemplo, imagine que existam classes para cadastrar bandas, músicas, gêneros musicais, álbuns, playlists e recomendações. Se cada pessoa do time escrever de um jeito, o código vira uma mistura de estilos:

```java
public class bandaService{
public void CadastrarBanda(String Nome){...}
}
```

```java
public class BandaService {

    public void cadastrarBanda(String nome) {
        ...
    }
}
```

Os dois exemplos podem representar uma ideia parecida, mas apenas o segundo segue uma forma mais idiomática para Java: nome de classe em `UpperCamelCase`, nome de método em `lowerCamelCase`, espaço antes da abertura do bloco e estrutura visual consistente.

A Oracle mantém um documento clássico de convenções de código para Java, hoje arquivado e não mais mantido ativamente, mas que ainda registra princípios importantes: convenções ajudam na legibilidade e facilitam que engenheiros entendam código novo com mais rapidez. O próprio documento informa que sua última revisão foi em 20 de abril de 1999, então ele deve ser tratado como referência histórica, não como a única fonte moderna de estilo. ([Oracle][1])

Os exemplos deste artigo usam Java 25 como referência, porque o JDK 25 é a versão LTS atual da plataforma Java SE, embora o JDK 26 já exista como release mais recente não-LTS. ([Oracle][2])

## Antes de entrar no código: uma ideia simples

Convenção de código é como combinar um idioma dentro do time.

Não muda a lógica do sistema, mas muda a facilidade de leitura. Da mesma forma que um texto sem pontuação fica cansativo, um código sem padrão visual obriga o leitor a gastar energia entendendo forma antes de entender comportamento.

Uma boa convenção responde perguntas como:

* Classes devem usar `BandaService` ou `banda_service`?
* Métodos devem usar `cadastrarMusica` ou `CadastrarMusica`?
* Deve existir espaço entre o nome do método e os parênteses?
* A chave `{` deve ficar na mesma linha ou na linha seguinte?
* Deve-se usar `var` em todo lugar ou apenas quando o tipo for óbvio?
* Um `if` de uma linha pode ficar sem chaves?
* Qual é o tamanho máximo aceitável de uma linha?
* O time vai usar formatter automático ou revisar isso manualmente?

Essas decisões parecem pequenas, mas em um projeto com centenas de classes elas afetam diretamente legibilidade, revisões de pull request, conflitos de merge e manutenção.

## Definição técnica

Convenções de código são regras adotadas por um projeto, time ou organização para padronizar a forma como o código-fonte é escrito.

Elas podem envolver:

* nomenclatura de pacotes, classes, interfaces, métodos, variáveis e constantes;
* formatação de blocos, chaves, indentação e quebras de linha;
* organização de arquivos, imports e membros da classe;
* uso de comentários e documentação;
* escolhas de estilo para recursos modernos da linguagem, como `record`, `var`, `switch expression` e `text blocks`;
* automação com formatadores, linters e validações no pipeline.

É importante separar duas coisas:

**A especificação da linguagem Java** define o que é sintaticamente válido e como o compilador deve interpretar o programa. A documentação Java SE 25 inclui, entre outras coisas, a Java Language Specification, a Java Virtual Machine Specification e a API Specification. ([Oracle Docs][3])

**A convenção de código** define como o time prefere escrever código válido de uma maneira consistente, legível e sustentável.

Por exemplo, isto pode ser válido para o compilador:

```java
public class Banda{public String nome(){return "Rush";}}
```

Mas não é desejável em um projeto profissional porque prejudica leitura, revisão e manutenção.

Uma versão mais convencional seria:

```java
public class Banda {

    public String nome() {
        return "Rush";
    }
}
```

A diferença não está na capacidade do compilador entender o código. Está na capacidade de pessoas entenderem e manterem o código com segurança.

## Como funciona em Java

Java tem algumas convenções amplamente adotadas pela comunidade. Elas não são todas obrigatórias pela linguagem, mas aparecem com frequência em bibliotecas, frameworks, documentação oficial, IDEs e projetos profissionais.

### Nomes de pacotes

Pacotes normalmente usam letras minúsculas, com componentes separados por ponto:

```java
package com.meucatalogomusical.catalogo;
```

Evite:

```java
package com.MeuCatalogoMusical.Catalogo;
```

A convenção clássica da Oracle recomenda prefixos de pacotes em letras minúsculas e nomes únicos baseados em domínio, como `com`, `org`, `net` e equivalentes. ([Oracle][4])

Em backend Java, isso ajuda a organizar responsabilidades:

```text
com.meucatalogomusical.catalogo.domain
com.meucatalogomusical.catalogo.application
com.meucatalogomusical.catalogo.infrastructure
```

### Nomes de classes, records e enums

Classes, records e enums normalmente usam `UpperCamelCase`:

```java
public class CatalogoMusicalService {
}
```

```java
public record MusicaResumo(String titulo, String banda) {
}
```

```java
public enum GeneroMusical {
    METAL,
    HARD_ROCK,
    ROCK_PROGRESSIVO
}
```

A convenção clássica da Oracle recomenda nomes de classes como substantivos, em mixed case, com a primeira letra de cada palavra interna em maiúscula, evitando abreviações desnecessárias. ([Oracle][4])

### Nomes de métodos e variáveis

Métodos e variáveis normalmente usam `lowerCamelCase`:

```java
public void cadastrarBanda(String nome) {
}
```

```java
var quantidadeDeMusicas = 12;
```

Evite nomes vagos:

```java
var x = 12;
var dados = buscar();
```

Prefira nomes que expressem intenção:

```java
var quantidadeDeMusicas = 12;
var bandasAtivas = buscarBandasAtivas();
```

O uso de `var` existe desde Java 10 pela JEP 286 e permite inferência de tipo para variáveis locais com inicializador. A própria JEP restringe o recurso a variáveis locais, índices de `for` e variáveis locais em loops, não permitindo seu uso em campos, parâmetros ou retornos de métodos. ([OpenJDK][5])

### Nomes de constantes

Constantes normalmente usam `UPPER_SNAKE_CASE`:

```java
private static final int LIMITE_PADRAO_DE_RESULTADOS = 20;
```

Esse padrão ajuda o leitor a reconhecer rapidamente que o valor é estático, final e tratado como constante.

### Chaves e blocos

Em Java, o estilo mais comum é abrir a chave `{` na mesma linha da declaração do método, classe, construtor, `if`, `for`, `while`, `switch` ou bloco equivalente:

```java
public void cadastrarMusica(String titulo) {
    validarTitulo(titulo);
    salvarMusica(titulo);
}
```

O Google Java Style Guide adota o estilo em que não há quebra de linha antes da chave de abertura, há quebra depois da chave de abertura e há quebra antes da chave de fechamento. ([Google GitHub][6])

Evite:

```java
public void cadastrarMusica(String titulo)
{
    validarTitulo(titulo);
    salvarMusica(titulo);
}
```

Esse segundo estilo pode existir em outras linguagens ou bases antigas, mas em Java moderno é menos comum.

### Espaços

Uma regra importante: não se coloca espaço entre o nome do método e o parêntese de abertura.

Correto:

```java
cadastrarMusica("Rebirth");
```

Evite:

```java
cadastrarMusica ("Rebirth");
```

A convenção clássica da Oracle recomenda não usar espaço entre o nome do método e o parêntese de abertura, justamente para diferenciar visualmente chamadas de método de palavras-chave como `if`, `while` e `for`. ([Oracle][7])

Por outro lado, estruturas de controle costumam ter espaço entre a palavra-chave e o parêntese:

```java
if (bandaAtiva) {
    publicarNoCatalogo();
}
```

Também é comum usar espaços ao redor de operadores binários:

```java
var total = quantidadeDeAlbuns + quantidadeDeSingles;
```

A convenção da Oracle também recomenda espaços ao redor de operadores binários, exceto o operador de acesso `.`. ([Oracle][7])

### Chaves mesmo em blocos de uma linha

Prefira sempre usar chaves em estruturas de controle, mesmo quando houver apenas uma instrução:

```java
if (bandaAtiva) {
    publicarNoCatalogo();
}
```

Evite:

```java
if (bandaAtiva)
    publicarNoCatalogo();
```

O problema do segundo estilo aparece quando alguém adiciona uma segunda linha depois:

```java
if (bandaAtiva)
    publicarNoCatalogo();
    enviarNotificacao();
```

Visualmente pode parecer que as duas chamadas estão dentro do `if`, mas apenas a primeira está. A convenção clássica da Oracle recomenda chaves mesmo para instruções únicas em estruturas de controle, reduzindo o risco de bugs ao adicionar novas instruções depois. ([Oracle][8])

### Linhas em branco

Linhas em branco devem separar ideias, não decorar o código aleatoriamente.

Bom uso:

```java
public void cadastrarBanda(NovaBanda novaBanda) {
    validarNome(novaBanda.nome());
    validarGenero(novaBanda.genero());

    var banda = new Banda(novaBanda.nome(), novaBanda.genero());

    bandaRepository.salvar(banda);
}
```

Aqui existem três blocos lógicos:

1. validação;
2. criação do objeto;
3. persistência.

A convenção clássica da Oracle recomenda linhas em branco para separar seções logicamente relacionadas do código, inclusive entre métodos e entre seções lógicas dentro de um método. ([Oracle][7])

## Exemplo prático

A seguir, um exemplo simples usando Java 25 para uma parte da aplicação fictícia `meucatalogomusical.com`.

```java
package com.meucatalogomusical.catalogo.application;

import java.util.List;
import java.util.Objects;

public class CatalogoMusicalService {

    private static final int LIMITE_PADRAO_DE_RESULTADOS = 20;

    private final BandaRepository bandaRepository;

    public CatalogoMusicalService(BandaRepository bandaRepository) {
        this.bandaRepository = Objects.requireNonNull(bandaRepository);
    }

    public List<BandaResumo> listarBandasPorGenero(GeneroMusical genero) {
        Objects.requireNonNull(genero);

        var limite = LIMITE_PADRAO_DE_RESULTADOS;
        var descricaoGenero = descreverGenero(genero);

        return bandaRepository.buscarPorGenero(descricaoGenero, limite);
    }

    private String descreverGenero(GeneroMusical genero) {
        return switch (genero) {
            case METAL -> "Metal";
            case HARD_ROCK -> "Hard Rock";
            case ROCK_PROGRESSIVO -> "Rock Progressivo";
        };
    }
}
```

```java
package com.meucatalogomusical.catalogo.application;

public enum GeneroMusical {
    METAL,
    HARD_ROCK,
    ROCK_PROGRESSIVO
}
```

```java
package com.meucatalogomusical.catalogo.application;

public record BandaResumo(
        String nome,
        String genero,
        int quantidadeDeMusicas
) {
}
```

```java
package com.meucatalogomusical.catalogo.application;

import java.util.List;

public interface BandaRepository {

    List<BandaResumo> buscarPorGenero(String genero, int limite);
}
```

## Analisando o código

O pacote segue uma estrutura em letras minúsculas:

```java
package com.meucatalogomusical.catalogo.application;
```

Isso comunica que a classe pertence à camada de aplicação do contexto de catálogo musical.

A classe usa `UpperCamelCase`:

```java
public class CatalogoMusicalService {
```

O nome sugere responsabilidade de serviço de aplicação para operações do catálogo musical. Não é um nome genérico como `Service`, `Manager` ou `Processor`.

A constante usa `UPPER_SNAKE_CASE`:

```java
private static final int LIMITE_PADRAO_DE_RESULTADOS = 20;
```

Isso deixa claro que o valor representa uma constante interna da classe.

O campo usa `lowerCamelCase`:

```java
private final BandaRepository bandaRepository;
```

O nome indica claramente a dependência usada para buscar bandas.

O construtor segue a mesma linha da abertura de chave:

```java
public CatalogoMusicalService(BandaRepository bandaRepository) {
```

Não há espaço entre o nome do construtor e `(`, mas existe um espaço entre `)` e `{`.

O método público também usa `lowerCamelCase`:

```java
public List<BandaResumo> listarBandasPorGenero(GeneroMusical genero) {
```

O nome começa com verbo e comunica a ação executada.

O uso de `Objects.requireNonNull` torna explícita uma pré-condição:

```java
Objects.requireNonNull(genero);
```

Em um sistema real, talvez essa validação fosse substituída por uma exceção de domínio mais descritiva, dependendo da arquitetura.

O uso de `var` aparece onde o tipo é fácil de inferir pelo lado direito:

```java
var limite = LIMITE_PADRAO_DE_RESULTADOS;
var descricaoGenero = descreverGenero(genero);
```

Apesar de `var` reduzir repetição, ele deve ser usado com cuidado. Se o lado direito não deixar claro o tipo ou a intenção, o código fica menos legível.

O `switch` expression é um recurso moderno estabilizado no Java 14 pela JEP 361. Ele permite usar `switch` como expressão, com `case ->`, evitando o comportamento tradicional de `fall through` quando se usa esse formato. ([OpenJDK][9])

```java
return switch (genero) {
    case METAL -> "Metal";
    case HARD_ROCK -> "Hard Rock";
    case ROCK_PROGRESSIVO -> "Rock Progressivo";
};
```

O `record` `BandaResumo` é adequado para transportar dados imutáveis de resumo:

```java
public record BandaResumo(
        String nome,
        String genero,
        int quantidadeDeMusicas
) {
}
```

Records foram finalizados no Java 16 pela JEP 395. ([OpenJDK][10])

### Comparação com versões anteriores do Java

As convenções principais de nomes, chaves, espaços e organização existem há muito tempo e não dependem do Java 25. O que mudou com as versões mais recentes foi o conjunto de construções que também precisam de estilo consistente.

Antes do Java 10, não existia `var` para variáveis locais. O código geralmente ficaria assim:

```java
int limite = LIMITE_PADRAO_DE_RESULTADOS;
String descricaoGenero = descreverGenero(genero);
```

A partir do Java 10, com a JEP 286, isso pode ser escrito assim:

```java
var limite = LIMITE_PADRAO_DE_RESULTADOS;
var descricaoGenero = descreverGenero(genero);
```

Antes do Java 14, o `switch` era usado principalmente como statement:

```java
private String descreverGenero(GeneroMusical genero) {
    switch (genero) {
        case METAL:
            return "Metal";
        case HARD_ROCK:
            return "Hard Rock";
        case ROCK_PROGRESSIVO:
            return "Rock Progressivo";
        default:
            throw new IllegalArgumentException("Gênero inválido: " + genero);
    }
}
```

A partir do Java 14, com a JEP 361, o `switch` expression permite uma escrita mais direta:

```java
private String descreverGenero(GeneroMusical genero) {
    return switch (genero) {
        case METAL -> "Metal";
        case HARD_ROCK -> "Hard Rock";
        case ROCK_PROGRESSIVO -> "Rock Progressivo";
    };
}
```

Antes do Java 16, um DTO simples normalmente exigia classe com campos, construtor, getters, `equals`, `hashCode` e `toString`. A partir do Java 16, com records, esse tipo de estrutura pode ser representado de forma mais compacta quando a intenção for modelar dados transparentes e imutáveis. ([OpenJDK][10])

Antes do Java 15, strings multilinha eram mais verbosas. A JEP 378 adicionou text blocks como literais multilinha, reduzindo escapes e concatenações em casos como JSON, SQL e HTML. ([OpenJDK][11])

Exemplo com text block:

```java
var json = """
        {
          "banda": "Iron Maiden",
          "genero": "Heavy metal",
          "musicasCadastradas": 42
        }
        """;
```

Nesse caso, a convenção precisa definir como alinhar o conteúdo do text block para não prejudicar leitura nem gerar surpresas em testes que comparam strings.

## O que acontece por baixo dos panos

O compilador Java não se importa com a maior parte da formatação visual usada por humanos.

A Java Language Specification define que o código-fonte passa por etapas léxicas até ser reduzido a elementos como espaços em branco, comentários e tokens. Depois, espaços em branco e comentários são descartados, ficando os tokens que compõem a gramática sintática da linguagem. ([Oracle Docs][12])

Por isso, para o compilador, estas duas versões podem ser equivalentes:

```java
public class Banda {
    public String nome() {
        return "Rush";
    }
}
```

```java
public class Banda{public String nome(){return "Rush";}}
```

Mas para humanos elas não são equivalentes. A primeira comunica estrutura. A segunda exige esforço desnecessário.

Também é importante entender que espaços podem ser necessários para separar tokens. A JLS mostra que `staticvoid` seria lido como um único identificador, enquanto `static void` é lido como duas palavras-chave separadas por espaço. ([Oracle Docs][12])

Identificadores em Java também têm regras próprias. A JLS define identificador como uma sequência de letras e dígitos Java, em que o primeiro caractere precisa ser uma letra Java, e informa que palavras reservadas não podem ser usadas como identificadores comuns. ([Oracle Docs][12])

Isso significa que convenção não substitui a regra da linguagem. Por exemplo:

```java
var class = "Metallica";
```

Esse código não compila porque `class` é palavra reservada.

Já isto pode compilar, mas viola convenções:

```java
var NomeDaBanda = "Metallica";
```

O problema aqui não é sintático. O problema é de estilo: variáveis locais em Java devem usar `lowerCamelCase`.

Text blocks merecem atenção especial. Ao contrário de espaços comuns fora de strings, a formatação dentro de um text block participa de um processamento específico de compilação. A JEP 378 descreve que o compilador normaliza terminadores de linha, remove espaços incidentais e interpreta escapes; depois, o conteúdo é registrado no class file como uma string constante, sem diferenciar se veio de text block ou string literal tradicional. ([OpenJDK][11])

Portanto, a maior parte da convenção não muda performance nem bytecode. Ela muda legibilidade. Mas em strings, text blocks e literais, espaços e quebras de linha podem alterar o valor real produzido.

## Boas práticas

### Escolha um guia base e adapte pouco

O time pode usar o Google Java Style Guide, as convenções clássicas da Oracle, ou uma convenção interna baseada nelas. O mais importante é não transformar cada pull request em uma discussão subjetiva sobre estilo.

O Google Java Style Guide define regras modernas para formatação, incluindo estilo de chaves, indentação de `switch` e tratamento de construções novas da linguagem. ([Google GitHub][6])

### Automatize formatação

Não dependa apenas de revisão humana para corrigir espaço, chave, import e indentação.

Use ferramentas como:

```text
.editorconfig
google-java-format
Checkstyle
Spotless
configurações compartilhadas da IDE
validação no pipeline
```

EditorConfig ajuda a manter estilos consistentes entre diferentes editores e IDEs por meio de um arquivo versionado no projeto. ([editorconfig.org][13])

O `google-java-format` reformata código Java para seguir o Google Java Style e informa que sua falta de configurabilidade é uma decisão deliberada para unificar o formato. ([GitHub][14])

### Padronize nomes por intenção

Prefira nomes que revelem o papel do elemento no domínio:

```java
var bandasAtivas = buscarBandasAtivas();
var musicaMaisTocada = ranking.primeiraColocada();
var generoPrincipal = banda.generoPrincipal();
```

Evite nomes que dependem de contexto mental:

```java
var lista = buscar();
var obj = ranking.primeiraColocada();
var tipo = banda.generoPrincipal();
```

### Use `var` com moderação

Bom uso:

```java
var bandas = bandaRepository.buscarPorGenero("Metal", 20);
```

Uso ruim:

```java
var resultado = processar(dados);
```

No segundo caso, o tipo e a intenção são vagos. O problema não é o `var` em si, mas a combinação de nome genérico com chamada genérica.

### Use chaves mesmo em blocos simples

Prefira:

```java
if (catalogoEstaPublicado) {
    atualizarIndiceDeBusca();
}
```

Evite:

```java
if (catalogoEstaPublicado)
    atualizarIndiceDeBusca();
```

Essa prática reduz bugs em alterações futuras.

### Evite abreviações desnecessárias

Prefira:

```java
quantidadeDeMusicas
```

Em vez de:

```java
qtdMsc
```

Abreviações economizam poucos caracteres e aumentam o custo de leitura.

### Organize a classe de forma previsível

Uma ordem comum é:

```text
constantes
campos
construtores
métodos públicos
métodos privados auxiliares
```

Exemplo:

```java
public class PlaylistService {

    private static final int TAMANHO_MINIMO_DA_PLAYLIST = 3;

    private final MusicaRepository musicaRepository;

    public PlaylistService(MusicaRepository musicaRepository) {
        this.musicaRepository = Objects.requireNonNull(musicaRepository);
    }

    public Playlist criarPlaylist(String nome, List<Long> musicasIds) {
        validarQuantidadeDeMusicas(musicasIds);
        return montarPlaylist(nome, musicasIds);
    }

    private void validarQuantidadeDeMusicas(List<Long> musicasIds) {
        if (musicasIds.size() < TAMANHO_MINIMO_DA_PLAYLIST) {
            throw new IllegalArgumentException("A playlist precisa ter pelo menos 3 músicas.");
        }
    }

    private Playlist montarPlaylist(String nome, List<Long> musicasIds) {
        var musicas = musicaRepository.buscarPorIds(musicasIds);
        return new Playlist(nome, musicas);
    }
}
```

## Erros comuns

### Colocar espaço entre nome do método e parênteses

Evite:

```java
cadastrarBanda ("Rush");
```

Prefira:

```java
cadastrarBanda("Rush");
```

### Usar nome de método começando com maiúscula

Evite:

```java
public void CadastrarBanda(String nome) {
}
```

Prefira:

```java
public void cadastrarBanda(String nome) {
}
```

### Nomear variável local como se fosse classe

Evite:

```java
var Banda = buscarBanda();
```

Prefira:

```java
var banda = buscarBanda();
```

### Omitir chaves em `if`, `for` e `while`

Evite:

```java
if (bandaAtiva)
    publicarBanda();
```

Prefira:

```java
if (bandaAtiva) {
    publicarBanda();
}
```

### Usar `var` quando o tipo não está claro

Evite:

```java
var resposta = cliente.executar(requisicao);
```

Prefira uma das opções abaixo, dependendo do contexto:

```java
RespostaCatalogo resposta = cliente.executar(requisicao);
```

Ou melhore o nome:

```java
var respostaDoCatalogo = cliente.executar(requisicao);
```

### Misturar estilos no mesmo projeto

Evite uma classe assim:

```java
public class BandaService
{
    public void cadastrar_banda(String Nome){
        if(Nome != null)
            salvar(Nome);
    }
}
```

Prefira consistência:

```java
public class BandaService {

    public void cadastrarBanda(String nome) {
        if (nome != null) {
            salvar(nome);
        }
    }
}
```

### Transformar convenção em gosto pessoal

Um erro comum em times é discutir estilo como preferência individual.

A pergunta correta não é:

> Eu gosto mais desse jeito?

A pergunta correta é:

> Qual padrão reduz custo de manutenção para o time?

## Decisões de engenharia

### Quando usar

Use convenções de código sempre que houver código mantido por mais de uma pessoa, revisado em pull requests, versionado em Git ou executado em produção.

Em sistemas backend, convenções são especialmente importantes porque o código atravessa várias camadas:

```text
Controller
Use Case
Service
Domain Model
Repository
Client HTTP
Mensageria
Testes
Configuração
```

Sem padrão, cada camada pode parecer escrita por um time diferente.

### Quando evitar excesso de regras

Evite criar uma convenção interna grande demais, cheia de exceções subjetivas.

Regras demais geram custo de adoção e discussões desnecessárias. O ideal é combinar um guia base, automatizar o que for mecânico e documentar apenas decisões específicas do projeto.

Evite, por exemplo, discutir manualmente em pull request:

```text
faltou espaço
quebrou linha errado
import fora de ordem
chave deveria subir
```

Esses pontos devem ser resolvidos por formatter ou linter.

### Alternativas

As principais alternativas são:

| Abordagem                              | Vantagem                             | Risco                                           |
| -------------------------------------- | ------------------------------------ | ----------------------------------------------- |
| Sem convenção formal                   | Menos configuração inicial           | Código inconsistente e revisões subjetivas      |
| Convenção interna manual               | Personalização total                 | Alto custo de manutenção                        |
| Guia conhecido, como Google Java Style | Fácil adoção e boa documentação      | Pode não refletir todas as preferências do time |
| Formatter opinativo                    | Reduz discussão humana               | Menos flexibilidade                             |
| Linter no pipeline                     | Impede inconsistência antes do merge | Pode incomodar se mal configurado               |

### Impacto em legibilidade

Convenções aumentam previsibilidade. O leitor reconhece rapidamente o que é classe, método, constante, variável local, pacote e bloco lógico.

### Impacto em manutenção

Código padronizado reduz tempo de onboarding, facilita reviews, diminui ruído em diffs e ajuda a encontrar problemas reais em vez de discutir formatação.

### Impacto em performance

Na maior parte dos casos, convenções de formatação não afetam performance. Chaves, indentação, quebras de linha e espaços fora de literais são relevantes para leitura humana, não para execução.

A exceção está em conteúdo textual: strings, text blocks, SQL, JSON e templates. Nesses casos, espaços e quebras podem fazer parte do valor final.

### Impacto em arquitetura

Convenções também reforçam arquitetura.

Em um projeto com Clean Architecture, por exemplo, nomes consistentes ajudam a deixar claro o papel de cada classe:

```text
CadastrarBandaUseCase
BandaRepository
BandaGateway
BandaController
BandaResponse
NovaBandaRequest
```

A convenção não cria a arquitetura, mas torna a arquitetura mais visível.

## Onde isso aparece em sistemas reais

### Pull requests

Em PRs, convenções reduzem comentários superficiais.

Sem padrão, a revisão vira:

```text
Ajustar indentação.
Renomear variável.
Subir chave.
Remover espaço.
Organizar imports.
```

Com padrão automatizado, a revisão foca em:

```text
Essa regra de negócio está correta?
Esse método pertence a essa classe?
Essa exceção faz sentido?
Esse teste cobre o cenário crítico?
Essa query escala?
```

### CI/CD

Pipelines podem bloquear merge quando o código viola estilo mínimo.

Exemplo de etapas:

```text
compile
test
formatCheck
checkstyle
spotbugs
jacocoTestReport
build
```

### Onboarding

Um novo desenvolvedor entende mais rápido o projeto quando nomes e estrutura são previsíveis.

Ao ver:

```java
public class CadastrarMusicaUseCase {
}
```

Ele entende uma intenção.

Ao ver:

```java
public class ProcMus {
}
```

Ele precisa investigar.

### Testes

Convenções também aparecem em testes:

```java
class CatalogoMusicalServiceTest {

    @Test
    void deveListarBandasPorGenero() {
        ...
    }
}
```

Nomes de teste claros reduzem a necessidade de comentários.

### APIs

Convenções internas de Java também influenciam contratos externos.

Uma classe Java pode usar:

```java
private String nomeDaBanda;
```

Mas a API JSON pode expor:

```json
{
  "nomeDaBanda": "Black Sabbath"
}
```

Ou, dependendo da convenção da API:

```json
{
  "nome_da_banda": "Black Sabbath"
}
```

O importante é separar convenção interna da linguagem Java e convenção externa do contrato HTTP/JSON.

Segue uma versão pronta para substituir a seção **Perguntas de revisão** no artigo:

## Perguntas de revisão

### 1 - Qual problema real as convenções de código resolvem em um time de desenvolvimento?

Resposta:
Elas reduzem o custo de leitura, revisão e manutenção do código. Em um time, o código raramente é lido apenas por quem escreveu. Convenções criam um padrão visual e estrutural que ajuda qualquer pessoa a entender classes, métodos, variáveis, constantes e blocos de código com menos esforço. A própria documentação clássica da Oracle destaca que convenções melhoram a legibilidade e ajudam engenheiros a entender código novo com mais rapidez.

### 2 - Qual é a diferença entre regra da linguagem Java e convenção de código?

Resposta:
Regra da linguagem é aquilo que o compilador exige para o programa ser válido. Convenção de código é aquilo que o time adota para o código ficar legível e consistente. Por exemplo, `public class Banda{}` pode compilar, mas não é uma boa forma de escrever em um projeto profissional. A Java Language Specification define a estrutura lexical e sintática da linguagem, enquanto guias de estilo tratam de legibilidade, organização e padronização.

### 3 - Por que métodos em Java normalmente usam `lowerCamelCase`?

Resposta:
Porque esse padrão diferencia métodos de classes e constantes, além de tornar o código mais previsível para quem lê. Em Java, métodos normalmente representam ações, então nomes como `cadastrarBanda`, `buscarMusicasPorGenero` e `atualizarCatalogo` comunicam comportamento. O Google Java Style Guide define que nomes de métodos devem ser escritos em `lowerCamelCase` e normalmente são verbos ou frases verbais.

### 4 - Por que classes normalmente usam `UpperCamelCase`?

Resposta:
Porque classes representam tipos, conceitos ou entidades do sistema. Usar `UpperCamelCase`, como `BandaService`, `MusicaRepository` ou `CatalogoMusicalController`, ajuda o leitor a reconhecer rapidamente que aquele identificador representa um tipo. A convenção clássica da Oracle recomenda nomes de classes como substantivos em mixed case, com a primeira letra de cada palavra em maiúscula.

### 5 - Por que constantes usam `UPPER_SNAKE_CASE`?

Resposta:
Porque esse padrão deixa claro que o valor é uma constante, ou seja, algo que não deve mudar durante a execução normal do programa. Ao ver `LIMITE_PADRAO_DE_RESULTADOS`, o leitor entende rapidamente que aquele valor tem papel fixo e especial no código. Isso evita confusão com variáveis locais, parâmetros ou campos comuns.

### 6 - Por que não é recomendado colocar espaço entre o nome do método e o parêntese de abertura?

Resposta:
Porque em Java a chamada de método deve ser visualmente diferente de estruturas de controle como `if`, `while` e `for`. O recomendado é escrever `cadastrarBanda("Rush")`, e não `cadastrarBanda ("Rush")`. A convenção clássica da Oracle orienta não usar espaço entre o nome do método e o parêntese de abertura, justamente para diferenciar chamadas de método de palavras-chave.
### 7 - Por que é recomendado usar chaves mesmo em blocos `if` de uma linha?

Resposta:
Porque isso evita bugs quando alguém adiciona novas instruções depois. Sem chaves, apenas a primeira linha pertence ao `if`, mesmo que visualmente pareça que mais linhas fazem parte do bloco. Por isso, prefira sempre:

```java
if (bandaAtiva) {
    publicarNoCatalogo();
}
```

Em vez de:

```java
if (bandaAtiva)
    publicarNoCatalogo();
```

A convenção da Oracle recomenda usar chaves em estruturas de controle mesmo quando há apenas uma instrução, para facilitar alterações futuras sem introduzir erros acidentais.

### 8 - O uso de `var` melhora ou piora a legibilidade? De que depende?

Resposta:
Depende do contexto. `var` melhora a legibilidade quando o tipo é óbvio pelo lado direito da atribuição ou quando repetir o tipo deixaria o código mais verboso sem ganho real. Mas pode piorar quando o nome da variável é genérico ou quando o método chamado não deixa claro qual tipo está sendo retornado.

Bom exemplo:

```java
var bandas = bandaRepository.buscarPorGenero("Metal", 20);
```

Exemplo ruim:

```java
var resultado = processar(dados);
```

O `var` foi introduzido no Java 10 pela JEP 286 para inferência de tipo em variáveis locais com inicializador. Ele não torna Java uma linguagem dinamicamente tipada; o tipo continua sendo definido em tempo de compilação.

### 9 - Convenções de código afetam performance?

Resposta:
Na maior parte dos casos, não. Indentação, espaços fora de strings, posição das chaves e nomes de variáveis não tornam o programa mais rápido ou mais lento. O impacto principal é em legibilidade, manutenção e revisão. Porém, convenções podem afetar indiretamente a qualidade do software, porque código mais claro tende a gerar menos bugs e ser mais fácil de otimizar quando necessário.

### 10 - Em quais situações espaços e quebras de linha podem alterar o comportamento real do programa?

Resposta:
Espaços e quebras de linha podem alterar o comportamento quando fazem parte de valores textuais, como `String`, text blocks, JSON, SQL, HTML ou mensagens comparadas em testes. Fora desses casos, muitos espaços e comentários são tratados como elementos lexicais e não afetam diretamente a execução. A JLS explica que o código-fonte é processado em elementos de entrada, incluindo espaços, comentários e tokens, e que a estrutura lexical é parte formal da linguagem.

### 11 - Como um formatter ajuda em pull requests?

Resposta:
Um formatter reduz discussões manuais sobre estilo. Em vez de gastar tempo comentando “faltou espaço”, “quebrou linha errado” ou “organize os imports”, o time deixa a ferramenta aplicar o padrão automaticamente. Isso permite que o pull request foque no que realmente importa: regra de negócio, arquitetura, testes, segurança, performance e clareza da solução. Ferramentas como `google-java-format` automatizam a formatação de código Java conforme um estilo definido.

### 12 - Qual é o risco de uma convenção interna grande demais?

Resposta:
O risco é transformar estilo de código em burocracia. Quando a convenção tem regras demais, exceções demais ou preferências muito subjetivas, o time passa a discutir detalhes pouco relevantes em vez de discutir qualidade técnica. Uma boa convenção deve ser simples, automatizável e fácil de seguir. O ideal é documentar o essencial e deixar ferramentas cuidarem do que for mecânico.

### 13 - Como convenções podem reforçar uma arquitetura em camadas?

Resposta:
Convenções tornam a arquitetura mais visível no código. Em uma aplicação backend, nomes como `CadastrarBandaUseCase`, `BandaRepository`, `BandaController`, `BandaResponse` e `NovaBandaRequest` ajudam o leitor a entender rapidamente o papel de cada classe. A convenção não cria a arquitetura sozinha, mas ajuda a comunicar limites, responsabilidades e intenção.

### 14 - Por que o documento clássico de convenções da Oracle deve ser lido como referência histórica?

Resposta:
Porque ele não está sendo mantido ativamente e sua última revisão foi feita em 20 de abril de 1999. Ainda assim, vários princípios continuam úteis, como legibilidade, nomes claros, uso consistente de espaços e chaves. O ideal é usá-lo como base histórica, complementando com guias mais atuais, ferramentas modernas e práticas adotadas pelo time.

### 15 - Como você configuraria um projeto Java para evitar discussões manuais sobre formatação?

Resposta:
Eu configuraria um padrão automatizado no projeto, versionado junto com o código. Uma boa configuração incluiria `.editorconfig` para padronizar indentação, charset e quebras de linha; um formatter como `google-java-format`; uma ferramenta de análise como Checkstyle ou Spotless; e uma etapa no pipeline para bloquear código fora do padrão. O EditorConfig ajuda a manter estilos consistentes entre diferentes editores e IDEs, especialmente em times com várias pessoas.

## Resumo final

Convenções de código em Java não existem para agradar preferências individuais. Elas existem para reduzir custo de leitura, revisão, manutenção e evolução do software.

A linguagem Java define o que é válido. A convenção define o que é legível, previsível e sustentável dentro de um time.

Em Java moderno, boas convenções envolvem nomes claros, pacotes em minúsculas, classes em `UpperCamelCase`, métodos e variáveis em `lowerCamelCase`, constantes em `UPPER_SNAKE_CASE`, chaves consistentes, uso cuidadoso de `var`, blocos sempre com chaves e automação com formatter, linter e pipeline.

Para um desenvolvedor pleno ou sênior, o ponto principal não é decorar regras isoladas. É entender que padrão de código é uma decisão de engenharia. Ele afeta colaboração, qualidade, revisão, arquitetura, onboarding e manutenção de longo prazo.

## Referências

* Oracle — Java Downloads: JDK 25 como LTS atual da plataforma Java SE. ([Oracle][2])
* Oracle — Java SE 25 Specifications: JLS, JVM Specification e API Specification. ([Oracle Docs][3])
* Oracle — The Java Language Specification, Java SE 25 Edition. ([Oracle Docs][15])
* Oracle — JLS, Chapter 3: Lexical Structure. ([Oracle Docs][12])
* Oracle — Code Conventions for the Java Programming Language. ([Oracle][1])
* Oracle — Code Conventions: Statements. ([Oracle][8])
* Oracle — Code Conventions: White Space. ([Oracle][7])
* Oracle — Code Conventions: Naming Conventions. ([Oracle][4])
* Google — Google Java Style Guide. ([Google GitHub][6])
* OpenJDK — JEP 286: Local-Variable Type Inference, Java 10. ([OpenJDK][5])
* OpenJDK — JEP 361: Switch Expressions, Java 14. ([OpenJDK][9])
* OpenJDK — JEP 378: Text Blocks, Java 15. ([OpenJDK][11])
* OpenJDK — JEP 395: Records, Java 16. ([OpenJDK][10])
* EditorConfig — configuração compartilhada de estilo entre editores e IDEs. ([editorconfig.org][13])
* Google — google-java-format. ([GitHub][14])

[1]: https://www.oracle.com/java/technologies/cc-java-programming-language.html "Code Conventions for the Java Programming Language"
[2]: https://www.oracle.com/java/technologies/downloads/ "Java Downloads | Oracle"
[3]: https://docs.oracle.com/en/java/javase/25/docs/specs/index.html "Java® Platform, Standard Edition & Java Development Kit Specifications Version 25"
[4]: https://www.oracle.com/java/technologies/javase/codeconventions-namingconventions.html "Code Conventions for the Java Programming Language: 9. Naming Conventions"
[5]: https://openjdk.org/jeps/286 "JEP 286: Local-Variable Type Inference"
[6]: https://google.github.io/styleguide/javaguide.html?hl=zh-CN "Google Java Style Guide"
[7]: https://www.oracle.com/java/technologies/javase/codeconventions-whitespace.html "Code Conventions for the Java Programming Language: 8. White Space"
[8]: https://www.oracle.com/java/technologies/javase/codeconventions-statements.html "Code Conventions for the Java Programming Language: 7. Statements"
[9]: https://openjdk.org/jeps/361 "JEP 361: Switch Expressions"
[10]: https://openjdk.org/jeps/395 "JEP 395: Records"
[11]: https://openjdk.org/jeps/378 "JEP 378: Text Blocks"
[12]: https://docs.oracle.com/javase/specs/jls/se25/html/jls-3.html "Chapter 3. Lexical Structure"
[13]: https://editorconfig.org/index "EditorConfig"
[14]: https://github.com/google/google-java-format "GitHub - google/google-java-format: Reformats Java source code to comply with Google Java Style. · GitHub"
[15]: https://docs.oracle.com/javase/specs/jls/se25/html/index.html "The Java® Language Specification"
