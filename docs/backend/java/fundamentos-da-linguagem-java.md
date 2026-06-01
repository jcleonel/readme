# Fundamentos da linguagem Java

Todo sistema Java, seja um pequeno programa de terminal, uma API REST com Spring Boot ou um serviço em microsserviços, começa com a mesma ideia central: escrever código-fonte, transformá-lo em uma forma executável pela plataforma Java e iniciar a execução por um ponto de entrada.

Em aplicações backend, esse fluxo aparece em vários lugares:

- ao rodar uma classe com `main`;
- ao iniciar uma aplicação Spring Boot;
- ao compilar um projeto com Maven ou Gradle;
- ao gerar um artefato `.jar`;
- ao executar esse artefato dentro de um container Docker;
- ao investigar erros como `ClassNotFoundException`, `NoClassDefFoundError` ou incompatibilidade de versão de bytecode.

Por isso, entender o primeiro programa Java não é apenas uma etapa inicial de aprendizado. É a base para compreender como a linguagem, o compilador, a JVM e o comando `java` trabalham juntos.

Neste artigo, os exemplos usam Java 25, versão LTS no ecossistema Oracle/JDK 25.

## Antes de entrar no código: uma ideia simples

Um programa Java passa por três momentos principais:

1. você escreve um arquivo `.java`;
2. o compilador Java transforma esse arquivo em bytecode, normalmente em arquivos `.class`;
3. a JVM carrega esse bytecode e executa o programa.

De forma simplificada:

```text
Código-fonte Java (.java)
        |
        | javac
        v
Bytecode Java (.class)
        |
        | java
        v
Execução na JVM
```

O arquivo `.java` é feito para humanos escreverem e lerem.

O arquivo `.class` é feito para a JVM interpretar, validar, carregar e executar.

Essa separação é uma das razões pelas quais Java ficou conhecido pela ideia de portabilidade: o código é compilado para bytecode, e o bytecode é executado por uma JVM compatível com a plataforma onde o programa está rodando.

## Definição técnica

Em Java, o código-fonte é organizado em unidades de compilação. Uma unidade de compilação geralmente corresponde a um arquivo `.java`, que pode declarar classes, interfaces, enums, records e outros tipos.

O compilador `javac` lê arquivos-fonte escritos na linguagem Java e os compila em arquivos de classe que rodam na Java Virtual Machine. A própria documentação oficial do `javac` define o comando como responsável por ler declarações Java e compilá-las em arquivos `.class`. ([Oracle Docs][1])

A JVM, por sua vez, é a máquina virtual responsável por executar esse bytecode. Quando usamos o comando `java`, ele inicia uma aplicação Java criando uma JVM, carregando a classe especificada e chamando o método `main`. ([Oracle Docs][2])

Portanto, três conceitos são fundamentais:

### JVM

A JVM, Java Virtual Machine, é o ambiente de execução do bytecode Java.

Ela é responsável por tarefas como:

* carregar classes;
* verificar a validade estrutural do bytecode;
* preparar classes para execução;
* resolver referências simbólicas;
* inicializar classes;
* executar instruções;
* gerenciar memória;
* lidar com pilha, heap, método, constantes e outros elementos internos.

A especificação da JVM descreve áreas de execução como heap, pilhas da JVM, method area, constant pool e o processo de loading, linking e initialization. ([Oracle Docs][3])

### javac

`javac` é o compilador Java.

Ele recebe arquivos `.java` e gera arquivos `.class`.

Exemplo:

```bash
javac ResumoCatalogoMusical.java
```

Esse comando tenta compilar `ResumoCatalogoMusical.java`. Se o código estiver válido, será gerado um arquivo como:

```text
ResumoCatalogoMusical.class
```

A documentação oficial também reforça que arquivos-fonte Java usam extensão `.java`, enquanto arquivos de classe usam extensão `.class`. ([Oracle Docs][1])

### java

`java` é o launcher, ou seja, o comando usado para iniciar uma aplicação Java.

Exemplo:

```bash
java ResumoCatalogoMusical
```

Nesse caso, não usamos `.class` no comando. Informamos o nome da classe que deve ser carregada e executada. O launcher localiza a classe, inicia a JVM e chama o método `main`.

## Como funciona em Java

Vamos começar com uma estrutura tradicional, ainda muito importante para aplicações backend.

Crie um arquivo chamado:

```text
ResumoCatalogoMusical.java
```

Com o seguinte conteúdo:

```java
public class ResumoCatalogoMusical {

    public static void main(String[] args) {
        String banda = "Mammoth";
        String genero = "Hard rock";
        int totalMusicas = 12;

        System.out.println("Resumo do catálogo musical");
        System.out.println("Banda: " + banda);
        System.out.println("Gênero: " + genero);
        System.out.println("Total de músicas cadastradas: " + totalMusicas);
    }
}
```

Esse programa representa uma saída simples para uma aplicação fictícia chamada `meucatalogomusical.com`, um catálogo musical com bandas e músicas de Metal, Hard Rock e Rock Progressivo.

Para compilar:

```bash
javac ResumoCatalogoMusical.java
```

Para executar:

```bash
java ResumoCatalogoMusical
```

Saída esperada:

```text
Resumo do catálogo musical
Banda: Mammoth
Gênero: Hard rock
Total de músicas cadastradas: 12
```

## Exemplo prático

Em um projeto mais próximo de um sistema real, é comum organizar o código em pacotes.

Estrutura sugerida:

```text
src/
└── com/
    └── meucatalogomusical/
        └── console/
            └── ResumoCatalogoMusical.java
```

Arquivo:

```java
package com.meucatalogomusical.console;

public class ResumoCatalogoMusical {

    public static void main(String[] args) {
        String banda = "Metallica";
        String album = "Master of Puppets";
        int musicasPublicadas = 8;
        boolean catalogoAtivo = true;

        System.out.println("meucatalogomusical.com");
        System.out.println("Banda: " + banda);
        System.out.println("Álbum: " + album);
        System.out.println("Músicas publicadas: " + musicasPublicadas);
        System.out.println("Catálogo ativo: " + catalogoAtivo);
    }
}
```

Compilando para uma pasta de saída:

```bash
javac -d out src/com/meucatalogomusical/console/ResumoCatalogoMusical.java
```

Executando:

```bash
java -cp out com.meucatalogomusical.console.ResumoCatalogoMusical
```

Aqui aparecem dois pontos importantes.

Primeiro, `-d out` diz ao `javac` para colocar os arquivos `.class` na pasta `out`. A própria documentação do `javac` recomenda especificar um diretório separado de destino com `-d` em vez de gravar os `.class` ao lado dos arquivos-fonte. ([Oracle Docs][1])

Segundo, `-cp out` informa ao comando `java` onde procurar as classes compiladas. Como a classe está dentro de um pacote, a execução precisa usar o nome totalmente qualificado:

```text
com.meucatalogomusical.console.ResumoCatalogoMusical
```

## Analisando o código

### package

```java
package com.meucatalogomusical.console;
```

A declaração `package` organiza a classe em um namespace.

Em sistemas reais, pacotes evitam conflito de nomes e ajudam a expressar a arquitetura do projeto.

Por exemplo:

```text
com.meucatalogomusical.domain
com.meucatalogomusical.application
com.meucatalogomusical.infrastructure
```

Em uma aplicação backend maior, essa organização facilita separar domínio, casos de uso, infraestrutura, controllers, persistência e integrações externas.

### public class

```java
public class ResumoCatalogoMusical {
```

Essa linha declara uma classe pública chamada `ResumoCatalogoMusical`.

O modificador `public` permite que a classe seja acessada a partir de outros pacotes, respeitando as regras de módulos quando existirem. A JLS explica que uma classe pode ser declarada `public` para ser referenciada por código de outros pacotes. ([Oracle Docs][4])

O nome da classe usa `PascalCase`, uma variação do CamelCase usada normalmente para nomes de classes em Java.

Java diferencia letras maiúsculas e minúsculas. Portanto:

```java
ResumoCatalogoMusical
resumoCatalogoMusical
RESUMOCATALOGOMUSICAL
```

são nomes diferentes.

### Nome do arquivo e nome da classe

Quando uma classe pública é declarada em um arquivo-fonte, a prática e a regra usual é que o nome do arquivo corresponda ao nome da classe pública.

Por isso:

```java
public class ResumoCatalogoMusical
```

deve estar em:

```text
ResumoCatalogoMusical.java
```

A JLS explica que, em ambientes baseados em sistema de arquivos, uma implementação pode exigir que uma classe ou interface pública seja encontrada em um arquivo com o nome do tipo mais a extensão, como `.java`; a própria especificação exemplifica uma classe pública `Toad` no arquivo `Toad.java`. ([Oracle Docs][5])

### main

```java
public static void main(String[] args) {
```

Esse é o ponto de entrada tradicional de uma aplicação Java.

Na forma clássica, o método precisa ser:

```java
public static void main(String[] args)
```

O comando `java` inicia a JVM, carrega a classe indicada e chama esse método `main`. A documentação oficial do comando `java` descreve exatamente esse fluxo. ([Oracle Docs][2])

Cada parte tem um papel:

```text
public
```

Permite que o launcher acesse o método.

```text
static
```

Permite chamar o método sem criar manualmente uma instância da classe.

```text
void
```

Indica que o método não retorna valor.

```text
main
```

É o nome reconhecido como ponto de entrada.

```text
String[] args
```

Representa os argumentos recebidos pela linha de comando.

Exemplo:

```bash
java ResumoCatalogoMusical Metallica
```

O valor `Metallica` poderia ser lido dentro do array `args`.

### Variáveis

```java
String banda = "Metallica";
String album = "Master of Puppets";
int musicasPublicadas = 8;
boolean catalogoAtivo = true;
```

Essas linhas criam variáveis locais dentro do método `main`.

* `String` representa texto;
* `int` representa número inteiro;
* `boolean` representa verdadeiro ou falso.

Essas variáveis existem apenas durante a execução do método.

### System.out.println

```java
System.out.println("meucatalogomusical.com");
```

`System` é uma classe da API padrão Java.

A documentação oficial descreve `System` como uma classe com campos e métodos úteis, incluindo acesso à entrada padrão, saída padrão e saída de erro. ([Oracle Docs][6])

`out` é a saída padrão.

`println` escreve uma linha de texto nessa saída.

Em programas de terminal, essa saída aparece no console. Em servidores, a saída padrão frequentemente é capturada por ferramentas de log, runtime de containers ou plataformas de observabilidade.

## O que acontece por baixo dos panos

Ao executar:

```bash
javac -d out src/com/meucatalogomusical/console/ResumoCatalogoMusical.java
```

o compilador realiza várias tarefas:

1. lê o arquivo `.java`;
2. analisa a sintaxe;
3. verifica tipos;
4. valida regras da linguagem;
5. resolve referências;
6. gera bytecode;
7. grava um arquivo `.class`.

Depois, ao executar:

```bash
java -cp out com.meucatalogomusical.console.ResumoCatalogoMusical
```

o launcher Java:

1. inicia a JVM;
2. procura a classe no classpath;
3. carrega a classe;
4. realiza linking;
5. verifica o bytecode;
6. inicializa a classe quando necessário;
7. chama o método `main`.

A JLS descreve linking como o processo de combinar a representação binária de uma classe ou interface ao estado de execução da JVM, e também explica que a verificação garante que a representação binária seja estruturalmente correta. ([Oracle Docs][7])

### Por que existe bytecode?

O bytecode é uma representação intermediária.

Em vez de compilar diretamente para uma arquitetura específica, como x86 ou ARM, o Java compila para um formato entendido pela JVM.

Isso permite que o mesmo `.class` ou `.jar` seja executado em ambientes diferentes, desde que exista uma JVM compatível.

Na prática, a JVM moderna não apenas interpreta bytecode. Implementações como HotSpot também usam compilação JIT, Just-In-Time, para transformar partes quentes do código em código nativo durante a execução.

### Por que o comando é `java` e não o nome do programa?

Porque o sistema operacional não executa bytecode diretamente.

Quem sabe carregar, verificar e executar bytecode Java é a JVM.

Então, quando você escreve:

```bash
java com.meucatalogomusical.console.ResumoCatalogoMusical
```

você está dizendo:

```text
Inicie a plataforma Java e execute esta classe.
```

O comando `java` é o ponto de contato entre o terminal e a JVM.

## Java 25 e a forma moderna de escrever pequenos programas

Até aqui usamos a forma tradicional, importante para entender projetos reais e aplicações backend.

Mas o Java 25 trouxe uma evolução relevante com a JEP 512: Compact Source Files and Instance Main Methods.

Essa feature foi entregue no JDK 25 e permite escrever programas pequenos com menos cerimônia, sem abandonar a linguagem Java nem criar um dialeto separado. ([OpenJDK][8])

### Forma tradicional

```java
public class VerificarCatalogo {

    public static void main(String[] args) {
        System.out.println("Catálogo musical verificado.");
    }
}
```

### Forma com instance main no Java 25

```java
class VerificarCatalogo {

    void main() {
        IO.println("Catálogo musical verificado.");
    }
}
```

Nesse exemplo:

* o método `main` não é `static`;
* o método `main` não recebe `String[] args`;
* `IO.println` é usado no lugar de `System.out.println`.

A classe `java.lang.IO` foi adicionada no Java 25 e fornece métodos estáticos simples para entrada e saída orientadas a linha, como `print`, `println` e `readln`. ([Oracle Docs][9])

### Forma compacta no Java 25

Em um arquivo compacto, também é possível escrever:

```java
void main() {
    IO.println("Catálogo musical verificado.");
}
```

Esse formato é útil para pequenos programas, experimentos, scripts simples e aprendizado progressivo.

Mas em aplicações backend profissionais, a forma tradicional com classes explícitas, pacotes, build tool, testes e empacotamento continua sendo a mais comum.

## Comparação com versões anteriores do Java

### Antes do Java 11

Antes do Java 11, normalmente era necessário compilar explicitamente antes de executar:

```bash
javac VerificarCatalogo.java
java VerificarCatalogo
```

### Java 11 — JEP 330

O Java 11 introduziu a JEP 330, que permite executar diretamente um arquivo-fonte único com o comando `java`. ([OpenJDK][10])

Exemplo:

```bash
java VerificarCatalogo.java
```

Nesse modo, o arquivo é compilado e executado em uma única etapa. A JEP 330 descreve esse comportamento como equivalente, de forma informal, a compilar em memória e executar a classe resultante. ([OpenJDK][10])

Uso recomendado:

```text
Bom para: estudos, scripts pequenos, experimentos e protótipos.
Evitar como padrão em: builds produtivos, pipelines, distribuição de aplicações e serviços backend.
```

### Java 22 — JEP 458

O Java 22 expandiu essa ideia com a JEP 458, permitindo lançar programas compostos por múltiplos arquivos-fonte sem uma etapa explícita de compilação. ([OpenJDK][11])

A JEP 458 permite que o launcher encontre e compile outros arquivos `.java` referenciados pelo programa, tornando a transição de programas pequenos para programas com múltiplos arquivos mais gradual. ([OpenJDK][11])

### Java 25 — JEP 512

O Java 25 finalizou a JEP 512, trazendo compact source files e instance main methods. ([OpenJDK][8])

Isso reduz a cerimônia inicial para pequenos programas, mas não substitui a estrutura robusta usada em aplicações reais.

Uma comparação simples:

| Versão             | Recurso                                         |     JEP | Exemplo de uso                                       |
| ------------------ | ----------------------------------------------- | ------: | ---------------------------------------------------- |
| Java 8 ou anterior | Compilar e executar separadamente               |       — | `javac Programa.java` + `java Programa`              |
| Java 11            | Executar arquivo-fonte único                    | JEP 330 | `java Programa.java`                                 |
| Java 22            | Executar programas com múltiplos arquivos-fonte | JEP 458 | `java Programa.java` encontrando dependências locais |
| Java 25            | `main` de instância e arquivos-fonte compactos  | JEP 512 | `void main() { IO.println("..."); }`                 |

## Boas práticas

### Use nomes claros para classes

Prefira:

```java
ResumoCatalogoMusical
ImportadorDeBandas
GeradorRelatorioBandas
```

Evite:

```java
Teste
Programa
Classe1
Main2
```

Nomes claros ajudam na leitura, manutenção e busca dentro do projeto.

### Mantenha o nome do arquivo consistente com a classe pública

Se a classe é:

```java
public class ImportadorDeBandas
```

o arquivo deve ser:

```text
ImportadorDeBandas.java
```

Essa consistência evita erros de compilação e melhora a organização do projeto.

### Use pacotes em projetos reais

Para estudos muito pequenos, uma classe sem pacote pode ser suficiente.

Para aplicações reais, use pacotes:

```java
package com.meucatalogomusical.application;
```

Isso reduz conflitos de nome e melhora a arquitetura.

### Compile para uma pasta separada

Prefira:

```bash
javac -d out src/com/meucatalogomusical/console/ResumoCatalogoMusical.java
```

em vez de espalhar `.class` junto dos arquivos `.java`.

Em projetos reais, Maven e Gradle automatizam isso.

### Entenda o básico antes de depender apenas da IDE

IDEs como IntelliJ IDEA facilitam a execução, mas é importante saber o que acontece por baixo:

```bash
javac
java
classpath
bytecode
main
JVM
```

Esse conhecimento ajuda muito na investigação de problemas em CI/CD, Docker, servidores e ambientes produtivos.

### Use `java Arquivo.java` para experimentos, não como padrão produtivo

Executar direto o arquivo-fonte é excelente para aprendizado e scripts pequenos.

Para aplicações backend, prefira build estruturado com Maven ou Gradle, testes automatizados, empacotamento e versionamento adequado.

## Erros comuns

### Erro 1: nome do arquivo diferente da classe pública

Arquivo:

```text
Catalogo.java
```

Código:

```java
public class ResumoCatalogoMusical {
}
```

Resultado provável:

```text
class ResumoCatalogoMusical is public, should be declared in a file named ResumoCatalogoMusical.java
```

Correção:

```text
ResumoCatalogoMusical.java
```

### Erro 2: tentar executar usando `.class`

Incorreto:

```bash
java ResumoCatalogoMusical.class
```

Correto:

```bash
java ResumoCatalogoMusical
```

O comando `java` espera o nome da classe, não o nome do arquivo `.class`.

### Erro 3: esquecer o classpath

Se a classe foi compilada para a pasta `out`, isto pode falhar:

```bash
java com.meucatalogomusical.console.ResumoCatalogoMusical
```

Correção:

```bash
java -cp out com.meucatalogomusical.console.ResumoCatalogoMusical
```

### Erro 4: executar classe com pacote como se ela não tivesse pacote

Se a classe possui:

```java
package com.meucatalogomusical.console;
```

não execute assim:

```bash
java ResumoCatalogoMusical
```

Execute com o nome totalmente qualificado:

```bash
java -cp out com.meucatalogomusical.console.ResumoCatalogoMusical
```

### Erro 5: alterar a assinatura tradicional do main em versões anteriores

Em código tradicional, isto está correto:

```java
public static void main(String[] args)
```

Mas isto pode não funcionar em versões antigas:

```java
void main()
```

`void main()` é suportado como parte da evolução finalizada no Java 25 pela JEP 512. Em projetos que precisam rodar em Java 17 ou Java 21, prefira a assinatura tradicional.

### Erro 6: confundir JDK com JRE

Para compilar, você precisa do JDK, pois ele inclui ferramentas como `javac`.

Para apenas executar aplicações Java já compiladas, um runtime compatível pode ser suficiente.

Em ambientes de desenvolvimento backend, normalmente usamos JDK.

## Decisões de engenharia

### Quando usar a forma tradicional com `public static void main`

Use quando:

* estiver criando uma aplicação backend real;
* estiver criando um projeto com Maven ou Gradle;
* precisar gerar `.jar`;
* estiver trabalhando com Spring Boot;
* precisar compatibilidade com versões anteriores;
* quiser clareza explícita sobre classe, pacote e ponto de entrada.

Impactos:

```text
Legibilidade: alta para projetos profissionais.
Manutenção: alta, pois segue convenções amplamente conhecidas.
Performance: não há prejuízo relevante por usar a forma tradicional.
Compatibilidade: excelente com versões antigas e atuais.
```

### Quando usar `java Arquivo.java`

Use quando:

* quiser testar rapidamente uma ideia;
* estiver escrevendo um script simples;
* estiver estudando a linguagem;
* quiser evitar uma configuração inicial de build.

Evite quando:

* o projeto precisa de dependências externas;
* há muitos arquivos;
* existe pipeline de CI/CD;
* o código será empacotado e distribuído;
* a aplicação é um serviço backend real.

Impactos:

```text
Legibilidade: boa para código pequeno.
Manutenção: limitada em projetos maiores.
Performance: o custo de compilação pode aparecer a cada execução.
Distribuição: fraca para aplicações produtivas.
```

### Quando usar compact source files do Java 25

Use quando:

* estiver escrevendo programas pequenos;
* estiver aprendendo ou ensinando conceitos básicos;
* estiver criando utilitários simples;
* quiser reduzir cerimônia inicial.

Evite quando:

* o código representa um componente de aplicação grande;
* você precisa de arquitetura explícita;
* o projeto precisa suportar Java anterior ao 25;
* a equipe ainda padroniza `public static void main`.

Impactos:

```text
Legibilidade: ótima para exemplos pequenos.
Manutenção: boa apenas enquanto o programa permanecer pequeno.
Compatibilidade: exige Java 25.
Arquitetura: limitada para aplicações complexas.
```

### Alternativas

Em vez de executar manualmente com `javac` e `java`, aplicações reais costumam usar:

```bash
mvn test
mvn package
java -jar target/app.jar
```

ou:

```bash
./gradlew test
./gradlew build
java -jar build/libs/app.jar
```

A diferença é que Maven e Gradle organizam compilação, testes, dependências, empacotamento e plugins.

## Onde isso aparece em sistemas reais

### Em aplicações Spring Boot

Mesmo quando você usa Spring Boot, existe um ponto de entrada.

Normalmente ele se parece com:

```java
public class MeuCatalogoMusicalApplication {

    public static void main(String[] args) {
        // Inicialização da aplicação
    }
}
```

A diferença é que, em vez de imprimir algo no console, o método inicia o contexto da aplicação, registra beans, configura servidor web, carrega propriedades e prepara a aplicação para receber requisições.

### Em builds Maven e Gradle

Quando você executa:

```bash
mvn package
```

ou:

```bash
./gradlew build
```

por baixo dos panos ocorre compilação Java.

O compilador transforma `.java` em `.class`, e o build tool organiza esses arquivos em diretórios como:

```text
target/classes
build/classes
```

Depois, esses arquivos podem ser empacotados em um `.jar`.

### Em containers Docker

Em um container, a aplicação Java geralmente é iniciada com algo como:

```bash
java -jar app.jar
```

Nesse cenário, o comando `java` continua sendo responsável por iniciar a JVM.

A diferença é que o ponto de entrada pode estar declarado no manifesto do JAR, e não diretamente na linha de comando.

### Em observabilidade e troubleshooting

Entender JVM e bytecode ajuda a interpretar problemas como:

```text
UnsupportedClassVersionError
ClassNotFoundException
NoClassDefFoundError
NoSuchMethodError
OutOfMemoryError
```

Esses erros são muito comuns em ambientes reais e geralmente envolvem versão do Java, classpath, empacotamento, dependências ou incompatibilidade entre compilação e execução.

## Perguntas de revisão

### 1. Qual é a diferença entre arquivo `.java` e arquivo `.class`?

**Resposta:**
O arquivo `.java` contém o código-fonte escrito pelo desenvolvedor. É o arquivo legível por humanos.

O arquivo `.class` contém o bytecode gerado pelo compilador `javac`. Esse bytecode é a forma que a JVM consegue carregar, verificar e executar. A documentação do `javac` define que arquivos-fonte Java usam extensão `.java` e arquivos de classe usam extensão `.class`.

---

### 2. Qual é a responsabilidade do `javac`?

**Resposta:**
O `javac` é o compilador Java. Ele lê arquivos-fonte escritos na linguagem Java e os transforma em arquivos `.class`, que podem ser executados pela JVM.

Em outras palavras:

```text
ResumoCatalogoMusical.java  --javac-->  ResumoCatalogoMusical.class
```

O `javac` também verifica erros de sintaxe, tipos, acesso a classes, uso de APIs e outras regras da linguagem antes de gerar o bytecode.

---

### 3. Qual é a responsabilidade do comando `java`?

**Resposta:**
O comando `java` é o launcher da aplicação Java. Ele inicia a JVM, carrega a classe indicada e chama o método `main`.

Por exemplo:

```bash
java ResumoCatalogoMusical
```

Nesse comando, `ResumoCatalogoMusical` é o nome da classe que será carregada e executada pela JVM. A documentação oficial descreve que o comando `java` inicia a JVM, carrega a classe especificada e chama o método `main`.

---

### 4. O que é a JVM?

**Resposta:**
A JVM, Java Virtual Machine, é a máquina virtual responsável por executar programas Java compilados em bytecode.

Ela não executa diretamente o arquivo `.java`. Ela executa o conteúdo do arquivo `.class`.

A JVM cuida de tarefas como:

* carregar classes;
* verificar bytecode;
* gerenciar memória;
* executar instruções;
* controlar pilha de chamadas;
* lidar com erros em tempo de execução;
* aplicar otimizações durante a execução.

A especificação oficial define a JVM como uma máquina abstrata que possui um conjunto de instruções e manipula áreas de memória em tempo de execução.

---

### 5. Por que, ao executar uma classe, usamos `java NomeDaClasse` e não `java NomeDaClasse.class`?

**Resposta:**
Porque o comando `java` espera o nome da classe, não o nome do arquivo físico.

Quando executamos:

```bash
java ResumoCatalogoMusical
```

estamos dizendo ao launcher:

```text
Procure, carregue e execute a classe ResumoCatalogoMusical.
```

O arquivo `ResumoCatalogoMusical.class` existe no sistema de arquivos, mas o comando `java` trabalha com o nome da classe que será carregada pela JVM. A documentação oficial explica que, por padrão, o primeiro argumento que não é uma opção é o nome totalmente qualificado da classe a ser chamada.

---

### 6. O que significa `public static void main(String[] args)`?

**Resposta:**
Esse é o ponto de entrada tradicional de uma aplicação Java.

Cada parte tem uma função:

```java
public static void main(String[] args)
```

* `public`: permite que o método seja acessado pelo launcher Java;
* `static`: permite chamar o método sem criar um objeto manualmente;
* `void`: indica que o método não retorna valor;
* `main`: é o nome reconhecido como ponto de entrada;
* `String[] args`: representa os argumentos recebidos pela linha de comando.

Em uma aplicação tradicional, quando o comando `java` executa uma classe, ele procura e chama esse método `main`.

---

### 7. Por que o nome do arquivo deve acompanhar o nome da classe pública?

**Resposta:**
Porque, em Java, quando uma classe pública está em um arquivo-fonte, o nome do arquivo deve corresponder ao nome dessa classe.

Exemplo correto:

```java
public class ResumoCatalogoMusical {
}
```

Arquivo:

```text
ResumoCatalogoMusical.java
```

Exemplo incorreto:

```text
Catalogo.java
```

com uma classe pública chamada:

```java
public class ResumoCatalogoMusical {
}
```

Isso gera erro de compilação, porque a organização do arquivo não corresponde à declaração pública da classe. A JLS permite que implementações baseadas em sistema de arquivos exijam essa correspondência entre nome do tipo e nome do arquivo.

---

### 8. O que acontece quando uma classe está dentro de um pacote?

**Resposta:**
Quando uma classe declara um pacote, ela passa a pertencer a um namespace específico.

Exemplo:

```java
package com.meucatalogomusical.console;

public class ResumoCatalogoMusical {
}
```

Nesse caso, o nome completo da classe não é apenas:

```text
ResumoCatalogoMusical
```

É:

```text
com.meucatalogomusical.console.ResumoCatalogoMusical
```

Por isso, para executar a classe compilada, usamos o nome totalmente qualificado:

```bash
java -cp out com.meucatalogomusical.console.ResumoCatalogoMusical
```

A JLS define que a declaração `package` indica a qual pacote uma unidade de compilação pertence. ([Oracle Docs][4])

---

### 9. Qual é a diferença entre executar `java Programa.java` e executar `javac Programa.java` seguido de `java Programa`?

**Resposta:**
Ao executar:

```bash
javac Programa.java
java Programa
```

você faz o processo em duas etapas explícitas:

1. compila o arquivo `.java` e gera o `.class`;
2. executa a classe compilada.

Ao executar:

```bash
java Programa.java
```

você usa o modo de execução direta de arquivo-fonte. Nesse caso, o Java compila e executa o programa em uma única etapa.

Esse recurso é útil para estudos, testes rápidos e pequenos scripts, mas em aplicações reais o mais comum é compilar, testar, empacotar e distribuir a aplicação com ferramentas como Maven ou Gradle. A documentação do comando `java` explica que, quando o argumento identifica um arquivo `.java`, o modo source-file é selecionado e o arquivo é compilado e executado.

---

### 10. Em quais cenários compact source files do Java 25 são úteis?

**Resposta:**
Compact source files, recurso finalizado no Java 25 pela JEP 512, são úteis para programas pequenos, estudos, exemplos didáticos, scripts simples e utilitários de linha de comando.

Com Java 25, é possível escrever algo como:

```java
void main() {
    IO.println("Catálogo musical verificado.");
}
```

Esse formato reduz a cerimônia inicial da linguagem, porque não obriga o iniciante ou o autor de um script simples a escrever imediatamente `public class`, `public static void main` e outros elementos mais ligados a programas maiores.

Mesmo assim, para aplicações backend profissionais, a forma tradicional com classes, pacotes, build tool, testes e empacotamento continua sendo mais adequada. A JEP 512 deixa claro que o objetivo é ajudar na escrita de programas pequenos e permitir evolução gradual, sem criar um dialeto separado da linguagem Java.

---

### 11. Por que aplicações backend reais normalmente usam Maven ou Gradle em vez de compilar manualmente com `javac`?

**Resposta:**
Porque aplicações backend reais geralmente têm muitas classes, pacotes, dependências externas, testes, profiles, plugins, empacotamento, versionamento e integração com pipelines de CI/CD.

Compilar manualmente com `javac` funciona para exemplos pequenos, mas fica difícil de manter em sistemas maiores.

Maven e Gradle ajudam a automatizar tarefas como:

* baixar dependências;
* compilar o projeto;
* executar testes;
* gerar `.jar`;
* separar código principal e código de teste;
* configurar plugins;
* padronizar o build entre máquinas e ambientes;
* integrar com Docker, CI/CD e deploy.

Ou seja, `javac` continua existindo por baixo, mas Maven e Gradle coordenam o processo completo de build.

---

### 12. Como o conhecimento de JVM ajuda a resolver problemas em produção?

**Resposta:**
Conhecer JVM ajuda porque muitos problemas de produção em Java não estão apenas no código-fonte, mas também no modo como a aplicação foi compilada, empacotada, carregada e executada.

Esse conhecimento ajuda a investigar problemas como:

```text
ClassNotFoundException
NoClassDefFoundError
UnsupportedClassVersionError
OutOfMemoryError
StackOverflowError
NoSuchMethodError
```

Por exemplo:

* `UnsupportedClassVersionError` pode indicar que o código foi compilado em uma versão mais nova do Java do que a versão usada para executar;
* `ClassNotFoundException` pode indicar problema de classpath ou dependência ausente;
* `OutOfMemoryError` pode indicar consumo excessivo de heap;
* `NoSuchMethodError` pode indicar incompatibilidade entre versões de bibliotecas.

A especificação da JVM mostra que arquivos `.class` possuem versão própria, e no Java SE 25 a versão principal mais recente suportada é a 69. Isso explica por que bytecodes gerados por versões diferentes do Java podem causar problemas quando executados em uma JVM incompatível.

## Resumo final

Um programa Java começa no código-fonte, geralmente em um arquivo `.java`.

O compilador `javac` transforma esse código em bytecode, normalmente em arquivos `.class`.

A JVM executa esse bytecode.

O comando `java` é o launcher usado para iniciar a JVM, carregar uma classe e chamar o método `main`.

A forma tradicional:

```java
public static void main(String[] args)
```

continua sendo essencial para entender aplicações Java reais.

O Java 11 introduziu a execução direta de arquivos-fonte únicos com a JEP 330.

O Java 22 expandiu esse modelo para múltiplos arquivos com a JEP 458.

O Java 25 finalizou compact source files e instance main methods com a JEP 512, reduzindo a cerimônia para programas pequenos.

Para estudos, scripts e experimentos, os recursos modernos do Java 25 são muito úteis.

Para aplicações backend profissionais, é importante dominar também a estrutura tradicional com classes, pacotes, compilação, classpath, JARs, build tools e JVM.

## Referências

* Java SE 25 & JDK 25 — `javac` Command. ([Oracle Docs][1])
* Java SE 25 & JDK 25 — `java` Command. ([Oracle Docs][2])
* Java Language Specification, Java SE 25 Edition. ([Oracle Docs][12])
* Java Virtual Machine Specification, Java SE 25 Edition. ([Oracle Docs][3])
* Java SE 25 API — `java.lang.System`. ([Oracle Docs][6])
* Java SE 25 API — `java.lang.IO`. ([Oracle Docs][9])
* OpenJDK JEP 330 — Launch Single-File Source-Code Programs. ([OpenJDK][10])
* OpenJDK JEP 458 — Launch Multi-File Source-Code Programs. ([OpenJDK][11])
* OpenJDK JEP 512 — Compact Source Files and Instance Main Methods. ([OpenJDK][8])
* Effective Java, Joshua Bloch.
* Java Concurrency in Practice, Brian Goetz.

[1]: https://docs.oracle.com/en/java/javase/25/docs/specs/man/javac.html "The javac Command"
[2]: https://docs.oracle.com/en/java/javase/25/docs/specs/man/java.html "The java Command"
[3]: https://docs.oracle.com/javase/specs/jvms/se25/html/index.html "The Java® Virtual Machine Specification"
[4]: https://docs.oracle.com/javase/specs/jls/se25/html/jls-8.html "Chapter 8. Classes"
[5]: https://docs.oracle.com/javase/specs/jls/se25/html/jls-7.html "Chapter 7. Packages and Modules"
[6]: https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/System.html "System (Java SE 25 & JDK 25)"
[7]: https://docs.oracle.com/javase/specs/jls/se25/html/jls-12.html "Chapter 12. Execution"
[8]: https://openjdk.org/jeps/512 "JEP 512: Compact Source Files and Instance Main Methods"
[9]: https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/IO.html "IO (Java SE 25 & JDK 25)"
[10]: https://openjdk.org/jeps/330 "JEP 330: Launch Single-File Source-Code Programs"
[11]: https://openjdk.org/jeps/458 "JEP 458: Launch Multi-File Source-Code Programs"
[12]: https://docs.oracle.com/javase/specs/jls/se25/html/index.html "The Java® Language Specification"
