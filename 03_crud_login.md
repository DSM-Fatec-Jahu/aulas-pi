**Aula: Sistema de Gerenciamento de Usuários Seguro em PHP**

---

## 1. Objetivos, Pré-requisitos e Estrutura do Projeto

### 1.1 Objetivos

* Implementar autenticação (login/logout) e autorização por sessão para proteger rotas.
* Construir em sequência as operações CRUD de usuários (Listar, Criar, Atualizar, Deletar).
* Validar entradas de usuário (campos obrigatórios, formato de e-mail, sanitização).
* Organizar o código em camadas: Models, Controllers, Rotas e Views.
* Inserir comentários detalhados para iniciantes.

### 1.2 Pré-requisitos

* PHP ≥ 7.4 com extensão PDO e servidor local (XAMPP, WAMP, etc.).
* MySQL com o banco `projeto_vendas` contendo tabela `usuarios` (id, nome, email) e tabela `admins` (id, usuario, senha\_hash).
* Composer instalado e configurado para autoload PSR-4 (`Classes\\` e `Controllers\\`).

### 1.3 Estrutura do Projeto

```bash
projeto_vendas/
├─ classes/
│   ├─ Database.php          # Conexão PDO
│   ├─ Usuario.php           # Model de usuário
│   └─ Auth.php              # Model de autenticação
├─ controllers/
│   ├─ AuthController.php    # Login/Logout
│   └─ UsuarioController.php # CRUD de usuários
├─ public/
│   └─ index.php             # Front controller (roteador)
├─ pages/
│   ├─ auth/
│   │   ├─ login.php         # Formulário de login
│   └─ usuarios/
│       ├─ index.php         # Listagem de usuários
│       └─ form.php          # Formulário de criação/edição
├─ database/
│   └─ projeto_vendas.sql    # Dump do banco
├─ composer.json
└─ vendor/
```

---

## 2. Autenticação e Autorização (Session)

### 2.1 Model de Autenticação (`classes/Auth.php`)

```php
<?php
namespace Classes;
use PDO;

class Auth {
    private $db;
    public function __construct() {
        // Inicia sessão e obtém conexão
        session_start();
        $this->db = (new Database())->getConnection();
    }
    /**
     * Verifica credenciais e inicia sessão
     */
    public function login(string $user, string $pass): bool {
        $sql = 'SELECT id, usuario, senha_hash FROM admins WHERE usuario = :u';
        $stmt = $this->db->prepare($sql);
        $stmt->execute(['u' => trim($user)]);
        $admin = $stmt->fetch(PDO::FETCH_ASSOC);
        if ($admin && password_verify($pass, $admin['senha_hash'])) {
            $_SESSION['admin_id'] = $admin['id'];
            $_SESSION['admin_usuario'] = $admin['usuario'];
            return true;
        }
        return false;
    }
    /**
     * Destrói sessão atual (logout)
     */
    public function logout(): void {
        session_unset();
        session_destroy();
    }
    /**
     * Retorna true se sessão ativa
     */
    public function check(): bool {
        return !empty($_SESSION['admin_id']);
    }
}
```

### 2.2 Controller de Autenticação (`controllers/AuthController.php`)

```php
<?php
namespace Controllers;
use Classes\Auth;

class AuthController {
    private $auth;
    public function __construct() {
        $this->auth = new Auth();
    }
    // Exibe formulário de login
    public function showLogin() {
        include __DIR__ . '/../pages/auth/login.php';
    }
    // Processa submissão do login
    public function login() {
        $user = $_POST['usuario'] ?? '';
        $pass = $_POST['senha'] ?? '';
        $errors = [];
        if (!$user || !$pass) {
            $errors[] = 'Informe usuário e senha.';
        }
        if (empty($errors) && $this->auth->login($user, $pass)) {
            header('Location: /usuarios'); exit;
        } elseif (empty($errors)) {
            $errors[] = 'Credenciais inválidas.';
        }
        include __DIR__ . '/../pages/auth/login.php';
    }
    // Logout
    public function logout() {
        $this->auth->logout();
        header('Location: /login'); exit;
    }
}
```

### 2.3 View de Login (`pages/auth/login.php`)

```php
<!DOCTYPE html>
<html lang="pt-br">
<head><meta charset="UTF-8"><title>Login</title></head>
<body>
<h1>Login</h1>
<?php if (!empty($errors)): ?>
  <ul style="color:red;">
    <?php foreach ($errors as $e): ?>
      <li><?= htmlspecialchars($e) ?></li>
    <?php endforeach; ?>
  </ul>
<?php endif; ?>
<form action="/login/processar" method="post">
  <label>Usuário:<br>
    <input type="text" name="usuario" value="<?= htmlspecialchars($_POST['usuario'] ?? '') ?>" required>
  </label><br><br>
  <label>Senha:<br>
    <input type="password" name="senha" required>
  </label><br><br>
  <button type="submit">Entrar</button>
</form>
</body>
</html>
```

---

## 3. Front Controller e Roteamento (`public/index.php`)

```php
<?php
require __DIR__ . '/../vendor/autoload.php';
use Classes\Auth;
use Controllers\AuthController;
use Controllers\UsuarioController;

$auth = new Auth();
$path = parse_url($_SERVER['REQUEST_URI'], PHP_URL_PATH);

// Rotas públicas
if ($path === '/login') {
    (new AuthController())->showLogin(); exit;
}
if ($path === '/login/processar' && $_SERVER['REQUEST_METHOD']==='POST') {
    (new AuthController())->login(); exit;
}
if ($path === '/logout') {
    (new AuthController())->logout(); exit;
}

// Protege demais rotas
if (!$auth->check()) {
    header('Location: /login'); exit;
}

// CRUD de Usuários
$ctrl = new UsuarioController();
switch (true) {
  case $path === '/usuarios':
    $ctrl->index(); break;
  case $path === '/usuarios/novo':
    $ctrl->novo(); break;
  case $path === '/usuarios/salvar' && $_SERVER['REQUEST_METHOD']==='POST':
    $ctrl->salvar(); break;
  case preg_match('#^/usuarios/(\d+)/editar$#',$path,$m):
    $ctrl->editar((int)$m[1]); break;
  case preg_match('#^/usuarios/(\d+)/salvar$#',$path,$m)&&$_SERVER['REQUEST_METHOD']==='POST':
    $ctrl->atualizar((int)$m[1]); break;
  case preg_match('#^/usuarios/(\d+)/deletar$#',$path,$m):
    $ctrl->deletar((int)$m[1]); break;
  default:
    http_response_code(404); echo 'Página não encontrada';
}
```

> **Comentário:** Roteamos login, logout e CRUD, protegendo todas as rotas de gerenciamento.

---

## 4. CRUD de Usuários em Sequência

### 4.1 Model de Usuário (`classes/Usuario.php`)

```php
<?php
namespace Classes;
use PDO;

class Usuario {
    private $db;
    public function __construct() {
        $this->db = (new Database())->getConnection();
    }
    // Listar todos
    public function listarTodos(): array {
        $stmt = $this->db->query('SELECT id,nome,email FROM usuarios');
        return $stmt->fetchAll(PDO::FETCH_ASSOC);
    }
    // Criar novo
    public function criar(string $nome, string $email): bool {
        $sql='INSERT INTO usuarios(nome,email)VALUES(:n,:e)';
        $stmt=$this->db->prepare($sql);
        return $stmt->execute(['n'=>trim($nome),'e'=>trim($email)]);
    }
    // Buscar por ID
    public function buscarPorId(int $id): ?array {
        $stmt=$this->db->prepare('SELECT id,nome,email FROM usuarios WHERE id=:i');
        $stmt->execute(['i'=>$id]);
        return $stmt->fetch(PDO::FETCH_ASSOC) ?: null;
    }
    // Atualizar
    public function atualizar(int $id,string $nome,string $email): bool {
        $sql='UPDATE usuarios SET nome=:n,email=:e WHERE id=:i';
        $stmt=$this->db->prepare($sql);
        return $stmt->execute(['i'=>$id,'n'=>trim($nome),'e'=>trim($email)]);
    }
    // Deletar
    public function deletar(int $id): bool {
        return $this->db->prepare('DELETE FROM usuarios WHERE id=:i')->execute(['i'=>$id]);
    }
}
```

> **Comentário:** Todas as operações do CRUD estão centralizadas neste Model.

### 4.2 Controller de Usuário (`controllers/UsuarioController.php`)

```php
<?php
namespace Controllers;
use Classes\Usuario;

class UsuarioController {
    private $model;
    public function __construct() {
        $this->model = new Usuario();
    }
    // Listagem (Read)
    public function index() {
        $usuarios = $this->model->listarTodos();
        include __DIR__.'/../pages/usuarios/index.php';
    }
    // Formulário Create
    public function novo() {
        $usuario=null; $errors=[];
        include __DIR__.'/../pages/usuarios/form.php';
    }
    // Processar Create
    public function salvar() {
        $nome=$_POST['nome']??'';
        $email=$_POST['email']??'';
        $errors=[];
        if(!$nome) $errors[]='Nome obrigatório';
        if(!$email||!filter_var($email,FILTER_VALIDATE_EMAIL)) $errors[]='E-mail inválido';
        if($errors){ include __DIR__.'/../pages/usuarios/form.php'; return;}        
        $this->model->criar($nome,$email);
        header('Location:/usuarios');
    }
    // Formulário Update
    public function editar(int $id) {
        $usuario=$this->model->buscarPorId($id);
        $errors=[];
        include __DIR__.'/../pages/usuarios/form.php';
    }
    // Processar Update
    public function atualizar(int $id) {
        $nome=$_POST['nome']??'';
        $email=$_POST['email']??'';
        $errors=[];
        if(!$nome) $errors[]='Nome obrigatório';
        if(!$email||!filter_var($email,FILTER_VALIDATE_EMAIL)) $errors[]='E-mail inválido';
        if($errors){ $usuario=['id'=>$id,'nome'=>$nome,'email'=>$email]; include __DIR__.'/../pages/usuarios/form.php'; return;}
        $this->model->atualizar($id,$nome,$email);
        header('Location:/usuarios');
    }
    // Delete
    public function deletar(int $id) {
        $this->model->deletar($id);
        header('Location:/usuarios');
    }
}
```

> **Comentário:** Validações de inputs ocorrem antes de cada operação de gravação.

### 4.3 Views de Usuários

* **Listagem**: `pages/usuarios/index.php`

```php
<a href="/logout">Sair</a>
<h1>Usuários</h1>
<a href="/usuarios/novo">Novo Usuário</a>
<table border="1" cellpadding="5">
  <tr><th>ID</th><th>Nome</th><th>E-mail</th><th>Ações</th></tr>
  <?php foreach($usuarios as $u): ?>
  <tr>
    <td><?=htmlspecialchars($u['id'])?></td>
    <td><?=htmlspecialchars($u['nome'])?></td>
    <td><?=htmlspecialchars($u['email'])?></td>
    <td>
      <a href="/usuarios/<?=$u['id']?>/editar">Editar</a> |
      <a href="/usuarios/<?=$u['id']?>/deletar" onclick="return confirm('Confirma exclusão?')">Deletar</a>
    </td>
  </tr>
  <?php endforeach; ?>
</table>
```

* **Formulário**: `pages/usuarios/form.php`

```php
<a href="/usuarios">Voltar</a>
<h1><?= $usuario ? 'Editar' : 'Novo' ?> Usuário</h1>
<?php if(!empty($errors)): ?><ul style="color:red;">
  <?php foreach($errors as $e): ?><li><?=htmlspecialchars($e)?></li><?php endforeach; ?>
</ul><?php endif; ?>
<form action="<?= $usuario?'/usuarios/'.$usuario['id'].'/salvar':'/usuarios/salvar'?>" method="post">
  <label>Nome:<br><input type="text" name="nome" value="<?=htmlspecialchars($_POST['nome']??$usuario['nome']??'')?>" required></label><br><br>
  <label>E-mail:<br><input type="email" name="email" value="<?=htmlspecialchars($_POST['email']??$usuario['email']??'')?>" required></label><br><br>
  <button type="submit"><?= $usuario?'Atualizar':'Criar' ?></button>
</form>
```

---

## 5. Conclusão e Próximos Passos

* Sistema com login/logout e proteção de rotas via sessão.
* CRUD completo com validações e feedback de erros.
* Arquitetura organizada em Models, Controllers, Views e Front Controller.

**Sugerido:**

* Implementar tokens CSRF para reforçar segurança.
* Introduzir níveis de permissão (roles) para autorização granular.
* Explorar frameworks leves de roteamento e templates.

**Parabéns!** Você concluiu um sistema seguro e completo para gerenciar usuários em PHP.
