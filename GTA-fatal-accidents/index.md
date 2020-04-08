## Visualizing Fatal Automobile Accidents in the GTA

dataset: http://data.torontopolice.on.ca/datasets?q=fatal%20collisions

Given this data, we ask a few questions, which we will attempt to visually address below.

### What time is it most unsafe to drive?

First, let's import our libraries and our dataset.

```markdown
library(readr)
library(dplyr)
library(ggplot2)

data <- read.csv("Fatal_Collisions.csv")
```

Looking through the data, I found 2 rows with null and incorrect district data. Given the large size of our data set, I removed the two rows using the code below:

```markdown
data <- data %>% 
  filter(District %in% c("North York", 
                         "Etobicoke York", 
                         "Scarborough",
                         "Toronto and East York"))
```


Now, let's look at the data!

``` markdown
data %>%
  group_by(YEAR) %>%
  summarize (n=n()) %>%
  arrange(desc(YEAR))
```

``` markdown
# A tibble: 11 x 2
    YEAR     n
   <int> <int>
 1  2018    66
 2  2017    63
 3  2016    77
 4  2015    65
 5  2014    50
 6  2013    63
 7  2012    44
 8  2011    35
 9  2010    43
10  2009    48
11  2008    54
```

The data seems pretty evenly dispersed across all years. What about other variables?

``` markdown
data %>%
  group_by(LIGHT) %>%
  summarize (n=n()) %>%
  arrange(desc(n))
 ```
 
 ``` markdown
# A tibble: 8 x 2
  LIGHT                    n
  <fct>                <int>
1 Daylight               317
2 Dark                   139
3 Dark, artificial       122
4 Dawn, artificial         7
5 Dusk                     7
6 Dusk, artificial         7
7 Daylight, artificial     5
8 Dawn                     4
```

We see that accidents happen with almost equal frequency during daylight or dark, but rarely in between. What about hour?

```
hours <- data %>%
  group_by(Hour) %>%
  summarize (n=n()) %>%
  arrange(desc(n))
  
ggplot(aes(x=Hour, y=n), data =hours) + 
  geom_line(size=1) + 
  ggtitle('Fatal Accidents by Hour in Toronto from 2008-2018') +
  ylab('Number') +
  xlab('Hour')
```

![GTA-fatal-accidents_hourly-graph](https://ky-feng.github.io/data-projects/GTA-fatal-accidents/hourly-graph.png)

I, like many others, enjoy late-night walks or drives to relax after a day's work. I suppose, given the above graph, that the safest time to be out (and not be fatally hit by a car) is approximately 4am, and the worst time for a walk or a drive is approximately 6pm. This coincides with GTA rush hour, which can be stressful and terrifying to drive in.

So now, what about date? Or more specifically, month? We assume that more accidents will happen in the winter months. Does the data confirm our hypothesis? Let's check to make sure.

The data given to us however, provides us with date, and not the month. We'll have to parse the string and extract our data to visualize it in R.

``` markdown

months <- data %>%
  group_by(DATE)

months <- months %>% 
  mutate(month=substr(DATE,6,7)) %>%
  group_by(month) %>%
  summarize(n=n()) %>%
  arrange(desc(n))
  
months
```

``` markdown
# A tibble: 12 x 2
   month     n
   <chr> <int>
 1 09       64
 2 07       61
 3 10       60
 4 11       58
 5 01       56
 6 08       56
 7 12       50
 8 05       48
 9 06       42
10 03       41
11 04       39
12 02       33
```

We definitely see that the data seems to align with our hypothesis. The later months, in autumn and winter, are prone to more accidents than months in spring or summer. 

So this data shows us the results. It's a little confusing to look at though. What about a graph?

``` markdown
ggplot(aes(x=as.numeric(month), y=n), data=months) + 
  geom_line(size=1) + 
  ggtitle('Fatal Accidents by Month in Toronto from 2008-2018') +
  ylab('Number') +
  xlab('Month') +
  scale_x_continuous(breaks=c(1,2,3,4,5,6,7,8,9,10,11,12))
```
![GTA-fatal-accidents_monthly-graph](https://ky-feng.github.io/data-projects/GTA-fatal-accidents/monthly-graph.png)

What surprised me the most about this data is the sharp drop in February. I still consider February a winter month - perhaps something has changed then?

Let's just look, for fun, at the relationship between fatal accidents in GTA districts and months.

``` markdown
months <- data %>%
  mutate(month=substr(DATE,6,7)) %>%
  group_by(District,month) %>%
  summarize(n=n())

ggplot(months, aes(District, month, fill = n)) +
  geom_tile(color = "black") +
  geom_text(aes(label=n), color='white') +
  ggtitle("Fatal Crash by District and Month") +
  xlab('District')
```

![GTA-fatal-accidents_district-month-heatmap](https://ky-feng.github.io/data-projects/GTA-fatal-accidents/district-month-tile-heatmap.png)

We see from our heatmap that Scaborough and Toronto and East York have a greater share of accidents, especially in the winter months. This corresponds with the numbers we see from analyzing the data:

``` markdown
data %>%
  group_by(District) %>%
  summarize (n=n()) %>%
  arrange(desc(n))
```

``` markdown
# A tibble: 4 x 2
  District                  n
  <fct>                 <int>
1 Toronto and East York   182
2 Scarborough             173
3 Etobicoke York          130
4 North York              123
```


### Where do fatal collisions most often occur?

Let's re-import our libraries and datasets, and clean our data.

```markdown
library(readr)
library(dplyr)
library(ggplot2)

data <- read.csv("Fatal_Collisions.csv")

data <- data %>% 
  filter(District %in% c("North York", 
                         "Etobicoke York", 
                         "Scarborough",
                         "Toronto and East York"))
```

We found previously that, in terms of Toronto districts, Toronto/East York and Scarborough have a lot more accidents than Etibocoke York/North York. But while we looked at districts, we failed to include neighborhoods. let's check that out now:

```markdown
data %>%
  group_by(Neighbourh,District=District) %>%
  summarise (n=n()) %>%
  arrange (desc(n))
```
```markdown
# A tibble: 143 x 3
# Groups:   Neighbourh [128]
   Neighbourh                          District                n
   <fct>                               <fct>               <int>
 1 West Humber-Clairville (1)          Etobicoke York         22
 2 South Parkdale (85)                 Toronto and East Y…    18
 3 Clairlea-Birchmount (120)           Scarborough            16
 4 Waterfront Communities-The Island … Toronto and East Y…    14
 5 Malvern (132)                       Scarborough            12
 6 Steeles (116)                       Scarborough            12
 7 Wexford/Maryvale (119)              Scarborough            12
 8 Islington-City Centre West (14)     Etobicoke York         11
 9 Moss Park (73)                      Toronto and East Y…    11
10 Clanton Park (33)                   North York             10
# ... with 133 more rows
```
That's a lot of rows! Upon closer inspection, we realize that there are many neighborhoods with very few accidents relative to other neighborhoods. We want to look at neighborhoods with the highest concentration of automobile fatalities. Let's look at neighborhoods with fatalities greater than 5...

```markdown
data %>%
  group_by(Neighbourh,District=District) %>%
  summarise (n=n()) %>%
  arrange (desc(n)) %>%
  filter (n >= 5)

ggplot(df,aes(x=District,y=n,fill=Neighbourh))+
  geom_bar(stat="identity",position="dodge")+
  scale_fill_discrete(name="Neighbourhoods",
                      breaks=c(),
                      labels=c())+
  xlab("Districts")+ylab("Count")
```

![GTA-fatal-accidents_district-neighbourh](https://ky-feng.github.io/data-projects/GTA-fatal-accidents/district-neighbourh.png)

We see that Etobicoke York has one neighborhood - West Humber-Clairville (1) - that has the highest accident fatality rate in all of the GTA in our dataset. That's scary! What we also notice is that this neighborhood is an exception - other than West Humber-Clairville, Etobicoke York has the fewest number of neighborhoods with accidents greater than or equal to five.

In comparison, Scarborough and Toronto/East York have many more bars, and thus many more neighborhoods with automobile fatalities. This does not necessarily mean that these districts have worse roads - perhaps population density is greater in Scarborough and Toronto/East York. Perhaps certain intersections are more congested...

For fun, let's look at a heat map of the GTA region:

```
library(leaflet)
library(leaflet.extras)
library(leaflet.providers)

leaflet(data) %>%
  addProviderTiles(providers$CartoDB.DarkMatter) %>%
  addHeatmap(
    lng = ~X, lat = ~Y, blur = 15, max = 0.02, radius = 12
  )
```

We look at the images, and then zoom in to check it out.

![GTA-fatal-accidents_heat-map-1](https://ky-feng.github.io/data-projects/GTA-fatal-accidents/heat-map-1.png)
![GTA-fatal-accidents_heat-map-2](https://ky-feng.github.io/data-projects/GTA-fatal-accidents/heat-map-2.png)

From far away, we can't really tell much from our heat map. But when we zoom in, we start seeing hotspots. Hard to tell from this image, but the zones (when zoomed in and scrolled) reveal that downtown Toronto has some of the highest rates of automobile deaths, as does major roads like Gardiner Expressway (near Parkdale) and Yonge Street.
