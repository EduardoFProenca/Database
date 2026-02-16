

# Guia Completo de Estudo: Banco de Dados e SQL

Este documento foi criado para servir como material de consulta e revisão para conceitos fundamentais de Banco de Dados, SQL e Modelagem.

---

## 1. Estruturas de Armazenamento (Objetos Físicos)

### 1.1. Tabela (Table)
**O que é?**
A estrutura fundamental de um banco de dados relacional. É onde os dados vivem fisicamente. Pense nela como uma planilha de Excel, mas com regras rígidas.
*   **Física:** Ocupa espaço no disco.
*   **Linhas (Registros):** Representam as entidades (ex: um cliente, um produto).
*   **Colunas (Atributos):** Representam as características (ex: nome, preço).

**Conceitos Chave:**
*   **Chave Primária (PRIMARY KEY):** Um identificador único (como um CPF). Não pode se repetir e não pode ser nulo. Garante que você não tenha dois produtos com o mesmo ID.
*   **NOT NULL:** Garante que aquela coluna é obrigatória. Você não pode cadastrar um produto sem nome, por exemplo.

**Sintaxe:**
```sql
CREATE TABLE nome_tabela (
    coluna1 TIPO PRIMARY KEY,
    coluna2 TIPO NOT NULL
);
```

**Exemplo Prático:**
```sql
CREATE TABLE PRODUTOS (
    ID_PRODUTO NUMBER(10) PRIMARY KEY,  -- Identificador único
    NOME_PRODUTO VARCHAR2(100) NOT NULL, -- Nome obrigatório (máx 100 chars)
    PRECO NUMBER(10, 2)                  -- Preço decimal (ex: 10,50)
);
```

---

### 1.2. Índice (Index)
**O que é?**
Uma estrutura de dados auxiliar criada para **acelerar as consultas**.
*   **Analogia:** É o índice de um livro. Se você quer encontrar sobre "Bananas", você não lê o livro todo (Scan da tabela), você vai no índice, procura a página e vai direto lá.
*   **Quando usar:** Em colunas usadas frequentemente em `WHERE` ou `JOIN`.
*   **Custo:** Deixa a leitura (`SELECT`) rápida, mas deixa a escrita (`INSERT/UPDATE`) um pouco mais lenta, pois o banco precisa atualizar o índice também.

**Sintaxe:**
```sql
CREATE INDEX nome_idx ON tabela (coluna1, coluna2);
```

**Exemplo Prático:**
```sql
-- Cria um índice para buscas rápidas combinando conta e série
CREATE INDEX idx_idcadcli_ctb_codcta1_triser 
ON idcadcli_ctb (codcta1, triser);
```

---

### 1.3. View Materializada (Materialized View)
**O que é?**
Diferente da View comum (que é apenas uma consulta salva), a Materialized View **armazena fisicamente o resultado** da consulta.
*   **Para que serve:** Performance. Se você tem um relatório que demora 1 minuto para rodar (porque faz soma de milhões de linhas), você cria uma Materialized View. O usuário consulta a MV que já tem o resultado pré-calculado, levando milissegundos.
*   **Refresh:** Como os dados originais mudam, a MV precisa ser atualizada.
    *   *COMPLETE:* Apaga tudo e recalcula (mais lento, mas garante dados).
    *   *FAST:* Atualiza só o que mudou (mais rápido, mas exige configuração de logs).
    *   *ON DEMAND:* Só atualiza quando você mandar.
    *   *ON COMMIT:* Atualiza automaticamente quando alguém insere dados na tabela original.

**Sintaxe:**
```sql
CREATE MATERIALIZED VIEW nome_mv 
REFRESH [FAST|COMPLETE] ON [COMMIT|DEMAND] 
AS SELECT ...;
```

**Exemplo Prático:**
```sql
-- Cria um "resumo" físico do gasto por cliente. Atualiza sob demanda.
CREATE MATERIALIZED VIEW mv_total_gasto_cliente 
BUILD IMMEDIATE 
REFRESH COMPLETE ON DEMAND 
AS 
SELECT id_cliente, SUM(valor_total) 
FROM pedidos 
GROUP BY id_cliente;
```

---

## 2. Lógica de Programação no Banco

### 2.1. Procedimento Armazenado (Stored Procedure)
**O que é?**
É um programa (script) que fica salvo dentro do banco de dados, escrito em PL/SQL (Oracle).
*   **Vantagem:** Em vez de o seu aplicativo enviar 10 comandos SQL separados, ele envia apenas 1 comando chamando a Procedure. Isso reduz o tráfego de rede.
*   **Encapsulamento:** As regras de negócio (ex: "ao vender, diminuir estoque") ficam protegidas dentro do banco, não no código da aplicação.

**Sintaxe:**
```sql
CREATE OR REPLACE PROCEDURE nome (p_param IN TIPO) IS
BEGIN
    -- Comandos SQL e lógica aqui
END;
```

**Exemplo Prático:**
```sql
-- Procedure que recebe dados e insere na tabela
CREATE OR REPLACE PROCEDURE p_ins_produto (
    p_id IN PRODUTOS.ID_PRODUTO%TYPE, 
    p_nome IN VARCHAR2, 
    p_preco IN NUMBER
) IS 
BEGIN 
    INSERT INTO PRODUTOS VALUES (p_id, p_nome, p_preco); 
    COMMIT; -- Salva a transação
END;
```

---

## 3. Virtualização e Segurança

### 3.1. View (Visão)
**O que é?**
Uma "tabela virtual". Ela não armazena dados; ela armazena uma **query SQL**.
*   **Segurança:** Você tem uma tabela de funcionários com salários. Pode criar uma View que mostra apenas Nome e Departamento, escondendo o Salário.
*   **Simplicidade:** Esconde joins complexos. O usuário faz `SELECT * FROM view_funcionarios` sem precisar saber como as tabelas estão ligadas.

**Sintaxe:**
```sql
CREATE [OR REPLACE] VIEW nome_da_view AS 
SELECT ... 
[WITH READ ONLY];
```

**Exemplo Prático:**
```sql
-- Cria uma visão que junta Empregados e Departamentos
CREATE VIEW vw_empregados_departamento AS 
SELECT e.nome, d.nome_depto 
FROM empregados e 
JOIN departamentos d ON e.cod_depto = d.cod_depto;
```

---

## 4. Modelagem de Dados (Conceitual)

### 4.1. Generalização e Especialização
**O que é?**
Uma técnica de modelagem (DER) para evitar repetição de dados e organizar hierarquias. É o conceito de "Herança" (Pai e Filho).
*   **Entidade Pai (Superclasse):** Contém os atributos comuns a todos.
*   **Entidades Filho (Subclasses):** Contêm atributos específicos.

**Analogia:**
*   **Pai:** "Pessoa" (tem Nome, CPF, Endereço).
*   **Filho 1:** "Cliente" (Pessoa + Data de Cadastro, Limite de Crédito).
*   **Filho 2:** "Funcionário" (Pessoa + Salário, Cargo).
*   *Benefício:* Você não cadastra Nome e CPF duas vezes (uma vez pra cliente, outra pra funcionário).

**Exemplo Prático (Livros):**
*   **Livro (Pai):** Código, Título, Autor.
*   **Livro Didático (Filho):** Herda Título/Autor + adicionar **Disciplina** e **Série**.
*   **Livro Não Didático (Filho):** Herda Título/Autor + adicionar **Tema**.

---

## 5. Operações de Consulta (JOINS)

Os JOINS servem para ligar duas tabelas baseadas em uma coluna em comum.

### 5.1. INNER JOIN (Junção Interna)
**O que faz?**
Traz **apenas** os dados que existem nas **duas** tabelas ao mesmo tempo.
*   **Lógica:** Intersecção. Se tem cliente na tabela A mas nenhum pedido na tabela B, esse cliente **não** aparece.

**Sintaxe:**
```sql
SELECT ... 
FROM T1 
INNER JOIN T2 ON T1.coluna = T2.coluna;
```

**Exemplo:**
```sql
-- Lista apenas clientes que já fizeram pedidos
SELECT c.nome_cliente, p.id_pedido 
FROM clientes c 
INNER JOIN pedidos p ON c.id_cliente = p.id_cliente;
```

### 5.2. LEFT JOIN (Junção Esquerda)
**O que faz?**
Traz **todos** os registros da tabela da **esquerda** (a primeira mencionada), e os correspondentes da direita.
*   Se não houver correspondência, ele retorna **NULL** (vazio) para as colunas da tabela da direita.
*   **Uso típico:** Listar todos os clientes, incluindo os que *nunca* compraram nada.

**Sintaxe:**
```sql
SELECT ... 
FROM T1 
LEFT JOIN T2 ON T1.coluna = T2.coluna;
```

**Exemplo:**
```sql
-- Lista todas as Unidades Habitacionais e quem é o dono (se houver)
SELECT UH.codUH, Prop.NomePessoa 
FROM UnidadeHabitacional UH 
LEFT JOIN Pessoa Prop ON UH.CodPessoaProprietario = Prop.CodPessoa;
```

### 5.3. CROSS JOIN (Produto Cartesiano)
**O que faz?**
Combina **cada** linha da primeira tabela com **todas** as linhas da segunda tabela.
*   **Cuidado:** Se a tabela A tem 100 linhas e a B tem 100 linhas, o resultado terá 10.000 linhas (100 x 100).
*   **Uso típico:** Gerar combinações possíveis (ex: todas as cores disponíveis para todos os tamanhos de camisa).

**Sintaxe:**
```sql
SELECT ... 
FROM T1 
CROSS JOIN T2;
```

**Exemplo:**
```sql
-- Gera uma lista de todas as combinações de Cor x Tamanho
SELECT c.nome_cor, t.nome_tamanho 
FROM cores c 
CROSS JOIN tamanhos t;
```

---

## Fontes de Referência
1. Desvendando o Mundo dos Bancos de Dados.pdf
2. Script Inicial_Avancado2.txt
3. [APO-LB]06 - Views.docx
4. [TEO-BD]Exercicio4_GeneralizacaoEsp.docx
5. JOIN no Oracle.docx
6. [APO-TE]O4- Generalização e Especialização.docx
7. Script Criacao Padrao.txt
