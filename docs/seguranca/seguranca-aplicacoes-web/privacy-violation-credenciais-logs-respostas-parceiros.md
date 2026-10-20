# Privacy Violation: credenciais, logs e respostas de parceiros

`Privacy Violation` é um tipo de finding comum em ferramentas SAST quando a aplicação manipula dados sensíveis e esses dados parecem sair de uma fronteira segura.

Essa saída pode acontecer de várias formas:

- envio para serviço externo;
- retorno em resposta HTTP;
- escrita em log;
- armazenamento em cache;
- propagação para outro sistema;
- inclusão em exceção;
- exposição em mensagem de erro;
- serialização de objeto com campos sensíveis;
- resposta de parceiro refletindo dados enviados;
- uso de credenciais em fluxos de autenticação.

Em aplicações Java/Spring, esse tipo de finding aparece muito em integrações server-to-server, especialmente quando o backend lida com:

- `client_id`;
- `client_secret`;
- senha;
- token;
- API key;
- header `Authorization`;
- documentos;
- dados pessoais;
- identificadores de cliente;
- payloads de autenticação;
- respostas de parceiros.

O ponto mais importante é entender que nem todo fluxo de dado sensível é automaticamente vulnerabilidade.

Muitas vezes, a aplicação precisa enviar uma credencial para um parceiro por requisito funcional. Por exemplo, no fluxo OAuth `client_credentials`, o `client_id` e o `client_secret` precisam ser enviados ao servidor de autorização.

A pergunta correta não é apenas:

> Existe dado sensível saindo da aplicação?

A pergunta correta é:

> Esse dado sensível está saindo para o destino correto, pelo canal correto, pelo motivo correto, sem ser exposto em logs, erros, respostas públicas ou locais indevidos?

Este artigo explica como interpretar e tratar `Privacy Violation` em aplicações Java/Spring, com foco em credenciais, logs e respostas de parceiros.

## O problema real que Privacy Violation resolve

Aplicações corporativas manipulam muitos dados sensíveis.

Mesmo uma aplicação fictícia como `meucatalogomusical.com`, um catálogo musical de bandas de Metal e Hard Rock, pode lidar com:

- e-mail do usuário;
- senha;
- token de sessão;
- token de API;
- credencial de parceiro;
- identificador interno;
- dados de assinatura;
- preferência privada;
- histórico de acesso;
- logs de operação;
- payloads de integração.

Imagine que `meucatalogomusical.com` tenha uma integração com um parceiro chamado `BackstagePartner`, responsável por liberar conteúdos premium de bandas como **Alice Cooper**, **Mötley Crüe** e **Ratt**.

Para obter um token, a aplicação envia:

```text
client_id=meucatalogo
client_secret=segredo-super-confidencial
grant_type=client_credentials
```

Esse envio pode ser totalmente legítimo.

Mas se o mesmo `client_secret` aparecer em um log, em uma resposta HTTP, em uma mensagem de erro ou em uma resposta refletida pelo parceiro, temos um problema de privacidade e exposição de dados sensíveis.

Privacy Violation ajuda a detectar esse tipo de risco.

## Antes de entrar nos detalhes: uma ideia simples

Pense em três fronteiras:

```text
Cliente -> meucatalogomusical.com -> parceiro externo
```

A aplicação recebe uma requisição de um cliente, chama um parceiro e devolve uma resposta.

Durante esse fluxo, dados sensíveis podem aparecer em vários pontos:

```text
Header de entrada
    -> DTO
    -> service
    -> client HTTP
    -> parceiro
    -> resposta do parceiro
    -> logs
    -> cache
    -> resposta pública
```

O risco de Privacy Violation aparece quando um dado sensível atravessa uma fronteira indevida.

Exemplos:

```text
client_secret -> log
```

```text
senha -> response body público
```

```text
Authorization header -> mensagem de erro
```

```text
token -> stack trace
```

```text
credencial enviada ao parceiro -> resposta do parceiro -> controller
```

A correção não é simplesmente remover todo dado sensível do sistema. Isso quebraria muitas integrações.

A correção é controlar onde o dado pode circular.

## Definição técnica

Privacy Violation, no contexto de SAST, é um finding em que a ferramenta identifica um possível fluxo de dado sensível para um destino onde esse dado não deveria aparecer.

Em termos de análise de fluxo:

```text
source: dado sensível
    -> transformações intermediárias
sink: saída, log, resposta, arquivo, sistema externo ou outro destino sensível
```

Exemplos de source:

- senha;
- token;
- `client_secret`;
- header `Authorization`;
- API key;
- CPF/CNPJ;
- e-mail;
- documento;
- payload de autenticação;
- credencial decodificada;
- resposta de parceiro contendo segredo;
- objeto de domínio com dados pessoais.

Exemplos de sink:

- log;
- resposta HTTP;
- exceção;
- arquivo;
- cache externo;
- banco de dados;
- chamada outbound;
- métrica;
- tracing;
- evento de auditoria;
- payload enviado a outro sistema.

O finding precisa ser analisado no contexto. Às vezes é vulnerabilidade real. Às vezes é risco mitigado. Às vezes é falso positivo.

## Privacy Violation não é sempre erro de código

Este é um ponto importante.

Uma ferramenta SAST pode apontar Privacy Violation em um fluxo funcional legítimo.

Exemplo:

```text
client_secret -> token endpoint OAuth
```

Esse fluxo é esperado no grant `client_credentials`.

O problema não é o envio em si.

O problema seria:

- enviar por HTTP;
- enviar para endpoint não autorizado;
- registrar em log;
- devolver para o cliente;
- incluir em exception;
- permitir que o parceiro reflita na resposta;
- armazenar sem necessidade;
- propagar para sistemas sem finalidade;
- manter em cache sem controle.

Portanto, a análise deve separar:

```text
Fluxo sensível funcional
```

de:

```text
Exposição indevida de dado sensível
```

Essa diferença é essencial para não quebrar integrações reais apenas para satisfazer o scanner.

## Exemplo inseguro: credencial em log

Imagine um serviço de autenticação com parceiro para liberar conteúdo exclusivo da banda **Quiet Riot**.

Código inseguro:

```java
package com.meucatalogomusical.parceiros.application;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Service;

@Service
public class PartnerTokenService {

    private static final Logger log = LoggerFactory.getLogger(PartnerTokenService.class);

    public void solicitarToken(String clientId, String clientSecret) {
        log.info("Solicitando token com clientId={} e clientSecret={}", clientId, clientSecret);

        // chamada ao parceiro
    }
}
```

Esse log é perigoso.

O `clientSecret` passa a existir em:

- arquivo de log;
- plataforma de observabilidade;
- agregador de logs;
- backup;
- ferramenta de busca;
- exportações;
- alertas;
- dashboards;
- possíveis tickets de suporte.

Mesmo que o canal com o parceiro seja HTTPS, o segredo foi exposto em outro lugar.

## Correção: logar evento, não segredo

Uma correção melhor é registrar o evento operacional sem incluir o segredo.

```java
package com.meucatalogomusical.parceiros.application;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Service;

@Service
public class PartnerTokenService {

    private static final Logger log = LoggerFactory.getLogger(PartnerTokenService.class);

    public void solicitarToken(String clientId, String clientSecret) {
        log.info("Solicitando token ao parceiro. clientId={}", mascararClientId(clientId));

        // chamada ao parceiro usando clientSecret, mas sem registrar o segredo
    }

    private String mascararClientId(String clientId) {
        if (clientId == null || clientId.length() <= 3) {
            return "***";
        }

        return clientId.substring(0, 3) + "***";
    }
}
```

Aqui o log continua útil:

```text
Solicitando token ao parceiro. clientId=meu***
```

Mas o segredo não é registrado.

Em muitos casos, até o `clientId` pode ser sensível dependendo do ambiente. Por isso, mascarar ou substituir por identificador interno pode ser melhor.

## O que não deve ir para logs

Logs são fundamentais para operação, auditoria e diagnóstico, mas não devem receber qualquer dado.

Evite registrar:

- senhas;
- tokens;
- `client_secret`;
- header `Authorization`;
- cookies;
- refresh tokens;
- API keys;
- documentos pessoais;
- payload completo de autenticação;
- body completo de resposta de token;
- query string com dados sensíveis;
- stack trace contendo segredo;
- exceções de bibliotecas que incluam headers;
- objetos inteiros com `toString()` inseguro.

Um erro comum é fazer:

```java
log.info("Request recebido: {}", request);
```

Se `request` contém credenciais, isso é perigoso.

Outro erro comum:

```java
log.error("Erro ao chamar parceiro", exception);
```

Isso pode ser aceitável em alguns contextos, mas precisa ser avaliado. Algumas exceções carregam mensagens com URL, headers, body ou detalhes sensíveis.

## Exemplo inseguro: exception com segredo

```java
package com.meucatalogomusical.parceiros.application;

public class PartnerAuthException extends RuntimeException {

    public PartnerAuthException(String clientId, String clientSecret) {
        super("Falha ao autenticar parceiro com clientId=" + clientId
                + " e clientSecret=" + clientSecret);
    }
}
```

Essa exceção é insegura porque o segredo entra na mensagem.

A mensagem pode aparecer em:

- log;
- resposta HTTP;
- APM;
- tracing;
- alertas;
- ticket;
- tela administrativa.

## Correção: mensagem segura

```java
package com.meucatalogomusical.parceiros.application;

public class PartnerAuthException extends RuntimeException {

    public PartnerAuthException() {
        super("Falha ao autenticar com parceiro.");
    }
}
```

A mensagem pública informa o erro sem expor dados.

Se for necessário diagnosticar, use correlation id, nome lógico do parceiro e código de erro, não o segredo.

Exemplo:

```java
log.warn("Falha ao autenticar com parceiro. parceiro={}, correlationId={}",
        parceiro, correlationId);
```

## Privacy Violation em respostas de parceiros

Um caso mais sutil acontece quando a aplicação envia uma credencial ao parceiro e depois devolve a resposta do parceiro para o cliente.

Fluxo:

```text
meucatalogomusical.com
    -> envia client_id/client_secret ao parceiro
    -> parceiro responde JSON
    -> aplicação devolve JSON ao consumidor
```

Se o parceiro responder corretamente:

```json
{
  "access_token": "token-gerado",
  "token_type": "Bearer",
  "expires_in": 3600
}
```

talvez o contrato funcione.

Mas imagine que, por erro do parceiro, ele devolva:

```json
{
  "client_id": "meucatalogo",
  "client_secret": "segredo-super-confidencial",
  "access_token": "token-gerado"
}
```

Se a aplicação repassar essa resposta sem filtro, ela pode expor o segredo.

Esse é o tipo de situação em que uma ferramenta SAST pode apontar Privacy Violation.

Mesmo que o envio da credencial ao parceiro seja legítimo, devolver a credencial para o consumidor pode ser uma vulnerabilidade.

## Controle: bloquear reflexão de credenciais

Uma forma de tratar isso é criar uma guarda que rejeita respostas de parceiro contendo segredo.

Exemplo didático:

```java
package com.meucatalogomusical.parceiros.infra;

import com.fasterxml.jackson.databind.JsonNode;
import org.springframework.stereotype.Component;

import java.util.Iterator;
import java.util.Map;

@Component
public class CredentialReflectionGuard {

    public void rejectCredentialReflection(
            JsonNode response,
            PartnerCredentials credentials
    ) {
        if (response == null) {
            return;
        }

        verify(response, credentials);
    }

    private void verify(JsonNode node, PartnerCredentials credentials) {
        if (node.isTextual()) {
            String value = node.asText();

            if (value.equals(credentials.clientSecret())) {
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

                verify(field.getValue(), credentials);
            }

            return;
        }

        if (node.isArray()) {
            for (JsonNode item : node) {
                verify(item, credentials);
            }
        }
    }

    private boolean isSecretField(String fieldName) {
        String normalized = fieldName.toLowerCase();

        return normalized.contains("password")
                || normalized.contains("secret")
                || normalized.contains("credential")
                || normalized.contains("authorization")
                || normalized.contains("token");
    }
}
```

DTO de credenciais:

```java
package com.meucatalogomusical.parceiros.infra;

public record PartnerCredentials(
        String clientId,
        String clientSecret
) {
}
```

Esse exemplo rejeita:

- campos com nomes sensíveis;
- valor textual igual ao `clientSecret`.

Em um sistema real, a regra precisa ser ajustada com cuidado. Por exemplo, rejeitar qualquer campo `token` pode ser inadequado se a resposta esperada é justamente um token. Nesse caso, a guarda deve distinguir campos permitidos de campos proibidos conforme o contrato do parceiro.

O ponto principal é: a resposta do parceiro também precisa ser tratada como dado não confiável.

## Cópia segura da resposta pública

Outra prática defensiva é construir uma resposta pública explícita em vez de devolver o objeto inteiro do parceiro.

Exemplo de resposta interna do parceiro:

```java
package com.meucatalogomusical.parceiros.infra;

public record PartnerTokenResponse(
        String accessToken,
        String tokenType,
        Long expiresIn,
        String clientSecret
) {
}
```

Esse DTO contém um campo que não deveria ir para fora.

Resposta pública segura:

```java
package com.meucatalogomusical.parceiros.api;

public record PublicTokenResponse(
        String tokenType,
        Long expiresIn
) {
}
```

Mapper:

```java
package com.meucatalogomusical.parceiros.application;

import com.meucatalogomusical.parceiros.api.PublicTokenResponse;
import com.meucatalogomusical.parceiros.infra.PartnerTokenResponse;
import org.springframework.stereotype.Component;

@Component
public class PartnerTokenResponseMapper {

    public PublicTokenResponse toPublicResponse(PartnerTokenResponse partnerResponse) {
        return new PublicTokenResponse(
                partnerResponse.tokenType(),
                partnerResponse.expiresIn()
        );
    }
}
```

Essa abordagem evita depender do cliente para ignorar campos sensíveis.

A API deve devolver apenas o que o consumidor precisa.

## Cuidado com serialização automática

Um erro comum em APIs Java/Spring é retornar entidades, objetos internos ou DTOs de parceiros diretamente.

Exemplo perigoso:

```java
@GetMapping("/token")
public PartnerTokenResponse token() {
    return partnerTokenService.solicitarToken();
}
```

Se `PartnerTokenResponse` contém `accessToken`, `refreshToken`, `clientSecret` ou dados internos, tudo pode ser serializado.

Prefira criar DTOs públicos explícitos.

```java
@GetMapping("/token")
public PublicTokenResponse token() {
    PartnerTokenResponse partnerResponse = partnerTokenService.solicitarToken();
    return mapper.toPublicResponse(partnerResponse);
}
```

Essa decisão reduz risco de exposição acidental.

## Exemplo completo: fluxo seguro com parceiro

Imagine que `meucatalogomusical.com` precisa consultar um parceiro para liberar conteúdo premium da banda **W.A.S.P.**.

### Configuração

```yaml
integracoes:
  backstage:
    token-url: https://auth.backstagepartner.example/oauth/token
    client-id: meucatalogo
    client-secret: ${BACKSTAGE_CLIENT_SECRET}
```

### Validador HTTPS

```java
package com.meucatalogomusical.parceiros.infra;

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

### Client HTTP seguro

```java
package com.meucatalogomusical.parceiros.infra;

import com.fasterxml.jackson.databind.JsonNode;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.http.MediaType;
import org.springframework.stereotype.Component;
import org.springframework.util.LinkedMultiValueMap;
import org.springframework.util.MultiValueMap;
import org.springframework.web.client.RestClient;

import java.net.URI;

@Component
public class BackstagePartnerClient {

    private final RestClient restClient;
    private final HttpsEndpointValidator endpointValidator;
    private final CredentialReflectionGuard credentialReflectionGuard;
    private final URI tokenEndpoint;
    private final PartnerCredentials credentials;

    public BackstagePartnerClient(
            RestClient.Builder restClientBuilder,
            HttpsEndpointValidator endpointValidator,
            CredentialReflectionGuard credentialReflectionGuard,
            @Value("${integracoes.backstage.token-url}") URI tokenEndpoint,
            @Value("${integracoes.backstage.client-id}") String clientId,
            @Value("${integracoes.backstage.client-secret}") String clientSecret
    ) {
        this.restClient = restClientBuilder.build();
        this.endpointValidator = endpointValidator;
        this.credentialReflectionGuard = credentialReflectionGuard;
        this.tokenEndpoint = tokenEndpoint;
        this.credentials = new PartnerCredentials(clientId, clientSecret);
    }

    public JsonNode solicitarToken() {
        URI endpointValidado = endpointValidator.requireHttps(tokenEndpoint);

        MultiValueMap<String, String> body = new LinkedMultiValueMap<>();
        body.add("client_id", credentials.clientId());
        body.add("client_secret", credentials.clientSecret());
        body.add("grant_type", "client_credentials");

        JsonNode response = restClient
                .post()
                .uri(endpointValidado)
                .contentType(MediaType.APPLICATION_FORM_URLENCODED)
                .body(body)
                .retrieve()
                .body(JsonNode.class);

        credentialReflectionGuard.rejectCredentialReflection(response, credentials);

        return response;
    }
}
```

Esse código demonstra:

- endpoint validado como HTTPS;
- credenciais enviadas somente ao parceiro;
- body separado da URI;
- resposta tratada como não confiável;
- bloqueio de reflexão de segredo;
- ausência de logs com credenciais.

## Logs operacionais seguros

Um log útil não precisa conter segredo.

```java
package com.meucatalogomusical.parceiros.application;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Service;

@Service
public class ConteudoPremiumService {

    private static final Logger log = LoggerFactory.getLogger(ConteudoPremiumService.class);

    private final BackstageTokenService backstageTokenService;

    public ConteudoPremiumService(BackstageTokenService backstageTokenService) {
        this.backstageTokenService = backstageTokenService;
    }

    public void liberarConteudoPremium(String banda) {
        log.info("Iniciando liberação de conteúdo premium. banda={}", banda);

        backstageTokenService.obterToken();

        log.info("Conteúdo premium liberado com sucesso. banda={}", banda);
    }
}
```

Esse log usa dado de negócio não sensível, como o nome da banda.

Exemplo:

```text
Iniciando liberação de conteúdo premium. banda=W.A.S.P.
```

Não registra:

- `client_secret`;
- token;
- header;
- body;
- resposta do parceiro.

## Privacy Violation em cache

Outro ponto comum é o cache.

Imagine que a aplicação guarda tokens por alguns segundos para evitar chamadas repetidas ao parceiro.

Código perigoso:

```java
cache.put(clientSecret, tokenResponse);
```

Aqui o segredo vira chave do cache.

Mesmo que o cache esteja em memória, isso pode ser ruim:

- o segredo fica mais tempo em memória;
- dumps podem expor;
- logs do cache podem registrar chave;
- métricas podem coletar chave;
- debug pode mostrar chave;
- a estrutura pode ser migrada futuramente para Redis ou outro storage.

Uma alternativa melhor é derivar uma chave segura, sem armazenar o segredo diretamente.

## Chave HMAC para cache

Exemplo didático:

```java
package com.meucatalogomusical.parceiros.infra;

import javax.crypto.Mac;
import javax.crypto.spec.SecretKeySpec;
import java.nio.charset.StandardCharsets;
import java.util.Base64;

public class CacheKeyGenerator {

    private final byte[] pepper;

    public CacheKeyGenerator(byte[] pepper) {
        this.pepper = pepper.clone();
    }

    public String generate(String clientId, String clientSecret) {
        try {
            Mac mac = Mac.getInstance("HmacSHA256");
            mac.init(new SecretKeySpec(pepper, "HmacSHA256"));

            String material = clientId + ":" + clientSecret;
            byte[] digest = mac.doFinal(material.getBytes(StandardCharsets.UTF_8));

            return Base64.getUrlEncoder().withoutPadding().encodeToString(digest);
        } catch (Exception ex) {
            throw new IllegalStateException("Não foi possível gerar chave de cache.", ex);
        }
    }
}
```

Esse exemplo evita usar o segredo diretamente como chave.

Atenção: HMAC não elimina a existência do segredo no fluxo. Ele apenas evita que o segredo seja usado como identificador visível no cache.

O segredo ainda deve ser protegido.

## Minimização de dados

Um princípio essencial em Privacy Violation é minimização.

A aplicação deve carregar, armazenar, logar e devolver apenas o necessário.

Exemplo ruim:

```java
public record UsuarioResponse(
        Long id,
        String nome,
        String email,
        String passwordHash,
        String documento,
        String internalRole,
        String tokenInterno
) {
}
```

Exemplo melhor:

```java
public record UsuarioPublicoResponse(
        Long id,
        String nome
) {
}
```

A pergunta prática é:

> O consumidor realmente precisa deste campo?

Se não precisa, não devolva.

## Privacy Violation e APIs

APIs frequentemente expõem dados demais quando serializam objetos internos diretamente.

Exemplo perigoso:

```java
@GetMapping("/usuarios/{id}")
public UsuarioEntity buscar(@PathVariable Long id) {
    return usuarioRepository.findById(id).orElseThrow();
}
```

Se `UsuarioEntity` contém campos internos, eles podem ir para a resposta.

Melhor:

```java
@GetMapping("/usuarios/{id}")
public UsuarioPublicoResponse buscar(@PathVariable Long id) {
    Usuario usuario = usuarioService.buscar(id);

    return new UsuarioPublicoResponse(
            usuario.id(),
            usuario.nomeExibicao()
    );
}
```

DTO público explícito é uma defesa importante contra exposição acidental.

## Privacy Violation e `toString()`

Muitas classes Java usam Lombok ou records, que geram representação textual automaticamente.

Exemplo perigoso:

```java
public record PartnerCredentials(
        String clientId,
        String clientSecret
) {
}
```

Se alguém fizer:

```java
log.info("Credenciais: {}", credentials);
```

o `record` pode imprimir ambos os campos.

Uma alternativa é sobrescrever `toString()`.

```java
package com.meucatalogomusical.parceiros.infra;

public record PartnerCredentials(
        String clientId,
        String clientSecret
) {

    @Override
    public String toString() {
        return "PartnerCredentials[clientId=" + mascarar(clientId) + ", clientSecret=***]";
    }

    private String mascarar(String value) {
        if (value == null || value.length() <= 3) {
            return "***";
        }

        return value.substring(0, 3) + "***";
    }
}
```

Isso não autoriza logar credenciais, mas reduz dano acidental.

A melhor prática continua sendo não passar o objeto sensível para log.

## Privacy Violation e tracing

Além de logs tradicionais, aplicações modernas usam:

- tracing;
- APM;
- métricas;
- spans;
- tags;
- baggage;
- correlation id;
- dashboards.

Esses mecanismos também podem vazar dados.

Evite colocar em tags de tracing:

- token;
- email completo;
- documento;
- `client_secret`;
- senha;
- body;
- payload de autenticação;
- query string sensível.

Um `correlationId` deve ser opaco e não sensível.

Exemplo seguro:

```text
correlationId=8f0b6f52-8f4d-4c0f-a725-5f4f9e9b2a7a
```

Exemplo ruim:

```text
correlationId=email-do-usuario@example.com
```

## Tratando erros de parceiro

Quando um parceiro falha, a aplicação precisa responder sem expor detalhes sensíveis.

Exemplo ruim:

```java
catch (Exception ex) {
    throw new IllegalStateException("Erro ao chamar parceiro com body: " + body, ex);
}
```

Exemplo melhor:

```java
catch (Exception ex) {
    log.warn("Falha ao chamar parceiro. tipoErro={}", ex.getClass().getSimpleName());
    throw new BadGatewayException("Falha ao consultar parceiro.");
}
```

Exceção pública:

```java
package com.meucatalogomusical.shared.api;

public class BadGatewayException extends RuntimeException {

    public BadGatewayException(String message) {
        super(message);
    }
}
```

A mensagem pública não contém segredo.

O log não contém body.

A operação ainda é diagnosticável pelo tipo de erro e correlation id.

## Testando ausência de segredo nos logs

Em um projeto real, você pode criar testes que usam marcadores sintéticos.

Exemplo conceitual:

```java
package com.meucatalogomusical.parceiros.application;

import org.junit.jupiter.api.Test;

import static org.junit.jupiter.api.Assertions.assertFalse;

class LogPrivacyTest {

    @Test
    void mensagemDeLogNaoDeveConterSecret() {
        String clientSecret = "SECRET-TESTE-MOTLEY-CRUE";

        String mensagem = "Falha ao chamar parceiro. tipoErro=TimeoutException";

        assertFalse(mensagem.contains(clientSecret));
    }
}
```

Esse teste é simples demais para um sistema real, mas ilustra a ideia.

Em um teste mais robusto, você capturaria o appender de log e validaria que nenhum marcador sensível apareceu após executar o fluxo completo.

Marcadores úteis:

```text
SECRET-TESTE-MOTLEY-CRUE
TOKEN-TESTE-ALICE-COOPER
AUTHORIZATION-TESTE-RATT
PASSWORD-TESTE-QUIET-RIOT
```

Nenhum deles deve aparecer em logs.

## Testando bloqueio de reflexão

Exemplo de teste para `CredentialReflectionGuard`.

```java
package com.meucatalogomusical.parceiros.infra;

import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.junit.jupiter.api.Test;

import static org.junit.jupiter.api.Assertions.assertThrows;

class CredentialReflectionGuardTest {

    private final ObjectMapper objectMapper = new ObjectMapper();
    private final CredentialReflectionGuard guard = new CredentialReflectionGuard();

    @Test
    void deveRejeitarRespostaComClientSecretRefletido() throws Exception {
        PartnerCredentials credentials = new PartnerCredentials(
                "meucatalogo",
                "SECRET-TESTE-WASP"
        );

        JsonNode response = objectMapper.readTree("""
                {
                  "access_token": "token-valido",
                  "client_secret": "SECRET-TESTE-WASP"
                }
                """);

        assertThrows(IllegalStateException.class,
                () -> guard.rejectCredentialReflection(response, credentials));
    }

    @Test
    void deveRejeitarCampoComNomeSensivel() throws Exception {
        PartnerCredentials credentials = new PartnerCredentials(
                "meucatalogo",
                "SECRET-TESTE-WASP"
        );

        JsonNode response = objectMapper.readTree("""
                {
                  "access_token": "token-valido",
                  "credential": "algum-valor"
                }
                """);

        assertThrows(IllegalStateException.class,
                () -> guard.rejectCredentialReflection(response, credentials));
    }
}
```

Esses testes provam que uma resposta insegura do parceiro falha fechado.

## Quando um Privacy Violation é vulnerabilidade real?

É vulnerabilidade real quando:

- segredo aparece em log;
- token é devolvido ao cliente errado;
- senha aparece em mensagem de erro;
- `client_secret` é retornado em resposta pública;
- DTO interno é serializado com campos sensíveis;
- dado pessoal é enviado a terceiro sem finalidade;
- payload sensível é registrado por tracing;
- cache expõe chave baseada em segredo puro;
- arquivo gerado contém dados sensíveis sem proteção;
- resposta de parceiro é repassada sem filtro e contém segredo.

Nesses casos, a correção deve ser no código, no contrato ou na arquitetura.

## Quando é risco mitigado?

É risco mitigado quando:

- o dado sensível precisa circular por requisito funcional;
- o destino é correto;
- o canal é seguro;
- logs não expõem;
- erros não expõem;
- resposta pública é filtrada;
- reflexão de segredo é bloqueada;
- cache usa chave derivada segura;
- há testes;
- há documentação de risco residual.

Exemplo:

```text
client_secret -> token endpoint OAuth HTTPS
```

Esse fluxo pode ser aceitável se controlado.

## Quando pode ser falso positivo?

Pode ser falso positivo quando:

- a ferramenta associa argumento sensível ao retorno de uma chamada, mas o código filtra a resposta;
- o segredo é usado apenas para gerar HMAC, não vai para log;
- a chamada de log não recebe o valor sensível;
- o DTO sensível não é serializado publicamente;
- o caminho apontado é inalcançável;
- o scanner não reconhece uma guarda customizada;
- o dado sensível é substituído por identificador opaco antes do sink.

Mesmo quando for falso positivo, vale perguntar:

> O código está claro o suficiente para humanos e ferramentas entenderem o controle?

Se não estiver, pode valer aplicar hardening.

## Como escrever justificativa técnica

Justificativa fraca:

```text
Não é vulnerabilidade porque é interno.
```

Justificativa forte:

```text
O finding aponta fluxo de client_secret para resposta pública. O envio de client_id/client_secret ao token endpoint é funcional e obrigatório no fluxo OAuth client_credentials. O endpoint é validado como HTTPS absoluto antes do envio. O body não é registrado em logs e exceções públicas não incluem credenciais. A resposta do parceiro passa por CredentialReflectionGuard, que rejeita campos secretos e valores iguais ao clientSecret antes de qualquer resposta pública. A resposta pública é construída por DTO explícito, sem receber PartnerCredentials. Testes cobrem rejeição de segredo refletido e ausência de marcadores sensíveis em logs. Classificação proposta: risco mitigado ou Not Exploitable.
```

A segunda justificativa explica:

- source;
- sink;
- fluxo funcional;
- controles;
- testes;
- classificação.

## Boas práticas

### 1. Não logue segredos

Nunca registre senha, token, `client_secret`, API key ou header `Authorization`.

### 2. Use DTOs públicos explícitos

Não devolva entidades ou DTOs internos diretamente.

### 3. Trate resposta de parceiro como não confiável

Parceiros também podem errar, refletir dados ou mudar contrato.

### 4. Bloqueie reflexão de credenciais

Se você envia segredo, impeça que ele volte para a resposta pública.

### 5. Separe dado funcional de dado público

O que é necessário para autenticar não é necessariamente permitido na resposta.

### 6. Proteja logs e tracing

Logs, spans, tags e métricas também são superfícies de vazamento.

### 7. Use correlation id opaco

Não use email, documento ou token como identificador de correlação.

### 8. Não use segredo puro como chave de cache

Use derivação segura quando precisar identificar combinações sensíveis.

### 9. Falhe fechado

Se uma resposta contém campo secreto inesperado, rejeite.

### 10. Crie testes com marcadores sensíveis

Use valores sintéticos e prove que eles não aparecem onde não deveriam.

## Erros comuns

### 1. Logar objeto inteiro

`log.info("request={}", request)` pode expor campos sensíveis.

### 2. Confiar no `toString()`

Records, Lombok e IDEs podem gerar `toString()` com todos os campos.

### 3. Devolver resposta bruta do parceiro

O parceiro pode incluir campos que você não deveria expor.

### 4. Confundir envio funcional com vazamento

Enviar `client_secret` ao OAuth pode ser legítimo. Vazamento é expor esse segredo fora do destino correto.

### 5. Colocar segredo em exception

Mensagens de erro circulam por logs, APIs e observabilidade.

### 6. Usar token como correlation id

Correlation id deve ser opaco e não sensível.

### 7. Guardar segredo como chave de cache

Chaves de cache podem aparecer em logs, métricas ou dumps.

### 8. Achar que HTTPS resolve Privacy Violation

HTTPS protege transmissão. Não protege logs, respostas, cache ou serialização indevida.

### 9. Criar sanitização artificial para SAST

A correção deve reduzir risco real, não apenas quebrar o taint flow.

### 10. Não documentar fluxo funcional autorizado

Se a credencial precisa sair por requisito, documente destino, canal, controles e testes.

## Decisões de engenharia

## Quando devolver resposta bruta do parceiro?

Evite devolver resposta bruta.

Só considere quando:

- o contrato legado exige;
- o consumidor é confiável;
- a resposta é validada;
- campos sensíveis são rejeitados;
- há testes;
- há documentação de risco.

Mesmo assim, prefira mapear para DTO público.

## Quando usar guarda de reflexão?

Use quando:

- você envia credenciais ao parceiro;
- a resposta do parceiro pode ser repassada;
- o contrato do parceiro não é totalmente controlado;
- há risco de eco de campos sensíveis;
- o SAST aponta fluxo de credencial para resposta.

## Quando mascarar?

Mascaramento é útil para:

- logs;
- mensagens operacionais;
- dashboards;
- telas administrativas;
- suporte.

Mas não masque antes de enviar para o protocolo funcional se o parceiro precisa do valor real.

## Quando classificar como Not Exploitable?

Considere quando:

- o fluxo sensível é funcional;
- o destino é correto;
- há HTTPS;
- não há log de segredo;
- resposta é filtrada;
- reflexão é bloqueada;
- DTO público não contém credencial;
- testes comprovam;
- risco residual está documentado.

## Quando envolver AppSec?

Envolva AppSec quando:

- o scanner não reconhece guarda customizada;
- o finding persiste após correção real;
- a classificação será `Not Exploitable`;
- há necessidade de ajustar query corporativa;
- o risco residual depende de política corporativa;
- a decisão envolve exceção de segurança.

## Onde isso aparece em sistemas reais

Privacy Violation aparece em:

- integração OAuth;
- cache de token;
- logs de erro;
- logs de debug;
- decorators de cache;
- gateways;
- APIs que retornam DTOs internos;
- respostas de parceiros;
- sistemas legados;
- integrações com bureau;
- upload/download;
- sistemas de documento;
- autenticação server-to-server;
- mensageria;
- auditoria;
- tracing e APM.

Em Java/Spring, pontos comuns são:

- controllers retornando objetos internos;
- services logando request/response;
- clients HTTP;
- interceptors;
- exception handlers;
- classes de cache;
- DTOs com `toString()`;
- serialização Jackson;
- integração com parceiros;
- filtros de correlation id.

## Checklist prático

### Dados sensíveis

- Existe senha?
- Existe token?
- Existe `client_secret`?
- Existe API key?
- Existe header `Authorization`?
- Existe documento?
- Existe dado pessoal?
- Existe payload de autenticação?

### Logs

- Algum objeto sensível é logado?
- Algum body completo é logado?
- Alguma exception pode carregar segredo?
- `toString()` expõe campos sensíveis?
- DEBUG em produção está controlado?
- Logs possuem retenção e acesso restrito?

### Respostas

- A API retorna DTO público?
- Alguma resposta contém token?
- Alguma resposta contém campo secreto?
- Resposta de parceiro é repassada diretamente?
- Existe filtro ou mapper?
- Existe bloqueio de reflexão?

### Integrações

- O destino é correto?
- O canal é HTTPS?
- O segredo é funcionalmente necessário?
- O segredo é enviado apenas ao parceiro correto?
- Erros do parceiro são sanitizados?
- Redirecionamentos são controlados?

### Cache

- Segredo é usado como chave?
- Token é armazenado?
- TTL é adequado?
- Logs do cache expõem chave?
- Há isolamento entre credenciais?
- Há limpeza ou expiração?

### Testes

- Marcadores sensíveis aparecem em logs?
- Resposta com segredo refletido é rejeitada?
- DTO público exclui campos sensíveis?
- Exception pública não contém segredo?
- Fluxo funcional continua funcionando?
- Novo scan foi executado?

## Resumo final

`Privacy Violation` é um finding que exige análise cuidadosa.

Ele pode indicar uma vulnerabilidade real, como segredo em log, token em resposta pública ou senha em exception.

Mas também pode aparecer em fluxos funcionais legítimos, como envio de `client_id` e `client_secret` para um token endpoint OAuth.

A diferença está nos controles.

Uma aplicação Java/Spring bem protegida deve:

- limitar onde dados sensíveis circulam;
- não registrar segredos;
- usar DTOs públicos explícitos;
- tratar respostas de parceiros como não confiáveis;
- bloquear reflexão de credenciais;
- proteger mensagens de erro;
- evitar segredo em chave de cache;
- usar correlation id opaco;
- criar testes com marcadores sensíveis;
- documentar triagens e riscos residuais.

A principal lição é:

> Dado sensível pode ser necessário para a integração, mas nunca deve aparecer fora da finalidade, do destino e do contexto corretos.

## Perguntas de revisão

### 1. O que é Privacy Violation?

É um finding que indica possível exposição ou propagação indevida de dados sensíveis para logs, respostas, arquivos, sistemas externos, exceções ou outros destinos.

### 2. Todo fluxo de credencial é vulnerabilidade?

Não. Alguns fluxos são funcionais e obrigatórios, como OAuth `client_credentials`. O importante é controlar destino, canal, logs, erros e resposta pública.

### 3. Por que logs são perigosos?

Porque logs podem ser armazenados, replicados, pesquisados, exportados, retidos por muito tempo e acessados por várias pessoas ou ferramentas.

### 4. O que não deve ir para logs?

Senhas, tokens, `client_secret`, headers de autenticação, cookies, documentos, payloads completos de autenticação e respostas sensíveis.

### 5. HTTPS resolve Privacy Violation?

Não completamente. HTTPS protege transmissão, mas não impede vazamento em logs, respostas, cache ou erros.

### 6. Por que resposta de parceiro deve ser tratada como não confiável?

Porque o parceiro pode refletir dados enviados, mudar contrato, incluir campos inesperados ou retornar informações que não devem ser públicas.

### 7. O que é reflexão de credencial?

É quando uma credencial enviada ao parceiro volta na resposta e pode ser propagada pela aplicação.

### 8. O que é DTO público explícito?

É um objeto de resposta desenhado apenas com campos que podem sair da aplicação, evitando serializar objetos internos ou respostas brutas.

### 9. É correto mascarar segredo antes de enviar ao parceiro?

Não, se o parceiro precisa do valor real para autenticar. Mascaramento é para logs, telas e mensagens, não para o protocolo funcional.

### 10. Por que `toString()` pode ser perigoso?

Porque pode incluir todos os campos do objeto, inclusive senhas e tokens, quando usado em logs.

### 11. O que é correlation id seguro?

É um identificador opaco, sem dados pessoais ou segredos, usado para rastrear requisições.

### 12. Quando usar HMAC em chave de cache?

Quando é necessário derivar uma chave a partir de material sensível sem usar o segredo puro como identificador visível.

### 13. Quando um finding pode ser falso positivo?

Quando o scanner associa dado sensível a um sink, mas o fluxo real não expõe o dado, ou há controles que impedem a exposição.

### 14. O que deve ter em uma justificativa de Not Exploitable?

Source, sink, fluxo apontado, fluxo real, controles, testes, evidências e risco residual.

### 15. Qual é a principal regra?

Dado sensível deve circular apenas onde há finalidade clara, controle adequado e necessidade real.

## Referências

- [OWASP Logging Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Logging_Cheat_Sheet.html)
- [OWASP Logging Vocabulary Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Logging_Vocabulary_Cheat_Sheet.html)
- [OWASP API Security Top 10 2023](https://api-security.owasp.org/editions/2023/en/0x11-t10/)
- [OWASP WSTG — Excessive Data Exposure](https://owasp.github.io/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/12-API_Testing/03-Excessive_Data_Exposure)
- [OWASP API10:2023 — Unsafe Consumption of APIs](https://api-security.owasp.org/editions/2023/en/0xaa-unsafe-consumption-of-apis/)
- [OWASP Developer Guide — Logging and Monitoring](https://devguide.owasp.org/en/04-design/02-web-app-checklist/09-logging-monitoring/)
- [CWE-532 — Insertion of Sensitive Information into Log File](https://cwe.mitre.org/data/definitions/532.html)
- [CWE-200 — Exposure of Sensitive Information to an Unauthorized Actor](https://cwe.mitre.org/data/definitions/200.html)
- [Spring Framework Reference — REST Clients](https://docs.spring.io/spring-framework/reference/integration/rest-clients.html)
- [Spring Boot Reference — Calling REST Services](https://docs.spring.io/spring-boot/reference/io/rest-client.html)
- [Oracle JDK 25 Documentation](https://docs.oracle.com/en/java/javase/25/)
- [Oracle Java Downloads — JDK 25 LTS](https://www.oracle.com/java/technologies/downloads/)