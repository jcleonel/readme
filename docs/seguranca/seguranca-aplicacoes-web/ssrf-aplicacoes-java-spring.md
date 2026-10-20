# SSRF em aplicações Java/Spring

SSRF significa Server-Side Request Forgery, ou falsificação de requisição do lado do servidor.

Essa vulnerabilidade acontece quando uma aplicação backend faz uma requisição para um recurso externo ou interno usando um destino que pode ser controlado, direta ou indiretamente, por entrada não confiável.

Em aplicações Java/Spring, esse risco costuma aparecer em pontos como:

- chamadas HTTP com `RestClient`;
- chamadas HTTP com `RestTemplate`;
- chamadas HTTP com `WebClient`;
- uso direto de `java.net.http.HttpClient`;
- download de arquivos informando URL;
- upload para serviço externo;
- geração de URL dinâmica;
- integração com APIs parceiras;
- consumo de webhooks;
- redirecionamentos automáticos;
- leitura de arquivos remotos;
- proxies internos;
- gateways corporativos.

O problema central é simples:

> A aplicação não deve permitir que o usuário decida livremente para onde o servidor vai fazer uma requisição.

No navegador, quando um usuário acessa uma URL, a requisição sai da máquina do usuário. Em SSRF, quem faz a requisição é o servidor da aplicação. Isso muda completamente o risco.

O servidor pode estar dentro de uma rede corporativa, com acesso a recursos internos, credenciais de ambiente, serviços privados, APIs restritas ou endpoints que não deveriam ser acessíveis diretamente pelo usuário.

Por isso, SSRF não é apenas “validar uma URL”. É controlar a fronteira outbound da aplicação.

## O problema real que SSRF resolve no estudo de segurança

Estudar SSRF ajuda a entender um ponto importante de arquitetura:

> Toda chamada outbound feita pelo backend precisa ter destino controlado.

Muitos desenvolvedores se preocupam com entrada de dados em banco, tela ou arquivo, mas esquecem que uma chamada HTTP feita pelo servidor também é um sink sensível.

Quando a aplicação recebe uma URL, concatena valores em um endpoint ou monta dinamicamente o destino de uma chamada, ela pode abrir uma porta para que a entrada externa influencie a rede interna.

Na aplicação fictícia `meucatalogomusical.com`, imagine uma funcionalidade que busca informações de shows, capas de álbuns ou dados de bandas em APIs parceiras.

Um fluxo seguro seria:

```text
Usuário escolhe um parceiro permitido
    -> aplicação resolve o endpoint internamente
    -> aplicação valida o destino
    -> aplicação faz a chamada outbound
```

Um fluxo perigoso seria:

```text
Usuário envia uma URL
    -> aplicação usa essa URL diretamente
    -> servidor faz a chamada para o destino informado
```

A diferença é enorme.

No primeiro caso, o usuário escolhe uma opção de negócio.

No segundo, ele controla o destino técnico da requisição server-side.

## Antes de entrar nos detalhes: uma ideia simples

Pense em `meucatalogomusical.com`.

A aplicação possui uma funcionalidade para importar informações públicas sobre bandas de Hard Rock. Um usuário administrativo quer enriquecer o catálogo com dados da banda **Whitesnake**.

Uma abordagem ingênua seria permitir que o usuário informe a URL completa da API parceira:

```text
POST /api/importacoes/banda?url=https://api-parceira.example/bandas/whitesnake
```

O backend recebe o parâmetro `url` e faz a chamada.

Parece flexível.

Mas flexibilidade excessiva em destino outbound pode ser perigosa.

Uma abordagem melhor é permitir que o usuário escolha um parceiro já cadastrado:

```text
POST /api/importacoes/banda?parceiro=CATALOGO_OFICIAL&banda=Whitesnake
```

A aplicação então monta internamente a chamada para um destino conhecido, validado e permitido.

Essa mudança reduz a superfície de ataque porque o usuário não controla a URL final.

## Definição técnica

SSRF é uma vulnerabilidade em que um atacante consegue induzir o servidor da aplicação a fazer requisições para destinos não pretendidos.

Em termos de SAST, o padrão costuma aparecer assim:

```text
source: entrada controlada pelo usuário
    -> controller
    -> service
    -> montagem de URL/URI/request
sink: cliente HTTP fazendo requisição outbound
```

Exemplos de sources:

- `@RequestParam String url`;
- `@RequestParam String fileId`;
- `@RequestParam String nomeArquivo`;
- `@RequestBody` com campo de endpoint;
- headers HTTP;
- dados vindos de arquivo;
- dados vindos de banco;
- dados vindos de storage;
- dados vindos de configuração editável;
- parâmetros usados para montar path, query string ou host.

Exemplos de sinks:

- `RestClient.get().uri(...).retrieve()`;
- `RestTemplate.getForObject(...)`;
- `WebClient.get().uri(...).retrieve()`;
- `HttpClient.send(...)`;
- `HttpURLConnection`;
- SDKs que fazem transferência de arquivo;
- clientes de storage;
- métodos como `transferTo`, quando encapsulam chamada remota;
- bibliotecas internas que montam URL de upload/download.

O risco não está em usar cliente HTTP. O risco está em deixar entrada não confiável controlar o destino da requisição.

## Por que SSRF é perigoso?

SSRF é perigoso porque a requisição parte do servidor, não do navegador do usuário.

O servidor pode ter acessos que o usuário não tem, como:

- serviços internos;
- APIs privadas;
- endpoints administrativos;
- redes internas;
- metadados de ambiente;
- credenciais temporárias;
- serviços de infraestrutura;
- storage interno;
- painéis restritos;
- bancos expostos apenas na rede interna;
- sistemas legados acessíveis somente pelo backend.

Mesmo quando a aplicação é interna, SSRF continua relevante. Um usuário autenticado, um sistema consumidor ou uma integração comprometida pode tentar influenciar destinos outbound.

Autenticação reduz superfície, mas não substitui validação de destino.

## SSRF em aplicações Java/Spring

Em Java/Spring, SSRF costuma aparecer quando o código aceita entrada externa para montar uma URL.

Exemplo inseguro:

```java
package com.meucatalogomusical.importacao.api;

import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/importacoes")
public class ImportacaoBandaController {

    private final ImportacaoBandaService importacaoBandaService;

    public ImportacaoBandaController(ImportacaoBandaService importacaoBandaService) {
        this.importacaoBandaService = importacaoBandaService;
    }

    @PostMapping("/banda")
    public String importar(@RequestParam String url) {
        return importacaoBandaService.importar(url);
    }
}
```

Service inseguro:

```java
package com.meucatalogomusical.importacao.application;

import org.springframework.stereotype.Service;
import org.springframework.web.client.RestClient;

@Service
public class ImportacaoBandaService {

    private final RestClient restClient;

    public ImportacaoBandaService(RestClient.Builder restClientBuilder) {
        this.restClient = restClientBuilder.build();
    }

    public String importar(String url) {
        return restClient
                .get()
                .uri(url)
                .retrieve()
                .body(String.class);
    }
}
```

O problema está aqui:

```java
.uri(url)
```

A entrada `url` veio do usuário e foi usada como destino da chamada outbound.

O SAST poderia apontar algo como:

```text
Query: SSRF
Source: ImportacaoBandaController.importar(url)
Sink: RestClient.uri(url).retrieve()
```

Esse finding merece atenção porque o usuário controla o destino acessado pelo servidor.

## HTTPS é necessário, mas não suficiente

Uma primeira reação comum é validar se a URL começa com `https://`.

Isso é melhor do que aceitar qualquer coisa, mas não resolve SSRF por completo.

Exemplo:

```java
if (!url.startsWith("https://")) {
    throw new IllegalArgumentException("A URL deve usar HTTPS.");
}
```

Essa validação é fraca por vários motivos:

- `startsWith` não entende URI de forma estrutural;
- não valida host;
- não valida porta;
- não valida destino permitido;
- não trata normalização;
- não lida bem com variações de formato;
- não impede que um host HTTPS não autorizado seja acessado;
- não resolve redirecionamentos;
- não diferencia URL de entrada e URL final após redirect.

HTTPS protege o canal, mas não responde à pergunta:

> Este destino é permitido para esta aplicação?

SSRF exige controle de destino, não apenas criptografia.

## A melhor defesa: não aceitar URL livre

A defesa mais forte é evitar receber URL livre do usuário.

Em vez disso, aceite uma opção de negócio e resolva o endpoint internamente.

Exemplo:

```java
package com.meucatalogomusical.importacao.domain;

public enum ParceiroMusical {
    CATALOGO_OFICIAL,
    AGENDA_SHOWS,
    MIDIA_BANDAS
}
```

Request:

```java
package com.meucatalogomusical.importacao.api;

import com.meucatalogomusical.importacao.domain.ParceiroMusical;
import jakarta.validation.constraints.NotBlank;
import jakarta.validation.constraints.NotNull;
import jakarta.validation.constraints.Size;

public record ImportacaoBandaRequest(

        @NotNull
        ParceiroMusical parceiro,

        @NotBlank
        @Size(max = 120)
        String nomeBanda
) {
}
```

Controller:

```java
package com.meucatalogomusical.importacao.api;

import jakarta.validation.Valid;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/importacoes")
public class ImportacaoBandaController {

    private final ImportacaoBandaService importacaoBandaService;

    public ImportacaoBandaController(ImportacaoBandaService importacaoBandaService) {
        this.importacaoBandaService = importacaoBandaService;
    }

    @PostMapping("/banda")
    public ImportacaoBandaResponse importar(@Valid @RequestBody ImportacaoBandaRequest request) {
        return importacaoBandaService.importar(request);
    }
}
```

Agora o usuário não envia `https://...`.

Ele informa:

- parceiro;
- nome da banda.

A aplicação decide o destino real.

Isso muda o desenho de segurança.

## Resolver endpoint por allowlist

Agora podemos resolver endpoints por lista permitida.

```java
package com.meucatalogomusical.importacao.infra;

import com.meucatalogomusical.importacao.domain.ParceiroMusical;
import org.springframework.stereotype.Component;

import java.net.URI;
import java.util.Map;

@Component
public class ParceiroMusicalEndpointResolver {

    private final Map<ParceiroMusical, URI> endpoints = Map.of(
            ParceiroMusical.CATALOGO_OFICIAL, URI.create("https://catalogo-parceiro.example/api/bandas"),
            ParceiroMusical.AGENDA_SHOWS, URI.create("https://agenda-parceira.example/api/shows"),
            ParceiroMusical.MIDIA_BANDAS, URI.create("https://midia-parceira.example/api/artistas")
    );

    private final OutboundUriValidator outboundUriValidator;

    public ParceiroMusicalEndpointResolver(OutboundUriValidator outboundUriValidator) {
        this.outboundUriValidator = outboundUriValidator;
    }

    public URI resolver(ParceiroMusical parceiro) {
        URI endpoint = endpoints.get(parceiro);

        if (endpoint == null) {
            throw new IllegalArgumentException("Parceiro musical não autorizado.");
        }

        return outboundUriValidator.requireAllowedHttpsEndpoint(endpoint);
    }
}
```

Aqui a origem do destino não é o usuário.

O destino vem de uma lista controlada pela aplicação.

O usuário escolhe uma opção permitida, não uma URL arbitrária.

## Validador de URI outbound

Um validador defensivo pode deixar explícitas as regras mínimas para chamadas outbound.

```java
package com.meucatalogomusical.importacao.infra;

import org.springframework.stereotype.Component;

import java.net.URI;
import java.util.Set;

@Component
public class OutboundUriValidator {

    private static final Set<String> HOSTS_PERMITIDOS = Set.of(
            "catalogo-parceiro.example",
            "agenda-parceira.example",
            "midia-parceira.example"
    );

    public URI requireAllowedHttpsEndpoint(URI uri) {
        if (uri == null) {
            throw new IllegalArgumentException("Endpoint outbound é obrigatório.");
        }

        if (!uri.isAbsolute()) {
            throw new IllegalArgumentException("Endpoint outbound deve ser absoluto.");
        }

        if (!"https".equalsIgnoreCase(uri.getScheme())) {
            throw new IllegalArgumentException("Endpoint outbound deve usar HTTPS.");
        }

        if (uri.getHost() == null || uri.getHost().isBlank()) {
            throw new IllegalArgumentException("Endpoint outbound deve possuir host.");
        }

        if (!HOSTS_PERMITIDOS.contains(uri.getHost().toLowerCase())) {
            throw new IllegalArgumentException("Host outbound não autorizado.");
        }

        int porta = uri.getPort();

        if (porta != -1 && porta != 443) {
            throw new IllegalArgumentException("Porta outbound não autorizada.");
        }

        return uri.normalize();
    }
}
```

Esse código valida:

- URI obrigatória;
- URI absoluta;
- esquema HTTPS;
- host presente;
- host dentro da allowlist;
- porta permitida;
- normalização da URI.

Em um ambiente corporativo real, essa lista poderia vir de configuração, mas ainda assim deveria ser tratada como configuração controlada, revisada e testada.

## Serviço seguro com RestClient

Agora o serviço pode usar o endpoint resolvido internamente.

```java
package com.meucatalogomusical.importacao.api;

public record ImportacaoBandaResponse(
        String parceiro,
        String nomeBanda,
        String status
) {
}
```

```java
package com.meucatalogomusical.importacao.application;

import com.meucatalogomusical.importacao.api.ImportacaoBandaRequest;
import com.meucatalogomusical.importacao.api.ImportacaoBandaResponse;
import com.meucatalogomusical.importacao.infra.ParceiroMusicalEndpointResolver;
import org.springframework.stereotype.Service;
import org.springframework.web.client.RestClient;

import java.net.URI;

@Service
public class ImportacaoBandaService {

    private final RestClient restClient;
    private final ParceiroMusicalEndpointResolver endpointResolver;

    public ImportacaoBandaService(
            RestClient.Builder restClientBuilder,
            ParceiroMusicalEndpointResolver endpointResolver
    ) {
        this.restClient = restClientBuilder.build();
        this.endpointResolver = endpointResolver;
    }

    public ImportacaoBandaResponse importar(ImportacaoBandaRequest request) {
        URI endpoint = endpointResolver.resolver(request.parceiro());

        String respostaParceiro = restClient
                .get()
                .uri(uriBuilder -> uriBuilder
                        .scheme(endpoint.getScheme())
                        .host(endpoint.getHost())
                        .path(endpoint.getPath())
                        .queryParam("banda", request.nomeBanda())
                        .build())
                .retrieve()
                .body(String.class);

        return new ImportacaoBandaResponse(
                request.parceiro().name(),
                request.nomeBanda(),
                respostaParceiro == null ? "SEM_RESPOSTA" : "IMPORTADO"
        );
    }
}
```

Nesse exemplo, o usuário controla apenas o valor de negócio `nomeBanda`, que é enviado como parâmetro.

Ele não controla:

- esquema;
- host;
- porta;
- path base;
- destino da chamada;
- protocolo;
- redirecionamento.

Esse desenho é muito mais seguro.

## Cuidado com uriBuilder

O `uriBuilder` ajuda a montar query parameters corretamente, mas ele não substitui validação de destino.

Se você fizer isto:

```java
.uri(urlRecebidaDoUsuario)
```

continua perigoso.

Se fizer isto:

```java
.uri(uriBuilder -> uriBuilder
        .scheme(esquemaRecebidoDoUsuario)
        .host(hostRecebidoDoUsuario)
        .path(pathRecebidoDoUsuario)
        .build())
```

também continua perigoso.

O ponto é: builder de URI não é sanitizer de SSRF.

Ele ajuda a construir a URI, mas não decide se o destino é permitido.

## Redirecionamentos

Redirecionamentos são uma preocupação importante em SSRF.

Mesmo que a primeira URL seja permitida, um servidor pode responder com redirect para outro destino.

Fluxo perigoso:

```text
Aplicação chama destino permitido
    -> servidor responde redirect
    -> cliente HTTP segue redirect automaticamente
    -> requisição final vai para destino não pretendido
```

Por isso, em chamadas sensíveis, é comum desabilitar redirects automáticos ou validar novamente o destino após redirect.

Em Spring Boot, a configuração de clientes HTTP pode ser centralizada. Um exemplo conceitual de configuração para não seguir redirects automaticamente:

```yaml
spring:
  http:
    clients:
      redirects: dont-follow
```

A configuração exata pode variar conforme versão do Spring Boot, cliente HTTP subjacente e starter utilizado. O princípio defensivo é: não permita que redirects transformem um destino validado em outro destino não validado.

## Usando HttpClient com redirects desabilitados

Se você usar diretamente o `java.net.http.HttpClient`, também pode deixar a política explícita.

```java
package com.meucatalogomusical.importacao.infra;

import org.springframework.stereotype.Component;

import java.net.http.HttpClient;
import java.time.Duration;

@Component
public class SecureHttpClientFactory {

    public HttpClient create() {
        return HttpClient.newBuilder()
                .connectTimeout(Duration.ofSeconds(3))
                .followRedirects(HttpClient.Redirect.NEVER)
                .build();
    }
}
```

Esse exemplo define:

- timeout de conexão;
- redirects desabilitados.

Timeout não é correção de SSRF, mas ajuda a reduzir impacto operacional em chamadas externas.

Redirect desabilitado ajuda a evitar bypass de validação por redirecionamento.

## Separar URI e body

Um aprendizado importante em aplicações reais é separar claramente:

- destino da requisição;
- corpo da requisição;
- headers;
- credenciais;
- parâmetros de negócio.

Código perigoso mistura tudo:

```java
String url = host + "?payload=" + payloadRecebido;
restClient.post().uri(url).retrieve();
```

Código melhor mantém o destino controlado e envia dados como body ou query parameter controlado:

```java
URI endpoint = endpointResolver.resolver(parceiro);

restClient
        .post()
        .uri(endpoint)
        .contentType(MediaType.APPLICATION_JSON)
        .body(requestParceiro)
        .retrieve()
        .body(String.class);
```

Separar URI e body ajuda em três aspectos:

1. reduz risco de entrada influenciar o destino;
2. facilita análise humana;
3. facilita interpretação do SAST.

A regra prática é:

> Dados de negócio podem ir no body. Destino da chamada deve vir de configuração controlada ou allowlist.

## SSRF e headers

Headers também podem influenciar chamadas outbound.

Exemplo perigoso:

```java
String partnerHost = request.getHeader("X-Partner-Host");
URI endpoint = URI.create("https://" + partnerHost + "/api/bandas");
```

Mesmo usando HTTPS, o usuário controla o host.

Uma abordagem melhor é usar headers apenas para dados de negócio ou correlação, não para escolher destino técnico.

Se o parceiro precisa ser selecionado dinamicamente, use identificadores controlados:

```java
ParceiroMusical parceiro = ParceiroMusical.valueOf(request.getHeader("X-Partner-Code"));
URI endpoint = endpointResolver.resolver(parceiro);
```

Ainda assim, é melhor validar e tratar erro adequadamente, porque `valueOf` lança exceção para valores inválidos.

## SSRF e configuração

Nem todo SSRF vem diretamente de `@RequestParam`.

Às vezes, o destino vem de configuração:

```yaml
integracoes:
  agenda-shows-url: https://agenda-parceira.example/api/shows
```

Isso geralmente é melhor do que receber URL do usuário, mas ainda exige validação.

Por quê?

Porque configuração também pode ser alterada por erro humano, variável de ambiente incorreta, pipeline mal configurado ou segredo/configuração comprometida.

Por isso, é útil validar configuração na inicialização ou imediatamente antes do sink.

Exemplo:

```java
@PostConstruct
void validarConfiguracao() {
    outboundUriValidator.requireAllowedHttpsEndpoint(agendaShowsUri);
}
```

Ou validar imediatamente antes da chamada, que é ainda mais próximo do sink.

## Validação próxima ao sink

Em segurança, é útil validar perto do ponto sensível.

Para SSRF, o ponto sensível é a chamada HTTP.

Exemplo:

```java
URI endpoint = endpointResolver.resolver(request.parceiro());

URI endpointValidado = outboundUriValidator.requireAllowedHttpsEndpoint(endpoint);

return restClient
        .get()
        .uri(endpointValidado)
        .retrieve()
        .body(String.class);
```

Mesmo que o endpoint já tenha sido validado antes, repetir a validação imediatamente antes da chamada pode ser aceitável quando o risco é alto.

Essa redundância não é necessariamente duplicação ruim. Pode ser uma fronteira de segurança explícita.

## Falha fechada

Em SSRF, a aplicação deve falhar fechada.

Ou seja:

- se a URI for nula, rejeite;
- se for relativa, rejeite;
- se não tiver host, rejeite;
- se não for HTTPS, rejeite;
- se o host não estiver autorizado, rejeite;
- se a porta não for permitida, rejeite;
- se houver redirect não validado, rejeite;
- se a configuração estiver ausente, rejeite.

Não faça fallback inseguro.

Exemplo ruim:

```java
if (endpoint == null) {
    endpoint = URI.create(request.urlAlternativa());
}
```

Exemplo melhor:

```java
if (endpoint == null) {
    throw new IllegalStateException("Endpoint parceiro não configurado.");
}
```

Quando a aplicação não sabe para onde deve chamar, ela não deve chamar lugar nenhum.

## SSRF e allowlist

Allowlist é uma lista positiva do que é permitido.

Para SSRF, allowlist costuma ser melhor que denylist.

Denylist tenta bloquear destinos ruins.

Allowlist permite apenas destinos bons.

Exemplo de denylist fraca:

```java
if (host.contains("localhost")) {
    throw new IllegalArgumentException("Host proibido.");
}
```

Isso é fraco porque existem muitas formas de representar destinos, hosts e redes.

Exemplo de allowlist melhor:

```java
private static final Set<String> HOSTS_PERMITIDOS = Set.of(
        "catalogo-parceiro.example",
        "agenda-parceira.example",
        "midia-parceira.example"
);
```

Com allowlist, a pergunta muda de:

> Esse destino parece perigoso?

para:

> Esse destino está explicitamente permitido?

Essa segunda pergunta é muito mais segura.

## SSRF e DNS

Em ambientes mais sensíveis, validar apenas o nome do host pode não ser suficiente.

Por quê?

Porque nomes de domínio são resolvidos por DNS, e o IP resultante pode mudar.

Alguns cenários exigem verificar:

- se o domínio é permitido;
- quais IPs ele resolve;
- se os IPs pertencem a faixas permitidas;
- se não apontam para redes internas;
- se não há mudança entre validação e uso;
- se o cliente HTTP não resolve diferente do validador;
- se a infraestrutura usa proxy controlado.

Esse tema é mais avançado e depende da arquitetura da empresa.

Para muitos sistemas corporativos, a melhor estratégia é não permitir URL arbitrária, usar allowlist de parceiros e centralizar egress em infraestrutura controlada.

## SSRF e infraestrutura

Código ajuda, mas SSRF também deve ser tratado na infraestrutura.

Controles possíveis:

- egress control;
- firewall de saída;
- proxy corporativo;
- service mesh;
- network policies;
- bloqueio de redes internas sensíveis;
- credenciais com menor privilégio;
- segmentação de rede;
- mTLS entre serviços autorizados;
- monitoramento de chamadas outbound;
- alertas para destinos inesperados.

Mesmo com código seguro, infraestrutura deve limitar danos.

E mesmo com infraestrutura forte, código deve validar destino.

É defesa em profundidade.

## Exemplo completo: importação segura de dados de banda

Vamos consolidar um fluxo seguro.

### Enum de parceiros

```java
package com.meucatalogomusical.importacao.domain;

public enum ParceiroMusical {
    CATALOGO_OFICIAL,
    AGENDA_SHOWS,
    MIDIA_BANDAS
}
```

### Request

```java
package com.meucatalogomusical.importacao.api;

import com.meucatalogomusical.importacao.domain.ParceiroMusical;
import jakarta.validation.constraints.NotBlank;
import jakarta.validation.constraints.NotNull;
import jakarta.validation.constraints.Size;

public record ImportacaoBandaRequest(

        @NotNull
        ParceiroMusical parceiro,

        @NotBlank
        @Size(max = 120)
        String nomeBanda
) {
}
```

### Controller

```java
package com.meucatalogomusical.importacao.api;

import jakarta.validation.Valid;
import org.springframework.http.HttpStatus;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/importacoes/bandas")
public class ImportacaoBandaController {

    private final ImportacaoBandaService importacaoBandaService;

    public ImportacaoBandaController(ImportacaoBandaService importacaoBandaService) {
        this.importacaoBandaService = importacaoBandaService;
    }

    @PostMapping
    @ResponseStatus(HttpStatus.ACCEPTED)
    public ImportacaoBandaResponse importar(@Valid @RequestBody ImportacaoBandaRequest request) {
        return importacaoBandaService.importar(request);
    }
}
```

### Validador

```java
package com.meucatalogomusical.importacao.infra;

import org.springframework.stereotype.Component;

import java.net.URI;
import java.util.Set;

@Component
public class OutboundUriValidator {

    private static final Set<String> HOSTS_PERMITIDOS = Set.of(
            "catalogo-parceiro.example",
            "agenda-parceira.example",
            "midia-parceira.example"
    );

    public URI requireAllowedHttpsEndpoint(URI endpoint) {
        if (endpoint == null) {
            throw new IllegalArgumentException("Endpoint outbound é obrigatório.");
        }

        if (!endpoint.isAbsolute()) {
            throw new IllegalArgumentException("Endpoint outbound deve ser absoluto.");
        }

        if (!"https".equalsIgnoreCase(endpoint.getScheme())) {
            throw new IllegalArgumentException("Endpoint outbound deve usar HTTPS.");
        }

        String host = endpoint.getHost();

        if (host == null || host.isBlank()) {
            throw new IllegalArgumentException("Endpoint outbound deve possuir host.");
        }

        if (!HOSTS_PERMITIDOS.contains(host.toLowerCase())) {
            throw new IllegalArgumentException("Host outbound não autorizado.");
        }

        int port = endpoint.getPort();

        if (port != -1 && port != 443) {
            throw new IllegalArgumentException("Porta outbound não autorizada.");
        }

        return endpoint.normalize();
    }
}
```

### Resolver

```java
package com.meucatalogomusical.importacao.infra;

import com.meucatalogomusical.importacao.domain.ParceiroMusical;
import org.springframework.stereotype.Component;

import java.net.URI;
import java.util.Map;

@Component
public class ParceiroMusicalEndpointResolver {

    private final Map<ParceiroMusical, URI> endpoints = Map.of(
            ParceiroMusical.CATALOGO_OFICIAL, URI.create("https://catalogo-parceiro.example/api/bandas"),
            ParceiroMusical.AGENDA_SHOWS, URI.create("https://agenda-parceira.example/api/shows"),
            ParceiroMusical.MIDIA_BANDAS, URI.create("https://midia-parceira.example/api/artistas")
    );

    private final OutboundUriValidator outboundUriValidator;

    public ParceiroMusicalEndpointResolver(OutboundUriValidator outboundUriValidator) {
        this.outboundUriValidator = outboundUriValidator;
    }

    public URI resolver(ParceiroMusical parceiro) {
        URI endpoint = endpoints.get(parceiro);

        if (endpoint == null) {
            throw new IllegalArgumentException("Parceiro não autorizado.");
        }

        return outboundUriValidator.requireAllowedHttpsEndpoint(endpoint);
    }
}
```

### Service

```java
package com.meucatalogomusical.importacao.api;

public record ImportacaoBandaResponse(
        String banda,
        String parceiro,
        String status
) {
}
```

```java
package com.meucatalogomusical.importacao.api;

import com.meucatalogomusical.importacao.infra.ParceiroMusicalEndpointResolver;
import org.springframework.http.MediaType;
import org.springframework.stereotype.Service;
import org.springframework.web.client.RestClient;

import java.net.URI;

@Service
public class ImportacaoBandaService {

    private final RestClient restClient;
    private final ParceiroMusicalEndpointResolver endpointResolver;

    public ImportacaoBandaService(
            RestClient.Builder restClientBuilder,
            ParceiroMusicalEndpointResolver endpointResolver
    ) {
        this.restClient = restClientBuilder.build();
        this.endpointResolver = endpointResolver;
    }

    public ImportacaoBandaResponse importar(ImportacaoBandaRequest request) {
        URI endpoint = endpointResolver.resolver(request.parceiro());

        String resposta = restClient
                .get()
                .uri(uriBuilder -> uriBuilder
                        .scheme(endpoint.getScheme())
                        .host(endpoint.getHost())
                        .path(endpoint.getPath())
                        .queryParam("nome", request.nomeBanda())
                        .build())
                .accept(MediaType.APPLICATION_JSON)
                .retrieve()
                .body(String.class);

        String status = resposta == null || resposta.isBlank()
                ? "SEM_DADOS"
                : "IMPORTADO";

        return new ImportacaoBandaResponse(
                request.nomeBanda(),
                request.parceiro().name(),
                status
        );
    }
}
```

Exemplo de uso:

```json
{
  "parceiro": "CATALOGO_OFICIAL",
  "nomeBanda": "Black Label Society"
}
```

A banda entra como dado de negócio.

O parceiro entra como enum.

O endpoint real é resolvido pela aplicação.

Esse é o tipo de desenho que reduz risco de SSRF.

## Testes do validador

Testes ajudam a transformar a política de segurança em comportamento verificável.

```java
package com.meucatalogomusical.importacao.infra;

import org.junit.jupiter.api.Test;

import java.net.URI;

import static org.junit.jupiter.api.Assertions.*;

class OutboundUriValidatorTest {

    private final OutboundUriValidator validator = new OutboundUriValidator();

    @Test
    void deveAceitarEndpointHttpsComHostPermitido() {
        URI endpoint = URI.create("https://catalogo-parceiro.example/api/bandas");

        URI validado = validator.requireAllowedHttpsEndpoint(endpoint);

        assertEquals(endpoint, validado);
    }

    @Test
    void deveRejeitarHttp() {
        URI endpoint = URI.create("http://catalogo-parceiro.example/api/bandas");

        assertThrows(IllegalArgumentException.class,
                () -> validator.requireAllowedHttpsEndpoint(endpoint));
    }

    @Test
    void deveRejeitarUriRelativa() {
        URI endpoint = URI.create("/api/bandas");

        assertThrows(IllegalArgumentException.class,
                () -> validator.requireAllowedHttpsEndpoint(endpoint));
    }

    @Test
    void deveRejeitarHostNaoPermitido() {
        URI endpoint = URI.create("https://nao-autorizado.example/api/bandas");

        assertThrows(IllegalArgumentException.class,
                () -> validator.requireAllowedHttpsEndpoint(endpoint));
    }

    @Test
    void deveRejeitarPortaNaoAutorizada() {
        URI endpoint = URI.create("https://catalogo-parceiro.example:8443/api/bandas");

        assertThrows(IllegalArgumentException.class,
                () -> validator.requireAllowedHttpsEndpoint(endpoint));
    }
}
```

Esses testes validam o comportamento esperado:

- HTTPS permitido;
- HTTP rejeitado;
- URI relativa rejeitada;
- host desconhecido rejeitado;
- porta não autorizada rejeitada.

Esse tipo de teste é essencial para sustentar a correção em scans futuros.

## Teste do resolver

```java
package com.meucatalogomusical.importacao.infra;

import com.meucatalogomusical.importacao.domain.ParceiroMusical;
import org.junit.jupiter.api.Test;

import java.net.URI;

import static org.junit.jupiter.api.Assertions.assertEquals;

class ParceiroMusicalEndpointResolverTest {

    private final ParceiroMusicalEndpointResolver resolver =
            new ParceiroMusicalEndpointResolver(new OutboundUriValidator());

    @Test
    void deveResolverCatalogoOficialParaEndpointPermitido() {
        URI endpoint = resolver.resolver(ParceiroMusical.CATALOGO_OFICIAL);

        assertEquals("https", endpoint.getScheme());
        assertEquals("catalogo-parceiro.example", endpoint.getHost());
        assertEquals("/api/bandas", endpoint.getPath());
    }
}
```

Esse teste garante que a opção de negócio se transforma em destino técnico permitido.

## Como um SAST pode enxergar esse código?

Na versão insegura, o fluxo era:

```text
@RequestParam url
    -> service.importar(url)
    -> restClient.uri(url)
```

Na versão segura, o fluxo passa a ser:

```text
@RequestBody parceiro
    -> enum ParceiroMusical
    -> resolver interno
    -> endpoint de allowlist
    -> validação HTTPS/host/porta
    -> restClient.uri(endpoint validado)
```

A diferença para o scanner e para o revisor humano é grande.

O usuário não controla mais a URL.

Ele controla uma opção de negócio limitada.

O destino é resolvido e validado dentro da aplicação.

## Quando SSRF aparece como falso positivo?

SSRF pode aparecer como falso positivo quando a ferramenta acredita que o usuário controla o destino, mas o código real não permite isso.

Exemplos:

### 1. O usuário controla o body, não a URI

O scanner pode ver que um valor vindo do request chega perto de uma chamada HTTP.

Mas, na prática, esse valor pode estar apenas no corpo da requisição.

Exemplo seguro:

```java
URI endpoint = endpointResolver.resolver(ParceiroMusical.CATALOGO_OFICIAL);

restClient
        .post()
        .uri(endpoint)
        .body(requestParceiro)
        .retrieve()
        .body(String.class);
```

Se `requestParceiro` vem do usuário, ele influencia o body, não o destino.

Isso não é SSRF por si só.

Pode haver outros riscos, mas não controle do destino outbound.

### 2. Endpoint vem de configuração validada

Se o endpoint vem de configuração controlada e é validado como HTTPS, absoluto e host permitido antes do sink, o risco de SSRF pode estar mitigado.

Ainda assim, é preciso documentar:

- quem controla a configuração;
- quando ela é validada;
- quais hosts são permitidos;
- se redirects são seguidos;
- quais testes existem;
- qual risco residual permanece.

### 3. O scanner não reconhece o validador customizado

Uma ferramenta SAST pode não entender que este método é uma barreira de segurança:

```java
outboundUriValidator.requireAllowedHttpsEndpoint(endpoint);
```

Nesse caso, não é correto criar um wrapper artificial apenas para o scanner.

O melhor caminho é:

- manter o controle real;
- criar testes;
- documentar source, sink e fluxo;
- pedir avaliação do time AppSec;
- eventualmente configurar a query corporativa para reconhecer a guarda, se fizer sentido.

## Quando SSRF é verdadeiro positivo?

SSRF tende a ser verdadeiro positivo quando:

- o usuário envia URL completa;
- o usuário controla host;
- o usuário controla esquema;
- o usuário controla porta;
- o usuário controla path usado como URL final;
- o usuário controla fileId usado para montar URL de upload/download sem validação;
- a aplicação segue redirects automaticamente;
- não há allowlist;
- não há validação perto do sink;
- há concatenação de URL com dados externos;
- o sink faz chamada HTTP, transferência remota ou acesso a recurso externo.

Exemplo de forte indício:

```java
String uploadUrl = baseUrl + "/" + nomeArquivo;
restClient.post().uri(uploadUrl).retrieve();
```

Se `nomeArquivo` influencia o destino e não há validação, o finding merece correção.

## Diferença entre SSRF e Open Redirect

SSRF e Open Redirect podem parecer parecidos porque ambos envolvem URL.

Mas são diferentes.

Open Redirect acontece quando a aplicação redireciona o navegador do usuário para uma URL controlada.

SSRF acontece quando o servidor faz a requisição.

Comparação:

| Vulnerabilidade | Quem faz a requisição? | Risco principal |
|---|---|---|
| Open Redirect | Navegador do usuário | Phishing, redirecionamento para site malicioso |
| SSRF | Servidor da aplicação | Acesso indevido a recursos internos ou não pretendidos |

Em SSRF, o risco é maior porque o servidor pode acessar destinos que o usuário não conseguiria acessar diretamente.

## Diferença entre SSRF e chamada outbound normal

Nem toda chamada outbound é SSRF.

Uma chamada normal:

```text
Aplicação chama endpoint fixo e autorizado de parceiro
```

Uma chamada com risco SSRF:

```text
Aplicação chama endpoint definido por entrada externa
```

A diferença está no controle do destino.

Chamadas outbound são normais em sistemas corporativos.

O problema é deixar o destino ser controlado por quem não deveria controlá-lo.

## Boas práticas

### 1. Não aceite URL livre

Prefira identificadores de negócio, enums ou chaves de parceiro.

### 2. Use allowlist

Permita apenas destinos conhecidos.

### 3. Valide URI estruturalmente

Use `URI`, não apenas `startsWith`.

Valide:

- esquema;
- host;
- porta;
- URI absoluta;
- normalização;
- destino permitido.

### 4. Valide próximo ao sink

Faça a validação imediatamente antes da chamada HTTP sempre que possível.

### 5. Desabilite redirects automáticos

Ou valide novamente o destino final.

### 6. Separe URI e body

Entrada de usuário pode compor body ou parâmetros de negócio, mas não deve controlar host/esquema/porta.

### 7. Configure timeouts

Timeout não corrige SSRF, mas evita travamentos e reduz impacto operacional.

### 8. Use egress control

A infraestrutura deve limitar para onde a aplicação pode sair.

### 9. Não logue segredos

Falhas de integração não devem registrar tokens, credenciais ou bodies sensíveis.

### 10. Crie testes

Teste HTTP rejeitado, host não permitido, URI relativa, porta indevida e redirect quando aplicável.

## Erros comuns

### 1. Validar apenas `startsWith("https://")`

Isso não garante host permitido.

### 2. Usar denylist

Denylist é frágil para SSRF.

### 3. Aceitar URL livre por “flexibilidade”

Flexibilidade de integração pode virar controle externo do servidor.

### 4. Confiar em autenticação

Usuários autenticados também podem explorar falhas.

### 5. Validar longe do sink

O dado pode mudar entre a validação e o uso.

### 6. Esquecer redirects

Redirect pode transformar destino permitido em destino não permitido.

### 7. Misturar destino com payload

URI e body devem ter responsabilidades separadas.

### 8. Tratar configuração como sempre confiável

Configuração também pode estar errada ou comprometida.

### 9. Esconder fluxo do scanner sem controle real

Wrappers artificiais não melhoram segurança.

### 10. Não documentar risco residual

Toda mitigação deve deixar claro o que ainda depende de infraestrutura, governança ou contrato.

## Decisões de engenharia

## Quando aceitar URL externa?

Evite aceitar URL externa sempre que possível.

Só considere aceitar URL externa quando essa for realmente a regra de negócio, como em um produto cujo objetivo é validar ou monitorar URLs informadas por clientes.

Mesmo nesses casos, a aplicação precisa de controles muito mais fortes:

- validação de esquema;
- validação de host;
- bloqueio de redes internas;
- resolução DNS controlada;
- egress control;
- sandbox;
- proxy;
- limites de tempo;
- limites de tamanho;
- redirects controlados;
- auditoria.

Para sistemas corporativos comuns, aceitar URL livre raramente é necessário.

## Quando usar allowlist?

Use allowlist quando os destinos são conhecidos.

Esse é o cenário mais comum em integrações corporativas:

- API de parceiro;
- gateway interno;
- serviço de documentos;
- serviço de autenticação;
- storage corporativo;
- bureau externo;
- plataforma de pagamento.

## Quando usar configuração?

Use configuração para destinos que variam por ambiente, como homologação e produção.

Mas valide a configuração.

Configuração não deve ser um bypass da política de segurança.

## Quando usar proxy corporativo?

Use proxy ou gateway outbound quando a empresa precisa controlar chamadas de saída de forma centralizada.

Isso ajuda em:

- auditoria;
- bloqueio de destinos;
- observabilidade;
- aplicação de política;
- rastreabilidade;
- resposta a incidentes.

Mesmo usando proxy, o código deve manter validação de destino.

## Quando classificar como falso positivo?

Considere falso positivo quando:

- o usuário não controla o destino;
- o valor apontado pelo scanner vai para o body, não para a URI;
- a URI vem de allowlist;
- há validação imediatamente antes do sink;
- redirects estão bloqueados ou revalidados;
- existem testes;
- há evidência clara de que a exploração não ocorre.

## Onde isso aparece em sistemas reais

SSRF aparece em muitas funcionalidades:

- importação por URL;
- upload para serviço externo;
- download de arquivo remoto;
- preview de link;
- leitura de avatar por URL;
- integração com APIs parceiras;
- webhooks;
- geração de PDF com imagens externas;
- conversão de documentos;
- validação de callback;
- proxy interno;
- upload de anexos;
- serviços de transferência;
- clientes HTTP genéricos;
- SDKs que escondem chamadas HTTP.

Em sistemas Java/Spring, os pontos mais comuns são services de integração, classes utilitárias, clientes HTTP e módulos de arquivo/anexo.

## Checklist prático

### Entrada

- O usuário envia URL completa?
- O usuário controla host?
- O usuário controla esquema?
- O usuário controla porta?
- O usuário controla path usado para montar URL?
- O usuário controla identificador que vira URL?
- O valor vem de header?
- O valor vem de arquivo ou storage?

### Destino

- O destino vem de allowlist?
- O destino vem de configuração controlada?
- O destino é HTTPS?
- O host é permitido?
- A porta é permitida?
- A URI é absoluta?
- Redirects são bloqueados ou revalidados?

### Código

- A validação ocorre perto do sink?
- URI e body estão separados?
- Existe concatenação de URL?
- `RestClient`, `RestTemplate`, `WebClient` ou `HttpClient` usam destino validado?
- Existem timeouts?
- Erros não expõem URI sensível?
- Logs não registram credenciais?

### Testes

- HTTP é rejeitado?
- URI relativa é rejeitada?
- Host não permitido é rejeitado?
- Porta não permitida é rejeitada?
- Parceiro válido é aceito?
- Redirect é tratado?
- Configuração inválida falha fechado?

### Infraestrutura

- Existe egress control?
- Existe proxy corporativo?
- Chamadas outbound são monitoradas?
- Serviços internos sensíveis são protegidos?
- A aplicação tem menor privilégio?
- Há alertas para destinos inesperados?

## Resumo final

SSRF acontece quando uma aplicação backend faz requisições server-side para destinos controlados, direta ou indiretamente, por entrada não confiável.

Em aplicações Java/Spring, o risco costuma aparecer em chamadas outbound com `RestClient`, `RestTemplate`, `WebClient`, `HttpClient`, SDKs internos, upload/download e serviços de integração.

As principais defesas são:

- não aceitar URL livre;
- usar identificadores de negócio;
- resolver endpoints internamente;
- aplicar allowlist;
- validar HTTPS, host, porta e URI absoluta;
- separar destino de body;
- desabilitar redirects automáticos;
- validar próximo ao sink;
- configurar timeouts;
- aplicar egress control;
- criar testes;
- documentar risco residual.

A principal lição é:

> O usuário pode escolher uma ação de negócio. Ele não deve escolher livremente para onde o servidor vai se conectar.

## Perguntas de revisão

### 1. O que é SSRF?

SSRF é uma vulnerabilidade em que um atacante consegue induzir o servidor da aplicação a fazer requisições para destinos não pretendidos.

### 2. Por que SSRF é perigoso?

Porque a requisição parte do servidor, que pode ter acesso a redes, serviços e credenciais que o usuário não acessaria diretamente.

### 3. Qual é o padrão típico de SSRF em SAST?

Entrada controlada pelo usuário chega a um sink de chamada HTTP ou transferência remota sem validação adequada do destino.

### 4. HTTPS resolve SSRF?

Não. HTTPS protege o canal, mas não garante que o destino é permitido.

### 5. Por que `startsWith("https://")` é insuficiente?

Porque não valida host, porta, allowlist, normalização, redirects ou destino final.

### 6. Qual é a melhor defesa contra SSRF?

Evitar URL livre e usar allowlist de destinos permitidos.

### 7. O usuário pode escolher um parceiro?

Sim, desde que escolha uma opção de negócio controlada, como um enum, e a aplicação resolva internamente o endpoint real.

### 8. O que é allowlist?

É uma lista positiva de destinos explicitamente permitidos. Tudo que não está na lista é rejeitado.

### 9. Por que redirects são perigosos em SSRF?

Porque um destino inicialmente permitido pode redirecionar para outro destino não validado.

### 10. O que significa falhar fechado?

Significa rejeitar a operação quando a aplicação não consegue validar o destino com segurança.

### 11. Configuração é sempre confiável?

Não. Configuração pode estar errada ou comprometida. Ela também deve ser validada.

### 12. Qual é a diferença entre SSRF e Open Redirect?

No Open Redirect, o navegador do usuário é redirecionado. No SSRF, o servidor faz a requisição.

### 13. Timeouts corrigem SSRF?

Não. Timeouts ajudam na resiliência, mas não impedem que o servidor acesse destino indevido.

### 14. Quando um finding SSRF pode ser falso positivo?

Quando o usuário não controla o destino, o valor vai apenas para o body, há allowlist validada, redirects estão controlados e existem evidências de teste.

### 15. Qual é a regra mais importante?

O destino da chamada outbound deve ser controlado pela aplicação, não pelo usuário.

## Referências

- [OWASP Server-Side Request Forgery Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Server_Side_Request_Forgery_Prevention_Cheat_Sheet.html)
- [OWASP Top 10:2021 — A10 Server-Side Request Forgery](https://top10.owasp.org/2021/A10_2021-Server-Side_Request_Forgery_%28SSRF%29/)
- [OWASP Top 10:2021 — A10 SSRF em português](https://top10.owasp.org/2021/pt-BR/A10_2021-Server-Side_Request_Forgery_%28SSRF%29/)
- [OWASP Proactive Controls — C10 Stop Server Side Request Forgery](https://top10proactive.owasp.org/the-top-10/c10-stop-server-side-request-forgery/)
- [CWE-918 — Server-Side Request Forgery](https://cwe.mitre.org/data/definitions/918.html)
- [Spring Framework Reference — REST Clients](https://docs.spring.io/spring-framework/reference/integration/rest-clients.html)
- [Spring Boot Reference — Calling REST Services](https://docs.spring.io/spring-boot/reference/io/rest-client.html)
- [Oracle JDK 25 Documentation](https://docs.oracle.com/en/java/javase/25/)
- [Oracle Java Downloads — JDK 25 LTS](https://www.oracle.com/java/technologies/downloads/)