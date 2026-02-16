# 📊 Funções de Agregação e Inteligência de Dados no Oracle

> **Manual completo sobre funções de agregação, GROUP BY, HAVING e análise de dados no Oracle Database**

---

## 📑 Índice

- [O Coração da Análise](#o-coração-da-análise-o-que-é-agregação)
- [Arsenal de Funções Estatísticas](#arsenal-de-funções-estatísticas)
- [Segmentação com GROUP BY](#segmentação-lógica-com-group-by)
- [Refinando com HAVING](#refinando-resultados-a-cláusula-having)
- [Aplicações Práticas](#aplicação-prática-extraindo-insights)
- [Boas Práticas](#nuances-técnicas-e-boas-práticas)

---

## O Coração da Análise: O que é Agregação?

**O que são funções de agregação?** São operações que transformam múltiplos registros em um único valor resultante, permitindo extrair inteligência de grandes volumes de dados.

### Por que usar agregação?

No ecossistema da Engenharia de Dados, as funções de agregação representam o estágio crucial onde **o dado bruto se transforma em métrica (KPI)**. Elas condensam múltiplos registros em um único valor, permitindo análise sem inspeção linha a linha.

### Benefícios para o Negócio

- **📈 Extração de KPIs:** Identificação imediata de tendências, médias de performance e valores extremos
- **⚡ Eficiência:** Processamento no servidor Oracle, reduzindo tráfego de rede
- **📋 Consolidação:** Transforma tabelas densas em resumos executivos claros

**Insight de especialista:** Dominar agregação permite atuar como **arquiteto de inteligência**, preparando o terreno para análises avançadas e dashboards de performance.

---

## Arsenal de Funções Estatísticas

O Oracle disponibiliza um conjunto robusto de funções para operações matemáticas e análises estatísticas. Vamos explorar usando a tabela de exemplo `AGREGACAO` (com colunas `valpro` para valores e `qtdpro` para quantidades).

### Tabela de Referência Rápida

| Função | O que faz | Exemplo Prático |
|--------|-----------|-----------------|
| **COUNT** | Conta o número de registros | `SELECT COUNT(*) FROM AGREGACAO;` |
| **SUM** | Soma os valores de uma coluna | `SELECT SUM(valpro) FROM AGREGACAO;` |
| **AVG** | Calcula a média aritmética | `SELECT AVG(valpro) FROM AGREGACAO;` |
| **MAX** | Identifica o maior valor | `SELECT MAX(valpro) FROM AGREGACAO;` |
| **MIN** | Identifica o menor valor | `SELECT MIN(valpro) FROM AGREGACAO;` |
| **STDDEV** | Calcula o desvio padrão | `SELECT STDDEV(valpro) FROM AGREGACAO;` |
| **VARIANCE** | Calcula a variância | `SELECT VARIANCE(valpro) FROM AGREGACAO;` |

### Exemplos Detalhados

#### COUNT - Contar registros

```sql
-- Contar todas as linhas (incluindo NULL)
SELECT COUNT(*) AS total_registros 
FROM AGREGACAO;

-- Contar apenas valores não-nulos
SELECT COUNT(valpro) AS valores_preenchidos 
FROM AGREGACAO;

-- Contar valores únicos
SELECT COUNT(DISTINCT codcli) AS total_clientes 
FROM AGREGACAO;
```

#### SUM - Somar valores

```sql
-- Somar todos os valores
SELECT SUM(valpro) AS valor_total 
FROM AGREGACAO;

-- Somar com filtro
SELECT SUM(valpro) AS valor_total_cliente 
FROM AGREGACAO 
WHERE codcli = 1;
```

#### AVG - Calcular média

```sql
-- Média simples
SELECT AVG(valpro) AS ticket_medio 
FROM AGREGACAO;

-- Média arredondada
SELECT ROUND(AVG(valpro), 2) AS ticket_medio 
FROM AGREGACAO;
```

#### MAX e MIN - Valores extremos

```sql
-- Maior e menor valor
SELECT 
    MAX(valpro) AS maior_venda,
    MIN(valpro) AS menor_venda,
    MAX(valpro) - MIN(valpro) AS amplitude
FROM AGREGACAO;
```

#### STDDEV e VARIANCE - Análise de dispersão

```sql
-- Desvio padrão e variância
SELECT 
    AVG(valpro) AS media,
    STDDEV(valpro) AS desvio_padrao,
    VARIANCE(valpro) AS variancia
FROM AGREGACAO;
```

### 💡 Insights de Especialista

**STDDEV e VARIANCE** são fundamentais para análise de qualidade de dados:
- **Desvio baixo:** Dados consistentes, seguem padrão esperado
- **Desvio alto:** Dados dispersos, possível inconsistência ou outliers

**Atenção ao NULL:**
- `COUNT(*)` → Conta TODAS as linhas (incluindo NULL)
- `COUNT(valpro)` → Conta apenas valores preenchidos (ignora NULL)
- `SUM(valpro)` → Ignora NULL automaticamente
- `AVG(valpro)` → Ignora NULL (cuidado: pode distorcer média!)

---

## Segmentação Lógica com GROUP BY

**O que faz?** O `GROUP BY` "fatia" o conjunto de dados antes da aplicação da função agregada. Se as funções nos dão a visão macro, o GROUP BY permite analisar subconjuntos específicos.

### Ordem Lógica de Execução

```
1. FROM     → Define a tabela origem
2. WHERE    → Filtra linhas individuais (antes da agregação)
3. GROUP BY → Agrupa os dados em segmentos
4. HAVING   → Filtra grupos (depois da agregação)
5. SELECT   → Define as colunas de saída
6. ORDER BY → Ordena o resultado final
```

### Sintaxe Básica

```sql
SELECT 
    coluna_agrupamento,
    FUNCAO_AGREGACAO(coluna_valores)
FROM tabela
GROUP BY coluna_agrupamento;
```

### Exemplos Práticos

#### Exemplo 1: Vendas por Cliente

```sql
-- Total de vendas por cliente
SELECT 
    codcli,
    SUM(valpro) AS total_vendas,
    COUNT(*) AS total_pedidos,
    AVG(valpro) AS ticket_medio
FROM AGREGACAO
GROUP BY codcli;
```

**Resultado esperado:**
```
CODCLI | TOTAL_VENDAS | TOTAL_PEDIDOS | TICKET_MEDIO
-------|--------------|---------------|-------------
1      | 15000.00     | 12            | 1250.00
2      | 8500.50      | 7             | 1214.36
3      | 22000.00     | 15            | 1466.67
```

#### Exemplo 2: Análise por Produto

```sql
-- Produtos mais vendidos
SELECT 
    codpro,
    SUM(qtdpro) AS quantidade_total,
    SUM(valpro) AS valor_total,
    COUNT(DISTINCT codcli) AS clientes_distintos
FROM AGREGACAO
GROUP BY codpro
ORDER BY valor_total DESC;
```

#### Exemplo 3: Agrupamento por múltiplas colunas

```sql
-- Vendas por cliente e produto
SELECT 
    codcli,
    codpro,
    SUM(valpro) AS total,
    COUNT(*) AS qtd_compras
FROM AGREGACAO
GROUP BY codcli, codpro;
```

### 🚨 Regra de Ouro da Sintaxe

**IMPORTANTE:** Qualquer coluna listada no `SELECT` que **NÃO** esteja dentro de uma função agregada **DEVE** estar no `GROUP BY`.

**❌ ERRADO:**
```sql
SELECT codcli, codpro, SUM(valpro)
FROM AGREGACAO
GROUP BY codcli;  -- Falta codpro aqui!
```

**✅ CORRETO:**
```sql
SELECT codcli, codpro, SUM(valpro)
FROM AGREGACAO
GROUP BY codcli, codpro;  -- Ambas as colunas no GROUP BY
```

### Exemplo Completo: Análise de Performance

```sql
-- Ranking de clientes com estatísticas completas
SELECT 
    codcli,
    COUNT(*) AS total_compras,
    SUM(valpro) AS valor_total,
    AVG(valpro) AS ticket_medio,
    MIN(valpro) AS menor_compra,
    MAX(valpro) AS maior_compra,
    STDDEV(valpro) AS desvio_padrao
FROM AGREGACAO
GROUP BY codcli
ORDER BY valor_total DESC;
```

---

## Refinando Resultados: A Cláusula HAVING

**O que faz?** Enquanto `WHERE` filtra linhas **antes** da agregação, `HAVING` filtra **resultados agregados** após os cálculos de grupo.

### WHERE vs. HAVING - Comparativo Técnico

| Cláusula | Momento | Filtra | Exemplo |
|----------|---------|--------|---------|
| **WHERE** | Antes da agregação | Linhas individuais | `WHERE valpro > 100` |
| **HAVING** | Após o agrupamento | Resultados agregados | `HAVING SUM(valpro) > 1000` |

### Quando usar cada um?

```sql
-- WHERE: Filtrar ANTES de agrupar (mais eficiente)
SELECT codcli, SUM(valpro) AS total
FROM AGREGACAO
WHERE valpro > 100  -- Filtra linhas individuais
GROUP BY codcli;

-- HAVING: Filtrar DEPOIS de agrupar
SELECT codcli, SUM(valpro) AS total
FROM AGREGACAO
GROUP BY codcli
HAVING SUM(valpro) > 5000;  -- Filtra grupos/totais
```

### Exemplos Práticos

#### Exemplo 1: Clientes VIP (volume alto)

```sql
-- Clientes que compraram mais de R$ 5.000
SELECT 
    codcli,
    SUM(valpro) AS total_vendas,
    COUNT(*) AS total_pedidos
FROM AGREGACAO
GROUP BY codcli
HAVING SUM(valpro) > 5000
ORDER BY total_vendas DESC;
```

#### Exemplo 2: Produtos com baixa movimentação

```sql
-- Produtos vendidos menos de 5 vezes
SELECT 
    codpro,
    COUNT(*) AS total_vendas,
    SUM(qtdpro) AS quantidade_total
FROM AGREGACAO
GROUP BY codpro
HAVING COUNT(*) < 5;
```

#### Exemplo 3: Combinando WHERE e HAVING

```sql
-- Clientes com compras acima de R$ 100 
-- que totalizaram mais de R$ 1.000
SELECT 
    codcli,
    COUNT(*) AS compras_acima_100,
    SUM(valpro) AS valor_total
FROM AGREGACAO
WHERE valpro > 100           -- Filtra antes (apenas vendas > 100)
GROUP BY codcli
HAVING SUM(valpro) > 1000    -- Filtra depois (total > 1000)
ORDER BY valor_total DESC;
```

#### Exemplo 4: Análise de frequência

```sql
-- Clientes que compraram mais de 10 vezes
-- com ticket médio acima de R$ 200
SELECT 
    codcli,
    COUNT(*) AS frequencia,
    AVG(valpro) AS ticket_medio,
    SUM(valpro) AS total
FROM AGREGACAO
GROUP BY codcli
HAVING COUNT(*) > 10 
   AND AVG(valpro) > 200;
```

### 💡 Dica de Performance

Use `WHERE` sempre que possível antes de `HAVING`:
- `WHERE` reduz o volume de dados **antes** da agregação (mais eficiente)
- `HAVING` processa todos os dados e depois filtra (menos eficiente)

**❌ Menos eficiente:**
```sql
SELECT codcli, SUM(valpro)
FROM AGREGACAO
GROUP BY codcli
HAVING codcli = 1;  -- Processa tudo, depois filtra
```

**✅ Mais eficiente:**
```sql
SELECT codcli, SUM(valpro)
FROM AGREGACAO
WHERE codcli = 1    -- Filtra antes de processar
GROUP BY codcli;
```

---

## Aplicação Prática: Extraindo Insights

Com base nas tabelas `AGREGACAO`, `Produtos` e `Clientes`, vamos responder perguntas críticas de negócio.

### 1️⃣ Ticket Médio por Nota Fiscal

**Pergunta de negócio:** Qual o valor médio dos itens em cada venda?

```sql
-- Ticket médio por nota fiscal
SELECT 
    numnf,
    COUNT(*) AS total_itens,
    SUM(valpro) AS valor_total_nf,
    AVG(valpro) AS ticket_medio_item,
    MIN(valpro) AS item_mais_barato,
    MAX(valpro) AS item_mais_caro
FROM AGREGACAO
GROUP BY numnf
ORDER BY valor_total_nf DESC;
```

**Insight:** Isso revela o valor médio dos itens vendidos em cada nota fiscal, ajudando a identificar vendas de alto valor.

### 2️⃣ Performance de Vendas por Cliente (Top Performers)

**Pergunta de negócio:** Quem são nossos melhores clientes?

```sql
-- Ranking dos melhores clientes
SELECT 
    a.codcli,
    c.nome_cliente,
    COUNT(*) AS total_compras,
    SUM(a.valpro) AS valor_total,
    AVG(a.valpro) AS ticket_medio,
    MAX(a.valpro) AS maior_compra
FROM AGREGACAO a
JOIN CLIENTES c ON a.codcli = c.id_cliente
GROUP BY a.codcli, c.nome_cliente
HAVING SUM(a.valpro) > 1000  -- Apenas clientes com volume significativo
ORDER BY valor_total DESC
FETCH FIRST 10 ROWS ONLY;  -- Top 10
```

### 3️⃣ Variedade de Mix de Produtos por Venda

**Pergunta de negócio:** Quantos produtos diferentes foram vendidos em cada transação?

```sql
-- Diversidade de produtos por nota fiscal
SELECT 
    numnf,
    COUNT(*) AS total_itens,           -- Total de linhas
    COUNT(DISTINCT codpro) AS produtos_unicos,  -- Produtos diferentes
    SUM(valpro) AS valor_total
FROM AGREGACAO
GROUP BY numnf
HAVING COUNT(DISTINCT codpro) > 5  -- Vendas com mais de 5 produtos diferentes
ORDER BY produtos_unicos DESC;
```

**Dica de Engenharia:** Use `DISTINCT` criteriosamente:
- `COUNT(codpro)` → Conta **movimentações** (inclui repetições)
- `COUNT(DISTINCT codpro)` → Conta **itens únicos** no catálogo

### 4️⃣ Análise de Sazonalidade (se houver coluna de data)

```sql
-- Vendas por mês (assumindo coluna data_venda)
SELECT 
    TO_CHAR(data_venda, 'YYYY-MM') AS mes,
    COUNT(DISTINCT numnf) AS total_vendas,
    COUNT(*) AS total_itens,
    SUM(valpro) AS receita_total,
    AVG(valpro) AS ticket_medio
FROM AGREGACAO
GROUP BY TO_CHAR(data_venda, 'YYYY-MM')
ORDER BY mes;
```

### 5️⃣ Produtos Campeões de Venda

```sql
-- Top produtos por faturamento
SELECT 
    a.codpro,
    p.nome_produto,
    SUM(a.qtdpro) AS quantidade_vendida,
    SUM(a.valpro) AS faturamento_total,
    COUNT(DISTINCT a.codcli) AS clientes_compraram,
    AVG(a.valpro) AS preco_medio_venda
FROM AGREGACAO a
JOIN PRODUTOS p ON a.codpro = p.id_produto
GROUP BY a.codpro, p.nome_produto
ORDER BY faturamento_total DESC
FETCH FIRST 20 ROWS ONLY;
```

### 6️⃣ Análise ABC de Clientes (Pareto)

```sql
-- Classificar clientes por faturamento (Curva ABC)
SELECT 
    codcli,
    SUM(valpro) AS faturamento,
    ROUND(
        100 * SUM(valpro) / (SELECT SUM(valpro) FROM AGREGACAO), 
        2
    ) AS percentual_faturamento,
    CASE 
        WHEN SUM(valpro) >= (SELECT SUM(valpro) * 0.8 FROM AGREGACAO) THEN 'A'
        WHEN SUM(valpro) >= (SELECT SUM(valpro) * 0.15 FROM AGREGACAO) THEN 'B'
        ELSE 'C'
    END AS classificacao_abc
FROM AGREGACAO
GROUP BY codcli
ORDER BY faturamento DESC;
```

### 7️⃣ Análise de Consistência de Preços

```sql
-- Verificar produtos com grande variação de preço
SELECT 
    codpro,
    COUNT(*) AS total_vendas,
    MIN(valpro) AS preco_minimo,
    MAX(valpro) AS preco_maximo,
    AVG(valpro) AS preco_medio,
    STDDEV(valpro) AS desvio_padrao,
    ROUND((MAX(valpro) - MIN(valpro)) / AVG(valpro) * 100, 2) AS variacao_percentual
FROM AGREGACAO
GROUP BY codpro
HAVING STDDEV(valpro) > 100  -- Produtos com alta variação
ORDER BY desvio_padrao DESC;
```

---

## Nuances Técnicas e Boas Práticas

Para garantir consultas performáticas e resultados confiáveis, siga estas diretrizes.

### 1. 🔢 Tipagem de Dados

**Regra:** Funções como `SUM` e `AVG` operam **apenas** sobre tipos numéricos.

```sql
-- ✅ CORRETO
SELECT SUM(valpro) FROM AGREGACAO;  -- valpro é NUMBER

-- ❌ ERRO
SELECT SUM(codpro) FROM AGREGACAO;  -- codpro pode ser VARCHAR
```

**Tipos aceitos:**
- `NUMBER`
- `DECIMAL`
- `INTEGER`
- `FLOAT`

### 2. ⚠️ Tratamento de Nulos

**Regra:** Exceto `COUNT(*)`, todas as funções **ignoram valores NULL**.

```sql
-- Exemplo com NULLs
CREATE TABLE TESTE (valor NUMBER);
INSERT INTO TESTE VALUES (10);
INSERT INTO TESTE VALUES (20);
INSERT INTO TESTE VALUES (NULL);
INSERT INTO TESTE VALUES (30);

-- Resultados:
SELECT COUNT(*) FROM TESTE;        -- 4 (conta tudo, incluindo NULL)
SELECT COUNT(valor) FROM TESTE;    -- 3 (ignora NULL)
SELECT SUM(valor) FROM TESTE;      -- 60 (10+20+30, ignora NULL)
SELECT AVG(valor) FROM TESTE;      -- 20 (60/3, não 60/4!)
```

**Cuidado com AVG:**
```sql
-- Se sua tabela tem NULLs, a média pode estar distorcida
-- Use NVL para tratar:
SELECT AVG(NVL(valpro, 0)) FROM AGREGACAO;  -- Considera NULL como 0
```

### 3. 💾 Persistência (COMMIT)

**Regra:** Sempre confirme suas transações.

```sql
-- Após inserções ou atualizações de teste
INSERT INTO AGREGACAO VALUES (...);
COMMIT;  -- ⚠️ SEM ISSO, SEUS DADOS NÃO SÃO SALVOS!

-- Se algo deu errado
ROLLBACK;  -- Desfaz alterações não confirmadas
```

### 4. 🔗 Integridade Referencial

**Regra:** Garanta que suas chaves estrangeiras estejam corretas.

```sql
-- Verificar "dados órfãos" (sem referência)
SELECT COUNT(*) 
FROM AGREGACAO a
LEFT JOIN CLIENTES c ON a.codcli = c.id_cliente
WHERE c.id_cliente IS NULL;  -- Se > 0, há dados órfãos

-- Mesmo para produtos
SELECT COUNT(*) 
FROM AGREGACAO a
LEFT JOIN PRODUTOS p ON a.codpro = p.id_produto
WHERE p.id_produto IS NULL;
```

### 5. 📊 Performance e Índices

**Dica:** Crie índices em colunas usadas no `GROUP BY` e `WHERE`.

```sql
-- Índices recomendados para agregação
CREATE INDEX idx_agregacao_codcli ON AGREGACAO(codcli);
CREATE INDEX idx_agregacao_codpro ON AGREGACAO(codpro);
CREATE INDEX idx_agregacao_numnf ON AGREGACAO(numnf);

-- Índice composto para queries frequentes
CREATE INDEX idx_agregacao_cli_pro ON AGREGACAO(codcli, codpro);
```

### 6. 🎯 Uso Eficiente do DISTINCT

```sql
-- ❌ Evite DISTINCT desnecessário (custoso)
SELECT DISTINCT codcli FROM AGREGACAO;  -- Se codcli já é único

-- ✅ Use DISTINCT quando realmente necessário
SELECT COUNT(DISTINCT codcli) FROM AGREGACAO;  -- Contagem de únicos
```

### 7. 📝 Aliases para Legibilidade

```sql
-- Use aliases descritivos
SELECT 
    codcli AS codigo_cliente,
    COUNT(*) AS total_pedidos,
    SUM(valpro) AS receita_total,
    AVG(valpro) AS ticket_medio,
    MAX(valpro) AS maior_venda
FROM AGREGACAO
GROUP BY codcli
ORDER BY receita_total DESC;
```

### 8. 🔍 Validação de Resultados

```sql
-- Sempre valide seus totais
SELECT 
    SUM(valpro) AS total_por_agregacao
FROM AGREGACAO;

-- Compare com a soma dos grupos
SELECT 
    SUM(total_grupo) AS soma_grupos
FROM (
    SELECT codcli, SUM(valpro) AS total_grupo
    FROM AGREGACAO
    GROUP BY codcli
);

-- Os dois valores devem ser iguais!
```

### 9. 💡 Documentação de Queries Complexas

```sql
-- Comente queries complexas
/* 
    Objetivo: Identificar clientes VIP (top 20%) por faturamento
    Critério: Faturamento acumulado >= 80% do total
    Tabelas: AGREGACAO, CLIENTES
*/
SELECT 
    a.codcli,
    c.nome_cliente,
    SUM(a.valpro) AS faturamento_total
FROM AGREGACAO a
JOIN CLIENTES c ON a.codcli = c.id_cliente
GROUP BY a.codcli, c.nome_cliente
HAVING SUM(a.valpro) >= (
    SELECT SUM(valpro) * 0.8 / COUNT(DISTINCT codcli)
    FROM AGREGACAO
)
ORDER BY faturamento_total DESC;
```

### 10. 🚀 Otimização com Materialized Views

Para queries pesadas executadas frequentemente:

```sql
-- Criar view materializada com agregações
CREATE MATERIALIZED VIEW mv_vendas_cliente
BUILD IMMEDIATE
REFRESH COMPLETE ON DEMAND
AS
SELECT 
    codcli,
    COUNT(*) AS total_compras,
    SUM(valpro) AS valor_total,
    AVG(valpro) AS ticket_medio,
    MAX(valpro) AS maior_compra,
    MIN(valpro) AS menor_compra
FROM AGREGACAO
GROUP BY codcli;

-- Usar a MV (muito mais rápido!)
SELECT * FROM mv_vendas_cliente WHERE valor_total > 5000;
```

---

## 🎯 Checklist de Boas Práticas

- [ ] Todas as colunas não-agregadas estão no GROUP BY
- [ ] Uso correto de WHERE (antes) vs HAVING (depois)
- [ ] Tratamento adequado de valores NULL
- [ ] Tipos de dados corretos (numéricos para SUM/AVG)
- [ ] COMMIT após alterações de teste
- [ ] Validação de integridade referencial
- [ ] Índices nas colunas de agrupamento
- [ ] Aliases descritivos nos resultados
- [ ] Documentação de queries complexas
- [ ] Validação dos totais agregados

---

## 📚 Resumo Executivo

**Funções de agregação** transformam dados brutos em inteligência de negócio:

| Função | Uso Principal | Cuidado |
|--------|---------------|---------|
| COUNT | Contar registros | COUNT(*) vs COUNT(coluna) |
| SUM | Totalizar valores | Apenas numéricos, ignora NULL |
| AVG | Calcular médias | NULL distorce resultado |
| MAX/MIN | Valores extremos | Útil para ranges |
| STDDEV/VARIANCE | Medir dispersão | Qualidade de dados |

**GROUP BY** segmenta dados para análise detalhada.
**HAVING** filtra resultados agregados.

**Diferencial profissional:** Dominar agregação separa o operador de banco de dados do **estrategista de informações**.

---

## 🚀 Próximos Passos

1. Converter queries complexas em **VIEWs** para reutilização
2. Criar **Materialized Views** para queries pesadas
3. Implementar **PROCEDUREs** para cálculos recorrentes
4. Desenvolver **dashboards** baseados em agregações
5. Estudar **Window Functions** para análises avançadas

---

**Licença:** MIT  
**Contribuições:** Bem-vindas via Pull Request  

**Bons estudos! 📊🚀**
