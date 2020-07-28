---
layout: post
title: "Predizendo notas do ENEM"
categories:
  - pt-br
tags:
  - python
  - predict
  - pandas
---

Desafio da aceleração do aceleradev era para predizer as notas de matemática do enem 2016. Na oitava semana o curso o desafio voltou. Minha primeira submissão para se não me engano foi 91%.

### O básico

Para os 90%, fu ino básico: um modelo simples, um pré-processamento mais cuidadosa.

O modelo escolhido foi o LinearRegression, com os parâmetros já pre-estabelecidos.

No tratamento, primeiro utilizei pandas-profiling para ter uma visão geral do banco de treinamento. A ferramente dá todas as estatísticas e já indica quais colunas tem correlação entre si. Retirei essas. Retirei também as que tinham 50% ou mais de valores nulos ou as colunas que tinham valor constante. Feita a separação entre features numericas e categoricas. As numéricas imputei os valores nulos com -1 e padronizei com Standard scaler. As categoricas imputei com a string "N/A" e usei o OneHotEncoder. Isso aumenta significativamente o numero de colunas, então tive que usar o TruncatedSVD para reduzir a dimensionalidade.

### Digievoluindo a solução

Feito o básico, resta fazer um trabalho mais rebuscado de engenharias de dados.

Algumas coisas simples a serem testadas seria trocar o standarscaler por outro normalizador como MinMAxScaler, RobustScaler, ou Normalizer. Trocar o encoder por outro CatBoost, LabelEncoder. Diminuir a dimensionalidade com o RFE, ou [LDA, ou NCA](https://scikit-learn.org/stable/auto_examples/neighbors/plot_nca_dim_reduction.html), SelectKBest

### Inserindo resumo das linguagens

Mais uma vez, Simon utilizou uma forma um pouco mais elegante. Dessa vez, foi usando a API em GraphQl para recuperar informações do GitHub.

Resolvi usar a API REST pela praticidade. Usei ela duas vezes: uma para recuperar os nomes dos repositórios (`https://api.github.com/users/<usuario>/repos`) e outra para recuperar as linguagens em cada repositório (`https://api.github.com/repos/<usuario>/<repo>/languages'`). A versão sem autenticação da API, além de acessar só os repositórios públicos, tem um limite de 60 requisições por hora, então para quem possui 60 repositórios públicos ou mais o recomendado é usar autenticação. Todas elas são consultadas utilizando a biblioteca requests.

A resposta da segunda API consiste em um dicionário com cada linguagem e sua contagem de bytes. Assim, para saber a proporção basta ir unindo essas informações e no final calcular as porcentagens. Para a exibição, é usada uma tabela onde na primeira linha ficam as logos das ferramentas (boa parte das logos vem desse [repositório](https://github.com/abranhe/programming-languages-logos) e na segunda os nome e as porcentagem de código em que são usadas. O código com todas essas funções está [aqui](https://github.com/nymarya/nymarya/blob/master/repositories.py).

### Agora sim a automatização

Todo o processo de escrita no arquivo é feita no arquivo [build_readme.py](https://github.com/nymarya/nymarya/blob/master/build_readme.py). Mas para fazer o preenchimento do arquivo primeiro precisamos indicar no README.md aonde os textos vão ser colocados. Isso é feito com comentários no markdown, que marcam as seções:

```markdown
<!-- <nome da seção> starts -->
<!-- <nome da seção> ends -->
```

Daí, usando a biblioteca `re` podemos usar expressões regulares para identificar esses blocos. A expressão usada é `"<!-- <nome da seção> starts -->.*<!-- <nome da seção> ends -->"`. Na hora de inserir o texto, os comentários se mantém, para na próxima atualização a seção poder ser encontrada novamente, e os caracteres `.*` são substituídos pelo conteúdo de fato, consequentemente também reescrevendo o texto anterior.

O passo final é configurar o workflow no Github Actions. Isso é feito indo para a aba específica, escolhendo "set up a workflow yourself" e então adicionando os comandos para configurar o Python e o pip, instalar dependências que estão no requeirements.txt, atualizar o README.md e finalmente dar commit no resultado, além de ajustar o cron para que a tarefa rode a cada 1 hora, entre 1:00 e 23:00. Ao salvar o `main.yml` que contém as configurações na pasta `.github/workflows`, o GitHub Actions vai identificar o arquivo. O workflow é basicamente o mesmo do Simon, a diferença é que o dele ainda inclui um comando para pegar variáveis de ambiente.

### Para quem tiver mais tempo e disposição

Existe uma série de coisas que poderiam ser automatizadas. Os releases mais recentes (como Simon fez), contagem de commits, interações no StackOverflow (não olhei se eles tem API, se não tiverem certamente o BeautifulSoup resolve), trabalho atual e formação do LinkedIn (se tiverem API deve ser paga, então novamente poderia usar BeautifulSoup), publicações no LinkedIn, posts do Medium, vídeos ou episódios de podcasts mais recentes para quem é desse mundo, e por aí vai.

Ansiosa para ver quais ideias vão surgir. Até mais!

## Links

[Post do Simon onde ele explica o que ele fez](https://simonwillison.net/2020/Jul/10/self-updating-profile-readme/)

[Outro post do Simon sobre automatização do README](https://simonwillison.net/2020/Apr/20/self-rewriting-readme/)
