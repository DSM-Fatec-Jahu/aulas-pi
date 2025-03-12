# Fundamentos de PHP para Graduação

## 1. Conceitos Básicos do PHP

### 1.1 Introdução ao PHP

PHP (PHP: Hypertext Preprocessor) é uma linguagem de script de código aberto amplamente utilizada para desenvolvimento web. Ela pode ser incorporada diretamente no HTML e é executada no servidor, gerando HTML que é enviado para o navegador do cliente.

#### Características principais:
- Linguagem de script server-side
- Gratuita e de código aberto
- Especializada em desenvolvimento web
- Possui ampla compatibilidade com diversos bancos de dados
- Suporta programação procedural e orientada a objetos

### 1.2 Configuração Básica

Um arquivo PHP começa com `<?php` e termina com `?>`. A extensão do arquivo deve ser `.php`:

```php
<?php
    // Código PHP aqui
    echo "Olá, mundo!";
?>
```

Obs: Em arquivos PHP puros, é recomendado omitir a tag de fechamento `?>` para evitar problemas com caracteres em branco.

### 1.3 Variáveis

Em PHP, as variáveis são prefixadas com o símbolo `$` e são tipadas dinamicamente:

```php
<?php
    // Declaração de variáveis
    $nome = "Maria";           // String
    $idade = 25;               // Inteiro
    $altura = 1.75;            // Ponto flutuante
    $estudante = true;         // Booleano
    $notas = [8.5, 9.0, 7.5];  // Array
    $pessoa = null;            // NULL
    
    // Exibição de variáveis
    echo "Nome: $nome, Idade: $idade, Altura: $altura m";
    // Saída: Nome: Maria, Idade: 25, Altura: 1.75 m
    
    // Concatenação com o operador '.'
    echo "<br>O aluno " . $nome . " tem " . $idade . " anos.";
    // Saída: O aluno Maria tem 25 anos.
?>
```

#### Escopo de variáveis:

PHP possui três escopos principais:

```php
<?php
    // Variável global
    $x = 10;
    
    function teste() {
        // Variável local
        $y = 5;
        
        // Para acessar variáveis globais dentro de funções:
        global $x;
        echo $x; // Agora funciona!
        
        // Alternativa: array $GLOBALS
        echo $GLOBALS['x']; // Também funciona!
    }
    
    teste();
    
    // Variáveis estáticas: mantêm seus valores entre chamadas de função
    function contador() {
        static $contador = 0;
        $contador++;
        echo $contador;
    }
    
    contador(); // Exibe: 1
    contador(); // Exibe: 2
    contador(); // Exibe: 3
?>
```

### 1.4 Tipos de dados

PHP suporta oito tipos primitivos:

#### Tipos escalares:
- `bool`: valores booleanos (true/false)
- `int`: números inteiros
- `float`: números de ponto flutuante
- `string`: sequências de caracteres

#### Tipos compostos:
- `array`: coleção ordenada de valores
- `object`: instância de uma classe
- `callable`: funções ou métodos que podem ser chamados
- `iterable`: arrays ou objetos que podem ser iterados

```php
<?php
    // Verificando o tipo de uma variável
    $valor = 42;
    echo gettype($valor); // Exibe: integer
    
    // Verificação de tipo
    var_dump(is_int($valor));     // bool(true)
    var_dump(is_string($valor));  // bool(false)
    
    // Conversão de tipo (casting)
    $numero = 3.14;
    $inteiro = (int)$numero;
    echo $inteiro; // Exibe: 3
    
    $texto = "42";
    $numero = (int)$texto;
    echo $numero; // Exibe: 42
?>
```

### 1.5 Estruturas Condicionais

#### if, else, elseif/else if

```php
<?php
    $nota = 85;
    
    // Estrutura if simples
    if ($nota >= 60) {
        echo "Aprovado!";
    }
    
    // Estrutura if-else
    if ($nota >= 60) {
        echo "Aprovado!";
    } else {
        echo "Reprovado!";
    }
    
    // Estrutura if-elseif-else
    if ($nota >= 90) {
        echo "Conceito A";
    } elseif ($nota >= 80) {
        echo "Conceito B";
    } elseif ($nota >= 70) {
        echo "Conceito C";
    } elseif ($nota >= 60) {
        echo "Conceito D";
    } else {
        echo "Conceito F";
    }
    
    // Sintaxe alternativa (útil em templates)
    if ($nota >= 60): 
        echo "Aprovado!";
    else:
        echo "Reprovado!";
    endif;
?>
```

#### Operador ternário

```php
<?php
    $idade = 20;
    
    // Formato: condição ? valor_se_verdadeiro : valor_se_falso
    $status = ($idade >= 18) ? "Maior de idade" : "Menor de idade";
    echo $status; // Exibe: Maior de idade
    
    // Operador de coalescência nula (PHP 7+)
    $nome = $_GET['nome'] ?? "Visitante";
    // Equivalente a: $nome = isset($_GET['nome']) ? $_GET['nome'] : "Visitante";
?>
```

#### switch

```php
<?php
    $dia = 3;
    
    switch ($dia) {
        case 1:
            echo "Domingo";
            break;
        case 2:
            echo "Segunda-feira";
            break;
        case 3:
            echo "Terça-feira";
            break;
        case 4:
            echo "Quarta-feira";
            break;
        case 5:
            echo "Quinta-feira";
            break;
        case 6:
            echo "Sexta-feira";
            break;
        case 7:
            echo "Sábado";
            break;
        default:
            echo "Dia inválido";
    }
    // Exibe: Terça-feira
    
    // Sintaxe alternativa
    switch ($dia):
        case 1:
            echo "Domingo";
            break;
        // ... outros casos ...
    endswitch;
?>
```

### 1.6 Estruturas de Repetição

#### for

```php
<?php
    // Loop for básico
    for ($i = 1; $i <= 5; $i++) {
        echo "$i ";
    }
    // Saída: 1 2 3 4 5
    
    // Loop for com um array
    $frutas = ["Maçã", "Banana", "Laranja", "Uva", "Morango"];
    for ($i = 0; $i < count($frutas); $i++) {
        echo $frutas[$i] . " ";
    }
    // Saída: Maçã Banana Laranja Uva Morango
?>
```

#### while

```php
<?php
    // Loop while básico
    $contador = 1;
    while ($contador <= 5) {
        echo "$contador ";
        $contador++;
    }
    // Saída: 1 2 3 4 5
    
    // Sintaxe alternativa
    $contador = 1;
    while ($contador <= 5):
        echo "$contador ";
        $contador++;
    endwhile;
    // Saída: 1 2 3 4 5
?>
```

#### do-while

```php
<?php
    // Loop do-while (executa pelo menos uma vez)
    $contador = 1;
    do {
        echo "$contador ";
        $contador++;
    } while ($contador <= 5);
    // Saída: 1 2 3 4 5
    
    // Exemplo de loop que sempre executa pelo menos uma vez
    $numero = 10;
    do {
        echo "Este texto será exibido uma vez!";
    } while ($numero < 5);
?>
```

#### foreach

```php
<?php
    // Iterando sobre um array indexado
    $cores = ["Vermelho", "Verde", "Azul", "Amarelo"];
    foreach ($cores as $cor) {
        echo "$cor ";
    }
    // Saída: Vermelho Verde Azul Amarelo
    
    // Iterando sobre um array associativo
    $pessoa = [
        "nome" => "João",
        "idade" => 30,
        "profissao" => "Programador"
    ];
    
    foreach ($pessoa as $chave => $valor) {
        echo "$chave: $valor<br>";
    }
    /* Saída:
    nome: João
    idade: 30
    profissao: Programador
    */
    
    // Modificando valores durante a iteração (usando referência &)
    $numeros = [1, 2, 3, 4, 5];
    foreach ($numeros as &$numero) {
        $numero *= 2;
    }
    unset($numero); // Importante: limpar a referência
    
    print_r($numeros);
    // Saída: Array ( [0] => 2 [1] => 4 [2] => 6 [3] => 8 [4] => 10 )
?>
```

### 1.7 Arrays (Vetores)

Em PHP, os arrays podem ser indexados por números (arrays indexados) ou por strings (arrays associativos).

#### Arrays indexados

```php
<?php
    // Declaração de array indexado
    $frutas = ["Maçã", "Banana", "Laranja"];
    
    // Forma alternativa de declaração
    $frutas = array("Maçã", "Banana", "Laranja");
    
    // Acessando elementos do array
    echo $frutas[0]; // Exibe: Maçã
    
    // Adicionando elementos
    $frutas[] = "Uva";
    array_push($frutas, "Morango");
    
    // Contando elementos
    echo count($frutas); // Exibe: 5
    
    // Verificando se um valor existe
    if (in_array("Banana", $frutas)) {
        echo "Banana está no array!";
    }
    
    // Encontrando o índice de um valor
    $indice = array_search("Laranja", $frutas);
    echo $indice; // Exibe: 2
    
    // Removendo elementos
    unset($frutas[1]); // Remove "Banana"
    
    // Reindexando o array após remover elementos
    $frutas = array_values($frutas);
?>
```

#### Arrays associativos

```php
<?php
    // Declaração de array associativo
    $aluno = [
        "nome" => "Ana Silva",
        "idade" => 22,
        "curso" => "Ciência da Computação",
        "notas" => [8.5, 9.0, 7.5]
    ];
    
    // Acessando elementos
    echo $aluno["nome"]; // Exibe: Ana Silva
    echo $aluno["notas"][1]; // Exibe: 9.0
    
    // Verificando se uma chave existe
    if (array_key_exists("curso", $aluno)) {
        echo "A chave 'curso' existe!";
    }
    
    // Obtendo todas as chaves
    $chaves = array_keys($aluno);
    print_r($chaves);
    
    // Obtendo todos os valores
    $valores = array_values($aluno);
    print_r($valores);
?>
```

#### Arrays multidimensionais

```php
<?php
    // Array multidimensional
    $turmas = [
        "A" => [
            ["nome" => "João", "nota" => 8.5],
            ["nome" => "Maria", "nota" => 9.0],
            ["nome" => "Pedro", "nota" => 7.5]
        ],
        "B" => [
            ["nome" => "Ana", "nota" => 9.5],
            ["nome" => "Carlos", "nota" => 8.0],
            ["nome" => "Lúcia", "nota" => 8.5]
        ]
    ];
    
    // Acessando elementos
    echo $turmas["A"][1]["nome"]; // Exibe: Maria
    echo $turmas["B"][0]["nota"]; // Exibe: 9.5
    
    // Percorrendo um array multidimensional
    foreach ($turmas as $turma => $alunos) {
        echo "Turma $turma:<br>";
        foreach ($alunos as $aluno) {
            echo "- {$aluno['nome']}: {$aluno['nota']}<br>";
        }
    }
?>
```

#### Funções úteis para arrays

```php
<?php
    $numeros = [5, 2, 8, 1, 9];
    
    // Ordenação
    sort($numeros); // Crescente: [1, 2, 5, 8, 9]
    rsort($numeros); // Decrescente: [9, 8, 5, 2, 1]
    
    $pessoa = ["nome" => "Ana", "idade" => 25, "cidade" => "São Paulo"];
    
    // Ordenação mantendo associações chave => valor
    asort($pessoa); // Ordena por valor, mantém chaves
    ksort($pessoa); // Ordena por chave
    
    // Funções de manipulação
    $slice = array_slice($numeros, 1, 3); // Extrai uma parte do array
    $merged = array_merge($numeros, [10, 11, 12]); // Mescla arrays
    $filtered = array_filter($numeros, function($n) { return $n > 5; }); // Filtra elementos
    $mapped = array_map(function($n) { return $n * 2; }, $numeros); // Mapeia/transforma elementos
    
    // Funções de redução
    $soma = array_sum($numeros); // Soma todos os valores
    $produto = array_product($numeros); // Multiplica todos os valores
    $resultado = array_reduce($numeros, function($carry, $item) {
        return $carry + $item;
    }, 0); // Reduz o array a um único valor
?>
```

### 1.8 Orientação a Objetos

PHP suporta programação orientada a objetos (POO) com classes, objetos, herança, interfaces, traits e mais.

#### Classes e objetos

```php
<?php
    // Definição de uma classe
    class Pessoa {
        // Propriedades (atributos)
        public $nome;
        private $idade;
        protected $email;
        
        // Construtor
        public function __construct($nome, $idade, $email) {
            $this->nome = $nome;
            $this->idade = $idade;
            $this->email = $email;
            echo "Um objeto Pessoa foi criado!<br>";
        }
        
        // Destrutor
        public function __destruct() {
            echo "O objeto Pessoa foi destruído!<br>";
        }
        
        // Método
        public function apresentar() {
            return "Olá, meu nome é {$this->nome} e tenho {$this->idade} anos.";
        }
        
        // Getters e Setters
        public function getIdade() {
            return $this->idade;
        }
        
        public function setIdade($idade) {
            // Validação simples
            if ($idade > 0 && $idade < 120) {
                $this->idade = $idade;
                return true;
            }
            return false;
        }
        
        // Método estático
        public static function calcularIdadePorAnoNascimento($anoNascimento) {
            $anoAtual = date("Y");
            return $anoAtual - $anoNascimento;
        }
    }
    
    // Criando um objeto
    $pessoa1 = new Pessoa("João", 30, "joao@email.com");
    echo $pessoa1->nome; // Exibe: João
    
    // Chamando um método
    echo $pessoa1->apresentar(); // Exibe: Olá, meu nome é João e tenho 30 anos.
    
    // Usando getters e setters
    echo $pessoa1->getIdade(); // Exibe: 30
    $pessoa1->setIdade(31);
    echo $pessoa1->getIdade(); // Exibe: 31
    
    // Chamando um método estático
    echo Pessoa::calcularIdadePorAnoNascimento(1990); // Exibe a idade atual
?>
```

#### Herança

```php
<?php
    // Classe base (pai)
    class Animal {
        protected $nome;
        
        public function __construct($nome) {
            $this->nome = $nome;
        }
        
        public function emitirSom() {
            return "Som genérico de animal";
        }
        
        public function getNome() {
            return $this->nome;
        }
    }
    
    // Classe derivada (filha)
    class Cachorro extends Animal {
        private $raca;
        
        public function __construct($nome, $raca) {
            parent::__construct($nome); // Chamando o construtor da classe pai
            $this->raca = $raca;
        }
        
        // Sobrescrevendo o método da classe pai
        public function emitirSom() {
            return "Au au!";
        }
        
        public function getRaca() {
            return $this->raca;
        }
    }
    
    // Usando as classes
    $animal = new Animal("Animal Genérico");
    echo $animal->emitirSom(); // Exibe: Som genérico de animal
    
    $cachorro = new Cachorro("Rex", "Labrador");
    echo $cachorro->getNome(); // Exibe: Rex
    echo $cachorro->getRaca(); // Exibe: Labrador
    echo $cachorro->emitirSom(); // Exibe: Au au!
?>
```

#### Abstração e interfaces

```php
<?php
    // Classe abstrata
    abstract class Forma {
        protected $cor;
        
        public function __construct($cor) {
            $this->cor = $cor;
        }
        
        // Método abstrato (deve ser implementado pelas classes filhas)
        abstract public function calcularArea();
        
        // Método concreto
        public function getCor() {
            return $this->cor;
        }
    }
    
    // Interface
    interface Redimensionavel {
        public function redimensionar($fator);
    }
    
    // Implementando classe abstrata e interface
    class Retangulo extends Forma implements Redimensionavel {
        private $largura;
        private $altura;
        
        public function __construct($cor, $largura, $altura) {
            parent::__construct($cor);
            $this->largura = $largura;
            $this->altura = $altura;
        }
        
        // Implementação do método abstrato
        public function calcularArea() {
            return $this->largura * $this->altura;
        }
        
        // Implementação do método da interface
        public function redimensionar($fator) {
            $this->largura *= $fator;
            $this->altura *= $fator;
        }
    }
    
    // Usando as classes
    $retangulo = new Retangulo("Azul", 5, 3);
    echo "Área: " . $retangulo->calcularArea(); // Exibe: Área: 15
    echo "Cor: " . $retangulo->getCor(); // Exibe: Cor: Azul
    
    $retangulo->redimensionar(2);
    echo "Nova área: " . $retangulo->calcularArea(); // Exibe: Nova área: 60
?>
```

#### Traits

Traits permitem reutilização de código em um sistema de herança única:

```php
<?php
    // Definindo um trait
    trait Logger {
        public function log($mensagem) {
            echo "[LOG] " . date("Y-m-d H:i:s") . " - $mensagem<br>";
        }
    }
    
    // Outro trait
    trait Timestampable {
        private $createdAt;
        private $updatedAt;
        
        public function setCreatedAt() {
            $this->createdAt = time();
        }
        
        public function setUpdatedAt() {
            $this->updatedAt = time();
        }
        
        public function getCreatedAt() {
            return date("Y-m-d H:i:s", $this->createdAt);
        }
        
        public function getUpdatedAt() {
            return date("Y-m-d H:i:s", $this->updatedAt);
        }
    }
    
    // Usando traits em uma classe
    class Produto {
        use Logger, Timestampable;
        
        private $nome;
        private $preco;
        
        public function __construct($nome, $preco) {
            $this->nome = $nome;
            $this->preco = $preco;
            $this->setCreatedAt();
            $this->log("Produto {$this->nome} criado");
        }
        
        public function atualizarPreco($novoPreco) {
            $this->preco = $novoPreco;
            $this->setUpdatedAt();
            $this->log("Preço do produto {$this->nome} atualizado para {$this->preco}");
        }
    }
    
    // Usando a classe
    $produto = new Produto("Smartphone", 1500);
    echo "Criado em: " . $produto->getCreatedAt() . "<br>";
    
    $produto->atualizarPreco(1600);
    echo "Atualizado em: " . $produto->getUpdatedAt() . "<br>";
?>
```
## 1.9 Introdução aos Métodos HTTP em PHP

No PHP, os métodos GET e POST são duas formas principais de enviar dados de um cliente (navegador) para o servidor:

### Método GET

O método GET envia dados através da URL, tornando-os visíveis na barra de endereços. É ideal para:
- Solicitações de busca
- Navegação entre páginas
- Dados não sensíveis

### Método POST

O método POST envia dados no corpo da requisição HTTP, não os exibindo na URL. É recomendado para:
- Envio de formulários com dados sensíveis
- Upload de arquivos
- Grandes volumes de dados

## Acesso aos Dados em PHP

### Acessando dados GET
```php
// Dados enviados via URL: pagina.php?nome=Maria&idade=25
$nome = $_GET['nome']; // Maria
$idade = $_GET['idade']; // 25
```

### Acessando dados POST
```php
// Dados enviados via formulário com method="post"
$email = $_POST['email'];
$senha = $_POST['senha'];
```

## Validações Simples com Estruturas Condicionais

Vamos implementar algumas validações básicas usando estruturas condicionais:

### Exemplo 1: Formulário de Login (POST)

**login.html**
```html
<!DOCTYPE html>
<html>
<head>
    <title>Login</title>
    <meta charset="utf-8">
</head>
<body>
    <h2>Login</h2>
    
    <form action="processa_login.php" method="post">
        <div>
            <label for="email">Email:</label>
            <input type="email" id="email" name="email" required>
        </div>
        <div>
            <label for="senha">Senha:</label>
            <input type="password" id="senha" name="senha" required>
        </div>
        <div>
            <button type="submit">Entrar</button>
        </div>
    </form>
</body>
</html>
```

**processa_login.php**
```php
<?php
// Verifica se o formulário foi enviado via POST
if ($_SERVER["REQUEST_METHOD"] == "POST") {
    // Captura os dados enviados
    $email = $_POST["email"];
    $senha = $_POST["senha"];
    
    // Validações simples
    $erros = [];
    
    // Validar se os campos foram preenchidos
    if (empty($email)) {
        $erros[] = "O campo email é obrigatório";
    }
    
    if (empty($senha)) {
        $erros[] = "O campo senha é obrigatório";
    }
    
    // Validar formato de email
    if (!empty($email) && !filter_var($email, FILTER_VALIDATE_EMAIL)) {
        $erros[] = "O formato do email é inválido";
    }
    
    // Validar comprimento da senha
    if (!empty($senha) && strlen($senha) < 6) {
        $erros[] = "A senha deve ter pelo menos 6 caracteres";
    }
    
    // Verificar se há erros
    if (count($erros) > 0) {
        // Exibir erros
        echo "<h3>Erros encontrados:</h3>";
        echo "<ul>";
        foreach ($erros as $erro) {
            echo "<li>$erro</li>";
        }
        echo "</ul>";
        echo "<p><a href='login.html'>Voltar</a></p>";
    } else {
        // Dados válidos - em um cenário real, aqui verificaria no banco de dados
        // Simulação de credenciais válidas
        $email_valido = "usuario@exemplo.com";
        $senha_valida = "123456";
        
        if ($email == $email_valido && $senha == $senha_valida) {
            echo "<h3>Login realizado com sucesso!</h3>";
            echo "<p>Bem-vindo, usuário!</p>";
        } else {
            echo "<h3>Falha no login</h3>";
            echo "<p>Email ou senha incorretos.</p>";
            echo "<p><a href='login.html'>Tentar novamente</a></p>";
        }
    }
} else {
    // Se tentar acessar diretamente o arquivo sem enviar o formulário
    echo "<p>Acesso negado. Por favor, utilize o formulário de login.</p>";
    echo "<p><a href='login.html'>Ir para o login</a></p>";
}
?>
```

### Exemplo 2: Página de Busca (GET)

**busca.php**
```php
<!DOCTYPE html>
<html>
<head>
    <title>Busca de Produtos</title>
    <meta charset="utf-8">
</head>
<body>
    <h2>Busca de Produtos</h2>
    
    <form action="busca.php" method="get">
        <div>
            <label for="termo">Termo de busca:</label>
            <input type="text" id="termo" name="termo" 
                   value="<?php echo isset($_GET['termo']) ? htmlspecialchars($_GET['termo']) : ''; ?>">
        </div>
        <div>
            <label for="categoria">Categoria:</label>
            <select id="categoria" name="categoria">
                <option value="">Todas</option>
                <option value="eletronicos" <?php echo (isset($_GET['categoria']) && $_GET['categoria'] == 'eletronicos') ? 'selected' : ''; ?>>Eletrônicos</option>
                <option value="roupas" <?php echo (isset($_GET['categoria']) && $_GET['categoria'] == 'roupas') ? 'selected' : ''; ?>>Roupas</option>
                <option value="alimentos" <?php echo (isset($_GET['categoria']) && $_GET['categoria'] == 'alimentos') ? 'selected' : ''; ?>>Alimentos</option>
            </select>
        </div>
        <div>
            <button type="submit">Buscar</button>
        </div>
    </form>
    
    <?php
    // Verifica se a busca foi realizada
    if (isset($_GET['termo']) || isset($_GET['categoria'])) {
        
        $termo = isset($_GET['termo']) ? $_GET['termo'] : '';
        $categoria = isset($_GET['categoria']) ? $_GET['categoria'] : '';
        
        // Validações
        $mensagens = [];
        $resultados = [];
        
        // Validar o termo de busca (opcional)
        if (!empty($termo) && strlen($termo) < 3) {
            $mensagens[] = "O termo de busca deve ter pelo menos 3 caracteres";
        }
        
        // Simulação de produtos
        $produtos = [
            ['nome' => 'Smartphone', 'categoria' => 'eletronicos', 'preco' => 1500],
            ['nome' => 'Notebook', 'categoria' => 'eletronicos', 'preco' => 3200],
            ['nome' => 'Camiseta', 'categoria' => 'roupas', 'preco' => 50],
            ['nome' => 'Calça Jeans', 'categoria' => 'roupas', 'preco' => 120],
            ['nome' => 'Chocolate', 'categoria' => 'alimentos', 'preco' => 8],
            ['nome' => 'Arroz', 'categoria' => 'alimentos', 'preco' => 20]
        ];
        
        // Filtrar produtos com base nos critérios
        if (count($mensagens) == 0) {
            foreach ($produtos as $produto) {
                $match_termo = empty($termo) || stripos($produto['nome'], $termo) !== false;
                $match_categoria = empty($categoria) || $produto['categoria'] == $categoria;
                
                if ($match_termo && $match_categoria) {
                    $resultados[] = $produto;
                }
            }
        }
        
        // Exibir mensagens
        if (count($mensagens) > 0) {
            echo "<div style='color: red; margin-top: 20px;'>";
            foreach ($mensagens as $mensagem) {
                echo "<p>$mensagem</p>";
            }
            echo "</div>";
        }
        
        // Exibir resultados
        echo "<h3>Resultados da busca:</h3>";
        
        if (count($resultados) > 0) {
            echo "<table border='1'>";
            echo "<tr><th>Nome</th><th>Categoria</th><th>Preço</th></tr>";
            
            foreach ($resultados as $produto) {
                echo "<tr>";
                echo "<td>" . htmlspecialchars($produto['nome']) . "</td>";
                echo "<td>" . htmlspecialchars($produto['categoria']) . "</td>";
                echo "<td>R$ " . number_format($produto['preco'], 2, ',', '.') . "</td>";
                echo "</tr>";
            }
            
            echo "</table>";
        } else {
            echo "<p>Nenhum produto encontrado.</p>";
        }
    }
    ?>
</body>
</html>
```

### Exemplo 3: Formulário de Cadastro (POST com múltiplas validações)

**cadastro.html**
```html
<!DOCTYPE html>
<html>
<head>
    <title>Cadastro de Usuário</title>
    <meta charset="utf-8">
</head>
<body>
    <h2>Cadastro de Usuário</h2>
    
    <form action="processa_cadastro.php" method="post">
        <div>
            <label for="nome">Nome Completo:</label>
            <input type="text" id="nome" name="nome" required>
        </div>
        <div>
            <label for="email">Email:</label>
            <input type="email" id="email" name="email" required>
        </div>
        <div>
            <label for="senha">Senha:</label>
            <input type="password" id="senha" name="senha" required>
        </div>
        <div>
            <label for="confirma_senha">Confirmar Senha:</label>
            <input type="password" id="confirma_senha" name="confirma_senha" required>
        </div>
        <div>
            <label for="idade">Idade:</label>
            <input type="number" id="idade" name="idade" min="18" required>
        </div>
        <div>
            <label>Gênero:</label>
            <input type="radio" id="masculino" name="genero" value="M" required>
            <label for="masculino">Masculino</label>
            <input type="radio" id="feminino" name="genero" value="F">
            <label for="feminino">Feminino</label>
            <input type="radio" id="outro" name="genero" value="O">
            <label for="outro">Outro</label>
        </div>
        <div>
            <label for="interesses">Interesses:</label><br>
            <input type="checkbox" id="tecnologia" name="interesses[]" value="tecnologia">
            <label for="tecnologia">Tecnologia</label><br>
            <input type="checkbox" id="esportes" name="interesses[]" value="esportes">
            <label for="esportes">Esportes</label><br>
            <input type="checkbox" id="musica" name="interesses[]" value="musica">
            <label for="musica">Música</label><br>
            <input type="checkbox" id="viagens" name="interesses[]" value="viagens">
            <label for="viagens">Viagens</label>
        </div>
        <div>
            <button type="submit">Cadastrar</button>
        </div>
    </form>
</body>
</html>
```

**processa_cadastro.php**
```php
<?php
// Verifica se o formulário foi enviado via POST
if ($_SERVER["REQUEST_METHOD"] == "POST") {
    // Captura os dados enviados
    $nome = isset($_POST["nome"]) ? trim($_POST["nome"]) : "";
    $email = isset($_POST["email"]) ? trim($_POST["email"]) : "";
    $senha = isset($_POST["senha"]) ? $_POST["senha"] : "";
    $confirma_senha = isset($_POST["confirma_senha"]) ? $_POST["confirma_senha"] : "";
    $idade = isset($_POST["idade"]) ? (int)$_POST["idade"] : 0;
    $genero = isset($_POST["genero"]) ? $_POST["genero"] : "";
    $interesses = isset($_POST["interesses"]) ? $_POST["interesses"] : [];
    
    // Array para armazenar erros
    $erros = [];
    
    // Validação do nome
    if (empty($nome)) {
        $erros[] = "O nome é obrigatório";
    } elseif (strlen($nome) < 5) {
        $erros[] = "O nome deve ter pelo menos 5 caracteres";
    }
    
    // Validação do email
    if (empty($email)) {
        $erros[] = "O email é obrigatório";
    } elseif (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
        $erros[] = "O formato do email é inválido";
    }
    
    // Validação da senha
    if (empty($senha)) {
        $erros[] = "A senha é obrigatória";
    } elseif (strlen($senha) < 8) {
        $erros[] = "A senha deve ter pelo menos 8 caracteres";
    } elseif (!preg_match('/[A-Z]/', $senha)) {
        $erros[] = "A senha deve conter pelo menos uma letra maiúscula";
    } elseif (!preg_match('/[0-9]/', $senha)) {
        $erros[] = "A senha deve conter pelo menos um número";
    }
    
    // Validação da confirmação de senha
    if ($senha != $confirma_senha) {
        $erros[] = "As senhas não coincidem";
    }
    
    // Validação da idade
    if (empty($idade)) {
        $erros[] = "A idade é obrigatória";
    } elseif ($idade < 18) {
        $erros[] = "Você deve ter pelo menos 18 anos para se cadastrar";
    } elseif ($idade > 120) {
        $erros[] = "Idade inválida";
    }
    
    // Validação do gênero
    if (empty($genero)) {
        $erros[] = "Selecione um gênero";
    } elseif (!in_array($genero, ['M', 'F', 'O'])) {
        $erros[] = "Opção de gênero inválida";
    }
    
    // Validação dos interesses (opcional)
    if (count($interesses) < 1) {
        $erros[] = "Selecione pelo menos um interesse";
    }
    
    // Verificar se há erros
    if (count($erros) > 0) {
        // Exibir erros
        echo "<h3>Erros encontrados:</h3>";
        echo "<ul>";
        foreach ($erros as $erro) {
            echo "<li>$erro</li>";
        }
        echo "</ul>";
        echo "<p><a href='cadastro.html'>Voltar ao formulário</a></p>";
    } else {
        // Dados válidos - em um cenário real, salvaria no banco de dados
        echo "<h3>Cadastro realizado com sucesso!</h3>";
        echo "<h4>Dados cadastrados:</h4>";
        echo "<p><strong>Nome:</strong> " . htmlspecialchars($nome) . "</p>";
        echo "<p><strong>Email:</strong> " . htmlspecialchars($email) . "</p>";
        echo "<p><strong>Idade:</strong> $idade anos</p>";
        echo "<p><strong>Gênero:</strong> ";
        
        switch ($genero) {
            case 'M':
                echo "Masculino";
                break;
            case 'F':
                echo "Feminino";
                break;
            case 'O':
                echo "Outro";
                break;
        }
        
        echo "</p>";
        
        echo "<p><strong>Interesses:</strong> ";
        if (count($interesses) > 0) {
            echo "<ul>";
            foreach ($interesses as $interesse) {
                echo "<li>" . htmlspecialchars($interesse) . "</li>";
            }
            echo "</ul>";
        } else {
            echo "Nenhum interesse selecionado";
        }
        
        echo "<p><a href='cadastro.html'>Voltar</a></p>";
    }
} else {
    // Se tentar acessar diretamente o arquivo sem enviar o formulário
    echo "<p>Acesso negado. Por favor, utilize o formulário de cadastro.</p>";
    echo "<p><a href='cadastro.html'>Ir para o cadastro</a></p>";
}
?>
```

## Diferenças Principais e Boas Práticas

### Quando usar GET:
- Para operações de leitura (não altera dados no servidor)
- Quando os dados não são sensíveis
- Quando deseja permitir que a URL seja compartilhada ou favoritada
- Para parâmetros simples e curtos

### Quando usar POST:
- Para operações que modificam dados no servidor
- Para envio de dados sensíveis (senhas, informações pessoais)
- Para grande volume de dados
- Para upload de arquivos

### Boas Práticas de Validação:
1. **Sempre verifique se os campos foram enviados antes de acessá-los**:
   ```php
   $nome = isset($_POST['nome']) ? $_POST['nome'] : '';
   ```

2. **Sanitize os dados recebidos**:
   ```php
   $email = filter_var($_POST['email'], FILTER_SANITIZE_EMAIL);
   ```

3. **Use estruturas condicionais para diferentes tipos de validação**:
   ```php
   if (empty($campo)) {
       $erros[] = "Campo obrigatório";
   } elseif (strlen($campo) < 5) {
       $erros[] = "Campo muito curto";
   }
   ```

4. **Verifique o método da requisição**:
   ```php
   if ($_SERVER["REQUEST_METHOD"] == "POST") {
       // Processar formulário POST
   }
   ```

5. **Use htmlspecialchars() ao exibir dados do usuário para prevenir XSS**:
   ```php
   echo htmlspecialchars($nome);
   ```

## 2. Sanitização e Validação de Dados

A sanitização e validação de dados são essenciais para garantir a segurança da aplicação e prevenir vulnerabilidades como injeção de SQL, XSS (Cross-Site Scripting) e outros ataques.

### 2.1 Funções de Sanitização

```php
<?php
    // Sanitizando entradas de texto
    $nome = $_POST['nome'] ?? '';
    $nome_limpo = filter_var($nome, FILTER_SANITIZE_STRING); // Deprecated no PHP 8.1+
    
    // Alternativa moderna
    $nome_limpo = htmlspecialchars($nome, ENT_QUOTES, 'UTF-8');
    
    // Sanitizando e-mail
    $email = $_POST['email'] ?? '';
    $email_limpo = filter_var($email, FILTER_SANITIZE_EMAIL);
    
    // Sanitizando URL
    $url = $_POST['url'] ?? '';
    $url_limpo = filter_var($url, FILTER_SANITIZE_URL);
    
    // Sanitizando números
    $idade = $_POST['idade'] ?? '';
    $idade_limpo = filter_var($idade, FILTER_SANITIZE_NUMBER_INT);
    
    // Sanitizando valores decimais
    $preco = $_POST['preco'] ?? '';
    $preco_limpo = filter_var($preco, FILTER_SANITIZE_NUMBER_FLOAT, FILTER_FLAG_ALLOW_FRACTION);
?>
```

### 2.2 Funções de Validação

```php
<?php
    // Validando e-mail
    $email = "usuario@exemplo.com";
    if (filter_var($email, FILTER_VALIDATE_EMAIL)) {
        echo "E-mail válido!";
    } else {
        echo "E-mail inválido!";
    }
    
    // Validando número inteiro
    $idade = "25";
    if (filter_var($idade, FILTER_VALIDATE_INT)) {
        echo "Idade válida!";
    } else {
        echo "Idade inválida!";
    }
    
    // Validando URL
    $url = "https://www.exemplo.com";
    if (filter_var($url, FILTER_VALIDATE_URL)) {
        echo "URL válida!";
    } else {
        echo "URL inválida!";
    }
    
    // Validando IP
    $ip = "192.168.1.1";
    if (filter_var($ip, FILTER_VALIDATE_IP)) {
        echo "IP válido!";
    } else {
        echo "IP inválido!";
    }
    
    // Validando com faixa
    $idade = 25;
    $min = 18;
    $max = 100;
    
    if (filter_var($idade, FILTER_VALIDATE_INT, ["options" => ["min_range" => $min, "max_range" => $max]])) {
        echo "Idade válida entre $min e $max!";
    } else {
        echo "Idade fora da faixa permitida!";
    }
?>
```

### 2.3 Exemplo Prático: Formulário com Validação

```php
<?php
    // Inicializa variáveis de erro
    $erros = [];
    $nome = $email = $comentario = '';
    
    // Verifica se o formulário foi enviado
    if ($_SERVER["REQUEST_METHOD"] == "POST") {
        // Validação e sanitização de nome
        if (empty($_POST["nome"])) {
            $erros['nome'] = "Nome é obrigatório";
        } else {
            $nome = htmlspecialchars($_POST["nome"], ENT_QUOTES, 'UTF-8');
            // Verifica se contém apenas letras e espaços
            if (!preg_match("/^[a-zA-Z ]*$/", $nome)) {
                $erros['nome'] = "Apenas letras e espaços são permitidos";
            }
        }
        
        // Validação e sanitização de e-mail
        if (empty($_POST["email"])) {
            $erros['email'] = "E-mail é obrigatório";
        } else {
            $email = filter_var($_POST["email"], FILTER_SANITIZE_EMAIL);
            if (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
                $erros['email'] = "Formato de e-mail inválido";
            }
        }
        
        // Validação e sanitização de comentário
        if (empty($_POST["comentario"])) {
            $erros['comentario'] = "Comentário é obrigatório";
        } else {
            $comentario = htmlspecialchars($_POST["comentario"], ENT_QUOTES, 'UTF-8');
        }
        
        // Se não houver erros, processa os dados
        if (empty($erros)) {
            // Processamento bem-sucedido
            echo "Dados recebidos com sucesso!";
            // Aqui você pode salvar os dados no banco, enviar e-mail, etc.
        }
    }
?>

<!DOCTYPE html>
<html>
<head>
    <title>Formulário com Validação</title>
    <style>
        .erro { color: red; }
    </style>
</head>
<body>
    <h2>Formulário de Contato</h2>
    <form method="post" action="<?php echo htmlspecialchars($_SERVER["PHP_SELF"]); ?>">
        <div>
            <label for="nome">Nome:</label>
            <input type="text" name="nome" id="nome" value="<?php echo $nome; ?>">
            <span class="erro"><?php echo $erros['nome'] ?? ''; ?></span>
        </div>
        
        <div>
            <label for="email">E-mail:</label>
            <input type="text" name="email" id="email" value="<?php echo $email; ?>">
            <span class="erro"><?php echo $erros['email'] ?? ''; ?></span>
        </div>
        
        <div>
            <label for="comentario">Comentário:</label>
            <textarea name="comentario" id="comentario"><?php echo $comentario; ?></textarea>
            <span class="erro"><?php echo $erros['comentario'] ?? ''; ?></span>
        </div>
        
        <div>
            <input type="submit" name="enviar" value="Enviar">
        </div>
    </form>
</body>
</html>
```
