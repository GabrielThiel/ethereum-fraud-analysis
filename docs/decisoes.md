# Registro de decisões

## ETFD

### 01 - Remoção de registros duplicados

**Data:** 25/08/2026

**Problema identificado:**  
Foram encontrados 1.533 registros duplicados no dataset ETFD.

**Investigação:**  
Foi verificado se registros com as mesmas features possuíam
classificações diferentes na variável `Fraud`.

**Resultado:**  
Não foram encontrados conflitos de classificação.

**Decisão:**  
Remover os registros duplicados, mantendo apenas uma ocorrência
de cada registro.

**Impacto:**
- Antes: 85.003 registros
- Depois: 83.470 registros
- Removidos: 1.533 registros

**Justificativa:**  
A remoção elimina redundâncias sem introduzir conflitos de
classificação e mantém a distribuição das classes aproximadamente
equilibrada.

## 05 - Avaliação da divisão temporal

**Resultado:**

A divisão temporal de 70% para treinamento e 30% para teste apresentou
forte mudança na distribuição da variável `Fraud`.

O conjunto de treinamento apresentou 70,56% de registros fraudulentos,
enquanto o conjunto de teste apresentou apenas 0,18%.

**Interpretação:**

O resultado evidencia uma forte mudança de distribuição ao longo do
`blockNumber`. Essa característica é consistente com a elevada capacidade
discriminativa observada anteriormente para `blockNumber`.

**Decisão:**

A divisão temporal não será utilizada isoladamente como estratégia
principal de comparação entre os modelos.

Será utilizada como experimento complementar de generalização temporal.

Para a comparação principal dos algoritmos, será utilizada uma divisão
estratificada, preservando a proporção das classes entre os conjuntos de
treinamento e teste.

**Justificativa:**

A divisão temporal de 70/30 produz um conjunto de teste com quantidade
muito reduzida de exemplos fraudulentos, dificultando uma avaliação
estatisticamente representativa da capacidade de classificação da classe
minoritaria nesse cenário específico.

## 02 - Análise de correlação

**Dataset:** ETFD

**Problema/observação:**  
A análise de correlação identificou relações muito fortes entre algumas
features.

**Principais resultados:**
- `blockNumber` × `confirmations`: aproximadamente -0,999992
- `mean_value_received` × `variance_value_received`: aproximadamente 0,976
- `variance_value_received` × `total_received`: aproximadamente 0,912
- `mean_value_received` × `total_received`: aproximadamente 0,896

Também foram observadas correlações elevadas entre algumas features e a
variável `Fraud`, especialmente `blockNumber` e `confirmations`.

**Decisão:**  
Nenhuma feature foi removida nesta etapa.

**Justificativa:**  
Correlação elevada não é suficiente, isoladamente, para determinar que
uma variável deve ser excluída. As variáveis serão analisadas considerando
seu significado, redundância e possível ocorrência de data leakage antes
da seleção definitiva de features.

## 03 - Dependência temporal da variável Fraud

**Dataset:** ETFD

**Observação:**  
Foi analisada a distribuição da variável `Fraud` ao longo das faixas de
`blockNumber`.

**Resultado:**  
Foi identificada uma forte variação temporal na proporção de fraude. As
primeiras faixas apresentaram aproximadamente 90% a 100% de registros
fraudulentos, enquanto as faixas posteriores apresentaram proporções
próximas de 0%.

**Interpretação:**  
Esse comportamento indica uma forte dependência entre o período
representado pelo `blockNumber` e a variável-alvo `Fraud`.

**Decisão:**  
Nenhuma feature foi removida nesta etapa.

**Justificativa:**  
Antes da exclusão de `blockNumber` ou `confirmations`, é necessário
investigar a origem dos dados, o processo de geração e rotulagem do ETFD
e a possibilidade de dependência temporal ou vazamento de informação.

Essa investigação será considerada na definição da estratégia de
divisão dos dados para treinamento e avaliação dos modelos.

## 04 - Análise do poder discriminativo individual das features

**Dataset:** ETFD

**Método:** ROC-AUC individual por feature.

**Principais resultados:**

- `blockNumber`: AUC ajustada = 0,967519
- `confirmations`: AUC = 0,966261
- `total_tx_sent_malicious`: AUC = 0,896753
- `total_tx_sent_malicious_unique`: AUC = 0,892559
- `total_tx_sent`: AUC ajustada = 0,758434
- `time_diff_first_last_received`: AUC ajustada = 0,736448
- `total_received`: AUC ajustada = 0,712851

**Observação:**

`blockNumber` e `confirmations` apresentaram capacidade discriminativa
individual muito elevada. Esse resultado é consistente com a forte
dependência temporal observada anteriormente.

As features relacionadas a transações maliciosas também apresentaram
elevado poder discriminativo individual. Como essas características
possuem relação com a informação utilizada na construção da variável
`Fraud`, elas serão investigadas quanto à possibilidade de vazamento de
informação.

**Decisão:**

Nenhuma feature foi removida nesta etapa.

**Justificativa:**

A análise de AUC foi utilizada como ferramenta exploratória para
identificar características potencialmente relevantes ou problemáticas.
A decisão definitiva sobre seleção ou exclusão será realizada após
comparação de diferentes conjuntos de features e definição adequada da
estratégia de treinamento e avaliação.
