# HTTP, DNS, Sessões e HTTPS na Segurança Web

No artigo anterior, vimos que aplicações web possuem uma natureza global, uma grande superfície de ataque e múltiplas camadas de complexidade. Também vimos que segurança de aplicações web não é a mesma coisa que segurança de rede, porque a segurança da aplicação depende diretamente de como o sistema recebe dados, processa requisições, valida usuários, autoriza ações e protege informações sensíveis.

Agora vamos avançar para a base técnica que sustenta boa parte da comunicação entre navegador e servidor: HTTP, DNS, sessões e HTTPS.

Esses conceitos aparecem em praticamente qualquer aplicação web moderna. Mesmo quando usamos frameworks como Spring Boot, bibliotecas de segurança e servidores gerenciados, esses fundamentos continuam existindo por baixo da abstração.

Quando um usuário acessa a aplicação fictícia `meucatalogomusical.com`, várias coisas acontecem:

1. O navegador precisa descobrir para qual servidor aquele domínio aponta.
2. O cliente estabelece comunicação com o servidor.
3. O navegador envia uma requisição HTTP.
4. O servidor processa a requisição.
5. A aplicação responde.
6. O navegador renderiza o conteúdo.
7. Se o usuário estiver autenticado, a aplicação precisa manter algum tipo de estado de sessão.
8. Se a comunicação for segura, tudo isso deve ocorrer sobre HTTPS.

Entender essas etapas ajuda a desenvolver aplicações mais seguras e a investigar problemas reais em produção.

## O que é HTTP?

HTTP significa Hypertext Transfer Protocol. Ele é o protocolo usado para troca de dados na web.

De forma simples, HTTP define como um cliente e um servidor se comunicam.

O cliente normalmente é o navegador, mas também pode ser:

* um aplicativo mobile;
* uma aplicação backend;
* uma ferramenta de linha de comando;
* um serviço externo;
* um API Gateway;
* um script automatizado.

O servidor é quem recebe a requisição, processa a solicitação e devolve uma resposta.

Por exemplo, quando alguém acessa:

```text
https://meucatalogomusical.com/bandas/judas-priest
```

o navegador envia uma requisição para o servidor responsável por `meucatalogomusical.com`, pedindo o recurso `/bandas/judas-priest`.

O servidor pode responder com uma página HTML, um JSON, um arquivo, um redirecionamento, uma mensagem de erro ou qualquer outro tipo de resposta suportada pela aplicação.

## HTTP segue o modelo cliente-servidor

O HTTP funciona em um modelo clássico de cliente e servidor.

O fluxo básico é:

1. O cliente abre uma conexão.
2. O cliente envia uma requisição.
3. O servidor recebe a requisição.
4. O servidor processa a requisição.
5. O servidor envia uma resposta.
6. O cliente interpreta a resposta.

Em uma aplicação web, esse ciclo acontece o tempo todo.

Ao abrir a página inicial de `meucatalogomusical.com`, o navegador pode fazer uma requisição para obter o HTML. Depois, pode fazer outras requisições para buscar CSS, JavaScript, imagens, fontes, dados da API e recursos adicionais.

Portanto, uma única tela pode envolver várias requisições HTTP.

## HTTP é sem estado

Um dos pontos mais importantes para segurança web é entender que HTTP é um protocolo sem estado.

Isso significa que, por padrão, cada requisição é independente. O servidor não lembra automaticamente quem fez a requisição anterior.

Por exemplo:

```text
GET /bandas
```

e depois:

```text
GET /usuarios/perfil
```

são duas requisições diferentes.

O protocolo HTTP, sozinho, não sabe que essas duas requisições vieram do mesmo usuário. Para resolver esse problema, aplicações web usam mecanismos adicionais, como cookies, tokens e sessões.

Esse detalhe é extremamente importante. Muitos problemas de segurança aparecem justamente na tentativa de criar estado sobre um protocolo originalmente sem estado.

## Requisição HTTP

Uma requisição HTTP é uma mensagem enviada pelo cliente ao servidor.

Ela geralmente possui:

* método HTTP;
* URL ou caminho do recurso;
* headers;
* parâmetros;
* corpo da requisição, quando aplicável.

Exemplo conceitual:

```http
GET /api/bandas?genero=HEAVY_METAL HTTP/1.1
Host: meucatalogomusical.com
Accept: application/json
```

Nesse caso, o cliente está pedindo uma lista de bandas filtradas pelo gênero `HEAVY_METAL`.

Em uma API REST, esse endpoint poderia retornar bandas como:

```json
[
  {
    "nome": "Judas Priest",
    "genero": "HEAVY_METAL"
  },
  {
    "nome": "Saxon",
    "genero": "HEAVY_METAL"
  }
]
```

## Resposta HTTP

Uma resposta HTTP é a mensagem devolvida pelo servidor.

Ela geralmente possui:

* código de status;
* headers;
* corpo da resposta.

Exemplo conceitual:

```http
HTTP/1.1 200 OK
Content-Type: application/json
```

Com o corpo:

```json
[
  {
    "nome": "Judas Priest",
    "genero": "HEAVY_METAL"
  }
]
```

O código `200 OK` indica que a requisição foi processada com sucesso.

Outros códigos comuns são:

* `201 Created`: recurso criado com sucesso;
* `204 No Content`: sucesso sem corpo de resposta;
* `400 Bad Request`: requisição inválida;
* `401 Unauthorized`: usuário não autenticado;
* `403 Forbidden`: usuário autenticado, mas sem permissão;
* `404 Not Found`: recurso não encontrado;
* `500 Internal Server Error`: erro interno no servidor.

Esses códigos não são apenas detalhes técnicos. Eles comunicam para clientes, APIs e ferramentas o resultado da operação.

## Métodos HTTP

Os métodos HTTP indicam a intenção da requisição.

Os mais comuns são:

* `GET`;
* `POST`;
* `PUT`;
* `PATCH`;
* `DELETE`.

Neste artigo, vamos focar principalmente em `GET` e `POST`, porque eles aparecem com muita frequência em aplicações web.

## GET

O método `GET` é usado para solicitar dados de um recurso.

Exemplo:

```http
GET /api/bandas?nome=sepultura HTTP/1.1
Host: meucatalogomusical.com
```

Nesse caso, o cliente está solicitando dados. A intenção não deveria ser criar, alterar ou excluir informações.

Exemplo em Spring:

```java
package com.meucatalogomusical.bandas.api;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

import java.util.List;

@RestController
public class ConsultaBandaController {

    private final ConsultaBandaService consultaBandaService;

    public ConsultaBandaController(ConsultaBandaService consultaBandaService) {
        this.consultaBandaService = consultaBandaService;
    }

    @GetMapping("/api/bandas")
    public List<BandaResponse> pesquisar(@RequestParam String nome) {
        return consultaBandaService.pesquisarPorNome(nome);
    }
}
```

Uma chamada possível seria:

```text
GET /api/bandas?nome=Sepultura
```

O ponto de atenção é que parâmetros enviados na URL podem aparecer em histórico do navegador, logs, ferramentas de monitoramento e registros intermediários. Por isso, dados sensíveis não devem ser enviados por URL.

Evite fazer algo como:

```text
GET /login?usuario=admin&senha=senha123
```

Esse tipo de desenho é inseguro porque expõe credenciais na própria URL.

## POST

O método `POST` é normalmente usado para enviar dados ao servidor, criar recursos ou iniciar operações.

Exemplo:

```http
POST /api/bandas HTTP/1.1
Host: meucatalogomusical.com
Content-Type: application/json
```

Corpo da requisição:

```json
{
  "nome": "Megadeth",
  "genero": "THRASH_METAL",
  "anoFormacao": 1983
}
```

Exemplo em Spring:

```java
package com.meucatalogomusical.bandas.api;

import jakarta.validation.Valid;
import org.springframework.http.HttpStatus;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/bandas")
public class CadastroBandaController {

    private final CadastroBandaService cadastroBandaService;

    public CadastroBandaController(CadastroBandaService cadastroBandaService) {
        this.cadastroBandaService = cadastroBandaService;
    }

    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public BandaResponse cadastrar(@Valid @RequestBody CadastroBandaRequest request) {
        return cadastroBandaService.cadastrar(request);
    }
}
```

Nesse caso, os dados são enviados no corpo da requisição, não na URL.

Isso não significa que `POST` seja automaticamente seguro. O corpo da requisição também pode ser interceptado se a comunicação não estiver protegida por HTTPS. Além disso, o backend ainda precisa validar os dados, aplicar autorização e tratar erros corretamente.

## GET é menos seguro que POST?

Uma simplificação comum é dizer que `POST` é mais seguro que `GET`.

Essa frase pode ser enganosa.

O que acontece é que, em uma requisição `GET`, os dados normalmente aparecem na URL. Isso pode torná-los mais expostos em alguns contextos, como histórico do navegador, favoritos, logs e ferramentas de monitoramento.

Já no `POST`, os dados ficam no corpo da requisição. Isso reduz a exposição na URL, mas não torna a requisição segura por si só.

A segurança real depende de outros fatores:

* uso de HTTPS;
* autenticação;
* autorização;
* validação de entrada;
* proteção contra CSRF quando aplicável;
* controle de sessão;
* configuração de cookies;
* tratamento seguro de erros;
* logs sem dados sensíveis.

Portanto, a regra prática é:

* use `GET` para leitura de recursos;
* use `POST` para envio de dados, criação de recursos ou operações que alteram estado;
* nunca envie dados sensíveis na URL;
* sempre use HTTPS para proteger a comunicação.

## GET deve alterar dados?

Em uma aplicação bem projetada, `GET` não deve alterar estado no servidor.

Isso significa que uma chamada como esta não é uma boa prática:

```text
GET /api/bandas/42/excluir
```

O problema é que `GET` pode ser executado por navegador, crawler, cache, ferramenta de preview ou outros mecanismos que assumem que a requisição é apenas uma leitura.

O ideal seria usar um método apropriado para exclusão:

```http
DELETE /api/bandas/42
```

ou, dependendo da regra de negócio, um `POST` para uma ação específica:

```http
POST /api/bandas/42/desativacao
```

A escolha do método HTTP comunica a intenção da operação.

## DNS: a lista telefônica da internet

DNS significa Domain Name System.

O DNS funciona como uma espécie de lista telefônica da internet. Ele associa nomes de domínio a endereços IP.

Quando o usuário digita:

```text
meucatalogomusical.com
```

o navegador precisa descobrir qual endereço IP corresponde a esse domínio.

O usuário lembra nomes como:

```text
meucatalogomusical.com
```

Mas os computadores se comunicam usando endereços IP.

O DNS resolve essa diferença.

## DNS direto e DNS reverso

Existem dois conceitos importantes:

* DNS direto;
* DNS reverso.

DNS direto é quando buscamos o endereço IP a partir de um nome de domínio.

Exemplo:

```text
meucatalogomusical.com -> 203.0.113.10
```

DNS reverso é quando buscamos o nome de domínio a partir de um endereço IP.

Exemplo:

```text
203.0.113.10 -> meucatalogomusical.com
```

Em termos simples:

* DNS direto: domínio para IP;
* DNS reverso: IP para domínio.

## Por que DNS importa para segurança?

DNS é uma parte essencial do caminho até a aplicação.

Se o usuário for direcionado para o servidor errado, ele pode acabar acessando uma página falsa ou um serviço malicioso.

DNS também aparece em várias atividades operacionais e de segurança, como:

* configuração de domínios;
* certificados TLS;
* validação de propriedade de domínio;
* investigação de incidentes;
* rastreamento de origem;
* configuração de ambientes;
* roteamento de tráfego;
* uso de CDN;
* proteção contra domínios falsos.

Em ambientes corporativos, a segurança não termina no código da aplicação. O domínio, o certificado, o roteamento, os registros DNS e a infraestrutura também fazem parte da experiência segura do usuário.

## Ciclo simplificado de uma requisição HTTP

Vamos imaginar o usuário acessando:

```text
https://meucatalogomusical.com/bandas/motorhead
```

Um fluxo simplificado seria:

1. O usuário digita a URL no navegador.
2. O navegador consulta o DNS para descobrir o IP do domínio.
3. O navegador estabelece conexão com o servidor.
4. Como a URL usa HTTPS, ocorre uma negociação TLS.
5. O navegador valida o certificado do servidor.
6. O navegador envia uma requisição HTTP.
7. O servidor processa a requisição.
8. A aplicação consulta dados, se necessário.
9. O servidor devolve uma resposta HTTP.
10. O navegador renderiza o resultado.

Cada etapa pode ter implicações de segurança.

Se o DNS estiver incorreto, o usuário pode ir para o destino errado.

Se o certificado TLS estiver inválido, a autenticidade do servidor fica comprometida.

Se a requisição carregar dados sensíveis sem HTTPS, esses dados podem ser expostos.

Se a sessão for mal protegida, um invasor pode tentar reutilizar identificadores de sessão.

## Sessões em aplicações web

Como HTTP é sem estado, aplicações precisam de algum mecanismo para reconhecer o usuário entre múltiplas requisições.

É aí que entram as sessões.

Uma sessão representa um contexto temporário associado a um usuário ou cliente.

Por exemplo, após fazer login em `meucatalogomusical.com`, o usuário pode navegar por várias páginas:

```text
GET /minha-conta
GET /minhas-bandas-favoritas
POST /avaliacoes
GET /meus-comentarios
```

A aplicação precisa saber que todas essas requisições pertencem ao mesmo usuário autenticado.

## Cookies de sessão

Um dos mecanismos mais comuns para manter sessão é o cookie.

O fluxo simplificado é:

1. O usuário faz login.
2. O servidor valida as credenciais.
3. O servidor cria uma sessão ou emite um identificador.
4. O servidor envia um cookie ao navegador.
5. O navegador armazena esse cookie.
6. Nas próximas requisições, o navegador envia o cookie automaticamente.
7. O servidor usa o cookie para reconhecer a sessão.

Exemplo conceitual de resposta:

```http
Set-Cookie: SESSION=abc123; Path=/; HttpOnly; Secure; SameSite=Lax
```

Esse cookie pode representar a sessão do usuário.

## Atributos importantes de cookies

Cookies de sessão precisam ser configurados com cuidado.

Alguns atributos importantes são:

* `HttpOnly`;
* `Secure`;
* `SameSite`;
* `Path`;
* `Domain`;
* tempo de expiração.

## HttpOnly

O atributo `HttpOnly` impede que o cookie seja acessado diretamente por JavaScript no navegador.

Isso é importante porque, em caso de vulnerabilidade XSS, o invasor teria mais dificuldade para ler o cookie de sessão via script.

Exemplo:

```http
Set-Cookie: SESSION=abc123; HttpOnly
```

Isso não elimina todos os riscos de XSS, mas reduz um impacto específico: o roubo direto do cookie por JavaScript.

## Secure

O atributo `Secure` instrui o navegador a enviar o cookie apenas por conexões HTTPS.

Exemplo:

```http
Set-Cookie: SESSION=abc123; Secure
```

Esse atributo ajuda a evitar que cookies sensíveis trafeguem em conexões HTTP sem criptografia.

Em aplicações modernas, cookies de sessão devem usar `Secure` em produção.

## SameSite

O atributo `SameSite` ajuda a controlar quando o navegador envia cookies em requisições cross-site.

Valores comuns:

* `Strict`;
* `Lax`;
* `None`.

Esse atributo é importante para reduzir certos riscos envolvendo envio automático de cookies em contextos externos.

Exemplo:

```http
Set-Cookie: SESSION=abc123; SameSite=Lax
```

A escolha entre `Strict`, `Lax` e `None` depende do comportamento esperado da aplicação.

## Configuração de sessão em Spring Boot

Em uma aplicação Spring Boot, algumas configurações de cookie podem ser definidas no `application.yml`.

Exemplo:

```yaml
server:
  servlet:
    session:
      cookie:
        name: MEU_CATALOGO_SESSION
        http-only: true
        secure: true
        same-site: lax
```

Essa configuração indica que o cookie de sessão:

* terá um nome específico;
* não deve ser acessível via JavaScript;
* deve trafegar apenas por HTTPS;
* terá política `SameSite=Lax`.

Em produção, o atributo `secure: true` é especialmente importante porque cookies de sessão não devem trafegar por HTTP.

## Exemplo de controller com sessão

Para fins didáticos, imagine que a aplicação permita armazenar uma preferência temporária de gênero musical na sessão do usuário.

```java
package com.meucatalogomusical.preferencias.api;

import jakarta.servlet.http.HttpSession;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/preferencias")
public class PreferenciaController {

    @PostMapping("/genero")
    public void definirGeneroPreferido(@RequestParam String genero, HttpSession session) {
        session.setAttribute("generoPreferido", genero);
    }

    @GetMapping("/genero")
    public String consultarGeneroPreferido(HttpSession session) {
        Object genero = session.getAttribute("generoPreferido");

        if (genero == null) {
            return "Nenhum gênero preferido definido.";
        }

        return genero.toString();
    }
}
```

Esse exemplo mostra como a sessão pode manter informação entre requisições.

Porém, em aplicações reais, é preciso tomar cuidado com o que é armazenado na sessão. Não devemos guardar dados sensíveis desnecessariamente, objetos muito grandes ou informações que possam gerar inconsistência.

## Sessão não substitui autorização

Uma sessão autenticada não significa que o usuário pode fazer qualquer coisa.

Depois de autenticar o usuário, a aplicação ainda precisa verificar autorização.

Exemplo:

```java
package com.meucatalogomusical.admin.api;

import org.springframework.security.access.prepost.PreAuthorize;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/admin/bandas")
public class AdminBandaController {

    private final AdminBandaService adminBandaService;

    public AdminBandaController(AdminBandaService adminBandaService) {
        this.adminBandaService = adminBandaService;
    }

    @DeleteMapping("/{id}")
    @PreAuthorize("hasRole('ADMIN')")
    public void excluir(@PathVariable Long id) {
        adminBandaService.excluir(id);
    }
}
```

Nesse exemplo, apenas usuários com papel `ADMIN` podem excluir uma banda.

Isso reforça uma diferença importante:

* autenticação verifica quem é o usuário;
* autorização verifica o que ele pode fazer.

## Riscos em sessões

Sessões mal protegidas podem gerar vulnerabilidades sérias.

Alguns riscos comuns são:

* roubo de cookie de sessão;
* sessão sem expiração adequada;
* cookie trafegando sem HTTPS;
* ausência de `HttpOnly`;
* ausência de `Secure`;
* sessão não invalidada após logout;
* sessão mantida por tempo excessivo;
* identificador previsível;
* reutilização de sessão após autenticação.

Um cookie de sessão deve ser tratado como dado sensível. Se alguém obtém um identificador de sessão válido, pode tentar se passar pelo usuário.

## Logout e invalidação de sessão

Quando o usuário faz logout, a sessão precisa ser encerrada corretamente.

Em aplicações com sessão no servidor, isso normalmente envolve invalidar a sessão.

Exemplo didático:

```java
package com.meucatalogomusical.autenticacao.api;

import jakarta.servlet.http.HttpSession;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class LogoutController {

    @PostMapping("/logout-manual")
    public void logout(HttpSession session) {
        session.invalidate();
    }
}
```

Em projetos reais com Spring Security, normalmente usamos os mecanismos próprios do framework para logout, porque eles já integram limpeza de contexto de segurança, sessão e cookies conforme a configuração.

## HTTPS

HTTPS significa Hypertext Transfer Protocol Secure.

Ele é a versão segura do HTTP, usando TLS para proteger a comunicação entre cliente e servidor.

Quando uma aplicação usa HTTPS, ela busca garantir:

* criptografia da comunicação;
* integridade dos dados em trânsito;
* autenticação do servidor por meio de certificado.

Em termos simples, HTTPS ajuda a evitar que terceiros leiam ou alterem facilmente os dados trafegados entre navegador e servidor.

## HTTP vs HTTPS

HTTP sem TLS trafega sem criptografia.

HTTPS usa TLS para proteger a comunicação.

Exemplo de URL com HTTP:

```text
http://meucatalogomusical.com
```

Exemplo de URL com HTTPS:

```text
https://meucatalogomusical.com
```

Para aplicações modernas, especialmente aquelas que envolvem login, sessão, dados pessoais ou operações sensíveis, HTTPS é indispensável.

## TLS

TLS significa Transport Layer Security.

Ele é o protocolo usado para proteger a comunicação no HTTPS.

O TLS ajuda a garantir que:

* os dados trafeguem criptografados;
* o servidor seja autenticado por certificado;
* a comunicação não seja facilmente alterada no caminho.

SSL foi um protocolo anterior ao TLS e hoje é considerado obsoleto. Em aplicações modernas, a configuração deve usar versões fortes de TLS, como TLS 1.2 e TLS 1.3, conforme compatibilidade e requisitos do ambiente.

## Certificado digital

Para usar HTTPS, o servidor precisa de um certificado digital.

Esse certificado ajuda o navegador a verificar se o servidor acessado corresponde ao domínio esperado.

Quando o usuário acessa:

```text
https://meucatalogomusical.com
```

o navegador verifica o certificado apresentado pelo servidor.

Se houver problema no certificado, o navegador pode exibir um alerta de segurança.

Problemas comuns incluem:

* certificado expirado;
* certificado emitido para outro domínio;
* cadeia de confiança inválida;
* configuração incorreta;
* uso de protocolos ou algoritmos fracos.

## HTTPS protege tudo?

HTTPS é essencial, mas não resolve todos os problemas de segurança da aplicação.

HTTPS protege os dados em trânsito entre cliente e servidor, mas não corrige, por exemplo:

* SQL Injection;
* XSS;
* autorização mal implementada;
* senha armazenada de forma insegura;
* regra de negócio vulnerável;
* exposição de dados em logs;
* dependências vulneráveis;
* erros de configuração;
* sessão mal gerenciada.

Portanto, HTTPS é uma camada fundamental, mas não é a única camada de proteção.

## Cookies seguros com HTTPS

Cookies de sessão devem ser combinados com HTTPS.

Um cookie com atributo `Secure` só deve ser enviado em conexões HTTPS.

Exemplo:

```http
Set-Cookie: MEU_CATALOGO_SESSION=abc123; HttpOnly; Secure; SameSite=Lax
```

Esse tipo de configuração ajuda a proteger a sessão contra exposição em conexões não criptografadas.

Em Spring Boot, a configuração poderia ser:

```yaml
server:
  servlet:
    session:
      cookie:
        name: MEU_CATALOGO_SESSION
        http-only: true
        secure: true
        same-site: lax
```

Além disso, em aplicações atrás de proxy, load balancer ou gateway, pode ser necessário configurar corretamente o reconhecimento de headers encaminhados, para que a aplicação entenda que a requisição original chegou por HTTPS.

## Exemplo de segurança no Spring Security

Uma configuração simples de segurança pode exigir autenticação para rotas privadas e permitir acesso público a rotas abertas.

Exemplo didático:

```java
package com.meucatalogomusical.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.web.SecurityFilterChain;

@Configuration
public class SecurityConfig {

    @Bean
    SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        return http
                .authorizeHttpRequests(authorize -> authorize
                        .requestMatchers("/", "/api/bandas/publicas/**").permitAll()
                        .requestMatchers("/api/admin/**").hasRole("ADMIN")
                        .anyRequest().authenticated()
                )
                .formLogin(form -> form
                        .defaultSuccessUrl("/minha-conta", true)
                )
                .logout(logout -> logout
                        .logoutUrl("/logout")
                        .invalidateHttpSession(true)
                        .deleteCookies("MEU_CATALOGO_SESSION")
                )
                .build();
    }
}
```

Esse exemplo mostra alguns pontos importantes:

* rotas públicas são permitidas;
* rotas administrativas exigem papel `ADMIN`;
* demais rotas exigem autenticação;
* logout invalida a sessão;
* logout remove o cookie de sessão.

Esse código é didático. Uma aplicação real pode exigir mais configurações, como CSRF, política de CORS, autenticação via OAuth2/OpenID Connect, integração com provedor de identidade, configuração de headers e outros controles.

## Headers de segurança

Além de HTTPS e cookies, aplicações web podem usar headers de segurança.

Alguns exemplos são:

* `Strict-Transport-Security`;
* `Content-Security-Policy`;
* `X-Content-Type-Options`;
* `Referrer-Policy`;
* `Frame-Options` ou diretivas equivalentes via CSP.

Um header importante relacionado a HTTPS é o HSTS, representado por `Strict-Transport-Security`.

Ele instrui o navegador a acessar o domínio apenas por HTTPS por determinado período.

Exemplo conceitual:

```http
Strict-Transport-Security: max-age=31536000; includeSubDomains
```

Esse header deve ser usado com cuidado, especialmente com `includeSubDomains`, porque uma configuração incorreta pode afetar todos os subdomínios.

## Dados sensíveis não devem aparecer em URLs

Como URLs podem ser armazenadas em histórico, logs e ferramentas intermediárias, evite colocar dados sensíveis nelas.

Evite:

```text
GET /login?email=usuario@example.com&senha=123456
```

Prefira enviar credenciais no corpo de uma requisição apropriada e sempre sobre HTTPS.

Mesmo assim, o backend precisa tomar cuidado para não registrar credenciais em logs.

## Cuidado com logs

Logs são fundamentais para diagnóstico, auditoria e investigação de incidentes, mas também podem se tornar fonte de vazamento de dados.

Evite registrar:

* senhas;
* tokens;
* cookies;
* dados pessoais desnecessários;
* números de documentos;
* segredos de API;
* headers sensíveis;
* payloads completos sem necessidade.

Exemplo ruim:

```java
log.info("Tentativa de login: email={}, senha={}", email, senha);
```

Exemplo melhor:

```java
log.info("Tentativa de login recebida para email={}", mascararEmail(email));
```

Com método auxiliar:

```java
private String mascararEmail(String email) {
    int indiceArroba = email.indexOf("@");

    if (indiceArroba <= 1) {
        return "***";
    }

    return email.charAt(0) + "***" + email.substring(indiceArroba);
}
```

Esse tipo de cuidado reduz o risco de logs armazenarem dados sensíveis.

## Relação entre HTTP, sessão e segurança

A relação entre esses conceitos pode ser resumida assim:

* HTTP permite a comunicação entre cliente e servidor.
* HTTP é sem estado.
* Sessões ajudam a manter contexto entre requisições.
* Cookies são frequentemente usados para identificar sessões.
* HTTPS protege a comunicação em trânsito.
* DNS permite localizar o servidor por meio de um domínio.
* Certificados ajudam a validar a identidade do servidor.
* Configurações incorretas podem comprometer a segurança.

Esses elementos formam a base da segurança web.

## Exemplo completo: fluxo de login em uma aplicação web

Imagine o fluxo de login em `meucatalogomusical.com`.

1. O usuário acessa:

```text
https://meucatalogomusical.com/login
```

2. O DNS resolve o domínio para um endereço IP.

3. O navegador estabelece conexão HTTPS.

4. O usuário envia e-mail e senha via `POST`.

```http
POST /login HTTP/1.1
Host: meucatalogomusical.com
Content-Type: application/x-www-form-urlencoded
```

5. O servidor valida as credenciais.

6. Se estiver correto, o servidor cria uma sessão.

7. O servidor envia um cookie de sessão.

```http
Set-Cookie: MEU_CATALOGO_SESSION=abc123; HttpOnly; Secure; SameSite=Lax
```

8. Nas próximas requisições, o navegador envia o cookie.

```http
GET /minha-conta HTTP/1.1
Host: meucatalogomusical.com
Cookie: MEU_CATALOGO_SESSION=abc123
```

9. O servidor reconhece o usuário pela sessão.

10. A aplicação aplica autorização para decidir o que aquele usuário pode acessar.

Esse fluxo mostra como HTTP, HTTPS, cookie, sessão, autenticação e autorização trabalham juntos.

## Boas práticas

Algumas boas práticas importantes:

* usar HTTPS em produção;
* evitar dados sensíveis em URLs;
* usar `GET` para leitura;
* usar `POST`, `PUT`, `PATCH` ou `DELETE` conforme a intenção da operação;
* configurar cookies de sessão com `HttpOnly`;
* configurar cookies de sessão com `Secure`;
* avaliar corretamente o uso de `SameSite`;
* invalidar sessão no logout;
* aplicar expiração adequada de sessão;
* não armazenar dados sensíveis desnecessários na sessão;
* validar entrada no backend;
* aplicar autorização em endpoints sensíveis;
* não expor detalhes internos em erros;
* evitar logs com credenciais, tokens e cookies;
* manter TLS e certificados bem configurados.

## Checklist de revisão

Use este checklist ao revisar uma aplicação web.

### HTTP

* Os métodos HTTP representam corretamente a intenção da operação?
* Operações de leitura usam `GET`?
* Operações que alteram estado evitam `GET`?
* Dados sensíveis ficam fora da URL?
* Códigos de status são usados corretamente?

### DNS e domínio

* O domínio aponta para o ambiente correto?
* Subdomínios antigos foram removidos ou protegidos?
* O certificado cobre o domínio correto?
* Ambientes de teste não expõem dados reais?
* Registros DNS estão documentados?

### Sessão

* Cookies de sessão usam `HttpOnly`?
* Cookies de sessão usam `Secure`?
* A política `SameSite` foi avaliada?
* A sessão expira após tempo adequado?
* O logout invalida a sessão?
* A aplicação evita armazenar dados sensíveis desnecessários na sessão?

### HTTPS/TLS

* A aplicação usa HTTPS em produção?
* HTTP redireciona para HTTPS?
* Certificados são válidos e monitorados?
* Protocolos antigos foram desativados?
* HSTS foi avaliado?
* Cookies sensíveis só trafegam por HTTPS?

### Logs

* Senhas não são registradas?
* Tokens não são registrados?
* Cookies não são registrados?
* Dados pessoais são mascarados quando possível?
* Logs possuem informação suficiente para investigação sem expor dados sensíveis?

## Resumo

HTTP, DNS, sessões e HTTPS são fundamentos essenciais para entender segurança web.

O HTTP permite a comunicação entre cliente e servidor, mas é sem estado. Por isso, aplicações usam sessões e cookies para manter contexto entre requisições. O DNS permite que nomes de domínio sejam traduzidos para endereços IP. O HTTPS, por meio do TLS, protege a comunicação contra leitura e alteração indevida durante o tráfego.

Os principais pontos deste artigo foram:

* HTTP funciona no modelo cliente-servidor;
* HTTP é sem estado;
* `GET` deve ser usado para leitura;
* `POST` é usado para envio de dados e criação de recursos;
* dados sensíveis não devem ir na URL;
* DNS associa nomes de domínio a endereços IP;
* DNS direto descobre o IP a partir do domínio;
* DNS reverso descobre o domínio a partir do IP;
* sessões permitem manter contexto entre requisições;
* cookies de sessão devem ser protegidos;
* `HttpOnly`, `Secure` e `SameSite` são atributos importantes;
* HTTPS usa TLS para proteger a comunicação;
* HTTPS é essencial, mas não substitui validação, autenticação, autorização e boas práticas de código.

Esses conceitos serão fundamentais para entender os próximos temas: SQL Injection, Cross-Site Scripting, DAST, conformidade e segurança no ciclo de desenvolvimento.

## Perguntas de revisão

### 1. O que é HTTP?

HTTP é o protocolo usado para troca de dados na web. Ele permite que clientes, como navegadores, façam requisições para servidores e recebam respostas.

### 2. O que significa dizer que HTTP é sem estado?

Significa que cada requisição é independente. O servidor não lembra automaticamente das requisições anteriores apenas por causa do protocolo HTTP.

### 3. Por que sessões são necessárias em aplicações web?

Porque aplicações precisam reconhecer o mesmo usuário entre várias requisições, especialmente após login. Como HTTP é sem estado, sessões ajudam a manter esse contexto.

### 4. Qual é a diferença entre GET e POST?

`GET` é usado para solicitar dados de um recurso. `POST` é usado para enviar dados ao servidor, criar recursos ou iniciar operações.

### 5. GET é mais seguro que POST?

Não. Na verdade, dados em `GET` aparecem na URL e podem ser mais facilmente expostos em histórico e logs. Porém, `POST` também não é seguro sozinho. A segurança depende de HTTPS, validação, autenticação, autorização e configuração adequada.

### 6. Por que não devemos enviar senhas pela URL?

Porque URLs podem aparecer em histórico do navegador, logs, ferramentas de monitoramento, proxies e outros registros intermediários.

### 7. O que é DNS?

DNS é o sistema que associa nomes de domínio a endereços IP. Ele funciona como uma lista telefônica da internet.

### 8. Qual é a diferença entre DNS direto e DNS reverso?

DNS direto descobre o endereço IP a partir de um domínio. DNS reverso descobre o nome de domínio a partir de um endereço IP.

### 9. O que é HTTPS?

HTTPS é a versão segura do HTTP. Ele usa TLS para criptografar a comunicação, proteger a integridade dos dados e ajudar a autenticar o servidor.

### 10. HTTPS resolve todos os problemas de segurança?

Não. HTTPS protege dados em trânsito, mas não corrige falhas como SQL Injection, XSS, autorização incorreta, logs inseguros ou dependências vulneráveis.

### 11. O que faz o atributo HttpOnly em um cookie?

`HttpOnly` impede que o cookie seja acessado diretamente por JavaScript no navegador.

### 12. O que faz o atributo Secure em um cookie?

`Secure` instrui o navegador a enviar o cookie apenas por conexões HTTPS.

### 13. O que faz o atributo SameSite em um cookie?

`SameSite` ajuda a controlar quando cookies são enviados em requisições cross-site, reduzindo certos riscos associados ao envio automático de cookies.

### 14. O que deve acontecer no logout?

A aplicação deve encerrar a sessão, limpar o contexto de autenticação quando aplicável e remover ou invalidar cookies de sessão.

### 15. Por que logs podem ser um risco de segurança?

Porque logs podem armazenar dados sensíveis, como senhas, tokens, cookies e informações pessoais, caso a aplicação registre dados sem cuidado.

## Referências

* MDN Web Docs — Overview of HTTP.
* MDN Web Docs — HTTP.
* IETF — RFC 9110: HTTP Semantics.
* OWASP Cheat Sheet Series — Session Management Cheat Sheet.
* OWASP Cheat Sheet Series — Transport Layer Security Cheat Sheet.
* Oracle — JDK 25 Documentation.
* Oracle — Java SE Downloads.
* Spring Boot Reference Documentation.
* Spring Security Reference Documentation.

[1]: https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/Overview "Overview of HTTP - MDN Web Docs"
[2]: https://developer.mozilla.org/en-US/docs/Web/HTTP "HTTP: Hypertext Transfer Protocol - MDN Web Docs"
