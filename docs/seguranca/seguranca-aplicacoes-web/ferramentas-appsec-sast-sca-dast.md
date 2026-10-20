# Ferramentas de AppSec: como SAST, SCA e DAST encontram vulnerabilidades

Ao longo dos artigos anteriores, falamos várias vezes sobre “ferramentas”: SAST, SCA, DAST, scanners, análise de dependências, taint flow, source, sink, CVE, false positive e Not Exploitable.

Agora chegou o momento de entender melhor essas ferramentas.

A pergunta central deste artigo é:

> Como uma ferramenta sabe que um trecho de código, uma resposta HTTP ou uma biblioteca pode representar uma vulnerabilidade?

A resposta depende do tipo de ferramenta.

De forma geral:

| Tipo | O que analisa | Exemplo de ferramenta |
|---|---|---|
| SAST | Código-fonte, bytecode ou artefatos | Checkmarx, Veracode, CodeQL, SonarQube, Semgrep |
| SCA | Dependências, bibliotecas, SBOM, CVEs | Veracode SCA, Checkmarx SCA, Snyk, OWASP Dependency-Check, Dependency-Track |
| DAST | Aplicação em execução | OWASP ZAP, Burp Suite DAST, Veracode Dynamic Analysis |
| Container/IaC scanning | Imagens, pacotes de sistema, Dockerfile, Kubernetes, Terraform | Trivy, Snyk Container, Checkmarx IaC, GitHub Advanced Security |
| Secret scanning | Chaves, tokens e credenciais em código/repositório | GitHub Secret Scanning, Gitleaks, TruffleHog, Snyk, Trivy |

## O problema real que essas ferramentas resolvem

Em uma aplicação Java/Spring, existem muitos lugares onde vulnerabilidades podem aparecer.

Exemplo em `meucatalogomusical.com`, nossa aplicação fictícia de catálogo musical:

- busca por bandas;
- upload de capas de álbuns;
- download de arquivos;
- importação de planilhas;
- integração com parceiros;
- autenticação;
- autorização;
- comentários;
- logs;
- headers HTTP;
- dependências Maven;
- containers;
- configurações.

Um desenvolvedor pode revisar muito código manualmente, mas ainda assim deixar passar pontos importantes.

Exemplo:

```java
@GetMapping("/buscar")
public List<BandaResponse> buscar(@RequestParam String nome) {
    return bandaRepository.buscarPorNome(nome);
}
```

Esse trecho parece simples.

Mas uma ferramenta SAST pode querer saber:

```text
Esse nome entra em uma query SQL?
Esse nome vai para uma resposta HTML?
Esse nome controla um arquivo?
Esse nome controla uma URL outbound?
Esse nome vai para log?
```

Já uma ferramenta SCA pode olhar para o `pom.xml` e perguntar:

```text
Quais bibliotecas esse projeto usa?
Há alguma CVE conhecida?
Essa dependência é direta ou transitiva?
Existe versão corrigida?
```

E uma ferramenta DAST pode rodar contra a aplicação em execução e perguntar:

```text
O endpoint retorna HSTS?
O Content-Type está correto?
Há formulário vulnerável?
Há comportamento suspeito quando envio entradas inesperadas?
```

Cada ferramenta enxerga um ângulo diferente do mesmo sistema.

## Antes de entrar nos detalhes: uma ideia simples

Pense em um show de Metal.

A segurança do evento não depende de uma única pessoa.

Existe:

- segurança na entrada;
- equipe de palco;
- técnico de som;
- bombeiro;
- controle de acesso;
- monitoramento;
- checagem dos equipamentos;
- inspeção da estrutura;
- plano de emergência.

Cada equipe olha para um risco diferente.

Com AppSec é igual.

SAST olha o código.

SCA olha os componentes.

DAST olha a aplicação funcionando.

Secret scanning olha segredos vazados.

Container scanning olha a imagem.

IaC scanning olha infraestrutura como código.

Nenhuma ferramenta vê tudo.

A segurança vem da combinação.

## Definição técnica

Ferramentas de AppSec são ferramentas usadas para identificar, priorizar, corrigir, monitorar ou prevenir vulnerabilidades em aplicações, dependências, configurações e ambientes.

As categorias mais comuns são:

- **SAST**: Static Application Security Testing;
- **SCA**: Software Composition Analysis;
- **DAST**: Dynamic Application Security Testing;
- **IAST**: Interactive Application Security Testing;
- **RASP**: Runtime Application Self-Protection;
- **Secret scanning**;
- **Container scanning**;
- **IaC scanning**;
- **SBOM analysis**.

A OWASP descreve análise estática como ferramentas que analisam código não executado, usando técnicas como **taint analysis** e **data flow analysis**; na análise de taint, variáveis controláveis por usuário são rastreadas até funções sensíveis, chamadas de sinks.

Essa definição conversa diretamente com os artigos anteriores sobre source, sink e taint flow.

## SAST: análise estática de código

SAST significa **Static Application Security Testing**.

É a análise de segurança feita sem executar a aplicação.

A ferramenta analisa:

- código-fonte;
- bytecode;
- artefatos compilados;
- AST;
- fluxo de dados;
- fluxo de controle;
- chamadas de método;
- imports;
- frameworks;
- padrões de código;
- regras de segurança.

No Java/Spring, SAST pode procurar vulnerabilidades como:

- SQL Injection;
- XSS;
- SSRF;
- Path Traversal;
- Privacy Violation;
- Cleartext Submission;
- Unchecked Input for Loop Condition;
- hardcoded secrets;
- uso inseguro de criptografia;
- logs com dados sensíveis;
- validação ausente;
- autorização incorreta;
- má configuração de headers.

Ferramentas conhecidas:

- Checkmarx SAST;
- Veracode Static Analysis;
- GitHub CodeQL;
- SonarQube/SonarCloud;
- Semgrep;
- Fortify;
- Coverity;
- CodeQL CLI;
- SpotBugs com plugins de segurança;
- PMD com regras customizadas.

O Checkmarx SAST é descrito pela própria documentação como uma solução de análise de código-fonte para identificar, rastrear e reparar falhas técnicas e lógicas, incluindo vulnerabilidades de segurança, compliance e problemas de lógica de negócio.

## Como o SAST “sabe” que um trecho pode ser vulnerável?

SAST não “entende” o sistema como um humano.

Ele usa modelos.

Esses modelos podem incluir:

- regras;
- padrões;
- queries;
- AST;
- call graph;
- control flow graph;
- data flow;
- taint analysis;
- listas de APIs perigosas;
- modelos de frameworks;
- sources conhecidos;
- sinks conhecidos;
- sanitizers conhecidos;
- validators conhecidos.

Exemplo conceitual:

```text
Source:
@RequestParam
@RequestBody
MultipartFile
HttpServletRequest.getHeader()

Sink:
JdbcTemplate.query(String sql)
Files.write(...)
RestClient.uri(...)
log.info(...)
ResponseEntity.ok(...)
```

A ferramenta tenta descobrir se existe um caminho entre o source e o sink.

Se existir e não houver sanitizer adequado, ela abre um finding.

## Source, sink e sanitizer em Java/Spring

Exemplo inseguro:

```java
@RestController
@RequestMapping("/api/bandas")
public class BandaController {

    private final JdbcTemplate jdbcTemplate;

    public BandaController(JdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
    }

    @GetMapping("/buscar")
    public List<BandaResponse> buscar(@RequestParam String nome) {
        String sql = "select id, nome, genero from bandas where nome = '" + nome + "'";

        return jdbcTemplate.query(sql, (rs, rowNum) -> new BandaResponse(
                rs.getLong("id"),
                rs.getString("nome"),
                rs.getString("genero")
        ));
    }
}
```

Uma ferramenta SAST pode modelar assim:

```text
Source:
@RequestParam String nome

Propagation:
nome -> concatenação de String -> sql

Sink:
jdbcTemplate.query(sql)

Possível vulnerabilidade:
SQL Injection
```

Correção:

```java
@GetMapping("/buscar")
public List<BandaResponse> buscar(@RequestParam String nome) {
    String sql = """
            select id, nome, genero
            from bandas
            where nome = ?
            """;

    return jdbcTemplate.query(sql, (rs, rowNum) -> new BandaResponse(
            rs.getLong("id"),
            rs.getString("nome"),
            rs.getString("genero")
    ), nome);
}
```

Agora a ferramenta pode reconhecer que o valor entra como parâmetro, não como parte da string SQL.

Esse é o motivo pelo qual ferramentas falam tanto de taint flow.

## Data flow e taint tracking

GitHub CodeQL documenta bibliotecas de **data flow** e **taint tracking** para rastrear o fluxo de dados em programas Java/Kotlin; o taint tracking estende a análise de fluxo para passos em que o valor pode não ser preservado exatamente, mas o objeto potencialmente inseguro continua propagado.

Isso é importante porque vulnerabilidades raramente aparecem em uma única linha.

Exemplo:

```java
@GetMapping("/capas")
public void salvarCapa(@RequestParam String nomeArquivo, MultipartFile arquivo) {
    capaService.salvar(nomeArquivo, arquivo);
}
```

```java
@Service
public class CapaService {

    private final FileStorage fileStorage;

    public CapaService(FileStorage fileStorage) {
        this.fileStorage = fileStorage;
    }

    public void salvar(String nomeArquivo, MultipartFile arquivo) {
        fileStorage.salvar(nomeArquivo, arquivo);
    }
}
```

```java
@Component
public class FileStorage {

    private final Path base = Path.of("/var/app/uploads");

    public void salvar(String nomeArquivo, MultipartFile arquivo) {
        Path destino = base.resolve(nomeArquivo);

        try {
            Files.write(destino, arquivo.getBytes());
        } catch (IOException ex) {
            throw new IllegalStateException("Falha ao salvar arquivo.", ex);
        }
    }
}
```

O finding pode ser:

```text
Source:
@RequestParam String nomeArquivo

Flow:
Controller -> Service -> FileStorage

Sink:
Files.write

Possível vulnerabilidade:
Path Traversal
```

O SAST precisa seguir o dado entre classes, métodos e arquivos.

Por isso, ferramentas mais avançadas falam em **interprocedural analysis**, **cross-file analysis** e **whole-program analysis**.

## Queries e regras

Cada ferramenta tem seu jeito de expressar regras.

No CodeQL, uma vulnerabilidade pode ser descrita por uma query.

No Semgrep, uma regra pode ser escrita em YAML.

No Checkmarx, existem queries SAST customizáveis; a documentação do Checkmarx SAST Scanner informa que o Query Editor permite customizar queries de análise ou configurar queries adicionais para segurança, qualidade e lógica de aplicação.

No SonarQube, existem regras de segurança e taint analysis; a documentação da SonarSource explica que o fluxo se torna problema quando o usuário controla a entrada, essa entrada não é validada ou sanitizada e chega a uma função sensível, ou sink.

Exemplo conceitual de regra:

```text
Se:
  dado externo chega em RestClient.uri(...)

E:
  não passa por validação de host permitido

Então:
  possível SSRF
```

Essa é uma simplificação, mas ajuda a entender o raciocínio.

## Exemplo conceitual de regra Semgrep

Semgrep tem um modo de taint analysis que usa conceitos como sources e sinks; a documentação da Semgrep define source como código que define dado tainted, geralmente input de usuário, e sink como função vulnerável chamada com dado potencialmente tainted.

Exemplo didático de regra:

```yaml
rules:
  - id: spring-restclient-ssrf
    message: "Possível SSRF: entrada externa usada como URI outbound."
    severity: ERROR
    languages:
      - java
    mode: taint
    pattern-sources:
      - pattern: |
          @RequestParam String $URL
    pattern-sinks:
      - pattern: |
          $CLIENT.get().uri($URL)
```

Essa regra é simplificada.

Uma regra real precisaria tratar:

- nomes variados;
- métodos diferentes;
- `RestClient`;
- `RestTemplate`;
- `WebClient`;
- `HttpClient`;
- sanitizers;
- validators;
- allowlists;
- falsos positivos.

Mas a ideia é clara:

```text
Entrada HTTP -> cliente HTTP outbound = possível SSRF
```

## Por que existem falsos positivos em SAST?

Falsos positivos acontecem porque a ferramenta trabalha por modelos.

Ela pode não entender:

- regra de negócio;
- configuração de gateway;
- validação customizada;
- serializers;
- DTOs públicos;
- filtros globais;
- annotations internas;
- constraints de domínio;
- limites matemáticos;
- contexto JSON versus HTML;
- uso real da dependência;
- fluxo inalcançável;
- contrato legado.

Exemplo:

```java
URI endpointValidado = outboundUriValidator.requireAllowedHttpsEndpoint(endpoint);

restClient
        .get()
        .uri(endpointValidado)
        .retrieve()
        .body(String.class);
```

Se a ferramenta não reconhece `requireAllowedHttpsEndpoint` como sanitizer/validator, ela pode continuar apontando SSRF.

Por isso, em muitos projetos corporativos, o time AppSec precisa ajustar regras ou cadastrar sanitizers customizados.

## Veracode: como enxerga flaws

A documentação da Veracode informa que o Upload and Scan executa Static Analysis para encontrar flaws no código e SCA para identificar bibliotecas open source vulneráveis, bibliotecas maliciosas e licenças.

A Veracode também documenta que sua metodologia aplica rating de explorabilidade a findings de Static Analysis e considera informações sobre o caminho de data flow, especificamente a source do dado tainted.

Na prática, para um desenvolvedor, isso significa que a Veracode pode reportar:

- categoria da falha;
- severidade;
- arquivo;
- linha;
- source;
- sink;
- caminho de fluxo;
- recomendação;
- status;
- histórico entre scans.

Esse modelo é parecido com o que estudamos nos artigos de SAST e triagem.

## Checkmarx: como enxerga findings

A documentação do Checkmarx SAST Scanner informa que o input de SAST é o código-fonte, não binários; também menciona queries, inputs, sinks e sanitizers nas configurações de análise.

Isso explica algo que você viu na prática: o Checkmarx pode seguir um dado desde um controller até um utilitário, como `FileUtil`, e apontar vulnerabilidades como:

- SSRF;
- Absolute Path Traversal;
- Reflected XSS;
- Privacy Violation;
- Cleartext Submission;
- Unchecked Input for Loop Condition.

O scanner não está “adivinhando”.

Ele está aplicando queries sobre o código-fonte e tentando montar caminhos entre entradas e operações sensíveis.

## CodeQL e GitHub Advanced Security

CodeQL é o mecanismo de análise de código desenvolvido pelo GitHub para automatizar verificações de segurança; a documentação do GitHub afirma que especialistas do GitHub, pesquisadores de segurança e contribuidores mantêm queries padrão usadas no code scanning.

O CodeQL funciona de forma interessante:

```text
Código-fonte -> banco CodeQL -> queries -> alerts
```

A ferramenta transforma o código em uma base consultável.

Depois, queries procuram padrões vulneráveis.

Isso é poderoso porque permite criar queries customizadas para regras específicas da empresa.

Exemplo de uso:

```yaml
name: codeql

on:
  pull_request:
  push:
    branches:
      - main

jobs:
  analyze:
    runs-on: ubuntu-latest

    permissions:
      security-events: write
      packages: read
      actions: read
      contents: read

    steps:
      - uses: actions/checkout@v4

      - uses: github/codeql-action/init@v3
        with:
          languages: java-kotlin

      - uses: github/codeql-action/analyze@v3
```

Esse pipeline executa CodeQL em Java/Kotlin.

## SonarQube e SonarCloud

SonarQube é muito conhecido por qualidade de código, mas também possui regras de segurança.

A documentação da SonarQube informa que, para mostrar fluxo de issues tainted, usa tecnologia de taint analysis em código-fonte; o problema ocorre quando entrada controlada pelo usuário chega a funções sensíveis sem validação ou sanitização adequada.

Exemplo de configuração básica:

```properties
sonar.projectKey=meucatalogomusical
sonar.projectName=meucatalogomusical
sonar.sources=src/main/java
sonar.tests=src/test/java
sonar.java.binaries=target/classes
```

Em pipelines Java/Spring, Sonar costuma ser usado para:

- bugs;
- code smells;
- coverage;
- duplicação;
- vulnerabilities;
- security hotspots;
- quality gate.

Ele não substitui Checkmarx ou Veracode em todos os contextos, mas pode ser uma camada importante.

## Semgrep

Semgrep é conhecido por regras rápidas e customizáveis.

Ele é útil quando o time quer criar regras específicas do projeto.

Exemplo:

```yaml
rules:
  - id: no-sensitive-log
    message: "Não registre clientSecret em logs."
    severity: ERROR
    languages:
      - java
    patterns:
      - pattern: |
          $LOG.$METHOD(..., $SECRET, ...)
      - metavariable-regex:
          metavariable: $SECRET
          regex: ".*(secret|password|token).*"
```

Essa regra é simplificada, mas ilustra uma ideia útil:

```text
Podemos escrever regras para padrões próprios da empresa.
```

No mundo real, regras de log seguro precisam ser bem planejadas para evitar ruído.

## SCA: análise de dependências

SCA significa **Software Composition Analysis**.

A ferramenta não está procurando uma concatenação insegura de SQL no seu código.

Ela está procurando riscos nos componentes usados pela aplicação.

A Veracode descreve SCA como uma forma de construir inventário de componentes de terceiros para identificar bibliotecas maliciosas e vulnerabilidades em bibliotecas open source e código comercial; ela compila uma lista de bibliotecas da aplicação e identifica vulnerabilidades conhecidas em cada biblioteca.

Ferramentas SCA conhecidas:

- Veracode SCA;
- Checkmarx SCA;
- Snyk Open Source;
- OWASP Dependency-Check;
- OWASP Dependency-Track;
- GitHub Dependabot;
- GitHub Dependency Review;
- Sonatype Nexus Lifecycle;
- Mend;
- Black Duck;
- Trivy;
- Grype.

## Como o SCA “sabe” que uma lib é vulnerável?

O SCA normalmente faz algo como:

```text
1. Lê manifestos do projeto
   pom.xml, build.gradle, package-lock.json, requirements.txt...

2. Resolve dependências
   diretas e transitivas

3. Identifica pacotes
   groupId, artifactId, version, package-url, CPE, hashes, metadados

4. Consulta bases de vulnerabilidades
   NVD, CVE, bases próprias, advisories, GitHub Advisory Database...

5. Compara versão instalada com ranges vulneráveis

6. Gera finding
   biblioteca, versão, CVE, severidade, caminho de dependência, correção
```

A Snyk documenta que, ao escanear open source, cria uma árvore hierárquica com dependências diretas e indiretas representadas no manifest e usa sua base de vulnerabilidades para encontrar vulnerabilidades em qualquer pacote da árvore.

O OWASP Dependency-Check identifica dependências do projeto e verifica vulnerabilidades publicamente divulgadas; se identifica um CPE, lista CVEs associadas no relatório.

## Exemplo com Maven

`pom.xml`:

```xml
<dependencies>
    <dependency>
        <groupId>org.apache.poi</groupId>
        <artifactId>poi-ooxml</artifactId>
    </dependency>
</dependencies>
```

Comando:

```bash
mvn dependency:tree
```

Saída conceitual:

```text
com.meucatalogomusical:api:jar:1.0.0
\- org.apache.poi:poi-ooxml:jar:5.x
   +- org.apache.xmlbeans:xmlbeans:jar:x.x
   \- org.apache.commons:commons-compress:jar:x.x
```

Uma ferramenta SCA olha essa árvore e pergunta:

```text
Alguma dessas versões possui CVE conhecida?
Existe versão corrigida?
A dependência é direta ou transitiva?
```

## OWASP Dependency-Check

Exemplo de execução:

```bash
mvn org.owasp:dependency-check-maven:check
```

Exemplo de plugin:

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

O OWASP Dependency-Check é uma ferramenta SCA que tenta detectar vulnerabilidades publicamente divulgadas em dependências do projeto; ele cruza evidências das dependências com CPE/CVE e gera relatório.

## OWASP Dependency-Track

Dependency-Track é diferente de simplesmente rodar um scanner local.

Ele trabalha muito bem com SBOM.

A OWASP descreve Dependency-Track como uma plataforma de Component Analysis que usa SBOM para identificar e reduzir risco na cadeia de suprimentos de software.

Fluxo típico:

```text
Projeto gera SBOM
    -> SBOM enviado ao Dependency-Track
    -> Dependency-Track monitora componentes
    -> novas vulnerabilidades aparecem
    -> projeto é notificado
```

Isso é útil porque uma aplicação pode estar segura hoje e vulnerável amanhã, quando uma nova CVE for publicada.

## Checkmarx SCA

A documentação do Checkmarx SCA informa que, para cada CVE suportada, o time de pesquisa identifica a função ou funções exatas que precisam ser chamadas para a vulnerabilidade ser disparada e documenta sua assinatura.

Isso é importante porque SCA tradicional pode dizer:

```text
Você usa biblioteca vulnerável.
```

Mas reachability/exploitable path tenta responder:

```text
Seu código realmente chama o método vulnerável?
```

Isso reduz ruído e ajuda a priorizar.

## Veracode SCA

A documentação da Veracode SCA informa que os scans agent-based analisam componentes open source, bibliotecas e licenças cedo no SDLC; para cada biblioteca identificada, a Veracode reporta vulnerabilidades com base em sua base de vulnerabilidades. Ela também menciona coleta de métodos vulneráveis encontrados durante criação de call chains do código.

Isso aproxima SCA de SAST em alguns casos:

```text
biblioteca vulnerável
+
seu código chama método vulnerável
=
prioridade maior
```

## Snyk

Snyk Open Source analisa dependências e licenças; a documentação do `snyk test` informa que o comando verifica projetos em busca de vulnerabilidades open source e problemas de licença.

Exemplo:

```bash
snyk test
```

Exemplo monitoramento contínuo:

```bash
snyk monitor
```

O `snyk monitor` cria um projeto na conta Snyk para monitoramento contínuo de vulnerabilidades open source e issues de licença.

## DAST: análise dinâmica

DAST significa **Dynamic Application Security Testing**.

Ao contrário do SAST, ele não lê seu código.

Ele testa a aplicação rodando.

O DAST interage com:

- URLs;
- formulários;
- APIs;
- headers;
- cookies;
- autenticação;
- parâmetros;
- respostas;
- redirects;
- conteúdo HTML;
- status HTTP;
- erros.

Ferramentas comuns:

- OWASP ZAP;
- Burp Suite;
- Burp Suite DAST;
- Veracode Dynamic Analysis;
- Invicti;
- Acunetix;
- StackHawk;
- Qualys WAS;
- Rapid7 InsightAppSec.

## Como o DAST “sabe” que existe vulnerabilidade?

DAST geralmente trabalha em duas fases:

```text
Crawl:
descobrir páginas, endpoints, links, formulários e parâmetros

Audit/Scan:
enviar entradas de teste, observar respostas e inferir vulnerabilidades
```

Burp Scanner é documentado como ferramenta que realiza crawl e audit; a documentação informa que o scanner foi projetado para replicar ações e metodologias de um testador manual habilidoso.

O OWASP ZAP, em seus materiais de introdução, explica que pode fazer passive scan de requests/responses trafegados por ele e active scan para atacar páginas, funcionalidades e parâmetros descobertos.

Exemplo:

```text
DAST envia:
GET /api/bandas?nome=...

Observa:
status 500
mensagem SQL no erro
tempo de resposta diferente
header ausente
reflexão do input na resposta
cookie sem Secure
ausência de HSTS
```

Com isso, ele abre findings.

## DAST passivo e ativo

DAST passivo observa.

Ele pode encontrar:

- headers ausentes;
- cookies sem `Secure`;
- cookies sem `HttpOnly`;
- `Content-Type` inseguro;
- informações sensíveis em resposta;
- versões expostas;
- erros detalhados;
- ausência de HSTS;
- ausência de `nosniff`.

DAST ativo interage.

Ele pode enviar entradas para testar:

- XSS;
- SQL Injection;
- command injection;
- path traversal;
- open redirect;
- SSRF em alguns casos;
- autenticação fraca;
- métodos HTTP inseguros.

PortSwigger alerta que checks ativos intrusivos simulam ataques reais e podem modificar ou danificar a aplicação ou seus dados.

Por isso, DAST deve rodar em ambiente autorizado e controlado.

## Exemplo de finding DAST: Missing HSTS

DAST não precisa ver seu código para apontar HSTS ausente.

Ele faz uma requisição:

```bash
curl -I https://meucatalogomusical.com/api/bandas/1
```

Resposta:

```http
HTTP/2 200
content-type: application/json
```

Se não houver:

```http
Strict-Transport-Security: max-age=...
```

a ferramenta pode apontar:

```text
Missing Strict-Transport-Security Header
```

A correção pode ser na aplicação, no gateway ou no load balancer.

O DAST só sabe o que a resposta real mostra.

## Exemplo de finding DAST: nosniff ausente

Resposta:

```http
HTTP/1.1 200 OK
Content-Type: application/json
```

Sem:

```http
X-Content-Type-Options: nosniff
```

Finding:

```text
Missing X-Content-Type-Options Header
```

A ferramenta detecta porque compara a resposta HTTP com um conjunto de políticas esperadas.

## Exemplo de OWASP ZAP em pipeline

Exemplo conceitual:

```yaml
name: dast-zap

on:
  workflow_dispatch:

jobs:
  zap:
    runs-on: ubuntu-latest

    steps:
      - name: Start application
        run: docker compose up -d

      - name: ZAP baseline scan
        uses: zaproxy/action-baseline@v0.12.0
        with:
          target: 'http://localhost:8080'
```

Esse tipo de scan deve ser ajustado para o ambiente real.

Para áreas autenticadas, é necessário configurar autenticação.

## Burp Suite DAST

Burp Suite é muito usado em testes manuais e DAST.

A documentação do Burp Suite DAST fala em configurações de crawl e audit, escopo e scans recorrentes/automatizados.

Na prática:

```text
Burp DAST
    -> recebe site/API
    -> faz crawl
    -> audita endpoints
    -> abre findings
    -> prioriza itens
    -> gera relatório
```

Burp é muito forte porque combina automação com uso manual.

Em times maduros, ele costuma ser usado tanto por AppSec quanto por pentesters.

## Por que SAST, SCA e DAST encontram coisas diferentes?

Porque cada ferramenta tem visibilidade diferente.

### SAST vê código

Ele pode encontrar:

```text
@RequestParam -> Files.write
```

Mesmo sem executar a aplicação.

Mas pode não saber se o endpoint é alcançável em produção.

### SCA vê bibliotecas

Ele pode encontrar:

```text
commons-compress versão X possui CVE Y
```

Mesmo que seu código pareça seguro.

Mas pode não saber se você usa o método vulnerável.

### DAST vê comportamento

Ele pode encontrar:

```text
Resposta real não possui HSTS.
```

Mesmo sem conhecer seu código.

Mas pode não alcançar endpoints autenticados ou fluxos complexos.

Por isso, ferramentas se complementam.

## Matriz comparativa

| Categoria | Vê código? | Executa app? | Vê dependências? | Exemplo de finding |
|---|---:|---:|---:|---|
| SAST | Sim | Não | Parcial | Path Traversal em `Files.write` |
| SCA | Parcial | Não | Sim | CVE em biblioteca Maven |
| DAST | Não | Sim | Não | HSTS ausente |
| Container scan | Não necessariamente | Não | Pacotes da imagem | CVE em imagem base |
| Secret scan | Sim | Não | Não | Token commitado |
| IaC scan | Sim | Não | Não | Security group aberto |

## Ferramentas comerciais e open source

### Comerciais comuns

- Veracode;
- Checkmarx;
- Snyk;
- SonarQube commercial editions;
- Fortify;
- Black Duck;
- Mend;
- Invicti;
- Acunetix;
- Burp Suite Enterprise/DAST;
- GitHub Advanced Security.

### Open source ou com uso gratuito relevante

- OWASP ZAP;
- OWASP Dependency-Check;
- OWASP Dependency-Track;
- Semgrep Community Edition;
- CodeQL para projetos open source e uso conforme licença;
- Trivy;
- Gitleaks;
- TruffleHog;
- SpotBugs;
- PMD;
- Bandit para Python;
- ESLint security plugins para JavaScript.

A OWASP mantém listas de ferramentas de análise de código-fonte, incluindo ferramentas comerciais e open source.

## Trivy e container scanning

Trivy é uma ferramenta muito usada para escanear containers, dependências, IaC, secrets e SBOM.

A documentação do Trivy informa que ele detecta automaticamente pacotes de sistema operacional ao escanear imagens de container e realiza detecção de vulnerabilidades com base no sistema operacional identificado.

Exemplo:

```bash
trivy image meucatalogomusical-api:1.0.0
```

Ele pode encontrar:

```text
CVE em pacote Debian/Ubuntu/Alpine
CVE em biblioteca Java
Segredo em imagem
Misconfiguration
```

Isso complementa o SCA Maven.

Uma aplicação Java pode estar com `pom.xml` limpo, mas rodar em uma imagem base vulnerável.

## Secret scanning

Secret scanning procura segredos em código e histórico.

Exemplos:

- API key;
- token GitHub;
- senha;
- private key;
- JWT;
- AWS access key;
- client secret;
- connection string.

Exemplo de segredo acidental:

```yaml
integracoes:
  backstage:
    client-secret: "SECRET-NAO-DEVERIA-ESTAR-AQUI"
```

Ferramentas:

- GitHub Secret Scanning;
- Gitleaks;
- TruffleHog;
- Snyk;
- Trivy secret scanning;
- detect-secrets.

O ideal é usar secrets por variável de ambiente, vault ou secret manager, nunca versionar em repositório.

## Como ferramentas validam se uma vulnerabilidade foi corrigida?

Depende da ferramenta.

### SAST

Roda novamente e verifica se o fluxo desapareceu ou se agora passa por sanitizer reconhecido.

Antes:

```text
@RequestParam nomeArquivo -> Files.write
```

Depois:

```text
MultipartFile -> nome interno gerado -> SafePathResolver -> Files.write
```

### SCA

Roda novamente e verifica se a versão vulnerável saiu da árvore.

Antes:

```text
biblioteca 1.2.0 vulnerável
```

Depois:

```text
biblioteca 1.2.3 corrigida
```

### DAST

Roda novamente e observa comportamento.

Antes:

```http
sem Strict-Transport-Security
```

Depois:

```http
Strict-Transport-Security: max-age=31536000
```

### Secret scanning

Roda novamente e verifica se o segredo não está mais presente.

Mas atenção: remover segredo do último commit pode não limpar histórico. Em muitos casos, é preciso revogar o segredo.

## Exemplo completo: do código ao scanner

Funcionalidade:

```text
Importar capa de álbum da banda Saxon.
```

Código inseguro:

```java
@PostMapping("/albuns/{albumId}/capa")
public void upload(
        @PathVariable Long albumId,
        @RequestParam String nomeArquivo,
        @RequestParam MultipartFile arquivo
) {
    Path destino = Path.of("/var/app/uploads").resolve(nomeArquivo);

    try {
        Files.write(destino, arquivo.getBytes());
    } catch (IOException ex) {
        throw new IllegalStateException("Falha ao salvar arquivo.", ex);
    }
}
```

### O que o SAST vê

```text
Source:
@RequestParam String nomeArquivo

Sink:
Files.write

Finding:
Path Traversal
```

### O que o DAST pode ver

Se houver upload ativo em ambiente de teste, DAST pode tentar entradas anômalas.

Mas talvez não consiga explorar sem autenticação ou sem entender o fluxo.

### O que o SCA vê

Nada sobre esse bug, porque ele está no código próprio, não em dependência.

### Correção

```java
@PostMapping("/albuns/{albumId}/capa")
public CapaResponse upload(
        @PathVariable Long albumId,
        @RequestParam MultipartFile arquivo
) {
    String nomeInterno = fileStorage.salvar(arquivo);

    return new CapaResponse(albumId, nomeInterno);
}
```

```java
@Component
public class FileStorage {

    private final Path baseDirectory = Path.of("/var/app/uploads")
            .toAbsolutePath()
            .normalize();

    public String salvar(MultipartFile arquivo) {
        String nomeInterno = UUID.randomUUID() + ".jpg";
        Path destino = baseDirectory.resolve(nomeInterno).normalize();

        if (!destino.startsWith(baseDirectory)) {
            throw new IllegalArgumentException("Caminho inválido.");
        }

        try {
            Files.createDirectories(baseDirectory);
            Files.write(destino, arquivo.getBytes());
            return nomeInterno;
        } catch (IOException ex) {
            throw new IllegalStateException("Falha ao salvar arquivo.", ex);
        }
    }
}
```

Agora o usuário não controla o caminho.

## Por que a ferramenta ainda pode apontar depois da correção?

Porque ela pode não reconhecer a correção.

Exemplo:

```java
Path destino = safePathResolver.resolve(nomeInterno);
Files.write(destino, arquivo.getBytes());
```

Se o scanner não sabe que `safePathResolver.resolve` valida o path, ele pode continuar apontando.

Nesse caso:

- crie testes;
- documente source/sink;
- mostre validação;
- peça revisão AppSec;
- configure sanitizer customizado se a ferramenta permitir;
- evite gambiarra anti-scanner.

## Ferramenta não substitui triagem

Ferramentas encontram sinais.

A triagem decide.

Exemplo:

```text
SAST:
Privacy Violation

Triagem:
client_secret sai apenas para token endpoint OAuth HTTPS.
Sem logs.
Resposta filtrada.
Bloqueio de reflexão.
Teste cobrindo segredo refletido.
Classificação: risco mitigado / Not Exploitable.
```

Esse tipo de decisão exige conhecimento humano.

Ferramenta ajuda, mas não é juiz final.

## Boas práticas para usar ferramentas de AppSec

### 1. Rode cedo

Quanto mais cedo o finding aparece, mais barato é corrigir.

### 2. Rode em pull request

O desenvolvedor vê o problema antes do merge.

### 3. Tenha baseline

Projetos legados podem ter muitos findings. Defina baseline e impeça novos problemas.

### 4. Configure severity gate

Bloqueie o que realmente precisa bloquear.

### 5. Evite ruído excessivo

Ferramenta com muito falso positivo perde credibilidade.

### 6. Ajuste regras corporativas

Frameworks internos precisam de sources, sinks e sanitizers customizados.

### 7. Combine ferramentas

SAST, SCA e DAST cobrem ângulos diferentes.

### 8. Gere evidência

Relatório, teste, scan novo e justificativa.

### 9. Treine o time

Desenvolvedor precisa entender source, sink, CVE, taint flow e false positive.

### 10. Não corrija só para “ficar verde”

Corrija risco real.

## Erros comuns

### 1. Achar que ferramenta encontra tudo

Nenhuma ferramenta encontra tudo.

### 2. Ignorar tudo como falso positivo

Isso deixa risco real aberto.

### 3. Corrigir sem entender

Pode quebrar regra de negócio.

### 4. Usar SCA sem olhar transitivas

Vulnerabilidade pode vir de dependência indireta.

### 5. Usar DAST em produção sem cuidado

Scans ativos podem alterar dados.

### 6. Não configurar autenticação no DAST

A ferramenta só verá páginas públicas.

### 7. Não versionar exceções

Exceção sem prazo vira dívida invisível.

### 8. Não rodar novo scan

Sem novo scan, não há evidência de correção.

### 9. Não ajustar para frameworks internos

Ferramentas podem não entender validadores próprios.

### 10. Usar muitas ferramentas sem processo

Mais ferramentas sem triagem geram mais ruído, não mais segurança.

## Decisões de engenharia

## Quando usar SAST?

Use SAST para encontrar problemas no código antes de executar a aplicação.

Ideal para:

- PR;
- branch;
- pipeline;
- revisão de código;
- legado;
- correção SAST corporativa.

## Quando usar SCA?

Use SCA sempre que houver dependências de terceiros.

Ideal para:

- Maven;
- Gradle;
- npm;
- Python;
- Docker;
- SBOM;
- monitoramento contínuo.

## Quando usar DAST?

Use DAST quando a aplicação estiver rodando em ambiente controlado.

Ideal para:

- homologação;
- staging;
- pré-produção;
- testes recorrentes;
- validação de headers;
- comportamento real.

## Quando usar secret scanning?

Use sempre em repositórios.

Ideal para:

- bloquear segredo em PR;
- detectar segredo em histórico;
- proteger tokens;
- reduzir incidentes.

## Quando usar container scanning?

Use se a aplicação roda em container.

Ideal para:

- imagem base;
- pacotes de SO;
- JRE;
- CVEs de sistema;
- Kubernetes;
- CI/CD.

## Onde isso aparece em sistemas reais

Ferramentas de AppSec aparecem em:

- GitHub Actions;
- Jenkins;
- GitLab CI;
- Azure DevOps;
- Bitbucket Pipelines;
- IDE;
- PR;
- quality gate;
- release gate;
- auditoria;
- AppSec corporativo;
- pipelines de container;
- esteiras DevSecOps.

Em projetos Java/Spring, o fluxo comum é:

```text
Pull Request
    -> build
    -> testes
    -> SAST
    -> SCA
    -> secret scanning
    -> merge
    -> deploy em staging
    -> DAST
    -> deploy produção
    -> monitoramento contínuo
```

## Checklist prático

### SAST

- A ferramenta entende Java/Spring?
- Reconhece controllers?
- Reconhece sources?
- Reconhece sinks?
- Reconhece validators internos?
- Permite customizar regras?
- Roda no PR?
- Gera path de fluxo?
- Permite triagem?
- Tem integração com IDE?

### SCA

- Lê `pom.xml` e `build.gradle`?
- Mostra dependências transitivas?
- Mostra CVEs?
- Mostra versão corrigida?
- Mostra caminho da dependência?
- Gera SBOM?
- Monitora novas CVEs?
- Tem política de exceção?
- Distingue runtime e test?
- Indica reachability?

### DAST

- Roda em ambiente autorizado?
- Tem autenticação configurada?
- Tem escopo limitado?
- Evita dados reais?
- Gera evidência HTTP?
- Detecta headers?
- Faz crawl adequado?
- Faz scan ativo com cuidado?
- Permite recorrência?
- Integra com CI/CD?

### Secret scanning

- Roda em PR?
- Roda no histórico?
- Bloqueia tokens conhecidos?
- Orienta revogação?
- Cobre arquivos YAML/properties?
- Cobre Dockerfile?
- Cobre scripts?

### Operação

- Quem faz triagem?
- Quem aprova falso positivo?
- Quem aceita risco?
- Qual SLA por severidade?
- Onde ficam evidências?
- Como exceções expiram?
- Como novos findings são acompanhados?

## Resumo final

Ferramentas de AppSec não “adivinham” vulnerabilidades.

Elas aplicam modelos.

SAST analisa código, procurando fluxos entre sources e sinks, padrões inseguros, ausência de sanitização e uso de APIs sensíveis.

SCA analisa dependências, versões, transitivas, CVEs, licenças, SBOM e bases de vulnerabilidade.

DAST analisa a aplicação rodando, observando comportamento real, headers, respostas, formulários, parâmetros e erros.

Ferramentas como **Veracode**, **Checkmarx**, **CodeQL**, **SonarQube**, **Semgrep**, **Snyk**, **OWASP Dependency-Check**, **Dependency-Track**, **OWASP ZAP**, **Burp Suite DAST** e **Trivy** olham para partes diferentes do risco.

A principal lição é:

> Ferramenta aponta sinais. Engenharia transforma sinais em decisão técnica.

## Perguntas de revisão

### 1. O que é SAST?

SAST é análise estática de segurança. Ela analisa código-fonte, bytecode ou artefatos sem executar a aplicação.

### 2. O que é SCA?

SCA é análise de composição de software. Ela identifica dependências, bibliotecas, versões, CVEs, licenças e riscos de componentes.

### 3. O que é DAST?

DAST é análise dinâmica. Ela testa a aplicação em execução por meio de URLs, formulários, APIs, headers e respostas.

### 4. Como SAST encontra vulnerabilidades?

Ele usa regras, queries, data flow, taint analysis, sources, sinks, sanitizers e modelos de frameworks.

### 5. Como SCA encontra vulnerabilidades?

Ele identifica dependências diretas e transitivas e cruza as versões com bases de vulnerabilidades, CVEs, advisories e bancos próprios.

### 6. Como DAST encontra vulnerabilidades?

Ele interage com a aplicação rodando, envia requisições, observa respostas e compara comportamentos com padrões de vulnerabilidade.

### 7. O que é source?

É o ponto onde dado externo ou não confiável entra na aplicação.

### 8. O que é sink?

É uma operação sensível, como query SQL, escrita de arquivo, chamada HTTP outbound, log ou resposta HTML.

### 9. O que é sanitizer?

É um controle que torna o dado seguro para determinado contexto.

### 10. Por que existem falsos positivos?

Porque a ferramenta trabalha por modelos e pode não entender regra de negócio, validação customizada ou configuração real.

### 11. Veracode e Checkmarx fazem a mesma coisa?

Ambas possuem soluções SAST e SCA, mas cada uma tem arquitetura, engine, relatórios, regras e modelo de priorização próprios.

### 12. SAST substitui DAST?

Não. SAST vê código. DAST vê comportamento em runtime.

### 13. SCA substitui SAST?

Não. SCA vê dependências. SAST vê código próprio.

### 14. Ferramenta open source basta?

Depende do contexto. Ferramentas open source são excelentes, mas empresas maiores podem precisar de governança, integração, relatórios, SLA, suporte e compliance.

### 15. Qual é a principal regra?

Não trate ferramenta como oráculo nem como burocracia. Use como instrumento de engenharia, com triagem e evidência.

## Referências

- [OWASP — Static Code Analysis](https://community.owasp.org/controls/Static_Code_Analysis)
- [OWASP — Source Code Analysis Tools](https://community.owasp.org/Source_Code_Analysis_Tools)
- [Veracode Docs — Upload and Scan](https://docs.veracode.com/r/Veracode_Upload_and_Scan)
- [Veracode Docs — Software Composition Analysis](https://docs.veracode.com/r/Software_Composition_Analysis)
- [Veracode Docs — SCA Agent-Based Scan](https://docs.veracode.com/r/Agent_Based_Scans)
- [Checkmarx Docs — SAST Overview](https://docs.checkmarx.com/en/34965-46311-checkmarx-sast-overview.html)
- [Checkmarx Docs — SAST Scanner](https://docs.checkmarx.com/en/34965-324470-sast-scanner.html)
- [Checkmarx Docs — SCA Scanner](https://docs.checkmarx.com/en/34965-322318-sca-scanner.html)
- [GitHub Docs — Code scanning with CodeQL](https://docs.github.com/en/code-security/concepts/code-scanning/codeql/codeql-code-scanning)
- [CodeQL Docs — Analyzing data flow in Java and Kotlin](https://codeql.github.com/docs/codeql-language-guides/analyzing-data-flow-in-java/)
- [CodeQL Docs — About data flow analysis](https://codeql.github.com/docs/writing-codeql-queries/about-data-flow-analysis/)
- [SonarQube Docs — Security-related rules](https://docs.sonarsource.com/sonarqube-server/2025.1/user-guide/rules/security-related-rules)
- [Semgrep Docs — Glossary](https://semgrep.dev/docs/writing-rules/glossary)
- [Snyk Docs — Snyk CLI for Open Source](https://docs.snyk.io/developer-tools/snyk-cli/scan-and-maintain-projects-using-the-cli/snyk-cli-for-open-source)
- [Snyk Docs — snyk test](https://docs.snyk.io/developer-tools/snyk-cli/commands/test)
- [OWASP Dependency-Check](https://owasp.org/projects/dependency-check)
- [OWASP Dependency-Track](https://owasp.org/projects/dependency-track)
- [OWASP ZAP — Getting Started](https://www.zaproxy.org/getting-started/)
- [OWASP ZAP — Passive Scanner](https://www.zaproxy.org/docs/desktop/addons/passive-scanner/)
- [PortSwigger — Burp Scanner](https://portswigger.net/burp/documentation/scanner)
- [PortSwigger — Auditing](https://portswigger.net/burp/documentation/scanner/auditing)
- [Trivy Docs — Vulnerability Scanning](https://trivy.dev/docs/latest/guide/scanner/vulnerability/)
- [Oracle JDK 25 Documentation](https://docs.oracle.com/en/java/javase/25/)
- [Oracle Java Downloads — JDK 25 LTS](https://www.oracle.com/java/technologies/downloads/)