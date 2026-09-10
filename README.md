# Checkpoint 4 — Bug Hunt StreamFIAP

## Identificação

**Grupo:** RM565010 (individual)

| Integrante | RM | Turma |
|---|---|---|
| Júlia Souza Marques | RM565010 | 2CCPW |

| Campo | |
|---|---|
| **Total de bugs corrigidos** | 12 / 12 |
| **Total de ajustes de Clean Code** | 6 / 6 |

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
| bug05 | Documentário sempre custava R$ 9,90 no cálculo de aluguel, mas o contrato exige gratuidade | `model/Documentario.java` — a classe não sobrescrevia `calcularPrecoAluguel()`, então herdava o valor fixo `9.90` de `Conteudo` | Adicionei `@Override public double calcularPrecoAluguel() { return 0.0; }` na classe `Documentario` | Herança e override (Aula 7): um bug por ausência de método é mais difícil de notar do que um método com erro, porque a classe parece "limpa" numa leitura rápida |
| bug06 | O nome do usuário nunca era salvo — vinha sempre nulo depois do cadastro | `model/Usuario.java`, construtor de `Usuario` — a linha `nome = nome` atribuía o parâmetro a ele mesmo, não ao campo da instância (faltava `this.`) | Troquei `nome = nome` por `this.nome = nome` | Shadowing de variável (Aula 3/6): quando o parâmetro do construtor tem o mesmo nome do atributo, é preciso usar `this.` para diferenciar o campo da instância do parâmetro local |
| bug07 | O sistema recusava aluguéis mesmo quando o usuário tinha créditos de sobra | `model/Usuario.java`, método `temCreditosSuficientes` — a condição era `preco >= this.creditos`, que retorna `true` (tem créditos) justamente quando o preço é maior que o saldo | Troquei `preco >= this.creditos` por `preco <= this.creditos` | Lógica booleana / operadores relacionais (Aula 3): a condição estava com o operador invertido, fazendo o método dizer o oposto do que seu próprio nome promete |
| bug08 | O método `alugar` nunca checava se o conteúdo estava disponível — dava pra alugar algo marcado como indisponível | `model/Usuario.java`, método `alugar` — faltava uma validação de `c.isDisponivel()` antes de prosseguir com o aluguel | Adicionei `if (!c.isDisponivel()) { throw new ConteudoIndisponivelException(...); }` como primeira checagem do método, antes da classificação etária | Fail Fast / regra de negócio no model (Aula 11): a regra "conteúdo indisponível não se aluga" existia no contrato mas não no código; validar cedo evita processar o resto do método com dado inválido |
| bug09 | Cadastro de usuário retornava `500 Internal Server Error` sem mensagem clara | `model/Usuario.java`, campo `id` — tinha só `@Id`, sem `@GeneratedValue(strategy = GenerationType.IDENTITY)`, diferente de `Conteudo`. O Hibernate tentava inserir `id = null`, e o Oracle recusava (`ORA-01400: não é possível inserir NULL`) | Adicionei `@GeneratedValue(strategy = GenerationType.IDENTITY)` acima de `@Id`. Também precisei apagar a tabela `usuarios` antiga no Oracle (via DBeaver) e deixar o Hibernate recriá-la, porque `ddl-auto=update` não consegue converter uma coluna existente para IDENTITY | Mapeamento JPA / geração de chave primária (Aula 13): sem `@GeneratedValue`, o JPA espera que o próprio código forneça o id, então salvar uma entidade nova sem id explícito quebra a inserção no banco |
| bug10 | Tentar alugar um conteúdo com idade insuficiente retornava `500 Internal Server Error` genérico, sem explicar o motivo | `exception/GlobalExceptionHandler.java` — tinha `@ExceptionHandler` para `ConteudoNaoEncontradoException`, `CreditosInsuficientesException` e `ConteudoIndisponivelException`, mas faltava um handler para `ClassificacaoIndicativaException` | Adicionei `@ExceptionHandler(ClassificacaoIndicativaException.class)` retornando `403 Forbidden` com a mensagem da exceção | Tratamento centralizado de exceções (Aula 11 + 13): a exceção existia e era lançada corretamente no model, mas sem um handler registrado o Spring devolve 500 genérico por padrão para qualquer exceção não mapeada |
| bug11 | Percebi que `AluguelController.alugar` e `Usuario.alugar` precisavam declarar `throws ClassificacaoIndicativaException` desnecessariamente | `exception/ClassificacaoIndicativaException.java` — estendia `Exception` (checked), diferente das outras exceções do projeto (`ConteudoIndisponivelException`, `ConteudoNaoEncontradoException`, `CreditosInsuficientesException`), que estendem `RuntimeException` | Troquei `extends Exception` por `extends RuntimeException` em `ClassificacaoIndicativaException` e removi o `throws ClassificacaoIndicativaException` das assinaturas de `AluguelController.alugar` e `Usuario.alugar` | Exceções checked vs unchecked (Aula 11): inconsistência no padrão de exceções do projeto forçava `throws` desnecessário |
| bug12 | Ao filtrar conteúdos por categoria, `listarPorCategoria` às vezes não retornava nada mesmo com categorias iguais no texto | `controller/ConteudoController.java`, método `listarPorCategoria` — usava `c.getCategoria() == categoria`, que compara referência de objeto, não o conteúdo do texto | Troquei por `categoria.equals(c.getCategoria())` (usei `categoria` como receptor do `.equals` em vez de `c.getCategoria()` porque o `@PathVariable` nunca é null, evitando NullPointerException caso `getCategoria()` retorne null) | Comparação de objetos em Java (Aula 3): `==` compara referência, `.equals()` compara valor/conteúdo |

## Parte 2 — Ajustes de Clean Code

| # | Onde estava | Qual princípio/boas práticas era violado | O que eu mudei |
|---|---|---|---|
| clean01 | `controller/ConteudoController.java`, método `buscarPorId` | Catch vazio nunca deve silenciar erros (Aula 11); método retornava `Conteudo` em vez de `ResponseEntity<Conteudo>` e engolia a exceção com `catch (Exception e) { // TODO: tratar isso depois }`, terminando com `return null` (sucesso vazio disfarçado de 200) | Reescrevi o método para retornar `ResponseEntity<Conteudo>`, removi o try/catch e deixei `ConteudoNaoEncontradoException` subir para o `GlobalExceptionHandler`, que já trata essa exceção retornando 404 |
| clean02 | `controller/ConteudoController.java`, final da classe | Código morto/comentado não deve permanecer no arquivo final — se não é usado, não deveria estar lá | Removi o método privado `calcularDescontoAntigo`, que nunca era chamado em lugar nenhum, e o bloco de código comentado com `TODO` sobre cupons |
| clean03 | `UsuarioController.java` e `AluguelController.java` | Consistência de padrão no tratamento de erros do projeto — exceções de domínio específicas em vez de exceções genéricas da linguagem; `IllegalArgumentException` não tinha handler no `GlobalExceptionHandler`, então caía em 500 genérico | Criei `exception/UsuarioNaoEncontradoException.java` (mesmo padrão das outras exceções do projeto), troquei `IllegalArgumentException` por ela em `UsuarioController.buscarPorId` e `AluguelController.alugar`, e adicionei um handler dedicado no `GlobalExceptionHandler` retornando 404 |
| clean04 | `model/Usuario.java`, método `alugar` | Nomes de variáveis devem ser descritivos, não abreviações de uma letra sem contexto | Renomeei a variável `p` para `precoAluguel` em todas as ocorrências dentro do método `alugar` (declaração, `temCreditosSuficientes`, `debitarCreditos` e o print do recibo) |
| clean05 | `model/Conteudo.java` e `controller/ConteudoController.java` | Encapsulamento (Aula 3/4) — campos de uma classe devem ser private, expostos apenas via getters/setters; `duracaoMinutos` era o único campo público da classe, permitindo alteração direta sem passar por validação | Troquei `public int duracaoMinutos` por `private int duracaoMinutos` em `Conteudo`; ajustei os acessos diretos `filme.duracaoMinutos`, `serie.duracaoMinutos` e `documentario.duracaoMinutos` em `ConteudoController` para usar `getDuracaoMinutos()` |
| clean06 | `model/Usuario.java`, método `debitarCreditos` | Comentários devem refletir o código corretamente — um comentário errado é pior do que nenhum comentário; o comentário dizia "adiciona", mas o código subtrai | Troquei o comentário para `// subtrai o valor dos créditos do usuário` |

---

## Parte 3 — Perguntas de reflexão

> Responda com suas palavras, 5 a 10 linhas cada, **usando o código real do projeto
> como exemplo**. Respostas genéricas de tutorial não pontuam.

### 1. Injeção de dependência (Aula 13)
Os controllers recebem os repositories via `@Autowired` (ex.: `ConteudoController`
usa `ConteudoRepository`). Explique por que o Spring precisa gerenciar esses objetos
em vez de criarmos com `new ConteudoRepository()`. O que exatamente o Spring faz ao
injetar um bean, e por que isso não funcionaria com um `new` comum?

ConteudoRepository é uma interface, não tem corpo de método — só a assinatura de
findByCategoria. Não dá nem pra fazer new ConteudoRepository(), porque não existe
implementação minha pra instanciar. Quem cria essa implementação em tempo de
execução (um proxy que conversa com o banco via JPA/Hibernate) é o Spring, quando
sobe o contexto da aplicação. O @Autowired em ConteudoController diz "me entrega
uma instância pronta, não vou criar isso na mão". O Spring resolve a dependência,
injeta o mesmo bean (por padrão um singleton) em todo lugar que precisar dele, e
cuida do ciclo de vida inteiro (conexão, transação, etc). Se eu tentasse um new
manual desse repository, teria que implementar sozinha toda a lógica de acesso a
dados que o Spring Data JPA já gera a partir da interface.

### 2. JDBC vs Spring Data JPA (Aulas 12 e 13)
Na Aula 12 escrevemos um `ProdutoDAO` na mão com `Connection`, `PreparedStatement` e
`ResultSet`. Aqui o `ConteudoRepository` tem 2 linhas e faz CRUD completo. Compare as
duas abordagens: o que o Spring Data JPA automatiza, o que o JDBC/DAO ainda resolve
melhor, e como o `findByCategoria` consegue funcionar sem implementação.

No ProdutoDAO eu escrevia a query SQL à mão, abria a Connection, montava o
PreparedStatement com os parâmetros, executava, percorria o ResultSet e convertia
cada linha em objeto — repetindo isso pra cada operação (inserir, buscar,
atualizar, deletar). O ConteudoRepository só declara extends
JpaRepository<Conteudo, Long> e já ganha save, findAll, findById, deleteById
prontos, sem uma linha de SQL. O findByCategoria(String categoria) funciona sem
implementação porque o Spring Data JPA usa query derivation: lê o nome do método,
reconhece o padrão findBy<NomeDoCampo> e monta a query JPQL/SQL sozinho na
inicialização. O JDBC/DAO ainda vale quando a query é muito específica, precisa de
ajuste fino de performance, ou usa recurso do banco que o JPA não expõe bem — o
Spring Data JPA automatiza o caso comum, mas quem quer controle total sobre a
query cai pro JDBC ou pra uma consulta nativa.

### 3. Exceções checked vs unchecked (Aula 11)
A `ClassificacaoIndicativaException` estourava como um erro genérico do servidor,
sem mensagem útil para o cliente. Explique a diferença entre `extends Exception` e
`extends RuntimeException` no contexto desse bug, e como você fez a mensagem da
regra (classificação indicativa) chegar de forma clara ao cliente da API.

Uma exceção checked (extends Exception) obriga quem chama o método a tratá-la com
try/catch ou repassar com throws — o compilador não deixa compilar sem isso. Uma
unchecked (extends RuntimeException) não tem essa exigência, sobe livre pela pilha
até alguém tratar. Nesse projeto, ClassificacaoIndicativaException era checked, o
que forçava Usuario.alugar e AluguelController.alugar a declararem throws
ClassificacaoIndicativaException — só que isso não resolvia o problema real, que
era não existir nenhum @ExceptionHandler pra ela no GlobalExceptionHandler. Por
isso o Spring devolvia 500 genérico mesmo com o throws lá. Corrigi em duas etapas:
primeiro adicionei o handler (@ExceptionHandler(ClassificacaoIndicativaException.class)
retornando 403 com a mensagem da exceção), depois, pra deixar consistente com as
outras exceções de domínio (ConteudoIndisponivelException, ConteudoNaoEncontradoException,
CreditosInsuficientesException), troquei extends Exception por extends
RuntimeException e removi o throws desnecessário das assinaturas. A mensagem da
regra de negócio agora chega ao cliente como JSON {"erro": "..."} com status HTTP
apropriado, em vez de um 500 sem explicação.

### 4. Sobrescrita vs sobrecarga (Aula 7)
Um dos bugs compilava sem nenhum erro: o método da `Serie` parecia sobrescrever
`calcularPrecoAluguel`, mas na verdade sobrecarregava. Explique a diferença entre
override e overload nesse caso e por que a anotação `@Override` teria impedido o bug.

Override é quando uma subclasse reimplementa um método da superclasse com a mesma
assinatura (nome + parâmetros), substituindo o comportamento herdado. Overload é
quando existem métodos com o mesmo nome mas parâmetros diferentes — são métodos
distintos, um não substitui o outro. No bug03, Conteudo.calcularPrecoAluguel() não
recebe parâmetro, mas Serie tinha calcularPrecoAluguel(double desconto). Assinatura
diferente = o Java não via ali um override, via um método novo (overload): a classe
Serie ficava com dois métodos calcularPrecoAluguel — o herdado (sem parâmetro,
retornando o valor fixo 9.90) e o novo (com parâmetro, nunca chamado por quem usava
Conteudo de forma polimórfica). Compilava sem erro porque overload é totalmente
válido em Java, só não fazia o que a intenção do código sugeria. Se eu tivesse
colocado @Override no método da Serie, o compilador teria acusado erro na hora,
porque não existe nenhum método na superclasse com aquela assinatura (com o
parâmetro desconto) pra sobrescrever — o @Override trava justamente isso: obriga o
compilador a confirmar que a assinatura bate com a da superclasse.

### 5. Onde blindar o objeto? (Aulas 3, 4 e 13)
Vimos bugs de dados inválidos aceitos (duração negativa, créditos negativos, campos
nulos). Em quais lugares (construtor, setter, método do model) cada tipo de validação
deve ficar? Justifique usando os bugs que você encontrou e explique por que validar só
em um lugar não foi suficiente.

Cada camada protege contra uma entrada diferente, não dá pra confiar em só um
ponto. O construtor valida o que é exigido pro objeto nascer num estado consistente
(Conteudo e Usuario recebem duracaoMinutos/creditos iniciais e poderiam recusar
valores negativos já na criação). O setter repete essa validação, porque o objeto
já existe e alguém pode chamar setDuracaoMinutos(-10) depois — é por isso que o
clean05 (tornar duracaoMinutos private em Conteudo) importa: com o campo público,
nem construtor nem setter conseguiam interceptar uma atribuição direta como
conteudo.duracaoMinutos = -10, então blindar só no construtor não adiantava nada.
Já a regra de negócio (diferente de validação de dado) fica no método do model que
a representa: temCreditosSuficientes e a checagem de isDisponivel()/classificação
etária dentro de Usuario.alugar (bug07 e bug08) são exemplos disso — é lógica de
domínio, não formato de dado, só faz sentido no momento da operação. Construtor e
setter blindam o estado do objeto isoladamente; o método de negócio blinda a
operação, dado o estado atual dos objetos envolvidos. Validar em um lugar só não
basta porque cada um cobre uma porta de entrada diferente pro dado incorreto.

### 6. Abstração e interface (Aulas 8 e 9)
`Conteudo` é abstrata e `Promocionavel` é uma interface. Explique a diferença de
propósito entre as duas nesse projeto e o que mudaria no código se o Documentário
passasse a ter promoções — quais classes/linhas seriam tocadas e quais ficariam
intactas? O que isso diz sobre o design do sistema?

Conteudo é uma classe abstrata porque representa um "é um": Filme, Serie e
Documentario são tipos de conteúdo por natureza, compartilham estado (titulo,
categoria, duracaoMinutos, etc) e um construtor protegido comum. Promocionavel é
uma interface porque representa uma capacidade opcional, um "pode fazer": nem todo
conteúdo tem promoção, então em vez de forçar isso em Conteudo, cada subclasse
decide se implementa aplicarPromocao(double preco) — hoje Filme e Serie fazem isso
(implements Promocionavel). O método calcularPrecoPromocional() em Conteudo usa
instanceof Promocionavel pra checar essa capacidade em tempo de execução sem
acoplar a superclasse a um tipo concreto. Se Documentario passasse a ter promoções,
eu só precisaria adicionar implements Promocionavel na declaração da classe e
implementar aplicarPromocao(double preco) nela. Nenhuma linha de Conteudo, Filme,
Serie ou do ConteudoController mudaria, porque calcularPrecoPromocional() já trata
qualquer Promocionavel de forma genérica. Isso é design aberto pra extensão e
fechado pra modificação (Open/Closed): a interface deixa adicionar comportamento
novo numa classe existente sem tocar no código que já funciona.

---

## Parte 4 — Espaço livre (opcional)

Alguma dificuldade, dúvida ou comentário sobre o checkpoint?

```


```