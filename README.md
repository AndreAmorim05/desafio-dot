# 📊 Dashboard de Cursos Online

Dashboard de análise de dados para acompanhamento de inscrições, receitas e métricas de cursos online. Este projeto foi desenvolvido no Power BI e exportado no formato PBIP (Power BI Project).

## 📋 Índice

- [Sobre o Projeto](#sobre-o-projeto)
- [Estrutura do Dashboard](#estrutura-do-dashboard)
- [Como Utilizar](#como-utilizar)
- [Documentação Adicional](#documentação-adicional)
- [Exportar para PBIX](#exportar-para-pbix)

## 📖 Sobre o Projeto

Este dashboard foi criado para fornecer uma visão completa das métricas de negócio relacionadas a cursos online, permitindo análises de:

- **Receita**: Acompanhamento de receita total, mensal e variações
- **Inscrições**: Monitoramento de inscrições totais e concluídas
- **Taxa de Conclusão**: Análise do percentual de alunos que finalizam os cursos
- **Performance por Canal**: Análise de desempenho por canal de marketing
- **Análise por Curso**: Ranking e performance individual dos cursos

## 📄 Estrutura do Dashboard

O dashboard conta com as seguintes páginas:

| Página | Descrição |
|--------|-----------|
| **Dashboard** | Página principal com visão geral dos indicadores, gráficos de evolução temporal, análises por canal de marketing e performance dos cursos |
| **Página 1** | Página auxiliar para análises adicionais |

### Indicadores Principais

- Receita Total
- Ticket Médio
- Receita MTD (Month-to-Date)
- Taxa de Conclusão
- Inscrições Totais
- Inscrições Concluídas

## 🚀 Como Utilizar

### Pré-requisitos

- **Power BI Desktop** (versão que suporte o formato PBIP)
- Acesso aos arquivos de dados fonte (arquivos CSV)

### Clonando o Repositório

```bash
git clone https://github.com/AndreAmorim05/desafio-dot.git
cd desafio-dot
```

### Abrindo o Projeto

1. Navegue até a pasta do projeto clonado
2. Abra o arquivo `Dashboard.pbip` com o Power BI Desktop
3. Caso necessário, atualize os caminhos das fontes de dados para corresponder à sua estrutura local

> **Nota**: Os arquivos de dados CSV devem estar no caminho correto ou você deverá atualizar as fontes no Power Query.

## 📚 Documentação Adicional

Para informações mais detalhadas, consulte a documentação específica:

| Documentação | Descrição |
|--------------|-----------|
| [📐 Medidas (DAX)](docs/MEDIDAS.md) | Documentação completa das medidas DAX utilizadas no modelo |
| [🗃️ Fontes de Dados](docs/FONTES_DE_DADOS.md) | Detalhamento das tabelas, origens e transformações aplicadas |

## 💾 Exportar para PBIX

Para exportar este projeto para o formato padrão do Power BI (`.pbix`):

1. Abra o arquivo `Dashboard.pbip` no Power BI Desktop
2. Vá em **Arquivo** → **Salvar como**
3. Na janela de salvamento, escolha a extensão **Power BI Report (.pbix)**
4. Selecione o local desejado e clique em **Salvar**

> **Dica**: O formato PBIX é mais prático para compartilhamento, enquanto o formato PBIP é ideal para versionamento e colaboração via Git.

---

## 📁 Estrutura de Arquivos

```
desafio-dot/
├── Dashboard.pbip                    # Arquivo principal do projeto
├── Dashboard.Report/                 # Definições do relatório
│   ├── definition/
│   │   ├── pages/                    # Configurações das páginas
│   │   └── report.json              # Configurações gerais do relatório
│   └── StaticResources/             # Recursos estáticos (temas, imagens)
├── Dashboard.SemanticModel/          # Modelo semântico (dados)
│   └── definition/
│       ├── model.tmdl               # Definição do modelo
│       ├── relationships.tmdl       # Relacionamentos entre tabelas
│       └── tables/                  # Definições das tabelas e medidas
├── docs/                            # Documentação adicional
│   ├── MEDIDAS.md                   # Documentação das medidas
│   └── FONTES_DE_DADOS.md          # Documentação das fontes de dados
└── README.md                        # Este arquivo
```

---

## 📝 Licença

Este projeto foi desenvolvido como parte de um desafio técnico.

## 👤 Autor

**André Amorim**

- GitHub: [@AndreAmorim05](https://github.com/AndreAmorim05)
