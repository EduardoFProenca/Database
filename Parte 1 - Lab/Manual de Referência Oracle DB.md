
# 🐘 Manual de Referência Oracle DB

> Um guia rápido de comandos SQL e PL/SQL para estudos e consulta no Oracle Database.

## 📋 Índice
1. [Comandos Básicos e Conexão](#comandos-básicos-e-conexão)
2. [DDL - Definição de Dados](#ddl---definição-de-dados)
3. [DML - Manipulação de Dados](#dml---manipulação-de-dados)
4. [TCL - Controle de Transação](#tcl---controle-de-transação)
5. [Consultas (SELECT)](#consultas-select)
6. [Funções Úteis](#funções-úteis)
7. [Joins (Junções)](#joins-junções)
8. [PL/SQL Básico](#plsql-básico)
9. [Outros Objetos (Views, Sequences)](#outros-objetos)

---

## 📌 Comandos Básicos e Conexão

Comandos essenciais para navegação e verificação de status.

```sql
-- Mostrar o usuário atual
SHOW USER;

-- Limpar a tela do terminal (SQL*Plus ou SQLcl)
CLEAR SCREEN;

-- Verificar a estrutura de uma tabela
DESC nome_da_tabela;

-- Editar a última query no editor padrão
EDIT;

-- Executar o script no buffer
RUN;

-- Verificar tabelas acessíveis ao usuário
SELECT table_name FROM user_tables;
```

---

## 🏗️ DDL - Definição de Dados

Comandos para criar, alterar e deletar estruturas.

### Criar Tabela (CREATE TABLE)
```sql
CREATE TABLE funcionarios (
    id NUMBER PRIMARY KEY,
    nome VARCHAR2(100) NOT NULL,
    salario NUMBER(10, 2),
    data_admissao DATE DEFAULT SYSDATE,
    dept_id NUMBER
);
```

### Alterar Tabela (ALTER TABLE)
```sql
-- Adicionar uma nova coluna
ALTER TABLE funcionarios ADD (email VARCHAR2(100));

-- Modificar tipo de dados
ALTER TABLE funcionarios MODIFY (nome VARCHAR2(150));

-- Adicionar uma chave estrangeira (Constraint)
ALTER TABLE funcionarios 
ADD CONSTRAINT fk_dept 
FOREIGN KEY (dept_id) 
REFERENCES departamentos(id);

-- Remover uma coluna
ALTER TABLE funcionarios DROP COLUMN email;
```

### Deletar Tabela (DROP TABLE)
```sql
-- Remove a tabela e os dados (não pode desfazer com ROLLBACK se não tiver Flashback)
DROP TABLE funcionarios;

-- Remove a tabela e move para a "Lixeira" (Recycle Bin)
DROP TABLE funcionarios PURGE;
```

### Renomear
```sql
-- Renomear tabela
RENAME funcionarios TO colaboradores;

-- Renomear coluna (Oracle 9i+)
ALTER TABLE funcionarios RENAME COLUMN nome TO nome_completo;
```

---

## ✏️ DML - Manipulação de Dados

Inserir, atualizar e deletar dados.

### Inserir (INSERT)
```sql
-- Inserção completa
INSERT INTO funcionarios (id, nome, salario, dept_id)
VALUES (1, 'João Silva', 5000.00, 10);

-- Inserção implícita (cuidado com a ordem das colunas)
INSERT INTO funcionarios VALUES (2, 'Maria Souza', 6000.00, SYSDATE, 20);

-- Inserção com SELECT (Copiar dados de outra tabela)
INSERT INTO historico_salarios
SELECT * FROM funcionarios;
```

### Atualizar (UPDATE)
```sql
UPDATE funcionarios
SET salario = 5500.00, nome = 'João Silva Jr.'
WHERE id = 1;

-- Atualizar múltiplas linhas
UPDATE funcionarios
SET salario = salario * 1.10 -- Aumento de 10%
WHERE dept_id = 10;
```

### Deletar (DELETE)
```sql
-- Deletar linhas específicas
DELETE FROM funcionarios 
WHERE id = 1;

-- Deletar tudo (cuidado, pode travar o banco se for muita coisa, use TRUNCATE para limpeza total)
DELETE FROM funcionarios;
```

### Truncate (Limpeza Rápida)
```sql
-- Remove todos os dados rapidamente e reseta o High Water Mark. Não pode ser desfeito.
TRUNCATE TABLE funcionarios;
```

---

## 💾 TCL - Controle de Transação

Gerencia as mudanças feitas pelo DML.

```sql
-- Salvar as mudanças permanentemente
COMMIT;

-- Desfazer as mudanças desde o último COMMIT
ROLLBACK;

-- Criar um ponto de salvamento específico
SAVEPOINT ponto_a;

-- Desfazer até o ponto de salvamento
ROLLBACK TO ponto_a;
```

---

## 🔍 Consultas (SELECT)

### Estrutura Básica
```sql
SELECT coluna1, coluna2
FROM nome_tabela
WHERE condição
ORDER BY coluna1 ASC/DESC;
```

### Filtros e Operadores
```sql
-- Distinto (remove duplicatas)
SELECT DISTINCT dept_id FROM funcionarios;

-- Between (Intervalo)
SELECT * FROM funcionarios 
WHERE salario BETWEEN 3000 AND 7000;

-- IN (Lista de valores)
SELECT * FROM funcionarios 
WHERE dept_id IN (10, 20, 30);

-- LIKE (Padrão de texto)
-- % : Qualquer sequência de caracteres
-- _ : Qualquer caractere único
SELECT * FROM funcionarios 
WHERE nome LIKE 'J%'; -- Começa com J

-- IS NULL (Valores nulos)
SELECT * FROM funcionarios 
WHERE email IS NULL;
```

### Agrupamento (GROUP BY)
```sql
SELECT dept_id, COUNT(*), AVG(salario)
FROM funcionarios
GROUP BY dept_id
HAVING AVG(salario) > 5000; -- Filtro após o agrupamento
```

### Ordenação
```sql
-- Ordenar por múltiplas colunas
SELECT * FROM funcionarios 
ORDER BY dept_id ASC, salario DESC;
```

---

## 🛠️ Funções Úteis

### Funções de Texto
```sql
-- Concatenar
SELECT nome || ' - ' || cargo FROM funcionarios;

-- Maiúscula / Minúscula / Inicial Maiúscula
SELECT UPPER(nome), LOWER(email), INITCAP(descricao) FROM produtos;

-- Tamanho da string
SELECT LENGTH(nome) FROM funcionarios;

-- Substituir texto
SELECT REPLACE(nome, 'Silva', 'Oliveira') FROM funcionarios;

-- Extrair parte do texto (substring)
SELECT SUBSTRING(nome, 1, 5) FROM funcionarios; -- Pega os 5 primeiros caracteres
```

### Funções Numéricas
```sql
-- Arredondar
SELECT ROUND(salario, -1) FROM funcionarios; -- Arredonda para dezena

-- Truncar (casa decimal sem arredondar)
SELECT TRUNC(salario, 2) FROM funcionarios;

-- Resto da divisão (Mod)
SELECT MOD(id, 2) FROM funcionarios;
```

### Funções de Data
```sql
-- Data atual
SELECT SYSDATE FROM dual;

-- Adicionar meses
ADD_MONTHS(SYSDATE, 3);

-- Diferença entre meses
MONTHS_BETWEEN(data_fim, data_inicio);

-- Último dia do mês
LAST_DAY(SYSDATE);

-- Próximo dia da semana (ex: próxima sexta)
NEXT_DAY(SYSDATE, 'FRIDAY');
```

### Funções de Conversão
```sql
-- Número para Texto
TO_CHAR(1234.56, 'R$9,999.99');

-- Texto para Data
TO_DATE('01/01/2023', 'DD/MM/YYYY');

-- Texto para Número
TO_NUMBER('1.200,99', '9G999D99');
```

### Funções Condicionais
```sql
-- NVL (Substitui nulo por outro valor)
SELECT nome, NVL(comissao, 0) FROM funcionarios;

-- DECODE (Case simples)
SELECT nome, DECODE(codigo, 1, 'Ativo', 2, 'Inativo', 'Desconhecido') status
FROM usuarios;

-- CASE (Expressão condicional padrão)
SELECT nome,
    CASE 
        WHEN salario < 3000 THEN 'Baixo'
        WHEN salario BETWEEN 3000 AND 6000 THEN 'Médio'
        ELSE 'Alto'
    END faixa_salarial
FROM funcionarios;
```

---

## 🔗 Joins (Junções)

```sql
-- INNER JOIN (Apenas o que existe em ambas)
SELECT f.nome, d.nome_dept
FROM funcionarios f
INNER JOIN departamentos d ON f.dept_id = d.id;

-- LEFT JOIN (Tudo da tabela da esquerda, mesmo sem correspondência na direita)
SELECT f.nome, d.nome_dept
FROM funcionarios f
LEFT JOIN departamentos d ON f.dept_id = d.id;

-- RIGHT JOIN (Tudo da tabela da direita)
SELECT f.nome, d.nome_dept
FROM funcionarios f
RIGHT JOIN departamentos d ON f.dept_id = d.id;

-- FULL JOIN (Tudo de ambas)
SELECT f.nome, d.nome_dept
FROM funcionarios f
FULL JOIN departamentos d ON f.dept_id = d.id;

-- CROSS JOIN (Produto cartesiano - todas as combinações)
SELECT f.nome, d.nome_dept
FROM funcionarios f
CROSS JOIN departamentos d;

-- SELF JOIN (Junção da tabela com ela mesma)
SELECT e.nome AS funcionario, g.nome AS gerente
FROM funcionarios e
LEFT JOIN funcionarios g ON e.gerente_id = g.id;
```

---

## ⚙️ PL/SQL Básico

### Bloco Anônimo
```sql
DECLARE
    -- Declaração de variáveis
    v_nome VARCHAR2(100);
    v_salario NUMBER := 0;
BEGIN
    -- Execução
    SELECT nome, salario INTO v_nome, v_salario
    FROM funcionarios
    WHERE id = 1;
    
    DBMS_OUTPUT.PUT_LINE('Funcionário: ' || v_nome || ' Salário: ' || v_salario);
EXCEPTION
    -- Tratamento de erro
    WHEN NO_DATA_FOUND THEN
        DBMS_OUTPUT.PUT_LINE('Nenhum registro encontrado.');
    WHEN OTHERS THEN
        DBMS_OUTPUT.PUT_LINE('Erro: ' || SQLERRM);
END;
/
```

### IF / ELSIF / ELSE
```sql
IF v_salario > 10000 THEN
    v_bonus := 1000;
ELSIF v_salario > 5000 THEN
    v_bonus := 500;
ELSE
    v_bonus := 0;
END IF;
```

### Loops (Laços)
```sql
-- Loop Simples
LOOP
    v_contador := v_contador + 1;
    EXIT WHEN v_contador > 10;
END LOOP;

-- While Loop
WHILE v_contador < 10 LOOP
    v_contador := v_contador + 1;
END LOOP;

-- For Loop
FOR i IN 1..10 LOOP
    DBMS_OUTPUT.PUT_LINE('Iteração: ' || i);
END LOOP;
```

### Cursor Explícito
```sql
DECLARE
    CURSOR c_func IS SELECT nome, salario FROM funcionarios;
    v_reg c_func%ROWTYPE;
BEGIN
    OPEN c_func;
    LOOP
        FETCH c_func INTO v_reg;
        EXIT WHEN c_func%NOTFOUND;
        DBMS_OUTPUT.PUT_LINE(v_reg.nome);
    END LOOP;
    CLOSE c_func;
END;
/
```

---

## 📦 Outros Objetos

### Views (Visões)
```sql
-- Criar View
CREATE VIEW vw_salarios_altos AS
SELECT nome, salario 
FROM funcionarios 
WHERE salario > 5000
WITH CHECK CONSTRAINT; -- Impede insert/update que viole a condição WHERE

-- Drop View
DROP VIEW vw_salarios_altos;
```

### Sequences (Sequenciadores)
```sql
-- Criar Sequence
CREATE SEQEUNCIE seq_func_id
START WITH 1
INCREMENT BY 1
NOCACHE
NOCYCLE;

-- Usar
INSERT INTO funcionarios (id, nome) VALUES (seq_func_id.NEXTVAL, 'Teste');

-- Ver valor atual (sem avançar)
SELECT seq_func_id.CURRVAL FROM dual;

-- Drop
DROP SEQUENCE seq_func_id;
```

### Índices
```sql
-- Criar índice básico
CREATE INDEX idx_func_nome ON funcionarios(nome);

-- Índice único
CREATE UNIQUE INDEX idx_func_email ON funcionarios(email);

-- Remover
DROP INDEX idx_func_nome;
```

---

## 💡 Dicas Rápidas

*   **Tabela DUAL:** É uma tabela dummy do Oracle usada para selecionar expressões que não envolvem uma tabela real (ex: `SELECT SYSDATE FROM dual;`).
*   **Comentários:** Use `--` para uma linha ou `/* ... */` para múltiplas linhas.
*   **Apelidos (Aliases:** Renomeiam colunas ou tabelas na query temporariamente (ex: `SELECT nome AS n FROM funcionarios f`).

---

*Fim do Manual*
```
