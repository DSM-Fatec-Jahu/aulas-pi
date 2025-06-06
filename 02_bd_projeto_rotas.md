# Aula 2: Conexão com MySQL, Projeto com Composer, PDO e Bootstrap 5.3

## 📋 Sumário da Aula

- [Introdução](#introdução)
- [1. Conectando ao MySQL](#1-conectando-ao-mysql-com-ip-e-porta)
- [2. Criação do Banco de Dados e Tabela](#2-criação-do-banco-e-da-tabela-usuarios)
- [3. Operações CRUD (SQL)](#3-operações-crud-sql-com-comentários)
- [4. Criando Projeto com Composer](#4-criando-o-projeto-com-composer)
- [5. Configurando Autoload PSR-4](#5-configurando-o-autoload-psr-4)
- [6. Estrutura de Pastas](#6-estrutura-de-pastas-e-componentes)
- [7. Template Bootstrap](#7-template-base-com-bootstrap)
- [8. Roteador Simples](#8-roteador-simples)
- [9. Conexão PDO](#9-conexão-pdo-com-ip-e-porta)
- [10. Próximos Passos](#10-próximos-passos)

## Introdução

Olá! Bem-vindo à segunda aula do nosso curso de PHP com MySQL. Nesta aula, vamos entender como construir um sistema web completo, desde a conexão com o banco de dados até a criação de uma estrutura organizada para o nosso projeto.

Não se preocupe se você não tem experiência prévia - vamos explicar cada passo detalhadamente. Ao final desta aula, você terá criado a base para um sistema web completo com PHP e MySQL.

## 1. Conectando ao MySQL com IP e Porta

Antes de começarmos a programar, precisamos nos conectar ao banco de dados MySQL através do terminal. Isso nos permite criar e gerenciar nosso banco de dados diretamente.

```bash
mysql -h 127.0.0.1 -P 3306 -u root -p
```

**Explicação detalhada:**
- `mysql`: Este é o comando que inicia o cliente do MySQL no terminal
- `-h 127.0.0.1`: Define o endereço IP do servidor MySQL. Neste caso, estamos usando o localhost (seu próprio computador)
- `-P 3306`: Define a porta onde o MySQL está rodando. A 3306 é a porta padrão do MySQL
- `-u root`: Define o usuário para conectar ao MySQL. "root" é o usuário administrador padrão
- `-p`: Indica que o MySQL deve solicitar a senha do usuário

> **Dica para iniciantes:** Se você estiver usando XAMPP, WAMP ou similar, o MySQL já está instalado e você só precisa iniciar o serviço antes de executar este comando.

## 2. Criação do Banco e da Tabela usuarios

Após conectar ao MySQL, vamos criar nosso banco de dados e a primeira tabela que irá armazenar as informações dos usuários do sistema.

### Criar banco e selecionar:

```sql
CREATE DATABASE sistema_projeto;

USE sistema_projeto;
```

**Explicação:**
- `CREATE DATABASE sistema_projeto;`: Cria um novo banco de dados chamado "sistema_projeto"
- `USE sistema_projeto;`: Seleciona o banco de dados recém-criado para uso

### Criar a tabela com todos os campos:

```sql
CREATE TABLE usuarios (
    id_usuario BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY, -- identificador único
    nome VARCHAR(255) NOT NULL, -- nome completo do usuário
    cpf VARCHAR(14), -- CPF no formato 000.000.000-00
    data_nascimento DATE, -- data no formato yyyy-mm-dd
    celular VARCHAR(20), -- celular com DDD
    rua VARCHAR(255), -- nome da rua
    numero VARCHAR(10), -- número da residência
    complemento VARCHAR(50), -- complemento (ex: apto)
    bairro VARCHAR(255), -- bairro
    cidade VARCHAR(255), -- cidade
    cep VARCHAR(10), -- CEP
    estado CHAR(2), -- estado (ex: SP, RJ)
    email VARCHAR(255) NOT NULL, -- e-mail válido
    tipo ENUM('Administrador', 'Funcionário', 'Cliente') NOT NULL, -- tipo de usuário
    senha VARCHAR(255) NOT NULL, -- senha criptografada
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP, -- data de criação
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP, -- data de alteração
    deleted_at TIMESTAMP NULL DEFAULT NULL -- marcação de exclusão lógica
);
```

**Explicação detalhada da estrutura:**

- `id_usuario`: Identificador único de cada usuário com as seguintes características:
  - `BIGINT`: Número inteiro grande (para suportar muitos registros)
  - `UNSIGNED`: Apenas números positivos
  - `AUTO_INCREMENT`: Incrementa automaticamente a cada novo registro
  - `PRIMARY KEY`: Define como chave primária (identificador único da tabela)

- `nome VARCHAR(255) NOT NULL`: 
  - Campo para armazenar o nome completo
  - `VARCHAR(255)`: Texto de até 255 caracteres
  - `NOT NULL`: Campo obrigatório (não pode ficar vazio)

- `cpf VARCHAR(14)`: 
  - Campo para armazenar o CPF incluindo pontos e traço (ex: 123.456.789-00)
  - Usamos VARCHAR em vez de número porque o CPF tem formatação

- `data_nascimento DATE`: 
  - Campo para armazenar a data de nascimento no formato YYYY-MM-DD

- Campos de endereço:
  - `rua`, `numero`, `complemento`, `bairro`, `cidade`, `cep`, `estado`
  - Permitem armazenar o endereço completo do usuário

- `email VARCHAR(255) NOT NULL`:
  - Campo para armazenar o e-mail
  - `NOT NULL`: É obrigatório fornecer um e-mail

- `tipo ENUM('Administrador', 'Funcionário', 'Cliente') NOT NULL`:
  - Define o tipo/nível de acesso do usuário
  - `ENUM`: Permite apenas os valores especificados entre parênteses
  - `NOT NULL`: Obrigatório definir um tipo

- `senha VARCHAR(255) NOT NULL`:
  - Armazena a senha criptografada do usuário
  - NUNCA armazenamos senhas em texto puro!
  - 255 caracteres para acomodar senhas com hash (criptografadas)

- Campos de controle de datas:
  - `created_at`: Data/hora de criação do registro
  - `updated_at`: Data/hora da última atualização
  - `deleted_at`: Campo para implementar exclusão lógica (null = ativo)

> **Conceito importante:** A tabela já inclui o conceito de **exclusão lógica** através do campo `deleted_at`. Quando este campo está NULL, significa que o usuário está ativo. Quando contém uma data/hora, significa que o usuário foi "excluído" (mas os dados ainda estão no banco).

## 3. Operações CRUD (SQL) com Comentários

CRUD é um acrônimo para as quatro operações básicas de persistência de dados:
- **C**reate (Criar) - Inserir dados
- **R**ead (Ler) - Consultar dados
- **U**pdate (Atualizar) - Modificar dados
- **D**elete (Excluir) - Remover dados

Vamos ver como realizar cada uma dessas operações com SQL.

### ✅ INSERT COMPLETO (Create)

```sql
-- Inserindo um novo usuário com todos os campos preenchidos
INSERT INTO usuarios (
    nome, cpf, data_nascimento, celular, rua, numero, complemento,
    bairro, cidade, cep, estado, email, tipo, senha
)
VALUES (
    'João da Silva',
    '123.456.789-00',
    '1990-05-15',
    '(11)99999-1234',
    'Rua das Flores',
    '123',
    'Apto 4B',
    'Centro',
    'São Paulo',
    '01000-000',
    'SP',
    'joao.silva@email.com',
    'Cliente',
    'senha_criptografada_com_hash'
);
```

**Explicação:**
- O comando `INSERT INTO` adiciona um novo registro na tabela
- Primeiro listamos os nomes das colunas que queremos preencher
- Em seguida, após `VALUES`, fornecemos os valores para cada coluna na mesma ordem
- As strings devem estar entre aspas simples
- Perceba que não inserimos valores para os campos `id_usuario`, `created_at`, `updated_at` e `deleted_at` porque eles são preenchidos automaticamente

### 🔄 UPDATE com 2 campos (Update)

```sql
-- Atualizando nome e celular
UPDATE usuarios
SET nome = 'João da Silva Atualizado',
    celular = '(11)98888-0000'
WHERE id_usuario = 1 AND deleted_at IS NULL;
```

**Explicação:**
- O comando `UPDATE` modifica registros existentes
- `SET` define quais campos serão alterados e seus novos valores
- `WHERE` é MUITO IMPORTANTE: especifica quais registros serão atualizados
- `AND deleted_at IS NULL` garante que apenas usuários ativos sejam modificados
- Se não incluíssemos a cláusula WHERE, TODOS os registros seriam atualizados!

### 🔍 READ (consulta)

```sql
-- Selecionar todos os usuários não deletados
SELECT * FROM usuarios WHERE deleted_at IS NULL;

-- Selecionar um usuário específico
SELECT * FROM usuarios WHERE id_usuario = 1 AND deleted_at IS NULL;
```

**Explicação:**
- `SELECT * FROM usuarios`: Seleciona todos os campos (*) da tabela usuarios
- `WHERE deleted_at IS NULL`: Filtra apenas usuários ativos (não deletados)
- No segundo exemplo, adicionamos `id_usuario = 1` para buscar um usuário específico
- Sempre incluímos a condição `deleted_at IS NULL` para ignorar registros excluídos logicamente

### ❌ DELETE LÓGICO

```sql
-- Marca o registro como deletado, mas mantém no banco
UPDATE usuarios
SET deleted_at = NOW()
WHERE id_usuario = 1;
```

**Explicação:**
- Usamos um `UPDATE` para implementar a exclusão lógica
- Definimos `deleted_at` com a data/hora atual (usando `NOW()`)
- Isso marca o registro como excluído sem removê-lo fisicamente do banco
- Todas as consultas devem incluir `WHERE deleted_at IS NULL` para ignorar esses registros

### ⚠️ DELETE FÍSICO

```sql
-- Remove o registro definitivamente do banco
DELETE FROM usuarios WHERE id_usuario = 1;
```

**Explicação:**
- Este comando remove PERMANENTEMENTE o registro do banco de dados
- Não há como recuperar dados após um DELETE físico (a menos que tenha backup)
- Use com extrema cautela!

### ⚖️ Diferença entre DELETE Lógico e Físico

| **Tipo de DELETE** | **Vantagens** | **Desvantagens** |
|-------------------|---------------|------------------|
| Lógico | • Permite recuperação<br>• Mantém histórico<br>• Mais seguro | • Banco cresce com dados "inativos"<br>• Precisa incluir condição `deleted_at IS NULL` em todas as consultas |
| Físico | • Banco mais limpo<br>• Consultas mais simples<br>• Economiza espaço | • Dados perdidos permanentemente<br>• Não há histórico<br>• Risco de exclusão acidental |

> **Recomendação para iniciantes:** Para sistemas de negócios, quase sempre é melhor usar exclusão lógica. Ela permite recuperar dados excluídos por engano e manter o histórico de informações.

## 4. Criando o Projeto com Composer

Agora que entendemos o banco de dados, vamos criar a estrutura do nosso projeto PHP. O Composer é uma ferramenta essencial para gerenciar dependências e organizar projetos PHP modernos.

### 1. Criando a pasta do projeto:

```bash
cd c:\projetos
mkdir sistema_estoque
cd sistema_estoque
```

**Explicação:**
- `cd c:\projetos`: Navega para a pasta onde criaremos nosso projeto
- `mkdir sistema_estoque`: Cria uma nova pasta chamada "sistema_estoque"
- `cd sistema_estoque`: Entra na pasta do projeto

> **Para usuários macOS/Linux:** Use `cd ~/projetos` ou o caminho da sua preferência

### 2. Iniciando o Composer:

```bash
composer init
```

Este comando inicia um assistente interativo para criar seu arquivo `composer.json`. Veja como preencher:

| **Pergunta** | **Exemplo** | **Explicação** |
|--------------|-------------|----------------|
| Package name | ronan/sistema-estoque | Formato: `vendor/projeto` (geralmente seu nome/nickname + nome do projeto) |
| Description | Sistema de estoque com rotas seguras | Breve descrição do que seu projeto faz |
| Author | Ronan Zenatti <ronan@exemplo.com> | Seu nome e e-mail |
| Minimum Stability | stable | Use apenas versões estáveis das dependências |
| License | MIT | Licença de software que você deseja usar |
| Definir dependências? | [n] (pressione ENTER) | Adicionaremos dependências mais tarde, se necessário |

> **Dica para iniciantes:** O Composer simplifica muito o desenvolvimento PHP moderno. Ele permite carregar automaticamente suas classes e gerenciar bibliotecas externas.

## 5. Configurando o Autoload PSR-4

PSR-4 é um padrão de carregamento automático de classes em PHP. Configurar o autoload permite que suas classes sejam carregadas automaticamente sem precisar de "includes" ou "requires" manuais.

No seu arquivo `composer.json`, adicione:

```json
"autoload": {
    "psr-4": {
        "App\\": "app/"
    }
}
```

**Explicação detalhada:**
- `"autoload"`: Define configurações de carregamento automático
- `"psr-4"`: Especifica que estamos usando o padrão PSR-4
- `"App\\": "app/"`: Todo namespace começando com `App\` será procurado na pasta `app/`
  - Note que no namespace usamos barra dupla `\\` porque a primeira barra é de escape em JSON

Após adicionar essa configuração, execute:

```bash
composer dump-autoload
```

**O que isso faz:**
- Gera os arquivos necessários para o autoload funcionar
- Quando criarmos classes com o namespace `App\`, o PHP conseguirá encontrá-las automaticamente
- Isso elimina a necessidade de fazer `require` de cada arquivo manualmente

## 6. Estrutura de Pastas e Componentes

Agora vamos organizar nosso projeto em uma estrutura de pastas lógica, seguindo boas práticas de desenvolvimento:

```
sistema_estoque/
├── app/
│   ├── Controllers/   -> Regras e lógicas das rotas
│   ├── Core/          -> Conexão com banco (Database.php)
│   ├── Models/        -> Comunicação com banco (ex: Usuario.php)
│   ├── Views/         -> HTML das páginas
│   │   ├── layouts/   -> Template base com Bootstrap
│   │   ├── home.php
│   │   ├── sobre.php
│   │   ├── login.php
│   │   ├── dashboard.php
│   │   └── usuarios/  -> Páginas de CRUD
├── public/            -> Ponto de entrada (index.php)
├── composer.json      -> Arquivo de configuração
├── vendor/            -> Pasta criada pelo Composer
```

**Explicação da estrutura:**

- **app/**: Contém todo o código da aplicação organizado por responsabilidade
  - **Controllers/**: Classes responsáveis por processar as requisições e conectar Models e Views
  - **Core/**: Componentes essenciais como conexão com banco de dados
  - **Models/**: Classes que manipulam os dados e se comunicam com o banco
  - **Views/**: Arquivos HTML/PHP para exibição ao usuário

- **public/**: Único diretório acessível pelo navegador
  - Contém o arquivo index.php que serve como ponto de entrada único
  - Aqui também ficariam arquivos CSS, JavaScript e imagens

- **vendor/**: Gerada automaticamente pelo Composer
  - Contém as dependências e o autoloader

> **Por que esta estrutura?** Separa claramente as responsabilidades, aumenta a segurança (apenas a pasta public é acessível) e facilita a manutenção do código.

## 7. Template Base com Bootstrap (app/Views/layouts/base.php)

Vamos criar um template base que será usado em todas as páginas do sistema. Utilizaremos o Bootstrap 5.3 para facilitar a criação de interfaces responsivas:

```php
<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <title><?= $title ?? 'Sistema' ?></title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
</head>
<body>
    <nav class="navbar navbar-expand-lg navbar-dark bg-dark">
        <div class="container">
            <a class="navbar-brand" href="/">Sistema</a>
            <ul class="navbar-nav">
                <li class="nav-item"><a class="nav-link" href="/">Home</a></li>
                <li class="nav-item"><a class="nav-link" href="/sobre">Sobre</a></li>
                <li class="nav-item"><a class="nav-link" href="/login">Login</a></li>
                <li class="nav-item"><a class="nav-link" href="/dashboard">Dashboard</a></li>
            </ul>
        </div>
    </nav>

    <div class="container mt-4">
        <?= $content ?>
    </div>

    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/js/bootstrap.bundle.min.js"></script>
</body>
</html>
```

**Explicação detalhada:**

- `<!DOCTYPE html>`: Define o documento como HTML5
- `<html lang="pt-br">`: Define o idioma da página como português do Brasil
- No `<head>`:
  - Definimos a codificação como UTF-8 para suportar caracteres especiais
  - O título da página usa o operador de coalescência nula (`??`) que significa: use `$title` se existir, senão use 'Sistema'
  - Carregamos o CSS do Bootstrap 5.3 via CDN (Content Delivery Network)

- No `<body>`:
  - Criamos uma barra de navegação escura com Bootstrap (`navbar-dark bg-dark`)
  - A navegação contém links para as principais páginas do sistema
  - `<div class="container mt-4">`: Cria um container com margem superior
  - `<?= $content ?>`: Aqui será injetado o conteúdo específico de cada página

- No final:
  - Carregamos o JavaScript do Bootstrap para componentes interativos

> **Sobre Bootstrap:** É um framework CSS que facilita a criação de interfaces responsivas. As classes como `container`, `navbar`, `mt-4` são do Bootstrap e formatam elementos automaticamente.

## 8. Roteador Simples (public/index.php)

Agora, vamos criar um sistema de rotas simples para direcionar as requisições para as páginas corretas:

```php
<?php

require __DIR__ . '/../vendor/autoload.php';

function render($view, $data = []) {
    extract($data);
    ob_start();
    require __DIR__ . '/../app/Views/' . $view;
    $content = ob_get_clean();
    require __DIR__ . '/../app/Views/layouts/base.php';
}

$url = parse_url($_SERVER['REQUEST_URI'], PHP_URL_PATH);

if ($url === '/' || $url === '/index.php') {
    render('home.php', ['title' => 'Página Inicial']);
} elseif ($url === '/sobre') {
    render('sobre.php', ['title' => 'Sobre']);
} elseif ($url === '/login') {
    render('login.php', ['title' => 'Login']);
} elseif ($url === '/dashboard') {
    render('dashboard.php', ['title' => 'Painel']);
} elseif ($url === '/usuarios') {
    render('usuarios/index.php', ['title' => 'Lista de Usuários']);
} elseif ($url === '/usuarios/novo') {
    render('usuarios/create.php', ['title' => 'Novo Usuário']);
} elseif ($url === '/usuarios/salvar' && $_SERVER['REQUEST_METHOD'] === 'POST') {
    echo "Usuário salvo!";
} elseif (preg_match('#^/usuarios/(\d+)$#', $url, $m)) {
    render('usuarios/show.php', ['usuarioId' => $m[1]]);
} elseif (preg_match('#^/usuarios/(\d+)/alterar$#', $url, $m)) {
    render('usuarios/edit.php', ['usuarioId' => $m[1]]);
} elseif (preg_match('#^/usuarios/(\d+)/deletar$#', $url, $m)) {
    echo "Usuário ID {$m[1]} deletado!";
} else {
    http_response_code(404);
    echo "Página não encontrada.";
}
```

**Explicação detalhada:**

1. **Inicialização:**
   - `require __DIR__ . '/../vendor/autoload.php';`: Carrega o autoloader do Composer

2. **Função render:**
   - Esta função facilita o carregamento de views com layout consistente
   - `extract($data);`: Converte chaves do array em variáveis (ex: `$data['title']` vira `$title`)
   - `ob_start();`: Inicia o buffer de saída para capturar o conteúdo
   - `require __DIR__ . '/../app/Views/' . $view;`: Carrega a view específica
   - `$content = ob_get_clean();`: Captura o conteúdo da view e limpa o buffer
   - `require __DIR__ . '/../app/Views/layouts/base.php';`: Carrega o template base

3. **Roteamento:**
   - `$url = parse_url($_SERVER['REQUEST_URI'], PHP_URL_PATH);`: Obtém a URL atual
   - Cada condição `if/elseif` representa uma rota diferente
   - Para rotas simples, usamos comparação direta: `$url === '/sobre'`
   - Para rotas com parâmetros, usamos expressões regulares: `preg_match('#^/usuarios/(\d+)$#', $url, $m)`
     - `(\d+)` captura números que serão armazenados em `$m[1]`
   - A rota `/usuarios/salvar` verifica se o método é POST (para salvar formulários)
   - Incluímos uma condição `else` para tratar URLs não encontradas (404)

> **Sobre roteamento:** Em projetos maiores, usaríamos um sistema de rotas mais robusto, mas esta implementação simples serve bem para entender o conceito.

## 9. Conexão PDO com IP e Porta (app/Core/Database.php)

Por fim, vamos criar a classe responsável pela conexão com o banco de dados usando PDO (PHP Data Objects), que é uma interface de acesso a banco de dados mais segura:

```php
<?php

namespace App\Core;

use PDO;
use PDOException;

class Database {
    public static function conectar() {
        $host = '127.0.0.1';
        $porta = '3306';
        $banco = 'sistema_projeto';
        $usuario = 'root';
        $senha = '';
        
        $dsn = "mysql:host=$host;port=$porta;dbname=$banco;charset=utf8";
        
        try {
            return new PDO($dsn, $usuario, $senha, [
                PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
                PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC
            ]);
        } catch (PDOException $e) {
            die("Erro na conexão: " . $e->getMessage());
        }
    }
}
```

**Explicação detalhada:**

1. **Namespace e Imports:**
   - `namespace App\Core;`: Define o namespace da classe (que corresponde à pasta app/Core/)
   - `use PDO;` e `use PDOException;`: Importa as classes do PHP para uso no código

2. **Classe Database:**
   - Uma classe estática que fornece métodos de conexão com o banco
   - `public static function conectar()`: Método que pode ser chamado sem instanciar a classe

3. **Configuração da Conexão:**
   - Define os parâmetros de conexão: host, porta, nome do banco, usuário e senha
   - `$dsn` (Data Source Name): String de conexão com todos os parâmetros

4. **Criação da Conexão PDO:**
   - `new PDO($dsn, $usuario, $senha, [...])`: Cria a conexão com os parâmetros fornecidos
   - Configurações adicionais:
     - `PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION`: Lança exceções quando ocorrem erros
     - `PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC`: Retorna resultados como arrays associativos

5. **Tratamento de Erros:**
   - `try/catch` captura erros de conexão
   - Se algo der errado, exibe a mensagem de erro

> **Por que usar PDO?** O PDO oferece maior segurança contra SQL Injection e suporta diversos bancos de dados, permitindo trocar facilmente entre MySQL, PostgreSQL, SQLite, etc.

## 10. Próximos Passos

Parabéns! Agora você tem a base de um sistema web completo com PHP e MySQL. Aqui estão os próximos passos para expandir seu projeto:

1. **Criar as views**: Desenvolva os arquivos PHP para home.php, sobre.php, login.php, etc.
2. **Implementar Controllers**: Crie classes para gerenciar a lógica de cada seção do sistema
3. **Desenvolver Models**: Implemente classes para manipular os dados no banco
4. **Autenticação de usuários**: Crie um sistema de login seguro
5. **Sessões e cookies**: Para manter usuários logados
6. **Validação de formulários**: Para garantir a integridade dos dados

---

Esta estrutura forma a base para um sistema PHP moderno, seguindo boas práticas de organização e segurança. Nos próximos módulos, expandiremos cada uma dessas áreas e transformaremos esta base em um sistema completo de gestão.

Lembre-se de praticar cada conceito individualmente antes de avançar. A prática é essencial para fixar o aprendizado!

---

## 📚 Recursos Adicionais

- [Documentação oficial do PHP](https://www.php.net/manual/pt_BR/)
- [Documentação do MySQL](https://dev.mysql.com/doc/)
- [Documentação do Composer](https://getcomposer.org/doc/)
- [Bootstrap 5.3 Documentation](https://getbootstrap.com/docs/5.3/getting-started/introduction/)
- [PHP The Right Way](https://phptherightway.com/)

---

Desenvolvido para o curso de PHP Seguro + MySQL com Rotas - Prof. Ronan Adriel Zenatti - FATEC Jaú
