# Introdução à Segurança de Aplicações Web

Aplicações web estão presentes em praticamente todos os tipos de negócio: bancos, e-commerces, sistemas internos, plataformas de ensino, catálogos, portais públicos, redes sociais e APIs consumidas por aplicativos móveis.

Quando uma aplicação fica disponível na web, ela deixa de ser apenas um software executando em um servidor. Ela passa a ser um ponto de entrada para usuários legítimos, integrações, bots, crawlers, ferramentas automatizadas e também invasores tentando encontrar falhas.

Segurança de aplicações web é o conjunto de práticas, processos, controles, validações, decisões de arquitetura e mecanismos técnicos usados para proteger uma aplicação contra ameaças. Isso envolve pensar em segurança desde a concepção do sistema, durante o desenvolvimento, nos testes, na implantação e também na operação em produção.

Em outras palavras, segurança não é uma etapa isolada no final do projeto. Segurança precisa fazer parte do ciclo de vida da aplicação.

Imagine a aplicação fictícia `meucatalogomusical.com`, um sistema onde usuários podem cadastrar bandas de Metal e Hard Rock, montar listas de álbuns favoritos, escrever comentários e avaliar shows. Mesmo sendo uma aplicação aparentemente simples, ela já possui vários pontos sensíveis:

* cadastro de usuários;
* autenticação;
* formulários;
* comentários;
* busca por bandas;
* upload de imagens;
* armazenamento de dados;
* APIs;
* logs;
* cookies de sessão;
* integração com banco de dados;
* configuração de servidor;
* deploy em ambiente cloud.

Cada um desses pontos pode se tornar uma via de ataque se não for tratado corretamente.

## O que é segurança de aplicações web?

Segurança de aplicações web é uma área da segurança da informação focada em proteger aplicações acessadas pela web contra uso indevido, vazamento de dados, alteração indevida de informações, interrupção de serviço e acesso não autorizado.

Ela envolve controles como:

* validação de entradas;
* autenticação;
* autorização;
* proteção de sessão;
* criptografia;
* tratamento seguro de erros;
* logs e auditoria;
* atualização de dependências;
* configuração segura;
* testes de segurança;
* monitoramento;
* resposta a incidentes.

Uma aplicação web segura não é aquela que “nunca será atacada”. Isso seria uma promessa irreal. Uma aplicação web segura é aquela que reduz a superfície de ataque, dificulta explorações, limita danos quando algo falha e permite detectar e responder rapidamente a comportamentos suspeitos.

## Segurança de aplicação web não é a mesma coisa que segurança de rede

É comum confundir segurança de rede com segurança de aplicação, mas elas não são a mesma coisa.

A segurança de rede se preocupa com elementos como:

* firewall;
* segmentação de rede;
* VPN;
* controle de portas;
* tráfego entre servidores;
* regras de entrada e saída;
* proteção da infraestrutura.

A segurança de aplicações web se preocupa com o comportamento da aplicação em si:

* o que acontece quando o usuário envia dados inválidos;
* como a aplicação valida permissões;
* como os dados são exibidos em páginas HTML;
* como consultas ao banco são montadas;
* como sessões são protegidas;
* como erros são tratados;
* como arquivos enviados são processados;
* como APIs validam entrada e autorização.

Um firewall pode bloquear tráfego indevido, mas ele não entende completamente a regra de negócio de `meucatalogomusical.com`.

Por exemplo, se um usuário comum tentar acessar uma funcionalidade administrativa para excluir a banda “Black Sabbath” do catálogo, isso é um problema de autorização da aplicação, não apenas de rede.

## Por que aplicações web são alvos tão comuns?

Aplicações web são muito visadas porque normalmente ficam acessíveis pela internet, recebem dados de usuários e estão conectadas a bancos de dados ou serviços internos.

Três características tornam aplicações web especialmente sensíveis:

1. natureza global;
2. grande superfície de ataque;
3. múltiplas camadas e complexidade.

## Natureza global

Uma aplicação web pública pode ser acessada de qualquer lugar do mundo. Isso é uma grande vantagem para o negócio, mas também aumenta o risco.

Se `meucatalogomusical.com` estiver disponível na internet, qualquer pessoa pode tentar acessar suas páginas, APIs, formulários, parâmetros de URL e endpoints públicos.

Isso significa que a aplicação não será usada apenas por usuários bem-intencionados. Ela também poderá receber:

* requisições automatizadas;
* tentativas de enumeração;
* acessos indevidos;
* entradas malformadas;
* bots;
* scanners;
* tentativas de exploração.

A natureza global da web aumenta a responsabilidade dos desenvolvedores. Não basta funcionar na máquina local. A aplicação precisa resistir a uso inesperado e malicioso.

## Grande superfície de ataque

Superfície de ataque é o conjunto de pontos pelos quais uma aplicação pode ser acessada, manipulada ou explorada.

Em uma aplicação web, a superfície de ataque pode incluir:

* parâmetros de URL;
* formulários HTML;
* campos ocultos;
* cabeçalhos HTTP;
* cookies;
* corpo de requisições JSON;
* upload de arquivos;
* endpoints REST;
* páginas públicas;
* páginas administrativas;
* integrações com terceiros;
* mensagens em filas;
* logs;
* banco de dados;
* cache;
* configurações;
* dependências externas.

No caso de `meucatalogomusical.com`, pense em uma tela de busca:

```text
GET /bandas?nome=Metallica
```

Esse parâmetro `nome` é um vetor de entrada. Mesmo sendo apenas uma busca, a aplicação precisa tratar esse valor como dado não confiável.

O mesmo vale para um formulário de cadastro:

```text
POST /bandas
```

Com um corpo JSON como:

```json
{
  "nome": "Iron Maiden",
  "genero": "Heavy Metal",
  "anoFormacao": 1975
}
```

Mesmo que os dados pareçam simples, eles vieram de fora da aplicação. Portanto, precisam ser validados antes de serem usados.

## Múltiplas camadas e complexidade

Aplicações modernas raramente são compostas por uma única camada. Normalmente, temos várias partes trabalhando juntas:

* frontend;
* backend;
* banco de dados;
* cache;
* API Gateway;
* autenticação;
* autorização;
* mensageria;
* storage;
* CDN;
* logs;
* observabilidade;
* serviços externos;
* infraestrutura cloud.

Cada camada adiciona responsabilidades e também possíveis falhas.

Em uma aplicação Spring, por exemplo, uma requisição pode passar por:

1. navegador;
2. DNS;
3. HTTPS/TLS;
4. servidor ou balanceador;
5. filtros de segurança;
6. controller;
7. service;
8. repository;
9. banco de dados;
10. resposta HTTP;
11. renderização no frontend.

Se qualquer uma dessas etapas for mal configurada, a aplicação pode ficar vulnerável.

Complexidade não é ruim por si só. O problema é quando a complexidade não é entendida, documentada, testada e protegida.

## Segurança precisa começar no início

Um erro comum é pensar em segurança apenas no final do desenvolvimento, quando a aplicação já está pronta para ir para produção.

Esse modelo é perigoso porque muitas decisões inseguras já podem ter sido incorporadas ao projeto:

* dados sensíveis expostos em logs;
* permissões mal definidas;
* ausência de validação;
* consultas ao banco montadas de forma insegura;
* endpoints administrativos sem proteção adequada;
* configurações diferentes entre ambientes;
* dependências vulneráveis;
* tratamento genérico de erros;
* ausência de auditoria.

Quanto mais tarde uma falha de segurança é encontrada, mais caro tende a ser corrigi-la.

Por isso, segurança deve fazer parte do ciclo de desenvolvimento:

1. **Requisitos**: identificar dados sensíveis, perfis de acesso e riscos.
2. **Design**: definir autenticação, autorização, segregação de responsabilidades e fluxos seguros.
3. **Implementação**: validar entradas, usar bibliotecas confiáveis e evitar padrões inseguros.
4. **Testes**: incluir testes automatizados, revisão de código e testes de segurança.
5. **Deploy**: aplicar configurações seguras, HTTPS, secrets protegidos e menor privilégio.
6. **Operação**: monitorar logs, métricas, eventos de segurança e dependências vulneráveis.

## Princípios fundamentais: CI4A

A segurança de aplicações web pode ser entendida a partir de alguns princípios fundamentais. Esses princípios aparecem como CI4A:

* Confidencialidade;
* Integridade;
* Disponibilidade;
* Autenticação;
* Autorização;
* Responsabilidade.

Esses conceitos ajudam a organizar o pensamento de segurança.

## Confidencialidade

Confidencialidade significa garantir que dados sensíveis sejam acessados apenas por quem tem permissão.

Em `meucatalogomusical.com`, exemplos de dados que exigem confidencialidade podem incluir:

* e-mail do usuário;
* dados de login;
* tokens;
* preferências privadas;
* dados administrativos;
* logs sensíveis;
* informações internas da aplicação.

Uma falha de confidencialidade ocorre quando dados privados são expostos para usuários não autorizados.

Exemplo: um usuário comum conseguir visualizar os dados de outro usuário por alterar um parâmetro na URL.

## Integridade

Integridade significa garantir que os dados não sejam modificados indevidamente.

Em `meucatalogomusical.com`, a integridade é importante para evitar que alguém altere dados sem autorização, como:

* trocar o nome de uma banda;
* excluir avaliações;
* modificar notas;
* alterar permissões;
* adulterar comentários;
* manipular listas de favoritos de outro usuário.

A integridade protege a confiança nos dados.

## Disponibilidade

Disponibilidade significa garantir que a aplicação esteja acessível quando necessário.

Uma aplicação indisponível pode gerar prejuízo, perda de confiança e impacto operacional.

Disponibilidade envolve:

* monitoramento;
* tratamento de erros;
* resiliência;
* backup;
* rate limiting;
* proteção contra abuso;
* capacidade de recuperação.

Se `meucatalogomusical.com` ficar fora do ar durante o lançamento de uma votação de “melhor álbum do ano”, o problema não é apenas técnico. Também é um problema de experiência e confiança.

## Autenticação

Autenticação é o processo de verificar quem é o usuário.

Normalmente, isso envolve:

* login e senha;
* tokens;
* autenticação multifator;
* provedores externos;
* certificados;
* chaves de API.

A pergunta principal da autenticação é:

> Quem é você?

Em uma aplicação web, autenticar corretamente é essencial para diferenciar usuários anônimos, usuários logados, moderadores e administradores.

## Autorização

Autorização é o processo de verificar o que um usuário autenticado pode fazer.

A pergunta principal da autorização é:

> O que você tem permissão para fazer?

Um usuário pode estar autenticado e, ainda assim, não ter autorização para determinada ação.

Por exemplo:

* um usuário comum pode avaliar álbuns;
* um moderador pode remover comentários ofensivos;
* um administrador pode cadastrar novas bandas;
* apenas um administrador pode alterar permissões de outros usuários.

Autenticação e autorização são conceitos diferentes. Fazer login não significa ter acesso a tudo.

## Responsabilidade

Responsabilidade, ou accountability, significa conseguir rastrear ações até usuários, processos ou sistemas.

Isso envolve:

* logs de auditoria;
* registro de eventos importantes;
* identificação de quem realizou determinada ação;
* horário da ação;
* origem da requisição;
* contexto suficiente para investigação.

Em `meucatalogomusical.com`, ações sensíveis poderiam gerar logs de auditoria, como:

* criação de banda;
* exclusão de álbum;
* alteração de permissões;
* bloqueio de usuário;
* tentativa de acesso negado.

Responsabilidade não significa registrar tudo indiscriminadamente. Logs também precisam ser seguros, úteis e não devem expor dados sensíveis desnecessariamente.

## Vetores de entrada

Um vetor de entrada é qualquer caminho pelo qual dados externos entram na aplicação.

Exemplos comuns:

* parâmetros de URL;
* formulários HTML;
* corpo JSON;
* cookies;
* headers;
* uploads;
* campos ocultos;
* parâmetros de rota;
* webhooks;
* mensagens de filas;
* dados vindos de APIs externas.

Todo vetor de entrada deve ser tratado como não confiável.

Isso não significa que todo usuário é malicioso. Significa apenas que a aplicação não deve depender da boa intenção do usuário para permanecer segura.

## Exemplo em Java e Spring: validação de entrada

Em uma aplicação Spring Boot moderna, usando Java LTS, podemos representar a entrada de cadastro de banda com um `record` e Bean Validation.

```java
package com.meucatalogomusical.bandas.api;

import jakarta.validation.constraints.Max;
import jakarta.validation.constraints.Min;
import jakarta.validation.constraints.NotBlank;
import jakarta.validation.constraints.Size;

public record CadastroBandaRequest(

        @NotBlank
        @Size(max = 120)
        String nome,

        @NotBlank
        @Size(max = 80)
        String genero,

        @Min(1950)
        @Max(2100)
        int anoFormacao
) {
}
```

Esse código não resolve todos os problemas de segurança, mas já estabelece um ponto importante: a aplicação não aceita qualquer entrada sem validação.

Agora podemos usar esse objeto em um controller:

```java
package com.meucatalogomusical.bandas.api;

import jakarta.validation.Valid;
import org.springframework.http.HttpStatus;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/bandas")
public class BandaController {

    private final CadastroBandaService cadastroBandaService;

    public BandaController(CadastroBandaService cadastroBandaService) {
        this.cadastroBandaService = cadastroBandaService;
    }

    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public BandaResponse cadastrar(@Valid @RequestBody CadastroBandaRequest request) {
        return cadastroBandaService.cadastrar(request);
    }
}
```

A anotação `@Valid` faz com que as regras definidas no `CadastroBandaRequest` sejam avaliadas antes de a requisição seguir para a regra de negócio.

Se alguém tentar cadastrar uma banda sem nome, com nome excessivamente grande ou com ano inválido, a entrada será recusada antes de chegar ao serviço.

## Exemplo de modelagem defensiva com enum

Quando um campo possui valores controlados, é melhor evitar texto livre.

Em vez de aceitar qualquer string para o gênero musical, poderíamos criar um enum:

```java
package com.meucatalogomusical.bandas.domain;

public enum GeneroMusical {
    HEAVY_METAL,
    THRASH_METAL,
    POWER_METAL,
    PROGRESSIVE_METAL,
    HARD_ROCK,
    CLASSIC_ROCK
}
```

E usar esse enum na entrada:

```java
package com.meucatalogomusical.bandas.api;

import com.meucatalogomusical.bandas.domain.GeneroMusical;
import jakarta.validation.constraints.Max;
import jakarta.validation.constraints.Min;
import jakarta.validation.constraints.NotBlank;
import jakarta.validation.constraints.NotNull;
import jakarta.validation.constraints.Size;

public record CadastroBandaRequest(

        @NotBlank
        @Size(max = 120)
        String nome,

        @NotNull
        GeneroMusical genero,

        @Min(1950)
        @Max(2100)
        int anoFormacao
) {
}
```

Com isso, a aplicação deixa claro quais valores são esperados.

Isso reduz ambiguidade, melhora a regra de negócio e diminui o risco de entradas inesperadas circularem pela aplicação.

## O que esse exemplo ensina?

Esse exemplo mostra um princípio essencial:

> Segurança começa tratando entrada externa como dado não confiável.

Validação de entrada não é apenas uma questão de “evitar erro de tela”. É uma prática importante para reduzir comportamentos inesperados e proteger as camadas internas da aplicação.

Mas validação não deve ser vista como única proteção. Uma aplicação segura combina várias camadas de defesa.

## Defesa em profundidade

Defesa em profundidade é a ideia de usar múltiplas camadas de proteção.

Em vez de confiar em um único mecanismo, a aplicação aplica controles em diferentes pontos.

Por exemplo:

* o frontend pode orientar o usuário;
* o backend valida de verdade;
* o banco aplica constraints;
* a autenticação identifica o usuário;
* a autorização limita ações;
* os logs registram eventos relevantes;
* o monitoramento detecta comportamento anormal;
* as dependências são atualizadas;
* o deploy usa configuração segura;
* os secrets ficam protegidos;
* o tráfego usa HTTPS.

Se uma camada falhar, outra pode reduzir o impacto.

## Erro comum: confiar apenas no frontend

Validações no frontend são úteis para melhorar a experiência do usuário, mas não são suficientes para segurança.

Qualquer pessoa pode enviar requisições diretamente para a API, sem usar a tela oficial da aplicação.

Por isso, regras importantes precisam existir no backend.

Exemplo:

```java
@PostMapping
@ResponseStatus(HttpStatus.CREATED)
public BandaResponse cadastrar(@Valid @RequestBody CadastroBandaRequest request) {
    return cadastroBandaService.cadastrar(request);
}
```

Mesmo que a tela valide o formulário, o backend continua validando a entrada.

Essa duplicidade é intencional:

* frontend melhora usabilidade;
* backend protege a aplicação.

## Erro comum: segurança apenas como ferramenta

Ferramentas são importantes, mas não substituem boas decisões de engenharia.

Ferramentas podem ajudar com:

* análise estática de código;
* análise de dependências;
* testes dinâmicos;
* verificação de configuração;
* monitoramento;
* detecção de vulnerabilidades conhecidas.

Mas elas não entendem completamente a regra de negócio.

Uma ferramenta pode apontar uma dependência vulnerável, mas talvez não saiba dizer se um usuário comum deveria ou não poder aprovar uma alteração administrativa em `meucatalogomusical.com`.

Segurança depende de ferramenta, processo e conhecimento do domínio.

## Erro comum: tratar segurança como responsabilidade de outro time

Outro erro comum é pensar que segurança é responsabilidade exclusiva do time de segurança.

Em aplicações modernas, segurança é responsabilidade compartilhada.

O time de segurança pode definir padrões, revisar riscos, apoiar testes e orientar boas práticas. Mas os desenvolvedores tomam decisões todos os dias que impactam diretamente a segurança:

* como validar entrada;
* como montar consultas;
* como tratar erros;
* como organizar permissões;
* como registrar logs;
* como consumir APIs externas;
* como configurar endpoints;
* como lidar com dados sensíveis.

Desenvolvedores não precisam se tornar especialistas ofensivos em segurança, mas precisam saber construir software de forma defensiva.

## Segurança e requisitos de negócio

Segurança não deve ser vista apenas como uma obrigação técnica. Ela também protege o negócio.

Uma falha de segurança pode causar:

* vazamento de dados;
* perda de confiança;
* indisponibilidade;
* impacto financeiro;
* multas regulatórias;
* dano à reputação;
* retrabalho técnico;
* perda de clientes;
* interrupção operacional.

No caso de `meucatalogomusical.com`, um vazamento de dados poderia expor e-mails, senhas mal protegidas, listas privadas de usuários ou dados internos de administração.

Mesmo que a aplicação seja fictícia e pareça simples, os princípios são os mesmos de aplicações corporativas reais.

## Segurança no ciclo de desenvolvimento

Um fluxo saudável de desenvolvimento seguro pode incluir:

1. mapear dados sensíveis;
2. definir papéis e permissões;
3. validar entradas no backend;
4. usar consultas parametrizadas;
5. proteger sessões;
6. aplicar HTTPS;
7. revisar logs para evitar exposição de dados;
8. atualizar dependências;
9. usar ferramentas de análise;
10. revisar configurações;
11. criar testes automatizados;
12. monitorar produção;
13. responder a incidentes.

A segurança precisa acompanhar a velocidade do desenvolvimento.

Com práticas de entrega contínua, código pode ser publicado semanalmente, diariamente ou até várias vezes ao dia. Portanto, os controles de segurança também precisam se adaptar a essa velocidade.

## Relação com OWASP

OWASP é uma das principais referências abertas sobre segurança de aplicações web.

Entre os materiais mais conhecidos estão:

* OWASP Top 10;
* OWASP ASVS;
* OWASP Web Security Testing Guide;
* OWASP Cheat Sheet Series.

O OWASP Top 10 ajuda na conscientização sobre os riscos mais críticos. O ASVS ajuda a transformar segurança em requisitos verificáveis. O WSTG ajuda a estruturar testes de segurança.

Esses materiais são úteis porque ajudam a transformar segurança em práticas concretas de desenvolvimento.

## Boas práticas iniciais

Algumas boas práticas que todo desenvolvedor web deve internalizar:

* nunca confiar cegamente em entrada do usuário;
* validar dados no backend;
* aplicar autorização em ações sensíveis;
* usar HTTPS;
* proteger cookies e sessões;
* evitar expor detalhes internos em mensagens de erro;
* não registrar dados sensíveis em logs;
* manter dependências atualizadas;
* aplicar o princípio do menor privilégio;
* usar bibliotecas confiáveis;
* revisar configurações de produção;
* criar testes automatizados para regras de segurança;
* monitorar comportamento suspeito.

Essas práticas não tornam a aplicação invulnerável, mas reduzem muito o risco.

## Checklist para revisar uma aplicação web

Use este checklist como ponto de partida:

### Entrada de dados

* Os parâmetros de URL são validados?
* Os dados de formulários são validados no backend?
* O corpo JSON das requisições é validado?
* Campos opcionais possuem limites?
* Strings possuem tamanho máximo?
* Valores controlados usam enum ou lista permitida?
* Uploads são tratados com cuidado?

### Autenticação e sessão

* O login usa mecanismo confiável?
* As senhas são armazenadas corretamente?
* Existe proteção contra sessão roubada?
* Cookies sensíveis usam atributos seguros?
* Tokens possuem expiração?
* Logout invalida sessão ou token quando aplicável?

### Autorização

* Cada ação sensível valida permissão?
* Usuários comuns não acessam recursos administrativos?
* A autorização é validada no backend?
* Existe separação clara entre perfis?

### Dados sensíveis

* Dados sensíveis são criptografados quando necessário?
* Logs evitam expor tokens, senhas e dados pessoais?
* Respostas de erro evitam detalhes internos?
* Backups são protegidos?

### Dependências e configuração

* Dependências são atualizadas?
* Secrets ficam fora do código-fonte?
* Ambientes possuem configurações adequadas?
* O modo debug está desativado em produção?
* Headers de segurança são configurados?

### Operação

* Existem logs úteis?
* Eventos sensíveis são auditados?
* Há monitoramento?
* Alertas importantes são tratados?
* Existe plano de resposta a incidentes?

## Resumo

Segurança de aplicações web é uma disciplina essencial para qualquer desenvolvedor que constrói sistemas acessados pela internet.

Aplicações web são expostas por natureza, possuem grande superfície de ataque e normalmente envolvem múltiplas camadas. Por isso, segurança precisa ser pensada desde o início do desenvolvimento, não apenas no final.

Os principais conceitos deste artigo foram:

* segurança de aplicação web é diferente de segurança de rede;
* aplicações web possuem natureza global;
* toda entrada externa deve ser tratada como não confiável;
* superfície de ataque inclui URLs, formulários, cookies, headers, APIs e uploads;
* múltiplas camadas aumentam a complexidade;
* CI4A ajuda a organizar os objetivos de segurança;
* validação no backend é indispensável;
* defesa em profundidade reduz riscos;
* segurança é responsabilidade compartilhada;
* ferramentas ajudam, mas não substituem boas decisões de engenharia.

A partir daqui, os próximos temas importantes são HTTP, DNS, sessões, HTTPS, SQL Injection, XSS, DAST e conformidade.

## Perguntas de revisão

### 1. O que é segurança de aplicações web?

Segurança de aplicações web é o conjunto de práticas, processos, controles e mecanismos usados para proteger aplicações acessadas pela web contra ameaças, acessos indevidos, vazamento de dados, alteração não autorizada e indisponibilidade.

### 2. Segurança de aplicação web é igual a segurança de rede?

Não. Segurança de rede protege a infraestrutura de comunicação, como portas, firewall e segmentação. Segurança de aplicação protege o comportamento da aplicação, como validação de entrada, autenticação, autorização, sessão, regras de negócio e tratamento de dados.

### 3. O que é superfície de ataque?

Superfície de ataque é o conjunto de pontos pelos quais uma aplicação pode ser acessada, manipulada ou explorada. Em aplicações web, isso inclui parâmetros de URL, formulários, APIs, cookies, headers, uploads e integrações.

### 4. Por que parâmetros de URL e formulários são vetores de entrada?

Porque são caminhos pelos quais dados externos entram na aplicação. Como esses dados podem ser manipulados pelo usuário, precisam ser tratados como não confiáveis.

### 5. Por que não devemos confiar apenas na validação do frontend?

Porque o frontend pode ser ignorado. Um usuário pode enviar requisições diretamente para a API. Por isso, validações importantes precisam existir no backend.

### 6. O que significa confidencialidade?

Confidencialidade significa garantir que dados sensíveis sejam acessados apenas por pessoas, sistemas ou processos autorizados.

### 7. O que significa integridade?

Integridade significa garantir que os dados não sejam modificados de forma indevida, acidental ou maliciosa.

### 8. O que significa disponibilidade?

Disponibilidade significa garantir que a aplicação e seus dados estejam acessíveis quando necessário.

### 9. Qual é a diferença entre autenticação e autorização?

Autenticação verifica quem é o usuário. Autorização verifica o que esse usuário tem permissão para fazer.

### 10. O que é responsabilidade em segurança?

Responsabilidade significa conseguir rastrear ações até usuários, processos ou sistemas, normalmente por meio de logs e auditoria.

### 11. O que é defesa em profundidade?

Defesa em profundidade é o uso de múltiplas camadas de proteção. A ideia é que, se uma camada falhar, outras ainda possam reduzir o impacto.

### 12. Segurança deve ser aplicada apenas no final do projeto?

Não. Segurança deve estar presente desde os requisitos, passando por design, implementação, testes, deploy e operação.

## Referências

* OWASP Top 10:2025.
* OWASP Application Security Verification Standard.
* OWASP Web Security Testing Guide.
* OWASP Cheat Sheet Series.
* Oracle Java SE 25 Documentation.
* Spring Boot 4 Documentation.
* Spring Framework 7 Documentation.

[1]: https://owasp.org/Top10/2025/en/ "OWASP Top 10:2025"
[2]: https://www.oracle.com/br/java/technologies/downloads/ "Java Downloads"
