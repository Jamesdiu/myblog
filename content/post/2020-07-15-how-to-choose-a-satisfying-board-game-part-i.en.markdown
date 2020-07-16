---
title: How to choose a satisfying board game? Part I
author: James Diu
date: '2020-07-15'
slug: how-to-choose-a-satisfying-board-game-part-i
categories:
  - R
  - boardgame
tags: []
backtotop: no
toc: no
---

{{< figure src="/post/2020-07-15-how-to-choose-a-satisfying-board-game-part-i.en_files/catan_GOT.png" title=""  width="600" >}}

Being a board game enthusiast, I always introduce board games to my friends. Some of them might be another enthusiast, while some might have an idea that the board game is equivalent to chess. Most of the time I use my instinct and my understanding of the taste of my friends to decide which game would provide the greatest fun for us. However, it can be very biased due to my personal preference. On the other hand, I love to collect *different* board games. I hope my collection can be diverse enough to serve all kinds of my friends' backgrounds and preferences. So how should I choose a game to meet general preference? And which game should I select, such that it will enrich my collection and make sure it could be fun? (It is quite upset when you find out a new game is boring...) Learning more about the general preference statistically would be a nice solution.

### Data description
Regarding the data of board games, [BoardGameGeek(BGG)](https://boardgamegeek.com/) is undoubtedly the biggest source. However, I am not going to pull the data directly. Instead, I will use the data from an interesting [post](https://dvatvani.github.io/BGG-Analysis-Part-1.html) about the trend and rating bias on BGG in 2018, which also come from BGG. Although it is not up-to-date data, I think it can still give us some insights and suggestions.


```r
games_Jan2018 <- read_csv("data/games_Jan2018.csv")
df <- games_Jan2018
```


In this dataset, we have total 95,777 board games, including the 80,105 base games and 15,672 expansion. The information can also be separated into 2 groups: games features and responses on BGG. After removing some duplicated or reduntant information, below are the useful columns: 

Features: `type`, `yearpublished`, `minplayers`, `maxplayers`, `playingtime`, `minage`, `types`, `categories`, `mechanics`, `desginers` 

Responses: `users_rated`, `bayes_average_rating`, `average_weight`

One of the most important data is `bayes_average_rating`, which is only available for the games with at least 30 users' ratings. It is used instead of the average player rating, which you normally see on the BGG when you search for any games, in order to exclude those games with biased rating due to small sample size. After filtering out those data less than 30 ratings, we still have 15,026 games and 4,714 expansion and we will place our focus on them. Apart from `bayes_average_rating`, it is necessary to define `average_weight`. From BGG, there is no explicit defination for weight. From my observation, I would assume `average_weight` as the complexity of a game. Although it is a subjective index given by the users and it is hard to obtain this data for those new games, I think this information would be still useful for us to determine if a complex game is more popular.

```r
df %>% filter(users_rated>=30) %>% group_by(type) %>% summarise(n=n())
```

```
## # A tibble: 2 x 2
##    type     n
##   <dbl> <int>
## 1     0 15026
## 2     1  4714
```

```r
df <- df %>% filter(users_rated>=30)
```

Now we have a rough idea about what cards are on our hand. Let's find out more from them. In the beginning, we start with the numerical data.

### Correlation - The rating has positive correlation with weight & minimum age

Fig.1 is the correlation table among different numerical features. We can see the BGG rating is positivly correlated with the average weight and minimum age. Meanwhile, weight and age also show positive correlation so means they have some colinearity. Considering that Besides, the rating is somehow independent of the number of players and the duration of playing time, but both features show some dependence on both features. Therefore we will go through all these features one by one.

```r
corrplot(cor(
  df %>% select(BGG_rating = bayes_average_rating,
                avg_weight = average_weight,
                yearpublished, minage,
                minplayers, maxplayers, playingtime),
  use = "pairwise.complete.obs" # igrone missing pairs of data
  ),
  method = "circle", type ="lower", tl.cex = 1, tl.srt = 0.1,
  title = "Fig.1: Correlation between numerical data",
  mar=c(0,0,2,0))
```

<div class="figure">
<img src="/post/2020-07-15-how-to-choose-a-satisfying-board-game-part-i.en_files/figure-html/fig1-1.png" alt="A fancy pie chart." width="672" />
<p class="caption">Figure 1: A fancy pie chart.</p>
</div>

{{< figure src="/post/2020-07-15-how-to-choose-a-satisfying-board-game-part-i.en_files/fig1-1.png" title="" width="800">}}

### Weight - Less complicated game is less popular
It obvious that the weight of games has positive skewness (fig. 2), with the mode at 2. Meanwhile, we should be reminded that this is a score provided by the users and it is a relative value to other games. 
<img src="/post/2020-07-15-how-to-choose-a-satisfying-board-game-part-i.en_files/figure-html/unnamed-chunk-3-1.png" width="672" />

{{< figure src="/post/2020-07-15-how-to-choose-a-satisfying-board-game-part-i.en_files/unnamed-chunk-3-1.png" title="" width="600">}}

From fig.3, we can see the rating moves slightly upwards when the complexity increase. One of the most interesting thing is no games with an average weight higher than 2 has a rating below 5, while all games rated above 8 also belong to this group.
<img src="/post/2020-07-15-how-to-choose-a-satisfying-board-game-part-i.en_files/figure-html/unnamed-chunk-4-1.png" width="672" />

{{< figure src="/post/2020-07-15-how-to-choose-a-satisfying-board-game-part-i.en_files/unnamed-chunk-5-1.png" title="" width="600">}}

### Year of publish - New published games has better rating in average
Since some of the games on the list do not have a clear publish date, only games published on or after 1985, which have already cover more than 90% of the games, would be kept for this feature. The rest will be set into *NA* as irrelevant due to the small sample size.

```r
sort(df$yearpublished)[round(nrow(df)/10)]
```

```
## [1] 1985
```

Fig.4 shows that the average BGG rating raises slightly over the years. However, it is hard to conclude whether the games published are getting better or the baseline of users rating is going upwards. Also the highest rating and rating range are going higher and wider among the years. Details can also be found in Fig.6.

<img src="/post/2020-07-15-how-to-choose-a-satisfying-board-game-part-i.en_files/figure-html/unnamed-chunk-6-1.png" width="672" />

{{< figure src="/post/2020-07-15-how-to-choose-a-satisfying-board-game-part-i.en_files/unnamed-chunk-6-1.png" title="" width="600">}}

The number of publish increase exponentially in recent years. It is also a strong evidence that the marketing is expanding very fast and why the rating range is getting larger. There are fewer pulished recorded after 2015. It is possibly because the time lag between publishing and reaching to the players. Considering the market provides more than 1,000 games in the past few years, it does take some time to get enough users to rate, especially not all players would rate for the games they tried.

```r
ggplot(df %>% filter(yearpublished >=1985) %>% group_by(yearpublished) %>% summarise(n=n())) +
  geom_point(aes(x=yearpublished, y=n, color = as.factor(yearpublished))) +
  theme(legend.position = "none") +
  scale_x_continuous(name = "")+
  scale_y_continuous(name = "")+
  ggtitle("Fig.5: Number of game published along year (user rated >=30)")
```

<img src="/post/2020-07-15-how-to-choose-a-satisfying-board-game-part-i.en_files/figure-html/fig5-1.png" width="672" />

{{< figure src="/post/2020-07-15-how-to-choose-a-satisfying-board-game-part-i.en_files/unnamed-chunk-7-1.png" title="" width="600">}}

<img src="/post/2020-07-15-how-to-choose-a-satisfying-board-game-part-i.en_files/figure-html/unnamed-chunk-7-1.png" width="672" />

{{< figure src="/post/2020-07-15-how-to-choose-a-satisfying-board-game-part-i.en_files/unnamed-chunk-9-1.png" title="" width="600">}}

### Minimum age - minimum age from 12-14 has the highest rating

We can know that the minimum age is positively correlated to weight and rating from the correlation chart before. It is reasonable because most of the BGG users are not children and I have sure they are not that interested in those games for children. This fact also implies the evaluation of children's games may be invalid since the focus group is different. Besides, the minimum age mainly falls into 5 ages: 8, 10, 12, 13 & 14 and the distribution is very inbalance. In general, the purpose of minimum age is an index for the complexity and content However, the judgment depends highly on the publisher (noted that games specially for 18+ are rare), therefore I would also prefer to transfer minimum age into 5 groups: <6, 6-8, 9-11, 12-14, 15+.


```r
df <- df %>% mutate(age_group = ifelse(minage>14, "15+",
                                        ifelse(minage>11, "12-14",
                                               ifelse(minage>8, "9-11",
                                                      ifelse(minage>5, "6-8","<6")))),
                     age_group = factor(age_group, levels = c("<6", "6-8", "9-11","12-14","15+")))
```
<img src="/post/2020-07-15-how-to-choose-a-satisfying-board-game-part-i.en_files/figure-html/unnamed-chunk-9-1.png" width="672" />

{{< figure src="/post/2020-07-15-how-to-choose-a-satisfying-board-game-part-i.en_files/unnamed-chunk-10-1.png" title="" width="600">}}

According to the plot, it is easy to see that <6 & 15+ has a lower rating than other. It reviews that the correlation between minimum age and rating may not be linear.

<img src="/post/2020-07-15-how-to-choose-a-satisfying-board-game-part-i.en_files/figure-html/unnamed-chunk-10-1.png" width="672" />

{{< figure src="/post/2020-07-15-how-to-choose-a-satisfying-board-game-part-i.en_files/unnamed-chunk-12-1.png" title="" width="600">}}

### Number of players - Most are designed for 2-4 players. Irrelevant to rating.

There are 1,039 games (~5%) designed for more than 10 players.

```r
nrow(df[df$maxplayers>10,"maxplayers"])
```

```
## [1] 1039
```
It is quite suprise to find out that quite a number of games allowing more than 100 players, 3 of them even 1000.  

```r
table(df[df$maxplayers>10,"maxplayers"])
```

```
## 
##  11  12  13  14  15  16  17  18  20  21  22  24  25  28  30  31  32  33  34  36 
##   6 213   7   9  36  56   5  15  51   3   4  13   1   2  46   1   2   2   1   9 
##  38  40  41  42  45  47  48  50  52  61  64  68  75  99 100 127 200 362 999 
##   1   3   1   1   1   1   1   4   2   1   1   4   3 103   8   1   1   1   3
```
Besides, most of the games are designed for 2, 2-4 or 2-6 players without any surprise.

<img src="/post/2020-07-15-how-to-choose-a-satisfying-board-game-part-i.en_files/figure-html/unnamed-chunk-13-1.png" width="672" />

{{< figure src="/post/2020-07-15-how-to-choose-a-satisfying-board-game-part-i.en_files/unnamed-chunk-14-1.png" title="" width="600">}}

It doesn't seem to have a relationship between minimum and maximum players.
<img src="/post/2020-07-15-how-to-choose-a-satisfying-board-game-part-i.en_files/figure-html/unnamed-chunk-14-1.png" width="672" />

{{< figure src="/post/2020-07-15-how-to-choose-a-satisfying-board-game-part-i.en_files/unnamed-chunk-16-1.png" title="" width="600">}}

### Playing time - Longer seems to be better

In most of the cases, the playing time is longer than its stated. Therefore, we will only consider `playingtime`, which is the same as `maxplaytime` but more completed, as the playing time of a game. Similar to the number of players, there are some extreme value in playing time. Around 10% of them lasts longer than 2 hours and the maximum playing time on the list (**The Campaign for North Africa**) is 1000 hours, which means more than a month! I am also very surprise that more than 100 users has rated for this game. Another unbelievable fact is: the minimum players for this game is 8!

```r
nrow(df %>% filter(playingtime > 120))
```

```
## [1] 2183
```

```r
head(df %>% arrange(desc(playingtime)) %>%
       select(type, name, minplayers, playingtime, BGG_rating = bayes_average_rating, users_rated), 5)
```

```
## # A tibble: 5 x 6
##    type name                       minplayers playingtime BGG_rating users_rated
##   <dbl> <chr>                           <dbl>       <dbl>      <dbl>       <dbl>
## 1     0 The Campaign for North Af…          8       60000       5.48         116
## 2     0 Case Blue                           1       22500       5.96         261
## 3     0 1914: Offensive à outrance          2       17280       5.66          81
## 4     0 Atlantic Wall: D-Day to F…          2       14400       5.57          40
## 5     0 Empires in Arms                     2       12000       6.46        1096
```

As stated playing time is only an estimation of actual time, it is better and simplier to group them together (Fig. 11):

|Group|Duration|
|:---|:---|
|Instant | less than 30|
|Short | 30 - 59 min|
|Normal | 60 - 119 min|
|Long | 120-179 min|
|Extreme |180 min or longer|



```r
 df <- df %>% mutate(duration = ifelse(playingtime < 30, "Instant",
                           ifelse(playingtime < 60, "Short",
                                  ifelse(playingtime <120, "Normal",
                                         ifelse(playingtime < 180, "Long", 
                                                ifelse(is.na(playingtime), NA, "Extreme"))))),
         duration = factor(duration, levels = c("Instant", "Short", "Normal", "Long", "Extreme")))
duration_group <- df %>% group_by(duration) %>% summarise(n=n())
```
<img src="/post/2020-07-15-how-to-choose-a-satisfying-board-game-part-i.en_files/figure-html/unnamed-chunk-18-1.png" width="672" />

{{< figure src="/post/2020-07-15-how-to-choose-a-satisfying-board-game-part-i.en_files/unnamed-chunk-19-1.png" title="" width="600">}}

Fig.12 shows us that the rating has a similar effect as age group: rating increases when the duration is longer and seems to reduce when the duration is extreme. From the relation with weight we know that complicated game has a higher rating and it is hard to create a complicated one within a short playing time. Meanwhile, like watching a movie, it is more possible to get bored when it is extreme long. 
<img src="/post/2020-07-15-how-to-choose-a-satisfying-board-game-part-i.en_files/figure-html/unnamed-chunk-19-1.png" width="672" />

{{< figure src="/post/2020-07-15-how-to-choose-a-satisfying-board-game-part-i.en_files/unnamed-chunk-21-1.png" title="" width="600">}}

