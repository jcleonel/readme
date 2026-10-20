# SQL Injection: Funcionamento, Impactos e Prevenção

SQL Injection é uma das vulnerabilidades mais conhecidas em aplicações web. Ela acontece quando dados externos, geralmente vindos de usuários, são misturados de forma insegura com comandos SQL.

Em uma aplicação web, é comum receber dados por parâmetros de URL, formulários, corpo JSON, filtros de pesquisa, telas de login, campos de ordenação e APIs. Esses dados muitas vezes são usados para consultar, inserir, atualizar ou excluir informações no banco de dados.

O problema começa quando a aplicação trata a entrada do usuário como se fosse parte confiável da consulta SQL.

Imagine a aplicação fictícia `meucatalogomusical.com`, um catálogo musical de bandas de Metal, Hard Rock e Rock Progressivo. Ela permite pesquisar bandas, cadastrar álbuns, avaliar músicas, favoritar artistas e administrar conteúdo.

Uma funcionalidade simples de busca poderia receber um nome de banda:

```text
GET /api/bandas?nome=Pantera
```

Esse valor `Pantera` pode parecer inofensivo. Porém, tecnicamente, ele veio de fora da aplicação. Logo, deve ser tratado como dado não confiável.

SQL Injection surge quando esse dado externo é incorporado de maneira insegura em uma consulta SQL.

## O problema real que SQL Injection resolve no estudo de segurança

Estudar SQL Injection ajuda a entender um problema maior: a separação entre código e dados.

Uma consulta SQL é código interpretado pelo banco de dados. Já a entrada do usuário deveria ser apenas dado.

Quando a aplicação não separa corretamente esses dois mundos, o banco pode interpretar parte da entrada como comando.

Esse é o ponto central da vulnerabilidade:

> SQL Injection acontece quando entrada não confiável consegue alterar o significado de uma consulta SQL.

O objetivo defensivo é garantir que dados enviados pelo usuário sejam tratados apenas como valores, nunca como parte da estrutura do comando SQL.

## Antes de entrar nos detalhes: uma ideia simples

Pense em uma aplicação que permite buscar bandas pelo nome.

O usuário deveria informar algo como:

```text
Dio
```

A aplicação deveria procurar bandas cujo nome contenha `Dio`.

Mas se a aplicação montar a consulta concatenando texto manualmente, ela pode acabar criando um comando SQL cujo significado muda conforme a entrada.

É como escrever uma frase com uma lacuna:

```text
Buscar bandas onde o nome seja igual a: [entrada do usuário]
```

Se a aplicação apenas cola o que o usuário enviou dentro da frase, sem separar corretamente dado e comando, o resultado pode deixar de ser uma simples busca e virar uma instrução inesperada para o banco.

A prevenção começa quando a aplicação deixa claro para o banco:

> “Isto aqui é o comando SQL. Isto aqui é apenas um valor.”

## Definição técnica

SQL Injection é uma vulnerabilidade de injeção em que dados não confiáveis são enviados para um interpretador SQL de forma que possam alterar a estrutura ou o comportamento da consulta executada.

Em aplicações Java/Spring, isso normalmente acontece quando consultas são montadas por concatenação de strings com valores recebidos de fora da aplicação.

Exemplos de entradas externas:

* parâmetros de URL;
* campos de formulário;
* corpo JSON;
* headers HTTP;
* cookies;
* campos de ordenação;
* filtros de relatório;
* dados recebidos de integrações;
* mensagens vindas de filas;
* dados importados de arquivos.

A vulnerabilidade não está no SQL em si. SQL é uma linguagem legítima e necessária. O problema está em construir comandos SQL de maneira insegura.

## Por que SQL Injection é perigoso?

SQL Injection é perigoso porque o banco de dados normalmente guarda informações críticas da aplicação.

Em `meucatalogomusical.com`, o banco pode conter:

* usuários;
* e-mails;
* senhas com hash;
* bandas cadastradas;
* álbuns;
* avaliações;
* comentários;
* permissões;
* logs administrativos;
* dados de auditoria;
* tokens ou chaves internas, caso tenham sido mal armazenados.

Uma falha de SQL Injection pode permitir impactos como:

* leitura indevida de dados;
* alteração de informações;
* exclusão de registros;
* acesso indevido a áreas restritas;
* violação de autenticação;
* vazamento de informações internas;
* comprometimento da integridade dos dados;
* indisponibilidade parcial ou total;
* danos financeiros e reputacionais.

O impacto depende do tipo de falha, dos privilégios da conta do banco, da arquitetura da aplicação e dos controles adicionais existentes.

## Como SQL Injection acontece na prática

A causa mais clássica é montar SQL por concatenação.

Exemplo inseguro:

```java
package com.meucatalogomusical.bandas.infra;

import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.stereotype.Repository;

import java.util.List;

@Repository
public class BandaRepositoryInseguro {

    private final JdbcTemplate jdbcTemplate;

    public BandaRepositoryInseguro(JdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
    }

    public List<BandaResumo> pesquisarPorNome(String nome) {
        String sql = """
                SELECT id, nome, genero
                FROM bandas
                WHERE nome = '""" + nome + "'";

        return jdbcTemplate.query(
                sql,
                (rs, rowNum) -> new BandaResumo(
                        rs.getLong("id"),
                        rs.getString("nome"),
                        rs.getString("genero")
                )
        );
    }
}
```

Esse código é inseguro porque o valor `nome` foi concatenado diretamente dentro da consulta.

A aplicação queria tratar `nome` como dado. Mas, ao concatenar o valor na string SQL, ela permite que a entrada influencie a estrutura final do comando.

Mesmo que o usuário envie apenas `Pantera`, o padrão continua inseguro. O problema não depende de uma entrada específica. O problema é o desenho da consulta.

## Por que concatenação é perigosa?

A concatenação mistura duas coisas que deveriam ficar separadas:

* estrutura da consulta;
* valores fornecidos pelo usuário.

A estrutura da consulta deveria ser fixa e controlada pelo desenvolvedor.

Os valores deveriam ser enviados ao banco como parâmetros.

Quando tudo vira uma única string, o banco recebe apenas o resultado final. Ele não sabe quais trechos vieram do código da aplicação e quais vieram da entrada do usuário.

Isso cria uma brecha para que dados externos alterem o significado da consulta.

## Correção com parâmetros usando JdbcTemplate

Uma forma segura é usar consulta parametrizada.

Exemplo defensivo:

```java
package com.meucatalogomusical.bandas.infra;

import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.stereotype.Repository;

import java.util.List;

@Repository
public class BandaRepository {

    private final JdbcTemplate jdbcTemplate;

    public BandaRepository(JdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
    }

    public List<BandaResumo> pesquisarPorNome(String nome) {
        String sql = """
                SELECT id, nome, genero
                FROM bandas
                WHERE nome = ?
                """;

        return jdbcTemplate.query(
                sql,
                (rs, rowNum) -> new BandaResumo(
                        rs.getLong("id"),
                        rs.getString("nome"),
                        rs.getString("genero")
                ),
                nome
        );
    }
}
```

Agora a consulta tem um marcador `?`.

Esse marcador representa um parâmetro. O valor de `nome` não é colado diretamente dentro da string SQL. Ele é enviado separadamente para o driver JDBC.

A ideia é:

* a consulta define a estrutura do comando;
* o parâmetro define apenas o valor;
* o banco trata o parâmetro como dado.

Essa separação é a principal defesa contra SQL Injection.

## Analisando o código seguro

A consulta segura é:

```java
String sql = """
        SELECT id, nome, genero
        FROM bandas
        WHERE nome = ?
        """;
```

O ponto importante é o `?`.

Ele indica ao JDBC que aquele trecho será preenchido por um valor, não por uma parte da sintaxe SQL.

Depois, o valor é passado separadamente:

```java
nome
```

Com isso, mesmo que o usuário envie caracteres inesperados, o valor será tratado como conteúdo do parâmetro.

Essa é uma diferença fundamental em relação à concatenação.

## Correção com PreparedStatement

Por baixo de muitas abstrações, a ideia é semelhante ao uso de `PreparedStatement`.

Exemplo com JDBC puro:

```java
package com.meucatalogomusical.bandas.infra;

import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;

public class BandaJdbcRepository {

    public BandaResumo buscarPorNome(Connection connection, String nome) throws SQLException {
        String sql = """
                SELECT id, nome, genero
                FROM bandas
                WHERE nome = ?
                """;

        try (PreparedStatement statement = connection.prepareStatement(sql)) {
            statement.setString(1, nome);

            try (ResultSet resultSet = statement.executeQuery()) {
                if (!resultSet.next()) {
                    return null;
                }

                return new BandaResumo(
                        resultSet.getLong("id"),
                        resultSet.getString("nome"),
                        resultSet.getString("genero")
                );
            }
        }
    }
}
```

Aqui, a consulta é preparada antes da entrada ser aplicada.

O método:

```java
statement.setString(1, nome);
```

define o primeiro parâmetro da consulta.

O valor é tratado como dado textual, não como SQL.

## Correção com Spring Data JPA

Em aplicações Spring, é comum usar Spring Data JPA.

Uma consulta segura com parâmetro nomeado poderia ser escrita assim:

```java
package com.meucatalogomusical.bandas.infra;

import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;

import java.util.List;

public interface BandaRepository extends JpaRepository<BandaEntity, Long> {

    @Query("""
            select b
            from BandaEntity b
            where lower(b.nome) like lower(concat('%', :nome, '%'))
            """)
    List<BandaEntity> pesquisarPorNome(@Param("nome") String nome);
}
```

Nesse exemplo, `:nome` é um parâmetro nomeado.

O valor recebido no método é vinculado ao parâmetro por meio de:

```java
@Param("nome") String nome
```

A consulta continua sendo definida pela aplicação, e o valor continua sendo tratado como parâmetro.

## Consulta derivada com Spring Data JPA

Em muitos casos, nem é necessário escrever a query manualmente.

O Spring Data JPA permite consultas derivadas pelo nome do método:

```java
package com.meucatalogomusical.bandas.infra;

import org.springframework.data.jpa.repository.JpaRepository;

import java.util.List;

public interface BandaRepository extends JpaRepository<BandaEntity, Long> {

    List<BandaEntity> findByNomeContainingIgnoreCase(String nome);
}
```

Esse método pode ser suficiente para uma busca simples por nome.

A vantagem é que o desenvolvedor evita montar SQL ou JPQL manualmente.

Porém, isso não elimina a necessidade de validação, paginação, autorização e cuidado com performance.

## Entidade de exemplo

Uma entidade simples para a aplicação poderia ser:

```java
package com.meucatalogomusical.bandas.infra;

import jakarta.persistence.*;

@Entity
@Table(name = "bandas")
public class BandaEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, length = 120)
    private String nome;

    @Column(nullable = false, length = 80)
    private String genero;

    @Column(nullable = false)
    private Integer anoFormacao;

    protected BandaEntity() {
    }

    public BandaEntity(String nome, String genero, Integer anoFormacao) {
        this.nome = nome;
        this.genero = genero;
        this.anoFormacao = anoFormacao;
    }

    public Long getId() {
        return id;
    }

    public String getNome() {
        return nome;
    }

    public String getGenero() {
        return genero;
    }

    public Integer getAnoFormacao() {
        return anoFormacao;
    }
}
```

Essa entidade representa a tabela `bandas`.

Exemplos de dados válidos poderiam ser:

* `Slayer`;
* `Anthrax`;
* `Dio`;
* `Helloween`;
* `Dream Theater`;
* `Rainbow`.

Nenhum desses valores deve ser usado para montar SQL por concatenação.

Mesmo valores aparentemente normais precisam passar pelo mesmo padrão seguro de parametrização.

## Validação de entrada não substitui consulta parametrizada

Validação é importante, mas não substitui parametrização.

Exemplo de entrada para busca:

```java
package com.meucatalogomusical.bandas.api;

import jakarta.validation.constraints.Size;

public record PesquisaBandaRequest(

        @Size(max = 120)
        String nome
) {
}
```

Essa validação limita o tamanho do campo.

Isso ajuda a evitar entradas exageradas e melhora a consistência da aplicação.

Mas a consulta ainda precisa ser parametrizada.

Validação responde à pergunta:

> Esse valor faz sentido para a minha aplicação?

Parametrização responde à pergunta:

> Esse valor será tratado apenas como dado pelo banco?

As duas práticas são complementares.

## Validação por lista permitida

Quando o campo possui um conjunto limitado de valores, uma lista permitida é uma boa defesa.

Por exemplo, em vez de aceitar qualquer texto para o gênero musical, podemos usar um enum:

```java
package com.meucatalogomusical.bandas.domain;

public enum GeneroMusical {
    HEAVY_METAL,
    THRASH_METAL,
    POWER_METAL,
    PROGRESSIVE_METAL,
    HARD_ROCK
}
```

E usar o enum no filtro:

```java
package com.meucatalogomusical.bandas.api;

import com.meucatalogomusical.bandas.domain.GeneroMusical;

public record FiltroBandaRequest(
        String nome,
        GeneroMusical genero
) {
}
```

Essa abordagem reduz entradas inesperadas.

Em campos como ordenação, lista permitida é ainda mais importante, porque nomes de colunas e direções de ordenação muitas vezes não podem ser parametrizados da mesma forma que valores comuns.

## Cuidado especial com ordenação dinâmica

Um caso comum de risco é a ordenação dinâmica.

Imagine um endpoint:

```text
GET /api/bandas?ordenarPor=nome
```

Um erro seria concatenar diretamente o campo recebido:

```java
String sql = """
        SELECT id, nome, genero
        FROM bandas
        ORDER BY """ + ordenarPor;
```

Esse padrão é inseguro, porque `ordenarPor` altera a estrutura da consulta.

A solução é mapear entradas permitidas para colunas conhecidas.

Exemplo defensivo:

```java
package com.meucatalogomusical.bandas.infra;

public enum OrdenacaoBanda {

    NOME("nome"),
    GENERO("genero"),
    ANO_FORMACAO("ano_formacao");

    private final String coluna;

    OrdenacaoBanda(String coluna) {
        this.coluna = coluna;
    }

    public String coluna() {
        return coluna;
    }
}
```

Uso:

```java
public String montarOrdenacao(OrdenacaoBanda ordenacao) {
    return switch (ordenacao) {
        case NOME -> "nome";
        case GENERO -> "genero";
        case ANO_FORMACAO -> "ano_formacao";
    };
}
```

Aqui, a aplicação não aceita qualquer nome de coluna vindo do usuário.

Ela aceita apenas opções previamente definidas.

## Impactos de SQL Injection

SQL Injection pode gerar vários impactos, dependendo da falha e do ambiente.

Os principais são:

* perda de confidencialidade;
* exposição de dados sensíveis;
* perda ou alteração de dados;
* violação de autenticação;
* comprometimento da integridade;
* indisponibilidade;
* impacto regulatório;
* prejuízo reputacional.

## Perda de confidencialidade

A confidencialidade é perdida quando dados privados são acessados por quem não deveria.

Em `meucatalogomusical.com`, isso poderia envolver:

* e-mails de usuários;
* dados de perfil;
* listas privadas;
* informações administrativas;
* registros internos.

Mesmo que a aplicação não seja financeira ou hospitalar, dados de usuários continuam merecendo proteção.

## Exposição de dados sensíveis

Uma falha pode expor dados que a aplicação nunca deveria retornar.

Por exemplo, uma busca pública por bandas não deveria retornar dados da tabela de usuários.

O endpoint:

```text
GET /api/bandas?nome=Rainbow
```

deveria retornar apenas informações públicas sobre bandas.

Se por causa de uma falha a aplicação expõe dados internos, há violação de segurança.

## Perda ou alteração de dados

Dependendo dos privilégios da conta usada pela aplicação no banco, uma falha pode permitir alteração ou exclusão de dados.

Isso pode afetar:

* bandas cadastradas;
* álbuns;
* avaliações;
* comentários;
* permissões;
* registros administrativos.

A gravidade aumenta muito quando a aplicação usa uma conta de banco com privilégios excessivos.

## Violação de autenticação

SQL Injection também pode afetar fluxos de autenticação quando consultas de login são construídas de forma insegura.

Um login seguro nunca deve depender de SQL concatenado.

Além disso, senhas não devem ser armazenadas em texto puro. Elas devem ser protegidas com algoritmo adequado de hash de senha.

Mesmo usando hash de senha, a consulta de autenticação precisa ser parametrizada.

## Menor privilégio no banco de dados

Uma das defesas mais importantes é aplicar o princípio do menor privilégio.

A conta usada pela aplicação em produção deve ter apenas as permissões necessárias.

Por exemplo, uma API de catálogo talvez precise:

* ler bandas;
* inserir avaliações;
* atualizar comentários do próprio usuário;
* consultar dados públicos.

Mas ela provavelmente não deveria ter permissão para:

* remover tabelas;
* criar extensões;
* alterar estrutura do banco;
* acessar schemas administrativos;
* gerenciar usuários do banco;
* executar comandos privilegiados.

Mesmo se uma falha ocorrer, privilégios limitados reduzem o dano possível.

## Separar usuários de aplicação, migração e administração

Uma boa prática é separar responsabilidades.

Por exemplo:

* usuário administrador do banco: usado apenas por DBAs ou automação controlada;
* usuário de migração: usado por ferramentas como Flyway ou Liquibase;
* usuário da aplicação: usado pelo backend em execução;
* usuário de leitura: usado por relatórios quando aplicável.

A aplicação em produção não precisa se conectar com o mesmo usuário que cria tabelas e altera schema.

Essa separação ajuda a limitar danos.

## Exemplo conceitual de privilégios

Em um banco relacional, a conta da aplicação poderia receber permissões apenas nas tabelas necessárias.

Exemplo conceitual:

```sql
GRANT SELECT, INSERT, UPDATE ON bandas TO app_meucatalogo;
GRANT SELECT, INSERT, UPDATE ON avaliacoes TO app_meucatalogo;
```

Esse exemplo é apenas ilustrativo. A estratégia real depende do banco, do schema, da arquitetura, da ferramenta de migração e dos processos operacionais.

O importante é evitar que a aplicação rode com privilégios administrativos sem necessidade.

## Tratamento seguro de erros

Mensagens de erro também podem ajudar um invasor a entender a estrutura interna da aplicação.

Evite retornar detalhes internos como:

* nome de tabela;
* nome de coluna;
* stack trace;
* SQL completo;
* nome do banco;
* driver;
* caminho interno;
* detalhes de infraestrutura.

Exemplo ruim de resposta:

```json
{
  "erro": "Erro SQL na tabela bandas, coluna nome, consulta SELECT ..."
}
```

Exemplo melhor:

```json
{
  "erro": "Não foi possível processar a solicitação."
}
```

Internamente, a aplicação pode registrar um log técnico controlado, sem expor dados sensíveis para o usuário final.

## Exemplo de tratamento de erro com Spring

Um `ControllerAdvice` pode padronizar respostas de erro.

```java
package com.meucatalogomusical.config;

import org.springframework.http.HttpStatus;
import org.springframework.web.bind.annotation.*;

@RestControllerAdvice
public class ApiExceptionHandler {

    @ExceptionHandler(Exception.class)
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    public ErroResponse tratarErroInesperado(Exception exception) {
        return new ErroResponse("Não foi possível processar a solicitação.");
    }

    public record ErroResponse(String mensagem) {
    }
}
```

Esse exemplo é simples e didático.

Em uma aplicação real, também seria importante:

* registrar logs internos com correlação;
* diferenciar erros de validação;
* tratar exceções de negócio;
* evitar capturar tudo de forma genérica sem observabilidade;
* não vazar detalhes internos para o cliente.

## Logs e SQL Injection

Logs são úteis para investigação, mas podem vazar dados se forem mal utilizados.

Evite registrar:

* SQL completo com dados sensíveis;
* senhas;
* tokens;
* cookies;
* dados pessoais desnecessários;
* payloads completos enviados por usuários;
* stack traces em resposta HTTP.

Um log seguro deve permitir investigação sem transformar o arquivo de log em uma nova fonte de vazamento.

Exemplo melhor:

```java
log.warn("Falha ao pesquisar bandas. correlationId={}, motivo={}", correlationId, "erro_consulta");
```

O log deve ajudar a operação, mas não deve expor indevidamente a entrada completa do usuário.

## SQL Injection e ORMs

Usar ORM, como JPA/Hibernate, reduz o risco de SQL Injection em muitos cenários, mas não elimina o problema automaticamente.

Consultas derivadas e parâmetros nomeados ajudam bastante.

Porém, ainda é possível escrever código inseguro se o desenvolvedor montar JPQL, HQL ou SQL nativo por concatenação.

Exemplo inseguro em JPQL:

```java
String jpql = "select b from BandaEntity b where b.nome = '" + nome + "'";
```

Mesmo não sendo SQL nativo, o padrão continua perigoso.

A correção é usar parâmetros:

```java
String jpql = "select b from BandaEntity b where b.nome = :nome";
```

E vincular o valor como parâmetro.

## Native queries exigem cuidado

Às vezes, uma aplicação precisa usar SQL nativo por performance, relatórios ou recursos específicos do banco.

Isso não é proibido, mas exige disciplina.

Exemplo com Spring Data JPA:

```java
package com.meucatalogomusical.bandas.infra;

import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.Repository;
import org.springframework.data.repository.query.Param;

import java.util.List;

public interface RelatorioBandaRepository extends Repository<BandaEntity, Long> {

    @Query(value = """
            SELECT b.id, b.nome, b.genero
            FROM bandas b
            WHERE b.genero = :genero
            ORDER BY b.nome
            """, nativeQuery = true)
    List<BandaProjection> listarPorGenero(@Param("genero") String genero);
}
```

Mesmo com SQL nativo, o valor `genero` deve ser parametrizado.

O problema não é usar SQL nativo. O problema é montar SQL nativo com entrada externa concatenada.

## Boas práticas

As principais boas práticas contra SQL Injection são:

* usar consultas parametrizadas;
* evitar concatenação de SQL com entrada externa;
* usar `PreparedStatement` quando estiver com JDBC puro;
* usar parâmetros nomeados em JPQL/HQL;
* usar recursos seguros do Spring Data JPA;
* validar entradas no backend;
* usar lista permitida para valores controlados;
* mapear campos de ordenação em vez de aceitar nomes livres;
* aplicar menor privilégio no banco;
* separar usuários de aplicação, migração e administração;
* evitar mensagens de erro detalhadas para o usuário;
* evitar logs com dados sensíveis;
* revisar consultas nativas com atenção;
* testar fluxos críticos;
* manter dependências atualizadas.

## Erros comuns

### 1. Concatenar parâmetros em SQL

Esse é o erro clássico:

```java
String sql = "SELECT * FROM bandas WHERE nome = '" + nome + "'";
```

A solução é usar parâmetro:

```java
String sql = "SELECT * FROM bandas WHERE nome = ?";
```

### 2. Achar que validação sozinha resolve

Validar entrada é importante, mas não substitui consulta parametrizada.

A aplicação precisa fazer as duas coisas.

### 3. Achar que ORM elimina qualquer risco

JPA e Hibernate ajudam, mas ainda é possível escrever queries inseguras por concatenação.

### 4. Usar usuário administrador na aplicação

A aplicação não deve se conectar ao banco com conta administrativa em produção.

### 5. Expor erro SQL para o usuário

Mensagens técnicas ajudam invasores a entender a estrutura interna.

### 6. Aceitar campos de ordenação livres

Parâmetros como `orderBy`, `sort` e `direction` devem ser tratados com lista permitida.

### 7. Registrar dados sensíveis em logs

Logs inseguros podem expor informações tão perigosas quanto uma resposta HTTP mal projetada.

## Decisões de engenharia

### Quando usar consultas derivadas do Spring Data JPA

Use quando a consulta for simples e expressável pelo nome do método.

Exemplo:

```java
List<BandaEntity> findByNomeContainingIgnoreCase(String nome);
```

Vantagens:

* menos código;
* menor risco de concatenação manual;
* boa legibilidade para consultas simples.

Evite quando:

* a consulta ficar complexa demais;
* o nome do método ficar enorme;
* houver necessidade de otimização específica.

### Quando usar `@Query` com JPQL

Use quando a consulta precisa ser explícita, mas ainda trabalha com entidades.

Exemplo:

```java
@Query("""
        select b
        from BandaEntity b
        where lower(b.nome) like lower(concat('%', :nome, '%'))
        """)
List<BandaEntity> pesquisarPorNome(@Param("nome") String nome);
```

Vantagens:

* consulta clara;
* parâmetros nomeados;
* integração com JPA.

Evite quando:

* a query depende fortemente de recursos específicos do banco;
* há perda significativa de performance;
* a consulta se torna difícil de manter.

### Quando usar SQL nativo

Use quando houver uma justificativa técnica real:

* performance;
* relatórios complexos;
* uso de funções específicas do banco;
* consultas difíceis de expressar em JPQL.

Cuidados:

* parametrizar valores;
* revisar com atenção;
* evitar concatenação;
* testar performance;
* limitar privilégios;
* documentar o motivo.

### Quando usar JdbcTemplate

Use quando quiser controle maior sobre SQL e mapeamento sem usar JPA em determinado caso.

Vantagens:

* simplicidade;
* controle direto;
* boa opção para queries específicas.

Cuidados:

* sempre usar placeholders;
* nunca concatenar entrada externa;
* mapear resultados com clareza;
* manter SQL legível.

## Onde isso aparece em sistemas reais

SQL Injection pode aparecer em vários pontos:

* tela de login;
* busca por nome;
* filtros de relatório;
* exportação de dados;
* endpoints administrativos;
* consultas com ordenação dinâmica;
* dashboards;
* importação de arquivos;
* integrações externas;
* APIs públicas;
* telas de auditoria;
* pesquisas com múltiplos filtros.

Em sistemas corporativos, endpoints de relatório merecem atenção especial, porque frequentemente possuem filtros complexos e SQL nativo.

## Checklist prático

Use este checklist ao revisar código com acesso a banco.

### Consultas

* A query usa parâmetros?
* Existe concatenação de entrada externa?
* Campos de ordenação são controlados por lista permitida?
* SQL nativo foi revisado?
* JPQL/HQL usa parâmetros nomeados?
* Consultas dinâmicas foram testadas?

### Entrada

* Parâmetros de URL são validados?
* Corpo JSON é validado?
* Campos possuem tamanho máximo?
* Valores controlados usam enum ou lista permitida?
* Filtros de relatório possuem limites?

### Banco de dados

* A conta da aplicação tem menor privilégio?
* A aplicação evita usar conta administrativa?
* Usuário de migração é separado do usuário da aplicação?
* Permissões foram revisadas por ambiente?
* A aplicação tem acesso apenas ao schema necessário?

### Erros e logs

* Erros SQL não são expostos ao usuário?
* Stack traces não aparecem na resposta HTTP?
* Logs evitam dados sensíveis?
* Existe correlationId para investigação?
* Falhas suspeitas são monitoradas?

### Arquitetura

* A aplicação usa camada de repositório bem definida?
* A regra de autorização é aplicada antes de ações sensíveis?
* Consultas críticas têm testes automatizados?
* Dependências são mantidas atualizadas?

## Resumo final

SQL Injection é uma vulnerabilidade que ocorre quando entrada não confiável consegue alterar o significado de uma consulta SQL.

A principal defesa é separar código SQL de dados usando consultas parametrizadas.

Em Java e Spring, isso pode ser feito com:

* `PreparedStatement`;
* `JdbcTemplate` com placeholders;
* Spring Data JPA com queries derivadas;
* JPQL com parâmetros nomeados;
* SQL nativo parametrizado quando necessário.

Além disso, uma aplicação segura precisa combinar várias camadas de proteção:

* validação de entrada;
* lista permitida;
* menor privilégio no banco;
* tratamento seguro de erros;
* logs responsáveis;
* revisão de queries dinâmicas;
* separação entre usuários de aplicação, migração e administração.

SQL Injection não é apenas um problema antigo. É um exemplo clássico de uma regra fundamental de segurança:

> Entrada externa deve ser tratada como dado não confiável e nunca deve controlar diretamente a estrutura de comandos executados pela aplicação.

## Perguntas de revisão

### 1. O que é SQL Injection?

SQL Injection é uma vulnerabilidade em que dados não confiáveis são incorporados de forma insegura em uma consulta SQL, permitindo que a entrada altere o significado do comando executado pelo banco.

### 2. Qual é a causa mais comum de SQL Injection?

A causa mais comum é montar consultas SQL por concatenação de strings com valores recebidos de usuários ou sistemas externos.

### 3. Por que concatenação de SQL é perigosa?

Porque mistura a estrutura da consulta com dados externos. Assim, a entrada do usuário pode influenciar o comando SQL final.

### 4. Qual é a principal defesa contra SQL Injection?

A principal defesa é usar consultas parametrizadas, como `PreparedStatement`, placeholders no `JdbcTemplate` ou parâmetros nomeados em JPQL/Spring Data JPA.

### 5. Validação de entrada substitui consulta parametrizada?

Não. Validação e parametrização são complementares. Validação verifica se o dado faz sentido para a aplicação. Parametrização garante que o banco trate o valor como dado, não como comando.

### 6. ORM elimina SQL Injection automaticamente?

Não. ORMs reduzem o risco em muitos cenários, mas ainda é possível criar vulnerabilidades se o desenvolvedor montar JPQL, HQL ou SQL nativo por concatenação.

### 7. O que é menor privilégio no banco de dados?

É conceder à conta da aplicação apenas as permissões necessárias para executar suas funções, evitando privilégios administrativos desnecessários.

### 8. Por que mensagens de erro detalhadas são perigosas?

Porque podem revelar nomes de tabelas, colunas, queries, tecnologias, caminhos internos e outras informações úteis para um invasor.

### 9. Por que campos de ordenação dinâmica exigem cuidado?

Porque nomes de colunas e direções de ordenação podem alterar a estrutura da consulta. Por isso, devem ser controlados com lista permitida.

### 10. Quando SQL nativo pode ser usado?

SQL nativo pode ser usado quando houver justificativa técnica, como performance, relatórios complexos ou recursos específicos do banco. Mesmo assim, os valores devem ser parametrizados.

### 11. Qual é o papel dos logs na prevenção e resposta?

Logs ajudam a investigar falhas e comportamentos suspeitos, mas devem evitar expor dados sensíveis, tokens, cookies, senhas ou SQL completo com informações privadas.

### 12. Qual é a diferença entre parâmetro e concatenação?

No parâmetro, o valor é enviado separadamente e tratado como dado. Na concatenação, o valor vira parte da string SQL final, podendo alterar a estrutura do comando.

## Referências

* OWASP Top 10:2025 — A05: Injection.
* OWASP SQL Injection Prevention Cheat Sheet.
* OWASP SQL Injection — OWASP Community.
* Oracle Java SE 25 — `java.sql.PreparedStatement`.
* Spring Data JPA Reference Documentation — Query Methods e Named Parameters.
* Spring Boot Reference Documentation — Data Access.
* PostgreSQL Documentation — Privileges e GRANT.

[1]: https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html "SQL Injection Prevention Cheat Sheet"
