# Headers de segurança: HSTS, JSON explícito e nosniff

Headers de segurança são cabeçalhos HTTP enviados pela aplicação para orientar o comportamento do navegador, clientes HTTP, proxies e ferramentas intermediárias.

Eles não substituem validação de entrada, autorização, autenticação, criptografia, logs seguros ou código defensivo. Mas ajudam a reduzir ambiguidade e a reforçar o contrato de segurança da aplicação.

Neste artigo, vamos focar em três temas:

- `Strict-Transport-Security`, também conhecido como **HSTS**;
- `Content-Type` explícito, especialmente `application/json`;
- `X-Content-Type-Options: nosniff`.

Esses três pontos aparecem com frequência em aplicações Java/Spring e em findings SAST, principalmente quando a ferramenta detecta:

- ausência de HSTS;
- resposta sem tipo claro;
- risco de MIME sniffing;
- possível XSS em endpoint que deveria ser apenas JSON;
- API REST que retorna dados sem contrato HTTP explícito;
- resposta interpretável como HTML por algum cliente.

A ideia central é:

> A aplicação deve deixar claro para o cliente como aquela resposta deve ser tratada.

Se a resposta é JSON, ela deve ser entregue como JSON.

Se a aplicação só deve funcionar por HTTPS, isso deve ser reforçado.

Se o servidor informou um `Content-Type`, o navegador não deve tentar “adivinhar” outro tipo.

No contexto da aplicação fictícia `meucatalogomusical.com`, imagine uma API que retorna dados de bandas como **Amon Amarth**, **Kreator** e **Testament**. Essa API não entrega HTML; ela entrega JSON. Então o contrato HTTP precisa deixar isso explícito.

## O problema real que esses headers resolvem

Muitos problemas de segurança não surgem apenas porque o código está errado.

Às vezes, o problema é ambiguidade.

Exemplos:

```text
A aplicação retorna JSON, mas não declara claramente Content-Type.
```

```text
O endpoint deveria ser consumido apenas por API, mas aceita negociação para text/html.
```

```text
O navegador tenta interpretar uma resposta de forma diferente da pretendida.
```

```text
A aplicação usa HTTPS, mas não orienta o navegador a evitar HTTP em acessos futuros.
```

```text
O scanner aponta XSS porque vê dado não confiável indo para a resposta, mas o endpoint é uma API JSON server-to-server.
```

Headers de segurança ajudam a deixar o contrato explícito.

Eles dizem:

```text
Esta resposta é JSON.
Não tente adivinhar outro MIME type.
Este domínio deve ser acessado por HTTPS.
```

Esse tipo de clareza ajuda:

- navegadores;
- clientes HTTP;
- ferramentas SAST/DAST;
- times de AppSec;
- revisores de código;
- consumidores de API;
- operadores de produção.

## Antes de entrar nos detalhes: uma ideia simples

Pense em um pacote enviado pelo correio.

Se o pacote não tem identificação clara, a pessoa que recebe pode tentar adivinhar o conteúdo.

Agora pense em uma resposta HTTP.

Se a aplicação devolve um corpo como este:

```json
{
  "banda": "Amon Amarth",
  "genero": "Melodic Death Metal"
}
```

mas não informa corretamente:

```http
Content-Type: application/json
```

o cliente pode tratar a resposta de forma incorreta.

E se o navegador tentar “adivinhar” o tipo do conteúdo, isso pode abrir espaço para comportamentos inseguros.

Por isso, uma API deve ser explícita:

```http
Content-Type: application/json
X-Content-Type-Options: nosniff
```

E uma aplicação web que exige HTTPS deve reforçar:

```http
Strict-Transport-Security: max-age=31536000; includeSubDomains
```

Cada header reduz um tipo de ambiguidade.

## Definição técnica

Headers de segurança são campos de cabeçalho HTTP usados para instruir clientes e navegadores sobre políticas de segurança da resposta.

Neste artigo, vamos tratar três:

```http
Strict-Transport-Security: max-age=31536000; includeSubDomains
```

```http
Content-Type: application/json
```

```http
X-Content-Type-Options: nosniff
```

O `Strict-Transport-Security` informa ao navegador que o host deve ser acessado apenas via HTTPS em acessos futuros. A MDN descreve esse header como uma forma de o site informar ao navegador que deve ser acessado somente por HTTPS, não por HTTP.

O `Content-Type` informa ao cliente o media type do conteúdo retornado, como `application/json`, `text/html`, `text/plain` ou `application/pdf`. A MDN descreve que, em respostas, o `Content-Type` informa ao cliente qual é o tipo de mídia do recurso devolvido.

O `X-Content-Type-Options: nosniff` instrui o navegador a respeitar o MIME type informado em `Content-Type`, evitando MIME sniffing. A MDN descreve esse header como um marcador usado pelo servidor para indicar que os MIME types anunciados em `Content-Type` não devem ser alterados ou “adivinhados”.

## Por que isso importa para segurança?

Esses headers ajudam a reduzir riscos como:

- downgrade para HTTP;
- interpretação ambígua de resposta;
- MIME sniffing;
- endpoints JSON tratados como HTML;
- findings de XSS em APIs que não produzem HTML;
- comportamento inconsistente entre navegadores;
- respostas de erro interpretadas fora do contexto esperado;
- exposição acidental por configuração HTTP incompleta.

Eles não tornam uma aplicação invulnerável.

Mas são camadas importantes de defesa.

Em segurança, uma prática recorrente é:

> Reduza ambiguidade. Explicite contratos. Falhe fechado.

Headers de segurança fazem exatamente isso no nível HTTP.

## HSTS: Strict-Transport-Security

HSTS significa HTTP Strict Transport Security.

Ele é configurado por meio do header:

```http
Strict-Transport-Security: max-age=31536000; includeSubDomains
```

Esse header informa ao navegador que, por determinado período, aquele host deve ser acessado apenas por HTTPS.

A MDN explica que o HSTS faz com que tentativas futuras de acesso via HTTP sejam automaticamente atualizadas para HTTPS pelo navegador, depois que o navegador já recebeu o header em uma conexão HTTPS válida.

Isso ajuda a reduzir riscos como:

- usuário digitando `http://`;
- links antigos usando HTTP;
- downgrade acidental;
- tentativa de interceptação em primeiro acesso posterior;
- cookies trafegando por canal inseguro;
- inconsistência entre endpoints HTTP e HTTPS.

## Como o HSTS funciona na prática

Imagine que o usuário acessou:

```text
https://meucatalogomusical.com
```

A resposta inclui:

```http
Strict-Transport-Security: max-age=31536000
```

O navegador armazena essa política.

Depois, se o usuário tentar acessar:

```text
http://meucatalogomusical.com/bandas/kreator
```

o navegador deve trocar automaticamente para:

```text
https://meucatalogomusical.com/bandas/kreator
```

antes de fazer a requisição.

Isso reforça o uso de HTTPS no navegador.

## HSTS não resolve tudo

HSTS é importante, mas possui limitações.

A própria MDN destaca uma fraqueza: o HSTS não tem efeito até que o navegador faça pelo menos uma conexão segura com o host e receba o header `Strict-Transport-Security`.

Além disso, HSTS não corrige:

- SQL Injection;
- XSS;
- SSRF;
- Path Traversal;
- autorização incorreta;
- cookies mal configurados;
- logs com dados sensíveis;
- dependências vulneráveis;
- APIs retornando dados demais;
- ausência de validação.

HSTS protege o uso de HTTPS pelo navegador. Ele não substitui segurança da aplicação.

## Cuidado com includeSubDomains

O header pode incluir:

```http
Strict-Transport-Security: max-age=31536000; includeSubDomains
```

`includeSubDomains` aplica a política também aos subdomínios.

Isso pode ser ótimo quando todos os subdomínios suportam HTTPS corretamente.

Mas pode quebrar ambientes se algum subdomínio legítimo ainda depende de HTTP.

Exemplo:

```text
meucatalogomusical.com
api.meucatalogomusical.com
admin.meucatalogomusical.com
legacy.meucatalogomusical.com
```

Se `legacy.meucatalogomusical.com` não suporta HTTPS e você habilita `includeSubDomains`, usuários podem perder acesso.

Por isso, HSTS precisa ser decisão técnica e operacional, não apenas um header colocado às pressas.

## max-age

`max-age` define por quantos segundos o navegador deve lembrar da política HSTS.

Exemplo:

```http
Strict-Transport-Security: max-age=31536000
```

`31536000` segundos equivalem a aproximadamente um ano.

Em ambientes reais, é comum começar com valores menores em homologação ou rollout controlado, validar todos os subdomínios e depois aumentar.

Exemplo de rollout gradual:

```http
Strict-Transport-Security: max-age=300
```

Depois:

```http
Strict-Transport-Security: max-age=86400
```

Depois:

```http
Strict-Transport-Security: max-age=31536000; includeSubDomains
```

A estratégia depende da empresa, dos domínios e da maturidade operacional.

## HSTS em Spring Security

O Spring Security fornece headers de segurança por padrão e documenta suporte para HSTS e `X-Content-Type-Options: nosniff`. A documentação informa que o Spring Security adiciona um conjunto padrão de headers relacionados a segurança e mostra `X-Content-Type-Options: nosniff` e comportamento padrão para HSTS.

Exemplo de configuração em Spring Security:

```java
package com.meucatalogomusical.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.Customizer;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.web.SecurityFilterChain;

import java.time.Duration;

@Configuration
public class SecurityHeadersConfig {

    @Bean
    SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        return http
                .headers(headers -> headers
                        .httpStrictTransportSecurity(hsts -> hsts
                                .maxAgeInSeconds(Duration.ofDays(365).toSeconds())
                                .includeSubDomains(true)
                        )
                        .contentTypeOptions(Customizer.withDefaults())
                )
                .authorizeHttpRequests(authorize -> authorize
                        .requestMatchers("/api/public/**").permitAll()
                        .anyRequest().authenticated()
                )
                .build();
    }
}
```

Esse exemplo configura:

- HSTS por 365 dias;
- inclusão de subdomínios;
- `X-Content-Type-Options`.

Em produção, essa configuração precisa estar alinhada com a infraestrutura, proxy, balanceador, TLS termination e política de domínio.

## HSTS atrás de proxy ou load balancer

Em muitas empresas, a aplicação Spring não recebe HTTPS diretamente.

Fluxo comum:

```text
Navegador
    -> HTTPS
Load Balancer / API Gateway
    -> HTTP interno
Aplicação Spring
```

Nesse cenário, o TLS termina no load balancer ou gateway.

A aplicação pode enxergar a requisição como HTTP, mesmo que o cliente tenha acessado por HTTPS.

Isso pode afetar geração de URLs, redirects, cookies `Secure` e políticas de segurança.

Por isso, é importante configurar corretamente headers encaminhados, como:

```text
X-Forwarded-Proto
Forwarded
```

e a configuração correspondente no Spring Boot e na infraestrutura.

O ponto de segurança é:

> O header HSTS deve refletir a política real vista pelo navegador.

Se o navegador acessa por HTTPS, o HSTS precisa aparecer na resposta HTTPS final.

## Missing HSTS Header em SAST

Um finding `Missing HSTS Header` normalmente indica que a ferramenta não encontrou o header `Strict-Transport-Security` em alguma resposta.

Isso pode acontecer por vários motivos:

- aplicação não configura HSTS;
- Spring Security não está habilitado;
- endpoint passa fora da cadeia de filtros;
- HSTS é configurado no gateway, mas o SAST analisou só código;
- ambiente de teste usa HTTP;
- scanner não entende configuração externa;
- response customizada não passa pelos headers padrão;
- arquivo/download é tratado por componente separado.

No CustomerData, o scan apontou `Missing_HSTS_Header` associado a `FileUtil.write`, o que mostra como ferramentas podem associar headers ausentes até a fluxos que não parecem, à primeira vista, relacionados a uma resposta HTTP.

A triagem deve responder:

```text
O HSTS é configurado na aplicação?
É configurado no gateway?
O endpoint passa por essa camada?
O header aparece na resposta real?
Há teste ou evidência HTTP?
```

## JSON explícito

Agora vamos ao segundo tema: JSON explícito.

Uma API REST deve declarar claramente que produz JSON.

Em Spring MVC, isso pode ser feito com `produces`.

A documentação do Spring Framework mostra o uso de `@GetMapping(..., produces = "application/json")` para declarar o media type produzido pelo endpoint.

Exemplo:

```java
package com.meucatalogomusical.bandas.api;

import org.springframework.http.MediaType;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/bandas")
public class BandaController {

    @GetMapping(
            value = "/{id}",
            produces = MediaType.APPLICATION_JSON_VALUE
    )
    public BandaResponse buscar(@PathVariable Long id) {
        return new BandaResponse(
                id,
                "Testament",
                "Thrash Metal"
        );
    }
}
```

DTO:

```java
package com.meucatalogomusical.bandas.api;

public record BandaResponse(
        Long id,
        String nome,
        String genero
) {
}
```

Esse endpoint deixa claro:

```text
Produzo application/json.
```

A resposta esperada:

```http
HTTP/1.1 200 OK
Content-Type: application/json
X-Content-Type-Options: nosniff
```

Corpo:

```json
{
  "id": 1,
  "nome": "Testament",
  "genero": "Thrash Metal"
}
```

## Por que `produces` importa?

Em muitos casos, o Spring já consegue retornar JSON automaticamente por causa do Jackson e da negociação de conteúdo.

Mas declarar `produces = MediaType.APPLICATION_JSON_VALUE` ajuda a deixar o contrato explícito.

Isso é útil porque:

- reduz ambiguidade;
- documenta intenção;
- ajuda testes;
- ajuda triagem SAST;
- evita negociação inesperada;
- diferencia API JSON de página HTML;
- ajuda a responder a findings de XSS em endpoints REST;
- reforça que o endpoint não deveria produzir `text/html`.

No caso do Databridge, a mitigação de findings de XSS passou por deixar claro que o dado permanecia como JSON, com `produces = application/json`, `Content-Type` JSON e `X-Content-Type-Options: nosniff`, sem entrar em contexto HTML, atributo, JavaScript ou DOM.

Esse é um aprendizado importante:

> Nem toda resposta com dado não confiável é XSS. O contexto de saída importa.

Se a resposta é JSON e não HTML, a mitigação correta pode ser reforçar o contrato JSON, não aplicar escape HTML aleatório.

## JSON não é HTML

Uma resposta JSON é dado.

Uma resposta HTML é documento interpretável pelo navegador.

Exemplo JSON:

```json
{
  "comentario": "Esse álbum do Kreator é excelente."
}
```

Exemplo HTML:

```html
<p>Esse álbum do Kreator é excelente.</p>
```

Se um dado não confiável entra em HTML sem escape, pode haver XSS.

Se o mesmo dado é retornado corretamente como JSON, com `Content-Type: application/json` e `nosniff`, ele não está automaticamente em um contexto de execução HTML.

Mas cuidado: o frontend que consumir esse JSON ainda precisa renderizar o valor com segurança.

Exemplo seguro no frontend:

```javascript
element.textContent = resposta.comentario;
```

Exemplo perigoso:

```javascript
element.innerHTML = resposta.comentario;
```

Ou seja:

- API deve produzir JSON corretamente;
- frontend deve renderizar corretamente;
- uma camada não anula a responsabilidade da outra.

## X-Content-Type-Options: nosniff

O header `X-Content-Type-Options` com valor `nosniff` instrui o navegador a respeitar o `Content-Type` informado.

```http
X-Content-Type-Options: nosniff
```

A MDN explica que esse header indica que os MIME types anunciados em `Content-Type` devem ser respeitados e não alterados.

Isso evita MIME sniffing.

MIME sniffing é quando o navegador tenta adivinhar o tipo do conteúdo, em vez de confiar apenas no `Content-Type`.

Isso pode ser perigoso quando uma resposta deveria ser tratada como texto ou JSON, mas o navegador tenta interpretá-la como HTML ou script.

A MDN recomenda configurar `X-Content-Type-Options: nosniff` e media types apropriados para arquivos servidos.

## Exemplo de nosniff

Resposta segura:

```http
HTTP/1.1 200 OK
Content-Type: application/json
X-Content-Type-Options: nosniff
```

Corpo:

```json
{
  "banda": "Overkill",
  "genero": "Thrash Metal"
}
```

Aqui o servidor diz:

```text
Isto é JSON.
Não tente adivinhar outro tipo.
```

Isso ajuda a impedir que o navegador interprete conteúdo em contexto diferente do esperado.

## Configurando nosniff no Spring Security

Com Spring Security, `X-Content-Type-Options: nosniff` faz parte dos headers de segurança padrão. A documentação do Spring Security mostra esse header dentro do conjunto de headers de segurança aplicados por padrão.

Configuração explícita:

```java
package com.meucatalogomusical.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.Customizer;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.web.SecurityFilterChain;

@Configuration
public class SecurityHeadersConfig {

    @Bean
    SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        return http
                .headers(headers -> headers
                        .contentTypeOptions(Customizer.withDefaults())
                )
                .authorizeHttpRequests(authorize -> authorize
                        .requestMatchers("/api/public/**").permitAll()
                        .anyRequest().authenticated()
                )
                .build();
    }
}
```

Em muitas aplicações, você não precisa configurar isso manualmente se o Spring Security já estiver adicionando os headers padrão. Mas explicitar pode ajudar quando a equipe quer deixar a política visível.

## Configurando headers manualmente com filtro

Quando não há Spring Security, ou quando existe um fluxo específico fora da cadeia padrão, você pode adicionar headers com filtro.

```java
package com.meucatalogomusical.config;

import jakarta.servlet.FilterChain;
import jakarta.servlet.ServletException;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import org.springframework.web.filter.OncePerRequestFilter;

import java.io.IOException;

public class SecurityHeadersFilter extends OncePerRequestFilter {

    @Override
    protected void doFilterInternal(
            HttpServletRequest request,
            HttpServletResponse response,
            FilterChain filterChain
    ) throws ServletException, IOException {

        response.setHeader("X-Content-Type-Options", "nosniff");

        if (request.isSecure()) {
            response.setHeader(
                    "Strict-Transport-Security",
                    "max-age=31536000; includeSubDomains"
            );
        }

        filterChain.doFilter(request, response);
    }
}
```

Esse exemplo é útil para fins didáticos.

Em uma aplicação real, prefira configurar headers de segurança de forma centralizada com Spring Security, gateway ou infraestrutura padronizada da empresa.

## HSTS deve ser enviado apenas sobre HTTPS

HSTS só faz sentido em resposta HTTPS.

Se você adiciona HSTS em uma resposta HTTP, navegadores não devem confiar naquela política, porque o canal não é seguro.

Por isso, o exemplo anterior usa:

```java
if (request.isSecure()) {
    response.setHeader(
            "Strict-Transport-Security",
            "max-age=31536000; includeSubDomains"
    );
}
```

Mas, atrás de proxy ou gateway, `request.isSecure()` pode depender da configuração correta de forwarded headers.

Em produção, valide a resposta real no navegador ou com `curl`.

## Testando headers com curl

Exemplo:

```bash
curl -I https://meucatalogomusical.com/api/bandas/1
```

Resposta esperada:

```http
HTTP/2 200
content-type: application/json
x-content-type-options: nosniff
strict-transport-security: max-age=31536000; includeSubDomains
```

Para um endpoint JSON, você quer ver:

```http
Content-Type: application/json
```

Para `nosniff`:

```http
X-Content-Type-Options: nosniff
```

Para HSTS em HTTPS:

```http
Strict-Transport-Security: max-age=...
```

Esse tipo de evidência é útil para fechar findings SAST/DAST.

## Testes com MockMvc

Teste do endpoint JSON:

```java
package com.meucatalogomusical.bandas.api;

import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.http.MediaType;
import org.springframework.test.web.servlet.MockMvc;

import static org.hamcrest.Matchers.containsString;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;

@WebMvcTest(BandaController.class)
class BandaControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @Test
    void deveRetornarJsonExplicitamente() throws Exception {
        mockMvc.perform(get("/api/bandas/1")
                        .accept(MediaType.APPLICATION_JSON))
                .andExpect(status().isOk())
                .andExpect(header().string("Content-Type", containsString("application/json")))
                .andExpect(jsonPath("$.nome").value("Testament"));
    }

    @Test
    void deveRecusarHtmlQuandoEndpointProduzSomenteJson() throws Exception {
        mockMvc.perform(get("/api/bandas/1")
                        .accept(MediaType.TEXT_HTML))
                .andExpect(status().isNotAcceptable());
    }
}
```

Esse teste valida duas coisas:

- o endpoint retorna JSON;
- o endpoint não negocia HTML.

Isso é muito útil quando você quer provar que o dado não entra em contexto HTML.

## Testando nosniff

Se você estiver com Spring Security carregado no teste, pode validar o header:

```java
package com.meucatalogomusical.bandas.api;

import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.context.annotation.Import;
import org.springframework.http.MediaType;
import org.springframework.test.web.servlet.MockMvc;

import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;

@WebMvcTest(BandaController.class)
@Import(SecurityHeadersConfig.class)
class BandaSecurityHeadersTest {

    @Autowired
    private MockMvc mockMvc;

    @Test
    void deveEnviarNosniff() throws Exception {
        mockMvc.perform(get("/api/bandas/1")
                        .accept(MediaType.APPLICATION_JSON))
                .andExpect(status().isOk())
                .andExpect(header().string("X-Content-Type-Options", "nosniff"));
    }
}
```

Dependendo da configuração de segurança do projeto, talvez você precise ajustar autenticação, filtros e contexto de teste.

O importante é ter evidência automatizada de que o header aparece.

## Testando HSTS

HSTS depende de requisição segura.

Em `MockMvc`, você pode simular uma requisição segura:

```java
package com.meucatalogomusical.bandas.api;

import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.context.annotation.Import;
import org.springframework.http.MediaType;
import org.springframework.test.web.servlet.MockMvc;

import static org.hamcrest.Matchers.containsString;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;

@WebMvcTest(BandaController.class)
@Import(SecurityHeadersConfig.class)
class BandaHstsTest {

    @Autowired
    private MockMvc mockMvc;

    @Test
    void deveEnviarHstsEmRequisicaoSegura() throws Exception {
        mockMvc.perform(get("/api/bandas/1")
                        .secure(true)
                        .accept(MediaType.APPLICATION_JSON))
                .andExpect(status().isOk())
                .andExpect(header().string(
                        "Strict-Transport-Security",
                        containsString("max-age=")
                ));
    }
}
```

Esse teste comprova que a aplicação envia HSTS quando a requisição é considerada segura.

Em produção, ainda é necessário validar no tráfego real, especialmente se houver proxy, gateway ou load balancer.

## Endpoint JSON seguro em Spring

Exemplo completo:

```java
package com.meucatalogomusical.bandas.api;

import org.springframework.http.MediaType;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping(
        value = "/api/bandas",
        produces = MediaType.APPLICATION_JSON_VALUE
)
public class BandaController {

    @GetMapping("/{id}")
    public BandaResponse buscar(@PathVariable Long id) {
        return new BandaResponse(
                id,
                "Amon Amarth",
                "Melodic Death Metal"
        );
    }

    @GetMapping
    public ListaBandasResponse listar() {
        return new ListaBandasResponse(
                java.util.List.of(
                        new BandaResponse(1L, "Amon Amarth", "Melodic Death Metal"),
                        new BandaResponse(2L, "Kreator", "Thrash Metal"),
                        new BandaResponse(3L, "Testament", "Thrash Metal")
                )
        );
    }
}
```

DTOs:

```java
package com.meucatalogomusical.bandas.api;

public record BandaResponse(
        Long id,
        String nome,
        String genero
) {
}
```

```java
package com.meucatalogomusical.bandas.api;

import java.util.List;

public record ListaBandasResponse(
        List<BandaResponse> bandas
) {
}
```

Colocar `produces = MediaType.APPLICATION_JSON_VALUE` no `@RequestMapping` da classe ajuda a declarar que todos os endpoints daquele controller produzem JSON.

## Endpoint que recebe JSON

Para requests com corpo JSON, também deixe claro o `consumes`.

```java
package com.meucatalogomusical.bandas.api;

import jakarta.validation.Valid;
import org.springframework.http.MediaType;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping(
        value = "/api/bandas",
        produces = MediaType.APPLICATION_JSON_VALUE
)
public class CadastroBandaController {

    @PostMapping(consumes = MediaType.APPLICATION_JSON_VALUE)
    public BandaResponse cadastrar(@Valid @RequestBody CadastroBandaRequest request) {
        return new BandaResponse(
                10L,
                request.nome(),
                request.genero()
        );
    }
}
```

DTO:

```java
package com.meucatalogomusical.bandas.api;

import jakarta.validation.constraints.NotBlank;
import jakarta.validation.constraints.Size;

public record CadastroBandaRequest(

        @NotBlank
        @Size(max = 120)
        String nome,

        @NotBlank
        @Size(max = 80)
        String genero
) {
}
```

Agora o contrato é claro:

```text
Recebo application/json.
Produzo application/json.
```

Isso reduz ambiguidade e melhora segurança.

## Erros também devem ser JSON

Não adianta o endpoint de sucesso retornar JSON se os erros retornam HTML padrão.

Exemplo de `ControllerAdvice`:

```java
package com.meucatalogomusical.shared.api;

import org.springframework.http.HttpStatus;
import org.springframework.http.MediaType;
import org.springframework.web.bind.annotation.*;

@RestControllerAdvice
@RequestMapping(produces = MediaType.APPLICATION_JSON_VALUE)
public class ApiExceptionHandler {

    @ExceptionHandler(IllegalArgumentException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public ErroResponse tratarEntradaInvalida(IllegalArgumentException ex) {
        return new ErroResponse("Requisição inválida.");
    }

    @ExceptionHandler(Exception.class)
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    public ErroResponse tratarErroInesperado(Exception ex) {
        return new ErroResponse("Não foi possível processar a solicitação.");
    }

    public record ErroResponse(String mensagem) {
    }
}
```

O objetivo é evitar:

- HTML de erro inesperado;
- stack trace exposto;
- `Content-Type` ambíguo;
- detalhes internos;
- resposta fora do contrato da API.

## Headers não substituem encoding

`Content-Type: application/json` e `nosniff` ajudam a garantir que a resposta será tratada como JSON.

Mas não substituem codificação correta em HTML.

Se outro sistema pegar o JSON e injetar em uma página com `innerHTML`, ainda pode haver XSS no consumidor.

Exemplo perigoso no frontend:

```javascript
resultado.innerHTML = resposta.comentario;
```

Exemplo seguro:

```javascript
resultado.textContent = resposta.comentario;
```

Portanto:

```text
API JSON segura ≠ frontend automaticamente seguro
```

Cada camada precisa respeitar o contexto de saída.

## Headers em download de arquivos

Para downloads, também é importante declarar `Content-Type` correto.

Exemplo:

```java
package com.meucatalogomusical.arquivos.api;

import org.springframework.core.io.Resource;
import org.springframework.http.*;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/arquivos")
public class ArquivoDownloadController {

    private final ArquivoService arquivoService;

    public ArquivoDownloadController(ArquivoService arquivoService) {
        this.arquivoService = arquivoService;
    }

    @GetMapping("/{id}/download")
    public ResponseEntity<Resource> download(@PathVariable Long id) {
        ArquivoDownload arquivo = arquivoService.carregar(id);

        return ResponseEntity.ok()
                .contentType(MediaType.APPLICATION_OCTET_STREAM)
                .header(
                        HttpHeaders.CONTENT_DISPOSITION,
                        "attachment; filename=\"" + arquivo.nomeSeguro() + "\""
                )
                .body(arquivo.resource());
    }
}
```

Para conteúdo não confiável, `attachment` costuma ser uma escolha mais conservadora do que `inline`.

Além disso, `nosniff` ajuda a evitar que o navegador tente interpretar o conteúdo de forma diferente.

## Quando usar text/plain?

Às vezes, uma resposta é texto puro.

Exemplo:

```java
@GetMapping(
        value = "/health-text",
        produces = MediaType.TEXT_PLAIN_VALUE
)
public String healthText() {
    return "OK";
}
```

Nesse caso, o `Content-Type` correto é:

```http
Content-Type: text/plain
```

Com:

```http
X-Content-Type-Options: nosniff
```

Se o corpo contiver caracteres que parecem HTML, o navegador não deve reinterpretar como HTML.

## Quando usar text/html?

Use `text/html` apenas quando a resposta realmente é HTML.

Exemplo com template server-side:

```java
@Controller
public class HomeController {

    @GetMapping("/")
    public String home() {
        return "home";
    }
}
```

Aqui o contexto muda.

Se a resposta é HTML, dados não confiáveis precisam de escape HTML contextual.

Esse é o ponto:

```text
JSON exige contrato JSON.
HTML exige escape HTML.
Arquivo exige Content-Type e Content-Disposition adequados.
```

Não existe um único tratamento que serve para tudo.

## Relação com XSS

Findings de XSS podem aparecer quando a ferramenta vê:

```text
dado não confiável -> resposta HTTP
```

Mas, para confirmar XSS, é preciso entender o contexto.

Perguntas:

```text
A resposta é HTML?
O dado entra em atributo HTML?
O dado entra em JavaScript?
O dado entra no DOM?
Ou permanece JSON?
```

Se o dado permanece em JSON com:

```http
Content-Type: application/json
X-Content-Type-Options: nosniff
```

e nenhum cliente intermediário injeta em HTML, o risco de XSS naquele endpoint pode ser falso positivo ou risco mitigado.

Foi exatamente esse tipo de raciocínio que apareceu no Databridge: a análise registrou que o dado permanecia em JSON e não entrava em HTML, atributo, JavaScript ou DOM.

## Relação com DAST

Ferramentas DAST podem apontar ausência de headers.

Exemplos:

```text
Missing Strict-Transport-Security Header
```

```text
Missing X-Content-Type-Options Header
```

```text
Content-Type not specified
```

Esses findings normalmente são verificados na resposta HTTP real.

Por isso, para corrigir, é importante validar:

```bash
curl -I https://meucatalogomusical.com/api/bandas/1
```

e não apenas olhar o código.

Às vezes o header é adicionado no gateway, não no Spring.

Às vezes é adicionado no Spring, mas removido no proxy.

Às vezes aparece em endpoints normais, mas não em erro ou download.

Teste a resposta real.

## Boas práticas

### 1. Habilite HSTS em produção

Use `Strict-Transport-Security` em respostas HTTPS, alinhado à política de domínio.

### 2. Faça rollout cuidadoso de HSTS

Cuidado com `includeSubDomains` e `preload`. Valide subdomínios antes.

### 3. Declare JSON explicitamente

Use `produces = MediaType.APPLICATION_JSON_VALUE` em APIs REST.

### 4. Declare consumes quando recebe JSON

Use `consumes = MediaType.APPLICATION_JSON_VALUE` em endpoints com `@RequestBody`.

### 5. Use nosniff

Configure `X-Content-Type-Options: nosniff`.

### 6. Teste headers

Use `MockMvc`, `curl`, DAST e validação em ambiente real.

### 7. Padronize erros

Erros também devem respeitar o contrato da API.

### 8. Não misture API JSON e HTML no mesmo endpoint

Separe controllers e contratos.

### 9. Cuidado com downloads

Use `Content-Type`, `Content-Disposition` e `nosniff`.

### 10. Documente decisões

Se um finding for falso positivo, documente contexto, headers e evidências.

## Erros comuns

### 1. Achar que HTTPS sozinho ativa HSTS

HTTPS e HSTS são coisas diferentes. HSTS precisa de header.

### 2. Ativar includeSubDomains sem mapear subdomínios

Isso pode quebrar subdomínios legados.

### 3. Retornar JSON sem Content-Type claro

Isso deixa espaço para interpretação ambígua.

### 4. Confiar que o framework sempre fará tudo certo

Framework ajuda, mas contrato explícito facilita auditoria e testes.

### 5. Não testar respostas de erro

Muitos headers somem justamente em erros, downloads ou fluxos customizados.

### 6. Usar `text/html` para resposta que é API

Isso aumenta risco de interpretação indevida.

### 7. Usar `innerHTML` no consumidor

Mesmo que a API seja JSON, o consumidor pode criar XSS se renderizar errado.

### 8. Achar que nosniff substitui validação

`nosniff` ajuda no cliente, mas não substitui validação e encoding.

### 9. Corrigir XSS com header errado

Headers ajudam, mas não corrigem saída HTML sem escape.

### 10. Não considerar gateway

Em produção, headers podem ser adicionados, alterados ou removidos pela infraestrutura.

## Decisões de engenharia

## Onde configurar headers?

Você pode configurar em:

- aplicação Spring;
- Spring Security;
- API Gateway;
- load balancer;
- reverse proxy;
- service mesh;
- CDN.

O ideal depende da arquitetura.

Em muitos ambientes corporativos, a política global fica no gateway, mas a aplicação também pode ter defaults defensivos.

O importante é evitar lacunas.

## HSTS na aplicação ou no gateway?

Se o TLS termina no gateway, muitas vezes faz sentido configurar HSTS no gateway.

Mas a aplicação também pode configurar se recebe corretamente a informação de que a requisição original era HTTPS.

A decisão deve considerar:

- onde TLS termina;
- quem controla domínios;
- quem controla subdomínios;
- consistência entre serviços;
- ambientes internos;
- testes;
- responsabilidade operacional.

## JSON explícito em todos os controllers?

Para APIs REST, sim, costuma ser boa prática.

Você pode declarar no controller:

```java
@RequestMapping(
        value = "/api/bandas",
        produces = MediaType.APPLICATION_JSON_VALUE
)
```

E declarar `consumes` nos métodos que recebem corpo.

Isso ajuda a padronizar.

## Quando não usar nosniff?

Em aplicações web modernas, `nosniff` deve ser padrão.

A MDN recomenda configurar `X-Content-Type-Options: nosniff` e media types adequados para os arquivos servidos.

Se algo quebra por causa de `nosniff`, normalmente é sinal de que o `Content-Type` está errado.

## Quando um finding pode ser falso positivo?

Pode ser falso positivo quando:

- HSTS é aplicado no gateway e não aparece no código;
- endpoint apontado como XSS produz apenas JSON;
- resposta tem `Content-Type: application/json`;
- resposta tem `X-Content-Type-Options: nosniff`;
- consumidor não injeta o JSON em HTML;
- evidência real mostra os headers;
- o scanner não entende a configuração da infraestrutura.

Mesmo assim, registre evidências.

## Onde isso aparece em sistemas reais

Esses headers aparecem em:

- APIs REST;
- gateways;
- downloads;
- sistemas internos;
- backoffices;
- respostas de erro;
- endpoints públicos;
- health checks;
- arquivos estáticos;
- integrações server-to-server;
- aplicações com frontend separado;
- serviços atrás de load balancer.

Em Java/Spring, os pontos comuns são:

- `SecurityFilterChain`;
- controllers REST;
- exception handlers;
- filtros servlet;
- endpoints de download;
- configurações do gateway;
- testes MockMvc;
- configurações de proxy;
- DAST em homologação;
- scans Checkmarx/SAST com heurísticas de XSS e headers.

## Checklist prático

### HSTS

- A aplicação usa HTTPS em produção?
- O header `Strict-Transport-Security` aparece na resposta real?
- `max-age` foi definido?
- `includeSubDomains` foi avaliado?
- Todos os subdomínios suportam HTTPS?
- HSTS é aplicado no gateway ou na aplicação?
- O teste considera proxy/load balancer?
- Há evidência com `curl -I`?

### JSON explícito

- Controllers REST declaram `produces = application/json`?
- Endpoints com body declaram `consumes = application/json`?
- Erros também retornam JSON?
- `Accept: text/html` é recusado quando o endpoint é apenas JSON?
- `Content-Type` aparece corretamente na resposta?
- DTOs públicos são usados?

### nosniff

- `X-Content-Type-Options: nosniff` aparece?
- Downloads também recebem headers adequados?
- Arquivos têm `Content-Type` correto?
- Respostas de erro mantêm o header?
- O gateway preserva o header?
- Testes automatizados verificam?

### XSS e contexto

- O endpoint realmente produz HTML?
- O dado entra em HTML, atributo, JavaScript ou DOM?
- Ou permanece JSON?
- O consumidor renderiza com segurança?
- Existe prova de `Content-Type` e `nosniff`?
- O finding SAST foi triado pelo contexto real?

### Operação

- Headers são consistentes entre ambientes?
- Homologação se parece com produção?
- Proxy remove algum header?
- CDN altera algum header?
- Load balancer adiciona HSTS?
- Existe monitoramento de regressão?

## Resumo final

Headers de segurança ajudam a reduzir ambiguidade no comportamento HTTP da aplicação.

Neste artigo, estudamos três pontos principais:

- `Strict-Transport-Security`;
- `Content-Type: application/json`;
- `X-Content-Type-Options: nosniff`.

O HSTS orienta o navegador a acessar o host apenas por HTTPS em acessos futuros.

O `Content-Type` explícito informa ao cliente qual é o tipo da resposta.

O `nosniff` instrui o navegador a respeitar o tipo informado, evitando MIME sniffing.

Em aplicações Java/Spring, esses headers são especialmente importantes para:

- fechar findings de Missing HSTS;
- reforçar APIs JSON;
- mitigar falsos positivos de XSS em endpoints que não produzem HTML;
- reduzir comportamento ambíguo;
- padronizar respostas;
- melhorar evidências para AppSec.

A principal lição é:

> Segurança também depende de contrato HTTP claro. Se a resposta é JSON, diga que é JSON. Se o domínio deve usar HTTPS, declare isso. Se o navegador não deve adivinhar o tipo, use nosniff.

## Perguntas de revisão

### 1. O que é HSTS?

HSTS é uma política HTTP enviada pelo header `Strict-Transport-Security` para informar ao navegador que o host deve ser acessado apenas por HTTPS.

### 2. HSTS substitui HTTPS?

Não. HSTS depende de HTTPS. Ele reforça acessos futuros por HTTPS.

### 3. Qual é uma limitação do HSTS?

Ele só passa a valer depois que o navegador acessa o host por HTTPS e recebe o header.

### 4. O que faz `includeSubDomains`?

Aplica a política HSTS também aos subdomínios.

### 5. Qual é o risco de `includeSubDomains`?

Pode quebrar subdomínios que ainda não suportam HTTPS corretamente.

### 6. O que é `Content-Type`?

É o header que informa o media type da resposta, como `application/json` ou `text/html`.

### 7. Por que declarar `application/json` explicitamente?

Para deixar claro que o endpoint produz JSON e reduzir ambiguidade de interpretação.

### 8. O que faz `X-Content-Type-Options: nosniff`?

Instrui o navegador a respeitar o `Content-Type` informado e não tentar adivinhar outro tipo.

### 9. `nosniff` corrige XSS?

Não. Ele ajuda a reduzir MIME sniffing, mas não substitui escape HTML, validação e renderização segura.

### 10. JSON pode causar XSS?

JSON corretamente servido como `application/json` não é contexto HTML. Mas o consumidor pode criar XSS se inserir dados do JSON em HTML de forma insegura.

### 11. O que testar em APIs JSON?

Teste `Content-Type`, `Accept`, erros, `nosniff` e ausência de negociação HTML quando o endpoint é apenas JSON.

### 12. Por que respostas de erro importam?

Porque muitas aplicações retornam HTML ou stack trace em erro, quebrando o contrato JSON.

### 13. Onde configurar headers?

Na aplicação, Spring Security, gateway, load balancer, reverse proxy ou CDN, dependendo da arquitetura.

### 14. Quando um finding de Missing HSTS pode ser falso positivo?

Quando o HSTS é aplicado fora do código, por exemplo no gateway, e há evidência da resposta real com o header.

### 15. Qual é a principal regra?

Declare explicitamente como a resposta deve ser tratada e garanta que o cliente não precise adivinhar.

## Referências

- [MDN — Strict-Transport-Security](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Strict-Transport-Security)
- [MDN — Strict-Transport-Security em português](https://developer.mozilla.org/pt-BR/docs/Web/HTTP/Reference/Headers/Strict-Transport-Security)
- [MDN — Content-Type](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Content-Type)
- [MDN — X-Content-Type-Options](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/X-Content-Type-Options)
- [MDN — X-Content-Type-Options em português](https://developer.mozilla.org/pt-BR/docs/Web/HTTP/Reference/Headers/X-Content-Type-Options)
- [MDN — MIME type verification](https://developer.mozilla.org/en-US/docs/Web/Security/Practical_implementation_guides/MIME_types)
- [Spring Security — Security HTTP Response Headers](https://docs.spring.io/spring-security/reference/features/exploits/headers.html)
- [Spring Framework — Mapping Requests](https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-controller/ann-requestmapping.html)
- [Oracle JDK 25 Documentation](https://docs.oracle.com/en/java/javase/25/)
- [Oracle Java Downloads — JDK 25 LTS](https://www.oracle.com/java/technologies/downloads/)