## Fatal Automobile Accidents in the GTA

Given this data, we ask a few questions, which will be addressed below.

### What time is it most unsafe to drive?

First, let's import our libraries and our dataset.

Input:
```markdown
library(readr)
library(dplyr)
library(ggplot2)

data <- read.csv("Fatal_Collisions.csv")
```

Looking through the data, I removed two rows with null/wrong data.

Input:
```markdown
data <- data %>% 
  filter(District %in% c("North York", 
                         "Etobicoke York", 
                         "Scarborough",
                         "Toronto and East York"))
```


Now, let's look at the data

``` markdown
data %>%
  group_by(YEAR) %>%
  summarize (n=n()) %>%
  arrange(desc(YEAR))
```

From here, we we get a tibble shown below:


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

We see that accidents usually happen during daylight or dark, but never in between. What about hour?

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

We get the following line chart:

![GTA-fatal-accidents_hourly-graph](https://ky-feng.github.io/data-projects/GTA-fatal-accidents_hourly-graph.png)

I suppose the best time for a walk/drive is approximately 4am, and the worst time for a walk or a drive is approximately 6pm! This coincides with GTA rush hour, which, according to is from . I suppose a few differences are .....

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

We definitely see that the data seems too align with our hypothesis. The later months, in autumn and winter, are prone to more accidents than spring or summer. So this data shows us the results. It's a little confusing to look at though. What about a graph?

``` markdown
ggplot(aes(x=as.numeric(month), y=n), data=months) + 
  geom_line(size=1) + 
  ggtitle('Fatal Accidents by Month in Toronto from 2008-2018') +
  ylab('Number') +
  xlab('Month') +
  scale_x_continuous(breaks=c(1,2,3,4,5,6,7,8,9,10,11,12))
```
![GTA-fatal-accidents_monthly-graph](https://ky-feng.github.io/data-projects/GTA-fatal-accidents_monthly-graph.png)

What surprised me the most about this data is the sharp drop in February. I still consider February a winter month - perhaps something has changed then?


Lastly, let's just look, for fun, at the relationship between fatal accidents in GTA districts and months.

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

![GTA-fatal-accidents_district-month-heatmap](https://ky-feng.github.io/data-projects/GTA-fatal-accidents_district-month-heatmap.png)

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
