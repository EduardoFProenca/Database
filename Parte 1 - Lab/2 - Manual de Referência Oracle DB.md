# 🐘 Manual de Referência Oracle DB
> Guia detalhado de SQL e PL/SQL — do básico ao intermediário.

---

## 📋 Índice
1. [Tipos de Dados](#1-tipos-de-dados)
2. [Comandos de Ambiente](#2-comandos-de-ambiente)
3. [DDL — Definição de Dados](#3-ddl--definição-de-dados)
4. [DML — Manipulação de Dados](#4-dml--manipulação-de-dados)
5. [TCL — Controle de Transação](#5-tcl--controle-de-transação)
6. [SELECT — Consultas](#6-select--consultas)
7. [Funções SQL](#7-funções-sql)
8. [Joins — Junções](#8-joins--junções)
9. [PL/SQL Básico](#9-plsql-básico)
10. [Views, Sequences e Índices](#10-views-sequences-e-índices)
11. [Dicas e Referência Rápida](#11-dicas-e-referência-rápida)

---

## 1. Tipos de Dados

| Tipo | Descrição |
|------|-----------|
| `VARCHAR2(n)` | Texto variável — até 4000 bytes. Tipo preferido para strings no Oracle. |
| `CHAR(n)` | Texto fixo — sempre ocupa n bytes. Útil para códigos padronizados (ex: `CHAR(1)` para flags S/N). |
| `NUMBER(p,s)` | Número com `p` dígitos totais e `s` casas decimais. Ex: `NUMBER(10,2)` = até 99999999.99 |
| `DATE` | Data + hora (hora, minuto, segundo). **Não** inclui fuso horário. |
| `TIMESTAMP` | Data + hora com frações de segundo. Maior precisão que `DATE`. |
| `CLOB` | Texto longo — até 4 GB. Use para textos grandes (logs, XMLs, JSON). |
| `BLOB` | Dados binários — até 4 GB. Use para imagens, PDFs, arquivos. |
| `BOOLEAN` | Apenas em PL/SQL (`TRUE` / `FALSE` / `NULL`). **Não existe** como tipo de coluna no SQL. |

> **Dica:** prefira `VARCHAR2` a `CHAR` para evitar espaços em branco desnecessários. Use `NUMBER` sem escala para inteiros e `NUMBER(p,s)` para valores monetários.

---

## 2. Comandos de Ambiente

Comandos usados no terminal **SQL\*Plus** ou **SQLcl** para configurar a sessão.
```sql
-- Exibir o usuário conectado no momento
SHOW USER;

-- Limpar a tela do terminal
CLEAR SCREEN;

-- Exibir a estrutura de uma tabela (colunas, tipos, nulabilidade)
DESC nome_da_tabela;
DESCRIBE nome_da_tabela; -- versão completa, mesmo resultado

-- Configurações de exibição
SET LINESIZE 200;       -- largura da linha de output (evita quebras indesejadas)
SET PAGESIZE 50;        -- quantas linhas por página antes do cabeçalho repetir
SET SERVEROUTPUT ON;    -- habilita o DBMS_OUTPUT.PUT_LINE (essencial para PL/SQL)
SET TIMING ON;          -- mostra o tempo de execução de cada comando

-- Executar o conteúdo do buffer SQL
RUN;  -- ou simplesmente /

-- Executar um arquivo de script externo
@caminho/do/arquivo.sql
START caminho/do/arquivo.sql

-- Salvar a query atual em arquivo
SAVE minha_query.sql

-- Carregar um arquivo no buffer
GET minha_query.sql
```

### Consultando o Dicionário de Dados

O Oracle mantém views de sistema para inspecionar objetos. Os prefixos são:
- `USER_` → objetos do usuário atual
- `ALL_`  → objetos acessíveis ao usuário
- `DBA_`  → todos os objetos (requer privilégio de DBA)
```sql
-- Listar tabelas do usuário
SELECT table_name FROM user_tables ORDER BY table_name;

-- Listar colunas de uma tabela específica
SELECT column_name, data_type, data_length, nullable
FROM user_tab_columns
WHERE table_name = 'FUNCIONARIOS'; -- nome em MAIÚSCULO

-- Listar constraints (PKs, FKs, UNIQUEs, CHECKs)
SELECT constraint_name, constraint_type, status
FROM user_constraints
WHERE table_name = 'FUNCIONARIOS';

-- Listar índices
SELECT index_name, index_type, status
FROM user_indexes
WHERE table_name = 'FUNCIONARIOS';

-- Listar sequences
SELECT sequence_name, last_number, increment_by
FROM user_sequences;

-- Listar procedures, functions e packages
SELECT object_name, object_type, status
FROM user_objects
WHERE object_type IN ('PROCEDURE', 'FUNCTION', 'PACKAGE');
```

---

## 3. DDL — Definição de Dados

DDL (Data Definition Language) cria, altera e remove **estruturas** do banco.

> ⚠️ **Atenção:** todo comando DDL executa um **COMMIT automático e implícito**. Qualquer DML pendente na sessão será confirmado automaticamente. DDL não pode ser desfeito com `ROLLBACK`.

### CREATE TABLE
```sql
-- Criação completa com constraints inline
CREATE TABLE funcionarios (
    id            NUMBER         GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    nome          VARCHAR2(100)  NOT NULL,
    email         VARCHAR2(150)  UNIQUE,
    salario       NUMBER(10, 2)  CHECK (salario > 0),
    ativo         CHAR(1)        DEFAULT 'S' CHECK (ativo IN ('S', 'N')),
    data_admissao DATE           DEFAULT SYSDATE,
    dept_id       NUMBER,
    -- constraint de chave estrangeira com nome explícito (boa prática)
    CONSTRAINT fk_func_dept
        FOREIGN KEY (dept_id) REFERENCES departamentos(id)
        ON DELETE SET NULL  -- ao deletar o dept, seta dept_id como NULL no funcionário
);

-- Criar tabela a partir de uma query (sem dados — WHERE impossível)
CREATE TABLE historico_func AS
SELECT * FROM funcionarios WHERE 1 = 0;

-- Criar tabela com dados copiados
CREATE TABLE backup_func AS
SELECT * FROM funcionarios;
```

### ALTER TABLE
```sql
-- Adicionar coluna
ALTER TABLE funcionarios ADD (telefone VARCHAR2(20));

-- Adicionar múltiplas colunas de uma vez
ALTER TABLE funcionarios ADD (
    linkedin      VARCHAR2(200),
    dt_nascimento DATE
);

-- Modificar tipo ou tamanho de coluna existente
ALTER TABLE funcionarios MODIFY (nome VARCHAR2(150));

-- Definir valor DEFAULT em coluna existente
ALTER TABLE funcionarios MODIFY (ativo DEFAULT 'S');

-- Tornar coluna NOT NULL
ALTER TABLE funcionarios MODIFY (email NOT NULL);

-- Remover coluna
ALTER TABLE funcionarios DROP COLUMN linkedin;

-- Renomear coluna
ALTER TABLE funcionarios RENAME COLUMN nome TO nome_completo;

-- Adicionar constraint de chave estrangeira depois da criação
ALTER TABLE funcionarios
ADD CONSTRAINT fk_dept FOREIGN KEY (dept_id) REFERENCES departamentos(id);

-- Desabilitar constraint temporariamente (útil para carga de dados em massa)
ALTER TABLE funcionarios DISABLE CONSTRAINT fk_dept;
ALTER TABLE funcionarios ENABLE  CONSTRAINT fk_dept;

-- Remover constraint
ALTER TABLE funcionarios DROP CONSTRAINT fk_dept;
```

### DROP, RENAME e TRUNCATE
```sql
-- Remover tabela permanentemente (cuidado — sem ROLLBACK sem Flashback)
DROP TABLE funcionarios;

-- Remover sem passar pela Recycle Bin (mais rápido, sem recuperação)
DROP TABLE funcionarios PURGE;

-- Recuperar tabela da Recycle Bin (se não usou PURGE)
FLASHBACK TABLE funcionarios TO BEFORE DROP;

-- Renomear tabela
RENAME funcionarios TO colaboradores;

-- TRUNCATE: apaga todos os dados rapidamente, reseta o High Water Mark
-- Não pode ser desfeito com ROLLBACK
TRUNCATE TABLE funcionarios;
```

> **TRUNCATE vs DELETE:** `TRUNCATE` é DDL — muito mais rápido para apagar tudo, pois não gera registros de undo/redo linha a linha. `DELETE` é DML — mais lento em grandes volumes, mas pode ser desfeito com `ROLLBACK`.

---

## 4. DML — Manipulação de Dados

DML (Data Manipulation Language) insere, atualiza e deleta **dados**. Faz parte de uma transação e pode ser desfeito com `ROLLBACK` antes do `COMMIT`.

### INSERT
```sql
-- Inserção explícita — especifique sempre as colunas (boa prática)
INSERT INTO funcionarios (nome, email, salario, dept_id)
VALUES ('João Silva', 'joao@empresa.com', 5000.00, 10);

-- Inserção com subquery — copia dados de outra tabela
INSERT INTO historico_salarios (func_id, salario, data_registro)
SELECT id, salario, SYSDATE
FROM funcionarios
WHERE dept_id = 10;

-- INSERT ALL — inserir em múltiplas tabelas de uma só vez
INSERT ALL
    INTO log_insercao (tabela, dt) VALUES ('funcionarios', SYSDATE)
    INTO auditoria    (acao,  dt) VALUES ('INSERT', SYSDATE)
SELECT 1 FROM dual;
```

### UPDATE
```sql
-- Atualizar um registro específico
UPDATE funcionarios
SET salario = 5500.00,
    nome    = 'João Silva Jr.'
WHERE id = 1;

-- Atualizar múltiplos registros com expressão
UPDATE funcionarios
SET salario = salario * 1.10  -- aumento de 10%
WHERE dept_id = 10 AND ativo = 'S';

-- UPDATE com subquery — define o novo valor a partir de outra consulta
UPDATE funcionarios f
SET f.salario = (
    SELECT AVG(salario)
    FROM funcionarios
    WHERE dept_id = f.dept_id
)
WHERE f.cargo = 'ANALISTA';
```

### DELETE
```sql
-- Deletar registro específico
DELETE FROM funcionarios WHERE id = 1;

-- Deletar com condição baseada em outra tabela
DELETE FROM funcionarios
WHERE dept_id NOT IN (SELECT id FROM departamentos);

-- Deletar tudo (prefira TRUNCATE para grandes volumes)
DELETE FROM funcionarios;
```

### MERGE (Upsert)

O `MERGE` insere ou atualiza dependendo se o registro já existe — muito útil para sincronização de dados.
```sql
MERGE INTO funcionarios tgt
USING (SELECT 99 AS id, 'Novo Nome' AS nome, 7000 AS salario FROM dual) src
ON (tgt.id = src.id)
WHEN MATCHED THEN
    UPDATE SET tgt.nome = src.nome, tgt.salario = src.salario
WHEN NOT MATCHED THEN
    INSERT (id, nome, salario) VALUES (src.id, src.nome, src.salario);
```

---

## 5. TCL — Controle de Transação

TCL (Transaction Control Language) gerencia as transações. Uma transação é iniciada automaticamente pelo primeiro DML e encerrada com `COMMIT` ou `ROLLBACK`.
```sql
-- Confirmar permanentemente todas as alterações da transação
COMMIT;

-- Desfazer todas as alterações desde o último COMMIT
ROLLBACK;

-- Criar um ponto de salvamento dentro da transação
SAVEPOINT antes_aumento;

-- Desfazer somente até o savepoint (o restante é mantido)
ROLLBACK TO SAVEPOINT antes_aumento;
```

**Exemplo prático com SAVEPOINT:**
```sql
-- Atualiza o depto 10
UPDATE funcionarios SET salario = salario * 1.05 WHERE dept_id = 10;
SAVEPOINT dept10_ok;

-- Tenta atualizar o depto 20
UPDATE funcionarios SET salario = salario * 1.08 WHERE dept_id = 20;

-- Ops! Erro identificado no depto 20 — desfaz só essa parte
ROLLBACK TO SAVEPOINT dept10_ok;

-- Confirma somente a atualização do depto 10
COMMIT;
```

> **Dica:** DDL (`CREATE`, `ALTER`, `DROP`, `TRUNCATE`) executa um `COMMIT` implícito — toda a transação atual é confirmada automaticamente.

---

## 6. SELECT — Consultas

### Estrutura e Ordem de Execução
```sql
-- Ordem de ESCRITA:
SELECT   colunas ou expressões
FROM     tabela
WHERE    filtro de linhas
GROUP BY agrupamento
HAVING   filtro de grupos
ORDER BY ordenação;

-- Ordem de EXECUÇÃO pelo banco (importante para entender erros):
-- FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY
```

### Filtros e Operadores
```sql
-- DISTINCT — elimina linhas duplicadas no resultado
SELECT DISTINCT dept_id FROM funcionarios;

-- BETWEEN — intervalo inclusivo nos dois extremos
SELECT * FROM funcionarios WHERE salario BETWEEN 3000 AND 7000;

-- IN — lista de valores aceitos
SELECT * FROM funcionarios WHERE dept_id IN (10, 20, 30);

-- NOT IN — valores excluídos
SELECT * FROM funcionarios WHERE dept_id NOT IN (10, 20);

-- LIKE — padrão de texto
--   % = qualquer sequência de caracteres (inclusive vazia)
--   _ = exatamente um caractere qualquer
SELECT * FROM funcionarios WHERE nome LIKE 'Jo%';    -- começa com "Jo"
SELECT * FROM funcionarios WHERE nome LIKE '%ão';    -- termina com "ão"
SELECT * FROM funcionarios WHERE nome LIKE '%ilv%';  -- contém "ilv"
SELECT * FROM funcionarios WHERE nome LIKE 'J_ão';   -- J + 1 char + "ão"

-- IS NULL / IS NOT NULL — único jeito correto de checar nulos
SELECT * FROM funcionarios WHERE email IS NULL;
SELECT * FROM funcionarios WHERE email IS NOT NULL;

-- Combinando com AND, OR, NOT
SELECT * FROM funcionarios
WHERE (dept_id = 10 OR dept_id = 20)
  AND salario > 5000
  AND nome IS NOT NULL;
```

### Agrupamento — GROUP BY e HAVING
```sql
-- Funções de agregação mais usadas
SELECT
    dept_id,
    COUNT(*)      AS total_funcionarios,   -- conta todas as linhas
    COUNT(email)  AS com_email,            -- COUNT ignora valores NULL
    SUM(salario)  AS folha_total,
    AVG(salario)  AS salario_medio,
    MIN(salario)  AS menor_salario,
    MAX(salario)  AS maior_salario
FROM funcionarios
GROUP BY dept_id
HAVING AVG(salario) > 5000   -- HAVING filtra GRUPOS (depois do agrupamento)
ORDER BY folha_total DESC;

-- Diferença entre WHERE e HAVING:
-- WHERE filtra linhas ANTES do agrupamento
-- HAVING filtra grupos DEPOIS do agrupamento

SELECT dept_id, COUNT(*), AVG(salario)
FROM funcionarios
WHERE ativo = 'S'          -- primeiro filtra só ativos
GROUP BY dept_id
HAVING COUNT(*) >= 3;      -- depois só grupos com 3 ou mais funcionários
```

### Subconsultas (Subqueries)
```sql
-- Subquery escalar no WHERE — retorna um único valor
SELECT nome, salario
FROM funcionarios
WHERE salario > (SELECT AVG(salario) FROM funcionarios);

-- Subquery com IN
SELECT nome FROM funcionarios
WHERE dept_id IN (
    SELECT id FROM departamentos WHERE uf = 'SP'
);

-- EXISTS — mais eficiente que IN para grandes volumes
-- Retorna TRUE assim que encontra a primeira linha que satisfaz a condição
SELECT d.nome_dept
FROM departamentos d
WHERE EXISTS (
    SELECT 1 FROM funcionarios f WHERE f.dept_id = d.id
);

-- CTE — Common Table Expression (WITH)
-- Deixa a query mais legível, especialmente quando reutiliza o resultado
WITH media_por_dept AS (
    SELECT dept_id, AVG(salario) AS media
    FROM funcionarios
    GROUP BY dept_id
)
SELECT f.nome, f.salario, m.media
FROM funcionarios f
JOIN media_por_dept m ON f.dept_id = m.dept_id
WHERE f.salario > m.media
ORDER BY f.salario DESC;
```

### Ordenação
```sql
-- Ordenar por coluna simples
SELECT * FROM funcionarios ORDER BY nome ASC;   -- ASC é o padrão
SELECT * FROM funcionarios ORDER BY salario DESC;

-- Ordenar por múltiplas colunas
SELECT * FROM funcionarios
ORDER BY dept_id ASC, salario DESC;

-- Ordenar por posição da coluna no SELECT (evite em produção — frágil)
SELECT nome, salario FROM funcionarios ORDER BY 2 DESC;

-- NULLS FIRST / NULLS LAST — controla onde os NULLs aparecem
SELECT nome, email FROM funcionarios
ORDER BY email NULLS LAST;
```

---

## 7. Funções SQL

### Funções de Texto
```sql
-- Concatenar com operador || (preferido no Oracle)
SELECT nome || ' — ' || cargo AS descricao FROM funcionarios;

-- Maiúscula, minúscula e capitalize
SELECT UPPER(nome)       FROM funcionarios;  -- 'JOÃO SILVA'
SELECT LOWER(email)      FROM funcionarios;  -- 'joao@email.com'
SELECT INITCAP(nome)     FROM funcionarios;  -- 'João Silva'

-- Tamanho da string em caracteres
SELECT nome, LENGTH(nome) AS tamanho FROM funcionarios;

-- Extrair parte do texto: SUBSTR(string, posição_inicial, comprimento)
-- Posição começa em 1. Valor negativo começa pelo fim.
SELECT SUBSTR(nome, 1, 5)  FROM funcionarios;  -- 5 primeiros chars
SELECT SUBSTR(nome, -3)    FROM funcionarios;  -- últimos 3 chars

-- Localizar posição de uma substring: INSTR(string, substring)
SELECT INSTR(email, '@') AS pos_arroba FROM funcionarios;
-- Retorna 0 se não encontrar

-- Substituir texto
SELECT REPLACE(nome, 'Silva', 'Oliveira') FROM funcionarios;

-- Remover espaços (ou outros caracteres)
SELECT TRIM('  Oracle  ')           FROM dual;  -- 'Oracle'
SELECT LTRIM('  Oracle  ')          FROM dual;  -- 'Oracle  '
SELECT RTRIM('  Oracle  ')          FROM dual;  -- '  Oracle'
SELECT TRIM('.' FROM '...texto...') FROM dual;  -- 'texto'

-- Preencher com caracteres: LPAD e RPAD (pad = preencher)
SELECT LPAD(id, 6, '0') FROM funcionarios;     -- '000001', '000042'
SELECT RPAD(nome, 30, '.') FROM funcionarios;  -- 'João.....................'
```

### Funções Numéricas
```sql
-- ROUND — arredonda para n casas decimais
SELECT ROUND(1234.567,  2) FROM dual;  -- 1234.57
SELECT ROUND(1234.567,  0) FROM dual;  -- 1235
SELECT ROUND(1234.567, -1) FROM dual;  -- 1230 (arredonda para a dezena)

-- TRUNC — trunca sem arredondar
SELECT TRUNC(1234.567,  2) FROM dual;  -- 1234.56
SELECT TRUNC(1234.567, -2) FROM dual;  -- 1200

-- CEIL e FLOOR — teto e piso
SELECT CEIL(4.1)  FROM dual;  -- 5 (sempre arredonda para cima)
SELECT FLOOR(4.9) FROM dual;  -- 4 (sempre arredonda para baixo)

-- MOD — resto da divisão (útil para identificar pares/ímpares)
SELECT MOD(10, 3) FROM dual;  -- 1
SELECT MOD(id, 2) FROM funcionarios;  -- 0 = par, 1 = ímpar

-- ABS — valor absoluto
SELECT ABS(-250) FROM dual;  -- 250

-- POWER e SQRT
SELECT POWER(2, 8) FROM dual;  -- 256
SELECT SQRT(81)    FROM dual;  -- 9
```

### Funções de Data
```sql
-- Data e hora atuais
SELECT SYSDATE      FROM dual;  -- data + hora do servidor Oracle
SELECT CURRENT_DATE FROM dual;  -- data + hora da sessão do cliente

-- Aritmética com datas (números = dias)
SELECT SYSDATE + 30               FROM dual;  -- 30 dias no futuro
SELECT SYSDATE - 7                FROM dual;  -- 7 dias atrás
SELECT SYSDATE - data_admissao    FROM funcionarios;  -- dias de empresa (NUMBER)

-- ADD_MONTHS — adicionar ou subtrair meses
SELECT ADD_MONTHS(SYSDATE,  6) FROM dual;  -- 6 meses no futuro
SELECT ADD_MONTHS(SYSDATE, -3) FROM dual;  -- 3 meses atrás

-- MONTHS_BETWEEN — diferença em meses entre duas datas
SELECT MONTHS_BETWEEN(SYSDATE, data_admissao) FROM funcionarios;

-- LAST_DAY — último dia do mês da data informada
SELECT LAST_DAY(SYSDATE)             FROM dual;  -- último dia deste mês
SELECT LAST_DAY(ADD_MONTHS(SYSDATE,1)) FROM dual;  -- último do próximo mês

-- NEXT_DAY — próxima ocorrência de um dia da semana
SELECT NEXT_DAY(SYSDATE, 'MONDAY')  FROM dual;  -- próxima segunda-feira
SELECT NEXT_DAY(SYSDATE, 'FRIDAY')  FROM dual;  -- próxima sexta-feira

-- TRUNC de datas — "zera" partes da data
SELECT TRUNC(SYSDATE)         FROM dual;  -- zera a hora (00:00:00)
SELECT TRUNC(SYSDATE, 'MM')   FROM dual;  -- primeiro dia do mês atual
SELECT TRUNC(SYSDATE, 'YYYY') FROM dual;  -- 1º de janeiro do ano atual

-- EXTRACT — extrai parte específica da data
SELECT EXTRACT(YEAR  FROM SYSDATE) FROM dual;
SELECT EXTRACT(MONTH FROM SYSDATE) FROM dual;
SELECT EXTRACT(DAY   FROM SYSDATE) FROM dual;
```

### Funções de Conversão
```sql
-- TO_CHAR — converte DATE ou NUMBER para VARCHAR2
SELECT TO_CHAR(SYSDATE, 'DD/MM/YYYY')            FROM dual;  -- '19/02/2026'
SELECT TO_CHAR(SYSDATE, 'DD/MM/YYYY HH24:MI:SS') FROM dual;  -- com hora
SELECT TO_CHAR(SYSDATE, 'Day, DD de Month')       FROM dual;  -- 'Thursday, 19 de February'

-- Formatos numéricos com TO_CHAR
-- 9 = dígito opcional   0 = dígito obrigatório (com zero à esquerda)
-- G = separador de milhar    D = separador decimal    L = moeda local
SELECT TO_CHAR(9876543.21, '9,999,999.99')  FROM dual;  -- '9,876,543.21'
SELECT TO_CHAR(salario, 'FM9999990.00')     FROM funcionarios;

-- TO_DATE — converte VARCHAR2 para DATE
SELECT TO_DATE('01/01/2024', 'DD/MM/YYYY')                   FROM dual;
SELECT TO_DATE('2024-12-31 23:59:59', 'YYYY-MM-DD HH24:MI:SS') FROM dual;

-- TO_NUMBER — converte VARCHAR2 para NUMBER
SELECT TO_NUMBER('1.200,50', '9G999D99') FROM dual;

-- CAST — conversão padrão SQL
SELECT CAST(salario AS VARCHAR2(20)) FROM funcionarios;
SELECT CAST('2024-01-15' AS DATE)    FROM dual;
```

### Funções Condicionais
```sql
-- NVL — substitui NULL por um valor padrão
SELECT nome, NVL(comissao, 0) AS comissao FROM funcionarios;

-- NVL2(expr, valor_se_nao_null, valor_se_null)
SELECT nome, NVL2(comissao, 'Tem comissão', 'Sem comissão') FROM funcionarios;

-- COALESCE — retorna o primeiro valor não-nulo da lista (preferido ao NVL)
SELECT COALESCE(celular, telefone, email, 'Sem contato') AS contato
FROM clientes;

-- NULLIF — retorna NULL se os dois valores forem iguais (evita divisão por zero)
SELECT total_vendas / NULLIF(dias_trabalhados, 0) AS media_diaria
FROM metas;

-- DECODE — switch-case simplificado (exclusivo Oracle)
SELECT nome,
    DECODE(dept_id,
        10, 'Tecnologia',
        20, 'Financeiro',
        30, 'RH',
            'Outros'   -- valor padrão
    ) AS departamento
FROM funcionarios;

-- CASE WHEN — expressão condicional padrão SQL (preferida ao DECODE)
SELECT nome, salario,
    CASE
        WHEN salario <  3000              THEN 'Júnior'
        WHEN salario BETWEEN 3000 AND 6000 THEN 'Pleno'
        WHEN salario BETWEEN 6001 AND 10000 THEN 'Sênior'
        ELSE 'Especialista'
    END AS nivel
FROM funcionarios;

-- CASE de igualdade (forma compacta)
SELECT CASE status
    WHEN 'A' THEN 'Ativo'
    WHEN 'I' THEN 'Inativo'
    ELSE 'Indefinido'
END AS situacao
FROM usuarios;
```

---

## 8. Joins — Junções

Joins combinam dados de duas ou mais tabelas com base em uma condição de relacionamento.
```sql
-- INNER JOIN — apenas linhas que têm correspondência em AMBAS as tabelas
SELECT f.nome, d.nome_dept
FROM funcionarios f
INNER JOIN departamentos d ON f.dept_id = d.id;
-- Funcionários sem departamento e departamentos sem funcionários são excluídos

-- LEFT JOIN — TODOS os registros da tabela da ESQUERDA
-- Quando não há correspondência na direita, as colunas direitas vêm como NULL
SELECT f.nome, d.nome_dept
FROM funcionarios f
LEFT JOIN departamentos d ON f.dept_id = d.id;
-- Funcionários sem departamento aparecem com nome_dept = NULL

-- RIGHT JOIN — TODOS os registros da tabela da DIREITA
SELECT f.nome, d.nome_dept
FROM funcionarios f
RIGHT JOIN departamentos d ON f.dept_id = d.id;
-- Departamentos sem funcionários aparecem com nome = NULL

-- FULL OUTER JOIN — TODOS os registros de AMBAS as tabelas
SELECT f.nome, d.nome_dept
FROM funcionarios f
FULL OUTER JOIN departamentos d ON f.dept_id = d.id;
-- Combina LEFT + RIGHT: nenhuma linha de nenhuma tabela é perdida

-- CROSS JOIN — produto cartesiano (todas as combinações possíveis)
-- N linhas de funcionarios × M linhas de departamentos = N×M resultados
SELECT f.nome, d.nome_dept
FROM funcionarios f
CROSS JOIN departamentos d;

-- SELF JOIN — a tabela se junta com ela mesma
-- Muito útil para hierarquias (funcionário → gerente)
SELECT e.nome AS funcionario, g.nome AS gerente
FROM funcionarios e
LEFT JOIN funcionarios g ON e.gerente_id = g.id;

-- JOIN com múltiplas condições
SELECT f.nome, p.descricao AS projeto
FROM funcionarios f
JOIN alocacoes a ON f.id = a.func_id AND a.ativo = 'S'
JOIN projetos   p ON a.proj_id = p.id AND p.status = 'EM_ANDAMENTO';
```

> **Dica:** sempre prefira a sintaxe `JOIN ... ON` (ANSI SQL) à sintaxe antiga `FROM tabela1, tabela2 WHERE ...`. A sintaxe ANSI é mais legível, menos propensa a erros e facilita identificar o tipo de join.

---

## 9. PL/SQL Básico

PL/SQL é a extensão procedural do SQL no Oracle. Permite criar lógica de programação diretamente no banco.

### Bloco Anônimo — Estrutura
```sql
DECLARE
    -- Seção de declaração de variáveis (opcional)
    v_nome     VARCHAR2(100);
    v_salario  NUMBER(10,2)  := 0;       -- := é o operador de atribuição
    v_data     DATE          := SYSDATE;
    c_taxa     CONSTANT NUMBER := 0.15;  -- constante (não pode ser alterada no BEGIN)

    -- %TYPE — herda o tipo de uma coluna (mais seguro que hardcodar o tipo)
    v_email    funcionarios.email%TYPE;

    -- %ROWTYPE — herda a estrutura inteira de uma tabela
    v_func     funcionarios%ROWTYPE;

BEGIN
    -- Corpo principal: lógica e queries

    -- SELECT INTO — traz resultado para variáveis PL/SQL
    -- OBRIGATÓRIO retornar EXATAMENTE uma linha (zero = NO_DATA_FOUND, 2+ = TOO_MANY_ROWS)
    SELECT nome, salario
    INTO   v_nome, v_salario
    FROM   funcionarios
    WHERE  id = 1;

    v_salario := v_salario * (1 + c_taxa);

    -- Exibir na tela (requer SET SERVEROUTPUT ON)
    DBMS_OUTPUT.PUT_LINE('Funcionário: ' || v_nome);
    DBMS_OUTPUT.PUT_LINE('Salário: ' || TO_CHAR(v_salario, 'FM9999990.99'));

EXCEPTION
    -- Tratamento de erros (opcional, mas recomendado)
    WHEN NO_DATA_FOUND THEN
        DBMS_OUTPUT.PUT_LINE('Erro: funcionário não encontrado.');
    WHEN TOO_MANY_ROWS THEN
        DBMS_OUTPUT.PUT_LINE('Erro: mais de uma linha retornada pelo SELECT INTO.');
    WHEN OTHERS THEN
        -- SQLERRM retorna a mensagem do erro, SQLCODE retorna o código (negativo)
        DBMS_OUTPUT.PUT_LINE('Erro: ' || SQLERRM);
        RAISE; -- re-lança a exceção para quem chamou
END;
/  -- o / executa o bloco no SQL*Plus/SQLcl
```

### IF / ELSIF / ELSE
```sql
IF v_salario > 10000 THEN
    v_bonus := 1500;
ELSIF v_salario > 5000 THEN
    v_bonus := 800;
ELSIF v_salario > 3000 THEN
    v_bonus := 400;
ELSE
    v_bonus := 0;
END IF;
```

### Loops
```sql
-- LOOP simples — use EXIT WHEN para sair
DECLARE v_i NUMBER := 1;
BEGIN
    LOOP
        DBMS_OUTPUT.PUT_LINE('i = ' || v_i);
        v_i := v_i + 1;
        EXIT WHEN v_i > 10;
    END LOOP;
END;
/

-- WHILE LOOP — verifica a condição antes de cada iteração
DECLARE v_i NUMBER := 1;
BEGIN
    WHILE v_i <= 10 LOOP
        DBMS_OUTPUT.PUT_LINE('i = ' || v_i);
        v_i := v_i + 1;
    END LOOP;
END;
/

-- FOR LOOP — mais simples quando se conhece o número de iterações
-- A variável de controle (i) é criada e destruída automaticamente
BEGIN
    FOR i IN 1..10 LOOP
        DBMS_OUTPUT.PUT_LINE('Iteração: ' || i);
    END LOOP;

    -- Reverso:
    FOR i IN REVERSE 1..5 LOOP
        DBMS_OUTPUT.PUT_LINE(i);  -- imprime 5, 4, 3, 2, 1
    END LOOP;
END;
/
```

### Cursores Explícitos

Usados quando um `SELECT` retorna múltiplas linhas.
```sql
DECLARE
    -- 1. Declarar o cursor com a query
    CURSOR c_funcionarios IS
        SELECT id, nome, salario
        FROM   funcionarios
        WHERE  ativo = 'S'
        ORDER  BY nome;

    v_reg c_funcionarios%ROWTYPE;  -- variável para armazenar cada linha

BEGIN
    OPEN c_funcionarios;  -- 2. Abrir: executa a query

    LOOP
        FETCH c_funcionarios INTO v_reg;  -- 3. Buscar próxima linha
        EXIT WHEN c_funcionarios%NOTFOUND;  -- sai quando não há mais linhas

        DBMS_OUTPUT.PUT_LINE(v_reg.nome || ' — R$ ' || v_reg.salario);
    END LOOP;

    CLOSE c_funcionarios;  -- 4. Fechar: libera recursos
END;
/

-- Atributos de cursor:
-- %FOUND     → TRUE se o último FETCH retornou uma linha
-- %NOTFOUND  → TRUE se o último FETCH não retornou linha (fim dos dados)
-- %ISOPEN    → TRUE se o cursor está aberto
-- %ROWCOUNT  → quantidade de linhas buscadas até o momento

-- FOR de cursor — forma mais simples e recomendada
-- Abre, itera e fecha automaticamente
BEGIN
    FOR rec IN (SELECT nome, salario FROM funcionarios WHERE ativo = 'S') LOOP
        DBMS_OUTPUT.PUT_LINE(rec.nome || ': ' || rec.salario);
    END LOOP;
END;
/
```

### Exceções Pré-definidas

| Exception | Quando ocorre |
|-----------|---------------|
| `NO_DATA_FOUND` | `SELECT INTO` retornou zero linhas |
| `TOO_MANY_ROWS` | `SELECT INTO` retornou mais de uma linha |
| `DUP_VAL_ON_INDEX` | Violação de constraint `UNIQUE` ou `PRIMARY KEY` |
| `VALUE_ERROR` | Erro de conversão de tipo ou tamanho de variável |
| `ZERO_DIVIDE` | Divisão por zero |
| `INVALID_CURSOR` | Operação inválida em cursor (ex: `FETCH` sem `OPEN`) |
| `OTHERS` | Captura qualquer exceção não tratada acima |
```sql
-- Lançar exceção customizada com mensagem
-- Códigos entre -20000 e -20999 são reservados para o usuário
RAISE_APPLICATION_ERROR(-20001, 'Salário não pode ser negativo.');
```

---

## 10. Views, Sequences e Índices

### Views

Views são consultas armazenadas que se comportam como tabelas virtuais. Simplificam queries complexas e adicionam uma camada de segurança.
```sql
-- Criar ou substituir view
CREATE OR REPLACE VIEW vw_funcionarios_ativos AS
SELECT f.id, f.nome, f.salario, d.nome_dept
FROM   funcionarios f
JOIN   departamentos d ON f.dept_id = d.id
WHERE  f.ativo = 'S';

-- Usar a view como se fosse uma tabela
SELECT * FROM vw_funcionarios_ativos WHERE salario > 5000;

-- WITH CHECK OPTION — impede INSERT/UPDATE que violaria o WHERE da view
CREATE OR REPLACE VIEW vw_tecnologia AS
SELECT * FROM funcionarios
WHERE dept_id = 10
WITH CHECK OPTION;

-- WITH READ ONLY — impede qualquer DML pela view
CREATE OR REPLACE VIEW vw_resumo_folha AS
SELECT dept_id, COUNT(*) qtd, SUM(salario) total
FROM funcionarios
GROUP BY dept_id
WITH READ ONLY;

-- Remover view
DROP VIEW vw_funcionarios_ativos;
```

### Sequences

Sequences são geradores de números únicos e sequenciais — ideais para chaves primárias.
```sql
-- Criar sequence
CREATE SEQUENCE seq_func_id
    START WITH   1
    INCREMENT BY 1
    MINVALUE     1
    MAXVALUE     999999999
    NOCYCLE           -- não reinicia ao atingir MAXVALUE (lança exceção)
    CACHE 20;         -- pré-aloca 20 valores em memória — melhora performance

-- NEXTVAL — avança e retorna o próximo valor (cada chamada incrementa)
INSERT INTO funcionarios (id, nome, salario)
VALUES (seq_func_id.NEXTVAL, 'Carlos Pereira', 4500);

-- CURRVAL — retorna o valor atual sem avançar
-- (disponível apenas após o primeiro NEXTVAL na sessão)
SELECT seq_func_id.CURRVAL FROM dual;

-- Alterar sequence
ALTER SEQUENCE seq_func_id INCREMENT BY 5;
ALTER SEQUENCE seq_func_id CACHE 50;

-- Remover sequence
DROP SEQUENCE seq_func_id;

-- Oracle 12c+ — IDENTITY column (alternativa moderna à sequence manual)
CREATE TABLE pedidos (
    id        NUMBER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    descricao VARCHAR2(200)
);
```

### Índices

Índices aceleram consultas ao custo de espaço em disco e leve overhead em operações DML.
```sql
-- Índice padrão B-Tree
CREATE INDEX idx_func_nome ON funcionarios(nome);

-- Índice UNIQUE — garante unicidade e acelera buscas
CREATE UNIQUE INDEX idx_func_email ON funcionarios(email);

-- Índice composto — eficiente para queries com WHERE em múltiplas colunas
-- A ordem das colunas importa: use as mais seletivas (maior variação) primeiro
CREATE INDEX idx_func_dept_sal ON funcionarios(dept_id, salario);

-- Índice baseado em função — necessário quando a função é usada no WHERE
CREATE INDEX idx_func_upper_nome ON funcionarios(UPPER(nome));
-- Para que esse índice seja usado: WHERE UPPER(nome) = 'JOÃO SILVA'

-- Reconstruir índice (desfragmenta e reorganiza)
ALTER INDEX idx_func_nome REBUILD;

-- Remover índice
DROP INDEX idx_func_nome;

-- Ver o plano de execução de uma query (verifica se o índice está sendo usado)
EXPLAIN PLAN FOR
SELECT * FROM funcionarios WHERE nome LIKE 'J%';

SELECT * FROM TABLE(DBMS_XPLAN.DISPLAY);
```

> **Dica:** o Oracle cria automaticamente um índice B-Tree para colunas com `PRIMARY KEY` e `UNIQUE`. Não é necessário criar manualmente.

---

## 11. Dicas e Referência Rápida

### Tabela DUAL

`DUAL` é uma tabela especial do Oracle com uma linha e uma coluna. Use para avaliar expressões e funções sem uma tabela real.
```sql
SELECT SYSDATE           FROM dual;
SELECT 2 + 2             FROM dual;
SELECT USER              FROM dual;  -- usuário conectado
SELECT UPPER('oracle')   FROM dual;
```

### Operadores e Símbolos

| Símbolo | Uso |
|---------|-----|
| `\|\|` | Concatenação de strings: `'Olá' \|\| ' ' \|\| 'Mundo'` |
| `:=` | Atribuição em PL/SQL: `v_x := 10` |
| `=>` | Named notation em chamadas PL/SQL: `proc(p_id => 5)` |
| `:NEW` / `:OLD` | Linha nova/antiga em Triggers (`FOR EACH ROW`) |
| `%TYPE` | Herda tipo de coluna: `v_sal funcionarios.salario%TYPE` |
| `%ROWTYPE` | Herda estrutura de tabela: `v_func funcionarios%ROWTYPE` |
| `SQL%ROWCOUNT` | Nº de linhas afetadas pelo último DML em PL/SQL |
| `SQL%FOUND` | `TRUE` se o último DML afetou ao menos uma linha |

### Comentários
```sql
-- Comentário de uma linha

/* Comentário
   de múltiplas
   linhas */
```

### Aliases (Apelidos)
```sql
-- Alias de coluna
SELECT nome AS nome_completo, salario * 12 AS salario_anual
FROM funcionarios;

-- Alias de tabela (simplifica queries com joins)
SELECT f.nome, d.nome_dept
FROM funcionarios f
JOIN departamentos d ON f.dept_id = d.id;
```

---

*Fim do Manual — Básico ao Intermediário*
