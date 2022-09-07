---
layout: post
title:  "NPR Puzzle: Largest City in a Country"
date:   2022-09-07 10:44:00 -0800
categories: NPR-Puzzle lapply reduce
---



Every Sunday, [NPR Puzzlemaster Will Shortz](https://www.npr.org/series/4473090/sunday-puzzle), provides a listener challenge, a unique puzzle to solve at home on [NPR's Weekend Edition Sunday](https://www.npr.org/programs/weekend-edition-sunday/). Correct submissions are entered into a drawing and the lucky winner gets to play along with the on air challenge the following week!!! 

Here's an old puzzle from October 2021, from [Blaine's Puzzle Blog](https://puzzles.blainesville.com/2021/10/npr-sunday-puzzle-oct-3-2021-country.html)

>Write down the name of a country and its largest city, one after the other. Hidden in this string, in consecutive letters, is another country's capital (in six letters)? What is it?

Whew, shouldn't be too difficult. I just need a few lists, a little R work and I'm on my way. For this challenge I practiced using the rvest package to scrape data tables from the Internet and the stringr package which plays very nice with strings! 

## Step One -- Get a list of countries and their largest city

So, this took a little bit of browsing but I found what appeared to be a pretty good list from a children's website called [Kiddle](https://kids.kiddle.co/List_of_countries_by_largest_and_second_largest_cities). This list has what I need so I ran this query to scrape the data and return a dataframe. 

{% highlight r %}
library(stringr)
library(rvest)
url <- "https://kids.kiddle.co/List_of_countries_by_largest_and_second_largest_cities"
country_largest <- url %>%
  read_html() %>%
  html_nodes(xpath='/html/body/div/div/div[2]/div[3]/div[1]/table') %>%
  html_table()
country_largest <- country_largest[[1]]
{% endhighlight %}

The table wasn't quite designed with data extraction in mind so it's not entirely perfect. It looks like the split up headers are combined into the first three rows of the dataframe. Also, this has more information than I need. 

{% highlight r %}
country_largest<-country_largest[4:nrow(country_largest),c(1:2)]
{% endhighlight %}

This is how I like to run simple subsets. The brackets [,] refer to the rows and vectors of the dataframe. I'm telling R to select all rows from row number 4 to the end. Then, I'm telling r to also give me the first two vectors. 

Next, I want a little cleanup and I want to combine the two vectors into a third vector as a single unspaced string using the str_to_upper function from the stringr package. 
{% highlight r %}
country_largest[1:2] <-
  lapply(country_largest[1:2], str_to_upper)

country_largest$combined <-
  paste0(country_largest$`Country or territory`,
         country_largest$`City proper`)
{% endhighlight %}

The first statement takes each vector and replaces it with a fully capitalized string. I like to do this so I don't run into any case issues. The second uses the paste0 function to build the full string for Part 1 of the puzzle 

{% highlight r %}
names(country_largest) <- c("country", "city", "combined")
{% endhighlight %}

Before moving forward, I wanted to change the names to be a bit more friendly. I want to get rid of extra spaces and have simple understandable names. 

## Step Two -- Get World Capitals

Wikipedia to the rescue! [Everything I need is right here](https://en.wikipedia.org/wiki/List_of_national_capitals)! Also, I'm pulling in the stringr package. 

{% highlight r %}
url <- "https://en.wikipedia.org/wiki/List_of_national_capitals"
capitals <- url %>%
  read_html() %>%
  html_nodes(xpath = '/html/body/div[3]/div[3]/div[5]/div[1]/table[2]') %>%
  html_table()
capitals <- capitals[[1]][, 1]
{% endhighlight %}

I added [,1] to this to return only the capital cities. 

Now for some clean up. I used str_extract from stringr and REGEX to extract the string before an open parenthesis ( 
There's a lot of stuff extra notes after the parenthesis on some entries. Since I'm not a regex expert I then ran trimws() to remove any additional whitespace. Finally, I uppercased everything. During this work, capitals becomes a vector and is no longer in a dataframe. 

{% highlight r %}
capitals <- str_extract(capitals$`City/Town`, "[^(]+")

capitals <- trimws(capitals)

capitals <- stringr::str_to_upper(capitals)
{% endhighlight %}

## Step Three -- Solve the Puzzle 

My solution is a bit sloppy. I decided to lump everything together rather than breaking it into individual pieces. I also used an anonymous function instead of a named function as this is only going to be used once to solve this particular puzzle. Had this been a larger project that would use code to reuse in the future, I would have used a name function instead. 

{% highlight r %}
solution_df <-
  do.call(rbind, lapply(1:nrow(country_largest), function(x) {
    out <-
      unique(str_extract(country_largest[x, ]$combined, capitals))
    if (length(out) > 1) {
      out <- out[(!is.na(out))]
    }
    final <- data.frame(country_largest[x, ], out)
    if (final$out != final$city & !is.na(final$out)) {
      return(final)
    }
  }))
{% endhighlight %}

So, what is this code doing? 

Breaking it down starting in line 2
{% highlight r %}
do.call(rbind, lapply(1:nrow(country_largest), function(x) {
{% endhighlight %}

Three things are starting here:  
1. An anonymous function - Where most of the work is happening. 
    * First, this is creating an in function variable that I have called "out". Out is looking at row x of country largest and is looking to extract anywhere where the list of capital is occurring in the combined (country+largest city) field. So, if x==249, out would look at the line for the United States of AmericaNew York City. This would not return NA for every entry in the capital vector. 
    * Since out is wrapped in the unique() function, the United States would get a single entry for out<-NA 
    * Next there's an if statement, if a country does return a value and NA, outs would be greater than 1. In this case, out will be equal to the not NA value. 
    * Next is building a dataframe this frame returns information from the row and includes the out variable. This dataframe is called final
    * Finally the last if statement is in place to build our return. If the out variable is NA or if the out variable matches the largest city, no return is done. While the rules don't technically specify this, it seems like the answer is invalidated if the country's largest city is also its capital. Hopefully this will just return a single answer, the correct answer! 
2. A lapply function -- Pretty straight forward, the lapply is iterating every row of the country_largest dataframe and returning the variable final in a list 
3. A do.call function -- Confusing but this is just combining each listed dataframe returned from the lapply to a single dataframe. 

## The Answer
Ideally, solution_df should return a single value. However, I have six. Five of these are from slight differences between the two datasets I brought in. This highlights how important it is to have normalized data when doing deep analysis. Luckily, since this is just a puzzle and our data set is quite small, I can eyeball this dataframe and see our correct answer. Karachi, Pakistan combines to KARACHIPAKISTAN which has the string ANKARA (largest city in Turkey) combined inside it. 

[Here's the full code](https://github.com/dephilli/NPR-Puzzle-Using-R/blob/master/largest%20cities.Rmd)

[For more puzzles check out Blaine's Puzzle Blog!](https://puzzles.blainesville.com/)