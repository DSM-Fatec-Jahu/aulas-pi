# Aula 3: Sistema de CRUD e Autenticação Segura para Projeto LivroTech

## 📋 Sumário da Aula

- [1. Objetivos e Estrutura](#1-objetivos-e-estrutura)
- [2. CRUD de Usuários](#2-crud-de-usuários)
  - [2.1 Model de Usuário Completo](#21-model-de-usuário-completo)
  - [2.2 Controller de Usuário](#22-controller-de-usuário)
  - [2.3 Views de Usuários](#23-views-de-usuários)
- [3. Configuração de Rotas](#3-configuração-de-rotas)
- [4. Conclusão e Próximos Passos](#4-conclusão-e-próximos-passos)

## 1. Objetivos e Estrutura

### 1.1 Objetivos

- Desenvolver operações CRUD completas para usuários (Create, Read, Update, Delete)
- Validar e sanitizar dados dos formulários
- Implementar autenticação (login/logout) e proteção de rotas
- Organizar código em camadas (Models, Controllers, Views) conforme o padrão MVC
- Aplicar conceitos de segurança na manipulação de dados do usuário

### 1.2 Estrutura Atual do Projeto

Observando a estrutura atual do projeto, já temos:

```
projeto_vendas/
├── app/
│   ├── Controllers/      # Controladores do sistema (ainda incompleto)
│   ├── Core/             # Núcleo do sistema (Database.php já implementado)
│   ├── Models/           # Modelos para comunicação com o banco (Usuario.php iniciado)
│   └── Views/            # Visualizações HTML
│       ├── layouts/      # Layout base com Bootstrap
│       ├── usuarios/     # Views para CRUD de usuários
│       ├── dashboard.php # Painel administrativo
├── public/               # Ponto de entrada (index.php já iniciado)
│   ├── css/              # Estilos CSS
│   └── index.php         # Front controller
├── vendor/               # Pacotes do Composer
└── composer.json         # Configuração do Composer
```

## 2. CRUD de Usuários

### 2.1 Model de Usuário Completo

O arquivo `app/Models/Usuario.php` já foi iniciado, mas precisa ser expandido para incluir todas as operações CRUD:

```php
<?php
namespace App\Models;

use App\Core\Database;
use PDO;

class Usuario {

    // Busca todos os usuários
    public static function all() {
        $pdo = Database::conectar();
        $sql = "SELECT * FROM usuarios WHERE deleted_at IS NULL";
        return $pdo->query($sql)->fetchAll();
    }

    // Busca um usuário pelo ID
    public static function find($id) {
        $pdo = Database::conectar();
        $sql = "SELECT * FROM usuarios WHERE id_usuario = :id AND deleted_at IS NULL";
        $stmt = $pdo->prepare($sql);
        $stmt->bindParam(':id', $id, PDO::PARAM_INT);
        $stmt->execute();
        return $stmt->fetch();
    }

    // Busca um usuário pelo email
    public static function findEmail($email) {
        $pdo = Database::conectar();
        $sql = "SELECT * FROM usuarios WHERE email = :email AND deleted_at IS NULL";
        $stmt = $pdo->prepare($sql);
        $stmt->bindParam(':email', $email, PDO::PARAM_STR);
        $stmt->execute();
        return $stmt->fetch();
    }

    // Insere um novo usuário
    public static function insert($dados) {
        $pdo = Database::conectar();

        // Criptografa a senha
        $senha_hash = password_hash($dados['senha'], PASSWORD_DEFAULT);

        $sql = "INSERT INTO usuarios (
            nome, cpf, data_nascimento, celular,
            rua, numero, complemento, bairro,
            cidade, cep, estado, email,
            tipo, senha
        ) VALUES (
            :nome, :cpf, :data_nascimento, :celular,
            :rua, :numero, :complemento, :bairro,
            :cidade, :cep, :estado, :email,
            :tipo, :senha
        )";

        $stmt = $pdo->prepare($sql);

        // Associa os parâmetros
        $stmt->bindParam(':nome', $dados['nome'], PDO::PARAM_STR);
        $stmt->bindParam(':cpf', $dados['cpf'], PDO::PARAM_STR);
        $stmt->bindParam(':data_nascimento', $dados['data_nascimento']);
        $stmt->bindParam(':celular', $dados['celular'], PDO::PARAM_STR);
        $stmt->bindParam(':rua', $dados['rua'], PDO::PARAM_STR);
        $stmt->bindParam(':numero', $dados['numero'], PDO::PARAM_STR);
        $stmt->bindParam(':complemento', $dados['complemento'], PDO::PARAM_STR);
        $stmt->bindParam(':bairro', $dados['bairro'], PDO::PARAM_STR);
        $stmt->bindParam(':cidade', $dados['cidade'], PDO::PARAM_STR);
        $stmt->bindParam(':cep', $dados['cep'], PDO::PARAM_STR);
        $stmt->bindParam(':estado', $dados['estado'], PDO::PARAM_STR);
        $stmt->bindParam(':email', $dados['email'], PDO::PARAM_STR);
        $stmt->bindParam(':tipo', $dados['tipo'], PDO::PARAM_STR);
        $stmt->bindParam(':senha', $senha_hash, PDO::PARAM_STR);

        return $stmt->execute();
    }

    // Atualiza um usuário existente
    public static function update($id, $dados) {
        $pdo = Database::conectar();

        // Verifica se a senha foi fornecida para atualização
        if (!empty($dados['senha'])) {
            $sql = "UPDATE usuarios SET
                nome = :nome, cpf = :cpf, data_nascimento = :data_nascimento,
                celular = :celular, rua = :rua, numero = :numero,
                complemento = :complemento, bairro = :bairro, cidade = :cidade,
                cep = :cep, estado = :estado, email = :email,
                tipo = :tipo, senha = :senha, updated_at = NOW()
                WHERE id_usuario = :id";

            $senha_hash = password_hash($dados['senha'], PASSWORD_DEFAULT);

        } else {
            $sql = "UPDATE usuarios SET
                nome = :nome, cpf = :cpf, data_nascimento = :data_nascimento,
                celular = :celular, rua = :rua, numero = :numero,
                complemento = :complemento, bairro = :bairro, cidade = :cidade,
                cep = :cep, estado = :estado, email = :email,
                tipo = :tipo, updated_at = NOW()
                WHERE id_usuario = :id";
        }

        $stmt = $pdo->prepare($sql);

        // Associa os parâmetros
        $stmt->bindParam(':id', $id, PDO::PARAM_INT);
        $stmt->bindParam(':nome', $dados['nome'], PDO::PARAM_STR);
        $stmt->bindParam(':cpf', $dados['cpf'], PDO::PARAM_STR);
        $stmt->bindParam(':data_nascimento', $dados['data_nascimento']);
        $stmt->bindParam(':celular', $dados['celular'], PDO::PARAM_STR);
        $stmt->bindParam(':rua', $dados['rua'], PDO::PARAM_STR);
        $stmt->bindParam(':numero', $dados['numero'], PDO::PARAM_STR);
        $stmt->bindParam(':complemento', $dados['complemento'], PDO::PARAM_STR);
        $stmt->bindParam(':bairro', $dados['bairro'], PDO::PARAM_STR);
        $stmt->bindParam(':cidade', $dados['cidade'], PDO::PARAM_STR);
        $stmt->bindParam(':cep', $dados['cep'], PDO::PARAM_STR);
        $stmt->bindParam(':estado', $dados['estado'], PDO::PARAM_STR);
        $stmt->bindParam(':email', $dados['email'], PDO::PARAM_STR);
        $stmt->bindParam(':tipo', $dados['tipo'], PDO::PARAM_STR);

        if (!empty($dados['senha'])) {
            $stmt->bindParam(':senha', $senha_hash, PDO::PARAM_STR);
        }

        return $stmt->execute();
    }

    // Exclusão lógica (soft delete)
    public static function softDelete($id) {
        $pdo = Database::conectar();
        $sql = "UPDATE usuarios SET deleted_at = NOW() WHERE id_usuario = :id";
        $stmt = $pdo->prepare($sql);
        $stmt->bindParam(':id', $id, PDO::PARAM_INT);
        return $stmt->execute();
    }

    // Exclusão física (não recomendada para produção)
    public static function fisicalDelete($id) {
        $pdo = Database::conectar();
        $sql = "DELETE FROM usuarios WHERE id_usuario = :id";
        $stmt = $pdo->prepare($sql);
        $stmt->bindParam(':id', $id, PDO::PARAM_INT);
        return $stmt->execute();
    }
}
```

### 2.2 Controller de Usuário

Crie o arquivo `app/Controllers/UsuarioController.php`:

```php
<?php
namespace App\Controllers;

use App\Models\Usuario;

class UsuarioController {

        public function index() {
        $usuarios = Usuario::all();
        render('usuarios/index.php', ['title' => 'Usuários', 'usuarios' => $usuarios]);
    }

    public function create() {
        render('usuarios/form-tabela.php', ['title' => 'Novo Usuário']);
    }

    public function store() {
        Usuario::create($_POST);
        header('Location: /usuarios');
    }

    public function edit($id) {
        $usuario = Usuario::find($id);
        render('usuarios/form-tabela.php', ['title' => 'Editar Usuário', 'usuario' => $usuario]);
    }

    public function update($id) {
        Usuario::update($id, $_POST);
        header('Location: /usuarios');
    }

    public function delete($id) {
        Usuario::delete($id);
        header('Location: /usuarios');
    }
```

### 2.3 Views de Usuários

O formulário já existe (`app/Views/usuarios/form-usuario.php`), mas precisamos atualizar para processar os dados enviados. Adicione o código para exibir erros e para diferenciar entre novo usuário e edição:

```php
<?php
    // Se estiver editando, usa os dados do usuário
    if (isset($usuario) && !empty($usuario)) {
        $dados = array_merge($dados, $usuario);
    }
?>
<h2><?= $usuario ? 'Editar Usuário' : 'Novo Usuário' ?></h2>
<form action="<?= $usuario ? '/usuarios/' . $usuario['id_usuario'] . '/atualizar' : '/usuarios/salvar' ?>" method="post">
    <label>Nome:</label><input type="text" name="nome" value="<?= $usuario['nome'] ?? '' ?>" required><br>
    <label>CPF:</label><input type="text" name="cpf" value="<?= $usuario['cpf'] ?? '' ?>"><br>
    <label>Email:</label><input type="email" name="email" value="<?= $usuario['email'] ?? '' ?>" required><br>
    <label>Tipo:</label>
    <select name="tipo" required>
        <option value="">Selecione</option>
        <option value="Administrador" <?= ($usuario['tipo'] ?? '') === 'Administrador' ? 'selected' : '' ?>>Administrador</option>
        <option value="Funcionário" <?= ($usuario['tipo'] ?? '') === 'Funcionário' ? 'selected' : '' ?>>Funcionário</option>
        <option value="Cliente" <?= ($usuario['tipo'] ?? '') === 'Cliente' ? 'selected' : '' ?>>Cliente</option>
    </select><br>

    <?php if (!$usuario): ?>
        <label>Senha:</label><input type="password" name="senha" required><br>
    <?php endif; ?>

    <button type="submit">Salvar</button>
</form>
```

Também precisamos atualizar a visualização da lista de usuários (`app/Views/usuarios/listar-usuarios.php`) para exibir os dados reais:

```php
<h2>Lista de Usuários</h2>
<a href="/usuarios/novo">Novo Usuário</a>
<table>
    <tr>
        <th>ID</th><th>Nome</th><th>Email</th><th>Tipo</th><th>Ações</th>
    </tr>
    <?php foreach ($usuarios as $u): ?>
        <tr>
            <td><?= $u['id_usuario'] ?></td>
            <td><?= $u['nome'] ?></td>
            <td><?= $u['email'] ?></td>
            <td><?= $u['tipo'] ?></td>
            <td>
                <a href="/usuarios/<?= $u['id_usuario'] ?>/editar">Editar</a> |
                <a href="/usuarios/<?= $u['id_usuario'] ?>/deletar">Excluir</a>
            </td>
        </tr>
    <?php endforeach; ?>
</table>
```

## 3. Configuração de Rotas

Para que o CRUD de usuários funcione corretamente, precisamos configurar as rotas no nosso front controller. Vamos atualizar o arquivo `public/index.php` para incluir todas as rotas necessárias:

```php
<?php
// Importa o autoload do Composer para carregar as classes
require __DIR__ . '/../vendor/autoload.php';

// Inicia a sessão para mensagens flash e autenticação
session_start();

use App\Models\Usuario;
use App\Controllers\UsuarioController;

// Injeta o conteudo das páginas de rota dentro do template base.php
function render($view, $data = [])
{
    extract($data);
    ob_start();
    // Carrega a página da rota
    require __DIR__ . '/../app/Views/' . $view;
    $content = ob_get_clean();
    // Carrega o template base.php
    require __DIR__ . '/../app/Views/layouts/base.php';
}

function render_sem_login($view, $data = [])
{
    extract($data);
    ob_start();
    $content = ob_get_clean();
    // Carrega a página da rota
    require __DIR__ . '/../app/Views/' . $view;
}

// Instancia o controlador de usuários
$usuarioController = new UsuarioController();

// Obtem a URL da requisição da navegação
$url = parse_url($_SERVER['REQUEST_URI'], PHP_URL_PATH);

// Rotas públicas (sem autenticação)
if ($url == "/" || $url == "/index.php") {
    header('Location: /entrar');
    exit;
} else if ($url == '/sobre') {
    render_sem_login('sobre.php', ['title' => 'Sobre o Sistema - LivroTech']);
    exit;
} else if ($url == '/entrar') {
    render_sem_login('auth/login.php', ['title' => 'Entrar no Sistema - LivroTech']);
    exit;
}

// Rotas do Dashboard e administrativas
if ($url == "/dashboard") {
    render('dashboard.php', ['title' => 'Dashboard - LivroTech']);
    exit;
}

// Rotas de Usuários
if ($url === '/usuarios') {
    $usuarioCtrl->index();
} elseif ($url === '/usuarios/novo') {
    $usuarioCtrl->create();
} elseif ($url === '/usuarios/salvar' && $_SERVER['REQUEST_METHOD'] === 'POST') {
    $usuarioCtrl->store();
} elseif (preg_match('#^/usuarios/(\d+)/editar$#', $url, $m)) {
    $usuarioCtrl->edit($m[1]);
} elseif (preg_match('#^/usuarios/(\d+)/atualizar$#', $url, $m) && $_SERVER['REQUEST_METHOD'] === 'POST') {
    $usuarioCtrl->update($m[1]);
} elseif (preg_match('#^/usuarios/(\d+)/deletar$#', $url, $m)) {
    $usuarioCtrl->delete($m[1]);
} else{
    http_response_code(404);
    echo '<h1>404 - Página não encontrada</h1>';
    // render('404.php', ['title' => 'Página não encontrada - LivroTech']);
}
```

## 4. Conclusão e Próximos Passos

Nesta aula, implementamos um CRUD completo para usuários no sistema LivroTech. Isso inclui:

### O que foi desenvolvido:

1. **Model de Usuário Completo**:

   - Operações de busca, inserção, atualização e exclusão
   - Criptografia segura de senhas com `password_hash()`
   - Métodos especializados para busca por email e ID

2. **Controller de Usuário**:

   - Processamento e validação de formulários
   - Gerenciamento de fluxo entre Model e View
   - Tratamento de erros e feedback para o usuário

3. **Views para CRUD de Usuários**:

   - Formulário para criação/edição
   - Listagem com ações (visualizar, editar, excluir)
   - Exibição de mensagens de sucesso/erro

4. **Sistema de Rotas**:
   - Definição de rotas para cada operação CRUD
   - Captura de parâmetros dinâmicos
   - Separação entre rotas públicas e administrativas

### Próximos Passos:

1. **Implementar Autenticação e Autorização**:

   - Sistema de login/logout
   - Proteção de rotas administrativas
   - Controle de acesso baseado no tipo de usuário

2. **Expandir para outros Módulos**:

   - CRUD de produtos/livros
   - Gerenciamento de vendas
   - Dashboard com estatísticas

3. **Melhorias de Segurança**:

   - Validação mais rigorosa de entradas
   - Proteção contra CSRF (Cross-Site Request Forgery)
   - Logs de atividades administrativas

4. **Otimizações e Refinamentos**:
   - Paginação para listas grandes
   - Filtros e busca avançada
   - Melhorias de UX/UI
