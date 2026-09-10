# Predição de Hospitalização em Casos Confirmados de Dengue

Projeto da disciplina de Inteligência Artificial (7ºJ SI - Noite), Faculdade de
Computação e Informática, Universidade Presbiteriana Mackenzie.

**Integrantes:**
- Pedro Henrique Mansano Fernandes - 10388037@mackenzista.com.br - RA: 10388037
- [Nome completo] - [email] - RA [000000]
- [Nome completo] - [email] - RA [000000]
- [Nome completo] - [email] - RA [000000]

## Objetivo

Prever, a partir de informações disponíveis apenas no primeiro atendimento
(sintomas, comorbidades e dados demográficos), se um paciente com dengue
confirmada será hospitalizado. Alvo: `HOSPITALIZ` (1 = Sim, 2 = Não).

## Dados

- **Fonte:** SINAN/DATASUS, notificações de dengue de São Paulo, 2025.
- **Link:** https://dadosabertos.saude.gov.br/dataset/arboviroses-dengue
- **Escopo:** casos confirmados (`CLASSI_FIN` ∈ {10, 11, 12} — Dengue, Dengue
  com sinais de alarme, Dengue grave). Ver nota sobre codificação no notebook.
- O dataset bruto não está versionado neste repositório por tamanho
  (~450 MB). Baixe pelo link acima e ajuste o caminho no notebook.

## Estrutura do repositório

```
├── README.md
├── notebooks/
│   └── analise_exploratoria_dengue.ipynb   # EDA e preparação dos dados
├── data/
│   └── (coloque aqui o DENGBR25.csv baixado do link acima)
└── docs/
    └── dicionario_de_dados.pdf                # dicionário de dados de referência
```

## Como executar

1. Baixe o CSV de notificações de dengue 2025 no link acima e salve em `data/`.
2. Instale as dependências:
3.  ``` pip install -r requirements.txt   ```
4. Abra `notebooks/analise_exploratoria_dengue.ipynb` e ajuste a variável
   `CSV_PATH` na célula de configuração.
5. Execute o notebook célula a célula.

## Metodologia (resumo)

- Filtragem por UF (SP) e classificação confirmada, evitando variáveis com
  vazamento de dados (ex.: `DT_INTERNA`, `DT_OBITO`).
- Análise exploratória: distribuição do alvo, perfil demográfico, prevalência
  de sintomas/comorbidades por desfecho.
- Tratamento de valores ignorados, remoção de outliers de idade, amostragem
  estratificada (~40k registros) e split treino/teste (80/20).

## Aspectos éticos

Dados públicos e anonimizados pelo Ministério da Saúde, utilizados
exclusivamente para fins acadêmicos. Discussão completa no relatório do
projeto (seção 6).

## Status

🔄 Em desenvolvimento — Primeiro Bimestre (N1): proposta, dataset e análise
exploratória concluídos. Modelagem prevista para o Segundo Bimestre (N2).
