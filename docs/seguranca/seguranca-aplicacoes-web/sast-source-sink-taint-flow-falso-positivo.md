# SAST na prática: Source, Sink, Taint Flow e falso positivo

SAST significa Static Application Security Testing, ou Teste Estático de Segurança de Aplicações.

Na prática, é uma forma de analisar o código-fonte, bytecode ou artefatos da aplicação sem precisar executar o sistema. A ferramenta examina o código procurando padrões que podem indicar vulnerabilidades.

Ferramentas SAST costumam apontar problemas como:

- SQL Injection;
- Cross-Site Scripting;
- Server-Side Request Forgery;
- Path Traversal;
- exposição de dados sensíveis em logs;
- envio de dados sensíveis por canal inseguro;
- uso de entrada externa em loops;
- headers de segurança ausentes;
- uso inseguro de arquivos;
- fluxos suspeitos de credenciais.

Mas usar SAST bem não significa apenas rodar o scanner e corrigir tudo mecanicamente.

O ponto mais importante é aprender a interpretar o finding.

Um finding normalmente mostra:

- de onde o dado veio;
- por onde ele passou;
- onde ele chegou;
- por que isso pode ser perigoso;
- qual vulnerabilidade a ferramenta acredita existir;
- qual severidade foi atribuída.

Esses elementos geralmente aparecem com nomes como:

- source;
- sink;
- taint flow;
- query;
- result state;
- severity;
- remediation.

Saber ler isso evita dois extremos perigosos:

1. ignorar vulnerabilidades reais achando que “é coisa do scanner”;
2. alterar código sem necessidade apenas para “calar” a ferramenta.

O objetivo deste artigo é explicar como interpretar findings SAST de forma técnica, defensiva e profissional.

## O problema real que SAST resolve

Desenvolvedores escrevem código todos os dias. Em uma aplicação web, esse código recebe dados externos, chama APIs, acessa banco de dados, grava arquivos, gera respostas HTTP, registra logs e integra com sistemas parceiros.

Cada uma dessas ações pode criar uma falha de segurança se os dados forem usados de forma incorreta.

O problema é que muitas vulnerabilidades não aparecem claramente em um teste funcional.

Um teste funcional pode confirmar que o cadastro de uma resenha da banda **Gojira** funciona. Mas talvez ele não perceba que:

- o campo de texto pode ser exibido sem escape;
- o nome de arquivo pode controlar um path;
- uma URL externa pode ser manipulada;
- uma credencial pode aparecer no log;
- uma entrada pode controlar um loop grande demais;
- um endpoint retorna JSON sem media type explícito;
- um serviço chama HTTP em vez de HTTPS.

O SAST ajuda justamente porque ele olha para fluxos de código que humanos podem deixar passar em uma revisão comum.

Ele não substitui o desenvolvedor. Ele aumenta a visibilidade.

## Antes de entrar nos detalhes: uma ideia simples

Pense na aplicação fictícia `meucatalogomusical.com`.

Ela permite que usuários criem resenhas de álbuns de Metal e Hard Rock.

Um usuário envia uma resenha sobre a banda **Mastodon**:

```text
Esse álbum tem uma produção pesada e progressiva.
```

Esse texto entra na aplicação por uma API.

Depois pode ser:

- validado;
- salvo no banco;
- exibido em uma página;
- enviado para outro serviço;
- registrado em log;
- retornado em JSON;
- usado em uma notificação.

Para o SAST, esse texto pode ser considerado um dado contaminado, ou seja, um dado vindo de uma fonte externa.

A ferramenta tenta responder:

> Esse dado externo chega a algum ponto sensível sem controle adequado?

Se chega a uma resposta HTML sem escape, pode ser XSS.

Se chega a uma query SQL concatenada, pode ser SQL Injection.

Se chega a uma chamada HTTP que controla o destino, pode ser SSRF.

Se chega a um caminho de arquivo, pode ser Path Traversal.

Se chega a um log contendo dado sensível, pode ser Privacy Violation.

SAST é, em grande parte, uma tentativa automatizada de rastrear esse caminho.

## Definição técnica

SAST é uma técnica de teste de segurança que analisa a aplicação sem executá-la, procurando padrões vulneráveis no código.

Em uma análise de fluxo de dados, a ferramenta procura principalmente:

- **source**: ponto onde um dado entra ou nasce;
- **sink**: ponto sensível onde esse dado é usado;
- **taint flow**: caminho percorrido pelo dado entre source e sink;
- **sanitizer**: controle que torna o dado seguro para determinado contexto;
- **validator**: controle que verifica formato, faixa, tipo ou regra esperada;
- **finding**: resultado reportado pelo scanner;
- **false positive**: finding reportado como vulnerabilidade, mas que não representa uma vulnerabilidade real no contexto analisado;
- **true positive**: finding que representa uma vulnerabilidade real;
- **not exploitable**: finding cujo fluxo existe parcialmente, mas não é explorável nas condições reais do sistema.

Esses conceitos são essenciais para interpretar relatórios SAST.

## Source

Source é o ponto de origem do dado analisado.

Normalmente, o source é uma entrada externa ou um dado não confiável.

Exemplos comuns em Java/Spring:

```java
@RequestParam String termo
```

```java
@PathVariable Long id
```

```java
@RequestBody NovaResenhaRequest request
```

```java
MultipartFile arquivo
```

```java
request.getHeader("X-Partner-Id")
```

```java
cookie.getValue()
```

```java
Files.newInputStream(path)
```

```java
s3Client.getObject(...)
```

Nem todo source é necessariamente malicioso. Ele é apenas uma origem que não deve ser tratada como automaticamente confiável.

Um erro comum é pensar:

> “Esse endpoint é interno, então a entrada é confiável.”

Mesmo em APIs internas, os dados ainda atravessam fronteiras de confiança. Autenticação reduz superfície, mas não transforma entrada externa em dado seguro para qualquer uso.

## Sink

Sink é um ponto sensível do código.

É uma operação que pode causar impacto de segurança se receber dado não confiável sem controle adequado.

Exemplos de sinks:

### Consulta SQL

```java
jdbcTemplate.query(sql, rowMapper);
```

### Escrita em arquivo

```java
Files.write(path, conteudo);
```

### Chamada HTTP outbound

```java
httpClient.send(request, BodyHandlers.ofString());
```

### Renderização HTML sem escape

```html
<div th:utext="${comentario.conteudo}"></div>
```

### Log

```java
log.info("Dados recebidos: {}", request);
```

### Loop

```java
for (int i = 0; i < totalLinhas; i++) {
    processarLinha(i);
}
```

### Resposta HTTP

```java
return ResponseEntity.ok(dados);
```

Um sink não é ruim por si só. Consultar banco, gravar arquivo, chamar API, registrar log e retornar resposta são atividades normais de uma aplicação.

O risco aparece quando um dado não confiável chega ao sink sem o controle correto para aquele contexto.

## Taint

Taint significa contaminação.

Quando o SAST identifica que um valor veio de uma fonte externa, ele pode marcar esse valor como tainted.

Exemplo:

```java
@GetMapping("/buscar")
public List<BandaResponse> buscar(@RequestParam String nome) {
    return bandaService.buscar(nome);
}
```

O valor `nome` veio da requisição HTTP. Portanto, a ferramenta pode considerá-lo tainted.

Se esse valor for passado por vários métodos, a ferramenta tenta acompanhar o fluxo:

```text
@RequestParam nome
    -> controller
    -> service
    -> repository
    -> query SQL
```

Se chegar a uma query SQL concatenada, o scanner pode apontar SQL Injection.

## Taint Flow

Taint flow é o caminho entre o source e o sink.

Exemplo conceitual:

```text
source: @RequestParam nomeArquivo
    -> AnexoController.upload(...)
    -> AnexoService.salvar(...)
    -> FileUtil.write(...)
sink: Files.write(...)
```

O SAST tenta reconstruir esse caminho analisando chamadas de método, variáveis, objetos, retornos e transformações.

Em relatórios de ferramentas como Checkmarx, esse caminho costuma aparecer como uma cadeia de arquivos, métodos, linhas e colunas.

Saber ler essa cadeia é mais importante do que olhar apenas o nome da vulnerabilidade.

## Exemplo inseguro: possível Path Traversal

Imagine uma funcionalidade de upload de imagem de capa para álbuns da banda **Epica**.

Exemplo inseguro:

```java
package com.meucatalogomusical.albuns.api;

import org.springframework.web.bind.annotation.*;
import org.springframework.web.multipart.MultipartFile;

@RestController
@RequestMapping("/api/albuns")
public class CapaAlbumController {

    private final CapaAlbumService capaAlbumService;

    public CapaAlbumController(CapaAlbumService capaAlbumService) {
        this.capaAlbumService = capaAlbumService;
    }

    @PostMapping("/{albumId}/capa")
    public void enviarCapa(
            @PathVariable Long albumId,
            @RequestParam String nomeArquivo,
            @RequestParam MultipartFile arquivo
    ) {
        capaAlbumService.salvarCapa(albumId, nomeArquivo, arquivo);
    }
}
```

Service:

```java
package com.meucatalogomusical.albuns.application;

import org.springframework.stereotype.Service;
import org.springframework.web.multipart.MultipartFile;

@Service
public class CapaAlbumService {

    private final FileStorage fileStorage;

    public CapaAlbumService(FileStorage fileStorage) {
        this.fileStorage = fileStorage;
    }

    public void salvarCapa(Long albumId, String nomeArquivo, MultipartFile arquivo) {
        fileStorage.salvar(nomeArquivo, arquivo);
    }
}
```

Storage inseguro:

```java
package com.meucatalogomusical.albuns.infra;

import org.springframework.stereotype.Component;
import org.springframework.web.multipart.MultipartFile;

import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Path;

@Component
public class FileStorage {

    private final Path diretorioBase = Path.of("/var/app/uploads");

    public void salvar(String nomeArquivo, MultipartFile arquivo) {
        try {
            Path destino = diretorioBase.resolve(nomeArquivo);
            Files.write(destino, arquivo.getBytes());
        } catch (IOException ex) {
            throw new IllegalStateException("Não foi possível salvar o arquivo.", ex);
        }
    }
}
```

O SAST poderia montar este fluxo:

```text
source: @RequestParam nomeArquivo
    -> CapaAlbumController.enviarCapa
    -> CapaAlbumService.salvarCapa
    -> FileStorage.salvar
sink: Files.write
```

A preocupação é que `nomeArquivo` vem da requisição e participa da construção de um caminho de arquivo.

Mesmo que a aplicação espere algo simples como:

```text
capa-epica.jpg
```

o código não garante que esse valor seja apenas um nome de arquivo seguro.

## Corrigindo com identificador interno e normalização

Uma abordagem melhor é não confiar no nome enviado pelo usuário como caminho de escrita.

Exemplo defensivo:

```java
package com.meucatalogomusical.albuns.infra;

import org.springframework.stereotype.Component;
import org.springframework.web.multipart.MultipartFile;

import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Path;
import java.util.Set;
import java.util.UUID;

@Component
public class FileStorage {

    private static final Set<String> EXTENSOES_PERMITIDAS = Set.of("jpg", "jpeg", "png", "webp");

    private final Path diretorioBase = Path.of("/var/app/uploads").toAbsolutePath().normalize();

    public String salvarImagem(MultipartFile arquivo) {
        String extensao = obterExtensaoPermitida(arquivo.getOriginalFilename());
        String nomeInterno = UUID.randomUUID() + "." + extensao;

        Path destino = diretorioBase.resolve(nomeInterno).normalize();

        if (!destino.startsWith(diretorioBase)) {
            throw new IllegalArgumentException("Caminho de arquivo inválido.");
        }

        try {
            Files.createDirectories(diretorioBase);
            Files.write(destino, arquivo.getBytes());
            return nomeInterno;
        } catch (IOException ex) {
            throw new IllegalStateException("Não foi possível salvar o arquivo.", ex);
        }
    }

    private String obterExtensaoPermitida(String nomeOriginal) {
        if (nomeOriginal == null || nomeOriginal.isBlank()) {
            throw new IllegalArgumentException("Nome do arquivo é obrigatório.");
        }

        int ultimoPonto = nomeOriginal.lastIndexOf(".");

        if (ultimoPonto < 0 || ultimoPonto == nomeOriginal.length() - 1) {
            throw new IllegalArgumentException("Extensão do arquivo é obrigatória.");
        }

        String extensao = nomeOriginal.substring(ultimoPonto + 1).toLowerCase();

        if (!EXTENSOES_PERMITIDAS.contains(extensao)) {
            throw new IllegalArgumentException("Extensão de arquivo não permitida.");
        }

        return extensao;
    }
}
```

Aqui existem várias decisões defensivas:

- o usuário não controla o nome final do arquivo;
- o nome interno é gerado pela aplicação;
- o diretório base é absoluto e normalizado;
- o destino final é normalizado;
- o destino precisa continuar dentro do diretório base;
- a extensão é validada por lista permitida;
- o código não usa o nome original como path final.

Esse é um exemplo de correção real, não apenas uma tentativa de esconder o fluxo do scanner.

## Sanitizer

Sanitizer é um controle que torna um dado seguro para determinado contexto.

Exemplos:

- escape HTML antes de renderizar texto em página;
- parametrização antes de enviar valor para SQL;
- validação de URI antes de chamada outbound;
- normalização e verificação de path antes de escrita em arquivo;
- lista permitida para nomes de colunas;
- remoção de campos secretos antes de devolver resposta pública.

Um ponto essencial:

> Um sanitizer só é sanitizer para um contexto específico.

Escapar HTML não torna uma URL segura contra SSRF.

Validar HTTPS não garante que o host é autorizado.

Limitar tamanho de string não torna uma query SQL segura.

Normalizar path não resolve XSS.

Por isso, ao ler um finding SAST, não basta perguntar:

> “Existe alguma validação?”

A pergunta correta é:

> “Existe um controle adequado para este sink e para este tipo de risco?”

## Validator não é sempre sanitizer

Validador e sanitizer são conceitos próximos, mas não idênticos.

Um validator verifica se o dado segue uma regra.

Exemplo:

```java
@NotBlank
@Size(max = 120)
String titulo
```

Isso valida obrigatoriedade e tamanho.

Mas esse campo ainda precisa ser escapado se for exibido em HTML.

Outro exemplo:

```java
URI uri = URI.create(valor);
```

Isso apenas cria uma URI. Não garante que ela seja HTTPS, absoluta, com host permitido e sem redirecionamento perigoso.

Validação ajuda, mas precisa ser compatível com o sink.

## Exemplo: possível SSRF

Imagine que `meucatalogomusical.com` tenha uma integração para importar informações de uma API parceira sobre a banda **Sabaton**.

Exemplo perigoso:

```java
package com.meucatalogomusical.integracoes.api;

import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/importacoes")
public class ImportacaoController {

    private final ImportacaoService importacaoService;

    public ImportacaoController(ImportacaoService importacaoService) {
        this.importacaoService = importacaoService;
    }

    @PostMapping("/banda")
    public String importar(@RequestParam String urlParceiro) {
        return importacaoService.importar(urlParceiro);
    }
}
```

Service:

```java
package com.meucatalogomusical.integracoes.application;

import org.springframework.stereotype.Service;
import org.springframework.web.client.RestTemplate;

@Service
public class ImportacaoService {

    private final RestTemplate restTemplate;

    public ImportacaoService(RestTemplate restTemplate) {
        this.restTemplate = restTemplate;
    }

    public String importar(String urlParceiro) {
        return restTemplate.getForObject(urlParceiro, String.class);
    }
}
```

Possível finding:

```text
Query: SSRF
Source: @RequestParam urlParceiro
Sink: RestTemplate.getForObject
```

A preocupação é que o usuário controla o destino da requisição server-side.

Uma correção melhor seria não aceitar URL livre.

## Correção com allowlist de parceiros

Exemplo defensivo:

```java
package com.meucatalogomusical.integracoes.domain;

public enum ParceiroMusical {
    CATALOGO_OFICIAL,
    API_EVENTOS
}
```

```java
package com.meucatalogomusical.integracoes.application;

import com.meucatalogomusical.integracoes.domain.ParceiroMusical;
import org.springframework.stereotype.Component;

import java.net.URI;
import java.util.Map;

@Component
public class ParceiroUriResolver {

    private final Map<ParceiroMusical, URI> endpoints = Map.of(
            ParceiroMusical.CATALOGO_OFICIAL, URI.create("https://api-parceira.example/catalogo"),
            ParceiroMusical.API_EVENTOS, URI.create("https://api-eventos.example/shows")
    );

    public URI resolver(ParceiroMusical parceiro) {
        URI uri = endpoints.get(parceiro);

        if (uri == null) {
            throw new IllegalArgumentException("Parceiro não autorizado.");
        }

        if (!"https".equalsIgnoreCase(uri.getScheme()) || uri.getHost() == null) {
            throw new IllegalStateException("Endpoint do parceiro mal configurado.");
        }

        return uri;
    }
}
```

```java
package com.meucatalogomusical.integracoes.application;

import com.meucatalogomusical.integracoes.domain.ParceiroMusical;
import org.springframework.stereotype.Service;
import org.springframework.web.client.RestTemplate;

import java.net.URI;

@Service
public class ImportacaoService {

    private final RestTemplate restTemplate;
    private final ParceiroUriResolver parceiroUriResolver;

    public ImportacaoService(
            RestTemplate restTemplate,
            ParceiroUriResolver parceiroUriResolver
    ) {
        this.restTemplate = restTemplate;
        this.parceiroUriResolver = parceiroUriResolver;
    }

    public String importar(ParceiroMusical parceiro) {
        URI endpoint = parceiroUriResolver.resolver(parceiro);
        return restTemplate.getForObject(endpoint, String.class);
    }
}
```

Agora o cliente não envia a URL final.

Ele escolhe um parceiro conhecido, e a aplicação resolve o endpoint a partir de configuração controlada.

Essa é uma mudança de desenho mais segura.

## Falso positivo

Falso positivo é um finding que a ferramenta reporta como vulnerabilidade, mas que, após análise técnica, não representa vulnerabilidade real no contexto.

Isso pode acontecer porque SAST trabalha por aproximação.

A ferramenta nem sempre entende:

- regra de negócio;
- configuração real;
- autenticação corporativa;
- contratos legados;
- serializers;
- frameworks;
- headers HTTP;
- media type;
- consumidores reais;
- controles customizados;
- validadores internos;
- branches inalcançáveis;
- limites matemáticos do algoritmo.

Mas cuidado: chamar algo de falso positivo exige evidência.

Não basta dizer:

> “Isso nunca vai acontecer.”

Uma boa triagem precisa demonstrar por que o fluxo não é explorável.

## Exemplos de falso positivo

### 1. Sink HTML inexistente

A ferramenta aponta XSS, mas o endpoint é uma API JSON server-to-server, com:

- `Content-Type: application/json`;
- `produces = application/json`;
- `X-Content-Type-Options: nosniff`;
- rejeição de `Accept: text/html`;
- ausência de frontend que injete o campo em DOM.

Nesse caso, pode existir dado não confiável retornando ao cliente, mas não existe contexto HTML ou JavaScript onde o XSS se concretize.

Mesmo assim, o hardening é útil: tornar o contrato HTTP explícito reduz ambiguidade para humanos, clientes e scanners.

### 2. Loop com limite matemático demonstrável

A ferramenta aponta que entrada externa controla um loop.

Após análise, o valor que chega ao loop é derivado de dois dígitos decimais e só pode produzir `1` ou `2`.

Nesse caso, o risco de número arbitrário de iterações pode não existir.

Ainda assim, pode fazer sentido deixar o limite explícito no código para facilitar manutenção e reduzir alertas futuros.

### 3. Credencial enviada em fluxo funcional autorizado

A ferramenta aponta Privacy Violation porque `client_id` e `client_secret` são enviados para fora da aplicação.

Mas o fluxo real é autenticação OAuth `client_credentials`, com:

- endpoint vindo de configuração;
- HTTPS obrigatório;
- ausência de log de credenciais;
- resposta pública sem reflexão de segredo;
- falha fechada se o parceiro ecoar segredo;
- contrato funcional preservado.

Nesse caso, a saída da credencial não é automaticamente uma vulnerabilidade. Ela é parte do protocolo. O risco precisa ser avaliado pelos controles existentes.

### 4. Controle customizado não reconhecido

A aplicação possui um método como:

```java
CredentialReflectionGuard.verifyAndReturn(response, credentials);
```

Esse método remove ou rejeita campos secretos.

Mas o scanner não sabe que ele é um sanitizer para aquela query.

Nesse caso, pode ser necessário:

- documentar a evidência;
- criar testes;
- justificar triagem;
- pedir ao time de AppSec para reconhecer o método como sanitizador específico;
- evitar gambiarras artificiais só para quebrar o fluxo.

## Verdadeiro positivo

True positive é um finding que representa uma vulnerabilidade real ou uma fragilidade que precisa ser corrigida.

Exemplo:

```java
String sql = "SELECT * FROM bandas WHERE nome = '" + nome + "'";
```

Se `nome` vem da requisição, isso é um problema real.

Outro exemplo:

```java
Path destino = diretorioBase.resolve(nomeArquivo);
Files.write(destino, bytes);
```

Se `nomeArquivo` vem do usuário e não há normalização, geração de nome interno ou verificação do diretório final, o finding de Path Traversal pode ser real.

Outro exemplo:

```java
log.info("Credenciais recebidas: {}", credentials);
```

Se `credentials` contém segredo, isso é uma exposição real em log.

## Risco mitigado

Às vezes, o finding não desaparece, mas o risco foi mitigado por controles reais.

Exemplo:

- a ferramenta continua apontando SSRF;
- o código agora só aceita URI absoluta HTTPS;
- o host vem de allowlist;
- redirects estão desabilitados;
- há teste cobrindo HTTP, URI relativa e host inválido;
- não há controle do destino pelo usuário.

Nesse caso, pode ser necessário classificar como mitigado ou solicitar ajuste da regra corporativa do scanner.

O ponto importante é: mitigação precisa ser comprovável.

## Not Exploitable

`Not Exploitable` é uma classificação usada quando o finding existe do ponto de vista do scanner, mas não é explorável no contexto real.

Exemplo:

- source existe;
- sink existe;
- a ferramenta liga os dois;
- mas existe uma guarda funcional que impede exploração;
- ou o caminho não é alcançável;
- ou o dado chega ao sink em um contexto que não executa;
- ou a operação é funcionalmente obrigatória e protegida por controles suficientes.

Para classificar como `Not Exploitable`, a justificativa deve responder:

1. Qual é o source?
2. Qual é o sink?
3. Qual fluxo o scanner acredita existir?
4. Qual é o fluxo real?
5. Qual controle impede a exploração?
6. Quais evidências provam isso?
7. Quais testes foram executados?
8. Qual risco residual permanece?

## Não faça correções artificiais

Um erro perigoso é alterar o código apenas para “enganar” o SAST.

Exemplos ruins:

- criar wrapper vazio sem função real;
- serializar e desserializar sem necessidade;
- usar reflexão para esconder fluxo;
- colocar condição inalcançável;
- renomear variável para confundir scanner;
- quebrar contrato público;
- mascarar dado necessário para protocolo;
- remover log operacional útil sem avaliar;
- adicionar escape HTML em dado que nunca será HTML;
- engolir exceções;
- mover código para outro método sem controle real.

Isso pode até fazer o finding desaparecer, mas não melhora a segurança.

A correção correta é implementar controle de segurança real ou documentar tecnicamente por que o finding não é explorável.

## Processo de triagem SAST

Um bom processo de triagem pode seguir estes passos.

### 1. Identificar a query

Primeiro, veja qual vulnerabilidade foi apontada.

Exemplos:

- SSRF;
- Reflected XSS;
- Stored XSS;
- Privacy Violation;
- Absolute Path Traversal;
- Unchecked Input for Loop Condition;
- Cleartext Submission;
- Missing HSTS Header.

A query define a hipótese da ferramenta.

Mas a hipótese precisa ser validada.

### 2. Ler source e sink

Identifique:

- arquivo de origem;
- linha de origem;
- variável de origem;
- arquivo de destino;
- linha de destino;
- operação sensível.

Exemplo:

```text
Source: @RequestParam nomeArquivo
Sink: Files.write
```

Isso já sugere o tipo de análise: entrada externa influenciando escrita em arquivo.

### 3. Reconstruir o fluxo real

Não confie apenas no relatório.

Abra o código e reconstrua:

```text
controller
    -> service
    -> util
    -> sink
```

Verifique:

- o dado é realmente o mesmo?
- há transformação?
- há validação?
- há sanitização?
- há troca de contexto?
- há branch que impede o fluxo?
- o método é realmente chamado?
- a linha está atualizada?
- o scanner está olhando código antigo?

### 4. Identificar a fronteira de confiança

Pergunte:

- o dado vem de usuário externo?
- vem de sistema interno?
- vem de configuração?
- vem de S3?
- vem de arquivo local?
- vem de banco?
- vem de parceiro?
- vem de header?
- vem de cookie?

Dado interno também pode ser não confiável dependendo da fronteira.

Um arquivo de configuração no S3, por exemplo, pode ser confiável para regra de negócio, mas ainda assim precisa ser tratado com cuidado se for retornado para consumidores.

### 5. Avaliar o contexto do sink

O mesmo dado pode ser perigoso ou seguro dependendo do contexto.

Exemplo:

```java
return ResponseEntity.ok(response);
```

Pode ser seguro se:

- é JSON;
- `Content-Type` é explícito;
- não há sniffing;
- nenhum consumidor injeta em HTML.

Pode ser perigoso se:

- retorna HTML;
- o conteúdo é renderizado sem escape;
- o navegador interpreta como script;
- o consumidor insere em DOM.

Contexto importa.

### 6. Procurar controles existentes

Verifique se existem controles reais.

Exemplos:

- parametrização SQL;
- escape HTML contextual;
- validação de URI;
- allowlist de host;
- `Content-Type` explícito;
- `nosniff`;
- limite de linhas;
- limite de arquivo;
- normalização de path;
- verificação `startsWith` no diretório base;
- remoção de campos secretos;
- ausência de log de credenciais;
- HTTPS obrigatório.

### 7. Avaliar explorabilidade

Pergunte:

- um atacante controla o source?
- consegue atingir o sink?
- consegue alterar o comportamento?
- consegue obter impacto?
- há autenticação?
- há autorização?
- o ambiente é interno?
- existe outro controle de infraestrutura?
- o controle está no código ou só em premissa?
- o scanner ignorou algum controle?
- o risco residual é aceitável?

Não use autenticação como desculpa automática.

Um endpoint autenticado ainda pode ter vulnerabilidade.

### 8. Decidir o tratamento

Possíveis decisões:

- corrigir;
- mitigar;
- abrir débito técnico;
- aceitar risco temporário;
- classificar como falso positivo;
- classificar como Not Exploitable;
- solicitar ajuste de query ao time AppSec.

### 9. Criar evidências

Evidências podem incluir:

- testes unitários;
- testes de integração;
- teste manual;
- diff do código;
- logs sem dado sensível;
- resposta HTTP com headers corretos;
- prova de `Content-Type`;
- prova de rejeição de entrada inválida;
- prova de falha fechada;
- explicação source/sink;
- justificativa de risco residual.

Sem evidência, a triagem vira opinião.

## Exemplo de justificativa técnica

Imagine um finding de Reflected XSS em um endpoint JSON.

Justificativa ruim:

```text
Não é XSS porque essa API é interna.
```

Justificativa melhor:

```text
O source indicado é o parâmetro `termo` recebido no controller. O sink indicado é o retorno do método REST. O endpoint não produz HTML: o mapping declara `produces = application/json`, a resposta possui `Content-Type: application/json` e `X-Content-Type-Options: nosniff`. A aplicação não possui frontend que injete esse campo em DOM, e clientes que solicitam exclusivamente `text/html` recebem erro de negociação. Portanto, o dado permanece como valor JSON e não entra em contexto HTML, atributo, JavaScript ou DOM. Classificação proposta: Not Exploitable para XSS, com hardening HTTP aplicado.
```

A segunda justificativa é muito mais forte porque explica:

- source;
- sink;
- contexto;
- controle;
- por que a exploração não ocorre;
- classificação proposta.

## Exemplo de teste para contrato JSON

Controller:

```java
package com.meucatalogomusical.busca.api;

import org.springframework.http.MediaType;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/busca")
public class BuscaController {

    @GetMapping(produces = MediaType.APPLICATION_JSON_VALUE)
    public BuscaResponse buscar(@RequestParam String termo) {
        return new BuscaResponse("Resultados para: " + termo);
    }

    public record BuscaResponse(String mensagem) {
    }
}
```

Teste:

```java
package com.meucatalogomusical.busca.api;

import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.http.MediaType;
import org.springframework.test.web.servlet.MockMvc;

import static org.hamcrest.Matchers.containsString;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;

@WebMvcTest(BuscaController.class)
class BuscaControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @Test
    void deveResponderJsonExplicitamente() throws Exception {
        mockMvc.perform(get("/api/busca")
                        .param("termo", "Lacuna Coil")
                        .accept(MediaType.APPLICATION_JSON))
                .andExpect(status().isOk())
                .andExpect(header().string("Content-Type", containsString("application/json")))
                .andExpect(jsonPath("$.mensagem").value("Resultados para: Lacuna Coil"));
    }

    @Test
    void deveRecusarHtmlQuandoEndpointProduzSomenteJson() throws Exception {
        mockMvc.perform(get("/api/busca")
                        .param("termo", "Lacuna Coil")
                        .accept(MediaType.TEXT_HTML))
                .andExpect(status().isNotAcceptable());
    }
}
```

Esse teste não “resolve XSS” sozinho.

Ele comprova uma parte importante da triagem: o endpoint foi desenhado para JSON e recusa negociação HTML.

## SAST e contexto de negócio

SAST não entende perfeitamente a regra de negócio.

Exemplo:

```java
@PreAuthorize("hasRole('ADMIN')")
@DeleteMapping("/api/admin/resenhas/{id}")
public void remover(@PathVariable Long id) {
    resenhaService.remover(id);
}
```

O scanner pode ver apenas que um `id` externo chega a uma operação sensível.

Mas ele talvez não entenda:

- se o endpoint exige papel administrativo;
- se há auditoria;
- se o service valida ownership;
- se a operação é permitida pela regra de negócio;
- se há soft delete;
- se a ação é reversível.

Por isso, revisão humana continua indispensável.

## SAST e frameworks

Frameworks como Spring ajudam muito, mas também podem confundir scanners.

Exemplos:

- `@Valid` pode ser reconhecido ou não;
- `@PreAuthorize` pode ser reconhecido ou não;
- serialização JSON pode ser entendida ou não;
- filtros de segurança podem estar fora do fluxo analisado;
- sanitizers customizados podem não ser reconhecidos;
- configurações em YAML podem não ser correlacionadas ao código;
- headers definidos em filtros podem não ser associados ao controller.

Por isso, quanto mais explícito for o contrato de segurança no código e nos testes, mais fácil será defender a triagem.

## Boas práticas ao corrigir findings SAST

### Preserve o contrato funcional

Antes de alterar, entenda:

- endpoint;
- request;
- response;
- payload;
- status HTTP;
- headers;
- clientes consumidores;
- regra de negócio;
- compatibilidade retroativa.

Correção de segurança não deve quebrar contrato sem decisão consciente.

### Aplique controle próximo ao sink

Sempre que possível, valide próximo da fronteira perigosa.

Exemplo:

- validar URI imediatamente antes da chamada HTTP;
- verificar path final imediatamente antes da escrita;
- parametrizar no ponto da query;
- escapar no ponto de saída HTML;
- filtrar segredo antes do log;
- limitar loop antes do processamento.

Isso facilita leitura humana e reduz ambiguidade para ferramentas.

### Não confie apenas em premissas externas

Premissas como “é interno”, “tem firewall” ou “só o legado chama” ajudam no risco, mas não substituem controle de código.

Use essas informações como contexto, não como única defesa.

### Escreva testes de segurança

Tests ajudam a provar que a correção é real.

Exemplos:

- HTTP é rejeitado;
- URI relativa é rejeitada;
- host não autorizado é rejeitado;
- arquivo fora do diretório base é rejeitado;
- segredo refletido é bloqueado;
- resposta HTML é recusada;
- loop respeita limite máximo;
- log não contém marcador sensível.

### Documente risco residual

Toda correção pode deixar riscos residuais.

Exemplo:

- API JSON explícita reduz XSS naquele endpoint, mas o consumidor ainda precisa evitar injetar JSON em HTML.
- HTTPS obrigatório protege canal, mas não substitui allowlist de host.
- HMAC no cache evita expor segredo como chave, mas o segredo ainda existe em memória durante o fluxo.
- Limite de linhas evita DoS por planilha enorme, mas ainda pode haver custo de parsing do arquivo.

Risco residual bem documentado ajuda futuras decisões.

## Erros comuns

### 1. Corrigir pelo nome da vulnerabilidade, não pelo fluxo

Ver “XSS” e aplicar escape HTML em qualquer lugar pode ser errado.

Primeiro entenda se existe contexto HTML.

### 2. Classificar falso positivo sem evidência

Dizer “não explora” sem demonstrar source, sink, controle e contexto não é triagem técnica.

### 3. Criar código artificial para quebrar taint

Isso reduz a qualidade do código e pode manter o risco real.

### 4. Ignorar severidade alta automaticamente

Severidade alta exige análise cuidadosa. Mesmo que depois seja falso positivo, precisa de evidência.

### 5. Achar que autenticação elimina vulnerabilidade

Usuários autenticados também podem explorar falhas.

### 6. Não executar novo scan

Depois da correção, rode novo scan no commit exato.

### 7. Não criar testes

Sem teste, o problema pode voltar no próximo refactor.

### 8. Não envolver AppSec quando a query precisa ser ajustada

Se o controle customizado é real, mas o scanner não reconhece, pode ser necessário ajuste controlado da query corporativa.

## Decisões de engenharia

## Quando corrigir no código?

Corrija quando:

- o fluxo é real;
- o usuário controla o source;
- o sink é sensível;
- não há controle adequado;
- o impacto é relevante;
- a correção não quebra contrato ou a quebra é aprovada.

Exemplo: entrada externa controlando path de escrita.

## Quando mitigar?

Mitigue quando:

- o risco existe, mas pode ser reduzido com controle específico;
- não é possível remover totalmente o fluxo por causa de contrato legado;
- há protocolo funcional obrigatório;
- há integração com parceiro;
- a correção completa exige mudança arquitetural maior.

Exemplo: credenciais precisam sair para OAuth, mas somente via HTTPS, sem logs e com prevenção de reflexão.

## Quando classificar como falso positivo?

Classifique como falso positivo quando:

- o fluxo apontado não existe;
- o sink não é realmente perigoso naquele contexto;
- o scanner confundiu retorno e argumento;
- o dado não chega ao ponto indicado;
- o caminho é inalcançável;
- existe prova técnica clara.

## Quando classificar como Not Exploitable?

Use quando:

- o scanner encontrou um caminho plausível;
- mas controles reais impedem exploração;
- o fluxo funcional precisa existir;
- o risco está mitigado;
- há testes e justificativa.

## Quando solicitar ajuste de query?

Solicite ajuste quando:

- o mesmo falso positivo aparece em vários scans;
- há sanitizer customizado robusto;
- a query corporativa não reconhece um padrão legítimo;
- a equipe AppSec concorda com a evidência;
- o ajuste não enfraquece detecção em outros projetos.

## Onde isso aparece em sistemas reais

SAST aparece em:

- pull requests;
- pipelines CI/CD;
- gates de qualidade;
- auditorias;
- esteiras corporativas;
- migração de ferramentas;
- correção de legado;
- preparação para produção;
- revisões de segurança;
- programas DevSecOps.

Em Java/Spring, os findings costumam aparecer em:

- controllers;
- services;
- repositories;
- classes utilitárias;
- filtros HTTP;
- clients REST;
- parsers de arquivos;
- serializers;
- decorators de cache;
- classes de configuração;
- templates HTML;
- integrações com storage;
- integrações com parceiros.

O ponto central é: SAST não analisa apenas “código ruim”. Ele analisa fluxos. Às vezes o problema está no desenho da fronteira de confiança, não em uma linha isolada.

## Checklist prático de triagem SAST

### Identificação

- Qual é a query?
- Qual é a severidade?
- É recorrente?
- O scan foi feito no commit correto?
- As linhas ainda correspondem ao código atual?

### Source

- De onde vem o dado?
- É entrada de usuário?
- É header?
- É arquivo?
- É configuração?
- É banco?
- É parceiro?
- É confiável ou cruza fronteira?

### Sink

- Qual é a operação sensível?
- SQL?
- HTML?
- HTTP outbound?
- Arquivo?
- Log?
- Loop?
- Header?
- Resposta pública?

### Fluxo

- O dado realmente chega ao sink?
- Passa por quais métodos?
- Há transformação?
- Há validação?
- Há sanitização?
- Há branch que bloqueia?
- O caminho é alcançável?

### Controle

- O controle é adequado para o sink?
- Está próximo da fronteira perigosa?
- Falha fechado?
- Possui teste?
- Preserva contrato?
- Reduz risco real ou só engana scanner?

### Decisão

- Corrigir?
- Mitigar?
- Marcar falso positivo?
- Marcar Not Exploitable?
- Aceitar risco?
- Acionar AppSec?
- Criar tarefa de refatoração?

### Evidência

- Há teste automatizado?
- Há teste manual?
- Há print ou log seguro?
- Há resposta HTTP comprovando headers?
- Há justificativa source/sink?
- Há risco residual documentado?

## Resumo final

SAST é uma ferramenta poderosa para encontrar vulnerabilidades no código, mas seu valor depende da capacidade do time de interpretar os findings.

Os conceitos principais são:

- **source**: origem do dado;
- **sink**: operação sensível;
- **taint**: dado considerado não confiável;
- **taint flow**: caminho entre source e sink;
- **sanitizer**: controle que torna o dado seguro para um contexto;
- **validator**: controle que verifica formato, tamanho ou regra;
- **false positive**: finding não vulnerável no contexto real;
- **true positive**: vulnerabilidade real;
- **not exploitable**: finding plausível para o scanner, mas não explorável devido ao contexto e controles reais.

O maior erro é tratar o SAST como oráculo.

O segundo maior erro é ignorá-lo.

A postura correta é técnica:

1. ler o finding;
2. reconstruir o fluxo;
3. entender o contexto;
4. aplicar controle real;
5. preservar contrato funcional;
6. criar testes;
7. documentar evidências;
8. executar novo scan;
9. envolver AppSec quando necessário.

SAST bem usado não é burocracia. É uma ferramenta para melhorar a engenharia do software.

## Perguntas de revisão

### 1. O que é SAST?

SAST é uma técnica de teste estático de segurança que analisa código-fonte, bytecode ou artefatos da aplicação sem executar o sistema, procurando padrões que podem indicar vulnerabilidades.

### 2. O que é source?

Source é o ponto de origem de um dado, geralmente uma entrada externa ou não confiável, como parâmetro HTTP, header, body JSON, cookie, arquivo ou dado vindo de integração.

### 3. O que é sink?

Sink é uma operação sensível que pode causar impacto de segurança se receber dado não confiável sem controle adequado, como query SQL, escrita em arquivo, log, chamada HTTP ou renderização HTML.

### 4. O que é taint?

Taint é a marcação de um dado como potencialmente não confiável. A ferramenta acompanha esse dado pelo código para verificar se ele chega a um sink perigoso.

### 5. O que é taint flow?

Taint flow é o caminho percorrido pelo dado entre o source e o sink.

### 6. O que é falso positivo?

Falso positivo é um finding reportado como vulnerabilidade, mas que, após análise técnica, não representa uma vulnerabilidade real no contexto analisado.

### 7. O que é true positive?

True positive é um finding que representa uma vulnerabilidade real ou uma fragilidade que precisa ser corrigida.

### 8. O que significa Not Exploitable?

Significa que o scanner encontrou um fluxo suspeito, mas a exploração não é possível no contexto real devido a controles, restrições ou características demonstráveis do sistema.

### 9. Qual é a diferença entre validator e sanitizer?

Validator verifica se o dado segue uma regra esperada. Sanitizer torna o dado seguro para um contexto específico. Um validador nem sempre é suficiente para proteger um sink.

### 10. Por que não devemos criar código artificial para calar o scanner?

Porque isso pode esconder o fluxo sem corrigir o risco real, além de piorar legibilidade, manutenção e qualidade do código.

### 11. Por que autenticação não elimina vulnerabilidade?

Porque usuários autenticados também podem explorar falhas. Autenticação reduz superfície, mas não substitui validação, autorização e controles adequados.

### 12. O que deve conter uma boa justificativa de falso positivo?

Deve conter source, sink, fluxo apontado, fluxo real, controles existentes, evidências, testes e risco residual.

### 13. Quando pedir ajuste da query corporativa do SAST?

Quando um controle customizado real não é reconhecido pela ferramenta e o mesmo falso positivo continua aparecendo, desde que AppSec valide a evidência e o ajuste não enfraqueça outros projetos.

### 14. Por que o controle deve ficar próximo ao sink?

Porque isso facilita análise humana, reduz ambiguidade para ferramentas e garante que o dado seja protegido imediatamente antes da operação sensível.

### 15. Qual é a principal lição sobre SAST?

SAST não deve ser tratado como oráculo nem ignorado. Ele deve ser usado como ferramenta de engenharia, com análise técnica, correções reais, evidências e triagem responsável.

## Referências

- OWASP — Static Code Analysis.
- OWASP — Secure Code Review Cheat Sheet.
- OWASP — Code Review Guide.
- OWASP — DevSecOps Guideline: Static Application Security Testing.
- OWASP — Web Security Testing Guide.
- CWE — Common Weakness Enumeration.
- Oracle — Java SE 25 Documentation.
- Spring Boot Reference Documentation.
- Spring Security Reference Documentation.