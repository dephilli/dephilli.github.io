---
layout: post
title:  "Unmatched Query in R"
date:   2022-05-19 06:44:00 -0800
categories: r, sql, tidyverse
---

## The SQL Way
An unmatched query is a quick way to find records that exist in one source but don't exist in another source. They are helpful in determining where problems exist, or as a quick way to check your work when complete. In typical SQL coding, you can run a quick unmatched query using the following structure:

{% highlight sql %}
select t1.primary_key
from table_one t1 left join
table_two t2 
on t1.primary_key=t2.primary_key
where t2.primary_key is null 
{% endhighlight%}

Simple enough. Remember, in SQL, we start at the from line. So, take all the records from table 1 and all the matches from table 2. The where clause filters out all the matches and the final query only returns all the records from table 1 that **ARE NOT** in Table 2.  

## How about between R dataframes? 

For this exercise, I've created two dataframes, mlb_teams and bigcities. mlb_teams lists all the Major League Baseball teams from the 2021 season and their Metropolitial Statistical Area (MSA) as defined by the United State Census. big_cities list the top 30 metro areas by population in the United States. 

I want to see the major cities in the US without a Major League Baseball team. If I was using an rdbms, I could adapt the SQL code from above insert mlb_teams to t1 and bigcities to t2, change my select to metro etc...

However, in Base R, this is actually simple and can be done in a single line using subsetting. For this, I'll create a new dataframe called, cities_no_ball. 

{% highlight r %}
cities_no_ball<-big_cities[!big_cities$metro %in% mlb_teams$metro,]
{% endhighlight %}

I'm subsetting the big_cities dataframe to return all the cities that don't match the metros in the mlb_teams file. Simple as that, I find seven results. Most are expected but I can also see that my data might have some issues. My largest MSA without a team, Riverside-San Bernardino-Ontario, CA MSA, isn't one I'm very interested in. This MSA is right next door to the Los Angeles-Long Beach-Anaheim, CA MSA where two teams exist! 

![Big cities without](/assets/images/major_ball.PNG)

Also, I can reverse this and see which baseball clubs aren't in the top 30 MSA. 

{% highlight r %}
small_ball<-mlb_teams[!mlb_teams$metro %in% big_cities$metro,]
{% endhighlight %}

I get five results. Some are expected, but I also see one potential issues. I have one team outside of the United States, the Toronto Blue Jays! My unmatched query helped to highlight these types of data issues. 

![small ball](/assets/images/small_ball.PNG)

## Tidyverse Way

If you're not comfortable using Base R, Tidyverse also can come to the rescue with the anti_join join type in dplyr. Here's a rewrite of our original Base R statement. 

{% highlight r %}
library(dplyr)

big_cities %>% 
  anti_join(mlb_teams, by="metro")
{% endhighlight %}