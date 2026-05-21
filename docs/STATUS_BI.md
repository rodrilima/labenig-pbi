# Status do BI — La Benig

> **Última atualização:** 2026-05-21
> **Versão:** Power BI Desktop · formato `.pbip`
> **Responsável pela atualização:** _preencher_

Documento de acompanhamento da construção do relatório Power BI da La Benig. Lista cada uma das **13 páginas oficiais**, os visuais que ela contém, a fonte de dados de cada visual (BigQuery vs. mock) e o status de validação pelos gestores responsáveis em duas dimensões: apresentação visual e correção dos dados.

---

## Como ler este documento

### Legenda

| Coluna | ✅ significa | ❌ significa |
|---|---|---|
| **Fonte** | Visual usa apenas dados reais (BigQuery ou tabela auxiliar estável) | Visual usa pelo menos uma tabela mock (dado fictício) |
| **Validação Visual** | Gestor aprovou a **apresentação** (layout, formato, eixos, cor, legenda) | Apresentação ainda aguarda revisão |
| **Validação Dados** | Gestor aprovou os **números e a lógica** do visual | Números ainda aguardam validação |

| Símbolo | Significado |
|:---:|---|
| ⚠️ | Inconsistência conhecida no modelo — ver §[Fontes de dados](#fontes-de-dados) |

### Convenções

- Apenas visuais que **apresentam ou filtram dados** estão listados. Botões de navegação, imagens, formas e caixas de texto foram omitidos para manter o documento focado.
- A coluna **Tabelas/Medidas** lista as entidades referenciadas pelo visual. `medidas_financeiro` é uma tabela apenas de medidas — herda a origem das tabelas que ela referencia internamente.
- Um visual é marcado ✅ na coluna **Fonte** quando todas as suas tabelas-base (excluindo `medidas_financeiro`, `calendario` e outras tabelas DAX) são produção. `protheus_filiais` é considerada produção mesmo sendo hardcoded — é uma decisão de design aceita. `Contas` e `Saldos`, carregadas de Google Sheets oficial da operação, também são consideradas produção.
- As três colunas de status são **independentes**: um visual pode estar em BigQuery (✅ Fonte) e ainda assim aguardar ambas as validações (❌ Validação Visual, ❌ Validação Dados).

---

## Resumo executivo

| Indicador | Valor |
|---|---|
| Páginas oficiais entregues | **14** de 14 |
| Visuais de dados rastreados | **132** |
| Visuais com fonte BigQuery (produção) | **82 (62%)** |
| Visuais com fonte Google Sheets (produção) | **5 (4%)** — toda a página FIN - Contas Bancárias |
| Visuais com validação visual concluída | **52 (39%)** — todo o módulo Financeiro, exceto Contas Bancárias |
| Visuais com validação de dados concluída | **0 (0%)** — a preencher |
| Páginas **100% BigQuery** | **4** — Caixa & Liquidez, Contas a Receber, Contas a Pagar, Resultado Operacional |
| Páginas **100% Google Sheets** | **1** — FIN - Contas Bancárias |
| Páginas predominantemente produção (≥ 80%) | **7** — 4 FIN BigQuery + 2 VEN Atacado + Contas Bancárias |
| Páginas com validação visual 100% concluída | **4** — Caixa & Liquidez, Contas a Receber, Contas a Pagar, Resultado Operacional |
| Páginas inteiramente em mock | **4** — LOG - Pedidos, LOG - SLAs, SAC - Atendimento, Visão Executiva |

> **Próximas entregas previstas:** _preencher conforme planejamento (ex.: migração de `fato_pedidos` → `protheus_pedidosVendas` na página E-commerce Performance, revisão da medida `CR Inadimplência`, validação financeira pelo gestor X até DD/MM)._

---

## Páginas oficiais

### Home

**Módulo:** Home
**Filtros da página:** —

| # | Visual | Tipo | Tabelas/Medidas | Fonte | Validação Visual | Validação Dados | Notas |
|---|---|---|---|:---:|:---:|:---:|---|
| 1 | Total de Pedidos | Card | `protheus_movimentosBancarios` | ✅ | ❌ | ❌ | |
| 2 | Faturamento | Gráfico de linhas | `protheus_movimentosBancarios` · `calendario` | ✅ | ❌ | ❌ | |
| 3 | Nota SAC | Card | `fato_sac` | ❌ | ❌ | ❌ | Aguarda integração da fonte real de SAC |
| 4 | Faturamento Total | Card | `protheus_movimentosBancarios` | ✅ | ❌ | ❌ | |
| 5 | Filtro de período | Slicer | `calendario` | ✅ | ❌ | ❌ | |
| 6 | Filtro de período | Slicer | `calendario` | ✅ | ❌ | ❌ | |
| 7 | Status de Pedidos | Donut | `fato_pedidos` | ❌ | ❌ | ❌ | Aguarda migração para `protheus_pedidosVendas` |
| 8 | Saldo de Caixa | Card | `protheus_contasBancarias` | ✅ | ❌ | ❌ | |
| 9 | SLA Coleta Logística | Gauge | `fato_logistica_pedidos` | ❌ | ❌ | ❌ | Aguarda integração da fonte real de logística |

---

### FIN - Caixa & Liquidez

**Módulo:** Financeiro
**Filtros da página:** —

| # | Visual | Tipo | Tabelas/Medidas | Fonte | Validação Visual | Validação Dados | Notas |
|---|---|---|---|:---:|:---:|:---:|---|
| 1 | Saída | Card | `protheus_contasPagar` · `medidas_financeiro` | ✅ | ✅ | ❌ | |
| 2 | Projetado | Tabela | `projecao_caixa` · `calendario` | ✅ | ✅ | ❌ | `projecao_caixa` é derivada de `protheus_contasPagar` + `protheus_contasReceber` |
| 3 | Realizado | Tabela | `protheus_movimentosBancarios` · `calendario` | ✅ | ✅ | ❌ | |
| 4 | Filtro por filial | Slicer | `protheus_filiais` | ✅ | ✅ | ❌ | |
| 5 | Entrada | Card | `protheus_contasReceber` · `medidas_financeiro` | ✅ | ✅ | ❌ | |
| 6 | Saldo | Card | `medidas_financeiro` | ✅ | ✅ | ❌ | |
| 7 | Filtro de período | Slicer | `calendario` | ✅ | ✅ | ❌ | |
| 8 | Fluxo Projetado x Realizado | Gráfico de combo | `protheus_contasReceber` · `protheus_contasPagar` · `calendario` · `medidas_financeiro` | ✅ | ✅ | ❌ | |
| 9 | Número Título | Filtro de texto | `protheus_numerosTitulos` | ✅ | ✅ | ❌ | |
| 10 | Filtro de período | Slicer | `calendario` | ✅ | ✅ | ❌ | |
| 11 | Filtro de período | Slicer | `calendario` | ✅ | ✅ | ❌ | |

---

### FIN - Contas Bancárias

**Módulo:** Financeiro
**Filtros da página:** —

| # | Visual | Tipo | Tabelas/Medidas | Fonte | Validação Visual | Validação Dados | Notas |
|---|---|---|---|:---:|:---:|:---:|---|
| 1 | Saldo Total | Card | `medidas_financeiro` · `Saldos` · `Contas` | ✅ | ❌ | ❌ | Snapshot do saldo mais recente por conta (Google Sheets) |
| 2 | Saldo Atual por Banco | Gráfico customizado (Deneb) | `Contas` · `medidas_financeiro` · `Saldos` | ✅ | ❌ | ❌ | Visual customizado |
| 3 | Maior Saldo | Card | `medidas_financeiro` · `Saldos` | ✅ | ❌ | ❌ | |
| 4 | Menor Saldo | Card | `medidas_financeiro` · `Saldos` | ✅ | ❌ | ❌ | |
| 5 | Contas Negativas | Card | `medidas_financeiro` · `Saldos` | ✅ | ❌ | ❌ | Quantidade de contas com saldo negativo |

> Página alimentada por **Google Sheets** (planilha oficial de saldos bancários). Medidas com prefixo `FIN CB ` agregam `Saldos` por `Contas`.

---

### FIN - Contas a Receber

**Módulo:** Financeiro
**Filtros da página:** `protheus_contasReceber[nome_cliente]`

| # | Visual | Tipo | Tabelas/Medidas | Fonte | Validação Visual | Validação Dados | Notas |
|---|---|---|---|:---:|:---:|:---:|---|
| 1 | Inadimplência (%) | Card | `medidas_financeiro` | ✅ | ✅ | ❌ | |
| 2 | Filtro de período | Slicer | `calendario` | ✅ | ✅ | ❌ | |
| 3 | Detalhe de títulos | Tabela | `protheus_contasReceber` | ✅ | ✅ | ❌ | |
| 4 | Filtro de período | Slicer | `calendario` | ✅ | ✅ | ❌ | |
| 5 | Aging dos Recebíveis | Gráfico de colunas agrupadas | `protheus_contasReceber` · `medidas_financeiro` | ✅ | ✅ | ❌ | |
| 6 | Recebido no Período | Card | `medidas_financeiro` | ✅ | ✅ | ❌ | |
| 7 | Nome Cliente | Filtro de texto | `protheus_contasReceber` | ✅ | ✅ | ❌ | |
| 8 | Filtro de período | Slicer | `calendario` | ✅ | ✅ | ❌ | |
| 9 | PMR — Prazo Médio de Recebimento | Card | `medidas_financeiro` | ✅ | ✅ | ❌ | |
| 10 | Filtro de status do cliente | Slicer | `protheus_contasReceber` | ✅ | ✅ | ❌ | |
| 11 | Total a Receber | Card | `medidas_financeiro` | ✅ | ✅ | ❌ | |
| 12 | Maiores Devedores | Tabela | `protheus_contasReceber` · `medidas_financeiro` | ✅ | ✅ | ❌ | |
| 13 | Filtro por filial | Slicer | `protheus_filiais` | ✅ | ✅ | ❌ | |
| 14 | Inadimplência (%) mês a mês | Gráfico de linhas | `calendario` · `medidas_financeiro` | ✅ | ✅ | ❌ | |
| 15 | Número Título | Filtro de texto | `protheus_numerosTitulos` | ✅ | ✅ | ❌ | |

---

### FIN - Contas a Pagar

**Módulo:** Financeiro
**Filtros da página:** `protheus_contasPagar[nome_fornecedor]`

| # | Visual | Tipo | Tabelas/Medidas | Fonte | Validação Visual | Validação Dados | Notas |
|---|---|---|---|:---:|:---:|:---:|---|
| 1 | Filtro por filial | Slicer | `protheus_filiais` | ✅ | ✅ | ❌ | |
| 2 | Nome Fornecedor | Filtro de texto | `protheus_contasPagar` | ✅ | ✅ | ❌ | |
| 3 | Pagamentos Realizados | Card | `medidas_financeiro` | ✅ | ✅ | ❌ | |
| 4 | Detalhe de títulos | Tabela | `protheus_contasPagar` | ✅ | ✅ | ❌ | |
| 5 | Em Atraso | Card | `medidas_financeiro` | ✅ | ✅ | ❌ | |
| 6 | Filtro de período | Slicer | `calendario` | ✅ | ✅ | ❌ | |
| 7 | Filtro de período | Slicer | `calendario` | ✅ | ✅ | ❌ | |
| 8 | Total a Pagar | Card | `medidas_financeiro` | ✅ | ✅ | ❌ | |
| 9 | PMP — Prazo Médio de Pagamento | Card | `medidas_financeiro` | ✅ | ✅ | ❌ | |
| 10 | Maiores Credores | Tabela | `protheus_contasPagar` · `medidas_financeiro` | ✅ | ✅ | ❌ | |
| 11 | Filtro de período | Slicer | `calendario` | ✅ | ✅ | ❌ | |
| 12 | Filtro de status | Slicer | `protheus_contasPagar` | ✅ | ✅ | ❌ | |
| 13 | Total Entrada | Card | `protheus_movimentosBancarios` · `medidas_financeiro` | ✅ | ✅ | ❌ | |
| 14 | Aging — Contas a Pagar | Gráfico de colunas | `protheus_contasPagar` · `medidas_financeiro` | ✅ | ✅ | ❌ | |
| 15 | Número Título | Filtro de texto | `protheus_numerosTitulos` | ✅ | ✅ | ❌ | |

---

### FIN - Resultado Operacional

**Módulo:** Financeiro
**Filtros da página:** —

| # | Visual | Tipo | Tabelas/Medidas | Fonte | Validação Visual | Validação Dados | Notas |
|---|---|---|---|:---:|:---:|:---:|---|
| 1 | Despesas Variáveis | Card | `protheus_movimentosBancarios` · `protheus_naturezas` | ✅ | ✅ | ❌ | |
| 2 | Filtro por movimento | Slicer | `protheus_movimentosBancarios` | ✅ | ✅ | ❌ | |
| 3 | Filtro por filial | Slicer | `protheus_filiais` | ✅ | ✅ | ❌ | |
| 4 | Filtro de período | Slicer | `calendario` | ✅ | ✅ | ❌ | |
| 5 | Receitas vs Despesas | Gráfico de barras agrupadas | `protheus_movimentosBancarios` · `protheus_naturezas` | ✅ | ✅ | ❌ | |
| 6 | Despesas Fixas | Card | `protheus_movimentosBancarios` · `protheus_naturezas` | ✅ | ✅ | ❌ | |
| 7 | Subcategorias | Gráfico de barras agrupadas | `protheus_movimentosBancarios` · `protheus_naturezas` | ✅ | ✅ | ❌ | |
| 8 | Filtro de período | Slicer | `calendario` | ✅ | ✅ | ❌ | |
| 9 | Receita Total | Card | `protheus_movimentosBancarios` · `protheus_naturezas` | ✅ | ✅ | ❌ | |
| 10 | Detalhamento por natureza | Tabela | `protheus_movimentosBancarios` · `protheus_naturezas` | ✅ | ✅ | ❌ | |
| 11 | Resultado Operacional | Card | `medidas_financeiro` | ✅ | ✅ | ❌ | |

---

### VEN - Atacado Visão Geral

**Módulo:** Vendas
**Filtros da página:** —

| # | Visual | Tipo | Tabelas/Medidas | Fonte | Validação Visual | Validação Dados | Notas |
|---|---|---|---|:---:|:---:|:---:|---|
| 1 | Faturamento Atacado | Card | `medidas_financeiro` | ✅ | ❌ | ❌ | Medida calculada sobre `protheus_pedidosVendas` |
| 2 | Ticket Médio Atacado | Card | `medidas_financeiro` | ✅ | ❌ | ❌ | |
| 3 | Clientes Ativos | Card | `medidas_financeiro` | ✅ | ❌ | ❌ | |
| 4 | CAC | Card | `fato_pedidos` | ❌ | ❌ | ❌ | Aguarda migração para `protheus_pedidosVendas` |
| 5 | Pedidos em Aberto | Card | `medidas_financeiro` | ✅ | ❌ | ❌ | |
| 6 | Custo Total Frete | Card | `protheus_pedidosVendas` | ✅ | ❌ | ❌ | |
| 7 | Clientes Inativos | Card | `medidas_financeiro` | ✅ | ❌ | ❌ | |
| 8 | Pedidos Faturados | Card | `medidas_financeiro` | ✅ | ❌ | ❌ | |
| 9 | Filtro por filial | Slicer | `protheus_filiais` | ✅ | ❌ | ❌ | |
| 10 | Filtro de período | Slicer | `calendario` | ✅ | ❌ | ❌ | |
| 11 | Tabela de pedidos | Tabela | `protheus_contatos` · `protheus_itensPedidosVendas` · `medidas_financeiro` | ✅ | ❌ | ❌ | |
| 12 | Filtro de período | Slicer | `calendario` | ✅ | ❌ | ❌ | |

---

### VEN - Atacado Mapa Geográfico

**Módulo:** Vendas
**Filtros da página:** —

| # | Visual | Tipo | Tabelas/Medidas | Fonte | Validação Visual | Validação Dados | Notas |
|---|---|---|---|:---:|:---:|:---:|---|
| 1 | Filtro de período | Slicer | `calendario` | ✅ | ❌ | ❌ | |
| 2 | Estados Atendidos | Card | `protheus_pedidosVendas` | ✅ | ❌ | ❌ | |
| 3 | Detalhe por estado | Tabela | `protheus_contatos` · `protheus_pedidosVendas` | ✅ | ❌ | ❌ | |
| 4 | Filtro por filial | Slicer | `protheus_filiais` | ✅ | ❌ | ❌ | |
| 5 | Ticket Médio por Estado | Gráfico de barras agrupadas | `protheus_contatos` · `protheus_pedidosVendas` | ✅ | ❌ | ❌ | |
| 6 | Faturamento por Estado (mapa) | Mapa | `dim_clientes` · `fato_pedidos` | ❌ | ❌ | ❌ | Mapa ainda em mock — migrar para tabelas Protheus |
| 7 | Faturamento Total | Card | `protheus_pedidosVendas` | ✅ | ❌ | ❌ | |
| 8 | Clientes por Estado | Card | `protheus_pedidosVendas` | ✅ | ❌ | ❌ | |
| 9 | Filtro de período | Slicer | `calendario` | ✅ | ❌ | ❌ | |

---

### VEN - E-commerce Performance

**Módulo:** Vendas
**Filtros da página:** —

| # | Visual | Tipo | Tabelas/Medidas | Fonte | Validação Visual | Validação Dados | Notas |
|---|---|---|---|:---:|:---:|:---:|---|
| 1 | Pedidos | Tabela | `fato_pedidos` | ❌ | ❌ | ❌ | |
| 2 | Cupom de Desconto | Tabela | `fato_pedidos` | ❌ | ❌ | ❌ | |
| 3 | CAC | Card | `fato_pedidos` | ❌ | ❌ | ❌ | |
| 4 | Filtro de período | Slicer | `calendario` | ✅ | ❌ | ❌ | |
| 5 | Faturamento Total | Card | `fato_pedidos` | ❌ | ❌ | ❌ | |
| 6 | Faturamento por Loja | Gráfico de colunas agrupadas | `dim_lojas` · `fato_pedidos` | ❌ | ❌ | ❌ | |
| 7 | Custo Total de Frete | Card | `fato_pedidos` | ❌ | ❌ | ❌ | |
| 8 | Filtro por filial | Slicer | `protheus_filiais` | ✅ | ❌ | ❌ | |
| 9 | Pedidos Cancelados | Card | `fato_pedidos` | ❌ | ❌ | ❌ | |
| 10 | Filtro de período | Slicer | `calendario` | ✅ | ❌ | ❌ | |
| 11 | Ticket Médio | Card | `fato_pedidos` | ❌ | ❌ | ❌ | |

> Página em modo de **prototipagem completa** — toda a base depende de `fato_pedidos` (mock). Próximo passo: migrar para fonte de e-commerce real.

---

### VEN - E-commerce Base Clientes

**Módulo:** Vendas
**Filtros da página:** —

| # | Visual | Tipo | Tabelas/Medidas | Fonte | Validação Visual | Validação Dados | Notas |
|---|---|---|---|:---:|:---:|:---:|---|
| 1 | Filtro de período | Slicer | `calendario` | ✅ | ❌ | ❌ | |
| 2 | Taxa Recompra | Card | `fato_comportamento_cliente` | ❌ | ❌ | ❌ | |
| 3 | Filtro de período | Slicer | `calendario` | ✅ | ❌ | ❌ | |
| 4 | Pedidos últimos 45 dias | Gráfico de barras agrupadas | `dim_clientes` · `fato_comportamento_cliente` | ❌ | ❌ | ❌ | |
| 5 | Classificação de Clientes | Tabela | `dim_clientes` · `fato_comportamento_cliente` | ❌ | ❌ | ❌ | |
| 6 | Clientes Ecommerce | Card | `dim_clientes` | ❌ | ❌ | ❌ | |
| 7 | Clientes Recorrentes | Card | `fato_comportamento_cliente` | ❌ | ❌ | ❌ | |
| 8 | Clientes Inativos | Card | `fato_comportamento_cliente` | ❌ | ❌ | ❌ | |
| 9 | Distribuição de clientes | Donut | `fato_comportamento_cliente` | ❌ | ❌ | ❌ | |

> Página em modo de **prototipagem completa** — depende de `fato_comportamento_cliente` (RFM mock).

---

### LOG - Pedidos

**Módulo:** Logística
**Filtros da página:** —

| # | Visual | Tipo | Tabelas/Medidas | Fonte | Validação Visual | Validação Dados | Notas |
|---|---|---|---|:---:|:---:|:---:|---|
| 1 | Pedidos Coletados | Card | `fato_logistica_pedidos` | ❌ | ❌ | ❌ | |
| 2 | Pedidos por status | Gráfico de barras agrupadas | `fato_logistica_pedidos` | ❌ | ❌ | ❌ | |
| 3 | Status Atual (funil) | Funil | `fato_logistica_pedidos` | ❌ | ❌ | ❌ | |
| 4 | Pedidos Faturados | Card | `fato_logistica_pedidos` | ❌ | ❌ | ❌ | |
| 5 | Em Conferência | Card | `fato_logistica_pedidos` | ❌ | ❌ | ❌ | |
| 6 | Em Separação | Card | `fato_logistica_pedidos` | ❌ | ❌ | ❌ | |
| 7 | Pedidos Coletados por loja | Gráfico de barras agrupadas | `dim_lojas` · `fato_logistica_pedidos` | ❌ | ❌ | ❌ | |

> Página inteira em mock — aguarda integração da fonte de logística.

---

### LOG - SLAs

**Módulo:** Logística
**Filtros da página:** —

| # | Visual | Tipo | Tabelas/Medidas | Fonte | Validação Visual | Validação Dados | Notas |
|---|---|---|---|:---:|:---:|:---:|---|
| 1 | Coleta x Meta Coleta | Gauge | `fato_logistica_pedidos` | ❌ | ❌ | ❌ | |
| 2 | SLA Separação Mensal | Gráfico de linhas | `dim_data` · `fato_logistica_pedidos` | ❌ | ❌ | ❌ | |
| 3 | Resumo SLA de Logística | Tabela dinâmica | `dim_lojas` · `fato_logistica_pedidos` | ❌ | ❌ | ❌ | |
| 4 | Separação x Meta Separação | Gauge | `fato_logistica_pedidos` | ❌ | ❌ | ❌ | |
| 5 | SLA Coleta Mensal | Gráfico de linhas | `dim_data` · `fato_logistica_pedidos` | ❌ | ❌ | ❌ | |

> Página inteira em mock — aguarda integração da fonte de logística.

---

### SAC - Atendimento

**Módulo:** SAC
**Filtros da página:** —

| # | Visual | Tipo | Tabelas/Medidas | Fonte | Validação Visual | Validação Dados | Notas |
|---|---|---|---|:---:|:---:|:---:|---|
| 1 | Chamados Abertos | Card | `fato_sac` | ❌ | ❌ | ❌ | |
| 2 | Percentual SLA Cumprido | Card | `fato_sac` | ❌ | ❌ | ❌ | |
| 3 | SLA Mensal SAC | Gráfico de linhas | `dim_data` · `fato_sac` | ❌ | ❌ | ❌ | |
| 4 | Nota de Atendimento | Gráfico de barras agrupadas | `fato_sac` | ❌ | ❌ | ❌ | |
| 5 | Status de Chamado | Donut | `fato_sac` | ❌ | ❌ | ❌ | |
| 6 | Nota Média SAC | Card | `fato_sac` | ❌ | ❌ | ❌ | |
| 7 | Total Chamados | Card | `fato_sac` | ❌ | ❌ | ❌ | |
| 8 | Nota Média Mensal | Gráfico de linhas | `dim_data` · `fato_sac` | ❌ | ❌ | ❌ | |

> Página inteira em mock — aguarda integração da fonte real de SAC.

---

### Visão Executiva

**Módulo:** Visão Executiva
**Filtros da página:** —

| # | Visual | Tipo | Tabelas/Medidas | Fonte | Validação Visual | Validação Dados | Notas |
|---|---|---|---|:---:|:---:|:---:|---|
| 1 | Faturamento Geral | Gráfico de linhas | `dim_data` · `fato_pedidos` | ❌ | ❌ | ❌ | |
| 2 | Resumo Financeiro | Card | `fato_fluxo_caixa` · `fato_resultado_operacional` · `fato_contas_receber` | ❌ | ❌ | ❌ | Migrar para `protheus_*` e `medidas_financeiro` |
| 3 | Resumo SAC | Card | `fato_sac` | ❌ | ❌ | ❌ | |
| 4 | Resumo Logística | Card | `fato_logistica_pedidos` | ❌ | ❌ | ❌ | |
| 5 | Resumo Vendas | Card | `fato_pedidos` | ❌ | ❌ | ❌ | |

> Página inteira em mock — depende da migração das fontes alimentadoras.

---

## Fontes de dados

Tabelas referenciadas por pelo menos um visual oficial deste documento.

| Tabela | Domínio | Fonte real | Observação |
|---|---|:---:|---|
| `protheus_contasReceber` | ERP — Contas a Receber | ✅ BigQuery | — |
| `protheus_contasPagar` | ERP — Contas a Pagar | ✅ BigQuery | — |
| `protheus_movimentosBancarios` | ERP — Bancos | ✅ BigQuery | — |
| `protheus_contasBancarias` | ERP — Bancos | ✅ BigQuery | — |
| `protheus_naturezas` | ERP — Plano de contas | ✅ BigQuery | — |
| `protheus_numerosTitulos` | ERP — Títulos | ✅ BigQuery | Carregada via `Value.NativeQuery` (SQL customizado) |
| `protheus_contatos` | ERP — Clientes | ✅ BigQuery | ⚠️ Etiquetada `queryGroup = mocks`, mas a partition é BigQuery — corrigir label |
| `protheus_pedidosVendas` | ERP — Vendas (cabeçalho) | ✅ BigQuery | ⚠️ Mesma inconsistência de `queryGroup` — corrigir label |
| `protheus_itensPedidosVendas` | ERP — Vendas (itens) | ✅ BigQuery | ⚠️ Mesma inconsistência de `queryGroup` — corrigir label |
| `projecao_caixa` | Projeção (derivada) | ✅ BigQuery | Combina `protheus_contasPagar` + `protheus_contasReceber` |
| `protheus_filiais` | Filiais | ✅ Produção | Hardcoded no modelo por decisão de design — conjunto pequeno e estável de filiais. Considerada tabela de produção. |
| `Contas` | Cadastro de contas bancárias | ✅ Produção (Google Sheets) | Planilha oficial de operação — cadastro de contas com filial, banco, agência, ID_Conta. Usada por medidas `FIN CB *` |
| `Saldos` | Snapshots diários de saldo | ✅ Produção (Google Sheets) | Histórico de saldos por data e conta. Relacionada com `Contas` via `ID_Conta` |
| `fato_pedidos` | Vendas | ❌ Mock | Substituir por `protheus_pedidosVendas` / `protheus_itensPedidosVendas` |
| `fato_sac` | SAC | ❌ Mock | Aguarda integração da fonte de SAC |
| `fato_logistica_pedidos` | Logística | ❌ Mock | Aguarda integração da fonte de logística |
| `fato_comportamento_cliente` | RFM / Clientes | ❌ Mock | Aguarda definição de fonte de RFM |
| `fato_fluxo_caixa` | Fluxo de caixa | ❌ Mock | Migrar para `medidas_financeiro` + `projecao_caixa` |
| `fato_resultado_operacional` | DRE | ❌ Mock | Migrar para `medidas_financeiro` + `protheus_movimentosBancarios` |
| `fato_contas_receber` | AR (mock) | ❌ Mock | Não confundir com `protheus_contasReceber` |
| `dim_clientes` | Dimensão de clientes | ❌ Mock | — |
| `dim_lojas` | Dimensão de lojas | ❌ Mock | — |
| `dim_data` | Dimensão de data | ❌ Mock | Substituir uso por `calendario` quando possível |
| `calendario` | Calendário | 🟦 Calculada (DAX) | Tabela DAX gerada com `CALENDAR()` — estável, sem dependência externa |
| `medidas_financeiro` | Tabela de medidas | — | Herda a fonte das tabelas que referencia internamente |

---

## Convenção de atualização

Para manter este documento vivo:

1. **Novo visual entregue** → adicionar linha na tabela da página correspondente. Marcar `Fonte` conforme as tabelas usadas e ambas as colunas de validação (`Validação Visual`, `Validação Dados`) como ❌.
2. **Visual migrou de mock para BigQuery** → trocar `Fonte` de ❌ para ✅ na linha do visual e revisar a tabela da seção [Fontes de dados](#fontes-de-dados).
3. **Gestor aprovou a apresentação visual** → trocar `Validação Visual` de ❌ para ✅. Opcional: anotar a data e o nome do gestor na coluna **Notas**.
4. **Gestor aprovou os números/lógica do visual** → trocar `Validação Dados` de ❌ para ✅. Esta validação é **independente** da `Validação Visual` — pode acontecer antes, depois ou em paralelo.
5. **Nova página oficial entregue** → adicionar uma nova subseção em [Páginas oficiais](#páginas-oficiais), e atualizar `_guia_nomenclatura_paginas.tmdl` no modelo semântico (regra de nomenclatura do `CLAUDE.md`).
6. **Sempre atualizar o cabeçalho** (data e responsável) ao editar.
7. **Recalcular o resumo executivo** após alterações em massa (totais e percentuais).
