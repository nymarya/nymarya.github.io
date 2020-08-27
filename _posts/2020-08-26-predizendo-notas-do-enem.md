---
layout: post
title: "Predizendo notas do ENEM"
image: "https://www.provafacilnaweb.com.br/wp-content/uploads/2018/10/corrigir-prova-f%C3%A1cil.jpg"
categories:
  - pt-br
tags:
  - python
  - predição
  - pandas
  - enem
---

Para entrar na aceleração de Data Science do [Codenation](https://www.codenation.dev), os alunos tinham que predizer as notas de matemática do enem 2016. A "nota de corte" era 90% de acerto das notas e minha primeira submissão deve ter sido uns 91%. Na oitava semana veio o penúltimo desafio, que era o mesmo da inscrição. Então nada melhor do que aproveitar a oportunidade para refazer a solução e usar os novos conhecimentos para tentar melhorar a pontuação. O código está disponível [aqui](https://github.com/nymarya/aceleradev/blob/master/enem-2).

### O feijão com arroz

Para os 90%, fui no básico: um modelo simples, um pré-processamento mais cuidadoso.

<div align="center">
<img src="https://media.giphy.com/media/xT1XH14zaGcdC4j2bC/giphy.gif">
</div>
<br>

O modelo escolhido foi o LinearRegression, com os parâmetros já pre-estabelecidos.

No tratamento, primeiro utilizei [pandas-profiling](https://github.com/pandas-profiling/pandas-profiling) para ter uma visão geral do banco de treinamento. A ferramente dá todas as estatísticas e já indica quais colunas tem correlação entre si. Retirei essas. Retirei também as que tinham 50% ou mais de valores nulos ou as colunas que tinham valor constante. Feita a separação entre features numericas e categoricas. As numéricas imputei os valores nulos com -1 e padronizei com Standard scaler. As categoricas imputei com a string "N/A" e usei o OneHotEncoder. Isso aumenta significativamente o numero de colunas, então tive que usar o TruncatedSVD para reduzir a dimensionalidade, que sem tratamento fica 61462.

### Digievoluindo a solução

<div align="center">
<img src="https://media.giphy.com/media/nDb2WPoK832fK/giphy.gif">
</div>
<br>

Feito o básico, resta fazer um trabalho mais rebuscado de engenharias de dados. A começar por mais uma etapa de limpeza de dados, dessa vez utilizando da [correlação](https://github.com/nymarya/aceleradev/blob/master/enem-2/src/features/build_features.py#L110) para retirar algumas colunas. Para evitar redundância de dados, são retiradas as colunas numéricas que tem correlação maior que 0.5 ou menor que -0.2 com a coluna alvo `NU_NOTA_MT`.

Algumas coisas simples a serem testadas seria trocar o StandardScaler por outro normalizador como MinMaxScaler, RobustScaler, ou Normalizer. Essa mudança pode ser muito importante pelo fato de que o StandardScaler assume que seus dados seguem uma distribuição normal. De fato, no [desafio](https://github.com/nymarya/aceleradev/blob/master/enem-4/) da semana seguinte, a mudança de StandardScaler para RobustScaler representou uma diferença de uns 5% no desempenho, um salto de 93% para 98%, mas neste caso o resultado não foi tão impactante.

Outra opção seria trocar o encoder por outro como CatBoost ou [OrdinalEncoder](https://scikit-learn.org/stable/modules/generated/sklearn.preprocessing.OrdinalEncoder.html), que, ao invés de cirar uma coluna para cada categoria como o OneHotEncoder, apenas tranforma as categorias na forma númerica. O primeiro não consegui inserir no código, mas o segundo foi testado e os resultados estão na tabela no  começo da próxima seção.

Também foram feitos experimentos com a redução de dimensionalidade testando [RFE](https://scikit-learn.org/stable/modules/generated/sklearn.feature_selection.RFE.html),  [LDA, NCA](https://scikit-learn.org/stable/auto_examples/neighbors/plot_nca_dim_reduction.html) ou [SelectKBest](https://scikit-learn.org/stable/modules/generated/sklearn.feature_selection.SelectKBest.html). O segundo e o terceiro não deram certo por não funcionarem com matriz esparsa (que é no que os dados se transformam após os passos anteriores). O primeiro, por ser recursivo, acaba sendo incompatível com o OneHotEncoder pela gigantesca quantidade de colunas geradas. O SelectKBest também apresentava um problema semelhante. Testei aplicando TruncatedSVD e depois outros para reduzir, mas não deu bom: muita demora, pouca mudança. As outras funções de redução mais lentas funcionaram com OrdinalEncoder por não aumentar a cardinalidade das colunas.

Por fim, em relação aos modelos de regressão, foram testados LinearRegression, [TheisenRegressor](https://scikit-learn.org/stable/modules/generated/sklearn.linear_model.TheilSenRegressor.html) e [RandomForestRegressor](https://scikit-learn.org/stable/modules/generated/sklearn.ensemble.RandomForestRegressor.html) e [DecisionTreeRegressor](https://scikit-learn.org/stable/modules/generated/sklearn.tree.DecisionTreeRegressor.html).

### Avaliando as avaliações

| Correlação | Scaler/Normalizer   | Encoder        | Reduce_dim      | Model                                                                                                                | Score |
| ---------- | ------------------- | -------------- | --------------- | -------------------------------------------------------------------------------------------------------------------- | ----- |
| Sim        | Normalizer          | OneHotEncoder  | SelectKBest(30) | LR                                                                                                                   | 91.72 |
| Sim        | RobustScaler        | OrdinalEncoder | SelectKBest(30) | LR                                                                                                                   | 93.28 |
| Sim        | RobustScaler        | OneHotEncoder  | SelectKBest(10) | LR                                                                                                                   | 93.25 |
| Sim        | RobustScaler        | OneHotEncoder  | SelectKBest(50) | TheisenRegressor                                                                                                     | 92.52 |
| Sim        | QuantileTransformer | OneHotEncoder  | SelectKBest(50) | TheisenRegressor                                                                                                     | 93.07 |
| Não        | QuantileTransformer | OneHotEncoder  | SelectKBest(50) | TheisenRegressor                                                                                                     | 93.36 |
| Não        | RobustScaler        | OneHotEncoder  | SelectKBest(50) | LR                                                                                                                   | 93.37 |
| Não        | RobustScaler        | OneHotEncoder  | RFE(50, LR)     | LR                                                                                                                   | 93.31 |
| Não        | RobustScaler        | OneHotEncoder  | RFE(30, LR)     | LR                                                                                                                   | 93.36 |
| Não        | RobustScaler        | OneHotEncoder  | RFE(30, SVR)    | LR                                                                                                                   | 93.37 |
| Não        | RobustScaler        | OneHotEncoder  | SelectKBest(50) | RandomForest                                                                                                         | 93.5  |
| Não        | RobustScaler        | OneHotEncoder  | SelectKBest(50) | RandomForest(n_estimator=50)                                                                                         | 93.56 |
| Não        | RobustScaler        | OneHotEncoder  | SelectKBest(50) | RandomForest( n_estimators=50, max_depth=4, min_samples_split=4,                                   max_features=0.5) | 93.64 |
| Não        | RobustScaler        | OneHotEncoder  | SelectKBest(50) | DecisionTreeRegressor                                                                                                | 91    |

Tirando o primeiro, todos os algoritmos tem algum fator aleatório, o que quer dizer que para ter uma noção boa do resultado, tem que ser feitas várias execuções para analisar estatisticamente os resultados. Por exemplo, o melhor resultado encontrado foi com RandomForestRegressor, mas ao rodar várias vezes o índice não foi observado novamente muito menos melhorou. De forma geral, os métodos mais rebuscados não tiveram uma diferença tão grande em relação à boa e velha regressão linear.

Concluindo, numa situação de vida real certamente o mais vantajoso, dentre os cenários vistos, seria não sair do feijão com arroz (regressão linear), apenas adicionar umas hortaliças para incrementar o PF (investir no tratamento dos dados).

> Disclaimer: os pontos mais importantes do curso, pessoalmente falando, foram as aulas sobre metodologia científica e estatística, coisas que não foram abordadas nesse post.

Enfim, é isto. Até mais!

## Links úteis

[Feature Scaling with scikit-learn](https://benalexkeen.com/feature-scaling-with-scikit-learn/)

[Repositório](https://github.com/nymarya/aceleradev/tree/master/enem-2)
