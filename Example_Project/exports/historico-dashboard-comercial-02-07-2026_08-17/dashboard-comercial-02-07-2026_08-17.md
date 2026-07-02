# Dashboard Comercial

> 📘 Documentação técnica gerada automaticamente pelo **BI Documentation Studio**

| Indicador | Qtd. | Indicador | Qtd. |
|---|---|---|---|
| KPIs | 4 | Medidas DAX | 6 |
| Queries | 5 | Relacionamentos | 5 |
| Páginas | 2 | Visuais | 6 |
| Filtros | 4 | Termos no Glossário | 7 |

## 📑 Sumário

- [📋 Projeto](#projeto)
- [📊 KPIs](#kpis)
  - [Faturamento Líquido](#kpi-faturamento-liquido-a1b2c3d4)
  - [Atingimento de Meta](#kpi-atingimento-de-meta-b2c3d4e5)
  - [Ticket Médio](#kpi-ticket-medio-c3d4e5f6)
  - [Crescimento vs. Mês Anterior](#kpi-crescimento-vs-mes-anterior-d4e5f6a7)
- [🗄️ Queries](#queries)
  - [Tabelas Fato](#queries-fato)
    - [ftVendas](#query-ftvendas-e5f6a7b8)
    - [ftMetas](#query-ftmetas-a6b7c8d9)
  - [Tabelas Dimensão](#queries-dimensao)
    - [dimClientes](#query-dimclientes-b7c8d9e0)
    - [dimProdutos](#query-dimprodutos-c8d9e0f1)
    - [dimTempo](#query-dimtempo-d9e0f1a2)
- [🔗 Relacionamentos](#relacionamentos)
- [📐 Medidas DAX](#medidas-dax)
  - [_Medidas[Faturamento Líquido]](#medida-faturamento-liquido-d5e6f7a8)
  - [_Medidas[Faturamento Mês Anterior]](#medida-faturamento-mes-anterior-e6f7a8b9)
  - [_Medidas[Meta do Período]](#medida-meta-do-periodo-f7a8b9c0)
  - [_Medidas[% Atingimento de Meta]](#medida--atingimento-de-meta-a8b9c0d1)
  - [_Medidas[Ticket Médio]](#medida-ticket-medio-b9c0d1e2)
  - [_Medidas[Faturamento YTD]](#medida-faturamento-ytd-c0d1e2f3)
- [📄 Páginas](#paginas)
  - [Visão Geral](#pagina-visao-geral-d1e2f3a4)
  - [Análise por Produto](#pagina-analise-por-produto-f9a0b1c2)
- [📖 Glossário](#glossario)

---

<a id="projeto"></a>
## 📋 Projeto

| Campo | Valor |
|---|---|
| **Nome do Relatório** | Dashboard Comercial |
| **Área / Departamento** | Comercial / Vendas |
| **Responsável** | Ana Paula Ferreira |
| **Data de Criação** | 01/2026 |
| **Última Atualização** | 07/2026 |
| **Versão do Schema** | 1.0.0 |

### Objetivo

Consolidar os principais indicadores de performance comercial da organização, permitindo o acompanhamento em tempo real do faturamento, atingimento de metas, ticket médio e evolução das vendas por região, produto e vendedor. O relatório apoia a tomada de decisão da gestão comercial e da diretoria executiva.

### Descrição Geral

O Dashboard Comercial centraliza informações oriundas do ERP e do CRM, cobrindo todo o ciclo de vendas: desde o registro do pedido até o faturamento efetivo.

O relatório é atualizado diariamente via Power BI Service e contempla análises por período, região, canal de vendas, linha de produto e vendedor. As metas são importadas de uma planilha Excel gerenciada pela equipe de Planejamento Comercial.

O modelo segue a arquitetura Star Schema, com uma tabela fato de vendas (ftVendas) e tabelas dimensão de clientes, produtos, vendedores, região e tempo. Uma segunda tabela fato (ftMetas) armazena as metas mensais por vendedor e produto, permitindo análises de atingimento.

### Fontes de Dados

- **SQL Server** — ERP Corporativo — base COMERCIAL_PROD
- **Planilha Excel** — Metas_Comerciais.xlsx — gerenciada pelo Planejamento Comercial

### Observações Gerais

O relatório não contempla vendas canceladas após faturamento. Devoluções e notas de crédito são tratadas como eventos negativos na tabela ftVendas, com o campo TipoMovimento = 'Devolucao'.

As metas são carregadas manualmente uma vez por mês pela equipe de Planejamento. Em caso de revisão de metas no meio do período, o arquivo Excel deve ser atualizado e o dataset republicado manualmente.

Colaboradores com perfil de Vendedor visualizam apenas os dados de sua própria carteira via RLS (Row-Level Security) configurado no Power BI Service.

### 💡 Possíveis Melhorias / Atualizações Futuras

- Implementar RLS dinâmico por hierarquia de gestores (Vendedor → Supervisor → Gerente)
- Adicionar análise de churn de clientes com base na recência de compras
- Integrar dados do CRM para incluir oportunidades e funil de vendas
- Criar página dedicada de análise de rentabilidade por produto e canal
- Automatizar a carga das metas via API da planilha no SharePoint

---

<a id="kpis"></a>
## 📊 KPIs (4)

<a id="kpi-faturamento-liquido-a1b2c3d4"></a>
### Faturamento Líquido

**Tipo de visual:** Card

**O que mede:** Soma do valor líquido de todas as vendas efetivadas no período selecionado, descontados impostos, devoluções e cancelamentos.

**Objetivo / Meta:** Acompanhar a receita líquida gerada pela operação comercial e comparar com a meta do período. Meta anual definida pelo planejamento estratégico.

**Fórmula:**

> SUM(ftVendas[ValorLiquido]) onde TipoMovimento = 'Venda'

#### Escopo do Cálculo

- ✅ **O que entra:** Notas fiscais de venda com status 'Faturado'.
Vendas de todas as filiais e canais (direto, distribuidor, e-commerce).
Mercado interno e exportação.
- ❌ **O que não entra:** Devoluções e notas de crédito (TipoMovimento = 'Devolucao').
Pedidos ainda não faturados (status 'Em Carteira' ou 'Aprovado').
Vendas canceladas antes do faturamento.
- ⚠️ **Exceções:** Vendas intercompany (entre empresas do mesmo grupo) são incluídas no faturamento bruto, mas excluídas no indicador consolidado de grupo quando o filtro 'Visão Consolidada' está ativo.

**Regras Temporais:** Considera a data de faturamento (DataNF), não a data do pedido. Para análises mensais, acumula todas as notas fiscais emitidas dentro do mês selecionado. O comparativo com mês anterior é calculado automaticamente pela medida de variação.

| | |
|---|---|
| **Fonte dos Dados** | ERP Corporativo — tabela fato ftVendas |
| **Responsável pela Validação** | Ana Paula Ferreira — Gestão Comercial |

**Regras de Negócio:**
- Considera apenas notas fiscais com status 'Faturado' no ERP.
- O valor líquido já desconta IPI, ICMS ST e descontos comerciais concedidos.
- Devoluções reduzem o faturamento no mês em que a nota de devolução é emitida, não no mês da venda original.

**Observações:** Principal KPI do relatório. Variações abruptas no indicador podem indicar problemas de integração entre o ERP e o Power BI — verificar se o dataset foi atualizado corretamente antes de interpretar quedas expressivas.

<details>
<summary>🔗 Referências</summary>

**Medidas DAX relacionadas:**
- [_Medidas[Faturamento Líquido]](#medida-faturamento-liquido-d5e6f7a8)
- [_Medidas[Faturamento YTD]](#medida-faturamento-ytd-c0d1e2f3)

**Utilizado nos visuais:**
- [Faturamento Líquido](#visual-faturamento-liquido-e2f3a4b5) _(página [Visão Geral](#pagina-visao-geral-d1e2f3a4))_
- [Evolução Faturamento vs Meta](#visual-evolucao-faturamento-vs-meta-a4b5c6d7) _(página [Visão Geral](#pagina-visao-geral-d1e2f3a4))_
- [Ranking de Vendedores](#visual-ranking-de-vendedores-b5c6d7e8) _(página [Visão Geral](#pagina-visao-geral-d1e2f3a4))_
- [Faturamento por Família](#visual-faturamento-por-familia-a0b1c2d3) _(página [Análise por Produto](#pagina-analise-por-produto-f9a0b1c2))_
- [Top 10 Produtos](#visual-top-10-produtos-b1c2d3e4) _(página [Análise por Produto](#pagina-analise-por-produto-f9a0b1c2))_

</details>

<a id="kpi-atingimento-de-meta-b2c3d4e5"></a>
### Atingimento de Meta

**Tipo de visual:** Gauge

**O que mede:** Percentual do faturamento líquido realizado em relação à meta comercial definida para o período selecionado.

**Objetivo / Meta:** Monitorar se a operação está no ritmo necessário para atingir a meta mensal e anual. Meta mínima aceitável: 90%. Acima de 100% indica superação.

**Fórmula:**

> DIVIDE([Faturamento Líquido], [Meta do Período], 0)

#### Escopo do Cálculo

- ✅ **O que entra:** Faturamento líquido do período.
Meta comercial aprovada para o mesmo período e granularidade (vendedor, produto, região).
- ❌ **O que não entra:** Períodos sem meta cadastrada retornam BLANK — não são tratados como meta zero.
- ⚠️ **Exceções:** Meses com revisão de meta aprovada pela diretoria utilizam a meta revisada, não a original. O histórico de revisões não é mantido no modelo atual.

**Regras Temporais:** Calculado para o período selecionado nos filtros. Para análise YTD, utiliza a medida específica de Atingimento Acumulado.

| | |
|---|---|
| **Fonte dos Dados** | ERP (realizado) e Excel Metas_Comerciais.xlsx (meta) |
| **Responsável pela Validação** | Carlos Mendes — Planejamento Comercial |

**Regras de Negócio:**
- A meta é definida mensalmente por vendedor e linha de produto.
- O denominador da divisão é protegido contra zero — retorna BLANK quando não há meta cadastrada.
- Consolidações regionais e de produto somam metas individuais — não há meta agregada separada.

**Observações:** O visual de gauge exibe faixas coloridas: vermelho (< 80%), amarelo (80–99%), verde (≥ 100%). Os limites das faixas são parametrizáveis via tabela de configuração no modelo.

<details>
<summary>🔗 Referências</summary>

**Medidas DAX relacionadas:**
- [_Medidas[Meta do Período]](#medida-meta-do-periodo-f7a8b9c0)
- [_Medidas[% Atingimento de Meta]](#medida--atingimento-de-meta-a8b9c0d1)

**Utilizado nos visuais:**
- [Atingimento de Meta](#visual-atingimento-de-meta-f3a4b5c6) _(página [Visão Geral](#pagina-visao-geral-d1e2f3a4))_
- [Evolução Faturamento vs Meta](#visual-evolucao-faturamento-vs-meta-a4b5c6d7) _(página [Visão Geral](#pagina-visao-geral-d1e2f3a4))_
- [Ranking de Vendedores](#visual-ranking-de-vendedores-b5c6d7e8) _(página [Visão Geral](#pagina-visao-geral-d1e2f3a4))_
- [Faturamento por Família](#visual-faturamento-por-familia-a0b1c2d3) _(página [Análise por Produto](#pagina-analise-por-produto-f9a0b1c2))_

</details>

<a id="kpi-ticket-medio-c3d4e5f6"></a>
### Ticket Médio

**Tipo de visual:** Card

**O que mede:** Valor médio de faturamento por pedido/nota fiscal no período selecionado.

**Objetivo / Meta:** Monitorar a evolução do valor médio por transação, identificando tendências de upsell, downsell ou mudança no mix de produtos.

**Fórmula:**

> DIVIDE([Faturamento Líquido], [Quantidade de Pedidos], 0)

#### Escopo do Cálculo

- ✅ **O que entra:** Faturamento líquido total do período.
Quantidade de notas fiscais de venda únicas (DISTINCTCOUNT de NFNumero).
- ❌ **O que não entra:** Notas de devolução não são contadas como pedidos, mas reduzem o faturamento líquido do numerador.
- ⚠️ **Exceções:** Pedidos fracionados em múltiplas notas fiscais (por questão logística ou fiscal) são contados como pedidos distintos, o que pode reduzir artificialmente o ticket médio em períodos com alto volume de fracionamento.

**Regras Temporais:** Calculado para o período filtrado. Não acumula — é sempre a média do período selecionado.

| | |
|---|---|
| **Fonte dos Dados** | ERP Corporativo — tabela fato ftVendas |
| **Responsável pela Validação** | Ana Paula Ferreira — Gestão Comercial |

**Regras de Negócio:**
- Considera a nota fiscal como unidade de pedido.
- O denominador é protegido contra zero via DIVIDE com valor alternativo 0.

**Observações:** Quedas no ticket médio combinadas com crescimento em volume de pedidos podem indicar pulverização de clientes ou promoções agressivas — analisar em conjunto com o mix de produtos e canal de venda.

<details>
<summary>🔗 Referências</summary>

**Medidas DAX relacionadas:**
- [_Medidas[Ticket Médio]](#medida-ticket-medio-b9c0d1e2)

**Utilizado nos visuais:**
- [Top 10 Produtos](#visual-top-10-produtos-b1c2d3e4) _(página [Análise por Produto](#pagina-analise-por-produto-f9a0b1c2))_

</details>

<a id="kpi-crescimento-vs-mes-anterior-d4e5f6a7"></a>
### Crescimento vs. Mês Anterior

**Tipo de visual:** Kpi nativo

**O que mede:** Variação percentual do faturamento líquido do mês atual em relação ao mês imediatamente anterior.

**Objetivo / Meta:** Identificar tendências de crescimento ou retração mês a mês, sinalizando acelerações ou desacelerações na operação comercial.

**Fórmula:**

> DIVIDE([Faturamento Líquido] - [Faturamento Mês Anterior], [Faturamento Mês Anterior], BLANK())

#### Escopo do Cálculo

- ✅ **O que entra:** Faturamento líquido do mês atual (conforme filtro).
Faturamento líquido do mês imediatamente anterior.
- ❌ **O que não entra:** Não é calculado quando não existe mês anterior disponível no modelo (ex: primeiro mês histórico da base).
- ⚠️ **Exceções:** Quando o filtro de data abrange múltiplos meses, a medida utiliza o mês mais recente como referência e o penúltimo como base de comparação.

**Regras Temporais:** Sempre compara o período atual com o período imediatamente anterior de mesma granularidade (mês vs. mês anterior). Não é uma comparação YoY.

| | |
|---|---|
| **Fonte dos Dados** | ERP Corporativo — tabela fato ftVendas |
| **Responsável pela Validação** | Ana Paula Ferreira — Gestão Comercial |

**Regras de Negócio:**
- O mês anterior é calculado usando DATEADD ou PREVIOUSMONTH da dimTempo.
- Retorna BLANK quando não há dados para o mês anterior — não exibe zero para não confundir com variação nula.

**Observações:** O visual KPI Nativo exibe seta verde (crescimento) ou vermelha (retração) automaticamente. A meta de referência exibida no visual é o faturamento do mês anterior.

<details>
<summary>🔗 Referências</summary>

**Medidas DAX relacionadas:**
- [_Medidas[Faturamento Mês Anterior]](#medida-faturamento-mes-anterior-e6f7a8b9)

</details>

---

<a id="queries"></a>
## 🗄️ Queries / Tabelas (5)

<a id="queries-fato"></a>
### 🟦 Tabelas Fato

<a id="query-ftvendas-e5f6a7b8"></a>
#### ftVendas

**Fonte:** SQL Server


Tabela fato central do modelo, contendo todos os eventos de venda e devolução faturados. Cada linha representa um item de nota fiscal, com seu valor líquido, impostos, quantidades e referências às dimensões. É a fonte de todos os KPIs de faturamento, volume e ticket médio.


<details>
<summary>📄 Ver código (SQL / M)</summary>

```sql
SELECT
    NF.NumNF                                    AS NFNumero,
    NF.DataEmissao                              AS DataNF,
    NF.TipoMovimento,
    CONCAT(CLI.CodCli, '.', CLI.CodLoja)        AS PKCliente,
    CONCAT(PROD.CodDep, '.', PROD.CodPro)       AS PKProduto,
    CONCAT(VEN.CodEmp, '.', VEN.CodVen)         AS PKVendedor,
    REG.CodRegiao                               AS PKRegiao,
    SUM(ITEM.Quantidade)                        AS Quantidade,
    SUM(ITEM.ValorBruto)                        AS ValorBruto,
    SUM(ITEM.ValorDesconto)                     AS ValorDesconto,
    SUM(ITEM.ValorLiquido)                      AS ValorLiquido,
    SUM(ITEM.ValorIPI)                          AS ValorIPI,
    SUM(ITEM.ValorICMSST)                       AS ValorICMSST

FROM
    NotasFiscais NF
    INNER JOIN ItensNF        ITEM ON NF.NumNF = ITEM.NumNF
    INNER JOIN Clientes       CLI  ON NF.CodCli = CLI.CodCli AND NF.CodLoja = CLI.CodLoja
    INNER JOIN Produtos       PROD ON ITEM.CodPro = PROD.CodPro AND ITEM.CodDep = PROD.CodDep
    INNER JOIN Vendedores     VEN  ON NF.CodVen = VEN.CodVen AND NF.CodEmp = VEN.CodEmp
    LEFT JOIN  RegiaoCliente  REG  ON CLI.CodRegiao = REG.CodRegiao

WHERE
    NF.DataEmissao >= '2023-01-01'
    AND NF.TipoMovimento IN ('Venda', 'Devolucao')
    AND NF.Status = 'Faturado'

GROUP BY
    NF.NumNF, NF.DataEmissao, NF.TipoMovimento,
    CLI.CodCli, CLI.CodLoja,
    PROD.CodDep, PROD.CodPro,
    VEN.CodEmp, VEN.CodVen,
    REG.CodRegiao
```

</details>

**Transformações Power Query:**
- = Table.TransformColumnTypes(Fonte, {{"DataNF", type date}, {"Quantidade", type number}, {"ValorBruto", type number}, {"ValorLiquido", type number}})
- = Table.AddColumn(TipoAlterado, "AnoMes", each Date.Year([DataNF]) * 100 + Date.Month([DataNF]), Int64.Type)

**Colunas Principais:**

| Coluna | Tipo | Descrição |
|---|---|---|
| `NFNumero` | Text | Número da nota fiscal — identificador único do documento fiscal |
| `DataNF` | Date | Data de emissão da nota fiscal — base para todos os filtros temporais |
| `TipoMovimento` | Text | Classificação do documento: 'Venda' ou 'Devolucao' |
| `PKCliente` | Text | Chave do cliente no formato CodCli.CodLoja — relaciona com dimClientes |
| `PKProduto` | Text | Chave do produto no formato CodDep.CodPro — relaciona com dimProdutos |
| `PKVendedor` | Text | Chave do vendedor no formato CodEmp.CodVen — relaciona com dimVendedores |
| `ValorLiquido` | Decimal | Valor líquido do item após descontos — base para o KPI de Faturamento |
| `Quantidade` | Decimal | Quantidade de unidades vendidas ou devolvidas |

**Observações:** A query agrupa por nota fiscal e item para reduzir a granularidade e melhorar a performance no Power BI. O filtro de data ('2023-01-01') define o horizonte histórico — deve ser revisado anualmente. A ausência de LEFT JOIN na tabela de produtos pode excluir itens descontinuados se não estiverem na dimensão ativa.

<a id="query-ftmetas-a6b7c8d9"></a>
#### ftMetas

**Fonte:** Excel / CSV


Tabela fato de metas comerciais, carregada a partir da planilha Metas_Comerciais.xlsx gerenciada pelo Planejamento Comercial. Contém as metas mensais por vendedor e linha de produto, sendo a base para os KPIs de atingimento.


<details>
<summary>📄 Ver código (SQL / M)</summary>

```sql
let
    Fonte = Excel.Workbook(File.Contents("C:/BI/Comercial/Metas_Comerciais.xlsx"), null, true),
    Metas_Table = Fonte{[Item="Metas", Kind="Table"]}[Data]
in
    Metas_Table
```

</details>

**Transformações Power Query:**
- = Table.TransformColumnTypes(Fonte, {{"AnoCom", Int64.Type}, {"MesCom", Int64.Type}, {"MetaValor", type number}})
- = Table.AddColumn(TipoAlterado, "AnoMes", each [AnoCom] * 100 + [MesCom], Int64.Type)
- = Table.AddColumn(AnoMesAdd, "PKVendedor", each Text.From([CodEmp]) & "." & Text.From([CodVen]), type text)

**Colunas Principais:**

| Coluna | Tipo | Descrição |
|---|---|---|
| `AnoMes` | INT | Chave temporal no formato AAAAMM para relacionamento com dimTempo |
| `PKVendedor` | Text | Chave do vendedor — deve corresponder exatamente ao PKVendedor da ftVendas |
| `MetaValor` | Decimal | Valor da meta em reais para o vendedor no mês de referência |

**Observações:** O caminho do arquivo é hardcoded — em ambiente de produção recomenda-se migrar para SharePoint ou substituir por query parametrizada. Revisões de meta no meio do mês exigem atualização manual da planilha e republicação do dataset.

<a id="queries-dimensao"></a>
### 🟩 Tabelas Dimensão

<a id="query-dimclientes-b7c8d9e0"></a>
#### dimClientes

**Fonte:** SQL Server


Dimensão de clientes contendo informações cadastrais, segmentação por porte, região, canal de atendimento e classificação ABC. Utilizada para análises de carteira, concentração e perfil da base de clientes.


<details>
<summary>📄 Ver código (SQL / M)</summary>

```sql
SELECT
    CONCAT(CLI.CodCli, '.', CLI.CodLoja)    AS PKCliente,
    CLI.NomCli                              AS NomeCliente,
    CLI.CNPJ,
    CLI.CidCli                              AS Cidade,
    CLI.EstCli                              AS Estado,
    REG.DesRegiao                           AS Regiao,
    SEG.DesSegmento                         AS Segmento,
    CASE
        WHEN CLI.FatAnual >= 5000000  THEN 'Grande'
        WHEN CLI.FatAnual >= 500000   THEN 'Medio'
        ELSE 'Pequeno'
    END                                     AS Porte,
    COALESCE(ABC.ClassABC, 'D')             AS ClassificacaoABC

FROM
    Clientes         CLI
    LEFT JOIN Regioes    REG ON CLI.CodRegiao = REG.CodRegiao
    LEFT JOIN Segmentos  SEG ON CLI.CodSegmento = SEG.CodSegmento
    LEFT JOIN ClienteABC ABC ON CLI.CodCli = ABC.CodCli

WHERE
    CLI.Ativo = 'S'
```

</details>

**Colunas Principais:**

| Coluna | Tipo | Descrição |
|---|---|---|
| `PKCliente` | Text | Chave primária da dimensão — relaciona com ftVendas[PKCliente] |
| `ClassificacaoABC` | Text | Classificação ABC do cliente baseada em faturamento anual — A (top 20%), B (20–50%), C (50–80%), D (restante) |
| `Porte` | Text | Segmentação por porte baseada no faturamento anual declarado: Grande, Médio ou Pequeno |

**Observações:** A query filtra apenas clientes ativos. Clientes inativos que ainda possuem histórico de vendas aparecem na ftVendas mas não na dimensão — isso pode causar linhas em branco nos visuais se não houver tratamento no modelo.

<a id="query-dimprodutos-c8d9e0f1"></a>
#### dimProdutos

**Fonte:** SQL Server


Dimensão de produtos com hierarquia de categorização (Família → Linha → Produto), margem de contribuição média e indicadores de atividade. Permite análises de mix de vendas, rentabilidade por categoria e performance de linha.


<details>
<summary>📄 Ver código (SQL / M)</summary>

```sql
SELECT
    CONCAT(PRD.CodDep, '.', PRD.CodPro)     AS PKProduto,
    PRD.DesPro                              AS NomeProduto,
    PRD.CodBarras,
    LIN.DesLinha                            AS Linha,
    FAM.DesFamilia                          AS Familia,
    PRD.PercMargem                          AS MargemMedia,
    CASE
        WHEN PRD.DtUltVenda >= DATEADD(MONTH, -3, GETDATE()) THEN 'Ativo'
        WHEN PRD.DtUltVenda >= DATEADD(MONTH, -6, GETDATE()) THEN 'Baixo Giro'
        ELSE 'Inativo'
    END                                     AS StatusGiro

FROM
    Produtos     PRD
    LEFT JOIN LinhasProduto LIN ON PRD.CodLinha = LIN.CodLinha
    LEFT JOIN Familias     FAM ON LIN.CodFamilia = FAM.CodFamilia
```

</details>

**Transformações Power Query:**
- = Table.TransformColumnTypes(Fonte, {{"MargemMedia", type number}})

**Colunas Principais:**

| Coluna | Tipo | Descrição |
|---|---|---|
| `PKProduto` | Text | Chave primária da dimensão no formato CodDep.CodPro |
| `StatusGiro` | Text | Classificação de atividade baseada na data da última venda: Ativo (< 3 meses), Baixo Giro (3–6 meses), Inativo (> 6 meses) |
| `MargemMedia` | Decimal | Margem de contribuição média percentual do produto — utilizada em análises de rentabilidade |

**Observações:** O cálculo de StatusGiro é dinâmico no SQL e reflete o estado atual no momento da atualização. Para histórico de giro em datas específicas seria necessário uma tabela de snapshots mensais.

<details>
<summary>🔗 Utilizado nos visuais</summary>

- [Faturamento por Família](#visual-faturamento-por-familia-a0b1c2d3) _(página [Análise por Produto](#pagina-analise-por-produto-f9a0b1c2))_
- [Top 10 Produtos](#visual-top-10-produtos-b1c2d3e4) _(página [Análise por Produto](#pagina-analise-por-produto-f9a0b1c2))_

</details>

<a id="query-dimtempo-d9e0f1a2"></a>
#### dimTempo

**Fonte:** Power Query


Dimensão de tempo gerada via Power Query, cobrindo todos os dias do período histórico do relatório. Contém atributos de dia, semana, mês, trimestre, semestre e ano, além de classificações de dia útil e feriado. É a tabela de datas padrão do modelo.


<details>
<summary>📄 Ver código (SQL / M)</summary>

```sql
let
    DataInicio = #date(2023, 1, 1),
    DataFim    = Date.From(DateTime.FixedLocalNow()),
    Duracao    = Duration.Days(DataFim - DataInicio) + 1,
    Fonte      = List.Dates(DataInicio, Duracao, #duration(1, 0, 0, 0)),
    Tabela     = Table.FromList(Fonte, Splitter.SplitByNothing(), {"Data"}),
    TipoData   = Table.TransformColumnTypes(Tabela, {{"Data", type date}}),
    AddAno     = Table.AddColumn(TipoData,  "Ano",       each Date.Year([Data]),          Int64.Type),
    AddMes     = Table.AddColumn(AddAno,    "Mes",       each Date.Month([Data]),         Int64.Type),
    AddNomeMes = Table.AddColumn(AddMes,    "NomeMes",   each Text.Proper(Date.MonthName([Data])), type text),
    AddTrim    = Table.AddColumn(AddNomeMes,"Trimestre", each Date.QuarterOfYear([Data]), Int64.Type),
    AddAnoMes  = Table.AddColumn(AddTrim,   "AnoMes",    each Date.Year([Data]) * 100 + Date.Month([Data]), Int64.Type)
in
    AddAnoMes
```

</details>

**Colunas Principais:**

| Coluna | Tipo | Descrição |
|---|---|---|
| `Data` | Date | Data base da dimensão — chave primária para relacionamentos com tabelas fato |
| `AnoMes` | INT | Chave no formato AAAAMM para relacionamento com ftMetas |
| `Trimestre` | INT | Número do trimestre (1 a 4) |

**Observações:** A data máxima é dinâmica (data atual). Para análises de meses futuros com metas cadastradas, estender DataFim para o final do ano fiscal.

<details>
<summary>🔗 Utilizado nos visuais</summary>

- [Evolução Faturamento vs Meta](#visual-evolucao-faturamento-vs-meta-a4b5c6d7) _(página [Visão Geral](#pagina-visao-geral-d1e2f3a4))_

</details>

---

<a id="relacionamentos"></a>
## 🔗 Relacionamentos (5)

| Origem | Destino | Col. Origem | Col. Destino | Cardinalidade | Direção | Ativo | Temp. | Observações |
|---|---|---|---|---|---|---|---|---|
| [`ftVendas`](#query-ftvendas-e5f6a7b8) | [`dimTempo`](#query-dimtempo-d9e0f1a2) | `DataNF` | `Data` | Muitos para Um (*:1) | Única (→) | ✅ | — | Relacionamento principal de tempo. Ativo e com direção única para evitar ambiguidade com possíveis relacionamentos de datas de entrega futuros. |
| [`ftVendas`](#query-ftvendas-e5f6a7b8) | [`dimClientes`](#query-dimclientes-b7c8d9e0) | `PKCliente` | `PKCliente` | Muitos para Um (*:1) | Única (→) | ✅ | — | Clientes sem cadastro ativo na dimensão aparecem como BLANK nos visuais segmentados por cliente. |
| [`ftVendas`](#query-ftvendas-e5f6a7b8) | [`dimProdutos`](#query-dimprodutos-c8d9e0f1) | `PKProduto` | `PKProduto` | Muitos para Um (*:1) | Única (→) | ✅ | — |  |
| [`ftMetas`](#query-ftmetas-a6b7c8d9) | [`dimTempo`](#query-dimtempo-d9e0f1a2) | `AnoMes` | `AnoMes` | Muitos para Um (*:1) | Única (→) | ✅ | — | Relacionamento via AnoMes (AAAAMM) pois as metas têm granularidade mensal, não diária. |
| [`ftVendas`](#query-ftvendas-e5f6a7b8) | [`dimClientes`](#query-dimclientes-b7c8d9e0) | `PKCliente` | `PKCliente` | Muitos para Um (*:1) | Bidirecional (↔) | ❌ | ⚡ | Relacionamento bidirecional temporário, utilizado via USERELATIONSHIP em medidas específicas de análise de carteira que precisam filtrar fatos a partir da dimensão de clientes. |

> ⚡ **Relacionamentos temporários** são ativados via `USERELATIONSHIP()` em medidas DAX específicas e não estão ativos no modelo por padrão.

---

<a id="medidas-dax"></a>
## 📐 Medidas DAX (6)

<a id="medida-faturamento-liquido-d5e6f7a8"></a>
### _Medidas[Faturamento Líquido]

Faturamento líquido total do período, considerando apenas notas de venda (excluindo devoluções). Medida base do modelo — quase todas as outras medidas dependem desta.


```dax
Faturamento Líquido =
CALCULATE(
    SUM(ftVendas[ValorLiquido]),
    ftVendas[TipoMovimento] = "Venda"
)
```

**KPIs relacionados:**
- [Faturamento Líquido](#kpi-faturamento-liquido-a1b2c3d4)

**Como validar:** Comparar o valor retornado com a soma do campo ValorLiquido na ftVendas filtrada por TipoMovimento = 'Venda' diretamente no SQL. Qualquer discrepância indica problema de relacionamento ou filtro de contexto.

<details>
<summary>🧪 Query de Validação</summary>

```sql
SELECT
    FORMAT(DataNF, 'yyyy-MM') AS AnoMes,
    SUM(ValorLiquido)         AS FaturamentoLiquido
FROM ftVendas
WHERE TipoMovimento = 'Venda'
GROUP BY FORMAT(DataNF, 'yyyy-MM')
ORDER BY 1
```

</details>

<details>
<summary>🔗 Referências</summary>

**Utilizada por outras medidas:**
- [_Medidas[Faturamento Mês Anterior]](#medida-faturamento-mes-anterior-e6f7a8b9)
- [_Medidas[% Atingimento de Meta]](#medida--atingimento-de-meta-a8b9c0d1)
- [_Medidas[Ticket Médio]](#medida-ticket-medio-b9c0d1e2)
- [_Medidas[Faturamento YTD]](#medida-faturamento-ytd-c0d1e2f3)

**Utilizada nos visuais:**
- [Faturamento Líquido](#visual-faturamento-liquido-e2f3a4b5) _(página [Visão Geral](#pagina-visao-geral-d1e2f3a4))_
- [Evolução Faturamento vs Meta](#visual-evolucao-faturamento-vs-meta-a4b5c6d7) _(página [Visão Geral](#pagina-visao-geral-d1e2f3a4))_
- [Ranking de Vendedores](#visual-ranking-de-vendedores-b5c6d7e8) _(página [Visão Geral](#pagina-visao-geral-d1e2f3a4))_
- [Faturamento por Família](#visual-faturamento-por-familia-a0b1c2d3) _(página [Análise por Produto](#pagina-analise-por-produto-f9a0b1c2))_
- [Top 10 Produtos](#visual-top-10-produtos-b1c2d3e4) _(página [Análise por Produto](#pagina-analise-por-produto-f9a0b1c2))_

</details>

<a id="medida-faturamento-mes-anterior-e6f7a8b9"></a>
### _Medidas[Faturamento Mês Anterior]

Faturamento líquido do mês imediatamente anterior ao período selecionado. Utilizada como base para o cálculo de variação e como target no visual KPI Nativo.


```dax
Faturamento Mês Anterior =
CALCULATE(
    [Faturamento Líquido],
    PREVIOUSMONTH(dimTempo[Data])
)
```

**Dependências:**
- [_Medidas[Faturamento Líquido]](#medida-faturamento-liquido-d5e6f7a8)

**KPIs relacionados:**
- [Crescimento vs. Mês Anterior](#kpi-crescimento-vs-mes-anterior-d4e5f6a7)

**Como validar:** Com o filtro em Março/2026, a medida deve retornar o faturamento de Fevereiro/2026. Validar cruzando diretamente na ftVendas com filtro de data explícito.

<details>
<summary>🔗 Referências</summary>

**Utilizada nos visuais:**
- [Faturamento Líquido](#visual-faturamento-liquido-e2f3a4b5) _(página [Visão Geral](#pagina-visao-geral-d1e2f3a4))_

</details>

<a id="medida-meta-do-periodo-f7a8b9c0"></a>
### _Medidas[Meta do Período]

Soma das metas comerciais cadastradas para o período selecionado. Considera apenas a dimensão de tempo e vendedor no contexto de filtro.


```dax
Meta do Período =
SUM(ftMetas[MetaValor])
```

**KPIs relacionados:**
- [Atingimento de Meta](#kpi-atingimento-de-meta-b2c3d4e5)

**Como validar:** Com filtro no mês de Janeiro/2026 e um vendedor específico, deve retornar exatamente o valor cadastrado na planilha de metas para aquela combinação. Se retornar BLANK, a combinação AnoMes + PKVendedor não existe na ftMetas.

<details>
<summary>🔗 Referências</summary>

**Utilizada por outras medidas:**
- [_Medidas[% Atingimento de Meta]](#medida--atingimento-de-meta-a8b9c0d1)

**Utilizada nos visuais:**
- [Evolução Faturamento vs Meta](#visual-evolucao-faturamento-vs-meta-a4b5c6d7) _(página [Visão Geral](#pagina-visao-geral-d1e2f3a4))_

</details>

<a id="medida--atingimento-de-meta-a8b9c0d1"></a>
### _Medidas[% Atingimento de Meta]

Percentual de atingimento da meta comercial no período. Retorna BLANK quando não há meta cadastrada para evitar distorção nos visuais.


```dax
% Atingimento de Meta =
DIVIDE(
    [Faturamento Líquido],
    [Meta do Período]
)
```

**Dependências:**
- [_Medidas[Faturamento Líquido]](#medida-faturamento-liquido-d5e6f7a8)
- [_Medidas[Meta do Período]](#medida-meta-do-periodo-f7a8b9c0)

**KPIs relacionados:**
- [Atingimento de Meta](#kpi-atingimento-de-meta-b2c3d4e5)

**Como validar:** Com Faturamento = R$ 850.000 e Meta = R$ 1.000.000, deve retornar 85% (0,85). O visual de gauge deve exibir a cor amarela (entre 80% e 99%). Com Meta = BLANK, deve retornar BLANK.

<details>
<summary>🔗 Referências</summary>

**Utilizada nos visuais:**
- [Atingimento de Meta](#visual-atingimento-de-meta-f3a4b5c6) _(página [Visão Geral](#pagina-visao-geral-d1e2f3a4))_
- [Ranking de Vendedores](#visual-ranking-de-vendedores-b5c6d7e8) _(página [Visão Geral](#pagina-visao-geral-d1e2f3a4))_
- [Faturamento por Família](#visual-faturamento-por-familia-a0b1c2d3) _(página [Análise por Produto](#pagina-analise-por-produto-f9a0b1c2))_

</details>

<a id="medida-ticket-medio-b9c0d1e2"></a>
### _Medidas[Ticket Médio]

Valor médio por nota fiscal de venda no período. Calculado como faturamento líquido dividido pela contagem distinta de notas fiscais.


```dax
Ticket Médio =
DIVIDE(
    [Faturamento Líquido],
    CALCULATE(
        DISTINCTCOUNT(ftVendas[NFNumero]),
        ftVendas[TipoMovimento] = "Venda"
    ),
    0
)
```

**Dependências:**
- [_Medidas[Faturamento Líquido]](#medida-faturamento-liquido-d5e6f7a8)

**KPIs relacionados:**
- [Ticket Médio](#kpi-ticket-medio-c3d4e5f6)

**Como validar:** Dividir o faturamento total pelo número de notas fiscais de venda do período. Selecionar 10 notas aleatórias do SQL e calcular a média manualmente para validar o resultado global.

<details>
<summary>🧪 Query de Validação</summary>

```sql
SELECT
    COUNT(DISTINCT NFNumero)         AS QtdNotas,
    SUM(ValorLiquido)                AS Faturamento,
    SUM(ValorLiquido)
        / COUNT(DISTINCT NFNumero)   AS TicketMedio
FROM ftVendas
WHERE TipoMovimento = 'Venda'
  AND DataNF BETWEEN '2026-01-01' AND '2026-01-31'
```

</details>

<details>
<summary>🔗 Referências</summary>

**Utilizada nos visuais:**
- [Top 10 Produtos](#visual-top-10-produtos-b1c2d3e4) _(página [Análise por Produto](#pagina-analise-por-produto-f9a0b1c2))_

</details>

<a id="medida-faturamento-ytd-c0d1e2f3"></a>
### _Medidas[Faturamento YTD]

Faturamento líquido acumulado desde o início do ano até o último dia do período selecionado. Utilizado para análises de performance anual e projeção de fechamento.


```dax
Faturamento YTD =
CALCULATE(
    [Faturamento Líquido],
    DATESYTD(dimTempo[Data])
)
```

**Dependências:**
- [_Medidas[Faturamento Líquido]](#medida-faturamento-liquido-d5e6f7a8)

**KPIs relacionados:**
- [Faturamento Líquido](#kpi-faturamento-liquido-a1b2c3d4)

**Como validar:** Com filtro em Março/2026, deve retornar a soma do faturamento de Janeiro + Fevereiro + Março/2026. Confirmar somando os três meses individualmente.

---

<a id="paginas"></a>
## 📄 Páginas (2)

<a id="pagina-visao-geral-d1e2f3a4"></a>
### Visão Geral

**Objetivo:** Apresentar os principais KPIs comerciais do período selecionado em um painel executivo, permitindo avaliação rápida da performance de faturamento, atingimento de meta e ticket médio.


Página principal do dashboard, projetada para uso pela gestão comercial e diretoria. Exibe indicadores consolidados no topo, seguidos de um gráfico de evolução mensal do faturamento vs. meta e uma tabela de ranking por vendedor. Todos os visuais respondem ao filtro de período e região.


![Visão Geral](imagens/paginas/img_visao-geral_pagina.png)

#### 🌐 Filtros de Relatório (todas as páginas)

| Filtro | Campo | Descrição |
|---|---|---|
| Ano | `dimTempo[Ano]` | Filtro de relatório aplicado automaticamente para limitar os dados ao ano atual e ao ano anterior. Evita que dados históricos mais antigos impactem o desempenho de carregamento. |

#### Visuais (4)

<a id="visual-faturamento-liquido-e2f3a4b5"></a>
##### Faturamento Líquido

**Tipo:** Cartao

**Objetivo:** Exibir o faturamento líquido total do período com comparativo vs. mês anterior.


Cartão com o faturamento líquido do período filtrado, exibindo o valor absoluto e a variação percentual em relação ao mês anterior em formato de tendência.


![Faturamento Líquido](imagens/visuais/img_visao-geral_faturamento-liquido_visual.png)


**KPIs:** [Faturamento Líquido](#kpi-faturamento-liquido-a1b2c3d4)

**Medidas:** [_Medidas[Faturamento Líquido]](#medida-faturamento-liquido-d5e6f7a8), [_Medidas[Faturamento Mês Anterior]](#medida-faturamento-mes-anterior-e6f7a8b9)

<a id="visual-atingimento-de-meta-f3a4b5c6"></a>
##### Atingimento de Meta

**Tipo:** Gauge

**Objetivo:** Visualizar o percentual de atingimento da meta do período em formato gauge com faixas de cor.


Gauge exibindo o % de atingimento da meta. Faixas: vermelho (< 80%), amarelo (80–99%), verde (≥ 100%). O valor mínimo do gauge é 0% e o máximo é 150% para acomodar superações expressivas.


![Atingimento de Meta](imagens/visuais/img_visao-geral_atingimento-de-meta_visual.png)


**KPIs:** [Atingimento de Meta](#kpi-atingimento-de-meta-b2c3d4e5)

**Medidas:** [_Medidas[% Atingimento de Meta]](#medida--atingimento-de-meta-a8b9c0d1)

<a id="visual-evolucao-faturamento-vs-meta-a4b5c6d7"></a>
##### Evolução Faturamento vs Meta

**Tipo:** Colunas Agrupadas Linha

**Objetivo:** Comparar a evolução mensal do faturamento realizado versus a meta mensal nos últimos 12 meses.


Gráfico combinado com colunas para o faturamento realizado e linha para a meta do período, segmentado por mês. Permite identificar tendências sazonais e meses com desvio expressivo entre realizado e planejado.


![Evolução Faturamento vs Meta](imagens/visuais/img_visao-geral_evolucao-faturamento-vs-meta_visual.png)


**KPIs:** [Faturamento Líquido](#kpi-faturamento-liquido-a1b2c3d4), [Atingimento de Meta](#kpi-atingimento-de-meta-b2c3d4e5)

**Medidas:** [_Medidas[Faturamento Líquido]](#medida-faturamento-liquido-d5e6f7a8), [_Medidas[Meta do Período]](#medida-meta-do-periodo-f7a8b9c0)

**Tabelas:** [dimTempo](#query-dimtempo-d9e0f1a2)

**Campos:** `NomeMes`, `Ano`

**Observações:** O eixo Y primário (colunas) é o faturamento em R$. O eixo Y secundário (linha) é a meta, também em R$. Ambos na mesma escala para facilitar comparação visual.

<a id="visual-ranking-de-vendedores-b5c6d7e8"></a>
##### Ranking de Vendedores

**Tipo:** Barras Clusterizado

**Objetivo:** Identificar os vendedores com maior faturamento no período e seu percentual de atingimento de meta.


Gráfico de barras horizontais ordenado decrescentemente por faturamento líquido, exibindo o nome do vendedor, o valor realizado e o percentual de atingimento de meta. Limitado ao Top 10 por padrão para manter legibilidade.


![Ranking de Vendedores](imagens/visuais/img_visao-geral_ranking-de-vendedores_visual.png)


**KPIs:** [Faturamento Líquido](#kpi-faturamento-liquido-a1b2c3d4), [Atingimento de Meta](#kpi-atingimento-de-meta-b2c3d4e5)

**Medidas:** [_Medidas[Faturamento Líquido]](#medida-faturamento-liquido-d5e6f7a8), [_Medidas[% Atingimento de Meta]](#medida--atingimento-de-meta-a8b9c0d1)

**Campos:** `NomeVendedor`

**Observações:** Em contexto de RLS, cada vendedor visualiza apenas sua própria barra. Gerentes visualizam todos os vendedores da sua equipe.

#### Filtros de Página (2)

| Filtro | Tipo | Campo | Descrição |
|---|---|---|---|
| Período | Slicer | `dimTempo[Data]` | Slicer de data com seleção por intervalo. Por padrão exibe o mês atual. Afeta todos os visuais da página. |
| Região | Slicer | `dimClientes[Regiao]` | Slicer de seleção múltipla por região. Permite analisar a performance de uma ou mais regiões comerciais isoladamente. |

<a id="pagina-analise-por-produto-f9a0b1c2"></a>
### Análise por Produto

**Objetivo:** Analisar a performance de vendas por linha de produto, família e SKU, identificando os itens de maior faturamento, melhor margem e tendência de crescimento.


Página focada no mix de produtos e rentabilidade. Permite ao gestor identificar quais linhas e produtos contribuem mais para o faturamento, quais possuem maior margem de contribuição e quais estão com giro baixo ou em declínio. Complementa a análise da página Visão Geral com drill-down na dimensão de produtos.


![Análise por Produto](imagens/paginas/img_analise-por-produto_pagina.png)

#### Visuais (2)

<a id="visual-faturamento-por-familia-a0b1c2d3"></a>
##### Faturamento por Família

**Tipo:** Treemap

**Objetivo:** Visualizar a participação de cada família de produtos no faturamento total de forma proporcional.


Treemap com cada retângulo representando uma família de produtos, dimensionado pelo faturamento líquido. A intensidade de cor representa o percentual de atingimento de meta daquela família — verde para acima da meta, vermelho para abaixo.


![Faturamento por Família](imagens/visuais/img_analise-por-produto_faturamento-por-familia_visual.png)


**KPIs:** [Faturamento Líquido](#kpi-faturamento-liquido-a1b2c3d4), [Atingimento de Meta](#kpi-atingimento-de-meta-b2c3d4e5)

**Medidas:** [_Medidas[Faturamento Líquido]](#medida-faturamento-liquido-d5e6f7a8), [_Medidas[% Atingimento de Meta]](#medida--atingimento-de-meta-a8b9c0d1)

**Tabelas:** [dimProdutos](#query-dimprodutos-c8d9e0f1)

**Campos:** `Familia`, `Linha`

<a id="visual-top-10-produtos-b1c2d3e4"></a>
##### Top 10 Produtos

**Tipo:** Tabela

**Objetivo:** Listar os 10 produtos com maior faturamento no período com suas métricas principais.


Tabela com os 10 produtos de maior faturamento, exibindo nome do produto, linha, faturamento líquido, quantidade vendida, ticket médio e margem de contribuição média. Ordenada decrescentemente por faturamento.


![Top 10 Produtos](imagens/visuais/img_analise-por-produto_top-10-produtos_visual.png)


**KPIs:** [Faturamento Líquido](#kpi-faturamento-liquido-a1b2c3d4), [Ticket Médio](#kpi-ticket-medio-c3d4e5f6)

**Medidas:** [_Medidas[Faturamento Líquido]](#medida-faturamento-liquido-d5e6f7a8), [_Medidas[Ticket Médio]](#medida-ticket-medio-b9c0d1e2)

**Tabelas:** [dimProdutos](#query-dimprodutos-c8d9e0f1)

**Campos:** `NomeProduto`, `Linha`, `Familia`, `MargemMedia`, `StatusGiro`

**Observações:** O TopN de 10 é aplicado via filtro de visual no Power BI. Para análise completa, o usuário pode remover o filtro de TopN ou exportar para Excel.

#### Filtros de Página (1)

| Filtro | Tipo | Campo | Descrição |
|---|---|---|---|
| Linha de Produto | Slicer | `dimProdutos[Linha]` | Slicer de seleção múltipla por linha de produto. Permite análise focada em uma ou mais linhas específicas. |

---

<a id="glossario"></a>
## 📖 Glossário (7)

**Atingimento de Meta:** Percentual do faturamento realizado em relação à meta comercial definida para o período. Valores acima de 100% indicam superação da meta.

**Classificação ABC:** Segmentação de clientes por importância estratégica baseada em faturamento anual. A = top 20% dos clientes (maior faturamento), B = 20% a 50%, C = 50% a 80%, D = demais clientes.

**Faturamento Líquido:** Receita de vendas após deducões de impostos (IPI, ICMS ST), descontos comerciais e devoluções. Representa o valor efetivamente reconhecido como receita pela operação comercial.

**RLS (Row-Level Security):** Segurança em nível de linha no Power BI Service. Restringe os dados visíveis a cada usuário com base em regras definidas no modelo. No contexto deste relatório, vendedores visualizam apenas seus próprios dados.

**Star Schema:** Modelo dimensional de dados composto por uma tabela fato central conectada a tabelas dimensão ao redor. É o padrão recomendado para modelos Power BI por oferecer melhor performance e clareza na escrita de medidas DAX.

**Ticket Médio:** Valor médio por transação (nota fiscal) em determinado período. Calculado como Faturamento Líquido dividido pela quantidade de notas fiscais de venda. Indica o valor médio que cada cliente gasta por compra.

**YTD (Year to Date):** Acumulado do ano até a data de referência. Ex: Faturamento YTD em Março representa a soma do faturamento de Janeiro, Fevereiro e Março.

---

*Documentado por: Ana Paula Ferreira*  
*Última revisão: 02/07/2026*