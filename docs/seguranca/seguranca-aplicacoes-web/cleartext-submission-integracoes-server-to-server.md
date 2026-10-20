# Cleartext Submission em integrações server-to-server

`Cleartext Submission of Sensitive Information` é uma vulnerabilidade relacionada ao envio de dados sensíveis por um canal sem proteção adequada.

Em termos simples, ela acontece quando uma aplicação transmite informações sensíveis, como credenciais, tokens, senhas, documentos, dados pessoais ou segredos de integração, por um canal que pode ser observado por terceiros.

Em aplicações Java/Spring, esse problema costuma aparecer em integrações server-to-server, principalmente quando o backend chama outro sistema usando HTTP sem TLS, configuração incompleta ou validação fraca do endpoint.

Exemplos comuns:

- envio de `client_id` e `client_secret` para endpoint OAuth;
- envio de usuário e senha para sistema legado;
- chamada `application/x-www-form-urlencoded` com credenciais;
- integração com bureau externo;
- envio de token em header;
- envio de arquivo com dados sensíveis;
- chamada para API parceira usando endpoint configurável;
- uso de `RestClient`, `RestTemplate`, `WebClient` ou `HttpClient`;
- configuração que aceita `http://` por engano;
- fallback inseguro quando endpoint HTTPS falha.

A ideia central é:

> Se o dado é sensível, ele não deve trafegar por um canal sem proteção criptográfica adequada.

No contexto da aplicação fictícia `meucatalogomusical.com`, imagine uma integração com um parceiro chamado `RockPartner`, responsável por validar dados de licenciamento de bandas e liberar acesso a conteúdos exclusivos. Para consultar esse parceiro, a aplicação precisa enviar `client_id`, `client_secret` e dados da banda consultada.

Se essa chamada for feita por HTTP, as credenciais podem ficar expostas no caminho entre os servidores.

Esse é o problema que Cleartext Submission tenta detectar.

## O problema real que esse conceito resolve

Em sistemas corporativos, integrações server-to-server são muito comuns.

Uma aplicação raramente vive sozinha. Ela chama:

- APIs internas;
- sistemas legados;
- serviços de autenticação;
- gateways corporativos;
- bureaus externos;
- parceiros comerciais;
- serviços de storage;
- filas;
- proxies;
- sistemas de documentos;
- plataformas de pagamento;
- serviços de observabilidade.

Essas chamadas frequentemente carregam dados sensíveis.

O risco surge quando a equipe trata uma integração interna como automaticamente segura.

Frases comuns em projetos reais:

```text
É interno, então pode ser HTTP.
```

```text
Está dentro da rede da empresa.
```

```text
Só o sistema legado chama.
```

```text
Esse endpoint nunca vai para internet.
```

Essas premissas podem até reduzir o risco, mas não eliminam a necessidade de proteger o canal.

Redes internas também podem ter:

- proxies;
- balanceadores;
- logs intermediários;
- ambientes compartilhados;
- tráfego lateral;
- configurações erradas;
- capturas de pacotes;
- serviços comprometidos;
- usuários internos maliciosos;
- vazamentos por observabilidade;
- problemas de roteamento.

Por isso, a regra segura é:

> Dados sensíveis devem trafegar por HTTPS/TLS também em integrações server-to-server.

## Antes de entrar nos detalhes: uma ideia simples

Pense no `meucatalogomusical.com`.

A aplicação possui uma integração para liberar conteúdo premium de bandas como **Dokken**, **Twisted Sister** e **Skid Row**.

Para consultar o parceiro, o backend precisa enviar:

```text
client_id=meucatalogo
client_secret=segredo-super-confidencial
grant_type=client_credentials
```

Se a aplicação envia isso para:

```text
http://auth.rockpartner.example/oauth/token
```

há risco de Cleartext Submission.

Se envia para:

```text
https://auth.rockpartner.example/oauth/token
```

o canal passa a usar TLS.

Mas ainda não acabou.

A aplicação também precisa garantir que:

- o endpoint é realmente HTTPS;
- a URI é absoluta;
- o host é permitido;
- redirects não levem para HTTP;
- credenciais não apareçam em logs;
- erros não exponham payloads;
- o endpoint validado é o mesmo usado no sink HTTP;
- não existe fallback para HTTP;
- o contrato legado foi preservado;
- o novo comportamento falha fechado.

HTTPS é a base, mas a engenharia em volta também importa.

## Definição técnica

Cleartext Submission of Sensitive Information ocorre quando uma aplicação envia dados sensíveis por um canal sem criptografia ou com proteção insuficiente.

Em um relatório SAST, o finding costuma aparecer como um fluxo entre:

```text
source: dado sensível
    -> montagem de request
    -> chamada HTTP
sink: envio outbound por canal potencialmente inseguro
```

Exemplos de source:

- senha;
- token;
- `client_secret`;
- `Authorization` header;
- API key;
- credenciais decodificadas;
- CPF/CNPJ;
- e-mail;
- dados pessoais;
- documento;
- payload de autenticação;
- arquivo sensível.

Exemplos de sink:

- `RestClient.post()`;
- `RestTemplate.postForEntity()`;
- `WebClient.post()`;
- `HttpClient.send()`;
- SDK interno de transferência;
- método utilitário de upload;
- cliente legado;
- chamada `form-urlencoded`;
- envio multipart;
- envio JSON para parceiro.

O problema não é enviar credenciais. Muitas integrações exigem isso.

O problema é enviar credenciais por canal que não garante confidencialidade e integridade adequadas.

## Cleartext não é apenas “senha em texto”

O nome pode dar a impressão de que o problema acontece apenas quando a senha aparece literalmente em texto.

Mas o conceito é mais amplo.

Pode envolver:

- senha;
- token;
- segredo;
- header de autenticação;
- documento;
- identificador pessoal;
- payload com dados privados;
- arquivo;
- resposta de parceiro;
- corpo de autenticação;
- dados de negócio confidenciais.

Além disso, codificar não é criptografar.

Base64, URL encoding e form encoding não protegem segredo.

Exemplo:

```text
client_secret=abc123
```

Enviado como `application/x-www-form-urlencoded` continua sendo sensível.

Mesmo que vire:

```text
client_secret=abc123%21
```

isso não é criptografia.

O canal ainda precisa ser protegido por TLS.

## Server-to-server também precisa de TLS

Um erro comum é pensar que HTTPS é importante apenas entre navegador e backend.

Mas integrações backend-to-backend também carregam dados críticos.

Exemplo:

```text
meucatalogomusical.com
    -> auth.rockpartner.example
    -> catalog-api.partner.example
    -> payment-gateway.example
```

Cada seta é uma comunicação que pode envolver dados sensíveis.

Se uma dessas setas usa HTTP, há risco.

Em aplicações corporativas, o tráfego pode passar por múltiplos componentes:

- API Gateway;
- proxy;
- load balancer;
- service mesh;
- firewall;
- NAT;
- observabilidade;
- logs de rede;
- appliance de segurança;
- infraestrutura compartilhada.

Por isso, proteger apenas a entrada pública da aplicação não basta.

## Exemplo inseguro com RestClient

Imagine uma integração de `meucatalogomusical.com` com um parceiro que fornece dados de licenciamento para conteúdos da banda **Europe**.

Controller:

```java
package com.meucatalogomusical.licenciamento.api;

import jakarta.validation.Valid;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/licenciamentos")
public class LicenciamentoController {

    private final LicenciamentoService licenciamentoService;

    public LicenciamentoController(LicenciamentoService licenciamentoService) {
        this.licenciamentoService = licenciamentoService;
    }

    @PostMapping("/consulta")
    public LicenciamentoResponse consultar(@Valid @RequestBody LicenciamentoRequest request) {
        return licenciamentoService.consultar(request);
    }
}
```

Request:

```java
package com.meucatalogomusical.licenciamento.api;

import jakarta.validation.constraints.NotBlank;
import jakarta.validation.constraints.Size;

public record LicenciamentoRequest(

        @NotBlank
        @Size(max = 120)
        String nomeBanda
) {
}
```

Service inseguro:

```java
package com.meucatalogomusical.licenciamento.application;

import com.meucatalogomusical.licenciamento.api.LicenciamentoRequest;
import com.meucatalogomusical.licenciamento.api.LicenciamentoResponse;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.http.MediaType;
import org.springframework.stereotype.Service;
import org.springframework.web.client.RestClient;

@Service
public class LicenciamentoService {

    private final RestClient restClient;

    @Value("${integracoes.rockpartner.token-url}")
    private String tokenUrl;

    @Value("${integracoes.rockpartner.client-id}")
    private String clientId;

    @Value("${integracoes.rockpartner.client-secret}")
    private String clientSecret;

    public LicenciamentoService(RestClient.Builder restClientBuilder) {
        this.restClient = restClientBuilder.build();
    }

    public LicenciamentoResponse consultar(LicenciamentoRequest request) {
        String token = restClient
                .post()
                .uri(tokenUrl)
                .contentType(MediaType.APPLICATION_FORM_URLENCODED)
                .body("client_id=" + clientId
                        + "&client_secret=" + clientSecret
                        + "&grant_type=client_credentials")
                .retrieve()
                .body(String.class);

        return new LicenciamentoResponse(request.nomeBanda(), token == null ? "SEM_TOKEN" : "CONSULTADO");
    }
}
```

O problema está na combinação:

```java
.uri(tokenUrl)
```

com credenciais no corpo:

```java
client_secret=" + clientSecret
```

Se `tokenUrl` estiver configurada como `http://...`, as credenciais trafegam sem TLS.

O SAST pode apontar:

```text
Query: Cleartext Submission of Sensitive Information
Source: clientSecret
Sink: RestClient.post().uri(tokenUrl)
```

Mesmo que o valor venha de configuração, o risco existe se a aplicação não impõe HTTPS.

## Corrigindo com validação de URI HTTPS

Uma correção mais robusta é criar uma fronteira de validação outbound.

```java
package com.meucatalogomusical.integracoes.infra;

import org.springframework.stereotype.Component;

import java.net.URI;

@Component
public class HttpsEndpointValidator {

    public URI requireHttps(URI uri) {
        if (uri == null) {
            throw new IllegalArgumentException("Endpoint é obrigatório.");
        }

        if (!uri.isAbsolute()) {
            throw new IllegalArgumentException("Endpoint deve ser absoluto.");
        }

        if (!"https".equalsIgnoreCase(uri.getScheme())) {
            throw new IllegalArgumentException("Endpoint deve usar HTTPS.");
        }

        if (uri.getHost() == null || uri.getHost().isBlank()) {
            throw new IllegalArgumentException("Endpoint deve possuir host.");
        }

        return uri.normalize();
    }
}
```

Esse validador garante:

- URI obrigatória;
- URI absoluta;
- esquema HTTPS;
- host presente;
- normalização.

Esse controle não resolve todos os riscos de SSRF, mas resolve o ponto central do Cleartext: impedir que dados sensíveis sejam enviados por canal HTTP.

Em integrações críticas, o ideal é combinar com allowlist de hosts, como vimos no artigo sobre SSRF.

## Corrigindo o service

Agora o endpoint é transformado em `URI` e validado antes do sink HTTP.

```java
package com.meucatalogomusical.licenciamento.application;

import com.meucatalogomusical.integracoes.infra.HttpsEndpointValidator;
import com.meucatalogomusical.licenciamento.api.LicenciamentoRequest;
import com.meucatalogomusical.licenciamento.api.LicenciamentoResponse;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.http.MediaType;
import org.springframework.stereotype.Service;
import org.springframework.util.LinkedMultiValueMap;
import org.springframework.util.MultiValueMap;
import org.springframework.web.client.RestClient;

import java.net.URI;

@Service
public class LicenciamentoService {

    private final RestClient restClient;
    private final HttpsEndpointValidator endpointValidator;
    private final URI tokenEndpoint;
    private final String clientId;
    private final String clientSecret;

    public LicenciamentoService(
            RestClient.Builder restClientBuilder,
            HttpsEndpointValidator endpointValidator,
            @Value("${integracoes.rockpartner.token-url}") URI tokenEndpoint,
            @Value("${integracoes.rockpartner.client-id}") String clientId,
            @Value("${integracoes.rockpartner.client-secret}") String clientSecret
    ) {
        this.restClient = restClientBuilder.build();
        this.endpointValidator = endpointValidator;
        this.tokenEndpoint = tokenEndpoint;
        this.clientId = clientId;
        this.clientSecret = clientSecret;
    }

    public LicenciamentoResponse consultar(LicenciamentoRequest request) {
        URI endpointValidado = endpointValidator.requireHttps(tokenEndpoint);

        MultiValueMap<String, String> body = new LinkedMultiValueMap<>();
        body.add("client_id", clientId);
        body.add("client_secret", clientSecret);
        body.add("grant_type", "client_credentials");

        String tokenResponse = restClient
                .post()
                .uri(endpointValidado)
                .contentType(MediaType.APPLICATION_FORM_URLENCODED)
                .body(body)
                .retrieve()
                .body(String.class);

        String status = tokenResponse == null || tokenResponse.isBlank()
                ? "SEM_TOKEN"
                : "CONSULTADO";

        return new LicenciamentoResponse(request.nomeBanda(), status);
    }
}
```

Resposta:

```java
package com.meucatalogomusical.licenciamento.api;

public record LicenciamentoResponse(
        String nomeBanda,
        String status
) {
}
```

Esse exemplo melhora o desenho em vários pontos:

- o endpoint precisa ser HTTPS;
- a URI é validada antes do envio;
- credenciais vão no body funcional exigido pelo protocolo;
- o body não é montado por concatenação manual;
- a resposta pública não devolve o token;
- a aplicação falha se o endpoint estiver mal configurado.

## Por que validar perto do sink?

Em SAST, é comum o scanner rastrear o fluxo até o ponto de envio.

Se a validação acontece muito longe, pode ficar difícil provar que o valor validado é o mesmo valor usado no sink.

Por isso, uma prática defensiva é validar imediatamente antes da chamada HTTP.

Exemplo:

```java
URI endpointValidado = endpointValidator.requireHttps(tokenEndpoint);

restClient
        .post()
        .uri(endpointValidado)
        .body(body)
        .retrieve()
        .body(String.class);
```

Essa estrutura deixa o invariante explícito:

> O sink recebe uma URI validada como HTTPS.

Isso ajuda:

- o desenvolvedor;
- o revisor;
- o time de AppSec;
- os testes;
- o SAST;
- a documentação da triagem.

## HTTPS obrigatório não é detalhe operacional

Não trate HTTPS como uma recomendação opcional.

Para dados sensíveis, HTTPS é requisito de segurança.

Exemplo de configuração inválida:

```yaml
integracoes:
  rockpartner:
    token-url: http://auth.rockpartner.example/oauth/token
```

Exemplo correto:

```yaml
integracoes:
  rockpartner:
    token-url: https://auth.rockpartner.example/oauth/token
```

Mas a aplicação não deve depender apenas da revisão visual do YAML.

Ela deve rejeitar a configuração inválida em runtime.

## Falha fechada

A aplicação deve falhar fechada quando o endpoint não é seguro.

Falhar fechado significa:

- não tentar chamada HTTP;
- não fazer fallback para HTTP;
- não enviar credenciais;
- não mascarar o problema;
- não seguir com comportamento inseguro;
- gerar erro controlado;
- não expor segredos na mensagem de erro.

Exemplo ruim:

```java
try {
    return chamarHttps();
} catch (Exception ex) {
    return chamarHttpFallback();
}
```

Exemplo melhor:

```java
URI endpointValidado = endpointValidator.requireHttps(tokenEndpoint);
```

Se a validação falhar, a chamada não acontece.

Esse comportamento é intencional.

## Redirects e Cleartext

Redirects podem reintroduzir risco.

Fluxo perigoso:

```text
POST https://auth.rockpartner.example/oauth/token
    -> 302 Location: http://auth-backup.rockpartner.example/oauth/token
    -> cliente segue redirect
    -> credenciais são reenviadas por HTTP
```

Por isso, em chamadas que carregam credenciais, é recomendável:

- não seguir redirects automaticamente;
- ou revalidar o destino final antes de reenviar qualquer dado sensível;
- evitar reenviar bodies sensíveis após redirect;
- testar comportamento do cliente HTTP usado.

Em integrações server-to-server, redirects raramente são necessários para endpoints de autenticação e APIs estáveis.

Se houver redirect, ele precisa ser uma decisão explícita, não comportamento automático invisível.

## Exemplo com Java HttpClient

Com `java.net.http.HttpClient`, é possível deixar redirect desabilitado explicitamente.

```java
package com.meucatalogomusical.integracoes.infra;

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

Esse cliente:

- define timeout;
- não segue redirects automaticamente.

Timeout não corrige Cleartext, mas melhora resiliência.

Redirect desabilitado evita reenviar dados sensíveis para destino não validado.

## Exemplo de envio seguro com HttpClient

```java
package com.meucatalogomusical.integracoes.infra;

import org.springframework.stereotype.Component;

import java.io.IOException;
import java.net.URI;
import java.net.URLEncoder;
import java.net.http.HttpRequest;
import java.net.http.HttpResponse;
import java.nio.charset.StandardCharsets;

@Component
public class TokenPartnerClient {

    private final SecureHttpClientFactory httpClientFactory;
    private final HttpsEndpointValidator endpointValidator;

    public TokenPartnerClient(
            SecureHttpClientFactory httpClientFactory,
            HttpsEndpointValidator endpointValidator
    ) {
        this.httpClientFactory = httpClientFactory;
        this.endpointValidator = endpointValidator;
    }

    public String solicitarToken(
            URI tokenEndpoint,
            String clientId,
            String clientSecret
    ) {
        URI endpointValidado = endpointValidator.requireHttps(tokenEndpoint);

        String body = "client_id=" + encode(clientId)
                + "&client_secret=" + encode(clientSecret)
                + "&grant_type=client_credentials";

        HttpRequest request = HttpRequest.newBuilder(endpointValidado)
                .header("Content-Type", "application/x-www-form-urlencoded")
                .timeout(java.time.Duration.ofSeconds(5))
                .POST(HttpRequest.BodyPublishers.ofString(body))
                .build();

        try {
            HttpResponse<String> response = httpClientFactory.create()
                    .send(request, HttpResponse.BodyHandlers.ofString());

            return response.body();
        } catch (IOException ex) {
            throw new IllegalStateException("Falha de comunicação com parceiro.", ex);
        } catch (InterruptedException ex) {
            Thread.currentThread().interrupt();
            throw new IllegalStateException("Chamada ao parceiro foi interrompida.", ex);
        }
    }

    private String encode(String value) {
        return URLEncoder.encode(value, StandardCharsets.UTF_8);
    }
}
```

Esse exemplo demonstra:

- endpoint validado antes do `HttpRequest`;
- body separado da URI;
- form encoding para o corpo;
- timeout;
- redirect controlado no `HttpClient`;
- exceção pública sem expor credenciais.

Atenção: `URLEncoder` protege a formação do body `application/x-www-form-urlencoded`; ele não protege o canal. A proteção do canal vem do HTTPS/TLS.

## Separar endpoint, headers e body

Uma das práticas mais importantes é manter separação clara entre:

- endpoint;
- headers;
- body;
- credenciais;
- dados de negócio.

Exemplo ruim:

```java
String url = tokenUrl
        + "?client_id=" + clientId
        + "&client_secret=" + clientSecret;

restClient.post().uri(url).retrieve();
```

Problemas:

- segredo aparece na URL;
- URL pode ir para logs;
- segredo pode aparecer em histórico de proxy;
- segredo pode aparecer em métricas;
- segredo pode aparecer em erro;
- destino e credencial ficam misturados.

Exemplo melhor:

```java
MultiValueMap<String, String> body = new LinkedMultiValueMap<>();
body.add("client_id", clientId);
body.add("client_secret", clientSecret);
body.add("grant_type", "client_credentials");

restClient
        .post()
        .uri(endpointValidado)
        .contentType(MediaType.APPLICATION_FORM_URLENCODED)
        .body(body)
        .retrieve()
        .body(String.class);
```

Agora:

- destino é uma URI validada;
- credenciais estão no body exigido pelo protocolo;
- body não define destino;
- logs podem evitar registrar o body;
- o canal HTTPS protege a transmissão.

## Dados sensíveis não devem ir na URL

Mesmo com HTTPS, evite colocar dados sensíveis na URL.

Motivos:

- URLs podem aparecer em logs;
- proxies podem registrar path e query string;
- ferramentas APM podem capturar URLs;
- gateways podem registrar rotas;
- erros podem incluir URL;
- métricas podem indexar query string;
- histórico e debug podem expor valores.

Exemplo ruim:

```text
https://auth.rockpartner.example/oauth/token?client_secret=abc123
```

Exemplo melhor:

```text
POST https://auth.rockpartner.example/oauth/token
Content-Type: application/x-www-form-urlencoded
```

Com o segredo no corpo e o canal protegido por HTTPS.

## Cleartext e logs

Cleartext Submission trata principalmente transmissão por canal inseguro, mas logs também entram no raciocínio de exposição de dados.

Não adianta enviar por HTTPS e depois registrar o body completo no log.

Exemplo ruim:

```java
log.info("Enviando body ao parceiro: {}", body);
```

Exemplo melhor:

```java
log.info("Solicitando token ao parceiro RockPartner.");
```

Em erro:

```java
log.warn("Falha ao solicitar token ao parceiro. tipoErro={}", ex.getClass().getSimpleName());
```

Evite registrar:

- `client_secret`;
- senha;
- token;
- header `Authorization`;
- cookie;
- payload completo de autenticação;
- resposta completa de token;
- URL com query string sensível;
- stack trace contendo body ou headers.

Logs precisam ser úteis, mas não podem virar canal de vazamento.

## Cleartext e mensagens de erro

Erros também podem vazar dados.

Exemplo ruim:

```java
throw new IllegalStateException("Erro ao chamar " + tokenEndpoint + " com secret " + clientSecret);
```

Exemplo melhor:

```java
throw new IllegalStateException("Falha ao consultar serviço de autenticação parceiro.");
```

A mensagem pública deve informar o suficiente para diagnóstico operacional, sem expor segredo.

Detalhes técnicos podem ir para logs controlados, mas ainda assim sem credenciais.

## OAuth client_credentials e Cleartext

O fluxo `client_credentials` exige envio de credenciais do cliente para o servidor de autorização.

Isso é normal.

O envio de `client_id` e `client_secret` não é automaticamente vulnerabilidade.

O risco aparece se:

- o endpoint usa HTTP;
- o endpoint é controlável pelo usuário;
- o host não é permitido;
- redirects podem levar para HTTP;
- credenciais aparecem em logs;
- resposta pública reflete segredo;
- erro inclui payload;
- token é devolvido para cliente indevido;
- configuração pode ser alterada sem governança.

Ou seja:

> O fluxo funcional pode ser legítimo, mas precisa ter controles.

Em um finding SAST, isso pode ser classificado como vulnerabilidade real, risco mitigado ou `Not Exploitable`, dependendo das evidências.

## Tratamento de resposta do parceiro

Além de proteger o envio, é importante cuidar da resposta.

Um endpoint de token pode retornar:

```json
{
  "access_token": "token-gerado",
  "token_type": "Bearer",
  "expires_in": 3600
}
```

Esse retorno é sensível.

A aplicação deve avaliar:

- quem realmente precisa receber esse token;
- se o token deve ficar apenas no backend;
- se deve ser armazenado em cache;
- se logs devem ocultar a resposta;
- se campos inesperados devem ser filtrados;
- se o parceiro poderia refletir dados sensíveis enviados.

Em integrações legadas, às vezes o contrato exige devolver a resposta completa ao consumidor. Nesses casos, o risco precisa ser documentado e mitigado tanto quanto possível.

## Bloqueio de reflexão de credenciais

Um controle útil é rejeitar respostas de parceiro que ecoem credenciais recebidas.

Exemplo conceitual:

```java
package com.meucatalogomusical.integracoes.infra;

import com.fasterxml.jackson.databind.JsonNode;
import org.springframework.stereotype.Component;

import java.util.Iterator;
import java.util.Map;

@Component
public class CredentialReflectionGuard {

    public void rejectCredentialReflection(
            JsonNode response,
            String clientId,
            String clientSecret
    ) {
        if (response == null) {
            return;
        }

        verifyNode(response, clientId, clientSecret);
    }

    private void verifyNode(JsonNode node, String clientId, String clientSecret) {
        if (node.isTextual()) {
            String value = node.asText();

            if (value.equals(clientSecret)) {
                throw new IllegalStateException("Resposta do parceiro refletiu segredo.");
            }

            return;
        }

        if (node.isObject()) {
            Iterator<Map.Entry<String, JsonNode>> fields = node.fields();

            while (fields.hasNext()) {
                Map.Entry<String, JsonNode> field = fields.next();

                if (isSecretField(field.getKey())) {
                    throw new IllegalStateException("Resposta do parceiro contém campo sensível.");
                }

                verifyNode(field.getValue(), clientId, clientSecret);
            }

            return;
        }

        if (node.isArray()) {
            for (JsonNode item : node) {
                verifyNode(item, clientId, clientSecret);
            }
        }
    }

    private boolean isSecretField(String fieldName) {
        String normalized = fieldName.toLowerCase();

        return normalized.contains("password")
                || normalized.contains("secret")
                || normalized.contains("credential")
                || normalized.contains("authorization");
    }
}
```

Esse tipo de guarda não substitui HTTPS.

Ela resolve outro risco: impedir que a resposta pública carregue de volta segredo enviado ao parceiro.

Em sistemas reais, essa regra precisa ser cuidadosamente testada para não quebrar contratos válidos.

## mTLS

HTTPS autentica o servidor para o cliente e protege o canal com TLS.

Em integrações mais sensíveis, pode ser necessário usar mTLS, Mutual TLS.

No mTLS, ambos os lados apresentam certificados:

- o cliente valida o servidor;
- o servidor valida o cliente.

Isso é útil quando a integração precisa garantir que apenas clientes autorizados consigam se conectar.

mTLS pode ser útil em:

- integrações financeiras;
- APIs internas críticas;
- comunicação entre microsserviços;
- parceiros de alta criticidade;
- ambientes com requisitos regulatórios;
- tráfego entre domínios corporativos.

Mas mTLS não substitui boas práticas de aplicação:

- não logar segredo;
- validar endpoint;
- não aceitar HTTP;
- controlar redirects;
- aplicar menor privilégio;
- proteger tokens;
- criar testes.

## TLS não protege dados em repouso

TLS protege dados em trânsito.

Depois que a mensagem chega ao destino, o dado pode:

- ser processado;
- entrar em log;
- ser armazenado;
- ir para cache;
- ser retornado;
- ser propagado para outro sistema;
- ser exibido em erro;
- ser persistido em banco.

Portanto, TLS é necessário, mas não resolve tudo.

Para dados sensíveis, pense no ciclo completo:

```text
entrada -> processamento -> transmissão -> resposta -> logs -> cache -> armazenamento -> descarte
```

Cleartext Submission foca na transmissão, mas a segurança real precisa olhar o fluxo inteiro.

## Testes do validador HTTPS

Teste o comportamento esperado da política.

```java
package com.meucatalogomusical.integracoes.infra;

import org.junit.jupiter.api.Test;

import java.net.URI;

import static org.junit.jupiter.api.Assertions.*;

class HttpsEndpointValidatorTest {

    private final HttpsEndpointValidator validator = new HttpsEndpointValidator();

    @Test
    void deveAceitarEndpointHttpsAbsolutoComHost() {
        URI endpoint = URI.create("https://auth.rockpartner.example/oauth/token");

        URI validado = validator.requireHttps(endpoint);

        assertEquals(endpoint, validado);
    }

    @Test
    void deveRejeitarEndpointHttp() {
        URI endpoint = URI.create("http://auth.rockpartner.example/oauth/token");

        assertThrows(IllegalArgumentException.class,
                () -> validator.requireHttps(endpoint));
    }

    @Test
    void deveRejeitarUriRelativa() {
        URI endpoint = URI.create("/oauth/token");

        assertThrows(IllegalArgumentException.class,
                () -> validator.requireHttps(endpoint));
    }

    @Test
    void deveRejeitarUriSemHost() {
        URI endpoint = URI.create("https:///oauth/token");

        assertThrows(IllegalArgumentException.class,
                () -> validator.requireHttps(endpoint));
    }

    @Test
    void deveRejeitarEndpointNulo() {
        assertThrows(IllegalArgumentException.class,
                () -> validator.requireHttps(null));
    }
}
```

Esses testes comprovam que a aplicação falha fechado quando o endpoint não é seguro.

## Teste para não registrar segredo

Um teste completo de logs depende da configuração de logging do projeto, mas a ideia é validar que marcadores sensíveis não aparecem.

Exemplo conceitual:

```java
package com.meucatalogomusical.integracoes.infra;

import org.junit.jupiter.api.Test;

import static org.junit.jupiter.api.Assertions.assertFalse;

class LogSanitizationTest {

    @Test
    void naoDeveRegistrarClientSecret() {
        String clientSecret = "SEGREDO-TESTE-DEF-LEPPARD";

        String mensagemLog = "Falha ao solicitar token ao parceiro.";

        assertFalse(mensagemLog.contains(clientSecret));
    }
}
```

Em um projeto real, o ideal seria capturar o appender de log em teste e procurar por marcadores sintéticos.

Exemplo de marcadores:

```text
SEGREDO-TESTE-DEF-LEPPARD
TOKEN-TESTE-POISON
AUTHORIZATION-TESTE-CINDERELLA
```

A regra é: nenhum marcador sensível deve aparecer nos logs.

## Testes de integração com servidor fake

Para validar que o endpoint HTTPS é obrigatório sem chamar um parceiro real, é possível usar servidor fake em testes.

A ideia geral:

- configurar endpoint `http://...`;
- executar o fluxo;
- verificar que a aplicação rejeita antes da chamada;
- configurar endpoint `https://...`;
- verificar que a requisição é montada corretamente;
- validar que credenciais não aparecem em logs.

Mesmo que o teste fake não valide TLS real, ele ajuda a provar que o código bloqueia HTTP antes do sink.

## Como o SAST pode enxergar a correção

Antes:

```text
clientSecret
    -> body
    -> tokenUrl possivelmente HTTP
    -> RestClient.post()
```

Depois:

```text
tokenEndpoint
    -> HttpsEndpointValidator.requireHttps
    -> endpointValidado
    -> RestClient.post(endpointValidado)

clientSecret
    -> body form-urlencoded
    -> enviado somente para endpointValidado
```

Essa separação ajuda a explicar a correção:

- o segredo continua existindo porque o protocolo exige;
- o segredo continua no body porque o contrato exige;
- o endpoint agora precisa ser HTTPS;
- a chamada falha antes do envio se a URI for insegura;
- redirects devem ser proibidos ou revalidados;
- logs não recebem segredo.

## Quando o finding pode persistir

Mesmo após corrigir, o SAST pode continuar apontando o fluxo.

Isso pode acontecer quando a ferramenta:

- não reconhece o validador customizado;
- não entende que `endpointValidado` só pode ser HTTPS;
- propaga taint pelo objeto de configuração;
- associa body sensível ao retorno da chamada;
- não diferencia endpoint de payload;
- não reconhece testes;
- não analisa configuração efetiva de redirects.

Nesse caso, não crie código artificial para enganar a ferramenta.

Faça uma triagem técnica:

1. explique source;
2. explique sink;
3. mostre a validação HTTPS;
4. mostre que a falha ocorre antes do envio;
5. mostre testes;
6. mostre que logs não contêm segredos;
7. documente risco residual;
8. solicite avaliação AppSec se necessário.

## Falso positivo, risco mitigado ou vulnerabilidade real?

### Vulnerabilidade real

É vulnerabilidade real quando:

- dados sensíveis podem ser enviados para `http://`;
- não há validação de endpoint;
- existe fallback HTTP;
- redirects podem levar para HTTP;
- o endpoint vem de usuário;
- credenciais aparecem em URL;
- credenciais aparecem em logs;
- erros expõem o payload.

### Risco mitigado

É risco mitigado quando:

- o protocolo exige envio de credencial;
- o endpoint é validado como HTTPS;
- o host é controlado ou permitido;
- redirects são proibidos ou revalidados;
- logs não expõem segredo;
- erros são sanitizados;
- testes comprovam o comportamento;
- o contrato funcional precisa ser preservado.

### Falso positivo

Pode ser falso positivo quando:

- o scanner aponta envio claro, mas o sink só recebe URI HTTPS validada;
- o valor sensível não chega ao canal indicado;
- o dado é usado apenas para HMAC/cache, não para log;
- o scanner confunde argumento enviado com resposta recebida;
- o fluxo é funcional e obrigatório, com controles demonstráveis.

Mesmo em falso positivo, pode valer aplicar hardening defensivo para deixar o controle mais explícito.

## Boas práticas

### 1. Imponha HTTPS no código

Não dependa apenas de documentação ou revisão manual.

### 2. Valide perto do sink

A URI usada pelo cliente HTTP deve ser validada imediatamente antes da chamada.

### 3. Separe URI e body

O destino não deve ser misturado com payload sensível.

### 4. Não coloque segredo na URL

Use body ou header apropriado conforme o protocolo, sempre sobre HTTPS.

### 5. Proíba fallback para HTTP

Falha em HTTPS deve ser falha operacional, não motivo para baixar segurança.

### 6. Controle redirects

Não siga redirects automaticamente quando a requisição carrega credenciais, ou revalide o destino final.

### 7. Não registre credenciais

Logs devem registrar eventos, não segredos.

### 8. Sanitize mensagens de erro

Erros não devem carregar endpoint completo com query sensível, body, token ou senha.

### 9. Use allowlist quando possível

Cleartext e SSRF muitas vezes aparecem juntos. HTTPS obrigatório é necessário; host permitido também pode ser necessário.

### 10. Escreva testes

Teste HTTP rejeitado, URI relativa rejeitada, ausência de host, logs sem segredo e comportamento com endpoint válido.

## Erros comuns

### 1. Achar que Base64 protege segredo

Base64 é codificação, não criptografia.

### 2. Achar que rede interna dispensa TLS

Rede interna também precisa de proteção, especialmente para credenciais.

### 3. Usar `http://` em homologação

Ambiente de homologação frequentemente carrega dados e credenciais reais ou parecidas com reais. Não deve ensinar a aplicação a aceitar HTTP.

### 4. Colocar segredo na query string

Query string aparece facilmente em logs e ferramentas intermediárias.

### 5. Fazer fallback para HTTP

Fallback inseguro destrói a garantia do canal.

### 6. Seguir redirects sem controle

Redirect pode levar para canal inseguro.

### 7. Logar body completo

O body pode conter segredo, token ou dados pessoais.

### 8. Corrigir removendo credencial funcional

Se o protocolo exige credencial, removê-la quebra a integração. A correção é proteger o envio, não destruir o contrato.

### 9. Mascarar dado antes de enviar

Mascarar `client_secret` antes de enviar quebra autenticação. Mascaramento é para log e resposta pública, não para o protocolo funcional.

### 10. Criar wrapper sem controle real

Wrapper vazio pode calar scanner, mas não protege o canal.

## Decisões de engenharia

## Quando bloquear HTTP totalmente?

Bloqueie HTTP sempre que a chamada carregar:

- credenciais;
- tokens;
- documentos;
- dados pessoais;
- dados financeiros;
- dados regulados;
- payloads de autenticação;
- informações confidenciais;
- dados de negócio sensíveis.

Na prática, em sistemas corporativos modernos, o padrão deve ser HTTPS.

## Quando aceitar HTTP?

Raramente.

Só faria sentido em cenários muito controlados, sem dados sensíveis, com rede isolada e justificativa formal. Mesmo assim, normalmente é melhor corrigir a infraestrutura para suportar HTTPS.

Para integrações server-to-server com credenciais, HTTP não deve ser aceito.

## Quando usar mTLS?

Use mTLS quando a integração exige autenticação forte entre serviços ou parceiros.

Exemplos:

- comunicação entre microsserviços críticos;
- APIs financeiras;
- integrações com parceiros regulados;
- ambientes com requisitos de autenticação mútua;
- sistemas com alto impacto em caso de uso indevido.

## Quando usar validação de host?

Use validação de host quando:

- há risco de SSRF;
- endpoint vem de configuração;
- há múltiplos parceiros;
- o destino pode variar por ambiente;
- você precisa provar que só chama hosts autorizados.

## Quando classificar como Not Exploitable?

Considere `Not Exploitable` quando:

- o envio sensível é funcional e obrigatório;
- o endpoint é validado como HTTPS;
- não há caminho HTTP;
- redirects estão controlados;
- logs não expõem dados;
- erros não expõem payload;
- resposta pública não reflete segredo;
- testes comprovam tudo.

## Onde isso aparece em sistemas reais

Cleartext Submission aparece em:

- autenticação OAuth;
- Basic Auth server-to-server;
- APIs legadas;
- integração com bureaus;
- pagamento;
- consulta cadastral;
- upload de documentos;
- download de arquivos;
- mensageria HTTP;
- webhooks;
- callbacks;
- proxy interno;
- geração de token;
- serviços de storage;
- comunicação com gateway corporativo.

Em Java/Spring, os pontos mais comuns são:

- services de integração;
- clients HTTP;
- classes utilitárias;
- módulos de autenticação;
- decorators de cache;
- código legado com `RestTemplate`;
- código moderno com `RestClient`;
- uso direto de `HttpClient`;
- configurações em YAML/properties;
- montagem manual de body form-urlencoded;
- tratamento de logs e exceções.

## Checklist prático

### Dados sensíveis

- Há senha?
- Há token?
- Há `client_secret`?
- Há header `Authorization`?
- Há API key?
- Há documento?
- Há dados pessoais?
- Há payload de autenticação?
- Há arquivo sensível?

### Canal

- O endpoint é HTTPS?
- A URI é absoluta?
- Há host válido?
- HTTP é rejeitado?
- URI relativa é rejeitada?
- Há fallback para HTTP?
- Redirects são seguidos?
- Redirects são revalidados?

### Código

- A validação ocorre antes do sink?
- O sink recebe a URI validada?
- URI e body estão separados?
- Segredo não vai na URL?
- Body usa `form-urlencoded` corretamente quando exigido?
- Erros não expõem payload?
- Logs não expõem segredo?

### Configuração

- Endpoints de produção usam HTTPS?
- Endpoints de homologação usam HTTPS?
- Secrets não estão versionados?
- Configuração inválida falha fechado?
- Host autorizado é controlado?
- Porta é esperada?

### Testes

- HTTP é rejeitado?
- URI relativa é rejeitada?
- URI sem host é rejeitada?
- Endpoint HTTPS válido é aceito?
- Logs não contêm segredo sintético?
- Erros não contêm credenciais?
- Redirect é tratado?
- Novo scan foi executado?

### Operação

- Certificados são monitorados?
- Expiração de certificado gera alerta?
- Há proxy corporativo?
- Há controle de egress?
- Há observabilidade sem vazamento?
- Há rotação de credenciais?
- Há política de retenção de logs?

## Resumo final

Cleartext Submission em integrações server-to-server acontece quando uma aplicação envia dados sensíveis por um canal sem proteção adequada.

Em Java/Spring, esse risco aparece principalmente em chamadas outbound com `RestClient`, `RestTemplate`, `WebClient`, `HttpClient` e SDKs internos de integração.

A principal defesa é impor HTTPS/TLS antes do envio de dados sensíveis.

Mas uma solução profissional envolve mais do que trocar `http://` por `https://`.

É preciso:

- validar URI;
- rejeitar HTTP;
- falhar fechado;
- separar endpoint e body;
- evitar segredo na URL;
- controlar redirects;
- não logar credenciais;
- tratar erros sem expor payload;
- criar testes;
- preservar contrato funcional;
- documentar risco residual.

A regra mais importante é:

> Se a integração carrega credencial ou dado sensível, o canal seguro não é opcional. Ele faz parte do contrato de segurança da aplicação.

## Perguntas de revisão

### 1. O que é Cleartext Submission?

É o envio de dados sensíveis por um canal sem criptografia ou com proteção insuficiente.

### 2. Cleartext acontece apenas com senha?

Não. Pode envolver tokens, documentos, dados pessoais, API keys, headers de autenticação, arquivos e dados de negócio sensíveis.

### 3. Base64 protege segredo?

Não. Base64 é codificação, não criptografia.

### 4. Server-to-server precisa de HTTPS?

Sim. Integrações entre servidores também podem transportar credenciais e dados sensíveis.

### 5. HTTPS resolve tudo?

Não. HTTPS protege o canal, mas ainda é preciso evitar logs sensíveis, erros com payload, redirects inseguros e endpoints não autorizados.

### 6. Por que validar a URI perto do sink?

Porque isso garante que a URI enviada ao cliente HTTP é a mesma que passou pela validação de segurança.

### 7. Por que não colocar segredo na URL?

Porque URLs aparecem facilmente em logs, proxies, métricas, erros e ferramentas de observabilidade.

### 8. O que significa falhar fechado?

Significa rejeitar a operação quando o endpoint não atende aos requisitos de segurança, sem fazer fallback inseguro.

### 9. Redirect pode causar Cleartext?

Sim. Um endpoint HTTPS pode redirecionar para HTTP se o cliente seguir redirects automaticamente.

### 10. O que fazer se o protocolo exige `client_secret`?

Enviar o segredo pelo canal correto, normalmente HTTPS, sem logar, sem expor em erros e sem permitir endpoint inseguro.

### 11. Quando usar mTLS?

Quando a integração exige autenticação forte entre cliente e servidor, especialmente em ambientes críticos ou regulados.

### 12. Quando um finding pode ser risco mitigado?

Quando o fluxo sensível é funcional e obrigatório, mas há HTTPS obrigatório, logs seguros, erros controlados, redirects tratados e testes comprovando o comportamento.

### 13. Qual é a diferença entre mascarar para log e mascarar para envio?

Mascarar para log é correto. Mascarar antes de enviar pode quebrar o protocolo, porque o parceiro precisa receber o valor real.

### 14. Qual é a relação entre Cleartext e SSRF?

Cleartext foca no canal inseguro. SSRF foca no controle indevido do destino. Os dois podem aparecer juntos em chamadas outbound.

### 15. Qual é a principal lição?

Credenciais podem precisar sair da aplicação por requisito funcional, mas nunca devem sair por canal inseguro ou sem controles claros.

## Referências

- [CWE-319 — Cleartext Transmission of Sensitive Information](https://cwe.mitre.org/data/definitions/319)
- [OWASP Transport Layer Security Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Transport_Layer_Security_Cheat_Sheet.html)
- [OWASP Web Security Testing Guide — Sensitive Information Sent via Unencrypted Channels](https://owasp.github.io/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/09-Weak_Cryptography/03-Sensitive_Information_Sent_via_Unencrypted_Channels)
- [OWASP Web Service Security Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Web_Service_Security_Cheat_Sheet.html)
- [OWASP Logging Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Logging_Cheat_Sheet.html)
- [Spring Framework Reference — REST Clients](https://docs.spring.io/spring-framework/reference/integration/rest-clients.html)
- [Spring Boot Reference — Calling REST Services](https://docs.spring.io/spring-boot/reference/io/rest-client.html)
- [Oracle Java SE 25 — HttpClient](https://docs.oracle.com/en/java/javase/25/docs/api/java.net.http/java/net/http/HttpClient.html)
- [Oracle Java SE 25 Documentation](https://docs.oracle.com/en/java/javase/25/)
- [Oracle Java Downloads — JDK 25 LTS](https://www.oracle.com/java/technologies/downloads/)