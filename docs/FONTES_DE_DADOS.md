# 🗃️ Documentação de Fontes de Dados

Este documento contém a documentação completa das fontes de dados utilizadas no Dashboard de Cursos Online, incluindo detalhamento das tabelas, origens e transformações aplicadas.

[← Voltar para README](../README.md)

---

## 📋 Índice

- [Visão Geral](#visão-geral)
- [Diagrama de Relacionamentos](#diagrama-de-relacionamentos)
- [Tabelas Fato](#tabelas-fato)
  - [inscricoes_marketing](#inscricoes_marketing)
- [Tabelas Dimensão](#tabelas-dimensão)
  - [cursos](#cursos)
  - [conclusoes](#conclusoes)
  - [calendario](#calendario)
- [Tabelas Auxiliares](#tabelas-auxiliares)
  - [_Medidas](#_medidas)
  - [_Indicadores](#_indicadores)

---

## 📖 Visão Geral

O modelo de dados utiliza uma estrutura em estrela (Star Schema) com as seguintes características:

| Tipo | Quantidade | Descrição |
|------|------------|-----------|
| **Tabelas Fato** | 1 | Contém os dados transacionais de inscrições |
| **Tabelas Dimensão** | 3 | Cursos, conclusões e calendário |
| **Tabelas Auxiliares** | 2 | Medidas e indicadores |

### Conector Utilizado

- **Tipo**: Arquivo CSV (`Csv.Document`)
- **Modo de Importação**: Import Mode
- **Encoding**: UTF-8 (65001) / Windows-1252 (1252)

---

## 🔗 Diagrama de Relacionamentos

```
┌─────────────────────┐     ┌─────────────────────┐
│      calendario     │     │       cursos        │
│─────────────────────│     │─────────────────────│
│ Data (PK)           │     │ ID_CURSO (PK)       │
│ Dia da Semana       │     │ NOME_CURSO          │
│ Nome do Dia         │     │ CATEGORIA_CURSO     │
│ ...                 │     │ RANKING_CURSO       │
└──────────┬──────────┘     └──────────┬──────────┘
           │ 1                         │ 1
           │                           │
           │ N                         │ N
┌──────────┴──────────────────────────┴──────────┐
│              inscricoes_marketing               │
│─────────────────────────────────────────────────│
│ ID_INSCRICAO (PK)                               │
│ ID_ALUNO                                        │
│ ID_CURSO (FK) ──────────────────────────────────┤
│ DATA_INSCRICAO (FK) ────────────────────────────┤
│ PRECO_PAGO                                      │
│ CANAL_MARKETING                                 │
└────────────────────────┬────────────────────────┘
                         │ 1
                         │
                         │ 1
┌────────────────────────┴────────────────────────┐
│                   conclusoes                     │
│─────────────────────────────────────────────────│
│ ID_INSCRICAO (PK/FK)                            │
│ DATA_CONCLUSAO                                  │
└─────────────────────────────────────────────────┘
```

### Detalhamento dos Relacionamentos

| De (Tabela) | Coluna | Para (Tabela) | Coluna | Cardinalidade | Filtro Cruzado |
|-------------|--------|---------------|--------|---------------|----------------|
| inscricoes_marketing | ID_CURSO | cursos | ID_CURSO | N:1 | Única direção |
| inscricoes_marketing | DATA_INSCRICAO | calendario | Data | N:1 | Única direção |
| conclusoes | ID_INSCRICAO | inscricoes_marketing | ID_INSCRICAO | 1:1 | Bidirecional |

---

## 📊 Tabelas Fato

### inscricoes_marketing

**Descrição**: Tabela principal de fatos contendo todas as inscrições realizadas nos cursos, incluindo informações de marketing.

**Origem**: Arquivo CSV  
**Caminho**: `dados/inscricoes_marketing.csv`  
**Grupo de Consultas**: Fatos  
**Encoding**: UTF-8 (65001)

#### Colunas

| Coluna | Tipo de Dado | Descrição | Sumarização |
|--------|--------------|-----------|-------------|
| ID_INSCRICAO | Inteiro (Int64) | Identificador único da inscrição | Nenhuma |
| ID_ALUNO | Inteiro (Int64) | Identificador do aluno | Soma |
| ID_CURSO | Inteiro (Int64) | Identificador do curso (FK) | Nenhuma |
| DATA_INSCRICAO | Data | Data em que a inscrição foi realizada | Nenhuma |
| PRECO_PAGO | Decimal (Currency) | Valor pago pela inscrição em R$ | Soma |
| CANAL_MARKETING | Texto | Canal de marketing de origem | Nenhuma |

#### Transformações Aplicadas (Power Query)

```powerquery
let
    Fonte = Csv.Document(
        File.Contents("dados/inscricoes_marketing.csv"),
        [Delimiter=",", Columns=6, Encoding=65001, QuoteStyle=QuoteStyle.None]
    ),
    #"Cabeçalhos Promovidos" = Table.PromoteHeaders(Fonte, [PromoteAllScalars=true]),
    #"Tipo Alterado" = Table.TransformColumnTypes(
        #"Cabeçalhos Promovidos",
        {
            {"ID_INSCRICAO", Int64.Type}, 
            {"ID_ALUNO", Int64.Type}, 
            {"ID_CURSO", Int64.Type}, 
            {"DATA_INSCRICAO", type date}, 
            {"PRECO_PAGO", Currency.Type}, 
            {"CANAL_MARKETING", type text}
        }
    )
in
    #"Tipo Alterado"
```

**Passos de Transformação**:
1. Leitura do arquivo CSV com delimitador vírgula
2. Promoção da primeira linha como cabeçalhos
3. Tipagem das colunas (inteiros, data e moeda)

---

## 📁 Tabelas Dimensão

### cursos

**Descrição**: Tabela dimensão contendo informações dos cursos disponíveis na plataforma.

**Origem**: Arquivo CSV  
**Caminho**: `dados/cursos.csv`  
**Grupo de Consultas**: Dimensões  
**Encoding**: UTF-8 (65001)

#### Colunas

| Coluna | Tipo de Dado | Descrição | Sumarização |
|--------|--------------|-----------|-------------|
| ID_CURSO | Inteiro (Int64) | Identificador único do curso (PK) | Nenhuma |
| NOME_CURSO | Texto | Nome do curso | Nenhuma |
| CATEGORIA_CURSO | Texto | Categoria do curso | Nenhuma |
| RANKING_CURSO | Inteiro | Ranking do curso por receita (calculada) | Nenhuma |

#### Transformações Aplicadas (Power Query)

```powerquery
let
    Fonte = Csv.Document(
        File.Contents("dados/cursos.csv"),
        [Delimiter=",", Columns=3, Encoding=65001, QuoteStyle=QuoteStyle.None]
    ),
    #"Cabeçalhos Promovidos" = Table.PromoteHeaders(Fonte, [PromoteAllScalars=true]),
    #"Tipo Alterado" = Table.TransformColumnTypes(
        #"Cabeçalhos Promovidos",
        {
            {"ID_CURSO", Int64.Type}, 
            {"NOME_CURSO", type text}, 
            {"CATEGORIA_CURSO", type text}
        }
    )
in
    #"Tipo Alterado"
```

**Passos de Transformação**:
1. Leitura do arquivo CSV com delimitador vírgula
2. Promoção da primeira linha como cabeçalhos
3. Tipagem das colunas

---

### conclusoes

**Descrição**: Tabela dimensão contendo os registros de conclusão de cursos pelos alunos.

**Origem**: Arquivo CSV  
**Caminho**: `dados/conclusoes.csv`  
**Grupo de Consultas**: Dimensões  
**Encoding**: Windows-1252 (1252)

#### Colunas

| Coluna | Tipo de Dado | Descrição | Sumarização |
|--------|--------------|-----------|-------------|
| ID_INSCRICAO | Inteiro (Int64) | Identificador da inscrição (PK/FK) | Nenhuma |
| DATA_CONCLUSAO | Data | Data em que o curso foi concluído | Nenhuma |

#### Transformações Aplicadas (Power Query)

```powerquery
let
    Fonte = Csv.Document(
        File.Contents("dados/conclusoes.csv"),
        [Delimiter=",", Columns=2, Encoding=1252, QuoteStyle=QuoteStyle.None]
    ),
    #"Cabeçalhos Promovidos" = Table.PromoteHeaders(Fonte, [PromoteAllScalars=true]),
    #"Tipo Alterado" = Table.TransformColumnTypes(
        #"Cabeçalhos Promovidos",
        {
            {"ID_INSCRICAO", Int64.Type}, 
            {"DATA_CONCLUSAO", type date}
        }
    )
in
    #"Tipo Alterado"
```

**Passos de Transformação**:
1. Leitura do arquivo CSV com delimitador vírgula
2. Promoção da primeira linha como cabeçalhos
3. Tipagem das colunas

---

### calendario

**Descrição**: Tabela dimensão de calendário gerada automaticamente via Power Query para análises temporais.

**Origem**: Gerada via código M (Power Query)  
**Grupo de Consultas**: Dimensões  
**Categoria de Dados**: Time (Tabela de Datas)

**Período**: 01/01/2025 a 01/01/2026

#### Colunas

| Coluna | Tipo de Dado | Descrição | Ordenação |
|--------|--------------|-----------|-----------|
| Data | Data | Data completa (PK) | - |
| Dia da Semana | Inteiro | Número do dia da semana (1-7) | - |
| Nome do Dia | Texto | Nome completo do dia | Por Dia da Semana |
| Dia | Texto | Abreviação do dia (3 letras) | Por Dia da Semana |
| Semana do Ano | Inteiro | Número da semana no ano | - |
| Semana | Texto | "Sem " + número da semana | Por Semana do Ano |
| Nome do Mês | Texto | Nome completo do mês | Por Num Mês |
| Num Mês | Inteiro | Número do mês (1-12) | - |
| Ano | Inteiro | Ano (ex: 2025) | - |
| Ano-Mês | Texto | Formato "AA-MM" | - |
| Trimestre | Inteiro | Número do trimestre (1-4) | - |
| Trimestre Texto | Texto | "Trim " + número | - |
| Ano-Trimestre | Texto | Formato "AAAA-TrimN" | - |

#### Transformações Aplicadas (Power Query)

```powerquery
let
    StartDate = #date(2025, 1, 1),
    EndDate = #date(2026, 1, 1),
    Source = List.Dates(StartDate, Duration.Days(EndDate - StartDate) + 1, #duration(1, 0, 0, 0)),
    TableFromList = Table.FromList(Source, Splitter.SplitByNothing(), null, null, ExtraValues.Error),
    RenamedColumns = Table.RenameColumns(TableFromList,{{"Column1", "Date"}}),
    ChangedType = Table.TransformColumnTypes(RenamedColumns,{{"Date", type date}}),
    #"Colunas Renomeadas" = Table.RenameColumns(ChangedType,{{"Date", "Data"}}),
    #"Dia da Semana Inserido" = Table.AddColumn(#"Colunas Renomeadas", "Dia da Semana", each Date.DayOfWeek([Data], 1)),
    #"Nome do Dia Inserido" = Table.AddColumn(#"Dia da Semana Inserido", "Nome do Dia", each Date.DayOfWeekName([Data]), type text),
    #"Primeiros Caracteres Inseridos" = Table.AddColumn(#"Nome do Dia Inserido", "Dia", each Text.Start(Text.Proper([Nome do Dia]), 3), type text),
    #"Semana do Ano Inserida" = Table.AddColumn(#"Primeiros Caracteres Inseridos", "Semana do Ano", each Date.WeekOfYear([Data], 1)),
    #"Coluna Mesclada Inserida" = Table.AddColumn(#"Semana do Ano Inserida", "Semana", each Text.Combine({"Sem ", Text.From([Semana do Ano], "pt-BR")}), type text),
    #"Nome do Mês Inserido" = Table.AddColumn(#"Coluna Mesclada Inserida", "Nome do Mês", each Date.MonthName([Data]), type text),
    #"Mês Inserido" = Table.AddColumn(#"Nome do Mês Inserido", "Num Mês", each Date.Month([Data]), Int64.Type),
    #"Ano Inserido" = Table.AddColumn(#"Mês Inserido", "Ano", each Date.Year([Data]), Int64.Type),
    #"Trimestre Inserido" = Table.AddColumn(#"Ano Inserido", "Trimestre", each Date.QuarterOfYear([Data]), Int64.Type),
    #"Trimestre Formatado" = Table.AddColumn(#"Trimestre Inserido", "Trimestre Texto", each "Trim " & Text.From([Trimestre]), type text),
    #"Ano-Trimestre Inserido" = Table.AddColumn(#"Trimestre Formatado", "Ano-Trimestre", each Text.From([Ano]) & "-Trim" & Text.From([Trimestre]), type text),
    #"Coluna Personalizada Adicionada" = Table.AddColumn(#"Ano-Trimestre Inserido", "Ano-Mês", each Text.Combine({Date.ToText([Data], "yy"), "-", Date.ToText([Data], "MM")}), type text),
    #"Changed Type" = Table.TransformColumnTypes(#"Coluna Personalizada Adicionada",{{"Semana do Ano", Int64.Type}, {"Dia da Semana", Int64.Type}}),
    #"Capitalized Each Word" = Table.TransformColumns(#"Changed Type",{{"Nome do Mês", Text.Proper, type text}})
in
    #"Capitalized Each Word"
```

**Passos de Transformação**:
1. Geração de lista de datas entre as datas inicial e final
2. Conversão para tabela
3. Adição de colunas de dia da semana (número e nome)
4. Adição de abreviação do dia (3 caracteres)
5. Adição de semana do ano (número e texto)
6. Adição de mês (número e nome)
7. Adição de ano
8. Adição de trimestre (número e texto)
9. Adição de colunas combinadas (Ano-Mês, Ano-Trimestre)
10. Formatação de texto (capitalização)

---

## 🔧 Tabelas Auxiliares

### _Medidas

**Descrição**: Tabela auxiliar criada para armazenar as medidas DAX do modelo. Esta é uma tabela vazia utilizada apenas como container para as medidas.

**Origem**: Gerada via código M (tabela vazia)

> Para detalhes completos sobre as medidas, consulte a [Documentação de Medidas](MEDIDAS.md).

---

### _Indicadores

**Descrição**: Tabela calculada que mapeia os indicadores para exibição dinâmica no dashboard, permitindo que o usuário selecione qual medida deseja visualizar.

**Origem**: Tabela Calculada (DAX)

#### Colunas

| Coluna | Descrição | Visível |
|--------|-----------|---------|
| _Indicadores | Nome amigável do indicador | ✅ Sim |
| _Indicadores Campos | Referência à medida | ❌ Não |
| _Indicadores Pedido | Ordem de exibição | ❌ Não |

#### Fórmula DAX

```dax
{
    ("Receita", NAMEOF('_Medidas'[VLR_Receita]), 0),
    ("Ticket Médio", NAMEOF('_Medidas'[Ticket_Medio]), 1),
    ("Receita MTD", NAMEOF('_Medidas'[Receita_MTD]), 2),
    ("Taxa Conclusão", NAMEOF('_Medidas'[Taxa_Conclusao]), 3),
    ("Inscrições Totais", NAMEOF('_Medidas'[QTD_Inscricoes]), 4),
    ("Inscrições Concluidas", NAMEOF('_Medidas'[QTD_Inscricoes_Concluidas]), 5)
}
```

---

## 📂 Estrutura de Arquivos de Dados

Os arquivos de dados esperados devem estar organizados da seguinte forma:

```
dados/
├── cursos.csv              # Cadastro de cursos
├── inscricoes_marketing.csv # Inscrições e dados de marketing
└── conclusoes.csv          # Registros de conclusão
```

### Formato dos Arquivos CSV

| Arquivo | Delimitador | Encoding | Colunas |
|---------|-------------|----------|---------|
| cursos.csv | Vírgula (,) | UTF-8 | ID_CURSO, NOME_CURSO, CATEGORIA_CURSO |
| inscricoes_marketing.csv | Vírgula (,) | UTF-8 | ID_INSCRICAO, ID_ALUNO, ID_CURSO, DATA_INSCRICAO, PRECO_PAGO, CANAL_MARKETING |
| conclusoes.csv | Vírgula (,) | Windows-1252 | ID_INSCRICAO, DATA_CONCLUSAO |

---

[← Voltar para README](../README.md)
