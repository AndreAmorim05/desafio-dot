# 📐 Documentação de Medidas (DAX)

Este documento contém a documentação completa das medidas DAX utilizadas no Dashboard de Cursos Online.

[← Voltar para README](../README.md)

---

## 📋 Índice

- [Visão Geral](#visão-geral)
- [Medidas de Quantidade](#medidas-de-quantidade)
- [Medidas de Valores](#medidas-de-valores)
- [Medidas de Taxas](#medidas-de-taxas)
- [Tabela de Indicadores](#tabela-de-indicadores)

---

## 📖 Visão Geral

As medidas estão organizadas na tabela `_Medidas`, agrupadas por categorias de acordo com seu propósito:

| Categoria | Descrição |
|-----------|-----------|
| **Quantidades** | Contagens de inscrições e registros |
| **Valores** | Métricas monetárias (receita, ticket médio) |
| **Taxas** | Percentuais e variações |

---

## 📊 Medidas de Quantidade

### QTD_Inscricoes_Concluidas

**Descrição**: Quantidade total de inscrições que foram finalizadas (apenas inscrições concluídas).

**Fórmula DAX**:
```dax
DISTINCTCOUNT(conclusoes[ID_INSCRICAO])
```

**Formato**: Número inteiro  
**Pasta**: Quantidades

---

### QTD_Inscricoes

**Descrição**: Quantidade total de inscrições, incluindo finalizadas e não finalizadas.

**Fórmula DAX**:
```dax
DISTINCTCOUNT(inscricoes_marketing[ID_INSCRICAO])
```

**Formato**: Número inteiro  
**Pasta**: Quantidades

---

## 💰 Medidas de Valores

### VLR_Receita

**Descrição**: Valor total em reais que entrou das inscrições (compreende apenas valores de inscrições concluídas).

**Fórmula DAX**:
```dax
SUM(inscricoes_marketing[PRECO_PAGO])
```

**Formato**: Moeda (R$)  
**Pasta**: Valores

---

### Ticket_Medio

**Descrição**: Média de quanto cada inscrição gerou de receita.

**Fórmula DAX**:
```dax
DIVIDE([VLR_Receita], [QTD_Inscricoes], 0)
```

**Formato**: Moeda (R$)  
**Pasta**: Valores

**Observações**: Retorna 0 caso não haja inscrições (evita divisão por zero).

---

### Receita_MTD

**Descrição**: Receita gerada desde o início do mês até a data atual (Month-to-Date).

**Fórmula DAX**:
```dax
CALCULATE(
    [VLR_Receita],
    DATESMTD(inscricoes_marketing[DATA_INSCRICAO])
)
```

**Formato**: Moeda (R$)  
**Pasta**: Valores

**Observações**: Utiliza a função `DATESMTD` que retorna todas as datas desde o início do mês até a data máxima no contexto atual do filtro.

---

### VLR_Receita_Concluidos

**Descrição**: Receita gerada apenas pelos clientes que concluíram seus cursos.

**Fórmula DAX**:
```dax
CALCULATE(
    SUM(inscricoes_marketing[PRECO_PAGO]),
    FILTER(
        inscricoes_marketing,
        CONTAINS(
            conclusoes,
            conclusoes[ID_INSCRICAO],
            inscricoes_marketing[ID_INSCRICAO]
        )
    )
)
```

**Formato**: Moeda (R$)  
**Pasta**: Valores

**Observações**: Filtra as inscrições que possuem registro correspondente na tabela de conclusões.

---

### VLR_Receita_Mes_Anterior

**Descrição**: Cálculo da receita do mês anterior ao observado.

**Fórmula DAX**:
```dax
CALCULATE(
    [VLR_Receita], 
    DATEADD('calendario'[Data], -1, MONTH)
)
```

**Formato**: Moeda (R$)  
**Pasta**: Valores

**Observações**: Utiliza a função `DATEADD` para deslocar o contexto de data em um mês para trás.

---

## 📈 Medidas de Taxas

### Taxa_Conclusao

**Descrição**: Taxa (percentual) de inscrições que foram devidamente concluídas.

**Fórmula DAX**:
```dax
DIVIDE([QTD_Inscricoes_Concluidas], [QTD_Inscricoes], 0)
```

**Formato**: Percentual (%)  
**Pasta**: Taxas

**Observações**: Retorna 0 caso não haja inscrições.

---

### Variacao_Receita_MoM

**Descrição**: Taxa de variação da receita, calculada mês a mês (Month-over-Month).

**Fórmula DAX**:
```dax
VAR ReceitaAtual = [VLR_Receita]
VAR ReceitaAnterior = [VLR_Receita_Mes_Anterior]
RETURN
    DIVIDE(
        ReceitaAtual - ReceitaAnterior,
        ReceitaAnterior,
        0
    )
```

**Formato**: Percentual (%)  
**Pasta**: Taxas

**Observações**: Caso não haja valor no mês anterior, é atribuído o valor zero.

---

## 🏷️ Tabela de Indicadores

A tabela `_Indicadores` é uma tabela calculada que mapeia os indicadores para exibição dinâmica no dashboard:

| Indicador | Medida Associada | Ordem |
|-----------|------------------|-------|
| Receita | VLR_Receita | 0 |
| Ticket Médio | Ticket_Medio | 1 |
| Receita MTD | Receita_MTD | 2 |
| Taxa Conclusão | Taxa_Conclusao | 3 |
| Inscrições Totais | QTD_Inscricoes | 4 |
| Inscrições Concluídas | QTD_Inscricoes_Concluidas | 5 |

Esta tabela permite criar visualizações dinâmicas onde o usuário pode selecionar qual indicador deseja analisar.

---

## 📊 Coluna Calculada

### RANKING_CURSO (Tabela: cursos)

**Descrição**: Ranking dos cursos baseado na receita total gerada.

**Fórmula DAX**:
```dax
RANKX(
    ALL(cursos[ID_CURSO]),
    SUMX(
        FILTER(
            inscricoes_marketing,
            inscricoes_marketing[ID_CURSO] = EARLIER(cursos[ID_CURSO])
        ),
        inscricoes_marketing[PRECO_PAGO]
    ),
    ,
    DESC,
    DENSE
)
```

**Formato**: Número inteiro  
**Tabela**: cursos

**Observações**: Utiliza ranking denso (DENSE) em ordem decrescente, considerando todos os cursos independente do contexto de filtro.

---

[← Voltar para README](../README.md)
