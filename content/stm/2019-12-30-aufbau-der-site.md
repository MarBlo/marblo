---
title: Aufbau der Site
author: Martin
date: '2019-12-30'
draft: true
slug: aufbau-der-site
categories:
  - stm
  - bicycling
tags: []
description: ''
featured: circuitboard.jpg
featuredalt: circuitboard picture
featuredpath: date
linktitle: ''
type: post
---

# Aufbau der Seite

Der Umgang mit der website wird durch die eigentliche Datenanalyse, die außerhalb des blogdown-Projekts gemacht wird,
sehr schnell sehr groß. Eine gewisse Dokumentation ist deshalb notwendig.

Die gesamte website ist direkt unter _Dokumente_ mit dem Namen <span style="font-weight:bold; font-size:
1.2em;">marblo.netlify.com</span> zu finden. Hier findet man das eigentliche blogdown-Projekt, das <span
style="font-weight:bold; font-size: 1.2em;">marblo</span> heißt, als auch die anderen R-Projekte, die zur Daten- und
Plotaufbereitung dienen. 

# Das Blogdown Projekt

# Die einzelnen Datenprojekte

Keines der hier aufgeführten Projekte sollte ohne genaue Prüfung gelöscht werden. Zwar sind die einzelnen Projekte im
Laufe der Entwicklung immer größer und komplexer geworden und sicher sind nie alle Inhalte der Projekt in dem
eigentlichen <span style="font-weight:bold; font-size: 1.2em;">Blogdown-Projekt</span> angewendet worden. aber einige
kleine Snippets oder sogar Daten werden u.U. in anderen Projekten verwendet.

> Deshalb keine der in dem Ordner <span style="font-weight:bold; font-size: 1.2em;">marblo.netlify.com</span>
> befindlichen Projekte oder auch nur Daten / Scripts o.a. löschen.


## Python Scraping

Das wird später gemacht.

## Datenaufbereitung

### BDR_Dec2019

Die aktuelle Datenaufbereitung wird in dem R-Projekt <span style="font-weight:bold; font-size:
1.2em;">BDR_Dec2019</span> gemacht. Hier werden die gescrapten Rohdaten in der Datei _dataPreparation.R_ eingelesen und
so aufbereitet, dass der wichtige Datensatz <span style="font-weight:bold; font-size: 1.2em;">ergebB.rds</span>
entsteht. Dieser Datensatz wird in verschiedene weitere Projekte kopiert und dient dort als Grundlage für bestimmte
Aufbereitungen / Darstellungen.

Ausgesprochen wichtig in diesem Projekt ist der helper-code _plotFunction.R_, der als Grundlage für die Slope-Diagramme,
die auch in der ShinyApp verwendet werden, dient. In weiteren scripts sind einige Darstellungen von slope-Diagramme
erstellt worden, von denen das eine oder andere es in den Blog geschafft hat.

### prepForShiny_RL

In diesem Projekt ist die eigentliche <span style="font-weight:bold; font-size: 1.2em;">ShinyApp</span> entwickelt und
ausprobiert worden. 

### rangliste2018_18_all

Hier ist die scharfe <span style="font-weight:bold; font-size: 1.2em;">ShinyApp</span> enthalten. Das Projekt hat die
genaue Bezeichnung _rangliste2018_19_all_ . Die eigentliche **APP** ist in `app_III.R` und wird von da aus auf den
Shiny-Server über _publish_ veröffentlicht. Durch hinterlegen, der eigentlichen Kontakt- und Accountdaten geht das mit
dem _publish_.Knopf sehr einfach. In diesem Projekt befinden sich einige Hilfsdateien, wie zum Beispiel auch die
_plotFunction.R_, aber auch einige unterschiedliche Entwicklungsstände der App.

### Animation_RanglisteFahrer

Die Entwicklung einer reinen Amateurrangliste und die Darstellung der Ergebnisse steht hier im Vordergrund.
Auch hier gibt es zwei unterschiedliche Entwicklungsstände in den beiden Dateien _second.R_ und _third.R_. Die erste
dieser Dateien (_first.R_) ist im Laufe der Entwicklung gelöscht worden. Dei beiden Dateien sind sich eigentlich sehr
ähnlich, unterscheiden sich aber s.w. in der Art und Weise der Definitionen von Rennen, die in die Betrachtung für die
Amateur_Rangliste einbezogen werden. Im Detail müssen die Unterschiede zwar noch einmal erarbeitet werden - wenn nötig -
der letzte Stand, sowie die Grundlage für die animierten Plots wird aber in _third.R_ gelegt, allerdings auch noch
einmal in dem script _versuchMitplot\_ly.R_ .

Aber auch in diesem Projekt wird noch einmal - nachdem die o.e. <span style="font-weight:bold; font-size:
1.2em;">ergebB.rds</span> eingelesen wurde. die Datei 

Hier werden die Farben für die einzelnen Teams in der Namensliste <span style="font-weight:bold; font-size:
1.2em;">whatyouwant</span> definiert und gespeichert. Diese Liste wird auch in anderen Darstellungen verwendet und ist
essentiell, um die Farben für die Teams immer konstant zu halten.

In diesem Script werden auch die für die **Amateur_Rangliste** relevanten Rennen definiert, und <span
style="font-weight:bold; font-size: 1.2em;">Ganz Wichtig ! </span> die verschiedenen Rankings abgeleitet. Hier entsteht
auch der DF <span style="font-weight:bold; font-size: 1.2em;">fahrer_Rank_Week</span>, der als _rds_ abgelegt wird und
in späteren Scripts gebraucht wird.

Außerdem werden hier die Animationen entwickelt, die über **gganimate** laufen. Die sehen zwar ganz gut aus und sind ja
auch schon in einem eigenen **st-Beitrag** behandelt worden. Besser und eindeutig publikationsfähig sind aber die mit
<span style="font-weight:bold; font-size: 1.2em;">plotly  </span> entwickelten Animationen. Die befinden sich in dem
Script _versuchMitplot\_ly.R_  und werden auch in dem nächsten Blog verwendet. In diesem script sind also **die letzten
relevanten Daten und Definitionen enthalten**.

> Hier besteht noch Verbesserungspotential und kann einiges zusammengeführt werden.


