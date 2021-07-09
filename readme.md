Analysis of US change in performance on the PISA assessment 2000-2018
================

> Analysis by [Matt Wilkins, PhD](https://www.mattwilkinsbio.com)  
> Data downloaded from the [National Center for Education
> Statistics](https://nces.ed.gov/surveys/pisa/idepisa/report.aspx?p=1%C3%81RMS%C3%811%C3%8120183%C3%8020153%C3%8020123%C3%8020093%C3%8020063%C3%8020033%C3%8020003%C3%81PVMATH%C3%81TOTAL%C3%81IN3%C3%80AUS%C3%80AUT%C3%80BEL%C3%80CAN%C3%80CHL%C3%80COL%C3%80CZE%C3%80DNK%C3%80EST%C3%80FIN%C3%80FRA%C3%80DEU%C3%80GRC%C3%80HUN%C3%80ISL%C3%80IRL%C3%80ISR%C3%80ITA%C3%80JPN%C3%80KOR%C3%80LVA%C3%80LUX%C3%80MEX%C3%80NLD%C3%80NZL%C3%80NOR%C3%80POL%C3%80PRT%C3%80SVK%C3%80SVN%C3%80ESP%C3%80SWE%C3%80CHE%C3%80TUR%C3%80GBR%C3%80USA%C3%80RUS%C3%80QCN%C3%80SGP%C3%80ARE%C3%80VNM%C3%81MN%C3%82MN%C3%81Y%C3%82J%C3%810%C3%810%C3%8137%C3%81N&Lang=1033)

##### Just a simple look at PISA statistics for the US and a few other jurisdictions

1.  First, we’ll plot math, reading, and science scores for available
    PISA data from 2000-2018 for the US and a few “jurisdictions” of
    interest.

Below, I include the [R](https://www.r-project.org/) code to wrangle
data and generate figures.

<details>
<summary style="color: #7A7A7A">
Click for code
</summary>

``` r
require(remotes)
install_github("galacticpolymath/galacticPubs") #custom theme stuff
require(pacman)
p_load(ggplot2,readxl,dplyr,galacticPubs)
d<-read_xlsx("data/pisa-data_2000-2018_tidy.xlsx")
#Rename Long name(s)
d$jurisdiction <- recode(d$jurisdiction,`International Average (OECD Countries)`="Avg of OECD Countries")
# Add ranks for each subject-year
d$sub_yr<-paste(d$subject,d$year)
d2<-lapply(unique(d$sub_yr),function(i) {
        d_i <- subset(d,sub_yr==i)
        d_i$rank<-rank(-d_i$average,na.last="keep",ties.method="average")
        d_i
      }) %>% bind_rows()
d2$year <- as.numeric(d2$year)

#define which jurisdictions to compare
for_comparison<-c("Avg of OECD Countries","United States","Canada","Shanghai - China","Russian Federation")

#plot score by year, grouped by subject
G<-d2 %>% subset(.,jurisdiction%in%for_comparison) %>% 
  ggplot(.,aes(x=year,y=average,col=jurisdiction))+
  geom_point()+geom_smooth(method="lm",se=F,formula='y~x')+
  facet_grid(~subject)+ggGalactic(font.cex =.8)+
  theme(strip.text=element_text(size=16))+
  scale_colour_manual(values=gpPal[[1]]$hex[c(6,5,2,4,1,3)])+
  scale_linetype_manual(values=c(3,1,1,1,2))+ylab("PISA average")
```

</details>

![](readme_files/figure-gfm/plot-it-1.png)<!-- -->

##### US scores are about the same as the average for [the 38 OECD peer countries](https://en.wikipedia.org/wiki/OECD). Meanwhile, Shanghai, China is far above everyone else; our Northern neighbor, Canada, is consistently performing higher; and Russia has overtaken the US in math and is rapidly closing the gap in reading.

------------------------------------------------------------------------

2.  Let’s see what it looks like if we combine the subject scores into a
    single value.

<details>
<summary style="color: #7A7A7A">
Click for code
</summary>

``` r
  require(reshape2)
  # Sum all the scores for each country for each year
  d3 = melt(d2[,c("year","jurisdiction","average")],id=c("year","jurisdiction")) %>% dcast(.,formula=jurisdiction+year~variable,fun.aggregate=sum)
  
  #Now plot the result
  G2= d3 %>% subset(.,jurisdiction%in%for_comparison) %>% 
  ggplot(.,aes(x=year,y=average,col=jurisdiction))+ylim(1350,1800)+
  geom_point()+geom_smooth(method="lm",se=F,formula='y~x')+
  ggGalactic(font.cex =.8)+
  theme(strip.text=element_text(size=16))+
  scale_colour_manual(values=gpPal[[1]]$hex[c(6,5,2,4,1,3)])+
  scale_linetype_manual(values=c(3,1,1,1,2))+ylab("Mean Combined PISA score")
```

</details>

![](readme_files/figure-gfm/unnamed-chunk-1-1.png)<!-- --> \#\#\#\#\#
Because the science test wasn’t offered initially (or in the same
years), this reduces the range of our data. But the combined US PISA
scores are incredibly flat for the last decade.

------------------------------------------------------------------------

3.  Next, let’s zoom in on the US line and calculate the slope.

<details>
<summary style="color: #7A7A7A">
Click for code
</summary>

``` r
  us<-subset(d3,jurisdiction=="United States")
  us
  #fit a line (it's just 4 points, but hey, we're not testing for significance)
  mod<-lm(average~year,data=us,na.rm=T)
  mod_form<-paste0("y = ",round(coef(mod)[2],2),"x + ",round(coef(mod)[1],2))
  #make graph and add formula to it
  G3 <- us %>%  ggplot(.,aes(x=year,y=average,col=jurisdiction))+
  geom_point(size=3)+geom_smooth(method="lm",se=F,formula='y~x')+
  ggGalactic(font.cex =.8)+ylim(1350,1800)+
  theme(strip.text=element_text(size=16))+
  scale_colour_manual(values=gpPal[[1]]$hex[1])+
  ylab("Mean Combined PISA score")+annotate("text",label=mod_form,col=gpColors("hydro"),size=10,x=2013,y=1530)
```

</details>

    ##      jurisdiction year  average
    ## 281 United States 2000       NA
    ## 282 United States 2003       NA
    ## 283 United States 2006       NA
    ## 284 United States 2009 1489.226
    ## 285 United States 2012 1476.358
    ## 286 United States 2015 1462.806
    ## 287 United States 2018 1485.978

![](readme_files/figure-gfm/plot-3-1.png)<!-- -->

##### US average combined PISA scores have declined by almost a point per year between 2009 and 2018. That is to say, the aggregate scores have not changed meaningfully, despite [numerous expensive, ongoing educational reforms across the country](https://time.com/5775795/education-reform-failed-america/).
