# Classes, objetos e métodos em Java

Em um sistema backend real, não basta armazenar dados. Um sistema precisa representar conceitos do domínio e executar comportamentos relacionados a esses conceitos.

No `meucatalogomusical.com`, por exemplo, não queremos apenas guardar textos como nome da banda, título da música, duração e quantidade de reproduções. Também queremos responder perguntas e executar ações:

* uma música é longa?
* qual é a duração formatada da faixa?
* a música deve entrar em uma lista de destaques?
* como gerar um resumo da música para exibir na API?
* como evitar repetir a mesma regra de pontuação em várias partes do sistema?

Classes, objetos e métodos ajudam a organizar esse tipo de código. A classe define um tipo do domínio. O objeto representa uma ocorrência concreta desse tipo em memória. O método concentra um comportamento que pertence àquele tipo ou que opera sobre ele.

Sem métodos, o código tende a ficar espalhado em vários arquivos, com cálculos duplicados, regras repetidas e nomes pouco expressivos. Com métodos bem definidos, a lógica ganha nome, lugar e responsabilidade.

## Antes de entrar no código: uma ideia simples

Uma classe pode ser vista como a definição de um conceito do sistema.

No domínio de um catálogo musical, podemos ter conceitos como:

* `Banda`;
* `Musica`;
* `ResumoMusica`;
* `CuradoriaCatalogo`.

Um objeto é uma instância concreta de uma classe.

Por exemplo:

* uma banda chamada `Dream Theater`;
* uma música chamada `Pull Me Under`;
* um resumo gerado para essa música;
* uma curadoria analisando se a música merece destaque.

Métodos são comportamentos nomeados. Eles dizem o que um objeto sabe fazer ou calcular.

Por exemplo, uma música pode:

* registrar uma nova reprodução;
* calcular sua duração em minutos;
* informar se é uma faixa longa;
* gerar um resumo;
* calcular uma pontuação interna para o catálogo.

A ideia central da orientação a objetos é aproximar dados e comportamentos relacionados. Se uma regra depende diretamente dos dados de uma música, faz sentido considerar se essa regra deveria estar perto da própria classe `Musica`.

## Definição técnica

Em Java, uma classe declara um tipo de referência. Ela pode conter membros, como campos e métodos.

Um campo representa estado. No exemplo deste artigo, `titulo`, `duracaoEmSegundos` e `totalReproducoes` são partes do estado de uma música.

Um método representa comportamento. Ele tem nome, tipo de retorno, lista de parâmetros e corpo.

De forma simplificada:

```java
tipoDeRetorno nomeDoMetodo(parametros) {
    corpoDoMetodo
}
```

Exemplo:

```java
boolean ehFaixaLonga() {
    return duracaoEmSegundos >= 420;
}
```

Nesse método:

* `boolean` é o tipo de retorno;
* `ehFaixaLonga` é o nome do método;
* `()` indica que o método não recebe parâmetros;
* o bloco `{ ... }` contém a implementação;
* `return` devolve o resultado para quem chamou o método.

Quando um método não precisa devolver valor, usamos `void`:

```java
void registrarReproducao() {
    totalReproducoes++;
}
```

Um método `void` não retorna um valor para ser usado em uma variável, mas ainda assim pode alterar o estado do objeto, imprimir algo, validar uma condição ou acionar outro comportamento.

## Como funciona em Java

Em Java, os métodos mais comuns em classes de domínio são métodos de instância. Isso significa que eles são chamados a partir de um objeto.

Exemplo:

```java
musica.registrarReproducao();
```

Nesse caso, `registrarReproducao` é executado sobre o objeto referenciado pela variável `musica`.

Cada objeto possui seu próprio estado. Dois objetos da mesma classe compartilham a mesma estrutura definida pela classe, mas podem ter valores diferentes nos campos.

Exemplo conceitual:

```java
Musica musica1 = new Musica();
musica1.titulo = "Pull Me Under";
musica1.totalReproducoes = 850;

Musica musica2 = new Musica();
musica2.titulo = "Fear of the Dark";
musica2.totalReproducoes = 1_500;
```

Ambas são objetos da classe `Musica`, mas cada uma tem seus próprios valores.

Quando chamamos:

```java
musica1.registrarReproducao();
musica2.registrarReproducao();
```

o mesmo método é executado, mas sobre objetos diferentes. Por isso, o resultado altera o estado de cada objeto separadamente.

Também é importante diferenciar parâmetros e argumentos.

Parâmetro é a variável declarada no método:

```java
boolean atingiuMinimoDeReproducoes(int minimoReproducoes) {
    return totalReproducoes >= minimoReproducoes;
}
```

Argumento é o valor passado na chamada:

```java
musica.atingiuMinimoDeReproducoes(1_000);
```

Nesse exemplo, `minimoReproducoes` é o parâmetro. O valor `1_000` é o argumento.

## Exemplo prático

Imagine que o `meucatalogomusical.com` precisa exibir músicas de bandas de Metal, Hard Rock e Rock Progressivo. Além de armazenar dados, o sistema precisa calcular pontuação, gerar resumos e decidir se uma faixa merece destaque.

### Banda.java

```java
public class Banda {

    String nome;
    String estilo;
    String paisOrigem;

}
```

A classe `Banda` representa uma banda cadastrada no catálogo.

Neste momento, ela contém apenas estado: nome, estilo e país de origem. Por enquanto não vamos falar de encapsulamento, construtores, getters ou setters, porque esses assuntos merecem estudo próprio.

### ResumoMusica.java

```java
public class ResumoMusica {

    String descricao;
    int duracaoEmSegundos;
    boolean faixaLonga;
    double pontuacaoCatalogo;

    boolean mereceDestaque() {
        return pontuacaoCatalogo >= 80;
    }

}
```

A classe `ResumoMusica` representa um objeto de retorno. Ela agrupa várias informações calculadas sobre uma música.

Isso é útil porque Java não retorna múltiplos valores separados com algo como:

```java
return descricao, duracaoEmSegundos, pontuacaoCatalogo;
```

Esse código não compila. Quando um método precisa devolver um conjunto de informações relacionadas, uma alternativa é criar uma classe que represente esse resultado.

O método `mereceDestaque` retorna um `boolean`. Em métodos booleanos, o nome pode ser escrito como uma pergunta sem ponto de interrogação:

```java
boolean mereceDestaque()
boolean ehFaixaLonga()
boolean atingiuMinimoDeReproducoes()
```

Isso deixa o código de chamada mais expressivo:

```java
if (resumo.mereceDestaque()) {
    System.out.println("Adicionar aos destaques");
}
```

### Musica.java

```java
public class Musica {

    String titulo;
    Banda banda;
    int duracaoEmSegundos;
    int totalReproducoes;
    double notaMedia;

    void registrarReproducao() {
        totalReproducoes++;
    }

    int calcularDuracaoEmMinutos() {
        return duracaoEmSegundos / 60;
    }

    int calcularSegundosRestantesDoMinuto() {
        return duracaoEmSegundos % 60;
    }

    boolean ehFaixaLonga() {
        return duracaoEmSegundos >= 420;
    }

    boolean atingiuMinimoDeReproducoes(int minimoReproducoes) {
        return totalReproducoes >= minimoReproducoes;
    }

    double calcularPontuacaoCatalogo() {
        double pontosPorNota = notaMedia * 20;
        double pontosPorReproducao = calcularPontosPorReproducao();
        int bonusDuracao = calcularBonusDuracao();

        return pontosPorNota + pontosPorReproducao + bonusDuracao;
    }

    double calcularPontosPorReproducao() {
        if (totalReproducoes > 1_000) {
            return 100;
        }

        return totalReproducoes / 10.0;
    }

    int calcularBonusDuracao() {
        int bonus = 0;

        if (ehFaixaLonga()) {
            bonus = 10;
        }

        return bonus;
    }

    ResumoMusica gerarResumo() {
        ResumoMusica resumo = new ResumoMusica();
        resumo.descricao = montarDescricao();
        resumo.duracaoEmSegundos = duracaoEmSegundos;
        resumo.faixaLonga = ehFaixaLonga();
        resumo.pontuacaoCatalogo = calcularPontuacaoCatalogo();

        return resumo;
    }

    String montarDescricao() {
        int minutos = calcularDuracaoEmMinutos();
        int segundos = calcularSegundosRestantesDoMinuto();

        return String.format("%s - %s (%d:%02d)",
                banda.nome, titulo, minutos, segundos);
    }

    void imprimirFichaTecnica() {
        if (titulo == null || banda == null) {
            System.out.println("Ficha técnica incompleta.");
            return;
        }

        ResumoMusica resumo = gerarResumo();

        System.out.printf("Música: %s%n", resumo.descricao);
        System.out.printf("Pontuação no catálogo: %.2f%n", resumo.pontuacaoCatalogo);
        System.out.printf("Faixa longa: %s%n", resumo.faixaLonga ? "sim" : "não");
    }

}
```

A classe `Musica` é mais rica que uma simples estrutura de dados. Ela não apenas armazena informações, mas também concentra comportamentos relacionados à música.

Ela sabe:

* registrar uma reprodução;
* calcular duração;
* identificar se é uma faixa longa;
* verificar se atingiu um mínimo de reproduções;
* calcular uma pontuação;
* gerar um resumo;
* imprimir uma ficha técnica simples.

Essa aproximação entre estado e comportamento é uma das ideias centrais da orientação a objetos.

### CuradoriaCatalogo.java

```java
public class CuradoriaCatalogo {

    boolean deveEntrarEmDestaques(Musica musica, int minimoReproducoes) {
        if (musica == null) {
            return false;
        }

        return musica.atingiuMinimoDeReproducoes(minimoReproducoes)
                && musica.calcularPontuacaoCatalogo() >= 80;
    }

    ResumoMusica analisar(Musica musica) {
        return musica.gerarResumo();
    }

}
```

A classe `CuradoriaCatalogo` representa uma regra de curadoria do catálogo.

A decisão de criar uma classe separada faz sentido quando a regra não pertence exclusivamente à música, mas ao processo de curadoria do sistema.

A música sabe calcular sua própria pontuação. A curadoria decide se essa pontuação, junto com outras regras, é suficiente para colocar a faixa em destaque.

Essa separação melhora a coesão:

* `Musica` cuida de comportamentos diretamente ligados à música;
* `CuradoriaCatalogo` cuida de decisões de curadoria.

### Aplicacao.java

```java
public class Aplicacao {

    public static void main(String[] args) {
        Banda dreamTheater = new Banda();
        dreamTheater.nome = "Dream Theater";
        dreamTheater.estilo = "Metal Progressivo";
        dreamTheater.paisOrigem = "Estados Unidos";

        Musica pullMeUnder = new Musica();
        pullMeUnder.titulo = "Pull Me Under";
        pullMeUnder.banda = dreamTheater;
        pullMeUnder.duracaoEmSegundos = 496;
        pullMeUnder.totalReproducoes = 850;
        pullMeUnder.notaMedia = 4.7;

        pullMeUnder.registrarReproducao();
        pullMeUnder.imprimirFichaTecnica();

        CuradoriaCatalogo curadoria = new CuradoriaCatalogo();
        ResumoMusica resumo = curadoria.analisar(pullMeUnder);

        if (curadoria.deveEntrarEmDestaques(pullMeUnder, 800)) {
            System.out.printf("%s entrou nos destaques do catálogo.%n",
                    resumo.descricao);
        }
    }

}
```

A classe `Aplicacao` apenas monta os objetos e executa o exemplo.

O ponto principal está nas chamadas:

```java
pullMeUnder.registrarReproducao();
pullMeUnder.imprimirFichaTecnica();
curadoria.analisar(pullMeUnder);
curadoria.deveEntrarEmDestaques(pullMeUnder, 800);
```

Essas chamadas mostram quatro ideias importantes:

1. um método pode alterar o estado do objeto;
2. um método pode executar uma ação sem retornar valor;
3. um método pode retornar outro objeto;
4. um método pode receber um objeto como argumento.

## Analisando o código

### Classe como modelo de um conceito

A classe `Musica` define o que uma música possui e o que uma música pode fazer dentro do sistema.

```java
public class Musica {

    String titulo;
    Banda banda;
    int duracaoEmSegundos;
    int totalReproducoes;
    double notaMedia;

}
```

Esses campos representam o estado da música.

Quando criamos um objeto com `new Musica()`, passamos a ter uma música concreta em memória.

```java
Musica pullMeUnder = new Musica();
```

A variável `pullMeUnder` não é o objeto em si. Ela guarda uma referência para o objeto.

### Método void

O método abaixo não retorna valor:

```java
void registrarReproducao() {
    totalReproducoes++;
}
```

Mesmo sendo `void`, ele faz algo relevante: altera o estado da música, incrementando `totalReproducoes`.

Um erro comum é pensar que método `void` “não faz nada”. Na verdade, `void` apenas significa que o método não devolve um valor para quem o chamou.

### Método com retorno

O método abaixo retorna um `boolean`:

```java
boolean ehFaixaLonga() {
    return duracaoEmSegundos >= 420;
}
```

Quem chama esse método pode usar o resultado diretamente em uma condição:

```java
if (pullMeUnder.ehFaixaLonga()) {
    System.out.println("Faixa longa");
}
```

O método abaixo retorna um `double`:

```java
double calcularPontuacaoCatalogo() {
    double pontosPorNota = notaMedia * 20;
    double pontosPorReproducao = calcularPontosPorReproducao();
    int bonusDuracao = calcularBonusDuracao();

    return pontosPorNota + pontosPorReproducao + bonusDuracao;
}
```

Esse método não imprime a pontuação. Ele calcula e devolve o valor.

Essa diferença é muito importante. Um método chamado `calcularPontuacaoCatalogo` deve calcular. Se ele apenas imprimisse o resultado, o nome ficaria enganoso e o valor não poderia ser reutilizado por outras partes do sistema.

### Métodos menores evitam duplicação

Observe que `calcularPontuacaoCatalogo` não concentra toda a lógica em um único bloco enorme.

Ele delega partes menores para outros métodos:

```java
double pontosPorReproducao = calcularPontosPorReproducao();
int bonusDuracao = calcularBonusDuracao();
```

Isso evita duplicação e melhora a leitura.

Se a regra de bônus por duração mudar, o ajuste fica centralizado em:

```java
int calcularBonusDuracao() {
    int bonus = 0;

    if (ehFaixaLonga()) {
        bonus = 10;
    }

    return bonus;
}
```

Sem essa separação, a mesma regra poderia aparecer repetida em tela, API, relatório, job de recomendação e teste automatizado.

Código duplicado costuma parecer inofensivo no começo, mas vira um problema sério quando a regra muda.

### Return encerra a execução do método

No método abaixo, o `return` é usado para sair cedo de um método `void`:

```java
void imprimirFichaTecnica() {
    if (titulo == null || banda == null) {
        System.out.println("Ficha técnica incompleta.");
        return;
    }

    ResumoMusica resumo = gerarResumo();

    System.out.printf("Música: %s%n", resumo.descricao);
    System.out.printf("Pontuação no catálogo: %.2f%n", resumo.pontuacaoCatalogo);
    System.out.printf("Faixa longa: %s%n", resumo.faixaLonga ? "sim" : "não");
}
```

Quando `titulo` ou `banda` estão ausentes, o método imprime uma mensagem e encerra. As linhas abaixo do `return` não são executadas.

Em métodos que retornam valor, o `return` também encerra o método e devolve o resultado:

```java
double calcularPontosPorReproducao() {
    if (totalReproducoes > 1_000) {
        return 100;
    }

    return totalReproducoes / 10.0;
}
```

Em Java, código colocado depois de um `return` incondicional não é executável. Em muitos casos, isso gera erro de compilação por código inalcançável.

### Um ou vários returns?

Existe uma recomendação tradicional de manter um único ponto de saída no método. Ela pode ajudar em métodos complexos, especialmente quando há muitos caminhos de execução.

Mas isso não deve ser tratado como uma regra absoluta.

Em Java moderno, retornos antecipados podem melhorar a legibilidade quando usados como cláusulas de guarda:

```java
if (musica == null) {
    return false;
}
```

O problema não é ter mais de um `return`. O problema é ter muitos retornos espalhados em um método grande e difícil de entender.

Uma boa regra prática:

* use retorno único quando isso deixar o fluxo mais simples;
* use retorno antecipado quando isso eliminar aninhamento desnecessário;
* evite métodos grandes com muitos caminhos de saída.

### Métodos que retornam objetos

O método abaixo retorna um objeto do tipo `ResumoMusica`:

```java
ResumoMusica gerarResumo() {
    ResumoMusica resumo = new ResumoMusica();
    resumo.descricao = montarDescricao();
    resumo.duracaoEmSegundos = duracaoEmSegundos;
    resumo.faixaLonga = ehFaixaLonga();
    resumo.pontuacaoCatalogo = calcularPontuacaoCatalogo();

    return resumo;
}
```

Esse método é útil porque agrupa dados relacionados em um objeto com significado no domínio.

Em vez de retornar vários valores soltos, ele retorna um resultado nomeado: `ResumoMusica`.

Isso melhora a clareza do código:

```java
ResumoMusica resumo = pullMeUnder.gerarResumo();
```

A variável `resumo` agora representa uma resposta completa da análise da música.

### Métodos com argumentos

O método abaixo recebe um valor como argumento:

```java
boolean atingiuMinimoDeReproducoes(int minimoReproducoes) {
    return totalReproducoes >= minimoReproducoes;
}
```

Esse método é melhor do que fixar o número diretamente dentro da classe, porque permite variar o critério conforme o contexto.

Exemplo:

```java
pullMeUnder.atingiuMinimoDeReproducoes(800);
pullMeUnder.atingiuMinimoDeReproducoes(1_000);
```

O mesmo método pode ser usado com limites diferentes.

### Passando objetos como argumentos

A classe `CuradoriaCatalogo` recebe uma `Musica` como argumento:

```java
boolean deveEntrarEmDestaques(Musica musica, int minimoReproducoes) {
    if (musica == null) {
        return false;
    }

    return musica.atingiuMinimoDeReproducoes(minimoReproducoes)
            && musica.calcularPontuacaoCatalogo() >= 80;
}
```

Passar um objeto como argumento costuma ser melhor do que passar muitos valores soltos.

Em vez de criar um método assim:

```java
boolean deveEntrarEmDestaques(String titulo,
        int duracaoEmSegundos,
        int totalReproducoes,
        double notaMedia) {
    // lógica
}
```

podemos passar o objeto que representa a música:

```java
boolean deveEntrarEmDestaques(Musica musica, int minimoReproducoes) {
    // lógica
}
```

Isso melhora a leitura e reduz o risco de trocar a ordem dos argumentos.

### Comparação com versões anteriores do Java

O modelo básico de classes, objetos, métodos, parâmetros, argumentos e `return` é parte fundamental da linguagem Java desde suas primeiras versões. O que mudou ao longo do tempo foi a quantidade de recursos disponíveis ao redor desse modelo.

No Java 10, a JEP 286 introduziu `var` para inferência de tipo em variáveis locais. Isso permite escrever:

```java
var resumo = pullMeUnder.gerarResumo();
```

Mesmo assim, neste artigo os exemplos evitam `var` de propósito. Para estudo de fundamentos, escrever o tipo explicitamente ajuda a enxergar melhor o que cada método retorna.

No Java 16, a JEP 395 finalizou os records. Um `record` pode ser útil para representar agregados simples de dados. Por exemplo, em uma versão mais avançada do código, `ResumoMusica` poderia ser avaliado como candidato a `record`, porque representa um resultado com dados agrupados. Ainda assim, records não substituem classes comuns quando o objetivo é modelar comportamento rico, estado mutável ou regras que evoluem.

No Java 25, a JEP 512 finalizou compact source files e instance main methods. Esse recurso facilita programas pequenos, estudos e scripts. Em sistemas backend profissionais, porém, ainda é comum organizar o código em classes explícitas, pacotes, camadas e arquivos separados.

Portanto, mesmo usando Java 25, o fundamento continua o mesmo: boas classes e bons métodos são essenciais para escrever código orientado a objetos legível e sustentável.

## O que acontece por baixo dos panos

### Objetos são acessados por referência

Quando escrevemos:

```java
Musica pullMeUnder = new Musica();
```

o objeto é criado em memória, e a variável `pullMeUnder` passa a guardar uma referência para esse objeto.

Quando chamamos:

```java
pullMeUnder.registrarReproducao();
```

a JVM executa o método sobre o objeto referenciado por `pullMeUnder`.

Isso explica por que dois objetos da mesma classe podem ter comportamentos iguais, mas resultados diferentes. O método é o mesmo, mas o estado consultado é o estado de cada objeto.

### Variáveis locais vivem durante a execução do método

Dentro de um método, podemos declarar variáveis locais:

```java
double pontosPorNota = notaMedia * 20;
```

Essa variável existe apenas dentro da execução daquele método. Ela não faz parte do estado do objeto.

Campos pertencem ao objeto:

```java
double notaMedia;
```

Variáveis locais pertencem à execução do método:

```java
double pontosPorNota = notaMedia * 20;
```

Essa diferença é essencial para entender escopo, memória e manutenção.

### Parâmetros recebem cópias dos valores dos argumentos

Quando um método recebe um `int`, o parâmetro recebe uma cópia daquele valor.

Quando um método recebe um objeto, o parâmetro recebe uma cópia da referência para o objeto.

Isso significa que Java não passa objetos “por referência” no sentido clássico. Java passa valores. No caso de objetos, o valor passado é uma referência.

Exemplo:

```java
ResumoMusica analisar(Musica musica) {
    return musica.gerarResumo();
}
```

O parâmetro `musica` recebe uma cópia da referência para o mesmo objeto usado na chamada. Por isso, o método consegue acessar os dados e comportamentos daquele objeto.

Se o método alterar o estado do objeto, quem chamou perceberá a alteração, porque ambos apontam para o mesmo objeto. Mas se o método apenas fizer o parâmetro apontar para outro objeto, isso não troca a variável original de quem chamou.

Esse detalhe é uma fonte comum de confusão em Java.

### Retornar objeto retorna uma referência

Quando um método faz:

```java
return resumo;
```

ele retorna uma referência para o objeto `ResumoMusica` criado dentro do método.

O objeto não é “desmontado” em vários valores. Também não é automaticamente copiado em uma nova estrutura independente. Quem recebe o retorno passa a ter uma referência para aquele objeto.

### Métodos pequenos podem ser otimizados

Uma preocupação comum é pensar que muitos métodos pequenos deixam o programa mais lento.

Na prática, em aplicações Java modernas, a JVM e o compilador JIT podem otimizar muitas chamadas de métodos durante a execução, inclusive aplicando inlining em cenários apropriados.

Isso não significa que performance nunca importa. Significa que, na maior parte dos códigos de negócio, clareza e coesão devem vir antes de micro-otimizações prematuras.

Métodos pequenos, bem nomeados e coesos geralmente melhoram a manutenção sem causar problema real de performance.

## Boas práticas

### Dê nomes de métodos que expressem intenção

Um método deve dizer o que faz.

Bons exemplos:

```java
calcularPontuacaoCatalogo()
gerarResumo()
registrarReproducao()
atingiuMinimoDeReproducoes()
```

Nomes ruins:

```java
processar()
executar()
fazer()
calcular()
```

Nomes genéricos demais escondem a intenção e obrigam o leitor a abrir a implementação para entender o comportamento.

### Use verbos para ações

Métodos que executam ações costumam começar com verbos:

```java
registrarReproducao()
gerarResumo()
calcularPontuacaoCatalogo()
imprimirFichaTecnica()
```

Métodos booleanos podem ser escritos como perguntas:

```java
ehFaixaLonga()
mereceDestaque()
atingiuMinimoDeReproducoes()
```

### Separe cálculo de apresentação

Um método chamado `calcularPontuacaoCatalogo` deve calcular e retornar a pontuação.

Evite misturar cálculo com impressão:

```java
double calcularPontuacaoCatalogo() {
    return pontosPorNota + pontosPorReproducao + bonusDuracao;
}
```

É melhor do que:

```java
void calcularPontuacaoCatalogo() {
    System.out.println(pontosPorNota + pontosPorReproducao + bonusDuracao);
}
```

O segundo exemplo calcula, mas prende o resultado à saída no console. Isso dificulta reaproveitar a pontuação em uma API, teste, relatório ou regra de curadoria.

### Extraia métodos para eliminar duplicação

Se uma regra aparece em dois lugares, considere centralizá-la.

Exemplo:

```java
boolean ehFaixaLonga() {
    return duracaoEmSegundos >= 420;
}
```

Agora qualquer parte da classe pode usar:

```java
if (ehFaixaLonga()) {
    // regra
}
```

Se o critério de faixa longa mudar de 7 para 8 minutos, a alteração acontece em um único lugar.

### Prefira poucos parâmetros

Métodos com muitos parâmetros ficam difíceis de ler e fáceis de chamar errado.

Ruim:

```java
boolean deveEntrarEmDestaques(String titulo,
        int duracaoEmSegundos,
        int totalReproducoes,
        double notaMedia,
        int minimoReproducoes) {
    // lógica
}
```

Melhor:

```java
boolean deveEntrarEmDestaques(Musica musica, int minimoReproducoes) {
    // lógica
}
```

Passar um objeto pode deixar o método mais expressivo quando os dados pertencem ao mesmo conceito.

### Retorne objetos quando o resultado tiver significado próprio

Se um método precisa retornar várias informações relacionadas, crie uma classe de retorno com nome claro.

Bom:

```java
ResumoMusica gerarResumo() {
    // lógica
}
```

Ruim:

```java
String gerarResumoComPontuacaoEDuracaoEIndicador() {
    // lógica misturada em texto
}
```

Um objeto de retorno permite evoluir o resultado sem transformar tudo em texto ou em valores soltos.

### Mantenha a classe coesa

Uma classe coesa tem responsabilidades relacionadas.

`Musica` pode calcular sua duração e pontuação.

Mas talvez não deva decidir sozinha todas as regras de curadoria da página inicial, campanhas, recomendações e destaques editoriais. Essas regras podem pertencer a classes específicas, como `CuradoriaCatalogo`.

Coesão evita classes que sabem demais e fazem coisas demais.

## Erros comuns

### Colocar toda a lógica no método main

O método de entrada deve apenas iniciar o fluxo do exemplo ou da aplicação.

Quando toda a lógica fica nele, o código perde nomes de domínio, fica difícil de testar e tende a duplicar regras.

Ruim:

```java
public static void main(String[] args) {
    double pontos = notaMedia * 20 + totalReproducoes / 10.0;
    // muitas regras aqui
}
```

Melhor:

```java
double pontos = musica.calcularPontuacaoCatalogo();
```

### Criar métodos que dizem uma coisa e fazem outra

Um método chamado `calcularPontuacaoCatalogo` deve retornar a pontuação, não apenas imprimir.

Um método chamado `gerarResumo` deve gerar e devolver um resumo, não alterar várias partes do sistema sem deixar isso claro.

Nome enganoso é um problema de manutenção.

### Duplicar cálculo em vários lugares

Se a duração formatada é calculada em várias telas, relatórios e serviços, qualquer mudança exigirá várias alterações.

Centralize:

```java
String montarDescricao() {
    int minutos = calcularDuracaoEmMinutos();
    int segundos = calcularSegundosRestantesDoMinuto();

    return String.format("%s - %s (%d:%02d)",
            banda.nome, titulo, minutos, segundos);
}
```

### Criar métodos grandes demais

Um método que valida, calcula, imprime, grava em banco e monta resposta HTTP provavelmente está fazendo coisas demais.

Métodos grandes escondem regras importantes e dificultam testes.

### Criar objetos de retorno sem significado

Nem todo conjunto de valores merece uma classe.

Uma classe como esta é fraca:

```java
public class Resultado {

    String valor1;
    String valor2;
    double valor3;

}
```

O nome não comunica nada.

Uma classe como esta é melhor:

```java
public class ResumoMusica {

    String descricao;
    int duracaoEmSegundos;
    boolean faixaLonga;
    double pontuacaoCatalogo;

}
```

Ela representa uma ideia do domínio.

### Passar muitos argumentos primitivos

Métodos com muitos `String`, `int` e `double` seguidos aumentam o risco de erro.

Exemplo problemático:

```java
cadastrarMusica("Pull Me Under", 496, 850, 4.7, 1992);
```

O leitor precisa lembrar a ordem e o significado de cada número.

Quando os dados pertencem ao mesmo conceito, um objeto pode melhorar a clareza.

### Confundir variável local com variável de instância

Variável de instância:

```java
int totalReproducoes;
```

Variável local:

```java
int bonus = 0;
```

A variável de instância faz parte do objeto. A variável local existe apenas durante a execução do método.

Confundir essas duas coisas gera bugs difíceis de rastrear.

### Achar que passar objeto como argumento cria uma cópia

Quando passamos `Musica` para um método, o objeto não é automaticamente copiado.

O método recebe uma cópia da referência para o mesmo objeto.

Isso é importante porque alterações no estado do objeto podem ser percebidas por quem chamou o método.

## Decisões de engenharia

### Quando usar métodos em uma classe de domínio

Use métodos na própria classe quando o comportamento depende diretamente do estado daquele objeto e representa algo natural do domínio.

Exemplos:

```java
musica.ehFaixaLonga()
musica.calcularPontuacaoCatalogo()
musica.gerarResumo()
```

Esses métodos fazem sentido porque usam dados da própria música.

### Quando evitar colocar o método na própria classe

Evite colocar o método na classe quando a regra pertence a outro processo do sistema.

Por exemplo, decidir se uma música entra nos destaques da página inicial pode depender de:

* estratégia editorial;
* quantidade mínima de reproduções;
* campanhas temporárias;
* regras de negócio externas;
* preferências do usuário;
* dados de outras músicas.

Nesse caso, uma classe como `CuradoriaCatalogo` pode ser mais adequada.

### Alternativas

Algumas alternativas de modelagem são:

* classe de domínio com comportamento;
* classe de serviço para regras de aplicação;
* objeto de retorno para agrupar resultados;
* record para agregados simples e imutáveis de dados;
* DTO para transportar dados entre camadas.

A escolha depende da responsabilidade.

Se o comportamento pertence ao conceito, considere colocá-lo na classe do conceito.

Se o comportamento coordena vários conceitos, considere uma classe de serviço.

Se o objeto só transporta uma resposta calculada, considere uma classe de retorno ou, em versões modernas do Java, um record.

### Impacto em legibilidade

Métodos pequenos e bem nomeados tornam o código mais fácil de ler.

Compare:

```java
if (musica.totalReproducoes >= 800
        && musica.calcularPontuacaoCatalogo() >= 80) {
    // destaque
}
```

com:

```java
if (curadoria.deveEntrarEmDestaques(musica, 800)) {
    // destaque
}
```

O segundo exemplo revela a intenção da regra.

### Impacto em manutenção

Quando uma regra está centralizada em um método, a mudança é mais segura.

Se amanhã o critério de destaque mudar, é melhor alterar um método com nome claro do que procurar a mesma condição repetida no controller, no service, no job e nos testes.

### Impacto em performance

Na maior parte dos sistemas backend, métodos pequenos não são o gargalo.

Gargalos costumam aparecer em:

* consultas ao banco;
* chamadas HTTP;
* serialização;
* I/O;
* locks;
* algoritmos ineficientes;
* processamento em lote mal planejado.

Evitar métodos pequenos por medo de custo de chamada normalmente piora o design antes de resolver qualquer problema real.

Performance deve ser medida, não presumida.

## Onde isso aparece em sistemas reais

### Entidades de domínio

Em sistemas backend, classes de domínio frequentemente possuem métodos que expressam regras.

Exemplos:

```java
pedido.calcularTotal()
pedido.podeSerCancelado()
assinatura.estaAtiva()
musica.gerarResumo()
```

Esses métodos evitam que regras importantes fiquem espalhadas.

### Services de aplicação

Quando uma operação coordena vários objetos, uma classe de serviço pode ser mais adequada.

Exemplo no domínio musical:

```java
curadoria.deveEntrarEmDestaques(musica, 800);
```

A curadoria pode usar dados da música, mas a decisão pertence ao processo de curadoria.

### APIs REST

Em uma API, objetos de retorno podem virar respostas JSON.

Um `ResumoMusica` poderia ser transformado em uma resposta como:

```json
{
  "descricao": "Dream Theater - Pull Me Under (8:16)",
  "duracaoEmSegundos": 496,
  "faixaLonga": true,
  "pontuacaoCatalogo": 103.1
}
```

O método `gerarResumo` concentra a criação desse resultado.

### Testes automatizados

Métodos pequenos são mais fáceis de testar.

Exemplo:

```java
boolean faixaLonga = musica.ehFaixaLonga();
```

Esse método pode ser testado isoladamente sem depender de banco de dados, API externa ou interface gráfica.

### Refatoração gradual

Em muitos sistemas legados, regras começam espalhadas em métodos grandes.

Uma refatoração comum é extrair pequenos métodos com nomes de domínio:

```java
calcularPontosPorReproducao()
calcularBonusDuracao()
atingiuMinimoDeReproducoes()
```

Isso melhora o código aos poucos sem precisar reescrever todo o sistema.

## Perguntas de revisão

1 - Qual é a diferença entre classe e objeto?

Resposta: A classe é a definição de um tipo. Ela descreve quais dados e comportamentos aquele tipo possui. O objeto é uma instância concreta dessa classe em memória. `Musica` é uma classe; uma música específica como `pullMeUnder` é um objeto.

2 - O que é um método em Java?

Resposta: Um método é um bloco de código nomeado, declarado dentro de uma classe ou estrutura compatível da linguagem, que representa um comportamento. Ele pode receber parâmetros, executar uma lógica e retornar ou não um valor.

3 - O que significa um método ser `void`?

Resposta: Significa que o método não devolve um valor para quem o chamou. Isso não quer dizer que ele não faz nada. Um método `void` pode alterar estado, imprimir mensagens, validar regras ou chamar outros métodos.

4 - Qual é a diferença entre imprimir um valor e retornar um valor?

Resposta: Imprimir envia o valor para uma saída, como o console. Retornar devolve o valor para o código que chamou o método. Um valor retornado pode ser armazenado, testado, enviado em uma API ou usado em outro cálculo.

5 - Por que `calcularPontuacaoCatalogo` deve retornar a pontuação em vez de apenas imprimir?

Resposta: Porque o nome indica uma operação de cálculo. Ao retornar o valor, o método permite que a pontuação seja reaproveitada por outras partes do sistema, como curadoria, testes, relatórios e respostas de API.

6 - O que a instrução `return` faz?

Resposta: `return` encerra a execução do método. Em métodos com retorno, ela também devolve um valor. Em métodos `void`, pode ser usada sem valor para sair do método antecipadamente.

7 - Código depois de um `return` sempre executa?

Resposta: Não. Depois que o `return` é executado, o método termina. Um código colocado logo após um `return` incondicional não será executado e pode gerar erro de compilação por ser inalcançável.

8 - É errado ter mais de um `return` no método?

Resposta: Não necessariamente. Muitos retornos espalhados em métodos grandes podem dificultar a leitura, mas retornos antecipados simples podem melhorar a clareza. O importante é manter o método pequeno, legível e com fluxo fácil de entender.

9 - O que é uma variável local?

Resposta: É uma variável declarada dentro de um método ou bloco. Ela existe apenas naquele escopo e durante aquela execução. Exemplo: `double pontosPorNota` dentro de `calcularPontuacaoCatalogo`.

10 - Qual é a diferença entre variável de instância e variável local?

Resposta: variável de instância faz parte do estado do objeto. Variável local pertence apenas à execução de um método ou bloco. variáveis de instância representam dados do objeto; variáveis locais ajudam a executar uma lógica temporária.

11 - Por que métodos pequenos ajudam na manutenção?

Resposta: Porque dão nome às partes da regra, evitam duplicação e facilitam mudanças futuras. Uma regra centralizada em um método pode ser alterada em um único lugar.

12 - Quando faz sentido um método retornar um objeto?

Resposta: Quando o resultado possui vários dados relacionados e tem significado próprio. Por exemplo, `gerarResumo` retorna um `ResumoMusica`, agrupando descrição, duração, indicador de faixa longa e pontuação.

13 - Java permite retornar dois valores diretamente com `return valor1, valor2`?

Resposta: Não. Java não permite retornar múltiplos valores soltos dessa forma. A alternativa é criar um objeto que represente o resultado, como `ResumoMusica`.

14 - O que é um parâmetro?

Resposta: Parâmetro é a variável declarada na assinatura do método. Exemplo: em `atingiuMinimoDeReproducoes(int minimoReproducoes)`, `minimoReproducoes` é o parâmetro.

15 - O que é um argumento?

Resposta: Argumento é o valor passado na chamada do método. Exemplo: em `musica.atingiuMinimoDeReproducoes(800)`, `800` é o argumento.

16 - Por que muitos parâmetros podem prejudicar a legibilidade?

Resposta: Porque deixam a chamada longa, aumentam a chance de trocar valores de ordem e dificultam entender o significado de cada argumento. Quando os dados pertencem ao mesmo conceito, passar um objeto pode ser mais claro.

17 - O que acontece quando passamos um objeto como argumento?

Resposta: O método recebe uma cópia da referência para o objeto. Ele não recebe uma cópia completa do objeto. Por isso, pode acessar o mesmo objeto e, se alterar seu estado, a alteração pode ser percebida por quem chamou.

18 - Java passa objetos por referência?

Resposta: Não no sentido clássico. Java passa valores. Quando o argumento é um objeto, o valor passado é uma referência para o objeto. Essa distinção é importante para entender alterações de estado e reatribuições.

19 - Como decidir se um método deve ficar na classe `Musica` ou em `CuradoriaCatalogo`?

Resposta: Se o comportamento depende diretamente da música e representa algo natural dela, pode ficar em `Musica`. Se a regra coordena decisões externas, políticas editoriais ou critérios do sistema, pode fazer mais sentido ficar em `CuradoriaCatalogo`.

20 - Records substituem classes comuns?

Resposta: Não. Records são úteis para agregados simples de dados, especialmente quando o objetivo é representar valores de forma concisa. Classes comuns continuam sendo mais adequadas quando o objeto possui comportamento rico, estado mutável ou regras mais flexíveis.

21 - Por que evitar colocar toda a lógica no `main`?

Resposta: Porque o `main` deve apenas iniciar o fluxo. Quando regras de negócio ficam nele, o código perde organização, fica difícil de testar e tende a duplicar lógica que deveria estar em classes e métodos do domínio.

22 - O que significa uma classe ser coesa?

Resposta: Uma classe é coesa quando suas responsabilidades estão relacionadas. Uma classe `Musica` com métodos de duração, pontuação e resumo é coesa. Uma classe `Musica` que também agenda shows, processa pagamento e envia e-mail provavelmente está fazendo coisas demais.

## Resumo final

Classes, objetos e métodos são fundamentos centrais da programação orientada a objetos em Java.

Uma classe define um tipo. Um objeto representa uma instância concreta desse tipo. Um método dá nome a um comportamento.

Métodos podem não retornar valor, usando `void`, ou podem retornar valores primitivos, textos, booleanos e objetos. Quando um resultado possui várias informações relacionadas, retornar um objeto com nome significativo costuma ser uma boa solução.

Métodos menores ajudam a evitar duplicação, melhoram a leitura e tornam a manutenção mais segura. Passar objetos como argumentos pode deixar o código mais expressivo quando os dados pertencem ao mesmo conceito.

Em sistemas backend reais, bons métodos ajudam a proteger regras de negócio contra duplicação e espalhamento. Eles tornam o código mais próximo da linguagem do domínio e mais fácil de evoluir.

O objetivo não é criar classes cheias de métodos sem critério. O objetivo é colocar comportamento no lugar certo, com nomes claros, responsabilidades coesas e regras fáceis de encontrar.

Ajustei apenas a seção **Referências**, agora com links em Markdown para você colar direto no artigo. Usei as fontes oficiais da Oracle/OpenJDK para Java/JEPs e páginas da InformIT/Pearson para os livros. ([Oracle Documentation][1])

## Referências

* [The Java Language Specification, Java SE 25 Edition](https://docs.oracle.com/javase/specs/jls/se25/html/index.html) — especificação oficial da linguagem Java. Consultar especialmente os capítulos sobre tipos de referência, escopo, classes, campos, métodos, instrução `return` e invocação de métodos.

* [Java SE 25 API Documentation](https://docs.oracle.com/en/java/javase/25/docs/api/index.html) — documentação oficial da API Java SE 25 e JDK 25.

* [JDK 25 Documentation](https://docs.oracle.com/en/java/javase/25/) — página central da documentação oficial do JDK 25, incluindo guias, documentação da API e notas de release.

* [OpenJDK — JDK 25](https://openjdk.org/projects/jdk/25/) — página oficial do projeto OpenJDK para o JDK 25, implementação de referência da plataforma Java SE 25.

* [OpenJDK — JEP 286: Local-Variable Type Inference](https://openjdk.org/jeps/286) — proposta que introduziu a inferência de tipo para variáveis locais com `var`, entregue no JDK 10.

* [OpenJDK — JEP 395: Records](https://openjdk.org/jeps/395) — proposta que finalizou `records` no JDK 16, úteis para modelar agregados simples de dados.

* [OpenJDK — JEP 512: Compact Source Files and Instance Main Methods](https://openjdk.org/jeps/512) — proposta finalizada no JDK 25 para simplificar programas pequenos com compact source files e instance main methods.

* [Effective Java, 3rd Edition — Joshua Bloch](https://www.informit.com/store/effective-java-9780134685991) — referência clássica sobre boas práticas de programação Java, design de classes, métodos, APIs e uso efetivo da linguagem.

* [Java Concurrency in Practice — Brian Goetz et al.](https://www.informit.com/store/java-concurrency-in-practice-9780321349606) — referência reconhecida sobre concorrência, thread safety, composição de classes seguras para múltiplas threads e Java Memory Model.

[1]: https://docs.oracle.com/javase/specs/jls/se25/html/index.html?utm_source=chatgpt.com "The Java® Language Specification"

