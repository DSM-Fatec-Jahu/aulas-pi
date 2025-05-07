Aula 3: Sistema de CRUD e Autenticação Segura para Projeto LivroTech
Sumário

Objetivos e Estrutura
CRUD de Usuários

Model de Usuário Completo
Controller de Usuário
Views de Usuários


Autenticação e Autorização

Model de Autenticação
Controller de Autenticação
View de Login


Configuração de Rotas
Conclusão e Próximos Passos

1. Objetivos e Estrutura
1.1 Objetivos

Desenvolver operações CRUD completas para usuários (Create, Read, Update, Delete)
Validar e sanitizar dados dos formulários
Implementar autenticação (login/logout) e proteção de rotas
Organizar código em camadas (Models, Controllers, Views) conforme o padrão MVC
Aplicar conceitos de segurança na manipulação de dados do usuário

1.2 Estrutura Atual do Projeto
Observando a estrutura atual do projeto, já temos:
projeto_vendas/
├── app/
│   ├── Controllers/      # Controladores do sistema (ainda incompleto)
│   ├── Core/             # Núcleo do sistema (Database.php já implementado)
│   ├── Models/           # Modelos para comunicação com o banco (Usuario.php iniciado)
│   └── Views/            # Visualizações HTML
│       ├── layouts/      # Layout base com Bootstrap
│       ├── usuarios/     # Views para CRUD de usuários
│       ├── auth/         # Views para autenticação
│       ├── dashboard.php # Painel administrativo
├── public/               # Ponto de entrada (index.php já iniciado)
│   ├── css/              # Estilos CSS
│   └── index.php         # Front controller
├── vendor/               # Pacotes do Composer
└── composer.json         # Configuração do Composer
2. CRUD de Usuários
2.1 Model de Usuário Completo
O arquivo app/Models/Usuario.php já foi iniciado, mas precisa ser expandido para incluir todas as operações CRUD:
php<?php
namespace App\Models;

use App\Core\Database;
use PDO;

class Usuario {
    // Propriedades da classe
    private $id_usuario;
    private $nome;
    private $email;
    private $senha;
    private $tipo;
    // outros campos conforme tabela usuarios

    // Busca todos os usuários
    public static function buscarTodos() {
        $pdo = Database::conectar();
        $sql = "SELECT * FROM usuarios WHERE deleted_at IS NULL";
        return $pdo->query($sql)->fetchAll();
    }

    // Busca um usuário pelo ID
    public static function buscarPorId($id) {
        $pdo = Database::conectar();
        $sql = "SELECT * FROM usuarios WHERE id_usuario = :id AND deleted_at IS NULL";
        $stmt = $pdo->prepare($sql);
        $stmt->bindParam(':id', $id, PDO::PARAM_INT);
        $stmt->execute();
        return $stmt->fetch();
    }

    // Busca um usuário pelo email
    public static function buscarPorEmail($email) {
        $pdo = Database::conectar();
        $sql = "SELECT * FROM usuarios WHERE email = :email AND deleted_at IS NULL";
        $stmt = $pdo->prepare($sql);
        $stmt->bindParam(':email', $email, PDO::PARAM_STR);
        $stmt->execute();
        return $stmt->fetch();
    }

    // Insere um novo usuário
    public static function inserir($dados) {
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
    public static function atualizar($id, $dados) {
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
    public static function excluir($id) {
        $pdo = Database::conectar();
        $sql = "UPDATE usuarios SET deleted_at = NOW() WHERE id_usuario = :id";
        $stmt = $pdo->prepare($sql);
        $stmt->bindParam(':id', $id, PDO::PARAM_INT);
        return $stmt->execute();
    }

    // Exclusão física (não recomendada para produção)
    public static function excluirFisicamente($id) {
        $pdo = Database::conectar();
        $sql = "DELETE FROM usuarios WHERE id_usuario = :id";
        $stmt = $pdo->prepare($sql);
        $stmt->bindParam(':id', $id, PDO::PARAM_INT);
        return $stmt->execute();
    }
}
2.2 Controller de Usuário
Crie o arquivo app/Controllers/UsuarioController.php:
php<?php
namespace App\Controllers;

use App\Models\Usuario;

class UsuarioController {
    
    // Exibe a lista de usuários
    public function listar() {
        $usuarios = Usuario::buscarTodos();
        require __DIR__ . '/../Views/usuarios/listar-usuarios.php';
    }
    
    // Exibe o formulário para criar um novo usuário
    public function novo() {
        require __DIR__ . '/../Views/usuarios/form-usuario.php';
    }
    
    // Exibe o formulário para editar um usuário existente
    public function editar($id) {
        $usuario = Usuario::buscarPorId($id);
        if (!$usuario) {
            $_SESSION['mensagem'] = "Usuário não encontrado!";
            $_SESSION['tipo_mensagem'] = "danger";
            header('Location: /usuarios');
            exit;
        }
        require __DIR__ . '/../Views/usuarios/form-usuario.php';
    }
    
    // Processa os dados do formulário para criar um novo usuário
    public function salvar() {
        $dados = $this->validarDados($_POST);
        
        // Verifica se já existe um usuário com este email
        if (Usuario::buscarPorEmail($dados['email'])) {
            $_SESSION['mensagem'] = "Este e-mail já está sendo utilizado!";
            $_SESSION['tipo_mensagem'] = "danger";
            header('Location: /usuarios/novo');
            exit;
        }
        
        if (Usuario::inserir($dados)) {
            $_SESSION['mensagem'] = "Usuário cadastrado com sucesso!";
            $_SESSION['tipo_mensagem'] = "success";
            header('Location: /usuarios');
        } else {
            $_SESSION['mensagem'] = "Erro ao cadastrar usuário!";
            $_SESSION['tipo_mensagem'] = "danger";
            header('Location: /usuarios/novo');
        }
        exit;
    }
    
    // Processa os dados do formulário para atualizar um usuário existente
    public function atualizar($id) {
        $usuario = Usuario::buscarPorId($id);
        if (!$usuario) {
            $_SESSION['mensagem'] = "Usuário não encontrado!";
            $_SESSION['tipo_mensagem'] = "danger";
            header('Location: /usuarios');
            exit;
        }
        
        $dados = $this->validarDados($_POST);
        
        // Verifica se o email já está sendo usado por outro usuário
        $usuarioExistente = Usuario::buscarPorEmail($dados['email']);
        if ($usuarioExistente && $usuarioExistente['id_usuario'] != $id) {
            $_SESSION['mensagem'] = "Este e-mail já está sendo utilizado por outro usuário!";
            $_SESSION['tipo_mensagem'] = "danger";
            header('Location: /usuarios/' . $id . '/editar');
            exit;
        }
        
        if (Usuario::atualizar($id, $dados)) {
            $_SESSION['mensagem'] = "Usuário atualizado com sucesso!";
            $_SESSION['tipo_mensagem'] = "success";
            header('Location: /usuarios');
        } else {
            $_SESSION['mensagem'] = "Erro ao atualizar usuário!";
            $_SESSION['tipo_mensagem'] = "danger";
            header('Location: /usuarios/' . $id . '/editar');
        }
        exit;
    }
    
    // Exclui um usuário (exclusão lógica)
    public function excluir($id) {
        if (Usuario::excluir($id)) {
            $_SESSION['mensagem'] = "Usuário excluído com sucesso!";
            $_SESSION['tipo_mensagem'] = "success";
        } else {
            $_SESSION['mensagem'] = "Erro ao excluir usuário!";
            $_SESSION['tipo_mensagem'] = "danger";
        }
        header('Location: /usuarios');
        exit;
    }
    
    // Valida e sanitiza os dados do formulário
    private function validarDados($dados) {
        $erros = [];
        $dadosValidados = [];
        
        // Validação do nome
        if (empty($dados['nome'])) {
            $erros[] = "O nome é obrigatório";
        } else {
            $dadosValidados['nome'] = htmlspecialchars(trim($dados['nome']));
        }
        
        // Validação do e-mail
        if (empty($dados['email'])) {
            $erros[] = "O e-mail é obrigatório";
        } elseif (!filter_var($dados['email'], FILTER_VALIDATE_EMAIL)) {
            $erros[] = "E-mail inválido";
        } else {
            $dadosValidados['email'] = filter_var($dados['email'], FILTER_SANITIZE_EMAIL);
        }
        
        // Validação da senha (apenas para novos usuários ou quando informada)
        if (empty($dados['id_usuario']) && empty($dados['senha'])) {
            $erros[] = "A senha é obrigatória para novos usuários";
        } elseif (!empty($dados['senha'])) {
            if (strlen($dados['senha']) < 6) {
                $erros[] = "A senha deve ter pelo menos 6 caracteres";
            } elseif ($dados['senha'] !== $dados['confirmar_senha']) {
                $erros[] = "As senhas não conferem";
            } else {
                $dadosValidados['senha'] = $dados['senha'];
            }
        }
        
        // Outros campos 
        $dadosValidados['cpf'] = empty($dados['cpf']) ? null : preg_replace('/[^0-9]/', '', $dados['cpf']);
        $dadosValidados['data_nascimento'] = empty($dados['data_nascimento']) ? null : $dados['data_nascimento'];
        $dadosValidados['celular'] = empty($dados['celular']) ? null : preg_replace('/[^0-9]/', '', $dados['celular']);
        $dadosValidados['rua'] = empty($dados['rua']) ? null : htmlspecialchars(trim($dados['rua']));
        $dadosValidados['numero'] = empty($dados['numero']) ? null : htmlspecialchars(trim($dados['numero']));
        $dadosValidados['complemento'] = empty($dados['complemento']) ? null : htmlspecialchars(trim($dados['complemento']));
        $dadosValidados['bairro'] = empty($dados['bairro']) ? null : htmlspecialchars(trim($dados['bairro']));
        $dadosValidados['cidade'] = empty($dados['cidade']) ? null : htmlspecialchars(trim($dados['cidade']));
        $dadosValidados['cep'] = empty($dados['cep']) ? null : preg_replace('/[^0-9]/', '', $dados['cep']);
        $dadosValidados['estado'] = empty($dados['estado']) ? null : htmlspecialchars(trim($dados['estado']));
        
        // Tipo de usuário
        if (empty($dados['tipo'])) {
            $erros[] = "O tipo de usuário é obrigatório";
        } elseif (!in_array($dados['tipo'], ['Administrador', 'Funcionário', 'Cliente'])) {
            $erros[] = "Tipo de usuário inválido";
        } else {
            $dadosValidados['tipo'] = $dados['tipo'];
        }
        
        // Se houver erros, armazena na sessão e redireciona
        if (!empty($erros)) {
            $_SESSION['erros'] = $erros;
            $_SESSION['dados'] = $dados;
            
            if (!empty($dados['id_usuario'])) {
                header('Location: /usuarios/' . $dados['id_usuario'] . '/editar');
            } else {
                header('Location: /usuarios/novo');
            }
            exit;
        }
        
        return $dadosValidados;
    }
}
2.3 Views de Usuários
O formulário já existe (app/Views/usuarios/form-usuario.php), mas precisamos atualizar para processar os dados enviados. Adicione o código para exibir erros e para diferenciar entre novo usuário e edição:
php<nav aria-label="breadcrumb">
    <ol class="breadcrumb">
        <li class="breadcrumb-item"><a href="/dashboard">Dashboard</a></li>
        <li class="breadcrumb-item"><a href="/usuarios">Usuários</a></li>
        <li class="breadcrumb-item active" aria-current="page">
            <?= isset($usuario) ? 'Editar Usuário' : 'Novo Usuário' ?>
        </li>
    </ol>
</nav>

<?php if (isset($_SESSION['erros']) && !empty($_SESSION['erros'])): ?>
<div class="alert alert-danger alert-dismissible fade show" role="alert">
    <strong>Erro!</strong>
    <ul>
        <?php foreach ($_SESSION['erros'] as $erro): ?>
            <li><?= $erro ?></li>
        <?php endforeach; ?>
    </ul>
    <button type="button" class="btn-close" data-bs-dismiss="alert" aria-label="Fechar"></button>
</div>
<?php unset($_SESSION['erros']); endif; ?>

<?php 
    // Recupera dados do formulário em caso de erro
    $dados = $_SESSION['dados'] ?? [];
    unset($_SESSION['dados']);
    
    // Se estiver editando, usa os dados do usuário
    if (isset($usuario) && !empty($usuario)) {
        $dados = array_merge($dados, $usuario);
    }
?>

<form action="<?= isset($usuario) ? '/usuarios/' . $usuario['id_usuario'] . '/atualizar' : '/usuarios/salvar' ?>" method="post">
    <!-- Aqui mantém o conteúdo do formulário já existente, mas adicionando os valores dos campos -->
    <div class="row g-4">
        <!-- Informações Pessoais -->
        <div class="col-lg-6">
            <div class="form-card card h-100">
                <div class="card-header bg-white">
                    <i class="fas fa-user text-primary"></i> Informações Pessoais
                </div>
                <div class="card-body p-4">
                    <div class="mb-3">
                        <label for="nomeCompleto" class="form-label">Nome Completo</label>
                        <input type="text" class="form-control" id="nomeCompleto" name="nome" value="<?= $dados['nome'] ?? '' ?>" required>
                    </div>
                    <div class="row">
                        <div class="col-md-6 mb-3">
                            <label for="cpf" class="form-label">CPF</label>
                            <input type="text" class="form-control" id="cpf" name="cpf" placeholder="123.456.789-00" value="<?= $dados['cpf'] ?? '' ?>" required>
                        </div>
                        <div class="col-md-6 mb-3">
                            <label for="dataNascimento" class="form-label">Data de Nascimento</label>
                            <input type="date" class="form-control" id="dataNascimento" name="data_nascimento" value="<?= $dados['data_nascimento'] ?? '' ?>" required>
                        </div>
                    </div>
                    <div class="">
                        <label for="telefone" class="form-label">Celular</label>
                        <input type="tel" class="form-control" id="telefone" name="celular" placeholder="(00) 00000-0000" value="<?= $dados['celular'] ?? '' ?>" required>
                    </div>
                </div>
            </div>
        </div>

        <!-- Informações de Acesso -->
        <div class="col-lg-6">
            <div class="form-card card h-100">
                <div class="card-header bg-white">
                    <i class="fas fa-lock text-primary"></i> Informações de Acesso
                </div>
                <div class="card-body p-4">
                    <div class="mb-3">
                        <label for="email" class="form-label">E-mail</label>
                        <input type="email" class="form-control" id="email" name="email" value="<?= $dados['email'] ?? '' ?>" required>
                    </div>
                    <div class="row">
                        <div class="col-md-6 mb-3">
                            <label for="senha" class="form-label">Senha <?= isset($usuario) ? '(deixe em branco para manter a atual)' : '' ?></label>
                            <input type="password" class="form-control" id="senha" name="senha" <?= isset($usuario) ? '' : 'required' ?>>
                        </div>
                        <div class="col-md-6 mb-3">
                            <label for="confirmarSenha" class="form-label">Confirmar Senha</label>
                            <input type="password" class="form-control" id="confirmarSenha" name="confirmar_senha" <?= isset($usuario) ? '' : 'required' ?>>
                        </div>
                    </div>
                    <div class="">
                        <label for="tipoUsuario" class="form-label">Tipo de Usuário</label>
                        <select class="form-select" id="tipoUsuario" name="tipo" required>
                            <option selected disabled value="">Selecione...</option>
                            <option value="Cliente" <?= (isset($dados['tipo']) && $dados['tipo'] == 'Cliente') ? 'selected' : '' ?>>Cliente</option>
                            <option value="Funcionário" <?= (isset($dados['tipo']) && $dados['tipo'] == 'Funcionário') ? 'selected' : '' ?>>Funcionário</option>
                            <option value="Administrador" <?= (isset($dados['tipo']) && $dados['tipo'] == 'Administrador') ? 'selected' : '' ?>>Administrador</option>
                        </select>
                    </div>
                </div>
            </div>
        </div>

        <!-- Continue com os campos de endereço como no formulário original, adicionando os valores dos dados -->
        <!-- ... -->

        <!-- Botões -->
        <div class="col-12">
            <div class="form-card card">
                <div class="card-body p-4 d-flex justify-content-between gap-3">
                    <a href="/usuarios" class="btn btn-outline-secondary">Voltar</a>
                    <button type="reset" class="btn btn-warning">Limpar</button>
                    <button type="submit" class="btn btn-primary">Salvar</button>
                </div>
            </div>
        </div>
    </div>
</form>
Também precisamos atualizar a visualização da lista de usuários (app/Views/usuarios/listar-usuarios.php) para exibir os dados reais:
php<nav aria-label="breadcrumb">
    <ol class="breadcrumb">
        <li class="breadcrumb-item"><a href="/dashboard">Dashboard</a></li>
        <li class="breadcrumb-item active" aria-current="page">Usuários</li>
    </ol>
</nav>

<?php if (isset($_SESSION['mensagem']) && !empty($_SESSION['mensagem'])): ?>
<div class="alert alert-<?= $_SESSION['tipo_mensagem'] ?> alert-dismissible fade show" role="alert">
    <?= $_SESSION['mensagem'] ?>
    <button type="button" class="btn-close" data-bs-dismiss="alert" aria-label="Fechar"></button>
</div>
<?php unset($_SESSION['mensagem'], $_SESSION['tipo_mensagem']); endif; ?>

<div
    class="d-flex justify-content-between flex-wrap flex-md-nowrap align-items-center pt-3 pb-2 mb-3 border-bottom">
    <h1 class="h2"><i class="fas fa-users me-2"></i>Lista de Usuários</h1>
    <a href="/usuarios/novo" class="btn btn-primary">
        <i class="fas fa-user-plus me-1"></i> Adicionar Novo Usuário
    </a>
</div>

<div class="card border-0 shadow-sm mb-4 table-container">
    <div class="card-body">
        <div class="row mb-3">
            <div class="col-md-6">
                <div class="input-group">
                    <input type="text" class="form-control" placeholder="Buscar usuários..."
                        aria-label="Buscar usuários">
                    <button class="btn btn-outline-secondary" type="button">
                        <i class="fas fa-search"></i>
                    </button>
                </div>
            </div>
            <div class="col-md-3 ms-auto">
                <select class="form-select">
                    <option selected>Todos os Usuários</option>
                    <option>Administradores</option>
                    <option>Funcionários</option>
                    <option>Clientes</option>
                </select>
            </div>
        </div>

        <div class="table-responsive">
            <table class="table table-hover align-middle">
                <thead>
                    <tr>
                        <th scope="col">ID</th>
                        <th scope="col">Nome</th>
                        <th scope="col">E-mail</th>
                        <th scope="col">Telefone</th>
                        <th scope="col">Tipo</th>
                        <th scope="col">Status</th>
                        <th scope="col">Ações</th>
                    </tr>
                </thead>
                <tbody>
                    <?php if (count($usuarios) > 0): ?>
                        <?php foreach ($usuarios as $usuario): ?>
                            <tr>
                                <td><?= $usuario['id_usuario'] ?></td>
                                <td><?= htmlspecialchars($usuario['nome']) ?></td>
                                <td><?= htmlspecialchars($usuario['email']) ?></td>
                                <td><?= htmlspecialchars($usuario['celular'] ?? 'Não informado') ?></td>
                                <td>
                                    <span class="badge <?= $usuario['tipo'] == 'Administrador' ? 'bg-primary' : ($usuario['tipo'] == 'Funcionário' ? 'bg-info text-dark' : 'bg-secondary') ?>">
                                        <?= htmlspecialchars($usuario['tipo']) ?>
                                    </span>
                                </td>
                                <td>
                                    <span class="badge bg-success">
                                        Ativo
                                    </span>
                                </td>
                                <td>
                                    <a href="/usuarios/<?= $usuario['id_usuario'] ?>"
                                        class="btn btn-sm btn-outline-primary btn-action" title="Visualizar">
                                        <i class="fas fa-eye"></i>
                                    </a>
                                    <a href="/usuarios/<?= $usuario['id_usuario'] ?>/editar"
                                        class="btn btn-sm btn-outline-success btn-action" title="Editar">
                                        <i class="fas fa-edit"></i>
                                    </a>
                                    <a href="/usuarios/<?= $usuario['id_usuario'] ?>/excluir" 
                                        onclick="return confirm('Tem certeza que deseja excluir este usuário?')"
                                        class="btn btn-sm btn-outline-danger btn-action" title="Excluir">
                                        <i class="fas fa-trash"></i>
                                    </a>
                                </td>
                            </tr>
                        <?php endforeach; ?>
                    <?php else: ?>
                        <tr>
                            <td colspan="7" class="text-center">Nenhum usuário cadastrado.</td>
                        </tr>
                    <?php endif; ?>
                </tbody>
            </table>
        </div>
    </div>
</div>
3. Autenticação e Autorização
3.1 Model de Autenticação
Crie o arquivo app/Models/Auth.php:
php<?php
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
        $sql = "SELECT id_usuario, nome, email, senha, tipo FROM usuarios WHERE emailTentar novamenteClaude atingiu o limite máximo de caracteres para uma mensagem e pausou sua resposta. Você pode escrever Continue para prosseguir com a conversa.RAContinuarEditarVou continuar com o desenvolvimento do modelo de autenticação:
php<?php
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
3.2 Controller de Autenticação
Crie o arquivo app/Controllers/AuthController.php:
php<?php
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
3.3 View de Login
A view de login já existe em app/Views/auth/login.php, mas precisamos atualizar para processar os dados enviados:
php<!DOCTYPE html>
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
4. Configuração de Rotas
Atualize o arquivo public/index.php para incluir as novas rotas:
php<?php
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
5. Conclusão e Próximos Passos
Nesta aula, desenvolvemos:

CRUD Completo de Usuários:

Model completo com todas as operações (listar, buscar, inserir, atualizar, excluir)
Controller para processar requisições e manipular a lógica de negócio
Views para exibir formulários e listas de usuários


Sistema de Autenticação e Autorização:

Login/logout seguro usando criptografia de senhas e sessões
Proteção de rotas com diferentes níveis de acesso
Validação de dados e feedback para o usuário


Roteamento Aprimorado:

Configuração de rotas públicas e protegidas
Uso de expressões regulares para capturar parâmetros da URL
Verificação de permissões para cada rota