# Analysis of Literariness and Poeticism in Novels

I have always been in avid reader - as a young girl, I spent my recesses wandering around my school library, my nose buried in some book. When I grew older, I realized the imporance of literary work. Literary novelists are scribes - they share knowledge of human experience, through which we share in human emotions. But how is it possible that lines on a page are transformed into emotion? Why is it that I read Fitzgerald's work, and feel wistfulness; or anger, when reading Sylvia Plath's poetry?

It's an interesting matter, subjective experience - non-reductionists would argue that it's impossible for hard data to truly detect wihch novels will make us cry, or which would transform our lives. However, I believe that data is important to some understanding of books. I'm willing to try to find out.

For fun, I started this project. I wanted to analyze novels that were "literary" and that had "poetic prose." I wanted to know what exactly it was about authors like Fitzgerald and Nabokov that I enjoyed reading. I also wanted to improve my own short stories, and see how I measure up.

It's still a work in progress, but here goes...

## What is literariness? What is poetic prose?

### What novels are considered literary?

Throughout my searching, I discovered a list, dubbed "1001 Books You Must Read Before You Die." I decided to use this list to measure literariness, especially this site: http://bucketlistbookreviews.com/the-lists/differences-between-the-original-and-current-1001-lists/

Let's save the list into a txt, and transform it into a csv.

Python code:
```markdown
import requests
import pandas as pd

file = open('1001books2012.txt','r', encoding='utf-8')
books = file.readlines()

authors=[]

for book in books:
    book = book.strip()
    loc = book.strip().find("~")
    author = book[loc+1:len(book)].strip()
    if author not in authors:
        authors.append(author)
        
authors={'author':authors}
authors=pd.DataFrame(authors)
display(authors)
authors.to_csv('authors.csv', index=False, header=False)
```

The list I get from Jupyter is kinda long, so I'll just show you the first 15 lines of the 731 I got from 2012.

![authorlist - 1001.png](https://ky-feng.github.io/data-projects/literature/authorlist - 1001.png)

Going through the list, I see familiar names: Haruki Murakami, Jennifer Egan, Thomas Pynchon, Jonathan Swift. So many novelists, with so many novels, but not all of them write "poetically" (especially not Hemingway). So how exactly do I measure poeticism?

### What novelists write "poetic" prose?

I often frequent a subreddit, "ProsePorn," which features excerpts from great authors. I figured that if I scraped that subreddit, I'd be able to find the authors whose works consitute as poetic. If i were to cross-reference these novelists with the list above, I'd be able to find literary authors iwth poetic prose.

First, to scrape reddit, I learned from this site: https://www.storybench.org/how-to-scrape-reddit-with-python/

```markdown
import praw
import pandas as pd
import string
import csv

#removed information here for privacy
reddit = praw.Reddit(client_id='~insert_id~', \
                     client_secret='~insert_code~', \
                     user_agent='proseporn', \
                     username='~insert_username~', \
                     password='~insert_password~')

subreddit = reddit.subreddit('proseporn')

top_subreddit = subreddit.top(limit=3000)

topics_dict = { "title":[], \
                "score":[], \
                "id":[], "url":[], \
                "comms_num": [], \
                "created": [], \
                "body":[]}

for submission in top_subreddit:
    topics_dict["title"].append(submission.title)
    topics_dict["score"].append(submission.score)
    topics_dict["id"].append(submission.id)
    topics_dict["url"].append(submission.url)
    topics_dict["comms_num"].append(submission.num_comments)
    topics_dict["created"].append(submission.created)
    topics_dict["body"].append(submission.selftext)
    
    
topics_data = pd.DataFrame(topics_dict)

row_titles = []

for index,row in topics_data.iterrows():
    row_titles.append(row['title'])
```

This gets me a lot of ugly data that needs cleaning. Here are the first few lines of the data for reference:

```markdown
Anagram Like A Boss [x-post from /r/pics]
Catch 22
No Country for Old Men - Cormac McCarthy
Notes From the Underground - Dostoevsky
Dr. Martin Luther King, Jr. - Letter from a Birmingham Jail
This Is How You Lose Her -- Junot Díaz
C.S. Lewis - The Problem of Pain
How To Change Your Mind by Michael Pollan
For Whom the Bell Tolls, Ernest Hemingway
Stephen Fry - "The Hippopotamus"
Either/Or:Part I by S. Kierkegaard
```

How do we cross-reference this data with the great list of authors? I decided to throw the last names of authors from 1001 novels into a list, and then cross reference these authors with every line from Reddit's title posts.

```markdown
#created the list of authors
authors_1001 = pd.read_csv("authors.csv") 
authors_1001 = authors_1001.values.tolist()
authors=[]
for author in authors_1001:
    for x in author:
        authors.append(x)
        
#compare to reddit
author_list = []
count_list = []

row_noauthor=[]

temp_list=[]

print (len(row_titles))

for author in authors:
    nws_author=str(author).replace(" ","").lower()    
    for title in row_titles:
        nws_title = title.replace(" ","").lower()

        if nws_author in nws_title:
            if author not in author_list:
                author_list.append(author)
                count_list.append(1)
            else:
                count_list[author_list.index(author)] += 1
            row_titles.remove(title)

author_counter = {'author':author_list,
                 'count':count_list}

pd.DataFrame(author_counter).sort_values('count',ascending=False)
```

What do we get? We get a list of authors who write "poetically" and who wrote novels that made it to the 1001 list. Here are the results:

```markdown

      author	count
23	Cormac McCarthy	73
6	Thomas Pynchon	60
92	James Joyce	28
58	Vladimir Nabokov	22
15	Don DeLillo	19
17	David Foster Wallace	19
105	Virginia Woolf	18
93	John Steinbeck	18
127	Herman Melville	17
98	William Faulkner	17
101	F. Scott Fitzgerald	14
56	Kurt Vonnegut	12
83	Samuel Beckett	10
85	Albert Camus	9
44	Italo Calvino	8
1	Philip Roth	8
82	Ernest Hemingway	8
```

Honestly, not surprised at the results (except for Hemingway). McCarthy's the only real modern author on the list, joined by many novelists renowned for their mastery of the English language: Nabokov, Fitzgerald, Faulkner, Joyce. Now that we have a list of some authors we want to analyze, let's find their literature.

## Finding Novels and their Data

After relentlessly hunting through the interwebs for txt versions of great novels, I found 47 novels to analyze. from these 47 novels, I derived certain variables...

- novel name
- author
- word count
- proportion of unique words in text
- proportion of length of words in text

The code for how I cleaned the data:

```markdown
import pandas as pd
import os
import csv

#create list of file names
files = [f for f in os.listdir('.') if os.path.isfile(f)]
novels = []

for f in files:
    length = len(f)
    if f[length-3:length] == 'txt':
        novels.append(f)

#create lists of individual items
titles=[]
authors=[]
prop_eachword=[]
words=[]

#create lists of dictionaries
prop_uniquewords = []

for novel in novels:
    file = open(novel,'r', encoding = 'latin-1')
    search = file.readlines()
    
    #fill titles/authors lists
    for line in search:
        if line.__contains__('Title:'):
            titles.append(line[7:].strip())
        if line.__contains__('Author:'):
            authors.append(line[8:].strip())

    #reset file
    file=open(novel,'r', encoding = 'latin-1')
    
    list_words = file.read().split()
    num_total_words = len(list_words)
    set_unique_words = set(list_words)
    num_unique_words = len(set_unique_words)
    sorted_words= sorted(set_unique_words, key=len)

    length_words = []

    for word in sorted(list_words):
        length_words.append(len(word))
        
        
    #list of dictionaries
    
    #entire novel sorted in words
    dict_wordcount_proportions={}
    for each in set(length_words):
        #this is just for readability
        if each < 10:
            dict_wordcount_proportions['wl_0'+ str(each)] = length_words.count(each)/len(length_words)
        else:
            dict_wordcount_proportions['wl_'+ str(each)] = length_words.count(each)/len(length_words)
    
    #APPEND to LISTS
    prop_uniquewords.append(num_unique_words/len(list_words))
    prop_eachword.append(dict_wordcount_proportions)
    words.append(num_total_words)
    
data_csv = {'novel':titles,
            'author':authors,
            'words':words,
            'prop_uniquewords': prop_uniquewords,
           }

#concat the two dataframes, replace all NA values with 0 (these are wordlengths that do not exist in the novel)
data = pd.concat([pd.DataFrame(data_csv),pd.DataFrame(prop_eachword)],axis=1).fillna(0)

#data.sort_values(by=['author','prop_uniquewords'])
data.sort_values(by=['words'])

data.to_csv(r'novels.csv')

display(data)
```

Some of my output/CSV:
![list_workingnovels.png](https://ky-feng.github.io/data-projects/literature/list_workingnovels.png)

A lot of my variables got cut off. But now that we have our csv, we can get going on the data...

## Visualizing the Data

For this, we go to R.

First, let's import our dataset and library.

```markdown
library(readr)
library(ggplot2)
library(dplyr)
library(tidyr)
library(zoom)
novels <- read_csv("novels.csv")
```

Now, let's look at how many novels of each author I have in my small dataset.

```markdown
novels %>% group_by (author) %>% summarize(n=n()) %>% arrange(desc(n))
```

```markdown
# A tibble: 18 x 2
   author                          n
   <chr>                       <int>
 1 Jane Austen                     6
 2 Virginia Woolf                  6
 3 F. Scott Fitzgerald             4
 4 Mark Twain (Samuel Clemens)     4
 5 William Faulkner                4
 6 George Eliot                    3
 7 James Joyce                     3
 8 Lewis, C. S.                    3
 9 Anne Bronte                     2
10 Cormac McCarthy                 2
11 Edith Wharton                   2
12 Ernest Hemingway                2
13 Nathaniel Hawthorne             2
14 Daphne DuMaurier                1
15 Herman Melville                 1
16 Oscar Wilde                     1
17 Plath, Sylvia                   1
18 Vladimir Nabokov                1
```

What about a graph?

```markdown

ggplot(novels,aes(words,prop_uniquewords)) +
  geom_point(data=novels,aes(colour=author)) + 
  scale_x_continuous(labels = comma)

```

![plot_wordspropunique.png](https://ky-feng.github.io/data-projects/literature/plot_wordspropunique.png)

The graph seems to demonstrate a negative correlation, though not very strong. Let's look at the data, and run some analysis for fun.

## Data Analysis

### Correlation

```
> cor(novels$prop_uniquewords,novels$words)
[1] -0.4180739
```

Definitely a correlation, though not terribly strong. Let's just say moderate. Let's plot this and analyze using linear regression.

### Linear Regression

We break up our data set using the Pareoto Principle (80-20 rule).

```
set.seed(123)
trainingData <- novels[sample(1:nrow(novels), 0.8*nrow(novels)), ]  
testData  <- novels[-sample(1:nrow(novels), 0.8*nrow(novels)), ]

lmNovels <- lm(words ~ prop_uniquewords, data=novels)
prediction <- predict(lmNovels, testData) 

summary(lmMod)
```
We get the following output:
```

Call:
lm(formula = words ~ prop_uniquewords, data = novels)

Residuals:
   Min     1Q Median     3Q    Max 
-93769 -31208 -11000  18560 183147 

Coefficients:
                 Estimate Std. Error t value Pr(>|t|)    
(Intercept)        216207      36091   5.991 2.99e-07 ***
prop_uniquewords  -719310     230445  -3.121  0.00311 ** 
---
Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1

Residual standard error: 56120 on 46 degrees of freedom
Multiple R-squared:  0.1748,	Adjusted R-squared:  0.1568 
F-statistic: 9.743 on 1 and 46 DF,  p-value: 0.003108
```

Equation of linear regression: `y = -719310 * x + 216207`

So what do we see? 
- Pr(>|t|) value of 0.00311 is statistically significant at less than 0.05, demonstrating that prop_uniquewords is a good addition to our model. We reject our null hypothesis that our prop_uniquewords estimate should be 0, and accept the value of -719310 as our slope.
- R-squared value of 0.1568 is close to 0 - the variability of our independent variable (words) seems to have very little to do with our dependent variable (prop_uniquewords), meaning our model is not good.
- F-statistic has a p-value less than 0.05. We reject the null hypothesis that the intercept-only fit is equivalent to our present model.

### K-Means Clustering

First, let's extract the two columns we want to use in our clustering algorithm.

```
library(dplyr)

data_kmeans <- novels %>%
  select(prop_uniquewords,words)
 
 data_kmeans
```

```
# A tibble: 48 x 2
   prop_uniquewords  words
              <dbl>  <dbl>
 1            0.143 190723
 2            0.168  69463
 3            0.151  78935
 4            0.134 136293
 5            0.119 166185
 6            0.101 159540
 7            0.119  58737
 8            0.108 121544
 9            0.162  52634
10            0.230  54512
# ... with 38 more rows
```

Given the large difference between values of prop_uniquewords and words, we should scale our data.

```
data_kmeans <- data_kmeans %>%
mutate(prop_uniquewords = scale(prop_uniquewords), 
    words = scale(words))
```

```
# A tibble: 48 x 2
   prop_uniquewords  words
              <dbl>  <dbl>
 1          -0.279   1.38 
 2           0.446  -0.605
 3          -0.0339 -0.450
 4          -0.526   0.489
 5          -0.951   0.978
 6          -1.44    0.869
 7          -0.957  -0.780
 8          -1.27    0.247
 9           0.274  -0.880
10           2.19   -0.849
# ... with 38 more rows
```

Now, to determine the number of clusters...

```
wss = 0

for (i in 1:10) {
  km.out <- kmeans(data_kmeans, centers = i, nstart = 20)
  wss[i] <- km.out$tot.withinss
}

plot(1:10, wss, type = "b", 
     xlab = "Number of Clusters", 
     ylab = "Within groups sum of squares")
```

![plot_clusters.png](https://ky-feng.github.io/data-projects/literature/plot_clusters.png)

We therefore choose 2 to be our number of clusters.

```
results_kmeans <- kmeans(data_kmeans,2)
results_kmeans
```

```
K-means clustering with 2 clusters of sizes 18, 30

Cluster means:
  prop_uniquewords      words
1       -0.8290527  1.0063013
2        0.4974316 -0.6037808

Clustering vector:
 [1] 1 2 2 1 1 1 2 1 2 2 1 2 2 2 1 2 2 2 2 2 2 1 1 1 1 2 2 1 2 2 2 2 2 2 1 1 1
[38] 2 2 1 1 2 2 2 2 2 2 1

Within cluster sum of squares by cluster:
[1] 21.49706 23.54379
 (between_SS / total_SS =  52.1 %)

Available components:

[1] "cluster"      "centers"      "totss"        "withinss"     "tot.withinss"
[6] "betweenss"    "size"         "iter"         "ifault"      
```

We see  2 clusters of size 18 and 30. Our scaled values demonstrate that our first cluster has greater words and lower proportion of uniquewords, while our second cluster has fewer words but greater proportion of unique words. Our within cluster sum of squares, at 52.1%, demonstrates that there is quite a lot of variation in the mean - our clusters won't be neat and tidy when we plot them.

```
library(cluster)
clusplot(data_kmeans, 
         results_kmeans$cluster)
```

![plot_kmeans.png](https://ky-feng.github.io/data-projects/literature/plot_kmeans.png)
