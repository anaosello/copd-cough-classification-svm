# Classificação de Tosse para Identificação de DPOC com SVM

Projeto desenvolvido para a disciplina **SCC0233 - Aplicações de Aprendizado de Máquina e Mineração de Dados**, do **ICMC/USP**, em 2026.

## Sobre o projeto

O objetivo é investigar a classificação de gravações de tosse entre **DPOC (Doença Pulmonar Obstrutiva Crônica)** e **saudável** utilizando características acústicas e **Support Vector Machines (SVM)**. O trabalho compara uma SVM com parâmetros padrão e outra com hiperparâmetros otimizados, analisando os efeitos da otimização sobre a identificação de gravações associadas à DPOC.

A proposta utiliza uma abordagem clássica de aprendizado de máquina, em contraste com o aprendizado profundo autossupervisionado empregado no estudo de referência. **Este é um experimento acadêmico e não constitui ferramenta de diagnóstico médico.**

## Dados e metodologia

Foi utilizado o conjunto público **[COPD-Cough (CC)](https://zenodo.org/records/10209837)**, com **853 gravações de tosse** em formato WAV:

- **221 gravações de DPOC (25,9%)**;
- **632 gravações de pessoas saudáveis (74,1%)**.

Os áudios originais possuem taxa de amostragem de 48 kHz e dois canais. O processamento incluiu conversão para mono, reamostragem para 16 kHz, normalização de amplitude e redução de trechos silenciosos.

Foram extraídos **88 atributos por gravação**, a partir de MFCCs, delta-MFCCs e descritores espectrais, utilizando médias e desvios-padrão. Os atributos foram padronizados com `StandardScaler` antes da classificação.

### Treinamento e avaliação

- **Classificador:** SVM com kernel radial (RBF).
- **Divisão dos dados:** `StratifiedGroupKFold`, com grupos definidos pelas pastas das gravações para impedir que arquivos da mesma pasta aparecessem no desenvolvimento e no teste.
- **Conjunto de desenvolvimento:** 685 gravações; **teste final:** 168 gravações.
- **Otimização:** `RandomizedSearchCV`, com 30 combinações de `C`, `gamma` e pesos de classe, utilizando cinco folds no conjunto de desenvolvimento.
- **Critério de seleção:** acurácia balanceada.
- **Métricas:** acurácia, acurácia balanceada, sensibilidade, especificidade, F1 para DPOC e AUC-ROC.

A melhor configuração encontrada utilizou **`C = 65,4121`**, **`gamma = 0,0003511`** e **`class_weight = "balanced"`**. O conjunto de teste não participou da seleção dos parâmetros.

## Resultados

**Resultados no teste final, com DPOC como classe positiva (168 gravações):**

| Modelo | Acurácia (%) | Ac. balanceada (%) | Sensibilidade (%) | Especificidade (%) | F1 DPOC | AUC-ROC |
|---|---:|---:|---:|---:|---:|---:|
| SVM padrão | **74,4** | 53,1 | 9,3 | **96,8** | 0,157 | 0,625 |
| SVM otimizada | 67,3 | **61,2** | **48,8** | 73,6 | **0,433** | **0,655** |

A otimização aumentou a sensibilidade para DPOC de **9,3% para 48,8%** e o F1 de **0,157 para 0,433**. O número de gravações positivas identificadas corretamente passou de **4 para 21**, mas os falsos positivos aumentaram de **4 para 33** e a acurácia total diminuiu.

Apesar da melhora, o modelo otimizado ainda classificou incorretamente **22 das 43 gravações de DPOC** presentes no teste. O estudo de referência obteve métricas superiores com aprendizado profundo autossupervisionado, mas empregou divisões dos dados e procedimentos experimentais diferentes, o que limita uma comparação direta.

### Limitações

O modelo classifica **gravações, não pacientes**. Não foi possível confirmar que cada pasta corresponde a uma pessoa distinta, e diferenças entre os períodos, equipamentos ou condições de coleta podem influenciar os sinais. A avaliação final utilizou apenas uma divisão de teste. Dessa forma, os resultados **não são suficientes para recomendar o classificador para uso clínico**.

## Como executar

O notebook foi preparado para o **Google Colab**:

1. Abra [`copd_cough_svm.ipynb`](copd_cough_svm.ipynb) no Google Colab.
2. Execute as células na ordem apresentada.
3. A primeira etapa instala as bibliotecas Python e o utilitário `unrar`, além de baixar e extrair automaticamente o conjunto **COPD-Cough (CC)** do Zenodo, caso ele ainda não esteja disponível em `/content/sounds`.
4. As etapas seguintes extraem os atributos, realizam a análise exploratória, treinam e otimizam a SVM e avaliam os modelos no conjunto de teste.

**Principais bibliotecas:** `librosa`, `soundfile`, `scikit-learn`, `numpy`, `pandas`, `scipy`, `matplotlib`, `seaborn` e `tqdm`.

> **Observação:** o download do dataset possui aproximadamente **549 MB** e exige espaço para extração. O notebook utiliza caminhos e comandos específicos do Google Colab (`/content`, `apt-get` e `unrar`). Para executar localmente, adapte essas etapas ao ambiente utilizado.

## Referências

- SUN, W. et al. **COPD-Cough (CC)**. Zenodo, 2023. [Conjunto de dados](https://zenodo.org/records/10209837).
- SUN, W. et al. **Self-supervised learning for intelligent disease diagnosis using audio signals: beyond COPD to a spectrum of diseases**. *Applied Intelligence*, v. 55, artigo 487, 2025. [Artigo](https://doi.org/10.1007/s10489-024-06028-2).
