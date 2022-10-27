---
title: "Using ggplot2 and plotly with R to produce interactive graphs
excerpt: ""
collection: portfolio
htmlwidgets: TRUE
---

A quick boxplot in R can with a few steps become increasingly detailed.

Using data from uddannelsesstatistik.dk and the `ggplot2` package with the `plotly` package produces a interactable boxplot, that shows the overall grades in the danish schools for the 9th grade for the last 4 years. 

<center><iframe src="/Boxplot.html" height = "500px" width = "60%"></iframe></center>

Created with code:
```r
sti <- "DataSkoler.xlsx"
df <- import(sti) %>% 
                  fill(Fag) %>% 
                  pivot_longer(c("2017/2018", "2018/2019", "2019/2020", 
                                 "2020/2021", "2021/2022"), names_to = "År", values_to = "Elevgennemsnit")

df[df == "Fællesprøve i fysik/kemi, biologi og geografi"] <- "FKBG"

plot <- ggplot(data = df , aes(x = Fag, y = Elevgennemsnit), fill = Fag)
fig <- plot + geom_boxplot() + stat_summary(fun=mean, geom="point", shape=23, size=4) + 
  theme(legend.position="none")

ggplotly(fig)
```
