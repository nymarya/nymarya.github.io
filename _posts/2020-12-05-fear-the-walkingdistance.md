---
layout: post
title: "Self-updating portfolio on README"
categories:
 - eng
tags:
 - python
 - pandana
 - maps
 - walkability
---

Technology has a primordial role in civic innovation, as it is helping to diagnose problems from society to the government and bringing solutions, besides improving decision-making. Among the applications, we can think of examples that include controlling waste in the city or encouraging the population to be more politically active.

Urban mobility is one of the various aspects to which authorities must pay attention. Since transportation can be a substantial source of stress, planning has a direct impact on life quality. When thinking of sustainability, this discussion becomes even more complex. It claims more bike lanes and, more importantly, the opportunity for pedestrians to resolve their business in the city by walking.

With that in mind, in this post, we try to measure the walkability in Natal, Brazil, using data science, open data, and maps. There are similar posts that were the source of great inspiration, including one from [Maceió](https://medium.com/pizzadedados/utilizando-dados-abertos-e-ci%C3%AAncia-de-dados-na-mobilidade-urbana-371a4c591639), another Brazilian city, [Toronto](https://medium.com/@matthewtenneyto/measuring-walking-times-across-toronto-to-nearest-ttc-stop-using-the-pedestrian-network-and-python-4263e0392a9b), and [New York](https://towardsdatascience.com/measuring-pedestrian-accessibility-97900f9e4d56).

All the code for the steps below is available in the [repository](https://github.com/nymarya/fear-the-walking-distance).

## Data first

The plan is to measure walking distance within a district and compare different neighborhoods. Before getting into that, we should go deeper and get information about the people there. That's a quick task since IBGE provides data about the income and the number of inhabitants through the [SIDRA API](sidra.ibge.gov.br), so the `requests` library will do all the work here. The per capita income and the number of inhabitants are described, respectively, by the variables 842 and 841 from table number 3170.

Now, the interesting bit. How to measure pedestrian accessibility within a district? Searching for an arbitrary approach, we got to this: picking a point right in the middle of the area and calculating the distance from it to the nearest point of interest. That means finding the center, name centroid, of a polygon. Luckily, the GeoJSON that describes the city has the geometry for the region of interest, consisting of a Polygon object, which has the centroid as an attribute.

The next step is to calculate the distances per se. We create a network with Pandana, which retrieves the data from [OpenStreetMap](https://wiki.openstreetmap.org/wiki/API) in a very efficient way. 

The library also provides a function to query the points of interest in the network. In this study, the search is for amenities that cover basic needs: food, education, money, health, and safety. In OSM API terms, the tags are 'hospital', 'restaurant', 'bank','school', 'police', and 'pharmacy'.

Following the workflow suggested in their [demo](https://github.com/UDST/pandana/blob/dev/examples/Pandana-demo.ipynb), we call `set_pois` to add the points to the network. That is done separately for each category. Here, we already specify that we only want to know the amenities within 10km. The next step will use this information.

```

```



Then, the calculation is done by calling `nearest_pois` and passing the max distance and the number of items that the function must return.

### It's visualization time!

<div>
<center>
<img src="https://media.giphy.com/media/rOTGSPxvJJY7m/giphy.gif">
</center>
</div>

The dataset is ready, so it's time to use [Folium](https://github.com/python-visualization/folium), a library responsible for unifying data manipulation with Python, and maps visualizations with JavaScript's [Leaflet](https://github.com/Leaflet/Leaflet). That is all a programmer needs to show their data when already having geographical data in their hands.

<div>
<center>
<iframe src="{{ site.url }}/assets/htmls/fear_the_walking_distance/population.html" height="400" width="400"></iframe>
<iframe src="{{ site.url }}/assets/htmls/fear_the_walking_distance/income.html" height="400" width="400"></iframe> 
<figcaption>Left: the darker the color, the greater the population. Right: The darker the color, the bigger the mean income</figcaption>   
</center>

</div>

The maps show that whereas the population is more concentrated in the north, followed by the west zone, the income has an inverse tendency. The mean income increases as we go in the south direction and even more when looking east, where the districts with higher incomes are.

<div>
<center>
<iframe src="{{ site.url }}/assets/htmls/fear_the_walking_distance/school.html" height="350" width="400"></iframe>
<iframe src="{{ site.url }}/assets/htmls/fear_the_walking_distance/restaurant.html" height="350" width="400"></iframe>  
<iframe src="{{ site.url }}/assets/htmls/fear_the_walking_distance/pharmacy.html" height="350" width="400"></iframe>  
</center>
</div>

There's a pattern when seeing the distance that someone in the middle of a neighborhood would have to walk to the nearest school, pharmacy, or restaurant. Districts with less accessibility, meaning colored in light or dark purple, are in the north and west areas, highlighting the Nossa Senhora da Apresentação district, colored with the darker purple in all three maps. Lagoa Azul, the neighboring region, shows similar _performance_, having the darker color in two out of three maps.

<center>
<iframe src="{{ site.url }}/assets/htmls/fear_the_walking_distance/hospital.html" height="350" width="400"></iframe>
<iframe src="{{ site.url }}/assets/htmls/fear_the_walking_distance/police.html" height="350" width="400"></iframe>
</center>

Regarding hospitals and police stations, there's a significant difference between the two sides of the river. In both maps, purple-colored districts are exclusively in the north, meaning that the inhabitants usually would have to walk at least 6km to get to the nearest hospital or police stations. There are some blue-painted districts in the west and the south, where the distance is between 2km and 6km and can be problematic if we imagine having to walk this much to get to a hospital.

<div>
<center>
<iframe src="{{ site.url }}/assets/htmls/fear_the_walking_distance/bank.html" height="380" width="400"></iframe>
</center>
</div>

The analysis of distances to banks is a little different from the previous ones. Here, the western districts have darker colors, indicating greater distances, than the northern districts.

Note that in all maps, the districts in the east have a light blue coloring, usually on a smaller scale than in the south, although the second region has a similar performance. This pattern matches the income, higher in the south and ever more distinguished in some eastern neighborhoods.

### Final thoughts

Open data is an essential tool to empower citizens through information, providing ways to make decisions and ask authorities. In the OpenStreet Map case, its only fault is regarding reliability. That's why it's so important to celebrate institutes such as IBGE, which not only are responsible for collecting information but also facilitate access to high-quality data.

Even though it's not possible to draw hard conclusions because there's a chance that the data from districts like Nossa Senhora da Apresentação, Lagoa Azul, and Guarapés are missing, the available information tells us that there's a lack of accessibility in these areas. This affirmation corroborates with the claims of the habitants in social media and local news. The fact that most of these neighborhoods are also populous implies that many people may suffer from the scarcity of investment.

Finally, future works include exploring more of Pandana. Also, there's interest in testing another approach to compare the distances. The alternative would be calculating the distances from each node from the neighborhood to the nearest amenity and then calculate the mean.

## Links

[Previous version of this study (PT-BR_)](https://medium.com/@mayradazevedo/ate-onde-e-posivel-chegar-a-pe-em-natal-4e50d1e46685)
