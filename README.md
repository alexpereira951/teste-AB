# Teste A/B — Otimização de Conversão e Receita

Projeto de **análise de dados e experimentação A/B** desenvolvido para uma grande loja online, com o objetivo de avaliar hipóteses de negócio e medir o impacto de uma mudança sobre **conversão, receita e tamanho médio dos pedidos**.

A análise combina **priorização de hipóteses com ICE e RICE**, tratamento e validação dos dados, análise exploratória, identificação de anomalias e **testes estatísticos de significância** para comparar os grupos de controle (A) e teste (B).

---

## 🎯 Objetivo do Projeto

O projeto busca responder, por meio de dados, duas questões principais:

- Quais hipóteses de negócio devem ser priorizadas considerando impacto, confiança, esforço e alcance?
- A mudança aplicada ao **Grupo B** produz uma diferença estatisticamente significativa em relação ao **Grupo A**?

As principais métricas analisadas foram:

- **Taxa de conversão**
- **Receita acumulada**
- **Tamanho médio do pedido**
- **Diferença relativa entre os grupos**

---

## 🧠 Abordagem / Arquitetura Técnica

O notebook segue um fluxo analítico dividido em cinco grandes etapas.

### 1. Preparação e otimização dos dados

Foram carregados os datasets de:

- hipóteses de negócio;
- pedidos;
- visitas.

Durante a preparação foram realizados:

- padronização dos nomes das colunas para `snake_case`;
- conversão das colunas de data para `datetime`;
- conversão da coluna `group` para tipo categórico;
- carregamento dos dados completos utilizando `dtype` e `parse_dates`;
- validação dos tipos e estruturas dos DataFrames;
- otimização do uso de memória.

No teste inicial com amostras, o uso de memória de `orders` foi reduzido de aproximadamente **13,1 KB para 3,6 KB**, enquanto `visits` passou de aproximadamente **7,2 KB para 1,4 KB**.

### 2. Validação da qualidade dos dados

Foram verificadas situações que poderiam comprometer o experimento:

- usuários presentes simultaneamente nos grupos A e B;
- registros duplicados;
- compatibilidade dos intervalos de datas entre visitas e pedidos;
- existência de receitas negativas;
- valores extremos de receita.

Foram identificados **181 usuários presentes nos dois grupos**, que foram removidos da análise para evitar ambiguidade na atribuição experimental.

Não foram encontradas linhas duplicadas, e os períodos de visitas e pedidos coincidem entre **01/08/2019 e 31/08/2019**.

### 3. Priorização de hipóteses

Foram utilizados dois frameworks:

#### ICE

$$
ICE = \frac{Impact \times Confidence}{Effort}
$$

O cálculo considera impacto, confiança e esforço necessário para implementar cada hipótese.

#### RICE

$$
RICE = \frac{Reach \times Impact \times Confidence}{Effort}
$$

O RICE acrescenta o **alcance (`reach`)** à avaliação, alterando a priorização em relação ao ICE.

A análise mostra, por exemplo, que a **Hipótese 8**, relacionada à inclusão de um formulário de inscrição nas principais páginas, sobe para a primeira posição no RICE devido ao seu alto alcance.

### 4. Análise do experimento A/B

Foram construídas análises temporais para os grupos A e B, incluindo:

- receita acumulada;
- tamanho médio acumulado do pedido;
- diferença relativa do tamanho médio;
- conversão acumulada.

As métricas foram agregadas por data e grupo para acompanhar a evolução do experimento ao longo do período analisado.

### 5. Testes de significância estatística

Para a conversão foi aplicado o **teste Z para duas proporções**, utilizando nível de significância:

$$
\alpha = 0,05
$$

Para o tamanho médio dos pedidos:

1. foi avaliada a normalidade com **Shapiro-Wilk**;
2. como os dados não apresentaram normalidade, foi utilizado o **teste de Mann-Whitney**.

A análise estatística foi executada tanto nos dados brutos quanto após a remoção das anomalias identificadas.

---

## 🧹 Tratamento de Anomalias

O projeto utiliza percentis para identificar valores potencialmente atípicos.

### Pedidos por usuário

A análise dos percentis 95 e 99 indicou que usuários com mais de **1 pedido** poderiam ser tratados como casos atípicos para a análise de conversão.

### Receita por pedido

O percentil 95 da receita foi utilizado como limite para identificar pedidos atípicos:

**Limite adotado: US$ 414,28**

Esse tratamento foi aplicado para avaliar se valores extremos estavam influenciando a comparação do tamanho médio dos pedidos.

---

## 📊 Principais Resultados

### Conversão — dados brutos

| Métrica | Grupo A | Grupo B |
|---|---:|---:|
| Conversão | 2,50% | 2,90% |
| Diferença relativa | — | **+15,98%** |
| P-valor | — | **0,0169** |

Com `p-valor < 0,05`, o notebook rejeita a hipótese nula de igualdade entre as taxas de conversão.

### Conversão — dados filtrados

Após a remoção dos usuários com mais de um pedido:

| Métrica | Grupo A | Grupo B |
|---|---:|---:|
| Conversão | 2,28% | 2,70% |
| Diferença relativa | — | **+18,30%** |
| P-valor | — | **0,0094** |

A diferença relativa aumentou de **15,98% para 18,30%** após o tratamento dos usuários considerados anômalos.

### Tamanho médio do pedido — dados brutos

Apesar dos valores médios observados de:

- Grupo A: **US$ 113,70**
- Grupo B: **US$ 145,35**

o teste de Mann-Whitney apresentou:

**p-valor = 0,8622**

Portanto, a análise do notebook não encontrou diferença estatisticamente significativa entre os grupos nessa métrica.

### Tamanho médio do pedido — dados filtrados

Após a remoção dos pedidos acima de US$ 414,28:

- Grupo A: **US$ 83,09**
- Grupo B: **US$ 78,33**
- **p-valor = 0,7324**

O resultado continua sem diferença estatisticamente significativa.

### Síntese

Os resultados apresentados no notebook indicam:

- **diferença estatisticamente significativa na conversão** entre A e B;
- aumento relativo da conversão do Grupo B após o tratamento das anomalias;
- **ausência de diferença estatisticamente significativa no tamanho médio dos pedidos**;
- presença de valores extremos capazes de alterar substancialmente a leitura do ticket médio.

Com base nesses resultados, o notebook recomenda o encerramento do teste e a implementação da versão B em 100% do tráfego.

---

## 🗂️ Estrutura do Repositório

```text
teste AB/
│
├── datasets/
│   ├── hypotheses_us.csv
│   ├── orders_us.csv
│   └── visits_us.csv
│
├── notebooks/
│   └── notebook.ipynb
│
└── requirements.txt
```

### Principais diretórios e arquivos

| Caminho | Descrição |
|---|---|
| `datasets/` | Contém os dados utilizados na análise |
| `datasets/hypotheses_us.csv` | Hipóteses e atributos utilizados nos frameworks ICE/RICE |
| `datasets/orders_us.csv` | Dados de pedidos e receita |
| `datasets/visits_us.csv` | Dados de visitas por data e grupo |
| `notebooks/` | Contém o notebook de análise |
| `notebooks/notebook.ipynb` | Implementação completa do processo analítico |
| `requirements.txt` | Dependências necessárias para executar o projeto |

---

## 🛠️ Stack Tecnológica

- 🐍 **Python**
- 🐼 **Pandas** — manipulação e transformação dos dados
- 🔢 **NumPy** — cálculos numéricos e percentis
- 📊 **Plotly Express / Plotly Graph Objects** — visualizações interativas
- 📐 **SciPy** — testes estatísticos
- 📓 **Jupyter Notebook** — desenvolvimento e documentação da análise

---

## 🚀 Instalação e Execução

### 1. Clone o repositório

```bash
git clone <URL_DO_REPOSITORIO>
cd "teste AB"
```

### 2. Crie um ambiente virtual

```bash
python -m venv .venv
```

Ativação no Windows:

```bash
.venv\Scripts\activate
```

Ativação no Linux/macOS:

```bash
source .venv/bin/activate
```

### 3. Instale as dependências

```bash
pip install -r requirements.txt
```

### 4. Execute o Jupyter Notebook

```bash
jupyter notebook
```

Depois, abra:

```text
notebooks/notebook.ipynb
```

> O notebook utiliza caminhos relativos para os datasets, portanto deve ser executado preservando a estrutura de diretórios apresentada neste README.

---

## 📌 Conclusões

O experimento A/B apresentado no notebook demonstra uma diferença estatisticamente significativa na **taxa de conversão**, enquanto o **tamanho médio dos pedidos não apresenta diferença estatisticamente significativa** após o tratamento dos valores atípicos.

A análise também evidencia a importância da etapa de qualidade dos dados: usuários alocados simultaneamente aos dois grupos e pedidos com valores extremos poderiam distorcer a interpretação dos resultados.

Considerando os critérios definidos no próprio notebook, a recomendação registrada na análise é **implementar a versão B em 100% do tráfego**, mantendo o acompanhamento das métricas após a implementação.

---

## ⚠️ Limitações

O projeto apresenta algumas limitações que devem ser consideradas na interpretação dos resultados:

1. **Período limitado de observação:** o experimento utiliza dados de **01/08/2019 a 31/08/2019**, portanto os resultados representam apenas o período analisado.

2. **Exclusão de usuários entre grupos:** foram identificados e removidos **181 usuários presentes simultaneamente nos grupos A e B**. Essa limpeza reduz a amostra disponível e pode alterar a composição final utilizada nos testes.

3. **Definição de anomalias por percentil:** os limiares adotados — mais de **1 pedido por usuário** e receita superior a **US$ 414,28** — são escolhas analíticas que influenciam os resultados filtrados.

4. **Escopo estatístico do experimento:** a análise utiliza nível de significância de **5%** e os testes definidos no notebook, mas não documenta uma análise prévia de poder estatístico ou um ajuste específico para múltiplas comparações.

---

## 📎 Dados Utilizados

O projeto utiliza três fontes principais:

- `hypotheses_us.csv`
- `orders_us.csv`
- `visits_us.csv`

Os dados são utilizados exclusivamente dentro do fluxo analítico documentado no notebook.

---

## 👤 Projeto de Portfólio

Este projeto demonstra competências em:

- **Análise exploratória de dados (EDA)**
- **Data Cleaning**
- **Otimização de memória em DataFrames**
- **Experimentação A/B**
- **Priorização de hipóteses com ICE e RICE**
- **Análise de métricas de negócio**
- **Detecção de anomalias**
- **Inferência estatística**
- **Visualização de dados**
- **Comunicação de resultados orientada ao negócio**
