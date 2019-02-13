---
layout: post
title: "Teste de hipótese"
categories:
  - Edge Case
tags:
  - estatistica
  - matematica
  - reductio ao absurdum
---

## O absurdo no teste de hipótese

### Teste de hipótese

Uma das parte mais mágicas da Estatística é ser capaz de fazer inferência sobre alguma característica de uma população, exergar padrões em certos comportamentos e assim conseguir tomar as decisões de forma consciente. Por exemplo, sabemos que o algoritmo *quick sort* tem uma complexidade de O($n^2$) quando o vetor já está ordenado. Mas será que mesmo assim vale a pena esse algoritmo no sistema? É comum que o algoritmo entre no pior caso, se escolhermos o pivot aleatoriamente?  Ao fazer a análise de complexidade média, vemos que a complexidade mais comum seria de O(n log n).

No entanto, alguns casos de estudo não são tão previsíveis como um algoritmo conhecido nem é possível fazer uma análise considerando a totalidade dos dados. Imagine descobrir as chances de um candidato ganhar, sabendo que é inviável entrevistar todos os eleitores num período próximo das eleições.

Vê-se então a utilidade da **inferência estatística**, que baseia-se em, a partir de uma amostra, fazer uma afirmação em relação a algum parâmetro (média, por exemplo) envolvendo toda a população. No entanto, ao formular uma hipótese estatística é preciso em posse de algum mecanismo que ofereça certa garantia de que a dedução feita ao analisar uma pequena parte do todo também vale no mundo real.

Uma das formas de obter essa resposta é através do **teste de hipótese**, cujo conceito foi forjado por [Ronald Fisher](https://en.wikipedia.org/wiki/Ronald_Fisher), considerado por alguns o pai da estatística moderna. São formuladas duas hipóteses: $H_0$, também conhecida como *hipótese nula*; e $H_1$, que recebe o nome de *hipótese alternativa* deve ser complementar à hipótese nula. O funcionamento consiste em testar $H_0$, buscando saber se ela é ou não aceitável e assumindo que $H_0$ é válida.

![blue_screen](/home/mayra/workspace/nymarya.github.io/images/posts/blue_screen.png)

Testar uma ideia supondo que ela é verdadeira e ainda por cima sem ter todos os dados? Parece contraditório. 

### Prova por contradição

O argumento lógico por trás desse método é o _reductio ao absurdum_. Ao supor uma proposição, é desenvolvido o pensamento até que o resultado seja algo tão absurdo que invalide a suposição inicial.

O método de prova por contradição usa desse argumento. É suposto que um evento B não acontece. A partir desse pressuposto é inferido que um evento A não acontece, donde concluisse que A acontecer implica em B acontecer.

Para o teste de hipótese, a linha de raciocínio começa assumindo que a hipótese nula é válida. Ao analisar os dados disponíveis é obtido o parâmetro.  Com base em um *nível de significância*, é visto o quão o parâmetro se distancia de um valor razoável de acordo com um modelo, ou seja, se esse valor seria muito raro (um absurdo) de se obter caso a hipótese nula fosse verdade. Se for o caso, é bem provável que nossa tese esteja errada, e podemos dizer que a rejeitamos.

### Atribuindo valor às variáveis

Enumerados os termos usados, um exemplo ajuda a visualizar melhor como os dois mundos se unem.

Digamos que nós queremos estudar o comportamento de acessos a um site que uma certa equipe é responsável. Depois de algum tempo, com base no retorno obtido das redes sociais, um dos integrantes acha que o percentual de usuário que possuem graduação é em média 60%, com 10 % de desvio padrão. Alguém da mesma equipe discorda, achando que a proporção deve ser maior. 

Para saber quem está mais próximo do resultado real, façamos com que $H_0$ represente a hipótese de que em média 60% dos usuários que acessam o sistema possuem graduação $H_1$, a afirmação que o percentual é maior que 60%.

Existem dois modos de verificar se a hipótese nula deve ser rejeitada, depois de escolhido o nível de significância $\alpha$, que geralmente é em torno de 0,05 e 0,1. Uma é usando _p_-valor, onde o _p_-valor menor que o nível de significância implica em rejeitar $H_0$. Outra é calculando a região crítica, ou regra de decisão, que define um intervalo para o parâmetro $\bar X$, no qual $H_0$ será rejeitado. Ao calcular o parâmetro, é testado ele está contido na região crítica e, se for o caso, a tese é rejeitada.

Vamos supor que ao fazer uma simulação dos usuários, foi obtido histograma abaixo e que ao calcular o _p_-valor, seu valor é de 0.03:

![simulation_site_access](/home/mayra/workspace/nymarya.github.io/images/posts/simulation_site_access.png)

Neste caso, $H_0$ é rejeitado, ou seja, podemos dizer que há evidências de que o percentual médio dos usuários que acessam o sistema possuem graduação é maior do que 60%.

## Links

[Aula do Khan Academy sobre testes de significância](https://www.khanacademy.org/math/ap-statistics/tests-significance-ap/idea-significance-tests)

[Discussão sobre reductio ad absurdum e prova por contradição](https://philosophy.stackexchange.com/questions/561/what-is-the-difference-between-reductio-ad-absurdum-and-proof-by-contradiction)

[Verbete sobre hipótese nula](https://en.wikipedia.org/wiki/Null_hypothesis#Goals_of_null_hypothesis_tests)

[Vídeo-aula sobre teste de hipótese do Crash Course Statistics](https://www.youtube.com/watch?v=bf3egy7TQ2Q&t=0s&list=PL8dPuuaLjXtNM_Y-bUAhblSAdWRnmBUcr&index=23)
