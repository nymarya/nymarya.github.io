---
layout: post
title: "Portfólio automatizado no README"
categories:
  - pt-br
tags:
  - python
  - github actions
  - readme
---

No dia 10 de julho de 2020, estava eu calmamente navegando pela minha timeline do Twiter, quando me aparece o tweet abaixo:

<div>
<blockquote class="twitter-tweet"><p lang="en" dir="ltr">Made myself a self-updating GitHub personal README! It uses a GitHub Action to update itself with my latest GitHub releases, blog entries and TILs <a href="https://t.co/Eve7FOrwYK">https://t.co/Eve7FOrwYK</a> <a href="https://t.co/oJPXLtFdgM">pic.twitter.com/oJPXLtFdgM</a></p>&mdash; Simon Willison (@simonw) <a href="https://twitter.com/simonw/status/1281435464474324993?ref_src=twsrc%5Etfw">July 10, 2020</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>
</div>

Para um desenvolvedor sua existência já está resumida na própria bio do GitHub, na página [about](https://nymarya.github.io) do site pessoal, na descrição do LinkedIn, às vezes até na descrição do Twitter, e com a possibilidade de automatizar. Depois da invenção do Simon, a feature nova pareceu mais interessante, podendo dar um diferencial em relação às outras de poder trazer informações sintetizadas que não precisam ser ajustadas manualmente (e consequentemente esquecidas de serem atualizadas).

### Começando a automatização README

O primeiro passo é escolher quais informações devem aparecer. No meu caso, queria mostrar os posts e um resumo do que tem no meu GitHub, as linguagens que estão presentes nele.

O próprio resumo escolhi escrever manualmente por motivos de: queria usar emoticons; queria escrever em inglês e português; algumas informações são pessoais, como os hobbies, e não encontraria lugares de onde tirar a informação; e até poderia criar um script que simplemente acrescenta o texto que eu escolhi, mas não acho que ficaria um código bonito e organizado.

Pensado isso, é hora de colocar a mão no código.

### Inserindo posts com BeautifulSoup

Simon em seu [post](https://simonwillison.net/2020/Jul/10/self-updating-profile-readme/) usa uma forma mais robusta e organizada para recuperar os posts, mas infelizmente meu blog no Github Pages não me dá as mesmas ferramentas. Porém, como a estrutura do HTML é bem organizada, não foi difícil recuperar o que eu queria.

A biblioteca [BeautifulSoup](https://pypi.org/project/beautifulsoup4/) é ótima para minerar informações de sites. Para pegar os posts separados por cada língua, recuperei o HTML da [página](https://nymarya.github.io/categories) onde os posts já estão divididos, identifiquei onde fica cada categoria, e simplemente salvei o endereço, título e data do número de links que eu queria. O código todo, que não dá nem 30 linhas) está [aqui](https://github.com/nymarya/nymarya/blob/master/posts.py).

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

O passo final é configurar o workflow no Github Actions. Isso é feito indo para a aba específica, escolhendo "set up a workflow yourself" e então adicionando os comandos para configurar o Python e o pip, instalar dependências que estão no requeirements.txt, atualizar o README.md e finalmente dar commit no resultado, além de ajustar o cron para que a tarefa rode a cada 1 hora. O workflow é basicamente o mesmo do Simon, a diferença é que o dele ainda inclui um comando para pegar variáveis de ambiente.

### Para quem tiver mais tempo e disposição

Existe uma série de coisas que poderiam ser automatizadas. Os releases mais recentes (como Simon fez), contagem de commits, interações no StackOverflow (não olhei se eles tem API, se não tiverem certamente o BeautifulSoup resolve), trabalho atual e formação do LinkedIn (se tiverem API deve ser paga, então novamente poderia usar BeautifulSoup), publicações no LinkedIn, posts do Medium, vídeos ou episódios de podcasts mais recentes para quem é desse mundo, e por aí vai.

Ansiosa para ver quais ideias vão surgir. Até mais!

## Links

[Post do Simon onde ele explica o que ele fez](https://simonwillison.net/2020/Jul/10/self-updating-profile-readme/)

[Outro post do Simon sobre automatização do README](https://simonwillison.net/2020/Apr/20/self-rewriting-readme/)
