# Unchecked Input for Loop Condition e prevenção de DoS por processamento

`Unchecked Input for Loop Condition` é um tipo de vulnerabilidade em que uma entrada externa influencia a condição de um laço sem validação adequada.

Em termos simples, acontece quando o usuário consegue controlar, direta ou indiretamente, quantas vezes a aplicação vai repetir uma operação.

Em Java/Spring, esse risco costuma aparecer em pontos como:

- importação de planilhas;
- processamento de CSV;
- leitura de arquivos XLSX;
- parsing de documentos;
- loops sobre linhas e colunas;
- geração de relatórios;
- processamento de imagens;
- criação de PDFs;
- validação de payloads grandes;
- repetição baseada em parâmetro HTTP;
- paginação sem limite;
- loops aninhados;
- retries sem limite;
- conversão de arquivos;
- algoritmos de hashing, encoding ou transformação;
- uso de bibliotecas que carregam arquivo inteiro em memória.

Esse tipo de finding está muito relacionado a **DoS por processamento**.

DoS significa Denial of Service, ou negação de serviço. A ideia é tornar a aplicação indisponível ou degradada, consumindo CPU, memória, disco, conexões, threads ou tempo de processamento.

No caso de `Unchecked Input for Loop Condition`, o risco é:

> Uma entrada externa faz a aplicação trabalhar mais do que deveria.

Na aplicação fictícia `meucatalogomusical.com`, imagine uma funcionalidade para importar uma planilha com bandas, álbuns e gêneros musicais. Um usuário autorizado envia uma planilha pequena com bandas como **Savatage**, **Type O Negative** e **Children of Bodom**. Tudo funciona.

Mas se alguém enviar uma planilha enorme, com milhares de linhas, muitas colunas, células gigantes ou estrutura maliciosa, a aplicação pode consumir CPU e memória até afetar o serviço.

O objetivo deste artigo é mostrar como evitar esse tipo de problema em aplicações Java/Spring.

## O problema real que esse conceito resolve

Muitos desenvolvedores validam se um arquivo existe, se a extensão parece correta e se o endpoint está autenticado.

Mas esquecem de validar o custo de processamento.

Uma entrada pode ser tecnicamente válida e ainda assim perigosa.

Exemplo:

```text
Arquivo XLSX válido.
Extensão correta.
Usuário autenticado.
Endpoint permitido.
Mas o arquivo possui linhas, colunas e células demais.
```

Nesse caso, o problema não é necessariamente acesso indevido.

O problema é consumo excessivo de recursos.

Em importações, relatórios e parsers, a aplicação pode executar loops como:

```java
for (int linha = 0; linha <= sheet.getLastRowNum(); linha++) {
    Row row = sheet.getRow(linha);

    for (int coluna = 0; coluna < row.getLastCellNum(); coluna++) {
        Cell cell = row.getCell(coluna);
        processar(cell);
    }
}
```

Se `getLastRowNum()` e `getLastCellNum()` forem influenciados pelo arquivo enviado, o usuário passa a influenciar o número de iterações.

Esse é exatamente o tipo de padrão que uma ferramenta SAST pode apontar.

## Antes de entrar nos detalhes: uma ideia simples

Pense em uma portaria.

Uma pessoa chega com uma caixa para ser conferida.

A portaria pode aceitar conferir a caixa, mas precisa impor limites:

```text
Tamanho máximo da caixa.
Peso máximo.
Quantidade máxima de itens.
Tempo máximo de atendimento.
Tipo permitido de material.
Responsável pela entrega.
```

Sem limites, alguém pode chegar com um caminhão inteiro e exigir que a portaria confira tudo imediatamente.

Com arquivos é parecido.

O usuário pode enviar uma planilha para importar bandas de Hard Rock.

Mas a aplicação precisa impor limites:

```text
Tamanho máximo do arquivo.
Quantidade máxima de linhas.
Quantidade máxima de colunas.
Quantidade máxima de células.
Tamanho máximo de texto por célula.
Tempo máximo de processamento.
Quantidade máxima de erros coletados.
Quantidade máxima de registros criados.
```

A regra é:

> Entrada externa não deve controlar processamento ilimitado.

## Definição técnica

`Unchecked Input for Loop Condition` ocorre quando um valor derivado de entrada externa é usado para controlar uma condição de loop sem validação de faixa, limite ou custo.

Em SAST, o fluxo costuma aparecer assim:

```text
source: arquivo, request, parâmetro, payload ou entrada externa
    -> parser
    -> cálculo de quantidade
sink: condição de loop
```

Exemplos de source:

```java
@RequestParam MultipartFile file
```

```java
@RequestBody ImportacaoRequest request
```

```java
@RequestParam int quantidade
```

```java
sheet.getLastRowNum()
```

```java
row.getLastCellNum()
```

```java
request.getItens().size()
```

```java
headers.size()
```

Exemplos de sink:

```java
for (int i = 0; i < quantidade; i++) {
    processar(i);
}
```

```java
while (iterator.hasNext()) {
    processar(iterator.next());
}
```

```java
for (int row = 0; row <= sheet.getLastRowNum(); row++) {
    processarLinha(row);
}
```

```java
for (int col = 0; col < row.getLastCellNum(); col++) {
    processarCelula(col);
}
```

O problema não é usar loop.

O problema é permitir que a entrada externa defina o tamanho do loop sem controle adequado.

## Relação com DoS por processamento

DoS por processamento acontece quando uma requisição ou entrada faz o servidor gastar muitos recursos.

Isso pode afetar:

- CPU;
- memória;
- disco;
- rede;
- conexões;
- threads;
- pool de banco;
- tempo de resposta;
- fila de processamento;
- outros usuários.

A OWASP Denial of Service Cheat Sheet recomenda, entre outras medidas, limitar tamanho e extensão de uploads para evitar esgotamento de recursos em funções que usam o arquivo como entrada, como processamento de imagens, criação de PDFs e outras operações custosas.

No caso de importação de arquivos, o risco não está apenas no tamanho do arquivo em bytes.

Também existe risco em:

- quantidade de linhas;
- quantidade de colunas;
- quantidade de células;
- tamanho de cada célula;
- fórmulas;
- estilos;
- arquivos compactados internamente;
- estruturas XML grandes;
- número de abas;
- número de erros coletados;
- número de objetos criados em memória;
- complexidade da validação por linha.

Uma planilha aparentemente pequena pode gerar muito trabalho se for processada sem limites.

## Exemplo inseguro: importação de planilha

Imagine uma funcionalidade em `meucatalogomusical.com` para importar bandas em lote.

O usuário envia uma planilha com bandas como **Manowar**, **Gamma Ray** e **Blue Öyster Cult**.

Controller:

```java
package com.meucatalogomusical.importacao.api;

import org.springframework.http.HttpStatus;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.multipart.MultipartFile;

@RestController
@RequestMapping("/api/importacoes")
public class ImportacaoBandasController {

    private final ImportacaoBandasService importacaoBandasService;

    public ImportacaoBandasController(ImportacaoBandasService importacaoBandasService) {
        this.importacaoBandasService = importacaoBandasService;
    }

    @PostMapping("/bandas")
    @ResponseStatus(HttpStatus.ACCEPTED)
    public ImportacaoBandasResponse importar(@RequestParam MultipartFile file) {
        return importacaoBandasService.importar(file);
    }
}
```

Service inseguro:

```java
package com.meucatalogomusical.importacao.application;

import org.springframework.stereotype.Service;
import org.springframework.web.multipart.MultipartFile;

@Service
public class ImportacaoBandasService {

    private final XlsxBandasParser parser;

    public ImportacaoBandasService(XlsxBandasParser parser) {
        this.parser = parser;
    }

    public ImportacaoBandasResponse importar(MultipartFile file) {
        int totalImportado = parser.parse(file).size();

        return new ImportacaoBandasResponse(totalImportado);
    }
}
```

Parser inseguro:

```java
package com.meucatalogomusical.importacao.infra;

import org.apache.poi.ss.usermodel.*;
import org.springframework.stereotype.Component;
import org.springframework.web.multipart.MultipartFile;

import java.util.ArrayList;
import java.util.List;

@Component
public class XlsxBandasParser {

    public List<BandaImportada> parse(MultipartFile file) {
        try (Workbook workbook = WorkbookFactory.create(file.getInputStream())) {
            Sheet sheet = workbook.getSheetAt(0);
            List<BandaImportada> bandas = new ArrayList<>();

            for (int r = 1; r <= sheet.getLastRowNum(); r++) {
                Row row = sheet.getRow(r);

                if (row == null) {
                    continue;
                }

                String nome = row.getCell(0).getStringCellValue();
                String genero = row.getCell(1).getStringCellValue();

                bandas.add(new BandaImportada(nome, genero));
            }

            return bandas;
        } catch (Exception ex) {
            throw new IllegalStateException("Não foi possível importar a planilha.", ex);
        }
    }
}
```

Esse parser tem um problema: o loop depende diretamente de `sheet.getLastRowNum()`.

Se a planilha tiver muitas linhas, a aplicação tentará processar todas.

O SAST pode apontar algo como:

```text
Query: Unchecked Input for Loop Condition
Source: MultipartFile file
Sink: for (int r = 1; r <= sheet.getLastRowNum(); r++)
```

O problema não é o método `getLastRowNum()` em si.

O problema é usá-lo sem limite de negócio.

## Por que isso é perigoso?

Esse tipo de código pode causar:

- consumo alto de CPU;
- consumo alto de memória;
- aumento no tempo de resposta;
- travamento de threads;
- esgotamento do pool de requisições;
- queda do serviço;
- lentidão para outros usuários;
- timeouts;
- falhas em cadeia.

Além disso, o custo pode crescer com loops aninhados.

Exemplo:

```java
for (int r = 0; r <= sheet.getLastRowNum(); r++) {
    Row row = sheet.getRow(r);

    for (int c = 0; c < row.getLastCellNum(); c++) {
        processar(row.getCell(c));
    }
}
```

Se houver 10.000 linhas e 100 colunas, são até 1.000.000 de células avaliadas.

Se cada célula fizer validações, consultas ou alocações, o impacto cresce rapidamente.

## Validação de upload no Spring Boot

A primeira camada é limitar tamanho de upload.

Exemplo:

```yaml
spring:
  servlet:
    multipart:
      max-file-size: 2MB
      max-request-size: 3MB
```

O Spring Boot possui propriedades para limitar tamanho máximo de arquivo e tamanho máximo da requisição multipart; a documentação da API `MultipartProperties` descreve `max-file-size` como o tamanho máximo permitido para arquivos enviados e `max-request-size` como o tamanho máximo permitido para requisições `multipart/form-data`.

Mas isso não basta.

Um arquivo de 2MB ainda pode conter uma estrutura que exige muito processamento.

Por isso, precisamos de limites internos.

## Criando limites de importação

Podemos transformar limites em configuração de aplicação.

```yaml
app:
  importacao-bandas:
    max-rows: 1000
    max-columns: 10
    max-cells: 10000
    max-cell-text-length: 200
```

Classe de propriedades:

```java
package com.meucatalogomusical.importacao.config;

import org.springframework.boot.context.properties.ConfigurationProperties;

@ConfigurationProperties(prefix = "app.importacao-bandas")
public record ImportacaoBandasProperties(
        int maxRows,
        int maxColumns,
        int maxCells,
        int maxCellTextLength
) {

    public ImportacaoBandasProperties {
        if (maxRows <= 0) {
            throw new IllegalArgumentException("maxRows deve ser maior que zero.");
        }

        if (maxColumns <= 0) {
            throw new IllegalArgumentException("maxColumns deve ser maior que zero.");
        }

        if (maxCells <= 0) {
            throw new IllegalArgumentException("maxCells deve ser maior que zero.");
        }

        if (maxCellTextLength <= 0) {
            throw new IllegalArgumentException("maxCellTextLength deve ser maior que zero.");
        }
    }
}
```

Configuração:

```java
package com.meucatalogomusical.importacao.config;

import org.springframework.boot.context.properties.EnableConfigurationProperties;
import org.springframework.context.annotation.Configuration;

@Configuration
@EnableConfigurationProperties(ImportacaoBandasProperties.class)
public class ImportacaoBandasConfig {
}
```

Agora os limites são explícitos, revisáveis e testáveis.

## Parser defensivo com limites

Vamos melhorar o parser.

DTO:

```java
package com.meucatalogomusical.importacao.infra;

public record BandaImportada(
        String nome,
        String genero
) {
}
```

Parser com limites:

```java
package com.meucatalogomusical.importacao.infra;

import com.meucatalogomusical.importacao.config.ImportacaoBandasProperties;
import org.apache.poi.ss.usermodel.*;
import org.springframework.stereotype.Component;
import org.springframework.web.multipart.MultipartFile;

import java.util.ArrayList;
import java.util.List;

@Component
public class XlsxBandasParser {

    private final ImportacaoBandasProperties properties;

    public XlsxBandasParser(ImportacaoBandasProperties properties) {
        this.properties = properties;
    }

    public List<BandaImportada> parse(MultipartFile file) {
        validarArquivo(file);

        try (Workbook workbook = WorkbookFactory.create(file.getInputStream())) {
            Sheet sheet = workbook.getSheetAt(0);

            validarQuantidadeDeLinhas(sheet);

            List<BandaImportada> bandas = new ArrayList<>();
            DataFormatter formatter = new DataFormatter();

            int cellsProcessed = 0;

            for (int rowIndex = 1; rowIndex <= sheet.getLastRowNum(); rowIndex++) {
                if (rowIndex > properties.maxRows()) {
                    throw new IllegalArgumentException("Planilha excede o limite máximo de linhas.");
                }

                Row row = sheet.getRow(rowIndex);

                if (row == null) {
                    continue;
                }

                validarQuantidadeDeColunas(row);

                cellsProcessed += Math.min(row.getLastCellNum(), properties.maxColumns());

                if (cellsProcessed > properties.maxCells()) {
                    throw new IllegalArgumentException("Planilha excede o limite máximo de células processadas.");
                }

                String nome = lerTexto(row, 0, formatter);
                String genero = lerTexto(row, 1, formatter);

                if (!nome.isBlank()) {
                    bandas.add(new BandaImportada(nome, genero));
                }
            }

            return bandas;
        } catch (IllegalArgumentException ex) {
            throw ex;
        } catch (Exception ex) {
            throw new IllegalStateException("Não foi possível importar a planilha.", ex);
        }
    }

    private void validarArquivo(MultipartFile file) {
        if (file == null || file.isEmpty()) {
            throw new IllegalArgumentException("Arquivo é obrigatório.");
        }
    }

    private void validarQuantidadeDeLinhas(Sheet sheet) {
        if (sheet.getLastRowNum() > properties.maxRows()) {
            throw new IllegalArgumentException("Planilha excede o limite máximo de linhas.");
        }
    }

    private void validarQuantidadeDeColunas(Row row) {
        short lastCellNum = row.getLastCellNum();

        if (lastCellNum > properties.maxColumns()) {
            throw new IllegalArgumentException("Planilha excede o limite máximo de colunas.");
        }
    }

    private String lerTexto(Row row, int columnIndex, DataFormatter formatter) {
        Cell cell = row.getCell(columnIndex);

        if (cell == null) {
            return "";
        }

        String value = formatter.formatCellValue(cell).trim();

        if (value.length() > properties.maxCellTextLength()) {
            throw new IllegalArgumentException("Célula excede o tamanho máximo de texto.");
        }

        return value;
    }
}
```

Esse parser agora possui limites explícitos.

Ele não permite que `sheet.getLastRowNum()` ou `row.getLastCellNum()` controlem processamento ilimitado.

## O que melhorou?

A versão defensiva aplica limites em várias camadas:

- arquivo obrigatório;
- limite de upload via Spring;
- limite máximo de linhas;
- limite máximo de colunas;
- limite máximo de células processadas;
- limite máximo de texto por célula;
- falha fechada quando o limite é excedido;
- mensagens controladas;
- propriedades configuráveis;
- código mais fácil de justificar em SAST.

A diferença para o scanner e para o revisor humano é clara:

```text
Antes:
sheet.getLastRowNum() controla o loop.

Depois:
sheet.getLastRowNum() é validado contra maxRows antes e durante o processamento.
```

## Limitar antes ou durante o loop?

Idealmente, faça os dois quando o custo permitir.

Antes:

```java
if (sheet.getLastRowNum() > properties.maxRows()) {
    throw new IllegalArgumentException("Planilha excede o limite máximo de linhas.");
}
```

Durante:

```java
if (rowIndex > properties.maxRows()) {
    throw new IllegalArgumentException("Planilha excede o limite máximo de linhas.");
}
```

A validação antes deixa o erro claro.

A validação durante protege contra mudanças futuras no código e deixa o limite próximo ao sink.

Isso é útil em segurança:

> Controles próximos ao sink são mais fáceis de auditar.

## Cuidado com APIs que carregam tudo em memória

Ao usar Apache POI, é preciso conhecer o custo da API.

A documentação do Apache POI informa que arquivos XLSX são baseados em XML e têm maior consumo de memória em comparação aos formatos binários antigos, e recomenda APIs de event model para leitura quando o objetivo é apenas processar dados da planilha.

Isso significa que limitar linhas dentro do loop é importante, mas pode não bastar se a biblioteca já carregou uma estrutura enorme em memória antes do loop.

Para arquivos grandes, avalie:

- event model;
- streaming;
- limites de upload menores;
- processamento assíncrono;
- rejeição antecipada;
- filas;
- quotas;
- timeout;
- validação externa;
- arquitetura de batch.

A escolha depende do volume real esperado.

## Exemplo com CSV

Para CSV, o mesmo princípio se aplica.

Parser inseguro:

```java
public List<String> parse(List<String> linhas) {
    List<String> resultado = new ArrayList<>();

    for (int i = 0; i < linhas.size(); i++) {
        resultado.add(linhas.get(i));
    }

    return resultado;
}
```

Se `linhas` veio de um arquivo enviado pelo usuário, `linhas.size()` é controlado pela entrada.

Parser com limite:

```java
public List<String> parse(List<String> linhas, int maxLinhas) {
    if (linhas.size() > maxLinhas) {
        throw new IllegalArgumentException("Arquivo excede o limite máximo de linhas.");
    }

    List<String> resultado = new ArrayList<>();

    for (int i = 0; i < linhas.size(); i++) {
        resultado.add(linhas.get(i));
    }

    return resultado;
}
```

Ainda melhor: leia linha por linha e pare quando o limite for excedido, em vez de carregar tudo antes.

```java
package com.meucatalogomusical.importacao.infra;

import org.springframework.stereotype.Component;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.Reader;
import java.util.ArrayList;
import java.util.List;

@Component
public class CsvBandasParser {

    private static final int MAX_ROWS = 1000;
    private static final int MAX_LINE_LENGTH = 500;

    public List<String> parse(Reader reader) {
        List<String> linhas = new ArrayList<>();

        try (BufferedReader bufferedReader = new BufferedReader(reader)) {
            String line;
            int count = 0;

            while ((line = bufferedReader.readLine()) != null) {
                count++;

                if (count > MAX_ROWS) {
                    throw new IllegalArgumentException("CSV excede o limite máximo de linhas.");
                }

                if (line.length() > MAX_LINE_LENGTH) {
                    throw new IllegalArgumentException("Linha do CSV excede o tamanho máximo permitido.");
                }

                linhas.add(line);
            }

            return linhas;
        } catch (IOException ex) {
            throw new IllegalStateException("Não foi possível ler o CSV.", ex);
        }
    }
}
```

Esse desenho evita carregar todas as linhas antes de validar quantidade.

## Loops aninhados

O risco aumenta quando há loops aninhados.

Exemplo:

```java
for (int albumIndex = 0; albumIndex < albuns.size(); albumIndex++) {
    for (int faixaIndex = 0; faixaIndex < albuns.get(albumIndex).faixas().size(); faixaIndex++) {
        processarFaixa(albuns.get(albumIndex).faixas().get(faixaIndex));
    }
}
```

Se `albuns` e `faixas` vêm de entrada externa, o custo total é aproximadamente:

```text
quantidade de álbuns x quantidade de faixas por álbum
```

Para importar discografias de bandas como **Blind Guardian**, **Stratovarius** e **Uriah Heep**, isso pode parecer natural.

Mas se o payload permitir milhares de álbuns com milhares de faixas, o processamento explode.

Defina limites por nível:

```java
public record ImportacaoDiscografiaRequest(
        List<AlbumRequest> albuns
) {
    public void validar() {
        if (albuns == null || albuns.isEmpty()) {
            throw new IllegalArgumentException("Lista de álbuns é obrigatória.");
        }

        if (albuns.size() > 100) {
            throw new IllegalArgumentException("Quantidade máxima de álbuns excedida.");
        }

        int totalFaixas = 0;

        for (AlbumRequest album : albuns) {
            album.validar();
            totalFaixas += album.faixas().size();

            if (totalFaixas > 1500) {
                throw new IllegalArgumentException("Quantidade total de faixas excedida.");
            }
        }
    }
}
```

```java
public record AlbumRequest(
        String titulo,
        List<String> faixas
) {
    public void validar() {
        if (titulo == null || titulo.isBlank()) {
            throw new IllegalArgumentException("Título do álbum é obrigatório.");
        }

        if (faixas == null || faixas.isEmpty()) {
            throw new IllegalArgumentException("Lista de faixas é obrigatória.");
        }

        if (faixas.size() > 30) {
            throw new IllegalArgumentException("Quantidade máxima de faixas por álbum excedida.");
        }
    }
}
```

Limites por nível evitam explosão combinatória.

## Não acumule todos os erros sem limite

Outro erro comum é coletar todos os erros de uma importação.

Exemplo perigoso:

```java
List<String> erros = new ArrayList<>();

for (LinhaImportacao linha : linhas) {
    try {
        validar(linha);
    } catch (Exception ex) {
        erros.add(ex.getMessage());
    }
}
```

Se o arquivo tiver muitos erros, a lista `erros` pode crescer muito.

Defina limite de erros coletados:

```java
private static final int MAX_ERRORS = 100;

private void adicionarErro(List<String> erros, String mensagem) {
    if (erros.size() >= MAX_ERRORS) {
        return;
    }

    erros.add(mensagem);
}
```

Ou interrompa:

```java
if (erros.size() >= MAX_ERRORS) {
    throw new IllegalArgumentException("Importação interrompida por excesso de erros.");
}
```

A estratégia depende da regra de negócio.

O ponto é: até o erro precisa ter limite.

## Não faça consulta ao banco dentro de loop sem controle

Outro risco é multiplicar chamadas ao banco.

Exemplo perigoso:

```java
for (BandaImportada banda : bandas) {
    boolean existe = bandaRepository.existsByNome(banda.nome());

    if (!existe) {
        bandaRepository.save(banda);
    }
}
```

Se a entrada tiver 10.000 bandas, são muitas consultas e escritas.

Melhor:

- limitar quantidade de linhas;
- carregar existentes em lote;
- salvar em batch;
- usar transações com cuidado;
- evitar loops N+1;
- processar assíncrono quando necessário.

Exemplo conceitual:

```java
Set<String> nomes = bandas.stream()
        .map(BandaImportada::nome)
        .collect(Collectors.toSet());

Set<String> existentes = bandaRepository.findNomesExistentes(nomes);

List<BandaImportada> novas = bandas.stream()
        .filter(banda -> !existentes.contains(banda.nome()))
        .toList();

bandaRepository.saveAll(novas);
```

Mesmo com essa melhoria, o limite de `bandas` continua necessário.

## Tempo máximo de processamento

Além de limitar quantidade, considere tempo.

Em processamento síncrono via HTTP, a requisição não deve ficar presa por muito tempo.

Para operações pesadas, prefira:

```text
receber arquivo
    -> validar tamanho e formato básico
    -> registrar job
    -> processar assíncrono
    -> devolver protocolo/status
```

Exemplo de resposta:

```json
{
  "importacaoId": 123,
  "status": "PENDING"
}
```

Depois:

```text
GET /api/importacoes/123
```

Resposta:

```json
{
  "importacaoId": 123,
  "status": "SUCCESS",
  "totalProcessado": 350
}
```

Essa arquitetura evita que uma requisição HTTP longa consuma thread por muito tempo.

## Rate limiting e quotas

Mesmo com limites por arquivo, um usuário pode enviar muitos arquivos pequenos.

Por isso, pense também em:

- limite por usuário;
- limite por tenant;
- limite por IP;
- limite por período;
- quantidade máxima de importações simultâneas;
- fila;
- backpressure;
- circuit breaker;
- controle de concorrência.

A OWASP REST Security Cheat Sheet recomenda validação implícita com tipos fortes e faixas fixas em parâmetros de API e cita API keys como mecanismo que pode reduzir impacto de ataques de DoS.

Em sistemas corporativos, a combinação é:

```text
limite por entrada + limite por usuário + limite por infraestrutura
```

## Diferença entre validação funcional e proteção contra DoS

Validação funcional pergunta:

```text
A banda possui nome?
O gênero é válido?
O ano está correto?
```

Proteção contra DoS pergunta:

```text
Quantas bandas podem vir de uma vez?
Quantas colunas posso processar?
Quantos erros vou armazenar?
Quanto tempo posso gastar?
Quanto de memória posso alocar?
Quantas importações simultâneas aceito?
```

As duas são necessárias.

Um payload pode passar na validação funcional e ainda assim ser perigoso para disponibilidade.

## Falso positivo em Unchecked Input for Loop Condition

Nem todo finding é vulnerabilidade real.

No caso do Databridge, por exemplo, a análise concluiu que o request influenciava o documento processado, mas não conseguia produzir número elevado de iterações; no fluxo público alcançável, o laço executava uma vez ou nem era alcançado, e os valores usados limitavam `repeatCount` a `1..2`. Mesmo assim, foi aplicado hardening defensivo para tornar os limites explícitos.

Esse é um bom exemplo de triagem madura.

A pergunta não é apenas:

```text
Existe dado externo perto de um loop?
```

A pergunta correta é:

```text
Esse dado externo consegue aumentar o número de iterações de forma relevante?
```

Se não consegue, pode ser falso positivo.

Mas ainda pode valer ajustar o código para deixar essa restrição clara.

## Exemplo de hardening em loop pequeno

Código pouco claro:

```java
for (int i = 0; i < repeatCount; i++) {
    resultado.append(valor);
}
```

Se `repeatCount` é derivado de entrada, o SAST pode suspeitar.

Código mais explícito:

```java
private static final int MIN_REPEAT = 1;
private static final int MAX_REPEAT = 2;

private String repetirComLimite(String valor, int repeatCount) {
    if (repeatCount < MIN_REPEAT || repeatCount > MAX_REPEAT) {
        throw new IllegalArgumentException("Quantidade de repetição fora do limite permitido.");
    }

    return valor.repeat(repeatCount);
}
```

Aqui o limite está documentado no código.

Além disso, `String.repeat` deixa a intenção mais clara do que um `for` manual para esse caso específico.

Isso não significa que `String.repeat` seja sempre a solução. Ele só faz sentido quando o limite é pequeno e validado.

## Quando é vulnerabilidade real?

Tende a ser vulnerabilidade real quando:

- entrada externa controla quantidade de iterações;
- não há limite máximo;
- o loop pode ser grande;
- há loops aninhados;
- cada iteração faz trabalho custoso;
- há alocação de memória por iteração;
- há chamada de banco por iteração;
- há chamada HTTP por iteração;
- erros são acumulados sem limite;
- arquivo é processado inteiro sem limite;
- endpoint é síncrono;
- não há rate limiting;
- não há timeout;
- não há quota.

Exemplo:

```java
for (int i = 0; i < request.quantidade(); i++) {
    gerarRelatorio();
}
```

Se `quantidade` vem do usuário e não tem limite, é risco real.

## Quando pode ser falso positivo?

Pode ser falso positivo quando:

- o valor já é limitado por enum;
- o valor é matematicamente limitado;
- o método é privado e só recebe valores controlados;
- o loop sempre executa quantidade pequena;
- o caminho apontado pelo scanner é inalcançável;
- a entrada não consegue influenciar o contador;
- a biblioteca retorna valor limitado por contrato;
- existe validação que o scanner não reconhece.

Mas falso positivo precisa de evidência.

Não basta dizer:

```text
Esse caso não acontece.
```

A justificativa deve mostrar:

- source;
- sink;
- fluxo real;
- limite demonstrável;
- testes;
- risco residual.

## Como escrever justificativa técnica

Justificativa fraca:

```text
Não é problema porque o endpoint é interno.
```

Justificativa forte:

```text
O source apontado é o arquivo enviado ao endpoint de importação. O sink apontado é o loop sobre linhas da planilha. A versão corrigida rejeita arquivos com mais de 1000 linhas antes do loop e também valida rowIndex durante a iteração. Colunas são limitadas a 10, células processadas a 10000 e texto por célula a 200 caracteres. O upload multipart possui limite de 2MB. Testes cobrem rejeição de excesso de linhas, colunas e células. Classificação: vulnerabilidade mitigada por validação de recurso e limites explícitos.
```

A segunda justificativa é técnica e auditável.

## Testes de limites

Teste de parser:

```java
package com.meucatalogomusical.importacao.infra;

import com.meucatalogomusical.importacao.config.ImportacaoBandasProperties;
import org.junit.jupiter.api.Test;

import static org.junit.jupiter.api.Assertions.assertThrows;

class XlsxBandasParserTest {

    @Test
    void deveRejeitarPlanilhaComMuitasLinhas() {
        ImportacaoBandasProperties properties =
                new ImportacaoBandasProperties(1000, 10, 10000, 200);

        XlsxBandasParser parser = new XlsxBandasParser(properties);

        // Neste exemplo didático, imagine que criarPlanilhaComLinhas monta
        // um MultipartFile de teste com quantidade acima do limite.
        var arquivo = PlanilhaTesteFactory.criarPlanilhaComLinhas(1001);

        assertThrows(IllegalArgumentException.class,
                () -> parser.parse(arquivo));
    }

    @Test
    void deveRejeitarPlanilhaComMuitasColunas() {
        ImportacaoBandasProperties properties =
                new ImportacaoBandasProperties(1000, 10, 10000, 200);

        XlsxBandasParser parser = new XlsxBandasParser(properties);

        var arquivo = PlanilhaTesteFactory.criarPlanilhaComColunas(11);

        assertThrows(IllegalArgumentException.class,
                () -> parser.parse(arquivo));
    }

    @Test
    void deveRejeitarCelulaComTextoMuitoGrande() {
        ImportacaoBandasProperties properties =
                new ImportacaoBandasProperties(1000, 10, 10000, 200);

        XlsxBandasParser parser = new XlsxBandasParser(properties);

        var arquivo = PlanilhaTesteFactory.criarPlanilhaComTextoGrande(201);

        assertThrows(IllegalArgumentException.class,
                () -> parser.parse(arquivo));
    }
}
```

O importante é testar os limites, não apenas o caminho feliz.

## Teste de limite matemático

Para um método de repetição pequena:

```java
package com.meucatalogomusical.shared.util;

import org.junit.jupiter.api.Test;

import static org.junit.jupiter.api.Assertions.*;

class RepetidorSeguroTest {

    private final RepetidorSeguro repetidor = new RepetidorSeguro();

    @Test
    void deveAceitarRepeticaoDentroDoLimite() {
        String resultado = repetidor.repetirComLimite("Metal", 2);

        assertEquals("MetalMetal", resultado);
    }

    @Test
    void deveRejeitarRepeticaoAcimaDoLimite() {
        assertThrows(IllegalArgumentException.class,
                () -> repetidor.repetirComLimite("Metal", 1000));
    }
}
```

Classe:

```java
package com.meucatalogomusical.shared.util;

public class RepetidorSeguro {

    private static final int MIN_REPEAT = 1;
    private static final int MAX_REPEAT = 2;

    public String repetirComLimite(String valor, int repeatCount) {
        if (valor == null) {
            throw new IllegalArgumentException("Valor é obrigatório.");
        }

        if (repeatCount < MIN_REPEAT || repeatCount > MAX_REPEAT) {
            throw new IllegalArgumentException("Quantidade de repetição fora do limite permitido.");
        }

        return valor.repeat(repeatCount);
    }
}
```

Esse tipo de teste ajuda a fechar findings de SAST com evidência.

## Boas práticas

### 1. Defina limites explícitos

Não deixe loops dependerem apenas do tamanho da entrada.

### 2. Valide antes do loop

Quando possível, rejeite entradas que excedem limites antes de começar o processamento.

### 3. Valide durante o loop

Mantenha guardas próximas da condição de iteração.

### 4. Limite upload

Use `spring.servlet.multipart.max-file-size` e `spring.servlet.multipart.max-request-size`.

### 5. Limite linhas, colunas e células

Em planilhas, limite todos os eixos.

### 6. Limite tamanho de texto

Células ou campos gigantes também consomem memória.

### 7. Limite quantidade de erros

Não acumule lista infinita de erros.

### 8. Evite N+1 dentro do loop

Não faça consulta remota ou de banco por item sem necessidade.

### 9. Use processamento assíncrono para cargas maiores

Requisição HTTP síncrona não é ideal para processamento pesado.

### 10. Crie testes de limite

Testes de segurança precisam cobrir excesso de linhas, colunas, células e tamanho.

## Erros comuns

### 1. Confiar apenas no tamanho do arquivo

Um arquivo pequeno ainda pode gerar processamento caro.

### 2. Confiar apenas na extensão

Extensão não diz nada sobre custo de processamento.

### 3. Usar `getLastRowNum()` diretamente no loop

Esse valor pode refletir estrutura controlada pelo arquivo.

### 4. Ignorar colunas

Limitar linhas sem limitar colunas pode manter custo alto.

### 5. Acumular todos os erros

A lista de erros também consome memória.

### 6. Fazer chamadas externas dentro do loop

Isso multiplica latência e risco de indisponibilidade.

### 7. Processar tudo de forma síncrona

Importações grandes devem ir para job assíncrono.

### 8. Não limitar concorrência

Várias importações pequenas simultâneas também podem causar DoS.

### 9. Tratar falso positivo sem evidência

Falso positivo precisa de prova técnica.

### 10. Criar gambiarra para calar SAST

A correção deve reduzir risco real ou deixar limite real explícito.

## Decisões de engenharia

## Quando processar síncrono?

Processe síncrono quando:

- arquivo é pequeno;
- limite é baixo;
- tempo esperado é curto;
- usuário precisa de resposta imediata;
- impacto é controlado;
- há testes de limite.

Exemplo: importar até 100 linhas simples.

## Quando processar assíncrono?

Processe assíncrono quando:

- arquivo pode ser grande;
- validação é pesada;
- há escrita em lote;
- há chamadas externas;
- há risco de timeout;
- usuários podem enviar muitas importações;
- o processamento precisa de retry;
- o resultado pode ser consultado depois.

## Quando rejeitar arquivo?

Rejeite quando:

- excede tamanho;
- excede linhas;
- excede colunas;
- excede células;
- contém texto grande demais;
- extensão não é permitida;
- formato não é suportado;
- estrutura exige processamento fora do limite.

## Quando usar streaming?

Use streaming ou event model quando:

- arquivos podem ser grandes;
- carregar workbook inteiro em memória é custoso;
- o processamento é sequencial;
- você não precisa editar o arquivo inteiro;
- a biblioteca suporta leitura incremental.

Para Apache POI, a documentação indica event model para leitura quando o objetivo é apenas processar dados, e informa que XSSF para XLSX pode consumir mais memória.

## Quando classificar como Not Exploitable?

Considere quando:

- o loop existe;
- o scanner liga source ao loop;
- mas há limite matemático pequeno;
- ou o caminho é inalcançável;
- ou a entrada não controla o contador;
- ou há validação efetiva não reconhecida;
- e existem testes/evidências.

## Onde isso aparece em sistemas reais

Esse tipo de vulnerabilidade aparece em:

- importação de Excel;
- importação de CSV;
- geração de relatórios;
- upload de imagens;
- processamento de PDF;
- processamento de XML;
- leitura de JSON grande;
- parsing de arquivos de integração;
- validação de listas;
- processamento de lotes;
- endpoints de bulk insert;
- cálculo de hashes;
- algoritmos de normalização;
- retries;
- integrações com sistemas legados.

Em Java/Spring, pontos comuns são:

- controllers com `MultipartFile`;
- services de importação;
- parsers `XlsxToMapParser`;
- loops usando `lastRowNum`;
- loops usando `lastCellNum`;
- conversores de documento;
- classes utilitárias;
- processamento em lote;
- validação de listas de DTOs.

## Checklist prático

### Entrada

- O arquivo tem tamanho máximo?
- A extensão é permitida?
- O content type é validado?
- O payload JSON tem tamanho controlado?
- Listas têm quantidade máxima?
- Strings têm tamanho máximo?

### Planilha

- Há limite de linhas?
- Há limite de colunas?
- Há limite de células?
- Há limite de abas?
- Há limite de texto por célula?
- Fórmulas são avaliadas?
- O parser carrega tudo em memória?

### Loop

- A condição depende de entrada externa?
- Existe limite máximo?
- O limite está perto do loop?
- Há loops aninhados?
- O custo por iteração é alto?
- Existe alocação por iteração?
- Existe chamada ao banco por iteração?
- Existe chamada HTTP por iteração?

### Erros

- A lista de erros tem limite?
- Mensagens de erro são curtas?
- O processamento para após excesso de erros?
- Erros não expõem path, stack trace ou dados sensíveis?

### Operação

- Há rate limiting?
- Há limite por usuário?
- Há limite por tenant?
- Há fila?
- Há controle de concorrência?
- Há timeout?
- Há monitoramento de tempo de processamento?

### SAST

- Source e sink foram identificados?
- O scanner aponta loop real?
- O contador é controlável?
- Existe validação?
- Existe teste?
- É vulnerabilidade real, risco mitigado ou falso positivo?

## Resumo final

`Unchecked Input for Loop Condition` acontece quando entrada externa influencia a quantidade de iterações de um loop sem validação adequada.

Esse tipo de problema pode causar DoS por processamento, especialmente em importações, parsers, planilhas, CSVs, relatórios e loops aninhados.

As principais defesas são:

- limitar tamanho de upload;
- limitar linhas;
- limitar colunas;
- limitar células;
- limitar tamanho de texto;
- limitar erros acumulados;
- evitar loops aninhados sem controle;
- evitar chamadas externas dentro de loops;
- validar antes e durante o processamento;
- usar processamento assíncrono para cargas maiores;
- aplicar quotas e rate limiting;
- criar testes de limite;
- documentar falso positivo com evidência.

A principal lição é:

> Toda entrada externa que influencia quantidade de processamento precisa ter limite explícito.

## Perguntas de revisão

### 1. O que é Unchecked Input for Loop Condition?

É uma vulnerabilidade em que uma entrada externa influencia a condição de um loop sem validação de limite, faixa ou custo.

### 2. Por que isso pode causar DoS?

Porque o usuário pode fazer a aplicação repetir operações demais, consumindo CPU, memória, threads, banco ou tempo de processamento.

### 3. Esse problema aparece apenas em loops `for`?

Não. Pode aparecer em `for`, `while`, streams, recursão, retries, parsers e qualquer repetição baseada em entrada externa.

### 4. Por que `getLastRowNum()` pode ser perigoso?

Porque reflete a estrutura da planilha enviada. Se usado sem limite, o arquivo controla a quantidade de linhas processadas.

### 5. Limitar tamanho do arquivo resolve?

Ajuda, mas não resolve sozinho. Também é preciso limitar linhas, colunas, células e tamanho de conteúdo.

### 6. Por que loops aninhados são mais perigosos?

Porque o custo cresce multiplicando dimensões, como linhas x colunas.

### 7. Por que limitar erros acumulados?

Porque uma lista de erros sem limite também pode consumir memória.

### 8. Quando usar processamento assíncrono?

Quando a importação pode ser demorada, pesada, sujeita a retry ou inadequada para uma requisição HTTP síncrona.

### 9. Quando um finding pode ser falso positivo?

Quando o valor do loop é matematicamente limitado, o caminho é inalcançável ou a entrada não consegue aumentar as iterações de forma relevante.

### 10. O que é hardening defensivo?

É ajustar o código para deixar limites e controles explícitos, mesmo quando o risco já é baixo ou o finding é falso positivo.

### 11. O que testar?

Teste excesso de linhas, colunas, células, texto grande, erros demais e comportamento dentro do limite.

### 12. O que não fazer?

Não crie wrappers artificiais apenas para calar o scanner. A correção deve refletir controle real.

### 13. Como justificar uma triagem?

Explique source, sink, fluxo real, limite aplicado, testes e risco residual.

### 14. Qual é a diferença entre validação funcional e proteção contra DoS?

Validação funcional verifica se o dado faz sentido. Proteção contra DoS verifica se o custo de processar o dado é aceitável.

### 15. Qual é a principal regra?

Entrada externa não pode controlar processamento ilimitado.

## Referências

- [CWE-606 — Unchecked Input for Loop Condition](https://cwe.mitre.org/data/definitions/606.html)
- [OWASP Denial of Service Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Denial_of_Service_Cheat_Sheet.html)
- [OWASP Input Validation Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Input_Validation_Cheat_Sheet.html)
- [OWASP Secure Code Review Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Secure_Code_Review_Cheat_Sheet.html)
- [OWASP REST Security Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/REST_Security_Cheat_Sheet.html)
- [OWASP API Security — Lack of Resources and Rate Limiting](https://api-security.owasp.org/editions/2019/pt-BR/0xa4-lack-of-resources-and-rate-limiting/)
- [Apache POI — HSSF and XSSF Limitations](https://poi.apache.org/components/spreadsheet/limitations.html)
- [Apache POI — HSSF/XSSF/SXSSF Spreadsheet APIs](https://poi.apache.org/components/spreadsheet/)
- [Spring Boot — MultipartProperties](https://docs.spring.io/spring-boot/4.0/api/java/org/springframework/boot/servlet/autoconfigure/MultipartProperties.html)
- [Spring Boot — System Requirements](https://docs.spring.io/spring-boot/4.0/system-requirements.html)
- [Oracle JDK 25 Documentation](https://docs.oracle.com/en/java/javase/25/)
- [Oracle Java Downloads — JDK 25 LTS](https://www.oracle.com/java/technologies/downloads/)