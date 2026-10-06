# Manutenção Preditiva — Análise e Comparação de Classificadores

Projeto de encerramento do Módulo 1 do curso **Desenvolvimento de IA para Análise Preditiva [T3]**.

**Autora:** Isabela Gomes da Costa.

## Objetivo

Construir um pipeline de Ciência de Dados para prever falhas mecânicas em equipamentos industriais a partir de medições de sensores. A variável alvo é `falha_maquina`: **0** representa funcionamento normal e **1**, falha.

O projeto compara KNN e Árvore de Decisão em dois tratamentos de dados ausentes: remoção de linhas incompletas, adotada como fluxo principal, e imputação pela mediana, como experimento complementar. O resultado é um notebook analítico; não inclui monitoramento em tempo real nem uma aplicação de produção.

## Base de dados

O CSV é carregado diretamente do [Google Drive](https://drive.google.com/file/d/1er0sPjY51RymKrRkzZkjFOVwJ8PEZoAZ/view). A execução depende de conexão com a internet e da disponibilidade pública do arquivo.

A base original contém **10.000 registros e 14 colunas**, com **339 falhas (3,39%)**. Existem 500 registros com ausência simultânea de temperatura do ar, temperatura do processo, velocidade de rotação e torque. O tratamento por remoção mantém 9.500 registros; a imputação mantém 10.000.

Conforme as anotações do Departamento de Engenharia:

- `udi` e `id_produto` são identificadores e não entram como preditoras.
- `falha_twf`, `falha_hdf`, `falha_pwf`, `falha_osf` e `falha_rnf` registram motivos de falha e são excluídas das preditoras para evitar vazamento de informação do evento.
- As preditoras incluem cinco sensores numéricos, a categoria `tipo` e duas variáveis derivadas.

## Organização do pipeline

1. **Análise exploratória:** dimensões, tipos, estatísticas descritivas, histograma, distribuição do alvo, correlação de Pearson e interpretação dos resultados.
2. **Limpeza e tratamento:** verificação de duplicatas, identificação dos nulos, comparação entre remoção e imputação pela mediana e boxplots dos sensores.
3. **Feature Engineering:** criação de `potencia = velocidade_rotacao_rpm × torque_nm` e `diferenca_temperatura_k = temperatura_processo_k − temperatura_ar_k` nas duas bases.
4. **Divisão e balanceamento:** divisão estratificada 80/20, com `random_state=42`, e SMOTENC exclusivamente no treino.
5. **Escalonamento:** codificação de `tipo` em indicadores 0/1 e StandardScaler nas variáveis numéricas destinadas ao KNN, com ajuste no treino e transformação do teste. A Árvore recebe valores na escala original.
6. **Ajuste de parâmetros:** KNN com K = 3, 5 e 7; Árvore com profundidades 3, 5 e sem limite. Comparação das acurácias de treino e teste.
7. **Avaliação final:** acurácia, precisão, recall, F1 e acurácia balanceada dos finalistas, com seleção e conclusão.

O produto chamado `potencia` está em **RPM × Nm** e é proporcional à potência mecânica; não é expresso em watts. A diferença térmica está em kelvin.

O SMOTENC preserva a natureza categórica de `tipo`. Para o cálculo das distâncias, os atributos numéricos são padronizados temporariamente usando apenas o treino e depois retornam à escala original. As fórmulas das variáveis derivadas são recalculadas nos registros sintéticos.

## Tecnologias

- **Python e Jupyter Notebook:** desenvolvimento e apresentação do pipeline.
- **Pandas e NumPy:** manipulação de dados e operações numéricas.
- **Matplotlib e Seaborn:** gráficos analíticos.
- **Scikit-learn:** divisão dos dados, codificação, escalonamento, classificadores e métricas.
- **Imbalanced-learn:** balanceamento por SMOTENC.
- **Git e GitHub:** versionamento do projeto.

## Estrutura de entrega

```text
Desenvolvimento-de-IA-para-Analise-Preditiva-Projeto-Final-Modulo-1/
├── desenvolvimento_de_ia_para_analise_preditiva_projeto_final_modulo_1.ipynb
├── README.md
└── requirements.txt
```

Como o notebook carrega o CSV por URL, não exige um diretório local de dados nem caminhos absolutos. Caso seja adotado carregamento local futuramente, organizar o arquivo em `data/` e usar caminho relativo.

## Como executar

O notebook salvo registra **Python 3.14.5**. Para reproduzir o ambiente, utilizar essa versão e instalar as dependências em um ambiente virtual.

### 1. Clonar o repositório

```powershell
git clone https://github.com/isabelagomesdacosta-gif/Desenvolvimento-de-IA-para-Analise-Preditiva-Projeto-Final-Modulo-1.git
cd Desenvolvimento-de-IA-para-Analise-Preditiva-Projeto-Final-Modulo-1
```

Selecionar a branch que contém o notebook. A branch identificada durante o desenvolvimento é `1_analise_exploratoria`; a entrega final deve estar na `main`. Confirmar quais arquivos já foram publicados antes de executar.

### 2. Criar o ambiente e instalar as dependências

No terminal PowerShell do Windows:

```powershell
py -3.14 -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
.\.venv\Scripts\python.exe -m ipykernel install --user --name manutencao-preditiva --display-name "Python (Manutenção Preditiva)"
```

O requirements fixa as versões confirmadas nas saídas de instalação do notebook e usa faixas para os pacotes cujas versões exatas não foram recuperadas. Portanto, ainda não é um congelamento integral do ambiente original. Após executar e validar o notebook, registrar as versões efetivamente utilizadas para fechar a reprodutibilidade.

### 3. Executar o notebook

1. Abrir a pasta no VS Code com as extensões Python e Jupyter instaladas.
2. Abrir `desenvolvimento_de_ia_para_analise_preditiva_projeto_final_modulo_1.ipynb`.
3. Selecionar o kernel **Python (Manutenção Preditiva)**.
4. Reiniciar o kernel e executar todas as células na ordem, do início ao fim.
5. Conferir os resultados e salvar o notebook.

A célula `%pip install imbalanced-learn` verifica a instalação da biblioteca no ambiente do notebook; com o requirements instalado, o pacote já estará disponível.

## Resultados registrados

Os valores abaixo são das execuções salvas no notebook, não de uma nova execução realizada para produzir este README. Os finalistas foram escolhidos pela maior acurácia de teste na Fase 6.

| Base | Modelo | Configuração | Acurácia | Precisão das falhas | Recall das falhas | F1 das falhas | Acurácia balanceada |
|---|---|---|---:|---:|---:|---:|---:|
| Sem nulos | KNN | K = 3 | 93,95% | 32,17% | 71,88% | 44,44% | 83,30% |
| Sem nulos | Árvore | Sem limite | 95,89% | 43,86% | 78,12% | 56,18% | 87,32% |
| Mediana | KNN | K = 3 | 93,80% | 30,28% | 63,24% | 40,95% | 79,06% |
| Mediana | Árvore | Sem limite | 95,20% | 38,52% | 69,12% | 49,47% | 82,62% |

A **Árvore sem limite de profundidade, na base sem nulos**, foi selecionada no fluxo principal. Superou o KNN nas cinco métricas exibidas e também venceu pelo critério de acurácia exigido no enunciado. A seleção complementar prioriza F1, usando recall, acurácia balanceada, precisão e acurácia como desempates.

Prever sempre ausência de falha alcançaria 96,63% de acurácia no teste sem nulos e 96,60% no teste com mediana, mas recall de falhas igual a zero. A Árvore detecta falhas que essa referência ignora. Entretanto, sua precisão de 43,86% indica que mais da metade dos alertas são falsos alarmes. A recomendação é avaliá-la como candidata à adoção, considerando os custos operacionais e validação adicional.

## Limitações e melhorias

- A imputação foi calculada antes da divisão, usando a mediana da base inteira. Melhorar ajustando a imputação somente no treino.
- As bases foram divididas separadamente: os testes possuem registros diferentes. Os resultados não isolam o efeito da remoção frente à imputação. Comparar os tratamentos em um conjunto de teste comum.
- O teste foi consultado na escolha dos hiperparâmetros. Usar validação cruzada estratificada no treino, mantendo um teste final independente e os tratamentos dentro de cada divisão da validação.
- A Árvore sem limite ajustou fortemente o treino. Avaliar poda, `min_samples_leaf` e outras configurações de regularização.
- O fluxo principal remove linhas incompletas; a imputação solicitada pelo enunciado foi implementada no fluxo complementar.
- SMOTENC cria exemplos sintéticos e não garante que todas as combinações correspondam a condições físicas reais. Validar a plausibilidade com a Engenharia.
- Avaliar custos de falhas não detectadas e alarmes falsos, limiares de decisão, curvas precisão-recall e novos dados de equipamentos.
- Validar a execução integral em ambiente limpo e fixar todas as versões efetivamente testadas.

## Versionamento e organização das tarefas

O trabalho foi organizado por fases do pipeline, mantendo a base original e versões separadas para comparar os tratamentos.

## Vídeo e entrega

**Link do vídeo: a adicionar após a gravação.**


