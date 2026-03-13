# 🛒 HASHTAG ELETRO — DASHBOARD DE VENDAS EM POWER BI

> **De dados brutos a inteligência de negócio:** um dashboard gerencial completo de uma rede varejista de eletronicos, com modelagem estrela, time intelligence e análise de devoluções (Este projeto faz parte do curso de formação em Análise de Dados da Hashtag Treinamentos).

![Power BI](https://img.shields.io/badge/Power%20BI-Desktop-F2C811?style=flat&logo=powerbi&logoColor=black)
![DAX](https://img.shields.io/badge/DAX-31%20medidas-0078D4?style=flat)
![Modelo Estrela](https://img.shields.io/badge/Modelo-Estrela-brightgreen?style=flat)
![Status](https://img.shields.io/badge/Status-Concluído-success?style=flat)

---

## 🎯 O Problema de Negócio

Uma rede varejista de eletrônicos com lojas físicas e canal online precisava de visibilidade centralizada sobre sua operação comercial. As perguntas que não tinham resposta rápida:

- Qual foi o faturamento do mês atual vs. o mesmo período do ano passado?
- Quais produtos lideram as vendas e qual é a margem de cada categoria?
- As devoluções estão crescendo? Quanto faturamento está sendo perdido?
- Qual loja performa melhor? O canal online está crescendo?
- Como está o crescimento MoM e YoY do negócio?

**A solução:** um dashboard analítico construído em Power BI Desktop, com modelagem dimensional, 31 medidas DAX e múltiplas perspectivas de análise.

---

## 🏗️ Arquitetura do Modelo de Dados

O modelo segue o padrão **Esquema Estrela**, com separação clara entre tabelas fato e dimensões:

```
        dCalendario ────────────────────────────────────────
             │                                              │
        dClientes          fVendas          fDevoluções
             │            ╱       ╲              │
          dLojas ────────         dProdutos ─────
                                      │
                                  Parâmetro (campo dinâmico)
```

### Tabelas do Modelo

| Tipo | Tabela | Descrição | Colunas |
|------|--------|-----------|---------|
| 📋 Fato | `fVendas` | Transações de venda | Data, SKU, Cliente, Loja, Qtd + coluna calculada *Tipo do Pedido* |
| 📋 Fato | `fDevoluções` | Devoluções de produtos | Data, Loja, SKU, Qtd, Motivo |
| 📐 Dimensão | `dProdutos` | Catálogo de produtos | SKU, Nome, Cor, Marca, Preço Unit., Custo Unit., Tipo, Ticket |
| 📐 Dimensão | `dClientes` | Base de clientes | Nome, Email, Gênero, Idade, *Faixa Etária* (calc.), Escolaridade |
| 📐 Dimensão | `dLojas` | Rede de lojas | Nome, Gerente, Tipo (Online/Física), País, Continente |
| 📅 Calendário | `dCalendario` | Tabela de datas | Ano, Mês, Trimestre, *Data Vigente?* (calc.) |
| ⚙️ Parâmetro | `Parâmetro` | Seleção dinâmica de campos | Rótulo, Campo, Ordem |

---

## 📐 Medidas DAX — 31 Indicadores

As 31 medidas foram organizadas em grupos temáticos de acordo com a perspectiva de análise:

### 💰 Faturamento e Receita

| Medida | Lógica DAX |
|--------|------------|
| `Faturamento Total` | `SUMX(fVendas, Qtd * RELATED(Preço Unitário))` |
| `Faturamento MTD` | `TOTALMTD([Faturamento Total], dCalendario[Datas])` |
| `Faturamento QTD` | `TOTALQTD([Faturamento Total], dCalendario[Datas])` |
| `Faturamento YTD` | `CALCULATE([Fat. Total], DATESYTD(...))` |
| `Faturamento Mês Anterior` | `CALCULATE(..., DATEADD(..., -1, MONTH))` |
| `Faturamento LY` | Ano anterior via `DATEADD(..., -1, YEAR)` com `HASONEVALUE` |
| `Faturamento YTD LY` | YTD do ano anterior para comparação acumulada |

### 📈 Análise de Crescimento

| Medida | Descrição |
|--------|-----------|
| `% Crescimento MoM` | Variação mensal: atual vs. mês anterior |
| `% Crescimento Faturamento YoY` | Variação anual com `VAR` + `HASONEVALUE` + `BLANK()` guard |
| `% Crescimento YoY YTD` | YTD atual vs. YTD do ano anterior |
| `% Crescimento Lucro YoY` | Crescimento da margem ano a ano |
| `% Crescimento Vendas YoY` | Volume de vendas ano a ano |

> **Destaque técnico:** as medidas YoY utilizam `HASONEVALUE(dCalendario[Ano])` para evitar resultados enganosos em contextos sem filtro de ano, retornando `"N/A"` quando necessário.

### 🏪 Vendas e Desempenho

| Medida | Descrição |
|--------|-----------|
| `Total Vendas` | `SUM(fVendas[Qtd Vendida])` |
| `Vendas Online` | `CALCULATE([Total Vendas], dLojas[Tipo] = "Online")` |
| `% Vendas Online` | Participação do canal digital no total |
| `Produto Mais Vendido` | `MAX` com `TOPN(1, ALL(...), [Total Vendas])` |
| `Qtd Vendida (Top 1 Produto)` | Volume do produto líder |
| `% das Vendas (Top 1 Produto)` | Concentração no produto mais vendido |
| `Loja + Vendas` | Loja com maior volume via `TOPN` |
| `Qtd Vendida (Loja + Vendas)` | Volume da loja líder |
| `% das Vendas (Loja + Vendas)` | Participação da loja líder |

### 🔄 Devoluções e Perdas

| Medida | Descrição |
|--------|-----------|
| `Total Devoluções` | `SUM(fDevoluções[Qtd Devolvida])` |
| `% Devoluções` | Taxa de devoluções sobre vendas |
| `Prejuízo Devoluções` | `SUMX` com `RELATED(Preço Unitário)` |
| `Faturamento Perdido` | Receita não realizada por devoluções |
| `% Fat Perdido` | Percentual do faturamento impactado |

### 💹 Lucratividade

| Medida | Descrição |
|--------|-----------|
| `Lucro Total` | `SUMX(fVendas, (Qtd * Preço * 0.9) - (Qtd * Custo))` |
| `% Margem de Lucro` | `[Lucro Total] / [Faturamento Total]` |

### ⚙️ Controle Dinâmico

| Medida | Uso |
|--------|-----|
| `Mês Selecionado` | `SELECTEDVALUE` para exibir o período filtrado |
| `Categoria Selecionada` | `SELECTEDVALUE` para exibir a categoria ativa |

---

## 🧠 Decisões Técnicas Relevantes

### Por que SUMX em vez de SUM para o faturamento?

A tabela `fVendas` armazena apenas `Qtd Vendida`. O preço unitário fica na dimensão `dProdutos`. Usar `SUM` seria impossível. O `SUMX` itera linha a linha, buscando o preço via `RELATED()` e multiplicando pelo volume. Isso mantém a normalização do modelo e garante que uma alteração de preço em `dProdutos` se reflita automaticamente em todo o histórico.

### Por que HASONEVALUE nas medidas YoY?

Sem essa verificação, uma medida de crescimento YoY exibiria valores incorretos quando o usuário não filtrasse por um único ano (ex.: ao ver o total geral). O padrão `VAR + HASONEVALUE + retorno "N/A"` garante que o KPI só apareça quando o contexto de filtro for válido para o cálculo de comparação.

### Coluna calculada Faixa Etária em vez de medida

A segmentação por faixa etária (`< 30`, `30–45`, `> 45`) foi implementada como **coluna calculada** em `dClientes`, não como medida. O motivo: essa classificação é uma propriedade permanente do cliente, não um agregado dependente de contexto. Isso facilita o uso direto em slicers e eixos de visuais sem lógica adicional.

### Tabela Parâmetro para campos dinâmicos

A tabela `Parâmetro` permite que o usuário selecione dinamicamente qual campo analisar em determinados visuais (ex.: alternar entre Marca, Tipo de Produto ou Ticket), sem precisar de páginas separadas. Uma única medida com `SELECTEDVALUE` interpreta a escolha e roteia o cálculo — padrão "Field Parameters" aplicado manualmente para maior controle.

---

## 📊 Perspectivas de Análise

O dashboard foi estruturado para responder perguntas em diferentes camadas:

**Visão Executiva**
→ KPIs de faturamento (MTD, QTD, YTD), crescimento MoM e YoY, margem de lucro

**Desempenho Comercial**
→ Ranking de produtos e lojas, participação do canal online, ticket médio por categoria

**Análise de Devoluções**
→ Taxa de devolução, faturamento perdido, motivos mais frequentes por produto/loja

**Comparativo Temporal**
→ Evolução mensal, comparação com mesmo período do ano anterior, sazonalidade por trimestre

**Perfil de Clientes**
→ Distribuição por faixa etária, gênero, estado civil, nível educacional

---

## 🗂️ Estrutura do Projeto

```
Hashtag Eletro (Power BI Desktop)
│
├── 📋 fVendas                  → Fato: transações de venda
├── 📋 fDevoluções              → Fato: devoluções de produtos
│
├── 📐 dProdutos                → Dimensão: catálogo de produtos
├── 📐 dClientes                → Dimensão: base de clientes
├── 📐 dLojas                   → Dimensão: rede de lojas (online + física)
├── 📅 dCalendario              → Dimensão: calendário para time intelligence
│
└── ⚙️ Parâmetro               → Tabela auxiliar para seleção dinâmica de campos
```

---

## 🚀 Tecnologias Utilizadas

| Tecnologia | Uso no Projeto |
|------------|----------------|
| **Power BI Desktop** | Modelagem, desenvolvimento e publicação |
| **DAX** | 31 medidas: time intelligence, TOPN, SUMX, SELECTEDVALUE |
| **Esquema Estrela** | Modelagem dimensional com 2 fatos e 4 dimensões |
| **Power Query (M)** | Transformação e limpeza dos dados na carga |

---

## 👨‍💻 Sobre o Projeto e o Autor

Este projeto foi desenvolvido por **Gustavo Santos**, Analista de Dados, como parte do desenvolvimento prático de habilidades em Power BI e modelagem dimensional do curso Hashtag Treinamentos.

O objetivo foi construir um modelo analítico robusto, com atenção à qualidade das medidas DAX, integridade do modelo estrela e usabilidade do dashboard para um decisor de negócio real.

**Habilidades demonstradas neste projeto:**
- Modelagem dimensional (Esquema Estrela)
- Time Intelligence com DAX (MTD, QTD, YTD, MoM, YoY)
- Análise de rentabilidade e devoluções
- Boas práticas de organização de medidas
- Uso de parâmetros dinâmicos
- Design orientado a perguntas de negócio

---

🔗 **[LinkedIn — Gustavo Santos](#)**

*Se este projeto foi útil ou te inspirou, deixa uma ⭐ no repositório!*

---

*Desenvolvido como projeto de portfólio · 2026*