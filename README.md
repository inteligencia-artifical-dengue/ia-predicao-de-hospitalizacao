# Predição de Hospitalização em Casos Confirmados de Dengue

Projeto da disciplina de **Inteligência Artificial** (7ºJ SI - Noite), Faculdade de Computação e Informática, Universidade Presbiteriana Mackenzie.
Professor: Dr. Leandro Zerbinatti.

| Documentos | Links |
|---|---|
| **Relatório** | [`Predição___Dengue.pdf`](./Predição___Dengue.pdf) |
| **Vídeo de apresentação** | [`Vídeo do YouTube`]([link](https://www.youtube.com/watch?v=GBB3h-Tl3-A)) |
| **Notebooks** | [`dengue_IA/notebooks/`](./dengue_IA/notebooks) |

## Integrantes

| Nome | E-mail | RA |
|---|---|---|
| Geovanna da Silva Lima | 10420059@mackenzista.com.br | 10420059 |
| Pedro Henrique Mansano Fernandes | 10388037@mackenzista.com.br | 10388037 |
| Priscila Raquel Herculano Correia | 10420400@mackenzista.com.br | 10420400 |

## Resumo

A dengue sobrecarrega os serviços de saúde em períodos de surto, e a triagem rápida de quem tem maior risco de agravamento é um desafio. Este projeto usa **Aprendizado de Máquina (scikit-learn)** para prever se um paciente com dengue **confirmada** será **hospitalizado**, usando apenas informações disponíveis no primeiro atendimento: idade, gestação, sintomas, comorbidades e o tempo entre o início dos sintomas e a notificação (`DIAS`).

O melhor modelo, **Gradient Boosting**, alcançou **AUC-ROC de 0,753** e **AUC-PR de 0,165** no teste (o acaso daria 0,047). Com o limiar ajustado para priorizar a sensibilidade, identifica **81% dos hospitalizados**, sinalizando 49% dos pacientes. A discriminação é **moderada**: a ferramenta apoia a triagem e **não substitui o julgamento clínico**.

## Problema

- **Tipo:** classificação binária supervisionada.
- **Alvo:** `HOSPITALIZ` (1 = Sim, 2 = Não). Apenas **4,7%** dos casos foram hospitalizados (classes muito desbalanceadas).
- **Por que hospitalização e não óbito:** o óbito é raro demais na base (~0,13%) para um modelo robusto.
- **Opção do projeto:** *Framework* (scikit-learn).

## Dados

- **Fonte:** SINAN/DATASUS, notificações de dengue (arboviroses), estado de São Paulo, 2025.
- **Link:** <https://dadosabertos.saude.gov.br/dataset/arboviroses-dengue/resource/5c9132a9-77c2-4b15-8afc-a43c58fc9ec0>
- **Escopo:** casos confirmados, com `CLASSI_FIN` ∈ {10, 11, 12} (Dengue, Dengue com sinais de alarme, Dengue grave). Essa codificação difere da do dicionário de dados oficial (que cita o esquema antigo 1 = Confirmado, 2 = Descartado) e foi validada diretamente na base.
- **Dataset bruto:** não está versionado por causa do tamanho (~450 MB, ~1,6 milhão de linhas e 121 colunas no arquivo nacional). Baixe pelo link acima e salve em `dengue_IA/data/DENGBR25.csv`.
- **Dicionário de dados:** [`dengue_IA/docs/dicionario_de_dados.pdf`](./dengue_IA/docs/dicionario_de_dados.pdf)

### Do dado bruto à amostra de modelagem

| Etapa | Casos |
|---|---|
| SP + casos confirmados | 886.244 |
| Com desfecho de hospitalização conhecido | 733.633 (34.517 hospitalizados, 4,7%) |
| Após remover nulos residuais | 729.776 |
| Amostra estratificada | 40.000 |
| Treino / Teste (80/20, estratificado) | 32.000 / 8.000 (4,7% de hospitalizados em ambos) |

### Variáveis usadas

- **Demográficas:** idade (`IDADE_ANOS`) e gestação (`CS_GESTANT`).
- **Tempo de doença:** `DIAS`, criada pelo grupo = `DT_NOTIFIC` − `DT_SIN_PRI` (valores fora de 0 a 60 dias tratados como ausentes).
- **Sinais e sintomas:** febre, mialgia, cefaleia, exantema, vômito, náusea, dor retro-orbital, dor nas costas, petéquias, leucopenia, artrite, artralgia, conjuntivite, prova do laço.
- **Comorbidades:** diabetes, hipertensão, doença hematológica, renal, hepática, autoimune, doença ácido-péptica.

### Variáveis excluídas

| Variável | Motivo |
|---|---|
| `CLASSI_FIN`, `DT_INTERNA`, `DT_OBITO` | **Vazamento de dados:** só existem após o desfecho (ex.: 90% dos casos graves foram internados). |
| `CS_RACA` (raça/cor) | Pode funcionar como *proxy* de acesso a leitos, e não de gravidade. O modelo reproduziria a desigualdade. |
| `CS_ESCOL_N` (escolaridade) | Quase metade dos registros sem informação, indicador socioeconômico e confundido com a idade. |
| `CS_SEXO` (sexo) | A taxa de hospitalização é praticamente igual entre os sexos (4,7% e 4,8%), então não traz informação. |

## Metodologia

1. **Preparação:** valores "ignorados" tratados como categoria própria, idades implausíveis (> 110 anos) como ausentes, amostragem e divisão treino/teste estratificadas.
2. **Pré-processamento em `Pipeline`:** *one-hot* na gestação, padronização de idade e `DIAS`, binárias mantidas em 0/1. Isso evita vazamento entre treino e teste.
3. **Modelos comparados:** classificador trivial (referência), Regressão Logística, Random Forest e Gradient Boosting (`HistGradientBoostingClassifier`).
4. **Desbalanceamento:** ponderação de classes (`class_weight="balanced"`, ~20× de peso na classe rara), que superou a subamostragem em testes preliminares.
5. **Validação:** validação cruzada estratificada (5 partições) e ajuste de hiperparâmetros por `RandomizedSearchCV`, com **AUC-PR** como critério.
6. **Limiar de decisão:** escolhido **só com o treino**, usando probabilidades *out-of-fold*: o de maior precisão entre os que atingem **recall ≥ 80%** (resultado: 0,360).
7. **Teste usado uma única vez**, com IC de 95% por *bootstrap* (1.000 reamostragens).
8. **Interpretabilidade:** razões de chances (Regressão Logística) e importância por permutação (modelo final).

## Resultados (conjunto de teste: 8.000 casos, 377 hospitalizados)

| Modelo | Limiar | AUC-ROC | AUC-PR | Recall | Precisão | % sinalizados |
|---|---|---|---|---|---|---|
| Classificador trivial | 0,50 | 0,489 | 0,047 | 0,024 | 0,025 | 4,5 |
| Regressão Logística | 0,50 | 0,726 | 0,125 | 0,602 | 0,099 | 28,7 |
| Random Forest | 0,50 | 0,746 | 0,157 | 0,634 | 0,112 | 26,7 |
| Gradient Boosting | 0,50 | 0,753 | 0,165 | 0,666 | 0,112 | 28,0 |
| **Gradient Boosting (final)** | **0,360** | **0,753** | **0,165** | **0,812** | **0,078** | **49,1** |

**Intervalos de confiança de 95% (Gradient Boosting):** AUC-ROC 0,723–0,779 · AUC-PR 0,135–0,199 · Recall 0,767–0,850 · Precisão 0,070–0,087.
**Diferença de AUC-PR vs. Regressão Logística:** 0,040 (IC 0,019–0,061), **significativa**, pois o intervalo não inclui zero.

**Matriz de confusão (limiar 0,360):**

| | Previsto não | Previsto sim |
|---|---|---|
| **Real não** | 3.999 | 3.624 |
| **Real sim** | 71 | 306 |

### Principais conclusões

- **Sem sobreajuste:** AUC-ROC de 0,752 na validação cruzada e 0,753 no teste.
- **Grupo de menor risco:** entre os pacientes não sinalizados, só 1,7% foram hospitalizados (contra 4,7% no geral).
- **Variável mais importante:** `DIAS`. O risco sobe até cerca de 2 semanas de doença e cai depois (relação **não linear**, o que favorece modelos de árvore). Em seguida vêm leucopenia, idade e vômito.
- **Fatores associados a maior chance (razão de chances):** leucopenia (4,2), doença hematológica (3,0), gestação (2,4 a 3,6), vômito (1,7), diabetes (1,6) e hipertensão (1,6). Sintomas típicos de quadros leves, como conjuntivite (0,54) e cefaleia (0,57), associam-se a menor chance. **São associações estatísticas, não causas.**
- **Acurácia não é adequada aqui:** o modelo final tem 54%, enquanto responder sempre "não hospitalizado" daria 95%.

## Limitações e aspectos éticos

- **Discriminação moderada:** a ficha do SINAN não traz sinais vitais, plaquetas nem hematócrito, que são os principais critérios clínicos de internação.
- **O desfecho reflete conduta e disponibilidade de leitos**, não apenas gravidade (o alvo registra quem *foi* internado).
- **Premissa:** `DIAS` e `LEUCOPENIA` estão disponíveis no primeiro atendimento. Onde isso não ocorrer, o desempenho tende a ser menor.
- **Probabilidades não calibradas** (a ponderação de classes as infla): use o escore para **ordenar o risco**, não como risco real.
- **Escopo:** um estado e um ano.
- **Dados:** públicos e sem identificação direta, mas de saúde (sensíveis, LGPD). Uso exclusivamente acadêmico.
- **O limiar de 0,360 é uma decisão clínica e de gestão** e deve ser revisto por quem for usar o modelo.
- **Uso pretendido:** apoio à triagem, **nunca** critério único de decisão.

A discussão completa está na seção 3 do relatório.

## Estrutura do repositório

```
.
├── README.md
├── LICENSE
├── Predição___Dengue.pdf                    # relatório completo (N2)
└── dengue_IA/
    ├── requirements.txt
    ├── data/                                # crie esta pasta e coloque aqui o DENGBR25.csv (não versionado)
    ├── docs/
    │   └── dicionario_de_dados.pdf
    └── notebooks/
        ├── analise_exploratoria_dengue.ipynb  # N1: EDA e preparação dos dados
        ├── modelagem_dengue.ipynb             # N2: modelagem, avaliação e interpretação
        ├── dengue_sp_treino.csv               # gerado pelo notebook de EDA (não versionado)
        └── dengue_sp_teste.csv                # gerado pelo notebook de EDA (não versionado)
```

## Como executar

1. Baixe o CSV de dengue 2025 no link da seção **Dados** e salve em `dengue_IA/data/DENGBR25.csv`.
2. Instale as dependências:
   ```bash
   pip install -r dengue_IA/requirements.txt
   ```
3. Execute `dengue_IA/notebooks/analise_exploratoria_dengue.ipynb`. Ele filtra a base, faz a EDA e gera os CSVs de treino e teste.
4. Execute `dengue_IA/notebooks/modelagem_dengue.ipynb` (~1 min). Ele treina, avalia e interpreta os modelos.

Os caminhos são resolvidos automaticamente a partir da pasta `dengue_IA/`. A semente aleatória é fixa (`SEED = 42`) para reprodutibilidade.

## Referências principais

- BRASIL. Ministério da Saúde. *Dengue: diagnóstico e manejo clínico: adulto e criança*. 6. ed. Brasília, 2024.
- BRASIL. Ministério da Saúde. SINAN: dados abertos, arboviroses – dengue. <https://dadosabertos.saude.gov.br/dataset/arboviroses-dengue>
- BRASIL. Lei nº 13.709/2018 (LGPD).
- PEDREGOSA, F. et al. Scikit-learn: machine learning in Python. *JMLR*, v. 12, 2011.
- SAITO, T.; REHMSMEIER, M. The precision-recall plot is more informative than the ROC plot when evaluating binary classifiers on imbalanced datasets. *PLoS ONE*, v. 10, n. 3, 2015.

A lista completa está no relatório.

## Status

- ✅ **N1:** proposta, dataset e análise exploratória.
- ✅ **N2:** modelagem, avaliação, interpretabilidade e relatório completo.

## Licença

Veja o arquivo [LICENSE](./LICENSE).
