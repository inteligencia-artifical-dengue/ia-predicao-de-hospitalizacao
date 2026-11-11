# Predição de Hospitalização em Casos Confirmados de Dengue

Projeto da disciplina de Inteligência Artificial (7ºJ SI - Noite), Faculdade de Computação e Informática, Universidade Presbiteriana Mackenzie.

**Integrantes:**
- Pedro Henrique Mansano Fernandes - 10388037@mackenzista.com.br - RA: 10388037
- Geovanna da Silva Lima - 10420059@mackenzista.com.br - RA 10420059
- Priscila Raquel Herculano Correia  - 10420400@mackenzista.com.br - RA 10420400

## Objetivo

Prever, a partir de informações disponíveis apenas no primeiro atendimento (sintomas, comorbidades e dados demográficos), se um paciente com dengue confirmada será hospitalizado. Alvo: `HOSPITALIZ` (1 = Sim, 2 = Não).

## Dados

- **Fonte:** SINAN/DATASUS, notificações de dengue de São Paulo, 2025.
- **Link:** https://dadosabertos.saude.gov.br/dataset/arboviroses-dengue/resource/5c9132a9-77c2-4b15-8afc-a43c58fc9ec0
- **Escopo:** casos confirmados (`CLASSI_FIN` ∈ {10, 11, 12} — Dengue, Dengue com sinais de alarme, Dengue grave). Ver nota sobre codificação no notebook.
- O dataset bruto não está versionado neste repositório por tamanho (~450 MB). Baixe pelo link acima e ajuste o caminho no notebook.

## Estrutura do repositório

```
dengue_IA/
├── requirements.txt
├── notebooks/
│   ├── analise_exploratoria_dengue.ipynb   # N1: EDA e preparação dos dados
│   ├── modelagem_dengue.ipynb              # N2: modelagem, avaliação e equidade
│   ├── dengue_sp_treino.csv                # gerado pelo notebook de EDA (não versionado)
│   └── dengue_sp_teste.csv                 # gerado pelo notebook de EDA (não versionado)
├── data/
│   └── (coloque aqui o DENGBR25.csv baixado do link acima)
└── docs/
    └── dicionario_de_dados.pdf             # dicionário de dados de referência
```

## Como executar

1. Baixe o CSV de notificações de dengue 2025 no link acima e salve em `dengue_IA/data/`.
2. Instale as dependências:
   ```
   pip install -r dengue_IA/requirements.txt
   ```
3. Execute `notebooks/analise_exploratoria_dengue.ipynb`, que gera os CSVs de treino e teste.
   Os caminhos são resolvidos automaticamente a partir da pasta `dengue_IA/`.
4. Execute `notebooks/modelagem_dengue.ipynb` (~1 min), que treina e avalia os modelos.

## Metodologia (resumo)

- Filtragem por UF (SP) e classificação confirmada, evitando variáveis com vazamento de dados (ex.: `DT_INTERNA`, `DT_OBITO`, `CLASSI_FIN`).
- Análise exploratória: distribuição do alvo, perfil demográfico, prevalência de sintomas/comorbidades por desfecho e tempo entre início dos sintomas e notificação (`DIAS`).
- Tratamento de valores ignorados, remoção de outliers de idade, amostragem estratificada (~40k registros) e split treino/teste (80/20).
- Modelagem: Regressão Logística, Random Forest e Gradient Boosting com ponderação de classes (vs. subamostragem), validação cruzada estratificada, ajuste de hiperparâmetros por AUC-PR e limiar de decisão escolhido para recall ≥ 80%.
- Avaliação no teste com intervalos de confiança por *bootstrap*, interpretabilidade (razões de chances e importância por permutação), ablação (`LEUCOPENIA`, `DIAS`, `CS_RACA`) e análise de equidade por sexo, raça/cor e faixa etária.

## Resultados (conjunto de teste)

| Modelo | AUC-ROC | AUC-PR | Recall | Precisão |
|---|---|---|---|---|
| Classificador trivial (estratificado) | 0,489 | 0,047 | 0,024 | 0,025 |
| Regressão Logística (limiar 0,5) | 0,730 | 0,128 | 0,637 | 0,102 |
| **Gradient Boosting (limiar 0,365)** | **0,760** (IC 95%: 0,731–0,785) | **0,171** | **0,801** | **0,081** |

## Aspectos éticos

Dados públicos e anonimizados pelo Ministério da Saúde, utilizados exclusivamente para fins acadêmicos. Discussão completa no relatório do projeto (seção 3) e na análise de equidade do notebook de modelagem.

## Status

✅ N1: proposta, dataset e análise exploratória concluídos.
🔄 N2: modelagem e avaliação concluídas no notebook; atualização do relatório em andamento.
