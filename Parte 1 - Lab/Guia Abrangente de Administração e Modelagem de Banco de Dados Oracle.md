# 📚 Guia Completo de Banco de Dados Oracle

> **Guia de estudos completo e simplificado para consulta rápida sobre objetos e operações em Oracle Database**

---

## 📑 Índice

- [O Que É Um Banco de Dados?](#o-que-é-um-banco-de-dados)
- [Objetos Físicos](#objetos-físicos)
  - [Tabela](#tabela)
  - [View Materializada](#view-materializada)
  - [Índice](#índice)
- [Objetos Lógicos](#objetos-lógicos)
  - [View (Visão)](#view-visão)
  - [Procedimento Armazenado](#procedimento-armazenado-procedure)
- [Operações de Junção (JOINs)](#operações-de-junção-joins)
  - [INNER JOIN](#inner-join)
  - [LEFT JOIN](#left-join)
  - [RIGHT JOIN](#right-join)
  - [FULL OUTER JOIN](#full-outer-join)
  - [CROSS JOIN](#cross-join)
- [Modelagem de Dados](#modelagem-de-dados)
  - [Generalização/Especialização](#generalizaçãoespecialização)
- [Comandos Básicos Essenciais](#comandos-básicos-essenciais)
- [Tipos de Dados Oracle](#tipos-de-dados-oracle)
- [Constraints (Restrições)](#constraints-restrições)
- [Referências](#referências)

---

## O Que É Um Banco de Dados?

Um **banco de dados** é um sistema organizado para armazenar, gerenciar e recuperar informações. O Oracle é um **SGBD** (Sistema Gerenciador de Banco de Dados) que usa SQL (Structured Query Language) para manipular dados.

**Conceitos básicos:**
- **Tabela:** Estrutura que armazena dados em linhas e colunas
- **Linha (Registro):** Uma entrada completa de dados
- **Coluna (Campo):** Um atributo específico dos dados
- **Chave Primária (PK):** Identifica unicamente cada registro
- **Chave Estrangeira (FK):** Relaciona tabelas diferentes

---

## Objetos Físicos

### Tabela

**O que é?** A tabela é onde os dados são realmente armazenados no disco. É a estrutura fundamental de qualquer banco de dados.

**Quando usar?** Sempre que você precisar armazenar dados permanentemente.

**Sintaxe básica:**
```sql
CREATE TABLE nome_da_tabela (
    coluna1 TIPO_DE_DADO RESTRIÇÕES,
    coluna2 TIPO_DE_DADO RESTRIÇÕES,
    ...
);
```

**Exemplo completo:**
```sql
-- Criar uma tabela de produtos
CREATE TABLE PRODUTOS (
    ID_PRODUTO NUMBER(10) PRIMARY KEY,        -- Identificador único
    NOME_PRODUTO VARCHAR2(100) NOT NULL,      -- Nome obrigatório
    PRECO NUMBER(10, 2),                      -- Preço com 2 casas decimais
    ESTOQUE NUMBER(10) DEFAULT 0,             -- Estoque inicial = 0
    DATA_CADASTRO DATE DEFAULT SYSDATE        -- Data atual
);
```

**Exemplo prático - Criar tabela de clientes:**
```sql
CREATE TABLE CLIENTES (
    ID_CLIENTE NUMBER(10) PRIMARY KEY,
    NOME VARCHAR2(100) NOT NULL,
    CPF VARCHAR2(11) UNIQUE,
    EMAIL VARCHAR2(100),
    DATA_NASCIMENTO DATE,
    ATIVO CHAR(1) DEFAULT 'S' CHECK (ATIVO IN ('S', 'N'))
);
```

**Comandos úteis para tabelas:**
```sql
-- Ver estrutura da tabela
DESC PRODUTOS;

-- Adicionar coluna
ALTER TABLE PRODUTOS ADD CATEGORIA VARCHAR2(50);

-- Modificar coluna
ALTER TABLE PRODUTOS MODIFY NOME_PRODUTO VARCHAR2(150);

-- Remover coluna
ALTER TABLE PRODUTOS DROP COLUMN CATEGORIA;

-- Deletar tabela
DROP TABLE PRODUTOS;

-- Renomear tabela
RENAME PRODUTOS TO PRODUTOS_LOJA;
```

---

### View Materializada

**O que é?** É como uma "fotografia" de uma consulta que é guardada fisicamente no banco. Diferente de uma VIEW normal, ela armazena dados reais.

**Quando usar?** Quando você tem consultas muito complexas que demoram para executar e precisa de performance.

**Diferença entre VIEW e VIEW MATERIALIZADA:**
- **VIEW:** Executa a consulta toda vez que é acessada (lenta em queries complexas)
- **VIEW MATERIALIZADA:** Guarda o resultado da consulta (rápida, mas precisa atualizar)

**Sintaxe:**
```sql
CREATE MATERIALIZED VIEW nome_mv
BUILD IMMEDIATE                    -- Cria e popula imediatamente
REFRESH [FAST|COMPLETE|FORCE]     -- Tipo de atualização
ON [COMMIT|DEMAND]                -- Quando atualizar
AS
SELECT ...;                        -- Sua consulta
```

**Tipos de REFRESH:**
- **FAST:** Atualiza apenas o que mudou (mais rápido, precisa de log)
- **COMPLETE:** Recria tudo do zero (mais lento, mas sempre correto)
- **FORCE:** Tenta FAST, se não conseguir faz COMPLETE

**Exemplo prático:**
```sql
-- Criar uma MV para relatório de vendas por cliente
CREATE MATERIALIZED VIEW mv_vendas_cliente
BUILD IMMEDIATE
REFRESH COMPLETE ON DEMAND
AS
SELECT 
    c.id_cliente,
    c.nome,
    COUNT(p.id_pedido) AS total_pedidos,
    SUM(p.valor_total) AS valor_total_gasto,
    AVG(p.valor_total) AS ticket_medio
FROM clientes c
LEFT JOIN pedidos p ON c.id_cliente = p.id_cliente
GROUP BY c.id_cliente, c.nome;

-- Atualizar manualmente a MV
EXEC DBMS_MVIEW.REFRESH('mv_vendas_cliente');

-- Consultar a MV (rápido!)
SELECT * FROM mv_vendas_cliente WHERE total_pedidos > 10;
```

---

### Índice

**O que é?** É como um "índice de livro" - ajuda a encontrar dados rapidamente sem precisar ler tudo.

**Quando usar?** Em colunas que você usa muito em pesquisas (WHERE, JOIN, ORDER BY).

**⚠️ Cuidado:** Índices aceleram consultas (SELECT) mas deixam inserções/atualizações mais lentas.

**Sintaxe:**
```sql
CREATE INDEX nome_indice 
ON tabela (coluna1, coluna2, ...);
```

**Exemplo prático:**
```sql
-- Criar índice simples no CPF (usado em buscas)
CREATE INDEX idx_clientes_cpf 
ON CLIENTES (CPF);

-- Criar índice composto (múltiplas colunas)
CREATE INDEX idx_pedidos_cliente_data 
ON PEDIDOS (ID_CLIENTE, DATA_PEDIDO);

-- Criar índice único (garante valores únicos)
CREATE UNIQUE INDEX idx_clientes_email 
ON CLIENTES (EMAIL);

-- Ver índices de uma tabela
SELECT INDEX_NAME, COLUMN_NAME 
FROM USER_IND_COLUMNS 
WHERE TABLE_NAME = 'CLIENTES';

-- Remover índice
DROP INDEX idx_clientes_cpf;
```

**Dica:** Crie índices em:
- Chaves primárias (Oracle cria automaticamente)
- Chaves estrangeiras
- Colunas usadas em WHERE frequentemente
- Colunas usadas em ORDER BY

---

## Objetos Lógicos

### View (Visão)

**O que é?** Uma "tabela virtual" que não armazena dados, apenas mostra o resultado de uma consulta. É como um "atalho" para uma query complexa.

**Quando usar?**
- Simplificar consultas complexas
- Ocultar colunas sensíveis (segurança)
- Padronizar acesso aos dados

**Sintaxe:**
```sql
CREATE [OR REPLACE] VIEW nome_view AS
SELECT ...
[WITH READ ONLY];  -- Opcional: impede alterações
```

**Exemplo prático:**
```sql
-- VIEW simples: produtos com estoque baixo
CREATE OR REPLACE VIEW vw_produtos_estoque_baixo AS
SELECT 
    ID_PRODUTO,
    NOME_PRODUTO,
    ESTOQUE,
    PRECO
FROM PRODUTOS
WHERE ESTOQUE < 10;

-- Usar a VIEW (como se fosse uma tabela)
SELECT * FROM vw_produtos_estoque_baixo;

-- VIEW com JOIN: pedidos com informações do cliente
CREATE OR REPLACE VIEW vw_pedidos_completos AS
SELECT 
    p.id_pedido,
    p.data_pedido,
    p.valor_total,
    c.nome AS nome_cliente,
    c.email AS email_cliente
FROM pedidos p
INNER JOIN clientes c ON p.id_cliente = c.id_cliente;

-- VIEW com agregação e segurança (oculta valores individuais)
CREATE OR REPLACE VIEW vw_vendas_por_mes AS
SELECT 
    TO_CHAR(DATA_PEDIDO, 'YYYY-MM') AS MES,
    COUNT(*) AS TOTAL_PEDIDOS,
    SUM(VALOR_TOTAL) AS TOTAL_VENDAS
FROM PEDIDOS
GROUP BY TO_CHAR(DATA_PEDIDO, 'YYYY-MM');

-- VIEW somente leitura
CREATE OR REPLACE VIEW vw_clientes_ativos AS
SELECT ID_CLIENTE, NOME, EMAIL
FROM CLIENTES
WHERE ATIVO = 'S'
WITH READ ONLY;

-- Remover VIEW
DROP VIEW vw_produtos_estoque_baixo;
```

---

### Procedimento Armazenado (Procedure)

**O que é?** Um "programa" guardado no banco de dados que executa uma sequência de comandos SQL. É como uma função que você pode chamar sempre que precisar.

**Quando usar?**
- Automatizar tarefas repetitivas
- Implementar regras de negócio
- Executar operações complexas em lote
- Melhorar segurança (controlar quem faz o quê)

**Sintaxe:**
```sql
CREATE OR REPLACE PROCEDURE nome_procedure (
    parametro1 IN TIPO,     -- IN: entrada
    parametro2 OUT TIPO,    -- OUT: saída
    parametro3 IN OUT TIPO  -- IN OUT: entrada e saída
) IS
    -- Declaração de variáveis locais
    variavel1 TIPO;
BEGIN
    -- Comandos SQL e lógica
    ...
    COMMIT;  -- ou ROLLBACK em caso de erro
EXCEPTION
    WHEN OTHERS THEN
        ROLLBACK;
        RAISE;
END;
/
```

**Exemplo 1: Procedure simples - Inserir produto**
```sql
CREATE OR REPLACE PROCEDURE sp_inserir_produto (
    p_id IN PRODUTOS.ID_PRODUTO%TYPE,
    p_nome IN VARCHAR2,
    p_preco IN NUMBER,
    p_estoque IN NUMBER DEFAULT 0
) IS
BEGIN
    INSERT INTO PRODUTOS (ID_PRODUTO, NOME_PRODUTO, PRECO, ESTOQUE)
    VALUES (p_id, p_nome, p_preco, p_estoque);
    
    COMMIT;
    
    DBMS_OUTPUT.PUT_LINE('Produto inserido com sucesso!');
EXCEPTION
    WHEN DUP_VAL_ON_INDEX THEN
        DBMS_OUTPUT.PUT_LINE('ERRO: Produto já existe!');
        ROLLBACK;
    WHEN OTHERS THEN
        DBMS_OUTPUT.PUT_LINE('ERRO: ' || SQLERRM);
        ROLLBACK;
END;
/

-- Executar a procedure
EXEC sp_inserir_produto(1, 'Notebook Dell', 3500.00, 15);
```

**Exemplo 2: Procedure com cálculo - Atualizar estoque**
```sql
CREATE OR REPLACE PROCEDURE sp_atualizar_estoque (
    p_id_produto IN NUMBER,
    p_quantidade IN NUMBER,
    p_tipo_operacao IN VARCHAR2  -- 'E' para entrada, 'S' para saída
) IS
    v_estoque_atual NUMBER;
BEGIN
    -- Buscar estoque atual
    SELECT ESTOQUE INTO v_estoque_atual
    FROM PRODUTOS
    WHERE ID_PRODUTO = p_id_produto;
    
    -- Atualizar conforme operação
    IF p_tipo_operacao = 'E' THEN
        UPDATE PRODUTOS 
        SET ESTOQUE = ESTOQUE + p_quantidade
        WHERE ID_PRODUTO = p_id_produto;
        
        DBMS_OUTPUT.PUT_LINE('Entrada de ' || p_quantidade || ' unidades');
    ELSIF p_tipo_operacao = 'S' THEN
        IF v_estoque_atual < p_quantidade THEN
            RAISE_APPLICATION_ERROR(-20001, 'Estoque insuficiente!');
        END IF;
        
        UPDATE PRODUTOS 
        SET ESTOQUE = ESTOQUE - p_quantidade
        WHERE ID_PRODUTO = p_id_produto;
        
        DBMS_OUTPUT.PUT_LINE('Saída de ' || p_quantidade || ' unidades');
    ELSE
        RAISE_APPLICATION_ERROR(-20002, 'Tipo de operação inválido!');
    END IF;
    
    COMMIT;
EXCEPTION
    WHEN NO_DATA_FOUND THEN
        DBMS_OUTPUT.PUT_LINE('ERRO: Produto não encontrado!');
    WHEN OTHERS THEN
        DBMS_OUTPUT.PUT_LINE('ERRO: ' || SQLERRM);
        ROLLBACK;
END;
/

-- Usar a procedure
EXEC sp_atualizar_estoque(1, 5, 'S');  -- Saída de 5 unidades
```

**Exemplo 3: Procedure com OUT - Calcular total**
```sql
CREATE OR REPLACE PROCEDURE sp_calcular_total_pedidos (
    p_id_cliente IN NUMBER,
    p_total OUT NUMBER,
    p_quantidade OUT NUMBER
) IS
BEGIN
    SELECT 
        NVL(SUM(VALOR_TOTAL), 0),
        COUNT(*)
    INTO p_total, p_quantidade
    FROM PEDIDOS
    WHERE ID_CLIENTE = p_id_cliente;
END;
/

-- Usar procedure com parâmetros OUT
DECLARE
    v_total NUMBER;
    v_qtd NUMBER;
BEGIN
    sp_calcular_total_pedidos(1, v_total, v_qtd);
    DBMS_OUTPUT.PUT_LINE('Total: R$ ' || v_total);
    DBMS_OUTPUT.PUT_LINE('Quantidade: ' || v_qtd);
END;
/
```

---

## Operações de Junção (JOINs)

**O que são JOINs?** Comandos para combinar dados de duas ou mais tabelas baseado em uma relação entre elas.

### Exemplo de tabelas para os JOINs:

```sql
-- Tabela CLIENTES
ID_CLIENTE | NOME
1          | João
2          | Maria
3          | Pedro

-- Tabela PEDIDOS
ID_PEDIDO | ID_CLIENTE | VALOR
101       | 1          | 100.00
102       | 1          | 200.00
103       | 2          | 150.00
```

---

### INNER JOIN

**O que faz?** Retorna apenas as linhas que têm correspondência nas DUAS tabelas.

**Quando usar?** Quando você quer apenas dados que existem em ambas as tabelas.

**Sintaxe:**
```sql
SELECT colunas
FROM tabela1
INNER JOIN tabela2 ON tabela1.coluna = tabela2.coluna;
```

**Exemplo:**
```sql
-- Listar pedidos com nome do cliente
SELECT 
    c.NOME AS cliente,
    p.ID_PEDIDO,
    p.VALOR
FROM CLIENTES c
INNER JOIN PEDIDOS p ON c.ID_CLIENTE = p.ID_CLIENTE;

-- Resultado:
-- cliente | ID_PEDIDO | VALOR
-- João    | 101       | 100.00
-- João    | 102       | 200.00
-- Maria   | 103       | 150.00
-- (Pedro não aparece pois não tem pedidos)
```

**Diagrama visual:**
```
CLIENTES ∩ PEDIDOS
(Apenas onde há match)
```

---

### LEFT JOIN

**O que faz?** Retorna TODAS as linhas da tabela da ESQUERDA e as correspondentes da direita. Se não houver correspondência, preenche com NULL.

**Quando usar?** Quando você quer todos os registros da primeira tabela, mesmo que não tenham relação com a segunda.

**Sintaxe:**
```sql
SELECT colunas
FROM tabela1
LEFT JOIN tabela2 ON tabela1.coluna = tabela2.coluna;
```

**Exemplo:**
```sql
-- Listar TODOS os clientes e seus pedidos (se tiverem)
SELECT 
    c.NOME AS cliente,
    p.ID_PEDIDO,
    p.VALOR
FROM CLIENTES c
LEFT JOIN PEDIDOS p ON c.ID_CLIENTE = p.ID_CLIENTE;

-- Resultado:
-- cliente | ID_PEDIDO | VALOR
-- João    | 101       | 100.00
-- João    | 102       | 200.00
-- Maria   | 103       | 150.00
-- Pedro   | NULL      | NULL    <- Aparece mesmo sem pedidos
```

**Uso prático - Encontrar clientes sem pedidos:**
```sql
SELECT c.NOME
FROM CLIENTES c
LEFT JOIN PEDIDOS p ON c.ID_CLIENTE = p.ID_CLIENTE
WHERE p.ID_PEDIDO IS NULL;
-- Resultado: Pedro
```

---

### RIGHT JOIN

**O que faz?** Retorna TODAS as linhas da tabela da DIREITA e as correspondentes da esquerda.

**Quando usar?** Similar ao LEFT JOIN, mas priorizando a tabela da direita.

**Sintaxe:**
```sql
SELECT colunas
FROM tabela1
RIGHT JOIN tabela2 ON tabela1.coluna = tabela2.coluna;
```

**Exemplo:**
```sql
-- TODOS os pedidos e clientes (se existirem)
SELECT 
    c.NOME AS cliente,
    p.ID_PEDIDO,
    p.VALOR
FROM CLIENTES c
RIGHT JOIN PEDIDOS p ON c.ID_CLIENTE = p.ID_CLIENTE;
```

**Nota:** RIGHT JOIN é menos usado. Geralmente usa-se LEFT JOIN invertendo as tabelas.

---

### FULL OUTER JOIN

**O que faz?** Retorna TODAS as linhas de AMBAS as tabelas, com NULL onde não há correspondência.

**Quando usar?** Quando você quer ver tudo de ambas as tabelas.

**Sintaxe:**
```sql
SELECT colunas
FROM tabela1
FULL OUTER JOIN tabela2 ON tabela1.coluna = tabela2.coluna;
```

**Exemplo:**
```sql
SELECT 
    c.NOME AS cliente,
    p.ID_PEDIDO,
    p.VALOR
FROM CLIENTES c
FULL OUTER JOIN PEDIDOS p ON c.ID_CLIENTE = p.ID_CLIENTE;

-- Mostra:
-- - Clientes com pedidos
-- - Clientes sem pedidos (NULL nos pedidos)
-- - Pedidos sem clientes (NULL no cliente) - se existissem
```

---

### CROSS JOIN

**O que faz?** Combina CADA linha da primeira tabela com CADA linha da segunda (produto cartesiano).

**Quando usar?** Raramente usado. Útil para gerar combinações de produtos, horários, etc.

**Sintaxe:**
```sql
SELECT colunas
FROM tabela1
CROSS JOIN tabela2;
```

**Exemplo prático:**
```sql
-- Combinar todas as cores com todos os tamanhos
SELECT 
    c.NOME_COR,
    t.NOME_TAMANHO
FROM CORES c
CROSS JOIN TAMANHOS t;

-- Se CORES tem: {Vermelho, Azul}
-- E TAMANHOS tem: {P, M, G}
-- Resultado (6 combinações):
-- Vermelho | P
-- Vermelho | M
-- Vermelho | G
-- Azul     | P
-- Azul     | M
-- Azul     | G
```

---

### Resumo Visual dos JOINs

```
INNER JOIN:      ●●●     (só o que está em ambas)
LEFT JOIN:     ●●●●●●    (todas da esquerda + matches)
RIGHT JOIN:      ●●●●●●  (todas da direita + matches)
FULL OUTER:    ●●●●●●●●● (tudo de ambas)
CROSS JOIN:    ●×●×●×●×● (todas as combinações)
```

---

## Modelagem de Dados

### Generalização/Especialização

**O que é?** Uma técnica de modelagem onde você tem uma entidade "pai" (genérica) e entidades "filhas" (específicas) que herdam características do pai.

**Conceito:** É como uma hierarquia de classes na programação orientada a objetos.

**Quando usar?** Quando você tem entidades que compartilham características comuns mas também têm atributos específicos.

**Exemplo conceitual:**

```
PESSOA (Entidade Genérica - Pai)
├── Atributos Comuns: ID, Nome, CPF, Data Nascimento
│
├── PESSOA FÍSICA (Especialização 1)
│   └── Atributos Específicos: RG, Estado Civil
│
└── PESSOA JURÍDICA (Especialização 2)
    └── Atributos Específicos: CNPJ, Razão Social, Inscrição Estadual
```

**Implementação no Oracle:**

**Opção 1: Tabela Única (Table per Hierarchy)**
```sql
CREATE TABLE PESSOAS (
    ID NUMBER PRIMARY KEY,
    TIPO CHAR(1) CHECK (TIPO IN ('F', 'J')),  -- F=Física, J=Jurídica
    
    -- Atributos comuns
    NOME VARCHAR2(100),
    EMAIL VARCHAR2(100),
    TELEFONE VARCHAR2(20),
    
    -- Atributos de Pessoa Física (NULL se for PJ)
    CPF VARCHAR2(11),
    RG VARCHAR2(20),
    DATA_NASCIMENTO DATE,
    
    -- Atributos de Pessoa Jurídica (NULL se for PF)
    CNPJ VARCHAR2(14),
    RAZAO_SOCIAL VARCHAR2(150),
    INSCRICAO_ESTADUAL VARCHAR2(20)
);

-- Inserir Pessoa Física
INSERT INTO PESSOAS (ID, TIPO, NOME, EMAIL, CPF, RG, DATA_NASCIMENTO)
VALUES (1, 'F', 'João Silva', 'joao@email.com', '12345678901', 'MG1234567', TO_DATE('1990-05-15', 'YYYY-MM-DD'));

-- Inserir Pessoa Jurídica
INSERT INTO PESSOAS (ID, TIPO, NOME, EMAIL, CNPJ, RAZAO_SOCIAL, INSCRICAO_ESTADUAL)
VALUES (2, 'J', 'Tech Solutions', 'contato@tech.com', '12345678000190', 'Tech Solutions LTDA', '123456789');

-- VIEW para Pessoas Físicas
CREATE VIEW VW_PESSOAS_FISICAS AS
SELECT ID, NOME, EMAIL, CPF, RG, DATA_NASCIMENTO
FROM PESSOAS
WHERE TIPO = 'F';

-- VIEW para Pessoas Jurídicas
CREATE VIEW VW_PESSOAS_JURIDICAS AS
SELECT ID, NOME, EMAIL, CNPJ, RAZAO_SOCIAL, INSCRICAO_ESTADUAL
FROM PESSOAS
WHERE TIPO = 'J';
```

**Opção 2: Tabelas Separadas (Table per Type)**
```sql
-- Tabela pai com atributos comuns
CREATE TABLE PESSOAS (
    ID NUMBER PRIMARY KEY,
    TIPO CHAR(1) CHECK (TIPO IN ('F', 'J')),
    NOME VARCHAR2(100),
    EMAIL VARCHAR2(100),
    TELEFONE VARCHAR2(20)
);

-- Tabela especializada - Pessoa Física
CREATE TABLE PESSOAS_FISICAS (
    ID NUMBER PRIMARY KEY,
    CPF VARCHAR2(11) NOT NULL,
    RG VARCHAR2(20),
    DATA_NASCIMENTO DATE,
    FOREIGN KEY (ID) REFERENCES PESSOAS(ID)
);

-- Tabela especializada - Pessoa Jurídica
CREATE TABLE PESSOAS_JURIDICAS (
    ID NUMBER PRIMARY KEY,
    CNPJ VARCHAR2(14) NOT NULL,
    RAZAO_SOCIAL VARCHAR2(150),
    INSCRICAO_ESTADUAL VARCHAR2(20),
    FOREIGN KEY (ID) REFERENCES PESSOAS(ID)
);

-- Inserir Pessoa Física
INSERT INTO PESSOAS (ID, TIPO, NOME, EMAIL) 
VALUES (1, 'F', 'Maria Santos', 'maria@email.com');

INSERT INTO PESSOAS_FISICAS (ID, CPF, DATA_NASCIMENTO)
VALUES (1, '98765432100', TO_DATE('1985-03-20', 'YYYY-MM-DD'));

-- VIEW completa de Pessoa Física
CREATE VIEW VW_PESSOAS_FISICAS_COMPLETA AS
SELECT 
    p.ID,
    p.NOME,
    p.EMAIL,
    p.TELEFONE,
    pf.CPF,
    pf.RG,
    pf.DATA_NASCIMENTO
FROM PESSOAS p
INNER JOIN PESSOAS_FISICAS pf ON p.ID = pf.ID
WHERE p.TIPO = 'F';
```

**Exemplo 2: Livros**

```
LIVRO (Genérico)
├── Código, Título, Autor, ISBN, Ano
│
├── LIVRO DIDÁTICO
│   └── Disciplina, Série, Nível Escolar
│
└── LIVRO NÃO DIDÁTICO
    └── Gênero Literário, Público Alvo
```

**Implementação:**
```sql
CREATE TABLE LIVROS (
    CODIGO NUMBER PRIMARY KEY,
    TIPO_LIVRO CHAR(1) CHECK (TIPO_LIVRO IN ('D', 'N')),  -- D=Didático, N=Não Didático
    TITULO VARCHAR2(200) NOT NULL,
    AUTOR VARCHAR2(150),
    ISBN VARCHAR2(20),
    ANO_PUBLICACAO NUMBER(4),
    
    -- Atributos de Livro Didático
    DISCIPLINA VARCHAR2(50),
    SERIE VARCHAR2(20),
    NIVEL_ESCOLAR VARCHAR2(30),
    
    -- Atributos de Livro Não Didático
    GENERO_LITERARIO VARCHAR2(50),
    PUBLICO_ALVO VARCHAR2(50)
);

-- Livro Didático
INSERT INTO LIVROS (CODIGO, TIPO_LIVRO, TITULO, AUTOR, ISBN, ANO_PUBLICACAO, DISCIPLINA, SERIE, NIVEL_ESCOLAR)
VALUES (1, 'D', 'Matemática Fundamental', 'Carlos Silva', '9788501234567', 2023, 'Matemática', '6º Ano', 'Fundamental II');

-- Livro Não Didático
INSERT INTO LIVROS (CODIGO, TIPO_LIVRO, TITULO, AUTOR, ISBN, ANO_PUBLICACAO, GENERO_LITERARIO, PUBLICO_ALVO)
VALUES (2, 'N', 'O Pequeno Príncipe', 'Antoine de Saint-Exupéry', '9788535914528', 1943, 'Fábula', 'Infantojuvenil');
```

---

## Comandos Básicos Essenciais

### SELECT - Consultar dados

```sql
-- Selecionar tudo
SELECT * FROM PRODUTOS;

-- Selecionar colunas específicas
SELECT NOME_PRODUTO, PRECO FROM PRODUTOS;

-- Com filtro (WHERE)
SELECT * FROM PRODUTOS WHERE PRECO > 100;

-- Com ordenação
SELECT * FROM PRODUTOS ORDER BY PRECO DESC;  -- DESC = decrescente, ASC = crescente

-- Com limite (apenas Oracle 12c+)
SELECT * FROM PRODUTOS FETCH FIRST 10 ROWS ONLY;

-- Com múltiplos filtros
SELECT * FROM PRODUTOS 
WHERE PRECO > 50 
AND ESTOQUE > 0 
AND NOME_PRODUTO LIKE '%Notebook%';

-- Com agregação
SELECT 
    CATEGORIA,
    COUNT(*) AS total_produtos,
    AVG(PRECO) AS preco_medio,
    SUM(ESTOQUE) AS estoque_total
FROM PRODUTOS
GROUP BY CATEGORIA
HAVING COUNT(*) > 5;  -- HAVING filtra depois do GROUP BY
```

### INSERT - Inserir dados

```sql
-- Inserir especificando colunas
INSERT INTO PRODUTOS (ID_PRODUTO, NOME_PRODUTO, PRECO, ESTOQUE)
VALUES (1, 'Mouse Gamer', 150.00, 50);

-- Inserir sem especificar colunas (deve seguir ordem da tabela)
INSERT INTO PRODUTOS VALUES (2, 'Teclado Mecânico', 350.00, 30, SYSDATE);

-- Inserir múltiplas linhas
INSERT ALL
    INTO PRODUTOS VALUES (3, 'Monitor 24"', 800.00, 15, SYSDATE)
    INTO PRODUTOS VALUES (4, 'Webcam HD', 200.00, 40, SYSDATE)
SELECT * FROM DUAL;

-- Inserir a partir de outra consulta
INSERT INTO PRODUTOS_BACKUP
SELECT * FROM PRODUTOS WHERE ESTOQUE < 10;
```

### UPDATE - Atualizar dados

```sql
-- Atualizar um campo
UPDATE PRODUTOS 
SET PRECO = 160.00 
WHERE ID_PRODUTO = 1;

-- Atualizar múltiplos campos
UPDATE PRODUTOS 
SET PRECO = PRECO * 1.10,  -- Aumenta 10%
    ESTOQUE = ESTOQUE - 1
WHERE ID_PRODUTO = 2;

-- Atualizar com subconsulta
UPDATE PRODUTOS p
SET PRECO = (SELECT AVG(PRECO) FROM PRODUTOS WHERE CATEGORIA = p.CATEGORIA)
WHERE ESTOQUE = 0;

-- Atualizar todos (cuidado!)
UPDATE PRODUTOS SET ATIVO = 'S';
```

### DELETE - Deletar dados

```sql
-- Deletar registro específico
DELETE FROM PRODUTOS WHERE ID_PRODUTO = 1;

-- Deletar com condição
DELETE FROM PRODUTOS WHERE ESTOQUE = 0 AND DATA_CADASTRO < ADD_MONTHS(SYSDATE, -12);

-- Deletar tudo (cuidado!)
DELETE FROM PRODUTOS;

-- Alternativa mais rápida para deletar tudo
TRUNCATE TABLE PRODUTOS;  -- Não permite ROLLBACK!
```

### COMMIT e ROLLBACK - Controle de transações

```sql
-- Iniciar transação (implícito no Oracle)
UPDATE PRODUTOS SET PRECO = 100 WHERE ID_PRODUTO = 1;
INSERT INTO PRODUTOS VALUES (5, 'Teste', 50, 10, SYSDATE);

-- Confirmar alterações
COMMIT;

-- OU desfazer alterações
ROLLBACK;

-- Savepoint para rollback parcial
UPDATE PRODUTOS SET PRECO = 200 WHERE ID_PRODUTO = 1;
SAVEPOINT ponto1;

DELETE FROM PRODUTOS WHERE ID_PRODUTO = 2;
SAVEPOINT ponto2;

-- Voltar para ponto1 (desfaz apenas o DELETE)
ROLLBACK TO ponto1;

-- Confirmar tudo
COMMIT;
```

---

## Tipos de Dados Oracle

### Tipos Numéricos

```sql
NUMBER(p, s)     -- p = precisão total, s = casas decimais
NUMBER(10)       -- Inteiro até 10 dígitos
NUMBER(10, 2)    -- Decimal com 8 dígitos antes e 2 depois da vírgula
INTEGER          -- Equivalente a NUMBER(38)
FLOAT            -- Número de ponto flutuante
```

**Exemplos:**
```sql
CREATE TABLE EXEMPLOS_NUMERICOS (
    ID INTEGER,
    IDADE NUMBER(3),           -- 0 a 999
    PRECO NUMBER(10, 2),       -- 99999999.99
    PESO FLOAT,
    PERCENTUAL NUMBER(5, 2)    -- 999.99 (até 100%)
);
```

### Tipos de Texto

```sql
VARCHAR2(tamanho)  -- Texto variável (mais usado) - até 4000 caracteres
CHAR(tamanho)      -- Texto fixo (preenche com espaços) - até 2000
NVARCHAR2(tamanho) -- Texto Unicode variável
CLOB               -- Textos muito grandes (até 4GB)
```

**Exemplos:**
```sql
CREATE TABLE EXEMPLOS_TEXTO (
    NOME VARCHAR2(100),       -- Nome com até 100 caracteres
    SIGLA CHAR(2),            -- Sempre 2 caracteres (ex: SP, RJ)
    CPF VARCHAR2(11),         -- 11 dígitos sem formatação
    DESCRICAO CLOB            -- Textos longos
);
```

### Tipos de Data/Hora

```sql
DATE              -- Data e hora (mais usado)
TIMESTAMP         -- Data e hora com frações de segundo
TIMESTAMP WITH TIME ZONE  -- Com fuso horário
```

**Exemplos:**
```sql
CREATE TABLE EXEMPLOS_DATA (
    DATA_NASCIMENTO DATE,
    DATA_CADASTRO DATE DEFAULT SYSDATE,  -- Data atual
    ULTIMA_ATUALIZACAO TIMESTAMP
);

-- Inserir datas
INSERT INTO EXEMPLOS_DATA (DATA_NASCIMENTO) 
VALUES (TO_DATE('15/05/1990', 'DD/MM/YYYY'));

-- Funções de data úteis
SELECT 
    SYSDATE,                           -- Data/hora atual
    TO_CHAR(SYSDATE, 'DD/MM/YYYY'),   -- Formatar data
    ADD_MONTHS(SYSDATE, 3),           -- Adicionar 3 meses
    TRUNC(SYSDATE),                   -- Remover hora
    MONTHS_BETWEEN(SYSDATE, DATA_NASCIMENTO) / 12 AS IDADE
FROM EXEMPLOS_DATA;
```

### Outros Tipos

```sql
BLOB              -- Dados binários grandes (imagens, PDFs) - até 4GB
RAW(tamanho)      -- Dados binários pequenos
ROWID             -- Identificador único de linha (automático)
```

---

## Constraints (Restrições)

**O que são?** Regras aplicadas nas colunas para garantir a integridade dos dados.

### PRIMARY KEY - Chave Primária

```sql
-- Define na criação da tabela
CREATE TABLE CLIENTES (
    ID_CLIENTE NUMBER PRIMARY KEY,  -- Não pode ser NULL e deve ser único
    NOME VARCHAR2(100)
);

-- Ou separadamente
CREATE TABLE CLIENTES (
    ID_CLIENTE NUMBER,
    NOME VARCHAR2(100),
    CONSTRAINT pk_clientes PRIMARY KEY (ID_CLIENTE)
);

-- Chave primária composta (múltiplas colunas)
CREATE TABLE ITENS_PEDIDO (
    ID_PEDIDO NUMBER,
    ID_PRODUTO NUMBER,
    QUANTIDADE NUMBER,
    CONSTRAINT pk_itens PRIMARY KEY (ID_PEDIDO, ID_PRODUTO)
);
```

### FOREIGN KEY - Chave Estrangeira

```sql
-- Relacionamento entre tabelas
CREATE TABLE PEDIDOS (
    ID_PEDIDO NUMBER PRIMARY KEY,
    ID_CLIENTE NUMBER,
    VALOR_TOTAL NUMBER(10, 2),
    
    -- Define o relacionamento
    CONSTRAINT fk_pedidos_clientes 
    FOREIGN KEY (ID_CLIENTE) 
    REFERENCES CLIENTES(ID_CLIENTE)
);

-- Com ações em cascata
CREATE TABLE PEDIDOS (
    ID_PEDIDO NUMBER PRIMARY KEY,
    ID_CLIENTE NUMBER,
    
    CONSTRAINT fk_pedidos_clientes 
    FOREIGN KEY (ID_CLIENTE) 
    REFERENCES CLIENTES(ID_CLIENTE)
    ON DELETE CASCADE  -- Deleta pedidos se cliente for deletado
);

-- ON DELETE SET NULL - Define NULL se o pai for deletado
-- ON DELETE CASCADE - Deleta os filhos se o pai for deletado
```

### NOT NULL - Não pode ser vazio

```sql
CREATE TABLE PRODUTOS (
    ID_PRODUTO NUMBER PRIMARY KEY,
    NOME_PRODUTO VARCHAR2(100) NOT NULL,  -- Obrigatório
    PRECO NUMBER(10, 2) NOT NULL,
    DESCRICAO VARCHAR2(500)  -- Opcional (pode ser NULL)
);
```

### UNIQUE - Valor único

```sql
CREATE TABLE USUARIOS (
    ID NUMBER PRIMARY KEY,
    EMAIL VARCHAR2(100) UNIQUE,  -- Não pode repetir
    CPF VARCHAR2(11) UNIQUE,
    NOME VARCHAR2(100)
);

-- Ou nomeado
CREATE TABLE USUARIOS (
    ID NUMBER PRIMARY KEY,
    EMAIL VARCHAR2(100),
    CONSTRAINT uk_usuarios_email UNIQUE (EMAIL)
);
```

### CHECK - Validação customizada

```sql
CREATE TABLE FUNCIONARIOS (
    ID NUMBER PRIMARY KEY,
    NOME VARCHAR2(100),
    SALARIO NUMBER(10, 2) CHECK (SALARIO > 0),  -- Deve ser positivo
    SEXO CHAR(1) CHECK (SEXO IN ('M', 'F')),    -- Apenas M ou F
    IDADE NUMBER CHECK (IDADE >= 18 AND IDADE <= 100),
    STATUS CHAR(1) DEFAULT 'A' CHECK (STATUS IN ('A', 'I'))  -- Ativo ou Inativo
);

-- Check nomeado
CREATE TABLE PRODUTOS (
    ID NUMBER PRIMARY KEY,
    PRECO NUMBER(10, 2),
    PRECO_PROMOCIONAL NUMBER(10, 2),
    
    CONSTRAINT ck_preco_promocional 
    CHECK (PRECO_PROMOCIONAL IS NULL OR PRECO_PROMOCIONAL < PRECO)
);
```

### DEFAULT - Valor padrão

```sql
CREATE TABLE PEDIDOS (
    ID_PEDIDO NUMBER PRIMARY KEY,
    DATA_PEDIDO DATE DEFAULT SYSDATE,  -- Data atual por padrão
    STATUS CHAR(1) DEFAULT 'P',        -- 'P' por padrão
    ATIVO CHAR(1) DEFAULT 'S'
);
```

### Gerenciar Constraints

```sql
-- Ver constraints de uma tabela
SELECT CONSTRAINT_NAME, CONSTRAINT_TYPE, SEARCH_CONDITION
FROM USER_CONSTRAINTS
WHERE TABLE_NAME = 'PRODUTOS';

-- Adicionar constraint depois
ALTER TABLE PRODUTOS 
ADD CONSTRAINT ck_preco_positivo CHECK (PRECO > 0);

-- Remover constraint
ALTER TABLE PRODUTOS 
DROP CONSTRAINT ck_preco_positivo;

-- Desabilitar temporariamente
ALTER TABLE PRODUTOS 
DISABLE CONSTRAINT fk_produtos_categoria;

-- Habilitar novamente
ALTER TABLE PRODUTOS 
ENABLE CONSTRAINT fk_produtos_categoria;
```

---

## Funções Úteis do Oracle

### Funções de Texto

```sql
SELECT 
    UPPER('texto')              -- TEXTO
    LOWER('TEXTO')              -- texto
    INITCAP('joao silva')       -- Joao Silva
    LENGTH('Oracle')            -- 6
    SUBSTR('Oracle DB', 1, 6)   -- Oracle
    REPLACE('ABC', 'B', 'X')    -- AXC
    TRIM('  teste  ')           -- teste
    CONCAT('A', 'B')            -- AB (ou use ||)
    INSTR('Oracle', 'a')        -- 3 (posição do 'a')
FROM DUAL;

-- DUAL é uma tabela fictícia para testes
```

### Funções Numéricas

```sql
SELECT 
    ROUND(15.567, 2)      -- 15.57 (arredonda)
    TRUNC(15.567, 2)      -- 15.56 (trunca)
    CEIL(15.3)            -- 16 (arredonda pra cima)
    FLOOR(15.8)           -- 15 (arredonda pra baixo)
    MOD(10, 3)            -- 1 (resto da divisão)
    POWER(2, 3)           -- 8 (2³)
    SQRT(16)              -- 4 (raiz quadrada)
    ABS(-10)              -- 10 (valor absoluto)
FROM DUAL;
```

### Funções de Data

```sql
SELECT 
    SYSDATE,                              -- Data/hora atual
    CURRENT_DATE,                         -- Data atual
    ADD_MONTHS(SYSDATE, 3),              -- Adiciona 3 meses
    MONTHS_BETWEEN(SYSDATE, '01-JAN-2020'), -- Diferença em meses
    NEXT_DAY(SYSDATE, 'MONDAY'),         -- Próxima segunda
    LAST_DAY(SYSDATE),                   -- Último dia do mês
    TRUNC(SYSDATE),                      -- Remove hora (00:00:00)
    TO_CHAR(SYSDATE, 'DD/MM/YYYY HH24:MI:SS'), -- Formatar
    TO_DATE('15/05/2023', 'DD/MM/YYYY')  -- Converter texto em data
FROM DUAL;
```

### Funções de Conversão

```sql
SELECT 
    TO_CHAR(SYSDATE, 'DD/MM/YYYY'),     -- Date para String
    TO_CHAR(1234.56, '9999.99'),        -- Number para String
    TO_NUMBER('1234'),                   -- String para Number
    TO_DATE('15/05/2023', 'DD/MM/YYYY'), -- String para Date
    CAST(123 AS VARCHAR2(10)),          -- Converte tipo
    NVL(coluna_nullable, 'Valor padrão') -- Se NULL, usa padrão
FROM DUAL;
```

### Funções Condicionais

```sql
-- CASE (como if/else)
SELECT 
    NOME_PRODUTO,
    PRECO,
    CASE 
        WHEN PRECO < 100 THEN 'Barato'
        WHEN PRECO < 500 THEN 'Médio'
        ELSE 'Caro'
    END AS CATEGORIA_PRECO
FROM PRODUTOS;

-- NVL (substituir NULL)
SELECT 
    NOME,
    NVL(EMAIL, 'Sem email') AS EMAIL
FROM CLIENTES;

-- NVL2 (se não for NULL, senão)
SELECT 
    NVL2(EMAIL, 'Tem email', 'Sem email')
FROM CLIENTES;

-- COALESCE (primeiro valor não-NULL)
SELECT 
    COALESCE(EMAIL, TELEFONE, 'Sem contato')
FROM CLIENTES;

-- DECODE (similar ao CASE)
SELECT 
    DECODE(SEXO, 'M', 'Masculino', 'F', 'Feminino', 'Outro')
FROM FUNCIONARIOS;
```

---

## Subconsultas (Subqueries)

**O que são?** Uma consulta dentro de outra consulta.

```sql
-- Subconsulta no WHERE
SELECT NOME_PRODUTO, PRECO
FROM PRODUTOS
WHERE PRECO > (SELECT AVG(PRECO) FROM PRODUTOS);

-- Subconsulta com IN
SELECT NOME
FROM CLIENTES
WHERE ID_CLIENTE IN (
    SELECT ID_CLIENTE 
    FROM PEDIDOS 
    WHERE VALOR_TOTAL > 1000
);

-- Subconsulta com EXISTS (verifica se existe)
SELECT c.NOME
FROM CLIENTES c
WHERE EXISTS (
    SELECT 1 
    FROM PEDIDOS p 
    WHERE p.ID_CLIENTE = c.ID_CLIENTE
);

-- Subconsulta no SELECT
SELECT 
    p.NOME_PRODUTO,
    p.PRECO,
    (SELECT AVG(PRECO) FROM PRODUTOS) AS PRECO_MEDIO
FROM PRODUTOS p;

-- Subconsulta no FROM (tabela derivada)
SELECT *
FROM (
    SELECT NOME, PRECO, PRECO * 0.9 AS PRECO_DESCONTO
    FROM PRODUTOS
)
WHERE PRECO_DESCONTO < 100;
```

---

## Operadores Úteis

```sql
-- Comparação
=, !=, <>, <, >, <=, >=

-- Lógicos
AND, OR, NOT

-- LIKE (padrões de texto)
SELECT * FROM CLIENTES WHERE NOME LIKE 'João%';  -- Começa com João
SELECT * FROM CLIENTES WHERE NOME LIKE '%Silva'; -- Termina com Silva
SELECT * FROM CLIENTES WHERE NOME LIKE '%Ana%';  -- Contém Ana
SELECT * FROM CLIENTES WHERE NOME LIKE 'J_ão';   -- _ substitui 1 caractere

-- IN (lista de valores)
SELECT * FROM PRODUTOS WHERE CATEGORIA IN ('Eletrônicos', 'Informática');

-- BETWEEN (intervalo)
SELECT * FROM PRODUTOS WHERE PRECO BETWEEN 100 AND 500;

-- IS NULL / IS NOT NULL
SELECT * FROM CLIENTES WHERE EMAIL IS NULL;
SELECT * FROM CLIENTES WHERE EMAIL IS NOT NULL;

-- DISTINCT (valores únicos)
SELECT DISTINCT CATEGORIA FROM PRODUTOS;
```

---

## Sequências (SEQUENCE) - Auto Incremento

**O que são?** Geradores automáticos de números sequenciais, usados para IDs.

```sql
-- Criar sequência
CREATE SEQUENCE seq_produtos
START WITH 1
INCREMENT BY 1
NOCACHE;

-- Usar na inserção
INSERT INTO PRODUTOS (ID_PRODUTO, NOME_PRODUTO, PRECO)
VALUES (seq_produtos.NEXTVAL, 'Produto Novo', 100.00);

-- Ver valor atual
SELECT seq_produtos.CURRVAL FROM DUAL;

-- Reiniciar sequência
ALTER SEQUENCE seq_produtos RESTART START WITH 100;

-- Deletar sequência
DROP SEQUENCE seq_produtos;

-- Exemplo completo
CREATE SEQUENCE seq_clientes START WITH 1 INCREMENT BY 1;

INSERT INTO CLIENTES (ID_CLIENTE, NOME, EMAIL)
VALUES (seq_clientes.NEXTVAL, 'Maria Silva', 'maria@email.com');
```

---

## Triggers - Gatilhos Automáticos

**O que são?** Códigos que executam automaticamente quando algo acontece na tabela (INSERT, UPDATE, DELETE).

```sql
-- Trigger para auditoria
CREATE OR REPLACE TRIGGER trg_auditoria_produtos
AFTER INSERT OR UPDATE OR DELETE ON PRODUTOS
FOR EACH ROW
BEGIN
    IF INSERTING THEN
        INSERT INTO LOG_AUDITORIA 
        VALUES (SYSDATE, USER, 'INSERT', 'PRODUTOS', :NEW.ID_PRODUTO);
    ELSIF UPDATING THEN
        INSERT INTO LOG_AUDITORIA 
        VALUES (SYSDATE, USER, 'UPDATE', 'PRODUTOS', :OLD.ID_PRODUTO);
    ELSIF DELETING THEN
        INSERT INTO LOG_AUDITORIA 
        VALUES (SYSDATE, USER, 'DELETE', 'PRODUTOS', :OLD.ID_PRODUTO);
    END IF;
END;
/

-- Trigger para auto-incremento (alternativa à sequence)
CREATE OR REPLACE TRIGGER trg_id_produtos
BEFORE INSERT ON PRODUTOS
FOR EACH ROW
BEGIN
    IF :NEW.ID_PRODUTO IS NULL THEN
        SELECT seq_produtos.NEXTVAL INTO :NEW.ID_PRODUTO FROM DUAL;
    END IF;
END;
/

-- Trigger para validação
CREATE OR REPLACE TRIGGER trg_valida_preco
BEFORE INSERT OR UPDATE ON PRODUTOS
FOR EACH ROW
BEGIN
    IF :NEW.PRECO < 0 THEN
        RAISE_APPLICATION_ERROR(-20001, 'Preço não pode ser negativo!');
    END IF;
    
    IF :NEW.PRECO_PROMOCIONAL >= :NEW.PRECO THEN
        RAISE_APPLICATION_ERROR(-20002, 'Preço promocional deve ser menor!');
    END IF;
END;
/

-- Ver triggers
SELECT TRIGGER_NAME, TABLE_NAME, STATUS 
FROM USER_TRIGGERS;

-- Desabilitar trigger
ALTER TRIGGER trg_auditoria_produtos DISABLE;

-- Habilitar trigger
ALTER TRIGGER trg_auditoria_produtos ENABLE;

-- Deletar trigger
DROP TRIGGER trg_auditoria_produtos;
```

---

## Dicas de Performance

### 1. Use índices estrategicamente
```sql
-- BOM: Índice em colunas de busca frequente
CREATE INDEX idx_clientes_cpf ON CLIENTES(CPF);

-- EVITE: Índice em tabelas pequenas ou colunas com poucos valores distintos
-- Exemplo: não crie índice em coluna SEXO (apenas 'M' e 'F')
```

### 2. Evite SELECT *
```sql
-- RUIM
SELECT * FROM CLIENTES;

-- BOM
SELECT ID_CLIENTE, NOME, EMAIL FROM CLIENTES;
```

### 3. Use EXPLAIN PLAN
```sql
EXPLAIN PLAN FOR
SELECT * FROM PRODUTOS WHERE PRECO > 100;

SELECT * FROM TABLE(DBMS_XPLAN.DISPLAY);
```

### 4. Use BIND VARIABLES em procedures
```sql
-- RUIM (SQL diferente cada vez)
EXECUTE IMMEDIATE 'SELECT * FROM PRODUTOS WHERE ID = ' || p_id;

-- BOM (SQL reutilizável)
OPEN cursor FOR 'SELECT * FROM PRODUTOS WHERE ID = :id' USING p_id;
```

---

## Comandos de Administração

```sql
-- Ver todas as tabelas do usuário
SELECT TABLE_NAME FROM USER_TABLES;

-- Ver estrutura de uma tabela
DESC PRODUTOS;

-- Ver espaço usado pelas tabelas
SELECT 
    SEGMENT_NAME,
    ROUND(BYTES/1024/1024, 2) AS MB
FROM USER_SEGMENTS
WHERE SEGMENT_TYPE = 'TABLE'
ORDER BY BYTES DESC;

-- Ver usuários conectados (requer privilégio)
SELECT USERNAME, STATUS, MACHINE 
FROM V$SESSION 
WHERE USERNAME IS NOT NULL;

-- Fazer backup de uma tabela
CREATE TABLE PRODUTOS_BACKUP AS SELECT * FROM PRODUTOS;

-- Ver locks/bloqueios
SELECT * FROM V$LOCKED_OBJECT;

-- Matar sessão (requer privilégio DBA)
ALTER SYSTEM KILL SESSION 'sid,serial#';
```

---

## Tratamento de Erros em PL/SQL

```sql
CREATE OR REPLACE PROCEDURE sp_exemplo_erros (
    p_id_produto IN NUMBER
) IS
    v_preco NUMBER;
    v_estoque NUMBER;
    
    -- Exceções customizadas
    estoque_insuficiente EXCEPTION;
    produto_inexistente EXCEPTION;
BEGIN
    -- Tentar buscar produto
    BEGIN
        SELECT PRECO, ESTOQUE 
        INTO v_preco, v_estoque
        FROM PRODUTOS
        WHERE ID_PRODUTO = p_id_produto;
        
    EXCEPTION
        WHEN NO_DATA_FOUND THEN
            RAISE produto_inexistente;
    END;
    
    -- Validar estoque
    IF v_estoque < 1 THEN
        RAISE estoque_insuficiente;
    END IF;
    
    -- Processar...
    DBMS_OUTPUT.PUT_LINE('Produto processado com sucesso!');
    
EXCEPTION
    WHEN produto_inexistente THEN
        DBMS_OUTPUT.PUT_LINE('ERRO: Produto não encontrado!');
        RAISE_APPLICATION_ERROR(-20001, 'Produto ID ' || p_id_produto || ' não existe');
        
    WHEN estoque_insuficiente THEN
        DBMS_OUTPUT.PUT_LINE('ERRO: Estoque insuficiente!');
        RAISE_APPLICATION_ERROR(-20002, 'Produto sem estoque');
        
    WHEN OTHERS THEN
        DBMS_OUTPUT.PUT_LINE('ERRO: ' || SQLERRM);
        ROLLBACK;
        RAISE;
END;
/
```

---

## Transações e ACID

**ACID** = Propriedades de uma transação:
- **A**tomicity (Atomicidade): Tudo ou nada
- **C**onsistency (Consistência): Dados sempre válidos
- **I**solation (Isolamento): Transações não interferem entre si
- **D**urability (Durabilidade): Dados persistem após COMMIT

```sql
-- Exemplo de transação completa
BEGIN
    -- Debitar da conta origem
    UPDATE CONTAS 
    SET SALDO = SALDO - 100 
    WHERE ID_CONTA = 1;
    
    -- Verificar saldo
    IF SQL%ROWCOUNT = 0 THEN
        ROLLBACK;
        RAISE_APPLICATION_ERROR(-20001, 'Conta inválida');
    END IF;
    
    -- Creditar na conta destino
    UPDATE CONTAS 
    SET SALDO = SALDO + 100 
    WHERE ID_CONTA = 2;
    
    -- Registrar histórico
    INSERT INTO HISTORICO_TRANSFERENCIAS 
    VALUES (SYSDATE, 1, 2, 100);
    
    -- Confirmar tudo
    COMMIT;
    
EXCEPTION
    WHEN OTHERS THEN
        ROLLBACK;  -- Desfaz tudo em caso de erro
        RAISE;
END;
/
```

---

## Níveis de Isolamento

```sql
-- Ler dados sem lock (pode ler dados inconsistentes)
SET TRANSACTION READ ONLY;

-- Ler com lock (dados consistentes)
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;

-- Lock explícito em linha
SELECT * FROM PRODUTOS WHERE ID = 1 FOR UPDATE;

-- Esperar ou falhar imediatamente se houver lock
SELECT * FROM PRODUTOS WHERE ID = 1 FOR UPDATE NOWAIT;
```

---

## Referências

Baseado nos materiais:
1. Desvendando o Mundo dos Bancos de Dados.pdf
2. Script Inicial_Avancado2.txt
3. [APO-LB]06 - Views.docx
4. [TEO-BD]Exercicio4_GeneralizaçãoEsp.docx
5. JOIN no Oracle.docx
6. [APO-TE]O4- Generalização e Especialização.docx
7. Script Criacao Padrao.txt

---

## 🎯 Checklist de Estudo

- [ ] Criar tabelas com constraints
- [ ] Fazer INSERT, UPDATE, DELETE
- [ ] Praticar JOINs (INNER, LEFT, RIGHT)
- [ ] Criar VIEWs simples
- [ ] Criar VIEWs Materializadas
- [ ] Criar índices
- [ ] Escrever PROCEDUREs simples
- [ ] Usar TRIGGERs para auditoria
- [ ] Praticar subconsultas
- [ ] Usar funções de data/texto/número
- [ ] Implementar tratamento de erros
- [ ] Usar transações (COMMIT/ROLLBACK)
- [ ] Criar SEQUENCEs para auto-incremento
- [ ] Modelar com Generalização/Especialização

---

## 💡 Dicas Finais

1. **Pratique muito!** Crie um banco de teste e experimente
2. **Sempre use COMMIT ou ROLLBACK** após modificar dados
3. **Faça backup** antes de comandos perigosos (DROP, TRUNCATE)
4. **Use DESC e USER_TABLES** para explorar a estrutura
5. **Teste em ambiente de desenvolvimento** antes de produção
6. **Documente** suas procedures e triggers
7. **Use nomes descritivos** para tabelas, colunas e índices
8. **Normalize** suas tabelas (evite redundância)
9. **Crie índices** apenas quando necessário
10. **Monitore performance** com EXPLAIN PLAN

---

**Licença:** MIT  
**Contribuições:** Bem-vindas via Pull Request  
**Dúvidas:** Abra uma Issue no GitHub
