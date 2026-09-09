# Checkpoint 4 — Bug Hunt StreamFIAP

## Identificação

**Grupo:** RM565010 (individual)

| Integrante | RM | Turma |
|---|---|---|
| Júlia Souza Marques | RM565010 | 2CCPW |

| Campo | |
|---|---|
| **Total de bugs corrigidos** | 4 / 12 |
| **Total de ajustes de Clean Code** | 0 / 6 |

---

## Parte 1 — Bugs encontrados

> Uma linha por bug, na ordem em que você os encontrou. Use a numeração dos seus
> commits (`fix: bug01 ...`). Preencha TODAS as colunas — metade da nota está aqui.

| # | Sintoma observado (o que fiz/vi) | Causa raiz (arquivo e linha aproximada) | Correção aplicada | Conceito da disciplina |
|---|---|---|---|---|
| bug01 | Testei o cálculo de promoção do `Filme` e o valor ficava maior em vez de menor | `model/Filme.java`, método `aplicarPromocao` — usava `preco * 1.2` | Troquei `preco * 1.2` por `preco * 0.8` | Interface como contrato (Aula 8/9): a interface `Promocionavel` documenta 20% de desconto, e a implementação aplicava um acréscimo, violando o contrato |
| bug02 | Percebi que o construtor de `Serie` não inicializava os campos herdados de `Conteudo` e nem recebia o parâmetro `disponivel` | `model/Serie.java`, construtor de `Serie` — não chamava `super(...)` e faltava o parâmetro `boolean disponivel` na assinatura | Adicionei `boolean disponivel` na assinatura do construtor e chamei `super(titulo, categoria, duracaoMinutos, classificacaoEtaria, disponivel)` no início | Herança e construtores (Aula 6/7): construtor de subclasse precisa chamar `super(...)` para inicializar os campos definidos na superclasse |
| bug03 | O preço de série sempre calculava como se fosse `9.90` fixo, ignorando o valor por temporada | `model/Serie.java`, método `calcularPrecoAluguel(double desconto)` — assinatura diferente da superclasse (`Conteudo.calcularPrecoAluguel()` sem parâmetro), então era overload e não override | Removi o parâmetro `desconto` e adicionei `@Override`, deixando a assinatura idêntica à da superclasse | Override vs Overload (Aula 7): mesmo nome com parâmetros diferentes cria um método novo em vez de substituir o da superclasse; `@Override` obriga o compilador a validar que a assinatura realmente bate |
| bug04 | O cadastro de série não compilava mais depois de corrigir o construtor (bug02) | `controller/ConteudoController.java`, método `cadastrarSerie` — chamava `new Serie(...)` com a assinatura antiga do construtor, sem passar `disponivel` | Adicionei `serie.isDisponivel()` na posição correta da chamada ao construtor | Bug em cascata entre camadas (Controller vs Model — Aula 13): corrigir o construtor no model expôs um bug que já existia no controller, mas que antes "coincidia" com a assinatura errada |
| bug05 | | | | |
| bug06 | | | | |
| bug07 | | | | |
| bug08 | | | | |
| bug09 | | | | |
| bug10 | | | | |
| bug11 | | | | |
| bug12 | | | | |

## Parte 2 — Ajustes de Clean Code

| # | Onde estava | Qual princípio/boas práticas era violado | O que eu mudei |
|---|---|---|---|
| clean01 | | | |
| clean02 | | | |
| clean03 | | | |
| clean04 | | | |
| clean05 | | | |
| clean06 | | | |

---

## Parte 3 — Perguntas de reflexão

> Responda com suas palavras, 5 a 10 linhas cada, **usando o código real do projeto
> como exemplo**. Respostas genéricas de tutorial não pontuam.

### 1. Injeção de dependência (Aula 13)
Os controllers recebem os repositories via `@Autowired` (ex.: `ConteudoController`
usa `ConteudoRepository`). Explique por que o Spring precisa gerenciar esses objetos
em vez de criarmos com `new ConteudoRepository()`. O que exatamente o Spring faz ao
injetar um bean, e por que isso não funcionaria com um `new` comum?

_(sua resposta aqui)_

### 2. JDBC vs Spring Data JPA (Aulas 12 e 13)
Na Aula 12 escrevemos um `ProdutoDAO` na mão com `Connection`, `PreparedStatement` e
`ResultSet`. Aqui o `ConteudoRepository` tem 2 linhas e faz CRUD completo. Compare as
duas abordagens: o que o Spring Data JPA automatiza, o que o JDBC/DAO ainda resolve
melhor, e como o `findByCategoria` consegue funcionar sem implementação.

_(sua resposta aqui)_

### 3. Exceções checked vs unchecked (Aula 11)
A `ClassificacaoIndicativaException` estourava como um erro genérico do servidor,
sem mensagem útil para o cliente. Explique a diferença entre `extends Exception` e
`extends RuntimeException` no contexto desse bug, e como você fez a mensagem da
regra (classificação indicativa) chegar de forma clara ao cliente da API.

_(sua resposta aqui)_

### 4. Sobrescrita vs sobrecarga (Aula 7)
Um dos bugs compilava sem nenhum erro: o método da `Serie` parecia sobrescrever
`calcularPrecoAluguel`, mas na verdade sobrecarregava. Explique a diferença entre
override e overload nesse caso e por que a anotação `@Override` teria impedido o bug.

_(sua resposta aqui)_

### 5. Onde blindar o objeto? (Aulas 3, 4 e 13)
Vimos bugs de dados inválidos aceitos (duração negativa, créditos negativos, campos
nulos). Em quais lugares (construtor, setter, método do model) cada tipo de validação
deve ficar? Justifique usando os bugs que você encontrou e explique por que validar só
em um lugar não foi suficiente.

_(sua resposta aqui)_

### 6. Abstração e interface (Aulas 8 e 9)
`Conteudo` é abstrata e `Promocionavel` é uma interface. Explique a diferença de
propósito entre as duas nesse projeto e o que mudaria no código se o Documentário
passasse a ter promoções — quais classes/linhas seriam tocadas e quais ficariam
intactas? O que isso diz sobre o design do sistema?

_(sua resposta aqui)_

---

## Parte 4 — Espaço livre (opcional)

Alguma dificuldade, dúvida ou comentário sobre o checkpoint?

```


```