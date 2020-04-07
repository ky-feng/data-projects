## Fatal Automobile Accidents in the GTA

Given this data, we ask a few questions, which will be addressed below.

### What time is it most unsafe to drive?

Markdown is a lightweight and easy-to-use syntax for styling your writing. It includes conventions for

```markdown

library(readr)
library(dplyr)
library(ggplot2)

data <- read.csv("Fatal_Collisions.csv")

data %>%
  group_by(YEAR) %>%
  summarize (n=n()) %>%
  arrange(desc(n))
  
data %>%
  group_by(LIGHT) %>%
  summarize (n=n()) %>%
  arrange(desc(n))
 
hours <- data %>%
  group_by(Hour) %>%
  summarize (n=n()) %>%
  arrange(desc(n))
  
ggplot(aes(x=Hour, y=n), data =hours) + 
  geom_line(size=1) + 
  ggtitle('Fatal Accidents by Hour in Toronto from 2008-2018') +
  ylab('Number') +
  xlab('Hour')

# Header 1
## Header 2
### Header 3

- Bulleted
- List

1. Numbered
2. List

**Bold** and _Italic_ and `Code` text

[Link](url) and ![Image](src)
```

For more details see [GitHub Flavored Markdown](https://guides.github.com/features/mastering-markdown/).

### Jekyll Themes

Your Pages site will use the layout and styles from the Jekyll theme you have selected in your [repository settings](https://github.com/ky-feng/miniature-octo-spork/settings). The name of this theme is saved in the Jekyll `_config.yml` configuration file.

### Support or Contact

Having trouble with Pages? Check out our [documentation](https://help.github.com/categories/github-pages-basics/) or [contact support](https://github.com/contact) and we’ll help you sort it out.
