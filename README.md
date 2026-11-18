# API de Gerenciamento de Arquivos

## Sobre o projeto

Este projeto consiste em uma API REST criada utilizando Python e Flask, com o objetivo de permitir o envio e o download de arquivos através de requisições HTTP.

Os arquivos recebidos passam por algumas verificações antes de serem armazenados no computador, garantindo que apenas formatos permitidos sejam aceitos. O armazenamento é feito localmente, dentro da pasta `uploads/`.

## Principais recursos

A API oferece os seguintes recursos:

- Upload de arquivos por meio de requisições HTTP POST.
- Download dos arquivos que já foram enviados.
- Verificação dos formatos de arquivos aceitos.
- Controle do tamanho das requisições recebidas.
- Proteção dos nomes dos arquivos com `secure_filename`.
- Registro das operações realizadas em um arquivo de log.
- Criação de um banco de dados SQLite para armazenar informações dos arquivos.

## Ferramentas e tecnologias

As tecnologias utilizadas durante o desenvolvimento foram:

- **Python:** linguagem utilizada para desenvolver a aplicação.
- **Flask:** framework responsável pelo funcionamento da API.
- **Werkzeug:** biblioteca utilizada em recursos de manipulação e segurança dos arquivos.
- **SQLite:** banco de dados utilizado para estruturar os metadados dos uploads.

## Organização dos arquivos

O projeto está dividido da seguinte maneira:

```text
Rafael-AD/
│
├── app.py
│
├── controllers/
│   ├── download_controller.py
│   └── upload_controller.py
│
├── database/
│   └── db.py
│
├── models/
│   └── download_model.py
│
├── routes/
│   ├── download_route.py
│   └── upload_route.py
│
├── uploads/
├── database.db
└── uploads.log
```

### O que cada parte faz?

- **app.py:** arquivo principal, responsável por iniciar o servidor Flask, configurar os diretórios, preparar o banco de dados e registrar as rotas.
- **controllers:** reúne a lógica responsável pelas operações de envio e recebimento dos arquivos.
- **routes:** contém os caminhos utilizados para acessar os recursos da API.
- **models:** possui a função de pesquisa de arquivos cadastrados no banco de dados.
- **database:** realiza a conexão com o SQLite e configura a tabela de uploads.
- **uploads:** diretório onde ficam armazenados os arquivos recebidos.
- **database.db:** arquivo que contém a estrutura do banco de dados.
- **uploads.log:** armazena informações sobre as tentativas e os resultados dos uploads.

## Como instalar e executar

Para utilizar a API, é necessário ter o Python 3 instalado na máquina.

**1.** Abra o terminal dentro da pasta principal do projeto.

**2.** Crie um ambiente virtual para instalar as dependências separadamente:

```bash
python -m venv .venv
```

**3.** Ative o ambiente virtual.

No Windows:

```bash
.venv\Scripts\activate
```

No Linux ou macOS:

```bash
source .venv/bin/activate
```

**4.** Instale o Flask:

```bash
pip install Flask
```

**5.** Inicie o servidor:

```bash
python app.py
```

Depois de iniciar, a API poderá ser acessada pelo endereço:

`http://127.0.0.1:5000`

Na primeira execução, a aplicação também prepara a pasta de armazenamento e cria a tabela necessária no banco SQLite, caso ainda não existam.

## Utilizando a API

Atualmente, a aplicação possui duas operações principais: enviar arquivos e realizar downloads.

### 1. Upload de arquivos

Essa funcionalidade permite enviar um arquivo para o servidor. Após passar pelas validações, ele é armazenado na pasta `uploads/`.

| Propriedade | Descrição |
|---|---|
| Método HTTP | POST |
| Endpoint | `/uploads/upload` |
| Formato | multipart/form-data |
| Nome do campo | `file` |
| Resposta de sucesso | 200 OK |
| Resposta de erro | 400 Bad Request |

**Exemplo de requisição:**

```bash
curl -X POST \
  -F "file=@./documento.pdf" \
  http://127.0.0.1:5000/uploads/upload
```

O arquivo precisa ser enviado utilizando o campo `file`.

**Retorno quando o envio é realizado:**

```json
{
  "mensagem": "Arquivo salvo com sucesso!",
  "arquivo": "documento.pdf"
}
```

**Exemplos de mensagens de erro:**

Caso nenhum arquivo seja enviado:

```json
{
  "erro": "Nenhum arquivo enviado"
}
```

Caso o nome esteja vazio:

```json
{
  "erro": "Nome do arquivo vazio"
}
```

Quando o tamanho ultrapassa o limite verificado:

```json
{
  "erro": "Arquivo excede o tamanho máximo permitido (15 KB)"
}
```

Quando o formato não é aceito:

```json
{
  "erro": "Tipo de arquivo não permitido!",
  "tipos_permitidos": ["png", "jpg", "jpeg", "gif", "pdf", "txt"]
}
```

**Formatos disponíveis para upload:**

- PNG
- JPG
- JPEG
- GIF
- PDF
- TXT

A validação das extensões não diferencia letras maiúsculas de minúsculas.

### 2. Download de arquivos

A segunda funcionalidade permite recuperar arquivos que já foram armazenados no servidor.

| Propriedade | Descrição |
|---|---|
| Método HTTP | GET |
| Endpoint | `/downloads/download/<filename>` |
| Sucesso | 200 OK |
| Arquivo não localizado | 404 Not Found |

Para realizar o download, basta informar o nome do arquivo no endereço da requisição.

**Exemplo:**

```bash
curl -X GET \
  http://127.0.0.1:5000/downloads/download/documento.pdf \
  --output documento.pdf
```

Se o arquivo estiver disponível, ele será enviado como anexo para download, utilizando o recurso `send_from_directory` do Werkzeug.

Caso o arquivo não exista, o Flask normalmente retorna uma resposta HTTP 404. Embora exista uma tentativa de tratamento de `FileNotFoundError` no controller, o Werkzeug utiliza uma exceção HTTP específica (`NotFound`) para esse caso. Por isso, a resposta pode não apresentar o JSON personalizado esperado.

## Critérios de validação

Antes de salvar um arquivo, a API realiza algumas verificações:

1. Confere se existe um arquivo no campo `file`.
2. Verifica se o nome informado não está vazio.
3. Analisa se a extensão pertence aos formatos aceitos.
4. Utiliza `secure_filename` para tratar o nome do arquivo.
5. Compara o tamanho informado na requisição com o limite configurado.

**Atenção ao tamanho permitido:**

No código atual, a constante `TAMANHO_MAXIMO_ARQUIVO` está definida como `30 * 1024` bytes (30 KiB). Entretanto, a mensagem de erro informa um limite de 15 KB.

Outro detalhe é que a verificação utiliza `request.content_length`, que representa o tamanho total da requisição HTTP, incluindo os dados do formulário, e não somente o conteúdo do arquivo.

## Integração com o banco de dados

A aplicação utiliza SQLite para preparar uma tabela chamada `uploads`, responsável por guardar informações relacionadas aos arquivos.

A estrutura criada é:

```sql
CREATE TABLE IF NOT EXISTS uploads (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    filename TEXT UNIQUE NOT NULL,
    filepath TEXT NOT NULL
);
```

O arquivo `database/db.py` é responsável pela criação dessa estrutura.

Já o arquivo `models/download_model.py` possui o método `find_by_filename(filename)`, utilizado para consultar um arquivo pelo nome.

**Importante:** apesar de o banco de dados e o método de consulta estarem implementados, o processo atual de upload não adiciona registros à tabela. Além disso, o download busca os arquivos diretamente na pasta `uploads/`, sem utilizar o model.

## Registro de atividades

A API possui um sistema de logs que registra as operações realizadas durante os uploads.

As informações ficam armazenadas no arquivo `uploads.log`, incluindo:

- Tentativas de envio que apresentaram erros.
- Arquivos enviados com sucesso.
- Nome do arquivo recebido.
- Endereço IP de origem da requisição nos envios concluídos.

Esse registro permite acompanhar as atividades realizadas na aplicação.

## Pontos de atenção

Apesar de apresentar as funcionalidades principais de upload e download, existem alguns aspectos que podem ser melhorados futuramente:

- **Autenticação:** atualmente, qualquer pessoa que consiga acessar a API pode utilizar seus endpoints.
- **Segurança:** a aplicação está configurada com `debug=True`, opção adequada apenas para desenvolvimento.
- **Nomes repetidos:** arquivos enviados com nomes iguais podem substituir os que já existem.
- **Verificação de formato:** a API analisa a extensão, mas não confirma o conteúdo real do arquivo.
- **Tamanho dos arquivos:** existe uma diferença entre o limite configurado e a mensagem apresentada ao usuário.
- **Banco de dados:** a tabela está criada, mas ainda não participa efetivamente do processo de upload e download.
- **Diretórios:** os caminhos utilizados dependem da pasta em que a aplicação é executada.

## Considerações finais

A API foi desenvolvida para demonstrar o funcionamento de operações de envio e download de arquivos utilizando Python e Flask. Sua estrutura separa as responsabilidades entre rotas, controllers, models e banco de dados, facilitando a organização e futuras alterações.

Embora já apresente os recursos básicos necessários, algumas melhorias relacionadas à segurança, ao armazenamento e à integração com o banco de dados podem ser implementadas para tornar o projeto mais completo.