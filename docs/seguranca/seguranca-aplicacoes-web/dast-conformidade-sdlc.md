# DAST, Conformidade e Segurança no Ciclo de Desenvolvimento

Nos artigos anteriores, estudamos os fundamentos da segurança de aplicações web, HTTP, DNS, sessões, HTTPS, SQL Injection e Cross-Site Scripting. Agora vamos fechar a série olhando para um ponto essencial: como transformar segurança em prática contínua dentro do desenvolvimento de software.

Uma aplicação segura não nasce apenas de uma boa configuração isolada ou de uma correção feita no final do projeto. Segurança precisa aparecer em requisitos, design, implementação, testes, revisão, deploy, monitoramento e evolução.

É nesse contexto que entram três temas importantes:

* DAST;
* conformidade;
* segurança no ciclo de desenvolvimento.

DAST ajuda a encontrar vulnerabilidades observando a aplicação em execução. Conformidade ajuda a alinhar a aplicação a normas, regulamentações e obrigações externas. Segurança no ciclo de desenvolvimento ajuda a transformar proteção em processo contínuo, não em ação emergencial.

Imagine a aplicação fictícia `meucatalogomusical.com`, um catálogo musical com bandas de Metal, Hard Rock e Rock Progressivo. Ela possui cadastro de usuários, login, comentários, avaliações, área administrativa e APIs públicas. Mesmo que o domínio seja musical, a aplicação ainda lida com dados, permissões, sessões, logs, integrações e entradas externas.

Se segurança for lembrada apenas perto do deploy, muitos problemas já podem estar espalhados pelo código, arquitetura e configuração. O ideal é que segurança acompanhe todo o ciclo de vida da aplicação.

## O problema real que esse conceito resolve

O problema real é que vulnerabilidades não aparecem apenas porque alguém “esqueceu uma validação”.

Elas surgem por vários motivos:

* requisitos incompletos;
* regras de autorização mal definidas;
* falta de validação de entrada;
* uso incorreto de frameworks;
* dependências vulneráveis;
* configurações inseguras;
* logs expondo dados sensíveis;
* ausência de testes de segurança;
* desconhecimento de riscos;
* pressão por entrega;
* falta de revisão;
* ambientes diferentes entre desenvolvimento e produção.

Se o time trata segurança apenas como uma etapa final, os problemas são descobertos tarde, quando a correção é mais cara, arriscada e demorada.

DAST, conformidade e práticas de ciclo de desenvolvimento seguro ajudam a resolver esse problema trazendo segurança para dentro do fluxo normal de engenharia.

A ideia principal é:

> Segurança deve ser parte do processo de construir software, não apenas uma inspeção no final.

## Antes de entrar nos detalhes: uma ideia simples

Pense no desenvolvimento de uma nova funcionalidade em `meucatalogomusical.com`: uma página para usuários avaliarem álbuns da banda Angra.

Um fluxo inseguro seria:

1. implementar a funcionalidade;
2. testar apenas se funciona;
3. publicar;
4. esperar que nenhum problema aconteça;
5. corrigir vulnerabilidades apenas quando alguém reportar.

Um fluxo melhor seria:

1. entender quais dados serão recebidos;
2. definir quem pode avaliar;
3. validar entradas;
4. proteger contra abuso;
5. revisar autorização;
6. criar testes automatizados;
7. executar análise de segurança;
8. testar a aplicação em execução;
9. revisar logs e configurações;
10. monitorar produção.

DAST entra principalmente na etapa em que a aplicação já está em execução em um ambiente controlado. Ele não substitui boas práticas de código, mas ajuda a encontrar problemas observáveis externamente.

Conformidade entra como um conjunto de requisitos e evidências que demonstram que a organização segue determinados padrões.

O ciclo de desenvolvimento seguro une tudo isso em um processo contínuo.

## Definição técnica de DAST

DAST significa Dynamic Application Security Testing, ou Teste Dinâmico de Segurança de Aplicações.

É uma abordagem de teste que avalia uma aplicação em execução, observando seu comportamento a partir de fora.

Em vez de analisar diretamente o código-fonte, o DAST interage com a aplicação por meio de suas interfaces expostas, como:

* páginas web;
* formulários;
* endpoints REST;
* autenticação;
* parâmetros de URL;
* headers HTTP;
* cookies;
* respostas do servidor;
* redirecionamentos;
* mensagens de erro.

A ideia é testar a aplicação de uma forma parecida com o que um cliente externo conseguiria observar.

Importante: neste artigo, o foco é defensivo. DAST deve ser usado em aplicações próprias, ambientes autorizados e pipelines controlados.

## DAST não é a mesma coisa que SAST

É comum comparar DAST com SAST.

SAST significa Static Application Security Testing. Ele analisa código-fonte, bytecode ou artefatos sem precisar executar a aplicação.

DAST analisa a aplicação rodando.

Comparação simples:

| Abordagem            | O que analisa                          | Quando costuma ser usada                  | O que encontra melhor              |
| -------------------- | -------------------------------------- | ----------------------------------------- | ---------------------------------- |
| SAST                 | Código ou artefatos                    | Durante desenvolvimento e build           | Padrões inseguros no código        |
| DAST                 | Aplicação em execução                  | Ambiente de teste, homologação ou staging | Falhas observáveis externamente    |
| Revisão manual       | Código, arquitetura e regra de negócio | Durante desenvolvimento e revisão         | Problemas de contexto e negócio    |
| Testes automatizados | Comportamento esperado                 | Durante CI/CD                             | Regressões e regras específicas    |
| Pentest autorizado   | Sistema em escopo definido             | Validações periódicas                     | Combinações de falhas e risco real |

Nenhuma abordagem resolve tudo sozinha.

O ideal é combinar várias técnicas.

## Como DAST funciona na prática

Um teste DAST normalmente segue uma ideia parecida com esta:

1. a aplicação sobe em um ambiente controlado;
2. a ferramenta recebe uma URL base;
3. a ferramenta navega ou interage com a aplicação;
4. ela identifica páginas, links, formulários e endpoints;
5. executa verificações de segurança;
6. observa respostas, códigos HTTP, headers e comportamentos;
7. gera um relatório de possíveis vulnerabilidades;
8. o time analisa os achados;
9. falsos positivos são descartados;
10. problemas reais entram no backlog de correção.

Exemplo de ambiente controlado:

```text id="qv4z6c"
https://staging.meucatalogomusical.com
```

Esse ambiente deve representar produção o suficiente para que o teste seja útil, mas sem expor dados reais desnecessários.

## O que DAST pode ajudar a encontrar

DAST pode ajudar a detectar problemas como:

* ausência de headers de segurança;
* mensagens de erro detalhadas;
* cookies sem atributos adequados;
* problemas simples de autenticação;
* problemas de configuração;
* endpoints expostos indevidamente;
* falhas de validação observáveis;
* comportamentos suspeitos em formulários;
* riscos comuns em páginas e APIs.

Ele também pode ajudar a apontar possíveis riscos relacionados a vulnerabilidades como SQL Injection e XSS, mas os achados precisam ser revisados por pessoas do time.

DAST gera indícios. Nem todo alerta é automaticamente uma vulnerabilidade real, e nem toda ausência de alerta significa que a aplicação está segura.

## Limitações do DAST

DAST é útil, mas possui limitações.

Ele pode não entender:

* regra de negócio;
* autorização complexa;
* fluxos internos;
* validações específicas do domínio;
* eventos assíncronos;
* mensageria;
* processamento em lote;
* decisões arquiteturais;
* permissões contextuais;
* vulnerabilidades que exigem estado específico.

Por exemplo, em `meucatalogomusical.com`, uma ferramenta DAST pode acessar a tela de avaliações. Mas ela talvez não saiba que apenas usuários que compraram ingresso para um show deveriam avaliar aquele show, caso essa fosse uma regra de negócio.

Esse tipo de regra exige testes específicos, revisão de código e entendimento do domínio.

## DAST deve ser usado em ambiente autorizado

DAST interage ativamente com uma aplicação em execução. Por isso, deve ser usado apenas em ambientes autorizados.

Não se deve apontar ferramentas de DAST para sistemas de terceiros sem permissão.

Em um fluxo profissional, o escopo deve estar claro:

* qual ambiente será testado;
* quais domínios podem ser acessados;
* quais credenciais de teste serão usadas;
* quais dados são fictícios;
* quais horários são adequados;
* quem receberá alertas;
* como lidar com falso positivo;
* como priorizar correções.

Essa organização evita impacto operacional e mantém o uso da ferramenta dentro de um contexto ético e controlado.

## Segurança no ciclo de desenvolvimento

Segurança no ciclo de desenvolvimento significa integrar práticas de segurança em todas as etapas do SDLC.

SDLC significa Software Development Life Cycle, ou ciclo de vida de desenvolvimento de software.

Um ciclo simplificado pode ter:

1. requisitos;
2. design;
3. implementação;
4. revisão;
5. testes;
6. build;
7. deploy;
8. operação;
9. monitoramento;
10. melhoria contínua.

A segurança deve aparecer em todas essas fases.

## Segurança em requisitos

Na fase de requisitos, o time deve perguntar:

* quais dados serão tratados?
* há dados pessoais?
* há dados de pagamento?
* há informações sensíveis?
* quem pode acessar cada funcionalidade?
* quais ações precisam de auditoria?
* existe requisito regulatório?
* qual é o impacto se essa funcionalidade falhar?
* quais abusos são possíveis?

Exemplo para `meucatalogomusical.com`:

Funcionalidade: cadastro de resenhas de álbuns da banda Rhapsody of Fire.

Perguntas de segurança:

* qualquer usuário pode criar resenha?
* resenhas passam por moderação?
* o texto aceita HTML?
* existe limite de tamanho?
* comentários podem ser editados?
* quem pode excluir?
* ações administrativas são auditadas?
* o conteúdo será exibido em páginas públicas?

Essas perguntas ajudam a evitar decisões inseguras antes mesmo do código existir.

## Segurança no design

Na fase de design, o time define como a funcionalidade será protegida.

Algumas decisões importantes:

* autenticação;
* autorização;
* fluxo de sessão;
* segregação de perfis;
* validação de entrada;
* modelagem de dados sensíveis;
* logs de auditoria;
* proteção contra abuso;
* limites de requisição;
* mensagens de erro;
* comunicação segura;
* integração com serviços externos.

Exemplo:

Para uma área administrativa de `meucatalogomusical.com`, o design pode definir:

* apenas usuários com papel `ADMIN` acessam;
* ações críticas exigem auditoria;
* comentários removidos mantêm registro histórico;
* logs não armazenam dados sensíveis desnecessários;
* endpoints administrativos não ficam misturados com endpoints públicos;
* alterações relevantes geram eventos de auditoria.

## Segurança na implementação

Na implementação, o desenvolvedor transforma requisitos e design em código.

Práticas importantes:

* validação de entrada;
* consultas parametrizadas;
* codificação de saída;
* controle de autorização;
* uso correto de frameworks;
* tratamento seguro de erros;
* logs responsáveis;
* menor privilégio;
* proteção de sessão;
* testes automatizados.

Exemplo de validação em Java/Spring:

```java id="o60aiw"
package com.meucatalogomusical.resenhas.api;

import jakarta.validation.constraints.Max;
import jakarta.validation.constraints.Min;
import jakarta.validation.constraints.NotBlank;
import jakarta.validation.constraints.Size;

public record NovaResenhaRequest(

        @NotBlank
        @Size(max = 120)
        String titulo,

        @NotBlank
        @Size(max = 3000)
        String texto,

        @Min(1)
        @Max(5)
        int nota
) {
}
```

Esse DTO define regras básicas para uma resenha:

* título obrigatório;
* título com tamanho máximo;
* texto obrigatório;
* texto com tamanho máximo;
* nota entre 1 e 5.

Isso não resolve todos os problemas, mas evita que qualquer entrada circule livremente pela aplicação.

## Exemplo de autorização em Spring

Uma funcionalidade administrativa deve validar permissão no backend.

```java id="nnawz7"
package com.meucatalogomusical.admin.api;

import org.springframework.security.access.prepost.PreAuthorize;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/admin/resenhas")
public class AdminResenhaController {

    private final AdminResenhaService adminResenhaService;

    public AdminResenhaController(AdminResenhaService adminResenhaService) {
        this.adminResenhaService = adminResenhaService;
    }

    @DeleteMapping("/{id}")
    @PreAuthorize("hasRole('ADMIN')")
    public void removerResenha(@PathVariable Long id) {
        adminResenhaService.remover(id);
    }
}
```

A anotação `@PreAuthorize("hasRole('ADMIN')")` reforça que apenas usuários autorizados podem executar a ação.

A interface da aplicação também deve esconder botões indevidos, mas a proteção real precisa estar no backend.

## Exemplo de auditoria simples

Ações sensíveis devem deixar rastros úteis.

```java id="4fhf1p"
package com.meucatalogomusical.auditoria.domain;

import java.time.OffsetDateTime;

public record EventoAuditoria(
        String usuario,
        String acao,
        String recurso,
        Long recursoId,
        OffsetDateTime ocorridoEm
) {
}
```

Uso conceitual em um serviço:

```java id="lk7lp5"
package com.meucatalogomusical.admin.application;

import com.meucatalogomusical.auditoria.domain.EventoAuditoria;

import java.time.OffsetDateTime;

public class AdminResenhaService {

    private final ResenhaRepository resenhaRepository;
    private final AuditoriaService auditoriaService;

    public AdminResenhaService(
            ResenhaRepository resenhaRepository,
            AuditoriaService auditoriaService
    ) {
        this.resenhaRepository = resenhaRepository;
        this.auditoriaService = auditoriaService;
    }

    public void remover(Long resenhaId, String usuarioAtual) {
        resenhaRepository.remover(resenhaId);

        auditoriaService.registrar(new EventoAuditoria(
                usuarioAtual,
                "REMOCAO_RESENHA",
                "RESENHA",
                resenhaId,
                OffsetDateTime.now()
        ));
    }
}
```

Esse exemplo mostra uma ideia importante: segurança também envolve rastreabilidade.

Porém, logs e auditoria não devem expor dados sensíveis sem necessidade.

## Segurança nos testes

Os testes automatizados ajudam a evitar regressões.

Exemplos de testes importantes:

* usuário anônimo não acessa área privada;
* usuário comum não acessa área administrativa;
* entrada inválida retorna erro adequado;
* campos obrigatórios são validados;
* dados sensíveis não aparecem na resposta;
* endpoint usa status HTTP correto;
* erros não expõem stack trace;
* autorização é aplicada no backend.

Exemplo de teste com Spring:

```java id="7yanas"
package com.meucatalogomusical.admin.api;

import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.security.test.context.support.WithMockUser;
import org.springframework.test.web.servlet.MockMvc;

import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.delete;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

@WebMvcTest(AdminResenhaController.class)
class AdminResenhaControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @Test
    @WithMockUser(roles = "USER")
    void deveBloquearUsuarioComumAoRemoverResenha() throws Exception {
        mockMvc.perform(delete("/api/admin/resenhas/10"))
                .andExpect(status().isForbidden());
    }
}
```

Esse teste garante que um usuário com papel `USER` não consegue remover uma resenha administrativa.

Esse tipo de teste é simples, mas importante. Ele protege regras de autorização contra regressões futuras.

## Segurança no build e CI/CD

Em um pipeline de CI/CD, o time pode incluir verificações como:

* testes unitários;
* testes de integração;
* análise de dependências;
* análise estática;
* validação de configuração;
* geração de artefato;
* deploy em ambiente de teste;
* DAST em ambiente controlado;
* publicação somente se critérios mínimos forem atendidos.

Um fluxo simplificado:

```text id="dmzycd"
commit -> build -> testes -> análise -> deploy em staging -> DAST -> revisão -> produção
```

A ideia não é bloquear toda entrega por qualquer alerta sem contexto. A ideia é criar critérios claros.

Exemplos:

* falhas críticas bloqueiam produção;
* alertas médios entram no backlog com prazo;
* falsos positivos são documentados;
* exceções precisam de justificativa;
* problemas recorrentes viram melhoria de processo.

## Exemplo conceitual de pipeline

Um pipeline simplificado poderia ser:

```yaml id="bf1m8c"
name: seguranca-aplicacao

on:
  pull_request:
  push:
    branches:
      - main

jobs:
  build-test:
    runs-on: ubuntu-latest

    steps:
      - name: Baixar código
        uses: actions/checkout@v4

      - name: Configurar Java
        uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: '25'

      - name: Executar testes
        run: ./gradlew test

      - name: Gerar aplicação
        run: ./gradlew bootJar
```

Esse exemplo não executa DAST diretamente. Ele mostra apenas a base de um pipeline.

Em projetos reais, a etapa DAST normalmente exigiria:

* subir a aplicação;
* usar banco de dados de teste;
* configurar credenciais de teste;
* apontar a ferramenta para ambiente autorizado;
* armazenar relatório;
* definir critérios de falha;
* revisar resultados.

## Preparando a aplicação para ser testável

Para que DAST e testes de segurança sejam úteis, a aplicação precisa ser testável.

Isso significa:

* ambientes reproduzíveis;
* dados fictícios;
* usuários de teste;
* rotas documentadas;
* autenticação configurável para testes autorizados;
* logs com correlationId;
* erros padronizados;
* deploy automatizado;
* reset de ambiente;
* documentação de escopo.

Exemplo de usuários de teste para `meucatalogomusical.com`:

| Usuário           | Perfil    | Finalidade                  |
| ----------------- | --------- | --------------------------- |
| `leitor.teste`    | USER      | Validar acesso comum        |
| `moderador.teste` | MODERATOR | Validar moderação           |
| `admin.teste`     | ADMIN     | Validar área administrativa |

Esses usuários devem existir apenas em ambiente controlado e usar dados fictícios.

## Ambientes e dados de teste

DAST não deve ser executado contra produção sem planejamento rigoroso e autorização explícita.

O melhor é usar ambientes como:

* local;
* integração;
* staging;
* homologação;
* ambiente efêmero de pull request.

Dados usados nesses ambientes devem ser fictícios ou devidamente mascarados.

Evite usar:

* e-mails reais;
* dados pessoais reais;
* credenciais reais;
* tokens de produção;
* dados de pagamento reais;
* segredos de produção.

Uma boa prática é automatizar criação e limpeza de dados de teste.

## Conformidade não é a mesma coisa que segurança

Conformidade significa atender a requisitos de normas, leis, padrões ou contratos.

Segurança significa reduzir riscos reais.

Essas duas coisas se relacionam, mas não são iguais.

Uma aplicação pode estar em conformidade com vários requisitos e ainda assim ter decisões inseguras em pontos específicos.

Também pode existir uma aplicação tecnicamente bem protegida, mas sem evidências, documentação e processos exigidos por uma auditoria.

O ideal é buscar os dois:

* segurança real;
* evidência de conformidade.

## PCI DSS

PCI DSS é um padrão de segurança voltado à proteção de dados de contas de pagamento.

Ele é relevante para organizações que armazenam, processam ou transmitem dados de cartão.

Se `meucatalogomusical.com` vendesse ingressos ou produtos oficiais de bandas como Avantasia, Stratovarius ou Kamelot e processasse pagamentos com cartão, o escopo de PCI poderia se tornar relevante.

A melhor decisão de engenharia, muitas vezes, é reduzir o escopo:

* usar um provedor de pagamento confiável;
* evitar armazenar dados de cartão;
* tokenizar quando aplicável;
* não trafegar dados sensíveis desnecessariamente;
* segmentar responsabilidades;
* seguir requisitos do provedor e do padrão aplicável.

Reduzir escopo não significa ignorar segurança. Significa projetar a aplicação para lidar com menos dados sensíveis diretamente.

## GDPR

GDPR é uma regulamentação europeia de proteção de dados pessoais.

Mesmo que uma aplicação seja brasileira, o GDPR pode ser relevante se ela tratar dados de pessoas localizadas na União Europeia, dependendo do contexto de oferta de serviços, monitoramento ou operação.

Conceitos importantes relacionados a privacidade:

* transparência;
* finalidade;
* minimização de dados;
* limitação de armazenamento;
* precisão;
* integridade;
* confidencialidade;
* responsabilidade.

Em `meucatalogomusical.com`, dados pessoais podem incluir:

* nome;
* e-mail;
* identificador de usuário;
* endereço IP;
* preferências;
* histórico de interação;
* dados de conta.

Uma boa decisão de engenharia é coletar apenas o necessário.

Exemplo: se o sistema só precisa de e-mail para login e recuperação de conta, talvez não precise pedir data de nascimento, telefone ou endereço.

## Conformidade no desenvolvimento

Conformidade deve influenciar o desenvolvimento desde cedo.

Exemplos de perguntas:

* quais dados pessoais são coletados?
* por que esses dados são necessários?
* por quanto tempo serão armazenados?
* quem tem acesso?
* existe auditoria?
* os dados são protegidos em trânsito?
* os dados são protegidos em repouso quando necessário?
* o usuário consegue exercer direitos previstos?
* há processo de exclusão ou anonimização?
* incidentes são tratados?
* evidências são armazenadas?

Essas perguntas ajudam a transformar conformidade em decisões concretas de software.

## ASVS como guia de requisitos verificáveis

O OWASP ASVS pode ser usado como referência para requisitos técnicos de segurança.

Ele ajuda times a transformar segurança em requisitos verificáveis.

Em vez de dizer apenas:

```text id="5g4d2x"
A aplicação deve ser segura.
```

podemos ter requisitos mais específicos:

```text id="onxmyb"
A aplicação deve validar autorização no backend para todas as ações administrativas.
```

Ou:

```text id="suxm0n"
Cookies de sessão devem usar atributos seguros em produção.
```

Ou:

```text id="3p99dk"
Mensagens de erro não devem expor detalhes internos.
```

Requisitos verificáveis são melhores porque podem virar teste, checklist, revisão de código ou critério de aceite.

## SAMM e maturidade de segurança

OWASP SAMM é um modelo de maturidade para melhorar segurança no ciclo de desenvolvimento.

A ideia não é que todas as empresas apliquem exatamente o mesmo processo. O modelo é orientado a risco e pode evoluir conforme a organização amadurece.

Em uma equipe pequena, o primeiro passo pode ser:

* checklist de segurança em pull requests;
* validação de dependências;
* revisão de autorização;
* testes básicos;
* DAST em staging antes de produção.

Em uma organização maior, pode envolver:

* governança;
* políticas;
* métricas;
* treinamento;
* threat modeling;
* automação;
* gestão formal de risco;
* integração com auditoria;
* resposta a incidentes;
* maturidade por times e produtos.

O ponto principal é: segurança evolui por maturidade, não por uma ação única.

## Segurança em pull requests

Pull requests são uma excelente oportunidade para revisar segurança.

Um checklist simples pode incluir:

* a funcionalidade recebe entrada externa?
* a entrada é validada?
* há dados sensíveis?
* há mudança de autorização?
* há novo endpoint?
* há query dinâmica?
* há uso de HTML não escapado?
* há novo cookie?
* há alteração em configuração?
* há log com dados sensíveis?
* há dependência nova?
* há impacto em conformidade?

Esse checklist não precisa ser burocrático. Ele ajuda o time a criar hábito.

## Exemplo de checklist no pull request

```markdown id="7o2tym"
## Checklist de segurança

- [ ] A entrada externa foi validada no backend.
- [ ] Endpoints privados exigem autenticação.
- [ ] A autorização foi revisada.
- [ ] Dados sensíveis não são registrados em logs.
- [ ] Consultas ao banco usam parâmetros.
- [ ] Saída em HTML é escapada.
- [ ] Cookies e sessão não foram enfraquecidos.
- [ ] Dependências novas foram justificadas.
- [ ] Mensagens de erro não expõem detalhes internos.
- [ ] Impactos de privacidade/conformidade foram avaliados.
```

Esse tipo de checklist é simples, mas ajuda a lembrar pontos que costumam passar despercebidos.

## Segurança e observabilidade

Segurança não termina no deploy.

Depois que a aplicação está em produção, é importante observar comportamentos.

Exemplos de eventos úteis:

* muitas tentativas de login malsucedidas;
* acessos negados repetidos;
* erros incomuns em endpoints críticos;
* tentativas de acessar rotas administrativas;
* picos de requisições;
* falhas de validação em massa;
* alterações administrativas;
* criação ou remoção de permissões;
* comportamento anormal por usuário ou IP.

Isso não significa vigiar usuários de forma invasiva. Significa registrar eventos relevantes para proteger a aplicação e investigar incidentes.

## Exemplo de correlationId

Um `correlationId` ajuda a rastrear uma requisição entre logs.

Exemplo de filtro simples:

```java id="u3pbbh"
package com.meucatalogomusical.config;

import jakarta.servlet.FilterChain;
import jakarta.servlet.ServletException;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import org.slf4j.MDC;
import org.springframework.web.filter.OncePerRequestFilter;

import java.io.IOException;
import java.util.UUID;

public class CorrelationIdFilter extends OncePerRequestFilter {

    private static final String HEADER_NAME = "X-Correlation-Id";
    private static final String MDC_KEY = "correlationId";

    @Override
    protected void doFilterInternal(
            HttpServletRequest request,
            HttpServletResponse response,
            FilterChain filterChain
    ) throws ServletException, IOException {

        String correlationId = request.getHeader(HEADER_NAME);

        if (correlationId == null || correlationId.isBlank()) {
            correlationId = UUID.randomUUID().toString();
        }

        MDC.put(MDC_KEY, correlationId);
        response.setHeader(HEADER_NAME, correlationId);

        try {
            filterChain.doFilter(request, response);
        } finally {
            MDC.remove(MDC_KEY);
        }
    }
}
```

Esse filtro cria ou reaproveita um identificador de correlação.

Com isso, logs relacionados à mesma requisição podem ser agrupados com mais facilidade.

Importante: correlationId não deve conter dados sensíveis.

## Segurança e dependências

Aplicações modernas usam muitas dependências.

Cada biblioteca adicionada ao projeto pode trazer:

* funcionalidades;
* produtividade;
* riscos;
* vulnerabilidades transitivas;
* necessidade de atualização;
* impacto de compatibilidade.

Boas práticas:

* adicionar dependência apenas quando houver necessidade;
* preferir bibliotecas conhecidas e mantidas;
* acompanhar vulnerabilidades;
* atualizar versões;
* remover dependências não utilizadas;
* evitar bibliotecas abandonadas;
* revisar mudanças em upgrades.

Em Java/Spring, dependências são parte central do projeto. Logo, gestão de dependências também é segurança.

## O que fazer com achados de DAST?

Um erro comum é tratar relatório de ferramenta como verdade absoluta.

O fluxo ideal é:

1. receber o relatório;
2. classificar achados;
3. remover duplicidades;
4. avaliar falsos positivos;
5. confirmar impacto;
6. definir severidade;
7. criar tarefas;
8. corrigir;
9. testar novamente;
10. registrar evidências.

Um alerta crítico deve ter prioridade alta. Um alerta informativo pode virar melhoria futura. Um falso positivo deve ser documentado para não gerar ruído recorrente.

## Priorização de vulnerabilidades

Nem toda vulnerabilidade tem o mesmo risco.

A priorização pode considerar:

* facilidade de exploração;
* impacto nos dados;
* exposição pública;
* existência de autenticação;
* perfil afetado;
* criticidade do sistema;
* requisitos regulatórios;
* existência de mitigação;
* histórico de incidentes;
* sensibilidade do dado;
* ambiente afetado.

Exemplo:

Uma falha em uma página pública de login pode ser mais urgente do que uma falha em uma ferramenta interna isolada, dependendo do impacto.

Mas uma falha interna em área administrativa também pode ser crítica se afetar dados sensíveis ou permissões.

## Métricas úteis

Algumas métricas podem ajudar a acompanhar maturidade:

* quantidade de vulnerabilidades por severidade;
* tempo médio de correção;
* reincidência por tipo de falha;
* percentual de endpoints cobertos por testes;
* dependências vulneráveis abertas;
* falhas críticas por release;
* cobertura de revisão de segurança;
* quantidade de exceções abertas;
* tempo de atualização de dependências;
* número de incidentes ou quase incidentes.

Métrica boa ajuda a tomar decisão. Métrica ruim vira burocracia.

O objetivo não é punir times, mas melhorar o processo.

## Boas práticas

Boas práticas para DAST, conformidade e ciclo seguro:

* executar DAST apenas em ambientes autorizados;
* usar dados fictícios em testes;
* definir escopo antes dos testes;
* revisar achados antes de criar tarefas;
* documentar falsos positivos;
* integrar segurança ao CI/CD;
* validar segurança em pull requests;
* usar ASVS como base para requisitos verificáveis;
* usar SAMM para evoluir maturidade;
* tratar conformidade desde requisitos;
* reduzir coleta de dados pessoais;
* reduzir escopo de dados de pagamento;
* aplicar menor privilégio;
* manter evidências de correção;
* monitorar produção;
* treinar desenvolvedores;
* revisar dependências;
* priorizar vulnerabilidades por risco.

## Erros comuns

### 1. Usar DAST apenas no final

Se DAST roda apenas antes de produção, os problemas aparecem tarde demais.

O ideal é executar testes de segurança ao longo do ciclo.

### 2. Ignorar falsos positivos

Falsos positivos precisam ser analisados e documentados. Ignorar tudo reduz confiança no processo.

### 3. Achar que ausência de alerta significa aplicação segura

DAST não encontra tudo. Ele é uma camada, não uma garantia absoluta.

### 4. Rodar testes em ambiente errado

DAST deve ser usado em ambiente autorizado, controlado e com dados apropriados.

### 5. Confundir conformidade com segurança real

Estar em conformidade não garante que todas as decisões técnicas são seguras.

### 6. Criar checklist burocrático demais

Checklist precisa ajudar o time, não virar obstáculo sem valor.

### 7. Não envolver desenvolvedores

Segurança não deve ser responsabilidade apenas de um time externo. Desenvolvedores precisam entender os riscos principais.

### 8. Não corrigir causa raiz

Corrigir apenas um endpoint vulnerável pode não resolver o padrão inseguro usado em outros lugares.

### 9. Não medir reincidência

Se o mesmo problema aparece sempre, talvez falte treinamento, padrão de código ou melhoria arquitetural.

### 10. Usar dados reais em ambiente de teste

Ambientes de teste devem evitar dados reais. Quando inevitável, dados precisam ser minimizados, mascarados ou protegidos conforme o risco.

## Decisões de engenharia

### Quando usar DAST?

Use DAST quando houver uma aplicação em execução em ambiente controlado.

Ele é útil para:

* validar configurações;
* identificar problemas observáveis;
* testar endpoints expostos;
* complementar SAST e testes automatizados;
* apoiar evidências de segurança;
* detectar regressões.

### Quando evitar DAST?

Evite executar DAST:

* em sistemas sem autorização;
* em produção sem planejamento;
* com dados reais desnecessários;
* sem escopo definido;
* sem equipe para analisar resultados;
* sem critério de priorização.

### Quando usar ASVS?

Use ASVS quando precisar transformar segurança em requisitos verificáveis.

Ele é útil para:

* checklists técnicos;
* critérios de aceite;
* revisões de arquitetura;
* definição de padrões internos;
* avaliação de maturidade técnica;
* preparação para auditorias.

### Quando usar SAMM?

Use SAMM quando a organização quiser evoluir maturidade de segurança no ciclo de desenvolvimento.

Ele é útil para:

* estruturar programa de segurança;
* medir evolução;
* definir roadmap;
* alinhar times;
* melhorar governança;
* equilibrar risco e esforço.

### Quando conformidade deve entrar?

Conformidade deve entrar desde requisitos.

Se a aplicação trata dados pessoais, dados de pagamento ou informações reguladas, o time precisa saber disso antes de desenhar a solução.

Quanto mais tarde a conformidade entra, maior o risco de retrabalho.

## Onde isso aparece em sistemas reais

Esses temas aparecem em praticamente qualquer aplicação corporativa.

Exemplos:

* bancos;
* fintechs;
* e-commerces;
* sistemas de saúde;
* sistemas educacionais;
* plataformas SaaS;
* marketplaces;
* APIs públicas;
* aplicações internas;
* backoffices;
* sistemas com login;
* sistemas que tratam dados pessoais;
* sistemas que processam pagamento.

Mesmo uma aplicação de catálogo musical pode evoluir para funcionalidades sensíveis:

* venda de ingressos;
* assinatura premium;
* pagamento com cartão;
* mensagens privadas;
* perfis públicos;
* integrações com parceiros;
* recomendação personalizada;
* moderação de conteúdo;
* área administrativa.

Quanto mais o sistema cresce, mais importante é ter segurança integrada ao processo.

## Checklist prático

### Requisitos

* Há dados pessoais?
* Há dados de pagamento?
* Há dados sensíveis?
* Os perfis de acesso foram definidos?
* Há necessidade de auditoria?
* Requisitos regulatórios foram identificados?
* Riscos principais foram discutidos?

### Design

* Autenticação foi definida?
* Autorização foi modelada?
* Dados sensíveis foram minimizados?
* Logs foram planejados?
* Erros foram padronizados?
* Integrações externas foram avaliadas?
* Menor privilégio foi considerado?

### Implementação

* Entradas são validadas?
* SQL usa parâmetros?
* Saída HTML é escapada?
* Cookies são seguros?
* Dados sensíveis não vão para logs?
* Endpoints administrativos têm proteção?
* Dependências novas foram avaliadas?

### Testes

* Há testes de autorização?
* Há testes para entradas inválidas?
* Há testes de erros?
* Há testes para endpoints privados?
* Há testes de regressão?
* DAST roda em ambiente autorizado?
* Achados são revisados?

### Deploy

* HTTPS está ativo?
* Secrets estão protegidos?
* Configuração de produção foi revisada?
* Debug está desativado?
* Headers de segurança foram avaliados?
* Banco usa menor privilégio?
* Logs vão para local seguro?

### Operação

* Há monitoramento?
* Eventos sensíveis são auditados?
* Existe correlationId?
* Alertas têm responsável?
* Vulnerabilidades são priorizadas?
* Dependências são atualizadas?
* Incidentes têm processo de resposta?

## Resumo final

DAST, conformidade e segurança no ciclo de desenvolvimento ajudam a transformar segurança em prática contínua.

DAST testa a aplicação em execução, observando comportamentos externos. Ele é útil para encontrar problemas de configuração, validação, headers, cookies, autenticação e outros riscos observáveis. Mas ele não substitui revisão de código, testes automatizados, SAST, modelagem de ameaças ou entendimento de regra de negócio.

Conformidade ajuda a atender requisitos de normas e regulamentações, como PCI DSS e GDPR. Porém, conformidade não é sinônimo automático de segurança. O ideal é buscar segurança real e evidências de conformidade.

Segurança no ciclo de desenvolvimento significa pensar em proteção desde requisitos até operação. Isso inclui validação, autorização, logs, testes, CI/CD, monitoramento, gestão de dependências, auditoria, resposta a incidentes e melhoria contínua.

A principal lição deste artigo é:

> Segurança não deve ser um evento no final do projeto. Segurança deve ser uma disciplina contínua dentro da engenharia de software.

## Perguntas de revisão

### 1. O que é DAST?

DAST é uma abordagem de teste dinâmico de segurança que avalia uma aplicação em execução a partir de suas interfaces expostas, como páginas, formulários e APIs.

### 2. Qual é a diferença entre DAST e SAST?

SAST analisa código ou artefatos sem executar a aplicação. DAST analisa a aplicação rodando, observando comportamentos externos.

### 3. DAST encontra todos os problemas de segurança?

Não. DAST é uma camada importante, mas não entende completamente regra de negócio, autorização complexa e fluxos internos.

### 4. Onde DAST deve ser executado?

Em ambientes autorizados e controlados, como staging, homologação, integração ou ambientes efêmeros de pull request.

### 5. Por que não é ideal rodar DAST sem escopo?

Porque a ferramenta pode gerar impacto, ruído, falsos positivos ou acessar áreas que não deveriam fazer parte do teste.

### 6. O que é conformidade?

Conformidade é o atendimento a requisitos de normas, leis, padrões, contratos ou políticas aplicáveis a uma organização ou sistema.

### 7. Conformidade é igual a segurança?

Não. Conformidade ajuda, mas não garante que todas as decisões técnicas são seguras. Segurança real exige análise de risco e boas práticas de engenharia.

### 8. O que é PCI DSS?

PCI DSS é um padrão de segurança voltado à proteção de dados de contas de pagamento, especialmente em ambientes que armazenam, processam ou transmitem dados de cartão.

### 9. O que é GDPR?

GDPR é uma regulamentação europeia de proteção de dados pessoais, baseada em princípios como transparência, finalidade, minimização, integridade, confidencialidade e responsabilidade.

### 10. O que é ASVS?

ASVS é um padrão da OWASP que fornece requisitos técnicos verificáveis para segurança de aplicações web.

### 11. O que é SAMM?

SAMM é um modelo de maturidade da OWASP para ajudar organizações a analisar e melhorar segurança ao longo do ciclo de desenvolvimento.

### 12. Segurança deve entrar em qual fase do desenvolvimento?

Segurança deve entrar em todas as fases: requisitos, design, implementação, revisão, testes, build, deploy, operação e melhoria contínua.

### 13. Por que pull requests são importantes para segurança?

Porque permitem revisar mudanças antes de chegarem à produção, incluindo validação de entrada, autorização, logs, dependências e configurações.

### 14. O que fazer com falsos positivos de ferramentas?

Eles devem ser analisados e documentados. Ignorá-los sem critério reduz a confiança no processo.

### 15. Por que dados reais devem ser evitados em testes?

Porque ambientes de teste podem ter controles mais fracos que produção. Usar dados reais aumenta risco de vazamento e impacto regulatório.

## Referências

* OWASP Web Security Testing Guide.
* OWASP DevSecOps Guideline — Dynamic Application Security Testing.
* OWASP Application Security Verification Standard.
* OWASP Software Assurance Maturity Model.
* OWASP Integration Standards — OWASP in SDLC.
* PCI Security Standards Council — PCI Data Security Standard.
* Regulation EU 2016/679 — General Data Protection Regulation.
* Oracle JDK 25 Documentation.
* Spring Boot Reference Documentation.
* Spring Security Reference Documentation.

[1]: https://owasp.org/www-project-devsecops-guideline/latest/02b-Dynamic-Application-Security-Testing "OWASP DevSecOps Guideline - v-0.2 | OWASP Foundation"
[2]: https://www.oracle.com/br/java/technologies/downloads/ "Java Downloads | Oracle Brasil"
