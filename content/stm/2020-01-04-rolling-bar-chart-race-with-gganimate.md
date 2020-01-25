---
title: Rolling Bar Chart Race with gganimate
author: Martin
date: '2020-01-04'
draft: true
slug: rolling-bar-chart-race-with-gganimate
categories:
  - R
  - gganimate
  - stm
tags: []
description: ''
featured: headup.jpg
featuredalt: headup picture
featuredpath: date
linktitle: ''
type: post
---

# Rolling Bar Chart Race

Diese Art der dynamischen Darstellung schien mir gut geeignet zu sein, um die Veränderungen unter den 
<span style="font-weight:bold; font-size: 1.2em;">Topx</span> im Laufe einer Saison darzustellen. Inspiration und gute
Hinweise habe ich über die beiden Seiten [emilykuehler](https://emilykuehler.github.io/bar-chart-race/) und 
[towardsdatascience](https://towardsdatascience.com/how-to-create-plots-with-beautiful-animation-using-gganimate-912f4279b073)
bekommen. 


Die Dynamik soll dadurch entstehen, dass die <span style="font-weight:bold; font-size: 1.2em;">Topx</span> sich
wöchentlich ändern. Daraus entsteht die Notwendigkeit, für jede Woche für **ALLE** Fahrer ein neues Ranking zu erstellen
und daraus die <span style="font-weight:bold; font-size: 1.2em;">Topx</span> zu filtern. 

Das bedeutet einen erheblichen Aufwand bei der Datenaufbereitung, die hier beschrieben werden soll. Ausgangspunkt ist
der Datensatz _ergebB_ der alle Ergebnisse aus den Jahren 2018 und 2019 enthält.


```r
library(tidyverse)
library(here)
library(plotly)

rm(list = ls())

ergebB <- readRDS(here("data", "processed", "ergebB.rds"))
```

Danach werden Teams und Fahrer für jeweils Amateure, KT's und Profis extrahiert.

```r
# ----------------------------------------------------------------
# --- Teams und Fahrer
# ----------------------------------------------------------------
# --- Teams 2019 -------------------------------------------------
alleTeams2019 <- ergebB %>%
  filter(Jahr == 2019) %>%
  distinct(Team)
alleVereine2019 <- ergebB %>%
  filter(Jahr == 2019) %>%
  distinct(Verein)
# --- Profis 2019 -------------------------------------------------
Profi_teams2019 <- alleTeams2019 %>%
  filter(str_detect(Team, "BORA|Trek|Soudal|Quick|Katusha|Sunweb|Bahrain|Lotto Soudal|
                          |Visma|Arkea|Movis|CCC|AG2R|Hrinkow|Mitchelton|Sapura|Corendon|
                          |Sky|INEOS")) %>%
  filter(Team != "Development Team Sunweb")
Profi_teams2019
Profi_fahrer2019 <- ergebB %>% filter(Jahr == 2019) %>% 
  filter(Team %in% Profi_teams2019$Team) %>%
  distinct(Name)
Profi_fahrer2019
# --- KT's 2019 -------------------------------------------------
KT_teams2019 <- ergebB %>% filter(Jahr == 2019) %>%
  filter(is.na(Verein) & !Team %in% Profi_teams2019$Team) %>%
  distinct(Team)
KT_teams2019
KT_fahrer2019 <- ergebB %>% filter(Jahr == 2019) %>%
  filter(Team %in% KT_teams2019$Team) %>%
  distinct(Name)
KT_fahrer2019
# --- Amateure 2019 -------------------------------------------------
Amateur_teams2019 <- ergebB %>% filter(Jahr == 2019) %>%
  filter(!Team %in% Profi_teams2019$Team & !Team %in% KT_teams2019$Team) %>%
  distinct(Team)
Amateur_teams2019
Amateur_fahrer2019 <- ergebB %>% filter(Jahr == 2019) %>%
  filter(Team %in% Amateur_teams2019$Team) %>%
  distinct(Name)
Amateur_fahrer2019
```

Dann werden die Rennen definiert, die **NICHT** mit in die Auswertung kommen sollen. Damit will ich sicherstellen, dass
nur vom BDR ausgeschriebene Rennen berücksichtigt werden. Fahrer, die sich den überwiegenden Teil Ihrer Platzierungen
und Punkte in Afrika oder Indonesien holen werden hier schon einmal rausgefiltert, bzw die dort stattgefundenen Rennen. 

```r
rennenNA <- ergebB %>%
  filter(is.na(Kategorie) | is.na(Ort) |
           str_detect(Veranstaltun, "Rwanda|Mevla|d'Alg|Mersin|Solidarnosc|Tour of Austria|
                              Oberösterreichrundfahrt|GP Adria Mobil (SLO)|Sahara|
                              GP Adria Mobil (SLO)|Umag|Xingtai|Ras Tailteann|Maroc|
                              Deutschland Tour|Rhodes|BinckBank|Wallonie|Slov|
                              York|Dubai|China|Baltic|Indonesia|Rhône-Alpes|
                              Course de la Paix|Carpathian|Qinghai|Antalya|Mallorca|
                              Marche verte|(FRA)|(ROU)|(CHN)|(BEL)|(AUT)|(IRL)|(GBR)|(NED)|
                              (ESP)|(POR)|(SPA)|(IRN)|(MAR)|(RSA)|(SLO)")) %>%
  distinct(Datum, Ort, Veranstaltun)
```

Eine weitere Filterung geschieht im folgenden über den Typ der Rennen.
Außerdem wird das Datum gesäubert und eine column für die Woche eingefügt.

```python
purrTest <- ergebB %>% filter(Jahr == 2019) %>%
  filter(str_detect(Typ, 'SR|RR|Kri.|Kri|BZF|EZF|MZF|GW|Ausscheidungsf.|Temporennen')) %>%
  filter(!str_detect(Kategorie, 'XC')) %>%
  filter(Name %in% Amateur_fahrer2019$Name) %>%
  filter(!Veranstaltun %in% rennenNA$Veranstaltun) %>%
  mutate(Datum = lubridate::dmy(str_sub(Datum,-10))) %>%  
  group_by(week = lubridate::week(Datum), Name, Team)
```

Die folgende loop läuft durch alle Wochen mit Ergebnissen und macht pro Woche ein eigenes Ranking und Addition der
Punkte. 

```r
df_fahrer <- data.frame(matrix(ncol = 5, nrow = 0))
x <- c("Name", "Team", "week", 'Verein',"c")
colnames(df_fahrer) <- x
df_temp <- data.frame()
for (we in seq(10,41,1)){
  df_temp <- purrTest %>% filter(week <= we) %>% 
    group_by(Name, Team, week, Verein) %>% 
    summarise(c = sum(Punkte, na.rm = T))
  df_fahrer <- bind_rows(df_fahrer, df_temp)%>% distinct()
}
```

 Einzelne Fahrer haben in bestimmten Wochen keine Punkte 
 gesammelt. Für die Darstellung muss die Woche aber eine
 kontinuierliche Folge sein. Das wird hier mit **complete**
 gemacht. Danach wird für jede - jetzt lückenlose -
 Wochenfolge neuer Rank und Punkte pro Fahrer ermittelt
 Teams mit NA werden auf 'farblos' gesetzt

```r
fahrer_Rank_Week <- df_fahrer %>% 
  complete(week, nesting(Name, Team, Verein)) %>% 
  unnest() %>% 
  mutate(c = replace_na(c, 0)) %>% ungroup() %>% 
  group_by(Name, Team, Verein) %>% 
  mutate(SumInWeek = cumsum(c)) %>% 
  ungroup() %>% 
  group_by(week) %>% 
  mutate(RankInWeek = min_rank(-SumInWeek)) %>% 
  mutate(Team = ifelse(is.na(Team), 'farblos', Team))
```

Hier werden die Farben für die Teams gemacht - ist bestimmt noch zu verbessern.

```r
nb.cols <- length(unique(fahrer_Rank_Week$Team))
mycolors <- colorRampPalette(brewer.pal(8, "Dark2"))(nb.cols)
tea <-  unique(fahrer_Rank_Week$Team)
teamColor <- data.frame(tea, mycolors) 
teamColor$tea <- as.character(teamColor$tea)
teamColor$mycolors <- as.character(teamColor$mycolors)
teamColor <- teamColor %>% 
  mutate(mycolors = case_when(
    str_detect(tea, 'Ehrmann') ~ 'cadetblue1',
    str_detect(tea, 'Erdinger') ~ 'antiquewhite',
    str_detect(tea, 'Kempten') ~ 'darkgray',
    str_detect(tea, 'Belle') ~ 'aquamarine2',
    str_detect(tea, 'Sigloch') ~ 'bisque2',
    str_detect(tea, 'erdgas') ~ 'lavenderblush2',
    str_detect(tea, 'Kern-Haus') ~ 'lightsalmon',
    TRUE ~ 'gray93'
  ))

whatyouwant <- setNames(as.character(teamColor$mycolors), as.character(teamColor$tea))
```

 Hier wird das gganimate gemacht.
 dat ist wichtig, hier wird eine neue column eingeführt, die 
 für jede Woche eine neue 'ordering' macht. Das dient auch dazu,
 Bei Punktgleichheit Fahrer nicht übereinander darzustellen
 im Ranking sondern hintereinander, geordnet nach Fahrername
 NB: Wenn man in dat den filter(week==40) benutzt und im 
     folgenden ggplot die animation abschaltet, kann man
     man ein statisches Plot machen und das tunen


```r
dat <- fahrer_Rank_Week %>% 
  group_by(week) %>% 
  # nächste Zeile auskommentieren für statischen Plot
  #filter(week == 40) %>% 
  arrange(RankInWeek, Team) %>% 
  mutate(ordering = row_number()) %>% 
  filter(RankInWeek <= 20)
```


Das eigentliche ggplot. Hier kommt **geom_tile** zum Tragen. Das brauch in aes ein y und in dem übergeordnetetn aes wird
das x mit **odering** gemacht. Dann folgen drei künstliche gridlines, die ursprünglichen gingen zu weit oben und unten
hinaus und werden unten abgeschaltet in **themes**. 
Das geom_tile wird halbiert und die _height_ macht die eigentliche Balkenhöhe. Die _width_ wird auf 0.9 gesetzt, damit
Abstand zwischen den einzelnen Balken zu erkennen ist.
über _scale_fill_manual_ werden die Teamfarben für geom_tile gesetzt. Ich hatte auch probiert die Fahrernamen mit den
Teamfarben auszustatten. Aber einige Farben sind etwas zu blaß und deshalb auf dem weißen Hintergrund nicht gut zu
erkennen - deshalb weggelassen.
Dann wird es mit _coord_cartesian_ und _coord_flip_ und _scale_y_reverse_ etwas kompliziert. Vor allem _coord_flip_ switched die Achsen. Nach
langem Rumfummeln habe ich aber dann gemerkt, dass die Achsenbezeichnung bleibt - also die x-Achse ist jetzt die
y-Achse, muss aber nach wie vor als x-Achse angesprochen werden.

Dann geht es an die eigentliche Animation über die Definition der _transition\_states_ . Es wird über **week** animiert.
**transition_length** definiert die Länge des Übergangs zwischen zwei Zuständen und **state_length** die Länge des
Zustandes. Worin die Länge gemessen wird, ist mir aber nicht klar, Sekunden, Millisekunden oder Frames?
Nach langem Experimentieren finden ich für die Darstellung aber diese Einstellung in Ordnung.


```r
anim <- ggplot(data = dat, mapping = aes(x = ordering, group = Name)) + 
  geom_hline(aes(yintercept = 0), alpha =0.1, color = 'gray')+
  geom_hline(aes(yintercept = 500), alpha =0.1, color = 'gray')+
  geom_hline(aes(yintercept = 1000), alpha =0.1, color = 'gray')+
  geom_tile(aes(y = SumInWeek/2, height = SumInWeek, width = .9, fill = Team)) + 
  geom_text(aes(y = -1000, label = paste(RankInWeek, '. ',Name)),
            hjust = 0, nudge_x = 0, size = 4) +
  scale_fill_manual(values = whatyouwant) +
  geom_text(aes(y = -150, label = paste(SumInWeek)),
            hjust = 0, nudge_x = 0, size = 4) +
  geom_text(aes(x = 21, y = max(SumInWeek - 120), 
                label = paste0('Woche: ', week)), col = 'black', alpha=.1, size =6) +
  theme_ridges() +
  scale_y_continuous(
    breaks = c(-500, 0, 500, 1000),
    labels = c('', '0', '500', '1000')
  )+
  theme(legend.position = 'none',
        axis.text.y = element_blank(),
        axis.ticks.x = element_blank(),
        #axis.text.x.bottom = element_blank(),
        panel.grid.major.y = element_blank(),
        panel.grid.major.x = element_blank(),
        plot.title = element_text(size =23)) +
  coord_cartesian(clip = "off", expand = FALSE) + 
  coord_flip() +
  scale_x_reverse() +
  labs(x='', y ='Punkte',
       title = 'Top 20 Amateure 2019', subtitle = '', caption = '@ marblo') +
  transition_states(week, wrap = T, transition_length = 7, state_length = 7) +
  enter_fade() +
  ease_aes('linear') 
```

Das folgende ist jetzt ganz wichtig, um eine saubere Animation zu erhalten. Genaueres wird unter 
[learngganimate](https://github.com/ropenscilabs/learngganimate/blob/master/animate.md) beschrieben.
**nframes** steht für die Anzahl der insgesamt zu erzeugenden frames - das sind eigentlich jeweils einzelne gif's - 
und **fps** für _frames per second_. Die Gesamtdauer der Animation ergibt sich dann aus diesen beiden Größen durch
Division. 
Die Einstellungen für das Speichern sind durch Experimentieren entstanden. Ich kann noch nicht erkennen, dass die
Auflösung **res** einen Einfluss auf die Qualität des Gifs im Browser hat.

```r
animate(anim, nframes = 200, fps =5, end_pause = 10)

# --- speichert letzte Animation als GIF
anim_save('second_I_Top20_res500_200x5_1000x1400.gif',animation = last_animation(),
          ani.width = 1000, ani.height = 1400, res =500)
```

<div class="parent">
  <img src="/img/2020/01/third_II_Top20_res500_300x5_1000x1400.gif">
</div>


Nisi adipisicing aute enim quis sunt non veniam veniam tempor. Sit anim deserunt ad dolore consequat anim commodo. Ipsum
cillum deserunt velit consequat culpa laborum velit quis irure quis pariatur occaecat elit dolor. Ipsum labore aute
tempor ullamco nostrud ullamco. Sit in velit enim eiusmod magna deserunt. Eu pariatur dolore voluptate tempor.


<div style="parent">
   <img src="/img/2020/01/second_III_Top20_res500_400x10_1000x1400.gif">
</div>


