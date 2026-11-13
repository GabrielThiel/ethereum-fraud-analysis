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
  
