---
title: How to choose a satisfying board game? Part II
author: James Diu
date: '2020-07-16'
slug: how-to-choose-a-satisfying-board-game-part-ii
categories:
  - boardgame
  - R
tags: []
backtotop: no
toc: no
draft: TRUE
---





```r
games_Jan2018 <- read_csv("data/games_Jan2018.csv")
df <- games_Jan2018
# These columns should have missing value instead of 0, replace with NA
list <- c("minplayers", "maxplayers", "playingtime","minplaytime", "maxplaytime")
df[list][df[list]==0]<-NA

# Update data by visiting BGG for accurate information
df[df$id==234768,]$minplayers <- 2

df[df$id==235285,]$minplayers <- 2
df[df$id==235285,]$maxplayers <- 6

df[df$id==75294,]$minplayers <- 2
df[df$id==75294,]$maxplayers <- 20

df<- df %>% 
  # filter games with BGG rating
  filter(users_rated>=30) %>% 
  mutate(
    # assign expansion as 1, boardgame as 0
    type = ifelse(type == "boardgame", 0, 1),
    # switch the value for those minplaytime > maxplaytime 
    maxplaytime = ifelse(maxplaytime<minplaytime, minplaytime, maxplaytime),
    minplaytime = ifelse(minplaytime<playingtime, minplaytime, playingtime),
    # create age group
    age_group = ifelse(minage>14, "15+",
                       ifelse(minage>11, "12-14",
                              ifelse(minage>8, "9-11",
                                     ifelse(minage>5, "6-8","<6")))),
    age_group = factor(age_group, levels = c("<6", "6-8", "9-11","12-14","15+")),
    # create duration group
    duration = ifelse(playingtime < 30, "Instant",
                      ifelse(playingtime < 60, "Short",
                             ifelse(playingtime <120, "Normal",
                                    ifelse(playingtime < 180, "Long",
                                           ifelse(is.na(playingtime),
                                                  NA,"Extreme"))))),
    duration = factor(duration, 
                      levels = c("Instant", "Short", "Normal", "Long", "Extreme"))
    ) %>%
  # remove useless columns
  select(id, type, name, yearpublished, age_group, duration, 
         weight = average_weight, users_rated, BGG_rating = bayes_average_rating,
         types, categories, mechanics, designers)
```



```r
mec_tags <- cSplit(df %>% select(id, mechanics), 
                   splitCols = "mechanics",
                   direction = "long")
length(unique(mec_tags$mechanics))
```

```
## [1] 52
```

```
## 
## Attaching package: 'scales'
```

```
## The following object is masked from 'package:purrr':
## 
##     discard
```

```
## The following object is masked from 'package:readr':
## 
##     col_factor
```

<img src="/post/2020-07-16-how-to-choose-a-satisfying-board-game-part-ii.en_files/figure-html/fig1_nmech_pie-1.png" width="672" />

<img src="/post/2020-07-16-how-to-choose-a-satisfying-board-game-part-ii.en_files/figure-html/fig2_nmech_treemap-1.png" width="672" />


<img src="/post/2020-07-16-how-to-choose-a-satisfying-board-game-part-ii.en_files/figure-html/fig3_mechrate_pie-1.png" width="672" />



```r
mec_dummy <- dcast(mec_tags, id ~ mechanics, function(x) 1, fill = 0) %>%
  left_join(df[c("id","BGG_rating")], by = "id")
```

```
## Using 'mechanics' as value column. Use 'value.var' to override
```

```r
fit <- lm(BGG_rating~.,data = mec_dummy %>% select(-id) %>%
            filter(BGG_rating != 0))
fits <- as.data.frame(fit$coefficients)
fits$mechanics <- as.factor(rownames(fits))
colnames(fits) <- c("coef","mechanics")
```



```r
ggplot(fits[-1,] %>% filter(mechanics != "`NA`"), aes(x=reorder(mechanics,coef),y=coef,fill=coef))+
  geom_bar(stat="identity")+
  coord_flip()+
  labs(title="Linear Model Coefficients for mechanics against rating",
       y="Coefficients",x="Mechanic")+
  scale_fill_gradient(low="red", high="green")+
  theme_light() +
  theme(axis.text.y = element_text(size = 6),
        axis.title.y = element_blank())
```

<img src="/post/2020-07-16-how-to-choose-a-satisfying-board-game-part-ii.en_files/figure-html/unnamed-chunk-4-1.png" width="672" />

```r
fits
```

```
##                                         coef                       mechanics
## (Intercept)                      5.570618259                     (Intercept)
## `NA`                            -0.008499225                            `NA`
## Acting                          -0.017803760                          Acting
## `Action / Movement Programming`  0.045482066 `Action / Movement Programming`
## `Action Point Allowance System`  0.005971358 `Action Point Allowance System`
## `Area Control / Area Influence`  0.150351170 `Area Control / Area Influence`
## `Area Enclosure`                 0.109018663                `Area Enclosure`
## `Area Movement`                  0.071971643                 `Area Movement`
## `Area-Impulse`                   0.053162734                  `Area-Impulse`
## `Auction/Bidding`                0.084446260               `Auction/Bidding`
## `Betting/Wagering`              -0.007666051              `Betting/Wagering`
## `Campaign / Battle Card Driven`  0.103620930 `Campaign / Battle Card Driven`
## `Card Drafting`                  0.123457469                 `Card Drafting`
## `Chit-Pull System`               0.019659200              `Chit-Pull System`
## `Co-operative Play`              0.062316943             `Co-operative Play`
## `Commodity Speculation`          0.044994027         `Commodity Speculation`
## `Crayon Rail System`             0.113243184            `Crayon Rail System`
## `Deck / Pool Building`           0.139559237          `Deck / Pool Building`
## `Dice Rolling`                   0.041841409                  `Dice Rolling`
## `Grid Movement`                  0.139045703                 `Grid Movement`
## `Hand Management`                0.092694341               `Hand Management`
## `Hex-and-Counter`                0.018098481               `Hex-and-Counter`
## `Line Drawing`                   0.075106648                  `Line Drawing`
## Memory                           0.013373754                          Memory
## `Modular Board`                  0.047118016                 `Modular Board`
## `Paper-and-Pencil`               0.005938252              `Paper-and-Pencil`
## Partnerships                     0.089924526                    Partnerships
## `Pattern Building`               0.002086264              `Pattern Building`
## `Pattern Recognition`           -0.019596308           `Pattern Recognition`
## `Pick-up and Deliver`            0.023104476           `Pick-up and Deliver`
## `Player Elimination`             0.076151768            `Player Elimination`
## `Point to Point Movement`        0.083991741       `Point to Point Movement`
## `Press Your Luck`                0.071058775               `Press Your Luck`
## `Rock-Paper-Scissors`           -0.093185987           `Rock-Paper-Scissors`
## `Role Playing`                   0.072277027                  `Role Playing`
## `Roll / Spin and Move`          -0.104947170          `Roll / Spin and Move`
## `Route/Network Building`         0.225455278        `Route/Network Building`
## `Secret Unit Deployment`         0.043797548        `Secret Unit Deployment`
## `Set Collection`                 0.077365532                `Set Collection`
## Simulation                       0.029785253                      Simulation
## `Simultaneous Action Selection`  0.075205066 `Simultaneous Action Selection`
## Singing                         -0.096644165                         Singing
## `Stock Holding`                 -0.016545811                 `Stock Holding`
## Storytelling                     0.060643875                    Storytelling
## `Take That`                     -0.037557221                     `Take That`
## `Tile Placement`                 0.079421844                `Tile Placement`
## `Time Track`                     0.115744877                    `Time Track`
## Trading                         -0.020727650                         Trading
## `Trick-taking`                   0.030833380                  `Trick-taking`
## `Variable Phase Order`           0.125502478          `Variable Phase Order`
## `Variable Player Powers`         0.095615103        `Variable Player Powers`
## Voting                           0.084719771                          Voting
## `Worker Placement`               0.236811816              `Worker Placement`
```

