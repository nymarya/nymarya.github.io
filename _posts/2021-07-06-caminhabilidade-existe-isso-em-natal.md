---
layout: post
title: "Caminhabilidade: existe isso em Natal?"
image: "../images/posts/distances_cmap.png"
categories:
 - ptbr
tags:
 - python
 - pandana
 - mapas
 - caminhabilidade
---

A tecnologia tem um papel primordial na inovação cívica, já que está ajudando a identificar problemas da sociedade para os governos e trazer soluções, além de melhorar o processo de tomadas de decisão. Dentre as aplicações, podemos citar desde o controle de gasto de água na cidade até encorajar a população a engajar mais na política.

Mobilidade urbana é um dos vários aspectos para os quais as autoridades devem atentar. Sabendo que a locomoção pode ser uma fonte substancial de estresse, vemos que o planejamento das vias tem um impacto direto na qualidade de vida das pessoas. Se pensarmos em sustentabilidade, a discussão se torna ainda mais complexa. Surge a necessidade de ciclovias e, mais ainda, a oportunidade dos habitantes circularem pela cidade a pé. Acessibilidade para pedestres tem um impacto considerável na saúde na saúde, além de promover um impacto positivo no meio ambiente.

Sabendo disso, neste post tentaremos medir caminhabilidade, a capacidade de acessar diferentes pontos importantes da cidade a pé. A análise usa dados de Natal, sendo feita através de ciência de dados, dados abertos e mapas. Existem posts parecidos que serviram de inspiração, como esses de [Maceió](https://medium.com/pizzadedados/utilizando-dados-abertos-e-ci%C3%AAncia-de-dados-na-mobilidade-urbana-371a4c591639), [Toronto](https://medium.com/@matthewtenneyto/measuring-walking-times-across-toronto-to-nearest-ttc-stop-using-the-pedestrian-network-and-python-4263e0392a9b), e [New York](https://towardsdatascience.com/measuring-pedestrian-accessibility-97900f9e4d56).

Todo o código necessário está disponível no [repositório](https://github.com/nymarya/fear-the-walking-distance).

## Primeiramente, os dados

O plano aqui é calcular as distâncias a pé dentro de um bairro e comparar as diferenças entre as áreas. Mas antes disso, vamos nos aprofundar um pouco e recolher informações sobre as pessoas. Isso pode ser feito bem rápido, visto que o IBGE provê dados sobre a renda e o número de habitantes pela API [SIDRA](sidra.ibge.gov.br), então a biblioteca `requests` será suficiente. A renda per capita e a população são descritas, respectivamente, pelas variáveis 842 e 841 da tabela 3170.

Agora vem a parte divertida. Como medir a acessibilidade para pedestres em um bairro? Buscando uma alternativa, pensamos nisso: escolher um ponto bem no meio da área a medir a distancia dali até o ponto de interesse mais próximo. Isso significa encontrar o centro, i.e. centróide, do polígono. Para nossa sorte, o GeoJSON que descreve a cidade tem a geometria da região que nos interessa, que consiste em um objeto do tipo Polygon, cujo centróide é um atributo.

O próximo passo é calcular a distâncias em si. Criamos a rede com [Pandana](https://udst.github.io/pandana/network.html), que recupera os dados da [OpenStreetMap](https://wiki.openstreetmap.org/wiki/API) de um jeito muito eficiente. 

A biblioteca também tem a funcionalidade de consultar os pontos de interesse da rede. Neste estudo, a busca é por estabelecimentos que cobrem as necessidades básicas: comida, educação, dinheiro, saúde e segurança. Nos termos da API OSM, as tags são 'hospital', 'restaurant', 'bank','school', 'police', and 'pharmacy'. Para manter esse post mais curto, os exemplos aqui usarão apenas escolas. Por favor, acesse o repositório para o código completo.

```python
bbox = [lat1, lon1, lat2, lon2]
net = osm.pdna_network_from_bbox(*bbox)
schools = osm.node_query(*bbox, tags='"amenity"="school"')
```

Seguindo o fluxo sugerido na [demo](https://github.com/UDST/pandana/blob/dev/examples/Pandana-demo.ipynb) disponibilizada na documentação do Pandana, chamamos `set_pois` para adicionar pontos à rede. Isso é feito separadamente para cada categoria. No exemplo abaixo, passamos os parâmetros de longitude e latitude das escolas, bem como o número de itens que a consulta que a função deve retornar. Aqui, também especificamos que queremos apenas os estabelecimentos dentro de um raio de 10km. O próximo passo usará essa informação.

```
net.set_pois("schools", 10000, 10, schools.lon, schools.lat)
```

Então o cálculo é feito ao chamar  `nearest_pois` e passar a distância máxima e o número de itens que devem ser retornados pela função. O resultado é uma tabela onde cada linha é referente a um nó x e cada coluna i representa o i-ésimo estabelecimento mais próximo, de modo que cada célula contém a distância entre esses dois pontos.

```python
net.nearest_pois(10000, "schools", num_pois=10)
```

<div>
<img src="/images/posts/distances_table.png">
</div>

Finalmente, salvamos no dataframe o valor que está na coluna 1 e na linha referente ao nó do bairro. E pronto, nossa base de dados está montada. 

### Hora de visualizar!

<div>
<center>
<img src="https://media.giphy.com/media/rOTGSPxvJJY7m/giphy.gif">
</center>
</div>

O dataset está pronto, então é hora de usar [Folium](https://github.com/python-visualization/folium), uma bilbioteca capaz de unir manipulação de dados com Python e visualização de mapas com a biblioteca JavaScript [Leaflet](https://github.com/Leaflet/Leaflet). Isso é tudo que uma pessoa programadora precisa para exibir seus dados geográficos.

Antes de prosseguir, é importante mencionar que Pandana possibilita criar visualizações bonitas, incluindo o mapa usado como capa desse post. A função `aggregation` conta quantos estabelecimentos estavam num raio de 10km de cada nó e o resultado é bonito, mas difícil de interpretar. Ainda que exista uma ideia de onde a maioria das escolas está (no meio da cidade, onde os pontos são mais vermelhos), mal dá para reconhecer a cidade, pior ainda se quisessemos comparar os bairros.

<div>
<img src="/images/posts/distances_cmap1.png">
</div>

Bom, vamos para os mapas interativos.

<div>
<center>
<iframe src="{{ site.url }}/assets/htmls/fear_the_walking_distance/population.html" height="400" width="400"></iframe>
<iframe src="{{ site.url }}/assets/htmls/fear_the_walking_distance/income.html" height="400" width="400"></iframe> 
<figcaption>Esquerda: quanto mais escura a cor, maior a população. Direita: quando mais escura a cor, maior a renda per capita</figcaption>   
</center>

</div>

Os mapas mostram que, enquanto a população está mais concentrada no norte, e em seguida na zona ao oeste, a renda segue uma tendência inversa. A renda média aumenta quando seguimos para o sul e é ainda maior se olharmos para o leste, onde estão os bairros com maior renda per capita.

<div>
<center>
<iframe src="{{ site.url }}/assets/htmls/fear_the_walking_distance/school.html" height="350" width="400"></iframe>
<iframe src="{{ site.url }}/assets/htmls/fear_the_walking_distance/restaurant.html" height="350" width="400"></iframe>  
<iframe src="{{ site.url }}/assets/htmls/fear_the_walking_distance/pharmacy.html" height="350" width="400"></iframe>  
<figcaption>Esquerda: distância até a escola mais próxima. Centro: distância até o restaurante mais próximo. Direita: distância até a farmácia mais próxima</figcaption>
</center>
</div>

Quando analisamos a distância que alguém no centro de um bairro tem que se deslocar até a escola, farmácia ou o restaurante mais próximo, vemos um padrão espacial. Bairros com menos acessibilidade, ou seja, coloridos com lilás ou roxo, estão localizados nas áreas ao norte e oeste da cidade. Destaca-se o bairro Nossa Senhora da Apresentação que possui o tom de roxo escuro em todos os três mapas. Lagoa Azul, distrito vizinho, tem caracterícticas semelhantes pois possui a cor mais escura em dois dos três mapas.

<center>
<iframe src="{{ site.url }}/assets/htmls/fear_the_walking_distance/hospital.html" height="350" width="400"></iframe>
<iframe src="{{ site.url }}/assets/htmls/fear_the_walking_distance/police.html" height="350" width="400"></iframe>
<figcaption>Esquerda: distância até o hospital mais próximo. Direita: distância até o posto de polícia mais próximo</figcaption>
</center>

No que diz respeito a hospitais e postos de polícia, há uma diferença significativa entre os dois lados do rio Potengi. Em ambos os mapas, os bairos em roxo estão exclusivamente no norte, o que significa dizer que os habitante dessa área teriam que andar pelo menos 6km para chegar ao hospital ou posto de polícia mais próximo. Temos também alguns bairros na cor azul ao oeste e ao sul, onde a distância fica entre 2km e 6km, o que também pode ser problemático quando imaginamos ter que andar tanto para chegar a um hospital.

<div>
<center>
<iframe src="{{ site.url }}/assets/htmls/fear_the_walking_distance/bank.html" height="380" width="400"></iframe>
<figcaption>Distância até o banco mais próximo</figcaption>
</center>
</div>

A análise das distâncias para bancos distoa um pouco das anteriores. Aqui, os bairros ao oeste possuem cores mais escuras, o que indica distâncias maiores, do que os bairros ao norte.

Note que em todos os mapas os bairros ao leste são coloridos com um azul claro, geralmente isso acontece em escala menor no sul,  mesmo que essa segunda região tenha um desempenho semelhante. Esse padrão condiz com a renda per capita, mais alta no sul e mais ainda em algumas partes da zona ao leste.

### Considerações finais

Dados abertos são uma ferramenta essencial para empoderar os cidadões através da informação, disponibilizando meios de tomar decisões e fazer cobranças às autoridades. No caso do OpenStreetMap, sua única desvantagem é a falta de confiabilidade. Por este motivo, é importante valorizar intituições como IBGE, que não apenas são responsáveis por coletar informação mas também facilitar o acesso dos dados de alta qualidade.

Ainda que não seja possível tirar conclusões mais definitivas, visto que há uma chance dos dados de bairros como Nossa Senhora da Apresentação, Lagoa Azul e Guarapés estarem faltando, a informação disponibilizada nos diz que existe uma falta de acessibilidade nessas áreas. Essa afirmação condiz com as reclamações dos moradores nas redes sociais e nos jornais locais. O fato de que a maioria desses bairros serem populosos implica que muitas pessoas podem sofrer com a falta de investimento. Então, respondendo a pergunta do título: Natal ainda tem um longo caminho pela frente no que diz respeito à acessibilidade para pedestres.

Finalmente, trabalhos futuros incluem explorar mais do Pandana. Há também um interesse em testar outra abordagem para comparar as distâncias. A alternativa seria computar as distâncias de cada nó dentro do bairro para o estabelecimento mais próximo, para então calcular a média para os nós da área e realizar as comparações.

E é isso! Fique à vontade para entrar em contato, fazer perguntas e sugerir novas ideias.

## Links

[Previous version of this study (PT-BR_)](https://medium.com/@mayradazevedo/ate-onde-e-posivel-chegar-a-pe-em-natal-4e50d1e46685)

[Walkability: entenda o que é a caminhabilidade (PT-BR)](https://www.ecycle.com.br/walkability-caminhabilidade/)
