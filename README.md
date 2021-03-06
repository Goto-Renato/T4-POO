#Super Mercado

## Execução
`java -jar T4.jar <porta>` Inicia o servidor na porta escolhida e salva os dados no diretório atual quando o servidor terminar.

Para comunicar com o servidor via texto pode-se usar `telnet`, `netcat` e equivalentes.

Interface gráfica?

## Compilação
Para compilar basta usar o comando `ant`.

## Design Patterns
O projeto segue o padrão de design MVC, onde as classes **Usuario** e **Produto** são *models* definidos pela interface **Registro**. A Classe **Servidor** é o *controller*, e a classe **Manipulador** é o *view*.  

## Protocolo
O protocolo usado para comunicação entre servidor e cliente é parecido com cabeçalhos HTTP, porém mais simples:
```
<COMANDO><\n>
[<VARIÁVEL>:{CONTEÚDO}<\n>]
<\n>
```
*A parte entre colchetes pode ser repetida 0 ou mais vezes.*

`<COMANDO>`: Comando que o cliente deseja executar. Deve ser composto apenas de letras maiúsculas ou sublinhados('_'). O primeiro caracter tem que ser uma letra.

`<VARIÁVEL>`: Nome da variável sendo definida. Pode ser composto por qualquer letra (maiúscula ou minúscula), dígitos e sublinhados ('_'). Não pode começar com um dígito.

`{CONTEÚDO}`: Pode ser qualquer sequencia de caractere ASCII válidos, exeto caracteres de controle. **Nota:** Espaços após os ':' são interpretados como parte da variável!.

`<\n>`: Representa uma quebra de linha.

**Nota:** Alguns comandos podem conter dados após o cabeçalho. Os dados e o formato podem variar de acordo com cada comando. 

### Comandos válidos:
#### Login
Quando o login é efetuado com sucesso o token identifica a sessão do usuário. Todas as ações que requerem login precisam de um token válido para serem efetuadas. Quando o usuário faz login novamente ou logout, o token é invalidado.

**Requisição:**
```
LOGIN
id:nome_de_exemplo
pass:s3nh4_s3cr374

```
**Respostas:**

Quando o faz login com sucesso:
```
OK
token:ExEmPlOeXeMpLoExEmPlO+==

```

Quando o login falha:
```
ERROR
msg:Usuário e/ou senha inválidos.

```

### Logout
Invalida o token do usuário.

**Requisição:**
```
LOGOUT
id:nome_de_exemplo
token:ExEmPlOeXeMpLoExEmPlO+==

```

**Resposta:**

Sucesso:
```
OK

```

Falhou:
```
ERRO
msg:Usuário e/ou token inválidos.

```

### Adicionar Usuário
Adiciona um novo usuário com os dados fornecidos.

**Requisição:**
```
ADD_USER
id:nome_de_exemplo
pass:s3nh4_s3cr374
name:Nome De Exemplo
addr1:Terra do Nunca - AC
addr2:Rua inexistente, NaN
email:user@example.com
phone:+551695551337

```

**Respostas:**

Quando adiciona com sucesso:
```
OK

```

Quando falha por colisão de ID:
```
ERROR
msg:ID já está cadastrado!

```

### Adicionar Produto
Adiciona um produto novo no estoque.

**Requisição:**

Note que o preço e a data devem ser números inteiros. A data é um unix timestamp e o preço é medido em centavos.
```
ADD_PRODUCT
name:sabre-de-luz
price:199995
exp_date:9999999999
supplier:the-empire
stock:42
id:nome_de_exemplo
token:ExEmPlOeXeMpLoExEmPlO+==

```

**Respostas:**

Quando adiciona com sucesso:
```
OK

```

Quando o produto já existe:
```
ERROR
msg:Produto já cadastrado!

```

### Estocar Produto
Aumenta a quantidade em estoque de um produto já existente.

**Requisição:**
```
STOCK_PRODUCT
name:sabre-de-luz
add:9001
id:nome_de_exemplo
token:ExEmPlOeXeMpLoExEmPlO+==

```

**Respostas:**

Quando estoca com sucesso:
```
STOCK
count:<Novo total>

```
`count` indica a quantiade total de produtos após a estocagem.

Quando o produto não existe:
```
ERROR
msg:Produto inexistente!

```

### Listar Produtos
Lista os produtos do super mercado. Há 3 filtros disponíveis: `ALL`, `AVAILABLE`, `UNAVAILABLE` que (respectivamente) listam todos os produtos4, apenas os disponíveis em estoque e apenas os em falta.

**Requisição:**
```
PRODUCT_LIST
filter:ALL
id:nome_de_exemplo
token:ExEmPlOeXeMpLoExEmPlO+==

```

**Resposta:**

Note que a contagem pode ser 0 caso não haja nenhum produto
```
PRODUCT_LIST
count:<Número de produtos>

<Prod 1>
<Prod 2>
...
```
Os produtos são representados no mesmo formato do arquivo CSV.

### Comprar Produto
Realiza a compra de um produto.

**Requisição:**
```
BUY
name:sabre-de-luz
id:nome_de_exemplo
token:ExEmPlOeXeMpLoExEmPlO+==

```

**Respostas:**

Quando efetua a compra com sucesso:
```
OK

```

Quando falha a mensagem de erro indica o motivo:

```
ERROR
msg:<Motivo>

```

## Comandos Especiais (sem respostas)


```
BYE

```
Fecha a conexão com o servidor.

```
STOP

```
Faz o servidor parar de aceitar novas conexões e fechar assim que o último cliente sair.
