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
```

### 2.2 Controller de Usuário

Crie o arquivo `app/Controllers/UsuarioController.php`:

```php
<?php
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

Com a base de CRUD que desenvolvemos, você pode facilmente expandir o sistema seguindo o mesmo padrão para outros elementos como produtos, categorias, vendas, etc. O padrão MVC implementado torna a manutenção e expansão do sistema muito mais organizadas e eficientes.
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
```

### 2.3 Views de Usuários

O formulário já existe (`app/Views/usuarios/form-usuario.php`), mas precisamos atualizar para processar os dados enviados. Adicione o código para exibir erros e para diferenciar entre novo usuário e edição:

```php
<nav aria-label="breadcrumb">
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
```

Também precisamos atualizar a visualização da lista de usuários (`app/Views/usuarios/listar-usuarios.php`) para exibir os dados reais:

```php
<nav aria-label="breadcrumb">
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

// Listar todos os usuários
if ($url == "/usuarios") {
    $usuarios = Usuario::buscarTodos();
    render("usuarios/listar-usuarios.php", [
        'title' => 'Usuários - LivroTech', 
        "usuarios" => $usuarios
    ]);
    exit;
} 

// Formulário de novo usuário
else if ($url == "/usuarios/novo") {
    render("usuarios/form-usuario.php", ['title' => 'Cadastrar Usuário - LivroTech']);
    exit;
}

// Salvar novo usuário (recebe POST do formulário)
else if ($url == "/usuarios/salvar" && $_SERVER['REQUEST_METHOD'] === 'POST') {
    $usuarioController->salvar();
    exit;
}

// Ver detalhes de um usuário específico
else if (preg_match('#^/usuarios/(\d+)$#', $url, $matches)) {
    $id = (int)$matches[1];
    $usuario = Usuario::buscarPorId($id);
    if (!$usuario) {
        $_SESSION['mensagem'] = "Usuário não encontrado!";
        $_SESSION['tipo_mensagem'] = "danger";
        header('Location: /usuarios');
        exit;
    }
    render("usuarios/ver-usuario.php", [
        'title' => 'Detalhes do Usuário - LivroTech',
        'usuario' => $usuario
    ]);
    exit;
}

// Formulário para editar usuário
else if (preg_match('#^/usuarios/(\d+)/editar$#', $url, $matches)) {
    $id = (int)$matches[1];
    $usuario = Usuario::buscarPorId($id);
    if (!$usuario) {
        $_SESSION['mensagem'] = "Usuário não encontrado!";
        $_SESSION['tipo_mensagem'] = "danger";
        header('Location: /usuarios');
        exit;
    }
    render("usuarios/form-usuario.php", [
        'title' => 'Editar Usuário - LivroTech',
        'usuario' => $usuario
    ]);
    exit;
}

// Processar atualização de usuário (recebe POST do formulário de edição)
else if (preg_match('#^/usuarios/(\d+)/atualizar$#', $url, $matches) && $_SERVER['REQUEST_METHOD'] === 'POST') {
    $id = (int)$matches[1];
    $usuarioController->atualizar($id);
    exit;
}

// Excluir usuário
else if (preg_match('#^/usuarios/(\d+)/excluir$#', $url, $matches)) {
    $id = (int)$matches[1];
    $usuarioController->excluir($id);
    exit;
}

// Para qualquer outra rota não encontrada
http_response_code(404);
echo '<h1>404 - Página não encontrada</h1>';
// render('404.php', ['title' => 'Página não encontrada - LivroTech']);
exit;