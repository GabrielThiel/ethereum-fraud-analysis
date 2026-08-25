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
