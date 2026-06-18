# Painel Analítico — Câmara dos Deputados (57ª Legislatura)

🔗 **Repositório:** [github.com/FlavioR77/Camara-dos-Deputados-57-Legislatura-PowerBI](https://github.com/FlavioR77/Camara-dos-Deputados-57-Legislatura-PowerBI)

📊 **Dashboard publicado:** [Acessar o relatório no Power BI](https://app.powerbi.com/view?r=eyJrIjoiOWQ0MGVjOWQtNGNlOC00ZGIzLTk5MzEtMGVkZjE3NThhMmI1IiwidCI6IjgyNWMwY2JlLWY0YjQtNGQzNi1iNzQwLTcxOGE2Mzc1NDc2ZCJ9)

Relatório em Power BI para exploração de gastos parlamentares (CEAP/Cotas Parlamentares) e atividade legislativa (proposições) dos deputados federais da 57ª Legislatura (2023–2026), com base nos dados abertos da Câmara dos Deputados.

> 📅 **Período coberto:** janeiro de 2023 a maio de 2026.

## 📌 Sobre o projeto

O objetivo é permitir a análise cruzada entre **comportamento de gastos** e **produção legislativa** de deputados e partidos, respondendo perguntas como:

- Quais deputados/partidos gastam mais por proposição apresentada?
- Como o gasto per capita se compara à produção legislativa per capita por partido?
- Qual a proporção entre autoria principal (proponente) e coautoria nas proposições?
- Como os gastos evoluem ano a ano (variação YoY, YTD, MTD)?

## 🗂️ Estrutura do relatório

O relatório possui duas páginas:

| Página | Conteúdo |
|---|---|
| **Deputado** | Indicadores individuais de gasto e produção legislativa por parlamentar |
| **Proposições** | Análise das proposições apresentadas, por tipo, tema e autoria |

## 🔗 Fonte dos dados

Dados públicos extraídos do Portal de Dados Abertos da Câmara dos Deputados:
**https://dadosabertos.camara.leg.br/**

Arquivos utilizados (exportações anuais, 2023 a 2026 — dados de 2026 disponíveis até maio):

- `Ano{ano}.xlsx` — Despesas da Cota para Exercício da Atividade Parlamentar (CEAP)
- `proposicoes{ano}.xlsx` — Proposições apresentadas, por ano
- `proposicoesAutores{ano}.xlsx` — Relação entre proposições e seus autores

> Apenas os seguintes tipos de proposição foram considerados no painel:
> - **139** — Projeto de Lei
> - **140** — Projeto de Lei Complementar
> - **136** — Proposta de Emenda à Constituição
> - **291** — Medida Provisória

## 🧱 Modelo de dados

O modelo é composto pelas seguintes tabelas, construídas a partir dos arquivos anuais via Power Query (M):

- **`Medidas`** — tabela auxiliar que organiza as medidas DAX do relatório
- **`Deputados_Legislatura57`** — cadastro dos deputados da 57ª Legislatura
- **`Gastos_Legislatura57`** — despesas da CEAP (Cota Parlamentar)
- **`Proposicao_Legislatura57`** — proposições apresentadas
- **`Autor_Proposicao_Legislatura57`** — autoria das proposições
- **`Temas_Proposicao`** — classificação temática das proposições
- **`Calendario`** — tabela calendário para cálculos de tempo (YoY, YTD, MTD)

### Chave de relacionamento

As tabelas de despesas (`Gastos_Legislatura57`) são relacionadas à tabela de deputados (`Deputados_Legislatura57`) através do campo **`ideCadastro`** (em `Gastos_Legislatura57`) e **`id`** (em `Deputados_Legislatura57`) — identificador numérico exclusivo de cada parlamentar, válido em toda a API de Dados Abertos.

### Diagrama de relacionamentos

```
Deputados_Legislatura57 (1) ──── (*) Gastos_Legislatura57
        │ 1
        │ (filtro único sentido)
        │ *
Autor_Proposicao_Legislatura57 (*) ◄──► (1) Proposicao_Legislatura57
                                              │ 1
                                              │ (bidirecional)
                                              │ *
                                        Temas_Proposicao
```

| Relacionamento | Cardinalidade | Direção do filtro |
|---|---|---|
| `Deputados_Legislatura57` → `Gastos_Legislatura57` | 1 : * | Único sentido |
| `Deputados_Legislatura57` → `Autor_Proposicao_Legislatura57` | 1 : * | Único sentido |
| `Proposicao_Legislatura57` ↔ `Autor_Proposicao_Legislatura57` | 1 : * | Bidirecional (necessário para cruzar filtros de proposição com autor) |
| `Proposicao_Legislatura57` ↔ `Temas_Proposicao` | 1 : * | Bidirecional (necessário pois uma proposição pode ter múltiplos temas, em múltiplas linhas) |

> ⚠️ **Atenção:** o campo `nuDeputadoId`, presente nos arquivos de CEAP, **não** deve ser usado para esse relacionamento — ele é um identificador interno do sistema de controle de cotas e não é reconhecido pelos demais endpoints/arquivos da API.

Registros com `ideCadastro` nulo (lançamentos de gabinetes de liderança partidária, não vinculados a um deputado individual) são filtrados na camada do Power Query.

### Principais campos da tabela `Deputados_Legislatura57`

| Campo | Descrição |
|---|---|
| `id` | Identificador único do parlamentar (= `ideCadastro` em `Gastos_Legislatura57`) |
| `nome` | Nome do parlamentar |
| `siglaPartido` | Sigla do partido |
| `siglaUf` | UF pela qual foi eleito |
| `idLegislatura` | Identificador da legislatura |
| `email` | E-mail institucional |

### Principais campos da tabela `Gastos_Legislatura57` (CEAP)

| Campo | Descrição |
|---|---|
| `ideCadastro` | Identificador único do parlamentar (chave de relacionamento) |
| `ideDocumento` | Identificador do documento comprobatório da despesa |
| `indTipoDocumento` | Tipo de documento fiscal (nota fiscal, recibo, etc.) |
| `numAno` / `nuLegislatura` | Ano de competência e legislatura |
| `numLote` | Lote de documentos para reembolso |
| `nuCarteiraParlamentar` | Número da carteira parlamentar |
| `datEmissao` | Data de emissão do documento |
| `cpf` | CPF do parlamentar beneficiário |

### Principais campos da tabela `Proposicao_Legislatura57`

| Campo | Descrição |
|---|---|
| `id` | Identificador único da proposição |
| `ano` | Ano de apresentação |
| `codTipo` | Código do tipo de proposição (139, 140, 136, 291) |
| `descricaoTipo` | Descrição do tipo (PL, PLP, PEC, MPV) |
| `numero` | Número da proposição |
| `dataApresentacao` | Data de apresentação |
| `ementa` / `ementaDetalhada` | Resumo e descrição completa do conteúdo |
| `keywords` | Palavras-chave associadas |

### Principais campos da tabela `Autor_Proposicao_Legislatura57`

| Campo | Descrição |
|---|---|
| `idProposicao` | Chave de relacionamento com `Proposicao_Legislatura57` |
| `idDeputadoAutor` | Chave de relacionamento com `Deputados_Legislatura57` |
| `nomeAutor` | Nome do autor (nem todos os autores são parlamentares) |
| `tipoAutor` / `codTipoAutor` | Tipo do autor |
| `proponente` | Indica se é o primeiro signatário (proponente) ou coautor |
| `ordemAssinatura` | Ordem da assinatura na proposição |
| `siglaPartidoAutor` / `siglaUFAutor` | Partido e UF do autor no momento da apresentação |

### Principais campos da tabela `Temas_Proposicao`

| Campo | Descrição |
|---|---|
| `idProposicao` | Chave de relacionamento com `Proposicao_Legislatura57` |
| `codTema` | Código do tema |
| `tema` | Descrição do tema |
| `ano` | Ano de apresentação da proposição |
| `siglaTipo` / `numero` | Identificação complementar da proposição |

> ℹ️ Uma proposição pode estar associada a mais de um tema — nesse caso, `Temas_Proposicao` traz múltiplas linhas para o mesmo `idProposicao`.

A documentação completa de todos os campos está disponível em [`Campos_Deputados.docx`](./Campos_Deputados.docx).

## 📐 Medidas DAX (destaques)

- Gasto total, variação YoY, YTD, MTD
- Rankings por deputado, partido e UF (exigem contexto de tabela/matriz — em cartões retornam valor fixo "1")
- Participação percentual de gasto
- Contagem de proposições, com separação entre autoria principal e coautoria
- Custo por proposição principal (cost-per-principal-proposition)

## ⚠️ Lições aprendidas / limitações conhecidas

- `SUMMARIZECOLUMNS` dentro de tabelas calculadas falha quando as medidas referenciam colunas de múltiplas tabelas.
- Relacionamentos bidirecionais (ou `CROSSFILTER()` explícito) são necessários para cruzamento de filtros entre as tabelas de Temas, Proposições e Autores.

## 🛠️ Ferramentas

- Power BI Desktop (Power Query / M e DAX)
- Dados abertos da Câmara dos Deputados (API e exportações em Excel)

## 📄 Licença dos dados

Os dados utilizados são públicos e disponibilizados pela Câmara dos Deputados através do Portal de Dados Abertos, sob a licença vigente do portal (https://dadosabertos.camara.leg.br/).
