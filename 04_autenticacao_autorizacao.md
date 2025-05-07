# Aula 4: Autenticação e Autorização no Projeto LivroTech

## 📋 Sumário da Aula

- [1. Introdução](#1-introdução)
- [2. Sistema de Autenticação](#2-sistema-de-autenticação)
  - [2.1 Model de Autenticação](#21-model-de-autenticação)
  - [2.2 Controller de Autenticação](#22-controller-de-autenticação)
  - [2.3 View de Login](#23-view-de-login)
- [3. Configuração de Rotas](#3-configuração-de-rotas)
- [4. Conclusão e Próximos Passos](#4-conclusão-e-próximos-passos)

## 1. Introdução

Nesta aula, vamos implementar um sistema de autenticação e autorização seguro para o projeto LivroTech. Isso permitirá:

- Login e logout de usuários
- Proteção de rotas administrativas
- Verificação de permissões baseada no tipo de usuário
- Feedback visual para ações de autenticação

Continuaremos usando a estrutura MVC (Model-View-Controller) para manter o código organizado e facilitar a manutenção.

## 2. Sistema de Autenticação

### 2.1 Model de Autenticação

Crie o arquivo `app/Models/Auth.php`:

```php
<?php
namespace App\Models;

use App\Core\Database;
use PDO;

class Auth {
    // Inicia a sessão se não estiver iniciada
    private static function iniciarSessao() {
        if (session_status() == PHP_SESSION_NONE) {
            session_start();
        }
    }
    
    // Verifica credenciais e realiza login
    public static function login($email, $senha) {
        self::iniciarSessao();
        
        // Busca o usuário pelo email
        $pdo = Database::conectar();
        $sql = "SELECT id_usuario, nome, email, senha, tipo FROM usuarios WHERE email = :email AND deleted_at IS NULL";
        $stmt = $pdo->prepare($sql);
        $stmt->bindParam(':email', $email, PDO::PARAM_STR);
        $stmt->execute();
        
        $usuario = $stmt->fetch(PDO::FETCH_ASSOC);
        
        // Verifica se encontrou o usuário e se a senha está correta
        if ($usuario && password_verify($senha, $usuario['senha'])) {
            // Armazena informações do usuário na sessão
            $_SESSION['usuario_id'] = $usuario['id_usuario'];
            $_SESSION['usuario_nome'] = $usuario['nome'];
            $_SESSION['usuario_email'] = $usuario['email'];
            $_SESSION['usuario_tipo'] = $usuario['tipo'];
            $_SESSION['logado'] = true;
            return true;
        }
        
        return false;
    }
    
    // Verifica se está logado
    public static function isLogado() {
        self::iniciarSessao();
        return isset($_SESSION['logado']) && $_SESSION['logado'] === true;
    }
    
    // Verifica se o usuário tem o tipo especificado
    public static function temPermissao($tipo) {
        self::iniciarSessao();
        if (!self::isLogado()) {
            return false;
        }
        
        // Se o tipo for um array, verifica se o usuário tem um dos tipos
        if (is_array($tipo)) {
            return in_array($_SESSION['usuario_tipo'], $tipo);
        }
        
        // Se o tipo for uma string, verifica se o usuário tem esse tipo
        return $_SESSION['usuario_tipo'] === $tipo;
    }
    
    // Faz logout do usuário
    public static function logout() {
        self::iniciarSessao();
        // Limpa todas as variáveis de sessão
        $_SESSION = array();
        
        // Destrói a sessão
        session_destroy();
    }
    
    // Recupera os dados do usuário logado
    public static function getUsuarioLogado() {
        self::iniciarSessao();
        if (!self::isLogado()) {
            return null;
        }
        
        return [
            'id' => $_SESSION['usuario_id'],
            'nome' => $_SESSION['usuario_nome'],
            'email' => $_SESSION['usuario_email'],
            'tipo' => $_SESSION['usuario_tipo']
        ];
    }
    
    // Redireciona para login se não estiver logado
    public static function requerLogin() {
        if (!self::isLogado()) {
            $_SESSION['mensagem'] = "Faça login para acessar esta página.";
            $_SESSION['tipo_mensagem'] = "warning";
            header('Location: /entrar');
            exit;
        }
    }
    
    // Redireciona se não tiver permissão
    public static function requerPermissao($tipo) {
        self::requerLogin();
        
        if (!self::temPermissao($tipo)) {
            $_SESSION['mensagem'] = "Você não tem permissão para acessar esta página.";
            $_SESSION['tipo_mensagem'] = "danger";
            header('Location: /dashboard');
            exit;
        }
    }
}
```

### 2.2 Controller de Autenticação

Crie o arquivo `app/Controllers/AuthController.php`:

```php
<?php
namespace App\Controllers;

use App\Models\Auth;
use App\Models\Usuario;

class AuthController {
    
    // Exibe a página de login
    public function showLogin() {
        // Se já estiver logado, redireciona para o dashboard
        if (Auth::isLogado()) {
            header('Location: /dashboard');
            exit;
        }
        
        require __DIR__ . '/../Views/auth/login.php';
    }
    
    // Processa o formulário de login
    public function login() {
        $email = filter_input(INPUT_POST, 'email', FILTER_SANITIZE_EMAIL);
        $senha = $_POST['senha'] ?? '';
        
        // Valida os campos
        $erros = [];
        if (empty($email)) {
            $erros[] = "O e-mail é obrigatório.";
        }
        if (empty($senha)) {
            $erros[] = "A senha é obrigatória.";
        }
        
        if (empty($erros)) {
            // Tenta fazer login
            if (Auth::login($email, $senha)) {
                // Login bem-sucedido, redireciona para o dashboard
                header('Location: /dashboard');
                exit;
            } else {
                $erros[] = "E-mail ou senha incorretos.";
            }
        }
        
        // Se chegou aqui, houve erros
        $_SESSION['erros'] = $erros;
        $_SESSION['email'] = $email; // Para preencher o campo de e-mail
        
        header('Location: /entrar');
        exit;
    }
    
    // Faz logout do sistema
    public function logout() {
        Auth::logout();
        $_SESSION['mensagem'] = "Logout realizado com sucesso!";
        $_SESSION['tipo_mensagem'] = "success";
        header('Location: /entrar');
        exit;
    }
    
    // Exibe o formulário de registro
    public function showRegistro() {
        require __DIR__ . '/../Views/auth/registro.php';
    }
    
    // Processa o formulário de registro
    public function registro() {
        $dados = [
            'nome' => filter_input(INPUT_POST, 'nome', FILTER_SANITIZE_SPECIAL_CHARS),
            'email' => filter_input(INPUT_POST, 'email', FILTER_SANITIZE_EMAIL),
            'senha' => $_POST['senha'] ?? '',
            'confirmar_senha' => $_POST['confirmar_senha'] ?? '',
            'tipo' => 'Cliente', // Usuários registrados são sempre clientes
        ];
        
        // Validações
        $erros = [];
        if (empty($dados['nome'])) {
            $erros[] = "O nome é obrigatório.";
        }
        
        if (empty($dados['email'])) {
            $erros[] = "O e-mail é obrigatório.";
        } elseif (!filter_var($dados['email'], FILTER_VALIDATE_EMAIL)) {
            $erros[] = "E-mail inválido.";
        } elseif (Usuario::buscarPorEmail($dados['email'])) {
            $erros[] = "Este e-mail já está sendo utilizado.";
        }
        
        if (empty($dados['senha'])) {
            $erros[] = "A senha é obrigatória.";
        } elseif (strlen($dados['senha']) < 6) {
            $erros[] = "A senha deve ter pelo menos 6 caracteres.";
        } elseif ($dados['senha'] !== $dados['confirmar_senha']) {
            $erros[] = "As senhas não conferem.";
        }
        
        if (empty($erros)) {
            // Tenta inserir o usuário
            if (Usuario::inserir($dados)) {
                $_SESSION['mensagem'] = "Cadastro realizado com sucesso! Faça login para continuar.";
                $_SESSION['tipo_mensagem'] = "success";
                header('Location: /entrar');
                exit;
            } else {
                $erros[] = "Erro ao cadastrar usuário.";
            }
        }
        
        // Se chegou aqui, houve erros
        $_SESSION['erros'] = $erros;
        $_SESSION['dados'] = $dados; // Para preencher os campos
        
        header('Location: /registrar');
        exit;
    }
    
    // Exibe o formulário de recuperação de senha
    public function showRecuperarSenha() {
        require __DIR__ . '/../Views/auth/recuperar-senha.php';
    }
    
    // Processa o formulário de recuperação de senha (simplificado)
    public function recuperarSenha() {
        $email = filter_input(INPUT_POST, 'email', FILTER_SANITIZE_EMAIL);
        
        if (empty($email)) {
            $_SESSION['erros'] = ["O e-mail é obrigatório."];
            header('Location: /recuperar-senha');
            exit;
        }
        
        // Em uma aplicação real, enviaria um e-mail com link para redefinir a senha
        // Para simplificar, apenas exibimos uma mensagem
        
        $_SESSION['mensagem'] = "Se este e-mail estiver cadastrado, você receberá um link para redefinir sua senha.";
        $_SESSION['tipo_mensagem'] = "info";
        header('Location: /entrar');
        exit;
    }
}
```

### 2.3 View de Login

A view de login já existe em `app/Views/auth/login.php`, mas precisamos atualizá-la para processar os dados enviados:

```php
<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Login - LivroTech</title>
    <!-- Bootstrap 5.3 CSS -->
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
    <!-- Font Awesome -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <!-- Custom CSS -->
    <link rel="stylesheet" href="/css/style.css">
</head>
<body>
    <!-- Cabeçalho -->
    <header class="bg-primary text-white">
        <nav class="navbar navbar-expand-lg navbar-dark">
            <div class="container">
                <a class="navbar-brand" href="/">
                    <i class="fas fa-book-open me-2"></i>LivroTech
                </a>
                <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navbarNav">
                    <span class="navbar-toggler-icon"></span>
                </button>
                <div class="collapse navbar-collapse" id="navbarNav">
                    <ul class="navbar-nav ms-auto">
                        <li class="nav-item">
                            <a class="nav-link" href="/">Início</a>
                        </li>
                        <li class="nav-item">
                            <a class="nav-link active" href="/entrar">Login</a>
                        </li>
                        <li class="nav-item">
                            <a class="nav-link" href="/registrar">Cadastrar</a>
                        </li>
                        <li class="nav-item">
                            <a class="nav-link" href="/sobre">Sobre</a>
                        </li>
                    </ul>
                </div>
            </div>
        </nav>
    </header>

    <!-- Conteúdo Principal -->
    <main class="auth-container py-5">
        <div class="container">
            <div class="row justify-content-center">
                <div class="col-md-6 col-lg-5">
                    <?php if (isset($_SESSION['mensagem']) && !empty($_SESSION['mensagem'])): ?>
                    <div class="alert alert-<?= $_SESSION['tipo_mensagem'] ?> alert-dismissible fade show" role="alert">
                        <?= $_SESSION['mensagem'] ?>
                        <button type="button" class="btn-close" data-bs-dismiss="alert" aria-label="Fechar"></button>
                    </div>
                    <?php unset($_SESSION['mensagem'], $_SESSION['tipo_mensagem']); endif; ?>
                    
                    <?php if (isset($_SESSION['erros']) && !empty($_SESSION['erros'])): ?>
                    <div class="alert alert-danger alert-dismissible fade show" role="alert">
                        <ul class="mb-0">
                            <?php foreach ($_SESSION['erros'] as $erro): ?>
                                <li><?= $erro ?></li>
                            <?php endforeach; ?>
                        </ul>
                        <button type="button" class="btn-close" data-bs-dismiss="alert" aria-label="Fechar"></button>
                    </div>
                    <?php unset($_SESSION['erros']); endif; ?>
                    
                    <div class="card shadow-lg border-0 rounded-lg animated">
                        <div class="card-header bg-primary text-white text-center py-3">
                            <h3 class="mb-0"><i class="fas fa-user-lock me-2"></i>Login</h3>
                        </div>
                        <div class="card-body p-4">
                            <div class="text-center mb-4">
                                <p>Entre com suas credenciais para acessar o sistema</p>
                            </div>
                            <form action="/entrar/processar" method="post">
                                <div class="mb-3">
                                    <label for="email" class="form-label">E-mail</label>
                                    <div class="input-group">
                                        <span class="input-group-text"><i class="fas fa-envelope"></i></span>
                                        <input type="email" class="form-control" id="email" name="email" 
                                               value="<?= isset($_SESSION['email']) ? htmlspecialchars($_SESSION['email']) : '' ?>"
                                               placeholder="seu@email.com" required>
                                    </div>
                                    <?php unset($_SESSION['email']); ?>
                                </div>
                                <div class="mb-3">
                                    <label for="senha" class="form-label">Senha</label>
                                    <div class="input-group">
                                        <span class="input-group-text"><i class="fas fa-lock"></i></span>
                                        <input type="password" class="form-control" id="senha" name="senha" placeholder="Sua senha" required>
                                    </div>
                                </div>
                                <div class="form-check mb-3">
                                    <input class="form-check-input" type="checkbox" id="lembrar" name="lembrar">
                                    <label class="form-check-label" for="lembrar">
                                        Lembrar de mim
                                    </label>
                                </div>
                                <div class="d-grid gap-2">
                                    <button type="submit" class="btn btn-primary btn-lg">Entrar</button>
                                </div>
                            </form>
                        </div>
                        <div class="card-footer text-center py-3">
                            <a href="/recuperar-senha" class="text-decoration-none">Esqueci minha senha</a>
                            <hr>
                            <div>Não tem uma conta? <a href="/registrar" class="text-decoration-none">Cadastre-se</a></div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </main>

    <!-- Rodapé -->
    <footer class="bg-dark text-white py-4">
        <div class="container">
            <div class="row">
                <div class="col-md-6">
                    <h5><i class="fas fa-book-open me-2"></i>LivroTech</h5>
                    <p>Sistema de Gestão para Livrarias</p>
                </div>
                <div class="col-md-6 text-md-end">
                    <ul class="list-inline mb-0">
                        <li class="list-inline-item"><a href="#" class="text-white"><i class="fab fa-facebook"></i></a></li>
                        <li class="list-inline-item"><a href="#" class="text-white"><i class="fab fa-instagram"></i></a></li>
                        <li class="list-inline-item"><a href="#" class="text-white"><i class="fab fa-twitter"></i></a></li>
                    </ul>
                    <p class="mt-2 mb-0">&copy; 2025 LivroTech. Todos os direitos reservados.</p>
                </div>
            </div>
        </div>
    </footer>

    <!-- Bootstrap 5.3 JS -->
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/js/bootstrap.bundle.min.js"></script>
</body>
</html>
```

## 3. Configuração de Rotas

Atualize o arquivo `public/index.php` para incluir as novas rotas de autenticação:

```php
<?php
// Importa o autoload do Composer para carregar as classes
require __DIR__ . '/../vendor/autoload.php';

// Inicia a sessão
session_start();

use App\Models\Usuario;
use App\Models\Auth;
use App\Controllers\UsuarioController;
use App\Controllers\AuthController;

// Injeta o conteúdo das páginas de rota dentro do template base.php
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

// Obtém a URL da requisição da navegação
$url = parse_url($_SERVER['REQUEST_URI'], PHP_URL_PATH);

// Controladores
$usuarioController = new UsuarioController();
$authController = new AuthController();

// Rotas públicas
if ($url == "/" || $url == "/index.php") {
    header('Location: /entrar');
    exit;
} else if ($url == '/sobre') {
    render_sem_login('sobre.php', ['title' => 'Sobre o Sistema - LivroTech']);
    exit;
} else if ($url == '/entrar') {
    $authController->showLogin();
    exit;
} else if ($url == '/entrar/processar' && $_SERVER['REQUEST_METHOD'] === 'POST') {
    $authController->login();
    exit;
} else if ($url == '/registrar') {
    $authController->showRegistro();
    exit;
} else if ($url == '/registrar/processar' && $_SERVER['REQUEST_METHOD'] === 'POST') {
    $authController->registro();
    exit;
} else if ($url == '/recuperar-senha') {
    $authController->showRecuperarSenha();
    exit;
} else if ($url == '/recuperar-senha/processar' && $_SERVER['REQUEST_METHOD'] === 'POST') {
    $authController->recuperarSenha();
    exit;
} else if ($url == '/sair') {
    $authController->logout();
    exit;
}

// Rotas protegidas (requer login)
Auth::requerLogin();

// Dashboard
if ($url == "/dashboard") {
    render('dashboard.php', ['title' => 'Dashboard - LivroTech']);
    exit;
}

// Rotas de usuários
else if ($url == "/usuarios") {
    // Apenas administradores e funcionários podem listar usuários
    Auth::requerPermissao(['Administrador', 'Funcionário']);
    $usuarios = Usuario::buscarTodos();
    render("usuarios/listar-usuarios.php", [
        'title' => 'Usuários - LivroTech', 
        "usuarios" => $usuarios
    ]);
    exit;
} 
else if ($url == "/usuarios/novo") {
    // Apenas administradores podem adicionar novos usuários
    Auth::requerPermissao('Administrador');
    render("usuarios/form-usuario.php", ['title' => 'Cadastrar Usuário - LivroTech']);
    exit;
}
else if ($url == "/usuarios/salvar" && $_SERVER['REQUEST_METHOD'] === 'POST') {
    Auth::requerPermissao('Administrador');
    $usuarioController->salvar();
    exit;
}
else if (preg_match('#^/usuarios/(\d+)$#', $url, $matches)) {
    Auth::requerPermissao(['Administrador', 'Funcionário']);
    $id = (int)$matches[1];
    $usuarioController->visualizar($id);
    exit;
}
else if (preg_match('#^/usuarios/(\d+)/editar$#', $url, $matches)) {
    Auth::requerPermissao('Administrador');
    $id = (int)$matches[1];
    $usuarioController->editar($id);
    exit;
}
else if (preg_match('#^/usuarios/(\d+)/atualizar$#', $url, $matches) && $_SERVER['REQUEST_METHOD'] === 'POST') {
    Auth::requerPermissao('Administrador');
    $id = (int)$matches[1];
    $usuarioController->atualizar($id);
    exit;
}
else if (preg_match('#^/usuarios/(\d+)/excluir$#', $url, $matches)) {
    Auth::requerPermissao('Administrador');
    $id = (int)$matches[1];
    $usuarioController->excluir($id);
    exit;
}

// Aqui irão outras rotas protegidas (produtos, vendas, etc.)

// Se nenhuma rota corresponder, exibe página 404
http_response_code(404);
echo '<h1>404 - Página não encontrada</h1>';
exit;
```

## 4. Conclusão e Próximos Passos

Nesta aula, desenvolvemos:

### Sistema de Autenticação e Autorização:

- **Autenticação Segura**: Implementamos login/logout com verificação de senhas criptografadas
- **Sessões Seguras**: Gerenciamento adequado de sessões para informações do usuário
- **Controle de Acesso**: Proteção de rotas com base em diferentes níveis de usuário
- **Feedback Visual**: Mensagens de erro e sucesso para melhor experiência do usuário

### Próximos Passos:

1. **Aprimorar o CRUD de Usuários**:
   - Adicionar paginação para listas grandes
   - Implementar filtros e busca de usuários
   - Melhorar a validação de dados

2. **Expandir para outros Módulos**:
   - CRUD de Produtos/Livros
   - Gerenciamento de Estoque
   - Sistema de Vendas e Controle de Caixa

3. **Melhorias de Segurança**:
   - Implementar proteção contra CSRF (Cross-Site Request Forgery)
   - Adicionar sistema de logs de atividades
   - Configurar regras de firewall e limitação de requisições

4. **Otimizações e Refatorações**:
   - Implementar um sistema de cache para consultas frequentes
   - Refatorar o código para maior reusabilidade
   - Adicionar testes automatizados

Com o sistema de autenticação e autorização implementado, o projeto LivroTech agora possui uma camada de segurança essencial para proteger tanto os recursos do sistema quanto os dados dos usuários. A estrutura organizada que desenvolvemos facilita a expansão e manutenção do sistema no futuro.

---

**Lembre-se**: A segurança é um processo contínuo. A implementação atual é um bom ponto de partida, mas sempre monitore e atualize os mecanismos de segurança conforme necessário.
