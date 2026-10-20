# Cross-Site Scripting (XSS): Tipos, Riscos e Prevenção

Cross-Site Scripting, mais conhecido como XSS, é uma vulnerabilidade de segurança em aplicações web que permite que conteúdo não confiável seja executado no navegador do usuário como se fizesse parte legítima da aplicação.

Em termos simples, o problema acontece quando uma aplicação recebe dados externos e depois exibe esses dados em uma página, resposta HTML, template, script ou atributo sem o tratamento correto.

Imagine a aplicação fictícia `meucatalogomusical.com`, um catálogo musical onde usuários podem comentar sobre bandas, avaliar álbuns, montar listas de favoritos e escrever resenhas sobre shows de Metal e Hard Rock.

Uma funcionalidade aparentemente simples, como comentários em uma página da banda Nightwish, já pode abrir espaço para XSS se a aplicação não tratar corretamente o conteúdo enviado pelos usuários.

Por exemplo, o usuário deveria escrever algo como:

```
Esse álbum do Nightwish tem uma atmosfera excelente.
```

A aplicação deveria armazenar esse comentário como texto e exibi-lo como texto.

O problema começa quando a aplicação permite que o conteúdo enviado seja interpretado pelo navegador como HTML, JavaScript ou outro tipo de código executável.

XSS é perigoso porque acontece no navegador da vítima, dentro do contexto de confiança da aplicação acessada.

## O problema real que XSS resolve no estudo de segurança

Estudar XSS ajuda a entender uma regra fundamental de segurança web:

> Toda entrada externa deve ser tratada como dado não confiável, inclusive quando ela será apenas exibida na tela.

Muitos desenvolvedores associam segurança apenas ao banco de dados, autenticação ou backend. Porém, XSS mostra que a camada de apresentação também é uma parte crítica da segurança.

Mesmo que o backend esteja protegido contra SQL Injection, a aplicação ainda pode ser vulnerável se exibir dados não confiáveis de forma insegura.

A pergunta central é:

> Quando eu mostro uma informação na página, o navegador vai interpretá-la como texto ou como código?

A prevenção contra XSS depende principalmente de garantir que conteúdo não confiável seja tratado como texto comum, e não como instrução executável.

## Antes de entrar nos detalhes: uma ideia simples

Pense em uma página de comentários da banda Blind Guardian.

O usuário envia um comentário.

A aplicação salva no banco.

Depois, outros usuários acessam a página e veem aquele comentário.

Fluxo esperado:

```
Usuário envia comentário -> aplicação salva -> página exibe comentário como texto
```

Fluxo perigoso:

```
Usuário envia conteúdo não confiável -> aplicação salva -> página exibe como HTML/código
```

O erro está em permitir que o navegador interprete o conteúdo enviado pelo usuário como parte da estrutura da página.

Em uma aplicação segura, comentários, nomes de usuário, títulos, descrições e resenhas devem ser exibidos como dados, não como código.

## Definição técnica

Cross-Site Scripting é uma vulnerabilidade em que conteúdo não confiável é inserido em uma página web sem tratamento adequado, permitindo que o navegador execute esse conteúdo no contexto da aplicação.

Esse conteúdo pode chegar por vários caminhos:

* comentários;
* formulários;
* parâmetros de URL;
* campos de perfil;
* mensagens;
* nomes de exibição;
* uploads;
* dados vindos de APIs;
* dados importados;
* conteúdo de integrações;
* dados armazenados no banco;
* mensagens de erro;
* trechos usados em JavaScript dinâmico.

O ponto central é que o navegador não sabe automaticamente qual conteúdo é confiável e qual não é. Se a aplicação entrega uma resposta dizendo que algo é HTML ou JavaScript, o navegador tende a interpretar esse conteúdo conforme o contexto.

## Por que o nome é Cross-Site Scripting?

O nome Cross-Site Scripting vem de ataques históricos em que scripts eram executados em um contexto envolvendo sites diferentes. Com o tempo, o termo passou a ser usado de forma mais ampla para vulnerabilidades em que conteúdo malicioso é injetado e executado no navegador dentro do contexto de uma aplicação confiável.

Na prática, o nome mais importante para o desenvolvedor é XSS.

Quando alguém fala “essa tela tem XSS”, normalmente significa:

> A aplicação está permitindo que entrada não confiável seja executada no navegador do usuário.

## Por que XSS é perigoso?

XSS é perigoso porque o código roda no navegador da vítima, dentro do contexto da aplicação.

Dependendo do cenário, isso pode permitir:

* roubo de informações sensíveis exibidas na página;
* execução de ações em nome do usuário;
* sequestro de sessão;
* alteração visual da página;
* redirecionamento para fluxos enganosos;
* captura de dados digitados;
* manipulação de requisições;
* impersonificação de usuário;
* dano à reputação da aplicação.

Em `meucatalogomusical.com`, uma falha de XSS poderia afetar páginas como:

* comentários de álbuns;
* resenhas de shows;
* perfil do usuário;
* biografia de bandas;
* mensagens entre usuários;
* busca por bandas;
* área administrativa;
* painel de moderação.

Mesmo um campo simples como “nome de exibição” pode ser perigoso se for exibido sem escape.

## XSS explora a confiança do usuário na aplicação

Um usuário confia que a página de `meucatalogomusical.com` está entregando conteúdo legítimo.

Se a aplicação exibe um conteúdo malicioso como se fosse parte da própria página, o navegador executa aquele conteúdo dentro do domínio confiável.

Esse é o problema: o usuário não está necessariamente acessando um site estranho. Ele está acessando a aplicação que conhece.

Por isso, XSS é uma vulnerabilidade de aplicação, não apenas um problema do navegador.

## Tipos principais de XSS

Os tipos mais comuns de XSS são:

* XSS armazenado;
* XSS refletido;
* XSS baseado em DOM.

## XSS armazenado

XSS armazenado acontece quando o conteúdo não confiável é salvo pela aplicação e depois exibido para outros usuários de forma insegura.

Exemplo conceitual:

1. Um usuário envia um comentário.
2. A aplicação salva o comentário no banco de dados.
3. Outro usuário abre a página.
4. A aplicação carrega o comentário salvo.
5. O navegador interpreta o conteúdo como código.

Esse tipo é especialmente perigoso porque o conteúdo fica persistido.

Em `meucatalogomusical.com`, isso poderia acontecer em:

* comentários sobre um álbum do Opeth;
* resenhas de shows do Scorpions;
* biografia colaborativa de uma banda;
* nome de usuário;
* descrição de playlist;
* mensagens privadas;
* avaliações textuais.

O problema não está em permitir comentários. O problema está em exibir comentários sem codificação de saída adequada.

## XSS refletido

XSS refletido acontece quando a entrada enviada na requisição é devolvida imediatamente na resposta da aplicação sem tratamento adequado.

Exemplo conceitual:

```
GET /buscar?termo=Queensryche
```

A página de busca pode exibir:

```
Resultados para: Queensryche
```

Isso é normal.

O risco aparece se o valor de `termo` for incluído na página sem escape.

Nesse caso, a entrada vem na requisição e é refletida na resposta.

Em `meucatalogomusical.com`, esse risco poderia aparecer em:

* página de busca;
* mensagens de erro;
* filtros;
* páginas de resultado;
* parâmetros de redirecionamento;
* telas que mostram “você pesquisou por...”.

XSS refletido normalmente depende de o usuário acessar uma URL ou fluxo específico contendo uma entrada manipulada. Mesmo assim, é uma falha séria.

## XSS baseado em DOM

XSS baseado em DOM acontece quando o próprio JavaScript do frontend manipula dados não confiáveis e insere esses dados no DOM de forma insegura.

DOM significa Document Object Model. É a representação da página HTML em memória no navegador.

Esse tipo de XSS pode ocorrer mesmo quando o backend não devolve diretamente conteúdo inseguro. O problema pode estar no JavaScript do lado do cliente.

Exemplo conceitual inseguro:

```javascript
const params = new URLSearchParams(window.location.search);
const termo = params.get("termo");

document.getElementById("resultado").innerHTML = "Resultados para: " + termo;
```

O risco está no uso de `innerHTML` com dado vindo da URL.

Uma alternativa mais segura, quando a intenção é exibir texto, é usar `textContent`:

```javascript
const params = new URLSearchParams(window.location.search);
const termo = params.get("termo");

document.getElementById("resultado").textContent = "Resultados para: " + termo;
```

A diferença é importante:

* `innerHTML` interpreta o valor como HTML;
* `textContent` trata o valor como texto.

Em aplicações modernas, muito conteúdo é montado no frontend. Por isso, XSS não é apenas uma preocupação do backend.

## O que XSS pode causar?

Os principais riscos de XSS incluem:

* roubo de informações sensíveis;
* atividades não autorizadas;
* sequestro de sessão;
* alteração da página;
* abuso da confiança do usuário;
* exposição de dados privados;
* danos à reputação.

## Roubo de informações sensíveis

Se uma página exibe dados privados, um XSS pode permitir que o conteúdo malicioso interaja com esses dados dentro do navegador.

Em `meucatalogomusical.com`, isso poderia envolver:

* e-mail do usuário;
* dados do perfil;
* listas privadas;
* histórico de avaliações;
* informações administrativas exibidas em tela;
* preferências pessoais.

Mesmo que os dados não sejam extremamente críticos, a exposição indevida ainda é um problema de privacidade e confiança.

## Atividades não autorizadas

Um XSS pode induzir o navegador do usuário a executar ações dentro da aplicação.

Se o usuário estiver autenticado, o navegador pode enviar cookies e credenciais automaticamente, dependendo da configuração da aplicação.

Por isso, XSS pode ser combinado com ações indevidas, como:

* alterar dados de perfil;
* publicar comentários;
* modificar preferências;
* acionar endpoints;
* interagir com funcionalidades que o usuário autenticado pode acessar.

Isso reforça a importância de várias camadas de defesa, como validação, autorização, proteção de sessão, CSRF quando aplicável e Content Security Policy.

## Sequestro de sessão

Sessões web muitas vezes dependem de cookies.

Se cookies de sessão forem mal protegidos, um XSS pode aumentar o risco de comprometimento da sessão.

O atributo `HttpOnly` ajuda porque impede que JavaScript acesse diretamente o cookie. Porém, ele não resolve todos os impactos de XSS. Mesmo com `HttpOnly`, um script executado no contexto da página pode tentar interagir com a aplicação em nome do usuário.

Por isso, `HttpOnly` é uma defesa importante, mas não substitui prevenção contra XSS.

## Alteração visual da página

Um XSS pode alterar o conteúdo exibido para o usuário.

Isso pode ser usado para:

* simular mensagens falsas;
* alterar botões;
* esconder avisos;
* modificar formulários;
* induzir ações;
* prejudicar a credibilidade da aplicação.

Em uma aplicação como `meucatalogomusical.com`, uma página de review poderia ser visualmente adulterada para enganar usuários ou administradores.

## Onde XSS aparece em sistemas reais

XSS pode aparecer em muitos lugares:

* comentários;
* fóruns;
* avaliações;
* chats;
* nomes de usuário;
* descrições de perfil;
* mensagens administrativas;
* dashboards;
* resultados de busca;
* páginas de erro;
* templates de e-mail;
* parâmetros de URL;
* campos importados de planilhas;
* dados vindos de integrações;
* sistemas internos de atendimento;
* painéis de backoffice.

Um erro comum é pensar que XSS só existe em sites públicos. Sistemas internos também podem ser vulneráveis, especialmente quando exibem dados enviados por usuários, clientes, integrações ou arquivos importados.

## Regra principal: codificação de saída contextual

A principal defesa contra XSS é fazer codificação de saída adequada ao contexto.

Isso significa que a aplicação deve transformar caracteres especiais em uma forma segura antes de inserir dados não confiáveis na resposta.

Mas essa codificação depende do local onde o dado será inserido.

Não é a mesma coisa inserir dado em:

* corpo HTML;
* atributo HTML;
* JavaScript;
* CSS;
* URL.

Cada contexto possui regras próprias.

Exemplo de contextos:

```html
<p>Comentário aqui</p>
```

```html
<input value="Comentário aqui">
```

```javascript
const comentario = "Comentário aqui";
```

```css
background-image: url("valor-aqui");
```

```html
<a href="valor-aqui">Link</a>
```

A saída precisa ser codificada conforme o contexto correto.

## Validação de entrada ajuda, mas não basta

Validação de entrada é importante.

Ela responde perguntas como:

* o campo é obrigatório?
* o tamanho máximo foi respeitado?
* o valor faz sentido?
* o formato é esperado?
* o tipo está correto?
* o valor pertence a uma lista permitida?

Por exemplo, um comentário pode ter limite de tamanho:

```java
package com.meucatalogomusical.comentarios.api;

import jakarta.validation.constraints.NotBlank;
import jakarta.validation.constraints.Size;

public record NovoComentarioRequest(

        @NotBlank
        @Size(max = 1000)
        String conteudo
) {
}
```

Essa validação é útil, mas não substitui codificação de saída.

Um texto pode ter tamanho válido e ainda assim conter caracteres que precisam ser escapados ao ser exibido.

Portanto:

* valide na entrada;
* codifique na saída;
* sanitize apenas quando realmente precisar permitir HTML limitado.

## Exemplo inseguro com Thymeleaf

Em aplicações Spring MVC, é comum usar Thymeleaf para renderizar HTML no servidor.

Imagine uma página que exibe comentários sobre um álbum do Rush.

Exemplo perigoso:

```html
<div class="comentario" th:utext="${comentario.conteudo}">
    Comentário do usuário
</div>
```

O atributo `th:utext` renderiza conteúdo sem escape.

Isso pode ser perigoso se `comentario.conteudo` vier de usuário ou de qualquer fonte não confiável.

A aplicação está dizendo ao template:

> Renderize este conteúdo como HTML.

Se o conteúdo não for totalmente confiável e sanitizado, isso pode abrir espaço para XSS.

## Exemplo seguro com Thymeleaf

Se a intenção é exibir o comentário como texto, use escape de saída.

Exemplo mais seguro:

```html
<div class="comentario" th:text="${comentario.conteudo}">
    Comentário do usuário
</div>
```

Nesse caso, o conteúdo é tratado como texto.

Se o usuário escreveu:

```text
Esse show do Rush foi excelente.
```

a página exibe o texto normalmente.

Se o usuário tentar enviar caracteres especiais de HTML, eles devem ser exibidos como texto, não interpretados como estrutura da página.

## Regra prática para templates

Uma boa regra é:

> Use saída escapada por padrão. Só use saída não escapada quando houver uma justificativa forte, sanitização robusta e revisão de segurança.

Em outras palavras:

* prefira `th:text`;
* evite `th:utext` para conteúdo de usuário;
* não renderize HTML livre enviado por usuário;
* se precisar permitir formatação, use uma biblioteca de sanitização confiável;
* revise cuidadosamente qualquer exceção.

## Exemplo de controller em Spring

Vamos criar um fluxo simples para comentários em `meucatalogomusical.com`.

```java
package com.meucatalogomusical.comentarios.api;

import jakarta.validation.Valid;
import org.springframework.http.HttpStatus;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/albuns/{albumId}/comentarios")
public class ComentarioController {

    private final ComentarioService comentarioService;

    public ComentarioController(ComentarioService comentarioService) {
        this.comentarioService = comentarioService;
    }

    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public ComentarioResponse comentar(
            @PathVariable Long albumId,
            @Valid @RequestBody NovoComentarioRequest request
    ) {
        return comentarioService.comentar(albumId, request);
    }
}
```

O DTO de entrada:

```java
package com.meucatalogomusical.comentarios.api;

import jakarta.validation.constraints.NotBlank;
import jakarta.validation.constraints.Size;

public record NovoComentarioRequest(

        @NotBlank
        @Size(max = 1000)
        String conteudo
) {
}
```

O DTO de saída:

```java
package com.meucatalogomusical.comentarios.api;

import java.time.OffsetDateTime;

public record ComentarioResponse(
        Long id,
        String autor,
        String conteudo,
        OffsetDateTime criadoEm
) {
}
```

Esse backend valida tamanho e obrigatoriedade, mas não tenta “executar” nem “interpretar” o comentário.

A responsabilidade de exibir o conteúdo de forma segura continua sendo da camada de apresentação.

## Devemos escapar antes de salvar no banco?

Em geral, não é recomendado salvar o texto já escapado no banco como regra universal.

O ideal costuma ser:

* armazenar o dado original como dado;
* validar regras de domínio;
* codificar/escapar na saída conforme o contexto.

Por quê?

Porque o mesmo dado pode ser exibido em contextos diferentes:

* HTML;
* JSON;
* atributo HTML;
* PDF;
* e-mail;
* log;
* JavaScript;
* mobile app.

Cada contexto pode exigir tratamento diferente.

Se você salva o dado já escapado para HTML, pode acabar criando problemas em outros contextos, como exibição duplicadamente escapada ou tratamento incorreto.

## Exemplo com resposta JSON

APIs REST geralmente retornam JSON.

Exemplo:

```json
{
  "id": 10,
  "autor": "joao",
  "conteudo": "Esse álbum do Van Halen é clássico.",
  "criadoEm": "2026-06-22T10:30:00-03:00"
}
```

Retornar JSON corretamente não significa que o frontend pode jogar `conteudo` diretamente em `innerHTML`.

No frontend, se a intenção é mostrar texto, deve-se usar APIs que tratem o valor como texto.

Exemplo seguro:

```javascript
const elemento = document.getElementById("comentario");
elemento.textContent = comentario.conteudo;
```

Exemplo arriscado:

```javascript
const elemento = document.getElementById("comentario");
elemento.innerHTML = comentario.conteudo;
```

A API pode estar correta, mas o frontend ainda pode introduzir XSS se renderizar o dado de forma insegura.

## Sanitização: quando faz sentido?

Sanitização é o processo de limpar um conteúdo, removendo ou neutralizando partes perigosas.

Ela faz sentido quando a aplicação realmente precisa permitir algum HTML controlado.

Exemplo: permitir que uma resenha use apenas formatação simples, como negrito, itálico e listas.

Mesmo nesse caso, a aplicação não deve aceitar HTML livre. Ela deve usar uma biblioteca confiável de sanitização e permitir apenas um conjunto restrito de tags e atributos.

Regra prática:

* se não precisa permitir HTML, não permita;
* se precisa permitir HTML, sanitize com biblioteca confiável;
* mesmo sanitizando, codifique corretamente no contexto de saída;
* não crie sanitizador manual baseado em substituições simples.

## Por que filtros simples são perigosos?

Um erro comum é tentar bloquear algumas palavras ou caracteres específicos.

Exemplo de ideia ruim:

```java
String conteudoSeguro = conteudo.replace("<script", "");
```

Esse tipo de abordagem é frágil.

XSS possui muitos contextos e variações. Bloquear algumas substrings raramente é suficiente.

A defesa correta não é tentar adivinhar todas as formas perigosas de entrada. A defesa correta é:

* usar saída escapada;
* aplicar codificação contextual;
* evitar APIs perigosas;
* sanitizar corretamente quando HTML for necessário;
* usar Content Security Policy como camada adicional;
* proteger cookies;
* revisar fluxos sensíveis.

## Content Security Policy

Content Security Policy, ou CSP, é um header HTTP que permite declarar quais fontes de conteúdo a página pode carregar e executar.

Ele pode ajudar a mitigar impactos de XSS, restringindo de onde scripts, estilos, imagens e outros recursos podem ser carregados.

Exemplo conceitual:

```http
Content-Security-Policy: default-src 'self'; script-src 'self'
```

Essa política diz, de forma simplificada, que recursos devem vir da própria origem.

CSP é uma camada importante de defesa, mas não deve ser vista como substituto para corrigir XSS na origem.

Se a aplicação renderiza conteúdo de usuário sem escape, a falha continua existindo. O CSP pode reduzir o impacto, mas não justifica manter uma renderização insegura.

## Configurando CSP com Spring Security

Em uma aplicação Spring, é possível configurar headers de segurança no Spring Security.

Exemplo didático:

```java
package com.meucatalogomusical.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.Customizer;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.web.SecurityFilterChain;

@Configuration
public class SecurityConfig {

    @Bean
    SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        return http
                .headers(headers -> headers
                        .contentSecurityPolicy(csp -> csp
                                .policyDirectives("default-src 'self'; script-src 'self'; object-src 'none'; base-uri 'self'")
                        )
                )
                .authorizeHttpRequests(authorize -> authorize
                        .requestMatchers("/", "/bandas/**", "/albuns/**").permitAll()
                        .anyRequest().authenticated()
                )
                .formLogin(Customizer.withDefaults())
                .build();
    }
}
```

Essa configuração é apenas um ponto de partida didático.

Uma política CSP real precisa ser planejada conforme:

* scripts usados pela aplicação;
* CDN;
* estilos;
* imagens;
* fontes;
* integrações;
* analytics;
* ferramentas de observabilidade;
* ambiente de produção;
* necessidades do frontend.

Uma CSP muito permissiva perde valor. Uma CSP muito restritiva pode quebrar a aplicação.

## Cookies e XSS

Cookies de sessão devem usar atributos seguros.

Exemplo de configuração em Spring Boot:

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

Esses atributos ajudam:

* `HttpOnly`: dificulta acesso direto ao cookie por JavaScript;
* `Secure`: envia o cookie apenas por HTTPS;
* `SameSite`: ajuda a controlar envio em contextos cross-site.

Esses atributos são importantes, mas não eliminam XSS.

Eles reduzem impacto em alguns cenários, mas a aplicação ainda precisa impedir que conteúdo não confiável execute no navegador.

## XSS e autorização

XSS não é apenas problema de frontend.

Se um usuário autenticado possui permissões administrativas, um XSS executado no navegador desse usuário pode ter impacto maior.

Por exemplo, imagine um moderador de `meucatalogomusical.com` acessando uma tela de comentários pendentes. Se essa tela renderizar comentários sem escape, um conteúdo malicioso armazenado por outro usuário poderia afetar o navegador do moderador.

Isso mostra por que telas administrativas também precisam de cuidado.

Na verdade, elas precisam de cuidado ainda maior, porque usuários administrativos têm mais permissões.

## XSS e APIs REST

Mesmo quando o backend é apenas uma API REST, ele ainda participa da prevenção contra XSS.

A API deve:

* validar entradas;
* definir tipos corretos de conteúdo;
* evitar retornar HTML quando deveria retornar JSON;
* não misturar dados com scripts;
* não expor mensagens de erro com conteúdo não confiável;
* documentar contratos de dados;
* evitar armazenar HTML livre sem necessidade.

O frontend, por sua vez, deve:

* renderizar dados como texto;
* evitar `innerHTML` com dados não confiáveis;
* usar frameworks corretamente;
* evitar bypass de mecanismos de escape;
* aplicar sanitização quando necessário;
* adotar CSP.

## Frameworks ajudam, mas não fazem milagre

Frameworks modernos ajudam bastante na prevenção de XSS.

Muitos template engines escapam valores por padrão. Muitos frameworks frontend também tratam interpolação de texto de forma segura.

Mas vulnerabilidades aparecem quando o desenvolvedor desativa ou contorna esses mecanismos.

Exemplos de risco:

* usar saída não escapada;
* usar `innerHTML`;
* montar HTML por concatenação;
* permitir HTML livre;
* inserir dados em JavaScript sem codificação adequada;
* usar bibliotecas antigas ou vulneráveis;
* copiar trechos de código sem entender o contexto;
* ignorar alertas de análise de segurança.

A regra é: entenda o padrão seguro do framework e evite APIs de escape bypass sem motivo forte.

## Exemplo de erro em mensagem de busca

Imagine uma tela de busca:

```
GET /buscar?termo=Kiss
```

A página exibe:

```
Resultados para: Kiss
```

No backend, o controller poderia ser:

```java
package com.meucatalogomusical.busca.web;

import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;

@Controller
public class BuscaController {

    @GetMapping("/buscar")
    public String buscar(@RequestParam String termo, Model model) {
        model.addAttribute("termo", termo);
        return "busca/resultados";
    }
}
```

Esse controller apenas coloca o termo no model.

O risco depende de como o template renderiza esse valor.

Template seguro:

```html
<p>Resultados para: <span th:text="${termo}">termo</span></p>
```

Template perigoso:

```html
<p>Resultados para: <span th:utext="${termo}">termo</span></p>
```

O problema não está necessariamente no controller. Está na decisão de saída do template.

## Codificação por contexto

A forma correta de escapar depende do contexto.

### Corpo HTML

Exemplo:

```html
<p th:text="${comentario.conteudo}">Comentário</p>
```

O valor deve ser tratado como texto HTML.

### Atributo HTML

Exemplo:

```html
<input type="text" th:value="${perfil.nomeExibicao}">
```

O valor precisa ser seguro para atributo HTML.

### URL

Exemplo:

```html
<a th:href="@{/bandas/{id}(id=${banda.id})}">Ver banda</a>
```

URLs precisam ser construídas com mecanismos apropriados, não por concatenação manual insegura.

### JavaScript

Inserir dados diretamente dentro de JavaScript exige muito cuidado.

Evite montar scripts com concatenação de dados não confiáveis.

Quando possível, prefira transferir dados por JSON seguro e deixar o framework lidar com serialização e renderização.

## XSS em mensagens de erro

Mensagens de erro também podem gerar XSS.

Exemplo de risco:

```text
Não foi possível buscar por: [termo informado pelo usuário]
```

Se o termo for refletido na página sem escape, a mensagem de erro vira vetor de XSS.

Portanto, até mensagens de erro precisam renderizar dados como texto.

Além disso, mensagens de erro não devem expor detalhes internos da aplicação.

## XSS em logs e painéis internos

Um ponto menos óbvio: logs e painéis administrativos também podem sofrer XSS.

Imagine que a aplicação registre o `User-Agent`, parâmetros de busca ou mensagens de erro. Depois, um painel interno exibe esses logs em HTML.

Se esse painel renderizar os dados sem escape, um valor malicioso registrado no log pode afetar o navegador de um analista, desenvolvedor ou administrador.

Portanto:

* dados em logs também são dados não confiáveis;
* painéis internos também precisam escapar saída;
* backoffices também fazem parte da superfície de ataque.

## Boas práticas

As principais boas práticas contra XSS são:

* escapar saída por padrão;
* usar codificação contextual;
* evitar renderizar HTML livre enviado por usuário;
* usar `textContent` em vez de `innerHTML` quando a intenção for texto;
* preferir `th:text` em vez de `th:utext` para conteúdo não confiável;
* validar entradas no backend;
* limitar tamanho de campos textuais;
* usar listas permitidas quando possível;
* sanitizar com biblioteca confiável quando HTML limitado for realmente necessário;
* configurar CSP como camada adicional;
* proteger cookies com `HttpOnly`, `Secure` e `SameSite`;
* evitar dados sensíveis acessíveis no frontend;
* aplicar autorização corretamente;
* testar telas públicas e administrativas;
* revisar templates e componentes de UI;
* evitar mensagens de erro que reflitam entrada sem escape;
* manter dependências atualizadas.

## Erros comuns

### 1. Achar que XSS é problema apenas do frontend

XSS envolve frontend e backend.

O backend recebe, valida, armazena e devolve dados. O frontend renderiza esses dados.

Ambos precisam trabalhar de forma segura.

### 2. Usar saída não escapada sem necessidade

Em Thymeleaf, usar `th:utext` com conteúdo de usuário é uma decisão perigosa.

Prefira `th:text`.

### 3. Usar `innerHTML` com dados externos

Se a intenção é exibir texto, use `textContent`.

### 4. Confiar apenas em validação de entrada

Validação ajuda, mas não substitui codificação de saída.

### 5. Criar sanitizador manual

Sanitização é difícil. Substituições simples não cobrem todos os contextos.

### 6. Achar que CSP resolve tudo

CSP é uma camada de mitigação. Ela não substitui correção da renderização insegura.

### 7. Esquecer telas administrativas

Backoffice, dashboards, logs e painéis internos também podem ser alvos.

### 8. Exibir mensagens de erro com dados não confiáveis

Toda saída que inclui entrada do usuário precisa ser escapada.

## Decisões de engenharia

### Quando apenas escapar saída é suficiente?

Quando a aplicação só precisa exibir texto.

Exemplos:

* comentários simples;
* nome de usuário;
* título de playlist;
* resenha sem formatação;
* campo de busca;
* descrição curta.

Nesses casos, a melhor decisão costuma ser armazenar o texto e escapar na saída.

### Quando permitir HTML limitado?

Permitir HTML limitado só faz sentido quando existe necessidade real de formatação.

Exemplos:

* editor rico de artigos;
* descrição formatada;
* documentação interna;
* conteúdo editorial controlado.

Mesmo assim, a aplicação deve usar sanitização confiável e permitir apenas tags e atributos seguros.

### Quando usar CSP?

Use CSP como camada adicional em aplicações web.

Ela é especialmente útil para reduzir impacto de injeções de conteúdo, controlar origem de scripts e dificultar execução de recursos inesperados.

Mas CSP deve ser planejada e testada para não quebrar a aplicação.

### Quando usar `HttpOnly`?

Use `HttpOnly` em cookies de sessão sempre que possível.

Cookies de sessão não devem ser acessados por JavaScript.

### Quando usar `innerHTML`?

Evite `innerHTML` com dados não confiáveis.

Use apenas quando houver necessidade real de inserir HTML e quando o conteúdo estiver controlado ou sanitizado adequadamente.

Na maioria dos casos de exibição de texto, `textContent` é a escolha correta.

## Checklist prático

Use este checklist ao revisar uma aplicação contra XSS.

### Entrada

* Campos textuais possuem tamanho máximo?
* Entradas obrigatórias são validadas?
* Valores controlados usam enum ou lista permitida?
* A aplicação evita aceitar HTML livre sem necessidade?
* Uploads e importações são tratados como não confiáveis?

### Saída

* Dados de usuário são escapados ao serem exibidos?
* Templates usam saída escapada por padrão?
* Há uso de saída não escapada?
* Todo uso de saída não escapada foi justificado e revisado?
* Dados em atributos HTML são tratados corretamente?
* Dados inseridos em JavaScript são tratados com cuidado?
* Dados em URLs são construídos com APIs seguras?

### Frontend

* O código evita `innerHTML` com dados externos?
* Usa `textContent` quando a intenção é texto?
* Frameworks frontend são usados sem bypass de escape?
* Componentes de UI tratam dados como texto por padrão?
* Conteúdo vindo da API é renderizado com segurança?

### Sessão e cookies

* Cookies de sessão usam `HttpOnly`?
* Cookies de sessão usam `Secure`?
* `SameSite` foi avaliado?
* A aplicação usa HTTPS?
* Logout invalida sessão corretamente?

### Headers

* Existe CSP?
* A política CSP é restritiva o suficiente?
* A política CSP foi testada?
* Headers de segurança estão configurados?
* Ambientes de produção e homologação foram revisados?

### Operação

* Logs são exibidos com escape em painéis internos?
* Mensagens de erro não refletem entrada sem escape?
* Eventos suspeitos são monitorados?
* Dependências são atualizadas?
* Revisões de código verificam uso de APIs perigosas?

## Exemplo de revisão em `meucatalogomusical.com`

Imagine que a aplicação possui uma tela de comentários sobre álbuns do AC/DC.

Pontos a revisar:

1. O campo de comentário tem limite de tamanho?
2. O backend valida a entrada?
3. O comentário é salvo como texto?
4. O template usa `th:text`?
5. Não há `th:utext` para comentário?
6. O frontend não usa `innerHTML`?
7. Cookies usam `HttpOnly` e `Secure`?
8. Existe CSP em produção?
9. A tela administrativa de moderação também escapa comentários?
10. Logs e painéis internos também tratam o comentário como texto?

Se todas essas respostas forem positivas, a aplicação está muito mais bem protegida contra XSS.

## Resumo final

Cross-Site Scripting é uma vulnerabilidade em que conteúdo não confiável é executado no navegador do usuário dentro do contexto de uma aplicação confiável.

Os principais tipos são:

* XSS armazenado;
* XSS refletido;
* XSS baseado em DOM.

Os principais riscos são:

* roubo de informações sensíveis;
* atividades não autorizadas;
* sequestro de sessão;
* alteração visual da página;
* abuso da confiança do usuário;
* impacto reputacional.

A defesa principal é a codificação de saída contextual.

Além disso, aplicações seguras devem combinar:

* validação de entrada;
* escape na saída;
* uso seguro de templates;
* cuidado com JavaScript no frontend;
* sanitização confiável quando HTML limitado for necessário;
* CSP;
* cookies seguros;
* autorização;
* revisão de telas públicas e administrativas.

A regra mais importante é:

> Se veio de fora da aplicação, trate como dado não confiável. Se vai para a tela, escape conforme o contexto.

## Perguntas de revisão

### 1. O que é XSS?

XSS é uma vulnerabilidade em que conteúdo não confiável é inserido em uma página web sem tratamento adequado, permitindo que o navegador execute esse conteúdo no contexto da aplicação.

### 2. Por que XSS é perigoso?

Porque o conteúdo executa no navegador da vítima, dentro do contexto de uma aplicação confiável. Isso pode permitir roubo de dados, ações indevidas, sequestro de sessão e alteração da página.

### 3. O que é XSS armazenado?

É o XSS em que o conteúdo malicioso é salvo pela aplicação, por exemplo em banco de dados, e depois exibido para outros usuários de forma insegura.

### 4. O que é XSS refletido?

É o XSS em que a entrada enviada em uma requisição é devolvida imediatamente na resposta da aplicação sem tratamento adequado.

### 5. O que é XSS baseado em DOM?

É o XSS que ocorre quando o JavaScript do frontend manipula dados não confiáveis e os insere no DOM de forma insegura.

### 6. Validação de entrada resolve XSS sozinha?

Não. Validação ajuda, mas a principal defesa contra XSS é a codificação de saída contextual.

### 7. O que é codificação de saída contextual?

É escapar dados de acordo com o local onde eles serão inseridos, como corpo HTML, atributo HTML, JavaScript, CSS ou URL.

### 8. Por que `innerHTML` pode ser perigoso?

Porque `innerHTML` interpreta o valor como HTML. Se o valor vier de fonte não confiável, pode permitir execução de conteúdo indesejado.

### 9. Qual é a alternativa mais segura ao `innerHTML` quando a intenção é exibir texto?

A alternativa é usar `textContent`, pois ele trata o valor como texto.

### 10. Em Thymeleaf, qual é a diferença entre `th:text` e `th:utext`?

`th:text` exibe o valor com escape. `th:utext` exibe o valor sem escape. Para conteúdo não confiável, `th:text` é a opção mais segura.

### 11. CSP resolve XSS completamente?

Não. CSP é uma camada adicional de mitigação. A aplicação ainda precisa corrigir renderizações inseguras e aplicar escape adequado.

### 12. Por que `HttpOnly` é importante?

Porque impede que JavaScript acesse diretamente o cookie. Isso reduz o impacto de alguns cenários de XSS envolvendo cookies de sessão.

### 13. Telas administrativas também podem ter XSS?

Sim. Backoffices, dashboards e painéis internos também podem exibir dados não confiáveis e precisam aplicar escape de saída.

### 14. Devemos salvar texto já escapado no banco?

Em geral, não. O ideal é salvar o dado original e aplicar escape na saída conforme o contexto em que ele será exibido.

### 15. Quando sanitização faz sentido?

Sanitização faz sentido quando a aplicação realmente precisa permitir HTML limitado. Mesmo assim, deve ser feita com biblioteca confiável e política restritiva.

## Referências

* OWASP Cross-Site Scripting Prevention Cheat Sheet.
* OWASP DOM Based XSS Prevention Cheat Sheet.
* OWASP Cross Site Scripting.
* MDN Web Docs — Cross-site scripting.
* MDN Web Docs — Content Security Policy.
* OWASP Content Security Policy Cheat Sheet.
* OWASP Session Management Cheat Sheet.
* Spring Security Reference — Security HTTP Response Headers.
* Thymeleaf Documentation — Using Thymeleaf.
* Oracle JDK 25 Documentation.
* Spring Boot Reference Documentation.

Para os exemplos, mantive Java 25 como LTS atual e Spring Boot 4.x/Spring Framework 7.x como base moderna compatível para o ecossistema Spring; a Oracle lista o JDK 25 como LTS mais recente e a documentação do Spring Boot informa compatibilidade da linha 4.1 com Java até 26 e Spring Framework 7.0.8 ou superior. ([oracle.com][2])

[1]: https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html "Cross Site Scripting Prevention Cheat Sheet"
[2]: https://www.oracle.com/br/java/technologies/downloads/ "Java Downloads | Oracle Brasil"
