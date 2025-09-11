# Projeto: Apoio Analítico à Alocação Orçamentária com IA

## 📌 Visão geral
Este repositório implementa um arranjo híbrido para apoio analítico à alocação orçamentária com dados públicos.  
A solução tem duas etapas:

1. **Previsão (XGBoost)**  
   - Treina um modelo multissaída para prever crescimento do PIB, inflação e índice de Gini a partir das 29 funções de despesa.  
   - Inclui aumento de dados (sintéticos) e avaliação (curva de aprendizado, MSE, R², R² ajustado).  

2. **Otimização (Optuna/TPE)**  
   - Busca uma configuração de alocação (em torno da mediana histórica ± 1 desvio-padrão) que maximiza:  

     ```
     score = (PIB) − (inflação) − (Gini) − (penalidade por desvio do baseline)
     ```

🔄 **Reprodutibilidade**: sementes fixadas em `42` (Python/NumPy/TensorFlow/XGBoost).  
Ainda assim, pequenas variações entre execuções podem ocorrer por paralelismo e ponto flutuante.

---

## ⚙️ Como reproduzir

### Ambiente

```bash
pip install -U pip
pip install xgboost==1.7.6 optuna==3.6.1 scikit-learn==1.3.2 \
            pandas==2.1.4 numpy==1.24.4 matplotlib==3.8.2 joblib==1.3.2 \
            tensorflow-cpu==2.13.1 keras==2.13.1

```

## Ordem sugerida de execução

**orcamento_ia_xgboost.ipynb**
→ Gera dados sintéticos, treina o XGBoost, produz métricas/gráficos e salva saved_model.pkl e arquivos de apoio.

**optimize_allocation.ipynb** 
→ Carrega saved_model.pkl e augmented_dataset.csv, roda a otimização e salva os resultados.

## 📂 Descrição dos arquivos
📊 **Dados/Arquivos de entrada**

**Raw_Data_Input_Output_Variables.xlsx**
Planilha base com as séries históricas (despesas por função e indicadores de saída). Fonte primária para preparação/normalização.
**Processed_Normalized_Data.csv**  
Arquivo intermediário resultante do pré-processamento da planilha bruta.  
Inclui as 29 funções de despesa (entradas) e os 3 indicadores socioeconômicos (saídas) já padronizados pelo método z-score.  
Usado como base para gerar dados sintéticos e treinar o modelo XGBoost
**augmented_dataset.csv**
Conjunto aumentado: dados originais + sintéticos (ruído gaussiano, bootstrapping e VAE para as entradas), já normalizados.
Usado para treinar/avaliar o XGBoost e, na otimização, para definir a mediana e o desvio-padrão de cada variável de entrada (limites de busca).

## 📁 Dados/Arquivos de saída / artefatos

**saved_model.pkl** — modelo XGBoost multissaída treinado (previsão de PIB, inflação e Gini).

**comparison_table.csv** — tabela com valores verdadeiros e previstos para cada variável de saída (todas as observações de validação agregadas).

**best_allocation.csv** — linha única com as 29 funções de despesa resultantes da melhor solução encontrada pela otimização (valores padronizados).

**original_inputs_descriptive_stats.csv** — estatísticas descritivas (média, desvio-padrão, mínimo e máximo) das entradas originais.

**synthetic_inputs_descriptive_stats.csv** — estatísticas descritivas das entradas sintéticas.

**original_outputs_descriptive_stats.csv** — estatísticas descritivas das saídas originais (PIB, inflação, Gini).

**synthetic_outputs_descriptive_stats.csv** — estatísticas descritivas das saídas sintéticas.


## 📒 Notebooks / Scripts
### 1) **orcamento_ia_xgboost.ipynb**

Pipeline de previsão e avaliação:

- Lê o dataset normalizado, separa 29 entradas e 3 saídas.

- **Geração de dados sintéticos:**

  1 Ruído gaussiano

  2 Bootstrapping (reamostragem)

  3 VAE (aplicado apenas às entradas)

- Concatena originais + sintéticos → augmented_dataset.csv.

- Treina XGBoost (n_estimators=980; learning_rate=0.002; seed=42) com K-Fold (k=5).

- Calcula MSE, RMSE, R² e R² ajustado (geral e por variável).

- Plota/salva comparações True vs. Predicted, gráficos de dispersão e curva de aprendizado.

- Exporta estatísticas descritivas (originais vs. sintéticos) e salva o modelo em saved_model.pkl.

### 2) **optimize_allocation.ipynb**

Pipeline de otimização (busca TPE/Optuna):

- Carrega saved_model.pkl e augmented_dataset.csv.

- Define baseline (mediana) e limites por variável: mediana ± 1 desvio-padrão.

- Função-objetivo:

> score = PIB − inflação − Gini − λ·|desvio|, com λ = 0,5

- Executa 1.000 trials (seed=42) e retorna os parâmetros da melhor alocação.

- Salva best_allocation.csv e graphs/optimized_allocation.png.

- Imprime as previsões (PIB, inflação, Gini) para a solução ótima.

- Produz os mesmos artefatos (best_allocation.csv, optimized_allocation.png) e imprime as previsões da melhor alocação.



**ℹ️ Observação: todos os valores usados pelo modelo estão padronizados (z-score), o que significa que resultados e alocações são reportados em desvios-padrão relativamente às distribuições históricas.**
