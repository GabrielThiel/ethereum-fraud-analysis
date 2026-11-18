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

## 06 - Feature Engineering das variáveis temporais

Foram analisadas as variáveis temporais `Month`, `Day` e `Hour`.

As variáveis `Month` e `Hour` foram transformadas utilizando representação
cíclica por seno e cosseno, conforme abordagem de Feature Engineering para
variáveis periódicas.

Foram criadas as seguintes features:

- `Month_sin`;
- `Month_cos`;
- `Hour_sin`;
- `Hour_cos`.

A variável `Day` foi mantida originalmente, pois a quantidade de dias varia
entre os meses e não foi assumido um ciclo fixo de 31 dias.

Após a transformação, o Cenário A passou de 14 para 16 features.

---

## 07 - Feature Selection com Mutual Information

Foi utilizado o método `SelectKBest` com `mutual_info_classif` para analisar
a relevância individual das features em relação à variável-alvo `Fraud`.

A seleção foi ajustada exclusivamente sobre os dados de treinamento.

Inicialmente foi realizado um teste exploratório com `k=8`. Em seguida,
foram avaliados os valores:

- k=4;
- k=6;
- k=8;
- k=10;
- k=12;
- k=14;
- k=16.

A escolha foi realizada por validação cruzada estratificada com 5 folds,
utilizando o PR-AUC médio como critério.

O maior PR-AUC médio foi obtido com `k=12`, aproximadamente 0,9837.

Com `k=12`, foram selecionadas:

- `confirmations`;
- `blockNumber`;
- `total_tx_sent_malicious`;
- `total_tx_sent_malicious_unique`;
- `mean_value_received`;
- `total_received`;
- `total_tx_sent`;
- `time_diff_first_last_received`;
- `variance_value_received`;
- `total_tx_sent_unique`;
- `Month_sin`;
- `Month_cos`.

Foram descartadas:

- `total_tx_received_malicious_unique`;
- `Hour_sin`;
- `Hour_cos`;
- `Day`.

A Regressão Logística com `k=12` apresentou PR-AUC de aproximadamente
0,9854, resultado próximo ao PR-AUC de aproximadamente 0,9864 obtido
com todas as features.

A redução do conjunto de atributos provocou, portanto, pequena perda de
desempenho preditivo.

---

## 08 - Comparação entre Regressão Logística e XGBoost

Foram comparados Regressão Logística e XGBoost utilizando o mesmo split
estratificado e quatro conjuntos de features.

### Cenário A - Todas as features

Regressão Logística:
- Precision: 0,9686;
- Recall: 0,9632;
- F1: 0,9659;
- PR-AUC: 0,9864.

XGBoost:
- Precision: 0,9966;
- Recall: 0,9989;
- F1: 0,9978;
- PR-AUC: 0,9999.

### Cenário B - Sem `blockNumber` e `confirmations`

Regressão Logística:
- Precision: 0,7699;
- Recall: 0,8513;
- F1: 0,8086;
- PR-AUC: 0,8921.

XGBoost:
- Precision: 0,9852;
- Recall: 0,9909;
- F1: 0,9880;
- PR-AUC: 0,9991.

### Cenário C - Sem features relacionadas a transações maliciosas

Regressão Logística:
- Precision: 0,9574;
- Recall: 0,9334;
- F1: 0,9452;
- PR-AUC: 0,9844.

XGBoost:
- Precision: 0,9937;
- Recall: 0,9972;
- F1: 0,9955;
- PR-AUC: 0,9993.

### Cenário D - Sem `blockNumber`, `confirmations` e features maliciosas

Regressão Logística:
- Precision: 0,6944;
- Recall: 0,8215;
- F1: 0,7527;
- PR-AUC: 0,8065.

XGBoost:
- Precision: 0,8862;
- Recall: 0,9564;
- F1: 0,9199;
- PR-AUC: 0,9731.

Nos experimentos realizados, o XGBoost apresentou desempenho superior à
Regressão Logística em todos os conjuntos de features avaliados.

Mesmo no cenário mais restritivo, no qual foram removidos `blockNumber`,
`confirmations` e as features relacionadas a transações maliciosas, o
XGBoost manteve PR-AUC de aproximadamente 0,9731.

Os resultados elevados devem, entretanto, ser interpretados considerando
que esta comparação utiliza uma divisão aleatória estratificada dos dados.
A análise temporal realizada anteriormente demonstrou uma forte mudança
na distribuição da variável `Fraud` ao longo do `blockNumber`.

Portanto, esses resultados caracterizam o desempenho dentro da distribuição
do dataset utilizado e ainda não comprovam a capacidade de generalização
para períodos futuros.

## 09 - Avaliação da Árvore de Decisão

A Árvore de Decisão foi utilizada como terceiro modelo supervisionado,
mantendo o mesmo split estratificado e os quatro conjuntos de features
utilizados nos experimentos anteriores.

### Cenário A - Todas as features

- Precision: 0,9960;
- Recall: 0,9978;
- F1: 0,9969;
- PR-AUC: 0,9975.

### Cenário B - Sem `blockNumber` e `confirmations`

- Precision: 0,9850;
- Recall: 0,9876;
- F1: 0,9863;
- PR-AUC: 0,9894.

### Cenário C - Sem features relacionadas a transações maliciosas

- Precision: 0,9923;
- Recall: 0,9950;
- F1: 0,9936;
- PR-AUC: 0,9949.

### Cenário D - Sem `blockNumber`, `confirmations` e features maliciosas

- Precision: 0,9207;
- Recall: 0,9202;
- F1: 0,9204;
- PR-AUC: 0,9476.

A Árvore de Decisão apresentou desempenho elevado nos quatro cenários.

Assim como observado no XGBoost, o modelo manteve desempenho elevado
mesmo após a remoção de `blockNumber` e `confirmations`.

No cenário mais restritivo, houve redução mais expressiva do PR-AUC,
indicando que a remoção simultânea dos dois grupos de features reduz
a capacidade de separação das classes.

Os resultados ainda correspondem a um modelo sem ajuste de
hiperparâmetros e serão comparados com os demais algoritmos antes da
definição dos modelos finais.

## 10 - Comparação preliminar dos modelos

Foram comparados três modelos supervisionados:

- Regressão Logística;
- XGBoost;
- Árvore de Decisão.

Cada algoritmo foi avaliado utilizando os mesmos quatro cenários de
features e o mesmo split estratificado de treinamento e teste.

Considerando o PR-AUC como uma das métricas principais, o XGBoost
apresentou os maiores valores nos quatro cenários avaliados.

PR-AUC por cenário:

| Modelo | A | B | C | D |
|---|---:|---:|---:|---:|
| Regressão Logística | 0.9864 | 0.8921 | 0.9844 | 0.8065 |
| XGBoost | 0.9999 | 0.9991 | 0.9993 | 0.9731 |
| Árvore de Decisão | 0.9975 | 0.9894 | 0.9949 | 0.9476 |

Mesmo no cenário mais restritivo, sem `blockNumber`, `confirmations`
e as features relacionadas a transações maliciosas, XGBoost e Árvore
de Decisão mantiveram elevado desempenho.

Os resultados ainda correspondem a uma divisão aleatória estratificada.
Portanto, não são considerados evidência suficiente de generalização
para períodos futuros, principalmente devido à forte mudança temporal
identificada anteriormente no dataset.

Antes da definição do modelo final, também serão consideradas avaliações
temporais e medidas de custo computacional.


## 11 - Investigação inicial do dataset Vagifa

O dataset Vagifa foi selecionado como segunda fonte de dados
para investigação no TCC.

Na análise inicial, foram identificadas 51 colunas,
incluindo a variável-alvo `FLAG`, o identificador
`Address` e as colunas `Unnamed: 0` e `Index`.

A coluna `Unnamed: 0` apresentou forte associação
com a variável-alvo:

- FLAG = 0: índices entre 0 e 7661;
- FLAG = 1: índices entre 7662 e 9840.

Essa organização representa risco de target leakage,
pois a posição dos registros pode permitir reproduzir
os rótulos sem analisar o comportamento transacional.

A coluna `Index` também apresentou associação com
a classe, exigindo investigação adicional.

Decisão preliminar:
- Excluir `Unnamed: 0` das features preditivas;
- Excluir `Index` das features preditivas;
- Manter `Address` apenas para verificações de
  identidade e integridade, sem utilizá-lo como
  entrada direta dos modelos.

  
## 12 - Associação entre valores ausentes e o rótulo no Vagifa

A auditoria identificou 25 colunas com valores ausentes e
2.720 registros contendo pelo menos um valor nulo.

A análise por quantidade de valores ausentes revelou:

| Nulos por registro | FLAG 0 | FLAG 1 |
|---|---:|---:|
| 0 | 5771 | 1350 |
| 1 | 1872 | 0 |
| 2 | 19 | 0 |
| 25 | 0 | 829 |

Todos os 829 registros com 25 valores ausentes pertencem à
classe fraudulenta. Já todos os registros com um ou dois
valores ausentes pertencem à classe não fraudulenta.

Essa associação indica que o padrão de ausência de
informações pode funcionar como um forte indicador da
variável-alvo.

Ainda não foi estabelecido se esse comportamento resulta
de características legítimas das contas ou do processo de
construção e rotulação do dataset.

Decisão:
- Preservar o dataset original.
- Não realizar imputação ou remoção automática dos nulos.
- Investigar quais atributos estão ausentes em cada grupo.
- Avaliar possíveis efeitos da ausência de dados na modelagem.
- Não interpretar métricas elevadas como prova de detecção
  real de fraude sem investigar esses padrões.
```

Esse registro será importante para justificar à banca que estamos auditando a qualidade e a origem das features antes de treinar os modelos no segundo dataset.

**Execute a célula de investigação e me envie o resultado.** Depois vamos definir o tratamento dos nulos, resolver os endereços duplicados e preparar a versão limpa do Vagifa.

A decisão será confirmada após a conclusão da
auditoria dos identificadores.


## 13 - Padrões de ausência de dados ERC-20 no Vagifa

A investigação dos valores ausentes identificou que os
25 atributos ERC-20 estão simultaneamente ausentes
em 829 registros, todos classificados como fraudulentos.

Outros 1.891 registros não fraudulentos apresentam
ausência de um ou dois atributos relacionados aos
nomes dos tokens ERC-20.

Os 7.121 registros restantes não apresentam valores ausentes.

Esses padrões demonstram forte associação entre a
ausência de informações ERC-20 e a variável FLAG.

Entretanto, ainda não foi comprovado que essa
associação representa target leakage.

Decisões:
- Preservar o dataset original.
- Não substituir automaticamente valores ausentes por zero.
- Não excluir os registros fraudulentos com dados ERC-20 ausentes.
- Investigar a origem dos valores ausentes.
- Preparar um cenário experimental sem atributos ERC-20.
- Comparar posteriormente com um cenário que utilize
  os atributos ERC-20 após tratamento apropriado.
```

**Execute agora a investigação dos endereços repetidos e me envie o resultado.** Depois vamos decidir o tratamento das duplicatas e preparar o primeiro DataFrame limpo do Vagifa, preservando todas essas descobertas para a versão revisada do TCC.

## 14 - Auditoria dos endereços duplicados no Vagifa

Foram identificados 25 endereços Ethereum repetidos, correspondentes a 50 registros.

A comparação dos atributos demonstrou que:

- Nenhum endereço apresentou diferenças nas features numéricas.
- Sete endereços apresentaram diferenças nos nomes de tokens ERC-20.
- Dezoito endereços apresentaram atributos idênticos, desconsiderando os índices artificiais.
- Nenhum endereço apresentou conflito na variável-alvo `FLAG`.

**Decisões metodológicas:**

1. Preservar o dataset original para fins de auditoria e rastreabilidade.
2. Normalizar os endereços Ethereum antes de identificar duplicações.
3. Manter apenas um registro por endereço na versão de modelagem, considerando a equivalência dos atributos numéricos e dos rótulos.
4. Excluir os identificadores artificiais `Unnamed: 0` e `Index`, devido ao risco de vazamento de informação.
5. Excluir inicialmente os nomes categóricos de tokens da versão de modelagem, mantendo-os disponíveis no dataset original.
6. Criar um cenário sem atributos ERC-20 para investigar o efeito dos padrões de valores ausentes sobre o desempenho preditivo.

A preparação não elimina a necessidade de investigar outras possíveis relações entre os atributos e o processo de rotulação.

## 15 - Preparação e auditoria numérica do dataset Vagifa

Após a investigação dos identificadores, endereços duplicados e valores ausentes, foi criada uma versão preparada do dataset Vagifa.

**Características da base preparada:**

- Dataset original: 9.841 registros e 51 colunas.
- Dataset preparado: 9.816 registros e 47 colunas.
- Primeiro cenário sem ERC-20: 9.816 registros e 22 features de entrada.
- Classe `FLAG = 0`: 7.637 registros (77,8%).
- Classe `FLAG = 1`: 2.179 registros (22,2%).
- Valores ausentes no cenário sem ERC-20: 0.
- Endereços duplicados após a preparação: 0.

Foram removidos da entrada dos modelos os índices artificiais e os identificadores de contas. Os atributos ERC-20 foram excluídos do primeiro cenário devido aos padrões de ausência associados à classe, mas permanecem disponíveis na base original para experimentos posteriores.

### Auditoria das features numéricas

Foram analisadas as 22 features numéricas, considerando tipos, valores distintos, mínimos, máximos, valores negativos, infinitos e frequência de zeros por classe.

Não foram identificados valores infinitos ou ausentes nas entradas do primeiro cenário.

Foram identificados 976 valores negativos, todos associados à variável `total ether balance`.

A análise indicou que essa variável corresponde aproximadamente à diferença entre o total de Ether recebido e enviado.

Os saldos negativos ocorreram em 10,84% dos registros não fraudulentos e 6,79% dos registros fraudulentos.

Apenas dois registros apresentaram diferença superior a 0,01 ETH entre o saldo registrado e o saldo calculado. Essas divergências permanecem em investigação.

**Decisão:** preservar os valores negativos e não realizar exclusões automáticas, pois o atributo parece representar um saldo líquido calculado. Sua definição precisa será considerada na documentação das features.

Também foram observadas diferenças relevantes na frequência de valores zero entre as classes, que serão examinadas posteriormente para investigar associações com o rótulo e possíveis atalhos de classificação.

### Revisão da fórmula de `total ether balance`

A investigação identificou que a variável `total ether balance` apresenta uma relação matemática com os valores de Ether recebidos, enviados e enviados para contratos.

Foi avaliada a seguinte expressão:

`total ether balance = total ether received - total Ether sent - total ether sent contracts`

A inclusão dos envios para contratos explicou as duas maiores divergências identificadas anteriormente, correspondentes aos registros 682 e 6752.

Com a fórmula revisada, a maior diferença absoluta observada foi de aproximadamente 0,001074 ETH, e nenhum registro apresentou diferença superior a 0,01 ETH.

Entretanto, 129 registros ainda apresentaram diferença superior a 0,000001 ETH, exigindo investigação complementar da precisão dos valores.

### Conclusão da auditoria do saldo Ethereum

A análise confirmou que `total ether balance` apresenta uma relação matemática aproximada com três atributos:

`total ether balance ≈ total ether received - total Ether sent - total ether sent contracts`

Após considerar os envios para contratos, a maior diferença absoluta encontrada foi de aproximadamente 0,001734 ETH.

Dos 9.816 registros analisados:

- 129 apresentaram diferença superior a 0,000001 ETH.
- 37 apresentaram diferença superior a 0,00001 ETH.
- 8 apresentaram diferença superior a 0,0001 ETH.
- 2 apresentaram diferença superior a 0,001 ETH.
- Nenhum apresentou diferença superior a 0,01 ETH.

A diferença mediana foi praticamente zero.

**Decisão metodológica:** preservar os registros e considerar `total ether balance` uma variável aproximadamente derivada de outros atributos. A possível redundância será examinada nos experimentos de seleção e remoção de features.

Não foram identificadas evidências suficientes para determinar a causa exata das pequenas diferenças residuais.

**Decisão:** manter os registros e preservar os valores originais. A variável será tratada como aproximadamente derivada de outros atributos, considerando essa relação na análise de redundância e seleção de features.

## 16 - Estratégia de particionamento do Vagifa

Após a preparação do segundo dataset, foram obtidos 9.816 registros únicos de contas Ethereum, com 22 features numéricas no cenário inicial sem ERC-20.

Foi realizado um particionamento aleatório estratificado, utilizando `random_state=42`, com a seguinte distribuição:

| Conjunto | Registros | FLAG 0 | FLAG 1 |
|---|---:|---:|---:|
| Treinamento | 5.889 | 4.582 | 1.307 |
| Validação | 1.963 | 1.527 | 436 |
| Teste | 1.964 | 1.528 | 436 |

A proporção de contas rotuladas como fraudulentas permaneceu próxima de 22,2% nos três conjuntos.

**Decisões metodológicas:**

- Utilizar o treinamento para ajustar transformações e modelos.
- Utilizar a validação para comparar cenários, configurações e thresholds.
- Preservar o teste para a avaliação final.
- Não utilizar os índices artificiais nem o endereço Ethereum como features preditivas.
- Investigar o efeito da remoção de atributos derivados, extremamente esparsos e de grupos específicos de características.
- Reconhecer que o particionamento aleatório não demonstra generalização temporal ou entre datasets.

A comparação inicial será realizada com Regressão Logística e XGBoost, priorizando Precision, Recall, F1 e métricas de curva Precision-Recall.

A ausência de uma referência cronológica adequada no Vagifa será documentada como limitação experimental.


## 17 - Avaliação inicial da Regressão Logística no Vagifa

Foram avaliados quatro cenários de features utilizando
Regressão Logística com StandardScaler.

Os experimentos utilizaram 5.889 registros de treinamento
e 1.963 registros de validação.

Com o limiar padrão de 0,5, os modelos não identificaram
nenhum registro fraudulento na validação.

O Cenário A apresentou um falso positivo, enquanto os
cenários B, C e D classificaram todos os registros como
não fraudulentos.

O Cenário D apresentou o maior valor de Average Precision
(AP = 0,6099), apesar de Recall e F1 iguais a zero no
limiar padrão.

Esse resultado demonstra que a acurácia de aproximadamente
77,8% não representa capacidade satisfatória de identificação
de fraudes, pois corresponde praticamente à frequência da
classe majoritária.

Decisão: investigar as distribuições das probabilidades,
os thresholds e o efeito do pré-processamento antes de
avançar para outros algoritmos.

## 18 - Investigação dos thresholds da Regressão Logística

A avaliação inicial da Regressão Logística no Vagifa apresentou Recall igual a zero com o threshold padrão de 0,5 nos quatro cenários.

A análise das probabilidades revelou que a maioria dos registros, inclusive fraudulentos, recebeu probabilidades inferiores a 0,5.

Foram avaliados thresholds de 0,05 a 0,50 exclusivamente no conjunto de validação.

Entre os thresholds avaliados, o maior F1 foi aproximadamente 0,5131, obtido no Cenário B com threshold de 0,30.

Nesse cenário, o Recall alcançou aproximadamente 0,8096, enquanto a Precision foi de 0,3755.

Os resultados mostram que a alteração do threshold melhora a identificação da classe fraudulenta, mas ainda produz quantidade elevada de falsos positivos.

**Decisão:** investigar o uso de pesos de classe durante o treinamento antes de definir um threshold ou selecionar a configuração final.

O conjunto de teste permanece reservado para a avaliação final.
O conjunto de teste permanece reservado.

## 19 - Regressão Logística com pesos de classe no Vagifa

Foi avaliada uma segunda configuração da Regressão Logística utilizando `class_weight="balanced"`, mantendo os mesmos quatro cenários de features e os conjuntos de treinamento e validação.

O objetivo foi investigar o efeito do desbalanceamento, uma vez que aproximadamente 22,2% dos registros pertencem à classe fraudulenta.

### Resultados na validação

| Cenário | Precision | Recall | F1 | AP |
|---|---:|---:|---:|---:|
| A - Completo | 0,3560 | 0,8509 | 0,5020 | 0,4849 |
| B - Sem saldo | 0,3557 | 0,8509 | 0,5017 | 0,4865 |
| C - Sem contratos | 0,3560 | 0,8509 | 0,5020 | 0,4891 |
| D - Atividade e relacionamentos | 0,2852 | 0,9427 | 0,4379 | 0,6094 |

O uso de pesos de classe elevou o Recall e o F1 no threshold padrão de 0,5 em comparação com os modelos não ponderados.

Entretanto, os modelos continuaram apresentando baixa Precision, especialmente no Cenário D, que produziu 1.030 falsos positivos.

A Average Precision permaneceu próxima dos valores encontrados antes do balanceamento. No Cenário D, passou de aproximadamente 0,6099 para 0,6094.

### Conclusões

- O balanceamento por pesos favoreceu a identificação da classe minoritária no threshold padrão.
- O aumento do Recall ocorreu acompanhado de muitos falsos positivos.
- Não foi identificada melhoria relevante na Average Precision.
- O Cenário D apresentou a maior AP, apesar de utilizar somente seis atributos.
- A acurácia isolada não é adequada para avaliar a capacidade de identificação das fraudes.

**Decisão:** manter ambas as configurações como experimentos preliminares e investigar o comportamento de um modelo não linear, preservando o conjunto de teste para avaliação final.

## 20 - Avaliação inicial do XGBoost no dataset Vagifa

Foi realizada a comparação do algoritmo XGBoost nos quatro cenários de features, utilizando 5.889 registros para treinamento e 1.963 para validação.

O conjunto de teste permaneceu reservado.

| Cenário | Features | Precision | Recall | F1 | AP |
|---|---:|---:|---:|---:|---:|
| A - Completo | 22 | 0,9127 | 0,7913 | 0,8477 | 0,9430 |
| B - Sem saldo | 21 | 0,9122 | 0,7867 | 0,8448 | 0,9396 |
| C - Sem contratos | 18 | 0,9110 | 0,7982 | 0,8509 | 0,9430 |
| D - Atividade e relacionamentos | 6 | 0,9207 | 0,6124 | 0,7355 | 0,8458 |

O XGBoost apresentou desempenho superior à Regressão Logística nas configurações avaliadas.

O Cenário C manteve desempenho praticamente equivalente ao cenário completo, apesar da remoção das quatro features extremamente esparsas relacionadas a valores enviados para contratos.

No Cenário C, foram identificadas corretamente 348 das 436 contas fraudulentas presentes na validação, com 34 falsos positivos e 88 falsos negativos.

O Cenário D, contendo apenas seis características de atividade e relacionamentos, apresentou menor Recall e Average Precision, indicando que os atributos de valores transacionais contribuem para o desempenho do modelo.

### Interpretação e limitações

Os resultados são preliminares e correspondem ao conjunto de validação de uma divisão aleatória estratificada.

As métricas elevadas não constituem evidência suficiente de generalização para contas Ethereum futuras ou bases independentes.

**Decisões:**

- Manter o XGBoost como candidato para avaliação posterior.
- Priorizar o Cenário C como configuração promissora, sem defini-lo ainda como modelo final.
- Investigar possíveis perfis transacionais idênticos entre treinamento e validação.
- Continuar a auditoria da origem dos rótulos e das features.
- Preservar o conjunto de teste para a avaliação final.

## 21 - Investigação de perfis repetidos entre treinamento e validação

Foi realizada uma auditoria para identificar perfis transacionais numericamente idênticos entre os conjuntos de treinamento e validação do Vagifa.

Embora os endereços Ethereum duplicados tenham sido removidos durante a limpeza, observou-se que diferentes contas podem apresentar exatamente os mesmos valores de features.

| Cenário | Duplicatas internas no treino | Registros de validação com perfil já visto |
|---|---:|---:|
| A - Completo | 330 | 115 (5,86%) |
| C - Sem contratos | 330 | 115 (5,86%) |
| D - Atividade e relacionamentos | 3.090 | 1.141 (58,13%) |

A redução da quantidade de atributos aumentou significativamente a frequência de perfis idênticos, especialmente no Cenário D.

Essa constatação não comprova vazamento de informação, mas indica que o particionamento aleatório pode permitir que modelos encontrem na validação combinações de atributos já observadas durante o treinamento.

**Decisões metodológicas:**

- Investigar conflitos de rótulos entre perfis idênticos.
- Comparar posteriormente o desempenho em perfis já vistos e novos.
- Considerar estratégias alternativas de particionamento por perfil.
- Manter o conjunto de teste reservado.
- Não interpretar os resultados elevados de AP como comprovação de generalização para novas contas ou períodos.

  ## 22 - Conflitos de rótulos em perfis transacionais do Vagifa

Foi realizada uma auditoria de perfis numéricos idênticos entre os conjuntos de treinamento e validação.

| Indicador | A - Completo | C - Sem contratos | D - Atividade e relacionamentos |
|---|---:|---:|---:|
| Perfis únicos no treinamento | 5.559 | 5.559 | 2.799 |
| Perfis presentes nas duas classes no treinamento | 3 | 3 | 84 |
| Registros de validação com perfis já vistos | 115 | 115 | 1.141 |
| Registros com rótulos incompatíveis com os observados no treinamento para o mesmo perfil | 1 | 1 | 26 |
| Registros de validação com perfis novos | 1.848 | 1.848 | 822 |

A análise demonstrou que contas Ethereum distintas podem apresentar características numéricas idênticas, inclusive quando pertencem a classes diferentes.

A redução das features aumenta significativamente a quantidade de perfis repetidos, principalmente no cenário que utiliza somente seis atributos.

Essas ocorrências não comprovam vazamento de informação, mas evidenciam possíveis ambiguidades na representação dos registros e limitações do particionamento aleatório.

**Decisões metodológicas:**

- Investigar separadamente o desempenho em perfis novos e já vistos.
- Considerar posteriormente uma estratégia de particionamento que mantenha perfis idênticos no mesmo conjunto.
- Não remover automaticamente perfis com rótulos diferentes.
- Preservar o conjunto de teste para avaliação final.
- Investigar a origem e os critérios de atribuição dos rótulos antes de concluir que os modelos aprenderam comportamentos fraudulentos generalizáveis.

## 23 - Desempenho do XGBoost em perfis novos e repetidos

Foi realizada uma avaliação complementar dos modelos XGBoost, separando registros de validação cujas combinações de features já haviam aparecido no treinamento daqueles com perfis numéricos novos.

### Cenário C — Sem valores enviados a contratos

| Indicador | Perfis já vistos | Perfis novos |
|---|---:|---:|
| Registros | 115 | 1.848 |
| Fraudes | 109 | 327 |
| Prevalência de fraude | 94,78% | 17,69% |
| Average Precision | 0,9986 | 0,9140 |
| F1-score | 0,9636 | 0,8094 |

A análise identificou uma forte concentração de registros fraudulentos no grupo de perfis já vistos.

A AP próxima de 1,0 nesse grupo exige cautela, pois a prevalência de fraude já é de 94,78%.

Nos perfis novos, o XGBoost manteve AP de aproximadamente 0,9140, superior à prevalência de fraude de 17,69%.

Esse resultado indica capacidade discriminativa em combinações de atributos não observadas exatamente no treinamento. Contudo, não comprova generalização temporal nem independência em relação ao processo de construção e rotulação do dataset.

**Decisões metodológicas:**

- Investigar quais perfis concentram os registros fraudulentos repetidos.
- Realizar validação agrupada por perfil, evitando compartilhamento de combinações numéricas idênticas entre treino e validação.
- Preservar o conjunto de teste sem avaliação nesta etapa.
- Não utilizar AP elevada isoladamente como comprovação de detecção de fraude.
- Continuar investigando a origem dos rótulos `FLAG` e dos atributos do Vagifa.
  
## 24 - Investigação dos perfis transacionais totalmente zerados no Vagifa

A auditoria do Cenário C identificou um grupo de 427 registros com todas as 18 features numéricas iguais a zero, considerando treinamento e validação.

Todos os 427 registros estão rotulados como fraudulentos (`FLAG = 1`).

| Característica | Treinamento | Validação |
|---|---:|---:|
| Perfis totalmente zerados | 322 | 105 |
| Fraudes entre os perfis zerados | 322 | 105 |
| Perfis zerados com ERC-20 ausente | 157 | 59 |
| Perfis zerados com ERC-20 presente | 165 | 46 |

A investigação demonstrou que o perfil totalmente zerado não coincide integralmente com o padrão de ausência dos atributos ERC-20.

Na validação, o grupo de perfis zerados representa aproximadamente 24,1% dos registros fraudulentos.

Essa associação perfeita entre o perfil zerado e a classe fraudulenta, nos conjuntos analisados, pode representar um atalho de classificação. Entretanto, sua origem ainda não foi determinada.

**Decisões metodológicas:**

- Preservar os dados originais e o rótulo das contas.
- Avaliar separadamente o desempenho do XGBoost nos registros com atividade transacional diferente de zero.
- Investigar o efeito de excluir esses perfis do treinamento.
- Não considerar a associação observada como comprovação automática de target leakage.
- Manter o conjunto de teste reservado.
- Continuar investigando a origem dos rótulos e os procedimentos de construção do dataset.

  ## 25 - Sensibilidade do XGBoost aos perfis totalmente zerados

Foi avaliada a influência dos perfis com todas as 18 features zeradas no desempenho do XGBoost, utilizando o Cenário C do dataset Vagifa.

A comparação foi realizada entre a validação completa e a validação sem os 105 registros com perfil totalmente zerado, todos classificados como fraudulentos.

| Métrica | Validação completa | Sem perfis zerados |
|---|---:|---:|
| Registros | 1.963 | 1.858 |
| Fraudes | 436 | 331 |
| Prevalência de fraude | 0,2221 | 0,1781 |
| Average Precision | 0,9430 | 0,9038 |
| Precision | 0,9110 | 0,8773 |
| Recall | 0,7982 | 0,7341 |
| F1-score | 0,8509 | 0,7993 |

Os 105 registros removidos correspondiam a fraudes corretamente classificadas pelo modelo original.

A exclusão desses registros reduziu a AP, o Recall e o F1. Entretanto, também alterou a prevalência da classe fraudulenta, de 22,21% para 17,81%.

O modelo manteve AP de 0,9038 nos registros não zerados, indicando capacidade discriminativa na amostra avaliada.

**Limitações:** o experimento utilizou um modelo originalmente treinado com os 322 perfis totalmente zerados. Portanto, não permite isolar completamente o efeito desses registros durante o aprendizado.

**Próxima decisão experimental:** treinar outro XGBoost excluindo os perfis zerados do treinamento e comparar os dois modelos na mesma validação filtrada.

O conjunto de teste permanece reservado.

## 26 - Retreinamento do XGBoost sem perfis zerados

Foi realizado um experimento de sensibilidade com o XGBoost no Cenário C do Vagifa.

Foram excluídos 322 registros com todas as 18 features zeradas do conjunto de treinamento. Os dois modelos foram avaliados nos mesmos 1.858 registros de validação com pelo menos uma feature diferente de zero.

| Métrica | Treinamento original | Treinamento sem perfis zerados |
|---|---:|---:|
| Registros de treinamento | 5.889 | 5.567 |
| AP | 0,9038 | 0,9117 |
| Precision | 0,8773 | 0,8885 |
| Recall | 0,7341 | 0,7462 |
| F1 | 0,7993 | 0,8112 |
| Falsos positivos | 34 | 31 |
| Falsos negativos | 88 | 84 |

A remoção dos perfis zerados do treinamento não prejudicou o desempenho nos registros com atividade transacional. Foram observadas pequenas melhorias nas métricas avaliadas.

Entretanto, a exclusão também alterou a prevalência da classe positiva no treinamento, impossibilitando atribuir toda a diferença exclusivamente à remoção dos perfis.

**Conclusão:** os perfis totalmente zerados não são indispensáveis para o desempenho observado nos registros não zerados, mas a generalização independente ainda não foi demonstrada.

**Próxima etapa:** comparar validação cruzada convencional e validação cruzada agrupada por perfil numérico, mantendo o conjunto de teste reservado.

## 27 - Validação cruzada agrupada por perfil

Foi realizada uma comparação entre validação cruzada estratificada convencional e validação cruzada estratificada agrupada por perfil numérico.

O experimento utilizou exclusivamente 5.567 registros não zerados do treinamento do Vagifa, considerando as 18 features do Cenário C.

Foram utilizados cinco folds e a mesma configuração do XGBoost.

| Métrica | Estratificada | Agrupada por perfil |
|---|---:|---:|
| AP média | 0,9176 | 0,9166 |
| Desvio padrão da AP | 0,0176 | 0,0205 |
| F1 médio | 0,8045 | 0,8099 |
| Recall médio | 0,7452 | 0,7469 |
| Precision média | 0,8757 | 0,8845 |
| Grupos compartilhados entre folds | 10 | 0 |

A estratégia agrupada impediu a presença de perfis numéricos idênticos em ambos os lados de cada divisão.

A diferença de Average Precision média entre as estratégias foi pequena, aproximadamente 0,0010.

**Conclusão:** neste experimento, a repetição exata de perfis não explicou isoladamente o desempenho elevado do XGBoost.

O resultado não comprova generalização temporal, independência entre contas semelhantes ou ausência de viés na construção do dataset.

**Próxima etapa:** investigar a proveniência dos rótulos do Vagifa, comparando os endereços e classificações com a base publicada por Farrugia, Ellul e Azzopardi (2020).

Referência:

FARRUGIA, S.; ELLUL, J.; AZZOPARDI, G. Detection of illicit accounts over the Ethereum blockchain. *Expert Systems with Applications*, v. 150, artigo 113318, 2020. DOI: 10.1016/j.eswa.2020.113318.

## 28 - Investigação da proveniência dos rótulos do Vagifa

Foi realizada uma comparação entre os endereços Ethereum do Vagifa e a base `Complete.csv` publicada por Farrugia, Ellul e Azzopardi (2020).

A análise utilizou exclusivamente os conjuntos de treinamento e validação, totalizando 7.852 registros.

| Classe | Encontrados na origem | Não encontrados |
|---|---:|---:|
| FLAG = 0 | 1.628 | 4.481 |
| FLAG = 1 | 1.743 | 0 |

Todos os 1.743 endereços fraudulentos analisados foram encontrados na base original, enquanto apenas 26,6% dos endereços não fraudulentos apresentaram correspondência.

A base original contém 4.681 registros, sendo 2.179 rotulados como ilícitos. O arquivo Kaggle Vagifa contém 9.841 registros, também com 2.179 rotulados como fraudulentos.

A documentação do projeto Vagif12 descreve a coleta de contas fraudulentas a partir do EtherScamDB e de uma lista adicional, com extração de atributos pela API do Etherscan.

**Interpretação:** os resultados indicam forte sobreposição dos endereços fraudulentos entre as fontes, mas a composição das contas não fraudulentas apresenta diferenças substanciais.

Foi identificado um endereço com rótulos conflitantes no arquivo original, ainda em investigação.

**Decisão:** investigar o conflito de rotulação e documentar a possível influência dos diferentes procedimentos de seleção das classes. Não considerar os datasets automaticamente independentes nem interpretar resultados elevados como comprovação de generalização.

### Referências

FARRUGIA, S.; ELLUL, J.; AZZOPARDI, G. Detection of illicit accounts over the Ethereum blockchain. *Expert Systems with Applications*, v. 150, 113318, 2020. DOI: 10.1016/j.eswa.2020.113318.

VAGIF12. *Ethereum Fraud Detection*. Disponível em: https://github.com/Vagif12/Ethereum-Fraud-Detection.

## 29 - Auditoria de conflitos de rotulação na base original

A investigação da base publicada por Farrugia, Ellul e Azzopardi (2020) identificou um endereço Ethereum associado simultaneamente aos rótulos FLAG = 0 e FLAG = 1.

O endereço conflitante não foi encontrado nos conjuntos de treinamento e validação do Vagifa.

Dessa forma, não foi identificado impacto direto desse conflito nos experimentos realizados até o momento.

**Decisões metodológicas:**

- Preservar os dados e os rótulos originais.
- Documentar o conflito como uma limitação da qualidade da fonte original.
- Não alterar os conjuntos de treinamento e validação.
- Manter o conjunto de teste reservado.
- Investigar possíveis vieses associados às diferenças de composição entre a base de Farrugia e o Vagifa.

A correspondência entre endereços e rótulos não garante que as fontes sejam independentes ou que seus processos de seleção estejam livres de vieses.

## 30 - Investigação de possível viés de seleção no Vagifa

Foi avaliado o comportamento do XGBoost no Cenário C, considerando a proveniência das contas não fraudulentas presentes no conjunto de validação.

| Indicador | Encontrado em Farrugia | Não encontrado em Farrugia |
|---|---:|---:|
| Registros | 368 | 1.159 |
| Probabilidade média | 0,0484 | 0,0697 |
| Probabilidade mediana | 0,0123 | 0,0129 |
| Falsos positivos | 3 | 31 |
| Taxa de falsos positivos | 0,8152% | 2,6747% |

A taxa de falsos positivos foi aproximadamente 3,3 vezes maior entre as contas não encontradas na base de Farrugia.

Essa diferença indica que o desempenho do modelo varia entre os grupos de proveniência. Entretanto, não comprova uma relação causal entre a origem dos registros e o desempenho.

**Decisão metodológica:** reconhecer a possibilidade de viés de seleção como uma limitação do dataset Vagifa e encerrar a etapa exploratória para avançar à avaliação final dos modelos.

## 31 - Seleção das configurações e thresholds finais do Vagifa

Foram selecionadas três configurações candidatas para a avaliação final do dataset Vagifa.

Os thresholds foram definidos exclusivamente no conjunto de validação, utilizando a maximização do F1-Score como critério.

| Modelo | Features | Threshold | Precision | Recall | F1 | AP |
|---|---:|---:|---:|---:|---:|---:|
| Regressão Logística | 6 | 0,2734 | 0,5556 | 0,5963 | 0,5752 | 0,6099 |
| Regressão Logística balanceada | 6 | 0,5649 | 0,5597 | 0,5917 | 0,5753 | 0,6094 |
| XGBoost | 18 | 0,3950 | 0,8806 | 0,8624 | 0,8714 | 0,9430 |

O ajuste do threshold do XGBoost permitiu aumentar o Recall de 0,7982 para 0,8624 e o F1 de 0,8509 para 0,8714, acompanhado de uma redução na Precision.

A Average Precision permaneceu inalterada, uma vez que a modificação do limiar não altera a ordenação das probabilidades produzidas pelo modelo.

**Decisões metodológicas:**

- Manter os modelos previamente treinados.
- Preservar os thresholds selecionados na validação com sua precisão numérica original.
- Utilizar o conjunto de teste exclusivamente para avaliação final.
- Não utilizar acurácia como critério de comparação.
- Não realizar novos ajustes com base nos resultados do teste.

A comparação final representa configurações selecionadas individualmente para cada algoritmo, incluindo diferentes conjuntos de features.

Os resultados não constituem, isoladamente, comprovação de generalização temporal ou entre datasets independentes.

## 32 - Avaliação final dos modelos no dataset Vagifa

Após a definição das configurações e dos thresholds exclusivamente no conjunto de validação, foi realizada a avaliação final utilizando os 1.964 registros reservados para teste.

O conjunto de teste contém 1.528 contas não fraudulentas e 436 contas fraudulentas, correspondendo a uma prevalência de fraude de aproximadamente 22,20%.

| Modelo | Features | Precision | Recall | F1 | AP | PR-AUC |
|---|---:|---:|---:|---:|---:|---:|
| Regressão Logística | 6 | 0,5603 | 0,6284 | 0,5924 | 0,6185 | 0,6040 |
| Regressão Logística balanceada | 6 | 0,5625 | 0,6193 | 0,5895 | 0,6257 | 0,6120 |
| XGBoost | 18 | 0,8485 | 0,8349 | 0,8416 | 0,9393 | 0,9393 |

O XGBoost apresentou o melhor desempenho preditivo entre as configurações avaliadas.

Sua matriz de confusão apresentou:

- Verdadeiros negativos: 1.463;
- Falsos positivos: 65;
- Falsos negativos: 72;
- Verdadeiros positivos: 364.

A Average Precision permaneceu próxima do resultado obtido na validação, passando de 0,9430 para 0,9393.

### Limitações

Os resultados correspondem a uma divisão aleatória estratificada do dataset Vagifa.

Apesar das auditorias de identificadores, valores ausentes, perfis repetidos, perfis zerados e proveniência dos rótulos, não é possível garantir ausência de vieses ou generalização para períodos futuros.

Os modelos também utilizam diferentes conjuntos de features, selecionados durante a validação.

### Decisão

Encerrar os ajustes dos modelos do Vagifa, preservando os resultados obtidos no conjunto de teste.

Os modelos serão posteriormente submetidos à avaliação de latência e throughput, permitindo comparar o desempenho preditivo com o custo computacional de inferência.

## 33 - Avaliação do desempenho computacional no Vagifa

Após a avaliação preditiva final, foram realizadas medições de latência e throughput das três configurações selecionadas para o dataset Vagifa.

### Ambiente experimental

- Plataforma: Google Colab.
- Sistema operacional: Linux.
- Arquitetura: x86_64.
- Processador: Intel Xeon CPU @ 2.20 GHz.
- CPUs lógicas: 2.
- Python: 3.13.18.
- Scikit-learn: 1.6.1.
- XGBoost: 3.4.1.
- NumPy: 2.4.6.
- Pandas: 2.2.3.

### Procedimento experimental

Foram selecionados 500 registros para medições individuais de latência, utilizando semente aleatória fixa.

Para a avaliação de throughput, foram realizadas 30 repetições de inferência em lotes contendo os 1.964 registros do conjunto de teste.

Foram executadas inferências de aquecimento antes das medições.

O tempo medido incluiu o pré-processamento interno dos modelos, a geração das probabilidades e a aplicação dos thresholds previamente selecionados.

### Resultados

| Modelo | Latência média (ms) | p50 (ms) | p95 (ms) | p99 (ms) | Throughput (registros/s) |
|---|---:|---:|---:|---:|---:|
| Regressão Logística | 5,4829 | 2,1771 | 19,4063 | 27,8510 | 517225,2058 |
| Regressão Logística balanceada | 2,5156 | 2,2083 | 4,9052 | 8,2243 | 864293,4458 |
| XGBoost | 8,3750 | 4,0374 | 18,0546 | 23,6608 | 252088,2724 |

O XGBoost apresentou maior latência média e menor throughput, mas obteve desempenho preditivo superior, com AP de 0,9393 e F1 de 0,8416.

### Limitações

As medições foram realizadas em um ambiente computacional compartilhado, sujeito a variações de desempenho.

As diferenças observadas entre as configurações da Regressão Logística não permitem concluir que o uso de pesos de classe proporciona ganho de velocidade na inferência.

Os modelos utilizam diferentes quantidades de features, de modo que a comparação representa o custo das configurações experimentais completas.

Os tempos não incluem coleta de transações, comunicação com a blockchain ou extração original de atributos.

### Decisão

Encerrar a avaliação experimental do Vagifa, preservando os resultados preditivos e computacionais.

O XGBoost apresentou o melhor desempenho preditivo entre os modelos avaliados, acompanhado de um custo computacional superior nas condições experimentais utilizadas.

Os resultados serão consolidados com os experimentos do ETFD na discussão final do trabalho.

## 34 - Avaliação temporal dos modelos no ETFD

Foi realizada uma avaliação cronológica complementar utilizando o atributo `blockNumber` como referência de ordenação.

O treinamento utilizou os primeiros 70% dos registros, enquanto o teste utilizou os 30% posteriores.

### Distribuição das classes

| Conjunto | Não fraude | Fraude | Total |
|---|---:|---:|---:|
| Treinamento | 17.204 | 41.224 | 58.428 |
| Teste | 24.996 | 46 | 25.042 |

A prevalência de fraude no conjunto de teste foi de aproximadamente 0,184%, demonstrando uma forte alteração temporal na distribuição das classes.

Foram avaliadas Regressão Logística, XGBoost e Árvore de Decisão nos quatro cenários de features, mantendo os hiperparâmetros anteriores e o threshold de 0,5.

### Resultados de Average Precision

| Modelo | Cenário A | Cenário B | Cenário C | Cenário D |
|---|---:|---:|---:|---:|
| Regressão Logística | 0,046722 | 0,002901 | 0,048494 | 0,002837 |
| XGBoost | 0,312258 | 0,437440 | 0,289699 | 0,325636 |
| Árvore de Decisão | 0,007850 | 0,028421 | 0,006045 | 0,007055 |

O XGBoost apresentou os maiores valores de AP nos quatro cenários temporais.

Entretanto, foram observadas quedas expressivas nas métricas em relação à avaliação aleatória estratificada.

No Cenário B, o XGBoost identificou 45 das 46 fraudes, mas apresentou 1.330 falsos positivos, resultando em Precision de aproximadamente 0,0327.

### Conclusões

- O equilíbrio global das classes não representa equilíbrio ao longo dos blocos.
- O desempenho obtido no particionamento aleatório não se manteve na avaliação temporal.
- Resultados de PR-AUC próximos de 1,0 não constituem evidência suficiente de generalização.
- A remoção de atributos temporais e relacionados a endereços maliciosos altera significativamente o comportamento dos modelos.
- A elevada diferença de prevalência limita a comparação direta entre as métricas dos dois particionamentos.
- A presença de somente 46 registros fraudulentos limita a estabilidade das estimativas.
- O particionamento temporal não elimina necessariamente o risco de vazamento presente na construção original das features agregadas.

### Decisão metodológica

Preservar os resultados estratificados e temporais como avaliações complementares, sem realizar ajustes com base no teste temporal.

Utilizar Average Precision, Precision, Recall, F1 e matrizes de confusão para fundamentar a discussão final, distinguindo AP da PR-AUC trapezoidal.

## 35 - Avaliação do desempenho computacional no ETFD

Foram realizadas medições de latência e throughput das 12 configurações avaliadas no particionamento temporal do ETFD.

### Ambiente experimental

- Plataforma: Google Colab;
- Sistema operacional: Linux;
- Processador: Intel Xeon CPU @ 2.20 GHz;
- Arquitetura: x86_64;
- CPUs lógicas: 2;
- Python: 3.13.18;
- Scikit-learn: 1.6.1;
- XGBoost: 3.4.1;
- NumPy: 2.4.6;
- Pandas: 2.2.3.

### Procedimentos

Foram realizadas 500 medições individuais de inferência e 30 repetições de processamento em lotes de 1.964 registros, precedidas por inferências de aquecimento.

As medições incluíram as transformações internas dos pipelines, a geração das probabilidades e a aplicação do threshold de classificação.

### Resultados do Cenário B

| Modelo | AP | Latência média (ms) | p95 (ms) | p99 (ms) | Throughput (registros/s) |
|---|---:|---:|---:|---:|---:|
| Regressão Logística | 0,002901 | 9,7733 | 22,3712 | 32,9877 | 291873,9614 |
| XGBoost | 0,437440 | 6,9829 | 7,8286 | 15,2225 | 172912,8712 |
| Árvore de Decisão | 0,028421 | 5,6011 | 6,7644 | 8,5422 | 309789,3856 |

O XGBoost apresentou maior capacidade discriminativa nesse cenário, embora tenha obtido menor throughput.

A Árvore de Decisão apresentou desempenho computacional favorável, mas menor capacidade discriminativa na avaliação temporal.

### Limitações

As medições foram realizadas em ambiente computacional compartilhado, sujeito a variações de desempenho.

O benchmark não inclui coleta de dados, comunicação com a blockchain ou extração original das features.

Embora os benchmarks dos dois datasets utilizem procedimentos semelhantes, foram realizados em sessões distintas do Google Colab.

Os resultados representam o desempenho das configurações experimentais e não comprovam a viabilidade de um sistema completo de detecção de fraudes em tempo real.

### Decisão

Encerrar as medições computacionais e consolidar os resultados preditivos e computacionais dos datasets ETFD e Vagifa.

## 36 - Padronização da comparação entre particionamentos do ETFD

Foram reproduzidos os experimentos estratificados utilizando as mesmas configurações de pipelines e hiperparâmetros empregados na avaliação temporal.

Foram calculadas Average Precision, PR-AUC trapezoidal, ROC-AUC, Precision, Recall, F1 e matrizes de confusão.

| Modelo | Cenário | AP estratificada | AP temporal |
|---|---|---:|---:|
| Regressão Logística | A | 0,986404 | 0,046722 |
| Regressão Logística | B | 0,892020 | 0,002901 |
| Regressão Logística | C | 0,984417 | 0,048494 |
| Regressão Logística | D | 0,806589 | 0,002837 |
| XGBoost | A | 0,999917 | 0,312258 |
| XGBoost | B | 0,999108 | 0,437440 |
| XGBoost | C | 0,999273 | 0,289699 |
| XGBoost | D | 0,973137 | 0,325636 |
| Árvore de Decisão | A | 0,994915 | 0,007850 |
| Árvore de Decisão | B | 0,979014 | 0,028421 |
| Árvore de Decisão | C | 0,989795 | 0,006045 |
| Árvore de Decisão | D | 0,905725 | 0,007055 |

A padronização confirmou que as métricas obtidas na divisão aleatória estratificada não se mantiveram na avaliação cronológica.

O XGBoost apresentou os maiores valores de AP nos quatro cenários temporais, destacando-se o Cenário B, com AP de 0,437440.

A prevalência de fraude passou de aproximadamente 49,44% no teste estratificado para 0,184% no teste temporal.

As diferenças de prevalência, tamanho dos conjuntos de treinamento e composição temporal limitam a comparação direta dos resultados.

O teste estratificado possui caráter exploratório, pois já havia sido utilizado em experimentos anteriores.

**Decisão:** encerrar os experimentos de modelagem do ETFD, preservando os resultados dos dois particionamentos e utilizando-os como avaliações complementares.

Nenhum novo ajuste será realizado com base nos resultados temporais.
