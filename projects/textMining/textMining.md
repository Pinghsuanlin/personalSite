# Text Mining
Text Mining = Data Mingin + Natural Language Processing

## Sentiment Analysis: 

Steps:

### 1. Get Text 

### 2. Explore and Prepare Text 
```
#Distribution of Rating
library(ggplot2); library(ggthemes)
ggplot(data=df,aes(x=rating))+
  geom_histogram(fill='sienna3')+
  theme_bw()+
  scale_x_reverse()+
  xlab('Review Rating')+
  coord_flip()
```

(1) Count the number of characters: 
```
nchar(df$review[617])
```
*character is every single token*

(2) Count the number of words: *The definition of a word is conveyed to the computer as a pattern.*
```
library(stringr)
str_count(string = df$review[617],pattern = '\\S+')

#or use strsplit()
str_count(strsplit(df$review[617], split = ' '))
```
* *' \ \S+' : 'S' as space; '+' as the thing after the space*

(3) Count the number of sentences:
The definition of a sentence is encoded as a regular expression. 
```
str_count(string = df$review[617],pattern = "[A-Za-z,;'\"\\s]+[^.!?]*[.?!]")
```
* Regular expressions (regex) is a framework for teaching a computer how to recognize patterns of text. 
* Can also use spicy package here.

**Is longer review related with rating?**
```
r_characters = cor.test(nchar(df$review),df$review_rating)

r_words = cor.test(str_count(string = df$review,pattern = '\\S+'),df$review_rating)

r_sentences = cor.test(str_count(string = df$review,pattern = "[A-Za-z,;'\"\\s]+[^.!?]*[.?!]"),df$review_rating)

correlations = data.frame(r = c(r_characters$estimate, r_words$estimate, r_sentences$estimate),p_value=c(r_characters$p.value, r_words$p.value, r_sentences$p.value))
rownames(correlations) = c('Characters','Words','Sentences')

correlations
```
![cor](cor.PNG)
From the table, we see that the relation is quite small, while p-value is significant. So, we could say that there's no relationship between length of review and rating.

(4) Detect Upper Case
```
percentUpper = 100*str_count(df$review,pattern='[A-Z]')/nchar(df$review)
summary(percentUpper)
```

(5) Detect Keyword: Examine if keyword 'well facilities'
```
library(stringr)
mean(str_detect(string = tolower(df$review),pattern = 'well facilities'))*100
```

(6) Look for common words: 
```
library(dplyr); library(tidytext); library(magrittr)

#Remove the top 25 stopwords like prepositions and articles using anti-join and tidytext::stop_words

df%>%
  unnest_tokens(input = review, output = word)%>%
  select(word)%>%
  anti_join(stop_words)%>%
  group_by(word)%>%
  summarize(count = n())%>%
  ungroup()%>%
  arrange(desc(count))%>%
  top_n(25)
```

### 3. Tokenize: Words are examined independent of their position in text. `tidytext::unnest_tokens`
(Break a text, character sequence, document into n word token)
```
library(dplyr); library(tidytext)
df %>%
  select(id,review)%>%
  group_by(id)%>%
  unnest_tokens(output = word,input=review)%>%
  ungroup()%>%
  group_by(id)%>%
  summarize(count = n())
  ```
  * The default token is word, but output can also be as ngram, characters, or sentences.

### 4. Categorize tokens using a lexicon
*Lexicon: Dictionary of words*

(1) Binary (positive/negative)

`tidytext:: get_sentiments('bing')`
`dplyr: inner_join()`
```
tidytext:: unnest_tokens(output = word, input = review ) %>%
inner_join(get_sentiments( 'bing' )) %>%
group_by (sentiment)
```

(2) Emotion

(3) Sentiment score

(4) Text Filters


### 5. Summarize results: Topic Modeling, Latent Semantic Analysis, Text Clustering and Document Clusterin