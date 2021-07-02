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

With that in mind, in this post, we try to measure the walkability in Natal, Brazil, using data science, open data, and maps. There are similar posts that were the source of great inspiration, such as one from Maceió, another Brazilian city, Toronto, and NY.

All the code for the steps below is available in the [repository](https://github.com/nymarya/fear-the-walking-distance).

## Data first

The plan is to measure walking distance within a district and compare different neighborhoods. Before getting into that, we should go deeper and get information about the people there. That's a quick task since IBGE provides data about the income and the number of inhabitants through the SIDRA API, so the `requests` library will do all the work here.

Now, the interesting bit. How to measure pedestrian accessibility within a district? Searching for an arbitrary approach, we got to this: picking a point right in the middle of the area and calculating the distance from it to the nearest point of interest. That means finding the center, name centroid, of a polygon. Luckily, the GeoJSON that describes the city has the geometry for the region of interest, consisting of a Polygon object, which has the centroid as an attribute.

The next step is to calculate the distances per se. We create a network with Pandana, which retrieves the data from OpenStreetMap in a very efficient way. 

The library also provides a function to query the points of interest in the network. In this study, the search is for amenities that cover basic needs: food, education, money, health, and safety. In OSM API terms, the tags are 'hospital', 'restaurant', 'bank','school', 'police', and 'pharmacy'.

Following the workflow suggested in their [demo](https://github.com/UDST/pandana/blob/dev/examples/Pandana-demo.ipynb), we call `set_pois` to add the points to the network. That is done separately for each category. Here, we already specify that we only want to know the amenities within 10km. The next step will use this information.

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
<iframe src="{{ site.url }}/assets/htmls/fear_the_walking_distance/school.html" height="380" width="400"></iframe>
<iframe src="{{ site.url }}/assets/htmls/fear_the_walking_distance/restaurant.html" height="380" width="400"></iframe>  
<iframe src="{{ site.url }}/assets/htmls/fear_the_walking_distance/pharmacy.html" height="380" width="400"></iframe>  
</center>
</div>

<center>
<iframe src="{{ site.url }}/assets/htmls/fear_the_walking_distance/hospital.html" height="380" width="400"></iframe>
<iframe src="{{ site.url }}/assets/htmls/fear_the_walking_distance/police.html" height="380" width="400"></iframe>
</center>

<div>
<center>
<iframe src="{{ site.url }}/assets/htmls/fear_the_walking_distance/bank.html" height="380" width="400"></iframe>
</center>
</div>



## Links

[Simon's post where he explains it all](https://simonwillison.net/2020/Jul/10/self-updating-profile-readme/)

[Yet another Simon's post on README automation](https://simonwillison.net/2020/Apr/20/self-rewriting-readme/)
