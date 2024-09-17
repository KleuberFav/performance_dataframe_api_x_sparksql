# Há diferença no tempo de processamento quando usamos DataFrame API ou Spark SQL?

Nos últimos tempos, surgiu uma questão interessante e relevante: Existe uma diferença significativa no tempo de processamento quando utilizamos a DataFrame API em comparação com o Spark SQL no Apache Spark? Este questionamento é crucial, especialmente para otimizar o desempenho de operações de dados e garantir a eficiência no processamento de grandes volumes de informação.


Para investigar essa questão, decidi realizar um estudo comparativo entre as duas abordagens. A ideia foi examinar como cada método se comporta em termos de tempo de execução para operações similares e determinar se existe uma diferença estatisticamente significativa na performance entre eles.

Para isso, realizei uma série de testes, medindo o tempo necessário para completar operações de agregação usando tanto a DataFrame API quanto o Spark SQL. Essas operações foram executadas em um ambiente controlado e repetidas diversas vezes para garantir a robustez dos resultados.

É importante destacar que, independentemente da abordagem utilizada, tanto a DataFrame API quanto o Spark SQL se beneficiam do Catalyst Optimizer, o componente de otimização do Spark. O Catalyst Optimizer é responsável por melhorar o desempenho das consultas, aplicando uma série de transformações e otimizações para gerar um plano de execução eficiente.

Como o Catalyst Optimizer é utilizado por ambas as abordagens, é razoável supor que, em muitos casos, não haverá uma diferença significativa de desempenho entre elas. O otimizador realiza otimizações como a reordenação de operações, eliminação de redundâncias e seleção de estratégias de execução eficientes, o que tende a equalizar o desempenho das consultas, independentemente da interface utilizada.

Portanto, a análise comparativa realizada visa explorar qualquer diferença remanescente que possa surgir, apesar da otimização proporcionada pelo Catalyst. O objetivo é fornecer uma análise detalhada que possa auxiliar na escolha da abordagem mais eficiente para diferentes cenários de processamento de dados no Spark. Vamos explorar os métodos e resultados obtidos para entender melhor como cada técnica se comporta em termos de desempenho e eficiência.

# Menu
- [Spark](#spark)
- [Dados](#dados)
- [Instância e Coleta de Dados](#instância-e-coleta-de-dados)
- [Testes](#testes)
- [Procedimento](#procedimento)
- [Autor](#autor)

# Spark

A história do Apache Spark começa em 2009, quando Matei Zaharia, um estudante da Universidade de Berkeley, observou as limitações do Hadoop MapReduce, que processava grandes volumes de dados de forma eficiente, mas com algumas barreiras, como a alta latência em operações iterativas e interativas. O Spark foi desenvolvido para superar essas limitações, trazendo uma solução que combinava velocidade e flexibilidade, capaz de processar dados em memória e reduzir drasticamente o tempo de execução de várias tarefas.

![alt text](https://github.com/KleuberFav/performance_dataframe_api_x_sparksql/blob/main/artefatos/image-2.png?raw=true)

Componentes principais e evolução do Spark:

- RDD (Resilient Distributed Dataset) - 2010 O primeiro grande avanço do Spark foi o RDD, um conceito fundamental que garante tolerância a falhas e execução eficiente de operações paralelas. Ele permite a reutilização de dados em memória, diferentemente do MapReduce, que exigia gravar os dados no disco a cada etapa de processamento.

- Spark SQL e o Catalyst Optimizer - 2014 O próximo marco foi a introdução do Spark SQL, que trouxe uma API estruturada para realizar consultas SQL sobre grandes volumes de dados. Ele é integrado ao Catalyst Optimizer, que otimiza tanto operações SQL quanto DataFrame, garantindo alto desempenho ao traduzir as consultas para um plano de execução otimizado. Isso permitiu que usuários de SQL interagissem com os dados no Spark de maneira familiar e eficiente.

- DataFrame API - 2015 Inspirada em bibliotecas como pandas (em Python) e R, a DataFrame API foi introduzida para fornecer uma abstração de dados estruturados e semiestruturados. Essa API facilita operações como agregações, joins e filtragens de dados distribuídos, com otimização automática via o Catalyst. A DataFrame API abstrai a complexidade de trabalhar diretamente com RDDs, tornando o Spark acessível para mais tipos de usuários.

- Dataset API - 2016 O Dataset é uma evolução da DataFrame API, que combina as vantagens dos RDDs com as da DataFrame, oferecendo uma estrutura de dados mais fortemente tipada. Ao mesmo tempo, ele fornece a capacidade de manipular dados estruturados e semiestruturados com otimização automática pelo Catalyst, preservando a segurança de tipo em tempo de compilação para linguagens como Scala e Java.

- MLlib (Machine Learning Library) - 2013 O Spark expandiu suas capacidades com o MLlib, uma biblioteca para aprendizado de máquina em larga escala. Ela oferece uma variedade de algoritmos de classificação, regressão, clustering e filtragem colaborativa, tornando o Spark uma ferramenta poderosa para cientistas de dados interessados em treinamento de modelos de machine learning em grandes volumes de dados distribuídos.

- GraphX - 2014 Com o GraphX, o Spark passou a suportar o processamento de dados gráficos, como redes sociais ou estruturas em forma de gráfico. Ele permite a manipulação e análise de gráficos distribuídos com uma API flexível, integrando-se com o Spark Core e aproveitando a mesma capacidade de otimização e distribuição de dados.

- Structured Streaming - 2016 O Spark também ampliou seu alcance para processamento de dados em tempo real com o Structured Streaming. Ao contrário de abordagens batch, essa API permite o processamento contínuo e incremental de dados, mantendo a simplicidade e a eficiência da API de DataFrame e Spark SQL. Ele permite que fluxos de dados sejam tratados como tabelas continuamente atualizadas, facilitando o trabalho com dados em tempo real.

### PySpark

O PySpark desempenha um papel central na popularização do Apache Spark. Ele permite que os usuários interajam com todas as funcionalidades do Spark usando uma interface Python, incluindo DataFrames, Spark SQL, MLlib, Structured Streaming e GraphX, trazendo as vantagens da computação distribuída para a ampla comunidade de Python.

### Catalyst Optimizer 

O Spark tem um otimizador chamado Catalyst, ele otimiza automaticamente as consultas, seja em SQL ou em API do Dataframe. Ele contém uma biblioteca para representar árvores e aplicar regras para manipulá-las Como vimos, tanto o Spark SQL quanto o Dataframe API usa o Catalyst para otimizar a consulta. A seguir um diagrama mostrando como o Catalyst funciona:

![alt text](https://github.com/KleuberFav/performance_dataframe_api_x_sparksql/blob/main/artefatos/image.png?raw=true)

- Plano de Execução: Catalyst transforma operações de alta abstração em planos de execução otimizados. Esses planos descrevem como as operações serão executadas de forma eficiente, considerando aspectos como paralelismo e distribuição.

- Regras de Otimização: Utiliza um conjunto de regras e heurísticas para otimizar o plano de execução. As regras aplicam transformações e reordenamentos para melhorar o desempenho, como a eliminação de operações redundantes e a fusão de etapas.

- Catalyst Optimizer: O núcleo do Catalyst é o otimizador, que é responsável por aplicar as regras de otimização. Ele trabalha em duas fases principais: uma fase de resolução e uma fase de execução, ajustando o plano conforme necessário.

- Abstração de Dados: Catalyst abstrai a complexidade do processamento de dados, permitindo que desenvolvedores trabalhem com operações de alto nível sem se preocupar com detalhes de implementação subjacentes.

- Execução Distribuída: Em ambientes de Big Data, Catalyst usa a execução distribuída para processar dados em paralelo em clusters, aproveitando a capacidade de processamento em larga escala.

Além disso, o Catalyst pode decidir se os dados devem ser processados em memória ou persistidos em disco, dependendo de vários fatores, como o tamanho dos dados, a complexidade das operações e os recursos disponíveis.

# Dados

Os dados referem-se a informações meteorológicas dos anos de 2005, 2020, 2021, 2022, 2023 e 2024 (até agosto). Extraídos do site do INMET, esses dados foram limpos, tratados e armazenados na camada Silver dentro do Lake localizado na minha máquina local. Esta camada está organizada por ano e mês, e os dados são armazenados no formato Parquet.

### Detalhes sobre o armazenamento e tamanho dos dados:
- **Formato de Armazenamento**: `EXTRACT_YEAR=YEAR/EXTRACT_MONTH=MONTH`
- **Número de Linhas**: 23.689.584
- **Número de Partições**: 1.014
- **Tamanho Total**: 853,74 MB

# Instância e  Coleta de Dados

Utilizamos o Spark no modo standalone com configurações específicas para otimizar a performance:

- Memória do Driver: 16 GB
- Memória do Executor: 16 GB
- TimeZone: America/Sao_Paulo
- Cores do Executor: 4
- Parallelism: 100
- Partições de Shuffle SQL: 200
- CPUs por Tarefa: 1


**Coleta de Dados**
   - Medimos o tempo de execução de operações de agregação usando tanto a DataFrame API quanto o Spark SQL.

```python
# Consulta usando DataFrame API
df.groupBy("AAAAMM").agg({"PRECIPITACAO": "sum", "TEMPERATURA_AR": "sum"}).collect()

# Consulta usando Spark SQL
spark.sql("""
    SELECT AAAAMM, SUM(PRECIPITACAO) AS PRECIPITACAO, SUM(TEMPERATURA_AR) AS TEMPERATURA_AR 
    FROM DF 
    GROUP BY AAAAMM
""").collect()
```

- PRECIPITACAO é o valor de precipitação em milímetros em uma hora.
- TEMPERATURA_AR é o valor da temperatura do ar em graus Celsius em uma hora.
- AAAAMM é o valor mês-ano.

```python
# Criando listas para armazenar os tempos de execução
df_times = []
sql_times = []

# Executando 60 vezes
for _ in range(60):
    # Limpar cache antes de cada execução
    spark.catalog.clearCache()

    # Usando a API DataFrame
    start_time = time()
    df.groupBy("AAAAMM").agg({"PRECIPITACAO": "sum", "TEMPERATURA_AR": "sum"}).collect()
    df_time = time() - start_time
    df_times.append(df_time)

    # Usando Spark SQL
    start_time = time()
    spark.sql("""
        SELECT AAAAMM, SUM(PRECIPITACAO) AS PRECIPITACAO, SUM(TEMPERATURA_AR) AS TEMPERATURA_AR 
        FROM DF 
        GROUP BY AAAAMM
    """).collect()
    sql_time = time() - start_time
    sql_times.append(sql_time)

# Criando um DataFrame com os resultados
result_df = pd.DataFrame({
    'time_api': df_times,
    'time_sql': sql_times
})
```

- Executamos cada operação 60 vezes para obter uma amostra representativa dos tempos de execução e salvamos em um dataframe pandas;
- spark.catalog.clearCache() é usado para garantir que o Spark não use dados armazenados em cache da execução anterior. Isso ajuda a obter uma medição mais precisa da performance de cada execução, sem a influência dos dados previamente cacheados;
- Time() é usado para capturar o tempo antes e depois da execução de cada abordagem, calculando assim o tempo total gasto em cada uma.
- Os tempos de execução são armazenados em listas (df_times e sql_times), e depois são convertidos em um DataFrame do Pandas para análise.

# Análise Exploratória e Descritiva das Amostras

Fizemos uma breve análise descritiva e exploratória dos dados

### Descritiva

Estatísticas Descritivas:
|        | time_api  | time_sql  |
|--------|-----------|-----------|
| count  | 60.000000 | 60.000000 |
| mean   | 11.883409 | 11.828649 |
| std    | 0.890139  | 0.678450  |
| min    | 10.164215 | 9.927937  |
| 25%    | 11.555114 | 11.593451 |
| 50%    | 11.872428 | 11.893422 |
| 75%    | 12.237808 | 12.123705 |
| max    | 16.472235 | 13.376702 |

- A média de ambos estão entre 11.8 e 11.9 segundos;
- o desvio padrão das amostras do Dataframe API é 0.89 e do Spark SQl um pouco menor, 0.68;
- o menor tempo para Dataframe API é 10.16 segundos e para Spark SQL é 9.92;
- os primeiros, segundos e terceiros quartis estão bem próximos;
- o maior tempo para Dataframe API é 16.47 segundos e para Spark SQL é 13.38;
- possivelmente esse valor máximo para Dataframe API é um outlier, talvez causado por alguma baixa na latência na hora da execução.

### Exploratória

![Gráfico de Disperção](https://github.com/KleuberFav/performance_dataframe_api_x_sparksql/blob/main/artefatos/image-3.png?raw=true)

![alt text](https://github.com/KleuberFav/performance_dataframe_api_x_sparksql/blob/main/artefatos/image-5.png?raw=true)

- Analisando o gráfico de dispersão, podemos ver uma concentração maior entre 11 e 12.5 segundos e variando entre 9 e 13.5 segundos;
- mais uma evidência de outlier em torno de 16 segundos;
- podemos ver no gráfico de boxplot que há vários outlier acima e abaixo dos intervalos interquartis no tempo de execução para Spark SQL;
- confirmamos que há um outlier em 16 segundos quando executamos para Dataframe API;
- há também alguns outlier abaixo do intervalo interquartil para Dataframe API.

# Testes

Para investigar se há uma diferença significativa no tempo de processamento entre a DataFrame API e o Spark SQL, realizamos uma série de testes. Aqui está um resumo do procedimento:

# Passo a Passo para Realizar um Teste de Hipótese

O **teste de hipótese** é uma ferramenta estatística usada para verificar se existe evidência suficiente para rejeitar uma suposição (hipótese nula) sobre uma população, com base em uma amostra de dados. Abaixo, apresento um passo a passo para realizar um teste de hipótese.

## 1. Definir as Hipóteses

### Hipótese Nula (H₀):
A hipótese nula é a suposição inicial, geralmente representando a ausência de efeito ou diferença.
- **Exemplo:** "A média de tempo de resposta da API é igual ao tempo de resposta do banco de dados SQL."

### Hipótese Alternativa (H₁):
A hipótese alternativa é a afirmação que você deseja testar, geralmente representa a existência de um efeito ou diferença.
- **Exemplo:** "A média de tempo de resposta da API é diferente do tempo de resposta do banco de dados SQL."

---

## 2. Escolher o Nível de Significância (α)

O nível de significância representa o risco que você está disposto a correr de rejeitar a hipótese nula incorretamente (falso positivo). Valores comuns de α são 0,05 (5%) ou 0,01 (1%).

- **Exemplo:** Definir α = 0,05 (5%).

---

## 3. Escolher o Teste Estatístico Adequado

Dependendo do tipo de dados e da distribuição, você pode escolher entre diferentes testes. Alguns exemplos são:
- **Teste t de Student** (para comparar médias de duas amostras).
- **Teste qui-quadrado** (para variáveis categóricas).
- **Teste ANOVA** (para comparar mais de duas médias).
- **Teste de Mann-Whitney** (não paramétrico).

---

## 4. Verificar a Normalidade dos Dados

Antes de aplicar muitos testes paramétricos, é importante verificar se os dados seguem uma distribuição normal. Isso é necessário porque muitos testes, como o teste t, assumem que os dados são aproximadamente normais.

### Métodos para Verificar a Normalidade:
- **Histogramas:** Visualize a distribuição dos dados.
- **Gráficos Q-Q (Quantile-Quantile):** Compare a distribuição dos dados com a distribuição normal teórica.
- **Testes de Normalidade:** 
  - **Teste de Shapiro-Wilk:** Adequado para amostras pequenas a moderadas.
  - **Teste de Kolmogorov-Smirnov:** Comparar a distribuição dos dados com a distribuição normal.

---

## 5. Definir a Região Crítica e a Estatística do Teste

A **região crítica** é o intervalo de valores onde a hipótese nula será rejeitada. O cálculo dessa região depende da distribuição da amostra e do teste escolhido. Para muitos testes, você calculará o valor crítico com base no nível de significância.

---

## 6. Coletar os Dados e Calcular a Estatística do Teste

Após escolher o teste adequado e verificar a normalidade, aplique o teste aos dados e calcule o valor da estatística de teste. Isso geralmente envolve fórmulas específicas para cada tipo de teste (como a fórmula do teste t ou do qui-quadrado).

---

## 7. Tomar uma Decisão

Compare o valor da estatística de teste com o valor crítico ou utilize o **p-valor**:

- **Se o p-valor ≤ α**: Rejeite a hipótese nula (há evidências significativas para a hipótese alternativa).
- **Se o p-valor > α**: Não rejeite a hipótese nula (não há evidências suficientes para rejeitar a hipótese nula).

---

## 8. Conclusão

Baseado na decisão do passo anterior, escreva a conclusão, interpretando os resultados de acordo com o contexto do problema.

- **Exemplo:** "Com um p-valor de 0,03, rejeitamos a hipótese nula. Portanto, há evidências suficientes para afirmar que o tempo de resposta da API é diferente do tempo de resposta do banco de dados SQL."

### Passo a Passo:
1. **Nível de significância:** α = 0,05
2. **Verificar normalidade:** Use testes de normalidade e gráficos.
3. **Teste escolhido:** Teste t de Student (para amostras independentes)
4. **Calcular o valor do teste:** Aplicar o teste aos dados coletados.
5. **Decisão:** Comparar o valor t com o valor crítico ou usar o p-valor para decidir.
6. **Conclusão:** Se o p-valor for menor que 0,05, rejeitamos H₀.

# Procedimento

1. **Definir as hipóteses:**
- Hipótese Nula (H0): As medianas dos tempos de execução entre a API DataFrame e o Spark SQL são iguais.
- Hipótese Alternativa (H1): As medianas dos tempos de execução entre a API DataFrame e o Spark SQL são diferentes.

2. **Definir o Nível de significância:** 
- α = 0,05

3. **Verificar normalidade**
   - Aplicamos o teste de Shapiro-Wilk para verificar se os dados seguem uma distribuição normal, a fim de avaliar a possibilidade de usar um teste t pareado para comparação das amostras de Spark SQL e DataFrame API.

**Como o Teste de Shapiro-Wilk Funciona**: O teste de Shapiro-Wilk é um teste estatístico que avalia a normalidade de uma distribuição de dados. Ele testa a hipótese nula de que a amostra vem de uma distribuição normal. O teste calcula uma estatística de W que mede a concordância entre os dados observados e a distribuição normal esperada. Um valor p pequeno (menor que um nível de significância pré-determinado, como 0,05) indica que os dados não seguem uma distribuição normal, enquanto um valor p maior sugere que a distribuição dos dados pode ser normal.

## Fórmula do Teste de Shapiro-Wilk

A fórmula para a estatística de teste \( W \) do teste de Shapiro-Wilk é dada por:

$$
W = \frac{\left(\sum_{i=1}^{n} a_i x_{(i)}\right)^2}{\sum_{i=1}^{n} (x_i - \bar{x})^2}
$$

Onde:

- \( x_{i} \) são os dados ordenados da amostra.
- \( \bar{x} \) é a média dos dados.
- \( a_i \) são os coeficientes derivados da distribuição normal.
- \( n \) é o número de observações na amostra.

A estatística \( W \) é então comparada com uma distribuição teórica para determinar o valor p, que é usado para avaliar a hipótese nula de que os dados seguem uma distribuição normal.


### Hipóteses:
* H0: Os resíduos das amostras não seguem uma distribuição normal
* H1: Os resíduos das amostras seguem uma distribuição normal

```python
from scipy.stats import shapiro

# Calculando as diferenças entre os pares
differences = np.array(df_times) - np.array(sql_times)

# Teste de normalidade
stat, p_value = shapiro(differences)
print(f'Estatística do teste de Shapiro-Wilk: {stat:.4f}')
print(f'Valor p: {p_value:.4f}')
```

![alt text](https://github.com/KleuberFav/performance_dataframe_api_x_sparksql/blob/main/artefatos/image-1.png?raw=true)
- Estatística do teste de Shapiro-Wilk: 0.8607
- Valor p: 0.0000

*Como o p-valor é menor que o valor de significância do teste (0.05), rejeitamos a hipótese nula. Logo, os resíduos não seguem uma distribuição normal*

4. **Teste escolhido:**
   - Como os dados não seguiram uma distribuição normal, optamos por testes não paramétricos para uma análise mais robusta.
   - Utilizamos o teste de Wilcoxon, um teste não paramétrico para amostras pareadas, adequado para comparar os tempos resultantes da mesma consulta em APIs diferentes (DataFrame API e Spark SQL).

**Como o Teste de Wilcoxon Funciona**: O teste de Wilcoxon de sinais rangos é usado para comparar duas amostras pareadas quando os dados não são normalmente distribuídos. Ele avalia a diferença entre pares de observações e verifica se as diferenças medianas são significativamente diferentes de zero. O teste calcula a soma dos rangos das diferenças positivas e negativas e usa essa soma para determinar a significância estatística. Um valor p baixo indica que há uma diferença significativa entre as amostras pareadas.

## Fórmula do Teste de Wilcoxon

O teste de Wilcoxon para amostras pareadas é utilizado para testar a hipótese de que duas amostras emparelhadas vêm da mesma distribuição. A fórmula para a estatística de teste \( W \) é dada por:

$$
W = \sum_{i=1}^{n} R_i
$$

Onde:

- \( R_i \) é a soma dos postos das diferenças absolutas entre pares, onde as diferenças são ordenadas em magnitudes.
- \( n \) é o número total de pares.

O cálculo dos postos envolve os seguintes passos:

1. Calcular as diferenças \( d_i \) entre os pares.
2. Ordenar as diferenças absolutas e atribuir os postos \( R_i \) a cada diferença.
3. Somar os postos das diferenças positivas e das diferenças negativas separadamente.
4. A estatística \( W \) é a menor dessas duas somas.



5. **Calcular o valor do teste:**

```python
from scipy.stats import wilcoxon

# Realizando o teste de Wilcoxon para amostras emparelhadas
stat, p_value = wilcoxon(df_times, sql_times)

# Exibindo o resultado
print(f'Estatística de Wilcoxon: {stat:.4f}')
print(f'Valor p: {p_value:.4f}')

# Definindo as hipóteses
print("\nHipóteses:")
print("Hipótese Nula (H0): As medianas dos tempos de execução entre a API DataFrame e o Spark SQL são iguais.")
print("Hipótese Alternativa (H1): As medianas dos tempos de execução entre a API DataFrame e o Spark SQL são diferentes.")

# Verificando o resultado
if p_value < 0.05:
    print("\nResultado:")
    print(f"Como o p-valor ({p_value}) é menor que o nível de significância de 5%, rejeitamos a hipótese nula. \nLogo, as medianas são significativamente diferentes.")
else:
    print("\nResultado:")
    print(f"Como o p-valor ({p_value}) é maior ou igual ao nível de significância de 5%, não há evidências suficientes para rejeitar a hipótese nula.  \nLogo não há diferença significativa entre as medianas.")
```

- Estatística de Wilcoxon: 896.0000
- Valor p: 0.8888

6. **Decisão:**
* Como o p-valor (0.8887623436669472) é maior ou igual ao nível de significância de 5%, não há evidências suficientes para rejeitar a hipótese nula. Logo não há diferença significativa entre as medianas.

7. **Conclusão:**

Após executar 60 amostras para cada tipo de execução (Spark SQL e Dataframe API), Verificamos se os resíduos das amostras entre os tipos de execução segue uma distribuição normal. Analisando o resultado do teste shapiro-wilk, verificamos que os resíduos não seguem uma distribuição normal, então seguimos com um teste paramétrico para verificar se há diferenças no tempo de execução entre as diferentes API.
O teste escolhido foi a Estatística de Wilcoxon, cujo o p-valor foi maior que o nível de significância de 5%. Logo chegamos a conclusão que não há evidências para rejeitar a hipótese nula. Portanto, ao nível de confiança de 95%, podemos dizer que não há diferença na performance entre Dataframe API e Spark SQL.

# Autor

### Kleuber Favacho 
*Engenheiro de Dados e Estatístico*

- [Linkedin](https://www.linkedin.com/in/kleuber-favacho/)
- [Github](https://github.com/KleuberFav/)