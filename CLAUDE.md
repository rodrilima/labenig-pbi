# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **Power BI Project (PBIP format)** — a Git-friendly representation of a Power BI report for **La Benig**, a Brazilian company. The `.pbip` format stores all report and model definitions as plain text files (JSON and TMDL), enabling version control.

Open `Projeto La Benig.pbip` in **Power BI Desktop** to work visually. All files in this repo are editable as plain text.

## Repository Structure

```
Projeto La Benig.pbip                  # Entry point — open this in Power BI Desktop
Projeto La Benig.Report/               # Report layer (visuals, pages, layout)
  definition/
    pages/<guid>/                      # One folder per report page
      visuals/<guid>/visual.json       # Individual visual definitions
    pages.json                         # Page order and active page
    report.json                        # Report-level settings and theme
Projeto La Benig.SemanticModel/        # Data model layer
  definition/
    model.tmdl                         # Model-level config, query groups, table refs
    relationships.tmdl                 # All table relationships
    tables/                            # One .tmdl file per table/measure table
```

## Data Architecture

### Query Groups (Data Sources)

| Group        | Description                                                                                                                                      |
| ------------ | ------------------------------------------------------------------------------------------------------------------------------------------------ |
| `datalake` | Live production data from**Google BigQuery** (`coherent-window-444219-c1` project, `datalake` schema). Used for `protheus_*` tables. |
| `mocks`    | In-memory M code tables for development/demo (most `fato_*` and `dim_*` tables).                                                             |
| `dw`       | Reserved query group (currently unused).                                                                                                         |

### Key Tables

**Protheus (ERP — BigQuery source):**

- `protheus_contasReceber` — Accounts receivable titles with aging, status, and customer data
- `protheus_contasPagar` — Accounts payable
- `protheus_contasBancarias` / `protheus_movimentosBancarios` — Bank accounts and transactions
- `protheus_contatos` — Customer contacts

**Fact tables (mock data):**

- `fato_pedidos` — Sales orders; `canal` column distinguishes `"Atacado"` vs `"E-commerce"`
- `fato_fluxo_caixa` — Cash flow
- `fato_contas_receber` / `fato_contas_pagar` — AR/AP (separate from Protheus, mock)
- `fato_resultado_operacional` — P&L
- `fato_comportamento_cliente` — Customer behavior / RFM
- `fato_logistica_pedidos` — Logistics and SLA tracking
- `fato_sac` — Customer service tickets

**Dimension tables:**

- `dim_clientes`, `dim_produtos`, `dim_lojas`, `dim_data`
- `calendario` — Used for Protheus date relationships (connected via `protheus_contasReceber[data_baixa]`)

**Navigation tables (all hidden):**

- `navegacao_modulos` — Top-level module menu (6 modules)
- `navegacao_submodulos` — Sub-page navigation per module
- `navegacao_menu` — Menu control
- `_guia_nomenclatura_paginas` — Reference table defining the official page naming convention

### Measure Table

All financial KPIs live in `medidas_financeiro.tmdl`. Measures use a `CR ` prefix for Contas a Receber metrics, organized in display folders: `Recebimentos`, `Inadimplência`, `PMR`, `Aging`, `Status Cliente`.

## Report Pages

Pages are named with a module prefix. The canonical list is defined in `_guia_nomenclatura_paginas`:

| Module               | Pages                                                                                            |
| -------------------- | ------------------------------------------------------------------------------------------------ |
| Home                 | Home                                                                                             |
| Financeiro (`FIN`) | Caixa & Liquidez, Contas a Receber, Contas a Pagar, Resultado Operacional                        |
| Vendas (`VEN`)     | Atacado Visão Geral, Atacado Mapa Geográfico, E-commerce Performance, E-commerce Base Clientes |
| Logística (`LOG`) | Pedidos, SLAs                                                                                    |
| SAC                  | Atendimento                                                                                      |
| Visão Executiva     | Visão Executiva                                                                                 |

Page folders use GUIDs as names. To find which GUID corresponds to which page, check the `displayName` inside each page's folder or match against `pages.json` ordering.

## Conventions

- **Language/culture:** Portuguese Brazil (`pt-BR`) throughout — DAX, column names, and page names.
- **Currency format:** `R$ #,##0.00`
- **Page naming:** `[MODULE] - [Page Name]` (e.g., `FIN - Contas a Receber`). Always follow this convention when adding pages; update `_guia_nomenclatura_paginas` and the relevant navigation table.
- **Measure naming:** Use descriptive prefixes by domain (e.g., `CR` for Contas a Receber).
- **Deprecated measures:** Mark with `/// DESCONTINUADA` in the docstring rather than deleting immediately.
- **Calculated columns vs measures:** Aging buckets (`faixa_aging`, `dias_em_aberto`) are calculated columns on `protheus_contasReceber`; KPI aggregations are measures in `medidas_financeiro`.

## Working with PBIP Files

- **TMDL files** (`.tmdl`) define tables, columns, measures, and partitions. Each table is one file.
- **visual.json** files define individual visuals. They are verbose JSON — make targeted edits.
- After editing files directly, open or refresh in Power BI Desktop to validate.
- The `annotation PBI_ProTooling = ["MCP-PBIModeling","DevMode"]` in `model.tmdl` indicates this model was built with Power BI MCP tooling.

## Semantic Model Editing Rules

> **NEVER edit `.tmdl` files directly.** All changes to the semantic model (measures, calculated columns, tables, relationships) must be made via the **Power BI MCP tools** (`mcp__powerbi-modeling-mcp__*`).

- `.tmdl` files are managed exclusively by Power BI Desktop + MCP tooling. Manual edits will be overwritten or cause conflicts.
- To add/update measures → use `mcp__powerbi-modeling-mcp__measure_operations` or `batch_measure_operations`
- To add/update calculated columns → use `mcp__powerbi-modeling-mcp__column_operations`
- To add/update tables → use `mcp__powerbi-modeling-mcp__table_operations`
- Before any MCP operation, connect to the running Power BI Desktop instance via `mcp__powerbi-modeling-mcp__connection_operations` (operation: `ListLocalInstances`, then `Connect`)
- **`visual.json` files** (report layer) CAN be edited directly with Edit/Write tools — they are not managed by MCP.

## Ordem de Edição (Workflow Obrigatório)

> **REGRA CRÍTICA — nunca violar esta ordem. Erros aqui causam perda de trabalho.**

**Sempre que uma tarefa envolver TMDL + visual.json:**

1. **Primeiro: TMDL via MCP** — criar/atualizar medidas, colunas ou tabelas
2. **→ Instrução ao usuário: "Salve no Power BI Desktop agora (Ctrl+S)"** — aguardar confirmação antes de continuar
3. **Segundo: visual.json via Edit/Write** — editar os visuais da camada de relatório
4. **→ Instrução ao usuário: "Reinicie o Power BI Desktop agora (feche e reabra o .pbip)"** — NÃO pedir Ctrl+S aqui

**Regras explícitas para as mensagens ao usuário:**

- Após etapa TMDL (MCP): pedir **Ctrl+S** (salvar)
- Após etapa visual.json (Edit/Write): pedir **reiniciar** (fechar e reabrir o .pbip) — **NUNCA pedir Ctrl+S após editar visual.json**
- O Power BI Desktop NÃO detecta arquivos visual.json novos/editados em tempo real — só carrega ao reabrir o projeto
- Ctrl+S no PBI Desktop **sobrescreve** o visual.json editado manualmente se feito após a edição
