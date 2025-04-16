# 20 Exercícios de PHP - Conceitos Mistos

## Exercício 1
Crie um formulário HTML com método POST que solicite ao usuário seu nome e idade. Em seguida, crie um projeto que receba esses dados e exiba uma mensagem personalizada baseada na idade:
- Se for menor de 18 anos: "[Nome], você é menor de idade."
- Se tiver entre 18 e 65 anos: "[Nome], você é adulto."
- Se for maior de 65 anos: "[Nome], você é idoso."

## Exercício 2
Desenvolva um projeto que receba um número via GET e exiba a tabuada desse número de 1 a 10 utilizando um laço de repetição `for`.

## Exercício 3
Crie um array associativo com 5 produtos contendo nome e preço. Utilize um laço `foreach` para exibir os produtos em uma tabela HTML. Se o preço for maior que R$ 50,00, a linha da tabela deve ter um background vermelho.

## Exercício 4
Desenvolva um formulário que permita ao usuário enviar 5 números via POST. No projeto, armazene esses números em um array, calcule a média e exiba quais números estão acima da média.

## Exercício 5
Construa um formulário onde o usuário informará via método POST o valor total de uma compra e também selecionará a forma de pagamento (Dinheiro, Cartão ou Pix). Com base na escolha do método de pagamento, utilize estruturas condicionais para aplicar diferentes percentuais de desconto e exibir o valor final após desconto claramente ao usuário.

## Exercício 6
Desenvolva um formulário que receba o nome de um mês via GET. Crie um array associativo contendo todos os meses do ano e suas respectivas quantidades de dias. Exiba quantos dias tem o mês informado pelo usuário. Se o mês não existir, exiba uma mensagem de erro.

## Exercício 7
Crie um array com 10 nomes de cidades. Desenvolva um projeto que receba um termo de busca via POST e retorne todas as cidades que contêm esse termo, utilizando estruturas de decisão e laços de repetição. Mostre também quantas cidades foram encontradas.

## Exercício 8
Crie um formulário que permita ao usuário adicionar itens a uma lista de compras (nome do item e quantidade). Use o método POST e armazene os itens em um array associativo. Permita que o usuário visualize, adicione e remova itens da lista utilizando sessões PHP.

## Exercício 9
Crie um array associativo com informações de 5 alunos (nome, nota1, nota2, nota3). Calcule a média de cada aluno e exiba uma tabela com todos os dados e a situação final: "Aprovado" (média >= 7), "Recuperação" (média entre 5 e 6.9) ou "Reprovado" (média < 5).

## Exercício 10


## Exercício 11


## Exercício 12
Desenvolva um validador de CPF usando PHP. O usuário deve inserir um CPF via POST, e seu código deve verificar se é válido seguindo o algoritmo oficial. Use arrays e laços de repetição para implementar a validação.

## Exercício 13
Crie um jogo de "Adivinhe o número" em PHP. O código deve gerar um número aleatório entre 1 e 100, e o usuário deve tentar adivinhar via formulário POST. A cada tentativa, o código deve informar se o número correto é maior ou menor, e contar quantas tentativas foram necessárias.

## Exercício 14
Desenvolva um código que receba uma data via GET (no formato dia/mês/ano) e calcule quantos dias faltam até o final do ano. Use estruturas de decisão para considerar anos bissextos e a quantidade correta de dias em cada mês.

## Exercício 15


## Exercício 16
Desenvolva um conversor de moedas em PHP. Crie um array associativo com 5 moedas e suas respectivas taxas de câmbio em relação ao Real. O usuário deve informar via POST um valor em Reais e a moeda desejada, e o código deve retornar o valor convertido.

## Exercício 17
Crie um código que receba uma frase via POST e conte:
- O número total de caracteres
- O número de palavras
- O número de ocorrências de cada letra do alfabeto (ignorando maiúsculas/minúsculas)
Use arrays associativos e laços de repetição para realizar as contagens.

## Exercício 18
Desenvolva um gerador de números para loteria. O usuário deve informar via GET qual loteria deseja (Mega-Sena: 6 números de 1 a 60, Quina: 5 números de 1 a 80, etc). Seu código deve gerar números aleatórios não repetidos de acordo com as regras da loteria escolhida, usando arrays e estruturas de decisão.

## Exercício 19
Crie um sistema de autenticação simples. Armazene em um array associativo 5 usuários com seus respectivos logins e senhas. Receba os dados de login via POST e verifique se correspondem a algum usuário cadastrado. Se sim, exiba uma mensagem de boas-vindas; se não, informe o erro (usuário inexistente ou senha incorreta).

## Exercício 20
Desenvolva um código que receba um texto via POST e implemente uma função de censura. O usuário também deve informar uma lista de palavras a serem censuradas. O código deve substituir todas as ocorrências dessas palavras por asteriscos (mantendo o mesmo número de caracteres) e exibir o texto censurado. Use arrays, laços de repetição e funções de manipulação de strings.
