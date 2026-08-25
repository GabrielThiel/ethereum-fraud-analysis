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
