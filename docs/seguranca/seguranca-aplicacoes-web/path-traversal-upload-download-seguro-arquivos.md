# Path Traversal e upload/download seguro de arquivos

Path Traversal é uma vulnerabilidade que acontece quando uma aplicação usa entrada externa para montar caminhos de arquivo sem limitar corretamente onde esse arquivo pode ser lido, escrito, movido ou baixado.

Em termos simples, o problema aparece quando o usuário consegue influenciar o caminho final usado pela aplicação no filesystem.

Em aplicações Java/Spring, esse risco costuma aparecer em funcionalidades como:

- upload de anexos;
- download de arquivos;
- renomeação de arquivos;
- geração de relatórios;
- importação de planilhas;
- leitura de templates;
- escrita temporária em disco;
- movimentação de arquivos;
- uso de `MultipartFile`;
- uso de `transferTo`;
- uso de `Files.write`;
- uso de `Path.resolve`;
- uso de `File`;
- armazenamento de imagens;
- download por nome de arquivo;
- anexos associados a propostas, usuários ou documentos.

O problema real não é a aplicação manipular arquivos. Isso é normal.

O problema é permitir que dados vindos de fora decidam diretamente o caminho físico do arquivo.

Na aplicação fictícia `meucatalogomusical.com`, imagine uma funcionalidade onde administradores podem enviar capas de álbuns de bandas como **Tesla**, **Accept** e **Yngwie Malmsteen**. Se o nome do arquivo enviado pelo usuário for usado diretamente para salvar no disco, a aplicação pode acabar escrevendo fora do diretório esperado.

A regra principal é:

> O usuário pode enviar um arquivo. Ele não deve escolher livremente onde esse arquivo será salvo no servidor.

## O problema real que Path Traversal resolve no estudo de segurança

Funcionalidades de arquivo parecem simples:

```text
Receber arquivo -> salvar arquivo -> permitir download depois
```

Mas, por baixo disso, existem várias decisões de segurança:

- onde o arquivo será salvo;
- qual nome será usado;
- quem pode baixar;
- como o arquivo será identificado;
- se o arquivo fica dentro ou fora do webroot;
- se o arquivo pode ser executado;
- se o nome original será preservado;
- se o tipo será validado;
- se o tamanho será limitado;
- se o download usa path físico ou identificador lógico;
- se logs expõem caminhos internos;
- se erros revelam estrutura do servidor.

Path Traversal aparece quando uma dessas decisões deixa o usuário controlar caminho demais.

Exemplo perigoso:

```text
POST /api/albuns/10/capa?nomeArquivo=cover.jpg
```

Se `nomeArquivo` for usado diretamente em:

```text
/var/app/uploads/{nomeArquivo}
```

a aplicação parece segura para o caso comum.

Mas o caso comum não é suficiente. A aplicação precisa resistir a entradas inesperadas.

O caminho físico deve ser uma decisão da aplicação, não do cliente.

## Antes de entrar nos detalhes: uma ideia simples

Pense em um hotel.

O hóspede pode pedir para guardar uma mala.

Mas ele não pode dizer:

```text
Guarde minha mala dentro do cofre da gerência.
```

ou:

```text
Coloque minha mala no quarto de outro hóspede.
```

O hotel recebe o item, identifica o hóspede, gera um comprovante e guarda a mala em um local controlado.

Com arquivos é parecido.

O usuário pode enviar:

```text
capa-tesla.jpg
```

Mas a aplicação deveria salvar internamente como algo controlado:

```text
/var/app/uploads/9f6c0d77-76fa-4d9d-b58c-90c6791d82e0.jpg
```

E guardar no banco:

```text
id do arquivo -> nome interno -> nome original -> dono -> tipo -> tamanho
```

Depois, para download, o usuário pede pelo identificador lógico:

```text
GET /api/arquivos/123/download
```

e não por um caminho físico como:

```text
GET /download?path=/var/app/uploads/arquivo.jpg
```

Essa diferença é fundamental.

## Definição técnica

Path Traversal é uma vulnerabilidade em que uma aplicação permite que entrada externa controle ou influencie um caminho de arquivo, possibilitando acesso a arquivos ou diretórios fora do local pretendido.

Em SAST, o padrão costuma aparecer como:

```text
source: entrada externa
    -> controller
    -> service
    -> utilitário de arquivo
sink: operação de arquivo
```

Exemplos de source:

- `@RequestParam String nomeArquivo`;
- `@RequestParam String path`;
- `@PathVariable String fileName`;
- `MultipartFile.getOriginalFilename()`;
- valor vindo de header;
- nome de arquivo vindo de planilha;
- chave de storage vinda do request;
- identificador usado para montar path;
- parâmetro usado para download;
- objeto vindo de banco sem validação de ownership.

Exemplos de sink:

- `Files.write`;
- `Files.readAllBytes`;
- `Files.copy`;
- `Files.move`;
- `Files.delete`;
- `MultipartFile.transferTo`;
- `new File(...)`;
- `Path.resolve(...)` seguido de escrita/leitura;
- abertura de `InputStream`;
- criação de `Resource`;
- retorno de arquivo em download.

O risco aumenta quando o source é usado para montar o caminho final sem normalização, validação, geração de nome interno e verificação do diretório base.

## Path Traversal em uploads

Upload é um dos pontos mais comuns.

Exemplo inseguro:

```java
package com.meucatalogomusical.albuns.api;

import org.springframework.web.bind.annotation.*;
import org.springframework.web.multipart.MultipartFile;

@RestController
@RequestMapping("/api/albuns")
public class CapaAlbumController {

    private final CapaAlbumService capaAlbumService;

    public CapaAlbumController(CapaAlbumService capaAlbumService) {
        this.capaAlbumService = capaAlbumService;
    }

    @PostMapping("/{albumId}/capa")
    public void enviarCapa(
            @PathVariable Long albumId,
            @RequestParam String nomeArquivo,
            @RequestParam MultipartFile arquivo
    ) {
        capaAlbumService.salvarCapa(albumId, nomeArquivo, arquivo);
    }
}
```

Service inseguro:

```java
package com.meucatalogomusical.albuns.application;

import org.springframework.stereotype.Service;
import org.springframework.web.multipart.MultipartFile;

@Service
public class CapaAlbumService {

    private final FileStorage fileStorage;

    public CapaAlbumService(FileStorage fileStorage) {
        this.fileStorage = fileStorage;
    }

    public void salvarCapa(Long albumId, String nomeArquivo, MultipartFile arquivo) {
        fileStorage.salvar(nomeArquivo, arquivo);
    }
}
```

Storage inseguro:

```java
package com.meucatalogomusical.albuns.infra;

import org.springframework.stereotype.Component;
import org.springframework.web.multipart.MultipartFile;

import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Path;

@Component
public class FileStorage {

    private final Path diretorioBase = Path.of("/var/app/uploads");

    public void salvar(String nomeArquivo, MultipartFile arquivo) {
        try {
            Path destino = diretorioBase.resolve(nomeArquivo);
            Files.write(destino, arquivo.getBytes());
        } catch (IOException ex) {
            throw new IllegalStateException("Não foi possível salvar o arquivo.", ex);
        }
    }
}
```

O problema está aqui:

```java
Path destino = diretorioBase.resolve(nomeArquivo);
```

O usuário controla `nomeArquivo`.

Se esse valor tiver separadores de diretório, caminho absoluto, `..` ou variações específicas do sistema operacional, a aplicação pode tentar escrever fora do diretório esperado.

## `getOriginalFilename()` também não é confiável

Um erro comum é trocar `@RequestParam String nomeArquivo` por:

```java
String nomeArquivo = arquivo.getOriginalFilename();
```

Isso não resolve o problema.

A documentação do Spring informa que `getOriginalFilename()` retorna o nome original do arquivo no filesystem do cliente; esse nome é fornecido pelo cliente, pode conter path information, pode conter `..` e não deve ser usado diretamente. A recomendação da própria documentação é gerar um nome único e armazenar o nome original apenas como referência quando necessário.

Exemplo perigoso:

```java
String nomeOriginal = arquivo.getOriginalFilename();
Path destino = diretorioBase.resolve(nomeOriginal);
arquivo.transferTo(destino);
```

Mesmo que pareça natural, esse desenho permite que o cliente influencie o nome final salvo no servidor.

O nome original deve ser tratado como metadado, não como caminho final.

## Correção: gerar nome interno no servidor

Uma abordagem segura é gerar o nome interno no servidor.

```java
package com.meucatalogomusical.arquivos.domain;

import java.util.UUID;

public record ArquivoId(UUID value) {

    public ArquivoId {
        if (value == null) {
            throw new IllegalArgumentException("ArquivoId é obrigatório.");
        }
    }

    public static ArquivoId novo() {
        return new ArquivoId(UUID.randomUUID());
    }

    @Override
    public String toString() {
        return value.toString();
    }
}
```

Agora o storage pode usar esse identificador para criar o nome físico.

```java
package com.meucatalogomusical.arquivos.infra;

import com.meucatalogomusical.arquivos.domain.ArquivoId;
import org.springframework.stereotype.Component;
import org.springframework.web.multipart.MultipartFile;

import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Path;
import java.util.Set;

@Component
public class FileStorage {

    private static final Set<String> EXTENSOES_PERMITIDAS = Set.of(
            "jpg", "jpeg", "png", "webp", "pdf"
    );

    private final Path diretorioBase = Path.of("/var/app/uploads")
            .toAbsolutePath()
            .normalize();

    public String salvar(MultipartFile arquivo) {
        String extensao = obterExtensaoPermitida(arquivo);
        String nomeInterno = ArquivoId.novo() + "." + extensao;

        Path destino = resolverDestinoSeguro(nomeInterno);

        try {
            Files.createDirectories(diretorioBase);
            Files.write(destino, arquivo.getBytes());
            return nomeInterno;
        } catch (IOException ex) {
            throw new IllegalStateException("Não foi possível salvar o arquivo.", ex);
        }
    }

    private Path resolverDestinoSeguro(String nomeInterno) {
        Path destino = diretorioBase.resolve(nomeInterno).normalize();

        if (!destino.startsWith(diretorioBase)) {
            throw new IllegalArgumentException("Caminho de arquivo inválido.");
        }

        return destino;
    }

    private String obterExtensaoPermitida(MultipartFile arquivo) {
        String nomeOriginal = arquivo.getOriginalFilename();

        if (nomeOriginal == null || nomeOriginal.isBlank()) {
            throw new IllegalArgumentException("Nome original do arquivo é obrigatório.");
        }

        int ultimoPonto = nomeOriginal.lastIndexOf(".");

        if (ultimoPonto < 0 || ultimoPonto == nomeOriginal.length() - 1) {
            throw new IllegalArgumentException("Extensão do arquivo é obrigatória.");
        }

        String extensao = nomeOriginal.substring(ultimoPonto + 1).toLowerCase();

        if (!EXTENSOES_PERMITIDAS.contains(extensao)) {
            throw new IllegalArgumentException("Extensão de arquivo não permitida.");
        }

        return extensao;
    }
}
```

Aqui o usuário não controla o caminho final.

O nome original é usado apenas para extrair a extensão, e mesmo assim a extensão passa por lista permitida.

O nome físico é gerado pela aplicação.

## O que esse código resolve?

Esse desenho aplica algumas defesas importantes:

- o cliente não escolhe o caminho;
- o cliente não escolhe o nome físico final;
- o diretório base é absoluto;
- o diretório base é normalizado;
- o destino final é normalizado;
- o destino final precisa começar com o diretório base;
- a extensão passa por allowlist;
- o arquivo recebe nome interno aleatório;
- o nome original não é usado diretamente como path.

A documentação do Java `Path` mostra que `normalize()` remove elementos redundantes como `.` e `..`, mas também deixa claro que isso não acessa o filesystem e que links simbólicos podem afetar o caminho real; por isso, `normalize()` é parte da solução, não a solução inteira.

## `normalize()` não basta sozinho

Este é um ponto importante.

Código incompleto:

```java
Path destino = diretorioBase.resolve(nomeArquivo).normalize();
Files.write(destino, arquivo.getBytes());
```

Falta a verificação:

```java
if (!destino.startsWith(diretorioBase)) {
    throw new IllegalArgumentException("Caminho de arquivo inválido.");
}
```

A normalização resolve a forma do path.

A verificação `startsWith` garante que o destino final ainda está dentro do diretório base.

A documentação do Java descreve `Path.startsWith` como teste se um path começa com outro path, comparando elementos do caminho e root component conforme o filesystem.

A sequência defensiva comum é:

```text
base absoluto e normalizado
    -> resolve(nome controlado pela aplicação)
    -> normalize()
    -> startsWith(base)
    -> operação de arquivo
```

## Cuidado com paths absolutos

`Path.resolve` tem um detalhe importante: quando o outro path é absoluto, o comportamento pode substituir a base, dependendo do filesystem. A documentação do Java informa que, se o path a ser resolvido tiver root component, a resolução é dependente da implementação.

Por isso, não use entrada do usuário diretamente em `resolve`.

Exemplo perigoso:

```java
Path destino = diretorioBase.resolve(nomeRecebidoDoUsuario).normalize();
```

Se `nomeRecebidoDoUsuario` puder representar caminho absoluto, você corre risco.

A melhor abordagem é não permitir que o usuário forneça path. Gere o nome interno no servidor.

## Upload seguro com metadados

Normalmente, não basta salvar o arquivo. Você também precisa registrar metadados.

```java
package com.meucatalogomusical.arquivos.domain;

import java.time.OffsetDateTime;

public record ArquivoCatalogo(
        Long id,
        String nomeOriginal,
        String nomeInterno,
        String contentType,
        long tamanhoBytes,
        Long usuarioId,
        OffsetDateTime criadoEm
) {
}
```

A aplicação pode salvar:

- nome original;
- nome interno;
- content type declarado;
- tamanho;
- dono;
- data;
- contexto;
- tipo de arquivo;
- status de validação.

O download depois deve usar o `id`, não o path.

## Exemplo de controller de upload

```java
package com.meucatalogomusical.albuns.api;

import org.springframework.http.HttpStatus;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.multipart.MultipartFile;

@RestController
@RequestMapping("/api/albuns")
public class CapaAlbumController {

    private final CapaAlbumService capaAlbumService;

    public CapaAlbumController(CapaAlbumService capaAlbumService) {
        this.capaAlbumService = capaAlbumService;
    }

    @PostMapping("/{albumId}/capa")
    @ResponseStatus(HttpStatus.CREATED)
    public CapaAlbumResponse enviarCapa(
            @PathVariable Long albumId,
            @RequestParam MultipartFile arquivo
    ) {
        return capaAlbumService.salvarCapa(albumId, arquivo);
    }
}
```

Response:

```java
package com.meucatalogomusical.albuns.api;

public record CapaAlbumResponse(
        Long arquivoId,
        String nomeOriginal,
        long tamanhoBytes
) {
}
```

Service:

```java
package com.meucatalogomusical.albuns.api;

import com.meucatalogomusical.arquivos.application.ArquivoService;
import com.meucatalogomusical.arquivos.domain.ArquivoCatalogo;
import org.springframework.stereotype.Service;
import org.springframework.web.multipart.MultipartFile;

@Service
public class CapaAlbumService {

    private final ArquivoService arquivoService;

    public CapaAlbumService(ArquivoService arquivoService) {
        this.arquivoService = arquivoService;
    }

    public CapaAlbumResponse salvarCapa(Long albumId, MultipartFile arquivo) {
        ArquivoCatalogo arquivoCatalogo = arquivoService.salvarArquivoDoAlbum(albumId, arquivo);

        return new CapaAlbumResponse(
                arquivoCatalogo.id(),
                arquivoCatalogo.nomeOriginal(),
                arquivoCatalogo.tamanhoBytes()
        );
    }
}
```

O controller não recebe `nomeArquivo`.

Isso reduz bastante o risco.

## Download inseguro por path

Agora imagine download.

Exemplo perigoso:

```java
@GetMapping("/download")
public byte[] download(@RequestParam String path) throws IOException {
    return Files.readAllBytes(Path.of(path));
}
```

Esse é o tipo de desenho que deve ser evitado.

O usuário não deve informar:

```text
/var/app/uploads/capa.jpg
```

nem:

```text
../../algum-arquivo
```

nem qualquer path físico.

A API deve receber um identificador lógico.

## Download seguro por id

Exemplo:

```java
package com.meucatalogomusical.arquivos.api;

import org.springframework.core.io.Resource;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/arquivos")
public class ArquivoDownloadController {

    private final ArquivoDownloadService arquivoDownloadService;

    public ArquivoDownloadController(ArquivoDownloadService arquivoDownloadService) {
        this.arquivoDownloadService = arquivoDownloadService;
    }

    @GetMapping("/{arquivoId}/download")
    public ResponseEntity<Resource> download(@PathVariable Long arquivoId) {
        return arquivoDownloadService.download(arquivoId);
    }
}
```

Service:

```java
package com.meucatalogomusical.arquivos.api;

import org.springframework.core.io.Resource;
import org.springframework.http.ResponseEntity;
import org.springframework.stereotype.Service;

@Service
public class ArquivoDownloadService {

    private final ArquivoRepository arquivoRepository;
    private final FileStorage fileStorage;

    public ArquivoDownloadService(
            ArquivoRepository arquivoRepository,
            FileStorage fileStorage
    ) {
        this.arquivoRepository = arquivoRepository;
        this.fileStorage = fileStorage;
    }

    public ResponseEntity<Resource> download(Long arquivoId) {
        ArquivoCatalogo arquivo = arquivoRepository.buscarPorId(arquivoId);

        validarPermissaoDeDownload(arquivo);

        Resource resource = fileStorage.carregarComoResource(arquivo.nomeInterno());

        return ResponseEntity.ok()
                .header("Content-Disposition", "attachment; filename=\"" + arquivo.nomeOriginalSeguro() + "\"")
                .body(resource);
    }

    private void validarPermissaoDeDownload(ArquivoCatalogo arquivo) {
        // Validar autenticação, autorização e ownership conforme a regra do sistema.
    }
}
```

Nesse desenho:

- o cliente informa `arquivoId`;
- o banco resolve o nome interno;
- a aplicação valida permissão;
- o storage resolve o path dentro do diretório base;
- o download usa nome original seguro apenas no header;
- o path físico não é exposto.

## Carregando arquivo como Resource com segurança

```java
package com.meucatalogomusical.arquivos.infra;

import org.springframework.core.io.Resource;
import org.springframework.core.io.UrlResource;
import org.springframework.stereotype.Component;

import java.net.MalformedURLException;
import java.nio.file.Files;
import java.nio.file.Path;

@Component
public class FileStorage {

    private final Path diretorioBase = Path.of("/var/app/uploads")
            .toAbsolutePath()
            .normalize();

    public Resource carregarComoResource(String nomeInterno) {
        Path arquivo = resolverDestinoSeguro(nomeInterno);

        if (!Files.exists(arquivo) || !Files.isRegularFile(arquivo)) {
            throw new IllegalArgumentException("Arquivo não encontrado.");
        }

        try {
            Resource resource = new UrlResource(arquivo.toUri());

            if (!resource.exists() || !resource.isReadable()) {
                throw new IllegalArgumentException("Arquivo não está disponível para leitura.");
            }

            return resource;
        } catch (MalformedURLException ex) {
            throw new IllegalStateException("Não foi possível carregar o arquivo.", ex);
        }
    }

    private Path resolverDestinoSeguro(String nomeInterno) {
        Path destino = diretorioBase.resolve(nomeInterno).normalize();

        if (!destino.startsWith(diretorioBase)) {
            throw new IllegalArgumentException("Caminho de arquivo inválido.");
        }

        return destino;
    }
}
```

Mesmo no download, o path é reconstruído de forma controlada.

O usuário não escolhe o path.

## Cuidado com `Content-Disposition`

O header `Content-Disposition` pode usar o nome original para sugerir o nome de download.

Mas o nome original também vem do usuário.

Portanto, precisa ser tratado.

Exemplo simples:

```java
package com.meucatalogomusical.arquivos.domain;

public record ArquivoCatalogo(
        Long id,
        String nomeOriginal,
        String nomeInterno,
        String contentType,
        long tamanhoBytes
) {

    public String nomeOriginalSeguro() {
        if (nomeOriginal == null || nomeOriginal.isBlank()) {
            return "arquivo";
        }

        return nomeOriginal
                .replace("\\", "_")
                .replace("/", "_")
                .replace("\"", "_")
                .replace("\r", "_")
                .replace("\n", "_");
    }
}
```

Isso evita alguns problemas básicos em header.

Em aplicações reais, você pode usar utilitários próprios do framework ou bibliotecas para construir headers corretamente, especialmente quando houver caracteres internacionais.

## Validação de extensão

Validar extensão não é suficiente, mas ajuda.

```java
private static final Set<String> EXTENSOES_PERMITIDAS = Set.of(
        "jpg", "jpeg", "png", "webp", "pdf"
);
```

A OWASP recomenda validar extensão, tamanho, nome, localização de armazenamento, permissões e outros controles; também recomenda gerar o nome armazenado no servidor para evitar que o atacante controle qual arquivo será atingido.

A extensão deve ser tratada como um sinal, não como prova absoluta de conteúdo.

Um arquivo chamado:

```text
capa.jpg
```

não necessariamente contém uma imagem JPEG válida.

## Validação de content type

O `Content-Type` informado no upload também vem do cliente.

Exemplo:

```java
String contentType = arquivo.getContentType();
```

Isso pode ser útil para rejeições básicas, mas não deve ser sua única defesa.

Melhor combinar:

- extensão permitida;
- content type permitido;
- validação do conteúdo quando necessário;
- limites de tamanho;
- armazenamento seguro;
- ausência de execução;
- autorização no download.

Para imagens, por exemplo, você pode tentar decodificar a imagem com biblioteca confiável antes de aceitar.

Para PDFs, pode haver outras validações e políticas.

## Limite de tamanho

Upload sem limite pode causar problemas de disponibilidade.

Exemplo de configuração:

```yaml
spring:
  servlet:
    multipart:
      max-file-size: 5MB
      max-request-size: 6MB
```

Essa configuração ajuda a evitar arquivos enormes.

Também é comum validar em código:

```java
private static final long TAMANHO_MAXIMO_BYTES = 5 * 1024 * 1024;

private void validarTamanho(MultipartFile arquivo) {
    if (arquivo.isEmpty()) {
        throw new IllegalArgumentException("Arquivo é obrigatório.");
    }

    if (arquivo.getSize() > TAMANHO_MAXIMO_BYTES) {
        throw new IllegalArgumentException("Arquivo excede o tamanho máximo permitido.");
    }
}
```

A OWASP File Upload Cheat Sheet recomenda limites de tamanho para proteger armazenamento e evitar abuso.

## Armazenar fora do webroot

Guardar uploads dentro do webroot pode permitir acesso direto.

Exemplo perigoso:

```text
/var/www/html/uploads/capa.jpg
```

Se o servidor web servir esse diretório diretamente, controles de autorização da aplicação podem ser contornados.

A OWASP recomenda escolher a localização de armazenamento conforme requisitos de segurança; entre as opções, cita armazenar em host diferente, fora do webroot ou, se dentro do webroot, aplicar controles apropriados.

Para muitos sistemas corporativos, uma boa decisão é:

```text
/var/app/uploads
```

fora do diretório público do servidor web.

O download deve passar pela aplicação, que valida autorização.

## Permissões do diretório

O diretório de upload não deve ter permissão de execução.

Boas práticas:

- diretório fora do webroot;
- permissão de escrita apenas para o usuário da aplicação;
- sem execução de scripts;
- separação por ambiente;
- backup controlado;
- monitoramento de espaço;
- política de retenção;
- limpeza de temporários;
- antivírus ou varredura quando aplicável;
- auditoria de acesso em arquivos sensíveis.

Upload seguro não é apenas código. É também filesystem, infraestrutura e operação.

## Upload temporário

Às vezes, a aplicação precisa salvar temporariamente antes de validar.

Nesse caso:

- use diretório temporário controlado;
- não use nome original como nome físico;
- aplique limite de tamanho;
- limpe arquivos temporários;
- não execute conteúdo;
- não sirva temporários publicamente;
- aplique permissões restritas.

Exemplo:

```java
Path temporario = Files.createTempFile("upload-", ".tmp");
```

Mesmo assim, o arquivo temporário precisa ser tratado como dado não confiável.

## Path Traversal e banco de dados

Salvar o `nomeInterno` no banco ajuda, mas também exige cuidado.

Não salve path absoluto se não for necessário.

Evite:

```text
/var/app/uploads/9f6c0d77.jpg
```

Prefira:

```text
9f6c0d77.jpg
```

A aplicação conhece o diretório base.

O banco guarda apenas identificador ou nome interno.

Isso reduz vazamento de estrutura do servidor e facilita migração de storage.

## Path Traversal e storage externo

Se os arquivos forem para S3, MinIO, Google Cloud Storage ou outro object storage, ainda há riscos semelhantes.

A chave do objeto não deve ser controlada livremente pelo usuário.

Exemplo perigoso:

```text
bucket/{nomeArquivoRecebido}
```

Melhor:

```text
bucket/albuns/{albumId}/{uuid}.{extensao}
```

Mesmo em object storage:

- gere chave interna;
- valide extensão;
- limite tamanho;
- controle content type;
- não exponha bucket público sem necessidade;
- use URLs assinadas com expiração quando apropriado;
- valide autorização antes de gerar download;
- não use path vindo do usuário como chave bruta.

Path Traversal é mais associado a filesystem, mas o princípio de controle de namespace vale também para storage de objetos.

## Path Traversal e importação de planilhas

Outro cenário comum é importação.

Por exemplo, `meucatalogomusical.com` permite importar uma planilha com discografia de **King Diamond**.

Riscos:

- planilha enorme;
- muitas linhas;
- muitas colunas;
- arquivo com extensão falsa;
- parser consumindo memória;
- nome de arquivo controlando destino;
- planilha salva em temporário inseguro;
- erros revelando caminho interno;
- arquivo importado ficando acessível sem autorização.

Mesmo que o foco seja parser, o upload ainda precisa ser seguro.

## Tratamento seguro de erros

Não exponha caminho físico em erro.

Exemplo ruim:

```java
throw new IllegalStateException("Erro ao salvar em /var/app/uploads/" + nomeArquivo, ex);
```

Exemplo melhor:

```java
throw new IllegalStateException("Não foi possível salvar o arquivo.", ex);
```

No log interno, evite expor path completo quando não for necessário. Se precisar, use correlation id e contexto suficiente, sem revelar estrutura sensível para o cliente.

## Exemplo de exception handler

```java
package com.meucatalogomusical.shared.api;

import org.springframework.http.HttpStatus;
import org.springframework.web.bind.annotation.*;

@RestControllerAdvice
public class ApiExceptionHandler {

    @ExceptionHandler(IllegalArgumentException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public ErroResponse tratarEntradaInvalida(IllegalArgumentException ex) {
        return new ErroResponse(ex.getMessage());
    }

    @ExceptionHandler(IllegalStateException.class)
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    public ErroResponse tratarErroInterno(IllegalStateException ex) {
        return new ErroResponse("Não foi possível processar o arquivo.");
    }

    public record ErroResponse(String mensagem) {
    }
}
```

Esse exemplo evita devolver stack trace ou path físico ao consumidor.

## Testes de upload seguro

Teste o comportamento do storage.

```java
package com.meucatalogomusical.arquivos.infra;

import org.junit.jupiter.api.Test;
import org.springframework.mock.web.MockMultipartFile;

import static org.junit.jupiter.api.Assertions.*;

class FileStorageTest {

    private final FileStorage fileStorage = new FileStorage();

    @Test
    void deveRejeitarArquivoSemExtensao() {
        MockMultipartFile arquivo = new MockMultipartFile(
                "arquivo",
                "capa",
                "image/jpeg",
                "conteudo".getBytes()
        );

        assertThrows(IllegalArgumentException.class,
                () -> fileStorage.salvar(arquivo));
    }

    @Test
    void deveRejeitarExtensaoNaoPermitida() {
        MockMultipartFile arquivo = new MockMultipartFile(
                "arquivo",
                "script.sh",
                "text/plain",
                "conteudo".getBytes()
        );

        assertThrows(IllegalArgumentException.class,
                () -> fileStorage.salvar(arquivo));
    }

    @Test
    void deveSalvarComNomeInternoGeradoPelaAplicacao() {
        MockMultipartFile arquivo = new MockMultipartFile(
                "arquivo",
                "capa-accept.jpg",
                "image/jpeg",
                "conteudo".getBytes()
        );

        String nomeInterno = fileStorage.salvar(arquivo);

        assertNotNull(nomeInterno);
        assertTrue(nomeInterno.endsWith(".jpg"));
        assertFalse(nomeInterno.contains("capa-accept"));
    }
}
```

Esse teste valida uma regra essencial:

> O nome original não vira nome físico final.

## Teste de path seguro

Se o método `resolverDestinoSeguro` for extraído para uma classe testável, dá para testar tentativa de fuga.

```java
package com.meucatalogomusical.arquivos.infra;

import org.junit.jupiter.api.Test;

import java.nio.file.Path;

import static org.junit.jupiter.api.Assertions.*;

class SafePathResolverTest {

    private final SafePathResolver resolver =
            new SafePathResolver(Path.of("/var/app/uploads"));

    @Test
    void deveResolverNomeInternoDentroDoDiretorioBase() {
        Path path = resolver.resolve("arquivo.jpg");

        assertTrue(path.startsWith(Path.of("/var/app/uploads").toAbsolutePath().normalize()));
    }

    @Test
    void deveRejeitarTentativaDeSairDoDiretorioBase() {
        assertThrows(IllegalArgumentException.class,
                () -> resolver.resolve("../arquivo.jpg"));
    }

    @Test
    void deveRejeitarCaminhoAbsoluto() {
        assertThrows(IllegalArgumentException.class,
                () -> resolver.resolve("/etc/passwd"));
    }
}
```

Classe:

```java
package com.meucatalogomusical.arquivos.infra;

import java.nio.file.Path;

public class SafePathResolver {

    private final Path baseDirectory;

    public SafePathResolver(Path baseDirectory) {
        this.baseDirectory = baseDirectory.toAbsolutePath().normalize();
    }

    public Path resolve(String fileName) {
        Path candidate = baseDirectory.resolve(fileName).normalize();

        if (!candidate.startsWith(baseDirectory)) {
            throw new IllegalArgumentException("Caminho de arquivo inválido.");
        }

        return candidate;
    }
}
```

Observação: em um sistema real, se você permitir apenas nomes internos gerados pela aplicação, esse resolver recebe uma entrada bem mais restrita. Isso é melhor do que aceitar qualquer nome vindo do usuário.

## SAST e Path Traversal

Ferramentas SAST costumam apontar Path Traversal quando veem este fluxo:

```text
source: MultipartFile ou request parameter
    -> controller
    -> service
    -> FileUtil
sink: write, transferTo, read, move, delete
```

Esse finding merece atenção.

Para triagem, pergunte:

- o usuário controla o path?
- o usuário controla o nome físico?
- o nome original é usado diretamente?
- existe diretório base fixo?
- o path é absoluto e normalizado?
- existe `startsWith(baseDirectory)`?
- o arquivo é salvo fora do webroot?
- existe autorização no download?
- o erro expõe caminho interno?
- existem testes?

Se o usuário controla o path e não existe verificação forte, provavelmente é vulnerabilidade real.

## Quando é falso positivo?

Pode ser falso positivo quando:

- o usuário não controla o path final;
- o nome físico é gerado pela aplicação;
- o input é apenas arquivo, não destino;
- há diretório base fixo;
- há normalização e verificação `startsWith`;
- o download usa id lógico;
- há validação de autorização;
- o método apontado pelo scanner recebe nome interno já controlado;
- existem testes comprovando tentativa de fuga rejeitada.

Mesmo assim, vale documentar claramente o fluxo.

## Quando é risco mitigado?

Pode ser risco mitigado quando:

- há algum uso de nome vindo de fora, mas ele é reduzido a nome simples;
- há allowlist de extensão;
- há diretório controlado;
- há normalização;
- há verificação de base;
- há permissão restrita no filesystem;
- há autorização no download;
- há testes.

Mas, sempre que possível, prefira eliminar o controle do usuário sobre o nome físico.

## Boas práticas

### 1. Não use nome do cliente como nome físico

`getOriginalFilename()` é metadado, não destino.

### 2. Gere nome interno

Use UUID, ULID ou outro identificador gerado pela aplicação.

### 3. Salve fora do webroot

Evite acesso direto sem autorização.

### 4. Use diretório base fixo

O diretório de armazenamento deve ser definido pela aplicação.

### 5. Normalize e verifique

Use `toAbsolutePath().normalize()` no base directory e `resolve(...).normalize()` no destino, com `startsWith(base)`.

### 6. Valide extensão por allowlist

Não aceite extensões arbitrárias.

### 7. Limite tamanho

Configure limites no Spring e valide quando necessário.

### 8. Não exponha path físico

APIs devem usar ids lógicos.

### 9. Valide autorização no download

Ter o id do arquivo não significa ter permissão para baixá-lo.

### 10. Não execute uploads

Diretório de upload não deve permitir execução de scripts.

## Erros comuns

### 1. Usar `getOriginalFilename()` diretamente

O nome vem do cliente e pode ser malicioso.

### 2. Fazer apenas `replace("../", "")`

Denylist é frágil. Existem variações de encoding, separadores e sistemas operacionais.

### 3. Confiar só na extensão

Extensão não prova conteúdo.

### 4. Salvar dentro do webroot

Pode permitir acesso direto sem autorização.

### 5. Fazer download por path

Use id lógico, não caminho físico.

### 6. Expor path em erro

Mensagens de erro não devem revelar estrutura interna.

### 7. Não limitar tamanho

Uploads grandes podem causar DoS por armazenamento.

### 8. Não limpar temporários

Arquivos temporários acumulados viram risco operacional.

### 9. Não validar autorização

Download precisa validar ownership ou permissão.

### 10. Criar wrapper artificial para SAST

A correção precisa reduzir risco real, não apenas esconder o fluxo.

## Decisões de engenharia

## Quando armazenar em filesystem?

Armazenar em filesystem pode fazer sentido quando:

- volume é moderado;
- infraestrutura é simples;
- há backup;
- há controle de permissão;
- há diretório fora do webroot;
- há limpeza e monitoramento;
- a aplicação controla download.

## Quando usar object storage?

Object storage pode ser melhor quando:

- há muitos arquivos;
- há necessidade de escala;
- há integração cloud;
- há lifecycle policy;
- há URL assinada;
- há separação entre app e storage;
- há versionamento;
- há auditoria.

Mesmo assim, use chave interna controlada.

## Quando armazenar no banco?

Armazenar arquivos no banco pode fazer sentido para arquivos pequenos, consistência transacional ou requisitos específicos.

Mas pode gerar:

- crescimento do banco;
- impacto em backup;
- impacto em performance;
- aumento de custo;
- complexidade operacional.

A decisão deve considerar DBA, operação e requisitos de retenção.

## Quando preservar nome original?

Preserve nome original como metadado quando for útil para o usuário.

Mas não use como nome físico.

Exemplo:

```text
nomeOriginal = capa-tesla.jpg
nomeInterno = 9f6c0d77-76fa-4d9d-b58c-90c6791d82e0.jpg
```

## Quando permitir download inline?

`inline` pode fazer sentido para imagens ou PDFs visualizados no navegador.

Mas `attachment` é mais seguro por padrão quando você não quer que o navegador interprete o conteúdo.

Avalie:

- tipo do arquivo;
- origem do conteúdo;
- risco de execução;
- Content-Type;
- `X-Content-Type-Options: nosniff`;
- necessidade de visualização;
- política de segurança.

## Onde isso aparece em sistemas reais

Path Traversal e upload seguro aparecem em:

- anexos de propostas;
- upload de documentos;
- importação de planilhas;
- fotos de perfil;
- capas de álbum;
- comprovantes;
- relatórios gerados;
- arquivos temporários;
- download de contratos;
- exportação CSV;
- storage local;
- integração com S3;
- gateways de arquivo;
- sistemas legados;
- painéis administrativos.

Em aplicações Java/Spring, os pontos mais sensíveis são:

- controllers multipart;
- services de anexo;
- classes `FileUtil`;
- `MultipartFile.transferTo`;
- `Files.write`;
- `Files.readAllBytes`;
- `Path.resolve`;
- downloads por query param;
- nomes de arquivo vindos do request;
- importadores de XLSX/CSV;
- diretórios temporários.

## Checklist prático

### Upload

- O usuário controla o nome físico?
- `getOriginalFilename()` é usado diretamente?
- O nome interno é gerado pela aplicação?
- Existe allowlist de extensão?
- Existe limite de tamanho?
- O content type é validado?
- O arquivo fica fora do webroot?
- O diretório não permite execução?
- Erros não expõem path?

### Path

- O diretório base é fixo?
- O diretório base é absoluto?
- O diretório base é normalizado?
- O destino é normalizado?
- Há verificação `startsWith(base)`?
- Caminhos absolutos vindos do usuário são rejeitados?
- `..` é neutralizado pela combinação de normalize e verificação de base?
- Links simbólicos foram considerados quando aplicável?

### Download

- Download usa id lógico?
- Há validação de autorização?
- O path físico não aparece na URL?
- O path físico não aparece na resposta?
- `Content-Disposition` é seguro?
- O arquivo existe e é regular?
- O usuário tem permissão para baixar?
- O content type é controlado?

### Operação

- Diretório tem permissão mínima?
- Uploads são monitorados?
- Há limpeza de temporários?
- Há backup adequado?
- Há política de retenção?
- Há limite de armazenamento?
- Há alerta de disco?
- Há varredura de malware quando necessário?

### SAST

- Source e sink foram entendidos?
- O scanner aponta `write`, `read`, `transferTo` ou `move`?
- O path final é controlado?
- Há testes de tentativa de fuga?
- O novo scan foi executado?
- A justificativa documenta o fluxo real?

## Resumo final

Path Traversal acontece quando entrada externa consegue influenciar caminhos de arquivo de forma insegura.

Em aplicações Java/Spring, isso aparece principalmente em upload, download, anexos, importação de arquivos e classes utilitárias de filesystem.

As principais defesas são:

- não usar nome enviado pelo cliente como caminho físico;
- gerar nome interno no servidor;
- armazenar fora do webroot;
- usar diretório base fixo;
- normalizar caminhos;
- verificar que o destino permanece dentro da base;
- limitar extensão e tamanho;
- validar autorização no download;
- não expor path físico;
- tratar erros sem revelar estrutura interna;
- criar testes de tentativa de fuga.

A lição principal é:

> O usuário pode enviar e baixar arquivos conforme a regra de negócio, mas nunca deve controlar diretamente o caminho físico usado pela aplicação.

## Perguntas de revisão

### 1. O que é Path Traversal?

É uma vulnerabilidade em que entrada externa influencia um caminho de arquivo e permite acessar, escrever ou manipular arquivos fora do diretório esperado.

### 2. Por que `getOriginalFilename()` é perigoso?

Porque o nome original vem do cliente, pode conter path information, caracteres maliciosos e não deve ser usado diretamente como nome de destino.

### 3. O que é nome interno?

É o nome físico gerado pela aplicação para armazenar o arquivo, geralmente usando UUID ou outro identificador seguro.

### 4. O nome original deve ser descartado?

Não necessariamente. Ele pode ser salvo como metadado, mas não deve ser usado como caminho físico.

### 5. `normalize()` resolve tudo?

Não. Ele remove elementos redundantes, mas precisa ser combinado com verificação de diretório base e cuidado com links simbólicos.

### 6. Para que serve `startsWith(baseDirectory)`?

Para verificar se o path final continua dentro do diretório permitido.

### 7. Por que salvar fora do webroot?

Para evitar que arquivos sejam acessados diretamente sem passar por autorização da aplicação.

### 8. Download deve receber path?

Não. O ideal é receber um identificador lógico e resolver o arquivo internamente.

### 9. Validar extensão é suficiente?

Não. Extensão ajuda, mas deve ser combinada com tamanho, content type, validação de conteúdo, storage seguro e autorização.

### 10. Por que limitar tamanho de upload?

Para evitar abuso de armazenamento, consumo excessivo de memória e indisponibilidade.

### 11. O que é path físico?

É o caminho real ou interno usado pelo servidor, como `/var/app/uploads/arquivo.jpg`.

### 12. O path físico deve aparecer para o usuário?

Não. A API deve expor identificadores lógicos.

### 13. Quando um finding Path Traversal pode ser falso positivo?

Quando o usuário não controla o path final, o nome interno é gerado pela aplicação, há diretório base fixo, normalização, verificação de base e testes.

### 14. Qual é o risco de usar `FileUtil` genérico?

Classes utilitárias genéricas podem concentrar sinks perigosos, como `write`, `read` e `transferTo`. Elas precisam de contratos claros e validações fortes.

### 15. Qual é a principal regra?

Entrada externa não deve controlar caminho físico de arquivo.

## Referências

- [OWASP Path Traversal](https://community.owasp.org/attacks/Path_Traversal)
- [OWASP File Upload Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/File_Upload_Cheat_Sheet.html)
- [OWASP Input Validation Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Input_Validation_Cheat_Sheet.html)
- [OWASP Unrestricted File Upload](https://community.owasp.org/vulnerabilities/Unrestricted_File_Upload)
- [CWE-22 — Improper Limitation of a Pathname to a Restricted Directory](https://cwe.mitre.org/data/definitions/22.html)
- [Spring Framework — MultipartFile API](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/multipart/MultipartFile.html)
- [Java SE 25 — Path API](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/nio/file/Path.html)
- [Oracle JDK 25 Documentation](https://docs.oracle.com/en/java/javase/25/)
- [Oracle Java Downloads — JDK 25 LTS](https://www.oracle.com/java/technologies/downloads/)