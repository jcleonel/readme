# SCA na prática: dependências vulneráveis, CVEs e gestão de bibliotecas

SCA significa **Software Composition Analysis**, ou Análise de Composição de Software.

Na prática, SCA é a disciplina de analisar os componentes que formam uma aplicação: bibliotecas, frameworks, plugins, dependências diretas, dependências transitivas, versões, licenças, CVEs e artefatos de terceiros.

Em aplicações Java/Spring, isso é extremamente importante porque uma aplicação corporativa raramente é feita apenas com código próprio.

Um projeto Spring Boot normalmente depende de:

- Spring Framework;
- Spring Boot;
- Jackson;
- Hibernate;
- Bean Validation;
- Tomcat, Jetty ou outro servidor embarcado;
- drivers JDBC;
- bibliotecas de logging;
- bibliotecas de parsing;
- clientes HTTP;
- bibliotecas de segurança;
- bibliotecas de teste;
- plugins Maven ou Gradle;
- dependências transitivas trazidas por outras dependências.

Mesmo que o código da aplicação esteja bem escrito, uma biblioteca vulnerável pode criar risco.

A ideia central é:

> Segurança da aplicação não depende apenas do código que você escreve. Depende também do código que você importa.

Na aplicação fictícia `meucatalogomusical.com`, imagine que usamos bibliotecas para importar planilhas de discografias de bandas como **Queensrÿche**, **Doro**, **Running Wild** e **Hanoi Rocks**. Se uma dessas bibliotecas tiver uma vulnerabilidade conhecida, a aplicação pode ficar exposta mesmo que nosso controller, service e repository estejam bem implementados.

## O problema real que SCA resolve

O desenvolvimento moderno é baseado em reutilização.

Isso é positivo. Ninguém quer implementar do zero:

- serialização JSON;
- cliente HTTP;
- parser XLSX;
- criptografia;
- servidor web;
- conexão com banco;
- logging;
- validação;
- autenticação;
- compressão;
- parsing XML;
- geração de PDF.

O problema é que cada biblioteca adicionada ao projeto aumenta a superfície de manutenção e risco.

Uma dependência pode:

- conter vulnerabilidade conhecida;
- trazer dependências transitivas vulneráveis;
- estar sem manutenção;
- ter licença incompatível;
- ser substituída por pacote malicioso;
- mudar comportamento em atualização;
- introduzir conflito de versão;
- carregar código não usado, mas ainda empacotado;
- gerar risco apenas em certas configurações;
- exigir upgrade que quebra compatibilidade.

SCA ajuda a responder perguntas como:

```text
Quais bibliotecas estão no projeto?
Quais versões estão em uso?
Existe alguma CVE conhecida?
A vulnerabilidade afeta dependência direta ou transitiva?
Existe versão corrigida?
A biblioteca vulnerável está realmente no artefato final?
A aplicação usa a parte vulnerável?
Qual é o impacto de atualizar?
```

Sem SCA, o time pode corrigir SQL Injection, XSS, SSRF e Path Traversal no código próprio, mas continuar empacotando bibliotecas vulneráveis.

## Antes de entrar nos detalhes: uma ideia simples

Pense em uma banda.

O show não depende apenas do vocalista.

Depende também de:

- guitarrista;
- baixista;
- baterista;
- técnico de som;
- roadies;
- instrumentos;
- cabos;
- pedais;
- amplificadores;
- mesa de som;
- iluminação.

Se um cabo importante falha, o show pode ser prejudicado.

Com software é parecido.

O código da aplicação é só uma parte do sistema.

As dependências também fazem parte da entrega.

Em `meucatalogomusical.com`, podemos escrever um código excelente para importar álbuns do **Mr. Big**, mas se a biblioteca de upload, JSON, XML ou compressão tiver uma CVE crítica explorável no nosso contexto, o risco continua existindo.

SCA serve para enxergar essa composição.

## Definição técnica

SCA é o processo de identificar, inventariar e analisar componentes de terceiros usados por uma aplicação, com foco em vulnerabilidades conhecidas, licenças, versões, dependências transitivas e risco de cadeia de suprimentos.

O OWASP Software Component Verification Standard define Component Analysis como o processo de identificar áreas potenciais de risco pelo uso de componentes de software de terceiros e open source.

Em um projeto Java/Spring, SCA normalmente analisa:

- `pom.xml`;
- `build.gradle`;
- lockfiles;
- dependências resolvidas;
- plugins;
- artefatos empacotados;
- SBOM;
- imagem Docker;
- bibliotecas diretas e transitivas;
- metadados de vulnerabilidade;
- CVEs;
- CPEs;
- pacotes Maven;
- versões efetivas.

A diferença importante é:

```text
SAST analisa o código.
SCA analisa os componentes usados pelo código.
```

Ambos são necessários.

## SAST, DAST e SCA

Na série anterior, estudamos SAST e DAST.

Agora o SCA entra como terceira camada.

| Técnica | O que analisa | Exemplo de problema |
|---|---|---|
| SAST | Código-fonte ou bytecode | SQL Injection, SSRF, Path Traversal |
| DAST | Aplicação em execução | Headers ausentes, comportamento inseguro em runtime |
| SCA | Dependências e componentes | Biblioteca com CVE conhecida |

Exemplo prático:

```text
SAST:
"Este parâmetro chega em Files.write sem validação."

DAST:
"A resposta real não possui X-Content-Type-Options."

SCA:
"O projeto empacota uma versão vulnerável de uma biblioteca de parsing."
```

Uma aplicação madura usa as três abordagens de forma complementar.

## O que é CVE?

CVE significa **Common Vulnerabilities and Exposures**.

É um identificador padronizado para vulnerabilidades conhecidas publicamente.

Exemplo conceitual:

```text
CVE-2026-12345
```

A página oficial do CVE Program descreve o programa como uma iniciativa que trabalha com membros da comunidade para expandir o conteúdo CVE e seu uso.

Uma CVE normalmente ajuda a responder:

- qual vulnerabilidade foi identificada;
- qual produto ou componente é afetado;
- quais versões são afetadas;
- qual é a descrição pública;
- quais referências existem;
- qual fornecedor ou CNA publicou;
- quais correções ou versões podem existir.

Mas uma CVE não diz sozinha se a sua aplicação está explorável.

Ela indica que existe uma vulnerabilidade conhecida associada a determinado componente ou produto.

## O que é CVSS?

CVSS significa **Common Vulnerability Scoring System**.

É um sistema de pontuação usado para classificar severidade de vulnerabilidades.

O NVD informa suporte a CVSS v2.0, v3.x e v4.0, além de classificações qualitativas como Low, Medium e High conforme as faixas definidas pelas respectivas versões do padrão.

Em relatórios SCA, é comum aparecer algo como:

```text
CVSS: 9.8 Critical
```

Isso indica severidade alta na métrica, mas ainda não substitui análise de contexto.

Uma CVE crítica em uma biblioteca que nem está empacotada em produção pode ter prioridade diferente de uma CVE alta em um componente exposto diretamente à internet.

## O que é CWE?

CWE significa **Common Weakness Enumeration**.

Enquanto CVE identifica uma vulnerabilidade específica, CWE descreve uma classe ou tipo de fraqueza.

Exemplo:

```text
CVE-2026-12345 -> vulnerabilidade específica
CWE-22 -> classe Path Traversal
CWE-79 -> classe XSS
CWE-918 -> classe SSRF
```

Em relatórios SCA, CVE e CWE podem aparecer juntos.

A CVE diz “qual vulnerabilidade”.

A CWE ajuda a entender “qual tipo de falha”.

## O que é CPE?

CPE significa **Common Platform Enumeration**.

Ferramentas SCA podem tentar associar uma dependência do projeto a um CPE para cruzar com bases de vulnerabilidade. A OWASP Dependency-Check, por exemplo, informa que, se um CPE é identificado, as CVEs associadas são listadas no relatório.

Esse processo pode gerar acertos e falsos positivos.

Por quê?

Porque nem sempre é simples mapear um pacote Maven para um produto afetado por uma CVE.

Às vezes:

- nomes são parecidos;
- artefatos são diferentes;
- forks confundem;
- a vulnerabilidade afeta outro módulo;
- a ferramenta associa o CPE errado;
- a dependência existe apenas em teste;
- a versão empacotada não é a mesma usada em runtime.

Por isso, SCA também exige triagem.

## Dependência direta e transitiva

Em Maven, uma dependência direta é aquela declarada explicitamente no `pom.xml`.

Exemplo:

```xml
<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi-ooxml</artifactId>
</dependency>
```

Uma dependência transitiva é trazida por outra dependência.

Exemplo conceitual:

```text
meucatalogomusical.com
    -> poi-ooxml
        -> xmlbeans
        -> commons-compress
        -> outras bibliotecas
```

Você não declarou todas manualmente, mas elas podem entrar no classpath.

É por isso que SCA é importante.

O risco pode estar em uma dependência que você nem sabia que estava usando.

## Maven dependency:tree

O Maven Dependency Plugin possui o objetivo `dependency:tree`, que exibe a árvore de dependências do projeto.

Comando:

```bash
mvn dependency:tree
```

Para filtrar uma dependência específica:

```bash
mvn dependency:tree -Dincludes=org.apache.poi
```

Para gravar em arquivo:

```bash
mvn dependency:tree -DoutputFile=dependency-tree.txt
```

Exemplo conceitual de saída:

```text
com.meucatalogomusical:api:jar:1.0.0
+- org.springframework.boot:spring-boot-starter-web:jar:4.0.8:compile
|  +- org.springframework:spring-webmvc:jar:7.0.9:compile
|  +- com.fasterxml.jackson.core:jackson-databind:jar:2.x:compile
+- org.apache.poi:poi-ooxml:jar:5.x:compile
   +- org.apache.xmlbeans:xmlbeans:jar:x.x:compile
   +- org.apache.commons:commons-compress:jar:x.x:compile
```

Esse comando ajuda a responder:

```text
Quem está trazendo essa biblioteca?
É dependência direta ou transitiva?
Qual versão foi resolvida?
Existe conflito de versão?
```

## Exemplo de POM com Spring Boot

Em aplicações Spring Boot, é comum usar o parent do Spring Boot ou importar o BOM de dependências.

Exemplo com parent:

```xml
<project>
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.meucatalogomusical</groupId>
    <artifactId>meucatalogomusical-api</artifactId>
    <version>1.0.0</version>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>4.0.8</version>
        <relativePath/>
    </parent>

    <properties>
        <java.version>25</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-webmvc</artifactId>
        </dependency>

        <dependency>
            <groupId>org.apache.poi</groupId>
            <artifactId>poi-ooxml</artifactId>
        </dependency>
    </dependencies>
</project>
```

O Spring Boot 4.0.8 exige pelo menos Java 17, é compatível até Java 26 e requer Spring Framework 7.0.9 ou superior; por isso, usar Java 25 LTS com Spring Boot 4.x está alinhado com a geração moderna do ecossistema Spring.

O ponto importante aqui é que muitas versões são gerenciadas pelo Spring Boot.

Você não precisa colocar versão manual em toda dependência.

## BOM e dependency management

BOM significa **Bill of Materials**.

No Maven, um BOM ajuda a centralizar versões de dependências para manter compatibilidade.

O guia de dependências do Maven explica que dependency management permite especificar versões de artefatos a serem usadas quando aparecem como dependências transitivas ou quando dependências não possuem versão especificada.

O Spring Boot usa o `spring-boot-dependencies` como BOM para gerenciar versões compatíveis de muitas bibliotecas. A documentação do Spring Boot Gradle Plugin informa que o plugin pode importar automaticamente o BOM `spring-boot-dependencies` da versão do Spring Boot em uso.

Isso é bom porque reduz combinações incompatíveis.

Mas não significa que você nunca terá CVE.

Significa apenas que o ecossistema está mais controlado.

## Evite sobrescrever versões sem necessidade

Um erro comum é sobrescrever versões gerenciadas pelo Spring Boot sem uma razão clara.

Exemplo:

```xml
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.12.0</version>
</dependency>
```

Se o Spring Boot já gerencia Jackson, forçar uma versão antiga pode criar risco.

Motivos aceitáveis para sobrescrever versão:

- correção emergencial de CVE;
- incompatibilidade conhecida;
- recomendação oficial;
- necessidade de versão corrigida ainda não gerenciada pelo BOM;
- decisão documentada de arquitetura.

Mesmo assim, a alteração precisa ser testada.

## Exemplo de vulnerabilidade em dependência

Imagine que `meucatalogomusical.com` usa uma biblioteca para ler metadados de álbuns de **Symphony X**.

```xml
<dependency>
    <groupId>com.exemplo</groupId>
    <artifactId>album-metadata-parser</artifactId>
    <version>1.2.0</version>
</dependency>
```

O SCA aponta:

```text
Dependency: com.exemplo:album-metadata-parser:1.2.0
Vulnerability: CVE-2026-99999
Severity: High
Fixed version: 1.2.3
```

A correção pode parecer simples:

```xml
<dependency>
    <groupId>com.exemplo</groupId>
    <artifactId>album-metadata-parser</artifactId>
    <version>1.2.3</version>
</dependency>
```

Mas antes de atualizar, precisamos analisar:

- a biblioteca é usada em produção?
- está no escopo `compile`, `runtime` ou `test`?
- a versão corrigida é compatível?
- há breaking changes?
- o trecho vulnerável é usado?
- existe mitigação temporária?
- os testes cobrem o fluxo?
- há risco de regressão?
- a versão corrigida é estável?
- o upgrade afeta dependências transitivas?

SCA não é apenas “mudar número de versão”.

É gestão de risco e compatibilidade.

## Escopos de dependência

Em Maven, o escopo importa.

Exemplos:

```xml
<scope>compile</scope>
```

```xml
<scope>runtime</scope>
```

```xml
<scope>test</scope>
```

Uma CVE em dependência de teste normalmente tem prioridade diferente de uma CVE em dependência empacotada em produção.

Mas cuidado: plugins de build e dependências de teste também podem representar risco em cadeia de suprimentos, especialmente no pipeline.

Avalie o contexto.

## Dependência transitiva vulnerável

Exemplo:

```text
meucatalogomusical-api
    -> biblioteca-importacao-discografia
        -> parser-xml-vulneravel
```

Você não declarou `parser-xml-vulneravel`, mas ele entrou no projeto.

Opções possíveis:

1. atualizar a dependência principal;
2. excluir a dependência transitiva;
3. forçar versão corrigida no dependency management;
4. substituir a biblioteca principal;
5. remover funcionalidade;
6. aceitar risco temporariamente com justificativa.

Exemplo de exclusão:

```xml
<dependency>
    <groupId>com.exemplo</groupId>
    <artifactId>biblioteca-importacao-discografia</artifactId>
    <version>3.1.0</version>
    <exclusions>
        <exclusion>
            <groupId>com.exemplo</groupId>
            <artifactId>parser-xml-vulneravel</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

Depois, incluir versão segura:

```xml
<dependency>
    <groupId>com.exemplo</groupId>
    <artifactId>parser-xml-vulneravel</artifactId>
    <version>3.1.4</version>
</dependency>
```

Esse tipo de correção exige teste.

Exclusão transitiva pode quebrar comportamento.

## Exemplo com dependencyManagement

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>com.exemplo</groupId>
            <artifactId>parser-xml-vulneravel</artifactId>
            <version>3.1.4</version>
        </dependency>
    </dependencies>
</dependencyManagement>
```

Isso força a versão resolvida quando a dependência aparecer na árvore.

Mas use com cuidado.

Forçar versão corrigida pode resolver CVE, mas também pode criar incompatibilidade com a biblioteca que esperava outra versão.

Sempre execute testes.

## Rodando OWASP Dependency-Check

O OWASP Dependency-Check é uma ferramenta SCA que detecta vulnerabilidades publicamente divulgadas em dependências do projeto.

Exemplo de execução via Maven:

```bash
mvn org.owasp:dependency-check-maven:check
```

Exemplo de plugin no `pom.xml`:

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.owasp</groupId>
            <artifactId>dependency-check-maven</artifactId>
            <version>12.1.8</version>
            <configuration>
                <failBuildOnCVSS>7.0</failBuildOnCVSS>
            </configuration>
        </plugin>
    </plugins>
</build>
```

O número exato de versão do plugin deve ser revisado no momento da implementação, porque ferramentas de segurança evoluem com frequência.

O ponto do exemplo é mostrar a ideia:

```text
Rodar SCA no build.
Gerar relatório.
Falhar o build acima de certo limiar.
Triar resultados.
```

## Cuidado com failBuildOnCVSS

Falhar build automaticamente pode ser útil, mas precisa de governança.

Exemplo:

```xml
<failBuildOnCVSS>7.0</failBuildOnCVSS>
```

Isso pode bloquear builds com CVSS alto.

Mas existem perguntas:

- a dependência está em produção?
- é falso positivo?
- há versão corrigida?
- a correção quebra compatibilidade?
- a vulnerabilidade é explorável no nosso uso?
- existe mitigação temporária?
- quem aprova exceção?
- qual prazo de correção?

Um pipeline maduro não é apenas bloqueio cego.

É bloqueio com processo.

## SBOM

SBOM significa **Software Bill of Materials**.

É uma lista estruturada dos componentes que compõem um software.

A CISA descreve SBOM como uma lista aninhada de ingredientes que compõem componentes de software. Ela também compara com uma lista de ingredientes em embalagem de alimentos, ajudando produtores e consumidores a entender componentes e riscos.

No ecossistema Java, uma forma comum de gerar SBOM é com CycloneDX.

O CycloneDX Maven Plugin gera SBOM contendo dependências diretas e transitivas do projeto.

Exemplo de plugin:

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.cyclonedx</groupId>
            <artifactId>cyclonedx-maven-plugin</artifactId>
            <version>2.9.1</version>
            <executions>
                <execution>
                    <phase>package</phase>
                    <goals>
                        <goal>makeAggregateBom</goal>
                    </goals>
                </execution>
            </executions>
            <configuration>
                <outputFormat>json</outputFormat>
            </configuration>
        </plugin>
    </plugins>
</build>
```

Saída conceitual:

```text
target/bom.json
```

O SBOM pode ser arquivado como evidência da composição da versão entregue.

## Por que SBOM importa?

SBOM ajuda quando surge uma nova CVE.

Exemplo:

```text
Nova CVE publicada em biblioteca X.
```

Sem SBOM, você precisa descobrir manualmente:

```text
Quais aplicações usam biblioteca X?
Quais versões?
Em quais ambientes?
Em quais releases?
```

Com SBOM, a empresa pode consultar inventário.

Isso melhora:

- resposta a incidentes;
- auditoria;
- rastreabilidade;
- gestão de risco;
- comunicação com clientes;
- conformidade;
- governança de supply chain.

SBOM não corrige vulnerabilidade sozinho.

Ele melhora visibilidade.

## Fluxo prático de triagem SCA

Quando um relatório SCA apontar vulnerabilidade, siga um fluxo.

### 1. Identificar o componente

Exemplo:

```text
org.exemplo:catalog-parser:1.4.0
```

Perguntas:

- é dependência direta?
- é transitiva?
- qual módulo traz?
- está em produção?
- está em teste?
- está no artefato final?

### 2. Identificar a CVE

Exemplo:

```text
CVE-2026-12345
```

Perguntas:

- qual descrição?
- quais versões afetadas?
- qual versão corrigida?
- qual severidade?
- há exploit público?
- há mitigação oficial?
- há nota do fornecedor?

### 3. Verificar contexto da aplicação

Perguntas:

- usamos o recurso vulnerável?
- o endpoint é exposto?
- a entrada é controlável?
- a dependência roda em runtime?
- a biblioteca é usada apenas em testes?
- há configuração que reduz risco?
- há outra camada de defesa?

### 4. Escolher tratamento

Opções:

- atualizar dependência;
- atualizar Spring Boot;
- atualizar BOM;
- forçar versão corrigida;
- excluir transitiva;
- substituir biblioteca;
- remover funcionalidade;
- mitigar configuração;
- aceitar risco temporário;
- marcar falso positivo.

### 5. Testar

Execute:

```bash
mvn test
```

```bash
mvn verify
```

```bash
mvn dependency:tree
```

```bash
mvn org.owasp:dependency-check-maven:check
```

Teste funcionalmente os fluxos que usam a biblioteca.

### 6. Documentar

Registre:

- CVE;
- biblioteca;
- versão afetada;
- versão corrigida;
- decisão;
- testes;
- risco residual;
- prazo;
- responsável.

## Exemplo prático com meucatalogomusical.com

Imagine que `meucatalogomusical.com` usa uma biblioteca fictícia para importar discografias de **Winger**, **FireHouse** e **Extreme**.

Dependência:

```xml
<dependency>
    <groupId>com.rockdata</groupId>
    <artifactId>discography-importer</artifactId>
    <version>2.0.0</version>
</dependency>
```

O relatório SCA aponta:

```text
Component: com.rockdata:discography-importer:2.0.0
CVE: CVE-2026-45678
Severity: High
Fixed version: 2.0.3
```

Primeiro, ver a árvore:

```bash
mvn dependency:tree -Dincludes=com.rockdata:discography-importer
```

Depois, atualizar:

```xml
<dependency>
    <groupId>com.rockdata</groupId>
    <artifactId>discography-importer</artifactId>
    <version>2.0.3</version>
</dependency>
```

Executar testes:

```bash
mvn test
mvn verify
```

Rodar novo SCA:

```bash
mvn org.owasp:dependency-check-maven:check
```

Testar fluxo real:

```text
Importar discografia de Winger.
Importar discografia de FireHouse.
Importar discografia com arquivo inválido.
Importar discografia grande.
Verificar logs.
Verificar erros.
```

A correção termina quando:

- versão vulnerável saiu da árvore;
- testes passaram;
- fluxo funcional continua;
- novo relatório não aponta a CVE;
- decisão foi documentada.

## Atualizar Spring Boot pode corrigir várias dependências

Às vezes, a melhor correção não é atualizar uma biblioteca isolada.

É atualizar o Spring Boot.

Exemplo:

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>4.0.8</version>
    <relativePath/>
</parent>
```

Como o Spring Boot gerencia versões de várias dependências, subir uma versão patch pode atualizar um conjunto compatível de bibliotecas.

Isso costuma ser mais seguro do que forçar versões isoladas sem necessidade.

Mas ainda exige:

- ler release notes;
- executar testes;
- validar compatibilidade;
- revisar breaking changes;
- rodar SCA novamente;
- gerar novo SBOM.

## Remover dependência pode ser melhor que atualizar

Às vezes, o melhor tratamento é remover a biblioteca.

Perguntas:

```text
Ainda usamos essa dependência?
Ela está no classpath sem necessidade?
Existe alternativa no JDK?
O Spring já fornece algo equivalente?
A biblioteca está abandonada?
A funcionalidade ainda existe?
```

Exemplo:

```bash
mvn dependency:tree -Dincludes=com.exemplo:biblioteca-antiga
```

Se não for mais usada, remova do POM.

Menos dependências significa:

- menos superfície;
- menos CVEs;
- menos conflitos;
- menos manutenção;
- builds menores.

## Cuidado com dependências opcionais e escopos

Algumas dependências aparecem apenas por causa de testes, plugins ou escopos específicos.

Exemplo:

```xml
<dependency>
    <groupId>com.exemplo</groupId>
    <artifactId>biblioteca-de-testes</artifactId>
    <scope>test</scope>
</dependency>
```

Se a CVE está em dependência `test`, o risco de produção pode ser menor.

Mas não ignore automaticamente.

Ela ainda pode afetar:

- pipeline;
- build;
- testes;
- geração de artefato;
- plugins;
- execução local;
- supply chain.

A triagem deve registrar o contexto.

## Falso positivo em SCA

SCA pode gerar falso positivo.

Cenários comuns:

- CPE associado incorretamente;
- biblioteca com nome parecido;
- dependência não empacotada;
- dependência apenas de teste;
- CVE afeta outro módulo;
- versão corrigida já está presente, mas ferramenta não reconheceu;
- vulnerabilidade depende de configuração que não existe;
- componente sombreado ou relocado;
- relatório baseado em cache antigo.

Mas falso positivo exige evidência.

Justificativa fraca:

```text
Não usamos isso.
```

Justificativa melhor:

```text
A CVE aponta o componente X em contexto de parsing XML remoto. No projeto, a dependência aparece apenas em escopo test, não é empacotada no artefato de produção, conforme dependency:tree e inspeção do JAR final. O fluxo vulnerável não é executado em runtime. Classificação proposta: falso positivo para produção, manter monitoramento no pipeline.
```

## Not Exploitable em SCA

Às vezes, a dependência está em produção, mas a vulnerabilidade não é explorável no contexto.

Exemplo:

```text
A CVE exige uso de modo servidor.
A aplicação usa apenas modo cliente.
```

Ou:

```text
A CVE exige parser de XML externo.
A aplicação nunca processa XML vindo de usuário.
```

Ou:

```text
A CVE afeta endpoint de administração da biblioteca.
A aplicação não expõe esse endpoint.
```

Mesmo assim, cuidado.

A melhor correção geralmente continua sendo atualizar a biblioteca.

`Not Exploitable` deve ser exceção temporária ou justificativa bem documentada, não desculpa permanente.

## Priorização de vulnerabilidades

Nem toda CVE tem a mesma prioridade.

Critérios úteis:

- severidade CVSS;
- componente exposto à internet;
- exploração pública conhecida;
- dependência em produção;
- facilidade de exploração;
- dados afetados;
- existência de patch;
- impacto em confidencialidade, integridade e disponibilidade;
- compensating controls;
- criticidade do sistema;
- uso real da biblioteca;
- existência de mitigação;
- prazo de correção.

Exemplo de prioridade alta:

```text
CVE crítica em biblioteca usada em endpoint público que processa entrada externa.
```

Exemplo de prioridade menor:

```text
CVE média em dependência de teste não empacotada em produção.
```

A severidade ajuda, mas o contexto decide a prioridade operacional.

## Gestão de exceções

Nem sempre é possível corrigir imediatamente.

Pode acontecer:

- não existe versão corrigida;
- a versão corrigida quebra compatibilidade;
- a biblioteca é legado crítico;
- a correção depende de fornecedor;
- a atualização exige refatoração maior;
- o risco é mitigado por configuração;
- a dependência está em módulo sem exposição.

Nesses casos, crie exceção formal.

A exceção deve ter:

- CVE;
- componente;
- versão;
- motivo;
- risco;
- mitigação;
- data de revisão;
- responsável;
- prazo de expiração;
- plano de correção.

Exceção sem prazo vira dívida invisível.

## SCA no pipeline

Um fluxo saudável pode ser:

```text
pull request
    -> build
    -> testes
    -> SAST
    -> SCA
    -> geração de SBOM
    -> empacotamento
    -> deploy em homologação
```

SCA deve rodar cedo.

O OWASP DevSecOps Guideline recomenda colocar Component Analysis cedo, antes de testes como SAST/DAST, para evitar que bibliotecas vulneráveis cheguem a produção e para permitir monitoramento contínuo.

Exemplo conceitual de pipeline:

```yaml
name: security-checks

on:
  pull_request:
  push:
    branches:
      - main

jobs:
  sca:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Set up Java
        uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: '25'

      - name: Build and test
        run: mvn verify

      - name: Dependency tree
        run: mvn dependency:tree -DoutputFile=target/dependency-tree.txt

      - name: OWASP Dependency-Check
        run: mvn org.owasp:dependency-check-maven:check

      - name: Generate SBOM
        run: mvn org.cyclonedx:cyclonedx-maven-plugin:makeAggregateBom
```

Esse exemplo mostra a ideia. Em uma empresa, a ferramenta pode ser Checkmarx SCA, GitHub Advanced Security, Snyk, Mend, Sonatype, OWASP Dependency-Check ou outra solução homologada.

## SCA e imagens Docker

SCA de aplicação não substitui análise de imagem.

Uma aplicação Spring Boot empacotada em container também depende de:

- imagem base;
- distribuição Linux;
- pacotes do sistema;
- JRE/JDK;
- certificados;
- bibliotecas nativas;
- ferramentas instaladas;
- usuário do container.

Exemplo:

```dockerfile
FROM eclipse-temurin:25-jre
COPY target/meucatalogomusical-api.jar app.jar
ENTRYPOINT ["java", "-jar", "/app.jar"]
```

Mesmo que o `pom.xml` esteja limpo, a imagem base pode ter vulnerabilidades.

Portanto, em ambientes containerizados, combine:

```text
SCA de dependências Java
+
scan de imagem container
+
SBOM
+
atualização de imagem base
```

## Boas práticas

### 1. Use BOM e dependency management

Evite versões espalhadas e incompatíveis.

### 2. Não sobrescreva versões sem justificativa

Deixe Spring Boot gerenciar o que ele já gerencia, salvo necessidade documentada.

### 3. Rode dependency:tree

Entenda quem traz a dependência vulnerável.

### 4. Rode SCA no pipeline

Não espere auditoria manual.

### 5. Gere SBOM

Tenha inventário da composição do software.

### 6. Atualize com testes

Upgrade sem teste pode corrigir CVE e quebrar funcionalidade.

### 7. Remova dependências não usadas

A melhor vulnerabilidade é aquela que não está no projeto.

### 8. Trate transitivas com cuidado

Exclusões e overrides precisam de teste.

### 9. Documente exceções

Risco aceito precisa de prazo e responsável.

### 10. Monitore continuamente

Nova CVE pode surgir depois do deploy.

## Erros comuns

### 1. Achar que SCA é só atualizar versão

SCA envolve contexto, teste, compatibilidade e risco.

### 2. Ignorar dependência transitiva

Muitas CVEs entram por transitivas.

### 3. Forçar versão sem testar

Pode quebrar runtime de forma sutil.

### 4. Ignorar dependência de teste sempre

Escopo test reduz risco de produção, mas ainda afeta pipeline.

### 5. Não gerar SBOM

Sem inventário, resposta a novas CVEs fica lenta.

### 6. Aceitar risco sem prazo

Exceção precisa expirar.

### 7. Atualizar tudo de uma vez sem estratégia

Pode criar regressões grandes.

### 8. Não revisar release notes

Algumas atualizações têm breaking changes.

### 9. Não escanear imagem Docker

Dependências do sistema operacional também importam.

### 10. Tratar CVSS como única verdade

CVSS ajuda, mas contexto da aplicação decide prioridade.

## Decisões de engenharia

## Quando atualizar só a biblioteca?

Quando:

- a versão corrigida é compatível;
- o impacto é pequeno;
- a dependência é direta;
- testes cobrem o uso;
- não há conflito com BOM;
- a correção é urgente.

## Quando atualizar Spring Boot?

Quando:

- a vulnerabilidade está em dependência gerenciada pelo Boot;
- há versão patch do Boot que atualiza a árvore;
- você quer manter compatibilidade do ecossistema;
- várias dependências serão corrigidas juntas.

## Quando excluir transitiva?

Quando:

- a transitiva vulnerável não é necessária;
- outra versão segura será usada;
- a biblioteca principal funciona sem ela;
- testes confirmam.

## Quando substituir biblioteca?

Quando:

- a biblioteca está abandonada;
- não há versão corrigida;
- há CVEs recorrentes;
- manutenção é fraca;
- existe alternativa melhor;
- o custo de manter exceções é alto.

## Quando aceitar risco temporariamente?

Quando:

- não há patch disponível;
- a exploração não é possível no contexto atual;
- existe mitigação;
- o impacto é baixo;
- há prazo curto para revisão;
- AppSec aprova.

## Quando bloquear build?

Quando:

- CVE crítica em runtime;
- exploração pública;
- componente exposto;
- patch disponível;
- risco incompatível com política;
- não há exceção aprovada.

## Onde isso aparece em sistemas reais

SCA aparece em:

- APIs Java/Spring;
- microsserviços;
- monólitos;
- batchs;
- jobs;
- aplicações serverless;
- containers;
- bibliotecas internas;
- SDKs corporativos;
- plugins Maven;
- pipelines CI/CD;
- imagens Docker;
- produtos SaaS;
- sistemas legados.

Em Java/Spring, os pontos mais comuns são:

- `pom.xml`;
- `build.gradle`;
- starters;
- drivers;
- parsers;
- clients HTTP;
- logging;
- JSON/XML;
- segurança;
- testes;
- plugins;
- BOMs;
- imagens base.

## Checklist prático

### Inventário

- Existe SBOM?
- Existe dependency tree?
- As dependências diretas são conhecidas?
- As transitivas críticas são conhecidas?
- Há dependências não usadas?
- Há bibliotecas abandonadas?

### Relatório SCA

- Qual componente foi apontado?
- Qual versão?
- Qual CVE?
- Qual severidade?
- Existe versão corrigida?
- A dependência é direta ou transitiva?
- Está em runtime, compile ou test?

### Contexto

- A aplicação usa a parte vulnerável?
- O componente está exposto a entrada externa?
- Está em produção?
- Existe mitigação?
- Existe exploit conhecido?
- Há compensating controls?

### Correção

- Atualizar dependência resolve?
- Atualizar Spring Boot resolve?
- Excluir transitiva resolve?
- Substituir biblioteca é melhor?
- A correção quebra compatibilidade?
- Os testes cobrem o fluxo?

### Pipeline

- SCA roda em PR?
- SCA roda na main?
- O build falha para severidade crítica?
- Exceções têm prazo?
- SBOM é gerado?
- Relatórios são arquivados?

### Operação

- Novas CVEs são monitoradas?
- Imagens Docker são escaneadas?
- Patches têm SLA?
- Riscos aceitos são revisados?
- Dependências internas também são monitoradas?
- O time sabe quem aprova exceções?

## Resumo final

SCA é a análise de composição de software.

Ela responde uma pergunta que SAST e DAST não respondem completamente:

> Quais riscos existem nas bibliotecas e componentes que a aplicação usa?

Em aplicações Java/Spring, SCA é essencial porque o projeto depende de muitas bibliotecas diretas e transitivas.

Os conceitos principais são:

- dependência direta;
- dependência transitiva;
- CVE;
- CVSS;
- CWE;
- CPE;
- BOM;
- SBOM;
- dependency tree;
- falso positivo;
- exceção de risco;
- gestão de atualização.

As principais práticas são:

- usar Spring Boot BOM;
- evitar versões manuais desnecessárias;
- rodar `mvn dependency:tree`;
- executar SCA no pipeline;
- gerar SBOM;
- atualizar dependências com teste;
- remover dependências não usadas;
- documentar exceções;
- escanear imagens container;
- monitorar novas CVEs continuamente.

A principal lição é:

> O código da aplicação pode estar seguro, mas uma dependência vulnerável também faz parte do risco da aplicação.

## Perguntas de revisão

### 1. O que é SCA?

SCA é Software Composition Analysis, a análise dos componentes, bibliotecas e dependências usados por uma aplicação.

### 2. SCA é igual a SAST?

Não. SAST analisa código. SCA analisa dependências e componentes.

### 3. O que é CVE?

CVE é um identificador padronizado para vulnerabilidades conhecidas publicamente.

### 4. O que é CVSS?

CVSS é um sistema de pontuação usado para indicar severidade de vulnerabilidades.

### 5. O que é dependência transitiva?

É uma dependência trazida por outra dependência, mesmo que você não a declare diretamente.

### 6. Por que `dependency:tree` é útil?

Porque mostra a árvore de dependências resolvida e ajuda a descobrir quem está trazendo determinada biblioteca.

### 7. O que é BOM?

BOM é Bill of Materials. No Maven, ajuda a centralizar versões compatíveis de dependências.

### 8. O que é SBOM?

SBOM é Software Bill of Materials, um inventário estruturado dos componentes que compõem o software.

### 9. Toda CVE em dependência é explorável?

Não. É preciso analisar contexto, escopo, uso real e exposição.

### 10. Uma dependência de teste importa?

Pode importar. Geralmente tem risco menor para produção, mas ainda pode afetar pipeline e cadeia de suprimentos.

### 11. O que fazer com dependência transitiva vulnerável?

Atualizar a dependência principal, excluir a transitiva, forçar versão corrigida, substituir biblioteca ou documentar exceção temporária.

### 12. É seguro sobrescrever versões gerenciadas pelo Spring Boot?

Só com justificativa. Preferencialmente, deixe o BOM gerenciar versões compatíveis.

### 13. Quando uma exceção de risco é aceitável?

Quando há justificativa, mitigação, responsável, prazo de expiração e aprovação adequada.

### 14. SCA substitui scan de imagem Docker?

Não. Dependências Java e pacotes da imagem base são camadas diferentes.

### 15. Qual é a principal regra?

Dependência também é código da aplicação. Se ela é empacotada e executada, ela precisa ser gerenciada como risco.

## Referências

- [OWASP Dependency-Check](https://owasp.org/projects/dependency-check)
- [OWASP Developer Guide — Dependency-Check](https://devguide.owasp.org/en/05-implementation/02-dependencies/01-dependency-check/)
- [OWASP Software Component Verification Standard — Component Analysis Requirements](https://scvs.owasp.org/scvs/v5-component-analysis/)
- [OWASP DevSecOps Guideline — Software Composition Analysis](https://owasp.github.io/www-project-devsecops-guideline/latest/02d-Software-Composition-Analysis)
- [CVE Program](https://www.cve.org/)
- [NVD — CVE FAQs](https://nvd.nist.gov/general/FAQ-Sections/CVE-FAQs)
- [NVD — Vulnerability Metrics / CVSS](https://nvd.nist.gov/vuln-metrics/cvss)
- [Apache Maven Dependency Plugin — dependency:tree](https://maven.apache.org/plugins/maven-dependency-plugin/tree-mojo.html)
- [Apache Maven — Introduction to the Dependency Mechanism](https://maven.apache.org/guides/introduction/introduction-to-dependency-mechanism.html)
- [Spring Boot — Managing Dependencies](https://docs.spring.io/spring-boot/gradle-plugin/managing-dependencies.html)
- [Spring Boot — System Requirements](https://docs.spring.io/spring-boot/4.0/system-requirements.html)
- [CycloneDX Maven Plugin](https://cyclonedx.github.io/cyclonedx-maven-plugin/)
- [CycloneDX Maven Plugin — makeBom](https://cyclonedx.github.io/cyclonedx-maven-plugin/makeBom-mojo.html)
- [Oracle JDK 25 Documentation](https://docs.oracle.com/en/java/javase/25/)
- [Oracle Java Downloads — JDK 25 LTS](https://www.oracle.com/java/technologies/downloads/)