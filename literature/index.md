In progress...

When I was younger, I was an avid reader... I'd always wanted to be a novelist (who am I kidding, I still do), and a good work of prose gets me every time, so much that I wanted to mimic it. But what exactly makes a piece of literature literary? And how do great writers, like F. Scott Fitzgerald, create works of art that defies 

In my introductory neuro course, my professor spoke of the nebulous relationship of mind and matter - how is it that words on a page evoke profound emotions? How do I, as an author, provoke the same wistfulness that underlies F. Scott Fitzgerald's novels; or encapsulate the same __ of vengeance of __?

For fun, I started this project. I wanted to analyze works of literature that had great prose. I also wanted to analyze works of literature that constituted as "literary."

## Who are the great novelists who write poetic prose?

### Who are the great novelists?

Throughout my searching, I discovered a list, dubbed "1001 Books You Must Read Before You Die." I decided to use this list, which many have posted on blogs. I specifically used the core list from this site: http://bucketlistbookreviews.com/the-lists/differences-between-the-original-and-current-1001-lists/

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

The list I get from Jupyter is kinda long, so I'll just show you the first 15 lines of the 731 I got from 2012. Going through the list, I see familiar names: Haruki Murakami, Jennifer Egan, Thomas Pynchon, Jonathan Swift... Now where to get their works of literature?

### What novelists write "poetic" prose?

I often frequent a subreddit, "ProsePorn," which features excerpts from great authors. I figured that if I scraped that subreddit, I'd be able to find the authors whose works consitute as poetic. If i were to cross-reference these novelists with the list above, I'd be able to find literary authors iwth poetic prose.

First, to scrape reddit, I learned from this site: https://www.storybench.org/how-to-scrape-reddit-with-python/

```markdown
import praw
import pandas as pd
import string
import csv

reddit = praw.Reddit(client_id='X-C8vFgxE4SPOg', \
                     client_secret='PH0DQXrJBkFWJOnho760D09n6G0', \
                     user_agent='proseporn', \
                     username='x-halcyon', \
                     password='12345x3')

subreddit = reddit.subreddit('proseporn')

#top_subreddit = subreddit.top()
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

This gets me a lot of whack data that needs cleaning... here are the first few lines of the data i received:

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

How do we cross-reference this data with the great list of authors? I decided to throw the last names of authors from 1001 novels into a list, and then cross reference every line from reddit's title posts.

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

what do we get? we get a list of authors who wrote novels that made it to the 1001  list, and who wrote poetic prose. Here are some results:

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
Honestly, not surprised at the results. McCarthy's the only real modern author on the list, joined by many great novelists: nabokov, Fitzgerald, Faulkner, Joyce. Now that we have a list of some authors we want to analyze, let's find their literature.

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

some of my output/csv:
IMAGE

Hoestly, a lot of my variables got cut off. Now that we have our csv, we can get going on the data...

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

The graph seems to demonstrate a negative correlation, though not very strong. Let's check just to make sure:

```
> cor(novels$prop_uniquewords,novels$words)
[1] -0.4180739
```

Definitely a correlation, though not terribly strong. Let's just say moderate. Let's plot this and analyze using linear regression.

```
set.seed(123)
trainingData <- novels[sample(1:nrow(novels), 0.8*nrow(novels)), ]  
testData  <- novels[-trainingRowIndex, ]

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

So what do we see? 
- We see that our p value of 0.00311 is statistically significant at less than 0.05, demonstrating that prop_uniquewords is a good addition to our model.We reject our null hypothesis that our estimate should be 0, and accept the value of -719310 as our slope.
- However, our R-squared value of 0.1568 is close to 0 - the variability of our independent variable (words) seems to have very little to do with our dependent variable (prop_uniquewords), meaning our model isn't very good. 
- REgardless, there seems to be a correlatin between our two variables - F-statistic has a p-value less than 0.05, demonstrating a relationshipp between our variables.

##